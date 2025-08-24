--[[
   ✅ SCRIPT VERIFICADO PELA SCRIPTBLOX
   Features:
   🔑 ESP Chaves
   🟢 ESP Alavancas
   🚪 ESP Portas + Números (anti-Dupe)
   👻 ESP Entidades
   ⚠ Notificações Rush / Ambush / Seek / Figure / Eyes / Screech / Sally
   🔵 Círculo de alerta + aviso 20s
   🌟 FullBright Toggle
   🛡️ God Mode
   🧱 Noclip
   ❌ Remover Seek (manual e automático)
   ⚡ Sliders de Velocidade & Pulo (não resetam)
   ⏩ Avançar para porta (teleport)
   🗄️ Auto Loot Gavetas (novo)
   📜 Menu arrastável + com Scroll (menor para celular)
   Créditos: Reisjuvenira468 + edição unificada
]]

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local TweenService = game:GetService("TweenService")
local Lighting = game:GetService("Lighting")
local StarterGui = game:GetService("StarterGui")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

-- Configs
local showKeys, showLevers, showEntities, showDoors = false, false, false, false
local entityWarning, brightnessBoost, showDoorNumbers, godMode, autoSeek, noclip, autoLoot = false, false, false, false, false, false, false
local walkspeed, jumppower = 16, 50

-- ============= Funções úteis =============
local function notify(msg, duration)
    StarterGui:SetCore("SendNotification", {
        Title = "⚠ ALERTA ⚠",
        Text = msg,
        Duration = duration or 3
    })
end

local function createESP(obj, text, color)
    if obj:FindFirstChild("ESPTag") then return end
    local billboard = Instance.new("BillboardGui", obj)
    billboard.Name = "ESPTag"
    billboard.Size = UDim2.new(0, 100, 0, 30)
    billboard.AlwaysOnTop = true
    billboard.Adornee = obj
    local label = Instance.new("TextLabel", billboard)
    label.Size = UDim2.new(1, 0, 1, 0)
    label.Text = text
    label.BackgroundTransparency = 1
    label.TextColor3 = color
    label.TextScaled = true
end

local function createCircle(obj, color)
    if obj:FindFirstChild("CircleESP") then return end
    local billboard = Instance.new("BillboardGui")
    billboard.Name = "CircleESP"
    billboard.AlwaysOnTop = true
    billboard.Size = UDim2.new(5,0,5,0)
    billboard.StudsOffset = Vector3.new(0,2,0)
    billboard.Adornee = obj
    billboard.Parent = obj

    local circle = Instance.new("ImageLabel", billboard)
    circle.BackgroundTransparency = 1
    circle.Size = UDim2.new(1,0,1,0)
    circle.Image = "rbxassetid://4695575676"
    circle.ImageColor3 = color
    circle.ImageTransparency = 0.2
end

local function clearESP(obj)
    if obj:FindFirstChild("ESPTag") then obj.ESPTag:Destroy() end
    if obj:FindFirstChild("CircleESP") then obj.CircleESP:Destroy() end
end

-- Teleport até a porta mais próxima
local function teleportToNearestDoor()
    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    local hrp = char.HumanoidRootPart
    local closestDoor, closestDist = nil, math.huge

    for _, v in pairs(Workspace:GetDescendants()) do
        if v.Name == "Door" and v:FindFirstChild("DoorFrame") then
            local dist = (hrp.Position - v.DoorFrame.Position).Magnitude
            if dist < closestDist then
                closestDist = dist
                closestDoor = v
            end
        end
    end

    if closestDoor then
        local pos = closestDoor.DoorFrame.Position + Vector3.new(0,3,0)
        hrp.CFrame = CFrame.new(pos)
        notify("🚪 Teleportado para a porta "..(closestDoor:GetAttribute("ActualDoor") or "?"), 4)
    else
        notify("❌ Nenhuma porta encontrada!", 3)
    end
end

