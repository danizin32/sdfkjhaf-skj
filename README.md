-- Variáveis essenciais
local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local hrp = character:WaitForChild("HumanoidRootPart")
local UIS = game:GetService("UserInputService")
local runService = game:GetService("RunService")

local workspaceEnemies = workspace:WaitForChild("Enemies") -- Ajuste conforme o nome correto no jogo

-- Estado do script
local state = {
    AutoFarm = false,
    AutoClick = false,
    AutoStats = false,
    WalkSpeedEnabled = false,
    WalkSpeedValue = 16,
    InfiniteJump = false,
    Noclip = false,
    StatType = "Melee",
    StatAmount = 1,
    WeaponName = "Sword", -- Coloque o nome correto da arma que quer equipar
    TargetEnemy = "Bandit" -- Nome padrão do inimigo alvo
}

-- Criando a interface
local playerGui = player:WaitForChild("PlayerGui")
local screenGui = Instance.new("ScreenGui", playerGui)
screenGui.Name = "AutoFarmGUI"
screenGui.ResetOnSpawn = false

local mainFrame = Instance.new("Frame", screenGui)
mainFrame.Size = UDim2.new(0, 350, 0, 400)
mainFrame.Position = UDim2.new(0.5, -175, 0.5, -200)
mainFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
mainFrame.Active = true
mainFrame.Draggable = true

-- Botão minimizar/abrir
local toggleButton = Instance.new("TextButton", mainFrame)
toggleButton.Size = UDim2.new(1, 0, 0, 25)
toggleButton.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
toggleButton.Text = "AutoFarm Script - Clique para Minimizar"
toggleButton.TextColor3 = Color3.new(1,1,1)
toggleButton.Font = Enum.Font.GothamBold
toggleButton.TextSize = 18

local contentFrame = Instance.new("Frame", mainFrame)
contentFrame.Size = UDim2.new(1, 0, 1, -25)
contentFrame.Position = UDim2.new(0, 0, 0, 25)
contentFrame.BackgroundTransparency = 1

local minimized = false
toggleButton.MouseButton1Click:Connect(function()
    minimized = not minimized
    contentFrame.Visible = not minimized
    if minimized then
        toggleButton.Text = "AutoFarm Script - Clique para Abrir"
        mainFrame.Size = UDim2.new(0, 350, 0, 25)
    else
        toggleButton.Text = "AutoFarm Script - Clique para Minimizar"
        mainFrame.Size = UDim2.new(0, 350, 0, 400)
    end
end)

-- Abas
local tabs = {}
local buttonsFrame = Instance.new("Frame", contentFrame)
buttonsFrame.Size = UDim2.new(1, 0, 0, 40)
buttonsFrame.BackgroundTransparency = 1

local function createTabButton(name, pos)
    local btn = Instance.new("TextButton", buttonsFrame)
    btn.Size = UDim2.new(0, 110, 1, 0)
    btn.Position = UDim2.new(0, pos, 0, 0)
    btn.Text = name
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 16
    btn.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
    btn.TextColor3 = Color3.new(1,1,1)
    tabs[name] = btn
    return btn
end

local autoFarmBtn = createTabButton("Auto Farm", 0)
local autoStatsBtn = createTabButton("Auto Stats", 115)
local miscBtn = createTabButton("Misc", 230)

local tabContent = Instance.new("Frame", contentFrame)
tabContent.Size = UDim2.new(1, 0, 1, -40)
tabContent.Position = UDim2.new(0, 0, 0, 40)
tabContent.BackgroundTransparency = 1

local function clearTab()
    for _, child in pairs(tabContent:GetChildren()) do
        child:Destroy()
    end
end

local currentTab = nil

-- Função para equipar arma
local function equipWeapon(name)
    local backpack = player.Backpack
    local char = player.Character
    if char and char:FindFirstChildOfClass("Tool") and char:FindFirstChildOfClass("Tool").Name == name then
        return true
    end
    local tool = backpack:FindFirstChild(name)
    if tool then
        tool.Parent = char
        return true
    end
    return false
end

-- Função para encontrar inimigo
local function findNearestEnemy(enemyName)
    local closest = nil
    local dist = math.huge
    for _, enemy in pairs(workspaceEnemies:GetChildren()) do
        if enemy:IsA("Model") and enemy.Name:lower():find(enemyName:lower()) then
            local hum = enemy:FindFirstChild("Humanoid")
            local hrpEnemy = enemy:FindFirstChild("HumanoidRootPart")
            if hum and hum.Health > 0 and hrpEnemy then
                local distance = (hrp.Position - hrpEnemy.Position).Magnitude
                if distance < dist then
                    dist = distance
                    closest = enemy
                end
            end
        end
    end
    return closest
end

