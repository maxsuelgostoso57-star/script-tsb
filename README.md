-- TSB Contra KaratekaFF - Hub Completo
local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()

local Window = Fluent:CreateWindow({
    Title = "TSB Contra KaratekaFF",
    SubTitle = "by KaratekaFF",
    TabWidth = 160,
    Theme = "Dark",
    Acrylic = false,
    Size = UDim2.fromOffset(580, 460),
    MinimizeKey = Enum.KeyCode.Insert
})

-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local player = Players.LocalPlayer

-- Variáveis Globais
local hiddenfling = false
local flingThread = nil
local flingAllEnabled = false
local targetPlayer = nil
local stickToPlayer = false
local stickConnection = nil
local spinPlayer = false
local spinConnection = nil
local spinDistance = 5

local farmKillsActive = false
local farmDistance = 10
local farmVelocity = 30

local walkSpeed = 16
local jumpPower = 50

-- Criar Abas
local Tabs = {
    Home = Window:AddTab({Title = "🏠 Home", Icon = "home"}),
    Fling = Window:AddTab({Title = "💥 Fling", Icon = "zap"}),
    FarmKills = Window:AddTab({Title = "🎯 Farm Kills", Icon = "target"}),
    AntiStun = Window:AddTab({Title = "⚡ Anti Stun", Icon = "shield"}),
    CaosTotal = Window:AddTab({Title = "💀 Caos Total", Icon = "skull"})
}

-- ========================================
-- ABA HOME
-- ========================================
Tabs.Home:AddParagraph({
    Title = "TSB by KaratekaFF",
    Content = "Hub completo para TSB"
})

Tabs.Home:AddParagraph({
    Title = "🎮 Discord Server",
    Content = "Entre no Discord para suporte e updates"
})

Tabs.Home:AddButton({
    Title = "📋 Copiar Link Discord",
    Description = "Copia: discord.gg/seulink",
    Callback = function()
        setclipboard("discord.gg/seulink")
        Fluent:Notify({
            Title = "✅ Copiado!",
            Content = "Link copiado para área de transferência",
            Duration = 3
        })
    end
})

Tabs.Home:AddParagraph({
    Title = "ℹ️ Como Usar",
    Content = "• Fling: Selecione um player, ative fling e use as opções\n• Farm Kills: Escolha alvo e ligue\n• Anti Stun: Configure velocidade e pulo\n• Caos Total: Modo destruição ativado"
})

-- ========================================
-- ABA FLING
-- ========================================

Tabs.Fling:AddParagraph({
    Title = "💥 Sistema de Fling",
    Content = "Selecione um jogador e use as opções abaixo"
})

-- Lista de Players
local playerNames = {}
local function updatePlayerList()
    playerNames = {}
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= player then
            table.insert(playerNames, plr.Name)
        end
    end
    return playerNames
end

local PlayerDropdown = Tabs.Fling:AddDropdown("PlayerDropdown", {
    Title = "🎯 Selecionar Jogador",
    Values = updatePlayerList(),
    Multi = false,
    Default = 1,
    Callback = function(Value)
        targetPlayer = Players:FindFirstChild(Value)
        if targetPlayer then
            Fluent:Notify({
                Title = "Alvo Selecionado",
                Content = "Target: " .. targetPlayer.Name,
                Duration = 3
            })
        end
    end
})

-- Atualizar lista
Players.PlayerAdded:Connect(function()
    wait(1)
    PlayerDropdown:SetValues(updatePlayerList())
end)

Players.PlayerRemoving:Connect(function()
    wait(1)
    PlayerDropdown:SetValues(updatePlayerList())
end)

-- Sistema de grudar no player
local function startStickToPlayer()
    if stickConnection then stickConnection:Disconnect() end
    
    stickConnection = RunService.Heartbeat:Connect(function()
        if stickToPlayer and targetPlayer and targetPlayer.Character and player.Character then
            local myRoot = player.Character:FindFirstChild("HumanoidRootPart")
            local targetRoot = targetPlayer.Character:FindFirstChild("HumanoidRootPart")
            
            if myRoot and targetRoot then
                myRoot.CFrame = targetRoot.CFrame * CFrame.new(0, 0, 3)
            end
        end
    end)
end

