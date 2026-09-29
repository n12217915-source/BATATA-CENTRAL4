--=============================================================
--  HITBOX EXPANDER — tamanho + transparência + modo "no lag"
--=============================================================
local Players           = game:GetService("Players")
local RunService        = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LocalPlayer       = Players.LocalPlayer

local Config = {
	Enabled       = false,
	Size          = 5,      -- studs (em cada eixo)
	Transparency  = 1,      -- 1 = invisível (recomendado)
	NoLag         = true,   -- aplica só no alvo mais próximo? senão em todos
	NoLagRange    = 100,
}

local originals = {}   -- [part] = {size, transparency, cancollide, cantouch}

local function expand(part)
	if not originals[part] then
		originals[part] = {
			size = part.Size,
			transparency = part.Transparency,
			cancollide = part.CanCollide,
			cantouch = part.CanTouch,
		}
	end
	local base = originals[part].size
	part.Size = Vector3.new(base.X + Config.Size, base.Y + Config.Size, base.Z + Config.Size)
	part.Transparency = Config.Transparency
	part.CanCollide = false
	part.CanTouch = true
end

local function restore(part)
	if originals[part] then
		part.Size = originals[part].size
		part.Transparency = originals[part].transparency
		part.CanCollide = originals[part].cancollide
		part.CanTouch = originals[part].cantouch
		originals[part] = nil
	end
end

local function restoreAll()
	for part in pairs(originals) do
		if part and part.Parent then restore(part) end
	end
	originals = {}
end

--=============================================================
--  Loop
--=============================================================
RunService.Heartbeat:Connect(function()
	if not Config.Enabled then
		if next(originals) then restoreAll() end
		return
	end

	local alvo = nil
	if Config.NoLag then
		-- escolhe mais próximo dentro do range
		local best, bestDist = nil, Config.NoLagRange
		local myPos = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
		if myPos then
			for _, plr in ipairs(Players:GetPlayers()) do
				if plr ~= LocalPlayer and plr.Character then
					local hrp = plr.Character:FindFirstChild("HumanoidRootPart")
					local hum = plr.Character:FindFirstChildOfClass("Humanoid")
					if hrp and hum and hum.Health > 0 then
						local d = (hrp.Position - myPos.Position).Magnitude
						if d < bestDist then bestDist = d best = plr end
					end
				end
			end
		end
		alvo = best
	end

	for _, plr in ipairs(Players:GetPlayers()) do
		if plr ~= LocalPlayer and plr.Character then
			local hum = plr.Character:FindFirstChildOfClass("Humanoid")
			local hrp = plr.Character:FindFirstChild("HumanoidRootPart")
			if hum and hum.Health > 0 and hrp then
				local aplicar = (not Config.NoLag) or (plr == alvo)
				if aplicar then
					expand(hrp)
					local head = plr.Character:FindFirstChild("Head")
					if head then expand(head) end
				else
					restore(hrp)
					local head = plr.Character:FindFirstChild("Head")
					if head then restore(head) end
				end
			end
		end
	end
end)

--=============================================================
--  Registro (delay 7.5s)
--=============================================================
task.wait(7.5)
local api = ReplicatedStorage:WaitForChild("BatataHub_RegisterTab")

