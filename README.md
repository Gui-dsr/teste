do
    local ReplicatedStorage = game:GetService("ReplicatedStorage")
    local Players = game:GetService("Players")
    local UserInputService = game:GetService("UserInputService")
    local ContextActionService = game:GetService("ContextActionService")
    local TweenService = game:GetService("TweenService")
    local RunService = game:GetService("RunService")
    local VirtualInputManager = game:GetService("VirtualInputManager")
    local Player = Players.LocalPlayer

    task.wait(1)

    local collectionStatus = {
        IsCollecting = false,
        LootBagsCollected = 0,
        GoldenStatuesCollected = 0,
        TotalLootBagsFound = 0,
        TotalGoldenStatuesFound = 0,
        RoomsSearched = 0,
        FlyingEnabled = false,
        StartPosition = nil,
        OriginalWalkSpeed = 16,
        OriginalJumpPower = 50,
        MinHealthPercent = 30,
        XRayEnabled = false
    }

    local oreHighlighterConfig = {
        FillColor = Color3.fromRGB(255, 235, 0),
        OutlineColor = Color3.fromRGB(255, 190, 60),
        FillTransparency = 0.4,
        OutlineTransparency = 0,
        DepthMode = Enum.HighlightDepthMode.AlwaysOnTop,
        ShowLabels = true,
        LabelSize = UDim2.fromOffset(150, 40),
        LabelFont = Enum.Font.GothamBold,
        NameColor = Color3.fromRGB(255, 255, 255),
        DistanceColor = Color3.fromRGB(200, 200, 200),
        NameTextSize = 14,
        DistanceTextSize = 12,
        LabelBackgroundTransparency = 0,
        LabelBackgroundColor = Color3.fromRGB(40, 40, 40),
        LabelYOffset = 2,
        DistanceUpdateRate = 0.1,
        HighlightNewModels = true,
        MaxRenderDistance = 10000,
        MaxLabelDistance = 225,
        FadeStartDistance = 200
    }

    local xrayHighlights = {
        createdHighlights = {},
        createdLabels = {},
        connections = {},
        highlightedCount = 0
    }

    -- Main GUI
    local gui = Instance.new("ScreenGui")
    gui.Name = "Collection Hub Menu"
    gui.ResetOnSpawn = false
    gui.Parent = Player:WaitForChild("PlayerGui")

    local mainFrame = Instance.new("Frame")
    mainFrame.Name = "MainFrame"
    mainFrame.Size = UDim2.new(0, 300, 0, 40)
    mainFrame.Position = UDim2.new(0.5, -125, 0.5, -75)
    mainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
    mainFrame.BorderSizePixel = 0
    mainFrame.ClipsDescendants = true
    mainFrame.Parent = gui

    local topBar = Instance.new("Frame")
    topBar.Name = "TopBar"
    topBar.Size = UDim2.new(1, 0, 0, 40)
    topBar.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    topBar.BorderSizePixel = 0
    topBar.Parent = mainFrame

    local titleLabel = Instance.new("TextLabel")
    titleLabel.Name = "TitleLabel"
    titleLabel.Size = UDim2.new(1, -10, 1, 0)
    titleLabel.Position = UDim2.new(0, 10, 0, 0)
    titleLabel.BackgroundTransparency = 1
    titleLabel.Text = "Doggo's Hub 🐕"
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.TextColor3 = Color3.fromRGB(230, 230, 230)
    titleLabel.TextSize = 18
    titleLabel.TextXAlignment = Enum.TextXAlignment.Left
    titleLabel.Parent = topBar

    local keyHint = Instance.new("TextLabel")
    keyHint.Name = "KeyHint"
    keyHint.Size = UDim2.new(0, 80, 1, 0)
    keyHint.Position = UDim2.new(1, -90, 0, 0)
    keyHint.BackgroundTransparency = 1
    keyHint.Text = "(K)"
    keyHint.Font = Enum.Font.GothamBold
    keyHint.TextColor3 = Color3.fromRGB(120, 200, 255)
    keyHint.TextSize = 16
    keyHint.Parent = topBar

    local contentContainer = Instance.new("Frame")
    contentContainer.Name = "ContentContainer"
    contentContainer.Size = UDim2.new(1, 0, 1, -40)
    contentContainer.Position = UDim2.new(0, 0, 0, 40)
    contentContainer.BackgroundTransparency = 1
    contentContainer.Visible = false
    contentContainer.Parent = mainFrame

    -- Auto Collect Toggle
    local toggleFrame = Instance.new("Frame")
    toggleFrame.Name = "ToggleFrame"
    toggleFrame.Size = UDim2.new(0.9, 0, 0, 40)
    toggleFrame.Position = UDim2.new(0.05, 0, 0.05, 0)
    toggleFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    toggleFrame.BorderSizePixel = 0
    toggleFrame.Parent = contentContainer

    local toggleLabel = Instance.new("TextLabel")
    toggleLabel.Name = "ToggleLabel"
    toggleLabel.Size = UDim2.new(0.7, 0, 1, 0)
    toggleLabel.Position = UDim2.new(0, 10, 0, 0)
    toggleLabel.BackgroundTransparency = 1
    toggleLabel.Text = "Auto Collect Items"
    toggleLabel.Font = Enum.Font.GothamBold
    toggleLabel.TextColor3 = Color3.fromRGB(230, 230, 230)
    toggleLabel.TextSize = 16
    toggleLabel.TextXAlignment = Enum.TextXAlignment.Left
    toggleLabel.Parent = toggleFrame

    local toggleButton = Instance.new("Frame")
    toggleButton.Name = "ToggleButton"
    toggleButton.Size = UDim2.new(0, 50, 0, 24)
    toggleButton.Position = UDim2.new(1, -60, 0.5, -12)
    toggleButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
    toggleButton.BorderSizePixel = 0
    toggleButton.Parent = toggleFrame

    local toggleCircle = Instance.new("Frame")
    toggleCircle.Name = "ToggleCircle"
    toggleCircle.Size = UDim2.new(0, 20, 0, 20)
    toggleCircle.Position = UDim2.new(0, 2, 0.5, -10)
    toggleCircle.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    toggleCircle.BorderSizePixel = 0
    toggleCircle.Parent = toggleButton

    -- XRay Toggle
    local xrayToggleFrame = Instance.new("Frame")
    xrayToggleFrame.Name = "XRayToggleFrame"
    xrayToggleFrame.Size = UDim2.new(0.9, 0, 0, 40)
    xrayToggleFrame.Position = UDim2.new(0.05, 0, 0.25, 0)
    xrayToggleFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    xrayToggleFrame.BorderSizePixel = 0
    xrayToggleFrame.Parent = contentContainer

    local xrayToggleLabel = Instance.new("TextLabel")
    xrayToggleLabel.Name = "XRayToggleLabel"
    xrayToggleLabel.Size = UDim2.new(0.7, 0, 1, 0)
    xrayToggleLabel.Position = UDim2.new(0, 10, 0, 0)
    xrayToggleLabel.BackgroundTransparency = 1
    xrayToggleLabel.Text = "XRay"
    xrayToggleLabel.Font = Enum.Font.GothamBold
    xrayToggleLabel.TextColor3 = Color3.fromRGB(230, 230, 230)
    xrayToggleLabel.TextSize = 16
    xrayToggleLabel.TextXAlignment = Enum.TextXAlignment.Left
    xrayToggleLabel.Parent = xrayToggleFrame

    local xrayToggleButton = Instance.new("Frame")
    xrayToggleButton.Name = "XRayToggleButton"
    xrayToggleButton.Size = UDim2.new(0, 50, 0, 24)
    xrayToggleButton.Position = UDim2.new(1, -60, 0.5, -12)
    xrayToggleButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
    xrayToggleButton.BorderSizePixel = 0
    xrayToggleButton.Parent = xrayToggleFrame

    local xrayToggleCircle = Instance.new("Frame")
    xrayToggleCircle.Name = "XRayToggleCircle"
    xrayToggleCircle.Size = UDim2.new(0, 20, 0, 20)
    xrayToggleCircle.Position = UDim2.new(0, 2, 0.5, -10)
    xrayToggleCircle.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    xrayToggleCircle.BorderSizePixel = 0
    xrayToggleCircle.Parent = xrayToggleButton

    -- Stats Panel
    local statsPanel = Instance.new("Frame")
    statsPanel.Name = "StatsPanel"
    statsPanel.Size = UDim2.new(0.9, 0, 0, 100)
    statsPanel.Position = UDim2.new(0.05, 0, 0.45, 0)
    statsPanel.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    statsPanel.BorderSizePixel = 0
    statsPanel.Parent = contentContainer

    local statsTitle = Instance.new("TextLabel")
    statsTitle.Name = "StatsTitle"
    statsTitle.Size = UDim2.new(1, 0, 0, 30)
    statsTitle.Position = UDim2.new(0, 0, 0, 0)
    statsTitle.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    statsTitle.BackgroundTransparency = 0
    statsTitle.BorderSizePixel = 0
    statsTitle.Text = "Collection Stats"
    statsTitle.Font = Enum.Font.GothamBold
    statsTitle.TextColor3 = Color3.fromRGB(230, 230, 230)
    statsTitle.TextSize = 14
    statsTitle.Parent = statsPanel

    local lootBagLabel = Instance.new("TextLabel")
    lootBagLabel.Name = "LootBagLabel"
    lootBagLabel.Size = UDim2.new(1, -20, 0, 20)
    lootBagLabel.Position = UDim2.new(0, 10, 0, 35)
    lootBagLabel.BackgroundTransparency = 1
    lootBagLabel.Text = "LootBags: 0"
    lootBagLabel.Font = Enum.Font.Gotham
    lootBagLabel.TextColor3 = Color3.fromRGB(220, 220, 220)
    lootBagLabel.TextSize = 14
    lootBagLabel.TextXAlignment = Enum.TextXAlignment.Left
    lootBagLabel.Parent = statsPanel

    local statueLabel = Instance.new("TextLabel")
    statueLabel.Name = "StatueLabel"
    statueLabel.Size = UDim2.new(1, -20, 0, 20)
    statueLabel.Position = UDim2.new(0, 10, 0, 55)
    statueLabel.BackgroundTransparency = 1
    statueLabel.Text = "Statues: 0"
    statueLabel.Font = Enum.Font.Gotham
    statueLabel.TextColor3 = Color3.fromRGB(220, 220, 220)
    statueLabel.TextSize = 14
    statueLabel.TextXAlignment = Enum.TextXAlignment.Left
    statueLabel.Parent = statsPanel

    local statusLabel = Instance.new("TextLabel")
    statusLabel.Name = "StatusLabel"
    statusLabel.Size = UDim2.new(1, -20, 0, 20)
    statusLabel.Position = UDim2.new(0, 10, 0, 75)
    statusLabel.BackgroundTransparency = 1
    statusLabel.Text = "Ready"
    statusLabel.Font = Enum.Font.Gotham
    statusLabel.TextColor3 = Color3.fromRGB(180, 180, 180)
    statusLabel.TextSize = 14
    statusLabel.TextXAlignment = Enum.TextXAlignment.Left
    statusLabel.Parent = statsPanel

    -------------------------------------------------------------------
    --  APPLY CORNERS / ROUNDED GUI COMPONENTS
    -------------------------------------------------------------------
    local function applyCorners()
        local mainCorner = Instance.new("UICorner")
        mainCorner.CornerRadius = UDim.new(0, 8)
        mainCorner.Parent = mainFrame

        local topCorner = Instance.new("UICorner")
        topCorner.CornerRadius = UDim.new(0, 8)
        topCorner.Parent = topBar

        local toggleFrameCorner = Instance.new("UICorner")
        toggleFrameCorner.CornerRadius = UDim.new(0, 6)
        toggleFrameCorner.Parent = toggleFrame

        local toggleButtonCorner = Instance.new("UICorner")
        toggleButtonCorner.CornerRadius = UDim.new(0, 12)
        toggleButtonCorner.Parent = toggleButton

        local toggleCircleCorner = Instance.new("UICorner")
        toggleCircleCorner.CornerRadius = UDim.new(1, 0)
        toggleCircleCorner.Parent = toggleCircle

        local xrayToggleFrameCorner = Instance.new("UICorner")
        xrayToggleFrameCorner.CornerRadius = UDim.new(0, 6)
        xrayToggleFrameCorner.Parent = xrayToggleFrame

        local xrayToggleButtonCorner = Instance.new("UICorner")
        xrayToggleButtonCorner.CornerRadius = UDim.new(0, 12)
        xrayToggleButtonCorner.Parent = xrayToggleButton

        local xrayToggleCircleCorner = Instance.new("UICorner")
        xrayToggleCircleCorner.CornerRadius = UDim.new(1, 0)
        xrayToggleCircleCorner.Parent = xrayToggleCircle

        local statsPanelCorner = Instance.new("UICorner")
        statsPanelCorner.CornerRadius = UDim.new(0, 6)
        statsPanelCorner.Parent = statsPanel

        local statsTitleCorner = Instance.new("UICorner")
        statsTitleCorner.CornerRadius = UDim.new(0, 6)
        statsTitleCorner.Parent = statsTitle

        local shadow = Instance.new("ImageLabel")
        shadow.Name = "Shadow"
        shadow.AnchorPoint = Vector2.new(0.5, 0.5)
        shadow.BackgroundTransparency = 1
        shadow.Position = UDim2.new(0.5, 0, 0.5, 0)
        shadow.Size = UDim2.new(1, 30, 1, 30)
        shadow.ZIndex = -1
        shadow.Image = "rbxassetid://6015897843"
        shadow.ImageColor3 = Color3.new(0, 0, 0)
        shadow.ImageTransparency = 0.6
        shadow.SliceCenter = Rect.new(49, 49, 450, 450)
        shadow.SliceScale = 0.1
        shadow.Parent = mainFrame
    end

    -------------------------------------------------------------------
    --  HELPER FUNCTIONS
    -------------------------------------------------------------------
    local function getPlayerCharacter()
        return Player and Player.Character
    end

    local function formatDistance(distance)
        if (distance < 10) then
            return string.format("%.1f studs", distance)
        else
            return string.format("%d studs", math.floor(distance))
        end
    end

    local function createInfoLabel(model)
        if (not model or not model:IsA("Model")) then
            return nil
        end

        local billboardGui = Instance.new("BillboardGui")
        billboardGui.Name = "OreInfoLabel"
        billboardGui.Size = oreHighlighterConfig.LabelSize

        local modelSize = model:GetExtentsSize()
        billboardGui.StudsOffset = Vector3.new(0, (modelSize.Y / 2) + oreHighlighterConfig.LabelYOffset, 0)

        local adornPart = model:FindFirstChild("HumanoidRootPart") or model.PrimaryPart
        if not adornPart then
            for _, part in pairs(model:GetDescendants()) do
                if part:IsA("BasePart") then
                    adornPart = part
                    break
                end
            end
            if (not adornPart and (#model:GetChildren() > 0)) then
                adornPart = model:GetChildren()[1]
            end
        end

        if not adornPart then
            return nil
        end

        billboardGui.Adornee = adornPart
        billboardGui.AlwaysOnTop = true

        local frame = Instance.new("Frame")
        frame.Size = UDim2.fromScale(1, 1)
        frame.BackgroundColor3 = oreHighlighterConfig.LabelBackgroundColor
        frame.BackgroundTransparency = oreHighlighterConfig.LabelBackgroundTransparency
        frame.BorderSizePixel = 0
        frame.Parent = billboardGui

        local uiCorner = Instance.new("UICorner")
        uiCorner.CornerRadius = UDim.new(0, 6)
        uiCorner.Parent = frame

        local nameLabel = Instance.new("TextLabel")
        nameLabel.Name = "NameLabel"
        nameLabel.Size = UDim2.fromScale(1, 0.5)
        nameLabel.Position = UDim2.fromScale(0, 0)
        nameLabel.BackgroundTransparency = 1
        nameLabel.Font = oreHighlighterConfig.LabelFont
        nameLabel.TextSize = oreHighlighterConfig.NameTextSize
        nameLabel.TextColor3 = oreHighlighterConfig.NameColor
        nameLabel.Text = model.Name
        nameLabel.Parent = frame

        local distanceLabel = Instance.new("TextLabel")
        distanceLabel.Name = "DistanceLabel"
        distanceLabel.Size = UDim2.fromScale(1, 0.5)
        distanceLabel.Position = UDim2.fromScale(0, 0.5)
        distanceLabel.BackgroundTransparency = 1
        distanceLabel.Font = oreHighlighterConfig.LabelFont
        distanceLabel.TextSize = oreHighlighterConfig.DistanceTextSize
        distanceLabel.TextColor3 = oreHighlighterConfig.DistanceColor
        distanceLabel.Text = "Calculating..."
        distanceLabel.Parent = frame

        billboardGui.Enabled = false
        billboardGui.Parent = model
        return billboardGui
    end

    local function applyHighlight(model)
        if not model:IsA("Model") then
            return
        end

        local highlight = Instance.new("Highlight")
        highlight.FillColor = oreHighlighterConfig.FillColor
        highlight.OutlineColor = oreHighlighterConfig.OutlineColor
        highlight.FillTransparency = oreHighlighterConfig.FillTransparency
        highlight.OutlineTransparency = oreHighlighterConfig.OutlineTransparency
        highlight.DepthMode = oreHighlighterConfig.DepthMode
        highlight.Parent = model

        table.insert(xrayHighlights.createdHighlights, highlight)

        if oreHighlighterConfig.ShowLabels then
            local label = createInfoLabel(model)
            if label then
                table.insert(xrayHighlights.createdLabels, label)
            end
        end

        xrayHighlights.highlightedCount = xrayHighlights.highlightedCount + 1
        return highlight
    end

    local function updateDistanceLabels()
        local character = getPlayerCharacter()
        if (not character or not character:FindFirstChild("HumanoidRootPart")) then
            return
        end

        local characterPosition = character.HumanoidRootPart.Position
        local oresFolder = workspace:FindFirstChild("SpawnedOres")

        if not oresFolder then
            return
        end

        for _, model in pairs(oresFolder:GetChildren()) do
            if not model:IsA("Model") then
                continue
            end

            local modelCenter = model:GetPivot().Position
            local distance = (characterPosition - modelCenter).Magnitude
            local billboardGui = model:FindFirstChild("OreInfoLabel")

            if billboardGui then
                local distanceLabel = billboardGui.Frame:FindFirstChild("DistanceLabel")
                if distanceLabel then
                    distanceLabel.Text = formatDistance(distance)
                end

                billboardGui.Enabled = (distance <= oreHighlighterConfig.MaxLabelDistance)

                if (distance <= oreHighlighterConfig.MaxLabelDistance) then
                    local frame = billboardGui.Frame
                    if frame then
                        local transparencyValue = 0
                        if (distance > oreHighlighterConfig.FadeStartDistance) then
                            local fadeProgress = (distance - oreHighlighterConfig.FadeStartDistance)
                                / (oreHighlighterConfig.MaxLabelDistance - oreHighlighterConfig.FadeStartDistance)
                            transparencyValue = fadeProgress * fadeProgress * 0.8
                        end
                        frame.BackgroundTransparency = transparencyValue
                        for _, child in pairs(frame:GetChildren()) do
                            if child:IsA("TextLabel") then
                                child.TextTransparency = transparencyValue
                            end
                        end
                    end
                end
            end

            local highlight = model:FindFirstChildOfClass("Highlight")
            if highlight then
                highlight.Enabled = true
            end
        end
    end

    local function clearAllHighlights()
        for _, connection in ipairs(xrayHighlights.connections) do
            if connection then
                connection:Disconnect()
            end
        end
        xrayHighlights.connections = {}

        for _, highlight in ipairs(xrayHighlights.createdHighlights) do
            if (highlight and highlight.Parent) then
                highlight:Destroy()
            end
        end
        xrayHighlights.createdHighlights = {}

        for _, label in ipairs(xrayHighlights.createdLabels) do
            if (label and label.Parent) then
                label:Destroy()
            end
        end
        xrayHighlights.createdLabels = {}
        xrayHighlights.highlightedCount = 0
    end

    -------------------------------------------------------------------
    --  XRAY
    -------------------------------------------------------------------
    local function enableXRay()
        if collectionStatus.XRayEnabled then
            return
        end

        collectionStatus.XRayEnabled = true
        local oresFolder = workspace:FindFirstChild("SpawnedOres")

        if not oresFolder then
            showNotification("SpawnedOres folder not found!", 3)
            return
        end

        for _, model in pairs(oresFolder:GetChildren()) do
            applyHighlight(model)
        end

        local lastUpdate = 0
        local updateConnection = RunService.Heartbeat:Connect(function(dt)
            lastUpdate = lastUpdate + dt
            if (lastUpdate >= oreHighlighterConfig.DistanceUpdateRate) then
                updateDistanceLabels()
                lastUpdate = 0
            end
        end)
        table.insert(xrayHighlights.connections, updateConnection)

        local newModelConnection = oresFolder.ChildAdded:Connect(function(child)
            task.wait(0.05)
            applyHighlight(child)
        end)
        table.insert(xrayHighlights.connections, newModelConnection)

        TweenService:Create(xrayToggleButton, TweenInfo.new(0.3), {
            BackgroundColor3 = Color3.fromRGB(50, 200, 50)
        }):Play()
        TweenService:Create(xrayToggleCircle, TweenInfo.new(0.3), {
            Position = UDim2.new(0, 28, 0.5, -10)
        }):Play()

        showNotification("XRay enabled", 2)
        log("XRay enabled")
    end

    local function disableXRay()
        if not collectionStatus.XRayEnabled then
            return
        end

        collectionStatus.XRayEnabled = false
        clearAllHighlights()

        TweenService:Create(xrayToggleButton, TweenInfo.new(0.3), {
            BackgroundColor3 = Color3.fromRGB(200, 50, 50)
        }):Play()
        TweenService:Create(xrayToggleCircle, TweenInfo.new(0.3), {
            Position = UDim2.new(0, 2, 0.5, -10)
        }):Play()

        showNotification("XRay disabled", 2)
        log("XRay disabled")
    end

    local function toggleXRay()
        if collectionStatus.XRayEnabled then
            disableXRay()
        else
            enableXRay()
        end
    end

    -------------------------------------------------------------------
    --  NOTIFICATION
    -------------------------------------------------------------------
    local notificationFrame = Instance.new("Frame")
    notificationFrame.Name = "NotificationFrame"
    notificationFrame.Size = UDim2.new(0, 300, 0, 40)
    notificationFrame.Position = UDim2.new(0.5, -150, 0.2, 0)
    notificationFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    notificationFrame.BorderSizePixel = 0
    notificationFrame.Visible = false
    notificationFrame.Parent = gui

    local notificationText = Instance.new("TextLabel")
    notificationText.Name = "NotificationText"
    notificationText.Size = UDim2.new(1, -20, 1, 0)
    notificationText.Position = UDim2.new(0, 10, 0, 0)
    notificationText.BackgroundTransparency = 1
    notificationText.Text = ""
    notificationText.Font = Enum.Font.GothamBold
    notificationText.TextColor3 = Color3.fromRGB(255, 255, 255)
    notificationText.TextSize = 16
    notificationText.Parent = notificationFrame

    local notificationCorner = Instance.new("UICorner")
    notificationCorner.CornerRadius = UDim.new(0, 8)
    notificationCorner.Parent = notificationFrame

    local notificationStroke = Instance.new("UIStroke")
    notificationStroke.Color = Color3.fromRGB(255, 100, 100)
    notificationStroke.Thickness = 2
    notificationStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    notificationStroke.Parent = notificationFrame

    local notificationIcon = Instance.new("ImageLabel")
    notificationIcon.Name = "NotificationIcon"
    notificationIcon.Size = UDim2.new(0, 24, 0, 24)
    notificationIcon.Position = UDim2.new(0, 10, 0.5, -12)
    notificationIcon.BackgroundTransparency = 1
    notificationIcon.Image = "rbxassetid://7734053426"
    notificationIcon.ImageColor3 = Color3.fromRGB(255, 100, 100)
    notificationIcon.Parent = notificationFrame

    notificationText.Position = UDim2.new(0, 44, 0, 0)

    local function showNotification(message, duration)
        duration = duration or 3
        if (not gui or not gui.Parent) then
            return
        end
        if (not notificationFrame or not notificationText
            or not notificationIcon or not notificationStroke) then
            return
        end
        if not TweenService then
            notificationText.Text = message
            notificationFrame.Visible = true
            task.delay(duration, function()
                notificationFrame.Visible = false
            end)
            return
        end

        notificationText.Text = message
        notificationFrame.Visible = true
        notificationFrame.Position = UDim2.new(0.5, -150, 0.15, 0)
        notificationFrame.BackgroundTransparency = 1
        notificationText.TextTransparency = 1
        notificationIcon.ImageTransparency = 1
        notificationStroke.Transparency = 1

        local function safeTween(instance, info, properties)
            if (instance and info and properties) then
                local success, tween = pcall(function()
                    return TweenService:Create(instance, info, properties)
                end)
                if (success and tween) then
                    tween:Play()
                    return tween
                else
                    return nil
                end
            end
        end

        safeTween(notificationFrame, TweenInfo.new(0.3, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {
            Position = UDim2.new(0.5, -150, 0.2, 0),
            BackgroundTransparency = 0
        })
        safeTween(notificationText, TweenInfo.new(0.3), {
            TextTransparency = 0
        })
        safeTween(notificationIcon, TweenInfo.new(0.3), {
            ImageTransparency = 0
        })
        safeTween(notificationStroke, TweenInfo.new(0.3), {
            Transparency = 0
        })

        task.delay(duration, function()
            if (not notificationFrame or not notificationFrame.Parent) then
                return
            end
            safeTween(notificationFrame, TweenInfo.new(0.3, Enum.EasingStyle.Quart, Enum.EasingDirection.In), {
                Position = UDim2.new(0.5, -150, 0.15, 0),
                BackgroundTransparency = 1
            })
            safeTween(notificationText, TweenInfo.new(0.3), {
                TextTransparency = 1
            })
            safeTween(notificationIcon, TweenInfo.new(0.3), {
                ImageTransparency = 1
            })
            safeTween(notificationStroke, TweenInfo.new(0.3), {
                Transparency = 1
            })

            task.delay(0.3, function()
                if notificationFrame then
                    notificationFrame.Visible = false
                end
            end)
        end)
    end

    -------------------------------------------------------------------
    --  MOUSE CONTROL
    -------------------------------------------------------------------
    local isMouseEnabled = false
    local oldMouseBehavior, oldCameraType

    local function checkPlayerHealth()
        local character = Player.Character
        if not character then
            return true
        end

        local humanoid = character:FindFirstChild("Humanoid")
        if not humanoid then
            return true
        end

        local healthPercent = (humanoid.Health / humanoid.MaxHealth) * 100
        if (healthPercent <= collectionStatus.MinHealthPercent) then
            pcall(function()
                showNotification("❗ Health low (" .. math.floor(healthPercent) .. "%), returning to safety!", 3)
            end)
            if (collectionStatus.IsCollecting and toggleButton and toggleCircle) then
                pcall(function()
                    toggleButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
                    toggleCircle.Position = UDim2.new(0, 2, 0.5, -10)
                end)
            end
            return false
        end
        return true
    end

    local function toggleMouseControl(forceClose)
        if forceClose then
            isMouseEnabled = false
        else
            isMouseEnabled = not isMouseEnabled
        end

        local openSize = UDim2.new(0, 300, 0, 300)
        local closedSize = UDim2.new(0, 300, 0, 40)

        if isMouseEnabled then
            oldMouseBehavior = UserInputService.MouseBehavior
            if workspace.CurrentCamera then
                oldCameraType = workspace.CurrentCamera.CameraType
            end
            UserInputService.MouseIconEnabled = true
            UserInputService.MouseBehavior = Enum.MouseBehavior.Default
            if workspace.CurrentCamera then
                workspace.CurrentCamera.CameraType = Enum.CameraType.Scriptable
            end

            ContextActionService:BindAction(
                "DisableMovement",
                function()
                    return Enum.ContextActionResult.Sink
                end,
                false,
                Enum.KeyCode.W,
                Enum.KeyCode.A,
                Enum.KeyCode.S,
                Enum.KeyCode.D,
                Enum.KeyCode.Space
            )

            ContextActionService:BindAction(
                "DisableCameraRotation",
                function()
                    return Enum.ContextActionResult.Sink
                end,
                false,
                Enum.UserInputType.MouseMovement,
                Enum.UserInputType.Touch
            )

            if (Player.Character and Player.Character:FindFirstChild("Humanoid")) then
                local humanoid = Player.Character.Humanoid
                if (collectionStatus.OriginalWalkSpeed == 16) then
                    collectionStatus.OriginalWalkSpeed = humanoid.WalkSpeed
                end
                if (collectionStatus.OriginalJumpPower == 50) then
                    collectionStatus.OriginalJumpPower = humanoid.JumpPower
                end
                humanoid.WalkSpeed = 0
                humanoid.JumpPower = 0
            end

            contentContainer.Visible = true

            if TweenService then
                pcall(function()
                    TweenService:Create(mainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {
                        Size = openSize
                    }):Play()
                    TweenService:Create(keyHint, TweenInfo.new(0.3), {
                        TextColor3 = Color3.fromRGB(255, 130, 130)
                    }):Play()
                end)
            else
                mainFrame.Size = openSize
                keyHint.TextColor3 = Color3.fromRGB(255, 130, 130)
            end
        else
            UserInputService.MouseIconEnabled = false
            UserInputService.MouseBehavior = oldMouseBehavior or Enum.MouseBehavior.LockCenter
            if (workspace.CurrentCamera and oldCameraType) then
                workspace.CurrentCamera.CameraType = oldCameraType
            end

            ContextActionService:UnbindAction("DisableMovement")
            ContextActionService:UnbindAction("DisableCameraRotation")

            if (Player.Character and Player.Character:FindFirstChild("Humanoid")) then
                local humanoid = Player.Character.Humanoid
                if not collectionStatus.IsCollecting then
                    humanoid.WalkSpeed = 65
                    humanoid.JumpPower = collectionStatus.OriginalJumpPower
                end
                humanoid.AutoRotate = true
                pcall(function()
                    humanoid:ChangeState(Enum.HumanoidStateType.Landed)
                    task.wait(0.05)
                    humanoid:ChangeState(Enum.HumanoidStateType.Running)
                end)
            end

            if (collectionStatus.FlyingEnabled and not collectionStatus.IsCollecting) then
                disableFlying()
            end

            if TweenService then
                pcall(function()
                    TweenService:Create(
                        mainFrame,
                        TweenInfo.new(0.3, Enum.EasingStyle.Quart, Enum.EasingDirection.Out),
                        { Size = closedSize }
                    ):Play()
                    TweenService:Create(keyHint, TweenInfo.new(0.3), {
                        TextColor3 = Color3.fromRGB(120, 200, 255)
                    }):Play()
                end)
            else
                mainFrame.Size = closedSize
                keyHint.TextColor3 = Color3.fromRGB(120, 200, 255)
            end

            task.delay(0.3, function()
                if not isMouseEnabled then
                    contentContainer.Visible = false
                end
            end)
        end

        if not isMouseEnabled then
            task.delay(0.5, function()
                ContextActionService:UnbindAction("DisableMovement")
                ContextActionService:UnbindAction("DisableCameraRotation")
                UserInputService.MouseIconEnabled = false
                UserInputService.MouseBehavior = Enum.MouseBehavior.LockCenter
            end)
        end
    end

    local function log(message)
        statusLabel.Text = message
    end

    local function updateStats()
        lootBagLabel.Text = "LootBags: " .. collectionStatus.LootBagsCollected
        statueLabel.Text = "Statues: " .. collectionStatus.GoldenStatuesCollected
    end

    -------------------------------------------------------------------
    --  FLYING
    -------------------------------------------------------------------
    local function enableFlying()
        local character = Player.Character
        if not character then
            return
        end

        local humanoid = character:FindFirstChild("Humanoid")
        if not humanoid then
            return
        end

        collectionStatus.OriginalWalkSpeed = humanoid.WalkSpeed
        collectionStatus.OriginalJumpPower = humanoid.JumpPower

        humanoid:ChangeState(Enum.HumanoidStateType.Physics)
        humanoid.PlatformStand = true
        collectionStatus.FlyingEnabled = true
        log("Flying mode enabled")
    end

    local function disableFlying()
        local character = Player.Character
        if not character then
            return
        end

        local humanoid = character:FindFirstChild("Humanoid")
        if not humanoid then
            return
        end

        pcall(function()
            humanoid:ChangeState(Enum.HumanoidStateType.Landed)
            task.wait(0.1)
            humanoid:ChangeState(Enum.HumanoidStateType.GettingUp)
            task.wait(0.1)
            humanoid:ChangeState(Enum.HumanoidStateType.Running)
        end)

        humanoid.PlatformStand = false
        humanoid.AutoRotate = true
        humanoid.WalkSpeed = 65
        humanoid.JumpPower = collectionStatus.OriginalJumpPower

        for _, part in pairs(character:GetDescendants()) do
            if part:IsA("BasePart") then
                pcall(function()
                    part.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
                    part.AssemblyAngularVelocity = Vector3.new(0, 0, 0)
                end)
            end
        end

        pcall(function()
            ContextActionService:UnbindAction("DisableMovement")
            ContextActionService:UnbindAction("DisableCameraRotation")
        end)
        collectionStatus.FlyingEnabled = false
        log("Flying mode disabled")
    end

    -------------------------------------------------------------------
    --  COLLECT ITEMS
    -------------------------------------------------------------------
    local function findCollectiblesInRoom(room)
        local collectibles = {}
        if (not room or not room.Parent) then
            return collectibles
        end

        for _, descendant in pairs(room:GetDescendants()) do
            if descendant:IsA("Model") then
                if (descendant.Name == "LootBag") then
                    table.insert(collectibles, descendant)
                    collectionStatus.TotalLootBagsFound = collectionStatus.TotalLootBagsFound + 1
                elseif (descendant.Name == "GoldenStatue") then
                    table.insert(collectibles, descendant)
                    collectionStatus.TotalGoldenStatuesFound = collectionStatus.TotalGoldenStatuesFound + 1
                end
            end
        end
        return collectibles
    end

    local function getRooms()
        local roomsFolder = workspace:FindFirstChild("SpawnedRooms")

        if not roomsFolder then
            local possibleContainers = { "Rooms", "Level", "Map", "World" }
            for _, name in ipairs(possibleContainers) do
                local container = workspace:FindFirstChild(name)
                if container then
                    roomsFolder = container
                    break
                end
            end

            if not roomsFolder then
                local potentialRooms = {}
                for _, child in pairs(workspace:GetChildren()) do
                    if (child:IsA("Model") and not child:FindFirstChild("Humanoid")) then
                        table.insert(potentialRooms, child)
                    end
                end
                if (#potentialRooms > 0) then
                    log("Found " .. #potentialRooms .. " potential rooms directly in workspace")
                    return potentialRooms
                end

                log("No rooms found anywhere!")
                return {}
            end
        end

        local rooms = {}
        for _, room in pairs(roomsFolder:GetChildren()) do
            if (room:IsA("Model") or room:IsA("BasePart")) then
                table.insert(rooms, room)
            end
        end

        collectionStatus.RoomsSearched = collectionStatus.RoomsSearched + 1
        log("Found " .. #rooms .. " rooms")

        if ((#rooms == 0) and roomsFolder) then
            for _, child in pairs(roomsFolder:GetDescendants()) do
                if (child:IsA("Model") and not rooms[child]) then
                    if (child.PrimaryPart or (#child:GetChildren() > 5)) then
                        table.insert(rooms, child)
                    end
                end
            end
            log("Found " .. #rooms .. " rooms after deeper search")
        end

        if (#rooms == 0) then
            for _, child in pairs(workspace:GetChildren()) do
                if (child:IsA("Model") and not child:FindFirstChild("Humanoid")) then
                    if (child.Name:find("Room") or child.Name:find("Area")
                        or child.Name:find("Zone") or child.Name:find("Section")) then
                        table.insert(rooms, child)
                    end
                end
            end
            log("Found " .. #rooms .. " rooms through fallback method")
        end

        if (#rooms == 0) then
            table.insert(rooms, workspace)
        end
        return rooms
    end

    local function pressKey()
        if not VirtualInputManager then
            VirtualInputManager = game:GetService("VirtualInputManager")
            if not VirtualInputManager then
                return
            end
        end

        local success, err = pcall(function()
            VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.E, false, game)
            task.wait(0.1)
            VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.E, false, game)
        end)
    end

    local function teleportToItem(item)
        if (not item or not item.Parent) then
            return false
        end

        if not collectionStatus.FlyingEnabled then
            enableFlying()
        end

        local targetPart = item.PrimaryPart
        if not targetPart then
            for _, part in pairs(item:GetDescendants()) do
                if part:IsA("BasePart") then
                    targetPart = part
                    break
                end
            end
        end

        if not targetPart then
            return false
        end

        local character = Player.Character
        if not character then
            return false
        end

        local hrp = character:FindFirstChild("HumanoidRootPart")
        if not hrp then
            return false
        end

        local itemPosition = targetPart.Position
        local offsetDirection = Vector3.new(1, 0, 1).Unit
        local teleportPosition = itemPosition - (offsetDirection * 3)

        teleportPosition = Vector3.new(teleportPosition.X, itemPosition.Y + 1, teleportPosition.Z)
        local lookCFrame = CFrame.new(teleportPosition, itemPosition)
        hrp.CFrame = lookCFrame
        task.wait(0.2)

        local humanoid = character:FindFirstChild("Humanoid")
        if humanoid then
            humanoid.AutoRotate = false
            humanoid:Move(Vector3.new(0, 0, 0))
            hrp.CFrame = CFrame.new(hrp.Position, itemPosition)
        end
        task.wait(0.1)
        return true
    end

    local function collectItem(item)
        if (not item or not item.Parent) then
            return false
        end

        if not teleportToItem(item) then
            return false
        end

        local attempts = 0
        local collected = false
        local positions = 0

        while (attempts < 5) and not collected do
            attempts = attempts + 1
            if not collectionStatus.IsCollecting then
                return false
            end

            if (not item or not item.Parent) then
                collected = true
                break
            end

            pressKey()
            task.wait(0.5)

            if (not item or not item.Parent) then
                collected = true
                break
            end

            if (((attempts % 2) == 0) and not collected and item and item.Parent) then
                positions = positions + 1
                local targetPart = item.PrimaryPart
                if not targetPart then
                    for _, part in pairs(item:GetDescendants()) do
                        if part:IsA("BasePart") then
                            targetPart = part
                            break
                        end
                    end
                end

                if targetPart then
                    local angle = (positions * math.pi) / 2
                    local offset = Vector3.new(math.cos(angle), 0, math.sin(angle)) * 3
                    local newPosition = targetPart.Position + offset
                    newPosition = Vector3.new(newPosition.X, targetPart.Position.Y + 1, newPosition.Z)

                    local character = Player.Character
                    if (character and character:FindFirstChild("HumanoidRootPart")) then
                        character.HumanoidRootPart.CFrame = CFrame.new(newPosition, targetPart.Position)
                        task.wait(0.2)
                    end
                end
            end
        end

        if collected then
            if (item.Name == "LootBag") then
                collectionStatus.LootBagsCollected = collectionStatus.LootBagsCollected + 1
            elseif (item.Name == "GoldenStatue") then
                collectionStatus.GoldenStatuesCollected = collectionStatus.GoldenStatuesCollected + 1
            end
            updateStats()
        end

        return collected
    end

    local collectionLoop = nil

    -------------------------------------------------------------------
    --  START & STOP
    -------------------------------------------------------------------
    local function startCollection()
        if collectionStatus.IsCollecting then
            return
        end

        if not checkPlayerHealth() then
            pcall(function()
                showNotification("❗ Health too low to start collection!", 3)
            end)
            return
        end

        local character = Player.Character
        if (character and character:FindFirstChild("HumanoidRootPart")) then
            collectionStatus.StartPosition = character.HumanoidRootPart.Position
        end
        if (character and character:FindFirstChild("Humanoid")) then
            local humanoid = character:FindFirstChild("Humanoid")
            collectionStatus.OriginalWalkSpeed = humanoid.WalkSpeed
            collectionStatus.OriginalJumpPower = humanoid.JumpPower
        end

        collectionStatus.IsCollecting = true
        collectionStatus.LootBagsCollected = 0
        collectionStatus.GoldenStatuesCollected = 0

        if TweenService then
            pcall(function()
                TweenService:Create(toggleButton, TweenInfo.new(0.3), {
                    BackgroundColor3 = Color3.fromRGB(50, 200, 50)
                }):Play()
                TweenService:Create(toggleCircle, TweenInfo.new(0.3), {
                    Position = UDim2.new(0, 28, 0.5, -10)
                }):Play()
            end)
        else
            if toggleButton then
                toggleButton.BackgroundColor3 = Color3.fromRGB(50, 200, 50)
            end
            if toggleCircle then
                toggleCircle.Position = UDim2.new(0, 28, 0.5, -10)
            end
        end

        pcall(function()
            showNotification("Collection starting...", 2)
        end)

        if isMouseEnabled then
            task.wait(0.3)
            toggleMouseControl(true)
        end

        pcall(function()
            enableFlying()
        end)

        if collectionLoop then
            task.cancel(collectionLoop)
            collectionLoop = nil
        end

        collectionLoop = task.spawn(function()
            while true do
                if not checkPlayerHealth() then
                    stopCollection()
                    if collectionStatus.StartPosition then
                        local character = Player.Character
                        if (character and character:FindFirstChild("HumanoidRootPart")) then
                            character.HumanoidRootPart.CFrame = CFrame.new(collectionStatus.StartPosition)
                            if character:FindFirstChild("Humanoid") then
                                local humanoid = character:FindFirstChild("Humanoid")
                                pcall(function()
                                    humanoid:ChangeState(Enum.HumanoidStateType.Landed)
                                    task.wait(0.1)
                                    humanoid:ChangeState(Enum.HumanoidStateType.GettingUp)
                                    task.wait(0.1)
                                    humanoid:ChangeState(Enum.HumanoidStateType.Running)
                                end)
                                humanoid.WalkSpeed = 65
                                humanoid.PlatformStand = false
                                humanoid.AutoRotate = true
                                humanoid.JumpPower = collectionStatus.OriginalJumpPower

                                for _, part in pairs(character:GetDescendants()) do
                                    if part:IsA("BasePart") then
                                        pcall(function()
                                            part.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
                                            part.AssemblyAngularVelocity = Vector3.new(0, 0, 0)
                                        end)
                                    end
                                end

                                pcall(function()
                                    ContextActionService:UnbindAction("DisableMovement")
                                    ContextActionService:UnbindAction("DisableCameraRotation")
                                end)
                            end
                        end
                    end
                    break
                end

                if not collectionStatus.IsCollecting then
                    break
                end

                local rooms = getRooms()
                if (#rooms == 0) then
                    pcall(function()
                        log("No rooms found, waiting...")
                    end)
                    task.wait(2)
                    continue
                end

                for i, room in ipairs(rooms) do
                    if not checkPlayerHealth() then
                        stopCollection()
                        if collectionStatus.StartPosition then
                            local character = Player.Character
                            if (character and character:FindFirstChild("HumanoidRootPart")) then
                                character.HumanoidRootPart.CFrame = CFrame.new(collectionStatus.StartPosition)
                                if character:FindFirstChild("Humanoid") then
                                    local humanoid = character:FindFirstChild("Humanoid")
                                    pcall(function()
                                        humanoid:ChangeState(Enum.HumanoidStateType.Landed)
                                        task.wait(0.1)
                                        humanoid:ChangeState(Enum.HumanoidStateType.GettingUp)
                                        task.wait(0.1)
                                        humanoid:ChangeState(Enum.HumanoidStateType.Running)
                                    end)
                                    humanoid.WalkSpeed = 65
                                    humanoid.PlatformStand = false
                                    humanoid.AutoRotate = true
                                    humanoid.JumpPower = collectionStatus.OriginalJumpPower

                                    for _, part in pairs(character:GetDescendants()) do
                                        if part:IsA("BasePart") then
                                            pcall(function()
                                                part.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
                                                part.AssemblyAngularVelocity = Vector3.new(0, 0, 0)
                                            end)
                                        end
                                    end
                                    pcall(function()
                                        ContextActionService:UnbindAction("DisableMovement")
                                        ContextActionService:UnbindAction("DisableCameraRotation")
                                    end)
                                end
                            end
                        end
                        break
                    end

                    if not collectionStatus.IsCollecting then
                        break
                    end

                    local collectibles = findCollectiblesInRoom(room)
                    if (#collectibles == 0) then
                        continue
                    end

                    pcall(function()
                        showNotification("Found " .. #collectibles .. " items to collect!", 2)
                    end)

                    for j, item in ipairs(collectibles) do
                        if not checkPlayerHealth() then
                            stopCollection()
                            if collectionStatus.StartPosition then
                                local character = Player.Character
                                if (character and character:FindFirstChild("HumanoidRootPart")) then
                                    character.HumanoidRootPart.CFrame = CFrame.new(collectionStatus.StartPosition)
                                    if character:FindFirstChild("Humanoid") then
                                        local humanoid = character:FindFirstChild("Humanoid")
                                        pcall(function()
                                            humanoid:ChangeState(Enum.HumanoidStateType.Landed)
                                            task.wait(0.1)
                                            humanoid:ChangeState(Enum.HumanoidStateType.GettingUp)
                                            task.wait(0.1)
                                            humanoid:ChangeState(Enum.HumanoidStateType.Running)
                                        end)
                                        humanoid.WalkSpeed = 65
                                        humanoid.PlatformStand = false
                                        humanoid.AutoRotate = true
                                        humanoid.JumpPower = collectionStatus.OriginalJumpPower

                                        for _, part in pairs(character:GetDescendants()) do
                                            if part:IsA("BasePart") then
                                                pcall(function()
                                                    part.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
                                                    part.AssemblyAngularVelocity = Vector3.new(0, 0, 0)
                                                end)
                                            end
                                        end
                                        pcall(function()
                                            ContextActionService:UnbindAction("DisableMovement")
                                            ContextActionService:UnbindAction("DisableCameraRotation")
                                        end)
                                    end
                                end
                            end
                            break
                        end

                        if not collectionStatus.IsCollecting then
                            break
                        end

                        pcall(function()
                            log("Collecting " .. item.Name .. "...")
                        end)

                        local success = collectItem(item)
                        if success then
                            pcall(function()
                                showNotification("Collected " .. item.Name .. "!", 1)
                            end)
                        end
                        task.wait(0.3)
                    end
                    task.wait(1)
                end

                pcall(function()
                    log("Scan complete, waiting for next cycle...")
                end)
                task.wait(3)
            end
        end)

        pcall(function()
            log("Collection active - searching for items...")
            showNotification("Collection started! Press K to check progress", 3)
        end)
    end

    local function stopCollection()
        if not collectionStatus.IsCollecting then
            return
        end

        collectionStatus.IsCollecting = false
        if TweenService then
            pcall(function()
                TweenService:Create(toggleButton, TweenInfo.new(0.3), {
                    BackgroundColor3 = Color3.fromRGB(200, 50, 50)
                }):Play()
                TweenService:Create(toggleCircle, TweenInfo.new(0.3), {
                    Position = UDim2.new(0, 2, 0.5, -10)
                }):Play()
            end)
        else
            if toggleButton then
                toggleButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
            end
            if toggleCircle then
                toggleCircle.Position = UDim2.new(0, 2, 0.5, -10)
            end
        end

        pcall(function()
            showNotification("Collection stopped, returning to start position", 3)
        end)

        if collectionStatus.StartPosition then
            local character = Player.Character
            if (character and character:FindFirstChild("HumanoidRootPart")) then
                character.HumanoidRootPart.CFrame = CFrame.new(collectionStatus.StartPosition)
                if character:FindFirstChild("Humanoid") then
                    local humanoid = character:FindFirstChild("Humanoid")
                    pcall(function()
                        humanoid:ChangeState(Enum.HumanoidStateType.Landed)
                        task.wait(0.1)
                        humanoid:ChangeState(Enum.HumanoidStateType.GettingUp)
                        task.wait(0.1)
                        humanoid:ChangeState(Enum.HumanoidStateType.Running)
                    end)
                    humanoid.WalkSpeed = 65
                    humanoid.PlatformStand = false
                    humanoid.AutoRotate = true
                    humanoid.JumpPower = collectionStatus.OriginalJumpPower

                    for _, part in pairs(character:GetDescendants()) do
                        if part:IsA("BasePart") then
                            pcall(function()
                                part.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
                                part.AssemblyAngularVelocity = Vector3.new(0, 0, 0)
                            end)
                        end
                    end
                    pcall(function()
                        ContextActionService:UnbindAction("DisableMovement")
                        ContextActionService:UnbindAction("DisableCameraRotation")
                    end)
                end
            end
        end

        pcall(function()
            disableFlying()
        end)

        if collectionLoop then
            task.cancel(collectionLoop)
            collectionLoop = nil
        end

        pcall(function()
            updateStats()
        end)
        pcall(function()
            log("Ready")
        end)
    end

    local function enhanceStatsPanel()
        if not statsPanel then
            return
        end

        local statsPanelStroke = Instance.new("UIStroke")
        statsPanelStroke.Color = Color3.fromRGB(80, 80, 80)
        statsPanelStroke.Thickness = 1
        statsPanelStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
        statsPanelStroke.Parent = statsPanel

        local lootBagIcon = Instance.new("ImageLabel")
        lootBagIcon.Name = "LootBagIcon"
        lootBagIcon.Size = UDim2.new(0, 18, 0, 18)
        lootBagIcon.Position = UDim2.new(0, 10, 0, 36)
        lootBagIcon.BackgroundTransparency = 1
        lootBagIcon.Image = "rbxassetid://6031280882"
        lootBagIcon.ImageColor3 = Color3.fromRGB(255, 220, 100)
        lootBagIcon.Parent = statsPanel

        if lootBagLabel then
            lootBagLabel.Position = UDim2.new(0, 36, 0, 35)
        end

        local statueIcon = Instance.new("ImageLabel")
        statueIcon.Name = "StatueIcon"
        statueIcon.Size = UDim2.new(0, 18, 0, 18)
        statueIcon.Position = UDim2.new(0, 10, 0, 56)
        statueIcon.BackgroundTransparency = 1
        statueIcon.Image = "rbxassetid://6031763426"
        statueIcon.ImageColor3 = Color3.fromRGB(255, 215, 0)
        statueIcon.Parent = statsPanel

        if statueLabel then
            statueLabel.Position = UDim2.new(0, 36, 0, 55)
        end

        local statusIcon = Instance.new("ImageLabel")
        statusIcon.Name = "StatusIcon"
        statusIcon.Size = UDim2.new(0, 18, 0, 18)
        statusIcon.Position = UDim2.new(0, 10, 0, 76)
        statusIcon.BackgroundTransparency = 1
        statusIcon.Image = "rbxassetid://6034684949"
        statusIcon.ImageColor3 = Color3.fromRGB(120, 200, 255)
        statusIcon.Parent = statsPanel

        if statusLabel then
            statusLabel.Position = UDim2.new(0, 36, 0, 75)
        end
    end

    -------------------------------------------------------------------
    --  BINDINGS
    -------------------------------------------------------------------
    toggleButton.InputBegan:Connect(function(input)
        if (input.UserInputType == Enum.UserInputType.MouseButton1) then
            if collectionStatus.IsCollecting then
                stopCollection()
            else
                startCollection()
            end
        end
    end)

    toggleButton.InputBegan:Connect(function(input)
        if (input.UserInputType == Enum.UserInputType.MouseButton1) then
            TweenService:Create(toggleButton, TweenInfo.new(0.1), {
                BackgroundTransparency = 0.2
            }):Play()
        end
    end)

    toggleButton.InputEnded:Connect(function(input)
        if (input.UserInputType == Enum.UserInputType.MouseButton1) then
            TweenService:Create(toggleButton, TweenInfo.new(0.1), {
                BackgroundTransparency = 0
            }):Play()
        end
    end)

    toggleFrame.MouseEnter:Connect(function()
        if not collectionStatus.IsCollecting then
            TweenService:Create(toggleButton, TweenInfo.new(0.3), {
                BackgroundColor3 = Color3.fromRGB(220, 70, 70)
            }):Play()
        else
            TweenService:Create(toggleButton, TweenInfo.new(0.3), {
                BackgroundColor3 = Color3.fromRGB(70, 220, 70)
            }):Play()
        end
        TweenService:Create(toggleFrame, TweenInfo.new(0.3), {
            BackgroundColor3 = Color3.fromRGB(45, 45, 45)
        }):Play()
    end)

    toggleFrame.MouseLeave:Connect(function()
        if not collectionStatus.IsCollecting then
            TweenService:Create(toggleButton, TweenInfo.new(0.3), {
                BackgroundColor3 = Color3.fromRGB(200, 50, 50)
            }):Play()
        else
            TweenService:Create(toggleButton, TweenInfo.new(0.3), {
                BackgroundColor3 = Color3.fromRGB(50, 200, 50)
            }):Play()
        end
        TweenService:Create(toggleFrame, TweenInfo.new(0.3), {
            BackgroundColor3 = Color3.fromRGB(35, 35, 35)
        }):Play()
    end)

    xrayToggleButton.InputBegan:Connect(function(input)
        if (input.UserInputType == Enum.UserInputType.MouseButton1) then
            toggleXRay()
        end
    end)

    xrayToggleButton.InputBegan:Connect(function(input)
        if (input.UserInputType == Enum.UserInputType.MouseButton1) then
            TweenService:Create(xrayToggleButton, TweenInfo.new(0.1), {
                BackgroundTransparency = 0.2
            }):Play()
        end
    end)

    xrayToggleButton.InputEnded:Connect(function(input)
        if (input.UserInputType == Enum.UserInputType.MouseButton1) then
            TweenService:Create(xrayToggleButton, TweenInfo.new(0.1), {
                BackgroundTransparency = 0
            }):Play()
        end
    end)

    xrayToggleFrame.MouseEnter:Connect(function()
        if not collectionStatus.XRayEnabled then
            TweenService:Create(xrayToggleButton, TweenInfo.new(0.3), {
                BackgroundColor3 = Color3.fromRGB(220, 70, 70)
            }):Play()
        else
            TweenService:Create(xrayToggleButton, TweenInfo.new(0.3), {
                BackgroundColor3 = Color3.fromRGB(70, 220, 70)
            }):Play()
        end
        TweenService:Create(xrayToggleFrame, TweenInfo.new(0.3), {
            BackgroundColor3 = Color3.fromRGB(45, 45, 45)
        }):Play()
    end)

    xrayToggleFrame.MouseLeave:Connect(function()
        if not collectionStatus.XRayEnabled then
            TweenService:Create(xrayToggleButton, TweenInfo.new(0.3), {
                BackgroundColor3 = Color3.fromRGB(200, 50, 50)
            }):Play()
        else
            TweenService:Create(xrayToggleButton, TweenInfo.new(0.3), {
                BackgroundColor3 = Color3.fromRGB(50, 200, 50)
            }):Play()
        end
        TweenService:Create(xrayToggleFrame, TweenInfo.new(0.3), {
            BackgroundColor3 = Color3.fromRGB(35, 35, 35)
        }):Play()
    end)

    UserInputService.InputBegan:Connect(function(input, gameProcessed)
        if ((input.KeyCode == Enum.KeyCode.K) and not gameProcessed) then
            toggleMouseControl()
        elseif ((input.KeyCode == Enum.KeyCode.X) and not gameProcessed) then
            toggleXRay()
        end
    end)

    topBar.InputBegan:Connect(function(input)
        if (input.UserInputType == Enum.UserInputType.MouseButton1) then
            local dragInput, dragStart, startPos
            local dragging = true

            dragStart = input.Position
            startPos = mainFrame.Position

            local connection = UserInputService.InputChanged:Connect(function(newInput)
                if (newInput.UserInputType == Enum.UserInputType.MouseMovement) then
                    dragInput = newInput
                end
            end)

            local function update()
                if (dragging and dragInput) then
                    local delta = dragInput.Position - dragStart
                    mainFrame.Position = UDim2.new(
                        startPos.X.Scale,
                        startPos.X.Offset + delta.X,
                        startPos.Y.Scale,
                        startPos.Y.Offset + delta.Y
                    )
                end
            end

            local renderStepped = RunService.RenderStepped:Connect(update)

            input.Changed:Connect(function()
                if (input.UserInputState == Enum.UserInputState.End) then
                    dragging = false
                    connection:Disconnect()
                    renderStepped:Disconnect()
                end
            end)
        end
    end)

    -------------------------------------------------------------------
    --  FINAL INIT
    -------------------------------------------------------------------
    applyCorners()
    enhanceStatsPanel()

    contentContainer.Visible = false
    UserInputService.MouseIconEnabled = false
end