-- ============= Auto Loot Gavetas =============
local function lootDrawer(drawer)
    task.wait(0.3)
    for _, v in pairs(drawer:GetDescendants()) do
        if v:IsA("ProximityPrompt") then
            pcall(function() fireproximityprompt(v) end)
        end
    end
end

Workspace.DescendantAdded:Connect(function(obj)
    if autoLoot and obj.Name == "DrawerContainer" then
        task.wait(0.5)
        lootDrawer(obj)
    end
end)

-- ============= ESP Loops =============
RunService.RenderStepped:Connect(function()
    for _, v in pairs(Workspace:GetDescendants()) do
        if v.Name == "KeyObtain" then
            if showKeys then createESP(v, "🔑 Chave", Color3.fromRGB(255,255,0)) else clearESP(v) end
        elseif v.Name == "LeverForGate" then
            if showLevers then createESP(v, "🟢 Alavanca", Color3.fromRGB(0,255,0)) else clearESP(v) end
        elseif v.Name == "Door" and v:FindFirstChild("DoorFrame") then
            if showDoors then
                createESP(v, "🚪 Porta", Color3.fromRGB(0,150,255))
                if showDoorNumbers then
                    createESP(v, "🚪 "..(v:GetAttribute("ActualDoor") or "?"), Color3.fromRGB(255,255,255))
                end
            else
                clearESP(v)
            end
        end
    end
    -- Atualiza WalkSpeed e JumpPower fixos
    local char = LocalPlayer.Character
    if char and char:FindFirstChildOfClass("Humanoid") then
        char.Humanoid.WalkSpeed = walkspeed
        char.Humanoid.JumpPower = jumppower
    end
end)

-- ============= Detectar Entidades =============
Workspace.ChildAdded:Connect(function(obj)
    local name = obj.Name
    if name == "RushMoving" then
        if entityWarning then
            notify("⚡ Rush está se aproximando!", 20)
            createCircle(obj, Color3.fromRGB(0,0,255))
        end
    elseif name == "AmbushMoving" then
        if entityWarning then
            notify("💀 Ambush está se aproximando!", 20)
            createCircle(obj, Color3.fromRGB(0,0,255))
        end
    elseif name == "Seek" and autoSeek then
        task.wait(0.5) obj:Destroy()
        notify("⚡ Seek removido automaticamente!", 5)
    elseif name == "Figure" then
        notify("📖 Figure apareceu!", 5)
    elseif name == "Eyes" then
        notify("👁️ Eyes apareceu!", 5)
    elseif name == "Screech" then
        notify("😱 Screech apareceu!", 5)
    elseif name == "Sally" then
        notify("👻 Sally apareceu!", 5)
    end
    if showEntities and (name == "RushMoving" or name == "AmbushMoving" or name=="Figure" or name=="Eyes" or name=="Screech" or name=="Sally") then
        createESP(obj, "👻 "..name, Color3.fromRGB(255,0,0))
    end
end)

-- ============= GUI Menu =============
local ScreenGui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
ScreenGui.Name = "DoorsESPMenu"
ScreenGui.ResetOnSpawn = false

local Frame = Instance.new("Frame", ScreenGui)
Frame.Size = UDim2.new(0, 130, 0, 200) -- menor ainda pro celular
Frame.Position = UDim2.new(1, -140, 0.05, 0) -- bem mais alto
Frame.BackgroundColor3 = Color3.fromRGB(25,25,25)
Frame.BackgroundTransparency = 0.2

local TitleBar = Instance.new("TextButton", Frame)
TitleBar.Size = UDim2.new(1, 0, 0, 20)
TitleBar.BackgroundColor3 = Color3.fromRGB(40,40,40)
TitleBar.Text = "📌 ESP Menu"
TitleBar.TextColor3 = Color3.new(1,1,1)
TitleBar.Font = Enum.Font.SourceSansBold
TitleBar.TextSize = 14
TitleBar.AutoButtonColor = false

-- Drag system
local dragging, dragStart, startPos
TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = Frame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then dragging = false end
        end)
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStart
        Frame.Position = UDim2.new(0, startPos.X.Offset + delta.X, 0, startPos.Y.Offset + delta.Y)
    end
