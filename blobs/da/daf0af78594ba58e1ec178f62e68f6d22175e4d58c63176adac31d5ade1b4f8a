--[[
    Poggy's Supply Drops & Scavenger Hunts - translations

    Every player-facing message lives here. Words in {braces} are filled in
    by the script ({item}, {location}, {amount}, {original}, {firstname},
    {lastname}); keep them in place when translating.

    How a notification looks (position, icon, colour, how long it stays) is
    part of the script (client/notifications.lua) and is drawn by poggy_core.
]]

-- Current language (change this to switch languages)
Config.Language = "en" -- Options: "en", "es", "fr"

Translations = {
    ["en"] = {
        -- Notification titles
        SupplyDropInboundTitle = "SUPPLY DROP INBOUND!",
        SupplyDropTitle = "SUPPLY DROP!",
        ScavengerHuntTitle = "SCAVENGER HUNT!",

        -- Messages
        SupplyDropInbound = "A balloon carrying {item} is descending over {location}! Act fast to claim it!",
        SupplyDrop = "A surplus of {item} has been delivered to {location}! Act fast to claim it!",
        SupplyDropCollected = "You collected {amount}x {item}",
        SupplyDropCollectedByOther = "The supply drop was collected by {firstname} {lastname}.",
        ScavengerHuntStarted = "A new treasure hunt has begun! Use /clue to view details.",
        ScavengerHuntCollectedItem = "You collected {amount}x {item}",
        ScavengerHuntCollectedMoney = "You collected {amount}",
        ScavengerHuntCollectedByOther = "The treasure was found by {firstname} {lastname}.",
        NoActiveScavengerHunt = "There are no active scavenger hunts at the moment.",
        SurrenderPenalty = "You'll receive {amount} instead of {original} due to revealing the clue.",
        CannotCarryMoreItems = "You cannot carry any more items!",
        CannotCarryMoreWeapons = "You cannot carry any more weapons!",
        ErrorProcessingReward = "Error processing reward. Please contact an administrator.",
        NoActiveSupplyDrop = "No active supply drop available.",
        NoActiveHunt = "No active scavenger hunt available.",
        ScavengerHuntsDisabled = "Scavenger hunts are currently disabled.",
    },

    -- Titles fall back to English when a language does not define them.
    ["es"] = {
        SupplyDropInbound = "¡Un globo aerostático con {item} está descendiendo sobre {location}! ¡Actúa rápido para reclamarlo!",
        SupplyDrop = "¡Un excedente de {item} ha sido entregado en {location}! ¡Actúa rápido para reclamarlo!",
        SupplyDropCollected = "Has recogido {amount}x {item}",
        SupplyDropCollectedByOther = "El paquete fue recogido por {firstname} {lastname}.",
        ScavengerHuntStarted = "¡Una nueva búsqueda del tesoro ha comenzado! Usa /clue para ver detalles.",
        ScavengerHuntCollectedItem = "Has recogido {amount}x {item}",
        ScavengerHuntCollectedMoney = "Has recogido {amount}",
        ScavengerHuntCollectedByOther = "El tesoro fue encontrado por {firstname} {lastname}.",
        NoActiveScavengerHunt = "No hay búsquedas del tesoro activas en este momento.",
        SurrenderPenalty = "Recibirás {amount} en lugar de {original} por revelar la pista.",
        CannotCarryMoreItems = "¡No puedes llevar más objetos!",
        CannotCarryMoreWeapons = "¡No puedes llevar más armas!",
        ErrorProcessingReward = "Error al procesar la recompensa. Por favor contacta a un administrador.",
        NoActiveSupplyDrop = "No hay suministro activo disponible.",
        NoActiveHunt = "No hay búsqueda del tesoro activa.",
        ScavengerHuntsDisabled = "Las búsquedas del tesoro están deshabilitadas actualmente.",
    },

    ["fr"] = {
        SupplyDropInbound = "Un ballon à air chaud transportant {item} descend sur {location} ! Agissez vite pour le récupérer !",
        SupplyDrop = "Un surplus de {item} a été livré à {location} ! Agissez vite pour le récupérer !",
        SupplyDropCollected = "Vous avez récupéré {amount}x {item}",
        SupplyDropCollectedByOther = "Le colis a été récupéré par {firstname} {lastname}.",
        ScavengerHuntStarted = "Une nouvelle chasse au trésor a commencé ! Utilisez /clue pour voir les détails.",
        ScavengerHuntCollectedItem = "Vous avez récupéré {amount}x {item}",
        ScavengerHuntCollectedMoney = "Vous avez récupéré {amount}",
        ScavengerHuntCollectedByOther = "Le trésor a été trouvé par {firstname} {lastname}.",
        NoActiveScavengerHunt = "Il n’y a aucune chasse au trésor active pour le moment.",
        SurrenderPenalty = "Vous recevrez {amount} au lieu de {original} pour avoir révélé l’indice.",
        CannotCarryMoreItems = "Vous ne pouvez pas porter plus d’objets !",
        CannotCarryMoreWeapons = "Vous ne pouvez pas porter plus d’armes !",
        ErrorProcessingReward = "Erreur lors du traitement de la récompense. Veuillez contacter un administrateur.",
        NoActiveSupplyDrop = "Aucun largage actif disponible.",
        NoActiveHunt = "Aucune chasse au trésor active disponible.",
        ScavengerHuntsDisabled = "Les chasses au trésor sont actuellement désactivées.",
    },
}

-- Get a message for the current language. `vars` is an optional table of
-- placeholder values: T("SupplyDropCollected", { amount = 5, item = "Coal" }).
function T(key, vars)
    local lang = Config.Language or "en"
    local set = Translations[lang] or Translations["en"]
    local text = set[key] or Translations["en"][key] or key
    if type(vars) == "table" then
        for name, value in pairs(vars) do
            text = text:gsub("{" .. name .. "}", function() return tostring(value) end)
        end
    end
    return text
end
