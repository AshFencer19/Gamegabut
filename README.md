# Gamegabut
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Kejar Emoji Lucu!</title>
    <style>
        /* --- CSS untuk Styling Umum --- */
       body {
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            background: linear-gradient(135deg, #a8e063, #56ab2f); /* Latar belakang ceria */
            margin: 0;
            font-family: 'Comic Sans MS', cursive, sans-serif;
            color: #333;
            overflow: hidden; /* Penting untuk emoji yang bergerak */
            user-select: none; /* Mencegah seleksi teks */
            cursor: default;
        }

        .game-container {
            background-color: rgba(255, 255, 255, 0.9);
            padding: 30px 50px;
            border-radius: 20px;
            box-shadow: 0 15px 30px rgba(0, 0, 0, 0.3);
            text-align: center;
            max-width: 800px;
            width: 90%;
            position: relative;
            z-index: 10; /* Pastikan di atas emoji */
        }

        h1 {
            color: #ff69b4; /* Warna judul lucu */
            font-size: 3em;
            margin-bottom: 10px;
            text-shadow: 2px 2px 5px rgba(0,0,0,0.1);
        }

        p {
            font-size: 1.2em;
            margin-bottom: 20px;
        }

        /* Ini adalah gaya untuk skor/waktu di LAYAR UTAMA (sebelum/setelah game) */
        .score-board, .high-score {
            font-size: 1.8em;
            font-weight: bold;
            color: #007bff;
            margin-top: 15px;
        }

        .input-group { /* Menghapus .level-select-group dari sini karena dropdown dihapus */
            margin-bottom: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        .input-group label { /* Menghapus .level-select-group label dari sini */
            font-size: 1.3em;
            margin-bottom: 8px;
            color: #333;
        }

        .input-group input[type="text"] { /* Menghapus .level-select-group select dari sini */
            padding: 10px 15px;
            font-size: 1.2em;
            border: 2px solid #007bff;
            border-radius: 10px;
            text-align: center;
            width: 70%;
            max-width: 300px;
            outline: none;
            transition: border-color 0.2s ease;
            background-color: #f8f9fa; /* Background untuk select */
            cursor: pointer;
        }

        .input-group input[type="text"]:focus { /* Menghapus .level-select-group select:focus dari sini */
            border-color: #ff69b4;
        }

        button {
            background-color: #ffd700; /* Kuning cerah */
            color: #333;
            padding: 15px 30px;
            border: none;
            border-radius: 30px;
            font-size: 1.5em;
            cursor: pointer;
            transition: transform 0.2s ease, background-color 0.2s ease;
            box-shadow: 0 5px 15px rgba(0,0,0,0.2);
            margin-top: 20px;
        }

        button:hover {
            transform: translateY(-3px);
            background-color: #ffec8b;
        }

        button:active {
            transform: translateY(1px);
        }

        /* --- CSS untuk Score SAAT game berjalan (pojok atas) --- */
        #game-info-overlay {
            position: fixed;
            top: 20px;
            width: 96%; /* Hampir memenuhi lebar layar */
            display: flex;
            justify-content: space-between; /* Untuk menata skor, level, dan nyawa */
            align-items: center;
            padding: 0 2%; /* Sedikit padding di kiri dan kanan */
            z-index: 20; /* Pastikan di atas emoji dan container utama */
            color: #fff; /* Warna teks putih agar kontras dengan background */
            font-size: 2.5em; /* Ukuran lebih besar agar jelas */
            font-weight: bold;
            text-shadow: 2px 2px 5px rgba(0,0,0,0.4);
            pointer-events: none; /* Penting agar tidak mengganggu klik emoji */
            visibility: hidden; /* Sembunyikan secara default */
            opacity: 0;
            transition: opacity 0.5s ease, visibility 0s 0.5s; /* Transisi untuk muncul/hilang */
        }

        #game-info-overlay.show {
            visibility: visible;
            opacity: 1;
            transition: opacity 0.5s ease; /* Transisi untuk muncul */
        }

        .display-in-game { /* Kelas umum untuk skor dan level */
            background-color: rgba(0, 0, 0, 0.4); /* Background semi-transparan */
            padding: 10px 20px;
            border-radius: 15px;
            min-width: 150px; /* Lebar minimum */
            text-align: center;
            border: 2px solid #ffd700; /* Border kuning */
        }

        /* Gaya khusus untuk tampilan nyawa - akan menampilkan ikon hati */
        #lives-in-game {
            color: #ff4d4d; /* Warna merah untuk hati */
            font-size: 1em; /* Ukuran hati agar proporsional */
        }

        /* --- CSS untuk Emoji yang Bergerak --- */
        .moving-emoji {
            position: absolute;
            font-size: 4em; /* Ukuran emoji besar */
            cursor: pointer;
            z-index: 5; /* Di bawah container game */
            opacity: 0; /* Awalnya tersembunyi */
            /* Animasi akan diatur via JS untuk durasi dinamis */
        }
        
        /* Gaya khusus untuk emoji bom */
        .moving-emoji.bomb {
            font-size: 4.5em; /* Bom sedikit lebih besar */
            filter: drop-shadow(0 0 8px rgba(255, 0, 0, 0.8)); /* Efek cahaya merah */
        }

        /* Animasi fade in dan fade out untuk emoji */
        @keyframes fadeInOut {
            0% { opacity: 0; transform: scale(0.5); }
            20% { opacity: 1; transform: scale(1); }
            80% { opacity: 1; transform: scale(1); }
            100% { opacity: 0; transform: scale(0.5); }
        }

        /* Efek pop saat diklik */
        .moving-emoji.clicked {
            transform: scale(0); 
            opacity: 0;
            transition: transform 0s, opacity 0s; /* Hapus transisi untuk instant effect */
        }

        /* --- Overlay Hasil Game --- */
        #gameOverOverlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100vw;
            height: 100vh;
            background-color: rgba(0, 0, 0, 0.7);
            color: white;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            font-size: 2em;
            z-index: 100;
            opacity: 0;
            visibility: hidden;
            transition: opacity 0.5s ease;
        }

        #gameOverOverlay.show {
            opacity: 1;
            visibility: visible;
        }

        #final-score {
            font-size: 2.5em;
            color: #ffd700;
            margin-top: 20px;
        }
        
        #final-personal-high-score-message {
            font-size: 1.5em;
            color: #fff;
            margin-top: 10px;
            margin-bottom: 20px;
            text-align: center; /* Pastikan teks di tengah */
            padding: 0 10px; /* Tambahkan padding agar tidak terlalu mepet */
        }

        /* NEW: Gaya untuk pesan motivasi */
        #motivation-message {
            font-size: 1.3em;
            color: #ffec8b; /* Kuning cerah */
            margin-top: 10px;
            margin-bottom: 20px;
            font-weight: bold;
            text-shadow: 1px 1px 3px rgba(0,0,0,0.3);
        }


        .leaderboard-title {
            font-size: 1.8em;
            color: #ffd700;
            margin-top: 30px;
            margin-bottom: 15px;
            text-shadow: 1px 1px 3px rgba(0,0,0,0.2);
        }

        #leaderboard {
            list-style: none;
            padding: 0;
            margin: 0;
            font-size: 1.2em;
            max-height: 250px; /* Batasi tinggi untuk scroll jika banyak item */
            overflow-y: auto; /* Aktifkan scrollbar jika item melebihi tinggi */
            width: 80%;
            max-width: 400px;
            background-color: rgba(255, 255, 255, 0.2);
            border-radius: 10px;
            padding: 10px;
        }

        #leaderboard li {
            background-color: rgba(0, 0, 0, 0.3);
            margin-bottom: 5px;
            padding: 10px 15px;
            border-radius: 8px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            color: #eee;
            font-weight: normal;
        }

        #leaderboard li:nth-child(odd) {
            background-color: rgba(0, 0, 0, 0.25);
        }

        #leaderboard li strong {
            color: #ffd700;
            font-weight: bold;
        }

        #restartButton, #homeButton { /* Gaya untuk kedua tombol di game over */
            margin-top: 20px;
            background-color: #28a745;
            color: white;
            padding: 15px 30px;
            border-radius: 30px;
            font-size: 1.5em;
            cursor: pointer;
            margin-right: 10px; /* Jarak antar tombol */
            margin-left: 10px;
        }
        #restartButton:hover, #homeButton:hover {
            background-color: #218838;
            transform: translateY(-3px);
        }
        /* Style khusus untuk tombol home jika perlu warna berbeda */
        #homeButton {
            background-color: #007bff; /* Warna biru untuk tombol home */
        }
        #homeButton:hover {
            background-color: #0056b3;
        }

        .button-group {
            display: flex;
            justify-content: center;
            margin-top: 30px;
        }

        .rules-list {
            list-style: none;
            padding: 0;
            margin-bottom: 20px;
            font-size: 1.1em;
            color: #555;
        }

        .rules-list li {
            margin-bottom: 8px;
            text-align: left;
            padding-left: 20px;
            position: relative;
        }

        .rules-list li::before {
            content: '👉'; /* Emoji pointer sebagai bullet point */
            position: absolute;
            left: 0;
            top: 0;
        }

        /* Gaya untuk daftar syarat level */
        .level-up-requirements {
            list-style: none;
            padding: 0;
            margin-top: 15px;
            margin-bottom: 20px;
            font-size: 1em;
            color: #555;
            background-color: rgba(240, 240, 240, 0.7);
            border-radius: 10px;
            padding: 15px;
            box-shadow: inset 0 0 5px rgba(0,0,0,0.1);
        }

        .level-up-requirements li {
            margin-bottom: 5px;
            text-align: center;
            font-weight: bold;
            color: #007bff; /* Warna biru untuk teks level */
        }
        .level-up-requirements li span {
            color: #ff69b4; /* Warna pink untuk skor */
        }


        /* --- Countdown Overlay Styles --- */
        #countdown-overlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100vw;
            height: 100vh;
            background-color: rgba(0, 0, 0, 0.6);
            color: white;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 8em;
            font-weight: bold;
            z-index: 999; /* Pastikan di atas segalanya */
            opacity: 0;
            visibility: hidden;
            transition: opacity 0.3s ease;
            text-shadow: 0 0 20px rgba(255, 255, 255, 0.7);
        }

        #countdown-overlay.show {
            opacity: 1;
            visibility: visible;
        }
        
        /* Gaya untuk footer pembuat */
        .creator-info {
            font-size: 0.9em;
            color: #555;
            margin-top: 25px; /* Jarak dari tombol start */
            font-weight: bold;
            text-shadow: 1px 1px 2px rgba(255,255,255,0.2);
        }
    </style>
