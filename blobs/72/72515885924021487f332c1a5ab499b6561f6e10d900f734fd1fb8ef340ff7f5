Config = {}

-- Global Settings
Config.Debug = false  -- Set to false to disable debug messages
Config.NotificationDuration = 8000 -- 8 seconds for all notifications
Config.ShowCollectedByNotifications = false -- Show notifications when other players collect items
Config.SuccessNotificationColor = "COLOR_GREEN" -- Color for successful collection notifications
Config.SuccessNotificationDict = "BLIPS" -- Texture dictionary for notification icon
Config.SuccessNotificationIcon = "blip_cash_bag" -- Icon for successful collection notifications (blip_cash_bag = 688589278)

Config.RandomEvent = {
    Enabled = true, -- Master switch for this new random event system

    -- Settings for how often RANDOM events are triggered.
    -- These timers start AFTER the specific cooldown for that event type has passed.
    SupplyDropTimer = {
        MinTimeBetweenEvents = 600000,  -- Minimum wait (ms) before a NEW random supply drop may occur: 600000 = 10 minutes
        MaxTimeBetweenEvents = 2700000  -- Maximum wait (ms) before a NEW random supply drop may occur: 2700000 = 45 minutes
                                        -- A random time between Min and Max is chosen.
    },
    ScavengerHuntTimer = {
        MinTimeBetweenEvents = 1200000, -- Minimum wait (ms) for a new random scavenger hunt: 1200000 = 20 minutes
        MaxTimeBetweenEvents = 3600000  -- Maximum wait (ms) for a new random scavenger hunt: 3600000 = 60 minutes
    },

    -- Cooldown settings after an event is triggered (randomly or manually)
    -- This is the period during which NO new random event of THIS TYPE can occur.
    SupplyDropCooldown = {
        Min = 3600000,       -- Minimum cooldown in milliseconds: 3600000 = 60 minutes
        RandomAdd = 1800000  -- Up to this much extra, chosen at random (ms): 1800000 = 30 minutes
                             -- Results in a 60-90 minute cooldown for supply drops.
    },
    ScavengerHuntCooldown = {
        Min = 5400000,       -- Minimum cooldown in milliseconds: 5400000 = 90 minutes
        RandomAdd = 3600000  -- Up to this much extra, chosen at random (ms): 3600000 = 60 minutes
                             -- Results in a 90-150 minute cooldown for scavenger hunts.
    }
}

-- Discord Webhook Settings
Config.Discord = {
    -- Supply Drops webhook
    SupplyDrops = {
        Enabled = false,
        WebhookURL = "YOUR DISCORD WEBHOOK HERE", -- your Discord webhook URL
        BotName = "Supply Drops",
        BotIcon = "", -- optional image URL for the bot avatar
        Color = 16776960, -- Yellow color in decimal (0xFFFF00)
        Footer = "Poggy's Supply Drops"
    },
    
    -- Scavenger Hunt webhook
    ScavengerHunts = {
        Enabled = false,
        WebhookURL = "YOUR DISCORD WEBHOOK HERE", -- your Discord webhook URL
        BotName = "Scavenger Hunts",
        BotIcon = "", -- optional image URL for the bot avatar
        Color = 65280, -- Green color in decimal (0x00FF00)
        Footer = "Poggy's Scavenger Hunts"
    }
}

-- Feature Toggles
Config.EnableSupplyDrops = true -- Enable or disable supply drops
Config.EnableScavengerHunts = true -- Enable or disable scavenger hunts

