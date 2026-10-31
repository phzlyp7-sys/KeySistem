-- language: Luau, target: Roblox MOBILE
-- Key Sistem 7zK — v3.4 (Estilo WindUI)

local API_URL     = "https://get-key.iphz7.workers.dev"
local URL_FREE    = "https://raw.githubusercontent.com/phzlyp7-sys/7Zk-FREE/refs/heads/main/FREE.md"
local URL_PREMIUM = "https://raw.githubusercontent.com/phzlyp7-sys/7zK-PREMIUM/refs/heads/main/Premium.md"

--=====================================================================
-- SERVIÇOS
--=====================================================================
local Players          = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService     = game:GetService("TweenService")
local HttpService      = game:GetService("HttpService")
local SoundService     = game:GetService("SoundService")
local Debris           = game:GetService("Debris")
local LocalPlayer      = Players.LocalPlayer

--=====================================================================
-- PALETA (IDÊNTICA À WINDUI)
--=====================================================================
local C_BG         = Color3.fromRGB(15, 15, 18)    -- Fundo geral
local C_CARD       = Color3.fromRGB(24, 24, 28)    -- Fundo do card
local C_ELEMENT    = Color3.fromRGB(36, 36, 40)    -- Fundo dos botões/inputs
local C_ELEMENT_H  = Color3.fromRGB(46, 46, 50)    -- Hover dos botões
local C_STROKE     = Color3.fromRGB(45, 45, 50)    -- Bordas
local C_TEXT       = Color3.fromRGB(255, 255, 255) -- Texto principal
local C_MUTED      = Color3.fromRGB(160, 160, 165) -- Texto secundário
local C_DIM        = Color3.fromRGB(100, 100, 105) -- Placeholder
local C_NEON       = Color3.fromRGB(40, 210, 120)  -- Verde WindUI
local C_NEON_H     = Color3.fromRGB(50, 230, 135)  -- Verde hover
local C_ERR        = Color3.fromRGB(235, 65, 65)   -- Vermelho erro
local C_RED_BG     = Color3.fromRGB(45, 25, 25)    -- Vermelho escuro (Obter Key)
local C_RED_BG_H   = Color3.fromRGB(60, 30, 30)    -- Vermelho escuro hover
local C_RED_TEXT   = Color3.fromRGB(255, 100, 100) -- Texto vermelho

--=====================================================================
-- ÁUDIO
--=====================================================================
local SOUNDS = {
    open    = "rbxasset://sounds/electronicpingshort.wav",
    click   = "rbxasset://sounds/switch.wav",
    success = "rbxasset://sounds/electronicpingshort.wav",
    error   = "rbxasset://sounds/snap.wav",
}

local lastSoundAt = 0
local SOUND_COOLDOWN = 0.04

local function playSound(name, volume, pitch)
    local id = SOUNDS[name]
    if not id then return end
    local agora = tick()
    if agora - lastSoundAt < SOUND_COOLDOWN then return end
    lastSoundAt = agora
    pcall(function()
        local s = Instance.new("Sound")
        s.SoundId = id
        s.Volume = volume or 0.4
        s.PlaybackSpeed = pitch or 1
        s.Parent = SoundService
        s:Play()
        Debris:AddItem(s, 3)
    end)
end

local function playOpenSound() playSound("open", 0.3, 1.0) end
local function playClick() playSound("click", 0.4, 1.0) end
local function playSuccess() playSound("success", 0.5, 1.2) end
local function playError() playSound("error", 0.5, 0.8) end

--=====================================================================
-- HWID
--=====================================================================
local function getHwid()
    for _, name in ipairs({"gethwid", "get_hwid", "getHWID", "hwid"}) do
        local fn = _G[name]
        if type(fn) == "function" then
            local ok, v = pcall(fn)
            if ok and v then return tostring(v) end
        end
    end
    local ok, id = pcall(function()
        return game:GetService("RbxAnalyticsService"):GetClientId()
    end)
    if ok and id then return id end
    return tostring(LocalPlayer.UserId) .. "|" .. tostring(game.PlaceId)
end

