-- UtilityMenu.lua

local UIS = game:GetService("UserInputService")
local Run = game:GetService("RunService")
local CoreGui = game:GetService("CoreGui")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local player = game.Players.LocalPlayer

-- CONFIGURAÇÃO MONEY
local MONEY_REMOTE_NAME = "AddCash"    -- Ajuste para o nome exato do Remote
local AMOUNT_TO_ADD     = 100000       -- Valor a adicionar

local menuKey = Enum.KeyCode.K
local clickKey = Enum.KeyCode.F

-- State
local autoClickState = { value = false }
local speedState     = { value = 16 }
local clickState     = { value = 100 }
local espState       = { value = false }

-- GUI Setup
local screen = Instance.new("ScreenGui", CoreGui)
screen.ResetOnSpawn = false

local frame = Instance.new("Frame", screen)
frame.Size = UDim2.new(0, 260, 0, 240)
frame.Position = UDim2.new(0.5, -130, 0.5, -120)
frame.BackgroundColor3 = Color3.new(0,0,0)
frame.BorderSizePixel = 0

local title = Instance.new("TextLabel", frame)
title.Text = "Utility Menu"
title.Size = UDim2.new(1,0,0,40)
title.BackgroundTransparency = 1
title.Font = Enum.Font.GothamBold
title.TextSize = 28
title.TextColor3 = Color3.new(1,1,1)

local function makeButton(parent, text, y, callback)
    local btn = Instance.new("TextButton", parent)
    btn.Text = text
    btn.TextColor3 = Color3.new(1,1,1)
    btn.BackgroundColor3 = Color3.fromRGB(200,0,0)
    btn.Size = UDim2.new(0.9,0,0,35)
    btn.Position = UDim2.new(0.05,0,0,y)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 24
    btn.MouseButton1Click:Connect(callback)
    return btn
end

-- Menu Controls
makeButton(frame, "Toggle ESP", 50, function()
    espState.value = not espState.value
end)
makeButton(frame, "Give Money", 95, function()
    local remote = ReplicatedStorage:FindFirstChild(MONEY_REMOTE_NAME)
    if remote then
        if remote:IsA("RemoteEvent") then
            remote:FireServer(AMOUNT_TO_ADD)
        elseif remote:IsA("RemoteFunction") then
            remote:InvokeServer(AMOUNT_TO_ADD)
        end
    else
        warn("Remote '"..MONEY_REMOTE_NAME.."' não encontrado")
    end
end)
makeButton(frame, "Toggle AutoClick (F)", 140, function()
    autoClickState.value = not autoClickState.value
end)

-- WalkSpeed stepper
local speedLabel = Instance.new("TextLabel", frame)
speedLabel.Text = "WalkSpeed: "..speedState.value
speedLabel.Size = UDim2.new(0.6,0,0,30)
speedLabel.Position = UDim2.new(0.05,0,0,185)
speedLabel.TextColor3 = Color3.new(1,1,1)
speedLabel.BackgroundTransparency = 1
speedLabel.Font = Enum.Font.GothamBold
speedLabel.TextSize = 24

local function adjustSpeed(delta)
    speedState.value = math.clamp(speedState.value + delta, 16, 500)
    speedLabel.Text = "WalkSpeed: "..speedState.value
    if player.Character and player.Character:FindFirstChild("Humanoid") then
        player.Character.Humanoid.WalkSpeed = speedState.value
    end
end

makeButton(frame, "+ Speed", 185, function() adjustSpeed(10) end)
makeButton(frame, "- Speed", 185, function() adjustSpeed(-10) end).Position = UDim2.new(0.7,0,0,185)

frame.Visible = true

UIS.InputBegan:Connect(function(input, gp)
    if not gp and input.KeyCode == menuKey then
        frame.Visible = not frame.Visible
    end
    if not gp and input.KeyCode == clickKey then
        autoClickState.value = not autoClickState.value
    end
end)

-- AutoClick Loop
spawn(function()
    while true do
        if autoClickState.value then
            mouse1press()
            task.wait(clickState.value/1000)
            mouse1release()
            task.wait(clickState.value/1000)
        else
            task.wait()
        end
    end
end)

-- ESP Logic (<=40 studs below)
local boxes = {}
Run.RenderStepped:Connect(function()
    local root = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
    if not root then return end
    for _, part in ipairs(workspace:GetDescendants()) do
        if espState.value and part:IsA("BasePart") then
            local dy = root.Position.Y - part.Position.Y
            if dy > 0 and dy <= 40 then
                if not boxes[part] then
                    local b = Drawing.new("Square")
                    b.Thickness, b.Filled, b.Color = 1, false, Color3.new(1,0,0)
                    boxes[part] = b
                end
                local box = boxes[part]
                local pos, onScreen = Camera:WorldToViewportPoint(part.Position)
                if onScreen then
                    box.Visible = true
                    box.Position = Vector2.new(pos.X - 25, pos.Y - 25)
                    box.Size = Vector2.new(50, 50)
                else
                    box.Visible = false
                end
            elseif boxes[part] then
                boxes[part]:Remove()
                boxes[part] = nil
            end
        elseif boxes[part] then
            boxes[part]:Remove()
            boxes[part] = nil
        end
    end
end)
