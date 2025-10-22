# TP Phase 1 - Quiz Game : Structure et Design de Base

## Objectifs pédagogiques

À la fin de ce TP, vous serez capable de :
- ✅ Structurer une page web avec HTML5 sémantique
- ✅ Créer une mise en page responsive avec CSS Flexbox
- ✅ Appliquer des styles modernes et cohérents
- ✅ Organiser votre code de manière professionnelle

**Prérequis :** Bases HTML/CSS

---

## Étape 1 : Préparation de l'environnement

### 1.1 Création de l'arborescence

Créez la structure de fichiers suivante :

```
quiz-game/
├── index.html
├── css/
│   └── styles.css
└── README.md
```

** Conseil :** Utilisez votre éditeur de code pour créer tous les dossiers d'un coup.

### 1.2 Configuration de base

Dans le fichier `README.md`, ajoutez :

```markdown
# Quiz Game - Questions pour un Champion

## Phase 1 : Structure de base
- [x] Arborescence créée
- [ ] HTML structure
- [ ] CSS de base
- [ ] Design responsive
```

** Objectif pédagogique :** Apprendre l'importance de l'organisation et de la documentation.

---

## Étape 2 : Structure HTML sémantique

### 2.1 Le squelette HTML5

Créez dans `index.html` la structure de base :

```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Quiz Game - Questions pour un Champion</title>
    <link rel="stylesheet" href="css/styles.css">
</head>
<body>
    <!-- Votre contenu ira ici -->
</body>
</html>
```

**❓ Question de réflexion :** Pourquoi utilise-t-on `lang="fr"` et `viewport` ?

### 2.2 Structure des écrans

Ajoutez dans le `<body>` la structure suivante :

```html
<!-- Écran d'accueil -->
<main id="home-screen" class="screen active">
    <header class="game-header">
        <h1 class="game-title">Questions pour un Champion</h1>
        <p class="game-subtitle">Testez vos connaissances !</p>
    </header>
    
    <section class="menu-section">
        <button class="btn btn-primary">Commencer le jeu</button>
        <button class="btn btn-secondary">Tableau des scores</button>
    </section>
</main>

<!-- Écran de sélection du mode -->
<main id="mode-screen" class="screen">
    <header class="screen-header">
        <h2>Choisissez votre mode de jeu</h2>
        <button class="btn btn-back">← Retour</button>
    </header>
    
    <section class="modes-grid">
        <article class="mode-card">
            <h3>Quiz Classique</h3>
            <p>20 questions, 30 secondes par question</p>
            <button class="btn btn-mode" data-mode="classic">Jouer</button>
        </article>
        
        <article class="mode-card">
            <h3>Contre-la-montre</h3>
            <p>2 minutes pour un maximum de questions</p>
            <button class="btn btn-mode" data-mode="speed">Jouer</button>
        </article>
        
        <article class="mode-card">
            <h3>Survie</h3>
            <p>Une seule erreur autorisée</p>
            <button class="btn btn-mode" data-mode="survival">Jouer</button>
        </article>
    </section>
</main>

<!-- Écran de jeu -->
<main id="game-screen" class="screen">
    <header class="game-info">
        <div class="score-display">
            <span class="label">Score :</span>
            <span class="value" id="score-value">0</span>
        </div>
        <div class="timer-display">
            <span class="label">Temps :</span>
            <span class="value" id="timer-value">30</span>
        </div>
        <div class="question-counter">
            <span class="label">Question :</span>
            <span class="value" id="question-counter">1/20</span>
        </div>
    </header>
    
    <section class="question-section">
        <h2 id="question-text" class="question-text">
            Votre question apparaîtra ici
        </h2>
        
        <div class="answers-grid">
            <button class="answer-btn" data-answer="0">Réponse A</button>
            <button class="answer-btn" data-answer="1">Réponse B</button>
            <button class="answer-btn" data-answer="2">Réponse C</button>
            <button class="answer-btn" data-answer="3">Réponse D</button>
        </div>
    </section>
</main>

<!-- Écran de fin -->
<main id="end-screen" class="screen">
    <header class="screen-header">
        <h2>Partie terminée !</h2>
    </header>
    
    <section class="results-section">
        <div class="final-score">
            <span class="score-label">Score final :</span>
            <span class="score-number" id="final-score">0</span>
        </div>
        
        <div class="stats">
            <div class="stat-item">
                <span class="stat-label">Bonnes réponses :</span>
                <span class="stat-value" id="correct-answers">0</span>
            </div>
            <div class="stat-item">
                <span class="stat-label">Taux de réussite :</span>
                <span class="stat-value" id="success-rate">0%</span>
            </div>
        </div>
        
        <div class="end-actions">
            <button class="btn btn-primary">Rejouer</button>
            <button class="btn btn-secondary">Menu principal</button>
        </div>
    </section>
</main>
```

