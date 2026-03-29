<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Tiny Island Survival</title>
    <style>
        body { margin:0; padding:0; background:#0b3d91; color:#fff; font-family:Arial; overflow:hidden; touch-action:none; }
        canvas { display:block; margin:10px auto; border:8px solid #d4a017; border-radius:12px; image-rendering:pixelated; }
        #ui { position:absolute; top:15px; left:15px; background:rgba(0,0,0,0.8); padding:12px; border-radius:8px; font-size:15px; }
        button { background:#d4a017; color:black; border:none; padding:14px; margin:6px 0; border-radius:8px; font-size:16px; width:100%; font-weight:bold; }
        #admin-button { position:absolute; top:15px; right:15px; background:#c00; color:white; padding:12px 20px; border:none; border-radius:8px; font-weight:bold; }
        #joystick { position:absolute; bottom:30px; left:30px; width:110px; height:110px; background:rgba(255,255,255,0.25); border:5px solid rgba(255,255,255,0.7); border-radius:50%; }
        #knob { position:absolute; width:40px; height:40px; background:#ffd700; border-radius:50%; top:50%; left:50%; transform:translate(-50%,-50%); }
    </style>
</head>
<body>
    <canvas id="canvas" width="800" height="500"></canvas>

    <div id="ui">
        ❤️ Health: <span id="health">100</span><br>
        🍖 Hunger: <span id="hunger">80</span><br>
        ⚡ Energy: <span id="energy">100</span><br>
        🪵 Wood: <span id="wood">0</span><br>
        🥩 Food: <span id="food">0</span><br>
        🪨 Stone: <span id="stone">0</span><br>
        Day <span id="day">1</span>
    </div>

    <button id="admin-button" onclick="toggleAdmin()">🔧 ADMIN</button>

    <div id="joystick">
        <div id="knob"></div>
    </div>

    <div style="position:absolute; bottom:30px; right:20px; display:flex; flex-direction:column; gap:8px;">
        <button onclick="craft('axe')">Craft Axe</button>
        <button onclick="craft('shelter')">Build Shelter</button>
        <button onclick="explore()">Explore Deeper</button>
        <button onclick="rest()">Rest</button>
    </div>

    <script>
        const canvas = document.getElementById('canvas');
        const ctx = canvas.getContext('2d');
        let player = {x:400, y:300, size:22};
        let resources = [];
        let inventory = {wood:0, food:0, stone:0};
        let stats = {health:100, hunger:80, energy:100, day:1};
        let progress = 0;
        let godMode = false;
        let moveX = 0, moveY = 0;

        function spawnResources() {
            resources = [];
            for(let i=0; i<12; i++){
                resources.push({
                    x: Math.random()*650 + 80,
                    y: Math.random()*350 + 80,
                    type: ['wood','food','stone'][Math.floor(Math.random()*3)]
                });
            }