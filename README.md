<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>BAD CLAN Tournament Registration</title>
    <!-- Load Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    fontFamily: {
                        sans: ['Inter', 'sans-serif'],
                    },
                }
            }
        }
    </script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@100..900&display=swap');
        
        body {
            background-color: #000508; 
            color: #E0F2F1; 
            font-family: 'Inter', sans-serif;
            overflow-x: hidden;
            /* Radial gradient for a dark, central focus effect */
            background-image: radial-gradient(circle at 50% 50%, rgba(12, 20, 30, 0.9) 0%, rgba(0, 5, 8, 1) 70%);
        }

        /* Custom Electric Glow Effects */
        .text-glow-blue {
            color: #00eaff; /* Electric Blue */
            text-shadow: 0 0 5px #00eaff, 0 0 15px #00eaff, 0 0 30px #00eaff;
            letter-spacing: 0.08em;
        }

        .text-glow-orange {
            color: #ff7f00; /* Electric Orange */
            text-shadow: 0 0 5px #ff7f00, 0 0 15px #ff7f00, 0 0 30px rgba(255, 127, 0, 0.7);
        }
        
        /* Main Panel Style - Metallic, layered frame */
        .main-panel {
            background-color: #0c141d; 
            border: 2px solid #3d4b63; 
            /* Multi-color glow effect for high-tech look */
            box-shadow: 
                0 0 15px rgba(0, 234, 255, 0.3),   
                0 0 30px rgba(255, 127, 0, 0.4),    
                inset 0 0 10px rgba(0, 0, 0, 0.8); 
        }
        
        /* Internal Card Style - Sharper internal borders */
        .internal-card {
            background-color: #0c141d;
            border: 1px solid #1f3045; 
            box-shadow: 0 0 10px rgba(0, 234, 255, 0.1);
        }

        .input-style {
            background-color: #172a3a; 
            border: 1px solid #3d4b63;
            color: #E0F2F1;
            transition: all 0.3s ease;
            box-shadow: inset 0 1px 3px rgba(0, 0, 0, 0.6);
        }
        .input-style:focus {
            border-color: #00eaff;
            box-shadow: 0 0 0 1px #00eaff, inset 0 1px 3px rgba(0, 0, 0, 0.6);
            outline: none;
        }

        .btn-primary {
            background-color: #ff7f00;
            color: #000508;
            font-weight: 900;
            transition: all 0.2s;
            box-shadow: 0 0 10px rgba(255, 127, 0, 0.8), 0 5px 15px rgba(0, 0, 0, 0.5);
        }
        .btn-primary:hover {
            background-color: #ff9120;
            box-shadow: 0 0 20px rgba(255, 127, 0, 1), 0 0 30px rgba(255, 127, 0, 0.5);
        }
        
        .nav-btn {
            background-color: #0c141d;
            border: 1px solid #00eaff;
            box-shadow: 0 0 8px rgba(0, 234, 255, 0.5);
            color: #00eaff;
            font-weight: 700;
        }
        .nav-btn:hover {
            background-color: #0c141d;
            color: #E0F2F1;
            box-shadow: 0 0 15px #00eaff;
        }

        /* 4-second cinematic drop animation */
        @keyframes fall-in {
            0% { transform: translateY(-200vh) rotateX(90deg) scale(0.5); opacity: 0; }
            70% { transform: translateY(0) rotateX(-5deg) scale(1); opacity: 1; }
            100% { transform: translateY(0) rotateX(0deg) scale(1); }
        }

        .falling-animation {
            animation: fall-in 4s cubic-bezier(0.2, 0.8, 0.4, 1.1) forwards;
            transform: translateY(-200vh); 
            opacity: 0;
        }
        
    </style>
