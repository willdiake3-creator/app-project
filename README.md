# app-project
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Global Uni Matcher</title>
    <!-- Tailwind CSS for styling -->
    <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-gray-50 text-gray-800 font-sans">

    <!-- Navigation Bar -->
    <nav class="bg-white shadow-sm p-4">
        <div class="max-w-6xl mx-auto flex justify-between items-center">
            <h1 class="text-2xl font-bold text-blue-600">UniMatch.</h1>
            <button class="bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700 transition">Log In</button>
        </div>
    </nav>

    <!-- Hero & Search Section -->
    <header class="bg-blue-50 py-16 text-center px-4">
        <div class="max-w-3xl mx-auto">
            <h2 class="text-4xl font-extrabold mb-4 text-gray-900">Find Your Perfect Program</h2>
            <p class="text-lg text-gray-600 mb-8">Filter by country, field of study, and your exact budget.</p>
            
            <!-- The Search Form (Wired with IDs) -->
            <form id="search-form" class="bg-white p-6 rounded-lg shadow-md flex flex-col md:flex-row gap-4 items-end">
                
                <div class="w-full md:w-1/3 text-left">
                    <label class="block text-sm font-medium text-gray-700 mb-1">Field of Study</label>
                    <select id="field-select" class="w-full border border-gray-300 rounded p-2 focus:ring-blue-500 focus:border-blue-500">
                        <option value="Computer Science">Computer Science</option>
                        <option value="Business">Business & Management</option>
                        <option value="Engineering">Engineering</option>
                    </select>
                </div>

                <div class="w-full md:w-1/3 text-left">
                    <label class="block text-sm font-medium text-gray-700 mb-1">Target Country</label>
                    <select id="country-select" class="w-full border border-gray-300 rounded p-2 focus:ring-blue-500 focus:border-blue-500">
                        <option value="Germany">Germany</option>
                        <option value="Canada">Canada</option>
                        <option value="Taiwan">Taiwan</option>
                    </select>
                </div>

                <div class="w-full md:w-1/3 text-left">
                    <label class="block text-sm font-medium text-gray-700 mb-1">Max Budget (USD/yr)</label>
                    <input id="budget-input" type="number" placeholder="e.g. 15000" required class="w-full border border-gray-300 rounded p-2 focus:ring-blue-500 focus:border-blue-500">
                </div>

                <button id="search-btn" type="submit" class="w-full md:w-auto bg-blue-600 text-white font-bold py-2 px-6 rounded hover:bg-blue-700 transition flex justify-center items-center">
                    <span>Search</span>
                </button>
            </form>
        </div>
    </header>

    <!-- Search Results Section -->
    <main class="max-w-6xl mx-auto py-12 px-4">
        <h3 class="text-2xl font-bold mb-6 border-b pb-2">Matching Programs</h3>
        
        <!-- Results Grid (Empty by default, populated by JavaScript) -->
        <div id="results-grid" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
            <!-- Loading State or Initial Message -->
            <p id="status-message" class="text-gray-500 col-span-full">Enter your preferences above to see matching universities.</p>
        </div>
    </main>

    <!-- JavaScript Logic -->
    <script type="module">
        // 1. Import Supabase securely from a CDN
        import { createClient } from 'https://cdn.jsdelivr.net/npm/@supabase/supabase-js/+esm';

        // 2. Initialize Supabase (REPLACE THESE WITH YOUR ACTUAL KEYS)
        const supabaseUrl = 'YOUR_SUPABASE_PROJECT_URL';
        const supabaseAnonKey = 'YOUR_SUPABASE_ANON_KEY';
        const supabase = createClient(supabaseUrl, supabaseAnonKey);

        // 3. Grab DOM Elements
        const searchForm = document.getElementById('search-form');
        const fieldSelect = document.getElementById('field-select');
        const countrySelect = document.getElementById('country-select');
        const budgetInput = document.getElementById('budget-input');
        const resultsGrid = document.getElementById('results-grid');
        const searchBtn = document.getElementById('search-btn');

        // 4. The Fetch Function (Matching Engine)
        async function findMatchingPrograms(field, country, maxBudget) {
            try {
                const { data, error } = await supabase
                    .from('programs')
                    .select(`
                        id,
                        name,
                        degree_type,
                        annual_tuition_usd,
                        universities!inner (
                            name,
                            country
                        )
                    `)
                    .eq('field_of_study', field)
                    .lte('annual_tuition_usd', maxBudget)
                    .eq('universities.country', country);

                if (error) throw error;
                return data;
            } catch (error) {
                console.error('Error fetching data:', error.message);
                return null;
            }
        }

        // 5. The Render Function (Creates HTML cards dynamically)
        function renderResults(programs) {
            resultsGrid.innerHTML = ''; // Clear previous results

            if (!programs || programs.length === 0) {
                resultsGrid.innerHTML = '<p class="text-gray-500 col-span-full">No programs found matching your criteria. Try increasing your budget or changing the country.</p>';
                return;
            }

            programs.forEach(program => {
                // Determine styling based on tuition cost
                const tuitionBadge = program.annual_tuition_usd === 0 
                    ? '<span class="bg-green-100 text-green-800 text-xs font-semibold px-2 py-1 rounded">Tuition-Free</span>'
                    : `<span class="bg-blue-100 text-blue-800 text-xs font-semibold px-2 py-1 rounded">${program.degree_type}</span>`;
                
                const formattedTuition = program.annual_tuition_usd === 0 
                    ? '$0' 
                    : `$${program.annual_tuition_usd.toLocaleString()}`;

                // Create the HTML structure for a single card
                const cardHTML = `
                    <div class="bg-white p-6 rounded-lg shadow border border-gray-200 hover:shadow-lg transition">
                        <div class="flex justify-between items-start mb-4">
                            ${tuitionBadge}
                            <span class="text-gray-500 text-sm">${program.universities.country}</span>
                        </div>
                        <h4 class="text-xl font-bold mb-1">${program.name}</h4>
                        <p class="text-gray-600 text-sm mb-4">${program.universities.name}</p>
                        <div class="border-t pt-4 flex justify-between items-center">
                            <span class="font-bold text-gray-900">${formattedTuition} / year</span>
                            <button class="text-blue-600 hover:underline text-sm font-medium">View Details &rarr;</button>
                        </div>
                    </div>
                `;
                
                // Append it to the grid
                resultsGrid.innerHTML += cardHTML;
            });
        }

        // 6. Handle the Form Submission
        searchForm.addEventListener('submit', async (e) => {
            e.preventDefault(); // Prevent page reload

            // Show loading state
            const originalBtnText = searchBtn.innerHTML;
            searchBtn.innerHTML = 'Searching...';
            resultsGrid.innerHTML = '<p class="text-gray-500 col-span-full animate-pulse">Searching database...</p>';

            // Get user inputs
            const field = fieldSelect.value;
            const country = countrySelect.value;
            const budget = parseInt(budgetInput.value, 10);

            // Fetch and render
            const results = await findMatchingPrograms(field, country, budget);
            renderResults(results);

            // Restore button state
            searchBtn.innerHTML = originalBtnText;
        });
    </script>
</body>
</html>
