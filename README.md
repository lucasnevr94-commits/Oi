--[[
    Developer & QA Console - Sistema de Depuração Profissional
    Versão: 2.0
    Autor: Assistente Técnico
    Descrição: Interface completa para desenvolvimento e testes
    Uso: Apenas em ambiente de desenvolvimento
--]]

local DeveloperConsole = {}
local isStudio = game:GetService("RunService"):IsStudio()

if not isStudio then
    print("[DevConsole] Desativado - Execute apenas no Studio")
    return
end

-- Serviços
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local CollectionService = game:GetService("CollectionService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local HttpService = game:GetService("HttpService")

-- Configurações da Interface
local UI_CONFIG = {
    Theme = {
        Background = Color3.fromRGB(20, 20, 25),
        Surface = Color3.fromRGB(30, 30, 38),
        Accent = Color3.fromRGB(0, 120, 255),
        Success = Color3.fromRGB(46, 204, 113),
        Warning = Color3.fromRGB(241, 196, 15),
        Danger = Color3.fromRGB(231, 76, 60),
        Text = Color3.fromRGB(255, 255, 255),
        TextSecondary = Color3.fromRGB(150, 150, 160)
    },
    Animation = {
        Duration = 0.3,
        EasingStyle = Enum.EasingStyle.Quad,
        EasingDirection = Enum.EasingDirection.Out
    }
}

-- Estado do Sistema
local SystemState = {
    DebugTools = {
        Hitboxes = false,
        CombatLog = false,
        Performance = false,
        NetworkMonitor = false
    },
    Stats = {},
    CombatEvents = {},
    PerformanceData = {
        FPS = 60,
        Ping = 0,
        Memory = 0,
        PhysicsTime = 0
    }
}

-- ================================
-- Criação da Interface Principal
-- ================================

local function createMainScreenGui()
    -- ScreenGui principal
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "DevConsole"
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    screenGui.ResetOnSpawn = false
    screenGui.Parent = game:GetService("CoreGui")
    
    -- Frame principal (container)
    local mainFrame = Instance.new("Frame")
    mainFrame.Name = "MainFrame"
    mainFrame.Size = UDim2.new(0, 380, 0, 600)
    mainFrame.Position = UDim2.new(0, 20, 0, 20)
    mainFrame.BackgroundColor3 = UI_CONFIG.Theme.Background
    mainFrame.BorderSizePixel = 0
    mainFrame.ClipsDescendants = true
    mainFrame.Parent = screenGui
    
    -- Sombra
    local shadow = Instance.new("Frame")
    shadow.Name = "Shadow"
    shadow.Size = UDim2.new(1, 0, 1, 0)
    shadow.Position = UDim2.new(0, 5, 0, 5)
    shadow.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    shadow.BackgroundTransparency = 0.7
    shadow.BorderSizePixel = 0
    shadow.Parent = mainFrame
    
    -- Título com gradiente
    local titleBar = Instance.new("Frame")
    titleBar.Name = "TitleBar"
    titleBar.Size = UDim2.new(1, 0, 0, 40)
    titleBar.BackgroundColor3 = UI_CONFIG.Theme.Surface
    titleBar.BorderSizePixel = 0
    titleBar.Parent = mainFrame
    
    local titleGradient = Instance.new("UIGradient")
    titleGradient.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, UI_CONFIG.Theme.Surface),
        ColorSequenceKeypoint.new(1, UI_CONFIG.Theme.Background)
    })
    titleGradient.Parent = titleBar
    
    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, -40, 1, 0)
    title.Position = UDim2.new(0, 15, 0, 0)
    title.BackgroundTransparency = 1
    title.Text = "🛠️ Developer & QA Console"
    title.TextColor3 = UI_CONFIG.Theme.Text
    title.TextSize = 16
    title.Font = Enum.Font.GothamSemibold
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Parent = titleBar
    
    -- Botão minimizar
    local minimizeBtn = Instance.new("TextButton")
    minimizeBtn.Size = UDim2.new(0, 30, 0, 30)
    minimizeBtn.Position = UDim2.new(1, -80, 0, 5)
    minimizeBtn.BackgroundColor3 = UI_CONFIG.Theme.Surface
    minimizeBtn.BackgroundTransparency = 0.5
    minimizeBtn.Text = "−"
    minimizeBtn.TextColor3 = UI_CONFIG.Theme.Text
    minimizeBtn.TextSize = 20
    minimizeBtn.Font = Enum.Font.GothamBold
    minimizeBtn.BorderSizePixel = 0
    minimizeBtn.Parent = titleBar
    
    -- Botão fechar
    local closeBtn = Instance.new("TextButton")
    closeBtn.Size = UDim2.new(0, 30, 0, 30)
    closeBtn.Position = UDim2.new(1, -40, 0, 5)
    closeBtn.BackgroundColor3 = UI_CONFIG.Theme.Surface
    closeBtn.BackgroundTransparency = 0.5
    closeBtn.Text = "✕"
    closeBtn.TextColor3 = UI_CONFIG.Theme.Danger
    closeBtn.TextSize = 16
    closeBtn.Font = Enum.Font.GothamBold
    closeBtn.BorderSizePixel = 0
    closeBtn.Parent = titleBar
    
    -- Container de abas (TabBar)
    local tabBar = Instance.new("Frame")
    tabBar.Name = "TabBar"
    tabBar.Size = UDim2.new(1, 0, 0, 45)
    tabBar.Position = UDim2.new(0, 0, 0, 40)
    tabBar.BackgroundColor3 = UI_CONFIG.Theme.Surface
    tabBar.BorderSizePixel = 0
    tabBar.Parent = mainFrame
    
    -- Container de conteúdo
    local contentContainer = Instance.new("ScrollingFrame")
    contentContainer.Name = "ContentContainer"
    contentContainer.Size = UDim2.new(1, 0, 1, -85)
    contentContainer.Position = UDim2.new(0, 0, 0, 85)
    contentContainer.BackgroundColor3 = UI_CONFIG.Theme.Background
    contentContainer.BackgroundTransparency = 0.5
    contentContainer.BorderSizePixel = 0
    contentContainer.CanvasSize = UDim2.new(0, 0, 0, 0)
    contentContainer.ScrollBarThickness = 6
    contentContainer.ScrollBarImageColor3 = UI_CONFIG.Theme.Accent
    contentContainer.Parent = mainFrame
    
    -- UIListLayout para conteúdo
    local contentLayout = Instance.new("UIListLayout")
    contentLayout.Padding = UDim.new(0, 10)
    contentLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    contentLayout.SortOrder = Enum.SortOrder.LayoutOrder
    contentLayout.Parent = contentContainer
    
    return screenGui, mainFrame, contentContainer, contentLayout
