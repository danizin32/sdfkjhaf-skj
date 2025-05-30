local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer
local HttpService = game:GetService("HttpService")

-- CONFIG
local config = {
    esp = true,
    aimbot = true,
    teamCheck = true,
    fovSize = 100
}
local configFile = "mobile_config.json"

if isfile and isfile(configFile) then
    local data = HttpService:JSONDecode(readfile(configFile))
    for k,v in pairs(data) do config[k] = v end
end

local function saveConfig()
    if writefile then
        writefile(configFile, HttpService:JSONEncode(config))
    end
end

-- GUI
local gui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
gui.Name = "MainMobileGUI"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true

local function createButton(txt, yPos, cb)
    local btn = Instance.new("TextButton", gui)
    btn.Size = UDim2.new(0.3, 0, 0.07, 0)
    btn.Position = UDim2.new(0.02, 0, yPos, 0)
    btn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    btn.Text = txt
    btn.TextColor3 = Color3.new(1,1,1)
    btn.TextScaled = true
    btn.MouseButton1Click:Connect(cb)
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
    config.fovSize += 20
    saveConfig()
end)

createButton("FOV -", 0.37, function()
    config.fovSize = math.max(20, config.fovSize - 20)
    saveConfig()
end)

-- Botão AIM
local aimBtn = Instance.new("TextButton", gui)
aimBtn.Size = UDim2.new(0.2, 0, 0.12, 0)
aimBtn.Position = UDim2.new(0.77, 0, 0.83, 0)
aimBtn.BackgroundColor3 = Color3.fromRGB(90,0,0)
aimBtn.Text = "AIM"
aimBtn.TextColor3 = Color3.new(1,1,1)
aimBtn.TextScaled = true

local aiming = false
aimBtn.MouseButton1Down:Connect(function() aiming = true end)
aimBtn.MouseButton1Up:Connect(function() aiming = false end)

-- FOV GUI (círculo central)
local fovCircle = Instance.new("Frame", gui)
fovCircle.AnchorPoint = Vector2.new(0.5, 0.5)
fovCircle.Position = UDim2.new(0.5, 0, 0.5, 0)
fovCircle.Size = UDim2.new(0, config.fovSize*2, 0, config.fovSize*2)
fovCircle.BackgroundColor3 = Color3.new(1,1,1)
fovCircle.BackgroundTransparency = 0.9
fovCircle.BorderColor3 = Color3.new(1,1,1)
fovCircle.BorderSizePixel = 1
fovCircle.ZIndex = 10
fovCircle.Visible = true
fovCircle.ClipsDescendants = true
local corner = Instance.new("UICorner", fovCircle)
corner.CornerRadius = UDim.new(1, 0)

-- Atualiza tamanho FOV dinamicamente
RunService.RenderStepped:Connect(function()
    fovCircle.Size = UDim2.new(0, config.fovSize * 2, 0, config.fovSize * 2)
end)

-- ESP por caixa (BoxHandleAdornment)
local function createBox(player)
    RunService.RenderStepped:Connect(function()
        if not config.esp then return end
        local char = player.Character
        if not char or not char:FindFirstChild("HumanoidRootPart") then return end
        if char.HumanoidRootPart:FindFirstChild("ESPBox") then return end

        local box = Instance.new("BoxHandleAdornment")
        box.Name = "ESPBox"
        box.Adornee = char.HumanoidRootPart
        box.Size = Vector3.new(4,6,2)
        box.Color3 = player.Team and player.Team.TeamColor.Color or Color3.new(1,1,1)
        box.AlwaysOnTop = true
        box.ZIndex = 1
        box.Transparency = 0.3
        box.Parent = char.HumanoidRootPart
    end)
end

-- Aimbot
local function getClosestPlayer()
    local closest = nil
    local shortest = config.fovSize

    for _,p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("Head") then
            if config.teamCheck and p.Team == LocalPlayer.Team then continue end
            local screenPos, onScreen = Camera:WorldToViewportPoint(p.Character.Head.Position)
            if onScreen then
                local dist = (Vector2.new(screenPos.X, screenPos.Y) - Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)).Magnitude
                if dist < shortest then
                    shortest = dist
                    closest = p.Character.Head
                end
            end
        end
    end
    return closest
end

-- Loop
RunService.RenderStepped:Connect(function()
    -- Aimbot
    if aiming and config.aimbot then
        local target = getClosestPlayer()
        if target then
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, target.Position)
        end
    end
end)

-- Criar ESP para todos os jogadores
for _,p in ipairs(Players:GetPlayers()) do
    if p ~= LocalPlayer then
        createBox(p)
    end
end

Players.PlayerAdded:Connect(function(p)
    if p ~= LocalPlayer then
        p.CharacterAdded:Connect(function()
            wait(1)
            createBox(p)
        end)
    end
end)
