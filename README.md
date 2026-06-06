<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>DevMarket Cap - Gráfico Parabólico de Repositórios</title>
    
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    
    <!-- Chart.js -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    
    <!-- Google Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;700&family=JetBrains+Mono:wght@400;700&display=swap" rel="stylesheet">
    
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #0b0e11; /* Tema escuro financeiro */
            color: #eaecef;
        }
        .font-mono {
            font-family: 'JetBrains Mono', monospace;
        }
        /* Custom Scrollbar */
        ::-webkit-scrollbar {
            width: 6px;
        }
        ::-webkit-scrollbar-track {
            background: #181a20; 
        }
        ::-webkit-scrollbar-thumb {
            background: #2b3139; 
            border-radius: 4px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #474d57; 
        }
        /* Efeito Glow para textos financeiros */
        .glow-text {
            text-shadow: 0 0 10px rgba(14, 203, 129, 0.5);
        }
        /* Piscar do cursor da máquina de escrever */
        .cursor-blink {
            animation: blink 1s step-end infinite;
        }
        @keyframes blink {
            50% { opacity: 0; }
        }
        
        /* Container do chart */
        .chart-container {
            position: relative;
            height: 400px;
            width: 100%;
        }
        
        /* Estilo dos inputs */
        .bg-panel {
            background-color: #181a20;
            border: 1px solid #2b3139;
        }
    </style>