** À analyser :**
1. Identifiez les balises sémantiques utilisées (`main`, `header`, `section`, `article`)
2. Observez l'usage des classes CSS (préfixage, cohérence)
3. Notez les attributs `data-*` - à quoi servent-ils ?

---

## Étape 3 : CSS de base et reset (30 min)

### 3.1 Reset et variables CSS

Commencez votre fichier `css/styles.css` :

```css
/* ===== RESET ET VARIABLES ===== */

/* Reset de base */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

/* Variables CSS (Custom Properties) */
:root {
    /* Couleurs principales */
    --primary-color: #3498db;
    --primary-dark: #2980b9;
    --secondary-color: #e74c3c;
    --success-color: #27ae60;
    --warning-color: #f39c12;
    
    /* Couleurs neutres */
    --dark-bg: #2c3e50;
    --light-bg: #ecf0f1;
    --text-dark: #2c3e50;
    --text-light: #ffffff;
    --border-color: #bdc3c7;
    
    /* Espacements */
    --spacing-xs: 0.5rem;
    --spacing-sm: 1rem;
    --spacing-md: 1.5rem;
    --spacing-lg: 2rem;
    --spacing-xl: 3rem;
    
    /* Typographie */
    --font-primary: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    --font-size-sm: 0.875rem;
    --font-size-base: 1rem;
    --font-size-lg: 1.125rem;
    --font-size-xl: 1.5rem;
    --font-size-xxl: 2rem;
    
    /* Autres */
    --border-radius: 8px;
    --box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
    --transition: all 0.3s ease;
}

/* Styles de base */
body {
    font-family: var(--font-primary);
    line-height: 1.6;
    color: var(--text-dark);
    background: linear-gradient(135deg, var(--primary-color), var(--primary-dark));
    min-height: 100vh;
}
```

** Objectif pédagogique :** Comprendre l'importance du reset CSS et l'utilisation des variables CSS pour la maintenance.

### 3.2 Système de mise en page

Ajoutez les styles pour la gestion des écrans :

```css
/* ===== SYSTÈME D'ÉCRANS ===== */

.screen {
    display: none; /* Tous les écrans sont cachés par défaut */
    min-height: 100vh;
    padding: var(--spacing-lg);
    max-width: 1200px;
    margin: 0 auto;
}

.screen.active {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
}

/* Responsive : ajustement sur mobile */
@media (max-width: 768px) {
    .screen {
        padding: var(--spacing-md);
    }
}
```

** Question :** Pourquoi utilise-t-on `display: flex` plutôt que `display: block` ?

---

## Étape 4 : Styles des composants

### 4.1 Typographie et en-têtes

```css
/* ===== TYPOGRAPHIE ===== */

.game-title {
    font-size: var(--font-size-xxl);
    color: var(--text-light);
    text-align: center;
    margin-bottom: var(--spacing-sm);
    text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
}

.game-subtitle {
    font-size: var(--font-size-lg);
    color: var(--light-bg);
    text-align: center;
    margin-bottom: var(--spacing-xl);
}

.screen-header {
    text-align: center;
    margin-bottom: var(--spacing-xl);
    width: 100%;
}

.screen-header h2 {
    font-size: var(--font-size-xl);
    color: var(--text-light);
    margin-bottom: var(--spacing-md);
}
```

### 4.2 Système de boutons

