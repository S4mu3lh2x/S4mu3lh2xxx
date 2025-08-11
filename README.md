-- Script para adicionar funcionalidades de Aimbot, Godmode, Fly, Money, ESP, Teleport e Weapon Selection
-- AVISO: Este script é para uso em um jogo próprio no Roblox Studio, não para exploits em jogos de terceiros.

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Configurações
local SETTINGS = {
    AimbotEnabled = false,
    FlyEnabled = false,
    GodmodeEnabled = false,
    ESPEnabled = false,
    FlySpeed = 50,
    MoneyAmount = 10000,
}

-- Função para dar Godmode
local function enableGodmode()
    if LocalPlayer.Character and SETTINGS.GodmodeEnabled then
        local humanoid = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid.MaxHealth = math.huge
            humanoid.Health = math.huge
            humanoid:GetPropertyChangedSignal("Health"):Connect(function()
                humanoid.Health = math.huge
            end)
        end
    end
end

-- Função para ativar Fly
local function enableFly()
    if LocalPlayer.Character and SETTINGS.FlyEnabled then
        local bodyVelocity = Instance.new("BodyVelocity")
        bodyVelocity.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
        bodyVelocity.Parent = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        
        RunService.RenderStepped:Connect(function()
            if SETTINGS.FlyEnabled then
                local moveDirection = Vector3.new(0, 0, 0)
                if UserInputService:IsKeyDown(Enum.KeyCode.W) then
                    moveDirection = moveDirection + Camera.CFrame.LookVector
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.S) then
                    moveDirection = moveDirection - Camera.CFrame.LookVector
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.A) then
                    moveDirection = moveDirection - Camera.CFrame.RightVector
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.D) then
                    moveDirection = moveDirection + Camera.CFrame.RightVector
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
                    moveDirection = moveDirection + Vector3.new(0, 1, 0)
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then
                    moveDirection = moveDirection - Vector3.new(0, 1, 0)
                end
                bodyVelocity.Velocity = moveDirection * SETTINGS.FlySpeed
            else
                bodyVelocity:Destroy()
            end
        end)
    end
end

-- Função para Aimbot
local function enableAimbot()
    RunService.RenderStepped:Connect(function()
        if SETTINGS.AimbotEnabled and UserInputService:IsMouseButtonPressed(Enum.UserInputCode.Right) then
            local closestPlayer = nil
            local closestDistance = math.huge
            local mousePos = UserInputService:GetMouseLocation()
            
            for _, player in pairs(Players:GetPlayers()) do
                if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
                    local head = player.Character.Head
                    local screenPos, onScreen = Camera:WorldToViewportPoint(head.Position)
                    if onScreen then
                        local distance = (Vector2.new(mousePos.X, mousePos.Y) - Vector2.new(screenPos.X, screenPos.Y)).Magnitude
                        if distance < closestDistance then
                            closestDistance = distance
                            closestPlayer = player
                        end
                    end
                end
            end
            
            if closestPlayer and closestPlayer.Character and closestPlayer.Character:FindFirstChild("Head") then
                Camera.CFrame = CFrame.new(Camera.CFrame.Position, closestPlayer.Character.Head.Position)
            end
        end
    end)
end

-- Função para ESP (mostrar nomes dos jogadores)
local function enableESP()
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local billboard = Instance.new("BillboardGui")
            billboard.Name = "ESP"
            billboard.Adornee = player.Character:FindFirstChild("Head")
            billboard.Size = UDim2.new(0, 100, 0, 40)
            billboard.StudsOffset = Vector3.new(0, 3, 0)
            billboard.AlwaysOnTop = true
            billboard.Parent = player.Character
            
            local textLabel = Instance.new("TextLabel")
            textLabel.Size = UDim2.new(1, 0, 1, 0)
            textLabel.BackgroundTransparency = 1
            textLabel.Text = player.Name
            textLabel.TextColor3 = Color3.new(1, 0, 0)
            textLabel.TextScaled = true
            textLabel.Parent = billboard
        end
    end
end

-- Função para dar dinheiro
local function giveMoney()
    -- Substitua "Money" pelo nome da propriedade de dinheiro no seu jogo
    if LocalPlayer:FindFirstChild("leaderstats") and LocalPlayer.leaderstats:FindFirstChild("Money") then
        LocalPlayer.leaderstats.Money.Value = LocalPlayer.leaderstats.Money.Value + SETTINGS.MoneyAmount
    else
        warn("Propriedade 'Money' não encontrada em leaderstats!")
    end
end

-- Função para teleporte
local function teleportToPosition(position)
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(position)
    end
end

-- Função para pegar qualquer arma
local function giveWeapon(weaponName)
    local tool = game.ReplicatedStorage:FindFirstChild(weaponName)
    if tool and tool:IsA("Tool") then
        tool:Clone().Parent = LocalPlayer.Backpack
    else
        warn("Arma '" .. weaponName .. "' não encontrada em ReplicatedStorage!")
    end
end

-- Interface de usuário simples (usando BillboardGui para controle)
local function createGUI()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Parent = LocalPlayer.PlayerGui
    
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(0, 200, 0, 300)
    frame.Position = UDim2.new(0, 10, 0, 10)
    frame.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
    frame.Parent = screenGui
    
    local function createButton(name, position, callback)
        local button = Instance.new("TextButton")
        button.Size = UDim2.new(0, 180, 0, 40)
        button.Position = UDim2.new(0, 10, 0, position)
        button.Text = name
        button.BackgroundColor3 = Color3.new(0.4, 0.4, 0.4)
        button.TextColor3 = Color3.new(1, 1, 1)
        button.Parent = frame
        button.MouseButton1Click:Connect(callback)
    end
    
    createButton("Toggle Aimbot", 10, function()
        SETTINGS.AimbotEnabled = not SETTINGS.AimbotEnabled
        print("Aimbot: " .. (SETTINGS.AimbotEnabled and "ON" or "OFF"))
    end)
    
    createButton("Toggle Godmode", 60, function()
        SETTINGS.GodmodeEnabled = not SETTINGS.GodmodeEnabled
        if SETTINGS.GodmodeEnabled then enableGodmode() end
        print("Godmode: " .. (SETTINGS.GodmodeEnabled and "ON" or "OFF"))
    end)
    
    createButton("Toggle Fly", 110, function()
        SETTINGS.FlyEnabled = not SETTINGS.FlyEnabled
        if SETTINGS.FlyEnabled then enableFly() end
        print("Fly: " .. (SETTINGS.FlyEnabled and "ON" or "OFF"))
    end)
    
    createButton("Toggle ESP", 160, function()
        SETTINGS.ESPEnabled = not SETTINGS.ESPEnabled
        if SETTINGS.ESPEnabled then enableESP() end
        print("ESP: " .. (SETTINGS.ESPEnabled and "ON" or "OFF"))
    end)
    
    createButton("Give 10k Money", 210, giveMoney)
    
    createButton("Teleport to Spawn", 260, function()
        teleportToPosition(Vector3.new(0, 5, 0)) -- Substitua pelas coordenadas desejadas
    end)
end

-- Inicialização
createGUI()
enableAimbot()

-- Exemplo de como dar uma arma (substitua "Sword" pelo nome da arma no seu jogo)
giveWeapon("Sword")