local ok, err = api:Invoke("Batata001", {
	Name = "HITBOX",
	BuildContent = function(page, ctx)
		local function makeToggle(text, y, initial, onChange)
			local btn = Instance.new("TextButton")
			btn.Position = UDim2.fromOffset(9,y)
			btn.Size = UDim2.new(1,-18,0,22)
			btn.BackgroundColor3 = ctx.colors.BUTTON or Color3.fromRGB(35,35,35)
			btn.Text = ""
			btn.BorderSizePixel = 0
			btn.Parent = page
			local c = Instance.new("UICorner") c.CornerRadius = UDim.new(0,5) c.Parent = btn
			local lbl = Instance.new("TextLabel")
			lbl.BackgroundTransparency = 1
			lbl.Position = UDim2.fromOffset(8,0)
			lbl.Size = UDim2.new(1,-50,1,0)
			lbl.Font = Enum.Font.Gotham
			lbl.TextSize = 11
			lbl.TextColor3 = ctx.colors.TEXT
			lbl.TextXAlignment = Enum.TextXAlignment.Left
			lbl.Text = text
			lbl.Parent = btn
			local st = Instance.new("TextLabel")
			st.BackgroundTransparency = 1
			st.Position = UDim2.new(1,-45,0,0)
			st.Size = UDim2.fromOffset(40,22)
			st.Font = Enum.Font.GothamBold
			st.TextSize = 11
			st.TextColor3 = initial and Color3.fromRGB(90,220,120) or Color3.fromRGB(220,90,90)
			st.Text = initial and "ON" or "OFF"
			st.Parent = btn
			btn.MouseButton1Click:Connect(function()
				initial = not initial
				st.Text = initial and "ON" or "OFF"
				st.TextColor3 = initial and Color3.fromRGB(90,220,120) or Color3.fromRGB(220,90,90)
				onChange(initial)
			end)
		end

		local function makeSlider(text, y, min, max, initial, onChange)
			local frame = Instance.new("Frame")
			frame.Position = UDim2.fromOffset(9,y)
			frame.Size = UDim2.new(1,-18,0,30)
			frame.BackgroundTransparency = 1
			frame.Parent = page
			local lbl = Instance.new("TextLabel")
			lbl.BackgroundTransparency = 1
			lbl.Size = UDim2.new(1,0,0,14)
			lbl.Font = Enum.Font.Gotham
			lbl.TextSize = 11
			lbl.TextColor3 = ctx.colors.SUBTEXT
			lbl.TextXAlignment = Enum.TextXAlignment.Left
			lbl.Text = text..": "..tostring(initial)
			lbl.Parent = frame
			local bar = Instance.new("Frame")
			bar.Position = UDim2.fromOffset(0,18)
			bar.Size = UDim2.new(1,0,0,6)
			bar.BackgroundColor3 = Color3.fromRGB(50,50,50)
			bar.BorderSizePixel = 0
			bar.Parent = frame
			local bc = Instance.new("UICorner") bc.CornerRadius = UDim.new(1,0) bc.Parent = bar
			local fill = Instance.new("Frame")
			fill.Size = UDim2.new((initial-min)/(max-min),0,1,0)
			fill.BackgroundColor3 = ctx.colors.ACCENT or Color3.fromRGB(120,180,255)
			fill.BorderSizePixel = 0
			fill.Parent = bar
			local fc = Instance.new("UICorner") fc.CornerRadius = UDim.new(1,0) fc.Parent = fill
			local dragging = false
			local function update(input)
				local pos = math.clamp((input.Position.X - bar.AbsolutePosition.X)/bar.AbsoluteSize.X,0,1)
				fill.Size = UDim2.new(pos,0,1,0)
				local val = math.floor(min+(max-min)*pos+0.5)
				lbl.Text = text..": "..tostring(val)
				onChange(val)
			end
			bar.InputBegan:Connect(function(input)
				if input.UserInputType == Enum.UserInputType.MouseButton1
				or input.UserInputType == Enum.UserInputType.Touch then
					dragging = true update(input)
				end
			end)
			UserInputService.InputChanged:Connect(function(input)
				if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement
				or input.UserInputType == Enum.UserInputType.Touch) then update(input) end
			end)
			UserInputService.InputEnded:Connect(function(input)
				if input.UserInputType == Enum.UserInputType.MouseButton1
				or input.UserInputType == Enum.UserInputType.Touch then dragging = false end
			end)
		end

		local t = Instance.new("TextLabel")
		t.BackgroundTransparency = 1
		t.Position = UDim2.fromOffset(9,8)
		t.Size = UDim2.new(1,-18,0,17)
		t.Font = Enum.Font.GothamBlack
		t.Text = "HITBOX EXPANDER"
		t.TextSize = 14
		t.TextColor3 = ctx.colors.TEXT
		t.TextXAlignment = Enum.TextXAlignment.Left
		t.Parent = page

		local scroll = Instance.new("ScrollingFrame")
		scroll.Position = UDim2.fromOffset(0,32)
		scroll.Size = UDim2.new(1,0,1,-32)
		scroll.BackgroundTransparency = 1
		scroll.BorderSizePixel = 0
		scroll.ScrollBarThickness = 4
		scroll.CanvasSize = UDim2.new(0,0,0,320)
		scroll.Parent = page

		makeToggle("Ativado",       8,  Config.Enabled, function(v)
			Config.Enabled=v
			if not v then restoreAll() end
		end)
		makeToggle("Modo No-Lag",   36, Config.NoLag,   function(v) Config.NoLag=v end)
		makeSlider("Tamanho",       72, 1, 30, Config.Size, function(v) Config.Size=v end)
		makeSlider("Transp x100",   112,0, 100, math.floor(Config.Transparency*100), function(v) Config.Transparency=v/100 end)
		makeSlider("Range No-Lag",  152,10, 500, Config.NoLagRange, function(v) Config.NoLagRange=v end)

		local info = Instance.new("TextLabel")
		info.BackgroundTransparency = 1
		info.Position = UDim2.fromOffset(9,190)
		info.Size = UDim2.new(1,-18,0,60)
		info.Font = Enum.Font.Gotham
		info.TextSize = 10
		info.TextColor3 = ctx.colors.SUBTEXT
		info.TextXAlignment = Enum.TextXAlignment.Left
		info.TextYAlignment = Enum.TextYAlignment.Top
		info.TextWrapped = true
		info.Text = "Tamanho em studs, somado em cada eixo (fiel ao original).\n"
			.. "Transp = 1 → invisível. Aplica em HRP + Head."
		info.Parent = scroll
	end
})

if not ok then warn("[Hitbox] registro falhou:", err) end
