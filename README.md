local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- --- CONFIGURAÇÕES GERAIS ---
local Settings = {
    AimbotEnabled = false,
    TeamCheck = false,
    ShowFOV = false,
    FOVRadius = 100,
    FOVColor = Color3.fromRGB(255, 255, 255),
    TargetPart = "Head", 
    AimKey = Enum.UserInputType.MouseButton2,
    AimDistance = 500,
    Smoothing = 2,
    Box = false,
    Skeleton = false,
    Tracers = false,
    Distance = false,
    Names = false,
    Health = false,
    LocalPlayer = false,
    ESPColor = Color3.fromRGB(255, 0, 85),
    Thickness = 1,
    BoxThickness = 2,
    MaxDistance = 500,
    FlyEnabled = false,
    IsFlying = false,
    FlySpeed = 20,
    FlyBoost = 350,
    FlyKey = Enum.KeyCode.CapsLock,
    InfJump = false,
    SelectedPlayer = nil,
    PuxarLoop = false,
    SpectateEnabled = false,
    SpectateDist = 15,
    SpectateRotation = 0
}

local Cache = {}
local isAiming = false

-- --- BIBLIOTECA DE UI (Xanax Library) ---
local XanaxLib = {} -- Nomeado especificamente para evitar conflitos nil
local theme = {
    bg = Color3.fromRGB(8, 8, 10),
    sidebar = Color3.fromRGB(12, 12, 15),
    element = Color3.fromRGB(18, 18, 22),
    elementHover = Color3.fromRGB(25, 25, 30),
    accent = Color3.fromRGB(140, 0, 255),
    accentGradient = Color3.fromRGB(90, 0, 180),
    text = Color3.fromRGB(255, 255, 255),
    subtext = Color3.fromRGB(150, 150, 155),
    stroke = Color3.fromRGB(30, 30, 35)
}

local function create(class, props)
    local obj = Instance.new(class)
    for i, v in pairs(props) do obj[i] = v end
    return obj
end

local function createDrawing(class, props)
    local obj = Drawing.new(class)
    for i, v in pairs(props) do obj[i] = v end
    return obj
end

local function tween(obj, info, props)
    local t = TweenService:Create(obj, TweenInfo.new(info, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), props)
    t:Play()
    return t
end

