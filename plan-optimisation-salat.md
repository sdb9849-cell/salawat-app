# Plan d’optimisation de Salawat

## Synthèse exécutive

Salawat possède déjà une base produit crédible : application Expo/React Native/Web, accueil avec prochaine prière et compte à rebours, recherche de mosquées, favoris persistants, fiches de mosquées, horaires d’adhan et d’iqama, calendrier mensuel, Quran avec reprise de lecture, Qibla et espace de gestion.

Cependant, l’écart entre le prototype fonctionnel actuel et une application de salât fiable en production est important sur quatre sujets critiques :

1. **Les horaires affichés sont encore principalement des données de démonstration.**
2. **Les notifications ne constituent pas encore un système fiable planifié localement.**
3. **Le calcul réel selon localisation, méthode, madhab, fuseau et hautes latitudes n’est pas encore intégré.**
4. **La Qibla n’est pas encore validée avec les capteurs natifs sur des appareils réels.**

La priorité ne doit donc pas être d’ajouter davantage d’écrans. Elle doit être de transformer Salawat en une source de confiance : horaires explicables, fonctionnement hors ligne, notifications diagnostiquables, comportement robuste lors des changements de lieu et UX calme centrée sur la prochaine prière.

> **Principe directeur :** mieux vaut cinq fonctions fiables et transparentes qu’une longue liste de fonctions impressionnantes mais incertaines.

---

## 1. État actuel et écart avec une version production

| Domaine | État constaté | Risque produit | Niveau de priorité |
|---|---|---|---|
| Accueil | Prochaine prière, compte à rebours, cinq horaires | Risque de perte de confiance si les heures sont fictives ou non expliquées | Critique |
| Calcul | Horaires de démonstration ; pas encore de moteur réel local | Horaires faux selon ville, méthode, saison ou madhab | Critique |
| Mosquées | Recherche, filtres, favoris, fiches, iqama et calendrier de démonstration | Confusion entre données publiques, données de mosquée et données simulées | Élevée |
| Notifications | Expo Notifications installé ; réglages surtout visuels | Les interrupteurs peuvent promettre une alerte qui n’est jamais planifiée | Critique |
| Qibla | Interface et calibration visuelle | Angle non fiable sans capteur magnétique/GPS réel | Élevée |
| Quran | 114 sourates, arabe, traduction française, favoris, reprise | Bon socle ; vérifier licences, attribution, cache et mode hors ligne | Moyenne |
| Gestion mosquée | Horaires, annonces, membres, statistiques | Besoin d’authentification, rôles, audit et validation des changements | Élevée |
| Web | Routes et interface disponibles | Le Web ne doit pas promettre la même précision capteur/notification que le natif | Moyenne |

### Règle de communication à appliquer immédiatement

Tant que les sources réelles ne sont pas branchées, afficher explicitement un état comme **« Mode démonstration »** ou **« Horaires d’exemple »**. Ne jamais présenter des heures fictives comme des horaires religieux réels. Cette transparence est plus importante que l’apparence de complétude.

---

## 2. Précision technique et fiabilité

### 2.1 Architecture recommandée pour les horaires

Le moteur doit séparer quatre notions qui sont souvent mélangées :

1. **Source de vérité** : horaires publiés par une mosquée, horaires calculés localement, ou fournisseur distant.
2. **Convention de calcul** : MWL, ISNA, Umm al-Qura, Égypte, Karachi, Tehran, France, Russie, Singapour, etc.
3. **Paramètres juridiques** : madhab Shafi’i/standard ou Hanafi pour Asr.
4. **Corrections locales** : ajustements en minutes, arrondi, hautes latitudes et cas polaires.

Utiliser une bibliothèque éprouvée de type `adhan-js` plutôt que de réimplémenter les formules. Elle documente notamment `CalculationMethod`, `Madhab`, `HighLatitudeRule`, les ajustements et l’arrondi [R1][R2].

#### Configuration minimale persistée

```text
latitude
longitude
city/country
IANA timezone
calculation method
madhab
high latitude rule
manual adjustments per prayer
rounding policy
source preference: mosque > calculated
last successful calculation
```

