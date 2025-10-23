# TP Phase 2 - Quiz Game : Interactivité JavaScript

## Objectifs pédagogiques

À la fin de ce TP, vous serez capable de :
- ✅ Manipuler le DOM avec JavaScript moderne
- ✅ Gérer les événements utilisateur de manière efficace
- ✅ Structurer du code JavaScript avec des fonctions
- ✅ Implémenter la logique métier d'un jeu simple
- ✅ Débugger votre code avec les outils de développement

**Durée estimée :** 4-5 heures  
**Prérequis :** TP Phase 1 terminé, bases JavaScript

---

## Rappel et préparation

### Vérification des prérequis

Assurez-vous que votre Phase 1 est fonctionnelle :
- ✅ Structure HTML complète
- ✅ CSS responsive opérationnel
- ✅ Tous les écrans visibles en ajoutant/retirant la classe `active`

### Extension de l'arborescence

Ajoutez ces fichiers à votre projet :

```
quiz-game/
├── index.html (existant)
├── css/
│   └── styles.css (existant)
├── js/
│   ├── main.js (nouveau)
│   ├── game.js (nouveau)
│   └── questions.js (nouveau)
└── README.md (à mettre à jour)
```

Mettez à jour votre `index.html` en ajoutant avant la fermeture de `</body>` :

```html
    <!-- Scripts JavaScript -->
    <script src="js/questions.js"></script>
    <script src="js/game.js"></script>
    <script src="js/main.js"></script>
</body>
</html>
```

**❓ Question :** Pourquoi l'ordre d'inclusion des scripts est-il important ?

---

## Étape 1 : Données et structure

### 1.1 Base de données des questions

Créez le fichier `js/questions.js` :

```javascript
// ===== BASE DE DONNÉES DES QUESTIONS =====

const QUESTIONS_DATABASE = [
    {
        id: 1,
        category: "Histoire",
        difficulty: 1,
        question: "En quelle année a eu lieu la Révolution française ?",
        answers: ["1789", "1792", "1799", "1804"],
        correct: 0,
        explanation: "La Révolution française a commencé en 1789 avec la prise de la Bastille."
    },
    {
        id: 2,
        category: "Sciences",
        difficulty: 1,
        question: "Quelle est la formule chimique de l'eau ?",
        answers: ["H2O", "CO2", "NaCl", "O2"],
        correct: 0,
        explanation: "L'eau est composée de deux atomes d'hydrogène et un atome d'oxygène : H2O."
    },
    {
        id: 3,
        category: "Géographie",
        difficulty: 2,
        question: "Quelle est la capitale de l'Australie ?",
        answers: ["Sydney", "Melbourne", "Canberra", "Perth"],
        correct: 2,
        explanation: "Canberra est la capitale de l'Australie, bien que Sydney soit la ville la plus connue."
    },
    {
        id: 4,
        category: "Sport",
        difficulty: 1,
        question: "Combien de joueurs composent une équipe de football sur le terrain ?",
        answers: ["10", "11", "12", "9"],
        correct: 1,
        explanation: "Une équipe de football est composée de 11 joueurs sur le terrain."
    },
    {
        id: 5,
        category: "Culture",
        difficulty: 2,
        question: "Qui a peint 'La Joconde' ?",
        answers: ["Pablo Picasso", "Vincent van Gogh", "Leonardo da Vinci", "Claude Monet"],
        correct: 2,
        explanation: "La Joconde a été peinte par Leonardo da Vinci entre 1503 et 1519."
    },
    {
        id: 6,
        category: "Sciences",
        difficulty: 3,
        question: "Quelle est la vitesse de la lumière dans le vide ?",
        answers: ["300 000 km/s", "150 000 km/s", "450 000 km/s", "299 792 458 m/s"],
        correct: 3,
        explanation: "La vitesse de la lumière dans le vide est exactement 299 792 458 mètres par seconde."
    },
    {
        id: 7,
        category: "Histoire",
        difficulty: 2,
        question: "Quel empereur français a été vaincu à Waterloo ?",
        answers: ["Napoléon Ier", "Napoléon III", "Louis XIV", "Charlemagne"],
        correct: 0,
        explanation: "Napoléon Ier a été définitivement vaincu à la bataille de Waterloo en 1815."
    },
    {
        id: 8,
        category: "Géographie",
        difficulty: 1,
        question: "Quel est le plus long fleuve du monde ?",
        answers: ["Amazon", "Nil", "Yangtsé", "Mississippi"],
        correct: 1,
        explanation: "Le Nil, avec ses 6 650 km, est généralement considéré comme le plus long fleuve du monde."
    }
];

// Fonction utilitaire pour mélanger un tableau
function shuffleArray(array) {
    const shuffled = [...array]; // Créer une copie
    for (let i = shuffled.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
    }
    return shuffled;
}

// Fonction pour obtenir des questions aléatoirement
function getRandomQuestions(count = 10, difficulty = null) {
    let filteredQuestions = QUESTIONS_DATABASE;
    
    // Filtrer par difficulté si spécifiée
    if (difficulty) {
        filteredQuestions = QUESTIONS_DATABASE.filter(q => q.difficulty === difficulty);
    }
    
    // Mélanger et prendre le nombre demandé
    const shuffled = shuffleArray(filteredQuestions);
    return shuffled.slice(0, count);
}
```

