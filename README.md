-- CONFIG
local TOGGLE_KEY = Enum.KeyCode.G

-- SERVICES
local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")

-- PLAYER
local player = Players.LocalPlayer

-- REMOTES
local AutoFarmEvent = ReplicatedStorage:WaitForChild("AutoFarmEvent")
local LockpickEvent = ReplicatedStorage:WaitForChild("LockpickEvent")

-- STATE
local autoFarmEnabled = false

-- GUI
local gui = Instance.new("ScreenGui", player.PlayerGui)
gui.Name = "TestMenu"

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.fromScale(0.25, 0.2)
frame.Position = UDim2.fromScale(0.05, 0.7)
frame.BackgroundColor3 = Color3.fromRGB(30,30,30)
frame.BorderSizePixel = 0

local corner = Instance.new("UICorner", frame)
corner.CornerRadius = UDim.new(0, 12)

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.fromScale(1, 0.3)
title.BackgroundTransparency = 1
title.Text = "RIO RP - TEST MENU"
title.TextColor3 = Color3.new(1,1,1)
title.TextScaled = true
title.Font = Enum.Font.GothamBold

local status = Instance.new("TextLabel", frame)
status.Position = UDim2.fromScale(0, 0.3)
status.Size = UDim2.fromScale(1, 0.25)
status.BackgroundTransparency = 1
status.Text = "Auto Farm: OFF"
status.TextColor3 = Color3.new(1,0,0)
status.TextScaled = true
status.Font = Enum.Font.Gotham

local lockpickBtn = Instance.new("TextButton", frame)
lockpickBtn.Position = UDim2.fromScale(0.1, 0.6)
lockpickBtn.Size = UDim2.fromScale(0.8, 0.3)
lockpickBtn.Text = "LOCKPICK CARRO"
lockpickBtn.TextScaled = true
lockpickBtn.BackgroundColor3 = Color3.fromRGB(50,50,50)
lockpickBtn.TextColor3 = Color3.new(1,1,1)
lockpickBtn.Font = Enum.Font.GothamBold

local btnCorner = Instance.new("UICorner", lockpickBtn)
btnCorner.CornerRadius = UDim.new(0, 8)

-- AUTO FARM LOOP
task.spawn(function()
	while true do
		if autoFarmEnabled then
			-- avisa o servidor para dar dinheiro / job / ação
			AutoFarmEvent:FireServer()
		end
		task.wait(1.5)
	end
end)

-- TOGGLE COM G
UIS.InputBegan:Connect(function(input, gp)
	if gp then return end
	if input.KeyCode == TOGGLE_KEY then
		autoFarmEnabled = not autoFarmEnabled

		if autoFarmEnabled then
			status.Text = "Auto Farm: ON"
			status.TextColor3 = Color3.new(0,1,0)
		else
			status.Text = "Auto Farm: OFF"
			status.TextColor3 = Color3.new(1,0,0)
		end
	end
end)

-- LOCKPICK
lockpickBtn.MouseButton1Click:Connect(function()
	-- Envia pedido de lockpick para o servidor
	LockpickEvent:FireServer()
end)
