--!strict
-- Services
local DataStoreService = game:GetService("DataStoreService")
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local ServerScriptService = game:GetService("ServerScriptService")

-- Configuration
local DATASTORE_NAME = "PlayerDataTestStore"
local MAX_RETRIES = 3
local RETRY_DELAY = 2 -- seconds

-- Data Store
local testDataStore = DataStoreService:GetDataStore(DATASTORE_NAME)

-- Remote Events
local DataLossRemote = Instance.new("RemoteEvent")
DataLossRemote.Name = "DataLossRemote"
DataLossRemote.Parent = ReplicatedStorage

-- Function to save test data with retry logic
local function saveTestData(player: Player, data: any)
    local userId = player.UserId
    local success = false
    
    for attempt = 1, MAX_RETRIES do
        local ok, err = pcall(function()
            testDataStore:SetAsync(userId, data)
        end)
        
        if ok then
            success = true
            break
        else
            warn("Error saving test data for player " .. player.Name .. " (Attempt " .. attempt .. "): " .. err)
            task.wait(RETRY_DELAY)
        end
    end
    
    return success
end

-- Function to load test data with retry logic
local function loadTestData(player: Player): any
    local userId = player.UserId
    
    for attempt = 1, MAX_RETRIES do
        local ok, result = pcall(function()
            return testDataStore:GetAsync(userId)
        end)
        
        if ok then
            return result
        else
            warn("Error loading test data for player " .. player.Name .. " (Attempt " .. attempt .. "): " .. result)
            task.wait(RETRY_DELAY)
        end
    end
    
    return nil
end

-- Function to wipe player data
local function wipePlayerData(player: Player): boolean
    local userId = player.UserId
    
    for attempt = 1, MAX_RETRIES do
        local ok, err = pcall(function()
            testDataStore:RemoveAsync(userId)
        end)
        
        if ok then
            return true
        else
            warn("Error wiping data for player " .. player.Name .. " (Attempt " .. attempt .. "): " .. err)
            task.wait(RETRY_DELAY)
        end
    end
    
    return false
end

-- Handle remote events
DataLossRemote.OnServerEvent:Connect(function(player: Player, action: string, data: any)
    if action == "SaveTestData" then
        local success = saveTestData(player, data)
        DataLossRemote:FireClient(player, "SaveResult", success)
    elseif action == "LoadTestData" then
        local data = loadTestData(player)
        DataLossRemote:FireClient(player, "LoadResult", data)
    elseif action == "WipeData" then
        local success = wipePlayerData(player)
        DataLossRemote:FireClient(player, "WipeResult", success)
    end
end)

-- Handle player leaving to ensure data is saved
Players.PlayerRemoving:Connect(function(player: Player)
    -- Optional: Save any pending data when player leaves
end)

print("Data Loss Test Server Script Initialized")
