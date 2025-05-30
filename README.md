-- // Criar GUI
local gui = Instance.new("ScreenGui")
gui.Name = "BlueLockTestUI"
gui.ResetOnSpawn = false
gui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 400, 0, 300)
frame.Position = UDim2.new(0.5, -200, 0.5, -150)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
frame.BorderSizePixel = 0
frame.BackgroundTransparency = 0.1

local uiCorner = Instance.new("UICorner", frame)
uiCorner.CornerRadius = UDim.new(0, 12)

-- Título
local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, 0, 0, 40)
title.Text = "Blue Lock - Teste de Estilo & Fluxo"
title.BackgroundTransparency = 1
title.TextColor3 = Color3.new(1, 1, 1)
title.Font = Enum.Font.GothamBold
title.TextSize = 20

-- Caixa de texto - Personagem
local estiloBox = Instance.new("TextBox", frame)
estiloBox.PlaceholderText = "Digite o nome do personagem"
estiloBox.Size = UDim2.new(1, -40, 0, 35)
estiloBox.Position = UDim2.new(0, 20, 0, 50)
estiloBox.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
estiloBox.TextColor3 = Color3.new(1, 1, 1)
estiloBox.Font = Enum.Font.Gotham
estiloBox.TextSize = 18

-- Caixa de texto - Fluxo
local fluxoBox = Instance.new("TextBox", frame)
fluxoBox.PlaceholderText = "Digite o tipo de fluxo (ex: Velocidade)"
fluxoBox.Size = UDim2.new(1, -40, 0, 35)
fluxoBox.Position = UDim2.new(0, 20, 0, 95)
fluxoBox.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
fluxoBox.TextColor3 = Color3.new(1, 1, 1)
fluxoBox.Font = Enum.Font.Gotham
fluxoBox.TextSize = 18

-- Botão
local giveButton = Instance.new("TextButton", frame)
giveButton.Text = "Give"
giveButton.Size = UDim2.new(0.5, -30, 0, 35)
giveButton.Position = UDim2.new(0.5, 10, 0, 140)
giveButton.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
giveButton.TextColor3 = Color3.new(1, 1, 1)
giveButton.Font = Enum.Font.GothamBold
giveButton.TextSize = 18

local cornerButton = Instance.new("UICorner", giveButton)
cornerButton.CornerRadius = UDim.new(0, 8)

-- Resultado
local resultado = Instance.new("TextLabel", frame)
resultado.Size = UDim2.new(1, -40, 0, 100)
resultado.Position = UDim2.new(0, 20, 0, 190)
resultado.BackgroundTransparency = 1
resultado.TextColor3 = Color3.new(1, 1, 1)
resultado.Font = Enum.Font.Gotham
resultado.TextSize = 16
resultado.TextWrapped = true
resultado.TextYAlignment = Enum.TextYAlignment.Top
resultado.Text = ""

-- Dados
local personagens = {
    Isagi = {"Visão de Jogo", "Antecipação", "Posicionamento"},
    Rin = {"Finalização", "Velocidade de Reação", "Frieza"},
    Barou = {"Chute Potente", "Dominância", "Pressão Ofensiva"},
    Nagi = {"Controle", "Recepção de Bola", "Criatividade"},
    Chigiri = {"Velocidade", "Impulsão", "Fuga Rápida"}
}

local fluxos = {
    Forca = {"Aumento de poder de chute", "Duelo corpo a corpo"},
    Velocidade = {"Corrida explosiva", "Desmarque eficiente"},
    Controle = {"Controle preciso da bola", "Estabilidade"},
    Resistencia = {"Mais turnos ativos", "Menor chance de erro"},
    Criatividade = {"Jogadas únicas", "Improvisação"}
}

-- Função quando clicar em "Give"
giveButton.MouseButton1Click:Connect(function()
    local estilo = estiloBox.Text
    local fluxo = fluxoBox.Text

    local habEstilo = personagens[estilo]
    local habFluxo = fluxos[fluxo]

    if not habEstilo then
        resultado.Text = "❌ Personagem não encontrado!"
        return
    end

    if not habFluxo then
        resultado.Text = "❌ Fluxo não encontrado!"
        return
    end

    local texto = "🎮 " .. estilo .. " selecionado\nHabilidades:\n"
    for _, h in ipairs(habEstilo) do
        texto = texto .. "  - " .. h .. "\n"
    end

    texto = texto .. "\n⚡ Fluxo: " .. fluxo .. "\n"
    for _, h in ipairs(habFluxo) do
        texto = texto .. "  - " .. h .. "\n"
    end

    resultado.Text = texto
end)
