-- UI principal
local ui = Instance.new("ScreenGui", game:GetService("CoreGui"))
ui.Name = "PainelVisual"

-- Quadro principal
local frame = Instance.new("Frame", ui)
frame.Size = UDim2.new(0, 260, 0, 300)
frame.Position = UDim2.new(0.05, 0, 0.1, 0)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true
frame.ClipsDescendants = true
Instance.new("UICorner", frame)

-- Área do conteúdo interno
local contentFrame = Instance.new("Frame", frame)
contentFrame.Size = UDim2.new(1, 0, 1, -30)
contentFrame.Position = UDim2.new(0, 0, 0, 30)
contentFrame.BackgroundTransparency = 1

local layout = Instance.new("UIListLayout", contentFrame)
layout.Padding = UDim.new(0, 8)
layout.SortOrder = Enum.SortOrder.LayoutOrder

-- Cabeçalho
local title = Instance.new("TextLabel", frame)
title.Text = "Painel de Controles"
title.Size = UDim2.new(1, 0, 0, 30)
title.TextColor3 = Color3.new(1, 1, 1)
title.Font = Enum.Font.GothamBold
title.TextScaled = true
title.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
Instance.new("UICorner", title)

-- Botão minimizar
local minimize = Instance.new("TextButton", frame)
minimize.Size = UDim2.new(0, 30, 0, 30)
minimize.Position = UDim2.new(1, -35, 0, 0)
minimize.Text = "-"
minimize.TextColor3 = Color3.new(1,1,1)
minimize.BackgroundTransparency = 1
minimize.Font = Enum.Font.GothamBold
minimize.TextScaled = true

-- Estado de minimização
local isMinimized = false

-- Animação de minimizar/abrir
local function toggleMinimize()
    isMinimized = not isMinimized
    local goalSize = isMinimized and UDim2.new(0, 260, 0, 30) or UDim2.new(0, 260, 0, 300)
    local tween = game:GetService("TweenService"):Create(frame, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size = goalSize})
    tween:Play()
    contentFrame.Visible = not isMinimized
end

minimize.MouseButton1Click:Connect(toggleMinimize)

-- Criação de botões e FOV
local y = 0
local function createToggle(name, setting)
    local btn = Instance.new("TextButton", contentFrame)
    btn.Size = UDim2.new(0.9, 0, 0, 40)
    btn.Position = UDim2.new(0.05, 0, 0, 0)
    btn.LayoutOrder = y
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

    y += 1
end

createToggle("ESP", "esp")
createToggle("Aimbot", "aimbot")

-- FOV label
local fovLabel = Instance.new("TextLabel", contentFrame)
fovLabel.Size = UDim2.new(1, -10, 0, 30)
fovLabel.Text = "FOV: " .. config.fovSize
fovLabel.TextColor3 = Color3.new(1,1,1)
fovLabel.BackgroundTransparency = 1
fovLabel.Font = Enum.Font.Gotham
fovLabel.TextScaled = true
fovLabel.LayoutOrder = y
y += 1

-- Botões de FOV
local plus = Instance.new("TextButton", contentFrame)
plus.Size = UDim2.new(0.45, 0, 0, 35)
plus.Text = "Aumentar FOV"
plus.TextColor3 = Color3.new(1,1,1)
plus.Font = Enum.Font.Gotham
plus.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
Instance.new("UICorner", plus)
plus.LayoutOrder = y

local minus = plus:Clone()
minus.Text = "Diminuir FOV"
minus.LayoutOrder = y
plus.Parent = contentFrame
minus.Parent = contentFrame
y += 1

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
