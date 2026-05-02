# DOM TOM IPTV Premium 2026 — Guide de configuration & validation (outil open-source)

Ce README documente **comment configurer et valider une source IPTV (M3U/EPG) dédiée à DOM TOM pour 2026**, dans la perspective d’un **outil open-source de vérification** (ex. contrôle de structure des playlists, cohérence EPG, qualité des liens, normalisation des formats). L’objectif est de vous aider à **préparer une liste** exploitable et **repérable** avant d’en faire un usage applicatif (lecteur, frontend, service de relecture, etc.).

> ⚠️ Remarque : ce projet vise la **validation technique** (format, parsing, cohérence EPG) et l’**assistance à la configuration**. Il ne fournit pas de contenus, n’illustre pas le contournement de restrictions et ne garantit pas la disponibilité d’une offre commerciale.

---

## Fonctionnalités principales

- **Validation M3U** : détection de tags manquants, encodage, structure `#EXTINF`.
- **Contrôle EPG** : vérification des champs (titre, start/end), collisions d’horaires, duplication d’IDs.
- **Normalisation** : harmonisation des noms de chaînes et des clés de correspondance.
- **Rapports** : sortie console + exports (JSON/CSV) pour faciliter le diagnostic.

---

## Pré-requis

- **Node.js** 18+ (ou équivalent si vous utilisez un langage différent)
- Accès à :
  - une **URL M3U** ou un fichier `.m3u/.m3u8`
  - une source **EPG** (XMLTV ou équivalent)
- Un terminal avec `curl`/`wget` disponibles (recommandé pour tester les URLs)

---

## Installation

### Option A — via npm
```bash
git clone https://github.com/votre-org/validation-domtom-iptv.git
cd validation-domtom-iptv
npm install
```

### Option B — exécution directe
```bash
node ./src/index.js --help
```

---

## Configuration (focalisée sur la liste)

L’outil s’appuie généralement sur un fichier de config. Exemple : `config.yml`.

```yaml
source:
  m3u:
    type: url        # "url" ou "file"
    value: "https://EXEMPLE/m3u"
  epg:
    type: url        # "url" ou "file"
    value: "https://EXEMPLE/epg.xml"
validation:
  sanitizeNames: true
  requireExtinf: true
  matchEpgBy:
    - channelName
    - tvg-id
output:
  reportFormat: json
  reportPath: ./reports
```

### Règles de correspondance (recommandées)
Pour DOM TOM IPTV “Premium 2026”, les différences de conventions (nom de chaîne, `tvg-id`, qualité des tags) peuvent varier. C’est pourquoi la configuration `matchEpgBy` doit être pensée pour **réduire les taux de non-correspondance** :

- Priorité à `tvg-id` si le flux le fournit proprement.
- Fallback sur `channelName` quand les identifiants sont inconsistants.
- Activer `sanitizeNames` pour uniformiser accents/espaces/majuscules.

---

## Exemple de commande (validation + rapport)

```bash
npm run validate \
  -- --config ./config.yml \
  --verbose
```

Vous devriez obtenir un rapport du type :

- **Chaînes chargées** : `N`
- **Chaînes avec EPG associé** : `M`
- **Erreurs parse M3U** : `K`
- **Anomalies EPG** : horaires incohérents / titres vides

> Astuce : si `K` est élevé, commencez par tester l’accès réseau aux URLs (voir section suivante), puis examinez l’encodage de la M3U.

---

## Vérification rapide des URLs (diagnostic)

```bash
curl -I "https://EXEMPLE/m3u"
curl -I "https://EXEMPLE/epg.xml"
```

Si vous obtenez :
- **401/403** : l’URL nécessite probablement un en-tête/autorisation spécifique.
- **404** : lien obsolète — remettez à jour la configuration.
- **200 mais contenu vide** : vérifier `content-type` et la validité du fichier.

---

## Formatage de la liste : check-list DOM TOM

Avant d’intégrer la liste dans un lecteur, vérifiez :

- **Présence de `#EXTINF`** pour chaque entrée.
- Champs typiques utiles :
  - `tvg-name`
  - `tvg-id`
  - `group-title`
- **Cohérence des groupes** : `group-title` doit rester stable pour le mapping EPG.

Exemple attendu (simplifié) :
```m3u
#EXTM3U
#EXTINF:-1 tvg-id="DOMTOM.TF1" tvg-name="TF1" group-title="France"
http://EXEMPLE/stream/tf1.m3u8
```

---

## Intégration communautaire & références

Si vous cherchez des retours d’usage autour de DOM TOM IPTV Premium 2026 (notamment sur la disponibilité, les conventions de noms et les pratiques de mise à jour des playlists), vous pouvez consulter la discussion suivante :  
[à propos d’abonnement DOM TOM IPTV 2026 et de configuration des flux](https://www.reddit.com/user/Pomegranate_Hani/comments/1t0blr2/iptv_domtom_abonnement_2026_iptv_r%C3%A9union_antilles/).

---

## Community

- Discussions / support : consultez les fils communautaires et comparez vos logs de validation.
- Contribution : ajoutez des règles de normalisation et des heuristiques de matching EPG/M3U (avec tests).

---

## Référence

- Documentation de l’API / modules internes (selon votre implémentation)
- Schémas EPG (XMLTV et variantes)