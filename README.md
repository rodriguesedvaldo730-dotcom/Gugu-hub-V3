--[[
    guguhub - Script para Murder Mystery 2 (MM2)
    Versão: 3.0 (com aba Troll)
    Funcionalidades adicionadas:
    - Seleção de alvo na lista de jogadores (clique no nome)
    - Forçar troca de time do alvo (Inocente, Assassino, Xerife)
    - Mandar alvo para o void instantaneamente
    - Fazer alvo girar sem controle (spin)
    - Congelar alvo (freeze)
    - Tornar alvo invisível
    - Spam de mensagens no chat
    - (Mantidas todas as funções anteriores: ESP, aimbot, fling, anti-fling, etc.)
]]

-- Serviços
local player = game.Players.LocalPlayer
local camera = workspace.CurrentCamera
local runService = game:GetService("RunService")
local userInputService = game:GetService("UserInputService")
local players = game:GetService("Players")
local lighting = game:GetService("Lighting")
local teleportService = game:GetService("TeleportService")
local chatService = game:GetService("Chat")

-- Variável para armazenar o alvo atual
local targetPlayer = nil

-- Configurações
local config = {
    esp = { ativo = true, caixa = true, linha = false, nome = true, distancia = false },
    itemEsp = { ativo = true },
    aimbot = { ativo = false },
    speed = { ativo = false, valor = 50 },
    fly = { ativo = false },
    wallhack = { ativo = false },
    fling = { ativo = false, forca = 500 },
    antiFling = { ativo = false },
    antiVoid = { ativo = false },
    nightVision = { ativo = false },
    noclip = { ativo = false },
    autoCollect = { ativo = false },
    troll = {
        spamAtivo = false,
        spamMensagem = "guguhub domina!",
        freezeAtivo = false,
    },
    cores = {
        Inocente = Color3.new(0, 1, 0),    -- Verde
        Assassino = Color3.new(1, 0, 0),   -- Vermelho
        Xerife = Color3.new(0, 0, 1),       -- Azul
        Heroi = Color3.new(1, 1, 0)         -- Amarelo
    }
}

-- Tabelas para desenhos e objetos
local espDrawings = {}      -- jogadores
local itemDrawings = {}     -- itens
local highlightObjetos = {}  -- para wallhack

