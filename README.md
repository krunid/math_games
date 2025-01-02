# math_games
เกมคณิตศาสตร์


<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>เกมคณิตศาสตร์</title>
    <style>
        body {
            font-family: 'Sarabun', sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            margin: 0;
            min-height: 100vh;
            position: relative;
            background: url('/api/placeholder/1920/1080') no-repeat center center fixed;
            background-size: cover;
        }
        body::before {
            content: '';
            position: absolute;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background: rgba(255, 255, 255, 0.65);
            z-index: 0;
        }
        .game-container {
            background-color: white;
            padding: 2rem;
            border-radius: 1rem;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            margin-top: 2rem;
            text-align: center;
            max-width: 600px;
            width: 90%;
            position: relative;
            z-index: 1;
        }
        }
        .problem {
            font-size: 2rem;
            margin: 1rem 0;
            min-height: 3rem;
        }
        .timer {
            font-size: 1.5rem;
            color: #e74c3c;
            margin: 1rem 0;
        }
        .score {
            font-size: 1.25rem;
            color: #2ecc71;
            margin: 1rem 0;
        }
        input {
            font-size: 1.5rem;
            width: 100px;
            padding: 0.5rem;
            margin: 1rem 0;
            border: 2px solid #3498db;
            border-radius: 0.5rem;
            text-align: center;
        }
        button {
            font-size: 1.25rem;
            padding: 0.5rem 2rem;
            background-color: #3498db;
            color: white;
            border: none;
            border-radius: 0.5rem;
            cursor: pointer;
            transition: background-color 0.3s;
            margin: 0.5rem;
        }
        button:hover {
            background-color: #2980b9;
        }
        .feedback-overlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.5);
            display: none;
            justify-content: center;
            align-items: center;
            z-index: 1000;
        }
        .feedback-popup {
            background-color: white;
            padding: 2rem;
            border-radius: 1rem;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            text-align: center;
            font-size: 1.5rem;
            min-width: 300px;
            animation: popupAnimation 0.3s ease-out;
        }
        @keyframes popupAnimation {
            0% {
                transform: scale(0.7);
                opacity: 0;
            }
            100% {
                transform: scale(1);
                opacity: 1;
            }
        }
        .correct {
            color: #2ecc71;
        }
        .incorrect {
            color: #e74c3c;
        }
        .game-over {
            font-size: 1.5rem;
            color: #e74c3c;
            margin: 1rem 0;
            display: none;
        }
        .difficulty-btn {
            background-color: #95a5a6;
        }
        .difficulty-btn.active {
            background-color: #2ecc71;
        }
        .operations-container {
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
            gap: 0.5rem;
            margin: 1rem 0;
        }
        .operation-btn {
            flex: 0 0 auto;
            padding: 0.5rem 1rem;
        }
        .operation-btn.active {
            background-color: #2ecc71;
        }
    </style>
