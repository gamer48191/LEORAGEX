--[[
    ╔═══════════════════════════════════════════════════════════════════╗
    ║                 🌊 LEORAGEX HUB — ULTRA V6 🌊                    ║
    ║   Animado · Sidebar Tabs · Multi-Payload Admin · Turbo Upgrades   ║
    ╚═══════════════════════════════════════════════════════════════════╝
--]]

-- ═══════════════════════════════════════════════════════════════
-- CLEANUP
-- ═══════════════════════════════════════════════════════════════
local CoreGui = game:GetService("CoreGui")
pcall(function() if CoreGui:FindFirstChild("LEORAGEX_HUB") then CoreGui.LEORAGEX_HUB:Destroy() end end)
if shared._LEORAGEX_Kill then pcall(shared._LEORAGEX_Kill) end

-- ═══════════════════════════════════════════════════════════════
-- SERVICES
-- ═══════════════════════════════════════════════════════════════
local Players          = game:GetService("Players")
local UIS              = game:GetService("UserInputService")
local RunService       = game:GetService("RunService")
local TweenService     = game:GetService("TweenService")
local RS               = game:GetService("ReplicatedStorage")
local Lighting         = game:GetService("Lighting")
local StarterGui       = game:GetService("StarterGui")
local LP               = Players.LocalPlayer

-- ═══════════════════════════════════════════════════════════════
-- REAL REMOTE REFERENCES
-- ═══════════════════════════════════════════════════════════════
local CoreFolder = RS:WaitForChild("Core", 5)
local AdminService = CoreFolder and CoreFolder:FindFirstChild("AdminService")
local AdminRemote = AdminService and AdminService:FindFirstChild("RunCommandRemote")

local RemoteRequest = CoreFolder and CoreFolder:FindFirstChild("RemoteRequest")
local CashDropRedeem = RemoteRequest and RemoteRequest:FindFirstChild("CashDropService") and RemoteRequest.CashDropService:FindFirstChild("Redeem")

local RemoteSignal = CoreFolder and CoreFolder:FindFirstChild("RemoteSignal")
local ClickFruitClicked = RemoteSignal and RemoteSignal:FindFirstChild("ClickFruitService") and RemoteSignal.ClickFruitService:FindFirstChild("Clicked")

-- ═══════════════════════════════════════════════════════════════
-- CONFIG
-- ═══════════════════════════════════════════════════════════════
local cfg = {
    -- Main
    autoIncome      = false,
    incomeMulti     = 5,
    autoBuy         = false,
    autoFruit       = false,
    autoCashDrop    = false,
    phoneAccept     = false,
    
    -- Upgrades
    upgradeStand    = false,
    upgradeDepot    = false,
    upgradeDash     = false,
    upgradeTrading  = false,
    upgradeLabs     = false,
    upgradeRobotics = false,
    turboUpgrade    = true,
    turboMultiplier = 15,
    upgradeDelay    = 0.005, -- Ultra Rápido
    
    -- Progression
    autoRebirth     = false,
    autoEvolve      = false,
    autoAscend      = false,
    autoStaircase   = false,
    autoPowers      = false,
    autoManager     = false,
    autoSewerKeys   = false,
    
    -- Local
    walkSpeed       = 16,
    noclip          = false,
    infJump         = false,
    fullbright      = false,
    antiAfk         = true,
}

-- ═══════════════════════════════════════════════════════════════
-- TYCOON DETECTION
-- ═══════════════════════════════════════════════════════════════
local MyTycoon, TycoonNum = nil, "1"
local function FindTycoon()
    MyTycoon = nil
    for _, f in ipairs(workspace:GetChildren()) do
        if f and string.match(f.Name, "^Tycoon") then
            local ov = f:FindFirstChild("Owner") or f:FindFirstChild("Player")
            if ov then
                if (typeof(ov) == "Instance" and ov:IsA("ObjectValue") and ov.Value == LP) or
                   (typeof(ov) == "Instance" and ov:IsA("StringValue") and ov.Value == LP.Name) then
                    MyTycoon = f
                end
            end
            if not MyTycoon and f:GetAttribute("Owner") == LP.Name then MyTycoon = f end
            if not MyTycoon and string.find(f.Name, LP.Name) then MyTycoon = f end
            if MyTycoon then TycoonNum = string.match(f.Name, "%d+") or "1"; break end
        end
    end
end
pcall(FindTycoon)

