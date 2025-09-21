-- ⚡ Unified Control Hub with Tabs (General + Teleport/Fling)

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "UnifiedControlHub"
screenGui.ResetOnSpawn = false
screenGui.Parent = playerGui

-- Main Frame
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 320, 0, 520)
frame.Position = UDim2.new(0.05, 0, 0.25, 0)
frame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true
frame.Parent = screenGui

Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 15)
local uiStroke = Instance.new("UIStroke")
uiStroke.Thickness = 3
uiStroke.Color = Color3.fromRGB(0, 255, 180)
uiStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
uiStroke.Parent = frame

-- Title bar
local titleBar = Instance.new("TextLabel")
titleBar.Size = UDim2.new(1, -40, 0, 40)
titleBar.BackgroundColor3 = Color3.fromRGB(30, 30, 50)
titleBar.Text = "⚡ Control Hub ⚡"
titleBar.TextColor3 = Color3.fromRGB(0, 255, 200)
titleBar.Font = Enum.Font.FredokaOne
titleBar.TextSize = 20
titleBar.Parent = frame
Instance.new("UICorner", titleBar).CornerRadius = UDim.new(0, 12)

-- Minimize Button
local minButton = Instance.new("TextButton")
minButton.Size = UDim2.new(0, 40, 0, 40)
minButton.Position = UDim2.new(1, -40, 0, 0)
minButton.BackgroundColor3 = Color3.fromRGB(50, 50, 70)
minButton.Text = "–"
minButton.TextColor3 = Color3.new(1,1,1)
minButton.Font = Enum.Font.GothamBold
minButton.TextSize = 22
minButton.Parent = frame
Instance.new("UICorner", minButton).CornerRadius = UDim.new(0, 12)

local minimized = false

-- Tab buttons
local tabFrame = Instance.new("Frame")
tabFrame.Size = UDim2.new(1, 0, 0, 35)
tabFrame.Position = UDim2.new(0, 0, 0, 40)
tabFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 40)
tabFrame.Parent = frame

local function makeTabButton(text, xPos)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(0.5, 0, 1, 0)
	btn.Position = UDim2.new(xPos, 0, 0, 0)
	btn.BackgroundColor3 = Color3.fromRGB(30, 30, 50)
	btn.Text = text
	btn.Font = Enum.Font.GothamBold
	btn.TextColor3 = Color3.fromRGB(255,255,255)
	btn.TextSize = 16
	btn.Parent = tabFrame
	return btn
end

local generalTab = makeTabButton("General", 0)
local teleportTab = makeTabButton("Teleport", 0.5)
generalTab.BackgroundColor3 = Color3.fromRGB(40, 40, 60)

-- Scrolling area
local scroll = Instance.new("ScrollingFrame")
scroll.Size = UDim2.new(1, 0, 1, -75)
scroll.Position = UDim2.new(0, 0, 0, 75)
scroll.BackgroundTransparency = 1
scroll.ScrollBarThickness = 6
scroll.CanvasSize = UDim2.new()
scroll.Parent = frame

-- General Container
local generalContainer = Instance.new("Frame")
generalContainer.Size = UDim2.new(1,0,0,0)
generalContainer.BackgroundTransparency = 1
generalContainer.Parent = scroll

local generalLayout = Instance.new("UIListLayout")
generalLayout.Parent = generalContainer
generalLayout.Padding = UDim.new(0,10)

-- Teleport Container
local teleportContainer = Instance.new("Frame")
teleportContainer.Size = UDim2.new(1,0,0,0)
teleportContainer.BackgroundTransparency = 1
teleportContainer.Parent = scroll
teleportContainer.Visible = false

local teleportLayout = Instance.new("UIListLayout")
teleportLayout.Parent = teleportContainer
teleportLayout.Padding = UDim.new(0,10)

-- Update scroll size automatically
local function updateCanvas(container)
	container.Size = UDim2.new(1,0,0,generalLayout.AbsoluteContentSize.Y)
	scroll.CanvasSize = UDim2.new(0,0,0,container.Size.Y.Offset)
