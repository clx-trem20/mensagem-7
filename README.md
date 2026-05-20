<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ecos do Tempo - Árvore de Recados</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Google Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Cinzel:wght@600;800&family=Plus+Jakarta+Sans:wght@300;400;600;800&family=Playfair+Display:ital,wght@0,700;1,400&display=swap" rel="stylesheet">
    <!-- FontAwesome for Ornament Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <style>
        body {
            font-family: 'Plus Jakarta Sans', sans-serif;
            background: radial-gradient(circle at center, #0d1b2a 0%, #010811 100%);
            overflow-x: hidden;
        }
        .serif-title {
            font-family: 'Cinzel', serif;
        }
        .handwritten {
            font-family: 'Playfair Display', serif;
            font-style: italic;
        }
        /* Custom Glowing effects */
        .glow-gold {
            text-shadow: 0 0 10px rgba(234, 179, 8, 0.6), 0 0 20px rgba(234, 179, 8, 0.4);
        }
        .glow-box-gold {
            box-shadow: 0 0 15px rgba(234, 179, 8, 0.3);
        }
        .glow-box-green {
            box-shadow: 0 0 15px rgba(16, 185, 129, 0.3);
        }
        /* Snowy background canvas */
        #snowCanvas {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            z-index: 1;
        }
        .content-container {
            position: relative;
            z-index: 2;
        }
        /* Custom scrollbar */
        ::-webkit-scrollbar {
            width: 6px;
        }
        ::-webkit-scrollbar-track {
            background: rgba(13, 27, 42, 0.8);
        }
        ::-webkit-scrollbar-thumb {
            background: #eab308;
            border-radius: 3px;
        }
        /* Ornament pulse animation */
        @keyframes pulse-slow {
            0%, 100% { transform: scale(1) translate(-50%, -50%); filter: drop-shadow(0 0 2px rgba(255,255,255,0.4)); }
            50% { transform: scale(1.1) translate(-45%, -45%); filter: drop-shadow(0 0 8px rgba(255,255,255,0.8)); }
        }
        .ornament-placed {
            animation: pulse-slow 3s infinite ease-in-out;
            transition: all 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
        }
        .ornament-placed:hover {
            transform: scale(1.3) translate(-38%, -38%) !important;
            z-index: 50 !important;
            filter: drop-shadow(0 0 12px rgba(255,255,255,1)) !important;
        }
    </style>