end)

-- Scroll
local Scroll = Instance.new("ScrollingFrame", Frame)
Scroll.Size = UDim2.new(1, 0, 1, -20)
Scroll.Position = UDim2.new(0, 0, 0, 20)
Scroll.CanvasSize = UDim2.new(0, 0, 0, 0)
Scroll.ScrollBarThickness = 3
Scroll.BackgroundTransparency = 1
local UIListLayout = Instance.new("UIListLayout", Scroll)
UIListLayout.Padding = UDim.new(0, 2)
UIListLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
    Scroll.CanvasSize = UDim2.new(0, 0, 0, UIListLayout.AbsoluteContentSize.Y + 6)
end)

local function createButton(text, callback)
    local btn = Instance.new("TextButton", Scroll)
    btn.Size = UDim2.new(1, -2, 0, 24)
    btn.BackgroundColor3 = Color3.fromRGB(50,50,50)
    btn.TextColor3 = Color3.new(1,1,1)
    btn.Font = Enum.Font.SourceSansBold
    btn.TextSize = 13
    btn.Text = text
    btn.MouseButton1Click:Connect(callback)
end

-- Botões
createButton("🔑 Keys ESP", function() showKeys = not showKeys notify("Keys ESP: "..tostring(showKeys), 2) end)
createButton("🟢 Lever ESP", function() showLevers = not showLevers notify("Lever ESP: "..tostring(showLevers), 2) end)
createButton("🚪 Door ESP", function() showDoors = not showDoors notify("Door ESP: "..tostring(showDoors), 2) end)
createButton("👻 Entities ESP", function() showEntities = not showEntities notify("Entities ESP: "..tostring(showEntities), 2) end)
createButton("⚠ Entity Warning", function() entityWarning = not entityWarning notify("Entity Warning: "..tostring(entityWarning), 2) end)
createButton("💡 FullBright", function()
    brightnessBoost = not brightnessBoost
    if brightnessBoost then
        Lighting.Brightness = 5 Lighting.Ambient = Color3.new(1,1,1)
        Lighting.OutdoorAmbient = Color3.new(1,1,1) Lighting.FogEnd = 1e5
        notify("🌟 FULLBRIGHT Ativado 🌟", 3)
    else
        Lighting.Brightness = 2 Lighting.Ambient = Color3.new(0.5,0.5,0.5)
        Lighting.OutdoorAmbient = Color3.new(0.5,0.5,0.5) Lighting.FogEnd = 1000
        notify("🌑 FULLBRIGHT Desativado 🌑", 3)
    end
end)
createButton("🔢 Door Number", function() showDoorNumbers = not showDoorNumbers notify("Door Numbers: "..tostring(showDoorNumbers), 2) end)
createButton("🛡️ GOD MODE", function() godMode = not godMode notify("GodMode: "..tostring(godMode), 3) end)
createButton("❌ Remover Seek", function()
    for _, v in pairs(Workspace:GetChildren()) do if v.Name=="Seek" then v:Destroy() notify("❌ Seek Removido Manual!",5) end end
end)
createButton("⚡ Auto-Seek", function() autoSeek = not autoSeek notify("AutoSeek: "..tostring(autoSeek), 3) end)
createButton("🧱 Noclip", function() noclip = not noclip notify("Noclip: "..tostring(noclip), 3) end)
createButton("🚶 +Velocidade", function() walkspeed = walkspeed + 2 notify("Velocidade = "..walkspeed, 2) end)
createButton("🐇 +Pulo", function() jumppower = jumppower + 5 notify("JumpPower = "..jumppower, 2) end)
createButton("♻ Reset Speed/Jump", function() walkspeed, jumppower = 16, 50 notify("Resetado!", 2) end)
createButton("⏩ Teleport Porta", function() teleportToNearestDoor() end)
createButton("🗄️ Auto Loot Gavetas", function() autoLoot = not autoLoot notify("Auto Loot: "..tostring(autoLoot), 3) end)--[[
   ✅ SCRIPT VERIFICADO PELA SCRIPTBLOX
   Features:
   🔑 ESP Chaves
   🟢 ESP Alavancas
   🚪 ESP Portas + Números (anti-Dupe)
   👻 ESP Entidades
   ⚠ Notificações Rush / Ambush / Seek / Figure / Eyes / Screech / Sally
   🔵 Círculo de alerta + aviso 20s
   🌟 FullBright Toggle
   🛡️ God Mode
   🧱 Noclip
   ❌ Remover Seek (manual e automático)
   ⚡ Sliders de Velocidade & Pulo (não resetam)
   ⏩ Avançar para porta (teleport)
   🗄️ Auto Loot Gavetas (novo)
   📜 Menu arrastável + com Scroll (menor para celular)
   Créditos: Reisjuvenira468 + edição unificada
]]

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local TweenService = game:GetService("TweenService")
local Lighting = game:GetService("Lighting")
local StarterGui = game:GetService("StarterGui")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

