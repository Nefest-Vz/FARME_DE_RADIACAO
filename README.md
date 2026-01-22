-- Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local collecting = false
local connection
local minimized = false

-- GUI
local gui = Instance.new("ScreenGui")
gui.Name = "AutoCollectGui"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

-- Main Frame
local main = Instance.new("Frame")
main.Size = UDim2.new(0, 260, 0, 140)
main.Position = UDim2.new(0.5, -130, 0.5, -70)
main.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
main.Active = true
main.Parent = gui

-- Rounded corners
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 12)
corner.Parent = main

-- RGB Border
local stroke = Instance.new("UIStroke")
stroke.Thickness = 2
stroke.Parent = main

-- Title
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -40, 0, 30)
title.Position = UDim2.new(0, 10, 0, 5)
title.BackgroundTransparency = 1
title.Text = "AUTO COLLECT"
title.TextColor3 = Color3.fromRGB(0, 255, 0)
title.TextScaled = true
title.Font = Enum.Font.GothamBold
title.Parent = main

-- Minimize Button
local minBtn = Instance.new("TextButton")
minBtn.Size = UDim2.new(0, 30, 0, 30)
minBtn.Position = UDim2.new(1, -35, 0, 5)
minBtn.Text = "-"
minBtn.TextScaled = true
minBtn.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
minBtn.TextColor3 = Color3.fromRGB(0, 255, 0)
minBtn.Parent = main

local minCorner = Instance.new("UICorner")
minCorner.CornerRadius = UDim.new(0, 8)
minCorner.Parent = minBtn

-- Collect Button
local button = Instance.new("TextButton")
button.Size = UDim2.new(1, -20, 0, 45)
button.Position = UDim2.new(0, 10, 0, 50)
button.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
button.TextColor3 = Color3.fromRGB(255, 255, 255)
button.TextScaled = true
button.Text = "Auto Collect: OFF"
button.Font = Enum.Font.GothamBold
button.Parent = main

local btnCorner = Instance.new("UICorner")
btnCorner.CornerRadius = UDim.new(0, 10)
btnCorner.Parent = button

-- Drag (Mobile + PC)
local dragging, dragStart, startPos

main.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = true
		dragStart = input.Position
		startPos = main.Position
	end
end)

main.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = false
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if dragging and (input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseMovement) then
		local delta = input.Position - dragStart
		main.Position = UDim2.new(
			startPos.X.Scale,
			startPos.X.Offset + delta.X,
			startPos.Y.Scale,
			startPos.Y.Offset + delta.Y
		)
	end
end)

-- RGB Border Animation
local hue = 0
RunService.RenderStepped:Connect(function()
	hue = (hue + 1) % 360
	stroke.Color = Color3.fromHSV(hue / 360, 1, 1)
end)

-- Coin teleport function (INTACTA)
local function teleportCoins()
	local character = player.Character
	if not character then return end

	local hrp = character:FindFirstChild("HumanoidRootPart")
	if not hrp then return end

	local eventParts = workspace:FindFirstChild("EventParts")
	if not eventParts then return end

	for _, coin in ipairs(eventParts:GetChildren()) do
		if coin:IsA("BasePart") then
			coin.CFrame = hrp.CFrame
		elseif coin:IsA("Model") then
			if not coin.PrimaryPart then
				local part = coin:FindFirstChildWhichIsA("BasePart")
				if part then
					coin.PrimaryPart = part
				end
			end
			if coin.PrimaryPart then
				coin:SetPrimaryPartCFrame(hrp.CFrame)
			end
		end
	end
end

-- Toggle collect
button.MouseButton1Click:Connect(function()
	collecting = not collecting

	if collecting then
		button.Text = "Auto Collect: ON"
		button.TextColor3 = Color3.fromRGB(0, 255, 0)

		connection = RunService.Heartbeat:Connect(teleportCoins)
	else
		button.Text = "Auto Collect: OFF"
		button.TextColor3 = Color3.fromRGB(255, 255, 255)

		if connection then
			connection:Disconnect()
			connection = nil
		end
	end
end)

-- Minimize
minBtn.MouseButton1Click:Connect(function()
	minimized = not minimized
	button.Visible = not minimized
	main.Size = minimized and UDim2.new(0, 260, 0, 40) or UDim2.new(0, 260, 0, 140)
	minBtn.Text = minimized and "+" or "-"
end)
