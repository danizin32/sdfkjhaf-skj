-- GUI confirmada funcionando anteriormente
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local HttpService = game:GetService("HttpService")

local config = {
    esp = true,
    aimbot = true,
    teamCheck = true,
    fovSize = 120
}

local configFile = "mobile_config.json"
if isfile and isfile(configFile) then
    local success, data = pcall(function()
        return HttpService:JSONDecode(readfile(configFile))
    end)
    if success then
        for k,v in pairs(data) do config[k] = v end
    end
end

local function saveConfig()
    if writefile then
        writefile(configFile, HttpService:JSONEncode(config))
    end
end

-- Criar GUI
local gui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
gui.Name = "MainMobileGUI"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true

local function createButton(txt, yPos, cb)
    local btn = Instance.new("TextButton", gui)
    btn.Size = UDim2.new(0.3, 0, 0.07, 0)
    btn.Position = UDim2.new(0.02, 0, yPos, 0)
    btn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    btn.Text = txt
    btn.TextColor3 = Color3.new(1,1,1)
    btn.TextScaled = true
    btn.MouseButton1Click:Connect(cb)
end

createButton("ESP", 0.05, function()
    config.esp = not config.esp
    saveConfig()
end)

createButton("Aimbot", 0.13, function()
    config.aimbot = not config.aimbot
    saveConfig()
end)

createButton("Team Check", 0.21, function()
    config.teamCheck = not config.teamCheck
    saveConfig()
end)

createButton("FOV +", 0.29, function()
    config.fovSize += 20
    saveConfig()
end)

createButton("FOV -", 0.37, function()
    config.fovSize = math.max(20, config.fovSize - 20)
    saveConfig()
end)

-- Botão de aimbot
local aimBtn = Instance.new("TextButton", gui)
aimBtn.Size = UDim2.new(0.2, 0, 0.12, 0)
aimBtn.Position = UDim2.new(0.77, 0, 0.83, 0)
aimBtn.BackgroundColor3 = Color3.fromRGB(90,0,0)
aimBtn.Text = "AIM"
aimBtn.TextColor3 = Color3.new(1,1,1)
aimBtn.TextScaled = true

local aiming = false
aimBtn.MouseButton1Down:Connect(function() aiming = true end)
aimBtn.MouseButton1Up:Connect(function() aiming = false end)

-- Função de ESP
function addESP(player)
    local function espLoop()
        RunService.RenderStepped:Connect(function()
            if not config.esp then return end
            if player and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                local hrp = player.Character.HumanoidRootPart
                local gui = hrp:FindFirstChild("ESP") or Instance.new("BillboardGui", hrp)
                gui.Name = "ESP"
                gui.Size = UDim2.new(0, 100, 0, 20)
                gui.AlwaysOnTop = true
                gui.StudsOffset = Vector3.new(0, 3, 0)

                local label = gui:FindFirstChild("Text") or Instance.new("TextLabel", gui)
                label.Name = "Text"
                label.Size = UDim2.new(1, 0, 1, 0)
                label.BackgroundTransparency = 1
                label.TextColor3 = Color3.new(1,1,1)
                label.TextStrokeTransparency = 0.5
                label.TextScaled = true

                local distance = math.floor((Camera.CFrame.Position - hrp.Position).Magnitude)
                label.Text = player.Name .. " ["..distance.."m]"
            end
        end)
    end
    espLoop()
end

-- Adicionar ESP a jogadores existentes
for _,p in ipairs(Players:GetPlayers()) do
    if p ~= LocalPlayer then
        addESP(p)
    end
end

-- Aimbot simples
function getClosestTarget()
    local shortest = config.fovSize
    local closest = nil
    for _,p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("Head") then
            if config.teamCheck and p.Team == LocalPlayer.Team then continue end
            local pos, onScreen = Camera:WorldToViewportPoint(p.Character.Head.Position)
            if onScreen then
                local dist = (Vector2.new(pos.X, pos.Y) - Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)).Magnitude
                if dist < shortest then
                    shortest = dist
                    closest = p.Character.Head
                end
            end
        end
    end
    return closest
end

RunService.RenderStepped:Connect(function()
    if aiming and config.aimbot then
        local target = getClosestTarget()
        if target then
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, target.Position)
        end
    end
end)
