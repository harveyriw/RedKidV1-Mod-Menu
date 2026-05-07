# RedKidV1-Mod-Menu
This is a mod menu for roblox at least mods for  6 games updateing it as mutch as i can.
Mod menu script (-- RedKidV1 Mod Menu -- RedKid V1 MEGA HUB (GLOBAL OWNER UPDATE)
-- Left sidebar nav, scrollable game list, movable top-left spawn.
-- Only Roblox ID 10902521287 can use owner functions and see the OWNER tag.

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local StarterGui = game:GetService("StarterGui")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")

local OWNER_ID = 10902521287
local player = Players.LocalPlayer or Players.PlayerAdded:Wait()

local function isOwner()
    return player.UserId == OWNER_ID
end

local function getCharacter()
    return player.Character or player.CharacterAdded:Wait()
end

local function getHumanoid()
    local char = getCharacter()
    return char and char:FindFirstChildOfClass("Humanoid")
end

local function getHumanoidRootPart()
    local char = getCharacter()
    return char and (char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("Torso"))
end

local function clearOwnerTag()
    local plr = player
    if not plr or not plr.Character then
        return
    end
    local existing = plr.Character:FindFirstChild("RedKidOwnerTag")
    if existing then
        existing:Destroy()
    end
end

local function makeOwnerTag()
    if not player or not player.Character then
        return
    end
    clearOwnerTag()

    local adorn = player.Character:FindFirstChild("Head") or player.Character:FindFirstChild("UpperTorso")
    if not adorn then
        return
    end

    local billboard = Instance.new("BillboardGui")
    billboard.Name = "RedKidOwnerTag"
    billboard.Adornee = adorn
    billboard.AlwaysOnTop = true
    billboard.Size = UDim2.new(0, 160, 0, 34)
    billboard.StudsOffset = Vector3.new(0, 2.4, 0)
    billboard.Parent = player.Character

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.TextScaled = true
    label.Font = Enum.Font.GothamBlack
    label.Text = "[ OWNER ]"
    label.TextColor3 = Color3.fromRGB(255, 54, 54)
    label.TextStrokeTransparency = 0.6
    label.TextStrokeColor3 = Color3.new(0, 0, 0)
    label.Parent = billboard
end

local function requestOwnerTag()
    if not isOwner() then
        makeNotification("Only owner can show OWNER tag.")
        return
    end
    makeOwnerTag()
end

local function makeNotification(message)
    pcall(function()
        StarterGui:SetCore("SendNotification", {
            Title = "RedKidV1",
            Text = message,
            Duration = 4,
        })
    end)
end

local function safeTeleport(cframe)
    local hrp = getHumanoidRootPart()
    if hrp and cframe then
        hrp.CFrame = cframe
    end
end

local function safeSetSpeed(value)
    local humanoid = getHumanoid()
    if humanoid then
        humanoid.WalkSpeed = value
    end
end

local function safeSetJump(value)
    local humanoid = getHumanoid()
    if humanoid then
        humanoid.JumpPower = value
    end
end

local function safeSetHealth(value)
    local humanoid = getHumanoid()
    if humanoid then
        humanoid.MaxHealth = value
        humanoid.Health = value
    end
end

local function safeSetGravity(value)
    if typeof(value) == "number" then
        Workspace.Gravity = value
    end
end

local function safeFindNearest(name)
    local hrp = getHumanoidRootPart()
    if not hrp then
        return nil
    end
    local best, bestDist = nil, math.huge
    local lowerName = name:lower()
    for _, obj in ipairs(Workspace:GetDescendants()) do
        if obj:IsA("BasePart") and obj.Name:lower():find(lowerName) then
            local dist = (obj.Position - hrp.Position).Magnitude
            if dist < bestDist then
                bestDist = dist
                best = obj
            end
        end
    end
    return best
end

local function toggleNoClip(duration)
    local conn
    conn = RunService.Stepped:Connect(function()
        local char = getCharacter()
        if char then
            for _, part in ipairs(char:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                end
            end
        end
    end)
    delay(duration or 10, function()
        if conn then
            conn:Disconnect()
        end
    end)
end

local function killNearbyPlayers()
    local hrp = getHumanoidRootPart()
    if not hrp then
        return
    end
    for _, other in ipairs(Players:GetPlayers()) do
        if other ~= player and other.Character then
            local humanoid = other.Character:FindFirstChildOfClass("Humanoid")
            local otherHRP = other.Character:FindFirstChild("HumanoidRootPart")
            if humanoid and otherHRP and (otherHRP.Position - hrp.Position).Magnitude <= 75 then
                humanoid.Health = 0
            end
        end
    end
end

local function healAllPlayers()
    for _, other in ipairs(Players:GetPlayers()) do
        if other.Character then
            local humanoid = other.Character:FindFirstChildOfClass("Humanoid")
            if humanoid then
                humanoid.Health = humanoid.MaxHealth
            end
        end
    end
end

local function flingNearbyPlayers()
    local hrp = getHumanoidRootPart()
    if not hrp then
        return
    end
    for _, other in ipairs(Players:GetPlayers()) do
        if other ~= player and other.Character then
            local target = other.Character:FindFirstChild("HumanoidRootPart")
            if target then
                target.Velocity = (target.Position - hrp.Position).Unit * 200
            end
        end
    end
end

local function createGarden()
    local hrp = getHumanoidRootPart()
    if not hrp then
        return
    end
    for i = 1, 12 do
        local part = Instance.new("Part")
        part.Size = Vector3.new(2, 0.5, 2)
        part.Anchored = true
        part.CanCollide = false
        part.Material = Enum.Material.Grass
        part.Color = Color3.fromRGB(60, 180, 70)
        part.CFrame = hrp.CFrame * CFrame.new(math.cos(i * math.pi / 6) * 8, -3, math.sin(i * math.pi / 6) * 8)
        part.Parent = Workspace
    end
    makeNotification("Garden grown.")
end

local function spawnAll99Knight()
    local hrp = getHumanoidRootPart()
    if not hrp then
        makeNotification("No character found.")
        return
    end

    local found = {}
    for _, obj in ipairs(Workspace:GetDescendants()) do
        if obj:IsA("BasePart") and not obj.Anchored then
            local lowerName = obj.Name:lower()
            if lowerName:find("food") or lowerName:find("armor") or lowerName:find("item")
                or lowerName:find("weapon") or lowerName:find("collect") then
                table.insert(found, obj)
            end
        end
    end

    if #found == 0 then
        makeNotification("No 99 Knights items found.")
        return
    end

    for i, obj in ipairs(found) do
        local x = ((i - 1) % 6 - 2.5) * 4
        local z = -math.floor((i - 1) / 6) * 4 - 8
        pcall(function()
            obj.CFrame = hrp.CFrame * CFrame.new(x, 0, z)
        end)
    end

    makeNotification("Spawn All moved 99 Knights items to you.")
end

local function createButton(parent, text, callback)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(1, 0, 0, 48)
    button.BackgroundColor3 = Color3.fromRGB(18, 18, 18)
    button.BorderSizePixel = 0
    button.Font = Enum.Font.GothamSemibold
    button.TextSize = 14
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.Text = text
    button.TextWrapped = true
    button.TextXAlignment = Enum.TextXAlignment.Center
    button.TextYAlignment = Enum.TextYAlignment.Center
    button.AutoButtonColor = true
    button.Parent = parent

    local stroke = Instance.new("UIStroke")
    stroke.Color = Color3.fromRGB(255, 0, 0)
    stroke.Thickness = 2
    stroke.Parent = button

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 12)
    corner.Parent = button

    button.MouseButton1Click:Connect(function()
        pcall(callback)
    end)

    return button
end

local function createTabButton(parent, text, callback)
    local tabButton = Instance.new("TextButton")
    tabButton.Size = UDim2.new(1, -20, 0, 44)
    tabButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    tabButton.BorderSizePixel = 0
    tabButton.Font = Enum.Font.GothamBold
    tabButton.TextSize = 14
    tabButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    tabButton.Text = text
    tabButton.AutoButtonColor = true
    tabButton.Parent = parent

    local stroke = Instance.new("UIStroke")
    stroke.Color = Color3.fromRGB(255, 0, 0)
    stroke.Thickness = 2
    stroke.Parent = tabButton

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 12)
    corner.Parent = tabButton

    tabButton.MouseButton1Click:Connect(callback)
    return tabButton