</head>
<body class="text-slate-100 min-h-screen pb-12">

    <!-- Canvas para Neve caindo -->
    <canvas id="snowCanvas"></canvas>

    <div class="content-container max-w-4xl mx-auto px-4 pt-6">
        
        <!-- HEADER GLOBAL -->
        <header class="text-center mb-8 flex flex-col items-center">
            <div class="flex items-center gap-2 mb-2 bg-slate-900/60 backdrop-blur-md px-4 py-1.5 rounded-full border border-yellow-500/20 shadow-lg">
                <span class="text-yellow-400 animate-pulse text-sm"><i class="fa-solid fa-tree"></i></span>
                <span class="text-xs uppercase tracking-widest text-slate-300 font-semibold">Ecos do Tempo</span>
            </div>
            <h1 class="serif-title text-3xl md:text-5xl font-extrabold tracking-wide text-transparent bg-clip-text bg-gradient-to-r from-yellow-200 via-amber-300 to-yellow-500 mb-1">
                Árvore de Recados
            </h1>
            <p class="text-slate-400 text-sm max-w-md">Pendure mensagens misteriosas que se revelam na hora mágica marcada pelo criador.</p>
        </header>

        <!-- AREA DE CARREGAMENTO INICIAL -->
        <div id="loadingScreen" class="flex flex-col items-center justify-center py-20 bg-slate-900/40 backdrop-blur-lg border border-slate-800 rounded-3xl p-8 text-center shadow-2xl">
            <div class="w-16 h-16 border-4 border-yellow-500 border-t-transparent rounded-full animate-spin mb-6"></div>
            <p class="text-slate-300 font-semibold text-lg animate-pulse">Conectando ao mundo dos desejos...</p>
            <p class="text-slate-500 text-sm mt-2">Por favor, aguarde um instante.</p>
        </div>

        <!-- 1. TELA DE CRIAÇÃO (Exibida apenas quando NÃO houver link/parâmetro de árvore ativo) -->
        <div id="homeScreen" class="hidden space-y-8">
            <div class="bg-gradient-to-b from-slate-900/90 to-slate-950/90 backdrop-blur-lg border border-slate-800 rounded-3xl p-6 md:p-8 shadow-2xl">
                <h2 class="serif-title text-2xl font-bold text-yellow-400 mb-4 flex items-center gap-2">
                    <i class="fa-solid fa-wand-magic-sparkles text-xl"></i> Cultive sua Árvore
                </h2>
                <p class="text-slate-300 text-sm leading-relaxed mb-6">
                    Você entrou no painel de criação! Crie um espaço mágico online onde seus amigos podem pendurar presentes, cartinhas e recados de carinho. Você define o dia exato em que os recados serão revelados! Até lá, as cartinhas ficam guardadas a sete chaves.
                </p>

                <form id="createTreeForm" class="space-y-5">
                    <!-- Nome da Árvore -->
                    <div>
                        <label class="block text-xs font-bold uppercase tracking-wider text-slate-400 mb-2">Seu Nome ou Nome do Grupo</label>
                        <input type="text" id="treeOwnerName" required placeholder="Ex: Lucas, Família Silva, Terceirão" 
                               class="w-full bg-slate-900/80 border border-slate-700 rounded-xl px-4 py-3 text-slate-100 placeholder-slate-500 focus:outline-none focus:border-yellow-500 focus:ring-1 focus:ring-yellow-500 transition-all">
                    </div>

                    <!-- Escolha do Tema -->
                    <div>
                        <label class="block text-xs font-bold uppercase tracking-wider text-slate-400 mb-2">Tema Visual da Árvore</label>
                        <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                            <!-- Tema 1 -->
                            <label class="cursor-pointer group relative block">
                                <input type="radio" name="treeTheme" value="christmas" checked class="peer hidden">
                                <div class="bg-slate-900/60 border-2 border-slate-800 rounded-2xl p-4 transition-all group-hover:border-emerald-600 peer-checked:border-emerald-500 peer-checked:bg-emerald-950/20">
                                    <div class="text-emerald-400 text-2xl mb-1"><i class="fa-solid fa-holly-berry"></i></div>
                                    <h3 class="font-bold text-slate-200 text-sm">Clássico de Natal</h3>
                                    <p class="text-xs text-slate-500 mt-1">Elegante pinheiro com luzes aconchegantes e tom verde tradicional.</p>
                                </div>
                            </label>
                            <!-- Tema 2 -->
                            <label class="cursor-pointer group relative block">
                                <input type="radio" name="treeTheme" value="enchanted" class="peer hidden">
                                <div class="bg-slate-900/60 border-2 border-slate-800 rounded-2xl p-4 transition-all group-hover:border-purple-600 peer-checked:border-purple-500 peer-checked:bg-purple-950/20">
                                    <div class="text-purple-400 text-2xl mb-1"><i class="fa-solid fa-wand-magic-sparkles"></i></div>
                                    <h3 class="font-bold text-slate-200 text-sm">Cúpula Encantada</h3>
                                    <p class="text-xs text-slate-500 mt-1">Um visual místico de árvores fantásticas com néons roxos e azuis.</p>
                                </div>
                            </label>
                            <!-- Tema 3 -->
                            <label class="cursor-pointer group relative block">
                                <input type="radio" name="treeTheme" value="golden" class="peer hidden">
                                <div class="bg-slate-900/60 border-2 border-slate-800 rounded-2xl p-4 transition-all group-hover:border-yellow-600 peer-checked:border-yellow-500 peer-checked:bg-yellow-950/20">
                                    <div class="text-yellow-400 text-2xl mb-1"><i class="fa-solid fa-crown"></i></div>
                                    <h3 class="font-bold text-slate-200 text-sm">Estelar & Ouro</h3>
                                    <p class="text-xs text-slate-500 mt-1">Estética requintada com galhos dourados, ideal para Ano Novo ou formaturas.</p>
                                </div>
                            </label>
                        </div>
                    </div>

                    <!-- Data de Revelação -->
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                        <div>
                            <label class="block text-xs font-bold uppercase tracking-wider text-slate-400 mb-2">Data e Hora de Abertura</label>
                            <input type="datetime-local" id="treeRevealDate" required
                                   class="w-full bg-slate-900/80 border border-slate-700 rounded-xl px-4 py-3 text-slate-100 focus:outline-none focus:border-yellow-500 focus:ring-1 focus:ring-yellow-500 transition-all">
                            <p class="text-[10px] text-slate-400 mt-1.5">Neste exato momento as mensagens se tornarão públicas para todos.</p>
                        </div>
                        <div>
                            <label class="block text-xs font-bold uppercase tracking-wider text-slate-400 mb-2">Código PIN do Criador (6 dígitos)</label>
                            <input type="password" id="treeAdminPin" pattern="[0-9]{6}" required maxlength="6" placeholder="Ex: 123456" 
                                   class="w-full bg-slate-900/80 border border-slate-700 rounded-xl px-4 py-3 text-slate-100 placeholder-slate-600 focus:outline-none focus:border-yellow-500 focus:ring-1 focus:ring-yellow-500 tracking-widest text-center transition-all">
                            <p class="text-[10px] text-slate-400 mt-1.5">Guarde este PIN para alterar o visual ou a data limite depois!</p>
                        </div>
                    </div>

                    <!-- Visibilidade Inicial dos Recados -->
                    <div>
                        <label class="block text-xs font-bold uppercase tracking-wider text-slate-400 mb-2">Visibilidade dos Recados</label>
                        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                            <!-- Opção Oculta -->
                            <label class="cursor-pointer group block">
                                <input type="radio" name="treeVisibility" value="locked" checked class="peer hidden">
                                <div class="bg-slate-900/60 border-2 border-slate-800 rounded-2xl p-4 transition-all group-hover:border-yellow-600 peer-checked:border-yellow-500 peer-checked:bg-yellow-950/20">
                                    <h3 class="font-bold text-slate-200 text-sm">🔒 Ocultar até a data limite</h3>
                                    <p class="text-xs text-slate-500 mt-1">Os segredos e carinhos ficam ocultos até o cronômetro zerar na data selecionada.</p>
                                </div>
                            </label>
                            <!-- Opção Visível -->
                            <label class="cursor-pointer group block">
                                <input type="radio" name="treeVisibility" value="unlocked" class="peer hidden">
                                <div class="bg-slate-900/60 border-2 border-slate-800 rounded-2xl p-4 transition-all group-hover:border-yellow-600 peer-checked:border-yellow-500 peer-checked:bg-yellow-950/20">
                                    <h3 class="font-bold text-slate-200 text-sm">🔓 Sempre Visíveis (Mural Aberto)</h3>
                                    <p class="text-xs text-slate-500 mt-1">Qualquer visitante pode abrir e ler todas as mensagens penduradas a qualquer hora.</p>
                                </div>
                            </label>
                        </div>
                    </div>

                    <button type="submit" class="w-full bg-gradient-to-r from-yellow-500 to-amber-600 hover:from-yellow-400 hover:to-amber-500 text-slate-950 font-bold py-4 px-6 rounded-xl transition-all shadow-xl hover:shadow-yellow-500/20 text-center flex items-center justify-center gap-2">
                        <i class="fa-solid fa-seedling text-lg"></i> Criar Minha Árvore Mágica
                    </button>
                </form>
            </div>

            <!-- Como Funciona Section -->
            <div class="bg-slate-900/40 border border-slate-800 rounded-2xl p-6 text-center">
                <h3 class="font-bold text-slate-200 mb-4">Como funciona a jornada?</h3>
                <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
                    <div class="flex flex-col items-center">
                        <div class="w-10 h-10 bg-yellow-500/10 border border-yellow-500/30 text-yellow-400 rounded-full flex items-center justify-center font-bold text-sm mb-2">1</div>
                        <h4 class="text-xs font-bold text-slate-300">Crie o seu link</h4>
                        <p class="text-[11px] text-slate-400 mt-1">Configure o visual de sua preferência e defina a data da grande revelação.</p>
                    </div>
                    <div class="flex flex-col items-center">
                        <div class="w-10 h-10 bg-yellow-500/10 border border-yellow-500/30 text-yellow-400 rounded-full flex items-center justify-center font-bold text-sm mb-2">2</div>
                        <h4 class="text-xs font-bold text-slate-300">Compartilhe o link</h4>
                        <p class="text-[11px] text-slate-400 mt-1">Envie para amigos no Instagram ou WhatsApp para que eles deixem desejos secretos.</p>
                    </div>
                    <div class="flex flex-col items-center">
                        <div class="w-10 h-10 bg-yellow-500/10 border border-yellow-500/30 text-yellow-400 rounded-full flex items-center justify-center font-bold text-sm mb-2">3</div>
                        <h4 class="text-xs font-bold text-slate-300">Vivencie a Revelação</h4>
                        <p class="text-[11px] text-slate-400 mt-1">No segundo exato do prazo limite, todos os enfeites da árvore podem ser abertos por qualquer um!</p>
                    </div>
                </div>
            </div>
        </div>

        <!-- 2. TELA DA ÁRVORE PRINCIPAL (Exibida apenas se houver o ID correto na URL) -->
        <div id="treeScreen" class="hidden space-y-6">
            
            <!-- Painel de Informações da Árvore e Temporizador -->
            <div id="treeCardHeader" class="bg-slate-900/90 backdrop-blur-md border border-slate-800 rounded-3xl p-6 flex flex-col md:flex-row items-center justify-between gap-6 shadow-2xl">
                <div class="text-center md:text-left space-y-1">
                    <span class="text-xs bg-slate-800 border border-slate-700 text-slate-300 px-3 py-1 rounded-full inline-block font-semibold">
                        Árvore de Recados de
                    </span>
                    <h2 id="displayTreeName" class="serif-title text-2xl md:text-3xl font-extrabold text-yellow-400">Carregando...</h2>
                    <p class="text-slate-400 text-xs">Clique na árvore para pendurar seus sentimentos em formato de enfeite.</p>
                </div>

                <!-- Painel do Cronômetro -->
                <div class="bg-slate-950/80 border border-yellow-500/20 px-6 py-4 rounded-2xl text-center glow-box-gold w-full md:w-auto">
                    <p id="countdownLabel" class="text-[10px] font-bold text-yellow-400 uppercase tracking-widest mb-1.5">
                        Revelando em:
                    </p>
                    <!-- Ticking countdown elements -->
                    <div id="countdownTimer" class="flex items-center justify-center gap-2 font-mono text-lg md:text-xl font-bold">
                        <div>
                            <span id="daysVal" class="bg-slate-900 border border-slate-800 px-2 py-1 rounded text-yellow-300">00</span>
                            <span class="text-[10px] text-slate-500 block font-sans mt-1">dias</span>
                        </div>
                        <span class="text-slate-500">:</span>
                        <div>
                            <span id="hoursVal" class="bg-slate-900 border border-slate-800 px-2 py-1 rounded text-yellow-300">00</span>
                            <span class="text-[10px] text-slate-500 block font-sans mt-1">h</span>
                        </div>
                        <span class="text-slate-500">:</span>
                        <div>
                            <span id="minsVal" class="bg-slate-900 border border-slate-800 px-2 py-1 rounded text-yellow-300">00</span>
                            <span class="text-[10px] text-slate-500 block font-sans mt-1">min</span>
                        </div>
                        <span class="text-slate-500">:</span>
                        <div>
                            <span id="secsVal" class="bg-slate-900 border border-slate-800 px-2 py-1 rounded text-yellow-300">00</span>
                            <span class="text-[10px] text-slate-500 block font-sans mt-1">seg</span>
                        </div>
                    </div>
                    <!-- Banner Aberto -->
                    <div id="revealedStatusBanner" class="hidden text-emerald-400 font-bold flex items-center justify-center gap-1.5 text-sm animate-bounce">
                        <i class="fa-solid fa-lock-open"></i> MENSAGENS REVELADAS!
                    </div>
                </div>
            </div>

            <!-- ÁREA DO RENDERIZADOR DA ÁRVORE INTERATIVA -->
            <div class="relative bg-slate-950/70 border border-slate-800 rounded-3xl p-4 md:p-8 shadow-2xl overflow-hidden flex flex-col items-center">
                
                <!-- Background ambient lights -->
                <div id="themeLightGlow" class="absolute w-72 h-72 rounded-full bg-emerald-500/10 blur-[80px] -z-10 top-1/4"></div>

                <!-- Feedback Visual de "Clique para posicionar" -->
                <div id="clickPlacementInstructions" class="bg-yellow-500/15 border border-yellow-500/30 rounded-xl px-4 py-2.5 text-center text-xs text-yellow-200 max-w-md mb-6 animate-pulse hidden">
                    🎯 <strong>Modo Posicionamento Ativo:</strong> Clique em qualquer galho da árvore abaixo para escolher onde pendurar seu enfeite!
                </div>

                <!-- O Conteiner da Árvore Física -->
                <div id="treeContainer" class="relative w-full max-w-[420px] aspect-[4/5] cursor-pointer bg-slate-900/20 rounded-2xl select-none">
                    
                    <!-- SVG Ilustrativo da Árvore baseado no Tema selecionado -->
                    <div id="treeSvgWrapper" class="w-full h-full flex items-center justify-center p-4">
                        <!-- Christmas Tree Base SVG -->
                        <svg id="svgChristmas" class="w-full h-full text-emerald-800" viewBox="0 0 100 120" fill="currentColor">
                            <!-- Tronco -->
                            <rect x="46" y="105" width="8" height="15" fill="#4a2c00" rx="1"/>
                            <!-- Vaso -->
                            <path d="M42 120 h16 l-3 -8 h-10 z" fill="#780000" />
                            <!-- Folhagem Base -->
                            <path d="M50 10 L15 105 h70 Z" fill="#064e3b" filter="drop-shadow(0 0 10px rgba(6,78,59,0.3))"/>
                            <!-- Folhas Detalhe 2 -->
                            <path d="M50 25 L20 95 h60 Z" fill="#047857"/>
                            <!-- Folhas Detalhe 3 -->
                            <path d="M50 40 L25 80 h50 Z" fill="#059669"/>
                            <!-- Folhas Detalhe 4 -->
                            <path d="M50 55 L32 65 h36 Z" fill="#10b981"/>
                            
                            <!-- Pisca-pisca Lines -->
                            <path d="M50 20 Q55 35 40 45 T65 65 T35 85 T60 100" fill="none" stroke="#fbbf24" stroke-width="0.5" stroke-dasharray="1 3" class="animate-pulse" />
                            
                            <!-- Estrela do Topo -->
                            <g fill="#f59e0b" filter="drop-shadow(0 0 8px #f59e0b)">
                                <path d="M50 2 L52 8 L58 10 L52 12 L50 18 L48 12 L42 10 L48 8 Z" />
                            </g>
                        </svg>

                        <!-- Enchanted Tree SVG -->
                        <svg id="svgEnchanted" class="w-full h-full text-purple-800 hidden" viewBox="0 0 100 120" fill="currentColor">
                            <!-- Tronco Místico -->
                            <path d="M44 105 Q47 115 42 120 h16 Q53 115 56 105 z" fill="#2d1b4e"/>
                            <!-- Folhagem Principal Espiral -->
                            <path d="M50 10 C15 70 20 105 50 105 C80 105 85 70 50 10" fill="#3b0764" filter="drop-shadow(0 0 15px rgba(59,7,100,0.4))"/>
                            <path d="M50 25 C25 70 30 95 50 95 C70 95 75 70 50 25" fill="#581c87"/>
                            <path d="M50 40 C35 70 38 85 50 85 C62 85 65 70 50 40" fill="#701a75"/>
                            <path d="M50 55 C42 70 45 78 50 78 C55 78 58 70 50 55" fill="#a21caf"/>
                            <!-- Pisca pisca azul -->
                            <path d="M50 15 C30 50 70 70 50 100" fill="none" stroke="#38bdf8" stroke-width="0.75" stroke-dasharray="1 4" class="animate-pulse" />
                            <!-- Estrela do Topo Encantada -->
                            <g fill="#c084fc" filter="drop-shadow(0 0 12px #c084fc)">
                                <circle cx="50" cy="10" r="4"/>
                                <path d="M50 2 L50 18 M42 10 L58 10" stroke="#c084fc" stroke-width="1"/>
                            </g>
                        </svg>

                        <!-- Golden / Estelar SVG -->
                        <svg id="svgGolden" class="w-full h-full text-amber-500 hidden" viewBox="0 0 100 120" fill="currentColor">
                            <!-- Tronco Dourado -->
                            <rect x="47" y="100" width="6" height="20" fill="#78350f" rx="1"/>
                            <!-- Cone Dourado de Luxo -->
                            <path d="M50 12 L20 100 h60 Z" fill="#78350f" />
                            <path d="M50 15 L22 97 h56 Z" fill="#b45309" filter="drop-shadow(0 0 10px rgba(180,83,9,0.4))" />
                            <path d="M50 25 L28 92 h44 Z" fill="#d97706" />
                            <path d="M50 40 L34 85 h32 Z" fill="#f59e0b" />
                            <path d="M50 55 L40 75 h20 Z" fill="#fbbf24" />
                            <!-- Pisca pisca cintilante -->
                            <path d="M50 20 L25 90 M50 20 L75 90 M50 40 L35 90 M50 40 L65 90" fill="none" stroke="#ffffff" stroke-width="0.5" stroke-dasharray="2 2" />
                            <!-- Grande Estrela Estelar -->
                            <g fill="#fffbeb" filter="drop-shadow(0 0 15px #fef08a)">
                                <polygon points="50,2 53,10 61,10 54,15 57,23 50,18 43,23 46,15 39,10 47,10" />
                            </g>
                        </svg>
                    </div>

                    <!-- Div de Ornatos Colocados via JS -->
                    <div id="ornamentsWrapper" class="absolute inset-0"></div>
                </div>

                <!-- Botões de Controle e Ações sob a Árvore -->
                <div class="mt-8 flex flex-wrap gap-4 items-center justify-center w-full max-w-lg">
                    <button id="btnPrepareOrnament" class="flex-1 min-w-[150px] bg-gradient-to-r from-emerald-500 to-green-600 hover:from-emerald-400 hover:to-green-500 text-slate-950 font-bold py-3.5 px-4 rounded-xl shadow-lg hover:shadow-green-500/20 transition-all flex items-center justify-center gap-2">
                        <i class="fa-solid fa-gift"></i> Pendurar Enfeite
                    </button>
                    <button id="btnShareTree" class="flex-1 min-w-[150px] bg-slate-800 hover:bg-slate-700 text-slate-100 font-bold py-3.5 px-4 rounded-xl border border-slate-700 transition-all flex items-center justify-center gap-2">
                        <i class="fa-solid fa-share-nodes"></i> Compartilhar Link
                    </button>
                    <button id="btnOpenAdminPanel" class="bg-slate-900 hover:bg-slate-800 text-slate-400 hover:text-yellow-400 p-3.5 rounded-xl border border-slate-800 transition-all" title="Gerenciar Árvore">
                        <i class="fa-solid fa-gear"></i>
                    </button>
                </div>

                <!-- Link de Voltar ou Criar Nova -->
                <div class="mt-4 text-center">
                    <a href="?" class="text-xs text-slate-500 hover:text-yellow-500 transition-all underline">Gostou da ideia? Clique aqui para criar a sua própria árvore!</a>
                </div>
            </div>
        </div>

    </div>

    <!-- ==================== MODALS ==================== -->

    <!-- MODAL 1: CRIAR ENFEITE (PENDURAR) -->
    <div id="modalOrnament" class="fixed inset-0 bg-slate-950/80 backdrop-blur-md items-center justify-center p-4 z-50 hidden">
        <div class="bg-slate-900 border border-slate-800 rounded-3xl w-full max-w-md p-6 relative shadow-2xl space-y-5 animate-in fade-in zoom-in-95 duration-200">
            <button class="absolute top-4 right-4 text-slate-500 hover:text-slate-200 text-xl btnCloseModal">
                <i class="fa-solid fa-xmark"></i>
            </button>
            
            <div class="text-center">
                <h3 class="serif-title text-xl font-bold text-yellow-400">Escreva sua Cartinha</h3>
                <p class="text-xs text-slate-400 mt-1">Sua mensagem ficará selada até a data limite programada.</p>
            </div>

            <!-- Formulário -->
            <form id="ornamentForm" class="space-y-4">
                <!-- Seletor de Enfeite Visual -->
                <div>
                    <label class="block text-[11px] font-bold uppercase tracking-wider text-slate-400 mb-2">1. Escolha seu Enfeite</label>
                    <div class="grid grid-cols-5 gap-2">
                        <!-- Bola -->
                        <label class="cursor-pointer text-center group">
                            <input type="radio" name="ornamentDesign" value="ball" checked class="peer hidden">
                            <div class="bg-slate-950 border border-slate-800 rounded-xl p-3 transition-all group-hover:border-red-500 peer-checked:border-red-500 peer-checked:bg-red-500/10 text-red-500">
                                <i class="fa-solid fa-circle text-2xl"></i>
                            </div>
                            <span class="text-[9px] text-slate-500 mt-1 block">Bola</span>
                        </label>
                        <!-- Estrela -->
                        <label class="cursor-pointer text-center group">
                            <input type="radio" name="ornamentDesign" value="star" class="peer hidden">
                            <div class="bg-slate-950 border border-slate-800 rounded-xl p-3 transition-all group-hover:border-yellow-500 peer-checked:border-yellow-500 peer-checked:bg-yellow-500/10 text-yellow-500">
                                <i class="fa-solid fa-star text-2xl"></i>
                            </div>
                            <span class="text-[9px] text-slate-500 mt-1 block">Estrela</span>
                        </label>
                        <!-- Sino -->
                        <label class="cursor-pointer text-center group">
                            <input type="radio" name="ornamentDesign" value="bell" class="peer hidden">
                            <div class="bg-slate-950 border border-slate-800 rounded-xl p-3 transition-all group-hover:border-amber-500 peer-checked:border-amber-500 peer-checked:bg-amber-500/10 text-amber-400">
                                <i class="fa-solid fa-bell text-2xl"></i>
                            </div>
                            <span class="text-[9px] text-slate-500 mt-1 block">Sino</span>
                        </label>
                        <!-- Presente -->
                        <label class="cursor-pointer text-center group">
                            <input type="radio" name="ornamentDesign" value="gift" class="peer hidden">
                            <div class="bg-slate-950 border border-slate-800 rounded-xl p-3 transition-all group-hover:border-blue-500 peer-checked:border-blue-500 peer-checked:bg-blue-500/10 text-blue-400">
                                <i class="fa-solid fa-gift text-2xl"></i>
                            </div>
                            <span class="text-[9px] text-slate-500 mt-1 block">Presente</span>
                        </label>
                        <!-- Meia -->
                        <label class="cursor-pointer text-center group">
                            <input type="radio" name="ornamentDesign" value="sock" class="peer hidden">
                            <div class="bg-slate-950 border border-slate-800 rounded-xl p-3 transition-all group-hover:border-emerald-500 peer-checked:border-emerald-500 peer-checked:bg-emerald-500/10 text-emerald-400">
                                <i class="fa-solid fa-socks text-2xl"></i>
                            </div>
                            <span class="text-[9px] text-slate-500 mt-1 block">Meia</span>
                        </label>
                    </div>
                </div>

                <!-- Quem Envia -->
                <div>
                    <label class="block text-[11px] font-bold uppercase tracking-wider text-slate-400 mb-1.5">2. Seu Nome (Ou apelido)</label>
                    <input type="text" id="ornamentSender" required placeholder="Ex: Maria Clara, Seu Admirador, Anônimo" 
                           class="w-full bg-slate-950 border border-slate-800 rounded-xl px-4 py-3 text-slate-100 placeholder-slate-600 focus:outline-none focus:border-yellow-500 transition-all text-sm">
                </div>

                <!-- Recado Secreto -->
                <div>
                    <label class="block text-[11px] font-bold uppercase tracking-wider text-slate-400 mb-1.5">3. Mensagem Secreta</label>
                    <textarea id="ornamentMessage" required rows="4" maxlength="300" placeholder="Escreva aqui seu recado, votos de felicidade ou carinho..." 
                              class="w-full bg-slate-950 border border-slate-800 rounded-xl px-4 py-3 text-slate-100 placeholder-slate-600 focus:outline-none focus:border-yellow-500 transition-all text-sm resize-none"></textarea>
                    <p class="text-[10px] text-slate-500 text-right mt-1">Máximo 300 caracteres.</p>
                </div>

                <div class="bg-slate-950/60 p-3 rounded-xl border border-slate-800 text-[11px] text-slate-400 leading-relaxed">
                    🌟 <strong>Após preencher:</strong> Você será instruído a clicar no local exato da árvore onde quer colocar seu enfeite.
                </div>

                <button type="submit" class="w-full bg-gradient-to-r from-emerald-500 to-green-600 hover:from-emerald-400 hover:to-green-500 text-slate-950 font-bold py-3 px-4 rounded-xl transition-all shadow-lg text-center flex items-center justify-center gap-2">
                    Continuar e Escolher Local <i class="fa-solid fa-arrow-right"></i>
                </button>
            </form>
        </div>
    </div>

    <!-- MODAL 2: VER CONTEÚDO DO ENFEITE (MENSAGEM) -->
    <div id="modalReadMessage" class="fixed inset-0 bg-slate-950/80 backdrop-blur-md items-center justify-center p-4 z-50 hidden">
        <div class="bg-slate-900 border border-slate-800 rounded-3xl w-full max-w-md p-6 relative shadow-2xl space-y-6 animate-in fade-in zoom-in-95 duration-200 text-center">
            <button class="absolute top-4 right-4 text-slate-500 hover:text-slate-200 text-xl btnCloseModal">
                <i class="fa-solid fa-xmark"></i>
            </button>

            <!-- Ícone do Enfeite -->
            <div id="readOrnamentIconWrapper" class="w-20 h-20 mx-auto rounded-full flex items-center justify-center text-4xl mb-2">
                <i class="fa-solid fa-circle"></i>
            </div>

            <!-- Status do Bloqueio -->
            <div id="readLockedContent" class="space-y-4">
                <div class="inline-flex items-center gap-1.5 bg-red-500/10 border border-red-500/20 text-red-400 px-3 py-1 rounded-full text-xs font-semibold">
                    <i class="fa-solid fa-lock"></i> MENSAGEM TRANCADA
                </div>
                <h3 class="serif-title text-xl font-extrabold text-slate-200">De: <span id="readLockedSender">Alguém Especial</span></h3>
                <p class="text-slate-400 text-sm leading-relaxed px-2">
                    Esta mensagem secreta foi pendurada com carinho, mas ela só poderá ser revelada quando o cronômetro da árvore for zerado!
                </p>
                <div class="bg-slate-950 p-4 rounded-2xl border border-slate-800">
                    <p class="text-[10px] text-yellow-400 uppercase tracking-widest font-bold mb-1">Revelação programada:</p>
                    <p id="readLockedTargetDate" class="font-mono text-slate-200 text-sm font-semibold">Carregando...</p>
                </div>
            </div>

            <!-- Status Aberto (Conteúdo Revelado) -->
            <div id="readUnlockedContent" class="space-y-4 hidden">
                <div class="inline-flex items-center gap-1.5 bg-emerald-500/10 border border-emerald-500/20 text-emerald-400 px-3 py-1 rounded-full text-xs font-semibold animate-pulse">
                    <i class="fa-solid fa-lock-open"></i> ENFEITE REVELADO
                </div>
                <div class="space-y-1">
                    <p class="text-xs text-slate-500">Uma mensagem especial de</p>
                    <h3 class="serif-title text-2xl font-extrabold text-yellow-400" id="readUnlockedSender">Anônimo</h3>
                </div>
                
                <div class="bg-slate-950/80 border border-slate-800 p-6 rounded-2xl relative">
                    <div class="text-slate-400 text-2xl absolute top-2 left-3 opacity-30"><i class="fa-solid fa-quote-left"></i></div>
                    <p id="readUnlockedMessage" class="handwritten text-lg text-slate-100 leading-relaxed py-2 break-words">
                        Mensagem secreta...
                    </p>
                    <div class="text-slate-400 text-2xl absolute bottom-2 right-3 opacity-30"><i class="fa-solid fa-quote-right"></i></div>
                </div>
                
                <p id="readUnlockedDate" class="text-[10px] text-slate-600">Enviado em...</p>
            </div>
            
            <button class="w-full bg-slate-800 hover:bg-slate-700 text-slate-300 font-bold py-3 px-4 rounded-xl transition-all btnCloseModal">
                Fechar Janela
            </button>
        </div>
    </div>

    <!-- MODAL 3: PAINEL DE CONTROLE DO CRIADOR (ADMIN) -->
    <div id="modalAdmin" class="fixed inset-0 bg-slate-950/80 backdrop-blur-md items-center justify-center p-4 z-50 hidden">
        <div class="bg-slate-900 border border-slate-800 rounded-3xl w-full max-w-md p-6 relative shadow-2xl space-y-5 animate-in fade-in zoom-in-95 duration-200">
            <button class="absolute top-4 right-4 text-slate-500 hover:text-slate-200 text-xl btnCloseModal">
                <i class="fa-solid fa-xmark"></i>
            </button>

            <div class="text-center">
                <h3 class="serif-title text-xl font-bold text-yellow-400">Painel de Controle</h3>
                <p class="text-xs text-slate-400 mt-1">Gerencie as regras e o visual de sua árvore.</p>
            </div>

            <!-- PIN Login State -->
            <div id="adminPinState" class="space-y-4">
                <p class="text-xs text-slate-300 text-center">Digite o PIN de 6 dígitos que você cadastrou ao criar esta árvore:</p>
                <div class="space-y-3">
                    <input type="password" id="adminPinInput" pattern="[0-9]{6}" maxlength="6" placeholder="******" 
                           class="w-full bg-slate-950 border border-slate-800 rounded-xl px-4 py-3 text-center tracking-widest text-lg font-bold text-slate-100 focus:outline-none focus:border-yellow-500 transition-all">
                    <button id="btnVerifyAdminPin" class="w-full bg-yellow-500 hover:bg-yellow-400 text-slate-950 font-bold py-3 px-4 rounded-xl transition-all shadow-lg text-center">
                        Verificar PIN de Acesso
                    </button>
                </div>
            </div>

            <!-- Admin Logged In State (Gerenciador) -->
            <div id="adminDashboardState" class="space-y-5 hidden">
                <div class="bg-slate-950/60 p-4 rounded-2xl border border-slate-800 text-center">
                    <p class="text-xs text-emerald-400 font-bold flex items-center justify-center gap-1"><i class="fa-solid fa-circle-check"></i> Identidade Confirmada</p>
                    <p class="text-slate-400 text-[10px] mt-1">Você pode alterar as regras e o design livremente abaixo.</p>
                </div>

                <!-- Formulário de Alteração da Data -->
                <div class="space-y-2">
                    <label class="block text-[11px] font-bold uppercase tracking-wider text-slate-400">Nova Data e Hora Limite para Revelação</label>
                    <input type="datetime-local" id="adminNewDateInput" required
                           class="w-full bg-slate-950 border border-slate-800 rounded-xl px-4 py-3 text-slate-100 focus:outline-none focus:border-yellow-500 transition-all text-sm">
                    <p class="text-[10px] text-slate-500">Se você alterar para uma data no passado, todas as mensagens serão reveladas instantaneamente.</p>
                </div>

                <!-- Formulário de Alteração do Tema Visual (Design) -->
                <div class="space-y-2">
                    <label class="block text-[11px] font-bold uppercase tracking-wider text-slate-400">Alterar Tema / Design Visual</label>
                    <div class="grid grid-cols-3 gap-2">
                        <!-- Clássico -->
                        <label class="cursor-pointer group relative block">
                            <input type="radio" name="adminTreeTheme" value="christmas" class="peer hidden">
                            <div class="bg-slate-950 border border-slate-800 rounded-xl p-3 text-center transition-all hover:border-emerald-600 peer-checked:border-emerald-500 peer-checked:bg-emerald-950/20">
                                <span class="text-emerald-400 block text-lg mb-1"><i class="fa-solid fa-holly-berry"></i></span>
                                <h4 class="font-bold text-slate-300 text-[10px]">Clássico</h4>
                            </div>
                        </label>
                        <!-- Encantada -->
                        <label class="cursor-pointer group relative block">
                            <input type="radio" name="adminTreeTheme" value="enchanted" class="peer hidden">
                            <div class="bg-slate-950 border border-slate-800 rounded-xl p-3 text-center transition-all hover:border-purple-600 peer-checked:border-purple-500 peer-checked:bg-purple-950/20">
                                <span class="text-purple-400 block text-lg mb-1"><i class="fa-solid fa-wand-magic-sparkles"></i></span>
                                <h4 class="font-bold text-slate-300 text-[10px]">Místico</h4>
                            </div>
                        </label>
                        <!-- Estelar -->
                        <label class="cursor-pointer group relative block">
                            <input type="radio" name="adminTreeTheme" value="golden" class="peer hidden">
                            <div class="bg-slate-950 border border-slate-800 rounded-xl p-3 text-center transition-all hover:border-yellow-600 peer-checked:border-yellow-500 peer-checked:bg-yellow-950/20">
                                <span class="text-yellow-400 block text-lg mb-1"><i class="fa-solid fa-crown"></i></span>
                                <h4 class="font-bold text-slate-300 text-[10px]">Estelar</h4>
                            </div>
                        </label>
                    </div>
                </div>

                <!-- Alteração de Privacidade/Visibilidade -->
                <div class="space-y-2">
                    <label class="block text-[11px] font-bold uppercase tracking-wider text-slate-400">Visibilidade dos Recados</label>
                    <div class="grid grid-cols-2 gap-3">
                        <label class="cursor-pointer group block">
                            <input type="radio" name="adminTreeVisibility" value="locked" class="peer hidden">
                            <div class="bg-slate-950 border border-slate-800 rounded-xl p-3 text-center transition-all hover:border-yellow-600 peer-checked:border-yellow-500 peer-checked:bg-yellow-950/20 text-xs font-semibold">
                                🔒 Ocultar até a data
                            </div>
                        </label>
                        <label class="cursor-pointer group block">
                            <input type="radio" name="adminTreeVisibility" value="unlocked" class="peer hidden">
                            <div class="bg-slate-950 border border-slate-800 rounded-xl p-3 text-center transition-all hover:border-yellow-600 peer-checked:border-yellow-500 peer-checked:bg-yellow-950/20 text-xs font-semibold">
                                🔓 Sempre Visíveis
                            </div>
                        </label>
                    </div>
                </div>

                <button id="btnSaveAdminSettings" class="w-full bg-gradient-to-r from-yellow-500 to-amber-600 hover:from-yellow-400 hover:to-amber-500 text-slate-950 font-bold py-3.5 px-4 rounded-xl transition-all shadow-lg text-center flex items-center justify-center gap-2">
                    <i class="fa-solid fa-floppy-disk"></i> Salvar Configurações
                </button>
            </div>
        </div>
    </div>

    <!-- TOAST (NOTIFICAÇÃO RÁPIDA CUSTOMIZADA) -->
    <div id="toastNotification" class="fixed bottom-6 left-1/2 -translate-x-1/2 bg-slate-900 border border-yellow-500/30 text-slate-100 px-6 py-3.5 rounded-2xl shadow-2xl z-50 flex items-center gap-2.5 max-w-sm w-11/12 text-sm justify-between transition-all duration-300 opacity-0 translate-y-10 pointer-events-none">
        <div class="flex items-center gap-2">
            <span id="toastIcon" class="text-yellow-400 text-base"><i class="fa-solid fa-bell"></i></span>
            <span id="toastMsg" class="font-medium text-slate-200 font-semibold">Notificação...</span>
        </div>
        <button onclick="hideToast()" class="text-slate-500 hover:text-slate-300 text-base"><i class="fa-solid fa-xmark"></i></button>
    </div>

    <!-- ==================== SCRIPTS E LOGICA ==================== -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInWithCustomToken, signInAnonymously, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, setDoc, getDoc, collection, query, onSnapshot, addDoc, updateDoc } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        const firebaseConfig = {
            apiKey: "AIzaSyA7sjqWxLJR1EwqrZKE4RvgINAtfWv0m9U",
            authDomain: "arvore-recados.firebaseapp.com",
            projectId: "arvore-recados",
            storageBucket: "arvore-recados.firebasestorage.app",
            messagingSenderId: "675694126478",
            appId: "1:675694126478:web:2f20635983ad791764f0df"
        };

        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);
        const appId = 'arvore-recados';

        // State variables
        let currentUser = null;
        let currentTreeId = null;
        let currentTreeData = null;
        let ornamentsList = [];
        let isPlacementActive = false;

        // Dom Elements
        const loadingScreen = document.getElementById('loadingScreen');
        const homeScreen = document.getElementById('homeScreen');
        const treeScreen = document.getElementById('treeScreen');
        
        // Form & input elements
        const createTreeForm = document.getElementById('createTreeForm');
        const treeOwnerName = document.getElementById('treeOwnerName');
        const treeRevealDate = document.getElementById('treeRevealDate');
        const treeAdminPin = document.getElementById('treeAdminPin');
        
        // Modals
        const modalOrnament = document.getElementById('modalOrnament');
        const modalReadMessage = document.getElementById('modalReadMessage');
        const modalAdmin = document.getElementById('modalAdmin');
        const toastNotification = document.getElementById('toastNotification');

        // Setup base paths (STRICT RULE 1)
        const getTreesCollection = () => collection(db, 'artifacts', appId, 'public', 'data', 'trees');
        const getOrnamentsCollection = () => collection(db, 'artifacts', appId, 'public', 'data', 'ornaments');

        // ==================== AUTH & ROUTING ====================
        
        window.onload = async () => {
            // Start Snow Animation
            initSnow();

            // Authentication Step with fallback (STRICT RULE 3)
            try {
                if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                    try {
                        await signInWithCustomToken(auth, __initial_auth_token);
                    } catch (tokenError) {
                        console.warn("Token customizado falhou, usando autenticação anônima:", tokenError);
                        await signInAnonymously(auth);
                    }
                } else {
                    await signInAnonymously(auth);
                }
            } catch (error) {
                console.error("Auth Error:", error);
                showToast("Erro ao conectar à rede mágica. Tente atualizar a página.", "fa-solid fa-triangle-exclamation");
            }

            // Track Auth state change
            onAuthStateChanged(auth, (user) => {
                if (user) {
                    currentUser = user;
                    routeView();
                }
            });
        };

        // Determine which screen/tree is loaded based on query params
        function routeView() {
            const params = new URLSearchParams(window.location.search);
            const treeId = params.get('tree');

            if (treeId && treeId.trim() !== "") {
                currentTreeId = treeId;
                loadTree(treeId);
            } else {
                // EXCLUSIVO: Se o usuário não tem o link no parâmetro, exibe apenas a tela de CRIAÇÃO
                loadingScreen.classList.add('hidden');
                homeScreen.classList.remove('hidden');
                treeScreen.classList.add('hidden');
            }
        }

        // ==================== FIREBASE DATA FETCH ====================

        async function loadTree(treeId) {
            if (!currentUser) return;
            
            try {
                // Fetch Tree Header Details (STRICT RULE 2 - Keep it simple)
                const treeDocRef = doc(db, 'artifacts', appId, 'public', 'data', 'trees', treeId);
                const treeDocSnap = await getDoc(treeDocRef);

                if (!treeDocSnap.exists()) {
                    showToast("Árvore não encontrada! Redirecionando para criação...", "fa-solid fa-triangle-exclamation");
                    setTimeout(() => {
                        window.location.href = window.location.pathname; 
                    }, 2500);
                    return;
                }

                currentTreeData = treeDocSnap.data();
                renderTreeHeaderAndTheme(currentTreeData);

                // Setup realtime listener for ornaments of this specific tree
                const ornamentsRef = getOrnamentsCollection();
                
                onSnapshot(ornamentsRef, (snapshot) => {
                    const allOrnaments = [];
                    snapshot.forEach(doc => {
                        allOrnaments.push({ id: doc.id, ...doc.data() });
                    });

                    ornamentsList = allOrnaments.filter(o => o.treeId === currentTreeId);
                    renderOrnamentsOnTree(ornamentsList);
                }, (error) => {
                    console.error("Error fetching ornaments:", error);
                });

                // Listen in real-time to tree configurations (including real-time theme / date changes!)
                onSnapshot(treeDocRef, (snap) => {
                    if (snap.exists()) {
                        currentTreeData = snap.data();
                        renderTreeHeaderAndTheme(currentTreeData);
                        updateCountdown();
                    }
                });

                // Start Live Countdown Thread
                setInterval(updateCountdown, 1000);

                // Switch UI Screens
                loadingScreen.classList.add('hidden');
                homeScreen.classList.add('hidden');
                treeScreen.classList.remove('hidden');

            } catch (err) {
                console.error("Error loading tree:", err);
                showToast("Falha técnica ao carregar árvore.", "fa-solid fa-circle-exclamation");
            }
        }

        // ==================== TREE CREATION ====================

        createTreeForm.addEventListener('submit', async (e) => {
            e.preventDefault();
            if (!currentUser) return;

            const name = treeOwnerName.value.trim();
            const revealStr = treeRevealDate.value;
            const pin = treeAdminPin.value;
            const theme = document.querySelector('input[name="treeTheme"]:checked').value;
            const revealMode = document.querySelector('input[name="treeVisibility"]:checked').value;

            if (!name || !revealStr || !pin || pin.length !== 6) {
                showToast("Preencha todos os campos corretamente (PIN com 6 dígitos).", "fa-solid fa-circle-exclamation");
                return;
            }

            // Create unique alphanumeric ID
            const newTreeId = Math.random().toString(36).substring(2, 10);
            const revealDateMs = new Date(revealStr).getTime();

            const newTreeData = {
                id: newTreeId,
                name: name,
                creatorId: currentUser.uid,
                adminPin: pin,
                revealDate: revealDateMs,
                theme: theme,
                revealMode: revealMode,
                createdAt: Date.now()
            };

            try {
                const treeDocRef = doc(db, 'artifacts', appId, 'public', 'data', 'trees', newTreeId);
                await setDoc(treeDocRef, newTreeData);

                showToast("Sua Árvore Mágica foi gerada com sucesso!", "fa-solid fa-wand-magic-sparkles");
                
                setTimeout(() => {
                    window.location.search = `?tree=${newTreeId}`;
                }, 1500);

            } catch (err) {
                console.error("Write error:", err);
                showToast("Erro ao salvar árvore no banco de dados.", "fa-solid fa-triangle-exclamation");
            }
        });

        // ==================== RENDERING & THEMES ====================

        function renderTreeHeaderAndTheme(tree) {
            document.getElementById('displayTreeName').innerText = `Árvore de ${tree.name}`;
            
            // Hide all SVG elements
            document.getElementById('svgChristmas').classList.add('hidden');
            document.getElementById('svgEnchanted').classList.add('hidden');
            document.getElementById('svgGolden').classList.add('hidden');

            const glow = document.getElementById('themeLightGlow');

            if (tree.theme === 'christmas') {
                document.getElementById('svgChristmas').classList.remove('hidden');
                glow.className = "absolute w-72 h-72 rounded-full bg-emerald-500/15 blur-[90px] -z-10 top-1/4";
            } else if (tree.theme === 'enchanted') {
                document.getElementById('svgEnchanted').classList.remove('hidden');
                glow.className = "absolute w-72 h-72 rounded-full bg-purple-500/15 blur-[90px] -z-10 top-1/4";
            } else if (tree.theme === 'golden') {
                document.getElementById('svgGolden').classList.remove('hidden');
                glow.className = "absolute w-72 h-72 rounded-full bg-amber-500/15 blur-[90px] -z-10 top-1/4";
            }
        }

        // Renders visual ornaments absolutely inside the tree wrapper based on percentages
        function renderOrnamentsOnTree(ornaments) {
            const wrapper = document.getElementById('ornamentsWrapper');
            wrapper.innerHTML = '';

            ornaments.forEach(item => {
                const button = document.createElement('button');
                button.className = "absolute ornament-placed flex items-center justify-center rounded-full transition-all focus:outline-none";
                button.style.left = `${item.position.x}%`;
                button.style.top = `${item.position.y}%`;
                button.style.transform = 'translate(-50%, -50%)';
                button.style.width = '32px';
                button.style.height = '32px';
                button.style.zIndex = '30';

                let iconClass = 'fa-solid ';
                let bgGradient = '';
                let borderTheme = '';

                switch (item.ornamentType) {
                    case 'ball':
                        iconClass += 'fa-circle';
                        bgGradient = 'from-red-500 to-rose-700';
                        borderTheme = 'border-red-400';
                        break;
                    case 'star':
                        iconClass += 'fa-star text-[10px]';
                        bgGradient = 'from-yellow-400 to-amber-600 text-yellow-950';
                        borderTheme = 'border-yellow-200';
                        break;
                    case 'bell':
                        iconClass += 'fa-bell text-xs';
                        bgGradient = 'from-amber-400 to-orange-500';
                        borderTheme = 'border-amber-300';
                        break;
                    case 'gift':
                        iconClass += 'fa-gift text-xs';
                        bgGradient = 'from-blue-500 to-sky-600';
                        borderTheme = 'border-blue-300';
                        break;
                    case 'sock':
                        iconClass += 'fa-socks text-xs';
                        bgGradient = 'from-emerald-500 to-green-600';
                        borderTheme = 'border-emerald-300';
                        break;
                    default:
                        iconClass += 'fa-heart';
                        bgGradient = 'from-pink-500 to-rose-600';
                        borderTheme = 'border-pink-300';
                }

                button.innerHTML = `<span class="bg-gradient-to-b ${bgGradient} border ${borderTheme} rounded-full w-7 h-7 flex items-center justify-center text-[10px] text-white shadow-md"><i class="${iconClass}"></i></span>`;
                
                button.addEventListener('click', (e) => {
                    e.stopPropagation(); 
                    openOrnamentMessage(item);
                });

                wrapper.appendChild(button);
            });
        }

        // ==================== TIMER LOGIC ====================

        function isRevealed() {
            if (!currentTreeData) return false;
            // Se o modo de visibilidade for definido como sempre visível, ignora o prazo limite
            if (currentTreeData.revealMode === 'unlocked') {
                return true;
            }
            return Date.now() >= currentTreeData.revealDate;
        }

        function updateCountdown() {
            if (!currentTreeData) return;

            const now = Date.now();
            const difference = currentTreeData.revealDate - now;

            const countdownLabel = document.getElementById('countdownLabel');
            const countdownTimer = document.getElementById('countdownTimer');
            const revealedStatusBanner = document.getElementById('revealedStatusBanner');

            if (isRevealed()) {
                countdownLabel.classList.add('hidden');
                countdownTimer.classList.add('hidden');
                revealedStatusBanner.classList.remove('hidden');
                
                // Exibe label diferente caso seja um mural permanentemente aberto
                if (currentTreeData.revealMode === 'unlocked') {
                    revealedStatusBanner.innerHTML = '<i class="fa-solid fa-lock-open"></i> MURAL DE RECADOS ABERTO!';
                } else {
                    revealedStatusBanner.innerHTML = '<i class="fa-solid fa-lock-open"></i> MENSAGENS REVELADAS!';
                }
            } else {
                countdownLabel.classList.remove('hidden');
                countdownTimer.classList.remove('hidden');
                revealedStatusBanner.classList.add('hidden');

                const d = Math.floor(difference / (1000 * 60 * 60 * 24));
                const h = Math.floor((difference % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60));
                const m = Math.floor((difference % (1000 * 60 * 60)) / (1000 * 60));
                const s = Math.floor((difference % (1000 * 60)) / 1000);

                document.getElementById('daysVal').innerText = String(d).padStart(2, '0');
                document.getElementById('hoursVal').innerText = String(h).padStart(2, '0');
                document.getElementById('minsVal').innerText = String(m).padStart(2, '0');
                document.getElementById('secsVal').innerText = String(s).padStart(2, '0');
            }
        }

        // ==================== INTERACTION: PLACE ORNAMENT ====================

        document.getElementById('btnPrepareOrnament').addEventListener('click', () => {
            modalOrnament.classList.remove('hidden');
            modalOrnament.classList.add('flex');
        });

        document.getElementById('ornamentForm').addEventListener('submit', (e) => {
            e.preventDefault();
            
            const sender = document.getElementById('ornamentSender').value.trim();
            const text = document.getElementById('ornamentMessage').value.trim();

            if (!sender || !text) {
                showToast("Preencha todos os campos do enfeite.", "fa-solid fa-triangle-exclamation");
                return;
            }

            closeAllModals();

            isPlacementActive = true;
            document.getElementById('clickPlacementInstructions').classList.remove('hidden');
            
            const treeContainer = document.getElementById('treeContainer');
            treeContainer.classList.add('ring-4', 'ring-yellow-500/50');
            
            showToast("Agora, toque/clique no local desejado na árvore!", "fa-solid fa-arrow-pointer");
        });

        // Click handler on actual Tree physical canvas bounds
        document.getElementById('treeContainer').addEventListener('click', async (e) => {
            if (!isPlacementActive) return;

            // Get exact bounding coordinates of clicking point inside parent tree
            const rect = e.currentTarget.getBoundingClientRect();
            const xVal = ((e.clientX - rect.left) / rect.width) * 100;
            const yVal = ((e.clientY - rect.top) / rect.height) * 100;

            // Simple safety constraints so coordinates don't map outside visible zones
            const boundedX = Math.min(Math.max(xVal, 5), 95);
            const boundedY = Math.min(Math.max(yVal, 10), 90);

            // Complete save workflow
            isPlacementActive = false;
            
            // Cleanup UI indicators
            document.getElementById('clickPlacementInstructions').classList.add('hidden');
            document.getElementById('treeContainer').classList.remove('ring-4', 'ring-yellow-500/50');

            const design = document.querySelector('input[name="ornamentDesign"]:checked').value;
            const sender = document.getElementById('ornamentSender').value.trim();
            const text = document.getElementById('ornamentMessage').value.trim();

            const newOrnamentObj = {
                treeId: currentTreeId,
                ornamentType: design,
                senderName: sender,
                message: text,
                position: { x: parseFloat(boundedX.toFixed(2)), y: parseFloat(boundedY.toFixed(2)) },
                createdAt: Date.now()
            };

            try {
                // Save directly to public collections with exact path constraints (STRICT RULE 1)
                await addDoc(getOrnamentsCollection(), newOrnamentObj);
                
                showToast("Seu enfeite foi pendurado com carinho!", "fa-solid fa-circle-check");
                
                // Clear the form fields for next time
                document.getElementById('ornamentForm').reset();

            } catch (err) {
                console.error("Save Ornament Error:", err);
                showToast("Erro ao gravar enfeite. Tente novamente.", "fa-solid fa-triangle-exclamation");
            }
        });

        // ==================== INTERACTION: READ ORNAMENT ====================

        function openOrnamentMessage(item) {
            const iconWrapper = document.getElementById('readOrnamentIconWrapper');
            const lockedState = document.getElementById('readLockedContent');
            const unlockedState = document.getElementById('readUnlockedContent');

            let iconMarkup = '';
            let bgGlow = '';

            switch (item.ornamentType) {
                case 'ball':
                    iconMarkup = '<i class="fa-solid fa-circle text-red-500"></i>';
                    bgGlow = 'bg-red-500/10 border border-red-500/20';
                    break;
                case 'star':
                    iconMarkup = '<i class="fa-solid fa-star text-yellow-400"></i>';
                    bgGlow = 'bg-yellow-500/10 border border-yellow-500/20';
                    break;
                case 'bell':
                    iconMarkup = '<i class="fa-solid fa-bell text-amber-400"></i>';
                    bgGlow = 'bg-amber-500/10 border border-amber-500/20';
                    break;
                case 'gift':
                    iconMarkup = '<i class="fa-solid fa-gift text-blue-400"></i>';
                    bgGlow = 'bg-blue-500/10 border border-blue-500/20';
                    break;
                case 'sock':
                    iconMarkup = '<i class="fa-solid fa-socks text-emerald-400"></i>';
                    bgGlow = 'bg-emerald-500/10 border border-emerald-500/20';
                    break;
            }

            iconWrapper.className = `w-20 h-20 mx-auto rounded-full flex items-center justify-center text-4xl mb-2 ${bgGlow}`;
            iconWrapper.innerHTML = iconMarkup;

            if (isRevealed()) {
                lockedState.classList.add('hidden');
                unlockedState.classList.remove('hidden');

                document.getElementById('readUnlockedSender').innerText = item.senderName;
                document.getElementById('readUnlockedMessage').innerText = item.message;
                
                const dateObj = new Date(item.createdAt);
                const formatOpts = { day: '2-digit', month: '2-digit', hour: '2-digit', minute: '2-digit' };
                document.getElementById('readUnlockedDate').innerText = `Enviado em ${dateObj.toLocaleString('pt-BR', formatOpts)}`;
            } else {
                unlockedState.classList.add('hidden');
                lockedState.classList.remove('hidden');

                document.getElementById('readLockedSender').innerText = item.senderName;

                const targetDate = new Date(currentTreeData.revealDate);
                document.getElementById('readLockedTargetDate').innerText = targetDate.toLocaleString('pt-BR');
            }

            modalReadMessage.classList.remove('hidden');
            modalReadMessage.classList.add('flex');
        }

        // ==================== INTERACTION: ADMIN PANEL ====================

        document.getElementById('btnOpenAdminPanel').addEventListener('click', () => {
            // Reset state inside Modal Admin before opening
            document.getElementById('adminPinState').classList.remove('hidden');
            document.getElementById('adminDashboardState').classList.add('hidden');
            document.getElementById('adminPinInput').value = '';
            
            modalAdmin.classList.remove('hidden');
            modalAdmin.classList.add('flex');
        });

        // PIN validation code
        document.getElementById('btnVerifyAdminPin').addEventListener('click', () => {
            const inputVal = document.getElementById('adminPinInput').value;

            if (!currentTreeData) return;

            if (inputVal === currentTreeData.adminPin) {
                // Login successful inside panel
                document.getElementById('adminPinState').classList.add('hidden');
                document.getElementById('adminDashboardState').classList.remove('hidden');

                // Prepopulate current lock date time
                const formatLocalDate = (ms) => {
                    const date = new Date(ms);
                    const pad = (n) => String(n).padStart(2, '0');
                    return `${date.getFullYear()}-${pad(date.getMonth() + 1)}-${pad(date.getDate())}T${pad(date.getHours())}:${pad(date.getMinutes())}`;
                };
                document.getElementById('adminNewDateInput').value = formatLocalDate(currentTreeData.revealDate);

                // Prepopulate current active visual theme
                const activeTheme = currentTreeData.theme || 'christmas';
                const themeRadio = document.querySelector(`input[name="adminTreeTheme"][value="${activeTheme}"]`);
                if (themeRadio) themeRadio.checked = true;

                // Prepopulate current active visibility mode
                const activeRevealMode = currentTreeData.revealMode || 'locked';
                const visibilityRadio = document.querySelector(`input[name="adminTreeVisibility"][value="${activeRevealMode}"]`);
                if (visibilityRadio) visibilityRadio.checked = true;

            } else {
                showToast("PIN incorreto! Verifique seu código.", "fa-solid fa-circle-xmark");
            }
        });

        // Save updated reveal date and theme visual design
        document.getElementById('btnSaveAdminSettings').addEventListener('click', async () => {
            const newDateStr = document.getElementById('adminNewDateInput').value;
            const newTheme = document.querySelector('input[name="adminTreeTheme"]:checked').value;
            const newRevealMode = document.querySelector('input[name="adminTreeVisibility"]:checked').value;

            if (!newDateStr) {
                showToast("Selecione uma data válida.", "fa-solid fa-triangle-exclamation");
                return;
            }

            const newDateMs = new Date(newDateStr).getTime();

            try {
                // Update both configuration variables in Firestore
                const treeDocRef = doc(db, 'artifacts', appId, 'public', 'data', 'trees', currentTreeId);
                await updateDoc(treeDocRef, { 
                    revealDate: newDateMs,
                    theme: newTheme,
                    revealMode: newRevealMode
                });

                showToast("Configurações da árvore atualizadas!", "fa-solid fa-circle-check");
                closeAllModals();

            } catch (err) {
                console.error("Error updating settings:", err);
                showToast("Erro ao tentar salvar configurações.", "fa-solid fa-triangle-exclamation");
            }
        });

        // ==================== CLIPBOARD & SHARE WORKFLOW ====================

        document.getElementById('btnShareTree').addEventListener('click', () => {
            const pathLink = window.location.href;
            
            const inputTemp = document.createElement('input');
            inputTemp.value = pathLink;
            document.body.appendChild(inputTemp);
            inputTemp.select();
            document.execCommand('copy');
            document.body.removeChild(inputTemp);

            showToast("Link copiado! Compartilhe com quem quiser.", "fa-solid fa-clipboard-check");
        });

        // ==================== UTILS AND HELPER PLUGINS ====================

        // Close action handlers on modal triggers
        document.querySelectorAll('.btnCloseModal').forEach(btn => {
            btn.addEventListener('click', closeAllModals);
        });

        function closeAllModals() {
            modalOrnament.classList.add('hidden');
            modalOrnament.classList.remove('flex');
            modalReadMessage.classList.add('hidden');
            modalReadMessage.classList.remove('flex');
            modalAdmin.classList.add('hidden');
            modalAdmin.classList.remove('flex');
        }

        // Custom beautiful and non-blocking notification Toasts
        let toastTimeout = null;
        function showToast(message, iconName = "fa-solid fa-bell") {
            const toast = document.getElementById('toastNotification');
            const msgSpan = document.getElementById('toastMsg');
            const iconSpan = document.getElementById('toastIcon');

            msgSpan.innerText = message;
            iconSpan.innerHTML = `<i class="${iconName}"></i>`;

            toast.classList.remove('opacity-0', 'translate-y-10', 'pointer-events-none');
            toast.classList.add('opacity-100', 'translate-y-0');

            if (toastTimeout) clearTimeout(toastTimeout);
            toastTimeout = setTimeout(hideToast, 4000);
        }

        window.hideToast = function hideToast() {
            const toast = document.getElementById('toastNotification');
            toast.classList.add('opacity-0', 'translate-y-10', 'pointer-events-none');
            toast.classList.remove('opacity-100', 'translate-y-0');
        };

        // Falling Snow Animation logic using HTML5 Canvas
        function initSnow() {
            const canvas = document.getElementById('snowCanvas');
            const ctx = canvas.getContext('2d');

            let width = window.innerWidth;
            let height = window.innerHeight;
            canvas.width = width;
            canvas.height = height;

            const particles = [];
            const maxParticles = 55;

            for (let i = 0; i < maxParticles; i++) {
                particles.push({
                    x: Math.random() * width,
                    y: Math.random() * height,
                    r: Math.random() * 2.5 + 0.5,
                    d: Math.random() * maxParticles,
                    speed: Math.random() * 0.6 + 0.2
                });
            }

            function draw() {
                ctx.clearRect(0, 0, width, height);
                ctx.fillStyle = "rgba(255, 255, 255, 0.75)";
                ctx.beginPath();
                for (let i = 0; i < maxParticles; i++) {
                    const p = particles[i];
                    ctx.moveTo(p.x, p.y);
                    ctx.arc(p.x, p.y, p.r, 0, Math.PI * 2, true);
                }
                ctx.fill();
                update();
            }

            function update() {
                for (let i = 0; i < maxParticles; i++) {
                    const p = particles[i];
                    p.y += p.speed;
                    p.x += Math.sin(p.d) * 0.2;

                    if (p.y > height) {
                        particles[i] = {
                            x: Math.random() * width,
                            y: -10,
                            r: p.r,
                            d: p.d,
                            speed: p.speed
                        };
                    }
                }
            }

            function loop() {
                draw();
                requestAnimationFrame(loop);
            }

            window.addEventListener('resize', () => {
                width = window.innerWidth;
                height = window.innerHeight;
                canvas.width = width;
                canvas.height = height;
            });

            loop();
        }
    </script>
</body>
</html>
