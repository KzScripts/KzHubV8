--carregar ui
local redzlib = loadstring(game:HttpGet("https://raw.githubusercontent.com/tbao143/Library-ui/refs/heads/main/Redzhubui"))()

--window
local Window = redzlib:MakeWindow({
  Title = "Kz Hub : Universal",
  SubTitle = "by kzscripts",
  SaveFolder = "testando | redz lib v5.lua"
})

Window:AddMinimizeButton({
    Button = { Image = "rbxassetid://121935994258369", BackgroundTransparency = 1 },
    Corner = { CornerRadius = UDim.new(35, 1) },
})

local Tab1 = Window:MakeTab({"Discord", "info"})

Tab1:AddDiscordInvite({
    Name = "Kz Hub",
    Description = "Join server",
    Logo = "rbxassetid://121935994258369",
    Invite = "https://discord.gg/mv6uWsNqSY",
})

local TabMain = Window:MakeTab({"Main", "layers"})

-- Toggle ESP
local ESPEnabled = false
TabMain:AddToggle({
    Name = "Ativar ESP",
    Flag = "ESP",
    Callback = function(state)
        ESPEnabled = state
        if ESPEnabled then
            -- Exemplo básico de ESP com Highlight
            for _, player in ipairs(game.Players:GetPlayers()) do
                if player ~= game.Players.LocalPlayer then
                    local char = player.Character
                    if char and not char:FindFirstChild("KZ_ESP") then
                        local highlight = Instance.new("Highlight")
                        highlight.Name = "KZ_ESP"
                        highlight.FillColor = Color3.fromRGB(255, 0, 0) -- vermelho padrão
                        highlight.FillTransparency = 0.5
                        highlight.OutlineColor = Color3.new(0, 0, 0)
                        highlight.OutlineTransparency = 0
                        highlight.Adornee = char
                        highlight.Parent = char
                    end
                end
            end
        else
            for _, player in ipairs(game.Players:GetPlayers()) do
                local char = player.Character
                if char and char:FindFirstChild("KZ_ESP") then
                    char.KZ_ESP:Destroy()
                end
            end
        end
    end
})

-- Dropdown de Cor do ESP
TabMain:AddDropdown({
    Name = "Cor do ESP",
    Default = "Vermelho",
    Options = {"Vermelho", "Azul", "Amarelo", "Roxo"},
    Callback = function(color)
        local cores = {
            ["Vermelho"] = Color3.fromRGB(255, 0, 0),
            ["Azul"] = Color3.fromRGB(0, 0, 255),
            ["Amarelo"] = Color3.fromRGB(255, 255, 0),
            ["Roxo"] = Color3.fromRGB(128, 0, 128)
        }

        for _, player in ipairs(game.Players:GetPlayers()) do
            local char = player.Character
            if char and char:FindFirstChild("KZ_ESP") then
                char.KZ_ESP.FillColor = cores[color] or Color3.fromRGB(255, 0, 0)
            end
        end
    end
})


local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer
local FOV = 200
local aimbotEnabled = false
local aimbotConnection = nil

-- Função para encontrar inimigo mais próximo dentro da FOV
local function GetClosestEnemy()
    local closest = nil
    local shortestDist = FOV

    for _, player in pairs(Players:GetPlayers()) do  
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then  
            local head = player.Character.Head  
            local screenPos, onScreen = Camera:WorldToViewportPoint(head.Position)  

            if onScreen then  
                local mousePos = UserInputService:GetMouseLocation()  
                local dist = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude  

                if dist < shortestDist then  
                    shortestDist = dist  
                    closest = player  
                end  
            end  
        end  
    end  

    return closest
end

-- Função para mover o mouse até o alvo
local function AimMouseAtTarget(target)
    if not target or not target.Character or not target.Character:FindFirstChild("Head") then return end

    local head = target.Character.Head  
    local screenPos, onScreen = Camera:WorldToViewportPoint(head.Position)  

    if onScreen then  
        VirtualInputManager:SendMouseMoveEvent(screenPos.X, screenPos.Y, game, 0)  
    end
