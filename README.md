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
    </style>
</head>
<body class="p-4 md:p-8">

    <div class="max-w-7xl mx-auto">
        <!-- Header -->
        <header class="flex flex-col md:flex-row justify-between items-center mb-8 bg-green-800 p-6 rounded-xl text-white shadow-lg">
            <div>
                <h1 class="text-3xl font-bold italic tracking-tighter uppercase">Logi-Gabon</h1>
                <p class="opacity-90 text-sm font-medium">Gestion Transit & Suivi des Dépenses - Libreville / Owendo</p>
            </div>
            <div id="clock" class="text-lg font-mono mt-4 md:mt-0 bg-green-900 px-4 py-1 rounded-full"></div>
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
                            <div>
                                <label class="text-[10px] font-bold text-gray-500 uppercase">Long (cm)</label>
                                <input type="number" id="long" required oninput="updateCalculations()" class="w-full p-2 border rounded">
                            </div>
                            <div>
                                <label class="text-[10px] font-bold text-gray-500 uppercase">Larg (cm)</label>
                                <input type="number" id="larg" required oninput="updateCalculations()" class="w-full p-2 border rounded">
                            </div>
                            <div>
                                <label class="text-[10px] font-bold text-gray-500 uppercase">Haut (cm) <span class="text-blue-500 cursor-help" title="Calcul automatique du volume en m³">🛈</span></label>
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

                <!-- Rappels Experts Owendo (Conseil Gemini) -->
                <div class="mt-6 p-3 bg-blue-50 border-l-4 border-blue-500 rounded text-[11px] text-blue-800">
                    <p class="font-bold uppercase mb-1 flex items-center">
                        <span class="mr-1">💡</span> Memo Expert Transit Gabon
                    </p>
                    <ul class="list-disc list-inside space-y-1 opacity-90">
                        <li><strong>Incoterm :</strong> Assurez-vous d'ajouter l'assurance (~2%) si achat FOB pour la base Douane.</li>
                        <li><strong>Owendo :</strong> Prévoir une marge pour le magasinage si le navire dépasse 5 jours à quai.</li>
                        <li><strong>Documents :</strong> Toujours scanner la copie du BL et la facture fournisseur avant livraison.</li>
                    </ul>
                </div>
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
                            <div class="expense-row text-red-500">
                                <span title="Moyenne Gabon : Droit de Douane + TVA + Taxes Communautaires">Douane Estimée (45%) ℹ️</span>
                                <span id="resTax" class="font-bold">0 FCFA</span>
                            </div>
                            <div class="expense-row"><span>Logistique / Port</span><span id="resOther">0 FCFA</span></div>
                            <div class="expense-row text-green-700 font-bold"><span>Frais Dossier Fixes</span><span>65 000 FCFA</span></div>
                        </div>

                        <div class="flex justify-between font-black pt-2 border-t border-gray-300 uppercase text-[11px] text-gray-700">
                            <span title="Inclus une marge de sécurité de 5% pour imprévus">Coût de revient total 🛡️</span>
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
    <div id="galleryModal" class="fixed inset-0 bg-black bg-opacity-95 hidden z-50 flex flex-col items-center justify-center p-4">
        <button onclick="closeGallery()" class="absolute top-6 right-6 text-white text-4xl font-bold hover:text-red-500 transition-colors">&times;</button>
        <div class="relative w-full max-w-4xl flex items-center justify-center">
            <button onclick="prevImg()" class="absolute left-0 md:-left-20 bg-white bg-opacity-20 hover:bg-opacity-40 text-white p-4 rounded-full transition-all">❮</button>
            <img id="modalImg" src="" class="max-w-full max-h-[80vh] rounded shadow-2xl transition-all duration-300">
            <button onclick="nextImg()" class="absolute right-0 md:-right-20 bg-white bg-opacity-20 hover:bg-opacity-40 text-white p-4 rounded-full transition-all">❯</button>
        </div>
        <p id="imgCounter" class="text-white mt-6 font-mono text-sm bg-gray-800 px-4 py-1 rounded-full"></p>
    </div>

    <script>
        const DOSSIER_FEE = 65000;
        let currentTab = 'active';
        let history = JSON.parse(localStorage.getItem('logi_active_v9')) || [];
        let archives = JSON.parse(localStorage.getItem('logi_archive_v9')) || [];
        let tempImages = []; 
        let editingDossierId = null; 
        let currentGallery = [];
        let currentImgIdx = 0;

        const form = document.getElementById('logiForm');
        const fileInput = document.getElementById('docPhotos');
        const multiPreview = document.getElementById('multiPreviewContainer');
        const searchInput = document.getElementById('searchInput');
        const submitBtn = document.getElementById('submitBtn');
        const cancelBtn = document.getElementById('cancelBtn');
        const formTitle = document.getElementById('formTitle');

        // Clock
        setInterval(() => document.getElementById('clock').innerText = new Date().toLocaleString('fr-FR'), 1000);

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
            
            // CONSEIL : Ajout d'une marge de sécurité de 5% sur les coûts totaux (Owendo Risk)
            const rawTotalCost = caf + tax + other + DOSSIER_FEE;
            const securityBuffer = rawTotalCost * 0.05; 
            const totalCost = rawTotalCost + securityBuffer;

            const margin = sell - totalCost;
            const marginPerc = sell > 0 ? (margin / sell) * 100 : 0;

            // Update UI
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

            // Logic Avertissements & Solutions
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
                    adviceBox.innerHTML = `⚠️ <strong>Solution :</strong> Vous perdez de l'argent. Augmentez le prix de vente à au moins <strong>${formatFCFA(totalCost * 1.15)} FCFA</strong> pour assurer 15% de marge.`;
                } else if (marginPerc < 10) {
                    financeCard.classList.add('border-orange-500');
                    alertBadge.innerText = "ALERTE : Faible Marge";
                    alertBadge.className = "px-2 py-1 rounded text-[10px] font-black uppercase bg-orange-500 text-white";
                    alertBadge.classList.remove('hidden');
                    adviceBox.innerHTML = `💡 <strong>Conseil :</strong> Votre rentabilité est faible (${marginPerc.toFixed(1)}%). Essayez de renégocier le fret ou d'ajuster le prix client.`;
                } else {
                    alertBadge.innerText = "SANTÉ : Excellente";
                    alertBadge.className = "px-2 py-1 rounded text-[10px] font-black uppercase bg-green-600 text-white";
                    alertBadge.classList.remove('hidden');
                    adviceBox.innerHTML = `✅ <strong>Analyse :</strong> Ce dossier est sain. Rentabilité confirmée incluant une sécurité logistique.`;
                }
            }
            
            return { cbm, tax, totalCost, margin, marginPerc, buy, ship, other, sell, dims: {l, w, h} };
        }

        function updateDashboard() {
            const list = [...history, ...archives];
            const totalCA = list.reduce((sum, item) => sum + (item.finance?.sell || 0), 0);
            const totalMarge = list.reduce((sum, item) => sum + (item.margin || 0), 0);
            const avgRent = list.length > 0 ? (totalMarge / totalCA) * 100 : 0;

            document.getElementById('statsCA').innerText = formatFCFA(totalCA) + " FCFA";
            document.getElementById('statsMarge').innerText = formatFCFA(totalMarge) + " FCFA";
            document.getElementById('statsMarge').className = `text-xl font-black ${totalMarge >= 0 ? 'text-green-700' : 'text-red-600'}`;
            document.getElementById('statsRentabilite').innerText = (isNaN(avgRent) ? 0 : avgRent.toFixed(1)) + "%";
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

        function removeTempImg(idx) {
            tempImages.splice(idx, 1);
            renderTempPreviews();
        }

        function switchTab(tab) {
            currentTab = tab;
            document.getElementById('tabActive').className = tab === 'active' ? 'pb-2 tab-active transition-all' : 'pb-2 hover:text-white transition-all';
            document.getElementById('tabArchive').className = tab === 'archive' ? 'pb-2 tab-active transition-all' : 'pb-2 hover:text-white transition-all';
            renderHistory();
        }

        function startEdit(id) {
            const dossier = history.find(i => i.id === id);
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
        }

        function resetForm() {
            editingDossierId = null;
            form.reset();
            tempImages = [];
            multiPreview.innerHTML = "";
            formTitle.innerHTML = `<span class="mr-2">📝</span> Nouveau Dossier Transit`;
            submitBtn.innerText = "Enregistrer";
            cancelBtn.classList.add('hidden');
            updateCalculations();
        }

        form.addEventListener('submit', (e) => {
            e.preventDefault();
            const calcs = updateCalculations();
            const dossierData = {
                id: editingDossierId || "LBV-" + Date.now().toString(36).toUpperCase(),
                date: editingDossierId ? history.find(i=>i.id===editingDossierId).date : new Date().toLocaleDateString('fr-FR'),
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
                finance: { buy: calcs.buy, ship: calcs.ship, other: calcs.other, sell: calcs.sell }
            };

            if (editingDossierId) {
                const idx = history.findIndex(i => i.id === editingDossierId);
                history[idx] = dossierData;
            } else {
                history.unshift(dossierData);
            }
            saveAll();
            renderHistory();
            resetForm();
        });

        function openGallery(images, startIdx = 0) {
            if (!images || images.length === 0) return;
            currentGallery = images; 
            currentImgIdx = startIdx; 
            updateGalleryUI();
            document.getElementById('galleryModal').classList.remove('hidden');
        }
        function closeGallery() { document.getElementById('galleryModal').classList.add('hidden'); }
        function updateGalleryUI() {
            document.getElementById('modalImg').src = currentGallery[currentImgIdx];
            document.getElementById('imgCounter').innerText = `Document ${currentImgIdx + 1} / ${currentGallery.length}`;
        }
        function nextImg() { currentImgIdx = (currentImgIdx + 1) % currentGallery.length; updateGalleryUI(); }
        function prevImg() { currentImgIdx = (currentImgIdx - 1 + currentGallery.length) % currentGallery.length; updateGalleryUI(); }

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

        function deleteItem(id) {
            if(!confirm("Supprimer ce dossier ?")) return;
            if(currentTab === 'active') history = history.filter(i => i.id !== id);
            else archives = archives.filter(i => i.id !== id);
            saveAll(); renderHistory();
        }

        function saveAll() {
            localStorage.setItem('logi_active_v9', JSON.stringify(history));
            localStorage.setItem('logi_archive_v9', JSON.stringify(archives));
            updateDashboard();
        }

        function filterHistory() { renderHistory(searchInput.value.toLowerCase()); }

        function renderHistory(filter = "") {
            const list = currentTab === 'active' ? history : archives;
            const body = document.getElementById('historyBody');
            const noRes = document.getElementById('noResult');
            const filtered = list.filter(item => 
                item.client.toLowerCase().includes(filter) || 
                item.bl.toLowerCase().includes(filter) || 
                item.name.toLowerCase().includes(filter)
            );

            noRes.classList.toggle('hidden', filtered.length > 0);
            body.innerHTML = filtered.map(item => {
                const statusColor = item.margin < 0 ? 'bg-red-100 text-red-700 border-red-200' : (item.marginPerc < 10 ? 'bg-orange-100 text-orange-700 border-orange-200' : 'bg-green-100 text-green-700 border-green-200');
                const statusLabel = item.margin < 0 ? 'Déficit' : (item.marginPerc < 10 ? 'Marge Faible' : 'Rentable');
                const hasImages = item.images && item.images.length > 0;
                const imagesJson = JSON.stringify(item.images || []).replace(/"/g, '&quot;');

                return `
                    <tr class="hover:bg-gray-50 transition-colors">
                        <td class="p-4">
                            <div class="font-black text-gray-800">${item.carrier}</div>
                            <div class="text-[10px] text-gray-400">BL: ${item.bl}</div>
                            <div class="text-[10px] text-gray-400">ID: ${item.id}</div>
                        </td>
                        <td class="p-4">
                            <div class="font-bold">${item.name}</div>
                            <div class="text-[10px] text-blue-600 font-mono">${item.cbm} m³</div>
                        </td>
                        <td class="p-4">
                            <div class="font-bold">${item.client}</div>
                            <div class="text-[10px] text-gray-500">${item.phone}</div>
                        </td>
                        <td class="p-4">
                            <span class="px-2 py-1 rounded-full text-[9px] font-black uppercase border ${statusColor}">${statusLabel}</span>
                        </td>
                        <td class="p-4">
                            <div class="font-black">${formatFCFA(item.finance.sell)}</div>
                            <div class="text-[9px] ${item.margin >= 0 ? 'text-green-600' : 'text-red-500'}">Marge: ${formatFCFA(item.margin)}</div>
                        </td>
                        <td class="p-4 text-right space-x-1">
                            ${hasImages ? `<button onclick="openGallery(${imagesJson})" class="p-2 bg-blue-100 text-blue-600 rounded hover:bg-blue-200" title="Voir les documents">🖼️</button>` : ''}
                            <button onclick="startEdit('${item.id}')" class="p-2 bg-gray-100 text-gray-600 rounded hover:bg-gray-200">✏️</button>
                            <button onclick="moveTask('${item.id}', ${currentTab === 'active'})" class="p-2 ${currentTab === 'active' ? 'bg-green-100 text-green-600' : 'bg-orange-100 text-orange-600'} rounded hover:opacity-80">
                                ${currentTab === 'active' ? '✅' : '🔄'}
                            </button>
                            <button onclick="deleteItem('${item.id}')" class="p-2 bg-red-100 text-red-600 rounded hover:bg-red-200">🗑️</button>
                        </td>
                    </tr>
                `;
            }).join('');
        }

        // Init
        window.onload = () => {
            updateDashboard();
            renderHistory();
        };
    </script>
</body>
</html>
