<!DOCTYPE html>
<html lang="ms">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SK Bukit Kuchai - Dashboard Rekod Perkembangan Murid</title>

    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        softpink: {
                            50: '#fff5f7',
                            100: '#ffe4e6',
                            200: '#fecdd3',
                            300: '#fda4af',
                            400: '#fb7185',
                            500: '#f43f5e',
                        },
                        magenta: {
                            500: '#d946ef',
                            600: '#c026d3',
                            700: '#a21caf',
                            800: '#86198f',
                        }
                    }
                }
            }
        }
    </script>

    <!-- FontAwesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Google Fonts Inter -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&display=swap" rel="stylesheet">
    <!-- Chart.js -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- PapaParse CSV Parser -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.3.2/papaparse.min.js"></script>

    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #fff5f7;
        }
        @media print {
            .no-print {
                display: none !important;
            }
            .print-only {
                display: block !important;
            }
            body {
                background-color: #ffffff !important;
                color: #000000 !important;
            }
            .page-break {
                page-break-before: always;
            }
            .print-card {
                border: 1px solid #d1d5db !important;
                box-shadow: none !important;
            }
        }
        .print-only {
            display: none;
        }
        /* Custom scrollbars */
        ::-webkit-scrollbar {
            width: 8px;
            height: 8px;
        }
        ::-webkit-scrollbar-track {
            background: #ffe4e6;
        }
        ::-webkit-scrollbar-thumb {
            background: #c026d3;
            border-radius: 4px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #a21caf;
        }
    </style>
