-- Werick Cheese Hub para Fuga do Queijo (Cheese Escape)
-- Compatível com Delta Mobile
-- Key: werick hub

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()

-- Key check
local KEY = "werick hub"
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
ScreenGui.Name = "WerickCheeseHub"

-- Função simples para criar contornos ESP
local function createESP(target, color, text)
    if not target or not target:IsA("BasePart") then return end
    if target:FindFirstChild("WerickESP") then return end

    local box = Instance.new("BoxHandleAdornment")
    box.Name = "WerickESP"
    box.Adornee = target
    box.AlwaysOnTop = true
    box.ZIndex = 10
    box.Size = target.Size + Vector3.new(0.1, 0.1, 0.1)
    box.Transparency = 0.5
    box.Color3 = color
    box.Parent = target

    if text then
        local bill = Instance.new("BillboardGui", target)
        bill.Name = "WerickESPText"
        bill.AlwaysOnTop = true
        bill.Size = UDim2.new(0, 100, 0, 30)
        bill.Adornee = target
        bill.StudsOffset = Vector3.new(0, 2, 0)

        local label = Instance.new("TextLabel", bill)
        label.Size = UDim2.new(1, 0, 1, 0)
        label.BackgroundTransparency = 1
        label.TextColor3 = color
        label.TextStrokeTransparency = 0
        label.TextStrokeColor3 = Color3.new(0, 0, 0)
        label.Font = Enum.Font.SourceSansBold
        label.Text = text
        label.TextScaled = true
    end
end

local function removeESP(target)
    if not target then return end
    if target:FindFirstChild("WerickESP") then
        target.WerickESP:Destroy()
    end
    if target:FindFirstChild("WerickESPText") then
        target.WerickESPText:Destroy()
    end
end

-- Estado das toggles
local ESPEnabled = false
local SpeedEnabled = false
local JumpBoostEnabled = false
local InstantApproachEnabled = false

-- Speed e Jump variables
local SpeedValue = 24
local JumpPowerBase = 50
local JumpPower = JumpPowerBase
local jumpCount = 0
local touchingGround = true

-- Função para aplicar Speed Boost
local function applySpeed()
    if SpeedEnabled then
        LocalPlayer.Character.Humanoid.WalkSpeed = SpeedValue
    else
        LocalPlayer.Character.Humanoid.WalkSpeed = 16
    end
end

-- Função para aplicar Jump Boost
local function onJumpRequest()
    if JumpBoostEnabled then
        jumpCount = jumpCount + 1
        JumpPower = JumpPowerBase + (jumpCount * 15)
        LocalPlayer.Character.Humanoid.JumpPower = JumpPower
        LocalPlayer.Character.Humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
    end
end

local function onStateChanged(old, new)
    if new == Enum.HumanoidStateType.Landed then
        jumpCount = 0
        JumpPower = JumpPowerBase
        LocalPlayer.Character.Humanoid.JumpPower = JumpPower
    end
end

-- Aproximar instantaneamente (Exemplo de uso, você pode adaptar)
local function instantApproach(item)
    if not item or not item:IsA("BasePart") then return end
    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end

    local hrp = char.HumanoidRootPart
    hrp.CFrame = item.CFrame + Vector3.new(0, 3, 0)
end

-- ESP Loop
local function updateESP()
    -- Limpar ESP antigo
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
            removeESP(plr.Character.HumanoidRootPart)
        end
    end

    -- ESP para jogadores
    if ESPEnabled then
        for _, plr in pairs(Players:GetPlayers()) do
            if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
                createESP(plr.Character.HumanoidRootPart, Color3.fromRGB(0, 255, 0), plr.Name)
            end
        end
    end

    -- ESP para rato gigante (assumindo que tem modelo com nome "Rat" ou parecido)
    local workspace = game:GetService("Workspace")
    for _, obj in pairs(workspace:GetChildren()) do
        if obj.Name:lower():find("rat") and obj:IsA("Model") and obj:FindFirstChild("HumanoidRootPart") then
            if ESPEnabled then
                createESP(obj.HumanoidRootPart, Color3.fromRGB(255, 0, 0), "Rato")
            else
                removeESP(obj.HumanoidRootPart)
            end
        end
    end

    -- ESP para chaves e queijos (Assumindo que eles estão com nomes ou tags específicas)
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("BasePart") then
            local n = obj.Name:lower()
            if ESPEnabled then
                if n:find("key") then
                    createESP(obj, Color3.fromRGB(0, 255, 255), "Chave")
                elseif n:find("cheese") or n:find("queijo") then
                    createESP(obj, Color3.fromRGB(255, 255, 0), "🧀 Cheese")
                else
                    removeESP(obj)
                end
            else
                removeESP(obj)
            end
        end
    end
end

-- Teleporte rápido (exemplo, você pode adicionar mais pontos)
local Teleports = {
    ["Sala Segura"] = Vector3.new(10, 5, 10), -- Ajuste as posições conforme o mapa do jogo
    ["Entrada"] = Vector3.new(0, 5, 0),
    ["Saída"] = Vector3.new(100, 5, 100),
}

local function teleportTo(pos)
    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    char.HumanoidRootPart.CFrame = CFrame.new(pos + Vector3.new(0, 3, 0))
end

