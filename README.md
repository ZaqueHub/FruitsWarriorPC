local _wait = task.wait
repeat _wait() until game:IsLoaded()
local _env = getgenv and getgenv() or {}

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local VirtualUser = game:GetService("VirtualUser")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")

local Player = Players.LocalPlayer

local rs_Monsters = ReplicatedStorage:WaitForChild("MonsterSpawn")
local Modules = ReplicatedStorage:WaitForChild("ModuleScript")
local OtherEvent = ReplicatedStorage:WaitForChild("OtherEvent")
local Monsters = workspace:WaitForChild("Monster")

local MQuestSettings = require(Modules:WaitForChild("Quest_Settings"))
local MSetting = require(Modules:WaitForChild("Setting"))

local NPCs = workspace:WaitForChild("NPCs")
local Raids = workspace:WaitForChild("Raids")
local Location = workspace:WaitForChild("Location")
local Region = workspace:WaitForChild("Region")
local Island = workspace:WaitForChild("Island")

local Quests_Npc = NPCs:WaitForChild("Quests_Npc")
local EnemyLocation = Location:WaitForChild("Enemy_Location")
local QuestLocation = Location:WaitForChild("QuestLocaion")

local Items = Player:WaitForChild("Items")
local QuestFolder = Player:WaitForChild("QuestFolder")
local Ability = Player:WaitForChild("Ability")
local PlayerData = Player:WaitForChild("PlayerData")
local PlayerLevel = PlayerData:WaitForChild("Level")

local sethiddenproperty = sethiddenproperty or (function()end)

local CFrame_Angles = CFrame.Angles
local CFrame_new = CFrame.new
local Vector3_new = Vector3.new

local _huge = math.huge

task.spawn(function()
  if not _env.LoadedHideUsername then
    _env.LoadedHideUsername = true
    local Label = Player.PlayerGui.MainGui.PlayerName
    
    local function Update()
      local Level = PlayerLevel.Value
      local IsMax = Level >= MSetting.Setting.MaxLevel
      Label.Text = ("%s • Lv. %i%s"):format("Anonymous", Level, IsMax and " (Max)" or "")
    end
    
    Label:GetPropertyChangedSignal("Text"):Connect(Update)Update()
  end
end)

