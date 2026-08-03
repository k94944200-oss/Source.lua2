--// Auto Server Hop For Fruits - HAZE SEAS EDITION
--// Solo Míticas/Legendarias | Auto Bag 7s + Auto Equip + Auto Fruit Bag

if getgenv().AutoServerHopForFruitsLoaded then return end
getgenv().AutoServerHopForFruitsLoaded = true

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local RunService = game:GetService("RunService")
local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local Workspace = game:GetService("Workspace")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local VirtualInputManager = game:GetService("VirtualInputManager")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local hrp = character:WaitForChild("HumanoidRootPart")

--// CONFIG
local config = {
    ESPFruitDrop = false,
    AutoTween = false,
    AutoBagFruit = false,
    AutoServerHop = false,
    SmartHopIfFull = true,
    AutoEquipFruit = true,
    AutoStoreFruit = true,
    TweenSpeed = 250,
    HopDelay = 15,
    ESPUpdateRate = 0.1,
    FruitScanRate = 0.3,
    BagRange = 20,
    BagHoldDuration = 7,
    StoreDelay = 2
}

--// SISTEMA DE GUARDADO DE CONFIGURACIÓN
local SaveSystem = {
    ConfigName = "HazeSeasAutoHopConfig",
    SavePath = "HazeSeas_AutoHop_SaveData"
}

function SaveSystem:SaveConfig()
    local success, err = pcall(function()
        local json = HttpService:JSONEncode(config)
        if writefile then
            writefile(self.SavePath .. ".json", json)
            print("✅ Configuración guardada exitosamente!")
        end
    end)
    
    if not success then
        warn("❌ Error al guardar la configuración: " .. tostring(err))
    end
end

function SaveSystem:LoadConfig()
    local success, data = pcall(function()
        if isfile and isfile(self.SavePath .. ".json") then
            local json = readfile(self.SavePath .. ".json")
            return HttpService:JSONDecode(json)
        end
        return nil
    end)
    
    if success and data then
        for key, value in pairs(data) do
            if config[key] ~= nil then
                config[key] = value
            end
        end
        print("✅ Configuración cargada exitosamente!")
        return true
    else
        print("ℹ️ No se encontró configuración guardada, usando valores predeterminados")
        return false
    end
end

function SaveSystem:ResetConfig()
    config = {
        ESPFruitDrop = false,
        AutoTween = false,
        AutoBagFruit = false,
        AutoServerHop = false,
        SmartHopIfFull = true,
        AutoEquipFruit = true,
        AutoStoreFruit = true,
        TweenSpeed = 250,
        HopDelay = 15,
        ESPUpdateRate = 0.1,
        FruitScanRate = 0.3,
        BagRange = 20,
        BagHoldDuration = 7,
        StoreDelay = 2
    }
    self:SaveConfig()
    print("🔄 Configuración reiniciada a valores predeterminados")
end

function SaveSystem:AutoLoad()
    self:LoadConfig()
end

-- Cargar configuración automáticamente
SaveSystem:AutoLoad()

--// FRUTAS MÍTICAS Y LEGENDARIAS
local FRUIT_NAMES = {
    "Dragon", "Gum", "Dough", "Electricity", "Phoenix", "Okuchi", 
    "Saturn", "Magma", "Darkness", "Venom", "Tremor", "Shadow", 
    "Magnet", "Wolf", "Leopard", "Soul"
}

--// COLORES
local COLORS = {
    BG = Color3.fromRGB(18, 18, 24),
    Text = Color3.fromRGB(255, 255, 255),
    TextDim = Color3.fromRGB(160, 160, 180),
    Green = Color3.fromRGB(34, 197, 94),
    Orange = Color3.fromRGB(251, 146, 60),
    Purple = Color3.fromRGB(168, 85, 247),
    Red = Color3.fromRGB(239, 68, 68),
    Yellow = Color3.fromRGB(250, 204, 21),
    Gray = Color3.fromRGB(75, 75, 85),
    Gold = Color3.fromRGB(255, 215, 0),
    Mythic = Color3.fromRGB(148, 0, 211),
    Cyan = Color3.fromRGB(6, 182, 212),
    Pink = Color3.fromRGB(236, 72, 153)
}

