-- xit hub v3
if getgenv().xit_hub and type(getgenv().xit_hub.destroy)=="function" then
    pcall(getgenv().xit_hub.destroy)
    task.wait(0.15)
end

local Svc = {
    Players = game:GetService("Players"),
    RS = game:GetService("ReplicatedStorage"),
    UIS = game:GetService("UserInputService"),
    RunService = game:GetService("RunService"),
    Http = game:GetService("HttpService"),
}
local me = Svc.Players.LocalPlayer

local Cfg = {
    MAX_DIST = 300,
    AIM_FOV = 360,
    AIM_FOV_ON = true,
    SKIP_TEAMMATES = true,
    SKIP_FORCEFIELD = true,
    MINECRAFT_SWORD = true,
    ANTI_WALLBANG = true,
    ESP_ON = false,
    FP_ON = false,
    AA_MODE = 0,
    BHOP_ON = false,
    CAM_NOCLIP = false,
}

local Binds = {
    KillAura = Enum.KeyCode.Y,
    ESP = Enum.KeyCode.N,
    Bhop = Enum.KeyCode.B,
    FP = Enum.KeyCode.F,
    ToggleGUI = Enum.KeyCode.BackSlash,
}

local St = {running = false}
local Ref = {connections = {}}

function getRoot(char)
    return char and (char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("Torso"))
end

function getHum(char)
    return char and char:FindFirstChildOfClass("Humanoid")
end

function myChar()
    return me.Character
end

function isAlive(char)
    if not char or not char.Parent then return false end
    local hum = getHum(char)
    if not hum then return false end
    if hum.Health <= 0 then return false end
    return true
end

function hasFF(char)
    return char and char:FindFirstChildOfClass("ForceField") ~= nil
end

function getHitParts(char)
    if not char then return {} end
    local out = {}
    for _, n in ipairs({"Head", "UpperTorso", "Torso", "HumanoidRootPart"}) do
        local p = char:FindFirstChild(n)
        if p and p:IsA("BasePart") then
            out[#out + 1] = p
        end
    end
    return out
end

function getTargets()
    local targets = {}
    for _, plr in ipairs(Svc.Players:GetPlayers()) do
        if plr ~= me then
            local char = plr.Character
            if char and isAlive(char) then
                if not hasFF(char) then
                    local root = getRoot(char)
                    if root then
                        local origin = myChar() and myChar().HumanoidRootPart
                        if origin then
                            local dist = (origin.Position - root.Position).Magnitude
                            if dist <= Cfg.MAX_DIST then
                                table.insert(targets, {plr = plr, char = char, dist = dist})
                            end
                        end
                    end
                end
            end
        end
    end
    table.sort(targets, function(a, b) return a.dist < b.dist end)
    return targets
end

function checkWallbang(origin, target)
    if not Cfg.ANTI_WALLBANG then return true end
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Exclude
    params.FilterDescendantsInstances = {myChar()}
    params.IgnoreWater = true
    local hit = workspace:Raycast(origin, (target - origin).Unit * (target - origin).Magnitude, params)
    if not hit then return true end
    return false
end

function shoot(target)
    if not target or not target.char then return end
    local root = getRoot(target.char)
    if not root then return end
    
    local origin = myChar() and myChar().HumanoidRootPart
    if not origin then return end
    
    local originPos = origin.Position
    local targetPos = root.Position
    
    if not checkWallbang(originPos, targetPos) then return end
    
    local fire = Instance.new("RemoteEvent")
    fire.Name = "SendRequest"
    fire.Parent = Svc.RS
    
    local args = {
        ["action"] = "fire",
        ["target"] = me.UserId,
        ["hit"] = target.plr.UserId,
    }
    
    pcall(function()
        fire:FireServer(args)
    end)
end

function KillAuraLoop()
    while St.running do
        if Cfg.MINECRAFT_SWORD then
            local targets = getTargets()
            if #targets > 0 then
                shoot(targets[1])
                task.wait(0.1)
            end
        end
        task.wait(0.05)
    end
end

function toggleKillAura()
    St.running = not St.running
    if St.running then
        task.spawn(KillAuraLoop)
    end
end

function toggleESP()
    Cfg.ESP_ON = not Cfg.ESP_ON
    if Cfg.ESP_ON then
        for _, plr in ipairs(Svc.Players:GetPlayers()) do
            if plr ~= me then
                local char = plr.Character
                if char then
                    for _, part in ipairs(char:GetDescendants()) do
                        if part:IsA("BasePart") then
                            local box = Instance.new("BoxHandleAdornment")
                            box.Size = part.Size + Vector3.new(1, 1, 1)
                            box.CFrame = part.CFrame
                            box.Color3 = Color3.fromRGB(255, 0, 0)
                            box.Transparency = 0.5
                            box.AlwaysOnTop = true
                            box.ZIndex = 5
                            box.Adornee = part
                            box.Parent = char
                            table.insert(Ref.connections, box)
                        end
                    end
                end
            end
        end
    else
        for _, conn in ipairs(Ref.connections) do
            pcall(function() conn:Destroy() end)
        end
        Ref.connections = {}
    end
end

function toggleFP()
    Cfg.FP_ON = not Cfg.FP_ON
end

function toggleBhop()
    Cfg.BHOP_ON = not Cfg.BHOP_ON
end

function toggleGUI()
    local gui = Ref.gui
    if gui then
        gui.Visible = not gui.Visible
    end
end

function createGUI()
    local screen = Instance.new("ScreenGui")
    screen.Name = "XitHubGUI"
    screen.Parent = me.PlayerGui
    
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(0, 300, 0, 400)
    frame.Position = UDim2.new(0.5, -150, 0.5, -200)
    frame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
    frame.BorderSizePixel = 0
    frame.Parent = screen
    
    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0, 40)
    title.BackgroundColor3 = Color3.fromRGB(30, 30, 50)
    title.Text = "XIT HUB v3"
    title.TextColor3 = Color3.fromRGB(255, 255, 255)
    title.Font = Enum.Font.GothamBold
    title.TextSize = 20
    title.Parent = frame
    
    local function createButton(text, y, callback)
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0.9, 0, 0, 35)
        btn.Position = UDim2.new(0.05, 0, 0, y)
        btn.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
        btn.Text = text
        btn.TextColor3 = Color3.fromRGB(255, 255, 255)
        btn.Font = Enum.Font.Gotham
        btn.TextSize = 14
        btn.Parent = frame
        btn.MouseButton1Click:Connect(callback)
        return btn
    end
    
    local y = 50
    createButton("Kill Aura [Y]", y, function()
        toggleKillAura()
    end)
    y = y + 45
    
    createButton("ESP [N]", y, function()
        toggleESP()
    end)
    y = y + 45
    
    createButton("Bhop [B]", y, function()
        toggleBhop()
    end)
    y = y + 45
    
    createButton("First Person [F]", y, function()
        toggleFP()
    end)
    y = y + 45
    
    local close = Instance.new("TextButton")
    close.Size = UDim2.new(0, 30, 0, 30)
    close.Position = UDim2.new(1, -35, 0, 5)
    close.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
    close.Text = "X"
    close.TextColor3 = Color3.fromRGB(255, 255, 255)
    close.Font = Enum.Font.GothamBold
    close.TextSize = 16
    close.Parent = frame
    close.MouseButton1Click:Connect(function()
        screen:Destroy()
        Ref.gui = nil
    end)
    
    Ref.gui = screen