end

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "RedKidV1Hub"
screenGui.ResetOnSpawn = false
screenGui.IgnoreGuiInset = true
screenGui.DisplayOrder = 999
local success = pcall(function()
    screenGui.Parent = game:GetService("CoreGui")
end)
if not success then
    screenGui.Parent = player:WaitForChild("PlayerGui")
end

local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 440, 0, 540)
mainFrame.Position = UDim2.new(0, 10, 0, 10)
mainFrame.BackgroundColor3 = Color3.fromRGB(14, 14, 14)
mainFrame.BorderSizePixel = 0
mainFrame.Parent = screenGui
mainFrame.Active = true
mainFrame.Draggable = true

local mainStroke = Instance.new("UIStroke")
mainStroke.Color = Color3.fromRGB(255, 0,  red)
mainStroke.Thickness = 3
mainStroke.Parent = mainFrame

local mainCorner = Instance.new("UICorner")
mainCorner.CornerRadius = UDim.new(0, 18)
mainCorner.Parent = mainFrame

local header = Instance.new("Frame")
header.Size = UDim2.new(1, 0, 0, 60)
header.BackgroundTransparency = 1
header.Parent = mainFrame

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, -20, 0, 32)
titleLabel.Position = UDim2.new(0, 10, 0, 8)
titleLabel.BackgroundTransparency = 1
titleLabel.Font = Enum.Font.GothamBlack
titleLabel.TextSize = 24
titleLabel.TextColor3 = Color3.fromRGB(255, 54, 54)
titleLabel.Text = "RedKidV1"
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.Parent = header

