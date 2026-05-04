-- LOAD UI
local repo = "https://raw.githubusercontent.com/deividcomsono/Obsidian/main/"
local Library = loadstring(game:HttpGet(repo .. "Library.lua"))()
local ThemeManager = loadstring(game:HttpGet(repo .. "addons/ThemeManager.lua"))()
local SaveManager = loadstring(game:HttpGet(repo .. "addons/SaveManager.lua"))()

local Options = Library.Options
local Toggles = Library.Toggles

-- WINDOW
local Window = Library:CreateWindow({
    Title = "QTE Hub",
    Footer = "Full Version",
})

local Tabs = {
    Main = Window:AddTab("Main", "user"),
    Teleport = Window:AddTab("Teleport", "map"),
    ["UI Settings"] = Window:AddTab("Settings", "settings"),
}

-- SERVICES
local RS = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local HttpService = game:GetService("HttpService")

local player = Players.LocalPlayer
local Remote = RS:WaitForChild("Remotes"):WaitForChild("Information"):WaitForChild("RemoteFunction")

-- =========================
-- SETTINGS
-- =========================
local Settings = {
    QTE_Delay = 1.3,
    Dodge_Delay = 0.05,
    TP_Speed = 3
}

-- =========================
-- MAIN
-- =========================
local MainBox = Tabs.Main:AddLeftGroupbox("Combat")

MainBox:AddToggle("AutoQTE",{Text="Auto QTE"})
MainBox:AddToggle("AutoDodge",{Text="Auto Dodge"})
MainBox:AddToggle("HideQTE",{Text="Hide QTE"})
MainBox:AddToggle("TPWalk",{Text="TP Walk"})
MainBox:AddToggle("InfJump",{Text="Infinite Jump"})
MainBox:AddToggle("AutoAttack",{Text="Auto Attack"})

MainBox:AddSlider("QTE_Delay",{Text="QTE Delay",Default=1.3,Min=0.1,Max=10,Rounding=2})
MainBox:AddSlider("Dodge_Delay",{Text="Dodge Delay",Default=0.05,Min=0.01,Max=1,Rounding=2})
MainBox:AddSlider("TP_Speed",{Text="TP Speed",Default=3,Min=1,Max=20,Rounding=0})

local RightBox = Tabs.Main:AddRightGroupbox("QTE Select")

RightBox:AddDropdown("QTE_Weapons",{
    Values={"FistQTE","SpearQTE","SwordQTE","DaggerQTE","MagicQTE"},
    Default={"DaggerQTE"},
    Multi=true,
    Text="Weapon QTE"
})

-- =========================
-- AUTO SKILL
-- =========================
local function getSkills()
    local skills={}
    local gui=player:FindFirstChild("PlayerGui")
    if not gui then return {"Strike"} end

    local ok,path=pcall(function()
        return gui.Combat.ActionBG.AttacksPage.Attack.ScrollingFrame
    end)

    if ok and path then
        for _,v in pairs(path:GetDescendants()) do
            if v:IsA("TextLabel") or v:IsA("TextButton") then
                local name=v.Text~="" and v.Text or v.Name
                if name~="" and not table.find(skills,name) then
                    table.insert(skills,name)
                end
            end
        end
    end

    if #skills==0 then table.insert(skills,"Strike") end
    return skills
end

local SkillBox = Tabs.Main:AddRightGroupbox("Auto Skill")

SkillBox:AddDropdown("SkillSelect",{
    Values=getSkills(),
    Default={"Strike"},
    Multi=true,
    Text="Skills"
})

SkillBox:AddButton("Refresh Skills",function()
    Options.SkillSelect:SetValues(getSkills())
end)

-- =========================
-- TELEPORT (SAVE FILE)
-- =========================
local TPBox = Tabs.Teleport:AddLeftGroupbox("Teleport System")
local savedPositions = {}
local fileName = "QTE_TP.json"

local function saveToFile()
    if writefile then
        writefile(fileName, HttpService:JSONEncode(savedPositions))
    end
end

local function loadFromFile()
    if isfile and isfile(fileName) then
        savedPositions = HttpService:JSONDecode(readfile(fileName))
    end
end

local function refreshTP()
    local list={}
    for n in pairs(savedPositions) do table.insert(list,n) end
    Options.TP_List:SetValues(list)
end

loadFromFile()

TPBox:AddInput("TP_Name",{Default="",Text="Point Name"})
TPBox:AddDropdown("TP_List",{Values={},Multi=false,Text="Saved Points"})

TPBox:AddButton("Save Position",function()
    local name=Options.TP_Name.Value
    if name=="" then return end

    local c=player.Character
    if c and c:FindFirstChild("HumanoidRootPart") then
        local p=c.HumanoidRootPart.Position
        savedPositions[name]={p.X,p.Y,p.Z}
        saveToFile()
        refreshTP()
    end
end)

TPBox:AddButton("Teleport",function()
    local n=Options.TP_List.Value
    if n and savedPositions[n] then
        local p=savedPositions[n]
        player.Character.HumanoidRootPart.CFrame=CFrame.new(p[1],p[2],p[3])
    end
end)

TPBox:AddButton("Delete Point",function()
    local n=Options.TP_List.Value
    if n then
        savedPositions[n]=nil
        saveToFile()
        refreshTP()
    end
end)

TPBox:AddButton("Refresh List",refreshTP)
refreshTP()

-- =========================
-- SYSTEMS
-- =========================
Options.QTE_Delay:OnChanged(function() Settings.QTE_Delay=Options.QTE_Delay.Value end)
Options.Dodge_Delay:OnChanged(function() Settings.Dodge_Delay=Options.Dodge_Delay.Value end)
Options.TP_Speed:OnChanged(function() Settings.TP_Speed=Options.TP_Speed.Value end)

task.spawn(function()
    while true do
        if Toggles.AutoQTE.Value then
            for w,en in pairs(Options.QTE_Weapons.Value) do
                if en then Remote:FireServer(true,w) end
            end
        end
        task.wait(Settings.QTE_Delay)
    end
end)

task.spawn(function()
    while true do
        if Toggles.AutoDodge.Value then
            Remote:FireServer({true,true},"DodgeMinigame")
        end
        task.wait(Settings.Dodge_Delay)
    end
end)

RunService.RenderStepped:Connect(function()
    if Toggles.TPWalk.Value then
        local c=player.Character
        if c then
            local h=c:FindFirstChild("Humanoid")
            local hrp=c:FindFirstChild("HumanoidRootPart")
            if h and hrp then
                local m=h.MoveDirection
                if m.Magnitude>0 then
                    hrp.CFrame+=m*Settings.TP_Speed
                end
            end
        end
    end
end)

UIS.JumpRequest:Connect(function()
    if Toggles.InfJump.Value then
        local h=player.Character and player.Character:FindFirstChild("Humanoid")
        if h then h:ChangeState(Enum.HumanoidStateType.Jumping) end
    end
end)

-- =========================
-- SETTINGS (เดิม 100%)
-- =========================
ThemeManager:SetLibrary(Library)
SaveManager:SetLibrary(Library)

SaveManager:IgnoreThemeSettings()
SaveManager:SetFolder("QTEHub")

SaveManager:BuildConfigSection(Tabs["UI Settings"])
ThemeManager:ApplyToTab(Tabs["UI Settings"])

SaveManager:LoadAutoloadConfig()