-- Botão TP para o player (e gruda)
Tabs.Fling:AddButton({
    Title = "🚀 TP e Grudar no Jogador",
    Description = "Teleporta e gruda em você no jogador selecionado",
    Callback = function()
        if not targetPlayer then
            Fluent:Notify({
                Title = "Erro!",
                Content = "Selecione um jogador primeiro",
                Duration = 3
            })
            return
        end
        
        if targetPlayer.Character and player.Character then
            local targetRoot = targetPlayer.Character:FindFirstChild("HumanoidRootPart")
            local myRoot = player.Character:FindFirstChild("HumanoidRootPart")
            
            if targetRoot and myRoot then
                myRoot.CFrame = targetRoot.CFrame * CFrame.new(0, 0, 3)
                stickToPlayer = true
                startStickToPlayer()
                Fluent:Notify({
                    Title = "Grudado!",
                    Content = "TP e grudado em " .. targetPlayer.Name,
                    Duration = 2
                })
            end
        end
    end
})

-- Toggle para desgrudar
local StickToggle = Tabs.Fling:AddToggle("StickToggle", {
    Title = "🔗 Grudar no Player",
    Description = "Mantém você grudado no player selecionado",
    Default = false,
    Callback = function(Value)
        stickToPlayer = Value
        if stickToPlayer then
            startStickToPlayer()
            Fluent:Notify({
                Title = "Grudado!",
                Content = "Você está grudado no player",
                Duration = 2
            })
        else
            if stickConnection then
                stickConnection:Disconnect()
                stickConnection = nil
            end
            Fluent:Notify({
                Title = "Descolado",
                Content = "Você não está mais grudado",
                Duration = 2
            })
        end
    end
})

-- Touch Fling (SISTEMA EXATO DO CÓDIGO)
local function fling()
    local lp = Players.LocalPlayer
    local c, hrp, vel, movel = nil, nil, nil, 0.1

    while hiddenfling do
        RunService.Heartbeat:Wait()
        c = lp.Character
        hrp = c and c:FindFirstChild("HumanoidRootPart")

        if hrp then
            vel = hrp.Velocity
            hrp.Velocity = vel * 10000 + Vector3.new(0, 10000, 0)
            RunService.RenderStepped:Wait()
            hrp.Velocity = vel
            RunService.Stepped:Wait()
            hrp.Velocity = vel + Vector3.new(0, movel, 0)
            movel = -movel
        end
    end
end

local FlingToggle = Tabs.Fling:AddToggle("FlingToggle", {
    Title = "💥 Touch Fling",
    Description = "Ativa o sistema de fling (código original)",
    Default = false,
    Callback = function(Value)
        hiddenfling = Value
        
        if hiddenfling then
            -- Criar detection
            if not ReplicatedStorage:FindFirstChild("juisdfj0i32i0eidsuf0iok") then
                local detection = Instance.new("Decal")
                detection.Name = "juisdfj0i32i0eidsuf0iok"
                detection.Parent = ReplicatedStorage
            end
            
            flingThread = coroutine.create(fling)
            coroutine.resume(flingThread)
            
            Fluent:Notify({
                Title = "Fling Ativado!",
                Content = "Touch Fling está ON",
                Duration = 3
            })
        else
            hiddenfling = false
            Fluent:Notify({
                Title = "Fling Desativado",
                Content = "Touch Fling está OFF",
                Duration = 3
            })
        end
    end
})

-- Spin ao redor do player
local function startSpinPlayer()
    if spinConnection then spinConnection:Disconnect() end
    
    spinConnection = RunService.RenderStepped:Connect(function()
        if spinPlayer and targetPlayer and targetPlayer.Character and player.Character then
            local myRoot = player.Character:FindFirstChild("HumanoidRootPart")
            local targetRoot = targetPlayer.Character:FindFirstChild("HumanoidRootPart")
            
            if myRoot and targetRoot then
                local angle = tick() * 3
                local offset = Vector3.new(
                    math.cos(angle) * spinDistance,
                    0,
                    math.sin(angle) * spinDistance
                )
                myRoot.CFrame = CFrame.new(targetRoot.Position + offset, targetRoot.Position)
            end
        end
    end)
end

local SpinToggle = Tabs.Fling:AddToggle("SpinToggle", {
    Title = "🌀 Spin no Player",
    Description = "Gira ao redor do player selecionado",
    Default = false,
    Callback = function(Value)
        spinPlayer = Value
        if spinPlayer then
            startSpinPlayer()
            Fluent:Notify({
                Title = "Spin Ativado!",
                Content = "Girando ao redor do player",
                Duration = 2
            })
        else
            if spinConnection then
                spinConnection:Disconnect()
                spinConnection = nil
            end
            Fluent:Notify({
                Title = "Spin Desativado",
                Content = "Parado de girar",
                Duration = 2
            })
        end
    end
})

