-- LocalScript (colocar em StarterGui)
-- Uso: script para Fly + Infinite Jump com uma GUI simples estilo "Fluent-like"
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")

-- ======= Configurações =======
local flySpeed = 100
local flyAccel = 60
local enabledFly = false
local enabledInfJump = true

-- ======= Cria GUI (simples, inspirado em Fluent) =======
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "DevAssistGUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

local mainFrame = Instance.new("Frame")
mainFrame.Name = "Main"
mainFrame.Size = UDim2.new(0, 220, 0, 120)
mainFrame.Position = UDim2.new(0, 20, 0, 20)
mainFrame.BackgroundColor3 = Color3.fromRGB(246, 248, 250) -- luz suave
mainFrame.BorderSizePixel = 0
mainFrame.AnchorPoint = Vector2.new(0,0)
mainFrame.Parent = screenGui
mainFrame.ClipsDescendants = true
mainFrame.ZIndex = 2
mainFrame.BackgroundTransparency = 0

-- Fluent-like rounded corners + shadow
local uicorner = Instance.new("UICorner", mainFrame)
uicorner.CornerRadius = UDim.new(0, 12)

local shadow = Instance.new("ImageLabel", mainFrame)
shadow.Size = UDim2.new(1,6,1,6)
shadow.Position = UDim2.new(0,-3,0,-3)
shadow.ZIndex = 0
shadow.BackgroundTransparency = 1
shadow.Image = "rbxassetid://7055227426" -- subtle shadow image (Roblox default)
shadow.ScaleType = Enum.ScaleType.Slice
shadow.SliceCenter = Rect.new(10,10,118,118)

-- Title
local title = Instance.new("TextLabel", mainFrame)
title.Size = UDim2.new(1, -16, 0, 28)
title.Position = UDim2.new(0, 8, 0, 8)
title.BackgroundTransparency = 1
title.Text = "Dev Assist"
title.Font = Enum.Font.GothamBold
title.TextSize = 16
title.TextColor3 = Color3.fromRGB(20,20,20)
title.TextXAlignment = Enum.TextXAlignment.Left

-- Fly Toggle
local flyToggle = Instance.new("TextButton", mainFrame)
flyToggle.Size = UDim2.new(1, -16, 0, 36)
flyToggle.Position = UDim2.new(0, 8, 0, 42)
flyToggle.BackgroundColor3 = Color3.fromRGB(255,255,255)
flyToggle.AutoButtonColor = true
flyToggle.Font = Enum.Font.Gotham
flyToggle.TextSize = 14
flyToggle.Text = "Fly: OFF"
flyToggle.TextColor3 = Color3.fromRGB(30,30,30)
flyToggle.BorderSizePixel = 0

local cornerToggle = Instance.new("UICorner", flyToggle)
cornerToggle.CornerRadius = UDim.new(0,8)

-- Infinite Jump Toggle
local ijToggle = Instance.new("TextButton", mainFrame)
ijToggle.Size = UDim2.new(1, -16, 0, 36)
ijToggle.Position = UDim2.new(0, 8, 0, 84)
ijToggle.BackgroundColor3 = Color3.fromRGB(255,255,255)
ijToggle.AutoButtonColor = true
ijToggle.Font = Enum.Font.Gotham
ijToggle.TextSize = 14
ijToggle.Text = "Infinite Jump: ON"
ijToggle.TextColor3 = Color3.fromRGB(30,30,30)
ijToggle.BorderSizePixel = 0

local cornerIJ = Instance.new("UICorner", ijToggle)
cornerIJ.CornerRadius = UDim.new(0,8)

-- ======= Fly implementation (Local) =======
local rootPart = nil
local flyVelocity = Vector3.new(0,0,0)
local bodyGyro = nil
local bodyVelocity = nil

