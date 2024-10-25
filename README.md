local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")

-- Configuración de velocidades y tiempo
local walkSpeed = 16
local runSpeed = 30
local maxRunTime = 5 -- Tiempo máximo de correr en segundos
local cooldownTime = 3 -- Tiempo de recuperación en segundos

-- Variables para resistencia
local isShiftPressed = false
local isCooldown = false
local runTime = 0

-- Servicios
local userInputService = game:GetService("UserInputService")
local replicatedStorage = game:GetService("ReplicatedStorage")
local runEvent = replicatedStorage:WaitForChild("RunEvent")

-- Función para verificar si el jugador se mueve hacia adelante
local function isMovingForward()
    local moveDirection = humanoid.MoveDirection
    local lookDirection = character.PrimaryPart.CFrame.LookVector
    return moveDirection:Dot(lookDirection) > 0 -- True si va hacia adelante
end

-- Detectar cuando Shift se presiona
userInputService.InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.LeftShift and not isCooldown then
        isShiftPressed = true
        if isMovingForward() then
            runEvent:FireServer(runSpeed)
        end
    end
end)

-- Detectar cuando Shift se suelta
userInputService.InputEnded:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.LeftShift then
        isShiftPressed = false
        runEvent:FireServer(walkSpeed)
        runTime = 0 -- Resetear el tiempo al soltar Shift
    end
end)

-- Verificar la dirección y tiempo de correr en cada frame
game:GetService("RunService").RenderStepped:Connect(function(deltaTime)
    if isShiftPressed and isMovingForward() and not isCooldown then
        runTime = runTime + deltaTime
        if runTime >= maxRunTime then
            -- Si se alcanza el límite de correr, activar cooldown
            isCooldown = true
            isShiftPressed = false
            runEvent:FireServer(walkSpeed) -- Volver a caminar
            runTime = 0

            -- Iniciar temporizador de recuperación
            task.delay(cooldownTime, function()
                isCooldown = false
            end)
        else
            runEvent:FireServer(runSpeed) -- Mantener corriendo
        end
    elseif not isMovingForward() then
        -- Resetear tiempo si deja de moverse hacia adelante
        runTime = 0
    end
end)