```css
/* ===== BOUTONS ===== */

.btn {
    border: none;
    padding: var(--spacing-sm) var(--spacing-lg);
    border-radius: var(--border-radius);
    font-size: var(--font-size-base);
    font-weight: 600;
    cursor: pointer;
    transition: var(--transition);
    text-decoration: none;
    display: inline-block;
    text-align: center;
    min-width: 120px;
}

.btn:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
}

.btn:active {
    transform: translateY(0);
}

/* Variantes de boutons */
.btn-primary {
    background: var(--success-color);
    color: var(--text-light);
}

.btn-primary:hover {
    background: #229954;
}

.btn-secondary {
    background: var(--light-bg);
    color: var(--text-dark);
}

.btn-secondary:hover {
    background: #d5dbdb;
}

.btn-back {
    background: var(--secondary-color);
    color: var(--text-light);
    padding: var(--spacing-xs) var(--spacing-md);
    min-width: auto;
}

.btn-mode {
    width: 100%;
    background: var(--primary-color);
    color: var(--text-light);
    margin-top: var(--spacing-md);
}
```

** Exercice pratique :** Ajoutez une variante `.btn-disabled` avec les styles appropriés.

### 4.3 Layout des modes de jeu

```css
/* ===== SÉLECTION DES MODES ===== */

.modes-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
    gap: var(--spacing-lg);
    width: 100%;
    max-width: 900px;
}

.mode-card {
    background: var(--text-light);
    padding: var(--spacing-lg);
    border-radius: var(--border-radius);
    box-shadow: var(--box-shadow);
    text-align: center;
    transition: var(--transition);
}

.mode-card:hover {
    transform: translateY(-5px);
    box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15);
}

.mode-card h3 {
    color: var(--primary-color);
    margin-bottom: var(--spacing-sm);
    font-size: var(--font-size-lg);
}

.mode-card p {
    color: var(--text-dark);
    margin-bottom: var(--spacing-md);
    opacity: 0.8;
}
```

** À observer :** Comment CSS Grid s'adapte automatiquement au contenu avec `auto-fit` et `minmax`.

---

## Étape 5 : Interface de jeu

### 5.1 En-tête du jeu avec informations

```css
/* ===== INTERFACE DE JEU ===== */

.game-info {
    display: flex;
    justify-content: space-between;
    align-items: center;
    background: rgba(255, 255, 255, 0.1);
    padding: var(--spacing-md);
    border-radius: var(--border-radius);
    margin-bottom: var(--spacing-xl);
    backdrop-filter: blur(10px);
    width: 100%;
    max-width: 800px;
}

.score-display,
.timer-display,
.question-counter {
    text-align: center;
    color: var(--text-light);
}

.score-display .label,
.timer-display .label,
.question-counter .label {
    display: block;
    font-size: var(--font-size-sm);
    opacity: 0.8;
    margin-bottom: var(--spacing-xs);
}

.score-display .value,
.timer-display .value,
.question-counter .value {
    display: block;
    font-size: var(--font-size-xl);
    font-weight: bold;
}

/* Responsive pour l'en-tête */
@media (max-width: 768px) {
    .game-info {
        flex-direction: column;
        gap: var(--spacing-sm);
    }
    
    .score-display,
    .timer-display,
    .question-counter {
        flex-direction: row;
        gap: var(--spacing-sm);
    }
    
    .score-display .label,
    .timer-display .label,
    .question-counter .label {
        margin-bottom: 0;
    }
}
```

### 5.2 Section des questions

```css
.question-section {
    width: 100%;
    max-width: 800px;
    text-align: center;
}

.question-text {
    background: var(--text-light);
    padding: var(--spacing-xl);
    border-radius: var(--border-radius);
    margin-bottom: var(--spacing-xl);
    box-shadow: var(--box-shadow);
    font-size: var(--font-size-lg);
    line-height: 1.8;
    color: var(--text-dark);
}

.answers-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: var(--spacing-md);
}

.answer-btn {
    padding: var(--spacing-lg);
    background: var(--text-light);
    border: 3px solid transparent;
    border-radius: var(--border-radius);
    font-size: var(--font-size-base);
    cursor: pointer;
    transition: var(--transition);
    text-align: left;
    min-height: 80px;
    display: flex;
    align-items: center;
    justify-content: center;
}

.answer-btn:hover {
    border-color: var(--primary-color);
    background: #f8f9fa;
    transform: scale(1.02);
}

/* Responsive pour les réponses */
@media (max-width: 768px) {
    .answers-grid {
        grid-template-columns: 1fr;
    }
}
```

