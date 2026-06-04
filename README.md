-- ========================================
-- REI HUB | BLOX FRUITS
-- Script com Quests SEA 1, 2 e 3
-- ========================================

local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")
local VirtualInputManager = game:GetService("VirtualInputManager")
local VirtualUser = game:GetService("VirtualUser")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")

-- ========================================
-- DETECTAR MUNDO
-- ========================================
local placeId = game.PlaceId
local World1 = placeId == 2753915549 or placeId == 85211729168715
local World2 = placeId == 4442272183 or placeId == 79091703265657
local World3 = placeId == 7449423635 or placeId == 100117331123089

if not World1 and not World2 and not World3 then
    Player:Kick("Mundo não suportado!")
end

-- ========================================
-- CONFIGURAÇÕES
-- ========================================
_G.ReiHub = {
    AutoFarm = false,
    FarmMode = "Level",
    SelectWeapon = "Melee",
    AutoBoss = false,
    SelectBoss = "",
    AutoAcceptQuest = false,
    AutoStats = false,
    StatType = "Melee",
    AutoHaki = false,
    AutoKen = false,
    AutoCollectChest = false,
    SpeedEnabled = false,
    SpeedValue = 50,
    NoClip = false,
    BringRange = 235,
    MobHeight = 20,
}

-- ========================================
-- SALVAR CONFIG
-- ========================================
local FolderName = "Rei Hub"
local FileName = "Settings.json"
local FullPath = FolderName .. "/" .. FileName

if makefolder and not isfolder(FolderName) then makefolder(FolderName) end

function SaveSettings()
    if not writefile then return end
    pcall(function()
        writefile(FullPath, game:GetService("HttpService"):JSONEncode(_G.ReiHub))
    end)
end

function LoadSettings()
    if isfile and isfile(FullPath) then
        pcall(function()
            local data = game:GetService("HttpService"):JSONDecode(readfile(FullPath))
            for k, v in pairs(data) do _G.ReiHub[k] = v end
        end)
    end
end
LoadSettings()

-- ========================================
-- FUNÇÕES AUXILIARES
-- ========================================
local function GetHRP()
    local char = Player.Character
    return char and char:FindFirstChild("HumanoidRootPart")
end

local function EquipWeapon(weaponName)
    local char = Player.Character
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    if not hum then return end
    local tool = Player.Backpack:FindFirstChild(weaponName) or char:FindFirstChild(weaponName)
    if tool and tool.Parent ~= char then
        hum:EquipTool(tool)
    end
end

local function TP(pos)
    local hrp = GetHRP()
    if hrp then hrp.CFrame = pos end
end

local function IsAlive(model)
    local hum = model and model:FindFirstChild("Humanoid")
    return hum and hum.Health > 0
end

-- ========================================
-- AUTO KEN
-- ========================================
local function HasKen()
    local char = Player.Character
    return char and char:FindFirstChild("HasKen")
end

task.spawn(function()
    while task.wait(0.3) do
        if _G.ReiHub.AutoKen and not HasKen() then
            pcall(function() ReplicatedStorage.Remotes.CommE:FireServer("Ken", true) end)
        end
    end
end)

-- ========================================
-- AUTO HAKI
-- ========================================
task.spawn(function()
    while task.wait(1) do
        if _G.ReiHub.AutoHaki then
            pcall(function()
                if not Player.Character:FindFirstChild("HasBuso") then
                    ReplicatedStorage.Remotes.CommF_:InvokeServer("Buso")
                end
            end)
        end
    end
end)

-- ========================================
-- AUTO STATS
-- ========================================
task.spawn(function()
    while task.wait(0.5) do
        if _G.ReiHub.AutoStats and Player.Data.Points.Value > 0 then
            pcall(function()
                local statMap = { Melee = "Melee", Defense = "Defense", Sword = "Sword", Gun = "Gun", Devil = "Demon Fruit" }
                ReplicatedStorage.Remotes.CommF_:InvokeServer("AddPoint", statMap[_G.ReiHub.StatType], 1)
            end)
        end
    end
end)

-- ========================================
-- AUTO BRING
-- ========================================
local BringPart = Instance.new("Part", Workspace)
BringPart.Name = "ReiHub_Bring"
BringPart.Size = Vector3.new(1, 1, 1)
BringPart.Anchored = true
BringPart.CanCollide = false
BringPart.Transparency = 1

local function BringEnemy(targetPos)
    if not _G.ReiHub.AutoFarm then return end
    for _, mob in pairs(Workspace.Enemies:GetChildren()) do
        local hum = mob:FindFirstChild("Humanoid")
        local root = mob:FindFirstChild("HumanoidRootPart")
        if hum and root and hum.Health > 0 then
            if (root.Position - targetPos).Magnitude <= _G.ReiHub.BringRange then
                local tween = TweenService:Create(root, TweenInfo.new(0.35), {CFrame = CFrame.new(targetPos)})
                tween:Play()
            end
        end
    end
end

local function KillMob(mob)
    if not mob or not IsAlive(mob) then return end
    local root = mob:FindFirstChild("HumanoidRootPart")
    if not root then return end
    if not mob:GetAttribute("Locked") then mob:SetAttribute("Locked", root.CFrame) end
    BringEnemy((mob:GetAttribute("Locked")).Position)
    EquipWeapon(_G.ReiHub.SelectWeapon)
    TP(root.CFrame * CFrame.new(0, _G.ReiHub.MobHeight, 0))
    local tool = Player.Character and Player.Character:FindFirstChildOfClass("Tool")
    if tool then
        VirtualUser:CaptureController()
        VirtualUser:Button1Down(Vector2.new(1280, 672))
        task.wait(0.1)
        VirtualUser:Button1Up(Vector2.new(1280, 672))
    end
end

