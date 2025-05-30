local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
local replicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")

local enemiesFolder = workspace:WaitForChild("Enemies")

local state = {
    autofarm = false,
    autoEquipWeapon = false,
    autoEquipWeaponName = "Sword",
    velocidadeCustom = false,
    speedValor = 100,
    puloInfinito = false,
    noclip = false,
    autoStats = false,
    autoStatsQtd = 5,
    autoStatsType = "Strength",
    selectedEnemyName = "Bandit",
}

local screenGui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
screenGui.Name = "BloxFruitsAutoFarmUI"

local mainFrame = Instance.new("Frame", screenGui)
mainFrame.Size = UDim2.new(0, 350, 0, 500)
mainFrame.Position = UDim2.new(0.5, -175, 0.5, -250)
mainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
mainFrame.BorderSizePixel = 0
mainFrame.ClipsDescendants = true

local titleBar = Instance.new("Frame", mainFrame)
titleBar.Size = UDim2.new(1, 0, 0, 40)
titleBar.BackgroundColor3 = Color3.fromRGB(40, 40, 70)

local titleLabel = Instance.new("TextLabel", titleBar)
titleLabel.Size = UDim2.new(1, -50, 1, 0)
titleLabel.Position = UDim2.new(0, 10, 0, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "Blox Fruits - AutoFarm"
titleLabel.TextColor3 = Color3.fromRGB(200, 200, 255)
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 20
titleLabel.TextXAlignment = Enum.TextXAlignment.Left

local closeBtn = Instance.new("TextButton", titleBar)
closeBtn.Size = UDim2.new(0, 40, 1, 0)
closeBtn.Position = UDim2.new(1, -40, 0, 0)
closeBtn.BackgroundColor3 = Color3.fromRGB(80, 20, 20)
closeBtn.Text = "X"
closeBtn.TextColor3 = Color3.fromRGB(255, 100, 100)
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 20
closeBtn.BorderSizePixel = 0

closeBtn.MouseButton1Click:Connect(function()
    screenGui:Destroy()
end)

local tabs = {"Auto Farm", "Utilidades", "Misc"}
local buttons = {}
local pages = {}

local currentPage = nil

local function switchPage(name)
    if currentPage then
        pages[currentPage].Visible = false
        buttons[currentPage].BackgroundColor3 = Color3.fromRGB(50, 50, 100)
    end
    pages[name].Visible = true
    buttons[name].BackgroundColor3 = Color3.fromRGB(80, 80, 160)
    currentPage = name
end

local btnContainer = Instance.new("Frame", mainFrame)
btnContainer.Size = UDim2.new(1, -20, 0, 30)
btnContainer.Position = UDim2.new(0, 10, 0, 45)
btnContainer.BackgroundTransparency = 1
btnContainer.LayoutOrder = 1

local UIListLayout = Instance.new("UIListLayout", btnContainer)
UIListLayout.FillDirection = Enum.FillDirection.Horizontal
UIListLayout.SortOrder = Enum.SortOrder.LayoutOrder
UIListLayout.Padding = UDim.new(0, 10)

for i, tabName in ipairs(tabs) do
    local btn = Instance.new("TextButton", btnContainer)
    btn.Text = tabName
    btn.Size = UDim2.new(0, 100, 1, 0)
    btn.BackgroundColor3 = Color3.fromRGB(50, 50, 100)
    btn.TextColor3 = Color3.fromRGB(220, 220, 255)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 18
    btn.BorderSizePixel = 0
    btn.AutoButtonColor = false

    btn.MouseButton1Click:Connect(function()
        switchPage(tabName)
    end)

    buttons[tabName] = btn

    local page = Instance.new("Frame", mainFrame)
    page.Size = UDim2.new(1, -20, 1, -100)
    page.Position = UDim2.new(0, 10, 0, 85)
    page.BackgroundTransparency = 1
    page.Visible = false
    pages[tabName] = page
end

switchPage("Auto Farm")

local function createToggle(parent, text, y, key)
    local label = Instance.new("TextLabel", parent)
    label.Position = UDim2.new(0, 5, 0, y)
    label.Size = UDim2.new(0, 200, 0, 25)
    label.BackgroundTransparency = 1
    label.Text = text
    label.Font = Enum.Font.SourceSans
    label.TextSize = 18
    label.TextColor3 = Color3.fromRGB(230,230,230)
    label.TextXAlignment = Enum.TextXAlignment.Left

    local toggle = Instance.new("TextButton", parent)
    toggle.Position = UDim2.new(0, 250, 0, y)
    toggle.Size = UDim2.new(0, 80, 0, 25)
    toggle.BackgroundColor3 = Color3.fromRGB(120, 40, 40)
    toggle.TextColor3 = Color3.fromRGB(255, 180, 180)
    toggle.Text = "OFF"
    toggle.Font = Enum.Font.SourceSansBold
    toggle.TextSize = 18
    toggle.BorderSizePixel = 0

    toggle.MouseButton1Click:Connect(function()
        state[key] = not state[key]
        if state[key] then
            toggle.Text = "ON"
            toggle.BackgroundColor3 = Color3.fromRGB(40, 120, 40)
            toggle.TextColor3 = Color3.fromRGB(180, 255, 180)
        else
            toggle.Text = "OFF"
            toggle.BackgroundColor3 = Color3.fromRGB(120, 40, 40)
            toggle.TextColor3 = Color3.fromRGB(255, 180, 180)
        end
    end)

    return toggle
end

local function createDropdown(parent, text, y, options, callback, default)
    local label = Instance.new("TextLabel", parent)
    label.Position = UDim2.new(0, 5, 0, y)
    label.Size = UDim2.new(0, 200, 0, 25)
    label.BackgroundTransparency = 1
    label.Text = text
    label.Font = Enum.Font.SourceSans
    label.TextSize = 18
    label.TextColor3 = Color3.fromRGB(230,230,230)
    label.TextXAlignment = Enum.TextXAlignment.Left

    local dropdown = Instance.new("TextButton", parent)
    dropdown.Position = UDim2.new(0, 250, 0, y)
    dropdown.Size = UDim2.new(0, 80, 0, 25)
    dropdown.BackgroundColor3 = Color3.fromRGB(50, 50, 100)
    dropdown.TextColor3 = Color3.fromRGB(220, 220, 255)
    dropdown.Text = default or options[1]
    dropdown.Font = Enum.Font.SourceSansBold
    dropdown.TextSize = 18
    dropdown.BorderSizePixel = 0

    dropdown.MouseButton1Click:Connect(function()
        local currentIndex = 1
        for i,v in ipairs(options) do
            if v == dropdown.Text then
                currentIndex = i
                break
            end
        end
        currentIndex = currentIndex + 1
        if currentIndex > #options then
            currentIndex = 1
        end
        dropdown.Text = options[currentIndex]
        callback(options[currentIndex])
    end)

    return dropdown
end

local function addLog(text)
    local newLog = Instance.new("TextLabel", logFrame)
    newLog.Size = UDim2.new(1, -10, 0, 20)
    newLog.BackgroundTransparency = 1
    newLog.TextColor3 = Color3.fromRGB(180, 180, 255)
    newLog.TextXAlignment = Enum.TextXAlignment.Left
    newLog.Font = Enum.Font.SourceSans
    newLog.TextSize = 18
    newLog.Text = text
    newLog.LayoutOrder = #logFrame:GetChildren() + 1
    logListLayout:DoLayout()
    logFrame.CanvasSize = UDim2.new(0, 0, 0, logListLayout.AbsoluteContentSize.Y + 10)
end

local autoFarmPage = pages["Auto Farm"]

local toggleAutoFarm = createToggle(autoFarmPage, "Auto Farm", 10, "autofarm")

local enemyNames = {"Bandit", "Marine", "Pirate", "None"}
local selectedEnemy = enemyNames[1]

local dropdownEnemy = createDropdown(autoFarmPage, "Selecionar Inimigo", 50, enemyNames, function(value)
    state.selectedEnemyName = value
    addLog("Inimigo selecionado: "..value)
end, enemyNames[1])

local toggleAutoEquip = createToggle(autoFarmPage, "Auto Equip Weapon", 90, "autoEquipWeapon")

local autoEquipNameDropdown = createDropdown(autoFarmPage, "Arma para Equipar", 130, {"Sword", "Gun", "None"}, function(value)
    state.autoEquipWeaponName = value
    addLog("Arma selecionada: "..value)
end, "Sword")

local autoStatsPage = pages["Utilidades"]

local toggleAutoStats = createToggle(autoStatsPage, "Auto Stats", 10, "autoStats")

local statsOptions = {"Strength", "Defense", "Speed", "Health"}

local dropdownStatsType = createDropdown(autoStatsPage, "Stat para Upar", 50, statsOptions, function(value)
    state.autoStatsType = value
    addLog("Stat selecionado: "..value)
end, statsOptions[1])

local statQtyInput = Instance.new("TextBox", autoStatsPage)
statQtyInput.Position = UDim2.new(0, 250, 0, 90)
statQtyInput.Size = UDim2.new(0, 80, 0, 25)
statQtyInput.PlaceholderText = "Qtd"
statQtyInput.Text = tostring(state.autoStatsQtd)
statQtyInput.ClearTextOnFocus = false
statQtyInput.Font = Enum.Font.SourceSans
statQtyInput.TextSize = 18
statQtyInput.TextColor3 = Color3.fromRGB(230,230,230)
statQtyInput.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
statQtyInput.BorderSizePixel = 0

statQtyInput.FocusLost:Connect(function(enter)
    local val = tonumber(statQtyInput.Text)
    if val and val > 0 then
        state.autoStatsQtd = val
        addLog("Quantidade de stat atualizada: "..val)
    else
        statQtyInput.Text = tostring(state.autoStatsQtd)
    end
end)

local miscPage = pages["Misc"]

local toggleSpeed = createToggle(miscPage, "Velocidade Custom", 10, "velocidadeCustom")

local speedInput = Instance.new("TextBox", miscPage)
speedInput.Position = UDim2.new(0, 250, 0, 10)
speedInput.Size = UDim2.new(0, 80, 0, 25)
speedInput.PlaceholderText = "Velocidade"
speedInput.Text = tostring(state.speedValor)
speedInput.ClearTextOnFocus = false
speedInput.Font = Enum.Font.SourceSans
speedInput.TextSize = 18
speedInput.TextColor3 = Color3.fromRGB(230,230,230)
speedInput.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
speedInput.BorderSizePixel = 0

speedInput.FocusLost:Connect(function(enter)
    local val = tonumber(speedInput.Text)
    if val and val > 16 and val <= 500 then
        state.speedValor = val
        addLog("Velocidade atualizada: "..val)
    else
        speedInput.Text = tostring(state.speedValor)
    end
end)

local toggleInfiniteJump = createToggle(miscPage, "Pulo Infinito", 50, "puloInfinito")

local toggleNoClip = createToggle(miscPage, "NoClip", 90, "noclip")

local logFrame = Instance.new("ScrollingFrame", mainFrame)
logFrame.Size = UDim2.new(1, -20, 0, 120)
logFrame.Position = UDim2.new(0, 10, 1, -130)
logFrame.BackgroundColor3 = Color3.fromRGB(10, 10, 20)
logFrame.BorderSizePixel = 0
logFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
logFrame.ScrollBarThickness = 5

local logListLayout = Instance.new("UIListLayout", logFrame)
logListLayout.SortOrder = Enum.SortOrder.LayoutOrder
logListLayout.Padding = UDim.new(0, 3)

addLog("Script iniciado com sucesso!")

-- Funções de AutoFarm

local function equipWeapon(name)
    if name == "None" then return end
    local backpack = player.Backpack
    local char = player.Character
    if not backpack or not char then return end

    local tool = backpack:FindFirstChild(name) or char:FindFirstChild(name)
    if tool and tool:IsA("Tool") then
        tool.Parent = char
        addLog("Equipou arma: "..name)
    end
end

local function findNearestEnemy(name)
    local nearest = nil
    local nearestDist = math.huge
    for _, enemy in pairs(enemiesFolder:GetChildren()) do
        if enemy.Name == name and enemy:FindFirstChild("HumanoidRootPart") and enemy:FindFirstChild("Humanoid") and enemy.Humanoid.Health > 0 then
            local dist = (enemy.HumanoidRootPart.Position - humanoidRootPart.Position).Magnitude
            if dist < nearestDist then
                nearestDist = dist
                nearest = enemy
            end
        end
    end
    return nearest
end

local function attackEnemy(enemy)
    if not enemy or not enemy:FindFirstChild("HumanoidRootPart") then return end
    humanoidRootPart.CFrame = enemy.HumanoidRootPart.CFrame * CFrame.new(0, 0, 3)
end

local autofarmCo

local function autoFarmLoop()
    while state.autofarm do
        local enemyName = state.selectedEnemyName
        if enemyName == "None" then
            wait(1)
            continue
        end
        local enemy = findNearestEnemy(enemyName)
        if enemy then
            attackEnemy(enemy)
            if state.autoEquipWeapon then
                equipWeapon(state.autoEquipWeaponName)
            end
            addLog("Atacando inimigo: "..enemyName)
        else
            addLog("Nenhum inimigo encontrado: "..enemyName)
        end
        wait(1)
    end
end

local function autoStatsLoop()
    while state.autoStats do
        -- Código para upar stat aqui (depende do seu jogo)
        addLog("Upando stat: "..state.autoStatsType.." +"..state.autoStatsQtd)
        wait(10)
    end
end

local function miscLoop()
    while true do
        if state.velocidadeCustom then
            humanoid.WalkSpeed = state.speedValor
        else
            humanoid.WalkSpeed = 16
        end

        if state.puloInfinito then
            humanoid.JumpPower = 100
        else
            humanoid.JumpPower = 50
        end

        if state.noclip then
            for _, part in pairs(character:GetChildren()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                end
            end
        else
            for _, part in pairs(character:GetChildren()) do
                if part:IsA("BasePart") then
                    part.CanCollide = true
                end
            end
        end

        wait(0.5)
    end
end

spawn(function()
    while true do
        if state.autofarm then
            autoFarmLoop()
        else
            wait(0.5)
        end
    end
end)

spawn(function()
    while true do
        if state.autoStats then
            autoStatsLoop()
        else
            wait(1)
        end
    end
end)

spawn(miscLoop)
