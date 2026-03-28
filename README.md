-- Serviços necessários
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")

-- Aguardar o jogador carregar
local player = Players.LocalPlayer
if not player then
    Players:GetPropertyChangedSignal("LocalPlayer"):Wait()
    player = Players.LocalPlayer
end

-- Aguardar o PlayerGui
local playerGui = player:WaitForChild("PlayerGui")

-- Variáveis globais para controlar os hacks (persistentes)
local aimbotEnabled = false
local espEnabled = false
local speedEnabled = false
local originalWalkSpeed = 16
local currentSpeed = 16
local espObjects = {}
local aimbotConnection = nil

-- Variável para controlar o estado do painel principal
local mainPanelOpen = false
local currentMainGui = nil

-- ==================== FUNÇÃO PARA MOVER PAINÉIS ====================
local function makeDraggable(frame)
    local dragging = false
    local dragInput = nil
    local dragStart = nil
    local startPos = nil
    
    local function update(input)
        local delta = input.Position - dragStart
        local newPosition = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        frame.Position = newPosition
    end
    
    frame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = frame.Position
            
            input.MouseEvent:Connect(function()
                -- Consumir evento
            end)
        end
    end)
    
    frame.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)
    
    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            local delta = input.Position - dragStart
            local newPosition = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
            frame.Position = newPosition
        end
    end)
    
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)
end

-- ==================== FUNÇÃO PARA CENTRALIZAR PAINEL ====================
local function centerPanel(frame, width, height)
    -- Calcula a posição central baseado no tamanho da tela
    frame.Size = UDim2.new(0, width, 0, height)
    frame.Position = UDim2.new(0.5, -width/2, 0.5, -height/2)
    frame.AnchorPoint = Vector2.new(0.5, 0.5)
end

-- ==================== FUNÇÕES DOS HACKS (PERSISTENTES) ====================

-- Função do AIMBOT
local function startAimbot()
    if aimbotConnection then
        aimbotConnection:Disconnect()
        aimbotConnection = nil
    end
    
    if aimbotEnabled then
        aimbotConnection = game:GetService("RunService").RenderStepped:Connect(function()
            local character = player.Character
            if character and character:FindFirstChild("Humanoid") and character:FindFirstChild("HumanoidRootPart") then
                local rootPart = character.HumanoidRootPart
                local closestPlayer = nil
                local closestDistance = math.huge
                
                for _, otherPlayer in ipairs(Players:GetPlayers()) do
                    if otherPlayer ~= player and otherPlayer.Character then
                        local otherRoot = otherPlayer.Character:FindFirstChild("HumanoidRootPart")
                        local otherHumanoid = otherPlayer.Character:FindFirstChild("Humanoid")
                        if otherRoot and otherHumanoid and otherHumanoid.Health > 0 then
                            local distance = (rootPart.Position - otherRoot.Position).Magnitude
                            if distance < closestDistance and distance < 100 then
                                closestDistance = distance
                                closestPlayer = otherPlayer
                            end
                        end
                    end
                end
                
                if closestPlayer and closestPlayer.Character then
                    local targetRoot = closestPlayer.Character:FindFirstChild("HumanoidRootPart")
                    if targetRoot then
                        local camera = workspace.CurrentCamera
                        if camera then
                            camera.CFrame = CFrame.new(camera.CFrame.Position, targetRoot.Position)
                        end
                    end
                end
            end
        end)
    end
end

-- Função do ESP
local function updateESP()
    -- Limpar ESPs antigos
    for _, obj in pairs(espObjects) do
        if obj and obj.Parent then
            obj:Destroy()
        end
    end
    espObjects = {}
    
    if espEnabled then
        for _, otherPlayer in ipairs(Players:GetPlayers()) do
            if otherPlayer ~= player and otherPlayer.Character then
                local humanoid = otherPlayer.Character:FindFirstChild("Humanoid")
                local rootPart = otherPlayer.Character:FindFirstChild("HumanoidRootPart")
                
                if humanoid and humanoid.Health > 0 and rootPart then
                    -- Criar highlight
                    local highlight = Instance.new("Highlight")
                    highlight.Parent = otherPlayer.Character
                    highlight.FillColor = Color3.fromRGB(255, 0, 0)
                    highlight.FillTransparency = 0.5
                    highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                    highlight.OutlineTransparency = 0.3
                    highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
                    table.insert(espObjects, highlight)
                    
                    -- Criar billboard com nome
                    local billboard = Instance.new("BillboardGui")
                    billboard.Name = "ESPName"
                    billboard.Size = UDim2.new(0, 200, 0, 50)
                    billboard.Adornee = rootPart
                    billboard.StudsOffset = Vector3.new(0, 3, 0)
                    billboard.AlwaysOnTop = true
                    billboard.Parent = rootPart
                    
                    local nameLabel = Instance.new("TextLabel")
                    nameLabel.Size = UDim2.new(1, 0, 1, 0)
                    nameLabel.BackgroundTransparency = 1
                    nameLabel.Text = otherPlayer.Name .. " | " .. math.floor(humanoid.Health) .. " HP"
                    nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
                    nameLabel.TextSize = 14
                    nameLabel.Font = Enum.Font.GothamBold
                    nameLabel.TextStrokeTransparency = 0.3
                    nameLabel.Parent = billboard
                    
                    table.insert(espObjects, billboard)
                end
            end
        end
    end
