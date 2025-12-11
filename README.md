local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
   Name = "Aura Hub - Mini City",
   Icon = nil,
   LoadingTitle = "Aura Hub",
   LoadingSubtitle = "by Aura - Mini City",
   Theme = "Default",
   ToggleUIKeybind = "K",

   DisableRayfieldPrompts = false,
   DisableBuildWarnings = false,

   ConfigurationSaving = {
      Enabled = true,
      FolderName = nil,
      FileName = "Aura Hub Mini City"
   },

   Discord = {
      Enabled = true,
      Invite = "JF2F2RANud",
      RememberJoins = true
   },

   KeySystem = true,
   KeySettings = {
      Title = "Aura Keys",
      Subtitle = "Key System",
      Note = "Para conseguir a key, entre no discord da Aura",
      FileName = "Key",
      SaveKey = true,
      GrabKeyFromSite = false,
      Key = {"Aura", "Ore", "AuraHub", "MiniCity"}
   }
})

-- Services
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")

-- Variáveis globais
local speedEnabled = false
local noclipEnabled = false
local flyEnabled = false
local aimbotEnabled = false
local aimbotTeamCheck = true
local aimbotKillCheck = true
local noClipConnection

-- Configurações
local speedValue = 50
local flySpeed = 50

-- ============================================
-- FUNÇÕES BÁSICAS
-- ============================================

local function SafeExecute(func)
    pcall(func)
end

local function GetCharacter()
    return LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
end

local function GetHumanoid()
    local char = GetCharacter()
    return char:WaitForChild("Humanoid")
end

local function GetRoot()
    local char = GetCharacter()
    return char:WaitForChild("HumanoidRootPart")
end

-- ============================================
-- FUNÇÕES DE COMBATE
-- ============================================

-- Hitbox Aumentada
local hitboxConnections = {}
local function ModifyHitbox(size)
    SafeExecute(function()
        -- Remove conexões antigas
        for _, conn in ipairs(hitboxConnections) do
            conn:Disconnect()
        end
        hitboxConnections = {}
        
        if size > 0 then
            local function updateHitbox(player)
                if player.Character then
                    local humanoid = player.Character:FindFirstChild("Humanoid")
                    if humanoid then
                        local connection = humanoid:GetPropertyChangedSignal("Health"):Connect(function()
                            if humanoid.Health < humanoid.MaxHealth then
                                humanoid.Health = humanoid.MaxHealth
                            end
                        end)
                        table.insert(hitboxConnections, connection)
                    end
                end
            end
            
            for _, player in ipairs(Players:GetPlayers()) do
                if player.Character then
                    updateHitbox(player)
                end
                player.CharacterAdded:Connect(function()
                    updateHitbox(player)
                end)
            end
            
            local conn = Players.PlayerAdded:Connect(function(player)
                player.CharacterAdded:Connect(function()
                    updateHitbox(player)
                end)
            end)
            table.insert(hitboxConnections, conn)
        end
    end)
end

-- Aimbot
local function Aimbot()
    spawn(function()
        while aimbotEnabled and task.wait(0.1) do
            SafeExecute(function()
                local closestPlayer = nil
                local closestDistance = math.huge
                local myRoot = GetRoot()
                
                if not myRoot then return end
                
                for _, player in ipairs(Players:GetPlayers()) do
                    if player ~= LocalPlayer and player.Character then
                        local targetRoot = player.Character:FindFirstChild("HumanoidRootPart")
                        local humanoid = player.Character:FindFirstChild("Humanoid")
                        
                        if targetRoot and humanoid and humanoid.Health > 0 then
                            if aimbotTeamCheck then
                                -- Adicione lógica de verificação de time aqui
                            end
                            
                            if aimbotKillCheck and humanoid.Health <= 0 then
                                continue
                            end
                            
                            local distance = (targetRoot.Position - myRoot.Position).Magnitude
                            if distance < closestDistance then
                                closestDistance = distance
                                closestPlayer = player
                            end
                        end
                    end
                end
                
                if closestPlayer then
                    local targetRoot = closestPlayer.Character.HumanoidRootPart
                    myRoot.CFrame = CFrame.new(myRoot.Position, Vector3.new(targetRoot.Position.X, myRoot.Position.Y, targetRoot.Position.Z))
                end
            end)
        end
    end)
end

