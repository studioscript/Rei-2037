--[[
    ╔══════════════════════════════════════════════════════════╗
    ║                   EMERALD HUB v3.0                       ║
    ║              Blox Fruits - Universal                     ║
    ║         Rayfield Gen2 | All Executors                    ║
    ║         Cor: Verde Esmeralda (80, 200, 80)              ║
    ╚══════════════════════════════════════════════════════════╝
    
    Compativel com TODOS os executores:
    Delta Executor, Arceus X, Codex, Hydrogen,
    Fluxus, Solara, Electron, Vega X, Krnl, Synapse Z
]]

-- ============================================
-- RAYFIELD GEN2 UI LIBRARY
-- ============================================
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

-- ============================================
-- SERVICOS E VARIAVEIS
-- ============================================
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local TeleportService = game:GetService("TeleportService")
local VirtualUser = game:GetService("VirtualUser")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")
local HttpService = game:GetService("HttpService")
local StarterGui = game:GetService("StarterGui")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local Player = Players.LocalPlayer
local Character = Player.Character or Player.CharacterAdded:Wait()
local HumanoidRootPart = Character:WaitForChild("HumanoidRootPart")
local Humanoid = Character:WaitForChild("Humanoid")

-- Atualizar referencias quando renascer
Player.CharacterAdded:Connect(function(char)
    Character = char
    HumanoidRootPart = char:WaitForChild("HumanoidRootPart")
    Humanoid = char:WaitForChild("Humanoid")
end)

-- ============================================
-- VARIAVEIS DE CONTROLE
-- ============================================
local EMERALD = {
    -- Auto Farm
    AutoFarm = {Enabled = false, Connection = nil},
    
    -- Auto Boss
    AutoBoss = {Enabled = false, Connection = nil, SelectedBoss = ""},
    
    -- Auto Stats
    AutoStats = {Enabled = false, Connection = nil, SelectedStat = "Melee"},
    
    -- Auto Haki
    AutoKen = {Enabled = false, Connection = nil},
    AutoBuso = {Enabled = false, Connection = nil},
    
    -- Movement
    SpeedHack = {Enabled = false, Value = 16},
    NoClip = {Enabled = false, Connection = nil},
    InfiniteJump = {Enabled = false, Connection = nil},
    Fly = {Enabled = false, Connection = nil, Speed = 50},
    
    -- Collect
    AutoChest = {Enabled = false, Connection = nil},
    AutoFruit = {Enabled = false, Connection = nil},
    AutoGem = {Enabled = false, Connection = nil},
    
    -- ESP
    ESP = {Enabled = false, Connection = nil},
    
    -- Lock Target
    LockTarget = {Enabled = false, Connection = nil},
    
    -- Teleport manual flag
    ManualTeleport = false,
}

-- ============================================
-- FUNCOES AUXILIARES
-- ============================================

-- Verificar se jogador esta vivo
local function IsAlive()
    return Character and Humanoid and Humanoid.Health > 0 and Character:FindFirstChild("HumanoidRootPart")
end

-- Teleport com Tween (sem interferencia)
local function Teleport(pos)
    if not IsAlive() then return false end
    
    EMERALD.ManualTeleport = true
    
    local success = pcall(function()
        local tween = TweenService:Create(
            HumanoidRootPart,
            TweenInfo.new(0.3, Enum.EasingStyle.Linear),
            {CFrame = CFrame.new(pos)}
        )
        tween:Play()
        tween.Completed:Wait()
    end)
    
    task.wait(0.5)
    EMERALD.ManualTeleport = false
    return success
end

-- Equipar ferramenta
local function EquipTool(name)
    if not IsAlive() then return false end
    
    for _, tool in ipairs(Character:GetChildren()) do
        if tool:IsA("Tool") and string.find(string.lower(tool.Name), string.lower(name)) then
            Humanoid:EquipTool(tool)
            return true
        end
    end
    return false
end

-- Encontrar inimigo mais proximo
local function GetNearestEnemy(mobs)
    if not IsAlive() or not Workspace:FindFirstChild("Enemies") then return nil end
    
    local nearest = nil
    local minDist = math.huge
    
    for _, enemy in ipairs(Workspace.Enemies:GetChildren()) do
        if enemy:FindFirstChild("HumanoidRootPart") and enemy:FindFirstChild("Humanoid") then
            if enemy.Humanoid.Health > 0 and table.find(mobs, enemy.Name) then
                local dist = (HumanoidRootPart.Position - enemy.HumanoidRootPart.Position).Magnitude
                if dist < minDist then
                    nearest = enemy
                    minDist = dist
                end
            end
        end
    end
    
    return nearest
end

-- Encontrar Boss
local function FindBoss(name)
    if not IsAlive() or not Workspace:FindFirstChild("Enemies") then return nil end
    
    for _, enemy in ipairs(Workspace.Enemies:GetChildren()) do
        if enemy.Name == name and enemy:FindFirstChild("HumanoidRootPart") and enemy:FindFirstChild("Humanoid") then
            if enemy.Humanoid.Health > 0 then
                return enemy
            end
        end
    end
    
    return nil
end

-- Detectar Sea
local function GetCurrentSea()
    local placeId = game.PlaceId
    
    local seaMap = {
        [2753915549] = 1,
        [4442272183] = 2,
        [7449423635] = 3,
        [13775256536] = 3,
    }
    
    if seaMap[placeId] then
        return seaMap[placeId]
    end
    
    -- Deteccao por elementos do mapa
    if Workspace:FindFirstChild("Map") then
        if Workspace.Map:FindFirstChild("StartIsland") then return 1 end
        if Workspace.Map:FindFirstChild("SecondSea") then return 2 end
        if Workspace.Map:FindFirstChild("ThirdSea") then return 3 end
    end
    
    return 1
end

-- ============================================
-- BANCO DE DADOS COMPLETO
-- ============================================

