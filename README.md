local Library = {}

local UIS = game:GetService("UserInputService")
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local connections = {}

-- Tema Moderno (Dark Premium)
local theme = {
    bg = Color3.fromRGB(15, 15, 18),
    sidebar = Color3.fromRGB(20, 20, 25),
    element = Color3.fromRGB(25, 25, 30),
    accent = Color3.fromRGB(255, 0, 85),
    text = Color3.fromRGB(255, 255, 255),
    subtext = Color3.fromRGB(160, 160, 165),
    stroke = Color3.fromRGB(40, 40, 45)
}

local function create(class, props)
    local obj = Instance.new(class)
    for i, v in pairs(props) do
        obj[i] = v
    end
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

    -- Sombra suave (opcional, simulada com stroke grosso)
    local main = create("Frame", {
        Name = "Main",
        Size = UDim2.new(0, 650, 0, 420), -- Tamanho maior e mais elegante
        Position = UDim2.new(0.5, -325, 0.5, -210),
        BackgroundColor3 = theme.bg,
        BorderSizePixel = 0,
        Parent = gui
    })

    create("UICorner", {CornerRadius = UDim.new(0, 10), Parent = main})
    
    local mainStroke = create("UIStroke", {
        Color = theme.stroke,
        Thickness = 1.2,
        Parent = main
    })

    -- Top Bar (Header)
    local header = create("Frame", {
        Size = UDim2.new(1, 0, 0, 50),
        BackgroundTransparency = 1,
        Parent = main
    })

    local title = create("TextLabel", {
        Size = UDim2.new(1, -40, 1, 0),
        Position = UDim2.new(0, 20, 0, 0),
        BackgroundTransparency = 1,
        Text = cfg.Title or "XANAX HUB",
        TextColor3 = theme.text,
        Font = Enum.Font.GothamBold,
        TextSize = 18,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = header
    })

    -- Divisor
    create("Frame", {
        Size = UDim2.new(1, 0, 0, 1),
        Position = UDim2.new(0, 0, 1, 0),
        BackgroundColor3 = theme.stroke,
        BorderSizePixel = 0,
        Parent = header
    })

    -- Estrutura Lateral
    local sidebar = create("Frame", {
        Size = UDim2.new(0, 170, 1, -51),
        Position = UDim2.new(0, 0, 0, 51),
        BackgroundColor3 = theme.sidebar,
        BorderSizePixel = 0,
        Parent = main
    })
    
    create("UICorner", {CornerRadius = UDim.new(0, 10), Parent = sidebar})
    -- Esconder cantos direitos da sidebar para encaixar no main
    create("Frame", {
        Size = UDim2.new(0, 20, 1, 0),
        Position = UDim2.new(1, -20, 0, 0),
        BackgroundColor3 = theme.sidebar,
        BorderSizePixel = 0,
        Parent = sidebar
    })

    local tabContainer = create("ScrollingFrame", {
        Size = UDim2.new(1, 0, 1, -10),
        Position = UDim2.new(0, 0, 0, 5),
        BackgroundTransparency = 1,
        ScrollBarThickness = 0,
        CanvasSize = UDim2.new(0, 0, 0, 0),
        Parent = sidebar
    })

    create("UIListLayout", {
        Parent = tabContainer,
        Padding = UDim.new(0, 2),
        HorizontalAlignment = Enum.HorizontalAlignment.Center
    })

    local content = create("Frame", {
        Size = UDim2.new(1, -170, 1, -51),
        Position = UDim2.new(0, 170, 0, 51),
        BackgroundTransparency = 1,
        Parent = main
    })

    local pages = {}
    local currentTab = nil

    -- Drag System
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
            main.Position = UDim2.new(
                startPos.X.Scale,
                startPos.X.Offset + delta.X,
                startPos.Y.Scale,
                startPos.Y.Offset + delta.Y
            )
        end
    end)

    UIS.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)

    -- Toggle Key (K para abrir/fechar, RShift para destruir)
    table.insert(connections, UIS.InputBegan:Connect(function(input, gp)
        if gp then return end
        if input.KeyCode == Enum.KeyCode.K then
            main.Visible = not main.Visible
        elseif input.KeyCode == Enum.KeyCode.RightShift then
            gui:Destroy()
            for _, c in pairs(connections) do pcall(function() c:Disconnect() end) end
        end
    end))

    function Window:CreateTab(name)
        local Tab = {}

        local tabBtn = create("TextButton", {
            Size = UDim2.new(0, 150, 0, 38),
            BackgroundTransparency = 1,
            Text = name,
            TextColor3 = theme.subtext,
            Font = Enum.Font.GothamMedium,
            TextSize = 14,
            AutoButtonColor = false,
            Parent = tabContainer
        })
        
        local tabIndicator = create("Frame", {
            Size = UDim2.new(0, 3, 0, 18),
            Position = UDim2.new(0, 0, 0.5, -9),
            BackgroundColor3 = theme.accent,
            BackgroundTransparency = 1,
            BorderSizePixel = 0,
            Parent = tabBtn
        })
        create("UICorner", {CornerRadius = UDim.new(1, 0), Parent = tabIndicator})

        local page = create("ScrollingFrame", {
            Size = UDim2.new(1, 0, 1, 0),
            BackgroundTransparency = 1,
            ScrollBarThickness = 2,
            ScrollBarImageColor3 = theme.accent,
            Visible = false,
            CanvasSize = UDim2.new(0, 0, 0, 0),
            Parent = content
        })

        create("UIPadding", {
            PaddingLeft = UDim.new(0, 20),
            PaddingRight = UDim.new(0, 20),
            PaddingTop = UDim.new(0, 20),
            PaddingBottom = UDim.new(0, 20),
            Parent = page
        })

        local pageLayout = create("UIListLayout", {
            Parent = page,
            Padding = UDim.new(0, 10),
            SortOrder = Enum.SortOrder.LayoutOrder
        })

        pageLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
            page.CanvasSize = UDim2.new(0, 0, 0, pageLayout.AbsoluteContentSize.Y + 40)
        end)

        table.insert(pages, {btn = tabBtn, pg = page, ind = tabIndicator})

        local function selectTab()
            for _, v in pairs(pages) do
                v.pg.Visible = false
                tween(v.btn, 0.3, {TextColor3 = theme.subtext})
                tween(v.ind, 0.3, {BackgroundTransparency = 1})
            end
            page.Visible = true
            tween(tabBtn, 0.3, {TextColor3 = theme.text})
            tween(tabIndicator, 0.3, {BackgroundTransparency = 0})
        end

        tabBtn.MouseButton1Click:Connect(selectTab)
        if #pages == 1 then selectTab() end

        -- Elementos da Aba
        function Tab:CreateToggle(cfg)
            local state = cfg.Default or false
            
            local holder = create("TextButton", {
                Name = "Toggle",
                Size = UDim2.new(1, 0, 0, 45),
                BackgroundColor3 = theme.element,
                AutoButtonColor = false,
                Text = "",
                Parent = page
            })
            create("UICorner", {CornerRadius = UDim.new(0, 8), Parent = holder})
            create("UIStroke", {Color = theme.stroke, Thickness = 1, Parent = holder})

            local label = create("TextLabel", {
                Size = UDim2.new(1, -60, 1, 0),
                Position = UDim2.new(0, 15, 0, 0),
                BackgroundTransparency = 1,
                Text = cfg.Name,
                TextColor3 = theme.text,
                Font = Enum.Font.Gotham,
                TextSize = 14,
                TextXAlignment = Enum.TextXAlignment.Left,
                Parent = holder
            })

            local bg = create("Frame", {
                Size = UDim2.new(0, 42, 0, 22),
                Position = UDim2.new(1, -57, 0.5, -11),
                BackgroundColor3 = state and theme.accent or Color3.fromRGB(45, 45, 50),
                Parent = holder
            })
            create("UICorner", {CornerRadius = UDim.new(1, 0), Parent = bg})

            local circle = create("Frame", {
                Size = UDim2.new(0, 16, 0, 16),
                Position = state and UDim2.new(1, -20, 0.5, -8) or UDim2.new(0, 4, 0.5, -8),
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
                Parent = bg
            })
            create("UICorner", {CornerRadius = UDim.new(1, 0), Parent = circle})

            holder.MouseButton1Click:Connect(function()
                state = not state
                tween(bg, 0.3, {BackgroundColor3 = state and theme.accent or Color3.fromRGB(45, 45, 50)})
                tween(circle, 0.3, {Position = state and UDim2.new(1, -20, 0.5, -8) or UDim2.new(0, 4, 0.5, -8)})
                if cfg.Callback then cfg.Callback(state) end
            end)
        end

        function Tab:CreateButton(cfg)
            local btn = create("TextButton", {
                Size = UDim2.new(1, 0, 0, 45),
                BackgroundColor3 = theme.element,
                AutoButtonColor = false,
                Text = cfg.Name,
                TextColor3 = theme.text,
                Font = Enum.Font.GothamMedium,
                TextSize = 14,
                Parent = page
            })
            create("UICorner", {CornerRadius = UDim.new(0, 8), Parent = btn})
            create("UIStroke", {Color = theme.stroke, Thickness = 1, Parent = btn})

            btn.MouseEnter:Connect(function() tween(btn, 0.2, {BackgroundColor3 = theme.stroke}) end)
            btn.MouseLeave:Connect(function() tween(btn, 0.2, {BackgroundColor3 = theme.element}) end)
            
            btn.MouseButton1Click:Connect(function()
                -- Efeito de clique rápido
                local oldColor = btn.BackgroundColor3
                btn.BackgroundColor3 = theme.accent
                wait(0.1)
                tween(btn, 0.2, {BackgroundColor3 = oldColor})
                if cfg.Callback then cfg.Callback() end
            end)
        end

        return Tab
    end

    return Window
end

return Library
