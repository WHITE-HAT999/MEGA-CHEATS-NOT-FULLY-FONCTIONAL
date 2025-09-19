local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

-- Configuration par défaut (personnalisable)
local Config = {
	-- Fonctionnalités
	Fly = {
		Enabled = false,
		Speed = 100,
		MaxSpeed = 500,
		Effects = true,
		EffectColor = Color3.fromRGB(0, 170, 255),
		EffectSize = 0.5,
		EffectRate = 25,
		NoClip = false
	},
	Speed = {
		Enabled = false,
		Value = 50,
		MaxValue = 500,
		JumpPower = 75
	},
	XRay = {
		Enabled = false,
		Transparency = 0.8,
		ExcludeCharacters = true
	},
	GodMode = {
		Enabled = false,
		AutoHeal = true,
		InfiniteJump = false
	},
	KillAura = {
		Enabled = false,
		Range = 15,
		Damage = 100,
		AutoTarget = true,
		VisualFeedback = true
	},
	Teleport = {
		Enabled = false,
		SavedLocations = {},
		ClickTP = false
	},
	ESP = {
		Enabled = false,
		ShowNames = true,
		ShowDistance = true,
		ShowHealth = true,
		TeamColors = true,
		BoxOutline = true
	},
	-- Interface
	UI = {
		MainColor = Color3.fromRGB(25, 25, 35),
		SecondaryColor = Color3.fromRGB(35, 35, 45),
		AccentColor = Color3.fromRGB(45, 125, 255),
		TextColor = Color3.fromRGB(255, 255, 255),
		Font = Enum.Font.GothamSemibold,
		ToggleKey = Enum.KeyCode.RightShift,
		Transparency = 0.1,
		Rounded = true,
		RoundRadius = 8,
		IconSize = 20,
		SliderSize = 150,
		MenuWidth = 220,
		MenuHeight = 300,
		ShowShadows = true
	}
}

-- Variables globales
local flyVelocity
local flyDirection = Vector3.new(0, 0, 0)
local currentMenu = nil
local espObjects = {}
local savedLocations = {}
local clickTPTool = nil
local characterConnections = {}
local tpLocations = {}
local playerESPs = {}
local originalProperties = {}

-- Créer l'interface principale
local gui = Instance.new("ScreenGui")
gui.Name = "UltraPowerMenu"
gui.ResetOnSpawn = false
gui.DisplayOrder = 1000
gui.Parent = PlayerGui

