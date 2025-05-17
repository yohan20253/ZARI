-- Script completo de Aimbot + ESP com menu em "P"
-- Deve ser inserido como LocalScript dentro de CheatGui

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

local gui = script.Parent
local frame = gui:WaitForChild("MainFrame")
local aimbotBtn = frame:WaitForChild("AimbotButton")
local espBtn = frame:WaitForChild("ESPButton")

-- Estado dos recursos
local aimbotEnabled = false
local espEnabled = false
local guiVisible = false
frame.Visible = false

-- Alvo do Aimbot
local function getClosestTarget()
	local closestPlayer = nil
	local shortestDistance = math.huge

	for _, p in pairs(Players:GetPlayers()) do
		if p ~= player and p.Team ~= player.Team and p.Character and p.Character:FindFirstChild("HumanoidRootPart") and p.Character:FindFirstChild("Humanoid") and p.Character.Humanoid.Health > 0 then
			local pos, onScreen = camera:WorldToViewportPoint(p.Character.HumanoidRootPart.Position)
			if onScreen then
				local mousePos = UIS:GetMouseLocation()
				local dist = (Vector2.new(pos.X, pos.Y) - Vector2.new(mousePos.X, mousePos.Y)).Magnitude
				if dist < shortestDistance then
					shortestDistance = dist
					closestPlayer = p
				end
			end
		end
	end

	return closestPlayer
end

-- Aimbot loop
RunService.RenderStepped:Connect(function()
	if aimbotEnabled then
		local target = getClosestTarget()
		if target and target.Character and target.Character:FindFirstChild("Head") then
			camera.CFrame = CFrame.new(camera.CFrame.Position, target.Character.Head.Position)
		end
	end
end)

-- ESP system
local espFolder = Instance.new("Folder", workspace)
espFolder.Name = "ESP_Objects"

local function addESP(playerTarget)
	if playerTarget.Character then
		local highlight = Instance.new("Highlight")
		highlight.Name = "ESPHighlight"
		highlight.FillColor = Color3.fromRGB(255, 0, 0)
		highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
		highlight.Adornee = playerTarget.Character
		highlight.Parent = espFolder
	end
end

local function removeESP()
	for _, obj in pairs(espFolder:GetChildren()) do
		obj:Destroy()
	end
end

local function updateESP()
	removeESP()
	for _, p in pairs(Players:GetPlayers()) do
		if p ~= player and p.Team ~= player.Team then
			addESP(p)
		end
	end
end

-- Atualiza ESP a cada 2 segundos
task.spawn(function()
	while true do
		if espEnabled then
			updateESP()
		else
			removeESP()
		end
		task.wait(2)
	end
end)

-- Botões
aimbotBtn.MouseButton1Click:Connect(function()
	aimbotEnabled = not aimbotEnabled
	aimbotBtn.Text = "Aimbot: " .. (aimbotEnabled and "ON" or "OFF")
end)

espBtn.MouseButton1Click:Connect(function()
	espEnabled = not espEnabled
	espBtn.Text = "ESP: " .. (espEnabled and "ON" or "OFF")
end)

-- Abrir/Fechar Menu com tecla P
UIS.InputBegan:Connect(function(input, gp)
	if not gp and input.KeyCode == Enum.KeyCode.P then
		guiVisible = not guiVisible
		frame.Visible = guiVisible
	end
end)