--// ESTADOS
local states = {
    tweening = false,
    hopping = false,
    hopTime = config.HopDelay,
    currentHop = config.HopDelay,
    hasFruit = false,
    inventoryFull = false,
    dragging = false,
    dragStart = nil,
    startPos = nil,
    bagging = false,
    bagStartTime = 0,
    currentBagFruit = nil,
    storing = false,
    lastStoredFruit = nil
}

--// GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AutoServerHopForFruits"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.DisplayOrder = 999999

if syn and syn.protect_gui then
    syn.protect_gui(screenGui)
    screenGui.Parent = CoreGui
elseif gethui then
    screenGui.Parent = gethui()
else
    screenGui.Parent = CoreGui
end

--// FRAME PRINCIPAL
local main = Instance.new("Frame")
main.Size = UDim2.new(0, 340, 0, 520)
main.Position = UDim2.new(0.5, -170, 0.5, -260)
main.BackgroundColor3 = COLORS.BG
main.BorderSizePixel = 0
main.Active = true
main.Parent = screenGui
Instance.new("UICorner", main).CornerRadius = UDim.new(0, 16)

local shadow = Instance.new("ImageLabel")
shadow.Size = UDim2.new(1, 40, 1, 40)
shadow.Position = UDim2.new(0, -20, 0, -20)
shadow.BackgroundTransparency = 1
shadow.Image = "rbxassetid://6015897843"
shadow.ImageColor3 = Color3.new(0, 0, 0)
shadow.ImageTransparency = 0.7
shadow.ZIndex = -1
shadow.Parent = main

--// TÍTULO
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, -24, 0, 50)
titleBar.Position = UDim2.new(0, 12, 0, 12)
titleBar.BackgroundTransparency = 1
titleBar.Active = true
titleBar.Parent = main

local appleIcon = Instance.new("TextLabel")
appleIcon.Size = UDim2.new(0, 24, 0, 24)
appleIcon.BackgroundTransparency = 1
appleIcon.Text = "👑"
appleIcon.Font = Enum.Font.GothamBold
appleIcon.TextSize = 20
appleIcon.Parent = titleBar

local titleText = Instance.new("TextLabel")
titleText.Size = UDim2.new(1, -80, 0, 24)
titleText.Position = UDim2.new(0, 28, 0, 0)
titleText.BackgroundTransparency = 1
titleText.Text = "HAZE SEAS - MYTHIC ONLY"
titleText.Font = Enum.Font.GothamBlack
titleText.TextSize = 15
titleText.TextColor3 = COLORS.Gold
titleText.TextXAlignment = Enum.TextXAlignment.Left
titleText.Parent = titleBar

local subTitle = Instance.new("TextLabel")
subTitle.Size = UDim2.new(1, -80, 0, 20)
subTitle.Position = UDim2.new(0, 28, 0, 22)
subTitle.BackgroundTransparency = 1
subTitle.Text = "Auto Bag + Auto Equip + Auto Store"
subTitle.Font = Enum.Font.GothamBold
subTitle.TextSize = 10
subTitle.TextColor3 = COLORS.TextDim
subTitle.TextXAlignment = Enum.TextXAlignment.Left
subTitle.Parent = titleBar

-- Botones
local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 28, 0, 28)
closeBtn.Position = UDim2.new(1, -28, 0, 0)
closeBtn.BackgroundColor3 = COLORS.Red
closeBtn.Text = "×"
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 16
closeBtn.TextColor3 = COLORS.Text
closeBtn.Parent = titleBar
Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0, 6)

