Config = {}

Config.Debug = false

-- Command to open the menu
Config.Command = 'multijob'
Config.CommandDescription = 'Open Multijob Menu'

-- Command to quick switch job
Config.SwitchCommand = 'mj'
Config.SwitchCommandDescription = 'Quick switch job (e.g. /mj 1)'

-- Max jobs a player can have
Config.MaxJobs = 5

-- Default job when quitting all jobs
Config.DefaultJob = 'unemployed'
Config.DefaultGrade = 0
Config.DefaultJobLabel = 'Unemployed'

-- Webhook for logging
Config.Webhook = "YOUR DISCORD WEBHOOK HERE"

-- Admin Groups that can access the admin menu
Config.AdminGroups = {
    'admin',
    'superadmin',
    'moderator'
}

-- Job Presets for Admin Panel
-- Categories: police, medical, business, shops
-- `job` must match a job name on your server exactly (VORP job names are case-sensitive).
-- The law and medical presets use the stock job names: police, sheriff, marshal, doctor.
Config.JobPresets = {
    police = {
        name = "Law Enforcement",
        jobs = {
            -- Police
            { job = "police", label = "CHIEF OF POLICE", grade = 3, category = "Police" },
            { job = "police", label = "SERGEANT", grade = 2, category = "Police" },
            { job = "police", label = "OFFICER", grade = 1, category = "Police" },
            { job = "police", label = "RECRUIT", grade = 0, category = "Police" },

            -- Sheriff
            { job = "sheriff", label = "SHERIFF", grade = 6, category = "Sheriff" },
            { job = "sheriff", label = "UNDERSHERIFF", grade = 5, category = "Sheriff" },
            { job = "sheriff", label = "LIEUTENANT", grade = 4, category = "Sheriff" },
            { job = "sheriff", label = "SERGEANT", grade = 3, category = "Sheriff" },
            { job = "sheriff", label = "DETECTIVE", grade = 2, category = "Sheriff" },
            { job = "sheriff", label = "DEPUTY", grade = 1, category = "Sheriff" },
            { job = "sheriff", label = "RECRUIT", grade = 0, category = "Sheriff" },

            -- Marshal
            { job = "marshal", label = "CHIEF MARSHAL", grade = 5, category = "Marshal" },
            { job = "marshal", label = "DEPUTY MARSHAL", grade = 0, category = "Marshal" },
        }
    },
    medical = {
        name = "Medical",
        jobs = {
            -- Doctor
            { job = "doctor", label = "CHIEF OF MEDICINE", grade = 7, category = "Doctor" },
            { job = "doctor", label = "DEPUTY CHIEF OF MEDICINE", grade = 6, category = "Doctor" },
            { job = "doctor", label = "CHIEF MEDICAL OFFICER", grade = 5, category = "Doctor" },
            { job = "doctor", label = "SENIOR DOCTOR", grade = 4, category = "Doctor" },
            { job = "doctor", label = "DOCTOR", grade = 3, category = "Doctor" },
            { job = "doctor", label = "FELLOW", grade = 2, category = "Doctor" },
            { job = "doctor", label = "RESIDENT", grade = 1, category = "Doctor" },
            { job = "doctor", label = "MEDICAL STUDENT", grade = 0, category = "Doctor" },
        }
    },
    business = {
        name = "Business Licenses",
        jobs = {
            { job = "miner", label = "Miner", grade = 0, category = "Trade Licenses" },
            { job = "lumberjack", label = "Lumberjack", grade = 0, category = "Trade Licenses" },
            { job = "blacksmith", label = "Blacksmith", grade = 0, category = "Trade Licenses" },
            { job = "distiller", label = "Distiller", grade = 0, category = "Trade Licenses" },
            { job = "horsetrainer", label = "Horse Trainer", grade = 0, category = "Animal Trades" },
            { job = "horsebreeder", label = "Horse Breeder", grade = 0, category = "Animal Trades" },
            { job = "wheelwright", label = "Wheelwright", grade = 0, category = "Trade Licenses" },
        }
    },
    -- The shop-owner jobs below are EXAMPLES, not stock VORP jobs. Rename them to
    -- match the jobs in your own jobs table, or delete the ones you do not use.
    shops = {
        name = "Shop Owners",
        jobs = {
            -- Stables
            { job = "emerstableowner", label = "Emerald Stable", grade = 3, category = "Stables" },
            { job = "valstableowner", label = "Valentine Stable", grade = 3, category = "Stables" },
            { job = "blacksmithowner", label = "Blacksmith Stable", grade = 3, category = "Stables" },
            { job = "scarmeadstableowner", label = "Scarlett Meadows Stable", grade = 3, category = "Stables" },
            { job = "stdenstableowner", label = "Saint Denis Stable", grade = 3, category = "Stables" },
            { job = "armstableowner", label = "Armadillo Stable", grade = 3, category = "Stables" },
            { job = "rhostableowner", label = "Rhodes Stable", grade = 3, category = "Stables" },
            { job = "strawstableowner", label = "Strawberry Stable", grade = 3, category = "Stables" },
            { job = "tumstableowner", label = "Tumbleweed Stable", grade = 3, category = "Stables" },
            { job = "bwstableowner", label = "Blackwater Stable", grade = 3, category = "Stables" },
            -- Gunsmiths
            { job = "gunsmithBW", label = "Blackwater Gunsmith", grade = 3, category = "Gunsmiths" },
            { job = "gunsmithV", label = "Valentine Gunsmith", grade = 3, category = "Gunsmiths" },
            { job = "gunsmithS", label = "Saint Denis Gunsmith", grade = 3, category = "Gunsmiths" },
            { job = "gunsmithR", label = "Rhodes Gunsmith", grade = 3, category = "Gunsmiths" },
            { job = "gunsmithT", label = "Tumbleweed Gunsmith", grade = 3, category = "Gunsmiths" },
            { job = "gunsmithA", label = "Annesburg Gunsmith", grade = 3, category = "Gunsmiths" },
            -- General Stores
            { job = "generalstoreBW", label = "Blackwater General Store", grade = 3, category = "General Stores" },
            { job = "generalstoreV", label = "Valentine General Store", grade = 3, category = "General Stores" },
            { job = "generalstoreSD", label = "Saint Denis General Store", grade = 3, category = "General Stores" },
            { job = "generalstoreR", label = "Rhodes General Store", grade = 3, category = "General Stores" },
            { job = "generalstoreT", label = "Tumbleweed General Store", grade = 3, category = "General Stores" },
            { job = "generalstoreAr", label = "Armadillo General Store", grade = 3, category = "General Stores" },
            { job = "generalstoreAn", label = "Annesburg General Store", grade = 3, category = "General Stores" },
            { job = "generalstoreE", label = "Emerald General Store", grade = 3, category = "General Stores" },
            { job = "generalstoreS", label = "Strawberry General Store", grade = 3, category = "General Stores" },
            { job = "generalstoreVH", label = "Van Horn General Store", grade = 3, category = "General Stores" },
            { job = "generalstoreW", label = "Wapiti General Store", grade = 3, category = "General Stores" },
            -- Saloons
            { job = "saloonBW", label = "Blackwater Saloon", grade = 3, category = "Saloons" },
            { job = "saloonV", label = "Valentine Saloon", grade = 3, category = "Saloons" },
            { job = "saloonSD", label = "Saint Denis Saloon", grade = 3, category = "Saloons" },
            { job = "saloonR", label = "Rhodes Saloon", grade = 3, category = "Saloons" },
            { job = "saloonT", label = "Tumbleweed Saloon", grade = 3, category = "Saloons" },
            { job = "saloonA", label = "Armadillo Saloon", grade = 3, category = "Saloons" },
            { job = "saloonE", label = "Emerald Saloon", grade = 3, category = "Saloons" },
            { job = "saloonAn", label = "Annesburg Saloon", grade = 3, category = "Saloons" },
            -- Blacksmiths
            { job = "blacksmithE", label = "Emerald Blacksmith", grade = 3, category = "Blacksmith Shops" },
            { job = "blacksmithBW", label = "Blackwater Blacksmith", grade = 3, category = "Blacksmith Shops" },
            { job = "blacksmithV", label = "Valentine Blacksmith", grade = 3, category = "Blacksmith Shops" },
            { job = "blacksmithSD", label = "Saint Denis Blacksmith", grade = 3, category = "Blacksmith Shops" },
            { job = "blacksmithR", label = "Rhodes Blacksmith", grade = 3, category = "Blacksmith Shops" },
            { job = "blacksmithS", label = "Strawberry Blacksmith", grade = 3, category = "Blacksmith Shops" },
            { job = "blacksmithA", label = "Armadillo Blacksmith", grade = 3, category = "Blacksmith Shops" },
            { job = "blacksmithAn", label = "Annesburg Blacksmith", grade = 3, category = "Blacksmith Shops" },
        }
    }
}
