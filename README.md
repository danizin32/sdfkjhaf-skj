-- Mobile Aimbot + ESP (Toque)
local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer
local HttpService = game:GetService("HttpService")

local configFile = "mobile_aimbot_config.json"
local config = {
    esp = true,
    aimbot = true,
    teamCheck = true,
    fovSize = 150
}

-- Carregar config se existir
if isfile and isfile(configFile) then
    local ok, data = pcall(function()
        return HttpService:JSONDecode(readfile(configFile))
    end)
    if ok then
        for k, v in pairs(data) do config[k] = v end
    end
end

local function saveConfig()
    if writefile then
        writefile(configFile, HttpService:JSONEncode(config))
    end
end

-- FOV Circle
local fov = Drawing.new("Circle")
fov.Color = Color3.new(1, 1, 1)
fov.Thickness = 1
fov.Filled = false
fov.Visible = true
fov.Radius = config.fovSize

-- GUI Touch Menu
local screenGui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
screenGui.Name = "MobileAimbotMenu"
screenGui.ResetOnSpawn = false

local function createButton(text, pos, callback)
    local btn = Instance.new("TextButton", screenGui)
    btn.Size = UDim2.new(0, 140, 0, 40)
    btn.Position = pos
    btn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Text = text
    btn.TextScaled = true
    btn.MouseButton1Click:Connect(callback)
    return btn
end

createButton("Toggle ESP", UDim2.new(0, 10, 0, 100), function()
    config.esp = not config.esp
    saveConfig()
end)

createButton("Toggle Aimbot", UDim2.new(0, 10, 0, 150), function()
    config.aimbot = not config.aimbot
    saveConfig()
end)

createButton("Team Check", UDim2.new(0, 10, 0, 200), function()
    config.teamCheck = not config.teamCheck
    saveConfig()
end)

createButton("FOV +", UDim2.new(0, 10, 0, 250), function()
    config.fovSize = config.fovSize + 10
    fov.Radius = config.fovSize
    saveConfig()
end)

createButton("FOV -", UDim2.new(0, 10, 0, 300), function()
    config.fovSize = math.max(10, config.fovSize - 10)
    fov.Radius = config.fovSize
    saveConfig()
end)

-- ESP
local function createESP(player)
    local char = player.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp or hrp:FindFirstChild("ESPName") then return end

    local gui = Instance.new("BillboardGui", hrp)
    gui.Name = "ESPName"
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
    local gui = hrp and hrp:FindFirstChild("ESPName")
    if gui and gui:FindFirstChildWhichIsA("TextLabel") then
        local dist = math.floor((hrp.Position - Camera.CFrame.Position).Magnitude)
        gui.TextLabel.Text = player.Name .. " [" .. dist .. "m]"
    end
end

-- Aimbot
local aiming = false
local function getClosestTarget()
    local mousePos = UIS:GetMouseLocation()
    local closest, minDist = nil, config.fovSize

    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("Head") then
            if config.teamCheck and p.Team == LocalPlayer.Team then continue end
            local head = p.Character.Head
            local screenPos, onScreen = Camera:WorldToViewportPoint(head.Position)
            local dist = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude
            if onScreen and dist < minDist then
                minDist = dist
                closest = head.Position
            end
        end
    end

    return closest
end

-- Botão na tela para ativar aimbot (pressionar e segurar)
local aimBtn = Instance.new("TextButton", screenGui)
aimBtn.Size = UDim2.new(0, 100, 0, 100)
aimBtn.Position = UDim2.new(1, -110, 1, -110)
aimBtn.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
aimBtn.TextColor3 = Color3.new(1, 1, 1)
aimBtn.Text = "AIM"
aimBtn.TextScaled = true
aimBtn.AutoButtonColor = false

aimBtn.MouseButton1Down:Connect(function() aiming = true end)
aimBtn.MouseButton1Up:Connect(function() aiming = false end)

-- Loop principal
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
