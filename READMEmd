-- MOBILE FULL SCRIPT (ESP, AIMBOT, GUI, FOV, SAVE)
local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer
local HttpService = game:GetService("HttpService")

local configFile = "mobile_config.json"
local config = {
    esp = true,
    aimbot = true,
    teamCheck = true,
    fovSize = 150
}

-- Tenta carregar configurações
if isfile and isfile(configFile) then
    local success, result = pcall(function()
        return HttpService:JSONDecode(readfile(configFile))
    end)
    if success then
        for k, v in pairs(result) do config[k] = v end
    end
end

local function saveConfig()
    if writefile then
        writefile(configFile, HttpService:JSONEncode(config))
    end
end

-- FOV Circle
local fov = Drawing.new("Circle")
fov.Color = Color3.fromRGB(255, 255, 255)
fov.Thickness = 1
fov.Filled = false
fov.Visible = true
fov.Radius = config.fovSize
fov.Position = Vector2.new(0, 0)

-- GUI Setup
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "MobileInterface"
screenGui.ResetOnSpawn = false
screenGui.IgnoreGuiInset = true
screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local function createButton(text, yPos, callback)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0.25, 0, 0.07, 0)
    button.Position = UDim2.new(0.02, 0, yPos, 0)
    button.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
    button.TextColor3 = Color3.new(1, 1, 1)
    button.Text = text
    button.TextScaled = true
    button.Parent = screenGui
    button.MouseButton1Click:Connect(callback)
end

createButton("Toggle ESP", 0.1, function()
    config.esp = not config.esp
    saveConfig()
end)

createButton("Toggle Aimbot", 0.18, function()
    config.aimbot = not config.aimbot
    saveConfig()
end)

createButton("Team Check", 0.26, function()
    config.teamCheck = not config.teamCheck
    saveConfig()
end)

createButton("FOV +", 0.34, function()
    config.fovSize = config.fovSize + 10
    fov.Radius = config.fovSize
    saveConfig()
end)

createButton("FOV -", 0.42, function()
    config.fovSize = math.max(20, config.fovSize - 10)
    fov.Radius = config.fovSize
    saveConfig()
end)

-- Aimbot botão (pressionar e segurar)
local aimBtn = Instance.new("TextButton", screenGui)
aimBtn.Size = UDim2.new(0.18, 0, 0.12, 0)
aimBtn.Position = UDim2.new(0.8, 0, 0.8, 0)
aimBtn.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
aimBtn.TextColor3 = Color3.new(1, 1, 1)
aimBtn.Text = "AIM"
aimBtn.TextScaled = true

local aiming = false
aimBtn.MouseButton1Down:Connect(function()
    aiming = true
end)
aimBtn.MouseButton1Up:Connect(function()
    aiming = false
end)

-- ESP Setup
local function createESP(player)
    local char = player.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp or hrp:FindFirstChild("ESPLabel") then return end

    local gui = Instance.new("BillboardGui", hrp)
    gui.Name = "ESPLabel"
    gui.Size = UDim2.new(0, 200, 0, 40)
    gui.StudsOffset = Vector3.new(0, 3, 0)
    gui.AlwaysOnTop = true

    local label = Instance.new("TextLabel", gui)
    label.Size = UDim2.new(1, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.TextColor3 = Color3.new(1, 1, 1)
    label.TextStrokeTransparency = 0.5
    label.TextScaled = true
    label.Text = player.Name
end

local function updateESP(player)
    local char = player.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    local gui = hrp and hrp:FindFirstChild("ESPLabel")
    if gui and gui:FindFirstChildWhichIsA("TextLabel") then
        local dist = math.floor((hrp.Position - Camera.CFrame.Position).Magnitude)
        gui.TextLabel.Text = player.Name .. " [" .. dist .. "m]"
    end
end

-- Aimbot target
local function getClosestTarget()
    local mousePos = UIS:GetMouseLocation()
    local closest, shortest = nil, config.fovSize
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("Head") then
            if config.teamCheck and p.Team == LocalPlayer.Team then continue end
            local head = p.Character.Head
            local screenPos, visible = Camera:WorldToViewportPoint(head.Position)
            local dist = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude
            if visible and dist < shortest then
                shortest = dist
                closest = head.Position
            end
        end
    end
    return closest
end

-- Loop Principal
RunService.RenderStepped:Connect(function()
    fov.Position = UIS:GetMouseLocation()

    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and config.esp then
            createESP(p)
            updateESP(p)
        end
    end

    if aiming and config.aimbot then
        local target = getClosestTarget()
        if target then
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, target)
        end
    end
end)