local subtitle = Instance.new("TextLabel")
subtitle.Size = UDim2.new(1, -20, 0, 18)
subtitle.Position = UDim2.new(0, 10, 0, 36)
subtitle.BackgroundTransparency = 1
subtitle.Font = Enum.Font.Gotham
subtitle.TextSize = 12
subtitle.TextColor3 = Color3.fromRGB(210, 210, 210)
subtitle.Text = "GLOBAL OWNER UPDATE"
subtitle.TextXAlignment = Enum.TextXAlignment.Left
subtitle.Parent = header

local contentWrapper = Instance.new("Frame")
contentWrapper.Size = UDim2.new(1, -20, 1, -90)
contentWrapper.Position = UDim2.new(0, 10, 0, 70)
contentWrapper.BackgroundTransparency = 1
contentWrapper.Parent = mainFrame

local sidebar = Instance.new("Frame")
sidebar.Size = UDim2.new(0, 110, 1, 0)
sidebar.Position = UDim2.new(0, 0, 0, 0)
sidebar.BackgroundColor3 = Color3.fromRGB(18, 18, 18)
sidebar.BorderSizePixel = 0
sidebar.Parent = contentWrapper

local sidebarStroke = Instance.new("UIStroke")
sidebarStroke.Color = Color3.fromRGB(255, 0, 0)
sidebarStroke.Thickness = 2
sidebarStroke.Parent = sidebar

local sidebarCorner = Instance.new("UICorner")
sidebarCorner.CornerRadius = UDim.new(0, 14)
sidebarCorner.Parent = sidebar

