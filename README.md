--==== CONFIGURAÇÃO ====
local UserInputService = game:GetService("UserInputService")
local RunService        = game:GetService("RunService")
local CoreGui           = game:GetService("CoreGui")
local Camera            = workspace.CurrentCamera

local menuKey       = Enum.KeyCode.K
local holding       = false
local espRocks      = false
local espCoal       = false
local clickEnabled  = false

local walkSpeed     = 16
local clickDelay    = 0.1

--==== GUI ====
local ScreenGui = Instance.new("ScreenGui", CoreGui)
ScreenGui.Name = "UtilityMenu"

local Frame = Instance.new("Frame", ScreenGui)
Frame.Size = UDim2.new(0, 220, 0, 200)
Frame.Position = UDim2.new(0.5, -110, 0.5, -100)
Frame.BackgroundTransparency = 0.4
Frame.Visible = true

local function makeToggle(text, y, callback)
    local btn = Instance.new("TextButton", Frame)
    btn.Text = text
    btn.Size = UDim2.new(1, -20, 0, 30)
    btn.Position = UDim2.new(0, 10, 0, y)
    btn.MouseButton1Click:Connect(function()
        callback()
        btn.Text = text.." : "..(callback.flag and "ON" or "OFF")
    end)
    return btn
end

local function makeSlider(label, y, min, max, default, callback)
    local sliderLabel = Instance.new("TextLabel", Frame)
    sliderLabel.Text = label..": "..default
    sliderLabel.Size = UDim2.new(1, -20, 0, 20)
    sliderLabel.Position = UDim2.new(0, 10, 0, y)

    local slider = Instance.new("TextButton", Frame)
    slider.Size = UDim2.new(1, -20, 0, 10)
    slider.Position = UDim2.new(0, 10, 0, y+20)
    slider.BackgroundColor3 = Color3.fromRGB(200,200,200)

    local dragging = false
    slider.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = true end
    end)
    slider.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
    end)
    RunService.RenderStepped:Connect(function()
        if dragging then
            local ratio = math.clamp((UserInputService:GetMouseLocation().X - slider.AbsolutePosition.X)/slider.AbsoluteSize.X, 0, 1)
            local value = math.floor(min + (max-min)*ratio)
            callback(value)
            sliderLabel.Text = label..": "..value
        end
    end)
end

-- Toggles
local btnESP_Rocks = makeToggle("ESP Rocks", 10, function() espRocks = not espRocks; makeToggle.flag = espRocks end)
local btnESP_Coal  = makeToggle("ESP Coal", 50, function() espCoal = not espCoal; makeToggle.flag = espCoal end)

-- Sliders
makeSlider("WalkSpeed", 90, 16, 500, walkSpeed, function(v) walkSpeed = v; game.Players.LocalPlayer.Character.Humanoid.WalkSpeed = v end)
makeSlider("Click Delay (ms)", 130, 1, 500, clickDelay*1000, function(v) clickDelay = v/1000 end)

--==== KEYBINDS ====
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == menuKey then Frame.Visible = not Frame.Visible end
    if input.KeyCode == Enum.KeyCode.F then clickEnabled = not clickEnabled end
end)

--==== ESP LOGIC ====
local function drawBox(part)
    local box = Drawing.new("Square")
    box.Thickness = 1
    box.Filled = false
    box.Transparency = 1
    return box
end

local espBoxes = {}
RunService.RenderStepped:Connect(function()
    for _, obj in pairs(workspace:GetDescendants()) do
        if (espRocks and obj.Name == "Rock") or (espCoal and obj.Name == "Coal") then
            if obj:IsA("BasePart") then
                if not espBoxes[obj] then espBoxes[obj] = drawBox(obj) end
                local box = espBoxes[obj]
                local pos, onscreen = Camera:WorldToViewportPoint(obj.Position)
                if onscreen then
                    box.Visible = true
                    box.Position = Vector2.new(pos.X-25, pos.Y-25)
                    box.Size = Vector2.new(50, 50)
                else box.Visible = false end
            end
        elseif espBoxes[obj] then
            espBoxes[obj]:Remove()
            espBoxes[obj] = nil
        end
    end
end)

--==== AUTO CLICKER ====
spawn(function()
    while true do
        if clickEnabled then
            mouse1press(); task.wait(clickDelay); mouse1release()
        else
            task.wait()
        end
    end
end)