local SpinDistanceSlider = Tabs.Fling:AddSlider("SpinDistanceSlider", {
    Title = "📏 Distância do Spin",
    Description = "Ajuste a distância ao girar no player",
    Default = 5,
    Min = 3,
    Max = 20,
    Rounding = 0,
    Callback = function(Value)
        spinDistance = Value
    end
})

-- Fling Todos
local flingAllThread = nil
local function startFlingAll()
    flingAllThread = task.spawn(function()
        while flingAllEnabled do
            for _, plr in ipairs(Players:GetPlayers()) do
                if plr ~= player and plr.Character then
                    local targetRoot = plr.Character:FindFirstChild("HumanoidRootPart")
                    local myRoot = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
                    
                    if targetRoot and myRoot then
                        myRoot.CFrame = targetRoot.CFrame * CFrame.new(0, 0, 2)
                        task.wait(0.5)
                    end
                end
            end
            task.wait(1)
        end
    end)
end

local FlingAllToggle = Tabs.Fling:AddToggle("FlingAllToggle", {
    Title = "🌪️ Fling TODOS",
    Description = "Da TP em todos os players (use com Touch Fling ativo)",
    Default = false,
    Callback = function(Value)
        flingAllEnabled = Value
        if flingAllEnabled then
            startFlingAll()
            Fluent:Notify({
                Title = "Fling Todos Ativado!",
                Content = "TP em todos os players ativo",
                Duration = 3
            })
        else
            if flingAllThread then
                task.cancel(flingAllThread)
                flingAllThread = nil
            end
            Fluent:Notify({
                Title = "Fling Todos Desativado",
                Content = "Modo todos desligado",
                Duration = 3
            })
        end
    end
})

-- ========================================
-- ABA FARM KILLS
-- ========================================

Tabs.FarmKills:AddParagraph({
    Title = "⚠️ AVISO IMPORTANTE",
    Content = "Este sistema gruda automaticamente no player e ataca sem parar. Use com cuidado!"
})

Tabs.FarmKills:AddParagraph({
    Title = "🎯 Farm Kills Automático",
    Content = "Sistema que gruda no player, spamma teclas e ataques automaticamente"
})

-- Farm System
local farmThread = nil
local farmMovementConnection = nil

local function startFarmKills()
    if not targetPlayer or not targetPlayer.Character then
        Fluent:Notify({
            Title = "Erro!",
            Content = "Selecione um alvo válido primeiro na aba Fling",
            Duration = 5
        })
        return false
    end
    
    -- Spam de teclas e ataques
    farmThread = task.spawn(function()
        while farmKillsActive and targetPlayer do
            -- Spam 1-2-3-4
            for _, key in ipairs({"One", "Two", "Three", "Four"}) do
                if not farmKillsActive then break end
                pcall(function()
                    VirtualInputManager:SendKeyEvent(true, Enum.KeyCode[key], false, game)
                    task.wait(0.01)
                    VirtualInputManager:SendKeyEvent(false, Enum.KeyCode[key], false, game)
                end)
            end
            
            task.wait(0.03)
            
            -- Mouse Click
            if farmKillsActive then
                pcall(function()
                    VirtualInputManager:SendMouseButtonEvent(0, 0, 0, true, game, 0)
                    task.wait(0.01)
                    VirtualInputManager:SendMouseButtonEvent(0, 0, 0, false, game, 0)
                end)
            end
            
            task.wait(0.02)
            
            -- Tecla G
            if farmKillsActive then
                pcall(function()
                    VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.G, false, game)
                    task.wait(0.01)
                    VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.G, false, game)
                end)
            end
            
            task.wait(0.05)
        end
    end)
    
    -- Movimento circular e lock
    farmMovementConnection = RunService.RenderStepped:Connect(function()
        if not farmKillsActive then 
            if farmMovementConnection then
                farmMovementConnection:Disconnect()
                farmMovementConnection = nil
            end
            return 
        end
        
        if targetPlayer and targetPlayer.Character and player.Character then
            local root = player.Character:FindFirstChild("HumanoidRootPart")
            local targetRoot = targetPlayer.Character:FindFirstChild("HumanoidRootPart")
            
            if root and targetRoot then
                local angle = tick() * farmVelocity
                local offset = Vector3.new(
                    math.cos(angle) * farmDistance, 
                    2, 
                    math.sin(angle) * farmDistance
                )
                root.CFrame = CFrame.new(targetRoot.Position + offset, targetRoot.Position)
                
                -- Lock camera
                local camera = workspace.CurrentCamera
                camera.CFrame = CFrame.new(camera.CFrame.Position, targetRoot.Position)
            end
        end
    end)
    
    return true