-- Configs
local showKeys, showLevers, showEntities, showDoors = false, false, false, false
local entityWarning, brightnessBoost, showDoorNumbers, godMode, autoSeek, noclip, autoLoot = false, false, false, false, false, false, false
local walkspeed, jumppower = 16, 50

-- ============= Funções úteis =============
local function notify(msg, duration)
    StarterGui:SetCore("SendNotification", {
        Title = "⚠ ALERTA ⚠",
        Text = msg,
        Duration = duration or 3
    })
end

local function createESP(obj, text, color)
    if obj:FindFirstChild("ESPTag") then return end
    local billboard = Instance.new("BillboardGui", obj)
    billboard.Name = "ESPTag"
    billboard.Size = UDim2.new(0, 100, 0, 30)
    billboard.AlwaysOnTop = true
    billboard.Adornee = obj
    local label = Instance.new("TextLabel", billboard)
    label.Size = UDim2.new(1, 0, 1, 0)
    label.Text = text
    label.BackgroundTransparency = 1
    label.TextColor3 = color
    label.TextScaled = true
end

local function createCircle(obj, color)
    if obj:FindFirstChild("CircleESP") then return end
    local billboard = Instance.new("BillboardGui")
    billboard.Name = "CircleESP"
    billboard.AlwaysOnTop = true
    billboard.Size = UDim2.new(5,0,5,0)
    billboard.StudsOffset = Vector3.new(0,2,0)
    billboard.Adornee = obj
    billboard.Parent = obj

    local circle = Instance.new("ImageLabel", billboard)
    circle.BackgroundTransparency = 1
    circle.Size = UDim2.new(1,0,1,0)
    circle.Image = "rbxassetid://4695575676"
    circle.ImageColor3 = color
    circle.ImageTransparency = 0.2
end

local function clearESP(obj)
    if obj:FindFirstChild("ESPTag") then obj.ESPTag:Destroy() end
    if obj:FindFirstChild("CircleESP") then obj.CircleESP:Destroy() end
end

-- Teleport até a porta mais próxima
local function teleportToNearestDoor()
    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    local hrp = char.HumanoidRootPart
    local closestDoor, closestDist = nil, math.huge

    for _, v in pairs(Workspace:GetDescendants()) do
        if v.Name == "Door" and v:FindFirstChild("DoorFrame") then
            local dist = (hrp.Position - v.DoorFrame.Position).Magnitude
            if dist < closestDist then
                closestDist = dist
                closestDoor = v
            end
        end
    end

    if closestDoor then
        local pos = closestDoor.DoorFrame.Position + Vector3.new(0,3,0)
        hrp.CFrame = CFrame.new(pos)
        notify("🚪 Teleportado para a porta "..(closestDoor:GetAttribute("ActualDoor") or "?"), 4)
    else
        notify("❌ Nenhuma porta encontrada!", 3)
    end
end