-- Quests por nivel (COMPLETO)
local QuestDatabase = {
    [1] = { -- Sea 1
        {Level = {1, 9}, Quest = "BanditQuest1", Mobs = {"Bandit"}, NPC = Vector3.new(1045, 27, 1560), MobPos = Vector3.new(1120, 27, 1590)},
        {Level = {10, 14}, Quest = "JungleQuest", Mobs = {"Monkey"}, NPC = Vector3.new(-1598, 35, 153), MobPos = Vector3.new(-1448, 67, 11)},
        {Level = {15, 29}, Quest = "JungleQuest", Mobs = {"Gorilla"}, NPC = Vector3.new(-1598, 35, 153), MobPos = Vector3.new(-1129, 40, -525)},
        {Level = {30, 39}, Quest = "BuggyQuest1", Mobs = {"Pirate"}, NPC = Vector3.new(-1141, 4, 3831), MobPos = Vector3.new(-1103, 13, 3896)},
        {Level = {40, 59}, Quest = "BuggyQuest1", Mobs = {"Brute"}, NPC = Vector3.new(-1141, 4, 3831), MobPos = Vector3.new(-1140, 14, 4322)},
        {Level = {60, 74}, Quest = "DesertQuest", Mobs = {"Desert Bandit"}, NPC = Vector3.new(894, 5, 4392), MobPos = Vector3.new(924, 6, 4481)},
        {Level = {75, 89}, Quest = "DesertQuest", Mobs = {"Desert Officer"}, NPC = Vector3.new(894, 5, 4392), MobPos = Vector3.new(1608, 8, 4371)},
        {Level = {90, 99}, Quest = "SnowQuest", Mobs = {"Snow Bandit"}, NPC = Vector3.new(1389, 88, -1298), MobPos = Vector3.new(1354, 87, -1393)},
        {Level = {100, 119}, Quest = "SnowQuest", Mobs = {"Snowman"}, NPC = Vector3.new(1389, 88, -1298), MobPos = Vector3.new(1201, 144, -1550)},
        {Level = {120, 149}, Quest = "MarineQuest2", Mobs = {"Chief Petty Officer"}, NPC = Vector3.new(-5039, 27, 4324), MobPos = Vector3.new(-4881, 22, 4273)},
        {Level = {150, 174}, Quest = "SkyQuest", Mobs = {"Sky Bandit"}, NPC = Vector3.new(-4839, 716, -2619), MobPos = Vector3.new(-4953, 295, -2899)},
        {Level = {175, 199}, Quest = "SkyQuest", Mobs = {"Dark Master"}, NPC = Vector3.new(-4839, 716, -2619), MobPos = Vector3.new(-5259, 391, -2229)},
        {Level = {200, 224}, Quest = "SkyQuest2", Mobs = {"Sky Guardian"}, NPC = Vector3.new(-4839, 716, -2619), MobPos = Vector3.new(-5050, 400, -2350)},
        {Level = {225, 249}, Quest = "PrisonQuest", Mobs = {"Prisoner"}, NPC = Vector3.new(4877, 5, 655), MobPos = Vector3.new(4750, 5, 550)},
        {Level = {250, 274}, Quest = "PrisonQuest", Mobs = {"Dangerous Prisoner"}, NPC = Vector3.new(4877, 5, 655), MobPos = Vector3.new(4700, 5, 450)},
        {Level = {275, 299}, Quest = "PrisonQuest", Mobs = {"Prison Guard"}, NPC = Vector3.new(4877, 5, 655), MobPos = Vector3.new(4600, 5, 500)},
        {Level = {300, 329}, Quest = "ColosseumQuest", Mobs = {"Toga Warrior"}, NPC = Vector3.new(-1568, 23, -2930), MobPos = Vector3.new(-1500, 23, -2850)},
        {Level = {330, 374}, Quest = "ColosseumQuest", Mobs = {"Gladiator"}, NPC = Vector3.new(-1568, 23, -2930), MobPos = Vector3.new(-1450, 23, -2800)},
        {Level = {375, 399}, Quest = "MagmaQuest", Mobs = {"Military Soldier"}, NPC = Vector3.new(-5232, 8, 8475), MobPos = Vector3.new(-5300, 8, 8400)},
        {Level = {400, 424}, Quest = "MagmaQuest", Mobs = {"Military Spy"}, NPC = Vector3.new(-5232, 8, 8475), MobPos = Vector3.new(-5400, 8, 8500)},
        {Level = {425, 449}, Quest = "FishmanQuest", Mobs = {"Fishman Warrior"}, NPC = Vector3.new(4605, 2, 1276), MobPos = Vector3.new(4500, 2, 1200)},
        {Level = {450, 474}, Quest = "FishmanQuest", Mobs = {"Fishman Commando"}, NPC = Vector3.new(4605, 2, 1276), MobPos = Vector3.new(4400, 2, 1100)},
        {Level = {475, 524}, Quest = "FishmanQuest", Mobs = {"Fishman Lord"}, NPC = Vector3.new(4605, 2, 1276), MobPos = Vector3.new(4300, 2, 1000)},
        {Level = {525, 574}, Quest = "SeaSoldierQuest", Mobs = {"Sea Soldier"}, NPC = Vector3.new(1050, 5, 1025), MobPos = Vector3.new(950, 5, 950)},
        {Level = {575, 624}, Quest = "SeaSoldierQuest", Mobs = {"Sea Captain"}, NPC = Vector3.new(1050, 5, 1025), MobPos = Vector3.new(850, 5, 850)},
        {Level = {625, 649}, Quest = "FountainQuest", Mobs = {"Galley Pirate"}, NPC = Vector3.new(5259, 37, 4050), MobPos = Vector3.new(5551, 78, 3930)},
        {Level = {650, 9999}, Quest = "FountainQuest", Mobs = {"Galley Captain"}, NPC = Vector3.new(5259, 37, 4050), MobPos = Vector3.new(5441, 42, 4950)},
    },
    [2] = { -- Sea 2
        {Level = {700, 724}, Quest = "Area1Quest", Mobs = {"Raider"}, NPC = Vector3.new(-429, 71, 1836), MobPos = Vector3.new(-728, 52, 2345)},
        {Level = {725, 774}, Quest = "Area1Quest", Mobs = {"Mercenary"}, NPC = Vector3.new(-429, 71, 1836), MobPos = Vector3.new(-1004, 80, 1424)},
        {Level = {775, 799}, Quest = "Area2Quest", Mobs = {"Swan Pirate"}, NPC = Vector3.new(638, 71, 918), MobPos = Vector3.new(1068, 137, 1322)},
        {Level = {800, 874}, Quest = "Area2Quest", Mobs = {"Factory Staff"}, NPC = Vector3.new(632, 73, 918), MobPos = Vector3.new(73, 81, -27)},
        {Level = {875, 899}, Quest = "MarineQuest3", Mobs = {"Marine Lieutenant"}, NPC = Vector3.new(-2440, 71, -3216), MobPos = Vector3.new(-2821, 75, -3070)},
        {Level = {900, 949}, Quest = "MarineQuest3", Mobs = {"Marine Captain"}, NPC = Vector3.new(-2440, 71, -3216), MobPos = Vector3.new(-1861, 80, -3254)},
        {Level = {950, 974}, Quest = "ZombieQuest", Mobs = {"Zombie"}, NPC = Vector3.new(-5497, 47, -795), MobPos = Vector3.new(-5657, 78, -928)},
        {Level = {975, 999}, Quest = "ZombieQuest", Mobs = {"Vampire"}, NPC = Vector3.new(-5497, 47, -795), MobPos = Vector3.new(-6037, 32, -1340)},
        {Level = {1000, 1024}, Quest = "SnowMountainQuest", Mobs = {"Snow Trooper"}, NPC = Vector3.new(608, 401, -5370), MobPos = Vector3.new(535, 401, -5275)},
        {Level = {1025, 1049}, Quest = "SnowMountainQuest", Mobs = {"Winter Warrior"}, NPC = Vector3.new(608, 401, -5370), MobPos = Vector3.new(700, 401, -5400)},
        {Level = {1050, 1074}, Quest = "SnowMountainQuest", Mobs = {"Lab Subordinate"}, NPC = Vector3.new(608, 401, -5370), MobPos = Vector3.new(400, 401, -5200)},
        {Level = {1075, 1124}, Quest = "IceCastleQuest", Mobs = {"Ice Knight"}, NPC = Vector3.new(1300, 125, -6800), MobPos = Vector3.new(1200, 125, -6700)},
        {Level = {1125, 1174}, Quest = "IceCastleQuest", Mobs = {"Ice Mage"}, NPC = Vector3.new(1300, 125, -6800), MobPos = Vector3.new(1100, 125, -6600)},
        {Level = {1175, 1224}, Quest = "ForgottenQuest", Mobs = {"Forgotten Swordsman"}, NPC = Vector3.new(-3054, 235, -10142), MobPos = Vector3.new(-3150, 235, -10050)},
        {Level = {1225, 1274}, Quest = "ForgottenQuest", Mobs = {"Forgotten Warrior"}, NPC = Vector3.new(-3054, 235, -10142), MobPos = Vector3.new(-3250, 235, -10100)},
        {Level = {1275, 1324}, Quest = "DarkArenaQuest", Mobs = {"Dark Knight"}, NPC = Vector3.new(-1850, 15, -3150), MobPos = Vector3.new(-1750, 15, -3050)},
        {Level = {1325, 1374}, Quest = "DarkArenaQuest", Mobs = {"Dark Sorcerer"}, NPC = Vector3.new(-1850, 15, -3150), MobPos = Vector3.new(-1650, 15, -2950)},
        {Level = {1375, 1424}, Quest = "HotColdQuest", Mobs = {"Hot Demon"}, NPC = Vector3.new(-6037, 32, -1340), MobPos = Vector3.new(-6100, 32, -1300)},
        {Level = {1425, 1449}, Quest = "HotColdQuest", Mobs = {"Cold Demon"}, NPC = Vector3.new(-6037, 32, -1340), MobPos = Vector3.new(-6150, 32, -1400)},
        {Level = {1450, 9999}, Quest = "ForgottenQuest", Mobs = {"Water Fighter"}, NPC = Vector3.new(-3054, 235, -10142), MobPos = Vector3.new(-3352, 285, -10534)},
    },
    [3] = { -- Sea 3
        {Level = {1500, 1524}, Quest = "PiratePortQuest", Mobs = {"Pirate Millionaire"}, NPC = Vector3.new(-290, 42, 5581), MobPos = Vector3.new(-246, 47, 5584)},
        {Level = {1525, 1574}, Quest = "PiratePortQuest", Mobs = {"Pistol Billionaire"}, NPC = Vector3.new(-290, 42, 5581), MobPos = Vector3.new(-187, 86, 6013)},
        {Level = {1575, 1599}, Quest = "DragonCrewQuest", Mobs = {"Dragon Crew Warrior"}, NPC = Vector3.new(6737, 127, -712), MobPos = Vector3.new(6709, 52, -1139)},
        {Level = {1600, 1649}, Quest = "DragonCrewQuest", Mobs = {"Dragon Crew Archer"}, NPC = Vector3.new(6737, 127, -712), MobPos = Vector3.new(6543, 62, -1152)},
        {Level = {1650, 1699}, Quest = "VenomCrewQuest", Mobs = {"Venomous Assailant"}, NPC = Vector3.new(5206, 1004, 748), MobPos = Vector3.new(4674, 1134, 996)},
        {Level = {1700, 1749}, Quest = "VenomCrewQuest", Mobs = {"Venomous Grunt"}, NPC = Vector3.new(5206, 1004, 748), MobPos = Vector3.new(4725, 1134, 1056)},
        {Level = {1750, 1774}, Quest = "DeepForestQuest2", Mobs = {"Forest Guardian"}, NPC = Vector3.new(-10581, 330, -8761), MobPos = Vector3.new(-10550, 331, -8680)},
        {Level = {1775, 1799}, Quest = "DeepForestQuest3", Mobs = {"Fishman Raider"}, NPC = Vector3.new(-10581, 330, -8761), MobPos = Vector3.new(-10407, 331, -8368)},
        {Level = {1800, 1824}, Quest = "DeepForestQuest3", Mobs = {"Fishman Captain"}, NPC = Vector3.new(-10581, 330, -8761), MobPos = Vector3.new(-10300, 331, -8250)},
        {Level = {1825, 1849}, Quest = "DeepForestQuest", Mobs = {"Forest Pirate"}, NPC = Vector3.new(-13234, 331, -7625), MobPos = Vector3.new(-13274, 332, -7769)},
        {Level = {1850, 1899}, Quest = "DeepForestQuest", Mobs = {"Mythological Pirate"}, NPC = Vector3.new(-13234, 331, -7625), MobPos = Vector3.new(-13680, 501, -6991)},
        {Level = {1900, 1949}, Quest = "DeepForestQuest", Mobs = {"Jungle Pirate"}, NPC = Vector3.new(-13234, 331, -7625), MobPos = Vector3.new(-13450, 332, -7400)},
        {Level = {1950, 1974}, Quest = "HauntedQuest2", Mobs = {"Living Zombie"}, NPC = Vector3.new(-9479, 141, 5566), MobPos = Vector3.new(-9200, 155, 6100)},
        {Level = {1975, 1999}, Quest = "HauntedQuest1", Mobs = {"Reborn Skeleton"}, NPC = Vector3.new(-9479, 141, 5566), MobPos = Vector3.new(-8763, 165, 6159)},
        {Level = {2000, 2024}, Quest = "HauntedQuest1", Mobs = {"Demonic Soul"}, NPC = Vector3.new(-9479, 141, 5566), MobPos = Vector3.new(-8700, 165, 5950)},
        {Level = {2025, 2049}, Quest = "HauntedQuest2", Mobs = {"Possessed Mummy"}, NPC = Vector3.new(-9479, 141, 5566), MobPos = Vector3.new(-9500, 155, 6300)},
        {Level = {2050, 2074}, Quest = "SnowMountainQuest3", Mobs = {"Snow Trooper"}, NPC = Vector3.new(608, 401, -5370), MobPos = Vector3.new(535, 401, -5275)},
        {Level = {2075, 2099}, Quest = "SnowMountainQuest3", Mobs = {"Winter Warrior"}, NPC = Vector3.new(608, 401, -5370), MobPos = Vector3.new(700, 401, -5400)},
        {Level = {2100, 2149}, Quest = "SnowMountainQuest3", Mobs = {"Lab Subordinate"}, NPC = Vector3.new(608, 401, -5370), MobPos = Vector3.new(400, 401, -5200)},
        {Level = {2150, 2199}, Quest = "CakeQuest2", Mobs = {"Cake Prince"}, NPC = Vector3.new(-2021, 37, -12028), MobPos = Vector3.new(-2200, 37, -12150)},
        {Level = {2200, 2224}, Quest = "CakeQuest1", Mobs = {"Cookie Crafter"}, NPC = Vector3.new(-2021, 37, -12028), MobPos = Vector3.new(-2374, 37, -12125)},
        {Level = {2225, 2249}, Quest = "CakeQuest1", Mobs = {"Cake Guard"}, NPC = Vector3.new(-2021, 37, -12028), MobPos = Vector3.new(-2450, 37, -12200)},
        {Level = {2250, 2274}, Quest = "CakeQuest1", Mobs = {"Baking Staff"}, NPC = Vector3.new(-2021, 37, -12028), MobPos = Vector3.new(-2300, 37, -12050)},
        {Level = {2275, 2299}, Quest = "CakeQuest2", Mobs = {"Head Baker"}, NPC = Vector3.new(-2021, 37, -12028), MobPos = Vector3.new(-2400, 37, -12250)},
        {Level = {2300, 2324}, Quest = "TurtleQuest1", Mobs = {"Turtle Guard"}, NPC = Vector3.new(-10581, 330, -8761), MobPos = Vector3.new(-10650, 331, -8700)},
        {Level = {2325, 2349}, Quest = "TurtleQuest1", Mobs = {"Turtle Warrior"}, NPC = Vector3.new(-10581, 330, -8761), MobPos = Vector3.new(-10700, 331, -8650)},
        {Level = {2350, 2374}, Quest = "TurtleQuest2", Mobs = {"Turtle General"}, NPC = Vector3.new(-10581, 330, -8761), MobPos = Vector3.new(-10750, 331, -8600)},
        {Level = {2375, 2399}, Quest = "CastleQuest", Mobs = {"Castle Guard"}, NPC = Vector3.new(-5400, 314, -2800), MobPos = Vector3.new(-5300, 314, -2750)},
        {Level = {2400, 2424}, Quest = "CastleQuest", Mobs = {"Castle Knight"}, NPC = Vector3.new(-5400, 314, -2800), MobPos = Vector3.new(-5200, 314, -2700)},
        {Level = {2425, 2449}, Quest = "CastleQuest", Mobs = {"Castle Mage"}, NPC = Vector3.new(-5400, 314, -2800), MobPos = Vector3.new(-5100, 314, -2650)},
        {Level = {2450, 2474}, Quest = "TikiQuest1", Mobs = {"Isle Outlaw"}, NPC = Vector3.new(-16548, 55, -172), MobPos = Vector3.new(-16479, 226, -300)},
        {Level = {2475, 2499}, Quest = "TikiQuest2", Mobs = {"Isle Bandit"}, NPC = Vector3.new(-16548, 55, -172), MobPos = Vector3.new(-16550, 226, -400)},
        {Level = {2500, 2524}, Quest = "TikiQuest2", Mobs = {"Isle Guard"}, NPC = Vector3.new(-16548, 55, -172), MobPos = Vector3.new(-16600, 226, -350)},
        {Level = {2525, 2549}, Quest = "TikiQuest3", Mobs = {"Tiki Warrior"}, NPC = Vector3.new(-16668, 105, 1568), MobPos = Vector3.new(-16650, 419, 1700)},
        {Level = {2550, 2574}, Quest = "TikiQuest3", Mobs = {"Tiki Sorcerer"}, NPC = Vector3.new(-16668, 105, 1568), MobPos = Vector3.new(-16700, 419, 1800)},
        {Level = {2575, 2599}, Quest = "TikiQuest3", Mobs = {"Skull Slayer"}, NPC = Vector3.new(-16668, 105, 1568), MobPos = Vector3.new(-16709, 419, 1751)},
    }
}