local minBtn = Instance.new("TextButton")
minBtn.Size = UDim2.new(0, 28, 0, 28)
minBtn.Position = UDim2.new(1, -60, 0, 0)
minBtn.BackgroundColor3 = COLORS.Gray
minBtn.Text = "−"
minBtn.Font = Enum.Font.GothamBold
minBtn.TextSize = 18
minBtn.TextColor3 = COLORS.Text
minBtn.Parent = titleBar
Instance.new("UICorner", minBtn).CornerRadius = UDim.new(0, 6)

--// STATUS LABELS
local invStatus = Instance.new("TextLabel")
invStatus.Size = UDim2.new(1, -24, 0, 16)
invStatus.Position = UDim2.new(0, 12, 0, 65)
invStatus.BackgroundTransparency = 1
invStatus.Text = "Inventario: Vacío"
invStatus.Font = Enum.Font.GothamBold
invStatus.TextSize = 11
invStatus.TextColor3 = COLORS.TextDim
invStatus.TextXAlignment = Enum.TextXAlignment.Left
invStatus.Parent = main

local bagStatus = Instance.new("TextLabel")
bagStatus.Size = UDim2.new(1, -24, 0, 18)
bagStatus.Position = UDim2.new(0, 12, 0, 82)
bagStatus.BackgroundTransparency = 1
bagStatus.Text = "🎒 Esperando..."
bagStatus.Font = Enum.Font.GothamBold
bagStatus.TextSize = 13
bagStatus.TextColor3 = COLORS.Cyan
bagStatus.TextXAlignment = Enum.TextXAlignment.Left
bagStatus.Parent = main

local storeStatus = Instance.new("TextLabel")
storeStatus.Size = UDim2.new(1, -24, 0, 16)
storeStatus.Position = UDim2.new(0, 12, 0, 100)
storeStatus.BackgroundTransparency = 1
storeStatus.Text = "💼 Auto Store: Listo"
storeStatus.Font = Enum.Font.GothamBold
storeStatus.TextSize = 11
storeStatus.TextColor3 = COLORS.Pink
storeStatus.TextXAlignment = Enum.TextXAlignment.Left
storeStatus.Parent = main

local timerLabel = Instance.new("TextLabel")
timerLabel.Size = UDim2.new(1, -24, 0, 16)
timerLabel.Position = UDim2.new(0, 12, 0, 116)
timerLabel.BackgroundTransparency = 1
timerLabel.Text = "⏱️ Hop in: " .. states.currentHop .. "s"
timerLabel.Font = Enum.Font.GothamBold
timerLabel.TextSize = 12
timerLabel.TextColor3 = COLORS.Orange
timerLabel.TextXAlignment = Enum.TextXAlignment.Left
timerLabel.Parent = main

local modeLabel = Instance.new("TextLabel")
modeLabel.Size = UDim2.new(1, -24, 0, 16)
modeLabel.Position = UDim2.new(0, 12, 0, 48)
modeLabel.BackgroundTransparency = 1
modeLabel.Text = "🎯 Modo: Solo Míticas/Legendarias"
modeLabel.Font = Enum.Font.GothamBold
modeLabel.TextSize = 10
modeLabel.TextColor3 = COLORS.Mythic
modeLabel.TextXAlignment = Enum.TextXAlignment.Left
modeLabel.Parent = main

--// CONTENEDOR DE TOGGLES
local toggleContainer = Instance.new("Frame")
toggleContainer.Size = UDim2.new(1, -24, 1, -185)
toggleContainer.Position = UDim2.new(0, 12, 0, 135)
toggleContainer.BackgroundTransparency = 1
toggleContainer.Parent = main

local toggleLayout = Instance.new("UIListLayout")
toggleLayout.Padding = UDim.new(0, 8)
toggleLayout.Parent = toggleContainer

--// FUNCIÓN TOGGLE
local toggleButtons = {}

