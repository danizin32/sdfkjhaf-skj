local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local hrp = character:WaitForChild("HumanoidRootPart")
local workspaceEnemies = workspace:FindFirstChild("Enemies") or workspace
local runService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")

local state = {
    AutoFarm = false,
    AutoEquip = false,
    WeaponName = "Katana",
    TargetEnemy = "Bandit",
    AutoStats = false,
    StatType = "Melee",
    StatAmount = 1,
    WalkSpeedEnabled = false,
    WalkSpeedValue = 100,
    InfiniteJump = false,
    Noclip = false,
    AutoClick = false,
}

-- Criar GUI
local gui = Instance.new("ScreenGui", player.PlayerGui)
gui.Name = "AutoFarmGUI"
gui.ResetOnSpawn = false

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 400, 0, 400)
frame.Position = UDim2.new(0, 20, 0.5, -200)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
frame.BorderSizePixel = 0

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, 0, 0, 40)
title.BackgroundColor3 = Color3.fromRGB(50, 50, 100)
title.Text = "Blox Fruits - AutoFarm Test Script"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Font = Enum.Font.GothamBold
title.TextSize = 16

-- Botão minimizar/abrir
local toggleVisibilityBtn = Instance.new("TextButton", title)
toggleVisibilityBtn.Size = UDim2.new(0, 40, 1, 0)
toggleVisibilityBtn.Position = UDim2.new(1, -40, 0, 0)
toggleVisibilityBtn.Text = "_"
toggleVisibilityBtn.Font = Enum.Font.GothamBold
toggleVisibilityBtn.TextSize = 24
toggleVisibilityBtn.TextColor3 = Color3.new(1,1,1)
toggleVisibilityBtn.BackgroundColor3 = Color3.fromRGB(20,20,80)

local isVisible = true
toggleVisibilityBtn.MouseButton1Click:Connect(function()
    isVisible = not isVisible
    for _, child in pairs(frame:GetChildren()) do
        if child ~= title then
            child.Visible = isVisible
        end
    end
end)

local tabContainer = Instance.new("Frame", frame)
tabContainer.Size = UDim2.new(1, 0, 0, 30)
tabContainer.Position = UDim2.new(0, 0, 0, 40)
tabContainer.BackgroundTransparency = 1

local contentFrame = Instance.new("Frame", frame)
contentFrame.Size = UDim2.new(1, -10, 1, -80)
contentFrame.Position = UDim2.new(0, 5, 0, 75)
contentFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
contentFrame.BorderSizePixel = 0

local tabs = {}
local tabNames = {"Auto Farm", "Auto Stats", "Misc"}

