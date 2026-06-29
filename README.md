<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Past Tense Motion Quest</title>
    <!-- MediaPipe Holistic (จับทั้งตัว หน้า และมือ) -->
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/control_utils/control_utils.js" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/drawing_utils/drawing_utils.js" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/holistic/holistic.js" crossorigin="anonymous"></script>
    
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            user-select: none;
        }

        body {
            background-color: #000;
            overflow: hidden;
            width: 100vw;
            height: 100vh;
            color: white;
        }

        #game-container {
            position: relative;
            width: 100vw;
            height: 100vh;
            overflow: hidden;
        }

        /* กล้องวิดีโอ (ซ่อนไว้ ใช้ดึงภาพเฉยๆ) */
        #webcam {
            display: none;
        }

        /* ภาพผลลัพธ์จากกล้อง (สลับซ้ายขวาให้เป็นกระจกใน CSS ที่เดียว) */
        #output_canvas {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            object-fit: cover;
            transform: scaleX(-1); 
            z-index: 1;
        }

        /* เลเยอร์ UI ทั่วไป */
        .screen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: 10;
            display: none;
            flex-direction: column;
            justify-content: center;
            align-items: center;
        }

        .active {
            display: flex;
        }

        /* Scene 1: Start Screen */
        #start-screen {
            background-image: url('https://lh3.googleusercontent.com/d/102UT5mwKcZ1-s1r2E71OHaSWgkNToUGj');
            background-size: cover;
            background-position: center;
            z-index: 50;
        }
        
        #btn-start {
            position: absolute;
            bottom: 15%;
            padding: 15px 50px;
            font-size: 2rem;
            font-weight: bold;
            color: white;
            background: linear-gradient(45deg, #ff5722, #ff9800);
            border: none;
            border-radius: 50px;
            cursor: pointer;
            box-shadow: 0 10px 20px rgba(0,0,0,0.5);
            transition: transform 0.2s;
        }

        #btn-start:hover { transform: scale(1.1); }

        /* Scene 2: Calibration */
        #calibration-screen {
            background: rgba(0,0,0,0.7);
        }

        .progress-container {
            width: 60%;
            height: 40px;
            background: #333;
            border-radius: 20px;
            border: 4px solid #fff;
            overflow: hidden;
            margin-top: 20px;
        }

        #calib-bar {
            width: 0%;
            height: 100%;
            background: #4CAF50;
            transition: width 0.3s;
        }

        /* Scene 3: Main Game */
        #hud {
            position: absolute;
            top: 20px;
            left: 0;
            width: 100%;
            padding: 0 40px;
            display: flex;
            justify-content: space-between;
            font-size: 2rem;
            font-weight: bold;
            text-shadow: 2px 2px 4px #000;
            z-index: 20;
            pointer-events: none;
        }

        #question-text {
            position: absolute;
            top: 10%;
            width: 100%;
            text-align: center;
            font-size: 5rem;
            font-weight: bold;
            color: #FFEB3B;
            text-shadow: 4px 4px 8px #000;
            z-index: 15;
            pointer-events: none;
        }

        /* กล่องคำตอบให้เล็กลง 50% และเตรียมลอยไปมา */
        .choice-btn {
            position: absolute;
            top: 50%;
            transform: translateY(-50%) scale(0.5); 
            padding: 40px 60px;
            font-size: 3.5rem;
            font-weight: bold;
            color: white;
            background: rgba(33, 150, 243, 0.9);
            border: 8px solid white;
            border-radius: 30px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.5);
            z-index: 15;
            pointer-events: none;
            transition: background 0.3s;
        }

        #choice-left { left: 2%; }
        #choice-right { right: 2%; }

        .correct { background: #4CAF50 !important; }
        .wrong { background: #F44336 !important; }

        /* Scene 4: Mini Game (ระบบวงกลม) */
        .target-circle {
            position: absolute;
            width: 90px;
            height: 90px;
            border-radius: 50%;
            background: radial-gradient(circle at 30% 30%, #00e5ff, #005cbf);
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 2.5rem;
            z-index: 25;
            box-shadow: 0 0 20px rgba(0, 229, 255, 0.8);
            border: 4px solid white;
            animation: popIn 0.3s ease-out;
        }

        /* วงกลมแบบพิเศษที่วิ่งหนีได้ */
        .target-circle.evasive {
            background: radial-gradient(circle at 30% 30%, #ff5252, #b71c1c);
            box-shadow: 0 0 20px rgba(255, 82, 82, 0.8);
        }

        @keyframes popIn {
            0% { transform: scale(0); }
            80% { transform: scale(1.2); }
            100% { transform: scale(1); }
        }

        /* เลเยอร์ปลดล็อกเสียงหน้าปก */
        #interaction-overlay {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.95);
            color: white;
            z-index: 999;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 3rem;
            cursor: pointer;
            text-align: center;
            flex-direction: column;
        }

        #minigame-alert {
            position: absolute;
            top: 40%;
            width: 100%;
            text-align: center;
            font-size: 4rem;
            color: #FF5252;
            font-weight: bold;
            text-shadow: 3px 3px 8px black;
            z-index: 30;
            animation: blink 0.8s infinite;
            display: none;
        }

        @keyframes blink { 50% { opacity: 0; } }

        /* Result Screen */
        #result-screen {
            background: rgba(0,0,0,0.9);
        }

        .trophy {
            font-size: 10rem;
            margin-bottom: 20px;
            animation: bounce 2s infinite;
        }

        @keyframes bounce {
            0%, 100% { transform: translateY(0); }
            50% { transform: translateY(-20px); }
        }

        /* Hand Cursor */
        .hand-cursor {
            position: absolute;
            width: 40px;
            height: 40px;
            background: rgba(255, 255, 255, 0.8);
            border: 4px solid #00E5FF;
            border-radius: 50%;
            pointer-events: none;
            z-index: 100;
            transform: translate(-50%, -50%);
            display: none;
            box-shadow: 0 0 15px #00E5FF;
        }
    </style>
</head>
<body>

    <div id="game-container">
        <!-- หน้าจอคลิกเพื่อปลดล็อกเสียง -->
        <div id="interaction-overlay">
            <div style="font-size: 5rem; margin-bottom: 20px;">👆</div>
            <h1>CLICK ANYWHERE TO ENTER</h1>
        </div>

        <!-- Audio Elements -->
        <audio id="bgm-title" loop preload="auto">
            <source src="https://raw.githubusercontent.com/nayklx250-dot/my-game/d791192c28b8ef69421283e4a6822ca88cd9373b/juliush-funny-organ-intro-outro-5008.mp3" type="audio/mpeg">
        </audio>
        <audio id="bgm-main" loop preload="auto">
            <source src="https://raw.githubusercontent.com/nayklx250-dot/my-game/2a5011053ec64ccc77c4e90f8136c87713150628/grand_project-funny-running-129223.mp3" type="audio/mpeg">
        </audio>
        <audio id="bgm-mini" loop preload="auto">
            <source src="https://raw.githubusercontent.com/nayklx250-dot/my-game/2a5011053ec64ccc77c4e90f8136c87713150628/paulyudin-fun-fun-fun-fun-fun-152388.mp3" type="audio/mpeg">
        </audio>
        <audio id="sfx-correct" src="https://assets.mixkit.co/active_storage/sfx/2000/2000-preview.mp3"></audio>
        <audio id="sfx-wrong" src="https://assets.mixkit.co/active_storage/sfx/2003/2003-preview.mp3"></audio>
        <audio id="sfx-pop" src="https://assets.mixkit.co/active_storage/sfx/2771/2771-preview.mp3"></audio>

        <!-- Camera & Canvas -->
        <video id="webcam" autoplay playsinline></video>
        <canvas id="output_canvas"></canvas>

        <!-- Hand Cursors -->
        <div id="cursor-left" class="hand-cursor"></div>
        <div id="cursor-right" class="hand-cursor"></div>

        <!-- 1. Start Screen -->
        <div id="start-screen" class="screen active">
            <button id="btn-start">START</button>
        </div>

        <!-- 2. Calibration Screen -->
        <div id="calibration-screen" class="screen">
            <h1>PLEASE STAND IN FRONT OF THE CAMERA</h1>
            <h2 id="calib-text">DETECTING FACE AND BODY... 0%</h2>
            <div class="progress-container">
                <div id="calib-bar"></div>
            </div>
        </div>

        <!-- 3. Main Game / HUD -->
        <div id="hud" style="display: none;">
            <div>SCORE: <span id="score">0</span></div>
            <div>TIME: <span id="time">180</span></div>
        </div>
        
        <div id="game-screen" class="screen">
            <div id="question-text">Loading...</div>
            <div id="choice-left" class="choice-btn">A</div>
            <div id="choice-right" class="choice-btn">B</div>
        </div>

        <!-- 4. Mini Game -->
        <div id="minigame-alert">MINI GAME!<br>TOUCH THE CIRCLES FAST!</div>
        <div id="minigame-screen" class="screen">
            <!-- วัตถุจะถูกสร้างผ่าน JS -->
        </div>

        <!-- 5. Result Screen -->
        <div id="result-screen" class="screen">
            <div class="trophy">🏆</div>
            <h1>TIME'S UP!</h1>
            <h2 style="margin-top: 20px; font-size: 3rem;">FINAL SCORE: <span id="final-score">0</span></h2>
            <button id="btn-restart" style="margin-top: 40px; padding: 15px 40px; font-size: 1.5rem; cursor:pointer;">PLAY AGAIN</button>
        </div>
    </div>

    <script>
        // --- ฐานข้อมูลข้อสอบ 32 ข้อ ---
        const questionBank = [
            {q: "is", c: ["was", "were"], a: "was"},
            {q: "come", c: ["comed", "came"], a: "came"},
            {q: "see", c: ["seed", "saw"], a: "saw"},
            {q: "talk", c: ["talked", "took"], a: "talked"},
            {q: "love", c: ["lost", "loved"], a: "loved"},
            {q: "want", c: ["wanted", "wold"], a: "wanted"},
            {q: "ask", c: ["as", "asked"], a: "asked"},
            {q: "return", c: ["returned", "repaeat"], a: "returned"},
            {q: "have", c: ["had", "has"], a: "had"},
            {q: "give", c: ["gived", "gave"], a: "gave"},
            {q: "meet", c: ["met", "meeted"], a: "met"},
            {q: "take", c: ["took", "taked"], a: "took"},
            {q: "say", c: ["sayed", "said"], a: "said"},
            {q: "begin", c: ["began", "begined"], a: "began"},
            {q: "hit", c: ["hit", "hitted"], a: "hit"},
            {q: "sail", c: ["sailed", "said"], a: "sailed"},
            {q: "stop", c: ["stood", "stopped"], a: "stopped"},
            {q: "fall", c: ["felled", "fell"], a: "fell"}, 
            {q: "hear", c: ["heard", "heared"], a: "heard"},
            {q: "miss", c: ["missed", "might"], a: "missed"},
            {q: "arrive", c: ["arrived", "left"], a: "arrived"},
            {q: "load", c: ["lounding", "loaded"], a: "loaded"},
            {q: "start", c: ["starting", "started"], a: "started"},
            {q: "pull", c: ["pulled", "pold"], a: "pulled"},
            {q: "shine", c: ["shone", "shied"], a: "shone"}, 
            {q: "are", c: ["was", "were"], a: "were"},
            {q: "cry", c: ["cried", "could"], a: "cried"},
            {q: "name", c: ["named", "meet"], a: "named"},
            {q: "buy", c: ["buyed", "bought"], a: "bought"},
            {q: "sell", c: ["selled", "sold"], a: "sold"},
            {q: "go", c: ["went", "goed"], a: "went"},
            {q: "play", c: ["plyed", "played"], a: "played"}
        ];

        // --- ตัวแปรระบบ ---
        let gameState = "START"; 
        let score = 0;
        let timeMain = 180; // ปรับเวลาเป็น 3 นาที
        let calibPercent = 0;
        let currentQuestion = null;
        let gameTimer = null;
        let minigameTimer = null;
        let minigameTimeLeft = 15; // มินิเกม 15 วิ
        let objects = []; 
        let lastTouchTime = 0;

        // ตัวแปรเช็คมินิเกม 3 รอบ (120, 60, 30 วิ)
        let miniGamePlayed120 = false;
        let miniGamePlayed60 = false;
        let miniGamePlayed30 = false;

        // ตัวแปรสำหรับช้อยส์เคลื่อนที่ (Floating Choices)
        let choiceL_Y = 50; 
        let choiceR_Y = 50;
        let choiceL_Dir = 0.3; 
        let choiceR_Dir = -0.4; 

        // --- อ้างอิง DOM ---
        const videoElement = document.getElementById('webcam');
        const canvasElement = document.getElementById('output_canvas');
        const canvasCtx = canvasElement.getContext('2d');
        const cursorLeft = document.getElementById('cursor-left');
        const cursorRight = document.getElementById('cursor-right');
        
        const audioMain = document.getElementById('bgm-main');
        const audioMini = document.getElementById('bgm-mini');
        
        // --- ฟังก์ชันช่วยเหลือ ---
        const switchScreen = (screenId) => {
            document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
            if(screenId) document.getElementById(screenId).classList.add('active');
        };

        const playSound = (id) => {
            const s = document.getElementById(id);
            s.currentTime = 0;
            s.play().catch(e=>console.log(e));
        };

        // เริ่มต้นการเลื่อนตำแหน่งปุ่มช้อยส์คำตอบให้ลอยไปมา
        const animateChoices = () => {
            if(gameState === "PLAY") {
                choiceL_Y += choiceL_Dir;
                if(choiceL_Y > 80 || choiceL_Y < 30) choiceL_Dir *= -1; 
                document.getElementById('choice-left').style.top = choiceL_Y + '%';

                choiceR_Y += choiceR_Dir;
                if(choiceR_Y > 80 || choiceR_Y < 30) choiceR_Dir *= -1;
                document.getElementById('choice-right').style.top = choiceR_Y + '%';
            }
            requestAnimationFrame(animateChoices);
        };
        animateChoices(); // สั่งรันลูปอนิเมชันทันที

        // --- ระบบเกมหลัก ---
        // 1. ปลดล็อกหน้าจอแรกเพื่อเล่นเพลงหน้าปก
        document.getElementById('interaction-overlay').addEventListener('click', function() {
            this.style.display = 'none';
            const titleBgm = document.getElementById('bgm-title');
            titleBgm.volume = 0.5;
            titleBgm.play().catch(e => console.log(e));
        });

        document.getElementById('btn-start').addEventListener('click', () => {
            // ปิดเพลงหน้าปกเมื่อกด START
            document.getElementById('bgm-title').pause();

            // ปลดล็อกการเล่นเสียงของ Browser 
            audioMain.volume = 0; 
            audioMain.play().then(() => audioMain.pause()).catch(e=>{});
            audioMini.volume = 0;
            audioMini.play().then(() => audioMini.pause()).catch(e=>{});

            gameState = "CALIB";
            switchScreen('calibration-screen');
            initCamera();
        });

        document.getElementById('btn-restart').addEventListener('click', () => {
            location.reload();
        });

        const startMainGame = () => {
            gameState = "PLAY";
            score = 0;
            timeMain = 180; // รีเซ็ตเวลาที่ 3 นาที
            
            // รีเซ็ตสถานะมินิเกมเมื่อเริ่มใหม่
            miniGamePlayed120 = false;
            miniGamePlayed60 = false;
            miniGamePlayed30 = false;
            
            document.getElementById('score').innerText = score;
            document.getElementById('hud').style.display = 'flex';
            switchScreen('game-screen');
            
            audioMain.volume = 0.5;
            audioMain.currentTime = 0;
            audioMain.play().catch(e=>console.log("Audio play blocked", e));
            
            nextQuestion();

            gameTimer = setInterval(() => {
                if(gameState === "PLAY" || gameState === "RELAY") {
                    timeMain--;
                    document.getElementById('time').innerText = timeMain;
                    
                    // ระบบทริกเกอร์มินิเกม 3 รอบ
                    if(timeMain === 120 && !miniGamePlayed120 && gameState !== "MINIGAME") {
                        miniGamePlayed120 = true;
                        startMiniGame();
                    } else if (timeMain === 60 && !miniGamePlayed60 && gameState !== "MINIGAME") {
                        miniGamePlayed60 = true;
                        startMiniGame();
                    } else if (timeMain === 30 && !miniGamePlayed30 && gameState !== "MINIGAME") {
                        miniGamePlayed30 = true;
                        startMiniGame();
                    } else if (timeMain <= 0) {
                        endGame();
                    }
                }
            }, 1000);
        };

        const nextQuestion = () => {
            if(gameState !== "PLAY") return;
            
            let qIndex = Math.floor(Math.random() * questionBank.length);
            currentQuestion = questionBank[qIndex];
            
            document.getElementById('question-text').innerText = currentQuestion.q;
            
            let btnL = document.getElementById('choice-left');
            let btnR = document.getElementById('choice-right');
            
            btnL.className = "choice-btn";
            btnR.className = "choice-btn";

            if(Math.random() > 0.5) {
                btnL.innerText = currentQuestion.c[0];
                btnR.innerText = currentQuestion.c[1];
            } else {
                btnL.innerText = currentQuestion.c[1];
                btnR.innerText = currentQuestion.c[0];
            }
        };

        const checkAnswer = (selectedText, element) => {
            if(gameState !== "PLAY") return;
            
            gameState = "RELAY"; // ล็อกเกมเพลย์ชั่วคราว
            
            if(selectedText.trim() === currentQuestion.a) {
                element.classList.add('correct');
                score++;
                document.getElementById('score').innerText = score;
                playSound('sfx-correct');
            } else {
                element.classList.add('wrong');
                playSound('sfx-wrong');
            }

            setTimeout(() => {
                if(gameState === "RELAY") {
                    gameState = "PLAY";
                    nextQuestion();
                }
            }, 2000);
        };

        // --- ระบบมินิเกม (แตะวงกลมเพิ่มเวลา) ---
        const startMiniGame = () => {
            gameState = "MINIGAME";
            document.getElementById('minigame-alert').style.display = 'block';
            switchScreen('minigame-screen');
            
            // สลับเพลง
            audioMain.pause();
            audioMini.volume = 1.0; 
            audioMini.currentTime = 0;
            audioMini.play().catch(e=>console.log(e));

            minigameTimeLeft = 15; // มินิเกมให้เวลา 15 วิ
            objects = [];
            
            let spawnInterval = setInterval(() => {
                if(gameState !== "MINIGAME") {
                    clearInterval(spawnInterval);
                    return;
                }
                spawnCircle(); // ใช้ฟังก์ชันสุ่มวงกลมขอบจอ
            }, 600); 

            let moveInterval = setInterval(() => {
                if(gameState !== "MINIGAME") {
                    clearInterval(moveInterval);
                    return;
                }
                updateMiniGameObjects();
            }, 50);

            minigameTimer = setInterval(() => {
                minigameTimeLeft--;
                if(minigameTimeLeft <= 0) {
                    clearInterval(minigameTimer);
                    endMiniGame();
                }
            }, 1000);
        };

        const spawnCircle = () => {
            const el = document.createElement('div');
            const isEvasive = Math.random() < 0.4; // โอกาส 40% ที่วงกลมจะหนีมือ
            el.className = `target-circle ${isEvasive ? 'evasive' : ''}`;
            el.innerText = '⭕';
            
            // สุ่มเกิดตามขอบ (ซ้าย ขวา บน ล่าง)
            let edge = Math.floor(Math.random() * 4);
            let x = 0, y = 0, margin = 100;
            
            if (edge === 0) { // ซ้าย
                x = margin + Math.random() * 50;
                y = margin + Math.random() * (window.innerHeight - margin * 2);
            } else if (edge === 1) { // ขวา
                x = window.innerWidth - margin - Math.random() * 50 - 90;
                y = margin + Math.random() * (window.innerHeight - margin * 2);
            } else if (edge === 2) { // บน
                x = margin + Math.random() * (window.innerWidth - margin * 2);
                y = margin + Math.random() * 50;
            } else { // ล่าง
                x = margin + Math.random() * (window.innerWidth - margin * 2);
                y = window.innerHeight - margin - Math.random() * 50 - 90;
            }
            
            el.style.left = x + 'px';
            el.style.top = y + 'px';
            document.getElementById('minigame-screen').appendChild(el);
            
            let objData = {
                element: el, x: x, y: y,
                type: 'circle', isEvasive: isEvasive,
                speedX: 0, speedY: 0
            };
            objects.push(objData);
            
            // วงกลมหายไปเองถ้าไม่จับภายใน 3.5 วินาที
            setTimeout(() => {
                if (el.parentNode) {
                    el.remove();
                    objects = objects.filter(o => o !== objData);
                }
            }, 3500);
        };

        const updateMiniGameObjects = () => {
            const hL = cursorLeft.getBoundingClientRect();
            const hR = cursorRight.getBoundingClientRect();

            for(let i = objects.length - 1; i >= 0; i--) {
                let obj = objects[i];
                
                // ถ้าระบบบอกว่าเป็นวงที่หนีมือได้
                if(obj.isEvasive) {
                    [hL, hR].forEach(hand => {
                        if(hand.width > 0) {
                            let hX = hand.left + hand.width/2;
                            let hY = hand.top + hand.height/2;
                            let oX = obj.x + 45; // รัศมี 45 (กว้าง 90)
                            let oY = obj.y + 45;
                            
                            let dist = Math.hypot(hX - oX, hY - oY);
                            // หนีทันทีถ้าระยะใกล้มือ (150px)
                            if(dist < 150) { 
                                obj.speedX = (oX > hX) ? 14 : -14; 
                                obj.speedY = (oY > hY) ? 14 : -14;
                            }
                        }
                    });
                }

                obj.y += obj.speedY;
                obj.x += obj.speedX;
                
                // แรงเสียดทานชะลอความเร็ว
                obj.speedX *= 0.85;
                obj.speedY *= 0.85;

                // ชนขอบแล้วเด้งกลับ ไม่ให้หลุดจอ
                if(obj.x < 0) { obj.x = 0; obj.speedX *= -1; }
                if(obj.x > window.innerWidth - 90) { obj.x = window.innerWidth - 90; obj.speedX *= -1; }
                if(obj.y < 0) { obj.y = 0; obj.speedY *= -1; }
                if(obj.y > window.innerHeight - 90) { obj.y = window.innerHeight - 90; obj.speedY *= -1; }

                obj.element.style.top = obj.y + 'px';
                obj.element.style.left = obj.x + 'px';
            }
        };

        const endMiniGame = () => {
            document.getElementById('minigame-alert').style.display = 'none';
            document.getElementById('minigame-screen').innerHTML = ''; 
            
            audioMini.pause();
            audioMain.play().catch(e=>console.log(e));
            
            gameState = "PLAY";
            switchScreen('game-screen');
        };

        const endGame = () => {
            gameState = "END";
            clearInterval(gameTimer);
            document.getElementById('hud').style.display = 'none';
            document.getElementById('final-score').innerText = score;
            switchScreen('result-screen');
            audioMain.pause();
        };

        // --- ระบบ MediaPipe Holistic ---
        function onResults(results) {
            canvasElement.width = window.innerWidth;
            canvasElement.height = window.innerHeight;
            canvasCtx.save();
            canvasCtx.clearRect(0, 0, canvasElement.width, canvasElement.height);
            
            canvasCtx.drawImage(results.image, 0, 0, canvasElement.width, canvasElement.height);

            if(gameState === "CALIB") {
                if(results.poseLandmarks && results.faceLandmarks) {
                    calibPercent += 2;
                    if(calibPercent > 100) calibPercent = 100;
                } else {
                    calibPercent -= 1;
                    if(calibPercent < 0) calibPercent = 0;
                }
                
                document.getElementById('calib-bar').style.width = calibPercent + "%";
                document.getElementById('calib-text').innerText = `DETECTING FACE AND BODY... ${calibPercent}%`;

                if(calibPercent >= 100) {
                    startMainGame();
                }
            }
            canvasCtx.restore();

            updateHandCursor(cursorLeft, results.leftHandLandmarks);
            updateHandCursor(cursorRight, results.rightHandLandmarks);
        }

        const updateHandCursor = (cursorEl, handLandmarks) => {
            if(handLandmarks) {
                const indexFinger = handLandmarks[8];
                const x = (1 - indexFinger.x) * window.innerWidth;
                const y = indexFinger.y * window.innerHeight;
                
                cursorEl.style.display = 'block';
                cursorEl.style.left = x + 'px';
                cursorEl.style.top = y + 'px';

                checkCollisions(cursorEl);
            } else {
                cursorEl.style.display = 'none';
            }
        };

        // --- ระบบตรวจจับการชน (Collision Detection AABB) ---
        const checkCollisions = (cursorEl) => {
            let now = Date.now();
            if(now - lastTouchTime < 500) return; // กันแตะรัว

            const cursorRect = cursorEl.getBoundingClientRect();
            
            if(gameState === "PLAY") {
                const btnL = document.getElementById('choice-left');
                const btnR = document.getElementById('choice-right');
                
                if(isColliding(cursorRect, btnL.getBoundingClientRect())) {
                    lastTouchTime = now;
                    checkAnswer(btnL.innerText, btnL);
                } else if(isColliding(cursorRect, btnR.getBoundingClientRect())) {
                    lastTouchTime = now;
                    checkAnswer(btnR.innerText, btnR);
                }
            } 
            else if (gameState === "MINIGAME") {
                for(let i = objects.length - 1; i >= 0; i--) {
                    let obj = objects[i];
                    if(isColliding(cursorRect, obj.element.getBoundingClientRect())) {
                        lastTouchTime = now;
                        
                        if(obj.type === 'circle') {
                            timeMain += 1; // เพิ่มเวลาให้ 1 วินาทีเมื่อแตะโดน
                            playSound('sfx-pop');
                        }
                        
                        document.getElementById('time').innerText = timeMain;
                        obj.element.remove();
                        objects.splice(i, 1);
                    }
                }
            }
        };

        const isColliding = (rect1, rect2) => {
            return !(rect1.right < rect2.left || 
                     rect1.left > rect2.right || 
                     rect1.bottom < rect2.top || 
                     rect1.top > rect2.bottom);
        };

        // --- เริ่มต้นระบบกล้องและ MediaPipe ---
        const initCamera = () => {
            const holistic = new Holistic({locateFile: (file) => {
                return `https://cdn.jsdelivr.net/npm/@mediapipe/holistic/${file}`;
            }});
            
            holistic.setOptions({
                modelComplexity: 1,
                smoothLandmarks: true,
                minDetectionConfidence: 0.5,
                minTrackingConfidence: 0.5
            });
            
            holistic.onResults(onResults);
            
            const camera = new Camera(videoElement, {
                onFrame: async () => {
                    await holistic.send({image: videoElement});
                },
                width: 1280,
                height: 720
            });
            camera.start();
        };
    </script>
</body>
</html>