local pageContainer = Instance.new("Frame")
pageContainer.Size = UDim2.new(1, -124, 1, 0)
pageContainer.Position = UDim2.new(0, 124, 0, 0)
pageContainer.BackgroundTransparency = 1
pageContainer.Parent = contentWrapper

local topTabs = Instance.new("Frame")
topTabs.Size = UDim2.new(1, 0, 0, 220)
topTabs.Position = UDim2.new(0, 0, 0, 0)
topTabs.BackgroundTransparency = 1
topTabs.Parent = sidebar

local bottomTabs = Instance.new("Frame")
bottomTabs.Size = UDim2.new(1, 0, 0, 170)
bottomTabs.Position = UDim2.new(0, 0, 1, -170)
bottomTabs.BackgroundTransparency = 1
bottomTabs.Parent = sidebar

screenContainer = pageContainer

local function createPage(name)
    local page = Instance.new("Frame")
    page.Name = name
    page.Size = UDim2.new(1, 0, 1, 0)
    page.BackgroundTransparency = 1
    page.Visible = false
    page.Parent = screenContainer
    return page
end

local function setPage(page)
    for _, child in ipairs(screenContainer:GetChildren()) do
        if child:IsA("Frame") then
            child.Visible = false
        end
    end
    if page then
        page.Visible = true
    end
end

local function clearFrame(frame)
    for _, child in ipairs(frame:GetChildren()) do
        child:Destroy()
    end
end

local function buildScrollPanel(page)
    local scroll = Instance.new("ScrollingFrame")
    scroll.Size = UDim2.new(1, 0, 1, -42)
    scroll.Position = UDim2.new(0, 0, 0, 42)
    scroll.BackgroundTransparency = 1
    scroll.BorderSizePixel = 0
    scroll.ScrollBarThickness = 10
    scroll.VerticalScrollBarInset = Enum.ScrollBarInset.Always
    scroll.CanvasSize = UDim2.new(0, 0, 0, 0)
    scroll.Parent = page

    local layout = Instance.new("UIListLayout")
    layout.Parent = scroll
    layout.SortOrder = Enum.SortOrder.LayoutOrder
    layout.Padding = UDim.new(0, 12)

    local padding = Instance.new("UIPadding")
    padding.PaddingTop = UDim.new(0, 12)
    padding.PaddingBottom = UDim.new(0, 12)
    padding.PaddingLeft = UDim.new(0, 12)
    padding.PaddingRight = UDim.new(0, 12)
    padding.Parent = scroll

    layout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        scroll.CanvasSize = UDim2.new(0, 0, 0, layout.AbsoluteContentSize.Y + 12)
    end)

    return scroll
end

local function clearScrollContent(scroll)
    for _, child in ipairs(scroll:GetChildren()) do
        if not child:IsA("UIListLayout") and not child:IsA("UIPadding") then
            child:Destroy()
        end
    end
end

local function buildMainPage(page)
    clearFrame(page)

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0, 30)
    title.Position = UDim2.new(0, 0, 0, 0)
    title.BackgroundTransparency = 1
    title.Font = Enum.Font.GothamBold
    title.TextSize = 18
    title.TextColor3 = Color3.fromRGB(255, 255, 255)
    title.Text = "MAIN MENU"
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Parent = page

    local scroll = buildScrollPanel(page)

    local aboutText = Instance.new("TextLabel")
    aboutText.Size = UDim2.new(1, 0, 0, 200)
    aboutText.BackgroundTransparency = 1
    aboutText.Font = Enum.Font.Gotham
    aboutText.TextSize = 14
    aboutText.TextColor3 = Color3.fromRGB(255, 255, 255)
    aboutText.Text = "Creator: RedKid\nRoblox User: Test1231317\n\nAbout this menu:\nRedKidV2 MEGA HUB is a global owner update mod menu with owner tools, game mods, and useful settings."
    aboutText.TextWrapped = true
    aboutText.TextXAlignment = Enum.TextXAlignment.Left
    aboutText.TextYAlignment = Enum.TextYAlignment.Top
    aboutText.Parent = scroll

    createButton(scroll, "ABOUT THE CREATOR", function()
        makeNotification("Creator: RedKid | Roblox: Test1231317")
    end)

    createButton(scroll, "ABOUT THIS MENU", function()
        makeNotification("RedKidV2 MEGA HUB loaded. Use the sidebar for Main, Games, Owner, and Settings.")
    end)
