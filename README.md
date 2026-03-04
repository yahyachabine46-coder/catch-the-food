<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Nitro Chef: Ultra Smooth</title>
    <style>
        body { background: #050505; color: #fff; font-family: sans-serif; display: flex; justify-content: center; align-items: center; height: 100vh; margin: 0; overflow: hidden; }
        #game-container { position: relative; width: 360px; height: 600px; border: 4px solid #1a1a1a; border-radius: 24px; background: #000; box-shadow: 0 20px 50px rgba(0,0,0,0.5); }
        canvas { display: block; border-radius: 20px; }
        #overlay { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.8); display: flex; flex-direction: column; align-items: center; justify-content: center; z-index: 10; border-radius: 20px; transition: opacity 0.3s; }
        h1 { color: #00ffaa; text-shadow: 0 0 20px #00ffaa; font-size: 36px; margin-bottom: 30px; letter-spacing: 2px; }
        #startBtn { background: #ff00ff; color: #fff; border: none; padding: 18px 60px; border-radius: 50px; font-weight: bold; font-size: 22px; cursor: pointer; box-shadow: 0 0 30px #ff00ff; transition: 0.2s; }
        #startBtn:hover { transform: scale(1.05); filter: brightness(1.2); }
        .ui-panel { position: absolute; top: 25px; left: 0; width: 100%; display: flex; justify-content: space-around; font-size: 20px; color: #00ffaa; font-weight: bold; pointer-events: none; text-shadow: 0 0 10px #000; }
    </style>
</head>
<body>

<div id="game-container">
    <div class="ui-panel">
        <div id="scoreVal">SCORE: 0</div>
    </div>
    <canvas id="foodCanvas" width="360" height="600"></canvas>
    <div id="overlay">
        <h1 id="statusText">NITRO CHEF</h1>
        <button id="startBtn" onclick="startGame()">START ENGINE</button>
    </div>
</div>

<script>
    const canvas = document.getElementById('foodCanvas');
    const ctx = canvas.getContext('2d');
    const overlay = document.getElementById('overlay');
    const scoreVal = document.getElementById('scoreVal');
    const statusText = document.getElementById('statusText');

    // --- CONFIG ---
    const LANES = [70, 180, 290];
    let player = {
        targetLane: 1,
        currentX: 180, // Start at center
        lerpSpeed: 0.2 // This controls the "smoothness" (0.1 = slow/floaty, 0.3 = snappy)
    };

    let items = [];
    let score = 0;
    let gameActive = false;
    let speed = 5;
    let frame = 0;
    let animationId;

    function drawBowl(x) {
        ctx.save();
        ctx.translate(x, 530);
        
        // Neon Glow
        ctx.shadowBlur = 20;
        ctx.shadowColor = "#00ffaa";
        ctx.fillStyle = "#00ffaa";
        
        // Smooth Bowl Shape
        ctx.beginPath();
        ctx.arc(0, 0, 48, 0, Math.PI, false);
        ctx.fill();
        
        // Inner detail
        ctx.shadowBlur = 0;
        ctx.fillStyle = "rgba(255,255,255,0.2)";
        ctx.beginPath();
        ctx.arc(0, 5, 35, 0, Math.PI, false);
        ctx.fill();
        
        ctx.restore();
    }

    function drawItem(item) {
        ctx.save();
        ctx.font = "55px serif"; // Jumbo size
        ctx.textAlign = "center";
        ctx.textBaseline = "middle";
        
        // Add a subtle rotation effect
        ctx.translate(item.x, item.y);
        ctx.rotate(Math.sin(frame / 10 + item.x) * 0.1); 
        
        ctx.fillText(item.type === 'good' ? "🥬" : "💣", 0, 0);
        ctx.restore();
    }

    function mainLoop() {
        if (!gameActive) return;

        // 1. Clear Frame
        ctx.clearRect(0, 0, canvas.width, canvas.height);

        // 2. Draw Decorative Road/Kitchen Lines
        ctx.strokeStyle = "rgba(0, 255, 170, 0.05)";
        ctx.lineWidth = 2;
        [125, 235].forEach(lx => {
            ctx.beginPath(); ctx.moveTo(lx, 0); ctx.lineTo(lx, 600); ctx.stroke();
        });

        // 3. Smooth Movement (Lerp)
        const targetX = LANES[player.targetLane];
        player.currentX += (targetX - player.currentX) * player.lerpSpeed;

        // 4. Update Items
        frame++;
        if (frame % 35 === 0) {
            items.push({ 
                x: LANES[Math.floor(Math.random() * 3)], 
                y: -60, 
                type: Math.random() > 0.75 ? 'bad' : 'good' 
            });
        }

        for (let i = items.length - 1; i >= 0; i--) {
            let it = items[i];
            it.y += speed;
            drawItem(it);

            // Collision Detection
            // Checks if item is near bowl Y and if bowl X is overlapping item X
            const hitThreshold = 45;
            if (it.y > 490 && it.y < 560 && Math.abs(it.x - player.currentX) < hitThreshold) {
                if (it.type === 'good') {
                    score++;
                    scoreVal.innerText = "SCORE: " + score;
                    items.splice(i, 1);
                    if (score % 5 === 0) speed += 0.4;
                } else {
                    endGame();
                }
            }
            if (it.y > 650) items.splice(i, 1);
        }

        // 5. Draw Player
        drawBowl(player.currentX);

        animationId = requestAnimationFrame(mainLoop);
    }

    function startGame() {
        gameActive = true;
        score = 0;
        speed = 5;
        items = [];
        player.targetLane = 1;
        player.currentX = LANES[1];
        frame = 0;
        scoreVal.innerText = "SCORE: 0";
        overlay.style.opacity
