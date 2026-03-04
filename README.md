<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Nitro Chef: Food Catcher</title>
    <style>
        body { background: #111; color: #fff; font-family: 'Segoe UI', sans-serif; display: flex; justify-content: center; align-items: center; height: 100vh; margin: 0; overflow: hidden; }
        #game-container { position: relative; width: 360px; height: 600px; border: 4px solid #222; border-radius: 20px; background: #0a0a0a; }
        canvas { display: block; border-radius: 16px; }
        #overlay { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.9); display: flex; flex-direction: column; align-items: center; justify-content: center; z-index: 10; border-radius: 16px; text-align: center; }
        .garage { background: #1a1a1a; padding: 20px; border-radius: 15px; width: 80%; margin-bottom: 20px; border: 1px solid #333; }
        h1 { color: #00ffaa; text-shadow: 0 0 10px #00ffaa; margin: 0 0 10px 0; }
        label { display: block; font-size: 11px; color: #888; margin-top: 10px; text-transform: uppercase; letter-spacing: 1px; }
        input, select { width: 100%; margin-top: 5px; background: #222; border: 1px solid #444; color: #fff; padding: 8px; border-radius: 5px; }
        #startBtn { background: #00ffaa; color: #000; border: none; padding: 15px 40px; border-radius: 50px; font-weight: bold; font-size: 18px; cursor: pointer; box-shadow: 0 0 20px #00ffaa; margin-top: 10px; transition: 0.2s; }
        #startBtn:hover { transform: scale(1.05); background: #fff; }
    </style>
</head>
<body>

<div id="game-container">
    <canvas id="foodCanvas" width="360" height="600"></canvas>
    
    <div id="overlay">
        <h1>NITRO CHEF</h1>
        <div class="garage">
            <label>Bowl Color</label>
            <input type="color" id="bowlColor" value="#00ffaa">
            
            <label>Bowl Material</label>
            <select id="bowlStyle">
                <option value="neon">Neon Glass</option>
                <option value="chrome">Polished Chrome</option>
                <option value="wood">Rustic Wood</option>
            </select>
        </div>
        <button id="startBtn">START COOKING</button>
        <p style="font-size: 11px; color: #555; margin-top: 15px;">TAP SIDES TO MOVE BOWL</p>
    </div>
</div>

<script>
    const canvas = document.getElementById('foodCanvas');
    const ctx = canvas.getContext('2d');
    const overlay = document.getElementById('overlay');
    const startBtn = document.getElementById('startBtn');
    
    // Game Settings
    const LANES = [70, 180, 290];
    let player = { lane: 1, x: 180, y: 520 };
    let items = [];
    let score = 0;
    let gameActive = false;
    let fallSpeed = 4;
    let frame = 0;

    function drawBowl(x, y) {
        const color = document.getElementById('bowlColor').value;
        const style = document.getElementById('bowlStyle').value;
        
        ctx.save();
        ctx.fillStyle = color;
        
        if (style === 'neon') {
            ctx.shadowBlur = 15;
            ctx.shadowColor = color;
        } else if (style === 'chrome') {
            let grad = ctx.createLinearGradient(x-30, y, x+30, y);
            grad.addColorStop(0, '#555');
            grad.addColorStop(0.5, '#fff');
            grad.addColorStop(1, '#555');
            ctx.fillStyle = grad;
        } else if (style === 'wood') {
            ctx.fillStyle = '#5d4037';
        }

        // Bowl Shape
        ctx.beginPath();
        ctx.arc(x, y, 40, 0, Math.PI, false);
        ctx.lineTo(x - 40, y);
        ctx.fill();
        ctx.restore();
    }

    function drawItem(item) {
        ctx.save();
        if (item.type === 'good') {
            ctx.fillStyle = '#00ffaa';
            ctx.beginPath();
            ctx.arc(item.x, item.y, 15, 0, Math.PI * 2);
            ctx.fill();
            // Simple leaf shape
            ctx.fillStyle = '#fff';
            ctx.fillText("🥬", item.x - 10, item.y + 5);
        } else {
            ctx.fillStyle = '#ff4444';
            ctx.fillRect(item.x - 15, item.y - 15, 30, 30);
            ctx.fillStyle = '#fff';
            ctx.fillText("💣", item.x - 10, item.y + 5);
        }
        ctx.restore();
    }

    function update() {
        if (!gameActive) return;
        frame++;

        // Spawn items
        if (frame % 40 === 0) {
            const isBad = Math.random() > 0.8;
            items.push({ 
                x: LANES[Math.floor(Math.random() * 3)], 
                y: -50, 
                type: isBad ? 'bad' : 'good' 
            });
        }

        items.forEach((item, i) => {
            item.y += fallSpeed;

            // Collision Detection
            if (item.y > player.y - 20 && item.y < player.y + 20 && item.x === LANES[player.lane]) {
                if (item.type === 'good') {
                    score++;
                    items.splice(i, 1);
                    if (score % 10 === 0) fallSpeed += 0.5;
                } else {
                    gameOver();
                }
            }

            // Remove if off screen
            if (item.y > 650) items.splice(i, 1);
        });

        draw();
        requestAnimationFrame(update);
    }

    function draw() {
        ctx.clearRect(0, 0, 360, 600);
        
        // Background Grid
        ctx.strokeStyle = '#1a1a1a';
        [120, 240].forEach(x => {
            ctx.beginPath(); ctx.moveTo(x, 0); ctx.lineTo(x, 600); ctx.stroke();
        });

        items.forEach(drawItem);
        drawBowl(LANES[player.lane], player.y);

        // UI
        ctx.fillStyle = '#00ffaa';
        ctx.font = 'bold 20px Arial';
        ctx.fillText("SCORE: " + score, 20, 40);
    }

    function gameOver() {
        gameActive = false;
        overlay.style.display = 'flex';
        document.querySelector('h1').innerText = "KITCHEN CLOSED!";
        startBtn.innerText = "RE-OPEN";
    }

    // Controls
    window.addEventListener('keydown', e => {
        if (e.key === "ArrowLeft" && player.lane > 0) player.lane--;
        if (e.key === "ArrowRight" && player.lane < 2) player.lane++;
    });

    canvas.addEventListener('click', e => {
        const x = e.clientX - canvas.getBoundingClientRect().left;
        if (x < 180 && player.lane > 0) player.lane--;
        else if (x > 180 && player.lane < 2) player.lane++;
    });

    startBtn.addEventListener('click', () => {
        gameActive = true;
        score = 0;
        items = [];
        fallSpeed = 4;
        overlay.style.display = 'none';
        update();
    });

    // Initial draw
    draw();
</script>
</body>
</html>
