-- Frost Boost Fps (Roblox / Luau)
-- FPS boost + FPS/Ping counter, mobile compatible
-- Mobile + PC compatible, draggable, animated GUI

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Lighting = game:GetService("Lighting")
local Stats = game:GetService("Stats")

local player = Players.LocalPlayer

-- Parent the GUI safely
local function getParent()
	local ok, ui = pcall(function()
		return (gethui and gethui()) or game:GetService("CoreGui")
	end)
	if ok and ui then return ui end
	return player:WaitForChild("PlayerGui")
end

local old = getParent():FindFirstChild("FrostBoostFPS")
if old then old:Destroy() end

local gui = Instance.new("ScreenGui")
gui.Name = "FrostBoostFPS"
gui.ResetOnSpawn = false
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.Parent = getParent()

local isMobile = UserInputService.TouchEnabled and not UserInputService.KeyboardEnabled
local W, H = isMobile and 210 or 240, 332

-- Main frame
local main = Instance.new("Frame")
main.Name = "Main"
main.Size = UDim2.fromOffset(W, H)
main.Position = UDim2.new(0.5, -W / 2, 0.15, 0)
main.BackgroundColor3 = Color3.fromRGB(18, 18, 26)
main.BorderSizePixel = 0
main.ClipsDescendants = true
main.Parent = gui
Instance.new("UICorner", main).CornerRadius = UDim.new(0, 12)

-- Animated rainbow border
local stroke = Instance.new("UIStroke")
stroke.Thickness = 2
stroke.Color = Color3.new(1, 1, 1)
stroke.Parent = main
local grad = Instance.new("UIGradient")
grad.Color = ColorSequence.new({
	ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 0, 128)),
	ColorSequenceKeypoint.new(0.33, Color3.fromRGB(0, 200, 255)),
	ColorSequenceKeypoint.new(0.66, Color3.fromRGB(120, 255, 120)),
	ColorSequenceKeypoint.new(1, Color3.fromRGB(255, 0, 128)),
})
grad.Parent = stroke

-- Header (drag handle)
local header = Instance.new("Frame")
header.Size = UDim2.new(1, 0, 0, 34)
header.BackgroundColor3 = Color3.fromRGB(28, 28, 40)
header.BorderSizePixel = 0
header.Parent = main

local title = Instance.new("TextLabel")
title.BackgroundTransparency = 1
title.Size = UDim2.new(1, -70, 1, 0)
title.Position = UDim2.fromOffset(12, 0)
title.Font = Enum.Font.GothamBold
title.TextSize = 13
title.TextXAlignment = Enum.TextXAlignment.Left
title.TextColor3 = Color3.new(1, 1, 1)
title.Text = "❄ FROST BOOST FPS"
title.Parent = header
local tg = Instance.new("UIGradient")
tg.Color = grad.Color
tg.Parent = title

local function headerButton(text, xOffset)
	local b = Instance.new("TextButton")
	b.Size = UDim2.fromOffset(26, 26)
	b.Position = UDim2.new(1, xOffset, 0.5, -13)
	b.BackgroundColor3 = Color3.fromRGB(45, 45, 65)
	b.Text = text
	b.Font = Enum.Font.GothamBold
	b.TextSize = 14
	b.TextColor3 = Color3.new(1, 1, 1)
	b.AutoButtonColor = true
	b.Parent = header
	Instance.new("UICorner", b).CornerRadius = UDim.new(0, 8)
	return b
end
local minBtn = headerButton("–", -66)
local closeBtn = headerButton("X", -34)

-- Stat labels
local function statLabel(y, color)
	local l = Instance.new("TextLabel")
	l.BackgroundTransparency = 1
	l.Size = UDim2.new(0.5, -14, 0, 24)
	l.Position = UDim2.new(y, y == 0 and 12 or 2, 0, 42)
	l.Font = Enum.Font.GothamBold
	l.TextSize = 18
	l.TextXAlignment = Enum.TextXAlignment.Left
	l.TextColor3 = color
	l.Parent = main
	return l
