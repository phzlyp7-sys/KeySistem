-- language: Luau, target: Roblox MOBILE
-- Key Sistem 7zK — v3.0 (cyber glass + efeitos)

local API_URL     = "https://obtendo-key.7zk.workers.dev"
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
local RunService       = game:GetService("RunService")
local LocalPlayer      = Players.LocalPlayer

--=====================================================================
-- PALETA
--=====================================================================
local C_BG         = Color3.fromRGB(5, 8, 7)
local C_CARD       = Color3.fromRGB(10, 21, 16)
local C_INPUT      = Color3.fromRGB(8, 16, 12)
local C_STROKE     = Color3.fromRGB(20, 59, 40)
local C_STROKE_HI  = Color3.fromRGB(31, 92, 61)
local C_TEXT       = Color3.fromRGB(216, 245, 230)
local C_MUTED      = Color3.fromRGB(107, 140, 122)
local C_DIM        = Color3.fromRGB(61, 90, 74)
local C_NEON       = Color3.fromRGB(0, 255, 136)
local C_NEON_H     = Color3.fromRGB(74, 255, 170)
local C_ERR        = Color3.fromRGB(255, 77, 94)
local C_WARN       = Color3.fromRGB(255, 176, 32)
local C_GHOST      = Color3.fromRGB(18, 42, 29)
local C_GHOST_H    = Color3.fromRGB(24, 52, 36)

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

local function playOpenSound()
    playSound("open", 0.32, 1.20)
    task.delay(0.06, function()
        lastSoundAt = 0
        playSound("open", 0.20, 0.85)
    end)
end

local function playClick() playSound("click", 0.40, 1.25) end

local function playSuccess()
    playSound("success", 0.55, 1.45)
    task.delay(0.10, function()
        lastSoundAt = 0
        playSound("success", 0.40, 1.80)
    end)
end

