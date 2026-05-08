<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Mensageiro CLX</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; height: 100dvh; margin: 0; padding: 0; overflow: hidden; }
        .glass { background: rgba(255, 255, 255, 0.95); backdrop-filter: blur(10px); }
        .msg-self { border-radius: 18px 18px 2px 18px; align-self: flex-end; }
        .msg-other { border-radius: 18px 18px 18px 2px; align-self: flex-start; }
        ::-webkit-scrollbar { width: 4px; }
        ::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 10px; }
        .active-chat-item { background-color: #f1f5f9; border-left: 4px solid #2563eb; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(8px); } to { opacity: 1; transform: translateY(0); } }
        .animate-in { animation: fadeIn 0.2s ease-out forwards; }
        #messages-container { display: flex; flex-direction: column; }
    </style>
</head>
<body class="bg-slate-100">

    <!-- Tela de Login -->
    <div id="login-screen" class="fixed inset-0 z-50 flex items-center justify-center bg-slate-900 px-4">
        <div class="bg-white p-8 rounded-3xl shadow-2xl w-full max-w-md">
            <div class="text-center mb-8">
                <div class="w-16 h-16 bg-blue-600 rounded-2xl flex items-center justify-center text-white text-3xl font-bold mx-auto mb-4 shadow-lg shadow-blue-200">CLX</div>
                <h1 class="text-2xl font-bold text-slate-800">Mensageiro CLX</h1>
                <p class="text-slate-500 text-sm">Conecte-se com segurança</p>
            </div>

            <div id="auth-fields" class="space-y-4">
                <input type="text" id="login-user" placeholder="Usuário" class="w-full px-4 py-3 rounded-xl border border-slate-200 focus:ring-2 focus:ring-blue-500 outline-none transition">
                <div id="email-field" class="hidden">
                    <input type="email" id="login-email" placeholder="Seu melhor E-mail" class="w-full px-4 py-3 rounded-xl border border-slate-200 focus:ring-2 focus:ring-blue-500 outline-none transition">
                </div>
                <input type="password" id="login-pass" placeholder="Senha" class="w-full px-4 py-3 rounded-xl border border-slate-200 focus:ring-2 focus:ring-blue-500 outline-none transition">
                
                <button onclick="handleAuth()" id="btn-auth" class="w-full bg-blue-600 text-white py-3 rounded-xl font-semibold hover:bg-blue-700 transition shadow-lg shadow-blue-100 active:scale-95">Entrar</button>
                
                <div class="text-center mt-4">
                    <button id="toggle-mode" onclick="toggleAuthMode()" class="text-sm text-blue-600 hover:underline">Não tem conta? Criar conta agora</button>
                </div>
            </div>
            <p id="login-error" class="text-red-500 text-xs mt-4 text-center hidden font-medium"></p>
        </div>
    </div>

    <!-- Tela Principal do App -->
    <div id="app-screen" class="hidden h-screen w-full flex overflow-hidden bg-white">
        
        <!-- Barra Lateral -->
        <aside id="sidebar" class="w-full md:w-80 flex flex-col border-r border-slate-200 bg-white z-20 h-full">
            <div class="p-4 border-b border-slate-100 bg-white">
                <div class="flex items-center justify-between mb-4">
                    <div class="flex items-center space-x-3 overflow-hidden">
                        <div id="my-avatar" class="w-10 h-10 bg-slate-800 rounded-full flex-shrink-0 flex items-center justify-center text-white font-bold text-sm shadow-inner">U</div>
                        <div class="overflow-hidden">
                            <p id="my-name" class="font-bold text-slate-800 leading-none truncate">Usuário</p>
                            <p id="my-id-display" class="text-[10px] text-blue-600 font-bold mt-1">#000000</p>
                        </div>
                    </div>
                    <button onclick="logout()" class="p-2 text-slate-400 hover:text-red-500 transition">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17 16l4-4m0 0l-4-4m4 4H7m6 4v1a3 3 0 01-3 3H6a3 3 0 01-3-3V7a3 3 0 013-3h4a3 3 0 013 3v1" /></svg>
                    </button>
                </div>

                <!-- Botões Master CLX -->
                <div id="master-controls" class="hidden grid grid-cols-2 gap-2 mb-4">
                    <button onclick="toggleMasterPanel('users')" class="text-[9px] font-black tracking-tighter bg-slate-900 text-white py-2 rounded-lg hover:bg-black transition uppercase">Usuários</button>
                    <button onclick="toggleMasterPanel('chats')" class="text-[9px] font-black tracking-tighter bg-blue-600 text-white py-2 rounded-lg hover:bg-blue-700 transition uppercase">Chats</button>
                </div>

                <div class="relative">
                    <input type="text" id="search-id" placeholder="Adicionar ID de alguém..." class="w-full pl-10 pr-4 py-2.5 bg-slate-100 border-none rounded-xl text-sm focus:ring-2 focus:ring-blue-500 transition outline-none">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 absolute left-3 top-3 text-slate-400" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4" /></svg>
                </div>
            </div>

            <!-- Lista de Conversas Recentes / Contatos -->
            <div id="contacts-list" class="flex-1 overflow-y-auto p-2 space-y-1 bg-white">
                <!-- Preenchido via Firebase -->
            </div>
        </aside>

        <main id="chat-area" class="flex-1 hidden md:flex flex-col bg-white h-full relative overflow-hidden">
            
            <!-- Mensagem Inicial -->
            <div id="chat-welcome" class="flex-1 flex flex-col items-center justify-center p-8 text-center bg-slate-50">
                <div class="w-20 h-20 bg-blue-100 rounded-full flex items-center justify-center mb-4">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-10 w-10 text-blue-600" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 12h.01M12 12h.01M16 12h.01M21 12c0 4.418-4.03 8-9 8a9.863 9.863 0 01-4.255-.949L3 20l1.395-3.72C3.512 15.042 3 13.574 3 12c0-4.418 4.03-8 9-8s9 3.582 9 8z" /></svg>
                </div>
                <h2 class="text-xl font-bold text-slate-800">Mensagens CLX</h2>
                <p class="text-slate-400 max-w-xs mx-auto mt-2 text-xs leading-relaxed">Selecione uma conversa ao lado ou adicione um ID para começar a enviar mensagens.</p>
            </div>

            <!-- Área Ativa do Chat -->
            <div id="active-chat" class="hidden flex-1 flex flex-col h-full bg-slate-50 overflow-hidden">
                <!-- Cabeçalho do Chat -->
                <header class="h-16 flex-shrink-0 flex items-center px-4 border-b border-slate-100 bg-white shadow-sm z-10">
                    <button onclick="backToSidebar()" class="md:hidden mr-3 p-2 text-slate-500 hover:bg-slate-100 rounded-full">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7" /></svg>
                    </button>
                    <div id="chat-avatar" class="w-10 h-10 bg-blue-600 rounded-full flex flex-shrink-0 items-center justify-center text-white font-bold shadow-sm text-sm">?</div>
                    <div class="ml-3 overflow-hidden">
                        <p id="chat-with-name" class="font-bold text-slate-800 leading-none truncate text-sm">Usuário</p>
                        <p class="text-[9px] text-green-500 font-bold uppercase tracking-widest mt-1">Chat Seguro 24h</p>
                    </div>
                </header>

                <!-- Container de Mensagens (Ocupa o meio) -->
                <div id="messages-container" class="flex-1 overflow-y-auto p-4 space-y-3">
                    <!-- Mensagens aqui -->
                </div>

                <!-- Rodapé para Escrever (Sempre no fundo) -->
                <footer class="flex-shrink-0 bg-white p-3 border-t border-slate-100">
                    <div class="flex items-center space-x-2 max-w-4xl mx-auto">
                        <input type="text" id="msg-input" placeholder="Escreva sua mensagem..." class="flex-1 px-4 py-3 rounded-2xl bg-slate-100 border-none focus:ring-2 focus:ring-blue-600 transition text-sm outline-none">
                        <button onclick="sendMessage()" class="p-3 bg-blue-600 text-white rounded-2xl hover:bg-blue-700 transition shadow-lg active:scale-95">
                            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 19l9 2-9-18-9 18 9-2zm0 0v-8" /></svg>
                        </button>
                    </div>
                </footer>
            </div>
        </main>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/9.6.10/firebase-app.js";
        import { getFirestore, collection, doc, setDoc, getDoc, query, where, onSnapshot, addDoc, orderBy, serverTimestamp, getDocs, deleteDoc, limit } from "https://www.gstatic.com/firebasejs/9.6.10/firebase-firestore.js";

        const firebaseConfig = {
            apiKey: "AIzaSyAJcdIIRpqeoG4Y9mA3w8WeRQxKe7JvWjs",
            authDomain: "site-de-mensagem-bda62.firebaseapp.com",
            projectId: "site-de-mensagem-bda62",
            storageBucket: "site-de-mensagem-bda62.firebasestorage.app",
            messagingSenderId: "264723387686",
            appId: "1:264723387686:web:6d60439df5f0edf66a1de7"
        };

        const app = initializeApp(firebaseConfig);
        const db = getFirestore(app);

        let currentUser = null;
        let activeChatId = null;
        let isSignUp = false;
        let unsubMessages = null;
        let unsubRooms = null;

        /* Alternar entre Login e Cadastro */
        window.toggleAuthMode = () => {
            isSignUp = !isSignUp;
            document.getElementById('email-field').classList.toggle('hidden', !isSignUp);
            document.getElementById('btn-auth').innerText = isSignUp ? 'Criar minha conta agora' : 'Entrar';
            document.getElementById('toggle-mode').innerText = isSignUp ? 'Já tem conta? Entrar' : 'Não tem conta? Criar conta agora';
        };

        window.handleAuth = async () => {
            const userRaw = document.getElementById('login-user').value.trim();
            const user = userRaw.toLowerCase();
            const pass = document.getElementById('login-pass').value.trim();
            const email = document.getElementById('login-email').value.trim();
            const errorEl = document.getElementById('login-error');

            if (!user || !pass) {
                errorEl.innerText = "Preencha usuário e senha!";
                errorEl.classList.remove('hidden');
                return;
            }
            errorEl.classList.add('hidden');

            try {
                if (isSignUp) {
                    if (!email) {
                        errorEl.innerText = "E-mail é obrigatório para cadastro!";
                        errorEl.classList.remove('hidden');
                        return;
                    }
                    const userDoc = await getDoc(doc(db, "clx_users", user));
                    if (userDoc.exists()) {
                        errorEl.innerText = "Este login já existe! Tente outro.";
                        errorEl.classList.remove('hidden');
                        return;
                    }
                    const randomId = Math.floor(100000 + Math.random() * 900000).toString();
                    const newUser = { login: userRaw, pass: pass, email: email, id: randomId };
                    await setDoc(doc(db, "clx_users", user), newUser);
                    loginSuccess(newUser);
                } else {
                    if (user === 'clx' && pass === '02072007') {
                        loginSuccess({ login: 'CLX', id: 'MASTER', isMaster: true });
                        return;
                    }
                    const userDoc = await getDoc(doc(db, "clx_users", user));
                    if (userDoc.exists() && userDoc.data().pass === pass) {
                        loginSuccess(userDoc.data());
                    } else {
                        errorEl.innerText = "Usuário ou senha incorretos!";
                        errorEl.classList.remove('hidden');
                    }
                }
            } catch (e) {
                console.error("Erro na autenticação:", e);
                errorEl.innerText = "Erro ao conectar. Verifique sua rede.";
                errorEl.classList.remove('hidden');
            }
        };

        function loginSuccess(userData) {
            currentUser = {
                login: userData.login || "Usuário",
                id: userData.id || "000000",
                isMaster: userData.isMaster || false
            };
            
            document.getElementById('login-screen').classList.add('hidden');
            document.getElementById('app-screen').classList.remove('hidden');
            document.getElementById('my-name').innerText = currentUser.login.toUpperCase();
            document.getElementById('my-id-display').innerText = `#${currentUser.id}`;
            document.getElementById('my-avatar').innerText = (currentUser.login[0] || "U").toUpperCase();
            
            if (currentUser.isMaster) {
                document.getElementById('master-controls').classList.remove('hidden');
            }

            loadConversations();
        }

        function loadConversations() {
            if (unsubRooms) unsubRooms();
            const roomsRef = collection(db, "clx_active_rooms");
            
            unsubRooms = onSnapshot(roomsRef, (snapshot) => {
                const list = document.getElementById('contacts-list');
                const currentMode = list.getAttribute('data-mode');
                if (currentMode === 'master-users') return;

                list.innerHTML = "";
                if (snapshot.empty) {
                    list.innerHTML = "<p class='text-center text-[10px] text-slate-400 mt-10 uppercase font-bold'>Nenhuma conversa ainda</p>";
                }
                
                snapshot.forEach(roomDoc => {
                    const data = roomDoc.data();
                    const participants = data.participantsIds || [];
                    const names = data.participantsNames || [];
                    
                    if (currentUser && (participants.includes(currentUser.id) || currentUser.isMaster)) {
                        const otherName = names.find(n => n.toLowerCase() !== currentUser.login.toLowerCase()) || "Conversa";
                        
                        const div = document.createElement('div');
                        div.className = `flex items-center p-3 rounded-xl cursor-pointer transition mb-1 ${activeChatId === roomDoc.id ? 'active-chat-item' : 'hover:bg-slate-50'}`;
                        div.onclick = () => selectChat(otherName, roomDoc.id);
                        div.innerHTML = `
                            <div class="w-10 h-10 bg-blue-100 rounded-full flex-shrink-0 flex items-center justify-center text-blue-600 font-bold text-sm">${otherName[0].toUpperCase()}</div>
                            <div class="ml-3 overflow-hidden">
                                <p class="text-sm font-bold text-slate-800 truncate">${otherName}</p>
                                <p class="text-[9px] text-slate-400 font-bold">ÚLTIMO CONTATO</p>
                            </div>
                        `;
                        list.appendChild(div);
                    }
                });
            });
        }

        async function selectChat(name, chatId) {
            activeChatId = chatId;
            
            if (window.innerWidth < 768) {
                document.getElementById('sidebar').classList.add('hidden');
                document.getElementById('chat-area').classList.remove('hidden');
                document.getElementById('chat-area').classList.add('flex');
            }

            document.getElementById('chat-welcome').classList.add('hidden');
            document.getElementById('active-chat').classList.remove('hidden');
            document.getElementById('active-chat').classList.add('flex');
            document.getElementById('chat-with-name').innerText = name;
            document.getElementById('chat-avatar').innerText = name[0].toUpperCase();

            startMessageListener();
        }

        function startMessageListener() {
            if (unsubMessages) unsubMessages();
            
            /* Antes era clx_messenger/chats/ID/messages (4 partes - erro). Agora é clx_chats/ID/messages (3 partes - ok) */
            const msgRef = collection(db, "clx_chats", activeChatId, "messages");
            const q = query(msgRef, orderBy("timestamp", "asc"));
            
            unsubMessages = onSnapshot(q, (snapshot) => {
                const container = document.getElementById('messages-container');
                container.innerHTML = "";
                const now = Date.now();
                const oneDayInMs = 24 * 60 * 60 * 1000;

                snapshot.forEach(async (mDoc) => {
                    const m = mDoc.data();
                    
                    // Verificação de 24 horas
                    if (m.timestamp && (now - m.timestamp.toMillis() > oneDayInMs)) {
                        await deleteDoc(doc(db, "clx_chats", activeChatId, "messages", mDoc.id));
                        return;
                    }

                    const isMine = m.senderId === currentUser.id;
                    const div = document.createElement('div');
                    div.className = `flex w-full ${isMine ? 'justify-end' : 'justify-start'} animate-in`;
                    div.innerHTML = `
                        <div class="max-w-[80%] p-3 shadow-sm ${isMine ? 'bg-blue-600 text-white msg-self' : 'bg-white text-slate-700 msg-other border border-slate-100'}">
                            <p class="text-[13px] leading-relaxed break-words">${m.text}</p>
                            <p class="text-[8px] mt-1 opacity-60 text-right font-bold">${m.timestamp ? new Date(m.timestamp.toMillis()).toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'}) : '...'}</p>
                        </div>
                    `;
                    container.appendChild(div);
                });
                // Rolar para o fim
                setTimeout(() => { container.scrollTop = container.scrollHeight; }, 100);
            });
        }

        window.sendMessage = async () => {
            const input = document.getElementById('msg-input');
            const text = input.value.trim();
            if (!text || !activeChatId) return;

            try {
                const msgRef = collection(db, "clx_chats", activeChatId, "messages");
                await addDoc(msgRef, {
                    text: text,
                    senderId: currentUser.id,
                    senderName: currentUser.login,
                    timestamp: serverTimestamp()
                });

                // Atualizar sala ativa para mostrar no topo da lista
                await setDoc(doc(db, "clx_active_rooms", activeChatId), { lastUpdate: serverTimestamp() }, { merge: true });

                input.value = "";
                input.focus();
            } catch (err) {
                console.error("Falha ao enviar:", err);
            }
        };

        document.getElementById('search-id').addEventListener('keypress', async (e) => {
            if (e.key === 'Enter') {
                const targetId = e.target.value.trim();
                if (!targetId || !currentUser || targetId === currentUser.id) return;

                const usersRef = collection(db, "clx_users");
                const q = query(usersRef, where("id", "==", targetId), limit(1));
                const snap = await getDocs(q);

                if (!snap.empty) {
                    const otherUser = snap.docs[0].data();
                    const chatId = [currentUser.id, otherUser.id].sort().join('_');
                    
                    await setDoc(doc(db, "clx_active_rooms", chatId), {
                        participantsIds: [currentUser.id, otherUser.id],
                        participantsNames: [currentUser.login, otherUser.login],
                        lastUpdate: serverTimestamp()
                    });

                    selectChat(otherUser.login, chatId);
                    e.target.value = "";
                } else {
                    alert("Usuário não encontrado!");
                }
            }
        });

        window.toggleMasterPanel = async (mode) => {
            const list = document.getElementById('contacts-list');
            list.innerHTML = "<p class='p-4 text-[10px] text-slate-400 font-bold uppercase animate-pulse text-center'>Acessando dados...</p>";
            
            if (mode === 'users') {
                list.setAttribute('data-mode', 'master-users');
                const snap = await getDocs(collection(db, "clx_users"));
                list.innerHTML = "<h3 class='p-3 text-[10px] font-black text-slate-400 bg-slate-50 uppercase tracking-widest'>Todos os Usuários</h3>";
                
                snap.forEach(uDoc => {
                    const u = uDoc.data();
                    const div = document.createElement('div');
                    div.className = "p-3 border-b border-slate-50 hover:bg-slate-50 transition";
                    div.innerHTML = `
                        <p class='text-xs font-bold text-slate-900'>${(u.login || "SEM NOME").toUpperCase()}</p>
                        <p class='text-[10px] text-blue-600 font-bold'>ID: #${u.id || '000000'}</p>
                    `;
                    list.appendChild(div);
                });
            } else {
                list.setAttribute('data-mode', 'chats');
                loadConversations();
            }
        };

        window.backToSidebar = () => {
            document.getElementById('sidebar').classList.remove('hidden');
            document.getElementById('chat-area').classList.add('hidden');
            activeChatId = null;
        };

        window.logout = () => location.reload();

        // Enviar com a tecla Enter
        document.getElementById('msg-input').addEventListener('keypress', (e) => {
            if (e.key === 'Enter') sendMessage();
        });

    </script>
</body>
</html>
