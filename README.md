# House-tracker
<!DOCTYPE html>
<html lang="en" class="h-full">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>House Construction Expense & Labor Tracker</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome for Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Chart.js for Visualizations -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- Inter Font -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; }
    </style>
</head>
<body class="bg-slate-50 text-slate-800 h-full flex flex-col">

    <!-- Custom Modal for Alerts/Confirms -->
    <div id="customModal" class="fixed inset-0 bg-slate-900/50 backdrop-blur-sm z-50 hidden flex items-center justify-center p-4">
        <div class="bg-white rounded-2xl shadow-xl max-w-md w-full p-6 space-y-4 border border-slate-100 animate-in fade-in zoom-in duration-200">
            <div class="flex items-center space-x-3 text-indigo-600">
                <div id="modalIcon" class="bg-indigo-50 p-3 rounded-xl text-xl">
                    <i class="fa-solid fa-circle-info"></i>
                </div>
                <h3 id="modalTitle" class="text-lg font-bold text-slate-900">Notice</h3>
            </div>
            <p id="modalMessage" class="text-slate-600 text-sm leading-relaxed"></p>
            <div id="modalButtons" class="flex justify-end gap-3 pt-2">
                <button id="modalCancelBtn" onclick="closeModal(false)" class="px-4 py-2 rounded-xl text-sm font-semibold text-slate-600 hover:bg-slate-100 transition hidden">Cancel</button>
                <button id="modalConfirmBtn" onclick="closeModal(true)" class="px-5 py-2 rounded-xl text-sm font-semibold bg-indigo-600 text-white hover:bg-indigo-700 transition shadow-sm">OK</button>
            </div>
        </div>
    </div>

    <!-- Cloud Sync Modal -->
    <div id="cloudModal" class="fixed inset-0 bg-slate-900/50 backdrop-blur-sm z-50 hidden flex items-center justify-center p-4">
        <div class="bg-white rounded-2xl shadow-xl max-w-md w-full p-6 space-y-4 border border-slate-100">
            <div class="flex justify-between items-center">
                <div class="flex items-center space-x-3 text-indigo-600">
                    <div class="bg-indigo-50 p-3 rounded-xl text-xl">
                        <i class="fa-solid fa-cloud-arrow-up"></i>
                    </div>
                    <h3 class="text-lg font-bold text-slate-900">Cloud Sync & Backup</h3>
                </div>
                <button onclick="toggleCloudModal(false)" class="text-slate-400 hover:text-slate-600 p-1">
                    <i class="fa-solid fa-xmark text-lg"></i>
                </button>
            </div>
            <p class="text-slate-600 text-sm leading-relaxed">
                Enable secure cloud backup so your construction records sync automatically and can never be lost!
            </p>
            <div class="bg-slate-50 p-4 rounded-xl border border-slate-200 space-y-2">
                <div class="flex justify-between items-center text-xs">
                    <span class="font-semibold text-slate-500 uppercase">Status:</span>
                    <span id="cloudStatusText" class="font-bold text-amber-600">Not Connected</span>
                </div>
                <div class="flex justify-between items-center text-xs">
                    <span class="font-semibold text-slate-500 uppercase">User ID:</span>
                    <span id="cloudUserIdDisplay" class="font-mono text-slate-700 truncate max-w-[200px]">-</span>
                </div>
            </div>
            <div class="flex flex-col gap-2 pt-2">
                <button id="cloudConnectBtn" onclick="initializeCloudSync()" class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-semibold py-3 rounded-xl transition shadow-sm flex items-center justify-center gap-2 text-sm">
                    <i class="fa-solid fa-cloud"></i> Connect & Sync to Cloud
                </button>
                <button onclick="toggleCloudModal(false)" class="w-full bg-slate-100 hover:bg-slate-200 text-slate-700 font-semibold py-2.5 rounded-xl transition text-sm">
                    Close
                </button>
            </div>
        </div>
    </div>

    <header class="bg-indigo-700 text-white shadow-md">
        <div class="max-w-7xl mx-auto px-4 py-4 sm:px-6 lg:px-8 flex flex-col sm:flex-row justify-between items-center gap-4">
            <div class="flex items-center space-x-3">
                <div class="bg-indigo-600 p-2.5 rounded-xl shadow-inner border border-indigo-500">
                    <i class="fa-solid fa-hard-hat text-2xl text-amber-300"></i>
                </div>
                <div>
                    <h1 class="text-xl sm:text-2xl font-bold tracking-tight">BuildTracker Pro</h1>
                    <p class="text-xs text-indigo-200">From Foundation to Full Furnishing • Labor, Contractors & Materials</p>
                </div>
            </div>
            <div class="flex flex-wrap gap-2">
                <button onclick="toggleCloudModal(true)" class="bg-indigo-600 hover:bg-indigo-500 text-white text-xs sm:text-sm font-medium px-3.5 py-2 rounded-lg transition shadow-sm flex items-center gap-2 border border-indigo-500">
                    <i class="fa-solid fa-cloud-arrow-up"></i> <span id="cloudButtonLabel">Cloud Sync</span>
                </button>
                <button onclick="exportData()" class="bg-indigo-600 hover:bg-indigo-500 text-white text-xs sm:text-sm font-medium px-3.5 py-2 rounded-lg transition shadow-sm flex items-center gap-2">
                    <i class="fa-solid fa-download"></i> Export Data
                </button>
                <label class="bg-indigo-600 hover:bg-indigo-500 text-white text-xs sm:text-sm font-medium px-3.5 py-2 rounded-lg transition shadow-sm flex items-center gap-2 cursor-pointer">
                    <i class="fa-solid fa-upload"></i> Import Data
                    <input type="file" id="importFile" class="hidden" accept=".json" onchange="importData(event)">
                </label>
                <button onclick="resetAllData()" class="bg-rose-600 hover:bg-rose-500 text-white text-xs sm:text-sm font-medium px-3.5 py-2 rounded-lg transition shadow-sm flex items-center gap-2">
                    <i class="fa-solid fa-trash-can"></i> Reset
                </button>
            </div>
        </div>
    </header>

    <main class="flex-1 max-w-7xl w-full mx-auto px-4 py-6 sm:px-6 lg:px-8 space-y-6 overflow-y-auto">
        <!-- Summary Cards -->
        <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4">
            <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-200 flex items-center justify-between">
                <div>
                    <p class="text-xs font-semibold uppercase tracking-wider text-slate-500">Total Construction Cost</p>
                    <h3 id="statTotalCost" class="text-2xl sm:text-3xl font-bold text-slate-900 mt-1">₹0</h3>
                </div>
                <div class="p-3 bg-emerald-50 text-emerald-600 rounded-xl">
                    <i class="fa-solid fa-wallet text-xl"></i>
                </div>
            </div>
            <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-200 flex items-center justify-between">
                <div>
                    <p class="text-xs font-semibold uppercase tracking-wider text-slate-500">Total Labor Expense</p>
                    <h3 id="statLaborCost" class="text-2xl sm:text-3xl font-bold text-blue-600 mt-1">₹0</h3>
                </div>
                <div class="p-3 bg-blue-50 text-blue-600 rounded-xl">
                    <i class="fa-solid fa-users text-xl"></i>
                </div>
            </div>
            <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-200 flex items-center justify-between">
                <div>
                    <p class="text-xs font-semibold uppercase tracking-wider text-slate-500">Total Contractors Expense</p>
                    <h3 id="statContractorCost" class="text-2xl sm:text-3xl font-bold text-purple-600 mt-1">₹0</h3>
                </div>
                <div class="p-3 bg-purple-50 text-purple-600 rounded-xl">
                    <i class="fa-solid fa-file-contract text-xl"></i>
                </div>
            </div>
            <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-200 flex items-center justify-between">
                <div>
                    <p class="text-xs font-semibold uppercase tracking-wider text-slate-500">Total Material Expense</p>
                    <h3 id="statMaterialCost" class="text-2xl sm:text-3xl font-bold text-amber-600 mt-1">₹0</h3>
                </div>
                <div class="p-3 bg-amber-50 text-amber-600 rounded-xl">
                    <i class="fa-solid fa-boxes-stacked text-xl"></i>
                </div>
            </div>
        </div>

        <div class="flex border-b border-slate-200 overflow-x-auto no-scrollbar">
            <button onclick="switchTab('labor')" id="tabBtn-labor" class="tab-btn px-6 py-3 font-semibold text-sm border-b-2 border-indigo-600 text-indigo-600 whitespace-nowrap flex items-center gap-2">
                <i class="fa-solid fa-helmet-safety"></i> Daily Labor & Wages
            </button>
            <button onclick="switchTab('contractors')" id="tabBtn-contractors" class="tab-btn px-6 py-3 font-semibold text-sm border-b-2 border-transparent text-slate-500 hover:text-slate-700 whitespace-nowrap flex items-center gap-2">
                <i class="fa-solid fa-user-tie"></i> Contractors & Subcontractors
            </button>
            <button onclick="switchTab('materials')" id="tabBtn-materials" class="tab-btn px-6 py-3 font-semibold text-sm border-b-2 border-transparent text-slate-500 hover:text-slate-700 whitespace-nowrap flex items-center gap-2">
                <i class="fa-solid fa-cubes"></i> Material Purchases
            </button>
            <button onclick="switchTab('analytics')" id="tabBtn-analytics" class="tab-btn px-6 py-3 font-semibold text-sm border-b-2 border-transparent text-slate-500 hover:text-slate-700 whitespace-nowrap flex items-center gap-2">
                <i class="fa-solid fa-chart-pie"></i> Cost Analytics & Reports
            </button>
        </div>

        <!-- TAB 1: DAILY LABOR -->
        <div id="tabContent-labor" class="tab-content space-y-6">
            <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                <!-- Add Daily Labor Log Form -->
                <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-200 lg:col-span-1 h-fit">
                    <h2 class="text-lg font-bold text-slate-900 mb-4 flex items-center gap-2">
                        <i class="fa-solid fa-user-plus text-indigo-600"></i> Log Daily Labor
                    </h2>
                    <form id="laborForm" onsubmit="handleLaborSubmit(event)" class="space-y-4">
                        <div>
                            <label class="block text-xs font-semibold uppercase text-slate-600 mb-1">Date</label>
                            <input type="date" id="laborDate" required class="w-full px-3.5 py-2.5 rounded-xl border border-slate-300 focus:outline-none focus:ring-2 focus:ring-indigo-500 text-sm">
                        </div>
                        <div>
                            <label class="block text-xs font-semibold uppercase text-slate-600 mb-1">Worker / Crew Name</label>
                            <input type="text" id="laborName" placeholder="e.g. Ramesh Crew" required class="w-full px-3.5 py-2.5 rounded-xl border border-slate-300 focus:outline-none focus:ring-2 focus:ring-indigo-500 text-sm">
                        </div>
                        <div>
                            <label class="block text-xs font-semibold uppercase text-slate-600 mb-1">Labor Role / Type</label>
                            <select id="laborRole" onchange="updateDefaultRate()" class="w-full px-3.5 py-2.5 rounded-xl border border-slate-300 focus:outline-none focus:ring-2 focus:ring-indigo-500 text-sm bg-white">
                                <option value="Helper">Helper (~₹500/day)</option>
                                <option value="Mason">Mason (~₹750/day)</option>
                                <option value="Specialist">Specialist / Seasonal (~₹800/day)</option>
                                <option value="Other">Other Custom Rate</option>
                            </select>
                        </div>
                        <div class="grid grid-cols-2 gap-3">
                            <div>
                                <label class="block text-xs font-semibold uppercase text-slate-600 mb-1">No. of Workers</label>
                                <input type="number" id="laborCount" min="1" value="1" required class="w-full px-3.5 py-2.5 rounded-xl border border-slate-300 focus:outline-none focus:ring-2 focus:ring-indigo-500 text-sm">
                            </div>
                            <div>
                                <label class="block text-xs font-semibold uppercase text-slate-600 mb-1">Daily Rate per Person (₹)</label>
                                <input type="number" id="laborRate" min="0" value="500" required class="w-full px-3.5 py-2.5 rounded-xl border border-slate-300 focus:outline-none focus:ring-2 focus:ring-indigo-500 text-sm">
                            </div>
                        </div>
                        <div>
                            <label class="block text-xs font-semibold uppercase text-slate-600 mb-1">Advance / Paid Amount (₹)</label>
                            <input type="number" id="laborPaid" min="0" value="0" required class="w-full px-3.5 py-2.5 rounded-xl border border-slate-300 focus:outline-none focus:ring-2 focus:ring-indigo-500 text-sm">
                        </div>
                        <div>
                            <label class="block text-xs font-semibold uppercase text-slate-600 mb-1">Notes / Work Done</label>
                            <input type="text" id="laborNotes" placeholder="e.g. Ground floor casting" class="w-full px-3.5 py-2.5 rounded-xl border border-slate-300 focus:outline-none focus:ring-2 focus:ring-indigo-500 text-sm">
                        </div>
                        <button type="submit" class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-semibold py-3 rounded-xl transition shadow-md shadow-indigo-100 flex items-center justify-center gap-2">
                            <i class="fa-solid fa-plus"></i> Save Labor Record
                        </button>
                    </form>
                </div>

                <!-- Labor Records Table -->
                <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-200 lg:col-span-2 flex flex-col">
                    <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center gap-3 mb-4">
                        <h2 class="text-lg font-bold text-slate-900">Labor Attendance & Wages Log</h2>
                        <div class="flex items-center gap-2 w-full sm:w-auto">
                            <input type="text" id="laborSearch" oninput="renderLaborTable()" placeholder="Search labor..." class="px-3 py-2 border border-slate-300 rounded-xl text-sm focus:outline-none focus:ring-2 focus:ring-indigo-500 w-full sm:w-48">
                        </div>
                    </div>
                    <div class="overflow-x-auto flex-1 border border-slate-100 rounded-xl">
                        <table class="w-full text-left border-collapse">
                            <thead>
                                <tr class="bg-slate-100 text-slate-600 text-xs uppercase font-semibold">
                                    <th class="p-3">Date</th>
                                    <th class="p-3">Worker / Details</th>
                                    <th class="p-3">Type</th>
                                    <th class="p-3">Count × Rate</th>
                                    <th class="p-3">Total Due</th>
                                    <th class="p-3">Paid</th>
                                    <th class="p-3 text-center">Action</th>
                                </tr>
                            </thead>
                            <tbody id="laborTableBody" class="divide-y divide-slate-100 text-sm">
                                <!-- Populated dynamically -->
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </div>

        <!-- TAB 2: CONTRACTORS -->
        <div id="tabContent-contractors" class="tab-content space-y-6 hidden">
            <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                <!-- Add Contractor Expense / Payment Form -->
                <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-200 lg:col-span-1 h-fit">
                    <h2 class="text-lg font-bold text-slate-900 mb-4 flex items-center gap-2">
                        <i class="fa-solid fa-user-tie text-purple-600"></i> Contractor Payment / Bill
                    </h2>
                    <form id="contractorForm" onsubmit="handleContractorSubmit(event)" class="space-y-4">
                        <div>
                            <label class="block text-xs font-semibold uppercase text-slate-600 mb-1">Date</label>
                            <input type="date" id="conDate" required class="w-full px-3.5 py-2.5 rounded-xl border border-slate-300 focus:outline-none focus:ring-2 focus:ring-purple-500 text-sm">
                        </div>
                        <div>
                            <label class="block text-xs font-semibold uppercase text-slate-600 mb-1">Contractor Category</label>
                            <select id="conCategory" class="w-full px-3.5 py-2.5 rounded-xl border border-slate-300 focus:outline-none focus:ring-2 focus:ring-purple-500 text-sm bg-white">
                                <option value="Iron (TMT/Fabrication) Contractor">Iron / Bar Bending Contractor</option>
                                <option value="Shuttering (Centering) Contractor">Shuttering / Centering Contractor</option>
                                <option value="Masonry / Structure Contractor">Masonry / Main Structure Contractor</option>
                                <option value="Plumbing & Sanitary Contractor">Plumbing & Sanitary Contractor</option>
                                <option value="Electrical & Wiring Contractor">Electrical & Wiring Contractor</option>
                                <option value="Flooring & Tiling Contractor">Flooring & Tiling Contractor</option>
                                <option value="Carpentry & Woodwork Contractor">Carpentry & Woodwork Contractor</option>
                                <option value="Painting & Wall Finish Contractor">Painting & Wall Finish Contractor</option>
                                <option value="False Ceiling & Interior Contractor">False Ceiling & Interior Contractor</option>
                                <option value="General Furnishing & Fittings">General Furnishing & Fittings Contractor</option>
                                <option value="Other Contractor">Other Specialized Contractor</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-xs font-semibold uppercase text-slate-600 mb-1">Contractor / Agency Name</label>
                            <input type="text" id="conName" placeholder="e.g. Verma Iron Works / Sharma Centering" required class="w-full px-3.5 py-2.5 rounded-xl border border-slate-300 focus:outline-none focus:ring-2 focus:ring-purple-500 text-sm">
                        </div>
                        <div class="grid grid-cols-2 gap-3">
                            <div>
                                <label class="block text-xs font-semibold uppercase text-slate-600 mb-1">Total Agreed Bill (₹)</label>
                                <input type="number" id="conTotalAmount" min="0" value="0" required class="w-full px-3.5 py-2.5 rounded-xl border border-slate-300 focus:outline-none focus:ring-2 focus:ring-purple-500 text-sm">
                            </div>
                            <div>
                                <label class="block text-xs font-semibold uppercase text-slate-600 mb-1">Amount Paid Now (₹)</label>
                                <input type="number" id="conPaidAmount" min="0" value="0" required class="w-full px-3.5 py-2.5 rounded-xl border border-slate-300 focus:outline-none focus:ring-2 focus:ring-purple-500 text-sm">
                            </div>
                        </div>
                        <div>
                            <label class="block text-xs font-semibold uppercase text-slate-600 mb-1">Milestone / Scope Notes</label>
                            <input type="text" id="conNotes" placeholder="e.g. Advance for roof slab shuttering & pillars" class="w-full px-3.5 py-2.5 rounded-xl border border-slate-300 focus:outline-none focus:ring-2 focus:ring-purple-500 text-sm">
                        </div>
                        <button type="submit" class="w-full bg-purple-600 hover:bg-purple-700 text-white font-semibold py-3 rounded-xl transition shadow-md shadow-purple-100 flex items-center justify-center gap-2">
                            <i class="fa-solid fa-plus"></i> Save Contractor Record
                        </button>
                    </form>
                </div>

                <!-- Contractors Table -->
                <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-200 lg:col-span-2 flex flex-col">
                    <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center gap-3 mb-4">
                        <h2 class="text-lg font-bold text-slate-900">Contractors & Milestone Payments Log</h2>
                        <div class="flex items-center gap-2 w-full sm:w-auto">
                            <input type="text" id="contractorSearch" oninput="renderContractorTable()" placeholder="Search contractors..." class="px-3 py-2 border border-slate-300 rounded-xl text-sm focus:outline-none focus:ring-2 focus:ring-purple-500 w-full sm:w-48">
                        </div>
                    </div>
                    <div class="overflow-x-auto flex-1 border border-slate-100 rounded-xl">
                        <table class="w-full text-left border-collapse">
                            <thead>
                                <tr class="bg-slate-100 text-slate-600 text-xs uppercase font-semibold">
                                    <th class="p-3">Date</th>
                                    <th class="p-3">Contractor / Name</th>
                                    <th class="p-3">Category</th>
                                    <th class="p-3">Agreed Total</th>
                                    <th class="p-3">Paid</th>
                                    <th class="p-3">Balance Due</th>
                                    <th class="p-3 text-center">Action</th>
                                </tr>
                            </thead>
                            <tbody id="contractorTableBody" class="divide-y divide-slate-100 text-sm">
                                <!-- Populated dynamically -->
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </div>

        <!-- TAB 3: MATERIALS -->
        <div id="tabContent-materials" class="tab-content space-y-6 hidden">
            <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                <!-- Add Material Purchase Form -->
                <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-200 lg:col-span-1 h-fit">
                    <h2 class="text-lg font-bold text-slate-900 mb-4 flex items-center gap-2">
                        <i class="fa-solid fa-cart-plus text-amber-600"></i> Add Material Purchase
                    </h2>
                    <form id="materialForm" onsubmit="handleMaterialSubmit(event)" class="space-y-4">
                        <div>
                            <label class="block text-xs font-semibold uppercase text-slate-600 mb-1">Date</label>
                            <input type="date" id="matDate" required class="w-full px-3.5 py-2.5 rounded-xl border border-slate-300 focus:outline-none focus:ring-2 focus:ring-indigo-500 text-sm">
                        </div>
                        <div>
                            <label class="block text-xs font-semibold uppercase text-slate-600 mb-1">Material Category</label>
                            <select id="matCategory" class="w-full px-3.5 py-2.5 rounded-xl border border-slate-300 focus:outline-none focus:ring-2 focus:ring-indigo-500 text-sm bg-white">
                                <option value="Cement">Cement</option>
                                <option value="Steel (TMT)">Steel (TMT Bars)</option>
                                <option value="Bricks / Blocks">Bricks / Blocks</option>
                                <option value="Sand (Fine/Coarse)">Sand</option>
                                <option value="Aggregates (Gravel)">Aggregates (Gravel)</option>
                                <option value="Plumbing & Electrical">Plumbing & Electrical Supplies</option>
                                <option value="Tiles & Flooring">Tiles & Flooring</option>
                                <option value="Wood, Doors & Windows">Wood, Doors & Windows</option>
                                <option value="Paints & Putty">Paints & Wall Putty</option>
                                <option value="Furnishing & Fixtures">Furnishing & Fixtures</option>
                                <option value="Other">Other Materials</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-xs font-semibold uppercase text-slate-600 mb-1">Item Description / Brand</label>
                            <input type="text" id="matName" placeholder="e.g. UltraTech Cement / Kajaria Tiles" required class="w-full px-3.5 py-2.5 rounded-xl border border-slate-300 focus:outline-none focus:ring-2 focus:ring-indigo-500 text-sm">
                        </div>
                        <div class="grid grid-cols-2 gap-3">
                            <div>
                                <label class="block text-xs font-semibold uppercase text-slate-600 mb-1">Quantity & Unit</label>
                                <input type="text" id="matQty" placeholder="e.g. 50 Bags / 500 sqft" required class="w-full px-3.5 py-2.5 rounded-xl border border-slate-300 focus:outline-none focus:ring-2 focus:ring-indigo-500 text-sm">
                            </div>
                            <div>
                                <label class="block text-xs font-semibold uppercase text-slate-600 mb-1">Total Cost (₹)</label>
                                <input type="number" id="matCost" min="0" required class="w-full px-3.5 py-2.5 rounded-xl border border-slate-300 focus:outline-none focus:ring-2 focus:ring-indigo-500 text-sm">
                            </div>
                        </div>
                        <div>
                            <label class="block text-xs font-semibold uppercase text-slate-600 mb-1">Supplier / Store Name</label>
                            <input type="text" id="matSupplier" placeholder="e.g. Sharma Hardware Store" class="w-full px-3.5 py-2.5 rounded-xl border border-slate-300 focus:outline-none focus:ring-2 focus:ring-indigo-500 text-sm">
                        </div>
                        <button type="submit" class="w-full bg-amber-600 hover:bg-amber-700 text-white font-semibold py-3 rounded-xl transition shadow-md shadow-amber-100 flex items-center justify-center gap-2">
                            <i class="fa-solid fa-plus"></i> Save Material Record
                        </button>
                    </form>
                </div>

                <!-- Material Purchases Table -->
                <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-200 lg:col-span-2 flex flex-col">
                    <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center gap-3 mb-4">
                        <h2 class="text-lg font-bold text-slate-900">Material Purchase Log</h2>
                        <div class="flex items-center gap-2 w-full sm:w-auto">
                            <input type="text" id="materialSearch" oninput="renderMaterialTable()" placeholder="Search materials..." class="px-3 py-2 border border-slate-300 rounded-xl text-sm focus:outline-none focus:ring-2 focus:ring-indigo-500 w-full sm:w-48">
                        </div>
                    </div>
                    <div class="overflow-x-auto flex-1 border border-slate-100 rounded-xl">
                        <table class="w-full text-left border-collapse">
                            <thead>
                                <tr class="bg-slate-100 text-slate-600 text-xs uppercase font-semibold">
                                    <th class="p-3">Date</th>
                                    <th class="p-3">Category</th>
                                    <th class="p-3">Description</th>
                                    <th class="p-3">Quantity</th>
                                    <th class="p-3">Cost</th>
                                    <th class="p-3">Supplier</th>
                                    <th class="p-3 text-center">Action</th>
                                </tr>
                            </thead>
                            <tbody id="materialTableBody" class="divide-y divide-slate-100 text-sm">
                                <!-- Populated dynamically -->
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </div>

        <!-- TAB 4: ANALYTICS -->
        <div id="tabContent-analytics" class="tab-content space-y-6 hidden">
            <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
                <!-- Chart: Expenses Breakdown -->
                <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-200">
                    <h3 class="text-base font-bold text-slate-900 mb-4 flex items-center gap-2">
                        <i class="fa-solid fa-chart-pie text-indigo-600"></i> Overall Construction Expense Breakdown
                    </h3>
                    <div class="relative h-72 flex justify-center items-center">
                        <canvas id="expensePieChart"></canvas>
                    </div>
                </div>
                <!-- Summary Card & Advice -->
                <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-200 flex flex-col justify-between">
                    <div>
                        <h3 class="text-base font-bold text-slate-900 mb-4 flex items-center gap-2">
                            <i class="fa-solid fa-lightbulb text-amber-500"></i> Construction Insights & Furnishing Stages
                        </h3>
                        <ul id="insightsList" class="space-y-3 text-sm text-slate-600">
                            <!-- Populated dynamically -->
                        </ul>
                    </div>
                    <div class="mt-6 pt-6 border-t border-slate-100 bg-indigo-50/50 p-4 rounded-xl">
                        <h4 class="font-semibold text-indigo-900 text-sm mb-1">Standard Daily Rates Reference</h4>
                        <p class="text-xs text-indigo-700 leading-relaxed">
                            Helpers average <strong>₹500/day</strong>, Masons average <strong>₹750/day</strong>, and specialized seasonal experts/contractors range around <strong>₹800+/day</strong>. Track iron & shuttering contractors under the Contractors tab!
                        </p>
                    </div>
                </div>
            </div>
        </div>
    </main>

    <!-- Firebase SDKs -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, setDoc, getDoc, onSnapshot } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Global Firebase handles exposed to window
        window.FB_SDK = {
            initializeApp, getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged,
            getFirestore, doc, setDoc, getDoc, onSnapshot
        };
        window.dispatchEvent(new Event('firebase-loaded'));
    </script>

    <script>
        // State management
        let state = {
            labor: [],
            contractors: [],
            materials: []
        };

        let expensePieChartInstance = null;
        let isCloudConnected = false;
        let unsubscribeCloud = null;
        let db = null, auth = null, appId = 'default-app-id', userId = null;

        // Custom Modal handler
        let modalResolveCallback = null;
        function showModal(title, message, isConfirm = false, iconClass = 'fa-solid fa-circle-info text-indigo-600') {
            document.getElementById('modalTitle').innerText = title;
            document.getElementById('modalMessage').innerText = message;
            document.getElementById('modalIcon').innerHTML = `<i class="${iconClass}"></i>`;
            
            const cancelBtn = document.getElementById('modalCancelBtn');
            const confirmBtn = document.getElementById('modalConfirmBtn');
            
            if (isConfirm) {
                cancelBtn.classList.remove('hidden');
                confirmBtn.innerText = 'Confirm';
            } else {
                cancelBtn.classList.add('hidden');
                confirmBtn.innerText = 'OK';
            }
            
            document.getElementById('customModal').classList.remove('hidden');
            return new Promise((resolve) => {
                modalResolveCallback = resolve;
            });
        }

        function closeModal(result) {
            document.getElementById('customModal').classList.add('hidden');
            if (modalResolveCallback) {
                modalResolveCallback(result);
                modalResolveCallback = null;
            }
        }

        function toggleCloudModal(show) {
            const modal = document.getElementById('cloudModal');
            if (show) {
                modal.classList.remove('hidden');
                if (userId) {
                    document.getElementById('cloudUserIdDisplay').innerText = userId;
                    document.getElementById('cloudStatusText').innerText = isCloudConnected ? 'Connected & Synced' : 'Connecting...';
                    document.getElementById('cloudStatusText').className = isCloudConnected ? 'font-bold text-emerald-600' : 'font-bold text-amber-600';
                }
            } else {
                modal.classList.add('hidden');
            }
        }

        // Initialize App
        window.addEventListener('DOMContentLoaded', () => {
            const today = new Date().toISOString().split('T')[0];
            document.getElementById('laborDate').value = today;
            document.getElementById('conDate').value = today;
            document.getElementById('matDate').value = today;

            updateDefaultRate();
            loadLocalData();
            renderAll();
        });

        function loadLocalData() {
            const savedLabor = localStorage.getItem('build_labor');
            const savedContractors = localStorage.getItem('build_contractors');
            const savedMaterials = localStorage.getItem('build_materials');
            if (savedLabor) state.labor = JSON.parse(savedLabor);
            if (savedContractors) state.contractors = JSON.parse(savedContractors);
            if (savedMaterials) state.materials = JSON.parse(savedMaterials);
        }

        function saveData() {
            localStorage.setItem('build_labor', JSON.stringify(state.labor));
            localStorage.setItem('build_contractors', JSON.stringify(state.contractors));
            localStorage.setItem('build_materials', JSON.stringify(state.materials));
            renderAll();

            if (isCloudConnected && db && userId) {
                syncToCloud();
            }
        }

        // Firebase Cloud Sync Setup
        window.addEventListener('firebase-loaded', async () => {
            try {
                appId = typeof __app_id !== 'undefined' ? __app_id : 'house-construction-tracker';
                const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : null;
                if (!firebaseConfig) return;

                const app = window.FB_SDK.initializeApp(firebaseConfig);
                db = window.FB_SDK.getFirestore(app);
                auth = window.FB_SDK.getAuth(app);

                const token = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : undefined;
                if (token) {
                    await window.FB_SDK.signInWithCustomToken(auth, token);
                } else {
                    await window.FB_SDK.signInAnonymously(auth);
                }

                userId = auth.currentUser?.uid || crypto.randomUUID();
                document.getElementById('cloudUserIdDisplay').innerText = userId;

                startCloudSyncListener();
            } catch (err) {
                console.error("Cloud init error:", err);
            }
        });

        async function initializeCloudSync() {
            if (!db || !auth) {
                await showModal('Cloud Sync', 'Cloud environment is initializing or unavailable in this view. Your data is safely stored locally in your browser!', false);
                return;
            }
            try {
                isCloudConnected = true;
                document.getElementById('cloudStatusText').innerText = 'Connected & Synced';
                document.getElementById('cloudStatusText').className = 'font-bold text-emerald-600';
                document.getElementById('cloudButtonLabel').innerText = 'Cloud Synced';
                document.getElementById('cloudConnectBtn').innerText = 'Connected to Cloud';
                document.getElementById('cloudConnectBtn').disabled = true;
                document.getElementById('cloudConnectBtn').classList.add('bg-emerald-600', 'hover:bg-emerald-600');

                syncToCloud();
                await showModal('Success!', 'Cloud sync successfully activated. Your data is now safely backed up.', false, 'fa-solid fa-cloud-arrow-up text-emerald-600');
            } catch (err) {
                await showModal('Sync Error', 'Failed to connect to cloud storage.', false, 'fa-solid fa-triangle-exclamation text-rose-600');
            }
        }

        function startCloudSyncListener() {
            if (!db || !userId) return;
            const docRef = window.FB_SDK.doc(db, 'artifacts', appId, 'users', userId, 'construction_data', 'main_record');
            
            if (unsubscribeCloud) unsubscribeCloud();

            unsubscribeCloud = window.FB_SDK.onSnapshot(docRef, (docSnap) => {
                if (docSnap.exists()) {
                    const cloudData = docSnap.data();
                    if (cloudData && (cloudData.labor || cloudData.contractors || cloudData.materials)) {
                        state.labor = cloudData.labor || [];
                        state.contractors = cloudData.contractors || [];
                        state.materials = cloudData.materials || [];
                        localStorage.setItem('build_labor', JSON.stringify(state.labor));
                        localStorage.setItem('build_contractors', JSON.stringify(state.contractors));
                        localStorage.setItem('build_materials', JSON.stringify(state.materials));
                        renderAll();
                        isCloudConnected = true;
                        document.getElementById('cloudButtonLabel').innerText = 'Cloud Synced';
                    }
                }
            }, (error) => {
                console.error("Firestore snapshot error:", error);
            });
        }

        async function syncToCloud() {
            if (!db || !userId || !isCloudConnected) return;
            try {
                const docRef = window.FB_SDK.doc(db, 'artifacts', appId, 'users', userId, 'construction_data', 'main_record');
                await window.FB_SDK.setDoc(docRef, {
                    labor: state.labor,
                    contractors: state.contractors,
                    materials: state.materials,
                    updatedAt: new Date().toISOString()
                });
            } catch (err) {
                console.error("Error saving to cloud:", err);
            }
        }

        // Tab switching
        function switchTab(tabName) {
            document.querySelectorAll('.tab-content').forEach(el => el.classList.add('hidden'));
            document.querySelectorAll('.tab-btn').forEach(btn => {
                btn.classList.remove('border-indigo-600', 'text-indigo-600');
                btn.classList.add('border-transparent', 'text-slate-500');
            });

            document.getElementById(`tabContent-${tabName}`).classList.remove('hidden');
            const activeBtn = document.getElementById(`tabBtn-${tabName}`);
            activeBtn.classList.add('border-indigo-600', 'text-indigo-600');
            activeBtn.classList.remove('border-transparent', 'text-slate-500');

            if (tabName === 'analytics') {
                updateCharts();
            }
        }

        function updateDefaultRate() {
            const role = document.getElementById('laborRole').value;
            const rateInput = document.getElementById('laborRate');
            if (role === 'Helper') rateInput.value = 500;
            else if (role === 'Mason') rateInput.value = 750;
            else if (role === 'Specialist') rateInput.value = 800;
        }

        // Labor Form Submit
        function handleLaborSubmit(e) {
            e.preventDefault();
            const newItem = {
                id: Date.now().toString(),
                date: document.getElementById('laborDate').value,
                name: document.getElementById('laborName').value,
                role: document.getElementById('laborRole').value,
                count: parseInt(document.getElementById('laborCount').value) || 1,
                rate: parseFloat(document.getElementById('laborRate').value) || 0,
                paid: parseFloat(document.getElementById('laborPaid').value) || 0,
                notes: document.getElementById('laborNotes').value
            };
            state.labor.unshift(newItem);
            document.getElementById('laborForm').reset();
            document.getElementById('laborDate').value = new Date().toISOString().split('T')[0];
            updateDefaultRate();
            saveData();
        }

        // Contractor Form Submit
        function handleContractorSubmit(e) {
            e.preventDefault();
            const newItem = {
                id: Date.now().toString(),
                date: document.getElementById('conDate').value,
                category: document.getElementById('conCategory').value,
                name: document.getElementById('conName').value,
                totalAmount: parseFloat(document.getElementById('conTotalAmount').value) || 0,
                paidAmount: parseFloat(document.getElementById('conPaidAmount').value) || 0,
                notes: document.getElementById('conNotes').value
            };
            state.contractors.unshift(newItem);
            document.getElementById('contractorForm').reset();
            document.getElementById('conDate').value = new Date().toISOString().split('T')[0];
            saveData();
        }

        // Material Form Submit
        function handleMaterialSubmit(e) {
            e.preventDefault();
            const newItem = {
                id: Date.now().toString(),
                date: document.getElementById('matDate').value,
                category: document.getElementById('matCategory').value,
                name: document.getElementById('matName').value,
                qty: document.getElementById('matQty').value,
                cost: parseFloat(document.getElementById('matCost').value) || 0,
                supplier: document.getElementById('matSupplier').value
            };
            state.materials.unshift(newItem);
            document.getElementById('materialForm').reset();
            document.getElementById('matDate').value = new Date().toISOString().split('T')[0];
            saveData();
        }

        function deleteLabor(id) {
            state.labor = state.labor.filter(item => item.id !== id);
            saveData();
        }

        function deleteContractor(id) {
            state.contractors = state.contractors.filter(item => item.id !== id);
            saveData();
        }

        function deleteMaterial(id) {
            state.materials = state.materials.filter(item => item.id !== id);
            saveData();
        }

        // Render everything
        function renderAll() {
            renderLaborTable();
            renderContractorTable();
            renderMaterialTable();
            updateStats();
            if (!document.getElementById('tabContent-analytics').classList.contains('hidden')) {
                updateCharts();
            }
        }

        function renderLaborTable() {
            const query = (document.getElementById('laborSearch').value || '').toLowerCase();
            const tbody = document.getElementById('laborTableBody');
            tbody.innerHTML = '';

            const filtered = state.labor.filter(item => 
                item.name.toLowerCase().includes(query) || 
                item.role.toLowerCase().includes(query) || 
                (item.notes && item.notes.toLowerCase().includes(query)) ||
                item.date.includes(query)
            );

            if (filtered.length === 0) {
                tbody.innerHTML = `<tr><td colspan="7" class="p-6 text-center text-slate-400">No labor records found. Start adding records using the form!</td></tr>`;
                return;
            }

            filtered.forEach(item => {
                const totalDue = item.count * item.rate;
                const balance = totalDue - item.paid;
                const tr = document.createElement('tr');
                tr.className = 'hover:bg-slate-50 transition border-b border-slate-100';
                tr.innerHTML = `
                    <td class="p-3 text-slate-600 whitespace-nowrap">${item.date}</td>
                    <td class="p-3 font-medium text-slate-900">
                        ${item.name}
                        ${item.notes ? `<div class="text-xs text-slate-400 font-normal">${item.notes}</div>` : ''}
                    </td>
                    <td class="p-3">
                        <span class="px-2.5 py-1 text-xs font-semibold rounded-full 
                            ${item.role === 'Mason' ? 'bg-amber-100 text-amber-700' : 
                              item.role === 'Specialist' ? 'bg-purple-100 text-purple-700' : 'bg-blue-100 text-blue-700'}">
                            ${item.role}
                        </span>
                    </td>
                    <td class="p-3 text-slate-600">${item.count} × ₹${item.rate.toLocaleString('en-IN')}</td>
                    <td class="p-3 font-semibold text-slate-900">₹${totalDue.toLocaleString('en-IN')}</td>
                    <td class="p-3 text-indigo-600 font-medium">₹${item.paid.toLocaleString('en-IN')} ${balance > 0 ? `<span class="text-xs text-rose-500 block">(Due: ₹${balance.toLocaleString('en-IN')})</span>` : ''}</td>
                    <td class="p-3 text-center">
                        <button onclick="deleteLabor('${item.id}')" class="text-slate-400 hover:text-rose-600 transition p-1">
                            <i class="fa-solid fa-trash-can"></i>
                        </button>
                    </td>
                `;
                tbody.appendChild(tr);
            });
        }

        function renderContractorTable() {
            const query = (document.getElementById('contractorSearch').value || '').toLowerCase();
            const tbody = document.getElementById('contractorTableBody');
            tbody.innerHTML = '';

            const filtered = state.contractors.filter(item => 
                item.name.toLowerCase().includes(query) || 
                item.category.toLowerCase().includes(query) || 
                (item.notes && item.notes.toLowerCase().includes(query)) ||
                item.date.includes(query)
            );

            if (filtered.length === 0) {
                tbody.innerHTML = `<tr><td colspan="7" class="p-6 text-center text-slate-400">No contractor records found. Log iron, shuttering, or furnishing contractors here!</td></tr>`;
                return;
            }

            filtered.forEach(item => {
                const balanceDue = item.totalAmount - item.paidAmount;
                const tr = document.createElement('tr');
                tr.className = 'hover:bg-slate-50 transition border-b border-slate-100';
                tr.innerHTML = `
                    <td class="p-3 text-slate-600 whitespace-nowrap">${item.date}</td>
                    <td class="p-3 font-medium text-slate-900">
                        ${item.name}
                        ${item.notes ? `<div class="text-xs text-slate-400 font-normal">${item.notes}</div>` : ''}
                    </td>
                    <td class="p-3">
                        <span class="px-2.5 py-1 text-xs font-semibold rounded-full bg-purple-100 text-purple-800">
                            ${item.category}
                        </span>
                    </td>
                    <td class="p-3 font-semibold text-slate-900">₹${item.totalAmount.toLocaleString('en-IN')}</td>
                    <td class="p-3 text-indigo-600 font-medium">₹${item.paidAmount.toLocaleString('en-IN')}</td>
                    <td class="p-3 font-semibold ${balanceDue > 0 ? 'text-rose-600' : 'text-emerald-600'}">₹${balanceDue.toLocaleString('en-IN')}</td>
                    <td class="p-3 text-center">
                        <button onclick="deleteContractor('${item.id}')" class="text-slate-400 hover:text-rose-600 transition p-1">
                            <i class="fa-solid fa-trash-can"></i>
                        </button>
                    </td>
                `;
                tbody.appendChild(tr);
            });
        }

        function renderMaterialTable() {
            const query = (document.getElementById('materialSearch').value || '').toLowerCase();
            const tbody = document.getElementById('materialTableBody');
            tbody.innerHTML = '';

            const filtered = state.materials.filter(item => 
                item.name.toLowerCase().includes(query) || 
                item.category.toLowerCase().includes(query) || 
                (item.supplier && item.supplier.toLowerCase().includes(query)) ||
                item.date.includes(query)
            );

            if (filtered.length === 0) {
                tbody.innerHTML = `<tr><td colspan="7" class="p-6 text-center text-slate-400">No material records found. Start adding purchases using the form!</td></tr>`;
                return;
            }

            filtered.forEach(item => {
                const tr = document.createElement('tr');
                tr.className = 'hover:bg-slate-50 transition border-b border-slate-100';
                tr.innerHTML = `
                    <td class="p-3 text-slate-600 whitespace-nowrap">${item.date}</td>
                    <td class="p-3">
                        <span class="px-2.5 py-1 text-xs font-semibold rounded-full bg-amber-100 text-amber-800">
                            ${item.category}
                        </span>
                    </td>
                    <td class="p-3 font-medium text-slate-900">${item.name}</td>
                    <td class="p-3 text-slate-600">${item.qty}</td>
                    <td class="p-3 font-semibold text-slate-900">₹${item.cost.toLocaleString('en-IN')}</td>
                    <td class="p-3 text-slate-600 text-xs">${item.supplier || '-'}</td>
                    <td class="p-3 text-center">
                        <button onclick="deleteMaterial('${item.id}')" class="text-slate-400 hover:text-rose-600 transition p-1">
                            <i class="fa-solid fa-trash-can"></i>
                        </button>
                    </td>
                `;
                tbody.appendChild(tr);
            });
        }

        function updateStats() {
            const totalLaborCost = state.labor.reduce((sum, item) => sum + (item.count * item.rate), 0);
            const totalContractorCost = state.contractors.reduce((sum, item) => sum + item.totalAmount, 0);
            const totalMaterialCost = state.materials.reduce((sum, item) => sum + item.cost, 0);
            const grandTotal = totalLaborCost + totalContractorCost + totalMaterialCost;

            document.getElementById('statTotalCost').innerText = `₹${grandTotal.toLocaleString('en-IN')}`;
            document.getElementById('statLaborCost').innerText = `₹${totalLaborCost.toLocaleString('en-IN')}`;
            document.getElementById('statContractorCost').innerText = `₹${totalContractorCost.toLocaleString('en-IN')}`;
            document.getElementById('statMaterialCost').innerText = `₹${totalMaterialCost.toLocaleString('en-IN')}`;
        }

        function updateCharts() {
            const totalLaborCost = state.labor.reduce((sum, item) => sum + (item.count * item.rate), 0);
            const totalContractorCost = state.contractors.reduce((sum, item) => sum + item.totalAmount, 0);
            
            const matCategories = {};
            state.materials.forEach(item => {
                matCategories[item.category] = (matCategories[item.category] || 0) + item.cost;
            });

            const labels = ['Labor Wages', 'Contractors', ...Object.keys(matCategories)];
            const dataValues = [totalLaborCost, totalContractorCost, ...Object.values(matCategories)];

            const ctx = document.getElementById('expensePieChart').getContext('2d');
            if (expensePieChartInstance) {
                expensePieChartInstance.destroy();
            }

            expensePieChartInstance = new Chart(ctx, {
                type: 'doughnut',
                data: {
                    labels: labels,
                    datasets: [{
                        data: dataValues,
                        backgroundColor: [
                            '#4f46e5', '#9333ea', '#f59e0b', '#10b981', '#ef4444', '#8b5cf6', '#ec4899', '#06b6d4', '#84cc16', '#3b82f6', '#f97316'
                        ],
                        borderWidth: 2,
                        borderColor: '#ffffff'
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: {
                            position: 'bottom',
                            labels: { boxWidth: 12, font: { family: 'Inter', size: 11 } }
                        }
                    }
                }
            });

            const insightsList = document.getElementById('insightsList');
            insightsList.innerHTML = '';

            const totalCost = totalLaborCost + totalContractorCost + Object.values(matCategories).reduce((a,b)=>a+b, 0);
            const laborPct = totalCost > 0 ? ((totalLaborCost / totalCost) * 100).toFixed(1) : 0;
            const contractorPct = totalCost > 0 ? ((totalContractorCost / totalCost) * 100).toFixed(1) : 0;

            const items = [
                totalCost === 0 ? `No expenses logged yet. Add your labor, contractor bills, or material purchases to view full insights.` : `Daily labor accounts for <strong>${laborPct}%</strong> and contractors account for <strong>${contractorPct}%</strong> of overall house construction expenses.`,
                `You have logged <strong>${state.labor.length}</strong> daily labor sessions, <strong>${state.contractors.length}</strong> contractor milestones (Iron, Shuttering, Furnishing, etc.), and <strong>${state.materials.length}</strong> material purchase bills.`,
                `Track your complete journey from foundation, iron & centering, brickwork, electrical/plumbing, down to full interior furnishing and painting.`
            ];

            items.forEach(text => {
                const li = document.createElement('li');
                li.className = 'flex items-start gap-2';
                li.innerHTML = `<i class="fa-solid fa-circle-check text-indigo-600 mt-1"></i> <span>${text}</span>`;
                insightsList.appendChild(li);
            });

            const safetyLi = document.createElement('li');
            safetyLi.className = 'flex items-start gap-2 pt-2 border-t border-slate-100 text-xs text-slate-500';
            safetyLi.innerHTML = `<i class="fa-solid fa-shield-halved text-emerald-600 mt-0.5"></i> <span><strong>Data Safety Guarantee:</strong> Your records are safely synced when Cloud Sync is connected. You can also export a backup JSON file anytime!</span>`;
            insightsList.appendChild(safetyLi);
        }

        // Export data as JSON
        function exportData() {
            const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(state, null, 2));
            const downloadAnchor = document.createElement('a');
            downloadAnchor.setAttribute("href", dataStr);
            downloadAnchor.setAttribute("download", `construction_full_expenses_${new Date().toISOString().split('T')[0]}.json`);
            document.body.appendChild(downloadAnchor);
            downloadAnchor.click();
            downloadAnchor.remove();
        }

        // Import data
        function importData(event) {
            const fileReader = new FileReader();
            if (event.target.files[0]) {
                fileReader.readAsText(event.target.files[0], "UTF-8");
                fileReader.onload = async (e) => {
                    try {
                        const parsed = JSON.parse(e.target.result);
                        if (parsed.labor && parsed.materials) {
                            state.labor = parsed.labor || [];
                            state.contractors = parsed.contractors || [];
                            state.materials = parsed.materials || [];
                            saveData();
                            await showModal('Success', 'Data imported successfully!', false);
                        } else {
                            await showModal('Error', 'Invalid file format.', false);
                        }
                    } catch (err) {
                        await showModal('Error', 'Error parsing JSON file.', false);
                    }
                };
            }
        }

        async function resetAllData() {
            const confirmed = await showModal('Reset All Data', 'Are you sure you want to clear all data? This cannot be undone.', true, 'fa-solid fa-triangle-exclamation text-rose-600');
            if (confirmed) {
                state = { labor: [], contractors: [], materials: [] };
                saveData();
            }
        }
    </script>
</body>
</html>