local function GetTF() return workspace:FindFirstChild("Tycoon" .. TycoonNum) or MyTycoon end
local function GetRemotes() local tf = GetTF(); return tf and tf:FindFirstChild("Remotes") end
local function GetPurchases() local tf = GetTF(); return tf and (tf:FindFirstChild("Purchases") or tf:FindFirstChild("GlobalPurchases")) end
local function GetTycoonRemote(name) local rf = GetRemotes(); return rf and rf:FindFirstChild(name) end

-- ═══════════════════════════════════════════════════════════════
-- UI THEME & ANIMATIONS
-- ═══════════════════════════════════════════════════════════════
local THEME = {
    bg          = Color3.fromRGB(10, 15, 25),
    sidebar     = Color3.fromRGB(15, 22, 35),
    accent      = Color3.fromRGB(0, 190, 255),
    accent_dark = Color3.fromRGB(0, 100, 150),
    card        = Color3.fromRGB(20, 30, 45),
    card_hover  = Color3.fromRGB(25, 38, 55),
    text        = Color3.fromRGB(220, 240, 255),
    text_dark   = Color3.fromRGB(120, 150, 180),
    toggle_on   = Color3.fromRGB(0, 220, 150),
    toggle_off  = Color3.fromRGB(180, 50, 70),
    admin_bg    = Color3.fromRGB(8, 12, 18),
}

local function Tween(obj, props, time, style)
    local t = TweenService:Create(obj, TweenInfo.new(time or 0.2, style or Enum.EasingStyle.Quart, Enum.EasingDirection.Out), props)
    t:Play()
    return t
end

-- ═══════════════════════════════════════════════════════════════
-- LEORAGEX HUB BUILDER
-- ═══════════════════════════════════════════════════════════════
local SG = Instance.new("ScreenGui")
SG.Name = "LEORAGEX_HUB"; SG.ResetOnSpawn = false; SG.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
pcall(function() SG.Parent = CoreGui end)
if not SG.Parent then pcall(function() SG.Parent = (gethui and gethui()) or LP:WaitForChild("PlayerGui", 3) end) end

-- Main Container
local MainFrame = Instance.new("Frame", SG)
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 600, 0, 400)
MainFrame.Position = UDim2.new(0.5, -300, 0.5, -200)
MainFrame.BackgroundColor3 = THEME.bg
MainFrame.BorderSizePixel = 0
MainFrame.ClipsDescendants = true
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 10)
local mfStroke = Instance.new("UIStroke", MainFrame)
mfStroke.Color = THEME.accent; mfStroke.Thickness = 2; mfStroke.Transparency = 0.5

-- Top Drag Bar
local TopBar = Instance.new("Frame", MainFrame)
TopBar.Size = UDim2.new(1, 0, 0, 35); TopBar.BackgroundColor3 = THEME.sidebar; TopBar.BorderSizePixel = 0
local Title = Instance.new("TextLabel", TopBar)
Title.Size = UDim2.new(1, -50, 1, 0); Title.Position = UDim2.new(0, 15, 0, 0)
Title.BackgroundTransparency = 1; Title.Text = "🌊 LEORAGEX"
Title.TextColor3 = THEME.accent; Title.Font = Enum.Font.GothamBold; Title.TextSize = 16
Title.TextXAlignment = Enum.TextXAlignment.Left

-- Close Button
local CloseBtn = Instance.new("TextButton", TopBar)
CloseBtn.Size = UDim2.new(0, 20, 0, 20); CloseBtn.Position = UDim2.new(1, -30, 0, 7)
CloseBtn.BackgroundColor3 = THEME.toggle_off; CloseBtn.Text = ""; CloseBtn.BorderSizePixel = 0
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(1, 0)
CloseBtn.MouseButton1Click:Connect(function()
    if shared._LEORAGEX_Kill then pcall(shared._LEORAGEX_Kill) end
    Tween(MainFrame, {Size = UDim2.new(0,0,0,0)}, 0.3).Completed:Wait()
    SG:Destroy()
end)

-- Dragging Logic
local dragging, dragStart, startPos = false, nil, nil
TopBar.InputBegan:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true; dragStart = i.Position; startPos = MainFrame.Position
        local rel; rel = UIS.InputEnded:Connect(function(e)
            if e.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false; rel:Disconnect() end
        end)
    end