-- Função UI AutoFarm
function createAutoFarmUI()
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

    local targetLabel = Instance.new("TextLabel", tabContent)
    targetLabel.Size = UDim2.new(1, -20, 0, 25)
    targetLabel.Position = UDim2.new(0, 10, 0, y)
    targetLabel.Text = "Nome do inimigo alvo:"
    targetLabel.TextColor3 = Color3.new(1,1,1)
    targetLabel.BackgroundTransparency = 1
    targetLabel.TextXAlignment = Enum.TextXAlignment.Left
    y = y + 30

    local targetInput = Instance.new("TextBox", tabContent)
    targetInput.Size = UDim2.new(1, -20, 0, 30)
    targetInput.Position = UDim2.new(0, 10, 0, y)
    targetInput.Text = state.TargetEnemy
    targetInput.ClearTextOnFocus = false
    y = y + 50

    targetInput.FocusLost:Connect(function(enterPressed)
        if enterPressed then
            state.TargetEnemy = targetInput.Text
        end
    end)

    local weaponLabel = Instance.new("TextLabel", tabContent)
    weaponLabel.Size = UDim2.new(1, -20, 0, 25)
    weaponLabel.Position = UDim2.new(0, 10, 0, y)
    weaponLabel.Text = "Nome da arma:"
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

    toggleAutoFarm.MouseButton1Click:Connect(function()
        state.AutoFarm = not state.AutoFarm
        if state.AutoFarm then
            toggleAutoFarm.Text = "Auto Farm: ON"
            toggleAutoFarm.BackgroundColor3 = Color3.fromRGB(0, 150, 0)

            -- Equipa arma e liga autoclick automaticamente
            equipWeapon(state.WeaponName)
            state.AutoClick = true
            toggleAutoClick.Text = "AutoClick: ON"
            toggleAutoClick.BackgroundColor3 = Color3.fromRGB(0, 150, 0)
        else
            toggleAutoFarm.Text = "Auto Farm: OFF"
            toggleAutoFarm.BackgroundColor3 = Color3.fromRGB(150, 0, 0)

            state.AutoClick = false
            toggleAutoClick.Text = "AutoClick: OFF"
            toggleAutoClick.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
        end
    end)
end

-- Função UI AutoStats
function createAutoStatsUI()
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
    statAmountLabel.Text = "Quantidade para distribuir:"
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

-- Função UI Misc
function createMiscUI()
    local y = 10

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
            humanoid.WalkSpeed = state.WalkSpeedValue
        else
            toggleWalkSpeed.Text = "WalkSpeed: OFF"
            toggleWalkSpeed.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
            humanoid.WalkSpeed = 16
        end
    end)

    local walkSpeedLabel = Instance.new("TextLabel", tabContent)
    walkSpeedLabel.Size = UDim2.new(1, -20, 0, 25)
    walkSpeedLabel.Position = UDim2.new(0, 10, 0, y)
    walkSpeedLabel.Text = "Valor WalkSpeed:"
    walkSpeedLabel.TextColor3 = Color3.new(1,1,1)
    walkSpeedLabel.BackgroundTransparency = 1
    walkSpeedLabel.TextXAlignment = Enum.TextXAlignment.Left
    y = y + 30

    local walkSpeedInput = Instance.new("TextBox", tabContent)
    walkSpeedInput.Size = UDim2.new(1, -20, 0, 30)
    walkSpeedInput.Position = UDim2.new(0, 10, 0, y)
    walkSpeedInput.Text = tostring(state.WalkSpeedValue)
    walkSpeedInput.ClearTextOnFocus = false
    y = y + 50

    walkSpeedInput.FocusLost:Connect(function(enterPressed)
        if enterPressed then
            local val = tonumber(walkSpeedInput.Text)
            if val and val > 0 then
                state.WalkSpeedValue = val
                if state.WalkSpeedEnabled then
                    humanoid.WalkSpeed = val
                end
            else
                walkSpeedInput.Text = tostring(state.WalkSpeedValue)
            end
        end
    end)

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

    UIS.JumpRequest:Connect(function()
        if state.InfiniteJump then
            humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
        end
    end)

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

    runService.Stepped:Connect(function()
        if state.Noclip then
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
    end)
end

-- Função para trocar de aba e criar conteúdo
local function selectTab(name)
    clearTab()
    currentTab = name
    if name == "Auto Farm" then
        createAutoFarmUI()
    elseif name == "Auto Stats" then
        createAutoStatsUI()
    elseif name == "Misc" then
        createMiscUI()
    end
    -- Marca botão ativo
    for k, btn in pairs(tabs) do
        if k == name then
            btn.BackgroundColor3 = Color3.fromRGB(0, 150, 0)
        else
            btn.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
        end
    end
end

autoFarmBtn.MouseButton1Click:Connect(function()
    selectTab("Auto Farm")
end)
autoStatsBtn.MouseButton1Click:Connect(function()
    selectTab("Auto Stats")
end)
miscBtn.MouseButton1Click:Connect(function()
    selectTab("Misc")
end)

-- Loop do AutoClick e AutoFarm
spawn(function()
    while true do
        runService.Heartbeat:Wait()
        if state.AutoFarm then
            local enemy = findNearestEnemy(state.TargetEnemy)
            if enemy then
                local enemyHRP = enemy:FindFirstChild("HumanoidRootPart")
                if enemyHRP then
                    -- Move até perto do inimigo
                    hrp.CFrame = enemyHRP.CFrame * CFrame.new(0, 0, 5)
                    equipWeapon(state.WeaponName)
                    -- Ataque básico
                    if state.AutoClick then
                        local tool = character:FindFirstChildOfClass("Tool")
                        if tool and tool:FindFirstChild("Handle") then
                            tool:Activate() -- Tenta ativar o ataque
                        end
                    end
                end
            end
        end
        if state.AutoStats then
            -- Ajuste esta parte para a função do seu jogo, aqui tem que mandar o evento que distribui stats
            local remote = player:WaitForChild("PlayerGui"):FindFirstChild("RemoteEventName") -- Coloque o nome correto
            if remote then
                remote:FireServer(state.StatType, state.StatAmount)
            end
        end
    end
end)

-- Inicializa com a aba Auto Farm aberta
selectTab("Auto Farm")