--=====================================================================
-- HTTP COM TIMEOUT
--=====================================================================
local function httpGetComTimeout(url, timeout)
    timeout = timeout or 8
    local result, done, ok = nil, false, false

    task.spawn(function()
        local success, resp = pcall(function() return game:HttpGet(url, true) end)
        ok = success
        result = resp
        done = true
    end)

    local start = tick()
    while not done and (tick() - start) < timeout do
        task.wait(0.05)
    end

    if not done then return false, nil, true end
    return ok, result, false
end

--=====================================================================
-- UI
--=====================================================================
local function criarUI()
    local pg = LocalPlayer:WaitForChild("PlayerGui")

    local gui = Instance.new("ScreenGui")
    gui.Name = "7zK_KeySystem"
    gui.ResetOnSpawn = false
    gui.DisplayOrder = 9999
    gui.IgnoreGuiInset = true
    gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    gui.Parent = pg

    -- OVERLAY ESCURO (Fundo da tela)
    local overlay = Instance.new("Frame", gui)
    overlay.Size = UDim2.new(1, 0, 1, 0)
    overlay.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    overlay.BackgroundTransparency = 0.3
    overlay.BorderSizePixel = 0
    overlay.ZIndex = 1

    -- CARD PRINCIPAL
    local card = Instance.new("Frame")
    card.Size = UDim2.new(0, 360, 0, 310)
    card.Position = UDim2.new(0.5, -180, 0.5, -155)
    card.BackgroundColor3 = C_CARD
    card.BorderSizePixel = 0
    card.ZIndex = 10
    card.Parent = gui

    local cc = Instance.new("UICorner", card)
    cc.CornerRadius = UDim.new(0, 12)

    local cs = Instance.new("UIStroke", card)
    cs.Color = C_STROKE
    cs.Thickness = 1
    cs.Transparency = 0.5

    -- HEADER
    local header = Instance.new("Frame", card)
    header.Size = UDim2.new(1, 0, 0, 65)
    header.Position = UDim2.new(0, 0, 0, 0)
    header.BackgroundTransparency = 1
    header.ZIndex = 11

    -- ÍCONE (Logo)
    local icon = Instance.new("ImageLabel", header)
    icon.Size = UDim2.new(0, 24, 0, 24)
    icon.Position = UDim2.new(0, 20, 0, 20)
    icon.BackgroundTransparency = 1
    icon.Image = "rbxassetid://4483345998" -- Ícone padrão WindUI
    icon.ImageColor3 = C_TEXT
    icon.ZIndex = 12

    -- TÍTULO E SUBTÍTULO
    local title = Instance.new("TextLabel", header)
    title.Size = UDim2.new(1, -70, 0, 20)
    title.Position = UDim2.new(0, 54, 0, 16)
    title.BackgroundTransparency = 1
    title.Text = "7zK | Key System"
    title.TextColor3 = C_TEXT
    title.TextSize = 16
    title.Font = Enum.Font.GothamBold
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.ZIndex = 12

    local subtitle = Instance.new("TextLabel", header)
    subtitle.Size = UDim2.new(1, -70, 0, 16)
    subtitle.Position = UDim2.new(0, 54, 0, 36)
    subtitle.BackgroundTransparency = 1
    subtitle.Text = "Autenticação necessária"
    subtitle.TextColor3 = C_MUTED
    subtitle.TextSize = 12
    subtitle.Font = Enum.Font.Gotham
    subtitle.TextXAlignment = Enum.TextXAlignment.Left
    subtitle.ZIndex = 12

    -- BOTÃO X (FECHAR)
    local closeBtn = Instance.new("TextButton", header)
    closeBtn.Size = UDim2.new(0, 30, 0, 30)
    closeBtn.Position = UDim2.new(1, -40, 0, 17)
    closeBtn.BackgroundTransparency = 1
    closeBtn.Text = "×"
    closeBtn.TextColor3 = C_MUTED
    closeBtn.TextSize = 22
    closeBtn.Font = Enum.Font.GothamBold
    closeBtn.BorderSizePixel = 0
    closeBtn.AutoButtonColor = false
    closeBtn.ZIndex = 100 -- ZIndex alto para receber o clique

    closeBtn.MouseEnter:Connect(function() TweenService:Create(closeBtn, TweenInfo.new(0.1), {TextColor3 = C_ERR}):Play() end)
    closeBtn.MouseLeave:Connect(function() TweenService:Create(closeBtn, TweenInfo.new(0.1), {TextColor3 = C_MUTED}):Play() end)

    -- ÁREA DE ARRASTAR (Apenas no lado esquerdo do header)
    local headerArea = Instance.new("TextButton", header)
    headerArea.Size = UDim2.new(1, -50, 1, 0)
    headerArea.Position = UDim2.new(0, 0, 0, 0)
    headerArea.BackgroundTransparency = 1
    headerArea.Text = ""
    headerArea.ZIndex = 20

    -- DIVIDER
    local divider = Instance.new("Frame", card)
    divider.Size = UDim2.new(1, -40, 0, 1)
    divider.Position = UDim2.new(0, 20, 0, 65)
    divider.BackgroundColor3 = C_STROKE
    divider.BorderSizePixel = 0
    divider.ZIndex = 12

    -- TELA 1 (ESCOLHA)
    local screen1 = Instance.new("Frame", card)
    screen1.Size = UDim2.new(1, 0, 0, 230)
    screen1.Position = UDim2.new(0, 0, 0, 70)
    screen1.BackgroundTransparency = 1
    screen1.ZIndex = 11

    local btnFree = Instance.new("TextButton", screen1)
    btnFree.Size = UDim2.new(1, -40, 0, 48)
    btnFree.Position = UDim2.new(0, 20, 0, 10)
    btnFree.BackgroundColor3 = C_ELEMENT
    btnFree.TextColor3 = C_TEXT
    btnFree.Text = "Entrar na Versão FREE"
    btnFree.TextSize = 14
    btnFree.Font = Enum.Font.GothamMedium
    btnFree.BorderSizePixel = 0
    btnFree.AutoButtonColor = false
    btnFree.ZIndex = 12

    local bfc = Instance.new("UICorner", btnFree)
    bfc.CornerRadius = UDim.new(0, 8)

    local btnKey = Instance.new("TextButton", screen1)
    btnKey.Size = UDim2.new(1, -40, 0, 48)
    btnKey.Position = UDim2.new(0, 20, 0, 68)
    btnKey.BackgroundColor3 = C_NEON
    btnKey.TextColor3 = Color3.fromRGB(10, 10, 10)
    btnKey.Text = "Inserir Key PREMIUM"
    btnKey.TextSize = 14
    btnKey.Font = Enum.Font.GothamBold
    btnKey.BorderSizePixel = 0
    btnKey.AutoButtonColor = false
    btnKey.ZIndex = 12

    local bkc = Instance.new("UICorner", btnKey)
    bkc.CornerRadius = UDim.new(0, 8)

    local keySub = Instance.new("TextLabel", screen1)
    keySub.Size = UDim2.new(1, -40, 0, 30)
    keySub.Position = UDim2.new(0, 20, 0, 130)
    keySub.BackgroundTransparency = 1
    keySub.Text = "Keys são obtidas no nosso site (link direto)"
    keySub.TextColor3 = C_MUTED
    keySub.TextSize = 12
    keySub.Font = Enum.Font.Gotham
    keySub.TextWrapped = true
    keySub.ZIndex = 12

    -- TELA 2 (KEY)
    local screen2 = Instance.new("Frame", card)
    screen2.Size = UDim2.new(1, 0, 0, 230)
    screen2.Position = UDim2.new(0, 0, 0, 70)
    screen2.BackgroundTransparency = 1
    screen2.Visible = false
    screen2.ZIndex = 11

    local labelKey = Instance.new("TextLabel", screen2)
    labelKey.Size = UDim2.new(1, -40, 0, 14)
    labelKey.Position = UDim2.new(0, 20, 0, 0)
    labelKey.BackgroundTransparency = 1
    labelKey.Text = "SUA CHAVE DE ACESSO"
    labelKey.TextColor3 = C_MUTED
    labelKey.TextSize = 11
    labelKey.Font = Enum.Font.GothamMedium
    labelKey.TextXAlignment = Enum.TextXAlignment.Left
    labelKey.ZIndex = 12

    local tb = Instance.new("TextBox", screen2)
    tb.Size = UDim2.new(1, -40, 0, 44)
    tb.Position = UDim2.new(0, 20, 0, 18)
    tb.BackgroundColor3 = C_ELEMENT
    tb.TextColor3 = C_TEXT
    tb.PlaceholderText = "Cole sua key aqui..."
    tb.PlaceholderColor3 = C_DIM
    tb.Text = ""
    tb.TextSize = 13
    tb.Font = Enum.Font.Gotham
    tb.BorderSizePixel = 0
    tb.ClearTextOnFocus = false
    tb.TextXAlignment = Enum.TextXAlignment.Left
    tb.ZIndex = 12

    local tbc = Instance.new("UICorner", tb)
    tbc.CornerRadius = UDim.new(0, 8)

    local tbp = Instance.new("UIPadding", tb)
    tbp.PaddingLeft = UDim.new(0, 12)
    tbp.PaddingRight = UDim.new(0, 12)

    local btnUsar = Instance.new("TextButton", screen2)
    btnUsar.Size = UDim2.new(1, -40, 0, 44)
    btnUsar.Position = UDim2.new(0, 20, 0, 72)
    btnUsar.BackgroundColor3 = C_NEON
    btnUsar.TextColor3 = Color3.fromRGB(10, 10, 10)
    btnUsar.Text = "Validar Key"
    btnUsar.TextSize = 14
    btnUsar.Font = Enum.Font.GothamBold
    btnUsar.BorderSizePixel = 0
    btnUsar.AutoButtonColor = false
    btnUsar.ZIndex = 12

    local bvc = Instance.new("UICorner", btnUsar)
    bvc.CornerRadius = UDim.new(0, 8)

    -- Botão Obter Key (Vermelho)
    local btnObter = Instance.new("TextButton", screen2)
    btnObter.Size = UDim2.new(1, -40, 0, 44)
    btnObter.Position = UDim2.new(0, 20, 0, 126)
    btnObter.BackgroundColor3 = C_RED_BG
    btnObter.TextColor3 = C_RED_TEXT
    btnObter.Text = "Obter Key (Link Direto)"
    btnObter.TextSize = 13
    btnObter.Font = Enum.Font.GothamBold
    btnObter.BorderSizePixel = 0
    btnObter.AutoButtonColor = false
    btnObter.ZIndex = 12

    local boc = Instance.new("UICorner", btnObter)
    boc.CornerRadius = UDim.new(0, 8)

    local bos = Instance.new("UIStroke", btnObter)
    bos.Color = C_ERR
    bos.Thickness = 1
    bos.Transparency = 0.8

    local btnVoltar = Instance.new("TextButton", screen2)
    btnVoltar.Size = UDim2.new(1, -40, 0, 36)
    btnVoltar.Position = UDim2.new(0, 20, 0, 180)
    btnVoltar.BackgroundColor3 = C_ELEMENT
    btnVoltar.TextColor3 = C_MUTED
    btnVoltar.Text = "Voltar"
    btnVoltar.TextSize = 12
    btnVoltar.Font = Enum.Font.GothamMedium
    btnVoltar.BorderSizePixel = 0
    btnVoltar.AutoButtonColor = false
    btnVoltar.ZIndex = 12

    local bvs2 = Instance.new("UICorner", btnVoltar)
    bvs2.CornerRadius = UDim.new(0, 8)

    -- STATUS FOOTER
    local statusText = Instance.new("TextLabel", card)
    statusText.Size = UDim2.new(1, -40, 0, 20)
    statusText.Position = UDim2.new(0, 20, 1, -28)
    statusText.BackgroundTransparency = 1
    statusText.Text = "Aguardando..."
    statusText.TextColor3 = C_MUTED
    statusText.TextSize = 11
    statusText.Font = Enum.Font.GothamMedium
    statusText.TextXAlignment = Enum.TextXAlignment.Left
    statusText.ZIndex = 12

    -- POPUP (CONFIRMAÇÃO DE FECHAMENTO)
    local popup = Instance.new("Frame", gui)
    popup.Size = UDim2.new(0, 280, 0, 140)
    popup.Position = UDim2.new(0.5, -140, 0.5, -70)
    popup.BackgroundColor3 = C_CARD
    popup.BorderSizePixel = 0
    popup.ZIndex = 30
    popup.Visible = false

    local pc = Instance.new("UICorner", popup)
    pc.CornerRadius = UDim.new(0, 12)

    local ps = Instance.new("UIStroke", popup)
    ps.Color = C_STROKE
    ps.Thickness = 1

    local popupTitle = Instance.new("TextLabel", popup)
    popupTitle.Size = UDim2.new(1, -40, 0, 30)
    popupTitle.Position = UDim2.new(0, 20, 0, 20)
    popupTitle.BackgroundTransparency = 1
    popupTitle.Text = "Fechar Sistema?"
    popupTitle.TextColor3 = C_TEXT
    popupTitle.TextSize = 15
    popupTitle.Font = Enum.Font.GothamBold
    popupTitle.TextXAlignment = Enum.TextXAlignment.Center
    popupTitle.ZIndex = 31

    local popupDesc = Instance.new("TextLabel", popup)
    popupDesc.Size = UDim2.new(1, -40, 0, 20)
    popupDesc.Position = UDim2.new(0, 20, 0, 50)
    popupDesc.BackgroundTransparency = 1
    popupDesc.Text = "Você precisará validar a key novamente."
    popupDesc.TextColor3 = C_MUTED
    popupDesc.TextSize = 11
    popupDesc.Font = Enum.Font.Gotham
    popupDesc.TextXAlignment = Enum.TextXAlignment.Center
    popupDesc.ZIndex = 31

    local btnSim = Instance.new("TextButton", popup)
    btnSim.Size = UDim2.new(0.5, -25, 0, 36)
    btnSim.Position = UDim2.new(0, 20, 0, 85)
    btnSim.BackgroundColor3 = C_ERR
    btnSim.TextColor3 = C_TEXT
    btnSim.Text = "Sim, fechar"
    btnSim.TextSize = 13
    btnSim.Font = Enum.Font.GothamBold
    btnSim.BorderSizePixel = 0
    btnSim.AutoButtonColor = false
    btnSim.ZIndex = 31

    local bsc = Instance.new("UICorner", btnSim)
    bsc.CornerRadius = UDim.new(0, 8)

    local btnNao = Instance.new("TextButton", popup)
    btnNao.Size = UDim2.new(0.5, -25, 0, 36)
    btnNao.Position = UDim2.new(0.5, 5, 0, 85)
    btnNao.BackgroundColor3 = C_ELEMENT
    btnNao.TextColor3 = C_TEXT
    btnNao.Text = "Cancelar"
    btnNao.TextSize = 13
    btnNao.Font = Enum.Font.GothamBold
    btnNao.BorderSizePixel = 0
    btnNao.AutoButtonColor = false
    btnNao.ZIndex = 31

    local bnc = Instance.new("UICorner", btnNao)
    bnc.CornerRadius = UDim.new(0, 8)

    -- HOVERS E ANIMAÇÕES
    tb.Focused:Connect(function() TweenService:Create(tb, TweenInfo.new(0.15), {BackgroundColor3 = C_ELEMENT_H}):Play() end)
    tb.FocusLost:Connect(function() TweenService:Create(tb, TweenInfo.new(0.15), {BackgroundColor3 = C_ELEMENT}):Play() end)

    btnFree.MouseEnter:Connect(function() TweenService:Create(btnFree, TweenInfo.new(0.1), {BackgroundColor3 = C_ELEMENT_H}):Play() end)
    btnFree.MouseLeave:Connect(function() TweenService:Create(btnFree, TweenInfo.new(0.1), {BackgroundColor3 = C_ELEMENT}):Play() end)

    btnKey.MouseEnter:Connect(function() TweenService:Create(btnKey, TweenInfo.new(0.1), {BackgroundColor3 = C_NEON_H}):Play() end)
    btnKey.MouseLeave:Connect(function() TweenService:Create(btnKey, TweenInfo.new(0.1), {BackgroundColor3 = C_NEON}):Play() end)

    btnUsar.MouseEnter:Connect(function() if btnUsar.Text == "Validar Key" then TweenService:Create(btnUsar, TweenInfo.new(0.1), {BackgroundColor3 = C_NEON_H}):Play() end end)
    btnUsar.MouseLeave:Connect(function() if btnUsar.Text == "Validar Key" then TweenService:Create(btnUsar, TweenInfo.new(0.1), {BackgroundColor3 = C_NEON}):Play() end end)

    btnObter.MouseEnter:Connect(function() 
        TweenService:Create(btnObter, TweenInfo.new(0.1), {BackgroundColor3 = C_RED_BG_H}):Play()
        TweenService:Create(bos, TweenInfo.new(0.1), {Transparency = 0.5}):Play()
    end)
    btnObter.MouseLeave:Connect(function() 
        TweenService:Create(btnObter, TweenInfo.new(0.1), {BackgroundColor3 = C_RED_BG}):Play()
        TweenService:Create(bos, TweenInfo.new(0.1), {Transparency = 0.8}):Play()
    end)

    btnVoltar.MouseEnter:Connect(function() TweenService:Create(btnVoltar, TweenInfo.new(0.1), {BackgroundColor3 = C_ELEMENT_H, TextColor3 = C_TEXT}):Play() end)
    btnVoltar.MouseLeave:Connect(function() TweenService:Create(btnVoltar, TweenInfo.new(0.1), {BackgroundColor3 = C_ELEMENT, TextColor3 = C_MUTED}):Play() end)

    btnSim.MouseEnter:Connect(function() TweenService:Create(btnSim, TweenInfo.new(0.1), {BackgroundColor3 = Color3.fromRGB(255, 85, 85)}):Play() end)
    btnSim.MouseLeave:Connect(function() TweenService:Create(btnSim, TweenInfo.new(0.1), {BackgroundColor3 = C_ERR}):Play() end)

    btnNao.MouseEnter:Connect(function() TweenService:Create(btnNao, TweenInfo.new(0.1), {BackgroundColor3 = C_ELEMENT_H}):Play() end)
    btnNao.MouseLeave:Connect(function() TweenService:Create(btnNao, TweenInfo.new(0.1), {BackgroundColor3 = C_ELEMENT}):Play() end)

    -- DRAG LÓGICA
    local drag, dragStart, startPos
    headerArea.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            drag = true
            dragStart = input.Position
            startPos = card.Position
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if drag and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            local d = input.Position - dragStart
            card.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X, startPos.Y.Scale, startPos.Y.Offset + d.Y)
        end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            drag = false
        end
    end)

    playOpenSound()

    return gui, card, screen1, screen2, tb, btnFree, btnKey,
           btnUsar, btnObter, btnVoltar, statusText, closeBtn, popup, btnSim, btnNao