end

-- ================================
-- Criação dos Módulos da Interface
-- ================================

-- Módulo: Ferramentas de Debug
local function createDebugToolsTab(parent)
    local container = Instance.new("Frame")
    container.Name = "DebugToolsTab"
    container.Size = UDim2.new(1, -20, 0, 0)
    container.BackgroundTransparency = 1
    container.AutomaticSize = Enum.AutomaticSize.Y
    container.Parent = parent
    
    local layout = Instance.new("UIListLayout")
    layout.Padding = UDim.new(0, 8)
    layout.SortOrder = Enum.SortOrder.LayoutOrder
    layout.Parent = container
    
    -- Título da seção
    local sectionTitle = Instance.new("TextLabel")
    sectionTitle.Size = UDim2.new(1, 0, 0, 30)
    sectionTitle.BackgroundTransparency = 1
    sectionTitle.Text = "🔧 Ferramentas de Depuração"
    sectionTitle.TextColor3 = UI_CONFIG.Theme.Text
    sectionTitle.TextSize = 14
    sectionTitle.Font = Enum.Font.GothamSemibold
    sectionTitle.TextXAlignment = Enum.TextXAlignment.Left
    sectionTitle.LayoutOrder = 1
    sectionTitle.Parent = container
    
    -- Botão Hitboxes
    local hitboxBtn = createToggleButton("🎯 Visualizar Hitboxes", "Mostra hitboxes de teste", "Hitboxes")
    hitboxBtn.LayoutOrder = 2
    hitboxBtn.Parent = container
    
    -- Botão Combat Log
    local combatLogBtn = createToggleButton("📊 Log de Combate", "Monitora eventos de combate", "CombatLog")
    combatLogBtn.LayoutOrder = 3
    combatLogBtn.Parent = container
    
    -- Botão Performance Monitor
    local perfBtn = createToggleButton("📈 Monitor de Performance", "Mostra FPS, ping e uso de memória", "Performance")
    perfBtn.LayoutOrder = 4
    perfBtn.Parent = container
    
    -- Botão Network Monitor
    local networkBtn = createToggleButton("🌐 Monitor de Rede", "Rastreia RemoteEvents", "NetworkMonitor")
    networkBtn.LayoutOrder = 5
    networkBtn.Parent = container
    
    -- Botão Limpar Logs
    local clearBtn = Instance.new("TextButton")
    clearBtn.Size = UDim2.new(1, 0, 0, 40)
    clearBtn.BackgroundColor3 = UI_CONFIG.Theme.Surface
    clearBtn.Text = "🗑️ Limpar Todos os Logs"
    clearBtn.TextColor3 = UI_CONFIG.Theme.Warning
    clearBtn.TextSize = 14
    clearBtn.Font = Enum.Font.GothamMedium
    clearBtn.BorderSizePixel = 0
    clearBtn.LayoutOrder = 6
    clearBtn.Parent = container
    
    clearBtn.MouseButton1Click:Connect(function()
        SystemState.CombatEvents = {}
        updateCombatLogDisplay()
    end)
    
    return container
