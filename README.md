<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mensageiro CLX</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; background-color: #f8fafc; }
        .glass { background: rgba(255, 255, 255, 0.8); backdrop-filter: blur(10px); }
        .chat-container { height: calc(100vh - 180px); }
        ::-webkit-scrollbar { width: 6px; }
        ::-webkit-scrollbar-track { background: transparent; }
        ::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 10px; }
        
        @keyframes bounce-subtle { 0%, 100% { transform: translateY(0); } 50% { transform: translateY(-3px); } }
        .animate-bounce-subtle { animation: bounce-subtle 2s infinite; }
        
        @keyframes slideIn { from { transform: translateX(100%); opacity: 0; } to { transform: translateX(0); opacity: 1; } }
        .notification-toast { animation: slideIn 0.3s ease-out forwards; }
    </style>
</head>
<body class="h-screen overflow-hidden">

    <div id="auth-screen" class="h-full flex items-center justify-center p-4 bg-gradient-to-br from-blue-50 to-indigo-100">
        <div class="max-w-md w-full bg-white p-8 rounded-3xl shadow-xl border border-white">
            <div class="text-center mb-8">
                <div class="w-16 h-16 bg-blue-600 rounded-2xl flex items-center justify-center mx-auto mb-4 shadow-lg">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-8 w-8 text-white" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 10h.01M12 10h.01M16 10h.01M9 16H5a2 2 0 01-2-2V6a2 2 0 012-2h14a2 2 0 012 2v8a2 2 0 01-2 2h-5l-5 5v-5z" /></svg>
                </div>
                <h1 id="auth-title" class="text-2xl font-bold text-slate-800">Mensageiro CLX</h1>
                <p id="auth-subtitle" class="text-slate-500 text-sm mt-2">Identifique-se para continuar</p>
            </div>

            <div class="space-y-4">
                <div id="email-field" class="hidden">
                    <label class="block text-xs font-bold text-slate-500 uppercase mb-1 ml-1">E-mail</label>
                    <input type="email" id="auth-email" placeholder="seu@email.com" class="w-full px-4 py-3 rounded-xl bg-slate-50 border border-slate-200 focus:outline-none focus:ring-2 focus:ring-blue-500 transition">
                </div>
                <div>
                    <label class="block text-xs font-bold text-slate-500 uppercase mb-1 ml-1">Usuário</label>
                    <input type="text" id="auth-user" placeholder="Seu login" class="w-full px-4 py-3 rounded-xl bg-slate-50 border border-slate-200 focus:outline-none focus:ring-2 focus:ring-blue-500 transition">
                </div>
                <div>
                    <label class="block text-xs font-bold text-slate-500 uppercase mb-1 ml-1">Senha</label>
                    <input type="password" id="auth-pass" placeholder="••••••••" class="w-full px-4 py-3 rounded-xl bg-slate-50 border border-slate-200 focus:outline-none focus:ring-2 focus:ring-blue-500 transition">
                </div>
                <button onclick="handleAuth()" id="auth-btn" class="w-full bg-blue-600 text-white font-bold py-3 rounded-xl hover:bg-blue-700 shadow-lg shadow-blue-200 transition-all active:scale-[0.98]">Entrar</button>
            </div>

            <div class="mt-6 text-center">
                <button onclick="toggleAuthMode()" id="toggle-auth-btn" class="text-sm text-blue-600 font-medium hover:underline">Criar conta nova</button>
            </div>
        </div>
    </div>

    <div id="app-screen" class="hidden h-screen flex flex-col md:flex-row overflow-hidden">
        <aside class="w-full md:w-80 bg-white border-r border-slate-200 flex flex-col z-10">
            <div class="p-6 border-b border-slate-100">
                <div class="flex items-center justify-between mb-4">
                    <div>
                        <h2 id="display-name" class="font-bold text-slate-800 text-lg leading-tight uppercase">Usuário</h2>
                        <p id="display-id" class="text-xs font-mono text-slate-400">ID: #000000</p>
                    </div>
                    <button onclick="location.reload()" class="p-2 text-slate-400 hover:text-red-500 transition">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17 16l4-4m0 0l-4-4m4 4H7m6 4v1a3 3 0 01-3 3H6a3 3 0 01-3-3V7a3 3 0 013-3h4a3 3 0 013 3v1" /></svg>
                    </button>
                </div>

                <div id="master-badge" class="hidden mb-4">
                    <span class="inline-flex items-center px-2.5 py-0.5 rounded-full text-[10px] font-bold bg-amber-100 text-amber-800 border border-amber-200 mb-2">MODO MASTER ATIVO</span>
                    <div class="grid grid-cols-2 gap-2">
                        <button onclick="toggleMasterPanel('users')" class="text-[9px] bg-slate-800 text-white py-2 rounded-lg font-bold">USUÁRIOS</button>
                        <button onclick="toggleMasterPanel('rooms')" class="text-[9px] bg-indigo-600 text-white py-2 rounded-lg font-bold">MONITORAR</button>
                    </div>
                </div>

                <div class="relative">
                    <input type="text" id="search-id" placeholder="Adicionar por ID..." class="w-full pl-3 pr-10 py-2 bg-slate-100 border-none rounded-xl text-sm focus:ring-2 focus:ring-blue-500">
                    <button onclick="addContactById()" class="absolute right-2 top-1/2 -translate-y-1/2 text-blue-600">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor"><path fill-rule="evenodd" d="M10 3a1 1 0 011 1v5h5a1 1 0 110 2h-5v5a1 1 0 11-2 0v-5H4a1 1 0 110-2h5V4a1 1 0 011-1z" clip-rule="evenodd" /></svg>
                    </button>
                </div>
            </div>

            <div id="master-list-view" class="hidden p-4 bg-slate-50 flex-1 overflow-y-auto">
                <div class="flex justify-between items-center mb-2">
                    <h3 id="master-view-title" class="text-[10px] font-bold text-slate-400 uppercase">Painel Master</h3>
                    <button onclick="closeMasterView()" class="text-[10px] text-red-500 font-bold">VOLTAR</button>
                </div>
                <div id="master-items-container"></div>
            </div>

            <div id="contacts-list" class="flex-1 overflow-y-auto p-4 space-y-2">
                <!-- Chats aparecerão aqui automaticamente -->
            </div>
        </aside>

        <main class="flex-1 flex flex-col bg-white relative">
            <div id="chat-welcome" class="flex-1 flex flex-col items-center justify-center p-8 text-center bg-slate-50">
                <div class="w-20 h-20 bg-blue-100 rounded-full flex items-center justify-center mb-4 text-blue-600 animate-bounce-subtle">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-10 w-10" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 12h.01M12 12h.01M16 12h.01M21 12c0 4.418-4.03 8-9 8a9.863 9.863 0 01-4.255-.949L3 20l1.395-3.72C3.512 15.042 3 13.574 3 12c0-4.418 4.03-8 9-8s9 3.582 9 8z" /></svg>
                </div>
                <h3 class="text-xl font-bold text-slate-800">Suas Conversas</h3>
                <p class="text-slate-400 text-sm max-w-xs mt-2">Novas mensagens aparecerão aqui em tempo real com aviso sonoro.</p>
            </div>

            <div id="active-chat" class="hidden flex-1 flex flex-col h-full">
                <header class="p-4 border-b border-slate-100 flex items-center space-x-4 glass">
                    <div id="chat-avatar" class="w-10 h-10 bg-blue-600 rounded-full flex items-center justify-center text-white font-bold shadow-sm">?</div>
                    <div>
                        <h3 id="chat-with-name" class="font-bold text-slate-800">Carregando...</h3>
                        <p class="text-[10px] text-green-500 font-bold uppercase tracking-widest flex items-center">
                            <span class="w-1.5 h-1.5 bg-green-500 rounded-full mr-1.5 animate-pulse"></span> Online Agora
                        </p>
                    </div>
                </header>
                <div id="messages-container" class="flex-1 overflow-y-auto p-6 space-y-4 bg-slate-50/50"></div>
                <footer class="p-4 bg-white border-t border-slate-100">
                    <div class="max-w-4xl mx-auto flex items-center space-x-2">
                        <input type="text" id="msg-input" placeholder="Digite sua mensagem..." class="flex-1 px-4 py-3 rounded-2xl bg-slate-100 border-none focus:ring-2 focus:ring-blue-600 transition">
                        <button onclick="sendMessage()" class="p-3 bg-blue-600 text-white rounded-2xl hover:bg-blue-700 transition shadow-lg shadow-blue-100 active:scale-95">
                            <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 19l9 2-9-18-9 18 9-2zm0 0v-8" /></svg>
                        </button>
                    </div>
                </footer>
            </div>

            <div id="toast-container" class="absolute top-4 right-4 z-50 flex flex-col space-y-2 pointer-events-none"></div>
        </main>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getFirestore, collection, doc, setDoc, getDoc, getDocs, query, orderBy, onSnapshot, addDoc, serverTimestamp, deleteDoc, where } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

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

        let currentUserData = null;
        let activeChatId = null;
        let activeContact = null;
        let unsubscribeMessages = null;
        let isSignUpMode = false;
        let lastKnownRooms = {}; // Para detectar novas mensagens e tocar som

        const MASTER_USER = "CLX";
        const MASTER_PASS = "02072007";

        // Som de notificação (Beep suave gerado via código)
        const notificationSound = new Audio("data:audio/wav;base64,UklGRl9vT19XQVZFZm10IBAAAAABAAEAQB8AAEAfAAABAAgAZGF0YV9vT197777vvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvv");

        window.toggleAuthMode = () => {
            isSignUpMode = !isSignUpMode;
            document.getElementById('auth-title').innerText = isSignUpMode ? "Criar conta CLX" : "Mensageiro CLX";
            document.getElementById('auth-btn').innerText = isSignUpMode ? "Cadastrar" : "Entrar";
            document.getElementById('toggle-auth-btn').innerText = isSignUpMode ? "Já tenho conta" : "Criar conta nova";
            document.getElementById('email-field').classList.toggle('hidden', !isSignUpMode);
        };

        window.handleAuth = async () => {
            const user = document.getElementById('auth-user').value.trim().toLowerCase();
            const pass = document.getElementById('auth-pass').value;
            const email = document.getElementById('auth-email').value.trim();

            if (!user || !pass || (isSignUpMode && !email)) return showToast("Campos vazios!");

            const userRef = doc(db, 'clx_users', user);
            const userDoc = await getDoc(userRef);

            if (isSignUpMode) {
                if (userDoc.exists()) return showToast("Este login já existe!");
                const newId = Math.floor(100000 + Math.random() * 900000).toString();
                const newUser = { username: user, password: pass, email: email, id: newId };
                await setDoc(userRef, newUser);
                currentUserData = newUser;
                initApp();
            } else {
                if (user.toUpperCase() === MASTER_USER && pass === MASTER_PASS) {
                    currentUserData = { username: 'clx', id: '000001' };
                    initApp();
                } else if (userDoc.exists() && userDoc.data().password === pass) {
                    currentUserData = userDoc.data();
                    initApp();
                } else {
                    showToast("Dados incorretos!");
                }
            }
        };

        function initApp() {
            document.getElementById('auth-screen').classList.add('hidden');
            document.getElementById('app-screen').classList.remove('hidden');
            document.getElementById('display-name').innerText = currentUserData.username;
            document.getElementById('display-id').innerText = `ID: #${currentUserData.id}`;
            
            if (currentUserData.username === 'clx') {
                document.getElementById('master-badge').classList.remove('hidden');
            }

            listenToActiveChats();
        }

        function listenToActiveChats() {
            const roomsRef = collection(db, 'clx_active_rooms');
            const q = query(roomsRef, where('participantsIds', 'array-contains', currentUserData.id));

            onSnapshot(q, (snapshot) => {
                const container = document.getElementById('contacts-list');
                container.innerHTML = '';
                
                snapshot.docChanges().forEach(change => {
                    const roomData = change.doc.data();
                    const roomId = change.doc.id;

                    // Se for mensagem nova E não for enviada por mim
                    if (change.type === 'modified' && roomData.lastSenderId !== currentUserData.id) {
                        if (roomId !== activeChatId) {
                            playNotification();
                            notifyUnread(roomData.lastSenderName, roomData.lastMessageText);
                        }
                    }
                });

                snapshot.forEach(docSnap => {
                    const data = docSnap.data();
                    const roomId = docSnap.id;
                    const otherIndex = data.participantsIds.indexOf(currentUserData.id) === 0 ? 1 : 0;
                    const otherName = data.participantsNames ? data.participantsNames[otherIndex] : "Desconhecido";
                    const otherId = data.participantsIds[otherIndex];
                    const isUnread = (roomId !== activeChatId && data.lastSenderId !== currentUserData.id);

                    const btn = document.createElement('button');
                    btn.className = `w-full flex items-center space-x-3 p-3 rounded-xl hover:bg-slate-50 transition text-left group bg-white border ${isUnread ? 'border-blue-400 shadow-md ring-1 ring-blue-100' : 'border-slate-100 shadow-sm'}`;
                    btn.onclick = () => selectChat({ username: otherName, id: otherId }, roomId);
                    btn.innerHTML = `
                        <div class="relative">
                            <div class="w-10 h-10 bg-blue-100 rounded-full flex items-center justify-center text-blue-600 font-bold">${otherName[0].toUpperCase()}</div>
                            ${isUnread ? '<div class="absolute -top-1 -right-1 w-3 h-3 bg-red-500 border-2 border-white rounded-full animate-pulse"></div>' : ''}
                        </div>
                        <div class="flex-1 min-w-0">
                            <p class="font-bold text-slate-800 text-sm uppercase truncate">${otherName}</p>
                            <p class="text-[10px] text-slate-400 truncate">${data.lastMessageText || 'ID: #' + otherId}</p>
                        </div>
                    `;
                    container.appendChild(btn);
                });

                if (snapshot.empty) {
                    container.innerHTML = '<p class="text-[10px] text-center text-slate-400 mt-4 italic">Nenhuma conversa ativa ainda.</p>';
                }
            });
        }

        function playNotification() {
            notificationSound.currentTime = 0;
            notificationSound.play().catch(e => console.log("Áudio bloqueado pelo navegador."));
            
            // Pisca o título da aba
            const originalTitle = document.title;
            document.title = "🔔 NOVA MENSAGEM!";
            setTimeout(() => { document.title = originalTitle; }, 3000);
        }

        function notifyUnread(sender, text) {
            const toastContainer = document.getElementById('toast-container');
            const toast = document.createElement('div');
            toast.className = "notification-toast bg-white border-l-4 border-blue-600 shadow-2xl p-4 rounded-xl flex items-start space-x-3 w-64 pointer-events-auto cursor-pointer";
            toast.onclick = () => toast.remove();
            toast.innerHTML = `
                <div class="w-8 h-8 bg-blue-600 text-white flex items-center justify-center rounded-lg font-bold shrink-0">${sender[0].toUpperCase()}</div>
                <div class="overflow-hidden">
                    <p class="text-[10px] font-bold text-blue-600 uppercase">Nova Mensagem</p>
                    <p class="text-xs font-bold text-slate-800 truncate">${sender.toUpperCase()}</p>
                    <p class="text-[10px] text-slate-500 truncate">${text}</p>
                </div>
            `;
            toastContainer.appendChild(toast);
            setTimeout(() => toast.remove(), 5000);
        }

        window.addContactById = async () => {
            const searchId = document.getElementById('search-id').value.trim();
            if (!searchId || searchId === currentUserData.id) return;

            const q = await getDocs(collection(db, 'clx_users'));
            let target = null;
            q.forEach(d => { if (d.data().id === searchId) target = d.data(); });

            if (target) {
                const tempId = [currentUserData.id, target.id].sort().join('_');
                selectChat({ username: target.username, id: target.id }, tempId);
                document.getElementById('search-id').value = '';
                showToast("Conversa iniciada!");
            } else {
                showToast("ID não encontrado!");
            }
        };

        async function selectChat(contact, chatId) {
            activeContact = contact;
            activeChatId = chatId;
            
            document.getElementById('chat-welcome').classList.add('hidden');
            document.getElementById('active-chat').classList.remove('hidden');
            document.getElementById('chat-with-name').innerText = contact.username.toUpperCase();
            document.getElementById('chat-avatar').innerText = contact.username[0].toUpperCase();

            startMessageListener(activeChatId);
        }

        function startMessageListener(chatId) {
            if (unsubscribeMessages) unsubscribeMessages();
            const q = query(collection(db, 'clx_chats', chatId, 'messages'), orderBy('timestamp', 'asc'));

            unsubscribeMessages = onSnapshot(q, (snapshot) => {
                const container = document.getElementById('messages-container');
                container.innerHTML = '';
                const now = Date.now();

                snapshot.forEach(docSnap => {
                    const m = docSnap.data();
                    const msgTime = m.timestamp ? m.timestamp.toMillis() : now;

                    if (m.timestamp && (now - msgTime > 86400000)) {
                        deleteDoc(docSnap.ref);
                    } else {
                        const isMe = m.senderId === currentUserData.id;
                        const div = document.createElement('div');
                        div.className = `flex ${isMe ? 'justify-end' : 'justify-start'} mb-4 animate-chat`;
                        div.innerHTML = `
                            <div class="max-w-[80%] px-4 py-2 rounded-2xl text-sm ${isMe ? 'bg-blue-600 text-white rounded-tr-none shadow-lg' : 'bg-white text-slate-800 rounded-tl-none border border-slate-100'} shadow-sm">
                                ${!isMe ? `<p class="text-[8px] font-bold opacity-50 uppercase mb-1">${m.senderName}</p>` : ''}
                                <p>${m.text}</p>
                            </div>
                        `;
                        container.appendChild(div);
                    }
                });
                container.scrollTop = container.scrollHeight;
            });
        }

        window.sendMessage = async () => {
            const input = document.getElementById('msg-input');
            const text = input.value.trim();
            if (!text || !activeChatId) return;

            await addDoc(collection(db, 'clx_chats', activeChatId, 'messages'), {
                senderId: currentUserData.id,
                senderName: currentUserData.username,
                text: text,
                timestamp: serverTimestamp()
            });

            await setDoc(doc(db, 'clx_active_rooms', activeChatId), {
                roomId: activeChatId,
                participantsIds: [currentUserData.id, activeContact.id],
                participantsNames: [currentUserData.username, activeContact.username],
                lastUpdate: serverTimestamp(),
                lastMessageText: text,
                lastSenderId: currentUserData.id,
                lastSenderName: currentUserData.username
            }, { merge: true });

            input.value = '';
        };

        window.toggleMasterPanel = async (mode) => {
            const container = document.getElementById('master-items-container');
            document.getElementById('master-list-view').classList.remove('hidden');
            document.getElementById('contacts-list').classList.add('hidden');
            container.innerHTML = '<p class="text-[10px] p-4 animate-pulse">Carregando dados globais...</p>';

            try {
                if (mode === 'users') {
                    document.getElementById('master-view-title').innerText = "Todos os Usuários";
                    const q = await getDocs(collection(db, 'clx_users'));
                    container.innerHTML = '';
                    q.forEach(d => {
                        const data = d.data();
                        const div = document.createElement('div');
                        div.className = "p-2 mb-1 bg-white rounded-lg text-[10px] flex justify-between items-center border border-slate-100 cursor-pointer hover:bg-slate-50";
                        div.innerHTML = `<span class="font-bold">${(data.username || 'S/N').toUpperCase()}</span> <span class="text-slate-400">ID: #${data.id || '000000'}</span>`;
                        div.onclick = () => selectChat({ username: data.username, id: data.id }, [currentUserData.id, data.id].sort().join('_'));
                        container.appendChild(div);
                    });
                } else {
                    document.getElementById('master-view-title').innerText = "Chats do Sistema";
                    const q = await getDocs(collection(db, 'clx_active_rooms'));
                    container.innerHTML = '';
                    
                    q.forEach(d => {
                        const data = d.data();
                        const pNames = data.participantsNames || ["Desconhecido"];
                        const div = document.createElement('div');
                        div.className = "p-2 mb-1 bg-indigo-50 rounded-lg text-[10px] cursor-pointer hover:bg-indigo-100 transition-all border border-indigo-100";
                        div.onclick = () => {
                            activeChatId = d.id;
                            activeContact = { username: pNames.join(' & '), id: 'monitor' };
                            document.getElementById('chat-welcome').classList.add('hidden');
                            document.getElementById('active-chat').classList.remove('hidden');
                            document.getElementById('chat-with-name').innerText = "MONITORANDO: " + pNames.join(' ↔ ').toUpperCase();
                            startMessageListener(d.id);
                        };
                        div.innerHTML = `<p class="font-bold text-indigo-900">${pNames.join(' ↔ ').toUpperCase()}</p>
                                         <p class="text-[8px] text-indigo-400 mt-1">${data.lastMessageText || 'Sem histórico'}</p>`;
                        container.appendChild(div);
                    });
                }
            } catch (error) {
                container.innerHTML = '<p class="text-[10px] text-red-500 p-4">Erro ao carregar painel.</p>';
            }
        };

        window.closeMasterView = () => {
            document.getElementById('master-list-view').classList.add('hidden');
            document.getElementById('contacts-list').classList.remove('hidden');
        };

        function showToast(msg) {
            const t = document.createElement('div');
            t.className = "fixed bottom-10 left-1/2 -translate-x-1/2 bg-slate-800 text-white px-6 py-2 rounded-full text-xs z-[100] animate-chat";
            t.innerText = msg;
            document.body.appendChild(t);
            setTimeout(() => { t.style.opacity='0'; setTimeout(()=>t.remove(), 500); }, 3000);
        }

        document.getElementById('msg-input').addEventListener('keypress', e => { if (e.key === 'Enter') sendMessage(); });

    </script>
</body>
</html>