end

--=====================================================================
local gui, card, screen1, screen2, tb,
      btnFree, btnKey, btnUsar, btnObter, btnVoltar,
      statusText, closeBtn, popup, btnSim, btnNao = criarUI()

--=====================================================================
-- ESTADO
--=====================================================================
local keyValue = ""
local tentativas = 0
local MAX_TENTATIVAS = 5
local KICK_MSG = "https://get-key.iphz7.workers.dev/request — pegue sua key válida"
local validando = false

--=====================================================================
-- HELPERS
--=====================================================================
local typingToken = 0

local function setStatus(txt, color)
    typingToken = typingToken + 1
    local myToken = typingToken
    statusText.Text = ""
    task.spawn(function()
        for i = 1, #txt do
            if myToken ~= typingToken then return end
            statusText.Text = txt:sub(1, i)
            task.wait(0.025)
        end
    end)
    TweenService:Create(statusText, TweenInfo.new(0.15), {TextColor3 = color}):Play()
end

local function shakeCard()
    local original = card.Position
    task.spawn(function()
        for i = 1, 4 do
            local offset = (i % 2 == 0) and 6 or -6
            card.Position = UDim2.new(original.X.Scale, original.X.Offset + offset,
                                      original.Y.Scale, original.Y.Offset)
            task.wait(0.04)
        end
        card.Position = original
    end)
