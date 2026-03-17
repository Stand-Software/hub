local Library = {}

local UIS = game:GetService("UserInputService")
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local connections = {}

local theme = {
    bg = Color3.fromRGB(12, 12, 14),
    sidebar = Color3.fromRGB(18, 18, 22),
    element = Color3.fromRGB(22, 22, 26),
    accent = Color3.fromRGB(255, 0, 85),
    text = Color3.fromRGB(255, 255, 255),
    subtext = Color3.fromRGB(150, 150, 155),
    stroke = Color3.fromRGB(35, 35, 40)
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
    
    local gui = create("ScreenGui", {
        Name = "XanaxHub_V2",
        ResetOnSpawn = false,
        Parent = playerGui,
        IgnoreGuiInset = true
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

    create("Frame", {
        Size = UDim2.new(1, 0, 0, 1),
        Position = UDim2.new(0, 0, 1, 0),
        BackgroundColor3 = theme.stroke,
        BorderSizePixel = 0,
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
    create("Frame", {Size = UDim2.new(0, 20, 1, 0), Position = UDim2.new(1, -20, 0, 0), BackgroundColor3 = theme.sidebar, BorderSizePixel = 0, Parent = sidebar})

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
    
    -- Dragging
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

    -- Atalhos: RightShift (Menu) / Delete (Kill App)
    table.insert(connections, UIS.InputBegan:Connect(function(input, gp)
        if gp then return end
        if input.KeyCode == Enum.KeyCode.RightShift then
            main.Visible = not main.Visible
        elseif input.KeyCode == Enum.KeyCode.Delete then
            gui:Destroy()
            for _, c in pairs(connections) do pcall(function() c:Disconnect() end) end
        end
    end))

    function Window:CreateTab(name)
        local Tab = {}
        local tabBtn = create("TextButton", {Size = UDim2.new(0, 170, 0, 42), BackgroundTransparency = 1, Text = name, TextColor3 = theme.subtext, Font = Enum.Font.GothamMedium, TextSize = 14, AutoButtonColor = false, Parent = tabContainer})
        local indicator = create("Frame", {Size = UDim2.new(0, 3, 0, 20), Position = UDim2.new(0, 0, 0.5, -10), BackgroundColor3 = theme.accent, BackgroundTransparency = 1, Parent = tabBtn})
        local page = create("ScrollingFrame", {Size = UDim2.new(1, 0, 1, 0), BackgroundTransparency = 1, ScrollBarThickness = 2, Visible = false, Parent = content})
        create("UIPadding", {PaddingLeft = UDim.new(0, 25), PaddingRight = UDim.new(0, 25), PaddingTop = UDim.new(0, 25), PaddingBottom = UDim.new(0, 25), Parent = page})
        local layout = create("UIListLayout", {Parent = page, Padding = UDim.new(0, 12), SortOrder = Enum.SortOrder.LayoutOrder})
        layout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() page.CanvasSize = UDim2.new(0, 0, 0, layout.AbsoluteContentSize.Y + 20) end)

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
            local checkmark = create("TextLabel", {Size = UDim2.new(1, 0, 1, 0), BackgroundTransparency = 1, Text = "✓", TextColor3 = Color3.new(1,1,1), TextSize = 14, Visible = state, Parent = box})

            holder.MouseButton1Click:Connect(function()
                state = not state
                tween(box, 0.2, {BackgroundColor3 = state and theme.accent or theme.bg})
                checkmark.Visible = state
                if cfg.Callback then cfg.Callback(state) end
            end)
        end

        function Tab:CreateButton(cfg)
            local btn = create("TextButton", {Size = UDim2.new(1, 0, 0, 50), BackgroundColor3 = theme.element, AutoButtonColor = false, Text = cfg.Name, TextColor3 = theme.text, Font = Enum.Font.GothamMedium, TextSize = 14, Parent = page})
            create("UICorner", {CornerRadius = UDim.new(0, 8), Parent = btn})
            create("UIStroke", {Color = theme.stroke, Parent = btn})
            btn.MouseButton1Click:Connect(function()
                local old = btn.BackgroundColor3
                btn.BackgroundColor3 = theme.accent
                wait(0.1)
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

        function Tab:CreateKeybind(cfg)
            local boundKey = cfg.Default or Enum.KeyCode.F
            local listening = false
            local holder = create("Frame", {Size = UDim2.new(1, 0, 0, 50), BackgroundColor3 = theme.element, Parent = page})
            create("UICorner", {CornerRadius = UDim.new(0, 8), Parent = holder})
            create("UIStroke", {Color = theme.stroke, Parent = holder})
            create("TextLabel", {Size = UDim2.new(1, -100, 1, 0), Position = UDim2.new(0, 15, 0, 0), BackgroundTransparency = 1, Text = cfg.Name, TextColor3 = theme.text, Font = Enum.Font.Gotham, TextSize = 14, TextXAlignment = Enum.TextXAlignment.Left, Parent = holder})
            local bindBtn = create("TextButton", {Size = UDim2.new(0, 80, 0, 30), Position = UDim2.new(1, -95, 0.5, -15), BackgroundColor3 = theme.bg, Text = boundKey.Name, TextColor3 = theme.accent, Font = Enum.Font.GothamMedium, TextSize = 12, Parent = holder})
            create("UICorner", {CornerRadius = UDim.new(0, 6), Parent = bindBtn})

            bindBtn.MouseButton1Click:Connect(function() listening = true bindBtn.Text = "..." end)
            UIS.InputBegan:Connect(function(input)
                if listening and input.UserInputType == Enum.UserInputType.Keyboard then
                    boundKey = input.KeyCode
                    bindBtn.Text = boundKey.Name
                    listening = false
                elseif not listening and input.KeyCode == boundKey then
                    if cfg.Callback then cfg.Callback() end
                end
            end)
        end

        return Tab
    end

    return Window
end

return Library
