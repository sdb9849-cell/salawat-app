# Spécification — Mosquées à proximité pour Salawat

## Décision d’architecture recommandée

Pour Salawat, la meilleure stratégie initiale est une architecture hybride :

1. **OpenStreetMap + Overpass** comme source principale de découverte, car les données sont ouvertes et les petites mosquées peuvent être ajoutées par la communauté.
2. **Une base indépendante Salawat** pour conserver les lieux validés, les corrections, les horaires d’iqama, les équipements et les contributions des utilisateurs.
3. **Un proxy backend Node/tRPC** entre l’application Expo et les fournisseurs externes.
4. **Un cache géographique partagé** afin de ne pas interroger un fournisseur à chaque ouverture de l’écran.
5. **Mapbox, Foursquare ou Google Places comme compléments optionnels**, activables par territoire ou fonctionnalité après validation des coûts et des licences.

Une recherche vide doit toujours être présentée comme une absence de données dans la zone interrogée, et non comme la preuve qu’il n’existe aucune mosquée.

---

## 1. Comparatif des sources et APIs

| Option | Intérêt principal | Qualité pour les petites salles | Coût et limites | Verdict pour Salawat |
|---|---|---|---|---|
| **OpenStreetMap + Overpass** | Données ouvertes, géométrie, tags communautaires, recherche autour d’un point | Variable, mais souvent la meilleure chance de trouver un lieu local si un contributeur l’a cartographié | Gratuit en données, mais les instances publiques n’offrent ni SLA ni quota commercial garanti. Les politiques recommandent de limiter fortement les requêtes et d’éviter d’en faire le backend d’une application à forte audience [1][2] | Source principale de découverte, avec cache, attribution et plan de montée en charge |
| **Google Places API (New)** | POI riches, recherche Nearby/Text, horaires et liens Google Maps | Bonne pour les lieux connus ; pas de type distinct garanti pour les salles de prière [3][4] | Paiement à l’usage, quotas par projet et restrictions fortes de cache/redistribution. Nearby Search et Text Search sont facturés selon le SKU et les champs demandés [5][6] | Complément ponctuel seulement, après validation juridique et budgétaire |
| **Mapbox Search Box** | Recherche interactive de POI, catégories, proximité et expérience cartographique cohérente | Variable ; aucune exhaustivité spécifique aux mosquées n’est garantie | Facturation selon produit/plan. Les résultats Search Box ne doivent pas être copiés dans une base durable sans vérifier les Product Terms ; le Geocoding v6 n’est plus la solution pour inventorier les POI [7][8] | Bon complément de recherche interactive si Mapbox est déjà utilisé pour la carte |
| **Foursquare Places** | Recherche de lieux, taxonomie importante, couverture internationale | Potentiellement utile, mais aucun taux de rappel spécifique aux mosquées ou musallas n’est garanti [9] | Pay-as-you-go, limite de débit documentée et règles de cache très restrictives selon le plan. Certains identifiants peuvent être conservés, mais pas tous les attributs [10][11] | Option de secours pour recherche ponctuelle, pas pour miroir permanent |
| **Crowdsourcing Salawat** | Données propres, corrections, horaires d’iqama et équipements réellement utiles | Potentiellement excellent localement après validation | Coût de modération et de vérification ; nécessite anti-abus, historique et rôles | À construire en parallèle pour réduire la dépendance aux fournisseurs |

### OpenStreetMap et Overpass

Les données OSM sont ouvertes sous ODbL, avec attribution et obligations spécifiques lorsqu’une base dérivée est distribuée [1]. Overpass est adapté à la lecture de données autour d’un point. Une requête peut rechercher les objets `nwr` — nœuds, chemins et relations — avec `around:rayon,latitude,longitude`.

La recherche doit combiner plusieurs formes de marquage :

- `amenity=place_of_worship` + `religion=muslim` pour les mosquées ;
- `place_of_worship=musalla` pour certaines salles de prière ;
- nom et variantes locales lorsque les tags religieux sont incomplets ;
- déduplication entre un nœud, une aire et une relation représentant le même lieu.

