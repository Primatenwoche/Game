<!DOCTYPE html>
<html lang="de">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Primaten Combat: Jungle & Zoo Edition</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- QRCode CDN -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
    <!-- PeerJS WebRTC für kostenlosen Online-Multiplayer -->
    <script src="https://unpkg.com/peerjs@1.5.4/dist/peerjs.min.js"></script>
    <style>
        body {
            font-family: 'Inter', system-ui, -apple-system, sans-serif;
            touch-action: manipulation;
            user-select: none;
        }
        canvas {
            display: block;
            box-shadow: 0 10px 30px rgba(0,0,0,0.5);
            border-radius: 1rem;
        }
        .select-card {
            transition: all 0.2s ease;
        }
        .select-card:hover {
            transform: translateY(-4px);
        }
        .select-card.selected-p1 {
            border-color: #3b82f6 !important;
            box-shadow: 0 0 15px rgba(59, 130, 246, 0.6);
        }
        .select-card.selected-p2 {
            border-color: #ef4444 !important;
            box-shadow: 0 0 15px rgba(239, 68, 68, 0.6);
        }
    </style>
</head>
<body class="bg-slate-950 text-slate-100 min-h-screen flex flex-col items-center justify-center p-2 sm:p-4">

    <!-- Header UI -->
    <header class="mb-2 text-center">
        <h1 class="text-2xl sm:text-4xl font-extrabold tracking-wider bg-gradient-to-r from-amber-400 via-orange-500 to-red-500 bg-clip-text text-transparent filter drop-shadow">
            🦍 PRIMATEN COMBAT 🐒
        </h1>
        <p class="text-xs sm:text-sm text-slate-400 font-medium">Jungle & Zoo Deluxe Edition</p>
    </header>

    <!-- Main Container -->
    <main class="w-full max-w-4xl flex flex-col items-center">

        <!-- Character & Arena Selection Menu -->
        <div id="selection-screen" class="w-full bg-slate-900/90 border border-slate-800 rounded-3xl p-4 sm:p-6 flex flex-col items-center gap-4 shadow-2xl backdrop-blur-md">
            
            <!-- Game Mode Switcher: Local vs Online -->
            <div class="flex gap-2 bg-slate-950/80 p-1.5 rounded-2xl border border-slate-800">
                <button id="mode-local-btn" onclick="switchGameMode('local')" class="px-4 py-2 rounded-xl text-xs sm:text-sm font-bold bg-amber-500 text-slate-950 transition-all shadow">
                    🏠 Lokales Spiel (1 Gerät)
                </button>
                <button id="mode-online-btn" onclick="switchGameMode('online')" class="px-4 py-2 rounded-xl text-xs sm:text-sm font-bold bg-slate-800 text-slate-400 hover:text-white transition-all">
                    🌐 Online-Raum (2 Geräte)
                </button>
            </div>

            <!-- Online Lobby Panel -->
            <div id="online-lobby-panel" class="hidden w-full max-w-lg bg-slate-950/90 border border-amber-500/30 rounded-2xl p-4 flex flex-col items-center gap-3">
                <div class="flex flex-wrap justify-center gap-3 w-full">
                    <button onclick="createOnlineRoom()" class="flex-1 min-w-[140px] px-4 py-2.5 bg-emerald-600 hover:bg-emerald-500 text-white font-bold text-xs rounded-xl shadow transition-all">
                        ➕ Raum Erstellen (P1)
                    </button>
                    <div class="flex gap-1 flex-1 min-w-[180px]">
                        <input id="room-code-input" type="text" placeholder="CODE" maxlength="4" class="w-full bg-slate-900 border border-slate-700 px-3 py-2 text-xs text-amber-300 font-mono text-center rounded-xl font-bold uppercase">
                        <button onclick="joinOnlineRoom()" class="px-4 py-2.5 bg-blue-600 hover:bg-blue-500 text-white font-bold text-xs rounded-xl shadow transition-all">
                            🔑 Beitreten (P2)
                        </button>
                    </div>
                </div>

                <div id="online-status-box" class="hidden w-full bg-slate-900 border border-slate-800 rounded-xl p-3 text-center">
                    <p id="online-status-text" class="text-xs text-amber-400 font-bold">Warte auf Mitspieler...</p>
                    <p id="online-room-code-display" class="text-sm font-mono text-white mt-1">RAUM-CODE: <span class="text-amber-400 font-black" id="current-room-code-val">----</span></p>
                </div>
            </div>

            <div class="text-center">
                <h2 id="select-title" class="text-lg sm:text-xl font-bold text-amber-400 flex items-center justify-center gap-2">
                    <span>🎮</span> Wähle deinen Primaten
                </h2>
                <p id="select-subtitle" class="text-xs text-slate-400 mt-1 font-medium">Wähle deine Primaten (P1 = Blau, P2 = Rot)</p>
            </div>

            <!-- Character Cards Grid -->
            <div class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-6 gap-2 sm:gap-3 w-full max-w-5xl">
                <!-- Gorilla -->
                <div id="card-gorilla" onclick="selectCharacter('gorilla')" class="select-card bg-slate-800/90 border-2 border-slate-700 rounded-2xl p-2.5 flex flex-col items-center cursor-pointer hover:border-amber-400">
                    <div class="text-3xl sm:text-4xl mb-1 filter drop-shadow">🦍</div>
                    <h3 class="font-bold text-xs sm:text-sm text-amber-400 font-mono">Gorilla</h3>
                    <div class="text-[9px] text-slate-300 text-center mt-1 space-y-0.5">
                        <p><span class="text-amber-300 font-bold">HP:</span> 180 (Tank)</p>
                        <p><span class="text-slate-400 font-bold">Passiv:</span> &le;40% HP 🛡️ Melee-Immun</p>
                    </div>
                </div>

                <!-- Orang-Utan -->
                <div id="card-orangutan" onclick="selectCharacter('orangutan')" class="select-card bg-slate-800/90 border-2 border-slate-700 rounded-2xl p-2.5 flex flex-col items-center cursor-pointer hover:border-amber-400">
                    <div class="text-3xl sm:text-4xl mb-1 filter drop-shadow">🦧</div>
                    <h3 class="font-bold text-xs sm:text-sm text-orange-400 font-mono">Orang-Utan</h3>
                    <div class="text-[9px] text-slate-300 text-center mt-1 space-y-0.5">
                        <p><span class="text-amber-300 font-bold">HP:</span> 150 (Balanced)</p>
                        <p><span class="text-orange-400 font-bold">Passiv:</span> &le;50% HP 💥 2x Melee</p>
                    </div>
                </div>

                <!-- Schimpanse -->
                <div id="card-schimpanse" onclick="selectCharacter('schimpanse')" class="select-card bg-slate-800/90 border-2 border-slate-700 rounded-2xl p-2.5 flex flex-col items-center cursor-pointer hover:border-amber-400">
                    <div class="text-3xl sm:text-4xl mb-1 filter drop-shadow">🐒</div>
                    <h3 class="font-bold text-xs sm:text-sm text-purple-400 font-mono">Schimpanse</h3>
                    <div class="text-[9px] text-slate-300 text-center mt-1 space-y-0.5">
                        <p><span class="text-amber-300 font-bold">HP:</span> 135 (Kämpfer)</p>
                        <p><span class="text-purple-300 font-bold">Passiv:</span> &le;50% HP 🔒 Sprung-Sperre</p>
                    </div>
                </div>

                <!-- Gibbon -->
                <div id="card-gibbon" onclick="selectCharacter('gibbon')" class="select-card bg-slate-800/90 border-2 border-slate-700 rounded-2xl p-2.5 flex flex-col items-center cursor-pointer hover:border-amber-400">
                    <div class="text-3xl sm:text-4xl mb-1 filter drop-shadow">🐒</div>
                    <h3 class="font-bold text-xs sm:text-sm text-emerald-400 font-mono">Gibbon</h3>
                    <div class="text-[9px] text-slate-300 text-center mt-1 space-y-0.5">
                        <p><span class="text-amber-300 font-bold">HP:</span> 120 (Agil)</p>
                        <p><span class="text-emerald-400 font-bold">Passiv:</span> &le;90% HP ⚡ Tempo</p>
                    </div>
                </div>

                <!-- Lemur -->
                <div id="card-lemur" onclick="selectCharacter('lemur')" class="select-card bg-slate-800/90 border-2 border-slate-700 rounded-2xl p-2.5 flex flex-col items-center cursor-pointer hover:border-amber-400">
                    <div class="text-3xl sm:text-4xl mb-1 filter drop-shadow">🦝</div>
                    <h3 class="font-bold text-xs sm:text-sm text-pink-400 font-mono">Lemur</h3>
                    <div class="text-[9px] text-slate-300 text-center mt-1 space-y-0.5">
                        <p><span class="text-amber-300 font-bold">HP:</span> 110 (Party-King)</p>
                        <p><span class="text-pink-400 font-bold">Spezial:</span> 🪩 Tanz-Bombe (5s)</p>
                        <p><span class="text-cyan-300 font-bold">Passiv:</span> &le;40% HP 🪞 Reflektion</p>
                    </div>
                </div>

                <!-- Mensch -->
                <div id="card-human" onclick="selectCharacter('human')" class="select-card bg-slate-800/90 border-2 border-slate-700 rounded-2xl p-2.5 flex flex-col items-center cursor-pointer hover:border-amber-400">
                    <div class="text-3xl sm:text-4xl mb-1 filter drop-shadow">🧍</div>
                    <h3 class="font-bold text-xs sm:text-sm text-blue-400 font-mono">Mensch</h3>
                    <div class="text-[9px] text-slate-300 text-center mt-1 space-y-0.5">
                        <p><span class="text-amber-300 font-bold">HP:</span> 100 (Taktiker)</p>
                        <p><span class="text-red-400 font-bold">Passiv:</span> &le;10% HP 🔥 AK47 (20 DMG)</p>
                    </div>
                </div>
            </div>

            <!-- Arena Selection -->
            <div class="w-full flex flex-col items-center mt-2">
                <h3 class="text-xs font-bold text-slate-400 uppercase tracking-widest mb-2">Wähle die Arena</h3>
                <div class="flex flex-wrap justify-center gap-2">
                    <button onclick="selectArena('jungle')" id="btn-arena-jungle" class="px-3 py-1.5 rounded-xl border border-amber-500/50 bg-amber-950/40 text-amber-300 text-xs font-semibold hover:bg-amber-900/50">🌴 Dschungel-Lichtung</button>
                    <button onclick="selectArena('zoo')" id="btn-arena-zoo" class="px-3 py-1.5 rounded-xl border border-slate-700 bg-slate-800 text-slate-400 text-xs font-semibold hover:bg-slate-700">🏛️ Zoo-Gehege</button>
                    <button onclick="selectArena('temple')" id="btn-arena-temple" class="px-3 py-1.5 rounded-xl border border-slate-700 bg-slate-800 text-slate-400 text-xs font-semibold hover:bg-slate-700">🗿 Tempel-Ruine</button>
                    <button onclick="selectArena('lab')" id="btn-arena-lab" class="px-3 py-1.5 rounded-xl border border-slate-700 bg-slate-800 text-slate-400 text-xs font-semibold hover:bg-slate-700">🔬 Bio-Labor</button>
                    <button onclick="selectArena('volcano')" id="btn-arena-volcano" class="px-3 py-1.5 rounded-xl border border-slate-700 bg-slate-800 text-slate-400 text-xs font-semibold hover:bg-slate-700">🌋 Vulkan-Höhle</button>
                </div>
            </div>

            <!-- Physics Toggle -->
            <div class="flex items-center gap-2 mt-2 bg-slate-800/80 px-4 py-2 rounded-xl border border-slate-700">
                <input type="checkbox" id="toggle-realistic-physics" class="w-4 h-4 text-amber-500 rounded cursor-pointer" onchange="toggleRealisticPhysics(this.checked)">
                <label for="toggle-realistic-physics" class="text-xs sm:text-sm font-semibold text-slate-300 cursor-pointer">🎯 Realistische Physik (Bullet-Drop & Schwerkraft)</label>
            </div>

            <!-- Start Game & Share Buttons -->
            <div class="flex flex-wrap justify-center gap-3 mt-2">
                <button onclick="startGame()" id="start-btn" class="px-8 py-3 bg-gradient-to-r from-amber-500 to-orange-600 text-slate-950 font-extrabold text-base rounded-2xl shadow-lg hover:from-amber-400 hover:to-orange-500 transform hover:scale-105 active:scale-95 transition-all">
                    KAMPF STARTEN ⚔️
                </button>
                <button onclick="openQRModal()" class="px-5 py-3 bg-slate-800 border border-slate-700 text-amber-400 font-bold text-sm rounded-2xl shadow-lg hover:bg-slate-700 transform hover:scale-105 active:scale-95 transition-all flex items-center gap-2">
                    📱 QR-Code / Spiel Teilen
                </button>
            </div>
        </div>

        <!-- QR Code Share Modal -->
        <div id="qr-modal" class="hidden fixed inset-0 z-50 bg-slate-950/80 backdrop-blur-md flex items-center justify-center p-4">
            <div class="bg-slate-900 border border-slate-800 rounded-3xl p-6 max-w-sm w-full flex flex-col items-center text-center shadow-2xl relative">
                <button onclick="closeQRModal()" class="absolute top-4 right-4 text-slate-400 hover:text-white font-bold text-lg">✕</button>
                <h3 class="text-lg font-bold text-amber-400 mb-2">📲 Spiel teilen via QR-Code</h3>
                <p class="text-xs text-slate-400 mb-4">Scanne diesen Code mit deinem Smartphone, um dem Online-Raum beizutreten!</p>
                
                <div id="qrcode" class="p-3 bg-white rounded-2xl shadow-inner mb-4"></div>

                <div class="w-full flex flex-col gap-2">
                    <input id="share-url-input" type="text" readonly class="w-full bg-slate-950 border border-slate-800 rounded-xl px-3 py-2 text-xs text-slate-300 font-mono text-center select-all">
                    <button onclick="copyShareUrl()" class="w-full py-2 bg-amber-500 text-slate-950 font-bold text-xs rounded-xl hover:bg-amber-400 transition-all">
                        Link in Zwischenablage kopieren 📋
                    </button>
                    <p id="copy-toast" class="text-[10px] text-emerald-400 font-semibold hidden">Link erfolgreich kopiert!</p>
                </div>
            </div>
        </div>

        <!-- In-Game View Container -->
        <div id="game-container" class="hidden relative w-full flex flex-col items-center">
            
            <!-- Compact HUD -->
            <div id="hud" class="w-full bg-slate-900/90 border border-slate-800 rounded-2xl p-2 sm:p-3 mb-2 flex items-center justify-between text-xs sm:text-sm font-bold shadow-lg backdrop-blur-md">
                
                <!-- Player 1 Status -->
                <div class="flex items-center gap-2 w-5/12">
                    <span id="p1-icon" class="text-2xl filter drop-shadow">🦍</span>
                    <div class="w-full flex flex-col">
                        <div class="flex justify-between items-center mb-0.5">
                            <span id="p1-name" class="text-blue-400 font-mono text-xs">P1 (Gorilla)</span>
                            <span id="p1-hp-text" class="text-[10px] text-slate-300 font-mono">180/180</span>
                        </div>
                        <div class="w-full bg-slate-950 rounded-full h-3 p-0.5 border border-slate-800">
                            <div id="p1-hp-bar" class="bg-gradient-to-r from-emerald-500 to-emerald-400 h-full rounded-full transition-all duration-200" style="width: 100%;"></div>
                        </div>
                        <div class="flex gap-2 text-[9px] text-slate-400 mt-1">
                            <span id="p1-shoot-cd">🍌 Schuss: Ready</span>
                            <span id="p1-special-cd">✨ Spezial: Ready</span>
                            <span id="p1-melee-cd">🥊 Schlag: Ready</span>
                        </div>
                    </div>
                </div>

                <!-- Arena Title & Score -->
                <div class="flex flex-col items-center">
                    <span id="hud-arena-name" class="text-[10px] text-amber-400 font-mono tracking-wide uppercase">🌴 Dschungel</span>
                    <span id="hud-score" class="text-base text-slate-200 font-black font-mono">0 - 0</span>
                </div>

                <!-- Player 2 Status -->
                <div class="flex items-center gap-2 w-5/12 flex-row-reverse">
                    <span id="p2-icon" class="text-2xl filter drop-shadow">🧍</span>
                    <div class="w-full flex flex-col text-right">
                        <div class="flex justify-between items-center mb-0.5 flex-row-reverse">
                            <span id="p2-name" class="text-red-400 font-mono text-xs">P2 (Mensch)</span>
                            <span id="p2-hp-text" class="text-[10px] text-slate-300 font-mono">100/100</span>
                        </div>
                        <div class="w-full bg-slate-950 rounded-full h-3 p-0.5 border border-slate-800">
                            <div id="p2-hp-bar" class="bg-gradient-to-r from-emerald-500 to-emerald-400 h-full rounded-full transition-all duration-200 ml-auto" style="width: 100%;"></div>
                        </div>
                        <div class="flex gap-2 text-[9px] text-slate-400 mt-1 justify-end">
                            <span id="p2-melee-cd">Schlag: Ready 🥊</span>
                            <span id="p2-special-cd">Spezial: Ready ✨</span>
                            <span id="p2-shoot-cd">Schuss: Ready 🍌</span>
                        </div>
                    </div>
                </div>

            </div>

            <!-- Game Canvas -->
            <div class="relative w-full aspect-[16/9] max-h-[500px]">
                <canvas id="gameCanvas" class="w-full h-full bg-slate-900 border border-slate-800"></canvas>
                
                <!-- Game Over Overlay -->
                <div id="game-over-overlay" class="hidden absolute inset-0 bg-slate-950/80 backdrop-blur-md rounded-2xl flex flex-col items-center justify-center gap-4">
                    <h2 id="winner-text" class="text-3xl sm:text-5xl font-black text-amber-400 filter drop-shadow">SPIELER 1 GEWINNT!</h2>
                    <div class="flex gap-4">
                        <button onclick="restartRound()" class="px-6 py-2.5 bg-amber-500 text-slate-950 font-bold rounded-xl hover:bg-amber-400 transition-all">NÄCHSTE RUNDE ⚔️</button>
                        <button onclick="backToMenu()" class="px-6 py-2.5 bg-slate-800 text-slate-200 font-bold rounded-xl hover:bg-slate-700 transition-all">MENÜ 🏠</button>
                    </div>
                </div>
            </div>

            <!-- Mobile Touch Controls -->
            <div id="mobile-touch-controls" class="w-full mt-2 grid grid-cols-2 gap-2 p-2 bg-slate-900/80 border border-slate-800 rounded-2xl">
                <!-- P1 Touch Controls -->
                <div class="flex flex-col gap-1 items-start">
                    <span id="touch-p1-title" class="text-[10px] text-blue-400 font-bold">Steuerung P1</span>
                    <div class="flex gap-1">
                        <button class="w-10 h-10 bg-slate-800 active:bg-blue-600 rounded-xl text-lg font-bold border border-slate-700 flex items-center justify-center select-none"
                            ontouchstart="keys['KeyA']=true;" ontouchend="keys['KeyA']=false;"
                            onmousedown="keys['KeyA']=true;" onmouseup="keys['KeyA']=false;">◄</button>
                        <button class="w-10 h-10 bg-slate-800 active:bg-blue-600 rounded-xl text-lg font-bold border border-slate-700 flex items-center justify-center select-none"
                            ontouchstart="keys['KeyD']=true;" ontouchend="keys['KeyD']=false;"
                            onmousedown="keys['KeyD']=true;" onmouseup="keys['KeyD']=false;">►</button>
                        <button class="w-10 h-10 bg-amber-500/20 active:bg-amber-500 text-amber-400 rounded-xl text-lg font-bold border border-amber-500/40 flex items-center justify-center select-none"
                            ontouchstart="keys['KeyW']=true;" ontouchend="keys['KeyW']=false;"
                            onmousedown="keys['KeyW']=true;" onmouseup="keys['KeyW']=false;">▲</button>
                    </div>
                    <div class="flex gap-1 mt-1">
                        <button class="px-2.5 h-9 bg-slate-800 active:bg-amber-500 text-xs font-bold rounded-lg border border-slate-700"
                            ontouchstart="keys['KeyE']=true;" ontouchend="keys['KeyE']=false;"
                            onmousedown="keys['KeyE']=true;" onmouseup="keys['KeyE']=false;">🍌 Schuss</button>
                        <button class="px-2.5 h-9 bg-slate-800 active:bg-purple-500 text-xs font-bold rounded-lg border border-slate-700"
                            ontouchstart="keys['KeyR']=true;" ontouchend="keys['KeyR']=false;"
                            onmousedown="keys['KeyR']=true;" onmouseup="keys['KeyR']=false;">✨ Spezial</button>
                        <button class="px-2.5 h-9 bg-slate-800 active:bg-red-500 text-xs font-bold rounded-lg border border-slate-700"
                            ontouchstart="keys['KeyF']=true;" ontouchend="keys['KeyF']=false;"
                            onmousedown="keys['KeyF']=true;" onmouseup