end

-- Toggle na Redz Library
TabMain:AddToggle({
    Name = "Aimbot",
    Flag = "AimbotMouse",
    Default = false,
    Callback = function(Value)
        aimbotEnabled = Value

        if aimbotConnection then
            aimbotConnection:Disconnect()
            aimbotConnection = nil
        end

        if Value then
            aimbotConnection = RunService.RenderStepped:Connect(function()
                local target = GetClosestEnemy()
                if target then
                    AimMouseAtTarget(target)
                end
            end)
        end
    end
})

local TabPlayer = Window:MakeTab({"Player", "user"})

-- Infinite Jump
TabPlayer:AddButton({
    Name = "Infinite Jump",
    Callback = function()
        game:GetService("UserInputService").JumpRequest:Connect(function()
            game.Players.LocalPlayer.Character:FindFirstChildOfClass("Humanoid"):ChangeState("Jumping")
        end)
    end
})

-- Fly (simples)
TabPlayer:AddButton({
    Name = "Fly",
    Callback = function()
        loadstring(game:HttpGet("https://rawscripts.net/raw/Universal-Script-Gui-Fly-v3-37111"))()
    end
})

TabPlayer:AddSlider({
    Name = "JumpPower",
    Min = 1,
    Max = 500,
    Increase = 1,
    Default = 50,
    Callback = function(Value)
        local player = game.Players.LocalPlayer
        local char = player.Character or player.CharacterAdded:Wait()
        local hum = char:FindFirstChild("Humanoid")
        if hum then
            hum.JumpPower = Value
        end

        -- Reaplica ao morrer
        game.Players.LocalPlayer.CharacterAdded:Connect(function(newChar)
            local newHum = newChar:WaitForChild("Humanoid")
            newHum.JumpPower = Value
        end)
    end
})

TabPlayer:AddSlider({
    Name = "Dash Length",
    Min = 1,
    Max = 100,
    Increase = 1,
    Default = 50,
    Callback = function(Value)
        getgenv().DashLength = Value
    end
})
TabPlayer:AddSlider({
    Name = "Speed",
    Min = 1,
    Max = 350,
    Increase = 1,
    Default = 16,
    Callback = function(Value)
        local player = game.Players.LocalPlayer
        local char = player.Character or player.CharacterAdded:Wait()
        local hum = char:FindFirstChild("Humanoid")
        if hum then
            hum.WalkSpeed = Value
        end

        -- Reaplica ao morrer
        game.Players.LocalPlayer.CharacterAdded:Connect(function(newChar)
            local newHum = newChar:WaitForChild("Humanoid")
            newHum.WalkSpeed = Value
        end)
    end
})



TabPlayer:AddToggle({
    Name = "Invisibilidade",
    Flag = "InvisToggle",
    Default = false,
    Callback = function(state)
        local player = game.Players.LocalPlayer
        local character = player.Character or player.CharacterAdded:Wait()

        if not character then return end

        for _, part in ipairs(character:GetDescendants()) do
            if part:IsA("BasePart") or part:IsA("Decal") then
                part.Transparency = state and 1 or 0
            elseif part:IsA("Accessory") then
                local handle = part:FindFirstChild("Handle")
                if handle then
                    handle.Transparency = state and 1 or 0
                end
            end
        end
    end
})

local TabSettings = Window:MakeTab({"Settings", "settings"})

local noclipActive = false
local noclipConnection

TabPlayer:AddToggle({
    Name = "No Clip",
    Flag = "NoClip",
    Default = false,
    Callback = function(state)
        noclipActive = state

        -- Encerra se desativar
        if not noclipActive and noclipConnection then
            noclipConnection:Disconnect()
            noclipConnection = nil
            return
        end

        -- Ativa o NoClip
        local player = game.Players.LocalPlayer
        local char = player.Character or player.CharacterAdded:Wait()

        noclipConnection = game:GetService("RunService").Stepped:Connect(function()
            if not noclipActive or not char then return end
            for _, part in pairs(char:GetDescendants()) do
                if part:IsA("BasePart") and part.CanCollide == true then
                    part.CanCollide = false
                end
            end
        end)
    end
})

