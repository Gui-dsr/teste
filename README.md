-- UtilityMenu.lua

local UIS = game:GetService("UserInputService")
local Run = game:GetService("RunService")
local CoreGui = game:GetService("CoreGui")
local Camera = workspace.CurrentCamera
local player = game.Players.LocalPlayer

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
frame.Size = UDim2.new(0, 260, 0, 200)
frame.Position = UDim2.new(0.5, -130, 0.5, -100)
frame.BackgroundColor3 = Color3.fromRGB(0,0,0)
frame.BorderSizePixel = 0

local title = Instance.new("TextLabel", frame)
title.Text = "Utility Menu"
title.Size = UDim2.new(1,0,0,40)
title.BackgroundTransparency = 1
title.Font = Enum.Font.GothamBold
title.TextSize = 24
title.TextColor3 = Color3.new(1,1,1)

-- Helper functions
local function makeButton(parent, text, posY)
    local btn = Instance.new("TextButton", parent)
    btn.Text = text
    btn.TextColor3 = Color3.new(1,1,1)
    btn.BackgroundColor3 = Color3.fromRGB(200,0,0)
    btn.Size = UDim2.new(0.9,0,0,30)
    btn.Position = UDim2.new(0.05,0,0,posY)
    btn.Font = Enum.Font.Gotham
    btn.TextSize = 18
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
    txt.TextColor3 = Color3.new(1,1,1)
    txt.BackgroundTransparency = 1
    txt.Size = UDim2.new(0.6,0,0,25)
    txt.Position = UDim2.new(0.05,0,0,y)
    txt.Font = Enum.Font.Gotham

    local function makeBtn(sign, x)
        local b = makeButton(frame, sign, y)
        b.Size = UDim2.new(0.15,0,0,25)
        b.Position = UDim2.new(x,0,0,y)
        b.MouseButton1Click:Connect(function()
            state.value = math.clamp(state.value + (sign=="+" and step or -step), min, max)
            txt.Text = label..": "..state.value
            if apply then apply(state.value) end
        end)
    end

    makeBtn("+", 0.7)
    makeBtn("-", 0.85)
end

toggleButton("ESP All Parts", 50, espState)
stepper("WalkSpeed", 90, speedState, 10, 16, 500, function(v)
    if player.Character and player.Character:FindFirstChild("Humanoid") then
        player.Character.Humanoid.WalkSpeed = v
    end
end)
stepper("Click Delay (ms)", 130, clickState, 10, 10, 1000)

frame.Visible = true

-- Input handlers
UIS.InputBegan:Connect(function(inp, gp)
    if gp then return end
    if inp.KeyCode == menuKey then frame.Visible = not frame.Visible end
    if inp.KeyCode == clickKey then autoClickState.value = not autoClickState.value end
end)

-- Auto-click loop
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

-- ESP: only show parts within 40 studs below player
local boxes = {}
Run.RenderStepped:Connect(function()
    local root = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
    if not root then return end

    for _, part in ipairs(workspace:GetDescendants()) do
        if espState.value and part:IsA("BasePart") then
            local dy = root.Position.Y - part.Position.Y
            if dy >= 0 and dy <= 40 then
                if not boxes[part] then
                    local b = Drawing.new("Square")
                    b.Thickness, b.Filled = 1, false
                    b.Color = Color3.new(1,0,0)
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