-- Supply Drops Configuration
Config.Supply = {
    -- Airdrop configuration
    UseAirdrops = true, -- Set to true to always use airdrops, false to never use them, or nil to use AirdropChance
    AirdropChance = 100, -- Percentage chance of a drop being an airdrop (if UseAirdrops is nil)
    UseBasket = false, -- Set to false to disable the basket and have just the balloon and item prop
    PropDistance = 6.0, -- Distance between balloon and prop (higher = lower prop position)
    BalloonHeight = 40.0, -- Height above ground where balloon starts
    DescentSpeed = 60.0, -- Seconds it takes to descend (lower = faster)
    HoverHeight = 8.0, -- Height above ground where balloon stops (prevents ground clipping)
    
    -- Blip settings
    Blip = {
        Sprite = "blip_ambient_crate",
        Color = "BLIP_MODIFIER_MP_COLOR_23", -- YELLOW_ORANGE
        Name = "Supply Drop",
    },
    
    -- How far away players can see the prompt (in units)
    PromptDistance = 2.0,
    
    -- How far away players can see the drop (optimization for rendering props)
    RenderDistance = 200.0,
    
    -- Time after which uncollected drops disappear (in milliseconds)
    DropTimeout = 900000, -- milliseconds: 900000 = 15 minutes
    
    -- Item drops configuration. Every item name must exist in your items table.
    ItemDrops = {
        {name = "coal", label = "Coal", amount = {5, 40}, chance = 20, prop = "p_cs_sackcoal01x"},
        {name = "apple", label = "Apple", amount = {5, 15}, chance = 25, prop = "p_crateapple01b"},
        {name = "alcohol", label = "Alcohol", amount = {1, 10}, chance = 5, prop = "p_barrelmoonshine"},
        {name = "iron", label = "Iron", amount = {10, 20}, chance = 10, prop = "mp001_p_mp_crateweapon_01a"},
        {name = "acid", label = "Acid", amount = {1, 10}, chance = 8, prop = "mp006_p_mp006_cratecanvase01x"},
        {name = "wood", label = "Soft Wood", amount = {1, 50}, chance = 20, prop = "mp004_mp_gfh_crategoods01x"},
        {name = "hwood", label = "Hard Wood", amount = {1, 50}, chance = 20, prop = "mp004_mp_gfh_crategoods01x"},
        {name = "copper", label = "Copper", amount = {1, 20}, chance = 20, prop = "mp001_p_mp_crateweapon_01a"},
        {name = "banana", label = "Banana", amount = {1, 50}, chance = 25, prop = "p_basketapple01x"},
        {name = "blueberry", label = "Blueberry", amount = {1, 50}, chance = 25, prop = "p_basketapple01x"},
        {name = "cherry", label = "Cherry", amount = {1, 50}, chance = 25, prop = "p_basketapple01x"},
        {name = "pumpkin", label = "Pumpkin", amount = {1, 50}, chance = 25, prop = "p_basketapple01x"},
        {name = "corn", label = "Corn", amount = {1, 50}, chance = 25, prop = "p_basketapple01x"},
        {name = "chocolate", label = "Chocolate", amount = {1, 50}, chance = 25, prop = "p_basketapple01x"},
        {name = "pork", label = "Pork", amount = {1, 30}, chance = 20, prop = "p_boxlrgmeat01x"},
        {name = "game", label = "Game Meat", amount = {1, 30}, chance = 20, prop = "p_boxlrgmeat01x"},
        {name = "biggame", label = "Big Game Meat", amount = {1, 30}, chance = 20, prop = "p_boxlrgmeat01x"},
        {name = "beef", label = "Beef", amount = {1, 30}, chance = 20, prop = "p_boxlrgmeat01x"},
        {name = "eggs", label = "Eggs", amount = {1, 40}, chance = 25, prop = "p_boxlrgmeat01x"},
        {name = "milk", label = "Milk", amount = {1, 40}, chance = 25, prop = "p_milkcan03x"},
        {name = "fibers", label = "Fibers", amount = {1, 40}, chance = 20, prop = "mp001_p_mp_crateweapon_01a"},
        {name = "wool", label = "Wool", amount = {1, 40}, chance = 20, prop = "mp001_p_mp_crateweapon_01a"},
        {name = "WEAPON_SNIPERRIFLE_CARCANO", label = "Carcano Rifle", amount = {1, 1}, chance = 1, prop = "mp001_p_mp_crateweapon_01a"},
        {name = "WEAPON_SNIPERRIFLE_ROLLINGBLOCK", label = "Rolling Block Rifle", amount = {1, 1}, chance = 1, prop = "mp001_p_mp_crateweapon_01a"},
        {name = "WEAPON_SHOTGUN_DOUBLEBARREL", label = "Double Barrel Shotgun", amount = {1, 1}, chance = 1, prop = "mp001_p_mp_crateweapon_01a"},
        {name = "WEAPON_PISTOL_M1899", label = "M1899 Pistol", amount = {1, 1}, chance = 1, prop = "mp001_p_mp_crateweapon_01a"},
        -- Example items: uncomment only if these exist in your items table
        -- {name = "golden_nugget", label = "Gold Nuggets", amount = {2, 25}, chance = 1, prop = "p_goldstack01x"},
        -- {name = "catfish_meat", label = "Catfish", amount = {1, 40}, chance = 20, prop = "p_fishbasin01x"},
    },
    
    -- Drop locations
    Locations = {
        {name = "Saint Denis Docks", location = vector3(2840.67, -1363.12, 44.56)},
        {name = "Rhodes Train Station", location = vector3(1223.37, -1281.76, 76.91)},
        {name = "Blackwater Docks", location = vector3(-725.92, -1272.91, 43.58)},
        {name = "Riggs Station", location = vector3(-1095.74, -569.51, 82.41)},
        {name = "Wallace Station", location = vector3(-1307.25, 401.34, 95.38)},
        {name = "Oil Fields", location = vector3(498.22, 664.28, 117.4)},
        {name = "Annesburg Train Station", location = vector3(2946.18, 1308.51, 44.49)},
        {name = "Bacchus Station", location = vector3(583.14, 1685.36, 187.67)},
        {name = "Armadillo Train Station", location = vector3(-3741.34, -2602.11, -13.23)},
        {name = "Emerald Train Station", location = vector3(1526.26, 439.02, 90.68)}, 
    }
}

