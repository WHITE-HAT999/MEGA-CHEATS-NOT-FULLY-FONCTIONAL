-- LocalScript inside StarterPlayer > StarterPlayerScripts

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "PlayerControlGUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = playerGui

-- Main Frame
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 280, 0, 440)
frame.Position = UDim2.new(0.05, 0, 0.25, 0)
frame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true
frame.Parent = screenGui

local uiCorner = Instance.new("UICorner")
uiCorner.CornerRadius = UDim.new(0, 15)
uiCorner.Parent = frame

local uiStroke = Instance.new("UIStroke")
uiStroke.Thickness = 3
uiStroke.Color = Color3.fromRGB(0, 255, 180)
uiStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
uiStroke.Parent = frame

-- Title bar
local titleBar = Instance.new("TextLabel")
titleBar.Size = UDim2.new(1, -40, 0, 40)
titleBar.Position = UDim2.new(0, 0, 0, 0)
titleBar.BackgroundColor3 = Color3.fromRGB(30, 30, 50)
titleBar.Text = "⚡ Control Hub ⚡"
titleBar.TextColor3 = Color3.fromRGB(0, 255, 200)
titleBar.Font = Enum.Font.FredokaOne
titleBar.TextSize = 20
titleBar.Parent = frame

local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 12)
titleCorner.Parent = titleBar

-- Minimize Button
local minButton = Instance.new("TextButton")
minButton.Size = UDim2.new(0, 40, 0, 40)
minButton.Position = UDim2.new(1, -40, 0, 0)
minButton.BackgroundColor3 = Color3.fromRGB(50, 50, 70)
minButton.Text = "–"
minButton.TextColor3 = Color3.fromRGB(255, 255, 255)
minButton.Font = Enum.Font.GothamBold
minButton.TextSize = 22
minButton.Parent = frame

local minCorner = Instance.new("UICorner")
minCorner.CornerRadius = UDim.new(0, 12)
minCorner.Parent = minButton

local minimized = false

-- Function to create input boxes
local function createBox(placeholder, yPos)
	local box = Instance.new("TextBox")
	box.PlaceholderText = placeholder
	box.Text = ""
	box.Size = UDim2.new(0.85, 0, 0, 30)
	box.Position = UDim2.new(0.075, 0, 0, yPos)
	box.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
	box.TextColor3 = Color3.fromRGB(255, 255, 255)
	box.Font = Enum.Font.Gotham
	box.TextSize = 16
	box.ClearTextOnFocus = false

	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 8)
	corner.Parent = box

	local stroke = Instance.new("UIStroke")
	stroke.Color = Color3.fromRGB(0, 200, 255)
	stroke.Thickness = 1.5
	stroke.Parent = box

	box.Parent = frame
	return box
end

-- Input boxes
local wsBox = createBox("WalkSpeed", 50)
local jpBox = createBox("JumpPower", 90)
local gBox  = createBox("Gravity", 130)
local hhBox = createBox("HipHeight", 170)

-- Feature Toggles
local toggles = {
	{ name = "Night Vision", key = "nightVision", state = false, y = 210 },
	{ name = "Infinite Jump", key = "infJump", state = false, y = 245 },
	{ name = "NoClip", key = "noclip", state = false, y = 280 },
	{ name = "Fly", key = "fly", state = false, y = 315 },
}

local toggleStates = {}

for _, data in pairs(toggles) do
	local button = Instance.new("TextButton")
	button.Text = "OFF - " .. data.name
	button.Size = UDim2.new(0.85, 0, 0, 25)
	button.Position = UDim2.new(0.075, 0, 0, data.y)
	button.BackgroundColor3 = Color3.fromRGB(80, 30, 30)
	button.TextColor3 = Color3.fromRGB(255, 255, 255)
	button.Font = Enum.Font.GothamBold
	button.TextSize = 14
	button.Parent = frame

	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 8)
	corner.Parent = button

	toggleStates[data.key] = false

	button.MouseButton1Click:Connect(function()
		toggleStates[data.key] = not toggleStates[data.key]
		if toggleStates[data.key] then
			button.Text = "ON - " .. data.name
			button.BackgroundColor3 = Color3.fromRGB(30, 80, 30)
		else
			button.Text = "OFF - " .. data.name
			button.BackgroundColor3 = Color3.fromRGB(80, 30, 30)
		end
	end)