local Loaded, Funcs, Folders = {}, {}, {} do
  Loaded.ItemsPrice = {
    Aura = function()
      return Funcs:GetMaterial("Meme Cube") > 0 and Funcs:GetData("Money") >= 10000000 -- 1x Meme Cube, $10.000.000
    end,
    FlashStep = function()
      return Funcs:GetData("Money") >= 100000 -- $100.000
    end,
    Instinct = function()
      return Funcs:GetData("Money") >= 2500000 -- $2.500.000
    end
  }
  Loaded.Shop = {
    {"Weapons", {
      {"Buy Katana", "$5.000 Money", {"Weapon_Seller", "Doge"}},
      {"Buy Hanger", "$25.000 Money", {"Weapon_Seller", "Hanger"}},
      {"Buy Flame Katana", "1x Cheems Cola and $50.000", {"Weapon_Seller", "Cheems"}},
      {"Buy Banana", "1x Cat Food and $350.000", {"Weapon_Seller", "Smiling Cat"}},
      {"Buy Bonk", "5x Money Bags and $1.000.000", {"Weapon_Seller", "Meme Man"}},
      {"Buy Pumpkin", "1x Nugget Man and $3.500.000", {"Weapon_Seller", "Gravestone"}},
      {"Buy Popcat", "10.000 Pops Clicker", {"Weapon_Seller", "Ohio Popcat"}}
    }},
    {"Ability", {
      {"Buy Flash Step", "$100.000 Money", {"Ability_Teacher", "Giga Chad"}},
      {"Buy Instinct", "$2.500.000 Money", {"Ability_Teacher", "Nugget Man"}},
      {"Buy Aura", "1x Meme Cube and $10.000.000", {"Ability_Teacher", "Aura Master"}}
    }},
    {"Fighting Style", {
      {"Buy Combat", "$0 Money", {"FightingStyle_Teacher", "Maxwell"}},
      {"Buy Baller", "10x Balls and $10.000.000", {"FightingStyle_Teacher", "Baller"}}
    }}
  }
  Loaded.WeaponsList = { "Fight", "Power", "Weapon" }
  Loaded.EnemeiesList = {}
  Loaded.EnemiesSpawns = {}
  Loaded.EnemiesQuests = {}
  Loaded.Islands = {}
  Loaded.Quests = {}
  
  local function RedeemCode(Code)
    return OtherEvent.MainEvents.Code:InvokeServer(Code)
  end
  
  Funcs.RAllCodes = function(self)
    if Modules:FindFirstChild("CodeList") then
      local List = require(Modules.CodeList)
      for Code, Info in pairs(type(List) == "table" and List or {}) do
        if type(Code) == "string" and type(Info) == "table" and Info.Status then RedeemCode(Code) end
      end
    end
  end
  
  Funcs.GetPlayerLevel = function(self)
    return PlayerLevel.Value
  end
  
  Funcs.GetCurrentQuest = function(self)
    for _,Quest in pairs(Loaded.Quests) do
      if Quest.Level <= self:GetPlayerLevel() and not Quest.RaidBoss and not Quest.SpecialQuest then
        return Quest
      end
    end
  end
  
  Funcs.CheckQuest = function(self)
    for _,v in ipairs(QuestFolder:GetChildren()) do
      if v.Target.Value ~= "None" then
        return v
      end
    end
  end
  
  Funcs.VerifySword = function(self, SName)
    local Swords = Items.Weapon
    return Swords:FindFirstChild(SName) and Swords[SName].Value > 0
  end
  
  Funcs.VerifyAccessory = function(self, AName)
    local Accessories = Items.Accessory
    return Accessories:FindFirstChild(AName) and Accessories[AName].Value > 0
  end
  
  Funcs.GetMaterial = function(self, MName)
    local ItemStorage = Items.ItemStorage
    return ItemStorage:FindFirstChild(MName) and ItemStorage[MName].Value or 0
  end
  
  Funcs.AbilityUnlocked = function(self, Ablt)
    return Ability:FindFirstChild(Ablt) and Ability[Ablt].Value
  end
  
  Funcs.CanBuy = function(self, Item)
    if Loaded.ItemsPrice[Item] then
      return Loaded.ItemsPrice[Item]()
    end
    return false
  end
  
  Funcs.GetData = function(self, Data)
    return PlayerData:FindFirstChild(Data) and PlayerData[Data].Value or 0
  end
  
  for Npc,Quest in pairs(MQuestSettings) do
    if QuestLocation:FindFirstChild(Npc) then
      table.insert(Loaded.Quests, {
        RaidBoss = Quest.Raid_Boss,
        SpecialQuest = Quest.Special_Quest,
        QuestPos = QuestLocation[Npc].CFrame,
        EnemyPos = EnemyLocation[Quest.Target].CFrame,
        Level = Quest.LevelNeed,
        Enemy = Quest.Target,
        NpcName = Npc
      })
    end
  end
  
  table.sort(Loaded.Quests, function(a, b) return a.Level > b.Level end)
  for _,v in ipairs(Loaded.Quests) do
    table.insert(Loaded.EnemeiesList, v.Enemy)Loaded.EnemiesQuests[v.Enemy] = v.NpcName
  end
end

local Settings = Settings or {} do
  Settings.AutoStats_Points = 1
  Settings.BringMobs = true
  Settings.FarmDistance = 6
  Settings.ViewHitbox = false
  Settings.AntiAFK = true
  Settings.AutoHaki = true
  Settings.AutoClick = true
  Settings.ToolFarm = "Fight" -- [[ "Fight", "Power", "Weapon" ]]
  Settings.FarmCFrame = CFrame_new(0, Settings.FarmDistance, 0) * CFrame_Angles(math.rad(-90), 0, 0)
end

local function PlayerClick()
  local Char = Player.Character
  if Char then
    if Settings.AutoClick then
      VirtualUser:CaptureController()
      VirtualUser:Button1Down(Vector2.new(1e4, 1e4))
    end
    if Settings.AutoHaki and Char:FindFirstChild("AuraColor_Folder") and Funcs:AbilityUnlocked("Aura") then
      if #Char.AuraColor_Folder:GetChildren() < 1 then
        OtherEvent.MainEvents.Ability:InvokeServer("Aura")
      end
    end
  end
