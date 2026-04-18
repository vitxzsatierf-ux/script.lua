--// SERVICES
local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local TweenService = game:GetService("TweenService")
local SoundService = game:GetService("SoundService")

local LP = Players.LocalPlayer

--// 🎨 DESIGN MODERNO (Tsc Style)
local COLOR_BG = Color3.fromRGB(15, 15, 15)
local COLOR_ELEMENT = Color3.fromRGB(25, 25, 25)
local ACCENT_COLOR = Color3.fromRGB(138, 43, 226) -- Roxo Neon
local TEXT_COLOR = Color3.fromRGB(240, 240, 240)

-- CORES DO ESP
local ESP_FILL_COLOR = Color3.fromRGB(255, 0, 0) -- Vermelho
local ESP_OUTLINE_COLOR = Color3.fromRGB(255, 255, 255) -- Branca

local FONT_MAIN = Enum.Font.GothamBold
local FONT_TITLE = Enum.Font.GothamBlack

--// SETTINGS 
local ESP_ENABLED = false
local AIMBOT_ENABLED = false

local MAX_DISTANCE = 400
local FOV_RADIUS = 70
local AIM_SMOOTHNESS = 0.2
local AIM_OFFSET = Vector3.new(0,0.5,0)

local TARGET_PARTS = {"Head","UpperTorso","HumanoidRootPart"}
local CURRENT_PART_INDEX = 1

local GUI_VISIBLE = true

--// SOUND
local function PlaySound()
	local s = Instance.new("Sound")
	s.SoundId = "rbxassetid://6895079853"
	s.Volume = 0.5
	s.PlayOnRemove = true
	s.Parent = SoundService
	s:Destroy()
end

--// STORAGE
local storage = Instance.new("Folder", workspace)
storage.Name = "Tsc_Render_Cache"

--// FOV
local FOVCircle
pcall(function()
	FOVCircle = Drawing.new("Circle")
	FOVCircle.Color = ACCENT_COLOR
	FOVCircle.Thickness = 1.5
	FOVCircle.NumSides = 100
	FOVCircle.Filled = false
end)

--// CLEAR
local function Clear()
	storage:ClearAllChildren()
	for _,p in ipairs(Players:GetPlayers()) do
		if p.Character then
			local h = p.Character:FindFirstChild("Head")
			if h and h:FindFirstChild("TAG") then
				h.TAG:Destroy()
			end
		end
	end
end

--// ESP (VERMELHO COM BORDA BRANCA)
local function UpdateESP()
	if not ESP_ENABLED then return end

	local char = LP.Character
	if not char then return end
	local root = char:FindFirstChild("HumanoidRootPart")
	if not root then return end

	for _,p in ipairs(Players:GetPlayers()) do
		if p ~= LP and p.Character then
			local c = p.Character
			local hum = c:FindFirstChildOfClass("Humanoid")
			local head = c:FindFirstChild("Head")
			local r = c:FindFirstChild("HumanoidRootPart")

			if not hum or hum.Health <= 0 or not r then continue end

			local dist = (r.Position - root.Position).Magnitude

			if dist > MAX_DISTANCE then
				local old = storage:FindFirstChild(p.UserId)
				if old then old:Destroy() end
				if head and head:FindFirstChild("TAG") then
					head.TAG:Destroy()
				end
				continue
			end

			local hl = storage:FindFirstChild(p.UserId)
			if not hl then
				hl = Instance.new("Highlight")
				hl.Name = p.UserId
				hl.Parent = storage
			end

			hl.Adornee = c
			hl.FillColor = ESP_FILL_COLOR
			hl.OutlineColor = ESP_OUTLINE_COLOR
			hl.FillTransparency = 0.5
			hl.OutlineTransparency = 0

			if head then
				local tag = head:FindFirstChild("TAG")

				if not tag then
					tag = Instance.new("BillboardGui", head)
					tag.Name = "TAG"
					tag.Size = UDim2.new(0,70,0,20)
					tag.StudsOffset = Vector3.new(0,2.5,0)
					tag.AlwaysOnTop = true

					local txt = Instance.new("TextLabel", tag)
					txt.Size = UDim2.new(1,0,0.6,0)
					txt.BackgroundTransparency = 1
					txt.TextScaled = true
					txt.Font = FONT_MAIN
					txt.TextColor3 = Color3.new(1,1,1)
					
					local stroke = Instance.new("UIStroke", txt)
					stroke.Thickness = 1.5
					stroke.Transparency = 0.5

					local bg = Instance.new("Frame", tag)
					bg.Size = UDim2.new(1,0,0.3,0)
					bg.Position = UDim2.new(0,0,0.7,0)
					bg.BackgroundColor3 = Color3.new(0,0,0)
					Instance.new("UICorner", bg).CornerRadius = UDim.new(0,4)

					local bar = Instance.new("Frame", bg)
					bar.Name = "Bar"
					bar.Size = UDim2.new(1,0,1,0)
					bar.BorderSizePixel = 0
					Instance.new("UICorner", bar).CornerRadius = UDim.new(0,4)
				end

				local txt = tag:FindFirstChildOfClass("TextLabel")
				local bar = tag:FindFirstChild("Frame").Bar

				local pct = math.clamp(hum.Health / hum.MaxHealth,0,1)
				txt.Text = math.floor(dist).."m" 
				bar.Size = UDim2.new(pct,0,1,0)
				bar.BackgroundColor3 = Color3.fromRGB(255*(1-pct),255*pct,0)
			end
		end
	end
