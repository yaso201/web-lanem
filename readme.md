# Projet Quiz Game - Questions pour un Champion

## Vue d'ensemble du projet

Un jeu de quiz interactif inspiré de "Questions pour un Champion" avec plusieurs modes de jeu, un système de score, et une interface moderne et responsive.

## Objectifs pédagogiques

- **HTML** : Structure sémantique, formulaires, navigation
- **CSS** : Mise en page moderne, animations, responsive design
- **JavaScript** : Manipulation DOM, gestion d'événements, logique de jeu, gestion de données

## Architecture du projet

```
quiz-game/
├── index.html
├── css/
│   ├── styles.css
│   └── animations.css
├── js/
│   ├── main.js
│   ├── game.js
│   ├── questions.js
│   └── utils.js
├── data/
│   └── questions.json
├── assets/
│   ├── images/
│   └── sounds/
└── README.md
```

## Progression incrémentale

### Phase 1 : Structure de base (HTML + CSS de base)
**Durée estimée : 3-4 heures**

#### Objectifs
- Créer la structure HTML sémantique
- Implémenter le CSS de base
- Comprendre le responsive design

#### Fonctionnalités
- Page d'accueil avec titre et bouton "Jouer"
- Écran de sélection du mode de jeu
- Interface de jeu basique (question, réponses, score)
- Écran de fin de partie

#### Compétences développées
- Structure HTML5 sémantique (`<header>`, `<main>`, `<section>`, `<article>`)
- CSS Flexbox et Grid
- Media queries pour le responsive
- Sélecteurs CSS avancés

---

### Phase 2 : Interactivité de base (JavaScript fondamental)
**Durée estimée : 4-5 heures**

#### Objectifs
- Ajouter l'interactivité avec JavaScript
- Gérer les événements utilisateur
- Manipuler le DOM

#### Fonctionnalités
- Navigation entre les écrans
- Affichage dynamique des questions
- Gestion des clics sur les réponses
- Calcul et affichage du score

#### Compétences développées
- Sélection et manipulation d'éléments DOM
- Gestion d'événements (`addEventListener`)
- Conditions et boucles en JavaScript
- Fonctions et portée des variables

---

### Phase 3 : Logique de jeu avancée
**Durée estimée : 5-6 heures**

#### Objectifs
- Implémenter différents modes de jeu
- Ajouter un système de timer
- Gérer les bonus et malus

#### Fonctionnalités

##### Mode 1 : Quiz Classique
- 20 questions à choix multiples
- 30 secondes par question
- +10 points bonne réponse, -5 points mauvaise réponse

##### Mode 2 : Contre-la-montre
- Questions illimitées en 2 minutes
- +15 points bonne réponse
- Bonus de vitesse (+5 points si réponse en moins de 10 secondes)

##### Mode 3 : Survie
- Une seule vie
- Questions de difficulté croissante
- Score multiplié par le niveau atteint

#### Compétences développées
- `setInterval` et `setTimeout`
- Gestion d'états complexes
- Structures de données (objets, tableaux)
- Algorithmes de jeu

---

### Phase 4 : Interface utilisateur avancée (CSS avancé)
**Durée estimée : 4-5 heures**

#### Objectifs
- Créer une interface moderne et attrayante
- Ajouter des animations et transitions
- Améliorer l'expérience utilisateur

#### Fonctionnalités
- Animations CSS pour les transitions d'écran
- Effets visuels pour les bonnes/mauvaises réponses
- Barre de progression pour le timer
- Interface responsive optimisée
- Thème sombre/clair

#### Compétences développées
- Animations CSS (`@keyframes`, `transform`, `transition`)
- CSS Variables (custom properties)
- Pseudo-éléments et pseudo-classes
- Gradients et ombres

---

### Phase 5 : Persistance et données externes
**Durée estimée : 4-5 heures**

#### Objectifs
- Charger des questions depuis un fichier JSON
- Sauvegarder les scores localement
- Gérer les erreurs

#### Fonctionnalités
- Chargement de questions depuis `questions.json`
- Sauvegarde des meilleurs scores (localStorage)
- Tableau des scores
- Gestion des erreurs de chargement
- Mode hors-ligne

#### Compétences développées
- `fetch()` et Promises
- JSON parsing
- LocalStorage API
- Gestion d'erreurs (try/catch)
- Programmation asynchrone

---

### Phase 6 : Fonctionnalités avancées (Bonus)
**Durée estimée : 6-8 heures**

#### Objectifs
- Ajouter des fonctionnalités premium
- Optimiser les performances
- Améliorer l'accessibilité

#### Fonctionnalités avancées
- **Jokers** : 50/50, aide du public, saut de question
- **Questions avec images** : Support des médias
- **Mode multijoueur local** : 2 joueurs en alternance
- **Statistiques détaillées** : Graphiques de performance
- **Sons et effets** : Feedback audio
- **Accessibilité** : Support clavier, screen readers

#### Compétences développées
- Canvas API pour les graphiques
- Web Audio API
- Accessibility (ARIA, navigation clavier)
- Optimisation des performances
- Patterns de design (MVC, Observer)

## Structure des données

### Format des questions (Phase 1-4)
```javascript
const questions = [
  {
    id: 1,
    category: "Histoire",
    difficulty: 1, // 1-5
    question: "En quelle année a eu lieu la Révolution française ?",
    answers: ["1789", "1792", "1799", "1804"],
    correct: 0, // index de la bonne réponse
    explanation: "La Révolution française a commencé en 1789 avec la prise de la Bastille."
  }
];
```

### Format JSON (Phase 5+)
```json
{
  "categories": ["Histoire", "Sciences", "Sport", "Culture"],
  "questions": [
    {
      "id": 1,
      "category": "Histoire",
      "difficulty": 1,
      "question": "En quelle année a eu lieu la Révolution française ?",
      "answers": ["1789", "1792", "1799", "1804"],
      "correct": 0,
      "explanation": "La Révolution française a commencé en 1789.",
      "media": null
    }
  ]
}
```

## Critères d'évaluation par phase

### Phase 1-2 : Fondamentaux
- ✅ Structure HTML valide et sémantique
- ✅ CSS responsive fonctionnel
- ✅ JavaScript de base opérationnel
- ✅ Navigation entre écrans

### Phase 3-4 : Fonctionnalités
- ✅ Modes de jeu implémentés
- ✅ Timer fonctionnel
- ✅ Animations fluides
- ✅ Interface moderne

### Phase 5-6 : Avancé
- ✅ Chargement de données externes
- ✅ Persistance des scores
- ✅ Gestion d'erreurs robuste
- ✅ Fonctionnalités bonus

## Ressources recommandées

### Documentation
- [MDN Web Docs](https://developer.mozilla.org/)
- [CSS Tricks](https://css-tricks.com/)
- [JavaScript.info](https://javascript.info/)

### Outils de développement
- VS Code avec extensions (Live Server, Prettier)
- Chrome DevTools
- Git pour le versioning

### Inspiration design
- [Dribbble](https://dribbble.com/search/quiz-app)
- [CodePen](https://codepen.io/search/pens?q=quiz)

## Conseils de développement

1. **Commencez simple** : Implémentez une fonctionnalité à la fois
2. **Testez fréquemment** : Vérifiez chaque ajout dans le navigateur
3. **Commentez votre code** : Facilitez la maintenance
4. **Utilisez Git** : Versionnez chaque phase
5. **Responsive first** : Testez sur différentes tailles d'écran
6. **Optimisez progressivement** : Performances et accessibilité

Ce projet vous donnera une base solide en développement web front-end tout en créant quelque chose d'amusant et d'interactif !