Le moteur doit retourner les horaires ainsi que les métadonnées qui expliquent le résultat : méthode utilisée, source, fuseau, coordonnées et éventuels ajustements. Un écran « Pourquoi cette heure ? » réduira fortement les incompréhensions.

### 2.2 Méthodes à supporter

- **Muslim World League (MWL)** : bon défaut international lorsque la région n’a pas de convention locale clairement établie.
- **ISNA** : fréquemment utilisé en Amérique du Nord.
- **Umm al-Qura / Makkah** : adapté au contexte saoudien, avec ses conventions propres ; ne pas l’utiliser automatiquement partout.
- **Égypte / Egyptian General Authority of Survey** : convention fréquente dans une partie du monde arabe et d’Afrique.
- **Hanafi / Shafi’i** : concerne principalement le calcul de l’Asr, pas une « méthode de pays ». L’interface doit donc proposer le madhab séparément de la méthode astronomique.
- **Méthode locale de mosquée** : prioritaire lorsque la mosquée publie ses horaires et que l’utilisateur l’a choisie comme référence.

Le choix par défaut doit être régional et explicable, avec possibilité de sélection manuelle. Il faut éviter de changer silencieusement de méthode lorsqu’un utilisateur voyage.

### 2.3 Priorité entre mosquée et calcul automatique

Recommandation : appliquer cette hiérarchie :

1. calendrier/horaires explicitement fournis par la mosquée favorite ;
2. horaires d’iqama fournis par cette mosquée ;
3. horaires calculés localement pour la position actuelle ;
4. dernier cache connu, clairement marqué comme tel ;
5. mode dégradé avec message d’absence de données.

L’adhan et l’iqama doivent rester visuellement distincts. L’adhan indique le début de la fenêtre ; l’iqama est l’horaire communautaire et peut être fixe, saisonnier ou spécifique à la mosquée.

### 2.4 GPS, cache et fonctionnement hors ligne

Le GPS ne doit pas être interrogé en continu. Recommandation :

- demander la localisation au moment utile, après avoir expliqué pourquoi ;
- conserver la dernière position valide ;
- recalculer si le déplacement dépasse un seuil raisonnable, par exemple 10–25 km, ou lors d’une action explicite « actualiser ma position » ;
- stocker un cache indexé par `date + latitude arrondie + longitude arrondie + fuseau + méthode + madhab + réglages` ;
- pré-calculer plusieurs jours, idéalement 30 jours, après chaque recalcul réussi ;
- continuer à afficher les horaires en mode avion ;
- signaler l’âge et la source du cache.

Les changements suivants doivent déclencher une invalidation et une replanification : changement de date, fuseau horaire, heure d’été/hiver, localisation, méthode, madhab, correction manuelle et calendrier fourni par la mosquée.

### 2.5 Fuseau horaire et changement d’heure

Ne pas stocker seulement une heure locale en texte. Stocker une date locale, un fuseau IANA et un identifiant de calcul. Les notifications doivent être replanifiées lorsque le fuseau change. Un voyage Paris–New York ne doit pas conserver des alarmes de Paris.

Prévoir des tests autour :

- passage à l’heure d’été et à l’heure d’hiver ;
- changement de jour lors d’un vol ;
- minuit local ;
- horaires proches de 00:00 ou 24:00 ;
- appareil avec heure automatique désactivée ;
- retour en ligne après plusieurs jours.

### 2.6 Fiabilité des notifications en arrière-plan

Expo Notifications peut planifier des déclencheurs locaux de type date/calendrier et gérer les canaux Android, mais cela ne signifie pas qu’une alerte sera toujours délivrée à la seconde près [R3].

#### Android

- Android 12+ introduit les contraintes relatives aux alarmes exactes ; Android 14+ peut refuser par défaut cette capacité à certaines nouvelles installations [R3].
- Doze et les optimisations constructeur peuvent retarder les alarmes inexactes. Android recommande de réserver les alarmes exactes aux cas réellement sensibles au temps [R4].
- Android 13+ demande une autorisation de notification ; la demande intervient après création d’un canal de notification [R3].
- Créer des canaux séparés : adhan, rappel avant prière, rappel après prière, annonces de mosquée.
- Ajouter un diagnostic visible : notifications autorisées, canal actif, batterie optimisée, alarme exacte disponible ou non, dernière planification réussie.

