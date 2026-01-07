loadstring(game:HttpGet("https://raw.githubusercontent.com/NervigeMuecke/Z3US-V2/refs/heads/main/Important/drawinglib.lua"))()

local players = game:GetService("Players")
local camera = workspace.CurrentCamera
local client = players.LocalPlayer
local runService = game:GetService("RunService")
local settings = {
    enabled = false,
    teamCheck = true,
    teamColor = false,
    maxDistance = 250,
    
    boxes = {
        enabled = false,
        color = Color3.fromRGB(19, 0, 255),
        outline = {
            enabled = false,
            color = Color3.fromRGB(0, 0, 0)
        },
        filled = {
            enabled = false,
            color = Color3.fromRGB(19, 0, 255),
            transparency = 0.25
        }
    },
    
    names = {
        enabled = false,
        color = Color3.fromRGB(19, 0, 255),
        outline = {
            enabled = false,
            color = Color3.fromRGB(0, 0, 0)
        }
    },
    
    health = {
        enabled = false,
        color = Color3.fromRGB(0, 255, 0),
        colorLow = Color3.fromRGB(255, 0, 0),
        outline = {
            enabled = false,
            color = Color3.fromRGB(0, 0, 0)
        },
        text = {
            enabled = false,
            outline = {
                enabled = false
            }
        }
    },
    
    distance = {
        enabled = false,
        color = Color3.fromRGB(19, 0, 255),
        outline = {
            enabled = false,
            color = Color3.fromRGB(0, 0, 0)
        }
    },
    
    weapon = {
        enabled = false,
        color = Color3.fromRGB(19, 0, 255),
        outline = {
            enabled = false,
            color = Color3.fromRGB(0, 0, 0)
        }
    }
}
local trackedPlayers = {}
local function createDrawing(class, properties)
    local drawing = Drawing.new(class)
    for property, value in properties do
        pcall(function() drawing[property] = value end)
    end
    return drawing
end
local function shouldShow(player, distance)
    if not settings.enabled or distance > settings.maxDistance then
        return false
    end
    
    if settings.teamCheck then
        if player.Team == client.Team then
            return false
        end
    end    
    
    return true
end
local function getTeamColor(player, defaultColor)
    return settings.teamColor and player.TeamColor.Color or defaultColor
end
local function setupPlayer(player)
    local drawings = {
        box = createDrawing("Square", {Visible = false}),
        boxFilled = createDrawing("Square", {Visible = false, Filled = true}),
        boxOutline = createDrawing("Square", {Visible = false}),
        name = createDrawing("Text", {Visible = false, Center = true}),
        health = createDrawing("Line", {Visible = false}),
        healthOutline = createDrawing("Line", {Visible = false}),
        healthText = createDrawing("Text", {Visible = false, Center = false}),
        distance = createDrawing("Text", {Visible = false, Center = true}),
        weapon = createDrawing("Text", {Visible = false, Center = true})
    }
    
    drawings.boxOutline.ZIndex = 0
    drawings.boxFilled.ZIndex = 0 
    drawings.name.ZIndex = 0
    drawings.healthOutline.ZIndex = 0
    
    trackedPlayers[player] = {
        drawings = drawings,
        lastUpdate = 0
    }
    
    local function onCharacterAdded(character)
        if not character:FindFirstChild("Humanoid") or not character:FindFirstChild("HumanoidRootPart") then
            local connection
            connection = character.ChildAdded:Connect(function(child)
                if child:IsA("Humanoid") or child.Name == "HumanoidRootPart" then
                    connection:Disconnect()
                end
            end)
        end
    end
    
    if player.Character then
        onCharacterAdded(player.Character)
    end
    
    player.CharacterAdded:Connect(onCharacterAdded)
end
local function removePlayer(player)
    local data = trackedPlayers[player]
    if not data then return end
    
    for _, drawing in pairs(data.drawings) do
        drawing:Remove()
    end
    
    trackedPlayers[player] = nil
