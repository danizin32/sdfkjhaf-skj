-- Serviços Roblox
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Configuração com valores padrão
local config = {
	esp = true,
	aimbot = false,
	teamCheck = true,
	fovSize = 100
}

-- GUI
local gui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
gui.Name = "AimbotESP_GUI"

local frame = Instance.new("Frame", gui)
frame.Position = UDim2.new(0, 10, 0, 10)
frame.Size = UDim2.new(0, 260, 0, 200)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
frame.BackgroundTransparency = 0.1
frame.BorderSizePixel = 0

local function createToggle(name, value, callback)
	local btn = Instance.new("TextButton", frame)
	btn.Size = UDim2.new(0, 240, 0, 30)
	btn.Position = UDim2.new(0, 10, 0, (#frame:GetChildren() - 1) * 35)
	btn.Text = name .. ": " .. (value and "ON" or "OFF")
	btn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)

	btn.MouseButton1Click:Connect(function()
		value = not value
		btn.Text = name .. ": " .. (value and "ON" or "OFF")
		callback(value)
	end)
end

local function createSlider(name, minValue, maxValue, defaultValue, callback)
	local text = Instance.new("TextLabel", frame)
	text.Text = name .. ": " .. tostring(defaultValue)
	text.Size = UDim2.new(0, 240, 0, 20)
	text.Position = UDim2.new(0, 10, 0, (#frame:GetChildren() - 1) * 35)
	text.TextColor3 = Color3.new(1, 1, 1)
	text.BackgroundTransparency = 1

	local inputBox = Instance.new("TextBox", frame)
	inputBox.Size = UDim2.new(0, 240, 0, 30)
	inputBox.Position = UDim2.new(0, 10, 0, (#frame:GetChildren() - 1) * 35)
	inputBox.Text = tostring(defaultValue)
	inputBox.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
	inputBox.TextColor3 = Color3.new(1,1,1)

	inputBox.FocusLost:Connect(function()
		local val = tonumber(inputBox.Text)
		if val and val >= minValue and val <= maxValue then
			text.Text = name .. ": " .. tostring(val)
			callback(val)
		else
			inputBox.Text = tostring(defaultValue)
		end
	end)
end

createToggle("ESP", config.esp, function(v) config.esp = v end)
createToggle("Aimbot", config.aimbot, function(v) config.aimbot = v end)
createToggle("Team Check", config.teamCheck, function(v) config.teamCheck = v end)
createSlider("FOV", 50, 500, config.fovSize, function(v) config.fovSize = v end)

-- Desenho do FOV
local fovCircle = Drawing.new("Circle")
fovCircle.Color = Color3.fromRGB(255, 255, 255)
fovCircle.Thickness = 1
fovCircle.Transparency = 0.5
fovCircle.Filled = false
fovCircle.Visible = true

-- ESP com nome e distância
local function updateESP(player)
	if player == LocalPlayer then return end
	local char = player.Character
	if not char then return end
	local hrp = char:FindFirstChild("HumanoidRootPart")
	if not hrp then return end

	-- Box ESP
	if not hrp:FindFirstChild("ESPBox") then
		local box = Instance.new("BoxHandleAdornment")
		box.Name = "ESPBox"
		box.Adornee = hrp
		box.Size = Vector3.new(2, 5, 1)
		box.Color3 = Color3.fromRGB(255, 0, 0)
		box.AlwaysOnTop = true
		box.ZIndex = 5
		box.Transparency = 0.5
		box.Parent = hrp
	end

	-- Texto ESP
	if not hrp:FindFirstChild("ESPName") then
		local label = Instance.new("BillboardGui")
		label.Name = "ESPName"
		label.Adornee = hrp
		label.Size = UDim2.new(0, 200, 0, 50)
		label.StudsOffset = Vector3.new(0, 3, 0)
		label.AlwaysOnTop = true

		local txt = Instance.new("TextLabel", label)
		txt.Size = UDim2.new(1, 0, 1, 0)
		txt.BackgroundTransparency = 1
		txt.TextColor3 = Color3.new(1, 1, 1)
		txt.TextStrokeTransparency = 0.5
		txt.TextScaled = true
		txt.Text = player.Name

		label.Parent = hrp
	end

	-- Atualiza distância
	local gui = hrp:FindFirstChild("ESPName")
	if gui and gui:FindFirstChildWhichIsA("TextLabel") then
		local dist = math.floor((hrp.Position - Camera.CFrame.Position).Magnitude)
		gui.TextLabel.Text = player.Name .. " [" .. dist .. "m]"
	end
end

local function clearESP(player)
	local char = player.Character
	if char and char:FindFirstChild("HumanoidRootPart") then
		local hrp = char.HumanoidRootPart
		if hrp:FindFirstChild("ESPBox") then hrp.ESPBox:Destroy() end
		if hrp:FindFirstChild("ESPName") then hrp.ESPName:Destroy() end
	end
end

-- Aimbot
local function getClosest()
	local mousePos = UIS:GetMouseLocation()
	local closest = nil
	local shortest = math.huge

	for _, player in ipairs(Players:GetPlayers()) do
		if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
			if config.teamCheck and player.Team == LocalPlayer.Team then continue end

			local head = player.Character.Head
			local screenPos, onScreen = Camera:WorldToViewportPoint(head.Position)
			if onScreen then
				local dist = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude
				if dist < config.fovSize and dist < shortest then
					shortest = dist
					closest = head.Position
				end
			end
		end
	end
	return closest
end

-- Entrada de tecla
local aiming = false
UIS.InputBegan:Connect(function(input)
	if input.KeyCode == Enum.KeyCode.E then aiming = true end
end)

UIS.InputEnded:Connect(function(input)
	if input.KeyCode == Enum.KeyCode.E then aiming = false end
end)

-- Loop principal
RunService.RenderStepped:Connect(function()
	fovCircle.Position = UIS:GetMouseLocation()
	fovCircle.Radius = config.fovSize

	for _, player in ipairs(Players:GetPlayers()) do
		if config.esp then
			updateESP(player)
		else
			clearESP(player)
		end
	end

	if config.aimbot and aiming then
		local target = getClosest()
		if target then
			Camera.CFrame = CFrame.new(Camera.CFrame.Position, target)
		end
	end
end)

-- Salvar configurações (SOMENTE fora do Roblox)
pcall(function()
	local file = "aimbot_config.txt"
	writefile(file, game:GetService("HttpService"):JSONEncode(config))
end)
