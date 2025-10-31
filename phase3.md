# TP Phase 3 - Quiz Game : Logique de jeu avancée et fonctionnalités

## Objectifs pédagogiques

À la fin de ce TP, vous serez capable de :
- ✅ Implémenter des algorithmes de jeu complexes
- ✅ Gérer des données persistantes avec localStorage
- ✅ Créer des systèmes de bonus et jokers
- ✅ Manipuler des structures de données avancées
- ✅ Optimiser les performances et l'expérience utilisateur

**Durée estimée :** 5-6 heures  
**Prérequis :** TP Phase 2 terminé et fonctionnel

---

## Préparation et extension

### Extension de l'arborescence

Ajoutez ces nouveaux fichiers à votre projet :

```
quiz-game/
├── index.html (à modifier)
├── css/
│   ├── styles.css (à étendre)
│   └── advanced.css (nouveau)
├── js/
│   ├── main.js (existant)
│   ├── game.js (à étendre)
│   ├── questions.js (à étendre)
│   ├── storage.js (nouveau)
│   ├── jokers.js (nouveau)
│   └── stats.js (nouveau)
└── README.md (à mettre à jour)
```

### Mise à jour des scripts dans index.html

Modifiez la section des scripts dans votre `index.html` :

```html
    <!-- Scripts JavaScript -->
    <script src="js/questions.js"></script>
    <script src="js/storage.js"></script>
    <script src="js/jokers.js"></script>
    <script src="js/stats.js"></script>
    <script src="js/game.js"></script>
    <script src="js/main.js"></script>
    
    <!-- Nouveau lien CSS -->
    <link rel="stylesheet" href="css/advanced.css">
</body>
</html>
```

**Objectif pédagogique :** Comprendre l'importance de l'ordre de chargement des modules.

---

## Étape 1 : Système de stockage persistant

### 1.1 Gestionnaire de stockage local

Créez le fichier `js/storage.js` :

```javascript
// ===== GESTIONNAIRE DE STOCKAGE LOCAL =====

const StorageManager = {
    // Clés de stockage
    keys: {
        highScores: 'quiz_high_scores',
        playerStats: 'quiz_player_stats',
        gameSettings: 'quiz_game_settings',
        unlockedAchievements: 'quiz_achievements'
    },

    // Vérifier si localStorage est disponible
    isAvailable() {
        try {
            const test = '__storage_test__';
            localStorage.setItem(test, test);
            localStorage.removeItem(test);
            return true;
        } catch (error) {
            console.warn("localStorage non disponible:", error);
            return false;
        }
    },

    // Sauvegarder des données
    save(key, data) {
        if (!this.isAvailable()) return false;
        
        try {
            const serializedData = JSON.stringify({
                data: data,
                timestamp: Date.now(),
                version: '1.0'
            });
            
            localStorage.setItem(key, serializedData);
            console.log(`Données sauvegardées: ${key}`);
            return true;
        } catch (error) {
            console.error("❌ Erreur de sauvegarde:", error);
            return false;
        }
    },

    // Charger des données
    load(key, defaultValue = null) {
        if (!this.isAvailable()) return defaultValue;
        
        try {
            const item = localStorage.getItem(key);
            if (!item) return defaultValue;
            
            const parsed = JSON.parse(item);
            console.log(`Données chargées: ${key}`);
            return parsed.data;
        } catch (error) {
            console.error("❌ Erreur de chargement:", error);
            return defaultValue;
        }
    },

    // Supprimer des données
    remove(key) {
        if (!this.isAvailable()) return false;
        
        try {
            localStorage.removeItem(key);
            console.log(`Données supprimées: ${key}`);
            return true;
        } catch (error) {
            console.error("❌ Erreur de suppression:", error);
            return false;
        }
    },

    // Effacer toutes les données du jeu
    clearAll() {
        if (!this.isAvailable()) return false;
        
        Object.values(this.keys).forEach(key => {
            this.remove(key);
        });
        
        console.log("Toutes les données effacées");
        return true;
    },

    // === MÉTHODES SPÉCIALISÉES ===

    // Sauvegarder un score
    saveScore(gameMode, score, stats) {
        const scores = this.getHighScores();
        const newScore = {
            mode: gameMode,
            score: score,
            stats: stats,
            date: new Date().toISOString(),
            id: Date.now()
        };

        // Ajouter le nouveau score
        if (!scores[gameMode]) {
            scores[gameMode] = [];
        }
        
        scores[gameMode].push(newScore);
        
        // Garder seulement les 10 meilleurs scores par mode
        scores[gameMode].sort((a, b) => b.score - a.score);
        scores[gameMode] = scores[gameMode].slice(0, 10);
        
        this.save(this.keys.highScores, scores);
        return newScore;
    },

    // Récupérer les meilleurs scores
    getHighScores() {
        return this.load(this.keys.highScores, {
            classic: [],
            speed: [],
            survival: []
        });
    },

    // Sauvegarder les statistiques du joueur
    savePlayerStats(stats) {
        const currentStats = this.getPlayerStats();
        
        // Fusionner les nouvelles stats avec les anciennes
        const updatedStats = {
            totalGames: (currentStats.totalGames || 0) + 1,
            totalQuestions: (currentStats.totalQuestions || 0) + stats.totalQuestions,
            totalCorrect: (currentStats.totalCorrect || 0) + stats.correctAnswers,
            totalScore: (currentStats.totalScore || 0) + stats.score,
            bestScore: Math.max(currentStats.bestScore || 0, stats.score),
            averageScore: 0, // Sera calculé
            gamesPerMode: {
                classic: (currentStats.gamesPerMode?.classic || 0) + (stats.mode === 'classic' ? 1 : 0),
                speed: (currentStats.gamesPerMode?.speed || 0) + (stats.mode === 'speed' ? 1 : 0),
                survival: (currentStats.gamesPerMode?.survival || 0) + (stats.mode === 'survival' ? 1 : 0)
            },
            lastPlayed: Date.now()
        };

        // Calculer la moyenne
        updatedStats.averageScore = Math.round(updatedStats.totalScore / updatedStats.totalGames);
        
        this.save(this.keys.playerStats, updatedStats);
        return updatedStats;
    },

    // Récupérer les statistiques du joueur
    getPlayerStats() {
        return this.load(this.keys.playerStats, {
            totalGames: 0,
            totalQuestions: 0,
            totalCorrect: 0,
            totalScore: 0,
            bestScore: 0,
            averageScore: 0,
            gamesPerMode: { classic: 0, speed: 0, survival: 0 },
            lastPlayed: null
        });
    },

    // Sauvegarder les paramètres
    saveSettings(settings) {
        this.save(this.keys.gameSettings, settings);
    },

    // Charger les paramètres
    getSettings() {
        return this.load(this.keys.gameSettings, {
            soundEnabled: true,
            animationsEnabled: true,
            difficulty: 'mixed',
            theme: 'default'
        });
    }
};

// Test de fonctionnement au chargement
console.log("StorageManager initialisé:", StorageManager.isAvailable() ? "✅" : "❌");
```

**Question de réflexion :** Pourquoi encapsuler les données dans un objet avec timestamp et version ?

### 1.2 Intégration du stockage dans le jeu

Modifiez votre `js/game.js` en ajoutant ces méthodes à `GameManager` :

```javascript
// Ajoutez ces méthodes à l'objet GameManager

// Méthode modifiée pour sauvegarder à la fin
endGame() {
    console.log("Fin de partie !");
    this.gameActive = false;
    this.clearTimer();
    
    // Calculer les statistiques
    const totalQuestions = this.currentQuestionIndex;
    const successRate = totalQuestions > 0 ? Math.round((this.correctAnswers / totalQuestions) * 100) : 0;
    
    const gameStats = {
        mode: this.gameMode,
        score: this.score,
        correctAnswers: this.correctAnswers,
        totalQuestions: totalQuestions,
        successRate: successRate,
        timeMode: this.modes[this.gameMode].name
    };
    
    // Sauvegarder les données
    this.saveGameData(gameStats);
    
    console.log(`Statistiques finales:
    - Score: ${this.score}
    - Bonnes réponses: ${this.correctAnswers}/${totalQuestions}
    - Taux de réussite: ${successRate}%`);
    
    // Afficher l'écran de fin avec indication si c'est un record
    const isNewRecord = this.checkIfNewRecord(this.score);
    this.showResults({
        ...gameStats,
        isNewRecord: isNewRecord
    });
},

// Sauvegarder les données de la partie
saveGameData(stats) {
    try {
        // Sauvegarder le score dans les records
        const savedScore = StorageManager.saveScore(this.gameMode, this.score, stats);
        
        // Mettre à jour les statistiques globales
        const updatedStats = StorageManager.savePlayerStats(stats);
        
        console.log("Données sauvegardées:", {
            score: savedScore,
            stats: updatedStats
        });
        
        return true;
    } catch (error) {
        console.error("❌ Erreur lors de la sauvegarde:", error);
        return false;
    }
},

// Vérifier si c'est un nouveau record
checkIfNewRecord(score) {
    const highScores = StorageManager.getHighScores();
    const modeScores = highScores[this.gameMode] || [];
    
    if (modeScores.length === 0) return true; // Premier score
    
    const bestScore = Math.max(...modeScores.map(s => s.score));
    return score > bestScore;
},

// Obtenir les statistiques pour l'affichage
getDisplayStats() {
    const playerStats = StorageManager.getPlayerStats();
    const highScores = StorageManager.getHighScores();
    
    return {
        player: playerStats,
        records: highScores
    };
}
```

