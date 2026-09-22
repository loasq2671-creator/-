--=====================================================================
--              B E R R E T A   •   UI LIBRARY v3.4
--=====================================================================

local Players  = game:GetService("Players")
local UIS      = game:GetService("UserInputService")
local TS       = game:GetService("TweenService")
local Lighting = game:GetService("Lighting")
local RunService = game:GetService("RunService")
local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local CoreGui  = game:GetService("CoreGui")
local LP       = Players.LocalPlayer

pcall(function()
	for _, v in ipairs(CoreGui:GetChildren()) do
		if v.Name == "Berreta" or v.Name == "BerretaCursor" or v.Name == "BerretaSilentAim" or v.Name == "BerretaTargetHUD" then
			v:Destroy()
		end
	end
end)

pcall(function()
	if not isfolder("Berreta") then makefolder("Berreta") end
	if not isfolder("Berreta/Configs") then makefolder("Berreta/Configs") end
end)

local function getParent()
	if gethui then
		local ok, res = pcall(gethui)
		if ok and res then return res end
	end
	local ok, res = pcall(function() return CoreGui end)
	if ok and res then return res end
	return LP:WaitForChild("PlayerGui")
end

local Theme = {
	Bg          = Color3.fromRGB(15, 15, 20),
	BgGrad      = Color3.fromRGB(22, 22, 30),
	Sidebar     = Color3.fromRGB(12, 12, 17),
	SidebarHl   = Color3.fromRGB(20, 20, 28),
	Element     = Color3.fromRGB(24, 24, 32),
	ElementHl   = Color3.fromRGB(32, 32, 42),
	ElementGrad = Color3.fromRGB(30, 30, 40),
	Accent      = Color3.fromRGB(139, 92, 246),
	AccentHl    = Color3.fromRGB(162, 122, 255),
	AccentDeep  = Color3.fromRGB(108, 66, 220),
	Pink        = Color3.fromRGB(236, 72, 153),
	Cyan        = Color3.fromRGB(34, 211, 238),
	Text        = Color3.fromRGB(245, 245, 250),
	SubText     = Color3.fromRGB(130, 130, 150),
	Stroke      = Color3.fromRGB(45, 45, 60),
	StrokeHl    = Color3.fromRGB(60, 60, 80),
	White       = Color3.fromRGB(255, 255, 255),
	Font        = Enum.Font.Gotham,
	FontMed     = Enum.Font.GothamMedium,
	FontBold    = Enum.Font.GothamBold,
}

local ChamsColors = {
	Purple = Color3.fromRGB(139, 92, 246),
	Red    = Color3.fromRGB(239, 68, 68),
	Green  = Color3.fromRGB(34, 197, 94),
	Blue   = Color3.fromRGB(59, 130, 246),
	Yellow = Color3.fromRGB(250, 204, 21),
	White  = Color3.fromRGB(255, 255, 255),
	Black  = Color3.fromRGB(30, 30, 30),
	Pink   = Color3.fromRGB(236, 72, 153),
	Cyan   = Color3.fromRGB(34, 211, 238),
	Orange = Color3.fromRGB(251, 146, 60),
	Gray   = Color3.fromRGB(120, 120, 130),
}

local function Create(class, props)
	local inst = Instance.new(class)
	local parent
	for k, v in next, props do
		if k == "Parent" then parent = v else inst[k] = v end
	end
	inst.Parent = parent
	return inst
end

local function Round(inst, r)
	Create("UICorner", { CornerRadius = UDim.new(0, r or 8), Parent = inst })
	return inst
end

local function Stroke(inst, color, thickness, transparency)
	Create("UIStroke", {
		Color = color or Theme.Stroke,
		Thickness = thickness or 1,
		Transparency = transparency or 0,
		ApplyStrokeMode = Enum.ApplyStrokeMode.Border,
		Parent = inst,
	})
	return inst
end

local function Gradient(inst, color1, color2, rotation)
	return Create("UIGradient", {
		Color = ColorSequence.new(color1, color2),
		Rotation = rotation or 90,
		Parent = inst,
	})
end

local function List(inst, gap)
	Create("UIListLayout", {
		Padding = UDim.new(0, gap or 8),
		SortOrder = Enum.SortOrder.LayoutOrder,
		HorizontalAlignment = Enum.HorizontalAlignment.Center,
		Parent = inst,
	})
	return inst
end

local function Pad(inst, x, y)
	Create("UIPadding", {
		PaddingTop    = UDim.new(0, y or 0),
		PaddingBottom = UDim.new(0, y or 0),
		PaddingLeft   = UDim.new(0, x or 0),
		PaddingRight  = UDim.new(0, x or 0),
		Parent = inst,
	})
	return inst
end

local function Tween(inst, time, props, style, dir)
	local tw = TS:Create(
		inst,
		TweenInfo.new(time or 0.15, style or Enum.EasingStyle.Quad, dir or Enum.EasingDirection.Out),
		props
	)
	tw:Play()
	return tw
end

local function Draggable(frame, handle)
	handle = handle or frame
	local dragging, dragInput, dragStart, startPos

	handle.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1
		or input.UserInputType == Enum.UserInputType.Touch then
			dragging  = true
			dragStart = input.Position
			startPos  = frame.Position
			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then
					dragging = false
				end
			end)
		end
	end)

	handle.InputChanged:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseMovement
		or input.UserInputType == Enum.UserInputType.Touch then
			dragInput = input
		end
	end)

	UIS.InputChanged:Connect(function(input)
		if dragging and input == dragInput then
			local d = input.Position - dragStart
			frame.Position = UDim2.new(
				startPos.X.Scale, startPos.X.Offset + d.X,
				startPos.Y.Scale, startPos.Y.Offset + d.Y
			)
		end
	end)
end

local Berreta = {}
Berreta.Flags       = {}
Berreta.Tabs        = {}
Berreta.CurrentTab  = nil
Berreta.ToggleKey   = Enum.KeyCode.RightShift

local gui = Create("ScreenGui", {
	Name = "Berreta",
	ResetOnSpawn = false,
	ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
	IgnoreGuiInset = true,
	DisplayOrder = 999,
	Parent = getParent(),
})

local blur = Instance.new("BlurEffect")
blur.Size = 0
blur.Parent = Lighting

local toggleBtn = Create("Frame", {
	Name = "ToggleButton",
	Size = UDim2.fromOffset(52, 52),
	Position = UDim2.new(0, 20, 0.5, -26),
	AnchorPoint = Vector2.new(0, 0.5),
	BackgroundColor3 = Theme.Accent,
	BorderSizePixel = 0,
	ZIndex = 100,
	Parent = gui,
})
Round(toggleBtn, 14)
Gradient(toggleBtn, Theme.Accent, Theme.Pink, 45)

local toggleBtnClick = Create("TextButton", {
	Size = UDim2.fromScale(1, 1),
	BackgroundTransparency = 1,
	Text = "",
	AutoButtonColor = false,
	ZIndex = 102,
	Parent = toggleBtn,
})

local toggleGlow = Create("UIStroke", {
	Color = Theme.Accent,
	Thickness = 2,
	Transparency = 0,
	ApplyStrokeMode = Enum.ApplyStrokeMode.Border,
	Parent = toggleBtn,
})

Create("TextLabel", {
	Name = "Icon",
	Size = UDim2.fromScale(1, 1),
	BackgroundTransparency = 1,
	Font = Theme.FontBold,
	Text = "B",
	TextSize = 24,
	TextColor3 = Theme.White,
	ZIndex = 101,
	Parent = toggleBtn,
})

task.spawn(function()
	local t = 0
	while toggleBtn.Parent do
		t += 0.05
		local s = (math.sin(t) + 1) / 2
		toggleGlow.Transparency = 0.3 + s * 0.5
		task.wait(0.05)
	end
end)

toggleBtnClick.MouseEnter:Connect(function()
	Tween(toggleBtn, 0.2, { Size = UDim2.fromOffset(58, 58) }, Enum.EasingStyle.Back, Enum.EasingDirection.Out)
	Tween(toggleGlow, 0.2, { Transparency = 0 })
end)
toggleBtnClick.MouseLeave:Connect(function()
	Tween(toggleBtn, 0.2, { Size = UDim2.fromOffset(52, 52) })
	Tween(toggleGlow, 0.2, { Transparency = 0.3 })
end)

local root = Create("CanvasGroup", {
	Name = "Root",
	Size = UDim2.fromOffset(640, 440),
	Position = UDim2.fromScale(0.5, 0.5),
	AnchorPoint = Vector2.new(0.5, 0.5),
	BackgroundTransparency = 1,
	GroupTransparency = 1,
	Visible = false,
	Parent = gui,
})
local rootScale = Create("UIScale", { Scale = 0.94, Parent = root })

local main = Create("Frame", {
	Name = "Main",
	Size = UDim2.fromScale(1, 1),
	BackgroundColor3 = Theme.Bg,
	BorderSizePixel = 0,
	ClipsDescendants = true,
	ZIndex = 1,
	Parent = root,
})
Round(main, 14)
Stroke(main, Theme.Stroke, 1, 0)
Gradient(main, Theme.BgGrad, Theme.Bg, 135)

local sidebar = Create("Frame", {
	Name = "Sidebar",
	Size = UDim2.new(0, 165, 1, 0),
	BackgroundColor3 = Theme.Sidebar,
	BorderSizePixel = 0,
	ZIndex = 2,
	Parent = main,
})
Round(sidebar, 14)

Create("Frame", {
	Size = UDim2.new(0, 14, 1, 0),
	Position = UDim2.new(1, -14, 0, 0),
	BackgroundColor3 = Theme.Sidebar,
	BorderSizePixel = 0,
	ZIndex = 2,
	Parent = sidebar,
})

Create("Frame", {
	Size = UDim2.new(0, 1, 1, -20),
	Position = UDim2.new(1, -1, 0, 10),
	BackgroundColor3 = Theme.Stroke,
	BackgroundTransparency = 0.5,
	BorderSizePixel = 0,
	ZIndex = 3,
	Parent = sidebar,
})

local logoArea = Create("Frame", {
	Name = "Logo",
	Size = UDim2.new(1, 0, 0, 64),
	BackgroundTransparency = 1,
	ZIndex = 3,
	Parent = sidebar,
})

local logoBox = Create("Frame", {
	Size = UDim2.fromOffset(30, 30),
	Position = UDim2.new(0, 16, 0.5, 0),
	AnchorPoint = Vector2.new(0, 0.5),
	BackgroundColor3 = Theme.Accent,
	BorderSizePixel = 0,
	ZIndex = 4,
	Parent = logoArea,
})
Round(logoBox, 9)
Gradient(logoBox, Theme.Accent, Theme.Pink, 45)

Create("TextLabel", {
	Size = UDim2.fromScale(1, 1),
	BackgroundTransparency = 1,
	Font = Theme.FontBold,
	Text = "B",
	TextSize = 17,
	TextColor3 = Theme.White,
	ZIndex = 5,
	Parent = logoBox,
})

Create("TextLabel", {
	Size = UDim2.new(1, -70, 1, 0),
	Position = UDim2.new(0, 56, 0, 0),
	BackgroundTransparency = 1,
	Font = Theme.FontBold,
	Text = "BERRETA",
	TextSize = 16,
	TextColor3 = Theme.Text,
	TextXAlignment = Enum.TextXAlignment.Left,
	ZIndex = 4,
	Parent = logoArea,
})

