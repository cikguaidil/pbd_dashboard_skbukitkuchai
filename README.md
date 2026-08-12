<!DOCTYPE html>
<html lang="ms">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dashboard PBD - SK Bukit Kuchai</title>
    
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Chart.js -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/3.9.1/chart.min.js"></script>
    <!-- PapaParse for CSV -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.3.2/papaparse.min.js"></script>
    <!-- FontAwesome -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        primary: '#1e3a8a', // blue-900
                        secondary: '#3b82f6', // blue-500
                        success: '#10b981',
                        warning: '#f59e0b',
                        danger: '#ef4444',
                        tp1: '#ef4444', // red
                        tp2: '#f97316', // orange
                        tp3: '#eab308', // yellow
                        tp4: '#84cc16', // lime
                        tp5: '#22c55e', // green
                        tp6: '#0ea5e9', // sky
                    }
                }
            }
        }
    </script>

    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');
        body { font-family: 'Inter', sans-serif; background-color: #f3f4f6; }
        
        /* Custom Scrollbar */
        ::-webkit-scrollbar { width: 8px; height: 8px; }
        ::-webkit-scrollbar-track { background: #f1f1f1; }
        ::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 4px; }
        ::-webkit-scrollbar-thumb:hover { background: #94a3b8; }

        /* Print Styles */
        @media print {
            body { background-color: white; }
            .no-print { display: none !important; }
            .print-only { display: block !important; }
            .print-hidden { display: none !important; }
            .print-w-full { width: 100% !important; max-width: 100% !important; margin: 0 !important; padding: 0 !important; box-shadow: none !important; }
            .print-break-inside-avoid { break-inside: avoid; }
            #main-content { margin-left: 0 !important; width: 100% !important; padding: 0 !important; }
            table { font-size: 12px; }
            th, td { padding: 4px !important; border: 1px solid #e5e7eb !important; }
            .tp-box { border: 1px solid #d1d5db !important; }

            /* Specific for Report Printing */
            body.printing-report #sidebar, 
            body.printing-report #main-content { display: none !important; }
            body.printing-report #modal-report { position: relative; display: block !important; z-index: 1000; opacity: 1 !important; transform: none !important; }
            body.printing-report #modal-report > div { box-shadow: none; border: none; height: auto; max-width: 100%; overflow: visible; }
            body.printing-report #modal-report .border-b { display: none; }
        }
    </style>
</head>
<body class="text-gray-800 antialiased h-screen flex overflow-hidden">

    <!-- Sidebar / Filter Panel -->
    <aside class="w-72 bg-white shadow-xl h-full flex flex-col z-20 no-print transition-all duration-300 flex-shrink-0" id="sidebar">
        <div class="p-6 bg-primary text-white flex items-center justify-between">
            <div>
                <h1 class="text-xl font-bold tracking-wider">PBD DASHBOARD</h1>
                <p class="text-sm text-blue-200">SK Bukit Kuchai</p>
            </div>
            <i class="fas fa-graduation-cap text-3xl opacity-80"></i>
        </div>

        <div class="p-5 flex-1 overflow-y-auto space-y-4 text-sm">
            <h2 class="font-semibold text-gray-500 uppercase tracking-wider mb-2 text-xs">Penapis Utama</h2>
            
            <!-- Fasa Penilaian -->
            <div class="mb-3 border-b pb-3 border-blue-100">
                <label class="block text-xs font-bold text-primary mb-1">Fasa Penilaian</label>
                <select id="filter-fasa" class="w-full rounded-md border-primary border-2 p-2 focus:ring-primary focus:border-primary shadow-sm bg-blue-50 font-semibold text-primary outline-none">
                    <option value="pertengahan">PBD Pertengahan Tahun</option>
                    <option value="akhir">PBD Akhir Tahun</option>
                </select>
            </div>

            <!-- Filters -->
            <div>
                <label class="block text-xs font-medium text-gray-700 mb-1">Subjek</label>
                <select id="filter-subjek" class="w-full rounded-md border-gray-300 border p-2 focus:ring-primary focus:border-primary shadow-sm"></select>
            </div>
            <div class="grid grid-cols-2 gap-2">
                <div>
                    <label class="block text-xs font-medium text-gray-700 mb-1">Tahun</label>
                    <select id="filter-tahun" class="w-full rounded-md border-gray-300 border p-2 focus:ring-primary focus:border-primary shadow-sm"></select>
                </div>
                <div>
                    <label class="block text-xs font-medium text-gray-700 mb-1">Kelas</label>
                    <select id="filter-kelas" class="w-full rounded-md border-gray-300 border p-2 focus:ring-primary focus:border-primary shadow-sm"></select>
                </div>
            </div>
            
            <div class="border-t pt-4 mt-4">
                <h2 class="font-semibold text-gray-500 uppercase tracking-wider mb-2 text-xs">Kandungan Kurikulum</h2>
                <div class="space-y-3">
                    <div>
                        <label class="block text-xs font-medium text-gray-700 mb-1">Tema</label>
                        <select id="filter-tema" class="w-full rounded-md border-gray-300 border p-2 focus:ring-primary focus:border-primary shadow-sm"></select>
                    </div>
                    <div>
                        <label class="block text-xs font-medium text-gray-700 mb-1">Bidang</label>
                        <select id="filter-bidang" class="w-full rounded-md border-gray-300 border p-2 focus:ring-primary focus:border-primary shadow-sm"></select>
                    </div>
                    <div>
                        <label class="block text-xs font-medium text-gray-700 mb-1">Standard Kandungan (SK)</label>
                        <select id="filter-sk" class="w-full rounded-md border-gray-300 border p-2 focus:ring-primary focus:border-primary shadow-sm text-xs truncate"></select>
                    </div>
                    <div>
                        <label class="block text-xs font-medium text-gray-700 mb-1">Standard Pembelajaran (SP) <span class="text-red-500">*Wajib</span></label>
                        <select id="filter-sp" class="w-full rounded-md border-gray-300 border p-2 focus:ring-primary focus:border-primary shadow-sm text-xs truncate bg-blue-50"></select>
                    </div>
                </div>
            </div>

            <div class="border-t pt-4 mt-4">
                <h2 class="font-semibold text-gray-500 uppercase tracking-wider mb-2 text-xs">Pengurusan Murid</h2>
                <div class="grid grid-cols-2 gap-2">
                    <button onclick="openModal('modal-add-single')" class="bg-emerald-50 text-emerald-700 border border-emerald-200 px-3 py-2 rounded-lg text-xs font-semibold hover:bg-emerald-100 transition flex items-center justify-center gap-1.5 shadow-sm active:scale-95">
                        <i class="fas fa-user-plus"></i> + Individu
                    </button>
                    <button onclick="openModal('modal-add-bulk')" class="bg-blue-50 text-blue-700 border border-blue-200 px-3 py-2 rounded-lg text-xs font-semibold hover:bg-blue-100 transition flex items-center justify-center gap-1.5 shadow-sm active:scale-95">
                        <i class="fas fa-users"></i> + Pukal
                    </button>
                </div>
            </div>
        </div>

        <div class="p-4 border-t bg-gray-50 flex justify-between">
            <div class="flex flex-col gap-2 w-full">
                <!-- Status Auto-Sync -->
                <div class="flex items-center justify-between text-[11px] text-gray-500 font-medium px-1">
                    <span class="flex items-center gap-1.5 text-emerald-600 font-semibold">
                        <span class="w-2 h-2 rounded-full bg-emerald-500 animate-pulse"></span>
                        Auto-Sync Aktif
                    </span>
                    <span id="last-sync-time" class="text-gray-400">Baru Sahaja</span>
                </div>
                <button onclick="syncGoogleSheets(true)" class="bg-white border-2 border-primary text-primary px-4 py-2 rounded-lg text-sm font-medium hover:bg-blue-50 transition w-full shadow-sm flex items-center justify-center gap-2">
                    <i class="fas fa-cloud-download-alt"></i> Kemaskini Sheet
                </button>
                <button id="btn-save" onclick="saveDataToFirebase()" class="bg-primary text-white px-4 py-2 rounded-lg text-sm font-medium hover:bg-blue-800 transition w-full shadow-md flex items-center justify-center gap-2">
                    <i class="fas fa-save"></i> Simpan Data
                </button>
            </div>
        </div>
    </aside>

    <!-- Main Content -->
    <main class="flex-1 flex flex-col h-full overflow-hidden bg-gray-50 print-w-full" id="main-content">
        <!-- Header -->
        <header class="bg-white shadow-sm border-b px-8 py-4 flex justify-between items-center no-print flex-wrap gap-3">
            <div class="flex items-center gap-4">
                <button id="mobile-menu" class="lg:hidden text-gray-600 hover:text-primary"><i class="fas fa-bars text-xl"></i></button>
                <h2 class="text-2xl font-bold text-gray-800">Rekod Perkembangan Murid</h2>
            </div>

            <!-- Bilah Carian Nama Murid -->
            <div class="relative flex-1 max-w-xs mx-2">
                <i class="fas fa-search absolute left-3 top-1/2 -translate-y-1/2 text-gray-400"></i>
                <input type="text" id="search-input" placeholder="Cari nama murid..." class="w-full pl-9 pr-4 py-2 border border-gray-300 rounded-lg text-sm focus:outline-none focus:ring-2 focus:ring-primary focus:border-transparent shadow-sm">
            </div>

            <div class="flex gap-3 items-center flex-wrap">
                <button onclick="generateReport()" class="bg-purple-600 text-white px-4 py-2 rounded-lg text-sm font-medium hover:bg-purple-700 transition shadow-sm flex items-center gap-2">
                    <i class="fas fa-file-invoice"></i> Laporan Keseluruhan
                </button>
                <button onclick="window.print()" class="bg-white border border-gray-300 text-gray-700 px-4 py-2 rounded-lg text-sm font-medium hover:bg-gray-100 transition shadow-sm flex items-center gap-2">
                    <i class="fas fa-print"></i> Cetak PDF
                </button>
                
                <!-- Explicit click-based Dropdown -->
                <div class="relative">
                    <button onclick="toggleStudentDropdown(event)" class="bg-success text-white px-4 py-2 rounded-lg text-sm font-semibold hover:bg-emerald-600 active:scale-95 transition shadow-sm flex items-center gap-2 cursor-pointer">
                        <i class="fas fa-user-plus"></i>
                        <span>Tambah Murid</span>
                        <i class="fas fa-chevron-down text-xs opacity-80"></i>
                    </button>
                    
                    <!-- Dropdown Menu -->
                    <div id="student-dropdown-menu" class="hidden absolute right-0 mt-2 w-56 bg-white rounded-xl shadow-2xl border border-gray-100 z-50 overflow-hidden py-1 transition-all">
                        <button onclick="openModal('modal-add-single'); closeStudentDropdown();" class="w-full text-left px-4 py-3 text-sm font-medium text-gray-700 hover:bg-emerald-50 hover:text-emerald-700 flex items-center gap-3 transition border-b border-gray-100">
                            <div class="w-8 h-8 rounded-lg bg-emerald-100 text-emerald-600 flex items-center justify-center text-xs flex-shrink-0">
                                <i class="fas fa-user"></i>
                            </div>
                            <div>
                                <div class="font-semibold text-gray-800">Individu</div>
                                <div class="text-xs text-gray-400 font-normal">Tambah 1 murid baharu</div>
                            </div>
                        </button>
                        <button onclick="openModal('modal-add-bulk'); closeStudentDropdown();" class="w-full text-left px-4 py-3 text-sm font-medium text-gray-700 hover:bg-blue-50 hover:text-blue-700 flex items-center gap-3 transition">
                            <div class="w-8 h-8 rounded-lg bg-blue-100 text-blue-600 flex items-center justify-center text-xs flex-shrink-0">
                                <i class="fas fa-users"></i>
                            </div>
                            <div>
                                <div class="font-semibold text-gray-800">Pukal (Bulk)</div>
                                <div class="text-xs text-gray-400 font-normal">Salin & tampal senarai</div>
                            </div>
                        </button>
                    </div>
                </div>
            </div>
        </header>

        <!-- Content Body -->
        <div class="flex-1 overflow-y-auto p-8 print-w-full" id="content-body">
            
            <!-- Context Header for Print -->
            <div class="hidden print-only mb-6 text-center">
                <h1 class="text-2xl font-bold">REKOD PERKEMBANGAN MURID (PBD)</h1>
                <h2 class="text-xl font-semibold mt-1">SK BUKIT KUCHAI</h2>
                <div class="grid grid-cols-2 text-left mt-6 border p-4 text-sm gap-2">
                    <p><strong>Subjek:</strong> <span id="print-subjek">-</span></p>
                    <p><strong>Tahun/Kelas:</strong> <span id="print-kelas">-</span></p>
                    <p><strong>Standard Kandungan:</strong> <span id="print-sk">-</span></p>
                    <p><strong>Standard Pembelajaran:</strong> <span id="print-sp">-</span></p>
                </div>
            </div>

            <!-- Stats & Analytics -->
            <div class="grid grid-cols-1 md:grid-cols-3 gap-6 mb-8 no-print">
                <div class="bg-white rounded-xl shadow-sm border border-gray-100 p-6 flex items-center gap-4 border-l-4 border-l-primary">
                    <div class="bg-blue-100 text-primary w-12 h-12 rounded-full flex items-center justify-center text-xl">
                        <i class="fas fa-users"></i>
                    </div>
                    <div>
                        <p class="text-sm text-gray-500 font-medium">Jumlah Murid</p>
                        <h3 class="text-2xl font-bold text-gray-800" id="stat-total">0</h3>
                    </div>
                </div>
                <div class="bg-white rounded-xl shadow-sm border border-gray-100 p-6 flex items-center gap-4 border-l-4 border-l-success">
                    <div class="bg-emerald-100 text-success w-12 h-12 rounded-full flex items-center justify-center text-xl">
                        <i class="fas fa-check-circle"></i>
                    </div>
                    <div>
                        <p class="text-sm text-gray-500 font-medium">Telah Dinilai (SP Semasa)</p>
                        <h3 class="text-2xl font-bold text-gray-800" id="stat-assessed">0</h3>
                    </div>
                </div>
                <div class="bg-white rounded-xl shadow-sm border border-gray-100 p-6 flex items-center gap-4 border-l-4 border-l-warning">
                    <div class="bg-amber-100 text-warning w-12 h-12 rounded-full flex items-center justify-center text-xl">
                        <i class="fas fa-chart-line"></i>
                    </div>
                    <div>
                        <p class="text-sm text-gray-500 font-medium">Purata TP Kelas</p>
                        <h3 class="text-2xl font-bold text-gray-800" id="stat-avg">0.0</h3>
                    </div>
                </div>
            </div>

            <!-- Chart -->
            <div class="bg-white rounded-xl shadow-sm border border-gray-100 p-6 mb-8 no-print">
                <h3 class="text-lg font-semibold text-gray-800 mb-4">Analisis Tahap Penguasaan (SP Semasa)</h3>
                <div class="h-64 w-full">
                    <canvas id="tpChart"></canvas>
                </div>
            </div>

            <!-- Student Table -->
            <div class="bg-white rounded-xl shadow-sm border border-gray-100 overflow-hidden print-w-full">
                <div class="p-5 border-b bg-gray-50 flex justify-between items-center no-print">
                    <h3 class="text-lg font-semibold text-gray-800">Senarai Nama Murid & Penilaian</h3>
                    <div class="flex items-center gap-2 text-sm text-gray-500">
                        <i class="fas fa-info-circle text-secondary"></i> Sila pilih <b>Standard Pembelajaran (SP)</b> di panel kiri sebelum mula menilai.
                    </div>
                </div>
                
                <div class="overflow-x-auto print-w-full">
                    <table class="w-full text-left border-collapse">
                        <thead>
                            <tr class="bg-gray-100 text-gray-600 text-sm uppercase tracking-wider border-b">
                                <th class="p-4 w-12 text-center font-semibold">No</th>
                                <th class="p-4 font-semibold">Nama Murid</th>
                                <th class="p-4 text-center font-semibold w-[350px]">Tahap Penguasaan (TP)</th>
                                <th id="th-markah" class="p-4 text-center font-semibold w-28 hidden">Markah</th>
                                <th class="p-4 text-center font-semibold w-24 no-print">Tindakan</th>
                            </tr>
                        </thead>
                        <tbody id="student-table-body" class="text-gray-700 text-sm divide-y divide-gray-100">
                            <!-- Rows will be injected by JS -->
                        </tbody>
                    </table>
                </div>
                <div id="empty-state" class="p-12 text-center hidden no-print">
                    <i class="fas fa-folder-open text-5xl text-gray-300 mb-4"></i>
                    <p class="text-gray-500 text-lg">Tiada murid dijumpai. Sila ubah tapisan atau tambah murid baru.</p>
                </div>
            </div>
            
            <div class="mt-8 text-center text-xs text-gray-400 no-print">
                Sistem Perekodan PBD - SK Bukit Kuchai &copy; 2026
            </div>
        </div>
    </main>

    <!-- Modal: Add Single Student -->
    <div id="modal-add-single" class="fixed inset-0 bg-black/60 z-50 hidden flex items-center justify-center opacity-0 transition-opacity no-print">
        <div class="bg-white rounded-xl shadow-2xl w-full max-w-md overflow-hidden transform scale-95 transition-transform duration-300" id="modal-single-content">
            <div class="px-6 py-4 border-b bg-gray-50 flex justify-between items-center">
                <h3 class="text-lg font-bold text-gray-800">Tambah Murid Individu</h3>
                <button onclick="closeModal('modal-add-single')" class="text-gray-400 hover:text-gray-600"><i class="fas fa-times text-xl"></i></button>
            </div>
            <div class="p-6 space-y-4">
                <div>
                    <label class="block text-sm font-medium text-gray-700 mb-1">Nama Penuh Murid</label>
                    <input type="text" id="add-nama" class="w-full border-gray-300 border rounded-lg p-2.5 focus:ring-2 focus:ring-primary focus:border-primary outline-none uppercase" placeholder="Contoh: AHMAD ALBA">
                </div>
                <div class="grid grid-cols-2 gap-4">
                    <div>
                        <label class="block text-sm font-medium text-gray-700 mb-1">Tahun</label>
                        <select id="add-tahun" class="w-full border-gray-300 border rounded-lg p-2.5 outline-none"></select>
                    </div>
                    <div>
                        <label class="block text-sm font-medium text-gray-700 mb-1">Kelas</label>
                        <select id="add-kelas" class="w-full border-gray-300 border rounded-lg p-2.5 outline-none"></select>
                    </div>
                </div>
            </div>
            <div class="px-6 py-4 border-t bg-gray-50 flex justify-end gap-3">
                <button onclick="closeModal('modal-add-single')" class="px-4 py-2 bg-white border rounded-lg text-gray-700 hover:bg-gray-50">Batal</button>
                <button onclick="saveSingleStudent()" class="px-4 py-2 bg-primary text-white rounded-lg hover:bg-blue-800 shadow-md">Simpan Murid</button>
            </div>
        </div>
    </div>

    <!-- Modal: Add Bulk Students -->
    <div id="modal-add-bulk" class="fixed inset-0 bg-black/60 z-50 hidden flex items-center justify-center opacity-0 transition-opacity no-print">
        <div class="bg-white rounded-xl shadow-2xl w-full max-w-2xl overflow-hidden transform scale-95 transition-transform duration-300">
            <div class="px-6 py-4 border-b bg-gray-50 flex justify-between items-center">
                <h3 class="text-lg font-bold text-gray-800">Tambah Murid Pukal (Salin & Tampal)</h3>
                <button onclick="closeModal('modal-add-bulk')" class="text-gray-400 hover:text-gray-600"><i class="fas fa-times text-xl"></i></button>
            </div>
            <div class="p-6 space-y-4">
                <div class="grid grid-cols-2 gap-4 mb-4 border-b pb-4">
                    <div>
                        <label class="block text-sm font-medium text-gray-700 mb-1">Tahun</label>
                        <select id="bulk-tahun" class="w-full border-gray-300 border rounded-lg p-2.5 outline-none"></select>
                    </div>
                    <div>
                        <label class="block text-sm font-medium text-gray-700 mb-1">Kelas</label>
                        <select id="bulk-kelas" class="w-full border-gray-300 border rounded-lg p-2.5 outline-none"></select>
                    </div>
                </div>
                <div>
                    <label class="block text-sm font-medium text-gray-700 mb-1 flex justify-between">
                        <span>Senarai Nama Murid</span>
                        <span class="text-xs text-gray-400 font-normal">Satu nama per baris (Tekan Enter)</span>
                    </label>
                    <textarea id="bulk-nama" rows="10" class="w-full border-gray-300 border rounded-lg p-3 focus:ring-2 focus:ring-primary focus:border-primary outline-none uppercase font-mono text-sm leading-relaxed" placeholder="Contoh:&#10;ALI BIN ABU&#10;SITI BINTI AWANG&#10;CHONG WEI"></textarea>
                </div>
            </div>
            <div class="px-6 py-4 border-t bg-gray-50 flex justify-between items-center">
                <span id="bulk-count" class="text-sm font-semibold text-secondary">0 Murid Sedia Ditambah</span>
                <div class="flex gap-3">
                    <button onclick="closeModal('modal-add-bulk')" class="px-4 py-2 bg-white border rounded-lg text-gray-700 hover:bg-gray-50">Batal</button>
                    <button onclick="saveBulkStudents()" class="px-4 py-2 bg-success text-white rounded-lg hover:bg-emerald-600 shadow-md">Simpan Semua</button>
                </div>
            </div>
        </div>
    </div>
    
    <!-- Modal: Laporan Keseluruhan -->
    <div id="modal-report" class="fixed inset-0 bg-black/60 z-50 hidden flex items-center justify-center opacity-0 transition-opacity no-print">
        <div class="bg-white rounded-xl shadow-2xl w-full max-w-5xl h-[90vh] flex flex-col overflow-hidden transform scale-95 transition-transform duration-300">
            <div class="px-6 py-4 border-b bg-gray-50 flex justify-between items-center no-print">
                <div>
                    <h3 class="text-xl font-bold text-gray-800">Laporan Penilaian Keseluruhan</h3>
                    <p class="text-sm text-gray-500 font-medium mt-1" id="report-context-subtitle"></p>
                </div>
                <div class="flex gap-3">
                    <button onclick="printReport()" class="bg-primary text-white px-4 py-2 rounded-lg text-sm font-medium hover:bg-blue-800 transition shadow-md flex items-center gap-2">
                        <i class="fas fa-print"></i> Cetak Laporan
                    </button>
                    <button onclick="closeModal('modal-report')" class="bg-white border text-gray-600 px-4 py-2 rounded-lg text-sm font-medium hover:bg-gray-50 transition shadow-sm">
                        Tutup
                    </button>
                </div>
            </div>
            <div class="p-8 flex-1 overflow-y-auto bg-white print-w-full" id="report-print-area">
                <div class="hidden print-only mb-8 text-center">
                    <h1 class="text-2xl font-bold uppercase tracking-wider border-b-2 border-black inline-block pb-1">Laporan Keseluruhan PBD</h1>
                    <h2 class="text-lg font-semibold mt-3">SK BUKIT KUCHAI</h2>
                    <p class="text-md mt-2 text-gray-700 bg-gray-100 p-2 rounded" id="print-report-context"></p>
                </div>
                
                <table class="w-full text-left border-collapse border border-gray-300">
                    <thead>
                        <tr class="bg-gray-100 text-gray-800 text-sm font-bold uppercase tracking-wider border-b-2 border-gray-400">
                            <th class="p-3 border border-gray-300 w-12 text-center">No</th>
                            <th class="p-3 border border-gray-300">Nama Murid</th>
                            <th class="p-3 border border-gray-300 text-center w-40">TP Keseluruhan</th>
                            <th class="p-3 border border-gray-300 text-center w-36 report-mark-col hidden">Markah P.Tahun</th>
                            <th class="p-3 border border-gray-300 text-center w-36 report-mark-col hidden">Markah A.Tahun</th>
                        </tr>
                    </thead>
                    <tbody id="report-table-body" class="text-gray-700 text-sm divide-y divide-gray-200">
                        <!-- Rows injected via JS -->
                    </tbody>
                </table>
            </div>
        </div>
    </div>

    <!-- Loading Overlay -->
    <div id="loading-overlay" class="fixed inset-0 bg-white/90 z-[999] flex flex-col items-center justify-center transition-opacity">
        <i class="fas fa-circle-notch fa-spin text-5xl text-primary mb-4"></i>
        <h2 class="text-xl font-bold text-gray-800 mb-2">Memuatkan Sistem PBD...</h2>
        <p class="text-gray-500 text-sm" id="loading-text">Menyambung ke Pangkalan Data Google Sheets & Awan</p>
    </div>

    <!-- Toast Notification -->
    <div id="toast" class="fixed bottom-5 right-5 transform translate-y-20 opacity-0 transition-all duration-300 bg-gray-800 text-white px-6 py-3 rounded-lg shadow-xl z-50 flex items-center gap-3">
        <i id="toast-icon" class="fas fa-check-circle text-success"></i>
        <span id="toast-msg" class="text-sm font-medium">Berjaya disimpan</span>
    </div>

    <!-- Firebase Logic & Application Engine -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, setDoc, getDoc, collection, getDocs, deleteDoc } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        const appId = typeof __app_id !== 'undefined' ? __app_id : 'pbd-sk-bukit-kuchai-123';
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : { /* Fallback config */ };
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

        let app, db, auth;
        let isFirebaseActive = false;
        let autoSyncTimer = null;

        try {
            app = initializeApp(firebaseConfig);
            db = getFirestore(app);
            auth = getAuth(app);
            isFirebaseActive = true;
        } catch (e) {
            console.error("Firebase init fallback mode.", e);
        }

        window.appState = {
            dbURL: 'https://docs.google.com/spreadsheets/d/e/2PACX-1vRxDf3bKsKRdGOdPvyEas8m3M0b7gR9wqQ-d_Duczf3hc7IuGXc-INuN8pRy9O3oyLrrAhIntul_Spz/pub?gid=531444998&single=true&output=csv',
            rawCSV: [],
            students: [],       // [{ id, name, tahun, kelas, subject }] 
            assessments: {},    // { "studentId_spCode_fasa": tpValue (1-6) }
            marks: {},          // { "studentId_subject": { pertengahan: x, akhir: y } }
            curriculumData: [], // structured curriculum for filters
            activeFilters: { fasa: 'pertengahan', subjek: '', tahun: '', kelas: '', tema: '', bidang: '', sk: '', sp: '', searchQuery: '' },
            chartInstance: null,
            hasUnsavedChanges: false
        };

        // Flexible CSV Header Reader
        function getRowValue(row, keys) {
            if (!row) return '';
            const rowKeys = Object.keys(row);
            for (const key of keys) {
                if (row[key] !== undefined && row[key] !== null && String(row[key]).trim() !== '') {
                    return String(row[key]).trim();
                }
                const foundKey = rowKeys.find(k => k.trim().toLowerCase() === key.toLowerCase());
                if (foundKey && row[foundKey] !== undefined && row[foundKey] !== null && String(row[foundKey]).trim() !== '') {
                    return String(row[foundKey]).trim();
                }
            }
            return '';
        }

        function parseCurriculumAndStudents(csvData) {
            window.appState.curriculumData = csvData.map(row => ({
                subjek: getRowValue(row, ['SUBJEK', 'Subjek', 'MATA PELAJARAN', 'Mata Pelajaran']),
                tahun: getRowValue(row, ['TAHUN', 'Tahun']),
                kelas: getRowValue(row, ['KELAS', 'Kelas']),
                tema: getRowValue(row, ['TEMA', 'Tema', 'KANDUNGAN/TEMA']),
                bidang: getRowValue(row, ['BIDANG', 'Bidang']),
                sk: getRowValue(row, ['STANDARD KANDUNGAN (SK)', 'Standard Kandungan (SK)', 'STANDARD KANDUNGAN', 'SK']),
                sp: getRowValue(row, ['STANDARD PEMBELAJARAN (SP)', 'Standard Pembelajaran (SP)', 'STANDARD PEMBELAJARAN', 'SP']),
            })).filter(r => r.subjek || r.sp || r.sk);

            let studentMap = new Map();
            window.appState.students.forEach(s => studentMap.set(s.id, s));

            csvData.forEach(row => {
                const name = getRowValue(row, ['Nama Murid', 'NAMA MURID', 'NAMA', 'Nama']).toUpperCase();
                const tahun = getRowValue(row, ['TAHUN', 'Tahun']);
                const kelas = getRowValue(row, ['KELAS', 'Kelas']);
                const subject = getRowValue(row, ['SUBJEK', 'Subjek']);

                if(name && (tahun || kelas)) {
                    const id = btoa(encodeURIComponent(`${name}_${tahun}_${kelas}`)).replace(/=/g, '');
                    if(!studentMap.has(id)) {
                        studentMap.set(id, { id, name, tahun, kelas, subject });
                    }
                }
            });

            window.appState.students = Array.from(studentMap.values());
        }

        const UI = {
            showLoading: (text) => { document.getElementById('loading-overlay').classList.remove('hidden', 'opacity-0'); if(text) document.getElementById('loading-text').innerText = text; },
            hideLoading: () => { 
                const l = document.getElementById('loading-overlay');
                l.classList.add('opacity-0'); 
                setTimeout(() => l.classList.add('hidden'), 300);
            },
            toast: (msg, type='success') => {
                const toast = document.getElementById('toast');
                document.getElementById('toast-msg').innerText = msg;
                document.getElementById('toast-icon').className = `fas ${type === 'success' ? 'fa-check-circle text-success' : type === 'error' ? 'fa-exclamation-circle text-danger' : 'fa-info-circle text-secondary'}`;
                toast.classList.remove('translate-y-20', 'opacity-0');
                setTimeout(() => toast.classList.add('translate-y-20', 'opacity-0'), 3000);
            },
            populateSelect: (id, options, selectedValue) => {
                const select = document.getElementById(id);
                select.innerHTML = '';
                const defaultOption = document.createElement('option');
                defaultOption.value = '';
                defaultOption.text = '-- Sila Pilih --';
                select.appendChild(defaultOption);
                
                [...new Set(options)].filter(Boolean).sort().forEach(opt => {
                    const el = document.createElement('option');
                    el.value = opt;
                    el.text = opt;
                    if(opt === selectedValue) el.selected = true;
                    select.appendChild(el);
                });
            }
        };

        async function loadFromFirebase() {
            if (!isFirebaseActive || !auth.currentUser) return;

            try {
                // 1. Load saved students
                const studentsSnap = await getDocs(collection(db, 'artifacts', appId, 'public', 'data', 'skbk_students'));
                if (!studentsSnap.empty) {
                    let studentMap = new Map();
                    window.appState.students.forEach(s => studentMap.set(s.id, s));
                    studentsSnap.forEach(docSnap => {
                        const data = docSnap.data();
                        if (data && data.id) {
                            studentMap.set(data.id, data);
                        }
                    });
                    window.appState.students = Array.from(studentMap.values());
                }

                // 2. Load assessments
                const assessmentsSnap = await getDocs(collection(db, 'artifacts', appId, 'public', 'data', 'skbk_assessments'));
                assessmentsSnap.forEach(docSnap => {
                    const data = docSnap.data();
                    if (data && data.studentId && data.spId && data.fasa && data.tp) {
                        const key = `${data.studentId}_${data.spId}_${data.fasa}`;
                        window.appState.assessments[key] = data.tp;
                    }
                });

                // 3. Load marks
                const marksSnap = await getDocs(collection(db, 'artifacts', appId, 'public', 'data', 'skbk_marks'));
                marksSnap.forEach(docSnap => {
                    const data = docSnap.data();
                    if (data && data.studentId && data.subject) {
                        const key = `${data.studentId}_${data.subject}`;
                        window.appState.marks[key] = {
                            pertengahan: data.pertengahan || '',
                            akhir: data.akhir || ''
                        };
                    }
                });
            } catch (e) {
                console.warn("Ralat memuatkan data dari Firebase:", e);
            }
        }

        async function initializeData() {
            UI.showLoading("Menguruskan Pengesahan Server...");
            
            if (isFirebaseActive) {
                try {
                    if (initialAuthToken) await signInWithCustomToken(auth, initialAuthToken);
                    else await signInAnonymously(auth);
                } catch (e) { console.warn("Auth error", e); }
            }

            UI.showLoading("Membaca Pangkalan Data Kurikulum (Google Sheets)...");
            
            Papa.parse(window.appState.dbURL, {
                download: true,
                header: true,
                complete: async function(results) {
                    window.appState.rawCSV = results.data;
                    parseCurriculumAndStudents(window.appState.rawCSV);

                    if (isFirebaseActive && auth.currentUser) {
                        UI.showLoading("Menyegerakkan Data Disimpan (Awan)...");
                        await loadFromFirebase();
                    }

                    setupFilters();
                    updateSyncTime();
                    startAutoSync();
                    UI.hideLoading();
                },
                error: function(err) {
                    UI.toast("Ralat membaca data CSV", "error");
                    UI.hideLoading();
                }
            });
        }

        window.syncGoogleSheets = async function(isManual = false) {
            if(isManual) UI.showLoading("Mengemas kini data dari Google Sheets...");
            
            Papa.parse(window.appState.dbURL, {
                download: true,
                header: true,
                complete: function(results) {
                    window.appState.rawCSV = results.data;
                    parseCurriculumAndStudents(window.appState.rawCSV);
                    
                    cascadeFilters('subjek'); 
                    refreshView();
                    updateSyncTime();
                    
                    if(isManual) {
                        UI.hideLoading();
                        UI.toast("Pangkalan Data Google Sheets Berjaya Dikemaskini!", "success");
                    }
                },
                error: function(err) {
                    if(isManual) {
                        UI.hideLoading();
                        UI.toast("Ralat memuat turun dari Google Sheets", "error");
                    }
                }
            });
        }

        function startAutoSync() {
            if (autoSyncTimer) clearInterval(autoSyncTimer);
            autoSyncTimer = setInterval(() => {
                window.syncGoogleSheets(false);
            }, 30000);

            document.addEventListener('visibilitychange', () => {
                if (document.visibilityState === 'visible') {
                    window.syncGoogleSheets(false);
                }
            });
        }

        function updateSyncTime() {
            const timeEl = document.getElementById('last-sync-time');
            if (timeEl) {
                const now = new Date();
                timeEl.innerText = now.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
            }
        }

        window.saveDataToFirebase = async function() {
            if(!isFirebaseActive || !auth.currentUser) {
                UI.toast("Mod Luar Talian (Tiada simpanan awan)", "error");
                return;
            }
            
            const btn = document.getElementById('btn-save');
            const originalHTML = btn.innerHTML;
            btn.innerHTML = '<i class="fas fa-spinner fa-spin"></i> Menyimpan...';
            btn.disabled = true;

            try {
                // Save students
                for (const student of window.appState.students) {
                    await setDoc(doc(db, 'artifacts', appId, 'public', 'data', 'skbk_students', student.id), student);
                }

                // Save assessments
                for (const [key, tp] of Object.entries(window.appState.assessments)) {
                    const parts = key.split('_');
                    const fasa = parts.pop();
                    const sId = parts[0];
                    const sp = parts.slice(1).join('_');
                    
                    const docId = btoa(encodeURIComponent(`${sId}_${sp}_${fasa}`)).replace(/=/g, ''); 
                    await setDoc(doc(db, 'artifacts', appId, 'public', 'data', 'skbk_assessments', docId), {
                        studentId: sId,
                        spId: sp,
                        fasa: fasa,
                        tp: tp,
                        timestamp: new Date().getTime()
                    });
                }

                // Save marks
                for (const [key, markData] of Object.entries(window.appState.marks)) {
                    const firstUnderscore = key.indexOf('_');
                    const sId = key.substring(0, firstUnderscore);
                    const subject = key.substring(firstUnderscore + 1);
                    const docId = btoa(encodeURIComponent(`${sId}_${subject}`)).replace(/=/g, '');
                    await setDoc(doc(db, 'artifacts', appId, 'public', 'data', 'skbk_marks', docId), {
                        studentId: sId,
                        subject: subject,
                        pertengahan: markData.pertengahan,
                        akhir: markData.akhir,
                        timestamp: new Date().getTime()
                    });
                }
                
                window.appState.hasUnsavedChanges = false;
                updateSaveButtonState();
                UI.toast("Semua data berjaya disimpan di awan!");
            } catch(e) {
                console.error("Save Error:", e);
                UI.toast("Ralat semasa menyimpan!", "error");
            } finally {
                btn.innerHTML = originalHTML;
                btn.disabled = false;
            }
        }

        function setupFilters() {
            const data = window.appState.curriculumData;
            
            const fixedTahun = ['1', '2', '3', '4', '5', '6'];
            const fixedKelas = ['INOVATIF', 'KREATIF', 'PROAKTIF'];
            
            UI.populateSelect('filter-subjek', data.map(d => d.subjek), window.appState.activeFilters.subjek);
            UI.populateSelect('filter-tahun', fixedTahun, window.appState.activeFilters.tahun);
            UI.populateSelect('filter-kelas', fixedKelas, window.appState.activeFilters.kelas);
            
            UI.populateSelect('add-tahun', fixedTahun);
            UI.populateSelect('add-kelas', fixedKelas);
            UI.populateSelect('bulk-tahun', fixedTahun);
            UI.populateSelect('bulk-kelas', fixedKelas);

            document.getElementById('filter-fasa').addEventListener('change', (e) => {
                window.appState.activeFilters.fasa = e.target.value;
                refreshView();
            });

            ['subjek', 'tahun', 'kelas', 'tema', 'bidang', 'sk', 'sp'].forEach(key => {
                document.getElementById(`filter-${key}`).addEventListener('change', (e) => {
                    window.appState.activeFilters[key] = e.target.value;
                    cascadeFilters(key);
                    refreshView();
                });
            });

            const searchInput = document.getElementById('search-input');
            if (searchInput) {
                searchInput.addEventListener('input', (e) => {
                    window.appState.activeFilters.searchQuery = e.target.value.toLowerCase().trim();
                    refreshView();
                });
            }
            
            initChart();
            refreshView();
        }

        function cascadeFilters(changedKey) {
            const f = window.appState.activeFilters;
            const data = window.appState.curriculumData;
            
            let cascadeList = ['subjek', 'tahun', 'kelas', 'tema', 'bidang', 'sk', 'sp'];
            let idx = cascadeList.indexOf(changedKey);
            
            for(let i = idx + 1; i < cascadeList.length; i++) {
                f[cascadeList[i]] = '';
            }

            let filteredData = data;
            if(f.subjek) filteredData = filteredData.filter(d => d.subjek === f.subjek);
            if(f.tahun) filteredData = filteredData.filter(d => d.tahun === f.tahun);
            if(f.kelas) filteredData = filteredData.filter(d => d.kelas === f.kelas || !d.kelas);
            
            if(idx < 3) UI.populateSelect('filter-tema', filteredData.map(d => d.tema), f.tema);
            if(f.tema) filteredData = filteredData.filter(d => d.tema === f.tema);
            
            if(idx < 4) UI.populateSelect('filter-bidang', filteredData.map(d => d.bidang), f.bidang);
            if(f.bidang) filteredData = filteredData.filter(d => d.bidang === f.bidang);
            
            if(idx < 5) UI.populateSelect('filter-sk', filteredData.map(d => d.sk), f.sk);
            if(f.sk) filteredData = filteredData.filter(d => d.sk === f.sk);
            
            if(idx < 6) UI.populateSelect('filter-sp', filteredData.map(d => d.sp), f.sp);
        }

        function refreshView() {
            const f = window.appState.activeFilters;
            
            let filteredStudents = window.appState.students;
            if (f.tahun) filteredStudents = filteredStudents.filter(s => s.tahun === f.tahun);
            if (f.kelas) filteredStudents = filteredStudents.filter(s => s.kelas === f.kelas);
            if (f.searchQuery) filteredStudents = filteredStudents.filter(s => s.name.toLowerCase().includes(f.searchQuery));
            
            filteredStudents.sort((a, b) => a.name.localeCompare(b.name));

            const tbody = document.getElementById('student-table-body');
            const emptyState = document.getElementById('empty-state');
            const thMarkah = document.getElementById('th-markah');
            tbody.innerHTML = '';

            const isTahap2 = ['4', '5', '6'].includes(f.tahun);
            if (isTahap2 && f.subjek) {
                thMarkah.classList.remove('hidden');
            } else {
                thMarkah.classList.add('hidden');
            }

            if (filteredStudents.length === 0) {
                emptyState.classList.remove('hidden');
            } else {
                emptyState.classList.add('hidden');
                
                filteredStudents.forEach((student, index) => {
                    const row = document.createElement('tr');
                    row.className = "hover:bg-blue-50/50 transition-colors group";
                    
                    const spId = f.sp ? btoa(encodeURIComponent(f.sp)).replace(/=/g, '') : '';
                    const assessmentKey = spId ? `${student.id}_${spId}_${f.fasa}` : null;
                    const currentTP = assessmentKey ? window.appState.assessments[assessmentKey] : null;

                    const markKey = `${student.id}_${f.subjek}`;
                    const marks = window.appState.marks[markKey] || { pertengahan: '', akhir: '' };
                    let markahHTML = '';
                    if (isTahap2 && f.subjek) {
                        const markVal = f.fasa === 'pertengahan' ? marks.pertengahan : marks.akhir;
                        markahHTML = `<td class="p-4 text-center">
                            <input type="number" min="0" max="100" placeholder="Markah" value="${markVal}" onchange="saveMark('${student.id}', '${f.subjek}', '${f.fasa}', this.value)" class="w-16 p-1 border border-gray-300 rounded bg-white text-center font-medium focus:ring-2 focus:ring-primary outline-none no-print shadow-inner">
                            <span class="hidden print-only text-sm font-semibold">${markVal || '-'}</span>
                        </td>`;
                    }

                    let tpHTML = `<div class="flex justify-center gap-1">`;
                    if (f.sp) {
                        for (let i = 1; i <= 6; i++) {
                            const isSelected = currentTP === i;
                            const baseClass = isSelected ? `tp-box bg-tp${i} text-white shadow-inner scale-110 font-bold z-10` : `tp-box bg-gray-100 text-gray-400 hover:bg-gray-200`;
                            tpHTML += `<button onclick="setTP('${student.id}', ${i})" class="w-8 h-8 md:w-9 md:h-9 rounded-lg flex items-center justify-center transition-all ${baseClass} border border-transparent hover:border-gray-300 no-print text-xs md:text-sm shadow-sm">TP${i}</button>`;
                            
                            if(isSelected) {
                                tpHTML += `<div class="hidden print-only text-center font-bold text-sm bg-gray-200 border w-full py-1 rounded">TP${i}</div>`;
                            }
                        }
                    } else {
                        tpHTML = `<span class="text-xs text-red-500 italic flex items-center justify-center h-10 no-print">Sila pilih SP dahulu</span>
                                  <span class="hidden print-only text-center text-xs">-</span>`;
                    }
                    tpHTML += `</div>`;

                    row.innerHTML = `
                        <td class="p-4 text-center text-gray-500">${index + 1}</td>
                        <td class="p-4 font-medium text-gray-800">
                            ${student.name}
                            <div class="text-xs text-gray-400 mt-0.5 print-hidden">${student.tahun} ${student.kelas}</div>
                        </td>
                        <td class="p-4">${tpHTML}</td>
                        ${markahHTML}
                        <td class="p-4 text-center no-print">
                            <button onclick="deleteStudent('${student.id}')" class="text-gray-400 hover:text-danger p-2 rounded-full hover:bg-red-50 transition opacity-0 group-hover:opacity-100">
                                <i class="fas fa-trash"></i>
                            </button>
                        </td>
                    `;
                    tbody.appendChild(row);
                });
            }

            updateStatsAndChart(filteredStudents, f.sp);
            updatePrintContext(f);
        }

        window.setTP = function(studentId, tpValue) {
            const sp = window.appState.activeFilters.sp;
            const fasa = window.appState.activeFilters.fasa;
            if (!sp) {
                UI.toast("Sila pilih Standard Pembelajaran (SP) dahulu!", "warning");
                return;
            }
            
            const spId = btoa(encodeURIComponent(sp)).replace(/=/g, '');
            const key = `${studentId}_${spId}_${fasa}`;
            
            if (window.appState.assessments[key] === tpValue) {
                delete window.appState.assessments[key];
            } else {
                window.appState.assessments[key] = tpValue;
            }
            
            markUnsaved();
            refreshView(); 
        }

        window.saveMark = function(studentId, subject, fasa, value) {
            const key = `${studentId}_${subject}`;
            if (!window.appState.marks[key]) {
                window.appState.marks[key] = { pertengahan: '', akhir: '' };
            }
            window.appState.marks[key][fasa] = value;
            markUnsaved();
        }

        function initChart() {
            const ctx = document.getElementById('tpChart').getContext('2d');
            window.appState.chartInstance = new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: ['Belum Dinilai', 'TP 1', 'TP 2', 'TP 3', 'TP 4', 'TP 5', 'TP 6'],
                    datasets: [{
                        label: 'Bilangan Murid',
                        data: [0, 0, 0, 0, 0, 0, 0],
                        backgroundColor: [
                            '#e5e7eb',
                            '#ef4444',
                            '#f97316',
                            '#eab308',
                            '#84cc16',
                            '#22c55e',
                            '#0ea5e9',
                        ],
                        borderRadius: 6,
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: { display: false }
                    },
                    scales: {
                        y: { beginAtZero: true, ticks: { stepSize: 1 } },
                        x: { grid: { display: false } }
                    }
                }
            });
        }

        function updateStatsAndChart(students, activeSP) {
            let total = students.length;
            let assessed = 0;
            let sumTP = 0;
            let dist = [0, 0, 0, 0, 0, 0, 0];
            const fasa = window.appState.activeFilters.fasa;

            if (activeSP) {
                const spId = btoa(encodeURIComponent(activeSP)).replace(/=/g, '');
                students.forEach(s => {
                    const tp = window.appState.assessments[`${s.id}_${spId}_${fasa}`];
                    if (tp) {
                        assessed++;
                        sumTP += tp;
                        dist[tp]++;
                    } else {
                        dist[0]++;
                    }
                });
            } else {
                dist[0] = total;
            }

            document.getElementById('stat-total').innerText = total;
            document.getElementById('stat-assessed').innerText = assessed;
            document.getElementById('stat-avg').innerText = assessed > 0 ? (sumTP / assessed).toFixed(2) : "0.0";

            if (window.appState.chartInstance) {
                window.appState.chartInstance.data.datasets[0].data = dist;
                window.appState.chartInstance.update();
            }
        }

        function updatePrintContext(f) {
            document.getElementById('print-subjek').innerText = f.subjek || 'Semua Subjek';
            document.getElementById('print-kelas').innerText = (f.tahun || 'Semua Tahun') + ' ' + (f.kelas || '');
            document.getElementById('print-sk').innerText = f.sk || '-';
            document.getElementById('print-sp').innerText = f.sp || '-';
        }

        window.generateReport = function() {
            const f = window.appState.activeFilters;
            if(!f.tahun || !f.subjek) {
                UI.toast("Sila pilih Tahun dan Subjek dari Penapis Utama untuk menjana laporan", "warning");
                return;
            }

            let filteredStudents = window.appState.students.filter(s => s.tahun === f.tahun);
            if(f.kelas) filteredStudents = filteredStudents.filter(s => s.kelas === f.kelas);
            filteredStudents.sort((a, b) => a.name.localeCompare(b.name));

            const isTahap2 = ['4', '5', '6'].includes(f.tahun);
            const markCols = document.querySelectorAll('.report-mark-col');
            markCols.forEach(col => isTahap2 ? col.classList.remove('hidden') : col.classList.add('hidden'));

            const subjectSPs = window.appState.curriculumData.filter(d => d.subjek === f.subjek).map(d => btoa(encodeURIComponent(d.sp)).replace(/=/g, ''));
            const uniqueSPs = [...new Set(subjectSPs)];

            const tbody = document.getElementById('report-table-body');
            tbody.innerHTML = '';

            filteredStudents.forEach((student, index) => {
                let totalTP = 0;
                let countTP = 0;
                
                uniqueSPs.forEach(spId => {
                    const tpPertengahan = window.appState.assessments[`${student.id}_${spId}_pertengahan`] || 0;
                    const tpAkhir = window.appState.assessments[`${student.id}_${spId}_akhir`] || 0;
                    const highest = Math.max(tpPertengahan, tpAkhir);
                    if (highest > 0) {
                        totalTP += highest;
                        countTP++;
                    }
                });

                const overallTP = countTP > 0 ? Math.round(totalTP / countTP) : 'Tiada Data';
                let tpBadge = overallTP !== 'Tiada Data' ? `<span class="bg-tp${overallTP} text-white px-3 py-1 rounded-full text-xs font-bold shadow-sm">TP ${overallTP}</span>` : `<span class="text-gray-400 italic text-xs">-</span>`;

                const markKey = `${student.id}_${f.subjek}`;
                const marks = window.appState.marks[markKey] || { pertengahan: '-', akhir: '-' };
                
                let rowHTML = `
                    <td class="p-3 border border-gray-300 text-center">${index + 1}</td>
                    <td class="p-3 border border-gray-300 font-medium">${student.name} <div class="text-xs text-gray-400 print-hidden">${student.kelas}</div></td>
                    <td class="p-3 border border-gray-300 text-center">${tpBadge}</td>
                `;

                if (isTahap2) {
                    rowHTML += `
                        <td class="p-3 border border-gray-300 text-center font-semibold">${marks.pertengahan || '-'}</td>
                        <td class="p-3 border border-gray-300 text-center font-semibold">${marks.akhir || '-'}</td>
                    `;
                }

                const tr = document.createElement('tr');
                tr.innerHTML = rowHTML;
                tbody.appendChild(tr);
            });

            document.getElementById('report-context-subtitle').innerText = `Mata Pelajaran: ${f.subjek} | Tahun: ${f.tahun} ${f.kelas || ''}`;
            document.getElementById('print-report-context').innerText = `Mata Pelajaran: ${f.subjek} | Tahun: ${f.tahun} ${f.kelas || ''}`;
            
            openModal('modal-report');
        }

        window.printReport = function() {
            document.body.classList.add('printing-report');
            window.print();
            setTimeout(() => {
                document.body.classList.remove('printing-report');
            }, 500);
        }

        window.openModal = function(id) {
            const modal = document.getElementById(id);
            modal.classList.remove('hidden');
            setTimeout(() => {
                modal.classList.remove('opacity-0');
                modal.firstElementChild.classList.remove('scale-95');
            }, 10);
            
            const f = window.appState.activeFilters;
            if(id === 'modal-add-single') {
                if(f.tahun) document.getElementById('add-tahun').value = f.tahun;
                if(f.kelas) document.getElementById('add-kelas').value = f.kelas;
                document.getElementById('add-nama').value = '';
                document.getElementById('add-nama').focus();
            }
            if(id === 'modal-add-bulk') {
                if(f.tahun) document.getElementById('bulk-tahun').value = f.tahun;
                if(f.kelas) document.getElementById('bulk-kelas').value = f.kelas;
                document.getElementById('bulk-nama').value = '';
            }
        }

        window.closeModal = function(id) {
            const modal = document.getElementById(id);
            modal.classList.add('opacity-0');
            modal.firstElementChild.classList.add('scale-95');
            setTimeout(() => {
                modal.classList.add('hidden');
            }, 300);
        }

        window.saveSingleStudent = function() {
            const name = document.getElementById('add-nama').value.trim().toUpperCase();
            const tahun = document.getElementById('add-tahun').value;
            const kelas = document.getElementById('add-kelas').value;

            if (!name || !tahun) {
                UI.toast("Sila masukkan Nama dan Tahun", "warning");
                return;
            }

            const id = btoa(encodeURIComponent(`${name}_${tahun}_${kelas}_${Date.now()}`)).replace(/=/g, '');
            window.appState.students.push({ id, name, tahun, kelas, subject: '' });
            
            markUnsaved();
            closeModal('modal-add-single');
            refreshView();
            UI.toast("Murid ditambah!");
        }

        const bulkNamaInput = document.getElementById('bulk-nama');
        if (bulkNamaInput) {
            bulkNamaInput.addEventListener('input', function(e) {
                const lines = e.target.value.split('\n').filter(l => l.trim().length > 0);
                document.getElementById('bulk-count').innerText = `${lines.length} Murid Sedia Ditambah`;
            });
        }

        window.saveBulkStudents = function() {
            const rawNames = document.getElementById('bulk-nama').value;
            const tahun = document.getElementById('bulk-tahun').value;
            const kelas = document.getElementById('bulk-kelas').value;

            if (!tahun) {
                UI.toast("Sila pilih Tahun", "warning");
                return;
            }

            const names = rawNames.split('\n').map(n => n.trim().toUpperCase()).filter(n => n.length > 0);
            if (names.length === 0) {
                UI.toast("Tiada nama untuk ditambah", "warning");
                return;
            }

            let addedCount = 0;
            names.forEach(name => {
                const id = btoa(encodeURIComponent(`${name}_${tahun}_${kelas}_${Date.now()}_${Math.random()}`)).replace(/=/g, '');
                window.appState.students.push({ id, name, tahun, kelas, subject: '' });
                addedCount++;
            });

            markUnsaved();
            closeModal('modal-add-bulk');
            refreshView();
            UI.toast(`${addedCount} Murid ditambah secara pukal!`);
        }

        window.deleteStudent = function(id) {
            const confirmBox = document.createElement('div');
            confirmBox.className = "fixed inset-0 bg-black/60 z-[100] flex items-center justify-center transition-opacity";
            confirmBox.innerHTML = `
                <div class="bg-white rounded-xl shadow-2xl p-6 w-full max-w-sm transform scale-100 text-center">
                    <div class="text-red-500 text-5xl mb-4"><i class="fas fa-exclamation-circle"></i></div>
                    <h3 class="text-xl font-bold text-gray-800 mb-2">Padam Rekod?</h3>
                    <p class="text-gray-500 text-sm mb-6">Adakah anda pasti untuk memadam murid ini? Semua rekod penilaiannya tidak akan dapat dikembalikan.</p>
                    <div class="flex gap-3 justify-center">
                        <button id="btn-cancel-delete" class="px-4 py-2 bg-gray-100 text-gray-700 rounded-lg hover:bg-gray-200 font-medium">Batal</button>
                        <button id="btn-confirm-delete" class="px-4 py-2 bg-red-500 text-white rounded-lg hover:bg-red-600 font-medium shadow-md">Ya, Padam</button>
                    </div>
                </div>
            `;
            document.body.appendChild(confirmBox);

            document.getElementById('btn-cancel-delete').onclick = () => confirmBox.remove();
            document.getElementById('btn-confirm-delete').onclick = () => {
                window.appState.students = window.appState.students.filter(s => s.id !== id);
                
                for (const key in window.appState.assessments) {
                    if (key.startsWith(`${id}_`)) {
                        delete window.appState.assessments[key];
                    }
                }
                
                markUnsaved();
                refreshView();
                confirmBox.remove();
                UI.toast("Rekod murid dipadam", "success");
            };
        }

        function markUnsaved() {
            window.appState.hasUnsavedChanges = true;
            updateSaveButtonState();
        }

        function updateSaveButtonState() {
            const btn = document.getElementById('btn-save');
            if(window.appState.hasUnsavedChanges) {
                btn.classList.remove('bg-primary', 'hover:bg-blue-800');
                btn.classList.add('bg-warning', 'hover:bg-amber-600');
                btn.innerHTML = '<i class="fas fa-save animate-bounce"></i> Simpan Data*';
            } else {
                btn.classList.add('bg-primary', 'hover:bg-blue-800');
                btn.classList.remove('bg-warning', 'hover:bg-amber-600');
                btn.innerHTML = '<i class="fas fa-check"></i> Tersimpan';
                setTimeout(() => {
                    if(!window.appState.hasUnsavedChanges) {
                        btn.innerHTML = '<i class="fas fa-save"></i> Simpan Data';
                    }
                }, 3000);
            }
        }

        window.addEventListener('beforeunload', function (e) {
            if (window.appState.hasUnsavedChanges) {
                e.preventDefault();
                e.returnValue = '';
            }
        });

        window.toggleStudentDropdown = function(e) {
            if(e) e.stopPropagation();
            const menu = document.getElementById('student-dropdown-menu');
            if(menu) menu.classList.toggle('hidden');
        };

        window.closeStudentDropdown = function() {
            const menu = document.getElementById('student-dropdown-menu');
            if(menu) menu.classList.add('hidden');
        };

        document.addEventListener('click', function(e) {
            const menu = document.getElementById('student-dropdown-menu');
            if(menu && !menu.classList.contains('hidden')) {
                menu.classList.add('hidden');
            }
        });

        document.addEventListener('DOMContentLoaded', () => {
            initializeData();

            const mobileBtn = document.getElementById('mobile-menu');
            const sidebar = document.getElementById('sidebar');
            if(mobileBtn && sidebar) {
                mobileBtn.addEventListener('click', () => {
                    sidebar.classList.toggle('hidden');
                });
            }
        });

    </script>
</body>
</html>