end

-- Dropdown for teleport
local tpDropdown = Instance.new("TextButton")
tpDropdown.Size = UDim2.new(0.85, 0, 0, 30)
tpDropdown.Position = UDim2.new(0.075, 0, 0, 350)
tpDropdown.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
tpDropdown.TextColor3 = Color3.fromRGB(255, 255, 255)
tpDropdown.Font = Enum.Font.GothamBold
tpDropdown.TextSize = 14
tpDropdown.Text = "Select Player"
tpDropdown.Parent = frame

local tpCorner = Instance.new("UICorner")
tpCorner.CornerRadius = UDim.new(0, 8)
tpCorner.Parent = tpDropdown

local selectedPlayer = nil

-- Dropdown logic
tpDropdown.MouseButton1Click:Connect(function()
	local dropdownFrame = Instance.new("Frame")
	dropdownFrame.Size = UDim2.new(0.85, 0, 0, 120)
	dropdownFrame.Position = UDim2.new(0.075, 0, 0, 380)
	dropdownFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 50)
	dropdownFrame.Parent = frame

	local listLayout = Instance.new("UIListLayout")
	listLayout.Parent = dropdownFrame

	for _, plr in pairs(Players:GetPlayers()) do
		if plr ~= player then
			local option = Instance.new("TextButton")
			option.Size = UDim2.new(1, 0, 0, 30)
			option.BackgroundColor3 = Color3.fromRGB(50, 50, 70)
			option.TextColor3 = Color3.fromRGB(255, 255, 255)
			option.Text = plr.Name
			option.Font = Enum.Font.Gotham
			option.TextSize = 14
			option.Parent = dropdownFrame

			option.MouseButton1Click:Connect(function()
				selectedPlayer = plr
				tpDropdown.Text = "Target: " .. plr.Name
				dropdownFrame:Destroy()
			end)
		end
	end
end)

-- Teleport button
local tpButton = Instance.new("TextButton")
tpButton.Text = "Teleport"
tpButton.Size = UDim2.new(0.85, 0, 0, 30)
tpButton.Position = UDim2.new(0.075, 0, 0, 410)
tpButton.BackgroundColor3 = Color3.fromRGB(0, 120, 200)
tpButton.TextColor3 = Color3.fromRGB(255, 255, 255)
tpButton.Font = Enum.Font.GothamBold
tpButton.TextSize = 16
tpButton.Parent = frame

local tpCorner2 = Instance.new("UICorner")
tpCorner2.CornerRadius = UDim.new(0, 8)
tpCorner2.Parent = tpButton

tpButton.MouseButton1Click:Connect(function()
	if selectedPlayer and selectedPlayer.Character and selectedPlayer.Character:FindFirstChild("HumanoidRootPart") then
		local character = player.Character or player.CharacterAdded:Wait()
		local root = character:FindFirstChild("HumanoidRootPart")
		if root then
			root.CFrame = selectedPlayer.Character.HumanoidRootPart.CFrame + Vector3.new(2,0,2)
		end
	end
end)

-- Apply Button
local applyButton = Instance.new("TextButton")
applyButton.Text = "✔ APPLY"
applyButton.Size = UDim2.new(0.85, 0, 0, 35)
applyButton.Position = UDim2.new(0.075, 0, 0, 450)
applyButton.BackgroundColor3 = Color3.fromRGB(0, 200, 150)
applyButton.TextColor3 = Color3.fromRGB(255, 255, 255)
applyButton.Font = Enum.Font.GothamBold
applyButton.TextSize = 18
applyButton.Parent = frame

local applyCorner = Instance.new("UICorner")
applyCorner.CornerRadius = UDim.new(0, 10)
applyCorner.Parent = applyButton

-- Reset Button
local resetButton = Instance.new("TextButton")
resetButton.Text = "🔄 RESET DEFAULTS"
resetButton.Size = UDim2.new(0.85, 0, 0, 30)
resetButton.Position = UDim2.new(0.075, 0, 0, 490)
resetButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
resetButton.TextColor3 = Color3.fromRGB(255, 255, 255)
resetButton.Font = Enum.Font.GothamBold
resetButton.TextSize = 16
resetButton.Parent = frame