function XanaxLib:CreateWindow(cfg)
    local Window = {CurrentTab = nil}
    local targetParent = (game:GetService("CoreGui") or player:WaitForChild("PlayerGui"))

    local gui = create("ScreenGui", {Name = "XanaxHub_V3", Parent = targetParent, ResetOnSpawn = false})
    local mainShadow = create("Frame", {Size = UDim2.new(0, 650, 0, 450), Position = UDim2.new(0.5, -325, 0.5, -225), BackgroundColor3 = Color3.new(0,0,0), BackgroundTransparency = 0.5, Parent = gui})
    create("UICorner", {CornerRadius = UDim.new(0, 15), Parent = mainShadow})
    
    local main = create("Frame", {Size = UDim2.new(1, 0, 1, 0), BackgroundColor3 = theme.bg, Parent = mainShadow})
    create("UICorner", {CornerRadius = UDim.new(0, 12), Parent = main})
    create("UIStroke", {Color = theme.stroke, Thickness = 1.2, Parent = main})

    local header = create("Frame", {Size = UDim2.new(1, 0, 0, 50), BackgroundTransparency = 1, Parent = main})
    local title = create("TextLabel", {Size = UDim2.new(1, -40, 1, 0), Position = UDim2.new(0, 20, 0, 0), BackgroundTransparency = 1, Text = cfg.Title or "XANAX HUB", TextColor3 = theme.text, Font = Enum.Font.GothamBold, TextSize = 16, TextXAlignment = Enum.TextXAlignment.Left, Parent = header})

    -- Alternar visibilidade (RightShift)
    UserInputService.InputBegan:Connect(function(input)
        if input.KeyCode == Enum.KeyCode.RightShift then
            mainShadow.Visible = not mainShadow.Visible
        end
    end)

    local sidebar = create("Frame", {Size = UDim2.new(0, 160, 1, -50), Position = UDim2.new(0, 0, 0, 50), BackgroundColor3 = theme.sidebar, Parent = main})
    create("UICorner", {CornerRadius = UDim.new(0, 12), Parent = sidebar})
    
    local tabContainer = create("ScrollingFrame", {Size = UDim2.new(1, 0, 1, -20), Position = UDim2.new(0, 0, 0, 10), BackgroundTransparency = 1, ScrollBarThickness = 0, Parent = sidebar})
    create("UIListLayout", {Parent = tabContainer, Padding = UDim.new(0, 2), HorizontalAlignment = Enum.HorizontalAlignment.Center})

    local content = create("Frame", {Size = UDim2.new(1, -160, 1, -50), Position = UDim2.new(0, 160, 0, 50), BackgroundTransparency = 1, Parent = main})

    -- Drag Logic
    local dragging, dragStart, startPos
    header.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = true; dragStart = input.Position; startPos = mainShadow.Position end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = input.Position - dragStart
            mainShadow.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
    UserInputService.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end end)

    local pages = {}
    function Window:CreateTab(name)
        local Tab = {}
        local tabBtn = create("TextButton", {Size = UDim2.new(0, 140, 0, 35), BackgroundTransparency = 1, Text = "", Parent = tabContainer})
        local tabLabel = create("TextLabel", {Size = UDim2.new(1, -10, 1, 0), Position = UDim2.new(0, 10, 0, 0), BackgroundTransparency = 1, Text = name, TextColor3 = theme.subtext, Font = Enum.Font.GothamMedium, TextSize = 13, TextXAlignment = Enum.TextXAlignment.Left, Parent = tabBtn})
        local page = create("ScrollingFrame", {Size = UDim2.new(1, 0, 1, 0), BackgroundTransparency = 1, ScrollBarThickness = 2, Visible = false, Parent = content})
        create("UIPadding", {PaddingLeft = UDim.new(0, 15), PaddingRight = UDim.new(0, 15), PaddingTop = UDim.new(0, 15), Parent = page})
        local layout = create("UIListLayout", {Parent = page, Padding = UDim.new(0, 8)})
        
        layout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() page.CanvasSize = UDim2.new(0, 0, 0, layout.AbsoluteContentSize.Y + 20) end)

        tabBtn.MouseButton1Click:Connect(function()
            for _, v in pairs(pages) do v.pg.Visible = false; v.label.TextColor3 = theme.subtext end
            page.Visible = true; tabLabel.TextColor3 = theme.text
        end)
        table.insert(pages, {label = tabLabel, pg = page})
        if #pages == 1 then page.Visible = true; tabLabel.TextColor3 = theme.text end

        function Tab:CreateLabel(txt)
            local l = create("TextLabel", {Size = UDim2.new(1, 0, 0, 25), BackgroundTransparency = 1, Text = txt, TextColor3 = theme.subtext, Font = Enum.Font.Gotham, TextSize = 12, TextXAlignment = Enum.TextXAlignment.Left, Parent = page})
            return {SetText = function(_, new) l.Text = new end}
        end

        function Tab:CreateButton(cfg)
            local b = create("TextButton", {Size = UDim2.new(1, 0, 0, 38), BackgroundColor3 = theme.element, Text = cfg.Name, TextColor3 = theme.text, Font = Enum.Font.GothamMedium, TextSize = 13, Parent = page})
            create("UICorner", {CornerRadius = UDim.new(0, 6), Parent = b})
            b.MouseButton1Click:Connect(cfg.Callback)
        end

        function Tab:CreateToggle(cfg)
            local state = cfg.Default or false
            local t = create("TextButton", {Size = UDim2.new(1, 0, 0, 40), BackgroundColor3 = theme.element, Text = "", Parent = page})
            create("UICorner", {CornerRadius = UDim.new(0, 6), Parent = t})
            local l = create("TextLabel", {Size = UDim2.new(1, -50, 1, 0), Position = UDim2.new(0, 12, 0, 0), BackgroundTransparency = 1, Text = cfg.Name, TextColor3 = theme.text, Font = Enum.Font.Gotham, TextSize = 13, TextXAlignment = Enum.TextXAlignment.Left, Parent = t})
            local box = create("Frame", {Size = UDim2.new(0, 34, 0, 18), Position = UDim2.new(1, -45, 0.5, -9), BackgroundColor3 = state and theme.accent or theme.bg, Parent = t})
            create("UICorner", {CornerRadius = UDim.new(1, 0), Parent = box})
            
            t.MouseButton1Click:Connect(function()
                state = not state
                box.BackgroundColor3 = state and theme.accent or theme.bg
                cfg.Callback(state)
            end)
        end

        function Tab:CreateSlider(cfg)
            local val = cfg.Default or cfg.Min
            local s = create("Frame", {Size = UDim2.new(1, 0, 0, 50), BackgroundColor3 = theme.element, Parent = page})
            create("UICorner", {CornerRadius = UDim.new(0, 6), Parent = s})
            local l = create("TextLabel", {Size = UDim2.new(1, -20, 0, 25), Position = UDim2.new(0, 10, 0, 5), BackgroundTransparency = 1, Text = cfg.Name .. ": " .. val, TextColor3 = theme.text, Font = Enum.Font.Gotham, TextSize = 12, TextXAlignment = Enum.TextXAlignment.Left, Parent = s})
            local bg = create("Frame", {Size = UDim2.new(1, -20, 0, 4), Position = UDim2.new(0, 10, 0, 35), BackgroundColor3 = theme.bg, Parent = s})
            local fill = create("Frame", {Size = UDim2.new((val - cfg.Min)/(cfg.Max-cfg.Min), 0, 1, 0), BackgroundColor3 = theme.accent, Parent = bg})
            
            local function update(input)
                local per = math.clamp((input.Position.X - bg.AbsolutePosition.X) / bg.AbsoluteSize.X, 0, 1)
                val = math.floor(cfg.Min + (cfg.Max - cfg.Min) * per)
                l.Text = cfg.Name .. ": " .. val
                fill.Size = UDim2.new(per, 0, 1, 0)
                cfg.Callback(val)
            end
            local active = false
            s.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then active = true; update(i) end end)
            UserInputService.InputChanged:Connect(function(i) if active and i.UserInputType == Enum.UserInputType.MouseMovement then update(i) end end)
            UserInputService.InputEnded:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then active = false end end)
        end

        function Tab:CreateKeybind(cfg)
            local key = cfg.Default
            local k = create("TextButton", {Size = UDim2.new(1, 0, 0, 40), BackgroundColor3 = theme.element, Text = "", Parent = page})
            create("UICorner", {CornerRadius = UDim.new(0, 6), Parent = k})
            create("TextLabel", {Size = UDim2.new(1, -100, 1, 0), Position = UDim2.new(0, 12, 0, 0), BackgroundTransparency = 1, Text = cfg.Name, TextColor3 = theme.text, Font = Enum.Font.Gotham, TextSize = 13, TextXAlignment = Enum.TextXAlignment.Left, Parent = k})
            local val = create("TextLabel", {Size = UDim2.new(0, 80, 0, 24), Position = UDim2.new(1, -90, 0.5, -12), BackgroundColor3 = theme.bg, Text = tostring(key):gsub("Enum.KeyCode.", ""), TextColor3 = theme.accent, Font = Enum.Font.GothamBold, TextSize = 11, Parent = k})
            create("UICorner", {CornerRadius = UDim.new(0, 4), Parent = val})
            
            k.MouseButton1Click:Connect(function()
                val.Text = "..."
                local conn; conn = UserInputService.InputBegan:Connect(function(i)
                    if i.UserInputType ~= Enum.UserInputType.MouseMovement then
                        key = i.KeyCode ~= Enum.KeyCode.Unknown and i.KeyCode or i.UserInputType
                        val.Text = tostring(key):gsub("Enum.", ""):gsub("KeyCode.", ""):gsub("UserInputType.", "")
                        cfg.Callback(key)
                        conn:Disconnect()
                    end
                end)
            end)
        end

        function Tab:CreateColorPicker(cfg)
            local col = cfg.Default
            local cp = create("TextButton", {Size = UDim2.new(1, 0, 0, 40), BackgroundColor3 = theme.element, Text = "", Parent = page})
            create("UICorner", {CornerRadius = UDim.new(0, 6), Parent = cp})
            create("TextLabel", {Size = UDim2.new(1, -60, 1, 0), Position = UDim2.new(0, 12, 0, 0), BackgroundTransparency = 1, Text = cfg.Name, TextColor3 = theme.text, Font = Enum.Font.Gotham, TextSize = 13, TextXAlignment = Enum.TextXAlignment.Left, Parent = cp})
            local preview = create("Frame", {Size = UDim2.new(0, 30, 0, 20), Position = UDim2.new(1, -40, 0.5, -10), BackgroundColor3 = col, Parent = cp})
            create("UICorner", {CornerRadius = UDim.new(0, 4), Parent = preview})
            cp.MouseButton1Click:Connect(function() cfg.Callback(col); preview.BackgroundColor3 = col end)
        end

        return Tab
    end
    return Window