local function createToggle(name, key, color)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 0, 40)
    frame.BackgroundTransparency = 1
    frame.Parent = toggleContainer
    
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, -80, 1, 0)
    label.BackgroundTransparency = 1
    label.Text = name
    label.Font = Enum.Font.GothamBold
    label.TextSize = 13
    label.TextColor3 = COLORS.Text
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = frame
    
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 60, 0, 30)
    btn.Position = UDim2.new(1, -60, 0.5, -15)
    btn.BackgroundColor3 = COLORS.Gray
    btn.Text = "OFF"
    btn.Font = Enum.Font.GothamBlack
    btn.TextSize = 12
    btn.TextColor3 = COLORS.Text
    btn.AutoButtonColor = false
    btn.Parent = frame
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)
    
    toggleButtons[key] = btn
    
    local function update()
        if config[key] then
            btn.BackgroundColor3 = color
            btn.Text = "ON"
        else
            btn.BackgroundColor3 = COLORS.Gray
            btn.Text = "OFF"
        end
    end
    
    btn.MouseButton1Click:Connect(function()
        config[key] = not config[key]
        update()
        SaveSystem:SaveConfig() -- Guardar automáticamente
    end)
    
    update()
    return btn
end

-- Toggles
createToggle("👑 ESP Mythic Only", "ESPFruitDrop", COLORS.Mythic)
createToggle("🏃 Auto Tween", "AutoTween", COLORS.Orange)
createToggle("🎒 Auto Bag (7s)", "AutoBagFruit", COLORS.Cyan)
createToggle("⚔️ Auto Equip", "AutoEquipFruit", COLORS.Gold)
createToggle("💼 Auto Fruit Bag", "AutoStoreFruit", COLORS.Pink)
createToggle("🌐 Auto Server Hop", "AutoServerHop", COLORS.Red)
createToggle("💼 Hop If Full", "SmartHopIfFull", COLORS.Yellow)

--// BOTONES DE GUARDADO Y RESET
local saveResetContainer = Instance.new("Frame")
saveResetContainer.Size = UDim2.new(1, -24, 0, 40)
saveResetContainer.Position = UDim2.new(0, 12, 0, 470)
saveResetContainer.BackgroundTransparency = 1
saveResetContainer.Parent = main

local saveBtn = Instance.new("TextButton")
saveBtn.Size = UDim2.new(0.48, -5, 1, 0)
saveBtn.Position = UDim2.new(0, 0, 0, 0)
saveBtn.BackgroundColor3 = COLORS.Green
saveBtn.Text = "💾 Guardar"
saveBtn.Font = Enum.Font.GothamBold
saveBtn.TextSize = 12
saveBtn.TextColor3 = COLORS.Text
saveBtn.Parent = saveResetContainer
Instance.new("UICorner", saveBtn).CornerRadius = UDim.new(0, 8)

local resetBtn = Instance.new("TextButton")
resetBtn.Size = UDim2.new(0.48, -5, 1, 0)
resetBtn.Position = UDim2.new(0.52, 0, 0, 0)
resetBtn.BackgroundColor3 = COLORS.Red
resetBtn.Text = "🔄 Reset"
resetBtn.Font = Enum.Font.GothamBold
resetBtn.TextSize = 12
resetBtn.TextColor3 = COLORS.Text
resetBtn.Parent = saveResetContainer
Instance.new("UICorner", resetBtn).CornerRadius = UDim.new(0, 8)

saveBtn.MouseButton1Click:Connect(function()
    SaveSystem:SaveConfig()
    print("💾 Configuración guardada manualmente")
end)