-- Container principal (pour tout l'UI)
local mainContainer = Instance.new("Frame")
mainContainer.Name = "MainContainer"
mainContainer.Size = UDim2.new(0, Config.UI.MenuWidth * 2 + 10, 0, Config.UI.MenuHeight)
mainContainer.Position = UDim2.new(0, 20, 0.5, -Config.UI.MenuHeight/2)
mainContainer.BackgroundTransparency = 1
mainContainer.Parent = gui

-- Menu principal
local mainMenu = Instance.new("Frame")
mainMenu.Name = "MainMenu"
mainMenu.Size = UDim2.new(0, Config.UI.MenuWidth, 0, Config.UI.MenuHeight)
mainMenu.Position = UDim2.new(0, 0, 0, 0)
mainMenu.BackgroundColor3 = Config.UI.MainColor
mainMenu.BackgroundTransparency = Config.UI.Transparency
mainMenu.Parent = mainContainer

-- Titre du menu principal
local mainTitle = Instance.new("TextLabel")
mainTitle.Name = "Title"
mainTitle.Size = UDim2.new(1, 0, 0, 40)
mainTitle.BackgroundColor3 = Config.UI.AccentColor
mainTitle.TextColor3 = Config.UI.TextColor
mainTitle.Text = "🔥 MEGA CHEATS 🔥"
mainTitle.Font = Config.UI.Font
mainTitle.TextSize = 20
mainTitle.Parent = mainMenu

-- Arrondir les coins si activé
if Config.UI.Rounded then
	local corners = Instance.new("UICorner")
	corners.CornerRadius = UDim.new(0, Config.UI.RoundRadius)
	corners.Parent = mainMenu

	local titleCorners = Instance.new("UICorner")
	titleCorners.CornerRadius = UDim.new(0, Config.UI.RoundRadius)
	titleCorners.Parent = mainTitle
end

-- Conteneur de boutons pour le menu principal
local buttonContainer = Instance.new("ScrollingFrame")
buttonContainer.Name = "ButtonContainer"
buttonContainer.Size = UDim2.new(1, 0, 1, -45)
buttonContainer.Position = UDim2.new(0, 0, 0, 45)
buttonContainer.BackgroundTransparency = 1
buttonContainer.ScrollBarThickness = 4
buttonContainer.ScrollingDirection = Enum.ScrollingDirection.Y
buttonContainer.AutomaticCanvasSize = Enum.AutomaticSize.Y
buttonContainer.CanvasSize = UDim2.new(0, 0, 0, 0)
buttonContainer.Parent = mainMenu

-- Organisation des boutons
local listLayout = Instance.new("UIListLayout")
listLayout.Padding = UDim.new(0, 8)
listLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
listLayout.Parent = buttonContainer

-- Padding pour l'espacement
local padding = Instance.new("UIPadding")
padding.PaddingTop = UDim.new(0, 8)
padding.PaddingBottom = UDim.new(0, 8)
padding.PaddingLeft = UDim.new(0, 8)
padding.PaddingRight = UDim.new(0, 8)
padding.Parent = buttonContainer

-- Sous-menu
local subMenu = Instance.new("Frame")
subMenu.Name = "SubMenu"
subMenu.Size = UDim2.new(0, Config.UI.MenuWidth, 0, Config.UI.MenuHeight)
subMenu.Position = UDim2.new(0, Config.UI.MenuWidth + 10, 0, 0)
subMenu.BackgroundColor3 = Config.UI.SecondaryColor
subMenu.BackgroundTransparency = Config.UI.Transparency
subMenu.Visible = false
subMenu.Parent = mainContainer

-- Titre du sous-menu
local subTitle = Instance.new("TextLabel")
subTitle.Name = "Title"
subTitle.Size = UDim2.new(1, 0, 0, 40)
subTitle.BackgroundColor3 = Config.UI.AccentColor
subTitle.TextColor3 = Config.UI.TextColor
subTitle.Text = "Options"
subTitle.Font = Config.UI.Font
subTitle.TextSize = 18
subTitle.Parent = subMenu

-- Arrondir les coins du sous-menu si activé
if Config.UI.Rounded then
	local corners = Instance.new("UICorner")
	corners.CornerRadius = UDim.new(0, Config.UI.RoundRadius)
	corners.Parent = subMenu

	local titleCorners = Instance.new("UICorner")
	titleCorners.CornerRadius = UDim.new(0, Config.UI.RoundRadius)
	titleCorners.Parent = subTitle
end

-- Conteneur de contrôles pour le sous-menu
local controlContainer = Instance.new("ScrollingFrame")
controlContainer.Name = "ControlContainer"
controlContainer.Size = UDim2.new(1, 0, 1, -45)
controlContainer.Position = UDim2.new(0, 0, 0, 45)
controlContainer.BackgroundTransparency = 1
controlContainer.ScrollBarThickness = 4
controlContainer.ScrollingDirection = Enum.ScrollingDirection.Y
controlContainer.AutomaticCanvasSize = Enum.AutomaticSize.Y
controlContainer.CanvasSize = UDim2.new(0, 0, 0, 0)
controlContainer.Parent = subMenu

-- Organisation des contrôles
local controlLayout = Instance.new("UIListLayout")
controlLayout.Padding = UDim.new(0, 10)
controlLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
controlLayout.Parent = controlContainer

-- Padding pour l'espacement
local controlPadding = Instance.new("UIPadding")
controlPadding.PaddingTop = UDim.new(0, 10)
controlPadding.PaddingBottom = UDim.new(0, 10)
controlPadding.PaddingLeft = UDim.new(0, 10)
controlPadding.PaddingRight = UDim.new(0, 10)
controlPadding.Parent = controlContainer

-- Panel de personnalisation UI
local customizeMenu = Instance.new("Frame")
customizeMenu.Name = "CustomizeMenu"
customizeMenu.Size = UDim2.new(0, Config.UI.MenuWidth, 0, Config.UI.MenuHeight)
customizeMenu.Position = UDim2.new(0, Config.UI.MenuWidth + 10, 0, 0)
customizeMenu.BackgroundColor3 = Config.UI.SecondaryColor
customizeMenu.BackgroundTransparency = Config.UI.Transparency
customizeMenu.Visible = false
customizeMenu.Parent = mainContainer

-- Titre du panel de personnalisation
local customizeTitle = Instance.new("TextLabel")
customizeTitle.Name = "Title"
customizeTitle.Size = UDim2.new(1, 0, 0, 40)
customizeTitle.BackgroundColor3 = Config.UI.AccentColor
customizeTitle.TextColor3 = Config.UI.TextColor
customizeTitle.Text = "Personnalisation"
customizeTitle.Font = Config.UI.Font
customizeTitle.TextSize = 18
customizeTitle.Parent = customizeMenu

-- Arrondir les coins du panel de personnalisation si activé
if Config.UI.Rounded then
	local corners = Instance.new("UICorner")
	corners.CornerRadius = UDim.new(0, Config.UI.RoundRadius)
	corners.Parent = customizeMenu

	local titleCorners = Instance.new("UICorner")
	titleCorners.CornerRadius = UDim.new(0, Config.UI.RoundRadius)
	titleCorners.Parent = customizeTitle
end

-- Conteneur de contrôles pour le panel de personnalisation
local customizeContainer = Instance.new("ScrollingFrame")
customizeContainer.Name = "CustomizeContainer"
customizeContainer.Size = UDim2.new(1, 0, 1, -45)
customizeContainer.Position = UDim2.new(0, 0, 0, 45)
customizeContainer.BackgroundTransparency = 1
customizeContainer.ScrollBarThickness = 4
customizeContainer.ScrollingDirection = Enum.ScrollingDirection.Y
customizeContainer.AutomaticCanvasSize = Enum.AutomaticSize.Y
customizeContainer.CanvasSize = UDim2.new(0, 0, 0, 0)
customizeContainer.Parent = customizeMenu

-- Organisation des contrôles
local customizeLayout = Instance.new("UIListLayout")
customizeLayout.Padding = UDim.new(0, 10)
customizeLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
customizeLayout.Parent = customizeContainer

-- Padding pour l'espacement
local customizePadding = Instance.new("UIPadding")
customizePadding.PaddingTop = UDim.new(0, 10)
customizePadding.PaddingBottom = UDim.new(0, 10)
customizePadding.PaddingLeft = UDim.new(0, 10)
customizePadding.PaddingRight = UDim.new(0, 10)
customizePadding.Parent = customizeContainer

-- Notification
local notification = Instance.new("Frame")
notification.Name = "Notification"
notification.Size = UDim2.new(0, 250, 0, 60)
notification.Position = UDim2.new(0.5, -125, 0.8, 0)
notification.BackgroundColor3 = Config.UI.MainColor
notification.BackgroundTransparency = Config.UI.Transparency
notification.Visible = false
notification.Parent = gui

-- Arrondir les coins de la notification si activé
if Config.UI.Rounded then
	local corners = Instance.new("UICorner")
	corners.CornerRadius = UDim.new(0, Config.UI.RoundRadius)
	corners.Parent = notification
end

-- Contenu de la notification
local notifText = Instance.new("TextLabel")
notifText.Name = "Text"
notifText.Size = UDim2.new(1, -20, 1, -20)
notifText.Position = UDim2.new(0, 10, 0, 10)
notifText.BackgroundTransparency = 1
notifText.TextColor3 = Config.UI.TextColor
notifText.Font = Config.UI.Font
notifText.TextSize = 16
notifText.TextWrapped = true
notifText.Text = ""
notifText.Parent = notification

-- Fonctions utilitaires
local function notify(message, duration)
	duration = duration or 3
	notifText.Text = message
	notification.Visible = true

	-- Animation
	notification:TweenPosition(UDim2.new(0.5, -125, 0.8, 0), Enum.EasingDirection.Out, Enum.EasingStyle.Quad, 0.5, true)

	-- Disparition après un délai
	task.delay(duration, function()
		notification:TweenPosition(UDim2.new(0.5, -125, 1.1, 0), Enum.EasingDirection.In, Enum.EasingStyle.Quad, 0.5, true, function()
			notification.Visible = false
			notification.Position = UDim2.new(0.5, -125, 0.8, 0)
		end)
	end)
end

local function createButton(text, parent, icon)
	local button = Instance.new("TextButton")
	button.Name = text:gsub("%s+", "") .. "Button"
	button.Size = UDim2.new(1, -16, 0, 40)
	button.BackgroundColor3 = Config.UI.SecondaryColor
	button.BackgroundTransparency = Config.UI.Transparency
	button.TextColor3 = Config.UI.TextColor
	button.Font = Config.UI.Font
	button.TextSize = 16
	button.Text = text
	button.Parent = parent

	-- Effet au survol
	button.MouseEnter:Connect(function()
		button:TweenBackgroundColor3(Config.UI.AccentColor, Enum.EasingDirection.Out, Enum.EasingStyle.Quad, 0.2, true)
	end)

	button.MouseLeave:Connect(function()
		button:TweenBackgroundColor3(Config.UI.SecondaryColor, Enum.EasingDirection.Out, Enum.EasingStyle.Quad, 0.2, true)
	end)

	-- Arrondir les coins si activé
	if Config.UI.Rounded then
		local corners = Instance.new("UICorner")
		corners.CornerRadius = UDim.new(0, Config.UI.RoundRadius)
		corners.Parent = button
	end

	-- Ajouter une icône si fournie
	if icon then
		local iconLabel = Instance.new("TextLabel")
		iconLabel.Size = UDim2.new(0, 20, 0, 20)
		iconLabel.Position = UDim2.new(0, 10, 0.5, -10)
		iconLabel.BackgroundTransparency = 1
		iconLabel.Text = icon
		iconLabel.TextSize = 20
		iconLabel.TextColor3 = Config.UI.TextColor
		iconLabel.Parent = button

		-- Ajuster le texte principal
		button.Text = "    " .. text
	end

	return button
end

local function createToggle(text, parent, defaultValue)
	local container = Instance.new("Frame")
	container.Name = text:gsub("%s+", "") .. "Container"
	container.Size = UDim2.new(1, -16, 0, 35)
	container.BackgroundTransparency = 1
	container.Parent = parent

	local label = Instance.new("TextLabel")
	label.Size = UDim2.new(0.7, 0, 1, 0)
	label.Position = UDim2.new(0, 0, 0, 0)
	label.BackgroundTransparency = 1
	label.TextColor3 = Config.UI.TextColor
	label.Font = Config.UI.Font
	label.TextSize = 16
	label.Text = text
	label.TextXAlignment = Enum.TextXAlignment.Left
	label.Parent = container

	local toggleFrame = Instance.new("Frame")
	toggleFrame.Size = UDim2.new(0, 50, 0, 24)
	toggleFrame.Position = UDim2.new(1, -50, 0.5, -12)
	toggleFrame.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
	toggleFrame.Parent = container

	-- Arrondir les coins si activé
	if Config.UI.Rounded then
		local corners = Instance.new("UICorner")
		corners.CornerRadius = UDim.new(0, 12)
		corners.Parent = toggleFrame
	end

	local toggleButton = Instance.new("Frame")
	toggleButton.Size = UDim2.new(0, 20, 0, 20)
	toggleButton.Position = UDim2.new(0, 2, 0.5, -10)
	toggleButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
	toggleButton.Parent = toggleFrame

	-- Arrondir les coins du bouton
	local buttonCorners = Instance.new("UICorner")
	buttonCorners.CornerRadius = UDim.new(0, 10)
	buttonCorners.Parent = toggleButton

	local toggled = defaultValue or false

	-- Fonction pour mettre à jour l'état du toggle
	local function updateToggle()
		if toggled then
			toggleFrame.BackgroundColor3 = Config.UI.AccentColor
			toggleButton:TweenPosition(UDim2.new(0, 28, 0.5, -10), Enum.EasingDirection.Out, Enum.EasingStyle.Quad, 0.2, true)
		else
			toggleFrame.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
			toggleButton:TweenPosition(UDim2.new(0, 2, 0.5, -10), Enum.EasingDirection.Out, Enum.EasingStyle.Quad, 0.2, true)
		end
	end

	-- Mettre à jour l'état initial
	updateToggle()

	toggleFrame.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			toggled = not toggled
			updateToggle()
			return toggled
		end
	end)

	-- Créer une méthode pour obtenir l'état
	local toggle = {
		Set = function(value)
			toggled = value
			updateToggle()
		end,
		Get = function()
			return toggled
		end,
		Container = container
	}

	return toggle
