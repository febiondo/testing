--!strict
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local PlayerGui = Players.LocalPlayer:WaitForChild("PlayerGui")

-- Remote Event
local DataLossRemote = ReplicatedStorage:WaitForChild("DataLossRemote")

-- Create GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "DataLossTestGui"
screenGui.Parent = PlayerGui

-- Main frame
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 350, 0, 400)
mainFrame.Position = UDim2.new(0.5, -175, 0.5, -200)
mainFrame.AnchorPoint = Vector2.new(0.5, 0.5)
mainFrame.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
mainFrame.BorderSizePixel = 0
mainFrame.Parent = screenGui

-- Title
local title = Instance.new("TextLabel")
title.Name = "Title"
title.Text = "DATA LOSS TEST INTERFACE"
title.Size = UDim2.new(1, 0, 0, 40)
title.Position = UDim2.new(0, 0, 0, 0)
title.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Font = Enum.Font.SourceSansBold
title.TextSize = 18
title.Parent = mainFrame

-- Test data input
local dataInputLabel = Instance.new("TextLabel")
dataInputLabel.Name = "DataInputLabel"
dataInputLabel.Text = "Test Data:"
dataInputLabel.Size = UDim2.new(1, -20, 0, 20)
dataInputLabel.Position = UDim2.new(0, 10, 0, 50)
dataInputLabel.BackgroundTransparency = 1
dataInputLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
dataInputLabel.TextXAlignment = Enum.TextXAlignment.Left
dataInputLabel.Font = Enum.Font.SourceSans
dataInputLabel.TextSize = 16
dataInputLabel.Parent = mainFrame

local dataInput = Instance.new("TextBox")
dataInput.Name = "DataInput"
dataInput.Size = UDim2.new(1, -20, 0, 100)
dataInput.Position = UDim2.new(0, 10, 0, 75)
dataInput.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
dataInput.TextColor3 = Color3.fromRGB(255, 255, 255)
dataInput.Text = '{"test": "data", "value": 123}'
dataInput.MultiLine = true
dataInput.TextWrapped = true
dataInput.ClearTextOnFocus = false
dataInput.Font = Enum.Font.SourceSans
dataInput.TextSize = 14
dataInput.Parent = mainFrame

-- Buttons
local saveButton = Instance.new("TextButton")
saveButton.Name = "SaveButton"
saveButton.Text = "SAVE DATA"
saveButton.Size = UDim2.new(1, -20, 0, 30)
saveButton.Position = UDim2.new(0, 10, 0, 190)
saveButton.BackgroundColor3 = Color3.fromRGB(0, 120, 215)
saveButton.TextColor3 = Color3.fromRGB(255, 255, 255)
saveButton.Font = Enum.Font.SourceSansBold
saveButton.TextSize = 16
saveButton.Parent = mainFrame

local loadButton = Instance.new("TextButton")
loadButton.Name = "LoadButton"
loadButton.Text = "LOAD DATA"
loadButton.Size = UDim2.new(1, -20, 0, 30)
loadButton.Position = UDim2.new(0, 10, 0, 230)
loadButton.BackgroundColor3 = Color3.fromRGB(0, 120, 215)
loadButton.TextColor3 = Color3.fromRGB(255, 255, 255)
loadButton.Font = Enum.Font.SourceSansBold
loadButton.TextSize = 16
loadButton.Parent = mainFrame

local wipeButton = Instance.new("TextButton")
wipeButton.Name = "WipeButton"
wipeButton.Text = "WIPE DATA"
wipeButton.Size = UDim2.new(1, -20, 0, 30)
wipeButton.Position = UDim2.new(0, 10, 0, 270)
wipeButton.BackgroundColor3 = Color3.fromRGB(215, 0, 0)
wipeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
wipeButton.Font = Enum.Font.SourceSansBold
wipeButton.TextSize = 16
wipeButton.Parent = mainFrame

-- Status label
local statusLabel = Instance.new("TextLabel")
statusLabel.Name = "StatusLabel"
statusLabel.Text = "Ready"
statusLabel.Size = UDim2.new(1, -20, 0, 60)
statusLabel.Position = UDim2.new(0, 10, 0, 320)
statusLabel.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
statusLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
statusLabel.TextWrapped = true
statusLabel.Font = Enum.Font.SourceSans
statusLabel.TextSize = 14
statusLabel.Parent = mainFrame

-- Close button
local closeButton = Instance.new("TextButton")
closeButton.Name = "CloseButton"
closeButton.Text = "X"
closeButton.Size = UDim2.new(0, 30, 0, 30)
closeButton.Position = UDim2.new(1, -30, 0, 0)
closeButton.BackgroundColor3 = Color3.fromRGB(215, 0, 0)
closeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
closeButton.Font = Enum.Font.SourceSansBold
closeButton.TextSize = 16
closeButton.Parent = mainFrame

-- Button functions
local function updateStatus(message: string, color: Color3)
    statusLabel.Text = message
    statusLabel.TextColor3 = color
end

saveButton.MouseButton1Click:Connect(function()
    local dataText = dataInput.Text
    local success, data = pcall(function()
        return game:GetService("HttpService"):JSONDecode(dataText)
    end)
    
    if not success then
        updateStatus("Invalid JSON data!", Color3.fromRGB(255, 0, 0))
        return
    end
    
    updateStatus("Saving data...", Color3.fromRGB(255, 255, 0))
    DataLossRemote:FireServer("SaveTestData", data)
end)

loadButton.MouseButton1Click:Connect(function()
    updateStatus("Loading data...", Color3.fromRGB(255, 255, 0))
    DataLossRemote:FireServer("LoadTestData")
end)

wipeButton.MouseButton1Click:Connect(function()
    updateStatus("Wiping data...", Color3.fromRGB(255, 0, 0))
    DataLossRemote:FireServer("WipeData")
end)

closeButton.MouseButton1Click:Connect(function()
    screenGui:Destroy()
end)

-- Handle server responses
DataLossRemote.OnClientEvent:Connect(function(action: string, result: any)
    if action == "SaveResult" then
        if result then
            updateStatus("Data saved successfully!", Color3.fromRGB(0, 255, 0))
        else
            updateStatus("Failed to save data!", Color3.fromRGB(255, 0, 0))
        end
    elseif action == "LoadResult" then
        if result then
            dataInput.Text = game:GetService("HttpService"):JSONEncode(result)
            updateStatus("Data loaded successfully!", Color3.fromRGB(0, 255, 0))
        else
            updateStatus("No data found or failed to load!", Color3.fromRGB(255, 0, 0))
        end
    elseif action == "WipeResult" then
        if result then
            dataInput.Text = "{}"
            updateStatus("Data wiped successfully!", Color3.fromRGB(0, 255, 0))
        else
            updateStatus("Failed to wipe data!", Color3.fromRGB(255, 0, 0))
        end
    end
end)

print("Data Loss Test Client Script Initialized")