**💡 Exercice :** Testez la sauvegarde en terminant une partie puis en rechargeant la page. Les données persistent-elles ?

---

## Étape 2 : Système de jokers et bonus

### 2.1 Gestionnaire de jokers

Créez le fichier `js/jokers.js` :

```javascript
// ===== GESTIONNAIRE DE JOKERS =====

const JokerManager = {
    // Types de jokers disponibles
    types: {
        fifty_fifty: {
            name: "50/50",
            description: "Élimine 2 mauvaises réponses",
            icon: "🎯",
            usesPerGame: 1,
            available: true
        },
        skip_question: {
            name: "Passer",
            description: "Passe à la question suivante sans pénalité",
            icon: "⏭️",
            usesPerGame: 1,
            available: true
        },
        extra_time: {
            name: "Temps +",
            description: "Ajoute 15 secondes au timer",
            icon: "⏱️",
            usesPerGame: 2,
            available: true
        },
        hint: {
            name: "Indice",
            description: "Affiche la catégorie et le niveau de difficulté",
            icon: "💡",
            usesPerGame: 3,
            available: true
        }
    },

    // État des jokers pour la partie en cours
    gameState: {},

    // Initialiser les jokers pour une nouvelle partie
    initForGame(gameMode) {
        console.log(`Initialisation des jokers pour le mode: ${gameMode}`);
        
        this.gameState = {};
        
        // Configurer les jokers selon le mode de jeu
        Object.keys(this.types).forEach(jokerId => {
            const joker = this.types[jokerId];
            
            this.gameState[jokerId] = {
                ...joker,
                remainingUses: this.getJokerLimit(jokerId, gameMode),
                used: false
            };
        });
        
        // Désactiver certains jokers selon le mode
        if (gameMode === 'survival') {
            this.gameState.skip_question.remainingUses = 0; // Pas de skip en survie
        }
        
        console.log("Jokers initialisés:", this.gameState);
    },

    // Obtenir le nombre de jokers autorisés selon le mode
    getJokerLimit(jokerId, gameMode) {
        const baseLimit = this.types[jokerId].usesPerGame;
        
        switch (gameMode) {
            case 'classic':
                return baseLimit;
            case 'speed':
                return Math.floor(baseLimit / 2) || 1; // Moins de jokers en speed
            case 'survival':
                return jokerId === 'skip_question' ? 0 : baseLimit;
            default:
                return baseLimit;
        }
    },

    // Vérifier si un joker est utilisable
    canUseJoker(jokerId) {
        const joker = this.gameState[jokerId];
        if (!joker) return false;
        
        return joker.remainingUses > 0 && joker.available;
    },

    // Utiliser un joker
    useJoker(jokerId, question) {
        if (!this.canUseJoker(jokerId)) {
            console.warn(`Joker ${jokerId} non utilisable`);
            return null;
        }

        console.log(`Utilisation du joker: ${jokerId}`);
        
        const joker = this.gameState[jokerId];
        joker.remainingUses--;
        
        // Exécuter l'effet du joker
        const result = this.executeJoker(jokerId, question);
        
        // Mettre à jour l'interface
        this.updateJokerDisplay();
        
        return result;
    },

    // Exécuter l'effet d'un joker spécifique
    executeJoker(jokerId, question) {
        switch (jokerId) {
            case 'fifty_fifty':
                return this.applyFiftyFifty(question);
            
            case 'skip_question':
                return this.applySkipQuestion();
            
            case 'extra_time':
                return this.applyExtraTime();
            
            case 'hint':
                return this.applyHint(question);
            
            default:
                console.error(`❌ Joker inconnu: ${jokerId}`);
                return null;
        }
    },

    // Joker 50/50 : éliminer 2 mauvaises réponses
    applyFiftyFifty(question) {
        const wrongAnswers = [];
        
        // Identifier les mauvaises réponses
        for (let i = 0; i < question.answers.length; i++) {
            if (i !== question.correct) {
                wrongAnswers.push(i);
            }
        }
        
        // Mélanger et prendre 2 réponses à éliminer
        const shuffled = wrongAnswers.sort(() => 0.5 - Math.random());
        const toRemove = shuffled.slice(0, 2);
        
        console.log(`50/50: Élimination des réponses ${toRemove}`);
        
        return {
            type: 'fifty_fifty',
            removedAnswers: toRemove,
            message: "2 mauvaises réponses éliminées !"
        };
    },

    // Joker passer : passer à la question suivante
    applySkipQuestion() {
        console.log("⏭️ Question passée");
        
        return {
            type: 'skip_question',
            message: "Question passée sans pénalité !"
        };
    },

    // Joker temps : ajouter du temps
    applyExtraTime() {
        const bonusTime = 15;
        console.log(` +${bonusTime} secondes ajoutées`);
        
        return {
            type: 'extra_time',
            bonusTime: bonusTime,
            message: `+${bonusTime} secondes ajoutées !`
        };
    },

    // Joker indice : afficher des informations
    applyHint(question) {
        const hint = {
            category: question.category,
            difficulty: this.getDifficultyName(question.difficulty),
            tip: this.generateTip(question)
        };
        
        console.log("💡 Indice fourni:", hint);
        
        return {
            type: 'hint',
            hint: hint,
            message: "Indice affiché !"
        };
    },

    // Obtenir le nom de la difficulté
    getDifficultyName(level) {
        const names = {
            1: "Facile",
            2: "Moyen", 
            3: "Difficile",
            4: "Expert",
            5: "Maître"
        };
        return names[level] || "Inconnu";
    },

    // Générer un conseil selon la question
    generateTip(question) {
        const tips = {
            "Histoire": "Pensez aux grandes dates et personnages historiques",
            "Sciences": "Rappelez-vous vos cours de physique-chimie",
            "Géographie": "Visualisez une carte du monde",
            "Sport": "Pensez aux règles et records sportifs",
            "Culture": "Faites appel à vos connaissances générales"
        };
        
        return tips[question.category] || "Faites confiance à votre instinct !";
    },

    // Mettre à jour l'affichage des jokers
    updateJokerDisplay() {
        // Cette méthode sera appelée par l'interface
        console.log("Mise à jour de l'affichage des jokers");
        
        if (window.ScreenManager && window.ScreenManager.updateJokersUI) {
            window.ScreenManager.updateJokersUI(this.gameState);
        }
    },

    // Obtenir l'état actuel des jokers
    getGameState() {
        return { ...this.gameState };
    },

    // Réinitialiser un joker spécifique (pour les tests)
    resetJoker(jokerId) {
        if (this.gameState[jokerId]) {
            this.gameState[jokerId].remainingUses = this.types[jokerId].usesPerGame;
            this.updateJokerDisplay();
        }
    }
};

// Fonctions utilitaires globales pour les jokers
window.JokerUtils = {
    // Appliquer l'effet visuel du 50/50
    applyFiftyFiftyVisual(removedAnswers) {
        const answerButtons = document.querySelectorAll('.answer-btn');
        
        removedAnswers.forEach(index => {
            if (answerButtons[index]) {
                answerButtons[index].style.opacity = '0.3';
                answerButtons[index].style.pointerEvents = 'none';
                answerButtons[index].classList.add('eliminated');
            }
        });
    },
    
    // Réinitialiser l'affichage des réponses
    resetAnswersVisual() {
        const answerButtons = document.querySelectorAll('.answer-btn');
        
        answerButtons.forEach(button => {
            button.style.opacity = '';
            button.style.pointerEvents = '';
            button.classList.remove('eliminated');
        });
    }
};

console.log("JokerManager initialisé");
```

### 2.2 Interface des jokers dans le HTML

Modifiez votre écran de jeu dans `index.html` en ajoutant la section des jokers :

