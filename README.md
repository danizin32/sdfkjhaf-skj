-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer
local HttpService = game:GetService("HttpService")

-- Configuração
local config = {
    esp = true,
    aimbot = false,
    fovSize = 120
}
local configFile = "mobile_config.json"
if isfile and isfile(configFile) then
    local data = HttpService:JSONDecode(readfile(configFile))
    for k,v in pairs(data) do config[k] = v end
end
local function saveConfig()
    if writefile then writefile(configFile, HttpService:JSONEncode(config)) end
end

-- GUI
local gui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
gui.Name = "AimbotInterface"
gui.ResetOnSpawn = false

-- Frame principal
local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 280, 0, 340)
frame.Position = UDim2.new(0.03, 0, 0.25, 0)
frame.BackgroundColor3 = Color3.fromRGB(0, 40, 80)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true
Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 12)

-- Arrastar manual (mobile compatível)
local dragging, dragInput, dragStart, startPos
frame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = frame.Position

        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then dragging = false end
        end)
    end
end)
frame.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)
RunService.Heartbeat:Connect(function()
    if dragging and dragInput then
        local delta = dragInput.Position - dragStart
        frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

-- Minimizar / Abrir botão
local minimizeBtn = Instance.new("TextButton", frame)
minimizeBtn.Size = UDim2.new(0, 40, 0, 40)
minimizeBtn.Position = UDim2.new(1, -45, 0, 5)
minimizeBtn.Text = "-"
minimizeBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 60)
minimizeBtn.TextColor3 = Color3.new(1, 1, 1)
minimizeBtn.Font = Enum.Font.GothamBold
minimizeBtn.TextScaled = true
Instance.new("UICorner", minimizeBtn)

local isMinimized = false
local fullSize = frame.Size
local minimizedSize = UDim2.new(0, 280, 0, 50)

minimizeBtn.MouseButton1Click:Connect(function()
    isMinimized = not isMinimized
    minimizeBtn.Text = isMinimized and "+" or "-"
    local goal = {Size = isMinimized and minimizedSize or fullSize}
    TweenService:Create(frame, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), goal):Play()
end)

-- Título
local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, -50, 0, 50)
title.Position = UDim2.new(0, 0, 0, 0)
title.BackgroundTransparency = 1
title.Text = "💠 Aimbot Mobile Menu"
title.TextColor3 = Color3.fromRGB(200, 230, 255)
title.Font = Enum.Font.GothamBold
title.TextScaled = true

-- Criar botão toggle
local y = 60
local function createToggle(name, setting)
    local btn = Instance.new("TextButton", frame)
    btn.Size = UDim2.new(0.9, 0, 0, 40)
    btn.Position = UDim2.new(0.05, 0, 0, y)
    btn.Text = name .. ": " .. (config[setting] and "ON" or "OFF")
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.BackgroundColor3 = config[setting] and Color3.fromRGB(0, 120, 255) or Color3.fromRGB(60, 60, 60)
    btn.Font = Enum.Font.GothamBold
    btn.TextScaled = true
    Instance.new("UICorner", btn)

    btn.MouseButton1Click:Connect(function()
        config[setting] = not config[setting]
        btn.Text = name .. ": " .. (config[setting] and "ON" or "OFF")
        btn.BackgroundColor3 = config[setting] and Color3.fromRGB(0, 120, 255) or Color3.fromRGB(60, 60, 60)
        saveConfig()
    end)
    y += 50
end

createToggle("ESP", "esp")
createToggle("Aimbot", "aimbot")

-- FOV Label e controles
local fovLabel = Instance.new("TextLabel", frame)
fovLabel.Size = UDim2.new(1, 0, 0, 30)
fovLabel.Position = UDim2.new(0, 0, 0, y)
fovLabel.BackgroundTransparency = 1
fovLabel.Text = "FOV: " .. config.fovSize
fovLabel.TextColor3 = Color3.new(1,1,1)
fovLabel.Font = Enum.Font.Gotham
fovLabel.TextScaled = true
y += 35

