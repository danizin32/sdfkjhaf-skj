local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
local replicatedStorage = game:GetService("ReplicatedStorage")
local workspaceEnemies = workspace:WaitForChild("Enemies")
local TweenService = game:GetService("TweenService")

local state = {
    autofarm = false,
    autoEquipWeapon = false,
    velocidadeCustom = false,
    puloInfinito = false,
    noclip = false,
    autoCollectFruit = false,
    autoBuyFruit = false,
    autoRaid = false,
    autoBoss = false,
    autoStats = false,
    autoStatsQtd = 5,
    autoStatsType = "Strength",
    speedValor = 100,
    selectedEnemyIndex = 1,
    enemies = {"Bandit", "Pirate", "Zombie", "Boss"},
    autoEquipWeaponName = "Sword",
}

local screenGui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
screenGui.Name = "BloxFruitsAutoFarm"

local mainFrame = Instance.new("Frame", screenGui)
mainFrame.Size = UDim2.new(0, 350, 0, 480)
mainFrame.Position = UDim2.new(0.5, -175, 0.5, -240)
mainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
mainFrame.BorderSizePixel = 0

local gradient = Instance.new("UIGradient", mainFrame)
gradient.Color = ColorSequence.new{
    ColorSequenceKeypoint.new(0, Color3.fromRGB(10, 10, 40)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(35, 35, 70))
}

local titleBar = Instance.new("Frame", mainFrame)
titleBar.Size = UDim2.new(1, 0, 0, 40)
titleBar.BackgroundColor3 = Color3.fromRGB(40, 40, 80)
titleBar.BorderSizePixel = 0

local titleLabel = Instance.new("TextLabel", titleBar)
titleLabel.Text = "Script Blox Fruits - Painel"
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 20
titleLabel.TextColor3 = Color3.fromRGB(200, 200, 255)
titleLabel.BackgroundTransparency = 1
titleLabel.Size = UDim2.new(1, -10, 1, 0)
titleLabel.Position = UDim2.new(0, 10, 0, 0)
titleLabel.TextXAlignment = Enum.TextXAlignment.Left

local closeButton = Instance.new("TextButton", titleBar)
closeButton.Size = UDim2.new(0, 40, 1, 0)
closeButton.Position = UDim2.new(1, -40, 0, 0)
closeButton.Text = "✕"
closeButton.Font = Enum.Font.GothamBold
closeButton.TextSize = 22
closeButton.TextColor3 = Color3.fromRGB(255, 100, 100)
closeButton.BackgroundColor3 = Color3.fromRGB(60, 20, 20)
closeButton.BorderSizePixel = 0

closeButton.MouseEnter:Connect(function()
    TweenService:Create(closeButton, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(200, 50, 50)}):Play()
end)
closeButton.MouseLeave:Connect(function()
    TweenService:Create(closeButton, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(60, 20, 20)}):Play()
end)
closeButton.MouseButton1Click:Connect(function()
    TweenService:Create(mainFrame, TweenInfo.new(0.5), {BackgroundTransparency = 1}):Play()
    wait(0.5)
    screenGui:Destroy()
end)

local tabNames = {"Auto Farm", "Utilidades", "Misc"}
local tabButtons = {}
local tabFrames = {}

local function createTabButton(name, y)
    local btn = Instance.new("TextButton", mainFrame)
    btn.Size = UDim2.new(0, 100, 0, 30)
    btn.Position = UDim2.new(0, 10, 0, y)
    btn.Text = name
    btn.Font = Enum.Font.GothamSemibold
    btn.TextSize = 18
    btn.TextColor3 = Color3.fromRGB(220, 220, 255)
    btn.BackgroundColor3 = Color3.fromRGB(50, 50, 120)
    btn.BorderSizePixel = 0
    btn.AutoButtonColor = false

    btn.MouseEnter:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.25), {BackgroundColor3 = Color3.fromRGB(70, 70, 170)}):Play()
    end)
    btn.MouseLeave:Connect(function()
        if currentTab ~= name then
            TweenService:Create(btn, TweenInfo.new(0.25), {BackgroundColor3 = Color3.fromRGB(50, 50, 120)}):Play()
        end
    end)
    return btn