-- ========================================
-- QUESTS SEA 1
-- ========================================
local QuestsSea1 = {
    ["Bandit"] = { Lv = 1, Quest = "BanditQuest1", Qty = 1, Npc = CFrame.new(1045.96, 27.00, 1560.82), Mob = CFrame.new(1120, 27, 1590) },
    ["Monkey"] = { Lv = 10, Quest = "JungleQuest", Qty = 1, Npc = CFrame.new(-1598.08, 35.55, 153.37), Mob = CFrame.new(-1448.51, 67.85, 11.46) },
    ["Gorilla"] = { Lv = 15, Quest = "JungleQuest", Qty = 2, Npc = CFrame.new(-1598.08, 35.55, 153.37), Mob = CFrame.new(-1129.88, 40.46, -525.42) },
    ["Pirate"] = { Lv = 30, Quest = "BuggyQuest1", Qty = 1, Npc = CFrame.new(-1141.07, 4.10, 3831.54), Mob = CFrame.new(-1103.51, 13.75, 3896.09) },
    ["Brute"] = { Lv = 40, Quest = "BuggyQuest1", Qty = 2, Npc = CFrame.new(-1141.07, 4.10, 3831.54), Mob = CFrame.new(-1140.08, 14.80, 4322.92) },
    ["Desert Bandit"] = { Lv = 60, Quest = "DesertQuest", Qty = 1, Npc = CFrame.new(894.48, 5.14, 4392.43), Mob = CFrame.new(924.79, 6.44, 4481.58) },
    ["Desert Officer"] = { Lv = 75, Quest = "DesertQuest", Qty = 2, Npc = CFrame.new(894.48, 5.14, 4392.43), Mob = CFrame.new(1608.28, 8.61, 4371.00) },
    ["Snow Bandit"] = { Lv = 90, Quest = "SnowQuest", Qty = 1, Npc = CFrame.new(1389.74, 88.15, -1298.90), Mob = CFrame.new(1354.34, 87.27, -1393.94) },
    ["Snowman"] = { Lv = 100, Quest = "SnowQuest", Qty = 2, Npc = CFrame.new(1389.74, 88.15, -1298.90), Mob = CFrame.new(1201.64, 144.57, -1550.06) },
    ["Chief Petty Officer"] = { Lv = 120, Quest = "MarineQuest2", Qty = 1, Npc = CFrame.new(-5039.58, 27.35, 4324.68), Mob = CFrame.new(-4881.23, 22.65, 4273.75) },
    ["Sky Bandit"] = { Lv = 150, Quest = "SkyQuest", Qty = 1, Npc = CFrame.new(-4839.53, 716.36, -2619.44), Mob = CFrame.new(-4953.20, 295.74, -2899.22) },
    ["Dark Master"] = { Lv = 175, Quest = "SkyQuest", Qty = 2, Npc = CFrame.new(-4839.53, 716.36, -2619.44), Mob = CFrame.new(-5259.84, 391.39, -2229.03) },
    ["Prisoner"] = { Lv = 190, Quest = "PrisonerQuest", Qty = 1, Npc = CFrame.new(5308.93, 1.65, 475.12), Mob = CFrame.new(5098.97, -0.32, 474.23) },
    ["Dangerous Prisoner"] = { Lv = 210, Quest = "PrisonerQuest", Qty = 2, Npc = CFrame.new(5308.93, 1.65, 475.12), Mob = CFrame.new(5654.56, 15.63, 866.29) },
    ["Toga Warrior"] = { Lv = 250, Quest = "ColosseumQuest", Qty = 1, Npc = CFrame.new(-1580.04, 6.35, -2986.47), Mob = CFrame.new(-1820.21, 51.68, -2740.66) },
    ["Gladiator"] = { Lv = 275, Quest = "ColosseumQuest", Qty = 2, Npc = CFrame.new(-1580.04, 6.35, -2986.47), Mob = CFrame.new(-1292.83, 56.38, -3339.03) },
    ["Military Soldier"] = { Lv = 300, Quest = "MagmaQuest", Qty = 1, Npc = CFrame.new(-5313.37, 10.95, 8515.29), Mob = CFrame.new(-5411.16, 11.08, 8454.29) },
    ["Military Spy"] = { Lv = 325, Quest = "MagmaQuest", Qty = 2, Npc = CFrame.new(-5313.37, 10.95, 8515.29), Mob = CFrame.new(-5802.86, 86.26, 8828.85) },
    ["Fishman Warrior"] = { Lv = 375, Quest = "FishmanQuest", Qty = 1, Npc = CFrame.new(61122.65, 18.49, 1569.39), Mob = CFrame.new(60878.30, 18.48, 1543.75) },
    ["Fishman Commando"] = { Lv = 400, Quest = "FishmanQuest", Qty = 2, Npc = CFrame.new(61122.65, 18.49, 1569.39), Mob = CFrame.new(61922.63, 18.48, 1493.93) },
    ["God's Guard"] = { Lv = 450, Quest = "SkyExp1Quest", Qty = 1, Npc = CFrame.new(-4721.88, 843.87, -1949.96), Mob = CFrame.new(-4710.04, 845.27, -1927.30) },
    ["Shanda"] = { Lv = 475, Quest = "SkyExp1Quest", Qty = 2, Npc = CFrame.new(-7859.09, 5544.19, -381.47), Mob = CFrame.new(-7678.48, 5566.40, -497.21) },
    ["Royal Squad"] = { Lv = 525, Quest = "SkyExp2Quest", Qty = 1, Npc = CFrame.new(-7906.81, 5634.66, -1411.99), Mob = CFrame.new(-7624.25, 5658.13, -1467.35) },
    ["Royal Soldier"] = { Lv = 550, Quest = "SkyExp2Quest", Qty = 2, Npc = CFrame.new(-7906.81, 5634.66, -1411.99), Mob = CFrame.new(-7836.75, 5645.66, -1790.62) },
    ["Galley Pirate"] = { Lv = 625, Quest = "FountainQuest", Qty = 1, Npc = CFrame.new(5259.81, 37.35, 4050.02), Mob = CFrame.new(5551.02, 78.90, 3930.41) },
    ["Galley Captain"] = { Lv = 650, Quest = "FountainQuest", Qty = 2, Npc = CFrame.new(5259.81, 37.35, 4050.02), Mob = CFrame.new(5441.95, 42.50, 4950.09) },
}

