--═══════════════════════════════════════════════════════════════
--  ✦  HUB — v33  •  Elegance Edition
--     Avatar do player + botão dark face
--═══════════════════════════════════════════════════════════════
print("[Hub] v33 iniciando...")

local Players            = game:GetService("Players")
local TweenService       = game:GetService("TweenService")
local UserInputService   = game:GetService("UserInputService")
local ContentProvider    = game:GetService("ContentProvider")
local HttpService        = game:GetService("HttpService")
local LocalPlayer        = Players.LocalPlayer

--═══════════════════ CONFIG PADRÃO ═══════════════════
local DEFAULT_CONFIG = {
    Title    = "Hub",
    Subtitle = "Elegance Edition",

    KEY      = "",
    KEY_HINT = "Digite sua key",

    -- ═══ FOTO DO BOTÃO ═══
    -- "" = botão preto padrão
    -- "rbxassetid://6703968311," = dark face
    BUTTON_IMAGE = "rbxassetid://6703968311,",

    -- ═══ FOTO DO AVATAR (hero) ═══
    -- "" = avatar do próprio jogador (Roblox)
    -- "rbxassetid://123" = foto customizada
    AVATAR_IMAGE = "",

    AccentName = "Blue",
    AccentRGB  = nil,

    Font = "Gotham",

    Animations   = true,
    DragEnabled  = true,

    WindowWidth  = 680,
    WindowHeight = 570,
}

--═══════════════════ CONFIG EXTERNO ═══════════════════
local CONFIG_FILE = "HubConfig.json"

local function loadExternalConfig()
    local cfg = {}
    if getgenv and getgenv().HubConfig and type(getgenv().HubConfig) == "table" then
        for k, v in pairs(getgenv().HubConfig) do cfg[k] = v end
    end
    if readfile and isfile then
        local ok, exists = pcall(isfile, CONFIG_FILE)
        if ok and exists then
            local ok2, content = pcall(readfile, CONFIG_FILE)
            if ok2 and content and content ~= "" then
                local ok3, parsed = pcall(function() return HttpService:JSONDecode(content) end)
                if ok3 and parsed and type(parsed) == "table" then
                    for k, v in pairs(parsed) do cfg[k] = v end
                end
            end
        end
    end
    return cfg
end

local function saveExternalConfig(cfg)
    if getgenv then getgenv().HubConfig = cfg end
    if writefile then
        local ok, json = pcall(function() return HttpService:JSONEncode(cfg) end)
        if ok and json then pcall(writefile, CONFIG_FILE, json) end
    end
end

local CONFIG = {}
local function buildConfig()
    local external = loadExternalConfig()
    for k, v in pairs(DEFAULT_CONFIG) do CONFIG[k] = v end
    for k, v in pairs(external) do CONFIG[k] = v end
end
buildConfig()

--═══════════════════ PALETA ═══════════════════
local T = {
    Bg         = Color3.fromRGB(10, 13, 22),
    Bg2        = Color3.fromRGB(16, 21, 36),
    Panel      = Color3.fromRGB(22, 28, 46),
    PanelDark  = Color3.fromRGB(16, 21, 34),
    Light      = Color3.fromRGB(31, 39, 60),
    Hover      = Color3.fromRGB(46, 58, 86),

    Accent     = Color3.fromRGB(120, 175, 255),
    Accent2    = Color3.fromRGB(160, 120, 250),
    AccentD    = Color3.fromRGB(70, 120, 210),
    Glow       = Color3.fromRGB(200, 220, 255),

    Text       = Color3.fromRGB(232, 238, 252),
    TextBright = Color3.fromRGB(255, 255, 255),
    Dim        = Color3.fromRGB(148, 164, 194),
    Soft       = Color3.fromRGB(92, 108, 138),

    Stroke     = Color3.fromRGB(40, 51, 76),
    StrokeL    = Color3.fromRGB(58, 72, 102),

    Danger     = Color3.fromRGB(255, 105, 128),
    Success    = Color3.fromRGB(105, 225, 165),

    ButtonBg     = Color3.fromRGB(15, 15, 18),
    ButtonStroke = Color3.fromRGB(60, 60, 70),
}

local DEFAULT_ACCENT = Color3.fromRGB(120, 175, 255)

--═══════════════════ 20 CORES ═══════════════════
local Presets = {
    {Name="Blue",    Color=Color3.fromRGB(120, 175, 255)},
    {Name="Sky",     Color=Color3.fromRGB(120, 200, 255)},
    {Name="Cyan",    Color=Color3.fromRGB(95, 215, 235)},
    {Name="Mint",    Color=Color3.fromRGB(125, 235, 205)},
    {Name="Green",   Color=Color3.fromRGB(110, 230, 150)},
    {Name="Lime",    Color=Color3.fromRGB(180, 235, 100)},
    {Name="Yellow",  Color=Color3.fromRGB(250, 225, 100)},
    {Name="Gold",    Color=Color3.fromRGB(255, 210, 115)},
    {Name="Orange",  Color=Color3.fromRGB(255, 170, 105)},
    {Name="Coral",   Color=Color3.fromRGB(255, 145, 130)},
    {Name="Red",     Color=Color3.fromRGB(255, 115, 135)},
    {Name="Rose",    Color=Color3.fromRGB(255, 160, 180)},
    {Name="Pink",    Color=Color3.fromRGB(255, 145, 195)},
    {Name="Magenta", Color=Color3.fromRGB(235, 120, 220)},
    {Name="Purple",  Color=Color3.fromRGB(175, 140, 255)},
    {Name="Violet",  Color=Color3.fromRGB(150, 115, 235)},
    {Name="Lavender",Color=Color3.fromRGB(210, 185, 255)},
    {Name="White",   Color=Color3.fromRGB(232, 238, 250)},
    {Name="Silver",  Color=Color3.fromRGB(195, 200, 215)},
    {Name="Dark",    Color=Color3.fromRGB(85, 95, 115)},
}

local function findPreset(name)
    for _, p in ipairs(Presets) do
        if p.Name == name then return p.Color end
    end
    return nil
end

--═══════════════════ FONTES ═══════════════════
local function safeFont(n, f)
    local ok, v = pcall(function() return Enum.Font[n] end)
    return (ok and v) or f or Enum.Font.Gotham
end
local Fonts = {
    {Name="Gotham",      Main=safeFont("Gotham"),      Bold=safeFont("GothamBold"),      Medium=safeFont("GothamMedium")},
    {Name="Source Sans", Main=safeFont("SourceSans"),  Bold=safeFont("SourceSansBold"),  Medium=safeFont("SourceSansSemibold")},
    {Name="Roboto",      Main=safeFont("Roboto"),      Bold=safeFont("Roboto"),          Medium=safeFont("Roboto")},
    {Name="Nunito",      Main=safeFont("Nunito"),      Bold=safeFont("Nunito"),          Medium=safeFont("Nunito")},
    {Name="Ubuntu",      Main=safeFont("Ubuntu"),      Bold=safeFont("Ubuntu"),          Medium=safeFont("Ubuntu")},
    {Name="Garamond",    Main=safeFont("Garamond"),    Bold=safeFont("Garamond"),        Medium=safeFont("Garamond")},
}

--═══════════════════ ESTADO ═══════════════════
local State = {
    Font = CONFIG.Font or "Gotham",
    UIOpen = false, SettingsOpen = false,
    Animations = CONFIG.Animations ~= false,
    DragEnabled = CONFIG.DragEnabled ~= false,
    Destroyed = false,
    CustomAccent = nil,
}
local REQUIRES_KEY = CONFIG.KEY ~= nil and CONFIG.KEY ~= ""
local KeyUnlocked = not REQUIRES_KEY