```html
<!-- Dans l'écran de jeu, après la section question-section -->
<section class="jokers-section" id="jokers-section">
    <h3 class="jokers-title">Jokers disponibles</h3>
    <div class="jokers-grid">
        <button class="joker-btn" data-joker="fifty_fifty" title="50/50">
            <span class="joker-icon">🎯</span>
            <span class="joker-name">50/50</span>
            <span class="joker-count" id="fifty_fifty-count">1</span>
        </button>
        
        <button class="joker-btn" data-joker="skip_question" title="Passer la question">
            <span class="joker-icon">⏭️</span>
            <span class="joker-name">Passer</span>
            <span class="joker-count" id="skip_question-count">1</span>
        </button>
        
        <button class="joker-btn" data-joker="extra_time" title="Temps supplémentaire">
            <span class="joker-icon">⏱️</span>
            <span class="joker-name">Temps +</span>
            <span class="joker-count" id="extra_time-count">2</span>
        </button>
        
        <button class="joker-btn" data-joker="hint" title="Indice">
            <span class="joker-icon">💡</span>
            <span class="joker-name">Indice</span>
            <span class="joker-count" id="hint-count">3</span>
        </button>
    </div>
    
    <!-- Zone d'affichage des messages de jokers -->
    <div class="joker-message" id="joker-message"></div>
</section>
```

### 2.3 Styles pour les jokers

Créez le fichier `css/advanced.css` :

```css
/* ===== STYLES AVANCÉS - JOKERS ET NOUVELLES FONCTIONNALITÉS ===== */

/* Section des jokers */
.jokers-section {
    margin-top: var(--spacing-xl);
    padding: var(--spacing-lg);
    background: rgba(255, 255, 255, 0.1);
    border-radius: var(--border-radius);
    backdrop-filter: blur(10px);
    width: 100%;
    max-width: 800px;
}

.jokers-title {
    color: var(--text-light);
    text-align: center;
    margin-bottom: var(--spacing-md);
    font-size: var(--font-size-lg);
}

.jokers-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
    gap: var(--spacing-md);
    margin-bottom: var(--spacing-lg);
}

.joker-btn {
    background: var(--text-light);
    border: 2px solid transparent;
    border-radius: var(--border-radius);
    padding: var(--spacing-md);
    cursor: pointer;
    transition: var(--transition);
    position: relative;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: var(--spacing-xs);
    min-height: 80px;
}

.joker-btn:hover:not(.disabled) {
    border-color: var(--primary-color);
    transform: translateY(-2px);
    box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
}

.joker-btn.disabled {
    opacity: 0.4;
    cursor: not-allowed;
    background: #f8f9fa;
}

.joker-btn.used {
    background: var(--border-color);
    color: #666;
}

.joker-icon {
    font-size: 1.5rem;
}

.joker-name {
    font-size: var(--font-size-sm);
    font-weight: 600;
    color: var(--text-dark);
}

.joker-count {
    position: absolute;
    top: 5px;
    right: 5px;
    background: var(--primary-color);
    color: var(--text-light);
    width: 20px;
    height: 20px;
    border-radius: 50%;
    font-size: 0.7rem;
    display: flex;
    align-items: center;
    justify-content: center;
    font-weight: bold;
}

.joker-count.zero {
    background: var(--secondary-color);
}

/* Message des jokers */
.joker-message {
    text-align: center;
    padding: var(--spacing-sm);
    border-radius: var(--border-radius);
    font-weight: 600;
    opacity: 0;
    transform: translateY(-10px);
    transition: all 0.3s ease;
}

.joker-message.show {
    opacity: 1;
    transform: translateY(0);
}

.joker-message.success {
    background: var(--success-color);
    color: var(--text-light);
}

.joker-message.info {
    background: var(--primary-color);
    color: var(--text-light);
}

/* Réponses éliminées par le 50/50 */
.answer-btn.eliminated {
    background: var(--border-color) !important;
    color: #999 !important;
    opacity: 0.3 !important;
    cursor: not-allowed !important;
    text-decoration: line-through;
}

/* Zone d'indice */
.hint-display {
    background: rgba(241, 196, 15, 0.1);
    border: 2px solid var(--warning-color);
    border-radius: var(--border-radius);
    padding: var(--spacing-md);
    margin-top: var(--spacing-md);
    animation: hintAppear 0.5s ease;
}

.hint-display h4 {
    color: var(--warning-color);
    margin-bottom: var(--spacing-sm);
    display: flex;
    align-items: center;
    gap: var(--spacing-xs);
}

.hint-info {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
    gap: var(--spacing-sm);
    margin-bottom: var(--spacing-sm);
}

.hint-item {
    background: rgba(255, 255, 255, 0.8);
    padding: var(--spacing-xs);
    border-radius: calc(var(--border-radius) / 2);
    text-align: center;
}

.hint-tip {
    font-style: italic;
    color: var(--text-dark);
    background: rgba(255, 255, 255, 0.9);
    padding: var(--spacing-sm);
    border-radius: calc(var(--border-radius) / 2);
}

@keyframes hintAppear {
    from {
        opacity: 0;
        transform: scale(0.9);
    }
    to {
        opacity: 1;
        transform: scale(1);
    }
}

/* Responsive pour les jokers */
@media (max-width: 768px) {
    .jokers-grid {
        grid-template-columns: repeat(2, 1fr);
    }
    
    .joker-btn {
        min-height: 70px;
        padding: var(--spacing-sm);
    }
    
    .joker-icon {
        font-size: 1.2rem;
    }
    
    .hint-info {
        grid-template-columns: 1fr;
    }
}

/* Animation d'utilisation des jokers */
.joker-btn.using {
    animation: jokerUsed 0.6s ease;
}

@keyframes jokerUsed {
    0% { transform: scale(1); }
    50% { 
        transform: scale(1.2);
        box-shadow: 0 0 20px var(--primary-color);
    }
    100% { transform: scale(1); }
}
```

**Exercice :** Testez l'affichage des jokers en ajoutant `jokers-section` à l'écran de jeu.

---

## Étape 3 : Intégration des jokers dans le jeu

### 3.1 Extension du GameManager

Ajoutez ces méthodes à votre `js/game.js` :

```javascript
// Ajoutez ces méthodes à GameManager

// Initialisation avec jokers
startGame(mode) {
    console.log(`🚀 Démarrage du jeu en mode : ${mode}`);
    this.gameMode = mode;
    this.reset();
    
    // Initialiser les jokers pour cette partie
    JokerManager.initForGame(mode);
    
    // Charger les questions selon le mode
    const modeConfig = this.modes[mode];
    this.questions = getRandomQuestions(modeConfig.questionCount);
    
    if (this.questions.length === 0) {
        console.error("❌ Aucune question disponible !");
        return false;
    }

    this.gameActive = true;
    this.loadQuestion();
    return true;
},

// Gestion des jokers
handleJoker(jokerId) {
    if (!this.gameActive) return false;
    
    const currentQuestion = this.questions[this.currentQuestionIndex];
    const result = JokerManager.useJoker(jokerId, currentQuestion);
    
    if (!result) {
        console.warn(`⚠️ Impossible d'utiliser le joker: ${jokerId}`);
        return false;
    }
    
    // Traiter l'effet du joker
    this.applyJokerEffect(result);
    
    return true;
},

// Appliquer l'effet d'un joker
applyJokerEffect(jokerResult) {
    switch (jokerResult.type) {
        case 'fifty_fifty':
            this.applyFiftyFiftyEffect(jokerResult.removedAnswers);
            break;
            
        case 'skip_question':
            this.applySkipEffect();
            break;
            
        case 'extra_time':
            this.applyExtraTimeEffect(jokerResult.bonusTime);
            break;
            
        case 'hint':
            this.applyHintEffect(jokerResult.hint);
            break;
    }
    
    // Afficher le message du joker
    this.showJokerMessage(jokerResult.message, 'success');
},

// Effet 50/50
applyFiftyFiftyEffect(removedAnswers) {
    console.log("Application du 50/50");
    window.JokerUtils.applyFiftyFiftyVisual(removedAnswers);
},

// Effet skip
applySkipEffect() {
    console.log("Application du skip");
    this.clearTimer();
    
    // Passer à la question suivante après un court délai
    setTimeout(() => {
        this.nextQuestion();
    }, 1500);
},

// Effet temps supplémentaire
applyExtraTimeEffect(bonusTime) {
    console.log(`Ajout de ${bonusTime} secondes`);
    this.timeRemaining += bonusTime;
    this.updateTimerDisplay();
    
    // Animation visuelle du timer
    const timerElement = document.getElementById('timer-value');
    if (timerElement) {
        timerElement.style.animation = 'timerBonus 0.8s ease';
        setTimeout(() => {
            timerElement.style.animation = '';
        }, 800);
    }
},

// Effet indice
applyHintEffect(hint) {
    console.log("Affichage de l'indice:", hint);
    this.showHintDisplay(hint);
},

// Afficher un message de joker
showJokerMessage(message, type = 'info') {
    const messageElement = document.getElementById('joker-message');
    if (!messageElement) return;
    
    messageElement.textContent = message;
    messageElement.className = `joker-message ${type} show`;
    
    // Masquer le message après 3 secondes
    setTimeout(() => {
        messageElement.classList.remove('show');
    }, 3000);
},