end
local fpsLabel = statLabel(0, Color3.fromRGB(120, 255, 120))
local pingLabel = statLabel(0.5, Color3.fromRGB(0, 200, 255))
fpsLabel.Text = "FPS: --"
pingLabel.Text = "Ping: --"

-- Mini FPS graph
local graph = Instance.new("Frame")
graph.Size = UDim2.new(1, -24, 0, 50)
graph.Position = UDim2.fromOffset(12, 72)
graph.BackgroundColor3 = Color3.fromRGB(12, 12, 18)
graph.BorderSizePixel = 0
graph.ClipsDescendants = true
graph.Parent = main
Instance.new("UICorner", graph).CornerRadius = UDim.new(0, 8)

local BARS = 30
local bars = {}
for i = 1, BARS do
	local bar = Instance.new("Frame")
	bar.AnchorPoint = Vector2.new(0, 1)
	bar.Size = UDim2.new(1 / BARS, -1, 0.1, 0)
	bar.Position = UDim2.new((i - 1) / BARS, 0, 1, 0)
	bar.BorderSizePixel = 0
	bar.BackgroundColor3 = Color3.fromRGB(120, 255, 120)
	bar.Parent = graph
	bars[i] = bar
end

-- Boost button
local boostBtn = Instance.new("TextButton")
boostBtn.Size = UDim2.new(1, -24, 0, 38)
boostBtn.Position = UDim2.fromOffset(12, 132)
boostBtn.BackgroundColor3 = Color3.fromRGB(0, 160, 90)
boostBtn.Text = "ENABLE BOOST"
boostBtn.Font = Enum.Font.GothamBold
boostBtn.TextSize = 15
boostBtn.TextColor3 = Color3.new(1, 1, 1)
boostBtn.Parent = main
Instance.new("UICorner", boostBtn).CornerRadius = UDim.new(0, 10)

local status = Instance.new("TextLabel")
status.BackgroundTransparency = 1
status.Size = UDim2.new(1, -24, 0, 40)
status.Position = UDim2.fromOffset(12, 284)
status.Font = Enum.Font.Gotham
status.TextSize = 12
status.TextWrapped = true
status.TextColor3 = Color3.fromRGB(170, 170, 190)
status.Text = "Boost is OFF. Tap to lower graphics and raise FPS."
status.Parent = main

------------------------------------------------------------
-- Dragging (mouse + touch)
------------------------------------------------------------
local dragging, dragStart, startPos
header.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1
		or input.UserInputType == Enum.UserInputType.Touch then
		dragging = true
		dragStart = input.Position
		startPos = main.Position
		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then
				dragging = false
			end
		end)
	end
end)
UserInputService.InputChanged:Connect(function(input)
	if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement
		or input.UserInputType == Enum.UserInputType.Touch) then
		local d = input.Position - dragStart
		main.Position = UDim2.new(
			startPos.X.Scale, startPos.X.Offset + d.X,
			startPos.Y.Scale, startPos.Y.Offset + d.Y
		)
	end
end)

------------------------------------------------------------
-- Minimize / close
------------------------------------------------------------
local minimized = false
minBtn.MouseButton1Click:Connect(function()
	minimized = not minimized
	TweenService:Create(main, TweenInfo.new(0.25, Enum.EasingStyle.Quad), {
		Size = minimized and UDim2.fromOffset(W, 34) or UDim2.fromOffset(W, H)
	}):Play()
	minBtn.Text = minimized and "+" or "–"
end)

------------------------------------------------------------
-- FPS boost logic
------------------------------------------------------------
local boosted = false
local saved = {}          -- original values for restore
local descConn

