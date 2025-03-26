-- CONFIGURAÇÕES INICIAIS
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")

local toggleKey = Enum.KeyCode.F
local menuKey  = Enum.KeyCode.Insert
local holding = false
local menuVisible = true

-- Delay entre cada clique (em segundos)
local clickDelay = 0.1

-- === GUI ===
local ScreenGui = Instance.new("ScreenGui", CoreGui)
ScreenGui.Name = "AutoClickerMenu"

local Frame = Instance.new("Frame", ScreenGui)
Frame.Size = UDim2.new(0, 200, 0, 120)
Frame.Position = UDim2.new(0.5, -100, 0.5, -60)
Frame.BackgroundTransparency = 0.3

local function createBindButton(text, y, callback)
    local btn = Instance.new("TextButton", Frame)
    btn.Text = text
    btn.Size = UDim2.new(1, -20, 0, 30)
    btn.Position = UDim2.new(0, 10, 0, y)
    btn.MouseButton1Click:Connect(callback)
    return btn
end

createBindButton("Set Toggle Key ("..toggleKey.Name..")", 10, function(btn)
    btn.Text = "Pressione qualquer tecla..."
    local conn
    conn = UserInputService.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.Keyboard then
            toggleKey = input.KeyCode
            btn.Text = "Set Toggle Key ("..toggleKey.Name..")"
            conn:Disconnect()
        end
    end)
end)

createBindButton("Set Menu Key ("..menuKey.Name..")", 50, function(btn)
    btn.Text = "Pressione qualquer tecla..."
    local conn
    conn = UserInputService.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.Keyboard then
            menuKey = input.KeyCode
            btn.Text = "Set Menu Key ("..menuKey.Name..")"
            conn:Disconnect()
        end
    end)
end)

-- Função de toggle
local function toggleHold()
    holding = not holding
end

UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.KeyCode == toggleKey then
        toggleHold()
    elseif input.KeyCode == menuKey then
        menuVisible = not menuVisible
        Frame.Visible = menuVisible
    end
end)

-- Loop de autoclick com delay
spawn(function()
    while true do
        if holding then
            mouse1press()
            task.wait(clickDelay)
            mouse1release()
            task.wait(clickDelay)
        else
            task.wait()
        end
    end
end)