resetBtn.MouseButton1Click:Connect(function()
    SaveSystem:ResetConfig()
    -- Actualizar toggles
    for key, btn in pairs(toggleButtons) do
        if config[key] then
            btn.BackgroundColor3 = key == "ESPFruitDrop" and COLORS.Mythic or
                                 key == "AutoTween" and COLORS.Orange or
                                 key == "AutoBagFruit" and COLORS.Cyan or
                                 key == "AutoEquipFruit" and COLORS.Gold or
                                 key == "AutoStoreFruit" and COLORS.Pink or
                                 key == "AutoServerHop" and COLORS.Red or
                                 COLORS.Yellow
            btn.Text = "ON"
        else
            btn.BackgroundColor3 = COLORS.Gray
            btn.Text = "OFF"
        end
    end
    print("🔄 Configuración reiniciada")
end)

--// BOTÓN FLOTANTE
local floatBtn = Instance.new("TextButton")
floatBtn.Size = UDim2.new(0, 55, 0, 55)
floatBtn.Position = UDim2.new(0, 20, 0.5, -27)
floatBtn.BackgroundColor3 = COLORS.BG
floatBtn.Text = "👑"
floatBtn.Font = Enum.Font.GothamBold
floatBtn.TextSize = 26
floatBtn.Visible = false
floatBtn.Parent = screenGui
Instance.new("UICorner", floatBtn).CornerRadius = UDim.new(1, 0)

--// SISTEMA DE DRAG
local function setupDrag(frame, target)
    target = target or frame
    frame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            states.dragging = true
            states.dragStart = input.Position
            states.startPos = target.Position
        end
    end)
    frame.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            states.dragInput = input
        end
    end)
end

setupDrag(titleBar, main)
setupDrag(floatBtn)

UserInputService.InputChanged:Connect(function(input)
    if states.dragging and input == states.dragInput then
        local delta = input.Position - states.dragStart
        main.Position = UDim2.new(
            states.startPos.X.Scale, 
            states.startPos.X.Offset + delta.X,
            states.startPos.Y.Scale, 
            states.startPos.Y.Offset + delta.Y
        )
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        states.dragging = false
    end
end)

--// BOTONES DE CONTROL
closeBtn.MouseButton1Click:Connect(function()
    screenGui:Destroy()
    if espFolder then espFolder:Destroy() end
    getgenv().AutoServerHopForFruitsLoaded = false
end)

minBtn.MouseButton1Click:Connect(function()
    main.Visible = false
    floatBtn.Visible = true
    floatBtn.Position = UDim2.new(0, main.AbsolutePosition.X + 20, 0, main.AbsolutePosition.Y + 20)
end)

floatBtn.MouseButton1Click:Connect(function()
    floatBtn.Visible = false
    main.Visible = true
end)

--// SISTEMA DE FRUTAS
local fruitCache = {}
local lastUpdate = 0

local FRUIT_FOLDERS = {
    Workspace,
    Workspace:FindFirstChild("Fruits"),
    Workspace:FindFirstChild("SpawnedFruits"),
    Workspace:FindFirstChild("Drops"),
    Workspace:FindFirstChild("Items"),
    Workspace:FindFirstChild("Map"),
    Workspace:FindFirstChild("World"),
    Workspace:FindFirstChild("Entities")
}

local MYTHIC_FRUITS = {
    ["Dragon"] = true, ["Dough"] = true, ["Okuchi"] = true,
    ["Saturn"] = true, ["Darkness"] = true, ["Leopard"] = true
}

local function getFruitRarity(fruitName)
    if MYTHIC_FRUITS[fruitName] then
        return "Mythic", COLORS.Mythic
    else
        return "Legendary", COLORS.Gold
    end
end

