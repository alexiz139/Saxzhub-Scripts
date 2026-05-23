--[[ 
    SAXZHUB X | ULTIMATE COMBAT SCRIPT
    Integración: Force Silent Aim + LEEHUB Auto Shoot + Duelos MvS
]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local LP = Players.LocalPlayer
local Cam = Workspace.CurrentCamera

---------------------------------------------------
-- 1. FUNCIONES DE LÓGICA (FORCE & LEEHUB)
---------------------------------------------------
local Combat = { SilentAim = false, AutoShoot = false, ShootDist = 300 }

local function SanitizeName(str) return tostring(str):gsub('%s+', '') end

-- Detección de enemigos mejorada para MvS
local function isEnemy(p)
    if p == LP then return false end
    -- Lógica MvS: Busca carpetas de equipos si existen
    local runningGames = workspace:FindFirstChild("RunningGames")
    if runningGames then
        for _, gameFolder in ipairs(runningGames:GetChildren()) do
            local alive = gameFolder:FindFirstChild("AlivePlayers")
            if alive then
                -- Lógica específica de equipos para duelos
                if alive:FindFirstChild("TeamBlue") and alive.TeamBlue:FindFirstChild(SanitizeName(LP.Name)) then
                    return alive:FindFirstChild("TeamRed") and alive.TeamRed:FindFirstChild(SanitizeName(p.Name))
                elseif alive:FindFirstChild("TeamRed") and alive.TeamRed:FindFirstChild(SanitizeName(LP.Name)) then
                    return alive:FindFirstChild("TeamBlue") and alive.TeamBlue:FindFirstChild(SanitizeName(p.Name))
                end
            end
        end
    end
    return true -- Por defecto si no hay sistema de equipos
end

local function fire()
    if not Combat.AutoShoot then return end
    local target, closest = nil, math.huge
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= LP and p.Character and isEnemy(p) then
            local head = p.Character:FindFirstChild("Head")
            if head then
                local dist = (Cam.CFrame.Position - head.Position).Magnitude
                if dist < Combat.ShootDist and dist < closest then
                    closest = dist; target = head
                end
            end
        end
    end
    
    if target then
        local weapon = LP.Character and LP.Character:FindFirstChildOfClass("Tool")
        if weapon and weapon:FindFirstChild("fire") then
            pcall(function() weapon.fire:FireServer() end)
        end
    end
end

---------------------------------------------------
-- 2. GUI (SAXZHUB X)
---------------------------------------------------
-- [Aquí inserta tu código de la interfaz SAXZHUB X desde la línea "local GUI = Instance.new..." hasta la "CombatPage"]

-- En la sección de la página Combat (CombatPage), añade los toggles:
local function CreateCombatToggle(Name, Callback)
    local Toggle = Instance.new("TextButton", CombatPage)
    Toggle.Size = UDim2.new(1, -30, 0, 45); Toggle.BackgroundColor3 = Color3.fromRGB(15,15,15)
    Toggle.Text = Name; Toggle.TextColor3 = Color3.new(1,1,1); Toggle.Font = Enum.Font.GothamBold
    Instance.new("UICorner", Toggle).CornerRadius = UDim.new(0, 10)
    
    Toggle.MouseButton1Click:Connect(function()
        local state = not (Toggle.BackgroundColor3 == Color3.fromRGB(60,0,0))
        Toggle.BackgroundColor3 = state and Color3.fromRGB(60,0,0) or Color3.fromRGB(15,15,15)
        Callback(state)
    end)
end

CreateCombatToggle("Silent Aim", function(v) Combat.SilentAim = v end)
CreateCombatToggle("Auto Shoot", function(v) Combat.AutoShoot = v end)

---------------------------------------------------
-- 3. BUCLE DE EJECUCIÓN CENTRAL
---------------------------------------------------
RunService.Heartbeat:Connect(function()
    if Combat.AutoShoot then fire() end
    if Combat.SilentAim then 
        -- Aquí puedes añadir el hook de tu Silent Aim
    end
end)
