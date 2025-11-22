# Mobile-friendlycatheader
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cat Header Game</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap" rel="stylesheet">
    <style>
        /* Custom CSS for a retro game feel */
        body {
            font-family: 'Press Start 2P', monospace;
            background-color: #333;
            color: #fff;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            padding: 10px;
        }

        #game-container {
            width: 100%;
            max-width: 480px; /* Max width for a clean game screen */
            background-color: #222;
            border: 8px solid #00f; /* Retro border */
            box-shadow: 0 0 20px rgba(0, 255, 255, 0.5); /* Glowing effect */
            border-radius: 12px;
            padding: 10px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        
        /* New container for the canvas and its absolute overlays */
        #game-canvas-area {
            position: relative; /* All overlays will be positioned relative to this */
            width: 100%;
            border-radius: 6px; /* Match canvas border radius */
            overflow: hidden; /* Important for containing absolute children */
        }

        canvas {
            background-color: #008000; /* Green field background */
            display: block;
            touch-action: none; /* Prevent browser scrolling on touch */
            border-radius: 6px;
            box-shadow: inset 0 0 10px rgba(0, 0, 0, 0.5);
            width: 100%; /* Make canvas fluid within container */
            max-height: 70vh; /* Prevent it from taking too much vertical space */
        }

        .control-button {
            @apply bg-blue-500 hover:bg-blue-700 text-white font-bold py-3 px-6 rounded-lg text-lg m-2 transition-transform duration-100 active:scale-95 shadow-xl shadow-blue-500/50;
            border: 4px solid #fff;
            user-select: none; /* Prevent text selection on touch */
        }
        
        #pause-button {
            @apply bg-yellow-500 hover:bg-yellow-600 text-gray-900 font-bold py-3 px-6 rounded-lg text-lg m-2 transition-transform duration-100 active:scale-95 shadow-xl shadow-yellow-500/50;
            border: 4px solid #fff;
            user-select: none;
        }

        .score-display {
            /* Used for both Current Score and Best Score */
            @apply text-xl my-4 text-yellow-300;
            text-shadow: 2px 2px #000;
        }
        
        #score-bar {
            /* Container to hold both scores and align them to the corners */
            @apply flex justify-between w-full px-2;
        }
        
        #lives-display {
            /* Now always visible but still styled for emphasis */
            @apply text-xl my-4 text-red-500 font-bold bg-yellow-900 px-3 py-1 rounded-full shadow-lg;
            border: 2px solid #fff;
            animation: pulse 1s infinite alternate;
        }
        
        @keyframes pulse {
            from { transform: scale(1); opacity: 0.9; }
            to { transform: scale(1.05); opacity: 1; }
        }
        
        #lives-container {
            /* Container for Score and Lives, aligned left */
            @apply flex items-center space-x-4;
        }
        
        #control-row {
            display: flex;
            justify-content: space-around;
            width: 100%;
            margin-top: 15px;
        }
        
        #movement-controls {
            display: flex;
        }
        
        /* OVERLAY STYLES - Now positioned relative to #game-canvas-area */
        .overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            /* Center content inside the overlay */
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            background-color: rgba(0, 0, 0, 0.9); /* Darker backdrop for menu screens */
            text-align: center;
            z-index: 10;
        }

        #start-screen h1 {
            @apply text-4xl mb-6 text-green-400;
        }
        
        #start-screen p {
            @apply text-lg mb-8 text-white;
        }

        #play-button, #restart-button {
            @apply bg-green-500 hover:bg-green-700 text-white font-bold py-4 px-8 rounded-full text-xl transition-all duration-200 shadow-2xl shadow-green-500/50;
            border: 4px solid #fff;
        }

        #game-over-screen h1 {
            @apply text-3xl mb-4 text-red-400 animate-pulse;
        }

        #game-over-screen p {
            @apply text-xl mb-6 text-white;
        }
        
        #pause-screen h1 {
             @apply text-5xl text-yellow-300 animate-pulse;
        }
        
        #pause-screen p {
            @apply text-lg mt-4 text-white;
        }


    </style>