-- Auto Revistar
local autoRevistar = false
local function AutoSearch()
    spawn(function()
        while autoRevistar and task.wait(1) do
            SafeExecute(function()
                for _, player in ipairs(Players:GetPlayers()) do
                    if player ~= LocalPlayer and player.Character then
                        local targetRoot = player.Character:FindFirstChild("HumanoidRootPart")
                        local myRoot = GetRoot()
                        
                        if targetRoot and myRoot and (targetRoot.Position - myRoot.Position).Magnitude < 10 then
                            local args = {
                                [1] = "searchPlayer",
                                [2] = player
                            }
                            game:GetService("ReplicatedStorage"):FindFirstChild("RemoteEvent", true):FireServer(unpack(args))
                        end
                    end
                end
            end)
        end
    end)
end

-- ============================================
-- FUNÇÕES DE MOVIMENTO
-- ============================================

-- Speed Hack
local speedConnection
local function ApplySpeed()
    SafeExecute(function()
        if speedConnection then
            speedConnection:Disconnect()
        end
        
        speedConnection = RunService.RenderStepped:Connect(function()
            local humanoid = GetHumanoid()
            if humanoid then
                if speedEnabled then
                    humanoid.WalkSpeed = speedValue
                else
                    humanoid.WalkSpeed = 16
                end
            end
        end)
    end)
end

