--[[
    ╔═══════════════════════════════════════════════════════════════╗
    ║        SELL LEMONS 🍋 — SCRIPT COMPLETO OP v3.0              ║
    ║   Estrutura real do jogo · Remotes verificados · Funcional   ║
    ║   Auto Income · Auto Buy · Auto Upgrade (6 máquinas) ·       ║
    ║   Auto Fruit · Auto Phone · Auto Cash Drops · Speed Hack     ║
    ╚═══════════════════════════════════════════════════════════════╝
    
    ESTRUTURA DO JOGO (verificada):
      workspace/Tycoon{N}/Remotes/WakeIncomeStream (RemoteFunction)
      workspace/Tycoon{N}/Remotes/PhoneOffer (RemoteEvent)
      workspace/Tycoon{N}/Remotes/SpecialIncome (RemoteEvent)
      workspace/Tycoon{N}/Purchases/{Machine}/Purchase (RemoteFunction)
      workspace/Tycoon{N}/Purchases/{Machine}/{Machine}/{Machine}/Upgrade (RemoteFunction)
      workspace/CashDrops/ (BaseParts chamadas "CashDrop")
      workspace/.../Constant/Trees/... (Fruits com ClickDetector/ProximityPrompt)
      
    MÁQUINAS DE UPGRADE:
      - Lemon Stand (InvokeServer(1))
      - Lemon Depot (InvokeServer(1))
      - Lemon Dash (InvokeServer(1))
      - Lemon Trading (InvokeServer(1))
      - Lemon Labs (InvokeServer(5))
      - Lemon Robotics (InvokeServer(25))
--]]

-- ═══════════════════════════════════════════════════════════════
-- CLEANUP: mata qualquer instância anterior do script
-- ═══════════════════════════════════════════════════════════════
local CoreGui = game:GetService("CoreGui")
if CoreGui:FindFirstChild("SellLemonsV3") then
    pcall(function() CoreGui.SellLemonsV3:Destroy() end)
end
if shared._SL_KillAutoBuy then pcall(shared._SL_KillAutoBuy) end
if shared._SL_KillFruit then pcall(shared._SL_KillFruit) end

-- ═══════════════════════════════════════════════════════════════
-- SERVICES
-- ═══════════════════════════════════════════════════════════════
local Players          = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService       = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer

-- ═══════════════════════════════════════════════════════════════
-- CONFIG
-- ═══════════════════════════════════════════════════════════════
local config = {
    autoIncome      = false,
    autoBuy         = false,
    upgradeStand    = false,
    upgradeDepot    = false,
    upgradeDash     = false,
    upgradeTrading  = false,
    upgradeLabs     = false,
    upgradeRobotics = false,
    autoPickFruit   = false,
    phoneAutoAccept = false,
    walkSpeed       = 16,
    incomeDelay     = 0.3,
    upgradeDelay    = 0.3,
    noclip          = false,
    infJump         = false,
    antiAfk         = true,
}

-- ═══════════════════════════════════════════════════════════════
-- TYCOON DETECTION (estrutura real: workspace/Tycoon{N})
-- ═══════════════════════════════════════════════════════════════
local MyTycoon = nil
local TycoonNumber = "1"

local function FindMyTycoon()
    MyTycoon = nil
    for _, folder in ipairs(workspace:GetChildren()) do
        if folder and string.match(folder.Name, "^Tycoon") then
            -- Method 1: Owner ObjectValue/StringValue
            local ownerValue = folder:FindFirstChild("Owner") or folder:FindFirstChild("Player")
            if ownerValue then
                if typeof(ownerValue) == "Instance" then
                    if ownerValue:IsA("ObjectValue") and ownerValue.Value == LocalPlayer then
                        MyTycoon = folder
                    elseif ownerValue:IsA("StringValue") and ownerValue.Value == LocalPlayer.Name then
                        MyTycoon = folder
                    end
                end
            end
            -- Method 2: Attribute
            if not MyTycoon and folder:GetAttribute("Owner") == LocalPlayer.Name then
                MyTycoon = folder
            end
            -- Method 3: Name contains player name
            if not MyTycoon and string.find(folder.Name, LocalPlayer.Name) then
                MyTycoon = folder
            end

            if MyTycoon then
                TycoonNumber = string.match(folder.Name, "%d+") or "1"
                break
            end
        end
    end
