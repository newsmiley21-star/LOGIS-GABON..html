<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Logis-Gabon | Gestionnaire de Fret Expert</title>
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
        /* --- STYLE DU VERROUILLAGE --- */
        #lockScreen {
            position: fixed;
            inset: 0;
            background-color: #064e3b; /* Vert très sombre */
            z-index: 9999;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            color: white;
            transition: opacity 0.5s ease-out, visibility 0.5s;
        }
        #lockScreen.hidden {
            opacity: 0;
            visibility: hidden;
            pointer-events: none;
        }
        .pin-dot {
            width: 16px;
            height: 16px;
            border: 2px solid white;
            border-radius: 50%;
            margin: 0 8px;
            transition: background 0.2s;
        }
        .pin-dot.filled {
            background-color: white;
            transform: scale(1.2);
        }
        .keypad-btn {
            width: 70px;
            height: 70px;
            background: rgba(255,255,255,0.1);
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 1.5rem;
            font-weight: bold;
            cursor: pointer;
            transition: background 0.2s;
        }
        .keypad-btn:active {
            background: rgba(255,255,255,0.3);
        }
        .shake {
            animation: shake 0.4s ease-in-out;
        }
        @keyframes shake {
            0%, 100% { transform: translateX(0); }
            25% { transform: translateX(-10px); }
            75% { transform: translateX(10px); }
        }

        /* Tes anciens styles */
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
    </style>
