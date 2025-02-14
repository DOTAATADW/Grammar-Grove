<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>森林语法棋</title>
    <style>
        :root {
            --forest-green: #C8D5B9;
            --leaf-green: #E9F3D7;
            --earth-brown: #DAD2C3;
            --accent-moss: #A4BA8A;
            --pause: #FCE588;
            --verb: #B8DFD8;
            --read: #E4C1F9;
            --meaning: #FFB3BA;
            --sentence: #A4C2F4;
            --reward: #C7E5C0;
        }

        body {
            margin: 0;
            padding: 15px;
            background: var(--forest-green);
            font-family: 'Segoe UI', sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            min-height: 100vh;
        }

        .header {
            width: 100%;
            max-width: 500px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 15px;
        }

        h1 {
            color: #4A6B3D;
            margin: 0;
            font-size: 1.5rem;
        }

        .board {
            display: grid;
            grid-template-columns: repeat(10, 1fr);
            gap: 2px;
            width: 100%;
            max-width: 500px;
            aspect-ratio: 1/1;
            background: var(--earth-brown);
            padding: 5px;
            border-radius: 8px;
            position: relative;
        }

        .cell {
            background: var(--leaf-green);
            aspect-ratio: 1;
            border-radius: 3px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 12px;
            position: relative;
            transition: transform 0.2s;
        }

        .pause { background: var(--pause); }
        .verb { background: var(--verb); }
        .read { background: var(--read); }
        .meaning { background: var(--meaning); }
        .sentence { background: var(--sentence); }
        .reward { background: var(--reward); }

        .player {
            width: 14px;
            height: 14px;
            border: 2px solid white;
            border-radius: 3px;
            position: absolute;
            transition: all 0.3s ease;
            pointer-events: none;
            transform: translate(-50%, -50%);
        }
        #player1 { background: #E74C3C; }
        #player2 { background: #3498DB; }

        .legend {
            width: 100%;
            max-width: 500px;
            margin: 15px 0;
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 8px;
            padding: 10px;
            background: #ffffff30;
            border-radius: 8px;
        }
        .legend-item {
            display: flex;
            align-items: center;
            gap: 5px;
            font-size: 12px;
        }
        .legend-color {
            width: 16px;
            height: 16px;
            border-radius: 3px;
        }

        .controls {
            margin-top: 15px;
            width: 100%;
            max-width: 500px;
            display: grid;
            grid-template-columns: 60px 1fr;
            gap: 10px;
        }

        .dice {
            width: 60px;
            height: 60px;
            background: var(--accent-moss);
            border-radius: 8px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 24px;
            color: white;
            cursor: pointer;
            user-select: none;
        }

        .score-panel {
            background: var(--earth-brown);
            padding: 10px;
            border-radius: 8px;
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 8px;
        }

        .core-btn {
            width: 28px;
            height: 28px;
            border: none;
            border-radius: 4px;
            background: var(--accent-moss);
            color: white;
            font-size: 16px;
            transition: transform 0.1s;
        }

        @keyframes dice-shake {
            0% { transform: rotate(0deg) scale(1); }
            25% { transform: rotate(-10deg) scale(1.1); }
            50% { transform: rotate(10deg) scale(0.9); }
            75% { transform: rotate(-5deg); }
            100% { transform: rotate(0deg) scale(1); }
        }

        @media (max-width: 480px) {
            .cell { font-size: 10px; }
            .dice { width: 50px; height: 50px; font-size: 20px; }
            .score-btn { width: 24px; height: 24px; }
            .legend { grid-template-columns: repeat(2, 1fr); }
        }
    </style>
