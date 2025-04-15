-- RAYFIELD
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
    Name = "Xenz Hub",
    Icon = 0,
    LoadingTitle = "Xenz Cheats",
    LoadingSubtitle = "by Xenz Cheats",
    Theme = "Default",
    DisableRayfieldPrompts = false,
    DisableBuildWarnings = false,
    ConfigurationSaving = {Enabled = true, FileName = "Big Hub"},
    Discord = {Enabled = false, Invite = "noinvitelink", RememberJoins = false},
    KeySystem = false
})

-- VARIÁVEIS BASE
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer
local ESPObjects = {}

-- ESP TOGGLE
local ESPEnabled = false
local ESPShowBox = true
local ESPShowName = true
local ESPShowHealth = true
local ESPShowDistance = true

-- AIMBOT VARIÁVEIS
local AimbotEnabled = false
local AimbotFOV = 150
local AimbotSmoothness = 0.1
local AimbotWallCheck = true
local AimbotTeamCheck = true

-- ABA VISUAL
local VisualTab = Window:CreateTab("Visual", 4483362458)
VisualTab:CreateToggle({Name = "ESP", CurrentValue = false, Flag = "ESP_Toggle", Callback = function(Value) ESPEnabled = Value end})
VisualTab:CreateToggle({Name = "Mostrar Caixa", CurrentValue = true, Flag = "ESP_ShowBox", Callback = function(Value) ESPShowBox = Value end})
VisualTab:CreateToggle({Name = "Mostrar Nome", CurrentValue = true, Flag = "ESP_ShowName", Callback = function(Value) ESPShowName = Value end})
VisualTab:CreateToggle({Name = "Mostrar Vida", CurrentValue = true, Flag = "ESP_ShowHealth", Callback = function(Value) ESPShowHealth = Value end})
VisualTab:CreateToggle({Name = "Mostrar Distância", CurrentValue = true, Flag = "ESP_ShowDistance", Callback = function(Value) ESPShowDistance = Value end})

-- ESP + CHAMS
local function CreateESP(player)
    local box = Drawing.new("Square")
    box.Thickness = 1
    box.Transparency = 1
    box.Filled = false

    local name = Drawing.new("Text")
    name.Size = 14
    name.Center = true
    name.Outline = true

    local info = Drawing.new("Text")
    info.Size = 13
    info.Center = true
    info.Outline = true

    local teamColor = player.Team and player.Team.TeamColor.Color or Color3.fromRGB(255, 255, 255)
    box.Color = teamColor
    name.Color = teamColor
    info.Color = Color3.new(0, 255, 0)

    local highlight = Instance.new("Highlight")
    highlight.Name = "XenzChams"
    highlight.FillColor = teamColor
    highlight.FillTransparency = 0.5
    highlight.OutlineColor = Color3.new(0, 0, 0)
    highlight.OutlineTransparency = 0
    highlight.Adornee = player.Character
    highlight.Parent = game.CoreGui

    ESPObjects[player] = {Box = box, Name = name, Info = info, Chams = highlight}
end

local function RemoveESP(player)
    if ESPObjects[player] then
        for _, obj in pairs(ESPObjects[player]) do
            if typeof(obj) == "Instance" then
                obj:Destroy()
            else
                obj:Remove()
            end
        end
        ESPObjects[player] = nil
    end
end