end

local function buildGamesPage(page)
    clearFrame(page)

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0, 30)
    title.Position = UDim2.new(0, 0, 0, 0)
    title.BackgroundTransparency = 1
    title.Font = Enum.Font.GothamBold
    title.TextSize = 18
    title.TextColor3 = Color3.fromRGB(255, 255, 255)
    title.Text = "GAMES"
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Parent = page

    local leftPanel = Instance.new("ScrollingFrame")
    leftPanel.Size = UDim2.new(0, 170, 1, -42)
    leftPanel.Position = UDim2.new(0, 0, 0, 42)
    leftPanel.BackgroundColor3 = Color3.fromRGB(16, 16, 16)
    leftPanel.BorderSizePixel = 0
    leftPanel.ScrollBarThickness = 8
    leftPanel.VerticalScrollBarInset = Enum.ScrollBarInset.Always
    leftPanel.CanvasSize = UDim2.new(0, 0, 0, 0)
    leftPanel.Parent = page

    local leftTitle = Instance.new("TextLabel")
    leftTitle.Size = UDim2.new(1, 0, 0, 26)
    leftTitle.Position = UDim2.new(0, 0, 0, 0)
    leftTitle.BackgroundTransparency = 1
    leftTitle.Font = Enum.Font.GothamBold
    leftTitle.TextSize = 14
    leftTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    leftTitle.Text = "GAME LIST"
    leftTitle.TextXAlignment = Enum.TextXAlignment.Center
    leftTitle.Parent = leftPanel

    local leftLayout = Instance.new("UIListLayout")
    leftLayout.Parent = leftPanel
    leftLayout.SortOrder = Enum.SortOrder.LayoutOrder
    leftLayout.Padding = UDim.new(0, 12)

    local leftPadding = Instance.new("UIPadding")
    leftPadding.PaddingTop = UDim.new(0, 36)
    leftPadding.PaddingLeft = UDim.new(0, 10)
    leftPadding.PaddingRight = UDim.new(0, 10)
    leftPadding.Parent = leftPanel

    leftLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        leftPanel.CanvasSize = UDim2.new(0, 0, 0, leftLayout.AbsoluteContentSize.Y + 42)
    end)

    local rightPanel = Instance.new("Frame")
    rightPanel.Size = UDim2.new(1, -190, 1, -42)
    rightPanel.Position = UDim2.new(0, 180, 0, 42)
    rightPanel.BackgroundColor3 = Color3.fromRGB(16, 16, 16)
    rightPanel.BorderSizePixel = 0
    rightPanel.Parent = page

    local rightTitle = Instance.new("TextLabel")
    rightTitle.Size = UDim2.new(1, 0, 0, 26)
    rightTitle.Position = UDim2.new(0, 0, 0, 0)
    rightTitle.BackgroundTransparency = 1
    rightTitle.Font = Enum.Font.GothamBold
    rightTitle.TextSize = 14
    rightTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    rightTitle.Text = "GAME MODS"
    rightTitle.TextXAlignment = Enum.TextXAlignment.Left
    rightTitle.Parent = rightPanel

    local modScroll = buildScrollPanel(rightPanel)

    local function showGameMods(gameInfo)
        clearScrollContent(modScroll)

        local header = Instance.new("TextLabel")
        header.Size = UDim2.new(1, 0, 0, 24)
        header.BackgroundTransparency = 1
        header.Font = Enum.Font.GothamBold
        header.TextSize = 15
        header.TextColor3 = Color3.fromRGB(255, 255, 255)
        header.Text = gameInfo.name .. " MODS"
        header.TextXAlignment = Enum.TextXAlignment.Left
        header.Parent = modScroll

        for _, modInfo in ipairs(gameInfo.mods) do
            createButton(modScroll, modInfo.label, modInfo.callback)
        end
    end

    local function teleportSearch(name)
        local target = safeFindNearest(name)
        if target then
            safeTeleport(target.CFrame + Vector3.new(0, 5, 0))
            makeNotification("Teleported to nearest " .. name .. ".")
        else
            makeNotification("No " .. name .. " found.")
        end
    end

    local games = {
        {
            name = "Brookhaven",
            mods = {
                {label = "Speed Boost", callback = function() safeSetSpeed(220); makeNotification("Brookhaven speed enabled.") end},
                {label = "Jump Boost", callback = function() safeSetJump(220); makeNotification("Brookhaven jump enabled.") end},
                {label = "Teleport Home", callback = function() teleportSearch("home") end},
            }
        },
        {
            name = "Blox Fruits",
            mods = {
                {label = "Speed Boost", callback = function() safeSetSpeed(240); makeNotification("Blox Fruits speed enabled.") end},
                {label = "Jump Boost", callback = function() safeSetJump(220); makeNotification("Blox Fruits jump enabled.") end},
                {label = "Grab Fruit", callback = function() teleportSearch("fruit") end},
            }
        },
        {
            name = "Prison Life",
            mods = {
                {label = "Prison Speed", callback = function() safeSetSpeed(220); makeNotification("Prison Life speed enabled.") end},
                {label = "Prison Jump", callback = function() safeSetJump(220); makeNotification("Prison Life jump enabled.") end},
                {label = "Prison Escape", callback = function() teleportSearch("spawn") end},
            }
        },
        {
            name = "Adopt Me",
            mods = {
                {label = "Speed Boost", callback = function() safeSetSpeed(220); makeNotification("Adopt Me speed enabled.") end},
                {label = "Jump Boost", callback = function() safeSetJump(220); makeNotification("Adopt Me jump enabled.") end},
                {label = "Auto Walk", callback = function() safeSetSpeed(120); makeNotification("Adopt Me auto walk enabled.") end},
            }
        },
        {
            name = "Pet Simulator X",
            mods = {
                {label = "Auto Collect", callback = function() teleportSearch("coin") end},
                {label = "Speed Pet", callback = function() safeSetSpeed(220); makeNotification("Pet speed enabled.") end},
                {label = "Gold Tracker", callback = function() teleportSearch("gold") end},
            }
        },
        {
            name = "Murder Mystery 2",
            mods = {
                {label = "Full Bright", callback = function() Lighting.Brightness = 5; makeNotification("MM2 full bright enabled.") end},
                {label = "Speed Run", callback = function() safeSetSpeed(180); makeNotification("MM2 speed enabled.") end},
                {label = "Anti Fall", callback = function() local humanoid = getHumanoid(); if humanoid then humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false); makeNotification("MM2 anti fall enabled.") end end},
            }
        },
        {
            name = "Arsenal",
            mods = {
                {label = "Speed Boost", callback = function() safeSetSpeed(220); makeNotification("Arsenal speed enabled.") end},
                {label = "No Recoil", callback = function() Lighting.Brightness = 6; makeNotification("Arsenal effect enabled.") end},
            }
        },
        {
            name = "Jailbreak",
            mods = {
                {label = "Speed Boost", callback = function() safeSetSpeed(240); makeNotification("Jailbreak speed enabled.") end},
                {label = "Car Finder", callback = function() teleportSearch("car") end},
                {label = "Cash Search", callback = function() teleportSearch("cash") end},
            }
        },
        {
            name = "Phantom Forces",
            mods = {
                {label = "Speed Boost", callback = function() safeSetSpeed(220); makeNotification("Phantom speed enabled.") end},
                {label = "Full Bright", callback = function() Lighting.Brightness = 5; makeNotification("PF full bright enabled.") end},
            }
        },
        {
            name = "Tower of Hell",
            mods = {
                {label = "No Fall", callback = function() local humanoid = getHumanoid(); if humanoid then humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false); makeNotification("Tower of Hell anti fall enabled.") end end},
                {label = "Jump Boost", callback = function() safeSetJump(260); makeNotification("Tower jump enabled.") end},
            }
        },
        {
            name = "King Legacy",
            mods = {
                {label = "Speed Boost", callback = function() safeSetSpeed(220); makeNotification("King Legacy speed enabled.") end},
                {label = "Jump Boost", callback = function() safeSetJump(220); makeNotification("King Legacy jump enabled.") end},
            }
        },
        {
            name = "Shindo Life",
            mods = {
                {label = "Speed Boost", callback = function() safeSetSpeed(220); makeNotification("Shindo Life speed enabled.") end},
                {label = "Jump Boost", callback = function() safeSetJump(220); makeNotification("Shindo Life jump enabled.") end},
            }
        },
        {
            name = "Piggy",
            mods = {
                {label = "Speed Boost", callback = function() safeSetSpeed(220); makeNotification("Piggy speed enabled.") end},
                {label = "No Clip", callback = function() toggleNoClip(10); makeNotification("Piggy noclip enabled 10s.") end},
            }
        },
        {
            name = "Bee Swarm Simulator",
            mods = {
                {label = "Speed Boost", callback = function() safeSetSpeed(220); makeNotification("Bee Swarm speed enabled.") end},
                {label = "Auto Collect", callback = function() teleportSearch("pollen") end},
            }
        }
    }

    for _, gameInfo in ipairs(games) do
        local gameBtn = Instance.new("TextButton")
        gameBtn.Size = UDim2.new(1, 0, 0, 48)
        gameBtn.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
        gameBtn.BorderSizePixel = 0
        gameBtn.Font = Enum.Font.GothamBold
        gameBtn.TextSize = 14
        gameBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        gameBtn.Text = gameInfo.name
        gameBtn.AutoButtonColor = true
        gameBtn.Parent = leftPanel

        local stroke = Instance.new("UIStroke")
        stroke.Color = Color3.fromRGB(255, 0, 0)
        stroke.Thickness = 2
        stroke.Parent = gameBtn

        local corner = Instance.new("UICorner")
        corner.CornerRadius = UDim.new(0, 10)
        corner.Parent = gameBtn

        gameBtn.MouseButton1Click:Connect(function()
            showGameMods(gameInfo)
        end)
    end

    if #games > 0 then
        showGameMods(games[1])
    end
