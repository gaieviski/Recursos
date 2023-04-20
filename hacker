local Tunnel = module("vrp", "lib/Tunnel")
local Proxy = module("vrp", "lib/Proxy")

vRP = Proxy.getInterface("vRP")
vRPclient = Tunnel.getInterface("vRP")

local progressBars = exports['progressBars']
local atm_hacking = false
local playerInService = false
local ponyentregue = false
local playerCoords = GetEntityCoords(GetPlayerPed(-1))
local markerPosition = {x = 954.91, y = -1511.76, z = 31.12}
local destinationPosition = {x = 939.55, y = -1501.63, z = 30.31}
local vehicle2 = nil
local hackeando = false
local Friends = false
local playerEnemyStatus = 0

local atms = {
  [1] = {x = -3241.14, y = 1008.62, z = 12.83, rot = 261.9031, carx = -3242.7678, cary = 990.4905, carz = 12.4773, carrot = 270.2892  },   
  [2] = {x = 540.35, y = 2670.45, z = 42.15, rot = 12.56, carx = 565.2198, cary = 2669.6272, carz = 42.0794, carrot = 1.5392 },    
  [3] = {x = 2682.51, y = 3286.91, z = 55.24, rot = 250.65, carx = 2684.0234, cary = 3292.6692, carz = 55.2407, carrot = 240.3749 },    
  [4] = {x = 1702.57, y = 4933.12, z = 42.06, rot = 320.91, carx = 1706.3153, cary = 4944.5981, carz = 42.2027, carrot = 53.7671 },  
  [5] = {x = 1735.61, y = 6411.26, z = 35.03, rot = 159.55, carx = 1717.6262, cary = 6417.9678, carz = 33.3873, carrot = 157.1014 },  
  [6] = {x = -676.82, y = 5835.20, z = 17.33, rot = 143.12, carx = -666.2936, cary = 5827.6118, carz = 17.3312, carrot = 129.1576 }    
}  