Les serveurs publics Overpass peuvent limiter ou refuser les requêtes lourdes. La politique documente des limitations dépendant de la durée, de la mémoire et de la charge, et déconseille les requêtes répétées, le quadrillage mondial et l’usage d’une instance publique comme backend d’une application à forte audience [2]. Il faut donc interroger côté serveur, limiter le rayon, regrouper les recherches et prévoir une instance gérée ou auto-hébergée si l’audience augmente.

Nominatim peut aider à convertir une ville ou un code postal en coordonnées, mais son instance publique impose notamment une limite absolue d’environ une requête par seconde, interdit l’autocomplétion et déconseille les collectes systématiques [12]. Il ne doit pas être utilisé comme moteur de recherche de mosquées à chaque frappe.

### Google Places

Places API (New) propose `Nearby Search` avec un cercle et `Text Search` pour des requêtes telles que « mosque », « masjid », « mosquée » ou « salle de prière ». Le type officiel `mosque` existe, mais Google ne garantit pas l’exhaustivité ni un type distinct pour les salles de prière [3][4].

Le coût dépend du SKU et du masque de champs. Les plafonds gratuits, les prix par 1 000 événements et les limites peuvent varier selon le niveau de données demandé, le volume, le pays et le compte de facturation [5][6]. L’application ne doit donc pas supposer un ancien crédit mensuel universel.

Point juridique important : Google limite le stockage, la redistribution et la création d’un annuaire autonome à partir de ses données. Les résultats affichés sur une carte doivent respecter les exigences Google Maps et d’attribution [13]. Google Places ne doit donc pas devenir la base permanente de Salawat sans validation des conditions applicables.

### Mapbox

Mapbox Search Box est adapté à la recherche interactive de POI par texte, catégorie et proximité. Le Geocoding API reste utile pour les adresses, mais n’est plus adapté à l’inventaire des points d’intérêt dans la version actuelle [7].

Search Box renvoie au maximum dix résultats et n’offre pas une pagination destinée à aspirer une zone. Les appels doivent être déclenchés après une action ou quelques caractères, avec debounce et session cohérente pour l’autocomplétion. La tarification dépend du produit, du plan et du type de requête [8].

Les résultats temporaires et permanents ont des conditions différentes. Ne pas écrire automatiquement les attributs Mapbox dans MySQL. La base Salawat doit stocker uniquement des données indépendantes, confirmées par l’utilisateur ou explicitement autorisées par les conditions Mapbox.

### Foursquare

Foursquare Place Search accepte une position, un rayon ou une zone rectangulaire, ainsi que la recherche par catégorie et texte. La documentation annonce une couverture internationale importante, mais ne fournit pas de taux de rappel spécifique aux mosquées ou salles de prière [9].

Le prix est un modèle pay-as-you-go. Les appels Pro disposent d’un tarif et de paliers propres au compte ; les endpoints Premium comme les photos et astuces coûtent davantage. La limite de débit documentée doit être respectée [10].

Les règles de cache sont particulièrement contraignantes : selon le plan, seuls certains identifiants sont conservables durablement, alors que les autres attributs sont limités à un cache local de courte durée ou interdits en cache serveur [11]. Cette option ne convient pas pour constituer un catalogue permanent de Salawat.

### Recommandation finale des sources

Pour la première version :

- **Découverte** : OSM/Overpass côté backend.
- **Persistance** : base Salawat pour les lieux validés et enrichis.
- **Géocodage manuel** : Nominatim avec cache et faible fréquence, ou un service commercial si les conditions de volume l’exigent.
- **Complément commercial** : un seul fournisseur à la fois, derrière une interface `MosqueProvider`, avec budget mensuel et coupe-circuit.
- **Carte** : utiliser un fournisseur de tuiles compatible avec la licence et l’attribution choisies ; ne pas utiliser les tuiles publiques OSM comme capacité commerciale ou hors ligne sans respecter leur politique [14].

---

## 2. Modèle de données

### Informations indispensables pour le fidèle

Une fiche doit afficher, dans cet ordre :