end
pcall(FindMyTycoon)

-- ═══════════════════════════════════════════════════════════════
-- REMOTE REFERENCES (estrutura real verificada)
-- ═══════════════════════════════════════════════════════════════
local function GetTycoonFolder()
    return workspace:FindFirstChild("Tycoon" .. TycoonNumber)
end

local function GetRemotesFolder()
    local tf = GetTycoonFolder()
    if tf then
        return tf:FindFirstChild("Remotes")
    end
    if MyTycoon then
        return MyTycoon:FindFirstChild("Remotes")
    end
    return nil
end

local function GetPurchasesFolder()
    local tf = GetTycoonFolder()
    if tf then
        return tf:FindFirstChild("Purchases")
    end
    if MyTycoon then
        return MyTycoon:FindFirstChild("Purchases")
    end
    return nil
end

local function GetIncomeRemote()
    local rf = GetRemotesFolder()
    return rf and rf:FindFirstChild("WakeIncomeStream")
end

local function GetPhoneRemote()
    local rf = GetRemotesFolder()
    return rf and rf:FindFirstChild("PhoneOffer")
end

local function GetSpecialIncomeRemote()
    local rf = GetRemotesFolder()
    return rf and rf:FindFirstChild("SpecialIncome")
end

-- ═══════════════════════════════════════════════════════════════
-- GUI BUILDER
-- ═══════════════════════════════════════════════════════════════
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "SellLemonsV3"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
local guiOk, _ = pcall(function() ScreenGui.Parent = CoreGui end)
if not guiOk then
    pcall(function() ScreenGui.Parent = (gethui and gethui()) or LocalPlayer:WaitForChild("PlayerGui", 3) end)
end

local MainFrame = Instance.new("Frame")
MainFrame.Name = "Main"
MainFrame.Size = UDim2.new(0, 280, 0, 620)
MainFrame.Position = UDim2.new(0.5, -140, 0.4, -280)
MainFrame.BackgroundColor3 = Color3.fromRGB(20, 22, 15)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 8)
local mainStroke = Instance.new("UIStroke", MainFrame)
mainStroke.Color = Color3.fromRGB(180, 200, 0); mainStroke.Thickness = 1.2

-- Drag system
local DragBar = Instance.new("Frame", MainFrame)
DragBar.Size = UDim2.new(1, 0, 0, 38)
DragBar.BackgroundColor3 = Color3.fromRGB(200, 220, 0)
DragBar.BorderSizePixel = 0
Instance.new("UICorner", DragBar).CornerRadius = UDim.new(0, 8)

local Title = Instance.new("TextLabel", DragBar)
Title.Text = "🍋 SELL LEMONS OP v3.0"
Title.Size = UDim2.new(1, -80, 1, 0)
Title.Position = UDim2.new(0, 10, 0, 0)
Title.BackgroundTransparency = 1
Title.TextColor3 = Color3.fromRGB(20, 20, 5)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 14
Title.TextXAlignment = Enum.TextXAlignment.Left

-- Dragging logic
local dragging, dragStart, startPos = false, nil, nil
DragBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true; dragStart = input.Position; startPos = MainFrame.Position
        local rel; rel = UserInputService.InputEnded:Connect(function(ei)
            if ei.UserInputType == Enum.UserInputType.MouseButton1 or ei.UserInputType == Enum.UserInputType.Touch then
                dragging = false; rel:Disconnect()
            end
        end)
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local d = input.Position - dragStart
        MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X, startPos.Y.Scale, startPos.Y.Offset + d.Y)
    end
end)

-- Minimize & Close
local MinBtn = Instance.new("TextButton", DragBar)
MinBtn.Size = UDim2.new(0, 28, 0, 28); MinBtn.Position = UDim2.new(1, -65, 0, 5)
MinBtn.Text = "−"; MinBtn.TextSize = 18; MinBtn.Font = Enum.Font.GothamBold
MinBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50); MinBtn.TextColor3 = Color3.new(1,1,1)
MinBtn.BorderSizePixel = 0; Instance.new("UICorner", MinBtn).CornerRadius = UDim.new(0, 4)

