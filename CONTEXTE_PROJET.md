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

- Chemin local : `C:\Users\falo-\Documents\GitRepos\ML-Course\ml_memos.html`
- URL GitHub Pages : `https://ngfalo.github.io/ML-Course/ml_memos.html`
- Repo GitHub : `github.com/ngfalo/ML-Course`
- Taille actuelle : ~763 lignes, ~47KB (données externalisées dans Google Sheets)

---

## 🏗️ Architecture technique

### Structure du fichier HTML
```
lignes 1–268     : <!DOCTYPE> + <style> CSS complet + HTML structure (header, nav, 4 tabs)
lignes 269–762   : <script> — tout le JS (data loader + logique + Firebase)
lignes 763–768   : Firebase Compat CDN scripts + initFirebase()
```

### Règles techniques IMPORTANTES
- **PAS de `type="module"`** sur le script principal — ça casse les `onclick=""` inline
- **Firebase Compat SDK** (pas ESM) — chargé via `<script src>` séparés APRÈS le script principal
- Pour écrire le fichier : utiliser `mcp__workspace__bash` avec Python ou `cat >` (le Write tool peut écrire directement dans ce dossier)
- **Les données (TERMS) ne sont plus dans le HTML** — elles sont dans Google Sheets

### Google Sheets — Source des données
- **URL de base** : `https://docs.google.com/spreadsheets/d/e/2PACX-1vTYMHIf-bdjE8v1BzY6X4GI-X0UjsohSAbyp3-x0Uj2KtC0IkbOYKw8kQqTEOtwzfHrdDiJwFUlkw4J/pub?output=csv&sheet=`
- **4 onglets** :
  - `Terms` — id, name, full, family, image, tip, d2t, t2d_correct, t2d_wrong1/2/3
  - `Rules` — term_id, val, label, type
  - `Scenarios` — term_id, question, correct_choice, wrong1/2/3, explanation
  - `TrueFalse` — term_id, statement, answer (TRUE/FALSE), explanation
- **Édition** : ouvrir le Google Sheet et modifier directement → le site se met à jour au prochain chargement
- **Ajout de terme** : ajouter une ligne dans `Terms` + lignes dans `Rules`/`Scenarios`/`TrueFalse`
- Pour que Claude vérifie ou modifie du contenu : donner l'URL `?output=csv&sheet=NomOnglet`

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
- Collection : `progress`, doc : `user.uid` (UID Firebase Auth — identique sur tous les appareils)
- **Auth : Google Sign-In** — `firebase.auth().signInWithPopup(GoogleAuthProvider)` + `onAuthStateChanged`
- SDK chargés : `firebase-app-compat.js`, `firebase-auth-compat.js`, `firebase-firestore-compat.js`
- Domaine autorisé : `ngfalo.github.io` (Firebase Console → Authentication → Authorized domains)
- Google Sign-In activé : Firebase Console → Authentication → Sign-in method → Google

---

## 🎯 Fonctionnalités implémentées

### Onglet 📚 Glossaire
- Termes ML en 6 familles : `eval`, `model`, `classif`, `summary`, `libs`, `choose`
- Filtrage par famille + recherche texte
- Chaque carte : image mentale, seuils (good/warn/bad), tip
- Badge ⭐ "Maîtrisé" visible si 5 bonnes réponses Anki

### Onglet 🎯 Quiz — Système de sessions (style Duolingo)
- **Écran de sélection de mode** (`mode-select-view`) : grille de 7 modes
- **Modes** : Mixte (10q), Scénario (10q), Vrai/Faux (10q), Terme→Déf (10q), Déf→Terme (10q), ⚡ Speed Round (15q, 15s/q), 💀 Boss Battle (20q, 1 erreur = game over)
- **Session immersive** (`session-view`) : header avec bouton Quitter, barre de progression, compteur
- **Navigation lockée** : changer d'onglet pendant une session déclenche modal d'abandon
- **Sauvegarde auto** : `localStorage` key `ml_quiz_session`, TTL 30 min — reprise au rechargement
- **Écran de fin** (`end-view`) : score %, XP (+10/bonne, +20 si 100%, +10 si ≥80%), liste des erreurs, message motivant
- **Fonctions clés** : `showModeSelect()`, `launchSession(mode)`, `showSessionView()`, `endSession(bossLost)`, `saveSession()`, `resumeSession()`, `confirmAbandon()`, `doAbandon()`
- **Compteurs session** : `sCorrect`, `sWrong`, `sessionMistakes[]` (séparés des compteurs globaux)
- Explication affichée après chaque réponse via `showFB()`

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
- **Sync cross-device** : fonctionne via Google Sign-In (même UID sur tous les appareils)

