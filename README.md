<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pengusir Malas - Ayo Fokus!</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Arial', sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            color: white;
            overflow: hidden;
        }

        .container {
            text-align: center;
            max-width: 500px;
            padding: 20px;
        }

        h1 {
            font-size: 2.5em;
            margin-bottom: 20px;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
            animation: pulse 2s infinite;
        }

        @keyframes pulse {
            0% { transform: scale(1); }
            50% { transform: scale(1.05); }
            100% { transform: scale(1); }
        }

        .book-container {
            position: relative;
            margin: 40px 0;
        }

        .book {
            width: 200px;
            height: 200px;
            background: linear-gradient(145deg, #ff6b6b, #feca57);
            border-radius: 20px;
            cursor: pointer;
            transition: all 0.3s ease;
            box-shadow: 0 20px 40px rgba(0,0,0,0.3);
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 4em;
            font-weight: bold;
            user-select: none;
            position: relative;
            overflow: hidden;
        }

        .book::before {
            content: '📚';
            font-size: 5em;
            z-index: 2;
        }

        .book::after {
            content: '';
            position: absolute;
            top: 0;
            left: -100%;
            width: 100%;
            height: 100%;
            background: linear-gradient(90deg, transparent, rgba(255,255,255,0.4), transparent);
            transition: left 0.5s;
        }

        .book.pressed {
            transform: scale(0.95) rotate(-5deg);
            background: linear-gradient(145deg, #ff5252, #ffb74d);
            box-shadow: 0 10px 20px rgba(0,0,0,0.4);
        }

        .book.holding {
            background: linear-gradient(145deg, #4caf50, #81c784);
            box-shadow: 0 15px 30px rgba(76, 175, 80, 0.4);
        }

        .timer {
            font-size: 3em;
            font-weight: bold;
            margin: 20px 0;
            min-height: 80px;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.5);
            transition: all 0.3s ease;
        }

        .timer.best {
            color: #ffd700;
            text-shadow: 0 0 20px #ffd700;
            animation: glow 1s infinite alternate;
        }

        @keyframes glow {
            from { text-shadow: 0 0 20px #ffd700; }
            to { text-shadow: 0 0 30px #ffd700, 0 0 40px #ffd700; }
        }

        .status {
            font-size: 1.5em;
            margin: 20px 0;
            opacity: 0;
            transform: translateY(20px);
            transition: all 0.5s ease;
        }

        .status.show {
            opacity: 1;
            transform: translateY(0);
        }

        .instructions {
            margin-top: 30px;
            font-size: 1.2em;
            line-height: 1.6;
            max-width: 400px;
        }

        .best-time {
            position: absolute;
            top: 20px;
            right: 20px;
            background: rgba(255,255,255,0.2);
            padding: 10px 20px;
            border-radius: 25px;
            backdrop-filter: blur(10px);
        }

        .particles {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            z-index: 1;
        }

        .particle {
            position: absolute;
            width: 8px;
            height: 8px;
            background: #ffd700;
            border-radius: 50%;
            pointer-events: none;
            animation: particle-float 2s ease-out forwards;
        }

        @keyframes particle-float {
            0% {
                opacity: 1;
                transform: translateY(0) scale(1);
            }
            100% {
                opacity: 0;
                transform: translateY(-100px) scale(0);
            }
        }
    </style>
</head>
<body>
    <div class="best-time" id="bestTime">🏆 Rekor: 0s</div>
    
    <div class="container">
        <h1>🛡️ Pengusir Malas</h1>
        
        <div class="book-container">
            <div class="book" id="book">
                <span id="timer" class="timer">Tahan!</span>
            </div>
        </div>
        
        <div class="status" id="status"></div>
        
        <div class="instructions">
            📖 <strong>Tekan & Tahan Buku</strong> selama mungkin!<br>
            Jangan Lepas atau MALAS akan menang! 💪
        </div>
    </div>

    <div class="particles" id="particles"></div>

    <script>
        class PengusirMalas {
            constructor() {
                this.book = document.getElementById('book');
                this.timerEl = document.getElementById('timer');
                this.statusEl = document.getElementById('status');
                this.particlesEl = document.getElementById('particles');
                this.bestTimeEl = document.getElementById('bestTime');
                
                this.isHolding = false;
                this.startTime = 0;
                this.currentTime = 0;
                this.bestTime = localStorage.getItem('bestTime') || 0;
                this.timerInterval = null;
                
                this.init();
            }
            
            init() {
                this.updateBestTime();
                this.bindEvents();
            }
            
            bindEvents() {
                this.book.addEventListener('mousedown', (e) => this.startHold(e));
                this.book.addEventListener('touchstart', (e) => this.startHold(e), { passive: false });
                
                document.addEventListener('mouseup', () => this.stopHold());
                document.addEventListener('touchend', () => this.stopHold());
                document.addEventListener('touchcancel', () => this.stopHold());
            }
            
            startHold(e) {
                e.preventDefault();
                if (this.isHolding) return;
                
                this.isHolding = true;
                this.startTime = Date.now();
                this.book.classList.add('holding');
                this.book.classList.add('pressed');
                
                this.timerInterval = setInterval(() => {
                    this.currentTime = Math.floor((Date.now() - this.startTime) / 1000);
                    this.timerEl.textContent = `${this.currentTime}s`;
                }, 100);
                
                this.statusEl.textContent = '💪 FOKUS! Jangan Lepas!';
                this.showStatus();
            }
            
            stopHold() {
                if (!this.isHolding) return;
                
                this.isHolding = false;
                clearInterval(this.timerInterval);
                
                this.book.classList.remove('holding', 'pressed');
                this.timerEl.textContent = `${this.currentTime}s`;
                this.timerEl.classList.add('best');
                
                // Cek rekor baru
                if (this.currentTime > this.bestTime) {
                    this.bestTime = this.currentTime;
                    localStorage.setItem('bestTime', this.bestTime);
                    this.updateBestTime();
                    this.createParticles();
                }
                
                // Suara "Ayo Fokus!" menggunakan Web Speech API
                this.speak('Ayo Fokus! Kamu bertahan ' + this.currentTime + ' detik! Kerja bagus!');
                
                this.statusEl.textContent = `⏱️ Bertahan ${this.currentTime}s! Coba lagi!`;
                this.showStatus();
                
                setTimeout(() => {
                    this.timerEl.classList.remove('best');
                    this.timerEl.textContent = 'Tahan!';
                    this.hideStatus();
                }, 3000);
            }
            
            speak(text) {
                if ('speechSynthesis' in window) {
                    const utterance = new SpeechSynthesisUtterance(text);
                    utterance.lang = 'id-ID';
                    utterance.rate = 0.9;
                    utterance.pitch = 1.2;
                    speechSynthesis.speak(utterance);
                }
            }
            
            showStatus() {
                this.statusEl.classList.add('show');
            }
            
            hideStatus() {
                this.statusEl.classList.remove('show');
            }
            
            updateBestTime() {
                this.bestTimeEl.textContent = `🏆 Rekor: ${this.bestTime}s`;
            }
            
            createParticles() {
                for (let i = 0; i < 20; i++) {
                    setTimeout(() => {
                        this.createParticle();
                    }, i * 50);
                }
            }
            
            createParticle() {
                const particle = document.createElement('div');
                particle.className = 'particle';
                particle.style.left = Math.random() * 100 + '%';
                particle.style.animationDuration = (Math.random() * 1 + 1) + 's';
                this.particlesEl.appendChild(particle);
                
                setTimeout(() => {
                    particle.remove();
                }, 2000);
            }
        }
        
        // Inisialisasi saat DOM loaded
        document.addEventListener('DOMContentLoaded', () => {
            new PengusirMalas();
        });
    </script>
</body>
</html>
