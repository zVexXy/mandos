local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local RaycastParams = RaycastParams.new()

-- CONFIG
local TEAM_MODE = true -- Troque para false para modo solo (todos contra todos)
local SHOT_INTERVAL = 0.12 -- Intervalo entre tiros automáticos (ajuste para anticheat)

-- INTERFACE
local gui = Instance.new("ScreenGui")
gui.Name = "AimbotESPGui"
gui.IgnoreGuiInset = true
gui.ResetOnSpawn = false

pcall(function() gui.Parent = game.CoreGui end)
if not gui.Parent then
    gui.Parent = LocalPlayer:WaitForChild("PlayerGui")
end

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 120, 0, 110)
frame.Position = UDim2.new(0, 15, 1, -130)
frame.BackgroundTransparency = 0
frame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true
frame.Parent = gui

local aimbotBtn = Instance.new("TextButton")
aimbotBtn.Size = UDim2.new(0, 100, 0, 24)
aimbotBtn.Position = UDim2.new(0, 10, 0, 8)
aimbotBtn.Text = "Aimbot: OFF"
aimbotBtn.BackgroundColor3 = Color3.fromRGB(80, 150, 80)
aimbotBtn.TextColor3 = Color3.fromRGB(255,255,255)
aimbotBtn.Font = Enum.Font.SourceSansBold
aimbotBtn.TextSize = 16
aimbotBtn.Parent = frame

local espBtn = Instance.new("TextButton")
espBtn.Size = UDim2.new(0, 100, 0, 24)
espBtn.Position = UDim2.new(0, 10, 0, 38)
espBtn.Text = "ESP: OFF"
espBtn.BackgroundColor3 = Color3.fromRGB(150, 80, 80)
espBtn.TextColor3 = Color3.fromRGB(255,255,255)
espBtn.Font = Enum.Font.SourceSansBold
espBtn.TextSize = 16
espBtn.Parent = frame

local modeBtn = Instance.new("TextButton")
modeBtn.Size = UDim2.new(0, 100, 0, 24)
modeBtn.Position = UDim2.new(0, 10, 0, 68)
modeBtn.Text = TEAM_MODE and "Modo: Time" or "Modo: Solo"
modeBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 150)
modeBtn.TextColor3 = Color3.fromRGB(255,255,255)
modeBtn.Font = Enum.Font.SourceSansBold
modeBtn.TextSize = 16
modeBtn.Parent = frame

local aimbotOn = false
local espOn = false

aimbotBtn.MouseButton1Click:Connect(function()
    aimbotOn = not aimbotOn
    aimbotBtn.Text = aimbotOn and "Aimbot: ON" or "Aimbot: OFF"
    aimbotBtn.BackgroundColor3 = aimbotOn and Color3.fromRGB(40, 180, 40) or Color3.fromRGB(80,150,80)
end)
espBtn.MouseButton1Click:Connect(function()
    espOn = not espOn
    espBtn.Text = espOn and "ESP: ON" or "ESP: OFF"
    espBtn.BackgroundColor3 = espOn and Color3.fromRGB(180, 40, 40) or Color3.fromRGB(150,80,80)
end)
modeBtn.MouseButton1Click:Connect(function()
    TEAM_MODE = not TEAM_MODE
    modeBtn.Text = TEAM_MODE and "Modo: Time" or "Modo: Solo"
end)

-- Para mobile, garantir que o Frame seja realmente movível:
if UIS.TouchEnabled then
    local dragging, dragInput, dragStart, startPos
    frame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = frame.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)
    frame.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.Touch then
            local delta = input.Position - dragStart
            frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
end

-- Função para pegar inimigo vivo mais próximo
local function GetClosestEnemy()
    local minDist = math.huge
    local closest = nil
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local hum = player.Character:FindFirstChildOfClass("Humanoid")
            local head = player.Character:FindFirstChild("Head")
            if hum and hum.Health > 0 and head then
                if TEAM_MODE then
                    if player.Team ~= LocalPlayer.Team then
                        -- Time adversário
                        local dist = (LocalPlayer.Character.Head.Position - head.Position).Magnitude
                        if dist < minDist then
                            minDist = dist
                            closest = player
                        end
                    end
                else
                    -- Modo solo (todos contra todos)
                    local dist = (LocalPlayer.Character.Head.Position - head.Position).Magnitude
                    if dist < minDist then
                        minDist = dist
                        closest = player
                    end
                end
            end
        end
    end
    return closest