local CloseBtn = Instance.new("TextButton", DragBar)
CloseBtn.Size = UDim2.new(0, 28, 0, 28); CloseBtn.Position = UDim2.new(1, -33, 0, 5)
CloseBtn.Text = "X"; CloseBtn.TextSize = 12; CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.BackgroundColor3 = Color3.fromRGB(160, 40, 40); CloseBtn.TextColor3 = Color3.new(1,1,1)
CloseBtn.BorderSizePixel = 0; Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 4)

local isMinimized = false
MinBtn.MouseButton1Click:Connect(function()
    isMinimized = not isMinimized
    MainFrame:TweenSize(
        isMinimized and UDim2.new(0, 280, 0, 38) or UDim2.new(0, 280, 0, 620),
        Enum.EasingDirection.Out, Enum.EasingStyle.Quart, 0.2, true
    )
    MinBtn.Text = isMinimized and "+" or "−"
end)

CloseBtn.MouseButton1Click:Connect(function()
    if shared._SL_KillAutoBuy then pcall(shared._SL_KillAutoBuy) end
    if shared._SL_KillFruit then pcall(shared._SL_KillFruit) end
    pcall(function()
        local h = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if h then h.WalkSpeed = 16 end
    end)
    ScreenGui:Destroy()
end)

-- Content scroll
local Content = Instance.new("ScrollingFrame", MainFrame)
Content.Size = UDim2.new(1, -16, 1, -48)
Content.Position = UDim2.new(0, 8, 0, 43)
Content.BackgroundTransparency = 1
Content.ScrollBarThickness = 3
Content.ScrollBarImageColor3 = Color3.fromRGB(200, 220, 0)
Content.AutomaticCanvasSize = Enum.AutomaticSize.Y
Content.CanvasSize = UDim2.new(0, 0, 0, 0)

local Layout = Instance.new("UIListLayout", Content)
Layout.Padding = UDim.new(0, 5)
Layout.SortOrder = Enum.SortOrder.LayoutOrder

local order = 0
local function nextOrd() order = order + 1; return order end

-- Section Header
local function Section(name)
    local f = Instance.new("Frame", Content)
    f.Size = UDim2.new(1, 0, 0, 26)
    f.BackgroundColor3 = Color3.fromRGB(180, 200, 0)
    f.BorderSizePixel = 0; f.LayoutOrder = nextOrd()
    Instance.new("UICorner", f).CornerRadius = UDim.new(0, 5)
    local l = Instance.new("TextLabel", f)
    l.Text = "  ▸ " .. name; l.Size = UDim2.new(1,0,1,0)
    l.BackgroundTransparency = 1; l.TextColor3 = Color3.fromRGB(20,20,5)
    l.Font = Enum.Font.GothamBold; l.TextSize = 12
    l.TextXAlignment = Enum.TextXAlignment.Left
end

-- Toggle Button
local function Toggle(label, configKey, onToggle)
    local toggled = config[configKey]

    local f = Instance.new("Frame", Content)
    f.Size = UDim2.new(1, 0, 0, 32)
    f.BackgroundColor3 = Color3.fromRGB(30, 32, 22)
    f.BorderSizePixel = 0; f.LayoutOrder = nextOrd()
    Instance.new("UICorner", f).CornerRadius = UDim.new(0, 5)

    local l = Instance.new("TextLabel", f)
    l.Text = "  " .. label; l.Size = UDim2.new(0.72, 0, 1, 0)
    l.BackgroundTransparency = 1; l.TextColor3 = Color3.fromRGB(200, 210, 160)
    l.Font = Enum.Font.GothamSemibold; l.TextSize = 12
    l.TextXAlignment = Enum.TextXAlignment.Left

    local dot = Instance.new("Frame", f)
    dot.Size = UDim2.new(0, 12, 0, 12)
    dot.Position = UDim2.new(0.88, 0, 0.5, -6)
    dot.BackgroundColor3 = toggled and Color3.fromRGB(0, 200, 100) or Color3.fromRGB(180, 60, 60)
    dot.BorderSizePixel = 0
    Instance.new("UICorner", dot).CornerRadius = UDim.new(1, 0)

    local btn = Instance.new("TextButton", f)
    btn.Size = UDim2.new(1, 0, 1, 0)
    btn.BackgroundTransparency = 1; btn.Text = ""

    btn.MouseButton1Click:Connect(function()
        if onToggle then
            onToggle(not config[configKey], dot)
        else
            config[configKey] = not config[configKey]
            dot.BackgroundColor3 = config[configKey] and Color3.fromRGB(0, 200, 100) or Color3.fromRGB(180, 60, 60)
        end
    end)

    return btn, dot
