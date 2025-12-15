edStorage = game:GetService("ReplicatedStorage")
local ContextActionService = game:GetService("ContextActionService")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Wait helper
local function waitEvent(name, timeout)
    local ok, ev = pcall(function() return ReplicatedStorage:WaitForChild(name, timeout or 10) end)
    if not ok or not ev then
        warn("[Client] RemoteEvent não encontrado:", name)
        return nil
    end
    return ev
end

local RequestOpenLockpickEvent = waitEvent("RequestOpenLockpick")
local OpenLockpickGuiEvent = waitEvent("OpenLockpickGui")
local AttemptLockpickEvent = waitEvent("AttemptLockpick")
local LockpickResultEvent = waitEvent("LockpickResult")
local ClientNotificationEvent = waitEvent("ClientNotification")
local ToggleAutoFarmEvent = waitEvent("ToggleAutoFarm") -- opcional

-- UI build
if playerGui:FindFirstChild("LockpickUI") then playerGui.LockpickUI:Destroy() end
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "LockpickUI"
screenGui.Parent = playerGui
screenGui.ResetOnSpawn = false

local frame = Instance.new("Frame", screenGui)
frame.Size = UDim2.new(0, 280, 0, 140)
frame.Position = UDim2.new(0, 12, 0, 70)
frame.BackgroundColor3 = Color3.fromRGB(22,22,22)
frame.BackgroundTransparency = 0
local corner = Instance.new("UICorner", frame); corner.CornerRadius = UDim.new(0,8)

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, -16, 0, 24); title.Position = UDim2.new(0,8,0,6)
title.BackgroundTransparency = 1; title.Text = "Ferramentas"; title.TextColor3 = Color3.fromRGB(230,230,230)
title.Font = Enum.Font.SourceSansSemibold; title.TextSize = 18; title.TextXAlignment = Enum.TextXAlignment.Left

local btnOpen = Instance.new("TextButton", frame)
btnOpen.Size = UDim2.new(0.94, 0, 0, 36); btnOpen.Position = UDim2.new(0.03,0,0,36)
btnOpen.Text = "Abrir Lockpick"; btnOpen.BackgroundColor3 = Color3.fromRGB(100,80,200); btnOpen.TextColor3 = Color3.fromRGB(245,245,245)

local btnToggleAF = Instance.new("TextButton", frame)
btnToggleAF.Size = UDim2.new(0.44,0,0,28); btnToggleAF.Position = UDim2.new(0.03,0,0,82)
btnToggleAF.Text = "Toggle AutoFarm (G)"; btnToggleAF.BackgroundColor3 = Color3.fromRGB(45,150,255); btnToggleAF.TextColor3 = Color3.new(1,1,1)

local status = Instance.new("TextLabel", frame)
status.Size = UDim2.new(1, -16, 0, 18); status.Position = UDim2.new(0,8,0,112)
status.BackgroundTransparency = 1; status.Text = ""; status.TextColor3 = Color3.fromRGB(200,200,200)

-- Minigame frame (popup)
local mg = Instance.new("Frame", screenGui)
mg.Size = UDim2.new(0, 420, 0, 140); mg.Position = UDim2.new(0.5, -210, 0.66, -70); mg.AnchorPoint = Vector2.new(0.5,0)
mg.BackgroundColor3 = Color3.fromRGB(18,18,18); mg.Visible = false
local mgCorner = Instance.new("UICorner", mg); mgCorner.CornerRadius = UDim.new(0,10)

local mgTitle = Instance.new("TextLabel", mg)
mgTitle.Size = UDim2.new(1, -24, 0, 28); mgTitle.Position = UDim2.new(0,12,0,8)
mgTitle.BackgroundTransparency = 1; mgTitle.Text = "Lockpick"; mgTitle.TextColor3 = Color3.fromRGB(230,230,230)
mgTitle.Font = Enum.Font.SourceSansSemibold; mgTitle.TextSize = 18

local bar = Instance.new("Frame", mg)
bar.Size = UDim2.new(0.92,0,0,36); bar.Position = UDim2.new(0.04,0,0,46); bar.BackgroundColor3 = Color3.fromRGB(200,200,200)
local barCorner = Instance.new("UICorner", bar); barCorner.CornerRadius = UDim.new(0,6)