</head>
<body>
    <div class="header">
        <h1>🌲 森林语法棋</h1>
        <button class="score-btn" onclick="resetGame()">↻</button>
    </div>

    <div class="board" id="board"></div>

    <div class="legend">
        <div class="legend-item"><div class="legend-color pause"></div>暂停</div>
        <div class="legend-item"><div class="legend-color verb"></div>动词填空</div>
        <div class="legend-item"><div class="legend-color read"></div>朗读单词</div>
        <div class="legend-item"><div class="legend-color meaning"></div>说出含义</div>
        <div class="legend-item"><div class="legend-color sentence"></div>重组句子</div>
        <div class="legend-item"><div class="legend-color reward"></div>奖励加分</div>
    </div>

    <div class="controls">
        <div class="dice" onclick="rollDice()">?</div>
        <div class="score-panel">
            <div style="display: flex; gap:6px; align-items:center">
                <span>玩家①:</span>
                <button class="score-btn" onclick="updateScore(1, -1)">-</button>
                <span id="score1">0</span>
                <button class="score-btn" onclick="updateScore(1, 1)">+</button>
            </div>
            <div style="display: flex; gap:6px; align-items:center">
                <span>玩家②:</span>
                <button class="score-btn" onclick="updateScore(2, -1)">-</button>
                <span id="score2">0</span>
                <button class="score-btn" onclick="updateScore(2, 1)">+</button>
            </div>
        </div>
    </div>

    <script>
        // 游戏状态管理器
        const gameState = {
            currentPlayer: 1,
            players: {
                1: { pos: 0, score: 0 },
                2: { pos: 0, score: 0 }
            },
            specialCells: generateSpecialCells(),
            diceLock: false
        };

        //===== 核心功能 =====//
        function updateScore(player, delta) {
            if (!gameState.players[player]) return;
            
            gameState.players[player].score = Math.max(
                0, 
                gameState.players[player].score + delta
            );
            
            document.getElementById(`score${player}`).textContent = 
                gameState.players[player].score;
        }

        function resetGame() {
            gameState.players[1] = { pos:0, score:0 };
            gameState.players[2] = { pos:0, score:0 };
            gameState.specialCells = generateSpecialCells();
            gameState.currentPlayer = 1;
            
            updateScores();
            initBoard();
            document.querySelector('.dice').textContent = '?';
        }

        //===== 辅助函数 =====//
        function generateSpecialCells() {
            const cells = new Array(100).fill(null);
            const typeRules = [
                {type:'pause',count:4}, {type:'verb',count:5},
                {type:'read',count:5}, {type:'meaning',count:5},
                {type:'sentence',count:5}, {type:'reward',count:6}
            ];

            typeRules.forEach(({type, count}) => {
                let placed = 0;
                while(placed < count) {
                    const pos = Math.floor(Math.random() * 99) + 1;
                    if(!cells[pos]) {
                        cells[pos] = type;
                        placed++;
                    }
                }
            });
            return cells;
        }

        function initBoard() {
            const board = document.getElementById('board');
            board.innerHTML = '';
            
            for(let i=0; i<100; i++){
                const cell = document.createElement('div');
                cell.className = `cell${gameState.specialCells[i] ? ' '+gameState.specialCells[i] : ''}`;
                cell.textContent = i+1;
                board.appendChild(cell);
            }

            ['player1', 'player2'].forEach(id => {
                const player = document.createElement('div');
                player.id = id;
                player.className = 'player';
                board.appendChild(player);
            });

            updatePositions();
        }

        function updatePositions() {
            [1, 2].forEach(playerId => {
                const {pos} = gameState.players[playerId];
                const cell = board.children[pos];
                const playerElement = document.getElementById(`player${playerId}`);
                
                if(cell) {
                    const {left, top, width, height} = cell.getBoundingClientRect();
                    const boardRect = board.getBoundingClientRect();
                    
                    playerElement.style.left = `${left - boardRect.left + width/2}px`;
                    playerElement.style.top = `${top - boardRect.top + height/2}px`;
                }
            });
        }

        //===== 游戏逻辑 =====//
        function rollDice() {
            if(gameState.diceLock) return;
            gameState.diceLock = true;

            const dice = document.querySelector('.dice');
            dice.style.animation = 'dice-shake 0.8s';

            setTimeout(() => {
                const steps = Math.floor(Math.random()*6)+1;
                dice.textContent = steps;
                movePlayer(steps);
                gameState.diceLock = false;
            }, 800);
        }

        function movePlayer(steps) {
            const player = gameState.players[gameState.currentPlayer];
            player.pos = Math.min(player.pos + steps, 99);
            updatePositions();
            triggerCellEffect();
            gameState.currentPlayer = gameState.currentPlayer === 1 ? 2 : 1;
        }

        function triggerCellEffect() {
            const cell = board.children[gameState.players[gameState.currentPlayer].pos];
            const cellType = cell.className.replace('cell ', '');

            const effectMap = {
                pause: () => {
                    alert('🚫 暂停一轮！');
                    gameState.currentPlayer = gameState.currentPlayer === 1 ? 2 : 1;
                },
                verb: () => handleChallenge('动词填空'),
                read: () => handleChallenge('朗读单词'),
                meaning: () => handleChallenge('说出含义'),
                sentence: () => handleChallenge('重组句子'),
                reward: () => {
                    gameState.players[gameState.currentPlayer].score += 3;
                    alert('🎉 获得3分奖励！');
                }
            };

            if(effectMap[cellType]) effectMap[cellType]();
            updateScores();
        }

        function handleChallenge(type) {
            const result = confirm(`${type}挑战\n回答正确请点击确定，错误请点取消`);
            const delta = result ? 2 : -1;
            updateScore(gameState.currentPlayer, delta);
        }

        function updateScores() {
            document.getElementById('score1').textContent = gameState.players[1].score;
            document.getElementById('score2').textContent = gameState.players[2].score;
        }

        //===== 初始化 =====//
        document.addEventListener('DOMContentLoaded', () => {
            initBoard();
            // 添加移动端支持
            document.querySelectorAll('.score-btn').forEach(btn => {
                btn.addEventListener('touchend', function(e) {
                    e.preventDefault();
                    this.click();
                });
            });
        });
        window.addEventListener('resize', updatePositions);
    </script>
</body>
</html>