end

-- ESP: mostra TODOS inimigos vivos na tela
local EspBoxes = {}
local function UpdateESP()
    for _, box in ipairs(EspBoxes) do
        box:Destroy()
    end
    EspBoxes = {}

    if not espOn then return end

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
            local hum = player.Character:FindFirstChildOfClass("Humanoid")
            if hum and hum.Health > 0 then
                if TEAM_MODE then
                    if player.Team ~= LocalPlayer.Team then
                        local pos, onScreen = Camera:WorldToViewportPoint(player.Character.Head.Position)
                        if onScreen then
                            local box = Instance.new("Frame", gui)
                            box.Size = UDim2.new(0, 28, 0, 28)
                            box.Position = UDim2.new(0, pos.X-14, 0, pos.Y-14)
                            box.BackgroundTransparency = 0.5
                            box.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
                            box.BorderSizePixel = 0
                            table.insert(EspBoxes, box)
                        end
                    end
                else
                    local pos, onScreen = Camera:WorldToViewportPoint(player.Character.Head.Position)
                    if onScreen then
                        local box = Instance.new("Frame", gui)
                        box.Size = UDim2.new(0, 28, 0, 28)
                        box.Position = UDim2.new(0, pos.X-14, 0, pos.Y-14)
                        box.BackgroundTransparency = 0.5
                        box.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
                        box.BorderSizePixel = 0
                        table.insert(EspBoxes, box)
                    end
                end
            end
        end
    end
end

-- Checa se inimigo está visível (sem parede/objeto na frente)
local function IsEnemyVisible(head)
    RaycastParams.FilterDescendantsInstances = {LocalPlayer.Character}
    RaycastParams.FilterType = Enum.RaycastFilterType.Blacklist
    RaycastParams.IgnoreWater = true
    local direction = (head.Position - Camera.CFrame.Position)
    local result = workspace:Raycast(Camera.CFrame.Position, direction, RaycastParams)
    if result then
        return result.Instance:IsDescendantOf(head.Parent)
    end
    return false
end

-- Função de disparo (PC e Mobile)
local function ShootAt(head)
    if UIS.TouchEnabled then
        -- Para mobile: simula toque no botão de disparo (ajuste para seu sistema)
        -- Aqui você pode customizar para seu jogo
        -- Exemplo: tente disparar usando Event remoto (caso seu jogo use RemoteEvent para disparo)
        local weapon = LocalPlayer.Character:FindFirstChildOfClass("Tool")
        if weapon and weapon:FindFirstChild("Fire") then
            weapon.Fire:FireServer(head.Position)
        else
            -- Simula toque (não funciona em todos jogos, ajuste para seu caso)
            local VirtualInputManager = game:GetService("VirtualInputManager")
            VirtualInputManager:SendTouchEvent(1, UIS:GetMouseLocation().X, UIS:GetMouseLocation().Y, true, game, 0)
            VirtualInputManager:SendTouchEvent(1, UIS:GetMouseLocation().X, UIS:GetMouseLocation().Y, false, game, 0)
        end
    else
        -- PC: simula clique do mouse esquerdo
        local VirtualInputManager = game:GetService("VirtualInputManager")
        VirtualInputManager:SendMouseButtonEvent(UIS:GetMouseLocation().X, UIS:GetMouseLocation().Y, 0, true, game, 0)
        VirtualInputManager:SendMouseButtonEvent(UIS:GetMouseLocation().X, UIS:GetMouseLocation().Y, 0, false, game, 0)
    end
end

local lastShot = 0

RunService.RenderStepped:Connect(function()
    UpdateESP()
    if aimbotOn then
        local enemy = GetClosestEnemy()
        if enemy and enemy.Character and enemy.Character:FindFirstChild("Head") then
            local head = enemy.Character.Head
            if IsEnemyVisible(head) then
                Camera.CFrame = CFrame.new(Camera.CFrame.Position, head.Position)
                if tick() - lastShot > SHOT_INTERVAL then
                    ShootAt(head)
                    lastShot = tick()
                end
            end
        end
    end
end)