end

local function createSlider(text, parent, min, max, defaultValue, step)
	min = min or 0
	max = max or 100
	defaultValue = defaultValue or min
	step = step or 1

	local container = Instance.new("Frame")
	container.Name = text:gsub("%s+", "") .. "Container"
	container.Size = UDim2.new(1, -16, 0, 60)
	container.BackgroundTransparency = 1
	container.Parent = parent

	local label = Instance.new("TextLabel")
	label.Size = UDim2.new(0.6, 0, 0, 20)
	label.Position = UDim2.new(0, 0, 0, 0)
	label.BackgroundTransparency = 1
	label.TextColor3 = Config.UI.TextColor
	label.Font = Config.UI.Font
	label.TextSize = 16
	label.Text = text
	label.TextXAlignment = Enum.TextXAlignment.Left
	label.Parent = container

	local valueLabel = Instance.new("TextLabel")
	valueLabel.Size = UDim2.new(0.4, 0, 0, 20)
	valueLabel.Position = UDim2.new(0.6, 0, 0, 0)
	valueLabel.BackgroundTransparency = 1
	valueLabel.TextColor3 = Config.UI.AccentColor
	valueLabel.Font = Config.UI.Font
	valueLabel.TextSize = 16
	valueLabel.Text = tostring(defaultValue)
	valueLabel.TextXAlignment = Enum.TextXAlignment.Right
	valueLabel.Parent = container

	local sliderBG = Instance.new("Frame")
	sliderBG.Name = "Background"
	sliderBG.Size = UDim2.new(1, 0, 0, 15)
	sliderBG.Position = UDim2.new(0, 0, 0, 30)
	sliderBG.BackgroundColor3 = Color3.fromRGB(60, 60, 70)
	sliderBG.Parent = container

	-- Arrondir les coins du fond
	if Config.UI.Rounded then
		local bgCorners = Instance.new("UICorner")
		bgCorners.CornerRadius = UDim.new(0, 7)
		bgCorners.Parent = sliderBG
	end

	local sliderFill = Instance.new("Frame")
	sliderFill.Name = "Fill"
	local fillRatio = (defaultValue - min) / (max - min)
	sliderFill.Size = UDim2.new(fillRatio, 0, 1, 0)
	sliderFill.BackgroundColor3 = Config.UI.AccentColor
	sliderFill.Parent = sliderBG

	-- Arrondir les coins du remplissage
	if Config.UI.Rounded then
		local fillCorners = Instance.new("UICorner")
		fillCorners.CornerRadius = UDim.new(0, 7)
		fillCorners.Parent = sliderFill
	end

	local sliderKnob = Instance.new("Frame")
	sliderKnob.Name = "Knob"
	sliderKnob.Size = UDim2.new(0, 15, 0, 15)
	sliderKnob.Position = UDim2.new(fillRatio, -7.5, 0, 0)
	sliderKnob.BackgroundColor3 = Config.UI.TextColor
	sliderKnob.Parent = sliderBG

	-- Arrondir le bouton
	local knobCorners = Instance.new("UICorner")
	knobCorners.CornerRadius = UDim.new(0, 7)
	knobCorners.Parent = sliderKnob

	-- Valeur actuelle
	local value = defaultValue

	-- Mettre à jour la valeur et l'affichage
	local function update(newValue)
		-- Arrondir à l'étape la plus proche
		local roundedValue = math.floor((newValue - min) / step + 0.5) * step + min
		value = math.clamp(roundedValue, min, max)

		-- Mettre à jour l'affichage
		valueLabel.Text = tostring(value)
		local ratio = (value - min) / (max - min)
		sliderFill:TweenSize(UDim2.new(ratio, 0, 1, 0), Enum.EasingDirection.Out, Enum.EasingStyle.Quad, 0.1, true)
		sliderKnob:TweenPosition(UDim2.new(ratio, -7.5, 0, 0), Enum.EasingDirection.Out, Enum.EasingStyle.Quad, 0.1, true)
	end

	-- Interaction avec le slider
	local dragging = false

	sliderBG.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging = true

			-- Calculer immédiatement la nouvelle position
			local mousePos = input.Position.X
			local sliderPos = sliderBG.AbsolutePosition.X
			local sliderWidth = sliderBG.AbsoluteSize.X
			local ratio = math.clamp((mousePos - sliderPos) / sliderWidth, 0, 1)
			local newValue = min + ratio * (max - min)
			update(newValue)
		end
	end)

	sliderBG.InputEnded:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging = false
		end
	end)

	-- Mise à jour lors du déplacement de la souris
	UIS.InputChanged:Connect(function(input)
		if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
			local mousePos = input.Position.X
			local sliderPos = sliderBG.AbsolutePosition.X
			local sliderWidth = sliderBG.AbsoluteSize.X
			local ratio = math.clamp((mousePos - sliderPos) / sliderWidth, 0, 1)
			local newValue = min + ratio * (max - min)
			update(newValue)
		end
	end)

	return {
		Set = function(newValue)
			update(newValue)
			return value
		end,
		Get = function()
			return value
		end,
		Container = container
	}