#### iOS

- Les notifications locales peuvent être planifiées avec des déclencheurs calendaires [R5].
- Le consentement doit être demandé dans le contexte de la fonctionnalité et le statut doit être vérifié avant chaque planification [R6].
- Le système et les réglages utilisateur peuvent modifier le son, l’affichage, le résumé programmé ou les modes de concentration ; Salawat ne doit donc pas promettre une garantie absolue.
- Afficher un état « notifications autorisées mais potentiellement silencieuses » si le son, le résumé programmé ou le mode concentration l’impose.

#### Stratégie de fiabilité

1. Calculer localement les prières à venir.
2. Annuler les identifiants précédents de Salawat.
3. Planifier le prochain jour ou les 7 prochains jours selon les limites natives observées.
4. Enregistrer les identifiants, la date de génération et la configuration.
5. Replanifier au lancement, au retour au premier plan, après changement de fuseau/localisation/réglage et après une mise à jour de l’app.
6. Afficher un journal technique minimal en cas d’échec.
7. Proposer un mode de repli : notification inexacte, rappel d’ouverture de l’app, ou désactivation explicite plutôt qu’une fausse promesse.

Les notifications audio d’adhan nécessitent en plus des fichiers sonores correctement déclarés et testés en arrière-plan. Cette fonctionnalité doit être validée sur un panel d’appareils, pas seulement dans Expo Go.

### 2.7 Qibla, capteurs et batterie

La Qibla doit combiner :

- coordonnées GPS de l’utilisateur ;
- coordonnées fixes de la Kaaba ;
- calcul géodésique du relèvement initial ;
- capteur magnétique/rotation natif ;
- compensation de l’inclinaison lorsque la plateforme le permet ;
- indication de précision et de disponibilité du capteur.

UX de calibration recommandée :

1. demander de s’éloigner des objets métalliques ;
2. afficher une animation courte en huit ;
3. indiquer « précision faible/moyenne/bonne » ;
4. proposer une vérification avec une carte ou un repère visuel ;
5. ne jamais masquer l’incertitude par une aiguille parfaitement stable.

Pour la batterie, ne pas activer de localisation continue. Lire les capteurs uniquement sur l’écran Qibla, à fréquence limitée, arrêter les listeners à la sortie et utiliser un lissage anti-tremblement. Prévoir un message spécifique sur les appareils sans magnétomètre.

---

## 3. UX et ergonomie

### 3.1 Écran principal

L’écran d’accueil doit répondre en moins de trois secondes à trois questions : **quelle prière est en cours, quelle est la prochaine, et quand arrive-t-elle ?**

Structure recommandée :

- en-tête discret : ville, source et état de localisation ;
- carte principale : prière en cours ou prochaine prière, heure, compte à rebours ;
- distinction claire entre « début de la prière/adhan » et « iqama » ;
- liste compacte des cinq prières avec état passé/en cours/à venir ;
- accès direct à la mosquée favorite ;
- accès aux réglages de méthode et de notification sans enfouissement ;
- état de confiance : calcul local, mosquée, cache ou démonstration.

Le compte à rebours doit gérer le passage de minuit, le changement de jour et la différence entre une prière qui vient de commencer et la suivante. Éviter les animations permanentes lourdes qui consomment la batterie.

### 3.2 Onboarding en quelques étapes

Proposer quatre étapes maximum, avec possibilité de passer et de revenir :

1. **Lieu** : localisation automatique ou ville manuelle.
2. **Référence horaire** : méthode recommandée, méthode manuelle, madhab.
3. **Notifications** : choix des prières, rappels avant/après, son, adhan ; demander la permission au moment où l’utilisateur active la fonction.
4. **Mosquée favorite** : recherche facultative, avec option de ne pas choisir de mosquée.

À la fin, afficher un récapitulatif : « Vos horaires sont calculés avec… ». Le premier écran ne doit pas demander simultanément GPS, notifications, compte, mosquée et préférences détaillées.

### 3.3 Accessibilité et design apaisant