end

--// GET CLOSEST
local function GetClosest()
	local mouse = UIS:GetMouseLocation()
	local closest, dist = nil, math.huge

	for _,p in ipairs(Players:GetPlayers()) do
		if p ~= LP and p.Character then
			local part = p.Character:FindFirstChild(TARGET_PARTS[CURRENT_PART_INDEX]) or p.Character:FindFirstChild("Head")

			if part then
				local pos, vis = Camera:WorldToViewportPoint(part.Position)
				if vis then
					local d = (Vector2.new(pos.X,pos.Y) - mouse).Magnitude
					if d < dist and d <= FOV_RADIUS then
						dist = d
						closest = part
					end
				end
			end
		end
	end
	return closest
end

--// LOOP RENDER (AIMBOT E FOV)
RunService.RenderStepped:Connect(function()
	local mouse = UIS:GetMouseLocation()

	if FOVCircle then
		FOVCircle.Position = mouse
		FOVCircle.Radius = FOV_RADIUS
		FOVCircle.Visible = AIMBOT_ENABLED
	end

	-- A LÓGICA DO AIMBOT: Ativo na variável (E) E pressionando o Botão Direito (M2)
	if AIMBOT_ENABLED and UIS:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
		local t = GetClosest()
		if t then
			local cam = Camera.CFrame.Position
			local dir = (t.Position + AIM_OFFSET - cam).Unit
			local sm = Camera.CFrame.LookVector:Lerp(dir, AIM_SMOOTHNESS)
			Camera.CFrame = CFrame.new(cam, cam + sm)
		end
	end
end)

--// UI 
local gui = Instance.new("ScreenGui", game.CoreGui)
gui.ResetOnSpawn = false

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0,240,0,280)
frame.Position = UDim2.new(0.02,0,0.35,0)
frame.BackgroundColor3 = COLOR_BG
Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 8)

local frameStroke = Instance.new("UIStroke", frame)
frameStroke.Color = ACCENT_COLOR
frameStroke.Thickness = 2
frameStroke.Transparency = 0.3

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1,0,0,30)
title.Position = UDim2.new(0,0,0.02,0)
title.BackgroundTransparency = 1
title.Text = "XenoScript by Tsc"
title.TextColor3 = TEXT_COLOR
title.Font = FONT_TITLE
title.TextSize = 18

local info = Instance.new("TextLabel", frame)
info.Size = UDim2.new(1,0,0,20)
info.Position = UDim2.new(0,0,0.9,0)
info.BackgroundTransparency = 1
info.Text = "E = Aim | L = ESP | R-CTRL = Hide"
info.TextColor3 = Color3.fromRGB(150, 150, 150)
info.Font = FONT_MAIN
info.TextSize = 12

local function btn(y,text)
	local b = Instance.new("TextButton", frame)
	b.Size = UDim2.new(0.8,0,0,30)
	b.Position = UDim2.new(0.1,0,y,0)
	b.Text = text
	b.BackgroundColor3 = COLOR_ELEMENT
	b.TextColor3 = TEXT_COLOR
	b.Font = FONT_MAIN
	b.TextSize = 14
	b.AutoButtonColor = false
	Instance.new("UICorner", b).CornerRadius = UDim.new(0,6)
	
	b.MouseEnter:Connect(function()
		TweenService:Create(b, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(40,40,40)}):Play()
	end)
	b.MouseLeave:Connect(function()
		local targetColor = (b.Text:find("ON") and ACCENT_COLOR or COLOR_ELEMENT)
		TweenService:Create(b, TweenInfo.new(0.2), {BackgroundColor3 = targetColor}):Play()
	end)
	
	return b
end

local aimB = btn(0.2,"AIMBOT: OFF")
local espB = btn(0.35,"ESP: OFF")
local partB = btn(0.5,"ALVO: Head")