-- Anti-AFK
local AntiAfkConnection
TabSettings:AddToggle({
    Name = "Anti-AFK",
    Flag = "AntiAFK",
    Default = false,
    Callback = function(enabled)
        if enabled then
            AntiAfkConnection = game:GetService("Players").LocalPlayer.Idled:Connect(function()
                game:GetService("VirtualUser"):Button2Down(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
                wait(1)
                game:GetService("VirtualUser"):Button2Up(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
            end)
        elseif AntiAfkConnection then
            AntiAfkConnection:Disconnect()
        end
    end
})

-- FPS Boost (desativa efeitos)
TabSettings:AddButton({
    Name = "FPS Boost",
    Callback = function()
        for _, v in ipairs(game:GetDescendants()) do
            if v:IsA("BasePart") then
                v.Material = Enum.Material.SmoothPlastic
                v.Reflectance = 0
            elseif v:IsA("Decal") or v:IsA("Texture") then
                v:Destroy()
            elseif v:IsA("ParticleEmitter") or v:IsA("Trail") then
                v.Enabled = false
            elseif v:IsA("Lighting") then
                v.GlobalShadows = false
                v.FogEnd = 1e10
                v.Brightness = 0
            end
        end
        setfpscap(60)
    end
})

--antban
local antibanEnabled = false

TabSettings:AddToggle({
    Name = "Anti-Ban",
    Flag = "AntiBan",
    Default = false,
    Callback = function(state)
        antibanEnabled = state

        if state then
            -- Desativa Kick e outros métodos comuns
            local mt = getrawmetatable(game)
            local oldNamecall = mt.__namecall
            setreadonly(mt, false)

            mt.__namecall = newcclosure(function(...)
                local args = {...}
                local method = getnamecallmethod()

                -- Intercepta tentativa de Kick
                if method == "Kick" and antibanEnabled then
                    warn("[KZ Hub] Tentativa de kick bloqueada.")
                    return nil
                end

                return oldNamecall(unpack(args))
            end)

            setreadonly(mt, true)

            -- Remove scripts suspeitos
            for _, v in pairs(game:GetDescendants()) do
                if v:IsA("LocalScript") and v.Name:lower():find("ban") then
                    v:Destroy()
                end
            end

            warn("[KZ Hub] Anti-Ban ativado.")
        else
            warn("[KZ Hub] Anti-Ban desativado. (requer reinício para resetar metatable)")
        end
    end
})


local TabScripts = Window:MakeTab({"Scripts", "shop"})

--BloxFruits
TabScripts:AddButton({
    Name = "BloxFruits",
    Callback = function()
        local Settings = {
    JoinTeam = "Pirates"; -- Pirates / Marines
    Translator = true;   -- true / false
}

loadstring(game:HttpGet("https://raw.githubusercontent.com/tlredz/Scripts/refs/heads/main/main.luau"))(Settings)
    end
})

--GrowGardem
TabScripts:AddButton({
    Name = "GrowGardem",
    Callback = function()
       loadstring(game:HttpGet('https://raw.githubusercontent.com/ThundarZ/Welcome/refs/heads/main/Main/GaG/Main.lua'))()
    end
})

--BladeBall
TabScripts:AddButton({
    Name = "BladeBall",
    Callback = function()
        loadstring(game:HttpGet("https://gist.githubusercontent.com/zyrn0x/62699883dd3ca39a2b11eaf260de0b01/raw/001c32b75d28ca592d06ccebb48b937458b1d34f/SillyHub",true))()
    end
})

--arise crossover
TabScripts:AddButton({
    Name = "AriseCrossover",
    Callback = function()
        loadstring(game:HttpGet("https://raw.githubusercontent.com/JustLevel/goombahub/main/AriseCrossover.lua"))()
    end
    
