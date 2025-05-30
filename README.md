local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer
local HttpService = game:GetService("HttpService")

-- CONFIG
local config = {
    esp = true,
    aimbot = true,
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

-- GUI Bonita
local gui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
gui.Name = "RealisticGUI"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true

local mainFrame = Instance.new("Frame", gui)
mainFrame.Size = UDim2.new(0, 250, 0, 300)
mainFrame.Position = UDim2.new(0.02, 0, 0.2, 0)
mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
mainFrame.BorderSizePixel = 0

local uicorner = Instance.new("UICorner", mainFrame)
uicorner.CornerRadius = UDim.new(0, 12)

local title = Instance.new("TextLabel", mainFrame)
title.Size = UDim2.new(1, 0, 0, 40)
title.BackgroundTransparency = 1
title.Text = "🔧 Mobile Aimbot GUI"
title.TextColor3 = Color3.new(1, 1, 1)
title.TextScaled = true
title.Font = Enum.Font.GothamBold

-- Criador de botões realistas ON/OFF
local yPos = 50
local function createToggle(name, setting)
    local button = Instance.new("TextButton", mainFrame)
    button.Size = UDim2.new(0.9, 0, 0, 40)
    button.Position = UDim2.new(0.05, 0, 0, yPos)
    button.BackgroundColor3 = config[setting] and Color3.fromRGB(60, 180, 75) or Color3.fromRGB(180, 60, 60)
    button.Text = name .. ": " .. (config[setting] and "ON" or "OFF")
    button.TextColor3 = Color3.new(1,1,1)
    button.Font = Enum.Font.GothamBold
    button.TextScaled = true

    local corner = Instance.new("UICorner", button)
    corner.CornerRadius = UDim.new(0, 8)

    button.MouseButton1Click:Connect(function()
        config[setting] = not config[setting]
        button.Text = name .. ": " .. (config[setting] and "ON" or "OFF")
        button.BackgroundColor3 = config[setting] and Color3.fromRGB(60, 180, 75) or Color3.fromRGB(180, 60, 60)
        saveConfig()
    end)

    yPos += 50
end

createToggle("ESP", "esp")
createToggle("Aimbot", "aimbot")

-- Botões FOV
local fovUp = Instance.new("TextButton", mainFrame)
fovUp.Size = UDim2.new(0.42, 0, 0, 40)
fovUp.Position = UDim2.new(0.05, 0, 0, yPos)
fovUp.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
fovUp.Text = "FOV +"
fovUp.TextColor3 = Color3.new(1,1,1)
fovUp.Font = Enum.Font.Gotham
fovUp.TextScaled = true
Instance.new("UICorner", fovUp)

local fovDown = fovUp:Clone()
fovDown.Text = "FOV -"
fovDown.Position = UDim2.new(0.53, 0, 0, yPos)
fovDown.Parent = mainFrame

fovUp.MouseButton1Click:Connect(function()
    config.fovSize += 20
    saveConfig()
end)

fovDown.MouseButton1Click:Connect(function()
    config.fovSize = math.max(20, config.fovSize - 20)
    saveConfig()
end)

-- Botão de AIM
local aimBtn = Instance.new("TextButton", gui)
aimBtn.Size = UDim2.new(0.2, 0, 0.12, 0)
aimBtn.Position = UDim2.new(0.77, 0, 0.83, 0)
aimBtn.BackgroundColor3 = Color3.fromRGB(90,0,0)
aimBtn.Text = "AIM"
aimBtn.TextColor3 = Color3.new(1,1,1)
aimBtn.TextScaled = true
Instance.new("UICorner", aimBtn)

local aiming = false
aimBtn.MouseButton1Down:Connect(function() aiming = true end)
aimBtn.MouseButton1Up:Connect(function() aiming = false end)

-- FOV Círculo
local fovCircle = Instance.new("Frame", gui)
fovCircle.AnchorPoint = Vector2.new(0.5, 0.5)
fovCircle.Position = UDim2.new(0.5, 0, 0.5, 0)
fovCircle.Size = UDim2.new(0, config.fovSize*2, 0, config.fovSize*2)
fovCircle.BackgroundColor3 = Color3.fromRGB(255,255,255)
fovCircle.BackgroundTransparency = 0.85
fovCircle.BorderColor3 = Color3.new(1,1,1)
fovCircle.BorderSizePixel = 1
Instance.new("UICorner", fovCircle).CornerRadius = UDim.new(1, 0)

RunService.RenderStepped:Connect(function()
    fovCircle.Size = UDim2.new(0, config.fovSize*2, 0, config.fovSize*2)
end)

-- ESP com caixa na cor do time
local function createESPBox(player)
    RunService.RenderStepped:Connect(function()
        if not config.esp then return end
        if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local part = player.Character.HumanoidRootPart
            if not part:FindFirstChild("ESPBox") then
                local box = Instance.new("BoxHandleAdornment")
                box.Name = "ESPBox"
                box.Adornee = part
                box.Size = Vector3.new(4,6,2)
                box.AlwaysOnTop = true
                box.ZIndex = 1
                box.Transparency = 0.3
                box.Color3 = player.Team and player.Team.TeamColor.Color or Color3.new(1,1,1)
                box.Parent = part
            end
        end
    end)
end

-- Aimbot: só em inimigos
local function getClosestEnemy()
    local closest = nil
    local shortest = config.fovSize

    for _,p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Team ~= LocalPlayer.Team and p.Character and p.Character:FindFirstChild("Head") then
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

RunService.RenderStepped:Connect(function()
    if aiming and config.aimbot then
        local target = getClosestEnemy()
        if target then
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, target.Position)
        end
    end
end)

-- Aplicar ESP
for _,p in ipairs(Players:GetPlayers()) do
    if p ~= LocalPlayer then
        createESPBox(p)
    end
end

Players.PlayerAdded:Connect(function(p)
    if p ~= LocalPlayer then
        p.CharacterAdded:Connect(function()
            wait(1)
            createESPBox(p)
        end)
    end
end)