</head>
<body class="p-4 md:p-8">

    <!-- ÉCRAN DE VERROUILLAGE (Overlay) -->
    <div id="lockScreen">
        <div class="text-center mb-8">
            <h1 class="text-3xl font-black italic tracking-tighter uppercase mb-2">Logis-Gabon</h1>
            <p class="text-green-200 text-sm">Accès Sécurisé - Administration</p>
        </div>

        <div id="pinDisplay" class="flex mb-12">
            <div class="pin-dot"></div>
            <div class="pin-dot"></div>
            <div class="pin-dot"></div>
            <div class="pin-dot"></div>
        </div>

        <div class="grid grid-cols-3 gap-6">
            <div onclick="pressPin('1')" class="keypad-btn">1</div>
            <div onclick="pressPin('2')" class="keypad-btn">2</div>
            <div onclick="pressPin('3')" class="keypad-btn">3</div>
            <div onclick="pressPin('4')" class="keypad-btn">4</div>
            <div onclick="pressPin('5')" class="keypad-btn">5</div>
            <div onclick="pressPin('6')" class="keypad-btn">6</div>
            <div onclick="pressPin('7')" class="keypad-btn">7</div>
            <div onclick="pressPin('8')" class="keypad-btn">8</div>
            <div onclick="pressPin('9')" class="keypad-btn">9</div>
            <div></div>
            <div onclick="pressPin('0')" class="keypad-btn">0</div>
            <div onclick="clearPin()" class="keypad-btn text-sm">Effacer</div>
        </div>
        
        <p id="lockMsg" class="mt-8 text-red-400 font-bold hidden">Code incorrect</p>
    </div>

    <!-- CONTENU DE L'APPLICATION (Ton code original) -->
    <div class="max-w-7xl mx-auto">
        <!-- Header -->
        <header class="flex flex-col md:flex-row justify-between items-center mb-8 bg-green-800 p-6 rounded-xl text-white shadow-lg">
            <div>
                <h1 class="text-3xl font-bold italic tracking-tighter uppercase">Logis-Gabon</h1>
                <p class="opacity-90 text-sm font-medium">Gestion Transit & Suivi des Dépenses - Libreville / Owendo</p>
            </div>
            <div class="flex items-center gap-4">
                <div id="clock" class="text-lg font-mono bg-green-900 px-4 py-1 rounded-full"></div>
                <button onclick="lockApp()" class="bg-red-700 hover:bg-red-800 p-2 rounded text-xs uppercase font-bold">🔒 Quitter</button>
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

        <!-- FORMULAIRE ET ANALYSE -->
        <div class="grid grid-cols-1 lg:grid-cols-3 gap-8 mb-8">
            <section class="card p-6 lg:col-span-2 relative">
                <div id="editModeIndicator" class="hidden absolute top-4 right-6 bg-orange-100 text-orange-700 px-3 py-1 rounded-full text-xs font-bold animate-pulse uppercase">
                    Mode Édition : <span id="editingId"></span>
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
                            <div><label class="text-[10px] font-bold text-gray-500 uppercase">Long (cm)</label><input type="number" id="long" required oninput="updateCalculations()" class="w-full p-2 border rounded"></div>
                            <div><label class="text-[10px] font-bold text-gray-500 uppercase">Larg (cm)</label><input type="number" id="larg" required oninput="updateCalculations()" class="w-full p-2 border rounded"></div>
                            <div><label class="text-[10px] font-bold text-gray-500 uppercase">Haut (cm)</label><input type="number" id="haut" required oninput="updateCalculations()" class="w-full p-2 border rounded"></div>
                        </div>
                    </div>
                    <!-- Client & Finance -->
                    <div class="space-y-4">
                        <h3 class="text-xs font-black text-green-700 uppercase tracking-widest text-[10px]">Client & Dépenses (FCFA)</h3>
                        <div class="grid grid-cols-2 gap-2">
                            <div><label class="block text-sm font-semibold text-gray-700">Client</label><input type="text" id="clientName" required class="w-full p-2 border rounded mt-1"></div>
                            <div><label class="block text-sm font-semibold text-gray-700">Téléphone</label><input type="tel" id="clientPhone" required class="w-full p-2 border rounded mt-1"></div>
                        </div>
                        <div class="grid grid-cols-2 gap-2">
                            <div><label class="block text-sm font-semibold text-gray-700">Achat Fournisseur</label><input type="number" id="buyPrice" required oninput="updateCalculations()" class="w-full p-2 border rounded mt-1 text-xs"></div>
                            <div><label class="block text-sm font-semibold text-gray-700">Fret / Transport</label><input type="number" id="shipping" required oninput="updateCalculations()" class="w-full p-2 border rounded mt-1 text-xs"></div>
                        </div>
                        <div class="grid grid-cols-2 gap-2">
                            <div><label class="block text-sm font-semibold text-gray-700">Autres Frais LBV</label><input type="number" id="otherFees" value="0" oninput="updateCalculations()" class="w-full p-2 border rounded mt-1 text-xs"></div>
                            <div><label class="block text-sm font-semibold text-gray-700">Prix de Vente</label><input type="number" id="sellPrice" required oninput="updateCalculations()" class="w-full p-2 border rounded mt-1 font-bold bg-green-50"></div>
                        </div>
                        <div class="p-3 bg-gray-50 border-2 border-dashed rounded-lg">
                            <label class="block text-xs font-black text-gray-600 mb-2 uppercase tracking-tighter italic">Pièces Jointes</label>
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

            <!-- ANALYSE FINANCIÈRE -->
            <section class="space-y-6">
                <div id="financeCard" class="card p-6 border-t-8 border-green-600 shadow-xl">
                    <h2 class="text-lg font-bold mb-4 text-gray-800 flex items-center justify-between">
                        <span>📊 Détail Financier</span>
                        <span id="alertBadge" class="hidden px-2 py-1 rounded text-[10px] font-black uppercase"></span>
                    </h2>
                    <div id="analysisContent" class="space-y-3 text-[13px]">
                        <div class="expense-row"><span class="text-gray-500 italic">Volume Chargement</span><span id="resCBM" class="font-bold text-blue-700">0.000 m³</span></div>
                        <div class="pt-2">
                            <p class="text-[10px] font-black text-gray-400 uppercase mb-2">Décompte des Dépenses</p>
                            <div class="expense-row"><span>Achat Marchandise</span><span id="resBuy">0 FCFA</span></div>
                            <div class="expense-row"><span>Fret Maritime/Aérien</span><span id="resShip">0 FCFA</span></div>
                            <div class="expense-row text-red-500"><span>Douane Estimée (45%) ℹ️</span><span id="resTax" class="font-bold">0 FCFA</span></div>
                            <div class="expense-row"><span>Logistique / Port</span><span id="resOther">0 FCFA</span></div>
                            <div class="expense-row text-green-700 font-bold"><span>Frais Dossier Fixes</span><span>65 000 FCFA</span></div>
                        </div>
                        <div class="flex justify-between font-black pt-2 border-t border-gray-300 uppercase text-[11px] text-gray-700">
                            <span>Coût de revient total 🛡️</span><span id="resTotalCost">0 FCFA</span>
                        </div>
                        <div class="py-4 mt-2 text-center border-2 rounded-xl bg-gray-50 border-dashed">
                            <p class="text-[10px] font-bold text-gray-400 uppercase">Marge Nette Prévue</p>
                            <span id="resMargin" class="text-2xl font-black text-gray-400">0 FCFA</span>
                            <div id="marginPercent" class="text-[10px] font-bold text-gray-500 mt-1">0% du prix de vente</div>
                        </div>
                        <div id="accountingAdvice" class="hidden mt-4 p-3 bg-white border border-gray-200 rounded-lg text-[11px] italic shadow-inner"></div>
                    </div>
                </div>
            </section>
        </div>

        <!-- REGISTRE -->
        <div class="card shadow-2xl overflow-hidden">
            <div class="p-4 bg-gray-800 flex flex-col md:flex-row justify-between items-center gap-4 text-white">
                <div class="flex space-x-6 text-sm font-bold uppercase">
                    <button id="tabActive" onclick="switchTab('active')" class="pb-2 tab-active transition-all text-green-400">En cours</button>
                    <button id="tabArchive" onclick="switchTab('archive')" class="pb-2 hover:text-green-400 transition-all opacity-50">Archives</button>
                </div>
                <div class="relative w-full md:w-96">
                    <input type="text" id="searchInput" onkeyup="filterHistory()" placeholder="Rechercher Dossier..." class="w-full pl-10 pr-4 py-2 rounded-full bg-gray-700 text-white text-sm outline-none border-none">
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

    <!-- MODAL GALERIE -->
    <div id="galleryModal" class="fixed inset-0 bg-black bg-opacity-95 hidden z-50 flex flex-col items-center justify-center p-4">
        <button onclick="closeGallery()" class="absolute top-6 right-6 text-white text-4xl">&times;</button>
        <div class="relative w-full max-w-4xl flex items-center justify-center">
            <button onclick="prevImg()" class="absolute left-0 md:-left-20 text-white p-4">❮</button>
            <img id="modalImg" src="" class="max-w-full max-h-[80vh] rounded shadow-2xl">
            <button onclick="nextImg()" class="absolute right-0 md:-right-20 text-white p-4">❯</button>
        </div>
        <p id="imgCounter" class="text-white mt-6 font-mono text-sm"></p>
    </div>

    <script>
        // --- LOGIQUE DE VERROUILLAGE ---
        let currentPin = "";
        const MASTER_PIN = "6122";

        function pressPin(num) {
            if (currentPin.length < 4) {
                currentPin += num;
                updatePinDisplay();
                if (currentPin.length === 4) {
                    checkPin();
                }
            }
        }

        function clearPin() {
            currentPin = "";
            updatePinDisplay();
        }

        function updatePinDisplay() {
            const dots = document.querySelectorAll('.pin-dot');
            dots.forEach((dot, i) => {
                if (i < currentPin.length) dot.classList.add('filled');
                else dot.classList.remove('filled');
            });
        }

        function checkPin() {
            const lockScreen = document.getElementById('lockScreen');
            const pinDisplay = document.getElementById('pinDisplay');
            const lockMsg = document.getElementById('lockMsg');

            if (currentPin === MASTER_PIN) {
                lockScreen.classList.add('hidden');
                currentPin = "";
                updatePinDisplay();
                lockMsg.classList.add('hidden');
            } else {
                pinDisplay.classList.add('shake');
                lockMsg.classList.remove('hidden');
                setTimeout(() => {
                    pinDisplay.classList.remove('shake');
                    clearPin();
                }, 400);
            }
        }

        function lockApp() {
            document.getElementById('lockScreen').classList.remove('hidden');
            clearPin();
        }

        // --- TON CODE ORIGINAL (MODIFIÉ POUR LES BOUTONS) ---
        const DOSSIER_FEE = 65000;
        let currentTab = 'active';
        let history = JSON.parse(localStorage.getItem('logi_active_v9')) || [];
        let archives = JSON.parse(localStorage.getItem('logi_archive_v9')) || [];
        let tempImages = []; 
        let editingDossierId = null; 
        let currentGallery = [];
        let currentImgIdx = 0;

        // Clock
        setInterval(() => {
            const el = document.getElementById('clock');
            if(el) el.innerText = new Date().toLocaleString('fr-FR');
        }, 1000);

        function formatFCFA(num) {
            return new Intl.NumberFormat('fr-FR').format(Math.round(num));
        }

        function updateCalculations() {
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
            const rawTotalCost = caf + tax + other + DOSSIER_FEE;
            const totalCost = rawTotalCost * 1.05; // Sécurité 5%
            const margin = sell - totalCost;
            const marginPerc = sell > 0 ? (margin / sell) * 100 : 0;

            document.getElementById('resCBM').innerText = cbm.toFixed(3) + " m³";
            document.getElementById('resBuy').innerText = formatFCFA(buy) + " FCFA";
            document.getElementById('resShip').innerText = formatFCFA(ship) + " FCFA";
            document.getElementById('resTax').innerText = formatFCFA(tax) + " FCFA";
            document.getElementById('resOther').innerText = formatFCFA(other) + " FCFA";
            document.getElementById('resTotalCost').innerText = formatFCFA(totalCost) + " FCFA";
            document.getElementById('resMargin').innerText = formatFCFA(margin) + " FCFA";
            document.getElementById('marginPercent').innerText = marginPerc.toFixed(1) + "% du prix de vente";

            const financeCard = document.getElementById('financeCard');
            const adviceBox = document.getElementById('accountingAdvice');
            adviceBox.classList.remove('hidden');

            if (margin < 0) {
                financeCard.className = "card p-6 border-t-8 shadow-xl alert-critical";
                adviceBox.innerHTML = `⚠️ <strong>Danger :</strong> Déficit. Vendez à au moins ${formatFCFA(totalCost * 1.15)} FCFA.`;
            } else {
                financeCard.className = "card p-6 border-t-8 shadow-xl border-green-600";
                adviceBox.innerHTML = `✅ <strong>Sain :</strong> Rentabilité positive sécurisée.`;
            }
            
            return { cbm, tax, totalCost, margin, marginPerc, buy, ship, other, sell, dims: {l, w, h} };
        }

        // Sauvegarde & Rendu
        function saveAll() {
            localStorage.setItem('logi_active_v9', JSON.stringify(history));
            localStorage.setItem('logi_archive_v9', JSON.stringify(archives));
            updateDashboard();
        }

        function updateDashboard() {
            const list = [...history, ...archives];
            const totalCA = list.reduce((sum, item) => sum + (item.finance?.sell || 0), 0);
            const totalMarge = list.reduce((sum, item) => sum + (item.margin || 0), 0);
            document.getElementById('statsCA').innerText = formatFCFA(totalCA) + " FCFA";
            document.getElementById('statsMarge').innerText = formatFCFA(totalMarge) + " FCFA";
            document.getElementById('statsDossiers').innerText = list.length;
        }

        function renderHistory() {
            const body = document.getElementById('historyBody');
            const list = currentTab === 'active' ? history : archives;
            body.innerHTML = list.map(item => `
                <tr>
                    <td class="p-4 font-bold text-blue-700">${item.id}<br><span class="text-[10px] text-gray-400 font-normal">${item.carrier} / ${item.bl}</span></td>
                    <td class="p-4">${item.name} <span class="text-gray-400 font-normal">(${item.cbm} m³)</span></td>
                    <td class="p-4">${item.client}<br><span class="text-gray-400 font-normal">${item.phone}</span></td>
                    <td class="p-4"><span class="px-2 py-0.5 rounded-full ${item.margin >= 0 ? 'bg-green-100 text-green-700' : 'bg-red-100 text-red-700'} font-bold">${item.margin >= 0 ? 'Bénéficiaire' : 'Déficit'}</span></td>
                    <td class="p-4 font-black">${formatFCFA(item.finance.sell)}</td>
                    <td class="p-4 text-right space-x-2">
                        <button onclick="startEdit('${item.id}')" class="text-blue-500">✏️</button>
                        <button onclick="moveTask('${item.id}', ${currentTab === 'active'})" class="text-gray-500">${currentTab === 'active' ? '📦' : '📂'}</button>
                        <button onclick="deleteItem('${item.id}')" class="text-red-500">🗑️</button>
                    </td>
                </tr>
            `).join('');
            updateDashboard();
        }

        function switchTab(tab) {
            currentTab = tab;
            document.getElementById('tabActive').className = tab === 'active' ? 'pb-2 tab-active text-green-400' : 'pb-2 opacity-50';
            document.getElementById('tabArchive').className = tab === 'archive' ? 'pb-2 tab-active text-green-400' : 'pb-2 opacity-50';
            renderHistory();
        }

        function resetForm() {
            editingDossierId = null;
            document.getElementById('logiForm').reset();
            document.getElementById('formTitle').innerText = "📝 Nouveau Dossier Transit";
            document.getElementById('submitBtn').innerText = "Enregistrer";
            document.getElementById('cancelBtn').classList.add('hidden');
            tempImages = [];
            document.getElementById('multiPreviewContainer').innerHTML = "";
            updateCalculations();
        }

        document.getElementById('logiForm').addEventListener('submit', (e) => {
            e.preventDefault();
            const calcs = updateCalculations();
            const data = {
                id: editingDossierId || "LBV-" + Date.now().toString(36).toUpperCase(),
                date: new Date().toLocaleDateString('fr-FR'),
                carrier: document.getElementById('carrier').value,
                bl: document.getElementById('blNumber').value,
                container: document.getElementById('containerNumber').value,
                name: document.getElementById('itemName').value,
                client: document.getElementById('clientName').value,
                phone: document.getElementById('clientPhone').value,
                cbm: calcs.cbm.toFixed(3),
                margin: calcs.margin,
                finance: calcs,
                images: tempImages
            };

            if(editingDossierId) history = history.map(i => i.id === editingDossierId ? data : i);
            else history.unshift(data);

            saveAll(); renderHistory(); resetForm();
        });

        function deleteItem(id) {
            if(!confirm("Supprimer ?")) return;
            if(currentTab === 'active') history = history.filter(i => i.id !== id);
            else archives = archives.filter(i => i.id !== id);
            saveAll(); renderHistory();
        }

        function moveTask(id, toArchive) {
            if(toArchive) {
                const item = history.find(i => i.id === id);
                archives.unshift(item); history = history.filter(i => i.id !== id);
            } else {
                const item = archives.find(i => i.id === id);
                history.unshift(item); archives = archives.filter(i => i.id !== id);
            }
            saveAll(); renderHistory();
        }

        // Init
        window.onload = () => {
            renderHistory();
            updateCalculations();
        };
    </script>
</body>
</html>
