--[[
    poggy_core — prompt library (client).

    Include it from any resource that wants on-screen control prompts:

        client_scripts {
            '@poggy_core/client/lib/prompts.lua',
            'client/my_script.lua',
        }

    It runs inside the INCLUDING resource, not inside poggy_core, so the prompts
    it creates belong to that resource and are deleted when that resource stops.
    A prompt is a per-frame UI object, which is why this is a library you include
    rather than a Poggy() verb: nothing that has to be redrawn every frame can
    travel through a request/response channel.

    Written for poggy_core directly against the RedM prompt natives. It replaces
    the third-party uiprompt library, which has no licence and so cannot ship
    inside a paid resource. The few method names scripts already relied on
    (setText, setEnabled, handleEvents) are kept, so moving a script across is a
    change to its constructors and nothing else.

        local group = PoggyPromptGroup:new("Balloon")
        local boost = PoggyPrompt:new(`INPUT_CONTEXT_B`, "Boost", group)
        local axis  = PoggyPrompt:new({ `INPUT_VEH_MOVE_LEFT_ONLY`, `INPUT_VEH_MOVE_RIGHT_ONLY` }, "Left/Right", group)

        -- every frame the group should be on screen:
        group:handleEvents()

    Optional event handlers, on a prompt or on a whole group:

        boost:on("justPressed", function(prompt) ... end)
        group:on("holdJustCompleted", function(group, prompt) ... end)

    Event names: justPressed, justReleased, pressed, released,
    standardCompleted, standardJustCompleted, holdRunning, holdCompleted,
    holdJustCompleted, mashCompleted, mashJustCompleted, controlPressed,
    controlJustPressed, controlReleased, controlJustReleased.
]]

local function truthy(v)
    return v == true or v == 1
end

local function label(text)
    return CreateVarString(10, "LITERAL_STRING", tostring(text or ""))
end

local function controlHash(control)
    if type(control) == "string" then return GetHashKey(control) end
    return control
end

-- Everything this resource has created, so it can all be deleted on stop.
PoggyPrompts = PoggyPrompts or { prompts = {}, groups = {} }
local Registry = PoggyPrompts

-- ---------------------------------------------------------------------------
-- Rising-edge detection
--
-- The natives only answer "has this completed". "Just completed" means it is
-- true this frame and was not last frame. The answer is cached per game-timer
-- tick, so asking twice in one frame gives the same result instead of true then
-- false. GetGameTimer rather than GetFrameCount: nothing on this build calls
-- GetFrameCount, and a library whose job is to just work should not rest on a
-- native nobody has proven here. The game timer is latched once per frame.
-- ---------------------------------------------------------------------------

local function edge(self, key, current)
    local frame = GetGameTimer()
    local st = self._edges[key]
    if not st then
        st = { frame = -1, was = false, result = false }
        self._edges[key] = st
    end
    if st.frame ~= frame then
        st.result = current and not st.was
        st.was = current
        st.frame = frame
    end
    return st.result
end

-- ---------------------------------------------------------------------------
-- PoggyPrompt
-- ---------------------------------------------------------------------------

PoggyPrompt = {}
PoggyPrompt.__index = PoggyPrompt

--- Create a prompt.
--- @param controls number|string|table one control, or a list shown as one prompt
--- @param text string the label
--- @param group table|number|nil a PoggyPromptGroup, or a raw group id
--- @param enabled boolean|nil false to create it disabled and hidden
function PoggyPrompt:new(controls, text, group, enabled)
    local self = setmetatable({}, PoggyPrompt)

    self.controls = type(controls) == "table" and controls or { controls }
    for i, c in ipairs(self.controls) do
        self.controls[i] = controlHash(c)
    end
    self.handlers = {}
    self._edges   = {}

    self.handle = PromptRegisterBegin()
    for _, c in ipairs(self.controls) do
        PromptSetControlAction(self.handle, c)
    end
    self:setText(text)

    -- Only disabling is explicit. A new prompt is enabled and visible already,
    -- and no mode is forced, so a script moved onto this library looks exactly
    -- as it did before.
    if enabled == false then
        PromptSetEnabled(self.handle, false)
        PromptSetVisible(self.handle, false)
    end

    if type(group) == "table" then
        group:add(self)
    elseif group then
        PromptSetGroup(self.handle, group, 0)
    end

    PromptRegisterEnd(self.handle)

    Registry.prompts[self] = true
    return self
end

function PoggyPrompt:getHandle() return self.handle end
function PoggyPrompt:getText()   return self.text end

function PoggyPrompt:setText(text)
    self.text = text
    PromptSetText(self.handle, label(text))
    return self
end

function PoggyPrompt:setEnabled(toggle)
    PromptSetEnabled(self.handle, toggle and true or false)
    return self
end

function PoggyPrompt:setVisible(toggle)
    PromptSetVisible(self.handle, toggle and true or false)
    return self
end

function PoggyPrompt:setEnabledAndVisible(toggle)
    return self:setEnabled(toggle):setVisible(toggle)