RunService.RenderStepped:Connect(function()
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            if not ESPObjects[player] then
                CreateESP(player)
            end

            local esp = ESPObjects[player]
            local root = player.Character:FindFirstChild("HumanoidRootPart")
            local head = player.Character:FindFirstChild("Head")
            local humanoid = player.Character:FindFirstChild("Humanoid")
            local pos, onScreen = Camera:WorldToViewportPoint(root.Position)

            if ESPEnabled and onScreen then
                local distance = (root.Position - Camera.CFrame.Position).Magnitude
                local size = Vector2.new(50, 100) / (distance / 50)
                local topLeft = Vector2.new(pos.X - size.X / 2, pos.Y - size.Y / 2)

                if ESPShowBox then
                    esp.Box.Size = size
                    esp.Box.Position = topLeft
                    esp.Box.Visible = true
                end

                if ESPShowName then
                    esp.Name.Position = Vector2.new(pos.X, topLeft.Y - 16)
                    esp.Name.Text = player.Name
                    esp.Name.Visible = true
                end

                if ESPShowHealth then
                    esp.Info.Position = Vector2.new(pos.X, pos.Y + size.Y / 2 + 5)
                    esp.Info.Text = "HP: " .. math.floor(humanoid.Health) .. " | " .. math.floor(distance) .. "m"
                    esp.Info.Visible = true
                end

                if esp.Chams then esp.Chams.Enabled = true end
            else
                for _, obj in pairs(esp) do
                    if typeof(obj) ~= "Instance" then obj.Visible = false end
                    if typeof(obj) == "Instance" and obj:IsA("Highlight") then obj.Enabled = false end
                end
            end
        else
            RemoveESP(player)
        end
    end
end)

-- AIMBOT ABA
local AimbotTab = Window:CreateTab("Aimbot", 4483362458)

AimbotTab:CreateToggle({Name = "Ativar Aimbot", CurrentValue = false, Flag = "AimbotToggle", Callback = function(v) AimbotEnabled = v end})
AimbotTab:CreateSlider({Name = "FOV", Range = {50, 500}, Increment = 10, Suffix = "px", CurrentValue = AimbotFOV, Flag = "FOVSlider", Callback = function(v) AimbotFOV = v end})
AimbotTab:CreateSlider({Name = "Suavidade", Range = {0, 1}, Increment = 0.01, CurrentValue = AimbotSmoothness, Flag = "SmoothSlider", Callback = function(v) AimbotSmoothness = v end})
AimbotTab:CreateToggle({Name = "Wall Check", CurrentValue = true, Flag = "WallCheckToggle", Callback = function(v) AimbotWallCheck = v end})
AimbotTab:CreateToggle({Name = "Team Check", CurrentValue = true, Flag = "TeamCheckToggle", Callback = function(v) AimbotTeamCheck = v end})

-- DRAW FOV
local fovCircle = Drawing.new("Circle")
fovCircle.Color = Color3.fromRGB(255, 255, 255)
fovCircle.Thickness = 1
fovCircle.NumSides = 100
fovCircle.Radius = AimbotFOV
fovCircle.Filled = false
fovCircle.Transparency = 1

RunService.RenderStepped:Connect(function()
    fovCircle.Visible = AimbotEnabled
    fovCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
    fovCircle.Radius = AimbotFOV
end)

local function IsVisible(targetPart)
    local origin = Camera.CFrame.Position
    local ray = Ray.new(origin, (targetPart.Position - origin).Unit * 500)
    local hit = workspace:FindPartOnRay(ray, LocalPlayer.Character, false, true)
    return hit == nil or hit:IsDescendantOf(targetPart.Parent)
end

local function GetClosestTarget()
    local closestPlayer, shortestDistance = nil, AimbotFOV
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
            if AimbotTeamCheck and player.Team == LocalPlayer.Team then continue end
            local head = player.Character.Head
            local pos, visible = Camera:WorldToViewportPoint(head.Position)
            if visible then
                local mouseDist = (Vector2.new(pos.X, pos.Y) - Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)).Magnitude
                if mouseDist < shortestDistance then
                    if not AimbotWallCheck or IsVisible(head) then
                        closestPlayer = head
                        shortestDistance = mouseDist
                    end
                end
            end
        end
    end
    return closestPlayer
end

RunService.RenderStepped:Connect(function()
    if AimbotEnabled then
        local target = GetClosestTarget()
        if target then
            local camPos = Camera.CFrame.Position
            local direction = (target.Position - camPos).Unit
            local newPos = camPos + direction *
