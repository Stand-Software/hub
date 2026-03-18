local Library = {}

local UIS = game:GetService("UserInputService")
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Tema Refinado (Vibe Dark/Modern)
local theme = {
    bg = Color3.fromRGB(10, 10, 12),
    sidebar = Color3.fromRGB(15, 15, 18),
    element = Color3.fromRGB(20, 20, 24),
    elementHover = Color3.fromRGB(28, 28, 32),
    accent = Color3.fromRGB(255, 0, 100), -- Um rosa mais vibrante
    accentGradient = Color3.fromRGB(200, 0, 150),
    text = Color3.fromRGB(255, 255, 255),
    subtext = Color3.fromRGB(160, 160, 165),
    stroke = Color3.fromRGB(35, 35, 40),
    accentStroke = Color3.fromRGB(255, 0, 100)
}

local function create(class, props)
    local obj = Instance.new(class)
    for i, v in pairs(props) do obj[i] = v end
    return obj
end

local function tween(obj, info, props)
    local t = TweenService:Create(obj, TweenInfo.new(info, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), props)
    t:Play()
    return t
end

function Library:CreateWindow(cfg)
    local Window = {CurrentTab = nil}
    
    local targetParent = playerGui
    pcall(function() targetParent = game:GetService("CoreGui") end)

    local gui = create("ScreenGui", {
        Name = "XanaxHub_V3",
        ResetOnSpawn = false,
        Parent = targetParent,
        DisplayOrder = 2147483647
    })

    -- Sombra suave externa
    local mainShadow = create("Frame", {
        Name = "Shadow",
        Size = UDim2.new(0, 750, 0, 500),
        Position = UDim2.new(0.5, -375, 0.5, -250),
        BackgroundColor3 = Color3.new(0,0,0),
        BackgroundTransparency = 0.5,
        Parent = gui
    })
    create("UICorner", {CornerRadius = UDim.new(0, 15), Parent = mainShadow})

    local main = create("Frame", {
        Name = "Main",
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundColor3 = theme.bg,
        BorderSizePixel = 0,
        Parent = mainShadow
    })
    create("UICorner", {CornerRadius = UDim.new(0, 12), Parent = main})
    create("UIStroke", {Color = theme.stroke, Thickness = 1.2, Parent = main})

    -- Alternar Visibilidade
    UIS.InputBegan:Connect(function(input)
        if input.KeyCode == Enum.KeyCode.RightShift then
            mainShadow.Visible = not mainShadow.Visible
        end
    end)

    -- Header (Draggable)
    local header = create("Frame", {
        Size = UDim2.new(1, 0, 0, 50),
        BackgroundTransparency = 1,
        Parent = main
    })

    local title = create("TextLabel", {
        Size = UDim2.new(1, -40, 1, 0),
        Position = UDim2.new(0, 20, 0, 0),
        BackgroundTransparency = 1,
        Text = cfg.Title or "XANAX HUB V3",
        TextColor3 = theme.text,
        Font = Enum.Font.GothamBold,
        TextSize = 16,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = header
    })

    -- Barra Lateral
    local sidebar = create("Frame", {
        Size = UDim2.new(0, 180, 1, -50),
        Position = UDim2.new(0, 0, 0, 50),
        BackgroundColor3 = theme.sidebar,
        BackgroundTransparency = 0.2,
        BorderSizePixel = 0,
        Parent = main
    })
    create("UICorner", {CornerRadius = UDim.new(0, 12), Parent = sidebar})
    
    -- Corretor de borda da sidebar (para não arredondar o lado direito)
    local sidebarFix = create("Frame", {
        Size = UDim2.new(0, 20, 1, 0),
        Position = UDim2.new(1, -20, 0, 0),
        BackgroundColor3 = theme.sidebar,
        BackgroundTransparency = 0.2,
        BorderSizePixel = 0,
        Parent = sidebar
    })

    local tabContainer = create("ScrollingFrame", {
        Size = UDim2.new(1, 0, 1, -20),
        Position = UDim2.new(0, 0, 0, 10),
        BackgroundTransparency = 1,
        ScrollBarThickness = 0,
        Parent = sidebar
    })
    create("UIListLayout", {Parent = tabContainer, Padding = UDim.new(0, 2), HorizontalAlignment = Enum.HorizontalAlignment.Center})

    local content = create("Frame", {
        Size = UDim2.new(1, -180, 1, -50),
        Position = UDim2.new(0, 180, 0, 50),
        BackgroundTransparency = 1,
        Parent = main
    })

    -- Sistema de Drag (Arrastar)
    local dragging, dragStart, startPos
    header.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = mainShadow.Position
        end
    end)
    UIS.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = input.Position - dragStart
            mainShadow.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
    UIS.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
    end)

    local pages = {}

    function Window:CreateTab(name)
        local Tab = {}
        
        local tabBtn = create("TextButton", {
            Size = UDim2.new(0, 160, 0, 38),
            BackgroundTransparency = 1,
            Text = "",
            AutoButtonColor = false,
            Parent = tabContainer
        })
        
        local tabLabel = create("TextLabel", {
            Size = UDim2.new(1, -15, 1, 0),
            Position = UDim2.new(0, 15, 0, 0),
            BackgroundTransparency = 1,
            Text = name,
            TextColor3 = theme.subtext,
            Font = Enum.Font.GothamMedium,
            TextSize = 13,
            TextXAlignment = Enum.TextXAlignment.Left,
            ZIndex = 2,
            Parent = tabBtn
        })

        local indicator = create("Frame", {
            Size = UDim2.new(0, 0, 0.8, 0),
            Position = UDim2.new(0, 5, 0.1, 0),
            BackgroundColor3 = theme.accent,
            BackgroundTransparency = 1,
            ZIndex = 1,
            Parent = tabBtn
        })
        create("UICorner", {CornerRadius = UDim.new(0, 6), Parent = indicator})
        create("UIGradient", {
            Color = ColorSequence.new(theme.accent, theme.accentGradient),
            Rotation = 45,
            Parent = indicator
        })

        local page = create("ScrollingFrame", {
            Size = UDim2.new(1, 0, 1, 0),
            BackgroundTransparency = 1,
            ScrollBarThickness = 2,
            ScrollBarImageColor3 = theme.accent,
            Visible = false,
            Parent = content
        })
        create("UIPadding", {PaddingLeft = UDim.new(0, 20), PaddingRight = UDim.new(0, 20), PaddingTop = UDim.new(0, 20), PaddingBottom = UDim.new(0, 20), Parent = page})
        local layout = create("UIListLayout", {Parent = page, Padding = UDim.new(0, 8), SortOrder = Enum.SortOrder.LayoutOrder})
        
        layout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() 
            page.CanvasSize = UDim2.new(0, 0, 0, layout.AbsoluteContentSize.Y + 20) 
        end)

        local function selectTab()
            for _, v in pairs(pages) do
                v.pg.Visible = false
                tween(v.label, 0.2, {TextColor3 = theme.subtext})
                tween(v.ind, 0.2, {Size = UDim2.new(0, 0, 0.8, 0), BackgroundTransparency = 1})
            end
            page.Visible = true
            tween(tabLabel, 0.2, {TextColor3 = theme.text})
            tween(indicator, 0.2, {Size = UDim2.new(1, -10, 0.8, 0), BackgroundTransparency = 0})
        end

        tabBtn.MouseButton1Click:Connect(selectTab)
        table.insert(pages, {label = tabLabel, pg = page, ind = indicator})
        if #pages == 1 then selectTab() end

        -- Componentes
        function Tab:CreateButton(cfg)
            local btnHolder = create("TextButton", {
                Size = UDim2.new(1, 0, 0, 40),
                BackgroundColor3 = theme.element,
                AutoButtonColor = false,
                Text = "",
                Parent = page
            })
            create("UICorner", {CornerRadius = UDim.new(0, 6), Parent = btnHolder})
            local stroke = create("UIStroke", {Color = theme.stroke, Parent = btnHolder})
            
            local label = create("TextLabel", {
                Size = UDim2.new(1, 0, 1, 0),
                BackgroundTransparency = 1,
                Text = cfg.Name,
                TextColor3 = theme.text,
                Font = Enum.Font.GothamMedium,
                TextSize = 13,
                Parent = btnHolder
            })

            btnHolder.MouseEnter:Connect(function() tween(btnHolder, 0.2, {BackgroundColor3 = theme.elementHover}) end)
            btnHolder.MouseLeave:Connect(function() tween(btnHolder, 0.2, {BackgroundColor3 = theme.element}) end)
            
            btnHolder.MouseButton1Click:Connect(function()
                local circle = create("Frame", {
                    Size = UDim2.new(0,0,0,0),
                    Position = UDim2.new(0.5,0,0.5,0),
                    BackgroundColor3 = theme.accent,
                    BackgroundTransparency = 0.5,
                    Parent = btnHolder,
                    ZIndex = 3
                })
                create("UICorner", {CornerRadius = UDim.new(1,0), Parent = circle})
                tween(circle, 0.4, {Size = UDim2.new(1,0,2,0), BackgroundTransparency = 1})
                task.delay(0.4, function() circle:Destroy() end)
                if cfg.Callback then cfg.Callback() end
            end)
        end

        function Tab:CreateToggle(cfg)
            local state = cfg.Default or false
            local toggle = create("TextButton", {
                Size = UDim2.new(1, 0, 0, 45),
                BackgroundColor3 = theme.element,
                AutoButtonColor = false,
                Text = "",
                Parent = page
            })
            create("UICorner", {CornerRadius = UDim.new(0, 6), Parent = toggle})
            create("UIStroke", {Color = theme.stroke, Parent = toggle})

            create("TextLabel", {
                Size = UDim2.new(1, -60, 1, 0),
                Position = UDim2.new(0, 15, 0, 0),
                BackgroundTransparency = 1,
                Text = cfg.Name,
                TextColor3 = theme.text,
                Font = Enum.Font.Gotham,
                TextSize = 13,
                TextXAlignment = Enum.TextXAlignment.Left,
                Parent = toggle
            })

            local box = create("Frame", {
                Size = UDim2.new(0, 36, 0, 18),
                Position = UDim2.new(1, -50, 0.5, -9),
                BackgroundColor3 = state and theme.accent or theme.bg,
                Parent = toggle
            })
            create("UICorner", {CornerRadius = UDim.new(1, 0), Parent = box})
            
            local dot = create("Frame", {
                Size = UDim2.new(0, 12, 0, 12),
                Position = state and UDim2.new(1, -15, 0.5, -6) or UDim2.new(0, 3, 0.5, -6),
                BackgroundColor3 = Color3.new(1,1,1),
                Parent = box
            })
            create("UICorner", {CornerRadius = UDim.new(1, 0), Parent = dot})

            toggle.MouseButton1Click:Connect(function()
                state = not state
                tween(box, 0.2, {BackgroundColor3 = state and theme.accent or theme.bg})
                tween(dot, 0.2, {Position = state and UDim2.new(1, -15, 0.5, -6) or UDim2.new(0, 3, 0.5, -6)})
                if cfg.Callback then cfg.Callback(state) end
            end)
        end

        function Tab:CreateSlider(cfg)
            local min, max = cfg.Min or 0, cfg.Max or 100
            local val = cfg.Default or min
            
            local holder = create("Frame", {
                Size = UDim2.new(1, 0, 0, 60),
                BackgroundColor3 = theme.element,
                Parent = page
            })
            create("UICorner", {CornerRadius = UDim.new(0, 6), Parent = holder})
            create("UIStroke", {Color = theme.stroke, Parent = holder})

            create("TextLabel", {
                Size = UDim2.new(1, -30, 0, 30),
                Position = UDim2.new(0, 15, 0, 5),
                BackgroundTransparency = 1,
                Text = cfg.Name,
                TextColor3 = theme.text,
                Font = Enum.Font.Gotham,
                TextSize = 13,
                TextXAlignment = Enum.TextXAlignment.Left,
                Parent = holder
            })

            local valLabel = create("TextLabel", {
                Size = UDim2.new(0, 50, 0, 30),
                Position = UDim2.new(1, -65, 0, 5),
                BackgroundTransparency = 1,
                Text = tostring(val),
                TextColor3 = theme.accent,
                Font = Enum.Font.GothamBold,
                TextSize = 12,
                TextXAlignment = Enum.TextXAlignment.Right,
                Parent = holder
            })

            local barBg = create("Frame", {
                Size = UDim2.new(1, -30, 0, 4),
                Position = UDim2.new(0, 15, 0, 45),
                BackgroundColor3 = theme.bg,
                Parent = holder
            })
            create("UICorner", {CornerRadius = UDim.new(1, 0), Parent = barBg})

            local barFill = create("Frame", {
                Size = UDim2.new((val - min) / (max - min), 0, 1, 0),
                BackgroundColor3 = theme.accent,
                Parent = barBg
            })
            create("UICorner", {CornerRadius = UDim.new(1, 0), Parent = barFill})

            local function update(input)
                local pos = math.clamp((input.Position.X - barBg.AbsolutePosition.X) / barBg.AbsoluteSize.X, 0, 1)
                val = math.floor(min + (max - min) * pos)
                valLabel.Text = tostring(val)
                tween(barFill, 0.1, {Size = UDim2.new(pos, 0, 1, 0)})
                if cfg.Callback then cfg.Callback(val) end
            end

            local sliding = false
            holder.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then sliding = true update(input) end end)
            UIS.InputChanged:Connect(function(input) if sliding and input.UserInputType == Enum.UserInputType.MouseMovement then update(input) end end)
            UIS.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then sliding = false end end)
        end

        return Tab
    end

    return Window
end

return Library
