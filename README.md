local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
    Name = "Xenz Hub",
    Icon = 0,
    LoadingTitle = "Xenz Cheats",
    LoadingSubtitle = "by Xenz Cheats",
    Theme = "Default",
    DisableRayfieldPrompts = false,
    DisableBuildWarnings = false,
    ConfigurationSaving = {
       Enabled = true,
       FolderName = nil,
       FileName = "Big Hub"
    },
    Discord = {
       Enabled = false,
       Invite = "noinvitelink",
       RememberJoins = false
    },
    KeySystem = false,
    KeySettings = {
       Title = "Untitled",
       Subtitle = "Key System",
       Note = "No method of obtaining the key is provided",
       FileName = "Key",
       SaveKey = true,
       GrabKeyFromSite = false,
       Key = {"Hello"}
    }
})

-- ⚙️ Serviços e Variáveis
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer
local ESPEnabled = false
local ESPObjects = {}

-- 📁 Cria a aba e toggle
local VisualTab = Window:CreateTab("Visual", 4483362458)

VisualTab:CreateToggle({
    Name = "ESP",
    CurrentValue = false,
    Flag = "ESP_Toggle",
    Callback = function(Value)
        ESPEnabled = Value
    end,
})

-- 📦 Cria ESP
local function CreateESP(player)
    local box = Drawing.new("Square")
    box.Thickness = 1
    box.Transparency = 1
    box.Color = Color3.fromRGB(255, 255, 0)
    box.Filled = false

    local name = Drawing.new("Text")
    name.Size = 14
    name.Center = true
    name.Outline = true
    name.Color = Color3.fromRGB(255, 255, 255)

    local info = Drawing.new("Text")
    info.Size = 13
    info.Center = true
    info.Outline = true
    info.Color = Color3.fromRGB(0, 255, 0)

    ESPObjects[player] = {Box = box, Name = name, Info = info}
end

-- ❌ Remove ESP
local function RemoveESP(player)
    if ESPObjects[player] then
        for _, obj in pairs(ESPObjects[player]) do
            obj:Remove()
        end
        ESPObjects[player] = nil
    end
end

-- 🔁 Loop de Renderização
RunService.RenderStepped:Connect(function()
    if not ESPEnabled then
        for _, esp in pairs(ESPObjects) do
            for _, obj in pairs(esp) do
                obj.Visible = false
            end
        end
        return
    end

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            if not ESPObjects[player] then
                CreateESP(player)
            end

            local root = player.Character:FindFirstChild("HumanoidRootPart")
            local head = player.Character:FindFirstChild("Head")
            local humanoid = player.Character:FindFirstChild("Humanoid")
            local pos, onScreen = Camera:WorldToViewportPoint(root.Position)

            local esp = ESPObjects[player]

            if onScreen then
                local distance = math.floor((root.Position - Camera.CFrame.Position).Magnitude)
                local size = Vector2.new(50, 100) / (distance / 50)
                local topLeft = Vector2.new(pos.X - size.X / 2, pos.Y - size.Y / 2)

                esp.Box.Size = size
                esp.Box.Position = topLeft
                esp.Box.Visible = true

                esp.Name.Position = Vector2.new(pos.X, topLeft.Y - 16)
                esp.Name.Text = player.Name
                esp.Name.Visible = true

                esp.Info.Position = Vector2.new(pos.X, pos.Y + size.Y / 2 + 5)
                esp.Info.Text = "HP: " .. math.floor(humanoid.Health) .. " | " .. distance .. "m"
                esp.Info.Visible = true
            else
                for _, obj in pairs(esp) do
                    obj.Visible = false
                end
            end
        else
            RemoveESP(player)
        end
    end
end)
