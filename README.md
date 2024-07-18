-- Ensure the text variable is set in the global environment
getgenv().NotificationText = "Type /console to see the list of word ;)"

-- Create the ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")

-- Create the Frame (Notification Box)
local notificationBox = Instance.new("Frame")
notificationBox.Size = UDim2.new(0, 300, 0, 100) -- Adjust size as needed
notificationBox.Position = UDim2.new(0.5, -150, 0, -100) -- Start off-screen at the top
notificationBox.BackgroundColor3 = Color3.fromRGB(0, 0, 0) -- Black color
notificationBox.BorderSizePixel = 0
notificationBox.Parent = screenGui

-- Create the TextLabel (Notification Text)
local notificationText = Instance.new("TextLabel")
notificationText.Size = UDim2.new(1, 0, 1, 0)
notificationText.Position = UDim2.new(0, 0, 0, 0)
notificationText.BackgroundTransparency = 1
notificationText.TextColor3 = Color3.fromRGB(255, 255, 255) -- White text
notificationText.TextScaled = true
notificationText.Text = getgenv().NotificationText -- Set text from getgenv
notificationText.Parent = notificationBox

-- TweenService for animations
local TweenService = game:GetService("TweenService")

-- Tweening function
local function tweenPosition(object, endPosition, duration, easingStyle, easingDirection)
    local tweenInfo = TweenInfo.new(duration, easingStyle, easingDirection)
    local goal = {Position = endPosition}
    local tween = TweenService:Create(object, tweenInfo, goal)
    tween:Play()
    tween.Completed:Wait() -- Wait for the tween to complete
end

-- Animate the notification
local function showNotification()
    -- Animate in from top to middle
    tweenPosition(notificationBox, UDim2.new(0.5, -150, 0.5, -50), 0.5, Enum.EasingStyle.Sine, Enum.EasingDirection.Out)
    wait(2) -- Wait for 1 second
    -- Animate out to bottom
    tweenPosition(notificationBox, UDim2.new(0.5, -150, 1, 100), 0.5, Enum.EasingStyle.Sine, Enum.EasingDirection.In)
    wait(0.5) -- Wait for the animation to finish
    notificationBox:Destroy() -- Remove the notification
end

-- Call the function to show the notification
showNotification()

-- Create Screen GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")
screenGui.Name = "Neptune Bypasser manual (made by dabu showcase in yt)"

-- Create Frame
local frame = Instance.new("Frame")
frame.Parent = screenGui
frame.Size = UDim2.new(0, 400, 0, 200)
frame.Position = UDim2.new(0.5, -200, 0.5, -100)
frame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
frame.BorderSizePixel = 0

-- Create Title
local title = Instance.new("TextLabel")
title.Parent = frame
title.Size = UDim2.new(1, 0, 0, 50)
title.Position = UDim2.new(0, 0, 0, 0)
title.Text = "Neptune bypasser manual (made by dabu showcase don't skid)"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
title.BorderSizePixel = 0
title.Font = Enum.Font.GothamBold
title.TextSize = 20

-- Create TextBox
local textBox = Instance.new("TextBox")
textBox.Parent = frame
textBox.Size = UDim2.new(0.8, 0, 0.3, 0)
textBox.Position = UDim2.new(0.1, 0, 0.35, 0)
textBox.PlaceholderText = "Enter a word..."
textBox.TextColor3 = Color3.fromRGB(0, 0, 0)
textBox.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
textBox.BorderSizePixel = 0
textBox.Font = Enum.Font.Gotham
textBox.TextSize = 18

-- Create Button
local button = Instance.new("TextButton")
button.Parent = frame
button.Size = UDim2.new(0.8, 0, 0.2, 0)
button.Position = UDim2.new(0.1, 0, 0.7, 0)
button.Text = "Autocorrect"
button.TextColor3 = Color3.fromRGB(255, 255, 255)
button.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
button.BorderSizePixel = 0
button.Font = Enum.Font.GothamBold
button.TextSize = 18