end

-- --- FUNÇÕES DE LÓGICA (AIMBOT / ESP) ---
local function getClosestPlayer()
    local target = nil
    local shortestDistance = math.huge
    local mousePos = UserInputService:GetMouseLocation()

    for _, p in pairs(Players:GetPlayers()) do
        if p ~= player and p.Character then
            if Settings.TeamCheck and p.Team == player.Team then continue end
            local char = p.Character
            local part = char:FindFirstChild(Settings.TargetPart)
            local hum = char:FindFirstChild("Humanoid")
            if part and hum and hum.Health > 0 then
                local screenPos, onScreen = camera:WorldToViewportPoint(part.Position)
                if onScreen then
                    local distFromMouse = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude
                    if distFromMouse <= Settings.FOVRadius then
                        if distFromMouse < shortestDistance then
                            target = part
                            shortestDistance = distFromMouse
                        end
                    end
                end
            end
        end
    end
    return target
end

-- Detectar Tecla de Aim
UserInputService.InputBegan:Connect(function(input, gpe)
    if gpe then return end
    if input.UserInputType == Settings.AimKey or input.KeyCode == Settings.AimKey then isAiming = true end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Settings.AimKey or input.KeyCode == Settings.AimKey then isAiming = false end
end)

-- --- INICIALIZAÇÃO DA INTERFACE ---
-- Isso agora roda na linha ~250, com XanaxLib garantido como definido.
local Window = XanaxLib:CreateWindow({Title = "Xanax Hub V3"})

