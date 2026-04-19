[index.html](https://github.com/user-attachments/files/26876868/index.html)
<!DOCTYPE html>
<html lang="he" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>משחק זיכרון - מילים בעברית</title>
    <style>
        :root {
            --primary: #4a90e2;
            --secondary: #7ed321;
            --bg: #f0f2f5;
            --card-back: #2c3e50;
            --card-front: #ffffff;
            --text: #333;
            --white: #ffffff;
            --shadow: 0 4px 6px rgba(0,0,0,0.1);
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg);
            color: var(--text);
            margin: 0;
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
            min-height: 100vh;
            user-select: none;
        }

        h1 {
            margin-bottom: 10px;
            color: var(--primary);
            font-size: 2.5rem;
            text-shadow: 1px 1px 2px rgba(0,0,0,0.1);
        }

        .stats-container {
            background: var(--white);
            padding: 15px 30px;
            border-radius: 50px;
            box-shadow: var(--shadow);
            margin-bottom: 20px;
            display: flex;
            gap: 20px;
            font-size: 1.2rem;
            font-weight: bold;
        }

        .controls {
            margin-bottom: 30px;
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
            gap: 10px;
        }

        button {
            padding: 12px 24px;
            font-size: 1rem;
            font-weight: bold;
            border: none;
            border-radius: 12px;
            cursor: pointer;
            transition: all 0.2s transform 0.1s;
            box-shadow: var(--shadow);
            color: white;
        }

        button:active {
            transform: scale(0.95);
        }

        .btn-easy { background-color: #4caf50; }
        .btn-medium { background-color: #ff9800; }
        .btn-hard { background-color: #f44336; }
        .btn-restart { background-color: var(--primary); margin-top: 15px; }

        .grid {
            display: grid;
            gap: 12px;
            max-width: 100%;
            perspective: 1000px;
        }

        .card {
            width: 80px;
            height: 80px;
            position: relative;
            transform-style: preserve-3d;
            transition: transform 0.6s cubic-bezier(0.4, 0, 0.2, 1);
            cursor: pointer;
        }

        /* הגדלת קלפים במסכים גדולים */
        @media (min-width: 600px) {
            .card { width: 100px; height: 100px; font-size: 1.2rem; }
            .grid { gap: 15px; }
        }

        .card.flipped {
            transform: rotateY(180deg);
        }

        .card-face {
            position: absolute;
            width: 100%;
            height: 100%;
            backface-visibility: hidden;
            display: flex;
            align-items: center;
            justify-content: center;
            border-radius: 12px;
            box-shadow: var(--shadow);
            font-weight: bold;
            font-size: 1.1rem;
            padding: 5px;
            box-sizing: border-box;
        }

        .card-back {
            background-color: var(--card-back);
            background-image: radial-gradient(circle, #3e5871 10%, transparent 11%);
            background-size: 20px 20px;
            color: white;
        }

        .card-front {
            background-color: var(--card-front);
            transform: rotateY(180deg);
            color: var(--text);
            border: 2px solid var(--primary);
        }

        .card.matched .card-front {
            background-color: var(--secondary);
            color: white;
            border-color: var(--secondary);
            animation: match-pulse 0.5s ease;
        }

        @keyframes match-pulse {
            0% { transform: rotateY(180deg) scale(1); }
            50% { transform: rotateY(180deg) scale(1.1); }
            100% { transform: rotateY(180deg) scale(1); }
        }

        #winScreen {
            position: fixed;
            top: 0; left: 0;
            width: 100%; height: 100%;
            background: rgba(0,0,0,0.85);
            display: none;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            color: white;
            z-index: 1000;
            text-align: center;
            padding: 20px;
        }

        #winScreen h2 { font-size: 3rem; margin: 0; color: var(--secondary); }
        #winScreen p { font-size: 1.5rem; margin: 20px 0; }

        .confetti {
            position: fixed;
            width: 10px;
            height: 10px;
            top: -10px;
            z-index: 999;
            animation: fall linear forwards;
        }

        @keyframes fall {
            to { transform: translateY(105vh) rotate(720deg); }
        }
    </style>
</head>
<body>

    <h1>משחק זיכרון</h1>

    <div class="controls">
        <button class="btn-easy" onclick="startGame('easy')">קל (6 זוגות)</button>
        <button class="btn-medium" onclick="startGame('medium')">בינוני (9 זוגות)</button>
        <button class="btn-hard" onclick="startGame('hard')">קשה (12 זוגות)</button>
    </div>

    <div class="stats-container">
        <span>ניקוד: <span id="score">0</span></span>
        <span>מהלכים: <span id="moves">0</span></span>
    </div>

    <div class="grid" id="game"></div>

    <div id="winScreen">
        <h2>🎉 כל הכבוד! 🎉</h2>
        <p>סיימת את המשחק ב-<span id="finalMoves">0</span> מהלכים!</p>
        <p>ציון סופי: <span id="finalScore">0</span></p>
        <button class="btn-restart" onclick="hideWin(); startGame(lastLevel)">שחק שוב</button>
    </div>

    <audio id="successSound" preload="auto">
        <source src="https://www.soundjay.com/buttons/sounds/button-4.mp3" type="audio/mpeg">
    </audio>

    <script>
        const words = ["חַלָּה","נָחָשׁ","בָּטָטָה","שָׁנָה","בַּת","כַּתָּב","זָנָב","דָּג","בַּלָּשׁ","פָּרָה","אַבָּא","בָּנָנָה"];
        
        let firstCard = null;
        let secondCard = null;
        let lockBoard = false;
        let score = 0;
        let moves = 0;
        let matchedCount = 0;
        let totalPairs = 0;
        let lastLevel = 'easy';

        function shuffle(array) {
            let currentIndex = array.length;
            while (currentIndex != 0) {
                let randomIndex = Math.floor(Math.random() * currentIndex);
                currentIndex--;
                [array[currentIndex], array[randomIndex]] = [array[randomIndex], array[currentIndex]];
            }
            return array;
        }

        function startGame(level) {
            lastLevel = level;
            let pairsCount = 6;
            if(level === 'medium') pairsCount = 9;
            if(level === 'hard') pairsCount = 12;

            totalPairs = pairsCount;
            matchedCount = 0;
            score = 0;
            moves = 0;
            firstCard = null;
            secondCard = null;
            lockBoard = false;

            document.getElementById("score").textContent = "0";
            document.getElementById("moves").textContent = "0";
            
            const gameGrid = document.getElementById("game");
            gameGrid.innerHTML = "";

            // קביעת מבנה הגריד לפי מספר הקלפים
            let cols = 4;
            if (pairsCount > 6) cols = window.innerWidth < 600 ? 4 : 6;
            gameGrid.style.gridTemplateColumns = `repeat(${cols}, auto)`;

            const selectedWords = words.slice(0, pairsCount);
            const gameCards = shuffle([...selectedWords, ...selectedWords]);

            gameCards.forEach(word => {
                const card = document.createElement("div");
                card.classList.add("card");
                card.dataset.word = word;

                card.innerHTML = `
                    <div class="card-face card-back">?</div>
                    <div class="card-face card-front">${word}</div>
                `;

                card.addEventListener("click", flipCard);
                gameGrid.appendChild(card);
            });
        }

        function flipCard() {
            if (lockBoard) return;
            if (this === firstCard) return;
            if (this.classList.contains('matched')) return;

            this.classList.add('flipped');

            if (!firstCard) {
                firstCard = this;
                return;
            }

            secondCard = this;
            moves++;
            document.getElementById("moves").textContent = moves;
            checkForMatch();
        }

        function checkForMatch() {
            let isMatch = firstCard.dataset.word === secondCard.dataset.word;
            isMatch ? disableCards() : unflipCards();
        }

        function disableCards() {
            lockBoard = true;
            setTimeout(() => {
                firstCard.classList.add('matched');
                secondCard.classList.add('matched');
                
                document.getElementById("successSound").currentTime = 0;
                document.getElementById("successSound").play().catch(() => {});
                
                score += 10;
                matchedCount++;
                document.getElementById("score").textContent = score;
                
                createConfetti();
                resetBoard();
                
                if (matchedCount === totalPairs) {
                    showWin();
                }
            }, 500);
        }

        function unflipCards() {
            lockBoard = true;
            setTimeout(() => {
                firstCard.classList.remove('flipped');
                secondCard.classList.remove('flipped');
                score = Math.max(0, score - 1);
                document.getElementById("score").textContent = score;
                resetBoard();
            }, 1000);
        }

        function resetBoard() {
            [firstCard, secondCard] = [null, null];
            lockBoard = false;
        }

        function createConfetti() {
            const colors = ['#f44336', '#e91e63', '#9c27b0', '#673ab7', '#3f51b5', '#2196f3', '#03a9f4', '#00bcd4', '#009688', '#4caf50', '#8bc34a', '#cddc39', '#ffeb3b', '#ffc107', '#ff9800', '#ff5722'];
            for (let i = 0; i < 15; i++) {
                const confetti = document.createElement('div');
                confetti.classList.add('confetti');
                confetti.style.left = Math.random() * 100 + 'vw';
                confetti.style.backgroundColor = colors[Math.floor(Math.random() * colors.length)];
                confetti.style.width = Math.random() * 10 + 5 + 'px';
                confetti.style.height = confetti.style.width;
                confetti.style.animationDuration = Math.random() * 2 + 1 + 's';
                confetti.style.opacity = Math.random();
                document.body.appendChild(confetti);
                setTimeout(() => confetti.remove(), 3000);
            }
        }

        function showWin() {
            setTimeout(() => {
                document.getElementById("finalMoves").textContent = moves;
                document.getElementById("finalScore").textContent = score;
                document.getElementById("winScreen").style.display = "flex";
                // הרבה קונפטי בניצחון
                for(let i=0; i<5; i++) setTimeout(createConfetti, i * 300);
            }, 600);
        }

        function hideWin() {
            document.getElementById("winScreen").style.display = "none";
        }

        // התחלת משחק ראשון
        window.onload = () => startGame('easy');
    </script>
</body>
</html>
