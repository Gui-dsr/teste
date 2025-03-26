-- UtilityMenu.lua

local UIS = game:GetService("UserInputService")
local Run = game:GetService("RunService")
local CoreGui = game:GetService("CoreGui")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local player = game.Players.LocalPlayer

-- CONFIGURAÇÕES
local MONEY_REMOTE_NAME = "AddCash"
local AMOUNT_TO_ADD     = 100000
local MINE_TOOL_FOLDER  = ReplicatedStorage:WaitForChild("MineTools")

local menuKey = Enum.KeyCode.K
local clickKey = Enum.KeyCode.F

-- States
local autoClick   = { value = false }
local speedState  = { value = 16 }
local clickState  = { value = 100 }
local espState    = { value = false }
local LevelUtil   = { value = 1 }
local UpgradeUtil = { value = 1 }

-- GUI Setup
local screen = Instance.new("ScreenGui", CoreGui)
screen.ResetOnSpawn = false

local frame = Instance.new("Frame", screen)
frame.Size = UDim2.new(0, 260, 0, 440)
frame.Position = UDim2.new(0.5, -130, 0.5, -220)
frame.BackgroundColor3 = Color3.new(0,0,0)
frame.BorderSizePixel = 0

local function makeButton(text, y, callback)
    local btn = Instance.new("TextButton", frame)
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

local title = Instance.new("TextLabel", frame)
title.Text = "Utility Menu"
title.Size = UDim2.new(1,0,0,40)
title.Position = UDim2.new(0,0,0,0)
title.BackgroundTransparency = 1
title.Font = Enum.Font.GothamBold
title.TextSize = 28
title.TextColor3 = Color3.new(1,1,1)

makeButton("Toggle ESP", 50, function() espState.value = not espState.value end)
makeButton("Give Money", 100, function()
    local remote = ReplicatedStorage:FindFirstChild(MONEY_REMOTE_NAME)
    if remote then
        if remote:IsA("RemoteEvent") then remote:FireServer(AMOUNT_TO_ADD)
        elseif remote:IsA("RemoteFunction") then remote:InvokeServer(AMOUNT_TO_ADD) end
    end
end)
makeButton("Give All MineTools", 150, function()
    for _, tool in ipairs(MINE_TOOL_FOLDER:GetChildren()) do
        tool:Clone().Parent = player.Backpack
    end
end)
makeButton("Toggle AutoClick (F)", 200, function() autoClick.value = not autoClick.value end)

-- WalkSpeed
local function makeStepper(label, posY, state, step, min, max, apply)
    local lbl = Instance.new("TextLabel", frame)
    lbl.Text = label..": "..state.value
    lbl.TextColor3 = Color3.new(1,1,1)
    lbl.BackgroundTransparency = 1
    lbl.Size = UDim2.new(0.6,0,0,30)
    lbl.Position = UDim2.new(0.05,0,0,posY)
    lbl.Font = Enum.Font.GothamBold
    lbl.TextSize = 24

    local inc = makeButton("+", posY, function()
        state.value = math.clamp(state.value + step, min, max)
        lbl.Text = label..": "..state.value
        if apply then apply(state.value) end
    end)
    inc.Size = UDim2.new(0.4,0,0,30)

    local dec = makeButton("-", posY, function()
        state.value = math.clamp(state.value - step, min, max)
        lbl.Text = label..": "..state.value
        if apply then apply(state.value) end
    end)
    dec.Position = UDim2.new(0.55,0,0,posY)
    dec.Size = UDim2.new(0.4,0,0,30)
end

makeStepper("WalkSpeed", 250, speedState, 10, 16, 500, function(v)
    if player.Character and player.Character:FindFirstChild("Humanoid") then
        player.Character.Humanoid.WalkSpeed = v
    end
end)
makeStepper("Click Delay (ms)", 300, clickState, 10, 10, 1000)

makeStepper("LevelUtil", 350, LevelUtil, 1, 1, math.huge)
makeStepper("UpgradeUtil", 400, UpgradeUtil, 1, 1, math.huge)

frame.Visible = true

UIS.InputBegan:Connect(function(input, gp)
    if not gp then
        if input.KeyCode == menuKey then frame.Visible = not frame.Visible end
        if input.KeyCode == clickKey then autoClick.value = not autoClick.value end
    end
end)

-- AutoClick
spawn(function()
    while true do
        if autoClick.value then
            mouse1press()
            task.wait(clickState.value/1000)
            mouse1release()
            task.wait(clickState.value/1000)
        else task.wait() end
    end
end)

-- ESP (only BaseParts 40 studs below)
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
                local pos,on = Camera:WorldToViewportPoint(part.Position)
                if on then
                    box.Visible = true
                    box.Position = Vector2.new(pos.X-25,pos.Y-25)
                    box.Size = Vector2.new(50,50)
                else box.Visible = false end
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
