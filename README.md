# VisuGraph

Application web de visualisation de fonctions mathématiques, sans installation ni serveur. Un seul fichier HTML à ouvrir dans un navigateur.

## Démarrage

```
Ouvrir index.html dans n'importe quel navigateur moderne.
```

Aucune dépendance à installer. Les librairies (Chart.js, mathjs) sont chargées depuis un CDN.

---

## Fonctionnalités

### Graphiques

Chaque graphique est une carte indépendante. On peut en créer autant que souhaité via le bouton **+ Graphique**.

Chaque graphique possède :
- un **nom**
- une **variable X** personnalisable (défaut : `n`)
- une **plage X** (min, max, pas)
- des **labels d'axes** optionnels
- une **échelle Y** automatique ou manuelle

Les graphiques sont **réordonnables** via les boutons ↑ ↓ dans l'en-tête de chaque carte.

---

### Courbes

Sur chaque graphique on peut ajouter plusieurs courbes. Chaque courbe a :
- un **nom**
- une **formule** en syntaxe mathjs (ex : `ceil(n/7)`)
- une **couleur** (palette de 11 couleurs prédéfinies + color picker libre)
- un **état actif/inactif** (case à cocher) — la courbe est conservée mais masquée

La section courbes est **repliable** via le bouton ▼ à côté du compteur d'activité.

---

### Variables

Le système de variables permet de paramétrer les formules sans les réécrire. Il fonctionne sur **trois niveaux de portée**, avec surcharge en cascade :

```
Courbe  >  Graphique  >  Global
```

Une variable définie à un niveau plus proche de la courbe prend la priorité sur les niveaux supérieurs.

#### Variables globales
Panneau en haut de la page. S'appliquent à toutes les formules de tous les graphiques.

#### Variables de graphique
Section dans chaque carte (repliable). S'appliquent à toutes les courbes de ce graphique. Peuvent surcharger les globales.

#### Variables locales de courbe
Section dans la modal "Modifier la courbe". S'appliquent uniquement à cette courbe. Peuvent surcharger graphique et global.

**Exemple :**
```
Global  :  k = 7
Graphique A :  k = 30       ← surcharge k pour ce graphique
Courbe 1 :  (aucune)        ← hérite k = 30
Courbe 2 :  k = 5           ← surcharge k pour cette courbe uniquement
```

Les variables peuvent se **référencer entre elles** dans l'ordre de définition :
```
a = 10
b = a * 2    → b vaut 20
```

Chaque variable affiche une préview `≈ valeur` calculée en temps réel.

---

### Syntaxe mathjs

La variable X est configurable par graphique (défaut : `n`). Les variables définies dans les différentes portées sont automatiquement disponibles dans les formules.

| Syntaxe | Description | Exemple |
|---|---|---|
| `+  -  *  /` | Opérations de base | `n * 2 + 1` |
| `n^2` | Puissance | `n^3 - n` |
| `sqrt(x)` | Racine carrée | `sqrt(n)` |
| `abs(x)` | Valeur absolue | `abs(n - 5)` |
| `ceil(x)` | Arrondi supérieur | `ceil(n / 7)` |
| `floor(x)` | Arrondi inférieur | `floor(n / 3)` |
| `round(x)` | Arrondi | `round(n / 4)` |
| `log(x)` | Logarithme naturel | `log(n + 1)` |
| `log(x, b)` | Logarithme base b | `log(n, 2)` |
| `sin(x)` | Sinus (radians) | `sin(n * pi)` |
| `cos(x)` | Cosinus (radians) | `cos(n * 2)` |
| `pi` | Constante π ≈ 3.14159 | `sin(n / pi)` |
| `e` | Constante e ≈ 2.71828 | `e^n` |
| `max(a, b)` | Maximum | `max(n, 0)` |
| `min(a, b)` | Minimum | `min(n^2, 10)` |

L'icône **?** en haut à droite affiche ce tableau de référence dans l'application.

---

### Sauvegarde

Toutes les données (graphiques, courbes, variables, paramètres) sont **sauvegardées automatiquement dans le `localStorage`** du navigateur et restaurées au rechargement de la page.

---

### Export / Import JSON

Les boutons **↓ Exporter** et **↑ Importer** dans l'en-tête permettent de :
- **Exporter** : télécharge un fichier `visuGraph_YYYY-MM-DD.json` contenant tous les graphiques, courbes et variables globales.
- **Importer** : charge un fichier JSON précédemment exporté, en remplaçant l'état actuel (avec sauvegarde dans le localStorage).

Le format JSON est rétro-compatible avec les exports sans variables globales.

---

---

### Mode sombre

Bouton 🌙/☀ dans l'en-tête. La préférence est persistée dans le localStorage.

---

## PWA — Mode hors ligne

VisuGraph est une Progressive Web App. Après le premier chargement en ligne, l'application fonctionne **entièrement hors réseau**.

Le service worker (`sw.js`) met en cache :
- `index.html`, `manifest.json`, `icon.svg`
- Chart.js et mathjs depuis jsDelivr

Stratégie **cache-first** : les ressources déjà vues sont servies instantanément. Les nouvelles ressources sont mises en cache à la volée.

> ⚠️ Le service worker nécessite **HTTPS** (ou `localhost`).

Sur mobile, l'application peut être installée sur l'écran d'accueil via le menu du navigateur.

---

## Déploiement via GitHub Actions

Le workflow `.github/workflows/deploy.yml` se déclenche automatiquement à chaque push sur `main` et déploie les fichiers via FTP.

À chaque déploiement, la version du cache service worker est automatiquement mise à jour avec le SHA du commit, garantissant que les utilisateurs reçoivent bien la nouvelle version.

### Secrets à configurer

Dans **Settings → Secrets and variables → Actions** du dépôt GitHub :

| Secret | Valeur |
|---|---|
| `FTP_HOST` | Hôte FTP (ex : `ftp.mondomaine.fr`) |
| `FTP_USER` | Identifiant FTP |
| `FTP_PASSWORD` | Mot de passe FTP |

Les fichiers `.git/`, `.github/` et `README.md` sont exclus du transfert.

---

## Architecture

```
index.html                        ← application complète (HTML + CSS + JS)
manifest.json                     ← manifest PWA
sw.js                             ← service worker (cache offline)
icon.svg                          ← icône PWA
.github/workflows/deploy.yml      ← CI/CD déploiement FTP
```

### Dépendances CDN
- **Chart.js 4.4** — rendu des graphiques
- **mathjs 12.4** — évaluation des formules

### Stockage
```json
localStorage["visuGraph_data"] = {
  "version": 1,
  "globalVars": [{ "id": "...", "name": "k", "value": "7" }],
  "graphs": [{
    "id": "...",
    "name": "Mon graphique",
    "variable": "n",
    "xmin": 0, "xmax": 100, "xstep": 1,
    "xlabel": "", "ylabel": "",
    "yAuto": true, "ymin": 0, "ymax": 100,
    "vars": [{ "id": "...", "name": "factor", "value": "2" }],
    "varsCollapsed": true,
    "curves": [{
      "id": "...",
      "name": "Semaines",
      "formula": "ceil(n / k)",
      "color": "#3b82f6",
      "enabled": true,
      "vars": []
    }],
    "showAddForm": false,
    "curvesCollapsed": false
  }]
}
```