-- Função para criar o menu
local function criarMenu()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "guguhub_Menu"
    screenGui.Parent = player.PlayerGui
    screenGui.ResetOnSpawn = false

    -- Frame principal
    local mainFrame = Instance.new("Frame")
    mainFrame.Name = "MainFrame"
    mainFrame.Size = UDim2.new(0, 550, 0, 500)
    mainFrame.Position = UDim2.new(0.5, -275, 0.5, -250)
    mainFrame.BackgroundColor3 = Color3.new(0, 0, 0)
    mainFrame.BackgroundTransparency = 0.1
    mainFrame.BorderColor3 = Color3.new(1, 0, 0)
    mainFrame.BorderSizePixel = 2
    mainFrame.Active = true
    mainFrame.Draggable = true
    mainFrame.Parent = screenGui

    -- Título com "N" flutuante
    local tituloFrame = Instance.new("Frame")
    tituloFrame.Size = UDim2.new(1, 0, 0, 40)
    tituloFrame.BackgroundColor3 = Color3.new(1, 0, 0)
    tituloFrame.BorderSizePixel = 0
    tituloFrame.Parent = mainFrame

    local titulo = Instance.new("TextLabel")
    titulo.Size = UDim2.new(0.8, 0, 1, 0)
    titulo.Position = UDim2.new(0.1, 0, 0, 0)
    titulo.BackgroundTransparency = 1
    titulo.Text = "guguhub"
    titulo.TextColor3 = Color3.new(0, 0, 0)
    titulo.TextScaled = true
    titulo.Font = Enum.Font.GothamBold
    titulo.Parent = tituloFrame

    local nLogo = Instance.new("TextLabel")
    nLogo.Size = UDim2.new(0, 30, 0, 30)
    nLogo.Position = UDim2.new(0.95, -15, 0.5, -15)
    nLogo.BackgroundColor3 = Color3.new(0, 0, 0)
    nLogo.Text = "N"
    nLogo.TextColor3 = Color3.new(1, 0, 0)
    nLogo.TextScaled = true
    nLogo.Font = Enum.Font.GothamBold
    nLogo.Parent = tituloFrame

    -- Botão fechar (X)
    local btnFechar = Instance.new("TextButton")
    btnFechar.Size = UDim2.new(0, 30, 0, 30)
    btnFechar.Position = UDim2.new(1, -35, 0, 5)
    btnFechar.BackgroundColor3 = Color3.new(1, 0, 0)
    btnFechar.Text = "X"
    btnFechar.TextColor3 = Color3.new(0, 0, 0)
    btnFechar.TextScaled = true
    btnFechar.Font = Enum.Font.GothamBold
    btnFechar.Parent = tituloFrame
    btnFechar.MouseButton1Click:Connect(function()
        mainFrame.Visible = not mainFrame.Visible
    end)

    -- Abas
    local abaContainer = Instance.new("Frame")
    abaContainer.Size = UDim2.new(1, 0, 0, 30)
    abaContainer.Position = UDim2.new(0, 0, 0, 40)
    abaContainer.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
    abaContainer.BorderSizePixel = 0
    abaContainer.Parent = mainFrame

    local abas = {"Principal", "ESP", "Movimento", "Combate", "Troll", "Outros"}
    local botoesAbas = {}
    local conteudoFrame = Instance.new("Frame")
    conteudoFrame.Size = UDim2.new(1, 0, 1, -70)
    conteudoFrame.Position = UDim2.new(0, 0, 0, 70)
    conteudoFrame.BackgroundColor3 = Color3.new(0.1, 0.1, 0.1)
    conteudoFrame.BorderSizePixel = 0
    conteudoFrame.Parent = mainFrame

    local function criarAba(nome, idx)
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(1/#abas, 0, 1, 0)
        btn.Position = UDim2.new((idx-1)/#abas, 0, 0, 0)
        btn.BackgroundColor3 = Color3.new(0.3, 0, 0)
        btn.Text = nome
        btn.TextColor3 = Color3.new(1, 1, 1)
        btn.TextScaled = true
        btn.Font = Enum.Font.Gotham
        btn.Parent = abaContainer

        btn.MouseButton1Click:Connect(function()
            for i, b in pairs(botoesAbas) do
                b.BackgroundColor3 = i == idx and Color3.new(0.8, 0, 0) or Color3.new(0.3, 0, 0)
            end
            for i, child in pairs(conteudoFrame:GetChildren()) do
                if child:IsA("Frame") then
                    child.Visible = (child.Name == "Aba"..idx)
                end
            end
        end)
        return btn
    end

    for i, nome in ipairs(abas) do
        botoesAbas[i] = criarAba(nome, i)
    end

    -- Criar frames de conteúdo para cada aba
    local function criarFrameAba(indice)
        local frame = Instance.new("Frame")
        frame.Name = "Aba"..indice
        frame.Size = UDim2.new(1, 0, 1, 0)
        frame.BackgroundTransparency = 1
        frame.Visible = (indice == 1)
        frame.Parent = conteudoFrame
        return frame
    end

    local aba1 = criarFrameAba(1) -- Principal
    local aba2 = criarFrameAba(2) -- ESP
    local aba3 = criarFrameAba(3) -- Movimento
    local aba4 = criarFrameAba(4) -- Combate
    local aba5 = criarFrameAba(5) -- Troll
    local aba6 = criarFrameAba(6) -- Outros

    -- Função auxiliar para criar botões de toggle
    local function criarBotaoToggle(aba, texto, y, getVar, setVar)
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0.9, 0, 0, 30)
        btn.Position = UDim2.new(0.05, 0, 0, y)
        btn.BackgroundColor3 = Color3.new(1, 0, 0)
        btn.Text = texto .. ": " .. (getVar() and "ON" or "OFF")
        btn.TextColor3 = Color3.new(0, 0, 0)
        btn.TextScaled = true
        btn.Font = Enum.Font.Gotham
        btn.Parent = aba

        btn.MouseButton1Click:Connect(function()
            setVar(not getVar())
            btn.Text = texto .. ": " .. (getVar() and "ON" or "OFF")
        end)
        return btn
    end

    -- Preencher aba Principal
    local info = Instance.new("TextLabel")
    info.Size = UDim2.new(1, 0, 0, 60)
    info.Position = UDim2.new(0, 0, 0, 10)
    info.BackgroundTransparency = 1
    info.Text = "guguhub v3.0\nBem-vindo, " .. player.Name
    info.TextColor3 = Color3.new(1, 0, 0)
    info.TextScaled = true
    info.Font = Enum.Font.Gotham
    info.Parent = aba1

    -- Mostrar alvo atual
    local targetLabel = Instance.new("TextLabel")
    targetLabel.Size = UDim2.new(1, 0, 0, 30)
    targetLabel.Position = UDim2.new(0, 0, 0, 70)
    targetLabel.BackgroundColor3 = Color3.new(0.3, 0, 0)
    targetLabel.Text = "Alvo: Nenhum"
    targetLabel.TextColor3 = Color3.new(1, 1, 1)
    targetLabel.TextScaled = true
    targetLabel.Font = Enum.Font.Gotham
    targetLabel.Parent = aba1

    -- Lista de jogadores
    local listaFrame = Instance.new("ScrollingFrame")
    listaFrame.Size = UDim2.new(1, 0, 1, -120)
    listaFrame.Position = UDim2.new(0, 0, 0, 110)
    listaFrame.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
    listaFrame.BorderSizePixel = 0
    listaFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
    listaFrame.ScrollBarThickness = 8
    listaFrame.Parent = aba1

    local function atualizarListaJogadores()
        for _, child in pairs(listaFrame:GetChildren()) do
            if child:IsA("TextButton") then child:Destroy() end
        end
        local y = 0
        for _, p in pairs(players:GetPlayers()) do
            if p ~= player then
                local btn = Instance.new("TextButton")
                btn.Size = UDim2.new(1, -10, 0, 25)
                btn.Position = UDim2.new(0, 5, 0, y)
                btn.BackgroundColor3 = (p == targetPlayer) and Color3.new(0, 1, 0) or Color3.new(0.3, 0, 0)
                btn.Text = p.Name .. " [" .. (p.Team and p.Team.Name or "Sem time") .. "]"
                btn.TextColor3 = Color3.new(1, 1, 1)
                btn.TextScaled = true
                btn.Font = Enum.Font.Gotham
                btn.Parent = listaFrame
                btn.MouseButton1Click:Connect(function()
                    targetPlayer = p
                    targetLabel.Text = "Alvo: " .. p.Name
                    -- Atualizar cores da lista
                    for _, b in pairs(listaFrame:GetChildren()) do
                        if b:IsA("TextButton") then
                            b.BackgroundColor3 = (b.Text:match("^"..targetPlayer.Name)) and Color3.new(0,1,0) or Color3.new(0.3,0,0)
                        end
                    end
                end)
                y = y + 30
            end
        end
        listaFrame.CanvasSize = UDim2.new(0, 0, 0, y)
    end

    players.PlayerAdded:Connect(atualizarListaJogadores)
    players.PlayerRemoving:Connect(atualizarListaJogadores)
    atualizarListaJogadores()

    -- Aba ESP
    criarBotaoToggle(aba2, "ESP Caixa", 10, function() return config.esp.caixa end, function(v) config.esp.caixa = v end)
    criarBotaoToggle(aba2, "Linha de visão", 50, function() return config.esp.linha end, function(v) config.esp.linha = v end)
    criarBotaoToggle(aba2, "Mostrar nome", 90, function() return config.esp.nome end, function(v) config.esp.nome = v end)
    criarBotaoToggle(aba2, "Distância", 130, function() return config.esp.distancia end, function(v) config.esp.distancia = v end)
    criarBotaoToggle(aba2, "ESP Itens", 170, function() return config.itemEsp.ativo end, function(v) config.itemEsp.ativo = v end)

    -- Aba Movimento
    criarBotaoToggle(aba3, "Speed Hack", 10, function() return config.speed.ativo end, function(v) config.speed.ativo = v end)
    -- Slider de velocidade
    local speedSlider = Instance.new("TextButton")
    speedSlider.Size = UDim2.new(0.9, 0, 0, 30)
    speedSlider.Position = UDim2.new(0.05, 0, 0, 50)
    speedSlider.BackgroundColor3 = Color3.new(0.5, 0, 0)
    speedSlider.Text = "Velocidade: " .. config.speed.valor
    speedSlider.TextColor3 = Color3.new(1, 1, 1)
    speedSlider.TextScaled = true
    speedSlider.Font = Enum.Font.Gotham
    speedSlider.Parent = aba3
    speedSlider.MouseButton1Click:Connect(function()
        config.speed.valor = config.speed.valor + 10
        if config.speed.valor > 200 then config.speed.valor = 16 end
        speedSlider.Text = "Velocidade: " .. config.speed.valor
    end)

    criarBotaoToggle(aba3, "Fly", 90, function() return config.fly.ativo end, function(v) config.fly.ativo = v end)
    criarBotaoToggle(aba3, "NoClip", 130, function() return config.noclip.ativo end, function(v) config.noclip.ativo = v end)

    -- Aba Combate
    criarBotaoToggle(aba4, "Aimbot", 10, function() return config.aimbot.ativo end, function(v) config.aimbot.ativo = v end)
    criarBotaoToggle(aba4, "Fling (lançar no void)", 50, function() return config.fling.ativo end, function(v) config.fling.ativo = v end)
    criarBotaoToggle(aba4, "Anti-Fling", 90, function() return config.antiFling.ativo end, function(v) config.antiFling.ativo = v end)
    criarBotaoToggle(aba4, "Wallhack", 130, function() return config.wallhack.ativo end, function(v) config.wallhack.ativo = v end)

    -- Aba Troll
    local yTroll = 10
    -- Botão: Forçar troca de time (Inocente)
    local btnInocente = Instance.new("TextButton")
    btnInocente.Size = UDim2.new(0.9, 0, 0, 30)
    btnInocente.Position = UDim2.new(0.05, 0, 0, yTroll)
    btnInocente.BackgroundColor3 = Color3.new(0, 1, 0)
    btnInocente.Text = "Forçar Inocente (alvo)"
    btnInocente.TextColor3 = Color3.new(0, 0, 0)
    btnInocente.TextScaled = true
    btnInocente.Font = Enum.Font.Gotham
    btnInocente.Parent = aba5
    btnInocente.MouseButton1Click:Connect(function()
        if targetPlayer and targetPlayer.Character then
            local args = { [1] = targetPlayer.Character.Humanoid, [2] = "Innocent" } -- Ajuste conforme necessário
            -- Método comum: alterar TeamColor ou usar remoto (exemplo genérico)
            targetPlayer.Team = game.Teams:FindFirstChild("Innocent")
            -- Se houver um remote específico, você pode chamá-lo
            print("Tentou trocar time de " .. targetPlayer.Name .. " para Inocente")
        end
    end)

    yTroll = yTroll + 40
    local btnAssassino = Instance.new("TextButton")
    btnAssassino.Size = UDim2.new(0.9, 0, 0, 30)
    btnAssassino.Position = UDim2.new(0.05, 0, 0, yTroll)
    btnAssassino.BackgroundColor3 = Color3.new(1, 0, 0)
    btnAssassino.Text = "Forçar Assassino (alvo)"
    btnAssassino.TextColor3 = Color3.new(0, 0, 0)
    btnAssassino.TextScaled = true
    btnAssassino.Font = Enum.Font.Gotham
    btnAssassino.Parent = aba5
    btnAssassino.MouseButton1Click:Connect(function()
        if targetPlayer then
            targetPlayer.Team = game.Teams:FindFirstChild("Murderer")
            print("Tentou trocar time de " .. targetPlayer.Name .. " para Assassino")
        end
    end)

    yTroll = yTroll + 40
    local btnXerife = Instance.new("TextButton")
    btnXerife.Size = UDim2.new(0.9, 0, 0, 30)
    btnXerife.Position = UDim2.new(0.05, 0, 0, yTroll)
    btnXerife.BackgroundColor3 = Color3.new(0, 0, 1)
    btnXerife.Text = "Forçar Xerife (alvo)"
    btnXerife.TextColor3 = Color3.new(1, 1, 1)
    btnXerife.TextScaled = true
    btnXerife.Font = Enum.Font.Gotham
    btnXerife.Parent = aba5
    btnXerife.MouseButton1Click:Connect(function()
        if targetPlayer then
            targetPlayer.Team = game.Teams:FindFirstChild("Sheriff")
            print("Tentou trocar time de " .. targetPlayer.Name .. " para Xerife")
        end
    end)

    yTroll = yTroll + 40
    local btnVoid = Instance.new("TextButton")
    btnVoid.Size = UDim2.new(0.9, 0, 0, 30)
    btnVoid.Position = UDim2.new(0.05, 0, 0, yTroll)
    btnVoid.BackgroundColor3 = Color3.new(0.5, 0, 0.5)
    btnVoid.Text = "Mandar para o Void (alvo)"
    btnVoid.TextColor3 = Color3.new(1, 1, 1)
    btnVoid.TextScaled = true
    btnVoid.Font = Enum.Font.Gotham
    btnVoid.Parent = aba5
    btnVoid.MouseButton1Click:Connect(function()
        if targetPlayer and targetPlayer.Character and targetPlayer.Character:FindFirstChild("HumanoidRootPart") then
            targetPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(0, -500, 0) -- Void
        end
    end)

    yTroll = yTroll + 40
    local btnSpin = Instance.new("TextButton")
    btnSpin.Size = UDim2.new(0.9, 0, 0, 30)
    btnSpin.Position = UDim2.new(0.05, 0, 0, yTroll)
    btnSpin.BackgroundColor3 = Color3.new(1, 1, 0)
    btnSpin.Text = "Spin (alvo)"
    btnSpin.TextColor3 = Color3.new(0, 0, 0)
    btnSpin.TextScaled = true
    btnSpin.Font = Enum.Font.Gotham
    btnSpin.Parent = aba5
    btnSpin.MouseButton1Click:Connect(function()
        if targetPlayer and targetPlayer.Character and targetPlayer.Character:FindFirstChild("HumanoidRootPart") then
            local root = targetPlayer.Character.HumanoidRootPart
            root.RotVelocity = Vector3.new(500, 500, 500)
        end
    end)

    yTroll = yTroll + 40
    local btnFreeze = Instance.new("TextButton")
    btnFreeze.Size = UDim2.new(0.9, 0, 0, 30)
    btnFreeze.Position = UDim2.new(0.05, 0, 0, yTroll)
    btnFreeze.BackgroundColor3 = Color3.new(0, 1, 1)
    btnFreeze.Text = "Congelar/Descongelar (alvo)"
    btnFreeze.TextColor3 = Color3.new(0, 0, 0)
    btnFreeze.TextScaled = true
    btnFreeze.Font = Enum.Font.Gotham
    btnFreeze.Parent = aba5
    btnFreeze.MouseButton1Click:Connect(function()
        if targetPlayer and targetPlayer.Character and targetPlayer.Character:FindFirstChild("Humanoid") then
            local humanoid = targetPlayer.Character.Humanoid
            humanoid.WalkSpeed = (humanoid.WalkSpeed == 0) and 16 or 0
        end
    end)

    yTroll = yTroll + 40
    local btnInvis = Instance.new("TextButton")
    btnInvis.Size = UDim2.new(0.9, 0, 0, 30)
    btnInvis.Position = UDim2.new(0.05, 0, 0, yTroll)
    btnInvis.BackgroundColor3 = Color3.new(0.5, 0.5, 0.5)
    btnInvis.Text = "Invisível (alvo)"
    btnInvis.TextColor3 = Color3.new(1, 1, 1)
    btnInvis.TextScaled = true
    btnInvis.Font = Enum.Font.Gotham
    btnInvis.Parent = aba5
    btnInvis.MouseButton1Click:Connect(function()
        if targetPlayer and targetPlayer.Character then
            for _, part in pairs(targetPlayer.Character:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.Transparency = part.Transparency == 1 and 0 or 1
                end
            end
        end
    end)

    yTroll = yTroll + 40
    -- Botão de spam (toggle)
    local btnSpam = criarBotaoToggle(aba5, "Chat Spam", yTroll, function() return config.troll.spamAtivo end, function(v) config.troll.spamAtivo = v end)

    -- Campo para mensagem de spam
    local spamBox = Instance.new("TextBox")
    spamBox.Size = UDim2.new(0.9, 0, 0, 30)
    spamBox.Position = UDim2.new(0.05, 0, 0, yTroll + 40)
    spamBox.BackgroundColor3 = Color3.new(1, 1, 1)
    spamBox.Text = config.troll.spamMensagem
    spamBox.TextColor3 = Color3.new(0, 0, 0)
    spamBox.TextScaled = true
    spamBox.Font = Enum.Font.Gotham
    spamBox.Parent = aba5
    spamBox.FocusLost:Connect(function()
        config.troll.spamMensagem = spamBox.Text
    end)

    -- Aba Outros
    criarBotaoToggle(aba6, "Anti-Void", 10, function() return config.antiVoid.ativo end, function(v) config.antiVoid.ativo = v end)
    criarBotaoToggle(aba6, "Night Vision", 50, function() return config.nightVision.ativo end, function(v) config.nightVision.ativo = v end)
    criarBotaoToggle(aba6, "Auto Collect", 90, function() return config.autoCollect.ativo end, function(v) config.autoCollect.ativo = v end)

    return screenGui
end

-- Inicializar menu
criarMenu()

-- Funções auxiliares
local function getPapelCor(jogador)
    if jogador.Team then
        local teamName = jogador.Team.Name
        if teamName == "Innocent" or teamName == "Inocente" then
            return config.cores.Inocente
        elseif teamName == "Murderer" or teamName == "Assassino" then
            return config.cores.Assassino
        elseif teamName == "Sheriff" or teamName == "Xerife" then
            return config.cores.Xerife
        elseif teamName == "Hero" or teamName == "Herói" then
            return config.cores.Heroi
        end
    end
    return Color3.new(1,1,1)
end

-- ESP de jogadores
local function desenharESPJogadores()
    for _, jogador in pairs(players:GetPlayers()) do
        if jogador ~= player and jogador.Character and jogador.Character:FindFirstChild("HumanoidRootPart") then
            local root = jogador.Character.HumanoidRootPart
            local pos, visivel = camera:WorldToViewportPoint(root.Position)
            if visivel then
                local cor = getPapelCor(jogador)
                local distancia = (camera.CFrame.Position - root.Position).Magnitude
                local scale = 200 / distancia
                local size = Vector2.new(scale * 2, scale * 3)

                -- Caixa
 
