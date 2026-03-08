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
        /* Custom Scrollbar */
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
        /* Auth Overlay */
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

    <!-- APPLICATION PRINCIPALE (Masquée par défaut) -->
    <div id="appContent" class="hidden max-w-7xl mx-auto">
        <!-- Header -->
        <header class="flex flex-col md:flex-row justify-between items-center mb-8 bg-green-800 p-6 rounded-xl text-white shadow-lg">
            <div>
                <h1 class="text-3xl font-bold italic tracking-tighter uppercase">Logi-Gabon</h1>
                <p class="opacity-90 text-sm font-medium">Gestion Transit & Suivi des Dépenses - Libreville / Owendo</p>
            </div>
            <div class="flex items-center gap-4 mt-4 md:mt-0">
                <div id="clock" class="text-lg font-mono bg-green-900 px-4 py-1 rounded-full"></div>
                <button onclick="logout()" class="bg-red-600 hover:bg-red-700 px-3 py-1 rounded text-xs font-bold uppercase transition-colors">Quitter</button>
            </div>
        </header>

        <!-- DASHBOARD COMPTABILITÉ -->
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
            
            <!-- Formulaire de Saisie / Modification -->
            <section class="card p-6 lg:col-span-2 relative">
                <div id="editModeIndicator" class="hidden absolute top-4 right-6 bg-orange-100 text-orange-700 px-3 py-1 rounded-full text-xs font-bold animate-pulse uppercase">
                    Mode Édition
                </div>
                
                <h2 id="formTitle" class="text-xl font-bold mb-6 border-b pb-2 flex items-center text-gray-800">
                    <span class="mr-2">📝</span> Nouveau Dossier Transit
                </h2>
                
                <form id="logiForm" class="grid grid-cols-1 md:grid-cols-2 gap-6">
                    <!-- Transport -->
                    <div class="space-y-4 border-r pr-0 md:pr-6">
                        <h3 class="text-xs font-black text-green-700 uppercase tracking-widest text-[10px]">Acheminement & Dimensions</h3>
                        
                        <div>
                            <label class="block text-sm font-semibold text-gray-700">Compagnie / Armateur</label>
                            <select id="carrier" required class="w-full p-2 border rounded mt-1 bg-white outline-none focus:ring-2 focus:ring-green-500">
                                <option value="">Choisir...</option>
                                <optgroup label="Maritime">
                                    <option value="CMA CGM">CMA CGM</option>
                                    <option value="Maersk Line">Maersk Line</option>
                                    <option value="MSC">MSC</option>
                                    <option value="Bolloré / AGL">Bolloré / AGL</option>
                                </optgroup>
                                <optgroup label="Aérien">
                                    <option value="Air France Cargo">Air France Cargo</option>
                                    <option value="Ethiopian Cargo">Ethiopian Cargo</option>
                                    <option value="DHL Aviation">DHL Aviation</option>
                                </optgroup>
                                <option value="Autre">Autre...</option>
                            </select>
                        </div>

                        <div class="grid grid-cols-2 gap-2">
                            <div>
                                <label class="block text-sm font-semibold text-gray-700">N° BL / LTA</label>
                                <input type="text" id="blNumber" required class="w-full p-2 border rounded mt-1 outline-none focus:ring-2 focus:ring-green-500">
                            </div>
                            <div>
                                <label class="block text-sm font-semibold text-gray-700">N° Conteneur</label>
                                <input type="text" id="containerNumber" required class="w-full p-2 border rounded mt-1 outline-none focus:ring-2 focus:ring-green-500">
                            </div>
                        </div>

                        <div>
                            <label class="block text-sm font-semibold text-gray-700">Nature Marchandise</label>
                            <input type="text" id="itemName" required class="w-full p-2 border rounded mt-1 outline-none focus:ring-2 focus:ring-green-500">
                        </div>

                        <div class="grid grid-cols-3 gap-2">
                            <div>
                                <label class="text-[10px] font-bold text-gray-500 uppercase">Long (cm)</label>
                                <input type="number" id="long" required oninput="updateCalculations()" class="w-full p-2 border rounded">
                            </div>
                            <div>
                                <label class="text-[10px] font-bold text-gray-500 uppercase">Larg (cm)</label>
                                <input type="number" id="larg" required oninput="updateCalculations()" class="w-full p-2 border rounded">
                            </div>
                            <div>
                                <label class="text-[10px] font-bold text-gray-500 uppercase">Haut (cm)</label>
                                <input type="number" id="haut" required oninput="updateCalculations()" class="w-full p-2 border rounded">
                            </div>
                        </div>
                    </div>

                    <!-- Client & Finance -->
                    <div class="space-y-4">
                        <h3 class="text-xs font-black text-green-700 uppercase tracking-widest text-[10px]">Client & Dépenses (FCFA)</h3>
                        
                        <div class="grid grid-cols-2 gap-2">
                            <div>
                                <label class="block text-sm font-semibold text-gray-700">Client</label>
                                <input type="text" id="clientName" required class="w-full p-2 border rounded mt-1">
                            </div>
                            <div>
                                <label class="block text-sm font-semibold text-gray-700">Téléphone</label>
                                <input type="tel" id="clientPhone" required class="w-full p-2 border rounded mt-1">
                            </div>
                        </div>

                        <div class="grid grid-cols-2 gap-2">
                            <div>
                                <label class="block text-sm font-semibold text-gray-700">Achat Fournisseur</label>
                                <input type="number" id="buyPrice" required oninput="updateCalculations()" class="w-full p-2 border rounded mt-1 text-xs">
                            </div>
                            <div>
                                <label class="block text-sm font-semibold text-gray-700">Fret / Transport</label>
                                <input type="number" id="shipping" required oninput="updateCalculations()" class="w-full p-2 border rounded mt-1 text-xs">
                            </div>
                        </div>
                        
                        <div class="grid grid-cols-2 gap-2">
                            <div>
                                <label class="block text-sm font-semibold text-gray-700">Autres Frais LBV</label>
                                <input type="number" id="otherFees" value="0" oninput="updateCalculations()" class="w-full p-2 border rounded mt-1 text-xs">
                            </div>
                            <div>
                                <label class="block text-sm font-semibold text-gray-700">Prix de Vente</label>
                                <input type="number" id="sellPrice" required oninput="updateCalculations()" class="w-full p-2 border rounded mt-1 font-bold bg-green-50">
                            </div>
                        </div>

                        <!-- MULTI DOCUMENTS SECTION -->
                        <div class="p-3 bg-gray-50 border-2 border-dashed rounded-lg">
                            <label class="block text-xs font-black text-gray-600 mb-2 uppercase tracking-tighter italic">Pièces Jointes (Photos/Scans)</label>
                            <input type="file" id="docPhotos" accept="image/*" multiple class="text-[10px] w-full mb-3 cursor-pointer">
                            <div id="multiPreviewContainer" class="flex flex-wrap gap-2 min-h-[40px]"></div>
                        </div>

                        <div class="flex gap-2">
                            <button type="submit" id="submitBtn" class="flex-1 btn-primary text-white font-black py-4 rounded-xl shadow-lg uppercase text-sm">Enregistrer</button>
                            <button type="button" id="cancelBtn" onclick="resetForm()" class="hidden bg-gray-200 hover:bg-gray-300 text-gray-700 font-bold py-4 px-6 rounded-xl uppercase text-xs">Annuler</button>
                        </div>
                    </div>
                </form>
            </section>

            <!-- Résumé Rapide & Alertes Comptables -->
            <section class="space-y-6">
                <div id="financeCard" class="card p-6 border-t-8 border-green-600 shadow-xl transition-all duration-500">
                    <h2 class="text-lg font-bold mb-4 text-gray-800 flex items-center justify-between">
                        <span>📊 Détail Financier</span>
                        <span id="alertBadge" class="hidden px-2 py-1 rounded text-[10px] font-black uppercase"></span>
                    </h2>
                    
                    <div id="analysisContent" class="space-y-3 text-[13px]">
                        <div class="expense-row">
                            <span class="text-gray-500 italic">Volume Chargement</span>
                            <span id="resCBM" class="font-bold text-blue-700">0.000 m³</span>
                        </div>
                        
                        <div class="pt-2">
                            <p class="text-[10px] font-black text-gray-400 uppercase mb-2">Décompte des Dépenses</p>
                            <div class="expense-row"><span>Achat Marchandise</span><span id="resBuy">0 FCFA</span></div>
                            <div class="expense-row"><span>Fret Maritime/Aérien</span><span id="resShip">0 FCFA</span></div>
                            <div class="expense-row text-red-500"><span>Douane Estimée (45%)</span><span id="resTax" class="font-bold">0 FCFA</span></div>
                            <div class="expense-row"><span>Logistique / Port</span><span id="resOther">0 FCFA</span></div>
                            <div class="expense-row text-green-700 font-bold"><span>Frais Dossier Fixes</span><span>65 000 FCFA</span></div>
                        </div>

                        <div class="flex justify-between font-black pt-2 border-t border-gray-300 uppercase text-[11px] text-gray-700">
                            <span>Coût de revient total</span>
                            <span id="resTotalCost">0 FCFA</span>
                        </div>

                        <div class="py-4 mt-2 text-center border-2 rounded-xl bg-gray-50 border-dashed relative">
                            <p class="text-[10px] font-bold text-gray-400 uppercase">Marge Nette Prévue</p>
                            <span id="resMargin" class="text-2xl font-black text-gray-400">0 FCFA</span>
                            <div id="marginPercent" class="text-[10px] font-bold text-gray-500 mt-1">0% du prix de vente</div>
                        </div>
                        
                        <!-- SOLUTION / CONSEIL COMPTABLE -->
                        <div id="accountingAdvice" class="hidden mt-4 p-3 bg-white border border-gray-200 rounded-lg text-[11px] italic leading-snug shadow-inner">
                            <!-- JS Injection -->
                        </div>
                    </div>
                </div>
            </section>
        </div>

        <!-- Section Registre & Recherche -->
        <div class="card shadow-2xl overflow-hidden">
            <div class="p-4 bg-gray-800 flex flex-col md:flex-row justify-between items-center gap-4">
                <div class="flex space-x-6 text-sm font-bold uppercase text-gray-400">
                    <button id="tabActive" onclick="switchTab('active')" class="pb-2 tab-active transition-all">Dossiers en cours</button>
                    <button id="tabArchive" onclick="switchTab('archive')" class="pb-2 hover:text-white transition-all">Archives Terminées</button>
                </div>
                
                <div class="relative w-full md:w-96">
                    <input type="text" id="searchInput" onkeyup="filterHistory()" placeholder="Rechercher Dossier..." 
                           class="w-full pl-10 pr-4 py-2 rounded-full bg-gray-700 text-white text-sm outline-none border-none">
                    <span class="absolute left-3 top-2.5">🔍</span>
                </div>
            </div>

            <div class="overflow-x-auto">
                <table class="w-full text-left border-collapse">
                    <thead>
                        <tr class="bg-gray-100 text-[10px] uppercase text-gray-500 font-black border-b">
                            <th class="p-4">Transport / Dossier</th>
                            <th class="p-4">Marchandise</th>
                            <th class="p-4">Client</th>
                            <th class="p-4">Statut Comptable</th>
                            <th class="p-4">Finance (FCFA)</th>
                            <th class="p-4 text-right">Action</th>
                        </tr>
                    </thead>
                    <tbody id="historyBody" class="text-xs divide-y"></tbody>
                </table>
            </div>
            <div id="noResult" class="hidden p-20 text-center text-gray-400 italic">Aucun dossier trouvé...</div>
        </div>
    </div>

    <!-- Modal Visionneuse Galerie -->
    <div id="galleryModal" class="fixed inset-0 bg-black bg-opacity-95 hidden z-[110] flex flex-col items-center justify-center p-4">
        <button onclick="closeGallery()" class="absolute top-6 right-6 text-white text-4xl font-bold hover:text-red-500 transition-colors">&times;</button>
        <div class="relative w-full max-w-4xl flex items-center justify-center">
            <button onclick="prevImg()" class="absolute left-0 md:-left-20 bg-white bg-opacity-20 hover:bg-opacity-40 text-white p-4 rounded-full transition-all">❮</button>
            <img id="modalImg" src="" class="max-w-full max-h-[80vh] rounded shadow-2xl transition-all duration-300">
            <button onclick="nextImg()" class="absolute right-0 md:-right-20 bg-white bg-opacity-20 hover:bg-opacity-40 text-white p-4 rounded-full transition-all">❯</button>
        </div>
        <p id="imgCounter" class="text-white mt-6 font-mono text-sm bg-gray-800 px-4 py-1 rounded-full"></p>
    </div>

    <!-- Firebase SDKs -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInWithEmailAndPassword, onAuthStateChanged, signOut, signInWithCustomToken, signInAnonymously } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, setDoc, getDoc, collection, query, onSnapshot, deleteDoc, updateDoc } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        const firebaseConfig = JSON.parse(__firebase_config);
        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'logigabon-default';

        // Constantes
        const DOSSIER_FEE = 65000;

        // État Global
        let currentUser = null;
        let currentTab = 'active';
        let historyData = [];
        let archivesData = [];
        let tempImages = []; 
        let editingDossierId = null; 
        let currentGallery = [];
        let currentImgIdx = 0;

        // DOM Elements
        const authOverlay = document.getElementById('authOverlay');
        const appContent = document.getElementById('appContent');
        const loginForm = document.getElementById('loginForm');
        const loginError = document.getElementById('loginError');
        const form = document.getElementById('logiForm');
        const fileInput = document.getElementById('docPhotos');
        const multiPreview = document.getElementById('multiPreviewContainer');
        const searchInput = document.getElementById('searchInput');
        const submitBtn = document.getElementById('submitBtn');
        const cancelBtn = document.getElementById('cancelBtn');
        const formTitle = document.getElementById('formTitle');

        // --- AUTHENTIFICATION ---

        const initAuth = async () => {
            if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                try {
                    await signInWithCustomToken(auth, __initial_auth_token);
                } catch (e) {
                    // Fallback invisible
                }
            }
        };
        initAuth();

        onAuthStateChanged(auth, (user) => {
            if (user) {
                currentUser = user;
                authOverlay.classList.add('hidden');
                appContent.classList.remove('hidden');
                setupFirestoreListeners(user.uid);
            } else {
                currentUser = null;
                authOverlay.classList.remove('hidden');
                appContent.classList.add('hidden');
            }
        });

        loginForm.addEventListener('submit', async (e) => {
            e.preventDefault();
            loginError.classList.add('hidden');
            const email = document.getElementById('loginEmail').value;
            const pass = document.getElementById('loginPassword').value;

            try {
                await signInWithEmailAndPassword(auth, email, pass);
            } catch (error) {
                loginError.innerText = "Accès refusé : Vérifiez vos identifiants.";
                loginError.classList.remove('hidden');
            }
        });

        window.logout = () => {
            signOut(auth);
        };

        // --- FIRESTORE LOGIC ---

        function setupFirestoreListeners(userId) {
            const activeCol = collection(db, 'artifacts', appId, 'users', userId, 'active');
            const archiveCol = collection(db, 'artifacts', appId, 'users', userId, 'archive');

            // Listen to Active
            onSnapshot(activeCol, (snapshot) => {
                historyData = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
                renderHistory();
            }, (err) => console.error("Error reading active", err));

            // Listen to Archive
            onSnapshot(archiveCol, (snapshot) => {
                archivesData = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
                renderHistory();
            }, (err) => console.error("Error reading archive", err));
        }

        // --- APP LOGIC ---

        setInterval(() => document.getElementById('clock').innerText = new Date().toLocaleString('fr-FR'), 1000);

        function formatFCFA(num) {
            return new Intl.NumberFormat('fr-FR').format(Math.round(num));
        }

        window.updateCalculations = () => {
            const l = parseFloat(document.getElementById('long').value) || 0;
            const w = parseFloat(document.getElementById('larg').value) || 0;
            const h = parseFloat(document.getElementById('haut').value) || 0;
            const buy = parseFloat(document.getElementById('buyPrice').value) || 0;
            const ship = parseFloat(document.getElementById('shipping').value) || 0;
            const other = parseFloat(document.getElementById('otherFees').value) || 0;
            const sell = parseFloat(document.getElementById('sellPrice').value) || 0;

            const cbm = (l * w * h) / 1000000;
            const caf = buy + ship;
            const tax = caf * 0.45;
            const totalCost = caf + tax + other + DOSSIER_FEE;
            const margin = sell - totalCost;
            const marginPerc = sell > 0 ? (margin / sell) * 100 : 0;

            document.getElementById('resCBM').innerText = cbm.toFixed(3) + " m³";
            document.getElementById('resBuy').innerText = formatFCFA(buy) + " FCFA";
            document.getElementById('resShip').innerText = formatFCFA(ship) + " FCFA";
            document.getElementById('resTax').innerText = formatFCFA(tax) + " FCFA";
            document.getElementById('resOther').innerText = formatFCFA(other) + " FCFA";
            document.getElementById('resTotalCost').innerText = formatFCFA(totalCost) + " FCFA";
            
            const marginEl = document.getElementById('resMargin');
            const percentEl = document.getElementById('marginPercent');
            const financeCard = document.getElementById('financeCard');
            const alertBadge = document.getElementById('alertBadge');
            const adviceBox = document.getElementById('accountingAdvice');

            marginEl.innerText = formatFCFA(margin) + " FCFA";
            percentEl.innerText = marginPerc.toFixed(1) + "% du prix de vente";

            financeCard.classList.remove('alert-critical', 'border-orange-500');
            alertBadge.classList.add('hidden');
            adviceBox.classList.add('hidden');

            if (sell > 0) {
                adviceBox.classList.remove('hidden');
                if (margin < 0) {
                    financeCard.classList.add('alert-critical');
                    alertBadge.innerText = "DANGER : Déficit";
                    alertBadge.className = "px-2 py-1 rounded text-[10px] font-black uppercase bg-red-600 text-white";
                    alertBadge.classList.remove('hidden');
                    adviceBox.innerHTML = `⚠️ <strong>Solution :</strong> Déficit détecté. Revisez le prix à min. <strong>${formatFCFA(totalCost * 1.15)} FCFA</strong>.`;
                } else if (marginPerc < 10) {
                    financeCard.classList.add('border-orange-500');
                    alertBadge.innerText = "ALERTE : Faible Marge";
                    alertBadge.className = "px-2 py-1 rounded text-[10px] font-black uppercase bg-orange-500 text-white";
                    alertBadge.classList.remove('hidden');
                    adviceBox.innerHTML = `💡 <strong>Conseil :</strong> Rentabilité faible (${marginPerc.toFixed(1)}%). Réajustez vos frais.`;
                } else {
                    alertBadge.innerText = "SANTÉ : Excellente";
                    alertBadge.className = "px-2 py-1 rounded text-[10px] font-black uppercase bg-green-600 text-white";
                    alertBadge.classList.remove('hidden');
                    adviceBox.innerHTML = `✅ <strong>Analyse :</strong> Marge opérationnelle valide.`;
                }
            }
            
            return { cbm, tax, totalCost, margin, marginPerc, buy, ship, other, sell, dims: {l, w, h} };
        }

        function updateDashboard() {
            const list = [...historyData, ...archivesData];
            const totalCA = list.reduce((sum, item) => sum + (item.finance?.sell || 0), 0);
            const totalMarge = list.reduce((sum, item) => sum + (item.margin || 0), 0);
            const avgRent = totalCA > 0 ? (totalMarge / totalCA) * 100 : 0;

            document.getElementById('statsCA').innerText = formatFCFA(totalCA) + " FCFA";
            document.getElementById('statsMarge').innerText = formatFCFA(totalMarge) + " FCFA";
            document.getElementById('statsMarge').className = `text-xl font-black ${totalMarge >= 0 ? 'text-green-700' : 'text-red-600'}`;
            document.getElementById('statsRentabilite').innerText = avgRent.toFixed(1) + "%";
            document.getElementById('statsDossiers').innerText = list.length;
        }

        fileInput.addEventListener('change', function() {
            const files = Array.from(this.files);
            files.forEach(file => {
                const reader = new FileReader();
                reader.onload = e => {
                    tempImages.push(e.target.result);
                    renderTempPreviews();
                }
                reader.readAsDataURL(file);
            });
            this.value = ""; 
        });

        function renderTempPreviews() {
            multiPreview.innerHTML = tempImages.map((img, idx) => `
                <div class="relative w-14 h-14 group">
                    <img src="${img}" class="w-full h-full object-cover rounded border border-gray-300 shadow-sm">
                    <div onclick="removeTempImg(${idx})" class="preview-badge hover:scale-110 transition-transform">×</div>
                </div>
            `).join('');
        }

        window.removeTempImg = (idx) => {
            tempImages.splice(idx, 1);
            renderTempPreviews();
        };

        window.switchTab = (tab) => {
            currentTab = tab;
            document.getElementById('tabActive').className = tab === 'active' ? 'pb-2 tab-active transition-all' : 'pb-2 hover:text-white transition-all';
            document.getElementById('tabArchive').className = tab === 'archive' ? 'pb-2 tab-active transition-all' : 'pb-2 hover:text-white transition-all';
            renderHistory();
        };

        window.startEdit = (id) => {
            const dossier = historyData.find(i => i.id === id);
            if(!dossier) return;
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
            tempImages = [...(dossier.images || [])];
            renderTempPreviews();
            updateCalculations();
            formTitle.innerHTML = `<span class="mr-2">✏️</span> Modifier Dossier`;
            submitBtn.innerText = "Mettre à jour";
            cancelBtn.classList.remove('hidden');
            window.scrollTo({ top: 0, behavior: 'smooth' });
        };

        window.resetForm = () => {
            editingDossierId = null;
            form.reset();
            tempImages = [];
            multiPreview.innerHTML = "";
            formTitle.innerHTML = `<span class="mr-2">📝</span> Nouveau Dossier Transit`;
            submitBtn.innerText = "Enregistrer";
            cancelBtn.classList.add('hidden');
            updateCalculations();
        };

        form.addEventListener('submit', async (e) => {
            e.preventDefault();
            if(!currentUser) return;
            
            const calcs = updateCalculations();
            const dossierId = editingDossierId || "LBV-" + Date.now().toString(36).toUpperCase();
            
            const dossierData = {
                carrier: document.getElementById('carrier').value,
                bl: document.getElementById('blNumber').value,
                container: document.getElementById('containerNumber').value,
                name: document.getElementById('itemName').value,
                client: document.getElementById('clientName').value,
                phone: document.getElementById('clientPhone').value,
                cbm: calcs.cbm.toFixed(3),
                margin: calcs.margin,
                marginPerc: calcs.marginPerc,
                totalCost: calcs.totalCost,
                images: [...tempImages],
                dims: calcs.dims,
                finance: { buy: calcs.buy, ship: calcs.ship, other: calcs.other, sell: calcs.sell },
                updatedAt: new Date().getTime()
            };

            try {
                const docRef = doc(db, 'artifacts', appId, 'users', currentUser.uid, 'active', dossierId);
                await setDoc(docRef, dossierData, { merge: true });
                resetForm();
            } catch (err) {
                console.error("Save error", err);
            }
        });

        window.openGallery = (imagesJsonStr) => {
            const images = JSON.parse(decodeURIComponent(imagesJsonStr));
            if (!images || images.length === 0) return;
            currentGallery = images; 
            currentImgIdx = 0; 
            updateGalleryUI();
            document.getElementById('galleryModal').classList.remove('hidden');
        };

        window.closeGallery = () => { document.getElementById('galleryModal').classList.add('hidden'); };
        
        function updateGalleryUI() {
            document.getElementById('modalImg').src = currentGallery[currentImgIdx];
            document.getElementById('imgCounter').innerText = `Document ${currentImgIdx + 1} / ${currentGallery.length}`;
        }
        
        window.nextImg = () => { currentImgIdx = (currentImgIdx + 1) % currentGallery.length; updateGalleryUI(); };
        window.prevImg = () => { currentImgIdx = (currentImgIdx - 1 + currentGallery.length) % currentGallery.length; updateGalleryUI(); };

        window.moveTask = async (id, toArchive) => {
            if(!currentUser) return;
            const fromCol = toArchive ? 'active' : 'archive';
            const toCol = toArchive ? 'archive' : 'active';
            
            const sourceDoc = doc(db, 'artifacts', appId, 'users', currentUser.uid, fromCol, id);
            const destDoc = doc(db, 'artifacts', appId, 'users', currentUser.uid, toCol, id);
            
            const snap = await getDoc(sourceDoc);
            if (snap.exists()) {
                await setDoc(destDoc, snap.data());
                await deleteDoc(sourceDoc);
            }
        };

        window.deleteItem = async (id) => {
            if(!currentUser || !confirm("Supprimer définitivement ce dossier ?")) return;
            const col = currentTab === 'active' ? 'active' : 'archive';
            await deleteDoc(doc(db, 'artifacts', appId, 'users', currentUser.uid, col, id));
        };

        window.filterHistory = () => { renderHistory(searchInput.value.toLowerCase()); };

        function renderHistory(filter = "") {
            const list = currentTab === 'active' ? historyData : archivesData;
            const body = document.getElementById('historyBody');
            const noRes = document.getElementById('noResult');
            
            const filtered = list
                .filter(item => item.client.toLowerCase().includes(filter) || item.bl.toLowerCase().includes(filter))
                .sort((a, b) => (b.updatedAt || 0) - (a.updatedAt || 0));

            noRes.classList.toggle('hidden', filtered.length > 0);
            body.innerHTML = filtered.map(item => {
                const statusColor = item.margin < 0 ? 'bg-red-100 text-red-700 border-red-200' : (item.marginPerc < 10 ? 'bg-orange-100 text-orange-700 border-orange-200' : 'bg-green-100 text-green-700 border-green-200');
                const statusLabel = item.margin < 0 ? 'Déficit' : (item.marginPerc < 10 ? 'Marge Faible' : 'Rentable');
                const hasImages = item.images && item.images.length > 0;
                const imagesJsonEncoded = encodeURIComponent(JSON.stringify(item.images || []));

                return `
                <tr class="hover:bg-gray-50 border-b group transition-colors">
                    <td class="p-4">
                        <div class="text-green-700 font-black uppercase text-[9px]">${item.carrier}</div>
                        <div class="text-gray-900 font-mono text-[11px] font-bold">${item.bl}</div>
                        <div class="text-[9px] text-gray-400">ID: ${item.id}</div>
                    </td>
                    <td class="p-4">
                        <div class="font-medium text-gray-800">${item.name}</div>
                        <div class="flex items-center gap-2 mt-1">
                            <span class="text-blue-700 font-bold text-[10px] bg-blue-50 px-1 rounded inline-block">${item.cbm} m³</span>
                            ${hasImages ? `<button onclick="openGallery('${imagesJsonEncoded}')" class="bg-blue-600 text-white text-[9px] px-1.5 py-0.5 rounded shadow-sm hover:bg-blue-700">📷 ${item.images.length}</button>` : ''}
                        </div>
                    </td>
                    <td class="p-4">
                        <div class="font-bold text-gray-800">${item.client}</div>
                        <div class="text-[10px] text-gray-400">${item.phone}</div>
                    </td>
                    <td class="p-4">
                        <span class="px-2 py-1 rounded-full text-[9px] font-black border uppercase ${statusColor}">${statusLabel}</span>
                    </td>
                    <td class="p-4">
                        <div class="flex flex-col text-[10px]">
                            <span class="text-gray-400">Vente: ${formatFCFA(item.finance.sell)}</span>
                            <span class="font-black ${item.margin > 0 ? 'text-green-600' : 'text-red-600'} text-xs">Net: ${formatFCFA(item.margin)}</span>
                        </div>
                    </td>
                    <td class="p-4 text-right">
                        <div class="flex justify-end space-x-1 opacity-0 group-hover:opacity-100">
                            ${currentTab === 'active' ? `<button onclick="startEdit('${item.id}')" class="p-2 bg-orange-50 text-orange-600 rounded">✏️</button>` : ''}
                            <button onclick="moveTask('${item.id}', ${currentTab === 'active'})" class="p-2 bg-blue-50 text-blue-600 rounded">${currentTab === 'active' ? '📥' : '📤'}</button>
                            <button onclick="deleteItem('${item.id}')" class="p-2 bg-red-50 text-red-500 rounded">🗑️</button>
                        </div>
                    </td>
                </tr>`;
            }).join('');
            updateDashboard();
        }

        updateCalculations();
    </script>
</body>
</html>