end

-- Função da VELOCIDADE
local function updateSpeed(value)
    local character = player.Character
    if character and character:FindFirstChild("Humanoid") then
        local humanoid = character.Humanoid
        if speedEnabled then
            humanoid.WalkSpeed = value
        else
            humanoid.WalkSpeed = originalWalkSpeed
        end
    end
end

-- Criar ScreenGui do Login
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "LoginPanel"
screenGui.Parent = playerGui
screenGui.ResetOnSpawn = false

-- Frame principal (fundo escuro semi-transparente)
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(1, 0, 1, 0)
mainFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
mainFrame.BackgroundTransparency = 0.6
mainFrame.Parent = screenGui

-- Frame do painel de login
local loginFrame = Instance.new("Frame")
loginFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
loginFrame.BorderSizePixel = 0
loginFrame.Parent = mainFrame

-- Centralizar o painel de login
centerPanel(loginFrame, 400, 500)

-- Barra de título para arrastar
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 40)
titleBar.Position = UDim2.new(0, 0, 0, 0)
titleBar.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
titleBar.BorderSizePixel = 0
titleBar.BackgroundTransparency = 0.8
titleBar.Parent = loginFrame

local titleBarCorners = Instance.new("UICorner")
titleBarCorners.CornerRadius = UDim.new(0, 12)
titleBarCorners.Parent = titleBar

-- Ícone de mover na barra de título
local dragIcon = Instance.new("TextLabel")
dragIcon.Size = UDim2.new(0, 30, 1, 0)
dragIcon.Position = UDim2.new(0, 10, 0, 0)
dragIcon.BackgroundTransparency = 1
dragIcon.Text = "⋮⋮"
dragIcon.TextColor3 = Color3.fromRGB(200, 200, 200)
dragIcon.TextSize = 20
dragIcon.Font = Enum.Font.Gotham
dragIcon.TextXAlignment = Enum.TextXAlignment.Center
dragIcon.Parent = titleBar

local titleText = Instance.new("TextLabel")
titleText.Size = UDim2.new(1, -60, 1, 0)
titleText.Position = UDim2.new(0, 45, 0, 0)
titleText.BackgroundTransparency = 1
titleText.Text = "🔐 PAINEL ROBLOX"
titleText.TextColor3 = Color3.fromRGB(255, 255, 255)
titleText.TextSize = 16
titleText.Font = Enum.Font.GothamBold
titleText.TextXAlignment = Enum.TextXAlignment.Left
titleText.Parent = titleBar

-- Arredondar bordas
local corners = Instance.new("UICorner")
corners.CornerRadius = UDim.new(0, 12)
corners.Parent = loginFrame

-- Subtítulo
local subtitle = Instance.new("TextLabel")
subtitle.Size = UDim2.new(1, 0, 0, 30)
subtitle.Position = UDim2.new(0, 0, 0, 55)
subtitle.BackgroundTransparency = 1
subtitle.Text = "Faça login para acessar o painel"
subtitle.TextColor3 = Color3.fromRGB(180, 180, 180)
subtitle.TextSize = 14
subtitle.Font = Enum.Font.Gotham
subtitle.Parent = loginFrame

-- Frame do campo de usuário
local usernameFrame = Instance.new("Frame")
usernameFrame.Size = UDim2.new(0.8, 0, 0, 50)
usernameFrame.Position = UDim2.new(0.1, 0, 0, 110)
usernameFrame.BackgroundColor3 = Color3.fromRGB(45, 45, 50)
usernameFrame.BorderSizePixel = 0
usernameFrame.Parent = loginFrame

local usernameCorners = Instance.new("UICorner")
usernameCorners.CornerRadius = UDim.new(0, 8)
usernameCorners.Parent = usernameFrame

