<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>DevMarket Analytics - Terminal</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;700&family=JetBrains+Mono:wght@400;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; background-color: #0b0e11; color: #eaecef; }
        .font-mono { font-family: 'JetBrains Mono', monospace; }
        .glow-text { text-shadow: 0 0 10px rgba(14, 203, 129, 0.5); }
        .cursor-blink { animation: blink 1s step-end infinite; }
        @keyframes blink { 50% { opacity: 0; } }
        .bg-panel { background-color: #181a20; border: 1px solid #2b3139; }
    </style>
</head>

<body class="min-h-screen flex flex-col items-center p-4 sm:p-8">
    <div class="w-full max-w-6xl flex flex-col gap-6">
        <header class="flex flex-col sm:flex-row justify-between items-center bg-panel p-6 rounded-xl shadow-lg">
            <div class="flex items-center gap-4">
                <div class="w-12 h-12 rounded-full bg-emerald-500/20 flex items-center justify-center border border-emerald-500">
                    <svg class="w-6 h-6 text-emerald-500" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 7h8m0 0v8m0-8l-8 8-4-4-6 6"></path></svg>
                </div>
                <div>
                    <h1 class="text-2xl font-bold glow-text text-emerald-400">DevMarket Analytics</h1>
                    <p class="text-xs text-gray-400 font-mono">IPC: ÍNDICE PARABÓLICO DE CÓDIGO</p>
                </div>
            </div>
            <div class="flex gap-2 w-full sm:w-auto mt-4 sm:mt-0">
                <input type="text" id="usernameInput" placeholder="Usuário GitHub..." class="w-full sm:w-64 bg-[#0b0e11] border border-gray-700 rounded-lg px-4 py-2 text-white font-mono focus:outline-none focus:border-emerald-500">
                <button onclick="fetchUserData()" class="bg-emerald-600 hover:bg-emerald-500 text-white px-6 py-2 rounded-lg font-semibold">Analisar</button>
            </div>
        </header>

        <main class="grid grid-cols-1 lg:grid-cols-3 gap-6">
            <div class="lg:col-span-2 bg-panel rounded-xl p-6 shadow-lg relative">
                <h2 class="text-lg font-semibold text-gray-200 mb-4">Ação de Preço (Repositórios)</h2>
                <div id="loadingOverlay" class="absolute inset-0 bg-[#181a20]/90 hidden z-10 flex-col items-center justify-center rounded-xl">
                    <div class="w-10 h-10 border-4 border-emerald-500 border-t-transparent rounded-full animate-spin"></div>
                </div>
                <div class="h-[400px]"><canvas id="evolutionChart"></canvas></div>
            </div>
            <div class="bg-panel rounded-xl p-6 shadow-lg h-[460px]">
                <h2 class="text-lg font-semibold text-gray-200 mb-4 border-b border-gray-700 pb-2">Liquidez por Linguagem</h2>
                <div id="languagesList" class="overflow-y-auto space-y-4"></div>
            </div>
        </main>

        <section class="bg-panel rounded-xl p-6 shadow-lg">
            <div class="bg-[#0b0e11] rounded-lg p-4 font-mono text-sm text-emerald-400 min-h-[120px] border border-gray-800">
                <span id="briefingText">Aguardando ticker do desenvolvedor...</span><span class="cursor-blink inline-block w-2 bg-emerald-400 h-4"></span>
            </div>
        </section>
    </div>

    <script>
        let myChart = null;
        const colors = { emerald: '#0ecb81', grid: '#2b3139', text: '#848e9c' };

        function initChart() {
            const ctx = document.getElementById('evolutionChart').getContext('2d');
            myChart = new Chart(ctx, {
                type: 'line',
                data: { labels: [], datasets: [{ data: [], borderColor: colors.emerald, tension: 0.4, fill: true, backgroundColor: 'rgba(14, 203, 129, 0.1)' }] },
                options: { responsive: true, maintainAspectRatio: false, plugins: { legend: { display: false } }, scales: { x: { grid: { color: colors.grid } }, y: { position: 'right', grid: { color: colors.grid } } } }
            });
        }

        async function fetchUserData() {
            const username = document.getElementById('usernameInput').value.trim();
            if (!username) return;
            document.getElementById('loadingOverlay').classList.remove('hidden');
            try {
                const response = await fetch(`https://api.github.com/users/${username}/repos?per_page=100&sort=created&direction=asc`);
                const repos = await response.json();
                processData(repos, username);
            } catch (e) { alert("Erro ao buscar dados"); }
            finally { document.getElementById('loadingOverlay').classList.add('hidden'); }
        }

        function processData(repos, username) {
            const langs = {};
            let total = 0;
            repos.forEach(r => { if (r.language) { langs[r.language] = (langs[r.language] || 0) + 1; total++; } });
            
            // Render
            updateChart(repos.map((_, i) => i + 1), repos.map((_, i) => i + 1));
            renderLangs(Object.entries(langs).sort((a,b) => b[1]-a[1]), total);
            typeBriefing(`ANÁLISE DE MERCADO: [${username.toUpperCase()}]. Ativo com ${repos.length} posições abertas. Linguagem dominante: ${Object.keys(langs)[0] || 'Indefinida'}. Tendência de alta confirmada.`);
        }

        function updateChart(labels, data) {
            myChart.data.labels = labels;
            myChart.data.datasets[0].data = data;
            myChart.update();
        }

        function renderLangs(langs, total) {
            const container = document.getElementById('languagesList');
            container.innerHTML = langs.map(([name, count]) => `
                <div>
                    <div class="flex justify-between text-xs mb-1"><span>${name}</span><span>${((count/total)*100).toFixed(1)}%</span></div>
                    <div class="w-full bg-gray-800 rounded-full h-2"><div class="h-2 bg-emerald-500 rounded-full" style="width: ${(count/total)*100}%"></div></div>
                </div>`).join('');
        }

        function typeBriefing(text) {
            const el = document.getElementById('briefingText');
            el.innerHTML = '';
            let i = 0;
            function type() {
                if (i < text.length) { el.innerHTML += text.charAt(i); i++; setTimeout(type, 30); }
            }
            type();
        }

        window.onload = initChart;
    </script>
</body>
</html>