-- ========================================
-- QUESTS SEA 2
-- ========================================
local QuestsSea2 = {
    ["Raider"] = { Lv = 700, Quest = "Area1Quest", Qty = 1, Npc = CFrame.new(-429.54, 71.77, 1836.18), Mob = CFrame.new(-728.32, 52.77, 2345.77) },
    ["Mercenary"] = { Lv = 725, Quest = "Area1Quest", Qty = 2, Npc = CFrame.new(-429.54, 71.77, 1836.18), Mob = CFrame.new(-1004.32, 80.15, 1424.61) },
    ["Swan Pirate"] = { Lv = 775, Quest = "Area2Quest", Qty = 1, Npc = CFrame.new(638.43, 71.76, 918.28), Mob = CFrame.new(1068.66, 137.61, 1322.10) },
    ["Factory Staff"] = { Lv = 800, Quest = "Area2Quest", Qty = 2, Npc = CFrame.new(632.69, 73.10, 918.66), Mob = CFrame.new(73.07, 81.86, -27.47) },
    ["Marine Lieutenant"] = { Lv = 875, Quest = "MarineQuest3", Qty = 1, Npc = CFrame.new(-2440.79, 71.71, -3216.06), Mob = CFrame.new(-2821.37, 75.89, -3070.08) },
    ["Marine Captain"] = { Lv = 900, Quest = "MarineQuest3", Qty = 2, Npc = CFrame.new(-2440.79, 71.71, -3216.06), Mob = CFrame.new(-1861.23, 80.17, -3254.69) },
    ["Zombie"] = { Lv = 950, Quest = "ZombieQuest", Qty = 1, Npc = CFrame.new(-5497.06, 47.59, -795.23), Mob = CFrame.new(-5657.77, 78.96, -928.68) },
    ["Vampire"] = { Lv = 975, Quest = "ZombieQuest", Qty = 2, Npc = CFrame.new(-5497.06, 47.59, -795.23), Mob = CFrame.new(-6037.66, 32.18, -1340.65) },
    ["Snow Trooper"] = { Lv = 1000, Quest = "SnowMountainQuest", Qty = 1, Npc = CFrame.new(609.85, 400.11, -5372.25), Mob = CFrame.new(549.14, 427.38, -5563.69) },
    ["Winter Warrior"] = { Lv = 1050, Quest = "SnowMountainQuest", Qty = 2, Npc = CFrame.new(609.85, 400.11, -5372.25), Mob = CFrame.new(1142.74, 475.63, -5199.41) },
    ["Lab Subordinate"] = { Lv = 1100, Quest = "IceSideQuest", Qty = 1, Npc = CFrame.new(-6064.06, 15.24, -4902.97), Mob = CFrame.new(-5707.47, 15.95, -4513.39) },
    ["Horned Warrior"] = { Lv = 1125, Quest = "IceSideQuest", Qty = 2, Npc = CFrame.new(-6064.06, 15.24, -4902.97), Mob = CFrame.new(-6341.36, 15.95, -5723.16) },
    ["Magma Ninja"] = { Lv = 1175, Quest = "FireSideQuest", Qty = 1, Npc = CFrame.new(-5428.03, 15.06, -5299.43), Mob = CFrame.new(-5449.67, 76.65, -5808.20) },
    ["Lava Pirate"] = { Lv = 1200, Quest = "FireSideQuest", Qty = 2, Npc = CFrame.new(-5428.03, 15.06, -5299.43), Mob = CFrame.new(-5213.33, 49.73, -4701.45) },
    ["Ship Deckhand"] = { Lv = 1250, Quest = "ShipQuest1", Qty = 1, Npc = CFrame.new(1037.80, 125.09, 32911.60), Mob = CFrame.new(1212.01, 150.79, 33059.24) },
    ["Ship Engineer"] = { Lv = 1275, Quest = "ShipQuest1", Qty = 2, Npc = CFrame.new(1037.80, 125.09, 32911.60), Mob = CFrame.new(919.47, 43.54, 32779.96) },
    ["Ship Steward"] = { Lv = 1300, Quest = "ShipQuest2", Qty = 1, Npc = CFrame.new(968.80, 125.09, 33244.12), Mob = CFrame.new(919.43, 129.55, 33436.03) },
    ["Ship Officer"] = { Lv = 1325, Quest = "ShipQuest2", Qty = 2, Npc = CFrame.new(968.80, 125.09, 33244.12), Mob = CFrame.new(1036.01, 181.43, 33315.72) },
    ["Arctic Warrior"] = { Lv = 1350, Quest = "FrostQuest", Qty = 1, Npc = CFrame.new(5667.65, 26.79, -6486.08), Mob = CFrame.new(5966.24, 62.97, -6179.38) },
    ["Snow Lurker"] = { Lv = 1375, Quest = "FrostQuest", Qty = 2, Npc = CFrame.new(5667.65, 26.79, -6486.08), Mob = CFrame.new(5407.07, 69.19, -6880.88) },
    ["Sea Soldier"] = { Lv = 1425, Quest = "ForgottenQuest", Qty = 1, Npc = CFrame.new(-3054.44, 235.54, -10142.81), Mob = CFrame.new(-3028.22, 64.67, -9775.42) },
    ["Water Fighter"] = { Lv = 1450, Quest = "ForgottenQuest", Qty = 2, Npc = CFrame.new(-3054.44, 235.54, -10142.81), Mob = CFrame.new(-3352.90, 285.01, -10534.84) },
}

