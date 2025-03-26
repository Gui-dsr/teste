-- CONFIGURAÇÕES INICIAIS
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")

local toggleKey = Enum.KeyCode.F
local menuKey  = Enum.KeyCode.Insert
local holding = false
local menuVisible = true

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

-- Botão para setar keybind toggle
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

-- Botão para setar keybind menu
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

-- === Funcionalidades ===
local function toggleHold()
    holding = not holding
    if holding then mouse1press() else mouse1release() end
end

-- Keybind handlers
UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end

    if input.KeyCode == toggleKey then
        toggleHold()
    elseif input.KeyCode == menuKey then
        menuVisible = not menuVisible
        Frame.Visible = menuVisible
    end
end)

-- Loop para manter o clique pressionado
spawn(function()
    while true do
        if holding then mouse1press() end
        task.wait()
    end
end)