end

local function createColorPicker(text, parent, defaultColor)
	defaultColor = defaultColor or Color3.fromRGB(255, 0, 0)

	local container = Instance.new("Frame")
	container.Name = text:gsub("%s+", "") .. "Container"
	container.Size = UDim2.new(1, -16, 0, 35)
	container.BackgroundTransparency = 1
	container.Parent = parent

	local label = Instance.new("TextLabel")
	label.Size = UDim2.new(0.7, 0, 1, 0)
	label.Position = UDim2.new(0, 0, 0, 0)
	label.BackgroundTransparency = 1
	label.TextColor3 = Config.UI.TextColor
	label.Font = Config.UI.Font
	label.TextSize = 16
	label.Text = text
	label.TextXAlignment = Enum.TextXAlignment.Left
	label.Parent = container

	local colorDisplay = Instance.new("Frame")
	colorDisplay.Size = UDim2.new(0, 30, 0, 30)
	colorDisplay.Position = UDim2.new(1, -30, 0.5, -15)
	colorDisplay.BackgroundColor3 = defaultColor
	colorDisplay.Parent = container

	-- Arrondir les coins si activé
	if Config.UI.Rounded then
		local corners = Instance.new("UICorner")
		corners.CornerRadius = UDim.new(0, 5)
		corners.Parent = colorDisplay
	end

	-- Panel de couleurs prédéfinies (visible au clic)
	local colorPanel = Instance.new("Frame")
	colorPanel.Size = UDim2.new(0, 150, 0, 100)
	colorPanel.Position = UDim2.new(1, -150, 1, 10)
	colorPanel.BackgroundColor3 = Config.UI.MainColor
	colorPanel.Visible = false
	colorPanel.ZIndex = 10
	colorPanel.Parent = container

	-- Arrondir les coins du panel
	if Config.UI.Rounded then
		local corners = Instance.new("UICorner")
		corners.CornerRadius = UDim.new(0, Config.UI.RoundRadius)
		corners.Parent = colorPanel
	end

	-- Grille de couleurs
	local gridLayout = Instance.new("UIGridLayout")
	gridLayout.CellSize = UDim2.new(0, 30, 0, 30)
	gridLayout.CellPadding = UDim2.new(0, 5, 0, 5)
	gridLayout.Parent = colorPanel

	local padding = Instance.new("UIPadding")
	padding.PaddingTop = UDim.new(0, 5)
	padding.PaddingBottom = UDim.new(0, 5)
	padding.PaddingLeft = UDim.new(0, 5)
	padding.PaddingRight = UDim.new(0, 5)
	padding.Parent = colorPanel

	-- Couleurs prédéfinies
	local colors = {
		Color3.fromRGB(255, 0, 0),    -- Rouge
		Color3.fromRGB(255, 165, 0),  -- Orange
		Color3.fromRGB(255, 255, 0),  -- Jaune
		Color3.fromRGB(0, 255, 0),    -- Vert
		Color3.fromRGB(0, 255, 255),  -- Cyan
		Color3.fromRGB(0, 0, 255),    -- Bleu
		Color3.fromRGB(138, 43, 226), -- Violet
		Color3.fromRGB(255, 0, 255),  -- Rose
		Color3.fromRGB(255, 255, 255),-- Blanc
		Color3.fromRGB(0, 0, 0),      -- Noir
		Color3.fromRGB(128, 128, 128),-- Gris
		Color3.fromRGB(139, 69, 19)   -- Marron
	}

	local currentColor = defaultColor

	-- Créer les boutons de couleur
	for i, color in ipairs(colors) do
		local colorButton = Instance.new("TextButton")
		colorButton.Size = UDim2.new(0, 25, 0, 25)
		colorButton.BackgroundColor3 = color
		colorButton.Text = ""
		colorButton.Parent = colorPanel

		if Config.UI.Rounded then
			local corners = Instance.new("UICorner")
			corners.CornerRadius = UDim.new(0, 3)
			corners.Parent = colorButton
		end

		colorButton.MouseButton1Click:Connect(function()
			currentColor = color
			colorDisplay.BackgroundColor3 = color
			colorPanel.Visible = false
		end)
	end

	-- Toggle du panel de couleurs
	colorDisplay.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			colorPanel.Visible = not colorPanel.Visible
		end
	end)

	return {
		Get = function()
			return currentColor
		end,
		Set = function(color)
			currentColor = color
			colorDisplay.BackgroundColor3 = color
		end,
		Container = container
	}
