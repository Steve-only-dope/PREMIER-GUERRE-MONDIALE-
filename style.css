// Récupération des éléments
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const scoreSpan = document.getElementById('score');
const livesSpan = document.getElementById('lives');
const highscoreSpan = document.getElementById('highscore');
const restartBtn = document.getElementById('restart');
const homeBtn = document.getElementById('home');
const startBtn = document.getElementById('start-game');
const homeScreen = document.getElementById('home-screen');
const gameScreen = document.getElementById('game-screen');

// Dimensions dynamiques
let canvasWidth, canvasHeight;

function resizeCanvas() {
    const rect = canvas.getBoundingClientRect();
    canvasWidth = rect.width;
    canvasHeight = rect.height;
    canvas.width = canvasWidth;
    canvas.height = canvasHeight;
    if (jeuActif) {
        joueurX = Math.min(Math.max(joueurX, 0), canvasWidth - TAILLE_AVION);
        joueurY = canvasHeight - TAILLE_AVION - 20;
    }
}

window.addEventListener('resize', resizeCanvas);

// Constantes
const AVION_JOUEUR = '✈️';
const AVIONS_ENNEMIS = ['🛩️', '✈️', '🚁', '💥'];
const BONUS = ['⭐', '💰', '💎', '🔵'];
const MISSILE = '💥';

const TAILLE_AVION_BASE = 0.12; // 12% de la largeur (un peu plus grand en vertical)
const TAILLE_MISSILE = 12;
let TAILLE_AVION = 50;

const VITESSE_INITIALE = 1.5; // Un peu plus lent pour le format vertical
const FREQ_INITIALE = 40;
const MAX_VITESSE = 7;
const MIN_FREQ = 18;
const VIES_INITIALES = 3;
const VITESSE_MISSILE = 7;
const DELAI_TIR = 12;

// Variables d'état
let joueurX = 0, joueurY = 0;
let objets = [];
let missiles = [];
let particules = [];
let score = 0;
let vies = VIES_INITIALES;
let highscore = localStorage.getItem('highscore') || 0;
highscoreSpan.textContent = highscore;
let jeuActif = false;
let frameCompteur = 0;
let frameTir = 0;
let vitesse = VITESSE_INITIALE;
let frequence = FREQ_INITIALE;

// Audio
let audioCtx = null;
function initAudio() {
    try {
        if (audioCtx) return;
        audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    } catch (e) {}
}
function playSound(type) {
    if (!audioCtx) initAudio();
    if (!audioCtx || audioCtx.state === 'closed') return;
    if (audioCtx.state === 'suspended') audioCtx.resume();
    try {
        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();
        osc.connect(gain);
        gain.connect(audioCtx.destination);
        if (type === 'tir') {
            osc.frequency.value = 600;
            gain.gain.value = 0.1;
            osc.start();
            osc.stop(audioCtx.currentTime + 0.05);
        } else if (type === 'explosion') {
            osc.frequency.value = 150;
            gain.gain.value = 0.3;
            osc.start();
            osc.stop(audioCtx.currentTime + 0.15);
        } else if (type === 'bonus') {
            osc.frequency.value = 1200;
            gain.gain.value = 0.1;
            osc.start();
            osc.stop(audioCtx.currentTime + 0.1);
        } else if (type === 'gameover') {
            osc.frequency.value = 80;
            gain.gain.value = 0.4;
            osc.start();
            osc.stop(audioCtx.currentTime + 0.5);
        }
    } catch (e) {}
}

// Affichage
function mettreAJourAffichage() {
    scoreSpan.textContent = score;
    livesSpan.textContent = vies;
    if (score > highscore) {
        highscore = score;
        localStorage.setItem('highscore', highscore);
        highscoreSpan.textContent = highscore;
    }
}

// Création objet
function creerObjet() {
    const x = Math.random() * (canvasWidth - TAILLE_AVION);
    const estBonus = Math.random() < 0.35; // 35% de bonus
    let type, emoji;
    
    if (estBonus) {
        type = 'bonus';
        emoji = BONUS[Math.floor(Math.random() * BONUS.length)];
    } else {
        type = 'ennemi';
        emoji = AVIONS_ENNEMIS[Math.floor(Math.random() * AVIONS_ENNEMIS.length)];
    }
    
    objets.push({
        x: x,
        y: -TAILLE_AVION,
        type: type,
        emoji: emoji
    });
}