// Afficher l'indice
showHintDisplay(hint) {
    const questionSection = document.querySelector('.question-section');
    if (!questionSection) return;
    
    // Supprimer l'ancien indice s'il existe
    const existingHint = questionSection.querySelector('.hint-display');
    if (existingHint) {
        existingHint.remove();
    }
    
    // Créer le nouvel affichage d'indice
    const hintElement = document.createElement('div');
    hintElement.className = 'hint-display';
    hintElement.innerHTML = `
        <h4>💡 Indice</h4>
        <div class="hint-info">
            <div class="hint-item">
                <strong>Catégorie:</strong><br>${hint.category}
            </div>
            <div class="hint-item">
                <strong>Difficulté:</strong><br>${hint.difficulty}
            </div>
        </div>
        <div class="hint-tip">${hint.tip}</div>
    `;
    
    questionSection.appendChild(hintElement);
},

// Réinitialiser les effets visuels des jokers
resetJokerEffects() {
    // Réinitialiser le 50/50
    window.JokerUtils.resetAnswersVisual();
    
    // Supprimer l'indice
    const hintDisplay = document.querySelector('.hint-display');
    if (hintDisplay) {
        hintDisplay.remove();
    }
},

// Méthode modifiée pour inclure la réinitialisation des jokers
loadQuestion() {
    if (this.currentQuestionIndex >= this.questions.length) {
        this.endGame();
        return;
    }

    // Réinitialiser les effets visuels des jokers précédents
    this.resetJokerEffects();

    const question = this.questions[this.currentQuestionIndex];
    console.log(`Chargement question ${this.currentQuestionIndex + 1}: ${question.question}`);
    
    // Mettre à jour l'affichage
    this.updateQuestionDisplay(question);
    this.updateGameInfo();
    
    // Démarrer le timer selon le mode
    this.startTimer();
}
```

### 3.2 Extension du ScreenManager

Ajoutez ces méthodes à votre `js/main.js` :

```javascript
// Ajoutez ces méthodes à ScreenManager

// Mise en cache des éléments de jokers (modifiez cacheElements)
cacheElements() {
    console.log("Mise en cache des éléments DOM");
    
    // ... code existant ...
    
    // Ajouter les éléments de jokers
    this.elements.jokers = {
        section: document.getElementById('jokers-section'),
        buttons: document.querySelectorAll('.joker-btn'),
        message: document.getElementById('joker-message'),
        counts: {
            fifty_fifty: document.getElementById('fifty_fifty-count'),
            skip_question: document.getElementById('skip_question-count'),
            extra_time: document.getElementById('extra_time-count'),
            hint: document.getElementById('hint-count')
        }
    };

    this.validateElements();
},

// Association des événements de jokers (modifiez bindEvents)
bindEvents() {
    console.log("Association des événements");
    
    // ... code existant ...
    
    // Événements des jokers
    this.elements.jokers.buttons.forEach(button => {
        button.addEventListener('click', () => {
            const jokerId = button.dataset.joker;
            console.log(`🃏 Tentative d'utilisation du joker: ${jokerId}`);
            this.handleJokerClick(jokerId, button);
        });
    });
},

// Gérer le clic sur un joker
handleJokerClick(jokerId, buttonElement) {
    // Vérifier si le joker est utilisable
    if (!JokerManager.canUseJoker(jokerId)) {
        console.warn(`Joker ${jokerId} non disponible`);
        this.showJokerMessage("Ce joker n'est plus disponible !", 'error');
        return;
    }
    
    // Demander confirmation pour certains jokers
    if (jokerId === 'skip_question') {
        if (!confirm("Êtes-vous sûr de vouloir passer cette question ?")) {
            return;
        }
    }
    
    // Animation du bouton
    buttonElement.classList.add('using');
    setTimeout(() => {
        buttonElement.classList.remove('using');
    }, 600);
    
    // Utiliser le joker via le GameManager
    const success = GameManager.handleJoker(jokerId);
    
    if (success) {
        // Mettre à jour l'affichage des jokers
        this.updateJokersUI(JokerManager.getGameState());
    }
},

// Mettre à jour l'interface des jokers
updateJokersUI(jokersState) {
    if (!this.elements.jokers.buttons) return;
    
    this.elements.jokers.buttons.forEach(button => {
        const jokerId = button.dataset.joker;
        const jokerData = jokersState[jokerId];
        
        if (!jokerData) return;
        
        // Mettre à jour le compteur
        const countElement = this.elements.jokers.counts[jokerId];
        if (countElement) {
            countElement.textContent = jokerData.remainingUses;
            
            // Changer la couleur si épuisé
            if (jokerData.remainingUses === 0) {
                countElement.classList.add('zero');
                button.classList.add('disabled');
            } else {
                countElement.classList.remove('zero');
                button.classList.remove('disabled');
            }
        }
    });
},

// Afficher un message de joker depuis l'interface
showJokerMessage(message, type = 'info') {
    const messageElement = this.elements.jokers.message;
    if (!messageElement) return;
    
    messageElement.textContent = message;
    messageElement.className = `joker-message ${type} show`;
    
    setTimeout(() => {
        messageElement.classList.remove('show');
    }, 3000);
},

// Réinitialiser l'affichage des jokers
resetJokersDisplay() {
    if (!this.elements.jokers.buttons) return;
    
    this.elements.jokers.buttons.forEach(button => {
        button.classList.remove('disabled', 'used');
    });
    
    // Réinitialiser les compteurs (sera fait par updateJokersUI)
    Object.values(this.elements.jokers.counts).forEach(countElement => {
        if (countElement) {
            countElement.classList.remove('zero');
        }
    });
},

// Méthode modifiée pour démarrer le jeu avec jokers
startGame(mode) {
    console.log(`Démarrage du jeu en mode: ${mode}`);
    
    if (GameManager.startGame(mode)) {
        this.showScreen('game');
        
        // Initialiser l'affichage des jokers
        setTimeout(() => {
            this.updateJokersUI(JokerManager.getGameState());
        }, 100);
    } else {
        console.error("❌ Impossible de démarrer le jeu");
        alert("Erreur lors du démarrage du jeu !");
    }
}
```

### 3.3 Ajout d'animations CSS pour les jokers

Ajoutez ces styles à `css/advanced.css` :

```css
/* Animations supplémentaires pour les jokers */

@keyframes timerBonus {
    0% { 
        transform: scale(1);
        color: var(--text-light);
    }
    50% { 
        transform: scale(1.3);
        color: var(--success-color);
        text-shadow: 0 0 10px var(--success-color);
    }
    100% { 
        transform: scale(1);
        color: var(--text-light);
    }
}

.joker-message.error {
    background: var(--secondary-color);
    color: var(--text-light);
}

/* Animation d'élimination pour le 50/50 */
.answer-btn.eliminating {
    animation: eliminateAnswer 0.8s ease-out forwards;
}

@keyframes eliminateAnswer {
    0% {
        opacity: 1;
        transform: scale(1);
    }
    50% {
        transform: scale(0.9);
        background: var(--secondary-color);
    }
    100% {
        opacity: 0.3;
        transform: scale(1);
        background: var(--border-color);
    }
}

/* Indicateur de joker utilisé récemment */
.joker-btn.recently-used {
    animation: jokerCooldown 2s ease-out;
}

@keyframes jokerCooldown {
    0% {
        background: var(--success-color);
        color: var(--text-light);
        transform: scale(1.05);
    }
    100% {
        background: var(--text-light);
        color: var(--text-dark);
        transform: scale(1);
    }
}
```

---

## Étape 4 : Système de statistiques et scores

### 4.1 Gestionnaire de statistiques

Créez le fichier `js/stats.js` :

```javascript
// ===== GESTIONNAIRE DE STATISTIQUES =====

