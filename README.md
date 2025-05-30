--[[
📦 SCRIPT COMPLETO: Aimbot + ESP + FOV + Interface com animação
Feito para uso mobile (Arceus X, Hydrogen)
Interface azul, realista, arrastável e profissional
]]--

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local Camera = workspace.CurrentCamera
local UIS = game:GetService("UserInputService")

local config = {
    esp = true,
    aimbot = true,
    fovSize = 120
}

-- Criar GUI principal
local screenGui = Instance.new("ScreenGui", game.CoreGui)
screenGui.ResetOnSpawn = false

-- Arrastável
local dragging, dragInput, dragStart, startPos

local function makeDraggable(gui)
    gui.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = gui.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)

    gui.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)

    UIS.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            local delta = input.Position - dragStart
            gui.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
end

-- Janela
local frame = Instance.new("Frame", screenGui)
frame.Size = UDim2.new(0, 260, 0, 320)
frame.Position = UDim2.new(0, 20, 0, 100)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
frame.BorderSizePixel = 0
frame.Active = true
makeDraggable(frame)

Instance.new("UICorner", frame)

-- Título
local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, 0, 0, 30)
title.BackgroundTransparency = 1
title.Text = "Painel de Controle"
title.TextColor3 = Color3.new(1,1,1)
title.Font = Enum.Font.GothamBold
title.TextScaled = true

-- Minimizar
local minimize = Instance.new("TextButton", frame)
minimize.Size = UDim2.new(0, 30, 0, 30)
minimize.Position = UDim2.new(1, -35, 0, 0)
minimize.Text = "-"
minimize.TextColor3 = Color3.new(1,1,1)
minimize.Font = Enum.Font.GothamBold
minimize.TextScaled = true
minimize.BackgroundTransparency = 1

-- Conteúdo
local content = Instance.new("Frame", frame)
content.Size = UDim2.new(1, 0, 1, -30)
content.Position = UDim2.new(0, 0, 0, 30)
content.BackgroundTransparency = 1

local layout = Instance.new("UIListLayout", content)
layout.SortOrder = Enum.SortOrder.LayoutOrder
layout.Padding = UDim.new(0, 8)

-- Toggle
local function createToggle(name, key)
    local btn = Instance.new("TextButton", content)
    btn.Size = UDim2.new(0.9, 0, 0, 40)
    btn.Position = UDim2.new(0.05, 0, 0, 0)
    btn.Text = name .. ": " .. (config[key] and "ON" or "OFF")
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.BackgroundColor3 = config[key] and Color3.fromRGB(0, 120, 255) or Color3.fromRGB(60, 60, 60)
    btn.Font = Enum.Font.GothamBold
    btn.TextScaled = true
    Instance.new("UICorner", btn)

    btn.MouseButton1Click:Connect(function()
        config[key] = not config[key]
        btn.Text = name .. ": " .. (config[key] and "ON" or "OFF")
        btn.BackgroundColor3 = config[key] and Color3.fromRGB(0, 120, 255) or Color3.fromRGB(60, 60, 60)
    end)
end

createToggle("ESP", "esp")
createToggle("Aimbot", "aimbot")

-- FOV
local fovLabel = Instance.new("TextLabel", content)
fovLabel.Size = UDim2.new(1, 0, 0, 30)
fovLabel.BackgroundTransparency = 1
fovLabel.Text = "FOV: " .. config.fovSize
fovLabel.TextColor3 = Color3.new(1,1,1)
fovLabel.Font = Enum.Font.Gotham
fovLabel.TextScaled = true

local fovFrame = Instance.new("Frame", content)
fovFrame.Size = UDim2.new(0.9, 0, 0, 35)
fovFrame.BackgroundTransparency = 1
Instance.new("UIListLayout", fovFrame).FillDirection = Enum.FillDirection.Horizontal

local plus = Instance.new("TextButton", fovFrame)
plus.Size = UDim2.new(0.5, -5, 1, 0)
plus.Text = "Aumentar"
plus.TextColor3 = Color3.new(1,1,1)
plus.Font = Enum.Font.Gotham
plus.TextScaled = true
plus.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
Instance.new("UICorner", plus)

local minus = plus:Clone()
minus.Text = "Diminuir"
minus.Parent = fovFrame

plus.MouseButton1Click:Connect(function()
    config.fovSize += 20
    fovLabel.Text = "FOV: " .. config.fovSize
end)

minus.MouseButton1Click:Connect(function()
    config.fovSize = math.max(20, config.fovSize - 20)
    fovLabel.Text = "FOV: " .. config.fovSize
end)

-- Minimizar com animação
local isMinimized = false
minimize.MouseButton1Click:Connect(function()
    isMinimized = not isMinimized
    local size = isMinimized and UDim2.new(0, 260, 0, 30) or UDim2.new(0, 260, 0, 320)
    TweenService:Create(frame, TweenInfo.new(0.25), {Size = size}):Play()
    content.Visible = not isMinimized
end)

-- FOV círculo
local fovCircle = Drawing.new("Circle")
fovCircle.Visible = true
fovCircle.Thickness = 1.5
fovCircle.Color = Color3.fromRGB(0, 120, 255)
fovCircle.Transparency = 0.4
fovCircle.Filled = false
fovCircle.Radius = config.fovSize

-- ESP
local boxes = {}

RunService.RenderStepped:Connect(function()
    fovCircle.Position = UIS:GetMouseLocation()
    fovCircle.Radius = config.fovSize

    for _, box in pairs(boxes) do box:Remove() end
    table.clear(boxes)

    if not config.esp then return end

    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") and plr.Team ~= LocalPlayer.Team then
            local pos, onscreen = Camera:WorldToViewportPoint(plr.Character.HumanoidRootPart.Position)
            if onscreen then
                local box = Drawing.new("Square")
                box.Size = Vector2.new(50, 60)
                box.Position = Vector2.new(pos.X - 25, pos.Y - 60)
                box.Color = plr.TeamColor.Color
                box.Thickness = 2
                box.Filled = false
                box.Visible = true
                table.insert(boxes, box)
            end
        end
    end
end)

-- Aimbot
RunService.RenderStepped:Connect(function()
    if not config.aimbot then return end

    local closest, dist = nil, config.fovSize
    local mousePos = UIS:GetMouseLocation()

    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Team ~= LocalPlayer.Team and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
            local pos, onscreen = Camera:WorldToViewportPoint(plr.Character.HumanoidRootPart.Position)
            local mag = (Vector2.new(pos.X, pos.Y) - mousePos).Magnitude
            if onscreen and mag < dist then
                dist = mag
                closest = plr
            end
        end
    end

    if closest and closest.Character and closest.Character:FindFirstChild("HumanoidRootPart") then
        Camera.CFrame = CFrame.new(Camera.CFrame.Position, closest.Character.HumanoidRootPart.Position)
    end
end)
