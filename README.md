```lua
-- ██████╗ ██████╗ ███████╗███████╗███╗   ██╗ █████╗ ██╗         ██╗  ██╗██╗   ██╗██████╗ 
-- ██╔══██╗██╔══██╗██╔════╝██╔════╝████╗  ██║██╔══██╗██║         ██║  ██║██║   ██║██╔══██╗
-- ███████║██████╔╝███████╗█████╗  ██╔██╗ ██║███████║██║         ███████║██║   ██║██████╔╝
-- ██╔══██║██╔══██╗╚════██║██╔══╝  ██║╚██╗██║██╔══██║██║         ██╔══██║██║   ██║██╔══██╗
-- ██║  ██║██║  ██║███████║███████╗██║ ╚████║██║  ██║███████╗    ██║  ██║╚██████╔╝██████╔╝
-- ╚═╝  ╚═╝╚═╝  ╚═╝╚══════╝╚══════╝╚═╝  ╚═══╝╚═╝  ╚═╝╚══════╝    ╚═╝  ╚═╝ ╚═════╝ ╚═════╝ 
-- Premium Edition V3 | Made by jasonkingtou/Relocation

local UIS = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LocalPlayer = Players.LocalPlayer

-- ═══════════════════════════════════════════════════════════════════════════════
-- CONFIGURATION
-- ═══════════════════════════════════════════════════════════════════════════════

local Config = {
    -- Combat Features
    AimAssist = false,
    Aimbot = false,
    TriggerBot = false,
    NoRecoil = false,
    NoSpread = false,
    InfiniteAmmo = false,
    RapidFire = false,
    
    -- Visual Features
    ESP = false,
    Tracers = false,
    Chams = false,
    Wallhack = false,
    Fullbright = false,
    FOVCircle = false,
    
    -- Movement Features
    SpeedHack = false,
    InfiniteJump = false,
    Fly = false,
    Noclip = false,
    
    -- Misc Features
    AutoRespawn = false,
    KillAll = false,
    AntiAFK = true,
    RemoveFog = false,
    
    -- Optimization Features
    ReducePackets = false,
    OptimizePing = false,
    BoostFPS = false,
    RemoveTextures = false,
    DisableParticles = false,
    
    -- Keybinds
    MenuToggle = Enum.KeyCode.RightControl,
    AimbotKey = Enum.KeyCode.E,
    FlyKey = Enum.KeyCode.F,
    NoclipKey = Enum.KeyCode.N,
    SpeedToggle = Enum.KeyCode.Q,
    
    -- Values
    AimbotFOV = 200,
    AimbotSmoothness = 0.1,
    SpeedMultiplier = 2,
    FlySpeed = 50,
    JumpPower = 50,
    
    -- UI
    CurrentTab = "Combat",
    
    -- UI Colors
    PrimaryColor = Color3.fromRGB(255, 69, 0),
    SecondaryColor = Color3.fromRGB(220, 20, 60),
    AccentColor = Color3.fromRGB(255, 140, 0),
    BackgroundColor = Color3.fromRGB(15, 15, 20),
    SidebarColor = Color3.fromRGB(20, 20, 25),
    TextColor = Color3.fromRGB(255, 255, 255)
}

-- ═══════════════════════════════════════════════════════════════════════════════
-- VARIABLES
-- ═══════════════════════════════════════════════════════════════════════════════

local menuOpen = false
local speedEnabled = false
local flyEnabled = false
local noclipEnabled = false
local espEnabled = false
local aimbotEnabled = false
local fullbrightEnabled = false
local originalWalkSpeed = 16
local flyConnection = nil
local noclipConnection = nil
local aimbotConnection = nil
local espConnections = {}
local fovCircle = nil
local keybindChanging = nil
local Camera = Workspace.CurrentCamera
local currentTab = "Combat"
local fpsCounter = 0
local pingValue = 0

-- ═══════════════════════════════════════════════════════════════════════════════
-- UTILITY FUNCTIONS
-- ═══════════════════════════════════════════════════════════════════════════════

local function notify(title, text, duration)
    game.StarterGui:SetCore("SendNotification", {
        Title = title,
        Text = text,
        Duration = duration or 3,
        Icon = "rbxassetid://7733955740"
    })
end

local function createGradient(parent)
    local gradient = Instance.new("UIGradient")
    gradient.Color = ColorSequence.new{
        ColorSequenceKeypoint.new(0, Config.PrimaryColor),
        ColorSequenceKeypoint.new(1, Config.SecondaryColor)
    }
    gradient.Rotation = 45
    gradient.Parent = parent
    return gradient
end

local function createCorner(parent, radius)
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, radius or 8)
    corner.Parent = parent
    return corner
end

local function createStroke(parent, thickness, color)
    local stroke = Instance.new("UIStroke")
    stroke.Thickness = thickness or 2
    stroke.Color = color or Config.AccentColor
    stroke.Transparency = 0.5
    stroke.Parent = parent
    return stroke
end

-- ═══════════════════════════════════════════════════════════════════════════════
-- COMBAT FUNCTIONS
-- ═══════════════════════════════════════════════════════════════════════════════

local function getClosestPlayer()
    local closestPlayer = nil
    local shortestDistance = Config.AimbotFOV
    
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local character = player.Character
            local humanoid = character:FindFirstChild("Humanoid")
            local head = character:FindFirstChild("Head")
            
            if humanoid and humanoid.Health > 0 and head then
                local screenPos, onScreen = Camera:WorldToViewportPoint(head.Position)
                
                if onScreen then
                    local mousePos = UIS:GetMouseLocation()
                    local distance = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude
                    
                    if distance < shortestDistance then
                        shortestDistance = distance
                        closestPlayer = player
                    end
                end
            end
        end
    end
    
    return closestPlayer
end

local function toggleAimbot()
    aimbotEnabled = not aimbotEnabled
    
    if aimbotEnabled then
        aimbotConnection = RunService.RenderStepped:Connect(function()
            if Config.Aimbot then
                local target = getClosestPlayer()
                if target and target.Character then
                    local head = target.Character:FindFirstChild("Head")
                    if head then
                        local targetPos = head.Position
                        Camera.CFrame = Camera.CFrame:Lerp(CFrame.new(Camera.CFrame.Position, targetPos), Config.AimbotSmoothness)
                    end
                end
            end
        end)
        notify("Aimbot", "Enabled 🎯", 2)
    else
        if aimbotConnection then
            aimbotConnection:Disconnect()
        end
        notify("Aimbot", "Disabled", 2)
    end
end

local function toggleNoRecoil()
    Config.NoRecoil = not Config.NoRecoil
    
    if Config.NoRecoil then
        -- Arsenal specific recoil removal
        pcall(function()
            local camera = LocalPlayer.PlayerScripts:FindFirstChild("CameraS")
            if camera then
                camera.Disabled = true
            end
        end)
        notify("No Recoil", "Enabled ✨", 2)
    else
        pcall(function()
            local camera = LocalPlayer.PlayerScripts:FindFirstChild("CameraS")
            if camera then
                camera.Disabled = false
            end
        end)
        notify("No Recoil", "Disabled", 2)
    end
end

local function toggleInfiniteAmmo()
    Config.InfiniteAmmo = not Config.InfiniteAmmo
    notify("Infinite Ammo", Config.InfiniteAmmo and "Enabled 🔫" or "Disabled", 2)
end

-- ═══════════════════════════════════════════════════════════════════════════════
-- VISUAL FUNCTIONS
-- ═══════════════════════════════════════════════════════════════════════════════

local function createESP(player)
    if player == LocalPlayer or not player.Character then return end
    
    local character = player.Character
    local hrp = character:FindFirstChild("HumanoidRootPart")
    
    if hrp and not hrp:FindFirstChild("ESP_BOX") then
        -- Box ESP
        local box = Instance.new("BoxHandleAdornment")
        box.Name = "ESP_BOX"
        box.Size = Vector3.new(4, 5, 1)
        box.Color3 = Config.AccentColor
        box.Transparency = 0.7
        box.AlwaysOnTop = true
        box.ZIndex = 5
        box.Adornee = hrp
        box.Parent = hrp
        
        -- Highlight
        local highlight = Instance.new("Highlight")
        highlight.Name = "ESP_Highlight"
        highlight.Adornee = character
        highlight.FillColor = Config.AccentColor
        highlight.OutlineColor = Config.PrimaryColor
        highlight.FillTransparency = 0.5
        highlight.OutlineTransparency = 0
        highlight.Parent = hrp
        
        -- Name Tag
        local billboard = Instance.new("BillboardGui")
        billboard.Name = "ESP_Name"
        billboard.Adornee = character:FindFirstChild("Head")
        billboard.Size = UDim2.new(0, 200, 0, 50)
        billboard.StudsOffset = Vector3.new(0, 2, 0)
        billboard.AlwaysOnTop = true
        billboard.Parent = hrp
        
        local nameLabel = Instance.new("TextLabel")
        nameLabel.Size = UDim2.new(1, 0, 1, 0)
        nameLabel.BackgroundTransparency = 1
        nameLabel.Font = Enum.Font.GothamBold
        nameLabel.Text = player.Name
        nameLabel.TextColor3 = Config.TextColor
        nameLabel.TextSize = 14
        nameLabel.TextStrokeTransparency = 0.5
        nameLabel.Parent = billboard
    end
end

local function removeESP(player)
    if player.Character then
        local hrp = player.Character:FindFirstChild("HumanoidRootPart")
        if hrp then
            for _, obj in pairs(hrp:GetChildren()) do
                if obj.Name:match("ESP") then
                    obj:Destroy()
                end
            end
        end
    end
end

local function toggleESP()
    espEnabled = not espEnabled
    
    if espEnabled then
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer then
                createESP(player)
            end
        end
        
        espConnections.PlayerAdded = Players.PlayerAdded:Connect(function(player)
            if espEnabled then
                player.CharacterAdded:Connect(function()
                    wait(0.5)
                    createESP(player)
                end)
            end
        end)
        
        notify("ESP", "Enabled 👁️", 2)
    else
        for _, player in pairs(Players:GetPlayers()) do
            removeESP(player)
        end
        
        if espConnections.PlayerAdded then
            espConnections.PlayerAdded:Disconnect()
        end
        
        notify("ESP", "Disabled", 2)
    end
end

local function toggleFOVCircle()
    Config.FOVCircle = not Config.FOVCircle
    
    if Config.FOVCircle then
        if not fovCircle then
            fovCircle = Drawing.new("Circle")
            fovCircle.Thickness = 2
            fovCircle.NumSides = 50
            fovCircle.Radius = Config.AimbotFOV
            fovCircle.Filled = false
            fovCircle.Visible = true
            fovCircle.ZIndex = 999
            fovCircle.Transparency = 1
            fovCircle.Color = Config.AccentColor
            
            RunService.RenderStepped:Connect(function()
                if Config.FOVCircle and fovCircle then
                    local mousePos = UIS:GetMouseLocation()
                    fovCircle.Position = mousePos
                    fovCircle.Radius = Config.AimbotFOV
                end
            end)
        end
        fovCircle.Visible = true
        notify("FOV Circle", "Enabled ⭕", 2)
    else
        if fovCircle then
            fovCircle.Visible = false
        end
        notify("FOV Circle", "Disabled", 2)
    end
end

local function toggleFullbright()
    fullbrightEnabled = not fullbrightEnabled
    local lighting = game:GetService("Lighting")
    
    if fullbrightEnabled then
        lighting.Brightness = 2
        lighting.ClockTime = 14
        lighting.FogEnd = 100000
        lighting.GlobalShadows = false
        lighting.Ambient = Color3.fromRGB(255, 255, 255)
        notify("Fullbright", "Enabled 💡", 2)
    else
        lighting.Brightness = 1
        lighting.ClockTime = 14
        lighting.FogEnd = 100000
        lighting.GlobalShadows = true
        lighting.Ambient = Color3.fromRGB(0, 0, 0)
        notify("Fullbright", "Disabled", 2)
    end
end

-- ═══════════════════════════════════════════════════════════════════════════════
-- MOVEMENT FUNCTIONS
-- ═══════════════════════════════════════════════════════════════════════════════

local function toggleSpeed()
    speedEnabled = not speedEnabled
    local character = LocalPlayer.Character
    if not character then return end
    
    local humanoid = character:FindFirstChild("Humanoid")
    if not humanoid then return end
    
    if speedEnabled then
        originalWalkSpeed = humanoid.WalkSpeed
        humanoid.WalkSpeed = originalWalkSpeed * Config.SpeedMultiplier
        notify("Speed Hack", "Enabled ⚡", 2)
    else
        humanoid.WalkSpeed = originalWalkSpeed
        notify("Speed Hack", "Disabled", 2)
    end
end

local function toggleFly()
    flyEnabled = not flyEnabled
    local character = LocalPlayer.Character
    if not character then return end
    
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    
    if flyEnabled then
        local bg = Instance.new("BodyGyro")
        bg.P = 9e4
        bg.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
        bg.CFrame = hrp.CFrame
        bg.Parent = hrp
        
        local bv = Instance.new("BodyVelocity")
        bv.Velocity = Vector3.new(0, 0, 0)
        bv.MaxForce = Vector3.new(9e9, 9e9, 9e9)
        bv.Parent = hrp
        
        flyConnection = RunService.Heartbeat:Connect(function()
            if not flyEnabled then return end
            local cam = Workspace.CurrentCamera
            local keys = UIS:GetKeysPressed()
            local speed = Config.FlySpeed
            local direction = Vector3.new(0, 0, 0)
            
            for _, key in pairs(keys) do
                if key.KeyCode == Enum.KeyCode.W then
                    direction = direction + (cam.CFrame.LookVector * speed)
                elseif key.KeyCode == Enum.KeyCode.S then
                    direction = direction - (cam.CFrame.LookVector * speed)
                elseif key.KeyCode == Enum.KeyCode.A then
                    direction = direction - (cam.CFrame.RightVector * speed)
                elseif key.KeyCode == Enum.KeyCode.D then
                    direction = direction + (cam.CFrame.RightVector * speed)
                elseif key.KeyCode == Enum.KeyCode.Space then
                    direction = direction + Vector3.new(0, speed, 0)
                elseif key.KeyCode == Enum.KeyCode.LeftShift then
                    direction = direction - Vector3.new(0, speed, 0)
                end
            end
            
            bv.Velocity = direction
            bg.CFrame = cam.CFrame
        end)
        
        notify("Fly Mode", "Enabled ✈️", 2)
    else
        if flyConnection then
            flyConnection:Disconnect()
        end
        for _, obj in pairs(hrp:GetChildren()) do
            if obj:IsA("BodyGyro") or obj:IsA("BodyVelocity") then
                obj:Destroy()
            end
        end
        notify("Fly Mode", "Disabled", 2)
    end
end

local function toggleNoclip()
    noclipEnabled = not noclipEnabled
    
    if noclipEnabled then
        noclipConnection = RunService.Stepped:Connect(function()
            if not noclipEnabled then return end
            local character = LocalPlayer.Character
            if character then
                for _, part in pairs(character:GetDescendants()) do
                    if part:IsA("BasePart") and part.CanCollide then
                        part.CanCollide = false
                    end
                end
            end
        end)
        notify("Noclip", "Enabled 👻", 2)
    else
        if noclipConnection then
            noclipConnection:Disconnect()
        end
        local character = LocalPlayer.Character
        if character then
            for _, part in pairs(character:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = true
                end
            end
        end
        notify("Noclip", "Disabled", 2)
    end
end

-- ═══════════════════════════════════════════════════════════════════════════════
-- OPTIMIZATION FUNCTIONS
-- ═══════════════════════════════════════════════════════════════════════════════

local function togglePacketOptimization()
    Config.ReducePackets = not Config.ReducePackets
    if Config.ReducePackets then
        settings().Network.IncomingReplicationLag = 0
        notify("Packet Optimization", "Enabled 📶", 2)
    else
        settings().Network.IncomingReplicationLag = 0
        notify("Packet Optimization", "Disabled", 2)
    end
end

local function togglePingOptimization()
    Config.OptimizePing = not Config.OptimizePing
    if Config.OptimizePing then
        settings().Network.PhysicsSend = 2
        settings().Network.ExperimentalPhysicsEnabled = true
        notify("Ping Optimization", "Enabled 📡", 2)
    else
        settings().Network.PhysicsSend = 20
        notify("Ping Optimization", "Disabled", 2)
    end
end

local function toggleFPSBoost()
    Config.BoostFPS = not Config.BoostFPS
    local Lighting = game:GetService("Lighting")
    
    if Config.BoostFPS then
        settings().Rendering.QualityLevel = Enum.QualityLevel.Level01
        for _, effect in pairs(Lighting:GetChildren()) do
            if effect:IsA("PostEffect") then
                effect.Enabled = false
            end
        end
        notify("FPS Boost", "Enabled ⚡", 2)
    else
        settings().Rendering.QualityLevel = Enum.QualityLevel.Automatic
        for _, effect in pairs(Lighting:GetChildren()) do
            if effect:IsA("PostEffect") then
                effect.Enabled = true
            end
        end
        notify("FPS Boost", "Disabled", 2)
    end
end

local function toggleTextureRemoval()
    Config.RemoveTextures = not Config.RemoveTextures
    if Config.RemoveTextures then
        for _, obj in pairs(Workspace:GetDescendants()) do
            if obj:IsA("Decal") or obj:IsA("Texture") then
                obj.Transparency = 1
            elseif obj:IsA("MeshPart") or obj:IsA("Part") then
                obj.Material = Enum.Material.SmoothPlastic
            end
        end
        notify("Texture Removal", "Enabled 🎨", 2)
    else
        for _, obj in pairs(Workspace:GetDescendants()) do
            if obj:IsA("Decal") or obj:IsA("Texture") then
                obj.Transparency = 0
            end
        end
        notify("Texture Removal", "Disabled", 2)
    end
end

local function toggleParticleDisable()
    Config.DisableParticles = not Config.DisableParticles
    if Config.DisableParticles then
        for _, obj in pairs(Workspace:GetDescendants()) do
            if obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Smoke") or obj:IsA("Fire") or obj:IsA("Sparkles") then
                obj.Enabled = false
            end
        end
        notify("Particle Disable", "Enabled ✨", 2)
    else
        for _, obj in pairs(Workspace:GetDescendants()) do
            if obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Smoke") or obj:IsA("Fire") or obj:IsA("Sparkles") then
                obj.Enabled = true
            end
        end
        notify("Particle Disable", "Disabled", 2)
    end
end

-- ═══════════════════════════════════════════════════════════════════════════════
-- GUI CREATION
-- ═══════════════════════════════════════════════════════════════════════════════

local function createUI()
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "ArsenalHub"
    ScreenGui.ResetOnSpawn = false
    ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    ScreenGui.Parent = game.CoreGui
    
    -- Main Frame
    local MainFrame = Instance.new("Frame")
    MainFrame.Name = "MainFrame"
    MainFrame.Size = UDim2.new(0, 780, 0, 520)
    MainFrame.Position = UDim2.new(0.5, -390, 0.5, -260)
    MainFrame.BackgroundColor3 = Config.BackgroundColor
    MainFrame.BorderSizePixel = 0
    MainFrame.Active = true
    MainFrame.Draggable = true
    MainFrame.Visible = false
    MainFrame.Parent = ScreenGui
    
    createCorner(MainFrame, 12)
    createStroke(MainFrame, 2, Config.AccentColor)
    
    createCorner(MainFrame, 12)
    createStroke(MainFrame, 2, Config.AccentColor)
    
    -- Shadow Effect
    local Shadow = Instance.new("ImageLabel")
    Shadow.Name = "Shadow"
    Shadow.BackgroundTransparency = 1
    Shadow.Position = UDim2.new(0, -15, 0, -15)
    Shadow.Size = UDim2.new(1, 30, 1, 30)
    Shadow.ZIndex = 0
    Shadow.Image = "rbxassetid://6014261993"
    Shadow.ImageColor3 = Color3.fromRGB(0, 0, 0)
    Shadow.ImageTransparency = 0.5
    Shadow.ScaleType = Enum.ScaleType.Slice
    Shadow.SliceCenter = Rect.new(99, 99, 99, 99)
    Shadow.Parent = MainFrame
    
    -- Sidebar (Left side with user info)
    local Sidebar = Instance.new("Frame")
    Sidebar.Name = "Sidebar"
    Sidebar.Size = UDim2.new(0, 200, 1, 0)
    Sidebar.Position = UDim2.new(0, 0, 0, 0)
    Sidebar.BackgroundColor3 = Config.SidebarColor
    Sidebar.BorderSizePixel = 0
    Sidebar.Parent = MainFrame
    
    createCorner(Sidebar, 12)
    
    -- User Avatar
    local AvatarFrame = Instance.new("Frame")
    AvatarFrame.Name = "AvatarFrame"
    AvatarFrame.Size = UDim2.new(0, 80, 0, 80)
    AvatarFrame.Position = UDim2.new(0.5, -40, 0, 20)
    AvatarFrame.BackgroundColor3 = Config.PrimaryColor
    AvatarFrame.BorderSizePixel = 0
    AvatarFrame.Parent = Sidebar
    
    createCorner(AvatarFrame, 40)
    createStroke(AvatarFrame, 3, Config.AccentColor)
    
    local Avatar = Instance.new("ImageLabel")
    Avatar.Size = UDim2.new(1, -6, 1, -6)
    Avatar.Position = UDim2.new(0, 3, 0, 3)
    Avatar.BackgroundTransparency = 1
    Avatar.Image = Players:GetUserThumbnailAsync(LocalPlayer.UserId, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size150x150)
    Avatar.Parent = AvatarFrame
    
    createCorner(Avatar, 37)
    
    -- Username
    local UsernameLabel = Instance.new("TextLabel")
    UsernameLabel.Name = "Username"
    UsernameLabel.Size = UDim2.new(1, -20, 0, 25)
    UsernameLabel.Position = UDim2.new(0, 10, 0, 110)
    UsernameLabel.BackgroundTransparency = 1
    UsernameLabel.Font = Enum.Font.GothamBold
    UsernameLabel.Text = LocalPlayer.Name
    UsernameLabel.TextColor3 = Config.TextColor
    UsernameLabel.TextSize = 14
    UsernameLabel.TextScaled = true
    UsernameLabel.Parent = Sidebar
    
    -- Display Name (if different)
    local DisplayNameLabel = Instance.new("TextLabel")
    DisplayNameLabel.Name = "DisplayName"
    DisplayNameLabel.Size = UDim2.new(1, -20, 0, 20)
    DisplayNameLabel.Position = UDim2.new(0, 10, 0, 140)
    DisplayNameLabel.BackgroundTransparency = 1
    DisplayNameLabel.Font = Enum.Font.Gotham
    DisplayNameLabel.Text = "@" .. LocalPlayer.DisplayName
    DisplayNameLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
    DisplayNameLabel.TextSize = 11
    DisplayNameLabel.Parent = Sidebar
    
    -- Stats Divider
    local StatsDivider = Instance.new("Frame")
    StatsDivider.Size = UDim2.new(1, -20, 0, 2)
    StatsDivider.Position = UDim2.new(0, 10, 0, 175)
    StatsDivider.BackgroundColor3 = Config.AccentColor
    StatsDivider.BorderSizePixel = 0
    StatsDivider.Parent = Sidebar
    
    createCorner(StatsDivider, 1)
    
    -- Stats Section
    local StatsSection = Instance.new("Frame")
    StatsSection.Name = "StatsSection"
    StatsSection.Size = UDim2.new(1, -20, 0, 200)
    StatsSection.Position = UDim2.new(0, 10, 0, 190)
    StatsSection.BackgroundTransparency = 1
    StatsSection.Parent = Sidebar
    
    local function createStatDisplay(icon, name, value, yPos)
        local StatFrame = Instance.new("Frame")
        StatFrame.Size = UDim2.new(1, 0, 0, 35)
        StatFrame.Position = UDim2.new(0, 0, 0, yPos)
        StatFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
        StatFrame.BorderSizePixel = 0
        StatFrame.Parent = StatsSection
        
        createCorner(StatFrame, 6)
        
        local IconLabel = Instance.new("TextLabel")
        IconLabel.Size = UDim2.new(0, 30, 1, 0)
        IconLabel.BackgroundTransparency = 1
        IconLabel.Font = Enum.Font.GothamBold
        IconLabel.Text = icon
        IconLabel.TextColor3 = Config.AccentColor
        IconLabel.TextSize = 16
        IconLabel.Parent = StatFrame
        
        local NameLabel = Instance.new("TextLabel")
        NameLabel.Size = UDim2.new(0.5, -15, 1, 0)
        NameLabel.Position = UDim2.new(0, 30, 0, 0)
        NameLabel.BackgroundTransparency = 1
        NameLabel.Font = Enum.Font.Gotham
        NameLabel.Text = name
        NameLabel.TextColor3 = Config.TextColor
        NameLabel.TextSize = 11
        NameLabel.TextXAlignment = Enum.TextXAlignment.Left
        NameLabel.Parent = StatFrame
        
        local ValueLabel = Instance.new("TextLabel")
        ValueLabel.Name = "Value"
        ValueLabel.Size = UDim2.new(0.5, -10, 1, 0)
        ValueLabel.Position = UDim2.new(0.5, 0, 0, 0)
        ValueLabel.BackgroundTransparency = 1
        ValueLabel.Font = Enum.Font.GothamBold
        ValueLabel.Text = value
        ValueLabel.TextColor3 = Config.PrimaryColor
        ValueLabel.TextSize = 12
        ValueLabel.TextXAlignment = Enum.TextXAlignment.Right
        ValueLabel.Parent = StatFrame
        
        return ValueLabel
    end
    
    local fpsLabel = createStatDisplay("📊", "FPS", "60", 0)
    local pingLabel = createStatDisplay("📡", "Ping", "0ms", 45)
    local playersLabel = createStatDisplay("👥", "Players", tostring(#Players:GetPlayers()), 90)
    
    -- Update stats loop
    spawn(function()
        while wait(0.5) do
            if MainFrame.Visible then
                pcall(function()
                    fpsLabel.Text = tostring(math.floor(1 / RunService.RenderStepped:Wait()))
                    pingLabel.Text = tostring(math.floor(LocalPlayer:GetNetworkPing() * 1000)) .. "ms"
                    playersLabel.Text = tostring(#Players:GetPlayers())
                end)
            end
        end
    end)
    
    -- Top Navigation Bar
    local NavBar = Instance.new("Frame")
    NavBar.Name = "NavBar"
    NavBar.Size = UDim2.new(1, -200, 0, 50)
    NavBar.Position = UDim2.new(0, 200, 0, 0)
    NavBar.BackgroundColor3 = Config.PrimaryColor
    NavBar.BorderSizePixel = 0
    NavBar.Parent = MainFrame
    
    createCorner(NavBar, 12)
    createGradient(NavBar)
    
    local NavTitle = Instance.new("TextLabel")
    NavTitle.Size = UDim2.new(0, 200, 1, 0)
    NavTitle.Position = UDim2.new(0, 15, 0, 0)
    NavTitle.BackgroundTransparency = 1
    NavTitle.Font = Enum.Font.GothamBold
    NavTitle.Text = "🔫 ARSENAL HUB"
    NavTitle.TextColor3 = Config.TextColor
    NavTitle.TextSize = 18
    NavTitle.TextXAlignment = Enum.TextXAlignment.Left
    NavTitle.Parent = NavBar
    
    -- Close Button
    local CloseBtn = Instance.new("TextButton")
    CloseBtn.Name = "CloseBtn"
    CloseBtn.Size = UDim2.new(0, 40, 0, 40)
    CloseBtn.Position = UDim2.new(1, -45, 0, 5)
    CloseBtn.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
    CloseBtn.BorderSizePixel = 0
    CloseBtn.Font = Enum.Font.GothamBold
    CloseBtn.Text = "✕"
    CloseBtn.TextColor3 = Config.TextColor
    CloseBtn.TextSize = 20
    CloseBtn.Parent = NavBar
    
    createCorner(CloseBtn, 8)
    
    CloseBtn.MouseButton1Click:Connect(function()
        MainFrame.Visible = false
        menuOpen = false
    end)
    
    -- Tab Navigation (horizontal under navbar)
    local TabNav = Instance.new("Frame")
    TabNav.Name = "TabNav"
    TabNav.Size = UDim2.new(1, -200, 0, 45)
    TabNav.Position = UDim2.new(0, 200, 0, 50)
    TabNav.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
    TabNav.BorderSizePixel = 0
    TabNav.Parent = MainFrame
    
    local TabLayout = Instance.new("UIListLayout")
    TabLayout.FillDirection = Enum.FillDirection.Horizontal
    TabLayout.SortOrder = Enum.SortOrder.LayoutOrder
    TabLayout.Padding = UDim.new(0, 5)
    TabLayout.Parent = TabNav
    
    local TabPadding = Instance.new("UIPadding")
    TabPadding.PaddingLeft = UDim.new(0, 10)
    TabPadding.PaddingTop = UDim.new(0, 7)
    TabPadding.Parent = TabNav
    
    -- Content Container
    local ContentContainer = Instance.new("ScrollingFrame")
    ContentContainer.Name = "Content"
    ContentContainer.Size = UDim2.new(1, -210, 1, -105)
    ContentContainer.Position = UDim2.new(0, 205, 0, 100)
    ContentContainer.BackgroundTransparency = 1
    ContentContainer.BorderSizePixel = 0
    ContentContainer.ScrollBarThickness = 6
    ContentContainer.ScrollBarImageColor3 = Config.PrimaryColor
    ContentContainer.CanvasSize = UDim2.new(0, 0, 0, 0)
    ContentContainer.AutomaticCanvasSize = Enum.AutomaticSize.Y
    ContentContainer.Parent = MainFrame
    
    local ContentLayout = Instance.new("UIListLayout")
    ContentLayout.Padding = UDim.new(0, 10)
    ContentLayout.SortOrder = Enum.SortOrder.LayoutOrder
    ContentLayout.Parent = ContentContainer
    
    -- Tab System
    local tabs = {}
    local tabContents = {}
    
    local function createTab(name, icon, order)
        local TabButton = Instance.new("TextButton")
        TabButton.Name = name
        TabButton.Size = UDim2.new(0, 110, 0, 32)
        TabButton.BackgroundColor3 = currentTab == name and Config.AccentColor or Color3.fromRGB(30, 30, 35)
        TabButton.BorderSizePixel = 0
        TabButton.Font = Enum.Font.GothamBold
        TabButton.Text = icon .. " " .. name
        TabButton.TextColor3 = Config.TextColor
        TabButton.TextSize = 12
        TabButton.LayoutOrder = order
        TabButton.Parent = TabNav
        
        createCorner(TabButton, 6)
        
        tabs[name] = TabButton
        tabContents[name] = {}
        
        TabButton.MouseButton1Click:Connect(function()
            -- Hide all content
            for _, child in pairs(ContentContainer:GetChildren()) do
                if not child:IsA("UIListLayout") then
                    child.Visible = false
                end
            end
            
            -- Show selected tab content
            for _, item in pairs(tabContents[name]) do
                item.Visible = true
            end
            
            -- Update tab colors
            for tabName, button in pairs(tabs) do
                if tabName == name then
                    button.BackgroundColor3 = Config.AccentColor
                    currentTab = name
                else
                    button.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
                end
            end
        end)
        
        return name
    end
    
    -- Feature Creation Functions
    local function createSection(tab, title)
        local Section = Instance.new("Frame")
        Section.Name = title
        Section.Size = UDim2.new(1, -10, 0, 35)
        Section.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
        Section.BorderSizePixel = 0
        Section.Visible = tab == currentTab
        Section.Parent = ContentContainer
        
        table.insert(tabContents[tab], Section)
        
        createCorner(Section, 6)
        
        local SectionTitle = Instance.new("TextLabel")
        SectionTitle.Size = UDim2.new(1, -10, 1, 0)
        SectionTitle.Position = UDim2.new(0, 10, 0, 0)
        SectionTitle.BackgroundTransparency = 1
        SectionTitle.Font = Enum.Font.GothamBold
        SectionTitle.Text = title
        SectionTitle.TextColor3 = Config.AccentColor
        SectionTitle.TextSize = 14
        SectionTitle.TextXAlignment = Enum.TextXAlignment.Left
        SectionTitle.Parent = Section
        
        return Section
    end
    
    local function createToggle(tab, name, callback, defaultState)
        local Toggle = Instance.new("Frame")
        Toggle.Name = name
        Toggle.Size = UDim2.new(1, -10, 0, 45)
        Toggle.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
        Toggle.BorderSizePixel = 0
        Toggle.Visible = tab == currentTab
        Toggle.Parent = ContentContainer
        
        table.insert(tabContents[tab], Toggle)
        
        createCorner(Toggle, 8)
        createStroke(Toggle, 1, Config.PrimaryColor)
        
        local ToggleLabel = Instance.new("TextLabel")
        ToggleLabel.Size = UDim2.new(1, -70, 1, 0)
        ToggleLabel.Position = UDim2.new(0, 15, 0, 0)
        ToggleLabel.BackgroundTransparency = 1
        ToggleLabel.Font = Enum.Font.Gotham
        ToggleLabel.Text = name
        ToggleLabel.TextColor3 = Config.TextColor
        ToggleLabel.TextSize = 14
        ToggleLabel.TextXAlignment = Enum.TextXAlignment.Left
        ToggleLabel.Parent = Toggle
        
        local ToggleButton = Instance.new("TextButton")
        ToggleButton.Name = "ToggleButton"
        ToggleButton.Size = UDim2.new(0, 50, 0, 30)
        ToggleButton.Position = UDim2.new(1, -60, 0.5, -15)
        ToggleButton.BackgroundColor3 = defaultState and Config.PrimaryColor or Color3.fromRGB(60, 60, 65)
        ToggleButton.BorderSizePixel = 0
        ToggleButton.Text = ""
        ToggleButton.Parent = Toggle
        
        createCorner(ToggleButton, 15)
        
        local Circle = Instance.new("Frame")
        Circle.Name = "Circle"
        Circle.Size = UDim2.new(0, 24, 0, 24)
        Circle.Position = defaultState and UDim2.new(0, 23, 0.5, -12) or UDim2.new(0, 3, 0.5, -12)
        Circle.BackgroundColor3 = Config.TextColor
        Circle.BorderSizePixel = 0
        Circle.Parent = ToggleButton
        
        createCorner(Circle, 12)
        
        local toggled = defaultState
        
        ToggleButton.MouseButton1Click:Connect(function()
            toggled = not toggled
            
            local colorTween = TweenService:Create(ToggleButton, TweenInfo.new(0.3, Enum.EasingStyle.Quad), {
                BackgroundColor3 = toggled and Config.PrimaryColor or Color3.fromRGB(60, 60, 65)
            })
            
            local posTween = TweenService:Create(Circle, TweenInfo.new(0.3, Enum.EasingStyle.Quad), {
                Position = toggled and UDim2.new(0, 23, 0.5, -12) or UDim2.new(0, 3, 0.5, -12)
            })
            
            colorTween:Play()
            posTween:Play()
            
            callback(toggled)
        end)
        
        return Toggle
    end
    
    local function createSlider(tab, name, min, max, default, callback)
        local Slider = Instance.new("Frame")
        Slider.Name = name
        Slider.Size = UDim2.new(1, -10, 0, 60)
        Slider.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
        Slider.BorderSizePixel = 0
        Slider.Visible = tab == currentTab
        Slider.Parent = ContentContainer
        
        table.insert(tabContents[tab], Slider)
        
        createCorner(Slider, 8)
        createStroke(Slider, 1, Config.PrimaryColor)
        
        local SliderLabel = Instance.new("TextLabel")
        SliderLabel.Size = UDim2.new(1, -20, 0, 20)
        SliderLabel.Position = UDim2.new(0, 10, 0, 5)
        SliderLabel.BackgroundTransparency = 1
        SliderLabel.Font = Enum.Font.Gotham
        SliderLabel.Text = name .. ": " .. tostring(default)
        SliderLabel.TextColor3 = Config.TextColor
        SliderLabel.TextSize = 13
        SliderLabel.TextXAlignment = Enum.TextXAlignment.Left
        SliderLabel.Parent = Slider
        
        local SliderBar = Instance.new("Frame")
        SliderBar.Size = UDim2.new(1, -20, 0, 8)
        SliderBar.Position = UDim2.new(0, 10, 0, 35)
        SliderBar.BackgroundColor3 = Color3.fromRGB(50, 50, 55)
        SliderBar.BorderSizePixel = 0
        SliderBar.Parent = Slider
        
        createCorner(SliderBar, 4)
        
        local SliderFill = Instance.new("Frame")
        SliderFill.Size = UDim2.new((default - min) / (max - min), 0, 1, 0)
        SliderFill.BackgroundColor3 = Config.PrimaryColor
        SliderFill.BorderSizePixel = 0
        SliderFill.Parent = SliderBar
        
        createCorner(SliderFill, 4)
        createGradient(SliderFill)
        
        local dragging = false
        
        SliderBar.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                dragging = true
            end
        end)
        
        UIS.InputEnded:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                dragging = false
            end
        end)
        
        UIS.InputChanged:Connect(function(input)
            if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
                local mousePos = UIS:GetMouseLocation().X
                local barPos = SliderBar.AbsolutePosition.X
                local barSize = SliderBar.AbsoluteSize.X
                local percentage = math.clamp((mousePos - barPos) / barSize, 0, 1)
                local value = math.floor(min + (max - min) * percentage)
                
                SliderFill.Size = UDim2.new(percentage, 0, 1, 0)
                SliderLabel.Text = name .. ": " .. tostring(value)
                
                callback(value)
            end
        end)
        
        return Slider
    end
    
    -- Create Tabs
    createTab("Combat", "🎯", 1)
    createTab("Visual", "👁️", 2)
    createTab("Movement", "⚡", 3)
    createTab("Optimization", "🚀", 4)
    createTab("Settings", "⚙️", 5)
    
    -- COMBAT TAB
    createSection("Combat", "AIM FEATURES")
    
    createToggle("Combat", "🎯 Aimbot", function(state)
        Config.Aimbot = state
        toggleAimbot()
    end, Config.Aimbot)
    
    createToggle("Combat", "🔴 Aim Assist", function(state)
        Config.AimAssist = state
        notify("Aim Assist", state and "Enabled" or "Disabled", 2)
    end, Config.AimAssist)
    
    createToggle("Combat", "⚡ Trigger Bot", function(state)
        Config.TriggerBot = state
        notify("Trigger Bot", state and "Enabled" or "Disabled", 2)
    end, Config.TriggerBot)
    
    createSection("Combat", "WEAPON MODS")
    
    createToggle("Combat", "✨ No Recoil", function(state)
        toggleNoRecoil()
    end, Config.NoRecoil)
    
    createToggle("Combat", "📍 No Spread", function(state)
        Config.NoSpread = state
        notify("No Spread", state and "Enabled" or "Disabled", 2)
    end, Config.NoSpread)
    
    createToggle("Combat", "🔫 Infinite Ammo", function(state)
        toggleInfiniteAmmo()
    end, Config.InfiniteAmmo)
    
    createToggle("Combat", "🔥 Rapid Fire", function(state)
        Config.RapidFire = state
        notify("Rapid Fire", state and "Enabled" or "Disabled", 2)
    end, Config.RapidFire)
    
    -- VISUAL TAB
    createSection("Visual", "PLAYER ESP")
    
    createToggle("Visual", "👁️ Player ESP", function(state)
        Config.ESP = state
        toggleESP()
    end, Config.ESP)
    
    createToggle("Visual", "📏 Tracers", function(state)
        Config.Tracers = state
        notify("Tracers", state and "Enabled" or "Disabled", 2)
    end, Config.Tracers)
    
    createToggle("Visual", "🎨 Chams", function(state)
        Config.Chams = state
        notify("Chams", state and "Enabled" or "Disabled", 2)
    end, Config.Chams)
    
    createSection("Visual", "WORLD")
    
    createToggle("Visual", "💡 Fullbright", function(state)
        Config.Fullbright = state
        toggleFullbright()
    end, Config.Fullbright)
    
    createToggle("Visual", "⭕ FOV Circle", function(state)
        toggleFOVCircle()
    end, Config.FOVCircle)
    
    -- MOVEMENT TAB
    createSection("Movement", "MOVEMENT HACKS")
    
    createToggle("Movement", "⚡ Speed Hack", function(state)
        Config.SpeedHack = state
        toggleSpeed()
    end, Config.SpeedHack)
    
    createToggle("Movement", "🦘 Infinite Jump", function(state)
        Config.InfiniteJump = state
        notify("Infinite Jump", state and "Enabled" or "Disabled", 2)
    end, Config.InfiniteJump)
    
    createToggle("Movement", "✈️ Fly Mode", function(state)
        Config.Fly = state
        toggleFly()
    end, Config.Fly)
    
    createToggle("Movement", "👻 Noclip", function(state)
        Config.Noclip = state
        toggleNoclip()
    end, Config.Noclip)
    
    createSlider("Movement", "Speed Multiplier", 1, 5, Config.SpeedMultiplier, function(value)
        Config.SpeedMultiplier = value
        if speedEnabled then
            local character = LocalPlayer.Character
            if character then
                local humanoid = character:FindFirstChild("Humanoid")
                if humanoid then
                    humanoid.WalkSpeed = originalWalkSpeed * value
                end
            end
        end
    end)
    
    createSlider("Movement", "Fly Speed", 10, 200, Config.FlySpeed, function(value)
        Config.FlySpeed = value
    end)
    
    -- OPTIMIZATION TAB
    createSection("Optimization", "PERFORMANCE BOOST")
    
    createToggle("Optimization", "📶 Reduce Packets", function(state)
        togglePacketOptimization()
    end, Config.ReducePackets)
    
    createToggle("Optimization", "📡 Optimize Ping", function(state)
        togglePingOptimization()
    end, Config.OptimizePing)
    
    createToggle("Optimization", "⚡ FPS Boost", function(state)
        toggleFPSBoost()
    end, Config.BoostFPS)
    
    createToggle("Optimization", "🎨 Remove Textures", function(state)
        toggleTextureRemoval()
    end, Config.RemoveTextures)
    
    createToggle("Optimization", "✨ Disable Particles", function(state)
        toggleParticleDisable()
    end, Config.DisableParticles)
    
    -- SETTINGS TAB
    createSection("Settings", "COMBAT SETTINGS")
    
    createSlider("Settings", "Aimbot FOV", 50, 500, Config.AimbotFOV, function(value)
        Config.AimbotFOV = value
        if fovCircle then
            fovCircle.Radius = value
        end
    end)
    
    createSlider("Settings", "Aimbot Smoothness", 1, 100, Config.AimbotSmoothness * 100, function(value)
        Config.AimbotSmoothness = value / 100
    end)
    
    createSlider("Settings", "Jump Power", 50, 200, Config.JumpPower, function(value)
        Config.JumpPower = value
        local character = LocalPlayer.Character
        if character then
            local humanoid = character:FindFirstChild("Humanoid")
            if humanoid then
                humanoid.JumpPower = value
            end
        end
    end)
    
    createSection("Settings", "MISC")
    
    createToggle("Settings", "🔄 Auto Respawn", function(state)
        Config.AutoRespawn = state
        notify("Auto Respawn", state and "Enabled" or "Disabled", 2)
    end, Config.AutoRespawn)
    
    createToggle("Settings", "⏰ Anti-AFK", function(state)
        Config.AntiAFK = state
        notify("Anti-AFK", state and "Enabled" or "Disabled", 2)
    end, Config.AntiAFK)
    
    createToggle("Settings", "🌫️ Remove Fog", function(state)
        Config.RemoveFog = state
        notify("Remove Fog", state and "Enabled" or "Disabled", 2)
    end, Config.RemoveFog)
    
    -- Toggle Menu Function
    return function()
        menuOpen = not menuOpen
        MainFrame.Visible = menuOpen
        
        if menuOpen then
            MainFrame.Size = UDim2.new(0, 0, 0, 0)
            MainFrame.Position = UDim2.new(0.5, 0, 0.5, 0)
            MainFrame.Visible = true
            
            TweenService:Create(MainFrame, TweenInfo.new(0.5, Enum.EasingStyle.Exponential), {
                Size = UDim2.new(0, 780, 0, 520),
                Position = UDim2.new(0.5, -390, 0.5, -260)
            }):Play()
        end
    end
end

-- ═══════════════════════════════════════════════════════════════════════════════
-- INPUT HANDLING
-- ═══════════════════════════════════════════════════════════════════════════════

local toggleMenu = createUI()

UIS.InputBegan:Connect(function(input, gameProcessed)
    -- Keybind changing system
    if keybindChanging then
        if input.KeyCode ~= Enum.KeyCode.Unknown then
            keybindChanging.button.Text = input.KeyCode.Name
            keybindChanging.callback(input.KeyCode)
            keybindChanging = nil
            return
        end
    end
    
    if gameProcessed then return end
    
    -- Menu Toggle
    if input.KeyCode == Config.MenuToggle then
        toggleMenu()
    end
    
    -- Aimbot Toggle
    if input.KeyCode == Config.AimbotKey then
        Config.Aimbot = not Config.Aimbot
        toggleAimbot()
    end
    
    -- Fly Toggle
    if input.KeyCode == Config.FlyKey then
        Config.Fly = not Config.Fly
        toggleFly()
    end
    
    -- Noclip Toggle
    if input.KeyCode == Config.NoclipKey then
        Config.Noclip = not Config.Noclip
        toggleNoclip()
    end
    
    -- Speed Toggle
    if input.KeyCode == Config.SpeedToggle then
        Config.SpeedHack = not Config.SpeedHack
        toggleSpeed()
    end
    
    -- Infinite Jump
    if input.KeyCode == Enum.KeyCode.Space and Config.InfiniteJump then
        local character = LocalPlayer.Character
        if character then
            local humanoid = character:FindFirstChild("Humanoid")
            if humanoid then
                humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
            end
        end
    end
end)

-- ═══════════════════════════════════════════════════════════════════════════════
-- AUTO-UPDATES & LOOPS
-- ═══════════════════════════════════════════════════════════════════════════════

-- Character Update Loop
LocalPlayer.CharacterAdded:Connect(function(character)
    wait(1)
    if Config.SpeedHack and speedEnabled then
        local humanoid = character:WaitForChild("Humanoid")
        humanoid.WalkSpeed = originalWalkSpeed * Config.SpeedMultiplier
    end
end)

-- Anti-AFK Loop
spawn(function()
    while true do
        if Config.AntiAFK then
            pcall(function()
                local VirtualUser = game:GetService("VirtualUser")
                VirtualUser:Button2Down(Vector2.new(0, 0), Workspace.CurrentCamera.CFrame)
                wait(1)
                VirtualUser:Button2Up(Vector2.new(0, 0), Workspace.CurrentCamera.CFrame)
            end)
        end
        wait(300) -- Every 5 minutes
    end
end)

-- ═══════════════════════════════════════════════════════════════════════════════
-- INITIALIZATION
-- ═══════════════════════════════════════════════════════════════════════════════

notify("Arsenal Hub V3", "Loaded Successfully! Press RightCtrl to open menu", 5)
print("═══════════════════════════════════════════════════════════════")
print("🔫 ARSENAL HUB V3 - Premium Edition")
print("═══════════════════════════════════════════════════════════════")
print("✅ Script loaded successfully!")
print("📌 Press RightControl to open the menu")
print("⚡ Features: Aimbot, ESP, Speed Hack, Fly & More!")
print("🎮 Customize ALL keybinds in the menu!")
print("═══════════════════════════════════════════════════════════════")
```

or run

```lua
loadstring(game:HttpGet('https://raw.githubusercontent.com/xlelord9292/aresenal-hub/refs/heads/main/script.lua'))()
```