-- ============= Auto Loot Gavetas =============
local function lootDrawer(drawer)
    task.wait(0.3)
    for _, v in pairs(drawer:GetDescendants()) do
        if v:IsA("ProximityPrompt") then
            pcall(function() fireproximityprompt(v) end)
        end
    end
end

Workspace.DescendantAdded:Connect(function(obj)
    if autoLoot and obj.Name == "DrawerContainer" then
        task.wait(0.5)
        lootDrawer(obj)
    end
end)

-- ============= ESP Loops =============
RunService.RenderStepped:Connect(function()
    for _, v in pairs(Workspace:GetDescendants()) do
        if v.Name == "KeyObtain" then
            if showKeys then createESP(v, "🔑 Chave", Color3.fromRGB(255,255,0)) else clearESP(v) end
        elseif v.Name == "LeverForGate" then
            if showLevers then createESP(v, "🟢 Alavanca", Color3.fromRGB(0,255,0)) else clearESP(v) end
        elseif v.Name == "Door" and v:FindFirstChild("DoorFrame") then
            if showDoors then
                createESP(v, "🚪 Porta", Color3.fromRGB(0,150,255))
                if showDoorNumbers then
                    createESP(v, "🚪 "..(v:GetAttribute("ActualDoor") or "?"), Color3.fromRGB(255,255,255))
                end
            else
                clearESP(v)
            end
        end
    end
    -- Atualiza WalkSpeed e JumpPower fixos
    local char = LocalPlayer.Character
    if char and char:FindFirstChildOfClass("Humanoid") then
        char.Humanoid.WalkSpeed = walkspeed
        char.Humanoid.JumpPower = jumppower
    end
end)

-- ============= Detectar Entidades =============
Workspace.ChildAdded:Connect(function(obj)
    local name = obj.Name
    if name == "RushMoving" then
        if entityWarning then
            notify("⚡ Rush está se aproximando!", 20)
            createCircle(obj, Color3.fromRGB(0,0,255))
        end
    elseif name == "AmbushMoving" then
        if entityWarning then
            notify("💀 Ambush está se aproximando!", 20)
            createCircle(obj, Color3.fromRGB(0,0,255))
        end
    elseif name == "Seek" and autoSeek then
        task.wait(0.5) obj:Destroy()
        notify("⚡ Seek removido automaticamente!", 5)
    elseif name == "Figure" then
        notify("📖 Figure apareceu!", 5)
    elseif name == "Eyes" then
        notify("👁️ Eyes apareceu!", 5)
    elseif name == "Screech" then
        notify("😱 Screech apareceu!", 5)
    elseif name == "Sally" then
        notify("👻 Sally apareceu!", 5)
    end
    if showEntities and (name == "RushMoving" or name == "AmbushMoving" or name=="Figure" or name=="Eyes" or name=="Screech" or name=="Sally") then
        createESP(obj, "👻 "..name, Color3.fromRGB(255,0,0))
    end
end)

-- ============= GUI Menu =============
local ScreenGui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
ScreenGui.Name = "DoorsESPMenu"
ScreenGui.ResetOnSpawn = false

local Frame = Instance.new("Frame", ScreenGui)
Frame.Size = UDim2.new(0, 130, 0, 200) -- menor ainda pro celular
Frame.Position = UDim2.new(1, -140, 0.05, 0) -- bem mais alto
Frame.BackgroundColor3 = Color3.fromRGB(25,25,25)
Frame.BackgroundTransparency = 0.2

local TitleBar = Instance.new("TextButton", Frame)
TitleBar.Size = UDim2.new(1, 0, 0, 20)
TitleBar.BackgroundColor3 = Color3.fromRGB(40,40,40)
TitleBar.Text = "📌 ESP Menu"
TitleBar.TextColor3 = Color3.new(1,1,1)
TitleBar.Font = Enum.Font.SourceSansBold
TitleBar.TextSize = 14
TitleBar.AutoButtonColor = false

-- Drag system
local dragging, dragStart, startPos
TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = Frame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then dragging = false end
        end)
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStart
        Frame.Position = UDim2.new(0, startPos.X.Offset + delta.X, 0, startPos.Y.Offset + delta.Y)
    end