end

-- ═══════════════════════════════════════════════════════════════
-- BUILD TOGGLES
-- ═══════════════════════════════════════════════════════════════
Section("AUTO INCOME & CASH")
Toggle("Auto Collect Money", "autoIncome")
Toggle("Auto Phone Offers", "phoneAutoAccept")

Section("AUTO BUY INFRASTRUCTURE")
Toggle("Auto Buy Buttons", "autoBuy", function(state, dot)
    dot.BackgroundColor3 = state and Color3.fromRGB(0, 200, 100) or Color3.fromRGB(180, 60, 60)
    toggleAutoBuySystem(state)
end)

Section("AUTO UPGRADES")
Toggle("Upgrade Lemon Stand", "upgradeStand")
Toggle("Upgrade Lemon Depot", "upgradeDepot")
Toggle("Upgrade Lemon Dash", "upgradeDash")
Toggle("Upgrade Lemon Trading", "upgradeTrading")
Toggle("Upgrade Lemon Labs", "upgradeLabs")
Toggle("Upgrade Lemon Robotics", "upgradeRobotics")

Section("AUTO FRUIT COLLECTION")
Toggle("Auto Collect Fruit (TP)", "autoPickFruit", function(state, dot)
    dot.BackgroundColor3 = state and Color3.fromRGB(0, 200, 100) or Color3.fromRGB(180, 60, 60)
    toggleFruitSystem(state)
end)

Section("MISC")
Toggle("Noclip", "noclip")
Toggle("Infinite Jump", "infJump")

-- ═══════════════════════════════════════════════════════════════
-- ENGINE 1: AUTO INCOME (WakeIncomeStream + CashDrops)
-- Remotes reais: WakeIncomeStream:InvokeServer("LemonStand") etc
-- CashDrops: workspace.CashDrops children chamados "CashDrop"
-- ═══════════════════════════════════════════════════════════════
task.spawn(function()
    while true do
        if not ScreenGui or not ScreenGui.Parent then break end
        if config.autoIncome then
            pcall(function()
                -- Re-detect tycoon if lost
                if not MyTycoon or not MyTycoon.Parent then
                    FindMyTycoon()
                end

                local incomeRemote = GetIncomeRemote()
                if incomeRemote and incomeRemote:IsA("RemoteFunction") then
                    task.spawn(function() pcall(function() incomeRemote:InvokeServer("LemonStand") end) end)
                    task.spawn(function() pcall(function() incomeRemote:InvokeServer("LemonDepot") end) end)
                    task.spawn(function() pcall(function() incomeRemote:InvokeServer("LemonDash") end) end)
                    task.spawn(function() pcall(function() incomeRemote:InvokeServer("LemonTrading") end) end)
                    task.spawn(function() pcall(function() incomeRemote:InvokeServer("LemonLabs") end) end)
                    task.spawn(function() pcall(function() incomeRemote:InvokeServer("LemonRobotics") end) end)
                elseif incomeRemote and incomeRemote:IsA("RemoteEvent") then
                    task.spawn(function() pcall(function() incomeRemote:FireServer("LemonStand") end) end)
                    task.spawn(function() pcall(function() incomeRemote:FireServer("LemonDepot") end) end)
                    task.spawn(function() pcall(function() incomeRemote:FireServer("LemonDash") end) end)
                    task.spawn(function() pcall(function() incomeRemote:FireServer("LemonTrading") end) end)
                    task.spawn(function() pcall(function() incomeRemote:FireServer("LemonLabs") end) end)
                    task.spawn(function() pcall(function() incomeRemote:FireServer("LemonRobotics") end) end)
                end

                -- Collect CashDrops (BaseParts in workspace.CashDrops)
                local rootPart = LocalPlayer.Character and (
                    LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                    or LocalPlayer.Character:FindFirstChild("UpperTorso")
                )
                local cashDropsFolder = workspace:FindFirstChild("CashDrops")

                if rootPart and cashDropsFolder and firetouchinterest then
                    for _, cashDrop in ipairs(cashDropsFolder:GetChildren()) do
                        if cashDrop.Name == "CashDrop" and cashDrop:IsA("BasePart") then
                            firetouchinterest(rootPart, cashDrop, 0)
                            task.wait()
                            firetouchinterest(rootPart, cashDrop, 1)
                        end
                    end
                end
            end)
        end
        task.wait(config.incomeDelay)
    end
end)

