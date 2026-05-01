ServerStorage = game:GetService("ServerStorage")
ReplicatedStorage = game:GetService("ReplicatedStorage")
Players = game:GetService("Players")

Maps = ServerStorage:WaitForChild("Mapas"):GetChildren()
Status = ReplicatedStorage:WaitForChild("GameStatus")

local voteEvent = ReplicatedStorage:WaitForChild("VoteEvent")
local frameEvent = ReplicatedStorage:WaitForChild("FrameEvent")
local mapaVerdeValue = ReplicatedStorage:WaitForChild("MapaVerde")
local mapaAzulValue = ReplicatedStorage:WaitForChild("MapaAzul")
local mapaRosaValue = ReplicatedStorage:WaitForChild("MapaRosa")

local intervalo = script.Intervalo.Value
local tempo = script.Tempo.Value

local function resetVotes()
	mapaVerdeValue.Value = 0
	mapaAzulValue.Value = 0
	mapaRosaValue.Value = 0
end

local function getWinningMap()
	local verde = mapaVerdeValue.Value
	local azul = mapaAzulValue.Value
	local rosa = mapaRosaValue.Value
	
	local maxVotes = math.max(verde, azul, rosa)
	local winners = {}
	
	if verde == maxVotes then table.insert(winners, "Mapa Verde") end
	if azul == maxVotes then table.insert(winners, "Mapa Azul") end
	if rosa == maxVotes then table.insert(winners, "Mapa Rosa") end
	
	if #winners == 1 then
		return winners[1]
	else
		return winners[math.random(1, #winners)]
	end
end

voteEvent.OnServerEvent:Connect(function(player, mapName, change)
	if mapName == "MapaVerde" then
		mapaVerdeValue.Value = math.max(0, mapaVerdeValue.Value + change)
	elseif mapName == "MapaAzul" then
		mapaAzulValue.Value = math.max(0, mapaAzulValue.Value + change)
	elseif mapName == "MapaRosa" then
		mapaRosaValue.Value = math.max(0, mapaRosaValue.Value + change)
	end
end)

while true do
	intervalo = 10
	TimeOfGame = nil

	local Countdown = intervalo

	repeat wait(1)
		Countdown -= 1
		Status.Value = 'O próximo mapa vai ser escolhido em '..Countdown.. " segundos."

		
		for _, Player in pairs(game.Players:GetChildren()) do
			local Humanoid = Player.Character and Player.Character:FindFirstChild("Humanoid")
			if Humanoid and Humanoid.Health > 0 then
				repeat wait(0.01)
					Player:FindFirstChild("inGame").Value = false
				until Player:FindFirstChild("inGame").Value == false

			end
		end
		
	until Countdown <= 0
	
	resetVotes()
	
	Status.Value = "Escolhendo mapa..."
	frameEvent:FireAllClients("Show")
	
	wait(15)
	
	frameEvent:FireAllClients("Hide")
	
	local winningMapName = getWinningMap()
	local ChosenMap = nil
	
	for _, map in ipairs(Maps) do
		if map.Name == winningMapName then
			ChosenMap = map:Clone()
			break
		end
	end
	
	if not ChosenMap then
		ChosenMap = Maps[math.random(1, #Maps)]:Clone()
	end
	
	resetVotes()
	
	Spawns = ChosenMap:WaitForChild("Spawns"):GetChildren()

	local RandomSpawn = Spawns[math.random(1, #Spawns)]
	
	wait(2) 
	
	Status.Value = "Mapa escolhido: "..ChosenMap.Name
	ChosenMap.Parent = workspace
	
	Countdown = 3
	
	repeat wait(1)
		Countdown -= 1
		Status.Value = "Players sendo teleportados em "..Countdown.." segundos"
	until Countdown <= 0
	
	Status.Value = "Teleportando Players"
	
	local plrsAlive = {}
	
	for _, Player in pairs(game.Players:GetChildren()) do
		local Humanoid = Player.Character and Player.Character:FindFirstChild("Humanoid")
		if Humanoid and Humanoid.Health > 0  then
			repeat wait(0.1)
				Player:FindFirstChild("inGame").Value = true
				table.insert(plrsAlive, Player)
				print("Players em jogo: "..#plrsAlive)
			until Player:FindFirstChild("inGame").Value == true 
		end
	end
	

	wait(1)
	
	for _, Player in pairs(Players:GetChildren()) do
		if Player.Character and Player.Character:FindFirstChild('Humanoid') and  Player.Character:FindFirstChild('Humanoid').Health >= 0 and Player:FindFirstChild("inGame")  and Player:FindFirstChild("inGame").Value == true then
			
			Player.Character.HumanoidRootPart.CFrame = RandomSpawn.CFrame
		    Player.Character:WaitForChild("Humanoid").WalkSpeed = 0
		end
	end
	
	wait(3)

	Countdown = 5
	
	repeat wait(1)
		Countdown -= 1
		Status.Value = "O jogo vai começar em "..Countdown.." segundos"
	until Countdown <= 0
	
	Status.Value = " "
	
	for _, Player in pairs(Players:GetChildren()) do
		if Player.Character and Player.Character:FindFirstChild('Humanoid') and  Player.Character:FindFirstChild('Humanoid').Health >= 0 and Player:FindFirstChild("inGame")  and Player:FindFirstChild("inGame").Value == true then
			Player.Character:WaitForChild("Humanoid").WalkSpeed = 16
		end
	end
	
	game.Players.PlayerRemoving:Connect(function(player)
		if player then
			table.remove(plrsAlive, table.find(plrsAlive, player))
		end
	end)    
	

	
	Countdown = tempo	
	
	while Countdown > 0 do
		wait(1)
		Countdown -= 1 
		local minutos = math.floor(Countdown / 60) 
		local segundos = Countdown % 60
		local tempoFormatado = string.format("%02d:%02d", minutos, segundos) 
		Status.Value = tempoFormatado



		if #plrsAlive == 0 and Countdown >= 3 then
			break	
		end


	end
	 

	
	wait(1)
	
	for _, jogadores in pairs(game.Players:GetPlayers()) do
		local Hum = jogadores.Character and jogadores.Character:FindFirstChild("Humanoid")
		if Hum and Hum.Health > 0 then
			local rootPart = Hum.RootPart
			if rootPart then
				rootPart.CFrame = workspace:WaitForChild("SpawnLocation").CFrame
				jogadores:FindFirstChild("inGame").Value = false 
			end
		end
	end



	

	ChosenMap:Destroy()
	Status.Value = "GAME OVER"

	

	wait(2)

end