local function playError() playSound("error", 0.50, 0.65) end

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
local function criarCard()
    local pg = LocalPlayer:WaitForChild("PlayerGui")

    local gui = Instance.new("ScreenGui")
    gui.Name = "7zK_KeySystem"
    gui.ResetOnSpawn = false
    gui.DisplayOrder = 9999
    gui.IgnoreGuiInset = true
    gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    gui.Parent = pg

    -- GRADE DE FUNDO
    local grid = Instance.new("Frame", gui)
    grid.Size = UDim2.new(1, 0, 1, 0)
    grid.BackgroundTransparency = 1
    grid.ZIndex = 0

    for i = 0, 24 do
        local line = Instance.new("Frame", grid)
        line.Size = UDim2.new(1, 0, 0, 1)
        line.Position = UDim2.new(0, 0, 0, i * 40)
        line.BackgroundColor3 = C_NEON
        line.BackgroundTransparency = 0.94
        line.BorderSizePixel = 0
    end
    for i = 0, 30 do
        local line = Instance.new("Frame", grid)
        line.Size = UDim2.new(0, 1, 1, 0)
        line.Position = UDim2.new(0, i * 40, 0, 0)
        line.BackgroundColor3 = C_NEON
        line.BackgroundTransparency = 0.94
        line.BorderSizePixel = 0
    end

    -- PARTÍCULAS DE FUNDO
    local particleLayer = Instance.new("Frame", gui)
    particleLayer.Size = UDim2.new(1, 0, 1, 0)
    particleLayer.BackgroundTransparency = 1
    particleLayer.ZIndex = 1

    for i = 1, 14 do
        local p = Instance.new("Frame", particleLayer)
        local sz = math.random(3, 6)
        p.Size = UDim2.new(0, sz, 0, sz)
        p.Position = UDim2.new(math.random(), 0, math.random(), 0)
        p.BackgroundColor3 = C_NEON
        p.BackgroundTransparency = math.random(70, 88) / 100
        p.BorderSizePixel = 0

        local pc = Instance.new("UICorner", p)
        pc.CornerRadius = UDim.new(1, 0)

        task.spawn(function()
            while p.Parent do
                local dur = math.random(250, 500) / 100
                local targetY = math.random(-10, 100) / 100
                TweenService:Create(p, TweenInfo.new(dur, Enum.EasingStyle.Sine), {
                    Position = UDim2.new(p.Position.X.Scale, 0, targetY, 0)
                }):Play()
                task.wait(dur)
            end
        end)
    end

    -- CARD
    local card = Instance.new("Frame")
    card.Size = UDim2.new(0, 340, 0, 340)
    card.Position = UDim2.new(0.5, -170, 0.5, -170)
    card.BackgroundColor3 = C_CARD
    card.BackgroundTransparency = 0.05
    card.BorderSizePixel = 0
    card.ZIndex = 10
    card.Parent = gui

    local cc = Instance.new("UICorner", card)
    cc.CornerRadius = UDim.new(0, 16)

    local cs = Instance.new("UIStroke", card)
    cs.Color = C_STROKE_HI
    cs.Thickness = 1
    cs.Transparency = 0.15

    local shadow = Instance.new("ImageLabel", card)
    shadow.AnchorPoint = Vector2.new(0.5, 0.5)
    shadow.Position = UDim2.new(0.5, 0, 0.5, 14)
    shadow.Size = UDim2.new(1, 32, 1, 32)
    shadow.BackgroundTransparency = 1
    shadow.Image = "rbxassetid://5028857084"
    shadow.ImageColor3 = Color3.fromRGB(0, 0, 0)
    shadow.ImageTransparency = 0.55
    shadow.ScaleType = Enum.ScaleType.Slice
    shadow.SliceCenter = Rect.new(24, 24, 276, 276)
    shadow.ZIndex = -1

    -- LINHA NEON DO TOPO
    local topGlow = Instance.new("Frame", card)
    topGlow.Size = UDim2.new(1, -32, 0, 2)
    topGlow.Position = UDim2.new(0, 16, 0, 0)
    topGlow.BackgroundColor3 = C_NEON
    topGlow.BorderSizePixel = 0
    topGlow.ZIndex = 12

    local tgc = Instance.new("UICorner", topGlow)
    tgc.CornerRadius = UDim.new(1, 0)

    -- glow pulsante no topo
    task.spawn(function()
        while card.Parent do
            TweenService:Create(topGlow, TweenInfo.new(1.8, Enum.EasingStyle.Sine), {
                BackgroundTransparency = 0.5
            }):Play()
            task.wait(1.8)
            if not card.Parent then break end
            TweenService:Create(topGlow, TweenInfo.new(1.8, Enum.EasingStyle.Sine), {
                BackgroundTransparency = 0
            }):Play()
            task.wait(1.8)
        end
    end)

    local glowBg = Instance.new("ImageLabel", card)
    glowBg.Size = UDim2.new(1, 32, 0, 24)
    glowBg.Position = UDim2.new(0, -16, 0, -12)
    glowBg.BackgroundTransparency = 1
    glowBg.Image = "rbxassetid://5028857084"
    glowBg.ImageColor3 = C_NEON
    glowBg.ImageTransparency = 0.55
    glowBg.ScaleType = Enum.ScaleType.Slice
    glowBg.SliceCenter = Rect.new(24, 24, 276, 276)
    glowBg.ZIndex = 11

    -- SCAN LINE
    local scan = Instance.new("Frame", card)
    scan.Size = UDim2.new(1, -20, 0, 2)
    scan.Position = UDim2.new(0, 10, 0, -10)
    scan.BackgroundColor3 = C_NEON
    scan.BackgroundTransparency = 0.7
    scan.BorderSizePixel = 0
    scan.ZIndex = 14

    local scanGrad = Instance.new("UIGradient", scan)
    scanGrad.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 1),
        NumberSequenceKeypoint.new(0.5, 0.4),
        NumberSequenceKeypoint.new(1, 1),
    })

    task.spawn(function()
        while card.Parent do
            scan.Position = UDim2.new(0, 10, 0, -10)
            TweenService:Create(scan, TweenInfo.new(4, Enum.EasingStyle.Linear), {
                Position = UDim2.new(0, 10, 1, 10)
            }):Play()
            task.wait(4)
        end
    end)

    -- CANTOS HUD
    local function hudCorner(pos, size)
        local c = Instance.new("Frame", card)
        c.Size = size
        c.Position = pos
        c.BackgroundColor3 = C_NEON
        c.BorderSizePixel = 0
        c.ZIndex = 13
    end

    hudCorner(UDim2.new(0, 10, 0, 10), UDim2.new(0, 14, 0, 2))
    hudCorner(UDim2.new(0, 10, 0, 10), UDim2.new(0, 2, 0, 14))
    hudCorner(UDim2.new(1, -24, 0, 10), UDim2.new(0, 14, 0, 2))
    hudCorner(UDim2.new(1, -12, 0, 10), UDim2.new(0, 2, 0, 14))
    hudCorner(UDim2.new(0, 10, 1, -12), UDim2.new(0, 14, 0, 2))
    hudCorner(UDim2.new(0, 10, 1, -24), UDim2.new(0, 2, 0, 14))
    hudCorner(UDim2.new(1, -24, 1, -12), UDim2.new(0, 14, 0, 2))
    hudCorner(UDim2.new(1, -12, 1, -24), UDim2.new(0, 2, 0, 14))

    -- HEADER
    local logo = Instance.new("Frame", card)
    logo.Size = UDim2.new(0, 70, 0, 22)
    logo.Position = UDim2.new(0, 26, 0, 24)
    logo.BackgroundTransparency = 1
    logo.ZIndex = 12

    local lbl7 = Instance.new("TextLabel", logo)
    lbl7.Size = UDim2.new(0, 14, 1, 0)
    lbl7.Position = UDim2.new(0, 0, 0, 0)
    lbl7.BackgroundTransparency = 1
    lbl7.Text = "7"
    lbl7.TextColor3 = C_TEXT
    lbl7.TextSize = 19
    lbl7.Font = Enum.Font.GothamBold
    lbl7.TextXAlignment = Enum.TextXAlignment.Left
    lbl7.ZIndex = 12

    local lblz = Instance.new("TextLabel", logo)
    lblz.Size = UDim2.new(0, 14, 1, 0)
    lblz.Position = UDim2.new(0, 12, 0, 0)
    lblz.BackgroundTransparency = 1
    lblz.Text = "z"
    lblz.TextColor3 = C_NEON
    lblz.TextSize = 19
    lblz.Font = Enum.Font.GothamBold
    lblz.TextXAlignment = Enum.TextXAlignment.Left
    lblz.ZIndex = 12

    local lblzStroke = Instance.new("UIStroke", lblz)
    lblzStroke.Color = C_NEON
    lblzStroke.Thickness = 1
    lblzStroke.Transparency = 0.4

    local lblK = Instance.new("TextLabel", logo)
    lblK.Size = UDim2.new(0, 22, 1, 0)
    lblK.Position = UDim2.new(0, 24, 0, 0)
    lblK.BackgroundTransparency = 1
    lblK.Text = "K"
    lblK.TextColor3 = C_TEXT
    lblK.TextSize = 19
    lblK.Font = Enum.Font.GothamBold
    lblK.TextXAlignment = Enum.TextXAlignment.Left
    lblK.ZIndex = 12

    local tag = Instance.new("TextLabel", card)
    tag.Size = UDim2.new(0, 100, 0, 22)
    tag.Position = UDim2.new(0, 96, 0, 24)
    tag.BackgroundTransparency = 1
    tag.Text = "A C E S S O"
    tag.TextColor3 = C_MUTED
    tag.TextSize = 9
    tag.Font = Enum.Font.Code
    tag.TextXAlignment = Enum.TextXAlignment.Left
    tag.ZIndex = 12

    local dot = Instance.new("Frame", card)
    dot.Size = UDim2.new(0, 7, 0, 7)
    dot.Position = UDim2.new(1, -30, 0, 35)
    dot.AnchorPoint = Vector2.new(1, 0.5)
    dot.BackgroundColor3 = C_DIM
    dot.BorderSizePixel = 0
    dot.ZIndex = 12

    local dotc = Instance.new("UICorner", dot)
    dotc.CornerRadius = UDim.new(1, 0)

    local dotShadow = Instance.new("UIStroke", dot)
    dotShadow.Color = C_DIM
    dotShadow.Thickness = 4
    dotShadow.Transparency = 0.85

    local statusText = Instance.new("TextLabel", card)
    statusText.Size = UDim2.new(0, 140, 0, 22)
    statusText.Position = UDim2.new(1, -44, 0, 24)
    statusText.AnchorPoint = Vector2.new(1, 0)
    statusText.BackgroundTransparency = 1
    statusText.Text = "escolha"
    statusText.TextColor3 = C_MUTED
    statusText.TextSize = 10
    statusText.Font = Enum.Font.Code
    statusText.TextXAlignment = Enum.TextXAlignment.Right
    statusText.ZIndex = 12

    local divider = Instance.new("Frame", card)
    divider.Size = UDim2.new(1, -48, 0, 1)
    divider.Position = UDim2.new(0, 24, 0, 60)
    divider.BackgroundColor3 = C_STROKE
    divider.BorderSizePixel = 0
    divider.ZIndex = 12

    -- TELA 1
    local screen1 = Instance.new("Frame", card)
    screen1.Size = UDim2.new(1, 0, 0, 270)
    screen1.Position = UDim2.new(0, 0, 0, 62)
    screen1.BackgroundTransparency = 1
    screen1.ZIndex = 11

    local subtitle = Instance.new("TextLabel", screen1)
    subtitle.Size = UDim2.new(1, -48, 0, 42)
    subtitle.Position = UDim2.new(0, 24, 0, 10)
    subtitle.BackgroundTransparency = 1
    subtitle.Text = "Escolha como entrar\nGrátis com funções básicas ou com key pra tudo"
    subtitle.TextColor3 = C_MUTED
    subtitle.TextSize = 11
    subtitle.Font = Enum.Font.Code
    subtitle.TextXAlignment = Enum.TextXAlignment.Left
    subtitle.TextYAlignment = Enum.TextYAlignment.Top
    subtitle.TextWrapped = true
    subtitle.ZIndex = 12

    local btnFree = Instance.new("TextButton", screen1)
    btnFree.Size = UDim2.new(1, -48, 0, 52)
    btnFree.Position = UDim2.new(0, 24, 0, 64)
    btnFree.BackgroundColor3 = C_GHOST
    btnFree.TextColor3 = C_TEXT
    btnFree.Text = "Versão sem Key"
    btnFree.TextSize = 14
    btnFree.Font = Enum.Font.GothamBold
    btnFree.BorderSizePixel = 0
    btnFree.AutoButtonColor = false
    btnFree.ZIndex = 12

    local bfc = Instance.new("UICorner", btnFree)
    bfc.CornerRadius = UDim.new(0, 10)

    local bfs = Instance.new("UIStroke", btnFree)
    bfs.Color = C_STROKE_HI
    bfs.Thickness = 1

    local btnKey = Instance.new("TextButton", screen1)
    btnKey.Size = UDim2.new(1, -48, 0, 52)
    btnKey.Position = UDim2.new(0, 24, 0, 134)
    btnKey.BackgroundColor3 = C_NEON
    btnKey.TextColor3 = Color3.fromRGB(5, 8, 7)
    btnKey.Text = "Tenho uma Key"
    btnKey.TextSize = 14
    btnKey.Font = Enum.Font.GothamBold
    btnKey.BorderSizePixel = 0
    btnKey.AutoButtonColor = false
    btnKey.ZIndex = 12

    local bkc = Instance.new("UICorner", btnKey)
    bkc.CornerRadius = UDim.new(0, 10)

    task.spawn(function()
        while btnKey.Parent do
            TweenService:Create(btnKey, TweenInfo.new(1.4, Enum.EasingStyle.Sine), {
                BackgroundColor3 = C_NEON_H
            }):Play()
            task.wait(1.4)
            if not btnKey.Parent then break end
            TweenService:Create(btnKey, TweenInfo.new(1.4, Enum.EasingStyle.Sine), {
                BackgroundColor3 = C_NEON
            }):Play()
            task.wait(1.4)
        end
    end)

    local keySub = Instance.new("TextLabel", screen1)
    keySub.Size = UDim2.new(1, -48, 0, 12)
    keySub.Position = UDim2.new(0, 24, 0, 192)
    keySub.BackgroundTransparency = 1
    keySub.Text = "Keys são grátis no nosso site"
    keySub.TextColor3 = C_ERR
    keySub.TextSize = 10
    keySub.Font = Enum.Font.GothamBold
    keySub.TextXAlignment = Enum.TextXAlignment.Center
    keySub.ZIndex = 12

    -- TELA 2
    local screen2 = Instance.new("Frame", card)
    screen2.Size = UDim2.new(1, 0, 0, 270)
    screen2.Position = UDim2.new(0, 0, 0, 62)
    screen2.BackgroundTransparency = 1
    screen2.Visible = false
    screen2.ZIndex = 11

    local labelKey = Instance.new("TextLabel", screen2)
    labelKey.Size = UDim2.new(1, -48, 0, 14)
    labelKey.Position = UDim2.new(0, 24, 0, 10)
    labelKey.BackgroundTransparency = 1
    labelKey.Text = "K E Y   D E   A C E S S O"
    labelKey.TextColor3 = C_MUTED
    labelKey.TextSize = 9
    labelKey.Font = Enum.Font.Code
    labelKey.TextXAlignment = Enum.TextXAlignment.Left
    labelKey.ZIndex = 12

    local tb = Instance.new("TextBox", screen2)
    tb.Size = UDim2.new(1, -48, 0, 44)
    tb.Position = UDim2.new(0, 24, 0, 28)
    tb.BackgroundColor3 = C_INPUT
    tb.TextColor3 = C_TEXT
    tb.PlaceholderText = "cole sua key aqui"
    tb.PlaceholderColor3 = C_DIM
    tb.Text = ""
    tb.TextSize = 13
    tb.Font = Enum.Font.Code
    tb.BorderSizePixel = 0
    tb.ClearTextOnFocus = false
    tb.TextXAlignment = Enum.TextXAlignment.Left
    tb.ZIndex = 12

    local tbc = Instance.new("UICorner", tb)
    tbc.CornerRadius = UDim.new(0, 10)

    local tbp = Instance.new("UIPadding", tb)
    tbp.PaddingLeft = UDim.new(0, 14)
    tbp.PaddingRight = UDim.new(0, 14)

    local tbs = Instance.new("UIStroke", tb)
    tbs.Color = C_STROKE
    tbs.Thickness = 1

    local btnUsar = Instance.new("TextButton", screen2)
    btnUsar.Size = UDim2.new(1, -48, 0, 46)
    btnUsar.Position = UDim2.new(0, 24, 0, 86)
    btnUsar.BackgroundColor3 = C_NEON
    btnUsar.TextColor3 = Color3.fromRGB(5, 8, 7)
    btnUsar.Text = "Usar Key"
    btnUsar.TextSize = 14
    btnUsar.Font = Enum.Font.GothamBold
    btnUsar.BorderSizePixel = 0
    btnUsar.AutoButtonColor = false
    btnUsar.ZIndex = 12

    local bvc = Instance.new("UICorner", btnUsar)
    bvc.CornerRadius = UDim.new(0, 10)

    local btnObter = Instance.new("TextButton", screen2)
    btnObter.Size = UDim2.new(1, -48, 0, 38)
    btnObter.Position = UDim2.new(0, 24, 0, 146)
    btnObter.BackgroundColor3 = C_GHOST
    btnObter.TextColor3 = C_TEXT
    btnObter.Text = "Obter Key"
    btnObter.TextSize = 13
    btnObter.Font = Enum.Font.Gotham
    btnObter.BorderSizePixel = 0
    btnObter.AutoButtonColor = false
    btnObter.ZIndex = 12

    local boc = Instance.new("UICorner", btnObter)
    boc.CornerRadius = UDim.new(0, 10)

    local bos = Instance.new("UIStroke", btnObter)
    bos.Color = C_STROKE_HI
    bos.Thickness = 1

    local btnVoltar = Instance.new("TextButton", screen2)
    btnVoltar.Size = UDim2.new(1, -48, 0, 30)
    btnVoltar.Position = UDim2.new(0, 24, 0, 192)
    btnVoltar.BackgroundColor3 = C_GHOST
    btnVoltar.TextColor3 = C_MUTED
    btnVoltar.Text = "Voltar"
    btnVoltar.TextSize = 11
    btnVoltar.Font = Enum.Font.Gotham
    btnVoltar.BorderSizePixel = 0
    btnVoltar.AutoButtonColor = false
    btnVoltar.ZIndex = 12

    local bvc2 = Instance.new("UICorner", btnVoltar)
    bvc2.CornerRadius = UDim.new(0, 10)

    local bvs2 = Instance.new("UIStroke", btnVoltar)
    bvs2.Color = C_STROKE_HI
    bvs2.Thickness = 1

    -- DRAG
    local drag, dragStart, startPos
    local headerArea = Instance.new("TextButton", card)
    headerArea.Size = UDim2.new(1, 0, 0, 60)
    headerArea.Position = UDim2.new(0, 0, 0, 0)
    headerArea.BackgroundTransparency = 1
    headerArea.Text = ""
    headerArea.ZIndex = 20
    headerArea.Parent = card

    headerArea.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then
            drag = true
            dragStart = input.Position
            startPos = card.Position
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if drag and (input.UserInputType == Enum.UserInputType.MouseMovement
        or input.UserInputType == Enum.UserInputType.Touch) then
            local d = input.Position - dragStart
            card.Position = UDim2.new(
                startPos.X.Scale, startPos.X.Offset + d.X,
                startPos.Y.Scale, startPos.Y.Offset + d.Y
            )
        end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then
            drag = false
        end
    end)

    tb.Focused:Connect(function()
        TweenService:Create(tbs, TweenInfo.new(0.15), {Color = C_NEON}):Play()
    end)
    tb.FocusLost:Connect(function()
        TweenService:Create(tbs, TweenInfo.new(0.15), {Color = C_STROKE}):Play()
    end)

    btnFree.MouseEnter:Connect(function()
        TweenService:Create(btnFree, TweenInfo.new(0.12), {BackgroundColor3 = C_GHOST_H}):Play()
        TweenService:Create(bfs, TweenInfo.new(0.12), {Color = C_NEON}):Play()
    end)
    btnFree.MouseLeave:Connect(function()
        TweenService:Create(btnFree, TweenInfo.new(0.12), {BackgroundColor3 = C_GHOST}):Play()
        TweenService:Create(bfs, TweenInfo.new(0.12), {Color = C_STROKE_HI}):Play()
    end)

    btnKey.MouseEnter:Connect(function()
        TweenService:Create(bkc, TweenInfo.new(0.12), {CornerRadius = UDim.new(0, 12)}):Play()
    end)
    btnKey.MouseLeave:Connect(function()
        TweenService:Create(bkc, TweenInfo.new(0.12), {CornerRadius = UDim.new(0, 10)}):Play()
    end)

    btnUsar.MouseEnter:Connect(function()
        if btnUsar.Text == "Usar Key" then
            TweenService:Create(btnUsar, TweenInfo.new(0.12), {BackgroundColor3 = C_NEON_H}):Play()
        end
    end)
    btnUsar.MouseLeave:Connect(function()
        if btnUsar.Text == "Usar Key" then
            TweenService:Create(btnUsar, TweenInfo.new(0.12), {BackgroundColor3 = C_NEON}):Play()
        end
    end)

    btnObter.MouseEnter:Connect(function()
        TweenService:Create(btnObter, TweenInfo.new(0.12), {BackgroundColor3 = C_GHOST_H}):Play()
        TweenService:Create(bos, TweenInfo.new(0.12), {Color = C_NEON}):Play()
    end)
    btnObter.MouseLeave:Connect(function()
        TweenService:Create(btnObter, TweenInfo.new(0.12), {BackgroundColor3 = C_GHOST}):Play()
        TweenService:Create(bos, TweenInfo.new(0.12), {Color = C_STROKE_HI}):Play()
    end)

    -- FADE IN
    local fadeAll = { card, divider, screen1, subtitle, btnFree, btnKey, keySub }
    for _, o in ipairs(fadeAll) do
        if o:IsA("TextLabel") or o:IsA("TextButton") or o:IsA("TextBox") then
            o.TextTransparency = 1
        end
        if o:IsA("Frame") or o:IsA("TextButton") or o:IsA("TextBox") then
            o.BackgroundTransparency = 1
        end
    end
    tbs.Transparency = 1
    cs.Transparency = 1
    bos.Transparency = 1
    bfs.Transparency = 1
    bvs2.Transparency = 1
    dot.BackgroundTransparency = 1
    dotShadow.Transparency = 1

    local info = TweenInfo.new(0.35, Enum.EasingStyle.Quart, Enum.EasingDirection.Out)
    for _, o in ipairs(fadeAll) do
        if o:IsA("TextLabel") or o:IsA("TextButton") or o:IsA("TextBox") then
            TweenService:Create(o, info, {TextTransparency = 0}):Play()
        end
    end
    TweenService:Create(card, info, {BackgroundTransparency = 0.05}):Play()
    TweenService:Create(btnFree, info, {BackgroundTransparency = 0}):Play()
    TweenService:Create(btnKey, info, {BackgroundTransparency = 0}):Play()
    TweenService:Create(divider, info, {BackgroundTransparency = 0}):Play()
    TweenService:Create(cs, info, {Transparency = 0.15}):Play()
    TweenService:Create(bfs, info, {Transparency = 0}):Play()
    TweenService:Create(dot, info, {BackgroundTransparency = 0}):Play()
    TweenService:Create(dotShadow, info, {Transparency = 0.85}):Play()

    playOpenSound()

    return gui, card, screen1, screen2, tb, tbs, btnFree, btnKey,
           btnUsar, btnObter, btnVoltar, bos, dot, dotShadow, statusText
