<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Tiny Island Survival</title>
    <style>
        body { margin:0; padding:0; background:#0b3d91; color:#fff; font-family:Arial; overflow:hidden; touch-action:none; text-align:center; }
        canvas { display:block; margin:10px auto; border:8px solid #d4a017; border-radius:12px; image-rendering:pixelated; }
        #ui { position:absolute; top:15px; left:15px; background:rgba(0,0,0,0.8); padding:12px; border-radius:8px; font-size:15px; line-height:1.4; }
        button { background:#d4a017; color:black; border:none; padding:14px; margin:8px 0; border-radius:8px; font-size:16px; width:90%; font-weight:bold; }
        #admin { position:absolute; top:15px; right:15px; background:#c00; color:white; padding:12px 20px; border:none; border-radius:8px; font-weight:bold; }
        #joystick { position:absolute; bottom:40px; left:30px; width:110px; height:110px; background:rgba(255,255,255,0.25); border:5px solid rgba(255,255,255,0.7); border-radius:50%; }
        #knob { position:absolute; width:38px; height:38px; background:#ffd700; border-radius:50%; top:50%; left:50%; transform:translate(-50%,-50%); }
    </style>
</head>
<body>
    <canvas id="c" width="800" height="500"></canvas>

    <div id="ui">
        ❤️ Health: <span id="h">100</span><br>
        🍖 Hunger: <span id="hu">80</span><br>
        ⚡ Energy: <span id="e">100</span><br>
        🪵 Wood: <span id="w">0</span> | 🥩 Food: <span id="f">0</span> | 🪨 Stone: <span id="s">0</span><br>
        Day <span id="d">1</span>
    </div>

    <button id="admin" onclick="alert('Admin: Tap Craft Axe 3 times for resources')">🔧 ADMIN</button>

    <div id="joystick"><div id="knob"></div></div>

    <div style="position:absolute; bottom:40px; right:20px; display:flex; flex-direction:column; gap:8px; width:160px;">
        <button onclick="alert('🪓 Axe crafted!')">Craft Axe</button>
        <button onclick="alert('🏠 Shelter built!')">Build Shelter</button>
        <button onclick="alert('🌲 Explored deeper...')">Explore Deeper</button>
        <button onclick="alert('😴 Rested')">Rest</button>
    </div>

    <script>
        const canvas = document.getElementById('c');
        const ctx = canvas.getContext('2d');
        let px = 400, py = 300;
        let mx = 0, my = 0;

        // Simple island background
        function draw() {
            ctx.fillStyle = '#0b8c5e';
            ctx.fillRect(0,0,800,500);
            ctx.fillStyle = '#d4a017';
            ctx.fillRect(120,120,560,280);

            // Player
            ctx.fillStyle = '#ffcc00';
            ctx.fillRect(px, py, 28, 28);

            // Some resources
            ctx.fillStyle = '#8b4513';
            ctx.fillRect(200,200,22,22);
            ctx.fillStyle = '#ff4444';
            ctx.fillRect(500,250,22,22);
        }

        function loop() {
            draw();
            px += mx * 6;
            py += my * 6;
            px = Math.max(140, Math.min(640, px));
            py = Math.max(140, Math.min(380, py));
            requestAnimationFrame(loop);
        }

        // Joystick
        const joy = document.getElementById('joystick');
        const knob = document.getElementById('knob');
        joy.addEventListener('touchmove', e => {
            const r = joy.getBoundingClientRect();
            let dx = e.touches[0].clientX - r.left - 55;
            let dy = e.touches[0].clientY - r.top - 55;
            const d = Math.sqrt(dx*dx + dy*dy);
            if (d > 45) { dx = dx/d * 45; dy = dy/d * 45; }
            knob.style.left = (50 + dx/1.2) + '%';
            knob.style.top = (50 + dy/1.2) + '%';
            mx = dx / 50;
            my = dy / 50;
        });
        joy.addEventListener('touchend', () => {
            mx = 0; my = 0;
            knob.style.left = '50%';
            knob.style.top = '50%';
        });

        loop();
    </script>
</body>
</html>