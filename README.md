<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Nitro Chef: Smooth & Jumbo</title>
    <style>
        body { background: #050505; color: #fff; font-family: sans-serif; display: flex; justify-content: center; align-items: center; height: 100vh; margin: 0; overflow: hidden; }
        #game-container { position: relative; width: 360px; height: 600px; border: 4px solid #1a1a1a; border-radius: 24px; background: #000; box-shadow: 0 20px 50px rgba(0,0,0,0.5); }
        canvas { display: block; border-radius: 20px; background: #080808; }
        #overlay { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.9); display: flex; flex-direction: column; align-items: center; justify-content: center; z-index: 10; border-radius: 20px; }
        h1 { color: #00ffaa; text-shadow: 0 0 20px #00ffaa; font-size: 32px; margin-bottom: 30px; }
        #startBtn { background: #ff00ff; color: #fff; border: none; padding: 20px 50px; border-radius: 50px; font-weight: bold; font-size: 22px; cursor: pointer; box-shadow: 0 0 20px #ff00ff; }
        #startBtn:active { transform: scale(0.95); }
        .score-box { position: absolute; top: 20px; left: 20px; font-size: 24px; color: #00ffaa; font-weight: bold; z-index: 5; pointer-events: none; }
    </style>
</head>
<body>

<div id="game-container">
    <div class="score-box" id="scoreVal">SCORE: 0</div>
    <canvas id="foodCanvas" width="360" height="600"></canvas>
    <div id="overlay">
        <h1 id="statusText">NITRO CHEF</h1>
        <button id="startBtn">START ENGINE</button>
    </div>
</div>

<script>
    const canvas = document.getElementById('foodCanvas');
    const ctx = canvas.getContext('2d');
    const overlay = document.getElementById('overlay');
    const scoreVal = document.getElementById('scoreVal');
    const statusText = document.getElementById('statusText');
    const startBtn = document.getElementById('startBtn');

    // --- GAME STATE ---
    const LANES = [70, 180, 290];
    let player = { targetLane: 1, currentX: 180, lerpSpeed: 0.2 };
    let items = [];
    let score = 0;
    let gameActive = false;
    let speed = 5;
    let frame = 0;
    let animationId;

    function drawBowl(x) {
        ctx.save();
        ctx.shadowBlur = 20;
        ctx.shadowColor = "#00ffaa";
        ctx.fillStyle = "#00ffaa";
        ctx.beginPath();
        ctx.arc(x, 530, 48, 0, Math.PI, false); // Jumbo Bowl
        ctx.fill();
        ctx.restore();
    }

    function drawItem(item) {
        ctx.save();
        ctx.font = "55px serif"; // Jumbo Lettuce & Bomb
        ctx.textAlign = "center";
        ctx.textBaseline = "middle";
        ctx.fillText(item.type === 'good' ? "🥬" : "💣", item.x, item.y);
        ctx.restore();
    }

    function mainLoop() {
        if (!gameActive) return;

        ctx.clearRect(0, 0, canvas.width, canvas.height);

        // Smooth Movement
        const targetX = LANES[player.targetLane];
        player.currentX += (targetX - player.currentX) * player.lerpSpeed;

        // Lane Lines
        ctx.strokeStyle = "#111";
        [120, 240].forEach(lx => {
            ctx.beginPath(); ctx.moveTo(lx, 0); ctx.lineTo(lx, 600); ctx.stroke();
        });

        // Spawning
        frame++;
        if (frame % 40 === 0) {
            items.push({ 
                x: LANES[Math.floor(Math.random() * 3)], 
                y: -60, 
                type: Math.random() > 0.75 ? 'bad' : 'good' 
            });
        }

        // Update Items
        for (let i = items.length - 1; i >= 0; i--) {
            let it = items[i];
            it.y += speed;
            drawItem(it);

            // Smooth Collision
            if (it.y > 490 && it.y < 560 && Math.abs(it.x - player.currentX) < 45) {
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

        drawBowl(player.currentX);
        animationId = requestAnimationFrame(mainLoop);
    }

    function startGame() {
        // Hard Reset
        cancelAnimationFrame(animationId);
        gameActive = true;
        score = 0;
        speed = 5;
        items = [];
        player.targetLane = 1;
        player.currentX = LANES[1];
        frame = 0;
        scoreVal.innerText = "SCORE: 0";
        overlay.style.display = 'none';
        
        mainLoop();
    }

    function endGame() {
        gameActive = false;
        cancelAnimationFrame(animationId);
        overlay.style.display = 'flex';
        statusText.innerText = "EXPLODED!";
        startBtn.innerText = "RE-START";
    }

    // --- CONTROLS ---
    window.addEventListener('keydown', e => {
        if (e.key === "ArrowLeft" && player.targetLane > 0) player.targetLane--;
        if (e.key === "ArrowRight" && player.targetLane < 2) player.targetLane++;
    });

    canvas.addEventListener('mousedown', e => {
        const x = e.clientX - canvas.getBoundingClientRect().left;
        if (x < 180 && player.targetLane > 0) player.targetLane--;
        else if (x > 180 && player.targetLane < 2) player.targetLane++;
    });

    startBtn.addEventListener('click', startGame);

    // Initial Render
    ctx.fillStyle = "#080808";
    ctx.fillRect(0, 0, 360, 600);
    drawBowl(LANES[1]);
</script>

</body>
</html>