const StatsManager = {
    // Calculer les statistiques détaillées d'une partie
    calculateGameStats(gameData) {
        const stats = {
            // Statistiques de base
            score: gameData.score,
            correctAnswers: gameData.correctAnswers,
            totalQuestions: gameData.totalQuestions,
            successRate: gameData.successRate,
            
            // Statistiques avancées
            averageTimePerQuestion: 0,
            perfectStreak: 0,
            longestStreak: 0,
            categoriesPlayed: new Set(),
            difficultyDistribution: {},
            
            // Métadonnées
            mode: gameData.mode,
            duration: 0,
            timestamp: Date.now()
        };
        
        // Analyser les questions pour des stats détaillées
        if (gameData.questionDetails) {
            this.analyzeQuestionDetails(stats, gameData.questionDetails);
        }
        
        return stats;
    },

    // Analyser les détails des questions
    analyzeQuestionDetails(stats, questionDetails) {
        let currentStreak = 0;
        let maxStreak = 0;
        let totalTime = 0;
        
        questionDetails.forEach((question, index) => {
            // Catégories
            stats.categoriesPlayed.add(question.category);
            
            // Distribution de difficulté
            const difficulty = question.difficulty || 1;
            stats.difficultyDistribution[difficulty] = 
                (stats.difficultyDistribution[difficulty] || 0) + 1;
            
            // Calculs de série
            if (question.correct) {
                currentStreak++;
                maxStreak = Math.max(maxStreak, currentStreak);
            } else {
                currentStreak = 0;
            }
            
            // Temps moyen (si disponible)
            if (question.timeSpent) {
                totalTime += question.timeSpent;
            }
        });
        
        stats.longestStreak = maxStreak;
        stats.averageTimePerQuestion = questionDetails.length > 0 ? 
            Math.round(totalTime / questionDetails.length) : 0;
        
        // Série parfaite si toutes les réponses sont correctes
        if (currentStreak === questionDetails.length && questionDetails.length > 0) {
            stats.perfectStreak = currentStreak;
        }
    },

    // Générer un rapport textuel des performances
    generatePerformanceReport(stats) {
        const report = {
            grade: this.calculateGrade(stats.successRate),
            analysis: [],
            recommendations: [],
            achievements: []
        };
        
        // Analyse des performances
        if (stats.successRate >= 90) {
            report.analysis.push("Excellente performance ! Vous maîtrisez parfaitement le sujet.");
        } else if (stats.successRate >= 70) {
            report.analysis.push("Bonne performance avec quelques erreurs à corriger.");
        } else if (stats.successRate >= 50) {
            report.analysis.push("Performance moyenne. Il y a de la place pour l'amélioration.");
        } else {
            report.analysis.push("Performance en dessous de la moyenne. Révision recommandée.");
        }
        
        // Analyse des séries
        if (stats.longestStreak >= 5) {
            report.analysis.push(`Excellente série de ${stats.longestStreak} bonnes réponses consécutives !`);
        }
        
        // Recommandations
        if (stats.averageTimePerQuestion > 25) {
            report.recommendations.push("Travaillez votre vitesse de réponse pour améliorer vos performances.");
        }
        
        if (Object.keys(stats.difficultyDistribution).length > 1) {
            const hardQuestions = stats.difficultyDistribution[3] || 0;
            if (hardQuestions > 0) {
                report.recommendations.push("Continuez à vous challenger avec des questions difficiles !");
            }
        }
        
        // Succès
        if (stats.perfectStreak > 0) {
            report.achievements.push("Série parfaite !");
        }
        
        if (stats.score > 200) {
            report.achievements.push("Score élevé !");
        }
        
        return report;
    },

    // Calculer une note basée sur le taux de réussite
    calculateGrade(successRate) {
        if (successRate >= 95) return { letter: 'A+', color: '#27ae60', description: 'Exceptionnel' };
        if (successRate >= 90) return { letter: 'A', color: '#2ecc71', description: 'Excellent' };
        if (successRate >= 85) return { letter: 'A-', color: '#58d68d', description: 'Très bien' };
        if (successRate >= 80) return { letter: 'B+', color: '#82e0aa', description: 'Bien' };
        if (successRate >= 75) return { letter: 'B', color: '#f39c12', description: 'Assez bien' };
        if (successRate >= 70) return { letter: 'B-', color: '#f7dc6f', description: 'Passable' };
        if (successRate >= 60) return { letter: 'C', color: '#eb984e', description: 'Médiocre' };
        if (successRate >= 50) return { letter: 'D', color: '#e67e22', description: 'Insuffisant' };
        return { letter: 'F', color: '#e74c3c', description: 'Échec' };
    },

    // Comparer avec les performances précédentes
    compareWithHistory(currentStats, playerStats) {
        const comparison = {
            scoreImprovement: 0,
            successRateImprovement: 0,
            isPersonalBest: false,
            trend: 'stable'
        };
        
        if (playerStats.totalGames > 1) {
            // Amélioration du score
            if (playerStats.bestScore > 0) {
                comparison.scoreImprovement = 
                    ((currentStats.score - playerStats.averageScore) / playerStats.averageScore) * 100;
            }
            
            // Record personnel
            comparison.isPersonalBest = currentStats.score > playerStats.bestScore;
            
            // Tendance générale
            if (comparison.scoreImprovement > 10) {
                comparison.trend = 'improving';
            } else if (comparison.scoreImprovement < -10) {
                comparison.trend = 'declining';
            }
        }
        
        return comparison;
    },

    // Générer des conseils personnalisés
    generatePersonalizedTips(stats, playerStats) {
        const tips = [];
        
        // Conseils basés sur les catégories
        const categoryStats = this.analyzeCategoryPerformance(playerStats);
        const weakestCategory = Object.keys(categoryStats).reduce((a, b) => 
            categoryStats[a] < categoryStats[b] ? a : b
        );
        
        if (weakestCategory && categoryStats[weakestCategory] < 60) {
            tips.push(`💡 Concentrez-vous sur la catégorie "${weakestCategory}" où vous avez le plus de difficultés.`);
        }
        
        // Conseils basés sur le mode de jeu
        if (stats.mode === 'speed' && stats.averageTimePerQuestion > 20) {
            tips.push("⚡ En mode vitesse, essayez de répondre plus rapidement en vous fiant à votre première intuition.");
        }
        
        if (stats.mode === 'survival' && stats.correctAnswers < stats.totalQuestions) {
            tips.push("🛡️ En mode survie, prenez le temps de bien réfléchir avant de répondre.");
        }
        
        // Conseils généraux
        if (stats.longestStreak < 3) {
            tips.push("Travaillez votre concentration pour maintenir des séries de bonnes réponses.");
        }
        
        return tips;
    },

    // Analyser les performances par catégorie (simulation)
    analyzeCategoryPerformance(playerStats) {
        // Simulation - dans une vraie app, ces données seraient stockées
        return {
            "Histoire": 75,
            "Sciences": 60,
            "Géographie": 80,
            "Sport": 70,
            "Culture": 85
        };
    },

    // Formater les statistiques pour l'affichage
    formatStatsForDisplay(stats) {
        return {
            score: stats.score.toLocaleString(),
            successRate: `${stats.successRate}%`,
            streak: stats.longestStreak,
            avgTime: stats.averageTimePerQuestion > 0 ? 
                `${stats.averageTimePerQuestion}s` : 'N/A',
            categories: Array.from(stats.categoriesPlayed).join(', '),
            grade: stats.grade || this.calculateGrade(stats.successRate)
        };
    }
};

// Utilitaires pour l'affichage des statistiques
window.StatsDisplay = {
    // Créer un graphique simple avec du CSS
    createPerformanceChart(data) {
        const chartContainer = document.createElement('div');
        chartContainer.className = 'performance-chart';
        
        const maxValue = Math.max(...Object.values(data));
        
        Object.entries(data).forEach(([label, value]) => {
            const bar = document.createElement('div');
            bar.className = 'chart-bar';
            
            const height = (value / maxValue) * 100;
            bar.innerHTML = `
                <div class="bar-fill" style="height: ${height}%"></div>
                <div class="bar-label">${label}</div>
                <div class="bar-value">${value}%</div>
            `;
            
            chartContainer.appendChild(bar);
        });
        
        return chartContainer;
    },
    
    // Afficher les succès débloqués
    showAchievements(achievements) {
        if (achievements.length === 0) return null;
        
        const container = document.createElement('div');
        container.className = 'achievements-container';
        container.innerHTML = '<h4>🏆 Succès débloqués</h4>';
        
        achievements.forEach(achievement => {
            const badge = document.createElement('div');
            badge.className = 'achievement-badge';
            badge.textContent = achievement;
            container.appendChild(badge);
        });
        
        return container;
    }
};

console.log("StatsManager initialisé");
```

### 4.2 Extension de l'écran de fin avec statistiques

Modifiez votre écran de fin dans `index.html` :

```html
<!-- Remplacez la section results-section dans end-screen -->
<section class="results-section">
    <div class="final-score">
        <span class="score-label">Score final :</span>
        <span class="score-number" id="final-score">0</span>
        <div class="grade-display" id="grade-display">
            <span class="grade-letter">A</span>
            <span class="grade-description">Excellent</span>
        </div>
    </div>
    
    <div class="stats-grid">
        <div class="stat-card">
            <div class="stat-icon">✅</div>
            <div class="stat-content">
                <div class="stat-value" id="correct-answers">0</div>
                <div class="stat-label">Bonnes réponses</div>
            </div>
        </div>
        
        <div class="stat-card">
            <div class="stat-icon">📊</div>
            <div class="stat-content">
                <div class="stat-value" id="success-rate">0%</div>
                <div class="stat-label">Taux de réussite</div>
            </div>
        </div>
        
        <div class="stat-card">
            <div class="stat-icon">🔥</div>
            <div class="stat-content">
                <div class="stat-value" id="best-streak">0</div>
                <div class="stat-label">Meilleure série</div>
            </div>
        </div>
        
        <div class="stat-card">
            <div class="stat-icon">⏱️</div>
            <div class="stat-content">
                <div class="stat-value" id="avg-time">0s</div>
                <div class="stat-label">Temps moyen</div>
            </div>
        </div>
    </div>
    
    <!-- Zone pour les graphiques et analyses -->
    <div class="detailed-stats" id="detailed-stats">
        <!-- Sera rempli dynamiquement -->
    </div>
    
    <!-- Zone pour les conseils personnalisés -->
    <div class="tips-section" id="tips-section">
        <!-- Sera rempli dynamiquement -->
    </div>
    
    <div class="end-actions">
        <button class="btn btn-primary">Rejouer</button>
        <button class="btn btn-secondary">Menu principal</button>
        <button class="btn btn-info" id="view-history">Historique</button>
    </div>