end)
UIS.InputChanged:Connect(function(i)
    if dragging and i.UserInputType == Enum.UserInputType.MouseMovement then
        local d = i.Position - dragStart
        MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X, startPos.Y.Scale, startPos.Y.Offset + d.Y)
    end
end)

-- Sidebar
local Sidebar = Instance.new("Frame", MainFrame)
Sidebar.Size = UDim2.new(0, 140, 1, -35); Sidebar.Position = UDim2.new(0, 0, 0, 35)
Sidebar.BackgroundColor3 = THEME.sidebar; Sidebar.BorderSizePixel = 0

local SidebarList = Instance.new("UIListLayout", Sidebar)
SidebarList.Padding = UDim.new(0, 5); SidebarList.SortOrder = Enum.SortOrder.LayoutOrder; SidebarList.HorizontalAlignment = Enum.HorizontalAlignment.Center
Instance.new("UIPadding", Sidebar).PaddingTop = UDim.new(0, 10)

-- Content Area
local ContentArea = Instance.new("Frame", MainFrame)
ContentArea.Size = UDim2.new(1, -140, 1, -35); ContentArea.Position = UDim2.new(0, 140, 0, 35)
ContentArea.BackgroundColor3 = THEME.bg; ContentArea.BorderSizePixel = 0

local Tabs = {}
local activeTab = nil

local function CreateTab(name, icon)
    -- Sidebar Button
    local TabBtn = Instance.new("TextButton", Sidebar)
    TabBtn.Size = UDim2.new(0.9, 0, 0, 35); TabBtn.BackgroundColor3 = THEME.bg; TabBtn.BorderSizePixel = 0
    TabBtn.Text = "  " .. icon .. " " .. name; TabBtn.TextColor3 = THEME.text_dark
    TabBtn.Font = Enum.Font.GothamSemibold; TabBtn.TextSize = 12; TabBtn.TextXAlignment = Enum.TextXAlignment.Left
    Instance.new("UICorner", TabBtn).CornerRadius = UDim.new(0, 6)
    
    -- Content Frame
    local TabFrame = Instance.new("ScrollingFrame", ContentArea)
    TabFrame.Size = UDim2.new(1, 0, 1, 0); TabFrame.BackgroundTransparency = 1
    TabFrame.ScrollBarThickness = 2; TabFrame.ScrollBarImageColor3 = THEME.accent
    TabFrame.Visible = false; TabFrame.CanvasSize = UDim2.new(0,0,0,0); TabFrame.AutomaticCanvasSize = Enum.AutomaticSize.Y
    
    local TLayout = Instance.new("UIListLayout", TabFrame)
    TLayout.Padding = UDim.new(0, 8); TLayout.SortOrder = Enum.SortOrder.LayoutOrder; TLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    Instance.new("UIPadding", TabFrame).PaddingTop = UDim.new(0, 10); Instance.new("UIPadding", TabFrame).PaddingBottom = UDim.new(0, 10)
    
    -- Logic
    TabBtn.MouseButton1Click:Connect(function()
        if activeTab == TabFrame then return end
        if activeTab then
            activeTab.Visible = false
            for _, v in ipairs(Sidebar:GetChildren()) do
                if v:IsA("TextButton") then
                    Tween(v, {BackgroundColor3 = THEME.bg, TextColor3 = THEME.text_dark}, 0.2)
                end
            end
        end
        activeTab = TabFrame
        TabFrame.Visible = true
        Tween(TabBtn, {BackgroundColor3 = THEME.accent_dark, TextColor3 = THEME.text}, 0.2)
        
        -- Slide Animation
        TabFrame.Position = UDim2.new(0, 20, 0, 0); TabFrame.CanvasPosition = Vector2.new(0,0)
        Tween(TabFrame, {Position = UDim2.new(0, 0, 0, 0)}, 0.3)
    end)
    
    TabBtn.MouseEnter:Connect(function() if activeTab ~= TabFrame then Tween(TabBtn, {BackgroundColor3 = THEME.card}, 0.2) end end)
    TabBtn.MouseLeave:Connect(function() if activeTab ~= TabFrame then Tween(TabBtn, {BackgroundColor3 = THEME.bg}, 0.2) end end)

    table.insert(Tabs, {Btn = TabBtn, Frame = TabFrame})
    return TabFrame
end

