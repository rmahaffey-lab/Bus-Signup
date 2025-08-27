<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Staff Bus Sign-Up</title>
    <!-- Tailwind CSS CDN for styling -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f7f9fc;
        }
        .container {
            max-width: 800px;
        }
        .card {
            background-color: #ffffff;
            border-radius: 12px;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        }
        .bus-stop-card {
            background-color: #f0f4f8;
            border-left: 4px solid #3b82f6;
        }
        .occupied {
            background-color: #fca5a5;
        }
        .signed-up {
            background-color: #6ee7b7;
        }
        .bus-list {
            max-height: 400px;
            overflow-y: auto;
        }
        .modal {
            background-color: rgba(0, 0, 0, 0.5);
        }
    </style>
</head>
<body>

    <div id="app" class="flex flex-col items-center justify-center min-h-screen p-4">
        <div id="loading" class="text-center text-gray-600">
            <h1 class="text-3xl font-bold text-gray-800">Loading App...</h1>
            <p class="mt-2">Please wait while we connect to the database.</p>
        </div>

        <div id="main-content" class="hidden w-full container mx-auto">
            <div id="auth-ui" class="flex flex-col items-center justify-center min-h-screen">
                <div class="card p-8 w-full max-w-sm text-center">
                    <h1 class="text-2xl font-bold text-gray-800 mb-4">Staff Bus Sign-Up</h1>
                    <p class="text-sm text-gray-500 mb-6">Choose how you would like to sign in.</p>
                    <div class="flex flex-col gap-4">
                        <button id="signInGoogleBtn" class="w-full bg-blue-600 text-white font-semibold py-2 px-4 rounded-md hover:bg-blue-700 transition duration-300">
                            Sign in with Google
                        </button>
                        <button id="signInAnonymousBtn" class="w-full bg-gray-600 text-white font-semibold py-2 px-4 rounded-md hover:bg-gray-700 transition duration-300">
                            Sign in Anonymously
                        </button>
                    </div>
                    <div id="error-message" class="text-red-500 text-sm mt-4 hidden"></div>
                </div>
            </div>

            <div id="app-ui" class="hidden flex flex-col gap-6">
                <header class="card p-6 flex flex-col sm:flex-row items-center justify-between">
                    <div class="text-center sm:text-left">
                        <h1 class="text-2xl font-bold text-gray-800">Staff Bus Sign-Up</h1>
                        <p id="user-info" class="text-sm text-gray-500 mt-1"></p>
                    </div>
                    <button id="signOutBtn" class="mt-4 sm:mt-0 bg-gray-200 text-gray-700 font-semibold py-2 px-4 rounded-md hover:bg-gray-300 transition duration-300">
                        Sign Out
                    </button>
                </header>

                <main class="grid md:grid-cols-2 gap-6">
                    <!-- Morning Bus Run -->
                    <div id="morning-run" class="card p-6 flex flex-col">
                        <h2 class="text-xl font-bold mb-4 text-gray-700">Morning Run</h2>
                        <div class="bus-list flex flex-col gap-3">
                            <!-- Bus Stops for Morning Run will be inserted here -->
                        </div>
                    </div>

                    <!-- Afternoon Bus Run -->
                    <div id="afternoon-run" class="card p-6 flex flex-col">
                        <h2 class="text-xl font-bold mb-4 text-gray-700">Afternoon Run</h2>
                        <div class="bus-list flex flex-col gap-3">
                            <!-- Bus Stops for Afternoon Run will be inserted here -->
                        </div>
                    </div>
                </main>
            </div>
        </div>

        <!-- Custom Modal for Name Input -->
        <div id="name-modal" class="modal fixed inset-0 flex items-center justify-center p-4 hidden">
            <div class="card p-8 w-full max-w-sm">
                <h3 class="text-xl font-bold text-gray-800 mb-4">Enter Your Name</h3>
                <p class="text-sm text-gray-500 mb-4">Your name will be visible to others on the sign-up sheet.</p>
                <input type="text" id="name-input" class="w-full p-2 border rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500 mb-4" placeholder="e.g. John Doe">
                <div class="flex justify-end gap-2">
                    <button id="cancel-name-btn" class="bg-gray-200 text-gray-700 font-semibold py-2 px-4 rounded-md hover:bg-gray-300 transition duration-300">
                        Cancel
                    </button>
                    <button id="confirm-name-btn" class="bg-blue-600 text-white font-semibold py-2 px-4 rounded-md hover:bg-blue-700 transition duration-300">
                        Sign Up
                    </button>
                </div>
            </div>
        </div>
    </div>

    <!-- Firebase SDKs -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAnalytics } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-analytics.js";
        import { getAuth, signInWithRedirect, GoogleAuthProvider, signOut, onAuthStateChanged, signInWithCustomToken, signInAnonymously, getRedirectResult } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, onSnapshot, collection, setDoc, updateDoc, FieldValue } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        let db, auth;
        let userId;

        // Your web app's Firebase configuration - HARDCODE IT HERE AS A FALLBACK
        const YOUR_FIREBASE_CONFIG = {
            apiKey: "AIzaSyChx1hUc5JhsNSvFCJqCFo4pAeUAbJC-O0", // Your actual API Key
            authDomain: "staff-route-sign-up.firebaseapp.com",
            projectId: "staff-route-sign-up",
            storageBucket: "staff-route-sign-up.appspot.com", // Corrected storageBucket
            messagingSenderId: "583733533491",
            appId: "1:583733533491:web:0419cc007b0c7a262abc46",
            measurementId: "G-MD7GNJ10GB" // If you have Google Analytics configured
        };

        // Use provided config if available (e.g., from a codelab), otherwise use your hard-coded one
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : YOUR_FIREBASE_CONFIG;

        // Ensure appId also falls back to your project ID
        const appId = typeof __app_id !== 'undefined' ? __app_id : YOUR_FIREBASE_CONFIG.projectId; // Or 'staff-route-sign-up' if you prefer

        // Ensure initialAuthToken is null if not provided
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

        // Bus stops
        const busStops = [
            'Hwy 133 park and ride',
            'Catherine Store RFTA',
            'El Jebel RFTA',
            'Willits RFTA',
            'Basalt RFTA',
            'Old Snowmass RFTA',
            'Aspen Village RFTA',
            'Intercept Lot (Brush Creek Park N\' Ride)',
            'ABC RFTA'
        ];

        // Function to sanitize text to be used as a document ID
        const sanitizeForId = (text) => text.toLowerCase().replace(/[^a-z0-9]/g, '');

        // Initialize Firebase and Authentication
        async function initializeFirebase() {
            try {
                // Check if Firebase config is available before initializing
                if (!firebaseConfig) {
                    throw new Error("Firebase configuration not found or is empty. Please ensure the app is configured correctly.");
                }

                const app = initializeApp(firebaseConfig);
                const analytics = getAnalytics(app);
                db = getFirestore(app);
                auth = getAuth(app);

                const redirectResult = await getRedirectResult(auth);
                if (redirectResult && redirectResult.user) {
                    console.log("Signed in with redirect:", redirectResult.user);
                } else if (initialAuthToken) {
                    await signInWithCustomToken(auth, initialAuthToken);
                }

                onAuthStateChanged(auth, async (user) => {
                    document.getElementById('loading').classList.add('hidden');
                    document.getElementById('main-content').classList.remove('hidden');

                    if (user) {
                        userId = user.uid;
                        const name = user.displayName || 'Anonymous User';
                        document.getElementById('user-info').textContent = `Signed in as ${name} (ID: ${userId})`;

                        document.getElementById('auth-ui').classList.add('hidden');
                        document.getElementById('app-ui').classList.remove('hidden');
                        
                        await setupRealtimeListener();

                    } else {
                        document.getElementById('auth-ui').classList.remove('hidden');
                        document.getElementById('app-ui').classList.add('hidden');
                    }
                });

            } catch (error) {
                console.error("Error initializing Firebase:", error);
                document.getElementById('error-message').textContent = "Error initializing app: " + error.message;
                document.getElementById('error-message').classList.remove('hidden');
                document.getElementById('loading').classList.add('hidden');
                document.getElementById('main-content').classList.remove('hidden');
            }
        }

        // Setup real-time listeners for bus sign-ups
        async function setupRealtimeListener() {
            if (!db || !auth || !auth.currentUser) {
                console.error("Database or authentication not ready.");
                return;
            }

            const staffId = auth.currentUser.uid;
            
            // Reference to the public sign-up data collection
            const morningRunRef = collection(db, `artifacts/${appId}/public/data/morning_run`);
            const afternoonRunRef = collection(db, `artifacts/${appId}/public/data/afternoon_run`);
            
            // Listener for morning run
            onSnapshot(morningRunRef, (snapshot) => {
                const morningRuns = {};
                snapshot.forEach(doc => {
                    morningRuns[doc.id] = doc.data();
                });
                renderBusStops('morning-run', morningRuns, staffId, 'morning_run');
            });
            
            // Listener for afternoon run
            onSnapshot(afternoonRunRef, (snapshot) => {
                const afternoonRuns = {};
                snapshot.forEach(doc => {
                    afternoonRuns[doc.id] = doc.data();
                });
                renderBusStops('afternoon-run', afternoonRuns, staffId, 'afternoon_run');
            });
        }

        // Render bus stops and sign-up buttons
        function renderBusStops(containerId, data, staffId, runType) {
            const container = document.getElementById(containerId).querySelector('.bus-list');
            container.innerHTML = '';
            
            busStops.forEach(stop => {
                const docId = sanitizeForId(stop);
                const busData = data[docId] || {};
                const signedUpUser = Object.values(busData).find(user => user.uid === staffId);
                const isSignedUp = signedUpUser !== undefined;
                const isFull = Object.keys(busData).length >= 14;

                const cardClass = isSignedUp ? 'signed-up' : (isFull ? 'occupied' : 'bus-stop-card');

                const card = document.createElement('div');
                card.className = `flex flex-col p-4 rounded-lg transition-all duration-300 ${cardClass}`;
                
                const stopName = document.createElement('div');
                stopName.className = `font-bold text-gray-800 text-sm md:text-base`;
                stopName.textContent = stop;
                
                const occupancy = document.createElement('div');
                occupancy.className = `text-xs text-gray-600 mt-1`;
                occupancy.textContent = `${Object.keys(busData).length} / 14 seats`;

                const occupantsList = document.createElement('ul');
                occupantsList.className = 'list-disc list-inside text-xs text-gray-700 mt-2';
                Object.values(busData).forEach(user => {
                    const li = document.createElement('li');
                    li.textContent = user.name || 'Anonymous User';
                    occupantsList.appendChild(li);
                });

                const button = document.createElement('button');
                button.className = `mt-3 py-2 px-4 rounded-md font-semibold transition duration-300`;
                
                if (isSignedUp) {
                    button.textContent = 'Cancel Sign-Up';
                    button.classList.add('bg-red-500', 'text-white', 'hover:bg-red-600');
                    button.onclick = () => removeSignUp(runType, docId, staffId);
                } else if (isFull) {
                    button.textContent = 'Full';
                    button.disabled = true;
                    button.classList.add('bg-gray-400', 'text-gray-200', 'cursor-not-allowed');
                } else {
                    button.textContent = 'Sign Up';
                    button.classList.add('bg-green-500', 'text-white', 'hover:bg-green-600');
                    button.onclick = () => showNameModal(runType, docId);
                }

                card.appendChild(stopName);
                card.appendChild(occupancy);
                card.appendChild(occupantsList);
                card.appendChild(button);
                container.appendChild(card);
            });
        }

        let pendingSignUpData = null;

        function showNameModal(runType, docId) {
            pendingSignUpData = { runType, docId };
            document.getElementById('name-modal').classList.remove('hidden');
        }

        document.getElementById('confirm-name-btn').addEventListener('click', () => {
            const nameInput = document.getElementById('name-input');
            const name = nameInput.value.trim();
            if (name) {
                addSignUp(pendingSignUpData.runType, pendingSignUpData.docId, auth.currentUser.uid, name);
                nameInput.value = '';
                document.getElementById('name-modal').classList.add('hidden');
            } else {
                nameInput.placeholder = 'Please enter your name!';
                nameInput.classList.add('border-red-500');
            }
        });

        document.getElementById('cancel-name-btn').addEventListener('click', () => {
            document.getElementById('name-input').value = '';
            document.getElementById('name-modal').classList.add('hidden');
        });

        // Function to add a sign-up
        async function addSignUp(runType, docId, staffId, staffName) {
            try {
                if (!db || !auth || !auth.currentUser) return;
                const docRef = doc(db, `artifacts/${appId}/public/data/${runType}/${docId}`);
                const data = { [staffId]: { uid: staffId, name: staffName, email: auth.currentUser.email || 'anonymous' } };
                await setDoc(docRef, data, { merge: true });
            } catch (e) {
                console.error("Error adding sign-up: ", e);
            }
        }

        // Function to remove a sign-up
        async function removeSignUp(runType, docId, staffId) {
            try {
                if (!db || !auth || !auth.currentUser) return;
                const docRef = doc(db, `artifacts/${appId}/public/data/${runType}/${docId}`);
                await updateDoc(docRef, {
                    [staffId]: FieldValue.delete()
                });
            } catch (e) {
                console.error("Error removing sign-up: ", e);
            }
        }

        // Event listeners for auth buttons
        document.getElementById('signInGoogleBtn').addEventListener('click', () => {
            const provider = new GoogleAuthProvider();
            signInWithRedirect(auth, provider);
        });

        document.getElementById('signInAnonymousBtn').addEventListener('click', () => {
            signInAnonymously(auth)
                .then(() => {
                    // Signed in anonymously
                }).catch((error) => {
                    const errorMessage = error.message;
                    console.error("Anonymous Sign-in Error:", errorMessage);
                    document.getElementById('error-message').textContent = errorMessage;
                    document.getElementById('error-message').classList.remove('hidden');
                });
        });

        document.getElementById('signOutBtn').addEventListener('click', () => {
            signOut(auth).then(() => {
                // Sign-out successful.
            }).catch((error) => {
                console.error("Sign-out error:", error);
            });
        });

        // Initialize on page load
        window.addEventListener('load', () => {
            initializeFirebase();
        });
    </script>

</body>
</html>