end

local function trocarTela(tela)
    if tela == "key" then
        screen1.Visible = false
        screen2.Visible = true
        setStatus("digite a key", C_MUTED)
    else
        screen1.Visible = true
        screen2.Visible = false
        setStatus("escolha uma opção", C_MUTED)
    end
end

local function fadeOutEDestroy()
    local fadeOut = TweenInfo.new(0.3, Enum.EasingStyle.Quart, Enum.EasingDirection.In)
    for _, o in ipairs(gui:GetDescendants()) do
        if o:IsA("TextLabel") or o:IsA("TextButton") or o:IsA("TextBox") then
            TweenService:Create(o, fadeOut, {TextTransparency = 1}):Play()
        elseif o:IsA("Frame") then
            TweenService:Create(o, fadeOut, {BackgroundTransparency = 1}):Play()
        elseif o:IsA("ImageLabel") then
            TweenService:Create(o, fadeOut, {ImageTransparency = 1}):Play()
        elseif o:IsA("UIStroke") then
            TweenService:Create(o, fadeOut, {Transparency = 1}):Play()
        end
    end
    task.wait(0.35)
    gui:Destroy()
end

local function carregar(url)
    local ok, err = pcall(function()
        loadstring(game:HttpGet(url))()
    end)
    if not ok then
        warn("[7zK] falha ao carregar: " .. tostring(err))
    end
