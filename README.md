--=====================================================================
--              B E R R E T A   •   UI LIBRARY v1.7 (TOGGLE BUTTON)
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
		if v.Name == "Berreta" then v:Destroy() end
	end
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
	Bg        = Color3.fromRGB(17, 17, 22),
	Sidebar   = Color3.fromRGB(13, 13, 17),
	Element   = Color3.fromRGB(26, 26, 33),
	ElementHl = Color3.fromRGB(36, 36, 46),
	Accent    = Color3.fromRGB(139, 92, 246),
	AccentHl  = Color3.fromRGB(162, 122, 255),
	Text      = Color3.fromRGB(238, 238, 245),
	SubText   = Color3.fromRGB(140, 140, 158),
	Stroke    = Color3.fromRGB(40, 40, 52),
	White     = Color3.fromRGB(255, 255, 255),
	Font      = Enum.Font.Gotham,
	FontMed   = Enum.Font.GothamMedium,
	FontBold  = Enum.Font.GothamBold,
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

--=====================================================================
--  ПЛАВАЮЩАЯ КНОПКА ОТКРЫТИЯ МЕНЮ
--=====================================================================
local toggleBtn = Create("TextButton", {
	Name = "ToggleButton",
	Size = UDim2.fromOffset(52, 52),
	Position = UDim2.new(0, 20, 0.5, -26),
	AnchorPoint = Vector2.new(0, 0.5),
	BackgroundColor3 = Theme.Bg,
	BorderSizePixel = 0,
	Text = "",
	AutoButtonColor = false,
	ZIndex = 100,
	Parent = gui,
})
Round(toggleBtn, 14)
Stroke(toggleBtn, Theme.Accent, 2, 0)

-- Иконка "B"
Create("TextLabel", {
	Name = "Icon",
	Size = UDim2.fromScale(1, 1),
	BackgroundTransparency = 1,
	Font = Theme.FontBold,
	Text = "B",
	TextSize = 24,
	TextColor3 = Theme.Accent,
	ZIndex = 101,
	Parent = toggleBtn,
})

-- Пульсирующая обводка (привлекает внимание)
local pulseStroke = toggleBtn:FindFirstChildOfClass("UIStroke")
task.spawn(function()
	while toggleBtn.Parent do
		if pulseStroke then
			Tween(pulseStroke, 1.2, { Transparency = 0.6 })
			task.wait(1.2)
			Tween(pulseStroke, 1.2, { Transparency = 0 })
			task.wait(1.2)
		end
	end
end)

toggleBtn.MouseEnter:Connect(function()
	Tween(toggleBtn, 0.15, { BackgroundColor3 = Theme.Accent })
	Tween(toggleBtn.Icon, 0.15, { TextColor3 = Theme.White })
end)
toggleBtn.MouseLeave:Connect(function()
	Tween(toggleBtn, 0.15, { BackgroundColor3 = Theme.Bg })
	Tween(toggleBtn.Icon, 0.15, { TextColor3 = Theme.Accent })
end)

--=====================================================================
--  ОСНОВНОЕ ОКНО
--=====================================================================
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
Round(main, 12)
Stroke(main, Theme.Stroke, 1, 0)

local sidebar = Create("Frame", {
	Name = "Sidebar",
	Size = UDim2.new(0, 160, 1, 0),
	BackgroundColor3 = Theme.Sidebar,
	BorderSizePixel = 0,
	ZIndex = 2,
	Parent = main,
})

local logoArea = Create("Frame", {
	Name = "Logo",
	Size = UDim2.new(1, 0, 0, 60),
	BackgroundTransparency = 1,
	ZIndex = 2,
	Parent = sidebar,
})

local logoBox = Create("Frame", {
	Size = UDim2.fromOffset(28, 28),
	Position = UDim2.new(0, 16, 0.5, 0),
	AnchorPoint = Vector2.new(0, 0.5),
	BackgroundColor3 = Theme.Accent,
	BorderSizePixel = 0,
	ZIndex = 3,
	Parent = logoArea,
})
Round(logoBox, 8)
Create("TextLabel", {
	Size = UDim2.fromScale(1, 1),
	BackgroundTransparency = 1,
	Font = Theme.FontBold,
	Text = "B",
	TextSize = 16,
	TextColor3 = Theme.White,
	ZIndex = 3,
	Parent = logoBox,
})
Create("TextLabel", {
	Size = UDim2.new(1, -70, 1, 0),
	Position = UDim2.new(0, 54, 0, 0),
	BackgroundTransparency = 1,
	Font = Theme.FontBold,
	Text = "BERRETA",
	TextSize = 15,
	TextColor3 = Theme.Text,
	TextXAlignment = Enum.TextXAlignment.Left,
	ZIndex = 3,
	Parent = logoArea,
})