local function CreateToggle(parent, label, key)
    local f = Instance.new("Frame", parent)
    f.Size = UDim2.new(0.95, 0, 0, 38); f.BackgroundColor3 = THEME.card; f.BorderSizePixel = 0
    Instance.new("UICorner", f).CornerRadius = UDim.new(0, 6)
    
    local l = Instance.new("TextLabel", f)
    l.Size = UDim2.new(0.7, 0, 1, 0); l.Position = UDim2.new(0, 15, 0, 0); l.BackgroundTransparency = 1
    l.Text = label; l.TextColor3 = THEME.text; l.Font = Enum.Font.GothamSemibold
    l.TextSize = 13; l.TextXAlignment = Enum.TextXAlignment.Left

    local toggleBg = Instance.new("Frame", f)
    toggleBg.Size = UDim2.new(0, 40, 0, 20); toggleBg.Position = UDim2.new(1, -55, 0.5, -10)
    toggleBg.BackgroundColor3 = cfg[key] and THEME.toggle_on or THEME.toggle_off
    Instance.new("UICorner", toggleBg).CornerRadius = UDim.new(1, 0)

    local toggleDot = Instance.new("Frame", toggleBg)
    toggleDot.Size = UDim2.new(0, 16, 0, 16)
    toggleDot.Position = cfg[key] and UDim2.new(1, -18, 0.5, -8) or UDim2.new(0, 2, 0.5, -8)
    toggleDot.BackgroundColor3 = Color3.new(1,1,1)
    Instance.new("UICorner", toggleDot).CornerRadius = UDim.new(1, 0)

    local btn = Instance.new("TextButton", f)
    btn.Size = UDim2.new(1,0,1,0); btn.BackgroundTransparency = 1; btn.Text = ""

    btn.MouseButton1Click:Connect(function()
        cfg[key] = not cfg[key]
        local state = cfg[key]
        Tween(toggleBg, {BackgroundColor3 = state and THEME.toggle_on or THEME.toggle_off}, 0.2)
        Tween(toggleDot, {Position = state and UDim2.new(1, -18, 0.5, -8) or UDim2.new(0, 2, 0.5, -8)}, 0.2)
    end)
    
    btn.MouseEnter:Connect(function() Tween(f, {BackgroundColor3 = THEME.card_hover}, 0.2) end)
    btn.MouseLeave:Connect(function() Tween(f, {BackgroundColor3 = THEME.card}, 0.2) end)
end

local function CreateButton(parent, label, callback)
    local f = Instance.new("Frame", parent)
    f.Size = UDim2.new(0.95, 0, 0, 38); f.BackgroundColor3 = THEME.accent_dark; f.BorderSizePixel = 0
    Instance.new("UICorner", f).CornerRadius = UDim.new(0, 6)

    local btn = Instance.new("TextButton", f)
    btn.Size = UDim2.new(1,0,1,0); btn.BackgroundTransparency = 1; btn.Text = "⚡ " .. label
    btn.TextColor3 = Color3.new(1,1,1); btn.Font = Enum.Font.GothamBold; btn.TextSize = 13

    btn.MouseButton1Click:Connect(function()
        Tween(f, {Size = UDim2.new(0.9, 0, 0, 34)}, 0.1, Enum.EasingStyle.Sine).Completed:Wait()
        Tween(f, {Size = UDim2.new(0.95, 0, 0, 38)}, 0.1, Enum.EasingStyle.Sine)
        if callback then callback() end
    end)
end

-- ═══════════════════════════════════════════════════════════════
-- TABS & CONTENT
-- ═══════════════════════════════════════════════════════════════
local TabMain     = CreateTab("Main", "🏠")
local TabUpgrades = CreateTab("Upgrades", "⬆️")
local TabFarm     = CreateTab("Farming", "🌾")
local TabProg     = CreateTab("Progression", "🔄")
local TabLocal    = CreateTab("LocalPlayer", "🏃")
local TabAdmin    = CreateTab("Admin", "🔑")

-- Main Tab
CreateToggle(TabMain, "Auto Collect Income", "autoIncome")
CreateToggle(TabMain, "Auto Buy Infrastructure", "autoBuy")
CreateToggle(TabMain, "Auto Cash Drops", "autoCashDrop")
CreateToggle(TabMain, "Auto Phone Offers", "phoneAccept")