</head>
<body>
    <div class="game-container">
        <h1>เกมคณิตศาสตร์</h1>
        <div class="settings">
            <h3>ระดับความยาก</h3>
            <button class="difficulty-btn active" data-level="easy">ง่าย</button>
            <button class="difficulty-btn" data-level="medium">ปานกลาง</button>
            <button class="difficulty-btn" data-level="hard">ยาก</button>
            
            <h3>เลือกการคำนวณ</h3>
            <div class="operations-container">
                <button class="operation-btn active" data-op="add">บวก</button>
                <button class="operation-btn" data-op="subtract">ลบ</button>
                <button class="operation-btn" data-op="multiply">คูณ</button>
                <button class="operation-btn" data-op="divide">หาร</button>
            </div>
        </div>
        <div class="timer">เวลา: 10:00</div>
        <div class="score">คะแนน: 0</div>
        <div class="problem"></div>
        <input type="number" id="answer" placeholder="คำตอบ">
        <button id="checkAnswer">ตอบ</button>
        <button id="startBtn">เริ่มเกม</button>
        <div class="game-over"></div>
    </div>
    <div class="feedback-overlay">
        <div class="feedback-popup"></div>
    </div>

    <script>
        // เก็บตัวแปรที่ใช้อ้างอิง DOM elements
        const elements = {
            overlay: document.querySelector('.feedback-overlay'),
            popup: document.querySelector('.feedback-popup'),
            answer: document.getElementById('answer'),
            problem: document.querySelector('.problem'),
            timer: document.querySelector('.timer'),
            score: document.querySelector('.score'),
            startBtn: document.getElementById('startBtn'),
            checkAnswerBtn: document.getElementById('checkAnswer'),
            gameOver: document.querySelector('.game-over')
        };

        // ตัวแปรควบคุมเกม
        let score = 0;
        let timeLeft = 600;
        let timer;
        let currentProblem = {};
        let isGameRunning = false;
        let selectedOperations = ['add'];
        let difficulty = 'easy';

        // ฟังก์ชันแสดงผล feedback
        function showFeedback(message, type) {
            if (window.feedbackTimer) {
                clearTimeout(window.feedbackTimer);
            }

            elements.popup.textContent = message;
            elements.popup.className = `feedback-popup ${type}`;
            elements.overlay.style.display = 'flex';

            window.feedbackTimer = setTimeout(() => {
                elements.overlay.style.display = 'none';
            }, 2000);
        }

        function generateProblem() {
            const operation = selectedOperations[Math.floor(Math.random() * selectedOperations.length)];
            let num1, num2, answer;

            switch(difficulty) {
                case 'easy':
                    switch(operation) {
                        case 'add':
                            num1 = Math.floor(Math.random() * 50) + 1;
                            num2 = Math.floor(Math.random() * 50) + 1;
                            answer = num1 + num2;
                            break;
                        case 'subtract':
                            num2 = Math.floor(Math.random() * 50) + 1;
                            answer = Math.floor(Math.random() * 50) + 1;
                            num1 = num2 + answer;
                            break;
                        case 'multiply':
                            num1 = Math.floor(Math.random() * 9) + 1;
                            num2 = Math.floor(Math.random() * 9) + 1;
                            answer = num1 * num2;
                            break;
                        case 'divide':
                            num2 = Math.floor(Math.random() * 8) + 2;
                            answer = Math.floor(Math.random() * 9) + 1;
                            num1 = num2 * answer;
                            break;
                    }
                    break;
                
                case 'medium':
                    switch(operation) {
                        case 'add':
                            num1 = Math.floor(Math.random() * 500) + 1;
                            num2 = Math.floor(Math.random() * 500) + 1;
                            answer = num1 + num2;
                            break;
                        case 'subtract':
                            num2 = Math.floor(Math.random() * 500) + 1;
                            answer = Math.floor(Math.random() * 500) + 1;
                            num1 = num2 + answer;
                            break;
                        case 'multiply':
                            num1 = Math.floor(Math.random() * 90) + 10;
                            num2 = Math.floor(Math.random() * 9) + 1;
                            answer = num1 * num2;
                            break;
                        case 'divide':
                            num2 = Math.floor(Math.random() * 8) + 2;
                            answer = Math.floor(Math.random() * 90) + 10;
                            num1 = num2 * answer;
                            break;
                    }
                    break;
                
                case 'hard':
                    switch(operation) {
                        case 'add':
                            num1 = Math.floor(Math.random() * 1000) + 1;
                            num2 = Math.floor(Math.random() * 1000) + 1;
                            answer = num1 + num2;
                            break;
                        case 'subtract':
                            num2 = Math.floor(Math.random() * 1000) + 1;
                            answer = Math.floor(Math.random() * 1000) + 1;
                            num1 = num2 + answer;
                            break;
                        case 'multiply':
                            num1 = Math.floor(Math.random() * 900) + 100;
                            num2 = Math.floor(Math.random() * 9) + 1;
                            answer = num1 * num2;
                            break;
                        case 'divide':
                            num2 = Math.floor(Math.random() * 8) + 2;
                            answer = Math.floor(Math.random() * 900) + 100;
                            num1 = num2 * answer;
                            break;
                    }
                    break;
            }

            currentProblem = {
                num1: num1,
                num2: num2,
                operation: operation,
                answer: answer
            };

            const operationSymbols = {
                add: '+',
                subtract: '-',
                multiply: '×',
                divide: '÷'
            };

            elements.problem.textContent = `${num1} ${operationSymbols[operation]} ${num2} = ?`;
        }

        function checkAnswer() {
            if (!isGameRunning) return;

            const userAnswer = parseInt(elements.answer.value);
            
            if (userAnswer === currentProblem.answer) {
                score += 1;
                showFeedback('ถูกต้อง! เก่งมาก', 'correct');
            } else {
                showFeedback(`ไม่ถูกต้อง คำตอบที่ถูกคือ ${currentProblem.answer}`, 'incorrect');
            }

            elements.score.textContent = `คะแนน: ${score}`;
            elements.answer.value = '';
            generateProblem();
        }

        function updateTimer() {
            const minutes = Math.floor(timeLeft / 60);
            const seconds = timeLeft % 60;
            elements.timer.textContent = `เวลา: ${minutes}:${seconds.toString().padStart(2, '0')}`;

            if (timeLeft === 0) {
                endGame();
            } else {
                timeLeft--;
            }
        }

        function startGame() {
            if (selectedOperations.length === 0) {
                alert('กรุณาเลือกการคำนวณอย่างน้อย 1 อย่าง');
                return;
            }

            score = 0;
            timeLeft = 600;
            isGameRunning = true;
            elements.score.textContent = 'คะแนน: 0';
            elements.startBtn.style.display = 'none';
            elements.gameOver.style.display = 'none';
            elements.answer.disabled = false;
            elements.checkAnswerBtn.disabled = false;
            
            // ปิดการใช้งานปุ่มตั้งค่า
            document.querySelectorAll('.difficulty-btn, .operation-btn').forEach(btn => {
                btn.style.opacity = '0.5';
            });
            
            generateProblem();
            timer = setInterval(updateTimer, 1000);
            elements.answer.focus();
        }

        function endGame() {
            clearInterval(timer);
            isGameRunning = false;
            elements.answer.disabled = true;
            elements.checkAnswerBtn.disabled = true;
            elements.startBtn.style.display = 'block';
            
            // เปิดการใช้งานปุ่มตั้งค่า
            document.querySelectorAll('.difficulty-btn, .operation-btn').forEach(btn => {
                btn.style.opacity = '1';
            });
            
            elements.gameOver.style.display = 'block';
            elements.gameOver.innerHTML = `
                เกมจบแล้ว!<br>
                คะแนนของคุณ: ${score}<br>
                เฉลี่ย ${(score / 10).toFixed(1)} ข้อต่อนาที
            `;
        }

        // Event Listeners
        document.querySelectorAll('.difficulty-btn').forEach(btn => {
            btn.addEventListener('click', function() {
                if (!isGameRunning) {
                    document.querySelectorAll('.difficulty-btn').forEach(b => b.classList.remove('active'));
                    this.classList.add('active');
                    difficulty = this.dataset.level;
                }
            });
        });

        document.querySelectorAll('.operation-btn').forEach(btn => {
            btn.addEventListener('click', function() {
                if (!isGameRunning) {
                    this.classList.toggle('active');
                    const operation = this.dataset.op;
                    if (this.classList.contains('active')) {
                        selectedOperations.push(operation);
                    } else {
                        selectedOperations = selectedOperations.filter(op => op !== operation);
                    }
                    if (selectedOperations.length === 0) {
                        this.classList.add('active');
                        selectedOperations.push(operation);
                    }
                }
            });
        });

        elements.answer.addEventListener('keypress', function(e) {
            if (e.key === 'Enter') {
                checkAnswer();
            }
        });

        elements.startBtn.addEventListener('click', startGame);
        elements.checkAnswerBtn.addEventListener('click', checkAnswer);
    </script>
</body>
</html>