- nom et type : mosquée, musalla ou salle de prière ;
- distance à vol d’oiseau et, si disponible, estimation de marche ;
- adresse lisible ;
- bouton d’itinéraire ;
- statut « ouverte maintenant » avec heure de dernière mise à jour ;
- horaires de prière ou lien vers le calendrier ;
- horaires d’iqama lorsque la mosquée les fournit ;
- Jumu’ah : présence, horaires et langue éventuelle ;
- espace femmes : oui, non, inconnu ;
- accès PMR : oui, non, inconnu ;
- ablutions, parking, entrée séparée, salle de classe ou Quran si confirmés ;
- téléphone, site ou réseaux sociaux officiels ;
- source, date de mise à jour et niveau de confiance.

Ne jamais transformer une donnée inconnue en « non ». Utiliser trois états pour les équipements : `true`, `false`, `unknown`.

### JSON type

```json
{
  "id": "salawat_mosque_8f3c",
  "source": "salawat|osm|google|mapbox|foursquare",
  "sourceId": "node/123456789",
  "name": "Mosquée Al Hidaya",
  "type": "mosque",
  "location": {
    "latitude": 48.85661,
    "longitude": 2.35222,
    "geohash": "u09tv"
  },
  "address": {
    "street": "12 rue Exemple",
    "postalCode": "75001",
    "city": "Paris",
    "countryCode": "FR",
    "formatted": "12 rue Exemple, 75001 Paris"
  },
  "distanceMeters": 850,
  "walkingDistanceMeters": null,
  "walkingDurationSeconds": null,
  "openingStatus": "open|closed|unknown",
  "openingHours": [],
  "jumuah": [
    {"time": "13:30", "language": "fr", "note": null}
  ],
  "iqama": {
    "fajr": null,
    "dhuhr": "13:45",
    "asr": null,
    "maghrib": null,
    "isha": "21:30"
  },
  "facilities": {
    "womenSpace": "yes|no|unknown",
    "wheelchairAccess": "yes|no|unknown",
    "ablution": "yes|no|unknown",
    "parking": "yes|no|unknown"
  },
  "contacts": {
    "phone": null,
    "website": null,
    "mapsUrl": null
  },
  "sourceMeta": {
    "provider": "osm",
    "lastImportedAt": "2026-11-21T18:00:00Z",
    "lastVerifiedAt": null,
    "confidence": "high|medium|low",
    "attribution": "OpenStreetMap contributors"
  },
  "userState": {
    "isFavorite": false,
    "lastReportedAt": null
  }
}
```

### Tables backend recommandées

Ajouter ou faire évoluer les tables suivantes :

- `mosques` : identité Salawat, nom, type, coordonnées, adresse normalisée, statut et confiance ;
- `mosque_sources` : fournisseur, identifiant externe, date d’import, payload minimal autorisé et attribution ;
- `mosque_schedules` : horaires d’adhan, d’iqama et Jumu’ah avec période de validité ;
- `mosque_facilities` : valeurs `yes/no/unknown`, source et date de vérification ;
- `mosque_reports` : erreur signalée, type, commentaire, statut, auteur et modérateur ;
- `mosque_edits` : historique de version avant/après ;
- `mosque_members` : rôles de gestion déjà prévus dans Salawat.

Créer un index spatial si MySQL le permet dans la configuration utilisée. À défaut, utiliser une grille/geohash pour préfiltrer, puis calculer la distance exacte côté serveur avec la formule de Haversine.

### Signalement et ajout communautaire

Depuis la fiche, proposer :

- « Signaler une erreur » : fermée, déplacée, doublon, mauvais horaire, mauvais équipement, autre ;
- « Ajouter une mosquée » : nom, adresse, position, téléphone/site facultatifs ;
- « Mettre à jour les horaires » : Jumu’ah, iqama ou calendrier ;
- « Je confirme ces informations » : action légère, avec date et éventuellement zone approximative.

Le flux recommandé est : contribution utilisateur → validation automatique minimale → file de modération → historique → publication. Les modifications sensibles — coordonnées, fermeture, horaires officiels — doivent conserver l’ancienne valeur et l’auteur de la décision.

