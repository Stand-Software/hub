local Library = {}

local UIS = game:GetService("UserInputService")
local Players = game:GetService("Players")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local connections = {}

local theme = {
    bg = Color3.fromRGB(20,20,25),
    sidebar = Color3.fromRGB(25,25,30),
    element = Color3.fromRGB(30,30,35),
    accent = Color3.fromRGB(255,0,80),
    text = Color3.fromRGB(255,255,255),
    subtext = Color3.fromRGB(170,170,170)
}

local function create(class, props)
    local obj = Instance.new(class)
    for i,v in pairs(props) do
        obj[i] = v
    end
    return obj
end

function Library:CreateWindow(cfg)
    local Window = {}

    local gui = create("ScreenGui", {
        Name = "XanaxHub",
        ResetOnSpawn = false,
        Parent = playerGui
    })

    local main = create("Frame", {
        Size = UDim2.new(0, 500, 0, 320),
        Position = UDim2.new(0.5, -250, 0.5, -160),
        BackgroundColor3 = theme.bg,
        Parent = gui
    })

    create("UICorner", {CornerRadius = UDim.new(0,12), Parent = main})
    create("UIStroke", {Color = theme.accent, Thickness = 1.5, Parent = main})

    local title = create("TextLabel", {
        Size = UDim2.new(1,0,0,45),
        BackgroundTransparency = 1,
        Text = cfg.Title or "Xanax Hub",
        TextColor3 = theme.text,
        Font = Enum.Font.GothamBold,
        TextSize = 20,
        Parent = main
    })

    local sidebar = create("Frame", {
        Size = UDim2.new(0,140,1,-45),
        Position = UDim2.new(0,0,0,45),
        BackgroundColor3 = theme.sidebar,
        Parent = main
    })

    local content = create("Frame", {
        Size = UDim2.new(1,-140,1,-45),
        Position = UDim2.new(0,140,0,45),
        BackgroundTransparency = 1,
        Parent = main
    })

    create("UIListLayout", {
        Parent = sidebar,
        Padding = UDim.new(0,5)
    })

    local pages = {}
    local currentTab = nil

    -- TOGGLE UI (SHIFT)
    table.insert(connections, UIS.InputBegan:Connect(function(input, gp)
        if gp then return end

        if input.KeyCode == Enum.KeyCode.RightShift then
            main.Visible = not main.Visible
        end

        if input.KeyCode == Enum.KeyCode.Delete then
            gui:Destroy()
            for _,c in pairs(connections) do
                pcall(function() c:Disconnect() end)
            end
        end
    end))

    -- DRAG
    local dragging, dragStart, startPos

    main.InputBegan:Connect(function(input)
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

    function Window:CreateTab(name)
        local Tab = {}

        local tabBtn = create("TextButton", {
            Size = UDim2.new(1,0,0,40),
            Text = name,
            BackgroundColor3 = theme.sidebar,
            TextColor3 = theme.subtext,
            Font = Enum.Font.Gotham,
            TextSize = 14,
            Parent = sidebar
        })

        local page = create("ScrollingFrame", {
            Size = UDim2.new(1,0,1,0),
            CanvasSize = UDim2.new(0,0,0,0),
            ScrollBarThickness = 4,
            Visible = false,
            BackgroundTransparency = 1,
            Parent = content
        })

        local layout = create("UIListLayout", {
            Parent = page,
            Padding = UDim.new(0,8)
        })

        table.insert(pages, page)

        local function selectTab()
            for _,p in pairs(pages) do
                p.Visible = false
            end

            page.Visible = true
            currentTab = tabBtn

            for _,b in pairs(sidebar:GetChildren()) do
                if b:IsA("TextButton") then
                    b.TextColor3 = theme.subtext
                end
            end

            tabBtn.TextColor3 = theme.text
        end

        tabBtn.MouseButton1Click:Connect(selectTab)

        if not currentTab then
            selectTab()
        end

        function Tab:CreateToggle(cfg)
            local state = cfg.Default or false

            local holder = create("Frame", {
                Size = UDim2.new(1,-10,0,40),
                BackgroundColor3 = theme.element,
                Parent = page
            })

            create("UICorner", {CornerRadius = UDim.new(0,8), Parent = holder})

            local label = create("TextLabel", {
                Size = UDim2.new(1,-60,1,0),
                Position = UDim2.new(0,10,0,0),
                BackgroundTransparency = 1,
                Text = cfg.Name,
                TextColor3 = theme.text,
                Font = Enum.Font.Gotham,
                TextSize = 14,
                TextXAlignment = Enum.TextXAlignment.Left,
                Parent = holder
            })

            local toggleBtn = create("Frame", {
                Size = UDim2.new(0,40,0,20),
                Position = UDim2.new(1,-50,0.5,-10),
                BackgroundColor3 = state and theme.accent or Color3.fromRGB(60,60,60),
                Parent = holder
            })

            create("UICorner", {CornerRadius = UDim.new(1,0), Parent = toggleBtn})

            local circle = create("Frame", {
                Size = UDim2.new(0,16,0,16),
                Position = state and UDim2.new(1,-18,0.5,-8) or UDim2.new(0,2,0.5,-8),
                BackgroundColor3 = Color3.fromRGB(255,255,255),
                Parent = toggleBtn
            })

            create("UICorner", {CornerRadius = UDim.new(1,0), Parent = circle})

            holder.InputBegan:Connect(function(input)
                if input.UserInputType == Enum.UserInputType.MouseButton1 then
                    state = not state

                    toggleBtn.BackgroundColor3 = state and theme.accent or Color3.fromRGB(60,60,60)
                    circle.Position = state and UDim2.new(1,-18,0.5,-8) or UDim2.new(0,2,0.5,-8)

                    if cfg.Callback then
                        cfg.Callback(state)
                    end
                end
            end)
        end

        return Tab
    end

    return Window
end

return Library