- contraste WCAG AA au minimum pour le texte courant ;
- ne pas communiquer un état uniquement par la couleur ;
- tailles de texte dynamiques ;
- zones tactiles suffisamment grandes ;
- labels pour lecteurs d’écran ;
- mode sombre réel, pas uniquement une inversion de couleurs ;
- respect du réglage système clair/sombre ;
- réduction des animations ;
- dates et heures localisées ;
- français simple, sans ton culpabilisant.

L’identité de Salawat doit rester originale : palette, iconographie, logo, composants, libellés et illustrations propres à Salawat. L’inspiration fonctionnelle est légitime, mais il ne faut pas reproduire l’identité visuelle, le logo ou le code de Mawaqit.

### 3.4 Widgets et raccourcis

À moyen terme :

- widget « prochaine prière » ;
- widget « cinq prières » ;
- raccourci vers Qibla ;
- raccourci vers la mosquée favorite ;
- raccourci vers le Quran repris.

Ces éléments sont utiles seulement après stabilisation du moteur et des données. Un widget affichant une heure incorrecte est pire que l’absence de widget.

---

## 4. Rétention et valeur ajoutée sans surcharge

### 4.1 Boucle de valeur principale

La boucle quotidienne doit rester simple :

1. ouvrir ou voir le widget ;
2. connaître la prochaine prière ;
3. recevoir un rappel fiable ;
4. accéder à la mosquée ou au Quran ;
5. revenir le lendemain sans friction.

### 4.2 Suivi discret des prières

Proposer un suivi privé et optionnel, sans gamification agressive :

- « accomplie », « à rattraper », « non renseignée » ;
- stockage local par défaut ;
- aucune publication sociale automatique ;
- pas de classement public ;
- statistiques hebdomadaires sobres ;
- possibilité d’effacer toutes les données.

Le suivi ne doit pas culpabiliser l’utilisateur ni prétendre mesurer sa pratique religieuse.

### 4.3 Rappels avant et après l’adhan

Rendre les rappels composables :

- avant : 5, 10, 15 ou 30 minutes ;
- à l’heure : notification ou adhan ;
- après : rappel facultatif, par exemple 10 ou 20 minutes ;
- silencieux, vibration, son personnalisé ;
- plages silencieuses ;
- désactivation par prière.

Limiter le nombre de notifications par défaut. Le réglage recommandé doit être calme ; l’utilisateur avancé peut ensuite personnaliser.

### 4.4 Voyage

Créer un mode voyage léger :

- détection du changement de fuseau ;
- confirmation du nouveau lieu avant de recalculer ;
- affichage de la distance depuis la mosquée favorite ;
- possibilité de conserver temporairement une ville manuelle ;
- recalcul Qibla ;
- téléchargement du calendrier de la destination lorsque disponible.

Éviter d’inférer automatiquement des règles religieuses personnelles. Salawat peut fournir les paramètres et rappeler que certaines décisions de voyage relèvent de l’utilisateur et de son école/autorité de référence.

### 4.5 Quran et mosquées

Le Quran est un différenciateur utile s’il reste rapide et hors ligne : cache progressif, reprise fiable, recherche, taille du texte, mode nuit et attribution claire des sources. Les conditions d’utilisation et d’attribution des textes doivent rester documentées.

Pour les mosquées, la valeur vient de la fraîcheur et de la confiance : date de dernière mise à jour, rôle du gestionnaire, validation des horaires, historique des modifications et distinction entre information officielle et contribution communautaire.

---

## 5. Architecture produit et qualité

### 5.1 Modules à isoler

- `prayer-calculation` : calcul pur, testable sans UI ;
- `location-context` : position, permission, ville et fuseau ;
- `mosque-schedule` : calendrier officiel, iqama et source ;
- `notification-planner` : permissions, canaux, planification, diagnostic ;
- `qibla-sensors` : capteurs, calibration, précision ;
- `offline-cache` : version, invalidation et migrations ;
- `user-preferences` : méthode, madhab, réglages et confidentialité.

### 5.2 Tests indispensables