local randomIndex = math.random(1, #atms)
local randomAtm = atms[randomIndex]


function criarPony(carx, cary, carz, carrot)
  Citizen.CreateThread(function()
      local model = GetHashKey("pony")

      if not IsModelValid(model) then
          print("Modelo inválido.")
          return
      end

      RequestModel(model)

      while not HasModelLoaded(model) do
          Citizen.Wait(0)
      end

      local vehicle2 = CreateVehicle(model, randomAtm.carx, randomAtm.cary, randomAtm.carz, randomAtm.carrot, true, false)
      SetVehicleDoorsLocked(vehicle2, 1)
      SetVehicleEngineOn(vehicle2, true, true, false) 
      local plate = "TDEA"..tostring(math.random(1000, 9999))
      SetVehicleNumberPlateText(vehicle2, plate)
      TriggerServerEvent("setPlateEveryone",plate)
      SetModelAsNoLongerNeeded(model)

      RegisterNetEvent("movePlayer")
      AddEventHandler("movePlayer", function(perigo)
      local playerPed = GetPlayerPed(-1)
      TaskWarpPedIntoVehicle(playerPed, vehicle2, -1)
      end)

      RegisterNetEvent("deleteVehicle2")
      AddEventHandler("deleteVehicle2", function()
        if vehicle2 ~= nil and DoesEntityExist(vehicle2) then
          -- Exclua o veículo
          SetEntityAsMissionEntity(vehicle2, true, true)
          DeleteEntity(vehicle2)
          DeleteVehicle(vehicle2)
          vehicle2 = nil
        end
      end)
  end)
end







-- Lista de coordenadas para entregar o dinheiro
local delivery_points = {
  {x = 947.74, y = -1472.53, z = 30.40}, -- insira as coordenadas X, Y e Z dos pontos de entrega
  -- adicione mais pontos de entrega conforme necessário
}

local hacker_job_locations = {
  {x = 939.42, y = -1461.67, z = 33.61}, -- insira as coordenadas X, Y e Z do blip de serviço para hackers
  -- adicione mais blips de serviço conforme necessário
}

-----------------------------------------------------------------------------
------------peds-------------------------------------------------------------
-----------------------------------------------------------------------------


local enemies = {}
local enemyPedVehicle
local enemyVehicleSpawnCount = 0
AddRelationshipGroup("enemies")


local weapons = {
  "weapon_revolver",
  "weapon_combatpistol",
  "weapon_snspistol_mk2"
}

function isValidSpawnLocation(coords)
  local isRoad, isIntersection = IsPointOnRoad(coords.x, coords.y, coords.z)

  return isRoad or isIntersection
end


function findValidSpawnLocation(playerCoords, minDist, maxDist)
  local spawnLocationFound = false
  local spawnCoords

  while not spawnLocationFound do
      local randomAngle = math.random() * 1 * math.pi
      local randomDist = math.random() * (maxDist - minDist) + minDist
      local newX = playerCoords.x + math.cos(randomAngle) * randomDist
      local newY = playerCoords.y + math.sin(randomAngle) * randomDist
      local newZ = playerCoords.z

      local _, groundZ = GetGroundZFor_3dCoord(newX, newY, newZ, 0)
      local isWater = TestVerticalProbeAgainstAllWater(newX, newY, newZ, 0)

      if groundZ ~= 0 and not isWater and isValidSpawnLocation(vector3(newX, newY, groundZ)) then
          spawnLocationFound = true
          spawnCoords = vector3(newX, newY, groundZ)
      end
  end

  return spawnCoords
end

function addBlipToVehicle(vehicle)
  local blip = AddBlipForEntity(vehicle)
  SetBlipSprite(blip, 225) -- Define o ícone do blip (225 é um carro)
  SetBlipColour(blip, 1) -- Define a cor do blip (1 é vermelho)
  SetBlipScale(blip, 0.8) -- Define o tamanho do blip
  SetBlipAsShortRange(blip, false) -- Define se o blip aparece apenas em curta distância (false para aparecer em qualquer distância)
  BeginTextCommandSetBlipName("STRING")
  AddTextComponentSubstringPlayerName("Seguranças")
  EndTextCommandSetBlipName(blip)
end


local enemiesKilled = 0

function gerarPeds()
  local playerPed = PlayerPedId()
  local playerCds = GetEntityCoords(playerPed)
  local spawnCoords = findValidSpawnLocation(playerCds, 50, 100) -- Defina minDist e maxDist conforme desejado
  local x, y, z = table.unpack(spawnCoords)
  local model = "a_m_y_business_02"
  local vehicleModel = "RRazirrisa"
  enemyPedVehicle = spawnVehicle(vehicleModel, x, y, z) 
  local weaponHash = GetHashKey(weapons[math.random(#weapons)])
  enemyVehicleSpawnCount = enemyVehicleSpawnCount + 1
  for i = 0, 4, 1 do
      local seat = i - 1
      enemies[i] = spawnPedInsideVehicle(enemyPedVehicle, 0, model, seat)
      SetPedAccuracy(enemies[i],40)
      SetPedCombatAbility(enemies[i], 2) 
      SetPedConfigFlag(enemies[i], 281, true) 
      SetCanAttackFriendly(enemies[i], false, true)
      SetPedDiesWhenInjured(enemies[i], false)
      SetPedFleeAttributes(enemies[i], 0, true)
      SetPedCombatAttributes(enemies[i], 0, true)
      SetPedCombatAttributes(enemies[i], 5, true)
      SetPedCombatAttributes(enemies[i], 46, true)
      SetPedCombatAttributes(enemies[i], 3, false)
      SetPedCombatAttributes(enemies[i], 1, true) -- Faz com que os peds sejam mais agressivos no combate
      SetPedCombatAttributes(enemies[i], 2, true) -- Faz com que os peds se movam mais rápido durante o combate
      SetPedDropsWeaponsWhenDead(enemies[i], false)
      SetPedCanRagdollFromPlayerImpact(enemies[i], false)
      GiveWeaponToPed(enemies[i], weaponHash, 500, false, true)
      SetPedRelationshipGroupHash(enemies[i], "enemies")
      if i == 0 then
      TaskVehicleDriveToCoord(enemies[i], enemyPedVehicle, playerCds[1], playerCds[2], playerCds[3], 100.0, 5.0, GetHashKey(vehicleModel), 4981248, 100.0, 1.0)
      end
  end
  SetRelationshipBetweenGroups(5, GetHashKey("PLAYER"), GetHashKey("enemies"))
  playerEnemyStatus = 5
end


Citizen.CreateThread(function()
  local enemyVehicleDestroyed = false
  local deadPedsCount = 0

  while true do
      if enemyPedVehicle and not enemyVehicleDestroyed then
          if IsEntityDead(enemyPedVehicle) or GetEntityHealth(enemyPedVehicle) <= 0 then
              enemyVehicleSpawnCount = enemyVehicleSpawnCount - 1
              enemyVehicleDestroyed = true
          end
      end

      if not enemyVehicleDestroyed then
          for i, ped in ipairs(enemies) do
              if ped and IsEntityDead(ped) then
                  deadPedsCount = deadPedsCount + 1
                  enemies[i] = nil
              end
          end

          if deadPedsCount >= 2 then
              print("um veiculo inimigo foi explodido")
              enemyVehicleSpawnCount = enemyVehicleSpawnCount - 1
              enemyVehicleDestroyed = true
          end
      end

      if enemyVehicleDestroyed then
          break
      end

      Citizen.Wait(1000) -- Verifique a saúde do veículo e a quantidade de peds mortos a cada 500ms
  end
end)





Citizen.CreateThread(function()
  local blip
  local blipAdded = false

  while true do
    local playerCoords = GetEntityCoords(PlayerPedId())
    local enemyVehicleCoords = GetEntityCoords(enemyPedVehicle)
    local distance = #(enemyVehicleCoords - playerCoords)

    if distance < 100 and not blipAdded then
      blip = AddBlipForEntity(enemyPedVehicle)
      SetBlipSprite(blip, 225)
      SetBlipColour(blip, 1)
      SetBlipScale(blip, 0.7)
      SetBlipAsShortRange(blip, true)
      BeginTextCommandSetBlipName("STRING")
      AddTextComponentString("Seguranças")
      EndTextCommandSetBlipName(blip)
      blipAdded = true
    elseif distance >= 100 and blipAdded then
      RemoveBlip(blip)
      blipAdded = false
    end

    Citizen.Wait(1000) -- Verifique a distância a cada segundo
  end
end)




function spawnPedInsideVehicle(vehicle, pedType, model, seat)
  loadModel(model)
  return CreatePedInsideVehicle(vehicle, pedType, GetHashKey(model), seat, false, false)
end

function spawnVehicle(model, x, y, z)
  loadModel(model)
  return CreateVehicle(GetHashKey(model), x, y, z, 0.0, true, true)
end

function loadModel(model)
  RequestModel(model)
  while not HasModelLoaded(model) do
      Wait(10)
  end
end

local deadPeds = {}
local active = false

function checkDistanceAndShoot()
  local active = false
  while not active do
    Citizen.Wait(1000)
      local playerPed = PlayerPedId()
      local playerCoords = GetEntityCoords(playerPed)
      local playerHealth = GetEntityHealth(playerPed)
    if not active then
      for i = 0, 3, 1 do
          local enemyCoords = GetEntityCoords(enemies[i])
          local distance = #(enemyCoords - playerCoords)

          if DoesEntityExist(enemies[i]) and not IsEntityDead(enemies[i]) then
              if distance > 500 then
                  DeleteEntity(enemies[i])
                  enemyVehicleSpawnCount = enemyVehicleSpawnCount - 1
              end

              if playerHealth >= 101 and playerEnemyStatus == 5 and playerInService then 
                  if distance <= 80 then
                      TaskCombatPed(enemies[i], playerPed, 0, 16)
                  end
              else
                if playerHealth < 101 and IsPedInAnyVehicle(enemies[i], false) then
                  ClearRelationshipBetweenGroups(0, GetHashKey("PLAYER"), GetHashKey("enemies"))
                  TaskVehicleDriveWander(enemies[i], enemyPedVehicle, 100.0, 262657)
                end
              end
          end
      end
    end
  end
end

RegisterNetEvent("setPerigo")
AddEventHandler("setPerigo", function(perigo)
    if perigo then
        if enemyVehicleSpawnCount >= 5 then
            return
        end
        Citizen.CreateThread(function()
            while perigo do
                gerarPeds()
                checkDistanceAndShoot()
                Citizen.Wait(1000) -- Adiciona um atraso de 1 segundo (1000 ms) antes de executar novamente o loop
            end
        end)
    end
end)

RegisterCommand("stopPeds", function(source, args, rawCommand)
  active = not active
end, false)

RegisterCommand("peds", function(source, args, rawCommand)
  TriggerEvent("setPerigo", true)
end, false)   

RegisterNetEvent("setPerigoOFF")
AddEventHandler("setPerigoOFF", function()
    active = not active
end)

                      
------------------------------------------------------------------------
----- fim peds ---------------------------------------------------------
------------------------------------------------------------------------


function createServiceBlips()
  for _, location in ipairs(hacker_job_locations) do
    local blip = AddBlipForCoord(location.x, location.y, location.z)
    SetBlipSprite(blip, 521) -- Você pode alterar o ícone do blip aqui
    SetBlipDisplay(blip, 4)
    SetBlipScale(blip, 0.8)
    SetBlipColour(blip, 2)
    SetBlipAsShortRange(blip, true)
    BeginTextCommandSetBlipName("STRING")
    AddTextComponentString("Serviço de Hacker")
    EndTextCommandSetBlipName(blip)
  end
end

Citizen.CreateThread(function()
  createServiceBlips()

  while true do
    Citizen.Wait(0)

    local player = GetPlayerPed(-1)
    local playerCoords = GetEntityCoords(player)

    for _, job_location in ipairs(hacker_job_locations) do
      local distanceToJob = Vdist(playerCoords.x, playerCoords.y, playerCoords.z, job_location.x, job_location.y, job_location.z)

      if distanceToJob <= 10.5 then
        DrawMarker(27, job_location.x, job_location.y, job_location.z - 0.96, 0, 0, 0, 0, 0, 0, 1.0, 1.0, 1.0, 0, 255, 0, 155, 0, 0, 2, 0, 0, 0, 0)
        if not playerInService then
          DrawText3D(job_location.x, job_location.y, job_location.z + 0.8, "Pressione ~b~E ~w~para entrar em serviço como hacker ")
        else
          DrawText3D(job_location.x, job_location.y, job_location.z + 0.8, "Pressione ~b~E ~w~para sair do serviço como hacker ")
        end

        if distanceToJob <= 2.0 then

          if IsControlJustReleased(1, 51) then
            if not playerInService then
              atm_hacking = true
              playerInService = true
              TriggerEvent("Notify",source,"Você entrou em serviço como hacker", 10000)
              blipCreating(randomAtm.x, randomAtm.y, randomAtm.z)
            else
              playerInService = false
              TriggerEvent("Notify",source,"Você saiu do serviço como hacker", 10000)
              RemoveBlip(blips)
            end
          
          end
        end
      end
    end
    
    function updateHackingMoney(startTime, duration)
      local elapsedTime = GetGameTimer() - startTime
      local progress = math.min(elapsedTime / duration, 1.0)
      local hackedMoney = math.floor(progress * 30000)
    
      return hackedMoney
    end

    if playerInService then
      if atm_hacking then
          local distance = Vdist(playerCoords.x, playerCoords.y, playerCoords.z, randomAtm.x, randomAtm.y, randomAtm.z)
          
          if distance <= 10.5 then
              DrawMarker(27, randomAtm.x, randomAtm.y, randomAtm.z - 0.96, 0, 0, 0, 0, 0, 0, 1.0, 1.0, 1.0, 0, 255, 0, 155, 0, 0, 2, 0, 0, 0, 0)
          
              if distance <= 2.0 then
                  DrawText3D(randomAtm.x, randomAtm.y, randomAtm.z + 0.8, "Pressione ~b~E ~w~para hackear o caixa eletrônico ")

                  if IsControlJustReleased(1, 51) then
                      local playerPed = GetPlayerPed(-1)
                      SetEntityCoords(playerPed, randomAtm.x, randomAtm.y, randomAtm.z - 1.0)
                      SetEntityHeading(playerPed, randomAtm.rot)
                      local hackingStartTime = GetGameTimer()
                      local hackingDuration = 10000
                      TriggerEvent("vrp_hackerjob:client")
                      criarPony(carx, cary, carz, carrot)
                      while atm_hacking and (GetGameTimer() - hackingStartTime) < hackingDuration do
                          local currentHackedMoney = updateHackingMoney(hackingStartTime, hackingDuration) * 2
                          drawTxt("Hackeando: ~g~R$ " .. currentHackedMoney .. " / R$ " .. hackingDuration .. "", 4, 0.5, 0.78, 0.50, 255, 255, 255, 180)
                          Citizen.Wait(0)
                          hackeando = true
                      end
                  end
              end
          end
      end
    end
  end
end)

function isPlayerInPony()
  local playerPed = GetPlayerPed(-1)
  local vehicle = GetVehiclePedIsIn(playerPed, false)
  local model = GetEntityModel(vehicle)
  local modelName = GetDisplayNameFromVehicleModel(model)

  if modelName == "PONY" and IsPedInVehicle(playerPed, vehicle, false) then
    return true
  else
    return false
  end
end

RegisterNetEvent("vrp_hackerjob:client")
AddEventHandler("vrp_hackerjob:client", function()
    local player = GetPlayerPed(-1)
    local playerPed = PlayerPedId()
    local isInVehicle = false
    TaskStartScenarioInPlace(player, "WORLD_HUMAN_MOBILE_FILM_SHOCKING", 0, true)
    progressBars:startUI(10000, "%")
    Citizen.Wait(10000)
    ClearPedTasksImmediately(player)
    atm_hacking = false
    caixahackeado = true
    blipCreating2(markerPosition.x, markerPosition.y, markerPosition.z)
    TriggerEvent("movePlayer")
end)

Citizen.CreateThread(function()
  while true do
    Citizen.Wait(20000)
    local playerPed = PlayerPedId()
    local playerHealth = GetEntityHealth(playerPed)
    if playerInService and hackeando and enemyVehicleSpawnCount <= 5 and playerHealth >= 102 and playerInService then
        TriggerEvent("setPerigo", true)
    end
  end
end)

Citizen.CreateThread(function()
  while true do
    Citizen.Wait(2000)
    print(enemyVehicleSpawnCount)
    
  end
end)


    local texto3d = nil
    Citizen.CreateThread(function()
      while true do
          local ped = PlayerPedId()
          local coords = GetEntityCoords(ped)
          local player = GetPlayerPed(-1)
  
          if not ponyentregue and caixahackeado and playerInService then
              SetPedComponentVariation(player, 5, 45, 0, 2)
              RemoveBlip(blips)
              local distance3 = #(vector3(coords[1], coords[2], coords[3]) - vector3(markerPosition.x, markerPosition.y, markerPosition.z))
              if distance3 < 5.1 then
                local texto3d = DrawText3D(markerPosition.x, markerPosition.y, markerPosition.z + 0.8, "Pressione ~b~E ~w~para entregar o veículo ")
                if IsControlJustReleased(1, 51) then
                    TriggerEvent("setPerigoOFF")   
                    local texto3d = nil
                    local vehicle = GetVehiclePedIsIn(GetPlayerPed(-1), false)
                    SetEntityAsMissionEntity(vehicle, true, true)
                    DeleteVehicle(vehicle)
                    SetEntityCoords(GetPlayerPed(-1), destinationPosition.x, destinationPosition.y, destinationPosition.z)
                    ponyentregue = true
                    RemoveBlip(blips2)
                end
            end
            
          end
    ---------------------
    --Sair servico
    --------------------
        if IsControlJustPressed(0,168) then 
          if playerInService then 
            playerInService = false
            atm_hacking = false
            hackeando = false
            Friends = true
            TriggerEvent("setPerigoOFF")   
            TriggerEvent("deleteVehicle2", source)
              if blips then 
                  RemoveBlip(blips)
                  RemoveBlip(blip)
                  RemoveBlip(blips2)
              end
          end
      end
    ------------------
    ------------------
          Citizen.Wait(0)
      end
    end)



        Citizen.CreateThread(function()                                    
            while true do
              Citizen.Wait(4)
   
              if ponyentregue and caixahackeado then
              local delivery_point = delivery_points[math.random(1, #delivery_points)]
              local delivery_blip = AddBlipForCoord(delivery_point.x, delivery_point.y, delivery_point.z)
              SetBlipSprite(delivery_blip, 207)
              SetBlipColour(delivery_blip, 2)
              SetBlipRoute(delivery_blip, true)
              SetBlipRouteColour(delivery_blip, 2)
  
              local playerCoords = GetEntityCoords(GetPlayerPed(-1))
              local distance = Vdist(playerCoords.x, playerCoords.y, playerCoords.z, delivery_point.x, delivery_point.y, delivery_point.z)
  
              if distance <= 10.5 and not atm_hacking then
                  DrawMarker(27, delivery_point.x, delivery_point.y, delivery_point.z - 0.96, 0, 0, 0, 0, 0, 0, 1.0, 1.0, 1.0, 0, 255, 0, 155, 0, 0, 2, 0, 0, 0, 0)
              end

              if distance <= 2.0 then
                  DrawText3D(delivery_point.x, delivery_point.y, delivery_point.z + 0.8, "Pressione ~b~E ~w~para entregar o dinheiro ")
  
                  function createMoneyObject()
                      local player = GetPlayerPed(-1)
                      local boneIndex = GetPedBoneIndex(player, 0xDEAD)
                      local model = GetHashKey("prop_cash_pile_02")
  
                      RequestModel(model)
  
                      while not HasModelLoaded(model) do
                        Citizen.Wait(100)
                    end

                    local moneyObject = CreateObject(model, 1.0, 1.0, 1.0, true, true, false)
                    AttachEntityToEntity(moneyObject, player, boneIndex, 0.12, 0.06, -0.03, 5.0, 160.0, 0.0, false, false, false, false, 2, true)

                    return moneyObject
                end

                function removeMoneyObject(moneyObject)
                    DeleteObject(moneyObject)
                end

                if IsControlJustReleased(1, 51) then
                    local moneyObject = createMoneyObject()
                    RemoveBlip(delivery_blip)
                    hackeando = false
                    -- Exibir progress bar de entrega do dinheiro
                    TaskStartScenarioInPlace(player, "PROP_HUMAN_PARKING_METER", 0, true)
                    progressBars:startUI(60000, "Depositando o dinheiro...")
                    Citizen.Wait(60000)
                    ClearPedTasksImmediately(player)
                    TriggerServerEvent("vrp_hackerjob:moneyDelivered")

                    -- Remover objeto de dinheiro
                    removeMoneyObject(moneyObject)
                    SetPedComponentVariation(player, 5, 0, 0, 2)

                    -- Enviar notificação
                    TriggerEvent("Notify", source, "Você entregou o dinheiro, fim do serviço.", 10000)
                    playerInService = false
                    ponyentregue = false
                    caixahackeado = false
                end
              end
            end
          end
          Citizen.Wait(1)
        end)




Citizen.CreateThread(function()
    while true do 
        local time = 1000
        if playerInService then
            time = 4
            drawTxt("EM SERVIÇO / ~r~F7~w~ PARA SAIR", 4, 0.5, 0.1, 0.50, 255, 255, 255,180)
        end
        Wait(time)
    end
end)

  function DisplayHelpText(str)
  SetTextComponentFormat("STRING")
  AddTextComponentString(str)
  DisplayHelpTextFromStringLabel(0, 0, 1, -1)
  end
          
  function DrawText3D(x, y, z, text)
  local onScreen, _x, _y = World3dToScreen2d(x, y, z)
  local p = GetGameplayCamCoords()
  local distance = GetDistanceBetweenCoords(p.x, p.y, p.z, x, y, z, 1)
  local scale = (1 / distance) * 2
  local fov = (1 / GetGameplayCamFov()) * 100
  local scale = scale * fov


  if onScreen then
      SetTextScale(0.0, 0.8 * scale)
      SetTextFont(0)
      SetTextProportional(1)
      SetTextColour(255, 255, 255, 255)
      SetTextDropshadow(0, 0, 0, 0, 255)
      SetTextEdge(2, 0, 0, 0, 150)
      SetTextDropShadow()
      SetTextOutline()
      SetTextEntry("STRING")
      SetTextCentre(1)
      AddTextComponentString(text)
      DrawText(_x, _y)
  end
  end



  function blipCreating2(x,y,z)
    blips2 = AddBlipForCoord(x,y,z)
    SetBlipSprite(blips2, 67)
    SetBlipColour(blips2, 2)
    SetBlipScale(blips2, 1.0)
    SetBlipAsShortRange(blips2, false)
    SetBlipRoute(blips2, true)
    SetBlipRouteColour(blips2, 2)
    BeginTextCommandSetBlipName("STRING")
    AddTextComponentString("Destino")
    EndTextCommandSetBlipName(blips2)
    end




  function blipCreating(x,y,z)
  blips = AddBlipForCoord(x,y,z)
  SetBlipSprite(blips, 739)
  SetBlipColour(blips, 1)
  SetBlipScale(blips, 1.0)
  SetBlipAsShortRange(blips, false)
  SetBlipRoute(blips, true)
  BeginTextCommandSetBlipName("STRING")
  AddTextComponentString("Destino")
  EndTextCommandSetBlipName(blips)
  end

  function drawTxt(text, font, x, y, scale, r, g, b, a)
  SetTextFont(font)
  SetTextScale(scale, scale)
  SetTextColour(r, g, b, a)
  SetTextOutline()
  SetTextCentre(1)
  SetTextEntry("STRING")
  AddTextComponentString(text)
  DrawText(x, y)
  end

  function playAnimation(animDict, animName, duration)
  local player = GetPlayerPed(-1)

  RequestAnimDict(animDict)
  while not HasAnimDictLoaded(animDict) do
    Citizen.Wait(100)
  end

  TaskPlayAnim(player, animDict, animName, 8.0, -8.0, duration, 0, 1, false, false, false)
  Citizen.Wait(duration)
  ClearPedTasks(player)
  end


  RegisterCommand("mascara", function(source, args, rawCommand)
  local player = GetPlayerPed(-1)
  if not maskOn then
    playAnimation("mp_masks@on_foot", "put_on_mask", 800) -- Animação colocando a máscara
    Citizen.Wait(800)
    SetPedComponentVariation(player, 1, 35, 0, 2) -- Coloca a máscara número 35
    maskOn = true
  else
    playAnimation("mp_masks@on_foot", "put_on_mask", 800) -- Animação retirando a máscara
    Citizen.Wait(800)
    SetPedComponentVariation(player, 1, 0, 0, 2) -- Remove a máscara
    maskOn = false
  end
  end, false)