-- SLIDER (FOV)
local function slider(y,min,max,callback)
	local h = Instance.new("Frame", frame)
	h.Size = UDim2.new(0.8,0,0,20)
	h.Position = UDim2.new(0.1,0,y,0)
	h.BackgroundTransparency = 1

	local line = Instance.new("Frame", h)
	line.Size = UDim2.new(1,0,0,4)
	line.Position = UDim2.new(0,0,0.5,-2)
	line.BackgroundColor3 = COLOR_ELEMENT
	Instance.new("UICorner", line)

	local fill = Instance.new("Frame", line)
	fill.Size = UDim2.new(0,0,1,0)
	fill.BackgroundColor3 = ACCENT_COLOR
	Instance.new("UICorner", fill)

	local knob = Instance.new("Frame", h)
	knob.Size = UDim2.new(0,12,0,12)
	knob.Position = UDim2.new(0,-6,0.5,-6)
	knob.BackgroundColor3 = Color3.new(1,1,1)
	Instance.new("UICorner", knob).CornerRadius = UDim.new(1,0)

	local dragging = false

	h.InputBegan:Connect(function(i)
		if i.UserInputType == Enum.UserInputType.MouseButton1 then dragging = true end
	end)

	UIS.InputEnded:Connect(function(i)
		if i.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
	end)

	RunService.RenderStepped:Connect(function()
		if dragging and GUI_VISIBLE then
			local pos = math.clamp((UIS:GetMouseLocation().X - h.AbsolutePosition.X)/h.AbsoluteSize.X,0,1)
			fill.Size = UDim2.new(pos,0,1,0)
			knob.Position = UDim2.new(pos,-6,0.5,-6)
			callback(min+(max-min)*pos)
		end
	end)
end

slider(0.68,60,300,function(v) FOV_RADIUS = math.floor(v) end)
slider(0.78,0.1,1,function(v) AIM_SMOOTHNESS = v end)

-- TOGGLES DA INTERFACE
local function ToggleAimbot()
	AIMBOT_ENABLED = not AIMBOT_ENABLED
	aimB.Text = AIMBOT_ENABLED and "AIMBOT: ON" or "AIMBOT: OFF"
	TweenService:Create(aimB, TweenInfo.new(0.3), {BackgroundColor3 = AIMBOT_ENABLED and ACCENT_COLOR or COLOR_ELEMENT}):Play()
	PlaySound()
end

local function ToggleESP()
	ESP_ENABLED = not ESP_ENABLED
	espB.Text = ESP_ENABLED and "ESP: ON" or "ESP: OFF"
	TweenService:Create(espB, TweenInfo.new(0.3), {BackgroundColor3 = ESP_ENABLED and ACCENT_COLOR or COLOR_ELEMENT}):Play()
	PlaySound()
	if not ESP_ENABLED then Clear() end
end

local function CyclePart()
	CURRENT_PART_INDEX = CURRENT_PART_INDEX % #TARGET_PARTS + 1
	partB.Text = "ALVO: "..TARGET_PARTS[CURRENT_PART_INDEX]
	PlaySound()
end

aimB.MouseButton1Click:Connect(ToggleAimbot)
espB.MouseButton1Click:Connect(ToggleESP)
partB.MouseButton1Click:Connect(CyclePart)

local function Fade(state)
	GUI_VISIBLE = state
	frame.Visible = state
	local goal = state and 0 or 1
	
	for _,v in ipairs(frame:GetDescendants()) do
		if v:IsA("TextLabel") or v:IsA("TextButton") then
			TweenService:Create(v,TweenInfo.new(0.2),{TextTransparency = goal}):Play()
			if v:IsA("TextButton") then
				local colorT = state and (v.Text:find("ON") and ACCENT_COLOR or COLOR_ELEMENT) or COLOR_ELEMENT
				TweenService:Create(v,TweenInfo.new(0.2),{BackgroundTransparency = goal, BackgroundColor3 = colorT}):Play()
			end
		elseif v:IsA("Frame") and v ~= frame then
			TweenService:Create(v,TweenInfo.new(0.2),{BackgroundTransparency = goal}):Play()
		elseif v:IsA("UIStroke") then
			TweenService:Create(v,TweenInfo.new(0.2),{Transparency = state and 0.3 or 1}):Play()
		end
	end
	TweenService:Create(frame,TweenInfo.new(0.2),{BackgroundTransparency = goal}):Play()
end

--// CONTROLE REFEITO (IGNORA BLOQUEIOS DO JOGO)
UIS.InputBegan:Connect(function(i, gp)
	-- Nova Trava Inteligente: Só bloqueia o comando se você estiver digitando no Chat
	if UIS:GetFocusedTextBox() then return end
	
	if i.KeyCode == Enum.KeyCode.E then ToggleAimbot() end
	if i.KeyCode == Enum.KeyCode.L then ToggleESP() end
	if i.KeyCode == Enum.KeyCode.RightControl then Fade(not GUI_VISIBLE) end
end)

task.spawn(function()
	while true do
		pcall(UpdateESP)
		task.wait(0.1)
	end
end)