end

local function createTabFrame(name)
    local frame = Instance.new("Frame", mainFrame)
    frame.Size = UDim2.new(0, 330, 0, 380)
    frame.Position = UDim2.new(0, 10, 0, 90)
    frame.BackgroundColor3 = Color3.fromRGB(30, 30, 70)
    frame.Visible = false
    return frame
end

for i, tabName in ipairs(tabNames) do
    tabButtons[tabName] = createTabButton(tabName, 50 + (i-1)*40)
    tabFrames[tabName] = createTabFrame(tabName)
end

local currentTab

local function switchTab(tabName)
    if currentTab then
        tabFrames[currentTab].Visible = false
        tabButtons[currentTab].BackgroundColor3 = Color3.fromRGB(50, 50, 120)
    end
    tabFrames[tabName].Visible = true
    tabButtons[tabName].BackgroundColor3 = Color3.fromRGB(90, 90, 190)
    currentTab = tabName
end

switchTab("Auto Farm")

local function createLabel(parent, text, posY)
    local lbl = Instance.new("TextLabel", parent)
    lbl.Size = UDim2.new(0, 320, 0, 25)
    lbl.Position = UDim2.new(0, 10, 0, posY)
    lbl.BackgroundTransparency = 1
    lbl.TextColor3 = Color3.new(1,1,1)
    lbl.Text = text
    lbl.Font = Enum.Font.SourceSans
    lbl.TextScaled = true
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    return lbl
end

local function createToggle(parent, text, posY, stateKey)
    local label = createLabel(parent, text, posY)
    local toggle = Instance.new("TextButton", parent)
    toggle.Size = UDim2.new(0, 50, 0, 25)
    toggle.Position = UDim2.new(0, 260, 0, posY)
    toggle.Text = "OFF"
    toggle.Font = Enum.Font.SourceSansBold
    toggle.TextColor3 = Color3.fromRGB(255, 80, 80)
    toggle.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    toggle.BorderSizePixel = 0

    toggle.MouseButton1Click:Connect(function()
        state[stateKey] = not state[stateKey]
        if state[stateKey] then
            toggle.Text = "ON"
            toggle.TextColor3 = Color3.fromRGB(80, 255, 80)
        else
            toggle.Text = "OFF"
            toggle.TextColor3 = Color3.fromRGB(255, 80, 80)
        end
    end)

    return toggle
end

local function createSlider(parent, text, minVal, maxVal, defaultVal, posY, callback)
    createLabel(parent, text, posY)
    local sliderBox = Instance.new("TextBox", parent)
    sliderBox.Size = UDim2.new(0, 80, 0, 25)
    sliderBox.Position = UDim2.new(0, 240, 0, posY)
    sliderBox.Text = tostring(defaultVal)
    sliderBox.ClearTextOnFocus = false
    sliderBox.TextColor3 = Color3.new(1,1,1)
    sliderBox.BackgroundColor3 = Color3.fromRGB(70,70,70)
    sliderBox.Font = Enum.Font.SourceSans
    sliderBox.TextScaled = true

    sliderBox.FocusLost:Connect(function()
        local val = tonumber(sliderBox.Text)
        if val and val >= minVal and val <= maxVal then
            callback(val)
        else
            sliderBox.Text = tostring(defaultVal)
        end
    end)
end

local function createDropdown(parent, text, options, defaultIndex, posY, callback)
    createLabel(parent, text, posY)
    local dd = Instance.new("TextButton", parent)
    dd.Size = UDim2.new(0, 120, 0, 30)
    dd.Position = UDim2.new(0, 200, 0, posY)
    dd.Text = options[defaultIndex]
    dd.BackgroundColor3 = Color3.fromRGB(70,70,70)
    dd.TextColor3 = Color3.new(1,1,1)
    dd.Font = Enum.Font.SourceSansBold
    dd.TextScaled = true

    dd.MouseButton1Click:Connect(function()
        local currentIndex = table.find(options, dd.Text) or 1
        currentIndex = currentIndex + 1
        if currentIndex > #options then currentIndex = 1 end
        dd.Text = options[currentIndex]
        callback(dd.Text)
    end)