Create("TextLabel", {
	Size = UDim2.new(1, -70, 0, 12),
	Position = UDim2.new(0, 56, 0, 40),
	BackgroundTransparency = 1,
	Font = Theme.Font,
	Text = "The best for mvs.",
	TextSize = 10,
	TextColor3 = Theme.SubText,
	TextXAlignment = Enum.TextXAlignment.Left,
	ZIndex = 4,
	Parent = logoArea,
})

local tabHolder = Create("Frame", {
	Name = "Tabs",
	Size = UDim2.new(1, 0, 1, -64),
	Position = UDim2.new(0, 0, 0, 64),
	BackgroundTransparency = 1,
	ZIndex = 3,
	Parent = sidebar,
})
List(tabHolder, 4)
Pad(tabHolder, 10, 0)

local content = Create("Frame", {
	Name = "Content",
	Size = UDim2.new(1, -165, 1, 0),
	Position = UDim2.new(0, 165, 0, 0),
	BackgroundTransparency = 1,
	ZIndex = 2,
	Parent = main,
})

local topbar = Create("Frame", {
	Name = "Topbar",
	Size = UDim2.new(1, 0, 0, 54),
	BackgroundTransparency = 1,
	ZIndex = 3,
	Parent = content,
})
Create("Frame", {
	Size = UDim2.new(1, -20, 0, 1),
	Position = UDim2.new(0, 10, 1, -1),
	BackgroundColor3 = Theme.Stroke,
	BackgroundTransparency = 0.5,
	BorderSizePixel = 0,
	ZIndex = 3,
	Parent = topbar,
})

local pageTitle = Create("TextLabel", {
	Size = UDim2.new(1, -140, 1, 0),
	Position = UDim2.new(0, 20, 0.5, 0),
	AnchorPoint = Vector2.new(0, 0.5),
	BackgroundTransparency = 1,
	Font = Theme.FontBold,
	Text = "Главная",
	TextSize = 16,
	TextColor3 = Theme.Text,
	TextXAlignment = Enum.TextXAlignment.Left,
	ZIndex = 3,
	Parent = topbar,
})

local function ctrlBtn(text, xOff, hoverColor)
	local b = Create("TextButton", {
		Size = UDim2.fromOffset(28, 28),
		Position = UDim2.new(1, xOff, 0.5, 0),
		AnchorPoint = Vector2.new(1, 0.5),
		BackgroundColor3 = Theme.Element,
		BackgroundTransparency = 1,
		BorderSizePixel = 0,
		Text = text,
		Font = Theme.FontBold,
		TextSize = 13,
		TextColor3 = Theme.SubText,
		AutoButtonColor = false,
		ZIndex = 4,
		Parent = topbar,
	})
	Round(b, 7)
	b.MouseEnter:Connect(function()
		Tween(b, 0.15, { BackgroundTransparency = 0, TextColor3 = hoverColor or Theme.Text })
	end)
	b.MouseLeave:Connect(function()
		Tween(b, 0.15, { BackgroundTransparency = 1, TextColor3 = Theme.SubText })
	end)
	return b
end

local closeBtn = ctrlBtn("✕", -18, Color3.fromRGB(248, 113, 113))
local minBtn   = ctrlBtn("—", -52)

local pages = Create("Frame", {
	Name = "Pages",
	Size = UDim2.new(1, 0, 1, -54),
	Position = UDim2.new(0, 0, 0, 54),
	BackgroundTransparency = 1,
	ZIndex = 3,
	Parent = content,
})

local notifHolder = Create("Frame", {
	Name = "Notifications",
	Size = UDim2.new(0, 290, 1, -40),
	Position = UDim2.new(1, -20, 0, 20),
	AnchorPoint = Vector2.new(1, 0),
	BackgroundTransparency = 1,
	ZIndex = 50,
	Parent = gui,
})
List(notifHolder, 8)

function Berreta:Notify(title, text, duration)
	duration = duration or 3
	local wrap = Create("Frame", {
		Size = UDim2.new(1, 0, 0, 62),
		BackgroundTransparency = 1,
		ZIndex = 50,
		Parent = notifHolder,
	})
	local card = Create("Frame", {
		Size = UDim2.fromScale(1, 1),
		Position = UDim2.new(1, 70, 0, 0),
		BackgroundColor3 = Theme.Element,
		BorderSizePixel = 0,
		ZIndex = 50,
		Parent = wrap,
	})
	Round(card, 10)
	Stroke(card, Theme.StrokeHl, 1, 0.3)
	Gradient(card, Theme.ElementGrad, Theme.Element, 135)

	local bar = Create("Frame", {
		Size = UDim2.new(0, 3, 1, -20),
		Position = UDim2.new(0, 8, 0.5, 0),
		AnchorPoint = Vector2.new(0, 0.5),
		BackgroundColor3 = Theme.Accent,
		BorderSizePixel = 0,
		ZIndex = 51,
		Parent = card,
	})
	Round(bar, 2)
	Gradient(bar, Theme.Accent, Theme.Pink, 90)

	Create("TextLabel", {
		Size = UDim2.new(1, -38, 0, 16),
		Position = UDim2.new(0, 22, 0, 12),
		BackgroundTransparency = 1,
		Font = Theme.FontBold,
		Text = title or "Berreta",
		TextSize = 13,
		TextColor3 = Theme.Text,
		TextXAlignment = Enum.TextXAlignment.Left,
		ZIndex = 51,
		Parent = card,
	})
	Create("TextLabel", {
		Size = UDim2.new(1, -38, 0, 16),
		Position = UDim2.new(0, 22, 0, 32),
		BackgroundTransparency = 1,
		Font = Theme.Font,
		Text = text or "",
		TextSize = 12,
		TextColor3 = Theme.SubText,
		TextXAlignment = Enum.TextXAlignment.Left,
		TextTruncate = Enum.TextTruncate.AtEnd,
		ZIndex = 51,
		Parent = card,
	})

	Tween(card, 0.35, { Position = UDim2.new(0, 0, 0, 0) }, Enum.EasingStyle.Quint)
	task.delay(duration, function()
		if not card.Parent then return end
		Tween(card, 0.25, { Position = UDim2.new(1, 70, 0, 0) }, Enum.EasingStyle.Quint, Enum.EasingDirection.In)
		task.wait(0.28)
		wrap:Destroy()
	end)
end

local opened = false

function Berreta:SetOpen(state)
	opened = state
	if state then
		root.Visible = true
		Tween(root, 0.3, { GroupTransparency = 0 })
		Tween(rootScale, 0.35, { Scale = 1 }, Enum.EasingStyle.Back, Enum.EasingDirection.Out)
		Tween(blur, 0.3, { Size = 18 })
	else
		Tween(root, 0.22, { GroupTransparency = 1 })
		Tween(rootScale, 0.28, { Scale = 0.92 })
		Tween(blur, 0.25, { Size = 0 })
		task.delay(0.3, function()
			if not opened then root.Visible = false end
		end)
	end
end

function Berreta:Toggle()
	self:SetOpen(not opened)
end

toggleBtnClick.MouseButton1Click:Connect(function() Berreta:Toggle() end)
minBtn.MouseButton1Click:Connect(function() Berreta:SetOpen(false) end)
closeBtn.MouseButton1Click:Connect(function()
	opened = false
	if blur then blur:Destroy() end
	gui:Destroy()
end)

local _toggleHeld = false
UIS.InputBegan:Connect(function(input, gpe)
	if gpe then return end
	if input.KeyCode == Berreta.ToggleKey then
		if not _toggleHeld then
			_toggleHeld = true
			Berreta:Toggle()
		end
	end
end)
UIS.InputEnded:Connect(function(input, gpe)
	if gpe then return end
	if input.KeyCode == Berreta.ToggleKey then
		_toggleHeld = false
	end
end)

do
	local dragging, dragInput, dragStart, startPos
	toggleBtnClick.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1
		or input.UserInputType == Enum.UserInputType.Touch then
			dragging  = true
			dragStart = input.Position
			startPos  = toggleBtn.Position
		end
	end)
	toggleBtnClick.InputEnded:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1
		or input.UserInputType == Enum.UserInputType.Touch then
			dragging = false
		end
	end)
	UIS.InputChanged:Connect(function(input)
		if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement
		or input.UserInputType == Enum.UserInputType.Touch) then
			local d = input.Position - dragStart
			toggleBtn.Position = UDim2.new(
				startPos.X.Scale, startPos.X.Offset + d.X,
				startPos.Y.Scale, startPos.Y.Offset + d.Y
			)
		end
	end)
end

local Tab = {}
Tab.__index = Tab

function Berreta:SelectTab(target)
	self.CurrentTab = target
	for _, t in ipairs(self.Tabs) do
		local active = (t == target)
		t.Page.Visible = active
		Tween(t.TabButton, 0.2, { BackgroundTransparency = active and 0 or 1 })
		Tween(t.TabLabel, 0.2, { TextColor3 = active and Theme.Text or Theme.SubText })
		Tween(t.TabIcon, 0.2, { TextColor3 = active and Theme.White or Theme.SubText })
		Tween(t.TabIndicator, 0.2, { Size = active and UDim2.fromOffset(3, 20) or UDim2.fromOffset(0, 0) })
		if t.TabGradient then t.TabGradient.Enabled = active end
	end
	pageTitle.Text = target.Name
end

function Berreta:Tab(name, icon)
	local btn = Create("TextButton", {
		Name = name,
		Size = UDim2.new(1, -20, 0, 40),
		BackgroundColor3 = Theme.Accent,
		BackgroundTransparency = 1,
		BorderSizePixel = 0,
		Text = "",
		AutoButtonColor = false,
		ZIndex = 3,
		Parent = tabHolder,
	})
	Round(btn, 10)
	local btnGradient = Gradient(btn, Theme.Accent, Theme.AccentDeep, 90)
	btnGradient.Enabled = false

	local btnStroke = Stroke(btn, Theme.Accent, 1, 1)

	local indicator = Create("Frame", {
		Size = UDim2.fromOffset(0, 0),
		Position = UDim2.new(0, 0, 0.5, 0),
		AnchorPoint = Vector2.new(0, 0.5),
		BackgroundColor3 = Theme.Accent,
		BorderSizePixel = 0,
		ZIndex = 5,
		Parent = btn,
	})
	Round(indicator, 2)

	local iconLbl = Create("TextLabel", {
		Size = UDim2.fromOffset(22, 22),
		Position = UDim2.new(0, 14, 0.5, 0),
		AnchorPoint = Vector2.new(0, 0.5),
		BackgroundTransparency = 1,
		Font = Theme.FontMed,
		Text = icon or "📄",
		TextSize = 15,
		TextColor3 = Theme.SubText,
		ZIndex = 4,
		Parent = btn,
	})

	local label = Create("TextLabel", {
		Size = UDim2.new(1, -44, 1, 0),
		Position = UDim2.new(0, 40, 0, 0),
		BackgroundTransparency = 1,
		Font = Theme.FontMed,
		Text = name,
		TextSize = 13,
		TextColor3 = Theme.SubText,
		TextXAlignment = Enum.TextXAlignment.Left,
		ZIndex = 4,
		Parent = btn,
	})

	local page = Create("ScrollingFrame", {
		Name = name,
		Size = UDim2.fromScale(1, 1),
		BackgroundTransparency = 1,
		BorderSizePixel = 0,
		ScrollBarThickness = 4,
		ScrollBarImageColor3 = Theme.Accent,
		ScrollBarImageTransparency = 0.3,
		CanvasSize = UDim2.new(),
		AutomaticCanvasSize = Enum.AutomaticSize.Y,
		ScrollingDirection = Enum.ScrollingDirection.Y,
		Visible = false,
		ZIndex = 3,
		Parent = pages,
	})
	List(page, 8)
	Pad(page, 16, 14)

	local tab = setmetatable({
		Page = page,
		TabButton = btn,
		TabLabel = label,
		TabIcon = iconLbl,
		TabIndicator = indicator,
		TabGradient = btnGradient,
		TabStroke = btnStroke,
		Name = name,
		Window = self,
	}, Tab)

	table.insert(self.Tabs, tab)

	btn.MouseEnter:Connect(function()
		if Berreta.CurrentTab ~= tab then
			Tween(btn, 0.2, { BackgroundTransparency = 0.5 })
			btnGradient.Enabled = true
			btnGradient.Color = ColorSequence.new(Theme.SidebarHl, Theme.SidebarHl)
			Tween(btnStroke, 0.2, { Transparency = 0.7 })
		end
	end)
	btn.MouseLeave:Connect(function()
		if Berreta.CurrentTab ~= tab then
			Tween(btn, 0.2, { BackgroundTransparency = 1 })
			btnGradient.Enabled = false
			Tween(btnStroke, 0.2, { Transparency = 1 })
		end
	end)
	btn.MouseButton1Click:Connect(function()
		Berreta:SelectTab(tab)
		btnGradient.Color = ColorSequence.new(Theme.Accent, Theme.AccentDeep)
		Tween(btnStroke, 0.2, { Transparency = 0 })
	end)

	if #self.Tabs == 1 then
		self:SelectTab(tab)
		btnGradient.Color = ColorSequence.new(Theme.Accent, Theme.AccentDeep)
		Tween(btnStroke, 0.2, { Transparency = 0 })
	end
	return tab