local tabHolder = Create("Frame", {
	Name = "Tabs",
	Size = UDim2.new(1, 0, 1, -60),
	Position = UDim2.new(0, 0, 0, 60),
	BackgroundTransparency = 1,
	ZIndex = 2,
	Parent = sidebar,
})
List(tabHolder, 4)
Pad(tabHolder, 10, 0)

local content = Create("Frame", {
	Name = "Content",
	Size = UDim2.new(1, -160, 1, 0),
	Position = UDim2.new(0, 160, 0, 0),
	BackgroundColor3 = Theme.Bg,
	BorderSizePixel = 0,
	ZIndex = 2,
	Parent = main,
})

local topbar = Create("Frame", {
	Name = "Topbar",
	Size = UDim2.new(1, 0, 0, 50),
	BackgroundTransparency = 1,
	ZIndex = 3,
	Parent = content,
})
Create("Frame", {
	Size = UDim2.new(1, 0, 0, 1),
	Position = UDim2.new(0, 0, 1, -1),
	BackgroundColor3 = Theme.Stroke,
	BorderSizePixel = 0,
	ZIndex = 3,
	Parent = topbar,
})

local pageTitle = Create("TextLabel", {
	Size = UDim2.new(1, -140, 1, 0),
	Position = UDim2.new(0, 18, 0.5, 0),
	AnchorPoint = Vector2.new(0, 0.5),
	BackgroundTransparency = 1,
	Font = Theme.FontBold,
	Text = "Главная",
	TextSize = 15,
	TextColor3 = Theme.Text,
	TextXAlignment = Enum.TextXAlignment.Left,
	ZIndex = 3,
	Parent = topbar,
})