-- ========================================
-- QUESTS SEA 3
-- ========================================
local QuestsSea3 = {
    ["Pirate Millionaire"] = { Lv = 1500, Quest = "PiratePortQuest", Qty = 1, Npc = CFrame.new(-290.07, 42.90, 5581.59), Mob = CFrame.new(-246.00, 47.31, 5584.10) },
    ["Pistol Billionaire"] = { Lv = 1525, Quest = "PiratePortQuest", Qty = 2, Npc = CFrame.new(-290.07, 42.90, 5581.59), Mob = CFrame.new(-187.33, 86.24, 6013.51) },
    ["Dragon Crew Warrior"] = { Lv = 1575, Quest = "DragonCrewQuest", Qty = 1, Npc = CFrame.new(6737.06, 127.41, -712.30), Mob = CFrame.new(6709.76, 52.34, -1139.02) },
    ["Dragon Crew Archer"] = { Lv = 1600, Quest = "DragonCrewQuest", Qty = 2, Npc = CFrame.new(6737.06, 127.41, -712.30), Mob = CFrame.new(6668.76, 481.37, 329.12) },
    ["Hydra Enforcer"] = { Lv = 1625, Quest = "VenomCrewQuest", Qty = 1, Npc = CFrame.new(5206.40, 1004.10, 748.35), Mob = CFrame.new(4547.11, 1003.10, 334.19) },
    ["Venomous Assailant"] = { Lv = 1650, Quest = "VenomCrewQuest", Qty = 2, Npc = CFrame.new(5206.40, 1004.10, 748.35), Mob = CFrame.new(4674.92, 1134.82, 996.30) },
    ["Marine Commodore"] = { Lv = 1700, Quest = "MarineTreeIsland", Qty = 1, Npc = CFrame.new(2180.54, 27.81, -6741.54), Mob = CFrame.new(2286.00, 73.13, -7159.80) },
    ["Marine Rear Admiral"] = { Lv = 1725, Quest = "MarineTreeIsland", Qty = 2, Npc = CFrame.new(2179.98, 28.73, -6740.05), Mob = CFrame.new(3656.77, 160.52, -7001.59) },
    ["Fishman Raider"] = { Lv = 1775, Quest = "DeepForestIsland3", Qty = 1, Npc = CFrame.new(-10581.65, 330.87, -8761.18), Mob = CFrame.new(-10407.52, 331.76, -8368.51) },
    ["Fishman Captain"] = { Lv = 1800, Quest = "DeepForestIsland3", Qty = 2, Npc = CFrame.new(-10581.65, 330.87, -8761.18), Mob = CFrame.new(-10994.70, 352.38, -9002.11) },
    ["Forest Pirate"] = { Lv = 1825, Quest = "DeepForestIsland", Qty = 1, Npc = CFrame.new(-13234.04, 331.48, -7625.40), Mob = CFrame.new(-13274.47, 332.37, -7769.58) },
    ["Mythological Pirate"] = { Lv = 1850, Quest = "DeepForestIsland", Qty = 2, Npc = CFrame.new(-13234.04, 331.48, -7625.40), Mob = CFrame.new(-13680.60, 501.08, -6991.18) },
    ["Jungle Pirate"] = { Lv = 1900, Quest = "DeepForestIsland2", Qty = 1, Npc = CFrame.new(-12680.38, 389.97, -9902.01), Mob = CFrame.new(-12256.16, 331.73, -10485.83) },
    ["Musketeer Pirate"] = { Lv = 1925, Quest = "DeepForestIsland2", Qty = 2, Npc = CFrame.new(-12680.38, 389.97, -9902.01), Mob = CFrame.new(-13457.90, 391.54, -9859.17) },
    ["Reborn Skeleton"] = { Lv = 1975, Quest = "HauntedQuest1", Qty = 1, Npc = CFrame.new(-9479.21, 141.21, 5566.09), Mob = CFrame.new(-8763.72, 165.72, 6159.86) },
    ["Living Zombie"] = { Lv = 2000, Quest = "HauntedQuest1", Qty = 2, Npc = CFrame.new(-9479.21, 141.21, 5566.09), Mob = CFrame.new(-10144.13, 138.62, 5838.08) },
    ["Demonic Soul"] = { Lv = 2025, Quest = "HauntedQuest2", Qty = 1, Npc = CFrame.new(-9516.99, 172.01, 6078.46), Mob = CFrame.new(-9505.87, 172.10, 6158.99) },
    ["Posessed Mummy"] = { Lv = 2050, Quest = "HauntedQuest2", Qty = 2, Npc = CFrame.new(-9516.99, 172.01, 6078.46), Mob = CFrame.new(-9582.02, 6.25, 6205.47) },
    ["Peanut Scout"] = { Lv = 2075, Quest = "NutsIslandQuest", Qty = 1, Npc = CFrame.new(-2104.39, 38.10, -10194.21), Mob = CFrame.new(-2143.24, 47.72, -10029.99) },
    ["Peanut President"] = { Lv = 2100, Quest = "NutsIslandQuest", Qty = 2, Npc = CFrame.new(-2104.39, 38.10, -10194.21), Mob = CFrame.new(-1859.35, 38.10, -10422.42) },
    ["Ice Cream Chef"] = { Lv = 2125, Quest = "IceCreamIslandQuest", Qty = 1, Npc = CFrame.new(-820.64, 65.81, -10965.79), Mob = CFrame.new(-872.24, 65.81, -10919.95) },
    ["Ice Cream Commander"] = { Lv = 2150, Quest = "IceCreamIslandQuest", Qty = 2, Npc = CFrame.new(-820.64, 65.81, -10965.79), Mob = CFrame.new(-558.06, 112.04, -11290.77) },
    ["Cookie Crafter"] = { Lv = 2200, Quest = "CakeQuest1", Qty = 1, Npc = CFrame.new(-2021.32, 37.79, -12028.72), Mob = CFrame.new(-2374.13, 37.79, -12125.30) },
    ["Cake Guard"] = { Lv = 2225, Quest = "CakeQuest1", Qty = 2, Npc = CFrame.new(-2021.32, 37.79, -12028.72), Mob = CFrame.new(-1598.30, 43.77, -12244.58) },
    ["Baking Staff"] = { Lv = 2250, Quest = "CakeQuest2", Qty = 1, Npc = CFrame.new(-1927.91, 37.79, -12842.53), Mob = CFrame.new(-1887.80, 77.61, -12998.35) },
    ["Head Baker"] = { Lv = 2275, Quest = "CakeQuest2", Qty = 2, Npc = CFrame.new(-1927.91, 37.79, -12842.53), Mob = CFrame.new(-2216.18, 82.88, -12869.29) },
    ["Cocoa Warrior"] = { Lv = 2300, Quest = "ChocQuest1", Qty = 1, Npc = CFrame.new(233.22, 29.87, -12201.23), Mob = CFrame.new(-21.55, 80.57, -12352.38) },
    ["Chocolate Bar Battler"] = { Lv = 2325, Quest = "ChocQuest1", Qty = 2, Npc = CFrame.new(233.22, 29.87, -12201.23), Mob = CFrame.new(582.59, 77.18, -12463.16) },
    ["Sweet Thief"] = { Lv = 2350, Quest = "ChocQuest2", Qty = 1, Npc = CFrame.new(150.50, 30.69, -12774.50), Mob = CFrame.new(165.18, 76.05, -12600.83) },
    ["Candy Rebel"] = { Lv = 2375, Quest = "ChocQuest2", Qty = 2, Npc = CFrame.new(150.50, 30.69, -12774.50), Mob = CFrame.new(134.86, 77.24, -12876.54) },
    ["Candy Pirate"] = { Lv = 2400, Quest = "CandyQuest1", Qty = 1, Npc = CFrame.new(-1150.04, 20.37, -14446.33), Mob = CFrame.new(-1310.50, 26.01, -14562.40) },
    ["Isle Outlaw"] = { Lv = 2450, Quest = "TikiQuest1", Qty = 1, Npc = CFrame.new(-16548.81, 55.60, -172.81), Mob = CFrame.new(-16479.90, 226.61, -300.31) },
    ["Island Boy"] = { Lv = 2475, Quest = "TikiQuest1", Qty = 2, Npc = CFrame.new(-16548.81, 55.60, -172.81), Mob = CFrame.new(-16849.39, 192.86, -150.78) },
    ["Sun-kissed Warrior"] = { Lv = 2500, Quest = "TikiQuest2", Qty = 1, Npc = CFrame.new(-16538, 55, 1049), Mob = CFrame.new(-16347, 64, 984) },
    ["Isle Champion"] = { Lv = 2525, Quest = "TikiQuest2", Qty = 2, Npc = CFrame.new(-16541.02, 57.30, 1051.46), Mob = CFrame.new(-16602.10, 130.38, 1087.24) },
    ["Serpent Hunter"] = { Lv = 2551, Quest = "TikiQuest3", Qty = 1, Npc = CFrame.new(-16668.03, 105.32, 1568.60), Mob = CFrame.new(-16645.64, 163.09, 1352.87) },
    ["Skull Slayer"] = { Lv = 2575, Quest = "TikiQuest3", Qty = 2, Npc = CFrame.new(-16668.03, 105.32, 1568.60), Mob = CFrame.new(-16709.49, 419.68, 1751.09) },
    ["Reef Bandit"] = { Lv = 2600, Quest = "SubmergedQuest1", Qty = 1, Npc = CFrame.new(10778.87, -2087.72, 9265.18), Mob = CFrame.new(11019.13, -2146.06, 9342.39) },
    ["Coral Pirate"] = { Lv = 2625, Quest = "SubmergedQuest1", Qty = 2, Npc = CFrame.new(10778.87, -2087.72, 9265.18), Mob = CFrame.new(10808.60, -2030.36, 9364.23) },
    ["Sea Chanter"] = { Lv = 2650, Quest = "SubmergedQuest2", Qty = 1, Npc = CFrame.new(10880.68, -2086.20, 10032.62), Mob = CFrame.new(10671.27, -2057.59, 10047.25) },
    ["Ocean Prophet"] = { Lv = 2675, Quest = "SubmergedQuest2", Qty = 2, Npc = CFrame.new(10880.68, -2086.20, 10032.62), Mob = CFrame.new(11008.51, -2007.72, 10223.07) },
    ["High Disciple"] = { Lv = 2700, Quest = "SubmergedQuest3", Qty = 1, Npc = CFrame.new(9640.08, -1992.44, 9613.65), Mob = CFrame.new(9750.41, -1966.93, 9753.36) },
    ["Grand Devotee"] = { Lv = 2725, Quest = "SubmergedQuest3", Qty = 2, Npc = CFrame.new(9640.08, -1992.44, 9613.65), Mob = CFrame.new(9611.70, -1993.47, 9882.68) },
}