end

local function resetBtn()
    btnUsar.Text = "Validar Key"
    btnUsar.TextColor3 = Color3.fromRGB(10, 10, 10)
    TweenService:Create(btnUsar, TweenInfo.new(0.15), {BackgroundColor3 = C_NEON}):Play()
end

local function kickarPorTentativas()
    playError()
    pcall(function() gui:Destroy() end)
    task.wait(0.2)
    pcall(function() LocalPlayer:Kick(KICK_MSG) end)
end

local function registrarFalha(motivo)
    playError()
    shakeCard()
    tentativas = tentativas + 1
    local restantes = MAX_TENTATIVAS - tentativas

    if tentativas >= MAX_TENTATIVAS then
        setStatus("bloqueado", C_ERR)
        btnUsar.Text = "Bloqueado"
        TweenService:Create(btnUsar, TweenInfo.new(0.15), {BackgroundColor3 = C_ERR}):Play()
        btnUsar.TextColor3 = C_TEXT
        task.wait(1.5)
        kickarPorTentativas()
        return
    end

    setStatus(motivo .. " (" .. restantes .. ")", C_ERR)
    btnUsar.Text = motivo
    TweenService:Create(btnUsar, TweenInfo.new(0.15), {BackgroundColor3 = C_ERR}):Play()
    btnUsar.TextColor3 = C_TEXT
    task.wait(2)
    resetBtn()
    setStatus("digite a key", C_MUTED)