-- Upgrades Tab (Turbo)
local ULabel = Instance.new("TextLabel", TabUpgrades)
ULabel.Size = UDim2.new(0.95, 0, 0, 20); ULabel.BackgroundTransparency = 1
ULabel.Text = "  ⚡ Turbo Level: x" .. cfg.turboMultiplier .. " | Delay: " .. cfg.upgradeDelay .. "s"
ULabel.TextColor3 = THEME.accent; ULabel.Font = Enum.Font.GothamBold; ULabel.TextSize = 11; ULabel.TextXAlignment = Enum.TextXAlignment.Left

CreateToggle(TabUpgrades, "Turbo Upgrade Mode", "turboUpgrade")
CreateToggle(TabUpgrades, "Upgrade Lemon Stand", "upgradeStand")
CreateToggle(TabUpgrades, "Upgrade Lemon Depot", "upgradeDepot")
CreateToggle(TabUpgrades, "Upgrade Lemon Dash", "upgradeDash")
CreateToggle(TabUpgrades, "Upgrade Lemon Trading", "upgradeTrading")
CreateToggle(TabUpgrades, "Upgrade Lemon Labs", "upgradeLabs")
CreateToggle(TabUpgrades, "Upgrade Lemon Robotics", "upgradeRobotics")

-- Farming Tab
CreateToggle(TabFarm, "Auto Fruit (TP + Remote)", "autoFruit")
CreateToggle(TabFarm, "Auto Sewer Keys (TP)", "autoSewerKeys")

-- Progression Tab
CreateToggle(TabProg, "Auto Rebirth", "autoRebirth")
CreateToggle(TabProg, "Auto Evolve", "autoEvolve")
CreateToggle(TabProg, "Auto Ascend", "autoAscend")
CreateToggle(TabProg, "Auto Staircase", "autoStaircase")
CreateToggle(TabProg, "Auto Powers", "autoPowers")
CreateToggle(TabProg, "Auto Hire Managers", "autoManager")

-- Local Tab
CreateToggle(TabLocal, "Noclip", "noclip")
CreateToggle(TabLocal, "Infinite Jump", "infJump")
CreateToggle(TabLocal, "Fullbright", "fullbright")
CreateToggle(TabLocal, "Anti-AFK", "antiAfk")
CreateButton(TabLocal, "Re-Detect Tycoon", function()
    FindTycoon()
    StarterGui:SetCore("SendNotification", {Title="LEORAGEX", Text="Tycoon: " .. (MyTycoon and MyTycoon.Name or "None"), Duration=3})
end)

-- ═══════════════════════════════════════════════════════════════
-- MULTI-PAYLOAD ADMIN EXECUTOR (FIX)
-- ═══════════════════════════════════════════════════════════════
local adminStatus = Instance.new("TextLabel", TabAdmin)
adminStatus.Size = UDim2.new(0.95, 0, 0, 20); adminStatus.BackgroundTransparency = 1
adminStatus.Text = AdminRemote and "✓ RunCommandRemote Ready" or "✗ RunCommandRemote Not Found"
adminStatus.TextColor3 = AdminRemote and THEME.toggle_on or THEME.toggle_off
adminStatus.Font = Enum.Font.GothamBold; adminStatus.TextSize = 11; adminStatus.TextXAlignment = Enum.TextXAlignment.Left

local InputFrame = Instance.new("Frame", TabAdmin)
InputFrame.Size = UDim2.new(0.95, 0, 0, 40); InputFrame.BackgroundColor3 = THEME.admin_bg; InputFrame.BorderSizePixel = 0
Instance.new("UICorner", InputFrame).CornerRadius = UDim.new(0, 6)

local CmdInput = Instance.new("TextBox", InputFrame)
CmdInput.Size = UDim2.new(0.7, 0, 1, 0); CmdInput.Position = UDim2.new(0, 10, 0, 0)
CmdInput.BackgroundTransparency = 1; CmdInput.Text = ""; CmdInput.PlaceholderText = "Type command (e.g. kill all)"
CmdInput.TextColor3 = THEME.accent; CmdInput.Font = Enum.Font.GothamSemibold; CmdInput.TextSize = 12
CmdInput.TextXAlignment = Enum.TextXAlignment.Left; CmdInput.ClearTextOnFocus = false