end

-- Função para criar botão toggle
function createToggleButton(text, description, toolName)
    local frame = Instance.new("Frame")
    frame.Name = toolName .. "Btn"
    frame.Size = UDim2.new(1, 0, 0, 50)
    frame.BackgroundColor3 = UI_CONFIG.Theme.Surface
    frame.BorderSizePixel = 0
    frame.AutomaticSize = Enum.AutomaticSize.Y
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = frame
    
    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, -60, 0, 25)
    title.Position = UDim2.new(0, 15, 0, 8)
    title.BackgroundTransparency = 1
    title.Text = text
    title.TextColor3 = UI_CONFIG.Theme.Text
    title.TextSize = 14
    title.Font = Enum.Font.GothamSemibold
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Parent = frame
    
    local desc = Instance.new("TextLabel")
    desc.Size = UDim2.new(1, -60, 0, 20)
    desc.Position = UDim2.new(0, 15, 0, 28)
    desc.BackgroundTransparency = 1
    desc.Text = description
    desc.TextColor3 = UI_CONFIG.Theme.TextSecondary
    desc.TextSize = 11
    desc.Font = Enum.Font.Gotham
    desc.TextXAlignment = Enum.TextXAlignment.Left
    desc.Parent = frame
    
    local toggle = Instance.new("TextButton")
    toggle.Size = UDim2.new(0, 40, 0, 40)
    toggle.Position = UDim2.new(1, -50, 0, 5)
    toggle.BackgroundColor3 = UI_CONFIG.Theme.Background
    toggle.Text = "OFF"
    toggle.TextColor3 = UI_CONFIG.Theme.Danger
    toggle.TextSize = 12
    toggle.Font = Enum.Font.GothamBold
    toggle.BorderSizePixel = 0
    toggle.Parent = frame
    
    local corner2 = Instance.new("UICorner")
    corner2.CornerRadius = UDim.new(0, 6)
    corner2.Parent = toggle
    
    toggle.MouseButton1Click:Connect(function()
        SystemState.DebugTools[toolName] = not SystemState.DebugTools[toolName]
        toggle.Text = SystemState.DebugTools[toolName] and "ON" or "OFF"
        toggle.TextColor3 = SystemState.DebugTools[toolName] and UI_CONFIG.Theme.Success or UI_CONFIG.Theme.Danger
        
        -- Chamar função correspondente
        if toolName == "Hitboxes" then
            toggleHitboxes(SystemState.DebugTools[toolName])
        elseif toolName == "CombatLog" then
            toggleCombatLog(SystemState.DebugTools[toolName])
        elseif toolName == "Performance" then
            togglePerformanceMonitor(SystemState.DebugTools[toolName])
        end
    end)
    
    return frame
end

