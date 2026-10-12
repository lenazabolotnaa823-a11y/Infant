-- PRIVATE HUB V4 (Расширенная и оптимизированная версия для Delta / Android)
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local player = Players.LocalPlayer

local gui = player:WaitForChild("PlayerGui")
local old = gui:FindFirstChild("MainHubV4")
if old then old:Destroy() end

-- Главное окно
local main = Instance.new("Frame")
main.Name = "MainHubV4"
main.Size = UDim2.new(0, 360, 0, 440)
main.Position = UDim2.new(0.5, -180, 0.5, -220)
main.BackgroundColor3 = Color3.fromRGB(18, 20, 30)
main.BorderSizePixel = 0
main.Active = true
main.Draggable = true
main.Parent = gui

Instance.new("UICorner", main).CornerRadius = UDim.new(0, 14)

-- Верхняя панель (Титл)
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -60, 0, 50)
title.Position = UDim2.new(0, 14, 0, 0)
title.BackgroundTransparency = 1
title.Text = "⚡ PRIVATE HUB V4"
title.TextColor3 = Color3.fromRGB(175, 110, 255)
title.TextSize = 20
title.Font = Enum.Font.GothamBold
title.TextXAlignment = Enum.TextXAlignment.Left
title.Parent = main

-- Кнопка свернуть (—)
local hide = Instance.new("TextButton")
hide.Size = UDim2.new(0, 36, 0, 36)
hide.Position = UDim2.new(1, -46, 0, 7)
hide.Text = "—"
hide.TextColor3 = Color3.new(1, 1, 1)
hide.BackgroundColor3 = Color3.fromRGB(35, 38, 55)
hide.Font = Enum.Font.GothamBold
hide.Parent = main
Instance.new("UICorner", hide).CornerRadius = UDim.new(0, 8)

-- Кнопка открытия (Реопен)
local reopen = Instance.new("TextButton")
reopen.Size = UDim2.new(0, 120, 0, 40)
reopen.Position = UDim2.new(0, 14, 0.5, -20)
reopen.Text = "📂 OPEN HUB"
reopen.TextColor3 = Color3.new(1, 1, 1)
reopen.BackgroundColor3 = Color3.fromRGB(80, 45, 140)
reopen.Font = Enum.Font.GothamBold
reopen.Visible = false
reopen.Parent = gui
Instance.new("UICorner", reopen).CornerRadius = UDim.new(0, 8)

hide.Activated:Connect(function()
    main.Visible = false
    reopen.Visible = true
end)

reopen.Activated:Connect(function()
    main.Visible = true
    reopen.Visible = false
end)

-- Панель вкладок
local tabs = Instance.new("Frame")
tabs.Position = UDim2.new(0, 12, 0, 55)
tabs.Size = UDim2.new(1, -24, 0, 36)
tabs.BackgroundTransparency = 1
tabs.Parent = main

local tabLayout = Instance.new("UIListLayout")
tabLayout.FillDirection = Enum.FillDirection.Horizontal
tabLayout.Padding = UDim.new(0, 5)
tabLayout.Parent = tabs

-- Область контента (Скролл)
local content = Instance.new("ScrollingFrame")
content.Position = UDim2.new(0, 12, 0, 100)
content.Size = UDim2.new(1, -24, 1, -112)
content.BackgroundColor3 = Color3.fromRGB(25, 28, 42)
content.BorderSizePixel = 0
content.ScrollBarThickness = 3
content.AutomaticCanvasSize = Enum.AutomaticSize.Y
content.CanvasSize = UDim2.new()
content.Parent = main
Instance.new("UICorner", content).CornerRadius = UDim.new(0, 10)

local layout = Instance.new("UIListLayout")
layout.Padding = UDim.new(0, 6)
layout.SortOrder = Enum.SortOrder.LayoutOrder
layout.Parent = content

local padding = Instance.new("UIPadding")
padding.PaddingTop = UDim.new(0, 8)
padding.PaddingLeft = UDim.new(0, 8)
padding.PaddingRight = UDim.new(0, 8)
padding.Parent = content

-- Список функций (добавлены Noclip и Infinite Jump)
local features = {
    Player = {"Speed", "Jump Boost", "Fly", "Noclip", "Infinite Jump"},
    Visuals = {"ESP Players", "Fullbright", "Name Tags"},
    Combat = {"Aura NPC", "Hitbox Expander"},
    Farm = {"Auto Collect", "Auto Farm Lite"},
    Extra = {"FPS Boost", "Anti-AFK", "Rejoin Game"}
}

local enabled = {}
local flyConn, noclipConn, infJumpConn = nil, nil, nil

