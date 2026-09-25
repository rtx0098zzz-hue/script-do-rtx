-- ServerScriptService/AutoFarm.server.lua

local Players = game:GetService("Players")

local NPCFolder = workspace:WaitForChild("NPCs")

local ATTACK_RANGE = 12
local DAMAGE = 10
local ATTACK_COOLDOWN = 0.6

local activeFarm = {}

local function getRoot(character)
	return character and character:FindFirstChild("HumanoidRootPart")
end

local function getNearestNPC(player)
	local character = player.Character
	local root = getRoot(character)

	if not root then
		return nil
	end

	local nearest = nil
	local distance = math.huge

	for _, npc in ipairs(NPCFolder:GetChildren()) do
		local humanoid = npc:FindFirstChildOfClass("Humanoid")
		local npcRoot = getRoot(npc)

		if humanoid and npcRoot and humanoid.Health > 0 then
			local d = (root.Position - npcRoot.Position).Magnitude

			if d < distance then
				distance = d
				nearest = npc
			end
		end
	end

	return nearest
end

local function farm(player)
	while activeFarm[player] do
		local character = player.Character
		local root = getRoot(character)

		if not root then
			task.wait(1)
			continue
		end

		local npc = getNearestNPC(player)

		if npc then
			local npcRoot = getRoot(npc)
			local humanoid = npc:FindFirstChildOfClass("Humanoid")

			if npcRoot and humanoid then
				-- Aproxima o jogador do NPC
				root.CFrame = npcRoot.CFrame * CFrame.new(0, 0, 6)

				-- Dano somente em NPCs
				if (root.Position - npcRoot.Position).Magnitude <= ATTACK_RANGE then
					humanoid:TakeDamage(DAMAGE)
				end
			end
		end

		task.wait(ATTACK_COOLDOWN)
	end
end

local function setFarm(player, enabled)
	if enabled then
		if activeFarm[player] then
			return
		end

		activeFarm[player] = true

		task.spawn(function()
			farm(player)
		end)
	else
		activeFarm[player] = nil
	end
end

Players.PlayerRemoving:Connect(function(player)
	activeFarm[player] = nil
end)

-- Exemplo de ativação:
-- setFarm(player, true)
-- setFarm(player, false)