- tests de référence pour plusieurs villes et dates ;
- comparaison avec `adhan-js` et une source de référence ;
- cas hautes latitudes ;
- passage à l’heure d’été/hiver ;
- voyage et changement de fuseau ;
- absence de GPS et mode avion ;
- refus puis réactivation des permissions ;
- planification, annulation et replanification des notifications ;
- appareils Android de plusieurs constructeurs ;
- iPhone réel avec modes concentration et résumé programmé ;
- capteur indisponible ou faible précision ;
- accessibilité avec taille de texte élevée.

### 5.3 Observabilité sans atteinte à la vie privée

Journaliser localement et anonymement les états techniques : dernière planification, nombre de notifications, erreur native, source des horaires et âge du cache. Ne pas collecter la localisation précise ou les habitudes de prière sans consentement explicite et finalité claire.

---

## 6. Roadmap priorisée

### Quick wins — cette semaine

| Action | Impact | Effort | Critère de sortie |
|---|---:|---:|---|
| Marquer clairement toutes les données de démonstration | Très élevé | Faible | Aucun horaire fictif n’est présenté comme réel |
| Ajouter une carte « source/méthode/fuseau » sur l’accueil | Élevé | Faible | L’utilisateur comprend pourquoi une heure est affichée |
| Séparer visuellement adhan et iqama | Élevé | Faible | Les deux notions ne sont plus confondues |
| Concevoir le modèle de préférences complet | Élevé | Faible | Méthode, madhab, corrections et source sont persistés |
| Ajouter des états vides et erreurs explicites | Élevé | Faible | GPS refusé, cache absent et mosquée indisponible sont gérés |
| Ajouter le mode sombre et les labels d’accessibilité de base | Moyen/élevé | Faible | Parcours principal lisible et navigable |
| Ajouter une page diagnostic notifications | Élevé | Moyen | Permission, canal, dernière planification et limitations visibles |

### Moyen terme — 2 à 4 semaines

| Chantier | Impact | Effort | Dépendances |
|---|---:|---:|---|
| Intégrer le moteur local basé sur `adhan-js` | Critique | Moyen | Modèle de préférences, tests de référence |
| Ajouter le cache hors ligne multi-jours | Critique | Moyen | Moteur local, fuseau stable |
| Planifier réellement les notifications locales | Critique | Élevé | Moteur, permissions natives, build de développement |
| Replanifier après changement de lieu/fuseau/réglage | Critique | Moyen | Service de planification |
| Brancher une vraie source de mosquées et calendriers | Critique | Élevé | API, qualité des données, modération |
| Valider Qibla native sur appareils Android/iOS | Élevé | Élevé | Capteurs natifs, tests physiques |
| Ajouter onboarding court et écran de choix de référence | Élevé | Moyen | Moteur de calcul |
| Tester sur appareils réels et documenter les limites | Critique | Moyen | Matériel Android/iOS |

### Long terme — 1 à 3 mois

| Chantier | Impact | Effort | Critère de succès |
|---|---:|---:|---|
| Calendrier officiel par mosquée avec gestion des versions | Très élevé | Élevé | Horaires publiés et historisés |
| Gestion avancée des rôles et validation des annonces | Élevé | Élevé | Aucun changement sensible sans autorisation |
| Widgets et raccourcis natifs | Élevé | Élevé | Widget exact après changement de date/fuseau |
| Mode voyage et téléchargement de destination | Élevé | Moyen | Fonctionnement hors ligne pendant un déplacement |
| Suivi privé des prières | Moyen/élevé | Moyen | Adoption sans pression ni partage imposé |
| Quran hors ligne enrichi | Moyen | Moyen | Lecture instantanée sans réseau |
| Monitoring qualité et rapports anonymisés | Élevé | Élevé | Baisse mesurable des erreurs de planification |
| Support multi-langue élargi | Moyen | Moyen | Traductions relues et heures localisées |

### Ordre de livraison recommandé

**D’abord la vérité des données, ensuite les alertes, puis la croissance.** Ne pas commencer par les widgets, les statistiques ou la gamification tant que les horaires et notifications ne sont pas fiables.

---

## 7. Critères de succès

### Fiabilité