end

local tabAF = tabFrames["Auto Farm"]

createToggle(tabAF, "Auto Farm", 10, "autofarm")

createToggle(tabAF, "Auto Equip Arma", 50, "autoEquipWeapon")

local weaponsOptions = {"Sword", "Gun", "None"}
createDropdown(tabAF, "Arma para Auto Equipar", weaponsOptions, 1, 90, function(val)
    state.autoEquipWeaponName = val
end)

createDropdown(tabAF, "Inimigo alvo", state.enemies, 1, 130, function(val)
    for i, v in ipairs(state.enemies) do
        if v == val then
            state.selectedEnemyIndex = i
            break
        end
    end
end)

local tabUtil = tabFrames["Utilidades"]

createToggle(tabUtil, "Auto Stats", 10, "autoStats")

local statsOptions = {"Strength", "Defense", "Speed", "Sword", "Gun", "Fruit"}
createDropdown(tabUtil, "Stat para AutoStats", statsOptions, 1, 50, function(val)
    state.autoStatsType = val
end)

createSlider(tabUtil, "Quantidade AutoStats", 1, 50, state.autoStatsQtd, 90, function(val)
    state.autoStatsQtd = val
end)

createToggle(tabUtil, "Auto Raid", 130, "autoRaid")

createToggle(tabUtil, "Auto Boss", 170, "autoBoss")

local tabMisc = tabFrames["Misc"]

createToggle(tabMisc, "Velocidade Custom", 10, "velocidadeCustom")

createSlider(tabMisc, "Velocidade (WalkSpeed)", 16, 300, state.speedValor, 50, function(val)
    state.speedValor = val
    if state.velocidadeCustom then
        humanoid.WalkSpeed = val
    end
end)

createToggle(tabMisc, "Pulo Infinito", 90, "puloInfinito")

createToggle(tabMisc, "Noclip", 130, "noclip")

createToggle(tabMisc, "Auto Coletar Frutas", 170, "autoCollectFruit")

createToggle(tabMisc, "Auto Comprar Frutas", 210, "autoBuyFruit")

local logFrame = Instance.new("ScrollingFrame", mainFrame)
logFrame.Size = UDim2.new(1, -20, 0, 90)
logFrame.Position = UDim2.new(0, 10, 1, -100)
logFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 50)
logFrame.BorderSizePixel = 0
logFrame.CanvasSize = UDim2.new(0, 0, 5, 0)
logFrame.ScrollBarThickness = 5

local logListLayout = Instance.new("UIListLayout", logFrame)
logListLayout.SortOrder = Enum.SortOrder.LayoutOrder

local function addLog(text)
    local lbl = Instance.new("TextLabel", logFrame)
    lbl.Text = text
    lbl.Font = Enum.Font.SourceSans
    lbl.TextColor3 = Color3.fromRGB(180, 180, 255)
    lbl.BackgroundTransparency = 1
    lbl.Size = UDim2.new(1, -10, 0, 20)
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    logFrame.CanvasSize = UDim2.new(0, 0, 0, logListLayout.AbsoluteContentSize.Y + 20)
end

addLog("Script iniciado. Boa sorte!")

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
    for _, enemy in pairs(workspaceEnemies:GetChildren()) do
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

local function autoFarmLoop()
    while state.autofarm do
        local enemyName = state.enemies[state.selectedEnemyIndex]
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

        if state.autoCollectFruit then
            addLog("Tentando coletar fruta...")
        end

        if state.autoBuyFruit then
            addLog("Tentando comprar fruta...")
        end

        wait(1)
    end
end

coroutine.wrap(function()
    while true do
        if state.autofarm then
            autoFarmLoop()
        else
            wait(0.5)
        end
    end
end)()

coroutine.wrap(function()
    while true do
        if state.autoStats then
            autoStatsLoop()
        else
            wait(1)
        end
    end
end)()

coroutine.wrap(miscLoop)()

tabMisc.ChildAdded:Connect(function(child)
    if child:IsA("TextBox") and child.Text == tostring(state.speedValor) then
        humanoid.WalkSpeed = state.speedValor
    end
end)
