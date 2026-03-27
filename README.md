-- Painel de Velocidade para Roblox
-- Insira este script em um ScreenGui ou em um LocalScript

local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")

-- Criando a interface
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "PainelVelocidade"
screenGui.Parent = player.PlayerGui

-- Frame principal (painel)
local painel = Instance.new("Frame")
painel.Size = UDim2.new(0, 300, 0, 400)
painel.Position = UDim2.new(0.5, -150, 0.5, -200)
painel.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
painel.BackgroundTransparency = 0.1
painel.BorderSizePixel = 0
painel.Active = true
painel.Draggable = true
painel.Parent = screenGui

-- Cantos arredondados
local corners = Instance.new("UICorner")
corners.CornerRadius = UDim.new(0, 15)
corners.Parent = painel

-- Sombra
local shadow = Instance.new("UIStroke")
shadow.Thickness = 2
shadow.Color = Color3.fromRGB(100, 100, 150)
shadow.Transparency = 0.5
shadow.Parent = painel

-- Título
local titulo = Instance.new("TextLabel")
titulo.Size = UDim2.new(1, 0, 0, 50)
titulo.Position = UDim2.new(0, 0, 0, 0)
titulo.BackgroundTransparency = 1
titulo.Text = "🌙 PAINEL LUA 🌙"
titulo.TextColor3 = Color3.fromRGB(255, 200, 100)
titulo.TextScaled = true
titulo.Font = Enum.Font.GothamBold
titulo.Parent = painel

-- Subtítulo
local subtitulo = Instance.new("TextLabel")
subtitulo.Size = UDim2.new(1, 0, 0, 30)
subtitulo.Position = UDim2.new(0, 0, 0, 45)
subtitulo.BackgroundTransparency = 1
subtitulo.Text = "Controle de Velocidade"
subtitulo.TextColor3 = Color3.fromRGB(200, 200, 220)
subtitulo.TextScaled = true
subtitulo.Font = Enum.Font.Gotham
subtitulo.Parent = painel

-- Frame da velocidade atual
local frameVelocidade = Instance.new("Frame")
frameVelocidade.Size = UDim2.new(0.8, 0, 0, 60)
frameVelocidade.Position = UDim2.new(0.1, 0, 0, 90)
frameVelocidade.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
frameVelocidade.BackgroundTransparency = 0.3
frameVelocidade.BorderSizePixel = 0
frameVelocidade.Parent = painel

local corners2 = Instance.new("UICorner")
corners2.CornerRadius = UDim.new(0, 10)
corners2.Parent = frameVelocidade

-- Texto da velocidade atual
local velAtual = Instance.new("TextLabel")
velAtual.Size = UDim2.new(1, 0, 0.5, 0)
velAtual.Position = UDim2.new(0, 0, 0, 0)
velAtual.BackgroundTransparency = 1
velAtual.Text = "Velocidade Atual:"
velAtual.TextColor3 = Color3.fromRGB(200, 200, 220)
velAtual.TextScaled = true
velAtual.Font = Enum.Font.Gotham
velAtual.Parent = frameVelocidade

local velValor = Instance.new("TextLabel")
velValor.Size = UDim2.new(1, 0, 0.5, 0)
velValor.Position = UDim2.new(0, 0, 0.5, 0)
velValor.BackgroundTransparency = 1
velValor.Text = "16"
velValor.TextColor3 = Color3.fromRGB(255, 200, 100)
velValor.TextScaled = true
velValor.Font = Enum.Font.GothamBold
velValor.Parent = frameVelocidade

-- Slider (controle deslizante)
local sliderFrame = Instance.new("Frame")
sliderFrame.Size = UDim2.new(0.8, 0, 0, 40)
sliderFrame.Position = UDim2.new(0.1, 0, 0, 165)
sliderFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
sliderFrame.BackgroundTransparency = 0.3
sliderFrame.BorderSizePixel = 0
sliderFrame.Parent = painel

local corners3 = Instance.new("UICorner")
corners3.CornerRadius = UDim.new(0, 20)
corners3.Parent = sliderFrame

local sliderBar = Instance.new("Frame")
sliderBar.Size = UDim2.new(0.9, 0, 0, 6)
sliderBar.Position = UDim2.new(0.05, 0, 0.5, -3)
sliderBar.BackgroundColor3 = Color3.fromRGB(80, 80, 100)
sliderBar.BorderSizePixel = 0
sliderBar.Parent = sliderFrame

local corners4 = Instance.new("UICorner")
corners4.CornerRadius = UDim.new(1, 0)
corners4.Parent = sliderBar

local sliderFill = Instance.new("Frame")
sliderFill.Size = UDim2.new(0.5, 0, 1, 0)
sliderFill.BackgroundColor3 = Color3.fromRGB(255, 200, 100)
sliderFill.BorderSizePixel = 0
sliderFill.Parent = sliderBar

local corners5 = Instance.new("UICorner")
corners5.CornerRadius = UDim.new(1, 0)
corners5.Parent = sliderFill

local knob = Instance.new("Frame")
knob.Size = UDim2.new(0, 20, 0, 20)
knob.Position = UDim2.new(0.5, -10, 0.5, -10)
knob.BackgroundColor3 = Color3.fromRGB(255, 220, 150)
knob.BorderSizePixel = 0
knob.Parent = sliderFrame