end

function PoggyPrompt:isEnabled() return truthy(PromptIsEnabled(self.handle)) end
function PoggyPrompt:isActive()  return truthy(PromptIsActive(self.handle)) end
function PoggyPrompt:isValid()   return truthy(PromptIsValid(self.handle)) end

-- --- modes --------------------------------------------------------------------

function PoggyPrompt:setStandardMode(toggle)
    PromptSetStandardMode(self.handle, toggle ~= false)
    return self
end

function PoggyPrompt:setHoldMode(toggle)
    PromptSetHoldMode(self.handle, toggle ~= false)
    return self
end

function PoggyPrompt:setMashMode(count)
    PromptSetMashMode(self.handle, tonumber(count) or 1)
    return self
end

function PoggyPrompt:setMashIndefinitelyMode()
    PromptSetMashIndefinitelyMode(self.handle)
    return self
end

-- --- prompt state ---------------------------------------------------------------

function PoggyPrompt:isJustPressed()  return truthy(PromptIsJustPressed(self.handle)) end
function PoggyPrompt:isJustReleased() return truthy(PromptIsJustReleased(self.handle)) end
function PoggyPrompt:isPressed()      return truthy(PromptIsPressed(self.handle)) end
function PoggyPrompt:isReleased()     return truthy(PromptIsReleased(self.handle)) end

function PoggyPrompt:hasStandardModeCompleted() return truthy(PromptHasStandardModeCompleted(self.handle)) end
function PoggyPrompt:isHoldModeRunning()        return truthy(PromptIsHoldModeRunning(self.handle)) end
function PoggyPrompt:hasHoldModeCompleted()     return truthy(PromptHasHoldModeCompleted(self.handle)) end
function PoggyPrompt:hasMashModeCompleted()     return truthy(PromptHasMashModeCompleted(self.handle)) end

function PoggyPrompt:hasStandardModeJustCompleted()
    return edge(self, "standard", self:hasStandardModeCompleted())
end

function PoggyPrompt:hasHoldModeJustCompleted()
    return edge(self, "hold", self:hasHoldModeCompleted())
end

function PoggyPrompt:hasMashModeJustCompleted()
    return edge(self, "mash", self:hasMashModeCompleted())
end

-- --- raw control state, across every control the prompt is bound to -----------
-- "Pressed" is true if ANY bound control is pressed. "Released" is true only
-- when ALL of them are, so a two-key axis prompt is not released while one key
-- is still held.

local function anyControl(self, native, pad)
    for _, c in ipairs(self.controls) do
        if native(pad or 0, c) then return true end
    end
    return false
end

local function allControls(self, native, pad)
    for _, c in ipairs(self.controls) do
        if not native(pad or 0, c) then return false end
    end
    return true
end

function PoggyPrompt:isControlPressed(pad)      return anyControl(self, IsControlPressed, pad) end
function PoggyPrompt:isControlJustPressed(pad)  return anyControl(self, IsControlJustPressed, pad) end
function PoggyPrompt:isControlJustReleased(pad) return anyControl(self, IsControlJustReleased, pad) end
function PoggyPrompt:isControlReleased(pad)     return allControls(self, IsControlReleased, pad) end

function PoggyPrompt:disableControlAction(pad)
    for _, c in ipairs(self.controls) do DisableControlAction(pad or 0, c, true) end
    return self
end

function PoggyPrompt:enableControlAction(pad)
    for _, c in ipairs(self.controls) do EnableControlAction(pad or 0, c, true) end
    return self
end

-- --- events -----------------------------------------------------------------------

local CHECKS = {
    { "justPressed",           PoggyPrompt.isJustPressed },
    { "justReleased",          PoggyPrompt.isJustReleased },
    { "pressed",               PoggyPrompt.isPressed },
    { "released",              PoggyPrompt.isReleased },
    { "standardCompleted",     PoggyPrompt.hasStandardModeCompleted },
    { "standardJustCompleted", PoggyPrompt.hasStandardModeJustCompleted },
    { "holdRunning",           PoggyPrompt.isHoldModeRunning },
    { "holdCompleted",         PoggyPrompt.hasHoldModeCompleted },
    { "holdJustCompleted",     PoggyPrompt.hasHoldModeJustCompleted },
    { "mashCompleted",         PoggyPrompt.hasMashModeCompleted },
    { "mashJustCompleted",     PoggyPrompt.hasMashModeJustCompleted },
    { "controlPressed",        PoggyPrompt.isControlPressed },
    { "controlJustPressed",    PoggyPrompt.isControlJustPressed },
    { "controlReleased",       PoggyPrompt.isControlReleased },
    { "controlJustReleased",   PoggyPrompt.isControlJustReleased },
}

local VALID = {}
for _, c in ipairs(CHECKS) do VALID[c[1]] = true end

local function checkEventName(name)
    if not VALID[name] then
        print(("^3[poggy_core:prompts]^7 unknown prompt event '%s' in %s")
            :format(tostring(name), GetCurrentResourceName()))
        return false
    end
    return true