end

-- Fonction pour basculer la visibilité des menus
local function showMenu(menu)
	-- Cacher tous les sous-menus
	subMenu.Visible = false
	customizeMenu.Visible = false

	-- Afficher le menu demandé
	if menu == "sub" then
		subMenu.Visible = true
		currentMenu = "sub"
	elseif menu == "customize" then
		customizeMenu.Visible = true
		currentMenu = "customize"
	else
		currentMenu = nil
	end
end

-- Fonctions des cheats
local function toggleFly()
	Config.Fly.Enabled = not Config.Fly.Enabled

	if Config.Fly.Enabled then
		notify("Vol activé! Utilisez WASD + Space/Shift")

		local character = LocalPlayer.Character
		if character and character:FindFirstChild("HumanoidRootPart") then
			local humanoidRootPart = character.HumanoidRootPart

			-- Créer BodyVelocity pour le vol
			flyVelocity = Instance.new("BodyVelocity")
			flyVelocity.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
			flyVelocity.Velocity = Vector3.new(0, 0, 0)
			flyVelocity.Parent = humanoidRootPart

			-- Connexion pour les contrôles de vol
			characterConnections.flyControl = RunService.Heartbeat:Connect(function()
				if not Config.Fly.Enabled or not flyVelocity then return end

				local character = LocalPlayer.Character
				if not character or not character:FindFirstChild("HumanoidRootPart") then return end

				local humanoid = character:FindFirstChild("Humanoid")
				local rootPart = character.HumanoidRootPart
				local camera = workspace.CurrentCamera

				flyDirection = Vector3.new(0, 0, 0)

				-- Contrôles directionnels
				if UIS:IsKeyDown(Enum.KeyCode.W) then
					flyDirection = flyDirection + camera.CFrame.LookVector
				end
				if UIS:IsKeyDown(Enum.KeyCode.S) then
					flyDirection = flyDirection - camera.CFrame.LookVector
				end
				if UIS:IsKeyDown(Enum.KeyCode.A) then
					flyDirection = flyDirection - camera.CFrame.RightVector
				end
				if UIS:IsKeyDown(Enum.KeyCode.D) then
					flyDirection = flyDirection + camera.CFrame.RightVector
				end
				if UIS:IsKeyDown(Enum.KeyCode.Space) then
					flyDirection = flyDirection + Vector3.new(0, 1, 0)
				end
				if UIS:IsKeyDown(Enum.KeyCode.LeftShift) then
					flyDirection = flyDirection - Vector3.new(0, 1, 0)
				end

				-- Appliquer la vélocité
				flyVelocity.Velocity = flyDirection.Unit * Config.Fly.Speed

				-- NoClip si activé
				if Config.Fly.NoClip and humanoid then
					humanoid:ChangeState(Enum.HumanoidStateType.Physics)
				end
			end)
		end
	else
		notify("Vol désactivé")

		-- Nettoyer les connexions et objets de vol
		if characterConnections.flyControl then
			characterConnections.flyControl:Disconnect()
			characterConnections.flyControl = nil
		end

		if flyVelocity then
			flyVelocity:Destroy()
			flyVelocity = nil
		end

		-- Remettre le personnage en état normal
		local character = LocalPlayer.Character
		if character and character:FindFirstChild("Humanoid") then
			character.Humanoid:ChangeState(Enum.HumanoidStateType.Running)
		end
	end
end

local function toggleSpeed()
	Config.Speed.Enabled = not Config.Speed.Enabled

	if Config.Speed.Enabled then
		notify("Vitesse activée: " .. Config.Speed.Value)

		local character = LocalPlayer.Character
		if character and character:FindFirstChild("Humanoid") then
			local humanoid = character.Humanoid
			humanoid.WalkSpeed = Config.Speed.Value
			humanoid.JumpPower = Config.Speed.JumpPower
		end

		-- Connexion pour maintenir la vitesse
		characterConnections.speedControl = LocalPlayer.CharacterAdded:Connect(function(character)
			local humanoid = character:WaitForChild("Humanoid")
			if Config.Speed.Enabled then
				humanoid.WalkSpeed = Config.Speed.Value
				humanoid.JumpPower = Config.Speed.JumpPower
			end
		end)
	else
		notify("Vitesse désactivée")

		local character = LocalPlayer.Character
		if character and character:FindFirstChild("Humanoid") then
			local humanoid = character.Humanoid
			humanoid.WalkSpeed = 16
			humanoid.JumpPower = 50
		end

		if characterConnections.speedControl then
			characterConnections.speedControl:Disconnect()
			characterConnections.speedControl = nil
		end
	end