-- GUI
local function createToggle(text, parent, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, -10, 0, 30)
    btn.Position = UDim2.new(0, 5, 0, 0)
    btn.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
    btn.TextColor3 = Color3.new(1,1,1)
    btn.Font = Enum.Font.SourceSansBold
    btn.TextSize = 18
    btn.Text = text .. ": OFF"
    btn.Parent = parent
    local enabled = false
    btn.MouseButton1Click:Connect(function()
        enabled = not enabled
        btn.Text = text .. (enabled and ": ON" or ": OFF")
        callback(enabled)
    end)
    return btn
end

-- Construir menu
local mainFrame = Instance.new("Frame")
mainFrame.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
mainFrame.Size = UDim2.new(0, 270, 0, 360)
mainFrame.Position = UDim2.new(0.1, 0, 0.2, 0)
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = ScreenGui

local title = Instance.new("TextLabel", mainFrame)
title.Text = "Werick Cheese Hub"
title.Size = UDim2.new(1, 0, 0, 40)
title.BackgroundTransparency = 1
title.TextColor3 = Color3.new(1,1,1)
title.Font = Enum.Font.SourceSansBold
title.TextSize = 24

local toggleESP = createToggle("ESP", mainFrame, function(v)
    ESPEnabled = v
    updateESP()
end)
toggleESP.Position = UDim2.new(0, 5, 0, 50)

local toggleSpeed = createToggle("Speed Boost", mainFrame, function(v)
    SpeedEnabled = v
    applySpeed()
end)
toggleSpeed.Position = UDim2.new(0, 5, 0, 90)

local toggleJump = createToggle("Jump Boost", mainFrame, function(v)
    JumpBoostEnabled = v
end)
toggleJump.Position = UDim2.new(0, 5, 0, 130)

local toggleInstant = createToggle("Aproximar Instantâneo", mainFrame, function(v)
    InstantApproachEnabled = v
end)
toggleInstant.Position = UDim2.new(0, 5, 0, 170)

-- Teleport buttons container
local tpLabel = Instance.new("TextLabel", mainFrame)
tpLabel.Text = "Teleports"
tpLabel.Size = UDim2.new(1, 0, 0, 30)
tpLabel.BackgroundTransparency = 1
tpLabel.TextColor3 = Color3.new(1,1,1)
tpLabel.Font = Enum.Font.SourceSansBold
tpLabel.TextSize = 20
tpLabel.Position = UDim2.new(0, 0, 0, 210)

local tpY = 240
for name, pos in pairs(Teleports) do
    local btn = Instance.new("TextButton", mainFrame)
    btn.Text = name
    btn.Size = UDim2.new(1, -10, 0, 30)
    btn.Position = UDim2.new(0, 5, 0, tpY)
    btn.BackgroundColor3 = Color3.fromRGB(180, 0, 0)
    btn.TextColor3 = Color3.new(1,1,1)
    btn.Font = Enum.Font.SourceSansBold
    btn.TextSize = 18
    btn.MouseButton1Click:Connect(function()
        teleportTo(pos)
    end)
    tpY = tpY + 40
end

-- Minimizar botão e bolinha
local minimizeBtn = Instance.new("TextButton", mainFrame)
minimizeBtn.Text = "–"
minimizeBtn.Size = UDim2.new(0, 40, 0, 30)
minimizeBtn.Position = UDim2.new(1, -45, 0, 5)
minimizeBtn.BackgroundColor3 = Color3.fromRGB(100,0,0)
minimizeBtn.TextColor3 = Color3.new(1,1,1)
minimizeBtn.Font = Enum.Font.SourceSansBold
minimizeBtn.TextSize = 24

local smallBtn = Instance.new("TextButton", ScreenGui)
smallBtn.Text = "W"
smallBtn.Size = UDim2.new(0, 40, 0, 40)
smallBtn.Position = UDim2.new(0.05, 0, 0.8, 0)
smallBtn.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
smallBtn.TextColor3 = Color3.new(1,1,1)
smallBtn.Font = Enum.Font.SourceSansBold
smallBtn.TextSize = 24
smallBtn.Visible = false
smallBtn.AutoButtonColor = false
smallBtn.Parent = ScreenGui
smallBtn.Active = true
smallBtn.Draggable = true

minimizeBtn.MouseButton1Click:Connect(function()
    mainFrame.Visible = false
    smallBtn.Visible = true
end)

smallBtn.MouseButton1Click:Connect(function()
    mainFrame.Visible = true
    smallBtn.Visible = false
end)

-- Jump control
UIS.JumpRequest:Connect(function()
    if JumpBoostEnabled then
        jumpCount = jumpCount + 1
        JumpPower = JumpPowerBase + (jumpCount * 15)
        local humanoid = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid.JumpPower = JumpPower
            humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
        end
    end
end)

local humanoid = nil
RunService.Heartbeat:Connect(function()
    local char = LocalPlayer.Character
    if char then
        if not humanoid then
            humanoid = char:FindFirstChildOfClass("Humanoid")
            if humanoid then
                humanoid.StateChanged:Connect(onStateChanged)
            end
        end
        applySpeed()
        if ESPEnabled then
            updateESP()
        end
    end
end)

-- Aproximar instantâneo em clique (exemplo)
Mouse.Button1Down:Connect(function()
    if InstantApproachEnabled then
        local target = Mouse.Target
        if target and (target.Name:lower():find("cheese") or target.Name:lower():find("key") or target.Name:lower():find("door")) then
            instantApproach(target)
        end
    end
end)

print("Werick Cheese Hub carregado com sucesso!")