end

--=====================================================================
-- VALIDAÇÃO
--=====================================================================
local function validarKey()
    if validando then return end

    local key = (keyValue or ""):gsub("%s+", "")
    if #key == 0 then
        playError()
        shakeCard()
        setStatus("digite uma key", C_ERR)
        TweenService:Create(tb, TweenInfo.new(0.15), {BackgroundColor3 = C_ERR}):Play()
        task.wait(1.5)
        setStatus("digite a key", C_MUTED)
        TweenService:Create(tb, TweenInfo.new(0.15), {BackgroundColor3 = C_ELEMENT}):Play()
        return
    end

    local fmtOk = key:match("^7[zZ]K%-%w+%-%w+%-%w+$")
                or key:match("^7[zZ]K%-%w+%-%w+%-%w+%-%w+$")
    if not fmtOk then
        playError()
        shakeCard()
        setStatus("formato invalido", C_ERR)
        TweenService:Create(tb, TweenInfo.new(0.15), {BackgroundColor3 = C_ERR}):Play()
        task.wait(1.8)
        setStatus("digite a key", C_MUTED)
        TweenService:Create(tb, TweenInfo.new(0.15), {BackgroundColor3 = C_ELEMENT}):Play()
        return
    end

    validando = true
    playClick()
    setStatus("verificando", C_MUTED)
    btnUsar.Text = "Verificando..."
    btnUsar.TextColor3 = C_MUTED
    TweenService:Create(btnUsar, TweenInfo.new(0.15), {BackgroundColor3 = C_ELEMENT}):Play()

    local url = API_URL .. "/validate?key=" .. HttpService:UrlEncode(key)
        .. "&username=" .. HttpService:UrlEncode(LocalPlayer.Name)
        .. "&hwid="     .. HttpService:UrlEncode(getHwid())

    local ok, resp, timeout = httpGetComTimeout(url, 8)

    if timeout then
        validando = false
        btnUsar.Text = "Tempo esgotado"
        TweenService:Create(btnUsar, TweenInfo.new(0.15), {BackgroundColor3 = C_ERR}):Play()
        btnUsar.TextColor3 = C_TEXT
        task.wait(1.5)
        registrarFalha("sem resposta")
        return
    end

    if not ok or not resp then
        validando = false
        btnUsar.Text = "Sem conexão"
        TweenService:Create(btnUsar, TweenInfo.new(0.15), {BackgroundColor3 = C_ERR}):Play()
        btnUsar.TextColor3 = C_TEXT
        task.wait(1.5)
        registrarFalha("sem conexao")
        return
    end

    local ok2, data = pcall(function() return HttpService:JSONDecode(resp) end)
    if not ok2 or type(data) ~= "table" then
        validando = false
        btnUsar.Text = "Erro"
        TweenService:Create(btnUsar, TweenInfo.new(0.15), {BackgroundColor3 = C_ERR}):Play()
        btnUsar.TextColor3 = C_TEXT
        task.wait(1.5)
        registrarFalha("resposta invalida")
        return
    end

    local aprovado = data.valid == true or data.ok == true

    if aprovado then
        validando = false
        playSuccess()
        setStatus("autenticado", C_NEON)
        btnUsar.Text = "Autenticado"
        TweenService:Create(btnUsar, TweenInfo.new(0.15), {BackgroundColor3 = C_NEON}):Play()
        btnUsar.TextColor3 = Color3.fromRGB(10, 10, 10)
        task.wait(0.8)
        fadeOutEDestroy()
        task.wait(0.1)
        carregar(URL_PREMIUM)
    else
        validando = false
        local motivos = {
            no_key                  = "sem key",
            no_user                 = "sem username",
            not_found               = "key nao existe",
            expired                 = "key expirada",
            banned                  = "key banida",
            blocked                 = "key bloqueada",
            frozen                  = "sistema congelado",
            account_limit_reached   = "ja usada em outra conta",
        }
        local msg = motivos[data.reason] or "invalida"
        registrarFalha(msg)
    end
