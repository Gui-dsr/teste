-- UtilityMenu.lua

local UIS         = game:GetService("UserInputService")
local Run         = game:GetService("RunService")
local CoreGui     = game:GetService("CoreGui")
local Camera      = workspace.CurrentCamera
local player      = game.Players.LocalPlayer

local menuKey     = Enum.KeyCode.K
local clickKey    = Enum.KeyCode.F

-- State
local autoClickState = { value = false }
local speedState     = { value = 16 }
local clickState     = { value = 100 }
local espState       = { value = false }
local rangeState     = { value = 10 }
local pickupState    = { value = 1 }
local cashState      = { value = 0 }

-- Require dynamically
local MineUtil, CashUtil
for _, mod in ipairs(game.ReplicatedStorage:GetDescendants()) do
    if mod:IsA("ModuleScript") then
        if mod.Name == "MineUtil" then
            MineUtil = require(mod)
            rangeState.value = MineUtil.Range or rangeState.value
        elseif mod.Name == "CashUtil" then
            CashUtil = require(mod)
        end
    end
end

-- GUI Setup
local screen = Instance.new("ScreenGui", CoreGui)
screen.ResetOnSpawn = false

local frame = Instance.new("Frame", screen)
frame.Size = UDim2.new(0, 260, 0, 300)
frame.Position = UDim2.new(0.5, -130, 0.5, -150)
frame.BackgroundColor3 = Color3.new(0,0,0)
frame.BorderSizePixel = 0

local title = Instance.new("TextLabel", frame)
title.Text = "Utility Menu"
title.Size = UDim2.new(1,0,0,40)
title.BackgroundTransparency = 1
title.Font = Enum.Font.GothamBold
title.TextSize = 28
title.TextColor3 = Color3.new(1,1,1)

local function makeButton(parent, text, y)
    local btn = Instance.new("TextButton", parent)
    btn.Text = text
    btn.TextColor3 = Color3.new(1,1,1)
    btn.BackgroundColor3 = Color3.fromRGB(200,0,0)
    btn.Size = UDim2.new(0.9,0,0,35)
    btn.Position = UDim2.new(0.05,0,0,y)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 24
    return btn
end

local function toggleButton(label, y, state)
    local btn = makeButton(frame, label..": OFF", y)
    btn.MouseButton1Click:Connect(function()
        state.value = not state.value
        btn.Text = label..(state.value and ": ON" or ": OFF")
    end)
end

local function stepper(label, y, state, step, min, max, apply)
    local txt = Instance.new("TextLabel", frame)
    txt.Text = label..": "..state.value
    txt.BackgroundTransparency = 1
    txt.Size = UDim2.new(0.6,0,0,30)
    txt.Position = UDim2.new(0.05,0,0,y)
    txt.Font = Enum.Font.GothamBold
    txt.TextSize = 24
    txt.TextColor3 = Color3.new(1,1,1)

local function makeBtn(sign, x)
        local btn = makeButton(frame, sign, y)
        btn.Size = UDim2.new(0.15,0,0,30)
        btn.Position = UDim2.new(x,0,0,y)
        btn.MouseButton1Click:Connect(function()
            state.value = math.clamp(state.value + (sign=="+" and step or -step), min, max)
            txt.Text = label..": "..state.value
            apply(state.value)
        end)
    end

makeBtn("+", 0.7)
makeBtn("-", 0.85)
end

-- UI
toggleButton("ESP All Parts", 50, espState)
stepper("WalkSpeed", 90, speedState, 10, 16, 500, function(v)
    if player.Character and player.Character:FindFirstChild("Humanoid") then
        player.Character.Humanoid.WalkSpeed = v
    end
end)
stepper("Click Delay (ms)", 140, clickState, 10, 10, 1000, function() end)

if MineUtil then
    stepper("Mining Range", 180, rangeState, 5, 10, 500, function(v)
        MineUtil.Range = v
    end)
end

stepper("PickUp Speed", 220, pickupState, 0.1, 0.1, 10, function(v)
    if player.Character and player.Character.PrimaryPart then
        player.Character.PrimaryPart:SetAttribute("PickUpSpeed", v)
    end
end)

if CashUtil then
    stepper("Set Cash", 260, cashState, 100, 0, 1e7, function(v)
        CashUtil.SetCash(CashUtil, v)
    end)
end

frame.Visible = true

UIS.InputBegan:Connect(function(input, gp)
    if not gp then
        if input.KeyCode == menuKey then frame.Visible = not frame.Visible end
        if input.KeyCode == clickKey then autoClickState.value = not autoClickState.value end
    end
end)

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

local boxes = {}
Run.RenderStepped:Connect(function()
    local root = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
    if not root then return end

for _, part in ipairs(workspace:GetDescendants()) do
        if espState.value and part:IsA("BasePart") then
            local pos, onScreen = Camera:WorldToViewportPoint(part.Position)
            if onScreen then
                if not boxes[part] then
                    local box = Drawing.new("Square")
                    box.Thickness, box.Filled = 1, false
                    box.Color = Color3.new(1,0,0)
                    boxes[part] = box
                end
                local box = boxes[part]
                box.Visible = true
                box.Position = Vector2.new(pos.X - 25, pos.Y - 25)
                box.Size = Vector2.new(50, 50)
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
