<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Tiny Island Survival</title>
    <style>
        body { margin:0; padding:0; font-family:Arial, sans-serif; background:#0b3d91; color:#fff; text-align:center; overflow: hidden; touch-action: none; }
        #game-container { position:relative; width:100%; max-width:800px; margin:0 auto; height:100vh; display:flex; flex-direction:column; }
        canvas { background:#0b8c5e; border:6px solid #d4a017; border-radius:8px; image-rendering:pixelated; width:100%; max-height:70vh; flex: 1; }
        #ui { position:absolute; top:10px; left:10px; background:rgba(0,0,0,0.75); padding:12px; border-radius:8px; text-align:left; font-size:14px; z-index: 10; }
        button { background:#d4a017; color:#000; border:none; padding:12px 16px; margin:6px 0; border-radius:6px; cursor:pointer; font-weight:bold; font-size:15px; width:100%; }
        button:hover, button:active { background:#f5c63d; }

        #joystick-container { position: absolute; bottom: 20px; left: 20px; width: 120px; height: 120px; z-index: 20; }
        #joystick { width: 100%; height: 100%; background: rgba(255,255,255,0.2); border: 4px solid rgba(255,255,255,0.6); border-radius: 50%; position: relative; }
        #joystick-knob { position: absolute; width: 40px; height: 40px; background: #ffd700; border-radius: 50%; top: 50%; left: 50%; transform: translate(-50%, -50%); box-shadow: 0 0 10px rgba(0,0,0,0.5); }

        #action-buttons { position: absolute; bottom: 30px; right: 20px; display: flex; flex-direction: column; gap: 8px; z-index: 20; }
        #action-buttons button { width: 140px; padding: 14px; font-size: 16px; }

        #admin-button { position: absolute; top: 15px; right: 15px; background: #c00; color: white; padding: 12px 24px; border: none; border-radius: 6px; font-size: 16px; font-weight: bold; cursor: pointer; z-index: 30; }
        #admin-panel { position: absolute; top: 70px; right: 15px; background: rgba(180, 0, 0, 0.92); padding: 15px; border-radius: 8px; width: 240px; text-align: left; display: none; border: 3px solid #ffd700; z-index: 30; }
        .cheat-btn { background: #ff4444; color: white; width: 100%; margin: 6px 0; padding: 14px; border: none; border-radius: 5px; cursor: pointer; font-weight: bold; font-size: 15px; }
        .stat { margin:8px 0; font-size:16px; }
    </style>
</head>
<body>
    <div id="game-container">
        <canvas id="canvas" width="800" height="500"></canvas>
        
        <div id="ui">
            <div class="stat">❤️ Health: <span id="health">100</span></div>
            <div class="stat">🍖 Hunger: <span id="hunger">80</span></div>
            <div class="stat">⚡ Energy: <span id="energy">100</span></div>
            <div class="stat">🪵 Wood: <span id="wood">0</span></div>
            <div class="stat">🥩 Food: <span id="food">0</span></div>
            <div class="stat">🪨 Stone: <span id="stone">0</span></div>
            <div class="stat">💰 Coins: <span id="coins">0</span></div>
            <div class="stat">Day <span id="day">1</span></div>
        </div>

        <div id="joystick-container">
            <div id="joystick">
                <div id="joystick-knob"></div>
            </div>
        </div>

        <div id="action-buttons">
            <button onclick="craftItem('axe')">Craft Axe</button>
            <button onclick="craftItem('shelter')">Build Shelter</button>
            <button onclick="exploreDeeper()">Explore Deeper 🌲</button>
            <button onclick="rest()">Rest</button>
        </div>

        <button id="admin-button" onclick="toggleAdminPanel()">🔧 ADMIN</button>

        <div id="admin-panel">
            <h3 style="margin:0 0 12px 0; color:#ffd700;">ADMIN CHEATS</h3>
            <button class="cheat-btn" onclick="giveUnlimitedResources()">Unlimited Resources</button>
            <button class="cheat-btn" onclick="addCoins(10000)">+10,000 Coins</button>
            <button class="cheat-btn" onclick="maxStats()">Max Stats</button>
            <button class="cheat-btn" onclick="instantProgress()">Deep Forest</button>
            <button class="cheat-btn" onclick="toggleGodMode()">God Mode</button>
            <button class="cheat-btn" onclick="resetCheats()">Reset Cheats</button>
        </div>
    </div>

    <p id="message" style="position:absolute; bottom:10px; left:0; right:0; font-size:16px; color:#ffd700; margin:0; pointer-events:none;"></p>

    <script>
        const canvas = document.getElementById('canvas');
        const ctx = canvas.getContext('2d');
        
        let player = { x: 400, y: 300, size: 20, speed: 4 };
        let resources = [];
        let enemies = [];
        let inventory = { wood: 0, food: 0, stone: 0 };
        let coins = 0;
        let stats = { health: 100, hunger: 80, energy: 100, day: 1 };
        let progress = 0;
        let hasAxe = false;
        let hasShelter = false;
        let gameOver = false;
        let godMode = false;

        let joystickActive = false;
        let moveX = 0, moveY = 0;
        const joystick = document.getElementById('joystick');
        const knob = document.getElementById('joystick-knob');

        function updateJoystick(e) {
            const rect = joystick.getBoundingClientRect();
            const touch = e.touches ? e.touches[0] : e;
            let dx = touch.clientX - rect.left - 60;
            let dy = touch.clientY - rect.top - 60;
            const dist = Math.sqrt(dx*dx + dy*dy);
            if (dist > 50) { dx = (dx/dist)*50; dy = (dy/dist)*50; }
            knob.style.left = `${60 + dx - 20}px`;
            knob.style.top = `${60 + dy - 20}px`;
            moveX = dx / 50;
            moveY = dy / 50;
        }

        joystick.addEventListener('touchstart', e => { joystickActive = true; updateJoystick(e); });
        joystick.addEventListener('touchmove', e => { if (joystickActive) updateJoystick(e); });
        joystick.addEventListener('touchend', () => { 
            joystickActive = false; 
            knob.style.left = '40px'; knob.style.top = '40px'; 
            moveX = 0; moveY = 0; 
        });

        function toggleAdminPanel() {
            const panel = document.getElementById('admin-panel');
            panel.style.display = panel.style.display === 'block' ? 'none' : 'block';
        }

        window.giveUnlimitedResources = function() {
            inventory.wood = inventory.food = inventory.stone = 9999;
            document.getElementById('message').textContent = '💎 Unlimited resources!';
            updateStats();
        };

        window.addCoins = function(amount) {
            coins += amount;
            document.getElementById('message').textContent = `💰 +${amount} Coins`;
            updateStats();
        };

        window.maxStats = function() {
            stats.health = stats.hunger = stats.energy = 100;
            document.getElementById('message').textContent = '❤️ Stats maxed!';
            updateStats();
        };

        window.instantProgress = function() {
            progress = 3; stats.day = 10;
            document.getElementById('message').textContent = '🌴 Deep Forest!';
            updateStats();
        };

        window.toggleGodMode = function() {
            godMode = !godMode;
            document.getElementById('message').textContent = godMode ? '🛡️ God Mode ON' : '🛡️ God Mode OFF';
        };

        window.resetCheats = function() {
            inventory = { wood: 0, food: 0, stone: 0 };
            coins = 0; godMode = false;
            document.getElementById('message').textContent = 'Cheats reset';
            updateStats();
        };

        function spawnResources() {
            resources = [];
            for (let i = 0; i < 12; i++) {
                resources.push({ x: Math.random()*700+50, y: Math.random()*400+50, type: ['wood','food','stone'][Math.floor(Math.random()*3)], size:18 });
            }
        }

        function spawnEnemy() {
            if (progress > 0 && Math.random() < 0.3) enemies.push({ x: 650, y: Math.random()*400+50, size:22 });
        }

        function drawIsland() {
            ctx.fillStyle = '#0b8c5e'; ctx.fillRect(0,0,800,500);
            ctx.fillStyle = '#d4a017'; ctx.fillRect(80,80,640,340);
            ctx.fillStyle = '#0b8c5e'; ctx.fillRect(100,100,600,300);
            
            for (let i = 0; i < 18; i++) {
                let tx = 120 + (i%9)*65;
                let ty = 120 + Math.floor(i/9)*120;
                ctx.fillStyle = '#0b3d91'; ctx.fillRect(tx+5, ty+20, 20, 40);
                ctx.fillStyle = '#0b8c5e'; ctx.beginPath(); ctx.arc(tx+15, ty+15, 25, 0, Math.PI*2); ctx.fill();
            }
            
            ctx.fillStyle = '#ffcc00'; ctx.fillRect(player.x, player.y, player.size, player.size);
            ctx.fillStyle = '#8b4513'; ctx.fillRect(player.x+4, player.y-8, 12, 10);
            
            resources.forEach(r => {
                ctx.fillStyle = r.type==='wood'?'#8b4513':r.type==='food'?'#ff4444':'#aaaaaa';
                ctx.fillRect(r.x, r.y, r.size, r.size);
            });
            
            ctx.fillStyle = '#ff0000';
            enemies.forEach(e => ctx.fillRect(e.x, e.y, e.size, e.size));
            
            ctx.fillStyle = 'rgba(255,255,255,0.7)'; ctx.fillRect(20,20,760,20);
            ctx.fillStyle = '#ffd700'; ctx.fillRect(20,20,760*(progress/3),20);
        }

        function updateStats() {
            document.getElementById('health').textContent = Math.floor(stats.health);
            document.getElementById('hunger').textContent = Math.floor(stats.hunger);
            document.getElementById('energy').textContent = Math.floor(stats.energy);
            document.getElementById('wood').textContent = inventory.wood;
            document.getElementById('food').textContent = inventory.food;
            document.getElementById('stone').textContent = inventory.stone;
            document.getElementById('coins').textContent = coins;
            document.getElementById('day').textContent = stats.day;
        }

        function checkCollision(a, b) { return Math.hypot(a.x - b.x, a.y - b.y) < 30; }

        function gameLoop() {
            if (gameOver) return;
            ctx.clearRect(0,0,800,500);
            drawIsland();
            
            if (!godMode) {
                stats.hunger -= 0.03;
                stats.energy -= 0.02;
                if (stats.hunger <= 0) stats.health -= 0.1;
                if (stats.energy <= 0) stats.health -= 0.05;
            }
            
            if (stats.health <= 0) {
                gameOver = true;
                document.getElementById('message').textContent = '💀 Game Over!';
                return;
            }
            
            player.x += moveX * player.speed * 1.8;
            player.y += moveY * player.speed * 1.8;
            player.x = Math.max(120, Math.min(680, player.x));
            player.y = Math.max(120, Math.min(380, player.y));
            
            for (let i = resources.length-1; i >= 0; i--) {
                if (checkCollision(player, resources[i])) {
                    inventory[resources[i].type]++;
                    resources.splice(i,1);
                }
            }
            
            for (let i = enemies.length-1; i >= 0; i--) {
                if (checkCollision(player, enemies[i])) {
                    stats.health -= 15;
                    enemies.splice(i,1);
                }
            }
            
            if (resources.length < 6) spawnResources();
            updateStats();
            requestAnimationFrame(gameLoop);
        }

        window.craftItem = function(item) {
            if (item === 'axe' && inventory.wood >= 5 && inventory.stone >= 3) {
                inventory.wood -= 5; inventory.stone -= 3;
                player.speed = 6;
                document.getElementById('message').textContent = '🪓 Axe crafted!';
            }
            if (item === 'shelter' && inventory.wood >= 10 && inventory.stone >= 5) {
                inventory.wood -= 10; inventory.stone -= 5;
                stats.health = Math.min(100, stats.health + 20);
                document.getElementById('message').textContent = '🏠 Shelter built!';
            }
            updateStats();
        };

        window.exploreDeeper = function() {
            if (stats.energy < 20) { document.getElementById('message').textContent = 'Not enough energy!'; return; }
            stats.energy -= 20;
            progress = Math.min(3, progress + 1);
            spawnEnemy();
            stats.day++;
            updateStats();
        };

        window.rest = function() {
            stats.energy = Math.min(100, stats.energy + 40);
            stats.hunger = Math.max(0, stats.hunger - 10);
            document.getElementById('message').textContent = '😴 Rested';
            updateStats();
        };

        function startGame() {
            spawnResources();
            spawnEnemy();
            gameLoop();
            setInterval(() => { if (!gameOver && Math.random() < 0.2) spawnEnemy(); }, 4000);
            setInterval(() => { if (!gameOver) stats.day++; }, 30000);
        }
        
        startGame();
    </script>
</body>
</html>
