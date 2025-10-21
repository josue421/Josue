local Players = game:GetService("Players") local RunService = game:GetService("RunService") local UserInputService = game:GetService("UserInputService") local LocalPlayer = Players.LocalPlayer local Camera = workspace.CurrentCamera

-- Config local HubName = "JOSU" local WalkspeedDefault = 16 local SpeedValue = 50 local JumpPowerDefault = 50 local FlySpeed = 60

local state = { Invisible = false, Speed = false, Salto = false, Holo = false, Fly = false, Noclip = false }

-- GUI local PlayerGui = LocalPlayer:WaitForChild("PlayerGui") local screenGui = Instance.new("ScreenGui") screenGui.Name = HubName .. "_GUI" screenGui.ResetOnSpawn = false screenGui.Parent = PlayerGui

local frame = Instance.new("Frame") frame.Size = UDim2.new(0, 300, 0, 400) frame.Position = UDim2.new(0.5, -150, 0.5, -200) frame.BackgroundColor3 = Color3.fromRGB(28,28,28) frame.BorderSizePixel = 0 frame.Parent = screenGui

local title = Instance.new("TextLabel") title.Size = UDim2.new(1,0,0,40) title.Position = UDim2.new(0,0,0,0) title.BackgroundTransparency = 1 title.Text = HubName title.TextColor3 = Color3.fromRGB(255,255,255) title.Font = Enum.Font.GothamBold title.TextScaled = true title.Parent = frame

-- Botones de cerrar / abrir local closeBtn = Instance.new("TextButton") closeBtn.Size = UDim2.new(0, 40, 0, 30) closeBtn.Position = UDim2.new(1,-90,0,10) closeBtn.BackgroundColor3 = Color3.fromRGB(150,0,0) closeBtn.Text = "-" closeBtn.TextColor3 = Color3.fromRGB(255,255,255) closeBtn.Font = Enum.Font.GothamBold closeBtn.TextScaled = true closeBtn.Parent = frame

local openBtn = Instance.new("TextButton") openBtn.Size = UDim2.new(0, 40, 0, 30) openBtn.Position = UDim2.new(1,-40,0,10) openBtn.BackgroundColor3 = Color3.fromRGB(0,150,0) openBtn.Text = "+" openBtn.TextColor3 = Color3.fromRGB(255,255,255) openBtn.Font = Enum.Font.GothamBold openBtn.TextScaled = true openBtn.Parent = frame

closeBtn.MouseButton1Click:Connect(function() frame.Visible = false end)

openBtn.MouseButton1Click:Connect(function() frame.Visible = true end)

-- Función para crear botones local function makeToggle(y, label) local btn = Instance.new("TextButton") btn.Size = UDim2.new(1,-20,0,36) btn.Position = UDim2.new(0,10,0,y) btn.BackgroundColor3 = Color3.fromRGB(60,60,60) btn.Text = label..": OFF" btn.Font = Enum.Font.Gotham btn.TextColor3 = Color3.fromRGB(220,220,220) btn.TextScaled = true btn.Parent = frame return btn end

-- Crear botones local y = 0.12 local toggles = {} local names = {"Invisible","Speed","Salto","Holo","Fly","Noclip"} for _, n in ipairs(names) do toggles[n] = makeToggle(y*400, n) y = y + 0.09 end

-- Funciones básicas local function getChar() local char = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait() local hum = char:FindFirstChildOfClass("Humanoid") return char, hum end

-- Invisible local function setInvisible(val) local char = LocalPlayer.Character if not char then return end for _, part in pairs(char:GetDescendants()) do if part:IsA("BasePart") then part.LocalTransparencyModifier = val and 1 or 0 part.CanCollide = not val end end end

-- Speed local function setSpeed(val) local _, hum = getChar() if hum then hum.WalkSpeed = val and SpeedValue or WalkspeedDefault end end

-- Salto infinito seguro local infJumpConn local function setInfiniteJump(val) local char, hum = getChar() if not hum then return end hum.JumpPower = JumpPowerDefault -- aseguramos valor por defecto seguro if val then infJumpConn = UserInputService.JumpRequest:Connect(function() if hum and hum.Health > 0 then hum:ChangeState(Enum.HumanoidStateType.Jumping) end end) else if infJumpConn then infJumpConn:Disconnect(); infJumpConn = nil end end end

-- Fly simple local flyConn local function toggleFly(val) local char, hum = getChar() local root = char:FindFirstChild("HumanoidRootPart") if not root then return end if flyConn then flyConn:Disconnect(); flyConn=nil end if val then flyConn = RunService.RenderStepped:Connect(function() local move = Vector3.new() if UserInputService:IsKeyDown(Enum.KeyCode.W) then move = move + Camera.CFrame.LookVector end if UserInputService:IsKeyDown(Enum.KeyCode.S) then move = move - Camera.CFrame.LookVector end if UserInputService:IsKeyDown(Enum.KeyCode.A) then move = move - Camera.CFrame.RightVector end if UserInputService:IsKeyDown(Enum.KeyCode.D) then move = move + Camera.CFrame.RightVector end if UserInputService:IsKeyDown(Enum.KeyCode.Space) then move = move + Vector3.new(0,1,0) end if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then move = move - Vector3.new(0,1,0) end if move.Magnitude > 0 then root.Velocity = move.Unit * FlySpeed end end) end end

-- Noclip simple local noclipConn local function toggleNoclip(val) if noclipConn then noclipConn:Disconnect(); noclipConn=nil end if val then noclipConn = RunService.Stepped:Connect(function() local char = LocalPlayer.Character if char then for _, part in pairs(char:GetDescendants()) do if part:IsA("BasePart") then part.CanCollide = false end end end end) end end

-- Holo simple (ESP líneas) local espLines = {} local espConn local function toggleESP(val) if espConn then espConn:Disconnect(); espConn=nil end for _, l in pairs(espLines) do pcall(function() l:Remove() end) end espLines = {} if val then espConn = RunService.RenderStepped:Connect(function() for _, p in pairs(Players:GetPlayers()) do if p~=LocalPlayer and p.Character and p.Character:FindFirstChild("Head") then local head = p.Character.Head local screenPos, onScreen = Camera:WorldToViewportPoint(head.Position) if onScreen then if not espLines[p.UserId] then local line = Drawing.new("Line") line.Thickness = 1 line.Transparency = 1 line.Color = Color3.fromRGB(0,255,255) line.Visible = true espLines[p.UserId] = line end local line = espLines[p.UserId] line.From = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y) line.To = Vector2.new(screenPos.X, screenPos.Y) line.Visible = true else if espLines[p.UserId] then espLines[p.UserId].Visible=false end end end end end) end end

-- Conectar botones for name, btn in pairs(toggles) do btn.MouseButton1Click:Connect(function() state[name] = not state[name] btn.Text = name..": "..(state[name] and "ON" or "OFF") if name=="Invisible" then setInvisible(state[name]) end if name=="Speed" then setSpeed(state[name]) end if name=="Salto" then setInfiniteJump(state[name]) end if name=="Holo" then toggleESP(state[name]) end if name=="Fly" then toggleFly(state[name]) end if name=="Noclip" then toggleNoclip(state[name]) end end) end

title.Text = HubName.." - listo (Usa (-) para cerrar / (+) para abrir)"# Josue
Viva los juegos
