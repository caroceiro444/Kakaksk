

local Player = game:GetService("Players").LocalPlayer
local Character = Player.Character or Player.CharacterAdded:Wait()
local Humanoid = Character:WaitForChild("Humanoid")
local RootPart = Character:WaitForChild("HumanoidRootPart")

-- Interface
local Library = loadstring(game:HttpGet("https://raw.githubusercontent.com/xHeptc/Kavo-UI-Library/main/source.lua"))()
local Window = Library.CreateLib("Blox Fruits Premium", "Serpent")

-- Tabs
local MainTab = Window:NewTab("Menu Principal")
local CombatTab = Window:NewTab("Combate")
local FarmTab = Window:NewTab("Farm Automático")
local TeleportTab = Window:NewTab("Teleportes")
local PlayerTab = Window:NewTab("Player")
local MiscTab = Window:NewTab("Miscelânea")

-- Sections
local MainSection = MainTab:NewSection("Configurações Principais")
local CombatSection = CombatTab:NewSection("Habilidades de Combate")
local AutoFarmSection = FarmTab:NewSection("Farm Automático")
local BossFarmSection = FarmTab:NewSection("Farm de Chefes")
local IslandSection = TeleportTab:NewSection("Ilhas")
local BossSection = TeleportTab:NewSection("Chefes")
local PlayerSection = PlayerTab:NewSection("Melhorias")
local MiscSection = MiscTab:NewSection("Extras")

-- Funções
function GetNearestPlayer(maxDist)
    local closest, dist = nil, maxDist or math.huge
    for _,v in pairs(game.Players:GetPlayers()) do
        if v ~= Player and v.Character and v.Character:FindFirstChild("HumanoidRootPart") then
            local d = (RootPart.Position - v.Character.HumanoidRootPart.Position).Magnitude
            if d < dist then
                closest = v
                dist = d
            end
        end
    end
    return closest, dist
end

function TweenTo(pos)
    local tweenInfo = TweenInfo.new((RootPart.Position - pos).Magnitude/300, Enum.EasingStyle.Linear)
    game:GetService("TweenService"):Create(RootPart, tweenInfo, {CFrame = CFrame.new(pos)}):Play()
end

-- Auto Farm
AutoFarmSection:NewToggle("Farm Automático", "Farm NPCs automaticamente", function(state)
    getgenv().AutoFarm = state
    
    spawn(function()
        while getgenv().AutoFarm do
            local target = nil
            local minDist = math.huge
            
            for _,v in pairs(workspace.Enemies:GetChildren()) do
                if v:FindFirstChild("Humanoid") and v.Humanoid.Health > 0 then
                    local dist = (RootPart.Position - v.HumanoidRootPart.Position).Magnitude
                    if dist < minDist then
                        minDist = dist
                        target = v
                    end
                end
            end
            
            if target then
                repeat
                    RootPart.CFrame = target.HumanoidRootPart.CFrame * CFrame.new(0, 0, 5)
                    game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("Attack", target.HumanoidRootPart.Position)
                    task.wait()
                until not target or not target:FindFirstChild("Humanoid") or target.Humanoid.Health <= 0 or not getgenv().AutoFarm
            end
            task.wait()
        end
    end)
end)

AutoFarmSection:NewSlider("Distância do Farm", "Distância para ficar do NPC", 30, 5, function(val)
    getgenv().FarmDistance = val
end)

-- Auto Boss
BossFarmSection:NewToggle("Farm de Chefes", "Farm chefes automaticamente", function(state)
    getgenv().BossFarm = state
    
    spawn(function()
        while getgenv().BossFarm do
            for _,bossName in pairs({"Boss1", "Boss2", "Boss3"}) do -- Substitua pelos nomes reais
                local boss = workspace.Enemies:FindFirstChild(bossName)
                if boss and boss:FindFirstChild("Humanoid") and boss.Humanoid.Health > 0 then
                    repeat
                        RootPart.CFrame = boss.HumanoidRootPart.CFrame * CFrame.new(0, 0, getgenv().FarmDistance or 10)
                        game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("Attack", boss.HumanoidRootPart.Position)
                        task.wait()
                    until not boss or not boss:FindFirstChild("Humanoid") or boss.Humanoid.Health <= 0 or not getgenv().BossFarm
                end
            end
            task.wait(1)
        end
    end)
end)

-- Combat
CombatSection:NewToggle("Auto Haki", "Ativa Haki automaticamente", function(state)
    getgenv().AutoHaki = state
    
    spawn(function()
        while getgenv().AutoHaki do
            game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("ActivateAbility")
            task.wait(1)
        end
    end)
end)

CombatSection:NewToggle("Auto Ken", "Bloqueia automaticamente", function(state)
    getgenv().AutoKen = state
    
    spawn(function()
        while getgenv().AutoKen do
            game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("Ken")
            task.wait(0.1)
        end
    end)
end)

-- Teleports
IslandSection:NewDropdown("Teleportar para Ilha", "Selecione a ilha", {"Starting Island", "Jungle", "Desert", "Snow Village", "Sky Island"}, function(option)
    local locations = {
        ["Starting Island"] = Vector3.new(100, 50, 100), -- Coordenadas exemplo
        ["Jungle"] = Vector3.new(200, 50, -500),
        -- Adicione outras localizações
    }
    
    if locations[option] then
        TweenTo(locations[option])
    end
end)

BossSection:NewDropdown("Teleportar para Chefe", "Selecione o chefe", {"Saber Expert", "Darkbeard", "D# Kakaksk
