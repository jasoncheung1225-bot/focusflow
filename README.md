# focusflow[ai_studio_code-5.html](https://github.com/user-attachments/files/25825055/ai_studio_code-5.html)
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>FocusFlow Pro</title>
    <style>
        :root {
            --primary: #38bdf8;
            --bg: #0f172a;
            --panel: rgba(30, 41, 59, 0.9);
        }

        * { 
            box-sizing: border-box; 
            -webkit-tap-highlight-color: transparent; 
            user-select: none; 
        }

        body, html {
            margin: 0; padding: 0; width: 100%; height: 100%;
            background-color: var(--bg);
            color: white; font-family: -apple-system, system-ui, sans-serif;
            overflow: hidden; touch-action: none;
        }

        /* HUD Layer */
        #hud {
            position: absolute; top: 15px; left: 15px;
            z-index: 10; pointer-events: none;
            font-weight: bold; font-variant-numeric: tabular-nums;
        }

        /* Menu Overlay */
        #overlay {
            position: absolute; inset: 0;
            background: var(--bg);
            display: flex; flex-direction: column;
            align-items: center; justify-content: center;
            z-index: 100; padding: 20px;
            text-align: center;
        }

        h1 { font-size: 2.5rem; margin: 0 0 10px; color: var(--primary); }
        
        .mode-container {
            display: flex; flex-direction: column;
            gap: 12px; width: 100%; max-width: 300px;
            margin-top: 20px;
        }

        .btn {
            background: #1e293b; border: 2px solid var(--primary);
            color: white; padding: 15px; font-size: 1.1rem;
            border-radius: 12px; cursor: pointer; font-weight: bold;
            transition: all 0.2s;
        }

        .btn:active { transform: scale(0.95); background: var(--primary); color: var(--bg); }
        .btn-easy { border-color: #4ade80; }
        .btn-medium { border-color: #fbbf24; }
        .btn-hard { border-color: #f87171; }

        canvas { display: block; }
    </style>
</head>
<body>

    <div id="hud">
        Score: <span id="scoreDisplay">0</span> | 
        Acc: <span id="accDisplay">100</span>% | 
        Time: <span id="timerDisplay">30</span>s
    </div>

    <div id="overlay">
        <h1 id="menuTitle">FocusFlow</h1>
        <p id="menuDesc">Select a difficulty to begin training.</p>
        
        <div class="mode-container" id="modeSelect">
            <button class="btn btn-easy" onclick="startGame('easy')">EASY</button>
            <button class="btn btn-medium" onclick="startGame('medium')">MEDIUM</button>
            <button class="btn btn-hard" onclick="startGame('hard')">DIFFICULT</button>
        </div>
    </div>

    <canvas id="gameCanvas"></canvas>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const overlay = document.getElementById('overlay');
        
        // Settings for Difficulties
        const DIFFICULTY = {
            easy:   { spawn: 1200, life: 2500, size: 45, color: '#4ade80' },
            medium: { spawn: 800,  life: 1800, size: 35, color: '#fbbf24' },
            hard:   { spawn: 500,  life: 1100, size: 22, color: '#f87171' }
        };

        let currentMode = 'easy';
        let gameActive = false;
        let score = 0, hits = 0, attempts = 0, timeLeft = 30;
        let targets = [];
        let lastTime = 0, spawnCounter = 0;

        function resize() {
            const dpr = window.devicePixelRatio || 1;
            canvas.width = window.innerWidth * dpr;
            canvas.height = window.innerHeight * dpr;
            ctx.scale(dpr, dpr);
            canvas.style.width = window.innerWidth + 'px';
            canvas.style.height = window.innerHeight + 'px';
        }

        window.addEventListener('resize', resize);
        resize();

        class Target {
            constructor(config) {
                this.maxRadius = config.size;
                this.x = this.maxRadius + Math.random() * (window.innerWidth - this.maxRadius * 2);
                this.y = this.maxRadius + Math.random() * (window.innerHeight - this.maxRadius * 2);
                this.maxLife = config.life;
                this.life = config.life;
                this.color = config.color;
            }

            draw() {
                const ratio = this.life / this.maxLife;
                const currentRadius = this.maxRadius * ratio;
                
                ctx.beginPath();
                ctx.arc(this.x, this.y, currentRadius, 0, Math.PI * 2);
                ctx.fillStyle = this.color;
                ctx.fill();
                ctx.strokeStyle = "white";
                ctx.lineWidth = 2;
                ctx.stroke();
                ctx.closePath();
            }
        }

        function startGame(mode) {
            currentMode = mode;
            score = 0; hits = 0; attempts = 0; timeLeft = 30;
            targets = [];
            gameActive = true;
            spawnCounter = 0;
            
            overlay.style.display = 'none';
            updateHUD();

            lastTime = performance.now();
            requestAnimationFrame(gameLoop);
            
            // Start Countdown
            const clock = setInterval(() => {
                if (!gameActive) { clearInterval(clock); return; }
                timeLeft--;
                document.getElementById('timerDisplay').innerText = timeLeft;
                if (timeLeft <= 0) endGame();
            }, 1000);
        }

        function endGame() {
            gameActive = false;
            overlay.style.display = 'flex';
            const acc = attempts === 0 ? 0 : Math.round((hits / attempts) * 100);
            
            document.getElementById('menuTitle').innerText = "Results";
            document.getElementById('menuDesc').innerHTML = 
                `Difficulty: ${currentMode.toUpperCase()}<br>` +
                `Score: ${score} | Accuracy: ${acc}%`;
        }

        function updateHUD() {
            document.getElementById('scoreDisplay').innerText = score;
            const acc = attempts === 0 ? 100 : Math.round((hits / attempts) * 100);
            document.getElementById('accDisplay').innerText = acc;
        }

        function gameLoop(time) {
            if (!gameActive) return;
            const dt = time - lastTime;
            lastTime = time;

            ctx.clearRect(0, 0, window.innerWidth, window.innerHeight);

            // Spawning
            spawnCounter += dt;
            if (spawnCounter > DIFFICULTY[currentMode].spawn) {
                targets.push(new Target(DIFFICULTY[currentMode]));
                spawnCounter = 0;
            }

            // Update/Draw Targets
            for (let i = targets.length - 1; i >= 0; i--) {
                targets[i].life -= dt;
                if (targets[i].life <= 0) {
                    targets.splice(i, 1);
                } else {
                    targets[i].draw();
                }
            }

            requestAnimationFrame(gameLoop);
        }

        // CRITICAL FIX: Precision input handling for mobile
        function handleInput(e) {
            if (!gameActive) return;
            
            // Prevent zooming/scrolling behavior
            if (e.cancelable) e.preventDefault();

            attempts++;
            
            // Get exact coordinates relative to the canvas scale
            const rect = canvas.getBoundingClientRect();
            const clientX = e.clientX || (e.touches && e.touches[0].clientX);
            const clientY = e.clientY || (e.touches && e.touches[0].clientY);
            
            const x = clientX - rect.left;
            const y = clientY - rect.top;

            let hitFound = false;
            for (let i = targets.length - 1; i >= 0; i--) {
                const t = targets[i];
                const dist = Math.hypot(x - t.x, y - t.y);
                
                // Allow a tiny bit of "fat finger" margin on mobile (10px)
                const margin = 10; 
                
                if (dist < t.maxRadius + margin) {
                    hits++;
                    score += (currentMode === 'hard' ? 200 : currentMode === 'medium' ? 150 : 100);
                    targets.splice(i, 1);
                    hitFound = true;
                    break;
                }
            }
            updateHUD();
        }

        // Pointer Events handle both Mouse and Touch reliably
        canvas.addEventListener('pointerdown', handleInput);

    </script>
</body>
</html>