-- Логика читов
local function toggleFeature(name, state)
    local char = player.Character
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    local hrp = char and char:FindFirstChild("HumanoidRootPart")

    if name == "Speed" then
        if hum then hum.WalkSpeed = state and 90 or 16 end
    elseif name == "Jump Boost" then
        if hum then hum.UseJumpPower = true hum.JumpPower = state and 130 or 50 end
    elseif name == "Fly" then
        if state and hrp and hum then
            local bv = Instance.new("BodyVelocity")
            bv.Name = "FlyVelocity"
            bv.MaxForce = Vector3.new(1e5, 1e5, 1e5)
            bv.Velocity = Vector3.new(0, 0, 0)
            bv.Parent = hrp
            flyConn = RunService.RenderStepped:Connect(function()
                if hum and hrp and bv then
                    bv.Velocity = hum.MoveDirection * 55 + Vector3.new(0, hum.Jump and 35 or 0, 0)
                end
            end)
        else
            if flyConn then flyConn:Disconnect() end
            if hrp and hrp:FindFirstChild("FlyVelocity") then hrp.FlyVelocity:Destroy() end
        end
    elseif name == "Noclip" then
        if state then
            noclipConn = RunService.Stepped:Connect(function()
                if player.Character then
                    for _, part in ipairs(player.Character:GetDescendants()) do
                        if part:IsA("BasePart") then part.CanCollide = false end
                    end
                end
            end)
        else
            if noclipConn then noclipConn:Disconnect() end
        end
    elseif name == "Infinite Jump" then
        if state then
            infJumpConn = UserInputService.JumpRequest:Connect(function()
                if hum then hum:ChangeState(Enum.HumanoidStateType.Jumping) end
            end)
        else
            if infJumpConn then infJumpConn:Disconnect() end
        end
    elseif name == "ESP Players" then
        if state then
            for _, p in ipairs(Players:GetPlayers()) do
                if p ~= player and p.Character and not p.Character:FindFirstChild("ESP_Box") then
                    local hl = Instance.new("Highlight")
                    hl.Name = "ESP_Box"
                    hl.FillColor = Color3.fromRGB(150, 50, 255)
                    hl.Parent = p.Character
                end
            end
        else
            for _, p in ipairs(Players:GetPlayers()) do
                if p.Character and p.Character:FindFirstChild("ESP_Box") then
                    p.Character.ESP_Box:Destroy()
                end
            end
        end
    elseif name == "Anti-AFK" then
        if state then
            local vu = game:GetService("VirtualUser")
            player.Idled:Connect(function()
                vu:Button2Down(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
                task.wait(1)
                vu:Button2Up(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
            end)
        end
    elseif name == "Rejoin Game" then
        game:GetService("TeleportService"):TeleportToPlaceInstance(game.PlaceId, game.JobId, player)
    end
end

local function clearContent()
    for _, obj in ipairs(content:GetChildren()) do
        if obj:IsA("TextButton") then obj:Destroy() end
    end
end

local function showTab(tabName)
    clearContent()
    for _, featureName in ipairs(features[tabName]) do
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(1, -4, 0, 42)
        btn.BackgroundColor3 = Color3.fromRGB(38, 42, 60)
        btn.TextColor3 = Color3.new(1, 1, 1)
        btn.TextSize = 13
        btn.Font = Enum.Font.GothamMedium
        btn.Parent = content
        Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)

        local function update()
            btn.Text = "   " .. featureName .. (enabled[featureName] and "  [ON]" or "  [OFF]")
            btn.BackgroundColor3 = enabled[featureName] and Color3.fromRGB(95, 50, 175) or Color3.fromRGB(38, 42, 60)
        end
        update()

        btn.Activated:Connect(function()
            enabled[featureName] = not enabled[featureName]
            update()
            toggleFeature(featureName, enabled[featureName])
        end)
    end
end

-- Создаем вкладки
for tabName in pairs(features) do
    local tab = Instance.new("TextButton")
    tab.Size = UDim2.new(0, 62, 1, 0)
    tab.BackgroundColor3 = Color3.fromRGB(55, 40, 95)
    tab.TextColor3 = Color3.new(1, 1, 1)
    tab.TextSize = 11
    tab.Font = Enum.Font.GothamBold
    tab.Text = tabName
    tab.Parent = tabs
    Instance.new("UICorner", tab).CornerRadius = UDim.new(0, 6)

    tab.Activated:Connect(function()
        showTab(tabName)
    end)
end

showTab("Player")
print("PRIVATE HUB V4 successfully loaded!")# Infant
