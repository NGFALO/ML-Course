# 🧠 ML Mémos — Contexte Projet (pour Claude)

> Lis ce fichier puis `ml_memos.html` et tu es opérationnel.

---

## 👤 Profil utilisateur — Falo

- **Objectif** : Maîtriser les termes ML pour des interviews en entreprise
- **Style d'apprentissage** : Très visuel — retient par images mentales et scénarios concrets
- **Niveau** : Apprenant ML (pas développeur expert, mais capable de suivre du code)
- **Email** : waterp974@gmail.com
- **GitHub** : ngfalo
- **Préférences Claude** : Réponses courtes et directes, pas de verbosité inutile

---

## 📁 Fichier principal

**`ml_memos.html`** — Application web single-file (HTML + CSS + JS vanilla)

- Chemin local : `C:\Users\falo-\Documents\GitRepos\ML-Course\ml_memos.html`
- URL GitHub Pages : `https://ngfalo.github.io/ML-Course/ml_memos.html`
- Repo GitHub : `github.com/ngfalo/ML-Course`
- Taille actuelle : ~850 lignes (redesign v2 complet)
- Version actuelle : **v2.4**

---

## 🏗️ Architecture technique

### Règles techniques IMPORTANTES
- **PAS de `type="module"`** sur le script principal — ça casse les `onclick=""` inline
- **Firebase Compat SDK** (pas ESM) — chargé via `<script src>` séparés APRÈS le script principal
- **Objet global `APP`** — tous les handlers onclick appellent `APP.methode()`
- **État global `S`** — objet unique contenant tout l'état (tab, quiz, anki, stats, user…)
- **`render()`** — reconstruit `#content` innerHTML selon `S.tab`
- **Les données (TERMS) ne sont pas dans le HTML** — elles viennent de Google Sheets

### Structure du fichier HTML
```
<head>       : meta + Google Fonts (Baloo 2, Hanken Grotesk) + CSS + theme-init script
<body>       : .app-bg > .app-col > #top-bar + #content + #bottom-nav + #abandon-modal
<script>     : constantes → état → helpers → Valkyrie SVG → data loader → Firebase
               → render helpers (home/gloss/quiz/anki/profil) → render() → APP object → init()
fin <body>   : Firebase Compat CDN scripts + initFirebase() + init()
```

### Design system (redesign v2)
- **Palette** : fond crème `--bg:#E9E4DB`, cartes blanches, dark mode via `data-theme="dark"` sur `<html>`
- **Polices** : Baloo 2 (titres/chiffres) + Hanken Grotesk (corps)
- **Layout** : colonne 480px max, centré desktop avec border-radius 32px + ombre
- **Top bar** : chips 🔥 streak / ⭐ XP / 🎯 mastery% + toggle thème + auth Google
- **Bottom nav** : 5 onglets — Accueil 🏠 · Glossaire 📖 · Quiz 🎯 · Révision 🃏 · Profil 📈
- **Valkyrie mascot** : SVG inline via `vkSVG(mood, w, h)` — moods: happy/wow/oops/idle
- **Favicon** : Valkyrie simplifiée inline en data URI SVG dans `<link rel="icon">`
- **localStorage keys** : `mlmemos_theme`, `mlmemos_v2`, `mlmemos_session_v1`, `mlmemos_terms_cache_v1`