local mover = Instance.new("Frame", bar)
mover.Size = UDim2.new(0.06,0,1,0); mover.Position = UDim2.new(0,0,0,0); mover.BackgroundColor3 = Color3.fromRGB(255,80,80)
local moverCorner = Instance.new("UICorner", mover); moverCorner.CornerRadius = UDim.new(0,4)

local target = Instance.new("Frame", bar)
target.Size = UDim2.new(0.14,0,1,0); target.Position = UDim2.new(0.43,0,0,0)
target.BackgroundColor3 = Color3.fromRGB(100,220,120); target.Transparency = 0.15
local targetCorner = Instance.new("UICorner", target); targetCorner.CornerRadius = UDim.new(0,6)

local mgInfo = Instance.new("TextLabel", mg)
mgInfo.Size = UDim2.new(1, -24, 0, 28); mgInfo.Position = UDim2.new(0,12,0,88)
mgInfo.BackgroundTransparency = 1; mgInfo.Text = "Clique para tentar"; mgInfo.TextColor3 = Color3.fromRGB(230,230,230)
mgInfo.Font = Enum.Font.SourceSans; mgInfo.TextSize = 16

-- minigame state
local running = false
local dir = 1
local speed = 0.9
local currentCar = nil

local function openMinigame(carName)
    currentCar = carName
    mg.Visible = true
    running = true
    mover.Position = UDim2.new(0,0,0,0)
    dir = 1
    mgInfo.Text = "Clique para tentar"
    status.Text = ""
end
local function closeMinigame()
    running = false
    currentCar = nil
    mg.Visible = false
end

-- Bindings
btnOpen.MouseButton1Click:Connect(function()
    if RequestOpenLockpickEvent then
        RequestOpenLockpickEvent:FireServer()
        status.Text = "Pedido enviado..."
    else
        status.Text = "Evento RequestOpenLockpick indisponível."
    end
end)

if ToggleAutoFarmEvent then
    ContextActionService:BindAction("ToggleAutoFarmKey", function(_, state)
        if state == Enum.UserInputState.Begin then
            ToggleAutoFarmEvent:FireServer()
            status.Text = "AutoFarm toggle solicitado..."
        end
    end, false, Enum.KeyCode.G)

    btnToggleAF.MouseButton1Click:Connect(function()
        ToggleAutoFarmEvent:FireServer()
        status.Text = "AutoFarm toggle solicitado..."
    end)
else
    btnToggleAF.Visible = false
end

bar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 and running and currentCar then
        running = false
        local moverCenter = mover.AbsolutePosition.X + mover.AbsoluteSize.X * 0.5
        local targetStart = target.AbsolutePosition.X
        local targetEnd = target.AbsolutePosition.X + target.AbsoluteSize.X
        local inside = moverCenter >= targetStart and moverCenter <= targetEnd
        if AttemptLockpickEvent then
            AttemptLockpickEvent:FireServer(currentCar, inside)
        end
        closeMinigame()
        status.Text = "Tentando lockpick..."
    end
end)

RunService.Heartbeat:Connect(function(dt)
    if running then
        local pos = mover.Position.X.Scale
        pos = pos + dir * speed * dt
        if pos <= 0 then pos = 0; dir = 1
        elseif pos + mover.Size.X.Scale >= 1 then pos = 1 - mover.Size.X.Scale; dir = -1 end
        mover.Position = UDim2.new(pos,0,0,0)
    end
end)

-- Server events
OpenLockpickGuiEvent.OnClientEvent:Connect(function(carNameOrFalse, maybeMsg)
    if carNameOrFalse == false then
        status.Text = maybeMsg or "Nenhum carro por perto."
        delay(2, function() if status then status.Text = "" end end)
        return
    end
    openMinigame(carNameOrFalse)
end)

LockpickResultEvent.OnClientEvent:Connect(function(success, message)
    status.Text = message or (success and "Desbloqueado!" or "Falhou.")
    delay(2, function() if status then status.Text = "" end end)
end)

if ClientNotificationEvent then
    ClientNotificationEvent.OnClientEvent:Connect(function(msg)
        status.Text = tostring(msg)
        delay(1.8, function() if status then status.Text = "" end end)
    end)
end

-- ESC fecha minigame
UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.KeyCode == Enum.KeyCode.Escape and mg.Visible then
        closeMinigame()
    end
end)

print("[Client] Lockpick UI carregada.")