** Objectif pédagogique :** Comprendre la structure des données et les fonctions utilitaires.

**💡 Exercice :** Ajoutez 5 questions de votre choix en respectant la structure.

### 1.2 Structure du gestionnaire de jeu

Créez le fichier `js/game.js` :

```javascript
// ===== GESTIONNAIRE DE JEU =====

// Objet principal qui gère l'état du jeu
const GameManager = {
    // État du jeu
    currentScreen: 'home',
    gameMode: null,
    questions: [],
    currentQuestionIndex: 0,
    score: 0,
    correctAnswers: 0,
    timeRemaining: 30,
    timerInterval: null,
    gameActive: false,

    // Configuration des modes de jeu
    modes: {
        classic: {
            name: "Quiz Classique",
            questionCount: 8, // Réduit pour le TP
            timePerQuestion: 30,
            pointsCorrect: 10,
            pointsIncorrect: -5
        },
        speed: {
            name: "Contre-la-montre",
            questionCount: 999, // Illimité
            totalTime: 120, // 2 minutes
            pointsCorrect: 15,
            pointsIncorrect: 0,
            bonusSpeed: 5 // Bonus si réponse rapide
        },
        survival: {
            name: "Survie",
            questionCount: 20,
            timePerQuestion: 20,
            pointsCorrect: 20,
            lives: 1 // Une seule erreur autorisée
        }
    },

    // Méthodes du gestionnaire de jeu
    init() {
        console.log("Initialisation du gestionnaire de jeu");
        this.reset();
    },

    reset() {
        this.currentQuestionIndex = 0;
        this.score = 0;
        this.correctAnswers = 0;
        this.timeRemaining = 30;
        this.gameActive = false;
        this.clearTimer();
        console.log(" Jeu remis à zéro");
    },

    startGame(mode) {
        console.log(`Démarrage du jeu en mode : ${mode}`);
        this.gameMode = mode;
        this.reset();
        
        // Charger les questions selon le mode
        const modeConfig = this.modes[mode];
        this.questions = getRandomQuestions(modeConfig.questionCount);
        
        if (this.questions.length === 0) {
            console.error("Aucune question disponible !");
            return false;
        }

        this.gameActive = true;
        this.loadQuestion();
        return true;
    },

    loadQuestion() {
        if (this.currentQuestionIndex >= this.questions.length) {
            this.endGame();
            return;
        }

        const question = this.questions[this.currentQuestionIndex];
        console.log(`Chargement question ${this.currentQuestionIndex + 1}: ${question.question}`);
        
        // Mettre à jour l'affichage
        this.updateQuestionDisplay(question);
        this.updateGameInfo();
        
        // Démarrer le timer selon le mode
        this.startTimer();
    },

    updateQuestionDisplay(question) {
        // Cette méthode sera complétée dans l'étape suivante
        console.log("Mise à jour de l'affichage de la question");
    },

    updateGameInfo() {
        // Cette méthode sera complétée dans l'étape suivante
        console.log("Mise à jour des informations de jeu");
    },

    startTimer() {
        this.clearTimer(); // Nettoyer le timer précédent
        
        const modeConfig = this.modes[this.gameMode];
        this.timeRemaining = modeConfig.timePerQuestion || 30;
        
        this.timerInterval = setInterval(() => {
            this.timeRemaining--;
            console.log(`Temps restant: ${this.timeRemaining}s`);
            
            if (this.timeRemaining <= 0) {
                this.handleTimeout();
            }
        }, 1000);
    },

    clearTimer() {
        if (this.timerInterval) {
            clearInterval(this.timerInterval);
            this.timerInterval = null;
        }
    },

    handleAnswer(answerIndex) {
        if (!this.gameActive) return;

        const question = this.questions[this.currentQuestionIndex];
        const isCorrect = answerIndex === question.correct;
        
        console.log(`💭 Réponse ${answerIndex}: ${isCorrect ? 'Correcte ✅' : 'Incorrecte ❌'}`);
        
        this.clearTimer();
        this.processAnswer(isCorrect);
        
        // Passer à la question suivante après un délai
        setTimeout(() => {
            this.nextQuestion();
        }, 2000);
    },

    processAnswer(isCorrect) {
        const modeConfig = this.modes[this.gameMode];
        
        if (isCorrect) {
            this.correctAnswers++;
            this.score += modeConfig.pointsCorrect;
            
            // Bonus de vitesse pour le mode speed
            if (this.gameMode === 'speed' && this.timeRemaining > 20) {
                this.score += modeConfig.bonusSpeed;
                console.log("🚀 Bonus vitesse !");
            }
        } else {
            // Gestion des points négatifs
            if (modeConfig.pointsIncorrect) {
                this.score += modeConfig.pointsIncorrect; // Déjà négatif
            }
            
            // Mode survie : fin de partie sur erreur
            if (this.gameMode === 'survival') {
                console.log("💀 Mode survie : fin de partie !");
                this.endGame();
                return;
            }
        }
        
        console.log(`📈 Score actuel: ${this.score}`);
    },

    handleTimeout() {
        console.log("Temps écoulé !");
        this.clearTimer();
        this.processAnswer(false); // Traiter comme une mauvaise réponse
        
        setTimeout(() => {
            this.nextQuestion();
        }, 2000);
    },

    nextQuestion() {
        this.currentQuestionIndex++;
        
        if (this.currentQuestionIndex < this.questions.length) {
            this.loadQuestion();
        } else {
            this.endGame();
        }
    },

    endGame() {
        console.log("Fin de partie !");
        this.gameActive = false;
        this.clearTimer();
        
        // Calculer les statistiques
        const totalQuestions = this.currentQuestionIndex;
        const successRate = totalQuestions > 0 ? Math.round((this.correctAnswers / totalQuestions) * 100) : 0;
        
        console.log(`Statistiques finales:
        - Score: ${this.score}
        - Bonnes réponses: ${this.correctAnswers}/${totalQuestions}
        - Taux de réussite: ${successRate}%`);
        
        // Afficher l'écran de fin
        this.showResults({
            score: this.score,
            correctAnswers: this.correctAnswers,
            totalQuestions: totalQuestions,
            successRate: successRate
        });
    },

    showResults(results) {
        // Cette méthode sera complétée dans l'étape suivante
        console.log("Affichage des résultats");
    }
};
```