local function getFruits()
    if tick() - lastUpdate < config.FruitScanRate then return fruitCache end
    
    fruitCache = {}
    local hrpPos = hrp and hrp.Position
    if not hrpPos then return fruitCache end
    
    local checked = {}
    local function scanContainer(container)
        if checked[container] then return end
        checked[container] = true
        
        for _, obj in pairs(container:GetChildren()) do
            local name = obj.Name
            for _, fruitName in ipairs(FRUIT_NAMES) do
                if name == fruitName then
                    local part = nil
                    local fruitModel = nil
                    
                    if obj:IsA("Model") then
                        fruitModel = obj
                        part = obj.PrimaryPart or obj:FindFirstChild("Handle") or obj:FindFirstChildWhichIsA("BasePart")
                        if not part then
                            for _, child in pairs(obj:GetDescendants()) do
                                if child:IsA("BasePart") then
                                    part = child
                                    break
                                end
                            end
                        end
                    elseif obj:IsA("Tool") then
                        fruitModel = obj
                        part = obj:FindFirstChild("Handle") or obj:FindFirstChildWhichIsA("BasePart")
                    elseif obj:IsA("BasePart") then
                        part = obj
                        fruitModel = obj
                    end
                    
                    if part and part:IsA("BasePart") and part.Parent then
                        local isStored = false
                        local current = part
                        while current do
                            if current == ReplicatedStorage then
                                isStored = true
                                break
                            end
                            current = current.Parent
                        end
                        
                        if not isStored then
                            local rarity, color = getFruitRarity(fruitName)
                            table.insert(fruitCache, {
                                obj = fruitModel,
                                part = part,
                                name = fruitName,
                                rarity = rarity,
                                rarityColor = color,
                                distance = (hrpPos - part.Position).Magnitude,
                                id = fruitModel:GetFullName() .. "_" .. tostring(fruitModel)
                            })
                        end
                    end
                    break
                end
            end
            
            if obj:IsA("Folder") or obj:IsA("Model") then
                scanContainer(obj)
            end
        end
    end
    
    for _, folder in ipairs(FRUIT_FOLDERS) do
        if folder then
            scanContainer(folder)
        end
    end
    
    table.sort(fruitCache, function(a, b)
        if a.rarity == b.rarity then
            return a.distance < b.distance
        end
        return a.rarity == "Mythic"
    end)
    
    lastUpdate = tick()
    return fruitCache
end

--// CHECK INVENTARIO
local function checkInventory()
    states.hasFruit = false
    states.inventoryFull = false
    
    local backpack = player:FindFirstChild("Backpack")
    if backpack then
        for _, item in pairs(backpack:GetChildren()) do
            for _, fruitName in ipairs(FRUIT_NAMES) do
                if item.Name == fruitName then
                    states.hasFruit = true
                    states.inventoryFull = true
                    break
                end
            end
        end
    end
    
    if character then
        for _, obj in pairs(character:GetChildren()) do
            if obj:IsA("Tool") then
                for _, fruitName in ipairs(FRUIT_NAMES) do
                    if obj.Name == fruitName then
                        states.hasFruit = true
                        states.inventoryFull = true
                        break
                    end
                end
            end
        end
    end
    
    local dataFolders = {"Data", "Inventory", "Fruits", "StoredFruits", "PlayerData", "PlayerStats"}
    for _, folderName in ipairs(dataFolders) do
        local folder = player:FindFirstChild(folderName) or (character and character:FindFirstChild(folderName))
        if folder then
            for _, item in pairs(folder:GetChildren()) do
                if table.find(FRUIT_NAMES, item.Name) then
                    states.hasFruit = true
                    states.inventoryFull = true
                    break
                end
            end
        end
    end
    
    if states.hasFruit then
        invStatus.Text = "Inventario: LLENO 👑"
        invStatus.TextColor3 = COLORS.Gold
    else
        invStatus.Text = "Inventario: Vacío"
        invStatus.TextColor3 = COLORS.TextDim
    end
end

--// ESP MEJORADO
local espFolder = Instance.new("Folder")
espFolder.Name = "FruitESP_MythicOnly"
espFolder.Parent = CoreGui

local espCache = {}