end
generalLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
	updateCanvas(generalContainer)
end)
teleportLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
	updateCanvas(teleportContainer)
end)

-- === GENERAL CONTENT ===
local function createBox(parent, placeholder)
	local box = Instance.new("TextBox")
	box.PlaceholderText = placeholder
	box.Size = UDim2.new(0.9, 0, 0, 30)
	box.BackgroundColor3 = Color3.fromRGB(40,40,60)
	box.TextColor3 = Color3.new(1,1,1)
	box.Font = Enum.Font.Gotham
	box.TextSize = 16
	box.ClearTextOnFocus = false
	box.Parent = parent
	Instance.new("UICorner", box).CornerRadius = UDim.new(0,8)
	return box
end

local wsBox = createBox(generalContainer,"WalkSpeed")
local jpBox = createBox(generalContainer,"JumpPower")
local gBox  = createBox(generalContainer,"Gravity")
local hhBox = createBox(generalContainer,"HipHeight")

local toggleStates = {nightVision=false, infJump=false, noclip=false, fly=false}
local function createToggle(name,key)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(0.9,0,0,30)
	btn.BackgroundColor3 = Color3.fromRGB(80,30,30)
	btn.Text = "OFF - "..name
	btn.TextColor3 = Color3.new(1,1,1)
	btn.Font = Enum.Font.GothamBold
	btn.TextSize = 14
	btn.Parent = generalContainer
	btn.MouseButton1Click:Connect(function()
		toggleStates[key]=not toggleStates[key]
		if toggleStates[key] then
			btn.Text="ON - "..name
			btn.BackgroundColor3=Color3.fromRGB(30,80,30)
		else
			btn.Text="OFF - "..name
			btn.BackgroundColor3=Color3.fromRGB(80,30,30)
		end
	end)
end

createToggle("Night Vision","nightVision")
createToggle("Infinite Jump","infJump")
createToggle("NoClip","noclip")
createToggle("Fly","fly")

local applyBtn = Instance.new("TextButton")
applyBtn.Size = UDim2.new(0.9,0,0,35)
applyBtn.Text = "✔ ACTIVATE"
applyBtn.BackgroundColor3 = Color3.fromRGB(0,200,150)
applyBtn.TextColor3 = Color3.new(1,1,1)
applyBtn.Font = Enum.Font.GothamBold
applyBtn.TextSize = 16
applyBtn.Parent = generalContainer

local resetBtn = Instance.new("TextButton")
resetBtn.Size = UDim2.new(0.9,0,0,30)
resetBtn.Text = "🔄 RESET DEFAULTS"
resetBtn.BackgroundColor3 = Color3.fromRGB(200,50,50)
resetBtn.TextColor3 = Color3.new(1,1,1)
resetBtn.Font = Enum.Font.GothamBold
resetBtn.TextSize = 14
resetBtn.Parent = generalContainer

-- === TELEPORT CONTENT ===
local nameBox = createBox(teleportContainer,"Type player name (full or partial)")
local dropdownBtn = Instance.new("TextButton")
dropdownBtn.Size = UDim2.new(0.9,0,0,28)
dropdownBtn.Text = "Select ▼"
dropdownBtn.BackgroundColor3 = Color3.fromRGB(50,50,50)
dropdownBtn.TextColor3 = Color3.new(1,1,1)
dropdownBtn.Font = Enum.Font.Gotham
dropdownBtn.TextSize = 15
dropdownBtn.Parent = teleportContainer

local dropdownContainer = Instance.new("ScrollingFrame")
dropdownContainer.Size = UDim2.new(0.9,0,0,100)
dropdownContainer.BackgroundColor3 = Color3.fromRGB(30,30,30)
dropdownContainer.Visible = false
dropdownContainer.ScrollBarThickness = 6
dropdownContainer.Parent = teleportContainer

local dropLayout = Instance.new("UIListLayout")
dropLayout.Parent = dropdownContainer
local dropdownButtons = {}