end

local function toggleXRay()
	Config.XRay.Enabled = not Config.XRay.Enabled

	if Config.XRay.Enabled then
		notify("X-Ray activé")

		-- Sauvegarder et modifier les propriétés des objets
		for _, obj in pairs(workspace:GetDescendants()) do
			if obj:IsA("BasePart") and obj.Name ~= "HumanoidRootPart" then
				if Config.XRay.ExcludeCharacters and obj.Parent:FindFirstChild("Humanoid") then
					continue
				end

				if not originalProperties[obj] then
					originalProperties[obj] = {
						Transparency = obj.Transparency,
						CanCollide = obj.CanCollide
					}
				end

				obj.Transparency = Config.XRay.Transparency
			end
		end
	else
		notify("X-Ray désactivé")

		-- Restaurer les propriétés originales
		for obj, props in pairs(originalProperties) do
			if obj and obj.Parent then
				obj.Transparency = props.Transparency
				obj.CanCollide = props.CanCollide
			end
		end
		originalProperties = {}
	end
end

local function toggleGodMode()
	Config.GodMode.Enabled = not Config.GodMode.Enabled

	if Config.GodMode.Enabled then
		notify("God Mode activé")

		local character = LocalPlayer.Character
		if character and character:FindFirstChild("Humanoid") then
			local humanoid = character.Humanoid

			-- Santé infinie
			if Config.GodMode.AutoHeal then
				characterConnections.godMode = humanoid.HealthChanged:Connect(function()
					if Config.GodMode.Enabled then
						humanoid.Health = humanoid.MaxHealth
					end
				end)
			end

			-- Saut infini
			if Config.GodMode.InfiniteJump then
				characterConnections.infiniteJump = UIS.JumpRequest:Connect(function()
					if Config.GodMode.Enabled then
						humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
					end
				end)
			end
		end
	else
		notify("God Mode désactivé")

		if characterConnections.godMode then
			characterConnections.godMode:Disconnect()
			characterConnections.godMode = nil
		end

		if characterConnections.infiniteJump then
			characterConnections.infiniteJump:Disconnect()
			characterConnections.infiniteJump = nil
		end
	end
end

local function toggleESP()
	Config.ESP.Enabled = not Config.ESP.Enabled

	if Config.ESP.Enabled then
		notify("ESP activé")

		-- Fonction pour créer l'ESP d'un joueur
		local function createESP(player)
			if player == LocalPlayer then return end

			local espGui = Instance.new("BillboardGui")
			espGui.Name = "ESP_" .. player.Name
			espGui.Adornee = nil
			espGui.Size = UDim2.new(0, 200, 0, 50)
			espGui.StudsOffset = Vector3.new(0, 2, 0)
			espGui.Parent = gui

			local espFrame = Instance.new("Frame")
			espFrame.Size = UDim2.new(1, 0, 1, 0)
			espFrame.BackgroundTransparency = 1
			espFrame.Parent = espGui

			local nameLabel = Instance.new("TextLabel")
			nameLabel.Size = UDim2.new(1, 0, 0.5, 0)
			nameLabel.BackgroundTransparency = 1
			nameLabel.Text = player.Name
			nameLabel.TextColor3 = Config.ESP.TeamColors and player.TeamColor.Color or Color3.fromRGB(255, 255, 255)
			nameLabel.Font = Config.UI.Font
			nameLabel.TextSize = 16
			nameLabel.TextStrokeTransparency = 0
			nameLabel.Parent = espFrame

			local distanceLabel = Instance.new("TextLabel")
			distanceLabel.Size = UDim2.new(1, 0, 0.5, 0)
			distanceLabel.Position = UDim2.new(0, 0, 0.5, 0)
			distanceLabel.BackgroundTransparency = 1
			distanceLabel.Text = "0m"
			distanceLabel.TextColor3 = Color3.fromRGB(255, 255, 0)
			distanceLabel.Font = Config.UI.Font
			distanceLabel.TextSize = 14
			distanceLabel.TextStrokeTransparency = 0
			distanceLabel.Parent = espFrame

			playerESPs[player] = espGui

			-- Mise à jour continue
			local connection
			connection = RunService.Heartbeat:Connect(function()
				if not Config.ESP.Enabled or not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then
					if playerESPs[player] then
						playerESPs[player]:Destroy()
						playerESPs[player] = nil
					end
					connection:Disconnect()
					return
				end

				local character = player.Character
				local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
				local humanoid = character:FindFirstChild("Humanoid")

				if humanoidRootPart then
					espGui.Adornee = humanoidRootPart

					-- Distance
					if Config.ESP.ShowDistance and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
						local distance = (LocalPlayer.Character.HumanoidRootPart.Position - humanoidRootPart.Position).Magnitude
						distanceLabel.Text = math.floor(distance) .. "m"
						distanceLabel.Visible = true
					else
						distanceLabel.Visible = false
					end

					-- Santé
					if Config.ESP.ShowHealth and humanoid then
						nameLabel.Text = player.Name .. " [" .. math.floor(humanoid.Health) .. "/" .. math.floor(humanoid.MaxHealth) .. "]"
					else
						nameLabel.Text = player.Name
					end

					nameLabel.Visible = Config.ESP.ShowNames
				end
			end)
		end

		-- Créer l'ESP pour tous les joueurs existants
		for _, player in pairs(Players:GetPlayers()) do
			createESP(player)
		end

		-- Créer l'ESP pour les nouveaux joueurs
		characterConnections.espPlayerAdded = Players.PlayerAdded:Connect(createESP)

		-- Nettoyer l'ESP des joueurs qui partent
		characterConnections.espPlayerRemoving = Players.PlayerRemoving:Connect(function(player)
			if playerESPs[player] then
				playerESPs[player]:Destroy()
				playerESPs[player] = nil
			end
		end)
	else
		notify("ESP désactivé")

		-- Nettoyer tous les ESPs
		for player, espGui in pairs(playerESPs) do
			espGui:Destroy()
		end
		playerESPs = {}

		if characterConnections.espPlayerAdded then
			characterConnections.espPlayerAdded:Disconnect()
			characterConnections.espPlayerAdded = nil
		end

		if characterConnections.espPlayerRemoving then
			characterConnections.espPlayerRemoving:Disconnect()
			characterConnections.espPlayerRemoving = nil
		end
	end