local corners6 = Instance.new("UICorner")
corners6.CornerRadius = UDim.new(1, 0)
corners6.Parent = knob

-- Botões de velocidade rápida
local botoesFrame = Instance.new("Frame")
botoesFrame.Size = UDim2.new(0.9, 0, 0, 100)
botoesFrame.Position = UDim2.new(0.05, 0, 0, 220)
botoesFrame.BackgroundTransparency = 1
botoesFrame.Parent = painel

-- Função para criar botões
local function criarBotao(texto, posX, velocidade)
    local botao = Instance.new("TextButton")
    botao.Size = UDim2.new(0.28, 0, 0, 40)
    botao.Position = UDim2.new(posX, 0, 0, 0)
    botao.Text = texto
    botao.TextColor3 = Color3.fromRGB(255, 255, 255)
    botao.TextScaled = true
    botao.Font = Enum.Font.GothamBold
    botao.BackgroundColor3 = Color3.fromRGB(50, 50, 70)
    botao.BorderSizePixel = 0
    
    local botaoCorners = Instance.new("UICorner")
    botaoCorners.CornerRadius = UDim.new(0, 10)
    botaoCorners.Parent = botao
    
    botao.MouseButton1Click:Connect(function()
        humanoid.WalkSpeed = velocidade
        atualizarVelocidade(velocidade)
    end)
    
    botao.Parent = botoesFrame
    return botao
end

-- Botões de velocidade
criarBotao("🐢 Lento", 0, 10)
criarBotao("🚶 Normal", 0.36, 16)
criarBotao("🐇 Rápido", 0.72, 32)

-- Botões extras (segunda linha)
local function criarBotao2(texto, posX, velocidade)
    local botao = Instance.new("TextButton")
    botao.Size = UDim2.new(0.28, 0, 0, 40)
    botao.Position = UDim2.new(posX, 0, 0, 50)
    botao.Text = texto
    botao.TextColor3 = Color3.fromRGB(255, 255, 255)
    botao.TextScaled = true
    botao.Font = Enum.Font.GothamBold
    botao.BackgroundColor3 = Color3.fromRGB(70, 50, 90)
    botao.BorderSizePixel = 0
    
    local botaoCorners = Instance.new("UICorner")
    botaoCorners.CornerRadius = UDim.new(0, 10)
    botaoCorners.Parent = botao
    
    botao.MouseButton1Click:Connect(function()
        humanoid.WalkSpeed = velocidade
        atualizarVelocidade(velocidade)
    end)
    
    botao.Parent = botoesFrame
    return botao
end

criarBotao2("⚡ Super", 0, 50)
criarBotao2("💨 Turbo", 0.36, 80)
criarBotao2("🚀 Light", 0.72, 120)

-- Botão de reset
local resetBtn = Instance.new("TextButton")
resetBtn.Size = UDim2.new(0.8, 0, 0, 45)
resetBtn.Position = UDim2.new(0.1, 0, 0, 340)
resetBtn.Text = "🌕 Resetar Velocidade"
resetBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
resetBtn.TextScaled = true
resetBtn.Font = Enum.Font.GothamBold
resetBtn.BackgroundColor3 = Color3.fromRGB(100, 70, 130)
resetBtn.BorderSizePixel = 0
resetBtn.Parent = painel

local resetCorners = Instance.new("UICorner")
resetCorners.CornerRadius = UDim.new(0, 10)
resetCorners.Parent = resetBtn

resetBtn.MouseButton1Click:Connect(function()
    humanoid.WalkSpeed = 16
    atualizarVelocidade(16)
    sliderFill.Size = UDim2.new(0.5, 0, 1, 0)
    knob.Position = UDim2.new(0.5, -10, 0.5, -10)
end)

-- Função para atualizar o valor exibido
local function atualizarVelocidade(valor)
    velValor.Text = tostring(math.floor(valor))
end

-- Sistema de arrastar do slider
local dragging = false

sliderFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
    end
end)

sliderFrame.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

game:GetService("UserInputService").InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local mousePos = input.Position.X
        local sliderAbsPos = sliderFrame.AbsolutePosition.X
        local sliderWidth = sliderFrame.AbsoluteSize.X
        
        local percent = (mousePos - sliderAbsPos) / sliderWidth
        percent = math.clamp(percent, 0, 1)
        
        local velocidade = 10 + (percent * 110) -- de 10 a 120
        humanoid.WalkSpeed = velocidade
        atualizarVelocidade(velocidade)
        
        sliderFill.Size = UDim2.new(percent, 0, 1, 0)
        knob.Position = UDim2.new(percent, -10, 0.5, -10)
    end
end)

-- Atualizar quando o personagem mudar
player.CharacterAdded:Connect(function(newChar)
    character = newChar
    humanoid = character:WaitForChild("Humanoid")
end)

-- Animação de entrada
painel.BackgroundTransparency = 0.2
painel:TweenSize(UDim2.new(0, 300, 0, 400), Enum.EasingDirection.Out, Enum.EasingStyle.Back, 0.5, true)

print("🌙 Painel Lua carregado com sucesso! 🌙")