Ajouter un anti-abus simple : limitation par compte/appareil, détection de doublon par distance et nom, possibilité d’annuler une contribution, et journal d’audit.

---

## 3. UX/UI recommandée

### Écran « Mosquées »

L’écran doit s’ouvrir sur la position connue la plus récente, sans relancer inutilement le GPS. En haut :

- ville ou quartier détecté ;
- bouton « changer » pour rechercher manuellement ;
- état de fraîcheur : « mis à jour il y a… » ;
- bouton de réglages de rayon.

Sous l’en-tête :

- bascule **Carte / Liste** persistée ;
- filtres horizontaux : `Distance`, `Ouverte maintenant`, `Jumu’ah`, `Espace femmes`, `PMR`, `Favoris` ;
- rayon : 1 km, 3 km, 5 km, 10 km, personnalisé.

### Carte

La carte doit afficher des marqueurs regroupés lorsque la zone est dense. Un appui sur un marqueur ouvre une fiche courte en bas de l’écran avec nom, distance, état et bouton « Détails ». Ne pas afficher simultanément trop de texte sur la carte.

Prévoir une légende discrète : source, date de mise à jour et attribution. Les marqueurs de sources externes ne doivent pas être présentés comme vérifiés par Salawat.

### Liste

La liste est triée par distance par défaut. Chaque `MosqueCard` doit afficher :

- nom ;
- distance ;
- statut d’ouverture ;
- Jumu’ah ou prochain iqama si connu ;
- deux actions maximum : `Détails` et `Itinéraire` ;
- icône favorite.

Sur petit écran, ne pas afficher tous les équipements dans la carte. Les détails apparaissent dans la fiche.

### Actions d’itinéraire

Le bouton doit ouvrir l’application disponible :

1. Google Maps si installé ou URL universelle Google Maps ;
2. Apple Plans sur iOS ;
3. Waze si l’utilisateur le choisit ;
4. navigateur en solution de secours.

La destination doit être transmise avec latitude, longitude et nom encodés. Salawat ne doit pas tenter de reproduire la navigation routière.

### Autorisation GPS refusée

Présenter un écran utile, sans blocage :

> « Autorisez la localisation pour voir les mosquées autour de vous, ou recherchez une ville, un code postal ou une adresse. »

Actions : `Autoriser`, `Rechercher manuellement`, `Réessayer`. Si l’utilisateur choisit une ville, mémoriser la recherche comme contexte temporaire et l’indiquer clairement.

### Mode hors ligne

Si une dernière zone est disponible :

- afficher les mosquées en cache ;
- afficher une bannière « Dernière mise à jour… » ;
- désactiver les informations nécessitant une fraîcheur garantie ;
- autoriser l’ouverture des détails stockés ;
- permettre l’ouverture d’un itinéraire si l’application cartographique peut gérer la destination.

Si aucun cache n’existe, proposer une recherche manuelle sans prétendre fournir des résultats hors ligne.

### Zone sans résultat

Ne pas afficher seulement « Aucun résultat ». Proposer :

- élargir le rayon ;
- rechercher une ville voisine ;
- retirer les filtres ;
- ajouter une mosquée ;
- signaler qu’une mosquée connue manque.

Afficher la cause si elle est connue : localisation approximative, fournisseur indisponible, filtres trop restrictifs ou absence de données.

### Ton et accessibilité

Le ton doit être calme et factuel. Éviter les formulations culpabilisantes. Prévoir contraste suffisant, grandes zones tactiles, labels VoiceOver/TalkBack, tailles de texte dynamiques et mode sombre. Un résultat « inconnu » doit être exprimé textuellement, pas uniquement par une couleur.

---

## 4. Architecture technique et performance

### Flux général

```text
Écran Mosquées
   ↓ demande de position ponctuelle
LocationService
   ↓ latitude, longitude, précision, fuseau
trpc.mosques.nearby
   ↓ vérifie cache géographique
MosqueRepository
   ├─ Base Salawat validée
   ├─ OSM/Overpass adapter
   └─ fournisseur commercial optionnel
   ↓ normalisation + déduplication + distance
Réponse courte vers Expo
   ↓
Carte / Liste / Fiche
```

