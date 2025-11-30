local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local localPlayer = Players.LocalPlayer

-- CONFIGURAÇÃO DO ESP
local CONFIG = {
    ENABLE = true,
    MAX_DISTANCE = 250,
    COLOR = Color3.fromRGB(0,255,0),

    FILL_TRANSPARENCY = 0.4,
    OUTLINE_TRANSPARENCY = 0,
    OUTLINE_THICKNESS = 4,
}

local espMap = {}

---------------------------------------------------------------------
-- 🟩 CRIAR BOTÃO PARA LIGAR / DESLIGAR O ESP
---------------------------------------------------------------------

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = localPlayer:WaitForChild("PlayerGui")

local Button = Instance.new("TextButton")
Button.Parent = ScreenGui
Button.Size = UDim2.new(0, 60, 0, 25)
Button.Position = UDim2.new(0.02, 0, 0.2, 0)
Button.Text = "ON"
Button.TextColor3 = Color3.fromRGB(0, 255, 0)
Button.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
Button.BorderSizePixel = 1
Button.Active = true

Button.MouseButton1Click:Connect(function()
    CONFIG.ENABLE = not CONFIG.ENABLE

    if CONFIG.ENABLE then
        Button.Text = "ON"
        Button.TextColor3 = Color3.fromRGB(0, 255, 0)
    else
        Button.Text = "OFF"
        Button.TextColor3 = Color3.fromRGB(255, 0, 0)
    end
end)

---------------------------------------------------------------------
-- 🟩 FUNÇÃO PARA CRIAR ESP EM CADA PLAYER
---------------------------------------------------------------------

local function createESP(player, character)
    if player == localPlayer then return end
    if not character then return end

    local hl = Instance.new("Highlight")
    hl.Adornee = character
    hl.FillColor = CONFIG.COLOR
    hl.OutlineColor = CONFIG.COLOR
    hl.FillTransparency = CONFIG.FILL_TRANSPARENCY
    hl.OutlineTransparency = CONFIG.OUTLINE_TRANSPARENCY
    hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    hl.Parent = workspace

    local conn = RunService.RenderStepped:Connect(function()
        local myChar = localPlayer.Character
        local myRoot = myChar and myChar:FindFirstChild("HumanoidRootPart")
        local targetRoot = character:FindFirstChild("HumanoidRootPart")
        if not (myRoot and targetRoot) then return end

        local dist = (myRoot.Position - targetRoot.Position).Magnitude

        if CONFIG.ENABLE and dist <= CONFIG.MAX_DISTANCE then
            hl.Enabled = true
        else
            hl.Enabled = false
        end
    end)

    espMap[player] = { hl = hl, conn = conn }
end

local function removeESP(player)
    local data = espMap[player]
    if not data then return end
    if data.conn then data.conn:Disconnect() end
    if data.hl then data.hl:Destroy() end
    espMap[player] = nil
end

---------------------------------------------------------------------
-- 🟩 APLICAR ESP NOS PLAYERS DO JOGO
---------------------------------------------------------------------

for _, plr in ipairs(Players:GetPlayers()) do
    if plr ~= localPlayer then
        if plr.Character then createESP(plr, plr.Character) end
        plr.CharacterAdded:Connect(function(char)
            createESP(plr, char)
        end)
    end
end

Players.PlayerAdded:Connect(function(plr)
    plr.CharacterAdded:Connect(function(char)
        createESP(plr, char)
    end)
end)

Players.PlayerRemoving:Connect(function(plr)
    removeESP(plr)
end)
