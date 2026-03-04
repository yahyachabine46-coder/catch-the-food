<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Nitro Chef: Jumbo Edition</title>
    <style>
        body { background: #0a0a0a; color: #fff; font-family: 'Segoe UI', sans-serif; display: flex; justify-content: center; align-items: center; height: 100vh; margin: 0; overflow: hidden; }
        #game-container { position: relative; width: 360px; height: 600px; border: 4px solid #333; border-radius: 20px; background: #000; box-shadow: 0 0 30px rgba(0,255,170,0.2); }
        canvas { display: block; border-radius: 16px; }
        #overlay { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.85); display: flex; flex-direction: column; align-items: center; justify-content: center; z-index: 10; border-radius: 16px; text-align: center; }
        h1 { color: #00ffaa; text-shadow: 0 0 15px #00ffaa; margin: 0 0 20px 0; font-size: 32px; }
        #startBtn { background: #00ffaa; color: #000; border: none; padding: 18px 50px; border-radius: 50px; font-weight: bold; font-size: 20px; cursor: pointer; box-shadow: 0 0 25px #00ffaa; transition: 0.3s; }
        #startBtn:hover { transform: scale(1.1); background: #fff; }
        .score-ui { position: absolute; top: 20px; left: 20px; font-size: 24px; font-weight: bold; color: #00ffaa; pointer-events: none; }
    </style>
</head>
<body>

<div id="game-container">
    <div class="score-ui" id="scoreDisplay">SCORE: 0</div>
    <canvas id="foodCanvas" width="360" height="600"></canvas>
    
    <div id="overlay">
        <h1 id="mainTitle">NITRO CHEF</h1>
        <button id="startBtn">START COOKING</button>
        <p style="color: #666; margin-top: 20px; font-size: 12px;">TOUCH SIDES TO MOVE</p>
    </div>
</div>

<script>
    const canvas = document.getElementById('foodCanvas');
    const ctx = canvas.getContext('2d');
    const overlay = document.getElementById('overlay');
    const startBtn = document.getElementById('startBtn');
    const scoreDisplay = document.getElementById('scoreDisplay');
    
    const LANES = [70, 180, 290];
    let player = { lane: 1, y: 520 };
    let items = [];
    let score = 0;
    let gameActive = false;
    let fallSpeed = 5;
    let frame = 0;

    // BIGGER SIZE SETTINGS
    const ITEM_RADIUS = 30; // Increased from 15

    function drawBowl(x, y) {
        ctx.save();
        ctx.