</section>
```

### 4.3 Styles pour les nouvelles statistiques

Ajoutez à `css/advanced.css` :

```css
/* ===== STATISTIQUES AVANCÉES ===== */

.grade-display {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: var(--spacing-sm);
    margin-top: var(--spacing-md);
    padding: var(--spacing-sm);
    border-radius: var(--border-radius);
    background: rgba(255, 255, 255, 0.1);
}

.grade-letter {
    font-size: 2rem;
    font-weight: bold;
    padding: var(--spacing-sm);
    border-radius: 50%;
    background: var(--success-color);
    color: var(--text-light);
    min-width: 60px;
    text-align: center;
}

.grade-description {
    font-size: var(--font-size-lg);
    font-weight: 600;
    color: var(--text-dark);
}

.stats-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
    gap: var(--spacing-md);
    margin: var(--spacing-xl) 0;
}

.stat-card {
    background: rgba(255, 255, 255, 0.9);
    padding: var(--spacing-md);
    border-radius: var(--border-radius);
    text-align: center;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
    transition: var(--transition);
}

.stat-card:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 15px rgba(0, 0, 0, 0.15);
}

.stat-icon {
    font-size: 1.5rem;
    margin-bottom: var(--spacing-xs);
}

.stat-value {
    font-size: var(--font-size-xl);
    font-weight: bold;
    color: var(--primary-color);
    margin-bottom: var(--spacing-xs);
}

.stat-label {
    font-size: var(--font-size-sm);
    color: var(--text-dark);
    opacity: 0.8;
}

/* Graphiques de performance */
.performance-chart {
    display: flex;
    align-items: end;
    justify-content: space-around;
    height: 200px;
    background: rgba(255, 255, 255, 0.9);
    padding: var(--spacing-lg);
    border-radius: var(--border-radius);
    margin: var(--spacing-lg) 0;
}

.chart-bar {
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 60px;
}

.bar-fill {
    width: 100%;
    background: linear-gradient(180deg, var(--primary-color), var(--primary-dark));
    border-radius: 4px 4px 0 0;
    transition: height 1s ease;
    margin-bottom: var(--spacing-xs);
}

.bar-label {
    font-size: var(--font-size-sm);
    font-weight: 600;
    text-align: center;
    margin-bottom: var(--spacing-xs);
}

.bar-value {
    font-size: var(--font-size-sm);
    color: var(--primary-color);
    font-weight: bold;
}

/* Section des conseils */
.tips-section {
    background: rgba(241, 196, 15, 0.1);
    border: 2px solid var(--warning-color);
    border-radius: var(--border-radius);
    padding: var(--spacing-lg);
    margin: var(--spacing-lg) 0;
}

.tips-section h4 {
    color: var(--warning-color);
    margin-bottom: var(--spacing-md);
    display: flex;
    align-items: center;
    gap: var(--spacing-xs);
}

.tip-item {
    background: rgba(255, 255, 255, 0.8);
    padding: var(--spacing-sm);
    margin-bottom: var(--spacing-sm);
    border-radius: calc(var(--border-radius) / 2);
    border-left: 4px solid var(--warning-color);
}

.tip-item:last-child {
    margin-bottom: 0;
}

/* Badges de succès */
.achievements-container {
    margin: var(--spacing-lg) 0;
}

.achievements-container h4 {
    color: var(--success-color);
    margin-bottom: var(--spacing-md);
    text-align: center;
}

.achievement-badge {
    display: inline-block;
    background: var(--success-color);
    color: var(--text-light);
    padding: var(--spacing-xs) var(--spacing-sm);
    border-radius: 20px;
    font-size: var(--font-size-sm);
    font-weight: 600;
    margin: var(--spacing-xs);
    animation: badgeAppear 0.5s ease;
}

@keyframes badgeAppear {
    from {
        opacity: 0;
        transform: scale(0.8);
    }
    to {
        opacity: 1;
        transform: scale(1);
    }
}

/* Responsive pour les statistiques */
@media (max-width: 768px) {
    .stats-grid {
        grid-template-columns: repeat(2, 1fr);
    }
    
    .performance-chart {
        height: 150px;
        padding: var(--spacing-md);
    }
    
    .chart-bar {
        width: 40px;
    }
    
    .grade-letter {
        font-size: 1.5rem;
        min-width: 50px;
    }
}

/* Écran d'historique des scores */
.history-screen {
    background: var(--text-light);
    border-radius: var(--border-radius);
    padding: var(--spacing-xl);
    max-width: 800px;
    width: 100%;
    max-height: 80vh;
    overflow-y: auto;
}

.history-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: var(--spacing-lg);
    border-bottom: 2px solid var(--border-color);
    padding-bottom: var(--spacing-md);
}

.mode-tabs {
    display: flex;
    gap: var(--spacing-sm);
    margin-bottom: var(--spacing-lg);
}

.mode-tab {
    padding: var(--spacing-sm) var(--spacing-md);
    border: 2px solid var(--border-color);
    background: transparent;
    border-radius: var(--border-radius);
    cursor: pointer;
    transition: var(--transition);
}

.mode-tab.active {
    background: var(--primary-color);
    color: var(--text-light);
    border-color: var(--primary-color);
}

.scores-list {
    display: flex;
    flex-direction: column;
    gap: var(--spacing-sm);
}

.score-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: var(--spacing-md);
    background: var(--light-bg);
    border-radius: var(--border-radius);
    transition: var(--transition);
}

.score-item:hover {
    background: #e8f4f8;
    transform: translateX(5px);
}

.score-item.best {
    border-left: 4px solid var(--success-color);
    background: rgba(39, 174, 96, 0.1);
}

.score-info {
    display: flex;
    flex-direction: column;
    gap: var(--spacing-xs);
}

.score-value {
    font-size: var(--font-size-lg);
    font-weight: bold;
    color: var(--primary-color);
}

.score-details {
    font-size: var(--font-size-sm);
    color: var(--text-dark);
    opacity: 0.8;
}

