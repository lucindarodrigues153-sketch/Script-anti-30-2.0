local Players = game:GetService("Players")
local ServerScriptService = game:GetService("ServerScriptService")

local NotificationService = require(ServerScriptService.Services.NotificationService)

local playerJoinTimes = {}
local STEAL_PROTECTION_TIME = 0 -- Muda para 0 para desativar completamente

-- Regista a hora em que o jogador entra
Players.PlayerAdded:Connect(function(player)
	playerJoinTimes[player.UserId] = os.clock()
end)

-- Limpa o registo quando o jogador sai
Players.PlayerRemoving:Connect(function(player)
	playerJoinTimes[player.UserId] = nil
end)

-- Função para verificar se o alvo ainda está protegido
local function CanStealFrom(player: Player, victimOwner: Player): boolean
	if not victimOwner or player == victimOwner then
		return true
	end

	local joinTime = playerJoinTimes[victimOwner.UserId]
	if joinTime then
		local elapsed = os.clock() - joinTime
		local remaining = STEAL_PROTECTION_TIME - elapsed

		if remaining > 0 then
			if not player:GetAttribute("AntiStealSpam") then
				player:SetAttribute("AntiStealSpam", true)
				NotificationService.Notify(
					player,
					string.format("<font color=\"#FF0000\">This player just joined, please wait %d seconds</font>", math.ceil(remaining)),
					3,
					"Sounds.Sfx.Error"
				)
				task.delay(0.8, function()
					if player then
						player:SetAttribute("AntiStealSpam", nil)
					end
				end)
			end
			return false -- Bloqueia o roubo
		end
	end

	return true -- Permite o roubo
end