-- ═══════════════════════════════════════════════════════════════
-- ENGINE 2: AUTO BUY INFRASTRUCTURE
-- Remote real: Purchases/{ButtonModel}/Purchase:InvokeServer(false)
-- Condição: ButtonModel tem Part visível (Transparency < 1)
-- ═══════════════════════════════════════════════════════════════
local activeBuyThread = nil

shared._SL_KillAutoBuy = function()
    config.autoBuy = false
    if activeBuyThread then
        pcall(task.cancel, activeBuyThread)
        activeBuyThread = nil
    end
end

function toggleAutoBuySystem(state)
    config.autoBuy = state
    if state then
        local purchases = GetPurchasesFolder()
        if not purchases then return end

        activeBuyThread = task.spawn(function()
            while config.autoBuy do
                if not ScreenGui or not ScreenGui.Parent then break end
                pcall(function()
                    local currentPurchases = GetPurchasesFolder()
                    if not currentPurchases then return end

                    for _, obj in ipairs(currentPurchases:GetDescendants()) do
                        if not config.autoBuy then break end

                        if obj and obj.Name == "Purchase" and (obj:IsA("RemoteFunction") or obj:IsA("RemoteEvent")) then
                            local buttonModel = obj.Parent
                            if buttonModel then
                                -- Find the visible part (Head, Pad, Button, or any BasePart)
                                local targetPart = buttonModel:FindFirstChild("Head")
                                    or buttonModel:FindFirstChild("Pad")
                                    or buttonModel:FindFirstChild("Button")
                                    or buttonModel:FindFirstChildOfClass("BasePart")

                                if targetPart and targetPart:IsA("BasePart") and targetPart.Transparency < 1 then
                                    pcall(function()
                                        if obj:IsA("RemoteFunction") then
                                            obj:InvokeServer(false)
                                        else
                                            obj:FireServer(false)
                                        end
                                    end)
                                end
                            end
                        end
                    end
                end)
                task.wait(0.15)
            end
        end)
    else
        shared._SL_KillAutoBuy()
    end
end

-- ═══════════════════════════════════════════════════════════════
-- ENGINE 3: AUTO PHONE OFFERS
-- Remote real: Remotes/PhoneOffer:FireServer("Accept")
-- Também: firesignal(PhoneOffer.OnClientEvent, true)
-- E: firesignal(SpecialIncome.OnClientEvent, "PhoneOffer", 19.68505631765)
-- ═══════════════════════════════════════════════════════════════
task.spawn(function()
    while true do
        if not ScreenGui or not ScreenGui.Parent then break end
        if config.phoneAutoAccept then
            pcall(function()
                local rf = GetRemotesFolder()
                if not rf then return end

                local phoneOffer = rf:FindFirstChild("PhoneOffer")
                local specialIncome = rf:FindFirstChild("SpecialIncome")
                local fire = firesignal or (syn and syn.firesignal)

                -- Method 1: firesignal (most reliable for phone accept)
                if fire then
                    if phoneOffer then
                        pcall(function() fire(phoneOffer.OnClientEvent, true) end)
                    end
                    if specialIncome then
                        pcall(function() fire(specialIncome.OnClientEvent, "PhoneOffer", 19.68505631765) end)
                    end
                end

                -- Method 2: FireServer/InvokeServer
                if phoneOffer then
                    if phoneOffer:IsA("RemoteEvent") then
                        pcall(function() phoneOffer:FireServer("Accept") end)
                        pcall(function() phoneOffer:FireServer(true) end)
                    elseif phoneOffer:IsA("RemoteFunction") then
                        pcall(function() phoneOffer:InvokeServer("Accept") end)
                    end
                end
            end)
        end
        task.wait(0.5)
    end
end)

