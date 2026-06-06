local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- SETTINGS
local LOCK_ENABLED = false
local MAX_DISTANCE = 150
local SMOOTHNESS = 0.08
local HEAD_OFFSET = Vector3.new(0, 0.5, 0)

-- UI
local gui = Instance.new("ScreenGui")
gui.ResetOnSpawn = false
gui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local toggle = Instance.new("TextButton")
toggle.Size = UDim2.new(0, 18, 0, 40)
toggle.Position = UDim2.new(0, -2, 0.5, -20)
toggle.Text = "•"
toggle.BackgroundTransparency = 0.3
toggle.TextScaled = true
toggle.Parent = gui

toggle.MouseButton1Click:Connect(function()
	LOCK_ENABLED = not LOCK_ENABLED

	if LOCK_ENABLED then
		toggle.BackgroundColor3 = Color3.fromRGB(0,170,0)
	else
		toggle.BackgroundColor3 = Color3.fromRGB(170,0,0)
	end
end)

local function holdingTool()
	local character = LocalPlayer.Character
	if not character then
		return false
	end

	for _, obj in ipairs(character:GetChildren()) do
		if obj:IsA("Tool") then
			return true
		end
	end

	return false
end

local function visible(part)
	local params = RaycastParams.new()
	params.FilterType = Enum.RaycastFilterType.Blacklist
	params.FilterDescendantsInstances = {
		LocalPlayer.Character
	}

	local origin = Camera.CFrame.Position
	local direction = part.Position - origin

	local result = workspace:Raycast(origin, direction, params)

	if not result then
		return true
	end

	return result.Instance:IsDescendantOf(part.Parent)
end

local function getTarget()
	local bestTarget
	local bestScore = math.huge

	for _, player in ipairs(Players:GetPlayers()) do
		if player ~= LocalPlayer then
			local char = player.Character

			if char then
				local head = char:FindFirstChild("Head")
				local humanoid = char:FindFirstChildOfClass("Humanoid")

				if head and humanoid and humanoid.Health > 0 then
					if visible(head) then

						local screenPos, onScreen =
							Camera:WorldToViewportPoint(head.Position)

						if onScreen then
							local center = Vector2.new(
								Camera.ViewportSize.X / 2,
								Camera.ViewportSize.Y / 2
							)

							local score =
								(Vector2.new(screenPos.X, screenPos.Y) - center).Magnitude

							if score < bestScore then
								bestScore = score
								bestTarget = head
							end
						end
					end
				end
			end
		end
	end

	return bestTarget
end

RunService.RenderStepped:Connect(function()

	if not LOCK_ENABLED then
		return
	end

	-- Only lock while holding a Tool
	if not holdingTool() then
		return
	end

	local target = getTarget()

	if target then
		local camPos = Camera.CFrame.Position

		local desiredCF = CFrame.lookAt(
			camPos,
			target.Position + HEAD_OFFSET
		)

		Camera.CFrame = Camera.CFrame:Lerp(
			desiredCF,
			SMOOTHNESS
		)
	end
end) 
