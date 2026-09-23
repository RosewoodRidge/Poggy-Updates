--[[===========================================================================
    poggy_crafting · client/main.lua
    ---------------------------------------------------------------------------
    The loop that decides whether the player is standing at something they can
    craft on: a bench from Config.Locations, or one of the world props in
    Config.CraftingProps.

    It sleeps for a second at a time until something is in range, and only then
    runs every frame -- a prop lookup is the most expensive call in this script
    and the reason Config.CraftingPropsEnabled exists.
===========================================================================]]--

PoggyCrafting = PoggyCrafting or {}
local PC = PoggyCrafting
local T  = PC.T

local promptGroup, craftPrompt
local blipsPlaced = false
local job         = nil    -- the player's job name, refreshed on open
local target      = nil    -- the bench or prop in range, or nil

local function distance(a, b)
    return #(vector3(a.x, a.y, a.z) - b)
end

--- Does this player have the job a Job field asks for? The prompt uses the
--- cached answer; the server checks again before anything is taken.
local function jobAllows(list)
    return PC.Allows(list, job)
end

--- Read the job again. This script starts on the client before the player
--- picks a character, when there is no job to read, so a job read only once
--- stays nil all session and every job-locked bench hides its prompt. It is
--- read again when a character loads, when the job changes, and on a timer.
local function refreshJob()
    job = PC.Ask('poggy_crafting:job')
end

AddEventHandler('poggy_core:charLoadedLocal', function()
    CreateThread(refreshJob)
end)

-- The event carries the new job name; the server is asked anyway, so a grade
-- change (which carries the old name) and a missed field both come out right.
AddEventHandler('poggy_core:jobChangedLocal', function()
    CreateThread(refreshJob)
end)

-- Every 5 s while there is no job yet (still at character select), every 60 s
-- after that, for a job set by something that sends no change event.
CreateThread(function()
    while true do
        Wait(job and 60000 or 5000)
        refreshJob()
    end
end)

-- ===========================================================================
--  Blips
-- ===========================================================================

local function placeBlips()
    if blipsPlaced then return end
    for _, loc in ipairs(Config.Locations) do
        if loc.Blip and loc.Blip.enable then
            local blip = BlipAddForCoords(1664425300, loc.x, loc.y, loc.z)
            SetBlipSprite(blip, joaat(loc.Blip.Hash), true)
            SetBlipScale(blip, loc.Blip.scale or 0.2)
            SetBlipName(blip, loc.name)
        end
    end
    blipsPlaced = true
end

-- ===========================================================================
--  Startup
-- ===========================================================================

CreateThread(function()
    -- Keep waiting rather than giving up: a client whose core took longer than
    -- PoggyReady's 30 s used to lose every bench prompt for the session.
    while not PoggyReady() do Wait(5000) end

    promptGroup = PoggyPromptGroup:new(T('prompt_bench'))
    craftPrompt = PoggyPrompt:new(Config.PromptControl, T('prompt_craft'), promptGroup)
    craftPrompt:setStandardMode(true)

    -- The press is handled here rather than polled below, so the prompt
    -- library owns the rising edge and the loop only decides what is in range.
    craftPrompt:on('standardJustCompleted', function()
        if not target or PC.uiOpen or PC.crafting then return end
        -- Re-read the job on open so a change made since the last one (a
        -- multijob switch, going on duty) counts.
        job = PC.Ask('poggy_crafting:job')
        if jobAllows(target.Job) then
            PC.OpenUI(target)
        else
            Poggy('notify.styled', { style = 'right', text = T('not_job'), duration = 4000 })
        end
    end)

    refreshJob()
    placeBlips()

    while true do
        local sleep = 1000

        if not PC.uiOpen and not PC.crafting then
            local ped = PlayerPedId()

            if not IsEntityDead(ped) then
                local coords = GetEntityCoords(ped)
                target = nil

                -- World props: campfires, ovens, forges.
                if Config.CraftingPropsEnabled and jobAllows(Config.CampfireJobLock) then
                    for _, entry in ipairs(Config.CraftingProps) do
                        local props = type(entry.prop) == 'table' and entry.prop or { entry.prop }
                        for _, model in ipairs(props) do
                            if DoesObjectOfTypeExistAtCoords(coords.x, coords.y, coords.z,
                                Config.Distances.campfire, joaat(model), false) then
                                target = { id = entry.title:lower(), name = entry.title }
                                break
                            end
                        end
                        if target then break end
                    end
                end

                -- Fixed benches. A bench in range wins over a prop, because it
                -- is the more specific thing to be standing at.
                for _, loc in ipairs(Config.Locations) do
                    if jobAllows(loc.Job) and distance(loc, coords) < Config.Distances.locations then
                        target = loc
                        break
                    end
                end

                if target then
                    sleep = 0
                    promptGroup:setText(target.name)
                    promptGroup:handleEvents()
                end
            end
        end

        -- The game's own prompts would draw over the browser.
        if PC.uiOpen or PC.crafting then
            sleep = 0
            UiPromptDisablePromptsThisFrame()
        end

        Wait(sleep)
    end
end)

-- ===========================================================================
--  /craft
-- ===========================================================================

-- Opens the browser anywhere. Recipes and categories that name a Location are
-- still out of reach, so this is a way to read the book, not to skip the bench.
if Config.Commands.craft then
    RegisterCommand(Config.Commands.craft, function()
        if PC.uiOpen or PC.crafting then return end
        if IsEntityDead(PlayerPedId()) then return end
        job = PC.Ask('poggy_crafting:job')
        PC.OpenUI({ id = nil, name = T('ui_title'), Categories = 0, Job = 0 })
    end, false)
end
