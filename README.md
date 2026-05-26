-- Roblox FPS Aimbot Script
-- Features: Aimbot, ESP, FOV customization

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()
local Camera = workspace.CurrentCamera
local TweenService = game:GetService("TweenService")

-- Create UI Panel
local panel = Instance.new("ScreenGui")
panel.Name = "FPS_Hack_Panel"
panel.Parent = LocalPlayer:WaitForChild("PlayerGui")

-- Create Frame for options
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 300, 0, 250)
frame.Position = UDim2.new(0, 50, 0, 50)
frame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
frame.BorderSizePixel = 0
frame.Parent = panel

-- Add title
local title = Instance.new("TextLabel")
title.Text = "FPS Aimbot Control Panel"
title.Size = UDim2.new(0, 300, 0, 30)
title.BackgroundTransparency = 1
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Font = Enum.Font.SourceSansBold
title.TextSize = 18
title.Parent = frame

-- Create buttons
local buttons = {}
local options = {
    {name="Aimbot", enabled=false},
    {name="ESP", enabled=false},
    {name="FOV Circle", enabled=false}
}

for i, option in ipairs(options) do
    local button = Instance.new("TextButton")
    button.Text = option.name
    button.Size = UDim2.new(0, 280, 0, 30)
    button.Position = UDim2.new(0, 10, 0, 40 + (i-1)*40)
    button.BackgroundColor3 = option.enabled and Color3.fromRGB(60, 180, 60) or Color3.fromRGB(80, 80, 80)
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.Font = Enum.Font.SourceSansBold
    button.TextSize = 14
    button.Parent = frame
    
    -- Add click event
    button.MouseButton1Click:Connect(function()
        option.enabled = not option.enabled
        button.BackgroundColor3 = option.enabled and Color3.fromRGB(60, 180, 60) or Color3.fromRGB(80, 80, 80)
        
        if option.name == "FOV Circle" then
            updateFOV(option.enabled)
        end
    end)
    
    table.insert(buttons, button)
end

-- Add FOV slider
local fovSlider = Instance.new("Slider")
fovSlider.Name = "FOV_Slider"
fovSlider.Size = UDim2.new(0, 280, 0, 20)
fovSlider.Position = UDim2.new(0, 10, 0, 160)
fovSlider.Value = 50
fovSlider.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
fovSlider.BorderColor3 = Color3.fromRGB(100, 100, 100)
fovSlider.Parent = frame

-- Add FOV value label
local fovLabel = Instance.new("TextLabel")
fovLabel.Text = "FOV: 50%"
fovLabel.Size = UDim2.new(0, 100, 0, 20)
fovLabel.Position = UDim2.new(0, 10, 0, 180)
fovLabel.BackgroundTransparency = 1
fovLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
fovLabel.Font = Enum.Font.SourceSansBold
fovLabel.TextSize = 14
fovLabel.Parent = frame

-- Update FOV value when slider changes
fovSlider.ValueChanged:Connect(function(value)
    fovLabel.Text = string.format("FOV: %d%%", math.floor(value))
end)

-- Add close button
local closeButton = Instance.new("TextButton")
closeButton.Text = "Close"
closeButton.Size = UDim2.new(0, 60, 0, 30)
closeButton.Position = UDim2.new(0, 230, 0, 200)
closeButton.BackgroundColor3 = Color3.fromRGB(200, 60, 60)
closeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
closeButton.Font = Enum.Font.SourceSansBold
closeButton.TextSize = 14
closeButton.Parent = frame

-- Close panel when button is clicked
closeButton.MouseButton1Click:Connect(function()
    panel:Destroy()
end)

-- Initialize variables
local players = {}
local espEnabled = false
local aimbotEnabled = false
local fovEnabled = false
local currentTarget = nil
local fovCircle = nil

