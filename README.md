local Library = {}

-- Services
local UIS = game:GetService("UserInputService")
local Players = game:GetService("Players")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Theme
local theme = {
    bg = Color3.fromRGB(18,18,22),
    light = Color3.fromRGB(28,28,32),
    accent = Color3.fromRGB(255,0,80),
    text = Color3.fromRGB(255,255,255)
}

-- Helper
local function create(class, props)
    local obj = Instance.new(class)
    for i,v in pairs(props) do
        obj[i] = v
    end
    return obj
end

-- Window
function Library:CreateWindow(cfg)
    local Window = {}

    local gui = create("ScreenGui", {
        Name = "XanaxHub",
        ResetOnSpawn = false,
        Parent = playerGui
    })

    local main = create("Frame", {
        Size = UDim2.new(0, 420, 0, 300),
        Position = UDim2.new(0.5, -210, 0.5, -150),
        BackgroundColor3 = theme.bg,
        Parent = gui
    })

    create("UICorner", {CornerRadius = UDim.new(0,10), Parent = main})
    create("UIStroke", {Color = theme.accent, Thickness = 2, Parent = main})

    local title = create("TextLabel", {
        Size = UDim2.new(1,0,0,40),
        BackgroundTransparency = 1,
        Text = cfg.Title or "Window",
        TextColor3 = theme.text,
        Font = Enum.Font.GothamBold,
        TextSize = 18,
        Parent = main
    })

    local tabBar = create("Frame", {
        Size = UDim2.new(0,120,1,-40),
        Position = UDim2.new(0,0,0,40),
        BackgroundColor3 = theme.light,
        Parent = main
    })

    local content = create("Frame", {
        Size = UDim2.new(1,-120,1,-40),
        Position = UDim2.new(0,120,0,40),
        BackgroundTransparency = 1,
        Parent = main
    })

    create("UIListLayout", {
        Parent = tabBar,
        SortOrder = Enum.SortOrder.LayoutOrder
    })

    local pages = {}

    -- Toggle UI
    UIS.InputBegan:Connect(function(input, gp)
        if gp then return end
        if input.KeyCode == Enum.KeyCode.RightShift then
            main.Visible = not main.Visible
        end
    end)

    -- Drag
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

    -- Tabs
    function Window:CreateTab(name)
        local Tab = {}

        local btn = create("TextButton", {
            Size = UDim2.new(1,0,0,40),
            Text = name,
            BackgroundColor3 = theme.light,
            TextColor3 = theme.text,
            Font = Enum.Font.Gotham,
            TextSize = 14,
            Parent = tabBar
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
            Padding = UDim.new(0,6)
        })

        table.insert(pages, page)

        btn.MouseButton1Click:Connect(function()
            for _,p in pairs(pages) do
                p.Visible = false
            end
            page.Visible = true
        end)

        if #pages == 1 then
            page.Visible = true
        end

        function Tab:CreateToggle(cfg)
            local state = cfg.Default or false

            local toggle = create("TextButton", {
                Size = UDim2.new(1,-10,0,40),
                Text = cfg.Name .. " : " .. (state and "ON" or "OFF"),
                BackgroundColor3 = theme.light,
                TextColor3 = theme.text,
                Parent = page
            })

            create("UICorner", {CornerRadius = UDim.new(0,8), Parent = toggle})

            toggle.MouseButton1Click:Connect(function()
                state = not state
                toggle.Text = cfg.Name .. " : " .. (state and "ON" or "OFF")

                if cfg.Callback then
                    cfg.Callback(state)
                end
            end)
        end

        return Tab
    end

    return Window
end

return Library