**À analyser :**
1. Comment l'objet `GameManager` organise-t-il les données et les méthodes ?
2. Pourquoi utilise-t-on `console.log` à ce stade ?
3. Comment le timer fonctionne-t-il avec `setInterval` ?

---

## Étape 2 : Navigation et gestion des écrans

### 2.1 Gestionnaire de navigation

Créez le fichier `js/main.js` :

```javascript
// ===== GESTIONNAIRE PRINCIPAL ET NAVIGATION =====

// Objet pour gérer la navigation entre les écrans
const ScreenManager = {
    currentScreen: 'home',
    
    // Références aux éléments DOM (seront initialisées plus tard)
    screens: {},
    elements: {},
    
    init() {
        console.log("Initialisation du gestionnaire d'écrans");
        this.cacheElements();
        this.bindEvents();
        this.showScreen('home');
    },

    // Mettre en cache les références aux éléments DOM
    cacheElements() {
        console.log("Mise en cache des éléments DOM");
        
        // Écrans
        this.screens = {
            home: document.getElementById('home-screen'),
            mode: document.getElementById('mode-screen'),
            game: document.getElementById('game-screen'),
            end: document.getElementById('end-screen')
        };

        // Éléments de l'interface de jeu
        this.elements = {
            // Boutons principaux
            startBtn: document.querySelector('#home-screen .btn-primary'),
            scoresBtn: document.querySelector('#home-screen .btn-secondary'),
            backBtn: document.querySelector('#mode-screen .btn-back'),
            
            // Boutons de mode
            modeButtons: document.querySelectorAll('.btn-mode'),
            
            // Interface de jeu
            scoreValue: document.getElementById('score-value'),
            timerValue: document.getElementById('timer-value'),
            questionCounter: document.getElementById('question-counter'),
            questionText: document.getElementById('question-text'),
            answerButtons: document.querySelectorAll('.answer-btn'),
            
            // Écran de fin
            finalScore: document.getElementById('final-score'),
            correctAnswers: document.getElementById('correct-answers'),
            successRate: document.getElementById('success-rate'),
            
            // Boutons de fin
            replayBtn: document.querySelector('#end-screen .btn-primary'),
            menuBtn: document.querySelector('#end-screen .btn-secondary')
        };

        // Vérifier que tous les éléments sont trouvés
        this.validateElements();
    },

    validateElements() {
        let missingElements = [];
        
        // Vérifier les écrans
        Object.entries(this.screens).forEach(([name, element]) => {
            if (!element) {
                missingElements.push(`Écran: ${name}`);
            }
        });

        // Vérifier les éléments essentiels
        const essentialElements = ['startBtn', 'questionText', 'scoreValue'];
        essentialElements.forEach(name => {
            if (!this.elements[name]) {
                missingElements.push(`Élément: ${name}`);
            }
        });

        if (missingElements.length > 0) {
            console.error("Éléments manquants:", missingElements);
        } else {
            console.log("Tous les éléments DOM trouvés");
        }
    },

    // Associer les événements aux éléments
    bindEvents() {
        console.log("🔗 Association des événements");

        // Bouton "Commencer le jeu"
        if (this.elements.startBtn) {
            this.elements.startBtn.addEventListener('click', () => {
                console.log("Clic sur 'Commencer le jeu'");
                this.showScreen('mode');
            });
        }

        // Bouton "Retour" de l'écran des modes
        if (this.elements.backBtn) {
            this.elements.backBtn.addEventListener('click', () => {
                console.log("Clic sur 'Retour'");
                this.showScreen('home');
            });
        }

        // Boutons de sélection de mode
        this.elements.modeButtons.forEach(button => {
            button.addEventListener('click', () => {
                const mode = button.dataset.mode;
                console.log(`Sélection du mode: ${mode}`);
                this.startGame(mode);
            });
        });

        // Boutons de réponse
        this.elements.answerButtons.forEach((button, index) => {
            button.addEventListener('click', () => {
                console.log(`Sélection de la réponse: ${index}`);
                this.handleAnswer(index);
            });
        });

        // Boutons de l'écran de fin
        if (this.elements.replayBtn) {
            this.elements.replayBtn.addEventListener('click', () => {
                console.log("Clic sur 'Rejouer'");
                this.showScreen('mode');
            });
        }

        if (this.elements.menuBtn) {
            this.elements.menuBtn.addEventListener('click', () => {
                console.log("Clic sur 'Menu principal'");
                this.showScreen('home');
            });
        }
    },

    // Afficher un écran spécifique
    showScreen(screenName) {
        console.log(`Changement d'écran: ${this.currentScreen} → ${screenName}`);
        
        // Masquer tous les écrans
        Object.values(this.screens).forEach(screen => {
            if (screen) {
                screen.classList.remove('active');
            }
        });

        // Afficher l'écran demandé
        if (this.screens[screenName]) {
            this.screens[screenName].classList.add('active');
            this.currentScreen = screenName;
        } else {
            console.error(`Écran inconnu: ${screenName}`);
        }
    },

    // Démarrer une partie
    startGame(mode) {
        console.log(`Démarrage du jeu en mode: ${mode}`);
        
        if (GameManager.startGame(mode)) {
            this.showScreen('game');
        } else {
            console.error("Impossible de démarrer le jeu");
            alert("Erreur lors du démarrage du jeu !");
        }
    },

    // Gérer une réponse
    handleAnswer(answerIndex) {
        // Désactiver temporairement les boutons pour éviter les clics multiples
        this.setAnswerButtonsEnabled(false);
        
        // Ajouter un feedback visuel
        this.showAnswerFeedback(answerIndex);
        
        // Déléguer au gestionnaire de jeu
        GameManager.handleAnswer(answerIndex);
    },

    setAnswerButtonsEnabled(enabled) {
        this.elements.answerButtons.forEach(button => {
            button.disabled = !enabled;
            if (enabled) {
                button.classList.remove('disabled');
            } else {
                button.classList.add('disabled');
            }
        });
    },

    showAnswerFeedback(selectedIndex) {
        const question = GameManager.questions[GameManager.currentQuestionIndex];
        const isCorrect = selectedIndex === question.correct;
        
        this.elements.answerButtons.forEach((button, index) => {
            button.classList.remove('correct', 'incorrect', 'selected');
            
            if (index === selectedIndex) {
                button.classList.add('selected');
                button.classList.add(isCorrect ? 'correct' : 'incorrect');
            }
            
            // Montrer la bonne réponse si l'utilisateur s'est trompé
            if (!isCorrect && index === question.correct) {
                button.classList.add('correct');
            }
        });
    }
};

