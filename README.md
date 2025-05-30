local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local hrp = character:WaitForChild("HumanoidRootPart")

local state = {
    AutoFarm = false,
    AutoClick = false,
    AutoStats = false,
    WalkSpeedEnabled = false,
    WalkSpeedValue = 50,
    InfiniteJump = false,
    Noclip = false,
    WeaponName = "Sword", -- Mude para sua arma padrão
    TargetEnemy = "EnemyName", -- Mude para o nome do inimigo alvo
    StatType = "Melee",
    StatAmount = 1,
    GuiVisible = true,
}

-- Função para equipar arma
local function equipWeapon(weaponName)
    local backpack = player:WaitForChild("Backpack")
    local tool = character:FindFirstChildOfClass("Tool") or backpack:FindFirstChild(weaponName)
    if tool then
        tool.Parent = character
    end
end

-- Função para encontrar inimigo mais próximo pelo nome
local function findNearestEnemy(enemyName)
    local nearest
    local nearestDist = math.huge
    local enemies = workspace:FindFirstChild("Enemies") or workspace -- Ajuste aqui conforme sua estrutura
    for _, v in pairs(enemies:GetChildren()) do
        if v.Name == enemyName and v:FindFirstChild("HumanoidRootPart") and v:FindFirstChildOfClass("Humanoid") and v.Humanoid.Health > 0 then
            local dist = (hrp.Position - v.HumanoidRootPart.Position).Magnitude
            if dist < nearestDist then
                nearestDist = dist
                nearest = v
            end
        end
    end
    return nearest
end

-- GUI básico
local screenGui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
screenGui.Name = "AutoFarmGUI"

local mainFrame = Instance.new("Frame", screenGui)
mainFrame.Size = UDim2.new(0, 350, 0, 400)
mainFrame.Position = UDim2.new(0, 100, 0, 100)
mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Draggable = true

-- Botão minimizar
local btnToggleGui = Instance.new("TextButton", screenGui)
btnToggleGui.Size = UDim2.new(0, 100, 0, 30)
btnToggleGui.Position = UDim2.new(0, 100, 0, 70)
btnToggleGui.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
btnToggleGui.TextColor3 = Color3.new(1,1,1)
btnToggleGui.Text = "Minimizar GUI"
btnToggleGui.Font = Enum.Font.GothamBold
btnToggleGui.TextSize = 18

btnToggleGui.MouseButton1Click:Connect(function()
    state.GuiVisible = not state.GuiVisible
    mainFrame.Visible = state.GuiVisible
    if state.GuiVisible then
        btnToggleGui.Text = "Minimizar GUI"
    else
        btnToggleGui.Text = "Abrir GUI"
    end
end)

-- Abas
local tabs = {}
local tabButtonsFrame = Instance.new("Frame", mainFrame)
tabButtonsFrame.Size = UDim2.new(1, 0, 0, 40)
tabButtonsFrame.Position = UDim2.new(0, 0, 0, 0)
tabButtonsFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)

local function createTabButton(name, posX)
    local btn = Instance.new("TextButton", tabButtonsFrame)
    btn.Size = UDim2.new(0, 110, 1, 0)
    btn.Position = UDim2.new(0, posX, 0, 0)
    btn.Text = name
    btn.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
    btn.TextColor3 = Color3.new(1,1,1)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 18
    tabs[name] = btn
    return btn
end

local autoFarmBtn = createTabButton("Auto Farm", 0)
local autoStatsBtn = createTabButton("Auto Stats", 110)
local miscBtn = createTabButton("Misc", 220)

local tabContent = Instance.new("Frame", mainFrame)
tabContent.Size = UDim2.new(1, 0, 1, -40)
tabContent.Position = UDim2.new(0, 0, 0, 40)
tabContent.BackgroundColor3 = Color3.fromRGB(20, 20, 20)

local currentTab = nil

local function clearTab()
    for _, child in pairs(tabContent:GetChildren()) do
        child:Destroy()
    end
end