-- Ícone do usuário
local userIcon = Instance.new("TextLabel")
userIcon.Size = UDim2.new(0, 40, 1, 0)
userIcon.Position = UDim2.new(0, 0, 0, 0)
userIcon.BackgroundTransparency = 1
userIcon.Text = "👤"
userIcon.TextSize = 24
userIcon.TextColor3 = Color3.fromRGB(150, 150, 150)
userIcon.Font = Enum.Font.Gotham
userIcon.Parent = usernameFrame

-- Campo de texto do usuário
local usernameBox = Instance.new("TextBox")
usernameBox.Size = UDim2.new(1, -50, 1, 0)
usernameBox.Position = UDim2.new(0, 45, 0, 0)
usernameBox.BackgroundTransparency = 1
usernameBox.PlaceholderText = "Nome de usuário"
usernameBox.PlaceholderColor3 = Color3.fromRGB(100, 100, 100)
usernameBox.Text = ""
usernameBox.TextColor3 = Color3.fromRGB(255, 255, 255)
usernameBox.TextSize = 16
usernameBox.Font = Enum.Font.Gotham
usernameBox.ClearTextOnFocus = false
usernameBox.Parent = usernameFrame

-- Frame do campo de senha
local passwordFrame = Instance.new("Frame")
passwordFrame.Size = UDim2.new(0.8, 0, 0, 50)
passwordFrame.Position = UDim2.new(0.1, 0, 0, 180)
passwordFrame.BackgroundColor3 = Color3.fromRGB(45, 45, 50)
passwordFrame.BorderSizePixel = 0
passwordFrame.Parent = loginFrame

local passwordCorners = Instance.new("UICorner")
passwordCorners.CornerRadius = UDim.new(0, 8)
passwordCorners.Parent = passwordFrame

-- Ícone da senha
local passIcon = Instance.new("TextLabel")
passIcon.Size = UDim2.new(0, 40, 1, 0)
passIcon.Position = UDim2.new(0, 0, 0, 0)
passIcon.BackgroundTransparency = 1
passIcon.Text = "🔒"
passIcon.TextSize = 24
passIcon.TextColor3 = Color3.fromRGB(150, 150, 150)
passIcon.Font = Enum.Font.Gotham
passIcon.Parent = passwordFrame

-- Campo de texto da senha
local passwordBox = Instance.new("TextBox")
passwordBox.Size = UDim2.new(1, -50, 1, 0)
passwordBox.Position = UDim2.new(0, 45, 0, 0)
passwordBox.BackgroundTransparency = 1
passwordBox.PlaceholderText = "Senha"
passwordBox.PlaceholderColor3 = Color3.fromRGB(100, 100, 100)
passwordBox.Text = ""
passwordBox.TextColor3 = Color3.fromRGB(255, 255, 255)
passwordBox.TextSize = 16
passwordBox.Font = Enum.Font.Gotham
passwordBox.ClearTextOnFocus = false
passwordBox.Parent = passwordFrame

-- Botão de login
local loginButton = Instance.new("TextButton")
loginButton.Size = UDim2.new(0.8, 0, 0, 55)
loginButton.Position = UDim2.new(0.1, 0, 0, 260)
loginButton.BackgroundColor3 = Color3.fromRGB(0, 120, 215)
loginButton.BorderSizePixel = 0
loginButton.Text = "ENTRAR"
loginButton.TextColor3 = Color3.fromRGB(255, 255, 255)
loginButton.TextSize = 18
loginButton.Font = Enum.Font.GothamBold
loginButton.Parent = loginFrame

local buttonCorners = Instance.new("UICorner")
buttonCorners.CornerRadius = UDim.new(0, 8)
buttonCorners.Parent = loginButton

-- Label de erro/mensagem
local messageLabel = Instance.new("TextLabel")
messageLabel.Size = UDim2.new(0.8, 0, 0, 40)
messageLabel.Position = UDim2.new(0.1, 0, 0, 330)
messageLabel.BackgroundTransparency = 1
messageLabel.Text = ""
messageLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
messageLabel.TextSize = 14
messageLabel.Font = Enum.Font.Gotham
messageLabel.TextWrapped = true
messageLabel.Parent = loginFrame

-- Frame de informações (rodapé)
local infoFrame = Instance.new("Frame")
infoFrame.Size = UDim2.new(1, 0, 0, 50)
infoFrame.Position = UDim2.new(0, 0, 1, -50)
infoFrame.BackgroundTransparency = 1
infoFrame.Parent = loginFrame

local infoText = Instance.new("TextLabel")
infoText.Size = UDim2.new(1, 0, 1, 0)
infoText.BackgroundTransparency = 1
infoText.Text = "Credenciais: gdraro / gdraro22 | Arraste a barra superior para mover"
infoText.TextColor3 = Color3.fromRGB(100, 100, 100)
infoText.TextSize = 11
infoText.Font = Enum.Font.Gotham
infoText.Parent = infoFrame