end

--=====================================================================
local gui, card, screen1, screen2, tb, tbs,
      btnFree, btnKey, btnUsar, btnObter, btnVoltar,
      bos, dot, dotShadow, statusText = criarCard()

--=====================================================================
-- ESTADO
--=====================================================================
local keyValue = ""
local tentativas = 0
local MAX_TENTATIVAS = 5
local KICK_MSG = "https://obtendo-key.7zk.workers.dev/request — pegue sua key válida"
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

local function setDot(color)
    dot.BackgroundColor3 = color
    dotShadow.Color = color
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
        setDot(C_MUTED)
    else
        screen1.Visible = true
        screen2.Visible = false
        setStatus("escolha", C_MUTED)
        setDot(C_DIM)
    end
end

local function fadeOutEDestroy()
    local fadeOut = TweenInfo.new(0.35, Enum.EasingStyle.Quart, Enum.EasingDirection.In)
    for _, o in ipairs(card:GetDescendants()) do
        if o:IsA("TextLabel") or o:IsA("TextButton") or o:IsA("TextBox") then
            TweenService:Create(o, fadeOut, {TextTransparency = 1}):Play()
        elseif o:IsA("Frame") then
            TweenService:Create(o, fadeOut, {BackgroundTransparency = 1}):Play()
        elseif o:IsA("UIStroke") then
            TweenService:Create(o, fadeOut, {Transparency = 1}):Play()
        end
    end
    task.wait(0.4)
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
    btnUsar.Text = "Usar Key"
    btnUsar.TextColor3 = Color3.fromRGB(5, 8, 7)
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
        setDot(C_ERR)
        btnUsar.Text = "Bloqueado"
        TweenService:Create(btnUsar, TweenInfo.new(0.15), {BackgroundColor3 = C_ERR}):Play()
        btnUsar.TextColor3 = Color3.fromRGB(255, 255, 255)
        task.wait(1.5)
        kickarPorTentativas()
        return
    end

    setStatus(motivo .. " (" .. restantes .. ")", C_ERR)
    setDot(C_ERR)
    btnUsar.Text = motivo
    TweenService:Create(btnUsar, TweenInfo.new(0.15), {BackgroundColor3 = C_ERR}):Play()
    btnUsar.TextColor3 = Color3.fromRGB(255, 255, 255)
    task.wait(2)
    resetBtn()
    setStatus("digite a key", C_MUTED)
    setDot(C_MUTED)
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
        setDot(C_ERR)
        TweenService:Create(tbs, TweenInfo.new(0.15), {Color = C_ERR}):Play()
        task.wait(1.5)
        setStatus("digite a key", C_MUTED)
        setDot(C_MUTED)
        TweenService:Create(tbs, TweenInfo.new(0.15), {Color = C_STROKE}):Play()
        return
    end

    local fmtOk = key:match("^7[zZ]K%-%w+%-%w+%-%w+$")
                or key:match("^7[zZ]K%-%w+%-%w+%-%w+%-%w+$")
    if not fmtOk then
        playError()
        shakeCard()
        setStatus("formato invalido", C_ERR)
        setDot(C_ERR)
        TweenService:Create(tbs, TweenInfo.new(0.15), {Color = C_ERR}):Play()
        task.wait(1.8)
        setStatus("digite a key", C_MUTED)
        setDot(C_MUTED)
        TweenService:Create(tbs, TweenInfo.new(0.15), {Color = C_STROKE}):Play()
        return
    end

    validando = true
    playClick()
    setStatus("verificando", C_WARN)
    setDot(C_WARN)
    btnUsar.Text = "Verificando..."
    btnUsar.TextColor3 = C_MUTED
    TweenService:Create(btnUsar, TweenInfo.new(0.15), {BackgroundColor3 = C_GHOST}):Play()

    local url = API_URL .. "/validate?key=" .. HttpService:UrlEncode(key)
        .. "&username=" .. HttpService:UrlEncode(LocalPlayer.Name)
        .. "&hwid="     .. HttpService:UrlEncode(getHwid())

    local ok, resp, timeout = httpGetComTimeout(url, 8)

    if timeout then
        validando = false
        btnUsar.Text = "Tempo esgotado"
        TweenService:Create(btnUsar, TweenInfo.new(0.15), {BackgroundColor3 = C_ERR}):Play()
        btnUsar.TextColor3 = Color3.fromRGB(255, 255, 255)
        task.wait(1.5)
        registrarFalha("sem resposta")
        return
    end

    if not ok or not resp then
        validando = false
        btnUsar.Text = "Sem conexão"
        TweenService:Create(btnUsar, TweenInfo.new(0.15), {BackgroundColor3 = C_ERR}):Play()
        btnUsar.TextColor3 = Color3.fromRGB(255, 255, 255)
        task.wait(1.5)
        registrarFalha("sem conexao")
        return
    end

    local ok2, data = pcall(function() return HttpService:JSONDecode(resp) end)
    if not ok2 or type(data) ~= "table" then
        validando = false
        btnUsar.Text = "Erro"
        TweenService:Create(btnUsar, TweenInfo.new(0.15), {BackgroundColor3 = C_ERR}):Play()
        btnUsar.TextColor3 = Color3.fromRGB(255, 255, 255)
        task.wait(1.5)
        registrarFalha("resposta invalida")
        return
    end

    local aprovado = data.valid == true or data.ok == true

    if aprovado then
        validando = false
        playSuccess()
        setStatus("autenticado", C_NEON)
        setDot(C_NEON)
        btnUsar.Text = "Autenticado"
        TweenService:Create(btnUsar, TweenInfo.new(0.15), {BackgroundColor3 = C_NEON}):Play()
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
    setStatus("entrando free", C_WARN)
    setDot(C_WARN)
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
    if btnUsar.Text ~= "Usar Key" then return end
    validarKey()
end)

tb.FocusLost:Connect(function(enterPressed)
    if enterPressed and btnUsar.Text == "Usar Key" then
        validarKey()
    end
end)

btnObter.MouseButton1Click:Connect(function()
    playClick()
    local url = API_URL .. "/request"
    if type(openbrowser) == "function" then
        local ok = pcall(openbrowser, url)
        if ok then
            setStatus("abrindo...", C_WARN)
            setDot(C_WARN)
            task.wait(1.5)
            setStatus("digite a key", C_MUTED)
            setDot(C_MUTED)
            return
        end
    end
    if setclipboard then
        pcall(setclipboard, url)
        setStatus("link copiado", C_NEON)
        setDot(C_NEON)
        TweenService:Create(bos, TweenInfo.new(0.15), {Color = C_NEON}):Play()
        task.wait(1.6)
        setStatus("digite a key", C_MUTED)
        setDot(C_MUTED)
        TweenService:Create(bos, TweenInfo.new(0.15), {Color = C_STROKE_HI}):Play()
    end
end)
