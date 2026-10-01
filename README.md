-- Auto-Stop Spin para Asesinos 2 (Slayers 2)
-- Sin comandos de chat - Usa GUI y teclas

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Configuración
local CLANES_RAROS = {
    -- Supremo (0.1%)
    "Kamado", "Rengoku", "Soyama", "Uzui",
    -- Mítico (0.9%)
    "Agatsuma", "Douma", "Himejima", "Iguro", "Shinazugawa", "Tamayo"
}

local autoSpinActivo = false
local connectionSpin = nil
local connectionDeteccion = nil
local guiVisible = true

-- Crear GUI principal
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AutoSpinGUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = playerGui

-- Frame principal (arrastrable)
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 300, 0, 180)
mainFrame.Position = UDim2.new(0.5, -150, 0.1, 0)
mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
mainFrame.BorderSizePixel = 0
mainFrame.Parent = screenGui

-- Esquinas redondeadas
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 12)
corner.Parent = mainFrame

-- Título
local titulo = Instance.new("TextLabel")
titulo.Name = "Titulo"
titulo.Size = UDim2.new(1, 0, 0, 35)
titulo.BackgroundColor3 = Color3.fromRGB(255, 0, 128)
titulo.Text = "🎯 Auto-Stop Spin"
titulo.TextColor3 = Color3.fromRGB(255, 255, 255)
titulo.TextScaled = true
titulo.Font = Enum.Font.GothamBold
titulo.Parent = mainFrame

local tituloCorner = Instance.new("UICorner")
tituloCorner.CornerRadius = UDim.new(0, 12)
tituloCorner.Parent = titulo

-- Botón Iniciar/Detener
local btnToggle = Instance.new("TextButton")
btnToggle.Name = "ToggleBtn"
btnToggle.Size = UDim2.new(0.9, 0, 0, 45)
btnToggle.Position = UDim2.new(0.05, 0, 0.3, 0)
btnToggle.BackgroundColor3 = Color3.fromRGB(0, 170, 0)
btnToggle.Text = "▶ INICIAR AUTO-SPIN"
btnToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
btnToggle.TextScaled = true
btnToggle.Font = Enum.Font.GothamBold
btnToggle.Parent = mainFrame

local btnCorner = Instance.new("UICorner")
btnCorner.CornerRadius = UDim.new(0, 8)
btnCorner.Parent = btnToggle

-- Estado
local estado = Instance.new("TextLabel")
estado.Name = "Estado"
estado.Size = UDim2.new(1, 0, 0, 25)
estado.Position = UDim2.new(0, 0, 0.65, 0)
estado.BackgroundTransparency = 1
estado.Text = "Estado: Esperando..."
estado.TextColor3 = Color3.fromRGB(200, 200, 200)
estado.TextScaled = true
estado.Font = Enum.Font.Gotham
estado.Parent = mainFrame

-- Info de teclas
local info = Instance.new("TextLabel")
info.Name = "Info"
info.Size = UDim2.new(1, 0, 0, 20)
info.Position = UDim2.new(0, 0, 0.85, 0)
info.BackgroundTransparency = 1
info.Text = "Presiona INSERT para ocultar/mostrar"
info.TextColor3 = Color3.fromRGB(150, 150, 150)
info.TextScaled = true
info.Font = Enum.Font.Gotham
info.Parent = mainFrame

-- Hacer arrastrable
local arrastrando = false
local inicioArrastre
local posicionInicial

titulo.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        arrastrando = true
        inicioArrastre = input.Position
        posicionInicial = mainFrame.Position
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if arrastrando and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - inicioArrastre
        mainFrame.Position = UDim2.new(
            posicionInicial.X.Scale, 
            posicionInicial.X.Offset + delta.X,
            posicionInicial.Y.Scale, 
            posicionInicial.Y.Offset + delta.Y
        )
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        arrastrando = false
    end
end)

-- Función para verificar si es clan raro
local function esClanRaro(nombreClan)
    if not nombreClan then return false end
    nombreClan = tostring(nombreClan)
    for _, clan in ipairs(CLANES_RAROS) do
        if string.find(nombreClan, clan, 1, true) then
            return true, clan
        end
    end
    return false
end

-- Función para detener y alertar
local function detenerYGanar(nombreClan)
    if not autoSpinActivo then return end
    
    autoSpinActivo = false
    if connectionSpin then
        connectionSpin:Disconnect()
        connectionSpin = nil
    end
    
    btnToggle.BackgroundColor3 = Color3.fromRGB(0, 170, 0)
    btnToggle.Text = "▶ INICIAR AUTO-SPIN"
    estado.Text = "Estado: ¡CLAN RARO!"
    estado.TextColor3 = Color3.fromRGB(255, 215, 0)
    
    -- Alerta grande
    local alerta = Instance.new("ScreenGui")
    alerta.Name = "RareAlert"
    alerta.Parent = playerGui
    
    local fondo = Instance.new("Frame")
    fondo.Size = UDim2.new(1, 0, 1, 0)
    fondo.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    fondo.BackgroundTransparency = 0.3
    fondo.Parent = alerta
    
    local panel = Instance.new("Frame")
    panel.Size = UDim2.new(0, 450, 0, 200)
    panel.Position = UDim2.new(0.5, -225, 0.5, -100)
    panel.BackgroundColor3 = Color3.fromRGB(255, 0, 128)
    panel.BorderSizePixel = 0
    panel.Parent = alerta
    
    local panelCorner = Instance.new("UICorner")
    panelCorner.CornerRadius = UDim.new(0, 20)
    panelCorner.Parent = panel
    
    local texto = Instance.new("TextLabel")
    texto.Size = UDim2.new(1, -40, 1, -40)
    texto.Position = UDim2.new(0, 20, 0, 20)
    texto.BackgroundTransparency = 1
    texto.Text = "🎉 ¡FELICIDADES! 🎉\n\nHas obtenido:\n" .. nombreClan .. "\n\nGiro detenido automáticamente"
    texto.TextColor3 = Color3.fromRGB(255, 255, 255)
    texto.TextScaled = true
    texto.Font = Enum.Font.GothamBold
    texto.Parent = panel
    
    -- Sonido
    local sonido = Instance.new("Sound")
    sonido.SoundId = "rbxassetid://9113083740"
    sonido.Volume = 1
    sonido.Parent = alerta
    sonido:Play()
    
    -- Animación
    panel.Size = UDim2.new(0, 0, 0, 0)
    TweenService:Create(panel, TweenInfo.new(0.5, Enum.EasingStyle.Back), {
        Size = UDim2.new(0, 450, 0, 200)
    }):Play()
    
    -- Cerrar al hacer clic
    task.delay(0.5, function()
        fondo.InputBegan:Connect(function()
            TweenService:Create(panel, TweenInfo.new(0.3), {
                Size = UDim2.new(0, 0, 0, 0)
            }):Play()
            task.wait(0.3)
            alerta:Destroy()
        end)
    end)
    
    print("¡CLAN RARO DETECTADO: " .. nombreClan .. "!")
