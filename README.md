<!DOCTYPE html>
<html lang="ms">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dashboard Rekod Perkembangan Murid SK Bukit Kuchai</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        fuchsia: {
                            50: '#fdf4ff',
                            100: '#fae8ff',
                            200: '#f5d0fe',
                            300: '#f0abfc',
                            400: '#e879f9',
                            500: '#d946ef',
                            600: '#c026d3',
                            700: '#a21caf',
                            800: '#86198f',
                            900: '#701a75',
                            950: '#4a044e',
                        }
                    }
                }
            }
        }
    </script>
    <!-- FontAwesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Chart.js -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- PapaParse for CSV parsing -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.4.1/papaparse.min.js"></script>
    
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');
        body {
            font-family: 'Inter', sans-serif;
            background-color: #fcf6fa;
        }

        /* Custom styling for sleek scrollbars */
        ::-webkit-scrollbar {
            width: 6px;
            height: 6px;
        }
        ::-webkit-scrollbar-track {
            background: #fae8ff;
        }
        ::-webkit-scrollbar-thumb {
            background: #d946ef;
            border-radius: 4px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #a21caf;
        }

        /* Print styles for PDF report generation */
        @media print {
            .no-print {
                display: none !important;
            }
            .print-only {
                display: block !important;
            }
            body {
                background-color: white !important;
                color: black !important;
                font-size: 11pt;
            }
            .page-break {
                page-break-before: always;
            }
            .card-shadow {
                box-shadow: none !important;
                border: 1px solid #ccc !important;
            }
        }

        .print-only {
            display: none;
        }

        .gradient-header {
            background: linear-gradient(135deg, #a21caf 0%, #c026d3 50%, #e879f9 100%);
        }

        .tp-btn-selected {
            ring: 2px;
            transform: scale(1.05);
            font-weight: 700;
        }
    </style>
</head>
<body class="min-h-screen text-slate-800 flex flex-col">

    <!-- APP HEADER -->
    <header class="gradient-header text-white shadow-lg no-print sticky top-0 z-40">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-3">
            <div class="flex flex-col md:flex-row items-center justify-between gap-4">
                <div class="flex items-center space-x-3">
                    <div class="w-12 h-12 bg-white/20 backdrop-blur-md rounded-2xl flex items-center justify-center border border-white/30 shadow-inner">
                        <i class="fa-solid fa-graduation-cap text-2xl text-white"></i>
                    </div>
                    <div>
                        <div class="flex items-center gap-2">
                            <span class="bg-white/20 text-xs px-2 py-0.5 rounded-full font-medium tracking-wide uppercase">SK Bukit Kuchai</span>
                            <span id="syncStatusBadge" class="bg-emerald-400/90 text-emerald-950 text-xs px-2 py-0.5 rounded-full font-semibold flex items-center gap-1">
                                <i class="fa-solid fa-arrows-rotate animate-spin text-[10px]"></i> Auto-Sync
                            </span>
                        </div>
                        <h1 class="text-xl md:text-2xl font-bold tracking-tight">Rekod Perkembangan Murid</h1>
                        <p class="text-xs text-fuchsia-100 flex items-center gap-1">
                            <i class="fa-solid fa-user-check"></i> Guru Penyelaras: <span class="font-semibold text-white">Cikgu Aidil Syuhada Jafri</span>
                        </p>
                    </div>
                </div>

                <!-- Top Quick Actions & Google Sheet Status -->
                <div class="flex items-center gap-2 flex-wrap justify-end">
                    <button onclick="refreshGoogleSheetsData()" class="bg-white/10 hover:bg-white/20 text-white text-xs px-3 py-2 rounded-xl transition flex items-center gap-1.5 border border-white/20 shadow-sm" title="Kemaskini data DSKP dari Google Sheets">
                        <i class="fa-solid fa-rotate text-fuchsia-200"></i> Muat Ulang Sheets
                    </button>
                    <button onclick="switchTab('printTab')" class="bg-white text-fuchsia-800 hover:bg-fuchsia-50 font-semibold text-xs px-3 py-2 rounded-xl transition flex items-center gap-1.5 shadow-md">
                        <i class="fa-solid fa-print"></i> Cetak Laporan PDF
                    </button>
                </div>
            </div>

            <!-- Navigation Tabs -->
            <nav class="flex space-x-1 mt-4 overflow-x-auto pb-1 no-scrollbar border-t border-white/20 pt-2">
                <button id="tab-records" onclick="switchTab('recordsTab')" class="tab-btn px-4 py-2 rounded-xl text-xs font-semibold whitespace-nowrap transition flex items-center gap-2 bg-white text-fuchsia-800 shadow">
                    <i class="fa-solid fa-clipboard-list"></i> Perekodan TP & Markah
                </button>
                <button id="tab-students" onclick="switchTab('studentsTab')" class="tab-btn px-4 py-2 rounded-xl text-xs font-semibold whitespace-nowrap transition flex items-center gap-2 hover:bg-white/10 text-white">
                    <i class="fa-solid fa-users"></i> Pengurusan Murid
                </button>
                <button id="tab-analytics" onclick="switchTab('analyticsTab')" class="tab-btn px-4 py-2 rounded-xl text-xs font-semibold whitespace-nowrap transition flex items-center gap-2 hover:bg-white/10 text-white">
                    <i class="fa-solid fa-chart-pie"></i> Analisis & Rumusan TP
                </button>
                <button id="tab-dskp" onclick="switchTab('dskpTab')" class="tab-btn px-4 py-2 rounded-xl text-xs font-semibold whitespace-nowrap transition flex items-center gap-2 hover:bg-white/10 text-white">
                    <i class="fa-solid fa-book-open"></i> Senarai DSKP & Standard
                </button>
                <button id="tab-print" onclick="switchTab('printTab')" class="tab-btn px-4 py-2 rounded-xl text-xs font-semibold whitespace-nowrap transition flex items-center gap-2 hover:bg-white/10 text-white">
                    <i class="fa-solid fa-file-pdf"></i> Format Cetakan (PDF)
                </button>
            </nav>
        </div>
    </header>

    <!-- MAIN CONTENT CONTAINER -->
    <main class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-6 flex-grow w-full">

        <!-- FILTER BAR (Global Selector) -->
        <section class="bg-white rounded-2xl p-4 shadow-sm border border-fuchsia-100 mb-6 no-print">
            <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-4 gap-4">
                <!-- Subjek Dropdown -->
                <div>
                    <label class="block text-xs font-bold text-fuchsia-900 uppercase tracking-wider mb-1">
                        <i class="fa-solid fa-book text-fuchsia-600 mr-1"></i> Subjek
                    </label>
                    <select id="filterSubject" onchange="onFilterChange()" class="w-full bg-fuchsia-50/50 border border-fuchsia-200 text-slate-800 text-sm rounded-xl p-2.5 font-semibold focus:ring-2 focus:ring-fuchsia-500 focus:outline-none transition">
                        <option value="PENDIDIKAN JASMANI (PJ)">PENDIDIKAN JASMANI (PJ)</option>
                        <option value="MATEMATIK">MATEMATIK</option>
                        <option value="PENDIDIKAN KESIHATAN (PK)">PENDIDIKAN KESIHATAN (PK)</option>
                    </select>
                </div>

                <!-- Tahun Dropdown -->
                <div>
                    <label class="block text-xs font-bold text-fuchsia-900 uppercase tracking-wider mb-1">
                        <i class="fa-solid fa-layer-group text-fuchsia-600 mr-1"></i> Tahun
                    </label>
                    <select id="filterYear" onchange="onFilterChange()" class="w-full bg-fuchsia-50/50 border border-fuchsia-200 text-slate-800 text-sm rounded-xl p-2.5 font-semibold focus:ring-2 focus:ring-fuchsia-500 focus:outline-none transition">
                        <option value="TAHUN 1">TAHUN 1</option>
                        <option value="TAHUN 2">TAHUN 2</option>
                        <option value="TAHUN 3">TAHUN 3</option>
                        <option value="TAHUN 4">TAHUN 4 (Tahap 2)</option>
                        <option value="TAHUN 5">TAHUN 5 (Tahap 2)</option>
                        <option value="TAHUN 6">TAHUN 6 (Tahap 2)</option>
                    </select>
                </div>

                <!-- Kelas Dropdown -->
                <div>
                    <label class="block text-xs font-bold text-fuchsia-900 uppercase tracking-wider mb-1">
                        <i class="fa-solid fa-door-open text-fuchsia-600 mr-1"></i> Kelas
                    </label>
                    <select id="filterClass" onchange="onFilterChange()" class="w-full bg-fuchsia-50/50 border border-fuchsia-200 text-slate-800 text-sm rounded-xl p-2.5 font-semibold focus:ring-2 focus:ring-fuchsia-500 focus:outline-none transition">
                        <option value="INOVATIF">INOVATIF</option>
                        <option value="KREATIF">KREATIF</option>
                        <option value="PROAKTIF">PROAKTIF</option>
                    </select>
                </div>

                <!-- Sesi Penilaian View -->
                <div>
                    <label class="block text-xs font-bold text-fuchsia-900 uppercase tracking-wider mb-1">
                        <i class="fa-solid fa-calendar-check text-fuchsia-600 mr-1"></i> Penilaian PBD
                    </label>
                    <select id="filterAssessment" onchange="onFilterChange()" class="w-full bg-fuchsia-50/50 border border-fuchsia-200 text-slate-800 text-sm rounded-xl p-2.5 font-semibold focus:ring-2 focus:ring-fuchsia-500 focus:outline-none transition">
                        <option value="PERTENGAHAN TAHUN">PBD PERTENGAHAN TAHUN</option>
                        <option value="AKHIR TAHUN">PBD AKHIR TAHUN</option>
                    </select>
                </div>
            </div>

            <!-- Context Info Pill -->
            <div id="filterInfoBar" class="mt-3 pt-3 border-t border-fuchsia-100 flex flex-wrap items-center justify-between gap-2 text-xs text-slate-600">
                <div class="flex items-center gap-2">
                    <span class="bg-fuchsia-100 text-fuchsia-800 font-semibold px-2.5 py-1 rounded-lg">
                        <i class="fa-solid fa-circle-info mr-1"></i> <span id="currentSelectionLabel">PENDIDIKAN JASMANI (PJ) - TAHUN 1 INOVATIF</span>
                    </span>
                    <span id="tahapTag" class="bg-slate-100 text-slate-700 px-2.5 py-1 rounded-lg font-medium">Tahap 1 (Tanpa Markah)</span>
                </div>
                <div class="text-right text-fuchsia-700 font-medium" id="studentCountBadge">
                    0 Murid Terdaftar
                </div>
            </div>
        </section>

        <!-- TAB 1: PEREKODAN TP & MARKAH -->
        <div id="recordsTab" class="tab-content block">
            <!-- Topic / SK / SP Selector Card -->
            <div class="bg-white rounded-2xl p-5 shadow-sm border border-fuchsia-100 mb-6">
                <div class="flex items-center justify-between mb-3">
                    <h2 class="text-base font-bold text-fuchsia-950 flex items-center gap-2">
                        <i class="fa-solid fa-sliders text-fuchsia-600"></i> Pilih Kemahiran / Standard Pembelajaran (SP)
                    </h2>
                    <span class="text-xs text-fuchsia-600 font-medium">Data dari Google Sheet</span>
                </div>

                <div class="grid grid-cols-1 md:grid-cols-2 gap-4 bg-fuchsia-50/30 p-3 rounded-xl border border-fuchsia-100">
                    <div>
                        <label class="block text-xs font-semibold text-slate-600 mb-1">Tema / Tajuk & SK:</label>
                        <select id="selectTopic" onchange="renderRecordsTable()" class="w-full bg-white border border-fuchsia-200 text-slate-800 text-xs rounded-xl p-2 font-medium focus:ring-2 focus:ring-fuchsia-500">
                            <!-- Populated dynamically -->
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs font-semibold text-slate-600 mb-1">Standard Pembelajaran (SP) Terpilih:</label>
                        <div id="selectedSpBox" class="bg-white border border-fuchsia-200 rounded-xl p-2 text-xs text-slate-700 font-normal min-h-[38px] flex items-center">
                            Memuatkan SP...
                        </div>
                    </div>
                </div>
            </div>

            <!-- Student TP Scoring Table -->
            <div class="bg-white rounded-2xl shadow-sm border border-fuchsia-100 overflow-hidden">
                <div class="p-4 bg-gradient-to-r from-fuchsia-900 to-fuchsia-800 text-white flex flex-col sm:flex-row items-center justify-between gap-3">
                    <div>
                        <h3 class="font-bold text-sm md:text-base flex items-center gap-2">
                            <i class="fa-solid fa-list-check text-fuchsia-300"></i> Senarai Murid & Penilaian TP1 - TP6
                        </h3>
                        <p class="text-xs text-fuchsia-200">Tekan pada butang Tahap Penguasaan (1-6) untuk menilai setiap murid.</p>
                    </div>
                    <div class="flex items-center gap-2">
                        <button onclick="openAddStudentModal()" class="bg-white text-fuchsia-900 hover:bg-fuchsia-100 font-semibold text-xs px-3 py-1.5 rounded-lg transition flex items-center gap-1.5">
                            <i class="fa-solid fa-user-plus text-fuchsia-600"></i> Tambah Murid
                        </button>
                        <button onclick="saveAllScoresToFirebase()" class="bg-fuchsia-500 hover:bg-fuchsia-400 text-white font-semibold text-xs px-3 py-1.5 rounded-lg transition flex items-center gap-1.5 shadow">
                            <i class="fa-solid fa-floppy-disk"></i> Simpan Rekod
                        </button>
                    </div>
                </div>

                <div class="overflow-x-auto">
                    <table class="w-full text-left border-collapse">
                        <thead>
                            <tr class="bg-fuchsia-50 text-fuchsia-950 text-xs uppercase font-bold border-b border-fuchsia-200">
                                <th class="p-3.5 w-12 text-center">Bil</th>
                                <th class="p-3.5 min-w-[180px]">Nama Murid</th>
                                <th class="p-3.5 min-w-[280px] text-center">
                                    Tahap Penguasaan SP Terpilih
                                    <div class="text-[10px] font-normal text-fuchsia-700 lowercase">(penilaian topik ini)</div>
                                </th>
                                <th class="p-3.5 text-center min-w-[120px]">
                                    Rumusan TP PBD
                                    <div id="assessmentHeaderLabel" class="text-[10px] font-normal text-fuchsia-700 lowercase">Pertengahan Tahun</div>
                                </th>
                                <th id="markahHeader" class="p-3.5 text-center min-w-[110px] hidden">
                                    Markah Ujian (%)
                                    <div class="text-[10px] font-normal text-fuchsia-700 lowercase">Tahap 2 Sahaja</div>
                                </th>
                                <th class="p-3.5 min-w-[180px]">Catatan / Ulasan Guru</th>
                            </tr>
                        </thead>
                        <tbody id="tpTableBody" class="divide-y divide-fuchsia-100 text-sm">
                            <!-- Populated Dynamically -->
                        </tbody>
                    </table>
                </div>

                <!-- Table Footer Legend -->
                <div class="p-4 bg-fuchsia-50/40 border-t border-fuchsia-100 flex flex-wrap items-center justify-between gap-3 text-xs">
                    <div id="tpLegendContainer" class="flex items-center gap-2 text-slate-600 flex-wrap">
                        <!-- Populated dynamically based on PJ or standard subject -->
                    </div>
                    <div class="text-slate-500 font-medium italic">
                        * Penilaian disimpan secara automatik
                    </div>
                </div>
            </div>
        </div>

        <!-- TAB 2: PENGURUSAN MURID -->
        <div id="studentsTab" class="tab-content hidden">
            <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                <!-- Add Student Form Options -->
                <div class="lg:col-span-1 space-y-6">
                    <!-- Individual Add Card -->
                    <div class="bg-white rounded-2xl p-5 shadow-sm border border-fuchsia-100">
                        <h3 class="text-base font-bold text-fuchsia-950 mb-3 flex items-center gap-2">
                            <i class="fa-solid fa-user-plus text-fuchsia-600"></i> Tambah Murid Individu
                        </h3>
                        <form id="individualStudentForm" onsubmit="handleAddIndividualStudent(event)" class="space-y-3">
                            <div>
                                <label class="block text-xs font-semibold text-slate-600 mb-1">Nama Penuh Murid:</label>
                                <input type="text" id="addStudentName" required placeholder="Contoh: MUHAMMAD AMIRUL BIN ROSLI" class="w-full bg-fuchsia-50/40 border border-fuchsia-200 rounded-xl p-2.5 text-xs font-medium focus:ring-2 focus:ring-fuchsia-500 focus:outline-none">
                            </div>
                            <div class="grid grid-cols-2 gap-2">
                                <div>
                                    <label class="block text-xs font-semibold text-slate-600 mb-1">Pilihan Kelas:</label>
                                    <select id="addStudentClass" class="w-full bg-fuchsia-50/40 border border-fuchsia-200 rounded-xl p-2 text-xs font-medium focus:ring-2 focus:ring-fuchsia-500">
                                        <option value="INOVATIF">INOVATIF</option>
                                        <option value="KREATIF">KREATIF</option>
                                        <option value="PROAKTIF">PROAKTIF</option>
                                    </select>
                                </div>
                                <div>
                                    <label class="block text-xs font-semibold text-slate-600 mb-1">Jantina:</label>
                                    <select id="addStudentGender" class="w-full bg-fuchsia-50/40 border border-fuchsia-200 rounded-xl p-2 text-xs font-medium focus:ring-2 focus:ring-fuchsia-500">
                                        <option value="Lelaki">Lelaki</option>
                                        <option value="Perempuan">Perempuan</option>
                                    </select>
                                </div>
                            </div>
                            <button type="submit" class="w-full bg-fuchsia-600 hover:bg-fuchsia-700 text-white font-semibold text-xs py-2.5 rounded-xl transition shadow">
                                <i class="fa-solid fa-plus mr-1"></i> Simpan Murid Baru
                            </button>
                        </form>
                    </div>

                    <!-- Bulk Add Card -->
                    <div class="bg-white rounded-2xl p-5 shadow-sm border border-fuchsia-100">
                        <h3 class="text-base font-bold text-fuchsia-950 mb-2 flex items-center gap-2">
                            <i class="fa-solid fa-users-rectangle text-fuchsia-600"></i> Muat Naik Pukal (Bulk Insert)
                        </h3>
                        <p class="text-xs text-slate-500 mb-3">Tampal senarai nama murid (satu nama setiap baris) dari Excel / Word.</p>
                        <form id="bulkStudentForm" onsubmit="handleBulkAddStudents(event)" class="space-y-3">
                            <div>
                                <textarea id="bulkStudentList" rows="5" required placeholder="Ahmad Bin Ali&#10;Siti Nurhaliza Binti Hassan&#10;Tan Wei Ming&#10;Kavitha A/P Gopal" class="w-full bg-fuchsia-50/40 border border-fuchsia-200 rounded-xl p-2.5 text-xs font-medium focus:ring-2 focus:ring-fuchsia-500 focus:outline-none"></textarea>
                            </div>
                            <button type="submit" class="w-full bg-slate-800 hover:bg-slate-900 text-white font-semibold text-xs py-2.5 rounded-xl transition shadow">
                                <i class="fa-solid fa-file-import mr-1"></i> Tambah Semua Murid Ini
                            </button>
                        </form>
                    </div>
                </div>

                <!-- Existing Student List Card -->
                <div class="lg:col-span-2 bg-white rounded-2xl p-5 shadow-sm border border-fuchsia-100">
                    <div class="flex items-center justify-between mb-4 pb-2 border-b border-fuchsia-100">
                        <div>
                            <h3 class="text-base font-bold text-fuchsia-950">
                                Senarai Murid <span id="currentClassTitle" class="text-fuchsia-600">TAHUN 1 INOVATIF</span>
                            </h3>
                            <p class="text-xs text-slate-500">Urus dan padam maklumat murid dalam kelas ini.</p>
                        </div>
                        <button onclick="clearCurrentClassStudents()" class="text-xs text-rose-600 hover:text-rose-800 font-semibold bg-rose-50 hover:bg-rose-100 px-3 py-1.5 rounded-lg transition border border-rose-200">
                            <i class="fa-solid fa-trash-can mr-1"></i> Kosongkan Kelas
                        </button>
                    </div>

                    <div class="overflow-x-auto max-h-[500px] overflow-y-auto">
                        <table class="w-full text-left border-collapse">
                            <thead class="sticky top-0 bg-fuchsia-50 text-fuchsia-900 text-xs uppercase font-bold">
                                <tr>
                                    <th class="p-3 w-12 text-center">Bil</th>
                                    <th class="p-3">Nama Murid</th>
                                    <th class="p-3">Jantina</th>
                                    <th class="p-3">Pilihan Kelas</th>
                                    <th class="p-3 text-center">Tindakan</th>
                                </tr>
                            </thead>
                            <tbody id="studentManagerTableBody" class="divide-y divide-fuchsia-100 text-xs">
                                <!-- Populated Dynamically -->
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </div>

        <!-- TAB 3: ANALISIS & RUMUSAN TP -->
        <div id="analyticsTab" class="tab-content hidden">
            <!-- KPI Summary Cards -->
            <div class="grid grid-cols-2 md:grid-cols-4 gap-4 mb-6">
                <div class="bg-white rounded-2xl p-4 shadow-sm border border-fuchsia-100 flex items-center gap-3">
                    <div class="w-12 h-12 rounded-xl bg-fuchsia-100 text-fuchsia-700 flex items-center justify-center text-xl font-bold">
                        <i class="fa-solid fa-users"></i>
                    </div>
                    <div>
                        <div class="text-xs text-slate-500 font-medium">Jumlah Murid</div>
                        <div id="kpiTotalStudents" class="text-xl font-extrabold text-fuchsia-950">0</div>
                    </div>
                </div>

                <div class="bg-white rounded-2xl p-4 shadow-sm border border-fuchsia-100 flex items-center gap-3">
                    <div class="w-12 h-12 rounded-xl bg-emerald-100 text-emerald-700 flex items-center justify-center text-xl font-bold">
                        <i class="fa-solid fa-circle-check"></i>
                    </div>
                    <div>
                        <div class="text-xs text-slate-500 font-medium">Menguasai (TP3-6)</div>
                        <div id="kpiPassRate" class="text-xl font-extrabold text-emerald-600">0%</div>
                    </div>
                </div>

                <div class="bg-white rounded-2xl p-4 shadow-sm border border-fuchsia-100 flex items-center gap-3">
                    <div class="w-12 h-12 rounded-xl bg-fuchsia-100 text-fuchsia-800 flex items-center justify-center text-xl font-bold">
                        <i class="fa-solid fa-star"></i>
                    </div>
                    <div>
                        <div class="text-xs text-slate-500 font-medium">Cemerlang / Mithali (TP5-6)</div>
                        <div id="kpiExcellenceRate" class="text-xl font-extrabold text-fuchsia-700">0%</div>
                    </div>
                </div>

                <div class="bg-white rounded-2xl p-4 shadow-sm border border-fuchsia-100 flex items-center gap-3">
                    <div class="w-12 h-12 rounded-xl bg-purple-100 text-purple-700 flex items-center justify-center text-xl font-bold">
                        <i class="fa-solid fa-chart-line"></i>
                    </div>
                    <div>
                        <div class="text-xs text-slate-500 font-medium">Purata TP Kelas</div>
                        <div id="kpiAvgTp" class="text-xl font-extrabold text-purple-900">0.0</div>
                    </div>
                </div>
            </div>

            <!-- Charts Section -->
            <div class="grid grid-cols-1 lg:grid-cols-2 gap-6 mb-6">
                <!-- Bar Chart TP Distribution -->
                <div class="bg-white rounded-2xl p-5 shadow-sm border border-fuchsia-100">
                    <h3 class="text-sm font-bold text-fuchsia-950 mb-3 flex items-center justify-between">
                        <span><i class="fa-solid fa-chart-column text-fuchsia-600 mr-2"></i> Taburan Tahap Penguasaan (TP1 - TP6)</span>
                        <span id="chartFilterBadge" class="text-xs bg-fuchsia-100 text-fuchsia-800 px-2.5 py-0.5 rounded-full font-semibold">Semua Subjek</span>
                    </h3>
                    <div class="relative h-64">
                        <canvas id="tpBarChart"></canvas>
                    </div>
                </div>

                <!-- Doughnut Chart Percentage -->
                <div class="bg-white rounded-2xl p-5 shadow-sm border border-fuchsia-100">
                    <h3 class="text-sm font-bold text-fuchsia-950 mb-3 flex items-center gap-2">
                        <i class="fa-solid fa-chart-pie text-fuchsia-600"></i> Peratus Penguasaan Murid
                    </h3>
                    <div class="relative h-64">
                        <canvas id="tpPieChart"></canvas>
                    </div>
                </div>
            </div>

            <!-- Detailed Breakdown Table -->
            <div class="bg-white rounded-2xl p-5 shadow-sm border border-fuchsia-100">
                <div class="flex items-center justify-between mb-3">
                    <h3 class="text-sm font-bold text-fuchsia-950 flex items-center gap-2">
                        <i class="fa-solid fa-table text-fuchsia-600"></i> Rumusan Analisis Menguasai Mengikut TP
                    </h3>
                    <span id="analyticsSubjectBadge" class="text-xs font-semibold px-2.5 py-1 bg-fuchsia-100 text-fuchsia-800 rounded-lg">Rujukan TP: PJ</span>
                </div>
                <div class="overflow-x-auto">
                    <table class="w-full text-xs text-left border-collapse">
                        <thead>
                            <tr class="bg-fuchsia-50 text-fuchsia-950 font-bold border-b border-fuchsia-200 uppercase">
                                <th class="p-3">Tahap Penguasaan (TP)</th>
                                <th class="p-3">Tafsiran Kualitatif DSKP</th>
                                <th class="p-3 text-center">Bilangan Murid</th>
                                <th class="p-3 text-center">Peratus (%)</th>
                                <th class="p-3">Status Penguasaan</th>
                            </tr>
                        </thead>
                        <tbody id="analyticsTableBody" class="divide-y divide-fuchsia-100">
                            <!-- Dynamically generated rows -->
                        </tbody>
                    </table>
                </div>
            </div>
        </div>

        <!-- TAB 4: SENARAI DSKP -->
        <div id="dskpTab" class="tab-content hidden">
            <div class="bg-white rounded-2xl p-5 shadow-sm border border-fuchsia-100">
                <div class="flex flex-col md:flex-row md:items-center justify-between gap-3 mb-4 pb-3 border-b border-fuchsia-100">
                    <div>
                        <h3 class="text-base font-bold text-fuchsia-950 flex items-center gap-2">
                            <i class="fa-solid fa-book text-fuchsia-600"></i> Senarai Tema, Tajuk, SK & SP DSKP
                        </h3>
                        <p class="text-xs text-slate-500">Data ini diambil secara live dari Google Sheet rasmi Cikgu Aidil Syuhada Jafri.</p>
                    </div>
                    <div class="w-full md:w-64">
                        <input type="text" id="searchDskp" onkeyup="filterDskpTable()" placeholder="Cari Tema, SK atau SP..." class="w-full bg-fuchsia-50/40 border border-fuchsia-200 rounded-xl p-2 text-xs font-medium focus:ring-2 focus:ring-fuchsia-500">
                    </div>
                </div>

                <div class="overflow-x-auto max-h-[600px] overflow-y-auto">
                    <table class="w-full text-xs text-left border-collapse">
                        <thead class="sticky top-0 bg-fuchsia-900 text-white uppercase font-bold">
                            <tr>
                                <th class="p-3 min-w-[100px]">Subjek</th>
                                <th class="p-3 min-w-[90px]">Tahun</th>
                                <th class="p-3 min-w-[150px]">Tema</th>
                                <th class="p-3 min-w-[150px]">Tajuk</th>
                                <th class="p-3 min-w-[200px]">Standard Kandungan (SK)</th>
                                <th class="p-3 min-w-[250px]">Standard Pembelajaran (SP)</th>
                            </tr>
                        </thead>
                        <tbody id="dskpTableBody" class="divide-y divide-fuchsia-100">
                            <!-- Dynamically populated from Google Sheet CSV -->
                        </tbody>
                    </table>
                </div>
            </div>
        </div>

        <!-- TAB 5: FORMAT CETAKAN LAPORAN (PDF) -->
        <div id="printTab" class="tab-content hidden">
            <!-- Print Control Panel (Hidden during printing) -->
            <div class="bg-fuchsia-900 text-white rounded-2xl p-4 shadow-sm mb-6 flex flex-col md:flex-row items-center justify-between gap-4 no-print">
                <div>
                    <h3 class="font-bold text-base flex items-center gap-2">
                        <i class="fa-solid fa-print text-fuchsia-300"></i> Pratinjau Laporan Perkembangan Murid
                    </h3>
                    <p class="text-xs text-fuchsia-200">Sedia untuk dicetak atau disimpan sebagai dokumen PDF.</p>
                </div>
                <div class="flex items-center gap-2">
                    <button onclick="window.print()" class="bg-white text-fuchsia-950 font-bold text-xs px-4 py-2.5 rounded-xl hover:bg-fuchsia-100 transition shadow flex items-center gap-2">
                        <i class="fa-solid fa-print text-fuchsia-600"></i> Cetak / Simpan PDF
                    </button>
                </div>
            </div>

            <!-- PRINTABLE A4 REPORT CONTAINER -->
            <div id="printableReport" class="bg-white rounded-2xl p-8 border border-slate-200 shadow-md max-w-4xl mx-auto print:max-w-none print:shadow-none print:p-0 print:border-none">
                <!-- Official School Header -->
                <div class="border-b-2 border-slate-900 pb-4 mb-6 flex items-center justify-between">
                    <div class="flex items-center gap-4">
                        <div class="w-16 h-16 bg-fuchsia-800 text-white rounded-2xl flex items-center justify-center font-bold text-2xl border-2 border-fuchsia-600">
                            SKBK
                        </div>
                        <div>
                            <h1 class="text-xl font-black text-slate-900 uppercase tracking-tight">SEKOLAH KEBANGSAAN BUKIT KUCHAI</h1>
                            <p class="text-xs text-slate-700 font-semibold">REKOD PERKEMBANGAN & PENTAKSIRAN BILIK DARJAH (PBD)</p>
                            <p class="text-[11px] text-slate-500">Jalan 17, Taman Bukit Kuchai, 47100 Puchong, Selangor</p>
                        </div>
                    </div>
                    <div class="text-right text-xs text-slate-600">
                        <div class="font-bold text-fuchsia-900">SULIT</div>
                        <div id="printDateStamp" class="text-[10px] text-slate-500">Tarikh: --/--/----</div>
                    </div>
                </div>

                <!-- Metadata Information Block -->
                <div class="bg-fuchsia-50/50 p-4 rounded-xl border border-fuchsia-200 grid grid-cols-2 md:grid-cols-4 gap-3 text-xs mb-6">
                    <div>
                        <span class="text-slate-500 block">SUBJEK:</span>
                        <strong id="pdfSubject" class="text-slate-900 uppercase">PENDIDIKAN JASMANI (PJ)</strong>
                    </div>
                    <div>
                        <span class="text-slate-500 block">TAHUN / KELAS:</span>
                        <strong id="pdfClass" class="text-slate-900 uppercase">TAHUN 1 INOVATIF</strong>
                    </div>
                    <div>
                        <span class="text-slate-500 block">PENILAIAN:</span>
                        <strong id="pdfAssessment" class="text-slate-900 uppercase">PERTENGAHAN TAHUN</strong>
                    </div>
                    <div>
                        <span class="text-slate-500 block">GURU MATA PELAJARAN:</span>
                        <strong class="text-slate-900">CIKGU AIDIL SYUHADA JAFRI</strong>
                    </div>
                </div>

                <!-- Printable TP Summary Table -->
                <h3 class="text-xs font-bold uppercase text-slate-800 tracking-wider mb-2 flex items-center justify-between">
                    <span>Rumusan Pentaksiran Murid</span>
                    <span id="pdfStudentCount" class="text-[11px] font-normal text-slate-500">0 Murid</span>
                </h3>
                <table class="w-full text-xs border-collapse border border-slate-300 mb-6">
                    <thead>
                        <tr class="bg-slate-100 text-slate-800 font-bold border-b border-slate-300">
                            <th class="border border-slate-300 p-2 text-center w-8">Bil</th>
                            <th class="border border-slate-300 p-2 text-left">Nama Murid</th>
                            <th class="border border-slate-300 p-2 text-center w-28">Rumusan TP</th>
                            <th id="pdfMarkahHeader" class="border border-slate-300 p-2 text-center w-24 hidden">Markah (%)</th>
                            <th class="border border-slate-300 p-2 text-left">Tahap Penguasaan / Ulasan</th>
                        </tr>
                    </thead>
                    <tbody id="pdfTableBody" class="divide-y divide-slate-200">
                        <!-- Populated dynamically -->
                    </tbody>
                </table>

                <!-- Printable TP Breakdown Summary -->
                <div class="grid grid-cols-2 gap-4 text-xs mb-8">
                    <div class="border border-slate-300 rounded-xl p-3 bg-slate-50">
                        <div class="font-bold text-slate-800 border-b border-slate-200 pb-1 mb-2">Ringkasan Taburan TP & Tafsiran</div>
                        <div id="pdfTpBreakdownText" class="space-y-1 text-slate-700">
                            <!-- Populated dynamically -->
                        </div>
                    </div>
                    <div class="border border-slate-300 rounded-xl p-3 bg-slate-50 flex flex-col justify-between">
                        <div>
                            <div class="font-bold text-slate-800 border-b border-slate-200 pb-1 mb-1">Status Penguasaan Keseluruhan</div>
                            <p class="text-slate-600 text-[11px]">Murid Capai Sekurang-kurangnya TP3: <strong id="pdfPassCount">0 (0%)</strong></p>
                        </div>
                        <div class="text-[10px] text-slate-500 italic">
                            Dokumen ini dijana secara automatik melalui Dashboard Rekod Perkembangan Murid SK Bukit Kuchai.
                        </div>
                    </div>
                </div>

                <!-- Signatures Footer -->
                <div class="grid grid-cols-2 gap-12 pt-8 border-t border-slate-300 text-xs mt-12 page-break-inside-avoid">
                    <div class="text-center">
                        <p class="text-slate-500 mb-12">Disediakan Oleh:</p>
                        <div class="border-b border-slate-800 w-48 mx-auto mb-1"></div>
                        <p class="font-bold text-slate-900 uppercase">CIKGU AIDIL SYUHADA JAFRI</p>
                        <p class="text-[10px] text-slate-600">Guru Mata Pelajaran SK Bukit Kuchai</p>
                    </div>
                    <div class="text-center">
                        <p class="text-slate-500 mb-12">Disahkan Oleh:</p>
                        <div class="border-b border-slate-800 w-48 mx-auto mb-1"></div>
                        <p class="font-bold text-slate-900 uppercase">GURU BESAR / PK PENTAKSIRAN</p>
                        <p class="text-[10px] text-slate-600">SK Bukit Kuchai</p>
                    </div>
                </div>
            </div>
        </div>

    </main>

    <!-- TOAST NOTIFICATION CONTAINER -->
    <div id="toastContainer" class="fixed bottom-5 right-5 z-50 flex flex-col gap-2 no-print"></div>

    <!-- ADD STUDENT MODAL -->
    <div id="addStudentModal" class="fixed inset-0 bg-slate-900/50 backdrop-blur-sm z-50 flex items-center justify-center hidden no-print">
        <div class="bg-white rounded-2xl max-w-md w-full mx-4 shadow-2xl border border-fuchsia-100 overflow-hidden transform transition-all">
            <div class="bg-gradient-to-r from-fuchsia-900 to-fuchsia-800 p-4 text-white flex items-center justify-between">
                <h3 class="font-bold text-sm flex items-center gap-2">
                    <i class="fa-solid fa-user-plus text-fuchsia-300"></i> Tambah Murid Baru
                </h3>
                <button onclick="closeAddStudentModal()" class="text-fuchsia-200 hover:text-white text-lg">
                    <i class="fa-solid fa-xmark"></i>
                </button>
            </div>
            <form id="modalAddStudentForm" onsubmit="handleModalAddStudent(event)" class="p-5 space-y-4">
                <div>
                    <label class="block text-xs font-semibold text-slate-700 mb-1">Nama Penuh Murid:</label>
                    <input type="text" id="modalStudentName" required placeholder="Contoh: MUHAMMAD AMIRUL BIN ROSLI" class="w-full bg-fuchsia-50/40 border border-fuchsia-200 rounded-xl p-2.5 text-xs font-medium focus:ring-2 focus:ring-fuchsia-500 focus:outline-none">
                </div>
                <div class="grid grid-cols-2 gap-3">
                    <div>
                        <label class="block text-xs font-semibold text-slate-700 mb-1">Kelas:</label>
                        <select id="modalStudentClass" class="w-full bg-fuchsia-50/40 border border-fuchsia-200 rounded-xl p-2.5 text-xs font-medium focus:ring-2 focus:ring-fuchsia-500">
                            <option value="INOVATIF">INOVATIF</option>
                            <option value="KREATIF">KREATIF</option>
                            <option value="PROAKTIF">PROAKTIF</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs font-semibold text-slate-700 mb-1">Jantina:</label>
                        <select id="modalStudentGender" class="w-full bg-fuchsia-50/40 border border-fuchsia-200 rounded-xl p-2.5 text-xs font-medium focus:ring-2 focus:ring-fuchsia-500">
                            <option value="Lelaki">Lelaki</option>
                            <option value="Perempuan">Perempuan</option>
                        </select>
                    </div>
                </div>
                <div class="flex items-center justify-end gap-2 pt-2 border-t border-fuchsia-100">
                    <button type="button" onclick="closeAddStudentModal()" class="px-4 py-2 rounded-xl text-xs font-semibold text-slate-600 hover:bg-slate-100 transition">
                        Batal
                    </button>
                    <button type="submit" class="px-4 py-2 rounded-xl text-xs font-semibold bg-fuchsia-600 hover:bg-fuchsia-700 text-white shadow transition">
                        Simpan Murid
                    </button>
                </div>
            </form>
        </div>
    </div>

    <!-- FOOTER -->
    <footer class="bg-white border-t border-fuchsia-100 py-4 mt-8 no-print">
        <div class="max-w-7xl mx-auto px-4 text-center text-xs text-slate-500">
            <p>© 2026 <strong>Rekod Perkembangan Murid SK Bukit Kuchai</strong>. Direka khas untuk <strong>Cikgu Aidil Syuhada Jafri</strong>.</p>
        </div>
    </footer>

    <script>
        // Google Sheet CSV URL provided by user
        const GOOGLE_SHEET_CSV_URL = "https://docs.google.com/spreadsheets/d/e/2PACX-1vRxDf3bKsKRdGOdPvyEas8m3M0b7gR9wqQ-d_Duczf3hc7IuGXc-INuN8pRy9O3oyLrrAhIntul_Spz/pub?gid=531444998&single=true&output=csv";

        // Global State
        let dskpData = []; // Standard DSKP items fetched from CSV
        let studentsList = {}; // Keyed by `${tahun}_${kelas}` -> array of student objects
        let tpRecords = {}; // Keyed by `${studentId}_${subject}_${topicId}` -> TP value
        let summaryRecords = {}; // Keyed by `${studentId}_${subject}_${assessment}` -> { overallTp, markah, ulasan }
        let currentTab = 'recordsTab';
        let tpChart = null;
        let pieChart = null;

        // PJ Specific TP Descriptor Map & Fallback Standard Descriptors
        function getTpDescriptors(subject) {
            const cleanSub = (subject || '').toUpperCase();
            if (cleanSub.includes("PENDIDIKAN JASMANI") || cleanSub.includes("PJ")) {
                return {
                    1: "Meniru atau Melakukan Perkara Asas",
                    2: "Memahami Perkara Asas",
                    3: "Mengaplikasi Kemahiran",
                    4: "Melakukan dengan Beradab / Sistematik",
                    5: "Melakukan dengan Beradab Terpuji",
                    6: "Melakukan dengan Beradab Mithali"
                };
            }
            // Default descriptors for other subjects like Matematik / PK
            return {
                1: "Sangat Terhad",
                2: "Terhad",
                3: "Memuaskan",
                4: "Baik",
                5: "Sangat Baik",
                6: "Cemerlang"
            };
        }

        // Comprehensive fallback DSKP dataset matching the teacher's spreadsheet structure
        const fallbackDskp = [
            // PENDIDIKAN JASMANI TAHUN 1 (From user spreadsheet)
            {
                subjek: "PENDIDIKAN JASMANI",
                tahun: "TAHUN 1",
                tema: "Kemahiran : Konsep Pergerakan",
                tajuk: "1. Kemahiran Pergerakan ( Domain Psikomotor )",
                sk: "1.1 Meneroka pelbagai corak pergerakan berdasarkan konsep pergerakan.",
                sp: "1.1.1 Melakukan pergerakan yang melibatkan kesedaran tubuh"
            },
            {
                subjek: "PENDIDIKAN JASMANI",
                tahun: "TAHUN 1",
                tema: "Kemahiran : Konsep Pergerakan",
                tajuk: "2. Aplikasi Pengetahuan Dalam Pergerakan ( Domain Kognitif )",
                sk: "2.1 Menggunakan pengetahuan konsep pergerakan semasa meneroka pelbagai corak pergerakan.",
                sp: "1.1.2 Melakukan pergerakan yang melibatkan kesedaran ruang diri, ruang am dan batasan ruang"
            },
            {
                subjek: "PENDIDIKAN JASMANI",
                tahun: "TAHUN 1",
                tema: "Kemahiran : Konsep Pergerakan",
                tajuk: "5. Kesukanan ( Domain Afektif )",
                sk: "5.1 Mematuhi dan mengamalkan elemen keselamatan.",
                sp: "1.1.3 Melakukan pergerakan yang melibatkan kesedaran arah"
            },
            {
                subjek: "PENDIDIKAN JASMANI",
                tahun: "TAHUN 1",
                tema: "Kemahiran : Pergerakan Asas - Kemahiran Lokomotor Dan Bukan Lokomotor",
                tajuk: "1. Kemahiran Pergerakan ( Domain Psikomotor )",
                sk: "1.2 Melakukan pelbagai pergerakan lokomotor",
                sp: "1.1.4 Menukar arah dari hadapan ke belakang dan sebaliknya"
            },
            {
                subjek: "PENDIDIKAN JASMANI",
                tahun: "TAHUN 1",
                tema: "Kemahiran : Pergerakan Asas - Kemahiran Lokomotor Dan Bukan Lokomotor",
                tajuk: "1. Kemahiran Pergerakan ( Domain Psikomotor )",
                sk: "1.3 Melakukan pelbagai pergerakan bukan lokomotor",
                sp: "1.1.5 Melakukan pergerakan dalam laluan lurus, melengkung dan zig-zag"
            },
            {
                subjek: "PENDIDIKAN JASMANI",
                tahun: "TAHUN 1",
                tema: "Kemahiran : Pergerakan Asas - Kemahiran Lokomotor Dan Bukan Lokomotor",
                tajuk: "2. Aplikasi Pengetahuan Dalam Pergerakan ( Domain Kognitif )",
                sk: "2.2 Menggunakan pengetahuan konsep pergerakan dan prinsip mekanik dalam pergerakan lokomotor dan bukan lokomotor.",
                sp: "1.1.6 Melakukan pergerakan yang berbeza kelajuan berdasarkan tempo, irama dan isyarat."
            },
            {
                subjek: "PENDIDIKAN JASMANI",
                tahun: "TAHUN 1",
                tema: "Kemahiran : Pergerakan Asas - Kemahiran Lokomotor Dan Bukan Lokomotor",
                tajuk: "5. Kesukanan ( Domain Afektif )",
                sk: "5.1 Mematuhi dan mengamalkan elemen keselamatan",
                sp: "1.1.7 Melakukan pergerakan yang berbeza kelajuan"
            },
            {
                subjek: "PENDIDIKAN JASMANI",
                tahun: "TAHUN 1",
                tema: "Kemahiran : Pergerakan Asas - Kemahiran Lokomotor Dan Bukan Lokomotor",
                tajuk: "5. Kesukanan ( Domain Afektif )",
                sk: "5.2 Menunjukkan keyakinan dan tanggungjawab kendiri semasa melakukan aktiviti",
                sp: "2.1.1 Menyatakan bentuk badan semasa melakukan pergerakan"
            },
            {
                subjek: "PENDIDIKAN JASMANI",
                tahun: "TAHUN 1",
                tema: "Kemahiran : Pergerakan Asas - Kemahiran Lokomotor Dan Bukan Lokomotor",
                tajuk: "5. Kesukanan ( Domain Afektif )",
                sk: "5.3 Berkomunikasi dalam pelbagai cara semasa melakukan aktiviti",
                sp: "2.1.2 Mengenal pasti ruang diri"
            },
            {
                subjek: "PENDIDIKAN JASMANI",
                tahun: "TAHUN 1",
                tema: "Kemahiran : Pergerakan Asas - Kemahiran Lokomotor Dan Bukan Lokomotor",
                tajuk: "5. Kesukanan ( Domain Afektif )",
                sk: "5.4 Membentuk kumpulan dan bekerjasama dalam kumpulan",
                sp: "2.1.3 Mengenal pasti ruang am"
            },
            {
                subjek: "PENDIDIKAN JASMANI",
                tahun: "TAHUN 1",
                tema: "Kemahiran : Pergerakan Asas - Kemahiran Lokomotor Dan Bukan Lokomotor",
                tajuk: "1. Kemahiran Pergerakan ( Domain Psikomotor )",
                sk: "1.4 Melakukan pelbagai kemahiran manipulasi.",
                sp: "2.1.4 Mengenal pasti arah pergerakan"
            },
            {
                subjek: "PENDIDIKAN JASMANI",
                tahun: "TAHUN 1",
                tema: "Kemahiran : Pergerakan Asas - Kemahiran Lokomotor Dan Bukan Lokomotor",
                tajuk: "2. Aplikasi Pengetahuan Dalam Pergerakan ( Domain Kognitif )",
                sk: "2.3 Menggunakan pengetahuan konsep pergerakan dan prinsip mekanik dalam manipulasi alatan.",
                sp: "2.1.5 Menyatakan laluan pergerakan"
            },
            // MATEMATIK TAHUN 1
            {
                subjek: "MATEMATIK",
                tahun: "TAHUN 1",
                tema: "NOMBOR DAN OPERASI",
                tajuk: "Nombor Hingga 100",
                sk: "1.1 Kuantiti secara intuitif",
                sp: "1.1.1 Menyatakan kuantiti secara intuitif melalui perbandingan"
            },
            {
                subjek: "MATEMATIK",
                tahun: "TAHUN 1",
                tema: "NOMBOR DAN OPERASI",
                tajuk: "Nombor Hingga 100",
                sk: "1.2 Nilai Nombor",
                sp: "1.2.1 Menamakan nombor hingga 100"
            },
            // PENDIDIKAN KESIHATAN TAHUN 1
            {
                subjek: "PENDIDIKAN KESIHATAN",
                tahun: "TAHUN 1",
                tema: "KESIHATAN DIRI DAN REPRODUKTIF",
                tajuk: "Menjaga Kesihatan Diri",
                sk: "1.1 Menjaga kebersihan anggota tubuh",
                sp: "1.1.1 Mengetahui anggota tubuh lelaki dan perempuan"
            }
        ];

        // Sample initial students for instant interactivity
        const defaultInitialStudents = {
            "TAHUN 1_INOVATIF": [
                { id: "M001", name: "ADAM HARIS BIN AZMAN", gender: "Lelaki", kelas: "INOVATIF" },
                { id: "M002", name: "AISYAH HUMAIRA BINTI KHALID", gender: "Perempuan", kelas: "INOVATIF" },
                { id: "M003", name: "DANISH FIRDAUS BIN ZULKIFLI", gender: "Lelaki", kelas: "INOVATIF" },
                { id: "M004", name: "NUR IMAN BINTI AHMAD", gender: "Perempuan", kelas: "INOVATIF" },
                { id: "M005", name: "MUHAMMAD HAZIQ BIN ZAKARIA", gender: "Lelaki", kelas: "INOVATIF" }
            ],
            "TAHUN 4_INOVATIF": [
                { id: "M401", name: "AMIRUL ASYRAF BIN AIDIL", gender: "Lelaki", kelas: "INOVATIF" },
                { id: "M402", name: "NUR BALQIS BINTI SYUHADA", gender: "Perempuan", kelas: "INOVATIF" }
            ]
        };

        window.onload = function() {
            // Load local storage initial cache if exists
            loadLocalCache();
            
            // Set print date
            const today = new Date().toLocaleDateString('ms-MY', { day: '2-digit', month: '2-digit', year: 'numeric' });
            document.getElementById('printDateStamp').innerText = 'Tarikh: ' + today;

            // Fetch DSKP CSV from Google Sheets
            fetchGoogleSheetsCSV();

            // Initialize views
            onFilterChange();
        };

        // Utility to normalize year strings e.g. "1" -> "TAHUN 1"
        function normalizeTahun(val) {
            if (!val) return "TAHUN 1";
            const str = val.toString().trim().toUpperCase();
            if (str === "1" || str === "TAHUN 1") return "TAHUN 1";
            if (str === "2" || str === "TAHUN 2") return "TAHUN 2";
            if (str === "3" || str === "TAHUN 3") return "TAHUN 3";
            if (str === "4" || str === "TAHUN 4") return "TAHUN 4";
            if (str === "5" || str === "TAHUN 5") return "TAHUN 5";
            if (str === "6" || str === "TAHUN 6") return "TAHUN 6";
            return str.startsWith("TAHUN") ? str : `TAHUN ${str}`;
        }

        // Subject matching helper to ignore brackets like (PJ) or (PK)
        function matchesSubject(dskpSub, filterSub) {
            if (!dskpSub || !filterSub) return false;
            const cleanDskp = dskpSub.toUpperCase().replace(/\s*\([^)]*\)/g, '').trim();
            const cleanFilter = filterSub.toUpperCase().replace(/\s*\([^)]*\)/g, '').trim();
            return cleanDskp.includes(cleanFilter) || cleanFilter.includes(cleanDskp);
        }

        // Year matching helper
        function matchesYear(dskpYr, filterYr) {
            if (!dskpYr || !filterYr) return false;
            const normDskp = normalizeTahun(dskpYr);
            const normFilter = normalizeTahun(filterYr);
            return normDskp === normFilter || dskpYr.toString().includes(filterYr.toString().replace('TAHUN ', '').trim());
        }

        function fetchGoogleSheetsCSV() {
            showToast('Sedang memuatkan DSKP dari Google Sheets...', 'info');
            
            Papa.parse(GOOGLE_SHEET_CSV_URL, {
                download: true,
                header: true,
                skipEmptyLines: true,
                complete: function(results) {
                    if (results.data && results.data.length > 0) {
                        // Forward-fill variables to handle merged cells in Google Sheets
                        let lastTema = "";
                        let lastTajuk = "";
                        let lastSubjek = "PENDIDIKAN JASMANI";
                        let lastTahun = "TAHUN 1";

                        dskpData = [];

                        results.data.forEach(row => {
                            // Extract raw values
                            const rawSubjek = (row['SUBJEK'] || row['subjek'] || '').trim();
                            const rawTahun = (row['TAHUN'] || row['tahun'] || '').trim();
                            const rawTema = (row['TEMA'] || row['tema'] || '').trim();
                            const rawTajuk = (row['TAJUK'] || row['tajuk'] || '').trim();
                            const rawSk = (row['STANDARD KANDUNGAN (SK)'] || row['SK'] || row['sk'] || '').trim();
                            const rawSp = (row['STANDARD PEMBELAJARAN (SP)'] || row['SP'] || row['sp'] || '').trim();

                            // Forward fill logic for merged sheet cells
                            if (rawSubjek) lastSubjek = rawSubjek.toUpperCase();
                            if (rawTahun) lastTahun = normalizeTahun(rawTahun);
                            if (rawTema) lastTema = rawTema;
                            if (rawTajuk) lastTajuk = rawTajuk;

                            if (rawSk || rawSp) {
                                dskpData.push({
                                    subjek: lastSubjek,
                                    tahun: lastTahun,
                                    tema: lastTema || '-',
                                    tajuk: lastTajuk || '-',
                                    sk: rawSk || '-',
                                    sp: rawSp || '-'
                                });
                            }
                        });

                        if (dskpData.length === 0) {
                            dskpData = fallbackDskp;
                        }
                        showToast(`Berjaya memuatkan ${dskpData.length} item DSKP!`, 'success');
                    } else {
                        dskpData = fallbackDskp;
                        showToast('Menggunakan data DSKP asas secara lalai.', 'info');
                    }
                    populateDskpTable();
                    populateTopicDropdown();
                    renderRecordsTable();
                },
                error: function(err) {
                    console.error("CSV Fetch Error:", err);
                    dskpData = fallbackDskp;
                    showToast('Ralat sambungan CSV Google Sheet. Menggunakan DSKP sampel.', 'warning');
                    populateDskpTable();
                    populateTopicDropdown();
                    renderRecordsTable();
                }
            });
        }

        function refreshGoogleSheetsData() {
            fetchGoogleSheetsCSV();
        }

        function switchTab(tabId) {
            currentTab = tabId;
            document.querySelectorAll('.tab-content').forEach(el => el.classList.add('hidden'));
            document.getElementById(tabId).classList.remove('hidden');

            document.querySelectorAll('.tab-btn').forEach(btn => {
                btn.classList.remove('bg-white', 'text-fuchsia-800', 'shadow');
                btn.classList.add('hover:bg-white/10', 'text-white');
            });

            const activeBtn = document.getElementById('tab-' + tabId.replace('Tab', ''));
            if (activeBtn) {
                activeBtn.classList.add('bg-white', 'text-fuchsia-800', 'shadow');
                activeBtn.classList.remove('hover:bg-white/10', 'text-white');
            }

            if (tabId === 'analyticsTab') {
                renderAnalyticsCharts();
            } else if (tabId === 'printTab') {
                renderPrintReport();
            } else if (tabId === 'studentsTab') {
                renderStudentManagerTable();
            }
        }

        function updateTpLegend() {
            const subject = document.getElementById('filterSubject').value;
            const desc = getTpDescriptors(subject);
            const container = document.getElementById('tpLegendContainer');
            if (!container) return;

            const isPj = subject.includes('PJ') || subject.includes('PENDIDIKAN JASMANI');

            container.innerHTML = `
                <span class="font-bold text-fuchsia-900 flex items-center gap-1">
                    Petunjuk TP ${isPj ? '<span class="bg-fuchsia-200 text-fuchsia-900 text-[10px] px-1.5 py-0.5 rounded">Rujukan DSKP PJ</span>' : ''}:
                </span>
                <span class="px-2 py-0.5 rounded bg-rose-100 text-rose-800 font-semibold" title="${desc[1]}">TP1: ${desc[1]}</span>
                <span class="px-2 py-0.5 rounded bg-orange-100 text-orange-800 font-semibold" title="${desc[2]}">TP2: ${desc[2]}</span>
                <span class="px-2 py-0.5 rounded bg-amber-100 text-amber-800 font-semibold" title="${desc[3]}">TP3: ${desc[3]}</span>
                <span class="px-2 py-0.5 rounded bg-emerald-100 text-emerald-800 font-semibold" title="${desc[4]}">TP4: ${desc[4]}</span>
                <span class="px-2 py-0.5 rounded bg-teal-100 text-teal-800 font-semibold" title="${desc[5]}">TP5: ${desc[5]}</span>
                <span class="px-2 py-0.5 rounded bg-fuchsia-100 text-fuchsia-800 font-semibold" title="${desc[6]}">TP6: ${desc[6]}</span>
            `;
        }

        function onFilterChange() {
            const subject = document.getElementById('filterSubject').value;
            const year = document.getElementById('filterYear').value;
            const className = document.getElementById('filterClass').value;
            const assessment = document.getElementById('filterAssessment').value;

            // Check if Tahap 2 (Tahun 4, 5, 6)
            const isTahap2 = year.includes('TAHUN 4') || year.includes('TAHUN 5') || year.includes('TAHUN 6');

            // Header UI updates
            document.getElementById('currentSelectionLabel').innerText = `${subject} - ${year} ${className}`;
            document.getElementById('tahapTag').innerText = isTahap2 ? 'Tahap 2 (Dengan Markah %)' : 'Tahap 1 (TP Sahaja)';
            document.getElementById('assessmentHeaderLabel').innerText = assessment;
            document.getElementById('currentClassTitle').innerText = `${year} ${className}`;

            // Markah column visibility toggle
            const markahCol = document.getElementById('markahHeader');
            if (isTahap2) {
                markahCol.classList.remove('hidden');
            } else {
                markahCol.classList.add('hidden');
            }

            // Update TP Legend Footer based on subject
            updateTpLegend();

            // Repopulate topics based on new subject/year
            populateTopicDropdown();
            renderRecordsTable();
            renderStudentManagerTable();

            if (currentTab === 'analyticsTab') {
                renderAnalyticsCharts();
            } else if (currentTab === 'printTab') {
                renderPrintReport();
            }
        }

        function populateTopicDropdown() {
            const subject = document.getElementById('filterSubject').value;
            const year = document.getElementById('filterYear').value;
            const selectTopic = document.getElementById('selectTopic');

            selectTopic.innerHTML = '';

            // Filter DSKP using matchesSubject & matchesYear helpers
            let filtered = dskpData.filter(item => {
                return matchesSubject(item.subjek, subject) && matchesYear(item.tahun, year);
            });

            if (filtered.length === 0) {
                // Fallback to match subject only if year filter has no items
                filtered = dskpData.filter(item => matchesSubject(item.subjek, subject));
            }

            if (filtered.length === 0) {
                filtered = fallbackDskp.filter(item => matchesSubject(item.subjek, subject));
            }

            if (filtered.length === 0) {
                filtered = fallbackDskp;
            }

            filtered.forEach((item, index) => {
                const option = document.createElement('option');
                option.value = index;
                option.dataset.sp = item.sp;
                const temaLabel = item.tema && item.tema !== '-' ? `[${item.tema}] ` : '';
                const tajukLabel = item.tajuk && item.tajuk !== '-' ? `${item.tajuk} - ` : '';
                option.innerText = `${temaLabel}${tajukLabel}${item.sk}`;
                selectTopic.appendChild(option);
            });

            updateSelectedSpBox();
        }

        function updateSelectedSpBox() {
            const selectTopic = document.getElementById('selectTopic');
            const box = document.getElementById('selectedSpBox');
            if (selectTopic.options.length > 0) {
                const selectedOpt = selectTopic.options[selectTopic.selectedIndex];
                box.innerText = selectedOpt.dataset.sp || "Tiada Standard Pembelajaran spesifik.";
            } else {
                box.innerText = "Tiada data DSKP dijumpai untuk subjek/tahun ini.";
            }
        }

        function renderRecordsTable() {
            updateSelectedSpBox();
            updateTpLegend();
            
            const year = document.getElementById('filterYear').value;
            const className = document.getElementById('filterClass').value;
            const subject = document.getElementById('filterSubject').value;
            const assessment = document.getElementById('filterAssessment').value;
            const isTahap2 = year.includes('TAHUN 4') || year.includes('TAHUN 5') || year.includes('TAHUN 6');

            const classKey = `${year}_${className}`;
            const students = studentsList[classKey] || [];
            const tpDescMap = getTpDescriptors(subject);

            document.getElementById('studentCountBadge').innerText = `${students.length} Murid Terdaftar`;

            const tbody = document.getElementById('tpTableBody');
            tbody.innerHTML = '';

            if (students.length === 0) {
                tbody.innerHTML = `
                    <tr>
                        <td colspan="6" class="p-8 text-center text-slate-400">
                            <i class="fa-solid fa-users-slash text-3xl mb-2 text-fuchsia-300 block"></i>
                            Tiada murid dalam kelas <strong>${year} ${className}</strong>.<br>
                            Sila tambah murid di tab <button onclick="switchTab('studentsTab')" class="text-fuchsia-600 font-bold underline">Pengurusan Murid</button>.
                        </td>
                    </tr>
                `;
                return;
            }

            const selectTopic = document.getElementById('selectTopic');
            const topicIndex = selectTopic.value || 0;

            students.forEach((student, idx) => {
                const tr = document.createElement('tr');
                tr.className = "hover:bg-fuchsia-50/50 transition";

                // Unique keys
                const topicTpKey = `${student.id}_${subject}_tp_${topicIndex}`;
                const summaryKey = `${student.id}_${subject}_${assessment}`;

                const currentTopicTp = tpRecords[topicTpKey] || 0;
                const currentSummary = summaryRecords[summaryKey] || { overallTp: 0, markah: '', ulasan: '' };

                // Build TP 1-6 selector buttons
                let tpButtonsHtml = `<div class="flex items-center justify-center gap-1">`;
                for (let tp = 1; tp <= 6; tp++) {
                    const isSelected = currentTopicTp == tp;
                    const colorMap = {
                        1: 'bg-rose-100 hover:bg-rose-200 text-rose-800 border-rose-300',
                        2: 'bg-orange-100 hover:bg-orange-200 text-orange-800 border-orange-300',
                        3: 'bg-amber-100 hover:bg-amber-200 text-amber-800 border-amber-300',
                        4: 'bg-emerald-100 hover:bg-emerald-200 text-emerald-800 border-emerald-300',
                        5: 'bg-teal-100 hover:bg-teal-200 text-teal-800 border-teal-300',
                        6: 'bg-fuchsia-100 hover:bg-fuchsia-200 text-fuchsia-900 border-fuchsia-300'
                    };
                    const selectedMap = {
                        1: 'bg-rose-600 text-white font-bold ring-2 ring-rose-400',
                        2: 'bg-orange-600 text-white font-bold ring-2 ring-orange-400',
                        3: 'bg-amber-600 text-white font-bold ring-2 ring-amber-400',
                        4: 'bg-emerald-600 text-white font-bold ring-2 ring-emerald-400',
                        5: 'bg-teal-600 text-white font-bold ring-2 ring-teal-400',
                        6: 'bg-fuchsia-600 text-white font-bold ring-2 ring-fuchsia-400'
                    };

                    const btnClass = isSelected ? selectedMap[tp] : colorMap[tp];

                    tpButtonsHtml += `
                        <button type="button" onclick="setTopicTp('${student.id}', '${subject}', ${topicIndex}, ${tp})" 
                                title="TP${tp}: ${tpDescMap[tp]}"
                                class="w-8 h-8 rounded-lg text-xs font-semibold border transition transform active:scale-95 flex items-center justify-center ${btnClass}">
                            ${tp}
                        </button>
                    `;
                }
                tpButtonsHtml += `</div>`;

                // Rumusan TP Dropdown
                let rumusanTpOptions = `<option value="0">Pilih TP</option>`;
                for (let t = 1; t <= 6; t++) {
                    const sel = currentSummary.overallTp == t ? 'selected' : '';
                    rumusanTpOptions += `<option value="${t}" ${sel}>TP ${t} - ${tpDescMap[t]}</option>`;
                }

                tr.innerHTML = `
                    <td class="p-3 text-center font-bold text-slate-500">${idx + 1}</td>
                    <td class="p-3">
                        <div class="font-bold text-slate-800">${student.name}</div>
                        <div class="text-[10px] text-slate-400">${student.kelas || className} • ${student.gender || 'L'}</div>
                    </td>
                    <td class="p-3 text-center">${tpButtonsHtml}</td>
                    <td class="p-3 text-center">
                        <select onchange="setSummaryField('${student.id}', '${subject}', '${assessment}', 'overallTp', this.value)" 
                                class="bg-fuchsia-50 border border-fuchsia-300 rounded-lg p-1.5 text-xs font-bold text-fuchsia-900 focus:ring-2 focus:ring-fuchsia-500 max-w-[160px]">
                            ${rumusanTpOptions}
                        </select>
                    </td>
                    ${isTahap2 ? `
                        <td class="p-3 text-center">
                            <input type="number" min="0" max="100" value="${currentSummary.markah || ''}" 
                                   placeholder="0 - 100" 
                                   onblur="setSummaryField('${student.id}', '${subject}', '${assessment}', 'markah', this.value)" 
                                   class="w-20 text-center bg-fuchsia-50 border border-fuchsia-300 rounded-lg p-1.5 text-xs font-bold text-slate-800">
                        </td>
                    ` : ''}
                    <td class="p-3">
                        <input type="text" value="${currentSummary.ulasan || ''}" 
                               placeholder="Tambah ulasan guru..." 
                               onblur="setSummaryField('${student.id}', '${subject}', '${assessment}', 'ulasan', this.value)" 
                               class="w-full bg-slate-50 border border-fuchsia-200 rounded-lg p-1.5 text-xs text-slate-700 focus:bg-white focus:ring-2 focus:ring-fuchsia-500">
                    </td>
                `;

                tbody.appendChild(tr);
            });
        }

        function setTopicTp(studentId, subject, topicIndex, tpValue) {
            const topicTpKey = `${studentId}_${subject}_tp_${topicIndex}`;
            tpRecords[topicTpKey] = tpValue;

            // Also auto-suggest overall TP if not yet set
            const assessment = document.getElementById('filterAssessment').value;
            const summaryKey = `${studentId}_${subject}_${assessment}`;
            if (!summaryRecords[summaryKey]) {
                summaryRecords[summaryKey] = { overallTp: tpValue, markah: '', ulasan: '' };
            } else if (!summaryRecords[summaryKey].overallTp || summaryRecords[summaryKey].overallTp == 0) {
                summaryRecords[summaryKey].overallTp = tpValue;
            }

            saveLocalCache();
            renderRecordsTable();
            showToast(`TP ${tpValue} direkodkan!`, 'success');
        }

        function setSummaryField(studentId, subject, assessment, field, value) {
            const summaryKey = `${studentId}_${subject}_${assessment}`;
            if (!summaryRecords[summaryKey]) {
                summaryRecords[summaryKey] = { overallTp: 0, markah: '', ulasan: '' };
            }
            summaryRecords[summaryKey][field] = value;
            saveLocalCache();
        }

        function openAddStudentModal() {
            const currentClass = document.getElementById('filterClass').value;
            const modalClassSelect = document.getElementById('modalStudentClass');
            if (modalClassSelect) {
                modalClassSelect.value = currentClass;
            }
            const modal = document.getElementById('addStudentModal');
            if (modal) {
                modal.classList.remove('hidden');
            }
        }

        function closeAddStudentModal() {
            const modal = document.getElementById('addStudentModal');
            if (modal) {
                modal.classList.add('hidden');
            }
            const nameInput = document.getElementById('modalStudentName');
            if (nameInput) nameInput.value = '';
        }

        function handleModalAddStudent(e) {
            e.preventDefault();
            const year = document.getElementById('filterYear').value;
            const selectedClass = document.getElementById('modalStudentClass').value;
            const classKey = `${year}_${selectedClass}`;

            const nameInput = document.getElementById('modalStudentName');
            const genderInput = document.getElementById('modalStudentGender');

            const name = nameInput.value.trim().toUpperCase();
            if (!name) return;

            if (!studentsList[classKey]) {
                studentsList[classKey] = [];
            }

            const newId = 'M' + Date.now().toString().substr(-5);
            studentsList[classKey].push({
                id: newId,
                name: name,
                gender: genderInput.value,
                kelas: selectedClass
            });

            saveLocalCache();
            closeAddStudentModal();
            showToast(`Murid '${name}' berjaya ditambah ke ${year} ${selectedClass}!`, 'success');
            renderStudentManagerTable();
            renderRecordsTable();
        }

        function handleAddIndividualStudent(e) {
            e.preventDefault();
            const year = document.getElementById('filterYear').value;
            const selectedClass = document.getElementById('addStudentClass').value;
            const classKey = `${year}_${selectedClass}`;

            const nameInput = document.getElementById('addStudentName');
            const genderInput = document.getElementById('addStudentGender');

            const name = nameInput.value.trim().toUpperCase();
            if (!name) return;

            if (!studentsList[classKey]) {
                studentsList[classKey] = [];
            }

            const newId = 'M' + Date.now().toString().substr(-5);
            studentsList[classKey].push({
                id: newId,
                name: name,
                gender: genderInput.value,
                kelas: selectedClass
            });

            saveLocalCache();
            nameInput.value = '';
            showToast(`Murid '${name}' berjaya ditambah ke ${year} ${selectedClass}!`, 'success');
            renderStudentManagerTable();
            renderRecordsTable();
        }

        function handleBulkAddStudents(e) {
            e.preventDefault();
            const year = document.getElementById('filterYear').value;
            const className = document.getElementById('filterClass').value;
            const classKey = `${year}_${className}`;

            const textarea = document.getElementById('bulkStudentList');
            const lines = textarea.value.split('\n');

            let addedCount = 0;
            if (!studentsList[classKey]) {
                studentsList[classKey] = [];
            }

            lines.forEach(line => {
                const trimmed = line.trim().toUpperCase();
                if (trimmed) {
                    const newId = 'MB' + Math.floor(1000 + Math.random() * 9000);
                    studentsList[classKey].push({
                        id: newId,
                        name: trimmed,
                        gender: 'Lelaki',
                        kelas: className
                    });
                    addedCount++;
                }
            });

            saveLocalCache();
            textarea.value = '';
            showToast(`${addedCount} murid berjaya ditambah secara pukal!`, 'success');
            renderStudentManagerTable();
            renderRecordsTable();
        }

        function changeStudentClass(oldYear, oldClass, studentId, newClass) {
            const oldClassKey = `${oldYear}_${oldClass}`;
            const newClassKey = `${oldYear}_${newClass}`;

            if (!studentsList[oldClassKey]) return;

            const studentIdx = studentsList[oldClassKey].findIndex(s => s.id === studentId);
            if (studentIdx > -1) {
                const [student] = studentsList[oldClassKey].splice(studentIdx, 1);
                student.kelas = newClass;

                if (!studentsList[newClassKey]) {
                    studentsList[newClassKey] = [];
                }
                studentsList[newClassKey].push(student);

                saveLocalCache();
                renderStudentManagerTable();
                renderRecordsTable();
                showToast(`Murid '${student.name}' dipindahkan ke kelas ${newClass}!`, 'success');
            }
        }

        function removeStudent(classKey, studentId) {
            if (studentsList[classKey]) {
                studentsList[classKey] = studentsList[classKey].filter(s => s.id !== studentId);
                saveLocalCache();
                renderStudentManagerTable();
                renderRecordsTable();
                showToast('Murid dipadam dari senarai.', 'info');
            }
        }

        function clearCurrentClassStudents() {
            const year = document.getElementById('filterYear').value;
            const className = document.getElementById('filterClass').value;
            const classKey = `${year}_${className}`;

            studentsList[classKey] = [];
            saveLocalCache();
            renderStudentManagerTable();
            renderRecordsTable();
            showToast(`Senarai murid ${year} ${className} dikosongkan.`, 'warning');
        }

        function renderStudentManagerTable() {
            const year = document.getElementById('filterYear').value;
            const className = document.getElementById('filterClass').value;
            const classKey = `${year}_${className}`;
            const students = studentsList[classKey] || [];

            const tbody = document.getElementById('studentManagerTableBody');
            tbody.innerHTML = '';

            if (students.length === 0) {
                tbody.innerHTML = `
                    <tr>
                        <td colspan="5" class="p-6 text-center text-slate-400">
                            Tiada murid terdaftar dalam kelas ini lagi.
                        </td>
                    </tr>
                `;
                return;
            }

            students.forEach((student, idx) => {
                const tr = document.createElement('tr');
                tr.className = "hover:bg-fuchsia-50/40 transition";
                tr.innerHTML = `
                    <td class="p-3 text-center font-bold text-slate-500">${idx + 1}</td>
                    <td class="p-3 font-semibold text-slate-800">${student.name}</td>
                    <td class="p-3">${student.gender}</td>
                    <td class="p-3">
                        <select onchange="changeStudentClass('${year}', '${className}', '${student.id}', this.value)" 
                                class="bg-fuchsia-50 border border-fuchsia-300 text-fuchsia-900 font-bold rounded-lg p-1.5 text-xs focus:ring-2 focus:ring-fuchsia-500">
                            <option value="INOVATIF" ${ (student.kelas || className) === 'INOVATIF' ? 'selected' : '' }>INOVATIF</option>
                            <option value="KREATIF" ${ (student.kelas || className) === 'KREATIF' ? 'selected' : '' }>KREATIF</option>
                            <option value="PROAKTIF" ${ (student.kelas || className) === 'PROAKTIF' ? 'selected' : '' }>PROAKTIF</option>
                        </select>
                    </td>
                    <td class="p-3 text-center">
                        <button onclick="removeStudent('${classKey}', '${student.id}')" class="text-rose-600 hover:text-rose-800 text-xs p-1">
                            <i class="fa-solid fa-user-minus"></i> Padam
                        </button>
                    </td>
                `;
                tbody.appendChild(tr);
            });
        }

        function renderAnalyticsCharts() {
            const year = document.getElementById('filterYear').value;
            const className = document.getElementById('filterClass').value;
            const subject = document.getElementById('filterSubject').value;
            const assessment = document.getElementById('filterAssessment').value;

            const isPj = subject.includes('PJ') || subject.includes('PENDIDIKAN JASMANI');
            document.getElementById('analyticsSubjectBadge').innerText = isPj ? 'Rujukan TP: Pendidikan Jasmani (PJ)' : 'Rujukan TP: Standard KSSR';

            const classKey = `${year}_${className}`;
            const students = studentsList[classKey] || [];

            document.getElementById('chartFilterBadge').innerText = `${subject} (${year} ${className})`;

            // Count TP distribution
            let counts = { 1: 0, 2: 0, 3: 0, 4: 0, 5: 0, 6: 0, unassigned: 0 };
            let totalStudents = students.length;
            let sumTp = 0;
            let assessedCount = 0;

            students.forEach(student => {
                const summaryKey = `${student.id}_${subject}_${assessment}`;
                const rec = summaryRecords[summaryKey];
                const tp = rec ? parseInt(rec.overallTp) || 0 : 0;

                if (tp >= 1 && tp <= 6) {
                    counts[tp]++;
                    sumTp += tp;
                    assessedCount++;
                } else {
                    counts.unassigned++;
                }
            });

            // Update KPI cards
            document.getElementById('kpiTotalStudents').innerText = totalStudents;

            const passCount = counts[3] + counts[4] + counts[5] + counts[6];
            const passRate = totalStudents > 0 ? Math.round((passCount / totalStudents) * 100) : 0;
            document.getElementById('kpiPassRate').innerText = `${passRate}%`;

            const excellenceCount = counts[5] + counts[6];
            const excellenceRate = totalStudents > 0 ? Math.round((excellenceCount / totalStudents) * 100) : 0;
            document.getElementById('kpiExcellenceRate').innerText = `${excellenceRate}%`;

            const avgTp = assessedCount > 0 ? (sumTp / assessedCount).toFixed(1) : "0.0";
            document.getElementById('kpiAvgTp').innerText = avgTp;

            // Render Bar Chart
            const ctxBar = document.getElementById('tpBarChart').getContext('2d');
            if (tpChart) tpChart.destroy();

            tpChart = new Chart(ctxBar, {
                type: 'bar',
                data: {
                    labels: ['TP1', 'TP2', 'TP3', 'TP4', 'TP5', 'TP6'],
                    datasets: [{
                        label: 'Bilangan Murid',
                        data: [counts[1], counts[2], counts[3], counts[4], counts[5], counts[6]],
                        backgroundColor: [
                            '#f43f5e', '#f97316', '#f59e0b', '#10b981', '#14b8a6', '#d946ef'
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
                        y: { beginAtZero: true, ticks: { stepSize: 1 } }
                    }
                }
            });

            // Render Pie Chart
            const ctxPie = document.getElementById('tpPieChart').getContext('2d');
            if (pieChart) pieChart.destroy();

            pieChart = new Chart(ctxPie, {
                type: 'doughnut',
                data: {
                    labels: ['Tahap Asas (TP1-2)', 'Beradab / Menguasai (TP3-4)', 'Terpuji & Mithali (TP5-6)'],
                    datasets: [{
                        data: [
                            counts[1] + counts[2],
                            counts[3] + counts[4],
                            counts[5] + counts[6]
                        ],
                        backgroundColor: ['#f43f5e', '#10b981', '#d946ef']
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

            // Populate Breakdown Table with PJ or standard descriptors
            const tbody = document.getElementById('analyticsTableBody');
            tbody.innerHTML = '';

            const tpDescMap = getTpDescriptors(subject);

            const tpInfo = [
                { tp: "TP 1", desc: tpDescMap[1], count: counts[1], bg: "bg-rose-100 text-rose-800" },
                { tp: "TP 2", desc: tpDescMap[2], count: counts[2], bg: "bg-orange-100 text-orange-800" },
                { tp: "TP 3", desc: tpDescMap[3], count: counts[3], bg: "bg-amber-100 text-amber-800" },
                { tp: "TP 4", desc: tpDescMap[4], count: counts[4], bg: "bg-emerald-100 text-emerald-800" },
                { tp: "TP 5", desc: tpDescMap[5], count: counts[5], bg: "bg-teal-100 text-teal-800" },
                { tp: "TP 6", desc: tpDescMap[6], count: counts[6], bg: "bg-fuchsia-100 text-fuchsia-800" }
            ];

            tpInfo.forEach(item => {
                const pct = totalStudents > 0 ? ((item.count / totalStudents) * 100).toFixed(1) : 0;
                const tr = document.createElement('tr');
                tr.innerHTML = `
                    <td class="p-3 font-bold"><span class="px-2 py-1 rounded ${item.bg}">${item.tp}</span></td>
                    <td class="p-3 font-semibold text-slate-800">${item.desc}</td>
                    <td class="p-3 text-center font-bold text-slate-800">${item.count}</td>
                    <td class="p-3 text-center font-bold text-slate-800">${pct}%</td>
                    <td class="p-3 font-semibold">
                        ${item.tp.includes('1') || item.tp.includes('2') ? 
                            '<span class="text-rose-600">Belum Menguasai (Perkara Asas)</span>' : 
                            '<span class="text-emerald-600">Telah Menguasai</span>'}
                    </td>
                `;
                tbody.appendChild(tr);
            });
        }

        function populateDskpTable() {
            const tbody = document.getElementById('dskpTableBody');
            tbody.innerHTML = '';

            dskpData.forEach((item, idx) => {
                const tr = document.createElement('tr');
                tr.className = "hover:bg-fuchsia-50/50 transition";
                tr.innerHTML = `
                    <td class="p-3 font-bold text-fuchsia-900">${item.subjek}</td>
                    <td class="p-3 font-semibold">${item.tahun}</td>
                    <td class="p-3 text-slate-700">${item.tema}</td>
                    <td class="p-3 font-medium text-slate-800">${item.tajuk}</td>
                    <td class="p-3 text-slate-700">${item.sk}</td>
                    <td class="p-3 text-slate-900 font-medium">${item.sp}</td>
                `;
                tbody.appendChild(tr);
            });
        }

        function filterDskpTable() {
            const input = document.getElementById('searchDskp').value.toLowerCase();
            const rows = document.querySelectorAll('#dskpTableBody tr');

            rows.forEach(row => {
                const text = row.innerText.toLowerCase();
                row.style.display = text.includes(input) ? '' : 'none';
            });
        }

        function renderPrintReport() {
            const subject = document.getElementById('filterSubject').value;
            const year = document.getElementById('filterYear').value;
            const className = document.getElementById('filterClass').value;
            const assessment = document.getElementById('filterAssessment').value;
            const isTahap2 = year.includes('TAHUN 4') || year.includes('TAHUN 5') || year.includes('TAHUN 6');

            const tpDescMap = getTpDescriptors(subject);

            document.getElementById('pdfSubject').innerText = subject;
            document.getElementById('pdfClass').innerText = `${year} ${className}`;
            document.getElementById('pdfAssessment').innerText = assessment;

            const classKey = `${year}_${className}`;
            const students = studentsList[classKey] || [];
            document.getElementById('pdfStudentCount').innerText = `${students.length} Murid`;

            const markahHeader = document.getElementById('pdfMarkahHeader');
            if (isTahap2) {
                markahHeader.classList.remove('hidden');
            } else {
                markahHeader.classList.add('hidden');
            }

            const tbody = document.getElementById('pdfTableBody');
            tbody.innerHTML = '';

            let tpCounts = { 1: 0, 2: 0, 3: 0, 4: 0, 5: 0, 6: 0, 0: 0 };

            if (students.length === 0) {
                tbody.innerHTML = `
                    <tr>
                        <td colspan="${isTahap2 ? 5 : 4}" class="p-4 text-center text-slate-400">
                            Tiada rekod murid untuk dicetak.
                        </td>
                    </tr>
                `;
            } else {
                students.forEach((student, idx) => {
                    const summaryKey = `${student.id}_${subject}_${assessment}`;
                    const rec = summaryRecords[summaryKey] || { overallTp: 0, markah: '', ulasan: '' };
                    const tpVal = rec.overallTp || 0;
                    tpCounts[tpVal] = (tpCounts[tpVal] || 0) + 1;

                    const descLabel = tpVal > 0 ? tpDescMap[tpVal] : 'Belum Dinilai';

                    const tr = document.createElement('tr');
                    tr.className = "border-b border-slate-200";
                    tr.innerHTML = `
                        <td class="border border-slate-300 p-2 text-center font-bold">${idx + 1}</td>
                        <td class="border border-slate-300 p-2 font-semibold text-slate-900">${student.name}</td>
                        <td class="border border-slate-300 p-2 text-center font-extrabold text-fuchsia-900">
                            ${tpVal > 0 ? `TP ${tpVal}` : 'Belum Dinilai'}
                        </td>
                        ${isTahap2 ? `<td class="border border-slate-300 p-2 text-center font-bold">${rec.markah || '-'}%</td>` : ''}
                        <td class="border border-slate-300 p-2 text-slate-800 font-medium">
                            ${tpVal > 0 ? `<div class="font-bold text-slate-900">${descLabel}</div>` : ''}
                            <span class="text-slate-600 text-[11px]">${rec.ulasan || 'Melaksanakan aktiviti dengan beradab dan teratur.'}</span>
                        </td>
                    `;
                    tbody.appendChild(tr);
                });
            }

            // PDF TP Breakdown summary text with descriptors
            const breakdownContainer = document.getElementById('pdfTpBreakdownText');
            breakdownContainer.innerHTML = `
                <div>• TP1 (${tpDescMap[1]}): <strong>${tpCounts[1]} murid</strong></div>
                <div>• TP2 (${tpDescMap[2]}): <strong>${tpCounts[2]} murid</strong></div>
                <div>• TP3 (${tpDescMap[3]}): <strong>${tpCounts[3]} murid</strong></div>
                <div>• TP4 (${tpDescMap[4]}): <strong>${tpCounts[4]} murid</strong></div>
                <div>• TP5 (${tpDescMap[5]}): <strong>${tpCounts[5]} murid</strong></div>
                <div>• TP6 (${tpDescMap[6]}): <strong>${tpCounts[6]} murid</strong></div>
            `;

            const passTotal = tpCounts[3] + tpCounts[4] + tpCounts[5] + tpCounts[6];
            const passPct = students.length > 0 ? Math.round((passTotal / students.length) * 100) : 0;
            document.getElementById('pdfPassCount').innerText = `${passTotal} / ${students.length} (${passPct}%)`;
        }

        function saveLocalCache() {
            try {
                localStorage.setItem('skbk_students', JSON.stringify(studentsList));
                localStorage.setItem('skbk_tp_records', JSON.stringify(tpRecords));
                localStorage.setItem('skbk_summary_records', JSON.stringify(summaryRecords));
            } catch (e) {
                console.warn('Storage error:', e);
            }
        }

        function loadLocalCache() {
            try {
                const s = localStorage.getItem('skbk_students');
                const t = localStorage.getItem('skbk_tp_records');
                const sum = localStorage.getItem('skbk_summary_records');

                studentsList = s ? JSON.parse(s) : defaultInitialStudents;
                tpRecords = t ? JSON.parse(t) : {};
                summaryRecords = sum ? JSON.parse(sum) : {};
            } catch (e) {
                studentsList = defaultInitialStudents;
                tpRecords = {};
                summaryRecords = {};
            }
        }

        function saveAllScoresToFirebase() {
            saveLocalCache();
            showToast('Semua rekod PBD berjaya disimpan ke storan simpanan!', 'success');
        }

        window.openAddStudentModal = openAddStudentModal;
        window.closeAddStudentModal = closeAddStudentModal;
        window.handleModalAddStudent = handleModalAddStudent;
        window.handleAddIndividualStudent = handleAddIndividualStudent;
        window.handleBulkAddStudents = handleBulkAddStudents;
        window.refreshGoogleSheetsData = refreshGoogleSheetsData;
        window.switchTab = switchTab;
        window.onFilterChange = onFilterChange;
        window.renderRecordsTable = renderRecordsTable;
        window.setTopicTp = setTopicTp;
        window.setSummaryField = setSummaryField;
        window.clearCurrentClassStudents = clearCurrentClassStudents;
        window.removeStudent = removeStudent;
        window.changeStudentClass = changeStudentClass;
        window.filterDskpTable = filterDskpTable;
        window.saveAllScoresToFirebase = saveAllScoresToFirebase;

        function showToast(message, type = 'info') {
            const container = document.getElementById('toastContainer');
            const toast = document.createElement('div');

            const bgMap = {
                success: 'bg-emerald-600 text-white',
                warning: 'bg-amber-600 text-white',
                info: 'bg-fuchsia-800 text-white',
                error: 'bg-rose-600 text-white'
            };

            const iconMap = {
                success: 'fa-circle-check',
                warning: 'fa-triangle-exclamation',
                info: 'fa-circle-info',
                error: 'fa-circle-xmark'
            };

            toast.className = `flex items-center gap-2 px-4 py-3 rounded-xl shadow-lg text-xs font-semibold transform transition-all duration-300 translate-y-2 opacity-0 ${bgMap[type] || bgMap.info}`;
            toast.innerHTML = `<i class="fa-solid ${iconMap[type] || iconMap.info}"></i> <span>${message}</span>`;

            container.appendChild(toast);

            setTimeout(() => {
                toast.classList.remove('translate-y-2', 'opacity-0');
            }, 10);

            setTimeout(() => {
                toast.classList.add('opacity-0', 'translate-y-2');
                setTimeout(() => toast.remove(), 300);
            }, 3500);
        }
    </script>
</body>
</html>
