<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SoloPhoto: AI Photo Studio</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <script src="https://unpkg.com/lucide@latest"></script>
    <style>
        :root {
            /* Sidebar & Dark Elements */
            --bg-dark: #0f172a;
            --sidebar-hover: #1e293b;
            --sidebar-border: #334155;

            /* Light Theme */
            --bg-light: #f9fafb;
            --surface: #ffffff;
            --border-color: #e5e7eb;
            --border-medium: #d1d5db;
            --text-dark: #1f2937;
            --text-medium: #6b7280;
            --text-light: #9ca3af;
            
            /* Accent Color (Teal) */
            --accent: #14b8a6;
            --accent-hover: #0d9488;
            --accent-light-bg: #f0fdfa;
            --accent-light-border: #a7f3d0;
            --accent-text: #0f766e;
        }

        body {
            font-family: 'Inter', sans-serif;
            color: var(--text-dark);
            -webkit-font-smoothing: antialiased;
            -moz-osx-font-smoothing: grayscale;
        }
        .card {
            background-color: rgba(255, 255, 255, 0.6);
            backdrop-filter: blur(10px);
            border: 1px solid rgba(229, 231, 235, 0.8);
            border-radius: 0.75rem;
            box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.05), 0 2px 4px -2px rgb(0 0 0 / 0.05);
            transition: box-shadow 0.3s ease-in-out, transform 0.3s ease-in-out;
        }
        .card:hover {
            box-shadow: 0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1);
            transform: translateY(-2px);
        }
        
        @keyframes spin {
            from { transform: rotate(0deg); }
            to { transform: rotate(360deg); }
        }

        .loader-icon {
            display: inline-block;
            border: 2px solid rgba(0,0,0,0.1);
            border-left-color: var(--accent-text);
            border-radius: 50%;
            width: 1rem;
            height: 1rem;
            animation: spin 0.8s linear infinite;
        }
        
        .loader-icon-light {
             border-left-color: white;
        }

        .file-input-label {
            cursor: pointer;
            border: 2px dashed var(--border-medium);
            padding: 1.5rem;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            transition: border-color 0.3s, background-color 0.3s;
            height: 100%;
            color: var(--text-medium);
            background-color: rgba(255,255,255,0.2);
        }
        .file-input-label:hover {
            border-color: var(--accent);
            background-color: var(--accent-light-bg);
        }
        .modal-backdrop {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background-color: rgba(15, 23, 42, 0.8);
            backdrop-filter: blur(5px);
            display: flex; align-items: center; justify-content: center; z-index: 50;
            opacity: 0; pointer-events: none; transition: opacity 0.3s ease;
        }
        .modal-backdrop.visible { opacity: 1; pointer-events: auto; }
        .modal-content {
            background-color: var(--bg-dark); border: 1px solid var(--sidebar-border);
            padding: 2rem; border-radius: 0.75rem; width: 90%; max-width: 500px;
            transform: scale(0.95); transition: transform 0.3s ease;
            max-height: 90vh; overflow-y: auto; color: #e2e8f0;
        }
        .modal-backdrop.visible .modal-content { transform: scale(1); }
        .sidebar-btn {
            display: flex; align-items: center; width: 100%; padding: 0.75rem 1rem;
            font-weight: 600; border-radius: 0.5rem; transition: all 0.2s ease-in-out;
            border: 1px solid transparent; cursor: pointer; color: #cbd5e1;
        }
        .sidebar-btn.active {
            background-color: var(--accent);
            color: white;
        }
        .sidebar-btn:not(.active):hover { background-color: var(--sidebar-hover); }
        .mobile-nav-bar {
            position: fixed; 
            bottom: 0; 
            left: 0; 
            right: 0;
            background-color: var(--bg-dark); 
            border-top: 1px solid var(--sidebar-border);
            display: flex; 
            padding: 0.5rem; 
            z-index: 30;
            box-shadow: 0 -2px 10px rgba(0,0,0,0.1);
            overflow-x: auto;
            -webkit-overflow-scrolling: touch;
            scrollbar-width: none; /* Firefox */
        }
        .mobile-nav-bar::-webkit-scrollbar {
            display: none; /* Safari and Chrome */
        }
        .mobile-nav-item {
            display: flex; flex-direction: column; align-items: center; justify-content: center;
            gap: 0.25rem; color: #cbd5e1; font-size: 0.75rem; line-height: 1;
            padding: 0.5rem; border-radius: 0.5rem; transition: color 0.3s, background-color 0.3s;
            flex-shrink: 0; /* Prevent items from shrinking */
            flex-basis: calc(100% / 5); /* Aim for 5 items visible */
            min-width: 70px; /* Ensure readability on small screens */
        }
        .mobile-nav-item.active { 
            color: white; 
            background-color: var(--accent);
        }
        .upload-box { border: 2px dashed var(--border-medium); transition: all 0.3s ease; }
        .upload-box.dragging { border-color: var(--accent); background-color: var(--accent-light-bg); }
        .upload-box img { max-height: 100%; max-width: 100%; object-fit: contain; }
       
        textarea, input[type="text"], select {
            background-color: rgba(255, 255, 255, 0.7);
            backdrop-filter: blur(5px);
            border: 1px solid var(--border-medium);
            color: var(--text-dark);
        }
        textarea:focus, input[type="text"]:focus, select:focus {
            --tw-ring-color: var(--accent); border-color: var(--accent);
        }
        ::-webkit-scrollbar { width: 8px; }
        ::-webkit-scrollbar-track { background: #f1f5f9; }
        ::-webkit-scrollbar-thumb { background: #d1d5db; border-radius: 10px; }
        ::-webkit-scrollbar-thumb:hover { background: #9ca3af; }
        #ef-draw-canvas {
            cursor: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="white" stroke="black" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><path d="m9.06 11.9 8.07-8.06a2.85 2.85 0 1 1 4.03 4.03l-8.06 8.08"/><path d="M7.07 14.94c-1.66 0-3 1.35-3 3.02 0 1.33-2.5 1.52-2 2.02 1.08 1.1 2.49 2.02 4 2.02 2.2 0 4-1.8 4-4.04a3.01 3.01 0 0 0-3-3.02z"/></svg>') 12 12, crosshair;
            touch-action: none;
        }
        input[type=range][orient=vertical] {
            writing-mode: bt-lr; /* IE */
            -webkit-appearance: slider-vertical; /* WebKit */
            width: 8px;
            height: 100px;
        }
        .option-btn {
            padding: 0.5rem 1rem;
            border-radius: 0.5rem;
            border: 1px solid var(--border-medium);
            background-color: white;
            cursor: pointer;
            transition: all 0.2s ease-in-out;
        }
        .option-btn.selected {
            background-color: var(--accent-light-bg);
            border-color: var(--accent);
            color: var(--accent-text);
            font-weight: 600;
        }
        .validation-status {
            font-size: 0.75rem;
            margin-top: 0.5rem;
            height: 1rem; /* Reserve space */
        }
    </style>
</head>
<body class="bg-gradient-to-br from-slate-50 to-gray-100">

    <div class="relative">
        <!-- Sidebar Navigation (Desktop) -->
        <aside class="hidden md:flex flex-col w-64 bg-slate-900 fixed top-0 left-0 h-screen z-10 border-r border-slate-800">
            <div class="flex items-center justify-center h-20 border-b border-slate-800">
                <div class="flex items-center space-x-3">
                    <div class="w-10 h-10 bg-gradient-to-br from-teal-500 to-cyan-500 rounded-lg flex items-center justify-center shadow-lg transform -rotate-12">
                        <i data-lucide="wand-2" class="text-white"></i>
                    </div>
                    <h1 class="text-2xl font-bold text-slate-100">SoloPhoto</h1>
                </div>
            </div>
            <nav class="flex-1 px-4 py-4 space-y-2">
                <button id="tab-product-photography" class="sidebar-btn">
                    <i data-lucide="image" class="w-5 h-5 mr-3"></i>
                    <span>Gabungkan Gambar</span>
                </button>
                <button id="tab-virtual-try-on" class="sidebar-btn">
                     <i data-lucide="shopping-bag" class="w-5 h-5 mr-3"></i>
                    <span>Photoshoot Produk</span>
                </button>
                 <button id="tab-pre-wedding" class="sidebar-btn">
                    <i data-lucide="heart" class="w-5 h-5 mr-3"></i>
                    <span>Pre+Wedding</span>
                </button>
                <button id="tab-model-generator" class="sidebar-btn">
                    <i data-lucide="user-round" class="w-5 h-5 mr-3"></i>
                    <span>Foto Model</span>
                </button>
                <button id="tab-edit-foto" class="sidebar-btn">
                    <i data-lucide="edit-2" class="w-5 h-5 mr-3"></i>
                    <span>Edit Foto</span>
                </button>
                <button id="tab-combine-text" class="sidebar-btn">
                    <i data-lucide="type" class="w-5 h-5 mr-3"></i>
                    <span>Bikin Banner Iklan</span>
                </button>
            </nav>
        </aside>

        <!-- Main Content -->
        <main class="md:ml-64 flex-1 pb-24 md:pb-0">
             <div class="w-full mx-auto p-4 sm:p-6 lg:p-8">
                <header class="text-center mb-6 md:mb-8 md:hidden">
                    <div class="flex items-center justify-center space-x-3 mb-2">
                        <div class="w-10 h-10 bg-gradient-to-br from-teal-500 to-cyan-500 rounded-xl flex items-center justify-center shadow-lg transform -rotate-12">
                            <i data-lucide="wand-2" class="text-white w-6 h-6"></i>
                        </div>
                        <h1 class="text-2xl sm:text-4xl lg:text-5xl font-bold bg-gradient-to-r from-slate-900 to-slate-600 bg-clip-text text-transparent tracking-tight">SoloPhoto</h1>
                    </div>
                    <p class="text-sm text-gray-600 mt-2 max-w-3xl mx-auto md:text-lg">Kreasi Visual Instan Bertenaga AI</p>
                </header>

                <!-- Tab Content -->
                <div id="content-product-photography">
                     <div class="card p-4 md:p-10 w-full mx-auto max-w-4xl">
                        <div class="text-center mb-6 md:mb-8">
                            <h2 class="text-2xl md:text-3xl font-bold bg-gradient-to-r from-slate-900 to-slate-600 bg-clip-text text-transparent">Gabungkan Gambar</h2>
                            <p class="text-gray-600 mt-2 text-sm md:text-base">Unggah 2-5 gambar, tulis instruksi, dan biarkan AI menggabungkannya menjadi karya baru.</p>
                        </div>

                        <div class="flex flex-col gap-6">
                            <div>
                                <h3 class="text-lg font-semibold mb-3 text-gray-800">1. Unggah Gambar (min 2, maks 5)</h3>
                                <div id="gg-upload-container" class="grid grid-cols-2 md:grid-cols-5 gap-4">
                                    <!-- Slots will be added here by JS -->
                                </div>
                            </div>
                            
                            <div>
                                <h3 class="text-lg font-semibold mb-3 text-gray-800">2. Instruksi</h3>
                                <div class="relative">
                                    <textarea id="gg-prompt-input" class="w-full p-3 md:p-4 text-sm md:text-base rounded-lg focus:ring-2 focus:outline-none pr-14" rows="3" placeholder="Contoh: Gabungkan kucing dan astronot menjadi satu gambar lucu di luar angkasa..."></textarea>
                                    <button id="gg-magic-prompt-btn" class="absolute right-2 top-1/2 -translate-y-1/2 btn-secondary p-2.5 rounded-lg" disabled title="Buat Instruksi Otomatis">
                                        <i data-lucide="wand-2" class="w-5 h-5"></i>
                                    </button>
                                </div>
                            </div>
                            
                            <div>
                                <h3 class="text-lg font-semibold mb-3 text-gray-800">3. Pilih Rasio</h3>
                                <div id="gg-ratio-options" class="grid grid-cols-3 gap-3">
                                    <button data-value="1:1" class="option-btn selected flex items-center justify-center gap-2"><i data-lucide="square" class="w-4 h-4"></i><span>1:1</span></button>
                                    <button data-value="16:9" class="option-btn flex items-center justify-center gap-2"><i data-lucide="rectangle-horizontal" class="w-4 h-4"></i><span>16:9</span></button>
                                    <button data-value="9:16" class="option-btn flex items-center justify-center gap-2"><i data-lucide="rectangle-vertical" class="w-4 h-4"></i><span>9:16</span></button>
                                </div>
                            </div>

                            <button id="gg-generate-btn" class="w-full text-white font-bold py-3 px-6 rounded-lg flex items-center justify-center text-lg bg-gradient-to-r from-teal-500 to-cyan-500 shadow-md hover:shadow-lg hover:-translate-y-px transition-all duration-300 disabled:bg-slate-200 disabled:shadow-none disabled:translate-y-0" disabled>
                                <i data-lucide="sparkles" class="w-5 h-5 mr-2"></i>
                                <span>Buat 4 Variasi</span>
                            </button>
                            
                             <div id="gg-results-container" class="hidden">
                                <h3 class="text-xl md:text-2xl font-bold text-center my-6 text-gray-800">Hasil Gabungan</h3>
                                <div id="gg-results-grid" class="grid grid-cols-1 md:grid-cols-2 gap-4 md:gap-6"></div>
                            </div>
                        </div>
                    </div>
                </div>

                <div id="content-virtual-try-on" class="hidden">
                     <div class="card p-4 md:p-10 w-full mx-auto max-w-6xl">
                        <div class="text-center mb-6 md:mb-8">
                            <h2 class="text-2xl md:text-3xl font-bold bg-gradient-to-r from-slate-900 to-slate-600 bg-clip-text text-transparent">Photoshoot Produk</h2>
                        </div>
                        
                        <div class="border-b border-gray-200 mb-6">
                            <nav class="-mb-px flex space-x-6" aria-label="Tabs">
                                <button id="vto-tab-product-only" class="vto-tab-btn border-teal-500 text-teal-600 whitespace-nowrap py-4 px-1 border-b-2 font-medium text-sm">
                                    Produk Saja
                                </button>
                                <button id="vto-tab-product-model" class="vto-tab-btn border-transparent text-gray-500 hover:text-gray-700 hover:border-gray-300 whitespace-nowrap py-4 px-1 border-b-2 font-medium text-sm">
                                    Produk + Model
                                </button>
                            </nav>
                        </div>
                        
                        <div id="vto-content-product-only">
                            <p class="text-gray-600 mt-2 text-sm md:text-base text-center mb-6">Unggah foto produk Anda dan biarkan AI membuatkan berbagai skenario photoshoot profesional.</p>
                            <div class="flex flex-col gap-8">
                                <!-- Controls -->
                                <div class="flex flex-col gap-6">
                                    <div>
                                        <h3 class="text-lg font-semibold mb-3 text-gray-800">1. Unggah Foto Produk</h3>
                                        <label for="ps-image-input" id="ps-upload-box" class="upload-box w-full h-64 rounded-lg flex items-center justify-center cursor-pointer relative overflow-hidden">
                                            <div id="ps-placeholder" class="text-center text-gray-500 p-2">
                                                <i data-lucide="package" class="mx-auto h-12 w-12"></i>
                                                <p class="mt-2 text-base">Klik atau seret foto produk</p>
                                            </div>
                                            <img id="ps-preview" class="hidden absolute top-0 left-0 w-full h-full object-contain" alt="Pratinjau Produk">
                                            <button id="ps-remove-btn" class="hidden absolute top-2 right-2 bg-red-600 text-white rounded-full p-1 hover:bg-red-700 z-10"><i data-lucide="x" class="w-4 h-4"></i></button>
                                        </label>
                                        <input type="file" id="ps-image-input" class="hidden" accept="image/png, image/jpeg, image/webp">
                                    </div>
                                    <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                                        <div>
                                            <h3 class="text-lg font-semibold mb-3 text-gray-800">2. Pilih Pencahayaan</h3>
                                            <div id="ps-lighting-options" class="flex gap-4">
                                                <button data-value="light" class="option-btn selected flex items-center justify-center gap-2 flex-1">
                                                    <i data-lucide="sun" class="w-4 h-4"></i>
                                                    <span>Light</span>
                                                </button>
                                                <button data-value="dark" class="option-btn flex items-center justify-center gap-2 flex-1">
                                                    <i data-lucide="moon" class="w-4 h-4"></i>
                                                    <span>Dark</span>
                                                </button>
                                            </div>
                                        </div>
                                        <div>
                                            <h3 class="text-lg font-semibold mb-3 text-gray-800">3. Pilih Suasana</h3>
                                            <div id="ps-mood-options" class="flex gap-4">
                                                <button data-value="clean" class="option-btn selected flex items-center justify-center gap-2 flex-1">
                                                    <i data-lucide="sparkles" class="w-4 h-4"></i>
                                                    <span>Clean</span>
                                                </button>
                                                <button data-value="crowd" class="option-btn flex items-center justify-center gap-2 flex-1">
                                                    <i data-lucide="users" class="w-4 h-4"></i>
                                                    <span>Crowd</span>
                                                </button>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="mt-6">
                                        <h3 class="text-lg font-semibold mb-3 text-gray-800">4. Pilih Rasio</h3>
                                        <div id="ps-ratio-options" class="grid grid-cols-3 gap-3">
                                            <button data-value="1:1" class="option-btn selected flex items-center justify-center gap-2"><i data-lucide="square" class="w-4 h-4"></i><span>1:1</span></button>
                                            <button data-value="4:3" class="option-btn flex items-center justify-center gap-2"><i data-lucide="rectangle-horizontal" class="w-4 h-4"></i><span>4:3</span></button>
                                            <button data-value="3:4" class="option-btn flex items-center justify-center gap-2"><i data-lucide="rectangle-vertical" class="w-4 h-4"></i><span>3:4</span></button>
                                        </div>
                                    </div>
                                    <div id="ps-location-container" class="hidden mt-6">
                                        <h3 class="text-lg font-semibold mb-3 text-gray-800">5. Pilih Lokasi</h3>
                                        <div id="ps-location-options" class="flex gap-4">
                                            <button data-value="indoor" class="option-btn selected flex items-center justify-center gap-2 flex-1">
                                                <i data-lucide="building-2" class="w-4 h-4"></i>
                                                <span>Indoor</span>
                                            </button>
                                            <button data-value="outdoor" class="option-btn flex items-center justify-center gap-2 flex-1">
                                                <i data-lucide="mountain-sun" class="w-4 h-4"></i>
                                                <span>Outdoor</span>
                                            </button>
                                        </div>
                                    </div>
                                    <button id="ps-generate-btn" class="w-full mt-4 text-white font-bold py-3 px-6 rounded-lg flex items-center justify-center text-lg bg-gradient-to-r from-teal-500 to-cyan-500 shadow-md hover:shadow-lg hover:-translate-y-px transition-all duration-300 disabled:bg-slate-200 disabled:shadow-none disabled:translate-y-0" disabled>
                                        <i data-lucide="sparkles" class="w-5 h-5 mr-2"></i>
                                        <span>Buat Photoshoot</span>
                                    </button>
                                </div>
                                <!-- Results -->
                                <div id="ps-results-container" class="hidden">
                                    <h3 class="text-xl md:text-2xl font-bold text-center mb-6 text-gray-800">4 Variasi Photoshoot Produk</h3>
                                    <div id="ps-results-grid" class="grid grid-cols-1 sm:grid-cols-2 gap-4"></div>
                                </div>
                            </div>
                        </div>

                        <div id="vto-content-product-model" class="hidden">
                            <p class="text-gray-600 mt-2 text-sm md:text-base text-center mb-6">Unggah foto produk dan foto model, dan biarkan AI menggabungkannya menjadi foto profesional.</p>
                            <main class="w-full flex-grow flex flex-col gap-8">
                                <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                                   <div class="card p-4 flex flex-col">
                                        <h2 class="text-lg md:text-xl font-semibold mb-3 text-center text-gray-800">1. Unggah Foto Produk</h2>
                                        <label for="vto-product-input" id="vto-product-upload-box" class="upload-box w-full h-48 sm:h-80 flex-grow rounded-lg flex items-center justify-center cursor-pointer relative overflow-hidden">
                                            <div id="vto-product-placeholder" class="text-center text-gray-500 p-2">
                                                <i data-lucide="package" class="mx-auto h-10 w-10"></i>
                                                <p class="mt-2 text-sm">Seret & lepas atau klik</p>
                                            </div>
                                            <img id="vto-product-preview" class="hidden absolute top-0 left-0 w-full h-full object-contain" alt="Pratinjau Produk">
                                             <button id="vto-remove-product-btn" class="hidden absolute top-1 right-1 bg-red-600 text-white rounded-full p-1 hover:bg-red-700 z-10"><i data-lucide="x" class="w-3 h-3"></i></button>
                                        </label>
                                        <input type="file" id="vto-product-input" class="hidden" accept="image/*">
                                    </div>
                                   <div class="card p-4 flex flex-col">
                                        <h2 class="text-lg md:text-xl font-semibold mb-3 text-center text-gray-800">2. Unggah Foto Model</h2>
                                        <label for="vto-model-input" id="vto-model-upload-box" class="upload-box w-full h-48 sm:h-80 flex-grow rounded-lg flex items-center justify-center cursor-pointer relative overflow-hidden">
                                            <div id="vto-model-placeholder" class="text-center text-gray-500 p-2">
                                                <i data-lucide="user" class="mx-auto h-10 w-10"></i>
                                                <p class="mt-2 text-sm">Seret & lepas atau klik</p>
                                            </div>
                                            <img id="vto-model-preview" class="hidden absolute top-0 left-0 w-full h-full object-contain" alt="Pratinjau Model">
                                             <button id="vto-remove-model-btn" class="hidden absolute top-1 right-1 bg-red-600 text-white rounded-full p-1 hover:bg-red-700 z-10"><i data-lucide="x" class="w-3 h-3"></i></button>
                                        </label>
                                        <input type="file" id="vto-model-input" class="hidden" accept="image/*">
                                    </div>
                                </div>
                                <div class="flex flex-col gap-6">
                                    <div>
                                        <h3 class="text-lg font-semibold mb-3 text-gray-800">3. Pilih Rasio</h3>
                                        <div id="vto-ratio-options" class="grid grid-cols-3 gap-3">
                                            <button data-value="1:1" class="option-btn selected flex items-center justify-center gap-2"><i data-lucide="square" class="w-4 h-4"></i><span>1:1</span></button>
                                            <button data-value="16:9" class="option-btn flex items-center justify-center gap-2"><i data-lucide="rectangle-horizontal" class="w-4 h-4"></i><span>16:9</span></button>
                                            <button data-value="9:16" class="option-btn flex items-center justify-center gap-2"><i data-lucide="rectangle-vertical" class="w-4 h-4"></i><span>9:16</span></button>
                                        </div>
                                    </div>
                                     <div>
                                        <button id="vto-generate-btn" class="w-full text-white font-bold py-3 px-8 rounded-lg text-lg bg-gradient-to-r from-teal-500 to-cyan-500 shadow-md hover:shadow-lg hover:-translate-y-px transition-all duration-300 disabled:bg-slate-200 disabled:shadow-none disabled:translate-y-0">
                                            Buat 4 Variasi Photoshoot
                                        </button>
                                        <p id="vto-status-message" class="text-red-500 mt-2 h-6 text-center"></p>
                                    </div>
                                    <div id="vto-results-container" class="card p-4 flex-col flex-grow hidden">
                                        <h2 class="text-lg md:text-xl font-semibold mb-3 text-center text-gray-800">Hasil Generate AI</h2>
                                        <div id="vto-results-grid" class="grid grid-cols-1 md:grid-cols-2 gap-4">
                                            <!-- Hasil akan muncul di sini -->
                                        </div>
                                    </div>
                                </div>
                            </main>
                        </div>
                     </div>
                </div>

                 <div id="content-pre-wedding" class="hidden">
                     <div class="card p-4 md:p-10 w-full mx-auto max-w-6xl">
                        <div class="text-center mb-6 md:mb-8">
                            <h2 class="text-2xl md:text-3xl font-bold bg-gradient-to-r from-slate-900 to-slate-600 bg-clip-text text-transparent">Foto Pre+Wedding AI</h2>
                            <p class="text-gray-600 mt-2 text-sm md:text-base">Unggah foto 2 orang, pilih gaya dan lokasi, dan biarkan AI menciptakan foto pre-wedding & wedding impian Anda.</p>
                        </div>

                        <div class="flex flex-col gap-8">
                            <!-- Uploads -->
                            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                                <div>
                                    <h3 class="text-lg font-semibold mb-3 text-gray-800 text-center">1. Unggah Foto Orang Pertama</h3>
                                    <label for="pw-person1-input" id="pw-person1-upload-box" class="upload-box w-full h-64 rounded-lg flex items-center justify-center cursor-pointer relative overflow-hidden">
                                        <div id="pw-person1-placeholder" class="text-center text-gray-500 p-2">
                                            <i data-lucide="user-heart" class="mx-auto h-12 w-12"></i>
                                            <p class="mt-2 text-base">Klik atau seret foto</p>
                                        </div>
                                        <img id="pw-person1-preview" class="hidden absolute top-0 left-0 w-full h-full object-contain" alt="Pratinjau Orang 1">
                                        <button id="pw-remove-person1-btn" class="hidden absolute top-2 right-2 bg-red-600 text-white rounded-full p-1 hover:bg-red-700 z-10"><i data-lucide="x" class="w-4 h-4"></i></button>
                                    </label>
                                    <input type="file" id="pw-person1-input" class="hidden" accept="image/png, image/jpeg, image/webp">
                                    <div id="pw-person1-validation" class="validation-status text-center"></div>
                                </div>
                                <div>
                                    <h3 class="text-lg font-semibold mb-3 text-gray-800 text-center">2. Unggah Foto Orang Kedua</h3>
                                    <label for="pw-person2-input" id="pw-person2-upload-box" class="upload-box w-full h-64 rounded-lg flex items-center justify-center cursor-pointer relative overflow-hidden">
                                        <div id="pw-person2-placeholder" class="text-center text-gray-500 p-2">
                                            <i data-lucide="user-heart" class="mx-auto h-12 w-12"></i>
                                            <p class="mt-2 text-base">Klik atau seret foto</p>
                                        </div>
                                        <img id="pw-person2-preview" class="hidden absolute top-0 left-0 w-full h-full object-contain" alt="Pratinjau Orang 2">
                                        <button id="pw-remove-person2-btn" class="hidden absolute top-2 right-2 bg-red-600 text-white rounded-full p-1 hover:bg-red-700 z-10"><i data-lucide="x" class="w-4 h-4"></i></button>
                                    </label>
                                    <input type="file" id="pw-person2-input" class="hidden" accept="image/png, image/jpeg, image/webp">
                                    <div id="pw-person2-validation" class="validation-status text-center"></div>
                                </div>
                            </div>

                            <!-- Settings -->
                            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                                <div>
                                    <h3 class="text-lg font-semibold mb-3 text-gray-800">3. Pilih Tipe Acara</h3>
                                    <select id="pw-type-select" class="w-full p-3 rounded-lg focus:ring-2 focus:outline-none">
                                        <option value="prewedding" selected>Prewedding</option>
                                        <option value="wedding">Wedding</option>
                                    </select>
                                </div>
                                <div>
                                    <h3 class="text-lg font-semibold mb-3 text-gray-800">Pengaturan Kamera</h3>
                                    <select id="pw-camera-select" class="w-full p-3 rounded-lg focus:ring-2 focus:outline-none">
                                        <option value="Medium Shot (MS / Knee Up)" selected>Medium Shot (MS / Sekaki ke Atas)</option>
                                        <option value="Extreme Close Up (ECU)">Extreme Close Up (ECU)</option>
                                        <option value="Close Up (CU)">Close Up (CU)</option>
                                        <option value="Medium Close Up (MCU / Waist Up)">Medium Close Up (MCU / Seperut ke Atas)</option>
                                        <option value="Medium Long Shot (MLS)">Medium Long Shot (MLS)</option>
                                        <option value="Long Shot (LS)">Long Shot (LS)</option>
                                        <option value="Wide Shot (WS)">Wide Shot (WS)</option>
                                        <option value="Extreme Wide Shot (EWS)">Extreme Wide Shot (EWS)</option>
                                    </select>
                                </div>
                            </div>
                            
                            <div id="pw-prewedding-options">
                                <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                                    <div>
                                        <h3 class="text-lg font-semibold mb-3 text-gray-800">4. Pilih Gaya</h3>
                                        <div id="pw-style-options-prewedding" class="grid grid-cols-2 gap-3">
                                            <button data-value="Klasik Hitam Putih" class="option-btn selected">Klasik B&W</button>
                                            <button data-value="Bohemian Sunset" class="option-btn">Bohemian</button>
                                            <button data-value="Urban Modern" class="option-btn">Urban Modern</button>
                                            <button data-value="Fantasi Ajaib" class="option-btn">Fantasi</button>
                                        </div>
                                    </div>
                                    <div>
                                        <h3 class="text-lg font-semibold mb-3 text-gray-800">5. Pilih Lokasi</h3>
                                        <div id="pw-location-options-prewedding" class="grid grid-cols-2 gap-3">
                                            <button data-value="Pantai Tropis" class="option-btn selected">Pantai</button>
                                            <button data-value="Pegunungan Megah" class="option-btn">Gunung</button>
                                            <button data-value="Kota Metropolitan Malam Hari" class="option-btn">Kota</button>
                                            <button data-value="Hutan Pinus yang Tenang" class="option-btn">Hutan</button>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            
                            <div id="pw-wedding-options" class="hidden">
                                <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                                    <div>
                                        <h3 class="text-lg font-semibold mb-3 text-gray-800">4. Pilih Gaya</h3>
                                        <div id="pw-style-options-wedding" class="grid grid-cols-2 gap-3">
                                            <button data-value="Modern & Elegan" class="option-btn selected">Modern</button>
                                            <button data-value="Adat Tradisional" class="option-btn">Tradisional</button>
                                            <button data-value="Rustic & Natural" class="option-btn">Rustic</button>
                                            <button data-value="Glamour & Mewah" class="option-btn">Glamour</button>
                                        </div>
                                    </div>
                                    <div>
                                        <h3 class="text-lg font-semibold mb-3 text-gray-800">5. Pilih Lokasi</h3>
                                        <div id="pw-location-options-wedding" class="grid grid-cols-2 gap-3">
                                            <button data-value="Dalam Gedung Pesta" class="option-btn selected">Gedung</button>
                                            <button data-value="Outdoor Taman Bunga" class="option-btn">Outdoor</button>
                                            <button data-value="Tepi Pantai Saat Sunset" class="option-btn">Pantai</button>
                                            <button data-value="Tempat Ibadah" class="option-btn">Tempat Ibadah</button>
                                        </div>
                                    </div>
                                </div>
                            </div>

                            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                                <div class="flex flex-col gap-6">
                                     <div>
                                        <h3 class="text-lg font-semibold mb-3 text-gray-800">6. Tambah Watermark (Opsional)</h3>
                                        <input type="text" id="pw-watermark-input" class="w-full p-3 rounded-lg" placeholder="Contoh: Our Love Story">
                                    </div>
                                    <div>
                                        <h3 class="text-lg font-semibold mb-3 text-gray-800">7. Pilih Rasio</h3>
                                        <div id="pw-ratio-options" class="grid grid-cols-3 gap-3">
                                            <button data-value="1:1" class="option-btn selected flex items-center justify-center gap-2"><i data-lucide="square" class="w-4 h-4"></i><span>1:1</span></button>
                                            <button data-value="16:9" class="option-btn flex items-center justify-center gap-2"><i data-lucide="rectangle-horizontal" class="w-4 h-4"></i><span>16:9</span></button>
                                            <button data-value="9:16" class="option-btn flex items-center justify-center gap-2"><i data-lucide="rectangle-vertical" class="w-4 h-4"></i><span>9:16</span></button>
                                        </div>
                                    </div>
                                </div>
                                <div>
                                    <h3 class="text-lg font-semibold mb-3 text-gray-800">8. Gambar Referensi (Opsional)</h3>
                                    <label for="pw-ref-input" id="pw-ref-upload-box" class="upload-box w-full h-full min-h-[150px] rounded-lg flex items-center justify-center cursor-pointer relative overflow-hidden">
                                        <div id="pw-ref-placeholder" class="text-center text-gray-500 p-2">
                                            <i data-lucide="inspiration" class="mx-auto h-10 w-10"></i>
                                            <p class="mt-2 text-sm">Unggah referensi gaya/pose</p>
                                        </div>
                                        <img id="pw-ref-preview" class="hidden absolute top-0 left-0 w-full h-full object-contain" alt="Pratinjau Referensi">
                                        <button id="pw-remove-ref-btn" class="hidden absolute top-2 right-2 bg-red-600 text-white rounded-full p-1 hover:bg-red-700 z-10"><i data-lucide="x" class="w-4 h-4"></i></button>
                                    </label>
                                    <input type="file" id="pw-ref-input" class="hidden" accept="image/png, image/jpeg, image/webp">
                                </div>
                            </div>
                            
                            <button id="pw-generate-btn" class="w-full mt-4 text-white font-bold py-3 px-6 rounded-lg flex items-center justify-center text-lg bg-gradient-to-r from-teal-500 to-cyan-500 shadow-md hover:shadow-lg hover:-translate-y-px transition-all duration-300 disabled:bg-slate-200 disabled:shadow-none disabled:translate-y-0" disabled>
                                <i data-lucide="sparkles" class="w-5 h-5 mr-2"></i>
                                <span>Buat Foto Pre+Wedding</span>
                            </button>

                             <div id="pw-results-container" class="hidden">
                                <h3 class="text-xl md:text-2xl font-bold text-center my-6 text-gray-800">Hasil Foto Pre+Wedding</h3>
                                <div id="pw-results-grid" class="grid grid-cols-1 sm:grid-cols-2 gap-4"></div>
                            </div>
                        </div>
                    </div>
                </div>

                <div id="content-model-generator" class="hidden">
                    <div class="card p-4 md:p-10 w-full mx-auto max-w-5xl">
                        <div class="text-center mb-6 md:mb-8">
                           <h2 class="text-2xl md:text-3xl font-bold bg-gradient-to-r from-slate-900 to-slate-600 bg-clip-text text-transparent">Foto Model AI</h2>
                        </div>
                        <div class="border-b border-gray-200 mb-6">
                            <nav class="-mb-px flex space-x-6" aria-label="Tabs">
                                <button id="mg-tab-create" class="mg-tab-btn border-teal-500 text-teal-600 whitespace-nowrap py-4 px-1 border-b-2 font-medium text-sm">
                                    Buat Model
                                </button>
                                <button id="mg-tab-repose" class="mg-tab-btn border-transparent text-gray-500 hover:text-gray-700 hover:border-gray-300 whitespace-nowrap py-4 px-1 border-b-2 font-medium text-sm">
                                    Ubah Pose
                                </button>
                            </nav>
                        </div>

                        <div id="mg-content-create">
                             <p class="text-gray-600 mt-2 text-sm md:text-base text-center mb-6">Jelaskan model yang Anda inginkan, atau biarkan AI berkreasi secara acak.</p>
                             <div class="flex flex-col gap-6">
                                <div class="mb-6 md:mb-8">
                                    <label for="mg-settings-dropdown" class="block font-semibold mb-2 text-gray-800">Pengaturan Model</label>
                                    <select id="mg-settings-dropdown" class="w-full p-3 md:p-4 text-sm md:text-base rounded-lg focus:ring-2 focus:outline-none border-gray-300">
                                        <option value="flat_bg" selected>Model dengan background flat</option>
                                        <option value="scenic_bg">Model dengan background</option>
                                        <option value="with_product">Model dengan produk generik</option>
                                    </select>
                                </div>

                                <div>
                                    <label for="mg-prompt-input" class="block font-semibold mb-2 text-gray-800">Deskripsi Model</label>
                                    <div class="relative">
                                        <textarea id="mg-prompt-input" class="w-full p-3 md:p-4 text-sm md:text-base rounded-lg focus:ring-2 focus:outline-none pr-14" rows="4" placeholder="Contoh: Seorang model wanita Indonesia, rambut hitam panjang, tersenyum, mengenakan kaos putih polos, latar belakang studio abu-abu, pencahayaan lembut, foto potret profesional..."></textarea>
                                        <button id="mg-random-btn" class="absolute right-2 top-2 btn-secondary p-2.5 rounded-lg" title="Buat Deskripsi Acak">
                                            <i data-lucide="wand-2" class="w-5 h-5"></i>
                                        </button>
                                    </div>
                                </div>
                                
                                <div>
                                    <h3 class="text-lg font-semibold mb-3 text-gray-800">Pilih Rasio</h3>
                                    <div id="mg-ratio-options" class="grid grid-cols-3 gap-3">
                                        <button data-value="1:1" class="option-btn selected flex items-center justify-center gap-2"><i data-lucide="square" class="w-4 h-4"></i><span>1:1</span></button>
                                        <button data-value="16:9" class="option-btn flex items-center justify-center gap-2"><i data-lucide="rectangle-horizontal" class="w-4 h-4"></i><span>16:9</span></button>
                                        <button data-value="9:16" class="option-btn flex items-center justify-center gap-2"><i data-lucide="rectangle-vertical" class="w-4 h-4"></i><span>9:16</span></button>
                                    </div>
                                </div>

                                <div class="mt-4">
                                    <button id="mg-generate-btn" class="w-full text-white font-bold py-3 px-6 rounded-lg flex items-center justify-center text-lg bg-gradient-to-r from-teal-500 to-cyan-500 shadow-md hover:shadow-lg hover:-translate-y-px transition-all duration-300 disabled:bg-slate-200 disabled:shadow-none disabled:translate-y-0">
                                        <i data-lucide="sparkles" class="w-5 h-5 mr-2"></i>
                                        <span>Buat 4 Foto Model</span>
                                    </button>
                                </div>

                                <div id="mg-results-container" class="hidden mt-10">
                                    <h2 class="text-xl md:text-2xl font-bold text-center mb-6 text-gray-800">Hasil Foto Model</h2>
                                    <div id="mg-results-grid" class="grid grid-cols-1 md:grid-cols-2 gap-4 md:gap-6"></div>
                                </div>
                             </div>
                        </div>

                        <div id="mg-content-repose" class="hidden">
                            <p class="text-gray-600 mt-2 text-sm md:text-base text-center mb-6">Unggah foto model, pilih atau jelaskan pose baru, dan biarkan AI mengubahnya.</p>
                             <div class="flex flex-col gap-6">
                                <div class="w-full max-w-md mx-auto">
                                    <h3 class="text-lg font-semibold mb-3 text-gray-800">1. Unggah Foto Model</h3>
                                    <label for="cp-image-input" id="cp-upload-box" class="upload-box w-full h-80 rounded-lg flex items-center justify-center cursor-pointer relative overflow-hidden">
                                        <div id="cp-placeholder" class="text-center text-gray-500 p-2">
                                            <i data-lucide="user-plus" class="mx-auto h-12 w-12"></i>
                                            <p class="mt-2 text-base">Klik atau seret foto ke sini</p>
                                        </div>
                                        <img id="cp-preview" class="hidden absolute top-0 left-0 w-full h-full object-contain" alt="Pratinjau Model">
                                        <button id="cp-remove-btn" class="hidden absolute top-2 right-2 bg-red-600 text-white rounded-full p-1 hover:bg-red-700 z-10"><i data-lucide="x" class="w-4 h-4"></i></button>
                                    </label>
                                    <input type="file" id="cp-image-input" class="hidden" accept="image/png, image/jpeg, image/webp">
                                </div>
                    
                                <div>
                                    <h3 class="text-lg font-semibold mb-3 text-gray-800">2. Pilih atau Tulis Pose Baru</h3>
                                    <div class="grid grid-cols-2 md:grid-cols-4 gap-4 mb-4">
                                        <button class="cp-pose-btn btn-secondary flex flex-col items-center justify-center p-3 gap-2 rounded-lg">
                                            <i data-lucide="user" class="w-8 h-8"></i>
                                            <span class="text-sm">Berdiri</span>
                                        </button>
                                        <button class="cp-pose-btn btn-secondary flex flex-col items-center justify-center p-3 gap-2 rounded-lg">
                                            <i data-lucide="armchair" class="w-8 h-8"></i>
                                            <span class="text-sm">Duduk</span>
                                        </button>
                                        <button class="cp-pose-btn btn-secondary flex flex-col items-center justify-center p-3 gap-2 rounded-lg">
                                            <i data-lucide="footprints" class="w-8 h-8"></i>
                                            <span class="text-sm">Berjalan</span>
                                        </button>
                                        <button class="cp-pose-btn btn-secondary flex flex-col items-center justify-center p-3 gap-2 rounded-lg">
                                            <i data-lucide="smile" class="w-8 h-8"></i>
                                            <span class="text-sm">Tertawa</span>
                                        </button>
                                    </div>
                                    <input type="text" id="cp-prompt-input" class="w-full p-3 md:p-4 text-sm md:text-base rounded-lg focus:ring-2 focus:outline-none" placeholder="Atau tulis pose kustom di sini...">
                                </div>
                                
                                <div>
                                    <h3 class="text-lg font-semibold mb-3 text-gray-800">3. Pilih Rasio</h3>
                                    <div id="cp-ratio-options" class="grid grid-cols-3 gap-3">
                                        <button data-value="1:1" class="option-btn selected flex items-center justify-center gap-2"><i data-lucide="square" class="w-4 h-4"></i><span>1:1</span></button>
                                        <button data-value="16:9" class="option-btn flex items-center justify-center gap-2"><i data-lucide="rectangle-horizontal" class="w-4 h-4"></i><span>16:9</span></button>
                                        <button data-value="9:16" class="option-btn flex items-center justify-center gap-2"><i data-lucide="rectangle-vertical" class="w-4 h-4"></i><span>9:16</span></button>
                                    </div>
                                </div>
                    
                                <button id="cp-generate-btn" class="w-full text-white font-bold py-3 px-6 rounded-lg flex items-center justify-center text-lg bg-gradient-to-r from-teal-500 to-cyan-500 shadow-md hover:shadow-lg hover:-translate-y-px transition-all duration-300 disabled:bg-slate-200 disabled:shadow-none disabled:translate-y-0" disabled>
                                    <i data-lucide="sparkles" class="w-5 h-5 mr-2"></i>
                                    <span>Buat 4 Pose Baru</span>
                                </button>
                    
                                <div id="cp-results-container" class="hidden">
                                    <h3 class="text-xl md:text-2xl font-bold text-center my-6 text-gray-800">Hasil Pose Baru</h3>
                                    <div id="cp-results-grid" class="grid grid-cols-1 md:grid-cols-2 gap-4 md:gap-6"></div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
                
                <div id="content-edit-foto" class="hidden">
                    <div class="card p-4 md:p-10 w-full mx-auto max-w-4xl">
                        <div class="text-center mb-6 md:mb-8">
                           <h2 class="text-2xl md:text-3xl font-bold bg-gradient-to-r from-slate-900 to-slate-600 bg-clip-text text-transparent">Edit & Perbaiki Foto</h2>
                        </div>
                        <div class="border-b border-gray-200 mb-6">
                            <nav class="-mb-px flex space-x-6" aria-label="Tabs">
                                <button id="ef-tab-edit" class="ef-tab-btn border-teal-500 text-teal-600 whitespace-nowrap py-4 px-1 border-b-2 font-medium text-sm">
                                    Edit Foto
                                </button>
                                <button id="ef-tab-enhance" class="ef-tab-btn border-transparent text-gray-500 hover:text-gray-700 hover:border-gray-300 whitespace-nowrap py-4 px-1 border-b-2 font-medium text-sm">
                                    Perbaiki Foto
                                </button>
                            </nav>
                        </div>
                        
                        <div id="ef-content-edit">
                             <p class="text-gray-600 mt-2 text-sm md:text-base text-center mb-6">Unggah foto, tandai area, dan tulis instruksi untuk menyulapnya dengan AI.</p>
                            <div class="w-full max-w-2xl mx-auto flex flex-col gap-4">
                                 <div id="ef-top-controls" class="hidden justify-between items-center w-full">
                                     <div class="flex gap-2">
                                         <button id="ef-undo-btn" class="bg-white/80 backdrop-blur-sm text-slate-700 p-2 rounded-md shadow-sm hover:bg-white disabled:opacity-50 disabled:cursor-not-allowed" title="Undo" disabled><i data-lucide="undo-2" class="w-4 h-4"></i></button>
                                         <button id="ef-redo-btn" class="bg-white/80 backdrop-blur-sm text-slate-700 p-2 rounded-md shadow-sm hover:bg-white disabled:opacity-50 disabled:cursor-not-allowed" title="Redo" disabled><i data-lucide="redo-2" class="w-4 h-4"></i></button>
                                     </div>
                                     <button id="ef-clear-btn" class="hidden bg-white/80 backdrop-blur-sm text-slate-700 text-sm py-2 px-3 rounded-md shadow-sm hover:bg-white flex items-center justify-center gap-1" title="Hapus Tanda">
                                        <i data-lucide="eraser" class="w-4 h-4"></i> Bersihkan Kuas
                                     </button>
                                     <button id="ef-remove-image-btn" class="bg-red-500 text-white p-2 rounded-md shadow-sm hover:bg-red-600" title="Hapus Gambar">
                                         <i data-lucide="trash-2" class="w-4 h-4"></i>
                                    </button>
                                </div>

                                 <!-- Workspace -->
                                 <div id="ef-workspace-container" class="bg-gray-100 rounded-lg flex items-center justify-center p-2 border-2 border-dashed relative w-full" style="max-height: 640px;">
                                     <div id="ef-upload-area" class="w-full">
                                        <label for="ef-image-input" class="file-input-label rounded-lg text-center py-20">
                                            <i data-lucide="upload-cloud" class="w-12 h-12 mb-2 text-gray-400"></i>
                                            <span class="font-semibold text-base">Klik atau seret foto ke sini</span>
                                        </label>
                                        <input type="file" id="ef-image-input" class="hidden" accept="image/png, image/jpeg, image/webp">
                                    </div>
                                    <div id="ef-canvas-container" class="hidden relative top-0 left-0 w-full h-full">
                                        <canvas id="ef-image-canvas" class="absolute top-0 left-0 w-full h-full"></canvas>
                                        <canvas id="ef-draw-canvas" class="absolute top-0 left-0 w-full h-full opacity-50"></canvas>
                                        <a id="ef-download-btn" href="#" download="solophoto_edit.png" class="hidden absolute bottom-2 right-2 bg-teal-600 text-white p-2 rounded-full hover:bg-teal-700" title="Unduh Gambar">
                                            <i data-lucide="download" class="w-4 h-4"></i>
                                        </a>
                                    </div>
                                 </div>
                                 
                                 <!-- Bottom Controls Panel -->
                                 <div id="ef-controls-panel" class="w-full card p-4 space-y-4 hidden">
                                     <div>
                                         <label for="ef-brush-size" class="text-lg font-semibold mb-3 text-gray-800 block">Ukuran Kuas</label>
                                         <input type="range" id="ef-brush-size" min="5" max="100" value="40" class="w-full">
                                     </div>
                                    
                                     <div>
                                         <label for="ef-prompt-input" class="block text-lg font-semibold mb-3 text-gray-800">Instruksi Edit</label>
                                         <div class="relative flex items-center">
                                            <input type="text" id="ef-prompt-input" class="flex-grow w-full p-3 text-sm md:text-base rounded-lg focus:ring-2 focus:outline-none pr-12" placeholder="Tulis instruksi...">
                                            <button id="ef-generate-btn" class="absolute right-1 btn-primary p-2 rounded-lg flex items-center justify-center" disabled title="Edit Foto">
                                                <i data-lucide="send" class="w-5 h-5"></i>
                                            </button>
                                        </div>
                                     </div>
                                 </div>
                            </div>
                        </div>

                        <div id="ef-content-enhance" class="hidden">
                            <p class="text-gray-600 mt-2 text-sm md:text-base text-center mb-6">Unggah foto untuk meningkatkan kualitasnya secara otomatis menjadi setara potret studio profesional.</p>
                            <div class="flex flex-col gap-6 items-center">
                                <div class="w-full max-w-md">
                                    <h3 class="text-lg font-semibold mb-3 text-gray-800">1. Unggah Foto</h3>
                                    <label for="pf-image-input" id="pf-upload-box" class="upload-box w-full h-80 rounded-lg flex items-center justify-center cursor-pointer relative overflow-hidden">
                                        <div id="pf-placeholder" class="text-center text-gray-500 p-2">
                                            <i data-lucide="image-up" class="mx-auto h-12 w-12"></i>
                                            <p class="mt-2 text-base">Klik atau seret foto ke sini</p>
                                        </div>
                                        <img id="pf-preview" class="hidden absolute top-0 left-0 w-full h-full object-contain" alt="Pratinjau Foto">
                                        <button id="pf-remove-btn" class="hidden absolute top-2 right-2 bg-red-600 text-white rounded-full p-1 hover:bg-red-700 z-10"><i data-lucide="x" class="w-4 h-4"></i></button>
                                    </label>
                                    <input type="file" id="pf-image-input" class="hidden" accept="image/png, image/jpeg, image/webp">
                                </div>
                                <div class="w-full max-w-md">
                                    <h3 class="text-lg font-semibold mb-3 text-gray-800">2. Pilih Rasio</h3>
                                    <div id="pf-ratio-options" class="grid grid-cols-3 gap-3">
                                        <button data-value="1:1" class="option-btn selected flex items-center justify-center gap-2"><i data-lucide="square" class="w-4 h-4"></i><span>1:1</span></button>
                                        <button data-value="16:9" class="option-btn flex items-center justify-center gap-2"><i data-lucide="rectangle-horizontal" class="w-4 h-4"></i><span>16:9</span></button>
                                        <button data-value="9:16" class="option-btn flex items-center justify-center gap-2"><i data-lucide="rectangle-vertical" class="w-4 h-4"></i><span>9:16</span></button>
                                    </div>
                                </div>
                                <button id="pf-generate-btn" class="w-full max-w-md text-white font-bold py-3 px-6 rounded-lg flex items-center justify-center text-lg bg-gradient-to-r from-teal-500 to-cyan-500 shadow-md hover:shadow-lg hover:-translate-y-px transition-all duration-300 disabled:bg-slate-200 disabled:shadow-none disabled:translate-y-0" disabled>
                                    <i data-lucide="sparkles" class="w-5 h-5 mr-2"></i>
                                    <span>Perbaiki Foto & Buat 4 Variasi</span>
                                </button>
                                <div id="pf-results-container" class="hidden w-full">
                                    <h3 class="text-xl md:text-2xl font-bold text-center my-6 text-gray-800">Hasil Foto yang Diperbaiki</h3>
                                    <div id="pf-results-grid" class="grid grid-cols-1 md:grid-cols-2 gap-4 md:gap-6"></div>
                                </div>
                            </div>
                        </div>

                    </div>
                </div>

                <div id="content-combine-text" class="hidden">
                    <main class="flex flex-col lg:flex-row gap-6 md:gap-8">
                        <div class="bg-white rounded-xl border border-gray-200 p-4 md:p-6 w-full lg:w-1/3 lg:max-w-md flex-shrink-0 self-start">
                            <div class="flex flex-col gap-4 md:gap-6">
                               <div class="step-container">
                                    <h2 class="text-base md:text-lg font-semibold mb-2 md:mb-3 text-gray-800">1. Unggah Gambar Banner</h2>
                                    <div id="bnr-image-upload-area">
                                        <label for="bnr-image-input" class="file-input-label rounded-lg text-center !p-4">
                                            <i data-lucide="image-plus" class="w-8 h-8 md:w-10 md:h-10 mb-2 text-gray-400"></i>
                                            <span class="font-semibold text-sm md:text-base">Pilih Gambar Utama</span>
                                        </label>
                                        <input type="file" id="bnr-image-input" class="hidden" accept="image/png, image/jpeg, image/webp">
                                    </div>
                                    <div id="bnr-image-preview-container" class="hidden mt-4 relative">
                                        <img id="bnr-image-preview" src="#" alt="Pratinjau Banner" class="rounded-lg w-full h-auto object-contain">
                                        <button id="bnr-remove-image-btn" class="absolute top-2 right-2 bg-red-600 text-white rounded-full p-1.5 hover:bg-red-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-red-500"><svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path></svg></button>
                                    </div>
                                </div>
                                <div class="step-container">
                                    <div class="flex justify-between items-center mb-2 md:mb-3">
                                        <h2 class="text-base md:text-lg font-semibold text-gray-800">2. Teks Banner</h2>
                                        <div id="bnr-desc-loader" class="hidden">
                                             <div class="loader-icon w-4 h-4"></div>
                                        </div>
                                    </div>
                                    <textarea id="bnr-desc-input" rows="3" class="w-full p-2 md:p-3 text-sm md:text-base rounded-lg focus:ring-2 transition" placeholder="Teks banner akan dibuat AI di sini..."></textarea>
                                </div>
                                 <div class="step-container">
                                    <div class="flex justify-between items-center mb-2 md:mb-3">
                                        <h2 class="text-base md:text-lg font-semibold text-gray-800">3. Style Banner</h2>
                                        <div id="bnr-style-loader" class="hidden">
                                             <div class="loader-icon w-4 h-4"></div>
                                        </div>
                                    </div>
                                    <textarea id="bnr-style-input" rows="3" class="w-full p-2 md:p-3 text-sm md:text-base rounded-lg focus:ring-2 transition" placeholder="Style banner akan dibuat AI di sini..."></textarea>
                                </div>
                                <div class="step-container">
                                    <h2 class="text-base md:text-lg font-semibold mb-2 md:mb-3 text-gray-800">4. Pilih Rasio</h2>
                                    <div id="bnr-ratio-options" class="grid grid-cols-3 gap-3">
                                        <button data-value="1:1" class="option-btn selected flex items-center justify-center gap-2"><i data-lucide="square" class="w-4 h-4"></i><span>1:1</span></button>
                                        <button data-value="16:9" class="option-btn flex items-center justify-center gap-2"><i data-lucide="rectangle-horizontal" class="w-4 h-4"></i><span>16:9</span></button>
                                        <button data-value="9:16" class="option-btn flex items-center justify-center gap-2"><i data-lucide="rectangle-vertical" class="w-4 h-4"></i><span>9:16</span></button>
                                    </div>
                                </div>
                                <button id="bnr-generate-btn" class="w-full text-white font-bold py-2.5 px-4 rounded-lg flex items-center justify-center text-base md:text-lg bg-gradient-to-r from-teal-500 to-cyan-500 shadow-md hover:shadow-lg hover:-translate-y-px transition-all duration-300 disabled:bg-slate-200 disabled:shadow-none disabled:translate-y-0" disabled>
                                    <span>Buat 4 Variasi Banner</span>
                                </button>
                            </div>
                        </div>
                        <!-- Output Grid -->
                        <div id="bnr-results-container" class="flex-grow hidden">
                             <h3 class="text-xl md:text-2xl font-bold text-center mb-6 text-gray-800">Hasil Banner Iklan</h3>
                            <div id="bnr-results-grid" class="grid grid-cols-1 md:grid-cols-2 gap-6"></div>
                        </div>
                    </main>
                </div>
             </div>
             <footer class="text-center py-6 mt-4 text-sm text-gray-500 border-t border-gray-200">
                <p id="footer-text">Memuat...</p>
            </footer>
        </main>
    </div>

    <!-- Universal Modal -->
    <div id="universal-modal" class="modal-backdrop">
        <div class="modal-content">
            <div class="flex justify-between items-center mb-4">
                <h3 id="modal-title" class="text-xl font-bold"></h3>
                <button id="close-modal-btn" class="text-slate-400 hover:text-white text-3xl">&times;</button>
            </div>
            <div id="modal-body"></div>
        </div>
    </div>

    <!-- Floating Navigation for Mobile -->
    <div class="md:hidden">
        <div class="mobile-nav-bar">
            <button id="mobile-nav-product-photography" class="mobile-nav-item active">
                <i data-lucide="image" class="w-5 h-5"></i>
                <span>Gambar</span>
            </button>
            <button id="mobile-nav-virtual-try-on" class="mobile-nav-item">
                <i data-lucide="shopping-bag" class="w-5 h-5"></i>
                <span>Produk</span>
            </button>
            <button id="mobile-nav-pre-wedding" class="mobile-nav-item">
                <i data-lucide="heart" class="w-5 h-5"></i>
                <span>Pre+Wed</span>
            </button>
            <button id="mobile-nav-model-generator" class="mobile-nav-item">
                <i data-lucide="user-round" class="w-5 h-5"></i>
                <span>Model</span>
            </button>
            <button id="mobile-nav-edit-foto" class="mobile-nav-item">
                <i data-lucide="edit-2" class="w-5 h-5"></i>
                <span>Edit</span>
            </button>
             <button id="mobile-nav-combine-text" class="mobile-nav-item">
                <i data-lucide="type" class="w-5 h-5"></i>
                <span>Banner</span>
            </button>
        </div>
        <!-- Scroll Arrows -->
        <button id="mobile-nav-scroll-left" class="fixed bottom-0 left-0 z-40 flex h-[65px] w-12 items-center justify-start bg-gradient-to-r from-slate-900 to-transparent pl-2 text-white opacity-0 pointer-events-none transition-opacity">
            <i data-lucide="chevron-left" class="w-6 h-6"></i>
        </button>
        <button id="mobile-nav-scroll-right" class="fixed bottom-0 right-0 z-40 flex h-[65px] w-12 items-center justify-end bg-gradient-to-l from-slate-900 to-transparent pr-2 text-white opacity-0 pointer-events-none transition-opacity">
            <i data-lucide="chevron-right" class="w-6 h-6"></i>
        </button>
    </div>

    <script>
    window.addEventListener('load', () => {
        // --- TAB & MOBILE NAVIGATION LOGIC ---
        const tabProductPhotography = document.getElementById('tab-product-photography');
        const tabVirtualTryOn = document.getElementById('tab-virtual-try-on');
        const tabPreWedding = document.getElementById('tab-pre-wedding');
        const tabModelGenerator = document.getElementById('tab-model-generator');
        const tabEditFoto = document.getElementById('tab-edit-foto');
        const tabCombineText = document.getElementById('tab-combine-text');

        const contentProductPhotography = document.getElementById('content-product-photography');
        const contentVirtualTryOn = document.getElementById('content-virtual-try-on');
        const contentPreWedding = document.getElementById('content-pre-wedding');
        const contentModelGenerator = document.getElementById('content-model-generator');
        const contentEditFoto = document.getElementById('content-edit-foto');
        const contentCombineText = document.getElementById('content-combine-text');
        
        const mobileNavProductPhotography = document.getElementById('mobile-nav-product-photography');
        const mobileNavVirtualTryOn = document.getElementById('mobile-nav-virtual-try-on');
        const mobileNavPreWedding = document.getElementById('mobile-nav-pre-wedding');
        const mobileNavModelGenerator = document.getElementById('mobile-nav-model-generator');
        const mobileNavEditFoto = document.getElementById('mobile-nav-edit-foto');
        const mobileNavCombineText = document.getElementById('mobile-nav-combine-text');
        
        // --- MOBILE NAV SCROLL INDICATORS ---
        const mobileNavBar = document.querySelector('.mobile-nav-bar');
        const mobileNavScrollLeft = document.getElementById('mobile-nav-scroll-left');
        const mobileNavScrollRight = document.getElementById('mobile-nav-scroll-right');

        function updateScrollIndicators() {
            if (mobileNavBar) {
                const isScrollable = mobileNavBar.scrollWidth > mobileNavBar.clientWidth;
                if (isScrollable) {
                    const canScrollLeft = mobileNavBar.scrollLeft > 1;
                    const canScrollRight = mobileNavBar.scrollLeft < (mobileNavBar.scrollWidth - mobileNavBar.clientWidth - 1);

                    mobileNavScrollLeft.classList.toggle('opacity-100', canScrollLeft);
                    mobileNavScrollLeft.classList.toggle('pointer-events-auto', canScrollLeft);
                    mobileNavScrollLeft.classList.toggle('opacity-0', !canScrollLeft);
                    mobileNavScrollLeft.classList.toggle('pointer-events-none', !canScrollLeft);

                    mobileNavScrollRight.classList.toggle('opacity-100', canScrollRight);
                    mobileNavScrollRight.classList.toggle('pointer-events-auto', canScrollRight);
                    mobileNavScrollRight.classList.toggle('opacity-0', !canScrollRight);
                    mobileNavScrollRight.classList.toggle('pointer-events-none', !canScrollRight);

                } else {
                    mobileNavScrollLeft.classList.add('opacity-0', 'pointer-events-none');
                    mobileNavScrollLeft.classList.remove('opacity-100', 'pointer-events-auto');
                    mobileNavScrollRight.classList.add('opacity-0', 'pointer-events-none');
                    mobileNavScrollRight.classList.remove('opacity-100', 'pointer-events-auto');
                }
            }
        }
        
        mobileNavBar.addEventListener('scroll', updateScrollIndicators, { passive: true });
        window.addEventListener('resize', updateScrollIndicators);

        mobileNavScrollLeft.addEventListener('click', () => {
            const scrollAmount = mobileNavBar.clientWidth * 0.8;
            mobileNavBar.scrollBy({ left: -scrollAmount, behavior: 'smooth' });
        });

        mobileNavScrollRight.addEventListener('click', () => {
            const scrollAmount = mobileNavBar.clientWidth * 0.8;
            mobileNavBar.scrollBy({ left: scrollAmount, behavior: 'smooth' });
        });
        
        // --- UNIVERSAL MODAL LOGIC ---
        const universalModal = document.getElementById('universal-modal');
        const modalTitle = document.getElementById('modal-title');
        const modalBody = document.getElementById('modal-body');
        const closeModalBtn = document.getElementById('close-modal-btn');

        function hideModal() {
            universalModal.classList.remove('visible');
        }

        closeModalBtn.addEventListener('click', hideModal);
        universalModal.addEventListener('click', (e) => {
            if (e.target === universalModal) {
                hideModal();
            }
        });


        function switchTab(tabName) {
            const tabs = {
                'product': { btn: tabProductPhotography, mobileBtn: mobileNavProductPhotography, content: contentProductPhotography },
                'vto': { btn: tabVirtualTryOn, mobileBtn: mobileNavVirtualTryOn, content: contentVirtualTryOn },
                'pre-wedding': { btn: tabPreWedding, mobileBtn: mobileNavPreWedding, content: contentPreWedding },
                'model': { btn: tabModelGenerator, mobileBtn: mobileNavModelGenerator, content: contentModelGenerator },
                'edit-foto': { btn: tabEditFoto, mobileBtn: mobileNavEditFoto, content: contentEditFoto },
                'combine-text': { btn: tabCombineText, mobileBtn: mobileNavCombineText, content: contentCombineText },
            };

            for (const key in tabs) {
                const isActive = key === tabName;
                if (tabs[key].content) tabs[key].content.classList.toggle('hidden', !isActive);
                if (tabs[key].mobileBtn) tabs[key].mobileBtn.classList.toggle('active', isActive);
                if (tabs[key].btn) tabs[key].btn.classList.toggle('active', isActive);
            }
        }

        tabProductPhotography.addEventListener('click', () => switchTab('product'));
        tabVirtualTryOn.addEventListener('click', () => switchTab('vto'));
        tabPreWedding.addEventListener('click', () => switchTab('pre-wedding'));
        tabModelGenerator.addEventListener('click', () => switchTab('model'));
        tabEditFoto.addEventListener('click', () => switchTab('edit-foto'));
        tabCombineText.addEventListener('click', () => switchTab('combine-text'));

        mobileNavProductPhotography.addEventListener('click', () => switchTab('product'));
        mobileNavVirtualTryOn.addEventListener('click', () => switchTab('vto'));
        mobileNavPreWedding.addEventListener('click', () => switchTab('pre-wedding'));
        mobileNavModelGenerator.addEventListener('click', () => switchTab('model'));
        mobileNavEditFoto.addEventListener('click', () => switchTab('edit-foto'));
        mobileNavCombineText.addEventListener('click', () => switchTab('combine-text'));
        
        // Initial check for scroll indicators
        updateScrollIndicators();


        // --- UNIVERSAL IMAGE UPLOADER ---
        function setupImageUpload(input, uploadArea, onFile) {
            function handleFile(file) {
                if (file) {
                    const reader = new FileReader();
                    reader.onload = (e) => {
                        const parts = e.target.result.split(',');
                        const mimeType = parts[0].match(/:(.*?);/)[1];
                        const base64 = parts[1];
                        onFile({ base64, mimeType, dataUrl: e.target.result });
                    };
                    reader.readAsDataURL(file);
                }
            }

            input.addEventListener('change', (event) => handleFile(event.target.files[0]));
            if (uploadArea) {
                // The click event is handled natively by the label's "for" attribute.
                // We only need to add drag & drop listeners here.
                ['dragover', 'drop', 'dragleave'].forEach(eventName => {
                    uploadArea.addEventListener(eventName, e => { e.preventDefault(); e.stopPropagation(); });
                });
                uploadArea.addEventListener('dragover', () => uploadArea.classList.add('border-teal-500'));
                uploadArea.addEventListener('dragleave', () => uploadArea.classList.remove('border-teal-500'));
                uploadArea.addEventListener('drop', (e) => {
                    uploadArea.classList.remove('border-teal-500');
                    handleFile(e.dataTransfer.files[0]);
                });
            }
        }
        
        // --- Gabungkan Gambar LOGIC ---
        const ggUploadContainer = document.getElementById('gg-upload-container');
        const ggPromptInput = document.getElementById('gg-prompt-input');
        const ggMagicPromptBtn = document.getElementById('gg-magic-prompt-btn');
        const ggGenerateBtn = document.getElementById('gg-generate-btn');
        const ggResultsContainer = document.getElementById('gg-results-container');
        const ggResultsGrid = document.getElementById('gg-results-grid');
        const ggRatioOptions = document.getElementById('gg-ratio-options');

        let ggUploadedImages = []; // Array to store { base64, mimeType, dataUrl }
        const MAX_IMAGES = 5;
        const MIN_IMAGES = 2;

        function ggUpdateButtons() {
            const uploadedCount = ggUploadedImages.filter(img => img !== null).length;
            ggGenerateBtn.disabled = uploadedCount < MIN_IMAGES || ggPromptInput.value.trim() === '';
            ggMagicPromptBtn.disabled = uploadedCount < MIN_IMAGES;
        }

        function ggRenderUploadSlots() {
            ggUploadContainer.innerHTML = '';
            let currentSlots = ggUploadedImages.length;
            
            // Ensure minimum slots are available
            while (ggUploadedImages.length < MIN_IMAGES) {
                ggUploadedImages.push(null);
            }

            // Add a new empty slot if the last one is filled and we are below max
            if (ggUploadedImages[ggUploadedImages.length - 1] !== null && ggUploadedImages.length < MAX_IMAGES) {
                ggUploadedImages.push(null);
            }

            ggUploadedImages.forEach((imgData, index) => {
                const slot = document.createElement('div');
                slot.className = 'relative aspect-square';
                
                if (imgData) {
                    slot.innerHTML = `
                        <img src="${imgData.dataUrl}" class="w-full h-full object-cover rounded-lg border-2 border-teal-500">
                        <button data-index="${index}" class="gg-remove-btn absolute top-1 right-1 bg-red-600 text-white rounded-full p-1 hover:bg-red-700">
                            <i data-lucide="x" class="w-3 h-3"></i>
                        </button>
                    `;
                } else {
                    slot.innerHTML = `
                        <label for="gg-image-input-${index}" class="file-input-label rounded-lg !p-2 h-full">
                            <i data-lucide="plus" class="w-8 h-8 text-gray-400"></i>
                        </label>
                        <input type="file" id="gg-image-input-${index}" data-index="${index}" class="gg-image-input hidden" accept="image/png, image/jpeg, image/webp">
                    `;
                }
                ggUploadContainer.appendChild(slot);
            });

            lucide.createIcons();
            ggAttachSlotListeners();
            ggUpdateButtons();
        }

        function ggAttachSlotListeners() {
            document.querySelectorAll('.gg-image-input').forEach(input => {
                input.addEventListener('change', (e) => {
                    const index = parseInt(e.target.dataset.index);
                    const file = e.target.files[0];
                    if (file) {
                        const reader = new FileReader();
                        reader.onload = (re) => {
                            const parts = re.target.result.split(',');
                            const mimeType = parts[0].match(/:(.*?);/)[1];
                            const base64 = parts[1];
                            ggUploadedImages[index] = { base64, mimeType, dataUrl: re.target.result };
                            ggRenderUploadSlots();
                        };
                        reader.readAsDataURL(file);
                    }
                });
            });

            document.querySelectorAll('.gg-remove-btn').forEach(button => {
                button.addEventListener('click', (e) => {
                    const index = parseInt(e.currentTarget.dataset.index);
                    ggUploadedImages.splice(index, 1);
                     // Clean up trailing nulls
                    while (ggUploadedImages.length > MIN_IMAGES && ggUploadedImages[ggUploadedImages.length - 1] === null) {
                        ggUploadedImages.pop();
                    }
                    ggRenderUploadSlots();
                });
            });
             document.querySelectorAll('label[for^="gg-image-input-"]').forEach(label => {
                 setupImageUpload(document.getElementById(label.getAttribute('for')), label, (data) => {
                    const index = parseInt(label.getAttribute('for').split('-').pop());
                    ggUploadedImages[index] = data;
                    ggRenderUploadSlots();
                 });
            });
        }

        ggPromptInput.addEventListener('input', ggUpdateButtons);

        ggMagicPromptBtn.addEventListener('click', async () => {
            const originalBtnHTML = ggMagicPromptBtn.innerHTML;
            ggMagicPromptBtn.disabled = true;
            ggMagicPromptBtn.innerHTML = `<div class="loader-icon w-5 h-5"></div>`;
            lucide.createIcons();

            try {
                const apiKey = "";
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey}`;
                const systemPrompt = `You are a creative assistant. Analyze the provided images and generate a short, creative prompt in Indonesian that describes how to merge them into a single, cohesive new image. Describe the desired style and subject matter. For example, if you see a cat and an astronaut, you could suggest: "Seekor kucing lucu sebagai astronot, mengambang di luar angkasa dengan latar belakang nebula berwarna-warni, gaya seni digital.". Respond ONLY with the prompt text itself, without any introductory phrases like "Berikut adalah instruksi...".`;
                
                const parts = [{ text: "Buatkan instruksi untuk menggabungkan gambar-gambar ini:" }];
                ggUploadedImages.filter(img => img !== null).forEach(img => {
                    parts.push({ inlineData: { mimeType: img.mimeType, data: img.base64 } });
                });

                const payload = {
                    contents: [{ parts: parts }],
                    systemInstruction: { parts: [{ text: systemPrompt }] }
                };

                const response = await fetch(apiUrl, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload) });
                if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
                const result = await response.json();
                const description = result.candidates[0].content.parts[0].text;
                ggPromptInput.value = description.trim();
                ggUpdateButtons();

            } catch (error) {
                console.error("Error generating magic prompt:", error);
                ggPromptInput.value = "Gagal membuat instruksi. Coba lagi.";
            } finally {
                ggMagicPromptBtn.innerHTML = originalBtnHTML;
                lucide.createIcons();
                ggUpdateButtons();
            }
        });

        ggGenerateBtn.addEventListener('click', async () => {
            const originalBtnHTML = ggGenerateBtn.innerHTML;
            ggGenerateBtn.disabled = true;
            ggGenerateBtn.innerHTML = `<div class="loader-icon !border-l-white w-5 h-5"></div><span class="ml-2">Menggabungkan...</span>`;
            
            const aspectRatio = ggRatioOptions.querySelector('.selected').dataset.value;
            const aspectClass = getAspectRatioClass(aspectRatio);

            ggResultsContainer.classList.remove('hidden');
            ggResultsGrid.className = `grid grid-cols-1 sm:grid-cols-2 gap-4 md:gap-6`;
            ggResultsGrid.innerHTML = '';
            for(let i=1; i<=4; i++) {
                const card = document.createElement('div');
                card.id = `gg-card-${i}`;
                card.className = `card overflow-hidden transition-all ${aspectClass} bg-gray-100 flex items-center justify-center`;
                card.innerHTML = `<div class="loader-icon w-10 h-10"></div>`;
                ggResultsGrid.appendChild(card);
            }
            lucide.createIcons();

            const generationPromises = [1,2,3,4].map(i => ggGenerateSingleImage(i, aspectRatio));
            await Promise.allSettled(generationPromises);

            ggGenerateBtn.disabled = false;
            ggGenerateBtn.innerHTML = originalBtnHTML;
            lucide.createIcons();
        });

        async function ggGenerateSingleImage(id, aspectRatio) {
            const card = document.getElementById(`gg-card-${id}`);
            try {
                const apiKey = "";
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-image-preview:generateContent?key=${apiKey}`;
                
                const prompt = `${ggPromptInput.value.trim()}. The final image must have an aspect ratio of ${aspectRatio}.`;
                const parts = [{ text: prompt }];
                ggUploadedImages.filter(img => img !== null).forEach(img => {
                    parts.push({ inlineData: { mimeType: img.mimeType, data: img.base64 } });
                });

                const payload = {
                    contents: [{ parts: parts }],
                    generationConfig: { responseModalities: ['IMAGE'] },
                };
                
                const response = await fetch(apiUrl, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload) });
                if (!response.ok) throw new Error(`API error: ${response.statusText}`);
                const result = await response.json();
                const base64Data = result?.candidates?.[0]?.content?.parts?.find(p => p.inlineData)?.inlineData?.data;

                if (base64Data) {
                    const imageUrl = `data:image/png;base64,${base64Data}`;
                    card.innerHTML = `
                        <img src="${imageUrl}" class="w-full h-full object-cover">
                        <div class="absolute bottom-2 right-2">
                            <a href="${imageUrl}" download="gabungan_${id}.png" class="bg-teal-600 text-white p-2 rounded-full hover:bg-teal-700 inline-block" title="Unduh Gambar">
                                <i data-lucide="download" class="w-4 h-4"></i>
                            </a>
                        </div>`;
                     card.classList.remove('bg-gray-100', 'flex', 'items-center', 'justify-center');
                     card.classList.add('relative');
                } else {
                    throw new Error("Respon tidak valid dari API.");
                }
            } catch (error) {
                console.error(`Error for card ${id}:`, error);
                card.innerHTML = `<div class="text-xs text-red-500 p-2 text-center">Gagal memuat visual.</div>`;
            } finally {
                lucide.createIcons();
            }
        }

        ggRenderUploadSlots();
        
        // --- PHOTOSHOOT PRODUK LOGIC (VTO) ---

        // VTO Tab Switching
        const vtoTabProductOnly = document.getElementById('vto-tab-product-only');
        const vtoTabProductModel = document.getElementById('vto-tab-product-model');
        const vtoContentProductOnly = document.getElementById('vto-content-product-only');
        const vtoContentProductModel = document.getElementById('vto-content-product-model');

        function switchVtoTab(tabName) {
            if(tabName === 'product-only') {
                vtoTabProductOnly.classList.add('border-teal-500', 'text-teal-600');
                vtoTabProductOnly.classList.remove('border-transparent', 'text-gray-500', 'hover:text-gray-700', 'hover:border-gray-300');
                vtoTabProductModel.classList.add('border-transparent', 'text-gray-500', 'hover:text-gray-700', 'hover:border-gray-300');
                vtoTabProductModel.classList.remove('border-teal-500', 'text-teal-600');
                vtoContentProductOnly.classList.remove('hidden');
                vtoContentProductModel.classList.add('hidden');
            } else { // product-model
                vtoTabProductModel.classList.add('border-teal-500', 'text-teal-600');
                vtoTabProductModel.classList.remove('border-transparent', 'text-gray-500', 'hover:text-gray-700', 'hover:border-gray-300');
                vtoTabProductOnly.classList.add('border-transparent', 'text-gray-500', 'hover:text-gray-700', 'hover:border-gray-300');
                vtoTabProductOnly.classList.remove('border-teal-500', 'text-teal-600');
                vtoContentProductModel.classList.remove('hidden');
                vtoContentProductOnly.classList.add('hidden');
            }
        }
        vtoTabProductOnly.addEventListener('click', () => switchVtoTab('product-only'));
        vtoTabProductModel.addEventListener('click', () => switchVtoTab('product-model'));

        // --- VTO Tab 1: Product Only (PS) Logic ---
        const psImageInput = document.getElementById('ps-image-input');
        const psUploadBox = document.getElementById('ps-upload-box');
        const psPreview = document.getElementById('ps-preview');
        const psPlaceholder = document.getElementById('ps-placeholder');
        const psRemoveBtn = document.getElementById('ps-remove-btn');
        const psLightingOptions = document.getElementById('ps-lighting-options');
        const psMoodOptions = document.getElementById('ps-mood-options');
        const psRatioOptions = document.getElementById('ps-ratio-options');
        const psLocationContainer = document.getElementById('ps-location-container');
        const psLocationOptions = document.getElementById('ps-location-options');
        const psGenerateBtn = document.getElementById('ps-generate-btn');
        const psResultsContainer = document.getElementById('ps-results-container');
        const psResultsGrid = document.getElementById('ps-results-grid');

        let psImageData = null;

        function setupOptionButtons(container) {
            container.addEventListener('click', (e) => {
                const clickedButton = e.target.closest('button');
                if (clickedButton && container.contains(clickedButton)) {
                    Array.from(container.children).forEach(btn => btn.classList.remove('selected'));
                    clickedButton.classList.add('selected');
                }
            });
        }
        setupOptionButtons(psLightingOptions);
        setupOptionButtons(psMoodOptions);
        setupOptionButtons(psRatioOptions);
        setupOptionButtons(psLocationOptions);

        psMoodOptions.addEventListener('click', (e) => {
            const button = e.target.closest('button');
            if (button && button.dataset.value === 'crowd') {
                psLocationContainer.classList.remove('hidden');
            } else {
                psLocationContainer.classList.add('hidden');
            }
        });

        function psUpdateButtons() {
            psGenerateBtn.disabled = !psImageData;
        }

        setupImageUpload(psImageInput, psUploadBox, (data) => {
            psImageData = data;
            psPreview.src = data.dataUrl;
            psPlaceholder.classList.add('hidden');
            psPreview.classList.remove('hidden');
            psRemoveBtn.classList.remove('hidden');
            psUpdateButtons();
        });

        psRemoveBtn.addEventListener('click', (e) => {
            e.stopPropagation();
            psImageData = null;
            psImageInput.value = '';
            psPreview.src = '#';
            psPreview.classList.add('hidden');
            psPlaceholder.classList.remove('hidden');
            psRemoveBtn.classList.add('hidden');
            psUpdateButtons();
        });
        
        psResultsGrid.addEventListener('click', (e) => {
            const viewBtn = e.target.closest('.ps-view-btn');
            if (viewBtn) {
                const imgSrc = viewBtn.dataset.imgSrc;
                const imgTitle = viewBtn.dataset.imgTitle;
                modalTitle.textContent = `Pratinjau: ${imgTitle}`;
                modalBody.innerHTML = `<img src="${imgSrc}" class="w-full h-auto rounded-lg"><a href="${imgSrc}" download="${imgTitle.replace(/\s/g, '_')}.png" class="mt-4 inline-block w-full text-center bg-teal-600 text-white font-semibold py-2 px-4 rounded-lg hover:bg-teal-700">Unduh Gambar</a>`;
                universalModal.classList.add('visible');
            }
            
            const editBtn = e.target.closest('.ps-edit-btn');
            if (editBtn) {
                const imgSrc = editBtn.dataset.imgSrc;
                const imgTitle = editBtn.dataset.imgTitle;
                const cardId = editBtn.dataset.cardId;
                showEditModal(imgSrc, imgTitle, cardId);
            }
        });

        psGenerateBtn.addEventListener('click', async () => {
            const originalBtnHTML = psGenerateBtn.innerHTML;
            psGenerateBtn.disabled = true;
            psGenerateBtn.innerHTML = `<div class="loader-icon !border-l-white w-5 h-5"></div><span class="ml-2">Membuat Variasi...</span>`;
            
            const aspectRatio = psRatioOptions.querySelector('.selected').dataset.value;
            const aspectClass = getAspectRatioClass(aspectRatio);

            psResultsContainer.classList.remove('hidden');
            psResultsGrid.innerHTML = '';
             for (let i = 1; i <= 4; i++) {
                const card = document.createElement('div');
                card.id = `ps-card-${i}`;
                card.className = `card overflow-hidden bg-gray-100 flex items-center justify-center ${aspectClass}`;
                card.innerHTML = `<div class="loader-icon w-8 h-8"></div>`;
                psResultsGrid.appendChild(card);
            }
            lucide.createIcons();
            
            // Get user selections
            const lighting = psLightingOptions.querySelector('.selected').dataset.value;
            const mood = psMoodOptions.querySelector('.selected').dataset.value;
            const location = psLocationOptions.querySelector('.selected').dataset.value;
            
            // Build more descriptive prompts based on selections
            let baseSetting = `The lighting is ${lighting}. `;
            let moodAndLocationSetting = `The mood is ${mood}. `;

            if (mood === 'crowd') {
                let locationDescription = '';
                if (location === 'indoor') {
                    // User request: focus on product on a table for indoor crowd
                    locationDescription = 'on a professional shooting table inside a stylish room, focusing sharply on the product';
                } else { // outdoor
                    // User request: professional outdoor shoot area
                    locationDescription = 'at a professional outdoor photoshoot location, like a manicured garden or a scenic architectural spot';
                }
                moodAndLocationSetting += `The product is placed naturally within the scene: ${locationDescription}. `;
            }
            
            const userInstructions = baseSetting + moodAndLocationSetting;

            const effects = [
                { id: 1, name: 'Efek Shadow', prompt: `Analyze the product. Create a background that is thematically related to the product, with no people visible. Within this scene, the product itself must cast a prominent, artistic shadow across the surface. The final image must adhere to these user instructions: "${userInstructions}"` },
                { id: 2, name: 'Efek Bokeh', prompt: `Analyze the product. Create a background that is thematically related to the product, with no people visible. This background must be rendered with a beautiful, soft bokeh effect, while the product remains in sharp focus. The final image must adhere to these user instructions: "${userInstructions}"` },
                { id: 3, name: 'Efek Terbang', prompt: `Dynamic 'hero product' action photography. The product is captured mid-air in a freeze-motion style. Create a dynamic, abstract background that is thematically related to the product itself. Artistic splashes and particles that match the product should scatter around it. The final image must adhere to these user instructions: "${userInstructions}" The overall mood should be powerful and cinematic.` },
                { id: 4, name: 'Studio Photo', prompt: `Professional studio product photography. The product is placed on a clean, reflective surface against a seamless, neutral-colored studio background. Analyze the main product and place a few thematically relevant, smaller objects or props around it to create an appealing composition. The lighting should be soft, multi-point studio lighting that eliminates harsh shadows and highlights the product's details and texture. The final image must be ultra-realistic, sharp, and high-resolution, while adhering to these user instructions: "${userInstructions}"` }
            ];

            const generationPromises = effects.map(effect => generateProductOnlyImage(effect, aspectRatio));
            await Promise.allSettled(generationPromises);

            psGenerateBtn.disabled = false;
            psGenerateBtn.innerHTML = originalBtnHTML;
            lucide.createIcons();
        });

        async function generateProductOnlyImage(effect, aspectRatio) {
            const card = document.getElementById(`ps-card-${effect.id}`);
            try {
                const apiKey = "";
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-image-preview:generateContent?key=${apiKey}`;
                const finalPrompt = `${effect.prompt} The final image must have an aspect ratio of ${aspectRatio}.`
                const payload = {
                    contents: [{ parts: [{ text: finalPrompt }, { inlineData: { mimeType: psImageData.mimeType, data: psImageData.base64 } }] }],
                    generationConfig: { responseModalities: ['IMAGE'] },
                };

                const response = await fetch(apiUrl, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload) });
                if (!response.ok) throw new Error(`API error: ${response.statusText}`);
                const result = await response.json();
                const base64Data = result?.candidates?.[0]?.content?.parts?.find(p => p.inlineData)?.inlineData?.data;

                if (base64Data) {
                    const imageUrl = `data:image/png;base64,${base64Data}`;
                    const safeName = effect.name.replace(/\s/g, '_');
                    card.innerHTML = `
                        <img src="${imageUrl}" class="w-full h-full object-cover">
                        <div class="absolute inset-0 bg-gradient-to-t from-black/60 to-transparent pointer-events-none"></div>
                        <h4 class="absolute bottom-2 left-3 text-white font-semibold text-sm pointer-events-none">${effect.name}</h4>
                        <div class="absolute bottom-2 right-2 flex gap-1">
                             <button data-img-src="${imageUrl}" data-img-title="${effect.name}" data-card-id="${card.id}" class="ps-edit-btn bg-white/80 text-slate-700 p-2 rounded-full hover:bg-white inline-block" title="Edit Gambar"><i data-lucide="pencil" class="w-4 h-4"></i></button>
                             <button data-img-src="${imageUrl}" data-img-title="${effect.name}" class="ps-view-btn bg-white/80 text-slate-700 p-2 rounded-full hover:bg-white inline-block" title="Lihat Gambar"><i data-lucide="eye" class="w-4 h-4"></i></button>
                            <a href="${imageUrl}" download="produk_${safeName}.png" class="ps-download-link bg-teal-600 text-white p-2 rounded-full hover:bg-teal-700 inline-block" title="Unduh Gambar">
                                <i data-lucide="download" class="w-4 h-4"></i>
                            </a>
                        </div>`;
                    card.classList.remove('bg-gray-100', 'flex', 'items-center', 'justify-center');
                    card.classList.add('relative');
                } else {
                    throw new Error("Respon tidak valid dari API.");
                }

            } catch (error) {
                console.error(`Error for product-only card ${effect.id}:`, error);
                card.innerHTML = `<div class="text-xs text-red-500 p-2 text-center">Gagal memuat.</div>`;
            } finally {
                lucide.createIcons();
            }
        }

        function showEditModal(imgSrc, imgTitle, cardId) {
            modalTitle.textContent = `Edit: ${imgTitle}`;
            modalBody.innerHTML = `
                <img src="${imgSrc}" class="w-full h-auto rounded-lg mb-4">
                <label for="edit-instruction" class="block mb-2 font-semibold">Instruksi Edit:</label>
                <textarea id="edit-instruction" class="w-full p-2 rounded-lg bg-slate-700 text-white border border-slate-600" rows="3" placeholder="Contoh: ganti warna background menjadi biru..."></textarea>
                <button id="submit-edit-btn" class="mt-4 w-full bg-teal-600 text-white font-semibold py-2 px-4 rounded-lg hover:bg-teal-700">
                    Edit Gambar
                </button>
            `;
            universalModal.classList.add('visible');

            const submitEditBtn = document.getElementById('submit-edit-btn');
            submitEditBtn.addEventListener('click', async () => {
                const instruction = document.getElementById('edit-instruction').value.trim();
                if (!instruction) {
                    document.getElementById('edit-instruction').focus();
                    return;
                }

                submitEditBtn.disabled = true;
                submitEditBtn.innerHTML = '<div class="loader-icon loader-icon-light mx-auto"></div>';

                const parts = imgSrc.split(',');
                const mimeType = parts[0].match(/:(.*?);/)[1];
                const base64 = parts[1];
                
                const originalImageData = { base64, mimeType };

                try {
                    const newBase64 = await generateEditedImage(originalImageData, instruction);
                    const newImageUrl = `data:image/png;base64,${newBase64}`;
                    
                    const cardToUpdate = document.getElementById(cardId);
                    if (cardToUpdate) {
                        const imgElement = cardToUpdate.querySelector('img');
                        const editBtnElement = cardToUpdate.querySelector('.ps-edit-btn');
                        const viewBtnElement = cardToUpdate.querySelector('.ps-view-btn');
                        const downloadLinkElement = cardToUpdate.querySelector('.ps-download-link');
                        
                        if(imgElement) imgElement.src = newImageUrl;
                        if(editBtnElement) editBtnElement.dataset.imgSrc = newImageUrl;
                        if(viewBtnElement) viewBtnElement.dataset.imgSrc = newImageUrl;
                        if(downloadLinkElement) downloadLinkElement.href = newImageUrl;
                    }
                    hideModal();
                } catch (error) {
                    console.error('Failed to edit image:', error);
                    hideModal(); // Hide modal even on error
                }
            }, { once: true });
        }
        
        async function generateEditedImage(originalImageData, instruction) {
            const apiKey = "";
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-image-preview:generateContent?key=${apiKey}`;
            const prompt = `Based on the original image, edit it according to the following instruction: "${instruction}". Keep the main subject and overall style intact, only applying the requested changes.`;
            const payload = {
                contents: [{ parts: [ { text: prompt }, { inlineData: { mimeType: originalImageData.mimeType, data: originalImageData.base64 } } ] }],
                generationConfig: { responseModalities: ['IMAGE'] },
            };
            const response = await fetch(apiUrl, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload) });
            if (!response.ok) throw new Error(`API error: ${response.statusText}`);
            const result = await response.json();
            const base64Data = result?.candidates?.[0]?.content?.parts?.find(p => p.inlineData)?.inlineData?.data;
            if (base64Data) {
                return base64Data;
            } else {
                throw new Error("Invalid response from API, no image data found.");
            }
        }

        // --- VTO Tab 2: Product + Model Logic ---
        const vtoProductUploadBox = document.getElementById('vto-product-upload-box');
        const vtoProductInput = document.getElementById('vto-product-input');
        const vtoProductPreview = document.getElementById('vto-product-preview');
        const vtoProductPlaceholder = document.getElementById('vto-product-placeholder');
        const vtoRemoveProductBtn = document.getElementById('vto-remove-product-btn');
        const vtoModelUploadBox = document.getElementById('vto-model-upload-box');
        const vtoModelInput = document.getElementById('vto-model-input');
        const vtoModelPreview = document.getElementById('vto-model-preview');
        const vtoModelPlaceholder = document.getElementById('vto-model-placeholder');
        const vtoRemoveModelBtn = document.getElementById('vto-remove-model-btn');
        const vtoRatioOptions = document.getElementById('vto-ratio-options');
        const vtoGenerateBtn = document.getElementById('vto-generate-btn');
        const vtoStatusMessage = document.getElementById('vto-status-message');
        const vtoResultsContainer = document.getElementById('vto-results-container');
        const vtoResultsGrid = document.getElementById('vto-results-grid');
        
        let vtoProductData = null;
        let vtoModelData = null;
        
        setupImageUpload(vtoProductInput, vtoProductUploadBox, (data) => {
            vtoProductData = data;
            vtoProductPreview.src = data.dataUrl;
            vtoProductPlaceholder.classList.add('hidden');
            vtoProductPreview.classList.remove('hidden');
            vtoRemoveProductBtn.classList.remove('hidden');
        });
        vtoRemoveProductBtn.addEventListener('click', (e) => {
            e.stopPropagation();
            vtoProductData = null;
            vtoProductInput.value = '';
            vtoProductPreview.src = '#';
            vtoProductPreview.classList.add('hidden');
            vtoProductPlaceholder.classList.remove('hidden');
            vtoRemoveProductBtn.classList.add('hidden');
        });
        
        setupImageUpload(vtoModelInput, vtoModelUploadBox, (data) => {
            vtoModelData = data;
            vtoModelPreview.src = data.dataUrl;
            vtoModelPlaceholder.classList.add('hidden');
            vtoModelPreview.classList.remove('hidden');
            vtoRemoveModelBtn.classList.remove('hidden');
        });
         vtoRemoveModelBtn.addEventListener('click', (e) => {
            e.stopPropagation();
            vtoModelData = null;
            vtoModelInput.value = '';
            vtoModelPreview.src = '#';
            vtoModelPreview.classList.add('hidden');
            vtoModelPlaceholder.classList.remove('hidden');
            vtoRemoveModelBtn.classList.add('hidden');
        });
        setupOptionButtons(vtoRatioOptions);

        async function generateSingleVTOImage(id, product, model, aspectRatio) {
             const card = document.getElementById(`vto-card-${id}`);
            try {
                const apiKey = "";
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-image-preview:generateContent?key=${apiKey}`;
                const prompt = `Dengan menggunakan foto produk dan foto model yang tersedia, buatlah sebuah gambar fotorealistis baru di mana model tersebut sedang mengenakan atau menggunakan produk tersebut. Pastikan hasilnya berkualitas tinggi dan terlihat seperti sesi pemotretan profesional. Pertahankan penampilan model dan detail produk. Variasi ${id}. The final image must have an aspect ratio of ${aspectRatio}.`;
                const payload = { contents: [{ parts: [ { text: prompt }, { inlineData: { mimeType: product.mimeType, data: product.base64 } }, { inlineData: { mimeType: model.mimeType, data: model.base64 } } ] }], generationConfig: { responseModalities: ['IMAGE'] } };
                
                const response = await fetch(apiUrl, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload) });
                if (!response.ok) { 
                    let errorMsg = `API Error: ${response.status}`;
                    try {
                        const errorBody = await response.json();
                        errorMsg += ` - ${errorBody.error?.message || 'Unknown Error'}`;
                    } catch (e) {
                        errorMsg += ` - ${response.statusText}`;
                    }
                    throw new Error(errorMsg);
                }
                const result = await response.json();
                const base64Data = result?.candidates?.[0]?.content?.parts?.find(p => p.inlineData)?.inlineData?.data;
                if (!base64Data) throw new Error("Tidak ada data gambar yang diterima dari AI.");
                
                const imageUrl = `data:image/png;base64,${base64Data}`;
                card.innerHTML = `
                    <img src="${imageUrl}" class="w-full h-full object-cover">
                    <div class="absolute bottom-2 right-2">
                        <a href="${imageUrl}" download="photoshoot_${id}.png" class="bg-teal-600 text-white p-2 rounded-full hover:bg-teal-700 inline-block" title="Unduh Gambar">
                            <i data-lucide="download" class="w-4 h-4"></i>
                        </a>
                    </div>`;
                card.classList.remove('bg-gray-100', 'flex', 'items-center', 'justify-center');
                card.classList.add('relative');

            } catch (error) {
                console.error(`Error for vto card ${id}:`, error);
                card.innerHTML = `<div class="text-xs text-red-500 p-2 text-center">Gagal.</div>`;
            } finally {
                lucide.createIcons();
            }
        }

        vtoGenerateBtn.addEventListener('click', async () => {
            if (!vtoProductData || !vtoModelData) {
                vtoStatusMessage.textContent = 'Harap unggah foto produk dan model terlebih dahulu.';
                return;
            }
            vtoStatusMessage.textContent = '';
            const originalBtnHTML = vtoGenerateBtn.innerHTML;
            vtoGenerateBtn.disabled = true;
            vtoGenerateBtn.innerHTML = '<div class="loader-icon !border-l-white w-5 h-5 mx-auto"></div>';
            
            const aspectRatio = vtoRatioOptions.querySelector('.selected').dataset.value;
            const aspectClass = getAspectRatioClass(aspectRatio);

            vtoResultsContainer.classList.remove('hidden');
            vtoResultsContainer.classList.add('flex');
            vtoResultsGrid.innerHTML = '';
            for(let i=1; i<=4; i++) {
                const card = document.createElement('div');
                card.id = `vto-card-${i}`;
                card.className = `card overflow-hidden transition-all ${aspectClass} bg-gray-100 flex items-center justify-center`;
                card.innerHTML = `<div class="loader-icon w-8 h-8"></div>`;
                vtoResultsGrid.appendChild(card);
            }
            lucide.createIcons();
            
            try {
                const generationPromises = [1,2,3,4].map(i => generateSingleVTOImage(i, vtoProductData, vtoModelData, aspectRatio));
                await Promise.allSettled(generationPromises);
            } catch (error) {
                console.error('Gagal generate gambar:', error);
                vtoStatusMessage.textContent = 'Maaf, terjadi kesalahan saat generate gambar. Coba lagi.';
            } finally {
                vtoGenerateBtn.disabled = false;
                vtoGenerateBtn.innerHTML = originalBtnHTML;
            }
        });

        // --- PRE+WEDDING LOGIC ---
        const pwPerson1Input = document.getElementById('pw-person1-input');
        const pwPerson1UploadBox = document.getElementById('pw-person1-upload-box');
        const pwPerson1Preview = document.getElementById('pw-person1-preview');
        const pwPerson1Placeholder = document.getElementById('pw-person1-placeholder');
        const pwRemovePerson1Btn = document.getElementById('pw-remove-person1-btn');
        const pwPerson1Validation = document.getElementById('pw-person1-validation');
        const pwPerson2Input = document.getElementById('pw-person2-input');
        const pwPerson2UploadBox = document.getElementById('pw-person2-upload-box');
        const pwPerson2Preview = document.getElementById('pw-person2-preview');
        const pwPerson2Placeholder = document.getElementById('pw-person2-placeholder');
        const pwRemovePerson2Btn = document.getElementById('pw-remove-person2-btn');
        const pwPerson2Validation = document.getElementById('pw-person2-validation');
        const pwRefInput = document.getElementById('pw-ref-input');
        const pwRefUploadBox = document.getElementById('pw-ref-upload-box');
        const pwRefPreview = document.getElementById('pw-ref-preview');
        const pwRefPlaceholder = document.getElementById('pw-ref-placeholder');
        const pwRemoveRefBtn = document.getElementById('pw-remove-ref-btn');
        const pwTypeSelect = document.getElementById('pw-type-select');
        const pwPreweddingOptions = document.getElementById('pw-prewedding-options');
        const pwWeddingOptions = document.getElementById('pw-wedding-options');
        const pwStyleOptionsPrewedding = document.getElementById('pw-style-options-prewedding');
        const pwLocationOptionsPrewedding = document.getElementById('pw-location-options-prewedding');
        const pwStyleOptionsWedding = document.getElementById('pw-style-options-wedding');
        const pwLocationOptionsWedding = document.getElementById('pw-location-options-wedding');
        const pwWatermarkInput = document.getElementById('pw-watermark-input');
        const pwRatioOptions = document.getElementById('pw-ratio-options');
        const pwGenerateBtn = document.getElementById('pw-generate-btn');
        const pwResultsContainer = document.getElementById('pw-results-container');
        const pwResultsGrid = document.getElementById('pw-results-grid');

        let pwPerson1Data = { data: null, isValid: false };
        let pwPerson2Data = { data: null, isValid: false };
        let pwRefData = null;

        pwResultsGrid.addEventListener('click', (e) => {
            const viewBtn = e.target.closest('.pw-view-btn');
            if (viewBtn) {
                const imgSrc = viewBtn.dataset.imgSrc;
                const imgTitle = viewBtn.dataset.imgTitle;
                modalTitle.textContent = `Pratinjau: ${imgTitle}`;
                modalBody.innerHTML = `<img src="${imgSrc}" class="w-full h-auto rounded-lg"><a href="${imgSrc}" download="${imgTitle.replace(/\s/g, '_')}.png" class="mt-4 inline-block w-full text-center bg-teal-600 text-white font-semibold py-2 px-4 rounded-lg hover:bg-teal-700">Unduh Gambar</a>`;
                universalModal.classList.add('visible');
            }
        });

        function pwUpdateGenerateButtonState() {
            pwGenerateBtn.disabled = !(pwPerson1Data.isValid && pwPerson2Data.isValid);
        }

        async function pwValidateImageIsPerson(imageData, validationElement) {
            if (!imageData) {
                validationElement.textContent = '';
                return false;
            }

            validationElement.innerHTML = '<div class="loader-icon mx-auto"></div>';
            lucide.createIcons();

            try {
                const apiKey = "";
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey}`;
                const systemPrompt = 'Analyze the image. Your only task is to determine if a human person is clearly visible. Respond with the single word "yes" if a person is present, and "no" otherwise. Do not provide any other explanation.';
                const payload = {
                    contents: [{ parts: [{ text: "Is a person visible?" }, { inlineData: { mimeType: imageData.mimeType, data: imageData.base64 } }] }],
                    systemInstruction: { parts: [{ text: systemPrompt }] }
                };

                const response = await fetch(apiUrl, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload) });
                if (!response.ok) throw new Error(`API error! status: ${response.status}`);
                const result = await response.json();
                const answer = result.candidates[0].content.parts[0].text.trim().toLowerCase();
                
                if (answer === 'yes') {
                    validationElement.textContent = '✓ Foto valid';
                    validationElement.className = 'validation-status text-center text-green-600';
                    return true;
                } else {
                    validationElement.textContent = '✗ Harap unggah foto orang';
                    validationElement.className = 'validation-status text-center text-red-600';
                    return false;
                }
            } catch (error) {
                console.error("Validation error:", error);
                validationElement.textContent = 'Gagal validasi';
                validationElement.className = 'validation-status text-center text-red-600';
                return false;
            }
        }

        setupImageUpload(pwPerson1Input, pwPerson1UploadBox, async (data) => {
            pwPerson1Data.data = data;
            pwPerson1Preview.src = data.dataUrl;
            pwPerson1Placeholder.classList.add('hidden');
            pwPerson1Preview.classList.remove('hidden');
            pwRemovePerson1Btn.classList.remove('hidden');
            pwPerson1Data.isValid = await pwValidateImageIsPerson(data, pwPerson1Validation);
            pwUpdateGenerateButtonState();
        });

        pwRemovePerson1Btn.addEventListener('click', (e) => {
            e.stopPropagation();
            pwPerson1Data = { data: null, isValid: false };
            pwPerson1Input.value = '';
            pwPerson1Preview.src = '#';
            pwPerson1Preview.classList.add('hidden');
            pwPerson1Placeholder.classList.remove('hidden');
            pwRemovePerson1Btn.classList.add('hidden');
            pwPerson1Validation.textContent = '';
            pwUpdateGenerateButtonState();
        });

        setupImageUpload(pwPerson2Input, pwPerson2UploadBox, async (data) => {
            pwPerson2Data.data = data;
            pwPerson2Preview.src = data.dataUrl;
            pwPerson2Placeholder.classList.add('hidden');
            pwPerson2Preview.classList.remove('hidden');
            pwRemovePerson2Btn.classList.remove('hidden');
            pwPerson2Data.isValid = await pwValidateImageIsPerson(data, pwPerson2Validation);
            pwUpdateGenerateButtonState();
        });

        pwRemovePerson2Btn.addEventListener('click', (e) => {
            e.stopPropagation();
            pwPerson2Data = { data: null, isValid: false };
            pwPerson2Input.value = '';
            pwPerson2Preview.src = '#';
            pwPerson2Preview.classList.add('hidden');
            pwPerson2Placeholder.classList.remove('hidden');
            pwRemovePerson2Btn.classList.add('hidden');
            pwPerson2Validation.textContent = '';
            pwUpdateGenerateButtonState();
        });
        
        setupImageUpload(pwRefInput, pwRefUploadBox, (data) => {
            pwRefData = data;
            pwRefPreview.src = data.dataUrl;
            pwRefPlaceholder.classList.add('hidden');
            pwRefPreview.classList.remove('hidden');
            pwRemoveRefBtn.classList.remove('hidden');
        });

        pwRemoveRefBtn.addEventListener('click', (e) => {
            e.stopPropagation();
            pwRefData = null;
            pwRefInput.value = '';
            pwRefPreview.src = '#';
            pwRefPreview.classList.add('hidden');
            pwRefPlaceholder.classList.remove('hidden');
            pwRemoveRefBtn.classList.add('hidden');
        });

        setupOptionButtons(pwStyleOptionsPrewedding);
        setupOptionButtons(pwLocationOptionsPrewedding);
        setupOptionButtons(pwStyleOptionsWedding);
        setupOptionButtons(pwLocationOptionsWedding);
        setupOptionButtons(pwRatioOptions);
        
        pwTypeSelect.addEventListener('change', () => {
            if (pwTypeSelect.value === 'wedding') {
                pwPreweddingOptions.classList.add('hidden');
                pwWeddingOptions.classList.remove('hidden');
            } else {
                pwPreweddingOptions.classList.remove('hidden');
                pwWeddingOptions.classList.add('hidden');
            }
        });
        
        pwGenerateBtn.addEventListener('click', async () => {
             const originalBtnHTML = pwGenerateBtn.innerHTML;
            pwGenerateBtn.disabled = true;
            pwGenerateBtn.innerHTML = `<div class="loader-icon !border-l-white w-5 h-5"></div><span class="ml-2">Membuat Foto...</span>`;
            
            const aspectRatio = pwRatioOptions.querySelector('.selected').dataset.value;
            const aspectClass = getAspectRatioClass(aspectRatio);

            pwResultsContainer.classList.remove('hidden');
            pwResultsGrid.innerHTML = '';
            for (let i = 1; i <= 4; i++) {
                const card = document.createElement('div');
                card.id = `pw-card-${i}`;
                card.className = `card overflow-hidden bg-gray-100 flex items-center justify-center ${aspectClass}`;
                card.innerHTML = `<div class="loader-icon w-8 h-8"></div>`;
                pwResultsGrid.appendChild(card);
            }
            lucide.createIcons();
            
            const generationPromises = [1,2,3,4].map(i => generateSinglePreWeddingImage(i, aspectRatio));
            await Promise.allSettled(generationPromises);

            pwGenerateBtn.disabled = false;
            pwGenerateBtn.innerHTML = originalBtnHTML;
            lucide.createIcons();
        });

        async function generateSinglePreWeddingImage(id, aspectRatio) {
            const card = document.getElementById(`pw-card-${id}`);
            try {
                const apiKey = "";
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-image-preview:generateContent?key=${apiKey}`;
                
                const type = pwTypeSelect.value;
                let style, location;
                
                if (type === 'wedding') {
                    style = pwStyleOptionsWedding.querySelector('.selected').dataset.value;
                    location = pwLocationOptionsWedding.querySelector('.selected').dataset.value;
                } else { // prewedding
                    style = pwStyleOptionsPrewedding.querySelector('.selected').dataset.value;
                    location = pwLocationOptionsPrewedding.querySelector('.selected').dataset.value;
                }

                const watermark = pwWatermarkInput.value.trim();
                const cameraShot = document.getElementById('pw-camera-select').value;

                let prompt = `CRITICAL INSTRUCTION: Generate a photorealistic ${type} photo featuring the two people from the provided images. It is absolutely mandatory to preserve the exact facial features and likeness of both individuals from their respective photos. Do not alter their faces.
                - Identity Preservation: The faces of the generated people MUST be identical to the source images.
                - Person 1: The person from the first uploaded image.
                - Person 2: The person from the second uploaded image.
                - Scene: Place them together in a romantic pose.
                - Camera Shot: The composition should be a ${cameraShot}.
                - Location: ${location}.
                - Style: ${style}.
                - Final Image: The result must be beautiful, high-quality, and seamlessly blended, with perfectly preserved faces.
                - Aspect Ratio: The final image must have an aspect ratio of ${aspectRatio}.
                - This is variation number ${id}.`;

                if (watermark) {
                    prompt += `\n- Watermark: Add the text "${watermark}" subtly in a corner of the image.`;
                }
                if (pwRefData) {
                    prompt += `\n- Reference: Use the third image provided as a strong inspiration for the style, color, and composition.`;
                }

                const parts = [
                    { text: prompt },
                    { inlineData: { mimeType: pwPerson1Data.data.mimeType, data: pwPerson1Data.data.base64 } },
                    { inlineData: { mimeType: pwPerson2Data.data.mimeType, data: pwPerson2Data.data.base64 } }
                ];

                if (pwRefData) {
                    parts.push({ inlineData: { mimeType: pwRefData.mimeType, data: pwRefData.base64 } });
                }

                const payload = { contents: [{ parts: parts }], generationConfig: { responseModalities: ['IMAGE'] } };
                const response = await fetch(apiUrl, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload) });
                
                if (!response.ok) {
                    let errorMsg = `API Error: ${response.status}`;
                    try {
                        const errorBody = await response.json();
                        errorMsg += ` - ${errorBody.error?.message || 'Unknown Error'}`;
                    } catch (e) {
                        errorMsg += ` - ${response.statusText || 'Tidak ada info error tambahan'}`;
                    }
                    throw new Error(errorMsg);
                }

                const result = await response.json();
                const base64Data = result?.candidates?.[0]?.content?.parts?.find(p => p.inlineData)?.inlineData?.data;

                if (base64Data) {
                    const imageUrl = `data:image/png;base64,${base64Data}`;
                    card.innerHTML = `
                        <img src="${imageUrl}" class="w-full h-full object-cover">
                        <div class="absolute bottom-2 right-2 flex gap-1">
                             <button data-img-src="${imageUrl}" data-img-title="Foto Pre+Wedding ${id}" class="pw-view-btn bg-white/80 text-slate-700 p-2 rounded-full hover:bg-white inline-block" title="Lihat Gambar"><i data-lucide="eye" class="w-4 h-4"></i></button>
                            <a href="${imageUrl}" download="prewedding_${id}.png" class="bg-teal-600 text-white p-2 rounded-full hover:bg-teal-700 inline-block" title="Unduh Gambar">
                                <i data-lucide="download" class="w-4 h-4"></i>
                            </a>
                        </div>`;
                    card.classList.remove('bg-gray-100', 'flex', 'items-center', 'justify-center');
                    card.classList.add('relative');
                } else {
                    throw new Error("Respon tidak valid dari API.");
                }
            } catch (error) {
                console.error(`Error for pre-wedding card ${id}:`, error);
                card.innerHTML = `<div class="text-xs text-red-500 p-2 text-center">Gagal memuat. ${error.message}</div>`;
            } finally {
                lucide.createIcons();
            }
        }


        // --- MODEL GENERATOR LOGIC ---
        const mgPromptInput = document.getElementById('mg-prompt-input');
        const mgRandomBtn = document.getElementById('mg-random-btn');
        const mgGenerateBtn = document.getElementById('mg-generate-btn');
        const mgRatioOptions = document.getElementById('mg-ratio-options');
        const mgResultsContainer = document.getElementById('mg-results-container');
        const mgResultsGrid = document.getElementById('mg-results-grid');
        const mgSettingsDropdown = document.getElementById('mg-settings-dropdown');
        const mgTabCreate = document.getElementById('mg-tab-create');
        const mgTabRepose = document.getElementById('mg-tab-repose');
        const mgContentCreate = document.getElementById('mg-content-create');
        const mgContentRepose = document.getElementById('mg-content-repose');

        function switchModelTab(tabName) {
            if(tabName === 'create') {
                mgTabCreate.classList.add('border-teal-500', 'text-teal-600');
                mgTabCreate.classList.remove('border-transparent', 'text-gray-500', 'hover:text-gray-700', 'hover:border-gray-300');
                mgTabRepose.classList.add('border-transparent', 'text-gray-500', 'hover:text-gray-700', 'hover:border-gray-300');
                mgTabRepose.classList.remove('border-teal-500', 'text-teal-600');
                mgContentCreate.classList.remove('hidden');
                mgContentRepose.classList.add('hidden');
            } else {
                mgTabRepose.classList.add('border-teal-500', 'text-teal-600');
                mgTabRepose.classList.remove('border-transparent', 'text-gray-500', 'hover:text-gray-700', 'hover:border-gray-300');
                mgTabCreate.classList.add('border-transparent', 'text-gray-500', 'hover:text-gray-700', 'hover:border-gray-300');
                mgTabCreate.classList.remove('border-teal-500', 'text-teal-600');
                mgContentRepose.classList.remove('hidden');
                mgContentCreate.classList.add('hidden');
            }
        }

        mgTabCreate.addEventListener('click', () => switchModelTab('create'));
        mgTabRepose.addEventListener('click', () => switchModelTab('repose'));
        setupOptionButtons(mgRatioOptions);


        async function getRandomModelPromptFromAI() {
            const apiKey = "";
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey}`;
            
            const setting = mgSettingsDropdown.value;
            let settingInstruction = "";
            switch (setting) {
                case 'flat_bg':
                    settingInstruction = " Pastikan latar belakangnya adalah latar studio yang polos atau flat berwarna netral.";
                    break;
                case 'scenic_bg':
                    settingInstruction = " Pastikan model berada di lokasi dengan background yang relevan dan menarik (bisa di dalam ruangan seperti kafe, studio, atau di luar ruangan seperti jalanan kota, taman).";
                    break;
                case 'with_product':
                    settingInstruction = " Pastikan model sedang memegang atau berinteraksi dengan sebuah produk generik yang tidak memiliki merek (contoh: smartphone, cangkir kopi, buku, tas tangan).";
                    break;
            }

            const systemPrompt = `Anda adalah asisten kreatif. Buatkan satu deskripsi atau prompt yang detail dalam Bahasa Indonesia untuk menghasilkan foto model yang fotorealistis. Sertakan detail seperti etnis, gaya rambut, ekspresi, pakaian, dan gaya fotografi.${settingInstruction} Jawab hanya dengan promptnya saja, tanpa teks pembuka atau penutup.`;
            
            const payload = {
                contents: [{ parts: [{ text: "Buatkan satu prompt acak untuk foto model, sesuai dengan pengaturan yang diberikan." }] }],
                systemInstruction: { parts: [{ text: systemPrompt }] }
            };
            
            const response = await fetch(apiUrl, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            });

            if (!response.ok) {
                const errorData = await response.json();
                throw new Error(errorData.error?.message || `HTTP error! status: ${response.status}`);
            }
            const result = await response.json();
            return result.candidates[0].content.parts[0].text.trim();
        }

        mgRandomBtn.addEventListener('click', async () => {
            const originalBtnHTML = mgRandomBtn.innerHTML;
            mgRandomBtn.disabled = true;
            mgRandomBtn.innerHTML = `<div class="loader-icon w-5 h-5"></div>`;
            lucide.createIcons();
            try {
                const randomPrompt = await getRandomModelPromptFromAI();
                mgPromptInput.value = randomPrompt;
            } catch (error) {
                console.error("Error generating random prompt:", error);
                mgPromptInput.value = "Gagal membuat prompt acak. Silakan coba lagi.";
            } finally {
                mgRandomBtn.disabled = false;
                mgRandomBtn.innerHTML = originalBtnHTML;
                 lucide.createIcons();
            }
        });

        mgGenerateBtn.addEventListener('click', async () => {
            const prompt = mgPromptInput.value.trim();
            if (!prompt) return;

            const aspectRatio = mgRatioOptions.querySelector('.selected').dataset.value;
            const aspectClass = getAspectRatioClass(aspectRatio);

            mgResultsContainer.classList.remove('hidden');
            mgResultsGrid.innerHTML = '';
            mgGenerateBtn.disabled = true;
            mgGenerateBtn.innerHTML = `<div class="loader-icon !border-l-white w-5 h-5"></div><span class="ml-2">Membuat Model...</span>`;

            for (let i = 1; i <= 4; i++) {
                const card = document.createElement('div');
                card.id = `mg-card-${i}`;
                card.className = `card overflow-hidden transition-all ${aspectClass} bg-gray-100 flex items-center justify-center`;
                card.innerHTML = `<div class="loader-icon w-10 h-10"></div>`;
                mgResultsGrid.appendChild(card);
            }
            lucide.createIcons();

            const generationPromises = [1, 2, 3, 4].map(i => generateModelImage(i, prompt, aspectRatio));
            await Promise.allSettled(generationPromises);
            
            mgGenerateBtn.disabled = false;
            mgGenerateBtn.innerHTML = `<i data-lucide="sparkles" class="w-5 h-5 mr-2"></i><span>Buat 4 Foto Model</span>`;
            lucide.createIcons();
        });

        async function generateModelImage(id, userPrompt, aspectRatio) {
            const card = document.getElementById(`mg-card-${id}`);
            try {
                const apiKey = "";
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/imagen-3.0-generate-002:predict?key=${apiKey}`;
                const finalPrompt = `${userPrompt}, photorealistic, 8k, high detail, professional photoshoot, sharp focus`;
                const payload = { instances: [{ prompt: finalPrompt }], parameters: { "sampleCount": 1, "aspectRatio": aspectRatio } };

                const response = await fetch(apiUrl, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload) });

                if (!response.ok) {
                    const errorBody = await response.json();
                    throw new Error(`API Error: ${response.status} - ${errorBody.error?.message || 'Unknown Error'}`);
                }

                const result = await response.json();
                if (!result.predictions || !result.predictions[0] || !result.predictions[0].bytesBase64Encoded) {
                    throw new Error("No image data in API response.");
                }
                const base64Data = result.predictions[0].bytesBase64Encoded;
                const imageUrl = `data:image/png;base64,${base64Data}`;
                const safePrompt = userPrompt.substring(0, 20).replace(/\s/g, '_');
                
                card.innerHTML = `
                    <img src="${imageUrl}" alt="Generated model" class="w-full h-full object-cover">
                    <div class="absolute bottom-2 right-2">
                        <a href="${imageUrl}" download="model_${safePrompt}_${id}.png" class="bg-teal-600 text-white p-2 rounded-full hover:bg-teal-700 inline-block" title="Unduh Gambar">
                           <i data-lucide="download" class="w-4 h-4"></i>
                        </a>
                    </div>`;
                card.classList.remove('bg-gray-100', 'flex', 'items-center', 'justify-center');
                card.classList.add('relative');
                lucide.createIcons();
            } catch (error) {
                console.error(`Error generating image for card ${id}:`, error);
                card.innerHTML = `<div class="text-xs text-red-500 p-2 text-center flex flex-col items-center justify-center"><i data-lucide="alert-triangle" class="w-8 h-8 mb-2"></i><span>Gagal memuat visual.</span></div>`;
                lucide.createIcons();
            }
        }
        
        // --- CHANGE POSE LOGIC ---
        const cpUploadBox = document.getElementById('cp-upload-box');
        const cpImageInput = document.getElementById('cp-image-input');
        const cpPreview = document.getElementById('cp-preview');
        const cpPlaceholder = document.getElementById('cp-placeholder');
        const cpRemoveBtn = document.getElementById('cp-remove-btn');
        const cpPromptInput = document.getElementById('cp-prompt-input');
        const cpRatioOptions = document.getElementById('cp-ratio-options');
        const cpGenerateBtn = document.getElementById('cp-generate-btn');
        const cpResultsContainer = document.getElementById('cp-results-container');
        const cpResultsGrid = document.getElementById('cp-results-grid');

        let cpImageData = null;

        function cpUpdateButtons() {
            cpGenerateBtn.disabled = !cpImageData || cpPromptInput.value.trim() === '';
        }
        
        document.querySelectorAll('.cp-pose-btn').forEach(btn => {
            btn.addEventListener('click', () => {
                cpPromptInput.value = btn.querySelector('span').textContent;
                cpUpdateButtons();
            });
        });
        
        setupImageUpload(cpImageInput, cpUploadBox, (data) => {
            cpImageData = data;
            cpPreview.src = data.dataUrl;
            cpPlaceholder.classList.add('hidden');
            cpPreview.classList.remove('hidden');
            cpRemoveBtn.classList.remove('hidden');
            cpUpdateButtons();
        });

        cpRemoveBtn.addEventListener('click', (e) => {
            e.stopPropagation();
            cpImageData = null;
            cpImageInput.value = '';
            cpPreview.src = '#';
            cpPreview.classList.add('hidden');
            cpPlaceholder.classList.remove('hidden');
            cpRemoveBtn.classList.add('hidden');
            cpUpdateButtons();
        });
        
        cpPromptInput.addEventListener('input', cpUpdateButtons);
        setupOptionButtons(cpRatioOptions);
        
        cpGenerateBtn.addEventListener('click', async () => {
            const originalBtnHTML = cpGenerateBtn.innerHTML;
            cpGenerateBtn.disabled = true;
            cpGenerateBtn.innerHTML = `<div class="loader-icon !border-l-white w-5 h-5"></div><span class="ml-2">Mengubah Pose...</span>`;
            
            const aspectRatio = cpRatioOptions.querySelector('.selected').dataset.value;
            const aspectClass = getAspectRatioClass(aspectRatio);

            cpResultsContainer.classList.remove('hidden');
            cpResultsGrid.className = `grid grid-cols-1 sm:grid-cols-2 gap-4 md:gap-6`;
            cpResultsGrid.innerHTML = '';

            for (let i = 1; i <= 4; i++) {
                const card = document.createElement('div');
                card.id = `cp-card-${i}`;
                card.className = `card overflow-hidden transition-all ${aspectClass} bg-gray-100 flex items-center justify-center`;
                card.innerHTML = `<div class="loader-icon w-10 h-10"></div>`;
                cpResultsGrid.appendChild(card);
            }
            lucide.createIcons();

            const generationPromises = [1, 2, 3, 4].map(i => generateReposedImage(i, aspectRatio));
            await Promise.allSettled(generationPromises);

            cpGenerateBtn.disabled = false;
            cpGenerateBtn.innerHTML = originalBtnHTML;
            lucide.createIcons();
        });

        async function generateReposedImage(id, aspectRatio) {
            const card = document.getElementById(`cp-card-${id}`);
            try {
                const apiKey = "";
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-image-preview:generateContent?key=${apiKey}`;
                const prompt = `Recreate the person in this image but change their pose to "${cpPromptInput.value.trim()}". Maintain the same person, clothing, and background. Variation ${id}. The final image must have an aspect ratio of ${aspectRatio}.`;
                
                const payload = {
                    contents: [{ parts: [ { text: prompt }, { inlineData: { mimeType: cpImageData.mimeType, data: cpImageData.base64 } } ] }],
                    generationConfig: { responseModalities: ['IMAGE'] },
                };

                const response = await fetch(apiUrl, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload) });
                if (!response.ok) throw new Error(`API error: ${response.statusText}`);
                
                const result = await response.json();
                const base64Data = result?.candidates?.[0]?.content?.parts?.find(p => p.inlineData)?.inlineData?.data;

                if (base64Data) {
                    const imageUrl = `data:image/png;base64,${base64Data}`;
                    card.innerHTML = `
                        <img src="${imageUrl}" class="w-full h-full object-cover">
                        <div class="absolute bottom-2 right-2">
                            <a href="${imageUrl}" download="pose_change_${id}.png" class="bg-teal-600 text-white p-2 rounded-full hover:bg-teal-700 inline-block" title="Unduh Gambar">
                                <i data-lucide="download" class="w-4 h-4"></i>
                            </a>
                        </div>`;
                    card.classList.remove('bg-gray-100', 'flex', 'items-center', 'justify-center');
                    card.classList.add('relative');
                } else {
                    throw new Error("Respon tidak valid dari API.");
                }
            } catch (error) {
                console.error(`Error for reposed card ${id}:`, error);
                card.innerHTML = `<div class="text-xs text-red-500 p-2 text-center">Gagal memuat visual.</div>`;
            } finally {
                lucide.createIcons();
            }
        }
        
        // --- EDIT FOTO LOGIC ---
        const efImageInput = document.getElementById('ef-image-input');
        const efUploadArea = document.getElementById('ef-upload-area');
        const efCanvasContainer = document.getElementById('ef-canvas-container');
        const efImageCanvas = document.getElementById('ef-image-canvas');
        const efDrawCanvas = document.getElementById('ef-draw-canvas');
        const efControlsPanel = document.getElementById('ef-controls-panel');
        const efBrushSize = document.getElementById('ef-brush-size');
        const efClearBtn = document.getElementById('ef-clear-btn');
        const efPromptInput = document.getElementById('ef-prompt-input');
        const efGenerateBtn = document.getElementById('ef-generate-btn');
        const efRemoveImageBtn = document.getElementById('ef-remove-image-btn');
        const efDownloadBtn = document.getElementById('ef-download-btn');
        const efTopControls = document.getElementById('ef-top-controls');
        const efUndoBtn = document.getElementById('ef-undo-btn');
        const efRedoBtn = document.getElementById('ef-redo-btn');
        const efTabEdit = document.getElementById('ef-tab-edit');
        const efTabEnhance = document.getElementById('ef-tab-enhance');
        const efContentEdit = document.getElementById('ef-content-edit');
        const efContentEnhance = document.getElementById('ef-content-enhance');

        function switchEditTab(tabName) {
            if(tabName === 'edit') {
                efTabEdit.classList.add('border-teal-500', 'text-teal-600');
                efTabEdit.classList.remove('border-transparent', 'text-gray-500', 'hover:text-gray-700', 'hover:border-gray-300');
                efTabEnhance.classList.add('border-transparent', 'text-gray-500', 'hover:text-gray-700', 'hover:border-gray-300');
                efTabEnhance.classList.remove('border-teal-500', 'text-teal-600');
                efContentEdit.classList.remove('hidden');
                efContentEnhance.classList.add('hidden');
            } else {
                efTabEnhance.classList.add('border-teal-500', 'text-teal-600');
                efTabEnhance.classList.remove('border-transparent', 'text-gray-500', 'hover:text-gray-700', 'hover:border-gray-300');
                efTabEdit.classList.add('border-transparent', 'text-gray-500', 'hover:text-gray-700', 'hover:border-gray-300');
                efTabEdit.classList.remove('border-teal-500', 'text-teal-600');
                efContentEnhance.classList.remove('hidden');
                efContentEdit.classList.add('hidden');
            }
        }
        
        efTabEdit.addEventListener('click', () => switchEditTab('edit'));
        efTabEnhance.addEventListener('click', () => switchEditTab('enhance'));
        
        let efUploadedImageData = { base64: null, mimeType: null };
        let efOriginalImage = null;
        let efHistory = [];
        let efHistoryIndex = -1;
        
        const imageCtx = efImageCanvas.getContext('2d');
        const drawCtx = efDrawCanvas.getContext('2d');
        let isDrawing = false;
        let lastX = 0;
        let lastY = 0;

        function efUpdateHistoryButtons() {
            efUndoBtn.disabled = efHistoryIndex <= 0;
            efRedoBtn.disabled = efHistoryIndex >= efHistory.length - 1;
            efClearBtn.classList.toggle('hidden', efHistoryIndex <= 0);
        }

        function efSaveState() {
            if (efHistoryIndex < efHistory.length - 1) {
                efHistory.splice(efHistoryIndex + 1);
            }
            efHistory.push(drawCtx.getImageData(0, 0, efDrawCanvas.width, efDrawCanvas.height));
            efHistoryIndex = efHistory.length - 1;
            efUpdateHistoryButtons();
        }

        function efUndo() {
            if (efHistoryIndex > 0) {
                efHistoryIndex--;
                drawCtx.putImageData(efHistory[efHistoryIndex], 0, 0);
                efUpdateHistoryButtons();
            }
        }

        function efRedo() {
            if (efHistoryIndex < efHistory.length - 1) {
                efHistoryIndex++;
                drawCtx.putImageData(efHistory[efHistoryIndex], 0, 0);
                efUpdateHistoryButtons();
            }
        }

        efUndoBtn.addEventListener('click', efUndo);
        efRedoBtn.addEventListener('click', efRedo);

        // Fix for mobile download issue where clicking the link refreshes the page
        efDownloadBtn.addEventListener('click', (e) => {
            // Only act if the href is a valid data URL
            if (e.currentTarget.href && e.currentTarget.href.startsWith('data:image')) {
                e.preventDefault();
                // Create a temporary link, click it programmatically, then remove it
                const tempLink = document.createElement('a');
                tempLink.href = e.currentTarget.href;
                tempLink.setAttribute('download', e.currentTarget.getAttribute('download') || 'solophoto_edit.png');
                document.body.appendChild(tempLink);
                tempLink.click();
                document.body.removeChild(tempLink);
            } else {
                 // Prevent default behavior for placeholder hrefs like '#'
                e.preventDefault();
            }
        });

        function efClearCanvas() {
            drawCtx.clearRect(0, 0, efDrawCanvas.width, efDrawCanvas.height);
            efSaveState();
        }

        function efSetupCanvas(imageSrc) {
            efOriginalImage = new Image();
            efOriginalImage.onload = () => {
                const workspace = efCanvasContainer.parentElement;
                const maxWidth = workspace.clientWidth;
                const maxHeight = 640; 

                let newWidth = efOriginalImage.naturalWidth;
                let newHeight = efOriginalImage.naturalHeight;

                if (newWidth > maxWidth) {
                    const ratio = maxWidth / newWidth;
                    newWidth = maxWidth;
                    newHeight *= ratio;
                }
                if (newHeight > maxHeight) {
                    const ratio = maxHeight / newHeight;
                    newHeight = maxHeight;
                    newWidth *= ratio;
                }
                
                workspace.style.width = `${newWidth}px`;
                workspace.style.height = `${newHeight}px`;
                workspace.style.minHeight = '0px';

                efImageCanvas.width = newWidth;
                efImageCanvas.height = newHeight;
                efDrawCanvas.width = newWidth;
                efDrawCanvas.height = newHeight;
                
                imageCtx.clearRect(0, 0, newWidth, newHeight);
                imageCtx.drawImage(efOriginalImage, 0, 0, newWidth, newHeight);

                efUploadArea.style.display = 'none';
                efCanvasContainer.classList.remove('hidden');
                efControlsPanel.classList.remove('hidden');
                efTopControls.classList.add('flex');
                efTopControls.classList.remove('hidden');
                efDownloadBtn.href = imageSrc;
                efDownloadBtn.classList.remove('hidden');

                // Clear drawing history for new image
                efHistory = [];
                efHistoryIndex = -1;
                efClearCanvas();
            };
            efOriginalImage.src = imageSrc;
        }

        function getMousePos(canvas, evt) {
            const rect = canvas.getBoundingClientRect();
            if (evt.touches) { // Touch event
                return {
                    x: evt.touches[0].clientX - rect.left,
                    y: evt.touches[0].clientY - rect.top
                };
            }
            // Mouse event
            return {
                x: evt.clientX - rect.left,
                y: evt.clientY - rect.top
            };
        }

        function startDrawing(e) {
            isDrawing = true;
            const { x, y } = getMousePos(efDrawCanvas, e);
            [lastX, lastY] = [x, y];
        }

        function draw(e) {
            if (!isDrawing) return;
            const { x, y } = getMousePos(efDrawCanvas, e);
            drawCtx.beginPath();
            drawCtx.strokeStyle = 'rgba(255, 0, 150, 1)';
            drawCtx.lineWidth = efBrushSize.value;
            drawCtx.lineCap = 'round';
            drawCtx.lineJoin = 'round';
            drawCtx.moveTo(lastX, lastY);
            drawCtx.lineTo(x, y);
            drawCtx.stroke();
            [lastX, lastY] = [x, y];
        }

        function stopDrawing() {
            if (isDrawing) {
                isDrawing = false;
                efSaveState();
            }
        }

        efDrawCanvas.addEventListener('mousedown', startDrawing);
        efDrawCanvas.addEventListener('mousemove', draw);
        efDrawCanvas.addEventListener('mouseup', stopDrawing);
        efDrawCanvas.addEventListener('mouseout', stopDrawing);

        efDrawCanvas.addEventListener('touchstart', (e) => { e.preventDefault(); startDrawing(e); }, { passive: false });
        efDrawCanvas.addEventListener('touchmove', (e) => { e.preventDefault(); draw(e); }, { passive: false });
        efDrawCanvas.addEventListener('touchend', (e) => { e.preventDefault(); stopDrawing(); });

        efClearBtn.addEventListener('click', efClearCanvas);

        function efUpdateGenerateButtonState() {
            const hasImage = !!efUploadedImageData.base64;
            const hasPrompt = efPromptInput.value.trim().length > 0;
            efGenerateBtn.disabled = !hasImage || !hasPrompt;
        }
        
        efPromptInput.addEventListener('input', efUpdateGenerateButtonState);
        
        setupImageUpload(efImageInput, efUploadArea, (data) => {
             if (data) {
                efUploadedImageData = { base64: data.base64, mimeType: data.mimeType };
                efSetupCanvas(data.dataUrl);
            }
            efUpdateGenerateButtonState();
        });
        
        efRemoveImageBtn.addEventListener('click', () => {
            efUploadedImageData = { base64: null, mimeType: null };
            efImageInput.value = '';
            const workspace = efCanvasContainer.parentElement;
            workspace.style.height = '';
            workspace.style.width = '100%';
            workspace.style.minHeight = '350px';
            
            efUploadArea.style.display = 'flex';
            efCanvasContainer.classList.add('hidden');
            efControlsPanel.classList.add('hidden');
            efTopControls.classList.add('hidden');
            efTopControls.classList.remove('flex');
            efDownloadBtn.classList.add('hidden');
            drawCtx.clearRect(0, 0, efDrawCanvas.width, efDrawCanvas.height);
            imageCtx.clearRect(0, 0, efImageCanvas.width, efImageCanvas.height);
            efHistory = [];
            efHistoryIndex = -1;
            efUpdateHistoryButtons();
            efUpdateGenerateButtonState();
        });

        efGenerateBtn.addEventListener('click', async () => {
             if (!efUploadedImageData.base64 || !efPromptInput.value.trim()) return;
            const originalBtnHTML = efGenerateBtn.innerHTML;
            efGenerateBtn.disabled = true;
            efGenerateBtn.innerHTML = `<div class="loader-icon !border-l-white w-5 h-5"></div>`;
            lucide.createIcons();

            try {
                const maskCanvas = document.createElement('canvas');
                maskCanvas.width = efOriginalImage.naturalWidth;
                maskCanvas.height = efOriginalImage.naturalHeight;
                const maskCtx = maskCanvas.getContext('2d');
                maskCtx.drawImage(efDrawCanvas, 0, 0, efOriginalImage.naturalWidth, efOriginalImage.naturalHeight);
                const maskBase64 = maskCanvas.toDataURL('image/png').split(',')[1];

                const userInstruction = efPromptInput.value.trim();
                const finalPrompt = `Inpaint the masked area of the image based on this instruction: "${userInstruction}". Do not change any other part of the image. The result should be photorealistic and seamlessly blended with the original image.`;
                const apiKey = "";
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-image-preview:generateContent?key=${apiKey}`;
                
                const payload = {
                    contents: [{
                        parts: [
                            { text: finalPrompt },
                            { inlineData: { mimeType: efUploadedImageData.mimeType, data: efUploadedImageData.base64 } },
                            { inlineData: { mimeType: 'image/png', data: maskBase64 } }
                        ]
                    }],
                    generationConfig: { responseModalities: ['IMAGE'] },
                };
                
                const response = await fetch(apiUrl, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload) });
                if (!response.ok) {
                    const errorBody = await response.json();
                    throw new Error(`API Error: ${response.status} - ${errorBody.error?.message || 'Unknown Error'}`);
                }
                const result = await response.json();
                const base64Data = result?.candidates?.[0]?.content?.parts?.find(p => p.inlineData)?.inlineData?.data;
                
                if (base64Data) {
                    const imageUrl = `data:image/png;base64,${base64Data}`;
                    efUploadedImageData = { base64: base64Data, mimeType: 'image/png' };
                    efSetupCanvas(imageUrl);
                } else {
                    throw new Error("Respon tidak valid dari API.");
                }

            } catch (error) {
                console.error("Error editing photo:", error);
                alert(`Gagal mengedit foto: ${error.message}`);
            } finally {
                efGenerateBtn.disabled = false;
                efGenerateBtn.innerHTML = originalBtnHTML;
                lucide.createIcons();
            }
        });
        
        // --- PHOTO FIXER (PERBAIKI FOTO) LOGIC ---
        const pfImageInput = document.getElementById('pf-image-input');
        const pfUploadBox = document.getElementById('pf-upload-box');
        const pfPreview = document.getElementById('pf-preview');
        const pfPlaceholder = document.getElementById('pf-placeholder');
        const pfRemoveBtn = document.getElementById('pf-remove-btn');
        const pfRatioOptions = document.getElementById('pf-ratio-options');
        const pfGenerateBtn = document.getElementById('pf-generate-btn');
        const pfResultsContainer = document.getElementById('pf-results-container');
        const pfResultsGrid = document.getElementById('pf-results-grid');

        let pfImageData = null;

        function pfUpdateButtons() {
            pfGenerateBtn.disabled = !pfImageData;
        }

        setupImageUpload(pfImageInput, pfUploadBox, (data) => {
            pfImageData = data;
            pfPreview.src = data.dataUrl;
            pfPlaceholder.classList.add('hidden');
            pfPreview.classList.remove('hidden');
            pfRemoveBtn.classList.remove('hidden');
            pfUpdateButtons();
        });

        pfRemoveBtn.addEventListener('click', (e) => {
            e.stopPropagation();
            pfImageData = null;
            pfImageInput.value = '';
            pfPreview.src = '#';
            pfPreview.classList.add('hidden');
            pfPlaceholder.classList.remove('hidden');
            pfRemoveBtn.classList.add('hidden');
            pfUpdateButtons();
        });
        
        setupOptionButtons(pfRatioOptions);

        pfGenerateBtn.addEventListener('click', async () => {
            const originalBtnHTML = pfGenerateBtn.innerHTML;
            pfGenerateBtn.disabled = true;
            pfGenerateBtn.innerHTML = `<div class="loader-icon !border-l-white w-5 h-5"></div><span class="ml-2">Memperbaiki...</span>`;
            
            const aspectRatio = pfRatioOptions.querySelector('.selected').dataset.value;
            const aspectClass = getAspectRatioClass(aspectRatio);

            pfResultsContainer.classList.remove('hidden');
            pfResultsGrid.className = `grid grid-cols-1 sm:grid-cols-2 gap-4 md:gap-6`;
            pfResultsGrid.innerHTML = '';

            for (let i = 1; i <= 4; i++) {
                const card = document.createElement('div');
                card.id = `pf-card-${i}`;
                card.className = `card overflow-hidden transition-all ${aspectClass} bg-gray-100 flex items-center justify-center`;
                card.innerHTML = `<div class="loader-icon w-10 h-10"></div>`;
                pfResultsGrid.appendChild(card);
            }
            lucide.createIcons();

            const generationPromises = [1, 2, 3, 4].map(i => generateEnhancedImage(i, aspectRatio));
            await Promise.allSettled(generationPromises);

            pfGenerateBtn.disabled = false;
            pfGenerateBtn.innerHTML = originalBtnHTML;
            lucide.createIcons();
        });

        async function generateEnhancedImage(id, aspectRatio) {
            const card = document.getElementById(`pf-card-${id}`);
            try {
                const apiKey = "";
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-image-preview:generateContent?key=${apiKey}`; 
                
                const prompt = `Enhance this image to professional studio portrait quality. Improve lighting, sharpness, and color balance. Make it look like a high-resolution photograph. Variation ${id}. The final image must have an aspect ratio of ${aspectRatio}.`;

                const payload = {
                    contents: [{
                        parts: [
                            { text: prompt },
                            { inlineData: { mimeType: pfImageData.mimeType, data: pfImageData.base64 } }
                        ]
                    }],
                    generationConfig: { responseModalities: ['IMAGE'] },
                };

                const response = await fetch(apiUrl, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload) });
                if (!response.ok) throw new Error(`API error: ${response.statusText}`);

                const result = await response.json();
                const base64Data = result?.candidates?.[0]?.content?.parts?.find(p => p.inlineData)?.inlineData?.data;

                if (base64Data) {
                    const imageUrl = `data:image/png;base64,${base64Data}`;
                    card.innerHTML = `
                        <img src="${imageUrl}" class="w-full h-full object-cover">
                        <div class="absolute bottom-2 right-2">
                            <a href="${imageUrl}" download="perbaikan_foto_${id}.png" class="bg-teal-600 text-white p-2 rounded-full hover:bg-teal-700 inline-block" title="Unduh Gambar">
                                <i data-lucide="download" class="w-4 h-4"></i>
                            </a>
                        </div>`;
                    card.classList.remove('bg-gray-100', 'flex', 'items-center', 'justify-center');
                    card.classList.add('relative');
                } else {
                    throw new Error("Respon tidak valid dari API.");
                }
            } catch (error) {
                console.error(`Error for enhanced photo card ${id}:`, error);
                card.innerHTML = `<div class="text-xs text-red-500 p-2 text-center">Gagal memuat visual.</div>`;
            } finally {
                lucide.createIcons();
            }
        }

        // --- BIKIN BANNER IKLAN LOGIC ---
        const bnrImageInput = document.getElementById('bnr-image-input');
        const bnrImageUploadArea = document.getElementById('bnr-image-upload-area');
        const bnrImagePreviewContainer = document.getElementById('bnr-image-preview-container');
        const bnrImagePreview = document.getElementById('bnr-image-preview');
        const bnrRemoveImageBtn = document.getElementById('bnr-remove-image-btn');
        const bnrDescInput = document.getElementById('bnr-desc-input');
        const bnrDescLoader = document.getElementById('bnr-desc-loader');
        const bnrStyleInput = document.getElementById('bnr-style-input');
        const bnrStyleLoader = document.getElementById('bnr-style-loader');
        const bnrRatioOptions = document.getElementById('bnr-ratio-options');
        const bnrGenerateBtn = document.getElementById('bnr-generate-btn');
        const bnrResultsContainer = document.querySelector('#content-combine-text .flex-grow');
        const bnrResultsGrid = document.getElementById('bnr-results-grid');

        let bnrUploadedImageData = { base64: null, mimeType: null };

        function bnrUpdateGenerateButtonState() {
            const hasImage = !!bnrUploadedImageData.base64;
            const hasDesc = bnrDescInput.value.trim().length > 0;
            const hasStyle = bnrStyleInput.value.trim().length > 0;
            bnrGenerateBtn.disabled = !hasImage || !hasDesc || !hasStyle;
        }
        
        async function bnrAutoGenerateStyle() {
            if (!bnrUploadedImageData.base64 || !bnrDescInput.value) return;

            bnrStyleLoader.classList.remove('hidden');
            lucide.createIcons();
            bnrStyleInput.value = 'AI sedang membuat style...';
            bnrStyleInput.disabled = true;
            
            try {
                const apiKey = "";
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey}`;
                const systemPrompt = `You are a professional graphic designer. Based on the image and the banner text, suggest one professional design style description in Indonesian for an AI image generator. For example: 'Minimalis dan bersih dengan font sans-serif tebal berwarna putih di tengah bawah', 'Gaya retro dengan font script dan efek cahaya neon'. Keep it short and descriptive.`;
                const userQuery = `Image analysis: A banner with the text "${bnrDescInput.value.trim()}". Suggest a style.`
                const payload = { contents: [{ parts: [ { text: userQuery }, { inlineData: { mimeType: bnrUploadedImageData.mimeType, data: bnrUploadedImageData.base64 } } ] }], systemInstruction: { parts: [{ text: systemPrompt }] } };

                const response = await fetch(apiUrl, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload) });
                if (!response.ok) { throw new Error(`HTTP error! status: ${response.status}`); }
                const result = await response.json();
                const style = result.candidates[0].content.parts[0].text;
                bnrStyleInput.value = style.trim().replace(/["*]/g, '');
            } catch(error) {
                console.error("Error generating banner style:", error);
                bnrStyleInput.value = "Gagal membuat style. Silakan isi manual.";
            } finally {
                bnrStyleLoader.classList.add('hidden');
                bnrStyleInput.disabled = false;
                bnrUpdateGenerateButtonState();
            }
        }

        async function bnrAutoGenerateDescription() {
            if (!bnrUploadedImageData.base64) return;
            bnrDescLoader.classList.remove('hidden');
            lucide.createIcons();
            bnrDescInput.value = 'AI sedang membuat deskripsi...';
            bnrDescInput.disabled = true;
            try {
                const apiKey = "";
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey}`;
                const systemPrompt = `You are a creative copywriter. Analyze the image and write a very short, catchy text for an advertising banner in Indonesian. The text must be a maximum of 10 words.`;
                const payload = { contents: [{ parts: [ { text: "Buatkan teks banner singkat untuk gambar ini." }, { inlineData: { mimeType: bnrUploadedImageData.mimeType, data: bnrUploadedImageData.base64 } } ] }], systemInstruction: { parts: [{ text: systemPrompt }] } };
                const response = await fetch(apiUrl, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload) });
                if (!response.ok) { throw new Error(`HTTP error! status: ${response.status}`); }
                const result = await response.json();
                const description = result.candidates[0].content.parts[0].text;
                bnrDescInput.value = description.trim().replace(/["*]/g, '');
                await bnrAutoGenerateStyle(); // Call style generation after description is done
            } catch(error) {
                console.error("Error generating banner description:", error);
                bnrDescInput.value = "Gagal membuat deskripsi. Coba lagi.";
                bnrDescLoader.classList.add('hidden');
                bnrDescInput.disabled = false;
            } finally {
                bnrDescLoader.classList.add('hidden');
                bnrDescInput.disabled = false;
                bnrUpdateGenerateButtonState();
            }
        }

        setupImageUpload(bnrImageInput, bnrImageUploadArea, (data) => {
             bnrUploadedImageData = { base64: data.base64, mimeType: data.mimeType };
             document.getElementById('bnr-image-preview').src = data.dataUrl;
             document.getElementById('bnr-image-upload-area').classList.add('hidden');
             document.getElementById('bnr-image-preview-container').classList.remove('hidden');
             bnrUpdateGenerateButtonState();
             bnrAutoGenerateDescription();
        });
        
        document.getElementById('bnr-remove-image-btn').addEventListener('click', () => {
             bnrUploadedImageData = { base64: null, mimeType: null };
             document.getElementById('bnr-image-preview-container').classList.add('hidden');
             document.getElementById('bnr-image-upload-area').classList.remove('hidden');
             bnrDescInput.value = '';
             bnrStyleInput.value = '';
             bnrUpdateGenerateButtonState();
        });
        
        bnrDescInput.addEventListener('input', bnrUpdateGenerateButtonState);
        bnrStyleInput.addEventListener('input', bnrUpdateGenerateButtonState);
        setupOptionButtons(bnrRatioOptions);
        
        bnrGenerateBtn.addEventListener('click', async () => {
            if (!bnrUploadedImageData.base64) return;
            bnrGenerateBtn.disabled = true;
            bnrGenerateBtn.innerHTML = `<div class="loader-icon !border-l-white w-5 h-5"></div><span class="ml-2">Merancang Banner...</span>`;
            
            const aspectRatio = bnrRatioOptions.querySelector('.selected').dataset.value;
            const aspectClass = getAspectRatioClass(aspectRatio);

            bnrResultsContainer.classList.remove('hidden');
            bnrResultsGrid.className = 'grid grid-cols-1 md:grid-cols-2 gap-6';
            bnrResultsGrid.innerHTML = '';
            for(let i=1; i<=4; i++) {
                const card = document.createElement('div');
                card.id = `bnr-card-${i}`;
                card.className = `card overflow-hidden transition-all ${aspectClass} bg-gray-100 flex items-center justify-center`;
                card.innerHTML = `<div class="loader-icon w-10 h-10"></div>`;
                bnrResultsGrid.appendChild(card);
            }
            lucide.createIcons();
            
            try {
                const bannerVariations = await getBannerVariations();
                const generationPromises = bannerVariations.slice(0, 4).map((idea, index) => generateSingleBannerImage(index + 1, idea.title, idea.prompt, aspectRatio));
                await Promise.allSettled(generationPromises);
            } catch (error) {
                console.error("Error during banner generation process:", error);
                bnrResultsGrid.innerHTML = `<div class="col-span-1 md:col-span-2 text-center py-10 text-red-500"><p>Terjadi kesalahan saat membuat variasi banner: ${error.message}</p></div>`;
            } finally {
                bnrGenerateBtn.disabled = false;
                bnrGenerateBtn.innerHTML = `<span>Buat 4 Variasi Banner</span>`;
            }
        });
        
        async function getBannerVariations() {
            const apiKey = "";
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey}`;
            const systemPrompt = `You are an expert AI graphic designer for social media banners. Analyze the provided image, banner text, and requested style. Generate 4 distinct banner design ideas. For each, provide a JSON object with a 'title' (short name in Indonesian) and a 'prompt' (detailed English prompt for an image generation AI). The prompt MUST instruct the AI to creatively place the provided banner text onto the original image, adhering to the specified style. Describe text placement, font style, color scheme, and any subtle background enhancements or graphic elements. The original image content must not be altered, only enhanced. Respond ONLY with a valid JSON array of 4 objects.`;
            let userQuery = `Banner Text: "${bnrDescInput.value.trim()}"\nDesign Style: "${bnrStyleInput.value.trim()}"`;
            
            const parts = [{ text: userQuery }, { inlineData: { mimeType: bnrUploadedImageData.mimeType, data: bnrUploadedImageData.base64 } }];
            const payload = { contents: [{ parts: parts }], systemInstruction: { parts: [{ text: systemPrompt }] }, generationConfig: { responseMimeType: "application/json", responseSchema: { type: "ARRAY", items: { type: "OBJECT", properties: { "title": { "type": "STRING" }, "prompt": { "type": "STRING" } }, required: ["title", "prompt"] } } } };
            
            const response = await fetch(apiUrl, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload) });
            if (!response.ok) { const errorData = await response.json(); throw new Error(errorData.error?.message || `HTTP error! status: ${response.status}`); }
            const result = await response.json();
            const jsonText = result.candidates[0].content.parts[0].text;
            return JSON.parse(jsonText);
        }

        async function generateSingleBannerImage(id, title, prompt, aspectRatio) {
            const card = document.getElementById(`bnr-card-${id}`);
            if (!card) return;
            
             try {
                const apiKey = "";
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-image-preview:generateContent?key=${apiKey}`;
                
                const finalPrompt = `${prompt}. The text to add is: "${bnrDescInput.value.trim()}". Ensure the text is clearly visible and beautifully integrated. Aspect ratio: ${aspectRatio}.`;
                const parts = [{ text: finalPrompt }, { inlineData: { mimeType: bnrUploadedImageData.mimeType, data: bnrUploadedImageData.base64 } }];
                
                const payload = { contents: [{ parts: parts }], generationConfig: { responseModalities: ['IMAGE'] }, };
                const response = await fetch(apiUrl, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload) });
                if (!response.ok) throw new Error(`API error: ${response.statusText}`);
                const result = await response.json();
                const base64Data = result?.candidates?.[0]?.content?.parts?.find(p => p.inlineData)?.inlineData?.data;

                if (base64Data) {
                    const imageUrl = `data:image/png;base64,${base64Data}`;
                    const safeTitle = title.replace(/[^a-z0-9]/gi, '_').toLowerCase();
                    
                    card.innerHTML = `<div class="relative w-full h-full group">
                        <img src="${imageUrl}" class="w-full h-full object-cover rounded-md" alt="${title}">
                        <div class="absolute bottom-2 right-2">
                            <a href="${imageUrl}" download="banner_${id}_${safeTitle}.png" class="bg-teal-600 text-white p-2 rounded-full hover:bg-teal-700" title="Unduh Gambar">
                                <i data-lucide="download" class="w-4 h-4"></i>
                            </a>
                        </div>
                    </div>`;
                    lucide.createIcons();
                } else { throw new Error("Respon tidak valid dari API."); }
            } catch (error) {
                console.error(`Error for banner card ${id}:`, error);
                card.innerHTML = `<div class="text-xs text-red-500 p-2 text-center">Gagal memuat visual.</div>`;
            }
        }
        
        function getAspectRatioClass(ratio) {
            switch (ratio) {
                case '1:1': return 'aspect-square';
                case '16:9': return 'aspect-video';
                case '9:16': return 'aspect-[9/16]';
                case '4:3': return 'aspect-[4/3]';
                case '3:4': return 'aspect-[3/4]';
                default: return 'aspect-square';
            }
        }

        // --- INITIALIZE ON LOAD ---
        
        lucide.createIcons();
        switchTab('product'); // Set initial active tab
        
   document.getElementById('footer-text').innerHTML = '© 2025. SoloPhoto v1.7 by Ichsan Labs';


    });
    </script>

</body>
</html>