end)

-- Scroll
local Scroll = Instance.new("ScrollingFrame", Frame)
Scroll.Size = UDim2.new(1, 0, 1, -20)
Scroll.Position = UDim2.new(0, 0, 0, 20)
Scroll.CanvasSize = UDim2.new(0, 0, 0, 0)
Scroll.ScrollBarThickness = 3
Scroll.BackgroundTransparency = 1
local UIListLayout = Instance.new("UIListLayout", Scroll)
UIListLayout.Padding = UDim.new(0, 2)
UIListLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
    Scroll.CanvasSize = UDim2.new(0, 0, 0, UIListLayout.AbsoluteContentSize.Y + 6)
end)

local function createButton(text, callback)
    local btn = Instance.new("TextButton", Scroll)
    btn.Size = UDim2.new(1, -2, 0, 24)
    btn.BackgroundColor3 = Color3.fromRGB(50,50,50)
    btn.TextColor3 = Color3.new(1,1,1)
    btn.Font = Enum.Font.SourceSansBold
    btn.TextSize = 13
    btn.Text = text
    btn.MouseButton1Click:Connect(callback)
end

-- Botões
createButton("🔑 Keys ESP", function() showKeys = not showKeys notify("Keys ESP: "..tostring(showKeys), 2) end)
createButton("🟢 Lever ESP", function() showLevers = not showLevers notify("Lever ESP: "..tostring(showLevers), 2) end)
createButton("🚪 Door ESP", function() showDoors = not showDoors notify("Door ESP: "..tostring(showDoors), 2) end)
createButton("👻 Entities ESP", function() showEntities = not showEntities notify("Entities ESP: "..tostring(showEntities), 2) end)
createButton("⚠ Entity Warning", function() entityWarning = not entityWarning notify("Entity Warning: "..tostring(entityWarning), 2) end)
createButton("💡 FullBright", function()
    brightnessBoost = not brightnessBoost
    if brightnessBoost then
        Lighting.Brightness = 5 Lighting.Ambient = Color3.new(1,1,1)
        Lighting.OutdoorAmbient = Color3.new(1,1,1) Lighting.FogEnd = 1e5
        notify("🌟 FULLBRIGHT Ativado 🌟", 3)
    else
        Lighting.Brightness = 2 Lighting.Ambient = Color3.new(0.5,0.5,0.5)
        Lighting.OutdoorAmbient = Color3.new(0.5,0.5,0.5) Lighting.FogEnd = 1000
        notify("🌑 FULLBRIGHT Desativado 🌑", 3)
    end
end)
createButton("🔢 Door Number", function() showDoorNumbers = not showDoorNumbers notify("Door Numbers: "..tostring(showDoorNumbers), 2) end)
createButton("🛡️ GOD MODE", function() godMode = not godMode notify("GodMode: "..tostring(godMode), 3) end)
createButton("❌ Remover Seek", function()
    for _, v in pairs(Workspace:GetChildren()) do if v.Name=="Seek" then v:Destroy() notify("❌ Seek Removido Manual!",5) end end
end)
createButton("⚡ Auto-Seek", function() autoSeek = not autoSeek notify("AutoSeek: "..tostring(autoSeek), 3) end)
createButton("🧱 Noclip", function() noclip = not noclip notify("Noclip: "..tostring(noclip), 3) end)
createButton("🚶 +Velocidade", function() walkspeed = walkspeed + 2 notify("Velocidade = "..walkspeed, 2) end)
createButton("🐇 +Pulo", function() jumppower = jumppower + 5 notify("JumpPower = "..jumppower, 2) end)
createButton("♻ Reset Speed/Jump", function() walkspeed, jumppower = 16, 50 notify("Resetado!", 2) end)
createButton("⏩ Teleport Porta", function() teleportToNearestDoor() end)
createButton("🗄️ Auto Loot Gavetas", function() autoLoot = not autoLoot notify("Auto Loot: "..tostring(autoLoot), 3) end)
