<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>JAHSSBEATS Rhyme Finder</title>
    <!-- Load Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    fontFamily: {
                        sans: ['Inter', 'sans-serif'],
                    },
                    colors: {
                        'primary-dark': '#1e293b', // Slate 800
                        'secondary-dark': '#334155', // Slate 700
                        'accent': '#6366f1', // Indigo 500
                    }
                }
            }
        }
    </script>
    <style>
        /* Custom styles for a dark, rounded aesthetic */
        body {
            background-color: #0f172a; /* Slate 900 */
            color: #f8fafc; /* Slate 50 */
            font-family: 'Inter', sans-serif;
            display: flex;
            justify-content: center;
            align-items: flex-start;
            min-height: 100vh;
            padding: 2rem 1rem;
        }
        .container-card {
            max-width: 700px;
            width: 100%;
        }
        .result-item:nth-child(even) {
            background-color: #1e293b; /* primary-dark */
        }
    </style>
</head>
<body onload="initApp()">

    <div class="container-card bg-secondary-dark p-8 rounded-xl shadow-2xl">
        
        <h1 class="text-4xl font-extrabold text-white mb-2 text-center tracking-tight">
            JAHSSBEATS Rhyme Finder
        </h1>
        <p class="text-slate-400 mb-8 text-center">
            Enter a word to find perfect rhymes based on song lyrics.
        </p>

        <!-- Input and Control Area -->
        <div class="space-y-4">
            <input 
                type="text" 
                id="searchInput" 
                placeholder="Enter a word (e.g., 'cat', 'dream')"
                class="w-full px-5 py-3 text-lg bg-primary-dark border border-slate-600 rounded-lg focus:ring-accent focus:border-accent text-white transition duration-200"
            />
            <button 
                id="searchButton"
                onclick="findRhymes()"
                class="w-full bg-accent hover:bg-indigo-600 text-white font-bold py-3 px-6 rounded-lg shadow-md transition duration-200 transform hover:scale-[1.01] active:scale-[0.99] disabled:opacity-50"
            >
                Find Rhymes
            </button>
        </div>

        <!-- Loading Indicator -->
        <div id="loadingIndicator" class="hidden mt-8 text-center">
            <svg class="animate-spin h-6 w-6 text-accent mx-auto" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
            </svg>
            <p class="mt-2 text-slate-400">Searching and processing lyrics...</p>
        </div>

        <!-- Results Display -->
        <div id="resultsContainer" class="mt-8">
            <h3 class="text-xl font-semibold mb-4 border-b border-slate-600 pb-2">Results for: <span id="currentWord" class="text-accent font-extrabold"></span></h3>
            
            <!-- Message Box (replaces alert()) -->
            <div id="messageBox" class="hidden bg-red-800/30 border border-red-600 text-red-300 p-4 rounded-lg mb-4" role="alert"></div>

            <div id="rhymeList" class="bg-primary-dark rounded-lg divide-y divide-slate-700 shadow-lg">
                <p id="initialMessage" class="p-4 text-center text-slate-400">Enter a word above to begin your search.</p>
            </div>
        </div>
    </div>

    <script>
        // DOM Elements
        const searchInput = document.getElementById('searchInput');
        const searchButton = document.getElementById('searchButton');
        const loadingIndicator = document.getElementById('loadingIndicator');
        const rhymeList = document.getElementById('rhymeList');
        const initialMessage = document.getElementById('initialMessage');
        const currentWordSpan = document.getElementById('currentWord');
        const messageBox = document.getElementById('messageBox');

        // --- Core Application Logic ---

        /**
         * Simulates a server-side API call to find rhymes.
         * In a real application, this would be a 'fetch' call to your own server 
         * (e.g., a Python Flask server) to safely run the Genius API logic.
         * @param {string} word 
         * @returns {Promise<string[]>} An array of rhyming words.
         */
        function mockRhymeApi(word) {
            // Mock data simulating a successful API response
            const mockRhymeData = {
                'cat': ['hat', 'mat', 'sat', 'flat', 'combat'],
                'dream': ['team', 'scream', 'seem', 'stream', 'supreme'],
                'fire': ['higher', 'acquire', 'desire', 'entire', 'vampire'],
            };

            // Convert word to lowercase for consistent mock lookups
            const normalizedWord = word.toLowerCase();

            return new Promise((resolve, reject) => {
                // Simulate network latency (300ms to 1000ms)
                const delay = Math.random() * 700 + 300; 
                setTimeout(() => {
                    if (mockRhymeData[normalizedWord]) {
                        resolve(mockRhymeData[normalizedWord]);
                    } else if (normalizedWord.length < 3) {
                        reject('Please enter a word longer than two letters.');
                    } else {
                        // Simulate case where no rhymes are found or word is too obscure
                        resolve([]);
                    }
                }, delay);
            });
        }

        /**
         * Hides or displays the custom message box.
         * @param {string} message - The message to display.
         * @param {boolean} isError - If true, displays as an error.
         */
        function showMessage(message, isError = false) {
            messageBox.textContent = message;
            messageBox.classList.remove('hidden', 'bg-red-800/30', 'border-red-600', 'bg-green-800/30', 'border-green-600');
            
            if (isError) {
                messageBox.classList.add('bg-red-800/30', 'border-red-600', 'text-red-300');
            } else {
                messageBox.classList.add('bg-green-800/30', 'border-green-600', 'text-green-300');
            }
        }

        function hideMessage() {
            messageBox.classList.add('hidden');
        }

        /**
         * Main function to handle the rhyme search interaction.
         */
        async function findRhymes() {
            const searchWord = searchInput.value.trim();
            hideMessage(); // Clear previous messages

            if (searchWord === "") {
                showMessage("Please enter a word to search for rhymes.", true);
                return;
            }

            // UI State: Disable button, show loading
            searchButton.disabled = true;
            loadingIndicator.classList.remove('hidden');
            rhymeList.innerHTML = '';
            currentWordSpan.textContent = searchWord.toUpperCase();
            initialMessage.classList.add('hidden');

            try {
                // Call the simulated API (where your Python logic would live)
                const rhymes = await mockRhymeApi(searchWord);
                
                if (rhymes.length > 0) {
                    renderRhymes(rhymes);
                    showMessage(`Found ${rhymes.length} rhymes for '${searchWord}'!`, false);
                } else {
                    rhymeList.innerHTML = `<p class="p-4 text-center text-slate-400">No perfect rhymes found for '${searchWord}' in our mock dataset. Try 'cat', 'dream', or 'fire'.</p>`;
                }

            } catch (error) {
                // Handle API error or validation error from mock function
                showMessage(`Search Error: ${error}`, true);
            } finally {
                // UI State: Enable button, hide loading
                searchButton.disabled = false;
                loadingIndicator.classList.add('hidden');
            }
        }

        /**
         * Renders the list of rhymes to the results container.
         * @param {string[]} rhymes 
         */
        function renderRhymes(rhymes) {
            let html = '';
            rhymes.forEach((rhyme, index) => {
                html += `
                    <div class="result-item p-4 flex justify-between items-center transition duration-150">
                        <span class="text-lg font-medium text-white">${index + 1}. ${rhyme}</span>
                    </div>
                `;
            });
            rhymeList.innerHTML = html;
        }

        /**
         * Initializes the application and sets up event listeners.
         */
        function initApp() {
            // Set up Enter key listener on the input field
            searchInput.addEventListener('keypress', function(e) {
                if (e.key === 'Enter') {
                    findRhymes();
                }
            });
            // Initial state update
            currentWordSpan.textContent = '...';
        }

    </script>
</body>
</html>
