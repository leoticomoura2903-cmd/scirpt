-- NexXit Painel RBLX v1.0
-- Made By NexXit
-- Professional Black & White Theme
-- AIMBOT: Mira na cabeça automática ao mirar

if getgenv().NexXitPainel and type(getgenv().NexXitPainel.destroy)=="function" then
    pcall(getgenv().NexXitPainel.destroy)
    task.wait(0.15)
end

-- services
local Svc = {
    Players    = game:GetService("Players"),
    RS         = game:GetService("ReplicatedStorage"),
    UIS        = game:GetService("UserInputService"),
    RunService = game:GetService("RunService"),
    InsertSvc  = game:GetService("InsertService"),
    TS         = game:GetService("TweenService"),
    Http       = game:GetService("HttpService"),
    VirtualInput = game:GetService("VirtualInputManager"),
}
local me = Svc.Players.LocalPlayer

-- config
local Cfg = {
    HIT_HASH    = "56d2123a3d",
    RELOAD_HASH = "2r232a424fh3a",
    AMMO_VALUE  = 13,
    MAX_DIST    = 300,
    FIRE_GAP          = 0.03,
    SHOT_DELAY        = 0.01,
    CYCLE_GAP         = 0.06,
    JITTER_AMT        = 0.02,
    HIT_RETRIES       = 2,
    MIN_SHOT_INTERVAL = 0.12,
    MAX_SAME_TARGET   = 2,
    ONLY_NEAREST    = true,
    MINECRAFT_SWORD = true,
    SKIP_FORCEFIELD = true,
    SKIP_TEAMMATES  = true,
    AUTO_RELOAD     = true,
    RELOAD_EVERY    = 8,
    RELOAD_DELAY    = 0.08,
    AIM_FOV    = 360,
    AIM_FOV_ON = true,
    LOS_X_SPREAD = 0.9,
    LOS_Y_SPREAD = 1.6,
    AFK_SPEED    = 1.2,
    ANTI_WALLBANG = true,
    AWB_EXTRA_DELAY = 0.15,
    AWB_JITTER = 0.04,
    AA_EMOTE_ID    = 121890511737627,
    EMOTE_AA_ON    = false,
    AA_EMOTE_SPEED = 1,
    BHOP_ON       = false,
    BHOP_MAX      = 35,
    BHOP_START    = 16,
    BHOP_ACCEL    = 1,
    BHOP_FRICTION = 0.85,
    BHOP_GAIN     = 0.8,
    BHOP_STRAFE   = 1.2,
    BHOP_VEL      = 25,
    BHOP_JUMP_DIST = 20,
    FP_ON = false,
    AA_MODE = 0, AA_SPEED = 22, AA_JITTER = 70, AA_SWAY = 35,
    ESP_ON  = false,
    ESP_MAX = 400,
    VM_ASSET_ID = 14074732713,
    VM_X = 2.7, VM_Y = -0.3, VM_Z = -6.3,
    VM_RX = 100, VM_RY = 250, VM_RZ = 0,
    VM_PIVOT_Y = 0.0,
    VM_SWING_X = 0, VM_SWING_Y = 0, VM_SWING_Z = -0.8,
    VM_SWING_DIR = -1,
    FP_SENS = 0.35,
    FP_PITCH_MIN = -85, FP_PITCH_MAX = 85,
    GUI_VISIBLE = true,
    CAM_NOCLIP = false,
    TEAM_COLOR = Color3.fromRGB(200, 200, 200),
    -- AIMBOT CONFIG
    AIMBOT_ON = false,
    AIMBOT_SMOOTH = 0.35,
    AIMBOT_FOV = 120,
    AIMBOT_VISUAL = true,
    AIMBOT_KEY = Enum.KeyCode.LeftAlt,
}

-- keybinds
local Binds = {
    KillAura   = Enum.KeyCode.Y,
    ESP        = Enum.KeyCode.N,
    Jitter     = Enum.KeyCode.G,
    Bhop       = Enum.KeyCode.B,
    FP         = Enum.KeyCode.F,
    Reload     = Enum.KeyCode.R,
    EmoteAA    = Enum.KeyCode.U,
    CamNoclip  = Enum.KeyCode.V,
    ToggleGUI  = Enum.KeyCode.BackSlash,
    AimBot     = Enum.KeyCode.LeftAlt,
}

-- state
local St = {
    running = false,
    combatToken = 0,
    lastShotAt = 0,
    lastShotPlr = nil,
    sameTargetCount = 0,
    shotCounter = 0,
    aaYaw = 0,
    currentSpeed = 0,
    wasGrounded = true,
    swingOffset = 0,
    swingToken = 0,
    fpYaw = 0, fpPitch = 0,
    fpOrigCamType = nil,
    fpOrigMouseBehavior = nil,
    fpToggling = false,
    vmLoading = false,
    resolvedEmoteAnimId = nil,
    emoteResolving = false,
    aaAnimateWasDisabled = false,
    aaEmoteChar = nil,
    bhopSmoothVel = 0,
    camLastDist = 12,
    rebinding = nil,
    glockLiverHidden = false,
    aimbotTarget = nil,
    aimbotDrawings = {},
    aimbotActive = false,
}

-- refs
local Ref = {
    Event = nil,
    connections = {},
    aaConn = nil,
    bhopConn = nil,
    acBypassConn = nil,
    watchdogConn = nil,
    fpCamConn = nil,
    vpConn = nil,
    espConn = nil,
    emoteAAConn = nil,
    camNoclipConn = nil,
    rebindConn = nil,
    aaEmoteTrack = nil,
    aaEmoteAnim = nil,
    aaEmoteWatch = nil,
    vpFrame = nil,
    vpCam = nil,
    vpModel = nil,
    vmHandleOffset = nil,
    gui = nil,
    mainWindow = nil,
    espDraws = {},
    playerFFState = {},
    ffTriggerQueue = {},
    deadCache = {},
    healthConns = {},
    teammates = {},
    teamConns = {},
    glockLiverCache = nil,
    aimbotConn = nil,
}

-- gui refs
local GUI = {
    toggles = {},
    bindLabels = {},
    statusLabel = nil,
    logLabel = nil,
    activeTab = "Combat",
    tabBtns = {},
    tabPages = {},
}

-- BLACK & WHITE THEME
local Theme = {
    bg         = Color3.fromRGB(18, 18, 18),
    bgDark     = Color3.fromRGB(10, 10, 10),
    bgCard     = Color3.fromRGB(25, 25, 25),
    accent     = Color3.fromRGB(255, 255, 255),
    accentDim  = Color3.fromRGB(180, 180, 180),
    white      = Color3.fromRGB(255, 255, 255),
    red        = Color3.fromRGB(255, 50, 50),
    textMain   = Color3.fromRGB(230, 230, 230),
    textDim    = Color3.fromRGB(160, 160, 160),
    textMuted  = Color3.fromRGB(80, 80, 80),
    border     = Color3.fromRGB(35, 35, 35),
    toggleOff  = Color3.fromRGB(45, 45, 45),
    toggleOn   = Color3.fromRGB(255, 255, 255),
    sliderBg   = Color3.fromRGB(30, 30, 30),
    sliderFill = Color3.fromRGB(255, 255, 255),
    tabActive  = Color3.fromRGB(255, 255, 255),
    tabIdle    = Color3.fromRGB(18, 18, 18),
}

local AA_NAMES = {[0]="OFF",[1]="SPIN",[2]="JITTER",[3]="INVERT",[4]="RANDOM",[5]="SWAY"}
local AA_COUNT = 6