-- ═══════════════════════════════════════════════════════════════
-- ENGINE 4: AUTO UPGRADE MACHINES
-- Estrutura real: Purchases/Lemon Stand/Lemon Stand/Lemon Stand/Upgrade
-- InvokeServer(1) para Stand/Depot/Dash/Trading
-- InvokeServer(5) para Labs
-- InvokeServer(25) para Robotics
-- ═══════════════════════════════════════════════════════════════
local function TryUpgrade(machineName, invokeArg)
    local purchases = GetPurchasesFolder()
    if not purchases then return end

    local folder = purchases:FindFirstChild(machineName)
    if not folder then return end

    -- Navigate: MachineName/MachineName/MachineName/Upgrade
    local model = folder:FindFirstChild(machineName)
    if not model then return end
    local subModel = model:FindFirstChild(machineName)
    if not subModel then
        -- try direct Upgrade on model
        local upgrade = model:FindFirstChild("Upgrade")
        if upgrade then
            pcall(function() upgrade:InvokeServer(invokeArg) end)
        end
        return
    end
    local upgrade = subModel:FindFirstChild("Upgrade")
    if upgrade then
        pcall(function() upgrade:InvokeServer(invokeArg) end)
    end
end

task.spawn(function()
    while true do
        if not ScreenGui or not ScreenGui.Parent then break end
        pcall(function()
            if config.upgradeStand then
                TryUpgrade("Lemon Stand", 1)
                task.wait(0.05)
            end
            if config.upgradeDepot then
                TryUpgrade("Lemon Depot", 1)
                task.wait(0.05)
            end
            if config.upgradeDash then
                -- Try both naming conventions
                TryUpgrade("Lemon Dash", 1)
                TryUpgrade("LemonDash", 1)
                task.wait(0.05)
            end
            if config.upgradeTrading then
                TryUpgrade("Lemon Trading", 1)
                task.wait(0.05)
            end
            if config.upgradeLabs then
                TryUpgrade("Lemon Labs", 5)
                task.wait(0.05)
            end
            if config.upgradeRobotics then
                TryUpgrade("Lemon Robotics", 25)
                task.wait(0.05)
            end
        end)
        task.wait(config.upgradeDelay)
    end
end)

-- ═══════════════════════════════════════════════════════════════
-- ENGINE 5: AUTO FRUIT COLLECTION (TP + ClickDetector/ProximityPrompt)
-- Path filter: só coleta se fullPath contém "constant" E "trees"
-- Ignora: purchases, buttons, multiplier, billboard
-- ═══════════════════════════════════════════════════════════════
local activeFruitThread = nil

shared._SL_KillFruit = function()
    config.autoPickFruit = false
    if activeFruitThread then
        pcall(task.cancel, activeFruitThread)
        activeFruitThread = nil
    end
end

function toggleFruitSystem(state)
    config.autoPickFruit = state
    if state then
        activeFruitThread = task.spawn(function()
            while config.autoPickFruit do
                if not ScreenGui or not ScreenGui.Parent then break end
                pcall(function()
                    local character = LocalPlayer.Character
                    local rootPart = character and (
                        character:FindFirstChild("HumanoidRootPart")
                        or character:FindFirstChild("UpperTorso")
                    )
                    if not rootPart then return end

                    for _, fruitObj in ipairs(workspace:GetDescendants()) do
                        if not config.autoPickFruit then break end

                        local objName = string.lower(fruitObj.Name)
                        local parentName = fruitObj.Parent and string.lower(fruitObj.Parent.Name) or ""
                        local grandName = (fruitObj.Parent and fruitObj.Parent.Parent) and string.lower(fruitObj.Parent.Parent.Name) or ""
                        local fullPath = string.lower(fruitObj:GetFullName())

                        -- Check if it's a fruit/lemon/tree related object
                        local isValidHarvestable = (
                            string.find(objName, "fruit") or string.find(parentName, "fruit") or string.find(grandName, "fruit") or
                            string.find(objName, "lemon") or string.find(parentName, "lemon") or string.find(grandName, "lemon") or
                            string.find(objName, "drop") or string.find(parentName, "drop") or
                            string.find(objName, "tree") or string.find(parentName, "tree")
                        )

                        if isValidHarvestable then
                            -- STRICT FILTER: Must be under Constant/Trees path
                            if not string.find(fullPath, "constant") or not string.find(fullPath, "trees") then
                                continue
                            end

                            -- Skip UI/shop elements
                            if string.find(fullPath, "purchases") or string.find(fullPath, "buttons")
                                or string.find(fullPath, "multiplier") or string.find(fullPath, "billboard") then
                                continue
                            end

                            local targetPos = nil
                            local triggerInstance = nil

                            if fruitObj:IsA("ClickDetector") or fruitObj:IsA("ProximityPrompt") then
                                triggerInstance = fruitObj
                                if fruitObj.Parent:IsA("BasePart") then
                                    targetPos = fruitObj.Parent.CFrame
                                end
                            elseif fruitObj:IsA("BasePart") then
                                local cd = fruitObj:FindFirstChildOfClass("ClickDetector")
                                local pp = fruitObj:FindFirstChildOfClass("ProximityPrompt")
                                if cd or pp then
                                    triggerInstance = cd or pp
                                    targetPos = fruitObj.CFrame
                                end
                            end

                            if targetPos and triggerInstance then
                                local savedCFrame = rootPart.CFrame

                                -- TP to fruit
                                rootPart.CFrame = targetPos
                                task.wait(0.08)

                                -- Interact
                                if triggerInstance:IsA("ClickDetector") then
                                    fireclickdetector(triggerInstance)
                                elseif triggerInstance:IsA("ProximityPrompt") then
                                    fireproximityprompt(triggerInstance)
                                end

                                -- Wait for collection
                                if fruitObj.Parent and triggerInstance.Parent then
                                    task.wait(0.06)
                                end

                                task.wait(0.08)
                                -- TP back
                                rootPart.CFrame = savedCFrame
                                task.wait(0.1)
                            end
                        end
                    end
                end)
                task.wait(0.5)
            end
        end)
    else
        shared._SL_KillFruit()
    end