-- Scavenger Hunt Configuration
Config.Scavenger = {
    
    -- UI Theme. Available themes: Crimson, Twilight, Oceanic, Forest, Monochrome
    Theme = "Oceanic", 

    -- Blip settings when a hunt is active
    Blip = {
        Sprite = "blip_treasure",
        Color = "BLIP_MODIFIER_MP_COLOR_8", -- GREEN
        Name = "Scavenger Hunt",
    },
    
    -- UI Settings
    UI = {
        -- Penalty percentage for revealing clue (0.0 to 1.0)
        CluePenalty = 0.3, -- 30% penalty for revealing the clue
        
        -- Default text for clue reveal button
        ShowClueText = "SHOW CLUE",
        
        -- Default text for the penalty warning
        PenaltyText = "Warning: Revealing the clue will reduce your reward by 30%"
    },
    
    -- How far away players can see the prompt (in units)
    PromptDistance = 2.0,
    
    -- How far away players can see the hunt object (optimization for rendering props)
    RenderDistance = 200.0,
    
    -- Time after which unclaimed hunts disappear (in milliseconds)
    HuntTimeout = 2700000, -- milliseconds: 2700000 = 45 minutes
    
    -- Hunt items configuration (can be money or items)
    RewardTypes = {
        {type = "money", currencyType = "cash", amount = {16.00, 80.00}, chance = 50}, -- Cash rewards
        {type = "item", name = "golden_nugget", label = "Gold Nuggets", amount = {10, 30}, chance = 10},
    }, -- Added comma here
    
    -- Scavenger Hunt Locations with props and images
    Hunts = {
        {
            name = "SCAVENGER HUNT", -- dead tree in Little Creek
            location = vector3(-2239.05, 609.08, 118.21),
            prop = "p_moneybag02x",
            image = "deadtree_littlecreek.jpg", -- Place in ui/images/
            clue = "Where water whispers beneath a bare trunk, search the tangle of green at its base."
        },
        {
            name = "SCAVENGER HUNT", -- Stillwater Creek
            location = vector3(-1810.9, -2399.67, 41.96),
            prop = "p_moneybag02x",
            image = "stillwatercreek.jpg", -- Place in ui/images/
            clue = "From Thieves’ Landing travel by rail, to creekside cabin down the winding trail. Beneath the deck where floorboards creak, your prize lies hidden in a burlap sack so meek."
        },
        {
            name = "SCAVENGER HUNT", -- Big Valley Reservation
            location = vector3(-2575.26, -89.08, 168.43),
            prop = "p_moneybag02x",
            image = "bigvalleyreservation.jpg", -- Place in ui/images/
            clue = "Along Owanjila’s quiet shore, a stony altar stands before. At its foot beneath the silent stone, a hidden pouch lies all alone."
        },
        {
            name = "SCAVENGER HUNT", -- Owanila Lake Dam
            location = vector3(-2321.95, -471.83, 142.59),
            prop = "p_moneybag02x",
            image = "owanjiladam.jpg", -- Place in ui/images/
            clue = "Beyond Strawberry, beside the stream, a wooden dam fulfills its dream. Lift up the plank where currents hum, and there you’ll find your prize to come."
        },
        {
            name = "SCAVENGER HUNT", -- New Austin Coal Mine in Gaptooth Ridge
            location = vector3(-5950.71, -3267.42, -21.82),
            prop = "p_moneybag02x",
            image = "gaptoothridgemine.jpg", -- Place in ui/images/
            clue = "At Gaptooth Ridge where red dust lies, an old coal tipple greets your eyes. By bent wood beams and iron rail, a hidden sack your prize unveils."
        },
        {
            name = "SCAVENGER HUNT", -- Annesburg Shipwreck Edge Map
            location = vector3(3586.68, 1412.69, 41.89),
            prop = "p_moneybag02x",
            image = "shipwreckannesburg.jpg", -- Place in ui/images/
            clue = "By barnacled beams and drifting foam, a battered hull becomes your home. Peer ‘neath the wood where waters meet, your treasured pouch lies safe and sweet."
        },
        {
            name = "SCAVENGER HUNT", -- Van Horn Dock House
            location = vector3(3004.7, 481.74, 44.66),
            prop = "p_moneybag02x",
            image = "vanhorndock.jpg", -- Place in ui/images/
            clue = "Where foghorn bells on houseboats toll, a cabin drifts upon the shoal. Beneath the floorboard close to beams, your treasure hides within the seams."
        },
        {
            name = "SCAVENGER HUNT", -- Gravestone in Lemoyne Swamp
            location = vector3(1992.25, -1895.53, 42.28), 
            prop = "p_moneybag02x",
            image = "gravestoneswamplemoyne.jpg", -- Place in ui/images/
            clue = "West of Saint Denis’ distant spires, in moss-clad mires where dusk retires. Atop the tombstone’s weathered face, your treasure hides in a solemn place."
        },
        {
            name = "SCAVENGER HUNT", -- Braithwaite Manor Trench Cannon
            location = vector3(1488.78, -1823.1, 54.0), 
            prop = "p_moneybag02x",
            image = "braithwaitetrenchcannon.jpg", -- Place in ui/images/
            clue = "At Bolger Blade where cannon lies, its iron wheel beneath gray skies. Beneath the spokes in rusted bed, your treasure waits just ahead."
        },
        {
            name = "SCAVENGER HUNT", -- Torture Basement in Valentine     
            location = vector3(-615.52, 532.8633422851562, 95.44000244140625), 
            prop = "p_moneybag02x",
            image = "torturebasementvalentine.jpg", -- Place in ui/images/
            clue = "Where masks of bone and candles burn, and every drip makes echoes turn, atop the cask where horrors meet, your treasure seeked lies discreet."
        },
        {
            name = "SCAVENGER HUNT", -- Manateca Falls Rock Island
            location = vector3(-1588.48, -2805.9, 40.96), 
            prop = "p_moneybag02x",
            image = "manatecafallsrockisland.jpg", -- Place in ui/images/
            clue = "By canyon walls of crimson hue, where echoes never still, a rugged isle, both old and new, awaits your cunning skill. Slide between two granite spines where earth and water blend, Your treasure lies beneath the lines, your journey at its end."
        },
        {
            name = "SCAVENGER HUNT", -- Bearclaw Cabins in the Well
            location = vector3(-2093.91, -1900.55, 105.75), 
            prop = "p_moneybag02x",
            image = "bearclawcabinwell.jpg", -- Place in ui/images/
            clue = "By Bearclaw cabins’ desolate air, a silent shaft, an open stare. Peer down its mouth where shadows lie, and lift your treasure from the eye."
        },
        {
            name = "SCAVENGER HUNT", -- Science Tower
            location = vector3(2516.82, 2297.34, 177.95), 
            prop = "p_moneybag02x",
            image = "sciencetower.jpg", -- Place in ui/images/
            clue = "In the tower where science reigns, beneath the stairs where knowledge gains. A hidden pouch in shadows deep, your treasure lies where secrets sleep."
        },
        {
            name = "SCAVENGER HUNT", -- Elysian Pool Waterfall in the hidden cave
            location = vector3(2286.73, 1068.02, 80.69), 
            prop = "p_moneybag02x",
            image = "elysianpool.jpg", -- Place in ui/images/
            clue = "Where eastern waters softly flow, behind the falls where secrets grow. In hidden cave where shadows creep, your treasure lies in silence deep."
        },
        {
            name = "SCAVENGER HUNT", -- Hanging Rock
            location = vector3(-3448.0, -2271.43, -0.05), 
            prop = "p_moneybag02x",
            image = "hangingrock.jpg", -- Place in ui/images/
            clue = "In desert land where shadows play, beneath the ledge where eagles sway. Hung by the rock where silence dwells, your treasure waits, its story tells."
        },
        {
            name = "SCAVENGER HUNT", -- Two Crows (on the well in New Austin)
            location = vector3(-3817.5, -2982.15, -5.06),
            prop = "p_moneybag02x",
            image = "twocrows.jpg", -- Place in ui/images/
            clue = "Where two dark sentinels survey the crimson dust below, a sunken shaft once held a macabre show Lean o’er its rim where shadows swell; at its mouth your treasure dwells."
        },
        {
            name = "SCAVENGER HUNT", -- Mount Shann "giant" bigfoot skeleton
            location = vector3(-1916.186767578125, -28.21517372131347, 287.8417663574219), 
            prop = "p_moneybag02x",
            image = "mountshanngiant.jpg", -- Place in ui/images/
            clue = "In the shadow of Mount Shann’s peak, where legends whisper and giants speak, a skeletal hand in earth is found, your treasure lies where bones abound."
        },
        {
            name = "SCAVENGER HUNT", -- Stuffed Gorilla ledge near the bridge
            location = vector3(-1739.37, 70.74, 155.75), 
            prop = "p_moneybag02x",
            image = "stuffedgorilla.jpg", -- Place in ui/images/
            clue = "East of Strawberry’s winding vein, a silent ape surveys the plain; Ascend the ledge of weathered stone, your secret treasure waits alone."
        },
        {
            name = "SCAVENGER HUNT", -- Witches Hut in Ambarino
            location = vector3(1183.61767578125, 2033.7552490234375, 324.5847473144531), 
            prop = "p_moneybag02x",
            image = "witcheshutamb.jpg", -- Place in ui/images/
            clue = "High in Grizzlies’ mist-clad trees, where potions steam on twisted knees. Upon the table ‘midst candle’s glow, your hidden treasure waits below."
        },
        {
            name = "SCAVENGER HUNT", -- Lemoyne Tiny Churh
            location = vector3(2414.265869140625, -738.834228515625, 42.10807800292969), 
            prop = "p_moneybag02x",
            image = "tinychurch.jpg", -- Place in ui/images/
            clue = "East of Lakay’s winding stream, a whitewashed house stands small, unseen. Upon its lone, worn pew you’ll find, the secret treasure you’ve surely mined."
        },
        {
            name = "SCAVENGER HUNT", -- Strawberry General Store Secret Moonshine Basement
            location = vector3(-1793.8092041015625, -386.64, 158.11000061035156), 
            prop = "p_moneybag02x",
            image = "moonshinestrawberrygeneral.jpg", -- Place in ui/images/
            clue = "Where Strawberry’s general store stands, a hidden cellar in the land. Beneath the floor where secrets brew, your treasure waits for you to view."
        },
        {
            name = "SCAVENGER HUNT", -- Crashed Airship location 
            location = vector3(-2567.5068359375, 744.05029296875, 156.06700134277344), 
            prop = "p_moneybag02x",
            image = "crashedairship.jpg", -- Place in ui/images/
            clue = "In the valley where the airship fell, a twisted wreck holds secrets to tell. On the rock where shadows creep, your treasure lies in silence deep."
        },
        {
            name = "SCAVENGER HUNT", -- Wooly Mammoth Bones - Ambarino
            location = vector3(-1733.22, 2167.27, 290.26), 
            prop = "p_moneybag02x",
            image = "woolymammothbones.jpg", -- Place in ui/images/
            clue = "In the snowy peaks where giants tread, a mammoth’s bones lie long since dead. On the stone where silence reigns, your treasure waits in icy chains."
        },
        {
            name = "SCAVENGER HUNT", -- Wapiti Reservation Oil Rig
            location = vector3(458.5663146972656, 2255.884033203125, 249.32864379882812), 
            prop = "p_moneybag02x",
            image = "wapitioilrig.jpg", -- Place in ui/images/
            clue = "Where native air softly blows, a rig stands in twilight’s glow. Beneath the platform where shadows play, your hidden treasure waits today."
        },
        {
            name = "SCAVENGER HUNT", -- Mount Hagen Mine on the Water Tower
            location = vector3(-1876.3646240234375, 1359.4229736328125, 211.50645446777344), 
            prop = "p_moneybag02x",
            image = "mounthagenwatertower.jpg", -- Place in ui/images/
            clue = "In the mine where shadows loom, a fluid spire holds your doom. Climb its rungs to reach the top, your treasure waits where echoes stop."
        },
        {
            name = "SCAVENGER HUNT", -- Plainview Campsite in New Austin on the Oil Rig
            location = vector3(-4687.2236328125, -3738.68896484375, 13.1964693069458), 
            prop = "p_moneybag02x",
            image = "plainviewoilrig.jpg", -- Place in ui/images/
            clue = "In the desert where the oil flows, a rig stands tall where the cactus grows. In plain view, there's a secret prize. Your treasure waits where the sun does rise."
        },
        {
            name = "SCAVENGER HUNT", -- Beneath Bard's Crossing Bridge 
            location = vector3(-813.755126953125, -584.883544921875, 55.94433212280273), 
            prop = "p_moneybag02x",
            image = "bardscrossingbridge.jpg", -- Place in ui/images/
            clue = "On solid ground where bards do cross, a cross stands tall, no gain, no loss. Beneath its arch where shadows play, your treasure waits in the light of day."
        },
        {
            name = "SCAVENGER HUNT", -- Van Horn Abandoned Train Station
            location = vector3(2890.333984375, 618.2111206054688, 57.78700637817383), 
            prop = "p_moneybag02x",
            image = "vanhorntrainstation.jpg", -- Place in ui/images/
            clue = "In a lawless land where they once ran, a structure rots where the tracks began. Beneath the platform where boards all creak, your treasure lies where no one speaks."
        },
        {
            name = "SCAVENGER HUNT", -- Tallest peak of Three Sisters Mountain
            location = vector3(1497.4365234375, 1164.3924560546875, 240.39236450195312), 
            prop = "p_moneybag02x",
            image = "threesisterspeak.jpg", -- Place in ui/images/
            clue = "The tallest peak of sisters three, it really is a sight to see. Climb to the top where the eagles soar, your treasure waits forevermore."
        },
        {
            name = "SCAVENGER HUNT", -- Gravesite by Fort Wallace
            location = vector3(317.4997863769531, 1470.9918212890625, 179.5106964111328), 
            prop = "p_moneybag02x",
            image = "fortwallacegravesite.jpg", -- Place in ui/images/
            clue = "At the place of soldier's passed, a fortress stands, its glory vast. Beside the stone where men did fall, your treasure waits, a silent call."
        },
    }
}

-- Player-facing messages (and their translations) live in translations.lua.
-- Notifications are drawn by poggy_core from client/notifications.lua.
