<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Aplikasi Absensi Digital - SMK RAUDLATUSSALAM</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/papaparse@5.3.0/papaparse.min.js"></script>
    <style>
        body {
            font-family: 'Poppins', sans-serif;
            background-color: #f0f4f8;
        }
        .tab-active {
            background-color: #4f46e5;
            color: white;
        }
        .page {
            display: none;
        }
        .page.active {
            display: block;
        }
        .attendance-btn.selected {
            background-color: #4f46e5;
            color: white;
        }
        .attendance-btn {
            transition: all 0.3s ease;
        }
        .attendance-btn:hover:not(.selected) {
            background-color: #e0e7ff;
        }
        @media print {
            .no-print {
                display: none !important;
            }
            .print-only {
                display: block !important;
            }
        }
        .status-hadir {
            background-color: rgba(34, 197, 94, 0.2);
        }
        .status-izin {
            background-color: rgba(59, 130, 246, 0.2);
        }
        .status-sakit {
            background-color: rgba(234, 179, 8, 0.2);
        }
        .status-absen {
            background-color: rgba(239, 68, 68, 0.2);
        }
        .monthly-table-container {
            overflow-x: auto;
            max-width: 100%;
        }
        .monthly-table th, .monthly-table td {
            min-width: 40px;
            height: 40px;
            text-align: center;
            padding: 8px 4px;
            font-size: 0.75rem;
        }
        .monthly-table th.name-column {
            min-width: 180px;
            text-align: left;
            position: sticky;
            left: 0;
            background-color: white;
            z-index: 10;
        }
        .monthly-table td.name-column {
            text-align: left;
            position: sticky;
            left: 0;
            background-color: white;
            z-index: 10;
        }
        .import-table {
            width: 100%;
            border-collapse: collapse;
        }
        .import-table th, .import-table td {
            border: 1px solid #e5e7eb;
            padding: 8px;
            text-align: left;
        }
        .import-table th {
            background-color: #f9fafb;
        }
        .import-table tr:nth-child(even) {
            background-color: #f9fafb;
        }
        .import-table input {
            width: 100%;
            padding: 6px;
            border: 1px solid #e5e7eb;
            border-radius: 4px;
        }
        .import-table select {
            width: 100%;
            padding: 6px;
            border: 1px solid #e5e7eb;
            border-radius: 4px;
        }
    </style>