-- Bosses por Sea
local BossDatabase = {
    [1] = {
        "The Gorilla King", "Bobby", "The Saw", "Yeti", 
        "Vice Admiral", "Saber Expert", "Magma Admiral", 
        "Fishman Lord", "Wysper", "Thunder God", "Cyborg", 
        "Ice Admiral", "Greybeard"
    },
    [2] = {
        "Diamond", "Jeremy", "Fajita", "Don Swan", 
        "Smoke Admiral", "Awakened Ice Admiral", "Tide Keeper", 
        "Darkbeard", "Cursed Captain", "Order"
    },
    [3] = {
        "Stone", "Hydra Leader", "Kilo Admiral", 
        "Captain Elephant", "Beautiful Pirate", "Cake Queen", 
        "Longma", "Soul Reaper"
    }
}

-- Posicoes de ilhas
local IslandPositions = {
    -- Sea 1
    ["Start Island"] = Vector3.new(1070, 16, 1450),
    ["Jungle"] = Vector3.new(-1250, 11, 150),
    ["Pirate Village"] = Vector3.new(-1122, 3, 3868),
    ["Desert"] = Vector3.new(1094, 3, 4266),
    ["Snow Island"] = Vector3.new(1384, 87, -1297),
    ["Marine Fortress"] = Vector3.new(-4823, 22, 4320),
    ["Skylands"] = Vector3.new(-4970, 717, -2623),
    ["Prison"] = Vector3.new(4877, 5, 655),
    ["Colosseum"] = Vector3.new(-1568, 23, -2930),
    ["Magma Village"] = Vector3.new(-5232, 8, 8475),
    ["Underwater City"] = Vector3.new(4605, 2, 1276),
    ["Fountain City"] = Vector3.new(5200, 27, 4100),
    ["Marine Base"] = Vector3.new(-2440, 71, -3216),
    
    -- Sea 2
    ["Kingdom of Rose"] = Vector3.new(500, 60, 650),
    ["Green Zone"] = Vector3.new(-650, 38, 2200),
    ["Graveyard"] = Vector3.new(-5650, 47, -928),
    ["Snow Mountain"] = Vector3.new(500, 401, -3000),
    ["Hot and Cold"] = Vector3.new(-6037, 32, -1340),
    ["Cursed Ship"] = Vector3.new(916, 181, 33422),
    ["Ice Castle"] = Vector3.new(1300, 125, -6800),
    ["Forgotten Island"] = Vector3.new(-3054, 235, -10142),
    ["Dark Arena"] = Vector3.new(-1850, 15, -3150),
    ["Swan Room"] = Vector3.new(2286, 20, 863),
    ["Factory"] = Vector3.new(448, 199, -441),
    ["Usoapp Island"] = Vector3.new(2200, 50, -4800),
    
    -- Sea 3
    ["Port Town"] = Vector3.new(-290, 42, 5581),
    ["Hydra Island"] = Vector3.new(5206, 1004, 748),
    ["Great Tree"] = Vector3.new(2280, 252, -6590),
    ["Castle on the Sea"] = Vector3.new(-5400, 314, -2800),
    ["Floating Turtle"] = Vector3.new(-10581, 330, -8761),
    ["Haunted Castle"] = Vector3.new(-9516, 172, 6078),
    ["Cake Island"] = Vector3.new(-2091, 70, -12142),
    ["Tiki Outpost"] = Vector3.new(-16548, 55, -172),
    ["Ice Cream Island"] = Vector3.new(-2250, 70, -11800),
    ["Chocolate Island"] = Vector3.new(-2300, 70, -11600),
}