end
local function updateESP()
    if not client.Character or not client.Character:FindFirstChild("HumanoidRootPart") then
        return
    end

    local humanoid = client.Character:FindFirstChildOfClass("Humanoid")
    local isAlive = humanoid and humanoid.Health > 0

    if not isAlive then
        for _, data in pairs(trackedPlayers) do
            for _, drawing in pairs(data.drawings) do
                drawing.Visible = false
            end
        end
        return
    end
    
    local clientPos = client.Character.HumanoidRootPart.Position
    local now = tick()
    
    for player, data in pairs(trackedPlayers) do
        if now - data.lastUpdate > 0 then
            data.lastUpdate = now
            
            local character = player.Character
            if not character then continue end
            
            local humanoid = character:FindFirstChildOfClass("Humanoid")
            local rootPart = character:FindFirstChild("HumanoidRootPart")
            if not humanoid or not rootPart or humanoid.Health <= 0 then
                for _, drawing in pairs(data.drawings) do
                    drawing.Visible = false
                end
                continue
            end
            
            local distance = (clientPos - rootPart.Position).Magnitude
            local position, visible = camera:WorldToViewportPoint(rootPart.Position)
            
            if visible and shouldShow(player, distance) then
                local scale = 1 / (position.Z * math.tan(math.rad(camera.FieldOfView * 0.5)) * 2) * 1000
                local width, height = math.floor(4.5 * scale), math.floor(6 * scale)
                local x, y = math.floor(position.X), math.floor(position.Y)
                local xPos, yPos = math.floor(x - width * 0.5), math.floor((y - height * 0.5) + (0.5 * scale))
                
                local box = data.drawings.box
                local boxFilled = data.drawings.boxFilled
                local boxOutline = data.drawings.boxOutline
                
                box.Size = Vector2.new(width, height)
                box.Position = Vector2.new(xPos, yPos)
                boxFilled.Size = box.Size
                boxFilled.Position = box.Position
                boxOutline.Size = box.Size
                boxOutline.Position = box.Position
                
                box.Color = getTeamColor(player, settings.boxes.color)
                box.Thickness = 1
                boxFilled.Color = getTeamColor(player, settings.boxes.filled.color)
                boxFilled.Transparency = settings.boxes.filled.transparency
                boxOutline.Color = settings.boxes.outline.color
                boxOutline.Thickness = 3
                
                boxOutline.ZIndex = box.ZIndex - 1
                boxFilled.ZIndex = boxOutline.ZIndex - 1
                
                local name = data.drawings.name
                name.Text = player.Name
                name.Size = math.max(math.min(math.abs(12.5 * scale), 12.5), 10)
                name.Position = Vector2.new(x, (yPos - name.TextBounds.Y) - 2)
                name.Color = getTeamColor(player, settings.names.color)
                name.Outline = settings.names.outline.enabled
                name.OutlineColor = settings.names.outline.color
                name.ZIndex = box.ZIndex + 1
                
                local healthPercent = math.clamp(humanoid.Health / humanoid.MaxHealth * 100, 0, 100)
                local health = data.drawings.health
                local healthOutline = data.drawings.healthOutline
                local healthText = data.drawings.healthText
                
                healthOutline.From = Vector2.new(xPos - 5, yPos)
                healthOutline.To = Vector2.new(xPos - 5, yPos + height)
                health.From = Vector2.new(xPos - 5, (yPos + height) - 1)
                health.To = Vector2.new(xPos - 5, ((health.From.Y - ((height / 100) * healthPercent))) + 2)
                healthText.Text =  "HP" .. math.floor(humanoid.Health)
                healthText.Size = math.max(math.min(math.abs(11 * scale), 11), 10)
                healthText.Position = Vector2.new(health.To.X - (healthText.TextBounds.X + 3), (health.To.Y - (2 / scale)))
                
                health.Color = settings.health.colorLow:Lerp(settings.health.color, healthPercent * 0.01)
                healthOutline.Color = settings.health.outline.color
                healthOutline.Thickness = 3
                healthText.Color = health.Color
                healthText.Outline = settings.health.text.outline.enabled
                healthText.OutlineColor = settings.health.outline.color
                healthOutline.ZIndex = health.ZIndex - 1
                
                local distanceDrawing = data.drawings.distance
                distanceDrawing.Text = math.floor(distance)
                distanceDrawing.Size = math.max(math.min(math.abs(11 * scale), 11), 10)
                distanceDrawing.Position = Vector2.new(x, (yPos + height) + (distanceDrawing.TextBounds.Y * 0.25))
                distanceDrawing.Color = getTeamColor(player, settings.distance.color)
                distanceDrawing.Outline = settings.distance.outline.enabled
                distanceDrawing.OutlineColor = settings.distance.outline.color
                
                local gunName = character:FindFirstChild("Gun")
                local weapon = gunName:GetAttribute("GunName") or "Knife"
                
                local weaponDrawing = data.drawings.weapon
                weaponDrawing.Text = weapon
                weaponDrawing.Size = math.max(math.min(math.abs(11 * scale), 11), 10)
                weaponDrawing.Position = settings.distance.enabled and 
                    Vector2.new(distanceDrawing.Position.x, distanceDrawing.Position.Y + (weaponDrawing.TextBounds.Y * 0.75)) or 
                    distanceDrawing.Position
                weaponDrawing.Color = getTeamColor(player, settings.weapon.color)
                weaponDrawing.Outline = settings.weapon.outline.enabled
                weaponDrawing.OutlineColor = settings.weapon.outline.color
                
                box.Visible = settings.boxes.enabled
                boxFilled.Visible = settings.boxes.filled.enabled and box.Visible
                boxOutline.Visible = settings.boxes.outline.enabled and box.Visible
                name.Visible = settings.names.enabled
                health.Visible = settings.health.enabled
                healthOutline.Visible = settings.health.outline.enabled and health.Visible
                healthText.Visible = settings.health.text.enabled and health.Visible
                distanceDrawing.Visible = settings.distance.enabled
                weaponDrawing.Visible = settings.weapon.enabled
            else
                for _, drawing in pairs(data.drawings) do
                    drawing.Visible = false
                end
            end
        end
        continue
    end
end
for _, player in ipairs(players:GetPlayers()) do
    if player ~= client then
        setupPlayer(player)
    end
end
players.PlayerAdded:Connect(function(player)
    if player ~= client then
        setupPlayer(player)
    end
end)
players.PlayerRemoving:Connect(function(player)
    removePlayer(player)
end)
runService.RenderStepped:Connect(function()
    updateESP()
end)

return {
    settings = settings
}