// Tirer un missile
function tirerMissile() {
    if (frameTir <= 0) {
        missiles.push({
            x: joueurX + TAILLE_AVION/2 - TAILLE_MISSILE/2,
            y: joueurY - TAILLE_MISSILE * 2,
            largeur: TAILLE_MISSILE,
            hauteur: TAILLE_MISSILE * 2
        });
        playSound('tir');
        frameTir = DELAI_TIR;
    }
}

// Particules (explosions)
function creerExplosion(x, y, couleur) {
    for (let i = 0; i < 12; i++) {
        particules.push({
            x: x,
            y: y,
            vx: (Math.random() - 0.5) * 10,
            vy: (Math.random() - 0.5) * 8 - 2,
            life: 1.0,
            couleur: couleur
        });
    }
}

// Contrôles tactiles
canvas.addEventListener('touchstart', (e) => {
    e.preventDefault();
    if (!jeuActif) return;
    
    tirerMissile();
    
    const touch = e.touches[0];
    const rect = canvas.getBoundingClientRect();
    const scaleX = canvas.width / rect.width;
    const touchX = (touch.clientX - rect.left) * scaleX;
    deplacerJoueur(touchX);
});

canvas.addEventListener('touchmove', (e) => {
    e.preventDefault();
    if (!jeuActif) return;
    const touch = e.touches[0];
    const rect = canvas.getBoundingClientRect();
    const scaleX = canvas.width / rect.width;
    const touchX = (touch.clientX - rect.left) * scaleX;
    deplacerJoueur(touchX);
});

canvas.addEventListener('touchend', (e) => e.preventDefault());

// Souris (debug)
canvas.addEventListener('mousedown', (e) => {
    if (!jeuActif) return;
    tirerMissile();
});

canvas.addEventListener('mousemove', (e) => {
    if (!jeuActif) return;
    if (e.buttons !== 1) return;
    const rect = canvas.getBoundingClientRect();
    const scaleX = canvas.width / rect.width;
    const mouseX = (e.clientX - rect.left) * scaleX;
    deplacerJoueur(mouseX);
});

function deplacerJoueur(x) {
    joueurX = Math.min(Math.max(x - TAILLE_AVION / 2, 0), canvasWidth - TAILLE_AVION);
}

// Mise à jour
function update() {
    if (!jeuActif) return;

    if (frameTir > 0) frameTir--;

    frameCompteur++;
    if (frameCompteur >= frequence) {
        creerObjet();
        frameCompteur = 0;
    }

    // Particules
    for (let i = particules.length - 1; i >= 0; i--) {
        const p = particules[i];
        p.x += p.vx;
        p.y += p.vy;
        p.vy += 0.2;
        p.life -= 0.02;
        if (p.life <= 0) particules.splice(i, 1);
    }

    // Missiles
    for (let i = missiles.length - 1; i >= 0; i--) {
        const m = missiles[i];
        m.y -= VITESSE_MISSILE;
        
        if (m.y + m.hauteur < 0) {
            missiles.splice(i, 1);
            continue;
        }
        
        for (let j = objets.length - 1; j >= 0; j--) {
            const obj = objets[j];
            if (obj.type === 'ennemi' &&
                m.x < obj.x + TAILLE_AVION &&
                m.x + m.largeur > obj.x &&
                m.y < obj.y + TAILLE_AVION &&
                m.y + m.hauteur > obj.y) {
                
                score += 20;
                playSound('explosion');
                creerExplosion(obj.x + TAILLE_AVION/2, obj.y + TAILLE_AVION/2, 'orange');
                objets.splice(j, 1);
                missiles.splice(i, 1);
                mettreAJourAffichage();
                break;
            }
        }
    }

    // Objets
    for (let i = objets.length - 1; i >= 0; i--) {
        const obj = objets[i];
        obj.y += vitesse;

        if (obj.y + TAILLE_AVION > joueurY && obj.y < joueurY + TAILLE_AVION &&
            obj.x + TAILLE_AVION > joueurX && obj.x < joueurX + TAILLE_AVION) {

            if (obj.type === 'bonus') {
                score += 10;
                playSound('bonus');
                creerExplosion(obj.x + TAILLE_AVION/2, obj.y + TAILLE_AVION/2, 'gold');
                if (vitesse < MAX_VITESSE) vitesse += 0.12;
                if (frequence > MIN_FREQ && score % 50 === 0) frequence -= 2;
            } else {
                vies--;
                playSound('explosion');
                if (navigator.vibrate) navigator.vibrate(100);
                creerExplosion(obj.x + TAILLE_AVION/2, obj.y + TAILLE_AVION/2, 'red');
                if (vies <= 0) {
                    jeuActif = false;
                    playSound('gameover');
                }
            }
            objets.splice(i, 1);
            mettreAJourAffichage();
            continue;
        }

        if (obj.y > canvasHeight) {
            objets.splice(i, 1);
        }
    }
}

