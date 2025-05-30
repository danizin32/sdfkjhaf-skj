-- Configurações iniciais
local config = {
    esp = false,
    aimbot = false,
    fovSize = 100
}

-- Função para salvar config (ajuste conforme necessário)
local function saveConfig()
    if writefile then
        writefile("aim_config.txt", game:GetService("HttpService"):JSONEncode(config))
    end
end

-- Função para carregar config (se existir)
if isfile and isfile("aim_config.txt") then
    local data = readfile("aim_config.txt")
    local success, result = pcall(function()
        return game:GetService("HttpService"):JSONDecode(data)
    end)
    if success and type(result) == "table" then
        config = result
    end
end

-- Criar GUI principal
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
ScreenGui.ResetOnSpawn = false

local mainFrame = Instance.new("Frame", ScreenGui)
mainFrame.Size = UDim2.new(0, 300, 0, 400)
mainFrame.Position = UDim2.new(0.5, -150, 0.5, -200)
mainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Name = "AimbotUI"

local corner = Instance.new("UICorner", mainFrame)
corner.CornerRadius = UDim.new(0, 12)

-- Título
local title = Instance.new("TextLabel", mainFrame)
title.Size = UDim2.new(1, 0, 0, 40)
title.BackgroundTransparency = 1
title.Text = "Painel de Controle"
title.TextColor3 = Color3.new(1, 1, 1)
title.Font = Enum.Font.GothamBold
title.TextScaled = true

-- Botão de Minimizar
local minimize = Instance.new("TextButton", mainFrame)
minimize.Size = UDim2.new(0, 30, 0, 30)
minimize.Position = UDim2.new(1, -35, 0, 5)
minimize.Text = "-"
minimize.TextColor3 = Color3.new(1, 1, 1)
minimize.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
minimize.Font = Enum.Font.GothamBold
Instance.new("UICorner", minimize)

-- Container para os botões
local contentHolder = Instance.new("Frame", mainFrame)
contentHolder.Position = UDim2.new(0, 0, 0, 50)
contentHolder.Size = UDim2.new(1, 0, 1, -50)
contentHolder.BackgroundTransparency = 1

local elements = {} -- Armazena os botões para esconder/mostrar

-- Criação dos botões toggle
local y = 10
local function createToggle(name, setting)
    local btn = Instance.new("TextButton", contentHolder)
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

    table.insert(elements, btn)
    y += 50
end

createToggle("ESP", "esp")
createToggle("Aimbot", "aimbot")

-- Label do FOV
local fovLabel = Instance.new("TextLabel", contentHolder)
fovLabel.Size = UDim2.new(1, 0, 0, 30)
fovLabel.Position = UDim2.new(0, 0, 0, y)
fovLabel.BackgroundTransparency = 1
fovLabel.Text = "FOV: " .. config.fovSize
fovLabel.TextColor3 = Color3.new(1, 1, 1)
fovLabel.Font = Enum.Font.Gotham
fovLabel.TextScaled = true
table.insert(elements, fovLabel)
y += 35

-- Botões de aumentar/diminuir FOV
local plus = Instance.new("TextButton", contentHolder)
plus.Size = UDim2.new(0.42, 0, 0, 35)
plus.Position = UDim2.new(0.05, 0, 0, y)
plus.Text = "Aumentar"
plus.TextColor3 = Color3.new(1, 1, 1)
plus.Font = Enum.Font.Gotham
plus.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
Instance.new("UICorner", plus)
table.insert(elements, plus)

local minus = plus:Clone()
minus.Text = "Diminuir"
minus.Position = UDim2.new(0.53, 0, 0, y)
minus.Parent = contentHolder
table.insert(elements, minus)

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

-- Minimizar com animação completa (tudo)
local minimized = false
minimize.MouseButton1Click:Connect(function()
    minimized = not minimized
    local goalSize = minimized and UDim2.new(0, 300, 0, 40) or UDim2.new(0, 300, 0, 400)

    game:GetService("TweenService"):Create(mainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Sine), {Size = goalSize}):Play()
    for _, element in ipairs(elements) do
        element.Visible = not minimized
    end
end)