-- AutoFarm UI
local function createAutoFarmUI()
    local y = 10

    local toggleAutoFarm = Instance.new("TextButton", tabContent)
    toggleAutoFarm.Size = UDim2.new(1, -20, 0, 40)
    toggleAutoFarm.Position = UDim2.new(0, 10, 0, y)
    toggleAutoFarm.Text = "Auto Farm: OFF"
    toggleAutoFarm.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
    toggleAutoFarm.TextColor3 = Color3.new(1,1,1)
    toggleAutoFarm.Font = Enum.Font.GothamBold
    toggleAutoFarm.TextSize = 18
    y = y + 50

    toggleAutoFarm.MouseButton1Click:Connect(function()
        state.AutoFarm = not state.AutoFarm
        if state.AutoFarm then
            toggleAutoFarm.Text = "Auto Farm: ON"
            toggleAutoFarm.BackgroundColor3 = Color3.fromRGB(0, 150, 0)

            -- Equipar arma e ligar AutoClick
            equipWeapon(state.WeaponName)
            state.AutoClick = true
            if toggleAutoClick then
                toggleAutoClick.Text = "AutoClick: ON"
                toggleAutoClick.BackgroundColor3 = Color3.fromRGB(0, 150, 0)
            end
        else
            toggleAutoFarm.Text = "Auto Farm: OFF"
            toggleAutoFarm.BackgroundColor3 = Color3.fromRGB(150, 0, 0)

            state.AutoClick = false
            if toggleAutoClick then
                toggleAutoClick.Text = "AutoClick: OFF"
                toggleAutoClick.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
            end
        end
    end)

    local toggleAutoClick = Instance.new("TextButton", tabContent)
    toggleAutoClick.Size = UDim2.new(1, -20, 0, 40)
    toggleAutoClick.Position = UDim2.new(0, 10, 0, y)
    toggleAutoClick.Text = "AutoClick: OFF"
    toggleAutoClick.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
    toggleAutoClick.TextColor3 = Color3.new(1,1,1)
    toggleAutoClick.Font = Enum.Font.GothamBold
    toggleAutoClick.TextSize = 18
    y = y + 50

    toggleAutoClick.MouseButton1Click:Connect(function()
        state.AutoClick = not state.AutoClick
        if state.AutoClick then
            toggleAutoClick.Text = "AutoClick: ON"
            toggleAutoClick.BackgroundColor3 = Color3.fromRGB(0, 150, 0)
        else
            toggleAutoClick.Text = "AutoClick: OFF"
            toggleAutoClick.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
        end
    end)

    local enemyLabel = Instance.new("TextLabel", tabContent)
    enemyLabel.Size = UDim2.new(1, -20, 0, 25)
    enemyLabel.Position = UDim2.new(0, 10, 0, y)
    enemyLabel.Text = "Nome do inimigo alvo:"
    enemyLabel.TextColor3 = Color3.new(1,1,1)
    enemyLabel.BackgroundTransparency = 1
    enemyLabel.TextXAlignment = Enum.TextXAlignment.Left
    y = y + 30

    local enemyInput = Instance.new("TextBox", tabContent)
    enemyInput.Size = UDim2.new(1, -20, 0, 30)
    enemyInput.Position = UDim2.new(0, 10, 0, y)
    enemyInput.Text = state.TargetEnemy
    enemyInput.ClearTextOnFocus = false
    y = y + 50

    enemyInput.FocusLost:Connect(function(enterPressed)
        if enterPressed then
            state.TargetEnemy = enemyInput.Text
        end
    end)

    local weaponLabel = Instance.new("TextLabel", tabContent)
    weaponLabel.Size = UDim2.new(1, -20, 0, 25)
    weaponLabel.Position = UDim2.new(0, 10, 0, y)
    weaponLabel.Text = "Nome da arma para equipar:"
    weaponLabel.TextColor3 = Color3.new(1,1,1)
    weaponLabel.BackgroundTransparency = 1
    weaponLabel.TextXAlignment = Enum.TextXAlignment.Left
    y = y + 30

    local weaponInput = Instance.new("TextBox", tabContent)
    weaponInput.Size = UDim2.new(1, -20, 0, 30)
    weaponInput.Position = UDim2.new(0, 10, 0, y)
    weaponInput.Text = state.WeaponName
    weaponInput.ClearTextOnFocus = false
    y = y + 50

    weaponInput.FocusLost:Connect(function(enterPressed)
        if enterPressed then
            state.WeaponName = weaponInput.Text
        end
    end)
end