// Dessin
function draw() {
    TAILLE_AVION = canvasWidth * TAILLE_AVION_BASE;
    joueurY = canvasHeight - TAILLE_AVION - 20;
    joueurX = Math.min(Math.max(joueurX, 0), canvasWidth - TAILLE_AVION);

    ctx.clearRect(0, 0, canvasWidth, canvasHeight);

    // Nuages décoratifs
    ctx.fillStyle = 'rgba(255,255,255,0.1)';
    for (let i = 0; i < 5; i++) {
        ctx.beginPath();
        ctx.arc(50 + i * 120, 100 + i * 80, 25, 0, Math.PI * 2);
        ctx.fill();
    }

    // Particules
    particules.forEach(p => {
        ctx.globalAlpha = p.life;
        ctx.fillStyle = p.couleur;
        ctx.beginPath();
        ctx.arc(p.x, p.y, 6, 0, Math.PI * 2);
        ctx.fill();
    });
    ctx.globalAlpha = 1.0;

    // Missiles
    missiles.forEach(m => {
        ctx.fillStyle = '#ffff00';
        ctx.shadowBlur = 10;
        ctx.shadowColor = '#ffaa00';
        ctx.fillRect(m.x, m.y, m.largeur, m.hauteur);
        ctx.fillStyle = '#ff9900';
        ctx.fillRect(m.x - 2, m.y + m.hauteur, m.largeur + 4, 4);
    });

    // Objets
    ctx.font = `${TAILLE_AVION}px "Segoe UI Emoji", "Apple Color Emoji", "Noto Color Emoji", sans-serif`;
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.shadowBlur = 5;
    objets.forEach(obj => {
        ctx.shadowColor = 'black';
        if (obj.type === 'bonus') {
            ctx.fillStyle = '#ffd700';
        } else {
            ctx.fillStyle = '#ff4444';
        }
        ctx.fillText(obj.emoji, obj.x + TAILLE_AVION/2, obj.y + TAILLE_AVION/2);
        ctx.strokeStyle = 'white';
        ctx.lineWidth = 2;
        ctx.strokeText(obj.emoji, obj.x + TAILLE_AVION/2, obj.y + TAILLE_AVION/2);
    });

    // Joueur
    ctx.shadowBlur = 15;
    ctx.shadowColor = '#00ffff';
    ctx.fillStyle = '#00aaff';
    ctx.fillText(AVION_JOUEUR, joueurX + TAILLE_AVION/2, joueurY + TAILLE_AVION/2);
    ctx.shadowBlur = 0;
    ctx.strokeStyle = 'white';
    ctx.lineWidth = 3;
    ctx.strokeText(AVION_JOUEUR, joueurX + TAILLE_AVION/2, joueurY + TAILLE_AVION/2);

    // Game Over
    if (!jeuActif && vies <= 0) {
        ctx.font = `bold ${canvasWidth * 0.08}px Arial`;
        ctx.fillStyle = 'rgba(0,0,0,0.8)';
        ctx.fillText('GAME OVER', canvasWidth/2, canvasHeight/2 - 20);
        ctx.font = `${canvasWidth * 0.05}px Arial`;
        ctx.fillText('Score: ' + score, canvasWidth/2, canvasHeight/2 + 20);
    }
}

// Boucle
function gameLoop() {
    update();
    draw();
    requestAnimationFrame(gameLoop);
}

// Navigation
function startGame() {
    jeuActif = true;
    objets = [];
    missiles = [];
    particules = [];
    score = 0;
    vies = VIES_INITIALES;
    vitesse = VITESSE_INITIALE;
    frequence = FREQ_INITIALE;
    frameCompteur = 0;
    frameTir = 0;
    resizeCanvas();
    joueurX = canvasWidth/2 - TAILLE_AVION/2;
    joueurY = canvasHeight - TAILLE_AVION - 20;
    mettreAJourAffichage();
    homeScreen.classList.add('hidden');
    gameScreen.classList.remove('hidden');
    initAudio();
}

function goHome() {
    jeuActif = false;
    homeScreen.classList.remove('hidden');
    gameScreen.classList.add('hidden');
}

startBtn.addEventListener('click', startGame);
restartBtn.addEventListener('click', startGame);
homeBtn.addEventListener('click', goHome);

resizeCanvas();
gameLoop();