.score-date {
    font-size: var(--font-size-sm);
    color: var(--text-dark);
    opacity: 0.6;
}
```

---

## Étape 5 : Intégration complète des statistiques

### 5.1 Extension du GameManager pour les statistiques

Modifiez votre `js/game.js` en ajoutant le tracking des données :

```javascript
// Ajoutez ces propriétés à GameManager
const GameManager = {
    // ... propriétés existantes ...
    
    // Nouvelles propriétés pour le tracking
    questionStartTime: 0,
    questionDetails: [], // Détails de chaque question répondue
    gameStartTime: 0,
    jokersUsed: [], // Liste des jokers utilisés
    
    // Méthode modifiée pour inclure le tracking
    startGame(mode) {
        console.log(`🚀 Démarrage du jeu en mode : ${mode}`);
        this.gameMode = mode;
        this.reset();
        
        // Initialiser le tracking
        this.gameStartTime = Date.now();
        this.questionDetails = [];
        this.jokersUsed = [];
        
        // Initialiser les jokers pour cette partie
        JokerManager.initForGame(mode);
        
        // Charger les questions selon le mode
        const modeConfig = this.modes[mode];
        this.questions = getRandomQuestions(modeConfig.questionCount);
        
        if (this.questions.length === 0) {
            console.error("❌ Aucune question disponible !");
            return false;
        }

        this.gameActive = true;
        this.loadQuestion();
        return true;
    },

    // Méthode modifiée pour tracker le temps
    loadQuestion() {
        if (this.currentQuestionIndex >= this.questions.length) {
            this.endGame();
            return;
        }

        // Démarrer le tracking du temps pour cette question
        this.questionStartTime = Date.now();

        // Réinitialiser les effets visuels des jokers précédents
        this.resetJokerEffects();

        const question = this.questions[this.currentQuestionIndex];
        console.log(`Chargement question ${this.currentQuestionIndex + 1}: ${question.question}`);
        
        // Mettre à jour l'affichage
        this.updateQuestionDisplay(question);
        this.updateGameInfo();
        
        // Démarrer le timer selon le mode
        this.startTimer();
    },

    // Méthode modifiée pour enregistrer les détails
    handleAnswer(answerIndex) {
        if (!this.gameActive) return;

        const question = this.questions[this.currentQuestionIndex];
        const isCorrect = answerIndex === question.correct;
        const timeSpent = Math.round((Date.now() - this.questionStartTime) / 1000);
        
        // Enregistrer les détails de cette question
        this.questionDetails.push({
            questionId: question.id,
            question: question.question,
            category: question.category,
            difficulty: question.difficulty,
            selectedAnswer: answerIndex,
            correctAnswer: question.correct,
            correct: isCorrect,
            timeSpent: timeSpent,
            remainingTime: this.timeRemaining
        });
        
        console.log(`💭 Réponse ${answerIndex}: ${isCorrect ? 'Correcte ✅' : 'Incorrecte ❌'} (${timeSpent}s)`);
        
        this.clearTimer();
        this.processAnswer(isCorrect);
        
        // Passer à la question suivante après un délai
        setTimeout(() => {
            this.nextQuestion();
        }, 2000);
    },

    // Méthode modifiée pour tracker les jokers
    handleJoker(jokerId) {
        if (!this.gameActive) return false;
        
        const currentQuestion = this.questions[this.currentQuestionIndex];
        const result = JokerManager.useJoker(jokerId, currentQuestion);
        
        if (!result) {
            console.warn(`⚠️ Impossible d'utiliser le joker: ${jokerId}`);
            return false;
        }
        
        // Enregistrer l'utilisation du joker
        this.jokersUsed.push({
            jokerId: jokerId,
            questionIndex: this.currentQuestionIndex,
            timestamp: Date.now()
        });
        
        // Traiter l'effet du joker
        this.applyJokerEffect(result);
        
        return true;
    },

    // Méthode modifiée pour les statistiques complètes
    endGame() {
        console.log("🏁 Fin de partie !");
        this.gameActive = false;
        this.clearTimer();
        
        // Calculer les statistiques complètes
        const gameDuration = Math.round((Date.now() - this.gameStartTime) / 1000);
        const totalQuestions = this.currentQuestionIndex;
        const successRate = totalQuestions > 0 ? Math.round((this.correctAnswers / totalQuestions) * 100) : 0;
        
        const gameStats = {
            mode: this.gameMode,
            score: this.score,
            correctAnswers: this.correctAnswers,
            totalQuestions: totalQuestions,
            successRate: successRate,
            timeMode: this.modes[this.gameMode].name,
            duration: gameDuration,
            questionDetails: this.questionDetails,
            jokersUsed: this.jokersUsed
        };
        
        // Générer les statistiques avancées
        const detailedStats = StatsManager.calculateGameStats(gameStats);
        const performanceReport = StatsManager.generatePerformanceReport(detailedStats);
        
        // Sauvegarder les données
        this.saveGameData(gameStats);
        
        console.log(`Statistiques finales:`, detailedStats);
        
        // Afficher l'écran de fin avec toutes les données
        const isNewRecord = this.checkIfNewRecord(this.score);
        this.showResults({
            ...gameStats,
            detailedStats: detailedStats,
            performanceReport: performanceReport,
            isNewRecord: isNewRecord
        });
    },

    // Nouvelle méthode pour afficher les résultats enrichis
    showResults(results) {
        const elements = ScreenManager.elements;
        
        // Statistiques de base
        if (elements.finalScore) {
            elements.finalScore.textContent = results.score;
        }
        
        if (elements.correctAnswers) {
            elements.correctAnswers.textContent = `${results.correctAnswers}/${results.totalQuestions}`;
        }
        
        if (elements.successRate) {
            elements.successRate.textContent = `${results.successRate}%`;
        }
        
        // Affichage de la note
        this.displayGrade(results.performanceReport.grade);
        
        // Statistiques détaillées
        this.displayDetailedStats(results.detailedStats);
        
        // Conseils personnalisés
        this.displayPersonalizedTips(results);
        
        // Afficher l'écran de fin
        ScreenManager.showScreen('end');
        
        console.log("Résultats complets affichés");
    },

    // Afficher la note
    displayGrade(grade) {
        const gradeDisplay = document.getElementById('grade-display');
        if (!gradeDisplay) return;
        
        const letterElement = gradeDisplay.querySelector('.grade-letter');
        const descriptionElement = gradeDisplay.querySelector('.grade-description');
        
        if (letterElement) {
            letterElement.textContent = grade.letter;
            letterElement.style.background = grade.color;
        }
        
        if (descriptionElement) {
            descriptionElement.textContent = grade.description;
        }
    },

    // Afficher les statistiques détaillées
    displayDetailedStats(stats) {
        const detailedStatsContainer = document.getElementById('detailed-stats');
        if (!detailedStatsContainer) return;
        
        // Nettoyer le contenu précédent
        detailedStatsContainer.innerHTML = '';
        
        // Afficher la série la plus longue
        const bestStreakElement = document.getElementById('best-streak');
        if (bestStreakElement) {
            bestStreakElement.textContent = stats.longestStreak;
        }
        
        // Afficher le temps moyen
        const avgTimeElement = document.getElementById('avg-time');
        if (avgTimeElement) {
            avgTimeElement.textContent = stats.averageTimePerQuestion > 0 ? 
                `${stats.averageTimePerQuestion}s` : 'N/A';
        }
        
        // Créer un graphique des catégories (si applicable)
        if (stats.categoriesPlayed.size > 1) {
            const categoryData = {};
            Array.from(stats.categoriesPlayed).forEach(category => {
                // Simulation de données - dans une vraie app, calculer les vrais pourcentages
                categoryData[category] = Math.floor(Math.random() * 40) + 60;
            });
            
            const chart = window.StatsDisplay.createPerformanceChart(categoryData);
            chart.insertAdjacentHTML('beforebegin', '<h4>📊 Performance par catégorie</h4>');
            detailedStatsContainer.appendChild(chart);
        }
        
        // Afficher les succès
        const achievements = this.calculateAchievements(stats);
        if (achievements.length > 0) {
            const achievementsDisplay = window.StatsDisplay.showAchievements(achievements);
            if (achievementsDisplay) {
                detailedStatsContainer.appendChild(achievementsDisplay);
            }
        }
    },

    // Calculer les succès débloqués
    calculateAchievements(stats) {
        const achievements = [];
        
        if (stats.successRate === 100) {
            achievements.push("🎯 Sans faute !");
        }
        
        if (stats.longestStreak >= 10) {
            achievements.push("🔥 Série de feu !");
        }
        
        if (stats.averageTimePerQuestion < 10) {
            achievements.push("⚡ Réflexes d'éclair !");
        }
        
        if (this.jokersUsed.length === 0) {
            achievements.push("🚫 Aucun joker utilisé !");
        }
        
        if (stats.score > 300) {
            achievements.push("💎 Score légendaire !");
        }
        
        return achievements;
    },

    // Afficher les conseils personnalisés
    displayPersonalizedTips(results) {
        const tipsContainer = document.getElementById('tips-section');
        if (!tipsContainer) return;
        
        // Obtenir les statistiques du joueur
        const playerStats = StorageManager.getPlayerStats();
        
        // Générer les conseils
        const tips = StatsManager.generatePersonalizedTips(results.detailedStats, playerStats);
        
        if (tips.length === 0) {
            tipsContainer.style.display = 'none';
            return;
        }
        
        tipsContainer.style.display = 'block';
        tipsContainer.innerHTML = '<h4>💡 Conseils personnalisés</h4>';
        
        tips.forEach(tip => {
            const tipElement = document.createElement('div');
            tipElement.className = 'tip-item';
            tipElement.textContent = tip;
            tipsContainer.appendChild(tipElement);
        });
    }
};
```

### 5.2 Écran d'historique des scores

Ajoutez cette nouvelle section à votre `index.html` :

```html
<!-- Nouvel écran d'historique (après end-screen) -->
<main id="history-screen" class="screen">
    <div class="history-screen">
        <header class="history-header">
            <h2>📈 Historique des scores</h2>
            <button class="btn btn-back" id="history-back">← Retour</button>
        </header>
        
        <div class="mode-tabs" id="mode-tabs">
            <button class="mode-tab active" data-mode="classic">Classique</button>
            <button class="mode-tab" data-mode="speed">Vitesse</button>
            <button class="mode-tab" data-mode="survival">Survie</button>
        </div>
        
        <div class="scores-list" id="scores-list">
            <!-- Sera rempli dynamiquement -->
        </div>
        
        <div class="history-stats" id="history-stats">
            <!-- Statistiques globales -->
        </div>
    </div>
</main>
```

### 5.3 Extension du ScreenManager pour l'historique

Ajoutez ces méthodes à votre `js/main.js` :

```javascript
// Ajoutez à l'objet ScreenManager

// Mise en cache des nouveaux éléments
cacheElements() {
    // ... code existant ...
    
    // Ajouter les éléments d'historique
    this.screens.history = document.getElementById('history-screen');
    
    this.elements.history = {
        back: document.getElementById('history-back'),
        tabs: document.querySelectorAll('.mode-tab'),
        scoresList: document.getElementById('scores-list'),
        stats: document.getElementById('history-stats')
    };
    
    this.validateElements();
},

