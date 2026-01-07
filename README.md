<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>扫雷游戏</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        body {
            font-family: 'Microsoft YaHei', Arial, sans-serif;
            background: #000;
            color: #fff;
            padding: 10px;
            min-height: 100vh;
            background: linear-gradient(135deg, #1a1a1a 0%, #000 100%);
        }
        .container {
            max-width: 100%;
            margin: 0 auto;
        }
        .game-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 8px;
            padding: 10px 15px;
            background: #c0c0c0;
            border: 3px outset #c0c0c0;
            border-radius: 6px;
            box-shadow: 0 4px 12px rgba(0,0,0,0.3);
        }
        .mines-counter, .timer {
            font-family: 'Digital', monospace;
            background: #000;
            color: #f00;
            padding: 8px 12px;
            border-radius: 4px;
            font-size: 18px;
            font-weight: bold;
            min-width: 80px;
            text-align: center;
            border: 2px inset #c0c0c0;
        }
        .smiley-btn {
            width: 45px;
            height: 45px;
            font-size: 20px;
            border: 3px outset #c0c0c0;
            border-radius: 6px;
            background: #c0c0c0;
            cursor: pointer;
            display: flex;
            align-items: center;
            justify-content: center;
            transition: all 0.1s;
        }
        .smiley-btn:active {
            border: 3px inset #c0c0c0;
        }
        .game-area {
            margin: 0 auto;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        /* ОБНОВЛЁННОЕ ПОЛЕ - пиксельное как в оригинале */
        .field {
            display: inline-grid;
            gap: 1px;
            background: #7f7f7f; /* СЕРЫЙ фон как в Windows */
            padding: 8px;
            border: 4px outset #a0a0a0; /* Классическая 3D граница */
            margin: 0 auto;
        }
        .cell {
            width: 24px;
            height: 24px;
            display: flex;
            align-items: center;
            justify-content: center;
            background: #c0c0c0; /* СЕРЫЙ фон */
            border: 3px outset #efefef; /* 3D эффект выпуклой кнопки */
            font-weight: bold;
            cursor: pointer;
            user-select: none;
            font-size: 16px; /* УВЕЛИЧЕННЫЙ шрифт */
            font-family: 'Microsoft YaHei', 'Segoe UI', sans-serif;
        }
        .cell:active {
            border: 3px inset #c0c0c0; /* Вдавленный эффект при нажатии */
        }
        .cell.revealed {
            border: none; /* Плоская серая граница */
            background: #c0c0c0; /* Остаётся серым */
        }
        .cell.mine.revealed {
            background: #ff0000;
            border: 1px solid #ff0000;
        }
        .cell.flag::before {
            content: "旗";
            color: #f00;
            font-weight: bold;
        }
        .cell.highlight {
            background: #ffff99 !important;
        }
        .cell.highlight-flag {
            background: #ffcccc !important;
        }
        /* ХАОТИЧНЫЕ СЛУЧАЙНЫЕ ЦВЕТА ДЛЯ ЦИФР */
        .cell.number-1 { color: #FF6B35; } /* оранжевый */
        .cell.number-2 { color: #58508D; } /* фиолетовый */
        .cell.number-3 { color: #00A896; } /* бирюзовый */
        .cell.number-4 { color: #F4A261; } /* персиковый */
        .cell.number-5 { color: #2A9D8F; } /* зелёно-голубой */
        .cell.number-6 { color: #E76F51; } /* коралловый */
        .cell.number-7 { color: #264653; } /* тёмно-синий */
        .cell.number-8 { color: #E9C46A; } /* жёлтый */
        
        /* СТИЛИ БОССА */
        .boss-mode .field {
            background: #800020 !important; /* Бордовый фон поля */
            border-color: #660000 !important;
        }
        .boss-mode .cell {
            background: #a04040 !important; /* Бордовые клетки */
            border-color: #8b0000 !important;
        }
        .boss-mode .cell.revealed {
            background: #c08080 !important;
        }
        .boss-cell {
            background: #ff0000 !important;
            color: white !important;
            font-weight: bold;
            position: relative;
            z-index: 10;
        }
        .boss-cell::before {
            content: "雷";
            font-size: 14px;
        }
        .boss-flashing {
            animation: bossFlash 0.5s infinite;
        }
        @keyframes bossFlash {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.5; }
        }
        .game-over-screen {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0,0,0,0.9);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 1000;
            font-size: 24px;
        }
        .restart-btn {
            margin-top: 20px;
            padding: 15px 30px;
            background: #4CAF50;
            color: white;
            border: none;
            border-radius: 10px;
            font-size: 18px;
            cursor: pointer;
        }
        
        .game-controls {
            margin: 20px auto;
            padding: 15px;
            background: #333;
            border-radius: 10px;
            width: 100%;
            max-width: 400px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.3);
        }
        .control-group {
            margin: 10px 0;
            display: flex;
            align-items: center;
            justify-content: space-between;
        }
        .control-group label {
            color: #ccc;
            font-size: 14px;
            min-width: 80px;
        }
        .control-group input {
            background: #222;
            color: #fff;
            border: 1px solid #555;
            border-radius: 4px;
            padding: 6px;
            width: 60px;
            text-align: center;
        }
        .mode-buttons {
            display: flex;
            gap: 8px;
            margin: 10px 0;
        }
        .mode-btn {
            flex: 1;
            padding: 10px;
            background: #555;
            color: #fff;
            border: none;
            border-radius: 6px;
            cursor: pointer;
            font-size: 13px;
            transition: background 0.2s;
        }
        .mode-btn.active {
            background: #007bff;
            box-shadow: 0 2px 8px rgba(0,123,255,0.3);
        }
        .action-buttons {
            display: flex;
            gap: 10px;
            margin: 12px 0;
        }
        .action-btn {
            flex: 1;
            padding: 10px;
            background: #444;
            color: #fff;
            border: none;
            border-radius: 6px;
            cursor: pointer;
            transition: background 0.2s;
        }
        .action-btn:hover {
            background: #555;
        }
        .seed-input {
            background: #222;
            color: #fff;
            border: 1px solid #555;
            border-radius: 6px;
            padding: 8px;
            width: 100%;
            margin-top: 8px;
        }
        .game-info {
            margin-top: 12px;
            padding: 10px;
            background: #2a2a2a;
            border-radius: 6px;
            font-size: 12px;
            color: #ccc;
        }
        .seed-info {
            font-family: monospace;
            background: #222;
            padding: 6px;
            border-radius: 4px;
            word-break: break-all;
            margin-top: 6px;
            font-size: 11px;
        }
        .instructions {
            margin-top: 10px;
            padding: 8px;
            background: #2a2a2a;
            border-radius: 6px;
            font-size: 11px;
            color: #aaa;
            line-height: 1.4;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="game-header">
            <div class="mines-counter">
                雷: <span id="remainingMines">十</span>
            </div>
            <button class="smiley-btn" onclick="startNewGame()">🇨🇳</button>
            <div class="timer">
                时: <span id="gameTimer">〇</span>
            </div>
        </div>

        <div class="game-area">
            <div id="gameField"></div>
        </div>

        <div class="instructions">
            <strong>操作说明:</strong><br>
            • 点击模式: 左键翻开格子<br>
            • 标记模式: 左键放置/移除旗帜<br>
            • 快速点击: 点击数字同时按住左右键(或双击)快速打开周围
        </div>

        <div class="game-controls">
            <div class="control-group">
                <label>宽度:</label>
                <input type="number" id="width" value="9" min="5" max="15">
                
                <label>高度:</label>
                <input type="number" id="height" value="9" min="5" max="15">
            </div>
            
            <div class="control-group">
                <label>地雷数:</label>
                <input type="number" id="mines" value="10" min="1" max="30">
            </div>

            <div class="mode-buttons">
                <button class="mode-btn active" onclick="setMode('reveal')">点击模式</button>
                <button class="mode-btn" onclick="setMode('flag')">标记模式</button>
            </div>

            <div class="action-buttons">
                <button class="action-btn" onclick="startNewGame()">新游戏</button>
                <button class="action-btn" onclick="generateWithSeed()">种子游戏</button>
            </div>

            <input type="text" id="customSeed" class="seed-input" placeholder="输入种子号">

            <div class="game-info">
                <div>已开: <span id="revealedCells">〇</span>/<span id="totalCells">八十一</span></div>
                <div>种子: <span class="seed-info" id="currentSeed">-</span></div>
                <div>胜利: <span id="totalWins">〇</span> (BOSS: <span id="bossChance">0%</span>)</div>
            </div>
        </div>
    </div>

    <div id="gameOverScreen" class="game-over-screen" style="display: none;">
        <div id="gameOverText">游戏结束</div>
        <button class="restart-btn" onclick="restartGame()">重新开始</button>
    </div>

    <script>
        // Функция для преобразования арабских цифр в китайские
        function toChineseNumber(num) {
            const chineseNumbers = {
                0: '〇',
                1: '一',
                2: '二', 
                3: '三',
                4: '四',
                5: '五',
                6: '六',
                7: '七',
                8: '八',
                9: '九'
            };
            
            // ВСЕГДА просто комбинация цифр
            return num.toString().split('').map(digit => chineseNumbers[digit]).join('');
        }

        // ==================== СИСТЕМА БОССА ====================
        let totalWins = 0;
        let bossActive = false;
        let bossInstance = null;
        let bossCells = new Set();
        let bossMoving = false;
        let bossCheckTimeout = null;

        const BOSS_CHANCES = {
            5: 0.15,   // 15%
            10: 0.30,  // 30%
            20: 0.40,  // 40%
            30: 0.50,  // 50%
            40: 0.60,  // 60%
            50: 0.70   // 70%
        };

        function getBossChance() {
            let maxChance = 0;
            for (const [wins, chance] of Object.entries(BOSS_CHANCES)) {
                if (totalWins >= parseInt(wins) && chance > maxChance) {
                    maxChance = chance;
                }
            }
            return maxChance;
        }

        function updateBossInfo() {
            document.getElementById('totalWins').textContent = toChineseNumber(totalWins);
            document.getElementById('bossChance').textContent = Math.round(getBossChance() * 100) + '%';
        }

        function checkBossSpawn() {
            if (bossActive || totalWins < 5) return false;
            return Math.random() < getBossChance();
        }

        function activateBossMode() {
            bossActive = true;
            document.body.classList.add('boss-mode');
            
            // Устанавливаем параметры босса
            document.getElementById('width').value = 30;
            document.getElementById('height').value = 30;
            document.getElementById('mines').value = 300;
            
            // Случайный seed для босса
            const bossSeed = Math.floor(Math.random() * 1000000000);
            document.getElementById('customSeed').value = bossSeed;
            
            // Запускаем игру
            generateWithSeed();
            
            // Создаём босса через секунду
            setTimeout(createBoss, 1000);
        }

        function createBoss() {
            bossCells.clear();
            const centerX = 15;
            const centerY = 15;
            
            // Создаём куб 9x9
            for (let y = centerY - 4; y <= centerY + 4; y++) {
                for (let x = centerX - 4; x <= centerX + 4; x++) {
                    if (x >= 0 && x < 30 && y >= 0 && y < 30) {
                        bossCells.add(`${x},${y}`);
                    }
                }
            }
            
            // Помечаем клетки босса
            updateBossDisplay();
            
            // Запускаем движение босса
            startBossMovement();
        }

        function updateBossDisplay() {
            const cells = document.querySelectorAll('.cell');
            cells.forEach(cell => {
                const x = parseInt(cell.dataset.x);
                const y = parseInt(cell.dataset.y);
                const key = `${x},${y}`;
                
                if (bossCells.has(key)) {
                    cell.classList.add('boss-cell');
                    cell.classList.add('boss-flashing');
                } else {
                    cell.classList.remove('boss-cell');
                    cell.classList.remove('boss-flashing');
                }
            });
        }

        function startBossMovement() {
            if (!bossActive) return;
            
            bossMoving = true;
            moveBoss();
            
            // Движение каждые 3 секунды
            setInterval(() => {
                if (bossActive && bossMoving) {
                    moveBoss();
                }
            }, 3000);
        }

        function moveBoss() {
            // Случайное движение
            const dx = Math.floor(Math.random() * 61) - 30; // -30 до +30
            const dy = Math.floor(Math.random() * 61) - 30;
            
            const newBossCells = new Set();
            bossCells.forEach(cellKey => {
                const [x, y] = cellKey.split(',').map(Number);
                let newX = x + dx;
                let newY = y + dy;
                
                // Ограничиваем границами поля
                newX = Math.max(0, Math.min(29, newX));
                newY = Math.max(0, Math.min(29, newY));
                
                newBossCells.add(`${newX},${newY}`);
            });
            
            bossCells = newBossCells;
            updateBossDisplay();
            
            // Стираем флажки под боссом
            clearFlagsUnderBoss();
        }

        function clearFlagsUnderBoss() {
            bossCells.forEach(cellKey => {
                game.flaggedPositions.delete(cellKey);
            });
            displayGame();
        }

        function handleBossClick(x, y) {
            if (!bossActive || bossCheckTimeout) return;
            
            const key = `${x},${y}`;
            if (!bossCells.has(key)) return;
            
            // Начинаем проверку
            bossCheckTimeout = setTimeout(() => {
                checkBossVictory();
            }, 3500);
            
            // Мигание босса
            const bossCellsElements = document.querySelectorAll('.boss-cell');
            let flashCount = 0;
            const flashInterval = setInterval(() => {
                bossCellsElements.forEach(cell => {
                    cell.style.backgroundColor = flashCount % 2 === 0 ? '#8b0000' : '#ff0000';
                });
                flashCount++;
                if (flashCount > 6) {
                    clearInterval(flashInterval);
                }
            }, 500);
        }

        function checkBossVictory() {
            bossCheckTimeout = null;
            
            // Проверяем что все клетки - мины
            let allCellsAreMines = true;
            for (let y = 0; y < 30; y++) {
                for (let x = 0; x < 30; x++) {
                    const key = `${x},${y}`;
                    const cellState = game.cellStates[key];
                    
                    if (!cellState || cellState !== 'revealed') {
                        // Клетка закрыта - должна быть миной
                        if (!bossCells.has(key)) {
                            allCellsAreMines = false;
                            break;
                        }
                    }
                }
                if (!allCellsAreMines) break;
            }
            
            if (allCellsAreMines) {
                // ПОБЕДА - босс падает
                bossVictory();
            } else {
                // ПОРАЖЕНИЕ - взрыв
                bossDefeat();
            }
        }

        function bossVictory() {
            bossActive = false;
            bossMoving = false;
            
            // Анимация падения босса
            const bossCellsElements = document.querySelectorAll('.boss-cell');
            let fallDistance = 0;
            
            const fallInterval = setInterval(() => {
                fallDistance += 5;
                bossCellsElements.forEach(cell => {
                    cell.style.transform = `translateY(${fallDistance}px)`;
                    cell.style.opacity = 1 - (fallDistance / 100);
                });
                
                if (fallDistance > 100) {
                    clearInterval(fallInterval);
                    showGameOver(true);
                }
            }, 50);
        }

        function bossDefeat() {
            bossActive = false;
            bossMoving = false;
            
            // Анимация взрыва
            const bossCellsElements = document.querySelectorAll('.boss-cell');
            bossCellsElements.forEach(cell => {
                cell.style.backgroundColor = '#ff0000';
                cell.style.transform = 'scale(1.5)';
                cell.style.transition = 'all 0.3s';
            });
            
            setTimeout(() => {
                showGameOver(false);
            }, 1000);
        }

        function showGameOver(isVictory) {
            const screen = document.getElementById('gameOverScreen');
            const text = document.getElementById('gameOverText');
            
            text.textContent = isVictory ? '恭喜！你赢了！' : '游戏结束！你输了！';
            screen.style.display = 'flex';
        }

        function restartGame() {
            document.getElementById('gameOverScreen').style.display = 'none';
            document.body.classList.remove('boss-mode');
            bossActive = false;
            totalWins = 0;
            updateBossInfo();
            startNewGame();
        }
        // ==================== КОНЕЦ СИСТЕМЫ БОССА ====================

        class MinesweeperGame {
            constructor(width = 9, height = 9, minesCount = 10) {
                this.width = width;
                this.height = height;
                this.minesCount = minesCount;
                this.seed = null;
                this.gameState = 'playing';
                this.revealedCount = 0;
                this.flaggedPositions = new Set();
                this.firstClick = true;
                this.startTime = null;
                this.timer = null;
                this.currentMode = 'reveal';
                this.cellStates = {};
                this.highlightedCells = new Set();
                this.isChordPressed = false;
            }
            
            generateGame(seed = null) {
                if (seed) {
                    this.seed = seed;
                } else {
                    this.seed = this.generateUniqueSeed();
                }
                
                this.setSeed(this.seed);
                this.gameState = 'playing';
                this.revealedCount = 0;
                this.flaggedPositions.clear();
                this.firstClick = true;
                this.cellStates = {};
                this.highlightedCells.clear();
                this.isChordPressed = false;
                this.stopTimer();
                this.startTime = null;
                
                const field = Array(this.height).fill().map(() => 
                    Array(this.width).fill(0)
                );
                
                return {
                    field: field,
                    seed: this.seed,
                    width: this.width,
                    height: this.height
                };
            }
            
            generateUniqueSeed() {
                return Date.now() + Math.floor(Math.random() * 1000000);
            }
            
            setSeed(seed) {
                let x = Math.sin(seed) * 10000;
                this.random = () => {
                    x = Math.sin(x * 10000) * 10000;
                    return x - Math.floor(x);
                };
            }
            
            placeMines(firstX, firstY, field) {
                const minesPositions = [];
                const totalCells = this.width * this.height;
                
                if (this.minesCount >= totalCells) {
                    this.minesCount = totalCells - 1;
                }
                
                while (minesPositions.length < this.minesCount) {
                    const x = Math.floor(this.random() * this.width);
                    const y = Math.floor(this.random() * this.height);
                    
                    if ((x === firstX && y === firstY) || 
                        Math.abs(x - firstX) <= 1 && Math.abs(y - firstY) <= 1) {
                        continue;
                    }
                    
                    const position = `${x},${y}`;
                    if (!minesPositions.includes(position)) {
                        minesPositions.push(position);
                        field[y][x] = -1;
                    }
                }
                
                this.updateNumbers(field, minesPositions);
                return minesPositions;
            }
            
            updateNumbers(field, minesPositions) {
                minesPositions.forEach(pos => {
                    const [x, y] = pos.split(',').map(Number);
                    
                    for (let dx = -1; dx <= 1; dx++) {
                        for (let dy = -1; dy <= 1; dy++) {
                            if (dx === 0 && dy === 0) continue;
                            
                            const nx = x + dx;
                            const ny = y + dy;
                            
                            if (nx >= 0 && nx < this.width && 
                                ny >= 0 && ny < this.height && 
                                field[ny][nx] !== -1) {
                                field[ny][nx]++;
                            }
                        }
                    }
                });
            }
            
            revealCell(x, y, field) {
                if (this.gameState !== 'playing') return false;
                if (this.flaggedPositions.has(`${x},${y}`)) return false;
                
                if (this.firstClick) {
                    this.placeMines(x, y, field);
                    this.firstClick = false;
                    this.startTimer();
                }
                
                if (field[y][x] === -1) {
                    this.gameState = 'gameover';
                    this.stopTimer();
                    return true;
                }
                
                if (!this.cellStates[`${x},${y}`]) {
                    this.cellStates[`${x},${y}`] = 'revealed';
                    this.revealedCount++;
                    
                    if (field[y][x] === 0) {
                        this.revealEmptyArea(x, y, field);
                    }
                }
                
                this.checkWinCondition(field);
                return true;
            }
            
            revealEmptyArea(x, y, field) {
                const queue = [[x, y]];
                const visited = new Set();
                
                while (queue.length > 0) {
                    const [cx, cy] = queue.shift();
                    const key = `${cx},${cy}`;
                    
                    if (visited.has(key)) continue;
                    if (this.flaggedPositions.has(key)) continue;
                    
                    visited.add(key);
                    
                    if (!this.cellStates[key]) {
                        this.cellStates[key] = 'revealed';
                        this.revealedCount++;
                    }
                    
                    if (field[cy][cx] === 0) {
                        for (let dx = -1; dx <= 1; dx++) {
                            for (let dy = -1; dy <= 1; dy++) {
                                const nx = cx + dx;
                                const ny = cy + dy;
                                
                                if (nx >= 0 && nx < this.width && 
                                    ny >= 0 && ny < this.height && 
                                    !visited.has(`${nx},${ny}`)) {
                                    queue.push([nx, ny]);
                                }
                            }
                        }
                    }
                }
            }
            
            handleCellClick(x, y) {
                if (this.gameState !== 'playing') return;
                
                const key = `${x},${y}`;
                
                // Проверяем клик на босса
                if (bossActive && bossCells.has(key)) {
                    handleBossClick(x, y);
                    return;
                }
                
                if (this.currentMode === 'reveal') {
                    if (!this.flaggedPositions.has(key)) {
                        this.revealCell(x, y, gameData.field);
                    }
                } else if (this.currentMode === 'flag') {
                    this.toggleFlag(x, y);
                }
            }
            
            toggleFlag(x, y) {
                const key = `${x},${y}`;
                
                if (this.cellStates[key] === 'revealed') return;
                
                if (this.flaggedPositions.has(key)) {
                    this.flaggedPositions.delete(key);
                } else {
                    this.flaggedPositions.add(key);
                }
            }
            
            chordClick(x, y) {
                if (this.gameState !== 'playing') return;
                if (this.cellStates[`${x},${y}`] !== 'revealed') return;
                if (gameData.field[y][x] <= 0) return;
                
                const cellValue = gameData.field[y][x];
                let flagCount = 0;
                const cellsToReveal = [];
                
                for (let dx = -1; dx <= 1; dx++) {
                    for (let dy = -1; dy <= 1; dy++) {
                        if (dx === 0 && dy === 0) continue;
                        
                        const nx = x + dx;
                        const ny = y + dy;
                        
                        if (nx >= 0 && nx < this.width && ny >= 0 && ny < this.height) {
                            const neighborKey = `${nx},${ny}`;
                            if (this.flaggedPositions.has(neighborKey)) {
                                flagCount++;
                            } else if (!this.cellStates[neighborKey]) {
                                cellsToReveal.push([nx, ny]);
                            }
                        }
                    }
                }
                
                if (flagCount === cellValue) {
                    let hitMine = false;
                    
                    for (const [nx, ny] of cellsToReveal) {
                        if (gameData.field[ny][nx] === -1) {
                            hitMine = true;
                        }
                        this.revealCell(nx, ny, gameData.field);
                    }
                    
                    if (hitMine) {
                        this.gameState = 'gameover';
                        this.stopTimer();
                    }
                }
            }
            
            highlightChordArea(x, y) {
                this.highlightedCells.clear();
                
                if (this.cellStates[`${x},${y}`] === 'revealed' && gameData.field[y][x] > 0) {
                    for (let dx = -1; dx <= 1; dx++) {
                        for (let dy = -1; dy <= 1; dy++) {
                            if (dx === 0 && dy === 0) continue;
                            
                            const nx = x + dx;
                            const ny = y + dy;
                            
                            if (nx >= 0 && nx < this.width && 
                                ny >= 0 && ny < this.height) {
                                const neighborKey = `${nx},${ny}`;
                                
                                if (!this.cellStates[neighborKey]) {
                                    if (this.flaggedPositions.has(neighborKey)) {
                                        this.highlightedCells.add(neighborKey + '-flag');
                                    } else {
                                        this.highlightedCells.add(neighborKey);
                                    }
                                }
                            }
                        }
                    }
                }
            }
            
            clearHighlight() {
                this.highlightedCells.clear();
            }
            
            checkWinCondition(field) {
                const totalSafeCells = this.width * this.height - this.minesCount;
                if (this.revealedCount === totalSafeCells) {
                    this.gameState = 'win';
                    this.stopTimer();
                    
                    // Увеличиваем счётчик побед
                    if (!bossActive) {
                        totalWins++;
                        updateBossInfo();
                        
                        // Проверяем появление босса
                        if (checkBossSpawn()) {
                            setTimeout(() => {
                                if (confirm(`你赢了 ${totalWins} 次！BOSS 出现了！准备战斗吗？`)) {
                                    activateBossMode();
                                }
                            }, 500);
                        }
                    }
                }
            }
            
            getRemainingMines() {
                return this.minesCount - this.flaggedPositions.size;
            }
            
            startTimer() {
                this.startTime = Date.now();
                this.timer = setInterval(updateTimer, 1000);
            }
            
            stopTimer() {
                if (this.timer) {
                    clearInterval(this.timer);
                    this.timer = null;
                }
            }
            
            getElapsedTime() {
                if (!this.startTime) return 0;
                return Math.floor((Date.now() - this.startTime) / 1000);
            }
            
            setMode(mode) {
                this.currentMode = mode;
            }
        }

        let currentGame = null;
        let gameData = null;
        const game = new MinesweeperGame();
        let chordActive = false;

        function startNewGame() {
            const width = parseInt(document.getElementById('width').value) || 9;
            const height = parseInt(document.getElementById('height').value) || 9;
            const mines = parseInt(document.getElementById('mines').value) || 10;
            
            game.width = width;
            game.height = height;
            game.minesCount = mines;
            
            gameData = game.generateGame();
            displayGame();
            updateBossInfo();
        }

        function generateWithSeed() {
            const seedInput = document.getElementById('customSeed').value;
            if (!seedInput) {
                alert('请输入种子号');
                return;
            }
            
            const width = parseInt(document.getElementById('width').value) || 9;
            const height = parseInt(document.getElementById('height').value) || 9;
            const mines = parseInt(document.getElementById('mines').value) || 10;
            
            game.width = width;
            game.height = height;
            game.minesCount = mines;
            
            const seed = parseInt(seedInput) || seedInput.hashCode();
            gameData = game.generateGame(seed);
            displayGame();
            updateBossInfo();
        }

        function setMode(mode) {
            game.setMode(mode);
            document.querySelectorAll('.mode-btn').forEach(btn => {
                btn.classList.remove('active');
            });
            event.target.classList.add('active');
        }

        function updateTimer() {
            if (game.gameState === 'playing' && game.startTime) {
                document.getElementById('gameTimer').textContent = toChineseNumber(game.getElapsedTime());
            }
        }

        function handleCellClick(x, y) {
            game.handleCellClick(x, y);
            displayGame();
        }

        function handleChordStart(x, y) {
            if (game.cellStates[`${x},${y}`] === 'revealed' && gameData.field[y][x] > 0) {
                chordActive = true;
                game.highlightChordArea(x, y);
                displayGame();
            }
        }

        function handleChordEnd(x, y) {
            if (chordActive) {
                chordActive = false;
                game.chordClick(x, y);
                game.clearHighlight();
                displayGame();
            }
        }

        function displayGame() {
            const gameField = document.getElementById('gameField');
            const currentSeed = document.getElementById('currentSeed');
            const totalCells = document.getElementById('totalCells');
            const remainingMines = document.getElementById('remainingMines');
            const revealedCells = document.getElementById('revealedCells');
            const gameTimer = document.getElementById('gameTimer');
            
            currentSeed.textContent = gameData.seed;
            totalCells.textContent = toChineseNumber(game.width * game.height);
            remainingMines.textContent = toChineseNumber(game.getRemainingMines());
            revealedCells.textContent = toChineseNumber(game.revealedCount);
            gameTimer.textContent = toChineseNumber(game.getElapsedTime());
            
            const fieldElement = document.createElement('div');
            fieldElement.className = 'field';
            fieldElement.style.gridTemplateColumns = `repeat(${game.width}, 24px)`;
            
            for (let y = 0; y < game.height; y++) {
                for (let x = 0; x < game.width; x++) {
                    const cell = document.createElement('div');
                    cell.className = 'cell';
                    cell.dataset.x = x;
                    cell.dataset.y = y;
                    
                    const key = `${x},${y}`;
                    const value = gameData.field[y][x];
                    const cellState = game.cellStates[key];
                    
                    if (cellState === 'revealed') {
                        cell.classList.add('revealed');
                        if (value === -1) {
                            cell.classList.add('mine');
                            cell.textContent = '雷';
                        } else if (value > 0) {
                            cell.classList.add(`number-${value}`);
                            cell.textContent = toChineseNumber(value);
                        }
                    } else {
                        if (game.flaggedPositions.has(key)) {
                            cell.classList.add('flag');
                        }
                    }
                    
                    if (game.highlightedCells.has(key)) {
                        cell.classList.add('highlight');
                    } else if (game.highlighted