-- Create FOV circle when enabled
function updateFOV(enabled)
    if fovEnabled == enabled then return end
    
    fovEnabled = enabled
    
    if fovEnabled and fovCircle == nil then
        fovCircle = Instance.new("ImageLabel")
        fovCircle.Name = "FOVCircle"
        fovCircle.Size = UDim2.new(0, 200, 0, 200)
        fovCircle.Position = UDim2.new(0.5, -100, 0.5, -100)
        fovCircle.AnchorPoint = Vector2.new(0.5, 0.5)
        fovCircle.Image = "rbxasset://textures/FOVCircle.png"
        fovCircle.ImageTransparency = 0.7
        fovCircle.Visible = true
        fovCircle.BackgroundTransparency = 1
        fovCircle.Parent = frame
        
        -- Update circle size based on slider value
        fovCircle.Changed:Connect(function(property)
            if property == "AbsoluteSize" then
                fovCircle.Size = UDim2.new(0, fovSlider.Value * 2, 0, fovSlider.Value * 2)
            end
        end)
    elseif not fovEnabled and fovCircle ~= nil then
        fovCircle:Destroy()
        fovCircle = nil
    end
end

-- Create ESP boxes for players
function createESP(player)
    local character = player.Character
    if character == nil then return end
    
    local humanoid = character:FindFirstChild("Humanoid")
    if humanoid == nil then return end
    
    local head = character:FindFirstChild("Head")
    if head == nil then return end
    
    local box = Instance.new("BoxHandleAdornment")
    box.Name = "ESP_Box"
    box.Size = Vector3.new(2, 4, 2)
    box.CFrame = CFrame.new(head.Position)
    box.Color3 = Color3.fromRGB(255, 0, 0)
    box.AlwaysOnTop = true
    box.Enabled = true
    box.Visible = true
    box.Adornee = head
    box.Parent = frame
    
    return box
end

-- Update ESP boxes for all players
function updateESP()
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            local espBox = player:FindFirstChild("ESP_Box")
            if espEnabled and espBox == nil then
                espBox = createESP(player)
            elseif not espEnabled and espBox ~= nil then
                espBox:Destroy()
            end
        end
    end
end

-- Find closest player to aim at
function findClosestPlayer()
    local closest = nil
    local minDistance = math.huge
    
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            local character = player.Character
            if character ~= nil then
                local head = character:FindFirstChild("Head")
                if head ~= nil then
                    local distance = (head.Position - Camera.CFrame.Position).magnitude
                    if distance < minDistance then
                        minDistance = distance
                        closest = player
                    end
                end
            end
        end
    end
    
    return closest
end

-- Aimbot logic
function updateAimbot()
    if not aimbotEnabled then return end
    
    local target = findClosestPlayer()
    if target == nil then return end
    
    local character = target.Character
    if character == nil then return end
    
    local head = character:FindFirstChild("Head")
    if head == nil then return end
    
    -- Calculate direction to head
    local direction = (head.Position - Camera.CFrame.Position).unit
    local angle = math.acos(direction:Dot(Camera.CFrame.LookVector))
    
    -- Only aim if target is within FOV
    if angle < fovSlider.Value / 100 * math.pi / 2 then
        currentTarget = target
        Mouse.TargetFilter = target
        Mouse.Hit = CFrame.new(head.Position)
    else
        currentTarget = nil
        Mouse.TargetFilter = nil
    end
end

-- Connect events
UserInputService.InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.F then
        -- Toggle aimbot
        aimbotEnabled = not aimbotEnabled
        buttons[1].BackgroundColor3 = aimbotEnabled and Color3.fromRGB(60, 180, 60) or Color3.fromRGB(80, 80, 80)
    elseif input.KeyCode == Enum.KeyCode.G then
        -- Toggle ESP
        espEnabled = not espEnabled
        buttons[2].BackgroundColor3 = espEnabled and Color3.fromRGB(60, 180, 60) or Color3.fromRGB(80, 80, 80)
        updateESP()
    end
end)

-- Run service loop
RunService.RenderStepped:Connect(function()
    updateESP()
    updateAimbot()
end)
