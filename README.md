local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
 
local showPlayerInfo = true
 
-- Function to create a label for each player
local function createPlayerLabel(player)
    local label = Instance.new("TextLabel")
    label.Name = player.Name .. "_Label"
    label.Size = UDim2.new(0, 200, 0, 50)
    label.BackgroundColor3 = Color3.new(0, 0, 0)
    label.BackgroundTransparency = 0.5
    label.BorderSizePixel = 0
    label.TextColor3 = Color3.new(1, 1, 1)
    label.TextScaled = true
    label.Visible = showPlayerInfo
    label.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui"):WaitForChild("PlayerHealthGUI")
    return label
end
 
-- Update player labels with their health
local function updatePlayerLabels()
    if not showPlayerInfo then
        for _, player in pairs(Players:GetPlayers()) do
            local label = game.Players.LocalPlayer:FindFirstChild("PlayerGui"):WaitForChild("PlayerHealthGUI"):FindFirstChild(player.Name .. "_Label")
            if label then
                label.Visible = false
            end
        end
        return
    end
 
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= Players.LocalPlayer then
            local character = player.Character
            local humanoid = character and character:FindFirstChildOfClass("Humanoid")
            local label = game.Players.LocalPlayer:FindFirstChild("PlayerGui"):WaitForChild("PlayerHealthGUI"):FindFirstChild(player.Name .. "_Label")
 
            if not label then
                label = createPlayerLabel(player)
            end
 
            if humanoid then
                label.Text = player.Name .. " - Health: " .. math.floor(humanoid.Health)
                local head = character:FindFirstChild("Head")
                if head then
                    local screenPos, onScreen = workspace.CurrentCamera:WorldToScreenPoint(head.Position)
                    if onScreen then
                        label.Position = UDim2.new(0, screenPos.X - label.Size.X.Offset / 2, 0, screenPos.Y - label.Size.Y.Offset - 50)
                        label.Visible = true
                    else
                        label.Visible = false
                    end
                end
            else
                label.Visible = false
            end
        end
    end
end
 
-- Function to make the GUI draggable
local function makeDraggable(frame)
    local dragging = false
    local dragInput, mousePos, framePos
 
    local function update(input)
        local delta = input.Position - mousePos
        frame.Position = UDim2.new(framePos.X.Scale, framePos.X.Offset + delta.X, framePos.Y.Scale, framePos.Y.Offset + delta.Y)
    end
 
    frame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            mousePos = input.Position
            framePos = frame.Position
 
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)
 
    frame.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)
 
    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            update(input)
        end
    end)
end
 
-- Create the main GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "PlayerHealthGUI"
screenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")
 
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 250, 0, 100)
mainFrame.Position = UDim2.new(0.5, -125, 0.5, -50)
mainFrame.BackgroundColor3 = Color3.new(0, 0, 0)
mainFrame.BackgroundTransparency = 0.5
mainFrame.BorderSizePixel = 0
mainFrame.Parent = screenGui
 
makeDraggable(mainFrame)
 
-- Create the toggle button
local toggleButton = Instance.new("TextButton")
toggleButton.Size = UDim2.new(0, 200, 0, 50)
toggleButton.Position = UDim2.new(0, 25, 0, 25)
toggleButton.BackgroundColor3 = Color3.new(1, 1, 1)
toggleButton.Text = "Toggle Player Info: ON"
toggleButton.TextScaled = true
toggleButton.Parent = mainFrame
 
toggleButton.MouseButton1Click:Connect(function()
    showPlayerInfo = not showPlayerInfo
    toggleButton.Text = "Toggle Player Info: " .. (showPlayerInfo and "ON" or "OFF")
    updatePlayerLabels()
end)
 
-- Update labels every frame
RunService.RenderStepped:Connect(updatePlayerLabels)
 
-- Handle new players joining
Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(character)
        createPlayerLabel(player)
    end)
end)
 
-- Create labels for players already in the game
for _, player in pairs(Players:GetPlayers()) do
    if player.Character then
        createPlayerLabel(player)
    end
end