-- ========================================
-- SELECIONAR QUEST POR LEVEL
-- ========================================
local function GetQuestByLevel()
    local level = Player.Data.Level.Value
    local quests = World1 and QuestsSea1 or (World2 and QuestsSea2 or QuestsSea3)
    local lastQuest = nil
    for _, quest in pairs(quests) do
        if level >= quest.Lv then
            lastQuest = quest
        end
    end
    return lastQuest
end

-- ========================================
-- AUTO FARM LOOP
-- ========================================
local CurrentMob = nil

task.spawn(function()
    while task.wait(0.3) do
        if not _G.ReiHub.AutoFarm or _G.ReiHub.FarmMode ~= "Level" then
            task.wait(1)
            continue
        end
        
        pcall(function()
            local hrp = GetHRP()
            if not hrp then return end
            
            local questData = GetQuestByLevel()
            if not questData then return end
            
            local questUI = Player.PlayerGui.Main.Quest
            local hasQuest = questUI and questUI.Visible
            
            -- Pegar quest
            if not hasQuest and _G.ReiHub.AutoAcceptQuest then
                TP(questData.Npc)
                if (hrp.Position - questData.Npc.Position).Magnitude <= 30 then
                    task.wait(1)
                    ReplicatedStorage.Remotes.CommF_:InvokeServer("StartQuest", questData.Quest, questData.Qty)
                end
                return
            end
            
            -- Teleportar se necessário para o Sea 2/3
            if World2 and questData.Lv >= 1250 then
                pcall(function()
                    if (hrp.Position - Vector3.new(923.21, 126.97, 32852.83)).Magnitude > 500 then
                        ReplicatedStorage.Remotes.CommF_:InvokeServer("requestEntrance", Vector3.new(923.21, 126.97, 32852.83))
                    end
                end)
            end
            
            if World3 and questData.Lv >= 2600 then
                pcall(function()
                    if (hrp.Position - Vector3.new(-16269.70, 25.22, 1373.65)).Magnitude > 500 then
                        ReplicatedStorage.Remotes.CommF_:InvokeServer("requestEntrance", Vector3.new(-16269.70, 25.22, 1373.65))
                        task.wait(2)
                        ReplicatedStorage.Modules.Net:FindFirstChild("RF/SubmarineWorkerSpeak"):InvokeServer("TravelToSubmergedIsland")
                    end
                end)
            end
            
            -- Procurar mob
            local closestMob = nil
            local closestDist = math.huge
            
            for _, mob in pairs(Workspace.Enemies:GetChildren()) do
                if IsAlive(mob) then
                    local mobName = mob.Name
                    local questMobName = questData.Name
                    if mobName == questMobName or (questMobName == "Pistol Billionaire" and mobName == "Pistol Billionaire") or (questMobName == "Dragon Crew Warrior" and mobName == "Dragon Crew Warrior") then
                        local root = mob:FindFirstChild("HumanoidRootPart")
                        if root then
                            local dist = (hrp.Position - root.Position).Magnitude
                            if dist < closestDist then
                                closestDist = dist
                                closestMob = mob
                            end
                        end
                    end
                end
            end
            
            if closestMob then
                KillMob(closestMob)
            else
                TP(questData.Mob)
            end
        end)
    end
end)

