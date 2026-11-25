# Sources techniques pour l’optimisation de Salawat

## Calcul des horaires

PrayTimes documente les principales conventions : MWL, ISNA, Egyptian General Authority of Survey, Umm al-Qura, Karachi, Tehran, Jafari, France, Russie et Singapour, avec les angles Fajr/Isha et les régions d’usage : https://praytimes.org/docs/methods

Adhan JS documente les paramètres nécessaires : méthode de calcul, angles Fajr/Maghrib/Isha, intervalle Isha après Maghrib, madhab Shafi/Hanafi pour Asr, règle de hautes latitudes, ajustements manuels, arrondi et résolution des cercles polaires : https://github.com/batoulapps/adhan-js/blob/develop/METHODS.md

## Notifications et arrière-plan

Expo Notifications précise qu’Android 12+ exige une permission d’alarme exacte pour les déclenchements précis et qu’Android 14+ peut la refuser par défaut pour une nouvelle installation. Android 13+ demande aussi l’autorisation utilisateur après création d’un canal de notification. Expo expose les déclencheurs date/calendrier et les niveaux `bestEffort`/`alarmClock` : https://docs.expo.dev/versions/latest/sdk/notifications/

Android recommande les alarmes inexactes pour la plupart des tâches afin de respecter Doze et réserve les alarmes exactes aux actions réellement sensibles au temps. Les alarmes exactes consomment davantage de ressources : https://developer.android.com/develop/background-work/services/alarms

Apple documente les déclencheurs calendaires locaux pour programmer une notification à une date et heure précises : https://developer.apple.com/documentation/usernotifications/uncalendarnotificationtrigger

Apple recommande de demander l’autorisation dans le contexte de la fonctionnalité, puis de vérifier systématiquement le statut d’autorisation avant de planifier les notifications : https://developer.apple.com/documentation/usernotifications/asking-permission-to-use-notifications

## État constaté dans Salawat

La base actuelle est Expo SDK 58, React Native, TypeScript et React Native Web. L’accueil et les horaires principaux utilisent encore des données de démonstration. La Qibla est une interface visuelle préparée mais n’utilise pas encore un capteur natif. Les réglages de notifications sont des interrupteurs locaux sans planification réelle. Les mosquées favorites, les horaires d’iqama et le calendrier mensuel ont été ajoutés récemment, avec des données de démonstration structurées. Le Quran est disponible avec favoris persistants.