### Règles de géolocalisation économes

- Demander une position uniquement à l’ouverture de l’écran, lors d’un geste « actualiser » ou après un déplacement significatif.
- Ne pas utiliser de suivi GPS continu pour une liste de mosquées.
- Utiliser une précision équilibrée (`balanced`) pour la recherche ; réserver la haute précision à un besoin de navigation qui relève de l’application externe.
- Conserver la dernière position avec sa précision et son heure.
- Déclencher une nouvelle recherche si l’utilisateur s’est déplacé de 500 m à 1 km, ou si la dernière recherche dépasse un TTL défini.
- Arrêter immédiatement le listener lorsque l’écran est démonté.
- Ne pas envoyer la position à plusieurs fournisseurs depuis le client.

### Cache géographique

Clé recommandée :

```text
provider:osm
cell:u09tv
radius:5000
filters:jumuah,women
language:fr
queryVersion:3
```

Politique :

- cache partagé backend, pas un cache par appareil uniquement ;
- réponse fraîche si le cache est récent ;
- réponse stale immédiatement si le fournisseur est lent, puis rafraîchissement en arrière-plan ;
- déduplication des requêtes concurrentes sur la même clé ;
- TTL plus court pour le statut « ouvert maintenant » et plus long pour les coordonnées ;
- invalidation après correction validée ;
- conservation de la source, de l’attribution et de la date ;
- limitation du rayon et du nombre de résultats.

Pour Google, Mapbox et Foursquare, le cache doit respecter les règles propres à chaque fournisseur. Il ne faut pas appliquer une politique générique de persistance aux données commerciales [6][8][11].

### Requête Overpass côté backend

Exemple conceptuel à exécuter côté serveur, avec rayon borné :

```overpass
[out:json][timeout:25];
(
  nwr[amenity=place_of_worship][religion=muslim](around:5000,48.8566,2.3522);
  nwr[place_of_worship=musalla](around:5000,48.8566,2.3522);
);
out center tags;
```

En production :

- utiliser une instance Overpass choisie avec une URL de configuration ;
- ajouter un User-Agent descriptif ;
- mettre un timeout et une taille maximale ;
- gérer 429, 504 et erreurs JSON ;
- fusionner les résultats nœuds/aires/relations ;
- ne jamais lancer une requête par carte affichée ou par marqueur ;
- journaliser le fournisseur, le rayon, la durée et la taille de réponse.

### Parsing et normalisation TypeScript

```ts
// server/mosques/overpass.ts
export type OverpassElement = {
  type: "node" | "way" | "relation";
  id: number;
  lat?: number;
  lon?: number;
  center?: { lat: number; lon: number };
  tags?: Record<string, string>;
};

export function parseOverpassElement(element: OverpassElement) {
  const point = element.lat != null && element.lon != null
    ? { latitude: element.lat, longitude: element.lon }
    : element.center
      ? { latitude: element.center.lat, longitude: element.center.lon }
      : null;

  if (!point) return null;

  const tags = element.tags ?? {};
  const name = tags.name?.trim() || tags["name:fr"]?.trim();
  if (!name) return null;

  return {
    source: "osm" as const,
    sourceId: `${element.type}/${element.id}`,
    name,
    type: tags["place_of_worship"] === "musalla" ? "musalla" : "mosque",
    location: point,
    address: {
      street: tags["addr:street"] ?? null,
      postalCode: tags["addr:postcode"] ?? null,
      city: tags["addr:city"] ?? null,
      formatted: null,
    },
    facilities: {
      womenSpace: "unknown" as const,
      wheelchairAccess: tags.wheelchair === "yes"
        ? "yes" as const
        : tags.wheelchair === "no" ? "no" as const : "unknown" as const,
      ablution: "unknown" as const,
      parking: "unknown" as const,
    },
    sourceMeta: {
      provider: "osm",
      confidence: "medium" as const,
      lastImportedAt: new Date().toISOString(),
      attribution: "OpenStreetMap contributors",
    },
  };
}
```