-- Tornar o painel arrastável
makeDraggable(loginFrame)

-- Efeito hover no botão
local function animateButton(button, isHover)
    local tweenInfo = TweenInfo.new(0.1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
    local tween = TweenService:Create(button, tweenInfo, {BackgroundColor3 = isHover and Color3.fromRGB(0, 100, 195) or Color3.fromRGB(0, 120, 215)})
    tween:Play()
end

loginButton.MouseEnter:Connect(function()
    animateButton(loginButton, true)
end)

loginButton.MouseLeave:Connect(function()
    animateButton(loginButton, false)
end)

-- Função para mostrar mensagem
local function showMessage(text, isError)
    messageLabel.Text = text
    messageLabel.TextColor3 = isError and Color3.fromRGB(255, 100, 100) or Color3.fromRGB(100, 255, 100)
    
    messageLabel.TextTransparency = 0
    task.wait(1.5)
    for i = 0, 1, 0.2 do
        messageLabel.TextTransparency = i
        task.wait(0.01)
    end
    messageLabel.Text = ""
end

-- ==================== FUNÇÃO DO PAINEL PRINCIPAL ====================
local function openMainPanel()
    -- Se já existe um painel aberto, não criar outro
    if mainPanelOpen and currentMainGui then
        return
    end
    
    local mainGui = Instance.new("ScreenGui")
    mainGui.Name = "MainPanel"
    mainGui.Parent = playerGui
    mainGui.ResetOnSpawn = false
    currentMainGui = mainGui
    mainPanelOpen = true
    
    -- Fundo escuro
    local background = Instance.new("Frame")
    background.Size = UDim2.new(1, 0, 1, 0)
    background.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    background.BackgroundTransparency = 0.5
    background.Parent = mainGui
    
    -- Frame principal do painel
    local panelFrame = Instance.new("Frame")
    panelFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
    panelFrame.BorderSizePixel = 0
    panelFrame.Parent = background
    
    -- Centralizar o painel principal
    centerPanel(panelFrame, 450, 620)
    
    -- Barra de título para arrastar
    local panelTitleBar = Instance.new("Frame")
    panelTitleBar.Size = UDim2.new(1, 0, 0, 45)
    panelTitleBar.Position = UDim2.new(0, 0, 0, 0)
    panelTitleBar.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
    panelTitleBar.BorderSizePixel = 0
    panelTitleBar.Parent = panelFrame
    
    local panelTitleBarCorners = Instance.new("UICorner")
    panelTitleBarCorners.CornerRadius = UDim.new(0, 12)
    panelTitleBarCorners.Parent = panelTitleBar
    
    -- Ícone de mover na barra de título
    local panelDragIcon = Instance.new("TextLabel")
    panelDragIcon.Size = UDim2.new(0, 30, 1, 0)
    panelDragIcon.Position = UDim2.new(0, 10, 0, 0)
    panelDragIcon.BackgroundTransparency = 1
    panelDragIcon.Text = "⋮⋮"
    panelDragIcon.TextColor3 = Color3.fromRGB(200, 200, 200)
    panelDragIcon.TextSize = 20
    panelDragIcon.Font = Enum.Font.Gotham
    panelDragIcon.TextXAlignment = Enum.TextXAlignment.Center
    panelDragIcon.Parent = panelTitleBar
    
    local panelTitleText = Instance.new("TextLabel")
    panelTitleText.Size = UDim2.new(1, -60, 1, 0)
    panelTitleText.Position = UDim2.new(0, 45, 0, 0)
    panelTitleText.BackgroundTransparency = 1
    panelTitleText.Text = "🎮 PAINEL DE CONTROLE [Pressione L para abrir/fechar]"
    panelTitleText.TextColor3 = Color3.fromRGB(255, 255, 255)
    panelTitleText.TextSize = 14
    panelTitleText.Font = Enum.Font.GothamBold
    panelTitleText.TextXAlignment = Enum.TextXAlignment.Left
    panelTitleText.Parent = panelTitleBar
    
    local panelCorners = Instance.new("UICorner")
    panelCorners.CornerRadius = UDim.new(0, 12)
    panelCorners.Parent = panelFrame
    
    -- Subtítulo com nome do jogador e dica da tecla L
    local welcomeText = Instance.new("TextLabel")
    welcomeText.Size = UDim2.new(1, 0, 0, 40)
    welcomeText.Position = UDim2.new(0, 0, 0, 55)
    welcomeText.BackgroundTransparency = 1
    welcomeText.Text = "Bem-vindo, " .. player.Name .. "! | Pressione L para abrir/fechar"
    welcomeText.TextColor3 = Color3.fromRGB(200, 200, 200)
    welcomeText.TextSize = 12
    welcomeText.Font = Enum.Font.Gotham
    welcomeText.Parent = panelFrame
    
    -- Linha divisória
    local divider = Instance.new("Frame")
    divider.Size = UDim2.new(0.9, 0, 0, 2)
    divider.Position = UDim2.new(0.05, 0, 0, 95)
    divider.BackgroundColor3 = Color3.fromRGB(60, 60, 65)
    divider.BorderSizePixel = 0
    divider.Parent = panelFrame
    
    -- Frame para as opções
    local optionsFrame = Instance.new("Frame")
    optionsFrame.Size = UDim2.new(0.9, 0, 0, 400)
    optionsFrame.Position = UDim2.new(0.05, 0, 0, 110)
    optionsFrame.BackgroundTransparency = 1
    optionsFrame.Parent = panelFrame
    
    -- ==================== OPÇÃO 1: AIMBOT ====================
    local aimbotFrame = Instance.new("Frame")
    aimbotFrame.Size = UDim2.new(1, 0, 0, 100)
    aimbotFrame.Position = UDim2.new(0, 0, 0, 0)
    aimbotFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
    aimbotFrame.BorderSizePixel = 0
    aimbotFrame.Parent = optionsFrame
    
    local aimbotCorners = Instance.new("UICorner")
    aimbotCorners.CornerRadius = UDim.new(0, 8)
    aimbotCorners.Parent = aimbotFrame
    
    local aimbotTitle = Instance.new("TextLabel")
    aimbotTitle.Size = UDim2.new(0.6, 0, 0, 30)
    aimbotTitle.Position = UDim2.new(0.02, 0, 0, 12)
    aimbotTitle.BackgroundTransparency = 1
    aimbotTitle.Text = "🎯 AIMBOT"
    aimbotTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    aimbotTitle.TextSize = 18
    aimbotTitle.Font = Enum.Font.GothamBold
    aimbotTitle.TextXAlignment = Enum.TextXAlignment.Left
    aimbotTitle.Parent = aimbotFrame
    
    local aimbotDesc = Instance.new("TextLabel")
    aimbotDesc.Size = UDim2.new(0.6, 0, 0, 45)
    aimbotDesc.Position = UDim2.new(0.02, 0, 0, 45)
    aimbotDesc.BackgroundTransparency = 1
    aimbotDesc.Text = "Mirar automaticamente nos inimigos mais próximos"
    aimbotDesc.TextColor3 = Color3.fromRGB(150, 150, 150)
    aimbotDesc.TextSize = 12
    aimbotDesc.Font = Enum.Font.Gotham
    aimbotDesc.TextXAlignment = Enum.TextXAlignment.Left
    aimbotDesc.TextWrapped = true
    aimbotDesc.Parent = aimbotFrame
    
    local aimbotToggle = Instance.new("TextButton")
    aimbotToggle.Size = UDim2.new(0, 100, 0, 45)
    aimbotToggle.Position = UDim2.new(1, -110, 0.5, -22)
    aimbotToggle.BackgroundColor3 = aimbotEnabled and Color3.fromRGB(0, 150, 100) or Color3.fromRGB(60, 60, 65)
    aimbotToggle.Text = aimbotEnabled and "ATIVADO" or "DESATIVADO"
    aimbotToggle.TextColor3 = aimbotEnabled and Color3.fromRGB(255, 255, 255) or Color3.fromRGB(255, 100, 100)
    aimbotToggle.TextSize = 14
    aimbotToggle.Font = Enum.Font.GothamBold
    aimbotToggle.Parent = aimbotFrame
    
    local aimbotToggleCorners = Instance.new("UICorner")
    aimbotToggleCorners.CornerRadius = UDim.new(0, 6)
    aimbotToggleCorners.Parent = aimbotToggle
    
    -- ==================== OPÇÃO 2: ESP ====================
    local espFrame = Instance.new("Frame")
    espFrame.Size = UDim2.new(1, 0, 0, 100)
    espFrame.Position = UDim2.new(0, 0, 0, 110)
    espFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
    espFrame.BorderSizePixel = 0
    espFrame.Parent = optionsFrame
    
    local espCorners = Instance.new("UICorner")
    espCorners.CornerRadius = UDim.new(0, 8)
    espCorners.Parent = espFrame
    
    local espTitle = Instance.new("TextLabel")
    espTitle.Size = UDim2.new(0.6, 0, 0, 30)
    espTitle.Position = UDim2.new(0.02, 0, 0, 12)
    espTitle.BackgroundTransparency = 1
    espTitle.Text = "👁️ ESP (Wallhack)"
    espTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    espTitle.TextSize = 18
    espTitle.Font = Enum.Font.GothamBold
    espTitle.TextXAlignment = Enum.TextXAlignment.Left
    espTitle.Parent = espFrame
    
    local espDesc = Instance.new("TextLabel")
    espDesc.Size = UDim2.new(0.6, 0, 0, 45)
    espDesc.Position = UDim2.new(0.02, 0, 0, 45)
    espDesc.BackgroundTransparency = 1
    espDesc.Text = "Ver jogadores através das paredes com nome e destaque"
    espDesc.TextColor3 = Color3.fromRGB(150, 150, 150)
    espDesc.TextSize = 12
    espDesc.Font = Enum.Font.Gotham
    espDesc.TextXAlignment = Enum.TextXAlignment.Left
    espDesc.TextWrapped = true
    espDesc.Parent = espFrame
    
    local espToggle = Instance.new("TextButton")
    espToggle.Size = UDim2.new(0, 100, 0, 45)
    espToggle.Position = UDim2.new(1, -110, 0.5, -22)
    espToggle.BackgroundColor3 = espEnabled and Color3.fromRGB(0, 150, 100) or Color3.fromRGB(60, 60, 65)
    espToggle.Text = espEnabled and "ATIVADO" or "DESATIVADO"
    espToggle.TextColor3 = espEnabled and Color3.fromRGB(255, 255, 255) or Color3.fromRGB(255, 100, 100)
    espToggle.TextSize = 14
    espToggle.Font = Enum.Font.GothamBold
    espToggle.Parent = espFrame
    
    local espToggleCorners = Instance.new("UICorner")
    espToggleCorners.CornerRadius = UDim.new(0, 6)
    espToggleCorners.Parent = espToggle
    
    -- ==================== OPÇÃO 3: VELOCIDADE ====================
    local speedFrame = Instance.new("Frame")
    speedFrame.Size = UDim2.new(1, 0, 0, 120)
    speedFrame.Position = UDim2.new(0, 0, 0, 220)
    speedFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
    speedFrame.BorderSizePixel = 0
    speedFrame.Parent = optionsFrame
    
    local speedCorners = Instance.new("UICorner")
    speedCorners.CornerRadius = UDim.new(0, 8)
    speedCorners.Parent = speedFrame
    
    local speedTitle = Instance.new("TextLabel")
    speedTitle.Size = UDim2.new(0.6, 0, 0, 30)
    speedTitle.Position = UDim2.new(0.02, 0, 0, 12)
    speedTitle.BackgroundTransparency = 1
    speedTitle.Text = "⚡ VELOCIDADE"
    speedTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    speedTitle.TextSize = 18
    speedTitle.Font = Enum.Font.GothamBold
    speedTitle.TextXAlignment = Enum.TextXAlignment.Left
    speedTitle.Parent = speedFrame
    
    local speedDesc = Instance.new("TextLabel")
    speedDesc.Size = UDim2.new(0.6, 0, 0, 45)
    speedDesc.Position = UDim2.new(0.02, 0, 0, 45)
    speedDesc.BackgroundTransparency = 1
    speedDesc.Text = "Aumentar velocidade do personagem (16 a 100)"
    speedDesc.TextColor3 = Color3.fromRGB(150, 150, 150)
    speedDesc.TextSize = 12
    speedDesc.Font = Enum.Font.Gotham
    speedDesc.TextXAlignment = Enum.TextXAlignment.Left
    speedDesc.TextWrapped = true
    speedDesc.Parent = speedFrame
    
    local speedToggle = Instance.new("TextButton")
    speedToggle.Size = UDim2.new(0, 100, 0, 45)
    speedToggle.Position = UDim2.new(1, -110, 0.3, -22)
    speedToggle.BackgroundColor3 = speedEnabled and Color3.fromRGB(0, 150, 100) or Color3.fromRGB(60, 60, 65)
    speedToggle.Text = speedEnabled and "ATIVADO" or "DESATIVADO"
    speedToggle.TextColor3 = speedEnabled and Color3.fromRGB(255, 255, 255) or Color3.fromRGB(255, 100, 100)
    speedToggle.TextSize = 14
    speedToggle.Font = Enum.Font.GothamBold
    speedToggle.Parent = speedFrame
    
    local speedToggleCorners = Instance.new("UICorner")
    speedToggleCorners.CornerRadius = UDim.new(0, 6)
    speedToggleCorners.Parent = speedToggle
    
    -- Slider para controlar a velocidade
    local speedValue = Instance.new("TextLabel")
    speedValue.Size = UDim2.new(0.3, 0, 0, 25)
    speedValue.Position = UDim2.new(0.65, 0, 0.7, 0)
    speedValue.BackgroundTransparency = 1
    speedValue.Text = tostring(currentSpeed)
    speedValue.TextColor3 = Color3.fromRGB(255, 200, 100)
    speedValue.TextSize = 14
    speedValue.Font = Enum.Font.GothamBold
    speedValue.TextXAlignment = Enum.TextXAlignment.Right
    speedValue.Parent = speedFrame
    
    local speedSlider = Instance.new("Frame")
    speedSlider.Size = UDim2.new(0.3, 0, 0, 4)
    speedSlider.Position = UDim2.new(0.65, 0, 0.85, 0)
    speedSlider.BackgroundColor3 = Color3.fromRGB(80, 80, 85)
    speedSlider.BorderSizePixel = 0
    speedSlider.Parent = speedFrame
    
    local speedSliderFill = Instance.new("Frame")
    local percent = (currentSpeed - 16) / 84
    speedSliderFill.Size = UDim2.new(percent, 0, 1, 0)
    speedSliderFill.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
    speedSliderFill.BorderSizePixel = 0
    speedSliderFill.Parent = speedSlider
    
    local speedSliderButton = Instance.new("TextButton")
    speedSliderButton.Size = UDim2.new(0, 15, 0, 15)
    speedSliderButton.Position = UDim2.new(percent, -7, 0.5, -7)
    speedSliderButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    speedSliderButton.BorderSizePixel = 0
    speedSliderButton.Text = ""
    speedSliderButton.Parent = speedSlider
    
    local sliderCorners = Instance.new("UICorner")
    sliderCorners.CornerRadius = UDim.new(1, 0)
    sliderCorners.Parent = speedSliderButton
    
    -- Botão de fechar
    local closeButton = Instance.new("TextButton")
    closeButton.Size = UDim2.new(0.8, 0, 0, 45)
    closeButton.Position = UDim2.new(0.1, 0, 1, -15)
    closeButton.BackgroundColor3 = Color3.fromRGB(200, 60, 60)
    closeButton.Text = "FECHAR PAINEL"
    closeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeButton.TextSize = 16
    closeButton.Font = Enum.Font.GothamBold
    closeButton.Parent = panelFrame
    
    local closeCorners = Instance.new("UICorner")
    closeCorners.CornerRadius = UDim.new(0, 8)
    closeCorners.Parent = closeButton
    
    -- Tornar o painel principal arrastável
    makeDraggable(panelFrame)
    
    -- ==================== CONTROLE DO SLIDER ====================
    local isDragging = false
    
    local function setSpeedValue(value)
        currentSpeed = math.clamp(value, 16, 100)
        speedValue.Text = tostring(math.floor(currentSpeed))
        local newPercent = (currentSpeed - 16) / 84
        speedSliderFill.Size = UDim2.new(newPercent, 0, 1, 0)
        speedSliderButton.Position = UDim2.new(newPercent, -7, 0.5, -7)
        
        if speedEnabled then
            updateSpeed(currentSpeed)
        end
    end
    
    speedSliderButton.MouseButton1Down:Connect(function()
        isDragging = true
        local mouse = player:GetMouse()
        
        local connection
        connection = mouse.Move:Connect(function()
            if isDragging then
                local relativeX = math.clamp((mouse.X - speedSlider.AbsolutePosition.X) / speedSlider.AbsoluteSize.X, 0, 1)
                local newSpeed = 16 + (relativeX * 84)
                setSpeedValue(newSpeed)
            end
        end)
        
        local releaseConnection
        releaseConnection = mouse.Button1Up:Connect(function()
            isDragging = false
            connection:Disconnect()
            releaseConnection:Disconnect()
        end)
    end)
    
    -- ==================== TOGGLES DOS BOTÕES ====================
    
    aimbotToggle.MouseButton1Click:Connect(function()
        aimbotEnabled = not aimbotEnabled
        if aimbotEnabled then
            aimbotToggle.Text = "ATIVADO"
            aimbotToggle.BackgroundColor3 = Color3.fromRGB(0, 150, 100)
            aimbotToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
        else
            aimbotToggle.Text = "DESATIVADO"
            aimbotToggle.BackgroundColor3 = Color3.fromRGB(60, 60, 65)
            aimbotToggle.TextColor3 = Color3.fromRGB(255, 100, 100)
        end
        startAimbot()
    end)
    
    espToggle.MouseButton1Click:Connect(function()
        espEnabled = not espEnabled
        if espEnabled then
            espToggle.Text = "ATIVADO"
            espToggle.BackgroundColor3 = Color3.fromRGB(0, 150, 100)
            espToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
        else
            espToggle.Text = "DESATIVADO"
            espToggle.BackgroundColor3 = Color3.fromRGB(60, 60, 65)
            espToggle.TextColor3 = Color3.fromRGB(255, 100, 100)
        end
        updateESP()
    end)
    
    speedToggle.MouseButton1Click:Connect(function()
        speedEnabled = not speedEnabled
        if speedEnabled then
            speedToggle.Text = "ATIVADO"
            speedToggle.BackgroundColor3 = Color3.fromRGB(0, 150, 100)
            speedToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
        else
            speedToggle.Text = "DESATIVADO"
            speedToggle.BackgroundColor3 = Color3.fromRGB(60, 60, 65)
            speedToggle.TextColor3 = Color3.fromRGB(255, 100, 100)
        end
        updateSpeed(currentSpeed)
    end)
    
    -- Monitorar quando o personagem respawna
    player.CharacterAdded:Connect(function(character)
        task.wait(0.5)
        if speedEnabled then
            updateSpeed(currentSpeed)
        end
    end)
    
    -- Monitorar quando jogadores entram/saem para atualizar ESP
    Players.PlayerAdded:Connect(function()
        if espEnabled then
            updateESP()
        end
    end)
    
    Players.PlayerRemoving:Connect(function()
        if espEnabled then
            updateESP()
        end
    end)
    
    -- Função para fechar APENAS o painel (NÃO desativa os hacks)
    local function closePanelOnly()
        mainGui:Destroy()
        mainPanelOpen = false
        currentMainGui = nil
    end
    
    closeButton.MouseButton1Click:Connect(function()
        closePanelOnly()
    end)
    
    -- ==================== TECLA L PARA ABRIR/FECHAR ====================
    local function onKeyPress(input)
        if input.KeyCode == Enum.KeyCode.L then
            if mainPanelOpen and currentMainGui then
                closePanelOnly()
            else
                openMainPanel()
            end
        end
    end
    
    UserInputService.InputBegan:Connect(onKeyPress)
    
    print("✅ Painel principal carregado! Pressione L para abrir/fechar | Hacks continuam ativos mesmo com painel fechado")
end

-- Função de login
local function attemptLogin()
    local username = usernameBox.Text:lower()
    local password = passwordBox.Text
    
    username = username:gsub("%s+", "")
    
    if username == "" or password == "" then
        showMessage("❌ Preencha todos os campos!", true)
        return
    end
    
    if username == "gdraro" and password == "gdraro22" then
        showMessage("✅ Login realizado com sucesso!", false)
        
        local successTween = TweenService:Create(loginButton, TweenInfo.new(0.15), {BackgroundColor3 = Color3.fromRGB(0, 200, 100)})
        successTween:Play()
        
        task.wait(0.2)
        
        screenGui:Destroy()
        openMainPanel()
    else
        showMessage("❌ Nome ou senha incorretos!", true)
        
        local originalColor = loginButton.BackgroundColor3
        for i = 1, 3 do
            loginButton.BackgroundColor3 = Color3.fromRGB(255, 80, 80)
            task.wait(0.03)
            loginButton.BackgroundColor3 = originalColor
            task.wait(0.03)
        end
        
        passwordBox.Text = ""
        passwordBox:CaptureFocus()
    end
end

loginButton.MouseButton1Click:Connect(attemptLogin)

local function onEnterPressed(input, box)
    if input.KeyCode == Enum.KeyCode.Return or input.KeyCode == Enum.KeyCode.KeypadEnter then
        if box == usernameBox or box == passwordBox then
            attemptLogin()
        end
    end
end

usernameBox.InputBegan:Connect(function(input)
    onEnterPressed(input, usernameBox)
end)

passwordBox.InputBegan:Connect(function(input)
    onEnterPressed(input, passwordBox)
end)

local function addFocusEffect(frame, box)
    local originalColor = frame.BackgroundColor3
    box.Focused:Connect(function()
        frame.BackgroundColor3 = Color3.fromRGB(55, 55, 65)
    end)
    box.FocusLost:Connect(function()
        frame.BackgroundColor3 = originalColor
    end)
end

addFocusEffect(usernameFrame, usernameBox)
addFocusEffect(passwordFrame, passwordBox)

print("✅ Painel de login carregado! Credenciais: gdraro / gdraro22 | Painel centralizado na tela")
