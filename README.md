<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Nitro Chef: Jumbo Edition FIXED</title>
    <style>
        body { background: #050505; color: #fff; font-family: sans-serif; display: flex; justify-content: center; align-items: center; height: 100vh; margin: 0; overflow: hidden; }
        #game-container { position: relative; width: 360px; height: 600px; border: 5px solid #222; border-radius: 20px; background: #000; }
        canvas { display: block; border-radius: 15px; background: #080808; }
        #overlay { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.9); display: flex; flex-direction: column; align-items: center; justify-content: center; z-index: 10; border-radius: 15px; }
        h1 { color: #00ffaa; text-shadow: 0 0 15px #00ffaa; font-size: 32px; margin-bottom: 20px; }
        #startBtn { background: #ff00ff; color: #fff; border: none; padding: 20px 50px; border-radius: 50px; font-weight: bold; font-size: 22px; cursor: pointer; box-shadow: 0 0 20px #ff00ff; }
        .score-box { position: absolute; top: 20px; left: 20px; font-size: 24px; color: #00ffaa; font-weight: bold; z-index: 5; pointer-events: none; }
    </style>
</head>
<body>

<div id="game-container">
    <div class="score-box" id="scoreVal">SCORE: 0</div>
    <canvas id="foodCanvas" width="360" height="600"></canvas>
    <div id="overlay">
        <h1 id="statusText">let him cook</h1>
        <button id="startBtn" onclick="startGame()">START ENGINE</button>
    </div>
</div>

<script>
    const canvas = document.getElementById('foodCanvas');
    const ctx = canvas.getContext('2d');
    const overlay = document.getElementById('overlay');
    const scoreVal = document.getElementById('scoreVal');
    const statusText = document.getElementById('statusText');

    // CONFIG
    const LANES = [70, 180, 290];
    let playerLane = 1;
    let items = [];
    let score = 0;
    let gameActive = false;
    let speed = 5;
    let frame = 0;
    let animationId;

    function drawBowl(x) {
        ctx.save();
        ctx.fillStyle = "#00ffaa";
        ctx.shadowBlur = 15;
        ctx.shadowColor = "#00ffaa";
        ctx.beginPath();
        ctx.arc(x, 530, 45, 0, Math.PI, false); // Jumbo Bowl
        ctx.fill();
        ctx.restore();
    }

    function drawItem(item) {
        ctx.font = "50px serif"; // Jumbo Lettuce & Bomb
        ctx.textAlign = "center";
        ctx.fillText(item.type === 'good' ? "🥬" : "💣", item.x, item.y);
    }

    function mainLoop() {
        if (!gameActive) return;

        // 1. Clear
        ctx.clearRect(0, 0, canvas.width, canvas.height);

        // 2. Draw Lanes
        ctx.strokeStyle = "#111";
        [120, 240].forEach(lx => {
            ctx.beginPath(); ctx.moveTo(lx, 0); ctx.lineTo(lx, 600); ctx.stroke();
        });

        // 3. Update Items
        frame++;
        if (frame % 40 === 0) {
            items.push({ 
                x: LANES[Math.floor(Math.random() * 3)], 
                y: -50, 
                type: Math.random() > 0.7 ? 'bad' : 'good' 
            });
        }

        items.forEach((it, i) => {
            it.y += speed;
            drawItem(it);

            // Collision Detection
            if (it.y > 490 && it.y < 550 && it.x === LANES[playerLane]) {
                if (it.type === 'good') {
                    score++;
                    scoreVal.innerText = "SCORE: " + score;
                    items.splice(i, 1);
                    if (score % 5 === 0) speed += 0.5;
                } else {
                    endGame();
                }
            }
            if (it.y > 650) items.splice(i, 1);
        });

        // 4. Draw Player
        drawBowl(LANES[playerLane]);

        animationId = requestAnimationFrame(mainLoop);
    }

    function startGame() {
        console.log("Starting game...");
        // Reset everything
        cancelAnimationFrame(animationId);
        gameActive = true;
        score = 0;
        speed = 5;
        items = [];
        playerLane = 1;
        frame = 0;
        scoreVal.innerText = "SCORE: 0";
        overlay.style.display = 'none';
        
        // Start the loop
        mainLoop();
    }

    function endGame() {
        gameActive = false;
        cancelAnimationFrame(animationId);
        overlay.style.display = 'flex';
        statusText.innerText = "EXPLODED!";
        document.getElementById('startBtn').innerText = "RESTART";
    }

    // Input Logic
    window.addEventListener('keydown', e => {
        if (e.key === "ArrowLeft" && playerLane > 0) playerLane--;
        if (e.key === "ArrowRight" && playerLane < 2) playerLane++;
    });

    canvas.addEventListener('mousedown', e => {
        const x = e.clientX - canvas.getBoundingClientRect().left;
        if (x < 180 && playerLane > 0) playerLane--;
        else if (x > 180 && playerLane < 2) playerLane++;
    });

    // Initial draw so it's not a black screen
    ctx.fillStyle = "#080808";
    ctx.fillRect(0, 0, 360, 600);
    drawBowl(LANES[1]);
</script>

</body>
</html>
