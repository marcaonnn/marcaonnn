<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>DevMarket Analytics</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body { background-color: #0f172a; color: #f1f5f9; font-family: 'Inter', sans-serif; }
        .bg-panel { background-color: #1e293b; border: 1px solid #334155; }
        .chart-container { position: relative; height: 300px; width: 100%; }
        .cursor-blink { animation: blink 1s step-end infinite; border-left: 4px solid #10b981; margin-left: 5px; }
        @keyframes blink { 50% { opacity: 0; } }
    </style>
</head>
<body class="p-6">
    <header class="mb-8">
        <h1 class="text-3xl font-bold text-emerald-400">DevMarket Analytics</h1>
        <p class="text-gray-400">Índice Parabólico de Código</p>
    </header>

    <div class="flex flex-col md:flex-row gap-6">
        <div class="w-full md:w-2/3 bg-panel p-6 rounded-2xl shadow-lg">
            <input type="text" id="usernameInput" placeholder="Digite seu usuário do GitHub..." class="w-full p-3 rounded-lg bg-slate-800 border border-slate-600 mb-4 text-white">
            <button onclick="fetchGitHubData()" class="w-full bg-emerald-600 hover:bg-emerald-500 text-white font-bold py-3 rounded-lg transition">Analisar Perfil</button>
            <div class="chart-container mt-6">
                <canvas id="evolutionChart"></canvas>
            </div>
        </div>

        <div class="w-full md:w-1/3 flex flex-col gap-6">
            <div id="languageList" class="bg-panel p-6 rounded-2xl shadow-lg h-full">
                <h2 class="text-lg font-semibold mb-4 text-gray-300">Dominância por Linguagem</h2>
                <div id="langItems" class="space-y-4"></div>
            </div>
        </div>
    </div>

    <section class="mt-6 bg-panel p-6 rounded-2xl">
        <h2 class="text-xl font-bold text-emerald-400 mb-2">Briefing de Evolução</h2>
        <p id="briefingText" class="text-gray-200 font-mono text-sm"></p>
    </section>

    <script>
        let chart = new Chart(document.getElementById('evolutionChart').getContext('2d'), {
            type: 'line',
            data: { labels: [], datasets: [{ data: [], borderColor: '#10b981', tension: 0.4, fill: true, backgroundColor: 'rgba(16, 185, 129, 0.1)' }] },
            options: { responsive: true, maintainAspectRatio: false, plugins: { legend: { display: false } } }
        });

        async function fetchGitHubData() {
            const user = document.getElementById('usernameInput').value;
            const res = await fetch(`https://api.github.com/users/${user}/repos?per_page=100`);
            const repos = await res.json();
            
            const langs = {};
            let total = 0;
            repos.forEach(r => { if(r.language) { langs[r.language] = (langs[r.language] || 0) + 1; total++; } });

            chart.data.labels = Object.keys(langs);
            chart.data.datasets[0].data = Object.values(langs);
            chart.update();

            const container = document.getElementById('langItems');
            container.innerHTML = Object.entries(langs).map(([name, count]) => `
                <div><div class="flex justify-between text-xs mb-1"><span>${name}</span><span>${((count/total)*100).toFixed(0)}%</span></div>
                <div class="w-full bg-slate-700 rounded-full h-2"><div class="bg-emerald-500 h-2 rounded-full" style="width: ${(count/total)*100}%"></div></div></div>
            `).join('');

            typeBriefing(`Desenvolvedor ${user.toUpperCase()} analisado. Baseado em ${repos.length} repositórios, a evolução parabólica indica domínio proeminente em ${Object.keys(langs)[0] || 'geral'}. O sistema projeta um crescimento exponencial na carreira.`);
        }

        function typeBriefing(text) {
            const el = document.getElementById('briefingText');
            el.innerHTML = ""; let i = 0;
            function type() { if (i < text.length) { el.innerHTML += text.charAt(i); i++; setTimeout(type, 30); } }
            type();
        }
    </script>
</body>
</html>