-- Módulo: Estatísticas do Personagem
local function createCharacterStatsTab(parent)
    local container = Instance.new("Frame")
    container.Name = "CharacterStatsTab"
    container.Size = UDim2.new(1, -20, 0, 0)
    container.BackgroundTransparency = 1
    container.AutomaticSize = Enum.AutomaticSize.Y
    container.Parent = parent
    
    local layout = Instance.new("UIListLayout")
    layout.Padding = UDim.new(0, 8)
    layout.SortOrder = Enum.SortOrder.LayoutOrder
    layout.Parent = container
    
    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0, 30)
    title.BackgroundTransparency = 1
    title.Text = "👤 Estatísticas do Jogador"
    title.TextColor3 = UI_CONFIG.Theme.Text
    title.TextSize = 14
    title.Font = Enum.Font.GothamSemibold
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.LayoutOrder = 1
    title.Parent = container
    
    -- Container para stats dinâmicas
    local statsContainer = Instance.new("Frame")
    statsContainer.Name = "StatsContainer"
    statsContainer.Size = UDim2.new(1, 0, 0, 0)
    statsContainer.BackgroundTransparency = 1
    statsContainer.AutomaticSize = Enum.AutomaticSize.Y
    statsContainer.LayoutOrder = 2
    statsContainer.Parent = container
    
    local statsLayout = Instance.new("UIListLayout")
    statsLayout.Padding = UDim.new(0, 6)
    statsLayout.SortOrder = Enum.SortOrder.LayoutOrder
    statsLayout.Parent = statsContainer
    
    -- Função para atualizar stats
    local function updateStats()
        -- Limpar container
        for _, child in ipairs(statsContainer:GetChildren()) do
            if child:IsA("Frame") then
                child:Destroy()
            end
        end
        
        local player = Players.LocalPlayer
        if not player or not player.Character then return end
        
        local character = player.Character
        local humanoid = character:FindFirstChild("Humanoid")
        local rootPart = character:FindFirstChild("HumanoidRootPart")
        
        local stats = {
            {label = "💚 Saúde", value = humanoid and string.format("%.0f/%.0f", humanoid.Health, humanoid.MaxHealth) or "N/A", color = UI_CONFIG.Theme.Success},
            {label = "⚡ Energia", value = humanoid and string.format("%.0f", humanoid.Energy) or "N/A", color = UI_CONFIG.Theme.Accent},
            {label = "📍 Posição", value = rootPart and string.format("X:%.1f Y:%.1f Z:%.1f", rootPart.Position.X, rootPart.Position.Y, rootPart.Position.Z) or "N/A", color = UI_CONFIG.Theme.Text},
            {label = "🏃 Velocidade", value = humanoid and string.format("%.1f/s", humanoid.WalkSpeed) or "N/A", color = UI_CONFIG.Theme.Text},
            {label = "🪂 Gravidade", value = humanoid and string.format("%.1f", humanoid.GravityScale) or "N/A", color = UI_CONFIG.Theme.Text},
            {label = "💀 Mortes", value = player:GetAttribute("Deaths") or 0, color = UI_CONFIG.Theme.Warning},
            {label = "⚔️ K/D", value = player:GetAttribute("KDRatio") or 0, color = UI_CONFIG.Theme.Accent}
        }
        
        for _, stat in ipairs(stats) do
            local statFrame = Instance.new("Frame")
            statFrame.Size = UDim2.new(1, 0, 0, 40)
            statFrame.BackgroundColor3 = UI_CONFIG.Theme.Surface
            statFrame.BorderSizePixel = 0
            statFrame.LayoutOrder = 1
            
            local corner = Instance.new("UICorner")
            corner.CornerRadius = UDim.new(0, 6)
            corner.Parent = statFrame
            
            local label = Instance.new("TextLabel")
            label.Size = UDim2.new(0.5, -10, 1, 0)
            label.Position = UDim2.new(0, 15, 0, 0)
            label.BackgroundTransparency = 1
            label.Text = stat.label
            label.TextColor3 = UI_CONFIG.Theme.TextSecondary
            label.TextSize = 13
            label.Font = Enum.Font.Gotham
            label.TextXAlignment = Enum.TextXAlignment.Left
            label.Parent = statFrame
            
            local value = Instance.new("TextLabel")
            value.Size = UDim2.new(0.5, -10, 1, 0)
            value.Position = UDim2.new(0.5, 5, 0, 0)
            value.BackgroundTransparency = 1
            value.Text = tostring(stat.value)
            value.TextColor3 = stat.color
            value.TextSize = 14
            value.Font = Enum.Font.GothamSemibold
            value.TextXAlignment = Enum.TextXAlignment.Right
            value.Parent = statFrame
            
            statFrame.Parent = statsContainer
        end
    end
    
    -- Atualizar stats periodicamente
    task.spawn(function()
        while true do
            if SystemState.DebugTools.Performance and statsContainer.Parent then
                updateStats()
            end
            task.wait(0.5)
        end
    end)
    
    return container