end

local function stopFarmKills()
    farmKillsActive = false
    if farmThread then
        task.cancel(farmThread)
        farmThread = nil
    end
    if farmMovementConnection then
        farmMovementConnection:Disconnect()
        farmMovementConnection = nil
    end
end

local FarmToggle = Tabs.FarmKills:AddToggle("FarmToggle", {
    Title = "🔥 Farm Kills",
    Description = "Ativa/desativa o farm automático",
    Default = false,
    Callback = function(Value)
        if Value then
            farmKillsActive = true
            local success = startFarmKills()
            if success then
                Fluent:Notify({
                    Title = "Farm Ativado!",
                    Content = "Farmando: " .. (targetPlayer and targetPlayer.Name or "Nenhum"),
                    Duration = 3
                })
            else
                FarmToggle:SetValue(false)
            end
        else
            stopFarmKills()
            Fluent:Notify({
                Title = "Farm Desativado",
                Content = "Farm de kills pausado",
                Duration = 3
            })
        end
    end
})

local DistanceSlider = Tabs.FarmKills:AddSlider("DistanceSlider", {
    Title = "📏 Distância do Alvo",
    Description = "Distância para circular ao redor do alvo",
    Default = 10,
    Min = 5,
    Max = 30,
    Rounding = 0,
    Callback = function(Value)
        farmDistance = Value
    end
})

local VelocitySlider = Tabs.FarmKills:AddSlider("VelocitySlider", {
    Title = "🌀 Velocidade de Giro",
    Description = "Velocidade de rotação ao redor do alvo",
    Default = 30,
    Min = 10,
    Max = 100,
    Rounding = 0,
    Callback = function(Value)
        farmVelocity = Value
    end
})

Tabs.FarmKills:AddParagraph({
    Title = "ℹ️ Nota",
    Content = "Use o toggle acima para pausar/continuar o farm. A tecla P não funciona neste modo."
})

-- ========================================
-- ABA ANTI STUN (COM PLAYER CONFIG)
-- ========================================

Tabs.AntiStun:AddParagraph({
    Title = "⚡ Anti Stun + Configurações",
    Content = "Faz o boneco deslizar suavemente e configure velocidade/pulo"
})

Tabs.AntiStun:AddParagraph({
    Title = "👤 Informações do Jogador",
    Content = "Nome: " .. player.Name .. "\nDisplay: " .. player.DisplayName .. "\nID: " .. tostring(player.UserId)
})

-- Aplicar configurações
local function applyPlayerSettings()
    local char = player.Character
    if char then
        local humanoid = char:FindFirstChild("Humanoid")
        if humanoid then
            humanoid.WalkSpeed = walkSpeed
            humanoid.JumpPower = jumpPower
        end
    end
end

-- Aplicar sempre ao spawnar
player.CharacterAdded:Connect(function(char)
    wait(0.5)
    applyPlayerSettings()
end)

-- Loop para manter ativo sempre
RunService.Heartbeat:Connect(function()
    pcall(applyPlayerSettings)
end)

local WalkSpeedSlider = Tabs.AntiStun:AddSlider("WalkSpeedSlider", {
    Title = "🏃 Velocidade de Caminhada",
    Description = "Ajuste a velocidade (fica ativo sempre)",
    Default = 16,
    Min = 16,
    Max = 200,
    Rounding = 0,
    Callback = function(Value)
        walkSpeed = Value
        applyPlayerSettings()
    end
})

local JumpPowerSlider = Tabs.AntiStun:AddSlider("JumpPowerSlider", {
    Title = "🦘 Altura do Pulo",
    Description = "Ajuste a altura do pulo (permanente até mudar)",
    Default = 50,
    Min = 50,
    Max = 300,
    Rounding = 0,
    Callback = function(Value)
        jumpPower = Value
        applyPlayerSettings()
    end
})