local function remember(inst, prop)
	saved[#saved + 1] = { inst, prop, inst[prop] }
end

local function optimize(inst)
	if inst:IsA("BasePart") then
		remember(inst, "Material")
		remember(inst, "Reflectance")
		remember(inst, "CastShadow")
		inst.Material = Enum.Material.SmoothPlastic
		inst.Reflectance = 0
		inst.CastShadow = false
	elseif inst:IsA("Decal") or inst:IsA("Texture") then
		remember(inst, "Transparency")
		inst.Transparency = 1
	elseif inst:IsA("ParticleEmitter") or inst:IsA("Trail")
		or inst:IsA("Smoke") or inst:IsA("Fire") or inst:IsA("Sparkles") then
		remember(inst, "Enabled")
		inst.Enabled = false
	elseif inst:IsA("PostEffect") then
		remember(inst, "Enabled")
		inst.Enabled = false
	end
end

local function enableBoost()
	boosted = true
	saved = {}

	-- Lighting
	for _, p in ipairs({ "GlobalShadows", "FogEnd", "Brightness" }) do
		pcall(remember, Lighting, p)
	end
	Lighting.GlobalShadows = false
	Lighting.FogEnd = 9e9
	Lighting.Brightness = 2
	for _, e in ipairs(Lighting:GetChildren()) do optimize(e) end

	-- Terrain water
	local terrain = workspace:FindFirstChildOfClass("Terrain")
	if terrain then
		for _, p in ipairs({ "WaterWaveSize", "WaterWaveSpeed", "WaterReflectance", "WaterTransparency" }) do
			pcall(remember, terrain, p)
			pcall(function() terrain[p] = 0 end)
		end
	end

	-- Quality level
	pcall(function()
		saved[#saved + 1] = { settings().Rendering, "QualityLevel", settings().Rendering.QualityLevel }
		settings().Rendering.QualityLevel = Enum.QualityLevel.Level01
	end)

	-- Workspace (chunked so it doesn't freeze)
	task.spawn(function()
		local n = 0
		for _, inst in ipairs(workspace:GetDescendants()) do
			if not boosted then break end
			if not inst:IsDescendantOf(player.Character or workspace) then
				pcall(optimize, inst)
			end
			n += 1
			if n % 400 == 0 then task.wait() end
		end
	end)

	descConn = workspace.DescendantAdded:Connect(function(inst)
		if boosted then pcall(optimize, inst) end
	end)

	boostBtn.Text = "DISABLE BOOST"
	TweenService:Create(boostBtn, TweenInfo.new(0.2), { BackgroundColor3 = Color3.fromRGB(190, 50, 60) }):Play()
	status.Text = "Boost is ON. Shadows, textures, particles and effects reduced."
end

local function disableBoost()
	boosted = false
	if descConn then descConn:Disconnect() descConn = nil end
	for i, v in ipairs(saved) do
		pcall(function() v[1][v[2]] = v[3] end)
		if i % 400 == 0 then task.wait() end
	end
	saved = {}
	boostBtn.Text = "ENABLE BOOST"
	TweenService:Create(boostBtn, TweenInfo.new(0.2), { BackgroundColor3 = Color3.fromRGB(0, 160, 90) }):Play()
	status.Text = "Boost is OFF. Original settings restored."
end

boostBtn.MouseButton1Click:Connect(function()
	if boosted then disableBoost() else enableBoost() end
end)

------------------------------------------------------------
-- Toggle buttons: Shadows / Hide Other Players / Dark Textures
------------------------------------------------------------
local scroll = Instance.new("ScrollingFrame")
scroll.Position = UDim2.fromOffset(12, 178)
scroll.Size = UDim2.new(1, -24, 0, 100)
scroll.BackgroundTransparency = 1
scroll.BorderSizePixel = 0
scroll.ScrollBarThickness = 3
scroll.ScrollBarImageColor3 = Color3.fromRGB(0, 200, 255)
scroll.CanvasSize = UDim2.new()
scroll.AutomaticCanvasSize = Enum.AutomaticSize.Y
scroll.ScrollingDirection = Enum.ScrollingDirection.Y
scroll.Parent = main
local scrollLayout = Instance.new("UIListLayout")
scrollLayout.Padding = UDim.new(0, 4)
scrollLayout.SortOrder = Enum.SortOrder.LayoutOrder
scrollLayout.Parent = scroll

local function makeToggle(text, order, initial, callback)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(1, -6, 0, 30)
	btn.LayoutOrder = order
	btn.BackgroundColor3 = Color3.fromRGB(32, 32, 46)
	btn.Text = ""
	btn.AutoButtonColor = true
	btn.Parent = scroll
	Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)

	local lbl = Instance.new("TextLabel")
	lbl.BackgroundTransparency = 1
	lbl.Size = UDim2.new(1, -60, 1, 0)
	lbl.Position = UDim2.fromOffset(10, 0)
	lbl.Font = Enum.Font.GothamMedium
	lbl.TextSize = 13
	lbl.TextXAlignment = Enum.TextXAlignment.Left
	lbl.TextColor3 = Color3.new(1, 1, 1)
	lbl.Text = text
	lbl.Parent = btn

	local pill = Instance.new("Frame")
	pill.Size = UDim2.fromOffset(36, 18)
	pill.Position = UDim2.new(1, -46, 0.5, -9)
	pill.BorderSizePixel = 0
	pill.Parent = btn
	Instance.new("UICorner", pill).CornerRadius = UDim.new(1, 0)

	local knob = Instance.new("Frame")
	knob.Size = UDim2.fromOffset(14, 14)
	knob.BackgroundColor3 = Color3.new(1, 1, 1)
	knob.BorderSizePixel = 0
	knob.Parent = pill
	Instance.new("UICorner", knob).CornerRadius = UDim.new(1, 0)

	local state = initial
	local function paint(animate)
		local pc = state and Color3.fromRGB(0, 170, 100) or Color3.fromRGB(70, 70, 90)
		local kp = state and UDim2.fromOffset(20, 2) or UDim2.fromOffset(2, 2)
		if animate then
			local ti = TweenInfo.new(0.15)
			TweenService:Create(pill, ti, { BackgroundColor3 = pc }):Play()
			TweenService:Create(knob, ti, { Position = kp }):Play()
		else
			pill.BackgroundColor3 = pc
			knob.Position = kp
		end
	end
	paint(false)

	btn.MouseButton1Click:Connect(function()
		state = not state
		paint(true)
		callback(state)
	end)

	return {
		set = function(on)
			if on == state then return end
			state = on
			paint(true)
		end,
		destroy = function() btn:Destroy() end,
	}
end

-- 1) Shadows ------------------------------------------------
local shadowToggle = makeToggle("Shadows", 178, Lighting.GlobalShadows, function(on)
	Lighting.GlobalShadows = on
	status.Text = on and "Shadows ON." or "Shadows OFF."
	task.spawn(function()
		local n = 0
		for _, inst in ipairs(workspace:GetDescendants()) do
			if inst:IsA("BasePart") then
				pcall(function() inst.CastShadow = on end)
			end
			n += 1
			if n % 400 == 0 then task.wait() end
		end
	end)
end)
-- keep the switch in sync if the boost button changes shadows
task.spawn(function()
	while gui.Parent do
		shadowToggle.set(Lighting.GlobalShadows)
		task.wait(0.5)
	end
end)

-- 2) Hide players (everyone, or pick specific people) ------
local hiddenState = {}   -- instance -> original value
local hideSet = {}       -- [Player] = true while hidden
local hideConns = {}     -- [Player] = { connections }
local rows = {}          -- [Player] = toggle row
local hideAllOn = false
local rowOrder = 1000

