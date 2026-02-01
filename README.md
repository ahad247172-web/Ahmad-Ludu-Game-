<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Classic Ludo - 4 Players</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            touch-action: manipulation;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            display: flex;
            flex-direction: column;
            align-items: center;
            overflow: hidden;
        }

        .game-container {
            width: 100%;
            max-width: 600px;
            padding: 10px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        .board-container {
            position: relative;
            width: 100%;
            max-width: 500px;
            aspect-ratio: 1;
            background: white;
            border-radius: 10px;
            box-shadow: 0 20px 60px rgba(0,0,0,0.3);
            overflow: hidden;
        }

        #ludoBoard {
            width: 100%;
            height: 100%;
            display: block;
        }

        .controls {
            margin-top: 20px;
            display: flex;
            gap: 15px;
            flex-wrap: wrap;
            justify-content: center;
            width: 100%;
        }

        .dice-container {
            width: 80px;
            height: 80px;
            background: white;
            border-radius: 15px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 40px;
            font-weight: bold;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
            cursor: pointer;
            transition: transform 0.2s;
            user-select: none;
        }

        .dice-container:active {
            transform: scale(0.95);
        }

        .dice-container.rolling {
            animation: roll 0.5s ease-in-out;
        }

        @keyframes roll {
            0%, 100% { transform: rotate(0deg); }
            25% { transform: rotate(90deg); }
            50% { transform: rotate(180deg); }
            75% { transform: rotate(270deg); }
        }

        .player-info {
            display: flex;
            gap: 10px;
            flex-wrap: wrap;
            justify-content: center;
        }

        .player-card {
            padding: 10px 20px;
            border-radius: 25px;
            color: white;
            font-weight: bold;
            opacity: 0.5;
            transition: all 0.3s;
            box-shadow: 0 2px 4px rgba(0,0,0,0.2);
        }

        .player-card.active {
            opacity: 1;
            transform: scale(1.1);
            box-shadow: 0 4px 12px rgba(0,0,0,0.3);
        }

        .player-card.red { background: #e74c3c; }
        .player-card.blue { background: #3498db; }
        .player-card.green { background: #27ae60; }
        .player-card.yellow { background: #f39c12; }

        button {
            padding: 15px 30px;
            font-size: 18px;
            border: none;
            border-radius: 25px;
            background: #2c3e50;
            color: white;
            cursor: pointer;
            font-weight: bold;
            transition: all 0.3s;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
        }

        button:hover:not(:disabled) {
            transform: translateY(-2px);
            box-shadow: 0 6px 12px rgba(0,0,0,0.2);
        }

        button:disabled {
            opacity: 0.5;
            cursor: not-allowed;
        }

        .status {
            margin-top: 10px;
            color: white;
            font-size: 18px;
            text-align: center;
            text-shadow: 0 2px 4px rgba(0,0,0,0.3);
            min-height: 30px;
        }

        .winner-modal {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0,0,0,0.8);
            display: none;
            align-items: center;
            justify-content: center;
            z-index: 1000;
        }

        .winner-content {
            background: white;
            padding: 40px;
            border-radius: 20px;
            text-align: center;
            animation: popIn 0.5s ease-out;
        }

        @keyframes popIn {
            0% { transform: scale(0); }
            80% { transform: scale(1.1); }
            100% { transform: scale(1); }
        }

        .winner-content h2 {
            font-size: 36px;
            margin-bottom: 20px;
            color: #2c3e50;
        }

        @media (max-width: 480px) {
            .board-container {
                max-width: 95vw;
            }
            .player-card {
                padding: 8px 15px;
                font-size: 14px;
            }
        }
    </style>
</head>
<body>

    <div class="game-container">
        <div class="board-container">
            <canvas id="ludoBoard"></canvas>
        </div>

        <div class="status" id="status">Red's Turn - Roll the dice!</div>

        <div class="controls">
            <div class="dice-container" id="dice">🎲</div>
            <button id="rollBtn" onclick="rollDice()">ROLL DICE</button>
            <button onclick="resetGame()">NEW GAME</button>
        </div>

        <div class="player-info">
            <div class="player-card red active" id="info-red">Red (0/4)</div>
            <div class="player-card blue" id="info-blue">Blue (0/4)</div>
            <div class="player-card green" id="info-green">Green (0/4)</div>
            <div class="player-card yellow" id="info-yellow">Yellow (0/4)</div>
        </div>
    </div>

    <div class="winner-modal" id="winnerModal">
        <div class="winner-content">
            <h2 id="winnerText">Red Wins!</h2>
            <button onclick="resetGame()">Play Again</button>
        </div>
    </div>

    <script>
        const canvas = document.getElementById('ludoBoard');
        const ctx = canvas.getContext('2d');
        
        // Set canvas size
        function resizeCanvas() {
            const container = canvas.parentElement;
            canvas.width = container.clientWidth;
            canvas.height = container.clientHeight;
            drawBoard();
        }
        window.addEventListener('resize', resizeCanvas);

        // Game State
        const players = ['red', 'blue', 'green', 'yellow'];
        const colors = {
            red: '#e74c3c',
            blue: '#3498db',
            green: '#27ae60',
            yellow: '#f39c12'
        };
        
        let gameState = {
            currentPlayer: 0,
            dice: 0,
            rolling: false,
            selectedToken: null,
            positions: {
                red: [0, 0, 0, 0], // 0 = home, 1-56 = path, 57 = finished
                blue: [0, 0, 0, 0],
                green: [0, 0, 0, 0],
                yellow: [0, 0, 0, 0]
            },
            finished: {
                red: 0, blue: 0, green: 0, yellow: 0
            }
        };

        // Ludo path coordinates (15x15 grid)
        const PATH = [];
        
        // Generate standard Ludo path
        function generatePath() {
            // Red start -> Blue start -> Yellow start -> Green start -> Red start
            const routes = [
                // Red path (bottom left area moving right then up)
                {start: [6, 13], dir: [0, -1], steps: 5},
                {start: [6, 8], dir: [1, 0], steps: 6},
                {start: [12, 8], dir: [0, -1], steps: 6},
                {start: [12, 2], dir: [-1, 0], steps: 6},
                {start: [6, 2], dir: [0, -1], steps: 6},
                {start: [6, -4], dir: [-1, 0], steps: 6},
                {start: [0, 2], dir: [0, 1], steps: 6},
                {start: [0, 8], dir: [1, 0], steps: 6},
                {start: [6, 8], dir: [0, 1], steps: 5} // Back to red area
            ];
            
            // Create standard 52-step path
            let x = 1, y = 6; // Start after red home
            
            // Standard Ludo track generation
            for (let i = 0; i < 52; i++) {
                PATH.push({x, y});
                
                // Logic for standard Ludo track
                if (y === 6 && x < 6) x++;
                else if (x === 6 && y > 0 && y < 6) y--;
                else if (x === 6 && y === 0) x++;
                else if (x > 6 && x < 12 && y === 0) x++;
                else if (x === 12 && y < 6) y++;
                else if (x === 12 && y === 6) y++;
                else if (x > 6 && y === 12) x--;
                else if (x === 6 && y > 6) y++;
                else if (x === 6 && y === 12) x--;
                else if (x < 6 && y === 12) x--;
                else if (x === 0 && y > 6) y--;
                else if (x === 0 && y === 6) x++;
            }
        }
        generatePath();

        // Home positions for each player
        const HOMES = {
            red: [[1.5, 10.5], [3.5, 10.5], [1.5, 12.5], [3.5, 12.5]],
            blue: [[10.5, 1.5], [12.5, 1.5], [10.5, 3.5], [12.5, 3.5]],
            green: [[1.5, 1.5], [3.5, 1.5], [1.5, 3.5], [3.5, 3.5]],
            yellow: [[10.5, 10.5], [12.5, 10.5], [10.5, 12.5], [12.5, 12.5]]
        };

        // Safe spots (star positions)
        const SAFE_SPOTS = [0, 8, 13, 21, 26, 34, 39, 47];

        function drawBoard() {
            const size = canvas.width;
            const cellSize = size / 15;
            ctx.clearRect(0, 0, size, size);

            // Draw grid
            ctx.strokeStyle = '#ddd';
            ctx.lineWidth = 1;
            for (let i = 0; i <= 15; i++) {
                ctx.beginPath();
                ctx.moveTo(i * cellSize, 0);
                ctx.lineTo(i * cellSize, size);
                ctx.stroke();
                ctx.beginPath();
                ctx.moveTo(0, i * cellSize);
                ctx.lineTo(size, i * cellSize);
                ctx.stroke();
            }

            // Draw home areas
            drawHomeArea(0, 9, colors.red);      // Red (bottom left)
            drawHomeArea(9, 0, colors.blue);     // Blue (top right)
            drawHomeArea(0, 0, colors.green);    // Green (top left)
            drawHomeArea(9, 9, colors.yellow);   // Yellow (bottom right)

            // Draw path
            PATH.forEach((pos, idx) => {
                const x = pos.x * cellSize;
                const y = pos.y * cellSize;
                
                // Color safe spots
                if (SAFE_SPOTS.includes(idx)) {
                    ctx.fillStyle = '#fffacd';
                    ctx.fillRect(x, y, cellSize, cellSize);
                    ctx.fillStyle = '#000';
                    ctx.font = `${cellSize/2}px Arial`;
                    ctx.textAlign = 'center';
                    ctx.textBaseline = 'middle';
                    ctx.fillText('★', x + cellSize/2, y + cellSize/2);
                } else {
                    ctx.fillStyle = '#f0f0f0';
                    ctx.fillRect(x + 2, y + 2, cellSize - 4, cellSize - 4);
                }
                
                ctx.strokeStyle = '#999';
                ctx.strokeRect(x, y, cellSize, cellSize);
            });

            // Draw home stretches (final paths)
            drawHomeStretch(1, 6, 'horizontal', colors.red);
            drawHomeStretch(6, 1, 'vertical', colors.blue);
            drawHomeStretch(11, 6, 'horizontal', colors.green);
            drawHomeStretch(6, 11, 'vertical', colors.yellow);

            // Draw center home
            ctx.fillStyle = '#2c3e50';
            ctx.beginPath();
            ctx.moveTo(6 * cellSize, 6 * cellSize);
            ctx.lineTo(9 * cellSize, 6 * cellSize);
            ctx.lineTo(9 * cellSize, 9 * cellSize);
            ctx.lineTo(6 * cellSize, 9 * cellSize);
            ctx.closePath();
            ctx.fill();

            // Draw tokens
            players.forEach(color => {
                gameState.positions[color].forEach((pos, idx) => {
                    drawToken(color, idx, pos, cellSize);
                });
            });
        }

        function drawHomeArea(startX, startY, color) {
            const size = canvas.width / 15;
            ctx.fillStyle = color;
            ctx.globalAlpha = 0.3;
            ctx.fillRect(startX * size, startY * size, 6 * size, 6 * size);
            ctx.globalAlpha = 1;
            
            ctx.strokeStyle = color;
            ctx.lineWidth = 3;
            ctx.strokeRect(startX * size, startY * size, 6 * size, 6 * size);
        }

        function drawHomeStretch(startX, startY, direction, color) {
            const size = canvas.width / 15;
            ctx.fillStyle = color;
            ctx.globalAlpha = 0.5;
            
            for (let i = 0; i < 5; i++) {
                if (direction === 'horizontal') {
                    ctx.fillRect((startX + i) * size, startY * size, size, size);
                } else {
                    ctx.fillRect(startX * size, (startY + i) * size, size, size);
                }
            }
            ctx.globalAlpha = 1;
        }

        function drawToken(color, index, position, cellSize) {
            let x, y;
            
            if (position === 0) {
                // In home
                const home = HOMES[color][index];
                x = home[0] * cellSize;
                y = home[1] * cellSize;
            } else if (position > 56) {
                // Finished
                return; // Don't draw finished tokens
            } else {
                // On path
                const pathPos = PATH[position - 1];
                x = pathPos.x * cellSize;
                y = pathPos.y * cellSize;
            }

            const centerX = x + cellSize / 2;
            const centerY = y + cellSize / 2;
            const radius = cellSize * 0.35;

            // Shadow
            ctx.beginPath();
            ctx.arc(centerX + 2, centerY + 2, radius, 0, Math.PI * 2);
            ctx.fillStyle = 'rgba(0,0,0,0.3)';
            ctx.fill();

            // Token body
            ctx.beginPath();
            ctx.arc(centerX, centerY, radius, 0, Math.PI * 2);
            ctx.fillStyle = colors[color];
            ctx.fill();
            
            ctx.strokeStyle = '#fff';
            ctx.lineWidth = 2;
            ctx.stroke();

            // Highlight if selectable
            if (canMove(color, index) && gameState.dice > 0 && !gameState.rolling) {
                ctx.strokeStyle = '#fff';
                ctx.lineWidth = 3;
                ctx.beginPath();
                ctx.arc(centerX, centerY, radius + 3, 0, Math.PI * 2);
                ctx.stroke();
                
                // Glow effect
                ctx.shadowColor = colors[color];
                ctx.shadowBlur = 10;
                ctx.stroke();
                ctx.shadowBlur = 0;
            }

            // Number
            ctx.fillStyle = '#fff';
            ctx.font = `bold ${cellSize * 0.4}px Arial`;
            ctx.textAlign = 'center';
            ctx.textBaseline = 'middle';
            ctx.fillText(index + 1, centerX, centerY);
        }

        function canMove(color, tokenIndex) {
            const pos = gameState.positions[color][tokenIndex];
            if (pos === 0) return gameState.dice === 6;
            if (pos > 56) return false;
            return pos + gameState.dice <= 57;
        }

        function rollDice() {
            if (gameState.rolling) return;
            
            gameState.rolling = true;
            const diceEl = document.getElementById('dice');
            const btn = document.getElementById('rollBtn');
            
            diceEl.classList.add('rolling');
            btn.disabled = true;

            let rolls = 0;
            const interval = setInterval(() => {
                diceEl.textContent = Math.floor(Math.random() * 6) + 1;
                rolls++;
                
                if (rolls >= 10) {
                    clearInterval(interval);
                    gameState.dice = Math.floor(Math.random() * 6) + 1;
                    diceEl.textContent = gameState.dice;
                    diceEl.classList.remove('rolling');
                    
                    checkMoves();
                }
            }, 100);
        }

        function checkMoves() {
            const color = players[gameState.currentPlayer];
            const positions = gameState.positions[color];
            let hasMoves = false;

            positions.forEach((pos, idx) => {
                if (canMove(color, idx)) hasMoves = true;
            });

            if (!hasMoves) {
                setTimeout(() => {
                    nextTurn();
                }, 1500);
            } else {
                document.getElementById('status').textContent = 
                    `${color.charAt(0).toUpperCase() + color.slice(1)}: Select a token to move!`;
                gameState.rolling = false;
            }
        }

        function moveToken(color, tokenIndex) {
            if (!canMove(color, tokenIndex)) return;
            
            const currentPos = gameState.positions[color][tokenIndex];
            let newPos;
            
            if (currentPos === 0) {
                newPos = 1; // Enter board
            } else {
                newPos = currentPos + gameState.dice;
            }

            // Check captures (except on safe spots)
            if (newPos <= 52) {
                players.forEach(p => {
                    if (p !== color) {
                        gameState.positions[p].forEach((opponentPos, oppIdx) => {
                            if (opponentPos === newPos && !SAFE_SPOTS.includes(newPos - 1)) {
                                // Capture!
                                gameState.positions[p][oppIdx] = 0;
                                document.getElementById('status').textContent = 
                                    `${color} captured ${p}!`;
                            }
                        });
                    }
                });
            }

            gameState.positions[color][tokenIndex] = newPos;
            
            if (newPos === 57) {
                gameState.finished[color]++;
                document.getElementById(`info-${color}`).textContent = 
                    `${color.charAt(0).toUpperCase() + color.slice(1)} (${gameState.finished[color]}/4)`;
                
                if (gameState.finished[color] === 4) {
                    showWinner(color);
                    return;
                }
            }

            drawBoard();
            
            if (gameState.dice === 6 || newPos === 57) {
                document.getElementById('status').textContent = 
                    `${color.charAt(0).toUpperCase() + color.slice(1)} gets another turn!`;
                gameState.dice = 0;
                document.getElementById('dice').textContent = '🎲';
                document.getElementById('rollBtn').disabled = false;
            } else {
                setTimeout(nextTurn, 1000);
            }
        }

        function nextTurn() {
            gameState.currentPlayer = (gameState.currentPlayer + 1) % 4;
            gameState.dice = 0;
            gameState.rolling = false;
            
            document.querySelectorAll('.player-card').forEach((el, idx) => {
                el.classList.toggle('active', idx === gameState.currentPlayer);
            });
            
            const color = players[gameState.currentPlayer];
            document.getElementById('status').textContent = 
                `${color.charAt(0).toUpperCase() + color.slice(1)}'s Turn - Roll the dice!`;
            document.getElementById('dice').textContent = '🎲';
            document.getElementById('rollBtn').disabled = false;
        }

        function showWinner(color) {
            document.getElementById('winnerText').textContent = 
                `${color.charAt(0).toUpperCase() + color.slice(1)} Wins! 🎉`;
            document.getElementById('winnerText').style.color = colors[color];
            document.getElementById('winnerModal').style.display = 'flex';
        }

        function resetGame() {
            gameState = {
                currentPlayer: 0,
                dice: 0,
                rolling: false,
                selectedToken: null,
                positions: {
                    red: [0, 0, 0, 0],
                    blue: [0, 0, 0, 0],
                    green: [0, 0, 0, 0],
                    yellow: [0, 0, 0, 0]
                },
                finished: {
                    red: 0, blue: 0, green: 0, yellow: 0
                }
            };
            
            document.getElementById('winnerModal').style.display = 'none';
            document.getElementById('dice').textContent = '🎲';
            document.getElementById('rollBtn').disabled = false;
            
            players.forEach(color => {
                document.getElementById(`info-${color}`).textContent = 
                    `${color.charAt(0).toUpperCase() + color.slice(1)} (0/4)`;
            });
            
            document.querySelectorAll('.player-card').forEach((el, idx) => {
                el.classList.toggle('active', idx === 0);
            });
            
            document.getElementById('status').textContent = "Red's Turn - Roll the dice!";
            drawBoard();
        }

        // Handle clicks on canvas
        canvas.addEventListener('click', (e) => {
            if (gameState.dice === 0 || gameState.rolling) return;
            
            const rect = canvas.getBoundingClientRect();
            const x = e.clientX - rect.left;
            const y = e.clientY - rect.top;
            const cellSize = canvas.width / 15;
            
            const gridX = Math.floor(x / cellSize);
            const gridY = Math.floor(y / cellSize);
            
            const color = players[gameState.currentPlayer];
            
            // Check if clicked on own token
            gameState.positions[color].forEach((pos, idx) => {
                let tokenX, tokenY;
                
                if (pos === 0) {
                    const home = HOMES[color][idx];
                    tokenX = Math.floor(home[0]);
                    tokenY = Math.floor(home[1]);
                } else if (pos <= 52) {
                    const pathPos = PATH[pos - 1];
                    tokenX = pathPos.x;
                    tokenY = pathPos.y;
                }
                
                if (gridX === tokenX && gridY === tokenY) {
                    if (canMove(color, idx)) {
                        moveToken(color, idx);
                    }
                }
            });
        });

        // Initialize
        resizeCanvas();
    </script>
</body>
</html>