-- ========================================
-- AUTO BOSS (COMPLETO)
-- ========================================
local Bosses = {}

if World1 then
    Bosses = {
        ["The Gorilla King"] = { Quest = "JungleQuest", Qty = 3, Pos = CFrame.new(-1088.75, 8.13, -488.55), Npc = CFrame.new(-1601.65, 36.85, 153.38) },
        ["Bobby"] = { Quest = "BuggyQuest1", Qty = 3, Pos = CFrame.new(-1087.37, 46.94, 4040.14), Npc = CFrame.new(-1140.17, 4.75, 3827.40) },
        ["The Saw"] = { Pos = CFrame.new(-784.89, 72.42, 1603.58) },
        ["Yeti"] = { Quest = "SnowQuest", Qty = 3, Pos = CFrame.new(1218.79, 138.01, -1488.02), Npc = CFrame.new(1386.80, 87.27, -1298.35) },
        ["Vice Admiral"] = { Quest = "MarineQuest2", Qty = 2, Pos = CFrame.new(-5006.54, 88.03, 4353.16), Npc = CFrame.new(-5036.24, 28.67, 4324.56) },
        ["Saber Expert"] = { Pos = CFrame.new(-1458.89, 29.88, -50.63) },
        ["Magma Admiral"] = { Quest = "MagmaQuest", Qty = 3, Pos = CFrame.new(-5765.89, 82.92, 8718.30), Npc = CFrame.new(-5314.62, 12.26, 8517.27) },
        ["Fishman Lord"] = { Quest = "FishmanQuest", Qty = 3, Pos = CFrame.new(61260.15, 30.95, 1193.43), Npc = CFrame.new(61122.65, 18.49, 1569.39) },
        ["Wysper"] = { Quest = "SkyExp1Quest", Qty = 3, Pos = CFrame.new(-7866.13, 5576.43, -546.74), Npc = CFrame.new(-7861.94, 5545.51, -379.85) },
        ["Thunder God"] = { Quest = "SkyExp2Quest", Qty = 3, Pos = CFrame.new(-7994.98, 5761.02, -2088.64), Npc = CFrame.new(-7903.38, 5635.98, -1410.92) },
        ["Cyborg"] = { Quest = "FountainQuest", Qty = 3, Pos = CFrame.new(6094.02, 73.77, 3825.73), Npc = CFrame.new(5258.27, 38.52, 4050.04) },
        ["Ice Admiral"] = { Pos = CFrame.new(1266.08, 26.17, -1399.57) },
        ["Greybeard"] = { Pos = CFrame.new(-5081.34, 85.22, 4257.35) },
    }
elseif World2 then
    Bosses = {
        ["Diamond"] = { Quest = "Area1Quest", Qty = 3, Pos = CFrame.new(-1576.71, 198.59, 13.72), Npc = CFrame.new(-427.56, 73.31, 1835.42) },
        ["Jeremy"] = { Quest = "Area2Quest", Qty = 3, Pos = CFrame.new(2006.92, 448.95, 853.98), Npc = CFrame.new(636.79, 73.41, 918.00) },
        ["Fajita"] = { Quest = "MarineQuest3", Qty = 3, Pos = CFrame.new(-2172.73, 103.32, -4015.02), Npc = CFrame.new(-2441.98, 73.35, -3217.53) },
        ["Don Swan"] = { Pos = CFrame.new(2286.20, 15.17, 863.83) },
        ["Smoke Admiral"] = { Quest = "IceSideQuest", Qty = 3, Pos = CFrame.new(-5275.19, 20.75, -5260.66), Npc = CFrame.new(-5429.04, 15.97, -5297.96) },
        ["Awakened Ice Admiral"] = { Quest = "FrostQuest", Qty = 3, Pos = CFrame.new(6403.54, 340.29, -6894.55), Npc = CFrame.new(5668.97, 28.51, -6483.35) },
        ["Tide Keeper"] = { Quest = "ForgottenQuest", Qty = 3, Pos = CFrame.new(-3795.64, 105.88, -11421.30), Npc = CFrame.new(-3053.98, 237.18, -10145.03) },
        ["Darkbeard"] = { Pos = CFrame.new(3677.08, 62.75, -3144.83) },
        ["Cursed Captain"] = { Pos = CFrame.new(916.92, 181.09, 33422) },
        ["Order"] = { Pos = CFrame.new(-6217.20, 28.04, -5053.13) },
    }
elseif World3 then
    Bosses = {
        ["Stone"] = { Quest = "PiratePortQuest", Qty = 3, Pos = CFrame.new(-1027.65, 92.40, 6578.85), Npc = CFrame.new(-289.76, 43.81, 5579.93) },
        ["Hydra Leader"] = { Quest = "AmazonQuest2", Qty = 3, Pos = CFrame.new(5821.89, 1019.09, -73.71), Npc = CFrame.new(5821.89, 1019.09, -73.71) },
        ["Kilo Admiral"] = { Quest = "MarineTreeIsland", Qty = 3, Pos = CFrame.new(2764.22, 432.46, -7144.45), Npc = CFrame.new(2179.30, 28.73, -6739.97) },
        ["Captain Elephant"] = { Quest = "DeepForestIsland", Qty = 3, Pos = CFrame.new(-13376.75, 433.28, -8071.39), Npc = CFrame.new(-13232.68, 332.40, -7626.01) },
        ["Beautiful Pirate"] = { Quest = "DeepForestIsland2", Qty = 3, Pos = CFrame.new(5283.60, 22.56, -110.78), Npc = CFrame.new(-12682.09, 390.88, -9902.12) },
        ["Cake Queen"] = { Quest = "IceCreamIslandQuest", Qty = 3, Pos = CFrame.new(-678.64, 381.35, -11114.20), Npc = CFrame.new(-819.37, 64.92, -10967.28) },
        ["Longma"] = { Pos = CFrame.new(-10238.87, 389.79, -9549.79) },
        ["Soul Reaper"] = { Pos = CFrame.new(-9524.78, 315.80, 6655.71) },
    }
