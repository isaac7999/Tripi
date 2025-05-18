-- [ PROTEÇÃO ANTI-DETECT ]
pcall(function() setfflag("CrashUploadToBacktracePercentage", "0") end)
pcall(function() setfflag("CrashUploadToBacktraceToBacktraceBasePercentage", "0") end)
pcall(function() settings().Rendering.QualityLevel = Enum.QualityLevel.Level01 end)

-- [ VARIÁVEIS ]
local player = game.Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local humanoid = char:WaitForChild("Humanoid")
local root = char:WaitForChild("HumanoidRootPart")
local destino = root.Position

-- [ GUI DISCRETO ]
local gui = Instance.new("ScreenGui", game.CoreGui)
gui.Name = "sys_gui_" .. math.random(10000,99999)

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 180, 0, 60)
frame.Position = UDim2.new(0.01, 0, 0.6, 0)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
frame.BorderSizePixel = 0
frame.BackgroundTransparency = 0.3
frame.Active = true
frame.Draggable = true

local input = Instance.new("TextBox", frame)
input.Size = UDim2.new(0.6, 0, 0.5, 0)
input.Position = UDim2.new(0.05, 0, 0.25, 0)
input.PlaceholderText = "ID"
input.Text = ""
input.Font = Enum.Font.Code
input.TextSize = 12
input.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
input.TextColor3 = Color3.fromRGB(180, 180, 180)
input.BorderSizePixel = 0

local button = Instance.new("TextButton", frame)
button.Size = UDim2.new(0.3, 0, 0.5, 0)
button.Position = UDim2.new(0.65, 0, 0.25, 0)
button.Text = "Go"
button.Font = Enum.Font.Code
button.TextSize = 12
button.BackgroundColor3 = Color3.fromRGB(50, 90, 50)
button.TextColor3 = Color3.fromRGB(200, 200, 200)
button.BorderSizePixel = 0

-- [ ESP DISCRETO DE CARROS ]
local espStorage = {}
local carroID = 1

function criarESP(modelo, numero)
	if espStorage[modelo] then return end

	local highlight = Instance.new("Highlight", modelo)
	highlight.FillColor = Color3.fromRGB(255, 0, 0) -- vermelho
	highlight.OutlineColor = Color3.fromRGB(150, 0, 0) -- vermelho escuro
	highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
	highlight.Name = "ESP_HL"

	local billboard = Instance.new("BillboardGui", modelo)
	billboard.Size = UDim2.new(0, 100, 0, 30)
	billboard.Adornee = modelo:FindFirstChild("PrimaryPart") or modelo:FindFirstChildWhichIsA("BasePart")
	billboard.AlwaysOnTop = true
	billboard.Name = "ESP_TAG"
	billboard.StudsOffset = Vector3.new(0, 50, 0) -- ERGUIDO MUITO

	local label = Instance.new("TextLabel", billboard)
	label.Size = UDim2.new(1, 0, 1, 0)
	label.BackgroundTransparency = 1
	label.Text = "ID: " .. tostring(numero)
	label.TextColor3 = Color3.fromRGB(255, 0, 0) -- vermelho
	label.Font = Enum.Font.Code
	label.TextSize = 11

	espStorage[modelo] = true
end

-- [ SCAN DE VEÍCULOS ]
function monitorarCarros()
	for _, v in ipairs(workspace:GetDescendants()) do
		if v:IsA("VehicleSeat") and v:FindFirstAncestorOfClass("Model") then
			local carro = v:FindFirstAncestorOfClass("Model")
			if not carro:FindFirstChild("ESP_HL") then
				carro.PrimaryPart = v
				criarESP(carro, carroID)
				carro:SetAttribute("CarID", carroID)
				carroID += 1
			end
		end
	end
end

monitorarCarros()
workspace.DescendantAdded:Connect(function(obj)
	task.wait(0.1)
	if obj:IsA("VehicleSeat") then
		monitorarCarros()
	end
end)

-- [ PUXAR CARRO ]
local function puxarCarroPorNumero(numero)
	for _, v in ipairs(workspace:GetDescendants()) do
		if v:IsA("VehicleSeat") and v:FindFirstAncestorOfClass("Model") then
			local carro = v:FindFirstAncestorOfClass("Model")
			if carro:GetAttribute("CarID") == tonumber(numero) then
				local seat = v
				local vehicle = carro
				root.CFrame = seat.CFrame * CFrame.new(0, 2.5, 0)
				task.wait(0.3)
				humanoid:ChangeState(Enum.HumanoidStateType.Seated)
				seat:Sit(humanoid)
				local timeout = 2
				while seat.Occupant ~= humanoid and timeout > 0 do
					seat:Sit(humanoid)
					task.wait(0.2)
					timeout -= 0.2
				end
				if seat.Occupant == humanoid then
					for _, part in ipairs(vehicle:GetDescendants()) do
						if part:IsA("BasePart") then
							part.Anchored = true
						end
					end
					task.wait(0.1)
					vehicle:SetPrimaryPartCFrame(CFrame.new(destino + Vector3.new(0, 2.5, 0)))
					task.wait(0.2)
					for _, part in ipairs(vehicle:GetDescendants()) do
						if part:IsA("BasePart") then
							part.Anchored = false
						end
					end
				end
				return
			end
		end
	end
end

-- [ BOTÃO DE ATIVAÇÃO ]
button.MouseButton1Click:Connect(function()
	local num = tonumber(input.Text)
	if num then
		puxarCarroPorNumero(num)
	end
end)