-- ============================================
-- FUNCOES PRINCIPAIS (CORRIGIDAS)
-- ============================================

-- Auto Farm Loop
local function StartAutoFarm()
    if EMERALD.AutoFarm.Connection then
        EMERALD.AutoFarm.Connection:Disconnect()
    end
    
    EMERALD.AutoFarm.Connection = RunService.Heartbeat:Connect(function()
        if not EMERALD.AutoFarm.Enabled then return end
        if not IsAlive() then return end
        if EMERALD.ManualTeleport then return end
        
        local sea = GetCurrentSea()
        local level = Player.Data.Level.Value
        local config = nil
        
        for _, q in ipairs(QuestDatabase[sea] or {}) do
            if level >= q.Level[1] and level <= q.Level[2] then
                config = q
                break
            end
        end
        
        if not config then return end
        
        -- Ir para NPC
        pcall(function()
            HumanoidRootPart.CFrame = CFrame.new(config.NPC)
        end)
        task.wait(0.3)
        
        -- Procurar inimigo
        local enemy = GetNearestEnemy(config.Mobs)
        
        if enemy then
            -- Ir para inimigo
            pcall(function()
                HumanoidRootPart.CFrame = enemy.HumanoidRootPart.CFrame + Vector3.new(0, 3, 0)
            end)
            task.wait(0.2)
            
            EquipTool("Combat")
            
            pcall(function()
                ReplicatedStorage.Remotes.CommF_:InvokeServer("ActivateSkill", "Melee", "Slot1")
            end)
        else
            -- Ir para area de spawn
            pcall(function()
                HumanoidRootPart.CFrame = CFrame.new(config.MobPos)
            end)
        end
        
        task.wait(0.1)
    end)
end

-- Auto Boss Loop
local function StartAutoBoss()
    if EMERALD.AutoBoss.Connection then
        EMERALD.AutoBoss.Connection:Disconnect()
    end
    
    EMERALD.AutoBoss.Connection = RunService.Heartbeat:Connect(function()
        if not EMERALD.AutoBoss.Enabled then return end
        if not IsAlive() then return end
        if EMERALD.ManualTeleport then return end
        if EMERALD.AutoBoss.SelectedBoss == "" then return end
        
        local boss = FindBoss(EMERALD.AutoBoss.SelectedBoss)
        
        if boss then
            pcall(function()
                HumanoidRootPart.CFrame = boss.HumanoidRootPart.CFrame + Vector3.new(0, 5, 0)
            end)
            task.wait(0.2)
            EquipTool("Combat")
            
            pcall(function()
                ReplicatedStorage.Remotes.CommF_:InvokeServer("ActivateSkill", "Melee", "Slot1")
            end)
        end
        
        task.wait(0.5)
    end)
end

-- Auto Stats Loop
local function StartAutoStats()
    if EMERALD.AutoStats.Connection then
        EMERALD.AutoStats.Connection:Disconnect()
    end
    
    EMERALD.AutoStats.Connection = RunService.Heartbeat:Connect(function()
        if not EMERALD.AutoStats.Enabled then return end
        if not IsAlive() then return end
        
        local points = Player.Data.Points.Value
        if points > 0 then
            pcall(function()
                ReplicatedStorage.Remotes.CommF_:InvokeServer("AddPoint", EMERALD.AutoStats.SelectedStat, 1)
            end)
        end
        
        task.wait(0.3)
    end)
end

-- Auto Ken Loop
local function StartAutoKen()
    if EMERALD.AutoKen.Connection then
        EMERALD.AutoKen.Connection:Disconnect()
    end
    
    EMERALD.AutoKen.Connection = RunService.Heartbeat:Connect(function()
        if not EMERALD.AutoKen.Enabled then return end
        if not IsAlive() then return end
        
        pcall(function()
            ReplicatedStorage.Remotes.CommF_:InvokeServer("ActivateInstinct")
        end)
        
        task.wait(1)
    end)
end

-- Auto Buso Loop
local function StartAutoBuso()
    if EMERALD.AutoBuso.Connection then
        EMERALD.AutoBuso.Connection:Disconnect()
    end
    
    EMERALD.AutoBuso.Connection = RunService.Heartbeat:Connect(function()
        if not EMERALD.AutoBuso.Enabled then return end
        if not IsAlive() then return end
        
        pcall(function()
            ReplicatedStorage.Remotes.CommF_:InvokeServer("Buso")
        end)
        
        task.wait(0.5)
    end)
end

-- No Clip Loop
local function StartNoClip()
    if EMERALD.NoClip.Connection then
        EMERALD.NoClip.Connection:Disconnect()
    end
    
    EMERALD.NoClip.Connection = RunService.Heartbeat:Connect(function()
        if not EMERALD.NoClip.Enabled then return end
        if not IsAlive() then return end
        
        for _, part in ipairs(Character:GetDescendants()) do
            if part:IsA("BasePart") and part.CanCollide then
                part.CanCollide = false
            end
        end
    end)
end

-- Infinite Jump Loop
local function StartInfiniteJump()
    if EMERALD.InfiniteJump.Connection then
        EMERALD.InfiniteJump.Connection:Disconnect()
    end
    
    EMERALD.InfiniteJump.Connection = UserInputService.JumpRequest:Connect(function()
        if not EMERALD.InfiniteJump.Enabled then return end
        if not IsAlive() then return end
        
        Humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
    end)
end

-- Fly Loop
local function StartFly()
    if EMERALD.Fly.Connection then
        EMERALD.Fly.Connection:Disconnect()
    end
    
    EMERALD.Fly.Connection = RunService.Heartbeat:Connect(function()
        if not EMERALD.Fly.Enabled then return end
        if not IsAlive() then return end
        
        local speed = EMERALD.Fly.Speed
        local moveDir = Vector3.zero
        
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then
            moveDir = moveDir + (Workspace.CurrentCamera.CFrame.LookVector * speed)
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then
            moveDir = moveDir - (Workspace.CurrentCamera.CFrame.LookVector * speed)
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then
            moveDir = moveDir - (Workspace.CurrentCamera.CFrame.RightVector * speed)
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then
            moveDir = moveDir + (Workspace.CurrentCamera.CFrame.RightVector * speed)
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
            moveDir = moveDir + (Vector3.new(0, speed, 0))
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
            moveDir = moveDir - (Vector3.new(0, speed, 0))
        end
        
        HumanoidRootPart.Velocity = moveDir
        Humanoid.PlatformStand = true
    end)
end

-- Auto Collect Chests Loop
local function StartAutoChest()
    if EMERALD.AutoChest.Connection then
        EMERALD.AutoChest.Connection:Disconnect()
    end
    
    EMERALD.AutoChest.Connection = RunService.Heartbeat:Connect(function()
        if not EMERALD.AutoChest.Enabled then return end
        if not IsAlive() then return end
        if EMERALD.ManualTeleport then return end
        
        for _, obj in ipairs(Workspace:GetDescendants()) do
            if obj:IsA("ProximityPrompt") and obj.Parent then
                local parent = obj.Parent
                if string.find(string.lower(parent.Name), "chest") then
                    local dist = (HumanoidRootPart.Position - parent.Position).Magnitude
                    if dist < 100 then
                        pcall(function()
                            HumanoidRootPart.CFrame = CFrame.new(parent.Position + Vector3.new(0, 5, 0))
                        end)
                        task.wait(0.2)
                        fireproximityprompt(obj)
                        task.wait(0.5)
                    end
                end
            end
        end
        
        task.wait(2)
    end)
end

-- Auto Collect Fruits Loop
local function StartAutoFruit()
    if EMERALD.AutoFruit.Connection then
        EMERALD.AutoFruit.Connection:Disconnect()
    end
    
    EMERALD.AutoFruit.Connection = RunService.Heartbeat:Connect(function()
        if not EMERALD.AutoFruit.Enabled then return end
        if not IsAlive() then return end
        if EMERALD.ManualTeleport then return end
        
        for _, obj in ipairs(Workspace:GetDescendants()) do
            if obj:IsA("Tool") and string.find(string.lower(obj.Name), "fruit") then
                local dist = (HumanoidRootPart.Position - obj.Position).Magnitude
                if dist < 150 then
                    pcall(function()
                        HumanoidRootPart.CFrame = CFrame.new(obj.Position)
                    end)
                    task.wait(0.3)
                    
                    local handle = obj:FindFirstChild("Handle") or obj:FindFirstChildWhichIsA("BasePart")
                    if handle then
                        firetouchinterest(HumanoidRootPart, handle, 0)
                        firetouchinterest(HumanoidRootPart, handle, 1)
                    end
                    task.wait(0.5)
                end
            end
        end
        
        task.wait(3)
    end)
end

