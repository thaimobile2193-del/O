local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local localPlayer = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- ==========================================
-- 1. ฟังก์ชันตรวจสอบศัตรู (ล็อกเงื่อนไขเดียวกับ ESP)
-- ==========================================
local function isValidTarget(targetPlayer)
    -- 1. ต้องมีตัวตนและไม่ใช่ตัวเรา
    if targetPlayer and targetPlayer ~= localPlayer then
        -- 2. ต้องมีตัวละคร (Character) และค่าสถานะ (Humanoid)
        if targetPlayer.Character and targetPlayer.Character:FindFirstChild("Humanoid") then
            -- 3. ต้องยังมีชีวิตอยู่ (เลือด > 0)
            if targetPlayer.Character.Humanoid.Health > 0 then
                -- 4. ต้องอยู่คนละทีมกับเรา
                if targetPlayer.Team ~= localPlayer.Team then
                    return true -- ถือว่าเป็นเป้าหมายที่ถูกต้อง
                end
            end
        end
    end
    return false
end

-- ==========================================
-- 2. สร้าง UI (Modern Dark Theme)
-- ==========================================
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "TargetSystemUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = localPlayer:WaitForChild("PlayerGui")

local fovRadius = 150
local maxFov = 600

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 260, 0, 280) 
mainFrame.Position = UDim2.new(0.5, -130, 0.8, -140)
mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Parent = screenGui

local frameCorner = Instance.new("UICorner")
frameCorner.CornerRadius = UDim.new(0, 12)
frameCorner.Parent = mainFrame

local frameStroke = Instance.new("UIStroke")
frameStroke.Color = Color3.fromRGB(70, 70, 80)
frameStroke.Thickness = 1.5
frameStroke.Parent = mainFrame

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, 0, 0, 45)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "Target System"
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 20
titleLabel.Parent = mainFrame

-- ระบบลาก UI หลัก (Draggable)
local dragging, dragInput, dragStart, startPos
mainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = mainFrame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then dragging = false end
        end)
    end
end)
mainFrame.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - dragStart
        mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

-- ปุ่ม 1: ESP
local espBtn = Instance.new("TextButton")
espBtn.Size = UDim2.new(0.85, 0, 0, 40)
espBtn.Position = UDim2.new(0.075, 0, 0, 50)
espBtn.BackgroundColor3 = Color3.fromRGB(220, 60, 60)
espBtn.Text = "ESP : OFF"
espBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
espBtn.Font = Enum.Font.GothamSemibold
espBtn.TextSize = 16
espBtn.Parent = mainFrame
Instance.new("UICorner", espBtn).CornerRadius = UDim.new(0, 8)

-- ปุ่ม 2: Lock-On
local aimBtn = Instance.new("TextButton")
aimBtn.Size = UDim2.new(0.85, 0, 0, 40)
aimBtn.Position = UDim2.new(0.075, 0, 0, 100)
aimBtn.BackgroundColor3 = Color3.fromRGB(220, 60, 60)
aimBtn.Text = "LOCK-ON : OFF"
aimBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
aimBtn.Font = Enum.Font.GothamSemibold
aimBtn.TextSize = 16
aimBtn.Parent = mainFrame
Instance.new("UICorner", aimBtn).CornerRadius = UDim.new(0, 8)

-- ==========================================
-- วงกลม FOV กลางจอ
-- ==========================================
local fovCircle = Instance.new("Frame")
fovCircle.AnchorPoint = Vector2.new(0.5, 0.5)
fovCircle.Position = UDim2.new(0.5, 0, 0.5, 0)
fovCircle.Size = UDim2.new(0, fovRadius * 2, 0, fovRadius * 2)
fovCircle.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
fovCircle.BackgroundTransparency = 1 
fovCircle.Active = true
fovCircle.Parent = screenGui

local fovCorner = Instance.new("UICorner")
fovCorner.CornerRadius = UDim.new(1, 0)
fovCorner.Parent = fovCircle

local fovStroke = Instance.new("UIStroke")
fovStroke.Color = Color3.fromRGB(255, 255, 255)
fovStroke.Thickness = 1.5
fovStroke.Transparency = 0.3
fovStroke.Parent = fovCircle

