local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
    Name = "[FPS] Passeio",
    LoadingTitle = "Carregando...",
    LoadingSubtitle = "",
    ConfigurationSaving = { Enabled = true, FolderName = "FPSPasseio", FileName = "Config" }
})

local AimbotTab = Window:CreateTab("Aimbot", 4483362458)
local EspTab = Window:CreateTab("ESP", 4483362458)

local aimbotAtivo = false
local espAtivo = false
local jogador = game.Players.LocalPlayer
local camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local fov = 40
local maxTransparency = 0.1
local suavidade = 0.3

local FOVring = Drawing.new("Circle")
FOVring.Visible = false
FOVring.Thickness = 2
FOVring.Color = Color3.fromRGB(128, 0, 128)
FOVring.Filled = false
FOVring.Radius = fov
FOVring.Position = camera.ViewportSize / 2
FOVring.Transparency = 0.1

local function lookAt(target)
    local posicaoAtual = camera.CFrame.Position
    local direcaoAtual = camera.CFrame.LookVector
    local direcaoAlvo = (target - posicaoAtual).unit
    
    local novoLookVector = direcaoAtual:Lerp(direcaoAlvo, suavidade)
    camera.CFrame = CFrame.lookAt(posicaoAtual, posicaoAtual + novoLookVector)
end

local function calculateTransparency(distance)
    local maxDistance = fov
    local transparency = (1 - (distance / maxDistance)) * maxTransparency
    return transparency
end

local function getClosestPlayerInFOV(trg_part)
    local nearest = nil
    local last = math.huge
    local playerMousePos = camera.ViewportSize / 2

    for _, player in ipairs(game.Players:GetPlayers()) do
        if player ~= jogador then
            local part = player.Character and player.Character:FindFirstChild(trg_part)
            if part then
                local ePos, isVisible = camera:WorldToViewportPoint(part.Position)
                local distance = (Vector2.new(ePos.x, ePos.y) - playerMousePos).Magnitude

                if distance < last and isVisible and distance < fov then
                    last = distance
                    nearest = player
                end
            end
        end
    end

    return nearest
end

local function aimbot()
    while aimbotAtivo do
        task.wait()
        local closest = getClosestPlayerInFOV("Head")
        if closest and closest.Character and closest.Character:FindFirstChild("Head") then
            lookAt(closest.Character.Head.Position)
        end
    end
end

local function atualizarESP()
    for _, player in pairs(game.Players:GetPlayers()) do
        if player ~= jogador and player.Character then
            local highlight = player.Character:FindFirstChild("ESP_Highlight")
            if espAtivo and player.Character and player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0 then
                if not highlight then
                    highlight = Instance.new("Highlight")
                    highlight.Name = "ESP_Highlight"
                    highlight.FillColor = Color3.fromRGB(255, 0, 0)
                    highlight.FillTransparency = 0.3
                    highlight.OutlineColor = Color3.fromRGB(255, 0, 0)
                    highlight.OutlineTransparency = 0
                    highlight.Parent = player.Character
                end
            else
                if highlight then
                    highlight:Destroy()
                end
            end
        end
    end
end

local function onCharacterAdded(player)
    player.CharacterAdded:Connect(function(character)
        character:WaitForChild("Humanoid")
        task.wait(0.5)
        atualizarESP()
    end)
end

for _, player in pairs(game.Players:GetPlayers()) do
    onCharacterAdded(player)
end

game.Players.PlayerAdded:Connect(function(player)
    onCharacterAdded(player)
    task.wait(0.5)
    atualizarESP()
end)

game.Players.PlayerRemoving:Connect(function()
    atualizarESP()
end)

AimbotTab:CreateToggle({
    Name = "🎯 Aimbot",
    CurrentValue = false,
    Callback = function(value)
        aimbotAtivo = value
        FOVring.Visible = value
        if value then
            task.spawn(aimbot)
        end
    end
})

EspTab:CreateToggle({
    Name = "🔴 ESP",
    CurrentValue = false,
    Callback = function(value)
        espAtivo = value
        atualizarESP()
    end
})

AimbotTab:CreateSlider({
    Name = "🎯 Tamanho do FOV",
    Min = 10,
    Max = 200,
    Default = 40,
    Callback = function(value)
        fov = value
        FOVring.Radius = value
    end
})

AimbotTab:CreateSlider({
    Name = "🔄 Suavidade da Mira",
    Min = 0,
    Max = 1,
    Default = 30,
    Callback = function(value)
        suavidade = value / 100
    end
})

RunService.RenderStepped:Connect(function()
    if aimbotAtivo then
        local centroTela = camera.ViewportSize / 2
        FOVring.Position = centroTela
        
        local closest = getClosestPlayerInFOV("Head")
        if closest then
            local ePos, isVisible = camera:WorldToViewportPoint(closest.Character.Head.Position)
            local distance = (Vector2.new(ePos.x, ePos.y) - (camera.ViewportSize / 2)).Magnitude
            FOVring.Transparency = calculateTransparency(distance)
        else
            FOVring.Transparency = 0.1
        end
    end
end)

UserInputService.InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.Delete then
        if aimbotAtivo then
            aimbotAtivo = false
            FOVring.Visible = false
        end
    end
end)