end

local function IsAlive(Char)
  local Hum = Char and Char:FindFirstChild("Humanoid")
  return Hum and Hum.Health > 0
end

local function GetNextEnemie(EnemieName)
  for _,v in ipairs(Monsters:GetChildren()) do
    if (not EnemieName or v.Name == EnemieName) and IsAlive(v) then
      return v
    end
  end
  return false
end

local function GoTo(CFrame, Move)
  local Char = Player.Character
  if IsAlive(Char) then
    return Move and ( Char:MoveTo(CFrame.p) or true ) or Char:SetPrimaryPartCFrame(CFrame)
  end
end

local function EquipWeapon()
  local Backpack, Char = Player:FindFirstChild("Backpack"), Player.Character
  if IsAlive(Char) and Backpack then
    for _,v in ipairs(Backpack:GetChildren()) do
      if v:IsA("Tool") and v.ToolTip:find(Settings.ToolFarm) then
        Char.Humanoid:EquipTool(v)
      end
    end
  end
end

local function BringMobsTo(_Enemie, TargetCFrame, SBring)
  local GatherPosition = TargetCFrame.Position -- Position to gather mobs
  for _,v in ipairs(Monsters:GetChildren()) do
      if (SBring or v.Name == _Enemie) and IsAlive(v) then
          local PP, Hum = v.PrimaryPart, v.Humanoid
          if PP and (PP.Position - TargetCFrame.Position).Magnitude < 500 then
              Hum.WalkSpeed = 0
              Hum:ChangeState(Enum.HumanoidStateType.Physics)
              
              -- Move the mobs to the gather position
              PP.CFrame = CFrame.new(GatherPosition)
              PP.CanCollide = false
              PP.Velocity = Vector3.new(0, 0, 0) 
              PP.RotVelocity = Vector3.new(0, 0, 0)
              
              PP.Size = Vector3.new(50, 50, 50)
          end
      end
  end
  return pcall(sethiddenproperty, Player, "SimulationRadius", _huge)
end



local function KillMonster(_Enemie, SBring)
  local Enemy = typeof(_Enemie) == "Instance" and _Enemie or GetNextEnemie(_Enemie)
  if IsAlive(Enemy) and Enemy.PrimaryPart then
    GoTo(Enemy.PrimaryPart.CFrame * Settings.FarmCFrame)EquipWeapon()
    if not Enemy:FindFirstChild("Reverse_Mark") then PlayerClick() end
    if Settings.BringMobs then BringMobsTo(_Enemie, Enemy.PrimaryPart.CFrame, SBring) end
    return true
  end
end

local function TakeQuest(QuestName, CFrame, Wait)
  local QuestGiver = Quests_Npc:FindFirstChild(QuestName)
  if QuestGiver and Player:DistanceFromCharacter(QuestGiver.WorldPivot.p) < 5 then
    return fireproximityprompt(QuestGiver.Block.QuestPrompt), _wait(Wait or 0.1)
  end
  GoTo(CFrame or QuestLocation[QuestName].CFrame)
end

local function ClearQuests(Ignore)
  for _,v in ipairs(QuestFolder:GetChildren()) do
    if v.QuestGiver.Value ~= Ignore and v.Target.Value ~= "None" then
      OtherEvent.QuestEvents.Quest:FireServer("Abandon_Quest", { QuestSlot = v.Name })
    end
  end
end

local function GetRaidEnemies()
  for _,v in ipairs(Monsters:GetChildren()) do
    if v:GetAttribute("Raid_Enemy") and IsAlive(v) then
      return v
    end
  end
end

local function GetRaidMap()
  for _,v in ipairs(Raids:GetChildren()) do
    if v.Joiners:FindFirstChild(Player.Name) then
      return v
    end
  end
end

local function VerifyQuest(QName)
  local Quest = Funcs:CheckQuest()
  return Quest and Quest.QuestGiver.Value == QName
end