end

local function toggleClickTP()
	Config.Teleport.ClickTP = not Config.Teleport.ClickTP

	if Config.Teleport.ClickTP then
		notify("Click TP activé - Ctrl + Clic pour téléporter")

		characterConnections.clickTP = UIS.InputBegan:Connect(function(input)
			if input.UserInputType == Enum.UserInputType.MouseButton1 and UIS:IsKeyDown(Enum.KeyCode.LeftControl) then
				local character = LocalPlayer.Character
				if character and character:FindFirstChild("HumanoidRootPart") then
					local mouse = LocalPlayer:GetMouse()
					local hit = mouse.Hit
					if hit then
						character.HumanoidRootPart.CFrame = CFrame.new(hit.Position + Vector3.new(0, 5, 0))
						notify("Téléporté!")
					end
				end
			end
		end)
	else
		notify("Click TP désactivé")

		if characterConnections.clickTP then
			characterConnections.clickTP:Disconnect()
			characterConnections.clickTP = nil
		end
	end
end

-- Créer les boutons du menu principal
local flyButton = createButton("🚀 Vol", buttonContainer, "✈️")
flyButton.MouseButton1Click:Connect(function()
	showMenu("sub")
	subTitle.Text = "Vol - Options"

	-- Nettoyer le conteneur
	for _, child in pairs(controlContainer:GetChildren()) do
		if child:IsA("Frame") and child.Name:find("Container") then
			child:Destroy()
		end
	end

	local flyToggle = createToggle("Activer Vol", controlContainer, Config.Fly.Enabled)
	flyToggle.Container.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			toggleFly()
			flyToggle.Set(Config.Fly.Enabled)
		end
	end)

	local speedSlider = createSlider("Vitesse", controlContainer, 10, Config.Fly.MaxSpeed, Config.Fly.Speed, 10)
	speedSlider.Container.Changed:Connect(function()
		Config.Fly.Speed = speedSlider.Get()
	end)

	local noClipToggle = createToggle("NoClip", controlContainer, Config.Fly.NoClip)
	noClipToggle.Container.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			Config.Fly.NoClip = not Config.Fly.NoClip
			noClipToggle.Set(Config.Fly.NoClip)
		end
	end)
end)

local speedButton = createButton("💨 Vitesse", buttonContainer, "⚡")
speedButton.MouseButton1Click:Connect(function()
	showMenu("sub")
	subTitle.Text = "Vitesse - Options"

	-- Nettoyer le conteneur
	for _, child in pairs(controlContainer:GetChildren()) do
		if child:IsA("Frame") and child.Name:find("Container") then
			child:Destroy()
		end
	end

	local speedToggle = createToggle("Activer Vitesse", controlContainer, Config.Speed.Enabled)
	speedToggle.Container.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			toggleSpeed()
			speedToggle.Set(Config.Speed.Enabled)
		end
	end)

	local speedSlider = createSlider("Vitesse", controlContainer, 16, Config.Speed.MaxValue, Config.Speed.Value, 1)
	speedSlider.Container.Changed:Connect(function()
		Config.Speed.Value = speedSlider.Get()
		if Config.Speed.Enabled then
			local character = LocalPlayer.Character
			if character and character:FindFirstChild("Humanoid") then
				character.Humanoid.WalkSpeed = Config.Speed.Value
			end
		end
	end)

	local jumpSlider = createSlider("Force Saut", controlContainer, 50, 200, Config.Speed.JumpPower, 5)
	jumpSlider.Container.Changed:Connect(function()
		Config.Speed.JumpPower = jumpSlider.Get()
		if Config.Speed.Enabled then
			local character = LocalPlayer.Character
			if character and character:FindFirstChild("Humanoid") then
				character.Humanoid.JumpPower = Config.Speed.JumpPower
			end
		end
	end)
end)

local xrayButton = createButton("👁️ X-Ray", buttonContainer, "🔍")
xrayButton.MouseButton1Click:Connect(function()
	showMenu("sub")
	subTitle.Text = "X-Ray - Options"

	-- Nettoyer le conteneur
	for _, child in pairs(controlContainer:GetChildren()) do
		if child:IsA("Frame") and child.Name:find("Container") then
			child:Destroy()
		end
	end

	local xrayToggle = createToggle("Activer X-Ray", controlContainer, Config.XRay.Enabled)
	xrayToggle.Container.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			toggleXRay()
			xrayToggle.Set(Config.XRay.Enabled)
		end
	end)

	local transparencySlider = createSlider("Transparence", controlContainer, 0.1, 0.9, Config.XRay.Transparency, 0.1)
	transparencySlider.Container.Changed:Connect(function()
		Config.XRay.Transparency = transparencySlider.Get()
	end)

	local excludeToggle = createToggle("Exclure Joueurs", controlContainer, Config.XRay.ExcludeCharacters)
	excludeToggle.Container.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			Config.XRay.ExcludeCharacters = not Config.XRay.ExcludeCharacters
			excludeToggle.Set(Config.XRay.ExcludeCharacters)
		end
	end)
end)