local function updateESP()
    if not config.ESPFruitDrop then
        for id, data in pairs(espCache) do
            if data.gui then data.gui:Destroy() end
        end
        espCache = {}
        return
    end
    
    local fruits = getFruits()
    local currentIds = {}
    local hrpPos = hrp and hrp.Position
    
    for _, fruit in ipairs(fruits) do
        currentIds[fruit.id] = true
        
        if not espCache[fruit.id] then
            local bill = Instance.new("BillboardGui")
            bill.Name = "ESP_" .. fruit.name
            bill.Adornee = fruit.part
            bill.Size = UDim2.new(0, 200, 0, 60)
            bill.StudsOffset = Vector3.new(0, 4.5, 0)
            bill.AlwaysOnTop = true
            bill.MaxDistance = 5000
            bill.Parent = espFolder
            
            local bg = Instance.new("Frame")
            bg.Name = "Background"
            bg.Size = UDim2.new(1, 0, 1, 0)
            bg.BackgroundColor3 = COLORS.BG
            bg.BackgroundTransparency = 0.05
            bg.BorderSizePixel = 0
            bg.Parent = bill
            Instance.new("UICorner", bg).CornerRadius = UDim.new(0, 12)
            
            local stroke = Instance.new("UIStroke")
            stroke.Name = "Border"
            stroke.Color = fruit.rarityColor
            stroke.Thickness = 3
            stroke.Parent = bg
            
            if fruit.rarity == "Mythic" then
                local glow = Instance.new("ImageLabel")
                glow.Name = "Glow"
                glow.Size = UDim2.new(1.2, 0, 1.2, 0)
                glow.Position = UDim2.new(-0.1, 0, -0.1, 0)
                glow.BackgroundTransparency = 1
                glow.Image = "rbxassetid://6015897843"
                glow.ImageColor3 = fruit.rarityColor
                glow.ImageTransparency = 0.8
                glow.ZIndex = -1
                glow.Parent = bg
            end
            
            local content = Instance.new("Frame")
            content.Name = "Content"
            content.Size = UDim2.new(1, -12, 1, -12)
            content.Position = UDim2.new(0, 6, 0, 6)
            content.BackgroundTransparency = 1
            content.Parent = bg
            
            local icon = Instance.new("TextLabel")
            icon.Name = "Icon"
            icon.Size = UDim2.new(0, 28, 0, 28)
            icon.BackgroundTransparency = 1
            icon.Text = fruit.rarity == "Mythic" and "🔮" or "⭐"
            icon.Font = Enum.Font.GothamBold
            icon.TextSize = 24
            icon.Parent = content
            
            local nameLabel = Instance.new("TextLabel")
            nameLabel.Name = "FruitName"
            nameLabel.Size = UDim2.new(1, -35, 0, 16)
            nameLabel.Position = UDim2.new(0, 32, 0, 2)
            nameLabel.BackgroundTransparency = 1
            nameLabel.Text = fruit.name
            nameLabel.Font = Enum.Font.GothamBlack
            nameLabel.TextSize = 15
            nameLabel.TextColor3 = fruit.rarityColor
            nameLabel.TextXAlignment = Enum.TextXAlignment.Left
            nameLabel.Parent = content
            
            local rarityLabel = Instance.new("TextLabel")
            rarityLabel.Name = "Rarity"
            rarityLabel.Size = UDim2.new(0, 60, 0, 14)
            rarityLabel.Position = UDim2.new(0, 32, 0, 18)
            rarityLabel.BackgroundColor3 = fruit.rarityColor
            rarityLabel.BackgroundTransparency = 0.8
            rarityLabel.Text = fruit.rarity:upper()
            rarityLabel.Font = Enum.Font.GothamBold
            rarityLabel.TextSize = 8
            rarityLabel.TextColor3 = COLORS.Text
            rarityLabel.Parent = content
            Instance.new("UICorner", rarityLabel).CornerRadius = UDim.new(0, 4)
            
            local distLabel = Instance.new("TextLabel")
            distLabel.Name = "Distance"
            distLabel.Size = UDim2.new(1, 0, 0, 20)
            distLabel.Position = UDim2.new(0, 0, 0, 32)
            distLabel.BackgroundTransparency = 1
            distLabel.Text = math.floor(fruit.distance) .. "m"
            distLabel.Font = Enum.Font.GothamBold
            distLabel.TextSize = 14
            distLabel.TextColor3 = fruit.distance < 50 and COLORS.Green or COLORS.Orange
            distLabel.TextXAlignment = Enum.TextXAlignment.Center
            distLabel.Parent = content
            
            espCache[fruit.id] = {
                gui = bill,
                part = fruit.part,
                distLabel = distLabel,
                stroke = stroke
            }
        else
            local data = espCache[fruit.id]
            if data and data.part and data.part.Parent then
                local newDist = hrpPos and (hrpPos - data.part.Position).Magnitude or fruit.distance
                data.distLabel.Text = math.floor(newDist) .. "m"
                data.distLabel.TextColor3 = newDist < 50 and COLORS.Green or COLORS.Orange
            end
        end
    end
    
    for id, data in pairs(espCache) do
        if not currentIds[id] then
            if data.gui then data.gui:Destroy() end
            espCache[id] = nil
        end
    end