-- Make the GUI Movable
local dragging
local dragInput
local dragStart
local startPos

local function update(input)
	local delta = input.Position - dragStart
	frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

frame.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		dragging = true
		dragStart = input.Position
		startPos = frame.Position
		
		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then
				dragging = false
			end
		end)
	end
end)

frame.InputChanged:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
		dragInput = input
	end
end)

game:GetService("UserInputService").InputChanged:Connect(function(input)
	if input == dragInput and dragging then
		update(input)
	end
end)

-- Autocorrect Functionality
local corrections = {
	fuck = "ғ̲ᴜ̲ᴄ̲ᴋ̲",
	cock = "ᴄ̲ᴏ̲ᴄ̲ᴋ̲",
	nigga = "ɴ̲ɪ̲ɢ̲ɢ̲ᴀ̲",
	femboy = "ғ̲ᴇ̲ᴍ̲ʙ̲ᴏ̲ʏ̲",
	gay = "ɢ̲ᴀ̲ʏ̲",
	porn = "ᴘ̲ᴏ̲ʀ̲ɴ̲",
	pedophile= "ᴘ̲ᴇ̲ᴅ̲ᴏ̲ᴘ̲ʜ̲ɪ̲ʟ̲ᴇ̲",
	shit = "s̲ʜ̲ɪ̲ᴛ̲",
	cum = "ᴄ̲ᴜ̲ᴍ̲",
	bitch= "ʙ̲ɪ̲ᴛ̲ᴄ̲ʜ̲",
	pussy = "ᴘ̲ᴜ̲s̲s̲ʏ̲",
	sex = "s̲ᴇ⁥⁥⁥⁥⁥⁥⁥⁥⁥⁥⁥⁥⁥⁥⁥̲x̲",
	slut = "s̲ʟ̲ᴜ̲ᴛ̲",
	cunt = "ᴄ̲ᴜ̲ɴ̲ᴛ̲",
	tits= "ᴛ̲ɪ̲ᴛ̲s̲",
	kys = "ᴋ̲ʏ̲s̲",
	dick = "ᴅ̲ɪ̲ᴄ̲ᴋ̲",
	boobs = "ʙ̲ᴏ̲ᴏ̲ʙ̲s̲",
	ass = "ᴀ̲s̲s̲",
	fucking = "ғ̲ᴜ̲ᴄ̲ᴋ̲ɪ̲ɴ̲ɢ̲",
	minors = "M̲ɪ̲ɴ̲ᴏ̲ʀ̲s̲",
	hoe = "ʜ̲ᴏ̲ᴇ̲",
	horny = "ʜ̲ᴏ̲ʀ̲ɴ̲ʏ̲",
	retard = "ʀ̲ᴇ̲ᴛ̲ᴀ̲ʀ̲ᴅ̲",
	faggot = "ғ̲ᴀ̲ɢ̲ɢ̲ᴏ̲ᴛ̲",
	masturbate = "ᴍ̲ᴀ̲s̲ᴛ̲ᴜ̲ʀ̲ʙ̲ᴀ̲ᴛ̲ᴇ̲",
	slave = "s̲ʟ̲ᴀ̲ᴠ̲ᴇ̲",
	slaves = "s̲ʟ̲ᴀ̲ᴠ̲ᴇ̲s̲",
}

-- Function to print the autocorrect list
local function printCorrectionList()
	print("Autocorrect List:")
	for key, value in pairs(corrections) do
		print(key .. " -> " .. value)
	end
end

-- Print the correction list when the script runs
printCorrectionList()

button.MouseButton1Click:Connect(function()
	local inputText = textBox.Text:lower()
	local words = inputText:split(" ")
	
	for i, word in ipairs(words) do
		words[i] = corrections[word] or word
	end
	
	local correctedText = table.concat(words, " ")
	game.ReplicatedStorage.DefaultChatSystemChatEvents.SayMessageRequest:FireServer(correctedText, "All")
end)