local function hideInst(inst)
	if hiddenState[inst] ~= nil then return end
	if inst:IsA("BasePart") then
		hiddenState[inst] = inst.LocalTransparencyModifier
		inst.LocalTransparencyModifier = 1
	elseif inst:IsA("Decal") then
		hiddenState[inst] = inst.Transparency
		inst.Transparency = 1
	elseif inst:IsA("BillboardGui") then
		hiddenState[inst] = inst.Enabled
		inst.Enabled = false
	end
end

local function restoreInst(inst, orig)
	pcall(function()
		if inst:IsA("BasePart") then
			inst.LocalTransparencyModifier = orig
		elseif inst:IsA("Decal") then
			inst.Transparency = orig
		elseif inst:IsA("BillboardGui") then
			inst.Enabled = orig
		end
	end)
end

local function setPlayerHidden(p, on)
	if p == player then return end
	if on then
		if hideSet[p] then return end
		hideSet[p] = true
		local conns = {}
		local function hook(char)
			for _, d in ipairs(char:GetDescendants()) do pcall(hideInst, d) end
			conns[#conns + 1] = char.DescendantAdded:Connect(function(d)
				pcall(hideInst, d)
			end)
		end
		if p.Character then hook(p.Character) end
		conns[#conns + 1] = p.CharacterAdded:Connect(hook)
		hideConns[p] = conns
	else
		if not hideSet[p] then return end
		hideSet[p] = nil
		for _, c in ipairs(hideConns[p] or {}) do c:Disconnect() end
		hideConns[p] = nil
		local char = p.Character
		if char then
			for inst, orig in pairs(hiddenState) do
				if inst:IsDescendantOf(char) then
					restoreInst(inst, orig)
					hiddenState[inst] = nil
				end
			end
		end
	end
end

makeToggle("Hide All Players", 212, false, function(on)
	hideAllOn = on
	for _, p in ipairs(Players:GetPlayers()) do
		if p ~= player then
			setPlayerHidden(p, on)
			if rows[p] then rows[p].set(on) end
		end
	end
	status.Text = on and "All other players hidden." or "Other players visible again."
end)

-- per-player picker (scroll down in the list)
local pickLabel = Instance.new("TextLabel")
pickLabel.LayoutOrder = 999
pickLabel.Size = UDim2.new(1, -6, 0, 20)
pickLabel.BackgroundTransparency = 1
pickLabel.Font = Enum.Font.GothamBold
pickLabel.TextSize = 12
pickLabel.TextXAlignment = Enum.TextXAlignment.Left
pickLabel.TextColor3 = Color3.fromRGB(0, 200, 255)
pickLabel.Text = "PICK WHO DISAPPEARS"
pickLabel.Parent = scroll

local function addRow(p)
	if p == player or rows[p] then return end
	rowOrder += 1
	rows[p] = makeToggle("• " .. p.DisplayName, rowOrder, hideSet[p] == true, function(on)
		setPlayerHidden(p, on)
		status.Text = (on and "Hidden: " or "Visible: ") .. p.DisplayName
	end)
end

local function removeRow(p)
	setPlayerHidden(p, false)
	if rows[p] then
		rows[p].destroy()
		rows[p] = nil
	end
end

for _, p in ipairs(Players:GetPlayers()) do addRow(p) end
Players.PlayerAdded:Connect(function(p)
	addRow(p)
	if hideAllOn then
		setPlayerHidden(p, true)
		if rows[p] then rows[p].set(true) end
	end
end)
Players.PlayerRemoving:Connect(removeRow)

-- 3) Dark textures ------------------------------------------
local DARK = Color3.fromRGB(38, 38, 44)
local DARK_TEX = Color3.fromRGB(45, 45, 50)
local darkSaved = {}
local darkConn

