# 🧠 ML Mémos — Contexte Projet (pour Claude)

> Colle ce fichier dans le dossier de projet et dis à Claude : "Lis CONTEXTE_PROJET.md et reprends le projet."

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

- Chemin local : `C:\Users\Utilisateur\Documents\Github_repos\ML-Course\ml_memos.html`
- URL GitHub Pages : `https://ngfalo.github.io/ML-Course/ml_memos.html`
- Repo GitHub : `github.com/ngfalo/ML-Course`
- Taille : ~1100 lignes, ~88KB

---

## 🏗️ Architecture technique

### Structure du fichier HTML
```
lignes 1–251     : <!DOCTYPE> + <style> CSS complet + HTML structure (header, nav, 4 tabs)
lignes 252–685   : <script> + const TERMS=[...] (30 termes) + const FI={...}
lignes 686–1107  : Tout le JS (fonctions) + Firebase config
lignes 1108–1112 : Firebase Compat CDN scripts + initFirebase()
```

### Règles techniques IMPORTANTES
- **PAS de `type="module"`** sur le script principal — ça casse les `onclick=""` inline
- **Firebase Compat SDK** (pas ESM) — chargé via `<script src>` séparés APRÈS le script principal
- Pour écrire le fichier : utiliser `mcp__workspace__bash` avec `cat > "/sessions/sweet-dreamy-gauss/mnt/ML Course/ml_memos.html"` (le Write tool ne peut pas écrire dans ce dossier directement)

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
- Collection : `progress`, doc : `user_XXXXXXXXX` (stocké dans `localStorage` sous `ml_memos_uid`)
- Domaine autorisé à ajouter : `ngfalo.github.io` (Firebase Console → Authentication → Authorized domains)

---

## 🎯 Fonctionnalités implémentées

### Onglet 📚 Glossaire
- 30 termes ML en 6 familles : `eval`, `model`, `classif`, `summary`, `libs`, `choose`
- Filtrage par famille + recherche texte
- Chaque carte : image mentale, seuils (good/warn/bad), tip
- Badge ⭐ "Maîtrisé" visible si 5 bonnes réponses Anki

### Onglet 🎯 Quiz
- **Mixte** : toutes questions mélangées (30 questions)
- **Scénario** : situations réelles avec 4 choix
- **Vrai/Faux**
- **Terme→Définition**
- **Définition→Terme**
- **⚡ Speed Round** : 15 questions, 15s par question, barre chrono colorée
- **💀 Boss Battle** : une seule erreur = game over
- Streak + score en temps réel
- Explication affichée après chaque réponse

### Onglet 🃏 Anki
- Répétition espacée (SRS) : intervalles 1j, 2j, 4j, 7j, 14j
- Clic sur carte = révèle image mentale + tip
- "✅ Je savais" / "❌ Je ne savais pas"
- 5 bonnes réponses consécutives = ⭐ Maîtrisé
- Cartes maîtrisées reviennent 1× sur 5
- Erreur : carte revient dans 5 minutes

### Onglet 📈 Historique
- Stats globales : maîtrisés / vus / nouveaux
- Barre de progression globale
- Liste tous les termes triés par maîtrise (pips verts)

### Sync Firebase
- `saveProgress()` : debounce 800ms, sauvegarde `{correct, wrong, bestStreak, cardStats, updatedAt}`
- `loadProgress()` : appelé au démarrage, restaure tout l'état
- `showSync()` : message discret "☁️ Sauvegardé" / "⚠️ Hors ligne"

---

## 📊 Structure d'un terme (TERMS array)

```js
{
  id: 'r2',                    // identifiant unique
  name: 'R²',                  // nom affiché
  full: 'Coefficient de Détermination',  // nom complet
  family: 'eval',              // eval | model | classif | summary | libs | choose
  image: '🎯 "..." ...',       // image mentale (texte riche)
  rules: [
    {val:'≥ 0.90', label:'Excellent ✅', type:'good'},   // good | warn | bad
    ...
  ],
  tip: 'Astuce...',            // conseil pratique (optionnel)
  d2t: 'Définition...',        // question mode Def→Terme
  t2d: ['bonne réponse', 'mauvais1', 'mauvais2', 'mauvais3'],  // index 0 = correct
  scenarios: [
    {q:'Question scénario?', choice:'bonne réponse', wrongs:['a','b','c'], answer:'explication'}
  ],
  tf: [
    {stmt:'Affirmation...', answer:true, explain:'Explication...'}
  ]
}
```

---

## 🐛 Bug corrigé (important à ne pas réintroduire)

**Bug** : Dans `buildQ()`, le code utilisait `qs.push({type:'sc', term:t, s})` (raccourci JS qui créait la propriété `s` au lieu de `sc`), ce qui faisait que `renderQ()` ne pouvait pas lire `q.sc.q` → aucun choix affiché en mode Scénario.

**Fix** : `qs.push({type:'sc', term:t, sc:sc})` — toujours écrire explicitement `sc:sc`.

---

## 📝 Décisions de design prises

- **Single-file HTML** : tout dans un seul fichier, pas de framework
- **Firebase Compat** : choisi car ESM casse les `onclick=""` inline
- **GitHub Pages** : hébergement gratuit, URL simple
- **Pas de VPS** : Falo n'a pas les compétences pour le gérer
- **Style** : colorful mais clean, family-pills colorées par catégorie
- **Gamification** : streak + "le moins d'erreurs possible" (pas de points complexes)

---

## 🚀 Pour déployer une nouvelle version sur GitHub

1. Modifie `ml_memos.html` localement
2. Va sur `github.com/ngfalo/ML-Course`
3. Clique `ml_memos.html` → ✏️ → colle le contenu → Commit

Ou en terminal :
```bash
git add ml_memos.html
git commit -m "description du changement"
git push
```

---

## 💬 Comment reprendre avec Claude

Dis simplement :
> *"Lis CONTEXTE_PROJET.md et reprends le projet ml_memos.html. Je veux [ajouter X / corriger Y / etc.]"*

Claude lira ce fichier + le fichier HTML et sera immédiatement opérationnel.