--═══════════════════ HELPERS ═══════════════════
local function new(c, p)
    local o = Instance.new(c)
    for k, v in pairs(p or {}) do o[k] = v end
    return o
end
local function corner(p, r)
    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0, r or 8)
    c.Parent = p
    return c
end
local function stroke(p, c, t)
    local s = Instance.new("UIStroke")
    s.Color = c or T.Stroke; s.Thickness = t or 1
    s.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    s.Parent = p
    return s
end
local function gradient(p, c1, c2, rot)
    local g = Instance.new("UIGradient")
    g.Color = ColorSequence.new(c1, c2); g.Rotation = rot or 90
    g.Parent = p
    return g
end
local function tw(o, p, t, style)
    return TweenService:Create(o, TweenInfo.new(t or 0.3, style or Enum.EasingStyle.Quart), p)
end

local function getFont(w)
    for _, f in ipairs(Fonts) do
        if f.Name == State.Font then
            if w == "Bold" then return f.Bold end
            if w == "Medium" then return f.Medium end
            return f.Main
        end
    end
    return safeFont("Gotham")
end

local FontReg = {}
local function addFont(obj, weight)
    FontReg[#FontReg + 1] = {obj = obj, weight = weight or "Main"}
    obj.Font = getFont(weight)
end
local function mkLabel(parent, weight, props)
    local p = props or {}
    if p.BackgroundTransparency == nil then p.BackgroundTransparency = 1 end
    local l = new("TextLabel", p); l.Parent = parent
    addFont(l, weight); return l
end
local function mkButton(parent, weight, props)
    local p = props or {}; p.AutoButtonColor = false
    local b = new("TextButton", p); b.Parent = parent
    addFont(b, weight); return b
end
local function applyFontAll()
    for i = 1, #FontReg do
        local e = FontReg[i]
        if e.obj and e.obj.Parent then
            pcall(function() e.obj.Font = getFont(e.weight) end)
        end
    end
end

--═══════════════════ ACCENT SYSTEM ═══════════════════
local AccentReg = {}
local toggleTracks = {}

local function regAccent(obj, prop, kind)
    if not obj or not prop then return end
    AccentReg[#AccentReg + 1] = {obj = obj, prop = prop, kind = kind or "accent"}
end
local function regAccentGradient(g, c1kind, c2kind)
    if not g then return end
    AccentReg[#AccentReg + 1] = {
        obj = g, prop = "Color",
        kind = "gradient", c1kind = c1kind or "accent", c2kind = c2kind or "accent2",
    }
end

local function getKindColor(kind)
    if kind == "accent"  then return T.Accent  end
    if kind == "accentD" then return T.AccentD end
    if kind == "accent2" then return T.Accent2 end
    if kind == "glow"    then return T.Glow    end
    return T.Accent
end

local function applyAccent()
    for i = 1, #AccentReg do
        local e = AccentReg[i]
        if e.obj then
            pcall(function()
                if e.kind == "gradient" then
                    local c1 = getKindColor(e.c1kind)
                    local c2 = getKindColor(e.c2kind)
                    e.obj.Color = ColorSequence.new(c1, c2)
                else
                    e.obj[e.prop] = getKindColor(e.kind)
                end
            end)
        end
    end
    for _, t in ipairs(toggleTracks) do
        if t.track and t.track.Parent then
            pcall(function()
                t.track.BackgroundColor3 = t.getState() and T.Accent or T.StrokeL
            end)
        end
    end
end

local function setAccentColor(c, save)
    T.Accent  = c
    T.AccentD = c:Lerp(Color3.new(0,0,0), 0.35)
    T.Accent2 = c:Lerp(Color3.fromRGB(160,120,250), 0.5)
    T.Glow    = c:Lerp(Color3.new(1,1,1), 0.45)
    applyAccent()

    if save then
        State.CustomAccent = c
        CONFIG.AccentRGB = {R = math.floor(c.R * 255), G = math.floor(c.G * 255), B = math.floor(c.B * 255)}
        saveExternalConfig(CONFIG)
    end
end

local function formatAge(d)
    if not d or d <= 0 then return "Novo" end
    if d < 30 then return d .. " dias" end
    local m = math.floor(d / 30)
    if m < 12 then return m .. (m == 1 and " mês" or " meses") end
    local y = math.floor(d / 365)
    local rm = math.floor((d - y * 365) / 30)
    if rm <= 0 then return y .. (y == 1 and " ano" or " anos") end
    return y .. (y == 1 and " ano" or " anos") .. " e " ..
           rm .. (rm == 1 and " mês" or " meses")
end

local function normalizeImage(url)
    if not url or url == "" then return "" end
    if url:match("^%d+$") then return "rbxassetid://" .. url end
    return url
end

--═══════════════════ API GLOBAL ═══════════════════
local HubAPI = {}

function HubAPI:SetColor(nameOrRGB)
    local color = nil
    if type(nameOrRGB) == "string" then
        color = findPreset(nameOrRGB)
    elseif type(nameOrRGB) == "table" and nameOrRGB.R and nameOrRGB.G and nameOrRGB.B then
        color = Color3.fromRGB(nameOrRGB.R, nameOrRGB.G, nameOrRGB.B)
    elseif typeof(nameOrRGB) == "Color3" then
        color = nameOrRGB
    end
    if color then setAccentColor(color, true); return true end
    return false
end

function HubAPI:SetFont(name)
    for _, f in ipairs(Fonts) do
        if f.Name == name then
            State.Font = name
            CONFIG.Font = name
            applyFontAll()
            saveExternalConfig(CONFIG)
            return true
        end
    end
    return false
end

function HubAPI:SetButtonImage(url)
    CONFIG.BUTTON_IMAGE = url
    saveExternalConfig(CONFIG)
    return true
end

function HubAPI:SetAvatar(url)
    CONFIG.AVATAR_IMAGE = url
    saveExternalConfig(CONFIG)
    if avatar then
        avatar.Image = normalizeImage(url)
    end
    return true
end

function HubAPI:GetConfig() return CONFIG end

function HubAPI:ReloadConfig()
    buildConfig()
    State.Font = CONFIG.Font or "Gotham"
    State.Animations = CONFIG.Animations ~= false
    State.DragEnabled = CONFIG.DragEnabled ~= false
    applyFontAll()
    local color = nil
    if CONFIG.AccentRGB then
        color = Color3.fromRGB(CONFIG.AccentRGB.R, CONFIG.AccentRGB.G, CONFIG.AccentRGB.B)
    elseif CONFIG.AccentName then
        color = findPreset(CONFIG.AccentName)
    end
    if color then setAccentColor(color, false) end
end

if getgenv then
    getgenv().Hub = HubAPI
    if not getgenv().HubConfig then getgenv().HubConfig = CONFIG end
end

--═══════════════════ CLEANUP ═══════════════════
for _, n in ipairs({"HubV15","HubV16","HubV17","HubV18","HubV19","HubV20","HubV21","HubV22","HubV23","HubV24","HubV25","HubV26","HubV27","HubV28","HubV29","HubV30","HubV31","HubV32","HubV33"}) do
    pcall(function()
        local cg = game:GetService("CoreGui"):FindFirstChild(n)
        if cg then cg:Destroy() end
        local pg = LocalPlayer:FindFirstChild("PlayerGui")
        if pg then local x = pg:FindFirstChild(n); if x then x:Destroy() end end
    end)
end

--═══════════════════ GUI ═══════════════════
local Gui = new("ScreenGui", {
    Name = "HubV33", ResetOnSpawn = false,
    ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
    IgnoreGuiInset = true,
})
local ok = pcall(function() Gui.Parent = game:GetService("CoreGui") end)
if not ok or not Gui.Parent then
    pcall(function() Gui.Parent = LocalPlayer:WaitForChild("PlayerGui") end)
end
print("[Hub] 1. GUI OK")

--═══════════════════ BOTÃO FLUTUANTE (dark face) ═══════════════════
local BUTTON_SIZE = 60
local ToggleBtn = new("TextButton", {
    Size = UDim2.fromOffset(BUTTON_SIZE, BUTTON_SIZE),
    Position = UDim2.fromOffset(24, 90),
    BackgroundColor3 = T.ButtonBg,
    Text = "",
    AutoButtonColor = false,
    ClipsDescendants = true,
    ZIndex = 100,
    Parent = Gui,
})
corner(ToggleBtn, BUTTON_SIZE / 2)

local btnStroke = stroke(ToggleBtn, T.ButtonStroke, 1.5)

local buttonText = new("TextLabel", {
    Size = UDim2.fromScale(1, 1),
    Text = "✦",
    TextSize = 28,
    TextColor3 = T.TextBright,
    BackgroundTransparency = 1,
    ZIndex = 102,
    Parent = ToggleBtn,
})
addFont(buttonText, "Bold")

local buttonImage = nil
local buttonImageUrl = normalizeImage(CONFIG.BUTTON_IMAGE)

if buttonImageUrl ~= "" then
    buttonImage = new("ImageLabel", {
        Size = UDim2.fromScale(1, 1),
        BackgroundTransparency = 1,
        Image = buttonImageUrl,
        ScaleType = Enum.ScaleType.Crop,
        ZIndex = 101,
        Parent = ToggleBtn,
    })
    corner(buttonImage, BUTTON_SIZE / 2)

    buttonImage.ImageFailed = function()
        if buttonImage then buttonImage:Destroy() buttonImage = nil end
        buttonText.Visible = true
    end

    task.spawn(function()
        local success = false
        pcall(function()
            ContentProvider:PreloadAsync({buttonImage}, function(_, status)
                if status == Enum.AssetFetchStatus.Success then success = true end
            end)
        end)
        if success then
            buttonText.Visible = false
            print("[Hub] ✓ foto do botão carregada (dark face)")
        else
            print("[Hub] ⚠ foto do botão falhou, usando ✦")
            if buttonImage then buttonImage:Destroy() buttonImage = nil end
            buttonText.Visible = true
        end
    end)
end

if not buttonImage then
    task.spawn(function()
        while ToggleBtn.Parent do
            tw(btnStroke, {Color = T.Glow, Thickness = 2.5}, 1.6):Play()
            task.wait(1.6)
            if not ToggleBtn.Parent then break end
            tw(btnStroke, {Color = T.ButtonStroke, Thickness = 1.5}, 1.6):Play()
            task.wait(1.6)
        end
    end)
end

ToggleBtn.MouseEnter:Connect(function()
    if not State.Destroyed then
        tw(ToggleBtn, {Size = UDim2.fromOffset(BUTTON_SIZE + 6, BUTTON_SIZE + 6)}, 0.2):Play()
    end
end)
ToggleBtn.MouseLeave:Connect(function()
    if not State.Destroyed then
        tw(ToggleBtn, {Size = UDim2.fromOffset(BUTTON_SIZE, BUTTON_SIZE)}, 0.2):Play()
    end
end)
print("[Hub] 2. Botão OK")

--═══════════════════ KEY SCREEN ═══════════════════
local KeyScreen, KeyCard, KeyBox, KeyStatus, KeyButton
if REQUIRES_KEY then
    KeyScreen = new("Frame", {
        Size = UDim2.fromScale(1, 1),
        BackgroundColor3 = Color3.new(0, 0, 0),
        BackgroundTransparency = 0.55,
        Visible = false, ZIndex = 500, Parent = Gui,
    })
    KeyCard = new("Frame", {
        Size = UDim2.fromOffset(380, 270),
        Position = UDim2.new(0.5, 0, 0.5, 0),
        AnchorPoint = Vector2.new(0.5, 0.5),
        BackgroundColor3 = T.Bg, BorderSizePixel = 0,
        ZIndex = 501, Parent = KeyScreen,
    })
    corner(KeyCard, 20)
    stroke(KeyCard, T.StrokeL, 1)
    gradient(KeyCard, T.Bg, T.Bg2, 145)

    local kHeader = new("Frame", {
        Size = UDim2.new(1, 0, 0, 56),
        BackgroundColor3 = T.Panel, BorderSizePixel = 0,
        ZIndex = 502, Parent = KeyCard,
    })
    corner(kHeader, 20)
    new("Frame", {
        Size = UDim2.new(1, 0, 0, 22),
        Position = UDim2.new(0, 0, 1, -22),
        BackgroundColor3 = T.Panel, BorderSizePixel = 0,
        ZIndex = 502, Parent = kHeader,
    })
    local kLine = new("Frame", {
        Size = UDim2.new(1, 0, 0, 2), BackgroundColor3 = T.Accent,
        BorderSizePixel = 0, ZIndex = 503, Parent = kHeader,
    })
    local kLineGrad = gradient(kLine, T.Accent, T.Accent2, 0)
    regAccentGradient(kLineGrad, "accent", "accent2")
    mkLabel(kHeader, "Bold", {
        Size = UDim2.new(1, -60, 0, 24), Position = UDim2.fromOffset(22, 16),
        Text = "🔒  Autenticação", TextSize = 16, TextColor3 = T.Text,
        TextXAlignment = Enum.TextXAlignment.Left, ZIndex = 503,
    })
    mkLabel(KeyCard, "Medium", {
        Size = UDim2.new(1, -40, 0, 18), Position = UDim2.fromOffset(20, 78),
        Text = "Digite a key para continuar.", TextSize = 12, TextColor3 = T.Dim,
        TextXAlignment = Enum.TextXAlignment.Left, ZIndex = 503,
    })
    KeyBox = new("TextBox", {
        Size = UDim2.new(1, -40, 0, 42), Position = UDim2.fromOffset(20, 108),
        BackgroundColor3 = T.Panel, BorderSizePixel = 0,
        Text = "", PlaceholderText = CONFIG.KEY_HINT,
        PlaceholderColor3 = T.Soft, TextSize = 13, TextColor3 = T.Text,
        ClearTextOnFocus = false,
        TextXAlignment = Enum.TextXAlignment.Left,
        ZIndex = 503, Parent = KeyCard,
    })
    corner(KeyBox, 10)
    addFont(KeyBox, "Medium")
    new("UIPadding", {PaddingLeft = UDim.new(0, 12), Parent = KeyBox})
    stroke(KeyBox, T.StrokeL, 1)
    KeyStatus = mkLabel(KeyCard, "Medium", {
        Size = UDim2.new(1, -40, 0, 16), Position = UDim2.fromOffset(20, 156),
        Text = "", TextSize = 11, TextColor3 = T.Danger,
        TextXAlignment = Enum.TextXAlignment.Left, ZIndex = 503,
    })
    KeyButton = mkButton(KeyCard, "Bold", {
        Size = UDim2.new(1, -40, 0, 44), Position = UDim2.fromOffset(20, 184),
        BackgroundColor3 = T.Accent, Text = "Entrar",
        TextSize = 14, TextColor3 = Color3.new(1, 1, 1), ZIndex = 503,
    })
    corner(KeyButton, 10)
    local keyBtnGrad = gradient(KeyButton, T.Accent, T.Accent2, 135)
    regAccentGradient(keyBtnGrad, "accent", "accent2")
end

--═══════════════════ ROOT ═══════════════════
local Root = new("Frame", {
    Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1,
    Visible = false, ZIndex = 1, Parent = Gui,
})

--═══════════════════ JANELA ═══════════════════
local W = CONFIG.WindowWidth or 680
local H = CONFIG.WindowHeight or 570

local Main = new("Frame", {
    Size = UDim2.fromOffset(W, H),
    Position = UDim2.new(0.5, 0, 0.5, 0),
    AnchorPoint = Vector2.new(0.5, 0.5),
    BackgroundColor3 = T.Bg, BorderSizePixel = 0,
    ClipsDescendants = true, Active = true, ZIndex = 2, Parent = Root,
})
corner(Main, 22)
gradient(Main, T.Bg, T.Bg2, 145)
stroke(Main, T.StrokeL, 1.5)

-- HEADER
local Header = new("Frame", {
    Size = UDim2.new(1, 0, 0, 70),
    BackgroundColor3 = T.Panel, BorderSizePixel = 0,
    ZIndex = 3, Parent = Main,
})
corner(Header, 22)
new("Frame", {
    Size = UDim2.new(1, 0, 0, 30), Position = UDim2.new(0, 0, 1, -30),
    BackgroundColor3 = T.Panel, BorderSizePixel = 0, ZIndex = 3, Parent = Header,
})
local topLine = new("Frame", {
    Size = UDim2.new(1, 0, 0, 2.5), BackgroundColor3 = T.Accent,
    BorderSizePixel = 0, ZIndex = 4, Parent = Header,
})
local topLineGrad = gradient(topLine, T.Accent, T.Accent2, 0)
regAccentGradient(topLineGrad, "accent", "accent2")

local iconBox = new("Frame", {
    Size = UDim2.fromOffset(40, 40), Position = UDim2.fromOffset(20, 15),
    BackgroundColor3 = T.Accent, BorderSizePixel = 0, ZIndex = 4, Parent = Header,
})
corner(iconBox, 20)
local iconGrad = gradient(iconBox, T.Accent, T.Accent2, 135)
regAccentGradient(iconGrad, "accent", "accent2")
local iconStroke = stroke(iconBox, T.Glow, 1.5)
regAccent(iconStroke, "Color", "glow")
mkLabel(iconBox, "Bold", {
    Size = UDim2.fromScale(1, 1), Text = "✦", TextSize = 22,
    TextColor3 = Color3.new(1, 1, 1), ZIndex = 5,
})

mkLabel(Header, "Bold", {
    Size = UDim2.new(0, 340, 0, 24), Position = UDim2.fromOffset(72, 14),
    Text = CONFIG.Title, TextSize = 20, TextColor3 = T.Text,
    TextXAlignment = Enum.TextXAlignment.Left, ZIndex = 4,
})
mkLabel(Header, "Medium", {
    Size = UDim2.new(0, 340, 0, 16), Position = UDim2.fromOffset(72, 38),
    Text = CONFIG.Subtitle, TextSize = 12, TextColor3 = T.Dim,
    TextXAlignment = Enum.TextXAlignment.Left, ZIndex = 4,
})

local function headerBtn(txt, xOff, danger)
    local b = mkButton(Header, "Bold", {
        Size = UDim2.fromOffset(38, 38),
        Position = UDim2.new(1, xOff, 0.5, -19),
        BackgroundColor3 = T.Light, Text = txt, TextSize = 17,
        TextColor3 = T.Text, ZIndex = 4,
    })
    corner(b, 12)
    b.MouseEnter:Connect(function() tw(b, {BackgroundColor3 = danger and T.Danger or T.Accent}, 0.2):Play() end)
    b.MouseLeave:Connect(function() tw(b, {BackgroundColor3 = T.Light}, 0.2):Play() end)
    return b
end
local CloseBtn = headerBtn("✕", -52, true)
local GearBtn  = headerBtn("⚙", -98, false)

-- SIDEBAR
local SB_W = 170
local Sidebar = new("Frame", {
    Size = UDim2.new(0, SB_W, 1, -90),
    Position = UDim2.fromOffset(16, 80),
    BackgroundColor3 = T.Panel, BorderSizePixel = 0,
    ZIndex = 3, Parent = Main,
})
corner(Sidebar, 16)
stroke(Sidebar, T.StrokeL, 1)
mkLabel(Sidebar, "Bold", {
    Size = UDim2.new(1, -24, 0, 14), Position = UDim2.fromOffset(16, 14),
    Text = "NAVEGAÇÃO", TextSize = 9, TextColor3 = T.Soft,
    TextXAlignment = Enum.TextXAlignment.Left, ZIndex = 4,
})

local TabList = new("Frame", {
    Size = UDim2.new(1, -16, 1, -48),
    Position = UDim2.fromOffset(8, 36),
    BackgroundTransparency = 1, ZIndex = 4, Parent = Sidebar,
})
new("UIListLayout", {
    Padding = UDim.new(0, 6),
    SortOrder = Enum.SortOrder.LayoutOrder, Parent = TabList,
})

local ContentArea = new("Frame", {
    Size = UDim2.new(1, -SB_W - 48, 1, -90),
    Position = UDim2.fromOffset(SB_W + 32, 80),
    BackgroundTransparency = 1, ZIndex = 3, Parent = Main,
})

local tabBtn = mkButton(TabList, "Medium", {
    Size = UDim2.new(1, 0, 0, 44),
    BackgroundColor3 = T.Light, Text = "",
    LayoutOrder = 1, ZIndex = 4,
})
corner(tabBtn, 12)
local tabBar = new("Frame", {
    Size = UDim2.new(0, 3, 0, 24),
    Position = UDim2.new(0, 0, 0.5, -12),
    BackgroundColor3 = T.Accent, BorderSizePixel = 0,
    ZIndex = 5, Parent = tabBtn,
})
corner(tabBar, 2)
regAccent(tabBar, "BackgroundColor3", "accent")
local tabIcon = mkLabel(tabBtn, "Bold", {
    Size = UDim2.fromOffset(26, 44), Position = UDim2.fromOffset(16, 0),
    Text = "◈", TextSize = 17, TextColor3 = T.Accent, ZIndex = 5,
})
regAccent(tabIcon, "TextColor3", "accent")
mkLabel(tabBtn, "Medium", {
    Size = UDim2.new(1, -50, 1, 0), Position = UDim2.fromOffset(46, 0),
    Text = "Principal", TextSize = 14, TextColor3 = T.TextBright,
    TextXAlignment = Enum.TextXAlignment.Left, ZIndex = 5,
})

local Page = new("ScrollingFrame", {
    Size = UDim2.fromScale(1, 1),
    BackgroundTransparency = 1, BorderSizePixel = 0,
    ScrollBarThickness = 4, ScrollBarImageColor3 = T.StrokeL,
    AutomaticCanvasSize = Enum.AutomaticSize.Y,
    CanvasSize = UDim2.new(0, 0, 0, 0),
    ZIndex = 4, Parent = ContentArea,
})
new("UIListLayout", {Padding = UDim.new(0, 14), SortOrder = Enum.SortOrder.LayoutOrder, Parent = Page})
new("UIPadding", {PaddingRight = UDim.new(0, 8), Parent = Page})

--═══════════════════ HERO CARD (com avatar!) ═══════════════════
local hero = new("Frame", {
    Size = UDim2.new(1, 0, 0, 210),
    BackgroundColor3 = T.Panel, BorderSizePixel = 0,
    ClipsDescendants = true, LayoutOrder = 1, ZIndex = 4, Parent = Page,
})
corner(hero, 20)
gradient(hero, T.Panel, T.Light, 145)
stroke(hero, T.StrokeL, 1)

local blob1 = new("Frame", {
    Size = UDim2.fromOffset(240, 240), Position = UDim2.new(1, -100, 0, -100),
    BackgroundColor3 = T.Accent, BackgroundTransparency = 0.93,
    BorderSizePixel = 0, ZIndex = 4, Parent = hero,
})
corner(blob1, 120)
regAccent(blob1, "BackgroundColor3", "accent")

local heroLine = new("Frame", {
    Size = UDim2.new(0, 100, 0, 3), Position = UDim2.new(0.5, -50, 0, 0),
    BackgroundColor3 = T.Accent, BorderSizePixel = 0,
    ZIndex = 5, Parent = hero,
})
corner(heroLine, 2)
regAccent(heroLine, "BackgroundColor3", "accent")

--═══════════════════ AVATAR DO PLAYER ═══════════════════
local AW, AH = 126, 160
local AX, AY = 24, 24

local avatarGlow = new("Frame", {
    Size = UDim2.fromOffset(AW + 14, AH + 14),
    Position = UDim2.fromOffset(AX - 7, AY - 7),
    BackgroundColor3 = T.Accent, BackgroundTransparency = 0.82,
    BorderSizePixel = 0, ZIndex = 4, Parent = hero,
})
corner(avatarGlow, 20)
regAccent(avatarGlow, "BackgroundColor3", "accent")

task.spawn(function()
    while avatarGlow.Parent do
        tw(avatarGlow, {BackgroundTransparency = 0.92}, 2):Play()
        task.wait(2)
        if not avatarGlow.Parent then break end
        tw(avatarGlow, {BackgroundTransparency = 0.78}, 2):Play()
        task.wait(2)
    end
end)

local avatarFrame = new("Frame", {
    Size = UDim2.fromOffset(AW, AH),
    Position = UDim2.fromOffset(AX, AY),
    BackgroundColor3 = T.PanelDark, BorderSizePixel = 0,
    ClipsDescendants = true, ZIndex = 5, Parent = hero,
})
corner(avatarFrame, 18)
local avStroke = stroke(avatarFrame, T.Accent, 2)
regAccent(avStroke, "Color", "accent")

local avatarFallback = mkLabel(avatarFrame, "Bold", {
    Size = UDim2.fromScale(1, 1),
    Text = string.upper(string.sub(LocalPlayer.Name, 1, 1)),
    TextSize = 62, TextColor3 = T.Text, ZIndex = 6,
})

local avatar = new("ImageLabel", {
    Size = UDim2.fromScale(1, 1),
    BackgroundTransparency = 1, Image = "",
    ScaleType = Enum.ScaleType.Fit,
    ImageTransparency = 1, ZIndex = 7, Parent = avatarFrame,
})
corner(avatar, 18)

local function loadAvatar()
    -- Usa CONFIG.AVATAR_IMAGE se preenchido, senão usa o avatar do Roblox
    local finalURL = normalizeImage(CONFIG.AVATAR_IMAGE)
    if finalURL == "" then
        finalURL = "rbxthumb://type=Avatar&id=" .. LocalPlayer.UserId .. "&w=420&h=420"
        print("[Hub] 👤 carregando avatar do Roblox")
    else
        print("[Hub] 📷 usando AVATAR_IMAGE customizada:", finalURL)
    end
    avatar.Image = finalURL
    task.spawn(function()
        local loaded = false
        pcall(function()
            ContentProvider:PreloadAsync({avatar}, function(_, status)
                if status == Enum.AssetFetchStatus.Success then loaded = true end
            end)
        end)
        if loaded then
            avatarFallback.Visible = false
            tw(avatar, {ImageTransparency = 0}, 0.4):Play()
            print("[Hub] ✓ avatar carregado")
        else
            -- Fallback: headshot do Roblox
            avatar.Image = "rbxthumb://type=AvatarHeadShot&id=" ..
                           LocalPlayer.UserId .. "&w=420&h=420"
            local ok2 = false
            pcall(function()
                ContentProvider:PreloadAsync({avatar}, function(_, status)
                    if status == Enum.AssetFetchStatus.Success then ok2 = true end
                end)
            end)
            if ok2 then
                avatarFallback.Visible = false
                tw(avatar, {ImageTransparency = 0}, 0.4):Play()
                print("[Hub] ✓ headshot carregado")
            else
                avatar.Visible = false
                avatarFallback.Visible = true
                print("[Hub] ⚠ avatar falhou — usando inicial do nome")
            end
        end
    end)
end
loadAvatar()

-- INFO (ao lado do avatar)
local IX = AX + AW + 26
local displayName = LocalPlayer.DisplayName
if not displayName or displayName == "" then displayName = LocalPlayer.Name end

mkLabel(hero, "Bold", {
    Size = UDim2.new(1, -IX - 200, 0, 34),
    Position = UDim2.fromOffset(IX, 34),
    Text = displayName, TextSize = 24, TextColor3 = T.TextBright,
    TextXAlignment = Enum.TextXAlignment.Left,
    TextTruncate = Enum.TextTruncate.AtEnd, ZIndex = 6,
})
mkLabel(hero, "Medium", {
    Size = UDim2.new(1, -IX - 200, 0, 18),
    Position = UDim2.fromOffset(IX, 72),
    Text = "@" .. LocalPlayer.Name,
    TextSize = 14, TextColor3 = T.Dim,
    TextXAlignment = Enum.TextXAlignment.Left,
    TextTruncate = Enum.TextTruncate.AtEnd, ZIndex = 6,
})

local onlineDot = new("Frame", {
    Size = UDim2.fromOffset(11, 11),
    Position = UDim2.fromOffset(IX, 104),
    BackgroundColor3 = T.Success, BorderSizePixel = 0,
    ZIndex = 6, Parent = hero,
})
corner(onlineDot, 6)
task.spawn(function()
    while onlineDot.Parent do
        tw(onlineDot, {BackgroundTransparency = 0.55}, 1.2):Play()
        task.wait(1.2)
        if not onlineDot.Parent then break end
        tw(onlineDot, {BackgroundTransparency = 0}, 1.2):Play()
        task.wait(1.2)
    end
end)

local age = 0
pcall(function() age = LocalPlayer.AccountAge end)
mkLabel(hero, "Medium", {
    Size = UDim2.new(1, -IX - 200, 0, 18),
    Position = UDim2.fromOffset(IX + 20, 101),
    Text = "Online  •  📅 " .. formatAge(age),
    TextSize = 13, TextColor3 = T.Dim,
    TextXAlignment = Enum.TextXAlignment.Left, ZIndex = 6,
})

-- TIME BOX
local TW, TH = 165, 100
local timeBox = new("Frame", {
    Size = UDim2.fromOffset(TW, TH),
    Position = UDim2.new(1, -TW - 22, 0, 24),
    BackgroundColor3 = T.Bg, BackgroundTransparency = 0.25,
    BorderSizePixel = 0, ZIndex = 5, Parent = hero,
})
corner(timeBox, 16)
stroke(timeBox, T.StrokeL, 1)
local timeGlow = new("Frame", {
    Size = UDim2.new(1, 0, 0, 36),
    BackgroundColor3 = T.Accent, BackgroundTransparency = 0.92,
    BorderSizePixel = 0, ZIndex = 5, Parent = timeBox,
})
corner(timeGlow, 16)
regAccent(timeGlow, "BackgroundColor3", "accent")

mkLabel(timeBox, "Bold", {
    Size = UDim2.new(1, 0, 0, 14), Position = UDim2.fromOffset(0, 14),
    Text = "🕐  BRASÍLIA", TextSize = 10, TextColor3 = T.Dim, ZIndex = 6,
})
local timeLbl = mkLabel(timeBox, "Bold", {
    Size = UDim2.new(1, 0, 0, 38), Position = UDim2.fromOffset(0, 30),
    Text = "--:--:--", TextSize = 30, TextColor3 = T.Accent, ZIndex = 6,
})
regAccent(timeLbl, "TextColor3", "accent")
local dateLbl = mkLabel(timeBox, "Medium", {
    Size = UDim2.new(1, 0, 0, 16), Position = UDim2.fromOffset(0, 70),
    Text = "--/--/----", TextSize = 12, TextColor3 = T.Dim, ZIndex = 6,
})
task.spawn(function()
    while timeLbl.Parent do
        local t = os.date("!*t", os.time() - (3 * 3600))
        timeLbl.Text = string.format("%02d:%02d:%02d", t.hour, t.min, t.sec)
        dateLbl.Text = string.format("%02d/%02d/%04d", t.day, t.month, t.year)
        task.wait(1)
    end
end)

local idBox = new("Frame", {
    Size = UDim2.fromOffset(160, 26),
    Position = UDim2.new(1, -182, 1, -34),
    BackgroundColor3 = T.Bg, BackgroundTransparency = 0.4,
    BorderSizePixel = 0, ZIndex = 5, Parent = hero,
})
corner(idBox, 7)
stroke(idBox, T.Stroke, 1)
mkLabel(idBox, "Medium", {
    Size = UDim2.new(1, -12, 1, 0), Position = UDim2.fromOffset(6, 0),
    Text = "ID: " .. LocalPlayer.UserId,
    TextSize = 10, TextColor3 = T.Soft,
    TextXAlignment = Enum.TextXAlignment.Center,
    TextTruncate = Enum.TextTruncate.AtEnd, ZIndex = 6,
})
print("[Hub] 3. Hero OK")

--═══════════════════ INFO ROW ═══════════════════
local infoRow = new("Frame", {
    Size = UDim2.new(1, 0, 0, 105),
    BackgroundTransparency = 1, LayoutOrder = 2, ZIndex = 4, Parent = Page,
})
new("UIListLayout", {
    FillDirection = Enum.FillDirection.Horizontal,
    Padding = UDim.new(0, 10),
    SortOrder = Enum.SortOrder.LayoutOrder, Parent = infoRow,
})

local function infoCard(order, icon, label, initial)
    local c = new("Frame", {
        Size = UDim2.new(1/2, -5, 1, 0),
        BackgroundColor3 = T.Panel, BorderSizePixel = 0,
        ClipsDescendants = true, LayoutOrder = order, ZIndex = 4, Parent = infoRow,
    })
    corner(c, 16)
    gradient(c, T.Panel, T.Light, 135)
    stroke(c, T.StrokeL, 1)
    local b = new("Frame", {
        Size = UDim2.fromOffset(44, 44), Position = UDim2.fromOffset(16, 14),
        BackgroundColor3 = T.Accent, BackgroundTransparency = 0.76,
        BorderSizePixel = 0, ZIndex = 5, Parent = c,
    })
    corner(b, 13)
    local bs = stroke(b, T.Accent, 1)
    regAccent(b, "BackgroundColor3", "accent")
    regAccent(bs, "Color", "accent")
    mkLabel(b, "Bold", {Size = UDim2.fromScale(1, 1), Text = icon, TextSize = 20, ZIndex = 6})
    mkLabel(c, "Bold", {
        Size = UDim2.new(1, -28, 0, 14), Position = UDim2.fromOffset(16, 66),
        Text = label:upper(), TextSize = 9, TextColor3 = T.Soft,
        TextXAlignment = Enum.TextXAlignment.Left, ZIndex = 5,
    })
    return mkLabel(c, "Bold", {
        Size = UDim2.new(1, -28, 0, 22), Position = UDim2.fromOffset(16, 82),
        Text = initial or "—", TextSize = 15, TextColor3 = T.Text,
        TextXAlignment = Enum.TextXAlignment.Left,
        TextTruncate = Enum.TextTruncate.AtEnd, ZIndex = 5,
    })
end

local serverVal = infoCard(1, "👥", "Servidor", "—")
local pingVal   = infoCard(2, "📡", "Ping", "—")

task.spawn(function()
    while serverVal.Parent or pingVal.Parent do
        local count = #Players:GetPlayers()
        local maxP = Players.MaxPlayers or 0
        local ping = 0
        pcall(function() ping = math.floor(LocalPlayer:GetNetworkPing() * 1000) end)
        if serverVal and serverVal.Parent then
            serverVal.Text = maxP > 0 and (count .. "/" .. maxP) or tostring(count)
        end
        if pingVal and pingVal.Parent then pingVal.Text = ping .. " ms" end
        task.wait(2)
    end
end)

print("[Hub] 4. Content OK")

--═══════════════════ CONFIG PANEL ═══════════════════
local SetBD = new("Frame", {
    Size = UDim2.fromScale(1, 1), BackgroundColor3 = Color3.new(0,0,0),
    BackgroundTransparency = 1, BorderSizePixel = 0, Visible = false,
    ZIndex = 200, Parent = Gui,
})
local SetP = new("Frame", {
    Size = UDim2.fromOffset(540, 620),
    Position = UDim2.new(0.5, 0, 0.5, 0),
    AnchorPoint = Vector2.new(0.5, 0.5),
    BackgroundColor3 = T.Bg, BorderSizePixel = 0, ZIndex = 201, Parent = SetBD,
})
corner(SetP, 20)
gradient(SetP, T.Bg, T.Bg2, 145)
stroke(SetP, T.StrokeL, 1)

local setH = new("Frame", {
    Size = UDim2.new(1, 0, 0, 60), BackgroundColor3 = T.Panel,
    BorderSizePixel = 0, ZIndex = 202, Parent = SetP,
})
corner(setH, 20)
new("Frame", {
    Size = UDim2.new(1, 0, 0, 24), Position = UDim2.new(0, 0, 1, -24),
    BackgroundColor3 = T.Panel, BorderSizePixel = 0, ZIndex = 202, Parent = setH,
})
local setL = new("Frame", {
    Size = UDim2.new(1, 0, 0, 2), BackgroundColor3 = T.Accent,
    BorderSizePixel = 0, ZIndex = 203, Parent = setH,
})
local setLGrad = gradient(setL, T.Accent, T.Accent2, 0)
regAccentGradient(setLGrad, "accent", "accent2")
mkLabel(setH, "Bold", {
    Size = UDim2.new(1, -60, 0, 24), Position = UDim2.fromOffset(22, 18),
    Text = "Configurações", TextSize = 17, TextColor3 = T.TextBright,
    TextXAlignment = Enum.TextXAlignment.Left, ZIndex = 203,
})
local setClose = mkButton(setH, "Bold", {
    Size = UDim2.fromOffset(36, 36), Position = UDim2.new(1, -52, 0.5, -18),
    BackgroundColor3 = T.Light, Text = "✕", TextSize = 16,
    TextColor3 = T.Text, ZIndex = 203,
})
corner(setClose, 10)
setClose.MouseEnter:Connect(function() tw(setClose, {BackgroundColor3 = T.Danger}, 0.2):Play() end)
setClose.MouseLeave:Connect(function() tw(setClose, {BackgroundColor3 = T.Light}, 0.2):Play() end)

local setS = new("ScrollingFrame", {
    Size = UDim2.new(1, -30, 1, -82), Position = UDim2.fromOffset(15, 76),
    BackgroundTransparency = 1, BorderSizePixel = 0,
    ScrollBarThickness = 4, ScrollBarImageColor3 = T.StrokeL,
    AutomaticCanvasSize = Enum.AutomaticSize.Y,
    CanvasSize = UDim2.new(0, 0, 0, 0), ZIndex = 202, Parent = SetP,
})
new("UIListLayout", {Padding = UDim.new(0, 12), SortOrder = Enum.SortOrder.LayoutOrder, Parent = setS})
new("UIPadding", {PaddingRight = UDim.new(0, 6), Parent = setS})

local function sTitle(parent, txt, ord)
    return mkLabel(parent, "Bold", {
        Size = UDim2.new(1, 0, 0, 16), Text = txt:upper(),
        TextSize = 10, TextColor3 = T.Soft,
        TextXAlignment = Enum.TextXAlignment.Left,
        LayoutOrder = ord or 1, ZIndex = 5,
    })
end
local function mCard(parent, h, ord)
    local c = new("Frame", {
        Size = UDim2.new(1, 0, 0, h),
        BackgroundColor3 = T.Panel, BorderSizePixel = 0,
        LayoutOrder = ord or 1, ZIndex = 4, Parent = parent,
    })
    corner(c, 14); stroke(c, T.Stroke, 1)
    return c
end

-- CORES
sTitle(setS, "Cor de destaque (20)", 1)
local ac2 = mCard(setS, 270, 2)
local ag = new("Frame", {
    Size = UDim2.new(1, -24, 1, -24), Position = UDim2.fromOffset(12, 12),
    BackgroundTransparency = 1, ZIndex = 5, Parent = ac2,
})
new("UIGridLayout", {
    CellSize = UDim2.fromOffset(92, 46),
    CellPadding = UDim2.fromOffset(6, 8),
    SortOrder = Enum.SortOrder.LayoutOrder, Parent = ag,
})
local accBtns = {}
local function refreshAcc()
    for name, d in pairs(accBtns) do
        local cur = (State.CustomAccent and d.color == State.CustomAccent) or
                    (not State.CustomAccent and d.color == DEFAULT_ACCENT)
        tw(d.stroke, {Thickness = cur and 3 or 0, Color = T.Glow}, 0.2):Play()
    end
end
for _, p in ipairs(Presets) do
    local cont = new("Frame", {
        Size = UDim2.fromOffset(92, 46),
        BackgroundTransparency = 1, ZIndex = 6, Parent = ag,
    })
    local btn = new("TextButton", {
        Size = UDim2.fromOffset(34, 34), Position = UDim2.fromOffset(6, 2),
        BackgroundColor3 = p.Color, Text = "",
        AutoButtonColor = false, ZIndex = 6, Parent = cont,
    })
    corner(btn, 8)
    local ring = stroke(btn, T.Glow, 0)
    mkLabel(cont, "Medium", {
        Size = UDim2.new(1, -44, 0, 14), Position = UDim2.fromOffset(46, 10),
        Text = p.Name, TextSize = 10, TextColor3 = T.Text,
        TextXAlignment = Enum.TextXAlignment.Left, ZIndex = 6,
    })
    btn.MouseEnter:Connect(function() tw(btn, {Size = UDim2.fromOffset(38, 38)}, 0.15):Play() end)
    btn.MouseLeave:Connect(function() tw(btn, {Size = UDim2.fromOffset(34, 34)}, 0.15):Play() end)
    btn.MouseButton1Click:Connect(function()
        CONFIG.AccentName = p.Name
        CONFIG.AccentRGB = nil
        setAccentColor(p.Color, true)
        refreshAcc()
    end)
    accBtns[p.Name] = {btn = btn, stroke = ring, color = p.Color}
end
refreshAcc()

-- FONTE
sTitle(setS, "Fonte", 3)
local fc = mCard(setS, 105, 4)
local fg = new("Frame", {
    Size = UDim2.new(1, -24, 1, -24), Position = UDim2.fromOffset(12, 12),
    BackgroundTransparency = 1, ZIndex = 5, Parent = fc,
})
new("UIGridLayout", {
    CellSize = UDim2.fromOffset(80, 34),
    CellPadding = UDim2.fromOffset(8, 8),
    SortOrder = Enum.SortOrder.LayoutOrder, Parent = fg,
})
local fBtns = {}
for _, f in ipairs(Fonts) do
    local b = new("TextButton", {
        Size = UDim2.fromOffset(80, 34), BackgroundColor3 = T.Panel,
        Text = f.Name, TextSize = 11, TextColor3 = T.Text, Font = f.Main,
        AutoButtonColor = false, ZIndex = 6, Parent = fg,
    })
    corner(b, 10)
    local s = stroke(b, T.Stroke, 1.5)
    b.MouseEnter:Connect(function() tw(s, {Color = T.Accent, Thickness = 2}, 0.15):Play() end)
    b.MouseLeave:Connect(function()
        if State.Font ~= f.Name then tw(s, {Color = T.Stroke, Thickness = 1.5}, 0.15):Play() end
    end)
    b.MouseButton1Click:Connect(function()
        State.Font = f.Name
        CONFIG.Font = f.Name
        applyFontAll()
        saveExternalConfig(CONFIG)
        for n, o in pairs(fBtns) do
            tw(o.stroke, {Color = (n == f.Name) and T.Accent or T.Stroke,
                          Thickness = (n == f.Name) and 2.5 or 1.5}, 0.2):Play()
        end
    end)
    fBtns[f.Name] = {button = b, stroke = s}
end

-- OPÇÕES
sTitle(setS, "Opções", 5)
local oc = mCard(setS, 90, 6)
local ol = new("Frame", {
    Size = UDim2.new(1, -20, 1, -20), Position = UDim2.fromOffset(10, 10),
    BackgroundTransparency = 1, Parent = oc,
})
new("UIListLayout", {Padding = UDim.new(0, 8), Parent = ol})

local function toggleRow(label, def, cb, configKey)
    local r = new("Frame", {
        Size = UDim2.new(1, 0, 0, 32), BackgroundTransparency = 1, Parent = ol,
    })
    mkLabel(r, "Medium", {
        Size = UDim2.new(1, -50, 1, 0), Text = label,
        TextSize = 13, TextColor3 = T.Text,
        TextXAlignment = Enum.TextXAlignment.Left,
    })
    local st = def and true or false
    local track = new("Frame", {
        Size = UDim2.fromOffset(38, 22), Position = UDim2.new(1, -38, 0.5, -11),
        BackgroundColor3 = st and T.Accent or T.StrokeL,
        BorderSizePixel = 0, Parent = r,
    })
    corner(track, 11)
    local knob = new("Frame", {
        Size = UDim2.fromOffset(18, 18),
        Position = st and UDim2.new(1, -20, 0.5, -9) or UDim2.fromOffset(2, 2),
        BackgroundColor3 = Color3.new(1,1,1), BorderSizePixel = 0, Parent = track,
    })
    corner(knob, 9)

    table.insert(toggleTracks, {track = track, getState = function() return st end})

    local clk = new("TextButton", {
        Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1,
        Text = "", Parent = r,
    })
    clk.MouseButton1Click:Connect(function()
        st = not st
        tw(track, {BackgroundColor3 = st and T.Accent or T.StrokeL}, 0.2):Play()
        tw(knob, {Position = st and UDim2.new(1,-20,0.5,-9) or UDim2.fromOffset(2,2)},
           0.25, Enum.EasingStyle.Back):Play()
        if configKey then
            CONFIG[configKey] = st
            saveExternalConfig(CONFIG)
        end
        if cb then cb(st) end
    end)
end
toggleRow("Animações", State.Animations, function(v) State.Animations = v end, "Animations")
toggleRow("Arrastar cabeçalho", State.DragEnabled, function(v) State.DragEnabled = v end, "DragEnabled")

-- RESET
local rst = mkButton(setS, "Bold", {
    Size = UDim2.new(1, 0, 0, 40), BackgroundColor3 = T.Panel,
    Text = "↺  Resetar cor", TextSize = 13, TextColor3 = T.Text, ZIndex = 5,
})
corner(rst, 10)
stroke(rst, T.StrokeL, 1)
rst.MouseEnter:Connect(function() tw(rst, {BackgroundColor3 = T.Light}, 0.15):Play() end)
rst.MouseLeave:Connect(function() tw(rst, {BackgroundColor3 = T.Panel}, 0.15):Play() end)
rst.MouseButton1Click:Connect(function()
    CONFIG.AccentName = "Blue"
    CONFIG.AccentRGB = nil
    setAccentColor(DEFAULT_ACCENT, true)
    refreshAcc()
end)
print("[Hub] 5. Config OK")

--═══════════════════ OPEN/CLOSE ═══════════════════
local function openSettings()
    if State.SettingsOpen or State.Destroyed then return end
    State.SettingsOpen = true
    SetBD.Visible = true; SetBD.BackgroundTransparency = 1
    SetP.Size = UDim2.fromOffset(0, 0)
    tw(SetBD, {BackgroundTransparency = 0.55}, 0.25):Play()
    tw(SetP, {Size = UDim2.fromOffset(540, 620)}, 0.35, Enum.EasingStyle.Back):Play()
end
local function closeSettings()
    if not State.SettingsOpen then return end
    State.SettingsOpen = false
    tw(SetBD, {BackgroundTransparency = 1}, 0.2):Play()
    tw(SetP, {Size = UDim2.fromOffset(0, 0)}, 0.2, Enum.EasingStyle.Back):Play()
    task.delay(0.22, function()
        if not State.SettingsOpen then SetBD.Visible = false end
    end)
end

local function openUI()
    if State.UIOpen or State.Destroyed then return end
    if REQUIRES_KEY and not KeyUnlocked then
        KeyScreen.Visible = true
        KeyScreen.BackgroundTransparency = 0.55
        KeyCard.Size = UDim2.fromOffset(0, 0)
        tw(KeyCard, {Size = UDim2.fromOffset(380, 270)}, 0.35, Enum.EasingStyle.Back):Play()
        if KeyBox then KeyBox:CaptureFocus() end
        return
    end
    State.UIOpen = true
    Root.Visible = true
    Main.Size = UDim2.fromOffset(0, 0)
    tw(Main, {Size = UDim2.fromOffset(W, H)}, State.Animations and 0.4 or 0.01, Enum.EasingStyle.Back):Play()
    buttonText.Text = "✕"
end
local function closeUI()
    if not State.UIOpen then return end
    State.UIOpen = false
    closeSettings()
    buttonText.Text = "✦"
    local t = tw(Main, {Size = UDim2.fromOffset(0, 0)}, State.Animations and 0.3 or 0.01, Enum.EasingStyle.Back)
    t.Completed:Connect(function() if not State.UIOpen then Root.Visible = false end end)
    t:Play()
end
local function killHub()
    if State.Destroyed then return end
    State.Destroyed = true; State.UIOpen = false; State.SettingsOpen = false
    SetBD.Visible = false
    tw(Main, {Size = UDim2.fromOffset(0, 0), BackgroundTransparency = 1}, 0.35, Enum.EasingStyle.Back):Play()
    tw(ToggleBtn, {Size = UDim2.fromOffset(0, 0), BackgroundTransparency = 1}, 0.35, Enum.EasingStyle.Back):Play()
    task.delay(0.4, function() pcall(function() Gui:Destroy() end) end)
end

HubAPI.Open = openUI
HubAPI.Close = closeUI
HubAPI.Destroy = killHub

GearBtn.MouseButton1Click:Connect(openSettings)
setClose.MouseButton1Click:Connect(closeSettings)
ToggleBtn.MouseButton1Click:Connect(function()
    if State.Destroyed then return end
    if State.UIOpen then closeUI() else openUI() end
end)
CloseBtn.MouseButton1Click:Connect(killHub)
print("[Hub] 6. Bindings OK")

--═══════════════════ KEY VERIFY ═══════════════════
if REQUIRES_KEY and KeyButton then
    local function tryUnlock()
        local entered = KeyBox.Text:gsub("%s+", "")
        if entered == CONFIG.KEY then
            KeyUnlocked = true
            KeyStatus.Text = "✅ Key válida!"
            KeyStatus.TextColor3 = T.Success
            task.wait(0.35)
            tw(KeyScreen, {BackgroundTransparency = 1}, 0.3):Play()
            local t = tw(KeyCard, {Size = UDim2.fromOffset(0, 0)}, 0.3, Enum.EasingStyle.Back)
            t.Completed:Connect(function() KeyScreen.Visible = false end)
            t:Play()
            State.UIOpen = true
            Root.Visible = true
            Main.Size = UDim2.fromOffset(0, 0)
            tw(Main, {Size = UDim2.fromOffset(W, H)}, 0.4, Enum.EasingStyle.Back):Play()
            buttonText.Text = "✕"
        else
            KeyStatus.Text = "❌ Key inválida"
            KeyStatus.TextColor3 = T.Danger
            local orig = KeyCard.Position
            for i = 1, 4 do
                tw(KeyCard, {Position = orig + UDim2.fromOffset(i % 2 == 0 and 8 or -8, 0)}, 0.05):Play()
                task.wait(0.05)
            end
            tw(KeyCard, {Position = orig}, 0.1):Play()
        end
    end
    KeyButton.MouseButton1Click:Connect(tryUnlock)
    KeyBox.FocusLost:Connect(function(enter) if enter then tryUnlock() end end)
end

--═══════════════════ DRAG ═══════════════════
local drag, dS, sP = false, nil, nil
Header.InputBegan:Connect(function(input)
    if not State.DragEnabled then return end
    if input.UserInputType == Enum.UserInputType.MouseButton1
    or input.UserInputType == Enum.UserInputType.Touch then
        drag = true; dS = input.Position; sP = Main.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then drag = false end
        end)
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if State.Destroyed then return end
    if drag and (input.UserInputType == Enum.UserInputType.MouseMovement
    or input.UserInputType == Enum.UserInputType.Touch) then
        local d = input.Position - dS
        Main.Position = UDim2.new(sP.X.Scale, sP.X.Offset + d.X, sP.Y.Scale, sP.Y.Offset + d.Y)
    end
end)

--═══════════════════ SHORTCUTS ═══════════════════
UserInputService.InputBegan:Connect(function(input, processed)
    if State.Destroyed or processed then return end
    if input.KeyCode == Enum.KeyCode.Escape then
        if State.SettingsOpen then closeSettings()
        elseif State.UIOpen then closeUI() end
    elseif input.KeyCode == Enum.KeyCode.F1 then
        if State.UIOpen then closeUI() else openUI() end
    end
end)

--═══════════════════ APLICA CONFIG INICIAL ═══════════════════
do
    local color = nil
    if CONFIG.AccentRGB then
        color = Color3.fromRGB(CONFIG.AccentRGB.R or 120, CONFIG.AccentRGB.G or 175, CONFIG.AccentRGB.B or 255)
        State.CustomAccent = color
    elseif CONFIG.AccentName then
        color = findPreset(CONFIG.AccentName)
        State.CustomAccent = color
    end
    if color then setAccentColor(color, false) end
    applyFontAll()
    saveExternalConfig(CONFIG)
end

--═══════════════════ ABRE AUTOMATICAMENTE ═══════════════════
task.delay(0.2, function()
    if not State.Destroyed and not State.UIOpen then
        openUI()
    end
end)

print("[Hub] ✅ v33 PRONTO")
print("[Hub] 👤 Avatar: " .. (CONFIG.AVATAR_IMAGE ~= "" and "custom" or "do jogador"))
print("[Hub] 🎭 Botão: " .. (CONFIG.BUTTON_IMAGE ~= "" and "dark face" or "preto"))