</head>
<body class="min-h-screen">
    <div class="container mx-auto px-4 py-8">
        <!-- Header with editable school name and logo -->
        <div class="bg-white rounded-lg shadow-lg p-6 mb-6">
            <div class="flex flex-col md:flex-row items-center justify-between">
                <div class="flex items-center mb-4 md:mb-0">
                    <div class="relative w-16 h-16 mr-4 bg-gray-100 rounded-full flex items-center justify-center overflow-hidden" id="logo-container">
                        <svg id="default-logo" class="w-12 h-12 text-indigo-600" fill="currentColor" viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg">
                            <path d="M10.394 2.08a1 1 0 00-.788 0l-7 3a1 1 0 000 1.84L5.25 8.051a.999.999 0 01.356-.257l4-1.714a1 1 0 11.788 1.838l-2.727 1.17 1.94.831a1 1 0 00.787 0l7-3a1 1 0 000-1.838l-7-3zM3.31 9.397L5 10.12v4.102a8.969 8.969 0 00-1.05-.174 1 1 0 01-.89-.89 11.115 11.115 0 01.25-3.762zM9.3 16.573A9.026 9.026 0 007 14.935v-3.957l1.818.78a3 3 0 002.364 0l5.508-2.361a11.026 11.026 0 01.25 3.762 1 1 0 01-.89.89 8.968 8.968 0 00-5.35 2.524 1 1 0 01-1.4 0zM6 18a1 1 0 001-1v-2.065a8.935 8.935 0 00-2-.712V17a1 1 0 001 1z"></path>
                        </svg>
                        <img id="uploaded-logo" class="hidden w-full h-full object-cover" src="" alt="Logo Sekolah">
                    </div>
                    <div>
                        <h1 id="school-name" class="text-2xl font-bold text-gray-800 cursor-pointer" contenteditable="true">SMK RAUDLATUSSALAM</h1>
                        <p class="text-sm text-gray-500">Aplikasi Absensi Digital</p>
                    </div>
                </div>
                <div class="flex items-center">
                    <label for="logo-upload" class="cursor-pointer bg-indigo-50 hover:bg-indigo-100 text-indigo-600 px-4 py-2 rounded-lg mr-2 text-sm">
                        <svg class="w-4 h-4 inline mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z"></path>
                        </svg>
                        Upload Logo
                    </label>
                    <input id="logo-upload" type="file" accept="image/*" class="hidden">
                </div>
            </div>
        </div>

        <!-- Navigation Tabs -->
        <div class="flex mb-6 bg-white rounded-lg shadow-md overflow-hidden no-print">
            <button id="tab-absensi" class="tab-active flex-1 py-3 px-4 text-center font-medium transition-all duration-200">
                Absensi
            </button>
            <button id="tab-data-siswa" class="flex-1 py-3 px-4 text-center font-medium text-gray-700 hover:bg-gray-50 transition-all duration-200">
                Data Siswa
            </button>
            <button id="tab-rekapitulasi" class="flex-1 py-3 px-4 text-center font-medium text-gray-700 hover:bg-gray-50 transition-all duration-200">
                Rekapitulasi
            </button>
        </div>

        <!-- Page 1: Absensi -->
        <div id="page-absensi" class="page active">
            <div class="bg-white rounded-lg shadow-lg p-6 mb-6">
                <h2 class="text-xl font-semibold mb-4 text-gray-800">Informasi Absensi</h2>
                <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                    <div>
                        <div class="mb-4">
                            <label class="block text-sm font-medium text-gray-700 mb-1">Nama Guru</label>
                            <input type="text" id="teacher-name" value="MUKHAMAD KHOIRUL RISKI S.KOM" class="w-full px-4 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500">
                        </div>
                        <div class="mb-4">
                            <label class="block text-sm font-medium text-gray-700 mb-1">Mata Pelajaran</label>
                            <input type="text" id="subject" value="Produktif" class="w-full px-4 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500">
                        </div>
                    </div>
                    <div>
                        <div class="mb-4">
                            <label class="block text-sm font-medium text-gray-700 mb-1">Kelas</label>
                            <div class="flex space-x-2">
                                <select id="class-level" class="w-1/3 px-4 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500">
                                    <option value="">Pilih Kelas</option>
                                    <option value="10">Kelas 10</option>
                                    <option value="11">Kelas 11</option>
                                    <option value="12">Kelas 12</option>
                                </select>
                                <select id="class-major" class="w-2/3 px-4 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500">
                                    <option value="">Pilih Jurusan</option>
                                    <option value="AKL">AKL</option>
                                    <option value="TKJ">TKJ</option>
                                    <option value="TO">TO</option>
                                </select>
                            </div>
                        </div>
                        <div class="mb-4">
                            <label class="block text-sm font-medium text-gray-700 mb-1">Tanggal</label>
                            <input type="date" id="attendance-date" class="w-full px-4 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500">
                        </div>
                    </div>
                </div>
            </div>

            <div class="bg-white rounded-lg shadow-lg p-6 mb-6">
                <div class="flex justify-between items-center mb-6">
                    <h2 class="text-xl font-semibold text-gray-800">Daftar Kehadiran Siswa</h2>
                    <div class="flex space-x-2">
                        <button id="mark-all-present" class="bg-green-500 hover:bg-green-600 text-white px-4 py-2 rounded-md text-sm font-medium transition-all duration-200">
                            Hadir Semua
                        </button>
                    </div>
                </div>

                <div class="overflow-x-auto">
                    <table class="min-w-full divide-y divide-gray-200">
                        <thead class="bg-gray-50">
                            <tr>
                                <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">No</th>
                                <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Nama Siswa</th>
                                <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">NISN</th>
                                <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Kelas</th>
                                <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Status Kehadiran</th>
                                <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Keterangan</th>
                            </tr>
                        </thead>
                        <tbody id="student-list" class="bg-white divide-y divide-gray-200">
                            <!-- Student rows will be inserted here -->
                        </tbody>
                    </table>
                </div>
            </div>

            <div class="flex justify-end space-x-4 mb-6 no-print">
                <button id="save-attendance" class="bg-indigo-600 hover:bg-indigo-700 text-white px-6 py-2 rounded-md font-medium transition-all duration-200">
                    Simpan Absensi
                </button>
            </div>
        </div>

        <!-- Page 2: Data Siswa -->
        <div id="page-data-siswa" class="page">
            <div class="bg-white rounded-lg shadow-lg p-6 mb-6">
                <div class="flex justify-between items-center mb-6">
                    <h2 class="text-xl font-semibold text-gray-800">Manajemen Data Siswa</h2>
                    <div class="flex space-x-2">
                        <button id="import-data-btn" class="bg-green-600 hover:bg-green-700 text-white px-4 py-2 rounded-md text-sm font-medium transition-all duration-200">
                            <svg class="w-4 h-4 inline mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 16a4 4 0 01-.88-7.903A5 5 0 1115.9 6L16 6a5 5 0 011 9.9M9 19l3 3m0 0l3-3m-3 3V10"></path>
                            </svg>
                            Import Data
                        </button>
                        <button id="add-student-btn" class="bg-indigo-600 hover:bg-indigo-700 text-white px-4 py-2 rounded-md text-sm font-medium transition-all duration-200">
                            <svg class="w-4 h-4 inline mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6"></path>
                            </svg>
                            Tambah Siswa
                        </button>
                    </div>
                </div>

                <div class="mb-4 grid grid-cols-1 md:grid-cols-3 gap-4">
                    <div class="relative">
                        <input type="text" id="search-student" placeholder="Cari siswa..." class="w-full px-4 py-2 pl-10 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500">
                        <svg class="w-5 h-5 text-gray-400 absolute left-3 top-2.5" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"></path>
                        </svg>
                    </div>
                    <div>
                        <select id="filter-class" class="w-full px-4 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500">
                            <option value="">Semua Kelas</option>
                            <option value="10">Kelas 10</option>
                            <option value="11">Kelas 11</option>
                            <option value="12">Kelas 12</option>
                        </select>
                    </div>
                    <div>
                        <select id="filter-major" class="w-full px-4 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500">
                            <option value="">Semua Jurusan</option>
                            <option value="AKL">AKL</option>
                            <option value="TKJ">TKJ</option>
                            <option value="TO">TO</option>
                        </select>
                    </div>
                </div>

                <div class="overflow-x-auto">
                    <table class="min-w-full divide-y divide-gray-200">
                        <thead class="bg-gray-50">
                            <tr>
                                <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">No</th>
                                <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Nama Siswa</th>
                                <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">NISN</th>
                                <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Kelas</th>
                                <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Jurusan</th>
                                <th class="px-6 py-3 text-center text-xs font-medium text-gray-500 uppercase tracking-wider">Aksi</th>
                            </tr>
                        </thead>
                        <tbody id="student-data-list" class="bg-white divide-y divide-gray-200">
                            <!-- Student data rows will be inserted here -->
                        </tbody>
                    </table>
                </div>
            </div>
        </div>

        <!-- Page 3: Rekapitulasi -->
        <div id="page-rekapitulasi" class="page">
            <div class="bg-white rounded-lg shadow-lg p-6 mb-6">
                <div class="flex justify-between items-center mb-6">
                    <h2 class="text-xl font-semibold text-gray-800">Rekapitulasi Kehadiran</h2>
                    <div class="flex space-x-2">
                        <button id="export-pdf" class="bg-red-600 hover:bg-red-700 text-white px-4 py-2 rounded-md text-sm font-medium transition-all duration-200">
                            Ekspor PDF
                        </button>
                        <button id="send-data" class="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded-md text-sm font-medium transition-all duration-200">
                            Kirim Data
                        </button>
                        <button id="reset-data" class="bg-gray-600 hover:bg-gray-700 text-white px-4 py-2 rounded-md text-sm font-medium transition-all duration-200">
                            Reset Data
                        </button>
                    </div>
                </div>

                <div class="mb-6">
                    <div class="flex mb-4">
                        <button id="tab-daily" class="tab-active px-4 py-2 rounded-l-md text-sm font-medium">Harian</button>
                        <button id="tab-monthly" class="px-4 py-2 rounded-r-md text-sm font-medium bg-gray-100 text-gray-700">Bulanan</button>
                    </div>

                    <div id="daily-report" class="report-section">
                        <div class="mb-4 grid grid-cols-1 md:grid-cols-3 gap-4">
                            <div>
                                <label class="block text-sm font-medium text-gray-700 mb-1">Pilih Tanggal</label>
                                <input type="date" id="report-date" class="w-full px-4 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500">
                            </div>
                            <div>
                                <label class="block text-sm font-medium text-gray-700 mb-1">Filter Kelas</label>
                                <select id="report-class" class="w-full px-4 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500">
                                    <option value="">Semua Kelas</option>
                                    <option value="10">Kelas 10</option>
                                    <option value="11">Kelas 11</option>
                                    <option value="12">Kelas 12</option>
                                </select>
                            </div>
                            <div>
                                <label class="block text-sm font-medium text-gray-700 mb-1">Filter Jurusan</label>
                                <select id="report-major" class="w-full px-4 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500">
                                    <option value="">Semua Jurusan</option>
                                    <option value="AKL">AKL</option>
                                    <option value="TKJ">TKJ</option>
                                    <option value="TO">TO</option>
                                </select>
                            </div>
                        </div>
                        <div class="overflow-x-auto mt-4">
                            <table id="daily-table" class="min-w-full divide-y divide-gray-200">
                                <thead class="bg-gray-50">
                                    <tr>
                                        <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">No</th>
                                        <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Nama Siswa</th>
                                        <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">NISN</th>
                                        <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Kelas</th>
                                        <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Status</th>
                                        <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Keterangan</th>
                                    </tr>
                                </thead>
                                <tbody id="daily-report-list" class="bg-white divide-y divide-gray-200">
                                    <!-- Daily report rows will be inserted here -->
                                </tbody>
                            </table>
                        </div>
                    </div>

                    <div id="monthly-report" class="report-section hidden">
                        <div class="mb-4 grid grid-cols-1 md:grid-cols-4 gap-4">
                            <div>
                                <label class="block text-sm font-medium text-gray-700 mb-1">Pilih Bulan</label>
                                <select id="report-month" class="w-full px-4 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500">
                                    <option value="0">Januari</option>
                                    <option value="1">Februari</option>
                                    <option value="2">Maret</option>
                                    <option value="3">April</option>
                                    <option value="4">Mei</option>
                                    <option value="5">Juni</option>
                                    <option value="6">Juli</option>
                                    <option value="7">Agustus</option>
                                    <option value="8">September</option>
                                    <option value="9">Oktober</option>
                                    <option value="10">November</option>
                                    <option value="11">Desember</option>
                                </select>
                            </div>
                            <div>
                                <label class="block text-sm font-medium text-gray-700 mb-1">Pilih Tahun</label>
                                <select id="report-year" class="w-full px-4 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500">
                                    <!-- Years will be dynamically added -->
                                </select>
                            </div>
                            <div>
                                <label class="block text-sm font-medium text-gray-700 mb-1">Filter Kelas</label>
                                <select id="monthly-report-class" class="w-full px-4 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500">
                                    <option value="">Semua Kelas</option>
                                    <option value="10">Kelas 10</option>
                                    <option value="11">Kelas 11</option>
                                    <option value="12">Kelas 12</option>
                                </select>
                            </div>
                            <div>
                                <label class="block text-sm font-medium text-gray-700 mb-1">Filter Jurusan</label>
                                <select id="monthly-report-major" class="w-full px-4 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500">
                                    <option value="">Semua Jurusan</option>
                                    <option value="AKL">AKL</option>
                                    <option value="TKJ">TKJ</option>
                                    <option value="TO">TO</option>
                                </select>
                            </div>
                        </div>
                        <div class="monthly-table-container mt-4">
                            <table id="monthly-table" class="monthly-table min-w-full divide-y divide-gray-200">
                                <thead class="bg-gray-50">
                                    <tr id="monthly-table-header">
                                        <th class="name-column px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Nama Siswa</th>
                                        <!-- Day columns will be added dynamically -->
                                    </tr>
                                </thead>
                                <tbody id="monthly-report-list" class="bg-white divide-y divide-gray-200">
                                    <!-- Monthly report rows will be inserted here -->
                                </tbody>
                            </table>
                        </div>
                        
                        <div class="mt-6">
                            <h3 class="text-lg font-medium mb-2 text-gray-700">Keterangan:</h3>
                            <div class="flex flex-wrap gap-4">
                                <div class="flex items-center">
                                    <div class="w-6 h-6 rounded status-hadir mr-2"></div>
                                    <span class="text-sm">H: Hadir</span>
                                </div>
                                <div class="flex items-center">
                                    <div class="w-6 h-6 rounded status-izin mr-2"></div>
                                    <span class="text-sm">I: Izin</span>
                                </div>
                                <div class="flex items-center">
                                    <div class="w-6 h-6 rounded status-sakit mr-2"></div>
                                    <span class="text-sm">S: Sakit</span>
                                </div>
                                <div class="flex items-center">
                                    <div class="w-6 h-6 rounded status-absen mr-2"></div>
                                    <span class="text-sm">A: Absen</span>
                                </div>
                                <div class="flex items-center">
                                    <div class="w-6 h-6 rounded border border-gray-300 mr-2"></div>
                                    <span class="text-sm">-: Tidak ada data</span>
                                </div>
                            </div>
                        </div>
                        
                        <div class="mt-6">
                            <h3 class="text-lg font-medium mb-2 text-gray-700">Ringkasan Kehadiran</h3>
                            <div class="overflow-x-auto">
                                <table class="min-w-full divide-y divide-gray-200">
                                    <thead class="bg-gray-50">
                                        <tr>
                                            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">No</th>
                                            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Nama Siswa</th>
                                            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">NISN</th>
                                            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Kelas</th>
                                            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Hadir</th>
                                            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Izin</th>
                                            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Sakit</th>
                                            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Absen</th>
                                        </tr>
                                    </thead>
                                    <tbody id="monthly-summary-list" class="bg-white divide-y divide-gray-200">
                                        <!-- Monthly summary rows will be inserted here -->
                                    </tbody>
                                </table>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- PDF Export Template (hidden) -->
        <div id="pdf-template" class="hidden print-only">
            <div class="p-6">
                <div class="flex items-center justify-center mb-6">
                    <div id="pdf-logo" class="w-16 h-16 mr-4"></div>
                    <div class="text-center">
                        <h1 id="pdf-school-name" class="text-2xl font-bold">SMK RAUDLATUSSALAM</h1>
                        <h2 class="text-xl font-semibold">Rekapitulasi Absensi Kehadiran</h2>
                    </div>
                </div>
                
                <div class="mb-4">
                    <p><strong>Guru:</strong> <span id="pdf-teacher"></span></p>
                    <p><strong>Mata Pelajaran:</strong> <span id="pdf-subject"></span></p>
                    <p><strong>Kelas:</strong> <span id="pdf-class"></span></p>
                    <p><strong>Periode:</strong> <span id="pdf-period"></span></p>
                </div>
                
                <table id="pdf-table" class="min-w-full border border-gray-300">
                    <!-- PDF table content will be inserted here -->
                </table>
            </div>
        </div>

        <!-- Add Student Modal -->
        <div id="add-student-modal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center hidden z-50">
            <div class="bg-white rounded-lg p-6 w-full max-w-md">
                <h3 class="text-lg font-semibold mb-4" id="student-modal-title">Tambah Siswa Baru</h3>
                <div class="mb-4">
                    <label class="block text-sm font-medium text-gray-700 mb-1">Nama Siswa</label>
                    <input type="text" id="new-student-name" class="w-full px-4 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500">
                </div>
                <div class="mb-4">
                    <label class="block text-sm font-medium text-gray-700 mb-1">NISN</label>
                    <input type="text" id="new-student-nisn" class="w-full px-4 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500">
                </div>
                <div class="mb-4 grid grid-cols-2 gap-4">
                    <div>
                        <label class="block text-sm font-medium text-gray-700 mb-1">Kelas</label>
                        <select id="new-student-class" class="w-full px-4 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500">
                            <option value="">Pilih Kelas</option>
                            <option value="10">Kelas 10</option>
                            <option value="11">Kelas 11</option>
                            <option value="12">Kelas 12</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-sm font-medium text-gray-700 mb-1">Jurusan</label>
                        <select id="new-student-major" class="w-full px-4 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500">
                            <option value="">Pilih Jurusan</option>
                            <option value="AKL">AKL</option>
                            <option value="TKJ">TKJ</option>
                            <option value="TO">TO</option>
                        </select>
                    </div>
                </div>
                <input type="hidden" id="edit-student-id">
                <div class="flex justify-end space-x-2">
                    <button id="cancel-student-modal" class="px-4 py-2 bg-gray-200 text-gray-700 rounded-md hover:bg-gray-300 transition-all duration-200">Batal</button>
                    <button id="confirm-student-modal" class="px-4 py-2 bg-indigo-600 text-white rounded-md hover:bg-indigo-700 transition-all duration-200">Tambah</button>
                </div>
            </div>
        </div>

        <!-- Import Data Modal -->
        <div id="import-data-modal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center hidden z-50">
            <div class="bg-white rounded-lg p-6 w-full max-w-4xl max-h-[90vh] overflow-y-auto">
                <div class="flex justify-between items-center mb-4">
                    <h3 class="text-lg font-semibold">Import Data Siswa</h3>
                    <button id="close-import-modal" class="text-gray-500 hover:text-gray-700">
                        <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path>
                        </svg>
                    </button>
                </div>
                
                <div class="mb-4">
                    <p class="text-sm text-gray-600 mb-2">Anda dapat menambahkan data siswa secara massal dengan mengisi tabel di bawah ini atau mengimpor file CSV.</p>
                    <div class="flex space-x-2 mb-4">
                        <button id="add-row-btn" class="px-3 py-1 bg-indigo-100 text-indigo-700 rounded-md hover:bg-indigo-200 text-sm">
                            <svg class="w-4 h-4 inline mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6"></path>
                            </svg>
                            Tambah Baris
                        </button>
                        <label for="csv-upload" class="px-3 py-1 bg-green-100 text-green-700 rounded-md hover:bg-green-200 text-sm cursor-pointer">
                            <svg class="w-4 h-4 inline mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 16a4 4 0 01-.88-7.903A5 5 0 1115.9 6L16 6a5 5 0 011 9.9M9 19l3 3m0 0l3-3m-3 3V10"></path>
                            </svg>
                            Import CSV
                        </label>
                        <input id="csv-upload" type="file" accept=".csv" class="hidden">
                        <button id="download-template-btn" class="px-3 py-1 bg-blue-100 text-blue-700 rounded-md hover:bg-blue-200 text-sm">
                            <svg class="w-4 h-4 inline mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4"></path>
                            </svg>
                            Download Template
                        </button>
                    </div>
                </div>
                
                <div class="overflow-x-auto mb-4">
                    <table class="import-table">
                        <thead>
                            <tr>
                                <th>No</th>
                                <th>Nama Siswa</th>
                                <th>NISN</th>
                                <th>Kelas</th>
                                <th>Jurusan</th>
                                <th>Aksi</th>
                            </tr>
                        </thead>
                        <tbody id="import-table-body">
                            <tr>
                                <td>1</td>
                                <td><input type="text" class="student-name" placeholder="Nama Siswa"></td>
                                <td><input type="text" class="student-nisn" placeholder="NISN"></td>
                                <td>
                                    <select class="student-class">
                                        <option value="">Pilih Kelas</option>
                                        <option value="10">Kelas 10</option>
                                        <option value="11">Kelas 11</option>
                                        <option value="12">Kelas 12</option>
                                    </select>
                                </td>
                                <td>
                                    <select class="student-major">
                                        <option value="">Pilih Jurusan</option>
                                        <option value="AKL">AKL</option>
                                        <option value="TKJ">TKJ</option>
                                        <option value="TO">TO</option>
                                    </select>
                                </td>
                                <td>
                                    <button class="delete-row px-2 py-1 bg-red-100 text-red-700 rounded-md hover:bg-red-200">
                                        <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path>
                                        </svg>
                                    </button>
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </div>
                
                <div class="flex justify-end space-x-2">
                    <button id="cancel-import" class="px-4 py-2 bg-gray-200 text-gray-700 rounded-md hover:bg-gray-300 transition-all duration-200">Batal</button>
                    <button id="confirm-import" class="px-4 py-2 bg-green-600 text-white rounded-md hover:bg-green-700 transition-all duration-200">Import Data</button>
                </div>
            </div>
        </div>

        <!-- Confirmation Modal -->
        <div id="confirmation-modal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center hidden z-50">
            <div class="bg-white rounded-lg p-6 w-full max-w-md">
                <h3 class="text-lg font-semibold mb-4">Konfirmasi</h3>
                <p id="confirmation-message" class="mb-6">Apakah Anda yakin ingin menghapus siswa ini?</p>
                <div class="flex justify-end space-x-2">
                    <button id="cancel-confirmation" class="px-4 py-2 bg-gray-200 text-gray-700 rounded-md hover:bg-gray-300 transition-all duration-200">Batal</button>
                    <button id="confirm-delete" class="px-4 py-2 bg-red-600 text-white rounded-md hover:bg-red-700 transition-all duration-200">Hapus</button>
                </div>
            </div>
        </div>

        <!-- Notification Toast -->
        <div id="notification" class="fixed bottom-4 right-4 bg-green-500 text-white px-6 py-3 rounded-md shadow-lg transform translate-y-20 opacity-0 transition-all duration-300 z-50">
            <span id="notification-message">Data berhasil disimpan!</span>
        </div>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', function() {
            // Initialize date input with today's date
            const today = new Date();
            const formattedDate = today.toISOString().split('T')[0];
            document.getElementById('attendance-date').value = formattedDate;
            document.getElementById('report-date').value = formattedDate;
            
            // Initialize year dropdown
            const yearSelect = document.getElementById('report-year');
            const currentYear = today.getFullYear();
            for (let year = currentYear - 5; year <= currentYear + 5; year++) {
                const option = document.createElement('option');
                option.value = year;
                option.textContent = year;
                if (year === currentYear) {
                    option.selected = true;
                }
                yearSelect.appendChild(option);
            }
            
            // Initialize month dropdown
            document.getElementById('report-month').value = today.getMonth();

            // Student data
            const initialStudents = [
                { name: "Amelia Eksa Purianti", nisn: generateNISN(), class: "10", major: "AKL" },
                { name: "Ana Citra Lestari", nisn: generateNISN(), class: "10", major: "AKL" },
                { name: "Andre Maulana", nisn: generateNISN(), class: "10", major: "TKJ" },
                { name: "Aprilia Sofi Andrea", nisn: generateNISN(), class: "10", major: "TKJ" },
                { name: "Aurellia Shera Shakila", nisn: generateNISN(), class: "10", major: "TO" },
                { name: "Dwi Anisa Lailatul Hidayah", nisn: generateNISN(), class: "11", major: "AKL" },
                { name: "Hasan Triasa Fu'adi", nisn: generateNISN(), class: "11", major: "AKL" },
                { name: "Kalyca Nafi Prasetya", nisn: generateNISN(), class: "11", major: "TKJ" },
                { name: "M Fikni Mustofa", nisn: generateNISN(), class: "11", major: "TKJ" },
                { name: "M. Jazuli Ihsanudin", nisn: generateNISN(), class: "11", major: "TO" },
                { name: "Mohammad Faisal Kudhori", nisn: generateNISN(), class: "11", major: "TO" },
                { name: "Muhamad Ali Muzaki", nisn: generateNISN(), class: "12", major: "AKL" },
                { name: "Muhammad Farel Ashraf", nisn: generateNISN(), class: "12", major: "AKL" },
                { name: "Muhammad Hamdi Syafi'i", nisn: generateNISN(), class: "12", major: "TKJ" },
                { name: "Niken Mayasari", nisn: generateNISN(), class: "12", major: "TKJ" },
                { name: "Ninis Rahmawati", nisn: generateNISN(), class: "12", major: "TO" },
                { name: "Nur Azizah", nisn: generateNISN(), class: "12", major: "TO" },
                { name: "Ria Agustianingsih", nisn: generateNISN(), class: "10", major: "AKL" },
                { name: "Siti Aishatun Nafizah", nisn: generateNISN(), class: "10", major: "TKJ" },
                { name: "Siti Munawaroh", nisn: generateNISN(), class: "11", major: "AKL" },
                { name: "Umi Musyawaroh", nisn: generateNISN(), class: "11", major: "TKJ" },
                { name: "Widiya Indah Lestari", nisn: generateNISN(), class: "12", major: "AKL" }
            ];

            // Load students from localStorage or use initial data
            let students = JSON.parse(localStorage.getItem('students')) || initialStudents;
            if (!students.length) {
                students = initialStudents;
                localStorage.setItem('students', JSON.stringify(students));
            }

            // Update old student data format if needed
            students = students.map(student => {
                if (!student.class) {
                    student.class = "10";
                }
                if (!student.major) {
                    student.major = "TKJ";
                }
                return student;
            });
            localStorage.setItem('students', JSON.stringify(students));

            // Load attendance data from localStorage
            let attendanceData = JSON.parse(localStorage.getItem('attendanceData')) || {};

            // Load teacher and subject data from localStorage
            const savedTeacherName = localStorage.getItem('teacherName');
            if (savedTeacherName) {
                document.getElementById('teacher-name').value = savedTeacherName;
            }

            const savedSubject = localStorage.getItem('subject');
            if (savedSubject) {
                document.getElementById('subject').value = savedSubject;
            }

            // Save teacher and subject data when changed
            document.getElementById('teacher-name').addEventListener('change', function() {
                localStorage.setItem('teacherName', this.value);
            });

            document.getElementById('subject').addEventListener('change', function() {
                localStorage.setItem('subject', this.value);
            });

            // Render students table
            renderStudentList();
            renderStudentDataList();

            // Tab navigation
            document.getElementById('tab-absensi').addEventListener('click', function() {
                switchTab('absensi');
            });

            document.getElementById('tab-data-siswa').addEventListener('click', function() {
                switchTab('data-siswa');
                renderStudentDataList();
            });

            document.getElementById('tab-rekapitulasi').addEventListener('click', function() {
                switchTab('rekapitulasi');
                updateReports();
            });

            // Report tabs
            document.getElementById('tab-daily').addEventListener('click', function() {
                document.getElementById('tab-daily').classList.add('tab-active');
                document.getElementById('tab-daily').classList.remove('bg-gray-100', 'text-gray-700');
                document.getElementById('tab-monthly').classList.remove('tab-active');
                document.getElementById('tab-monthly').classList.add('bg-gray-100', 'text-gray-700');
                document.getElementById('daily-report').classList.remove('hidden');
                document.getElementById('monthly-report').classList.add('hidden');
                
                // Save the active tab preference
                localStorage.setItem('activeReportTab', 'daily');
            });

            document.getElementById('tab-monthly').addEventListener('click', function() {
                document.getElementById('tab-monthly').classList.add('tab-active');
                document.getElementById('tab-monthly').classList.remove('bg-gray-100', 'text-gray-700');
                document.getElementById('tab-daily').classList.remove('tab-active');
                document.getElementById('tab-daily').classList.add('bg-gray-100', 'text-gray-700');
                document.getElementById('monthly-report').classList.remove('hidden');
                document.getElementById('daily-report').classList.add('hidden');
                updateMonthlyReport();
                
                // Save the active tab preference
                localStorage.setItem('activeReportTab', 'monthly');
            });

            // Restore active report tab preference
            const activeReportTab = localStorage.getItem('activeReportTab');
            if (activeReportTab === 'monthly') {
                document.getElementById('tab-monthly').click();
            }

            // Logo upload
            document.getElementById('logo-upload').addEventListener('change', function(e) {
                const file = e.target.files[0];
                if (file) {
                    const reader = new FileReader();
                    reader.onload = function(event) {
                        document.getElementById('default-logo').classList.add('hidden');
                        const uploadedLogo = document.getElementById('uploaded-logo');
                        uploadedLogo.src = event.target.result;
                        uploadedLogo.classList.remove('hidden');
                        
                        // Save logo to localStorage
                        localStorage.setItem('schoolLogo', event.target.result);
                    };
                    reader.readAsDataURL(file);
                }
            });

            // Load saved logo if exists
            const savedLogo = localStorage.getItem('schoolLogo');
            if (savedLogo) {
                document.getElementById('default-logo').classList.add('hidden');
                const uploadedLogo = document.getElementById('uploaded-logo');
                uploadedLogo.src = savedLogo;
                uploadedLogo.classList.remove('hidden');
            }

            // School name edit
            document.getElementById('school-name').addEventListener('blur', function() {
                localStorage.setItem('schoolName', this.textContent);
            });

            // Load saved school name if exists
            const savedSchoolName = localStorage.getItem('schoolName');
            if (savedSchoolName) {
                document.getElementById('school-name').textContent = savedSchoolName;
            }

            // Class and major selection for attendance
            document.getElementById('class-level').addEventListener('change', function() {
                filterStudentsForAttendance();
                // Save the selected class level
                localStorage.setItem('selectedClassLevel', this.value);
            });

            document.getElementById('class-major').addEventListener('change', function() {
                filterStudentsForAttendance();
                // Save the selected class major
                localStorage.setItem('selectedClassMajor', this.value);
            });

            // Load saved class and major selections
            const savedClassLevel = localStorage.getItem('selectedClassLevel');
            if (savedClassLevel) {
                document.getElementById('class-level').value = savedClassLevel;
            }

            const savedClassMajor = localStorage.getItem('selectedClassMajor');
            if (savedClassMajor) {
                document.getElementById('class-major').value = savedClassMajor;
            }

            // Apply saved filters on load
            filterStudentsForAttendance();

            // Mark all students as present
            document.getElementById('mark-all-present').addEventListener('click', function() {
                const attendanceButtons = document.querySelectorAll('.attendance-btn[data-status="hadir"]');
                attendanceButtons.forEach(button => {
                    const row = button.closest('tr');
                    selectAttendance(row, 'hadir');
                });
                showNotification('Semua siswa ditandai hadir!');
            });

            // Add student button (from Data Siswa page)
            document.getElementById('add-student-btn').addEventListener('click', function() {
                openAddStudentModal();
            });

            // Import data button
            document.getElementById('import-data-btn').addEventListener('click', function() {
                openImportDataModal();
            });

            // Add row button in import modal
            document.getElementById('add-row-btn').addEventListener('click', function() {
                addImportTableRow();
            });

            // Download template button
            document.getElementById('download-template-btn').addEventListener('click', function() {
                downloadCSVTemplate();
            });

            // CSV upload
            document.getElementById('csv-upload').addEventListener('change', function(e) {
                const file = e.target.files[0];
                if (file) {
                    parseCSV(file);
                }
            });

            // Close import modal
            document.getElementById('close-import-modal').addEventListener('click', function() {
                closeImportDataModal();
            });

            // Cancel import
            document.getElementById('cancel-import').addEventListener('click', function() {
                closeImportDataModal();
            });

            // Confirm import
            document.getElementById('confirm-import').addEventListener('click', function() {
                importStudentData();
            });

            // Cancel student modal
            document.getElementById('cancel-student-modal').addEventListener('click', function() {
                closeStudentModal();
            });

            // Confirm student modal (add or edit)
            document.getElementById('confirm-student-modal').addEventListener('click', function() {
                const name = document.getElementById('new-student-name').value.trim();
                let nisn = document.getElementById('new-student-nisn').value.trim();
                const studentClass = document.getElementById('new-student-class').value;
                const major = document.getElementById('new-student-major').value;
                const editId = document.getElementById('edit-student-id').value;
                
                if (!name) {
                    showNotification('Nama siswa tidak boleh kosong!', 'error');
                    return;
                }
                
                if (!nisn) {
                    nisn = generateNISN();
                }
                
                if (!studentClass) {
                    showNotification('Kelas tidak boleh kosong!', 'error');
                    return;
                }
                
                if (!major) {
                    showNotification('Jurusan tidak boleh kosong!', 'error');
                    return;
                }
                
                if (editId) {
                    // Edit existing student
                    const index = parseInt(editId);
                    students[index] = { name, nisn, class: studentClass, major };
                    showNotification('Data siswa berhasil diperbarui!');
                } else {
                    // Add new student
                    students.push({ name, nisn, class: studentClass, major });
                    showNotification('Siswa baru berhasil ditambahkan!');
                }
                
                localStorage.setItem('students', JSON.stringify(students));
                renderStudentList();
                renderStudentDataList();
                closeStudentModal();
            });

            // Search student
            document.getElementById('search-student').addEventListener('input', function() {
                filterStudentDataList();
                // Save search term
                localStorage.setItem('searchStudentTerm', this.value);
            });

            // Load saved search term
            const savedSearchTerm = localStorage.getItem('searchStudentTerm');
            if (savedSearchTerm) {
                document.getElementById('search-student').value = savedSearchTerm;
            }

            // Filter students by class and major
            document.getElementById('filter-class').addEventListener('change', function() {
                filterStudentDataList();
                // Save filter
                localStorage.setItem('filterClass', this.value);
            });

            document.getElementById('filter-major').addEventListener('change', function() {
                filterStudentDataList();
                // Save filter
                localStorage.setItem('filterMajor', this.value);
            });

            // Load saved filters
            const savedFilterClass = localStorage.getItem('filterClass');
            if (savedFilterClass) {
                document.getElementById('filter-class').value = savedFilterClass;
            }

            const savedFilterMajor = localStorage.getItem('filterMajor');
            if (savedFilterMajor) {
                document.getElementById('filter-major').value = savedFilterMajor;
            }

            // Apply saved filters on load
            filterStudentDataList();

            // Report filters
            document.getElementById('report-class').addEventListener('change', function() {
                updateDailyReport();
                // Save filter
                localStorage.setItem('reportClass', this.value);
            });

            document.getElementById('report-major').addEventListener('change', function() {
                updateDailyReport();
                // Save filter
                localStorage.setItem('reportMajor', this.value);
            });

            document.getElementById('monthly-report-class').addEventListener('change', function() {
                updateMonthlyReport();
                // Save filter
                localStorage.setItem('monthlyReportClass', this.value);
            });

            document.getElementById('monthly-report-major').addEventListener('change', function() {
                updateMonthlyReport();
                // Save filter
                localStorage.setItem('monthlyReportMajor', this.value);
            });

            // Load saved report filters
            const savedReportClass = localStorage.getItem('reportClass');
            if (savedReportClass) {
                document.getElementById('report-class').value = savedReportClass;
            }

            const savedReportMajor = localStorage.getItem('reportMajor');
            if (savedReportMajor) {
                document.getElementById('report-major').value = savedReportMajor;
            }

            const savedMonthlyReportClass = localStorage.getItem('monthlyReportClass');
            if (savedMonthlyReportClass) {
                document.getElementById('monthly-report-class').value = savedMonthlyReportClass;
            }

            const savedMonthlyReportMajor = localStorage.getItem('monthlyReportMajor');
            if (savedMonthlyReportMajor) {
                document.getElementById('monthly-report-major').value = savedMonthlyReportMajor;
            }

            // Save attendance
            document.getElementById('save-attendance').addEventListener('click', function() {
                const date = document.getElementById('attendance-date').value;
                const teacherName = document.getElementById('teacher-name').value;
                const subject = document.getElementById('subject').value;
                const classLevel = document.getElementById('class-level').value;
                const classMajor = document.getElementById('class-major').value;
                const className = classLevel && classMajor ? `${classLevel} ${classMajor}` : '';
                
                if (!date) {
                    showNotification('Tanggal tidak boleh kosong!', 'error');
                    return;
                }
                
                if (!classLevel || !classMajor) {
                    showNotification('Kelas dan jurusan tidak boleh kosong!', 'error');
                    return;
                }
                
                const attendanceRows = document.querySelectorAll('#student-list tr');
                const dailyData = [];
                
                attendanceRows.forEach(row => {
                    const studentId = row.getAttribute('data-id');
                    const status = row.getAttribute('data-status') || 'absen';
                    const notes = row.querySelector('.notes-input').value;
                    
                    dailyData.push({
                        studentId,
                        name: students[studentId].name,
                        nisn: students[studentId].nisn,
                        class: students[studentId].class,
                        major: students[studentId].major,
                        status,
                        notes
                    });
                });
                
                if (!attendanceData[date]) {
                    attendanceData[date] = {};
                }
                
                attendanceData[date] = {
                    teacherName,
                    subject,
                    className,
                    classLevel,
                    classMajor,
                    students: dailyData
                };
                
                localStorage.setItem('attendanceData', JSON.stringify(attendanceData));
                showNotification('Data absensi berhasil disimpan!');
                
                // Update reports
                updateReports();
            });

            // Export to PDF
            document.getElementById('export-pdf').addEventListener('click', function() {
                const { jsPDF } = window.jspdf;
                
                // Prepare PDF content
                const pdfSchoolName = document.getElementById('school-name').textContent;
                document.getElementById('pdf-school-name').textContent = pdfSchoolName;
                
                const teacherName = document.getElementById('teacher-name').value;
                document.getElementById('pdf-teacher').textContent = teacherName;
                
                const subject = document.getElementById('subject').value;
                document.getElementById('pdf-subject').textContent = subject;
                
                // Get class info
                let classInfo = "";
                if (document.getElementById('daily-report').classList.contains('hidden')) {
                    // Monthly report
                    const classFilter = document.getElementById('monthly-report-class').value;
                    const majorFilter = document.getElementById('monthly-report-major').value;
                    
                    if (classFilter && majorFilter) {
                        classInfo = `Kelas ${classFilter} ${majorFilter}`;
                    } else if (classFilter) {
                        classInfo = `Kelas ${classFilter}`;
                    } else if (majorFilter) {
                        classInfo = `Jurusan ${majorFilter}`;
                    } else {
                        classInfo = "Semua Kelas";
                    }
                } else {
                    // Daily report
                    const classFilter = document.getElementById('report-class').value;
                    const majorFilter = document.getElementById('report-major').value;
                    
                    if (classFilter && majorFilter) {
                        classInfo = `Kelas ${classFilter} ${majorFilter}`;
                    } else if (classFilter) {
                        classInfo = `Kelas ${classFilter}`;
                    } else if (majorFilter) {
                        classInfo = `Jurusan ${majorFilter}`;
                    } else {
                        classInfo = "Semua Kelas";
                    }
                }
                
                document.getElementById('pdf-class').textContent = classInfo;
                
                // Determine which report is active
                const isDailyReport = !document.getElementById('daily-report').classList.contains('hidden');
                let tableHTML, periodText;
                
                if (isDailyReport) {
                    const reportDate = document.getElementById('report-date').value;
                    const formattedDate = new Date(reportDate).toLocaleDateString('id-ID', { 
                        day: 'numeric', month: 'long', year: 'numeric' 
                    });
                    periodText = formattedDate;
                    tableHTML = document.getElementById('daily-table').outerHTML;
                } else {
                    const monthIndex = parseInt(document.getElementById('report-month').value);
                    const year = document.getElementById('report-year').value;
                    const monthName = new Date(year, monthIndex, 1).toLocaleDateString('id-ID', { month: 'long' });
                    periodText = `${monthName} ${year}`;
                    
                    // For monthly report, use the summary table instead of the daily attendance grid
                    tableHTML = document.querySelector('#monthly-summary-list').closest('table').outerHTML;
                }
                
                document.getElementById('pdf-period').textContent = periodText;
                
                // Set logo
                const pdfLogo = document.getElementById('pdf-logo');
                if (localStorage.getItem('schoolLogo')) {
                    pdfLogo.innerHTML = `<img src="${localStorage.getItem('schoolLogo')}" class="w-full h-full object-contain">`;
                } else {
                    pdfLogo.innerHTML = document.getElementById('default-logo').outerHTML;
                }
                
                // Set table
                document.getElementById('pdf-table').outerHTML = tableHTML;
                
                // Generate PDF
                const pdfTemplate = document.getElementById('pdf-template');
                pdfTemplate.classList.remove('hidden');
                
                html2canvas(pdfTemplate, {
                    scale: 2,
                    useCORS: true,
                    logging: false
                }).then(canvas => {
                    const imgData = canvas.toDataURL('image/png');
                    const pdf = new jsPDF('p', 'mm', 'a4');
                    const imgWidth = 210;
                    const imgHeight = canvas.height * imgWidth / canvas.width;
                    
                    pdf.addImage(imgData, 'PNG', 0, 0, imgWidth, imgHeight);
                    pdf.save(`Rekapitulasi_Absensi_${periodText.replace(/ /g, '_')}.pdf`);
                    
                    pdfTemplate.classList.add('hidden');
                });
                
                showNotification('Ekspor PDF berhasil!');
            });

            // Send data button (simulation)
            document.getElementById('send-data').addEventListener('click', function() {
                showNotification('Data berhasil dikirim!');
            });

            // Reset data
            document.getElementById('reset-data').addEventListener('click', function() {
                if (confirm('Apakah Anda yakin ingin menghapus semua data absensi? Tindakan ini tidak dapat dibatalkan.')) {
                    attendanceData = {};
                    localStorage.setItem('attendanceData', JSON.stringify(attendanceData));
                    updateReports();
                    showNotification('Data absensi berhasil direset!');
                }
            });

            // Report date change
            document.getElementById('report-date').addEventListener('change', function() {
                updateDailyReport();
                // Save selected date
                localStorage.setItem('reportDate', this.value);
            });

            // Load saved report date
            const savedReportDate = localStorage.getItem('reportDate');
            if (savedReportDate) {
                document.getElementById('report-date').value = savedReportDate;
            }

            // Report month/year change
            document.getElementById('report-month').addEventListener('change', function() {
                updateMonthlyReport();
                // Save selected month
                localStorage.setItem('reportMonth', this.value);
            });
            
            document.getElementById('report-year').addEventListener('change', function() {
                updateMonthlyReport();
                // Save selected year
                localStorage.setItem('reportYear', this.value);
            });

            // Load saved report month/year
            const savedReportMonth = localStorage.getItem('reportMonth');
            if (savedReportMonth) {
                document.getElementById('report-month').value = savedReportMonth;
            }

            const savedReportYear = localStorage.getItem('reportYear');
            if (savedReportYear) {
                document.getElementById('report-year').value = savedReportYear;
            }

            // Confirmation modal events
            document.getElementById('cancel-confirmation').addEventListener('click', function() {
                document.getElementById('confirmation-modal').classList.add('hidden');
            });

            document.getElementById('confirm-delete').addEventListener('click', function() {
                const studentId = document.getElementById('confirm-delete').getAttribute('data-id');
                if (studentId) {
                    deleteStudent(parseInt(studentId));
                }
                document.getElementById('confirmation-modal').classList.add('hidden');
            });

            // Initialize reports
            updateReports();

            // Set up event delegation for delete row buttons in import table
            document.getElementById('import-table-body').addEventListener('click', function(e) {
                if (e.target.closest('.delete-row')) {
                    const row = e.target.closest('tr');
                    if (row && document.querySelectorAll('#import-table-body tr').length > 1) {
                        row.remove();
                        updateImportTableNumbers();
                    } else {
                        showNotification('Minimal harus ada satu baris!', 'error');
                    }
                }
            });

            // Save active tab to localStorage
            function saveActiveTab(tabName) {
                localStorage.setItem('activeTab', tabName);
            }

            // Load active tab from localStorage
            const savedActiveTab = localStorage.getItem('activeTab');
            if (savedActiveTab) {
                switchTab(savedActiveTab);
            }

            // Functions
            function generateNISN() {
                return Math.floor(1000000000 + Math.random() * 9000000000).toString();
            }

            function renderStudentList() {
                const studentList = document.getElementById('student-list');
                studentList.innerHTML = '';
                
                const classLevel = document.getElementById('class-level').value;
                const classMajor = document.getElementById('class-major').value;
                
                let filteredStudents = students;
                
                if (classLevel && classMajor) {
                    filteredStudents = students.filter(student => 
                        student.class === classLevel && student.major === classMajor
                    );
                } else if (classLevel) {
                    filteredStudents = students.filter(student => student.class === classLevel);
                } else if (classMajor) {
                    filteredStudents = students.filter(student => student.major === classMajor);
                }
                
                if (filteredStudents.length === 0) {
                    const row = document.createElement('tr');
                    row.innerHTML = `
                        <td colspan="6" class="px-6 py-4 text-center text-sm text-gray-500">Tidak ada siswa yang ditemukan untuk kelas ini</td>
                    `;
                    studentList.appendChild(row);
                    return;
                }
                
                filteredStudents.forEach((student, index) => {
                    const actualIndex = students.findIndex(s => s.name === student.name && s.nisn === student.nisn);
                    const row = document.createElement('tr');
                    row.setAttribute('data-id', actualIndex);
                    
                    row.innerHTML = `
                        <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">${index + 1}</td>
                        <td class="px-6 py-4 whitespace-nowrap">
                            <div class="text-sm font-medium text-gray-900">${student.name}</div>
                        </td>
                        <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">${student.nisn}</td>
                        <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">Kelas ${student.class} ${student.major}</td>
                        <td class="px-6 py-4 whitespace-nowrap">
                            <div class="flex space-x-2">
                                <button class="attendance-btn px-3 py-1 rounded-md text-xs font-medium border" data-status="hadir">Hadir</button>
                                <button class="attendance-btn px-3 py-1 rounded-md text-xs font-medium border" data-status="izin">Izin</button>
                                <button class="attendance-btn px-3 py-1 rounded-md text-xs font-medium border" data-status="sakit">Sakit</button>
                                <button class="attendance-btn px-3 py-1 rounded-md text-xs font-medium border" data-status="absen">Absen</button>
                            </div>
                        </td>
                        <td class="px-6 py-4 whitespace-nowrap">
                            <input type="text" class="notes-input px-3 py-1 border border-gray-300 rounded-md text-sm w-full" placeholder="Keterangan">
                        </td>
                    `;
                    
                    studentList.appendChild(row);
                    
                    // Add event listeners to attendance buttons
                    const attendanceButtons = row.querySelectorAll('.attendance-btn');
                    attendanceButtons.forEach(button => {
                        button.addEventListener('click', function() {
                            const status = this.getAttribute('data-status');
                            selectAttendance(row, status);
                        });
                    });
                });

                // Check if there's saved attendance data for today
                const date = document.getElementById('attendance-date').value;
                if (attendanceData[date]) {
                    // Pre-fill attendance status from saved data
                    const savedAttendance = attendanceData[date].students;
                    
                    savedAttendance.forEach(record => {
                        const studentRow = document.querySelector(`tr[data-id="${record.studentId}"]`);
                        if (studentRow) {
                            selectAttendance(studentRow, record.status);
                            studentRow.querySelector('.notes-input').value = record.notes || '';
                        }
                    });
                }
            }

            function filterStudentsForAttendance() {
                renderStudentList();
            }

            function renderStudentDataList(searchTerm = '') {
                filterStudentDataList();
            }

            function filterStudentDataList() {
                const searchTerm = document.getElementById('search-student').value.toLowerCase();
                const classFilter = document.getElementById('filter-class').value;
                const majorFilter = document.getElementById('filter-major').value;
                
                const studentDataList = document.getElementById('student-data-list');
                studentDataList.innerHTML = '';
                
                let filteredStudents = students;
                
                // Apply filters
                if (searchTerm) {
                    filteredStudents = filteredStudents.filter(student => 
                        student.name.toLowerCase().includes(searchTerm) || 
                        student.nisn.includes(searchTerm)
                    );
                }
                
                if (classFilter) {
                    filteredStudents = filteredStudents.filter(student => student.class === classFilter);
                }
                
                if (majorFilter) {
                    filteredStudents = filteredStudents.filter(student => student.major === majorFilter);
                }
                
                if (filteredStudents.length === 0) {
                    const row = document.createElement('tr');
                    row.innerHTML = `
                        <td colspan="6" class="px-6 py-4 text-center text-sm text-gray-500">Tidak ada data siswa yang ditemukan</td>
                    `;
                    studentDataList.appendChild(row);
                    return;
                }
                
                filteredStudents.forEach((student, index) => {
                    const actualIndex = students.findIndex(s => s.name === student.name && s.nisn === student.nisn);
                    const row = document.createElement('tr');
                    
                    row.innerHTML = `
                        <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">${index + 1}</td>
                        <td class="px-6 py-4 whitespace-nowrap">
                            <div class="text-sm font-medium text-gray-900">${student.name}</div>
                        </td>
                        <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">${student.nisn}</td>
                        <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">Kelas ${student.class}</td>
                        <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">${student.major}</td>
                        <td class="px-6 py-4 whitespace-nowrap text-right text-sm font-medium">
                            <div class="flex justify-center space-x-2">
                                <button class="edit-student px-3 py-1 bg-blue-100 text-blue-700 rounded-md hover:bg-blue-200" data-id="${actualIndex}">
                                    <svg class="w-4 h-4 inline" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15.232 5.232l3.536 3.536m-2.036-5.036a2.5 2.5 0 113.536 3.536L6.5 21.036H3v-3.572L16.732 3.732z"></path>
                                    </svg>
                                    Edit
                                </button>
                                <button class="delete-student px-3 py-1 bg-red-100 text-red-700 rounded-md hover:bg-red-200" data-id="${actualIndex}">
                                    <svg class="w-4 h-4 inline" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path>
                                    </svg>
                                    Hapus
                                </button>
                            </div>
                        </td>
                    `;
                    
                    studentDataList.appendChild(row);
                    
                    // Add event listener to edit button
                    const editButton = row.querySelector('.edit-student');
                    editButton.addEventListener('click', function() {
                        const studentId = this.getAttribute('data-id');
                        openEditStudentModal(parseInt(studentId));
                    });
                    
                    // Add event listener to delete button
                    const deleteButton = row.querySelector('.delete-student');
                    deleteButton.addEventListener('click', function() {
                        const studentId = this.getAttribute('data-id');
                        openDeleteConfirmation(parseInt(studentId));
                    });
                });
            }

            function selectAttendance(row, status) {
                row.setAttribute('data-status', status);
                
                const buttons = row.querySelectorAll('.attendance-btn');
                buttons.forEach(btn => {
                    btn.classList.remove('selected');
                    if (btn.getAttribute('data-status') === status) {
                        btn.classList.add('selected');
                    }
                });
                
                // Set colors based on status
                const statusColors = {
                    'hadir': 'bg-green-100 text-green-800',
                    'izin': 'bg-blue-100 text-blue-800',
                    'sakit': 'bg-yellow-100 text-yellow-800',
                    'absen': 'bg-red-100 text-red-800'
                };
                
                buttons.forEach(btn => {
                    const btnStatus = btn.getAttribute('data-status');
                    btn.className = 'attendance-btn px-3 py-1 rounded-md text-xs font-medium border';
                    
                    if (btnStatus === status) {
                        btn.classList.add('selected', ...statusColors[status].split(' '));
                    }
                });
            }

            function switchTab(tabName) {
                // Update tab buttons
                document.getElementById('tab-absensi').classList.remove('tab-active');
                document.getElementById('tab-data-siswa').classList.remove('tab-active');
                document.getElementById('tab-rekapitulasi').classList.remove('tab-active');
                
                document.getElementById(`tab-${tabName}`).classList.add('tab-active');
                
                // Show/hide pages
                document.getElementById('page-absensi').classList.remove('active');
                document.getElementById('page-data-siswa').classList.remove('active');
                document.getElementById('page-rekapitulasi').classList.remove('active');
                
                document.getElementById(`page-${tabName}`).classList.add('active');
                
                // Save active tab to localStorage
                saveActiveTab(tabName);
            }

            function openAddStudentModal() {
                document.getElementById('student-modal-title').textContent = 'Tambah Siswa Baru';
                document.getElementById('new-student-name').value = '';
                document.getElementById('new-student-nisn').value = '';
                document.getElementById('new-student-class').value = '';
                document.getElementById('new-student-major').value = '';
                document.getElementById('edit-student-id').value = '';
                document.getElementById('confirm-student-modal').textContent = 'Tambah';
                document.getElementById('add-student-modal').classList.remove('hidden');
            }

            function openEditStudentModal(studentId) {
                const student = students[studentId];
                document.getElementById('student-modal-title').textContent = 'Edit Data Siswa';
                document.getElementById('new-student-name').value = student.name;
                document.getElementById('new-student-nisn').value = student.nisn;
                document.getElementById('new-student-class').value = student.class;
                document.getElementById('new-student-major').value = student.major;
                document.getElementById('edit-student-id').value = studentId;
                document.getElementById('confirm-student-modal').textContent = 'Simpan';
                document.getElementById('add-student-modal').classList.remove('hidden');
            }

            function closeStudentModal() {
                document.getElementById('add-student-modal').classList.add('hidden');
                document.getElementById('new-student-name').value = '';
                document.getElementById('new-student-nisn').value = '';
                document.getElementById('new-student-class').value = '';
                document.getElementById('new-student-major').value = '';
                document.getElementById('edit-student-id').value = '';
            }

            function openImportDataModal() {
                // Clear existing rows except the first one
                const importTableBody = document.getElementById('import-table-body');
                importTableBody.innerHTML = '';
                
                // Add a single empty row
                addImportTableRow();
                
                document.getElementById('import-data-modal').classList.remove('hidden');
            }

            function closeImportDataModal() {
                document.getElementById('import-data-modal').classList.add('hidden');
                document.getElementById('csv-upload').value = '';
            }

            function addImportTableRow() {
                const importTableBody = document.getElementById('import-table-body');
                const rowCount = importTableBody.querySelectorAll('tr').length;
                
                const newRow = document.createElement('tr');
                newRow.innerHTML = `
                    <td>${rowCount + 1}</td>
                    <td><input type="text" class="student-name" placeholder="Nama Siswa"></td>
                    <td><input type="text" class="student-nisn" placeholder="NISN"></td>
                    <td>
                        <select class="student-class">
                            <option value="">Pilih Kelas</option>
                            <option value="10">Kelas 10</option>
                            <option value="11">Kelas 11</option>
                            <option value="12">Kelas 12</option>
                        </select>
                    </td>
                    <td>
                        <select class="student-major">
                            <option value="">Pilih Jurusan</option>
                            <option value="AKL">AKL</option>
                            <option value="TKJ">TKJ</option>
                            <option value="TO">TO</option>
                        </select>
                    </td>
                    <td>
                        <button class="delete-row px-2 py-1 bg-red-100 text-red-700 rounded-md hover:bg-red-200">
                            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path>
                            </svg>
                        </button>
                    </td>
                `;
                
                importTableBody.appendChild(newRow);
            }

            function updateImportTableNumbers() {
                const rows = document.querySelectorAll('#import-table-body tr');
                rows.forEach((row, index) => {
                    row.cells[0].textContent = index + 1;
                });
            }

            function downloadCSVTemplate() {
                const headers = ['Nama Siswa', 'NISN', 'Kelas', 'Jurusan'];
                const sampleData = [
                    ['Nama Siswa 1', '1234567890', '10', 'TKJ'],
                    ['Nama Siswa 2', '0987654321', '11', 'AKL']
                ];
                
                let csvContent = headers.join(',') + '\n';
                sampleData.forEach(row => {
                    csvContent += row.join(',') + '\n';
                });
                
                const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
                const link = document.createElement('a');
                const url = URL.createObjectURL(blob);
                
                link.setAttribute('href', url);
                link.setAttribute('download', 'template_data_siswa.csv');
                link.style.visibility = 'hidden';
                
                document.body.appendChild(link);
                link.click();
                document.body.removeChild(link);
            }

            function parseCSV(file) {
                Papa.parse(file, {
                    header: true,
                    skipEmptyLines: true,
                    complete: function(results) {
                        if (results.data && results.data.length > 0) {
                            populateImportTable(results.data);
                        } else {
                            showNotification('File CSV kosong atau tidak valid!', 'error');
                        }
                    },
                    error: function(error) {
                        showNotification('Error saat membaca file CSV: ' + error, 'error');
                    }
                });
            }

            function populateImportTable(data) {
                const importTableBody = document.getElementById('import-table-body');
                importTableBody.innerHTML = '';
                
                data.forEach((row, index) => {
                    const name = row['Nama Siswa'] || '';
                    const nisn = row['NISN'] || '';
                    const studentClass = row['Kelas'] || '';
                    const major = row['Jurusan'] || '';
                    
                    const newRow = document.createElement('tr');
                    newRow.innerHTML = `
                        <td>${index + 1}</td>
                        <td><input type="text" class="student-name" placeholder="Nama Siswa" value="${name}"></td>
                        <td><input type="text" class="student-nisn" placeholder="NISN" value="${nisn}"></td>
                        <td>
                            <select class="student-class">
                                <option value="">Pilih Kelas</option>
                                <option value="10" ${studentClass === '10' ? 'selected' : ''}>Kelas 10</option>
                                <option value="11" ${studentClass === '11' ? 'selected' : ''}>Kelas 11</option>
                                <option value="12" ${studentClass === '12' ? 'selected' : ''}>Kelas 12</option>
                            </select>
                        </td>
                        <td>
                            <select class="student-major">
                                <option value="">Pilih Jurusan</option>
                                <option value="AKL" ${major === 'AKL' ? 'selected' : ''}>AKL</option>
                                <option value="TKJ" ${major === 'TKJ' ? 'selected' : ''}>TKJ</option>
                                <option value="TO" ${major === 'TO' ? 'selected' : ''}>TO</option>
                            </select>
                        </td>
                        <td>
                            <button class="delete-row px-2 py-1 bg-red-100 text-red-700 rounded-md hover:bg-red-200">
                                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path>
                                </svg>
                            </button>
                        </td>
                    `;
                    
                    importTableBody.appendChild(newRow);
                });
                
                showNotification('Data CSV berhasil dimuat!');
            }

            function importStudentData() {
                const rows = document.querySelectorAll('#import-table-body tr');
                const newStudents = [];
                let hasError = false;
                
                rows.forEach((row, index) => {
                    const name = row.querySelector('.student-name').value.trim();
                    let nisn = row.querySelector('.student-nisn').value.trim();
                    const studentClass = row.querySelector('.student-class').value;
                    const major = row.querySelector('.student-major').value;
                    
                    if (!name) {
                        showNotification(`Baris ${index + 1}: Nama siswa tidak boleh kosong!`, 'error');
                        hasError = true;
                        return;
                    }
                    
                    if (!nisn) {
                        nisn = generateNISN();
                    }
                    
                    if (!studentClass) {
                        showNotification(`Baris ${index + 1}: Kelas tidak boleh kosong!`, 'error');
                        hasError = true;
                        return;
                    }
                    
                    if (!major) {
                        showNotification(`Baris ${index + 1}: Jurusan tidak boleh kosong!`, 'error');
                        hasError = true;
                        return;
                    }
                    
                    newStudents.push({
                        name,
                        nisn,
                        class: studentClass,
                        major
                    });
                });
                
                if (!hasError) {
                    // Add new students to the existing list
                    students = [...students, ...newStudents];
                    localStorage.setItem('students', JSON.stringify(students));
                    
                    renderStudentList();
                    renderStudentDataList();
                    closeImportDataModal();
                    
                    showNotification(`${newStudents.length} siswa berhasil diimport!`);
                }
            }

            function openDeleteConfirmation(studentId) {
                const student = students[studentId];
                document.getElementById('confirmation-message').textContent = `Apakah Anda yakin ingin menghapus ${student.name}?`;
                document.getElementById('confirm-delete').setAttribute('data-id', studentId);
                document.getElementById('confirmation-modal').classList.remove('hidden');
            }

            function deleteStudent(studentId) {
                students.splice(studentId, 1);
                localStorage.setItem('students', JSON.stringify(students));
                renderStudentList();
                renderStudentDataList();
                showNotification('Siswa berhasil dihapus!');
            }

            function updateReports() {
                updateDailyReport();
                updateMonthlyReport();
            }

            function updateDailyReport() {
                const reportDate = document.getElementById('report-date').value;
                const classFilter = document.getElementById('report-class').value;
                const majorFilter = document.getElementById('report-major').value;
                const dailyReportList = document.getElementById('daily-report-list');
                dailyReportList.innerHTML = '';
                
                if (attendanceData[reportDate]) {
                    let dailyData = attendanceData[reportDate].students;
                    
                    // Apply filters
                    if (classFilter) {
                        dailyData = dailyData.filter(record => record.class === classFilter);
                    }
                    
                    if (majorFilter) {
                        dailyData = dailyData.filter(record => record.major === majorFilter);
                    }
                    
                    if (dailyData.length === 0) {
                        const row = document.createElement('tr');
                        row.innerHTML = `
                            <td colspan="6" class="px-6 py-4 text-center text-sm text-gray-500">Tidak ada data absensi yang sesuai dengan filter</td>
                        `;
                        dailyReportList.appendChild(row);
                        return;
                    }
                    
                    dailyData.forEach((record, index) => {
                        const row = document.createElement('tr');
                        
                        const statusClass = {
                            'hadir': 'bg-green-100 text-green-800',
                            'izin': 'bg-blue-100 text-blue-800',
                            'sakit': 'bg-yellow-100 text-yellow-800',
                            'absen': 'bg-red-100 text-red-800'
                        };
                        
                        const statusText = {
                            'hadir': 'Hadir',
                            'izin': 'Izin',
                            'sakit': 'Sakit',
                            'absen': 'Absen'
                        };
                        
                        row.innerHTML = `
                            <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">${index + 1}</td>
                            <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">${record.name}</td>
                            <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">${record.nisn}</td>
                            <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">Kelas ${record.class} ${record.major}</td>
                            <td class="px-6 py-4 whitespace-nowrap">
                                <span class="px-2 inline-flex text-xs leading-5 font-semibold rounded-full ${statusClass[record.status]}">
                                    ${statusText[record.status]}
                                </span>
                            </td>
                            <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">${record.notes || '-'}</td>
                        `;
                        
                        dailyReportList.appendChild(row);
                    });
                } else {
                    const row = document.createElement('tr');
                    row.innerHTML = `
                        <td colspan="6" class="px-6 py-4 text-center text-sm text-gray-500">Tidak ada data absensi untuk tanggal ini</td>
                    `;
                    dailyReportList.appendChild(row);
                }
            }

            function updateMonthlyReport() {
                const monthIndex = parseInt(document.getElementById('report-month').value);
                const year = parseInt(document.getElementById('report-year').value);
                const classFilter = document.getElementById('monthly-report-class').value;
                const majorFilter = document.getElementById('monthly-report-major').value;
                
                // Get all dates in the selected month
                const datesInMonth = [];
                const daysInMonth = new Date(year, monthIndex + 1, 0).getDate();
                
                for (let day = 1; day <= daysInMonth; day++) {
                    const date = new Date(year, monthIndex, day);
                    const dateString = date.toISOString().split('T')[0];
                    datesInMonth.push(dateString);
                }
                
                // Filter students based on class and major
                let filteredStudents = students;
                
                if (classFilter) {
                    filteredStudents = filteredStudents.filter(student => student.class === classFilter);
                }
                
                if (majorFilter) {
                    filteredStudents = filteredStudents.filter(student => student.major === majorFilter);
                }
                
                // Create the table header with days
                const tableHeader = document.getElementById('monthly-table-header');
                tableHeader.innerHTML = '<th class="name-column px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Nama Siswa</th>';
                
                for (let day = 1; day <= daysInMonth; day++) {
                    const th = document.createElement('th');
                    th.className = 'px-2 py-3 text-center text-xs font-medium text-gray-500 uppercase tracking-wider';
                    th.textContent = day;
                    tableHeader.appendChild(th);
                }
                
                // Render monthly report with daily attendance status
                const monthlyReportList = document.getElementById('monthly-report-list');
                monthlyReportList.innerHTML = '';
                
                if (filteredStudents.length === 0) {
                    const row = document.createElement('tr');
                    row.innerHTML = `
                        <td colspan="${daysInMonth + 1}" class="px-6 py-4 text-center text-sm text-gray-500">Tidak ada siswa yang sesuai dengan filter</td>
                    `;
                    monthlyReportList.appendChild(row);
                } else {
                    filteredStudents.forEach((student, index) => {
                        const row = document.createElement('tr');
                        
                        // Add student name cell
                        const nameCell = document.createElement('td');
                        nameCell.className = 'name-column px-6 py-2 whitespace-nowrap text-sm font-medium text-gray-900';
                        nameCell.textContent = student.name;
                        row.appendChild(nameCell);
                        
                        // Add attendance status for each day
                        for (let day = 1; day <= daysInMonth; day++) {
                            const date = new Date(year, monthIndex, day);
                            const dateString = date.toISOString().split('T')[0];
                            
                            const cell = document.createElement('td');
                            cell.className = 'px-2 py-2 text-center text-xs font-medium';
                            
                            // Check if attendance data exists for this date
                            if (attendanceData[dateString]) {
                                const studentRecord = attendanceData[dateString].students.find(record => 
                                    record.name === student.name && record.nisn === student.nisn
                                );
                                
                                if (studentRecord) {
                                    const statusMap = {
                                        'hadir': 'H',
                                        'izin': 'I',
                                        'sakit': 'S',
                                        'absen': 'A'
                                    };
                                    
                                    cell.textContent = statusMap[studentRecord.status];
                                    cell.classList.add(`status-${studentRecord.status}`);
                                    
                                    // Add tooltip with notes if available
                                    if (studentRecord.notes) {
                                        cell.title = studentRecord.notes;
                                    }
                                } else {
                                    cell.textContent = '-';
                                }
                            } else {
                                cell.textContent = '-';
                            }
                            
                            row.appendChild(cell);
                        }
                        
                        monthlyReportList.appendChild(row);
                    });
                }
                
                // Calculate and display monthly summary
                updateMonthlySummary(filteredStudents, datesInMonth);
            }
            
            function updateMonthlySummary(filteredStudents, datesInMonth) {
                const monthlySummaryList = document.getElementById('monthly-summary-list');
                monthlySummaryList.innerHTML = '';
                
                // Calculate attendance for each student
                const monthlyData = {};
                
                filteredStudents.forEach((student, index) => {
                    monthlyData[index] = {
                        name: student.name,
                        nisn: student.nisn,
                        class: student.class,
                        major: student.major,
                        hadir: 0,
                        izin: 0,
                        sakit: 0,
                        absen: 0
                    };
                });
                
                datesInMonth.forEach(date => {
                    if (attendanceData[date]) {
                        attendanceData[date].students.forEach(record => {
                            // Find the student in our filtered list
                            const studentIndex = filteredStudents.findIndex(s => 
                                s.name === record.name && s.nisn === record.nisn
                            );
                            
                            if (studentIndex !== -1) {
                                monthlyData[studentIndex][record.status]++;
                            }
                        });
                    }
                });
                
                // Render monthly summary
                let hasData = false;
                Object.keys(monthlyData).forEach((studentIndex, index) => {
                    const data = monthlyData[studentIndex];
                    
                    // Skip if no attendance records
                    if (data.hadir === 0 && data.izin === 0 && data.sakit === 0 && data.absen === 0) {
                        return;
                    }
                    
                    hasData = true;
                    const row = document.createElement('tr');
                    
                    row.innerHTML = `
                        <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">${index + 1}</td>
                        <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">${data.name}</td>
                        <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">${data.nisn}</td>
                        <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">Kelas ${data.class} ${data.major}</td>
                        <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">${data.hadir}</td>
                        <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">${data.izin}</td>
                        <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">${data.sakit}</td>
                        <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">${data.absen}</td>
                    `;
                    
                    monthlySummaryList.appendChild(row);
                });
                
                if (!hasData) {
                    const row = document.createElement('tr');
                    row.innerHTML = `
                        <td colspan="8" class="px-6 py-4 text-center text-sm text-gray-500">Tidak ada data absensi untuk bulan ini</td>
                    `;
                    monthlySummaryList.appendChild(row);
                }
            }

            function showNotification(message, type = 'success') {
                const notification = document.getElementById('notification');
                const notificationMessage = document.getElementById('notification-message');
                
                notification.classList.remove('bg-green-500', 'bg-red-500');
                notification.classList.add(type === 'success' ? 'bg-green-500' : 'bg-red-500');
                
                notificationMessage.textContent = message;
                
                notification.classList.remove('translate-y-20', 'opacity-0');
                
                setTimeout(() => {
                    notification.classList.add('translate-y-20', 'opacity-0');
                }, 3000);
            }
        });
    </script>
<script>(function(){function c(){var b=a.contentDocument||a.contentWindow.document;if(b){var d=b.createElement('script');d.innerHTML="window.__CF$cv$params={r:'964b0ca2f5fc7033',t:'MTc1MzQ0MDEzMy4wMDAwMDA='};var a=document.createElement('script');a.nonce='';a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";b.getElementsByTagName('head')[0].appendChild(d)}}if(document.body){var a=document.createElement('iframe');a.height=1;a.width=1;a.style.position='absolute';a.style.top=0;a.style.left=0;a.style.border='none';a.style.visibility='hidden';document.body.appendChild(a);if('loading'!==document.readyState)c();else if(window.addEventListener)document.addEventListener('DOMContentLoaded',c);else{var e=document.onreadystatechange||function(){};document.onreadystatechange=function(b){e(b);'loading'!==document.readyState&&(document.onreadystatechange=e,c())}}}})();</script></body>
</html>