// Association des événements d'historique
bindEvents() {
    // ... code existant ...
    
    // Bouton "Voir l'historique"
    const viewHistoryBtn = document.getElementById('view-history');
    if (viewHistoryBtn) {
        viewHistoryBtn.addEventListener('click', () => {
            console.log("Affichage de l'historique");
            this.showHistoryScreen();
        });
    }
    
    // Bouton retour de l'historique
    if (this.elements.history.back) {
        this.elements.history.back.addEventListener('click', () => {
            console.log("Retour depuis l'historique");
            this.showScreen('end');
        });
    }
    
    // Onglets des modes
    this.elements.history.tabs.forEach(tab => {
        tab.addEventListener('click', () => {
            const mode = tab.dataset.mode;
            console.log(`Changement d'onglet: ${mode}`);
            this.switchHistoryMode(mode);
        });
    });
},

// Afficher l'écran d'historique
showHistoryScreen() {
    this.showScreen('history');
    this.loadHistoryData('classic'); // Mode par défaut
},

// Changer le mode dans l'historique
switchHistoryMode(mode) {
    // Mettre à jour les onglets
    this.elements.history.tabs.forEach(tab => {
        tab.classList.toggle('active', tab.dataset.mode === mode);
    });
    
    // Charger les données du mode
    this.loadHistoryData(mode);
},

// Charger les données d'historique
loadHistoryData(mode) {
    const highScores = StorageManager.getHighScores();
    const modeScores = highScores[mode] || [];
    
    this.displayScoresList(modeScores, mode);
    this.displayHistoryStats(mode);
},

// Afficher la liste des scores
displayScoresList(scores, mode) {
    const scoresList = this.elements.history.scoresList;
    if (!scoresList) return;
    
    if (scores.length === 0) {
        scoresList.innerHTML = `
            <div class="no-scores">
                <p>Aucun score enregistré pour ce mode.</p>
                <p>Jouez une partie pour voir vos résultats ici !</p>
            </div>
        `;
        return;
    }
    
    // Identifier le meilleur score
    const bestScore = Math.max(...scores.map(s => s.score));
    
    scoresList.innerHTML = '';
    
    scores.forEach((scoreData, index) => {
        const scoreItem = document.createElement('div');
        scoreItem.className = `score-item ${scoreData.score === bestScore ? 'best' : ''}`;
        
        const date = new Date(scoreData.date).toLocaleDateString('fr-FR', {
            day: '2-digit',
            month: '2-digit',
            year: 'numeric',
            hour: '2-digit',
            minute: '2-digit'
        });
        
        scoreItem.innerHTML = `
            <div class="score-info">
                <div class="score-value">${scoreData.score.toLocaleString()}</div>
                <div class="score-details">
                    ${scoreData.stats.correctAnswers}/${scoreData.stats.totalQuestions} 
                    (${scoreData.stats.successRate}%)
                    ${scoreData.score === bestScore ? '👑' : ''}
                </div>
            </div>
            <div class="score-date">${date}</div>
        `;
        
        scoresList.appendChild(scoreItem);
    });
},

// Afficher les statistiques globales d'historique
displayHistoryStats(mode) {
    const statsContainer = this.elements.history.stats;
    if (!statsContainer) return;
    
    const playerStats = StorageManager.getPlayerStats();
    const highScores = StorageManager.getHighScores();
    const modeScores = highScores[mode] || [];
    
    if (modeScores.length === 0) {
        statsContainer.innerHTML = '';
        return;
    }
    
    // Calculer les statistiques du mode
    const totalGames = playerStats.gamesPerMode[mode] || 0;
    const bestScore = Math.max(...modeScores.map(s => s.score));
    const averageScore = Math.round(modeScores.reduce((sum, s) => sum + s.score, 0) / modeScores.length);
    
    statsContainer.innerHTML = `
        <div class="mode-summary">
            <h4>📊 Résumé du mode ${this.getModeDisplayName(mode)}</h4>
            <div class="summary-grid">
                <div class="summary-item">
                    <span class="summary-value">${totalGames}</span>
                    <span class="summary-label">Parties jouées</span>
                </div>
                <div class="summary-item">
                    <span class="summary-value">${bestScore.toLocaleString()}</span>
                    <span class="summary-label">Meilleur score</span>
                </div>
                <div class="summary-item">
                    <span class="summary-value">${averageScore.toLocaleString()}</span>
                    <span class="summary-label">Score moyen</span>
                </div>
            </div>
        </div>
    `;
},

// Obtenir le nom d'affichage du mode
getModeDisplayName(mode) {
    const names = {
        classic: 'Classique',
        speed: 'Contre-la-montre',
        survival: 'Survie'
    };
    return names[mode] || mode;
}
```

### 5.4 Styles pour l'historique

Ajoutez ces styles complémentaires à `css/advanced.css` :

```css
/* Styles complémentaires pour l'historique */

.no-scores {
    text-align: center;
    padding: var(--spacing-xl);
    color: var(--text-dark);
    opacity: 0.7;
}

.no-scores p {
    margin-bottom: var(--spacing-sm);
}

.mode-summary {
    margin-top: var(--spacing-xl);
    padding: var(--spacing-lg);
    background: var(--light-bg);
    border-radius: var(--border-radius);
}

.mode-summary h4 {
    color: var(--primary-color);
    margin-bottom: var(--spacing-md);
    text-align: center;
}

.summary-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
    gap: var(--spacing-md);
}

.summary-item {
    text-align: center;
    padding: var(--spacing-md);
    background: var(--text-light);
    border-radius: var(--border-radius);
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.summary-value {
    display: block;
    font-size: var(--font-size-lg);
    font-weight: bold;
    color: var(--primary-color);
    margin-bottom: var(--spacing-xs);
}

.summary-label {
    font-size: var(--font-size-sm);
    color: var(--text-dark);
    opacity: 0.8;
}

/* Animation d'apparition pour l'historique */
.score-item {
    animation: scoreItemAppear 0.3s ease;
    animation-fill-mode: both;
}

.score-item:nth-child(1) { animation-delay: 0.1s; }
.score-item:nth-child(2) { animation-delay: 0.2s; }
.score-item:nth-child(3) { animation-delay: 0.3s; }
.score-item:nth-child(4) { animation-delay: 0.4s; }
.score-item:nth-child(5) { animation-delay: 0.5s; }

@keyframes scoreItemAppear {
    from {
        opacity: 0;
        transform: translateX(-20px);
    }
    to {
        opacity: 1;
        transform: translateX(0);
    }
}
```

---

## Étape 6 : Tests et finalisation

### 6.1 Tests systématiques

**Test 1 : Stockage persistant**
```javascript
// Dans la console du navigateur :
StorageManager.clearAll(); // Nettoyer
// Jouer une partie complète
// Recharger la page
// Vérifier que les scores sont conservés
```

**Test 2 : Fonctionnement des jokers**
```javascript
// Tester chaque joker individuellement :
// - 50/50 : vérifier l'élimination visuelle
// - Skip : vérifier le passage automatique
// - Temps+ : vérifier l'ajout de temps
// - Indice : vérifier l'affichage des informations
```

**Test 3 : Statistiques et historique**
1. Jouer plusieurs parties avec différents scores
2. Vérifier l'écran de fin avec toutes les statistiques
3. Consulter l'historique pour chaque mode
4. Vérifier les conseils personnalisés

### 6.2 Checklist de validation complète

**✅ Fonctionnalités de base :**
- [ ] Stockage et récupération des données
- [ ] Tous les jokers fonctionnent correctement
- [ ] Statistiques détaillées affichées
- [ ] Historique accessible et fonctionnel
- [ ] Conseils personnalisés pertinents

**✅ Interface utilisateur :**
- [ ] Animations fluides
- [ ] Feedback visuel approprié
- [ ] Responsive sur tous les écrans
- [ ] Navigation intuitive

**✅ Performance et robustesse :**
- [ ] Aucune erreur JavaScript
- [ ] Gestion des cas d'erreur
- [ ] Performance acceptable
- [ ] Compatibilité navigateurs

### 6.3 Améliorations finales optionnelles

**Système de thèmes** :
```javascript
// Ajoutez à main.js
const ThemeManager = {
    themes: {
        default: 'Style par défaut',
        dark: 'Mode sombre',
        colorful: 'Coloré'
    },
    
    apply(themeName) {
        document.body.className = `theme-${themeName}`;
        StorageManager.saveSettings({ theme: themeName });
    },
    
    init() {
        const settings = StorageManager.getSettings();
        this.apply(settings.theme || 'default');
    }
};
```

**Export des données** :
```javascript
// Fonction d'export des statistiques
function exportStats() {
    const data = {
        scores: StorageManager.getHighScores(),
        stats: StorageManager.getPlayerStats(),
        exportDate: new Date().toISOString()
    };
    
    const blob = new Blob([JSON.stringify(data, null, 2)], 
        { type: 'application/json' });
    const url = URL.createObjectURL(blob);
    
    const a = document.createElement('a');
    a.href = url;
    a.download = 'quiz-stats.json';
    a.click();
    
    URL.revokeObjectURL(url);
}
```