_env.FarmFuncs = {
  {"_Floppa Sword", (function()
    if not Funcs:VerifySword("Floppa") then
      if VerifyQuest("Cool Floppa Quest") then
        GoTo(CFrame_new(794, -31, -440))
        fireproximityprompt(Island.FloppaIsland["Lava Floppa"].ClickPart.ProximityPrompt)
      else
        ClearQuests("Cool Floppa Quest")
        TakeQuest("Cool Floppa Quest", CFrame_new(758, -31, -424))
      end
      return true
    end
  end)},
  {"Meme Beast", (function()
    local MemeBeast = Monsters:FindFirstChild("Meme Beast") or rs_Monsters:FindFirstChild("Meme Beast")
    if MemeBeast then
      GoTo(MemeBeast.WorldPivot)EquipWeapon()PlayerClick()
      return true
    end
  end)},
  {"Lord Sus", (function()
    local LordSus = Monsters:FindFirstChild("Lord Sus") or rs_Monsters:FindFirstChild("Lord Sus")
    if LordSus then
      if not VerifyQuest("Floppa Quest 32") and Funcs:GetPlayerLevel() >= 1550 then
        ClearQuests("Floppa Quest 32")TakeQuest("Floppa Quest 32", nil, 1)
      else
        KillMonster(LordSus)
      end
      return true
    elseif Funcs:GetMaterial("Sussy Orb") > 0 then
      if Player:DistanceFromCharacter(Vector3_new(6644, -95, 4811)) < 5 then
        fireproximityprompt(Island.ForgottenIsland.Summon3.Summon.SummonPrompt)
      else GoTo(CFrame_new(6644, -95, 4811)) end
      return true
    end
  end)},
  {"Evil Noob", (function()
    local EvilNoob = Monsters:FindFirstChild("Evil Noob") or rs_Monsters:FindFirstChild("Evil Noob")
    if EvilNoob then
      if not VerifyQuest("Floppa Quest 29") and Funcs:GetPlayerLevel() >= 1400 then
        ClearQuests("Floppa Quest 29")TakeQuest("Floppa Quest 29", nil, 1)
      else
        KillMonster(EvilNoob)
      end
      return true
    elseif Funcs:GetMaterial("Noob Head") > 0 then
      if Player:DistanceFromCharacter(Vector3_new(-2356, -81, 3180)) < 5 then
        fireproximityprompt(Island.MoaiIsland.Summon2.Summon.SummonPrompt)
      else GoTo(CFrame_new(-2356, -81, 3180)) end
      return true
    end
  end)},
  {"Giant Pumpkin", (function()
    local Pumpkin = Monsters:FindFirstChild("Giant Pumpkin") or rs_Monsters:FindFirstChild("Giant Pumpkin")
    if Pumpkin then
      if not VerifyQuest("Floppa Quest 23") and Funcs:GetPlayerLevel() >= 1100 then
        ClearQuests("Floppa Quest 23")TakeQuest("Floppa Quest 23", nil, 1)
      else
        KillMonster(Pumpkin)
      end
      return true
    elseif Funcs:GetMaterial("Flame Orb") > 0 then
      if Player:DistanceFromCharacter(Vector3_new(-1180, -93, 1462)) < 5 then
        fireproximityprompt(Island.PumpkinIsland.Summon1.Summon.SummonPrompt)
      else GoTo(CFrame_new(-1180, -93, 1462)) end
      return true
    end
  end)},
  {"Race V2 Orb", (function()
    if Funcs:GetPlayerLevel() >= 500 then
      local Quest, Enemy = "Dancing Banana Quest", "Sogga"
      if VerifyQuest(Quest) then
        if KillMonster(Enemy) then else GoTo(EnemyLocation[Enemy].CFrame) end
      else ClearQuests(Quest)TakeQuest(Quest, CFrame_new(-2620, -80, -2001)) end
      return true
    end
  end)},
  {"Level Farm", (function()
    local Quest, QuestChecker = Funcs:GetCurrentQuest(), Funcs:CheckQuest()
    if Quest then
      if QuestChecker then
        local _QuestName = QuestChecker.QuestGiver.Value
        if _QuestName == Quest.NpcName then
          if KillMonster(Quest.Enemy) then else GoTo(Quest.EnemyPos) end
        else
          if KillMonster(QuestChecker.Target.Value) then else GoTo(QuestLocation[_QuestName].CFrame) end
        end
      else TakeQuest(Quest.NpcName) end
    end
    return true
  end)},
  {"Raid Farm", (function()
    if Funcs:GetPlayerLevel() >= 1000 then
      local RaidMap = GetRaidMap()
      if RaidMap then
        if RaidMap:GetAttribute("Starting") ~= 0 then
          OtherEvent.MiscEvents.StartRaid:FireServer("Start")_wait(1)
        else
          local Enemie = GetRaidEnemies()
          if Enemie then KillMonster(Enemie, true) else
            local Spawn = RaidMap:FindFirstChild("Spawn_Location")
            if Spawn then GoTo(Spawn.CFrame) end
          end
        end
      else
        local Raid = Region:FindFirstChild("RaidArea")
        if Raid then GoTo(CFrame_new(Raid.Position)) end
      end
      return true
    end
  end)},
  {"FS Enemie", (function()
    local Enemy = _env.SelecetedEnemie
    local Quest = Loaded.EnemiesQuests[Enemy]
    if VerifyQuest(Quest) or not _env["FS Take Quest"] then
      if KillMonster(Enemy) then else GoTo(EnemyLocation[Enemy].CFrame) end
    else ClearQuests(Quest)TakeQuest(Quest) end
    return true
  end)},
  {"Nearest Farm", (function() return KillMonster(GetNextEnemie()) end)}
}

