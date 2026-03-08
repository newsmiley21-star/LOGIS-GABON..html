<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Logi-Gabon | Gestionnaire de Fret Expert</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body {
            background-color: #f3f4f6;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        .card {
            background: white;
            border-radius: 12px;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
        }
        .btn-primary {
            background-color: #057a55;
            transition: all 0.3s;
        }
        .btn-primary:hover {
            background-color: #046c4e;
            transform: translateY(-1px);
        }
        .preview-badge {
            position: absolute;
            top: -5px;
            right: -5px;
            background: #ef4444;
            color: white;
            border-radius: 50%;
            width: 18px;
            height: 18px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 10px;
            cursor: pointer;
            font-weight: bold;
            box-shadow: 0 2px 4px rgba(0,0,0,0.2);
        }
        .tab-active {
            border-bottom: 4px solid #057a55;
            color: #057a55;
        }
        ::-webkit-scrollbar { width: 6px; height: 6px; }
        ::-webkit-scrollbar-track { background: #f1f1f1; }
        ::-webkit-scrollbar-thumb { background: #057a55; border-radius: 10px; }
        
        .expense-row {
            display: flex;
            justify-content: space-between;
            padding: 4px 0;
            border-bottom: 1px dashed #e5e7eb;
        }
        .expense-row:last-child { border-bottom: none; }

        @keyframes pulse-red {
            0%, 100% { background-color: #fee2e2; }
            50% { background-color: #fecaca; }
        }
        .alert-critical {
            animation: pulse-red 2s infinite;
            border: 1px solid #ef4444;
        }
        #authOverlay {
            background-image: linear-gradient(rgba(0,0,0,0.6), rgba(0,0,0,0.6)), url('https://images.unsplash.com/photo-1586528116311-ad8dd3c8310d?auto=format&fit=crop&q=80&w=2000');
            background-size: cover;
            background-position: center;
        }
    </style>
</head>
<body class="p-4 md:p-8">

    <!-- ÉCRAN DE CONNEXION -->
    <div id="authOverlay" class="fixed inset-0 z-[100] flex items-center justify-center p-4">
        <div class="bg-white p-8 rounded-2xl shadow-2xl w-full max-w-md border-t-8 border-green-700">
            <div class="text-center mb-8">
                <h1 class="text-3xl font-black italic tracking-tighter uppercase text-green-800">Logi-Gabon</h1>
                <p class="text-gray-500 text-sm">Accès sécurisé - Administration</p>
            </div>
            <form id="loginForm" class="space-y-4">
                <div>
                    <label class="block text-xs font-bold uppercase text-gray-400 mb-1">Email professionnel</label>
                    <input type="email" id="loginEmail" required class="w-full p-3 border rounded-xl outline-none focus:ring-2 focus:ring-green-500" placeholder="admin@logigabon.ga">
                </div>
                <div>
                    <label class="block text-xs font-bold uppercase text-gray-400 mb-1">Mot de passe</label>
                    <input type="password" id="loginPassword" required class="w-full p-3 border rounded-xl outline-none focus:ring-2 focus:ring-green-500" placeholder="••••••••">
                </div>
                <button type="submit" class="w-full btn-primary text-white font-bold py-4 rounded-xl shadow-lg uppercase tracking-widest mt-4">Se Connecter</button>
            </form>
            <div id="loginError" class="mt-4 text-center text-red-500 text-xs font-bold hidden"></div>
        </div>
    </div>

    <!-- APPLICATION PRINCIPALE -->
    <div id="appContent" class="hidden max-w-7xl mx-auto">
        <header class="flex flex-col md:flex-row justify-between items-center mb-8 bg-green-800 p-6 rounded-xl text-white shadow-lg">
            <div>
                <h1 class="text-3xl font-bold italic tracking-tighter uppercase">Logi-Gabon</h1>
                <p class="opacity-90 text-sm font-medium">Gestion Transit & Suivi des Dépenses - Libreville / Owendo</p>
            </div>
            <div class="flex items-center gap-4 mt-4 md:mt-0">
                <div id="clock" class="text-lg font-mono bg-green-900 px-4 py-1 rounded-full"></div>
                <button id="logoutBtn" class="bg-red-600 hover:bg-red-700 px-3 py-1 rounded text-xs font-bold uppercase transition-colors">Quitter</button>
            </div>
        </header>

        <!-- STATS -->
        <div class="grid grid-cols-1 md:grid-cols-4 gap-4 mb-8">
            <div class="card p-4 border-l-4 border-blue-500">
                <p class="text-[10px] font-black text-gray-400 uppercase">Chiffre d'Affaires Global</p>
                <h3 id="statsCA" class="text-xl font-black text-gray-800">0 FCFA</h3>
            </div>
            <div class="card p-4 border-l-4 border-green-500">
                <p class="text-[10px] font-black text-gray-400 uppercase">Marge Totale Réalisée</p>
                <h3 id="statsMarge" class="text-xl font-black text-green-700">0 FCFA</h3>
            </div>
            <div class="card p-4 border-l-4 border-orange-500">
                <p class="text-[10px] font-black text-gray-400 uppercase">Rentabilité Moyenne</p>
                <h3 id="statsRentabilite" class="text-xl font-black text-orange-600">0%</h3>
            </div>
            <div class="card p-4 border-l-4 border-purple-500">
                <p class="text-[10px] font-black text-gray-400 uppercase">Nombre de Dossiers</p>
                <h3 id="statsDossiers" class="text-xl font-black text-purple-700">0</h3>
            </div>
        </div>

        <div class="grid grid-cols-1 lg:grid-cols-3 gap-8 mb-8">
            <section class="card p-6 lg:col-span-2 relative">
                <div id="editModeIndicator" class="hidden absolute top-4 right-6 bg-orange-100 text-orange-700 px-3 py-1 rounded-full text-xs font-bold animate-pulse uppercase">Mode Édition</div>
                <h2 id="formTitle" class="text-xl font-bold mb-6 border-b pb-2 flex items-center text-gray-800">📝 Nouveau Dossier</h2>
                
                <form id="logiForm" class="grid grid-cols-1 md:grid-cols-2 gap-6">
                    <div class="space-y-4 border-r pr-0 md:pr-6">
                        <h3 class="text-xs font-black text-green-700 uppercase tracking-widest text-[10px]">Acheminement</h3>
                        <select id="carrier" required class="w-full p-2 border rounded mt-1 bg-white outline-none focus:ring-2 focus:ring-green-500">
                            <option value="">Choisir...</option>
                            <option value="CMA CGM">CMA CGM</option>
                            <option value="Maersk Line">Maersk Line</option>
                            <option value="MSC">MSC</option>
                            <option value="Air France Cargo">Air France Cargo</option>
                        </select>
                        <div class="grid grid-cols-2 gap-2">
                            <input type="text" id="blNumber" placeholder="N° BL / LTA" required class="w-full p-2 border rounded outline-none focus:ring-2 focus:ring-green-500">
                            <input type="text" id="containerNumber" placeholder="N° Conteneur" required class="w-full p-2 border rounded outline-none focus:ring-2 focus:ring-green-500">
                        </div>
                        <input type="text" id="itemName" placeholder="Nature Marchandise" required class="w-full p-2 border rounded outline-none focus:ring-2 focus:ring-green-500">
                        <div class="grid grid-cols-3 gap-2">
                            <input type="number" id="long" placeholder="L (cm)" required oninput="updateCalculations()" class="w-full p-2 border rounded">
                            <input type="number" id="larg" placeholder="l (cm)" required oninput="updateCalculations()" class="w-full p-2 border rounded">
                            <input type="number" id="haut" placeholder="h (cm)" required oninput="updateCalculations()" class="w-full p-2 border rounded">
                        </div>
                    </div>

                    <div class="space-y-4">
                        <h3 class="text-xs font-black text-green-700 uppercase tracking-widest text-[10px]">Client & Finance (FCFA)</h3>
                        <div class="grid grid-cols-2 gap-2">
                            <input type="text" id="clientName" placeholder="Client" required class="w-full p-2 border rounded">
                            <input type="tel" id="clientPhone" placeholder="Téléphone" required class="w-full p-2 border rounded">
                        </div>
                        <div class="grid grid-cols-2 gap-2">
                            <input type="number" id="buyPrice" placeholder="Achat" required oninput="updateCalculations()" class="w-full p-2 border rounded text-xs">
                            <input type="number" id="shipping" placeholder="Fret" required oninput="updateCalculations()" class="w-full p-2 border rounded text-xs">
                        </div>
                        <div class="grid grid-cols-2 gap-2">
                            <input type="number" id="otherFees" value="0" placeholder="Autres LBV" oninput="updateCalculations()" class="w-full p-2 border rounded text-xs">
                            <input type="number" id="sellPrice" placeholder="Prix Vente" required oninput="updateCalculations()" class="w-full p-2 border rounded font-bold bg-green-50">
                        </div>
                        <div class="p-3 bg-gray-50 border-2 border-dashed rounded-lg">
                            <label class="block text-xs font-black text-gray-600 mb-2 uppercase italic">Pièces Jointes</label>
                            <input type="file" id="docPhotos" accept="image/*" multiple class="text-[10px] w-full mb-3 cursor-pointer">
                            <div id="multiPreviewContainer" class="flex flex-wrap gap-2 min-h-[40px]"></div>
                        </div>
                        <div class="flex gap-2">
                            <button type="submit" id="submitBtn" class="flex-1 btn-primary text-white font-black py-4 rounded-xl shadow-lg uppercase text-sm">Enregistrer</button>
                            <button type="button" id="cancelBtn" class="hidden bg-gray-200 hover:bg-gray-300 text-gray-700 font-bold py-4 px-6 rounded-xl uppercase text-xs">Annuler</button>
                        </div>
                    </div>
                </form>
            </section>

            <section class="space-y-6">
                <div id="financeCard" class="card p-6 border-t-8 border-green-600">
                    <h2 class="text-lg font-bold mb-4 text-gray-800 flex justify-between">
                        <span>📊 Détail Financier</span>
                        <span id="alertBadge" class="hidden px-2 py-1 rounded text-[10px] font-black uppercase"></span>
                    </h2>
                    <div id="analysisContent" class="space-y-3 text-[13px]">
                        <div class="expense-row"><span>Volume Chargement</span><span id="resCBM" class="font-bold text-blue-700">0.000 m³</span></div>
                        <div class="expense-row"><span>Achat Marchandise</span><span id="resBuy">0 FCFA</span></div>
                        <div class="expense-row"><span>Fret Maritime/Aérien</span><span id="resShip">0 FCFA</span></div>
                        <div class="expense-row text-red-500"><span>Douane Estimée (45%)</span><span id="resTax" class="font-bold">0 FCFA</span></div>
                        <div class="expense-row font-black pt-2 border-t"><span>Coût de revient</span><span id="resTotalCost">0 FCFA</span></div>
                        <div class="py-4 mt-2 text-center border-2 rounded-xl bg-gray-50 border-dashed">
                            <p class="text-[10px] font-bold text-gray-400 uppercase">Marge Nette Prévue</p>
                            <span id="resMargin" class="text-2xl font-black text-gray-400">0 FCFA</span>
                        </div>
                    </div>
                </div>
            </section>
        </div>

        <!-- REGISTRE -->
        <div class="card shadow-2xl overflow-hidden">
            <div class="p-4 bg-gray-800 flex flex-col md:flex-row justify-between items-center gap-4">
                <div class="flex space-x-6 text-sm font-bold uppercase text-gray-400">
                    <button id="tabActive" class="pb-2 tab-active">En cours</button>
                    <button id="tabArchive" class="pb-2">Archives</button>
                </div>
                <input type="text" id="searchInput" placeholder="Rechercher..." class="w-full md:w-96 px-4 py-2 rounded-full bg-gray-700 text-white text-sm outline-none">
            </div>
            <div class="overflow-x-auto">
                <table class="w-full text-left">
                    <thead>
                        <tr class="bg-gray-100 text-[10px] uppercase text-gray-500 font-black border-b">
                            <th class="p-4">Transport</th>
                            <th class="p-4">Marchandise</th>
                            <th class="p-4">Client</th>
                            <th class="p-4">Finance (FCFA)</th>
                            <th class="p-4 text-right">Action</th>
                        </tr>
                    </thead>
                    <tbody id="historyBody" class="text-xs divide-y"></tbody>
                </table>
            </div>
        </div>
    </div>

    <!-- Firebase SDKs -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInWithEmailAndPassword, onAuthStateChanged, signOut, signInWithCustomToken, signInAnonymously } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, setDoc, collection, query, onSnapshot, deleteDoc, updateDoc, addDoc } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        const firebaseConfig = JSON.parse(__firebase_config);
        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'logigabon-default';

        let currentUser = null;
        let currentTab = 'active';
        let historyData = [];
        let archivesData = [];
        let tempImages = [];
        let editingDossierId = null;

        const DOSSIER_FEE = 65000;

        // Init Auth
        const initAuth = async () => {
            if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                try { await signInWithCustomToken(auth, __initial_auth_token); } catch (e) {}
            } else {
                try { await signInAnonymously(auth); } catch (e) {}
            }
        };
        initAuth();

        onAuthStateChanged(auth, (user) => {
            if (user) {
                currentUser = user;
                document.getElementById('authOverlay').classList.add('hidden');
                document.getElementById('appContent').classList.remove('hidden');
                setupFirestoreListeners(user.uid);
            } else {
                document.getElementById('authOverlay').classList.remove('hidden');
                document.getElementById('appContent').classList.add('hidden');
            }
        });

        // Listeners Firestore
        function setupFirestoreListeners(uid) {
            const activeCol = collection(db, 'artifacts', appId, 'users', uid, 'active');
            const archiveCol = collection(db, 'artifacts', appId, 'users', uid, 'archive');

            onSnapshot(activeCol, (snap) => {
                historyData = snap.docs.map(d => ({ id: d.id, ...d.data() }));
                renderHistory();
            }, (err) => console.error(err));

            onSnapshot(archiveCol, (snap) => {
                archivesData = snap.docs.map(d => ({ id: d.id, ...d.data() }));
                renderHistory();
            }, (err) => console.error(err));
        }

        // Form Submission
        document.getElementById('logiForm').addEventListener('submit', async (e) => {
            e.preventDefault();
            if (!currentUser) return;

            const calc = updateCalculations();
            const dossierData = {
                carrier: document.getElementById('carrier').value,
                bl: document.getElementById('blNumber').value,
                container: document.getElementById('containerNumber').value,
                name: document.getElementById('itemName').value,
                client: document.getElementById('clientName').value,
                phone: document.getElementById('clientPhone').value,
                dims: calc.dims,
                finance: { buy: calc.buy, ship: calc.ship, other: calc.other, sell: calc.sell },
                margin: calc.margin,
                images: tempImages,
                timestamp: Date.now()
            };

            const path = collection(db, 'artifacts', appId, 'users', currentUser.uid, 'active');
            
            try {
                if (editingDossierId) {
                    await updateDoc(doc(db, 'artifacts', appId, 'users', currentUser.uid, 'active', editingDossierId), dossierData);
                } else {
                    await addDoc(path, dossierData);
                }
                resetForm();
            } catch (err) {
                console.error("Save error", err);
            }
        });

        window.updateCalculations = () => {
            const l = parseFloat(document.getElementById('long').value) || 0;
            const w = parseFloat(document.getElementById('larg').value) || 0;
            const h = parseFloat(document.getElementById('haut').value) || 0;
            const buy = parseFloat(document.getElementById('buyPrice').value) || 0;
            const ship = parseFloat(document.getElementById('shipping').value) || 0;
            const other = parseFloat(document.getElementById('otherFees').value) || 0;
            const sell = parseFloat(document.getElementById('sellPrice').value) || 0;

            const cbm = (l * w * h) / 1000000;
            const tax = (buy + ship) * 0.45;
            const totalCost = buy + ship + tax + other + DOSSIER_FEE;
            const margin = sell - totalCost;

            document.getElementById('resCBM').innerText = cbm.toFixed(3) + " m³";
            document.getElementById('resTotalCost').innerText = Math.round(totalCost).toLocaleString() + " FCFA";
            document.getElementById('resMargin').innerText = Math.round(margin).toLocaleString() + " FCFA";
            document.getElementById('resMargin').className = `text-2xl font-black ${margin < 0 ? 'text-red-500' : 'text-green-700'}`;
            
            return { dims: {l, w, h}, buy, ship, other, sell, margin };
        };

        function renderHistory() {
            const body = document.getElementById('historyBody');
            const data = currentTab === 'active' ? historyData : archivesData;
            body.innerHTML = data.map(item => `
                <tr class="hover:bg-gray-50 transition-colors">
                    <td class="p-4 font-bold">${item.carrier}<br><span class="text-[10px] text-gray-400">#${item.bl}</span></td>
                    <td class="p-4">${item.name}</td>
                    <td class="p-4">${item.client}<br><span class="text-[10px]">${item.phone}</span></td>
                    <td class="p-4 font-black ${item.margin >= 0 ? 'text-green-700' : 'text-red-600'}">${item.finance?.sell?.toLocaleString()} FCFA</td>
                    <td class="p-4 text-right space-x-2">
                        <button onclick="window.startEdit('${item.id}')" class="text-blue-600 font-bold">Modif</button>
                        <button onclick="window.deleteItem('${item.id}')" class="text-red-600 font-bold">X</button>
                    </td>
                </tr>
            `).join('');
            updateDashboard();
        }

        function updateDashboard() {
            const all = [...historyData, ...archivesData];
            const ca = all.reduce((s, i) => s + (i.finance?.sell || 0), 0);
            const margin = all.reduce((s, i) => s + (i.margin || 0), 0);
            document.getElementById('statsCA').innerText = ca.toLocaleString() + " FCFA";
            document.getElementById('statsMarge').innerText = margin.toLocaleString() + " FCFA";
            document.getElementById('statsDossiers').innerText = all.length;
        }

        window.startEdit = (id) => {
            const dossier = historyData.find(i => i.id === id);
            if (!dossier) return;
            editingDossierId = id;
            document.getElementById('carrier').value = dossier.carrier;
            document.getElementById('blNumber').value = dossier.bl;
            document.getElementById('containerNumber').value = dossier.container;
            document.getElementById('itemName').value = dossier.name;
            document.getElementById('clientName').value = dossier.client;
            document.getElementById('clientPhone').value = dossier.phone;
            document.getElementById('long').value = dossier.dims?.l || 0;
            document.getElementById('larg').value = dossier.dims?.w || 0;
            document.getElementById('haut').value = dossier.dims?.h || 0;
            document.getElementById('buyPrice').value = dossier.finance?.buy || 0;
            document.getElementById('shipping').value = dossier.finance?.ship || 0;
            document.getElementById('otherFees').value = dossier.finance?.other || 0;
            document.getElementById('sellPrice').value = dossier.finance?.sell || 0;
            tempImages = dossier.images || [];
            
            document.getElementById('editModeIndicator').classList.remove('hidden');
            document.getElementById('cancelBtn').classList.remove('hidden');
            document.getElementById('submitBtn').innerText = "Mettre à jour";
            updateCalculations();
        };

        window.deleteItem = async (id) => {
            if (confirm("Supprimer ce dossier ?") && currentUser) {
                await deleteDoc(doc(db, 'artifacts', appId, 'users', currentUser.uid, currentTab, id));
            }
        };

        function resetForm() {
            editingDossierId = null;
            document.getElementById('logiForm').reset();
            document.getElementById('editModeIndicator').classList.add('hidden');
            document.getElementById('cancelBtn').classList.add('hidden');
            document.getElementById('submitBtn').innerText = "Enregistrer";
            tempImages = [];
            updateCalculations();
        }

        document.getElementById('loginForm').addEventListener('submit', async (e) => {
            e.preventDefault();
            const email = document.getElementById('loginEmail').value;
            const pass = document.getElementById('loginPassword').value;
            try {
                await signInWithEmailAndPassword(auth, email, pass);
            } catch (err) {
                const errEl = document.getElementById('loginError');
                errEl.innerText = "Erreur de connexion.";
                errEl.classList.remove('hidden');
            }
        });

        document.getElementById('logoutBtn').addEventListener('click', () => signOut(auth));
        document.getElementById('cancelBtn').addEventListener('click', resetForm);
        document.getElementById('tabActive').addEventListener('click', () => { currentTab = 'active'; renderHistory(); });
        document.getElementById('tabArchive').addEventListener('click', () => { currentTab = 'archive'; renderHistory(); });
        
        setInterval(() => document.getElementById('clock').innerText = new Date().toLocaleString('fr-FR'), 1000);
    </script>
</body>
</html>