### Procédure tRPC de proximité

```ts
// server/routers.ts — structure indicative à adapter au routeur existant
export const mosquesRouter = router({
  nearby: publicProcedure
    .input(z.object({
      latitude: z.number().min(-90).max(90),
      longitude: z.number().min(-180).max(180),
      radiusMeters: z.number().int().min(500).max(10000).default(5000),
      filters: z.object({
        openNow: z.boolean().optional(),
        jumuah: z.boolean().optional(),
        womenSpace: z.boolean().optional(),
        wheelchair: z.boolean().optional(),
      }).default({}),
      language: z.string().default("fr"),
    }))
    .query(async ({ input, ctx }) => {
      const key = makeGeoCacheKey(input);
      const cached = await ctx.cache.get(key);
      if (cached && !cached.isStale) return cached.value;

      const local = await findValidatedMosques(ctx.db, input);
      const osm = await ctx.mosqueProviders.osm.search(input);
      const merged = deduplicateAndRank([...local, ...osm], input);

      await ctx.cache.set(key, {
        value: merged,
        fetchedAt: new Date().toISOString(),
      });
      return merged;
    }),
});
```

Dans une implémentation réelle, `new Date()` doit être encapsulé pour faciliter les tests, les erreurs fournisseur doivent être capturées et le cache stale doit pouvoir être retourné lorsque l’appel distant échoue.

### Affichage React Native minimal

```tsx
const nearbyQuery = trpc.mosques.nearby.useQuery(
  location
    ? {
        latitude: location.latitude,
        longitude: location.longitude,
        radiusMeters: selectedRadius,
        filters,
        language: "fr",
      }
    : skipToken,
  { staleTime: 5 * 60 * 1000 }
);

if (nearbyQuery.isLoading) return <MosqueListSkeleton />;
if (nearbyQuery.error) return <MosqueErrorState onRetry={nearbyQuery.refetch} />;
if (!nearbyQuery.data?.length) return <NoMosqueResults onExpandRadius={expandRadius} />;

return (
  <FlatList
    data={nearbyQuery.data}
    keyExtractor={(item) => item.id ?? `${item.source}-${item.sourceId}`}
    renderItem={({ item }) => <MosqueCard mosque={item} />}
  />
);
```

Pour Expo Web, prévoir une carte et une stratégie de permission différentes. La recherche manuelle doit rester utilisable lorsque la géolocalisation navigateur n’est pas accordée.

---

## 5. Plan d’implémentation par étapes

### Étape 1 — Fondations, 2 à 4 jours

- définir le modèle `Mosque` commun ;
- ajouter source, date et confiance à chaque résultat ;
- créer l’adaptateur `MosqueProvider` ;
- implémenter la localisation ponctuelle ;
- ajouter la recherche manuelle par ville/code postal ;
- ajouter l’attribution OSM ;
- afficher clairement les données issues du cache.

### Étape 2 — OSM/Overpass, 3 à 5 jours

- créer la procédure tRPC `mosques.nearby` ;
- ajouter requête OSM groupée et parsing ;
- dédupliquer nœuds/aires/relations ;
- calculer distance et tri ;
- gérer timeout, 429, 504 et réponse vide ;
- ajouter cache géographique et déduplication concurrente.

### Étape 3 — UX carte/liste, 3 à 5 jours

- bascule carte/liste ;
- filtres rapides ;
- fiche détaillée ;
- favoris ;
- itinéraire Google Maps/Apple Plans/Waze ;
- états GPS refusé, hors ligne et sans résultat ;
- accessibilité et mode sombre.

### Étape 4 — Contribution, 1 à 2 semaines

- signalement d’erreur ;
- ajout de mosquée ;
- détection de doublons ;
- file de modération ;
- historique des modifications ;
- enrichissement Jumu’ah, femmes, PMR, ablutions, parking et iqama.

### Étape 5 — Fournisseurs complémentaires, après mesure

