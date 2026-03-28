-- MRC HUB FINAL FIX (FUNCIONANDO DE VERDADE)

local Players = game:GetService("Players")
local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")

local player = Players.LocalPlayer
local placeId = game.PlaceId

-- CONFIG
local API_URL = "https://petfindervm003-default-rtdb.firebaseio.com/pets"
local MIN_VALUE = 30000000

-- ================= GUI =================

local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))

-- BOTÃO FLUTUANTE
local float = Instance.new("TextButton", gui)
float.Size = UDim2.new(0,50,0,50)
float.Position = UDim2.new(0.02,0,0.4,0)
float.Text = "MRC"
float.BackgroundColor3 = Color3.fromRGB(0,255,127)
float.TextColor3 = Color3.new(0,0,0)
float.Draggable = true
Instance.new("UICorner", float)

-- MAIN
local main = Instance.new("Frame", gui)
main.Size = UDim2.new(0,250,0,300)
main.Position = UDim2.new(0.5,-125,0.5,-150)
main.BackgroundColor3 = Color3.fromRGB(25,25,25)
main.Visible = true
main.Active = true
main.Draggable = true
Instance.new("UICorner", main)

float.MouseButton1Click:Connect(function()
	main.Visible = not main.Visible
end)

-- TITLE
local title = Instance.new("TextLabel", main)
title.Size = UDim2.new(1,0,0,40)
title.BackgroundTransparency = 1
title.Text = "MRC FINAL"
title.TextColor3 = Color3.fromRGB(0,255,127)

-- STATUS
local status = Instance.new("TextLabel", main)
status.Size = UDim2.new(1,0,0,30)
status.Position = UDim2.new(0,0,0,40)
status.BackgroundTransparency = 1
status.Text = "Idle"

-- BOTÕES
local autoBtn = Instance.new("TextButton", main)
autoBtn.Size = UDim2.new(0.8,0,0,40)
autoBtn.Position = UDim2.new(0.1,0,0.3,0)
autoBtn.Text = "Auto Join: OFF"
autoBtn.BackgroundColor3 = Color3.fromRGB(255,80,80)
Instance.new("UICorner", autoBtn)

local notifBtn = Instance.new("TextButton", main)
notifBtn.Size = UDim2.new(0.8,0,0,40)
notifBtn.Position = UDim2.new(0.1,0,0.55,0)
notifBtn.Text = "Notificação: ON"
notifBtn.BackgroundColor3 = Color3.fromRGB(0,170,255)
Instance.new("UICorner", notifBtn)

-- ================= VAR =================

local autoJoin = false
local notifications = true

-- ================= FUNÇÕES =================

local function format(v)
	return math.floor(v/1000000).."M"
end

local function getValue(text)
	local num = text:match("%d+%.?%d*")
	if not num then return 0 end
	local v = tonumber(num)
	if text:find("M") then v *= 1000000 end
	if text:find("K") then v *= 1000 end
	return v
end

local function hasPet()
	local plots = workspace:FindFirstChild("Plots")
	if not plots then return false end
	
	for _,p in pairs(plots:GetChildren()) do
		local pods = p:FindFirstChild("AnimalPodiums")
		if not pods then continue end
		
		for _,pod in pairs(pods:GetChildren()) do
			local gen = pod:FindFirstChild("Base", true)
			if gen then gen = gen:FindFirstChild("Generation", true) end
			
			if gen and gen:IsA("TextLabel") then
				local v = getValue(gen.Text)
				if v >= MIN_VALUE then
					return true, v
				end
			end
		end
	end
	
	return false, 0
end

-- 🔔 NOTIFICAÇÃO
local function notify(text)
	if not notifications then return end
	
	local n = Instance.new("TextLabel", gui)
	n.Size = UDim2.new(0,250,0,50)
	n.Position = UDim2.new(1,-260,1,-60)
	n.BackgroundColor3 = Color3.fromRGB(30,30,30)
	n.TextColor3 = Color3.new(1,1,1)
	n.Text = text
	Instance.new("UICorner", n)
	
	task.delay(5, function()
		n:Destroy()
	end)
end

-- 🔄 SERVER HOP
local function hop()
	status.Text = "Procurando..."
	
	local req = game:HttpGet("https://games.roblox.com/v1/games/"..placeId.."/servers/Public?sortOrder=Asc&limit=100")
	local data = HttpService:JSONDecode(req)
	
	for _,v in pairs(data.data) do
		if v.playing < v.maxPlayers then
			TeleportService:TeleportToPlaceInstance(placeId, v.id, player)
			break
		end
	end
end

-- ================= LOOP =================

task.spawn(function()
	while true do
		task.wait(8)
		
		if autoJoin then
			status.Text = "Verificando..."
			
			local ok, val = hasPet()
			
			if ok then
				status.Text = "PET "..format(val).." ✅"
				notify("PET ENCONTRADO: "..format(val))
			else
				status.Text = "Sem pet... trocando"
				hop()
			end
		else
			-- só notificação sem auto join
			local ok, val = hasPet()
			if ok then
				notify("PET: "..format(val))
			end
		end
	end
end)

-- ================= BOTÕES =================

autoBtn.MouseButton1Click:Connect(function()
	autoJoin = not autoJoin
	
	if autoJoin then
		autoBtn.Text = "Auto Join: ON"
		autoBtn.BackgroundColor3 = Color3.fromRGB(0,170,255)
	else
		autoBtn.Text = "Auto Join: OFF"
		autoBtn.BackgroundColor3 = Color3.fromRGB(255,80,80)
	end
end)

notifBtn.MouseButton1Click:Connect(function()
	notifications = not notifications
	
	if notifications then
		notifBtn.Text = "Notificação: ON"
	else
		notifBtn.Text = "Notificação: OFF"
	end
end)