</head>
<body class="text-gray-800 min-h-screen flex flex-col antialiased">

    <!-- Navigation Header -->
    <header class="bg-gradient-to-r from-pink-500 via-magenta-600 to-pink-700 text-white shadow-lg no-print sticky top-0 z-40">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-3">
            <div class="flex flex-wrap justify-between items-center gap-4">
                <div class="flex items-center space-x-3">
                    <div class="bg-white p-2 rounded-xl text-magenta-600 shadow-md">
                        <i class="fa-solid fa-graduation-cap text-2xl"></i>
                    </div>
                    <div>
                        <h1 class="font-extrabold text-xl sm:text-2xl tracking-tight leading-tight">SK BUKIT KUCHAI</h1>
                        <p class="text-xs sm:text-sm text-pink-100 font-medium">Dashboard Rekod Perkembangan Murid (PBD)</p>
                    </div>
                </div>

                <div class="flex flex-wrap items-center gap-2">
                    <!-- Auto Sync Indicator Toggle -->
                    <label class="inline-flex items-center cursor-pointer bg-white/20 hover:bg-white/30 text-white text-xs px-3 py-2 rounded-lg transition">
                        <input type="checkbox" id="autoSyncToggle" onchange="toggleAutoSync(this.checked)" class="sr-only peer" checked>
                        <div class="relative w-7 h-4 bg-gray-300 peer-focus:outline-none rounded-full peer peer-checked:after:translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-[2px] after:left-[2px] after:bg-white after:border-gray-300 after:border after:rounded-full after:h-3 after:w-3 after:transition-all peer-checked:bg-emerald-400"></div>
                        <span class="ml-2 font-medium">Auto-Sync (30s)</span>
                    </label>

                    <button onclick="syncDataFromGoogleSheet(true)" title="Kemaskini data dari Google Sheet sekarang" class="bg-white/20 hover:bg-white/30 text-white text-xs sm:text-sm font-semibold py-2 px-3 rounded-lg flex items-center gap-2 transition duration-200">
                        <i class="fa-solid fa-rotate text-pink-200" id="syncIcon"></i>
                        <span>Sync Google Sheet</span>
                    </button>
                    <button onclick="saveAllDataToLocal()" title="Simpan perubahan tempatan" class="bg-magenta-700 hover:bg-magenta-800 text-white text-xs sm:text-sm font-semibold py-2 px-4 rounded-lg shadow-md flex items-center gap-2 transition duration-200">
                        <i class="fa-solid fa-floppy-disk"></i>
                        <span>Simpan Perubahan</span>
                    </button>
                </div>
            </div>
        </div>
    </header>

    <!-- Global Control Bar / Filters -->
    <section class="bg-white border-b border-pink-200 shadow-sm no-print sticky top-[61px] z-30">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-3">
            <div class="grid grid-cols-2 md:grid-cols-4 lg:grid-cols-5 gap-3 items-center">
                
                <!-- Dropdown Tahun -->
                <div>
                    <label class="block text-2xs font-bold text-gray-600 uppercase tracking-wider mb-1">Tahun</label>
                    <select id="filterTahun" onchange="handleFilterChange()" class="w-full bg-pink-50 border border-pink-300 text-gray-800 text-xs sm:text-sm rounded-lg focus:ring-magenta-500 focus:border-magenta-500 p-2 font-semibold">
                        <option value="1">Tahun 1</option>
                        <option value="2">Tahun 2</option>
                        <option value="3">Tahun 3</option>
                        <option value="4">Tahun 4 (Tahap 2)</option>
                        <option value="5">Tahun 5 (Tahap 2)</option>
                        <option value="6">Tahun 6 (Tahap 2)</option>
                    </select>
                </div>

                <!-- Dropdown Kelas -->
                <div>
                    <label class="block text-2xs font-bold text-gray-600 uppercase tracking-wider mb-1">Kelas</label>
                    <select id="filterKelas" onchange="handleFilterChange()" class="w-full bg-pink-50 border border-pink-300 text-gray-800 text-xs sm:text-sm rounded-lg focus:ring-magenta-500 focus:border-magenta-500 p-2 font-semibold">
                        <option value="INOVATIF">INOVATIF</option>
                        <option value="KREATIF">KREATIF</option>
                        <option value="PROAKTIF">PROAKTIF</option>
                    </select>
                </div>

                <!-- Dropdown Subjek -->
                <div>
                    <div class="flex justify-between items-center mb-1">
                        <label class="block text-2xs font-bold text-gray-600 uppercase tracking-wider">Subjek</label>
                        <button onclick="addNewSubjekModal()" title="Tambah Subjek Baru" class="text-2xs bg-magenta-100 hover:bg-magenta-200 text-magenta-700 font-bold px-1.5 py-0.5 rounded flex items-center gap-1 transition">
                            <i class="fa-solid fa-plus text-[10px]"></i> Subjek
                        </button>
                    </div>
                    <select id="filterSubjek" onchange="handleFilterChange()" class="w-full bg-pink-50 border border-pink-300 text-gray-800 text-xs sm:text-sm rounded-lg focus:ring-magenta-500 focus:border-magenta-500 p-2 font-semibold">
                        <!-- Populated dynamically from Google Sheet / State -->
                        <option value="BAHASA MELAYU">Bahasa Melayu</option>
                        <option value="BAHASA INGGERIS">Bahasa Inggeris</option>
                        <option value="MATEMATIK">Matematik</option>
                        <option value="SAINS">Sains</option>
                        <option value="PENDIDIKAN ISLAM">Pendidikan Islam</option>
                        <option value="PENDIDIKAN JASMANI">Pendidikan Jasmani</option>
                        <option value="PENDIDIKAN KESIHATAN">Pendidikan Kesihatan</option>
                        <option value="SEJARAH">Sejarah</option>
                        <option value="REKA BENTUK DAN TEKNOLOGI">RBT</option>
                    </select>
                </div>

                <!-- Dropdown Fasa Penilaian -->
                <div>
                    <label class="block text-2xs font-bold text-gray-600 uppercase tracking-wider mb-1">Fasa Penilaian</label>
                    <select id="filterFasa" onchange="handleFilterChange()" class="w-full bg-pink-50 border border-pink-300 text-gray-800 text-xs sm:text-sm rounded-lg focus:ring-magenta-500 focus:border-magenta-500 p-2 font-semibold">
                        <option value="PERTENGAHAN TAHUN">PBD Pertengahan Tahun</option>
                        <option value="AKHIR TAHUN">PBD Akhir Tahun</option>
                    </select>
                </div>

                <!-- Status Badge -->
                <div class="col-span-2 md:col-span-4 lg:col-span-1 flex items-center justify-end">
                    <span id="levelBadge" class="px-3 py-1.5 rounded-full text-xs font-bold bg-pink-100 text-pink-700 border border-pink-300">
                        <i class="fa-solid fa-layer-group mr-1"></i> Tahap 1
                    </span>
                </div>
            </div>
        </div>
    </section>

    <!-- Tab Navigation -->
    <nav class="bg-pink-100 border-b border-pink-200 no-print">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
            <div class="flex overflow-x-auto space-x-2 py-2 text-sm font-semibold">
                <button onclick="switchTab('tab-dashboard')" id="btn-tab-dashboard" class="tab-btn px-4 py-2 rounded-lg bg-magenta-600 text-white shadow-sm flex items-center gap-2 transition whitespace-nowrap">
                    <i class="fa-solid fa-chart-pie"></i> Analisis TP & Grafik
                </button>
                <button onclick="switchTab('tab-assessment')" id="btn-tab-assessment" class="tab-btn px-4 py-2 rounded-lg text-gray-700 hover:bg-pink-200 flex items-center gap-2 transition whitespace-nowrap">
                    <i class="fa-solid fa-list-check"></i> Borang Pentaksiran SP
                </button>
                <button onclick="switchTab('tab-students')" id="btn-tab-students" class="tab-btn px-4 py-2 rounded-lg text-gray-700 hover:bg-pink-200 flex items-center gap-2 transition whitespace-nowrap">
                    <i class="fa-solid fa-users"></i> Pengurusan Murid
                </button>
                <button onclick="switchTab('tab-report')" id="btn-tab-report" class="tab-btn px-4 py-2 rounded-lg text-gray-700 hover:bg-pink-200 flex items-center gap-2 transition whitespace-nowrap">
                    <i class="fa-solid fa-file-pdf"></i> Laporan & Cetakan PDF
                </button>
                <button onclick="switchTab('tab-settings')" id="btn-tab-settings" class="tab-btn px-4 py-2 rounded-lg text-gray-700 hover:bg-pink-200 flex items-center gap-2 transition whitespace-nowrap">
                    <i class="fa-solid fa-sliders"></i> Tetapan Database
                </button>
            </div>
        </div>
    </nav>

    <!-- Main Content Container -->
    <main class="flex-grow max-w-7xl w-full mx-auto px-4 sm:px-6 lg:px-8 py-6">

        <!-- Toast Notification Alert -->
        <div id="toastNotification" class="hidden no-print mb-4 p-4 rounded-xl shadow-md bg-emerald-500 text-white flex justify-between items-center transition duration-300">
            <div class="flex items-center gap-3">
                <i class="fa-solid fa-circle-check text-xl"></i>
                <span id="toastMessage" class="font-medium text-sm">Maklumat berjaya dikemaskini.</span>
            </div>
            <button onclick="hideToast()" class="text-white hover:text-gray-200 font-bold text-lg">&times;</button>
        </div>

        <!-- ================= TAB 1: DASHBOARD ANALISIS ================= -->
        <section id="tab-dashboard" class="tab-content block space-y-6">
            <!-- Stat Cards -->
            <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4">
                <div class="bg-white p-5 rounded-2xl border border-pink-200 shadow-sm flex items-center justify-between">
                    <div>
                        <p class="text-2xs font-bold uppercase text-gray-500">Jumlah Murid</p>
                        <h3 id="statTotalStudents" class="text-3xl font-extrabold text-pink-600 mt-1">0</h3>
                        <p class="text-xs text-gray-400 mt-1" id="statClassInfo">Tahun 1 INOVATIF</p>
                    </div>
                    <div class="w-12 h-12 rounded-xl bg-pink-100 flex items-center justify-center text-magenta-600">
                        <i class="fa-solid fa-user-graduate text-xl"></i>
                    </div>
                </div>

                <div class="bg-white p-5 rounded-2xl border border-pink-200 shadow-sm flex items-center justify-between">
                    <div>
                        <p class="text-2xs font-bold uppercase text-gray-500">Minima Penguasaan</p>
                        <h3 id="statPassRate" class="text-3xl font-extrabold text-emerald-600 mt-1">0%</h3>
                        <p class="text-xs text-gray-400 mt-1">(Capai TP3 hingga TP6)</p>
                    </div>
                    <div class="w-12 h-12 rounded-xl bg-emerald-100 flex items-center justify-center text-emerald-600">
                        <i class="fa-solid fa-chart-line text-xl"></i>
                    </div>
                </div>

                <div class="bg-white p-5 rounded-2xl border border-pink-200 shadow-sm flex items-center justify-between">
                    <div>
                        <p class="text-2xs font-bold uppercase text-gray-500">Mod TP Kebanyakan</p>
                        <h3 id="statModeTP" class="text-3xl font-extrabold text-magenta-600 mt-1">TP -</h3>
                        <p class="text-xs text-gray-400 mt-1">Rumusan Keseluruhan</p>
                    </div>
                    <div class="w-12 h-12 rounded-xl bg-magenta-100 flex items-center justify-center text-magenta-600">
                        <i class="fa-solid fa-award text-xl"></i>
                    </div>
                </div>

                <div id="statMarkahCard" class="bg-white p-5 rounded-2xl border border-pink-200 shadow-sm flex items-center justify-between">
                    <div>
                        <p class="text-2xs font-bold uppercase text-gray-500">Purata Markah (Tahap 2)</p>
                        <h3 id="statAvgMarkah" class="text-3xl font-extrabold text-amber-600 mt-1">N/A</h3>
                        <p class="text-xs text-gray-400 mt-1" id="statMarkahSub">Hanya Tahun 4, 5, 6</p>
                    </div>
                    <div class="w-12 h-12 rounded-xl bg-amber-100 flex items-center justify-center text-amber-600">
                        <i class="fa-solid fa-percent text-xl"></i>
                    </div>
                </div>
            </div>

            <!-- Charts Grid -->
            <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
                <!-- Bar Chart TP Distribution -->
                <div class="bg-white p-5 rounded-2xl border border-pink-200 shadow-sm">
                    <div class="flex justify-between items-center mb-4">
                        <h3 class="font-bold text-gray-800 text-base flex items-center gap-2">
                            <i class="fa-solid fa-chart-column text-magenta-600"></i> Taburan Tahap Penguasaan (TP1 - TP6)
                        </h3>
                        <span class="text-xs font-semibold px-2.5 py-1 bg-pink-100 text-pink-700 rounded-md" id="chartLabelSub">BAHASA MELAYU</span>
                    </div>
                    <div class="relative h-64">
                        <canvas id="tpBarChart"></canvas>
                    </div>
                </div>

                <!-- Donut Chart TP Proportion -->
                <div class="bg-white p-5 rounded-2xl border border-pink-200 shadow-sm">
                    <div class="flex justify-between items-center mb-4">
                        <h3 class="font-bold text-gray-800 text-base flex items-center gap-2">
                            <i class="fa-solid fa-chart-pie text-magenta-600"></i> Peratusan Pencapaian Murid
                        </h3>
                    </div>
                    <div class="relative h-64 flex justify-center">
                        <canvas id="tpPieChart"></canvas>
                    </div>
                </div>
            </div>

            <!-- Ringkasan Senarai Murid Quick Overview Table -->
            <div class="bg-white rounded-2xl border border-pink-200 shadow-sm overflow-hidden">
                <div class="p-5 border-b border-pink-100 flex flex-wrap justify-between items-center gap-2">
                    <h3 class="font-bold text-gray-800 text-base flex items-center gap-2">
                        <i class="fa-solid fa-table-list text-magenta-600"></i> Ringkasan Pencapaian Kelas (<span id="quickTableTitle">Tahun 1 INOVATIF</span>)
                    </h3>
                    <button onclick="switchTab('tab-assessment')" class="text-xs bg-pink-100 hover:bg-pink-200 text-pink-800 font-bold py-1.5 px-3 rounded-lg transition">
                        Kemaskini TP Murid &rarr;
                    </button>
                </div>
                <div class="overflow-x-auto">
                    <table class="w-full text-sm text-left text-gray-600">
                        <thead class="text-xs text-gray-700 uppercase bg-pink-50 border-b border-pink-100">
                            <tr>
                                <th class="px-4 py-3">#</th>
                                <th class="px-4 py-3">Nama Murid</th>
                                <th class="px-4 py-3 text-center">Rumusan TP Keseluruhan</th>
                                <th class="px-4 py-3 text-center markah-col hidden">Markah Ujian (%)</th>
                                <th class="px-4 py-3 text-center">Status Penguasaan</th>
                            </tr>
                        </thead>
                        <tbody id="quickOverviewTbody">
                            <!-- Populated dynamically -->
                        </tbody>
                    </table>
                </div>
            </div>
        </section>

        <!-- ================= TAB 2: BORANG PENTAKSIRAN SP ================= -->
        <section id="tab-assessment" class="tab-content hidden space-y-6">
            <div class="bg-white p-5 rounded-2xl border border-pink-200 shadow-sm">
                <div class="flex flex-wrap justify-between items-center gap-4 mb-4">
                    <div>
                        <h2 class="text-lg font-bold text-gray-800 flex items-center gap-2">
                            <i class="fa-solid fa-clipboard-check text-magenta-600"></i>
                            Borang Penilaian Standard Pembelajaran (SP)
                        </h2>
                        <p class="text-xs text-gray-500">
                            Ketik butang 1 hingga 6 bagi setiap SP murid. Hasil Rumusan TP dikira di penghujung kolum.
                        </p>
                    </div>

                    <div class="flex items-center gap-2">
                        <button onclick="addCustomSP()" class="bg-pink-100 hover:bg-pink-200 text-pink-700 text-xs font-bold py-2 px-3 rounded-lg border border-pink-300 transition flex items-center gap-1">
                            <i class="fa-solid fa-plus"></i> Tambah SP
                        </button>
                        <button onclick="saveAllDataToLocal()" class="bg-magenta-600 hover:bg-magenta-700 text-white text-xs font-bold py-2 px-4 rounded-lg shadow transition flex items-center gap-1">
                            <i class="fa-solid fa-floppy-disk"></i> Simpan Penilaian
                        </button>
                    </div>
                </div>

                <!-- SP Selection Info Banner -->
                <div id="spListSummary" class="bg-pink-50/70 border border-pink-200 p-3 rounded-xl mb-4 text-xs text-gray-700 flex flex-wrap gap-4 justify-between items-center">
                    <div>
                        <span class="font-bold text-pink-800">Tema/Bidang/SK:</span> <span id="lblTemaBidang">Mendengar dan Tutur</span>
                    </div>
                    <div class="flex items-center gap-2">
                        <button onclick="addNewSubjekModal()" class="bg-pink-200 hover:bg-pink-300 text-pink-800 text-xs font-bold py-1 px-2.5 rounded-lg transition flex items-center gap-1">
                            <i class="fa-solid fa-plus text-[10px]"></i> Subjek Baru
                        </button>
                        <div>
                            <span class="font-bold text-pink-800">Jumlah SP Terlibat:</span> <span id="lblTotalSP" class="font-extrabold text-magenta-600">0 SP</span>
                        </div>
                    </div>
                </div>

                <!-- Assessment Table -->
                <div class="overflow-x-auto rounded-xl border border-pink-200 shadow-inner">
                    <table class="w-full text-xs text-left text-gray-700">
                        <thead class="text-xs uppercase bg-gradient-to-r from-pink-100 to-pink-200 text-pink-900 border-b border-pink-300 font-bold">
                            <tr>
                                <th class="p-3 w-10 text-center">Bil</th>
                                <th class="p-3 min-w-[220px]">Nama Murid (A-Z)</th>
                                <!-- Dynamic Columns for SPs will be generated here -->
                                <th id="spHeaderContainer" class="p-0" colspan="1">
                                    <!-- Dynamic headers generated in JS -->
                                </th>
                                <th class="p-3 text-center bg-pink-300/60 font-extrabold w-32">Rumusan TP Keseluruhan</th>
                                <th class="p-3 text-center bg-amber-200/60 font-extrabold w-28 markah-col hidden">Markah (%)</th>
                            </tr>
                        </thead>
                        <tbody id="assessmentTbody" class="divide-y divide-pink-100 bg-white">
                            <!-- Populated dynamically -->
                        </tbody>
                    </table>
                </div>
            </div>
        </section>

        <!-- ================= TAB 3: PENGURUSAN MURID ================= -->
        <section id="tab-students" class="tab-content hidden space-y-6">
            <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">

                <!-- Left Column: Add Student Forms -->
                <div class="space-y-6">
                    <!-- Add Individual Student -->
                    <div class="bg-white p-5 rounded-2xl border border-pink-200 shadow-sm">
                        <h3 class="font-bold text-gray-800 text-sm mb-3 flex items-center gap-2">
                            <i class="fa-solid fa-user-plus text-magenta-600"></i> Tambah Murid Individu
                        </h3>
                        <form id="formAddIndividual" onsubmit="handleAddIndividualStudent(event)" class="space-y-3">
                            <div>
                                <label class="block text-2xs font-semibold text-gray-600 mb-1">Nama Penuh Murid</label>
                                <input type="text" id="inputStudentName" required placeholder="Contoh: AHMAD DANISH BIN AZMAN" class="w-full bg-pink-50 border border-pink-300 text-gray-800 text-xs sm:text-sm rounded-lg p-2.5 focus:ring-magenta-500 focus:border-magenta-500 uppercase">
                            </div>

                            <div class="grid grid-cols-2 gap-2 text-xs">
                                <div>
                                    <label class="block font-semibold text-gray-600 mb-1">Tahun Target</label>
                                    <input type="text" id="targetTahunDisplay" readonly class="w-full bg-gray-100 border border-gray-300 font-bold text-center text-gray-700 rounded-lg p-2">
                                </div>
                                <div>
                                    <label class="block font-semibold text-gray-600 mb-1">Kelas Target</label>
                                    <input type="text" id="targetKelasDisplay" readonly class="w-full bg-gray-100 border border-gray-300 font-bold text-center text-gray-700 rounded-lg p-2">
                                </div>
                            </div>

                            <button type="submit" class="w-full bg-magenta-600 hover:bg-magenta-700 text-white font-bold py-2 px-4 rounded-lg text-xs sm:text-sm shadow transition">
                                <i class="fa-solid fa-plus mr-1"></i> Tambah Murid Ini
                            </button>
                        </form>
                    </div>

                    <!-- Bulk Import Students -->
                    <div class="bg-white p-5 rounded-2xl border border-pink-200 shadow-sm">
                        <h3 class="font-bold text-gray-800 text-sm mb-2 flex items-center gap-2">
                            <i class="fa-solid fa-users-line text-magenta-600"></i> Tambah Murid secara Pukal (Bulk)
                        </h3>
                        <p class="text-xs text-gray-500 mb-3">
                            Tampal (paste) senarai nama murid baris demi baris dari APDM / Excel mengikut Tahun & Kelas terpilih:
                        </p>
                        <form id="formAddBulk" onsubmit="handleAddBulkStudents(event)" class="space-y-3">
                            <textarea id="bulkStudentNames" rows="5" placeholder="SITI NURHALIZA BINTI ADNAN&#10;MUHAMMAD HAZIQ BIN ISMAIL&#10;NUR AIN BINTI KASSIM" class="w-full bg-pink-50 border border-pink-300 text-gray-800 text-xs rounded-lg p-2.5 focus:ring-magenta-500 focus:border-magenta-500 uppercase font-mono"></textarea>

                            <button type="submit" class="w-full bg-pink-600 hover:bg-pink-700 text-white font-bold py-2 px-4 rounded-lg text-xs sm:text-sm shadow transition">
                                <i class="fa-solid fa-file-import mr-1"></i> Muat Naik Senarai Pukal
                            </button>
                        </form>
                    </div>
                </div>

                <!-- Right Column: Student List Table (A-Z) -->
                <div class="lg:col-span-2 bg-white p-5 rounded-2xl border border-pink-200 shadow-sm">
                    <div class="flex flex-wrap justify-between items-center gap-2 mb-4">
                        <div>
                            <h3 class="font-bold text-gray-800 text-base flex items-center gap-2">
                                <i class="fa-solid fa-address-book text-magenta-600"></i> Senarai Murid Berdaftar (<span id="mgmtClassLabel">Tahun 1 INOVATIF</span>)
                            </h3>
                            <p class="text-xs text-gray-500">Disusun mengikut abjad (A-Z)</p>
                        </div>

                        <div class="flex items-center gap-2">
                            <button onclick="clearAllStudentsInClass()" class="text-xs bg-red-100 hover:bg-red-200 text-red-700 font-bold py-1.5 px-3 rounded-lg border border-red-300 transition">
                                <i class="fa-solid fa-trash-can mr-1"></i> Padam Semua Kelas Ini
                            </button>
                        </div>
                    </div>

                    <div class="overflow-x-auto rounded-xl border border-pink-200">
                        <table class="w-full text-xs text-left text-gray-700">
                            <thead class="text-xs uppercase bg-pink-100 text-pink-900 border-b border-pink-200 font-bold">
                                <tr>
                                    <th class="p-3 w-12 text-center">Bil</th>
                                    <th class="p-3">Nama Penuh Murid</th>
                                    <th class="p-3 text-center">Tahun</th>
                                    <th class="p-3 text-center">Kelas</th>
                                    <th class="p-3 text-center w-24">Tindakan</th>
                                </tr>
                            </thead>
                            <tbody id="studentMgmtTbody" class="divide-y divide-pink-100">
                                <!-- Dynamic rows -->
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </section>

        <!-- ================= TAB 4: LAPORAN & CETAKAN PDF ================= -->
        <section id="tab-report" class="tab-content hidden space-y-6">
            <!-- Controls (no-print) -->
            <div class="bg-white p-5 rounded-2xl border border-pink-200 shadow-sm no-print flex flex-wrap justify-between items-center gap-4">
                <div>
                    <h2 class="text-lg font-bold text-gray-800 flex items-center gap-2">
                        <i class="fa-solid fa-file-invoice text-magenta-600"></i> Laporan Perkembangan Murid (PBD)
                    </h2>
                    <p class="text-xs text-gray-500">Pilih jenis cetakan mengikut kelas atau per individu murid.</p>
                </div>

                <div class="flex flex-wrap items-center gap-2">
                    <select id="reportStudentSelect" onchange="generatePrintReport()" class="bg-pink-50 border border-pink-300 text-xs rounded-lg p-2 font-semibold">
                        <option value="ALL">-- SEMUA MURID KELAS --</option>
                        <!-- Dynamic Student List Options -->
                    </select>

                    <button onclick="window.print()" class="bg-magenta-600 hover:bg-magenta-700 text-white text-xs sm:text-sm font-bold py-2 px-4 rounded-lg shadow transition flex items-center gap-2">
                        <i class="fa-solid fa-print text-base"></i> Cetak / Simpan PDF
                    </button>
                </div>
            </div>

            <!-- Print Printable Template Area -->
            <div id="printReportContainer" class="bg-white p-8 rounded-2xl border border-pink-200 shadow-sm print-card space-y-6">
                <!-- Dynamic print template populated by JS -->
            </div>
        </section>

        <!-- ================= TAB 5: TETAPAN & DATABASE SYNC ================= -->
        <section id="tab-settings" class="tab-content hidden space-y-6">
            <div class="bg-white p-6 rounded-2xl border border-pink-200 shadow-sm max-w-3xl mx-auto space-y-6">
                <div>
                    <h2 class="text-lg font-bold text-gray-800 flex items-center gap-2">
                        <i class="fa-solid fa-database text-magenta-600"></i> Integration & Tetapan Database Google Sheet
                    </h2>
                    <p class="text-xs text-gray-500">
                        Pengurusan penyegerakan automatik dengan Google Sheets.
                    </p>
                </div>

                <!-- Google Sheets Config -->
                <div class="p-4 bg-pink-50 rounded-xl border border-pink-200 space-y-3">
                    <h3 class="text-sm font-bold text-pink-900 flex items-center gap-2">
                        <i class="fa-solid fa-file-csv"></i> Pautan Google Sheets (Published CSV)
                    </h3>
                    <p class="text-xs text-gray-600">
                        Pautan CSV dari Google Sheet terbitan web anda:
                    </p>
                    <input type="text" id="sheetCsvUrl" value="https://docs.google.com/spreadsheets/d/1PYyDVG9lsLiw8Ufs_ZBHUoyL8lLWw7IsQknEuEsLeFs/export?format=csv" class="w-full text-xs font-mono bg-white border border-pink-300 rounded-lg p-2.5 text-gray-700">

                    <div class="flex flex-wrap gap-2 justify-end">
                        <button onclick="syncDataFromGoogleSheet(true)" class="bg-magenta-600 hover:bg-magenta-700 text-white text-xs font-bold py-2 px-4 rounded-lg shadow transition">
                            <i class="fa-solid fa-cloud-arrow-down mr-1"></i> Muat Naik & Selaras Data Sekarang
                        </button>
                    </div>
                </div>

                <!-- Backup & Reset -->
                <div class="p-4 bg-gray-50 rounded-xl border border-gray-200 space-y-3">
                    <h3 class="text-sm font-bold text-gray-800 flex items-center gap-2">
                        <i class="fa-solid fa-toolbox"></i> Pentadbiran Data Tempatan
                    </h3>
                    <div class="flex flex-wrap gap-3">
                        <button onclick="exportDataJSON()" class="bg-emerald-600 hover:bg-emerald-700 text-white text-xs font-bold py-2 px-4 rounded-lg shadow transition">
                            <i class="fa-solid fa-download mr-1"></i> Eksport Data JSON
                        </button>
                        <button onclick="resetDataToDefault()" class="bg-red-600 hover:bg-red-700 text-white text-xs font-bold py-2 px-4 rounded-lg shadow transition">
                            <i class="fa-solid fa-rotate-left mr-1"></i> Reset Ke Data Asal
                        </button>
                    </div>
                </div>
            </div>
        </section>

    </main>

    <!-- Footer -->
    <footer class="bg-pink-100 border-t border-pink-200 py-4 text-center text-xs text-pink-800 font-semibold no-print">
        <p>&copy; 2026 SK BUKIT KUCHAI — Dashboard Rekod Perkembangan Murid (PBD). Hak Cipta Terpelihara.</p>
    </footer>

    <script>
        // ================= APPLICATION STATE & INITIAL DATA =================
        const SPREADSHEET_CSV_URL = "https://docs.google.com/spreadsheets/d/1PYyDVG9lsLiw8Ufs_ZBHUoyL8lLWw7IsQknEuEsLeFs/export?format=csv";

        // Default Curriculum (Fallbacks if sheet is empty or offline)
        const DEFAULT_CURRICULUM = {
            "BAHASA MELAYU": {
                "1": [
                    { id: "sp1", code: "SP 1.1.1", desc: "Mendengar dan menyebut perkataan, frasa dan ayat", sk: "SK 1.1", bidang: "Mendengar & Tutur", tema: "Kemahiran Asas" },
                    { id: "sp2", code: "SP 2.1.1", desc: "Membaca sebutan betul dan intonasi sesuai", sk: "SK 2.1", bidang: "Membaca", tema: "Kemahiran Asas" },
                    { id: "sp3", code: "SP 3.1.1", desc: "Menulis perkataan dan frasa secara mekanis", sk: "SK 3.1", bidang: "Menulis", tema: "Kemahiran Asas" },
                    { id: "sp4", code: "SP 5.1.1", desc: "Memahami dan menggunakan kata nama mengikut konteks", sk: "SK 5.1", bidang: "Seni Bahasa", tema: "Tata-bahasa" }
                ],
                "4": [
                    { id: "sp1", code: "SP 1.1.1", desc: "Mendengar, mengecam dan menyebut frasa", sk: "SK 1.1", bidang: "Mendengar", tema: "Masyarakat" },
                    { id: "sp2", code: "SP 2.2.1", desc: "Membaca dan memahami bahan grafik", sk: "SK 2.2", bidang: "Membaca", tema: "Masyarakat" }
                ]
            },
            "PENDIDIKAN JASMANI": {
                "1": [
                    { id: "pj1", code: "SP 1.1.1", desc: "Melakukan pergerakan yang melibatkan kesedaran tubuh badan", sk: "1.1 Meneroka pelbagai corak pergerakan berdasarkan konsep pergerakan.", bidang: "1. Kemahiran Pergerakan ( Domain Psikomotor )", tema: "Kemahiran : Konsep Pergerakan" },
                    { id: "pj2", code: "SP 1.1.2", desc: "Melakukan pergerakan yang melibatkan kesedaran ruang diri, ruang am dan batasan ruang", sk: "2.1 Menggunakan pengetahuan konsep pergerakan semasa meneroka pelbagai corak pergerakan.", bidang: "2. Aplikasi Pengetahuan Dalam Pergerakan ( Domain Kognitif )", tema: "Kemahiran : Konsep Pergerakan" },
                    { id: "pj3", code: "SP 1.1.6", desc: "Melakukan pergerakan yang berbeza kelajuan berdasarkan tempo, irama dan isyarat.", sk: "2.2 Menggunakan pengetahuan konsep pergerakan dan prinsip mekanik dalam pergerakan lokomotor dan bukan lokomotor.", bidang: "2. Aplikasi Pengetahuan Dalam Pergerakan ( Domain Kognitif )", tema: "Kemahiran : Pergerakan Asas - Kemahiran Lokomotor Dan Bukan Lokomotor." }
                ]
            },
            "MATEMATIK": {
                "1": [
                    { id: "sp1", code: "SP 1.1.1", desc: "Membilang dan menamakan nombor bulat hingga 100", sk: "SK 1.1", bidang: "Nombor & Operasi", tema: "Nombor Bulat" },
                    { id: "sp2", code: "SP 2.1.1", desc: "Menambah dua nombor tanpa mengumpul semula", sk: "SK 2.1", bidang: "Operasi Asas", tema: "Tambah & Tolak" },
                    { id: "sp3", code: "SP 3.1.1", desc: "Mengenal pasti nilai wang kertas dan syiling", sk: "SK 3.1", bidang: "Wang", tema: "Pengurusan Kewangan" }
                ]
            }
        };

        // Seed initial students
        const INITIAL_STUDENTS = [
            { id: "m1", name: "ADAM HARIS BIN ZULKIFLI", tahun: "1", kelas: "INOVATIF" },
            { id: "m2", name: "AINA SOFEA BINTI MOHD FAIZ", tahun: "1", kelas: "INOVATIF" },
            { id: "m3", name: "AMIRUL ASYRAF BIN ROSLI", tahun: "1", kelas: "INOVATIF" },
            { id: "m4", name: "FARAH DIANA BINTI KHALID", tahun: "1", kelas: "INOVATIF" },
            { id: "m5", name: "MUHAMMAD DANISH BIN IMRAN", tahun: "1", kelas: "INOVATIF" },
            { id: "m6", name: "NUR AISYAH BINTI SYAHMI", tahun: "1", kelas: "KREATIF" },
            { id: "m7", name: "RAYYAN MIKHAIL BIN AFIQ", tahun: "1", kelas: "KREATIF" },
            { id: "m8", name: "BATRISYIA QISTINA BINTI HAKIM", tahun: "4", kelas: "INOVATIF" },
            { id: "m9", name: "MUHAMMAD KHAIRUL BIN HASSAN", tahun: "4", kelas: "INOVATIF" },
            { id: "m10", name: "NURUL IMAN BINTI RAZAK", tahun: "4", kelas: "INOVATIF" }
        ];

        // Global State
        let appData = {
            students: [],
            assessments: {}, // Key: `${studentId}_${subjek}_${fasa}` -> { spScores: {spId: tpValue}, markah: number }
            curriculum: DEFAULT_CURRICULUM,
            customSubjects: [] // Stores manually added subject names
        };

        let barChartInstance = null;
        let pieChartInstance = null;
        let autoSyncTimer = null;

        window.onload = function() {
            loadDataFromLocal();
            // Initial sync from Google Sheet
            syncDataFromGoogleSheet(false);
            // Enable auto sync loop default
            toggleAutoSync(true);
            updateUI();
        };

        function loadDataFromLocal() {
            const saved = localStorage.getItem('sk_bukit_kuchai_pbd_data_v2');
            if (saved) {
                try {
                    appData = JSON.parse(saved);
                    if (!appData.curriculum) appData.curriculum = DEFAULT_CURRICULUM;
                } catch(e) {
                    console.error("Error loading local storage:", e);
                    appData.students = INITIAL_STUDENTS;
                }
            } else {
                appData.students = INITIAL_STUDENTS;
                seedDummyAssessments();
            }
        }

        function seedDummyAssessments() {
            appData.students.forEach(st => {
                const key1 = `${st.id}_BAHASA MELAYU_PERTENGAHAN TAHUN`;
                appData.assessments[key1] = {
                    spScores: { "sp1": 4, "sp2": 5, "sp3": 4, "sp4": 3 },
                    markah: parseInt(st.tahun) >= 4 ? 78 : null
                };
            });
        }

        function saveAllDataToLocal() {
            localStorage.setItem('sk_bukit_kuchai_pbd_data_v2', JSON.stringify(appData));
            showToast("Maklumat berjaya disimpan ke memori!");
        }

        function showToast(msg) {
            const toast = document.getElementById('toastNotification');
            const toastMsg = document.getElementById('toastMessage');
            toastMsg.innerText = msg;
            toast.classList.remove('hidden');
            setTimeout(() => {
                toast.classList.add('hidden');
            }, 3500);
        }

        function hideToast() {
            document.getElementById('toastNotification').classList.add('hidden');
        }

        function toggleAutoSync(enable) {
            if (autoSyncTimer) clearInterval(autoSyncTimer);
            if (enable) {
                autoSyncTimer = setInterval(() => {
                    syncDataFromGoogleSheet(false);
                }, 30000); // 30 seconds poll
            }
        }

        function syncDataFromGoogleSheet(isManual = false) {
            const syncIcon = document.getElementById('syncIcon');
            if (syncIcon) syncIcon.classList.add('fa-spin');

            const csvUrl = document.getElementById('sheetCsvUrl')?.value || SPREADSHEET_CSV_URL;

            Papa.parse(csvUrl, {
                download: true,
                header: true,
                skipEmptyLines: true,
                complete: function(results) {
                    if (syncIcon) syncIcon.classList.remove('fa-spin');
                    if (results.data && results.data.length > 0) {
                        processGoogleSheetData(results.data);
                        if (isManual) showToast("Data Google Sheets berjaya diselaraskan!");
                    }
                },
                error: function(err) {
                    if (syncIcon) syncIcon.classList.remove('fa-spin');
                    if (isManual) console.warn("Could not fetch CSV directly.", err);
                }
            });
        }

        function processGoogleSheetData(rows) {
            let updatedStudentsCount = 0;

            rows.forEach(row => {
                // Parse Headers dynamically
                const nama = (row['Nama Murid'] || row['NAMA MURID'] || row['Nama'] || row['NAMA'] || '').trim();
                const tahun = (row['TAHUN'] || row['Tahun'] || '').trim().replace('Tahun', '').trim();
                const kelas = (row['KELAS'] || row['Kelas'] || '').trim().toUpperCase();
                const subjek = (row['SUBJEK'] || row['Subjek'] || row['MATA PELAJARAN'] || 'BAHASA MELAYU').trim().toUpperCase();

                const tema = (row['TEMA'] || row['Tema'] || 'ASAS').trim();
                const bidang = (row['TAJUK'] || row['BIDANG'] || row['Tajuk'] || row['Bidang'] || 'UMUM').trim();
                const sk = (row['STANDARD KANDUNGAN (SK)'] || row['STANDARD KANDUNGAN'] || row['SK'] || '').trim();
                const sp = (row['STANDARD PEMBELAJARAN (SP)'] || row['STANDARD PEMBELAJARAN'] || row['SP'] || '').trim();

                // Auto register subject into customSubjects list if new
                if (subjek && !appData.customSubjects.includes(subjek)) {
                    appData.customSubjects.push(subjek);
                }

                // 1. Update Student Database if name present
                if (nama && tahun && kelas) {
                    let student = appData.students.find(s => s.name.toUpperCase() === nama.toUpperCase() && s.tahun === tahun && s.kelas === kelas);

                    if (!student) {
                        student = {
                            id: 'm_' + Date.now() + '_' + Math.floor(Math.random()*1000),
                            name: nama.toUpperCase(),
                            tahun: tahun,
                            kelas: kelas
                        };
                        appData.students.push(student);
                        updatedStudentsCount++;
                    }
                }

                // 2. Update Curriculum Database if SP present
                if (subjek && tahun && sp) {
                    if (!appData.curriculum[subjek]) appData.curriculum[subjek] = {};
                    if (!appData.curriculum[subjek][tahun]) appData.curriculum[subjek][tahun] = [];

                    const curList = appData.curriculum[subjek][tahun];
                    
                    // Improved SP Code detection (Handles formats like "1.1.1 Melakukan...", "SP 1.1.1", "1.1.2")
                    let code = "";
                    const spMatchWithPrefix = sp.match(/SP\s*(\d+(\.\d+)*)/i);
                    const spMatchNumberOnly = sp.match(/^(\d+(\.\d+)+)/);

                    if (spMatchWithPrefix) {
                        code = spMatchWithPrefix[0].toUpperCase();
                    } else if (spMatchNumberOnly) {
                        code = `SP ${spMatchNumberOnly[1]}`;
                    } else {
                        code = `SP ${curList.length + 1}`;
                    }

                    const exists = curList.some(item => item.desc === sp || item.code === code);
                    if (!exists) {
                        curList.push({
                            id: 'sp_' + Date.now() + '_' + Math.floor(Math.random()*1000),
                            code: code,
                            desc: sp,
                            sk: sk || 'SK Standard',
                            bidang: bidang,
                            tema: tema
                        });
                    }
                }
            });

            // Refresh Subjek Select Dropdown if new subjects detected
            updateSubjekDropdownOptions();
            saveAllDataToLocal();
            updateUI();
        }

        function addNewSubjekModal() {
            const newSubjekInput = prompt("Masukkan Nama Subjek / Mata Pelajaran Baru (Contoh: PENDIDIKAN JASMANI, BAHASA ARAB, PENDIDIKAN MUZIK):");
            if (!newSubjekInput || !newSubjekInput.trim()) return;

            const subjekClean = newSubjekInput.trim().toUpperCase();

            if (!appData.customSubjects) appData.customSubjects = [];
            if (!appData.customSubjects.includes(subjekClean)) {
                appData.customSubjects.push(subjekClean);
            }

            if (!appData.curriculum[subjekClean]) {
                appData.curriculum[subjekClean] = {};
            }

            saveAllDataToLocal();
            updateSubjekDropdownOptions();

            // Set current active subject filter to newly added subject
            const subjekSelect = document.getElementById('filterSubjek');
            subjekSelect.value = subjekClean;
            handleFilterChange();

            showToast(`Subjek "${subjekClean}" berjaya ditambah!`);
        }

        function updateSubjekDropdownOptions() {
            const subjekSelect = document.getElementById('filterSubjek');
            const currentSelected = subjekSelect.value;
            
            const defaultSubjects = [
                "BAHASA MELAYU", 
                "BAHASA INGGERIS", 
                "MATEMATIK", 
                "SAINS", 
                "PENDIDIKAN ISLAM", 
                "PENDIDIKAN JASMANI", 
                "PENDIDIKAN KESIHATAN", 
                "PENDIDIKAN MORAL", 
                "PENDIDIKAN SENI VISUAL", 
                "PENDIDIKAN MUZIK", 
                "BAHASA ARAB", 
                "SEJARAH", 
                "REKA BENTUK DAN TEKNOLOGI"
            ];
            
            const subjectsFromCurriculum = Object.keys(appData.curriculum || {});
            const customSubs = appData.customSubjects || [];

            const allSubjects = Array.from(new Set([...defaultSubjects, ...subjectsFromCurriculum, ...customSubs]));

            subjekSelect.innerHTML = '';
            allSubjects.forEach(s => {
                const opt = document.createElement('option');
                opt.value = s;
                opt.innerText = s;
                if (s === currentSelected) opt.selected = true;
                subjekSelect.appendChild(opt);
            });
        }

        function getFilters() {
            return {
                tahun: document.getElementById('filterTahun').value,
                kelas: document.getElementById('filterKelas').value,
                subjek: document.getElementById('filterSubjek').value,
                fasa: document.getElementById('filterFasa').value
            };
        }

        function handleFilterChange() {
            updateUI();
        }

        function switchTab(tabId) {
            document.querySelectorAll('.tab-content').forEach(el => el.classList.add('hidden'));
            document.getElementById(tabId).classList.remove('hidden');

            document.querySelectorAll('.tab-btn').forEach(btn => {
                btn.classList.remove('bg-magenta-600', 'text-white', 'shadow-sm');
                btn.classList.add('text-gray-700');
            });

            const activeBtn = document.getElementById('btn-' + tabId);
            if (activeBtn) {
                activeBtn.classList.add('bg-magenta-600', 'text-white', 'shadow-sm');
                activeBtn.classList.remove('text-gray-700');
            }

            if (tabId === 'tab-dashboard') renderCharts();
            if (tabId === 'tab-report') generatePrintReport();
        }

        function updateUI() {
            const filter = getFilters();
            const isTahap2 = parseInt(filter.tahun) >= 4;

            // Update Badge
            const badge = document.getElementById('levelBadge');
            badge.innerHTML = isTahap2 
                ? `<i class="fa-solid fa-layer-group mr-1"></i> Tahap 2 (Markah Aktif)` 
                : `<i class="fa-solid fa-layer-group mr-1"></i> Tahap 1`;
            badge.className = isTahap2 
                ? "px-3 py-1.5 rounded-full text-xs font-bold bg-purple-100 text-purple-700 border border-purple-300"
                : "px-3 py-1.5 rounded-full text-xs font-bold bg-pink-100 text-pink-700 border border-pink-300";

            // Update Markah Columns Visibility
            document.querySelectorAll('.markah-col').forEach(col => {
                if (isTahap2) col.classList.remove('hidden');
                else col.classList.add('hidden');
            });

            // Target displays for student mgmt
            document.getElementById('targetTahunDisplay').value = `Tahun ${filter.tahun}`;
            document.getElementById('targetKelasDisplay').value = filter.kelas;
            document.getElementById('mgmtClassLabel').innerText = `Tahun ${filter.tahun} ${filter.kelas}`;
            document.getElementById('quickTableTitle').innerText = `Tahun ${filter.tahun} ${filter.kelas} - ${filter.subjek}`;

            // Render Views
            renderQuickOverviewTable();
            renderAssessmentGrid();
            renderStudentManagementTable();
            renderCharts();
            populateReportStudentSelect();
        }

        // Get Filtered & Sorted Student List (A-Z)
        function getFilteredStudents() {
            const filter = getFilters();
            return appData.students
                .filter(s => s.tahun === filter.tahun && s.kelas === filter.kelas)
                .sort((a, b) => a.name.localeCompare(b.name));
        }

        // Get Current SPs for Subjek & Tahun
        function getCurrentSPs() {
            const filter = getFilters();
            if (appData.curriculum[filter.subjek] && appData.curriculum[filter.subjek][filter.tahun]) {
                return appData.curriculum[filter.subjek][filter.tahun];
            }
            // Fallback default
            return (DEFAULT_CURRICULUM["BAHASA MELAYU"]["1"]);
        }

        // Helper: Calculate Overall TP summary for student
        function calculateOverallTP(studentId, subjek, fasa) {
            const key = `${studentId}_${subjek}_${fasa}`;
            const record = appData.assessments[key];
            if (!record || !record.spScores) return 0;

            const sps = getCurrentSPs();
            let sum = 0;
            let count = 0;

            sps.forEach(sp => {
                const score = record.spScores[sp.id];
                if (score && score > 0) {
                    sum += score;
                    count++;
                }
            });

            if (count === 0) return 0;
            return Math.round(sum / count);
        }

        function renderAssessmentGrid() {
            const students = getFilteredStudents();
            const sps = getCurrentSPs();
            const filter = getFilters();
            const isTahap2 = parseInt(filter.tahun) >= 4;

            // Render SP Table Headers
            const headerContainer = document.getElementById('spHeaderContainer');
            if (sps.length === 0) {
                headerContainer.innerHTML = `<div class="p-2 text-center text-gray-400 italic">Tiada SP didaftarkan bagi subjek/tahun ini. Sila tambah SP secara manual atau kemaskini Google Sheet.</div>`;
            } else {
                let headersHTML = `<div class="grid grid-cols-${sps.length} divide-x divide-pink-300">`;
                sps.forEach(sp => {
                    headersHTML += `
                        <div class="p-2 text-center text-2xs font-bold text-pink-900 min-w-[130px]">
                            <div class="text-magenta-700 font-extrabold">${sp.code}</div>
                            <div class="truncate text-[10px] text-gray-600" title="${sp.desc}">${sp.desc}</div>
                        </div>
                    `;
                });
                headersHTML += `</div>`;
                headerContainer.innerHTML = headersHTML;
            }

            // Render Info Banner
            if (sps.length > 0) {
                document.getElementById('lblTemaBidang').innerText = `${sps[0].tema} / ${sps[0].bidang} (${sps[0].sk})`;
                document.getElementById('lblTotalSP').innerText = `${sps.length} SP`;
            } else {
                document.getElementById('lblTemaBidang').innerText = `-`;
                document.getElementById('lblTotalSP').innerText = `0 SP`;
            }

            // Render Rows
            const tbody = document.getElementById('assessmentTbody');
            tbody.innerHTML = '';

            if (students.length === 0) {
                tbody.innerHTML = `<tr><td colspan="10" class="p-6 text-center text-gray-400 font-semibold">Tiada murid didaftarkan untuk Tahun ${filter.tahun} ${filter.kelas}. Sila tambah murid di tab 'Pengurusan Murid'.</td></tr>`;
                return;
            }

            students.forEach((st, idx) => {
                const key = `${st.id}_${filter.subjek}_${filter.fasa}`;
                const assessment = appData.assessments[key] || { spScores: {}, markah: '' };
                const overallTP = calculateOverallTP(st.id, filter.subjek, filter.fasa);

                let tr = document.createElement('tr');
                tr.className = "hover:bg-pink-50/50 transition";

                let html = `
                    <td class="p-2.5 text-center font-bold text-gray-500">${idx + 1}</td>
                    <td class="p-2.5 font-bold text-gray-800 uppercase tracking-wide">${st.name}</td>
                    <td class="p-0">
                        <div class="grid grid-cols-${sps.length || 1} divide-x divide-pink-100 items-center">
                `;

                if (sps.length === 0) {
                    html += `<div class="p-2 text-center text-gray-400 text-xs italic">Sila tambah SP dahulu</div>`;
                } else {
                    // SP Radio Boxes
                    sps.forEach(sp => {
                        const currentVal = assessment.spScores[sp.id] || 0;
                        html += `<div class="p-2 flex flex-wrap justify-center gap-1 min-w-[130px]">`;
                        for (let tp = 1; tp <= 6; tp++) {
                            const isChecked = currentVal === tp;
                            html += `
                                <button type="button" onclick="setStudentSPScore('${st.id}', '${sp.id}', ${tp})" 
                                    class="w-5 h-5 sm:w-6 sm:h-6 text-[10px] font-bold rounded-md border transition ${
                                        isChecked 
                                        ? 'bg-magenta-600 text-white border-magenta-700 shadow-sm scale-110' 
                                        : 'bg-white text-gray-600 border-gray-300 hover:bg-pink-100'
                                    }">
                                    ${tp}
                                </button>
                            `;
                        }
                        html += `</div>`;
                    });
                }

                html += `
                        </div>
                    </td>
                    <td class="p-2.5 text-center bg-pink-50 font-extrabold text-sm sm:text-base text-magenta-700">
                        ${overallTP > 0 ? `<span class="inline-block px-3 py-1 bg-magenta-100 border border-magenta-300 rounded-lg">TP ${overallTP}</span>` : '<span class="text-gray-300 text-xs font-normal">Belum Dinilai</span>'}
                    </td>
                `;

                if (isTahap2) {
                    html += `
                        <td class="p-2 text-center bg-amber-50">
                            <input type="number" min="0" max="100" placeholder="%" value="${assessment.markah || ''}" 
                                onchange="setStudentMarkah('${st.id}', this.value)"
                                class="w-16 p-1 text-center font-bold border border-amber-300 rounded-md text-xs focus:ring-amber-500">
                        </td>
                    `;
                }

                tr.innerHTML = html;
                tbody.appendChild(tr);
            });
        }

        function setStudentSPScore(studentId, spId, tpValue) {
            const filter = getFilters();
            const key = `${studentId}_${filter.subjek}_${filter.fasa}`;

            if (!appData.assessments[key]) {
                appData.assessments[key] = { spScores: {}, markah: null };
            }

            if (appData.assessments[key].spScores[spId] === tpValue) {
                appData.assessments[key].spScores[spId] = 0;
            } else {
                appData.assessments[key].spScores[spId] = tpValue;
            }

            saveAllDataToLocal();
            renderAssessmentGrid();
            renderQuickOverviewTable();
            renderCharts();
        }

        function setStudentMarkah(studentId, markahVal) {
            const filter = getFilters();
            const key = `${studentId}_${filter.subjek}_${filter.fasa}`;

            if (!appData.assessments[key]) {
                appData.assessments[key] = { spScores: {}, markah: null };
            }

            appData.assessments[key].markah = parseFloat(markahVal) || 0;
            saveAllDataToLocal();
            renderQuickOverviewTable();
            renderCharts();
        }

        function addCustomSP() {
            const filter = getFilters();
            const code = prompt("Masukkan Kod SP (Contoh: SP 2.1.3):", "SP 2.1.3");
            if (!code) return;
            const desc = prompt("Masukkan Keterangan SP:", "Membaca dan memahami ayat secara intonasi betul");
            if (!desc) return;

            const newSp = {
                id: 'sp_' + Date.now(),
                code: code.toUpperCase(),
                desc: desc,
                sk: "SK Custom",
                bidang: "Kemahiran Asas",
                tema: "Standard Pembelajaran"
            };

            if (!appData.curriculum[filter.subjek]) appData.curriculum[filter.subjek] = {};
            if (!appData.curriculum[filter.subjek][filter.tahun]) appData.curriculum[filter.subjek][filter.tahun] = [];

            appData.curriculum[filter.subjek][filter.tahun].push(newSp);
            saveAllDataToLocal();
            updateUI();
            showToast("Standard Pembelajaran baru berjaya ditambah!");
        }

        function renderQuickOverviewTable() {
            const students = getFilteredStudents();
            const filter = getFilters();
            const isTahap2 = parseInt(filter.tahun) >= 4;
            const tbody = document.getElementById('quickOverviewTbody');
            tbody.innerHTML = '';

            if (students.length === 0) {
                tbody.innerHTML = `<tr><td colspan="5" class="p-4 text-center text-gray-400">Tiada rekod murid.</td></tr>`;
                return;
            }

            students.forEach((st, idx) => {
                const key = `${st.id}_${filter.subjek}_${filter.fasa}`;
                const assessment = appData.assessments[key] || { spScores: {}, markah: 0 };
                const overallTP = calculateOverallTP(st.id, filter.subjek, filter.fasa);

                let statusBadge = '<span class="text-xs bg-gray-100 text-gray-600 px-2 py-0.5 rounded-full">Belum Dinilai</span>';
                if (overallTP >= 3) {
                    statusBadge = '<span class="text-xs font-bold bg-emerald-100 text-emerald-700 px-2.5 py-1 rounded-full border border-emerald-200"><i class="fa-solid fa-check mr-1"></i> Mencapai Tahap Minima</span>';
                } else if (overallTP > 0) {
                    statusBadge = '<span class="text-xs font-bold bg-rose-100 text-rose-700 px-2.5 py-1 rounded-full border border-rose-200"><i class="fa-solid fa-triangle-exclamation mr-1"></i> Perlu Bimbingan (TP1-2)</span>';
                }

                let tr = document.createElement('tr');
                tr.className = "border-b border-pink-50 hover:bg-pink-50/40 transition";
                tr.innerHTML = `
                    <td class="px-4 py-3 font-semibold text-gray-500">${idx + 1}</td>
                    <td class="px-4 py-3 font-bold text-gray-800">${st.name}</td>
                    <td class="px-4 py-3 text-center">
                        <span class="font-extrabold px-3 py-1 bg-pink-100 text-magenta-700 rounded-lg">${overallTP > 0 ? 'TP ' + overallTP : '-'}</span>
                    </td>
                    ${isTahap2 ? `<td class="px-4 py-3 text-center font-bold text-amber-700 markah-col">${assessment.markah ? assessment.markah + '%' : '-'}</td>` : ''}
                    <td class="px-4 py-3 text-center">${statusBadge}</td>
                `;
                tbody.appendChild(tr);
            });
        }

        function renderStudentManagementTable() {
            const students = getFilteredStudents();
            const tbody = document.getElementById('studentMgmtTbody');
            tbody.innerHTML = '';

            if (students.length === 0) {
                tbody.innerHTML = `<tr><td colspan="5" class="p-6 text-center text-gray-400">Tiada murid berdaftar dalam kelas ini. Sila tambah murid secara individu atau pukal.</td></tr>`;
                return;
            }

            students.forEach((st, idx) => {
                let tr = document.createElement('tr');
                tr.className = "hover:bg-pink-50/40 transition border-b border-pink-100";
                tr.innerHTML = `
                    <td class="p-3 text-center font-bold text-gray-500">${idx + 1}</td>
                    <td class="p-3 font-bold text-gray-800 uppercase">${st.name}</td>
                    <td class="p-3 text-center font-semibold text-gray-600">Tahun ${st.tahun}</td>
                    <td class="p-3 text-center font-semibold text-pink-700">${st.kelas}</td>
                    <td class="p-3 text-center">
                        <button onclick="deleteStudent('${st.id}')" class="text-xs bg-red-100 hover:bg-red-200 text-red-600 p-2 rounded-lg transition" title="Padam Murid">
                            <i class="fa-solid fa-trash"></i>
                        </button>
                    </td>
                `;
                tbody.appendChild(tr);
            });
        }

        function handleAddIndividualStudent(e) {
            e.preventDefault();
            const filter = getFilters();
            const input = document.getElementById('inputStudentName');
            const name = input.value.trim().toUpperCase();

            if (!name) return;

            const exists = appData.students.some(s => s.name === name && s.tahun === filter.tahun && s.kelas === filter.kelas);
            if (exists) {
                alert("Murid dengan nama ini telah wujud dalam kelas ini!");
                return;
            }

            const newStudent = {
                id: 'm_' + Date.now(),
                name: name,
                tahun: filter.tahun,
                kelas: filter.kelas
            };

            appData.students.push(newStudent);
            input.value = '';
            saveAllDataToLocal();
            updateUI();
            showToast("Murid individu berjaya ditambah!");
        }

        function handleAddBulkStudents(e) {
            e.preventDefault();
            const filter = getFilters();
            const textarea = document.getElementById('bulkStudentNames');
            const lines = textarea.value.split('\n');

            let addedCount = 0;

            lines.forEach(line => {
                const name = line.trim().toUpperCase();
                if (name && name.length > 2) {
                    const exists = appData.students.some(s => s.name === name && s.tahun === filter.tahun && s.kelas === filter.kelas);
                    if (!exists) {
                        appData.students.push({
                            id: 'm_' + Date.now() + '_' + Math.floor(Math.random()*1000),
                            name: name,
                            tahun: filter.tahun,
                            kelas: filter.kelas
                        });
                        addedCount++;
                    }
                }
            });

            textarea.value = '';
            saveAllDataToLocal();
            updateUI();
            showToast(`${addedCount} murid pukal berjaya dimasukkan!`);
        }

        function deleteStudent(studentId) {
            if (confirm("Adakah anda pasti mahu memadamkan rekod murid ini?")) {
                appData.students = appData.students.filter(s => s.id !== studentId);
                saveAllDataToLocal();
                updateUI();
                showToast("Murid berjaya dipadam!");
            }
        }

        function clearAllStudentsInClass() {
            const filter = getFilters();
            if (confirm(`Adakah anda pasti mahu memadamkan SEMUA murid bagi Tahun ${filter.tahun} ${filter.kelas}?`)) {
                appData.students = appData.students.filter(s => !(s.tahun === filter.tahun && s.kelas === filter.kelas));
                saveAllDataToLocal();
                updateUI();
                showToast("Semua murid kelas dipadam!");
            }
        }

        function renderCharts() {
            const filter = getFilters();
            const students = getFilteredStudents();
            const isTahap2 = parseInt(filter.tahun) >= 4;

            // Stat totals
            document.getElementById('statTotalStudents').innerText = students.length;
            document.getElementById('statClassInfo').innerText = `Tahun ${filter.tahun} ${filter.kelas}`;
            document.getElementById('chartLabelSub').innerText = `${filter.subjek} (${filter.fasa})`;

            const tpCounts = { 1: 0, 2: 0, 3: 0, 4: 0, 5: 0, 6: 0, unassigned: 0 };
            let totalPassed = 0;
            let sumMarkah = 0;
            let markahCount = 0;

            students.forEach(st => {
                const overallTP = calculateOverallTP(st.id, filter.subjek, filter.fasa);
                if (overallTP > 0) {
                    tpCounts[overallTP] = (tpCounts[overallTP] || 0) + 1;
                    if (overallTP >= 3) totalPassed++;
                } else {
                    tpCounts.unassigned++;
                }

                if (isTahap2) {
                    const key = `${st.id}_${filter.subjek}_${filter.fasa}`;
                    const markah = appData.assessments[key]?.markah;
                    if (markah !== undefined && markah !== null && markah !== '') {
                        sumMarkah += parseFloat(markah);
                        markahCount++;
                    }
                }
            });

            // Minimum mastery rate (TP3+)
            const evaluatedCount = students.length - tpCounts.unassigned;
            const passRate = evaluatedCount > 0 ? Math.round((totalPassed / evaluatedCount) * 100) : 0;
            document.getElementById('statPassRate').innerText = `${passRate}%`;

            // Mode TP
            let maxCount = -1;
            let modeTP = '-';
            for (let tp = 1; tp <= 6; tp++) {
                if (tpCounts[tp] > maxCount && tpCounts[tp] > 0) {
                    maxCount = tpCounts[tp];
                    modeTP = `TP${tp}`;
                }
            }
            document.getElementById('statModeTP').innerText = modeTP;

            // Avg Markah
            if (isTahap2 && markahCount > 0) {
                const avg = (sumMarkah / markahCount).toFixed(1);
                document.getElementById('statAvgMarkah').innerText = `${avg}%`;
                document.getElementById('statMarkahSub').innerText = `${markahCount} murid dinilai`;
            } else {
                document.getElementById('statAvgMarkah').innerText = 'N/A';
                document.getElementById('statMarkahSub').innerText = isTahap2 ? 'Tiada markah diisi' : 'Hanya Tahap 2';
            }

            // Render Chart.js Bar Chart
            const ctxBar = document.getElementById('tpBarChart').getContext('2d');
            if (barChartInstance) barChartInstance.destroy();

            barChartInstance = new Chart(ctxBar, {
                type: 'bar',
                data: {
                    labels: ['TP 1', 'TP 2', 'TP 3', 'TP 4', 'TP 5', 'TP 6'],
                    datasets: [{
                        label: 'Bilangan Murid',
                        data: [tpCounts[1], tpCounts[2], tpCounts[3], tpCounts[4], tpCounts[5], tpCounts[6]],
                        backgroundColor: [
                            '#f87171', // TP1 Red
                            '#fb923c', // TP2 Orange
                            '#facc15', // TP3 Yellow
                            '#60a5fa', // TP4 Blue
                            '#a78bfa', // TP5 Purple
                            '#34d399'  // TP6 Green
                        ],
                        borderRadius: 8
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: { display: false }
                    },
                    scales: {
                        y: {
                            beginAtZero: true,
                            ticks: { stepSize: 1 }
                        }
                    }
                }
            });

            // Render Chart.js Donut/Pie Chart
            const ctxPie = document.getElementById('tpPieChart').getContext('2d');
            if (pieChartInstance) pieChartInstance.destroy();

            pieChartInstance = new Chart(ctxPie, {
                type: 'doughnut',
                data: {
                    labels: ['TP 1-2 (Perlu Bimbingan)', 'TP 3-4 (Memuaskan)', 'TP 5-6 (Cemerlang)'],
                    datasets: [{
                        data: [
                            tpCounts[1] + tpCounts[2],
                            tpCounts[3] + tpCounts[4],
                            tpCounts[5] + tpCounts[6]
                        ],
                        backgroundColor: ['#f87171', '#60a5fa', '#34d399']
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: { position: 'bottom' }
                    }
                }
            });
        }

        function populateReportStudentSelect() {
            const select = document.getElementById('reportStudentSelect');
            const students = getFilteredStudents();
            select.innerHTML = `<option value="ALL">-- SEMUA MURID KELAS --</option>`;

            students.forEach(st => {
                select.innerHTML += `<option value="${st.id}">${st.name}</option>`;
            });
        }

        function generatePrintReport() {
            const filter = getFilters();
            const students = getFilteredStudents();
            const sps = getCurrentSPs();
            const selectedStudentId = document.getElementById('reportStudentSelect').value;
            const isTahap2 = parseInt(filter.tahun) >= 4;

            const container = document.getElementById('printReportContainer');
            container.innerHTML = '';

            let printStudents = students;
            if (selectedStudentId !== 'ALL') {
                printStudents = students.filter(s => s.id === selectedStudentId);
            }

            if (printStudents.length === 0) {
                container.innerHTML = `<p class="text-center text-gray-500 py-8">Tiada maklumat murid untuk dijana bagi cetakan.</p>`;
                return;
            }

            printStudents.forEach((st, idx) => {
                const key = `${st.id}_${filter.subjek}_${filter.fasa}`;
                const assessment = appData.assessments[key] || { spScores: {}, markah: null };
                const overallTP = calculateOverallTP(st.id, filter.subjek, filter.fasa);

                let reportCard = document.createElement('div');
                reportCard.className = `p-6 bg-white border border-pink-200 rounded-xl space-y-4 ${idx > 0 ? 'page-break mt-6' : ''}`;

                let html = `
                    <!-- Header Sekolah -->
                    <div class="text-center border-b border-gray-300 pb-4">
                        <h2 class="text-lg font-black tracking-wide text-gray-900">SEKOLAH KEBANGSAAN BUKIT KUCHAI</h2>
                        <p class="text-xs text-gray-600 font-semibold">LAPORAN REKOD PERKEMBANGAN MURID (PBD)</p>
                        <p class="text-2xs uppercase tracking-widest text-pink-700 font-bold mt-1">FASA PENILAIAN: ${filter.fasa}</p>
                    </div>

                    <!-- Maklumat Murid -->
                    <div class="grid grid-cols-2 text-xs gap-2 bg-pink-50/50 p-3 rounded-lg border border-pink-100 font-semibold text-gray-700">
                        <div><span class="text-gray-500">NAMA MURID:</span> <strong class="text-gray-900">${st.name}</strong></div>
                        <div><span class="text-gray-500">SUBJEK:</span> <strong class="text-pink-800">${filter.subjek}</strong></div>
                        <div><span class="text-gray-500">TAHUN & KELAS:</span> <strong>TAHUN ${st.tahun} ${st.kelas}</strong></div>
                        <div><span class="text-gray-500">TARIKH CETAKAN:</span> <strong>${new Date().toLocaleDateString('ms-MY')}</strong></div>
                    </div>

                    <!-- Jadual Standard Pembelajaran & TP -->
                    <table class="w-full text-xs text-left border-collapse border border-gray-300">
                        <thead>
                            <tr class="bg-gray-100 text-gray-800 font-bold text-2xs uppercase border-b border-gray-300">
                                <th class="p-2 border border-gray-300 w-16 text-center">Kod SP</th>
                                <th class="p-2 border border-gray-300">Standard Pembelajaran (SP) / Tajuk</th>
                                <th class="p-2 border border-gray-300 text-center w-28">Tahap Penguasaan</th>
                            </tr>
                        </thead>
                        <tbody>
                `;

                if (sps.length === 0) {
                    html += `<tr><td colspan="3" class="p-4 text-center text-gray-400">Tiada rekod Standard Pembelajaran.</td></tr>`;
                } else {
                    sps.forEach(sp => {
                        const tpVal = assessment.spScores[sp.id] || 0;
                        html += `
                            <tr class="border-b border-gray-200">
                                <td class="p-2 border border-gray-300 font-bold text-center">${sp.code}</td>
                                <td class="p-2 border border-gray-300">
                                    <div class="font-semibold text-gray-900">${sp.desc}</div>
                                    <div class="text-[10px] text-gray-500">${sp.tema} | ${sp.bidang}</div>
                                </td>
                                <td class="p-2 border border-gray-300 text-center font-bold">
                                    ${tpVal > 0 ? `<span class="px-2 py-0.5 bg-pink-100 text-pink-800 rounded">TP ${tpVal}</span>` : '<span class="text-gray-400 font-normal">-</span>'}
                                </td>
                            </tr>
                        `;
                    });
                }

                html += `
                        </tbody>
                    </table>

                    <!-- Rumusan PBD & Markah -->
                    <div class="grid grid-cols-2 gap-4 pt-2">
                        <div class="p-3 bg-pink-100/60 border border-pink-300 rounded-lg text-center">
                            <p class="text-2xs font-bold text-gray-600 uppercase">Rumusan TP Keseluruhan</p>
                            <h3 class="text-2xl font-black text-magenta-700 mt-1">${overallTP > 0 ? 'TP ' + overallTP : 'Belum Dinilai'}</h3>
                        </div>

                        ${isTahap2 ? `
                        <div class="p-3 bg-amber-100/60 border border-amber-300 rounded-lg text-center">
                            <p class="text-2xs font-bold text-gray-600 uppercase">Markah Ujian (Tahap 2)</p>
                            <h3 class="text-2xl font-black text-amber-800 mt-1">${assessment.markah ? assessment.markah + '%' : 'N/A'}</h3>
                        </div>
                        ` : `
                        <div class="p-3 bg-emerald-50 border border-emerald-200 rounded-lg text-center flex flex-col justify-center">
                            <p class="text-2xs font-bold text-emerald-800 uppercase">Status Tahap 1</p>
                            <p class="text-xs font-bold text-emerald-700 mt-1">${overallTP >= 3 ? 'Mencapai Tahap Minima' : 'Perlu Bimbingan'}</p>
                        </div>
                        `}
                    </div>

                    <!-- Tandatangan Ruang Cetak -->
                    <div class="grid grid-cols-2 pt-8 text-xs font-semibold text-gray-700 text-center">
                        <div>
                            <p>................................................</p>
                            <p class="mt-1">Tandatangan Guru Subjek</p>
                        </div>
                        <div>
                            <p>................................................</p>
                            <p class="mt-1">Tandatangan Guru Besar / PK</p>
                        </div>
                    </div>
                `;

                reportCard.innerHTML = html;
                container.appendChild(reportCard);
            });
        }

        // Export/Reset Utilities
        function exportDataJSON() {
            const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(appData, null, 2));
            const downloadAnchor = document.createElement('a');
            downloadAnchor.setAttribute("href", dataStr);
            downloadAnchor.setAttribute("download", `sk_bukit_kuchai_pbd_${Date.now()}.json`);
            document.body.appendChild(downloadAnchor);
            downloadAnchor.click();
            downloadAnchor.remove();
        }

        function resetDataToDefault() {
            if (confirm("Adakah anda pasti mahu meriset semua data ke asal? Sila buat salinan dahulu.")) {
                localStorage.removeItem('sk_bukit_kuchai_pbd_data_v2');
                appData = {
                    students: INITIAL_STUDENTS,
                    assessments: {},
                    curriculum: DEFAULT_CURRICULUM
                };
                seedDummyAssessments();
                updateUI();
                showToast("Data telah direset semula.");
            }
        }
    </script>
</body>
</html>