local godModeButton = createButton("🛡️ God Mode", buttonContainer, "⚡")
godModeButton.MouseButton1Click:Connect(function()
	showMenu("sub")
	subTitle.Text = "God Mode - Options"

	-- Nettoyer le conteneur
	for _, child in pairs(controlContainer:GetChildren()) do
		if child:IsA("Frame") and child.Name:find("Container") then
			child:Destroy()
		end
	end

	local godToggle = createToggle("Activer God Mode", controlContainer, Config.GodMode.Enabled)
	godToggle.Container.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			toggleGodMode()
			godToggle.Set(Config.GodMode.Enabled)
		end
	end)

	local healToggle = createToggle("Auto Heal", controlContainer, Config.GodMode.AutoHeal)
	healToggle.Container.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			Config.GodMode.AutoHeal = not Config.GodMode.AutoHeal
			healToggle.Set(Config.GodMode.AutoHeal)
		end
	end)

	local jumpToggle = createToggle("Saut Infini", controlContainer, Config.GodMode.InfiniteJump)
	jumpToggle.Container.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			Config.GodMode.InfiniteJump = not Config.GodMode.InfiniteJump
			jumpToggle.Set(Config.GodMode.InfiniteJump)
		end
	end)
end)

local espButton = createButton("📍 ESP", buttonContainer, "👁️")
espButton.MouseButton1Click:Connect(function()
	showMenu("sub")
	subTitle.Text = "ESP - Options"

	-- Nettoyer le conteneur
	for _, child in pairs(controlContainer:GetChildren()) do
		if child:IsA("Frame") and child.Name:find("Container") then
			child:Destroy()
		end
	end

	local espToggle = createToggle("Activer ESP", controlContainer, Config.ESP.Enabled)
	espToggle.Container.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			toggleESP()
			espToggle.Set(Config.ESP.Enabled)
		end
	end)

	local namesToggle = createToggle("Afficher Noms", controlContainer, Config.ESP.ShowNames)
	namesToggle.Container.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			Config.ESP.ShowNames = not Config.ESP.ShowNames
			namesToggle.Set(Config.ESP.ShowNames)
		end
	end)

	local distanceToggle = createToggle("Afficher Distance", controlContainer, Config.ESP.ShowDistance)
	distanceToggle.Container.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			Config.ESP.ShowDistance = not Config.ESP.ShowDistance
			distanceToggle.Set(Config.ESP.ShowDistance)
		end
	end)

	local healthToggle = createToggle("Afficher Santé", controlContainer, Config.ESP.ShowHealth)
	healthToggle.Container.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			Config.ESP.ShowHealth = not Config.ESP.ShowHealth
			healthToggle.Set(Config.ESP.ShowHealth)
		end
	end)
end)

local teleportButton = createButton("🌍 Téléportation", buttonContainer, "📍")
teleportButton.MouseButton1Click:Connect(function()
	showMenu("sub")
	subTitle.Text = "Téléportation - Options"

	-- Nettoyer le conteneur
	for _, child in pairs(controlContainer:GetChildren()) do
		if child:IsA("Frame") and child.Name:find("Container") then
			child:Destroy()
		end
	end

	local clickTPToggle = createToggle("Click TP (Ctrl+Clic)", controlContainer, Config.Teleport.ClickTP)
	clickTPToggle.Container.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			toggleClickTP()
			clickTPToggle.Set(Config.Teleport.ClickTP)
		end
	end)

	-- Bouton pour téléporter vers spawn
	local spawnButton = createButton("Aller au Spawn", controlContainer)
	spawnButton.MouseButton1Click:Connect(function()
		local character = LocalPlayer.Character
		if character and character:FindFirstChild("HumanoidRootPart") then
			character.HumanoidRootPart.CFrame = CFrame.new(0, 10, 0)
			notify("Téléporté au spawn!")
		end
	end)
end)

local customizeButton = createButton("🎨 Personnaliser", buttonContainer, "⚙️")
customizeButton.MouseButton1Click:Connect(function()
	showMenu("customize")

	-- Nettoyer le conteneur
	for _, child in pairs(customizeContainer:GetChildren()) do
		if child:IsA("Frame") and child.Name:find("Container") then
			child:Destroy()
		end
	end

	-- Couleurs du thème
	local mainColorPicker = createColorPicker("Couleur Principale", customizeContainer, Config.UI.MainColor)
	local secondaryColorPicker = createColorPicker("Couleur Secondaire", customizeContainer, Config.UI.SecondaryColor)
	local accentColorPicker = createColorPicker("Couleur Accent", customizeContainer, Config.UI.AccentColor)

	-- Transparence
	local transparencySlider = createSlider("Transparence", customizeContainer, 0, 0.8, Config.UI.Transparency, 0.1)
	transparencySlider.Container.Changed:Connect(function()
		Config.UI.Transparency = transparencySlider.Get()
		mainMenu.BackgroundTransparency = Config.UI.Transparency
		subMenu.BackgroundTransparency = Config.UI.Transparency
		customizeMenu.BackgroundTransparency = Config.UI.Transparency
	end)

	-- Coins arrondis
	local roundedToggle = createToggle("Coins Arrondis", customizeContainer, Config.UI.Rounded)
	roundedToggle.Container.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			Config.UI.Rounded = not Config.UI.Rounded
			roundedToggle.Set(Config.UI.Rounded)
			notify("Redémarrez pour appliquer les changements")
		end
	end)
end)

-- Gestion des touches de raccourci
UIS.InputBegan:Connect(function(input, gameProcessed)
	if gameProcessed then return end

	if input.KeyCode == Config.UI.ToggleKey then
		mainContainer.Visible = not mainContainer.Visible
		if mainContainer.Visible then
			showMenu(nil) -- Fermer les sous-menus
		end
	end
end)

-- Nettoyage lors de la reconnexion du personnage
LocalPlayer.CharacterAdded:Connect(function()
	-- Reconnecter les fonctionnalités actives
	task.wait(1) -- Attendre que le personnage se charge

	if Config.Speed.Enabled then
		local character = LocalPlayer.Character
		if character and character:FindFirstChild("Humanoid") then
			character.Humanoid.WalkSpeed = Config.Speed.Value
			character.Humanoid.JumpPower = Config.Speed.JumpPower
		end
	end
end)

-- Nettoyage lors de la fermeture
game:BindToClose(function()
	for _, connection in pairs(characterConnections) do
		if connection then
			connection:Disconnect()
		end
	end
end)

-- Message de bienvenue
notify("MEGA CHEATS chargé! Appuyez sur " .. Config.UI.ToggleKey.Name .. " pour ouvrir le menu", 5)
