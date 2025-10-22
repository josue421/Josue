if game.CoreGui:FindFirstChild("JOSU_Hub") then
	game.CoreGui.JOSU_Hub:Destroy()
end

-- Crear GUI segura
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "JOSU_Hub"
ScreenGui.ResetOnSpawn = false
if syn and syn.protect_gui then
	syn.protect_gui(ScreenGui)
end
ScreenGui.Parent = game:GetService("CoreGui")

-- Fondo del hub
local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 220, 0, 220)
Frame.Position = UDim2.new(0.5, -110, 0.5, -110)
Frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
Frame.BorderSizePixel = 0
Frame.Active = true
Frame.Draggable = true
Frame.Parent = ScreenGui

local Title = Instance.new("TextLabel")
Title.Text = "🌐 JOSU HUB 🌐"
Title.Size = UDim2.new(1, 0, 0, 40)
Title.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
Title.TextColor3 = Color3.fromRGB(0, 255, 200)
Title.Font = Enum.Font.SourceSansBold
Title.TextSize = 20
Title.Parent = Frame

-- Función crear botones
local function makeButton(text, y)
	local btn = Instance.new("TextButton")
	btn.Text = text
	btn.Size = UDim2.new(0.8, 0, 0, 40)
	btn.Position = UDim2.new(0.1, 0, 0, y)
	btn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
	btn.TextColor3 = Color3.fromRGB(255, 255, 255)
	btn.Font = Enum.Font.SourceSansBold
	btn.TextSize = 18
	btn.Parent = Frame
	btn.AutoButtonColor = true
	return btn
end

-- Botones
local btnTP = makeButton("TP (Guardar Pos)", 60)
local btnTP2 = makeButton("TP2 (Ir Pos)", 110)
local btnTras = makeButton("Tras (Paredes)", 160)

-- Variables
local savedCFrame = nil
local noclip = false
local RunService = game:GetService("RunService")

-- TP
btnTP.MouseButton1Click:Connect(function()
	local plr = game.Players.LocalPlayer
	if plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
		savedCFrame = plr.Character.HumanoidRootPart.CFrame
		btnTP.Text = "✅ Pos Guardada"
		task.wait(1)
		btnTP.Text = "TP (Guardar Pos)"
	end
end)

-- TP2
btnTP2.MouseButton1Click:Connect(function()
	local plr = game.Players.LocalPlayer
	if savedCFrame and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
		plr.Character.HumanoidRootPart.CFrame = savedCFrame
	end
end)

-- Tras (noclip)
btnTras.MouseButton1Click:Connect(function()
	noclip = not noclip
	btnTras.Text = noclip and "Tras: ON" or "Tras: OFF"
end)

RunService.Stepped:Connect(function()
	if noclip then
		local char = game.Players.LocalPlayer.Character
		if char then
			for _, p in pairs(char:GetDescendants()) do
				if p:IsA("BasePart") then
					p.CanCollide = false
				end
			end
		end
	end
end)