if not _env.LoadedFarm then
  _env.LoadedFarm = true
  task.spawn(function()
    while _wait() do
      for _,f in _env.FarmFuncs do
        if _env[f[1]] then local s,r=pcall(f[2])if s and r then break end;end
      end
    end
  end)
end

local Fluent = loadstring(game:HttpGet("https://raw.githubusercontent.com/Tienvn123tkvn/Test/main/ZINERHUB_Ui.lua"))()
local SaveManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/Tienvn123tkvn/Test/main/ZierhubManager.lua"))()
local InterfaceManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/Tienvn123tkvn/Test/main/ZierhubfaceManager.lua"))()
do
local Window = Fluent:CreateWindow({
    Title = "Shiny Hub",
    SubTitle = "Tempest",
    TabWidth = 160,
    Size = UDim2.fromOffset(500, 320),
    Acrylic = true,
    Theme = "Black",
    MinimizeKey = Enum.KeyCode.End
})

local Tabs = {
    profile = Window:AddTab({ Title = "Information", Icon = "scan-face" }),
    Main = Window:AddTab({ Title = "Main Farm", Icon = "home" }),
    Boss = Window:AddTab({ Title = "Boss Farm", Icon = "home" }),
    Misc = Window:AddTab({ Title = "Misc Farm", Icon = "list-plus" }),
    Tele = Window:AddTab({ Title = "Teleport", Icon = "palmtree" }),
    Setting = Window:AddTab({ Title = "Settings", Icon = "settings" }),
}
local Options = Fluent.Options

Tabs.profile:AddParagraph({
    Title = "Owner",
    Content = "Siesta"
})

Tabs.profile:AddButton({
Title = "Discord Shiny Hub",
Description = "Joinnnn",
Callback = function()
    setclipboard("https://discord.gg/mvrtGqjT38")
end
})     

-- Toggle for Farm Level
local ToggleLevel = Tabs.Main:AddToggle("ToggleLevel", {
    Title = "Farm Level",
    Description = "",
    Default = false
})

ToggleLevel:OnChanged(function(Value)
    _env["Level Farm"] = Value -- Activate or deactivate the Level Farm function based on toggle
end)

-- Toggle for Bring Mob
local ToggleBring = Tabs.Setting:AddToggle("ToggleBring", {
    Title = "Bring Mob",
    Description = "Turn on auto skill for fast farm",
    Default = true
})


local ToggleSkill = Tabs.Setting:AddToggle("ToggleSkill", {
  Title = "Spam Skill",
  Description = "",
  Default = false
})