end

--- Attach a handler. fn(prompt, ...) — the extra arguments are whatever was
--- passed to handleEvents.
function PoggyPrompt:on(name, fn)
    if checkEventName(name) then self.handlers[name] = fn end
    return self
end

--- Fire this prompt's handlers, and its group's. Call every frame it matters.
--- Checks are only made for events someone is listening to, so an unused prompt
--- costs no native calls here.
function PoggyPrompt:handleEvents(...)
    if not self:isEnabled() then return end
    local group = self.group
    for _, c in ipairs(CHECKS) do
        local name, check = c[1], c[2]
        local own = self.handlers[name]
        local shared = group and group.handlers[name]
        if (own or shared) and check(self) then
            if own then own(self, ...) end
            if shared then shared(group, self, ...) end
        end
    end
end

function PoggyPrompt:delete()
    if self.group then
        local list = self.group.prompts
        for i = #list, 1, -1 do
            if list[i] == self then table.remove(list, i) end
        end
        self.group = nil
    end
    Registry.prompts[self] = nil
    PromptDelete(self.handle)
end

-- ---------------------------------------------------------------------------
-- PoggyPromptGroup
-- ---------------------------------------------------------------------------

PoggyPromptGroup = {}
PoggyPromptGroup.__index = PoggyPromptGroup

--- Create a group. Its prompts are shown together under one title.
--- @param text string the group title
--- @param active boolean|nil false to create it hidden
function PoggyPromptGroup:new(text, active)
    local self = setmetatable({}, PoggyPromptGroup)
    self.groupId  = GetRandomIntInRange(0, 0xFFFFFF)
    self.text     = text
    self.prompts  = {}
    self.handlers = {}
    self.active   = active ~= false
    Registry.groups[self] = true
    return self
end

function PoggyPromptGroup:getGroupId() return self.groupId end
function PoggyPromptGroup:getText()    return self.text end
function PoggyPromptGroup:getPrompts() return self.prompts end
function PoggyPromptGroup:isActive()   return self.active end

function PoggyPromptGroup:setText(text)
    self.text = text
    return self
end

function PoggyPromptGroup:setActive(toggle)
    self.active = toggle ~= false
    return self
end

--- Add a prompt. Accepts a PoggyPrompt or a raw prompt handle.
function PoggyPromptGroup:add(prompt)
    if type(prompt) == "table" then
        PromptSetGroup(prompt.handle, self.groupId)
        prompt.group = self
        self.prompts[#self.prompts + 1] = prompt
    else
        PromptSetGroup(prompt, self.groupId, 0)
    end
    return prompt
end
PoggyPromptGroup.addPrompt = PoggyPromptGroup.add

--- Put the group on screen for this frame only.
function PoggyPromptGroup:show()
    -- Two arguments, exactly as uiprompt calls it.
    PromptSetActiveGroupThisFrame(self.groupId, label(self.text))
    return self
end
PoggyPromptGroup.setActiveThisFrame = PoggyPromptGroup.show

--- Attach a handler for every prompt in the group. fn(group, prompt, ...).
function PoggyPromptGroup:on(name, fn)
    if checkEventName(name) then self.handlers[name] = fn end
    return self
end

--- Show the group this frame, if active, and fire its prompts' handlers.
--- Call every frame the group should be visible.
function PoggyPromptGroup:handleEvents(...)
    if not self.active then return end
    self:show()
    for _, p in ipairs(self.prompts) do
        p:handleEvents(...)
    end
end

function PoggyPromptGroup:delete()
    local copy = {}
    for i, p in ipairs(self.prompts) do copy[i] = p end
    for _, p in ipairs(copy) do p:delete() end
    Registry.groups[self] = nil
end

-- ---------------------------------------------------------------------------
-- Optional automatic thread, for scripts that do not run their own frame loop.
-- ---------------------------------------------------------------------------

local threadRunning = false

function PoggyPrompts.startThread()
    if threadRunning then return end
    threadRunning = true
    CreateThread(function()
        while threadRunning do
            for g in pairs(Registry.groups) do g:handleEvents() end
            for p in pairs(Registry.prompts) do
                if not p.group then p:handleEvents() end
            end
            Wait(0)
        end
    end)
end

function PoggyPrompts.stopThread()
    threadRunning = false
end

-- ---------------------------------------------------------------------------
-- Cleanup. Prompts outlive a restarted resource unless they are deleted, and
-- an orphaned prompt keeps drawing with nothing left to answer it.
-- ---------------------------------------------------------------------------

AddEventHandler("onResourceStop", function(resource)
    if resource ~= GetCurrentResourceName() then return end
    threadRunning = false
    local prompts = {}
    for p in pairs(Registry.prompts) do prompts[#prompts + 1] = p end
    for _, p in ipairs(prompts) do
        pcall(PromptDelete, p.handle)
    end
    Registry.prompts, Registry.groups = {}, {}
end)
