# Salawat — TODO

## [x] 6. Ajouter le Quran
- Afficher les 114 sourates avec recherche et navigation.
- Charger le texte arabe et une traduction française depuis des sources publiques attribuées.
- Permettre l’ouverture d’une sourate, la lecture verset par verset et les favoris de versets.
- Afficher la source et la version de la traduction dans l’écran de lecture.
- Ajouter la récitation audio d’une sourate avec lecture, pause, progression et attribution de la source audio.
- Charger le catalogue des éditions audio Al Quran Cloud et conserver le lecteur préféré localement.
- Mettre en cache localement la liste et le texte des sourates ouvertes pour permettre leur lecture hors connexion.
- Permettre de télécharger la récitation de la sourate et du lecteur choisis, puis de la supprimer ou de l’écouter hors connexion.

## [~] 9. Ajouter l’adhan sonore
- Permettre d’activer ou désactiver le son séparément pour Fajr, Dhuhr, Asr, Maghrib et Isha.
- Ajouter une prévisualisation audio depuis une source Wikimedia Commons annoncée CC0.
- Relier la planification native aux horaires de prière réels, au fuseau, à la localisation et aux permissions avant de présenter les alertes comme fiables.

## [~] 10. Comptes et financement responsable
- Permettre à chaque personne de créer un compte ou de se connecter via le flux OAuth existant.
- Afficher l’état du compte et proposer une déconnexion sûre depuis Profil.
- Préparer un emplacement d’annonce discret et clairement identifié, sans publicité trompeuse dans le contenu religieux.
- Activer une régie réelle (par exemple AdMob) uniquement avec un identifiant éditeur, le consentement requis et un build natif compatible ; ne pas casser l’utilisation Expo Go.

## [x] 7. Ajouter les horaires d’iqama et le calendrier des mosquées
- Ouvrir une fiche détaillée depuis chaque mosquée.
- Afficher séparément l’adhan et l’iqama pour les cinq prières.
- Afficher le calendrier mensuel de la mosquée sélectionnée.
- Permettre de partager le calendrier avec les horaires publiés.

## [x] 1. Livrer un accueil de prière mobile et web
- Afficher le nom Salawat, la ville active, la date et un état météo/localisation visuellement cohérent.
- Afficher une carte principale indiquant la prochaine prière, son heure et le temps restant.
- Afficher les cinq prières de la journée avec leurs heures et distinguer la prière active.
- Fournir des raccourcis vers Mosquées, Qibla et Gestion.

## [x] 2. Livrer l’exploration mondiale des mosquées
- Permettre de rechercher une mosquée ou une ville.
- Afficher des résultats avec distance, statut, horaires et services essentiels.
- Afficher une fiche de mosquée avec adresse, téléphone, responsable, validation et prochain horaire.
- Préparer les états de chargement, vide et données de démonstration pour une couverture mondiale progressive.

## [x] 8. Rechercher les mosquées à proximité
- Demander une localisation ponctuelle et afficher les mosquées OSM dans un rayon borné.
- Conserver un cache local des derniers résultats et afficher son âge hors ligne.
- Permettre une recherche manuelle par ville ou code postal lorsque le GPS est refusé.
- Trier par distance et conserver les filtres existants, avec filtres Jumu’ah, femmes et ouvertes.
- Proposer l’ouverture d’un itinéraire dans une application cartographique externe.
- Afficher une attribution OpenStreetMap et signaler les résultats issus de données communautaires.

## [x] 3. Livrer les réglages personnels et la Qibla
- Présenter une vue Qibla avec angle de référence, calibration et accès aux réglages.
- Permettre de modifier les préférences de calcul, les notifications, le format horaire et la langue au niveau de l’interface.
- Conserver une navigation utilisable en portrait sur mobile et en largeur sur Web.

## [~] 4. Livrer l’espace de gestion
- Afficher les indicateurs de gestion d’une mosquée : validation, membres, annonces et complétude des horaires.
- Fournir des accès d’édition pour les horaires, les informations, les annonces et les membres.
- Restreindre les mutations durables aux procédures serveur protégées par un rôle responsable/admin.
- Afficher clairement qu’une donnée de démonstration n’est pas encore une donnée publiée.

## [x] 5. Préparer les données et l’identité
- Ajouter les tables de mosquées, horaires, membres et annonces en conservant la table users du starter.
- Ajouter les procédures tRPC de lecture publique et les points d’entrée protégés de gestion.
- Déclarer le manifeste de routes Web.
- Appliquer l’identité Quran à l’icône et aux surfaces natives sans coins arrondis intégrés.
- Vérifier le diagnostic TypeScript et le démarrage des deux processus Expo/API avant livraison.