-- Variable to control AutoSkill state
local AutoSkill = false

-- Connect the ToggleSkill to control AutoSkill
ToggleSkill:OnChanged(function(value)
  AutoSkill = value
end)

-- Auto skill function
spawn(function(AutoSkill)
  while wait() do
      pcall(function()
          if AutoSkill then
              if Skillz then
                  game:service('VirtualInputManager'):SendKeyEvent(true, "Z", false, game)
                  wait(.1)
                  game:service('VirtualInputManager'):SendKeyEvent(false, "Z", false, game)
              end
              if Skillx then
                  game:service('VirtualInputManager'):SendKeyEvent(true, "X", false, game)
                  wait(.1)
                  game:service('VirtualInputManager'):SendKeyEvent(false, "X", false, game)
              end
              if Skillc then
                  game:service('VirtualInputManager'):SendKeyEvent(true, "C", false, game)
                  wait(.1)
                  game:service('VirtualInputManager'):SendKeyEvent(false, "C", false, game)
              end
              if Skillv then
                  game:service('VirtualInputManager'):SendKeyEvent(true, "V", false, game)
                  wait(.1)
                  game:service('VirtualInputManager'):SendKeyEvent(false, "V", false, game)
              end
          end
      end)
  end
end)

-- Create the toggles for each skill
local ToggleZ = Tabs.Setting:AddToggle("ToggleZ", {
    Title = "Auto Skill Z",
    Description = "",
    Default = false
})

local ToggleX = Tabs.Setting:AddToggle("ToggleX", {
    Title = "Auto Skill X",
    Description = "",
    Default = false
})

local ToggleC = Tabs.Setting:AddToggle("ToggleC", {
    Title = "Auto Skill C",
    Description = "",
    Default = false
})

local ToggleV = Tabs.Setting:AddToggle("ToggleV", {
    Title = "Auto Skill V",
    Description = "",
    Default = false
})

-- Set up the toggle behavior
ToggleZ:OnChanged(function(value)
    Skillz = value
end)

ToggleX:OnChanged(function(value)
    Skillx = value
end)

ToggleC:OnChanged(function(value)
    Skillc = value
end)

ToggleV:OnChanged(function(value)
    Skillv = value
end)

ToggleBring:OnChanged(function(Value)
    Settings.BringMobs = Value -- Activate or deactivate the Bring Mobs functionality based on toggle
end)
-- List of weapon types based on your settings
local WeaponList = {"Fight", "Power", "Weapon"}

-- Dropdown for selecting a weapon to equip
local WeaponDropdown = Tabs.Main:AddDropdown("WeaponDropdown", {
    Title = "Select Weapon",
    Values = WeaponList,
    Default = "Fight" -- Default weapon type to equip
})

WeaponDropdown:OnChanged(function(Value)
    -- Update the Settings.ToolFarm with the selected weapon type
    Settings.ToolFarm = Value
    
    -- Function to equip the selected weapon type
    local function EquipSelectedWeapon()
        local Backpack, Char = Player:FindFirstChild("Backpack"), Player.Character
        if IsAlive(Char) and Backpack then
            for _,v in ipairs(Backpack:GetChildren()) do
                if v:IsA("Tool") and v.ToolTip:find(Settings.ToolFarm) then
                    Char.Humanoid:EquipTool(v)
                end
            end
        end
    end
    
    -- Call the function to equip the selected weapon type
    EquipSelectedWeapon()
end)

-- Add this part to define the dropdown
local EnemyDropdown = Tabs.Misc:AddDropdown("EnemyDropdown", {
    Title = "Select Enemy",
    Values = Loaded.EnemeiesList,
    Default = Loaded.EnemeiesList[1],
    Multi = false,
    Callback = function(selectedEnemy)
        _env.SelecetedEnemie = selectedEnemy
    end
})

-- Ensure _env.SelecetedEnemie is defined somewhere in your script
_env.SelecetedEnemie = Loaded.EnemeiesList[1] -- or set a default value

local ToggleFSFarm = Tabs.Misc:AddToggle("ToggleFSFarm", {
    Title = "Farm mob select",
    Description = "Farm selected enemy",
    Default = false,
    Callback = function(value)
        _env["FS Enemie"] = value
    end
})