end

local function buildOwnerPage(page)
    clearFrame(page)

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0, 30)
    title.Position = UDim2.new(0, 0, 0, 0)
    title.BackgroundTransparency = 1
    title.Font = Enum.Font.GothamBold
    title.TextSize = 18
    title.TextColor3 = Color3.fromRGB(255, 54, 54)
    title.Text = "OWNER MENU"
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Parent = page

    local ownerLabel = Instance.new("TextLabel")
    ownerLabel.Size = UDim2.new(1, 0, 0, 22)
    ownerLabel.Position = UDim2.new(0, 0, 0, 32)
    ownerLabel.BackgroundTransparency = 1
    ownerLabel.Font = Enum.Font.GothamBold
    ownerLabel.TextSize = 14
    ownerLabel.TextColor3 = Color3.fromRGB(255, 54, 54)
    ownerLabel.Text = isOwner() and "OWNER ACCESS GRANTED" or "OWNER ACCESS REQUIRED"
    ownerLabel.TextXAlignment = Enum.TextXAlignment.Left
    ownerLabel.Parent = page

    local scroll = buildScrollPanel(page)

    if isOwner() then
        createButton(scroll, "SET OWNER TAG", requestOwnerTag)
        createButton(scroll, "OWNER GOD", function() safeSetHealth(999999); makeNotification("Owner god enabled.") end)
        createButton(scroll, "OWNER SPEED", function() safeSetSpeed(260); makeNotification("Owner speed enabled.") end)
        createButton(scroll, "OWNER JUMP", function() safeSetJump(320); makeNotification("Owner jump enabled.") end)
        createButton(scroll, "OWNER FLING", function() flingNearbyPlayers(); makeNotification("Owner fling executed.") end)
        createButton(scroll, "OWNER GARDEN", createGarden)
        createButton(scroll, "BRING ALL (99 KNIGHT)", spawnAll99Knight)
    else
        createButton(scroll, "LOCKED: YOU ARE NOT OWNER", function()
            makeNotification("Owner only menu.")
        end)
    end
