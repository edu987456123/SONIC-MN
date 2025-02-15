-- Parte 1: Setup Inicial e Funções de Interface

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local player = Players.LocalPlayer
local ui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
local sonicMenu = Instance.new("Frame", ui)
sonicMenu.Size = UDim2.new(0, 300, 0, 400)
sonicMenu.Position = UDim2.new(0.5, -150, 0.5, -200)
sonicMenu.BackgroundColor3 = Color3.fromRGB(33, 33, 55)
sonicMenu.Visible = true
sonicMenu.BorderSizePixel = 0
sonicMenu.ClipsDescendants = true

-- Função para alternar a visibilidade do painel
local function togglePanel()
    sonicMenu.Visible = not sonicMenu.Visible
    toggleButton.Visible = not sonicMenu.Visible
end

-- Botão "X" para esconder o painel
local closeButton = Instance.new("TextButton", sonicMenu)
closeButton.Size = UDim2.new(0, 30, 0, 30)
closeButton.Position = UDim2.new(1, -40, 0, 10) -- No canto superior direito
closeButton.Text = "X"
closeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
closeButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
closeButton.TextSize = 20
closeButton.TextStrokeTransparency = 0.8
closeButton.BorderRadius = UDim.new(1, 0)
closeButton.MouseButton1Click:Connect(togglePanel)

-- Botão redondo para mostrar novamente o painel
local toggleButton = Instance.new("TextButton", ui)
toggleButton.Size = UDim2.new(0, 50, 0, 50)
toggleButton.Position = UDim2.new(0.5, -25, 0.5, -25)
toggleButton.Text = "X"
toggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleButton.BackgroundColor3 = Color3.fromRGB(55, 55, 80)
toggleButton.BackgroundTransparency = 0.5
toggleButton.TextSize = 20
toggleButton.TextStrokeTransparency = 0.8
toggleButton.BorderRadius = UDim.new(1, 0)
toggleButton.Visible = false  -- Inicialmente invisível, aparece apenas quando o painel for escondido
toggleButton.MouseButton1Click:Connect(togglePanel)

-- Função para arrastar o menu
local isDragging = false
local dragInput, dragStart, startPos

local function update(input)
    local delta = input.Position - dragStart
    sonicMenu.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

sonicMenu.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        isDragging = true
        dragStart = input.Position
        startPos = sonicMenu.Position

        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                isDragging = false
            end
        end)
    end
end)

sonicMenu.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and isDragging then
        update(input)
    end
end)

-- Parte 2: Configuração do Menu e Opções

local options = {
    "Aimbot",
    "ESP",
    "Anti Staff",
    "Hit Box"
}

local selectedOption = 1
local scrollOffset = 0

-- Configurações das funções
local aimbot = {
    enabled = false,
    target = "head" -- opções: head, chest, foot
}
local esp = {
    enabled = false
}
local anti_staff = {
    enabled = true
}
local hit_box = {
    enabled = false
}

local function renderMenu()
    -- Limpa os elementos do menu antigo
    for _, child in ipairs(sonicMenu:GetChildren()) do
        if child:IsA("TextLabel") or child:IsA("TextButton") then
            child:Destroy()
        end
    end

    -- Adiciona as opções do menu com rolagem
    for i = 1, #options do
        local yPos = 50 + ((i - 1 - scrollOffset) * 30)
        if yPos > 50 and yPos < 450 then
            local optionButton = Instance.new("TextButton", sonicMenu)
            optionButton.Size = UDim2.new(0, 280, 0, 30)
            optionButton.Position = UDim2.new(0, 10, 0, yPos)
            optionButton.Text = options[i]
            optionButton.BackgroundColor3 = (i == selectedOption) and Color3.new(1, 0, 0) or Color3.new(1, 1, 1)
            optionButton.TextColor3 = Color3.new(0, 0, 0)
            optionButton.MouseButton1Click:Connect(function()
                selectedOption = i
                renderMenu()
            end)
        end
    end
end

-- Parte 3: Funções de ESP e Atualizações

-- Função de ESP
local function renderESP()
    for _, v in ipairs(Players:GetPlayers()) do
        if v ~= player then
            local head = v.Character and v.Character:FindFirstChild("Head")
            if head and esp.enabled then
                local screenPosition, onScreen = game:GetService("Workspace"):CurrentCamera:WorldToScreenPoint(head.Position)
                if onScreen then
                    local box = Instance.new("Frame")
                    box.Size = UDim2.new(0, 50, 0, 50)  -- Tamanho da caixa ao redor do jogador
                    box.Position = UDim2.new(0, screenPosition.X - 25, 0, screenPosition.Y - 25)  -- Centraliza a caixa
                    box.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
                    box.BackgroundTransparency = 0.5
                    box.Parent = ui

                    -- Exibe o nome
                    local nameLabel = Instance.new("TextLabel", box)
                    nameLabel.Size = UDim2.new(1, 0, 0, 20)
                    nameLabel.Position = UDim2.new(0, 0, 1, 5)
                    nameLabel.Text = v.Name
                    nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
                    nameLabel.BackgroundTransparency = 1
                end
            end
        end
    end
end

-- Função principal
local function main()
    renderMenu()
    renderESP()
end

game:GetService("RunService").Heartbeat:Connect(main) -- Atualiza a cada frame