Tabs.AntiStun:AddParagraph({
    Title = "ℹ️ Importante",
    Content = "As configurações ficam ativas SEMPRE, mesmo após morrer. Mude os valores para alterar."
})

-- ========================================
-- ABA CAOS TOTAL
-- ========================================

Tabs.CaosTotal:AddParagraph({
    Title = "💀 MODO CAOS TOTAL 💀",
    Content = "⚠️ MUITO IMPORTANTE ⚠️\n\nQuer ser TÓXICO e DESTRUIR com esse jogo?\nAtivar isso 🤏👇👌👍"
})

local caosAtivo = false
local caosThread = nil

local function ativarCaosTotal()
    -- Ativa TUDO de uma vez
    -- Fling
    if not hiddenfling then
        FlingToggle:SetValue(true)
    end
    
    -- Fling All
    if not flingAllEnabled then
        FlingAllToggle:SetValue(true)
    end
    
    -- Velocidade máxima
    walkSpeed = 200
    WalkSpeedSlider:SetValue(200)
    
    -- Pulo máximo
    jumpPower = 300
    JumpPowerSlider:SetValue(300)
    
    -- Auto farm no primeiro player disponível
    if not targetPlayer then
        local players = updatePlayerList()
        if #players > 0 then
            targetPlayer = Players:FindFirstChild(players[1])
        end
    end
    
    if targetPlayer and not farmKillsActive then
        FarmToggle:SetValue(true)
    end
    
    -- Spam extremo de tudo
    caosThread = task.spawn(function()
        while caosAtivo do
            -- Spam todas as teclas
            for i = 1, 9 do
                pcall(function()
                    local keyName = tostring(i)
                    if keyName == "1" then keyName = "One"
                    elseif keyName == "2" then keyName = "Two"
                    elseif keyName == "3" then keyName = "Three"
                    elseif keyName == "4" then keyName = "Four"
                    elseif keyName == "5" then keyName = "Five"
                    elseif keyName == "6" then keyName = "Six"
                    elseif keyName == "7" then keyName = "Seven"
                    elseif keyName == "8" then keyName = "Eight"
                    elseif keyName == "9" then keyName = "Nine"
                    end
                    VirtualInputManager:SendKeyEvent(true, Enum.KeyCode[keyName], false, game)
                    task.wait(0.001)
                    VirtualInputManager:SendKeyEvent(false, Enum.KeyCode[keyName], false, game)
                end)
            end
            
            -- Spam click ultra rápido
            for i = 1, 10 do
                VirtualInputManager:SendMouseButtonEvent(0, 0, 0, true, game, 0)
                VirtualInputManager:SendMouseButtonEvent(0, 0, 0, false, game, 0)
            end
            
            task.wait(0.01)
        end
    end)
end

local function desativarCaosTotal()
    caosAtivo = false
    if caosThread then
        task.cancel(caosThread)
        caosThread = nil
    end
end

local CaosToggle = Tabs.CaosTotal:AddToggle("CaosToggle", {
    Title = "💣 ATIVAR CAOS TOTAL 💣",
    Description = "ATIVA TUDO DE UMA VEZ - MODO DESTRUIÇÃO MÁXIMA",
    Default = false,
    Callback = function(Value)
        if Value then
            caosAtivo = true
            ativarCaosTotal()
            Fluent:Notify({
                Title = "💀 CAOS ATIVADO! 💀",
                Content = "MODO DESTRUIÇÃO TOTAL LIGADO!",
                Duration = 5
            })
        else
            desativarCaosTotal()
            Fluent:Notify({
                Title = "Caos Desativado",
                Content = "Voltando ao normal...",
                Duration = 3
            })
        end
    end
})

Tabs.CaosTotal:AddParagraph({
    Title = "⚠️ AVISO FINAL",
    Content = "Este modo ativa:\n• Fling máximo\n• Fling em todos\n• Velocidade 200\n• Pulo 300\n• Farm automático\n• Spam extremo de tudo\n\nUSE POR SUA CONTA E RISCO!"
})

-- ========================================
-- NOTIFICAÇÃO INICIAL
-- ========================================
Fluent:Notify({
    Title = "TSB Contra KaratekaFF",
    Content = "Hub carregado! Use INSERT para minimizar",
    Duration = 5
})

print("✅ TSB Contra KaratekaFF carregado!")
print("📌 Pressione INSERT para minimizar")