local function ctrlBtn(text, xOff, hoverColor)
	local b = Create("TextButton", {
		Size = UDim2.fromOffset(26, 26),
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
	Round(b, 6)
	b.MouseEnter:Connect(function()
		Tween(b, 0.15, { BackgroundTransparency = 0, TextColor3 = hoverColor or Theme.Text })
	end)
	b.MouseLeave:Connect(function()
		Tween(b, 0.15, { BackgroundTransparency = 1, TextColor3 = Theme.SubText })
	end)
	return b
end

local closeBtn = ctrlBtn("✕", -18, Color3.fromRGB(248, 113, 113))
local minBtn   = ctrlBtn("—", -50)

local pages = Create("Frame", {
	Name = "Pages",
	Size = UDim2.new(1, 0, 1, -50),
	Position = UDim2.new(0, 0, 0, 50),
	BackgroundTransparency = 1,
	ZIndex = 3,
	Parent = content,
})

local notifHolder = Create("Frame", {
	Name = "Notifications",
	Size = UDim2.new(0, 280, 1, -40),
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
		Size = UDim2.new(1, 0, 0, 58),
		BackgroundTransparency = 1,
		ZIndex = 50,
		Parent = notifHolder,
	})
	local card = Create("Frame", {
		Size = UDim2.fromScale(1, 1),
		Position = UDim2.new(1, 60, 0, 0),
		BackgroundColor3 = Theme.Element,
		BorderSizePixel = 0,
		ZIndex = 50,
		Parent = wrap,
	})
	Round(card, 8)
	Stroke(card, Theme.Stroke, 1, 0.3)

	Create("Frame", {
		Size = UDim2.new(0, 3, 1, -16),
		Position = UDim2.new(0, 8, 0.5, 0),
		AnchorPoint = Vector2.new(0, 0.5),
		BackgroundColor3 = Theme.Accent,
		BorderSizePixel = 0,
		ZIndex = 51,
		Parent = card,
	})
	Create("TextLabel", {
		Size = UDim2.new(1, -34, 0, 16),
		Position = UDim2.new(0, 20, 0, 11),
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
		Size = UDim2.new(1, -34, 0, 16),
		Position = UDim2.new(0, 20, 0, 29),
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

	Tween(card, 0.3, { Position = UDim2.new(0, 0, 0, 0) }, Enum.EasingStyle.Quint)
	task.delay(duration, function()
		if not card.Parent then return end
		Tween(card, 0.25, { Position = UDim2.new(1, 60, 0, 0) }, Enum.EasingStyle.Quint, Enum.EasingDirection.In)
		task.wait(0.28)
		wrap:Destroy()
	end)
end

--=====================================================================
--  ОТКРЫТИЕ / ЗАКРЫТИЕ
--=====================================================================
local opened = false

function Berreta:SetOpen(state)
	opened = state
	if state then
		root.Visible = true
		Tween(root, 0.25, { GroupTransparency = 0 })
		Tween(rootScale, 0.3, { Scale = 1 }, Enum.EasingStyle.Back, Enum.EasingDirection.Out)
		Tween(blur, 0.3, { Size = 16 })
	else
		Tween(root, 0.2, { GroupTransparency = 1 })
		Tween(rootScale, 0.25, { Scale = 0.94 })
		Tween(blur, 0.25, { Size = 0 })
		task.delay(0.25, function()
			if not opened then root.Visible = false end
		end)
	end
end

function Berreta:Toggle()
	self:SetOpen(not opened)
end

-- Клик по плавающей кнопке
toggleBtn.MouseButton1Click:Connect(function()
	Berreta:Toggle()
end)

-- Кнопка-минус и крестик
minBtn.MouseButton1Click:Connect(function() Berreta:SetOpen(false) end)
closeBtn.MouseButton1Click:Connect(function()
	opened = false
	if blur then blur:Destroy() end
	gui:Destroy()
end)

-- Отслеживание клавиши (с защитой от удержания)
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

-- Перетаскивание плавающей кнопки
do
	local dragging, dragInput, dragStart, startPos
	toggleBtn.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1
		or input.UserInputType == Enum.UserInputType.Touch then
			dragging  = true
			dragStart = input.Position
			startPos  = toggleBtn.Position
		end
	end)
	toggleBtn.InputEnded:Connect(function(input)
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
		Tween(t.TabButton, 0.15, {
			BackgroundTransparency = active and 0 or 1,
			BackgroundColor3 = Theme.Element,
		})
		Tween(t.TabLabel, 0.15, {
			TextColor3 = active and Theme.Text or Theme.SubText,
		})
		Tween(t.TabIcon, 0.15, {
			TextColor3 = active and Theme.Accent or Theme.SubText,
		})
		t.TabIndicator.Visible = active
	end
	pageTitle.Text = target.Name
end

function Berreta:Tab(name, icon)
	local btn = Create("TextButton", {
		Name = name,
		Size = UDim2.new(1, -20, 0, 38),
		BackgroundColor3 = Theme.Element,
		BackgroundTransparency = 1,
		BorderSizePixel = 0,
		Text = "",
		AutoButtonColor = false,
		ZIndex = 3,
		Parent = tabHolder,
	})
	Round(btn, 8)

	local indicator = Create("Frame", {
		Size = UDim2.fromOffset(3, 16),
		Position = UDim2.new(0, 0, 0.5, 0),
		AnchorPoint = Vector2.new(0, 0.5),
		BackgroundColor3 = Theme.Accent,
		BorderSizePixel = 0,
		Visible = false,
		ZIndex = 4,
		Parent = btn,
	})
	Round(indicator, 2)

	local iconLbl = Create("TextLabel", {
		Size = UDim2.fromOffset(20, 20),
		Position = UDim2.new(0, 12, 0.5, 0),
		AnchorPoint = Vector2.new(0, 0.5),
		BackgroundTransparency = 1,
		Font = Theme.FontMed,
		Text = icon or "📄",
		TextSize = 14,
		TextColor3 = Theme.SubText,
		ZIndex = 4,
		Parent = btn,
	})

	local label = Create("TextLabel", {
		Size = UDim2.new(1, -40, 1, 0),
		Position = UDim2.new(0, 36, 0, 0),
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
		Name = name,
		Window = self,
	}, Tab)

	table.insert(self.Tabs, tab)

	btn.MouseEnter:Connect(function()
		if Berreta.CurrentTab ~= tab then
			Tween(btn, 0.15, { BackgroundTransparency = 0.6, BackgroundColor3 = Theme.Element })
		end
	end)
	btn.MouseLeave:Connect(function()
		if Berreta.CurrentTab ~= tab then
			Tween(btn, 0.15, { BackgroundTransparency = 1 })
		end
	end)
	btn.MouseButton1Click:Connect(function() Berreta:SelectTab(tab) end)

	if #self.Tabs == 1 then self:SelectTab(tab) end
	return tab
end

function Tab:Section(text)
	local holder = Create("Frame", {
		Size = UDim2.new(1, -32, 0, 26),
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
		Size = UDim2.new(1, -32, 0, 36),
		BackgroundColor3 = Theme.Element,
		BorderSizePixel = 0,
		Text = name,
		Font = Theme.FontMed,
		TextSize = 13,
		TextColor3 = textColor or Theme.Text,
		AutoButtonColor = false,
		Parent = self.Page,
	})
	Round(btn, 8)

	btn.MouseEnter:Connect(function() Tween(btn, 0.15, { BackgroundColor3 = Theme.ElementHl }) end)
	btn.MouseLeave:Connect(function() Tween(btn, 0.15, { BackgroundColor3 = Theme.Element }) end)
	btn.MouseButton1Click:Connect(function()
		Tween(btn, 0.08, { BackgroundColor3 = Theme.Accent })
		task.delay(0.1, function()
			if btn.Parent then Tween(btn, 0.2, { BackgroundColor3 = Theme.ElementHl }) end
		end)
		if callback then task.spawn(callback) end
	end)
	return btn
end

function Tab:ButtonRow(buttons)
	local holder = Create("Frame", {
		Size = UDim2.new(1, -32, 0, 34),
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
		Round(btn, 6)
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
		Size = UDim2.new(1, -32, 0, 40),
		BackgroundColor3 = Theme.Element,
		BorderSizePixel = 0,
		Parent = self.Page,
	})
	Round(holder, 8)

	Create("TextLabel", {
		Size = UDim2.new(1, -80, 1, 0),
		Position = UDim2.new(0, 14, 0, 0),
		BackgroundTransparency = 1,
		Font = Theme.Font,
		Text = name,
		TextSize = 13,
		TextColor3 = Theme.Text,
		TextXAlignment = Enum.TextXAlignment.Left,
		Parent = holder,
	})

	local track = Create("TextButton", {
		Size = UDim2.fromOffset(40, 22),
		Position = UDim2.new(1, -54, 0.5, 0),
		AnchorPoint = Vector2.new(0, 0.5),
		BackgroundColor3 = state and Theme.Accent or Theme.ElementHl,
		BorderSizePixel = 0,
		Text = "",
		AutoButtonColor = false,
		Parent = holder,
	})
	Round(track, 11)

	local knob = Create("Frame", {
		Size = UDim2.fromOffset(16, 16),
		Position = UDim2.new(0, state and 21 or 3, 0.5, 0),
		AnchorPoint = Vector2.new(0, 0.5),
		BackgroundColor3 = Theme.White,
		BorderSizePixel = 0,
		Parent = track,
	})
	Round(knob, 8)

	local function set(value, fire)
		state = value and true or false
		Tween(track, 0.18, { BackgroundColor3 = state and Theme.Accent or Theme.ElementHl })
		Tween(knob, 0.18, { Position = UDim2.new(0, state and 21 or 3, 0.5, 0) })
		if fire and callback then task.spawn(callback, state) end
	end

	track.MouseButton1Click:Connect(function() set(not state, true) end)

	local api = {
		Set = function(v) set(v, false) end,
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
		Size = UDim2.new(1, -32, 0, 56),
		BackgroundColor3 = Theme.Element,
		BorderSizePixel = 0,
		Parent = self.Page,
	})
	Round(holder, 8)

	Create("TextLabel", {
		Size = UDim2.new(1, -80, 0, 16),
		Position = UDim2.new(0, 14, 0, 10),
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
		Position = UDim2.new(1, -84, 0, 10),
		BackgroundTransparency = 1,
		Font = Theme.FontMed,
		Text = tostring(default),
		TextSize = 12,
		TextColor3 = Theme.Accent,
		TextXAlignment = Enum.TextXAlignment.Right,
		Parent = holder,
	})

	local track = Create("Frame", {
		Size = UDim2.new(1, -28, 0, 6),
		Position = UDim2.new(0, 14, 0, 38),
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

	local knob = Create("Frame", {
		Size = UDim2.fromOffset(14, 14),
		Position = UDim2.new(0, 0, 0.5, 0),
		AnchorPoint = Vector2.new(0.5, 0.5),
		BackgroundColor3 = Theme.White,
		BorderSizePixel = 0,
		ZIndex = 2,
		Parent = track,
	})
	Round(knob, 7)
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
		end
	end)

	setValue(default, false)
	local api = {
		Set = function(v) setValue(v, false) end,
		Get = function() return tonumber(valueLabel.Text) end,
	}
	Berreta.Flags[name] = api
	return api
end

--=====================================================================
--  ЗАПУСК
--=====================================================================
Draggable(root, logoArea)
Draggable(root, topbar)

task.wait(0.15)
Berreta:SetOpen(false)
task.wait(0.3)
Berreta:Notify("Berreta", "Нажми кнопку 'B' слева или " .. Berreta.ToggleKey.Name, 4)

local MainTab   = Berreta:Tab("Главная", "🔒")
local PlayerTab = Berreta:Tab("Игрок", "👤")
local VisualTab = Berreta:Tab("Визуал", "✨")
local MiscTab   = Berreta:Tab("Разное", "⚙️")
local CombatTab = Berreta:Tab("Combat", "⚔️")

-- ГЛАВНАЯ
MainTab:Section("Добро пожаловать")
MainTab:Label("Berreta — The best for mvs.", Theme.Accent)
MainTab:Label("Используй вкладки слева для навигации.")

-- ИГРОК
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

-- ВИЗУАЛ
VisualTab:Section("Освещение")
VisualTab:Toggle("Fullbright", false, function(state)
	Lighting.Brightness = state and 3 or 1
	Lighting.Ambient = state and Color3.fromRGB(255, 255, 255) or Color3.fromRGB(0, 0, 0)
	Lighting.OutdoorAmbient = state and Color3.fromRGB(255, 255, 255) or Color3.fromRGB(0, 0, 0)
end)

VisualTab:Toggle("No Fog", false, function(state)
	Lighting.FogEnd = state and 100000 or 1000
	Lighting.FogStart = state and 100000 or 0
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
	h.FillTransparency = 0.5
	h.OutlineTransparency = 0
	h.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
	h.Parent = plr.Character
end

VisualTab:Toggle("Player ESP", false, function(state)
	if state then
		for _, plr in ipairs(Players:GetPlayers()) do applyESP(plr) end

		table.insert(espConns, Players.PlayerAdded:Connect(function(plr)
			table.insert(espConns, plr.CharacterAdded:Connect(function()
				task.wait(0.3)
				applyESP(plr)
			end))
		end))

		for _, plr in ipairs(Players:GetPlayers()) do
			if plr ~= LP then
				table.insert(espConns, plr.CharacterAdded:Connect(function()
					task.wait(0.3)
					applyESP(plr)
				end))
			end
		end
		Berreta:Notify("Visual", "Player ESP: ВКЛ", 2)
	else
		clearESP()
		Berreta:Notify("Visual", "Player ESP: ВЫКЛ", 2)
	end
end)

-- РАЗНОЕ
MiscTab:Section("Меню")
MiscTab:Label("Клавиша открытия: " .. Berreta.ToggleKey.Name, Theme.SubText)

MiscTab:Button("Сменить клавишу меню", function()
	Berreta:Notify("Меню", "Нажми любую клавишу... (Escape — отмена)", 5)
	local conn
	conn = UIS.InputBegan:Connect(function(input, gpe)
		if gpe then return end
		if input.UserInputType == Enum.UserInputType.Keyboard then
			if input.KeyCode == Enum.KeyCode.Escape then
				conn:Disconnect()
				Berreta:Notify("Меню", "Отменено", 2)
				return
			end
			Berreta.ToggleKey = input.KeyCode
			conn:Disconnect()
			Berreta:Notify("Меню", "Клавиша: " .. input.KeyCode.Name, 3)
		end
	end)
end)

MiscTab:Button("Сбросить позицию кнопки", function()
	toggleBtn.Position = UDim2.new(0, 20, 0.5, -26)
	Berreta:Notify("Меню", "Кнопка возвращена на место", 2)
end)

MiscTab:Section("Сервер")

MiscTab:Button("Rejoin Server", function()
	Berreta:Notify("Сервер", "Переподключение...", 2)
	task.wait(0.5)
	TeleportService:Teleport(game.PlaceId, LP)
end, Theme.SubText)

MiscTab:Button("Server Hop", function()
	Berreta:Notify("Сервер", "Поиск нового сервера...", 2)
	task.spawn(function()
		local ok = pcall(function()
			local servers = game:HttpGet("https://games.roblox.com/v1/games/" .. game.PlaceId .. "/servers/Public?sortOrder=Asc&limit=100")
			local data = HttpService:JSONDecode(servers)
			for _, srv in ipairs(data.data) do
				if srv.playing < srv.maxPlayers and srv.id ~= game.JobId then
					TeleportService:TeleportToPlaceInstance(game.PlaceId, srv.id, LP)
					return
				end
			end
			Berreta:Notify("Сервер", "Свободных серверов нет", 2)
		end)
		if not ok then
			Berreta:Notify("Ошибка", "Server Hop не сработал", 2)
		end
	end)
end, Theme.SubText)

MiscTab:Section("Информация")
MiscTab:Label("Berreta v1.0", Theme.SubText)
MiscTab:Label("Made with ❤️", Color3.fromRGB(255, 105, 180))

-- COMBAT
CombatTab:Section("Hitbox Expander")

local hitboxConn
local function clearHitboxes()
	for _, plr in ipairs(Players:GetPlayers()) do
		if plr.Character then
			for _, part in ipairs(plr.Character:GetDescendants()) do
				if part:IsA("BasePart") then
					if part:GetAttribute("BerretaOrigSize") then
						part.Size = part:GetAttribute("BerretaOrigSize")
						part.Transparency = part:GetAttribute("BerretaOrigTrans") or part.Transparency
					end
				end
			end
		end
	end
end

local function expandHitbox(plr)
	if plr == LP then return end
	if not plr.Character then return end

	local flagEnabled = Berreta.Flags["Enable Hitbox"] and Berreta.Flags["Enable Hitbox"].Get()
	if not flagEnabled then return end

	local teamCheck = Berreta.Flags["Team Check"] and Berreta.Flags["Team Check"].Get()
	if teamCheck and plr.Team == LP.Team then return end

	local sizeVal = Berreta.Flags["Hitbox Size"] and Berreta.Flags["Hitbox Size"].Get() or 15
	local transVal = Berreta.Flags["Transparency %"] and Berreta.Flags["Transparency %"].Get() or 70

	for _, part in ipairs(plr.Character:GetDescendants()) do
		if part:IsA("BasePart") then
			if not part:GetAttribute("BerretaOrigSize") then
				part:SetAttribute("BerretaOrigSize", part.Size)
				part:SetAttribute("BerretaOrigTrans", part.Transparency)
			end
			part.Size = Vector3.new(sizeVal, sizeVal, sizeVal)
			part.Transparency = 1 - (transVal / 100)
			part.CanCollide = false
		end
	end
end

CombatTab:Toggle("Enable Hitbox", false, function(state)
	if state then
		hitboxConn = RunService.Heartbeat:Connect(function()
			for _, plr in ipairs(Players:GetPlayers()) do
				expandHitbox(plr)
			end
		end)
		Berreta:Notify("Combat", "Hitbox Expander: ВКЛ", 2)
	else
		if hitboxConn then hitboxConn:Disconnect(); hitboxConn = nil end
		clearHitboxes()
		Berreta:Notify("Combat", "Hitbox Expander: ВЫКЛ", 2)
	end
end)

CombatTab:Slider("Hitbox Size", 1, 30, 15, function(value) end)
CombatTab:Slider("Transparency %", 0, 100, 70, function(value) end)
CombatTab:Toggle("Team Check", false, function(state) end)

CombatTab:Section("Пресеты")
CombatTab:ButtonRow({
	{ name = "Маленький", callback = function()
		Berreta.Flags["Hitbox Size"]:Set(5)
		Berreta:Notify("Пресет", "Размер установлен на 5", 2)
	end },
	{ name = "Средний", callback = function()
		Berreta.Flags["Hitbox Size"]:Set(10)
		Berreta:Notify("Пресет", "Размер установлен на 10", 2)
	end },
	{ name = "Большой", callback = function()
		Berreta.Flags["Hitbox Size"]:Set(20)
		Berreta:Notify("Пресет", "Размер установлен на 20", 2)
	end },
	{ name = "XXL", callback = function()
		Berreta.Flags["Hitbox Size"]:Set(30)
		Berreta:Notify("Пресет", "Размер установлен на 30", 2)
	end },
})

CombatTab:Section("Reach")
CombatTab:Slider("Reach Distance", 5, 50, 10, function(v)
	Berreta.Flags.Reach = v
end)

--=====================================================================
return Berreta
--=====================================================================
