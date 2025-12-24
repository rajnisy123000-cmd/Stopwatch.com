<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <meta name="description" content="A high-precision professional stopwatch with AI analysis and dark mode interface.">
    <meta name="theme-color" content="#0f172a">
    
    <!-- Open Graph / Facebook -->
    <meta property="og:type" content="website">
    <meta property="og:title" content="Pro AI Stopwatch">
    <meta property="og:description" content="Precision timing with Gemini-powered performance analysis.">
    
    <!-- PWA / Mobile Capable -->
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <meta name="apple-mobile-web-app-title" content="AI Stopwatch">
    
    <title>Pro AI Stopwatch</title>
    
    <!-- Favicon (Inline SVG) -->
    <link rel="icon" href="data:image/svg+xml,<svg xmlns=%22http://www.w3.org/2000/svg%22 viewBox=%220 0 100 100%22><text y=%22.9em%22 font-size=%2290%22>⏱️</text></svg>">
    
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Google Fonts: Roboto Mono for monospace numbers -->
    <link href="https://fonts.googleapis.com/css2?family=Roboto+Mono:wght@400;600&family=Inter:wght@400;600&display=swap" rel="stylesheet">
    <!-- Font Awesome for Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Markdown Parser -->
    <script src="https://cdn.jsdelivr.net/npm/marked/marked.min.js"></script>
    
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #0f172a; /* slate-950 */
            overscroll-behavior-y: none;
        }
        .mono-font {
            font-family: 'Roboto Mono', monospace;
            font-variant-numeric: tabular-nums;
        }
        ::-webkit-scrollbar { width: 4px; }
        ::-webkit-scrollbar-track { background: transparent; }
        ::-webkit-scrollbar-thumb { background: #334155; border-radius: 4px; }
        
        .btn-press {
            transition: transform 0.1s ease-in-out, background-color 0.2s, box-shadow 0.2s;
            touch-action: manipulation;
        }
        .btn-press:active { transform: scale(0.96); }
        
        .glass {
            background: rgba(30, 41, 59, 0.7);
            backdrop-filter: blur(12px);
            -webkit-backdrop-filter: blur(12px);
            border: 1px solid rgba(255, 255, 255, 0.1);
        }

        /* Loading Spinner */
        .spinner {
            border: 3px solid rgba(255,255,255,0.1);
            border-left-color: #a855f7; /* purple-500 */
            border-radius: 50%;
            width: 24px;
            height: 24px;
            animation: spin 1s linear infinite;
        }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
        
        /* Markdown Styles for Modal */
        .prose p { margin-bottom: 0.5em; font-size: 0.95rem; line-height: 1.5; }
        .prose strong { color: #e2e8f0; font-weight: 600; }
        .prose ul { list-style-type: disc; padding-left: 1.2em; margin-bottom: 0.5em; }
    </style>
</head>
<body class="h-screen w-full flex flex-col items-center justify-center text-slate-200 overflow-hidden bg-slate-950">

    <!-- Main Container -->
    <div class="w-full max-w-md px-4 flex flex-col h-full max-h-[100dvh] pt-4 pb-6">
        
        <!-- Header -->
        <div class="text-center mb-6 mt-2 shrink-0">
            <h1 class="text-xs font-bold tracking-[0.2em] text-cyan-500 uppercase flex items-center justify-center gap-2">
                <i class="fa-solid fa-stopwatch"></i> Chrono<span class="text-white">Master</span> AI
            </h1>
        </div>

        <!-- Timer Display Card -->
        <div class="glass rounded-3xl p-8 mb-8 relative overflow-hidden group shrink-0 shadow-2xl">
            <!-- decorative glow -->
            <div class="absolute -top-10 -left-10 w-32 h-32 bg-cyan-500/20 rounded-full blur-3xl"></div>
            <div class="absolute -bottom-10 -right-10 w-32 h-32 bg-emerald-500/20 rounded-full blur-3xl"></div>
            
            <div class="text-center relative z-10 flex flex-col items-center justify-center">
                <!-- Main Time -->
                <div id="display" class="mono-font text-6xl min-[400px]:text-7xl font-bold text-white tracking-tight leading-none mb-1 drop-shadow-lg">
                    00:00:00
                </div>
                <!-- Milliseconds -->
                <div id="milliseconds" class="mono-font text-3xl min-[400px]:text-4xl font-medium text-cyan-400 drop-shadow-md">
                    .00
                </div>
            </div>
        </div>

        <!-- Controls -->
        <div class="grid grid-cols-3 gap-4 mb-6 shrink-0">
            <!-- Reset Button -->
            <button id="resetBtn" aria-label="Reset Timer" class="btn-press bg-slate-800 hover:bg-slate-700 text-slate-300 rounded-2xl py-5 flex flex-col items-center justify-center gap-1 shadow-lg disabled:opacity-30 disabled:cursor-not-allowed border border-slate-700/50" disabled>
                <i class="fa-solid fa-rotate-left text-xl"></i>
                <span class="text-[10px] font-bold uppercase tracking-wider">Reset</span>
            </button>

            <!-- Start/Stop Button (Main) -->
            <button id="startStopBtn" aria-label="Start Timer" class="btn-press bg-emerald-500 hover:bg-emerald-400 text-white rounded-2xl py-5 flex flex-col items-center justify-center gap-1 shadow-[0_0_20px_rgba(16,185,129,0.3)] border border-emerald-400/20">
                <i id="playIcon" class="fa-solid fa-play text-2xl pl-1"></i>
                <span id="playText" class="text-[10px] font-bold uppercase tracking-wider mt-1">Start</span>
            </button>

            <!-- Lap Button -->
            <button id="lapBtn" aria-label="Record Lap" class="btn-press bg-slate-800 hover:bg-slate-700 text-slate-300 rounded-2xl py-5 flex flex-col items-center justify-center gap-1 shadow-lg disabled:opacity-30 disabled:cursor-not-allowed border border-slate-700/50" disabled>
                <i class="fa-solid fa-flag text-xl"></i>
                <span class="text-[10px] font-bold uppercase tracking-wider">Lap</span>
            </button>
        </div>

        <!-- Laps Header & AI Button -->
        <div class="flex justify-between items-end px-4 mb-2 shrink-0">
            <div class="flex gap-4 text-[10px] font-bold text-slate-500 uppercase tracking-wider">
                <span>Lap</span>
                <span>Split</span>
            </div>
            
            <!-- AI Analysis Button -->
            <button id="analyzeBtn" onclick="analyzeSession()" class="hidden text-xs bg-gradient-to-r from-purple-500 to-indigo-600 text-white px-3 py-1.5 rounded-full font-medium shadow-lg hover:shadow-purple-500/30 transition-all items-center gap-2 transform hover:-translate-y-0.5 btn-press">
                <i class="fa-solid fa-wand-magic-sparkles text-yellow-300"></i> Coach Report
            </button>
        </div>

        <!-- Laps List Container -->
        <div class="flex-1 overflow-y-auto glass rounded-2xl relative min-h-0">
            <div id="lapsList" class="divide-y divide-slate-700/30">
                <!-- Empty State -->
                <div id="emptyState" class="h-full min-h-[150px] flex flex-col items-center justify-center text-slate-600">
                    <i class="fa-regular fa-clock text-3xl mb-3 opacity-30"></i>
                    <p class="text-xs font-medium opacity-50">Ready to track</p>
                </div>
            </div>
        </div>
    </div>

    <!-- AI Analysis Modal -->
    <div id="aiModal" class="fixed inset-0 z-50 flex items-center justify-center px-4 opacity-0 pointer-events-none transition-opacity duration-300">
        <div class="absolute inset-0 bg-black/80 backdrop-blur-sm" onclick="closeModal()"></div>
        <div class="bg-slate-800 border border-slate-700 rounded-2xl w-full max-w-sm p-6 relative z-10 shadow-2xl transform scale-95 transition-transform duration-300" id="modalContent">
            
            <button onclick="closeModal()" class="absolute top-4 right-4 text-slate-400 hover:text-white">
                <i class="fa-solid fa-xmark text-xl"></i>
            </button>

            <div class="flex items-center gap-3 mb-4">
                <div class="w-10 h-10 rounded-full bg-gradient-to-br from-purple-500 to-indigo-600 flex items-center justify-center shadow-lg">
                    <i class="fa-solid fa-robot text-white"></i>
                </div>
                <div>
                    <h3 class="font-bold text-white">Coach Gemini</h3>
                    <p class="text-xs text-slate-400">Performance Analysis</p>
                </div>
            </div>

            <!-- Loading State -->
            <div id="aiLoading" class="hidden flex-col items-center justify-center py-8">
                <div class="spinner mb-3"></div>
                <p class="text-xs text-purple-400 animate-pulse">Analyzing split times...</p>
            </div>

            <!-- Content State -->
            <div id="aiResult" class="text-slate-300 text-sm leading-relaxed prose prose-invert max-w-none">
                <!-- AI text goes here -->
            </div>
            
            <!-- Error State -->
            <div id="aiError" class="hidden text-center py-4">
                <i class="fa-solid fa-circle-exclamation text-rose-500 text-2xl mb-2"></i>
                <p class="text-xs text-rose-400">Analysis failed. Please try again.</p>
            </div>

        </div>
    </div>

    <script>
        // API Configuration
        const apiKey = ""; // System provides key
        
        // State Variables
        let startTime = 0;
        let elapsedTime = 0;
        let lastLapTime = 0; // For tracking individual lap duration
        let isRunning = false;
        let lapCounter = 1;
        let lapsData = []; // Store numeric data for AI: { lap: 1, duration: 2500, split: 2500 }

        // DOM Elements
        const displayEl = document.getElementById('display');
        const msEl = document.getElementById('milliseconds');
        const startStopBtn = document.getElementById('startStopBtn');
        const resetBtn = document.getElementById('resetBtn');
        const lapBtn = document.getElementById('lapBtn');
        const lapsList = document.getElementById('lapsList');
        const playIcon = document.getElementById('playIcon');
        const playText = document.getElementById('playText');
        const emptyState = document.getElementById('emptyState');
        const analyzeBtn = document.getElementById('analyzeBtn');

        // Modal Elements
        const aiModal = document.getElementById('aiModal');
        const modalContent = document.getElementById('modalContent');
        const aiLoading = document.getElementById('aiLoading');
        const aiResult = document.getElementById('aiResult');
        const aiError = document.getElementById('aiError');

        // Helper: Format Time
        function formatTime(time) {
            const minutes = String(Math.floor(time / 60000) % 60).padStart(2, '0');
            const seconds = String(Math.floor(time / 1000) % 60).padStart(2, '0');
            const hours = String(Math.floor(time / 3600000)).padStart(2, '0');
            const ms = String(Math.floor((time % 1000) / 10)).padStart(2, '0');
            
            const mainTime = (parseInt(hours) > 0) ? `${hours}:${minutes}:${seconds}` : `${minutes}:${seconds}`;
            
            return {
                main: mainTime,
                ms: `.${ms}`,
                full: `${hours}:${minutes}:${seconds}.${ms}`
            };
        }

        // Core: Update Display
        function updateDisplay() {
            const timeNow = Date.now();
            const time = elapsedTime + (timeNow - startTime);
            const formatted = formatTime(time);
            
            displayEl.textContent = formatted.main;
            msEl.textContent = formatted.ms;
        }

        // Core: Animation Loop
        function animate() {
            if (isRunning) {
                updateDisplay();
                requestAnimationFrame(animate);
            }
        }

        // Logic: Toggle Timer
        function toggleTimer() {
            if (isRunning) {
                // STOP
                isRunning = false;
                elapsedTime += Date.now() - startTime;
                
                // Visuals
                startStopBtn.className = "btn-press bg-emerald-500 hover:bg-emerald-400 text-white rounded-2xl py-5 flex flex-col items-center justify-center gap-1 shadow-[0_0_20px_rgba(16,185,129,0.3)] border border-emerald-400/20";
                playIcon.className = "fa-solid fa-play text-2xl pl-1";
                playText.textContent = "Resume";
                
                resetBtn.disabled = false;
                lapBtn.disabled = true;
                
                // Show AI Button if we have laps
                if (lapsData.length > 0) {
                    analyzeBtn.classList.remove('hidden');
                    analyzeBtn.classList.add('flex');
                }

            } else {
                // START
                isRunning = true;
                startTime = Date.now();
                animate();

                // Visuals
                startStopBtn.className = "btn-press bg-rose-500 hover:bg-rose-400 text-white rounded-2xl py-5 flex flex-col items-center justify-center gap-1 shadow-[0_0_20px_rgba(244,63,94,0.3)] border border-rose-400/20";
                playIcon.className = "fa-solid fa-pause text-2xl";
                playText.textContent = "Stop";

                resetBtn.disabled = true;
                lapBtn.disabled = false;
                
                // Hide AI Button while running
                analyzeBtn.classList.add('hidden');
                analyzeBtn.classList.remove('flex');
                
                if(emptyState) emptyState.style.display = 'none';
            }
        }

        // Logic: Reset
        function resetTimer() {
            if (isRunning) return;
            
            isRunning = false;
            elapsedTime = 0;
            startTime = 0;
            lastLapTime = 0;
            lapCounter = 1;
            lapsData = [];
            
            displayEl.textContent = "00:00";
            msEl.textContent = ".00";
            
            playText.textContent = "Start";
            playIcon.className = "fa-solid fa-play text-2xl pl-1";
            startStopBtn.className = "btn-press bg-emerald-500 hover:bg-emerald-400 text-white rounded-2xl py-5 flex flex-col items-center justify-center gap-1 shadow-[0_0_20px_rgba(16,185,129,0.3)] border border-emerald-400/20";

            resetBtn.disabled = true;
            lapBtn.disabled = true;
            
            // Hide AI Button
            analyzeBtn.classList.add('hidden');
            analyzeBtn.classList.remove('flex');
            
            lapsList.innerHTML = '';
            lapsList.appendChild(emptyState);
            emptyState.style.display = 'flex';
        }

        // Logic: Record Lap
        function recordLap() {
            if (!isRunning) return;

            const currentTotalTime = elapsedTime + (Date.now() - startTime);
            const lapDuration = currentTotalTime - lastLapTime;
            
            // Store data for AI
            lapsData.push({
                lap: lapCounter,
                duration_ms: lapDuration,
                total_ms: currentTotalTime,
                formatted: formatTime(currentTotalTime).full
            });

            lastLapTime = currentTotalTime;
            const formatted = formatTime(currentTotalTime);

            const lapItem = document.createElement('div');
            lapItem.className = "flex justify-between items-center p-4 hover:bg-white/5 transition-colors animate-fade-in group";
            
            lapItem.innerHTML = `
                <div class="flex items-center gap-3">
                    <span class="text-cyan-500 font-mono text-xs opacity-70 group-hover:opacity-100 transition-opacity">#${String(lapCounter).padStart(2, '0')}</span>
                </div>
                <div class="mono-font text-slate-200 font-medium tracking-wide">
                    ${formatted.full}
                </div>
            `;

            lapsList.insertBefore(lapItem, lapsList.firstChild);
            
            // Highlight effect
            lapItem.animate([
                { backgroundColor: 'rgba(6, 182, 212, 0.2)' },
                { backgroundColor: 'transparent' }
            ], { duration: 500 });

            lapCounter++;
        }

        // --- GEMINI AI INTEGRATION ---

        async function analyzeSession() {
            if (lapsData.length === 0) return;

            // Show Modal
            openModal();
            aiLoading.classList.remove('hidden');
            aiLoading.classList.add('flex');
            aiResult.innerHTML = '';
            aiError.classList.add('hidden');

            // Construct Prompt
            const dataString = JSON.stringify(lapsData.map(l => ({
                lap: l.lap,
                duration_ms: l.duration_ms
            })));
            
            const prompt = `
                You are an expert athletic coach. 
                Analyze these stopwatch lap times (in milliseconds): ${dataString}.
                1. Identify the fastest lap and the slowest lap.
                2. Comment on the pacing consistency (standard deviation or trends).
                3. Give a 1-sentence specific tip for improvement based on this data.
                Format the response in Markdown with bold key points. Be encouraging but technical. Keep it under 100 words.
            `;

            try {
                const response = await callGeminiWithRetry(prompt);
                
                // Parse markdown to HTML
                aiResult.innerHTML = marked.parse(response);
                
                aiLoading.classList.add('hidden');
                aiLoading.classList.remove('flex');
            } catch (error) {
                console.error("AI Error:", error);
                aiLoading.classList.add('hidden');
                aiLoading.classList.remove('flex');
                aiError.classList.remove('hidden');
            }
        }

        async function callGeminiWithRetry(prompt, retries = 3) {
            const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`;
            const payload = {
                contents: [{ parts: [{ text: prompt }] }]
            };

            for (let i = 0; i < retries; i++) {
                try {
                    const response = await fetch(url, {
                        method: 'POST',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify(payload)
                    });

                    if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
                    
                    const data = await response.json();
                    return data.candidates[0].content.parts[0].text;
                } catch (e) {
                    if (i === retries - 1) throw e;
                    await new Promise(r => setTimeout(r, 1000 * Math.pow(2, i)));
                }
            }
        }

        // --- Modal Logic ---

        function openModal() {
            aiModal.classList.remove('pointer-events-none', 'opacity-0');
            modalContent.classList.remove('scale-95');
            modalContent.classList.add('scale-100');
        }

        function closeModal() {
            aiModal.classList.add('pointer-events-none', 'opacity-0');
            modalContent.classList.remove('scale-100');
            modalContent.classList.add('scale-95');
        }

        // Event Listeners
        startStopBtn.addEventListener('click', toggleTimer);
        resetBtn.addEventListener('click', resetTimer);
        lapBtn.addEventListener('click', recordLap);

        // Keyboard Support
        document.addEventListener('keydown', (e) => {
            if (e.code === 'Space') {
                e.preventDefault();
                toggleTimer();
            }
            if (e.code === 'KeyR') resetTimer();
            if (e.code === 'KeyL') recordLap();
            if (e.code === 'Escape') closeModal();
        });

        // Add Animation Styles
        const styleSheet = document.createElement("style");
        styleSheet.innerText = `
            @keyframes fadeIn {
                from { opacity: 0; transform: translateY(-5px); }
                to { opacity: 1; transform: translateY(0); }
            }
            .animate-fade-in {
                animation: fadeIn 0.3s cubic-bezier(0.4, 0, 0.2, 1) forwards;
            }
        `;
        document.head.appendChild(styleSheet);
    </script>
</body>
</html>