local function isCharPart(inst)
	local p = inst.Parent
	while p and p ~= workspace do
		if p:IsA("Model") and p:FindFirstChildOfClass("Humanoid") then return true end
		p = p.Parent
	end
	return false
end

local function darken(inst)
	if inst:IsA("BasePart") then
		if isCharPart(inst) then return end
		darkSaved[#darkSaved + 1] = { inst, "Color", inst.Color }
		inst.Color = DARK
	elseif inst:IsA("Decal") or inst:IsA("Texture") then
		if isCharPart(inst) then return end
		darkSaved[#darkSaved + 1] = { inst, "Color3", inst.Color3 }
		inst.Color3 = DARK_TEX
	elseif inst:IsA("SurfaceAppearance") then
		darkSaved[#darkSaved + 1] = { inst, "Color", inst.Color }
		inst.Color = DARK_TEX
	end
end

local function setDark(on)
	if on then
		darkSaved = {}
		for _, p in ipairs({ "Ambient", "OutdoorAmbient" }) do
			darkSaved[#darkSaved + 1] = { Lighting, p, Lighting[p] }
			Lighting[p] = Color3.fromRGB(32, 32, 38)
		end
		status.Text = "Dark textures ON."
		task.spawn(function()
			local n = 0
			for _, inst in ipairs(workspace:GetDescendants()) do
				pcall(darken, inst)
				n += 1
				if n % 400 == 0 then task.wait() end
			end
		end)
		darkConn = workspace.DescendantAdded:Connect(function(inst)
			pcall(darken, inst)
		end)
	else
		if darkConn then darkConn:Disconnect() darkConn = nil end
		task.spawn(function()
			for i, v in ipairs(darkSaved) do
				pcall(function() v[1][v[2]] = v[3] end)
				if i % 400 == 0 then task.wait() end
			end
			darkSaved = {}
		end)
		status.Text = "Dark textures OFF."
	end
end
makeToggle("Dark Textures", 246, false, setDark)

-- 3b) Auto Redeem Codes (sends "/code NAME" in chat) -------
local TextChatService = game:GetService("TextChatService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- CONFIG -------------------------------------------------
local CODE_COMMAND = "/code"        -- use "/redeem" etc. if your game uses another command
local DEFAULT_CODES = {}            -- pre-fill codes, e.g. { "WELCOME", "UPDATE1" }
local AUTO_REDEEM_ON_START = false  -- true = redeem DEFAULT_CODES automatically on execute
local REDEEM_DELAY = 2              -- seconds between codes (avoids chat spam limits)
------------------------------------------------------------

local function sendChat(msg)
	local sent = false
	pcall(function()
		if TextChatService.ChatVersion == Enum.ChatVersion.TextChatService then
			local channels = TextChatService:FindFirstChild("TextChannels")
			local general = channels and channels:FindFirstChild("RBXGeneral")
			if general then
				general:SendAsync(msg)
				sent = true
			end
		else
			local ev = ReplicatedStorage:FindFirstChild("DefaultChatSystemChatEvents")
			local say = ev and ev:FindFirstChild("SayMessageRequest")
			if say then
				say:FireServer(msg, "All")
				sent = true
			end
		end
	end)
	return sent
end

local function makeBox(order, placeholder, text)
	local box = Instance.new("TextBox")
	box.LayoutOrder = order
	box.Size = UDim2.new(1, -6, 0, 28)
	box.BackgroundColor3 = Color3.fromRGB(24, 24, 36)
	box.BorderSizePixel = 0
	box.Font = Enum.Font.Gotham
	box.TextSize = 13
	box.TextColor3 = Color3.new(1, 1, 1)
	box.PlaceholderText = placeholder
	box.PlaceholderColor3 = Color3.fromRGB(120, 120, 140)
	box.Text = text or ""
	box.ClearTextOnFocus = false
	box.TextXAlignment = Enum.TextXAlignment.Left
	box.Parent = scroll
	Instance.new("UICorner", box).CornerRadius = UDim.new(0, 8)
	local pad = Instance.new("UIPadding")
	pad.PaddingLeft = UDim.new(0, 8)
	pad.Parent = box
	return box
end

local cmdBox = makeBox(100, "command (default /code)", CODE_COMMAND)
local codesBox = makeBox(101, "codes: space or comma", table.concat(DEFAULT_CODES, " "))

local redeemBtn = Instance.new("TextButton")
redeemBtn.LayoutOrder = 102
redeemBtn.Size = UDim2.new(1, -6, 0, 30)
redeemBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 200)
redeemBtn.Text = "REDEEM CODES"
redeemBtn.Font = Enum.Font.GothamBold
redeemBtn.TextSize = 13
redeemBtn.TextColor3 = Co