// Initialisation quand le DOM est chargé
document.addEventListener('DOMContentLoaded', () => {
    console.log("DOM chargé, initialisation de l'application");
    ScreenManager.init();
    GameManager.init();
});
```

### 2.2 Ajout des styles pour le feedback

Ajoutez ces styles à votre `css/styles.css` :

```css
/* ===== FEEDBACK VISUEL POUR LES RÉPONSES ===== */

.answer-btn.disabled {
    opacity: 0.6;
    cursor: not-allowed;
}

.answer-btn.selected {
    transform: scale(1.05);
}

.answer-btn.correct {
    background: var(--success-color) !important;
    color: var(--text-light) !important;
    border-color: var(--success-color) !important;
}

.answer-btn.incorrect {
    background: var(--secondary-color) !important;
    color: var(--text-light) !important;
    border-color: var(--secondary-color) !important;
}

/* Animation pour le feedback */
.answer-btn.selected {
    animation: answerSelected 0.3s ease;
}

@keyframes answerSelected {
    0% { transform: scale(1); }
    50% { transform: scale(1.08); }
    100% { transform: scale(1.05); }
}
```

**Exercice :** Testez la navigation entre les écrans en cliquant sur les boutons.

---

## Étape 3 : Intégration complète

### 3.1 Finalisation de la logique d'affichage

Complétez les méthodes manquantes dans `js/game.js` :

```javascript
// Ajoutez ces méthodes à l'objet GameManager