end

function Tab:Section(text)
	local holder = Create("Frame", {
		Size = UDim2.new(1, -32, 0, 28),
		BackgroundTransparency = 1,
		Parent = self.Page,
	})
	Create("TextLabel", {
		Size = UDim2.new(1, 0, 0, 16),
		Position = UDim2.new(0, 0, 0, 6),
		BackgroundTransparency = 1,
		Font = Theme.FontBold,
		Text = string.upper(text),
		TextSize = 11,
		TextColor3 = Theme.Accent,
		TextXAlignment = Enum.TextXAlignment.Left,
		Parent = holder,
	})
	return holder
end

function Tab:Label(text, color)
	return Create("TextLabel", {
		Size = UDim2.new(1, -32, 0, 18),
		BackgroundTransparency = 1,
		Font = Theme.Font,
		Text = text,
		TextSize = 12,
		TextColor3 = color or Theme.SubText,
		TextXAlignment = Enum.TextXAlignment.Left,
		TextWrapped = true,
		Parent = self.Page,
	})
end

function Tab:Button(name, callback, textColor)
	local btn = Create("TextButton", {
		Name = name,
		Size = UDim2.new(1, -32, 0, 38),
		BackgroundColor3 = Theme.Element,
		BorderSizePixel = 0,
		Text = name,
		Font = Theme.FontMed,
		TextSize = 13,
		TextColor3 = textColor or Theme.Text,
		AutoButtonColor = false,
		Parent = self.Page,
	})
	Round(btn, 10)
	local btnStroke = Stroke(btn, Theme.Stroke, 1, 0.5)

	btn.MouseEnter:Connect(function()
		Tween(btn, 0.15, { BackgroundColor3 = Theme.ElementHl })
		Tween(btnStroke, 0.15, { Color = Theme.Accent, Transparency = 0.3 })
	end)
	btn.MouseLeave:Connect(function()
		Tween(btn, 0.15, { BackgroundColor3 = Theme.Element })
		Tween(btnStroke, 0.15, { Color = Theme.Stroke, Transparency = 0.5 })
	end)
	btn.MouseButton1Click:Connect(function()
		Tween(btn, 0.08, { BackgroundColor3 = Theme.Accent })
		task.delay(0.1, function()
			if btn.Parent then Tween(btn, 0.25, { BackgroundColor3 = Theme.ElementHl }) end
		end)
		if callback then task.spawn(callback) end
	end)
	return btn
end

function Tab:ButtonRow(buttons)
	local holder = Create("Frame", {
		Size = UDim2.new(1, -32, 0, 36),
		BackgroundTransparency = 1,
		Parent = self.Page,
	})
	Create("UIListLayout", {
		Padding = UDim.new(0, 8),
		SortOrder = Enum.SortOrder.LayoutOrder,
		FillDirection = Enum.FillDirection.Horizontal,
		HorizontalAlignment = Enum.HorizontalAlignment.Left,
		Parent = holder,
	})
	for _, data in ipairs(buttons) do
		local btn = Create("TextButton", {
			Name = data.name,
			Size = UDim2.new(0.25, -6, 1, 0),
			BackgroundColor3 = Theme.Element,
			BorderSizePixel = 0,
			Text = data.name,
			Font = Theme.FontMed,
			TextSize = 12,
			TextColor3 = Theme.Text,
			AutoButtonColor = false,
			Parent = holder,
		})
		Round(btn, 8)
		Stroke(btn, Theme.Stroke, 1, 0.5)
		btn.MouseEnter:Connect(function() Tween(btn, 0.15, { BackgroundColor3 = Theme.ElementHl }) end)
		btn.MouseLeave:Connect(function() Tween(btn, 0.15, { BackgroundColor3 = Theme.Element }) end)
		btn.MouseButton1Click:Connect(function()
			if data.callback then task.spawn(data.callback) end
		end)
	end
	return holder
end

function Tab:Toggle(name, default, callback)
	local state = default and true or false
	local holder = Create("Frame", {
		Name = name,
		Size = UDim2.new(1, -32, 0, 44),
		BackgroundColor3 = Theme.Element,
		BorderSizePixel = 0,
		Parent = self.Page,
	})
	Round(holder, 10)
	Stroke(holder, Theme.Stroke, 1, 0.6)

	Create("TextLabel", {
		Size = UDim2.new(1, -80, 1, 0),
		Position = UDim2.new(0, 16, 0, 0),
		BackgroundTransparency = 1,
		Font = Theme.Font,
		Text = name,
		TextSize = 13,
		TextColor3 = Theme.Text,
		TextXAlignment = Enum.TextXAlignment.Left,
		Parent = holder,
	})

	local track = Create("TextButton", {
		Size = UDim2.fromOffset(44, 24),
		Position = UDim2.new(1, -58, 0.5, 0),
		AnchorPoint = Vector2.new(0, 0.5),
		BackgroundColor3 = state and Theme.Accent or Theme.ElementHl,
		BorderSizePixel = 0,
		Text = "",
		AutoButtonColor = false,
		Parent = holder,
	})
	Round(track, 12)
	local trackGradient = Gradient(track, Theme.Accent, Theme.Pink, 90)
	trackGradient.Enabled = state

	local knob = Create("Frame", {
		Size = UDim2.fromOffset(18, 18),
		Position = UDim2.new(0, state and 23 or 3, 0.5, 0),
		AnchorPoint = Vector2.new(0, 0.5),
		BackgroundColor3 = Theme.White,
		BorderSizePixel = 0,
		Parent = track,
	})
	Round(knob, 9)

	local function set(value, fire)
		state = value and true or false
		Tween(track, 0.2, { BackgroundColor3 = state and Theme.Accent or Theme.ElementHl })
		Tween(knob, 0.2, { Position = UDim2.new(0, state and 23 or 3, 0.5, 0) }, Enum.EasingStyle.Back)
		trackGradient.Enabled = state
		if fire and callback then task.spawn(callback, state) end
	end

	track.MouseButton1Click:Connect(function() set(not state, true) end)

	local api = {
		Set = function(v) set(v, true) end,
		Get = function() return state end,
		Toggle = function() set(not state, true) end,
	}
	Berreta.Flags[name] = api
	return api
end

function Tab:Slider(name, min, max, default, callback, step)
	min = min or 0; max = max or 100; step = step or 1
	default = math.clamp(default or min, min, max)

	local holder = Create("Frame", {
		Name = name,
		Size = UDim2.new(1, -32, 0, 58),
		BackgroundColor3 = Theme.Element,
		BorderSizePixel = 0,
		Parent = self.Page,
	})
	Round(holder, 10)
	Stroke(holder, Theme.Stroke, 1, 0.6)

	Create("TextLabel", {
		Size = UDim2.new(1, -80, 0, 16),
		Position = UDim2.new(0, 16, 0, 10),
		BackgroundTransparency = 1,
		Font = Theme.Font,
		Text = name,
		TextSize = 13,
		TextColor3 = Theme.Text,
		TextXAlignment = Enum.TextXAlignment.Left,
		Parent = holder,
	})

	local valueLabel = Create("TextLabel", {
		Size = UDim2.new(0, 70, 0, 16),
		Position = UDim2.new(1, -86, 0, 10),
		BackgroundTransparency = 1,
		Font = Theme.FontMed,
		Text = tostring(default),
		TextSize = 12,
		TextColor3 = Theme.Accent,
		TextXAlignment = Enum.TextXAlignment.Right,
		Parent = holder,
	})

	local track = Create("Frame", {
		Size = UDim2.new(1, -32, 0, 6),
		Position = UDim2.new(0, 16, 0, 40),
		BackgroundColor3 = Theme.ElementHl,
		BorderSizePixel = 0,
		Parent = holder,
	})
	Round(track, 3)

	local fill = Create("Frame", {
		Size = UDim2.new(0, 0, 1, 0),
		BackgroundColor3 = Theme.Accent,
		BorderSizePixel = 0,
		Parent = track,
	})
	Round(fill, 3)
	Gradient(fill, Theme.Accent, Theme.Pink, 0)

	local knob = Create("Frame", {
		Size = UDim2.fromOffset(16, 16),
		Position = UDim2.new(0, 0, 0.5, 0),
		AnchorPoint = Vector2.new(0.5, 0.5),
		BackgroundColor3 = Theme.White,
		BorderSizePixel = 0,
		ZIndex = 2,
		Parent = track,
	})
	Round(knob, 8)
	Stroke(knob, Theme.Accent, 2, 0)

	local function setValue(v, fire)
		v = math.clamp(v, min, max)
		if step >= 1 then v = math.floor(v / step + 0.5) * step
		else v = math.floor(v * 100 + 0.5) / 100 end
		v = math.clamp(v, min, max)

		local alpha = (max - min) > 0 and (v - min) / (max - min) or 0
		fill.Size = UDim2.new(alpha, 0, 1, 0)
		knob.Position = UDim2.new(alpha, 0, 0.5, 0)
		valueLabel.Text = (step >= 1) and tostring(math.floor(v)) or string.format("%.2f", v)

		if fire and callback then task.spawn(callback, v) end
		return v
	end

	local dragging = false
	local function process(input)
		local rel = (input.Position.X - track.AbsolutePosition.X) / track.AbsoluteSize.X
		rel = math.clamp(rel, 0, 1)
		setValue(min + (max - min) * rel, true)
	end

	track.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			dragging = true; process(input)
			Tween(knob, 0.1, { Size = UDim2.fromOffset(20, 20) })
		end
	end)
	UIS.InputChanged:Connect(function(input)
		if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
			process(input)
		end
	end)
	UIS.InputEnded:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			dragging = false
			Tween(knob, 0.15, { Size = UDim2.fromOffset(16, 16) })
		end
	end)

	setValue(default, false)
	local api = {
		Set = function(v) setValue(v, true) end,
		Get = function() return tonumber(valueLabel.Text) end,
	}
	Berreta.Flags[name] = api
	return api
end