</head>
<body>

<div id="game-container">
    
    <!-- Score Bar: Displays current score (left) and best score (right) -->
    <div id="score-bar">
        <!-- Current Score and Lives (Top Left) -->
        <div id="lives-container">
            <div class="score-display">Score: <span id="score-value">0</span></div>
            <!-- Lives Display (Always visible) -->
            <div id="lives-display">Lives: <span id="lives-value">5</span></div>
        </div>
        <!-- Best Score (Top Right) -->
        <div class="score-display text-red-400">Best: <span id="best-score-value">0</span></div>
    </div>
    
    <div id="game-canvas-area">
        <canvas id="gameCanvas"></canvas>
        
        <!-- START SCREEN -->
        <div id="start-screen" class="overlay">
            <h1>CAT HEADER</h1>
            <p>Keep the ball in the air using the cat's head!</p>
            <button id="play-button">PLAY GAME</button>
            <p class="text-sm mt-8 opacity-70">Controls: A/D or Left/Right Arrows</p>
        </div>

        <!-- Game Over Screen -->
        <div id="game-over-screen" class="overlay" style="display: none;">
            <h1 class="text-3xl">GAME OVER!</h1>
            <p>Final Score: <span id="final-score-value">0</span></p>
            <button id="restart-button">RESTART</button>
        </div>
        
        <!-- Pause Overlay -->
        <div id="pause-screen" class="overlay" style="display: none;">
            <h1 class="text-5xl text-yellow-300 animate-pulse">PAUSED</h1>
            <p class="text-lg mt-4 text-white">Press 'Esc' or click RESUME to continue.</p>
        </div>
    </div>
    
    <div id="control-row">
        <div id="movement-controls">
            <button id="left-button" class="control-button" ontouchstart="moveCat(-1)" ontouchend="stopCat()" onmousedown="moveCat(-1)" onmouseup="stopCat()" onmouseleave="stopCat()">
                &lt;&lt; LEFT (A)
            </button>
            <button id="right-button" class="control-button" ontouchstart="moveCat(1)" ontouchend="stopCat()" onmousedown="moveCat(1)" onmouseup="stopCat()" onmouseleave="stopCat()">
                RIGHT &gt;&gt; (D)
            </button>
        </div>
        <button id="pause-button">PAUSE (Esc)</button>
    </div>

</div>