local resetCorner = Instance.new("UICorner")
resetCorner.CornerRadius = UDim.new(0, 10)
resetCorner.Parent = resetButton

-- APPLY SETTINGS
applyButton.MouseButton1Click:Connect(function()
	local character = player.Character or player.CharacterAdded:Wait()
	local humanoid = character:WaitForChild("Humanoid")

	humanoid.UseJumpPower = true

	local ws = tonumber(wsBox.Text)
	if ws then humanoid.WalkSpeed = ws end

	local jp = tonumber(jpBox.Text)
	if jp then humanoid.JumpPower = jp end

	local g = tonumber(gBox.Text)
	if g then workspace.Gravity = g end

	local hh = tonumber(hhBox.Text)
	if hh then humanoid.HipHeight = hh end
end)

-- RESET DEFAULTS
resetButton.MouseButton1Click:Connect(function()
	local character = player.Character or player.CharacterAdded:Wait()
	local humanoid = character:WaitForChild("Humanoid")

	humanoid.WalkSpeed = 16
	humanoid.JumpPower = 50
	humanoid.HipHeight = 2
	workspace.Gravity = 196.2

	wsBox.Text, jpBox.Text, gBox.Text, hhBox.Text = "", "", "", ""
	selectedPlayer = nil
	tpDropdown.Text = "Select Player"

	for key, _ in pairs(toggleStates) do toggleStates[key] = false end
end)

-- FEATURE LOGIC
-- Night Vision
RunService.RenderStepped:Connect(function()
	if toggleStates.nightVision then
		game.Lighting.Brightness = 5
		game.Lighting.Ambient = Color3.new(0, 1, 0.5)
	else
		game.Lighting.Brightness = 2
		game.Lighting.Ambient = Color3.new(0.5, 0.5, 0.5)
	end
end)

-- Infinite Jump
UserInputService.JumpRequest:Connect(function()
	if toggleStates.infJump then
		local character = player.Character
		if character and character:FindFirstChild("Humanoid") then
			character.Humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
		end
	end
end)

-- NoClip
RunService.Stepped:Connect(function()
	local character = player.Character
	if character then
		for _, part in pairs(character:GetDescendants()) do
			if part:IsA("BasePart") then
				part.CanCollide = not toggleStates.noclip
			end
		end
	end
end)

-- Fly System
local flySpeed = 50
local flying = false

RunService.RenderStepped:Connect(function()
	if toggleStates.fly then
		local character = player.Character
		local root = character and character:FindFirstChild("HumanoidRootPart")
		if root then
			flying = true
			local move = Vector3.zero
			if UserInputService:IsKeyDown(Enum.KeyCode.W) then
				move = move + (workspace.CurrentCamera.CFrame.LookVector)
			end
			if UserInputService:IsKeyDown(Enum.KeyCode.S) then
				move = move - (workspace.CurrentCamera.CFrame.LookVector)
			end
			if UserInputService:IsKeyDown(Enum.KeyCode.A) then
				move = move - (workspace.CurrentCamera.CFrame.RightVector)
			end
			if UserInputService:IsKeyDown(Enum.KeyCode.D) then
				move = move + (workspace.CurrentCamera.CFrame.RightVector)
			end
			if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
				move = move + Vector3.new(0,1,0)
			end
			if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
				move = move - Vector3.new(0,1,0)
			end
			root.Velocity = move * flySpeed
		end
	else
		flying = false
	end
end)

-- Minimize/maximize
minButton.MouseButton1Click:Connect(function()
	if minimized then
		frame.Size = UDim2.new(0, 280, 0, 520)
		minButton.Text = "–"
		for _, child in pairs(frame:GetChildren()) do
			if child ~= titleBar and child ~= minButton then
				child.Visible = true
			end
		end
	else
		frame.Size = UDim2.new(0, 280, 0, 40)
		minButton.Text = "+"
		for _, child in pairs(frame:GetChildren()) do
			if child ~= titleBar and child ~= minButton then
				child.Visible = false
			end
		end
	end
	minimized = not minimized
end)