---

## 📊 Structure d'un terme (Google Sheets → TERMS array)

Le JS reconstruit automatiquement cette structure depuis les 4 onglets Sheets :

```js
{
  id: 'r2',                    // identifiant unique (onglet Terms)
  name: 'R²',                  // nom affiché
  full: 'Coefficient de Détermination',
  family: 'eval',              // eval | model | classif | summary | libs | choose
  image: '🎯 "..." ...',       // image mentale
  tip: 'Astuce...',
  d2t: 'Définition...',        // question mode Def→Terme
  t2d: ['bonne réponse', 'mauvais1', 'mauvais2', 'mauvais3'],  // index 0 = correct
  rules: [                     // onglet Rules
    {val:'≥ 0.90', label:'Excellent ✅', type:'good'}  // good | warn | bad
  ],
  scenarios: [                 // onglet Scenarios
    {q:'Question?', choice:'bonne réponse', wrongs:['a','b','c'], answer:'explication'}
  ],
  tf: [                        // onglet TrueFalse
    {stmt:'Affirmation...', answer:true, explain:'Explication...'}
  ]
}
```

---

## 🔜 Prochaine fonctionnalité — Dictionnaire / Tooltips

- Nouveau onglet Google Sheets : `Dictionary` — colonnes `word`, `definition`
- Les mots ML utiles dans l'app sont **soulignés** (ex: ROC, AUC, feature, Modèle, régularisation…)
- Cliquer sur un mot souligné → popup tooltip avec définition brève
- Claude juge quels mots sont "utiles ML" (pas les mots basiques ou UI)
- À implémenter : charger `Dictionary` depuis Sheets, scanner le texte affiché, wrapper les mots dans `<span class="dict-word" data-word="ROC">ROC</span>`, tooltip CSS/JS

---

## 🐛 Bugs corrigés (important à ne pas réintroduire)

**Bug 1** : Dans `buildQ()`, le code utilisait `qs.push({type:'sc', term:t, s})` (raccourci JS qui créait la propriété `s` au lieu de `sc`), ce qui faisait que `renderQ()` ne pouvait pas lire `q.sc.q` → aucun choix affiché en mode Scénario.
**Fix** : `qs.push({type:'sc', term:t, sc:sc})` — toujours écrire explicitement `sc:sc`.

**Bug 2** : L'ancien `getUserId()` générait un ID aléatoire stocké en `localStorage` → chaque appareil avait son propre doc Firebase → zéro sync cross-device.
**Fix** : Google Sign-In — l'UID Firebase est identique sur tous les appareils.

---

## 📝 Décisions de design prises

- **Single-file HTML** : tout dans un seul fichier, pas de framework
- **Firebase Compat** : choisi car ESM casse les `onclick=""` inline
- **Google Sheets** : source des données éditable sans toucher au HTML
- **GitHub Pages** : hébergement gratuit, URL simple
- **Pas de VPS** : Falo n'a pas les compétences pour le gérer
- **Style** : colorful mais clean, family-pills colorées par catégorie
- **Gamification** : streak + "le moins d'erreurs possible" (pas de points complexes)

---

## 🚀 Pour déployer une nouvelle version sur GitHub

```bash
git add ml_memos.html
git commit -m "description du changement"
git push
```

Ou via GitHub web : `github.com/ngfalo/ML-Course` → `ml_memos.html` → ✏️ → coller → Commit

---

## 💬 Comment reprendre avec Claude

Dis simplement :
> *"Lis CONTEXTE_PROJET.md et reprends le projet. Je veux [ajouter X / corriger Y / etc.]"*

Pour modifier du **contenu** (termes, quiz, phrases) :
> *"Lis le Google Sheet [URL] et [modifie / ajoute / vérifie] ..."*

Claude lira ce fichier + le fichier HTML et sera immédiatement opérationnel.