</head>
<body>

    <div id="game-info-overlay">
        <div class="display-in-game">Skor: <span id="score-in-game">0</span></div>
        <div class="display-in-game">Level: <span id="level-in-game">1</span></div>
        <div class="display-in-game">Nyawa: <span id="lives-in-game"></span></div>
    </div>

    <div class="game-container">
        <h1>😂 Kejar Emoji Lucu! 😂</h1>
        <p>Ayo, seberapa cepat kamu bisa menangkap emoji-emoji ini!</p>
        <p>Hati-hati, jangan klik bomnya! 💣</p>

        <div class="rules-section">
            <p style="font-weight: bold; margin-bottom: 10px;">Aturan Bermain:</p>
            <ul class="rules-list">
                <li>Tekan semua emoji yang lucu untuk mendapatkan poin! ✨</li>
                <li>Hindari emoji bom! 💥 Jika kamu menekannya, game akan berakhir.</li>
                <li>Jika emoji lucu menghilang tanpa diklik, kamu kehilangan 1 nyawa! ❤️</li>
                <li>Semakin tinggi skormu, semakin cepat dan banyak emoji yang muncul! 🚀</li>
            </ul>
        </div>

        <div class="level-requirements-section">
            <p style="font-weight: bold; margin-bottom: 10px; color: #ff69b4;">Syarat Naik Level:</p>
            <ul id="levelUpRequirementsList" class="level-up-requirements">
                </ul>
        </div>
        <div class="input-group">
            <label for="playerName">Masukkan Namamu:</label>
            <input type="text" id="playerName" placeholder="Anonim" maxlength="15">
        </div>

        <div class="score-board">Skor: <span id="score-main-screen">0</span></div>
        <div class="high-score">Skor Tertinggimu: <span id="highScore">0</span></div>

        <button id="startButton">Mulai Game!</button>

        <p class="creator-info">Dibuat oleh: AshFencer</p>
    </div>

    <div id="gameOverOverlay">
        <h2>Game Berakhir!</h2>
        <p>Skor Akhirmu:</p>
        <div id="final-score">0</div>
        <div id="final-personal-high-score-message"></div>
        <p id="motivation-message">Jangan sedih, ayooo coba lagiii! 💪😊</p>
        <h3 class="leaderboard-title">Papan Peringkat</h3>
        <ul id="leaderboard">
            </ul>

        <div class="button-group">
            <button id="restartButton">Main Lagi!</button>
            <button id="homeButton">Kembali ke Beranda</button>
        </div>
    </div>

    <div id="countdown-overlay">
        <span id="countdown-number">3</span>
    </div>

    <script>
        // --- JavaScript untuk Logika Game ---
        // Elemen yang ada di main screen (sebelum/setelah game)
        const gameContainer = document.querySelector('.game-container');
        const scoreMainScreenDisplay = document.getElementById('score-main-screen');
        const startButton = document.getElementById('startButton');
        const highScoreDisplay = document.getElementById('highScore');
        const playerNameInput = document.getElementById('playerName');
        const levelUpRequirementsList = document.getElementById('levelUpRequirementsList'); 

        // Elemen untuk overlay game over
        const gameOverOverlay = document.getElementById('gameOverOverlay');
        const finalScoreDisplay = document.getElementById('final-score');
        const finalPersonalHighScoreMessage = document.getElementById('final-personal-high-score-message');
        const motivationMessage = document.getElementById('motivation-message'); 
        const leaderboardList = document.getElementById('leaderboard');
        const restartButton = document.getElementById('restartButton');
        const homeButton = document.getElementById('homeButton');

        // Elemen untuk tampilan saat game berlangsung
        const gameInfoOverlay = document.getElementById('game-info-overlay');
        const scoreInGameDisplay = document.getElementById('score-in-game');
        const levelInGameDisplay = document.getElementById('level-in-game');
        const livesInGameDisplay = document.getElementById('lives-in-game');

        // Elemen untuk countdown
        const countdownOverlay = document.getElementById('countdown-overlay');
        const countdownNumber = document.getElementById('countdown-number');

        let score = 0;
        let gameLevel = 1; 
        let lives = 3;     
        let emojiSpawnIntervalId;
        let activeEmojis = [];
        let personalHighScore = { name: 'Anonim', score: 0 };
        let globalLeaderboard = [];
        let gameActive = false; 

        const normalEmojis = ['😊', '😂', '😜', '😍', '🤩', '🥳', '😎', '😇', '😋', '🤓', '🚀', '🌈', '🌟', '🍕', '🍦'];
        const bombEmoji = '💣';
        const MAX_LEADERBOARD_ENTRIES = 10;

        // --- Konfigurasi Level Kesulitan ---
        // Aturan untuk setiap level
        const levelSettings = {
            1: {
                emojiSpeed: 2.0,       
                spawnDelay: 1000,      
                bombChance: 0.05,      
                nextLevelScore: 20     
            },
            2: {
                emojiSpeed: 1.5,
                spawnDelay: 700,
                bombChance: 0.15,
                nextLevelScore: 50     
            },
            3: {
                emojiSpeed: 1.0,
                spawnDelay: 500,
                bombChance: 0.25,
                nextLevelScore: 100    
            },
            4: {
                emojiSpeed: 0.7,
                spawnDelay: 300,
                bombChance: 0.40,
                nextLevelScore: Infinity 
            }
        };

        // Fungsi untuk menampilkan syarat naik level di beranda
        function displayLevelRequirements() {
            levelUpRequirementsList.innerHTML = ''; 

            for (let i = 1; i <= Object.keys(levelSettings).length; i++) {
                const settings = levelSettings[i];
                if (settings.nextLevelScore !== Infinity) {
                    const listItem = document.createElement('li');
                    listItem.innerHTML = `Level ${i} ke ${i + 1}: Perlu Skor <span>${settings.nextLevelScore}</span>`;
                    levelUpRequirementsList.appendChild(listItem);
                } else {
                    const listItem = document.createElement('li');
                    listItem.innerHTML = `Level ${i}: Ini adalah Level Maksimum!`;
                    levelUpRequirementsList.appendChild(listItem);
                }
            }
        }

        // Fungsi untuk memuat data high score (pribadi dan leaderboard) dari localStorage
        function loadGameData() {
            const storedPersonalHighScore = localStorage.getItem('personalHighScore');
            if (storedPersonalHighScore) {
                personalHighScore = JSON.parse(storedPersonalHighScore);
            }

            const storedLeaderboard = localStorage.getItem('globalLeaderboard');
            if (storedLeaderboard) {
                globalLeaderboard = JSON.parse(storedLeaderboard);
            }
            updateDisplays();
            displayLevelRequirements(); 
        }

        // Fungsi untuk memperbarui semua tampilan terkait skor
        function updateDisplays() {
            highScoreDisplay.textContent = `${personalHighScore.name}: ${personalHighScore.score}`;
            scoreMainScreenDisplay.textContent = score;
            renderLeaderboard();
        }

        // Fungsi untuk merender papan peringkat
        function renderLeaderboard() {
            leaderboardList.innerHTML = ''; 

            // Urutkan leaderboard dari skor tertinggi ke terendah
            globalLeaderboard.sort((a, b) => b.score - a.score);

            globalLeaderboard.forEach((entry, index) => {
                const listItem = document.createElement('li');
                let medalEmoji = '';
                if (index === 0) {
                    medalEmoji = '🥇 '; 
                } else if (index === 1) {
                    medalEmoji = '🥈 '; 
                } else if (index === 2) {
                    medalEmoji = '🥉 '; 
                }
                listItem.innerHTML = `<span>${medalEmoji}${index + 1}. ${entry.name}</span> <strong>${entry.score}</strong>`;
                leaderboardList.appendChild(listItem);
            });

            if (globalLeaderboard.length === 0) {
                const noEntry = document.createElement('li');
                noEntry.textContent = "Belum ada skor di papan peringkat.";
                noEntry.style.justifyContent = 'center';
                leaderboardList.appendChild(noEntry);
            }
        }

        // Fungsi untuk memperbarui tampilan nyawa
        function updateLivesDisplay() {
            let hearts = '';
            for (let i = 0; i < lives; i++) {
                hearts += '❤️'; 
            }
            livesInGameDisplay.textContent = hearts;
        }

        // Fungsi untuk memperbarui pengaturan game berdasarkan level saat ini
        function applyLevelSettings() {
            clearInterval(emojiSpawnIntervalId); 
            const settings = levelSettings[gameLevel];
            if (settings) {
                emojiSpawnIntervalId = setInterval(() => {
                    if (gameActive) { 
                        createEmoji(settings);
                    }
                }, settings.spawnDelay);
                levelInGameDisplay.textContent = gameLevel; 
                console.log(`Level: ${gameLevel}, Speed: ${settings.emojiSpeed}s, Spawn: ${settings.spawnDelay}ms, Bomb Chance: ${settings.bombChance*100}%`);
            } else {
                console.warn("Pengaturan level tidak ditemukan untuk level:", gameLevel);
                const lastKnownLevel = Object.keys(levelSettings).length;
                const lastSettings = levelSettings[lastKnownLevel];
                if (lastSettings) {
                    emojiSpawnIntervalId = setInterval(() => {
                        if (gameActive) {
                            createEmoji(lastSettings);
                        }
                    }, lastSettings.spawnDelay);
                    levelInGameDisplay.textContent = lastKnownLevel;
                }
            }
        }

        // Fungsi yang akan dipanggil setelah countdown selesai
        function actualStartGame() {
            gameActive = true; 
            applyLevelSettings(); 
            countdownOverlay.classList.remove('show'); 
            gameInfoOverlay.classList.add('show');
        }

        // Fungsi untuk memulai countdown
        function startCountdown() {
            score = 0;
            gameLevel = 1;
            lives = 3;
            scoreInGameDisplay.textContent = score;
            levelInGameDisplay.textContent = gameLevel;
            updateLivesDisplay();

            gameContainer.style.display = 'none';
            gameOverOverlay.classList.remove('show');
            gameInfoOverlay.classList.add('show'); 

            activeEmojis.forEach(emoji => emoji.remove());
            activeEmojis = [];

            // Tampilkan countdown overlay
            countdownOverlay.classList.add('show');
            let count = 3;
            countdownNumber.textContent = count;

            const countdownInterval = setInterval(() => {
                count--;
                if (count > 0) {
                    countdownNumber.textContent = count;
                } else if (count === 0) {
                    countdownNumber.textContent = 'GO!';
                } else {
                    clearInterval(countdownInterval);
                    actualStartGame(); 
                }
            }, 1000);
        }

        // Fungsi untuk mengakhiri game
        function endGame() {
            gameActive = false; 
            clearInterval(emojiSpawnIntervalId);

            activeEmojis.forEach(emoji => emoji.remove());
            activeEmojis = [];

            gameInfoOverlay.classList.remove('show');
            countdownOverlay.classList.remove('show'); 

            let currentName = playerNameInput.value || 'Anonim';

            if (score > personalHighScore.score) {
                personalHighScore.score = score;
                personalHighScore.name = currentName;
                localStorage.setItem('personalHighScore', JSON.stringify(personalHighScore));
                finalPersonalHighScoreMessage.textContent = `Selamat! Kamu membuat skor tertinggi barumu: ${currentName}: ${score}`;
            } else {
                finalPersonalHighScoreMessage.textContent = '';
            }

            globalLeaderboard.push({ name: currentName, score: score });
            globalLeaderboard.sort((a, b) => b.score - a.score);
            if (globalLeaderboard.length > MAX_LEADERBOARD_ENTRIES) {
                globalLeaderboard = globalLeaderboard.slice(0, MAX_LEADERBOARD_ENTRIES);
            }
            localStorage.setItem('globalLeaderboard', JSON.stringify(globalLeaderboard));

            updateDisplays();

            finalScoreDisplay.textContent = score;
            gameOverOverlay.classList.add('show');

            motivationMessage.style.display = 'block';

            playerNameInput.style.display = 'block';
            document.querySelector('.input-group label').style.display = 'block';
        }

        // Fungsi untuk mengatur ulang UI kembali ke layar utama
        function resetGameUI() {
            gameOverOverlay.classList.remove('show');
            gameContainer.style.display = 'block';
            scoreMainScreenDisplay.textContent = 0; 
            loadGameData(); 

            motivationMessage.style.display = 'none';
        }

        // Fungsi untuk membuat emoji baru
        function createEmoji(settings) {
            if (!gameActive) return;

            const emoji = document.createElement('div');
            
            const isBomb = Math.random() < settings.bombChance;
            
            emoji.textContent = isBomb ? bombEmoji : normalEmojis[Math.floor(Math.random() * normalEmojis.length)];

            emoji.classList.add('moving-emoji');
            if (isBomb) {
                emoji.classList.add('bomb');
            }

            emoji.style.animation = `fadeInOut ${settings.emojiSpeed}s forwards`;

            const bodyRect = document.body.getBoundingClientRect();
            const containerRect = document.querySelector('.game-container').getBoundingClientRect();

            let randomX, randomY;
            let maxAttempts = 100;

            do {
                randomX = Math.random() * (bodyRect.width - 120) + 60;
                randomY = Math.random() * (bodyRect.height - 120) + 60;
                maxAttempts--;
                if (maxAttempts <= 0) break;
            } while (
                randomX < (containerRect.right + 20) &&
                randomX + emoji.offsetWidth > (containerRect.left - 20) &&
                randomY < (containerRect.bottom + 20) &&
                randomY + emoji.offsetHeight > (containerRect.top - 20)
            );

            emoji.style.left = `${randomX}px`;
            emoji.style.top = `${randomY}px`;

            emoji.addEventListener('click', function() {
                if (gameActive && !this.classList.contains('clicked')) {
                    if (this.classList.contains('bomb')) {
                        this.classList.add('clicked');
                        this.remove();
                        activeEmojis = activeEmojis.filter(e => e !== this);
                        endGame(); 
                    } else {
                        score++;
                        scoreInGameDisplay.textContent = score;

                        if (levelSettings[gameLevel] && score >= levelSettings[gameLevel].nextLevelScore) {
                            gameLevel++;
                            if (levelSettings[gameLevel]) {
                                applyLevelSettings(); 
                            } else {
                                console.log("Mencapai level maksimum!");
                                levelInGameDisplay.textContent = Object.keys(levelSettings).length; 
                            }
                        }
                    }

                    this.classList.add('clicked');
                    this.remove();
                    activeEmojis = activeEmojis.filter(e => e !== this);
                }
            });

            emoji.addEventListener('animationend', () => {
                if (gameActive && !emoji.classList.contains('clicked') && !emoji.classList.contains('bomb') && document.body.contains(emoji)) {
                    lives--; 
                    updateLivesDisplay(); 
                    emoji.remove();
                    activeEmojis = activeEmojis.filter(e => e !== emoji);

                    if (lives <= 0) {
                        endGame(); 
                    }
                } else if (document.body.contains(emoji)) {
                    emoji.remove();
                    activeEmojis = activeEmojis.filter(e => e !== emoji);
                }
            });

            document.body.appendChild(emoji);
            activeEmojis.push(emoji);
        }

        // Event listener untuk tombol mulai dan main lagi
        startButton.addEventListener('click', startCountdown); 
        restartButton.addEventListener('click', startCountdown); 
        homeButton.addEventListener('click', resetGameUI);

        // Inisialisasi tampilan awal
        loadGameData();
    </script>
</body>
</html>