<script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const scoreValue = document.getElementById('score-value');
    const bestScoreValue = document.getElementById('best-score-value');
    const livesDisplay = document.getElementById('lives-display'); 
    const livesValue = document.getElementById('lives-value'); 
    const gameOverScreen = document.getElementById('game-over-screen');
    const startScreen = document.getElementById('start-screen');
    const playButton = document.getElementById('play-button');
    const finalScoreValue = document.getElementById('final-score-value');
    const restartButton = document.getElementById('restart-button');
    const pauseButton = document.getElementById('pause-button');
    const pauseScreen = document.getElementById('pause-screen');

    // --- Game Variables ---
    let animationFrameId;
    let isGameOver = false;
    let isPaused = false;
    let isGameRunning = false;
    let score = 0;
    let bestScore = 0; 
    let catSpeed = 0;
    
    // Lives State: Fixed 5 lives now
    let lives = 0;
    const STARTING_LIVES = 5; 
    
    // Constants
    const CAT_MOVE_VELOCITY = 10; 
    const CAT_SIZE = 50;
    const BALL_SIZE = 20;
    const MAX_BOUNCE_HEIGHT_RATIO = 0.6; 
    const BASE_GRAVITY = 0.23; // ADJUSTED: Significantly slower ball speed (approx 75% of previous)
    const GRAVITY_SCALE_PER_SCORE = 0.005; // ADJUSTED: Slower difficulty ramp
    const BEST_SCORE_STORAGE_KEY = 'catHeaderBestScore';

    // Ball object
    let cat = { x: 0, y: 0, width: CAT_SIZE, height: CAT_SIZE };
    let ball = { x: 0, y: 0, radius: BALL_SIZE, velY: 0, gravity: BASE_GRAVITY, velX: 0 };

    // --- High Score Functions ---

    /**
     * Loads the high score from local storage and displays it.
     */
    function loadBestScore() {
        const savedScore = localStorage.getItem(BEST_SCORE_STORAGE_KEY);
        bestScore = savedScore ? parseInt(savedScore, 10) : 0;
        bestScoreValue.textContent = bestScore;
    }

    /**
     * Checks if the new score is a record, saves it, and updates the display.
     * @param {number} newScore The score achieved in the current game.
     * @returns {boolean} True if a new high score was set.
     */
    function saveBestScore(newScore) {
        if (newScore > bestScore) {
            bestScore = newScore;
            localStorage.setItem(BEST_SCORE_STORAGE_KEY, bestScore);
            bestScoreValue.textContent = bestScore;
            return true; // New high score achieved
        }
        return false;
    }

    // --- Utility Functions ---

    /**
     * Handles the resizing of the canvas to fit the container.
     */
    function resizeCanvas() {
        const container = document.getElementById('game-container');
        const canvasArea = document.getElementById('game-canvas-area');

        // Set fixed aspect ratio (e.g., 3:4) or match container width
        const containerWidth = container.clientWidth - 20; // Account for container padding
        const newHeight = Math.min(containerWidth * 1.5, window.innerHeight * 0.7); 
        
        canvas.width = containerWidth;
        canvas.height = newHeight;
        
        // Ensure the canvas area container has the same height as the canvas
        canvasArea.style.height = `${newHeight}px`;

        // Reposition cat after resize (only if game is running/initialized)
        if (isGameRunning) {
            cat.x = Math.max(0, Math.min(canvas.width - cat.width, cat.x));
            cat.y = canvas.height - cat.height - 10;
        }
    }

    /**
     * Toggles the pause state of the game.
     */
    function togglePause() {
        if (isGameOver || !isGameRunning) return;
        
        isPaused = !isPaused;
        pauseButton.textContent = isPaused ? 'RESUME (Esc)' : 'PAUSE (Esc)';
        
        // Show/hide the pause screen
        if (isPaused) {
            pauseScreen.style.display = 'flex';
        } else {
            pauseScreen.style.display = 'none';
        }
    }
    
    window.togglePause = togglePause; // Expose to global scope for button click

    /**
     * Sets up the game state and starts the main loop. Called by Play/Restart buttons.
     */
    function startGame() {
        isGameOver = false;
        isPaused = false;
        isGameRunning = true; // Game is now active
        score = 0; // Start score at 0
        
        // Initialize lives to 5 and display them
        lives = STARTING_LIVES; 
        livesValue.textContent = lives;

        catSpeed = 0;
        scoreValue.textContent = score;
        
        // Reset Game Over heading
        gameOverScreen.querySelector('h1').textContent = "GAME OVER!"; 
        
        // Hide all overlays
        startScreen.style.display = 'none';
        gameOverScreen.style.display = 'none';
        pauseScreen.style.display = 'none';
        pauseButton.textContent = 'PAUSE (Esc)';

        resizeCanvas();

        // Initial cat position
        cat.x = canvas.width / 2 - cat.width / 2;
        cat.y = canvas.height - cat.height - 10;

        resetBall();

        if (animationFrameId) {
            cancelAnimationFrame(animationFrameId);
        }
        gameLoop();
    }

    /**
     * Resets the ball to the top with a random X position.
     */
    function resetBall() {
        const xRange = canvas.width - ball.radius * 2;
        ball.x = ball.radius + Math.random() * xRange;
        ball.y = ball.radius;
        ball.velY = 0;
        ball.velX = 0; // Reset horizontal velocity
    }

    /**
     * Ends the game and displays the game over screen.
     */
    function endGame() {
        isGameOver = true;
        isPaused = false; // Ensure it's not paused when ending
        isGameRunning = false;
        cancelAnimationFrame(animationFrameId);
        
        // Check and save best score
        const isNewBest = saveBestScore(score);

        // Update game over message
        finalScoreValue.textContent = score;
        
        if (isNewBest) {
             gameOverScreen.querySelector('h1').textContent = "NEW BEST SCORE!";
        } else {
             gameOverScreen.querySelector('h1').textContent = "GAME OVER!";
        }
        
        gameOverScreen.style.display = 'flex';
        
        // Log the final score
        console.log(`Game Over! Final Score: ${score}`);
    }

    // --- Drawing Functions ---

    /**
     * Draws the cat on the canvas.
     */
    function drawCat() {
        ctx.fillStyle = '#FFC0CB'; // Cat body color (pink/white for contrast)
        ctx.font = `${CAT_SIZE}px sans-serif`;
        ctx.textAlign = 'center';
        // Using a cat emoji for simple, effective graphics
        ctx.fillText('🐱', cat.x + cat.width / 2, cat.y + cat.height - 5);
    }

    /**
     * Draws the ball on the canvas.
     */
    function drawBall() {
        ctx.font = `${BALL_SIZE * 1.5}px sans-serif`;
        ctx.textAlign = 'center';
        // Using a soccer ball emoji
        ctx.fillText('⚽', ball.x, ball.y + ball.radius * 0.6);
    }

    /**
     * Draws the background/field lines (optional decoration).
     */
    function drawField() {
        // Draw the ground line
        ctx.fillStyle = '#6B8E23'; // Darker green for the ground
        ctx.fillRect(0, canvas.height - 10, canvas.width, 10);
        
        // Center circle (optional flair)
        ctx.beginPath();
        ctx.arc(canvas.width / 2, canvas.height / 2, 40, 0, Math.PI * 2, false);
        ctx.strokeStyle = 'rgba(255, 255, 255, 0.3)';
        ctx.lineWidth = 2;
        ctx.stroke();
    }

    // --- Update Logic ---

    /**
     * Updates the position of game objects and checks for collisions.
     */
    function update() {
        // 1. Calculate dynamic gravity based on score
        ball.gravity = BASE_GRAVITY + score * GRAVITY_SCALE_PER_SCORE;
        
        // 2. Calculate required upward velocity for fixed bounce height (60% of screen)
        const groundY = canvas.height - 10;
        // Target Y position (60% up from the ground, plus ball radius offset)
        const targetY = groundY * (1 - MAX_BOUNCE_HEIGHT_RATIO); 
        // Distance the ball needs to travel vertically
        const targetDistance = groundY - targetY; 
        
        // Calculate the required upward velocity (V = sqrt(2 * g * h))
        const requiredUpwardVel = -Math.sqrt(2 * ball.gravity * targetDistance); 


        // 3. Update Cat Position (Movement)
        cat.x += catSpeed * CAT_MOVE_VELOCITY;
        
        // Cat boundary check (keep cat on screen)
        if (cat.x < 0) {
            cat.x = 0;
        } else if (cat.x + cat.width > canvas.width) {
            cat.x = canvas.width - cat.width;
        }

        // 4. Update Ball Position (Gravity)
        ball.velY += ball.gravity;
        ball.y += ball.velY;

        // 5. Check for Collision with Ground
        if (ball.y + ball.radius > groundY) {
            
            // --- LIVES LOGIC ---
            lives--;
            livesValue.textContent = lives;
            
            if (lives <= 0) {
                endGame();
                return;
            }
            // --- END LIVES LOGIC ---
            
            // Perform the bounce: set velocity to calculated magnitude for fixed height
            ball.velY = requiredUpwardVel;
            ball.y = groundY - ball.radius; // Position above the ground
            
            // Introduce a small horizontal friction/change on ground bounce
            ball.velX *= 0.8; 
            
            return; // Exit update cycle after ground bounce
        }
        
        // 6. Check for Collision with Cat (The "Header")
        // Simple AABB (Axis-Aligned Bounding Box) collision check
        if (
            ball.x - ball.radius < cat.x + cat.width &&
            ball.x + ball.radius > cat.x &&
            ball.y + ball.radius > cat.y &&
            ball.y - ball.radius < cat.y + cat.height &&
            ball.velY > 0 // Only check if ball is moving downwards
        ) {
            // Collision detected - Perform the bounce (header)

            // Bounce the ball: set velocity to calculated magnitude for fixed height
            ball.velY = requiredUpwardVel;
            ball.y = cat.y - ball.radius; // Ensure the ball doesn't get stuck in the cat

            // 6b. Add score
            score++;
            scoreValue.textContent = score;
            
            // 6c. Horizontal deflection based on where the ball hit the cat
            const hitPoint = (ball.x - cat.x) / cat.width; // 0 (left) to 1 (right)
            const deflectionMagnitude = 12; 
            const newVelX = (hitPoint - 0.5) * deflectionMagnitude;
            
            // Introduce a small random side velocity to make it less predictable
            ball.velX = newVelX + (Math.random() - 0.5) * 5; 
        }

        // 7. Apply horizontal velocity (if any)
        if (ball.velX) {
            ball.x += ball.velX;

            // Ball boundary check for sides
            if (ball.x - ball.radius < 0 || ball.x + ball.radius > canvas.width) {
                ball.velX *= -1; // Bounce off the side walls
                ball.x = Math.max(ball.radius, Math.min(ball.x, canvas.width - ball.radius));
            }
        }
    }

    /**
     * The main game loop function.
     */
    function gameLoop() {
        if (isGameOver || !isGameRunning) return;
        
        // Check for pause state before updating logic
        if (!isPaused) {
             // Clear the canvas
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            // Draw the field and objects
            drawField();
            drawCat();
            drawBall();

            // Update logic (movement, gravity, collision)
            update();
        } else {
            // Game is paused, stop cat movement and keep drawing the current frame
            stopCat(); 
        }

        // Request the next frame regardless of pause state
        animationFrameId = requestAnimationFrame(gameLoop);
    }

    // --- Input Handling Functions ---

    /**
     * Starts moving the cat in the given direction (-1 for left, 1 for right).
     * @param {number} direction
     */
    function moveCat(direction) {
        // Prevent movement if paused, game over, or game not started
        if (isGameOver || isPaused || !isGameRunning) return; 
        catSpeed = direction;
    }

    /**
     * Stops the cat's horizontal movement.
     */
    function stopCat() {
        catSpeed = 0;
    }
    
    /**
     * Initial setup function called once on load.
     */
    function setupApp() {
        resizeCanvas();
        loadBestScore(); // Load the best score on startup
        
        // Ensure only the start screen is visible initially
        startScreen.style.display = 'flex'; 
        gameOverScreen.style.display = 'none';
        pauseScreen.style.display = 'none';
    }


    // Export controls for use in HTML buttons (ontouchstart/onmousedown)
    window.moveCat = moveCat;
    window.stopCat = stopCat;

    // Keyboard controls for desktop
    window.addEventListener('keydown', (e) => {
        const key = e.key.toLowerCase();
        
        // Use 'escape' key for pause/resume
        if (key === 'escape' && isGameRunning) {
            togglePause();
            return;
        }

        if (isGameOver || isPaused || !isGameRunning) return;
        
        if (key === 'arrowleft' || key === 'a') {
            moveCat(-1);
        } else if (key === 'arrowright' || key === 'd') {
            moveCat(1);
        }
    });

    window.addEventListener('keyup', (e) => {
        const key = e.key.toLowerCase();
        if (key === 'arrowleft' || key === 'a' || key === 'arrowright' || key === 'd') {
            stopCat();
        }
    });

    // Handle button clicks
    playButton.addEventListener('click', startGame); // New: Start the game from the main menu
    restartButton.addEventListener('click', startGame); 
    pauseButton.addEventListener('click', togglePause);

    // Initial setup and resize handler
    window.addEventListener('resize', resizeCanvas);
    window.onload = setupApp; // Show the start screen first

</script>

</body>
</html>