-- AutoStats UI
local function createAutoStatsUI()
    local y = 10

    local toggleAutoStats = Instance.new("TextButton", tabContent)
    toggleAutoStats.Size = UDim2.new(1, -20, 0, 40)
    toggleAutoStats.Position = UDim2.new(0, 10, 0, y)
    toggleAutoStats.Text = "Auto Stats: OFF"
    toggleAutoStats.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
    toggleAutoStats.TextColor3 = Color3.new(1,1,1)
    toggleAutoStats.Font = Enum.Font.GothamBold
    toggleAutoStats.TextSize = 18
    y = y + 50

    toggleAutoStats.MouseButton1Click:Connect(function()
        state.AutoStats = not state.AutoStats
        if state.AutoStats then
            toggleAutoStats.Text = "Auto Stats: ON"
            toggleAutoStats.BackgroundColor3 = Color3.fromRGB(0, 150, 0)
        else
            toggleAutoStats.Text = "Auto Stats: OFF"
            toggleAutoStats.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
        end
    end)

    local statTypeLabel = Instance.new("TextLabel", tabContent)
    statTypeLabel.Size = UDim2.new(1, -20, 0, 25)
    statTypeLabel.Position = UDim2.new(0, 10, 0, y)
    statTypeLabel.Text = "Tipo de Stat (Melee, Defense, Sword, Gun, Fruit):"
    statTypeLabel.TextColor3 = Color3.new(1,1,1)
    statTypeLabel.BackgroundTransparency = 1
    statTypeLabel.TextXAlignment = Enum.TextXAlignment.Left
    y = y + 30

    local statTypeInput = Instance.new("TextBox", tabContent)
    statTypeInput.Size = UDim2.new(1, -20, 0, 30)
    statTypeInput.Position = UDim2.new(0, 10, 0, y)
    statTypeInput.Text = state.StatType
    statTypeInput.ClearTextOnFocus = false
    y = y + 50

    statTypeInput.FocusLost:Connect(function(enterPressed)
        if enterPressed then
            state.StatType = statTypeInput.Text
        end
    end)

    local statAmountLabel = Instance.new("TextLabel", tabContent)
    statAmountLabel.Size = UDim2.new(1, -20, 0, 25)
    statAmountLabel.Position = UDim2.new(0, 10, 0, y)
    statAmountLabel.Text = "Quantidade por vez (ex: 1):"
    statAmountLabel.TextColor3 = Color3.new(1,1,1)
    statAmountLabel.BackgroundTransparency = 1
    statAmountLabel.TextXAlignment = Enum.TextXAlignment.Left
    y = y + 30

    local statAmountInput = Instance.new("TextBox", tabContent)
    statAmountInput.Size = UDim2.new(1, -20, 0, 30)
    statAmountInput.Position = UDim2.new(0, 10, 0, y)
    statAmountInput.Text = tostring(state.StatAmount)
    statAmountInput.ClearTextOnFocus = false
    y = y + 50

    statAmountInput.FocusLost:Connect(function(enterPressed)
        if enterPressed then
            local val = tonumber(statAmountInput.Text)
            if val and val > 0 then
                state.StatAmount = val
            else
                statAmountInput.Text = tostring(state.StatAmount)
            end
        end
    end)
end

-- Misc UI
local function createMiscUI()
    local y = 10

    -- WalkSpeed Toggle
    local toggleWalkSpeed = Instance.new("TextButton", tabContent)
    toggleWalkSpeed.Size = UDim2.new(1, -20, 0, 40)
    toggleWalkSpeed.Position = UDim2.new(0, 10, 0, y)
    toggleWalkSpeed.Text = "WalkSpeed: OFF"
    toggleWalkSpeed.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
    toggleWalkSpeed.TextColor3 = Color3.new(1,1,1)
    toggleWalkSpeed.Font = Enum.Font.GothamBold
    toggleWalkSpeed.TextSize = 18
    y = y + 50

    toggleWalkSpeed.MouseButton1Click:Connect(function()
        state.WalkSpeedEnabled = not state.WalkSpeedEnabled
        if state.WalkSpeedEnabled then
            toggleWalkSpeed.Text = "WalkSpeed: ON"
            toggleWalkSpeed.BackgroundColor3 = Color3.fromRGB(0, 150, 0)
        else
            toggleWalkSpeed.Text = "WalkSpeed: OFF"
            toggleWalkSpeed.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
            if humanoid then
                humanoid.WalkSpeed = 16
            end
        end
    end)

    local speedLabel = Instance.new("TextLabel", tabContent)
    speedLabel.Size = UDim2.new(1, -20, 0, 25)
    speedLabel.Position = UDim2.new(0, 10, 0, y)
    speedLabel.Text = "Velocidade (WalkSpeed):"
    speedLabel.TextColor3 = Color3.new(1,1,1)
    speedLabel.BackgroundTransparency = 1
    speedLabel.TextXAlignment = Enum.TextXAlignment.Left
    y = y + 30

    local speedInput = Instance.new("TextBox", tabContent)
    speedInput.Size = UDim2.new(1, -20, 0, 30)
    speedInput.Position = UDim2.new(0, 10, 0, y)
    speedInput.Text = tostring(state.WalkSpeedValue)
    speedInput.ClearTextOnFocus = false
    y = y + 50

    speedInput.FocusLost:Connect(function(enterPressed)
        if enterPressed then
            local val = tonumber(speedInput.Text)
            if val and val >= 16 then
                state.WalkSpeedValue = val
                if state.WalkSpeedEnabled and humanoid then
                    humanoid.WalkSpeed = val
                end
            else
                speedInput.Text = tostring(state.WalkSpeedValue)
            end
        end
    end)

    -- Infinite Jump Toggle
    local toggleInfiniteJump = Instance.new("TextButton", tabContent)
    toggleInfiniteJump.Size = UDim2.new(1, -20, 0, 40)
    toggleInfiniteJump.Position = UDim2.new(0, 10, 0, y)
    toggleInfiniteJump.Text = "Infinite Jump: OFF"
    toggleInfiniteJump.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
    toggleInfiniteJump.TextColor3 = Color3.new(1,1,1)
    toggleInfiniteJump.Font = Enum.Font.GothamBold
    toggleInfiniteJump.TextSize = 18
    y = y + 50

    toggleInfiniteJump.MouseButton1Click:Connect(function()
        state.InfiniteJump = not state.InfiniteJump
        if state.InfiniteJump then
            toggleInfiniteJump.Text = "Infinite Jump: ON"
            toggleInfiniteJump.BackgroundColor3 = Color3.fromRGB(0, 150, 0)
        else
            toggleInfiniteJump.Text = "Infinite Jump: OFF"
            toggleInfiniteJump.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
        end
    end)

    -- Noclip Toggle
    local toggleNoclip = Instance.new("TextButton", tabContent)
    toggleNoclip.Size = UDim2.new(1, -20, 0, 40)
    toggleNoclip.Position = UDim2.new(0, 10, 0, y)
    toggleNoclip.Text = "Noclip: OFF"
    toggleNoclip.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
    toggleNoclip.TextColor3 = Color3.new(1,1,1)
    toggleNoclip.Font = Enum.Font.GothamBold
    toggleNoclip.TextSize = 18
    y = y + 50

    toggleNoclip.MouseButton1Click:Connect(function()
        state.Noclip = not state.Noclip
        if state.Noclip then
            toggleNoclip.Text = "Noclip: ON"
            toggleNoclip.BackgroundColor3 = Color3.fromRGB(0, 150, 0)
        else
            toggleNoclip.Text = "Noclip: OFF"
            toggleNoclip.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
        end
    end)