- 100 % des écrans indiquent la source des horaires.
- Les changements de fuseau et de méthode déclenchent une replanification vérifiable.
- Les horaires restent disponibles sans réseau après une première synchronisation.
- Aucun crash bloquant lorsque GPS, capteur ou notification est refusé.

### Notifications

- Taux de planification réussie mesuré sur Android et iOS réels.
- Diagnostic compréhensible par un utilisateur non technique.
- Tests documentés sur Doze, batterie optimisée, mode concentration et résumé programmé.
- Aucun réglage activé visuellement sans planification effective.

### UX

- L’utilisateur identifie la prochaine prière en moins de trois secondes.
- Onboarding terminé en moins de deux minutes sans compte obligatoire.
- Méthode, madhab et source sont modifiables en moins de deux actions depuis l’accueil.
- Parcours principal utilisable avec grande taille de texte et mode sombre.

### Produit

- Rétention à 7 jours mesurée séparément pour horaires, mosquées, Quran et Qibla.
- Taux d’activation des notifications suivi sans collecter le contenu de la pratique.
- Retours qualitatifs sur la confiance, la clarté et la fatigue liée aux notifications.

---

## Conclusion

Salawat a déjà les bons piliers fonctionnels pour devenir une application mondiale : horaires, mosquées, iqama, calendrier, Qibla, Quran et gestion. La prochaine étape n’est pas de reproduire davantage de fonctionnalités visibles d’une application existante ; c’est d’établir une promesse propre à Salawat : **des horaires expliqués, disponibles hors ligne, des alertes honnêtes sur leurs limites et une interface paisible qui accompagne sans culpabiliser**.

La version production devrait être déclarée prête seulement lorsque trois preuves existent :

1. les horaires sont calculés ou publiés à partir d’une source réelle et traçable ;
2. les notifications ont été testées sur appareils réels avec diagnostic des restrictions natives ;
3. la Qibla et le mode voyage ont été validés sur plusieurs lieux et orientations.

---

## Références

- **[R1]** PrayTimes, méthodes et conventions de calcul : https://praytimes.org/docs/methods
- **[R2]** Adhan JS, méthodes, madhab, hautes latitudes, ajustements et arrondi : https://github.com/batoulapps/adhan-js/blob/develop/METHODS.md
- **[R3]** Expo Notifications, permissions, canaux Android, déclencheurs et alarmes exactes : https://docs.expo.dev/versions/latest/sdk/notifications/
- **[R4]** Android Developers, alarmes et comportement exact/inexact sous Android/Doze : https://developer.android.com/develop/background-work/services/alarms
- **[R5]** Apple Developer, `UNCalendarNotificationTrigger` : https://developer.apple.com/documentation/usernotifications/uncalendarnotificationtrigger
- **[R6]** Apple Developer, demande et vérification des autorisations de notifications : https://developer.apple.com/documentation/usernotifications/asking-permission-to-use-notifications
- **[R7]** Mawaqit, fonctionnalités publiques de référence : https://www.mawaqit.net/fr
- **[R8]** Mawaqit, pourquoi les horaires diffèrent selon les mosquées : https://help.mawaqit.net/en/articles/9047426-why-are-prayer-times-different-from-one-mosque-to-another
- **[R9]** Mawaqit, téléchargement du calendrier de prière d’une mosquée : https://help.mawaqit.net/en/articles/8999582-how-to-download-the-prayer-times-of-my-mosque

---

## Questions ciblées pour la prochaine itération

1. Quelle sera la source prioritaire des horaires réels : une API externe, les calendriers gérés par chaque mosquée, ou le calcul local avec une source de secours ?
2. Souhaitez-vous que les horaires de la mosquée favorite priment toujours sur le calcul automatique, y compris lorsque l’utilisateur se déplace ?
3. Les notifications doivent-elles inclure un adhan audio complet, ou commencer par des alertes sonores courtes et plus robustes ?
4. Quelles régions et latitudes faut-il valider en premier : France/Europe, Amérique du Nord, pays du Golfe, Afrique, ou couverture mondiale dès la première version ?
5. Disposez-vous d’appareils Android et iOS réels pour tester Doze, les permissions, le changement de fuseau et la Qibla, ou faut-il d’abord préparer une matrice de test ?
