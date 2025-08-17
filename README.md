local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local HttpService = game:GetService("HttpService")

local Player = Players.LocalPlayer
local PlayerGui = Player:WaitForChild("PlayerGui")

-- Verificação para evitar múltiplos GUIs
local existingGui = PlayerGui:FindFirstChild("DataLossTestGui")
if existingGui then
    existingGui:Destroy()
end

-- Remote Event
local DataLossRemote = ReplicatedStorage:FindFirstChild("DataLossRemote")
if not DataLossRemote or not DataLossRemote:IsA("RemoteEvent") then
    warn("DataLossRemote não encontrado ou é inválido.")
    return
end

-- Criação do GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "DataLossTestGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = PlayerGui

-- (Demais elementos do GUI permanecem como no seu código original)
-- A única mudança técnica está no OnClientEvent abaixo:

DataLossRemote.OnClientEvent:Connect(function(action, result)
    if action == "SaveResult" then
        if result then
            updateStatus("Data saved successfully!", Color3.fromRGB(0, 255, 0))
        else
            updateStatus("Failed to save data!", Color3.fromRGB(255, 0, 0))
        end
    elseif action == "LoadResult" then
        if result then
            dataInput.Text = HttpService:JSONEncode(result)
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