</head>
<body class="min-h-screen flex flex-col items-center p-4 sm:p-8">

    <div class="w-full max-w-6xl flex flex-col gap-6">
        
        <!-- Header -->
        <header class="flex flex-col sm:flex-row justify-between items-center bg-panel p-6 rounded-xl shadow-lg">
            <div class="flex items-center gap-4 mb-4 sm:mb-0">
                <div class="w-12 h-12 rounded-full bg-emerald-500/20 flex items-center justify-center border border-emerald-500">
                    <svg class="w-6 h-6 text-emerald-500" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 7h8m0 0v8m0-8l-8 8-4-4-6 6"></path></svg>
                </div>
                <div>
                    <h1 class="text-2xl font-bold tracking-tight glow-text text-emerald-400">DevMarket Analytics</h1>
                    <p class="text-xs text-gray-400 font-mono">ÍNDICE PARABÓLICO DE CÓDIGO (IPC)</p>
                </div>
            </div>
            
            <div class="flex gap-2 w-full sm:w-auto">
                <input type="text" id="usernameInput" placeholder="Usuário do GitHub..." class="w-full sm:w-64 bg-[#0b0e11] border border-gray-700 rounded-lg px-4 py-2 text-white font-mono focus:outline-none focus:border-emerald-500 transition-colors">
                <button onclick="fetchUserData()" class="bg-emerald-600 hover:bg-emerald-500 text-white px-6 py-2 rounded-lg font-semibold transition-colors flex items-center gap-2">
                    Analisar
                </button>
            </div>
        </header>

        <main class="grid grid-cols-1 lg:grid-cols-3 gap-6">
            
            <!-- Coluna Esquerda: Gráfico (2/3) -->
            <div class="lg:col-span-2 bg-panel rounded-xl p-6 shadow-lg flex flex-col relative">
                <div class="flex justify-between items-center mb-4">
                    <h2 class="text-lg font-semibold text-gray-200">Ação de Preço (Repositórios Criados)</h2>
                    <div class="flex gap-4 font-mono text-sm">
                        <span class="text-gray-400">TICKER: <span id="tickerName" class="text-white">---</span></span>
                        <span class="text-gray-400">VOL: <span id="volTotal" class="text-white">0</span></span>
                    </div>
                </div>
                
                <!-- Loading Overlay -->
                <div id="loadingOverlay" class="absolute inset-0 bg-[#181a20]/80 z-10 hidden flex-col items-center justify-center rounded-xl backdrop-blur-sm">
                    <div class="w-10 h-10 border-4 border-emerald-500 border-t-transparent rounded-full animate-spin"></div>
                    <p class="mt-4 text-emerald-500 font-mono animate-pulse">Sincronizando com a Blockchain do GitHub...</p>
                </div>
                
                <div class="chart-container">
                    <canvas id="evolutionChart"></canvas>
                </div>
            </div>

            <div class="bg-panel rounded-xl p-6 shadow-lg flex flex-col h-[500px]">
                <h2 class="text-lg font-semibold text-gray-200 mb-4 border-b border-gray-700 pb-2">Liquidez por Linguagem</h2>
                
                <!-- Lista de Linguagens -->
                <div id="languagesList" class="flex-1 overflow-y-auto pr-2 space-y-4">
                    <div class="text-center text-gray-500 mt-20 font-mono text-sm">
                        Aguardando input do usuário...
                    </div>
                </div>
            </div>
        </main>

        <section class="bg-panel rounded-xl p-6 shadow-lg">
            <h2 class="text-lg font-semibold text-gray-200 mb-4 flex items-center gap-2">
                <svg class="w-5 h-5 text-emerald-500" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"></path></svg>
                Terminal de Perfil e Evolução
            </h2>
            <div class="bg-[#0b0e11] rounded-lg p-4 font-mono text-sm text-emerald-400 min-h-[120px] relative border border-gray-800">
                <span class="text-gray-500 select-none">>_ </span>
                <span id="briefingText">Sistema pronto. Aguardando inicialização do ticker do desenvolvedor.</span><span id="cursor" class="cursor-blink inline-block w-2 bg-emerald-400 ml-1 h-4 align-middle"></span>
            </div>
        </section>
        
        <!-- Alertas de Erro -->
        <div id="errorAlert" class="hidden bg-red-900/50 border border-red-500 text-red-200 px-4 py-3 rounded-lg relative font-mono text-sm" role="alert">
            <strong class="font-bold">Erro de Execução: </strong>
            <span class="block sm:inline" id="errorMessage">Usuário não encontrado.</span>
        </div>

    </div>

    <script>
        // Variáveis Globais
        let myChart = null;
        let isTyping = false;
        let typeTimeout = null;

        // Cores Financeiras
        const colors = {
            emerald: '#0ecb81',
            red: '#f6465d',
            bgDark: '#0b0e11',
            panel: '#181a20',
            grid: '#2b3139',
            text: '#848e9c',
            textHighlight: '#eaecef'
        };

        // Paleta para linguagens
        const langColors = ['#0ecb81', '#f6465d', '#fcd53f', '#2962ff', '#e040fb', '#00e5ff', '#ff9100'];

        function initChart() {
            const ctx = document.getElementById('evolutionChart').getContext('2d');
            
            // Gradiente para o efeito "parabólico financeiro"
            const gradient = ctx.createLinearGradient(0, 0, 0, 400);
            gradient.addColorStop(0, 'rgba(14, 203, 129, 0.4)'); // Verde em cima
            gradient.addColorStop(1, 'rgba(14, 203, 129, 0.0)'); // Transparente embaixo

            myChart = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: ['Jan', 'Fev', 'Mar', 'Abr', 'Mai', 'Jun'],
                    datasets: [{
                        label: 'Acumulação Base',
                        data: [0, 0, 0, 0, 0, 0],
                        borderColor: colors.emerald,
                        backgroundColor: gradient,
                        borderWidth: 2,
                        pointRadius: 0,
                        pointHitRadius: 10,
                        pointHoverRadius: 5,
                        pointHoverBackgroundColor: '#fff',
                        fill: true,
                        tension: 0.5 // Cria o efeito parabólico/curvado suave
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    interaction: {
                        mode: 'index',
                        intersect: false,
                    },
                    plugins: {
                        legend: { display: false },
                        tooltip: {
                            backgroundColor: 'rgba(24, 26, 32, 0.9)',
                            titleColor: colors.emerald,
                            bodyColor: colors.textHighlight,
                            borderColor: colors.grid,
                            borderWidth: 1,
                            padding: 10,
                            displayColors: false,
                            callbacks: {
                                label: function(context) {
                                    return `Volume Total: ${context.parsed.y} repos`;
                                }
                            }
                        }
                    },
                    scales: {
                        x: {
                            grid: { color: colors.grid, drawBorder: false },
                            ticks: { color: colors.text, font: { family: 'JetBrains Mono', size: 10 } }
                        },
                        y: {
                            position: 'right', // Estilo de mercado financeiro
                            grid: { color: colors.grid, drawBorder: false },
                            ticks: { color: colors.text, font: { family: 'JetBrains Mono', size: 10 } },
                            beginAtZero: true
                        }
                    },
                    animation: {
                        duration: 2000,
                        easing: 'easeOutQuart'
                    }
                }
            });
        }

        async function fetchUserData() {
            const username = document.getElementById('usernameInput').value.trim();
            if (!username) return;

            // UI Feedback
            document.getElementById('errorAlert').classList.add('hidden');
            document.getElementById('loadingOverlay').classList.remove('hidden');
            document.getElementById('languagesList').innerHTML = '';
            
            try {
                // Buscar repositórios (paginando até 100 para simplificar, mas traz os mais recentes/populares)
                const response = await fetch(`https://api.github.com/users/${username}/repos?per_page=100&sort=created&direction=asc`);
                
                if (!response.ok) {
                    throw new Error(response.status === 404 ? 'Usuário não encontrado' : 'Erro na comunicação com o servidor');
                }

                const repos = await response.json();
                
                if (repos.length === 0) {
                    throw new Error('Este usuário não possui repositórios públicos.');
                }

                processData(repos, username);

            } catch (error) {
                document.getElementById('errorAlert').classList.remove('hidden');
                document.getElementById('errorMessage').innerText = error.message;
                typeWriter("Erro no processamento. Conexão recusada ou ticker inválido.", document.getElementById('briefingText'));
            } finally {
                document.getElementById('loadingOverlay').classList.add('hidden');
            }
        }

        function processData(repos, username) {
            // Arrays para o gráfico
            const timeLabels = [];
            const cumulativeData = [];
            
            // Mapas para estatísticas
            const languages = {};
            let totalRepos = 0;
            let currentAccumulation = 0;
            let firstDate = null;

            // Ordenar por data de criação (antigo para o novo)
            repos.sort((a, b) => new Date(a.created_at) - new Date(b.created_at));

            repos.forEach(repo => {
                // Gráfico Parabólico (Crescimento no tempo)
                const date = new Date(repo.created_at);
                if (!firstDate) firstDate = date;
                
                const dateString = `${date.getMonth()+1}/${date.getFullYear().toString().substr(-2)}`;
                
                currentAccumulation++;
                timeLabels.push(dateString);
                cumulativeData.push(currentAccumulation);

                // Cálculo de Linguagens
                if (repo.language) {
                    languages[repo.language] = (languages[repo.language] || 0) + 1;
                    totalRepos++;
                }
            });

            // Atualizar UI Textual Superior
            document.getElementById('tickerName').innerText = username.toUpperCase();
            document.getElementById('volTotal').innerText = repos.length;

            // Atualizar Gráfico
            updateChart(timeLabels, cumulativeData);

            // Ordenar e renderizar linguagens
            const sortedLangs = Object.entries(languages)
                .sort((a, b) => b[1] - a[1]); // Ordena do maior pro menor

            renderLanguages(sortedLangs, totalRepos);
            
            // Gerar Briefing Dinâmico
            generateBriefing(username, sortedLangs, totalRepos, repos.length, firstDate);
        }

        function updateChart(labels, data) {
            const ctx = document.getElementById('evolutionChart').getContext('2d');
            
            // Recriar gradiente baseado no tema
            const gradient = ctx.createLinearGradient(0, 0, 0, 400);
            gradient.addColorStop(0, 'rgba(14, 203, 129, 0.4)');
            gradient.addColorStop(1, 'rgba(14, 203, 129, 0.0)');

            // Calcular a inclinação para ver se está "Bullish" (Verde) ou estagnado
            const isBullish = data.length > 1 && (data[data.length-1] > data[data.length-2]);
            const lineColor = isBullish ? colors.emerald : colors.emerald; 

            myChart.data.labels = labels;
            myChart.data.datasets[0].data = data;
            myChart.data.datasets[0].borderColor = lineColor;
            myChart.data.datasets[0].backgroundColor = gradient;
            
            // Força a animação de "desenho" da esquerda para a direita novamente
            myChart.options.animation = {
                x: {
                    type: 'number',
                    easing: 'linear',
                    duration: 1500,
                    from: NaN, // Cria o efeito de linha traçando
                    delay: (ctx) => {
                        if (ctx.type !== 'data' || ctx.xStarted) {
                            return 0;
                        }
                        ctx.xStarted = true;
                        return ctx.index * 10;
                    }
                },
                y: {
                    type: 'number',
                    easing: 'linear',
                    duration: 1500,
                    from: (ctx) => {
                        return ctx.index === 0 ? ctx.chart.scales.y.getPixelForValue(100) : ctx.chart.getDatasetMeta(ctx.datasetIndex).data[ctx.index - 1].getProps(['y'], true).y;
                    },
                    delay: (ctx) => {
                        if (ctx.type !== 'data' || ctx.yStarted) {
                            return 0;
                        }
                        ctx.yStarted = true;
                        return ctx.index * 10;
                    }
                }
            };

            myChart.update();
        }

        function renderLanguages(sortedLangs, totalCount) {
            const container = document.getElementById('languagesList');
            container.innerHTML = '';

            if (sortedLangs.length === 0) {
                container.innerHTML = '<div class="text-gray-500 font-mono text-sm mt-10">Ativos não identificados (Repositórios sem linguagem definida).</div>';
                return;
            }

            sortedLangs.forEach((lang, index) => {
                const langName = lang[0];
                const count = lang[1];
                const percentage = ((count / totalCount) * 100).toFixed(1);
                const color = langColors[index % langColors.length]; // Pega uma cor da paleta

                const html = `
                    <div class="mb-4">
                        <div class="flex justify-between items-end mb-1">
                            <span class="text-sm font-semibold text-gray-200">${langName}</span>
                            <div class="flex items-center gap-2">
                                <span class="text-xs text-gray-500">${count} proj.</span>
                                <span class="text-sm font-mono" style="color: ${color}">${percentage}%</span>
                            </div>
                        </div>
                        <div class="w-full bg-[#0b0e11] rounded-full h-2.5 border border-gray-800">
                            <div class="h-2.5 rounded-full" style="width: 0%; background-color: ${color}; transition: width 1s ease-out;" data-width="${percentage}%"></div>
                        </div>
                    </div>
                `;
                container.insertAdjacentHTML('beforeend', html);
            });

            // Animar as barras após um pequeno delay para a DOM renderizar
            setTimeout(() => {
                const bars = container.querySelectorAll('.h-2\\.5.rounded-full');
                bars.forEach(bar => {
                    bar.style.width = bar.getAttribute('data-width');
                });
            }, 100);
        }

        function generateBriefing(username, sortedLangs, totalWithLang, totalAbsolute, firstDate) {
            let briefing = `ANÁLISE DE MERCADO: Desenvolvedor [${username.toUpperCase()}].\n\n`;
            
            const anosDeAtuacao = new Date().getFullYear() - firstDate.getFullYear();
            
            // Cálculo de Nível (Arbitrário/Lúdico)
            let level = "Júnior";
            if (totalAbsolute > 30 || anosDeAtuacao > 4) level = "Sênior";
            else if (totalAbsolute > 15 || anosDeAtuacao > 2) level = "Pleno";

            briefing += `O ativo indica forte consolidação no nível [${level}], acumulando capital de código desde ${firstDate.getFullYear()} (aprox. ${anosDeAtuacao} anos em operação). O gráfico parabólico evidencia uma taxa de commits e deploy constante, típico de perfis com alta liquidez técnica.\n\n`;

            if (sortedLangs.length > 0) {
                const topLang = sortedLangs[0][0];
                const topPerc = ((sortedLangs[0][1] / totalWithLang) * 100).toFixed(1);
                
                briefing += `>> ATIVO PRINCIPAL: Seu maior monopólio é em [${topLang}], que domina expressivos ${topPerc}% da sua carteira de projetos. \n`;
                
                if (topPerc >= 70) {
                    briefing += `Esta hiper-concentração sugere um perfil de ESPECIALISTA de alta precisão. Cuidado com oscilações de mercado tecnológico.\n`;
                } else if (sortedLangs.length > 4) {
                    briefing += `Com investimentos em ${sortedLangs.length} diferentes stacks, nota-se uma matriz diversificada (POLIGLOTA), mitigando riscos e abraçando versatilidade estrutural.\n`;
                }

                if (sortedLangs.length > 1) {
                    briefing += `>> EMERGENTES: Fortes tendências secundárias operando em [${sortedLangs[1][0]}] e [${sortedLangs[2] ? sortedLangs[2][0] : 'outras'}].\n`;
                }
            }

            briefing += `\n>> PARECER TÉCNICO: Tendência de alta (BULL MARKET) mantida. Recomenda-se continuar aportando em projetos open-source para maximizar os dividendos intelectuais.`;

            // Chamar efeito de máquina de escrever
            const outputElement = document.getElementById('briefingText');
            typeWriter(briefing, outputElement);
        }

        function typeWriter(text, element) {
            // Resetar o estado anterior
            if (typeTimeout) clearTimeout(typeTimeout);
            element.innerHTML = '';
            
            let i = 0;
            // Tratar quebras de linha substituindo \n por <br>
            const formattedText = text.replace(/\n/g, '<br>');
            
            // Para não escrever as tags HTML letra por letra, usamos uma abordagem mais inteligente:
            // Escrevemos em um buffer temporário invisível e jogamos no innerHTML.
            // Para simplificar a animação pura, converteremos <br> em caracteres especiais para tratar no loop
            
            const chars = text.split('');

            function type() {
                if (i < chars.length) {
                    if (chars[i] === '\n') {
                        element.innerHTML += '<br>';
                    } else {
                        element.innerHTML += chars[i];
                    }
                    i++;
                    
                    // Velocidade variável para parecer mais "robótico/terminal"
                    const speed = Math.random() * 20 + 10; 
                    typeTimeout = setTimeout(type, speed);
                }
            }
            
            type();
        }

        // Permitir busca pelo "Enter"
        document.getElementById('usernameInput').addEventListener('keypress', function (e) {
            if (e.key === 'Enter') {
                fetchUserData();
            }
        });

        // Inicializar aplicação com gráfico vazio
        window.onload = () => {
            initChart();
        };

    </script>
</body>
</html>