-- ระบบลาก FOV วงกลม
local isFovLocked = true
local fovDragging, fovDragInput, fovDragStart, fovStartPos

fovCircle.InputBegan:Connect(function(input)
    if not isFovLocked and (input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch) then
        fovDragging = true
        fovDragStart = input.Position
        fovStartPos = fovCircle.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then fovDragging = false end
        end)
    end
end)
fovCircle.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        fovDragInput = input
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if input == fovDragInput and fovDragging and not isFovLocked then
        local delta = input.Position - fovDragStart
        fovCircle.Position = UDim2.new(fovStartPos.X.Scale, fovStartPos.X.Offset + delta.X, fovStartPos.Y.Scale, fovStartPos.Y.Offset + delta.Y)
    end
end)

-- Slider ปรับขนาดวงกลม
local fovLabel = Instance.new("TextLabel")
fovLabel.Size = UDim2.new(0.85, 0, 0, 20)
fovLabel.Position = UDim2.new(0.075, 0, 0, 155)
fovLabel.BackgroundTransparency = 1
fovLabel.Text = "FOV Size : " .. fovRadius
fovLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
fovLabel.Font = Enum.Font.GothamSemibold
fovLabel.TextSize = 14
fovLabel.Parent = mainFrame

local sliderBg = Instance.new("Frame")
sliderBg.Size = UDim2.new(0.85, 0, 0, 12)
sliderBg.Position = UDim2.new(0.075, 0, 0, 180)
sliderBg.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
sliderBg.Parent = mainFrame
Instance.new("UICorner", sliderBg).CornerRadius = UDim.new(1, 0)

local sliderFill = Instance.new("Frame")
sliderFill.Size = UDim2.new(fovRadius / maxFov, 0, 1, 0)
sliderFill.BackgroundColor3 = Color3.fromRGB(60, 200, 100)
sliderFill.Parent = sliderBg
Instance.new("UICorner", sliderFill).CornerRadius = UDim.new(1, 0)

local sliderBtn = Instance.new("TextButton")
sliderBtn.Size = UDim2.new(1, 0, 2.5, 0)
sliderBtn.Position = UDim2.new(0, 0, -0.75, 0)
sliderBtn.BackgroundTransparency = 1
sliderBtn.Text = ""
sliderBtn.Parent = sliderBg

local draggingSlider = false
sliderBtn.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        draggingSlider = true
    end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        draggingSlider = false
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if draggingSlider and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local inputX = input.Position.X
        local sliderX = sliderBg.AbsolutePosition.X
        local sliderWidth = sliderBg.AbsoluteSize.X
        
        local percent = math.clamp((inputX - sliderX) / sliderWidth, 0.05, 1)
        sliderFill.Size = UDim2.new(percent, 0, 1, 0)
        fovRadius = math.floor(percent * maxFov)
        fovLabel.Text = "FOV Size : " .. fovRadius
        fovCircle.Size = UDim2.new(0, fovRadius * 2, 0, fovRadius * 2)
    end
end)

-- ปุ่ม 3: Lock/Unlock FOV Circle
local lockFovBtn = Instance.new("TextButton")
lockFovBtn.Size = UDim2.new(0.85, 0, 0, 40)
lockFovBtn.Position = UDim2.new(0.075, 0, 0, 215) 
lockFovBtn.BackgroundColor3 = Color3.fromRGB(220, 60, 60)
lockFovBtn.Text = "CIRCLE : LOCKED"
lockFovBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
lockFovBtn.Font = Enum.Font.GothamSemibold
lockFovBtn.TextSize = 16
lockFovBtn.Parent = mainFrame
Instance.new("UICorner", lockFovBtn).CornerRadius = UDim.new(0, 8)

lockFovBtn.MouseButton1Click:Connect(function()
    isFovLocked = not isFovLocked
    if isFovLocked then
        lockFovBtn.BackgroundColor3 = Color3.fromRGB(220, 60, 60)
        lockFovBtn.Text = "CIRCLE : LOCKED"
        fovCircle.BackgroundTransparency = 1 
    else
        lockFovBtn.BackgroundColor3 = Color3.fromRGB(60, 200, 100)
        lockFovBtn.Text = "CIRCLE : UNLOCKED"
        fovCircle.BackgroundTransparency = 0.85 
    end
end)