end

--=====================================================================
-- EVENTOS
--=====================================================================
tb:GetPropertyChangedSignal("Text"):Connect(function()
    keyValue = tb.Text
end)

btnFree.MouseButton1Click:Connect(function()
    playClick()
    setStatus("entrando free", C_MUTED)
    fadeOutEDestroy()
    task.wait(0.1)
    carregar(URL_FREE)
end)

btnKey.MouseButton1Click:Connect(function()
    playClick()
    trocarTela("key")
end)

btnVoltar.MouseButton1Click:Connect(function()
    playClick()
    trocarTela("choice")
end)

btnUsar.MouseButton1Click:Connect(function()
    if btnUsar.Text ~= "Validar Key" then return end
    validarKey()
end)

tb.FocusLost:Connect(function(enterPressed)
    if enterPressed and btnUsar.Text == "Validar Key" then
        validarKey()
    end
end)

btnObter.MouseButton1Click:Connect(function()
    playClick()
    local url = API_URL .. "/request"
    if type(openbrowser) == "function" then
        local ok = pcall(openbrowser, url)
        if ok then
            setStatus("abrindo...", C_MUTED)
            task.wait(1.5)
            setStatus("digite a key", C_MUTED)
            return
        end
    end
    if setclipboard then
        pcall(setclipboard, url)
        setStatus("link copiado", C_NEON)
        task.wait(1.6)
        setStatus("digite a key", C_MUTED)
    end
end)

-- EVENTO DO BOTÃO X (ABRE O POPUP)
closeBtn.MouseButton1Click:Connect(function()
    playClick()
    popup.Visible = true
end)

-- AÇÃO DO SIM (FECHA)
btnSim.MouseButton1Click:Connect(function()
    playClick()
    popup.Visible = false
    fadeOutEDestroy()
end)

-- AÇÃO DO NÃO (CANCELA)
btnNao.MouseButton1Click:Connect(function()
    playClick()
    popup.Visible = false
end)