---

## Étape 6 : Écran de fin et responsive

### 6.1 Styles pour l'écran de résultats

```css
/* ===== ÉCRAN DE FIN ===== */

.results-section {
    background: var(--text-light);
    padding: var(--spacing-xl);
    border-radius: var(--border-radius);
    box-shadow: var(--box-shadow);
    text-align: center;
    max-width: 500px;
    width: 100%;
}

.final-score {
    margin-bottom: var(--spacing-xl);
}

.score-label {
    display: block;
    font-size: var(--font-size-lg);
    color: var(--text-dark);
    margin-bottom: var(--spacing-sm);
}

.score-number {
    display: block;
    font-size: 3rem;
    font-weight: bold;
    color: var(--primary-color);
}

.stats {
    margin-bottom: var(--spacing-xl);
    padding: var(--spacing-lg);
    background: var(--light-bg);
    border-radius: var(--border-radius);
}

.stat-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: var(--spacing-sm);
}

.stat-item:last-child {
    margin-bottom: 0;
}

.stat-label {
    font-weight: 600;
    color: var(--text-dark);
}

.stat-value {
    font-weight: bold;
    color: var(--primary-color);
}

.end-actions {
    display: flex;
    gap: var(--spacing-md);
    justify-content: center;
}

@media (max-width: 768px) {
    .end-actions {
        flex-direction: column;
    }
}
```

### 6.2 Finalisation responsive

```css
/* ===== RESPONSIVE GLOBAL ===== */

/* Tablettes */
@media (max-width: 1024px) {
    .game-title {
        font-size: 2.5rem;
    }
    
    .modes-grid {
        grid-template-columns: 1fr 1fr;
    }
}

/* Mobiles */
@media (max-width: 768px) {
    :root {
        --font-size-xxl: 1.8rem;
        --font-size-xl: 1.3rem;
        --spacing-xl: 2rem;
    }
    
    .modes-grid {
        grid-template-columns: 1fr;
    }
    
    .question-text {
        padding: var(--spacing-lg);
        font-size: var(--font-size-base);
    }
}

/* Très petits écrans */
@media (max-width: 480px) {
    .screen {
        padding: var(--spacing-sm);
    }
    
    .btn {
        width: 100%;
        margin-bottom: var(--spacing-sm);
    }
}
```

---

## Étape 7 : Test et validation

### 7.1 Checklist de validation

Testez votre travail :

1. **Structure HTML :**
   - [ ] Le HTML passe la validation W3C
   - [ ] Toutes les balises sont bien fermées
   - [ ] Les attributs `id` sont uniques

2. **CSS :**
   - [ ] Pas d'erreurs dans la console
   - [ ] Les variables CSS sont bien utilisées
   - [ ] Le design est cohérent

3. **Responsive :**
   - [ ] Test sur desktop (1920px)
   - [ ] Test sur tablette (768px)
   - [ ] Test sur mobile (320px)

### 7.2 Amélioration optionnelle

Ajoutez ces améliorations si vous avez fini en avance :

```css
/* Animation d'entrée pour les écrans */
.screen.active {
    animation: fadeInUp 0.5s ease;
}

@keyframes fadeInUp {
    from {
        opacity: 0;
        transform: translateY(30px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

/* Hover sophistiqué pour les cartes */
.mode-card {
    position: relative;
    overflow: hidden;
}

.mode-card::before {
    content: '';
    position: absolute;
    top: 0;
    left: -100%;
    width: 100%;
    height: 100%;
    background: linear-gradient(90deg, transparent, rgba(52, 152, 219, 0.1), transparent);
    transition: left 0.5s;
}

.mode-card:hover::before {
    left: 100%;
}
```

---

## Ressources complémentaires

- [Guide Flexbox MDN](https://developer.mozilla.org/fr/docs/Web/CSS/CSS_Flexible_Box_Layout)
- [Guide CSS Grid MDN](https://developer.mozilla.org/fr/docs/Web/CSS/CSS_Grid_Layout)
- [Validator W3C](https://validator.w3.org/)
- [Can I Use](https://caniuse.com/) pour la compatibilité

**Prochaine étape :** Phase 2 - Ajout de l'interactivité JavaScript !
