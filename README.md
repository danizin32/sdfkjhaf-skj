-- ✅ Script completo com ESP, Aimbot, FOV, GUI para MOBILE
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer
local HttpService = game:GetService("HttpService")

-- Configuração inicial e tentativa de carregar arquivo
local config = {
    esp = true,
    aimbot = true,
    teamCheck = true,
    fovSize = 120
}

local configFile = "mobile_config.json"
if isfile and isfile(configFile) then
    local success, data = pcall(function()
        return HttpService:JSONDecode(readfile(configFile))
    end)
    if success then
        for k, v in pairs(data) do config[k] = v end
    end
end

local function saveConfig()
    if writefile then
        writefile(configFile, HttpService:JSONEncode(config))
    end
end

-- Interface
local playerGui = LocalPlayer:WaitForChild("PlayerGui", 5)

if playerGui:FindFirstChild("MobileFullGUI") then
    playerGui:FindFirstChild("MobileFullGUI"):Destroy()
end

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "MobileFullGUI"
screenGui.ResetOnSpawn = false
screenGui.IgnoreGuiInset = true
screenGui.Parent = playerGui

local function createButton(text, yPos, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0.28, 0, 0.07, 0)
    btn.Position = UDim2.new(0.02, 0, yPos, 0)
    btn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Text = text
    btn.TextScaled = true
    btn.Parent = screenGui
    btn.MouseButton1Click:Connect(callback)
end

createButton("ESP", 0.05, function()
    config.esp = not config.esp
    saveConfig()
end)

createButton("Aimbot", 0.13, function()
    config.aimbot = not config.aimbot
    saveConfig()
end)

createButton("Team Check", 0.21, function()
    config.teamCheck = not config.teamCheck
    saveConfig()
end)

createButton("FOV +", 0.29, function()
    config.fovSize = config.fovSize + 10
    fov.Radius = config.fovSize
    saveConfig()
end)

createButton("FOV -", 0.37, function()
    config.fovSize = math.max(10, config.fovSize - 10)
    fov.Radius = config.fovSize
    saveConfig()
end)

-- Botão para ativar Aimbot (toque e segure)
local aimButton = Instance.new("TextButton", screenGui)
aimButton.Size = UDim2.new(0.18, 0, 0.12, 0)
aimButton.Position = UDim2.new(0.78, 0, 0.83, 0)
aimButton.BackgroundColor3 = Color3.fromRGB(90, 0, 0)
aimButton.Text = "AIM"
aimButton.TextColor3 = Color3.new(1, 1, 1)
aimButton.TextScaled = true

local aiming = false
aimButton.MouseButton1Down:Connect(function() aiming = true end)
aimButton.MouseButton1Up:Connect(function() aiming = false end)

-- FOV circle
local fov = Drawing.new("Circle")
fov.Color = Color3.fromRGB(255, 255, 255)
fov.Thickness = 1
fov.Filled = false
fov.Visible = true
fov.Radius = config.fovSize
fov.Position = UserInputService:GetMouseLocation()

-- ESP
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
    local mousePos = UserInputService:GetMouseLocation()
    local closest, shortest = nil, config.fovSize
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("Head") then
            if config.teamCheck and p.Team == LocalPlayer.Team then continue end
            local head = p.Character.Head
            local screenPos, onScreen = Camera:WorldToViewportPoint(head.Position)
            local dist = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude
            if onScreen and dist < shortest then
                shortest = dist
                closest = head.Position
            end
        end
    end
    return closest
end

-- Loop Principal
RunService.RenderStepped:Connect(function()
    fov.Position = UserInputService:GetMouseLocation()

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