local SendBtn = Instance.new("TextButton", InputFrame)
SendBtn.Size = UDim2.new(0.25, 0, 0, 30); SendBtn.Position = UDim2.new(0.73, 0, 0.5, -15)
SendBtn.BackgroundColor3 = THEME.accent_dark; SendBtn.Text = "EXECUTE"; SendBtn.TextColor3 = Color3.new(1,1,1)
SendBtn.Font = Enum.Font.GothamBold; SendBtn.TextSize = 11; Instance.new("UICorner", SendBtn).CornerRadius = UDim.new(0, 5)

local LogBox = Instance.new("TextLabel", TabAdmin)
LogBox.Size = UDim2.new(0.95, 0, 0, 80); LogBox.BackgroundColor3 = THEME.admin_bg; LogBox.BorderSizePixel = 0
LogBox.Text = "> Waiting for commands...\nNote: Server might have a UserID whitelist if commands do not apply."; LogBox.TextColor3 = THEME.text_dark
LogBox.Font = Enum.Font.Code; LogBox.TextSize = 11; LogBox.TextXAlignment = Enum.TextXAlignment.Left; LogBox.TextYAlignment = Enum.TextYAlignment.Top
LogBox.TextWrapped = true; Instance.new("UICorner", LogBox).CornerRadius = UDim.new(0, 6)
Instance.new("UIPadding", LogBox).PaddingTop = UDim.new(0, 5); Instance.new("UIPadding", LogBox).PaddingLeft = UDim.new(0, 5)

-- The Executor
local function ExecuteAdmin(fullCmd)
    if not fullCmd or fullCmd == "" or not AdminRemote then return end
    
    local args = string.split(fullCmd, " ")
    local baseCmd = table.remove(args, 1)
    local result = ""
    local success = false
    local err = ""

    local function fire(...)
        if AdminRemote:IsA("RemoteFunction") then return AdminRemote:InvokeServer(...)
        else AdminRemote:FireServer(...); return "Fired Event" end
    end

    -- Payload 1: Raw String
    success, err = pcall(function() result = fire(fullCmd) end)
    
    -- Payload 2: Cmd + Args unpacked
    if not success or result == nil then
        success, err = pcall(function() result = fire(baseCmd, unpack(args)) end)
    end
    
    -- Payload 3: Table with Command/Args array
    if not success or result == nil then
        success, err = pcall(function() result = fire({Command = baseCmd, Args = args}) end)
    end
    
    -- Payload 4: Table with Cmd/Arguments array
    if not success or result == nil then
        success, err = pcall(function() result = fire({Cmd = baseCmd, Arguments = args}) end)
    end

    -- Payload 5: Lowercase table
    if not success or result == nil then
        success, err = pcall(function() result = fire({command = baseCmd, args = args}) end)
    end

    if success then
        LogBox.Text = "> " .. fullCmd .. "\n< SUCCESS: " .. tostring(result or "Fired but no return")
    else
        LogBox.Text = "> " .. fullCmd .. "\n< FAILED: " .. tostring(err) .. "\n(Possible Whitelist/Patch)"
    end
end

SendBtn.MouseButton1Click:Connect(function() ExecuteAdmin(CmdInput.Text); CmdInput.Text = "" end)
CmdInput.FocusLost:Connect(function(ep) if ep then ExecuteAdmin(CmdInput.Text); CmdInput.Text = "" end end)

CreateButton(TabAdmin, "give money 999999", function() ExecuteAdmin("give money 999999") end)
CreateButton(TabAdmin, "kill all", function() ExecuteAdmin("kill all") end)
CreateButton(TabAdmin, "btools", function() ExecuteAdmin("btools") end)

-- Initial Tab
Tabs[1].Btn.BackgroundColor3 = THEME.accent_dark
Tabs[1].Btn.TextColor3 = THEME.text
Tabs[1].Frame.Visible = true
activeTab = Tabs[1].Frame

-- ═══════════════════════════════════════════════════════════════
-- ENGINES (Turbo & Stealth)
-- ═══════════════════════════════════════════════════════════════

