# Salawat — plan produit et technique

## Périmètre confirmé

Salawat est une application mobile et web destinée aux musulmans du monde entier. La première version doit présenter une expérience claire et premium autour des horaires de prière, de la découverte des mosquées, de la Qibla, des notifications et de la gestion des mosquées. L’identité est originale et utilise une icône représentant un Coran.

Le produit sera construit dans le starter Expo/React Native/TypeScript déjà initialisé. Le même code cible Android, iOS et le Web responsive. La base de données gérée par Manus servira aux comptes, mosquées, horaires, paramètres de calcul et annonces. Manus OAuth reste le mécanisme de connexion par défaut ; les permissions distinguent les fidèles, responsables de mosquées et administrateurs.

## Découpage fonctionnel

- **Accueil** : ville active, date hijri/grégorienne de démonstration, grande carte de la prochaine prière, cinq horaires du jour, accès rapide aux mosquées, Qibla et réglages.
- **Mosquées** : recherche par ville ou pays, filtres (ouvert maintenant, distance, services), liste de mosquées et carte visuelle ; fiche avec adresse, horaires, services, contact et validation.
- **Qibla** : écran d’orientation préparé pour le compas natif, avec angle de référence pour la ville sélectionnée et état de calibration.
- **Profil / réglages** : notifications, méthode de calcul, école juridique, format 12/24 h, langue et localisation.
- **Quran** : catalogue des 114 sourates, recherche, lecture arabe verset par verset, traduction française, favoris de versets et affichage clair des sources.
- **Gestion** : tableau de bord responsable avec statistiques, édition des horaires, informations de mosquée, annonces, membres et état de validation. La première version expose le parcours et les états principaux ; les mutations durables passent ensuite par les procédures serveur protégées.

## Architecture

- `app/` contient les routes Expo Router pour les quatre onglets et les écrans de gestion.
- `components/` contient les cartes réutilisables (prière, mosquée, statistiques, section, bouton).
- `constants/` porte la palette, les libellés et les données de démonstration contrôlées.
- `drizzle/schema.ts` ajoute les entités `mosques`, `prayerSchedules`, `mosqueMembers` et `announcements` avec des clés stables, des champs de localisation et des statuts de validation.
- `server/` expose les lectures publiques et les procédures de gestion protégées ; l’absence de base en environnement local conserve un fallback de démonstration pour que l’interface reste navigable.
- `public/manus-routes.json` déclare les routes page-level connues à Expo Web.
- `assets/images/` contient l’identité native : icône Quran, splash et variantes Android/favicons dérivées du même langage visuel.

## Direction design choisie

### Alternatives considérées

- **Émeraude éditoriale et or discret** — probabilité 0,07.
- **Sable méditerranéen et bleu nuit** — probabilité 0,05.
- **Noir encre et turquoise lumineux** — probabilité 0,03.

### Choix : émeraude éditoriale et or discret

- **Mouvement** : tableau de bord calme inspiré d’un agenda éditorial et d’une signalétique de mosquée contemporaine, sans copier MAWAQIT.
- **Principes** : hiérarchie forte, respiration généreuse, cartes tactiles à faible densité, accents dorés réservés aux moments importants.
- **Philosophie couleur** : l’émeraude installe confiance et sérénité ; l’or signale la prochaine prière et les actions prioritaires ; les fonds ivoire évitent l’effet clinique.
- **Paradigme de mise en page** : une colonne mobile guidée par une carte de prochaine prière, puis des bandes horizontales et des raccourcis ; sur grand écran, une composition asymétrique avec rail latéral de navigation et contenu principal.
- **Signatures** : pastilles d’heure en chiffres larges, filet doré sous les titres, petits motifs géométriques en arrière-plan des cartes de prière.
- **Interaction** : chaque carte donne une action claire ; les filtres restent visibles et les états admin sont exprimés par badges plutôt que par tableaux denses.
- **Animation** : transitions courtes et douces, apparition par fondu et translation de quelques pixels, aucun mouvement décoratif pendant la lecture des horaires.
- **Typographie** : système sans-serif pour la lisibilité, titres semi-bold avec grands chiffres, petites capitales/lettres espacées uniquement pour les métadonnées.
- **Essence de marque** : « Les prières de votre journée, les mosquées de votre communauté, au même endroit. » Personnalité : sereine, précise, accueillante.
- **Voix** : directe, chaleureuse, respectueuse. Exemples : « Votre prochaine prière commence dans 42 min » et « Retrouvez une mosquée près de vous ».
- **Signe / icône** : Coran fermé stylisé, couverture émeraude et détail doré, sans texte ni coins arrondis intégrés.
- **Couleur signature** : émeraude `#0F6B58`.

## Contraintes et limites de la première livraison

Les horaires et mosquées affichés immédiatement sont des données de démonstration réalistes afin de montrer le produit avant le branchement d’un fournisseur mondial de calculs/localisation. La structure de données et les procédures serveur préparent la couverture mondiale ; les notifications et le compas réel nécessitent ensuite les permissions et vérifications natives sur un appareil. Aucun logo, code ou copie de MAWAQIT n’est repris.

Le contenu Quran est chargé à la demande depuis Al Quran Cloud pour le texte arabe et QuranEnc pour la traduction française Rachid Maach, version 1.0.3. L’écran affiche cette attribution et conserve le texte de traduction sans modification.
