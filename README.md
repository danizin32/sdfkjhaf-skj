local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

-- Aguarda PlayerGui carregar no mobile
local playerGui = LocalPlayer:WaitForChild("PlayerGui", 5)

-- Garante que não duplique
if playerGui:FindFirstChild("MobileTestGUI") then
    playerGui:FindFirstChild("MobileTestGUI"):Destroy()
end

-- Cria a GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "MobileTestGUI"
screenGui.ResetOnSpawn = false
screenGui.IgnoreGuiInset = true
screenGui.Parent = playerGui

-- Botão de teste
local button = Instance.new("TextButton")
button.Size = UDim2.new(0.3, 0, 0.1, 0)
button.Position = UDim2.new(0.05, 0, 0.1, 0)
button.Text = "FUNCIONANDO!"
button.TextScaled = true
button.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
button.TextColor3 = Color3.new(1, 1, 1)
button.Parent = screenGui