end

task.spawn(function()
    while task.wait(0.5) do
        if not _G.ReiHub.AutoBoss or not _G.ReiHub.SelectBoss then
            task.wait(1)
            continue
        end
        
        pcall(function()
            local hrp = GetHRP()
            if not hrp then return end
            
            local bossData = Bosses[_G.ReiHub.SelectBoss]
            if not bossData then return end
            
            local boss = Workspace.Enemies:FindFirstChild(_G.ReiHub.SelectBoss) or Workspace:FindFirstChild(_G.ReiHub.SelectBoss)
            
            if boss and IsAlive(boss) then
                local root = boss:FindFirstChild("HumanoidRootPart")
                if root then
                    TP(root.CFrame * CFrame.new(0, 22, 0))
                    KillMob(boss)
                end
                return
            end
            
            if bossData.Quest and _G.ReiHub.AutoAcceptQuest then
                local hasQuest = Player.PlayerGui.Main.Quest and Player.PlayerGui.Main.Quest.Visible
                if not hasQuest then
                    TP(bossData.Npc)
                    if (hrp.Position - bossData.Npc.Position).Magnitude <= 30 then
                        task.wait(1)
                        ReplicatedStorage.Remotes.CommF_:InvokeServer("StartQuest", bossData.Quest, bossData.Qty)
                    end
                    return
                end
            end
            
            TP(bossData.Pos)
        end)
    end
end)

-- ========================================
-- AUTO COLLECT CHEST
-- ========================================
task.spawn(function()
    while task.wait(0.5) do
        if not _G.ReiHub.AutoCollectChest then
            task.wait(1)
            continue
        end
        
        pcall(function()
            local hrp = GetHRP()
            if not hrp then return end
            
            local collection = game:GetService("CollectionService")
            local chests = collection:GetTagged("_ChestTagged")
            
            local closest = nil
            local closestDist = math.huge
            
            for _, chest in pairs(chests) do
                if not chest:GetAttribute("IsDisabled") then
                    local pos = chest:GetPivot().Position
                    local dist = (hrp.Position - pos).Magnitude
                    if dist < closestDist then
                        closestDist = dist
                        closest = chest
                    end
                end
            end
            
            if closest then
                TP(closest:GetPivot())
            end
        end)
    end
end)

-- ========================================
-- SPEED HACK
-- ========================================
task.spawn(function()
    while task.wait(0.1) do
        if _G.ReiHub.SpeedEnabled then
            local char = Player.Character
            local hum = char and char:FindFirstChildOfClass("Humanoid")
            if hum and hum.WalkSpeed ~= _G.ReiHub.SpeedValue then
                hum.WalkSpeed = _G.ReiHub.SpeedValue
            end
        end
    end
end)