end

local function buildSettingsPage(page)
    clearFrame(page)

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0, 30)
    title.Position = UDim2.new(0, 0, 0, 0)
    title.BackgroundTransparency = 1
    title.Font = Enum.Font.GothamBold
    title.TextSize = 18
    title.TextColor3 = Color3.fromRGB(255, 255, 255)
    title.Text = "SETTINGS"
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Parent = page

    local scroll = buildScrollPanel(page)

    createButton(scroll, "RESET SPEED/JUMP", function()
        safeSetSpeed(16)
        safeSetJump(50)
        makeNotification("Speed and jump reset.")
    end)

    createButton(scroll, "RESET HEALTH", function()
        safeSetHealth(100)
        makeNotification("Health reset.")
    end)

    createButton(scroll, "RESET GRAVITY", function()
        safeSetGravity(196.2)
        makeNotification("Gravity reset.")
    end)

    createButton(scroll, "TOGGLE FULL BRIGHT", function()
        Lighting.Brightness = 5
        Lighting.ClockTime = 14
        makeNotification("Full bright enabled.")
    end)

    createButton(scroll, "TOGGLE OWNER TAG", function()
        requestOwnerTag()
        makeNotification("Owner tag toggled.")
    end)

    createButton(scroll, "RESET UI", function()
        local gui = player:FindFirstChildOfClass("PlayerGui") and player.PlayerGui:FindFirstChild("RedKidV1Hub")
        if gui then
            gui:Destroy()
        end
        makeNotification("UI reset. Re-run script.")
    end)