-- ==========================================
-- 3. ระบบการทำงาน (Logic)
-- ==========================================
local espActive = false
local aimActive = false
local espConnection
local aimConnection

-- [ ฟังก์ชันระบบ ESP แบบลดอาการแลค (Optimized) ] --
local function highlightEnemies()
    if not espActive then
        -- ถ้าปิด ESP ลบเส้นขอบออกทั้งหมด
        for _, player in pairs(Players:GetPlayers()) do
            if player.Character and player.Character:FindFirstChild("EnemyHighlight") then
                player.Character.EnemyHighlight:Destroy()
            end
        end
        return
    end

    for _, player in pairs(Players:GetPlayers()) do
        if player.Character then
            local hasHighlight = player.Character:FindFirstChild("EnemyHighlight")
            
            -- ถ้าตรงตามเงื่อนไขเดียวกับ Lock-on เป๊ะๆ ให้แสดงสีแดง
            if isValidTarget(player) then
                if not hasHighlight then
                    local highlight = Instance.new("Highlight")
                    highlight.Name = "EnemyHighlight"
                    highlight.FillColor = Color3.fromRGB(255, 0, 0)
                    highlight.FillTransparency = 0.5
                    highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                    highlight.Parent = player.Character
                end
            else
                -- ถ้าตาย หรือ ย้ายทีม ให้ลบขอบแดงทิ้งทันที
                if hasHighlight then
                    hasHighlight:Destroy()
                end
            end
        end
    end
end

-- [ ฟังก์ชันหาศัตรูในวงกลมหน้าจอ (FOV) ] --
local function getEnemyInFOV()
    local closestDist = fovRadius 
    local closestTarget = nil
    
    if not localPlayer.Character or not localPlayer.Character:FindFirstChild("HumanoidRootPart") then 
        return nil 
    end
    
    local circleCenter = fovCircle.AbsolutePosition + (fovCircle.AbsoluteSize / 2)

    for _, player in pairs(Players:GetPlayers()) do
        -- *** ล็อกเป้าเฉพาะคนที่ผ่านเงื่อนไขเดียวกับ ESP เท่านั้น ***
        if isValidTarget(player) then
            local rootPart = player.Character:FindFirstChild("HumanoidRootPart")
            if rootPart then
                local screenPos, onScreen = camera:WorldToViewportPoint(rootPart.Position)
                
                if onScreen then
                    local dist = (Vector2.new(screenPos.X, screenPos.Y) - circleCenter).Magnitude
                    
                    if dist < closestDist then
                        closestDist = dist
                        closestTarget = rootPart
                    end
                end
            end
        end
    end
    return closestTarget
end

-- ==========================================
-- 4. เชื่อมต่อปุ่มกด (Events)
-- ==========================================
espBtn.MouseButton1Click:Connect(function()
    espActive = not espActive
    if espActive then
        espBtn.BackgroundColor3 = Color3.fromRGB(60, 200, 100)
        espBtn.Text = "ESP : ON"
        espConnection = RunService.RenderStepped:Connect(function()
            highlightEnemies()
        end)
    else
        espBtn.BackgroundColor3 = Color3.fromRGB(220, 60, 60)
        espBtn.Text = "ESP : OFF"
        if espConnection then espConnection:Disconnect() end
        highlightEnemies()
    end
end)

aimBtn.MouseButton1Click:Connect(function()
    aimActive = not aimActive
    if aimActive then
        aimBtn.BackgroundColor3 = Color3.fromRGB(60, 200, 100)
        aimBtn.Text = "LOCK-ON : ON"
        
        aimConnection = RunService.RenderStepped:Connect(function()
            local targetPart = getEnemyInFOV()
            if targetPart then
                camera.CFrame = CFrame.lookAt(camera.CFrame.Position, targetPart.Position)
            end
        end)
    else
        aimBtn.BackgroundColor3 = Color3.fromRGB(220, 60, 60)
        aimBtn.Text = "LOCK-ON : OFF"
        if aimConnection then aimConnection:Disconnect() end
    end
end)