end

-- ═══════════════════════════════════════════════════════════════
-- ENGINE 6: MISC (Speed, Noclip, InfJump, Anti-AFK)
-- ═══════════════════════════════════════════════════════════════

-- Speed
task.spawn(function()
    while true do
        if not ScreenGui or not ScreenGui.Parent then break end
        pcall(function()
            local h = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
            if h and h.WalkSpeed ~= config.walkSpeed then
                h.WalkSpeed = config.walkSpeed
            end
        end)
        task.wait(0.1)
    end
end)

-- Noclip
RunService.Stepped:Connect(function()
    if config.noclip and LocalPlayer.Character then
        for _, p in ipairs(LocalPlayer.Character:GetDescendants()) do
            if p:IsA("BasePart") then p.CanCollide = false end
        end
    end
end)

-- Infinite Jump
UserInputService.JumpRequest:Connect(function()
    if config.infJump and LocalPlayer.Character then
        local h = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if h then h:ChangeState(Enum.HumanoidStateType.Jumping) end
    end
end)

-- Anti-AFK
if config.antiAfk then
    local VirtualUser = game:GetService("VirtualUser")
    task.spawn(function()
        while true do
            if not ScreenGui or not ScreenGui.Parent then break end
            pcall(function()
                VirtualUser:CaptureController()
                VirtualUser:ClickButton2(Vector2.new(0, 0))
            end)
            task.wait(math.random(15, 30))
        end
    end)
end

-- ═══════════════════════════════════════════════════════════════
-- KEYBIND: Right Ctrl toggle GUI
-- ═══════════════════════════════════════════════════════════════
UserInputService.InputBegan:Connect(function(input, gpe)
    if not gpe and input.KeyCode == Enum.KeyCode.RightControl then
        MainFrame.Visible = not MainFrame.Visible
    end
end)

-- ═══════════════════════════════════════════════════════════════
-- READY
-- ═══════════════════════════════════════════════════════════════
pcall(function()
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "🍋 Sell Lemons v3.0",
        Text = "Script loaded! Tycoon: " .. (MyTycoon and MyTycoon.Name or "Not found") .. "\nToggle: RCtrl",
        Duration = 5,
    })
end)

print("═══════════════════════════════════════")
print("  🍋 SELL LEMONS OP v3.0 — LOADED")
print("  Tycoon: " .. (MyTycoon and MyTycoon.Name or "Scanning..."))
print("  Remotes: " .. (GetIncomeRemote() and "✓ Found" or "✗ Not found"))
print("  Purchases: " .. (GetPurchasesFolder() and "✓ Found" or "✗ Not found"))
print("  Toggle GUI: Right Ctrl")
print("═══════════════════════════════════════")