updateQuestionDisplay(question) {
    const elements = ScreenManager.elements;
    
    // Mettre à jour le texte de la question
    if (elements.questionText) {
        elements.questionText.textContent = question.question;
    }
    
    // Mettre à jour les boutons de réponse
    elements.answerButtons.forEach((button, index) => {
        if (question.answers[index]) {
            button.textContent = question.answers[index];
            button.style.display = 'flex';
        } else {
            button.style.display = 'none';
        }
        
        // Réinitialiser les classes
        button.classList.remove('correct', 'incorrect', 'selected', 'disabled');
        button.disabled = false;
    });
    
    console.log("Affichage de la question mis à jour");
},

updateGameInfo() {
    const elements = ScreenManager.elements;
    const modeConfig = this.modes[this.gameMode];
    
    // Mettre à jour le score
    if (elements.scoreValue) {
        elements.scoreValue.textContent = this.score;
    }
    
    // Mettre à jour le compteur de questions
    if (elements.questionCounter) {
        const total = modeConfig.questionCount === 999 ? '∞' : modeConfig.questionCount;
        elements.questionCounter.textContent = `${this.currentQuestionIndex + 1}/${total}`;
    }
    
    // Mettre à jour le timer (sera mis à jour par le timer lui-même)
    if (elements.timerValue) {
        elements.timerValue.textContent = this.timeRemaining;
    }
    
    console.log("Informations de jeu mises à jour");
},

