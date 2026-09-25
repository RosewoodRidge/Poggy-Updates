-- ############################################################
-- ##  LANGUAGE SELECTION #####################################
-- ############################################################
Config = Config or {}
Config.Language = "en"   -- "en", "es" or "fr"

-- ############################################################
-- ##  TRANSLATIONS TABLE #####################################
-- ############################################################
Translations = {
    ------------------------------------------------------------------
    -- ENGLISH -------------------------------------------------------
    ------------------------------------------------------------------
    en = {
        -- General / existing keys
        SOMEONE_SAW                 = "Someone saw what you did!",
        NO_ONE_SAW                  = "No one saw what you did!",
        ALL_STOPPED                 = "All witnesses have been stopped!",
        MULTIPLE_PEOPLE_SAW         = "%s people saw what you did!",
        CLEANED_UP_BLIPS_SINGULAR   = "Cleaned up 1 alert blip.",
        CLEANED_UP_BLIPS_PLURAL     = "Cleaned up %s alert blips.",
        ONE_ESCAPED                 = "One of the witnesses escaped!",
        ONE_PERSON_SAW              = "Someone saw what you did!",
        WITNESS_TITLE               = "WITNESS",
        WITNESSES_TITLE             = "WITNESSES",
        ALERT_SYSTEM_TITLE          = "Alert System",
        ALERT_TITLE                 = "Alert",
        ALERT_COOLDOWN              = "You must wait %s before using /%s again",  
        -- NEW alert-specific keys
        ALERT_SHOOTING_NAME                    = "SOMEONE HAS BEEN SHOT!",
        ALERT_SHOOTING_MESSAGE                 = "A scallywag was seen a-shootin'! Best investigate, lawman!",
        ALERT_SHOOTING_ALERTER_NOTIFICATION    = "Your six-shooter spree got eyes on it! The law's comin'!",

        ALERT_MELEE_NAME                       = "SOMEONE IS FIGHTING!",
        ALERT_MELEE_MESSAGE                    = "Heard tell of a dust-up! Go sort 'em out, lawman!",
        ALERT_MELEE_ALERTER_NOTIFICATION       = "Your roughhousing raised an alarm! Lawmen coming!",

        ALERT_LASSOING_NAME                    = "SOMEONE IS HOGTIED!",
        ALERT_LASSOING_MESSAGE                 = "A body's been hogtied! Rustle up and check it, Marshal!",
        ALERT_LASSOING_ALERTER_NOTIFICATION    = "Your fancy rope work didn't go unnoticed! Expect law!",

        ALERT_TRAMPLING_NAME                   = "TRAMPLING!",
        ALERT_TRAMPLING_MESSAGE                = "Someone's been run down by a nag! See to it, lawman!",
        ALERT_TRAMPLING_ALERTER_NOTIFICATION   = "You ran someone over! The law's been called!",

        ALERT_THREATENING_NAME                 = "SOMEONE IS THREATENED!",
        ALERT_THREATENING_MESSAGE              = "Someone's being threatened with violence!",
        ALERT_THREATENING_ALERTER_NOTIFICATION = "You can't just point a gun at folks! The law's on the way!",

        ALERT_HIJACKING_NAME                   = "SOMEONE WAS HIJACKED!",
        ALERT_HIJACKING_MESSAGE                = "Someone's been thrown from their horse or wagon! Check on it, lawman!",
        ALERT_HIJACKING_ALERTER_NOTIFICATION   = "You can't just steal horses and wagons! The law's on the way!",

        ALERT_CARRYING_HOSTAGE_NAME                   = "HOSTAGE ON HORSEBACK!",
        ALERT_CARRYING_HOSTAGE_MESSAGE                = "Someone's been spotted riding with a bound captive on their horse! Intercept them, lawman!",
        ALERT_CARRYING_HOSTAGE_ALERTER_NOTIFICATION   = "Folks saw you riding with a hogtied soul on your horse! The law's been called!",

        -- Law Response System
        LAW_SYSTEM_TITLE              = "Law System",
        LAW_RESET_STATE               = "Law response state has been reset",
        LAW_REMOTE_LOCATION           = "Law officers cannot reach this remote location",
        LAW_ESCAPED                   = "You escaped from law enforcement",
        LAW_DEFEATED                  = "You defeated all law officers",
        LAW_OFFICER_NAME              = "Lawman",
        
        -- Law Notifications
        LAW_RESPONSE_SAINT_DENIS      = "Saint Denis Police have been alerted",
        LAW_RESPONSE_RHODES           = "Rhodes Sheriff's Department has been notified",
        LAW_RESPONSE_VALENTINE        = "Valentine Sheriff is on the way",
        LAW_RESPONSE_STRAWBERRY       = "Strawberry Sheriff has been notified",
        LAW_RESPONSE_BLACKWATER       = "Blackwater Sheriff's Department is responding",
        LAW_RESPONSE_TUMBLEWEED       = "Tumbleweed Sheriff's Department is on the move",
        LAW_RESPONSE_ARMADILLO        = "Armadillo Sheriff has been notified",
        LAW_RESPONSE_ANNESBURG        = "Annesburg Sheriff is responding",
        LAW_RESPONSE_WILDERNESS       = "Bounty Hunters have been dispatched",

        -- Law Response Notifications
        LAW_DEFEATED_TITLE   = "LAW DEFEATED",
        LAW_DEFEATED_MESSAGE = "You have defeated all the lawmen!",
        LAW_ESCAPED_TITLE    = "ESCAPED LAW",
        LAW_ESCAPED_MESSAGE  = "You've escaped from the lawmen.",

        -- NPC arrest (surrender)
        LAW_ARRESTED_TITLE   = "ARRESTED",
        LAW_ARRESTED_MESSAGE = "You have been arrested by the law",
        LAW_JAILED           = "You have been arrested and jailed for %s minutes",
        -- The built-in Sisika jail (1.4.0)
        JAIL_SENTENCED       = "You have been sent to Sisika Penitentiary for %s minutes",
        JAIL_RESUMED         = "Your sentence in Sisika continues: %s left",
        JAIL_RELEASED        = "You have been released from Sisika Penitentiary. You are free to go.",
        JAIL_ESCAPED         = "You escaped from Sisika Penitentiary!",
        JAIL_CANT_LEAVE      = "You cannot leave the prison grounds!",
        JAIL_OUT_OF_BOUNDS   = "You are past the prison walls. Keep going and you are an escaped convict!",
        JAIL_TIMER           = "Sisika Penitentiary: %s left",
        JAIL_ESCAPE_ALERT_NAME = "PRISON BREAK!",
        JAIL_ESCAPE_ALERT_MESSAGE = "A convict has escaped from Sisika Penitentiary! Round them up, lawman!",

        -- Crimes added in 1.4.0
        ALERT_LOOTING_NAME                  = "SOMEONE IS LOOTING A BODY!",
        ALERT_LOOTING_MESSAGE               = "Someone was seen going through a dead man's pockets! Look into it, lawman!",
        ALERT_LOOTING_ALERTER_NOTIFICATION  = "Folks saw you rob the dead! The law's been told!",
        ALERT_POACHING_NAME                 = "POACHING!",
        ALERT_POACHING_MESSAGE              = "Someone's been killing protected game! Track them down, lawman!",
        ALERT_POACHING_ALERTER_NOTIFICATION = "Someone saw you poaching! The law's on the way!",
        -- Alert details and alerts from other scripts (1.4.0)
        ALERT_WHERE_IN                      = "In %s",
        ALERT_WHERE_NEAR                    = "Near %s",
        ALERT_WHERE_WILDS                   = "Out in the wilds",
        ALERT_REPORTED_AT                   = "reported at %s",
        ALERT_DEFAULT_NAME                  = "LAW ALERT",
        ALERT_DEFAULT_MESSAGE               = "A crime has been reported. Respond to the location.",
        ALERT_DEFAULT_ALERTER               = "Someone saw you! The law has been called!",
        NOBODY_NOTICED                      = "Nothing happened. No one noticed.",
        ALERT_ARRIVED                       = "You have arrived at the alert location.",
        WAYPOINT_CLEARED                    = "Waypoint cleared.",
        MARKER_CLEARED                      = "Marker cleared.",
        ALERTS_CLEARED                      = "Cleared %s alert(s) and waypoints.",
        ALERTS_ALL_CLEARED                  = "All alerts and waypoints cleared.",
        TIME_MIN_SEC                        = "%d min %d s",
        TIME_SEC                            = "%d s",
        -- NPC law: warnings and surrender (1.4.0)
        LAW_WARNED                          = "The lawmen want you to surrender! Put your hands up or run.",
        LAW_SURRENDER_GROUP                 = "Lawmen",
        LAW_SURRENDER_PROMPT                = "Surrender",
        LAW_SURRENDER_TITLE                 = "SURRENDER",
        LAW_SURRENDERING                    = "You are surrendering to the law",
        LAW_SURRENDER_ACCEPTED              = "The lawmen have accepted your surrender",
        LAW_SURRENDER_BROKEN_SHOT           = "You broke your surrender by shooting!",
        LAW_SURRENDER_BROKEN_MOVED          = "You broke your surrender by moving!",
        ALERT_NOBODY_TO_ANSWER              = "Word got out, but there is no law on duty to answer.",
        LAW_SURRENDER_BROKEN_DREW           = "You broke your surrender by drawing a weapon!",
        LAW_SURRENDER_ARREST_FAILED         = "The lawmen could not get to you.",
        LAW_RELEASED_TITLE                  = "RELEASED",
        LAW_RELEASED_MESSAGE                = "The deputies let you go with a warning. Don't let them catch you again.",
        WITNESS_BLIP                        = "Witness"
    },

    ------------------------------------------------------------------
    -- ESPAÑOL -------------------------------------------------------
    ------------------------------------------------------------------
    es = {
        -- General / existing keys
        SOMEONE_SAW                 = "¡Alguien vio lo que hiciste!",
        NO_ONE_SAW                  = "¡Nadie vio lo que hiciste!",
        ALL_STOPPED                 = "¡Todos los testigos han sido detenidos!",
        MULTIPLE_PEOPLE_SAW         = "¡%s personas vieron lo que hiciste!",
        CLEANED_UP_BLIPS_SINGULAR   = "Se limpió 1 blip de alerta.",
        CLEANED_UP_BLIPS_PLURAL     = "Se limpiaron %s blips de alerta.",
        ONE_ESCAPED                 = "¡Uno de los testigos escapó!",
        ONE_PERSON_SAW              = "¡Alguien vio lo que hiciste!",
        WITNESS_TITLE               = "TESTIGO",
        WITNESSES_TITLE             = "TESTIGOS",
        ALERT_SYSTEM_TITLE          = "Sistema de Alerta",
        ALERT_TITLE                 = "Alerta",
        ALERT_COOLDOWN              = "Debes esperar %s antes de usar /%s nuevamente",  
        -- NUEVAS claves de alerta
        ALERT_SHOOTING_NAME                    = "¡ALGUIEN HA SIDO DISPARADO!",
        ALERT_SHOOTING_MESSAGE                 = "¡Se ha visto a un bandido disparando! ¡Investiga, sheriff!",
        ALERT_SHOOTING_ALERTER_NOTIFICATION    = "¡Tu tiroteo fue visto! ¡La ley se acerca!",

        ALERT_MELEE_NAME                       = "¡ALGUIEN ESTÁ PELEANDO!",
        ALERT_MELEE_MESSAGE                    = "¡Se oyó una pelea! ¡Pon orden, alguacil!",
        ALERT_MELEE_ALERTER_NOTIFICATION       = "¡Tus golpes llamaron la atención! ¡La ley viene!",

        ALERT_LASSOING_NAME                    = "¡ALGUIEN ESTÁ ATRAPADO!",
        ALERT_LASSOING_MESSAGE                 = "¡Hay alguien amarrado! ¡Compruébalo, marshal!",
        ALERT_LASSOING_ALERTER_NOTIFICATION    = "¡Tu juego de cuerdas no pasó desapercibido! ¡Espera a la ley!",

        ALERT_TRAMPLING_NAME                   = "¡ATROPELLO!",
        ALERT_TRAMPLING_MESSAGE                = "¡Alguien fue arrollado por un caballo! ¡Revísalo, sheriff!",
        ALERT_TRAMPLING_ALERTER_NOTIFICATION   = "¡Atropellaste a alguien! ¡La ley ha sido avisada!",

        ALERT_THREATENING_NAME                 = "¡ALGUIEN ESTÁ AMENAZADO!",
        ALERT_THREATENING_MESSAGE              = "¡Alguien está siendo amenazado con violencia!",
        ALERT_THREATENING_ALERTER_NOTIFICATION = "¡No puedes apuntar armas así! ¡La ley viene!",

        ALERT_HIJACKING_NAME                   = "¡ALGUIEN FUE DESMONTADO!",
        ALERT_HIJACKING_MESSAGE                = "¡Arrojaron a alguien de su caballo o carruaje! ¡Revísalo, sheriff!",
        ALERT_HIJACKING_ALERTER_NOTIFICATION   = "¡No puedes robar caballos ni carruajes! ¡La ley viene!",

        ALERT_CARRYING_HOSTAGE_NAME                   = "¡REHÉN A CABALLO!",
        ALERT_CARRYING_HOSTAGE_MESSAGE                = "¡Alguien fue visto montando con un prisionero atado en su caballo! ¡Interceéptalo, sheriff!",
        ALERT_CARRYING_HOSTAGE_ALERTER_NOTIFICATION   = "¡La gente te vio montar con alguien amarrado! ¡La ley ha sido avisada!",

        -- Sistema de Respuesta Policial
        LAW_SYSTEM_TITLE              = "Sistema de la Ley",
        LAW_RESET_STATE               = "El estado de respuesta de la ley ha sido restablecido",
        LAW_REMOTE_LOCATION           = "Los agentes de la ley no pueden llegar a esta ubicación remota",
        LAW_ESCAPED                   = "Has escapado de las fuerzas del orden",
        LAW_DEFEATED                  = "Derrotaste a todos los agentes de la ley",
        LAW_OFFICER_NAME              = "Alguacil",
        
        -- Notificaciones de la Ley
        LAW_RESPONSE_SAINT_DENIS      = "La Policía de Saint Denis ha sido alertada",
        LAW_RESPONSE_RHODES           = "La Oficina del Sheriff de Rhodes ha sido notificada",
        LAW_RESPONSE_VALENTINE        = "El Sheriff de Valentine está en camino",
        LAW_RESPONSE_STRAWBERRY       = "El Sheriff de Strawberry ha sido notificado",
        LAW_RESPONSE_BLACKWATER       = "La Oficina del Sheriff de Blackwater está respondiendo",
        LAW_RESPONSE_TUMBLEWEED       = "La Oficina del Sheriff de Tumbleweed está en movimiento",
        LAW_RESPONSE_ARMADILLO        = "El Sheriff de Armadillo ha sido notificado",
        LAW_RESPONSE_ANNESBURG        = "El Sheriff de Annesburg está respondiendo",
        LAW_RESPONSE_WILDERNESS       = "Los Cazarrecompensas han sido despachados",

        -- Notificaciones de Respuesta de la Ley
        LAW_DEFEATED_TITLE   = "LEY DERROTADA",
        LAW_DEFEATED_MESSAGE = "¡Has derrotado a todos los agentes de la ley!",
        LAW_ESCAPED_TITLE    = "LEY ESCAPADA",
        LAW_ESCAPED_MESSAGE  = "¡Has escapado de los agentes de la ley!",

        -- Arresto por la ley NPC (rendición)
        LAW_ARRESTED_TITLE   = "ARRESTADO",
        LAW_ARRESTED_MESSAGE = "Has sido arrestado por la ley",
        LAW_JAILED           = "Has sido arrestado y encarcelado durante %s minutos",
        -- The built-in Sisika jail (1.4.0)
        JAIL_SENTENCED       = "Has sido enviado a la penitenciaría de Sisika durante %s minutos",
        JAIL_RESUMED         = "Tu condena en Sisika continúa: quedan %s",
        JAIL_RELEASED        = "Has salido de la penitenciaría de Sisika. Eres libre.",
        JAIL_ESCAPED         = "¡Te has fugado de la penitenciaría de Sisika!",
        JAIL_CANT_LEAVE      = "¡No puedes salir del recinto de la prisión!",
        JAIL_OUT_OF_BOUNDS   = "Has cruzado los muros de la prisión. ¡Si sigues, serás un fugitivo!",
        JAIL_TIMER           = "Penitenciaría de Sisika: quedan %s",
        JAIL_ESCAPE_ALERT_NAME = "¡FUGA DE LA PRISIÓN!",
        JAIL_ESCAPE_ALERT_MESSAGE = "¡Un preso se ha fugado de la penitenciaría de Sisika! ¡Atrápalo, agente!",

        -- Crímenes nuevos en 1.4.0
        ALERT_LOOTING_NAME                  = "¡ALGUIEN SAQUEA UN CADÁVER!",
        ALERT_LOOTING_MESSAGE               = "¡Vieron a alguien registrando los bolsillos de un muerto! ¡Investígalo, sheriff!",
        ALERT_LOOTING_ALERTER_NOTIFICATION  = "¡La gente te vio robar a los muertos! ¡La ley ha sido avisada!",
        ALERT_POACHING_NAME                 = "¡CAZA FURTIVA!",
        ALERT_POACHING_MESSAGE              = "¡Alguien está matando animales protegidos! ¡Encuéntralo, sheriff!",
        ALERT_POACHING_ALERTER_NOTIFICATION = "¡Alguien te vio cazando furtivamente! ¡La ley viene!",
        -- Detalles de alerta y alertas de otros scripts (1.4.0)
        ALERT_WHERE_IN                      = "En %s",
        ALERT_WHERE_NEAR                    = "Cerca de %s",
        ALERT_WHERE_WILDS                   = "En tierras salvajes",
        ALERT_REPORTED_AT                   = "avisado a las %s",
        ALERT_DEFAULT_NAME                  = "ALERTA DE LA LEY",
        ALERT_DEFAULT_MESSAGE               = "Se ha denunciado un crimen. Acude al lugar.",
        ALERT_DEFAULT_ALERTER               = "¡Alguien te vio! ¡Han llamado a la ley!",
        NOBODY_NOTICED                      = "No pasó nada. Nadie se dio cuenta.",
        ALERT_ARRIVED                       = "Has llegado al lugar de la alerta.",
        WAYPOINT_CLEARED                    = "Punto de ruta borrado.",
        MARKER_CLEARED                      = "Marcador borrado.",
        ALERTS_CLEARED                      = "Se borraron %s alerta(s) y puntos de ruta.",
        ALERTS_ALL_CLEARED                  = "Todas las alertas y puntos de ruta borrados.",
        TIME_MIN_SEC                        = "%d min %d s",
        TIME_SEC                            = "%d s",
        -- Ley NPC: avisos y rendición (1.4.0)
        LAW_WARNED                          = "¡Los agentes quieren que te rindas! Levanta las manos o huye.",
        LAW_SURRENDER_GROUP                 = "Agentes de la ley",
        LAW_SURRENDER_PROMPT                = "Rendirse",
        LAW_SURRENDER_TITLE                 = "RENDICIÓN",
        LAW_SURRENDERING                    = "Te estás rindiendo a la ley",
        LAW_SURRENDER_ACCEPTED              = "Los agentes han aceptado tu rendición",
        LAW_SURRENDER_BROKEN_SHOT           = "¡Rompiste tu rendición al disparar!",
        LAW_SURRENDER_BROKEN_MOVED          = "¡Rompiste tu rendición al moverte!",
        ALERT_NOBODY_TO_ANSWER              = "Se corrió la voz, pero no hay ley de servicio para responder.",
        LAW_SURRENDER_BROKEN_DREW           = "¡Rompiste tu rendición al sacar un arma!",
        LAW_SURRENDER_ARREST_FAILED         = "Los agentes no pudieron llegar hasta ti.",
        LAW_RELEASED_TITLE                  = "LIBERADO",
        LAW_RELEASED_MESSAGE                = "Los ayudantes te dejan ir con una advertencia. Que no te vuelvan a atrapar.",
        WITNESS_BLIP                        = "Testigo"
    },

    ------------------------------------------------------------------
    -- FRANÇAIS ------------------------------------------------------
    ------------------------------------------------------------------
    fr = {
        -- Général / clés existantes
        SOMEONE_SAW                 = "Quelqu'un a vu ce que vous avez fait !",
        NO_ONE_SAW                  = "Personne n'a vu ce que vous avez fait !",
        ALL_STOPPED                 = "Tous les témoins ont été arrêtés !",
        MULTIPLE_PEOPLE_SAW         = "%s personnes ont vu ce que vous avez fait !",
        CLEANED_UP_BLIPS_SINGULAR   = "1 blip d'alerte nettoyé.",
        CLEANED_UP_BLIPS_PLURAL     = "%s blips d'alerte nettoyés.",
        ONE_ESCAPED                 = "Un des témoins s'est échappé !",
        ONE_PERSON_SAW              = "Quelqu'un a vu ce que vous avez fait !",
        WITNESS_TITLE               = "TÉMOIN",
        WITNESSES_TITLE             = "TÉMOINS",
        ALERT_SYSTEM_TITLE          = "Système d'Alerte",
        ALERT_TITLE                 = "Alerte",
        ALERT_COOLDOWN              = "Vous devez attendre %s avant d'utiliser /%s à nouveau",  
        -- NOUVELLES clés d'alerte
        ALERT_SHOOTING_NAME                    = "QUELQU'UN A ÉTÉ TOUCHÉ PAR BALLE !",
        ALERT_SHOOTING_MESSAGE                 = "On a vu un vaurien tirer ! Allez enquêter, shérif !",
        ALERT_SHOOTING_ALERTER_NOTIFICATION    = "Votre fusillade n'est pas passée inaperçue ! La loi arrive !",

        ALERT_MELEE_NAME                       = "QUELQU'UN SE BAT !",
        ALERT_MELEE_MESSAGE                    = "On parle d'une rixe ! Allez mettre de l'ordre, shérif !",
        ALERT_MELEE_ALERTER_NOTIFICATION       = "Votre bagarre a fait du bruit ! Les autorités arrivent !",

        ALERT_LASSOING_NAME                    = "QUELQU'UN EST LIGOTÉ !",
        ALERT_LASSOING_MESSAGE                 = "Un individu a été ligoté ! Vérifiez, marshal !",
        ALERT_LASSOING_ALERTER_NOTIFICATION    = "Votre tour de corde a été remarqué ! Attendez la loi !",

        ALERT_TRAMPLING_NAME                   = "PIÉTINEMENT !",
        ALERT_TRAMPLING_MESSAGE                = "Quelqu'un a été piétiné par un canasson ! Allez voir, shérif !",
        ALERT_TRAMPLING_ALERTER_NOTIFICATION   = "Vous avez renversé quelqu'un ! La loi a été prévenue !",

        ALERT_THREATENING_NAME                 = "QUELQU'UN EST MENACÉ !",
        ALERT_THREATENING_MESSAGE              = "Quelqu'un est menacé de violence !",
        ALERT_THREATENING_ALERTER_NOTIFICATION = "On ne pointe pas son arme sur les gens ! La loi arrive !",

        ALERT_HIJACKING_NAME                   = "VOL DE MONTURE !",
        ALERT_HIJACKING_MESSAGE                = "On a jeté quelqu'un de son cheval ou de sa charrette ! Allez voir, shérif !",
        ALERT_HIJACKING_ALERTER_NOTIFICATION   = "On ne vole pas chevaux ni chariots ! La loi arrive !",

        ALERT_CARRYING_HOSTAGE_NAME                   = "OTAGE À CHEVAL !",
        ALERT_CARRYING_HOSTAGE_MESSAGE                = "On a vu quelqu'un chevaucher avec un prisonnier ligoté sur son cheval ! Interceptez-le, shérif !",
        ALERT_CARRYING_HOSTAGE_ALERTER_NOTIFICATION   = "Des gens vous ont vu chevaucher avec quelqu'un de ligoté ! La loi a été prévenue !",

        -- Système de Réponse des Forces de l'Ordre
        LAW_SYSTEM_TITLE              = "Système Légal",
        LAW_RESET_STATE               = "L'état de réponse légale a été réinitialisé",
        LAW_REMOTE_LOCATION           = "Les forces de l'ordre ne peuvent pas atteindre cet endroit isolé",
        LAW_ESCAPED                   = "Vous avez échappé aux forces de l'ordre",
        LAW_DEFEATED                  = "Vous avez vaincu tous les représentants de la loi",
        LAW_OFFICER_NAME              = "Agent de la Loi",
        
        -- Notifications de la Loi
        LAW_RESPONSE_SAINT_DENIS      = "La Police de Saint Denis a été alertée",
        LAW_RESPONSE_RHODES           = "Le Bureau du Shérif de Rhodes a été notifié",
        LAW_RESPONSE_VALENTINE        = "Le Shérif de Valentine est en route",
        LAW_RESPONSE_STRAWBERRY       = "Le Shérif de Strawberry a été notifié",
        LAW_RESPONSE_BLACKWATER       = "Le Bureau du Shérif de Blackwater répond à l'appel",
        LAW_RESPONSE_TUMBLEWEED       = "Le Bureau du Shérif de Tumbleweed est en mouvement",
        LAW_RESPONSE_ARMADILLO        = "Le Shérif d'Armadillo a été notifié",
        LAW_RESPONSE_ANNESBURG        = "Le Shérif d'Annesburg répond à l'appel",
        LAW_RESPONSE_WILDERNESS       = "Des Chasseurs de Primes ont été envoyés",

        -- Notifications de Réponse de la Loi
        LAW_DEFEATED_TITLE   = "LOI VAINCUE",
        LAW_DEFEATED_MESSAGE = "Vous avez vaincu tous les hommes de loi !",
        LAW_ESCAPED_TITLE    = "LOI ÉCHAPPÉE",
        LAW_ESCAPED_MESSAGE  = "Vous avez échappé aux hommes de loi.",

        -- Arrestation par la loi PNJ (reddition)
        LAW_ARRESTED_TITLE   = "ARRÊTÉ",
        LAW_ARRESTED_MESSAGE = "Vous avez été arrêté par la loi",
        LAW_JAILED           = "Vous avez été arrêté et emprisonné pour %s minutes",
        -- The built-in Sisika jail (1.4.0)
        JAIL_SENTENCED       = "Vous avez été envoyé au pénitencier de Sisika pour %s minutes",
        JAIL_RESUMED         = "Votre peine à Sisika continue : il reste %s",
        JAIL_RELEASED        = "Vous êtes sorti du pénitencier de Sisika. Vous êtes libre.",
        JAIL_ESCAPED         = "Vous vous êtes évadé du pénitencier de Sisika !",
        JAIL_CANT_LEAVE      = "Vous ne pouvez pas quitter l'enceinte de la prison !",
        JAIL_OUT_OF_BOUNDS   = "Vous avez franchi les murs de la prison. Continuez et vous serez un évadé !",
        JAIL_TIMER           = "Pénitencier de Sisika : il reste %s",
        JAIL_ESCAPE_ALERT_NAME = "ÉVASION !",
        JAIL_ESCAPE_ALERT_MESSAGE = "Un détenu s'est évadé du pénitencier de Sisika ! Retrouvez-le, shérif !",

        -- Nouveaux crimes en 1.4.0
        ALERT_LOOTING_NAME                  = "QUELQU'UN DÉTROUSSE UN CADAVRE !",
        ALERT_LOOTING_MESSAGE               = "On a vu quelqu'un fouiller les poches d'un mort ! Allez voir, shérif !",
        ALERT_LOOTING_ALERTER_NOTIFICATION  = "Des gens vous ont vu détrousser les morts ! La loi a été prévenue !",
        ALERT_POACHING_NAME                 = "BRACONNAGE !",
        ALERT_POACHING_MESSAGE              = "Quelqu'un abat des animaux protégés ! Retrouvez-le, shérif !",
        ALERT_POACHING_ALERTER_NOTIFICATION = "Quelqu'un vous a vu braconner ! La loi arrive !",
        -- Détails des alertes et alertes des autres scripts (1.4.0)
        ALERT_WHERE_IN                      = "À %s",
        ALERT_WHERE_NEAR                    = "Près de %s",
        ALERT_WHERE_WILDS                   = "En pleine nature",
        ALERT_REPORTED_AT                   = "signalé à %s",
        ALERT_DEFAULT_NAME                  = "ALERTE DE LA LOI",
        ALERT_DEFAULT_MESSAGE               = "Un crime a été signalé. Rendez-vous sur place.",
        ALERT_DEFAULT_ALERTER               = "Quelqu'un vous a vu ! La loi a été appelée !",
        NOBODY_NOTICED                      = "Il ne s'est rien passé. Personne n'a rien remarqué.",
        ALERT_ARRIVED                       = "Vous êtes arrivé sur le lieu de l'alerte.",
        WAYPOINT_CLEARED                    = "Point de passage effacé.",
        MARKER_CLEARED                      = "Marqueur effacé.",
        ALERTS_CLEARED                      = "%s alerte(s) et points de passage effacés.",
        ALERTS_ALL_CLEARED                  = "Toutes les alertes et points de passage effacés.",
        TIME_MIN_SEC                        = "%d min %d s",
        TIME_SEC                            = "%d s",
        -- Loi PNJ : sommations et reddition (1.4.0)
        LAW_WARNED                          = "Les hommes de loi vous somment de vous rendre ! Levez les mains ou fuyez.",
        LAW_SURRENDER_GROUP                 = "Hommes de loi",
        LAW_SURRENDER_PROMPT                = "Se rendre",
        LAW_SURRENDER_TITLE                 = "REDDITION",
        LAW_SURRENDERING                    = "Vous vous rendez à la loi",
        LAW_SURRENDER_ACCEPTED              = "Les hommes de loi ont accepté votre reddition",
        LAW_SURRENDER_BROKEN_SHOT           = "Vous avez rompu votre reddition en tirant !",
        LAW_SURRENDER_BROKEN_MOVED          = "Vous avez rompu votre reddition en bougeant !",
        ALERT_NOBODY_TO_ANSWER              = "La nouvelle s'est répandue, mais aucun homme de loi n'est en service pour répondre.",
        LAW_SURRENDER_BROKEN_DREW           = "Vous avez rompu votre reddition en sortant une arme !",
        LAW_SURRENDER_ARREST_FAILED         = "Les hommes de loi n'ont pas pu vous atteindre.",
        LAW_RELEASED_TITLE                  = "LIBÉRÉ",
        LAW_RELEASED_MESSAGE                = "Les adjoints vous laissent partir avec un avertissement. Qu'ils ne vous reprennent pas.",
        WITNESS_BLIP                        = "Témoin"
    }
}

-- ############################################################
-- ##  TRANSLATION HELPER FUNCTION ############################
-- ##  DO NOT EDIT BELOW THIS LINE ############################
-- ############################################################
function T(key, ...)
    local lang      = Config.Language or "en"
    local langTable = Translations[lang] or Translations["en"] -- fallback
    local text      = langTable[key] or key                   -- fallback to key

    if select('#', ...) > 0 then
        local success, result = pcall(string.format, text, ...)
        if success then return result end
        print(string.format("[Translations] Format error for key '%s' (%s): %s", key, lang, tostring(result)))
    end
    return text
end