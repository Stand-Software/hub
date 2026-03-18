local Library = {}

local UIS = game:GetService("UserInputService")
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local theme = {
    bg = Color3.fromRGB(8, 8, 10),
    sidebar = Color3.fromRGB(12, 12, 15),
    element = Color3.fromRGB(18, 18, 22),
    elementHover = Color3.fromRGB(25, 25, 30),
    accent = Color3.fromRGB(140, 0, 255),
    accentGradient = Color3.fromRGB(90, 0, 180),
    text = Color3.fromRGB(255, 255, 255),
    subtext = Color3.fromRGB(150, 150, 155),
    stroke = Color3.fromRGB(30, 30, 35),
    accentStroke = Color3.fromRGB(140, 0, 255)
}

local function create(class, props)
    local obj = Instance.new(class)
    for i, v in pairs(props) do obj[i] = v end
    return obj
end

local function tween(obj, info, props)
    TweenService:Create(obj, TweenInfo.new(info, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), props):Play()
end

function Library:CreateWindow(cfg)
    local Window = {}
    
    -- Tenta encontrar o CoreGui (comum em executores) para ficar acima do menu ESC
    local targetParent = playerGui
    local s, e = pcall(function()
        targetParent = game:GetService("CoreGui")
    end)

    local gui = create("ScreenGui", {
        Name = "XanaxHub_V2",
        ResetOnSpawn = false,
        Parent = targetParent,
        IgnoreGuiInset = true,
        DisplayOrder = 2147483647, -- Ordem máxima para ficar por cima de tudo
        ZIndexBehavior = Enum.ZIndexBehavior.Global
    })

    local main = create("Frame", {
        Name = "Main",
        Size = UDim2.new(0, 750, 0, 500), 
        Position = UDim2.new(0.5, -375, 0.5, -250),
        BackgroundColor3 = theme.bg,
        BorderSizePixel = 0,
        Parent = gui
    })

    create("UICorner", {CornerRadius = UDim.new(0, 12), Parent = main})
    create("UIStroke", {Color = theme.stroke, Thickness = 1.5, Parent = main})

    -- Lógica para fechar/abrir com RightShift
    UIS.InputBegan:Connect(function(input, gpe)
        if input.KeyCode == Enum.KeyCode.RightShift then
            main.Visible = not main.Visible
        end
    end)

    local header = create("Frame", {
        Size = UDim2.new(1, 0, 0, 60),
        BackgroundTransparency = 1,
        Parent = main
    })

    create("TextLabel", {
        Size = UDim2.new(1, -40, 1, 0),
        Position = UDim2.new(0, 25, 0, 0),
        BackgroundTransparency = 1,
        Text = cfg.Title or "XANAX HUB V2",
        TextColor3 = theme.text,
        Font = Enum.Font.GothamBold,
        TextSize = 20,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = header
    })

    local sidebar = create("Frame", {
        Size = UDim2.new(0, 190, 1, -61),
        Position = UDim2.new(0, 0, 0, 61),
        BackgroundColor3 = theme.sidebar,
        BorderSizePixel = 0,
        Parent = main
    })
    
    create("UICorner", {CornerRadius = UDim.new(0, 12), Parent = sidebar})
    local sidebarFix = create("Frame", {Size = UDim2.new(0, 20, 1, 0), Position = UDim2.new(1, -20, 0, 0), BackgroundColor3 = theme.sidebar, BorderSizePixel = 0, Parent = sidebar})

    local tabContainer = create("ScrollingFrame", {
        Size = UDim2.new(1, 0, 1, -20),
        Position = UDim2.new(0, 0, 0, 10),
        BackgroundTransparency = 1,
        ScrollBarThickness = 0,
        Parent = sidebar
    })

    create("UIListLayout", {Parent = tabContainer, Padding = UDim.new(0, 5), HorizontalAlignment = Enum.HorizontalAlignment.Center})

    local content = create("Frame", {
        Size = UDim2.new(1, -190, 1, -61),
        Position = UDim2.new(0, 190, 0, 61),
        BackgroundTransparency = 1,
        Parent = main
    })

    local pages = {}
    
    local dragging, dragStart, startPos
    header.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = main.Position
        end
    end)
    UIS.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = input.Position - dragStart
            main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
    UIS.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
    end)

    function Window:CreateTab(name)
        local Tab = {}
        local tabBtn = create("TextButton", {Size = UDim2.new(0, 170, 0, 42), BackgroundTransparency = 1, Text = name, TextColor3 = theme.subtext, Font = Enum.Font.GothamMedium, TextSize = 14, AutoButtonColor = false, Parent = tabContainer})
        local indicator = create("Frame", {Size = UDim2.new(0, 3, 0, 20), Position = UDim2.new(0, 0, 0.5, -10), BackgroundColor3 = theme.accent, BackgroundTransparency = 1, Parent = tabBtn})
        local page = create("ScrollingFrame", {Size = UDim2.new(1, 0, 1, 0), BackgroundTransparency = 1, ScrollBarThickness = 2, Visible = false, Parent = content})
        create("UIPadding", {PaddingLeft = UDim.new(0, 25), PaddingRight = UDim.new(0, 25), PaddingTop = UDim.new(0, 25), PaddingBottom = UDim.new(0, 25), Parent = page})
        local layout = create("UIListLayout", {Parent = page, Padding = UDim.new(0, 12), SortOrder = Enum.SortOrder.LayoutOrder})
        
        layout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() 
            page.CanvasSize = UDim2.new(0, 0, 0, layout.AbsoluteContentSize.Y + 20) 
        end)

        table.insert(pages, {btn = tabBtn, pg = page, ind = indicator})

        local function selectTab()
            for _, v in pairs(pages) do
                v.pg.Visible = false
                tween(v.btn, 0.3, {TextColor3 = theme.subtext})
                tween(v.ind, 0.3, {BackgroundTransparency = 1})
            end
            page.Visible = true
            tween(tabBtn, 0.3, {TextColor3 = theme.text})
            tween(indicator, 0.3, {BackgroundTransparency = 0})
        end

        tabBtn.MouseButton1Click:Connect(selectTab)
        if #pages == 1 then selectTab() end

        function Tab:CreateLabel(text)
            return create("TextLabel", {Size = UDim2.new(1, 0, 0, 20), BackgroundTransparency = 1, Text = text, TextColor3 = theme.subtext, Font = Enum.Font.Gotham, TextSize = 13, TextXAlignment = Enum.TextXAlignment.Left, Parent = page})
        end

        function Tab:CreateToggle(cfg)
            local state = cfg.Default or false
            local holder = create("TextButton", {Size = UDim2.new(1, 0, 0, 50), BackgroundColor3 = theme.element, AutoButtonColor = false, Text = "", Parent = page})
            create("UICorner", {CornerRadius = UDim.new(0, 8), Parent = holder})
            create("UIStroke", {Color = theme.stroke, Parent = holder})
            create("TextLabel", {Size = UDim2.new(1, -60, 1, 0), Position = UDim2.new(0, 15, 0, 0), BackgroundTransparency = 1, Text = cfg.Name, TextColor3 = theme.text, Font = Enum.Font.Gotham, TextSize = 14, TextXAlignment = Enum.TextXAlignment.Left, Parent = holder})
            local box = create("Frame", {Size = UDim2.new(0, 22, 0, 22), Position = UDim2.new(1, -37, 0.5, -11), BackgroundColor3 = state and theme.accent or theme.bg, Parent = holder})
            create("UICorner", {CornerRadius = UDim.new(0, 5), Parent = box})

            holder.MouseButton1Click:Connect(function()
                state = not state
                tween(box, 0.2, {BackgroundColor3 = state and theme.accent or theme.bg})
                if cfg.Callback then cfg.Callback(state) end
            end)
        end

        function Tab:CreateKeybind(cfg)
            local key = cfg.Default or Enum.KeyCode.F
            local holder = create("TextButton", {Size = UDim2.new(1, 0, 0, 50), BackgroundColor3 = theme.element, AutoButtonColor = false, Text = "", Parent = page})
            create("UICorner", {CornerRadius = UDim.new(0, 8), Parent = holder})
            create("UIStroke", {Color = theme.stroke, Parent = holder})
            
            create("TextLabel", {Size = UDim2.new(1, -60, 1, 0), Position = UDim2.new(0, 15, 0, 0), BackgroundTransparency = 1, Text = cfg.Name, TextColor3 = theme.text, Font = Enum.Font.Gotham, TextSize = 14, TextXAlignment = Enum.TextXAlignment.Left, Parent = holder})
            
            local keyLabel = create("TextLabel", {Size = UDim2.new(0, 80, 0, 30), Position = UDim2.new(1, -95, 0.5, -15), BackgroundColor3 = theme.bg, Text = (typeof(key) == "EnumItem" and key.Name or "Mouse2"), TextColor3 = theme.accent, Font = Enum.Font.GothamMedium, TextSize = 13, Parent = holder})
            create("UICorner", {CornerRadius = UDim.new(0, 6), Parent = keyLabel})
            create("UIStroke", {Color = theme.stroke, Parent = keyLabel})

            local waiting = false
            holder.MouseButton1Click:Connect(function()
                waiting = true
                keyLabel.Text = "..."
            end)

            UIS.InputBegan:Connect(function(input)
                if waiting then
                    if input.UserInputType == Enum.UserInputType.Keyboard then
                        key = input.KeyCode
                        keyLabel.Text = key.Name
                        waiting = false
                        if cfg.Callback then cfg.Callback(key) end
                    elseif input.UserInputType == Enum.UserInputType.MouseButton2 then
                        key = Enum.UserInputType.MouseButton2
                        keyLabel.Text = "Mouse2"
                        waiting = false
                        if cfg.Callback then cfg.Callback(key) end
                    end
                end
            end)
        end

        function Tab:CreateButton(cfg)
            local btn = create("TextButton", {Size = UDim2.new(1, 0, 0, 50), BackgroundColor3 = theme.element, AutoButtonColor = false, Text = cfg.Name, TextColor3 = theme.text, Font = Enum.Font.GothamMedium, TextSize = 14, Parent = page})
            create("UICorner", {CornerRadius = UDim.new(0, 8), Parent = btn})
            create("UIStroke", {Color = theme.stroke, Parent = btn})
            btn.MouseButton1Click:Connect(function()
                local old = btn.BackgroundColor3
                btn.BackgroundColor3 = theme.accent
                task.wait(0.1)
                tween(btn, 0.3, {BackgroundColor3 = old})
                if cfg.Callback then cfg.Callback() end
            end)
        end

        function Tab:CreateSlider(cfg)
            local min, max = cfg.Min or 0, cfg.Max or 100
            local val = cfg.Default or min
            local holder = create("Frame", {Size = UDim2.new(1, 0, 0, 65), BackgroundColor3 = theme.element, Parent = page})
            create("UICorner", {CornerRadius = UDim.new(0, 8), Parent = holder})
            create("UIStroke", {Color = theme.stroke, Parent = holder})
            create("TextLabel", {Size = UDim2.new(1, -30, 0, 30), Position = UDim2.new(0, 15, 0, 5), BackgroundTransparency = 1, Text = cfg.Name, TextColor3 = theme.text, Font = Enum.Font.Gotham, TextSize = 14, TextXAlignment = Enum.TextXAlignment.Left, Parent = holder})
            local valLabel = create("TextLabel", {Size = UDim2.new(0, 50, 0, 30), Position = UDim2.new(1, -65, 0, 5), BackgroundTransparency = 1, Text = tostring(val), TextColor3 = theme.subtext, Font = Enum.Font.Gotham, TextSize = 13, TextXAlignment = Enum.TextXAlignment.Right, Parent = holder})
            local barBg = create("Frame", {Size = UDim2.new(1, -30, 0, 6), Position = UDim2.new(0, 15, 0, 45), BackgroundColor3 = theme.bg, Parent = holder})
            create("UICorner", {CornerRadius = UDim.new(1, 0), Parent = barBg})
            local barFill = create("Frame", {Size = UDim2.new((val - min) / (max - min), 0, 1, 0), BackgroundColor3 = theme.accent, Parent = barBg})
            create("UICorner", {CornerRadius = UDim.new(1, 0), Parent = barFill})

            local function update(input)
                local pos = math.clamp((input.Position.X - barBg.AbsolutePosition.X) / barBg.AbsoluteSize.X, 0, 1)
                val = math.floor(min + (max - min) * pos)
                valLabel.Text = tostring(val)
                barFill.Size = UDim2.new(pos, 0, 1, 0)
                if cfg.Callback then cfg.Callback(val) end
            end

            local sliding = false
            holder.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then sliding = true update(input) end end)
            UIS.InputChanged:Connect(function(input) if sliding and input.UserInputType == Enum.UserInputType.MouseMovement then update(input) end end)
            UIS.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then sliding = false end end)
        end

        function Tab:CreateColorPicker(cfg)
            local color = cfg.Default or Color3.fromRGB(255, 0, 85)
            local holder = create("TextButton", {Size = UDim2.new(1, 0, 0, 50), BackgroundColor3 = theme.element, AutoButtonColor = false, Text = "", Parent = page})
            create("UICorner", {CornerRadius = UDim.new(0, 8), Parent = holder})
            create("UIStroke", {Color = theme.stroke, Parent = holder})
            create("TextLabel", {Size = UDim2.new(1, -60, 1, 0), Position = UDim2.new(0, 15, 0, 0), BackgroundTransparency = 1, Text = cfg.Name, TextColor3 = theme.text, Font = Enum.Font.Gotham, TextSize = 14, TextXAlignment = Enum.TextXAlignment.Left, Parent = holder})
            
            local preview = create("Frame", {Size = UDim2.new(0, 30, 0, 20), Position = UDim2.new(1, -45, 0.5, -10), BackgroundColor3 = color, Parent = holder})
            create("UICorner", {CornerRadius = UDim.new(0, 4), Parent = preview})

            local pickerOpen = false
            local pickerFrame = create("Frame", {Size = UDim2.new(1, 0, 0, 40), BackgroundColor3 = theme.element, Visible = false, ClipsDescendants = true, Parent = page})
            create("UICorner", {CornerRadius = UDim.new(0, 8), Parent = pickerFrame})
            create("UIStroke", {Color = theme.stroke, Parent = pickerFrame})

            local hueBar = create("Frame", {Size = UDim2.new(1, -30, 0, 10), Position = UDim2.new(0, 15, 0.5, -5), Parent = pickerFrame})
            local gradient = create("UIGradient", {Color = ColorSequence.new({ColorSequenceKeypoint.new(0, Color3.fromHSV(0, 1, 1)), ColorSequenceKeypoint.new(0.16, Color3.fromHSV(0.16, 1, 1)), ColorSequenceKeypoint.new(0.33, Color3.fromHSV(0.33, 1, 1)), ColorSequenceKeypoint.new(0.5, Color3.fromHSV(0.5, 1, 1)), ColorSequenceKeypoint.new(0.66, Color3.fromHSV(0.66, 1, 1)), ColorSequenceKeypoint.new(0.83, Color3.fromHSV(0.83, 1, 1)), ColorSequenceKeypoint.new(1, Color3.fromHSV(1, 1, 1))}), Parent = hueBar})
            create("UICorner", {CornerRadius = UDim.new(1, 0), Parent = hueBar})

            local function updateHue(input)
                local pos = math.clamp((input.Position.X - hueBar.AbsolutePosition.X) / hueBar.AbsoluteSize.X, 0, 1)
                color = Color3.fromHSV(pos, 1, 1)
                preview.BackgroundColor3 = color
                if cfg.Callback then cfg.Callback(color) end
            end

            local sliding = false
            hueBar.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then sliding = true updateHue(input) end end)
            UIS.InputChanged:Connect(function(input) if sliding and input.UserInputType == Enum.UserInputType.MouseMovement then updateHue(input) end end)
            UIS.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then sliding = false end end)

            holder.MouseButton1Click:Connect(function()
                pickerOpen = not pickerOpen
                pickerFrame.Visible = pickerOpen
            end)
        end

        return Tab
    end

    return Window
end

return Library
