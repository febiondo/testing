local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local HttpService = game:GetService("HttpService")
local Player = Players.LocalPlayer
local PlayerGui = Player:WaitForChild("PlayerGui")

-- RemoteEvent
local DataLossRemote = ReplicatedStorage:WaitForChild("DataLossRemote")

-- Criar GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "DataLossTestGui"
screenGui.ResetOnSpawn = false
screenGui.IgnoreGuiInset = true
screenGui.Parent = PlayerGui

-- Main frame
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.fromOffset(380, 420)
mainFrame.Position = UDim2.fromScale(0.5, 0.5)
mainFrame.AnchorPoint = Vector2.new(0.5, 0.5)
mainFrame.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
mainFrame.BorderSizePixel = 0
mainFrame.Parent = screenGui

local uiCorner = Instance.new("UICorner")
uiCorner.CornerRadius = UDim.new(0, 10)
uiCorner.Parent = mainFrame

local padding = Instance.new("UIPadding")
padding.PaddingLeft = UDim.new(0, 10)
padding.PaddingRight = UDim.new(0, 10)
padding.Parent = mainFrame

-- Title
local title = Instance.new("TextLabel")
title.Name = "Title"
title.Text = "DATA LOSS TEST INTERFACE"
title.Size = UDim2.new(1, -40, 0, 40)
title.Position = UDim2.new(0, 10, 0, 10)
title.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Font = Enum.Font.SourceSansBold
title.TextSize = 18
title.Parent = mainFrame

local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 6)
titleCorner.Parent = title

-- Close button
local closeButton = Instance.new("TextButton")
closeButton.Name = "CloseButton"
closeButton.Text = "X"
closeButton.Size = UDim2.fromOffset(30, 30)
closeButton.Position = UDim2.new(1, -40, 0, 10)
closeButton.BackgroundColor3 = Color3.fromRGB(215, 0, 0)
closeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
closeButton.Font = Enum.Font.SourceSansBold
closeButton.TextSize = 16
closeButton.Parent = mainFrame

local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(0, 6)
closeCorner.Parent = closeButton

-- Label de entrada
local dataInputLabel = Instance.new("TextLabel")
dataInputLabel.Name = "DataInputLabel"
dataInputLabel.Text = "Test Data (JSON):"
dataInputLabel.Size = UDim2.new(1, -20, 0, 20)
dataInputLabel.Position = UDim2.new(0, 10, 0, 60)
dataInputLabel.BackgroundTransparency = 1
dataInputLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
dataInputLabel.TextXAlignment = Enum.TextXAlignment.Left
dataInputLabel.Font = Enum.Font.SourceSans
dataInputLabel.TextSize = 16
dataInputLabel.Parent = mainFrame

-- Caixa de texto
local dataInput = Instance.new("TextBox")
dataInput.Name = "DataInput"
dataInput.Size = UDim2.new(1, -20, 0, 120)
dataInput.Position = UDim2.new(0, 10, 0, 85)
dataInput.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
dataInput.TextColor3 = Color3.fromRGB(255, 255, 255)
dataInput.Text = '{"test":"data","value":123}'
dataInput.MultiLine = true
dataInput.TextWrapped = true
dataInput.ClearTextOnFocus = false
dataInput.Font = Enum.Font.SourceSans
dataInput.TextSize = 14
dataInput.PlaceholderText = '{"key":"value"}'
dataInput.Parent = mainFrame

local inputCorner = Instance.new("UICorner")
inputCorner.CornerRadius = UDim.new(0, 6)
inputCorner.Parent = dataInput

-- Botões
local function makeButton(name, text, y, bg)
	local b = Instance.new("TextButton")
	b.Name = name
	b.Text = text
	b.Size = UDim2.new(1, -20, 0, 34)
	b.Position = UDim2.new(0, 10, 0, y)
	b.BackgroundColor3 = bg
	b.TextColor3 = Color3.fromRGB(255, 255, 255)
	b.Font = Enum.Font.SourceSansBold
	b.TextSize = 16
	b.AutoButtonColor = true
	b.Parent = mainFrame
	local c = Instance.new("UICorner")
	c.CornerRadius = UDim.new(0, 6)
	c.Parent = b
	return b
end

local saveButton = makeButton("SaveButton", "SAVE DATA", 215, Color3.fromRGB(0, 120, 215))
local loadButton = makeButton("LoadButton", "LOAD DATA", 255, Color3.fromRGB(0, 120, 215))
local wipeButton = makeButton("WipeButton", "WIPE DATA", 295, Color3.fromRGB(215, 0, 0))

-- Status
local statusLabel = Instance.new("TextLabel")
statusLabel.Name = "StatusLabel"
statusLabel.Text = "Ready"
statusLabel.Size = UDim2.new(1, -20, 0, 70)
statusLabel.Position = UDim2.new(0, 10, 0, 340)
statusLabel.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
statusLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
statusLabel.TextWrapped = true
statusLabel.Font = Enum.Font.SourceSans
statusLabel.TextSize = 14
statusLabel.Parent = mainFrame

local statusCorner = Instance.new("UICorner")
statusCorner.CornerRadius = UDim.new(0, 6)
statusCorner.Parent = statusLabel

-- Funções
local function updateStatus(message: string, color: Color3)
	statusLabel.Text = message
	statusLabel.TextColor3 = color
end

local function parseJSON(text: string)
	local ok, data = pcall(function()
		return HttpService:JSONDecode(text)
	end)
	return ok, data
end

-- Eventos
saveButton.MouseButton1Click:Connect(function()
	local jsonText = dataInput.Text
	local ok, data = parseJSON(jsonText)

	if not ok then
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

-- Respostas do servidor
DataLossRemote.OnClientEvent:Connect(function(action: string, result: any)
	if action == "SaveResult" then
		if result then
			updateStatus("Data saved successfully!", Color3.fromRGB(0, 255, 0))
		else
			updateStatus("Failed to save data!", Color3.fromRGB(255, 0, 0))
		end
	elseif action == "LoadResult" then
		if result ~= nil then
			local ok, json = pcall(function()
				return HttpService:JSONEncode(result)
			end)
			if ok then
				dataInput.Text = json
				updateStatus("Data loaded successfully!", Color3.fromRGB(0, 255, 0))
			else
				updateStatus("Loaded data could not be encoded to JSON.", Color3.fromRGB(255, 0, 0))
			end
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
	else
		updateStatus(("Unknown action: %s"):format(tostring(action)), Color3.fromRGB(255, 0, 0))
	end
end)

print("Data Loss Test Client Script Initialized")
