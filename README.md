-- UtilityMenu.lua (Atualizado Completo)

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
local autoClick     = { value = false }
local speedState    = { value = 16 }
local clickState    = { value = 100 }
local espState      = { value = false }
local LevelUtil     = { value = 1 }
local UpgradeUtil   = { value = 1 }

-- GUI Setup
local screen = Instance.new("ScreenGui", CoreGui)
screen.ResetOnSpawn = false

local frame = Instance.new("Frame", screen)
frame.Size = UDim2.new(0, 260, 0, 380)
frame.Position = UDim2.new(0.5, -130, 0.5, -190)
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

-- Title
local title = Instance.new("TextLabel", frame)
title.Text = "Utility Menu"
title.Size = UDim2.new(1,0,0,40)
title.Position = UDim2.new(0,0,0,0)
title.BackgroundTransparency = 1
title.Font = Enum.Font.GothamBold
title.TextSize = 28
title.TextColor3 = Color3.new(1,1,1)

-- Buttons
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

-- WalkSpeed Stepper
local speedLabel = Instance.new("TextLabel", frame)
speedLabel.Text = "WalkSpeed: "..speedState.value
speedLabel.TextColor3 = Color3.new(1,1,1)
speedLabel.BackgroundTransparency = 1
speedLabel.Size = UDim2.new(0.6,0,0,30)
speedLabel.Position = UDim2.new(0.05,0,0,255)
speedLabel.Font = Enum.Font.GothamBold
speedLabel.TextSize = 24

makeButton("+ Speed", 255, function()
    speedState.value = math.clamp(speedState.value + 10, 16, 500)
    speedLabel.Text = "WalkSpeed: "..speedState.value
    if player.Character and player.Character:FindFirstChild("Humanoid") then
        player.Character.Humanoid.WalkSpeed = speedState.value
    end
end).Size = UDim2.new(0.4,0,0,30)

makeButton("- Speed", 255, function()
    speedState.value = math.clamp(speedState.value - 10, 16, 500)
    speedLabel.Text = "WalkSpeed: "..speedState.value
    if player.Character and player.Character:FindFirstChild("Humanoid") then
        player.Character.Humanoid.WalkSpeed = speedState.value
    end
end).Position = UDim2.new(0.55,0,0,255)

-- Click Delay Stepper
local clickLabel = Instance.new("TextLabel", frame)
clickLabel.Text = "Click Delay: "..clickState.value.."ms"
clickLabel.TextColor3 = Color3.new(1,1,1)
clickLabel.BackgroundTransparency = 1
clickLabel.Size = UDim2.new(0.6,0,0,30)
clickLabel.Position = UDim2.new(0.05,0,0,300)
clickLabel.Font = Enum.Font.GothamBold
clickLabel.TextSize = 24

makeButton("+ Delay", 300, function()
    clickState.value = math.clamp(clickState.value + 10, 10, 1000)
    clickLabel.Text = "Click Delay: "..clickState.value.."ms"
end).Size = UDim2.new(0.4,0,0,30)

makeButton("- Delay", 300, function()
    clickState.value = math.clamp(clickState.value - 10, 10, 1000)
    clickLabel.Text = "Click Delay: "..clickState.value.."ms"
end).Position = UDim2.new(0.55,0,0,300)

-- LevelUtil Stepper
local levelLabel = Instance.new("TextLabel", frame)
levelLabel.Text = "LevelUtil: "..LevelUtil.value
levelLabel.TextColor3 = Color3.new(1,1,1)
levelLabel.BackgroundTransparency = 1
levelLabel.Size = UDim2.new(0.6,0,0,30)
levelLabel.Position = UDim2.new(0.05,0,0,345)
levelLabel.Font = Enum.Font.GothamBold
levelLabel.TextSize = 24

makeButton("+ Level", 345, function()
    LevelUtil.value += 1
    levelLabel.Text = "LevelUtil: "..LevelUtil.value
end).Size = UDim2.new(0.4,0,0,30)

makeButton("- Level", 345, function()
    LevelUtil.value = math.max(1, LevelUtil.value - 1)
    levelLabel.Text = "LevelUtil: "..LevelUtil.value
end).Position = UDim2.new(0.55,0,0,345)

-- UpgradeUtil Stepper
local upgradeLabel = Instance.new("TextLabel", frame)
upgradeLabel.Text = "UpgradeUtil: "..UpgradeUtil.value
upgradeLabel.TextColor3 = Color3.new(1,1,1)
upgradeLabel.BackgroundTransparency = 1
upgradeLabel.Size = UDim2.new(0.6,0,0,30)
upgradeLabel.Position = UDim2.new(0.05,0,0,390)
upgradeLabel.Font = Enum.Font.GothamBold
upgradeLabel.TextSize = 24

makeButton("+ Upgrade", 390, function()
    UpgradeUtil.value += 1
    upgradeLabel.Text = "UpgradeUtil: "..UpgradeUtil.value
end).Size = UDim2.new(0.4,0,0,30)

makeButton("- Upgrade", 390, function()
    UpgradeUtil.value = math.max(1, UpgradeUtil.value - 1)
    upgradeLabel.Text = "UpgradeUtil: "..UpgradeUtil.value
end).Position = UDim2.new(0.55,0,0,390)

frame.Visible = true

-- Input handlers
UIS.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed then
        if input.KeyCode == menuKey then frame.Visible = not frame.Visible end
        if input.KeyCode == clickKey then autoClick.value = not autoClick.value end
    end
end)

-- Auto-click loop
spawn(function()
    while true do
        if autoClick.value then
            mouse1press()
            task.wait(clickState.value/1000)
            mouse1release()
            task.wait(clickState.value/1000)
        else
            task.wait()
        end
    end
end)

-- ESP logic: only BaseParts 40 studs below player
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