function Tab:Dropdown(name, options, default, callback)
	options = options or {}
	local selected = default or options[1]

	local holder = Create("Frame", {
		Name = name,
		Size = UDim2.new(1, -32, 0, 40),
		BackgroundColor3 = Theme.Element,
		BorderSizePixel = 0,
		ClipsDescendants = false,
		ZIndex = 2,
		Parent = self.Page,
	})
	Round(holder, 10)
	Stroke(holder, Theme.Stroke, 1, 0.6)

	Create("TextLabel", {
		Size = UDim2.new(0.5, -16, 1, 0),
		Position = UDim2.new(0, 16, 0, 0),
		BackgroundTransparency = 1,
		Font = Theme.Font,
		Text = name,
		TextSize = 13,
		TextColor3 = Theme.Text,
		TextXAlignment = Enum.TextXAlignment.Left,
		ZIndex = 3,
		Parent = holder,
	})

	local btn = Create("TextButton", {
		Size = UDim2.new(0.5, -26, 1, -14),
		Position = UDim2.new(0.5, 13, 0, 7),
		BackgroundColor3 = Theme.ElementHl,
		BorderSizePixel = 0,
		Text = "",
		AutoButtonColor = false,
		ZIndex = 3,
		Parent = holder,
	})
	Round(btn, 7)
	Stroke(btn, Theme.Stroke, 1, 0.5)

	local valueLabel = Create("TextLabel", {
		Size = UDim2.new(1, -28, 1, 0),
		Position = UDim2.new(0, 12, 0, 0),
		BackgroundTransparency = 1,
		Font = Theme.FontMed,
		Text = tostring(selected),
		TextSize = 12,
		TextColor3 = Theme.Accent,
		TextXAlignment = Enum.TextXAlignment.Left,
		TextTruncate = Enum.TextTruncate.AtEnd,
		ZIndex = 4,
		Parent = btn,
	})
	Create("TextLabel", {
		Size = UDim2.new(0, 16, 1, 0),
		Position = UDim2.new(1, -18, 0, 0),
		BackgroundTransparency = 1,
		Font = Theme.FontBold,
		Text = "▾",
		TextSize = 12,
		TextColor3 = Theme.SubText,
		ZIndex = 4,
		Parent = btn,
	})

	local list = Create("Frame", {
		Size = UDim2.new(1, 0, 0, 0),
		Position = UDim2.new(0, 0, 1, 6),
		BackgroundColor3 = Theme.Element,
		BorderSizePixel = 0,
		ClipsDescendants = true,
		Visible = false,
		ZIndex = 20,
		Parent = holder,
	})
	Round(list, 10)
	Stroke(list, Theme.StrokeHl, 1, 0.2)
	List(list, 2)
	Pad(list, 4, 4)

	local isOpen = false

	local function close()
		isOpen = false
		Tween(list, 0.15, { Size = UDim2.new(1, 0, 0, 0) })
		task.delay(0.15, function()
			if not isOpen then list.Visible = false end
		end)
	end

	local function open()
		isOpen = true
		list.Visible = true
		Tween(list, 0.15, { Size = UDim2.new(1, 0, 0, #options * 28 + 8) })
	end

	for _, opt in ipairs(options) do
		local ob = Create("TextButton", {
			Size = UDim2.new(1, 0, 0, 26),
			BackgroundColor3 = Theme.Element,
			BackgroundTransparency = 1,
			BorderSizePixel = 0,
			Text = tostring(opt),
			Font = Theme.Font,
			TextSize = 12,
			TextColor3 = Theme.Text,
			TextXAlignment = Enum.TextXAlignment.Left,
			AutoButtonColor = false,
			ZIndex = 21,
			Parent = list,
		})
		Round(ob, 6)
		Create("UIPadding", { PaddingLeft = UDim.new(0, 10), Parent = ob })

		ob.MouseEnter:Connect(function()
			Tween(ob, 0.12, { BackgroundTransparency = 0, BackgroundColor3 = Theme.Accent })
		end)
		ob.MouseLeave:Connect(function()
			Tween(ob, 0.12, { BackgroundTransparency = 1 })
		end)
		ob.MouseButton1Click:Connect(function()
			selected = opt
			valueLabel.Text = tostring(opt)
			close()
			if callback then task.spawn(callback, opt) end
		end)
	end

	btn.MouseButton1Click:Connect(function()
		if isOpen then close() else open() end
	end)

	local api = {
		Set = function(v)
			selected = v
			valueLabel.Text = tostring(v)
			if callback then task.spawn(callback, v) end
		end,
		Get = function() return selected end,
	}
	Berreta.Flags[name] = api
	return api
end

--=====================================================================
--  СИСТЕМА КОНФИГОВ
--=====================================================================
Berreta.ConfigFolder = "Berreta/Configs"
Berreta.ConfigPrefix = "BRT1:"

local function encodeString(str)
	local ok, res = pcall(function() return HttpService:Base64Encode(str) end)
	if ok and res then return res end
	return str
end

local function decodeString(str)
	local ok, res = pcall(function() return HttpService:Base64Decode(str) end)
	if ok and res then return res end
	return str
end

function Berreta:BuildConfigData()
	local data = { ToggleKey = Berreta.ToggleKey.Name, Flags = {}, Version = "1.0" }
	for k, api in pairs(Berreta.Flags) do
		if type(api) == "table" and api.Get then
			local ok, v = pcall(api.Get)
			if ok and (type(v) == "number" or type(v) == "string" or type(v) == "boolean") then
				data.Flags[k] = v
			end
		end
	end
	return data
end

function Berreta:ApplyConfigData(data)
	if type(data) ~= "table" then return false, "Неверные данные" end
	if data.Flags then
		for k, v in pairs(data.Flags) do
			local api = Berreta.Flags[k]
			if api and api.Set then pcall(api.Set, v) end
		end
	end
	if data.ToggleKey then
		local keyCode = Enum.KeyCode[data.ToggleKey]
		if keyCode then Berreta.ToggleKey = keyCode end
	end
	return true
end

function Berreta:SaveConfig(name)
	name = tostring(name):gsub("[^%w_%-]", "")
	if name == "" then return false, "Пустое имя" end
	local data = self:BuildConfigData()
	local ok, err = pcall(function()
		if not isfolder("Berreta") then makefolder("Berreta") end
		if not isfolder(self.ConfigFolder) then makefolder(self.ConfigFolder) end
		writefile(self.ConfigFolder .. "/" .. name .. ".json", HttpService:JSONEncode(data))
	end)
	if ok then return true else return false, err end
end

function Berreta:LoadConfig(name)
	name = tostring(name):gsub("[^%w_%-]", "")
	local path = self.ConfigFolder .. "/" .. name .. ".json"
	local ok, exists = pcall(isfile, path)
	if not ok or not exists then return false, "Файл не найден" end
	local readOk, content = pcall(readfile, path)
	if not readOk then return false, "Ошибка чтения" end
	local decodeOk, data = pcall(HttpService.JSONDecode, HttpService, content)
	if not decodeOk then return false, "Ошибка формата" end
	return self:ApplyConfigData(data)
end

function Berreta:DeleteConfig(name)
	name = tostring(name):gsub("[^%w_%-]", "")
	local path = self.ConfigFolder .. "/" .. name .. ".json"
	local ok = pcall(function() if isfile(path) then delfile(path) end end)
	return ok
end

function Berreta:GetConfigList()
	local list = {}
	local ok, files = pcall(function()
		if not isfolder(self.ConfigFolder) then return {} end
		return listfiles(self.ConfigFolder)
	end)
	if not ok or not files then return list end
	for _, f in ipairs(files) do
		if f:sub(-5) == ".json" then
			local name = f:match("([^/\\]+)%.json$")
			if name then table.insert(list, name) end
		end
	end
	table.sort(list)
	return list
end

function Berreta:ExportString()
	local data = self:BuildConfigData()
	local json = HttpService:JSONEncode(data)
	return self.ConfigPrefix .. encodeString(json)
end

function Berreta:ImportString(str)
	if type(str) ~= "string" then return false, "Пустая строка" end
	str = str:gsub("%s+", "")
	if str:sub(1, #self.ConfigPrefix) == self.ConfigPrefix then
		str = str:sub(#self.ConfigPrefix + 1)
	end
	local decoded = decodeString(str)
	local ok, data = pcall(HttpService.JSONDecode, HttpService, decoded)
	if not ok or type(data) ~= "table" then return false, "Неверный формат" end
	return self:ApplyConfigData(data)
end

function Berreta:ExportFile(name)
	name = tostring(name):gsub("[^%w_%-]", "")
	local path = self.ConfigFolder .. "/" .. name .. ".json"
	local ok, exists = pcall(isfile, path)
	if not ok or not exists then return false, "Файл не найден" end
	local readOk, content = pcall(readfile, path)
	if not readOk then return false, "Ошибка чтения" end
	return self.ConfigPrefix .. encodeString(content)
end

--=====================================================================
--  ЗАПУСК
--=====================================================================
Draggable(root, logoArea)
Draggable(root, topbar)

task.wait(0.15)
Berreta:SetOpen(false)
task.wait(0.3)
Berreta:Notify("Berreta", "Нажми 'B' слева или " .. Berreta.ToggleKey.Name, 4)

local MainTab   = Berreta:Tab("Главная", "🔒")
local PlayerTab = Berreta:Tab("Игрок", "👤")
local VisualTab = Berreta:Tab("Визуал", "✨")
local CombatTab = Berreta:Tab("Combat", "⚔️")
local ConfigTab = Berreta:Tab("Конфиги", "💾")
local MiscTab   = Berreta:Tab("Разное", "⚙️")

MainTab:Section("Добро пожаловать")
MainTab:Label("Berreta — The best for mvs.", Theme.Accent)
MainTab:Label("Используй вкладки слева для навигации.")

--============================ ИГРОК ==================================
local _ws = 16
local _jp = 50

local function applyMovement()
	local char = LP.Character
	if not char then return end
	local hum = char:FindFirstChildOfClass("Humanoid")
	if hum then
		hum.WalkSpeed = _ws
		hum.UseJumpPower = true
		hum.JumpPower = _jp
	end
end

PlayerTab:Section("Скорость")
PlayerTab:Slider("WalkSpeed", 0, 200, 16, function(value)
	_ws = value
	local hum = LP.Character and LP.Character:FindFirstChildOfClass("Humanoid")
	if hum then hum.WalkSpeed = value end
end)

PlayerTab:Section("Прыжок")
PlayerTab:Slider("JumpPower", 0, 300, 50, function(value)
	_jp = value
	local hum = LP.Character and LP.Character:FindFirstChildOfClass("Humanoid")
	if hum then
		hum.UseJumpPower = true
		hum.JumpPower = value
	end
end)

PlayerTab:Section("Функции")

local infJumpConn
PlayerTab:Toggle("Infinite Jump", false, function(state)
	if state then
		infJumpConn = UIS.JumpRequest:Connect(function()
			local char = LP.Character
			if char then
				local hum = char:FindFirstChildOfClass("Humanoid")
				if hum then hum.Jump = true end
			end
		end)
	else
		if infJumpConn then infJumpConn:Disconnect(); infJumpConn = nil end
	end
end)

local noclipConn
PlayerTab:Toggle("Noclip", false, function(state)
	if state then
		noclipConn = RunService.Stepped:Connect(function()
			local char = LP.Character
			if char then
				for _, part in ipairs(char:GetDescendants()) do
					if part:IsA("BasePart") then part.CanCollide = false end
				end
			end
		end)
	else
		if noclipConn then noclipConn:Disconnect(); noclipConn = nil end
	end
end)

LP.CharacterAdded:Connect(function()
	task.wait(0.5)
	applyMovement()
end)
if LP.Character then applyMovement() end

PlayerTab:Section("Спид Глич")
local speedGlitchConn = nil
local glitchPower = 60
local glitchUpward = 20
local glitchDuration = 0.25

PlayerTab:Slider("Glitch Power", 20, 200, 60, function(v) glitchPower = v end)
PlayerTab:Slider("Glitch Upward", 0, 80, 20, function(v) glitchUpward = v end)
PlayerTab:Slider("Glitch Duration (ms)", 50, 500, 250, function(v) glitchDuration = v / 1000 end)

local function getMoveDirection()
	local cam = workspace.CurrentCamera
	if not cam then return Vector3.new(0, 0, -1) end
	local forward = Vector3.new(0, 0, 0)
	local right = Vector3.new(0, 0, 0)
	if UIS:IsKeyDown(Enum.KeyCode.W) then forward = forward + cam.CFrame.LookVector end
	if UIS:IsKeyDown(Enum.KeyCode.S) then forward = forward - cam.CFrame.LookVector end
	if UIS:IsKeyDown(Enum.KeyCode.A) then right = right - cam.CFrame.RightVector end
	if UIS:IsKeyDown(Enum.KeyCode.D) then right = right + cam.CFrame.RightVector end
	local dir = forward + right
	if dir.Magnitude < 0.01 then return cam.CFrame.LookVector end
	dir = Vector3.new(dir.X, 0, dir.Z)
	return dir.Unit
end

PlayerTab:Toggle("Speed Glitch", false, function(state)
	if state then
		speedGlitchConn = UIS.JumpRequest:Connect(function()
			local char = LP.Character
			if not char then return end
			local hrp = char:FindFirstChild("HumanoidRootPart")
			if not hrp then return end
			local old = hrp:FindFirstChild("BerretaSpeedGlitch")
			if old then old:Destroy() end
			local dir = getMoveDirection()
			local bv = Instance.new("BodyVelocity")
			bv.Name = "BerretaSpeedGlitch"
			bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
			bv.P = 10000
			bv.Velocity = dir * glitchPower + Vector3.new(0, glitchUpward, 0)
			bv.Parent = hrp
			task.delay(glitchDuration, function()
				if bv and bv.Parent then bv:Destroy() end
			end)
		end)
		Berreta:Notify("Player", "Speed Glitch: ВКЛ", 2)
	else
		if speedGlitchConn then speedGlitchConn:Disconnect(); speedGlitchConn = nil end
		Berreta:Notify("Player", "Speed Glitch: ВЫКЛ", 2)
	end
end)

--============================ ВИЗУАЛ =================================
VisualTab:Section("Освещение")
VisualTab:Toggle("Fullbright", false, function(state)
	Lighting.Brightness = state and 3 or 1
	Lighting.Ambient = state and Color3.fromRGB(255, 255, 255) or Color3.fromRGB(0, 0, 0)
	Lighting.OutdoorAmbient = state and Color3.fromRGB(255, 255, 255) or Color3.fromRGB(0, 0, 0)
end)
VisualTab:Toggle("No Fog", false, function(state)
	if state then Lighting.FogStart = 100000; Lighting.FogEnd = 100000
	else Lighting.FogStart = 0; Lighting.FogEnd = 1000 end
end)

VisualTab:Section("Камера")
local cam = workspace.CurrentCamera
local baseFOV = 70
local function applyCamera()
	if cam then cam.FieldOfView = math.clamp(baseFOV, 1, 120) end
end
VisualTab:Slider("FOV", 70, 120, 70, function(v) baseFOV = v; applyCamera() end)
VisualTab:Button("Сбросить FOV", function()
	baseFOV = 70; Berreta.Flags["FOV"]:Set(70); applyCamera()
	Berreta:Notify("Камера", "FOV сброшен", 2)
end)

VisualTab:Section("Туман")
local fogColor = Color3.fromRGB(139, 92, 246)
local fogStart = 10
local fogEnd = 150
local fogEnabled = false
local function applyFog()
	if not fogEnabled then return end
	Lighting.FogColor = fogColor
	Lighting.FogStart = fogStart
	Lighting.FogEnd = fogEnd
end
VisualTab:Toggle("Custom Fog", false, function(state)
	fogEnabled = state
	if state then applyFog() else
		Lighting.FogStart = 0; Lighting.FogEnd = 1000
		Lighting.FogColor = Color3.fromRGB(192, 192, 192)
	end
end)
VisualTab:Dropdown("Fog Цвет", {"Purple","Red","Green","Blue","Yellow","Pink","Cyan","Orange","Gray","White","Black"}, "Purple", function(v) fogColor = ChamsColors[v] or ChamsColors.Purple; applyFog() end)
VisualTab:Slider("Fog Start", 0, 500, 10, function(v) fogStart = v; applyFog() end)
VisualTab:Slider("Fog End", 20, 2000, 150, function(v) fogEnd = v; applyFog() end)

VisualTab:Section("Курсор")
local cursorGui = Create("ScreenGui", { Name = "BerretaCursor", ResetOnSpawn = false, IgnoreGuiInset = true, DisplayOrder = 99999, ZIndexBehavior = Enum.ZIndexBehavior.Sibling, Parent = getParent() })
local cursorHolder = Create("Frame", { Name = "Cursor", Size = UDim2.fromOffset(24, 24), AnchorPoint = Vector2.new(0.5, 0.5), BackgroundTransparency = 1, ZIndex = 10, Visible = false, Parent = cursorGui })
local cursorState = { enabled = false, style = "Cross", color = ChamsColors.Purple, size = 24, transparency = 0 }

local function rebuildCursor()
	for _, c in ipairs(cursorHolder:GetChildren()) do c:Destroy() end
	cursorHolder.Size = UDim2.fromOffset(cursorState.size, cursorState.size)
	local col, trans = cursorState.color, cursorState.transparency
	if cursorState.style == "Dot" then
		local d = Create("Frame", { Size = UDim2.fromScale(0.45, 0.45), Position = UDim2.fromScale(0.5, 0.5), AnchorPoint = Vector2.new(0.5, 0.5), BackgroundColor3 = col, BackgroundTransparency = trans, BorderSizePixel = 0, Parent = cursorHolder })
		Round(d, 999)
	elseif cursorState.style == "Cross" then
		Create("Frame", { Size = UDim2.new(1, 0, 0, 2), Position = UDim2.fromScale(0.5, 0.5), AnchorPoint = Vector2.new(0.5, 0.5), BackgroundColor3 = col, BackgroundTransparency = trans, BorderSizePixel = 0, Parent = cursorHolder })
		Create("Frame", { Size = UDim2.new(0, 2, 1, 0), Position = UDim2.fromScale(0.5, 0.5), AnchorPoint = Vector2.new(0.5, 0.5), BackgroundColor3 = col, BackgroundTransparency = trans, BorderSizePixel = 0, Parent = cursorHolder })
	elseif cursorState.style == "Circle" then
		local c = Create("Frame", { Size = UDim2.fromScale(0.85, 0.85), Position = UDim2.fromScale(0.5, 0.5), AnchorPoint = Vector2.new(0.5, 0.5), BackgroundTransparency = 1, Parent = cursorHolder })
		Round(c, 999)
		Create("UIStroke", { Color = col, Thickness = 2, Transparency = trans, Parent = c })
	elseif cursorState.style == "Square" then
		local s = Create("Frame", { Size = UDim2.fromScale(0.85, 0.85), Position = UDim2.fromScale(0.5, 0.5), AnchorPoint = Vector2.new(0.5, 0.5), BackgroundTransparency = 1, Parent = cursorHolder })
		Round(s, 3)
		Create("UIStroke", { Color = col, Thickness = 2, Transparency = trans, Parent = s })
	elseif cursorState.style == "X" then
		Create("Frame", { Size = UDim2.new(1, 0, 0, 2), Position = UDim2.fromScale(0.5, 0.5), AnchorPoint = Vector2.new(0.5, 0.5), BackgroundColor3 = col, BackgroundTransparency = trans, BorderSizePixel = 0, Rotation = 45, Parent = cursorHolder })
		Create("Frame", { Size = UDim2.new(1, 0, 0, 2), Position = UDim2.fromScale(0.5, 0.5), AnchorPoint = Vector2.new(0.5, 0.5), BackgroundColor3 = col, BackgroundTransparency = trans, BorderSizePixel = 0, Rotation = -45, Parent = cursorHolder })
	elseif cursorState.style == "Arrow" then
		Create("TextLabel", { Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1, Font = Theme.FontBold, Text = "➤", TextSize = cursorState.size, TextColor3 = col, TextTransparency = trans, Parent = cursorHolder })
	end
end

RunService.RenderStepped:Connect(function()
	if cursorState.enabled then
		local pos = UIS:GetMouseLocation()
		cursorHolder.Position = UDim2.fromOffset(pos.X, pos.Y)
	end
end)

VisualTab:Toggle("Custom Cursor", false, function(state)
	cursorState.enabled = state
	cursorHolder.Visible = state
	UIS.MouseIconEnabled = not state
	if state then rebuildCursor() end
end)
VisualTab:Dropdown("Стиль курсора", {"Cross","Dot","Circle","Square","X","Arrow"}, "Cross", function(v) cursorState.style = v; rebuildCursor() end)
VisualTab:Dropdown("Цвет курсора", {"Purple","Red","Green","Blue","Yellow","Pink","Cyan","Orange","White","Black"}, "Purple", function(v) cursorState.color = ChamsColors[v] or ChamsColors.Purple; rebuildCursor() end)
VisualTab:Slider("Размер курсора", 8, 80, 24, function(v) cursorState.size = v; rebuildCursor() end)
VisualTab:Slider("Прозрачность курсора %", 0, 100, 0, function(v) cursorState.transparency = v / 100; rebuildCursor() end)

VisualTab:Section("Chams")
local chamsColor = ChamsColors.Purple
local chamsFill = 0.3
local chamsConn = {}

local function clearChams()
	for _, c in ipairs(chamsConn) do c:Disconnect() end
	chamsConn = {}
	for _, plr in ipairs(Players:GetPlayers()) do
		if plr.Character then
			for _, part in ipairs(plr.Character:GetDescendants()) do
				if part:IsA("BasePart") then
					if part:GetAttribute("BerretaChamsColor") ~= nil then part.Color = part:GetAttribute("BerretaChamsColor"); part:SetAttribute("BerretaChamsColor", nil) end
					if part:GetAttribute("BerretaChamsMat") ~= nil then part.Material = Enum.Material.Plastic; part:SetAttribute("BerretaChamsMat", nil) end
					if part:GetAttribute("BerretaChamsTrans") ~= nil then part.Transparency = part:GetAttribute("BerretaChamsTrans"); part:SetAttribute("BerretaChamsTrans", nil) end
				end
			end
		end
	end
end

local function applyChams(plr)
	if plr == LP then return end
	if not plr.Character then return end
	local flag = Berreta.Flags["Chams"]
	if not (flag and flag.Get()) then return end
	for _, part in ipairs(plr.Character:GetDescendants()) do
		if part:IsA("BasePart") then
			if part:GetAttribute("BerretaChamsColor") == nil then
				part:SetAttribute("BerretaChamsColor", part.Color)
				part:SetAttribute("BerretaChamsMat", part.Material)
				part:SetAttribute("BerretaChamsTrans", part.Transparency)
			end
			part.Color = chamsColor
			part.Material = Enum.Material.ForceField
			part.Transparency = chamsFill
		end
	end
end

VisualTab:Toggle("Chams", false, function(state)
	if state then
		for _, plr in ipairs(Players:GetPlayers()) do applyChams(plr) end
		table.insert(chamsConn, Players.PlayerAdded:Connect(function(plr)
			table.insert(chamsConn, plr.CharacterAdded:Connect(function() task.wait(0.3); applyChams(plr) end))
		end))
		for _, plr in ipairs(Players:GetPlayers()) do
			if plr ~= LP then table.insert(chamsConn, plr.CharacterAdded:Connect(function() task.wait(0.3); applyChams(plr) end)) end
		end
	else
		clearChams()
	end
end)
VisualTab:Dropdown("Chams Цвет", {"Purple","Red","Green","Blue","Yellow","Pink","White","Black"}, "Purple", function(v)
	chamsColor = ChamsColors[v] or ChamsColors.Purple
	if Berreta.Flags["Chams"] and Berreta.Flags["Chams"].Get() then
		for _, plr in ipairs(Players:GetPlayers()) do
			if plr ~= LP and plr.Character then
				for _, part in ipairs(plr.Character:GetDescendants()) do
					if part:IsA("BasePart") and part:GetAttribute("BerretaChamsColor") ~= nil then part.Color = chamsColor end
				end
			end
		end
	end
end)
VisualTab:Slider("Chams Прозрачность %", 0, 100, 30, function(v)
	chamsFill = v / 100
	if Berreta.Flags["Chams"] and Berreta.Flags["Chams"].Get() then
		for _, plr in ipairs(Players:GetPlayers()) do
			if plr ~= LP and plr.Character then
				for _, part in ipairs(plr.Character:GetDescendants()) do
					if part:IsA("BasePart") and part:GetAttribute("BerretaChamsColor") ~= nil then part.Transparency = chamsFill end
				end
			end
		end
	end
end)

VisualTab:Section("ESP")
local espConns = {}
local function clearESP()
	for _, c in ipairs(espConns) do c:Disconnect() end
	espConns = {}
	for _, plr in ipairs(Players:GetPlayers()) do
		if plr.Character then
			local h = plr.Character:FindFirstChild("BerretaESP")
			if h then h:Destroy() end
		end
	end
end

local function applyESP(plr)
	if plr == LP then return end
	if not plr.Character then return end
	local espFlag = Berreta.Flags["Player ESP"]
	if not (espFlag and espFlag.Get()) then return end
	if plr.Character:FindFirstChild("BerretaESP") then return end
	local h = Instance.new("Highlight")
	h.Name = "BerretaESP"
	h.FillColor = Theme.Accent
	h.OutlineColor = Theme.White
	h.FillTransparency = 0.4
	h.OutlineTransparency = 0
	h.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
	h.Parent = plr.Character
end

VisualTab:Toggle("Player ESP", false, function(state)
	if state then
		for _, plr in ipairs(Players:GetPlayers()) do applyESP(plr) end
		table.insert(espConns, Players.PlayerAdded:Connect(function(plr)
			table.insert(espConns, plr.CharacterAdded:Connect(function() task.wait(0.3); applyESP(plr) end))
		end))
		for _, plr in ipairs(Players:GetPlayers()) do
			if plr ~= LP then table.insert(espConns, plr.CharacterAdded:Connect(function() task.wait(0.3); applyESP(plr) end)) end
		end
	else
		clearESP()
	end
end)

--============================ CONFIG TAB =============================
ConfigTab:Section("Управление")
local inputHolder = Create("Frame", { Size = UDim2.new(1, -32, 0, 40), BackgroundColor3 = Theme.Element, BorderSizePixel = 0, Parent = ConfigTab.Page })
Round(inputHolder, 10)
Stroke(inputHolder, Theme.Stroke, 1, 0.6)
local nameBox = Create("TextBox", {
	Size = UDim2.new(1, -24, 1, 0), Position = UDim2.new(0, 12, 0, 0),
	BackgroundTransparency = 1, Font = Theme.Font, Text = "",
	PlaceholderText = "Введи имя конфига...", PlaceholderColor3 = Theme.SubText,
	TextSize = 13, TextColor3 = Theme.Text, TextXAlignment = Enum.TextXAlignment.Left,
	ClearTextOnFocus = false, Parent = inputHolder,
})

local refreshList = function() end
ConfigTab:Button("Сохранить конфиг", function()
	local name = nameBox.Text
	if name == "" then Berreta:Notify("Конфиг", "Введи имя!", 2); return end
	local ok = Berreta:SaveConfig(name)
	if ok then Berreta:Notify("Конфиг", "Сохранён: " .. name, 2); refreshList() end
end)
ConfigTab:Button("Обновить список", function() refreshList() end)

ConfigTab:Section("Поделиться")
local shareHolder = Create("Frame", { Size = UDim2.new(1, -32, 0, 76), BackgroundColor3 = Theme.Element, BorderSizePixel = 0, Parent = ConfigTab.Page })
Round(shareHolder, 10)
Stroke(shareHolder, Theme.Stroke, 1, 0.6)
local shareBox = Create("TextBox", {
	Size = UDim2.new(1, -24, 1, -12), Position = UDim2.new(0, 12, 0, 6),
	BackgroundTransparency = 1, Font = Theme.Font, Text = "",
	PlaceholderText = "Вставь строку...", PlaceholderColor3 = Theme.SubText,
	TextSize = 11, TextColor3 = Theme.Text, TextXAlignment = Enum.TextXAlignment.Left,
	TextYAlignment = Enum.TextYAlignment.Top, TextWrapped = true,
	ClearTextOnFocus = false, MultiLine = true, Parent = shareHolder,
})

ConfigTab:Button("Экспорт текущих настроек", function()
	local str = Berreta:ExportString()
	shareBox.Text = str
	pcall(function() if setclipboard then setclipboard(str) end end)
	Berreta:Notify("Конфиг", "Скопировано!", 3)
end)
ConfigTab:Button("Импорт из строки", function()
	local ok, err = Berreta:ImportString(shareBox.Text)
	if ok then Berreta:Notify("Конфиг", "Импорт OK!", 3) else Berreta:Notify("Конфиг", "Ошибка: " .. tostring(err), 3) end
end)
ConfigTab:Button("Очистить поле", function() shareBox.Text = "" end)

ConfigTab:Section("Сохранённые конфиги")
local listHolder = Create("Frame", { Size = UDim2.new(1, -32, 0, 0), AutomaticSize = Enum.AutomaticSize.Y, BackgroundTransparency = 1, Parent = ConfigTab.Page })
List(listHolder, 6)

refreshList = function()
	for _, child in ipairs(listHolder:GetChildren()) do
		if child:IsA("Frame") or child:IsA("TextLabel") then child:Destroy() end
	end
	local configs = Berreta:GetConfigList()
	if #configs == 0 then
		Create("TextLabel", { Size = UDim2.new(1, 0, 0, 30), BackgroundTransparency = 1, Font = Theme.Font, Text = "Нет конфигов", TextSize = 12, TextColor3 = Theme.SubText, TextXAlignment = Enum.TextXAlignment.Left, Parent = listHolder })
		return
	end
	for _, cfgName in ipairs(configs) do
		local row = Create("Frame", { Size = UDim2.new(1, 0, 0, 38), BackgroundColor3 = Theme.Element, BorderSizePixel = 0, Parent = listHolder })
		Round(row, 10)
		Stroke(row, Theme.Stroke, 1, 0.6)
		Create("TextLabel", { Size = UDim2.new(1, -250, 1, 0), Position = UDim2.new(0, 12, 0, 0), BackgroundTransparency = 1, Font = Theme.FontMed, Text = cfgName, TextSize = 12, TextColor3 = Theme.Text, TextXAlignment = Enum.TextXAlignment.Left, TextTruncate = Enum.TextTruncate.AtEnd, Parent = row })

		local expBtn = Create("TextButton", { Size = UDim2.fromOffset(70, 26), Position = UDim2.new(1, -190, 0.5, 0), AnchorPoint = Vector2.new(0, 0.5), BackgroundColor3 = Theme.ElementHl, BorderSizePixel = 0, Text = "Скопир.", Font = Theme.FontMed, TextSize = 10, TextColor3 = Theme.Text, AutoButtonColor = false, Parent = row })
		Round(expBtn, 7)
		expBtn.MouseButton1Click:Connect(function()
			local str = Berreta:ExportFile(cfgName)
			if str then
				shareBox.Text = str
				pcall(function() if setclipboard then setclipboard(str) end end)
				Berreta:Notify("Конфиг", "Скопировано", 2)
			end
		end)

		local loadBtn = Create("TextButton", { Size = UDim2.fromOffset(76, 26), Position = UDim2.new(1, -112, 0.5, 0), AnchorPoint = Vector2.new(0, 0.5), BackgroundColor3 = Theme.Accent, BorderSizePixel = 0, Text = "Загрузить", Font = Theme.FontMed, TextSize = 10, TextColor3 = Theme.White, AutoButtonColor = false, Parent = row })
		Round(loadBtn, 7)
		loadBtn.MouseButton1Click:Connect(function()
			local ok = Berreta:LoadConfig(cfgName)
			if ok then Berreta:Notify("Конфиг", "Загружен", 2) end
		end)

		local delBtn = Create("TextButton", { Size = UDim2.fromOffset(26, 26), Position = UDim2.new(1, -30, 0.5, 0), AnchorPoint = Vector2.new(1, 0.5), BackgroundColor3 = Theme.ElementHl, BorderSizePixel = 0, Text = "✕", Font = Theme.FontBold, TextSize = 12, TextColor3 = Color3.fromRGB(248, 113, 113), AutoButtonColor = false, Parent = row })
		Round(delBtn, 7)
		delBtn.MouseButton1Click:Connect(function()
			Berreta:DeleteConfig(cfgName)
			refreshList()
		end)
	end
end
task.defer(refreshList)

--============================ MISC TAB ==============================
MiscTab:Section("Меню")
MiscTab:Label("Клавиша: " .. Berreta.ToggleKey.Name, Theme.SubText)
MiscTab:Button("Сменить клавишу меню", function()
	Berreta:Notify("Меню", "Нажми клавишу (Escape — отмена)", 5)
	local conn
	conn = UIS.InputBegan:Connect(function(input, gpe)
		if gpe then return end
		if input.UserInputType == Enum.UserInputType.Keyboard then
			conn:Disconnect()
			if input.KeyCode == Enum.KeyCode.Escape then
				Berreta:Notify("Меню", "Отменено", 2)
			else
				Berreta.ToggleKey = input.KeyCode
				Berreta:Notify("Меню", "Клавиша: " .. input.KeyCode.Name, 3)
			end
		end
	end)
end)
MiscTab:Button("Сбросить позицию кнопки", function()
	toggleBtn.Position = UDim2.new(0, 20, 0.5, -26)
end)

MiscTab:Section("Сервер")
MiscTab:Button("Rejoin Server", function()
	TeleportService:Teleport(game.PlaceId, LP)
end, Theme.SubText)
MiscTab:Button("Server Hop", function()
	task.spawn(function()
		pcall(function()
			local servers = game:HttpGet("https://games.roblox.com/v1/games/" .. game.PlaceId .. "/servers/Public?sortOrder=Asc&limit=100")
			local data = HttpService:JSONDecode(servers)
			for _, srv in ipairs(data.data) do
				if srv.playing < srv.maxPlayers and srv.id ~= game.JobId then
					TeleportService:TeleportToPlaceInstance(game.PlaceId, srv.id, LP)
					return
				end
			end
		end)
	end)
end, Theme.SubText)

MiscTab:Section("Информация")
MiscTab:Label("Berreta v3.4", Theme.SubText)
MiscTab:Label("Made with ❤️", Color3.fromRGB(255, 105, 180))

--============================ COMBAT TAB =============================
CombatTab:Section("Hitbox Expander")
local hitboxConn
local activeBoxes = {}
local activeOutlines = {}

local function removeHitbox(plr)
	local box = activeBoxes[plr]
	if box and box.Parent then box:Destroy() end
	activeBoxes[plr] = nil
	local outline = activeOutlines[plr]
	if outline and outline.Parent then outline:Destroy() end
	activeOutlines[plr] = nil
	if plr.Character then
		for _, part in ipairs(plr.Character:GetDescendants()) do
			if part:IsA("BasePart") and part.Name ~= "BerretaVisualBox" then
				if part:GetAttribute("BerretaOrigSize") ~= nil then part.Size = part:GetAttribute("BerretaOrigSize"); part:SetAttribute("BerretaOrigSize", nil) end
				if part:GetAttribute("BerretaOrigTrans") ~= nil then part.Transparency = part:GetAttribute("BerretaOrigTrans"); part:SetAttribute("BerretaOrigTrans", nil) end
				if part:GetAttribute("BerretaOrigCollide") ~= nil then part.CanCollide = part:GetAttribute("BerretaOrigCollide"); part:SetAttribute("BerretaOrigCollide", nil) end
				if part:GetAttribute("BerretaOrigMassless") ~= nil then part.Massless = part:GetAttribute("BerretaOrigMassless"); part:SetAttribute("BerretaOrigMassless", nil) end
			end
		end
	end
end

local function clearHitboxes()
	for plr in pairs(activeBoxes) do removeHitbox(plr) end
	activeBoxes = {}
	activeOutlines = {}
end

local function createHitbox(plr)
	if plr == LP then return end
	if not plr.Character then return end
	local flagEnabled = Berreta.Flags["Enable Hitbox"] and Berreta.Flags["Enable Hitbox"].Get()
	if not flagEnabled then removeHitbox(plr); return end
	local teamCheck = Berreta.Flags["Team Check"] and Berreta.Flags["Team Check"].Get()
	if teamCheck and plr.Team == LP.Team then removeHitbox(plr); return end
	local sizeVal = Berreta.Flags["Hitbox Size"] and Berreta.Flags["Hitbox Size"].Get() or 10
	local transVal = Berreta.Flags["Transparency %"] and Berreta.Flags["Transparency %"].Get() or 60
	local char = plr.Character
	local hrp = char:FindFirstChild("HumanoidRootPart")
	if not hrp then return end
	local existingBox = activeBoxes[plr]
	if existingBox and (not existingBox.Parent or existingBox.Parent ~= char) then
		if existingBox.Parent then existingBox:Destroy() end
		activeBoxes[plr] = nil
		local eo = activeOutlines[plr]
		if eo and eo.Parent then eo:Destroy() end
		activeOutlines[plr] = nil
	end
	for _, part in ipairs(char:GetDescendants()) do
		if part:IsA("BasePart") and part.Name ~= "BerretaVisualBox" then
			if part:GetAttribute("BerretaOrigSize") == nil then
				part:SetAttribute("BerretaOrigSize", part.Size)
				part:SetAttribute("BerretaOrigTrans", part.Transparency)
				part:SetAttribute("BerretaOrigCollide", part.CanCollide)
				part:SetAttribute("BerretaOrigMassless", part.Massless)
			end
			part.Size = Vector3.new(sizeVal, sizeVal, sizeVal)
			part.Transparency = 1
			part.CanCollide = false
			part.Massless = true
		end
	end
	local box = activeBoxes[plr]
	if not box or not box.Parent then
		box = Instance.new("Part")
		box.Name = "BerretaVisualBox"
		box.Shape = Enum.PartType.Block
		box.Material = Enum.Material.Neon
		box.Color = Color3.fromRGB(139, 92, 246)
		box.Anchored = true
		box.CanCollide = false
		box.CanQuery = false
		box.CanTouch = false
		box.Massless = true
		box.CastShadow = false
		box.TopSurface = Enum.SurfaceType.Smooth
		box.BottomSurface = Enum.SurfaceType.Smooth
		box.Parent = char
		activeBoxes[plr] = box
	end
	box.Size = Vector3.new(sizeVal, sizeVal, sizeVal)
	box.Transparency = 1 - (transVal / 100)
	box.CFrame = hrp.CFrame
	local outline = activeOutlines[plr]
	if not outline or not outline.Parent then
		outline = Instance.new("SelectionBox")
		outline.Name = "BerretaHitboxOutline"
		outline.Color3 = Color3.fromRGB(180, 140, 255)
		outline.LineThickness = 0.15
		outline.SurfaceTransparency = 1
		outline.Adornee = box
		outline.Parent = char
		activeOutlines[plr] = outline
	end
	outline.Adornee = box
end

CombatTab:Toggle("Enable Hitbox", false, function(state)
	if state then
		activeBoxes = {}
		activeOutlines = {}
		for _, plr in ipairs(Players:GetPlayers()) do
			if plr ~= LP and plr.Character then createHitbox(plr) end
		end
		hitboxConn = RunService.Heartbeat:Connect(function()
			for _, plr in ipairs(Players:GetPlayers()) do createHitbox(plr) end
		end)
	else
		if hitboxConn then hitboxConn:Disconnect(); hitboxConn = nil end
		clearHitboxes()
	end
end)

for _, plr in ipairs(Players:GetPlayers()) do
	if plr ~= LP then
		plr.CharacterAdded:Connect(function()
			removeHitbox(plr)
			task.wait(0.3)
			if Berreta.Flags["Enable Hitbox"] and Berreta.Flags["Enable Hitbox"].Get() then createHitbox(plr) end
		end)
	end
end
Players.PlayerAdded:Connect(function(plr)
	plr.CharacterAdded:Connect(function()
		removeHitbox(plr)
		task.wait(0.3)
		if Berreta.Flags["Enable Hitbox"] and Berreta.Flags["Enable Hitbox"].Get() then createHitbox(plr) end
	end)
end)
Players.PlayerRemoving:Connect(function(plr) removeHitbox(plr) end)

CombatTab:Slider("Hitbox Size", 1, 30, 10, function() end)
CombatTab:Slider("Transparency %", 0, 100, 60, function() end)
CombatTab:Toggle("Team Check", false, function() end)

CombatTab:Section("Пресеты")
CombatTab:ButtonRow({
	{ name = "Маленький", callback = function() Berreta.Flags["Hitbox Size"]:Set(5) end },
	{ name = "Средний", callback = function() Berreta.Flags["Hitbox Size"]:Set(10) end },
	{ name = "Большой", callback = function() Berreta.Flags["Hitbox Size"]:Set(20) end },
	{ name = "XXL", callback = function() Berreta.Flags["Hitbox Size"]:Set(30) end },
})

CombatTab:Section("Reach")
CombatTab:Slider("Reach Distance", 5, 50, 10, function(v) Berreta.Flags.Reach = v end)

CombatTab:Section("Silent Aim")
local silentAimEnabled = false
local silentAimFOV = 150
local silentAimHitPart = "Head"
local silentAimTeamCheck = false
local silentAimWallCheck = false
local silentAimVisible = true
local silentAimKey = nil
local silentAimKeyHeld = false

local aimGui = Create("ScreenGui", { Name = "BerretaSilentAim", ResetOnSpawn = false, IgnoreGuiInset = true, DisplayOrder = 99998, ZIndexBehavior = Enum.ZIndexBehavior.Sibling, Parent = getParent() })
local fovCircle = Create("Frame", { Size = UDim2.fromOffset(silentAimFOV * 2, silentAimFOV * 2), Position = UDim2.fromScale(0.5, 0.5), AnchorPoint = Vector2.new(0.5, 0.5), BackgroundTransparency = 1, Visible = false, ZIndex = 5, Parent = aimGui })
Round(fovCircle, 9999)
Create("UIStroke", { Color = Theme.Accent, Thickness = 1.5, Transparency = 0.3, Parent = fovCircle })

local function updateFovCircle()
	fovCircle.Size = UDim2.fromOffset(silentAimFOV * 2, silentAimFOV * 2)
	local c2 = workspace.CurrentCamera
	if c2 then fovCircle.Position = UDim2.fromOffset(c2.ViewportSize.X / 2, c2.ViewportSize.Y / 2) end
end

RunService.RenderStepped:Connect(function()
	if silentAimVisible and silentAimEnabled then fovCircle.Visible = true; updateFovCircle()
	else fovCircle.Visible = false end
end)

local function getClosestTarget()
	local c2 = workspace.CurrentCamera
	if not c2 then return nil end
	local center = Vector2.new(c2.ViewportSize.X / 2, c2.ViewportSize.Y / 2)
	local closest, shortest = nil, silentAimFOV
	for _, plr in ipairs(Players:GetPlayers()) do
		if plr ~= LP and plr.Character then
			local hum = plr.Character:FindFirstChildOfClass("Humanoid")
			local part = plr.Character:FindFirstChild(silentAimHitPart)
			if hum and hum.Health > 0 and part then
				local skip = silentAimTeamCheck and plr.Team == LP.Team
				if not skip then
					local sp, onScreen = c2:WorldToViewportPoint(part.Position)
					if onScreen then
						local d = (Vector2.new(sp.X, sp.Y) - center).Magnitude
						if d < shortest then
							if silentAimWallCheck then
								local params = RaycastParams.new()
								params.FilterType = Enum.RaycastFilterType.Exclude
								params.FilterDescendantsInstances = {LP.Character, c2}
								local ray = workspace:Raycast(c2.CFrame.Position, part.Position - c2.CFrame.Position, params)
								if ray and ray.Instance and ray.Instance:IsDescendantOf(plr.Character) then
									shortest = d; closest = part
								end
							else
								shortest = d; closest = part
							end
						end
					end
				end
			end
		end
	end
	return closest
end

local hooked = false
pcall(function()
	local mt = getrawmetatable(game)
	local oldNamecall = mt.__namecall
	setreadonly(mt, false)
	mt.__namecall = newcclosure(function(self, ...)
		local method = getnamecallmethod()
		if silentAimEnabled and (silentAimKey == nil or silentAimKeyHeld) then
			if method == "Raycast" or method == "FindPartOnRay"
			or method == "FindPartOnRayWithIgnoreList"
			or method == "FindPartOnRayWithWhitelist" then
				local target = getClosestTarget()
				if target then
					local c2 = workspace.CurrentCamera
					local args = {...}
					if method == "Raycast" then
						return oldNamecall(self, c2.CFrame.Position, (target.Position - c2.CFrame.Position), unpack(args, 3))
					else
						return oldNamecall(self, Ray.new(c2.CFrame.Position, (target.Position - c2.CFrame.Position)), unpack(args, 2))
					end
				end
			end
		end
		return oldNamecall(self, ...)
	end)
	setreadonly(mt, true)
	hooked = true
end)

CombatTab:Toggle("Silent Aim", false, function(state)
	if not hooked then Berreta:Notify("Combat", "Hook не установлен", 3); return end
	silentAimEnabled = state
end)
CombatTab:Slider("Aim FOV", 30, 400, 150, function(v) silentAimFOV = v end)
CombatTab:Dropdown("Hit Part", {"Head", "UpperTorso", "LowerTorso", "HumanoidRootPart"}, "Head", function(v) silentAimHitPart = v end)
CombatTab:Toggle("Aim Team Check", false, function(state) silentAimTeamCheck = state end)
CombatTab:Toggle("Aim Wall Check", false, function(state) silentAimWallCheck = state end)
CombatTab:Toggle("Show FOV Circle", true, function(state) silentAimVisible = state end)
CombatTab:Button("Установить клавишу удержания", function()
	Berreta:Notify("Silent Aim", "Нажми клавишу", 5)
	local conn
	conn = UIS.InputBegan:Connect(function(input, gpe)
		if gpe then return end
		if input.UserInputType == Enum.UserInputType.Keyboard then
			conn:Disconnect()
			if input.KeyCode == Enum.KeyCode.Escape then
				silentAimKey = nil
			else
				silentAimKey = input.KeyCode
			end
		end
	end)
end)
UIS.InputBegan:Connect(function(input, gpe)
	if gpe then return end
	if silentAimKey and input.KeyCode == silentAimKey then silentAimKeyHeld = true end
end)
UIS.InputEnded:Connect(function(input, gpe)
	if gpe then return end
	if silentAimKey and input.KeyCode == silentAimKey then silentAimKeyHeld = false end
end)

--============================ TARGET HUD =============================
CombatTab:Section("Target HUD")

local targetHudEnabled = false
local showDistance = true
local showHealthBar = true
local targetHudFOV = 200
local targetHudGui = nil
local targetHudFrame = nil
local targetNameLbl, targetDistLbl, targetHealthFill, targetHealthLbl, targetHbWrap

local hudPosX = 0.5
local hudPosY = 1
local hudAnchorX = 0.5
local hudAnchorY = 1
local hudOffX = 0
local hudOffY = -40

local function applyHudPosition()
	if not targetHudFrame then return end
	targetHudFrame.Position = UDim2.new(hudPosX, hudOffX, hudPosY, hudOffY)
	targetHudFrame.AnchorPoint = Vector2.new(hudAnchorX, hudAnchorY)
end

local function buildTargetHud()
	if targetHudGui then targetHudGui:Destroy() end

	targetHudGui = Create("ScreenGui", {
		Name = "BerretaTargetHUD",
		ResetOnSpawn = false,
		IgnoreGuiInset = true,
		DisplayOrder = 99997,
		ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
		Parent = getParent(),
	})

	targetHudFrame = Create("Frame", {
		Name = "HUD",
		Size = UDim2.fromOffset(220, 48),
		Position = UDim2.new(hudPosX, hudOffX, hudPosY, hudOffY),
		AnchorPoint = Vector2.new(hudAnchorX, hudAnchorY),
		BackgroundTransparency = 1,
		Visible = false,
		Active = true,
		Parent = targetHudGui,
	})

	local topBar = Create("Frame", {
		Size = UDim2.new(1, 0, 0, 22),
		BackgroundColor3 = Color3.fromRGB(18, 18, 24),
		BackgroundTransparency = 0.1,
		BorderSizePixel = 0,
		Parent = targetHudFrame,
	})
	Round(topBar, 6)
	Stroke(topBar, Theme.Stroke, 1, 0.3)

	targetNameLbl = Create("TextLabel", {
		Size = UDim2.new(0.62, -10, 1, 0),
		Position = UDim2.new(0, 10, 0, 0),
		BackgroundTransparency = 1,
		Font = Theme.FontBold,
		Text = "Player",
		TextSize = 13,
		TextColor3 = Theme.White,
		TextXAlignment = Enum.TextXAlignment.Left,
		TextTruncate = Enum.TextTruncate.AtEnd,
		Parent = topBar,
	})

	targetDistLbl = Create("TextLabel", {
		Size = UDim2.new(0.38, -10, 1, 0),
		Position = UDim2.new(0.62, 0, 0, 0),
		BackgroundTransparency = 1,
		Font = Theme.FontMed,
		Text = "0.0 m",
		TextSize = 12,
		TextColor3 = Theme.SubText,
		TextXAlignment = Enum.TextXAlignment.Right,
		Parent = topBar,
	})

	targetHbWrap = Create("Frame", {
		Size = UDim2.new(1, 0, 0, 22),
		Position = UDim2.new(0, 0, 0, 26),
		BackgroundColor3 = Color3.fromRGB(15, 15, 20),
		BackgroundTransparency = 0.05,
		BorderSizePixel = 0,
		ClipsDescendants = true,
		Parent = targetHudFrame,
	})
	Round(targetHbWrap, 6)
	Stroke(targetHbWrap, Theme.Stroke, 1, 0.3)

	targetHealthFill = Create("Frame", {
		Size = UDim2.fromScale(1, 1),
		BackgroundColor3 = Theme.Accent,
		BorderSizePixel = 0,
		Parent = targetHbWrap,
	})
	Round(targetHealthFill, 6)
	Gradient(targetHealthFill, Theme.Pink, Theme.Accent, 0)

	targetHealthLbl = Create("TextLabel", {
		Size = UDim2.fromScale(1, 1),
		BackgroundTransparency = 1,
		Font = Theme.FontBold,
		Text = "100.0",
		TextSize = 13,
		TextColor3 = Theme.White,
		ZIndex = 2,
		Parent = targetHbWrap,
	})
end

local function setHudPreset(preset)
	if not targetHudFrame then return end
	if preset == "TopLeft" then
		hudPosX, hudPosY = 0, 0; hudAnchorX, hudAnchorY = 0, 0; hudOffX, hudOffY = 20, 20
	elseif preset == "TopCenter" then
		hudPosX, hudPosY = 0.5, 0; hudAnchorX, hudAnchorY = 0.5, 0; hudOffX, hudOffY = 0, 20
	elseif preset == "TopRight" then
		hudPosX, hudPosY = 1, 0; hudAnchorX, hudAnchorY = 1, 0; hudOffX, hudOffY = -20, 20
	elseif preset == "CenterLeft" then
		hudPosX, hudPosY = 0, 0.5; hudAnchorX, hudAnchorY = 0, 0.5; hudOffX, hudOffY = 20, 0
	elseif preset == "Center" then
		hudPosX, hudPosY = 0.5, 0.5; hudAnchorX, hudAnchorY = 0.5, 0.5; hudOffX, hudOffY = 0, 0
	elseif preset == "CenterRight" then
		hudPosX, hudPosY = 1, 0.5; hudAnchorX, hudAnchorY = 1, 0.5; hudOffX, hudOffY = -20, 0
	elseif preset == "BottomLeft" then
		hudPosX, hudPosY = 0, 1; hudAnchorX, hudAnchorY = 0, 1; hudOffX, hudOffY = 20, -40
	elseif preset == "BottomCenter" then
		hudPosX, hudPosY = 0.5, 1; hudAnchorX, hudAnchorY = 0.5, 1; hudOffX, hudOffY = 0, -40
	elseif preset == "BottomRight" then
		hudPosX, hudPosY = 1, 1; hudAnchorX, hudAnchorY = 1, 1; hudOffX, hudOffY = -20, -40
	end
	applyHudPosition()
end

local function findTargetPlayer()
	local c2 = workspace.CurrentCamera
	if not c2 then return nil end
	local center = Vector2.new(c2.ViewportSize.X / 2, c2.ViewportSize.Y / 2)
	local closest, shortest = nil, targetHudFOV
	for _, plr in ipairs(Players:GetPlayers()) do
		if plr ~= LP and plr.Character then
			local hum = plr.Character:FindFirstChildOfClass("Humanoid")
			local hrp = plr.Character:FindFirstChild("HumanoidRootPart")
			if hum and hum.Health > 0 and hrp then
				local pos, onScreen = c2:WorldToViewportPoint(hrp.Position)
				if onScreen then
					local d = (Vector2.new(pos.X, pos.Y) - center).Magnitude
					if d < shortest then shortest = d; closest = plr end
				end
			end
		end
	end
	return closest
end

RunService.RenderStepped:Connect(function()
	if not targetHudEnabled or not targetHudFrame then return end
	local target = findTargetPlayer()
	if not target then
		targetHudFrame.Visible = false
		return
	end
	targetHudFrame.Visible = true
	local char = target.Character
	local hrp = char and char:FindFirstChild("HumanoidRootPart")
	local hum = char and char:FindFirstChildOfClass("Humanoid")
	if not hrp or not hum then return end
	targetNameLbl.Text = target.Name
	if showDistance then
		local myHrp = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
		if myHrp then
			local d = (hrp.Position - myHrp.Position).Magnitude
			targetDistLbl.Text = string.format("%.1f m", d)
			targetDistLbl.Visible = true
		end
	else
		targetDistLbl.Visible = false
	end
	local pct = math.clamp(hum.Health / hum.MaxHealth, 0, 1)
	targetHealthFill.Size = UDim2.new(pct, 0, 1, 0)
	targetHealthLbl.Text = string.format("%.1f", hum.Health)
end)

CombatTab:Toggle("Target HUD", false, function(state)
	targetHudEnabled = state
	if state then
		buildTargetHud()
		Berreta:Notify("Combat", "Target HUD: ВКЛ", 2)
	else
		if targetHudGui then targetHudGui:Destroy(); targetHudGui = nil; targetHudFrame = nil end
		Berreta:Notify("Combat", "Target HUD: ВЫКЛ", 2)
	end
end)

CombatTab:Slider("Target HUD FOV", 50, 500, 200, function(v) targetHudFOV = v end)
CombatTab:Toggle("Show Distance", true, function(state) showDistance = state end)
CombatTab:Toggle("Show Health Bar", true, function(state) showHealthBar = state end)

-- Пресеты позиции
CombatTab:Section("Target HUD Позиция")

CombatTab:ButtonRow({
	{ name = "TopLeft", callback = function() setHudPreset("TopLeft") end },
	{ name = "TopCenter", callback = function() setHudPreset("TopCenter") end },
	{ name = "TopRight", callback = function() setHudPreset("TopRight") end },
	{ name = "Center", callback = function() setHudPreset("Center") end },
})

CombatTab:ButtonRow({
	{ name = "BotLeft", callback = function() setHudPreset("BottomLeft") end },
	{ name = "BotCenter", callback = function() setHudPreset("BottomCenter") end },
	{ name = "BotRight", callback = function() setHudPreset("BottomRight") end },
	{ name = "MidRight", callback = function() setHudPreset("CenterRight") end },
})

CombatTab:Slider("HUD X Offset", -600, 600, 0, function(v)
	hudOffX = v
	applyHudPosition()
end)

CombatTab:Slider("HUD Y Offset", -400, 400, -40, function(v)
	hudOffY = v
	applyHudPosition()
end)

CombatTab:Button("Включить режим перетаскивания HUD", function()
	if not targetHudFrame then
		Berreta:Notify("Target HUD", "Сначала включи Target HUD", 2)
		return
	end
	Berreta:Notify("Target HUD", "Теперь тащи HUD мышкой", 3)

	local dragging, dragStart, startPos
	local conn1 = targetHudFrame.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1
		or input.UserInputType == Enum.UserInputType.Touch then
			dragging = true
			dragStart = input.Position
			startPos = targetHudFrame.Position
		end
	end)
	local conn2 = targetHudFrame.InputEnded:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1
		or input.UserInputType == Enum.UserInputType.Touch then
			dragging = false
		end
	end)
	local conn3 = UIS.InputChanged:Connect(function(input)
		if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement
		or input.UserInputType == Enum.UserInputType.Touch) then
			local d = input.Position - dragStart
			targetHudFrame.Position = UDim2.new(
				startPos.X.Scale, startPos.X.Offset + d.X,
				startPos.Y.Scale, startPos.Y.Offset + d.Y
			)
		end
	end)
end)

--=====================================================================
return Berreta
--=====================================================================