local function enableFly()
    if not player.Character then return end
    character = player.Character
    rootPart = character:FindFirstChild("HumanoidRootPart")
    if not rootPart then return end

    bodyGyro = Instance.new("BodyGyro")
    bodyGyro.MaxTorque = Vector3.new(1e5,1e5,1e5)
    bodyGyro.P = 1250
    bodyGyro.Parent = rootPart

    bodyVelocity = Instance.new("BodyVelocity")
    bodyVelocity.MaxForce = Vector3.new(1e5,1e5,1e5)
    bodyVelocity.P = 1250
    bodyVelocity.Parent = rootPart

    enabledFly = true
    flyToggle.Text = "Fly: ON"
end

local function disableFly()
    if bodyGyro then bodyGyro:Destroy(); bodyGyro = nil end
    if bodyVelocity then bodyVelocity:Destroy(); bodyVelocity = nil end
    enabledFly = false
    flyToggle.Text = "Fly: OFF"
end

-- movement state
local moveVec = Vector3.new(0,0,0)
local speed = flySpeed

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    -- toggle with RightControl (example)
    if input.KeyCode == Enum.KeyCode.RightControl then
        if enabledFly then disableFly() else enableFly() end
    end
end)

-- GUI button handlers
flyToggle.MouseButton1Click:Connect(function()
    if enabledFly then disableFly() else enableFly() end
end)

ijToggle.MouseButton1Click:Connect(function()
    enabledInfJump = not enabledInfJump
    ijToggle.Text = "Infinite Jump: " .. (enabledInfJump and "ON" or "OFF")
end)

-- track movement input
local forward = 0
local right = 0
local up = 0

local function updateMove()
    if not rootPart then return end
    local cam = workspace.CurrentCamera
    local camCFrame = cam.CFrame
    local lookVector = Vector3.new(camCFrame.LookVector.X, 0, camCFrame.LookVector.Z).Unit
    if lookVector ~= lookVector then lookVector = Vector3.new(0,0,-1) end -- guard NaN
    local rightVector = Vector3.new(camCFrame.RightVector.X, 0, camCFrame.RightVector.Z).Unit
    local moveDirection = (lookVector * forward) + (rightVector * right) + Vector3.new(0, up, 0)
    moveDirection = moveDirection.Unit ~= moveDirection.Unit and Vector3.new(0,0,0) or moveDirection
    if bodyVelocity then
        bodyVelocity.Velocity = moveDirection * speed
    end
    if bodyGyro then
        bodyGyro.CFrame = CFrame.new(rootPart.Position, rootPart.Position + Vector3.new(camCFrame.LookVector.X, 0, camCFrame.LookVector.Z))
    end
end

-- keyboard control for fly
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.W then forward = 1
    elseif input.KeyCode == Enum.KeyCode.S then forward = -1
    elseif input.KeyCode == Enum.KeyCode.A then right = -1
    elseif input.KeyCode == Enum.KeyCode.D then right = 1
    elseif input.KeyCode == Enum.KeyCode.E then up = 1
    elseif input.KeyCode == Enum.KeyCode.Q then up = -1
    end
end)
UserInputService.InputEnded:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.W or input.KeyCode == Enum.KeyCode.S then forward = 0 end
    if input.KeyCode == Enum.KeyCode.A or input.KeyCode == Enum.KeyCode.D then right = 0 end
    if input.KeyCode == Enum.KeyCode.E or input.KeyCode == Enum.KeyCode.Q then up = 0 end
end)

-- ======= Infinite jump =======
UserInputService.JumpRequest:Connect(function()
    if enabledInfJump and humanoid and humanoid.Health > 0 then
        humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
    end
end)

-- ======= Update loop =======
RunService.RenderStepped:Connect(function(dt)
    if enabledFly and rootPart then
        updateMove()
    end
end)

-- cleanup on respawn
player.CharacterAdded:Connect(function(char)
    character = char
    humanoid = character:WaitForChild("Humanoid")
    rootPart = character:WaitForChild("HumanoidRootPart")
    -- ensure fly off when respawned (optional)
    if not enabledFly then
        if bodyGyro then bodyGyro:Destroy(); bodyGyro=nil end
        if bodyVelocity then bodyVelocity:Destroy(); bodyVelocity=nil end
    end
end)