local AimTab = Window:CreateTab("Aimbot")
local VisTab = Window:CreateTab("Visuals")
local SelfTab = Window:CreateTab("Pessoal")
local PlayersTab = Window:CreateTab("Jogadores")

-- FOV Circle Drawing
local FOVCircle = createDrawing("Circle", {Thickness = 1, NumSides = 100, Filled = false, Transparency = 1, Visible = false})

-- Aimbot Configs
AimTab:CreateToggle({Name = "Aimbot", Default = false, Callback = function(v) Settings.AimbotEnabled = v end})
AimTab:CreateToggle({Name = "Show FOV", Default = false, Callback = function(v) Settings.ShowFOV = v end})
AimTab:CreateToggle({Name = "Team Check", Default = false, Callback = function(v) Settings.TeamCheck = v end})
AimTab:CreateKeybind({Name = "Keybind", Default = Enum.UserInputType.MouseButton2, Callback = function(v) Settings.AimKey = v end})
AimTab:CreateSlider({Name = "Smoothing", Min = 1, Max = 10, Default = 2, Callback = function(v) Settings.Smoothing = v end})
AimTab:CreateSlider({Name = "Fov Radius", Min = 50, Max = 500, Default = 100, Callback = function(v) Settings.FOVRadius = v end})
AimTab:CreateColorPicker({Name = "Fov Color", Default = Settings.FOVColor, Callback = function(v) Settings.FOVColor = v; FOVCircle.Color = v end})

-- Visuals Configs
VisTab:CreateLabel("Componentes Visuais")
VisTab:CreateToggle({Name = "Usernames", Default = false, Callback = function(v) Settings.Names = v end})
VisTab:CreateToggle({Name = "Box Corner", Default = false, Callback = function(v) Settings.Box = v end})
VisTab:CreateToggle({Name = "Health Bar", Default = false, Callback = function(v) Settings.Health = v end})
VisTab:CreateToggle({Name = "Skeleton", Default = false, Callback = function(v) Settings.Skeleton = v end})
VisTab:CreateToggle({Name = "Tracers", Default = false, Callback = function(v) Settings.Tracers = v end})
VisTab:CreateColorPicker({Name = "Cor do ESP", Default = Settings.ESPColor, Callback = function(v) Settings.ESPColor = v end})

-- Fly Configs
SelfTab:CreateToggle({Name = "Fly", Default = false, Callback = function(v) Settings.FlyEnabled = v end})
SelfTab:CreateKeybind({Name = "Keybind Fly", Default = Enum.KeyCode.CapsLock, Callback = function(key) Settings.FlyKey = key end})
SelfTab:CreateSlider({Name = "Speed Fly", Min = 10, Max = 300, Default = 20, Callback = function(v) Settings.FlySpeed = v end})

-- Players Tab
local SelectedLabel = PlayersTab:CreateLabel("Selecionado: Nenhum")
PlayersTab:CreateButton({Name = "Teleport", Callback = function()
    if Settings.SelectedPlayer and Settings.SelectedPlayer.Character then
        player.Character:MoveTo(Settings.SelectedPlayer.Character.HumanoidRootPart.Position)
    end
end})

-- --- LOOP PRINCIPAL ---
RunService.RenderStepped:Connect(function()
    -- FOV Logic
    FOVCircle.Visible = Settings.ShowFOV
    FOVCircle.Radius = Settings.FOVRadius
    FOVCircle.Position = UserInputService:GetMouseLocation()
    
    -- Aimbot Logic
    if Settings.AimbotEnabled and isAiming then
        local target = getClosestPlayer()
        if target then
            local screenPos, onScreen = camera:WorldToViewportPoint(target.Position)
            if onScreen then
                local mousePos = UserInputService:GetMouseLocation()
                local moveX = (screenPos.X - mousePos.X) / Settings.Smoothing
                local moveY = (screenPos.Y - mousePos.Y) / Settings.Smoothing
                if mousemoverel then
                    mousemoverel(moveX, moveY)
                end
            end
        end
    end

    -- ESP Logic (Resumo para performance)
    -- Para cada jogador, você usaria o Cache[p] para atualizar os desenhos 2D aqui.
end)