-- Auto Collect Gems Loop
local function StartAutoGem()
    if EMERALD.AutoGem.Connection then
        EMERALD.AutoGem.Connection:Disconnect()
    end
    
    EMERALD.AutoGem.Connection = RunService.Heartbeat:Connect(function()
        if not EMERALD.AutoGem.Enabled then return end
        if not IsAlive() then return end
        if EMERALD.ManualTeleport then return end
        
        for _, obj in ipairs(Workspace:GetDescendants()) do
            if obj:IsA("ProximityPrompt") and obj.Parent then
                local parent = obj.Parent
                if string.find(string.lower(parent.Name), "gem") then
                    local dist = (HumanoidRootPart.Position - parent.Position).Magnitude
                    if dist < 100 then
                        pcall(function()
                            HumanoidRootPart.CFrame = CFrame.new(parent.Position + Vector3.new(0, 3, 0))
                        end)
                        task.wait(0.2)
                        fireproximityprompt(obj)
                        task.wait(0.5)
                    end
                end
            end
        end
        
        task.wait(2)
    end)
end

-- ESP Loop
local function StartESP()
    if EMERALD.ESP.Connection then
        EMERALD.ESP.Connection:Disconnect()
    end
    
    local espFolder = Instance.new("Folder")
    espFolder.Name = "EMERALD_ESP"
    espFolder.Parent = Workspace
    
    EMERALD.ESP.Connection = RunService.Heartbeat:Connect(function()
        if not EMERALD.ESP.Enabled then
            -- Limpar ESP
            espFolder:ClearAllChildren()
            return
        end
        if not IsAlive() then return end
        
        espFolder:ClearAllChildren()
        
        -- ESP para inimigos
        if Workspace:FindFirstChild("Enemies") then
            for _, enemy in ipairs(Workspace.Enemies:GetChildren()) do
                if enemy:FindFirstChild("HumanoidRootPart") and enemy:FindFirstChild("Humanoid") then
                    if enemy.Humanoid.Health > 0 then
                        local highlight = Instance.new("Highlight")
                        highlight.Name = enemy.Name
                        highlight.FillColor = Color3.fromRGB(255, 0, 0)
                        highlight.FillTransparency = 0.5
                        highlight.OutlineColor = Color3.fromRGB(80, 200, 80)
                        highlight.OutlineTransparency = 0
                        highlight.Parent = espFolder
                        highlight.Adornee = enemy
                    end
                end
            end
        end
        
        -- ESP para jogadores
        for _, otherPlayer in ipairs(Players:GetPlayers()) do
            if otherPlayer ~= Player and otherPlayer.Character then
                local otherChar = otherPlayer.Character
                if otherChar:FindFirstChild("HumanoidRootPart") then
                    local highlight = Instance.new("Highlight")
                    highlight.Name = otherPlayer.Name
                    highlight.FillColor = Color3.fromRGB(0, 255, 255)
                    highlight.FillTransparency = 0.5
                    highlight.OutlineColor = Color3.fromRGB(80, 200, 80)
                    highlight.OutlineTransparency = 0
                    highlight.Parent = espFolder
                    highlight.Adornee = otherChar
                end
            end
        end
    end)
end

-- Lock Target Loop
local function StartLockTarget()
    if EMERALD.LockTarget.Connection then
        EMERALD.LockTarget.Connection:Disconnect()
    end
    
    EMERALD.LockTarget.Connection = RunService.Heartbeat:Connect(function()
        if not EMERALD.LockTarget.Enabled then return end
        if not IsAlive() then return end
        if EMERALD.ManualTeleport then return end
        
        local nearest = nil
        local minDist = math.huge
        
        if Workspace:FindFirstChild("Enemies") then
            for _, enemy in ipairs(Workspace.Enemies:GetChildren()) do
                if enemy:FindFirstChild("HumanoidRootPart") and enemy:FindFirstChild("Humanoid") then
                    if enemy.Humanoid.Health > 0 then
                        local dist = (HumanoidRootPart.Position - enemy.HumanoidRootPart.Position).Magnitude
                        if dist < minDist and dist < 500 then
                            nearest = enemy
                            minDist = dist
                        end
                    end
                end
            end
        end
        
        if nearest then
            pcall(function()
                HumanoidRootPart.CFrame = nearest.HumanoidRootPart.CFrame + Vector3.new(0, 3, 0)
            end)
            task.wait(0.1)
            EquipTool("Combat")
            pcall(function()
                ReplicatedStorage.Remotes.CommF_:InvokeServer("ActivateSkill", "Melee", "Slot1")
            end)
        end
    end)
end