showResults(results) {
    const elements = ScreenManager.elements;
    
    // Mettre à jour les résultats
    if (elements.finalScore) {
        elements.finalScore.textContent = results.score;
    }
    
    if (elements.correctAnswers) {
        elements.correctAnswers.textContent = `${results.correctAnswers}/${results.totalQuestions}`;
    }
    
    if (elements.successRate) {
        elements.successRate.textContent = `${results.successRate}%`;
    }
    
    // Afficher l'écran de fin
    ScreenManager.showScreen('end');
    
    console.log("Résultats affichés");
}
```

### 3.2 Amélioration du timer en temps réel

Modifiez la méthode `startTimer` dans `GameManager` :

```javascript
startTimer() {
    this.clearTimer();
    
    const modeConfig = this.modes[this.gameMode];
    
    // Configuration spéciale pour le mode contre-la-montre
    if (this.gameMode === 'speed') {
        this.timeRemaining = modeConfig.totalTime;
        this.startSpeedModeTimer();
        return;
    }
    
    this.timeRemaining = modeConfig.timePerQuestion || 30;
    
    // Mettre à jour l'affichage immédiatement
    this.updateTimerDisplay();
    
    this.timerInterval = setInterval(() => {
        this.timeRemaining--;
        this.updateTimerDisplay();
        
        if (this.timeRemaining <= 0) {
            this.handleTimeout();
        }
    }, 1000);
},

startSpeedModeTimer() {
    this.updateTimerDisplay();
    
    this.timerInterval = setInterval(() => {
        this.timeRemaining--;
        this.updateTimerDisplay();
        
        if (this.timeRemaining <= 0) {
            console.log("Temps global écoulé en mode speed !");
            this.endGame();
        }
    }, 1000);
},

updateTimerDisplay() {
    const elements = ScreenManager.elements;
    if (elements.timerValue) {
        elements.timerValue.textContent = this.timeRemaining;
        
        // Changer la couleur si le temps devient critique
        if (this.timeRemaining <= 10) {
            elements.timerValue.style.color = 'var(--secondary-color)';
        } else {
            elements.timerValue.style.color = '';
        }
    }
}
```

### 3.3 Amélioration de la navigation

Ajoutez cette méthode à `ScreenManager` pour gérer la remise à zéro :

```javascript
// Ajoutez cette méthode à ScreenManager

resetAnswerButtons() {
    this.elements.answerButtons.forEach(button => {
        button.classList.remove('correct', 'incorrect', 'selected', 'disabled');
        button.disabled = false;
    });
},

// Modifiez la méthode handleAnswer pour utiliser cette remise à zéro
handleAnswer(answerIndex) {
    this.setAnswerButtonsEnabled(false);
    this.showAnswerFeedback(answerIndex);
    GameManager.handleAnswer(answerIndex);
    
    // Réinitialiser les boutons après la transition vers la question suivante
    setTimeout(() => {
        this.setAnswerButtonsEnabled(true);
        this.resetAnswerButtons();
    }, 2100);
}
```

---

## Étape 4 : Tests et débogage

### 4.1 Console de débogage

Ajoutez ces utilitaires de débogage dans `js/main.js` :

```javascript
// Utilitaires de débogage (à ajouter à la fin du fichier)

// Objet pour faciliter le débogage
window.DebugUtils = {
    // Afficher l'état actuel du jeu
    showGameState() {
        console.log("État actuel du jeu:", {
            screen: ScreenManager.currentScreen,
            gameActive: GameManager.gameActive,
            mode: GameManager.gameMode,
            question: GameManager.currentQuestionIndex + 1,
            score: GameManager.score,
            timeRemaining: GameManager.timeRemaining
        });
    },
    
    // Forcer l'affichage d'un écran
    goToScreen(screenName) {
        console.log(`Force l'affichage de l'écran: ${screenName}`);
        ScreenManager.showScreen(screenName);
    },
    
    // Simuler une bonne réponse
    correctAnswer() {
        if (GameManager.gameActive) {
            const question = GameManager.questions[GameManager.currentQuestionIndex];
            console.log(`Simulation bonne réponse: ${question.correct}`);
            ScreenManager.handleAnswer(question.correct);
        }
    },
    
    // Passer à la question suivante
    skipQuestion() {
        if (GameManager.gameActive) {
            console.log("Passage à la question suivante");
            GameManager.nextQuestion();
        }
