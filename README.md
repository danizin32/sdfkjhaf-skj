-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- CONFIGURAÇÃO INICIAL
local config = {
	esp = true,
	aimbot = true,
	teamCheck = true,
	fovSize = 100
}

-- GUI
local gui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
gui.Name = "AimbotESP_GUI"
gui.ResetOnSpawn = false

-- Janela principal
local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 220, 0, 150)
frame.Position = UDim2.new(0, 10, 0, 10)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
frame.BorderSizePixel = 0

-- Função de botão
local function createToggle(name, key, y)
	local button = Instance.new("TextButton", frame)
	button.Position = UDim2.new(0, 10, 0, y)
	button.Size = UDim2.new(0, 200, 0, 30)
	button.TextColor3 = Color3.new(1, 1, 1)
	button.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
	button.Text = name .. ": ON"

	button.MouseButton1Click:Connect(function()
		config[key] = not config[key]
		button.Text = name .. ": " .. (config[key] and "ON" or "OFF")
	end)
end

createToggle("ESP", "esp", 10)
createToggle("Aimbot", "aimbot", 50)
createToggle("Team Check", "teamCheck", 90)

-- FOV Circle (apenas visual no Studio)
local fovCircle = Drawing.new("Circle")
fovCircle.Thickness = 1
fovCircle.NumSides = 64
fovCircle.Radius = config.fovSize
fovCircle.Color = Color3.new(1, 1, 1)
fovCircle.Filled = false
fovCircle.Transparency = 1
fovCircle.Visible = true

-- Aimbot
local function getClosestPlayer()
	local mouse = UIS:GetMouseLocation()
	local closestPlayer, closestDistance = nil, config.fovSize

	for _, player in ipairs(Players:GetPlayers()) do
		if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
			if config.teamCheck and player.Team == LocalPlayer.Team then continue end

			local head = player.Character.Head
			local screenPoint, onScreen = Camera:WorldToViewportPoint(head.Position)
			local distance = (Vector2.new(screenPoint.X, screenPoint.Y) - mouse).Magnitude

			if onScreen and distance < closestDistance then
				closestDistance = distance
				closestPlayer = head
			end
		end
	end

	return closestPlayer
end

-- ESP
local function drawESP(player)
	local char = player.Character
	if not char then return end
	local hrp = char:FindFirstChild("HumanoidRootPart")
	if not hrp or hrp:FindFirstChild("ESPLabel") then return end

	local gui = Instance.new("BillboardGui", hrp)
	gui.Name = "ESPLabel"
	gui.Size = UDim2.new(0, 200, 0, 50)
	gui.StudsOffset = Vector3.new(0, 3, 0)
	gui.AlwaysOnTop = true

	local label = Instance.new("TextLabel", gui)
	label.Size = UDim2.new(1, 0, 1, 0)
	label.BackgroundTransparency = 1
	label.TextColor3 = Color3.new(1, 1, 1)
	label.TextStrokeTransparency = 0.5
	label.TextScaled = true
	label.Text = player.Name
end

-- Atualiza distância
local function updateESP(player)
	if not player.Character then return end
	local hrp = player.Character:FindFirstChild("HumanoidRootPart")
	local gui = hrp and hrp:FindFirstChild("ESPLabel")
	if gui and gui:FindFirstChildWhichIsA("TextLabel") then
		local dist = math.floor((hrp.Position - Camera.CFrame.Position).Magnitude)
		gui.TextLabel.Text = player.Name .. " [" .. dist .. "m]"
	end
end

-- Tecla de ativação do aimbot
local aiming = false
UIS.InputBegan:Connect(function(input)
	if input.KeyCode == Enum.KeyCode.E then
		aiming = true
	end
end)

UIS.InputEnded:Connect(function(input)
	if input.KeyCode == Enum.KeyCode.E then
		aiming = false
	end
end)

-- Loop principal
RunService.RenderStepped:Connect(function()
	fovCircle.Position = UIS:GetMouseLocation()
	fovCircle.Radius = config.fovSize

	for _, player in ipairs(Players:GetPlayers()) do
		if player ~= LocalPlayer then
			if config.esp then
				drawESP(player)
				updateESP(player)
			end
		end
	end

	if config.aimbot and aiming then
		local target = getClosestPlayer()
		if target then
			Camera.CFrame = CFrame.new(Camera.CFrame.Position, target.Position)
		end
	end
end)