### Google Sheets — Source des données
- **URL pub** : `https://docs.google.com/spreadsheets/d/e/2PACX-1vTYMHIf-bdjE8v1BzY6X4GI-X0UjsohSAbyp3-x0Uj2KtC0IkbOYKw8kQqTEOtwzfHrdDiJwFUlkw4J/pub?output=csv`
- **Fetch stratégie** : `Promise.any([name-URL, GID-URL])` pour chaque onglet — couvre les deux modes de publication Google
- **GIDs** : Terms=`2111289426`, Rules=`378027362`, Scenarios=`1767817621`, TrueFalse=`1007638947`
- **Onglets** :
  - `Terms` — id, name, full, family, image, tip, d2t, t2d_correct, t2d_wrong1/2/3
  - `Rules` — term_id, val, label, type
  - `TrueFalse` — term_id, statement, answer (TRUE/FALSE), explanation
  - `Dictionary` — word, definition (chargé en background, enrichit DICT)
  - `Scenarios` — **obsolète** (données malformées, le mode Scénario ne l'utilise plus)
- **Cache** : `mlmemos_terms_cache_v1` en localStorage — hydrate instantanément au reload

### Firebase
```js
const FIREBASE_CONFIG = {
  apiKey: "AIzaSyCSJyr0WHkmQUZz32n1OCZhuuh3k2IrSYI",
  authDomain: "ml-course-a11e7.firebaseapp.com",
  projectId: "ml-course-a11e7",
  storageBucket: "ml-course-a11e7.firebasestorage.app",
  messagingSenderId: "1050580397485",
  appId: "1:1050580397485:web:04f892ec7afc7c4ed58524"
};
```
- Collection : `progress`, doc : `user.uid`
- **Auth : Google Sign-In** avec `prompt:'select_account'` (force le sélecteur de compte)
- SDK : `firebase-app-compat.js`, `firebase-auth-compat.js`, `firebase-firestore-compat.js`
- **Règles Firestore** (à jour) : `allow read, write: if request.auth.uid == userId` — chaque user ne voit que ses données
- Sign-out : vide `S.stats` + `render()` pour effacer les données de l'ancien compte

---

## 🎯 Fonctionnalités implémentées

### Onglet 🏠 Accueil
- Carte salutation + Valkyrie animée (mood selon objectif du jour atteint ou non)
- Ring SVG : progression objectif du jour (DAILY_GOAL = 12 questions)
- CTA "Commencer/Continuer la session" + raccourci Révision du jour
- Grille des 6 domaines avec barre de maîtrise par famille

### Onglet 📖 Glossaire
- Termes ML en 6 familles : `eval`, `model`, `classif`, `summary`, `libs`, `choose`
- Filtrage par famille + recherche texte **sans perte de focus** (seul `#gloss-cards` est re-rendu, pas tout le tab)
- Chaque carte : image mentale, seuils (good/warn/bad), tip, badge ⭐ si maîtrisé

### Onglet 🎯 Quiz
- **7 modes** : Mixte · Scénario · Vrai/Faux · Terme→Déf · Déf→Terme · ⚡ Speed (15s/q) · 💀 Boss (1 vie)
- **Mode Scénario** : affiche `term.image` comme question, 4 choix = noms de termes (1 correct + 3 aléatoires) — **ne dépend plus de l'onglet Scenarios**
- **Sessions** : sauvegarde auto `mlmemos_session_v1` TTL 30min, reprise au rechargement
- **Navigation lockée** : modal d'abandon si on quitte pendant une session
- **Fin de session** : score %, XP (+10/bonne, +20 si 100%, +10 si ≥80%), liste des erreurs
- **Dictionnaire inline** : bouton `?` → panel avec les termes ML détectés dans la question courante

### Onglet 🃏 Révision (Anki)
- Répétition espacée SRS : intervalles 1j · 2j · 4j · 7j · 14j
- Clic = révèle image mentale + tip
- "✅ Je savais" / "❌ À revoir" — erreur repasse dans 5 minutes
- 5 ✓ consécutives = ⭐ Maîtrisé (revient 1× sur 5 dans la queue)

### Onglet 📈 Profil (remplace Historique)
- Carte niveau (XP/100 par niveau) + barre maîtrise globale
- Stats : bonnes/mauvaises réponses totales
- Tableau de tous les termes triés par maîtrise (pips colorés)

### Sync Firebase
- `saveAll()` : debounce 800ms → Firestore + localStorage
- `loadCloud()` : au login, charge la progression du compte
- `flashSync()` : message discret "☁️ Sauvegardé" / "⚠️ Sync échouée"
- Sign-out : reset `S.stats` + re-render immédiat

### XP & Streak
- `bumpDaily(gotXp)` : appelé après chaque réponse — met à jour streak, todayCount, XP
- Streak : incrémente si lastActive = hier, reset à 1 sinon
- XP : +10 par bonne réponse + bonus de fin de session

---

## 📊 Structure d'un terme (TERMS array)

```js
{
  id: 'r2',
  name: 'R²',
  full: 'Coefficient de Détermination',
  family: 'eval',           // eval | model | classif | summary | libs | choose
  image: '🎯 ...',          // image mentale — utilisée comme question en mode Scénario
  tip: 'Astuce...',
  d2t: 'Définition...',     // question mode Déf→Terme
  t2d: ['correct', 'w1', 'w2', 'w3'],  // index 0 = correct
  rules: [{val, label, type}],          // good | warn | bad
  tf: [{stmt, answer:bool, explain}],   // Vrai/Faux
  scenarios: [...],                     // obsolète, non utilisé
}
```

### Types de questions dans `makeCur()`
| type  | question              | choices                        |
|-------|-----------------------|--------------------------------|
| `t2d` | "Que signifie X ?"    | t2d array (1 correct + 3 wrong)|
| `d2t` | d2t text              | 4 noms de termes               |
| `img` | term.image (scénario) | 4 noms de termes               |
| `tf`  | tf.stmt               | VRAI / FAUX                    |

---

## 🐛 Bugs corrigés importants

- **Quiz ne se lance pas** : `loadTerms()` fetchait le mauvais onglet (GID Terms ≠ premier onglet). Fix : `Promise.any([name-URL, GID-URL])` pour tous les onglets.
- **Scénarios vides** : données malformées dans l'onglet Scenarios. Fix : mode Scénario utilise désormais `type:'img'` (image mentale → identifier le terme).
- **Recherche glossaire perd le focus** : `render()` complet détruisait l'input. Fix : `setGSearch` ne re-rend que `#gloss-cards`.
- **Sign-out sans refresh** : `onAuthStateChanged` ne re-rendait pas. Fix : reset `S.stats` + `render()` sur user=null.
- **Connexion sans sélecteur de compte** : Fix : `prompt:'select_account'` dans GoogleAuthProvider.
- **Navigation lockée cassée** : `APP.showAbandon()` → `APP.confirmAbandon()` (typo).

---

## 📝 Décisions de design

- **Single-file HTML** : tout dans un seul fichier, pas de framework
- **Firebase Compat SDK** : choisi car ESM casse les `onclick=""` inline
- **Google Sheets** : source de données éditable sans toucher au HTML
- **Pas d'onglet Scenarios** : données trop difficiles à maintenir proprement — le mode Scénario génère les questions depuis `term.image` dynamiquement
- **Pas de PWA pour l'instant** : idée notée — PWA + pwabuilder.com pour APK Android plus tard
- **Rendu innerHTML** : pas de framework, `render()` reconstruit le tab courant complet (sauf glossaire search)

---

## 🔐 Sécurité

- **Firestore rules** : `allow read, write: if request.auth != null && request.auth.uid == userId` — chaque user ne voit que ses données ✅
- **Firebase API key publique** : normal pour le client-side, pas un risque si les rules sont correctes
- **Repo public GitHub** : aucun secret réel — la clé Firebase est conçue pour être publique

---

## 🚀 Déployer une nouvelle version

```bash
git add ml_memos.html
git commit -m "description du changement"
git push
```

---

## 💬 Comment reprendre avec Claude

> *"Lis CONTEXTE_PROJET.md et reprends le projet."*

Pour modifier du **contenu** (termes, quiz) :
> *"Lis le Google Sheet [URL] et [modifie / ajoute / vérifie] ..."*
