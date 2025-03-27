local player = game.Players.LocalPlayer
local mouse = player:GetMouse()

-- Menu Principal
local ScreenGui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
ScreenGui.Name = "ExploitMenu"

local Frame = Instance.new("Frame", ScreenGui)
Frame.Size = UDim2.new(0, 220, 0, 360)
Frame.Position = UDim2.new(0, 10, 0, 10)
Frame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
Frame.BackgroundTransparency = 0.1
Frame.Active = true
Frame.Draggable = true
Frame.BorderSizePixel = 0
Frame.Visible = true

local UICorner = Instance.new("UICorner", Frame)
UICorner.CornerRadius = UDim.new(0, 10)

local UIListLayout = Instance.new("UIListLayout", Frame)
UIListLayout.Padding = UDim.new(0, 6)
UIListLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
UIListLayout.VerticalAlignment = Enum.VerticalAlignment.Center

-- Botão Minimizar/Maximizar
local toggleBtn = Instance.new("TextButton", ScreenGui)
toggleBtn.Size = UDim2.new(0, 40, 0, 40)
toggleBtn.Position = UDim2.new(0, 240, 0, 10)
toggleBtn.BackgroundColor3 = Color3.fromRGB(255, 85, 0)
toggleBtn.Text = "–"
toggleBtn.TextSize = 28
toggleBtn.Font = Enum.Font.SourceSansBold
toggleBtn.TextColor3 = Color3.new(1,1,1)

local toggleCorner = Instance.new("UICorner", toggleBtn)
toggleCorner.CornerRadius = UDim.new(0, 10)

toggleBtn.MouseButton1Click:Connect(function()
    Frame.Visible = not Frame.Visible
    toggleBtn.Text = Frame.Visible and "–" or "+"
end)

-- Criador de Botões
local function criarBotao(texto, acao)
    local botao = Instance.new("TextButton", Frame)
    botao.Size = UDim2.new(0, 200, 0, 35)
    botao.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
    botao.TextColor3 = Color3.fromRGB(255, 255, 255)
    botao.Font = Enum.Font.SourceSansBold
    botao.TextSize = 17
    botao.Text = texto
    
    local corner = Instance.new("UICorner", botao)
    corner.CornerRadius = UDim.new(0, 6)

    botao.MouseButton1Click:Connect(acao)
end

-- Variáveis Globais
local autoMine = false
local mineInterval = 0.1
local mineCoroutine = nil
local humanoid = player.Character:WaitForChild("Humanoid")

-- Função para Mineração Automática
local function startAutoMine()
    if mineCoroutine then coroutine.close(mineCoroutine) end
    mineCoroutine = coroutine.create(function()
        while autoMine do
            mouse1press()
            wait(mineInterval)
            mouse1release()
        end
    end)
    coroutine.resume(mineCoroutine)
end

-- Atalho tecla F
mouse.KeyDown:Connect(function(key)
    if key:lower() == "f" then
        autoMine = not autoMine
        if autoMine then
            startAutoMine()
        else
            if mineCoroutine then coroutine.close(mineCoroutine) end
            mouse1release()
        end
    end
end)

-- Botão Minerar Automático
criarBotao("Minerar Auto (F)", function()
    autoMine = not autoMine
    if autoMine then
        startAutoMine()
    else
        if mineCoroutine then coroutine.close(mineCoroutine) end
        mouse1release()
    end
end)

-- Ajuste Velocidade Mineração
criarBotao("⬆️ Velocidade Mine", function()
    mineInterval = math.max(0.01, mineInterval - 0.05)
end)

criarBotao("⬇️ Velocidade Mine", function()
    mineInterval = mineInterval + 0.05
end)

-- Ajuste Velocidade Personagem
criarBotao("⬆️ Velocidade Player", function()
    humanoid.WalkSpeed = humanoid.WalkSpeed + 10
end)

criarBotao("⬇️ Velocidade Player", function()
    humanoid.WalkSpeed = math.max(16, humanoid.WalkSpeed - 10)
end)

-- Duplicar Ferramentas na Backpack
criarBotao("Duplicar Ferramentas", function()
    local backpack = player.Backpack
    for _, tool in pairs(backpack:GetChildren()) do
        if tool:IsA("Tool") then
            tool:Clone().Parent = backpack
        end
    end
end)

-- Alcance Máximo Ferramentas (Tools)
criarBotao("Alcance Máximo Tools", function()
    local backpack = player.Backpack
    for _, tool in pairs(backpack:GetChildren()) do
        local range = tool:FindFirstChild("Range")
        if range and range:IsA("NumberValue") then
            range.Value = 10000
        end
    end
end)

-- Revelar Minérios no Mapa
criarBotao("Revelar Minérios", function()
    local oresFolder = workspace:FindFirstChild("SpawnedOres")
    if oresFolder then
        for _, ore in pairs(oresFolder:GetChildren()) do
            if ore:IsA("Part") then
                if not ore:FindFirstChildOfClass("Highlight") then
                    local highlight = Instance.new("Highlight")
                    highlight.FillColor = Color3.fromRGB(255, 0, 0)
                    highlight.OutlineTransparency = 0
                    highlight.FillTransparency = 0.5
                    highlight.Parent = ore
                end
            end
        end
    end
end)

-- Teleportar para Minério mais próximo
criarBotao("Teleportar p/ Minério", function()
    local oresFolder = workspace:FindFirstChild("SpawnedOres")
    local character = player.Character
    local rootPart = character and character:FindFirstChild("HumanoidRootPart")

    if oresFolder and rootPart then
        local nearestOre, shortestDist = nil, math.huge
        for _, ore in pairs(oresFolder:GetChildren()) do
            if ore:IsA("Part") then
                local dist = (rootPart.Position - ore.Position).Magnitude
                if dist < shortestDist then
                    nearestOre, shortestDist = ore, dist
                end
            end
        end

        if nearestOre then
            rootPart.CFrame = nearestOre.CFrame * CFrame.new(0, 5, 0)
        end
    end
end)
