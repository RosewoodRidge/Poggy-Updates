-- ============================================================================
-- File: shared/api.lua
-- Description: Public export / API allowing other resources to trigger the
--              FULL witness mechanic for custom actions (grave robbing, illegal
--              NPC sales, custom robberies, kidnapping systems, scripted RP
--              crimes, etc.).
--
--              Unlike the simple alert commands, this runs the normal Witnesses
--              flow:
--                1. Finds / assigns a nearby NPC witness.
--                2. Makes the witness react and flee.
--                3. Lets the player stop the witness (kill / detain / chase off).
--                4. Only fires the law / job alert if the witness SUCCESSFULLY
--                   reports the crime.
--                5. Optionally triggers the NPC law response based on the
--                   configured / supplied action type.
--
-- This file is loaded as a shared_script and branches on IsDuplicityVersion()
-- so the server export and client handler live together.
-- ============================================================================
--
-- SERVER USAGE (from another resource's server script):
--
--   local requestId = exports.poggy_witnesses:CreateWitness(source, "GraveRobbing", {
--       radius              = 25.0,                 -- search radius for an NPC witness (default 20.0)
--       alertCommand        = "graverobbing8x2kp19",-- a Config.Alerts command to fire if reported
--       triggerLawResponse  = true,                 -- spawn the NPC law response if reported
--       lawActionType       = "Shooting",           -- maps to Config.LawResponse.CrimeSeverity (default = actionType)
--       forceWitness        = false,                -- if no NPC is nearby, still report after a short delay
--       showNotifications   = true,                 -- show the "Someone saw what you did" toast (default true)
--       timeout             = 60000,                -- max time (ms) to wait for a report (default 60000)
--   })
--
-- CLIENT USAGE (from another resource's client script):
--
--   exports.poggy_witnesses:CreateWitnessLocal("GraveRobbing", { alertCommand = "graverobbing8x2kp19" })
--
-- The `actionType` may be ANY string. If it matches an entry in
-- Config.LawResponse.CrimeSeverity the law response severity is resolved
-- automatically; otherwise supply `lawActionType` to map it.
--
-- ----------------------------------------------------------------------------
-- SERVER EVENTS (listen with AddEventHandler in your own server resource):
--
--   AddEventHandler('poggy_witnesses:onWitnessCreated',  function(src, actionType, requestId, data) end)
--   AddEventHandler('poggy_witnesses:onWitnessStopped',  function(src, actionType, requestId, data) end) -- killed / chased off / no report
--   AddEventHandler('poggy_witnesses:onWitnessReported', function(src, actionType, requestId, data) end) -- escaped & reported
--   AddEventHandler('poggy_witnesses:onAlertSent',       function(src, actionType, requestId, alertCommand) end)
--
-- ============================================================================

if IsDuplicityVersion() then
    -- ========================================================================
    -- SERVER SIDE
    -- ========================================================================

    -- requestId -> { source, actionType, options }
    local pendingApiWitnesses = {}

    local function generateRequestId()
        return string.format("wapi_%d_%d", os.time(), math.random(100000, 999999))
    end

    -- Core entry point shared by the export and the client-initiated path.
    local function StartWitnessRequest(src, actionType, options)
        src = tonumber(src)
        if not src or src <= 0 then
            return false
        end
        if type(actionType) ~= "string" or actionType == "" then
            actionType = "Shooting"
        end
        if type(options) ~= "table" then
            options = {}
        end

        local requestId = generateRequestId()
        pendingApiWitnesses[requestId] = {
            source = src,
            actionType = actionType,
            options = options,
        }

        -- Relay to the player's client to run the witness flow locally.
        TriggerClientEvent('poggy_witnesses:api:spawn', src, requestId, actionType, options)
        return requestId
    end

    -- Public export: exports.poggy_witnesses:CreateWitness(source, actionType, options)
    exports('CreateWitness', function(source, actionType, options)
        return StartWitnessRequest(source, actionType, options)
    end)

    -- Allow client-initiated requests (from the CreateWitnessLocal client export)
    -- to go through the exact same routing so server events / alerts still fire.
    RegisterNetEvent('poggy_witnesses:api:request', function(actionType, options)
        StartWitnessRequest(source, actionType, options)
    end)

    -- Stage updates coming back from the client as the witness flow progresses.
    RegisterNetEvent('poggy_witnesses:api:stage', function(requestId, stage, data)
        local src = source
        local req = pendingApiWitnesses[requestId]

        -- Validate the update belongs to the player who owns the request.
        if not req or req.source ~= src then
            return
        end

        local actionType = req.actionType
        local options = req.options or {}
        data = type(data) == "table" and data or {}

        if stage == "created" then
            TriggerEvent('poggy_witnesses:onWitnessCreated', src, actionType, requestId, data)

        elseif stage == "stopped" then
            TriggerEvent('poggy_witnesses:onWitnessStopped', src, actionType, requestId, data)
            pendingApiWitnesses[requestId] = nil

        elseif stage == "reported" then
            TriggerEvent('poggy_witnesses:onWitnessReported', src, actionType, requestId, data)

            -- Fire the configured law / job alert (reusing Config.Alerts).
            local alertCommand = options.alertCommand
            if alertCommand then
                for _, alert in ipairs(Config.Alerts or {}) do
                    if alert.command == alertCommand then
                        Citizen.CreateThread(function()
                            AlertPlayer(src, alert)
                        end)
                        TriggerEvent('poggy_witnesses:onAlertSent', src, actionType, requestId, alertCommand)
                        break
                    end
                end
            end

            pendingApiWitnesses[requestId] = nil
        end
    end)

    -- Clean up any dangling requests when a player disconnects.
    AddEventHandler('playerDropped', function()
        local src = source
        for id, req in pairs(pendingApiWitnesses) do
            if req.source == src then
                pendingApiWitnesses[id] = nil
            end
        end
    end)

else
    -- ========================================================================
    -- CLIENT SIDE
    -- ========================================================================

    -- Locate the focal "victim" ped for the crime: a nearby valid NPC the
    -- witness search will exclude and center around (mirrors the command flow).
    local function FindApiWitnessTarget(radius)
        local playerPed = PlayerPedId()
        local coords = GetEntityCoords(playerPed)
        local itemSet = CreateItemset(true)
        local size = Citizen.InvokeNative(0x59B57C4B06531E1E, coords, radius, itemSet, 1, Citizen.ResultAsInteger())
        local target = nil

        if size > 0 then
            for i = 0, size - 1 do
                local entity = GetIndexedItemInItemset(i, itemSet)
                if DoesEntityExist(entity) and IsEntityAPed(entity) and not IsPedAPlayer(entity)
                    and not IsEntityDead(entity) and entity ~= playerPed and CanBeVictim(entity) then
                    target = entity
                    break
                end
            end
        end

        if IsItemsetValid(itemSet) then
            DestroyItemset(itemSet)
        end
        return target
    end

    local function ShowSawNotification()
        if Config.DisableTopNotifications then return end
        if type(Notify) ~= "table" or type(Notify.Top) ~= "function" then return end
        local titleText = (type(T) == "function") and T("WITNESS_TITLE") or "WITNESS"
        local messageText = (type(T) == "function") and T("SOMEONE_SAW") or "Someone saw what you did!"
        Notify.Top(titleText, messageText, 5000)
    end

    -- Runs the full witness flow for an API request and reports each stage back
    -- to the server so public events / alerts can be dispatched.
    local function HandleApiWitness(requestId, actionType, options)
        options = type(options) == "table" and options or {}
        local radius            = tonumber(options.radius) or 20.0
        local timeout           = tonumber(options.timeout) or 60000
        local force             = options.forceWitness == true
        local showNotifications = options.showNotifications ~= false
        local triggerLaw        = options.triggerLawResponse == true
        local lawActionType     = options.lawActionType or actionType

        -- Snapshot witnesses that already exist so we only track the ones we spawn.
        local baseline = {}
        for id in pairs(activeWitnesses or {}) do
            baseline[id] = true
        end

        local target = FindApiWitnessTarget(radius)

        -- No suitable NPC nearby.
        if not target then
            if force then
                if showNotifications then ShowSawNotification() end
                TriggerServerEvent('poggy_witnesses:api:stage', requestId, 'created', { forced = true })
                Citizen.Wait(3000)
                if triggerLaw then
                    TriggerEvent('poggy_witnesses:TriggerLawResponse', lawActionType)
                end
                TriggerServerEvent('poggy_witnesses:api:stage', requestId, 'reported', { forced = true })
            else
                TriggerServerEvent('poggy_witnesses:api:stage', requestId, 'stopped', { reason = 'no_target' })
            end
            return
        end

        -- Kick off the normal witness creation pipeline.
        CreateWitness(target, actionType)

        local startTime = GetGameTimer()
        local ourWitnesses = {}
        local firedCreated = false
        local sawAny = false

        while true do
            Citizen.Wait(500)

            local anyActive = false
            local reported = false

            for id, dataEntry in pairs(activeWitnesses or {}) do
                if not baseline[id] then
                    ourWitnesses[id] = true
                end
                if ourWitnesses[id] then
                    anyActive = true
                    sawAny = true
                    if dataEntry.state == "Reported" then
                        reported = true
                    end
                end
            end

            -- Fire "created" once the first witness for this request is tracked.
            if not firedCreated and sawAny then
                firedCreated = true
                TriggerServerEvent('poggy_witnesses:api:stage', requestId, 'created', {})
            end

            if reported then
                -- Core already triggers law response when the witnessed actionType
                -- is in Config.LawResponse.CrimeSeverity; this covers custom types.
                if triggerLaw then
                    TriggerEvent('poggy_witnesses:TriggerLawResponse', lawActionType)
                end
                TriggerServerEvent('poggy_witnesses:api:stage', requestId, 'reported', {})
                return
            end

            -- All of our witnesses are gone and none reported => stopped / killed.
            if sawAny and not anyActive then
                TriggerServerEvent('poggy_witnesses:api:stage', requestId, 'stopped', { reason = 'stopped' })
                return
            end

            if (GetGameTimer() - startTime) > timeout then
                if force and sawAny then
                    if triggerLaw then
                        TriggerEvent('poggy_witnesses:TriggerLawResponse', lawActionType)
                    end
                    TriggerServerEvent('poggy_witnesses:api:stage', requestId, 'reported', { reason = 'timeout_forced' })
                else
                    TriggerServerEvent('poggy_witnesses:api:stage', requestId, 'stopped', { reason = sawAny and 'timeout' or 'no_witness' })
                end
                return
            end
        end
    end

    -- Server -> client trigger for a witness request.
    RegisterNetEvent('poggy_witnesses:api:spawn', function(requestId, actionType, options)
        Citizen.CreateThread(function()
            HandleApiWitness(requestId, actionType, options)
        end)
    end)

    -- Client convenience export: exports.poggy_witnesses:CreateWitnessLocal(actionType, options)
    -- Routes through the server so the same events / alerts are dispatched.
    exports('CreateWitnessLocal', function(actionType, options)
        TriggerServerEvent('poggy_witnesses:api:request', actionType, options)
    end)
end