-- Hop Server
local function HopServer()
    local req = request or http_request or syn.request
    
    if not req then
        Rayfield:Notify({
            Title = "EMERALD HUB",
            Content = "Executor nao suporta HTTP requests!",
            Duration = 5,
            Image = 4483362458,
        })
        return
    end
    
    Rayfield:Notify({
        Title = "EMERALD HUB",
        Content = "Procurando servidores...",
        Duration = 3,
        Image = 4483362458,
    })
    
    local servers = {}
    local cursor = ""
    
    repeat
        local success, response = pcall(function()
            return req({
                Url = "https://games.roblox.com/v1/games/" .. game.PlaceId .. "/servers/Public?sortOrder=Asc&limit=100&cursor=" .. cursor
            })
        end)
        
        if success and response then
            local body = HttpService:JSONDecode(response.Body)
            if body and body.data then
                for _, server in ipairs(body.data) do
                    if server.playing < server.maxPlayers and server.id ~= game.JobId then
                        table.insert(servers, server.id)
                    end
                end
                cursor = body.nextPageCursor or ""
            else
                break
            end
        else
            break
        end
    until cursor == "" or #servers >= 50
    
    if #servers > 0 then
        local target = servers[math.random(1, #servers)]
        TeleportService:TeleportToPlaceInstance(game.PlaceId, target, Player)
    else
        Rayfield:Notify({
            Title = "EMERALD HUB",
            Content = "Nenhum servidor encontrado!",
            Duration = 5,
            Image = 4483362458,
        })
    end
end

-- ============================================
-- CRIAR INTERFACE RAYFIELD GEN2
-- ============================================
local Window = Rayfield:CreateWindow({
    Name = "EMERALD HUB v3.0",
    LoadingTitle = "EMERALD HUB Loading...",
    LoadingSubtitle = "by Emerald Team | Universal Executor",
    ConfigurationSaving = {
        Enabled = true,
        FolderName = "EmeraldHub",
        FileName = "Settings_v3"
    },
    Discord = {
        Enabled = false,
    },
    KeySystem = false,
})

-- Notificacao de carregamento
Rayfield:Notify({
    Title = "EMERALD HUB v3.0",
    Content = "Carregado com sucesso! Sea " .. GetCurrentSea(),
    Duration = 5,
    Image = 4483362458,
})

-- ============================================
-- ABA 1: AUTO FARM
-- ============================================
local FarmTab = Window:CreateTab("Auto Farm", 4483362458)

FarmTab:CreateSection("AUTO FARM LEVEL")

FarmTab:CreateToggle({
    Name = "Auto Farm Level",
    CurrentValue = false,
    Flag = "AutoFarm",
    Callback = function(Value)
        EMERALD.AutoFarm.Enabled = Value
        if Value then
            -- Desativar Auto Boss se estiver ativo
            if EMERALD.AutoBoss.Enabled then
                EMERALD.AutoBoss.Enabled = false
                if EMERALD.AutoBoss.Connection then
                    EMERALD.AutoBoss.Connection:Disconnect()
                end
            end
            StartAutoFarm()
            Rayfield:Notify({
                Title = "EMERALD HUB",
                Content = "Auto Farm ATIVADO",
                Duration = 3,
                Image = 4483362458,
            })
        else
            if EMERALD.AutoFarm.Connection then
                EMERALD.AutoFarm.Connection:Disconnect()
            end
            Rayfield:Notify({
                Title = "EMERALD HUB",
                Content = "Auto Farm DESATIVADO",
                Duration = 3,
                Image = 4483362458,
            })
        end
    end,
})

FarmTab:CreateSection("INFORMACOES")

FarmTab:CreateButton({
    Name = "Detectar Sea Atual",
    Callback = function()
        local sea = GetCurrentSea()
        local seaNames = {"", "First Sea", "Second Sea", "Third Sea"}
        Rayfield:Notify({
            Title = "EMERALD HUB",
            Content = "Voce esta na: " .. seaNames[sea],
            Duration = 5,
            Image = 4483362458,
        })
    end,
})

FarmTab:CreateButton({
    Name = "Ver Meu Nivel",
    Callback = function()
        local level = Player.Data.Level.Value
        local sea = GetCurrentSea()
        Rayfield:Notify({
            Title = "EMERALD HUB",
            Content = "Nivel: " .. level .. " | Sea: " .. sea,
            Duration = 5,
            Image = 4483362458,
        })
    end,
})

FarmTab:CreateSection("FUNCOES EXTRAS")

FarmTab:CreateToggle({
    Name = "Lock Target (Aimlock)",
    CurrentValue = false,
    Flag = "LockTarget",
    Callback = function(Value)
        EMERALD.LockTarget.Enabled = Value
        if Value then
            StartLockTarget()
            Rayfield:Notify({
                Title = "EMERALD HUB",
                Content = "Lock Target ATIVADO",
                Duration = 3,
                Image = 4483362458,
            })
        else
            if EMERALD.LockTarget.Connection then
                EMERALD.LockTarget.Connection:Disconnect()
            end
            Rayfield:Notify({
                Title = "EMERALD HUB",
                Content = "Lock Target DESATIVADO",
                Duration = 3,
                Image = 4483362458,
            })
        end
    end,
})

FarmTab:CreateToggle({
    Name = "Kill Aura (Auto Atacar)",
    CurrentValue = false,
    Flag = "KillAura",
    Callback = function(Value)
        -- Kill Aura usa o mesmo loop do Lock Target
        EMERALD.LockTarget.Enabled = Value
        if Value then
            StartLockTarget()
        else
            if EMERALD.LockTarget.Connection then
                EMERALD.LockTarget.Connection:Disconnect()
            end
        end
    end,
})

-- ============================================
-- ABA 2: AUTO BOSS
-- ============================================
local BossTab = Window:CreateTab("Auto Boss", 4483362458)

BossTab:CreateSection("AUTO BOSS")

BossTab:CreateToggle({
    Name = "Auto Boss",
    CurrentValue = false,
    Flag = "AutoBoss",
    Callback = function(Value)
        EMERALD.AutoBoss.Enabled = Value
        if Value then
            -- Desativar Auto Farm se estiver ativo
            if EMERALD.AutoFarm.Enabled then
                EMERALD.AutoFarm.Enabled = false
                if EMERALD.AutoFarm.Connection then
                    EMERALD.AutoFarm.Connection:Disconnect()
                end
            end
            StartAutoBoss()
            Rayfield:Notify({
                Title = "EMERALD HUB",
                Content = "Auto Boss ATIVADO",
                Duration = 3,
                Image = 4483362458,
            })
        else
            if EMERALD.AutoBoss.Connection then
                EMERALD.AutoBoss.Connection:Disconnect()
            end
            Rayfield:Notify({
                Title = "EMERALD HUB",
                Content = "Auto Boss DESATIVADO",
                Duration = 3,
                Image = 4483362458,
            })
        end
    end,
})

local currentSea = GetCurrentSea()
local bossList = BossDatabase[currentSea] or BossDatabase[1]

BossTab:CreateDropdown({
    Name = "Selecionar Boss",
    Options = bossList,
    CurrentOption = bossList[1],
    Flag = "SelectedBoss",
    Callback = function(Option)
        EMERALD.AutoBoss.SelectedBoss = Option
        Rayfield:Notify({
            Title = "EMERALD HUB",
            Content = "Boss: " .. Option,
            Duration = 3,
            Image = 4483362458,
        })
    end,
})

BossTab:CreateSection("BOSS POR SEA")

BossTab:CreateButton({
    Name = "Listar Bosses Sea 1",
    Callback = function()
        local bosses = table.concat(BossDatabase[1], ", ")
        Rayfield:Notify({
            Title = "EMERALD HUB",
            Content = "Sea 1: " .. bosses,
            Duration = 8,
            Image = 4483362458,
        })
    end,
})

BossTab:CreateButton({
    Name = "Listar Bosses Sea 2",
    Callback = function()
        local bosses = table.concat(BossDatabase[2], ", ")
        Rayfield:Notify({
            Title = "EMERALD HUB",
            Content = "Sea 2: " .. bosses,
            Duration = 8,
            Image = 4483362458,
        })
    end,
})

BossTab:CreateButton({
    Name = "Listar Bosses Sea 3",
    Callback = function()
        local bosses = table.concat(BossDatabase[3], ", ")
        Rayfield:Notify({
            Title = "EMERALD HUB",
            Content = "Sea 3: " .. bosses,
            Duration = 8,
            Image = 4483362458,
        })
    end,
})

-- ============================================
-- ABA 3: AUTO STATS
-- ============================================
local StatsTab = Window:CreateTab("Auto Stats", 4483362458)

StatsTab:CreateSection("AUTO STATS")

StatsTab:CreateToggle({
    Name = "Auto Stats",
    CurrentValue = false,
    Flag = "AutoStats",
    Callback = function(Value)
        EMERALD.AutoStats.Enabled = Value
        if Value then
            StartAutoStats()
            Rayfield:Notify({
                Title = "EMERALD HUB",
                Content = "Auto Stats ATIVADO",
                Duration = 3,
                Image = 4483362458,
            })
        else
            if EMERALD.AutoStats.Connection then
                EMERALD.AutoStats.Connection:Disconnect()
            end
            Rayfield:Notify({
                Title = "EMERALD HUB",
                Content = "Auto Stats DESATIVADO",
                Duration = 3,
                Image = 4483362458,
            })
        end
    end,
})

StatsTab:CreateDropdown({
    Name = "Atributo",
    Options = {"Melee", "Defense", "Sword", "Gun", "Devil Fruit"},
    CurrentOption = "Melee",
    Flag = "SelectedStat",
    Callback = function(Option)
        EMERALD.AutoStats.SelectedStat = Option
    end,
})

StatsTab:CreateSection("STATUS")

StatsTab:CreateButton({
    Name = "Ver Status Points",
    Callback = function()
        local points = Player.Data.Points.Value
        local melee = Player.Data.MeleeStat.Value
        local defense = Player.Data.DefenseStat.Value
        local sword = Player.Data.SwordStat.Value
        local gun = Player.Data.GunStat.Value
        local fruit = Player.Data.DemonFruitStat.Value
        
        local info = string.format(
            "Points: %d\nMelee: %d | Defense: %d\nSword: %d | Gun: %d\nFruit: %d",
            points, melee, defense, sword, gun, fruit
        )
        Rayfield:Notify({
            Title = "EMERALD HUB",
            Content = info,
            Duration = 8,
            Image = 4483362458,
        })
    end,
})

StatsTab:CreateButton({
    Name = "Reset Stats (Precisa Code)",
    Callback = function()
        Rayfield:Notify({
            Title = "EMERALD HUB",
            Content = "Compre Reset Stats no shop ou use codigo!",
            Duration = 5,
            Image = 4483362458,
        })
    end,
})

-- ============================================
-- ABA 4: AUTO HAKI
-- ============================================
local HakiTab = Window:CreateTab("Auto Haki", 4483362458)

HakiTab:CreateSection("AUTO HAKI")

HakiTab:CreateToggle({
    Name = "Auto Ken (Observation)",
    CurrentValue = false,
    Flag = "AutoKen",
    Callback = function(Value)
        EMERALD.AutoKen.Enabled = Value
        if Value then
            StartAutoKen()
            Rayfield:Notify({
                Title = "EMERALD HUB",
                Content = "Auto Ken ATIVADO",
                Duration = 3,
                Image = 4483362458,
            })
        else
            if EMERALD.AutoKen.Connection then
                EMERALD.AutoKen.Connection:Disconnect()
            end
            Rayfield:Notify({
                Title = "EMERALD HUB",
                Content = "Auto Ken DESATIVADO",
                Duration = 3,
                Image = 4483362458,
            })
        end
    end,
})

HakiTab:CreateToggle({
    Name = "Auto Buso (Armament)",
    CurrentValue = false,
    Flag = "AutoBuso",
    Callback = function(Value)
        EMERALD.AutoBuso.Enabled = Value
        if Value then
            StartAutoBuso()
            Rayfield:Notify({
                Title = "EMERALD HUB",
                Content = "Auto Buso ATIVADO",
                Duration = 3,
                Image = 4483362458,
            })
        else
            if EMERALD.AutoBuso.Connection then
                EMERALD.AutoBuso.Connection:Disconnect()
            end
            Rayfield:Notify({
                Title = "EMERALD HUB",
                Content = "Auto Buso DESATIVADO",
                Duration = 3,
                Image = 4483362458,
            })
        end
    end,
})

HakiTab:CreateSection("HAKI MANUAL")

HakiTab:CreateButton({
    Name = "Ativar Ken (Observation)",
    Callback = function()
        pcall(function()
            ReplicatedStorage.Remotes.CommF_:InvokeServer("ActivateInstinct")
        end)
        Rayfield:Notify({
            Title = "EMERALD HUB",
            Content = "Ken ativado manualmente!",
            Duration = 3,
            Image = 4483362458,
        })
    end,
})

HakiTab:CreateButton({
    Name = "Ativar Buso (Armament)",
    Callback = function()
        pcall(function()
            ReplicatedStorage.Remotes.CommF_:InvokeServer("Buso")
        end)
        Rayfield:Notify({
            Title = "EMERALD HUB",
            Content = "Buso ativado manualmente!",
            Duration = 3,
            Image = 4483362458,
        })
    end,
})

HakiTab:CreateButton({
    Name = "Ativar Haki Completo",
    Callback = function()
        pcall(function()
            ReplicatedStorage.Remotes.CommF_:InvokeServer("ActivateInstinct")
            ReplicatedStorage.Remotes.CommF_:InvokeServer("Buso")
        end)
        Rayfield:Notify({
            Title = "EMERALD HUB",
            Content = "Haki completo ativado!",
            Duration = 3,
            Image = 4483362458,
        })
    end,
})

-- ============================================
-- ABA 5: MOVEMENT
-- ============================================
local MoveTab = Window:CreateTab("Movement", 4483362458)

MoveTab:CreateSection("SPEED HACK")

MoveTab:CreateSlider({
    Name = "Walk Speed",
    Range = {16, 250},
    Increment = 1,
    Suffix = "studs",
    CurrentValue = 16,
    Flag = "WalkSpeed",
    Callback = function(Value)
        EMERALD.SpeedHack.Value = Value
        if IsAlive() and EMERALD.SpeedHack.Enabled then
            Humanoid.WalkSpeed = Value
        end
    end,
})

MoveTab:CreateToggle({
    Name = "Speed Hack",
    CurrentValue = false,
    Flag = "SpeedHack",
    Callback = function(Value)
        EMERALD.SpeedHack.Enabled = Value
        if Value then
            if IsAlive() then
                Humanoid.WalkSpeed = EMERALD.SpeedHack.Value
            end
        else
            if IsAlive() then
                Humanoid.WalkSpeed = 16
            end
        end
    end,
})

MoveTab:CreateSection("FLY")

MoveTab:CreateToggle({
    Name = "Fly",
    CurrentValue = false,
    Flag = "Fly",
    Callback = function(Value)
        EMERALD.Fly.Enabled = Value
        if Value then
            StartFly()
            Rayfield:Notify({
                Title = "EMERALD HUB",
                Content = "Fly ATIVADO (WASD + Space/Shift)",
                Duration = 3,
                Image = 4483362458,
            })
        else
            if EMERALD.Fly.Connection then
                EMERALD.Fly.Connection:Disconnect()
            end
            if IsAlive() then
                Humanoid.PlatformStand = false
            end
        end
    end,
})

MoveTab:CreateSlider({
    Name = "Fly Speed",
    Range = {10, 200},
    Increment = 5,
    Suffix = "speed",
    CurrentValue = 50,
    Flag = "FlySpeed",
    Callback = function(Value)
        EMERALD.Fly.Speed = Value
    end,
})

MoveTab:CreateSection("OUTROS")

MoveTab:CreateToggle({
    Name = "No Clip",
    CurrentValue = false,
    Flag = "NoClip",
    Callback = function(Value)
        EMERALD.NoClip.Enabled = Value
        if Value then
            StartNoClip()
        else
            if EMERALD.NoClip.Connection then
                EMERALD.NoClip.Connection:Disconnect()
            end
            -- Restaurar colisao
            if Character then
                for _, part in ipairs(Character:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.CanCollide = true
                    end
                end
            end
        end
    end,
})

MoveTab:CreateToggle({
    Name = "Infinite Jump",
    CurrentValue = false,
    Flag = "InfiniteJump",
    Callback = function(Value)
        EMERALD.InfiniteJump.Enabled = Value
        if Value then
            StartInfiniteJump()
        else
            if EMERALD.InfiniteJump.Connection then
                EMERALD.InfiniteJump.Connection:Disconnect()
            end
        end
    end,
})

MoveTab:CreateSlider({
    Name = "Jump Power",
    Range = {50, 300},
    Increment = 10,
    Suffix = "power",
    CurrentValue = 50,
    Flag = "JumpPower",
    Callback = function(Value)
        if IsAlive() then
            Humanoid.JumpPower = Value
        end
    end,
})

-- ============================================
-- ABA 6: COLLECT
-- ============================================
local CollectTab = Window:CreateTab("Collect", 4483362458)

CollectTab:CreateSection("AUTO COLLECT")

CollectTab:CreateToggle({
    Name = "Auto Collect Chests",
    CurrentValue = false,
    Flag = "AutoChest",
    Callback = function(Value)
        EMERALD.AutoChest.Enabled = Value
        if Value then
            StartAutoChest()
        else
            if EMERALD.AutoChest.Connection then
                EMERALD.AutoChest.Connection:Disconnect()
            end
        end
    end,
})

CollectTab:CreateToggle({
    Name = "Auto Collect Fruits",
    CurrentValue = false,
    Flag = "AutoFruit",
    Callback = function(Value)
        EMERALD.AutoFruit.Enabled = Value
        if Value then
            StartAutoFruit()
        else
            if EMERALD.AutoFruit.Connection then
                EMERALD.AutoFruit.Connection:Disconnect()
            end
        end
    end,
})

CollectTab:CreateToggle({
    Name = "Auto Collect Gems",
    CurrentValue = false,
    Flag = "AutoGem",
    Callback = function(Value)
        EMERALD.AutoGem.Enabled = Value
        if Value then
            StartAutoGem()
        else
            if EMERALD.AutoGem.Connection then
                EMERALD.AutoGem.Connection:Disconnect()
            end
        end
    end,
})

CollectTab:CreateSection("COLLECT MANUAL")

CollectTab:CreateButton({
    Name = "Coletar Fruta Proxima",
    Callback = function()
        if not IsAlive() then return end
        
        local found = false
        for _, obj in ipairs(Workspace:GetDescendants()) do
            if obj:IsA("Tool") and string.find(string.lower(obj.Name), "fruit") then
                local dist = (HumanoidRootPart.Position - obj.Position).Magnitude
                if dist < 200 then
                    found = true
                    Teleport(obj.Position)
                    task.wait(0.3)
                    local handle = obj:FindFirstChild("Handle") or obj:FindFirstChildWhichIsA("BasePart")
                    if handle then
                        firetouchinterest(HumanoidRootPart, handle, 0)
                        firetouchinterest(HumanoidRootPart, handle, 1)
                    end
                    Rayfield:Notify({
                        Title = "EMERALD HUB",
                        Content = "Fruta coletada!",
                        Duration = 3,
                        Image = 4483362458,
                    })
                    break
                end
            end
        end
        
        if not found then
            Rayfield:Notify({
                Title = "EMERALD HUB",
                Content = "Nenhuma fruta proxima!",
                Duration = 3,
                Image = 4483362458,
            })
        end
    end,
})

-- ============================================
-- ABA 7: ESP
-- ============================================
local ESPTab = Window:CreateTab("ESP", 4483362458)

ESPTab:CreateSection("ESP")

ESPTab:CreateToggle({
    Name = "ESP (Ver inimigos/jogadores)",
    CurrentValue = false,
    Flag = "ESP",
    Callback = function(Value)
        EMERALD.ESP.Enabled = Value
        if Value then
            StartESP()
            Rayfield:Notify({
                Title = "EMERALD HUB",
                Content = "ESP ATIVADO",
                Duration = 3,
                Image = 4483362458,
            })
        else
            if EMERALD.ESP.Connection then
                EMERALD.ESP.Connection:Disconnect()
            end
            -- Limpar ESP
            local espFolder = Workspace:FindFirstChild("EMERALD_ESP")
            if espFolder then
                espFolder:Destroy()
            end
            Rayfield:Notify({
                Title = "EMERALD HUB",
                Content = "ESP DESATIVADO",
                Duration = 3,
                Image = 4483362458,
            })
        end
    end,
})

ESPTab:CreateSection("VISUAL")

ESPTab:CreateToggle({
    Name = "Full Bright (Claridade)",
    CurrentValue = false,
    Flag = "FullBright",
    Callback = function(Value)
        if Value then
            Lighting.Brightness = 2
            Lighting.ClockTime = 14
            Lighting.FogEnd = 100000
            Lighting.GlobalShadows = false
            Lighting.OutdoorAmbient = Color3.fromRGB(128, 128, 128)
            Rayfield:Notify({
                Title = "EMERALD HUB",
                Content = "Full Bright ATIVADO",
                Duration = 3,
                Image = 4483362458,
            })
        else
            Lighting.Brightness = 1
            Lighting.ClockTime = 14
            Lighting.FogEnd = 10000
            Lighting.GlobalShadows = true
            Lighting.OutdoorAmbient = Color3.fromRGB(70, 70, 70)
        end
    end,
})

ESPTab:CreateSlider({
    Name = "Field of View",
    Range = {70, 120},
    Increment = 1,
    Suffix = "FOV",
    CurrentValue = 70,
    Flag = "FOV",
    Callback = function(Value)
        Workspace.CurrentCamera.FieldOfView = Value
    end,
})

-- ============================================
-- ABA 8: TELEPORTS
-- ============================================
local TeleTab = Window:CreateTab("Teleports", 4483362458)

TeleTab:CreateSection("SEA TELEPORT")

TeleTab:CreateButton({
    Name = "Teleport para Sea 1",
    Callback = function()
        EMERALD.ManualTeleport = true
        pcall(function()
            ReplicatedStorage.Remotes.CommF_:InvokeServer("TravelMain")
        end)
        task.wait(2)
        EMERALD.ManualTeleport = false
        Rayfield:Notify({
            Title = "EMERALD HUB",
            Content = "Teleportado para Sea 1",
            Duration = 3,
            Image = 4483362458,
        })
    end,
})

TeleTab:CreateButton({
    Name = "Teleport para Sea 2",
    Callback = function()
        EMERALD.ManualTeleport = true
        pcall(function()
            ReplicatedStorage.Remotes.CommF_:InvokeServer("TravelDressrosa")
        end)
        task.wait(2)
        EMERALD.ManualTeleport = false
        Rayfield:Notify({
            Title = "EMERALD HUB",
            Content = "Teleportado para Sea 2",
            Duration = 3,
            Image = 4483362458,
        })
    end,
})

TeleTab:CreateButton({
    Name = "Teleport para Sea 3",
    Callback = function()
        EMERALD.ManualTeleport = true
        pcall(function()
            ReplicatedStorage.Remotes.CommF_:InvokeServer("TravelZou")
        end)
        task.wait(2)
        EMERALD.ManualTeleport = false
        Rayfield:Notify({
            Title = "EMERALD HUB",
            Content = "Teleportado para Sea 3",
            Duration = 3,
            Image = 4483362458,
        })
    end,
})

TeleTab:CreateSection("ILHAS - SEA 1")

local sea1Islands = {
    "Start Island", "Jungle", "Pirate Village", "Desert", 
    "Snow Island", "Marine Fortress", "Skylands", "Prison", 
    "Colosseum", "Magma Village", "Underwater City", "Fountain City",
    "Marine Base"
}

for _, island in ipairs(sea1Islands) do
    TeleTab:CreateButton({
        Name = "[S1] " .. island,
        Callback = function()
            local pos = IslandPositions[island]
            if pos and IsAlive() then
                Teleport(pos)
                Rayfield:Notify({
                    Title = "EMERALD HUB",
                    Content = "Teleportado para " .. island,
                    Duration = 2,
                    Image = 4483362458,
                })
            end
        end,
    })
end

TeleTab:CreateSection("ILHAS - SEA 2")

local sea2Islands = {
    "Kingdom of Rose", "Green Zone", "Graveyard", "Snow Mountain", 
    "Hot and Cold", "Cursed Ship", "Ice Castle", "Forgotten Island", 
    "Dark Arena", "Swan Room", "Factory", "Usoapp Island"
}

for _, island in ipairs(sea2Islands) do
    TeleTab:CreateButton({
        Name = "[S2] " .. island,
        Callback = function()
            local pos = IslandPositions[island]
            if pos and IsAlive() then
                Teleport(pos)
                Rayfield:Notify({
                    Title = "EMERALD HUB",
                    Content = "Teleportado para " .. island,
                    Duration = 2,
                    Image = 4483362458,
                })
            end
        end,
    })
end

TeleTab:CreateSection("ILHAS - SEA 3")

local sea3Islands = {
    "Port Town", "Hydra Island", "Great Tree", "Castle on the Sea", 
    "Floating Turtle", "Haunted Castle", "Cake Island", "Tiki Outpost", 
    "Ice Cream Island", "Chocolate Island"
}

for _, island in ipairs(sea3Islands) do
    TeleTab:CreateButton({
        Name = "[S3] " .. island,
        Callback = function()
            local pos = IslandPositions[island]
            if pos and IsAlive() then
                Teleport(pos)
                Rayfield:Notify({
                    Title = "EMERALD HUB",
                    Content = "Teleportado para " .. island,
                    Duration = 2,
                    Image = 4483362458,
                })
            end
        end,
    })
end

-- ============================================
-- ABA 9: SERVER
-- ============================================
local ServerTab = Window:CreateTab("Server", 4483362458)

ServerTab:CreateSection("SERVER OPTIONS")

ServerTab:CreateButton({
    Name = "Hop Server (Trocar Servidor)",
    Callback = function()
        HopServer()
    end,
})

ServerTab:CreateButton({
    Name = "Rejoin Server (Re-entrar)",
    Callback = function()
        TeleportService:Teleport(game.PlaceId, Player)
    end,
})

ServerTab:CreateButton({
    Name = "Copy Job ID",
    Callback = function()
        pcall(function()
            setclipboard(game.JobId)
            Rayfield:Notify({
                Title = "EMERALD HUB",
                Content = "Job ID copiado!",
                Duration = 5,
                Image = 4483362458,
            })
        end)
    end,
})

ServerTab:CreateButton({
    Name = "Copy Place ID",
    Callback = function()
        pcall(function()
            setclipboard(tostring(game.PlaceId))
            Rayfield:Notify({
                Title = "EMERALD HUB",
                Content = "Place ID copiado!",
                Duration = 5,
                Image = 4483362458,
            })
        end)
    end,
})

ServerTab:CreateButton({
    Name = "Server Info",
    Callback = function()
        local info = string.format(
            "Place: %d\nPlayers: %d/%d\nPing: %dms\nFPS: %d",
            game.PlaceId,
            #Players:GetPlayers(),
            Players.MaxPlayers,
            math.floor(Player:GetNetworkPing() * 1000),
            math.floor(1 / RunService.Heartbeat:Wait())
        )
        Rayfield:Notify({
            Title = "EMERALD HUB",
            Content = info,
            Duration = 8,
            Image = 4483362458,
        })
    end,
})

ServerTab:CreateSection("RECOMMENDED SERVERS")

ServerTab:CreateButton({
    Name = "Join Small Server",
    Callback = function()
        HopServer()
    end,
})

-- ============================================
-- ABA 10: CONFIG
-- ============================================
local ConfigTab = Window:CreateTab("Config", 4483362458)

ConfigTab:CreateSection("SAVE/LOAD")

ConfigTab:CreateButton({
    Name = "Save Configuration",
    Callback = function()
        Rayfield:SaveConfiguration()
        Rayfield:Notify({
            Title = "EMERALD HUB",
            Content = "Configuracoes salvas!",
            Duration = 3,
            Image = 4483362458,
        })
    end,
})

ConfigTab:CreateButton({
    Name = "Load Configuration",
    Callback = function()
        Rayfield:LoadConfiguration()
        Rayfield:Notify({
            Title = "EMERALD HUB",
            Content = "Configuracoes carregadas!",
            Duration = 3,
            Image = 4483362458,
        })
    end,
})

ConfigTab:CreateSection("RESET")

ConfigTab:CreateButton({
    Name = "Reset All Functions",
    Callback = function()
        -- Desativar todos os loops
        for name, module in pairs(EMERALD) do
            if module.Connection then
                module.Connection:Disconnect()
                module.Connection = nil
            end
            if module.Enabled ~= nil then
                module.Enabled = false
            end
        end
        
        EMERALD.ManualTeleport = false
        
        -- Restaurar velocidade
        if IsAlive() then
            Humanoid.WalkSpeed = 16
            Humanoid.JumpPower = 50
            Humanoid.PlatformStand = false
        end
        
        -- Restaurar colisao
        if Character then
            for _, part in ipairs(Character:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = true
                end
            end
        end
        
        -- Limpar ESP
        local espFolder = Workspace:FindFirstChild("EMERALD_ESP")
        if espFolder then
            espFolder:Destroy()
        end
        
        -- Restaurar lighting
        Lighting.Brightness = 1
        Lighting.FogEnd = 10000
        Lighting.GlobalShadows = true
        
        Rayfield:Notify({
            Title = "EMERALD HUB",
            Content = "Todas as funcoes foram resetadas!",
            Duration = 5,
            Image = 4483362458,
        })
    end,
})

ConfigTab:CreateSection("INFORMATION")

ConfigTab:CreateLabel("EMERALD HUB v3.0")
ConfigTab:CreateLabel("by Emerald Team")
ConfigTab:CreateLabel("Rayfield Gen2 Interface")
ConfigTab:CreateLabel("Universal Executor Support")
ConfigTab:CreateLabel("Current Sea: " .. GetCurrentSea())

-- ============================================
-- INICIALIZACAO FINAL
-- ============================================
Rayfield:LoadConfiguration()

-- Anti-AFK
Player.Idled:Connect(function()
    VirtualUser:CaptureController()
    VirtualUser:ClickButton2(Vector2.new())
end)

-- Manter speed hack e fly
RunService.Heartbeat:Connect(function()
    if EMERALD.SpeedHack.Enabled and IsAlive() and not EMERALD.Fly.Enabled then
        Humanoid.WalkSpeed = EMERALD.SpeedHack.Value
    end
end)

-- Restaurar colisao quando NoClip desativado
RunService.Heartbeat:Connect(function()
    if not EMERALD.NoClip.Enabled and Character then
        for _, part in ipairs(Character:GetDescendants()) do
            if part:IsA("BasePart") and not part.CanCollide then
                -- So restaura se nao estiver em Fly
                if not EMERALD.Fly.Enabled then
                    part.CanCollide = true
                end
            end
        end
    end
end)

-- Restaurar PlatformStand quando Fly desativado
RunService.Heartbeat:Connect(function()
    if not EMERALD.Fly.Enabled and IsAlive() and Humanoid.PlatformStand then
        Humanoid.PlatformStand = false
    end
end)

-- Console message
print([[
====================================================
    EMERALD HUB v3.0 - CARREGADO COM SUCESSO
    Blox Fruits - Universal Edition
    Rayfield Gen2 Interface
    
    Compativel com TODOS os executores:
    Delta, Arceus X, Codex, Hydrogen,
    Fluxus, Solara, Electron, Vega X, Krnl
    
    Sea detectada: ]] .. GetCurrentSea() .. [[
    Nivel: ]] .. Player.Data.Level.Value .. [[
====================================================
]])

-- Notificacao final
Rayfield:Notify({
    Title = "EMERALD HUB v3.0",
    Content = "Tudo pronto! Nivel: " .. Player.Data.Level.Value .. " | Sea: " .. GetCurrentSea() .. " | Funcoes: 50+",
    Duration = 6,
    Image = 4483362458,
})
