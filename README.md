local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer

local function createToggleGUI()
	local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
	gui.Name = "FlyToggleGUI"
	gui.ResetOnSpawn = false

	local frame = Instance.new("Frame", gui)
	frame.Size = UDim2.new(0, 150, 0, 50)
	frame.Position = UDim2.new(0.5, -75, 0, 10)
	frame.AnchorPoint = Vector2.new(0.5, 0)
	frame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
	frame.BorderSizePixel = 0

	local label = Instance.new("TextLabel", frame)
	label.Size = UDim2.new(1, 0, 0.5, 0)
	label.Text = "Jump to Fly:"
	label.TextColor3 = Color3.new(1, 1, 1)
	label.BackgroundTransparency = 1
	label.TextScaled = true

	local button = Instance.new("TextButton", frame)
	button.Name = "ToggleButton"
	button.Size = UDim2.new(0.8, 0, 0.5, -5)
	button.Position = UDim2.new(0.1, 0, 0.5, 0)
	button.Text = "OFF"
	button.TextColor3 = Color3.new(1, 1, 1)
	button.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
	button.TextScaled = true

	return button
end

local function setupFly(toggleButton)
	local enabled = false
	local flying = false
	local velocity, gyro
	local humanoidRootPart = nil

	local function updateButton()
		toggleButton.Text = enabled and "ON" or "OFF"
		toggleButton.BackgroundColor3 = enabled and Color3.fromRGB(50, 255, 50) or Color3.fromRGB(255, 50, 50)
	end

	local function startFly()
		if not humanoidRootPart then return end
		if flying then return end
		flying = true

		velocity = Instance.new("BodyVelocity")
		velocity.MaxForce = Vector3.new(1e5, 1e5, 1e5)
		velocity.Velocity = Vector3.zero
		velocity.Parent = humanoidRootPart

		gyro = Instance.new("BodyGyro")
		gyro.MaxTorque = Vector3.new(1e5, 1e5, 1e5)
		gyro.CFrame = humanoidRootPart.CFrame
		gyro.P = 5000
		gyro.Parent = humanoidRootPart

		RunService:BindToRenderStep("FlyControl", Enum.RenderPriority.Input.Value, function()
			local moveDir = Vector3.zero
			if UserInputService:IsKeyDown(Enum.KeyCode.W) then moveDir += workspace.CurrentCamera.CFrame.LookVector end
			if UserInputService:IsKeyDown(Enum.KeyCode.S) then moveDir -= workspace.CurrentCamera.CFrame.LookVector end
			if UserInputService:IsKeyDown(Enum.KeyCode.A) then moveDir -= workspace.CurrentCamera.CFrame.RightVector end
			if UserInputService:IsKeyDown(Enum.KeyCode.D) then moveDir += workspace.CurrentCamera.CFrame.RightVector end

			velocity.Velocity = moveDir.Unit * 50
			if moveDir == Vector3.zero then
				velocity.Velocity = Vector3.zero
			end
			gyro.CFrame = workspace.CurrentCamera.CFrame
		end)
	end

	local function stopFly()
		if not flying then return end
		flying = false
		RunService:UnbindFromRenderStep("FlyControl")
		if velocity then velocity:Destroy() end
		if gyro then gyro:Destroy() end
	end

	toggleButton.MouseButton1Click:Connect(function()
		enabled = not enabled
		updateButton()
		if not enabled and flying then
			stopFly()
		end
	end)

	UserInputService.JumpRequest:Connect(function()
		if not enabled then return end
		if not humanoidRootPart then return end
		if flying then
			stopFly()
		else
			startFly()
		end
	end)

	player.CharacterAdded:Connect(function(char)
		humanoidRootPart = char:WaitForChild("HumanoidRootPart")
	end)

	if player.Character then
		humanoidRootPart = player.Character:FindFirstChild("HumanoidRootPart")
	end

	updateButton()
end

local button = createToggleGUI()
setupFly(button)
