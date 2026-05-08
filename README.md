<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Mensageiro CLX</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    <style>
        :root { --sat: env(safe-area-inset-top); --sab: env(safe-area-inset-bottom); }
        body { 
            font-family: 'Inter', sans-serif; 
            height: 100dvh; 
            margin: 0; 
            padding: 0; 
            overflow: hidden; 
            background-color: #f8fafc;
        }
        .safe-bottom { padding-bottom: var(--sab); }
        
        .msg-self { border-radius: 18px 18px 2px 18px; background-color: #2563eb; color: white; align-self: flex-end; }
        .msg-other { border-radius: 18px 18px 18px 2px; background-color: white; color: #1e293b; align-self: flex-start; border: 1px solid #e2e8f0; }
        
        #messages-container { flex: 1; overflow-y: auto; display: flex; flex-direction: column; padding: 1rem; gap: 0.75rem; }
        
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
        .animate-msg { animation: fadeIn 0.2s ease-out forwards; }

        /* Ajuste para garantir que o campo de texto não suma */
        #active-chat-view { height: 100%; display: flex; flex-direction: column; position: relative; }
        #input-area { flex-shrink: 0; z-index: 10; background: white; border-top: 1px solid #e2e8f0; padding: 0.75rem 1rem; }
    </style>
</head>
<body>

    <div id="login-screen" class="fixed inset-0 z-[100] flex items-center justify-center bg-slate-900 px-4">
        <div class="bg-white p-8 rounded-3xl shadow-2xl w-full max-w-md">
            <div class="text-center mb-6">
                <div class="w-16 h-16 bg-blue-600 rounded-2xl flex items-center justify-center text-white text-3xl font-bold mx-auto mb-4 shadow-lg">CLX</div>
                <h1 class="text-2xl font-bold text-slate-800">Mensageiro CLX</h1>
                <p id="auth-subtitle" class="text-slate-500 text-sm">Conecte-se com segurança</p>
            </div>

            <div class="space-y-3">
                <input type="text" id="login-user" placeholder="Nome de Usuário" class="w-full px-4 py-3 rounded-xl border border-slate-200 focus:ring-2 focus:ring-blue-500 outline-none transition">
                
                <div id="email-container" class="hidden">
                    <input type="email" id="login-email" placeholder="Seu melhor E-mail" class="w-full px-4 py-3 rounded-xl border border-slate-200 focus:ring-2 focus:ring-blue-500 outline-none transition">
                </div>

                <input type="password" id="login-pass" placeholder="Sua Senha" class="w-full px-4 py-3 rounded-xl border border-slate-200 focus:ring-2 focus:ring-blue-500 outline-none transition">
                
                <button onclick="handleAuth()" id="btn-auth" class="w-full bg-blue-600 text-white py-3.5 rounded-xl font-bold hover:bg-blue-700 transition active:scale-95">Entrar</button>
                
                <button onclick="toggleAuthMode()" id="toggle-mode" class="w-full text-sm text-blue-600 font-medium hover:underline mt-2">Não tem conta? Criar conta</button>
            </div>
            <p id="login-error" class="text-red-500 text-xs mt-4 text-center hidden font-bold bg-red-50 p-2 rounded-lg"></p>
        </div>
    </div>

    <div id="app-screen" class="hidden flex h-full w-full bg-white overflow-hidden">
        
        <aside id="sidebar" class="w-full md:w-80 flex flex-col border-r border-slate-200 bg-white z-50 h-full">
            <header class="p-4 border-b border-slate-100">
                <div class="flex items-center justify-between mb-4">
                    <div class="flex items-center space-x-3">
                        <div id="my-avatar" class="w-10 h-10 bg-slate-800 rounded-full flex items-center justify-center text-white font-bold">U</div>
                        <div class="leading-tight">
                            <p id="my-name" class="font-bold text-slate-800 truncate text-sm">USUÁRIO</p>
                            <p id="my-id-display" class="text-[10px] text-blue-600 font-black">#000000</p>
                        </div>
                    </div>
                    <button onclick="location.reload()" class="p-2 text-slate-400 hover:text-red-500">Sair</button>
                </div>

                <div id="master-panel" class="hidden grid grid-cols-2 gap-2 mb-4">
                    <button onclick="loadMasterView('users')" class="bg-slate-900 text-white text-[10px] font-bold py-2 rounded-lg uppercase">Usuários</button>
                    <button onclick="loadMasterView('chats')" class="bg-blue-600 text-white text-[10px] font-bold py-2 rounded-lg uppercase">Monitorar</button>
                </div>

                <div class="relative">
                    <input type="text" id="search-input" placeholder="Adicionar por ID..." class="w-full pl-10 pr-4 py-2.5 bg-slate-100 rounded-xl text-xs focus:ring-2 focus:ring-blue-500 outline-none">
                </div>
            </header>

            <div id="list-container" class="flex-1 overflow-y-auto p-2 space-y-1">
                <!-- Lista dinâmica -->
            </div>
        </aside>

        <main id="chat-area" class="flex-1 hidden md:block bg-slate-50 relative h-full">
            <div id="welcome-view" class="h-full flex flex-col items-center justify-center p-8 text-center">
                <h2 class="text-xl font-bold text-slate-800">Mensagens CLX</h2>
                <p class="text-slate-400 text-xs mt-2">Adicione um ID para conversar.</p>
            </div>

            <div id="active-chat-view" class="hidden">
                <header class="h-16 flex-shrink-0 bg-white border-b border-slate-200 px-4 flex items-center shadow-sm">
                    <button onclick="closeChat()" class="md:hidden mr-3 p-2 text-slate-500 bg-slate-100 rounded-full">←</button>
                    <div id="target-avatar" class="w-10 h-10 bg-blue-600 rounded-full flex items-center justify-center text-white font-bold mr-3">?</div>
                    <p id="target-name" class="font-bold text-slate-800 text-sm">USUÁRIO</p>
                </header>

                <div id="messages-container"></div>

                <footer id="input-area" class="safe-bottom">
                    <div class="flex items-center space-x-2">
                        <input type="text" id="message-input" placeholder="Mensagem..." class="flex-1 bg-slate-100 rounded-xl px-4 py-3 text-sm outline-none">
                        <button onclick="sendMessage()" class="p-3 bg-blue-600 text-white rounded-xl">Enviar</button>
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
            document.getElementById('email-container').classList.toggle('hidden', !isSignUp);
            document.getElementById('btn-auth').innerText = isSignUp ? 'Criar minha conta' : 'Entrar';
            document.getElementById('auth-subtitle').innerText = isSignUp ? 'Cadastre-se para começar' : 'Conecte-se com segurança';
            document.getElementById('toggle-mode').innerText = isSignUp ? 'Já tem conta? Entrar' : 'Não tem conta? Criar conta';
        };

        window.handleAuth = async () => {
            const userRaw = document.getElementById('login-user').value.trim();
            const user = userRaw.toLowerCase();
            const pass = document.getElementById('login-pass').value.trim();
            const email = document.getElementById('login-email').value.trim();
            const errorEl = document.getElementById('login-error');

            errorEl.classList.add('hidden');

            if (!user || !pass) {
                errorEl.innerText = "Preencha usuário e senha!";
                errorEl.classList.remove('hidden');
                return;
            }

            try {
                if (isSignUp) {
                    if (!email.includes('@')) {
                        errorEl.innerText = "E-mail inválido!";
                        errorEl.classList.remove('hidden');
                        return;
                    }

                    /* Verificar se login já existe */
                    const userDoc = await getDoc(doc(db, "clx_users", user));
                    if (userDoc.exists()) {
                        errorEl.innerText = "Este nome de usuário já existe!";
                        errorEl.classList.remove('hidden');
                        return;
                    }

                    /* Criar Novo Usuário */
                    const randomId = Math.floor(100000 + Math.random() * 900000).toString();
                    const userData = { 
                        login: userRaw, 
                        pass: pass, 
                        email: email, 
                        id: randomId,
                        createdAt: serverTimestamp() 
                    };
                    
                    await setDoc(doc(db, "clx_users", user), userData);
                    loginSuccess(userData);

                } else {
                    /* Login Master */
                    if (user === 'clx' && pass === '02072007') {
                        loginSuccess({ login: 'CLX', id: 'MASTER', isMaster: true });
                        return;
                    }

                    /* Login Normal */
                    const userDoc = await getDoc(doc(db, "clx_users", user));
                    if (userDoc.exists()) {
                        const data = userDoc.data();
                        if (data.pass === pass) {
                            loginSuccess(data);
                        } else {
                            errorEl.innerText = "Senha incorreta!";
                            errorEl.classList.remove('hidden');
                        }
                    } else {
                        errorEl.innerText = "Usuário não encontrado!";
                        errorEl.classList.remove('hidden');
                    }
                }
            } catch (e) {
                console.error(e);
                errorEl.innerText = "Erro ao conectar com o servidor.";
                errorEl.classList.remove('hidden');
            }
        };

        function loginSuccess(userData) {
            currentUser = { ...userData, isMaster: userData.isMaster || false };
            document.getElementById('login-screen').classList.add('hidden');
            document.getElementById('app-screen').classList.remove('hidden');
            document.getElementById('my-name').innerText = (currentUser.login || "USUÁRIO").toUpperCase();
            document.getElementById('my-id-display').innerText = `#${currentUser.id || '000000'}`;
            document.getElementById('my-avatar').innerText = (currentUser.login?.[0] || "U").toUpperCase();
            
            if (currentUser.isMaster) document.getElementById('master-panel').classList.remove('hidden');
            
            loadRecentConversations();
        }

        function loadRecentConversations() {
            if (unsubRooms) unsubRooms();
            unsubRooms = onSnapshot(collection(db, "clx_rooms"), (snapshot) => {
                const list = document.getElementById('list-container');
                if (list.getAttribute('data-view') === 'master-users') return;
                list.innerHTML = "";

                snapshot.forEach(docSnap => {
                    const data = docSnap.data();
                    const pIds = data.pIds || [];
                    const pNames = data.pNames || [];

                    if (currentUser && (pIds.includes(currentUser.id) || currentUser.isMaster)) {
                        const otherIdx = pNames.findIndex(n => n.toLowerCase() !== (currentUser.login || "").toLowerCase());
                        const otherName = otherIdx !== -1 ? pNames[otherIdx] : "Conversa";
                        
                        const div = document.createElement('div');
                        div.className = `flex items-center p-3 rounded-xl cursor-pointer transition ${activeChatId === docSnap.id ? 'bg-blue-50 border-l-4 border-blue-600' : 'hover:bg-slate-50'}`;
                        div.onclick = () => selectChat(otherName, docSnap.id);
                        div.innerHTML = `
                            <div class="w-10 h-10 bg-blue-100 rounded-full flex items-center justify-center text-blue-600 font-bold">${otherName[0].toUpperCase()}</div>
                            <div class="ml-3 overflow-hidden">
                                <p class="text-xs font-bold text-slate-800 truncate">${otherName}</p>
                                <p class="text-[9px] text-green-500 font-bold uppercase">Chat Ativo</p>
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
            }
            document.getElementById('welcome-view').classList.add('hidden');
            document.getElementById('active-chat-view').classList.remove('hidden');
            document.getElementById('active-chat-view').classList.add('flex');
            document.getElementById('target-name').innerText = name;
            document.getElementById('target-avatar').innerText = name[0].toUpperCase();
            startListeningMessages();
        }

        function startListeningMessages() {
            if (unsubMessages) unsubMessages();
            const msgRef = collection(db, "clx_chats", activeChatId, "messages");
            const q = query(msgRef, orderBy("timestamp", "asc"));
            
            unsubMessages = onSnapshot(q, (snapshot) => {
                const container = document.getElementById('messages-container');
                container.innerHTML = "";
                const now = Date.now();
                const dayMs = 24 * 60 * 60 * 1000;

                snapshot.forEach(async (mDoc) => {
                    const m = mDoc.data();
                    
                    /* Regra 24h: Deletar se for antigo */
                    if (m.timestamp && (now - m.timestamp.toMillis() > dayMs)) {
                        await deleteDoc(doc(db, "clx_chats", activeChatId, "messages", mDoc.id));
                        return;
                    }

                    const isMine = m.sId === currentUser.id;
                    const div = document.createElement('div');
                    div.className = `flex w-full ${isMine ? 'justify-end' : 'justify-start'} animate-msg`;
                    div.innerHTML = `
                        <div class="max-w-[75%] p-3 shadow-sm ${isMine ? 'msg-self' : 'msg-other'}">
                            <p class="text-[13px] leading-snug break-words">${m.txt}</p>
                            <p class="text-[8px] mt-1 opacity-60 text-right font-bold">
                                ${m.timestamp ? new Date(m.timestamp.toMillis()).toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'}) : ''}
                            </p>
                        </div>
                    `;
                    container.appendChild(div);
                });
                setTimeout(() => { container.scrollTop = container.scrollHeight; }, 100);
            });
        }

        window.sendMessage = async () => {
            const input = document.getElementById('message-input');
            const txt = input.value.trim();
            if (!txt || !activeChatId) return;

            try {
                await addDoc(collection(db, "clx_chats", activeChatId, "messages"), {
                    txt: txt,
                    sId: currentUser.id,
                    sName: currentUser.login,
                    timestamp: serverTimestamp()
                });
                await setDoc(doc(db, "clx_rooms", activeChatId), { last: serverTimestamp() }, { merge: true });
                input.value = "";
                input.focus();
            } catch (err) { console.error(err); }
        };

        document.getElementById('search-input').addEventListener('keypress', async (e) => {
            if (e.key === 'Enter') {
                const id = e.target.value.trim();
                if (!id || id === currentUser.id) return;

                const q = query(collection(db, "clx_users"), where("id", "==", id), limit(1));
                const snap = await getDocs(q);

                if (!snap.empty) {
                    const other = snap.docs[0].data();
                    const chatId = [currentUser.id, other.id].sort().join('_');
                    await setDoc(doc(db, "clx_rooms", chatId), {
                        pIds: [currentUser.id, other.id],
                        pNames: [currentUser.login, other.login],
                        last: serverTimestamp()
                    }, { merge: true });
                    selectChat(other.login, chatId);
                    e.target.value = "";
                }
            }
        });

        window.loadMasterView = async (mode) => {
            const list = document.getElementById('list-container');
            list.innerHTML = "<p class='text-center p-4 text-[10px] font-bold animate-pulse'>CARREGANDO...</p>";
            
            if (mode === 'users') {
                list.setAttribute('data-view', 'master-users');
                const snap = await getDocs(collection(db, "clx_users"));
                list.innerHTML = "<div class='text-[10px] font-black p-2 text-slate-400 uppercase'>Todos Usuários</div>";
                snap.forEach(uDoc => {
                    const u = uDoc.data();
                    const div = document.createElement('div');
                    div.className = "p-3 border-b border-slate-100";
                    div.innerHTML = `<p class='text-xs font-bold'>${(u.login || "N/A").toUpperCase()}</p><p class='text-[10px] text-blue-600 font-bold'>ID: #${u.id || '000000'}</p>`;
                    list.appendChild(div);
                });
            } else {
                list.setAttribute('data-view', 'chats');
                loadRecentConversations();
            }
        };

        window.closeChat = () => {
            document.getElementById('sidebar').classList.remove('hidden');
            document.getElementById('chat-area').classList.add('hidden');
            activeChatId = null;
        };

        document.getElementById('message-input').addEventListener('keypress', (e) => { if(e.key === 'Enter') sendMessage(); });
    </script>
</body>
</html>