end

--// TWEEN MEJORADO
local function tweenToFruit(fruit)
    if not fruit or not fruit.part or not hrp or states.tweening then return false end
    if not fruit.part.Parent then return false end
    
    states.tweening = true
    
    local startPos = hrp.Position
    local targetPos = fruit.part.Position
    local distance = (startPos - targetPos).Magnitude
    
    if distance < 6 then
        states.tweening = false
        return true
    end
    
    local tweenTime = math.clamp(distance / config.TweenSpeed, 0.5, 6)
    
    print("🚀 [" .. fruit.rarity:upper() .. "] " .. fruit.name .. " (" .. math.floor(distance) .. "m)")
    
    local offset = CFrame.new(0, 3, 0)
    local targetCFrame = CFrame.new(targetPos) * offset
    
    local tweenInfo = TweenInfo.new(tweenTime, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
    local tween = TweenService:Create(hrp, tweenInfo, {CFrame = targetCFrame})
    
    local connection = nil
    local cancelled = false
    
    connection = RunService.Heartbeat:Connect(function()
        if not fruit.part or not fruit.part.Parent then
            cancelled = true
            tween:Cancel()
            if connection then connection:Disconnect() end
        end
    end)
    
    tween:Play()
    tween.Completed:Wait()
    
    if connection then connection:Disconnect() end
    states.tweening = false
    
    if cancelled then
        print("⚠️ Fruta desapareció")
        return false
    end
    
    print("✅ Llegado a " .. fruit.name)
    return true
end

--// AUTO BAG FRUIT - MANTENER PRESIONADO 7 SEGUNDOS
local function autoBagFruit(fruit)
    if not fruit or not fruit.part then return false end
    if states.bagging then return false end
    
    local distance = (hrp.Position - fruit.part.Position).Magnitude
    if distance > config.BagRange then
        return false
    end
    
    states.bagging = true
    states.currentBagFruit = fruit
    
    print("🎒 INICIANDO BAG: " .. fruit.name .. " (7s manteniendo)")
    bagStatus.Text = "🎒 BAGGING: " .. fruit.name .. " [7s]"
    bagStatus.TextColor3 = COLORS.Yellow
    
    local success = false
    
    -- MÉTODO 1: firetouchinterest mantenido
    local touchHeld = false
    pcall(function()
        if firetouchinterest then
            firetouchinterest(hrp, fruit.part, 0)
            touchHeld = true
        end
    end)
    
    -- MÉTODO 2: Tecla E mantenida
    local keyHeld = false
    pcall(function()
        VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.E, false, game)
        keyHeld = true
    end)
    
    -- CONTAR 7 SEGUNDOS
    local startTime = tick()
    while tick() -