-- ========================================
-- NO CLIP
-- ========================================
task.spawn(function()
    RunService.Stepped:Connect(function()
        if _G.ReiHub.NoClip then
            local char = Player.Character
            if char then
                for _, part in pairs(char:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.CanCollide = false
                    end
                end
            end
        end
    end)
end)

-- ========================================
-- AUTO TEAM
-- ========================================
task.spawn(function()
    task.wait(2)
    if Player.Team and Player.Team.Name ~= "Marines" then
        pcall(function() ReplicatedStorage.Remotes.CommF_:InvokeServer("SetTeam", "Marines") end)
    end
end)

-- ========================================
-- FULL BRIGHT
-- ========================================
Lighting.Ambient = Color3.new(0.7, 0.7, 0.7)
Lighting.Brightness = 2
Lighting.FogEnd = 1e10

-- ========================================
-- UI - REI HUB
-- ========================================
local Library = loadstring(game:HttpGet("https://raw.githubusercontent.com/Jadelly/Ui/refs/heads/main/NewZynLib"))()
local Window = Library:MakeWindow({
    Title = "Rei Hub | Farm",
    SubTitle = "Blox Fruits - Sea 1/2/3",
    SaveFolder = true,
})

-- Tab Principal
local MainTab = Window:MakeTab({ Title = "Main", Icon = "rbxassetid://10709769508" })

MainTab:AddSection({"Auto Farm"})

MainTab:AddDropdown({
    Name = "Select Weapon",
    Options = {"Melee", "Sword", "Blox Fruit", "Gun"},
    Default = _G.ReiHub.SelectWeapon,
    Callback = function(v) _G.ReiHub.SelectWeapon = v; SaveSettings() end
})

MainTab:AddDropdown({
    Name = "Farm Mode",
    Options = {"Level", "Boss"},
    Default = _G.ReiHub.FarmMode,
    Callback = function(v) _G.ReiHub.FarmMode = v; SaveSettings() end
})

MainTab:AddToggle({
    Name = "Start Auto Farm",
    Default = _G.ReiHub.AutoFarm,
    Callback = function(v) _G.ReiHub.AutoFarm = v; SaveSettings() end
})

MainTab:AddToggle({
    Name = "Auto Accept Quests",
    Default = _G.ReiHub.AutoAcceptQuest,
    Callback = function(v) _G.ReiHub.AutoAcceptQuest = v; SaveSettings() end
})

MainTab:AddSection({"Auto Boss"})

local BossList = {}
for name in pairs(Bosses) do table.insert(BossList, name) end

MainTab:AddDropdown({
    Name = "Select Boss",
    Options = BossList,
    Default = _G.ReiHub.SelectBoss or BossList[1],
    Callback = function(v) _G.ReiHub.SelectBoss = v; SaveSettings() end
})

MainTab:AddToggle({
    Name = "Start Auto Boss",
    Default = _G.ReiHub.AutoBoss,
    Callback = function(v) _G.ReiHub.AutoBoss = v; SaveSettings() end
})

MainTab:AddSection({"Auto Stats"})

MainTab:AddDropdown({
    Name = "Stat Type",
    Options = {"Melee", "Defense", "Sword", "Gun", "Devil"},
    Default = _G.ReiHub.StatType,
    Callback = function(v) _G.ReiHub.StatType = v; SaveSettings() end
})

MainTab:AddToggle({
    Name = "Auto Stats",
    Default = _G.ReiHub.AutoStats,
    Callback = function(v) _G.ReiHub.AutoStats = v; SaveSettings() end
})

-- Tab Settings
local SettingsTab = Window:MakeTab({ Title = "Settings", Icon = "rbxassetid://10734950309" })

SettingsTab:AddSection({"Haki / Abilities"})

SettingsTab:AddToggle({
    Name = "Auto Ken (Observation)",
    Default = _G.ReiHub.AutoKen,
    Callback = function(v) _G.ReiHub.AutoKen = v; SaveSettings() end
})

SettingsTab:AddToggle({
    Name = "Auto Haki (Buso)",
    Default = _G.ReiHub.AutoHaki,
    Callback = function(v) _G.ReiHub.AutoHaki = v; SaveSettings() end
})

SettingsTab:AddSection({"Movement"})

SettingsTab:AddToggle({
    Name = "Speed Hack",
    Default = _G.ReiHub.SpeedEnabled,
    Callback = function(v) _G.ReiHub.SpeedEnabled = v; SaveSettings() end
})

SettingsTab:AddTextBox({
    Name = "Speed Value",
    Placeholder = "50",
    Default = tostring(_G.ReiHub.SpeedValue),
    Callback = function(v) local num = tonumber(v); if num then _G.ReiHub.SpeedValue = num; SaveSettings() end end
})

SettingsTab:AddToggle({
    Name = "No Clip",
    Default = _G.ReiHub.NoClip,
    Callback = function(v) _G.ReiHub.NoClip = v; SaveSettings() end
})

SettingsTab:AddSection({"Auto Collect"})

SettingsTab:AddToggle({
    Name = "Auto Collect Chest",
    Default = _G.ReiHub.AutoCollectChest,
    Callback = function(v) _G.ReiHub.AutoCollectChest = v; SaveSettings() end
})

SettingsTab:AddSection({"Bring Settings"})

SettingsTab:AddTextBox({
    Name = "Bring Range",
    Placeholder = "235",
    Default = tostring(_G.ReiHub.BringRange),
    Callback = function(v) local num = tonumber(v); if num then _G.ReiHub.BringRange = num end; SaveSettings() end
})

SettingsTab:AddTextBox({
    Name = "Mob Height",
    Placeholder = "20",
    Default = tostring(_G.ReiHub.MobHeight),
    Callback = function(v) local num = tonumber(v); if num then _G.ReiHub.MobHeight = num end; SaveSettings() end
})

-- Tab Teleports
local TeleportTab = Window:MakeTab({ Title = "Teleports", Icon = "rbxassetid://10734906975" })

TeleportTab:AddSection({"Worlds"})

TeleportTab:AddButton({ Name = "Teleport to Sea 1", Callback = function() pcall(function() ReplicatedStorage.Remotes.CommF_:InvokeServer("TravelMain") end) end })
TeleportTab:AddButton({ Name = "Teleport to Sea 2", Callback = function() pcall(function() ReplicatedStorage.Remotes.CommF_:InvokeServer("TravelDressrosa") end) end })
TeleportTab:AddButton({ Name = "Teleport to Sea 3", Callback = function() pcall(function() ReplicatedStorage.Remotes.CommF_:InvokeServer("TravelZou") end) end })

TeleportTab:AddSection({"Sea 2"})

TeleportTab:AddButton({ Name = "TP to Cursed Ship", Callback = function() pcall(function() ReplicatedStorage.Remotes.CommF_:InvokeServer("requestEntrance", Vector3.new(923.21, 126.97, 32852.83)) end) end })
TeleportTab:AddButton({ Name = "TP to Swan Room", Callback = function() pcall(function() ReplicatedStorage.Remotes.CommF_:InvokeServer("requestEntrance", Vector3.new(2285, 15, 905)) end) end })

TeleportTab:AddSection({"Sea 3"})

TeleportTab:AddButton({ Name = "TP to Hydra", Callback = function() pcall(function() ReplicatedStorage.Remotes.CommF_:InvokeServer("requestEntrance", Vector3.new(5643.45, 1013.08, -340.51)) end) TP(CFrame.new(5206.40, 1004.10, 748.35)) end })
TeleportTab:AddButton({ Name = "TP to Cake Island", Callback = function() TP(CFrame.new(-2091.91, 70.00, -12142.83)) end })
TeleportTab:AddButton({ Name = "TP to Haunted Castle", Callback = function() TP(CFrame.new(-9516.99, 172.01, 6078.46)) end })
TeleportTab:AddButton({ Name = "TP to Tiki Outpost", Callback = function() TP(CFrame.new(-16548.81, 55.60, -172.81)) end })
TeleportTab:AddButton({ Name = "TP to Submerged Island", Callback = function() pcall(function() ReplicatedStorage.Modules.Net:FindFirstChild("RF/SubmarineWorkerSpeak"):InvokeServer("TravelToSubmergedIsland") end) end })

-- Tab Server
local ServerTab = Window:MakeTab({ Title = "Server", Icon = "rbxassetid://7040410130" })

ServerTab:AddButton({
    Name = "Hop Server",
    Callback = function()
        pcall(function()
            for i = math.random(1, 75), 100 do
                local servers = ReplicatedStorage.__ServerBrowser:InvokeServer(i)
                for id, data in pairs(servers) do
                    if tonumber(data.Count) < 12 then
                        game:GetService("TeleportService"):TeleportToPlaceInstance(game.PlaceId, id)
                        return
                    end
                end
            end
        end)
    end
})

ServerTab:AddButton({ Name = "Rejoin Server", Callback = function() game:GetService("TeleportService"):Teleport(game.PlaceId, Player) end })
ServerTab:AddButton({ Name = "Copy Job ID", Callback = function() setclipboard(game.JobId) end })

print("✅ Rei Hub carregado! Mundo: " .. (World1 and "Sea 1" or (World2 and "Sea 2" or "Sea 3")))