for i, name in ipairs(tabNames) do
    local tabBtn = Instance.new("TextButton", tabContainer)
    tabBtn.Size = UDim2.new(1/#tabNames, 0, 1, 0)
    tabBtn.Position = UDim2.new((i-1)/#tabNames, 0, 0, 0)
    tabBtn.Text = name
    tabBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    tabBtn.TextColor3 = Color3.new(1,1,1)
    tabBtn.Font = Enum.Font.Gotham
    tabBtn.TextSize = 14
    tabBtn.Name = name:gsub(" ", "").."Tab"
    tabs[name] = tabBtn
end

local activeTab = nil
local function selectTab(name)
    if activeTab then
        tabs[activeTab].BackgroundColor3 = Color3.fromRGB(40,40,40)
    end
    activeTab = name
    tabs[name].BackgroundColor3 = Color3.fromRGB(0,120,255)
    contentFrame:ClearAllChildren()
    if name == "Auto Farm" then
        createAutoFarmUI()
    elseif name == "Auto Stats" then
        createAutoStatsUI()
    elseif name == "Misc" then
        createMiscUI()
    end
end

function createAutoFarmUI()
    local y = 10

    local label = Instance.new("TextLabel", contentFrame)
    label.Size = UDim2.new(1, -20, 0, 25)
    label.Position = UDim2.new(0, 10, 0, y)
    label.Text = "Nome do Inimigo:"
    label.TextColor3 = Color3.new(1,1,1)
    label.BackgroundTransparency = 1
    label.TextXAlignment = Enum.TextXAlignment.Left
    y = y + 30

    local enemyInput = Instance.new("TextBox", contentFrame)
    enemyInput.Size = UDim2.new(1, -20, 0, 30)
    enemyInput.Position = UDim2.new(0, 10, 0, y)
    enemyInput.Text = state.TargetEnemy
    enemyInput.ClearTextOnFocus = false
    y = y + 40

    enemyInput.FocusLost:Connect(function(enterPressed)
        if enterPressed then
            state.TargetEnemy = enemyInput.Text
        end
    end)

    local label2 = Instance.new("TextLabel", contentFrame)
    label2.Size = UDim2.new(1, -20, 0, 25)
    label2.Position = UDim2.new(0, 10, 0, y)
    label2.Text = "Nome da Arma:"
    label2.TextColor3 = Color3.new(1,1,1)
    label2.BackgroundTransparency = 1
    label2.TextXAlignment = Enum.TextXAlignment.Left
    y = y + 30

    local weaponInput = Instance.new("TextBox", contentFrame)
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

    local toggleAutoFarm = Instance.new("TextButton", contentFrame)
    toggleAutoFarm.Size = UDim2.new(1, -20, 0, 40)
    toggleAutoFarm.Position = UDim2.new(0, 10, 0, y)
    toggleAutoFarm.Text = "Auto Farm: OFF"
    toggleAutoFarm.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
    toggleAutoFarm.TextColor3 = Color3.new(1,1,1)
    toggleAutoFarm.Font = Enum.Font.GothamBold
    toggleAutoFarm.TextSize = 18

    toggleAutoFarm.MouseButton1Click:Connect(function()
        state.AutoFarm = not state.AutoFarm
        if state.AutoFarm then
            toggleAutoFarm.Text = "Auto Farm: ON"
            toggleAutoFarm.BackgroundColor3 = Color3.fromRGB(0, 150, 0)
        else
            toggleAutoFarm.Text = "Auto Farm: OFF"
            toggleAutoFarm.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
        end
    end)

    local toggleAutoEquip = Instance.new("TextButton", contentFrame)
    toggleAutoEquip.Size = UDim2.new(1, -20, 0, 40)
    toggleAutoEquip.Position = UDim2.new(0, 10, 0, y + 50)
    toggleAutoEquip.Text = "Auto Equip Arma: OFF"
    toggleAutoEquip.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
    toggleAutoEquip.TextColor3 = Color3.new(1,1,1)
    toggleAutoEquip.Font = Enum.Font.GothamBold
    toggleAutoEquip.TextSize = 18

    toggleAutoEquip.MouseButton1Click:Connect(function()
        state.AutoEquip = not state.AutoEquip
        if state.AutoEquip then
            toggleAutoEquip.Text = "Auto Equip Arma: ON"
            toggleAutoEquip.BackgroundColor3 = Color3.fromRGB(0, 150, 0)
        else
            toggleAutoEquip.Text = "Auto Equip Arma: OFF"
            toggleAutoEquip.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
        end
    end)

    local toggleAutoClick = Instance.new("TextButton", contentFrame)
    toggleAutoClick.Size = UDim2.new(1, -20, 0, 40)
    toggleAutoClick.Position = UDim2.new(0, 10, 0, y + 100)
    toggleAutoClick.Text = "AutoClick: OFF"
    toggleAutoClick.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
    toggleAutoClick.TextColor3 = Color3.new(1,1,1)
    toggleAutoClick.Font = Enum.Font.GothamBold
    toggleAutoClick.TextSize = 18

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
end

function createAutoStatsUI()
    local y = 10

    local label = Instance.new("TextLabel", contentFrame)
    label.Size = UDim2.new(1, -20, 0, 25)
    label.Position = UDim2.new(0, 10, 0, y)
    label.Text = "Tipo de Stat (Melee, Defense, Sword, etc):"
    label.TextColor3 = Color3.new(1,1,1)
    label.BackgroundTransparency = 1
    label.TextXAlignment = Enum.TextXAlignment.Left
    y = y + 30

    local statInput = Instance.new("TextBox", contentFrame)
    statInput.Size = UDim2.new(1, -20, 0, 30)
    statInput.Position = UDim2.new(0, 10, 0, y)
    statInput.Text = state.StatType
    statInput.ClearTextOnFocus = false
    y = y + 40

    statInput.FocusLost:Connect(function(enterPressed)
        if enterPressed then
            state.StatType = statInput.Text
        end
    end)

    local label2 = Instance.new("TextLabel", contentFrame)
    label2.Size = UDim2.new(1, -20, 0, 25)
    label2.Position = UDim2.new(0, 10, 0, y)
    label2.Text = "Quantidade de Pontos:"
    label2.TextColor3 = Color3.new(1,1,1)
    label2.BackgroundTransparency = 1
    label2.TextXAlignment = Enum.TextXAlignment.Left
    y = y + 30

    local amountInput = Instance.new("TextBox", contentFrame)
    amountInput.Size = UDim2.new(1, -20, 0, 30)
    amountInput.Position = UDim2.new(0, 10, 0, y)
    amountInput.Text = tostring(state.StatAmount)
    amountInput.ClearTextOnFocus = false
    y = y + 50

    amountInput.FocusLost:Connect(function(enterPressed)
        if enterPressed then
            local num = tonumber(amountInput.Text)
            if num then
                state.StatAmount = math.floor(num)
            else
                amountInput.Text = tostring(state.StatAmount)
            end
        end
    end)

    local toggleAutoStats = Instance.new("TextButton", contentFrame)
    toggleAutoStats.Size = UDim2.new(1, -20, 0, 40)
    toggleAutoStats.Position = UDim2.new(0, 10, 0, y)
    toggleAutoStats.Text = "Auto Stats: OFF"
    toggleAutoStats.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
    toggleAutoStats.TextColor3 = Color3.new(1,1,1)
    toggleAutoStats.Font = Enum.Font.GothamBold
    toggleAutoStats.TextSize = 18

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
end

function createMiscUI()
    local y = 10

    local toggleWalkSpeed = Instance.new("TextButton", contentFrame)
    toggleWalkSpeed.Size = UDim2.new(1, -20, 0, 40)
    toggleWalkSpeed.Position = UDim2.new(0, 10, 0, y)
    toggleWalkSpeed.Text = "WalkSpeed: OFF"
    toggleWalkSpeed.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
    toggleWalkSpeed.TextColor3 = Color3.new(1,1,1)
    toggleWalkSpeed.Font = Enum.Font.GothamBold
    toggleWalkSpeed.TextSize = 18

    toggleWalkSpeed.MouseButton1Click:Connect(function()
        state.WalkSpeedEnabled = not state.WalkSpeedEnabled
        if state.WalkSpeedEnabled then
            toggleWalkSpeed.Text = "WalkSpeed: ON"
            toggleWalkSpeed.BackgroundColor3 = Color3.fromRGB(0, 150, 0)
        else
            toggleWalkSpeed.Text = "WalkSpeed: OFF"
            toggleWalkSpeed.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
            humanoid.WalkSpeed = 16
        end
    end)

    y = y + 50

    local speedSliderLabel = Instance.new("TextLabel", contentFrame)
    speedSliderLabel.Size = UDim2.new(1, -20, 0, 25)
    speedSliderLabel.Position = UDim2.new(0, 10, 0, y)
    speedSliderLabel.Text = "WalkSpeed Valor: "..state.WalkSpeedValue
    speedSliderLabel.TextColor3 = Color3.new(1,1,1)
    speedSliderLabel.BackgroundTransparency = 1
    speedSliderLabel.TextXAlignment = Enum.TextXAlignment.Left
    y = y + 30

    local speedSlider = Instance.new("TextBox", contentFrame)
    speedSlider.Size = UDim2.new(1, -20, 0, 30)
    speedSlider.Position = UDim2.new(0, 10, 0, y)
    speedSlider.Text = tostring(state.WalkSpeedValue)
    speedSlider.ClearTextOnFocus = false
    y = y + 40

    speedSlider.FocusLost:Connect(function(enterPressed)
        if enterPressed then
            local val = tonumber(speedSlider.Text)
            if val and val > 16 and val <= 500 then
                state.WalkSpeedValue = val
                speedSliderLabel.Text = "WalkSpeed Valor: "..val
            else
                speedSlider.Text = tostring(state.WalkSpeedValue)
            end
        end
    end)

    local toggleInfiniteJump = Instance.new("TextButton", contentFrame)
    toggleInfiniteJump.Size = UDim2.new(1, -20, 0, 40)
    toggleInfiniteJump.Position = UDim2.new(0, 10, 0, y)
    toggleInfiniteJump.Text = "Infinite Jump: OFF"
    toggleInfiniteJump.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
    toggleInfiniteJump.TextColor3 = Color3.new(1,1,1)
    toggleInfiniteJump.Font = Enum.Font.GothamBold
    toggleInfiniteJump.TextSize = 18

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

    y = y + 50

    local toggleNoclip = Instance.new("TextButton", contentFrame)
    toggleNoclip.Size = UDim2.new(1, -20, 0, 40)
    toggleNoclip.Position = UDim2.new(0, 10, 0, y)
    toggleNoclip.Text = "NoClip: OFF"
    toggleNoclip.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
    toggleNoclip.TextColor3 = Color3.new(1,1,1)
    toggleNoclip.Font = Enum.Font.GothamBold
    toggleNoclip.TextSize = 18

    toggleNoclip.MouseButton1Click:Connect(function()
        state.Noclip = not state.Noclip
        if state.Noclip then
            toggleNoclip.Text = "NoClip: ON"
            toggleNoclip.BackgroundColor3 = Color3.fromRGB(0, 150, 0)
        else
            toggleNoclip.Text = "NoClip: OFF"
            toggleNoclip.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
        end
    end)
end

-- Setup inicial: seleciona primeira aba
selectTab("Auto Farm")

-- Controla troca de abas pelo clique
for name, btn in pairs(tabs) do
    btn.MouseButton1Click:Connect(function()
        selectTab(name)
    end)
end

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

-- Função para encontrar o inimigo mais próximo pelo nome
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

-- Função para atacar inimigo
local function attackEnemy(enemy)
    if not enemy then return end
    local enemyHRP = enemy:FindFirstChild("HumanoidRootPart")
    if not enemyHRP then return end
    -- Aproximar-se
    hrp.CFrame = enemyHRP.CFrame * CFrame.new(0, 0, 5)
    -- Equipar arma
    if state.AutoEquip then
        equipWeapon(state.WeaponName)
    end
end

-- Loop para ataque e autoclick
coroutine.wrap(function()
    while true do
        if state.AutoFarm then
            local enemy = findNearestEnemy(state.TargetEnemy)
            if enemy then
                attackEnemy(enemy)
                if state.AutoClick then
                    -- Simular clique para atacar
                    pcall(function()
                        game:GetService("VirtualUser"):Button1Down(Vector2.new(0,0))
                        wait(0.1)
                        game:GetService("VirtualUser"):Button1Up(Vector2.new(0,0))
                    end)
                end
            else
                wait(1)
            end
        else
            wait(0.5)
        end
        wait(0.3)
    end
end)()

-- Função para Auto Stats
local function autoStats()
    if not state.AutoStats then return end
    local remote = player:FindFirstChild("RemoteEvent") or game:GetService("ReplicatedStorage"):FindFirstChild("RemoteEvent")
    if not remote then return end
    for i = 1, state.StatAmount do
        pcall(function()
            remote:FireServer("AddPoint", state.StatType)
            wait(0.2)
        end)
    end
end

-- Loop para Auto Stats
coroutine.wrap(function()
    while true do
        if state.AutoStats then
            autoStats()
        end
        wait(1)
    end
end)()

-- Velocidade
runService.Heartbeat:Connect(function()
    if state.WalkSpeedEnabled then
        humanoid.WalkSpeed = state.WalkSpeedValue
    else
        humanoid.WalkSpeed = 16
    end
end)

-- Pulo Infinito
UIS.JumpRequest:Connect(function()
    if state.InfiniteJump then
        humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
    end
end)

-- NoClip
runService.Heartbeat:Connect(function()
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

print("AutoFarm Script carregado com sucesso!")