end

-- Trocar de aba
local function selectTab(tabName)
    clearTab()
    currentTab = tabName
    if tabName == "Auto Farm" then
        createAutoFarmUI()
    elseif tabName == "Auto Stats" then
        createAutoStatsUI()
    elseif tabName == "Misc" then
        createMiscUI()
    end
end

autoFarmBtn.MouseButton1Click:Connect(function() selectTab("Auto Farm") end)
autoStatsBtn.MouseButton1Click:Connect(function() selectTab("Auto Stats") end)
miscBtn.MouseButton1Click:Connect(function() selectTab("Misc") end)

selectTab("Auto Farm")

-- Loop para AutoFarm
spawn(function()
    while true do
        RunService.Heartbeat:Wait()
        if state.AutoFarm then
            local enemy = findNearestEnemy(state.TargetEnemy)
            if enemy and enemy.HumanoidRootPart and enemy:FindFirstChildOfClass("Humanoid") then
                hrp.CFrame = CFrame.new(enemy.HumanoidRootPart.Position) * CFrame.new(0, 0, 3) -- Fica 3 studs atrás do inimigo
                -- Aqui pode adicionar ataque automático ou outras ações
            end
        end
    end
end)

-- Loop AutoClick
spawn(function()
    while true do
        RunService.Heartbeat:Wait()
        if state.AutoClick then
            local tool = character:FindFirstChildOfClass("Tool")
            if tool then
                pcall(function()
                    tool:Activate()
                end)
            end
        end
    end
end)

-- Loop AutoStats (exemplo, ajuste para seu jogo)
spawn(function()
    while true do
        RunService.Heartbeat:Wait(1)
        if state.AutoStats then
            -- Exemplo: enviar evento remoto para distribuir stats
            local remote = player:FindFirstChild("RemoteEventName") or player.PlayerGui:FindFirstChild("RemoteEventName") -- Ajuste nome
            if remote then
                remote:FireServer(state.StatType, state.StatAmount)
            end
        end
    end
end)

-- WalkSpeed aplicado todo frame quando ativado
RunService.Heartbeat:Connect(function()
    if humanoid then
        if state.WalkSpeedEnabled then
            humanoid.WalkSpeed = state.WalkSpeedValue
        else
            humanoid.WalkSpeed = 16
        end
    end
end)

-- Infinite Jump
UIS.JumpRequest:Connect(function()
    if state.InfiniteJump and humanoid and humanoid:GetState() ~= Enum.HumanoidStateType.Freefall then
        humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
    end
end)

-- Noclip
RunService.Stepped:Connect(function()
    if state.Noclip and character then
        for _, part in pairs(character:GetChildren()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
    elseif character then
        for _, part in pairs(character:GetChildren()) do
            if part:IsA("BasePart") then
                part.CanCollide = true
            end
        end
    end
end)

-- Atualizar humanoid quando personagem troca
player.CharacterAdded:Connect(function(char)
    character = char
    humanoid = character:WaitForChild("Humanoid")
    hrp = character:WaitForChild("HumanoidRootPart")
end)