local plus = Instance.new("TextButton", frame)
plus.Size = UDim2.new(0.42, 0, 0, 35)
plus.Position = UDim2.new(0.05, 0, 0, y)
plus.Text = "Aumentar"
plus.TextColor3 = Color3.new(1,1,1)
plus.Font = Enum.Font.Gotham
plus.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
Instance.new("UICorner", plus)

local minus = plus:Clone()
minus.Text = "Diminuir"
minus.Position = UDim2.new(0.53, 0, 0, y)
minus.Parent = frame

plus.Parent = frame
plus.MouseButton1Click:Connect(function()
    config.fovSize += 20
    fovLabel.Text = "FOV: " .. config.fovSize
    saveConfig()
end)
minus.MouseButton1Click:Connect(function()
    config.fovSize = math.max(20, config.fovSize - 20)
    fovLabel.Text = "FOV: " .. config.fovSize
    saveConfig()
end)

-- Círculo FOV
local fovCircle = Instance.new("Frame", gui)
fovCircle.AnchorPoint = Vector2.new(0.5, 0.5)
fovCircle.Position = UDim2.new(0.5, 0, 0.5, 0)
fovCircle.Size = UDim2.new(0, config.fovSize * 2, 0, config.fovSize * 2)
fovCircle.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
fovCircle.BackgroundTransparency = 0.7
fovCircle.BorderSizePixel = 0
Instance.new("UICorner", fovCircle).CornerRadius = UDim.new(1, 0)

RunService.RenderStepped:Connect(function()
    fovCircle.Size = UDim2.new(0, config.fovSize * 2, 0, config.fovSize * 2)
end)

-- Aimbot lógica
local function getClosestEnemy()
    local shortest, target = config.fovSize, nil
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Team ~= LocalPlayer.Team and p.Character and p.Character:FindFirstChild("Head") then
            local head = p.Character.Head
            local screen, onScreen = Camera:WorldToViewportPoint(head.Position)
            if onScreen then
                local dist = (Vector2.new(screen.X, screen.Y) - Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)).Magnitude
                if dist < shortest then
                    shortest = dist
                    target = head
                end
            end
        end
    end
    return target
end

RunService.RenderStepped:Connect(function()
    if config.aimbot then
        local target = getClosestEnemy()
        if target then
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, target.Position)
        end
    end
end)

-- ESP
local function createESP(player)
    RunService.RenderStepped:Connect(function()
        if config.esp and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local part = player.Character.HumanoidRootPart
            if not part:FindFirstChild("ESPBox") then
                local box = Instance.new("BoxHandleAdornment")
                box.Name = "ESPBox"
                box.Adornee = part
                box.Size = Vector3.new(4, 6, 2)
                box.Color3 = player.Team and player.Team.TeamColor.Color or Color3.new(1, 1, 1)
                box.AlwaysOnTop = true
                box.ZIndex = 1
                box.Transparency = 0.4
                box.Parent = part

                local tag = Instance.new("BillboardGui", part)
                tag.Name = "ESPName"
                tag.Size = UDim2.new(0, 100, 0, 40)
                tag.AlwaysOnTop = true
                tag.StudsOffset = Vector3.new(0, 3, 0)

                local label = Instance.new("TextLabel", tag)
                label.Size = UDim2.new(1, 0, 1, 0)
                label.BackgroundTransparency = 1
                label.Text = player.Name .. " [" .. math.floor((part.Position - Camera.CFrame.Position).Magnitude) .. "m]"
                label.TextScaled = true
                label.TextColor3 = Color3.new(1, 1, 1)
                label.Font = Enum.Font.Gotham
            end
        end
    end)
end

for _, p in ipairs(Players:GetPlayers()) do if p ~= LocalPlayer then createESP(p) end end
Players.PlayerAdded:Connect(function(p)
    p.CharacterAdded:Connect(function() wait(1) createESP(p) end)
end)