-- Noclip
local function EnableNoclip()
    SafeExecute(function()
        if noClipConnection then
            noClipConnection:Disconnect()
        end
        
        noClipConnection = RunService.Stepped:Connect(function()
            if noclipEnabled and GetCharacter() then
                for _, part in ipairs(GetCharacter():GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.CanCollide = false
                    end
                end
            end
        end)
    end)
end

-- Fly System
local flyVelocity
local function EnableFly()
    SafeExecute(function()
        local root = GetRoot()
        if not root then return end
        
        flyVelocity = Instance.new("BodyVelocity")
        flyVelocity.MaxForce = Vector3.new(10000, 10000, 10000)
        flyVelocity.Velocity = Vector3.new(0, 0, 0)
        flyVelocity.Parent = root
        
        UserInputService.InputBegan:Connect(function(input)
            if not flyEnabled then return end
            
            if input.KeyCode == Enum.KeyCode.Space then
                flyVelocity.Velocity = Vector3.new(0, flySpeed, 0)
            elseif input.KeyCode == Enum.KeyCode.LeftShift then
                flyVelocity.Velocity = Vector3.new(0, -flySpeed, 0)
            end
        end)
        
        UserInputService.InputEnded:Connect(function(input)
            if input.KeyCode == Enum.KeyCode.Space or input.KeyCode == Enum.KeyCode.LeftShift then
                flyVelocity.Velocity = Vector3.new(0, 0, 0)
            end
        end)
        
        while flyEnabled and task.wait() do
            if not root then break end
            
            local camera = Workspace.CurrentCamera
            local forward = camera.CFrame.LookVector
            local right = camera.CFrame.RightVector
            
            local moveDirection = Vector3.new(0, 0, 0)
            
            if UserInputService:IsKeyDown(Enum.KeyCode.W) then
                moveDirection = moveDirection + forward
            end
            if UserInputService:IsKeyDown(Enum.KeyCode.S) then
                moveDirection = moveDirection - forward
            end
            if UserInputService:IsKeyDown(Enum.KeyCode.D) then
                moveDirection = moveDirection + right
            end
            if UserInputService:IsKeyDown(Enum.KeyCode.A) then
                moveDirection = moveDirection - right
            end
            
            if moveDirection.Magnitude > 0 then
                moveDirection = moveDirection.Unit * flySpeed
                flyVelocity.Velocity = Vector3.new(moveDirection.X, flyVelocity.Velocity.Y, moveDirection.Z)
            else
                flyVelocity.Velocity = Vector3.new(0, flyVelocity.Velocity.Y, 0)
            end
        end
        
        if flyVelocity then
            flyVelocity:Destroy()
            flyVelocity = nil
        end
    end)
end

-- ============================================
-- FUNÇÕES DE EXPLOIT
-- ============================================

-- Sistema de Explodir
local function ExplodeAll()
    SafeExecute(function()
        -- Método 1: RemoteEvents
        for i = 1, 5 do
            local args = {
                [1] = "explode",
                [2] = "all"
            }
            game:GetService("ReplicatedStorage"):FindFirstChild("RemoteEvent", true):FireServer(unpack(args))
        end
        
        -- Método 2: BreakJoints
        for _, player in ipairs(Players:GetPlayers()) do
            if player.Character then
                for _, part in ipairs(player.Character:GetChildren()) do
                    if part:IsA("BasePart") then
                        part:BreakJoints()
                        task.wait(0.01)
                    end
                end
            end
        end
        
        -- Método 3: Alterar física
        Workspace.Gravity = 196.2
        task.wait(0.3)
        Workspace.Gravity = 0
        task.wait(0.3)
        Workspace.Gravity = 196.2
    end)
end

-- Bypass Anti-Cheat
local function EnableAntiCheatBypass()
    SafeExecute(function()
        local mt = getrawmetatable(game)
        local oldNamecall = mt.__namecall
        setreadonly(mt, false)
        
        mt.__namecall = newcclosure(function(...)
            local method = getnamecallmethod()
            local args = {...}
            
            -- Bypass para anti-cheat
            if method == "FireServer" then
                local remote = args[1]
                if tostring(remote):find("AntiCheat") or tostring(remote):find("AC") then
                    return nil
                end
            end
            
            -- Bypass para kick/ban
            if method == "Kick" or method == "Destroy" then
                return nil
            end
            
            return oldNamecall(...)
        end)
        
        setreadonly(mt, true)
        
        -- Hook em outras funções de detecção
        local oldIndex = mt.__index
        mt.__index = newcclosure(function(self, key)
            if key == "CFrame" or key == "Position" then
                return oldIndex(self, key)
            end
            return oldIndex(self, key)
        end)
    end)
end

-- Função de Teleport (essencial)
local function TeleportTo(position)
    SafeExecute(function()
        local root = GetRoot()
        if root then
            root.CFrame = CFrame.new(position)
        end
    end)
end

-- Função de Cura (essencial)
local function HealHealth()
    SafeExecute(function()
        local humanoid = GetHumanoid()
        if humanoid then
            humanoid.Health = humanoid.MaxHealth
        end
    end)
end

-- ============================================
-- GUI INTERFACE SIMPLIFICADA
-- ============================================

local MainTab = Window:CreateTab("Menu Principal", 4483362458)
local CombatTab = Window:CreateTab("Combate", 4483362458)
local ExploitTab = Window:CreateTab("Exploit", 4483362458)

-- Seção Principal
MainTab:CreateSection("Movimento")

local SpeedToggle = MainTab:CreateToggle({
    Name = "Speed Hack",
    CurrentValue = false,
    Callback = function(Value)
        speedEnabled = Value
        ApplySpeed()
    end
})

local NoclipToggle = MainTab:CreateToggle({
    Name = "Noclip",
    CurrentValue = false,
    Callback = function(Value)
        noclipEnabled = Value
        EnableNoclip()
    end
})

local FlyToggle = MainTab:CreateToggle({
    Name = "Fly",
    CurrentValue = false,
    Callback = function(Value)
        flyEnabled = Value
        if Value then
            EnableFly()
        end
    end
})

-- Sliders de configuração
MainTab:CreateSection("Configurações")

local SpeedSlider = MainTab:CreateSlider({
    Name = "Velocidade",
    Range = {16, 200},
    Increment = 5,
    Suffix = "",
    CurrentValue = 50,
    Callback = function(Value)
        speedValue = Value
        if speedEnabled then
            ApplySpeed()
        end
    end
})

local FlySpeedSlider = MainTab:CreateSlider({
    Name = "Velocidade do Fly",
    Range = {10, 200},
    Increment = 5,
    Suffix = "",
    CurrentValue = 50,
    Callback = function(Value)
        flySpeed = Value
    end
})

-- Botão de cura
local HealButton = MainTab:CreateButton({
    Name = "Recuperar Vida",
    Callback = HealHealth
})

-- Seção Combate
CombatTab:CreateSection("Aimbot")

local AimbotToggle = CombatTab:CreateToggle({
    Name = "Aimbot",
    CurrentValue = false,
    Callback = function(Value)
        aimbotEnabled = Value
        if Value then
            Aimbot()
        end
    end
})

local TeamCheckToggle = CombatTab:CreateToggle({
    Name = "Team Check",
    CurrentValue = true,
    Callback = function(Value)
        aimbotTeamCheck = Value
    end
})

local KillCheckToggle = CombatTab:CreateToggle({
    Name = "Kill Check",
    CurrentValue = true,
    Callback = function(Value)
        aimbotKillCheck = Value
    end
})

CombatTab:CreateSection("Outros")

local HitboxSlider = CombatTab:CreateSlider({
    Name = "Tamanho da Hitbox",
    Range = {1, 10},
    Increment = 1,
    Suffix = "x",
    CurrentValue = 5,
    Callback = function(Value)
        ModifyHitbox(Value)
    end
})

local AutoSearchToggle = CombatTab:CreateToggle({
    Name = "Auto Revistar",
    CurrentValue = false,
    Callback = function(Value)
        autoRevistar = Value
        if Value then
            AutoSearch()
        end
    end
})

-- Teleport rápido para locais úteis
CombatTab:CreateSection("Teleport Rápido")

local TeleportButtons = {
    {"Safe Zone", Vector3.new(100, 100, 100)},
    {"Prefeitura", Vector3.new(100, 50, 100)},
    {"Hospital", Vector3.new(200, 50, 200)},
    {"Delegacia", Vector3.new(300, 50, 300)}
}

for _, buttonData in ipairs(TeleportButtons) do
    local buttonName = buttonData[1]
    local position = buttonData[2]
    
    CombatTab:CreateButton({
        Name = "TP: " .. buttonName,
        Callback = function()
            TeleportTo(position)
        end
    })
end

-- Seção Exploit
ExploitTab:CreateSection("Exploits Principais")

local ExplodeButton = ExploitTab:CreateButton({
    Name = "EXPLODIR TUDO",
    Callback = ExplodeAll
})

local AntiCheatToggle = ExploitTab:CreateToggle({
    Name = "Ativar Bypass Anti-Cheat",
    CurrentValue = false,
    Callback = function(Value)
        if Value then
            EnableAntiCheatBypass()
        end
    end
})

-- Exploits adicionais
ExploitTab:CreateSection("Outros Exploits")

-- Kill All Players
ExploitTab:CreateButton({
    Name = "Matar Todos Jogadores",
    Callback = function()
        SafeExecute(function()
            for _, player in ipairs(Players:GetPlayers()) do
                if player.Character then
                    local humanoid = player.Character:FindFirstChild("Humanoid")
                    if humanoid then
                        humanoid.Health = 0
                    end
                end
            end
        end)
    end
})

-- Freeze All Players
local freezeEnabled = false
ExploitTab:CreateToggle({
    Name = "Congelar Todos Jogadores",
    CurrentValue = false,
    Callback = function(Value)
        freezeEnabled = Value
        SafeExecute(function()
            for _, player in ipairs(Players:GetPlayers()) do
                if player ~= LocalPlayer and player.Character then
                    local root = player.Character:FindFirstChild("HumanoidRootPart")
                    if root then
                        if Value then
                            root.Anchored = true
                        else
                            root.Anchored = false
                        end
                    end
                end
            end
        end)
    end
})

-- Crash Server (uso extremo)
ExploitTab:CreateButton({
    Name = "CRASHAR SERVER (EXTREMO)",
    Callback = function()
        SafeExecute(function()
            -- Envia múltiplas requisições para crashar
            for i = 1, 1000 do
                local args = {
                    [1] = "crash",
                    [2] = "server"
                }
                game:GetService("ReplicatedStorage"):FindFirstChild("RemoteEvent", true):FireServer(unpack(args))
                task.wait(0.001)
            end
        end)
    end
})

-- Keybinds para funções rápidas
ExploitTab:CreateSection("Keybinds Rápidos")

local KeybindHeal = ExploitTab:CreateKeybind({
    Name = "Tecla para Curar",
    CurrentKeybind = "H",
    HoldToInteract = false,
    Callback = function()
        HealHealth()
    end
})

local KeybindExplode = ExploitTab:CreateKeybind({
    Name = "Tecla para Explodir",
    CurrentKeybind = "X",
    HoldToInteract = false,
    Callback = function()
        ExplodeAll()
    end
})

-- Informações do script
ExploitTab:CreateParagraph({
    Title = "Informações",
    Content = "Aura Hub - Versão Essencial\nUse com responsabilidade\nDiscord: discord.gg/JF2F2RANud"
})

-- Inicialização
spawn(function()
    -- Prepara sistema de proteção básico
    SafeExecute(function()
        -- Remove logs de detecção
        local oldWarn = warn
        warn = function(...)
            -- Silencia warns suspeitos
            local args = {...}
            if not tostring(args[1]):find("AntiCheat") then
                oldWarn(...)
            end
        end
    end)
end)

-- Carrega configurações
Rayfield:LoadConfiguration()

print("╔══════════════════════════════════╗")
print("║     Aura Hub - Mini City         ║")
print("║      Versão Essencial            ║")
print("║                                  ║")
print("║  Funções Ativas:                 ║")
print("║  • Aimbot                        ║")
print("║  • Fly                           ║")
print("║  • Speed/Noclip                  ║")
print("║  • Hitbox                        ║")
print("║  • Exploits                      ║")
print("╚══════════════════════════════════╝")