end

local mainPage = createPage("MainPage")
local gamesPage = createPage("GamesPage")
local ownerPage = createPage("OwnerPage")
local settingsPage = createPage("SettingsPage")

buildMainPage(mainPage)
buildGamesPage(gamesPage)
buildOwnerPage(ownerPage)
buildSettingsPage(settingsPage)

setPage(mainPage)

local navMain = createTabButton(topTabs, "MAIN MENU", function()
    setPage(mainPage)
end)

local navGames = createTabButton(topTabs, "GAMES", function()
    setPage(gamesPage)
end)

local navOwner = createTabButton(topTabs, "OWNER", function()
    setPage(ownerPage)
end)

local navSettings = createTabButton(bottomTabs, "SETTINGS", function()
    setPage(settingsPage)
end)

navMain.Position = UDim2.new(0.07, 0, 0, 15)
navGames.Position = UDim2.new(0.07, 0, 0, 75)
navOwner.Position = UDim2.new(0.07, 0, 0, 135)
navSettings.Position = UDim2.new(0.07, 0, 1, -55)

local dragging, dragInput, dragStart, startPos

mainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = mainFrame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

mainFrame.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and input == dragInput then
        local delta = input.Position - dragStart
        mainFrame.Position = UDim2.new(
            startPos.X.Scale,
            startPos.X.Offset + delta.X,
            startPos.Y.Scale,
            startPos.Y.Offset + delta.Y
        )
    end
end)

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Enum.KeyCode.RightShift then
        mainFrame.Visible = not mainFrame.Visible
    end
end)

if isOwner() then
    makeOwnerTag()
end

makeNotification("RedKidV1 loaded. Press RightShift to toggle.")