end

function setupKeybinds()
    Svc.UIS.InputBegan:Connect(function(input, gameProcessed)
        if gameProcessed then return end
        if input.KeyCode == Binds.KillAura then
            toggleKillAura()
        elseif input.KeyCode == Binds.ESP then
            toggleESP()
        elseif input.KeyCode == Binds.Bhop then
            toggleBhop()
        elseif input.KeyCode == Binds.FP then
            toggleFP()
        elseif input.KeyCode == Binds.ToggleGUI then
            toggleGUI()
        end
    end)
end

function BhopLoop()
    while true do
        if Cfg.BHOP_ON then
            local char = myChar()
            if char then
                local hum = getHum(char)
                local root = getRoot(char)
                if hum and root then
                    if hum:GetState() == Enum.HumanoidStateType.Landed then
                        hum.Jump = true
                        task.wait(0.05)
                    end
                end
            end
        end
        task.wait(0.05)
    end
end

function FPLoop()
    while true do
        if Cfg.FP_ON then
            local cam = workspace.CurrentCamera
            local char = myChar()
            if cam and char then
                local head = char:FindFirstChild("Head")
                if head then
                    cam.CFrame = CFrame.new(head.Position + Vector3.new(0, 0.15, 0))
                end
            end
        end
        task.wait()
    end
end

-- Inicialização
task.spawn(BhopLoop)
task.spawn(FPLoop)
task.spawn(function()
    while true do
        if Cfg.CAM_NOCLIP and myChar() then
            local root = getRoot(myChar())
            if root then
                root.CanCollide = false
            end
        end
        task.wait(0.5)
    end
end)

createGUI()
setupKeybinds()

print("[XIT HUB v3] Carregado com sucesso!")
print("Y - Kill Aura | N - ESP | B - Bhop | F - First Person | \\ - GUI")

getgenv().xit_hub = {
    destroy = function()
        if Ref.gui then Ref.gui:Destroy() end
        for _, conn in ipairs(Ref.connections) do
            pcall(function() conn:Destroy() end)
        end
        St.running = false
    end
}