</head>
<body class="min-h-screen p-4 flex justify-center items-center">

    <!-- Main Container -->
    <div id="app-container" class="w-full max-w-3xl main-panel rounded-lg p-6 md:p-12 falling-animation">
        
        <!-- Navigation Header -->
        <header class="flex justify-between items-center pb-4 mb-8 border-b border-[#3d4b63]">
            <h1 class="text-3xl font-extrabold text-glow-blue">BAD CLAN // SYSTEM</h1>
            <button id="nav-button" onclick="switchView()" class="text-sm font-semibold transition duration-200 py-2 px-4 rounded-lg nav-btn">
                Admin Login
            </button>
        </header>

        <!-- Main Content Area -->
        <div id="content-area">
            <!-- Loading State -->
            <div class="text-center p-8">
                <div class="animate-spin rounded-full h-10 w-10 border-t-4 border-b-4 border-00eaff inline-block"></div>
                <p class="mt-4 text-sm text-gray-400">Initializing Core System...</p>
            </div>
        </div>

    </div>

    <!-- Firebase SDK Imports and Logic -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, addDoc, onSnapshot, collection, query, serverTimestamp, setDoc, setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Global Firebase Variables
        let app;
        let db;
        let auth;
        let userId = null;
        let isAuthReady = false;
        let unsubscribeConfig = null;
        let unsubscribeRegistrations = null; 

        // State variables
        let currentView = 'register';
        let registrations = [];
        let isRegistered = false; 
        const ADMIN_PASSWORD = 'Teddy786'; 
        const DEFAULT_ENTRY_FEE = '200 PKR'; 

        // Default Configuration including payment methods
        const DEFAULT_PAYMENT_METHODS = [
            { name: 'EasyPaisa', details: '0300-1234567 (Account Holder: Bad Clan Admin)' },
            { name: 'JazzCash', details: '0301-7654321 (Account Holder: Tournament Manager)' },
            { name: 'NayaPay', details: '0302-9876543 (Virtual Account)' }
        ];

        let config = { 
            entryFee: DEFAULT_ENTRY_FEE, 
            schedule: 'Tournament timings and schedule will be announced soon. Check back later!', 
            thankYouMessage: 'Your participation is confirmed! Get ready for battle!',
            paymentMethods: DEFAULT_PAYMENT_METHODS
        };

        let tempPaymentMethods = []; // Temporary state for admin editing of payment methods

        // Global variables provided by the Canvas environment (These MUST remain for the code to work in the environment)
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : null;
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;
        
        // Firestore path helpers
        const getRegistrationCollectionPath = () => 
            `/artifacts/${appId}/public/data/tournament_registrations`;
        
        const getConfigDocPath = () =>
            `/artifacts/${appId}/public/data/tournament_config/main_config`;

        /**
         * Initializes Firebase and authenticates the user.
         */
        async function initFirebase() {
            // Note: When deployed outside this environment, you would need to manually 
            // replace __app_id, __firebase_config, and __initial_auth_token with real values.
            if (!firebaseConfig) {
                console.error("Firebase config is missing. Cannot initialize.");
                document.getElementById('content-area').innerHTML = `<p class="text-center text-red-500">Error: Firebase configuration missing.</p>`;
                return;
            }

            try {
                // setLogLevel('debug'); 
                app = initializeApp(firebaseConfig);
                db = getFirestore(app);
                auth = getAuth(app);

                await new Promise((resolve) => {
                    const setupAuth = async () => {
                        if (initialAuthToken) {
                            try {
                                await signInWithCustomToken(auth, initialAuthToken);
                            } catch (error) {
                                console.warn("Custom token sign-in failed, falling back to anonymous:", error);
                                await signInAnonymously(auth);
                            }
                        } else {
                            await signInAnonymously(auth);
                        }
                        resolve();
                    };
                    setupAuth();
                });
                
                onAuthStateChanged(auth, (user) => {
                    // Set userId using auth uid if available, otherwise a random UUID
                    userId = user ? user.uid : crypto.randomUUID();
                    isAuthReady = true;
                    setupConfigListener(); 
                });
                
                const container = document.getElementById('app-container');
                // Remove the cinematic animation class after a delay
                setTimeout(() => {
                    container.classList.remove('falling-animation');
                }, 4100); 
                
            } catch (error) {
                console.error("Error initializing Firebase:", error);
                document.getElementById('content-area').innerHTML = `<p class="text-center text-red-500">Error initializing Firebase. Check console for details.</p>`;
            }
        }

        /**
         * Sets up a real-time listener for the global tournament configuration.
         */
        function setupConfigListener() {
            if (!db) return;
            if (unsubscribeConfig) unsubscribeConfig(); 

            unsubscribeConfig = onSnapshot(doc(db, getConfigDocPath()), (docSnapshot) => {
                if (docSnapshot.exists()) {
                    const fetchedConfig = docSnapshot.data();
                    config = { 
                        ...config, 
                        ...fetchedConfig,
                        // Ensure paymentMethods is an array if fetched (or use default if missing)
                        paymentMethods: Array.isArray(fetchedConfig.paymentMethods) ? fetchedConfig.paymentMethods : DEFAULT_PAYMENT_METHODS
                    };
                } else {
                    console.log("Config document not found. Initializing...");
                    saveConfig(config); // Initialize with defaults if not found
                }
                renderApp(); 
            }, (error) => {
                console.error("Error listening to config:", error);
                renderApp();
            });
        }

        /**
         * Renders the appropriate view based on the current state.
         */
        function renderApp() {
            if (!isAuthReady) return; 

            const contentArea = document.getElementById('content-area');
            const navButton = document.getElementById('nav-button');
            
            // Cleanup registration listener when leaving the dashboard
            if (currentView !== 'admin-dashboard' && unsubscribeRegistrations) {
                unsubscribeRegistrations();
                unsubscribeRegistrations = null;
            }

            if (currentView === 'register') {
                contentArea.innerHTML = getRegistrationHTML();
                navButton.textContent = 'Admin Login';
            } else if (currentView === 'confirmation') {
                contentArea.innerHTML = getConfirmationHTML();
                navButton.textContent = 'Go Back';
            } else if (currentView === 'admin-login') {
                contentArea.innerHTML = getAdminLoginHTML();
                navButton.textContent = 'Go Back';
            } else if (currentView === 'admin-dashboard') {
                contentArea.innerHTML = getAdminDashboardHTML();
                navButton.textContent = 'Logout';
                setupRegistrationListener();
                initTempPaymentMethods(); // Initialize temp state when entering dashboard
                // Use a slight delay to ensure the DOM element exists before rendering the editor
                setTimeout(renderAdminPaymentEditor, 100); 
            }
        }

        /**
         * Switches the view between registration, confirmation, and admin login/dashboard.
         */
        window.switchView = function() {
            if (currentView === 'register' || currentView === 'confirmation') {
                isRegistered = false;
                currentView = 'admin-login';
            } else if (currentView === 'admin-login') {
                currentView = 'register';
            } else if (currentView === 'admin-dashboard') {
                currentView = 'register'; 
            }
            renderApp();
        };

        // --- View HTML Generators ---

        function getRegistrationHTML() {
            // Options for the Payment Platform dropdown
            const paymentOptions = config.paymentMethods.map(method => `<option value="${method.name}">${method.name}</option>`).join('');
            
            // List of Payment Methods for the user to see
            const paymentMethodsList = config.paymentMethods.map(method => `
                <div class="bg-[#172a3a] p-3 rounded-lg border border-[#00eaff]/30 shadow-md">
                    <p class="font-bold text-lg text-ff7f00">${method.name} GATEWAY</p>
                    <pre class="whitespace-pre-wrap text-sm text-gray-400 mt-1">${method.details}</pre>
                </div>
            `).join('');

            return `
                <div class="text-center">
                    <h2 class="text-3xl md:text-4xl font-extrabold text-glow-blue mb-2">BAD CLAN OPEN TOURNAMENT</h2>
                    <p class="text-lg text-gray-400 mb-8">TDM AND WOW ROOMS | COMPETITIVE ACCESS</p>
                    
                    <!-- Reward Section -->
                    <div class="internal-card p-6 md:p-8 rounded-xl mb-8 border-b-4 border-ff7f00 shadow-2xl">
                        <h3 class="text-3xl font-bold text-glow-orange uppercase mb-4">REWARDS FOR WINNER</h3>
                        <div class="h-24 w-full bg-[#030a10] border-2 border-dashed border-[#ff7f00]/50 flex items-center justify-center rounded-lg">
                            <p class="text-md font-semibold text-gray-600 italic">REWARDS WILL BE ANNOUNCED SOON</p>
                        </div>
                    </div>

                    <!-- Registration Form -->
                    <form id="registration-form" onsubmit="event.preventDefault(); handleRegistration(event);">
                        
                        <!-- Player Info Input -->
                        <div class="internal-card p-6 rounded-xl shadow-xl mb-8">
                            <h4 class="text-xl font-semibold text-E0F2F1 mb-6 border-b border-[#3d4b63] pb-3">PLAYER REGISTRATION INPUT</h4>
                            <div class="space-y-5 text-left">
                                <div>
                                    <label for="name" class="block text-sm font-medium text-gray-400">Player/Team Name <span class="text-red-400">*</span></label>
                                    <input type="text" id="name" required class="input-style mt-1 block w-full p-3 rounded-lg" placeholder="Enter name">
                                </div>
                                <div>
                                    <label for="gameId" class="block text-sm font-medium text-gray-400">In-Game ID (UID) <span class="text-red-400">*</span></label>
                                    <input type="text" id="gameId" required class="input-style mt-1 block w-full p-3 rounded-lg" placeholder="Enter your unique game ID">
                                </div>
                                <div>
                                    <label for="contact" class="block text-sm font-medium text-gray-400">Contact Number/Discord ID <span class="text-red-400">*</span></label>
                                    <input type="text" id="contact" required class="input-style mt-1 block w-full p-3 rounded-lg" placeholder="e.g., +92 XXXXXXXXXX or Gamer#1234">
                                </div>
                            </div>
                        </div>

                        <!-- Payment Section -->
                        <div class="internal-card p-6 rounded-xl shadow-xl mb-8 border border-[#00eaff]/30">
                            <h4 class="text-xl font-semibold text-E0F2F1 mb-6 border-b border-[#3d4b63] pb-3 text-glow-blue">FEE & PAYMENT CONFIRMATION</h4>

                            <!-- Entry Fee Display -->
                            <div class="flex justify-between items-center mt-4 p-4 mb-6 bg-[#030a10] rounded-xl border border-ff7f00/50 shadow-lg shadow-ff7f00/20">
                                <span class="text-xl font-bold text-glow-orange">ENTRY FEE:</span>
                                <span class="text-2xl font-extrabold text-E0F2F1 text-glow-blue">${config.entryFee}</span>
                                <input type="hidden" id="entryFee" value="${config.entryFee}">
                            </div>

                            <!-- Payment Methods List -->
                            <div class="space-y-4 mb-8">
                                ${paymentMethodsList}
                            </div>
                            
                            <!-- Payment Proof Input -->
                            <h5 class="text-lg font-semibold text-E0F2F1 mb-4 border-t border-[#3d4b63] pt-4">SUBMIT PAYMENT DETAILS</h5>
                            <div class="space-y-5 text-left">
                                <div>
                                    <label for="paymentMethodUsed" class="block text-sm font-medium text-gray-400">Payment Platform Used <span class="text-red-400">*</span></label>
                                    <select id="paymentMethodUsed" required class="input-style mt-1 block w-full p-3 rounded-lg">
                                        <option value="" disabled selected>Select method used...</option>
                                        ${paymentOptions}
                                    </select>
                                </div>
                                <div>
                                    <label for="transactionId" class="block text-sm font-medium text-gray-400">Transaction ID/Reference No. <span class="text-red-400">*</span></label>
                                    <input type="text" id="transactionId" required class="input-style mt-1 block w-full p-3 rounded-lg" placeholder="e.g., TPC-123456789 (Must be accurate)">
                                </div>
                            </div>
                        </div>


                        <button type="submit" class="btn-primary w-full py-4 mt-6 rounded-lg text-lg">
                            <span id="reg-button-text">CONFIRM REGISTRATION</span>
                        </button>
                        <p id="reg-message" class="mt-4 text-center text-sm font-medium text-red-400"></p>
                    </form>
                </div>
            `;
        }
        
        function getConfirmationHTML() {
            return `
                <div class="text-center p-8 internal-card rounded-xl shadow-2xl border border-00eaff/50">
                    <svg class="mx-auto h-16 w-16 text-green-400 mb-4 text-glow-blue" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z" />
                    </svg>
                    <h2 class="text-3xl font-bold text-green-400 mb-2 text-glow-blue">Registration Successful!</h2>
                    <p class="text-xl font-semibold text-E0F2F1 mb-6">${config.thankYouMessage}</p>
                    
                    <div class="mt-8 text-left p-5 bg-[#030a10] rounded-lg border border-gray-700">
                        <h3 class="text-lg font-bold text-glow-orange mb-2">TOURNAMENT SCHEDULE & TIMING</h3>
                        <pre class="whitespace-pre-wrap text-sm text-gray-300 p-2 bg-gray-900 rounded-md">${config.schedule}</pre>
                    </div>

                    <button onclick="currentView = 'register'; isRegistered = false; renderApp();" class="btn-primary py-3 px-8 mt-8 rounded-lg">
                        Return to Home Console
                    </button>
                </div>
            `;
        }

        function getAdminLoginHTML() {
            return `
                <div class="max-w-md mx-auto p-8 internal-card rounded-xl shadow-2xl border border-gray-700">
                    <h2 class="text-3xl font-bold text-E0F2F1 mb-6 text-center text-glow-blue">ADMIN ACCESS PANEL</h2>
                    <form id="admin-login-form" onsubmit="event.preventDefault(); handleAdminLogin(event);">
                        <div>
                            <label for="admin-password" class="block text-sm font-medium text-gray-400 mb-2">AUTH CODE</label>
                            <input type="password" id="admin-password" required class="input-style mt-1 block w-full p-4 rounded-lg" placeholder="Enter secure password">
                        </div>
                        <button type="submit" class="btn-primary w-full py-3 mt-6 rounded-lg">
                            INITIATE LOGIN SEQUENCE
                        </button>
                        <p id="admin-message" class="mt-4 text-center text-sm font-medium"></p>
                    </form>
                    <p class="text-center text-xs text-gray-500 mt-4">Password is <code>Teddy786</code>.</p>
                </div>
            `;
        }

        function getAdminDashboardHTML() {
            const tableRows = registrations.length > 0
                ? registrations.map((reg, index) => `
                    <tr class="border-b border-gray-800 hover:bg-[#111827] transition duration-150">
                        <td class="p-3 whitespace-nowrap text-gray-400">${index + 1}</td>
                        <td class="p-3 font-medium text-E0F2F1">${reg.name}</td>
                        <td class="p-3 text-00eaff font-mono text-sm">${reg.gameId}</td>
                        <td class="p-3 text-sm">${reg.contact}</td>
                        <td class="p-3 font-medium text-ff7f00">${reg.payment?.method || 'N/A'}</td>
                        <td class="p-3 text-00eaff font-mono text-xs">${reg.payment?.transactionId || 'Pending'}</td>
                        <td class="p-3 text-xs text-gray-500">${new Date(reg.timestamp?.toDate ? reg.timestamp.toDate() : Date.now()).toLocaleString()}</td>
                    </tr>
                  `).join('')
                : `<tr><td colspan="7" class="p-6 text-center text-gray-500">No registrations found yet.</td></tr>`;

            return `
                <div class="p-2">
                    <h2 class="text-3xl font-bold text-E0F2F1 mb-6 text-glow-orange border-b pb-2 border-gray-700">ADMIN CONTROL PANEL</h2>
                    
                    <!-- Configuration Panel -->
                    <div class="internal-card p-6 rounded-xl shadow-lg mb-8 border border-ff7f00/30">
                        <h3 class="text-xl font-bold text-glow-orange mb-4 border-b border-gray-700 pb-2">Tournament Configuration Access</h3>
                        <form id="config-form" onsubmit="event.preventDefault(); handleSaveConfig(event);">
                            <div class="space-y-4">
                                <div>
                                    <label for="edit-entryFee" class="block text-sm font-medium text-gray-400">Entry Fee (e.g., 200 PKR)</label>
                                    <input type="text" id="edit-entryFee" value="${config.entryFee}" required class="input-style mt-1 block w-full p-3 rounded-lg">
                                </div>
                                <div>
                                    <label for="edit-thankYouMessage" class="block text-sm font-medium text-gray-400">Registration Thank You Message</label>
                                    <textarea id="edit-thankYouMessage" rows="2" required class="input-style mt-1 block w-full p-3 rounded-lg">${config.thankYouMessage}</textarea>
                                </div>
                                <div>
                                    <label for="edit-schedule" class="block text-sm font-medium text-gray-400">Schedule & Timing Details</label>
                                    <textarea id="edit-schedule" rows="5" required class="input-style mt-1 block w-full p-3 rounded-lg">${config.schedule}</textarea>
                                </div>
                            </div>
                            <button type="submit" class="btn-primary py-3 px-6 mt-6 rounded-lg">
                                <span id="config-button-text">SAVE CORE CONFIG</span>
                            </button>
                            <p id="config-message" class="mt-4 text-sm font-medium text-center"></p>
                        </form>
                    </div>
                    
                    <!-- Payment Method Configuration Panel -->
                    <div class="internal-card p-6 rounded-xl shadow-lg mb-8 border border-00eaff/30">
                        <h3 class="text-xl font-bold text-glow-blue mb-4 border-b border-gray-700 pb-2">Payment Method Management</h3>
                        <div id="payment-methods-editor" class="space-y-4 mb-4">
                            <!-- Payment methods will be rendered here by renderAdminPaymentEditor() -->
                        </div>
                        <div class="flex flex-col sm:flex-row justify-between items-center space-y-3 sm:space-y-0">
                            <button onclick="addPaymentMethodTemp()" class="py-2 px-4 rounded-lg text-sm bg-gray-600 hover:bg-gray-700 text-E0F2F1 transition duration-200 w-full sm:w-auto" type="button">
                                + Add New Payment Method
                            </button>
                            <button onclick="handleSavePaymentMethods()" class="btn-primary py-2 px-6 rounded-lg text-sm w-full sm:w-auto" type="button">
                                SAVE PAYMENT METHODS
                            </button>
                        </div>
                        <p id="payment-config-message" class="mt-4 text-sm font-medium text-center"></p>
                    </div>


                    <!-- Registrations Table -->
                    <h3 class="text-xl font-bold text-glow-blue mb-4 border-b pb-2 border-gray-700">PLAYER REGISTRY (${registrations.length} TOTAL)</h3>
                    <div class="overflow-x-auto shadow-lg rounded-xl internal-card">
                        <table class="min-w-full divide-y divide-gray-700">
                            <thead class="bg-[#111827]">
                                <tr>
                                    <th class="p-3 text-left text-xs font-semibold uppercase tracking-wider text-gray-300">#</th>
                                    <th class="p-3 text-left text-xs font-semibold uppercase tracking-wider text-gray-300">Name</th>
                                    <th class="p-3 text-left text-xs font-semibold uppercase tracking-wider text-gray-300">Game ID</th>
                                    <th class="p-3 text-left text-xs font-semibold uppercase tracking-wider text-gray-300">Contact</th>
                                    <th class="p-3 text-left text-xs font-semibold uppercase tracking-wider text-gray-300">Payment Method</th>
                                    <th class="p-3 text-left text-xs font-semibold uppercase tracking-wider text-gray-300">Transaction ID</th>
                                    <th class="p-3 text-left text-xs font-semibold uppercase tracking-wider text-gray-300">Time</th>
                                </tr>
                            </thead>
                            <tbody id="registration-table-body" class="divide-y divide-gray-800">
                                ${tableRows}
                            </tbody>
                        </table>
                    </div>
                </div>
            `;
        }

        // --- Firebase Interaction Functions ---

        /**
         * Saves the current configuration state to Firestore.
         */
        async function saveConfig(newConfig) {
            const docRef = doc(db, getConfigDocPath());
            try {
                // Remove any empty objects/nulls from paymentMethods before saving
                if (newConfig.paymentMethods) {
                    newConfig.paymentMethods = newConfig.paymentMethods.filter(m => m.name && m.details);
                }
                await setDoc(docRef, newConfig, { merge: true });
                return true;
            } catch (e) {
                console.error("Error saving config:", e);
                return false;
            }
        }

        /**
         * Handles the saving of editable configuration fields by the admin (Fee, Schedule, Message).
         */
        window.handleSaveConfig = async function(event) {
            const form = event.target;
            const button = document.getElementById('config-button-text');
            const message = document.getElementById('config-message');

            const newEntryFee = form.elements['edit-entryFee'].value.trim();
            const newSchedule = form.elements['edit-schedule'].value.trim();
            const newThankYouMessage = form.elements['edit-thankYouMessage'].value.trim();
            
            const newCoreConfig = {
                entryFee: newEntryFee,
                schedule: newSchedule,
                thankYouMessage: newThankYouMessage
            };

            button.textContent = 'SAVING...';
            button.disabled = true;
            message.textContent = '';

            const success = await saveConfig(newCoreConfig);

            if (success) {
                message.className = 'mt-4 text-sm font-medium text-green-400';
                message.textContent = 'Core configuration saved successfully!';
            } else {
                message.className = 'mt-4 text-sm font-medium text-red-400';
                message.textContent = 'Error saving core configuration.';
            }

            button.textContent = 'SAVE CORE CONFIG';
            button.disabled = false;
        };

        /**
         * Initializes the temporary payment method state for the admin editor.
         */
        function initTempPaymentMethods() {
            // Deep copy the current config when entering the dashboard
            tempPaymentMethods = JSON.parse(JSON.stringify(config.paymentMethods || []));
        }

        /**
         * Renders the dynamic payment methods editor in the admin panel.
         */
        window.renderAdminPaymentEditor = function() {
            const editorContainer = document.getElementById('payment-methods-editor');
            if (!editorContainer) return; 

            editorContainer.innerHTML = tempPaymentMethods.map((method, index) => `
                <div class="flex flex-col md:flex-row space-y-2 md:space-y-0 md:space-x-2 bg-[#172a3a] p-3 rounded-lg border border-gray-700">
                    <input type="text" placeholder="Name (e.g., JazzCash)" value="${method.name}" 
                           oninput="updatePaymentMethodTemp(${index}, 'name', this.value)" 
                           class="input-style flex-1 p-2 rounded-lg text-sm" required>
                    <input type="text" placeholder="Details (e.g., 0300-1234567)" value="${method.details}" 
                           oninput="updatePaymentMethodTemp(${index}, 'details', this.value)" 
                           class="input-style flex-2 p-2 rounded-lg text-sm" required>
                    <button onclick="removePaymentMethodTemp(${index})" 
                            class="bg-red-600 hover:bg-red-700 text-white font-bold py-1 px-3 rounded-lg text-sm transition duration-200 w-full md:w-auto" 
                            type="button">
                        Remove
                    </button>
                </div>
            `).join('');
        };

        window.addPaymentMethodTemp = function() {
            tempPaymentMethods.push({ name: '', details: '' });
            renderAdminPaymentEditor();
        };

        window.removePaymentMethodTemp = function(index) {
            tempPaymentMethods.splice(index, 1);
            renderAdminPaymentEditor();
        };

        window.updatePaymentMethodTemp = function(index, key, value) {
            if (tempPaymentMethods[index]) {
                tempPaymentMethods[index][key] = value;
            }
        };

        /**
         * Saves the temporary payment methods state to the Firestore config.
         */
        window.handleSavePaymentMethods = async function() {
            const message = document.getElementById('payment-config-message');
            const saveButton = document.querySelector('[onclick="handleSavePaymentMethods()"]');

            // Validation: Filter out incomplete methods for saving
            const validMethods = tempPaymentMethods.filter(m => m.name.trim() && m.details.trim());
            
            if (validMethods.length === 0) {
                message.className = 'mt-4 text-sm font-medium text-red-400';
                message.textContent = 'Error: You must have at least one valid payment method.';
                return;
            }

            saveButton.textContent = 'SAVING...';
            saveButton.disabled = true;
            message.textContent = '';

            const newConfig = { paymentMethods: validMethods };
            
            const success = await saveConfig(newConfig);

            if (success) {
                message.className = 'mt-4 text-sm font-medium text-green-400';
                message.textContent = 'Payment methods saved successfully! (Refreshing...)';
                // Force a config refresh (which calls renderApp)
                setupConfigListener(); 
            } else {
                message.className = 'mt-4 text-sm font-medium text-red-400';
                message.textContent = 'Error saving payment methods.';
            }

            saveButton.textContent = 'SAVE PAYMENT METHODS';
            saveButton.disabled = false;
        };


        /**
         * Handles player registration form submission.
         */
        window.handleRegistration = async function(event) {
            const form = event.target;
            const button = document.getElementById('reg-button-text');
            const message = document.getElementById('reg-message');

            const name = form.elements['name'].value.trim();
            const gameId = form.elements['gameId'].value.trim();
            const contact = form.elements['contact'].value.trim();
            const entryFee = form.elements['entryFee'].value; 
            const transactionId = form.elements['transactionId'].value.trim();
            const paymentMethodUsed = form.elements['paymentMethodUsed'].value.trim();
            
            if (!db || !isAuthReady) {
                message.textContent = 'Application is still loading. Please wait a moment.';
                return;
            }

            button.textContent = 'REGISTERING...';
            button.disabled = true;
            message.textContent = '';

            try {
                // Write registration data to the public collection
                await addDoc(collection(db, getRegistrationCollectionPath()), {
                    name: name,
                    gameId: gameId,
                    contact: contact,
                    entryFeePaid: entryFee,
                    payment: {
                        method: paymentMethodUsed,
                        transactionId: transactionId
                    },
                    timestamp: serverTimestamp(),
                    registeredBy: userId 
                });

                isRegistered = true;
                currentView = 'confirmation';
                renderApp();

                form.reset();
            } catch (e) {
                console.error("Error adding document: ", e);
                message.textContent = 'Registration failed. Please try again.';
            } finally {
                button.textContent = 'CONFIRM REGISTRATION';
                button.disabled = false;
            }
        };

        /**
         * Handles the admin login attempt.
         */
        window.handleAdminLogin = function(event) {
            const form = event.target;
            const password = form.elements['admin-password'].value;
            const message = document.getElementById('admin-message');
            
            if (password === ADMIN_PASSWORD) {
                message.className = 'mt-4 text-center text-sm font-medium text-green-400';
                message.textContent = 'Access granted! Loading dashboard...';
                
                setTimeout(() => {
                    currentView = 'admin-dashboard';
                    renderApp();
                }, 500);
            } else {
                message.className = 'mt-4 text-center text-sm font-medium text-red-500';
                message.textContent = 'ACCESS DENIED. Invalid AUTH CODE.';
                form.elements['admin-password'].value = '';
            }
        };

        /**
         * Sets up a real-time listener for registrations when in admin view.
         */
        function setupRegistrationListener() {
            if (!db || !isAuthReady) return;
            if (unsubscribeRegistrations) unsubscribeRegistrations(); 

            const q = query(collection(db, getRegistrationCollectionPath()));

            unsubscribeRegistrations = onSnapshot(q, (snapshot) => {
                const fetchedRegistrations = [];
                snapshot.forEach((doc) => {
                    const data = doc.data();
                    if (!data.timestamp) {
                        data.timestamp = { toDate: () => new Date() }; 
                    }
                    fetchedRegistrations.push(data);
                });
                
                // Sort by timestamp descending (newest first)
                fetchedRegistrations.sort((a, b) => {
                    const timeA = a.timestamp.toDate().getTime();
                    const timeB = b.timestamp.toDate().getTime();
                    return timeB - a.timeB;
                });
                
                registrations = fetchedRegistrations;
                if (currentView === 'admin-dashboard') {
                    renderApp(); // Re-render the dashboard with new data
                }
            }, (error) => {
                console.error("Error listening to registrations:", error);
            });
        }


        // --- Initialization ---
        initFirebase();
    </script>
</body>
</html>