local ToggleAutoGiantPumpkin = Tabs.Boss:AddToggle("ToggleAutoGiantPumpkin", {
    Title = "Auto Giant Pumpkin",
    Description = "Automatically farm Giant Pumpkin",
    Default = false,
    Callback = function(value)
        _env["Giant Pumpkin"] = value
    end
})

local ToggleAutoEvilNoob = Tabs.Boss:AddToggle("ToggleAutoEvilNoob", {
    Title = "Auto Evil Noob",
    Description = "Automatically farm Evil Noob",
    Default = false,
    Callback = function(value)
        _env["Evil Noob"] = value
    end
})

local ToggleAutoLordSus = Tabs.Boss:AddToggle("ToggleAutoLordSus", {
    Title = "Auto Lord Sus",
    Description = "Automatically farm Lord Sus",
    Default = false,
    Callback = function(value)
        _env["Lord Sus"] = value
    end
})

local ToggleBuyPower = Tabs.Setting:AddToggle("ToggleBuyPower", {
  Title = "Auto Buy Power",
  Description = "Automatically Buy power",
  Default = false,
  Callback = function(value)
      _env["Buy Power"] = value
      if value then
          -- Start auto-buying when the toggle is enabled
          spawn(function()
              while _env["Buy Power"] do
                  wait(1) -- Adjust the interval if necessary
                  pcall(function()
                      OtherEvent.MainEvents.Modules:FireServer("Random_Power", {
                          Type = "Decuple",
                          NPCName = "Floppa Gacha",
                          GachaType = "Money"
                      })
                  end)
              end
          end)
      end
  end
})

local ToggleAutoStorePower = Tabs.Setting:AddToggle("ToggleAutoStorePower", {
  Title = "Auto Store Power",
  Description = "Automatically store power",
  Default = false,
  Callback = function(value)
      _env.AutoStorePowers = value
      while _env.AutoStorePowers do
          task.wait() -- Using task.wait() is better than _wait() for performance
          for _,v in ipairs(Player.Backpack:GetChildren()) do
              if v:IsA("Tool") and v.ToolTip == "Power" and v:GetAttribute("Using") == nil then
                  v.Parent = Player.Character
                  OtherEvent.MainEvents.Modules:FireServer("Eatable_Power", { Action = "Store", Tool = v })
              end
          end
      end
  end
})

local ToggleAutoFarmNearest = Tabs.Misc:AddToggle("ToggleAutoFarmNearest", {
    Title = "Auto Farm Nearest",
    Description = "Automatically farm nearest enemy",
    Default = false,
    Callback = function(value)
        _env["Nearest Farm"] = value
    end
})


-- Toggle for taking quests
local ToggleTakeQuest = Tabs.Setting:AddToggle("ToggleTakeQuest", {
    Title = "Take Quest",
    Description = "Enable or disable automatic quest-taking",
    Default = true -- Default to enabled
})

ToggleTakeQuest:OnChanged(function(Value)
    _env["FS Take Quest"] = Value -- Store the toggle state
end)

-- Function to handle farming the selected enemy, including quest logic
local function FarmSelectedEnemy()
    -- Check if the enemy is valid and get the corresponding quest
    local Quest = Loaded.EnemiesQuests[_env.SelecetedEnemie]
    
    if Quest then
        -- If a quest is required and the toggle is enabled, take the quest
        if not VerifyQuest(Quest) and _env["FS Take Quest"] then
            ClearQuests(Quest)  -- Clear any other quests
            TakeQuest(Quest)    -- Take the quest for the selected enemy
        end
        
        -- Now, farm the selected enemy
        if KillMonster(_env.SelecetedEnemie) then
            -- Enemy was killed successfully
        else
            -- Move to the enemy's position if not already there
            GoTo(EnemyLocation[_env.SelecetedEnemie].CFrame)
        end
    else
        -- Handle case where the enemy or quest is not found

    end
end

-- Updated EnemyDropdown with the quest logic toggle
EnemyDropdown:OnChanged(function(Value)
    _env.SelecetedEnemie = Value
    FarmSelectedEnemy()
end)

