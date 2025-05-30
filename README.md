-- Configurações iniciais
local config = {
    esp = false,
    aimbot = false,
    fovSize = 100
}

-- Função para salvar configurações
local function saveConfig()
    if writefile then
        writefile("aim_config.txt", game:GetService("HttpService"):JSONEncode(config))
    end
end

-- Função para carregar configurações
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

y += 50

-- Aba de Créditos
local creditsLabel = Instance.new("TextLabel", contentHolder)
creditsLabel.Size = UDim2.new(1, 0, 0, 30)
creditsLabel.Position = UDim2.new(0, 0, 0, y)
creditsLabel.BackgroundTransparency = 1
creditsLabel.Text = "Créditos: SeuNomeAqui"
creditsLabel.TextColor3 = Color3.new(1, 1, 1)
creditsLabel.Font = Enum.Font.Gotham
creditsLabel.TextScaled = true
table.insert(elements, creditsLabel)

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

-- Função para encontrar o jogador mais próximo
local function getClosestPlayer()
    local players = game:GetService("Players")
    local localPlayer = players.LocalPlayer
    local camera = workspace.CurrentCamera
    local closestDistance = math.huge
    local closestPlayer = nil

    for _, player in pairs(players:GetPlayers()) do
        if player ~= localPlayer and player.Character and player.Character:FindFirstChild("Head") then
            if config.aimbot then
                if config.teamCheck and player.Team == localPlayer.Team then
                    continue
                end
                local screenPoint, onScreen = camera:WorldToViewportPoint(player.Character.Head.Position)
                if onScreen then
                    local mouseLocation = UserInputService:GetMouseLocation()
                    local distance = (Vector2.new(screenPoint.X, screenPoint.Y) - Vector2.new(mouseLocation.X, mouseLocation.Y)).magnitude
                    if distance < closestDistance and distance < config.fovSize then
                        closestDistance = distance
                        closestPlayer = player
                    end
                end
            end
        end
    end
    return closestPlayer
end

-- Aimbot
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

RunService.RenderStepped:Connect(function()
    if config.aimbot then
        local target = getClosestPlayer()
        if target and target.Character and target.Character:FindFirstChild("Head") then
            workspace.CurrentCamera.CFrame = CFrame.new(workspace.CurrentCamera.CFrame.Position, target.Character.Head.Position)
        end
    end
end)

-- ESP
local function createESP(player)
    local box = Instance.new("BoxHandleAdornment")
    box.Name = "ESPBox"
    box.Adornee = player.Character
    box.AlwaysOnTop = true
    box.ZIndex = 5
    box.Size = Vector3.new(4, 6, 4)
    box.Transparency = 0.5
    box.Color3 = Color3.fromRGB(255, 0, 0)
    box.Parent = player.Character
end

local function removeESP(player)
    if player.Character and player.Character:FindFirstChild("ESPBox") then
        player.Character.ESPBox:Destroy()
    end
end

game:GetService("Players").PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        if config.esp then
            createESP(player)
        end
    end)
end)

for _, player in pairs(game:GetService("Players"):GetPlayers()) do
    if player ~= game:GetService("Players").LocalPlayer then
        if config.esp then
            createESP(player)
        end
    end
end

-- Atualizar ESP quando o toggle for alterado
local function updateESP()
    for _, player in pairs(game:GetService("Players"):GetPlayers()) do
        if player ~= game:GetService("Players").LocalPlayer then
            if config.esp then
                if not player.Character:FindFirstChild
::contentReference[oaicite:0]{index=0}
 