local function updateDropdown()
	for _,btn in pairs(dropdownButtons) do btn:Destroy() end
	dropdownButtons={}
	for _,p in ipairs(Players:GetPlayers()) do
		if p~=player then
			local btn = Instance.new("TextButton")
			btn.Size = UDim2.new(1,-5,0,25)
			btn.BackgroundColor3 = Color3.fromRGB(45,45,45)
			btn.Text=p.Name
			btn.TextColor3 = Color3.new(1,1,1)
			btn.Font=Enum.Font.Gotham
			btn.TextSize=14
			btn.Parent=dropdownContainer
			btn.MouseButton1Click:Connect(function()
				nameBox.Text=p.Name
				dropdownContainer.Visible=false
			end)
			table.insert(dropdownButtons,btn)
		end
	end
	dropdownContainer.CanvasSize=UDim2.new(0,0,0,#dropdownButtons*30)
end
dropdownBtn.MouseButton1Click:Connect(function()
	updateDropdown()
	dropdownContainer.Visible=not dropdownContainer.Visible
end)

local flingBtn = Instance.new("TextButton")
flingBtn.Size = UDim2.new(0.9,0,0,35)
flingBtn.Text="▶️ Fling Once"
flingBtn.BackgroundColor3=Color3.fromRGB(70,70,70)
flingBtn.TextColor3=Color3.new(1,1,1)
flingBtn.Font=Enum.Font.GothamBold
flingBtn.TextSize=16
flingBtn.Parent=teleportContainer

local loopBtn = Instance.new("TextButton")
loopBtn.Size = UDim2.new(0.9,0,0,35)
loopBtn.Text="🔁 Loop Fling: OFF"
loopBtn.BackgroundColor3=Color3.fromRGB(70,70,70)
loopBtn.TextColor3=Color3.new(1,1,1)
loopBtn.Font=Enum.Font.GothamBold
loopBtn.TextSize=14
loopBtn.Parent=teleportContainer

-- === LOGIC ===
applyBtn.MouseButton1Click:Connect(function()
	local char=player.Character or player.CharacterAdded:Wait()
	local hum=char:WaitForChild("Humanoid")
	hum.UseJumpPower=true
	if tonumber(wsBox.Text) then hum.WalkSpeed=tonumber(wsBox.Text) end
	if tonumber(jpBox.Text) then hum.JumpPower=tonumber(jpBox.Text) end
	if tonumber(gBox.Text) then workspace.Gravity=tonumber(gBox.Text) end
	if tonumber(hhBox.Text) then hum.HipHeight=tonumber(hhBox.Text) end
end)

resetBtn.MouseButton1Click:Connect(function()
	local char=player.Character or player.CharacterAdded:Wait()
	local hum=char:WaitForChild("Humanoid")
	hum.WalkSpeed,hum.JumpPower,hum.HipHeight=16,50,2
	workspace.Gravity=196.2
	wsBox.Text,jpBox.Text,gBox.Text,hhBox.Text=""
	for k in pairs(toggleStates) do toggleStates[k]=false end
end)

-- Night Vision
RunService.RenderStepped:Connect(function()
	if toggleStates.nightVision then
		game.Lighting.Brightness=5
		game.Lighting.Ambient=Color3.new(0,1,0.5)
	else
		game.Lighting.Brightness=2
		game.Lighting.Ambient=Color3.new(0.5,0.5,0.5)
	end
end)

-- Infinite Jump
UserInputService.JumpRequest:Connect(function()
	if toggleStates.infJump then
		local char=player.Character
		if char and char:FindFirstChild("Humanoid") then
			char.Humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
		end
	end
end)

-- Noclip
RunService.Stepped:Connect(function()
	if toggleStates.noclip then
		local char=player.Character
		if char then
			for _,part in pairs(char:GetDescendants()) do
				if part:IsA("BasePart") then
					part.CanCollide=false
				end
			end
		end
	end
end)

-- Fly
local flySpeed=50
RunService.RenderStepped:Connect(function()
	if toggleStates.fly then
		local char=player.Character
		local root=char and char:FindFirstChild("HumanoidRootPart")
		if root then
			local move=Vector3.zero
			if UserInputService:IsKeyDown(Enum.KeyCode.W) then move+=workspace.CurrentCamera.CFrame.LookVector end
			if UserInputService:IsKeyDown(Enum.KeyCode.S) then move-=workspace.CurrentCamera.CFrame.LookVector end
			if UserInputService:IsKeyDown(Enum.KeyCode.A) then move-=workspace.CurrentCamera.CFrame.RightVector end
			if UserInputService:IsKeyDown(Enum.KeyCode.D) then move+=workspace.CurrentCamera.CFrame.RightVector end
			if UserInputService:IsKeyDown(Enum.KeyCode.Space) then move+=Vector3.new(0,1,0) end
			if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then move-=Vector3.new(0,1,0) end
			root.Velocity=move*flySpeed
		end
	end
end)

-- Fling system
local flingSpin
local looping=false
local function startFling()
	if flingSpin then flingSpin:Destroy() end
	local char=player.Character
	if not char then return end
	local hrp=char:FindFirstChild("HumanoidRootPart")
	if not hrp then return end
	flingSpin=Instance.new("BodyAngularVelocity")
	flingSpin.AngularVelocity=Vector3.new(0,999999,0)
	flingSpin.MaxTorque=Vector3.new(999999,999999,999999)
	flingSpin.P=10000
	flingSpin.Parent=hrp
end
local function stopFling()
	if flingSpin then flingSpin:Destroy() flingSpin=nil end
end
local function findTarget(name)
	local nameLower=name:lower()
	for _,p in pairs(Players:GetPlayers()) do
		if p~=player and p.Name:lower():sub(1,#nameLower)==nameLower then
			return p
		end
	end
end
local function flingOnce(target)
	if not target or not target.Character or not target.Character:FindFirstChild("HumanoidRootPart") then return end
	local hrp=player.Character and player.Character:FindFirstChild("HumanoidRootPart")
	if hrp then
		hrp.CFrame=target.Character.HumanoidRootPart.CFrame+Vector3.new(2,0,0)
		startFling()
	end
end
flingBtn.MouseButton1Click:Connect(function()
	local t=findTarget(nameBox.Text)
	if t then flingOnce(t) else warn("Player not found") end
end)
loopBtn.MouseButton1Click:Connect(function()
	looping=not looping
	loopBtn.Text=looping and "🔁 Loop Fling: ON" or "🔁 Loop Fling: OFF"
	if not looping then stopFling() end
end)
RunService.Heartbeat:Connect(function()
	if looping then
		local t=findTarget(nameBox.Text)
		if t and t.Character and t.Character:FindFirstChild("HumanoidRootPart") then
			local hrp=player.Character and player.Character:FindFirstChild("HumanoidRootPart")
			if hrp then
				hrp.CFrame=t.Character.HumanoidRootPart.CFrame+Vector3.new(2,0,0)
				startFling()
			end
		else stopFling() end
	end
end)

-- Tab switching
generalTab.MouseButton1Click:Connect(function()
	generalContainer.Visible=true
	teleportContainer.Visible=false
	generalTab.BackgroundColor3=Color3.fromRGB(40,40,60)
	teleportTab.BackgroundColor3=Color3.fromRGB(30,30,50)
	updateCanvas(generalContainer)
end)
teleportTab.MouseButton1Click:Connect(function()
	generalContainer.Visible=false
	teleportContainer.Visible=true
	generalTab.BackgroundColor3=Color3.fromRGB(30,30,50)
	teleportTab.BackgroundColor3=Color3.fromRGB(40,40,60)
	updateCanvas(teleportContainer)
end)

-- Minimize
minButton.MouseButton1Click:Connect(function()
	if minimized then
		frame.Size=UDim2.new(0,320,0,520)
		minButton.Text="–"
		scroll.Visible=true
		tabFrame.Visible=true
	else
		frame.Size=UDim2.new(0,320,0,40)
		minButton.Text="+"
		scroll.Visible=false
		tabFrame.Visible=false
	end
	minimized=not minimized
end)