-- Add toggle for Auto Spam Skills

local ToggleAutoMemeBeast = Tabs.Boss:AddToggle("ToggleAutoMemeBeast", {
    Title = "Auto Meme Beast",
    Description = "Automatically farm Meme Beast",
    Default = false,
    Callback = function(value)
        _env["AutoMemeBeast"] = value
    end
})

-- Add toggle for Auto Farm Raid
-- Thêm toggle cho tính năng auto farm raid
local ToggleAutoFarmRaid = Tabs.Boss:AddToggle("ToggleAutoFarmRaid", {
    Title = "Auto Farm Raid",
    Description = "Automatically farms raid",
    Default = false,
    Callback = function(value)
        _env["Raid Farm"] = value
    end
})

-- Vòng lặp kiểm tra trạng thái toggle và thực hiện các hành động tương ứng
task.spawn(function()
    while true do
        _wait(1) -- Đợi 1 giây trước khi kiểm tra lại
        
        AutoFarmMrBeast()
        AutoFarmRaid()
        AutoAwakeningOrb()
        AutoBuyPower()
    end
end)




-- Add toggle for Auto Awakening Orb
local ToggleAutoAwakeningOrb = Tabs.Misc:AddToggle("ToggleAutoAwakeningOrb", {
    Title = "Auto Awakening Orb",
    Description = "Automatically farms Awakening Orb",
    Default = false,
    Callback = function(value)
        _env["Race V2 Orb"] = value
    end
})

-- Add toggle for Auto Floppa
local ToggleAutoFloppa = Tabs.Misc:AddToggle("ToggleAutoFloppa", {
    Title = "Auto Floppa",
    Description = "Automatically farm Floppa",
    Default = false,
    Callback = function(value)
        _env["AutoFloppa"] = value
        print("Auto Floppa: " .. tostring(value))
    end
})

task.spawn(function()
    while _wait() do
        if _env["AutoMemeBeast"] then
            AutoMemeBeast()
        end
        if _env["AutoFarmRaid"] then
            AutoFarmRaid()
        end
        if _env["AutoAwakeningOrb"] then
            AutoAwakeningOrb()
        end
        if _env["AutoFloppa"] then
            AutoFloppa()
        end
    end
end)

-- Lấy tất cả các hòn đảo từ workspace
-- Function to get the list of island names
local function GetIslandList()
  local islandList = {}
  local islandFolder = workspace:FindFirstChild("Island")
  
  if islandFolder then
      for _, island in ipairs(islandFolder:GetChildren()) do
          if island:IsA("Model") and island.Name then
              table.insert(islandList, island.Name)
          end
      end
  end
  
  return islandList
end

-- Retrieve the list of islands
local IslandList = GetIslandList()

-- Create dropdown with the island list
local IslandDropdown = Tabs.Tele:AddDropdown("IslandDropdown", {
  Title = "Select Island",
  Values = IslandList, -- Use the retrieved list of islands
  Default = IslandList[1] or "DefaultIsland", -- Set a default island (use the first one if available)
  Callback = function(selectedIsland)
      -- Function to teleport to the selected island
      local selectedIslandCFrame = Loaded.Islands[selectedIsland] and Loaded.Islands[selectedIsland].CFrame
      if selectedIslandCFrame then
          GoTo(selectedIslandCFrame) -- Perform the teleport
      end
  end
})


local ScreenGui = Instance.new("ScreenGui")
local ImageButton = Instance.new("ImageButton")
local UICorner = Instance.new("UICorner")

ScreenGui.Parent = game.CoreGui
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

ImageButton.Parent = ScreenGui
ImageButton.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
ImageButton.BorderSizePixel = 0
ImageButton.Position = UDim2.new(0.120833337, 0, 0.0952890813, 0)
ImageButton.Size = UDim2.new(0, 50, 0, 50)
ImageButton.Draggable = true
ImageButton.Image = "http://www.roblox.com/asset/?id=18622932541"
ImageButton.MouseButton1Down:connect(function()
    game:GetService("VirtualInputManager"):SendKeyEvent(true,Enum.KeyCode.End,false,game)
end)
end