end

-- Función para hacer clic en el botón de giro
local function hacerClickGiro()
    -- Buscar el botón de giro en las interfaces del juego
    for _, gui in ipairs(playerGui:GetChildren()) do
        if gui:IsA("ScreenGui") then
            for _, btn in ipairs(gui:GetDescendants()) do
                if btn:IsA("TextButton") or btn:IsA("ImageButton") then
                    local texto = btn.Text or ""
                    -- Detectar botones de giro comunes
                    if string.find(texto:lower(), "roll") or 
                       string.find(texto:lower(), "giro") or
                       string.find(texto:lower(), "spin") or
                       string.find(texto, "1 Giro") or
                       string.find(texto, "10 Giros") then
                        
                        -- Simular clic
                        pcall(function()
                            btn.MouseButton1Click:Fire()
                        end)
                        return true
                    end
                end
            end
        end
    end
    return false
end

-- Detección de clanes raros
local function iniciarDeteccion()
    connectionDeteccion = RunService.Heartbeat:Connect(function()
        for _, gui in ipairs(playerGui:GetChildren()) do
            if gui:IsA("ScreenGui") and gui.Name ~= "AutoSpinGUI" and gui.Name ~= "RareAlert" then
                for _, elemento in ipairs(gui:GetDescendants()) do
                    if elemento:IsA("TextLabel") or elemento:IsA("TextButton") then
                        local texto = elemento.Text
                        local esRaro, nombre = esClanRaro(texto)
                        
                        if esRaro then
                            -- Verificar si es un resultado de giro (tamaño grande o color especial)
                            local esResultado = false
                            
                            -- Detectar por color de texto (oro, rosa, púrpura)
                            if elemento.TextColor3 then
                                local color = elemento.TextColor3
                                if color.R > 0.8 or (color.R > 0.6 and color.G > 0.5) then
                                    esResultado = true
                                end
                            end
                            
                            -- Detectar por tamaño de fuente grande
                            if elemento.TextSize and elemento.TextSize >= 18 then
                                esResultado = true
                            end
                            
                            if esResultado then
                                detenerYGanar(nombre or texto)
                                return
                            end
                        end
                    end
                end
            end
        end
    end)
end

-- Toggle Auto-Spin
local function toggleAutoSpin()
    autoSpinActivo = not autoSpinActivo
    
    if autoSpinActivo then
        btnToggle.BackgroundColor3 = Color3.fromRGB(170, 0, 0)
        btnToggle.Text = "⏹ DETENER AUTO-SPIN"
        estado.Text = "Estado: Girando..."
        estado.TextColor3 = Color3.fromRGB(0, 255, 0)
        
        -- Iniciar bucle de giro
        connectionSpin = RunService.Heartbeat:Connect(function()
            if autoSpinActivo then
                hacerClickGiro()
                task.wait(0.5) -- Delay entre giros
            end
        end)
    else
        btnToggle.BackgroundColor3 = Color3.fromRGB(0, 170, 0)
        btnToggle.Text = "▶ INICIAR AUTO-SPIN"
        estado.Text = "Estado: Detenido"
        estado.TextColor3 = Color3.fromRGB(200, 200, 200)
        
        if connectionSpin then
            connectionSpin:Disconnect()
            connectionSpin = nil
        end
    end
end

-- Conectar botón
btnToggle.MouseButton1Click:Connect(toggleAutoSpin)

-- Tecla INSERT para mostrar/ocultar
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if input.KeyCode == Enum.KeyCode.Insert then
        guiVisible = not guiVisible
        mainFrame.Visible = guiVisible
    end
    
    -- Tecla END para detener emergencia
    if input.KeyCode == Enum.KeyCode.End then
        if autoSpinActivo then
            toggleAutoSpin()
        end
    end
end)

-- Iniciar detección automática
iniciarDeteccion()

-- Animación de entrada
mainFrame.Position = UDim2.new(0.5, -150, 0, -200)
TweenService:Create(mainFrame, TweenInfo.new(0.5, Enum.EasingStyle.Back), {
    Position = UDim2.new(0.5, -150, 0.1, 0)
}):Play()

print("✅ Auto-Stop Spin cargado!")
print("🖱️  Clic en el botón para iniciar")
print("⌨️  INSERT: Ocultar/Mostrar GUI")
print("⌨️  END: Detener emergencia")
