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
            <div id="online-lobby-panel" style="display: none;" class="hidden w-full max-w-lg bg-slate-950/90 border border-amber-500/30 rounded-2xl p-4 flex flex-col items-center gap-3">
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

                <div id="online-status-box" style="display: none;" class="hidden w-full bg-slate-900 border border-slate-800 rounded-xl p-3 text-center">
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
                        <p><span class="text-amber-300 font-bold">HP:</span> 110 (Party-König)</p>
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
                    <button onclick="selectArena('jungle')" id="btn-arena-jungle" class="px-3 py-1.5 rounded-xl border border-amber-500/50 bg