-- Engine 1: Income Turbo & Cash Drops
task.spawn(function()
    local streams = {"LemonStand", "LemonDepot", "LemonDash", "LemonTrading", "LemonLabs", "LemonRobotics"}
    while true do
        if not SG or not SG.Parent then break end
        if cfg.autoIncome then
            pcall(function()
                local wr = GetTycoonRemote("WakeIncomeStream")
                if wr then
                    for _ = 1, cfg.incomeMulti do
                        for _, s in ipairs(streams) do
                            task.spawn(function()
                                pcall(function()
                                    if wr:IsA("RemoteFunction") then wr:InvokeServer(s) else wr:FireServer(s) end
                                end)
                            end)
                        end
                    end
                end
            end)
        end
        if cfg.autoCashDrop then
            pcall(function()
                if CashDropRedeem then
                    pcall(function() if CashDropRedeem:IsA("RemoteFunction") then CashDropRedeem:InvokeServer() else CashDropRedeem:FireServer() end end)
                    pcall(function() CashDropRedeem:InvokeServer("all") end)
                end
                local rp = LP.Character and (LP.Character:FindFirstChild("HumanoidRootPart") or LP.Character:FindFirstChild("UpperTorso"))
                local cf = workspace:FindFirstChild("CashDrops")
                if rp and cf and firetouchinterest then
                    for _, d in ipairs(cf:GetChildren()) do
                        if d:IsA("BasePart") then firetouchinterest(rp, d, 0); task.wait(); firetouchinterest(rp, d, 1) end
                    end
                end
            end)
        end
        task.wait(0.05)
    end
end)

-- Engine 2: Auto Buy
task.spawn(function()
    while true do
        if not SG or not SG.Parent then break end
        if cfg.autoBuy then
            pcall(function()
                local p = GetPurchases()
                if not p then return end
                for _, obj in ipairs(p:GetDescendants()) do
                    if not cfg.autoBuy then break end
                    if obj.Name == "Purchase" and (obj:IsA("RemoteFunction") or obj:IsA("RemoteEvent")) then
                        local bm = obj.Parent
                        if bm then
                            local tp = bm:FindFirstChild("Head") or bm:FindFirstChild("Pad") or bm:FindFirstChild("Button") or bm:FindFirstChildOfClass("BasePart")
                            if tp and tp:IsA("BasePart") and tp.Transparency < 1 then
                                pcall(function() if obj:IsA("RemoteFunction") then obj:InvokeServer(false) else obj:FireServer(false) end end)
                            end
                        end
                    end
                end
            end)
        end
        task.wait(0.1)
    end
end)

-- Engine 3: ULTRA Turbo Upgrades
local upMap = {
    {"Lemon Stand", 1, "upgradeStand"}, {"Lemon Depot", 1, "upgradeDepot"}, {"Lemon Dash", 1, "upgradeDash"}, {"LemonDash", 1, "upgradeDash"},
    {"Lemon Trading", 1, "upgradeTrading"}, {"Lemon Labs", 5, "upgradeLabs"}, {"Lemon Robotics", 25, "upgradeRobotics"}
}
local function DoUp(machine, arg)
    local p = GetPurchases(); if not p then return end
    local f = p:FindFirstChild(machine); if not f then return end
    local up = nil
    local cur = f
    for _=1, 4 do local n = cur:FindFirstChild(machine); if n then cur=n else break end end
    up = cur:FindFirstChild("Upgrade")
    if not up then
        for _, o in ipairs(f:GetDescendants()) do
            if o.Name == "Upgrade" and (o:IsA("RemoteFunction") or o:IsA("RemoteEvent")) then up = o; break end
        end
    end
    if up then
        pcall(function() if up:IsA("RemoteFunction") then up:InvokeServer(arg) else up:FireServer(arg) end end)
    end
end

task.spawn(function()
    while true do
        if not SG or not SG.Parent then break end
        pcall(function()
            local reps = cfg.turboUpgrade and cfg.turboMultiplier or 1
            for _ = 1, reps do
                for _, m in ipairs(upMap) do
                    if cfg[m[3]] then DoUp(m[1], m[2]) end
                end
            end
        end)
        task.wait(cfg.upgradeDelay) -- 0.005s MAX SPEED
    end
end)