end

-- Módulo: Logs de Combate
local function createCombatLogTab(parent)
    local container = Instance.new("Frame")
    container.Name = "CombatLogTab"
    container.Size = UDim2.new(1, -20, 0, 400)
    container.BackgroundColor3 = UI_CONFIG.Theme.Surface
    container.BorderSizePixel = 0
    container.Parent = parent
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = container
    
    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0, 30)
    title.Position = UDim2.new(0, 15, 0, 5)
    title.BackgroundTransparency = 1
    title.Text = "⚔️ Logs de Combate"
    title.TextColor3 = UI_CONFIG.Theme.Text
    title.TextSize = 14
    title.Font = Enum.Font.GothamSemibold
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Parent = container
    
    local logScroll = Instance.new("ScrollingFrame")
    logScroll.Size = UDim2.new(1, -20, 1, -50)
    logScroll.Position = UDim2.new(0, 10, 0, 40)
    logScroll.BackgroundColor3 = UI_CONFIG.Theme.Background
    logScroll.BackgroundTransparency = 0.3
    logScroll.BorderSizePixel = 0
    logScroll.CanvasSize = UDim2.new(0, 0, 0, 0)
    logScroll.ScrollBarThickness = 6
    logScroll.Parent = container
    
    local logLayout = Instance.new("UIListLayout")
    logLayout.Padding = UDim.new(0, 4)
    logLayout.SortOrder = Enum.SortOrder.LayoutOrder
    logLayout.Parent = logScroll
    
    -- Função global para atualizar display de log
    _G.updateCombatLogDisplay = function()
        -- Limpar logs antigos
        for _, child in ipairs(logScroll:GetChildren()) do
            if child:IsA("TextLabel") then
                child:Destroy()
            end
        end
        
        -- Mostrar últimos 50 logs
        for i = math.max(1, #SystemState.CombatEvents - 49), #SystemState.CombatEvents do
            local event = SystemState.CombatEvents[i]
            local logEntry = Instance.new("TextLabel")
            logEntry.Size = UDim2.new(1, -10, 0, 25)
            logEntry.BackgroundTransparency = 1
            logEntry.Text = string.format("[%s] %s → %s: %.1f dano", 
                event.Time or "00:00", 
                event.Attacker or "?", 
                event.Victim or "?", 
                event.Damage or 0)
            logEntry.TextColor3 = event.Type == "Hit" and UI_CONFIG.Theme.Danger or UI_CONFIG.Theme.Success
            logEntry.TextSize = 11
            logEntry.Font = Enum.Font.Gotham
            logEntry.TextXAlignment = Enum.TextXAlignment.Left
            logEntry.LayoutOrder = i
            logEntry.Parent = logScroll
        end
        
        -- Ajustar canvas size
        logScroll.CanvasSize = UDim2.new(0, 0, 0, #SystemState.CombatEvents * 25)
    end
    
    return container
end

-- Módulo: Monitor de Performance
local function createPerformanceTab(parent)
    local container = Instance.new("Frame")
    container.Name = "PerformanceTab"
    container.Size = UDim2.new(1, -20, 0, 0)
    container.BackgroundTransparency = 1
    container.AutomaticSize = Enum.AutomaticSize.Y
    container.Parent = parent
    
    local layout = Instance.new("UIListLayout")
    layout.Padding = UDim.new(0, 8)
    layout.SortOrder = Enum.SortOrder.LayoutOrder
    layout.Parent = container
    
    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0, 30)
    title.BackgroundTransparency = 1
    title.Text = "📊 Monitor de Performance"
    title.TextColor3 = UI_CONFIG.Theme.Text
    title.TextSize = 14
    title.Font = Enum.Font.GothamSemibold
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.LayoutOrder = 1
    title.Parent = container
    
    -- FPS Meter
    local fpsFrame = createMetricCard("🎮 FPS", "0", UI_CONFIG.Theme.Accent)
    fpsFrame.LayoutOrder = 2
    fpsFrame.Parent = container
    
    -- Ping Meter
    local pingFrame = createMetricCard("🌐 Ping", "0 ms", UI_CONFIG.Theme.Accent)
    pingFrame.LayoutOrder = 3
    pingFrame.Parent = container
    
    -- Memory Usage
    local memFrame = createMetricCard("💾 Memória", "0 MB", UI_CONFIG.Th