N’ajouter qu’un fournisseur commercial après avoir mesuré :

- taux de résultats OSM par ville ;
- doublons ;
- taux de signalements ;
- coût par recherche ;
- latence ;
- besoin réel de données absentes.

Chaque fournisseur doit être isolé derrière le même contrat et désactivable par configuration.

---

## 6. Tests et critères d’acceptation

### Fonctionnels

- Une position valide affiche les lieux dans le rayon sélectionné.
- La liste est triée par distance calculée côté serveur.
- La carte et la liste représentent le même jeu de résultats.
- Les filtres ne transforment pas `unknown` en `false`.
- Un lieu sans horaires n’est pas présenté comme fermé.
- Le bouton itinéraire fonctionne avec une solution de secours.
- Une correction utilisateur conserve un historique.

### Résilience

- GPS refusé : recherche manuelle fonctionnelle.
- Réseau indisponible : dernier cache affiché avec son âge.
- Overpass en 429/504 : réponse stale ou message explicite, sans crash.
- Zone dense : marqueurs regroupés et temps de réponse acceptable.
- Aucun résultat : élargissement de rayon et ajout manuel proposés.

### Performance et vie privée

- Aucun suivi GPS continu pour l’écran Mosquées.
- Pas d’appel fournisseur à chaque rendu React.
- Les appels sont regroupés et dédupliqués.
- La clé Google/Mapbox/Foursquare reste côté serveur.
- La position précise n’est pas conservée plus longtemps que nécessaire sans consentement.
- Les données commerciales ne sont pas copiées dans MySQL sans autorisation explicite.

---

## Les trois pièges les plus fréquents

### 1. Interroger une API à chaque ouverture ou chaque mouvement de carte

Cela crée de la latence, épuise la batterie, dépasse les quotas et peut provoquer un blocage. Utiliser un cache géographique backend, un rayon borné, un debounce, une déduplication des requêtes et un rafraîchissement stale-while-revalidate.

### 2. Supposer qu’un résultat vide signifie qu’il n’existe aucune mosquée

Les salles de prière de quartier sont souvent mal étiquetées ou absentes des bases. Combiner plusieurs tags et variantes linguistiques, afficher la source et l’incertitude, et proposer l’ajout communautaire avec modération.

### 3. Stocker les données d’un fournisseur sans vérifier sa licence

OSM, Google, Mapbox et Foursquare n’ont pas les mêmes règles d’attribution, de cache et de redistribution. Isoler les adaptateurs, ne persister que les champs autorisés et conserver une base Salawat indépendante pour les données validées.

---

## Références

[1]: https://www.openstreetmap.org/copyright "OpenStreetMap Copyright and Licence"
[2]: https://dev.overpass-api.de/overpass-doc/en/preface/commons.html "Overpass API Commons and Usage Limits"
[3]: https://developers.google.com/maps/documentation/places/web-service/place-types "Google Places API Place Types"
[4]: https://developers.google.com/maps/documentation/places/web-service/nearby-search "Google Places API Nearby Search"
[5]: https://developers.google.com/maps/billing-and-pricing/pricing "Google Maps Platform Pricing"
[6]: https://developers.google.com/maps/documentation/places/web-service/usage-and-billing "Google Places API Usage and Billing"
[7]: https://docs.mapbox.com/api/search/search-box/ "Mapbox Search Box API"
[8]: https://www.mapbox.com/pricing "Mapbox Pricing"
[9]: https://location.foursquare.com/places/docs/supported-countries "Foursquare Supported Countries and Coverage"
[10]: https://docs.foursquare.com/developer/reference/rate-limits "Foursquare API Rate Limits"
[11]: https://docs.foursquare.com/developer/reference/usage-guidelines "Foursquare API Usage Guidelines"
[12]: https://operations.osmfoundation.org/policies/nominatim/ "Nominatim Usage Policy"
[13]: https://developers.google.com/maps/documentation/places/web-service/policies "Google Places API Policies"
[14]: https://operations.osmfoundation.org/policies/tiles/ "OpenStreetMap Tile Usage Policy"