-- Engine 4: Phone & Fruit
task.spawn(function()
    while true do
        if not SG or not SG.Parent then break end
        if cfg.phoneAccept then
            pcall(function()
                local rf = GetRemotes(); if not rf then return end
                local po = rf:FindFirstChild("PhoneOffer"); local si = rf:FindFirstChild("SpecialIncome")
                local f = firesignal or (syn and syn.firesignal)
                if f then
                    if po then pcall(function() f(po.OnClientEvent, true) end) end
                    if si then pcall(function() f(si.OnClientEvent, "PhoneOffer", 19.685056) end) end
                end
                if po then
                    if po:IsA("RemoteEvent") then pcall(function() po:FireServer("Accept") end) pcall(function() po:FireServer(true) end)
                    elseif po:IsA("RemoteFunction") then pcall(function() po:InvokeServer("Accept") end) end
                end
            end)
        end
        if cfg.autoFruit then
            pcall(function()
                if ClickFruitClicked and ClickFruitClicked:IsA("RemoteEvent") then
                    pcall(function() ClickFruitClicked:FireServer() end)
                    pcall(function() ClickFruitClicked:FireServer("all") end)
                end
                
                local rp = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
                if rp then
                    for _, obj in ipairs(workspace:GetDescendants()) do
                        if not cfg.autoFruit then break end
                        local fp = string.lower(obj:GetFullName())
                        if string.find(fp, "constant") and string.find(fp, "trees") and not string.find(fp, "purchases") then
                            local tr, pos = nil, nil
                            if obj:IsA("ClickDetector") or obj:IsA("ProximityPrompt") then
                                tr = obj; if obj.Parent and obj.Parent:IsA("BasePart") then pos = obj.Parent.CFrame end
                            elseif obj:IsA("BasePart") then
                                local c = obj:FindFirstChildOfClass("ClickDetector"); local p = obj:FindFirstChildOfClass("ProximityPrompt")
                                if c or p then tr = c or p; pos = obj.CFrame end
                            end
                            if tr and pos then
                                local s = rp.CFrame; rp.CFrame = pos; task.wait(0.04)
                                if tr:IsA("ClickDetector") and fireclickdetector then fireclickdetector(tr)
                                elseif tr:IsA("ProximityPrompt") and fireproximityprompt then fireproximityprompt(tr) end
                                task.wait(0.04); rp.CFrame = s; task.wait(0.05)
                            end
                        end
                    end
                end
            end)
        end
        task.wait(0.2)
    end
end)

-- Engine 5: Progression (Rebirth, Evolve, Ascend, etc)
task.spawn(function()
    while true do
        if not SG or not SG.Parent then break end
        pcall(function()
            local function check(n, flag)
                if not cfg[flag] then return false end
                return n:find(flag:lower():gsub("auto","")) or (flag == "autoStaircase" and n:find("stair")) or (flag == "autoPowers" and (n:find("power") or n:find("ability"))) or (flag == "autoManager" and (n:find("manager") or n:find("worker")))
            end
            
            for _, src in ipairs({GetRemotes(), RS}) do
                if src then
                    for _, o in ipairs(src:GetDescendants()) do
                        local n = o.Name:lower()
                        if check(n, "autoRebirth") or check(n, "autoEvolve") or check(n, "autoAscend") or check(n, "autoStaircase") or check(n, "autoPowers") or check(n, "autoManager") then
                            if o:IsA("RemoteFunction") then pcall(function() o:InvokeServer() end) pcall(function() o:InvokeServer(true) end)
                            elseif o:IsA("RemoteEvent") then pcall(function() o:FireServer() end) pcall(function() o:FireServer(true) end) end
                        end
                    end
                end
            end
        end)
        task.wait(5)
    end
end)

-- Engine 6: Misc
task.spawn(function()
    while true do
        if not SG or not SG.Parent then break end
        pcall(function()
            local h = LP.Character and LP.Character:FindFirstChildOfClass("Humanoid")
            if h and cfg.walkSpeed > 0 then h.WalkSpeed = cfg.walkSpeed end
            if cfg.fullbright then Lighting.Brightness = 2; Lighting.ClockTime = 14; Lighting.GlobalShadows = false end
        end)
        task.wait(0.1)
    end
end)

RunService.Stepped:Connect(function()
    if cfg.noclip and LP.Character then for _, p in ipairs(LP.Character:GetDescendants()) do if p:IsA("BasePart") then p.CanCollide = false end end end
end)
UIS.JumpRequest:Connect(function()
    if cfg.infJump and LP.Character then local h = LP.Character:FindFirstChildOfClass("Humanoid"); if h then h:ChangeState(Enum.HumanoidStateType.Jumping) end end
end)

-- Kill Switch
shared._LEORAGEX_Kill = function()
    for k, v in pairs(cfg) do if typeof(v) == "boolean" then cfg[k] = false end end
end

-- Intro Animation
MainFrame.Size = UDim2.new(0,0,0,0)
Tween(MainFrame, {Size = UDim2.new(0, 600, 0, 400)}, 0.4, Enum.EasingStyle.Back)