-- ============ UTILITIES ============
function getRoot(char) return char and (char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("Torso") or char:FindFirstChild("UpperTorso")) end
function getHum(char) return char and char:FindFirstChildOfClass("Humanoid") end
function myChar() return me.Character end
function hasFF(char) return char and char:FindFirstChildOfClass("ForceField") ~= nil end
function holdingTool() local c = myChar(); return c and c:FindFirstChildOfClass("Tool") ~= nil end
function isAFK(root) if not root then return false end; local v = root.AssemblyLinearVelocity; return Vector3.new(v.X,0,v.Z).Magnitude <= Cfg.AFK_SPEED end
function getPart(char) return char and (char:FindFirstChild("Head") or char:FindFirstChild("UpperTorso") or char:FindFirstChild("Torso") or char:FindFirstChild("HumanoidRootPart")) end
function getHitParts(char) if not char then return {} end; local out, seen = {}, {}; for _, n in ipairs({"Head","UpperTorso","Torso","HumanoidRootPart","LowerTorso"}) do local p = char:FindFirstChild(n); if p and p:IsA("BasePart") and not seen[p] then seen[p]=true; out[#out+1]=p end end; if #out==0 then local p=getPart(char); if p then out[1]=p end end; return out end
function genUID() return "{"..string.upper(string.gsub("xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx","[xy]",function(c) local v=(c=="x") and math.random(0,15) or math.random(8,11); return string.format("%x",v) end)).."}" end
function keyName(kc) if not kc then return "?" end; local n = tostring(kc):gsub("Enum.KeyCode.", ""); if n == "BackSlash" then return "\\" end; if n == "Space" then return "SPACE" end; if n == "LeftShift" then return "LSHIFT" end; if n == "RightShift" then return "RSHIFT" end; if n == "LeftControl" then return "LCTRL" end; if n == "RightControl" then return "RCTRL" end; if n == "LeftAlt" then return "LALT" end; if n == "RightAlt" then return "RALT" end; if #n == 1 then return n end; return n end

function myOrigin()
    local char = myChar(); if not char then return nil end
    local tool = char:FindFirstChildOfClass("Tool")
    if tool then
        for _, n in ipairs({"Muzzle","Fire","Barrel","Handle"}) do
            local p = tool:FindFirstChild(n,true)
            if p and p:IsA("BasePart") then return p.Position end
        end
        local p = tool:FindFirstChildWhichIsA("BasePart"); if p then return p.Position end
    end
    local h = char:FindFirstChild("Head"); if h then return h.Position end
    local r = getRoot(char); return r and r.Position or nil
end

function findGlockLiver()
    if Ref.glockLiverCache and Ref.glockLiverCache.Parent then return Ref.glockLiverCache end
    Ref.glockLiverCache = nil
    local char = myChar()
    if char then
        local gl = char:FindFirstChild("GlockLiver") or char:FindFirstChild("GlockLiver", true)
        if gl then Ref.glockLiverCache = gl; return gl end
    end
    local wsChar = workspace:FindFirstChild(me.Name)
    if wsChar and wsChar ~= char then
        local gl = wsChar:FindFirstChild("GlockLiver", true)
        if gl then Ref.glockLiverCache = gl; return gl end
    end
    return nil
end

function setGlockLiverVisible(visible)
    local gl = findGlockLiver()
    if not gl then if not visible then St.glockLiverHidden = true end; return end
    local mod = visible and 0 or 1
    local function apply(o) if o:IsA("BasePart") or o:IsA("Decal") or o:IsA("Texture") then pcall(function() o.LocalTransparencyModifier = mod end) end end
    apply(gl)
    if gl:IsA("Model") or gl:IsA("Folder") then for _, desc in ipairs(gl:GetDescendants()) do apply(desc) end end
    St.glockLiverHidden = not visible
end

function setLocalTransp(char,t)
    if not char then return end
    for _,p in ipairs(char:GetDescendants()) do
        if p:IsA("BasePart") or p:IsA("Decal") or p:IsA("Texture") then
            pcall(function() p.LocalTransparencyModifier=t end)
        end
    end
end

function getgenv() return _G end

-- ============ TEAMMATES ============
do
    local function isTeammate(plr)
        if not plr then return false end
        return Ref.teammates[plr.UserId] == true
    end
    getgenv()._tsIsTeammate = isTeammate

    local function detectMySide(data)
        for _, entry in pairs(data.A or {}) do if entry.userId == me.UserId then return "A" end end
        for _, entry in pairs(data.B or {}) do if entry.userId == me.UserId then return "B" end end
        return "A"
    end

    local function rebuildTeammates(data)
        Ref.teammates = {}
        local mySide = detectMySide(data)
        local myTeam = (mySide == "A") and data.A or data.B
        if not myTeam then return end
        for _, entry in pairs(myTeam) do
            if entry.userId and entry.userId ~= me.UserId then
                Ref.teammates[entry.userId] = true
            end
        end
    end

    local function clearTeammates() Ref.teammates = {} end

    local function hookArenaEvents()
        local ok, arenaFolder = pcall(function() return Svc.RS:WaitForChild("Remotes", 5):WaitForChild("Arena", 5) end)
        if not ok or not arenaFolder then task.delay(3, hookArenaEvents); return end
        local syncOK, arenaSync = pcall(function() return arenaFolder:WaitForChild("ArenaSync", 5) end)
        local clearOK, arenaClear = pcall(function() return arenaFolder:WaitForChild("ArenaClear", 5) end)
        if syncOK and arenaSync then
            Ref.teamConns[#Ref.teamConns+1] = arenaSync.OnClientEvent:Connect(function(data)
                if type(data) ~= "table" then return end
                rebuildTeammates(data)
            end)
        end
        if clearOK and arenaClear then
            Ref.teamConns[#Ref.teamConns+1] = arenaClear.OnClientEvent:Connect(function() clearTeammates() end)
        end
    end
    hookArenaEvents()
end

-- ============ IS ALIVE ============
do
    local function isDead_gameSpecific(char, hum)
        local deadAttr = false
        pcall(function() deadAttr = hum:GetAttribute("Dead")==true end)
        if deadAttr then return true end
        local hrp = char:FindFirstChild("HumanoidRootPart")
        if hrp then local sz=hrp.Size; if sz.X<0.5 and sz.Y<0.5 then return true end end
        local ws=0; pcall(function() ws=hum.WalkSpeed end)
        if ws==0 then local ok,state=pcall(function() return hum:GetState() end); if ok and state==Enum.HumanoidStateType.Physics then return true end end
        return false
    end

    function isAlive(char, hum)
        if not char or not char.Parent then return false end
        if not char:IsDescendantOf(workspace) then return false end
        hum = hum or getHum(char)
        if not hum or not hum.Parent then return false end
        for _, plr in ipairs(Svc.Players:GetPlayers()) do
            if plr.Character==char then if Ref.deadCache[plr] then return false end; break end
        end
        if isDead_gameSpecific(char,hum) then return false end
        local maxHp,hp = hum.MaxHealth,hum.Health
        if maxHp<=0 or hp<=0 then return false end
        local ok,state = pcall(function() return hum:GetState() end)
        if ok and state==Enum.HumanoidStateType.Dead then return false end
        local av = char:FindFirstChild("IsAlive")
        if av and av:IsA("BoolValue") and av.Value==false then return false end
        local root = getRoot(char)
        if root and root.Position.Y<-200 then return false end
        return true
    end
end

-- ============ EVENT FINDER ============
do
    local function getCombatEvent()
        local ok, res
        ok, res = pcall(function() return Svc.RS.ZexisShared.Packages._Index["sleitnick_knit@1.4.7"].knit.Services.CombatService.RE.SendRequest end)
        if ok and res and typeof(res) == "Instance" then return res end
        ok, res = pcall(function()
            local m = Svc.RS:WaitForChild("ZexisShared",5):WaitForChild("Packages",5):WaitForChild("_Index",5):WaitForChild("sleitnick_knit@1.4.7",5):WaitForChild("knit",5)
            if m and m:IsA("ModuleScript") then return require(m).Services.CombatService.RE.SendRequest end
        end)
        if ok and res and typeof(res) == "Instance" then return res end
        ok, res = pcall(function()
            for _, v in ipairs(Svc.RS:GetDescendants()) do
                if v:IsA("RemoteEvent") and v.Name == "SendRequest" then return v end
            end
        end)
        if ok and res then return res end
        return nil
    end

    function loadEvent()
        Ref.Event = getCombatEvent()
        if Ref.Event then print("[NexXit] event: "..Ref.Event:GetFullName()) else task.delay(1.5, loadEvent) end
    end
    loadEvent()
end

-- ============ DEAD CACHE ============
do
    local function unwatchHealth(plr)
        if not Ref.healthConns[plr] then return end
        local c = Ref.healthConns[plr]
        if type(c)=="table" then for _,conn in ipairs(c) do pcall(function() conn:Disconnect() end) end
        else pcall(function() c:Disconnect() end) end
        Ref.healthConns[plr]=nil
    end
    getgenv()._tsUnwatchHealth = unwatchHealth

    function watchHealth(plr)
        if plr==me then return end; unwatchHealth(plr)
        local function attach(char)
            if not char then return end
            local hum = char:FindFirstChildOfClass("Humanoid")
            if not hum then pcall(function() char:WaitForChild("Humanoid",3) end); hum=char:FindFirstChildOfClass("Humanoid") end
            if not hum then return end
            Ref.deadCache[plr]=nil; local conns={}
            conns[#conns+1]=hum.HealthChanged:Connect(function(hp) if hp<=0 then Ref.deadCache[plr]=true end end)
            conns[#conns+1]=hum.Died:Connect(function() Ref.deadCache[plr]=true end)
            conns[#conns+1]=hum:GetAttributeChangedSignal("Dead"):Connect(function()
                local v=false; pcall(function() v=hum:GetAttribute("Dead") end)
                if v==true then Ref.deadCache[plr]=true elseif v==false or v==nil then Ref.deadCache[plr]=nil end
            end)
            local hrp = char:FindFirstChild("HumanoidRootPart")
            if hrp then
                conns[#conns+1]=hrp:GetPropertyChangedSignal("Size"):Connect(function()
                    local sz=hrp.Size
                    if sz.X<0.5 and sz.Y<0.5 then Ref.deadCache[plr]=true
                    else local da=false; pcall(function() da=hum:GetAttribute("Dead")==true end)
                        if not da then Ref.deadCache[plr]=nil end end
                end)
            end
            Ref.healthConns[plr]=conns
        end
        if plr.Character then task.spawn(attach,plr.Character) end
        plr.CharacterAdded:Connect(function(char) Ref.deadCache[plr]=nil; task.spawn(attach,char) end)
        plr.CharacterRemoving:Connect(function() Ref.deadCache[plr]=nil end)
    end
end

-- ============ CONFIG SAVE/LOAD ============
do
    local CFG_DIR = "NexXitPainel"
    local CFG_PATH = "NexXitPainel/Config.json"

    local saveKeysBool = {
        "ANTI_WALLBANG","AIM_FOV_ON","ONLY_NEAREST","SKIP_FORCEFIELD",
        "SKIP_TEAMMATES","AUTO_RELOAD","MINECRAFT_SWORD","CAM_NOCLIP",
        "AIMBOT_ON","AIMBOT_VISUAL",
    }
    local saveKeysNum = {
        "AIM_FOV","MAX_DIST","AMMO_VALUE","SHOT_DELAY","AWB_EXTRA_DELAY",
        "RELOAD_EVERY","ESP_MAX","FP_SENS","AA_MODE","AA_SPEED","AA_JITTER",
        "AA_SWAY","AA_EMOTE_SPEED","BHOP_MAX","BHOP_ACCEL","BHOP_STRAFE",
        "BHOP_JUMP_DIST","VM_X","VM_Y","VM_Z","VM_RX","VM_RY","VM_RZ",
        "VM_PIVOT_Y","VM_SWING_X","VM_SWING_Y","VM_SWING_Z","VM_SWING_DIR",
        "FIRE_GAP","CYCLE_GAP","JITTER_AMT","MIN_SHOT_INTERVAL",
        "MAX_SAME_TARGET","RELOAD_DELAY","AWB_JITTER","BHOP_START",
        "BHOP_FRICTION","BHOP_GAIN","BHOP_VEL","FP_PITCH_MIN","FP_PITCH_MAX",
        "LOS_X_SPREAD","LOS_Y_SPREAD","AFK_SPEED","AIMBOT_SMOOTH","AIMBOT_FOV",
    }
    local saveKeysStr = {
        "HIT_HASH","RELOAD_HASH",
    }

    local function validateVal(key, val)
        for _, k in ipairs(saveKeysBool) do if k == key then if type(val) == "boolean" then return val end; if type(val) == "number" then return val ~= 0 end; return nil end end
        for _, k in ipairs(saveKeysNum) do if k == key then local n = tonumber(val); if n then return n end; return nil end end
        for _, k in ipairs(saveKeysStr) do if k == key then if type(val) == "string" and #val > 0 then return val end; return nil end end
        return nil
    end

    function saveConfig()
        local ok, err = pcall(function()
            if not isfolder(CFG_DIR) then makefolder(CFG_DIR) end
            local t = {}
            for _, k in ipairs(saveKeysBool) do t[k] = Cfg[k] and true or false end
            for _, k in ipairs(saveKeysNum) do t[k] = tonumber(Cfg[k]) or 0 end
            for _, k in ipairs(saveKeysStr) do t[k] = tostring(Cfg[k] or "") end
            local bindsData = {}
            for action, kc in pairs(Binds) do bindsData[action] = tostring(kc) end
            t["_binds"] = bindsData
            t["_ver"] = 2
            writefile(CFG_PATH, Svc.Http:JSONEncode(t))
            print("[NexXit] config saved ✓")
        end)
        if not ok then print("[NexXit] config save FAILED: "..tostring(err)) end
    end

    function loadConfig()
        local ok, err = pcall(function()
            if not isfile(CFG_PATH) then print("[NexXit] no config file, using defaults"); return end
            local raw = readfile(CFG_PATH)
            if not raw or #raw == 0 then print("[NexXit] config file empty, skipping"); return end
            local data = Svc.Http:JSONDecode(raw)
            if type(data) ~= "table" then print("[NexXit] config parse error, skipping"); return end
            local loaded = 0
            for _, k in ipairs(saveKeysBool) do local v = validateVal(k, data[k]); if v ~= nil then Cfg[k] = v; loaded = loaded + 1 end end
            for _, k in ipairs(saveKeysNum) do local v = validateVal(k, data[k]); if v ~= nil then Cfg[k] = v; loaded = loaded + 1 end end
            for _, k in ipairs(saveKeysStr) do local v = validateVal(k, data[k]); if v ~= nil then Cfg[k] = v; loaded = loaded + 1 end end
            if type(data._binds) == "table" then
                for action, kcStr in pairs(data._binds) do
                    if Binds[action] ~= nil and type(kcStr) == "string" then
                        local keyPart = kcStr:gsub("Enum.KeyCode.", "")
                        local ok2, kc = pcall(function() return Enum.KeyCode[keyPart] end)
                        if ok2 and kc then Binds[action] = kc; loaded = loaded + 1 end
                    end
                end
            end
            print("[NexXit] config loaded ✓ ("..loaded.." keys)")
        end)
        if not ok then print("[NexXit] config load FAILED: "..tostring(err)) end
    end

    loadConfig()
end

-- ============ FOV CHECK WITH CIRCLE DRAWING ============
local fovCircle = nil

function createFOVCircle()
    if fovCircle then
        pcall(function() fovCircle.Visible = false end)
        pcall(function() fovCircle:Remove() end)
        fovCircle = nil
    end
    if not Cfg.AIMBOT_VISUAL or not Cfg.AIMBOT_ON then return end
    local ok, circle = pcall(function()
        local c = Drawing.new("Circle")
        c.Radius = Cfg.AIMBOT_FOV
        c.Thickness = 1
        c.Color = Color3.fromRGB(255, 255, 255)
        c.Transparency = 0.6
        c.Filled = false
        c.Visible = true
        c.NumSides = 32
        return c
    end)
    if ok and circle then
        fovCircle = circle
        updateFOVCirclePosition()
    end
end

function updateFOVCirclePosition()
    if not fovCircle then return end
    local mousePos = Svc.UIS:GetMouseLocation()
    fovCircle.Position = Vector2.new(mousePos.X, mousePos.Y)
end

function destroyFOVCircle()
    if fovCircle then
        pcall(function() fovCircle.Visible = false end)
        pcall(function() fovCircle:Remove() end)
        fovCircle = nil
    end
end

function toggleFOVCircle()
    if Cfg.AIMBOT_ON and Cfg.AIMBOT_VISUAL then
        createFOVCircle()
    else
        destroyFOVCircle()
    end
end

-- ============ AIMBOT SYSTEM ============
function getClosestPlayerToCursor()
    local cam = workspace.CurrentCamera
    if not cam then return nil end
    local mousePos = Svc.UIS:GetMouseLocation()
    local bestDist = Cfg.AIMBOT_FOV
    local bestTarget = nil
    local bestPos = nil
    local bestPart = nil
    
    for _, plr in ipairs(Svc.Players:GetPlayers()) do
        if plr == me then continue end
        if Cfg.SKIP_TEAMMATES and getgenv()._tsIsTeammate(plr) then continue end
        local char = plr.Character
        if not char or not char.Parent then continue end
        local hum = getHum(char)
        if not isAlive(char, hum) then continue end
        if Cfg.SKIP_FORCEFIELD and hasFF(char) then continue end
        local root = getRoot(char)
        if not root then continue end
        local origin = myOrigin()
        if not origin then continue end
        if (origin - root.Position).Magnitude > Cfg.MAX_DIST then continue end
        
        -- Pega a posição da cabeça
        local head = char:FindFirstChild("Head")
        local targetPos = head and head.Position or root.Position + Vector3.new(0, 1.5, 0)
        
        -- Converte para posição na tela
        local screenPos, onScreen = cam:WorldToViewportPoint(targetPos)
        if not onScreen then continue end
        
        -- Calcula distância do cursor
        local dist = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude
        
        -- Verifica se está dentro do FOV
        if dist <= bestDist then
            -- Verifica linha de visão
            local origin2 = cam.CFrame.Position
            local params = RaycastParams.new()
            params.FilterType = Enum.RaycastFilterType.Exclude
            local c = myChar()
            params.FilterDescendantsInstances = c and {c} or {}
            params.IgnoreWater = true
            
            local hit = workspace:Raycast(origin2, (targetPos - origin2).Unit * (origin2 - targetPos).Magnitude, params)
            if not hit or (hit.Instance and hit.Instance:IsDescendantOf(char)) then
                bestDist = dist
                bestTarget = plr
                bestPos = targetPos
                bestPart = head or root
            end
        end
    end
    
    return bestTarget, bestPos, bestPart
end

function startAimbot()
    if Ref.aimbotConn then pcall(function() Ref.aimbotConn:Disconnect() end); Ref.aimbotConn = nil end
    
    Cfg.AIMBOT_ON = true
    St.aimbotActive = true
    createFOVCircle()
    
    Ref.aimbotConn = Svc.RunService.RenderStepped:Connect(function()
        if not Cfg.AIMBOT_ON then return end
        if not Svc.UIS:IsKeyDown(Cfg.AIMBOT_KEY) then
            St.aimbotTarget = nil
            return
        end
        
        local target, targetPos, targetPart = getClosestPlayerToCursor()
        if not target or not targetPos then
            St.aimbotTarget = nil
            return
        end
        
        local cam = workspace.CurrentCamera
        if not cam then return end
        
        -- Suaviza a mira
        local currentCF = cam.CFrame
        local targetCF = CFrame.lookAt(cam.CFrame.Position, targetPos)
        
        -- Interpolação suave
        local smooth = Cfg.AIMBOT_SMOOTH
        local newCF = currentCF:Lerp(targetCF, smooth)
        
        -- Aplica a nova rotação
        cam.CFrame = newCF
        
        -- Atualiza o alvo
        St.aimbotTarget = target
        
        -- Atualiza o círculo FOV
        updateFOVCirclePosition()
    end)
    
    print("[NexXit] AimBot ON")
end

function stopAimbot()
    Cfg.AIMBOT_ON = false
    St.aimbotActive = false
    St.aimbotTarget = nil
    destroyFOVCircle()
    if Ref.aimbotConn then
        pcall(function() Ref.aimbotConn:Disconnect() end)
        Ref.aimbotConn = nil
    end
    print("[NexXit] AimBot OFF")
end

function toggleAimbot()
    if Cfg.AIMBOT_ON then
        stopAimbot()
    else
        startAimbot()
    end
    if GUI.toggles["AimBot"] then GUI.toggles["AimBot"](Cfg.AIMBOT_ON) end
end

-- ============ FOV / LOS / AIM / ANTIWALLBANG ============
function inFOV(worldPos)
    if not Cfg.AIM_FOV_ON or Cfg.AIM_FOV>=360 then return true end
    local cam = workspace.CurrentCamera; if not cam then return true end
    local lp = cam.CFrame:PointToObjectSpace(worldPos)
    if lp.Z>-0.1 then return false end
    local half = Cfg.AIM_FOV*0.5
    return math.abs(math.deg(math.atan2(lp.X,-lp.Z)))<=half and math.abs(math.deg(math.atan2(lp.Y,-lp.Z)))<=half
end

local function isPathClear(origin, targetPos, targetChar)
    local dir = targetPos - origin
    local dist = dir.Magnitude
    if dist < 0.5 then return true end
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Exclude
    local c = myChar()
    local filter = c and {c} or {}
    if targetChar then filter[#filter+1] = targetChar end
    params.FilterDescendantsInstances = filter
    params.IgnoreWater = true
    local hit = workspace:Raycast(origin, dir.Unit * (dist - 0.3), params)
    if not hit then return true end
    return false
end
getgenv()._tsPathClear = isPathClear

local function checkWallbang(origin, targetPos, targetChar)
    if not Cfg.ANTI_WALLBANG then return false, 0 end
    local dir = targetPos - origin
    local dist = dir.Magnitude
    if dist < 1 then return false, 0 end
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Exclude
    local c = myChar()
    params.FilterDescendantsInstances = c and {c} or {}
    params.IgnoreWater = true
    local hit = workspace:Raycast(origin, dir.Unit * dist, params)
    if not hit then return false, 0 end
    if targetChar and hit.Instance and hit.Instance:IsDescendantOf(targetChar) then return false, 0 end
    local wallPoint = hit.Position
    local remaining = (targetPos - wallPoint).Magnitude
    local params2 = RaycastParams.new()
    params2.FilterType = Enum.RaycastFilterType.Exclude
    params2.FilterDescendantsInstances = c and {c} or {}
    params2.IgnoreWater = true
    local hit2 = workspace:Raycast(wallPoint + dir.Unit * 0.1, dir.Unit * remaining, params2)
    local thickness = remaining
    if hit2 then thickness = (hit2.Position - wallPoint).Magnitude end
    return true, thickness
end
getgenv()._tsCheckWallbang = checkWallbang

local function rayHit(origin, targetPos, targetChar)
    local dir = targetPos - origin; local dist = dir.Magnitude
    if dist < 0.4 then return true, targetPos end
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Exclude
    local c = myChar(); params.FilterDescendantsInstances = c and {c} or {}
    params.IgnoreWater = true
    local hit = workspace:Raycast(origin, dir.Unit * dist, params)
    if not hit then return true, targetPos end
    if targetChar and hit.Instance and hit.Instance:IsDescendantOf(targetChar) then return true, hit.Position end
    return false, nil
end

function bestVisibleAim(origin, char)
    if not origin or not char then return nil,nil,nil end
    local candidates={}
    for _,p in ipairs(getHitParts(char)) do
        local cf=p.CFrame
        candidates[#candidates+1]={part=p, pos=p.Position}
        if p.Name=="Head" then
            local hs=p.Size*0.45
            for _,o in ipairs({
                Vector3.new(0,hs.Y,0), Vector3.new(0,-hs.Y,0),
                Vector3.new(hs.X,0,0), Vector3.new(-hs.X,0,0),
                Vector3.new(0,0,-hs.Z), Vector3.new(0,0,hs.Z),
                Vector3.new(hs.X,hs.Y,0), Vector3.new(-hs.X,hs.Y,0),
                Vector3.new(hs.X,-hs.Y,0), Vector3.new(-hs.X,-hs.Y,0),
            }) do candidates[#candidates+1]={part=p, pos=cf:PointToWorldSpace(o)} end
        else
            local rx,ry=Cfg.LOS_X_SPREAD,Cfg.LOS_Y_SPREAD
            for _,o in ipairs({
                Vector3.new(0,ry*0.5,0), Vector3.new(rx*0.5,0,0), Vector3.new(-rx*0.5,0,0),
                Vector3.new(0,ry,0), Vector3.new(0,-ry*0.3,0),
                Vector3.new(rx*0.4,ry*0.4,0), Vector3.new(-rx*0.4,ry*0.4,0),
            }) do candidates[#candidates+1]={part=p, pos=cf:PointToWorldSpace(o)} end
        end
    end
    table.sort(candidates, function(a,b)
        local aH=a.part.Name=="Head"; local bH=b.part.Name=="Head"
        if aH~=bH then return aH end; return false
    end)
    for _,c in ipairs(candidates) do
        if inFOV(c.pos) then
            local ok,hp=rayHit(origin,c.pos,c.part.Parent)
            if ok then return c.part, hp or c.pos, c.pos end
        end
    end
    return nil,nil,nil
end

function valid(plr)
    if plr==me then return false end
    if Cfg.SKIP_TEAMMATES and getgenv()._tsIsTeammate(plr) then return false end
    local char=plr.Character; if not char or not char.Parent then return false end
    local hum=getHum(char); if not isAlive(char,hum) then return false end
    if Cfg.SKIP_FORCEFIELD and hasFF(char) then return false end
    local root=getRoot(char); if not root then return false end
    local origin=myOrigin(); if not origin then return false end
    if (origin-root.Position).Magnitude>Cfg.MAX_DIST then return false end
    local head=char:FindFirstChild("Head")
    if not inFOV(root.Position) and not (head and inFOV(head.Position)) then return false end
    local part,hitPos,aimPos=bestVisibleAim(origin,char); if not part then return false end
    if not isPathClear(origin, hitPos, char) then return false end
    return true,char,part,root,(origin-root.Position).Magnitude,origin,hitPos,aimPos
end

-- ============ KILLAURA / COMBAT ============
function fireReload(reason)
    if not Ref.Event then loadEvent(); return false end
    local ok=pcall(function() Ref.Event:FireServer(Cfg.RELOAD_HASH,{}) end)
    if ok then St.shotCounter=0; print("[NexXit] reload | "..tostring(reason)) end; return ok
end

local function maybeAutoReload()
    if Cfg.AUTO_RELOAD and St.shotCounter>=Cfg.RELOAD_EVERY then task.wait(Cfg.RELOAD_DELAY); fireReload("auto") end
end

local function fireEntry(entry,reason)
    if not Ref.Event then loadEvent(); return false end
    if Cfg.MINECRAFT_SWORD and not holdingTool() then return false end
    local now=tick()
    if now-St.lastShotAt<Cfg.MIN_SHOT_INTERVAL then task.wait(Cfg.MIN_SHOT_INTERVAL-(now-St.lastShotAt)) end
    local origin=myOrigin(); if not origin then return false end
    if not entry.char or not entry.char.Parent then return false end
    if not isAlive(entry.char) then return false end
    if Cfg.SKIP_TEAMMATES and getgenv()._tsIsTeammate(entry.plr) then return false end
    local part,hitPos,aimPos=bestVisibleAim(origin,entry.char)
    if not part or not hitPos then return false end
    if not inFOV(aimPos or hitPos) then return false end
    if not getgenv()._tsPathClear(origin, hitPos, entry.char) then return false end

    local isWB = getgenv()._tsCheckWallbang(origin, hitPos, entry.char)
    if isWB and Cfg.ANTI_WALLBANG then
        task.wait(Cfg.AWB_EXTRA_DELAY + math.random()*Cfg.AWB_JITTER)
        if not entry.char or not entry.char.Parent or not isAlive(entry.char) then return false end
        origin = myOrigin() or origin
        part, hitPos, aimPos = bestVisibleAim(origin, entry.char)
        if not part or not hitPos then return false end
        if not getgenv()._tsPathClear(origin, hitPos, entry.char) then return false end
    end

    local uid=genUID()
    local ok1=pcall(function() Ref.Event:FireServer("Fire",{{Direction=hitPos,Origin=origin,UID=uid}},Cfg.AMMO_VALUE) end)
    if not ok1 then return false end
    if Cfg.FIRE_GAP>0 then task.wait(Cfg.FIRE_GAP) end
    origin=myOrigin() or origin
    if not isAlive(entry.char) then return false end
    if not getgenv()._tsPathClear(origin, hitPos, entry.char) then return false end

    local sent=false
    for _=1,Cfg.HIT_RETRIES do
        if not entry.char or not entry.char.Parent then break end
        if not isAlive(entry.char) then break end
        origin=myOrigin() or origin
        local p,hp=bestVisibleAim(origin,entry.char); if not p or not hp then break end
        if not getgenv()._tsPathClear(origin, hp, entry.char) then break end
        local ok2=pcall(function() Ref.Event:FireServer(Cfg.HIT_HASH,entry.char,p,uid,hp) end)
        if ok2 then sent=true; break end; task.wait(0.02)
    end
    if not sent then return false end
    St.lastShotAt=tick(); St.shotCounter=St.shotCounter+1
    if Cfg.MINECRAFT_SWORD then triggerSwing() end
    if GUI.statusLabel then GUI.statusLabel.Text=("%d shots | %s"):format(St.shotCounter,reason) end
    if Cfg.SHOT_DELAY>0 then task.wait(Cfg.SHOT_DELAY+math.random()*Cfg.JITTER_AMT) end
    maybeAutoReload(); return true
end

local function pickTargets()
    local list={}
    for _,plr in ipairs(Svc.Players:GetPlayers()) do
        local ok,char,part,root,dist,origin,hitPos,aimPos=valid(plr)
        if ok then list[#list+1]={plr=plr,char=char,part=part,root=root,dist=dist,origin=origin,hitPos=hitPos,aimPos=aimPos,afk=isAFK(root)} end
    end
    table.sort(list,function(a,b) if a.afk~=b.afk then return a.afk end; return a.dist<b.dist end)
    if Cfg.ONLY_NEAREST and list[1] then return {list[1]} end; return list
end

local function combatLoop(token)
    while St.running and St.combatToken==token do
        pcall(function()
            if not Ref.Event or not Ref.Event.Parent then loadEvent() end
            while #Ref.ffTriggerQueue>0 and St.running and St.combatToken==token do
                local e=table.remove(Ref.ffTriggerQueue,1)
                if e and e.char and e.char.Parent and isAlive(e.char) and not hasFF(e.char) then fireEntry(e,"FF") end
            end
            if Cfg.MINECRAFT_SWORD and not holdingTool() then task.wait(0.15); return end
            local targets=pickTargets()
            if #targets==0 then St.lastShotPlr=nil; St.sameTargetCount=0; task.wait(0.1); return end
            for i=1,#targets do
                if not St.running or St.combatToken~=token then break end
                local e=targets[i]
                if not (e.char and e.char.Parent and isAlive(e.char)) then continue end
                if Cfg.SKIP_FORCEFIELD and hasFF(e.char) then continue end
                if Cfg.SKIP_TEAMMATES and getgenv()._tsIsTeammate(e.plr) then continue end
                if e.plr==St.lastShotPlr then
                    St.sameTargetCount=St.sameTargetCount+1
                    if St.sameTargetCount>Cfg.MAX_SAME_TARGET then if #targets>1 then continue end; St.sameTargetCount=0 end
                else St.lastShotPlr=e.plr; St.sameTargetCount=1 end
                fireEntry(e,e.afk and "AFK" or "KA")
            end
        end)
        task.wait(Cfg.CYCLE_GAP)
    end
end

function setRunning(on)
    if on then
        St.combatToken=St.combatToken+1; St.running=true; St.lastShotPlr=nil; St.sameTargetCount=0
        if GUI.statusLabel then GUI.statusLabel.Text="ACTIVE"; GUI.statusLabel.TextColor3=Theme.white end
        task.spawn(combatLoop,St.combatToken)
    else
        St.running=false; St.combatToken=St.combatToken+1; Ref.ffTriggerQueue={}; St.lastShotPlr=nil; St.sameTargetCount=0
        if GUI.statusLabel then GUI.statusLabel.Text="IDLE"; GUI.statusLabel.TextColor3=Theme.textDim end
    end
    if GUI.toggles["KillAura"] then GUI.toggles["KillAura"](on) end
end

-- ============ VIEWMODEL / FP CAMERA ============
local SWING_DELTA = 60
local SWING_DURATION = 0.4

function triggerSwing()
    St.swingToken=St.swingToken+1; local token=St.swingToken
    task.spawn(function()
        local t=0
        while token==St.swingToken and t<SWING_DURATION do
            local dt=task.wait(); t=math.min(t+dt,SWING_DURATION)
            St.swingOffset=SWING_DELTA*math.sin((t/SWING_DURATION)*math.pi)
        end
        if token==St.swingToken then St.swingOffset=0 end
    end)
end

local function vmOffset()
    local base = CFrame.new(Cfg.VM_X,Cfg.VM_Y,Cfg.VM_Z) * CFrame.Angles(math.rad(Cfg.VM_RX), math.rad(Cfg.VM_RY), math.rad(Cfg.VM_RZ))
    local H
    if Cfg.VM_SWING_X~=0 or Cfg.VM_SWING_Y~=0 or Cfg.VM_SWING_Z~=0 then
        H = Vector3.new(Cfg.VM_SWING_X, Cfg.VM_SWING_Y, Cfg.VM_SWING_Z)
    else
        H = Ref.vmHandleOffset or Vector3.new()
    end
    local swing = CFrame.Angles(0, math.rad(St.swingOffset), 0)
    if H.Magnitude < 0.001 then return base * swing end
    return base * CFrame.new(H) * swing * CFrame.new(-H)
end

function initFPAngles()
    local cam = workspace.CurrentCamera
    if cam then
        local look = cam.CFrame.LookVector
        St.fpYaw = math.deg(math.atan2(-look.X,-look.Z))
        St.fpPitch = math.clamp(math.deg(math.asin(math.clamp(look.Y,-1,1))), Cfg.FP_PITCH_MIN, Cfg.FP_PITCH_MAX)
    else
        local char=myChar(); local root=getRoot(char)
        if root then local look=root.CFrame.LookVector; St.fpYaw=math.deg(math.atan2(-look.X,-look.Z)) end
        St.fpPitch=0
    end
end

function startFPCamera()
    if Ref.fpCamConn then pcall(function() Ref.fpCamConn:Disconnect() end); Ref.fpCamConn = nil end
    local cam = workspace.CurrentCamera
    if not cam then return end
    if cam.CameraType ~= Enum.CameraType.Scriptable then St.fpOrigCamType = cam.CameraType end
    St.fpOrigMouseBehavior = Svc.UIS.MouseBehavior
    pcall(function() cam.CameraType = Enum.CameraType.Scriptable end)
    pcall(function() Svc.UIS.MouseBehavior = Enum.MouseBehavior.LockCenter end)
    initFPAngles()
    Ref.fpCamConn = Svc.RunService.RenderStepped:Connect(function()
        if not Cfg.FP_ON then return end
        local char = myChar()
        if not char then return end
        local root = getRoot(char)
        local head = char:FindFirstChild("Head")
        if not root then return end
        local delta = Svc.UIS:GetMouseDelta()
        St.fpYaw = St.fpYaw - delta.X * Cfg.FP_SENS
        St.fpPitch = math.clamp(St.fpPitch - delta.Y * Cfg.FP_SENS, Cfg.FP_PITCH_MIN, Cfg.FP_PITCH_MAX)
        local eyePos = head and (head.Position + Vector3.new(0, 0.15, 0)) or (root.Position + Vector3.new(0, 1.5, 0))
        local camCF = CFrame.new(eyePos) * CFrame.Angles(0, math.rad(St.fpYaw), 0) * CFrame.Angles(math.rad(St.fpPitch), 0, 0)
        local curCam = workspace.CurrentCamera
        if curCam then pcall(function() curCam.CFrame = camCF end) end
        if Cfg.AA_MODE == 0 and not Cfg.EMOTE_AA_ON then
            pcall(function()
                local vel = root.AssemblyLinearVelocity
                root.CFrame = CFrame.new(root.Position) * CFrame.Angles(0, math.rad(St.fpYaw), 0)
                root.AssemblyLinearVelocity = vel
            end)
        end
    end)
end

function stopFPCamera()
    if Ref.fpCamConn then pcall(function() Ref.fpCamConn:Disconnect() end); Ref.fpCamConn = nil end
    local cam = workspace.CurrentCamera
    if cam then local restoreType = St.fpOrigCamType or Enum.CameraType.Custom; pcall(function() cam.CameraType = restoreType end) end
    local restoreMouse = St.fpOrigMouseBehavior or Enum.MouseBehavior.Default
    pcall(function() Svc.UIS.MouseBehavior = restoreMouse end)
    St.fpOrigCamType = nil; St.fpOrigMouseBehavior = nil
end

local function buildSword(parent)
    local model=Instance.new("Model"); model.Name="VM_Sword_Fallback"
    local function part(sz,col,mat) local p=Instance.new("Part"); p.Size=sz; p.Color=col; p.Material=mat or Enum.Material.SmoothPlastic; p.CastShadow=true; p.CanCollide=false; p.CanTouch=false; p.Anchored=false; p.Parent=model; return p end
    local function weld(a,b,offset) local w=Instance.new("Weld"); w.Part0=a; w.Part1=b; w.C1=offset or CFrame.new(); w.Parent=a end
    local grip=part(Vector3.new(0.1,0.55,0.1),Color3.fromRGB(90,55,20),Enum.Material.Wood)
    grip.Name="Handle"; model.PrimaryPart=grip
    local blade=part(Vector3.new(0.07,2.3,0.2),Color3.fromRGB(182,187,198),Enum.Material.Metal)
    blade.Name="Blade"; weld(grip,blade,CFrame.new(0,-1.42,0))
    local guard=part(Vector3.new(0.09,0.1,0.62),Color3.fromRGB(65,58,50),Enum.Material.Metal)
    guard.Name="Guard"; weld(grip,guard,CFrame.new(0,0.33,0))
    local pommel=part(Vector3.new(0.17,0.17,0.17),Color3.fromRGB(65,58,50),Enum.Material.Metal)
    pommel.Name="Pommel"; weld(grip,pommel,CFrame.new(0,-0.37,0))
    model.Parent=parent; return model
end

local function toReadyModel(obj)
    if obj:IsA("BasePart") then
        pcall(function() obj.CanCollide=false; obj.CanTouch=false; obj.Anchored=false; obj.CastShadow=true end)
        local model=Instance.new("Model"); model.Name="VM_Asset"
        local root=Instance.new("Part"); root.Name="Root"; root.Size=Vector3.new(0.05,0.05,0.05)
        root.Transparency=1; root.CanCollide=false; root.CanTouch=false; root.Anchored=false; root.CastShadow=false; root.Parent=model
        obj.Parent=model; model.PrimaryPart=root
        local w=Instance.new("Weld"); w.Part0=root; w.Part1=obj; w.C1=CFrame.new(0,Cfg.VM_PIVOT_Y,0); w.Parent=root
        return model,1
    end
    local parts,handle={},nil
    for _,d in ipairs(obj:GetDescendants()) do
        if d:IsA("BasePart") then
            pcall(function() d.CanCollide=false; d.CanTouch=false; d.Anchored=false; d.CastShadow=true end)
            parts[#parts+1]=d
            if not handle and d.Name=="Handle" then handle=d end
        end
    end
    if #parts==0 then return nil,0 end
    if obj:IsA("Model") then if not obj.PrimaryPart then obj.PrimaryPart=handle or parts[1] end; return obj,#parts end
    local wrapper=Instance.new("Model"); wrapper.Name="VM_Asset"; obj.Parent=wrapper; wrapper.PrimaryPart=handle or parts[1]; return wrapper,#parts
end

local function computeHandleOffset(model)
    local prim = model.PrimaryPart
    if not prim then return Vector3.new() end
    local parts = {}
    for _,d in ipairs(model:GetDescendants()) do if d:IsA("BasePart") then parts[#parts+1]=d end end
    if #parts==0 then return Vector3.new() end
    for _,p in ipairs(parts) do local n = p.Name:lower(); if n:find("handle") or n:find("grip") or n:find("hilt") then return prim.CFrame:PointToObjectSpace(p.Position) end end
    local sum, n = Vector3.new(), 0
    for _,p in ipairs(parts) do
        local c = p.Color; local r,g,b = c.R*255, c.G*255, c.B*255
        if r>=55 and r<=150 and g>=25 and g<=100 and b>=5 and b<=70 and r > b+15 and r > g+5 then sum = sum + p.Position; n = n + 1 end
    end
    if n > 0 then return prim.CFrame:PointToObjectSpace(sum / n) end
    local mn = Vector3.new(math.huge,math.huge,math.huge); local mx = Vector3.new(-math.huge,-math.huge,-math.huge)
    for _,p in ipairs(parts) do
        local cf = p.CFrame; local sz = p.Size/2
        for _,corner in ipairs({Vector3.new(1,1,1),Vector3.new(1,1,-1),Vector3.new(1,-1,1),Vector3.new(1,-1,-1),Vector3.new(-1,1,1),Vector3.new(-1,1,-1),Vector3.new(-1,-1,1),Vector3.new(-1,-1,-1)}) do
            local w = cf:PointToWorldSpace(Vector3.new(corner.X*sz.X,corner.Y*sz.Y,corner.Z*sz.Z))
            local l = prim.CFrame:PointToObjectSpace(w)
            mn = Vector3.new(math.min(mn.X,l.X),math.min(mn.Y,l.Y),math.min(mn.Z,l.Z))
            mx = Vector3.new(math.max(mx.X,l.X),math.max(mx.Y,l.Y),math.max(mx.Z,l.Z))
        end
    end
    local range = mx - mn
    local ax, val = "X", range.X
    if range.Y>val then ax,val="Y",range.Y end
    if range.Z>val then ax,val="Z",range.Z end
    local dir = Cfg.VM_SWING_DIR or -1
    local center = (mn+mx)*0.5
    if ax=="X" then return Vector3.new(center.X + dir*range.X*0.4, center.Y, center.Z)
    elseif ax=="Y" then return Vector3.new(center.X, center.Y + dir*range.Y*0.4, center.Z)
    else return Vector3.new(center.X, center.Y, center.Z + dir*range.Z*0.4) end
end

function setupVPFrame(guiParent)
    if Ref.vpFrame then pcall(function() Ref.vpFrame:Destroy() end) end
    if Ref.vpCam then pcall(function() Ref.vpCam:Destroy() end) end
    Ref.vpFrame,Ref.vpCam=nil,nil
    Ref.vpFrame=Instance.new("ViewportFrame")
    Ref.vpFrame.Size=UDim2.new(1,0,1,0); Ref.vpFrame.Position=UDim2.new(0,0,0,0)
    Ref.vpFrame.BackgroundTransparency=1; Ref.vpFrame.BorderSizePixel=0
    Ref.vpFrame.Ambient=Color3.new(1,1,1); Ref.vpFrame.LightDirection=Vector3.new(-0.5,-1,-0.8)
    Ref.vpFrame.Visible=false; Ref.vpFrame.ZIndex=5; Ref.vpFrame.Parent=guiParent
    Ref.vpCam=Instance.new("Camera"); Ref.vpCam.FieldOfView=70
    Ref.vpCam.Parent=Ref.vpFrame; Ref.vpFrame.CurrentCamera=Ref.vpCam
end

function startVPSync()
    if Ref.vpConn then pcall(function() Ref.vpConn:Disconnect() end); Ref.vpConn=nil end
    Ref.vpConn=Svc.RunService.RenderStepped:Connect(function()
        if not Ref.vpFrame or not Ref.vpCam or not Ref.vpModel then return end
        local cam=workspace.CurrentCamera; if not cam then return end
        pcall(function()
            Ref.vpCam.FieldOfView=cam.FieldOfView
            Ref.vpCam.CFrame=cam.CFrame
            Ref.vpModel:PivotTo(cam.CFrame * vmOffset())
        end)
    end)
end

function loadVPModel()
    if St.vmLoading or Ref.vpModel then return end; St.vmLoading=true
    task.spawn(function()
        local rawId=tostring(Cfg.VM_ASSET_ID); local loaded=nil
        local ok1,res1=pcall(function() return game:GetObjects("rbxassetid://"..rawId) end)
        if ok1 and res1 and res1[1] then
            for i=2,#res1 do pcall(function() res1[i]:Destroy() end) end
            local model,n=toReadyModel(res1[1])
            if model and n>0 then loaded=model else pcall(function() res1[1]:Destroy() end) end
        end
        if not loaded then
            local ok2,res2=pcall(function() return Svc.InsertSvc:LoadAsset(tonumber(rawId)) end)
            if ok2 and res2 then
                local inner=res2:GetChildren()[1]
                if inner then
                    inner.Parent=nil; pcall(function() res2:Destroy() end)
                    local model,n=toReadyModel(inner)
                    if model and n>0 then loaded=model else pcall(function() inner:Destroy() end) end
                else pcall(function() res2:Destroy() end) end
            end
        end
        if not loaded then loaded=buildSword(Ref.vpFrame) else loaded.Parent=Ref.vpFrame end
        Ref.vmHandleOffset = computeHandleOffset(loaded)
        Ref.vpModel=loaded; St.vmLoading=false
        if Cfg.MINECRAFT_SWORD then startVPSync() end
    end)
end

function showViewmodel(on)
    if Ref.vpFrame then Ref.vpFrame.Visible=on end
    local char=myChar()
    if on then
        if char then setLocalTransp(char, Cfg.FP_ON and 1 or 0) end
        if not Ref.vpModel then loadVPModel() elseif not Ref.vpConn then startVPSync() end
    else
        if char then setLocalTransp(char, Cfg.FP_ON and 1 or 0) end
        if Ref.vpConn then pcall(function() Ref.vpConn:Disconnect() end); Ref.vpConn=nil end
    end
end

function toggleSword()
    Cfg.MINECRAFT_SWORD = not Cfg.MINECRAFT_SWORD
    showViewmodel(Cfg.MINECRAFT_SWORD)
    if GUI.toggles["Minecraft Sword"] then GUI.toggles["Minecraft Sword"](Cfg.MINECRAFT_SWORD) end
    print("[NexXit] "..(Cfg.MINECRAFT_SWORD and "Sword ON" or "Sword OFF"))
end

function toggleFP()
    if St.fpToggling then return end
    St.fpToggling = true
    Cfg.FP_ON = not Cfg.FP_ON
    if Cfg.FP_ON then
        startFPCamera()
        setLocalTransp(myChar(), 1)
        setGlockLiverVisible(false)
        print("[NexXit] FP ON | char + GlockLiver hidden")
    else
        stopFPCamera()
        setLocalTransp(myChar(), 0)
        setGlockLiverVisible(true)
        print("[NexXit] FP OFF | char + GlockLiver visible")
    end
    if GUI.toggles["FP Camera"] then GUI.toggles["FP Camera"](Cfg.FP_ON) end
    task.delay(0.25, function() St.fpToggling = false end)
end

-- ============ AC BYPASS ============
function startACBypass()
    pcall(function()
        local ban=Svc.RS:FindFirstChild("Remotes") and Svc.RS.Remotes:FindFirstChild("ExploitBan")
        if ban then
            local mt=getrawmetatable(game); local old=mt.__namecall
            setreadonly(mt,false)
            mt.__namecall=newcclosure(function(self,...)
                local m=getnamecallmethod()
                if self==ban and (m=="FireServer" or m=="InvokeServer") then return end
                return old(self,...)
            end)
            setreadonly(mt,true)
        end
    end)
    if Ref.acBypassConn then Ref.acBypassConn:Disconnect() end
    Ref.acBypassConn=Svc.RunService.Heartbeat:Connect(function()
        local char=myChar(); if not char then return end
        local hum,root=getHum(char),getRoot(char)
        pcall(function()
            if hum then
                if hum.WalkSpeed>16 then hum.WalkSpeed=16 end
                if hum.UseJumpPower and hum.JumpPower>50 then hum.JumpPower=50
                elseif hum.JumpHeight and hum.JumpHeight>7.2 then hum.JumpHeight=7.2 end
            end
            if root then local s=root.Size; if s.X>2.1 or s.Y>2.1 or s.Z>1.1 then root.Size=Vector3.new(2,2,1) end end
        end)
    end)
    Ref.connections[#Ref.connections+1]=Ref.acBypassConn
end

function startWatchdog()
    if Ref.watchdogConn then Ref.watchdogConn:Disconnect() end
    Ref.watchdogConn=Svc.RunService.Heartbeat:Connect(function()
        if not St.running then return end
        if tick()-St.lastShotAt<5 then return end
        if Cfg.MINECRAFT_SWORD and not holdingTool() then return end
        for _,plr in ipairs(Svc.Players:GetPlayers()) do
            if plr~=me and valid(plr) then St.lastShotAt=tick(); setRunning(true); return end
        end
    end)
    Ref.connections[#Ref.connections+1]=Ref.watchdogConn
end

function startFFWatcher()
    Ref.connections[#Ref.connections+1]=Svc.RunService.Heartbeat:Connect(function()
        if not St.running then return end
        for _,plr in ipairs(Svc.Players:GetPlayers()) do
            if plr==me then continue end
            local char=plr.Character; local cur,prev=hasFF(char),Ref.playerFFState[plr]
            if prev==nil then Ref.playerFFState[plr]=cur; continue end
            if prev==true and cur==false then
                local ok,c,part,root,dist,origin,hitPos,aimPos=valid(plr)
                if ok then Ref.ffTriggerQueue[#Ref.ffTriggerQueue+1]={plr=plr,char=c,part=part,root=root,dist=dist,origin=origin,hitPos=hitPos,aimPos=aimPos} end
            end
            Ref.playerFFState[plr]=cur
        end
    end)
end

-- ============ BHOP ============
function stopBhop()
    Cfg.BHOP_ON=false; St.currentSpeed=0; St.bhopSmoothVel=0
    if Ref.bhopConn then Ref.bhopConn:Disconnect(); Ref.bhopConn=nil end
    if GUI.toggles["Bunny Hop"] then GUI.toggles["Bunny Hop"](false) end
end

function startBhop()
    stopBhop(); Cfg.BHOP_ON=true; St.currentSpeed=Cfg.BHOP_START; St.bhopSmoothVel=Cfg.BHOP_START
    if GUI.toggles["Bunny Hop"] then GUI.toggles["Bunny Hop"](true) end
    Ref.bhopConn=Svc.RunService.Heartbeat:Connect(function(dt)
        if not Cfg.BHOP_ON then return end
        local char=myChar(); if not char then return end
        local hum,root=getHum(char),getRoot(char); if not hum or not root then return end
        local grounded=hum.FloorMaterial~=Enum.Material.Air
        local move,vel=hum.MoveDirection,root.AssemblyLinearVelocity
        local flat=Vector3.new(vel.X,0,vel.Z)
        local smoothAccel=Cfg.BHOP_ACCEL*dt*60
        local jumpVel=Cfg.BHOP_JUMP_DIST*0.8
        if grounded then
            if not St.wasGrounded then
                local target=math.min(flat.Magnitude+Cfg.BHOP_GAIN,Cfg.BHOP_MAX)
                St.bhopSmoothVel=St.bhopSmoothVel+(target-St.bhopSmoothVel)*math.min(smoothAccel*0.5,1)
            else
                if move.Magnitude<0.05 then
                    St.bhopSmoothVel=math.max(Cfg.BHOP_START,St.bhopSmoothVel*Cfg.BHOP_FRICTION)
                else
                    St.bhopSmoothVel=St.bhopSmoothVel+(Cfg.BHOP_MAX-St.bhopSmoothVel)*math.min(smoothAccel*0.15,0.3)
                end
            end
            St.currentSpeed=math.clamp(St.bhopSmoothVel,Cfg.BHOP_START,Cfg.BHOP_MAX)
            if move.Magnitude>0.05 then
                local w=Vector3.new(move.X,0,move.Z).Unit*math.min(St.currentSpeed,Cfg.BHOP_MAX)
                root.AssemblyLinearVelocity=Vector3.new(w.X,jumpVel,w.Z)
            else
                root.AssemblyLinearVelocity=Vector3.new(0,jumpVel,0)
            end
            St.wasGrounded=true
        else
            if move.Magnitude>0.05 then
                local wish=Vector3.new(move.X,0,move.Z).Unit
                local curr=flat
                local add=St.currentSpeed*Cfg.BHOP_STRAFE-curr:Dot(wish)
                if add>0 then curr=curr+wish*math.min(smoothAccel*St.currentSpeed*0.5,add) end
                St.bhopSmoothVel=St.bhopSmoothVel+(Cfg.BHOP_MAX-St.bhopSmoothVel)*math.min(smoothAccel*0.05,0.15)
                St.currentSpeed=math.clamp(St.bhopSmoothVel,Cfg.BHOP_START,Cfg.BHOP_MAX)
                if curr.Magnitude>Cfg.BHOP_MAX then curr=curr.Unit*Cfg.BHOP_MAX end
                root.AssemblyLinearVelocity=Vector3.new(curr.X,vel.Y,curr.Z)
            end
            St.wasGrounded=false
        end
    end)
end

function toggleBhop() if Cfg.BHOP_ON then stopBhop() else startBhop() end end

-- ============ ANTIAIM ============
function stopAA()
    if Ref.aaConn then pcall(function() Ref.aaConn:Disconnect() end); Ref.aaConn=nil end
    if not Cfg.EMOTE_AA_ON and not Cfg.FP_ON then
        local hum=getHum(myChar()); if hum then pcall(function() hum.AutoRotate=true end) end
    end
end

function applyAA()
    if Ref.aaConn then pcall(function() Ref.aaConn:Disconnect() end); Ref.aaConn=nil end
    if Cfg.AA_MODE==0 then
        if not Cfg.EMOTE_AA_ON and not Cfg.FP_ON then
            local hum=getHum(myChar()); if hum then pcall(function() hum.AutoRotate=true end) end
        end
        return
    end
    Ref.aaConn=Svc.RunService.RenderStepped:Connect(function(dt)
        if Cfg.AA_MODE==0 then return end
        if Cfg.FP_ON then return end
        local char=myChar(); if not char then return end
        local root,hum=getRoot(char),getHum(char); if not root or not hum then return end
        pcall(function() hum.AutoRotate=false end)
        local cam=workspace.CurrentCamera
        local base=cam and cam.CFrame.LookVector or root.CFrame.LookVector
        local baseYaw=math.atan2(-base.X,-base.Z); local yaw=baseYaw
        if      Cfg.AA_MODE==1 then St.aaYaw=St.aaYaw+Cfg.AA_SPEED*dt*15; yaw=math.rad(St.aaYaw)
        elseif  Cfg.AA_MODE==2 then yaw=baseYaw+math.rad((math.floor(tick()*12)%2==0) and Cfg.AA_JITTER or -Cfg.AA_JITTER)
        elseif  Cfg.AA_MODE==3 then yaw=baseYaw+math.pi
        elseif  Cfg.AA_MODE==4 then
            if math.floor(tick()*8)~=math.floor((tick()-dt)*8) then St.aaYaw=math.random()*math.pi*2 end
            yaw=St.aaYaw
        elseif  Cfg.AA_MODE==5 then
            if Cfg.EMOTE_AA_ON then yaw=baseYaw else yaw=baseYaw+math.rad(math.sin(tick()*6)*Cfg.AA_SWAY) end
        end
        local vel=root.AssemblyLinearVelocity
        root.CFrame=CFrame.new(root.Position)*CFrame.Angles(0,yaw,0)
        root.AssemblyLinearVelocity=vel
    end)
end

function cycleAA(dir) Cfg.AA_MODE=(Cfg.AA_MODE+(dir or 1))%AA_COUNT; applyAA() end
function toggleJitter() Cfg.AA_MODE=Cfg.AA_MODE==2 and 0 or 2; applyAA() end

-- ============ EMOTE AA ============
local function getAnimate(char) char=char or myChar(); return char and char:FindFirstChild("Animate") end
local function noanim() local a=getAnimate(); if a and not a.Disabled then St.aaAnimateWasDisabled=true; pcall(function() a.Disabled=true end) end end
local function reanim() local a=getAnimate(); if a and St.aaAnimateWasDisabled then pcall(function() a.Disabled=false end) end; St.aaAnimateWasDisabled=false end
local function deepFindAnim(r) if r:IsA("Animation") and r.AnimationId~="" then return r end; for _,d in ipairs(r:GetDescendants()) do if d:IsA("Animation") and d.AnimationId and d.AnimationId~="" then return d end end; return nil end

local function resolveEmoteAnimId(catalogId,cb)
    if St.resolvedEmoteAnimId then cb(St.resolvedEmoteAnimId); return end
    if St.emoteResolving then return end; St.emoteResolving=true
    task.spawn(function()
        local rawId=tostring(catalogId):match("(%d+)") or tostring(catalogId)
        local okGO,goRes=pcall(function() return game:GetObjects("rbxassetid://"..rawId) end)
        if okGO and goRes then
            for _,obj in ipairs(goRes) do
                local found=deepFindAnim(obj)
                if found then St.resolvedEmoteAnimId=found.AnimationId; pcall(function() obj:Destroy() end); St.emoteResolving=false; cb(St.resolvedEmoteAnimId); return end
                pcall(function() obj:Destroy() end)
            end
        end
        local okIS,model=pcall(function() return Svc.InsertSvc:LoadAsset(tonumber(rawId)) end)
        if okIS and model then
            local found=deepFindAnim(model)
            if found then St.resolvedEmoteAnimId=found.AnimationId; pcall(function() model:Destroy() end); St.emoteResolving=false; cb(St.resolvedEmoteAnimId); return end
            pcall(function() model:Destroy() end)
        end
        St.resolvedEmoteAnimId="rbxassetid://"..rawId; St.emoteResolving=false; cb(St.resolvedEmoteAnimId)
    end)
end

local function stopAAEmote()
    local watch=Ref.aaEmoteWatch; Ref.aaEmoteWatch=nil
    if watch then pcall(function() task.cancel(watch) end) end
    if Ref.emoteAAConn then pcall(function() Ref.emoteAAConn:Disconnect() end); Ref.emoteAAConn=nil end
    if Ref.aaEmoteTrack then pcall(function() Ref.aaEmoteTrack:Stop(0) end); pcall(function() Ref.aaEmoteTrack:Destroy() end); Ref.aaEmoteTrack=nil end
    if Ref.aaEmoteAnim then pcall(function() Ref.aaEmoteAnim:Destroy() end); Ref.aaEmoteAnim=nil end
    local char=myChar()
    local hum=char and (char:FindFirstChildOfClass("Humanoid") or char:FindFirstChildOfClass("AnimationController"))
    if hum then
        local want=tostring(Cfg.AA_EMOTE_ID)
        pcall(function()
            for _,t in ipairs(hum:GetPlayingAnimationTracks()) do
                local aid=""; pcall(function() aid=t.Animation and t.Animation.AnimationId or "" end)
                if aid:find(want,1,true) then pcall(function() t:Stop(0) end); pcall(function() t:Destroy() end) end
            end
        end)
    end
    local h2=char and getHum(char); if h2 then pcall(function() h2.AutoRotate=true end) end
    St.aaEmoteChar=nil; reanim()
end

local function playAAEmoteWithId(animId)
    local char=myChar(); if not char then return false end
    local hum=char:FindFirstChildWhichIsA("Humanoid"); if not hum then return false end
    if not isAlive(char,hum) then return false end
    if Ref.aaEmoteTrack and St.aaEmoteChar==char then
        local playing=false; pcall(function() playing=Ref.aaEmoteTrack.IsPlaying end)
        if playing then pcall(function() Ref.aaEmoteTrack:AdjustSpeed(tonumber(Cfg.AA_EMOTE_SPEED) or 1) end); return true end
    end
    local watch=Ref.aaEmoteWatch; Ref.aaEmoteWatch=nil
    if watch then pcall(function() task.cancel(watch) end) end
    if Ref.aaEmoteTrack then pcall(function() Ref.aaEmoteTrack:Stop(0) end); pcall(function() Ref.aaEmoteTrack:Destroy() end); Ref.aaEmoteTrack=nil end
    if Ref.aaEmoteAnim then pcall(function() Ref.aaEmoteAnim:Destroy() end); Ref.aaEmoteAnim=nil end
    local rawId=tostring(Cfg.AA_EMOTE_ID):match("(%d+)") or tostring(Cfg.AA_EMOTE_ID)
    pcall(function() for _,t in ipairs(hum:GetPlayingAnimationTracks()) do local aid=""; pcall(function() aid=t.Animation and t.Animation.AnimationId or "" end); if aid:find(rawId,1,true) then t:Stop(0); t:Destroy() end end end)
    noanim()
    local speed=tonumber(Cfg.AA_EMOTE_SPEED) or 1; local track,animInst=nil,nil
    local okPE,peTrack=pcall(function() return hum:PlayEmoteAndGetAnimTrackById(tonumber(rawId)) end)
    if okPE and peTrack then track=peTrack; pcall(function() track.Looped=true; if not track.IsPlaying then track:Play() end; track:AdjustSpeed(speed) end) end
    if not track then
        pcall(function()
            local anim=Instance.new("Animation"); anim.AnimationId=animId; animInst=anim
            local t=hum:LoadAnimation(anim); t.Priority=Enum.AnimationPriority.Action4; t.Looped=true; t:Play()
            if speed~=1 then t:AdjustSpeed(speed) end; track=t
        end)
        if not track then pcall(function() if animInst then animInst:Destroy() end end); animInst=nil end
    end
    if not track then
        pcall(function()
            local animator=hum:FindFirstChildOfClass("Animator")
            if not animator then animator=Instance.new("Animator"); animator.Parent=hum end
            local anim=Instance.new("Animation"); anim.AnimationId=animId; animInst=anim
            local t=animator:LoadAnimation(anim); t.Priority=Enum.AnimationPriority.Action4; t.Looped=true; t:Play()
            if speed~=1 then t:AdjustSpeed(speed) end; track=t
        end)
        if not track then pcall(function() if animInst then animInst:Destroy() end end); animInst=nil; reanim(); return false end
    end
    pcall(function() track.Looped=true; if not track.IsPlaying then track:Play() end; track:AdjustSpeed(speed) end)
    Ref.aaEmoteTrack=track; Ref.aaEmoteAnim=animInst; St.aaEmoteChar=char
    Ref.aaEmoteWatch=task.spawn(function()
        while Cfg.EMOTE_AA_ON do
            task.wait(0.35); if not Cfg.EMOTE_AA_ON then break end
            local c=myChar(); local h=c and c:FindFirstChildWhichIsA("Humanoid")
            if not c or not h or not isAlive(c,h) then continue end
            local a=c:FindFirstChild("Animate")
            if a and not a.Disabled then pcall(function() a.Disabled=true end); St.aaAnimateWasDisabled=true end
            if St.aaEmoteChar~=c then if Cfg.EMOTE_AA_ON then resolveEmoteAnimId(Cfg.AA_EMOTE_ID,playAAEmoteWithId) end; return end
            local playing=false; pcall(function() playing=Ref.aaEmoteTrack and Ref.aaEmoteTrack.IsPlaying end)
            if not playing then if Cfg.EMOTE_AA_ON then resolveEmoteAnimId(Cfg.AA_EMOTE_ID,playAAEmoteWithId) end; return
            else pcall(function() Ref.aaEmoteTrack.Looped=true; Ref.aaEmoteTrack:AdjustSpeed(tonumber(Cfg.AA_EMOTE_SPEED) or 1) end) end
        end
    end)
    return true
end

local function playAAEmote() resolveEmoteAnimId(Cfg.AA_EMOTE_ID,playAAEmoteWithId) end

local function startEmoteAARender()
    if Ref.emoteAAConn then pcall(function() Ref.emoteAAConn:Disconnect() end); Ref.emoteAAConn=nil end
    Ref.emoteAAConn=Svc.RunService.RenderStepped:Connect(function()
        if not Cfg.EMOTE_AA_ON then return end
        local char=myChar(); if not char then return end
        local root,hum=getRoot(char),getHum(char); if not root or not hum then return end
        if St.aaEmoteChar~=char or not Ref.aaEmoteTrack then playAAEmote(); return end
        local playing=false; pcall(function() playing=Ref.aaEmoteTrack.IsPlaying end)
        if not playing then playAAEmote(); return end
        pcall(function() hum.AutoRotate=false end)
        if Cfg.AA_MODE~=0 then return end
        if Cfg.FP_ON then return end
        local cam=workspace.CurrentCamera
        local base=cam and cam.CFrame.LookVector or root.CFrame.LookVector
        local yaw=math.atan2(-base.X,-base.Z)+math.pi
        local vel=root.AssemblyLinearVelocity
        root.CFrame=CFrame.new(root.Position)*CFrame.Angles(0,yaw,0)
        root.AssemblyLinearVelocity=vel
    end)
end

function startEmoteAA()
    Cfg.EMOTE_AA_ON=true; task.defer(playAAEmote); startEmoteAARender()
    if GUI.toggles["Emote AA"] then GUI.toggles["Emote AA"](true) end
end

function stopEmoteAA()
    Cfg.EMOTE_AA_ON=false; stopAAEmote()
    if GUI.toggles["Emote AA"] then GUI.toggles["Emote AA"](false) end
end

function toggleEmoteAA() if Cfg.EMOTE_AA_ON then stopEmoteAA() else startEmoteAA() end end

-- ============ ESP ============
local function safeRem(o) if not o then return end; pcall(function() o.Visible=false end); pcall(function() o:Remove() end) end
local function destroyESP(plr)
    local d=Ref.espDraws[plr]; if not d then return end
    for _,k in ipairs({"boxBg","box","name","dist","hpBg","hpFg","snap"}) do safeRem(d[k]) end
    if d.corners then for i=1,#d.corners do safeRem(d.corners[i]) end end; Ref.espDraws[plr]=nil
end
local function destroyAllESP() for p in pairs(Ref.espDraws) do destroyESP(p) end; Ref.espDraws={} end
local function mkDraw(class,props)
    local ok,d=pcall(function() local o=Drawing.new(class); for k,v in pairs(props) do pcall(function() o[k]=v end) end; o.Visible=false; return o end); return ok and d or nil
end
local function ensureESP(plr)
    local ex=Ref.espDraws[plr]
    if ex and ex.box then if pcall(function() ex.box.Transparency=ex.box.Transparency end) then return ex end; destroyESP(plr)
    elseif ex then destroyESP(plr) end
    local d={
        boxBg=mkDraw("Square",{Thickness=2,Color=Color3.new(0,0,0),Filled=false}),
        box=mkDraw("Square",{Thickness=1,Color=Color3.fromRGB(255,255,255),Filled=false}),
        corners={},
        name=mkDraw("Text",{Center=true,Outline=true,OutlineColor=Color3.new(0,0,0),Color=Color3.new(1,1,1),Size=14,Font=2}),
        dist=mkDraw("Text",{Center=true,Outline=true,OutlineColor=Color3.new(0,0,0),Color=Color3.fromRGB(180,190,200),Size=12,Font=2}),
        hpBg=mkDraw("Square",{Thickness=0,Color=Color3.fromRGB(20,20,20),Filled=true}),
        hpFg=mkDraw("Square",{Thickness=0,Filled=true}),
        snap=mkDraw("Line",{Thickness=1,Color=Color3.fromRGB(255,255,255),Transparency=0.55}),
        charRef=nil,
    }
    for i=1,8 do d.corners[i]=mkDraw("Line",{Thickness=2,Color=Color3.fromRGB(255,255,255)}) end
    if not d.box or not d.name then
        for _,k in ipairs({"boxBg","box","name","dist","hpBg","hpFg","snap"}) do safeRem(d[k]) end
        for i=1,8 do safeRem(d.corners[i]) end; return nil
    end
    Ref.espDraws[plr]=d; return d
end
local function hideESP(d)
    if not d then return end
    pcall(function()
        for _,k in ipairs({"boxBg","box","name","dist","hpBg","hpFg","snap"}) do if d[k] then d[k].Visible=false end end
        if d.corners then for i=1,8 do if d.corners[i] then d.corners[i].Visible=false end end end
    end)
end
local function updateESP()
    if not Cfg.ESP_ON then for _,d in pairs(Ref.espDraws) do hideESP(d) end; return end
    local cam=workspace.CurrentCamera; if not cam then return end
    local myRoot=getRoot(myChar()); local myPos=myRoot and myRoot.Position or cam.CFrame.Position
    local vp,seen=cam.ViewportSize,{}
    for _,plr in ipairs(Svc.Players:GetPlayers()) do
        if plr==me then continue end; seen[plr]=true
        local char=plr.Character; local hum=char and getHum(char)
        local root=char and getRoot(char); local head=char and char:FindFirstChild("Head")
        local d=ensureESP(plr); if not d then continue end
        if char and d.charRef and d.charRef~=char then hideESP(d); d.charRef=char elseif char then d.charRef=char end
        if not char or not char.Parent or not hum or not root or not isAlive(char,hum) then hideESP(d); continue end
        local dist=(myPos-root.Position).Magnitude
        if dist>Cfg.ESP_MAX then hideESP(d); continue end
        local topY=head and (head.Position.Y+0.7) or (root.Position.Y+2.5)
        local botY=root.Position.Y-3
        local vTop=cam:WorldToViewportPoint(Vector3.new(root.Position.X,topY,root.Position.Z))
        local vBot=cam:WorldToViewportPoint(Vector3.new(root.Position.X,botY,root.Position.Z))
        if vTop.Z<0.1 and vBot.Z<0.1 then hideESP(d); continue end
        local cx,cy=(vTop.X+vBot.X)*0.5,(vTop.Y+vBot.Y)*0.5
        if cx<-80 or cy<-80 or cx>vp.X+80 or cy>vp.Y+80 then hideESP(d); continue end
        local h=math.clamp(math.abs(vBot.Y-vTop.Y),6,vp.Y*1.5)
        local w=math.clamp(h*0.55,12,280); local x=vTop.X-w*0.5; local y=math.min(vTop.Y,vBot.Y)
        local hp=math.clamp(hum.Health/math.max(hum.MaxHealth,1),0,1)
        local ff,afk=hasFF(char),isAFK(root)
        local inF=inFOV(root.Position) or (head and inFOV(head.Position))
        local isMate=getgenv()._tsIsTeammate(plr)
        local col
        if isMate then col=Cfg.TEAM_COLOR
        elseif ff then col=Color3.fromRGB(255,210,60)
        elseif afk then col=Color3.fromRGB(180,120,255)
        elseif not inF then col=Color3.fromRGB(100,100,110)
        elseif hp<0.3 then col=Color3.fromRGB(255,55,55)
        elseif hp<0.6 then col=Color3.fromRGB(255,170,50)
        else col=Color3.fromRGB(255,255,255) end
        pcall(function()
            d.boxBg.Size=Vector2.new(w+2,h+2); d.boxBg.Position=Vector2.new(x-1,y-1); d.boxBg.Visible=true
            d.box.Size=Vector2.new(w,h); d.box.Position=Vector2.new(x,y); d.box.Color=col; d.box.Visible=true
            local len=math.clamp(h*0.22,5,16); local c=d.corners
            local function sc(i,a,b) if c[i] then c[i].From=a; c[i].To=b; c[i].Color=col; c[i].Visible=true end end
            sc(1,Vector2.new(x,y),Vector2.new(x+len,y)); sc(2,Vector2.new(x,y),Vector2.new(x,y+len))
            sc(3,Vector2.new(x+w,y),Vector2.new(x+w-len,y)); sc(4,Vector2.new(x+w,y),Vector2.new(x+w,y+len))
            sc(5,Vector2.new(x,y+h),Vector2.new(x+len,y+h)); sc(6,Vector2.new(x,y+h),Vector2.new(x,y+h-len))
            sc(7,Vector2.new(x+w,y+h),Vector2.new(x+w-len,y+h)); sc(8,Vector2.new(x+w,y+h),Vector2.new(x+w,y+h-len))
            d.hpBg.Size=Vector2.new(5,h+2); d.hpBg.Position=Vector2.new(x-8,y-1); d.hpBg.Visible=true
            local fh=math.max(h*hp,1)
            d.hpFg.Size=Vector2.new(3,fh); d.hpFg.Position=Vector2.new(x-7,y+(h-fh))
            d.hpFg.Color=Color3.fromRGB(math.floor(255*(1-hp)),math.floor(255*hp),40); d.hpFg.Visible=true
            local nm=plr.DisplayName; if not nm or nm=="" then nm=plr.Name end
            if isMate then nm="[TEAM] "..nm end
            d.name.Text=nm; d.name.Position=Vector2.new(x+w*0.5,y-17); d.name.Color=col; d.name.Visible=true
            local flags={}
            if isMate then flags[#flags+1]="ALLY" end
            if ff then flags[#flags+1]="FF" end; if afk then flags[#flags+1]="AFK" end
            if not inF then flags[#flags+1]="FOV" end; if Ref.deadCache[plr] then flags[#flags+1]="DEAD" end
            d.dist.Text=string.format("%dm | %dhp%s",math.floor(dist),math.floor(hum.Health),#flags>0 and (" | "..table.concat(flags," ")) or "")
            d.dist.Position=Vector2.new(x+w*0.5,y+h+3); d.dist.Visible=true
            if d.snap then
                d.snap.From=Vector2.new(vp.X*0.5,vp.Y); d.snap.To=Vector2.new(x+w*0.5,y+h)
                d.snap.Color=col; d.snap.Visible=dist<200 and inF and not isMate
            end
        end)
    end
    for plr in pairs(Ref.espDraws) do if not seen[plr] then destroyESP(plr) end end
end

function stopESP()
    Cfg.ESP_ON=false
    if Ref.espConn then pcall(function() Ref.espConn:Disconnect() end); Ref.espConn=nil end
    destroyAllESP()
    if GUI.toggles["ESP"] then GUI.toggles["ESP"](false) end
end

function startESP()
    stopESP(); Cfg.ESP_ON=true
    if GUI.toggles["ESP"] then GUI.toggles["ESP"](true) end
    Ref.espConn=Svc.RunService.RenderStepped:Connect(function() pcall(updateESP) end)
end

function toggleESP() if Cfg.ESP_ON then stopESP() else startESP() end end

-- ============ CAMERA NOCLIP ============
function stopCamNoclip()
    Cfg.CAM_NOCLIP=false
    if Ref.camNoclipConn then pcall(function() Ref.camNoclipConn:Disconnect() end); Ref.camNoclipConn=nil end
    if GUI.toggles["Cam Noclip"] then GUI.toggles["Cam Noclip"](false) end
end

function startCamNoclip()
    stopCamNoclip(); Cfg.CAM_NOCLIP=true; St.camLastDist=12
    if GUI.toggles["Cam Noclip"] then GUI.toggles["Cam Noclip"](true) end
    Ref.camNoclipConn=Svc.RunService.RenderStepped:Connect(function()
        if not Cfg.CAM_NOCLIP then return end
        local cam=workspace.CurrentCamera
        if not cam or cam.CameraType~=Enum.CameraType.Custom then return end
        local char=myChar(); if not char then return end
        local head=char:FindFirstChild("Head"); if not head then return end
        local headPos=head.Position+Vector3.new(0,1.5,0)
        local camPos=cam.CFrame.Position
        local offset=camPos-headPos; local dist=offset.Magnitude
        if dist>1.5 then St.camLastDist=St.camLastDist+(dist-St.camLastDist)*0.3; if St.camLastDist<2 then St.camLastDist=2 end end
        if dist<St.camLastDist*0.85 and dist>0.5 then cam.CFrame=CFrame.new(headPos+offset.Unit*St.camLastDist,headPos) end
    end)
end

function toggleCamNoclip() if Cfg.CAM_NOCLIP then stopCamNoclip() else startCamNoclip() end end

-- ============ GUI ============
pcall(function()
    local p=(gethui and gethui()) or game:GetService("CoreGui")
    local o=p:FindFirstChild("NexXitPainel"); if o then o:Destroy() end
end)

local gui=Instance.new("ScreenGui")
gui.Name="NexXitPainel"; gui.ResetOnSpawn=false; gui.ZIndexBehavior=Enum.ZIndexBehavior.Sibling
pcall(function() gui.Parent=(gethui and gethui()) or game:GetService("CoreGui") end)
if not gui.Parent then gui.Parent=me:WaitForChild("PlayerGui") end
Ref.gui=gui
setupVPFrame(gui)

local main=Instance.new("Frame",gui)
main.Name="MainWindow"; main.Size=UDim2.new(0,520,0,380)
main.Position=UDim2.new(0.5,-260,0.5,-190)
main.BackgroundColor3=Theme.bg; main.BorderSizePixel=0
main.Active=true; main.Draggable=true; main.ZIndex=10
main.Visible=Cfg.GUI_VISIBLE; Ref.mainWindow=main
Instance.new("UICorner",main).CornerRadius=UDim.new(0,10)

-- White Glow Border
local glow=Instance.new("UIStroke",main)
glow.Color=Theme.white; glow.Thickness=2; glow.Transparency=0.3

local mainStroke=Instance.new("UIStroke",main)
mainStroke.Color=Theme.border; mainStroke.Thickness=1

local titleBar=Instance.new("Frame",main)
titleBar.Size=UDim2.new(1,0,0,36); titleBar.BackgroundColor3=Theme.bgDark
titleBar.BorderSizePixel=0; titleBar.ZIndex=11
Instance.new("UICorner",titleBar).CornerRadius=UDim.new(0,10)
local fixBot=Instance.new("Frame",titleBar)
fixBot.Size=UDim2.new(1,0,0,10); fixBot.Position=UDim2.new(0,0,1,-10)
fixBot.BackgroundColor3=Theme.bgDark; fixBot.BorderSizePixel=0; fixBot.ZIndex=11
local titleText=Instance.new("TextLabel",titleBar)
titleText.Size=UDim2.new(1,-140,1,0); titleText.Position=UDim2.new(0,14,0,0)
titleText.BackgroundTransparency=1; titleText.ZIndex=12
titleText.Font=Enum.Font.GothamBold; titleText.TextSize=14
titleText.TextColor3=Theme.white; titleText.TextXAlignment=Enum.TextXAlignment.Left
titleText.Text="NEXXIT PAINEL RBLX"
local verText=Instance.new("TextLabel",titleBar)
verText.Size=UDim2.new(0,130,1,0); verText.Position=UDim2.new(1,-140,0,0)
verText.BackgroundTransparency=1; verText.ZIndex=12
verText.Font=Enum.Font.Code; verText.TextSize=10
verText.TextColor3=Theme.textMuted; verText.TextXAlignment=Enum.TextXAlignment.Right
verText.Text="v1.0 Made By NexXit"

local sidebar=Instance.new("Frame",main)
sidebar.Size=UDim2.new(0,130,1,-36); sidebar.Position=UDim2.new(0,0,0,36)
sidebar.BackgroundColor3=Theme.bgDark; sidebar.BorderSizePixel=0; sidebar.ZIndex=11
local tabList=Instance.new("UIListLayout",sidebar)
tabList.SortOrder=Enum.SortOrder.LayoutOrder; tabList.Padding=UDim.new(0,2)
tabList.HorizontalAlignment=Enum.HorizontalAlignment.Center
local tabPad=Instance.new("UIPadding",sidebar)
tabPad.PaddingTop=UDim.new(0,10); tabPad.PaddingLeft=UDim.new(0,8); tabPad.PaddingRight=UDim.new(0,8)
local tabs={"Combat","Movement","Render","Player","Weapon","Misc"}

local content=Instance.new("Frame",main)
content.Size=UDim2.new(1,-130,1,-60); content.Position=UDim2.new(0,130,0,36)
content.BackgroundColor3=Theme.bg; content.BorderSizePixel=0; content.ZIndex=11
content.ClipsDescendants=true

local statusBar=Instance.new("Frame",main)
statusBar.Size=UDim2.new(1,-130,0,24); statusBar.Position=UDim2.new(0,130,1,-24)
statusBar.BackgroundColor3=Theme.bgDark; statusBar.BorderSizePixel=0; statusBar.ZIndex=12
Instance.new("UICorner",statusBar).CornerRadius=UDim.new(0,6)
GUI.statusLabel=Instance.new("TextLabel",statusBar)
GUI.statusLabel.Size=UDim2.new(0.5,-8,1,0); GUI.statusLabel.Position=UDim2.new(0,8,0,0)
GUI.statusLabel.BackgroundTransparency=1; GUI.statusLabel.ZIndex=13
GUI.statusLabel.Font=Enum.Font.Code; GUI.statusLabel.TextSize=10
GUI.statusLabel.TextColor3=Theme.textDim; GUI.statusLabel.TextXAlignment=Enum.TextXAlignment.Left
GUI.statusLabel.Text="IDLE"
GUI.logLabel=Instance.new("TextLabel",statusBar)
GUI.logLabel.Size=UDim2.new(0.5,-8,1,0); GUI.logLabel.Position=UDim2.new(0.5,0,0,0)
GUI.logLabel.BackgroundTransparency=1; GUI.logLabel.ZIndex=13
GUI.logLabel.Font=Enum.Font.Code; GUI.logLabel.TextSize=9
GUI.logLabel.TextColor3=Theme.textMuted; GUI.logLabel.TextXAlignment=Enum.TextXAlignment.Right
GUI.logLabel.Text="ready | \\ to toggle"

-- GUI Helpers
local function makeToggle(parent, name, default, callback, order)
    local row=Instance.new("Frame",parent)
    row.Size=UDim2.new(1,-16,0,32); row.BackgroundTransparency=1; row.ZIndex=12; row.LayoutOrder=order or 0
    local lbl=Instance.new("TextLabel",row)
    lbl.Size=UDim2.new(1,-50,1,0); lbl.BackgroundTransparency=1; lbl.ZIndex=12
    lbl.Font=Enum.Font.Gotham; lbl.TextSize=12
    lbl.TextColor3=Theme.textMain; lbl.TextXAlignment=Enum.TextXAlignment.Left; lbl.Text=name
    local track=Instance.new("TextButton",row)
    track.Size=UDim2.new(0,40,0,20); track.Position=UDim2.new(1,-44,0.5,-10)
    track.BackgroundColor3=default and Theme.toggleOn or Theme.toggleOff
    track.BorderSizePixel=0; track.ZIndex=13; track.Text=""; track.AutoButtonColor=false
    Instance.new("UICorner",track).CornerRadius=UDim.new(1,0)
    local knob=Instance.new("Frame",track)
    knob.Size=UDim2.new(0,16,0,16)
    knob.Position=default and UDim2.new(1,-18,0.5,-8) or UDim2.new(0,2,0.5,-8)
    knob.BackgroundColor3=Color3.new(1,1,1); knob.BorderSizePixel=0; knob.ZIndex=14
    Instance.new("UICorner",knob).CornerRadius=UDim.new(1,0)
    local state=default
    local function setVisual(on)
        state=on
        Svc.TS:Create(track,TweenInfo.new(0.2),{BackgroundColor3=on and Theme.toggleOn or Theme.toggleOff}):Play()
        Svc.TS:Create(knob,TweenInfo.new(0.2),{Position=on and UDim2.new(1,-18,0.5,-8) or UDim2.new(0,2,0.5,-8)}):Play()
    end
    track.MouseButton1Click:Connect(function() state=not state; setVisual(state); if callback then callback(state) end end)
    GUI.toggles[name]=function(v) if v~=state then setVisual(v) end end
    return row
end

local function makeSlider(parent, name, min, max, default, callback, order, step)
    step=step or 1
    local row=Instance.new("Frame",parent)
    row.Size=UDim2.new(1,-16,0,38); row.BackgroundTransparency=1; row.ZIndex=12; row.LayoutOrder=order or 0
    local lbl=Instance.new("TextLabel",row)
    lbl.Size=UDim2.new(0.6,0,0,16); lbl.BackgroundTransparency=1; lbl.ZIndex=12
    lbl.Font=Enum.Font.Gotham; lbl.TextSize=11
    lbl.TextColor3=Theme.textMain; lbl.TextXAlignment=Enum.TextXAlignment.Left; lbl.Text=name
    local valLbl=Instance.new("TextLabel",row)
    valLbl.Size=UDim2.new(0.4,0,0,16); valLbl.BackgroundTransparency=1; valLbl.ZIndex=12
    valLbl.Font=Enum.Font.Code; valLbl.TextSize=11
    valLbl.TextColor3=Theme.white; valLbl.TextXAlignment=Enum.TextXAlignment.Right; valLbl.Text=tostring(default)
    local sliderBg=Instance.new("TextButton",row)
    sliderBg.Size=UDim2.new(1,0,0,6); sliderBg.Position=UDim2.new(0,0,0,22)
    sliderBg.BackgroundColor3=Theme.sliderBg; sliderBg.BorderSizePixel=0; sliderBg.ZIndex=12
    sliderBg.Text=""; sliderBg.AutoButtonColor=false
    Instance.new("UICorner",sliderBg).CornerRadius=UDim.new(1,0)
    local fill=Instance.new("Frame",sliderBg)
    fill.Size=UDim2.new((default-min)/(max-min),0,1,0)
    fill.BackgroundColor3=Theme.sliderFill; fill.BorderSizePixel=0; fill.ZIndex=13
    Instance.new("UICorner",fill).CornerRadius=UDim.new(1,0)
    local curVal=default
    local function updateFromX(x)
        local rel=math.clamp((x-sliderBg.AbsolutePosition.X)/sliderBg.AbsoluteSize.X,0,1)
        curVal=math.floor((min+rel*(max-min))/step+0.5)*step
        curVal=math.clamp(curVal,min,max)
        fill.Size=UDim2.new((curVal-min)/(max-min),0,1,0)
        valLbl.Text=tostring(curVal)
        if callback then callback(curVal) end
    end
    local dragging=false
    sliderBg.MouseButton1Down:Connect(function() dragging=true; updateFromX(Svc.UIS:GetMouseLocation().X) end)
    Svc.UIS.InputChanged:Connect(function(inp) if dragging and inp.UserInputType==Enum.UserInputType.MouseMovement then updateFromX(inp.Position.X) end end)
    Svc.UIS.InputEnded:Connect(function(inp) if inp.UserInputType==Enum.UserInputType.MouseButton1 then dragging=false end end)
    return row
end

local function makeCycleBtn(parent, name, options, default, callback, order)
    local row=Instance.new("Frame",parent)
    row.Size=UDim2.new(1,-16,0,32); row.BackgroundTransparency=1; row.ZIndex=12; row.LayoutOrder=order or 0
    local lbl=Instance.new("TextLabel",row)
    lbl.Size=UDim2.new(0.55,0,1,0); lbl.BackgroundTransparency=1; lbl.ZIndex=12
    lbl.Font=Enum.Font.Gotham; lbl.TextSize=12
    lbl.TextColor3=Theme.textMain; lbl.TextXAlignment=Enum.TextXAlignment.Left; lbl.Text=name
    local btn=Instance.new("TextButton",row)
    btn.Size=UDim2.new(0.4,0,0,22); btn.Position=UDim2.new(0.58,0,0.5,-11)
    btn.BackgroundColor3=Theme.bgCard; btn.BorderSizePixel=0; btn.ZIndex=13
    btn.Font=Enum.Font.GothamBold; btn.TextSize=11
    btn.TextColor3=Theme.white; btn.Text=options[default] or "?"
    Instance.new("UICorner",btn).CornerRadius=UDim.new(0,6)
    local btnStroke=Instance.new("UIStroke",btn); btnStroke.Color=Theme.border; btnStroke.Thickness=1
    local idx=default
    btn.MouseButton1Click:Connect(function() idx=idx%#options+1; btn.Text=options[idx]; if callback then callback(idx) end end)
    return row
end

local function makeKeybind(parent, actionName, order)
    local row=Instance.new("Frame",parent)
    row.Size=UDim2.new(1,-16,0,30); row.BackgroundTransparency=1; row.ZIndex=12; row.LayoutOrder=order or 0
    local lbl=Instance.new("TextLabel",row)
    lbl.Size=UDim2.new(0.6,0,1,0); lbl.BackgroundTransparency=1; lbl.ZIndex=12
    lbl.Font=Enum.Font.Gotham; lbl.TextSize=11
    lbl.TextColor3=Theme.textMain; lbl.TextXAlignment=Enum.TextXAlignment.Left
    lbl.Text=actionName
    local bindBtn=Instance.new("TextButton",row)
    bindBtn.Size=UDim2.new(0.35,0,0,22); bindBtn.Position=UDim2.new(0.63,0,0.5,-11)
    bindBtn.BackgroundColor3=Theme.bgCard; bindBtn.BorderSizePixel=0; bindBtn.ZIndex=13
    bindBtn.Font=Enum.Font.Code; bindBtn.TextSize=11
    bindBtn.TextColor3=Theme.white
    bindBtn.Text=keyName(Binds[actionName])
    bindBtn.AutoButtonColor=false
    Instance.new("UICorner",bindBtn).CornerRadius=UDim.new(0,5)
    local bs=Instance.new("UIStroke",bindBtn); bs.Color=Theme.border; bs.Thickness=1
    GUI.bindLabels[actionName]=bindBtn
    bindBtn.MouseButton1Click:Connect(function()
        if St.rebinding then return end
        St.rebinding=actionName
        bindBtn.Text="..."
        bindBtn.TextColor3=Theme.white
        bs.Color=Theme.white
        if Ref.rebindConn then pcall(function() Ref.rebindConn:Disconnect() end) end
        Ref.rebindConn=Svc.UIS.InputBegan:Connect(function(input,gp)
            if St.rebinding~=actionName then return end
            if input.UserInputType==Enum.UserInputType.Keyboard then
                local newKey=input.KeyCode
                if newKey==Enum.KeyCode.Escape then
                    bindBtn.Text=keyName(Binds[actionName])
                    bindBtn.TextColor3=Theme.white; bs.Color=Theme.border
                    St.rebinding=nil
                    pcall(function() Ref.rebindConn:Disconnect() end); Ref.rebindConn=nil
                    return
                end
                Binds[actionName]=newKey
                bindBtn.Text=keyName(newKey)
                bindBtn.TextColor3=Theme.white; bs.Color=Theme.border
                St.rebinding=nil
                pcall(function() Ref.rebindConn:Disconnect() end); Ref.rebindConn=nil
                print("[NexXit] "..actionName.." -> "..keyName(newKey))
            end
        end)
        task.delay(5, function()
            if St.rebinding==actionName then
                bindBtn.Text=keyName(Binds[actionName])
                bindBtn.TextColor3=Theme.white; bs.Color=Theme.border
                St.rebinding=nil
                if Ref.rebindConn then pcall(function() Ref.rebindConn:Disconnect() end); Ref.rebindConn=nil end
            end
        end)
    end)
    return row
end

-- Build Pages
local pages={}
for i,tabName in ipairs(tabs) do
    local page=Instance.new("ScrollingFrame",content)
    page.Name=tabName; page.Size=UDim2.new(1,-8,1,0); page.Position=UDim2.new(0,4,0,0)
    page.BackgroundTransparency=1; page.BorderSizePixel=0; page.ZIndex=11
    page.ScrollBarThickness=3; page.ScrollBarImageColor3=Theme.white
    page.CanvasSize=UDim2.new(0,0,0,0); page.AutomaticCanvasSize=Enum.AutomaticSize.Y
    page.Visible=(i==1)
    local layout=Instance.new("UIListLayout",page)
    layout.SortOrder=Enum.SortOrder.LayoutOrder; layout.Padding=UDim.new(0,4)
    pages[tabName]=page; GUI.tabPages[tabName]=page
    local tabBtn=Instance.new("TextButton",sidebar)
    tabBtn.Size=UDim2.new(1,0,0,32)
    tabBtn.BackgroundColor3=(i==1) and Theme.tabActive or Theme.tabIdle
    tabBtn.BorderSizePixel=0; tabBtn.ZIndex=12
    tabBtn.Font=Enum.Font.GothamBold; tabBtn.TextSize=11
    tabBtn.TextColor3=(i==1) and Theme.bg or Theme.textDim
    tabBtn.Text=tabName; tabBtn.TextXAlignment=Enum.TextXAlignment.Left; tabBtn.LayoutOrder=i
    Instance.new("UICorner",tabBtn).CornerRadius=UDim.new(0,6)
    local pad=Instance.new("UIPadding",tabBtn); pad.PaddingLeft=UDim.new(0,10)
    GUI.tabBtns[tabName]=tabBtn
    tabBtn.MouseButton1Click:Connect(function()
        GUI.activeTab=tabName
        for tn,tp in pairs(GUI.tabPages) do tp.Visible=(tn==tabName) end
        for tn,tb in pairs(GUI.tabBtns) do
            local active=(tn==tabName)
            tb.BackgroundColor3=active and Theme.tabActive or Theme.tabIdle
            tb.TextColor3=active and Theme.bg or Theme.textDim
        end
    end)
end

-- Combat Tab
do
    local pg=pages["Combat"]
    makeToggle(pg,"KillAura",false,function(v) setRunning(v) end,1)
    makeToggle(pg,"AimBot",Cfg.AIMBOT_ON,function(v) if v then startAimbot() else stopAimbot() end end,2)
    makeToggle(pg,"FOV Visual",Cfg.AIMBOT_VISUAL,function(v) Cfg.AIMBOT_VISUAL=v; toggleFOVCircle() end,3)
    makeToggle(pg,"AntiWallBang",Cfg.ANTI_WALLBANG,function(v) Cfg.ANTI_WALLBANG=v end,4)
    makeToggle(pg,"Skip Teammates",Cfg.SKIP_TEAMMATES,function(v) Cfg.SKIP_TEAMMATES=v end,5)
    makeToggle(pg,"Auto Reload",Cfg.AUTO_RELOAD,function(v) Cfg.AUTO_RELOAD=v end,6)
    makeToggle(pg,"Skip ForceField",Cfg.SKIP_FORCEFIELD,function(v) Cfg.SKIP_FORCEFIELD=v end,7)
    makeToggle(pg,"Only Nearest",Cfg.ONLY_NEAREST,function(v) Cfg.ONLY_NEAREST=v end,8)
    makeToggle(pg,"FOV Check",Cfg.AIM_FOV_ON,function(v) Cfg.AIM_FOV_ON=v end,9)
    makeSlider(pg,"FOV",30,360,Cfg.AIM_FOV,function(v) Cfg.AIM_FOV=v end,10,5)
    makeSlider(pg,"AimBot FOV",10,300,Cfg.AIMBOT_FOV,function(v) Cfg.AIMBOT_FOV=v; if fovCircle then fovCircle.Radius = v end end,11,5)
    makeSlider(pg,"AimBot Smooth",1,100,math.floor(Cfg.AIMBOT_SMOOTH*100),function(v) Cfg.AIMBOT_SMOOTH=v/100 end,12,1)
    makeSlider(pg,"Max Distance",50,600,Cfg.MAX_DIST,function(v) Cfg.MAX_DIST=v end,13,10)
    makeSlider(pg,"Ammo",1,50,Cfg.AMMO_VALUE,function(v) Cfg.AMMO_VALUE=v end,14,1)
    makeSlider(pg,"Fire Delay (ms)",1,300,math.floor(Cfg.SHOT_DELAY*1000),function(v) Cfg.SHOT_DELAY=v/1000 end,15,1)
    makeSlider(pg,"AWB Delay (ms)",0,500,math.floor(Cfg.AWB_EXTRA_DELAY*1000),function(v) Cfg.AWB_EXTRA_DELAY=v/1000 end,16,10)
end

-- Movement Tab
do
    local pg=pages["Movement"]
    makeToggle(pg,"Bunny Hop",false,function(v) if v then startBhop() else stopBhop() end end,1)
    makeSlider(pg,"Max Speed",10,80,Cfg.BHOP_MAX,function(v) Cfg.BHOP_MAX=v end,2,1)
    makeSlider(pg,"Acceleration",0.1,5,Cfg.BHOP_ACCEL,function(v) Cfg.BHOP_ACCEL=v end,3,0.1)
    makeSlider(pg,"Strafe",0.2,3,Cfg.BHOP_STRAFE,function(v) Cfg.BHOP_STRAFE=v end,4,0.1)
    makeSlider(pg,"Jump Distance",5,50,Cfg.BHOP_JUMP_DIST,function(v) Cfg.BHOP_JUMP_DIST=v end,5,1)
end

-- Render Tab
do
    local pg=pages["Render"]
    makeToggle(pg,"ESP",false,function(v) if v then startESP() else stopESP() end end,1)
    makeToggle(pg,"FP Camera",false,function(v) toggleFP() end,2)
    makeToggle(pg,"Cam Noclip",Cfg.CAM_NOCLIP,function(v) if v then startCamNoclip() else stopCamNoclip() end end,3)
    makeSlider(pg,"ESP Distance",50,800,Cfg.ESP_MAX,function(v) Cfg.ESP_MAX=v end,4,10)
    makeSlider(pg,"FP Sensitivity",5,100,math.floor(Cfg.FP_SENS*100),function(v) Cfg.FP_SENS=v/100 end,5,1)
end

-- Player Tab
do
    local pg=pages["Player"]
    local aaOpts={}
    for i=0,AA_COUNT-1 do aaOpts[i+1]=AA_NAMES[i] end
    makeCycleBtn(pg,"Anti-Aim",aaOpts,Cfg.AA_MODE+1,function(idx) Cfg.AA_MODE=idx-1; applyAA() end,1)
    makeToggle(pg,"Emote AA",false,function(v) if v then startEmoteAA() else stopEmoteAA() end end,2)
    makeSlider(pg,"AA Speed",5,60,Cfg.AA_SPEED,function(v) Cfg.AA_SPEED=v end,3,1)
    makeSlider(pg,"AA Jitter",10,180,Cfg.AA_JITTER,function(v) Cfg.AA_JITTER=v end,4,5)
    makeSlider(pg,"AA Sway",5,90,Cfg.AA_SWAY,function(v) Cfg.AA_SWAY=v end,5,5)
    makeSlider(pg,"Emote Speed",1,10,Cfg.AA_EMOTE_SPEED,function(v) Cfg.AA_EMOTE_SPEED=v end,6,1)
end

-- Weapon Tab
do
    local pg=pages["Weapon"]
    makeToggle(pg,"Minecraft Sword",Cfg.MINECRAFT_SWORD,function(v) Cfg.MINECRAFT_SWORD=v; showViewmodel(v) end,1)
    makeSlider(pg,"VM X",-10,10,Cfg.VM_X,function(v) Cfg.VM_X=v end,2,0.1)
    makeSlider(pg,"VM Y",-10,10,Cfg.VM_Y,function(v) Cfg.VM_Y=v end,3,0.1)
    makeSlider(pg,"VM Z",-15,5,Cfg.VM_Z,function(v) Cfg.VM_Z=v end,4,0.1)
    makeSlider(pg,"VM RX",-180,180,Cfg.VM_RX,function(v) Cfg.VM_RX=v end,5,5)
    makeSlider(pg,"VM RY",-180,360,Cfg.VM_RY,function(v) Cfg.VM_RY=v end,6,5)
    makeSlider(pg,"VM RZ",-180,180,Cfg.VM_RZ,function(v) Cfg.VM_RZ=v end,7,5)
    makeSlider(pg,"Swing Pivot X",-5,5,Cfg.VM_SWING_X,function(v) Cfg.VM_SWING_X=v end,8,0.1)
    makeSlider(pg,"Swing Pivot Y",-5,5,Cfg.VM_SWING_Y,function(v) Cfg.VM_SWING_Y=v end,9,0.1)
    makeSlider(pg,"Swing Pivot Z",-5,5,Cfg.VM_SWING_Z,function(v) Cfg.VM_SWING_Z=v end,10,0.1)
    local swRow=Instance.new("Frame",pg)
    swRow.Size=UDim2.new(1,-16,0,32); swRow.BackgroundTransparency=1; swRow.ZIndex=12; swRow.LayoutOrder=11
    local swBtn=Instance.new("TextButton",swRow)
    swBtn.Size=UDim2.new(1,0,0,26); swBtn.Position=UDim2.new(0,0,0.5,-13)
    swBtn.BackgroundColor3=Theme.bgCard; swBtn.BorderSizePixel=0; swBtn.ZIndex=13
    swBtn.Font=Enum.Font.GothamBold; swBtn.TextSize=11
    swBtn.TextColor3=Theme.white; swBtn.Text="TEST SWING"
    Instance.new("UICorner",swBtn).CornerRadius=UDim.new(0,6)
    local ss=Instance.new("UIStroke",swBtn); ss.Color=Theme.border; ss.Thickness=1
    swBtn.MouseButton1Click:Connect(function() triggerSwing() end)
end

-- Misc Tab
do
    local pg=pages["Misc"]
    makeKeybind(pg,"KillAura",1)
    makeKeybind(pg,"ESP",2)
    makeKeybind(pg,"Jitter",3)
    makeKeybind(pg,"Bhop",4)
    makeKeybind(pg,"FP",5)
    makeKeybind(pg,"Reload",6)
    makeKeybind(pg,"EmoteAA",7)
    makeKeybind(pg,"CamNoclip",8)
    makeKeybind(pg,"ToggleGUI",9)
    makeKeybind(pg,"AimBot",10)

    local svRow=Instance.new("Frame",pg)
    svRow.Size=UDim2.new(1,-16,0,32); svRow.BackgroundTransparency=1; svRow.ZIndex=12; svRow.LayoutOrder=50
    local svBtn=Instance.new("TextButton",svRow)
    svBtn.Size=UDim2.new(1,0,0,26); svBtn.Position=UDim2.new(0,0,0.5,-13)
    svBtn.BackgroundColor3=Theme.bgCard; svBtn.BorderSizePixel=0; svBtn.ZIndex=13
    svBtn.Font=Enum.Font.GothamBold; svBtn.TextSize=11
    svBtn.TextColor3=Theme.white; svBtn.Text="SAVE CONFIG"
    Instance.new("UICorner",svBtn).CornerRadius=UDim.new(0,6)
    local svs=Instance.new("UIStroke",svBtn); svs.Color=Theme.border; svs.Thickness=1
    svBtn.MouseButton1Click:Connect(function() saveConfig() end)

    local dRow=Instance.new("Frame",pg)
    dRow.Size=UDim2.new(1,-16,0,32); dRow.BackgroundTransparency=1; dRow.ZIndex=12; dRow.LayoutOrder=99
    local dBtn=Instance.new("TextButton",dRow)
    dBtn.Size=UDim2.new(1,0,0,26); dBtn.Position=UDim2.new(0,0,0.5,-13)
    dBtn.BackgroundColor3=Color3.fromRGB(60,10,10); dBtn.BorderSizePixel=0; dBtn.ZIndex=13
    dBtn.Font=Enum.Font.GothamBold; dBtn.TextSize=11
    dBtn.TextColor3=Theme.red; dBtn.Text="DESTROY"
    Instance.new("UICorner",dBtn).CornerRadius=UDim.new(0,6)
    local ds=Instance.new("UIStroke",dBtn); ds.Color=Color3.fromRGB(100,20,20); ds.Thickness=1
    dBtn.MouseButton1Click:Connect(function()
        if getgenv().NexXitPainel and getgenv().NexXitPainel.destroy then getgenv().NexXitPainel.destroy() end
    end)
end

if Cfg.MINECRAFT_SWORD then showViewmodel(true) end
if Cfg.CAM_NOCLIP then startCamNoclip() end

-- ============ KEYBINDS ============
Ref.connections[#Ref.connections+1]=Svc.UIS.InputBegan:Connect(function(input,gp)
    if gp then return end
    if St.rebinding then return end
    if input.UserInputType~=Enum.UserInputType.Keyboard then return end
    local k=input.KeyCode
    if      k==Binds.ToggleGUI then
        Cfg.GUI_VISIBLE=not Cfg.GUI_VISIBLE
        if Ref.mainWindow then Ref.mainWindow.Visible=Cfg.GUI_VISIBLE end
    elseif  k==Binds.KillAura then setRunning(not St.running)
    elseif  k==Binds.ESP then toggleESP()
    elseif  k==Binds.Jitter then toggleJitter()
    elseif  k==Binds.Bhop then toggleBhop()
    elseif  k==Binds.FP then toggleFP()
    elseif  k==Binds.Reload then fireReload("key")
    elseif  k==Binds.EmoteAA then toggleEmoteAA()
    elseif  k==Binds.CamNoclip then toggleCamNoclip()
    elseif  k==Binds.AimBot then toggleAimbot()
    end
end)

-- Atualiza a posição do círculo FOV quando o mouse se move
Ref.connections[#Ref.connections+1]=Svc.UIS.InputChanged:Connect(function(input,gp)
    if gp then return end
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        if Cfg.AIMBOT_ON and Cfg.AIMBOT_VISUAL then
            updateFOVCirclePosition()
        end
    end
end)

-- ============ PLAYER EVENTS ============
Ref.connections[#Ref.connections+1]=me.CharacterAdded:Connect(function(char)
    task.wait(0.4)
    St.currentSpeed=Cfg.BHOP_START; St.wasGrounded=true; Ref.playerFFState={}
    St.shotCounter=0; St.lastShotAt=tick(); St.lastShotPlr=nil; St.sameTargetCount=0
    St.swingOffset=0; St.swingToken=St.swingToken+1; St.bhopSmoothVel=Cfg.BHOP_START
    Ref.aaEmoteTrack=nil; Ref.aaEmoteAnim=nil; St.aaEmoteChar=nil; St.aaAnimateWasDisabled=false
    Ref.glockLiverCache=nil; St.glockLiverHidden=false
    if Ref.aaEmoteWatch then pcall(function() task.cancel(Ref.aaEmoteWatch) end); Ref.aaEmoteWatch=nil end
    if Ref.emoteAAConn then pcall(function() Ref.emoteAAConn:Disconnect() end); Ref.emoteAAConn=nil end
    if Cfg.ESP_ON then for p in pairs(Ref.espDraws) do pcall(function() Ref.espDraws[p]=nil end) end end
    if Cfg.BHOP_ON then startBhop() end
    if Cfg.AA_MODE~=0 then applyAA() end
    if Cfg.EMOTE_AA_ON then task.delay(0.5,function() startEmoteAA() end) end
    if Cfg.MINECRAFT_SWORD then task.delay(0.3,function() if Ref.vpModel and not Ref.vpConn then startVPSync() end end) end
    if Cfg.FP_ON then task.delay(0.3,function() setLocalTransp(char,1); setGlockLiverVisible(false); initFPAngles() end) end
    if St.running then setRunning(true) end
    if Cfg.AIMBOT_ON then task.delay(0.5, function() startAimbot() end) end
end)

Ref.connections[#Ref.connections+1]=Svc.Players.PlayerRemoving:Connect(function(plr)
    Ref.playerFFState[plr]=nil; Ref.deadCache[plr]=nil
    getgenv()._tsUnwatchHealth(plr)
    if St.lastShotPlr==plr then St.lastShotPlr=nil; St.sameTargetCount=0 end
end)

Ref.connections[#Ref.connections+1]=Svc.Players.PlayerAdded:Connect(function(plr) watchHealth(plr) end)
for _,p in ipairs(Svc.Players:GetPlayers()) do watchHealth(p) end

-- ============ INIT ============
startFFWatcher()
startACBypass()
startWatchdog()
St.lastShotAt=tick()

-- Cria o círculo FOV se o AimBot estiver ativado
if Cfg.AIMBOT_ON and Cfg.AIMBOT_VISUAL then
    createFOVCircle()
end

-- ============ API ============
getgenv().NexXitPainel={
    start=function() setRunning(true) end,
    stop=function() setRunning(false) end,
    bhop=toggleBhop, fp=toggleFP, jitter=toggleJitter, esp=toggleESP,
    sword=toggleSword, camnoclip=toggleCamNoclip,
    aimbot=toggleAimbot,
    aa=function(m) if m then Cfg.AA_MODE=math.clamp(tonumber(m) or 0,0,AA_COUNT-1) end; applyAA() end,
    reload=function() fireReload("api") end,
    swing=function() triggerSwing() end,
    setSens=function(s) Cfg.FP_SENS=math.max(0.05,tonumber(s) or 0.35) end,
    emote=function(id,speed)
        if id then Cfg.AA_EMOTE_ID=id; St.resolvedEmoteAnimId=nil; St.emoteResolving=false end
        if speed then Cfg.AA_EMOTE_SPEED=tonumber(speed) or 1 end; startEmoteAA()
    end,
    stopEmote=stopEmoteAA,
    setHash=function(h) Cfg.HIT_HASH=tostring(h) end,
    setReload=function(h) Cfg.RELOAD_HASH=tostring(h) end,
    setRange=function(d) Cfg.MAX_DIST=d or 300 end,
    setAmmo=function(n) Cfg.AMMO_VALUE=n or 13 end,
    saveCfg=saveConfig, loadCfg=loadConfig,
    isTeammate=function(plr) return getgenv()._tsIsTeammate(plr) end,
    rebind=function(action,key)
        if Binds[action] and key then
            local ok,kc=pcall(function() return Enum.KeyCode[key] end)
            if ok then Binds[action]=kc
                if GUI.bindLabels[action] then GUI.bindLabels[action].Text=keyName(kc) end
            end
        end
    end,
    toggleGUI=function()
        Cfg.GUI_VISIBLE=not Cfg.GUI_VISIBLE
        if Ref.mainWindow then Ref.mainWindow.Visible=Cfg.GUI_VISIBLE end
    end,
    hideGlock=function() setGlockLiverVisible(false) end,
    showGlock=function() setGlockLiverVisible(true) end,
    recomputeSwingPivot=function()
        if Ref.vpModel then Ref.vmHandleOffset = computeHandleOffset and nil or nil end
    end,
    destroy=function()
        St.running=false; St.combatToken=St.combatToken+1; St.swingToken=St.swingToken+1; St.swingOffset=0
        if Cfg.FP_ON then stopFPCamera(); setLocalTransp(myChar(), 0); setGlockLiverVisible(true) end
        stopEmoteAA(); stopBhop(); stopAA(); stopESP(); stopCamNoclip(); stopAimbot()
        Cfg.FP_ON=false; Cfg.MINECRAFT_SWORD=false
        showViewmodel(false)
        destroyFOVCircle()
        if Ref.fpCamConn then pcall(function() Ref.fpCamConn:Disconnect() end); Ref.fpCamConn=nil end
        if Ref.vpConn then pcall(function() Ref.vpConn:Disconnect() end); Ref.vpConn=nil end
        if Ref.vpModel then pcall(function() Ref.vpModel:Destroy() end); Ref.vpModel=nil end
        if Ref.vpFrame then pcall(function() Ref.vpFrame:Destroy() end); Ref.vpFrame=nil end
        if Ref.acBypassConn then pcall(function() Ref.acBypassConn:Disconnect() end) end
        if Ref.watchdogConn then pcall(function() Ref.watchdogConn:Disconnect() end) end
        if Ref.rebindConn then pcall(function() Ref.rebindConn:Disconnect() end) end
        for _,c in ipairs(Ref.connections) do pcall(function() c:Disconnect() end) end
        for _,c in ipairs(Ref.teamConns) do pcall(function() c:Disconnect() end) end
        for plr in pairs(Ref.healthConns) do getgenv()._tsUnwatchHealth(plr) end
        if Ref.gui then Ref.gui:Destroy() end
        Ref.glockLiverCache=nil; Ref.vmHandleOffset=nil
        getgenv().NexXitPainel=nil
    end,
}

print("[NexXit Painel RBLX] v1.0 Made By NexXit | \\ to toggle GUI")
print("[NexXit] ready | rebind keys in Misc tab")
print("[NexXit] AimBot: segure LALT para mirar na cabeça")
