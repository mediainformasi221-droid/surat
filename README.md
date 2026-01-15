<!doctype html>
<html lang="id" class="h-full">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Sistem Pengiriman Surat - KPKNL Serang</title>
  
  <!-- Pastikan koneksi CDN menggunakan HTTPS -->
  <script src="https://cdn.tailwindcss.com"></script>
  
  <!-- Pustaka untuk PDF -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  
  <style>
    /* Menggunakan font sistem sebagai cadangan jika Google Fonts gagal dimuat di GitHub */
    @import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@300;400;500;600;700;800&display=swap');
    
    :root {
      --primary: #0f172a;
      --accent: #4f46e5;
      --bg-soft: #fcfcfd;
    }

    * { 
      font-family: 'Plus Jakarta Sans', -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; 
      box-sizing: border-box; 
    }
    
    body {
      background-color: var(--bg-soft);
      background-image: 
        radial-gradient(at 0% 0%, rgba(79, 70, 229, 0.05) 0px, transparent 50%),
        radial-gradient(at 100% 100%, rgba(59, 130, 246, 0.05) 0px, transparent 50%);
      color: #1e293b;
      margin: 0;
    }

    /* Modern Card Style */
    .modern-card {
      background: rgba(255, 255, 255, 0.8);
      backdrop-filter: blur(12px);
      -webkit-backdrop-filter: blur(12px); /* Dukungan Safari */
      border: 1px solid rgba(226, 232, 240, 0.6);
      box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.02), 0 10px 15px -3px rgba(0, 0, 0, 0.03);
      transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    }

    .modern-card:hover {
      box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.05), 0 8px 10px -6px rgba(0, 0, 0, 0.05);
    }

    .toast {
      position: fixed; bottom: 32px; right: 32px;
      padding: 1.25rem 2rem; border-radius: 20px;
      color: white; z-index: 1000; 
      box-shadow: 0 25px 50px -12px rgba(0,0,0,0.25);
      animation: slideUp 0.5s cubic-bezier(0.16, 1, 0.3, 1);
      display: flex; align-items: center; gap: 12px;
      font-weight: 600; font-size: 0.875rem;
    }

    @keyframes slideUp { from { transform: translateY(100%); opacity: 0; } to { transform: translateY(0); opacity: 1; } }
    
    .tab-btn {
      position: relative;
      padding: 1rem 0.5rem;
      font-weight: 600;
      color: #94a3b8;
      transition: all 0.2s;
    }
    .tab-btn:hover { color: #475569; }
    .tab-btn.active { color: var(--accent); }
    .tab-btn.active::after {
      content: '';
      position: absolute; bottom: 0; left: 0; right: 0;
      height: 3px; background: var(--accent);
      border-radius: 10px;
    }

    .loader-ring {
      display: inline-block; width: 1.5rem; height: 1.5rem;
      border: 3px solid rgba(79, 70, 229, 0.1); border-radius: 50%;
      border-top-color: var(--accent); animation: spin 0.8s ease-in-out infinite;
    }
    @keyframes spin { to { transform: rotate(360deg); } }

    /* Laporan / Paper Preview */
    .paper-preview {
      background: white;
      width: 210mm;
      margin: 0 auto;
      min-height: 297mm;
      padding: 2.5cm;
      box-shadow: 0 0 40px rgba(0,0,0,0.05);
      color: #000;
      font-family: "Times New Roman", Times, serif;
    }
    
    .table-laporan { width: 100%; border-collapse: collapse; margin-top: 1.5rem; }
    .table-laporan th, .table-laporan td { border: 1px solid #000; padding: 10px; font-size: 13px; }
    .table-laporan th { background-color: #f1f5f9; text-transform: uppercase; }

    input:focus, textarea:focus, select:focus {
      outline: none;
      border-color: var(--accent);
      ring: 2px; ring-color: rgba(79, 70, 229, 0.2);
    }

    /* Print logic */
    @media print {
      body * { visibility: hidden; }
      #printableArea, #printableArea * { visibility: visible; }
      #printableArea {
        position: absolute;
        left: 0;
        top: 0;
        width: 100%;
        margin: 0 !important;
        padding: 0 !important;
        box-shadow: none !important;
      }
      @page { margin: 1.5cm; }
    }
  </style>
</head>
<body class="h-full">
  <div id="app" class="min-h-screen flex flex-col">
    
    <!-- Header / Navbar -->
    <header class="bg-white/80 backdrop-blur-md sticky top-0 z-50 no-print border-b border-slate-200/60">
      <div class="max-w-7xl mx-auto px-6 py-4 flex items-center justify-between">
        <div class="flex items-center gap-4">
          <div class="w-12 h-12 bg-indigo-600 rounded-2xl flex items-center justify-center shadow-lg shadow-indigo-600/20">
            <svg class="w-6 h-6 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M3 8l7.89 5.26a2 2 0 002.22 0L21 8M5 19h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v10a2 2 0 002 2z"></path></svg>
          </div>
          <div>
            <h1 class="text-xl font-extrabold text-slate-900 tracking-tight leading-none uppercase">Sistem Pengiriman Surat</h1>
            <div class="flex items-center gap-2 mt-1">
              <span id="syncDot" class="w-2 h-2 bg-emerald-500 rounded-full animate-pulse"></span>
              <span id="statusText" class="text-[10px] font-bold text-slate-400 uppercase tracking-widest">Sistem Aktif</span>
            </div>
          </div>
        </div>
        
        <nav class="hidden md:flex items-center gap-8">
            <button id="tabInput" class="tab-btn active text-sm uppercase tracking-wider">Input Baru</button>
            <button id="tabRekap" class="tab-btn text-sm uppercase tracking-wider">Arsip Data</button>
            <button id="tabLaporan" class="tab-btn text-sm uppercase tracking-wider">Laporan</button>
        </nav>

        <div class="flex items-center gap-3">
            <button onclick="fetchAllData()" class="p-3 text-slate-400 hover:text-indigo-600 hover:bg-indigo-50 rounded-2xl transition-all" title="Refresh">
                <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15"></path></svg>
            </button>
        </div>
      </div>
    </header>

    <main class="flex-grow max-w-7xl mx-auto w-full px-6 py-12">
      
      <!-- PAGE 1: INPUT -->
      <section id="contentInput" class="tab-content transition-all">
        <div class="max-w-4xl mx-auto grid grid-cols-1 lg:grid-cols-5 gap-10">
          <div class="lg:col-span-3 space-y-6">
            <div class="modern-card p-10 rounded-[32px]">
                <div class="mb-10">
                    <h2 class="text-2xl font-extrabold text-slate-900">Registrasi Surat</h2>
                    <p class="text-slate-500 text-sm mt-1">Input data pengiriman surat hari ini secara detail.</p>
                </div>
                
                <form id="formPengiriman" class="space-y-6">
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                        <div class="space-y-2">
                            <label class="text-xs font-bold text-slate-500 uppercase tracking-wider ml-1">Nomor Surat</label>
                            <input type="text" id="nomorSurat" required class="w-full px-6 py-4 bg-slate-50/50 border border-slate-200 rounded-2xl focus:bg-white" placeholder="S-000/KNL.0601/2026">
                        </div>
                        <div class="space-y-2">
                            <label class="text-xs font-bold text-slate-500 uppercase tracking-wider ml-1">Tanggal</label>
                            <input type="date" id="tanggalKirim" required class="w-full px-6 py-4 bg-slate-50/50 border border-slate-200 rounded-2xl focus:bg-white">
                        </div>
                    </div>
                    
                    <div class="space-y-2">
                        <label class="text-xs font-bold text-slate-500 uppercase tracking-wider ml-1">Alamat Tujuan</label>
                        <textarea id="alamatTujuan" required rows="3" class="w-full px-6 py-4 bg-slate-50/50 border border-slate-200 rounded-2xl focus:bg-white" placeholder="Nama Instansi & Alamat lengkap..."></textarea>
                    </div>

                    <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                        <div class="space-y-2">
                            <label class="text-xs font-bold text-slate-500 uppercase tracking-wider ml-1">Jumlah (Berkas)</label>
                            <input type="number" id="jumlahSurat" required value="1" min="1" class="w-full px-6 py-4 bg-slate-50/50 border border-slate-200 rounded-2xl focus:bg-white">
                        </div>
                        <div class="space-y-2">
                            <label class="text-xs font-bold text-slate-500 uppercase tracking-wider ml-1">Petugas Pengirim</label>
                            <input type="text" id="namaPengirim" required class="w-full px-6 py-4 bg-slate-50/50 border border-slate-200 rounded-2xl focus:bg-white" placeholder="Nama Anda">
                        </div>
                    </div>

                    <button type="submit" id="submitBtn" class="w-full py-5 bg-indigo-600 hover:bg-indigo-700 text-white font-bold rounded-2xl shadow-xl shadow-indigo-100 transition-all flex items-center justify-center gap-3">
                        <span class="text-sm uppercase tracking-widest">Simpan Data</span>
                        <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 7l5 5m0 0l-5 5m5-5H6"></path></svg>
                    </button>
                </form>
            </div>
          </div>
          
          <div class="lg:col-span-2 space-y-6">
            <div class="modern-card p-8 rounded-[32px] bg-gradient-to-br from-indigo-600 to-blue-700 text-white border-none">
                <p class="text-indigo-100 text-xs font-bold uppercase tracking-widest mb-2">Statistik Hari Ini</p>
                <h3 id="todayCount" class="text-5xl font-black mb-6">0</h3>
                <p class="text-sm text-indigo-100/80 leading-relaxed">Surat telah tercatat di sistem pada hari ini.</p>
                <div class="mt-8 pt-8 border-t border-white/10 flex items-center justify-between">
                    <span class="text-xs font-semibold opacity-60">Status: Terkoneksi</span>
                    <div class="w-2 h-2 bg-emerald-400 rounded-full"></div>
                </div>
            </div>
            <div class="modern-card p-8 rounded-[32px] overflow-hidden">
                <h4 class="font-bold text-slate-800 mb-4 text-sm">Update Terakhir</h4>
                <div id="recentLog" class="space-y-4">
                    <p class="text-xs text-slate-400 italic">Menunggu data baru...</p>
                </div>
            </div>
          </div>
        </div>
      </section>

      <!-- PAGE 2: REKAP -->
      <section id="contentRekap" class="tab-content hidden animate-in fade-in duration-300">
        <div class="modern-card rounded-[32px] overflow-hidden">
          <div class="p-8 border-b border-slate-100 flex flex-col md:flex-row justify-between items-center gap-6">
            <div>
                <h2 class="text-xl font-extrabold text-slate-900">Arsip Pengiriman</h2>
                <p class="text-xs text-slate-500 mt-1">Daftar lengkap surat yang tersimpan di cloud.</p>
            </div>
            <div class="relative w-full md:w-80">
                <input type="text" id="searchBox" onkeyup="handleSearch(this.value)" class="w-full pl-12 pr-6 py-3.5 bg-slate-50 border border-slate-200 rounded-2xl text-sm focus:bg-white" placeholder="Cari instansi atau nomor...">
                <svg class="w-4 h-4 absolute left-4 top-4 text-slate-400" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"></path></svg>
            </div>
          </div>
          <div class="overflow-x-auto">
            <table class="w-full text-left">
              <thead>
                <tr class="text-[10px] font-black uppercase text-slate-400 bg-slate-50/50 border-b tracking-widest">
                  <th class="px-10 py-5">Identitas Resi</th>
                  <th class="px-10 py-5">Nomor Surat</th>
                  <th class="px-10 py-5">Instansi Tujuan</th>
                  <th class="px-10 py-5">Pengirim</th>
                  <th class="px-10 py-5 text-center">Jml</th>
                </tr>
              </thead>
              <tbody id="rekapTableBody" class="divide-y divide-slate-100">
                <tr><td colspan="5" class="p-32 text-center"><div class="loader-ring"></div></td></tr>
              </tbody>
            </table>
          </div>
        </div>
      </section>

      <!-- PAGE 3: LAPORAN -->
      <section id="contentLaporan" class="tab-content hidden animate-in fade-in duration-300">
        <div class="max-w-5xl mx-auto space-y-12">
          
          <div class="modern-card p-10 rounded-[32px] no-print">
            <div class="flex items-center gap-3 mb-8">
                <div class="w-1.5 h-8 bg-indigo-600 rounded-full"></div>
                <h3 class="font-bold text-slate-900 uppercase tracking-widest text-sm">Generator Laporan Resmi</h3>
            </div>
            
            <div class="grid grid-cols-1 md:grid-cols-3 gap-8 items-end">
              <div class="space-y-3">
                <label class="text-xs font-bold text-slate-400 uppercase tracking-wider ml-1">Dari Tanggal</label>
                <input type="date" id="reportStartDate" class="w-full px-6 py-4 bg-slate-50 border border-slate-200 rounded-2xl focus:bg-white font-medium">
              </div>
              <div class="space-y-3">
                <label class="text-xs font-bold text-slate-400 uppercase tracking-wider ml-1">Sampai Tanggal</label>
                <input type="date" id="reportEndDate" class="w-full px-6 py-4 bg-slate-50 border border-slate-200 rounded-2xl focus:bg-white font-medium">
              </div>
              <button onclick="previewReportAsLetter()" class="w-full py-4.5 bg-slate-900 hover:bg-black text-white font-bold rounded-2xl transition-all shadow-xl shadow-slate-200 uppercase text-xs tracking-widest">
                Tampilkan Preview
              </button>
            </div>
            
            <div class="mt-10 pt-10 border-t border-slate-100 flex flex-wrap gap-4">
                <button onclick="window.print()" class="px-8 py-4 bg-indigo-600 hover:bg-indigo-700 text-white font-bold rounded-2xl transition-all flex items-center gap-3 text-xs uppercase tracking-widest shadow-lg shadow-indigo-100">
                    <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17 17h2a2 2 0 002-2v-4a2 2 0 00-2-2H5a2 2 0 00-2 2v4a2 2 0 002 2h2m2 4h6a2 2 0 002-2v-4a2 2 0 00-2-2H9a2 2 0 00-2 2v4a2 2 0 002 2zm8-12V5a2 2 0 00-2-2H9a2 2 0 00-2 2v4h10z"></path></svg>
                    Cetak Laporan
                </button>
                <button onclick="downloadPDF()" class="px-8 py-4 bg-white hover:bg-slate-50 text-slate-700 border border-slate-200 font-bold rounded-2xl transition-all flex items-center gap-3 text-xs uppercase tracking-widest">
                    <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4"></path></svg>
                    Download PDF
                </button>
            </div>
          </div>

          <!-- Preview Area -->
          <div id="previewSuratWrapper" class="hidden pb-20">
            <div class="flex justify-center mb-6 no-print">
                <span class="px-4 py-1.5 bg-emerald-100 text-emerald-700 text-[10px] font-black uppercase tracking-widest rounded-full">Pratinjau Kertas A4</span>
            </div>
            
            <div id="printableArea" class="paper-preview">
              <div class="text-center border-b-[3px] border-double border-black pb-4 mb-10">
                  <p class="text-[14px] font-bold leading-tight">KEMENTERIAN KEUANGAN REPUBLIK INDONESIA</p>
                  <p class="text-[12px] font-bold leading-tight">DIREKTORAT JENDERAL KEKAYAAN NEGARA</p>
                  <p class="text-[10px] font-bold leading-tight">KANTOR WILAYAH DJKN BANTEN</p>
                  <p class="text-[11px] font-bold leading-tight">KANTOR PELAYANAN KEKAYAAN NEGARA DAN LELANG SERANG</p>
                  <p class="text-[9px] mt-1 normal-case font-normal italic">Jalan Raya Serang-Cilegon Km. 3, Serang 42162 Telepon (0254) 216361</p>
              </div>

              <div class="space-y-8">
                <div class="text-center">
                    <p class="font-bold underline uppercase text-[15px] tracking-[0.2em]">LAPORAN REKAPITULASI EKSPEDISI SURAT</p>
                    <p class="text-[10px] font-bold mt-2 uppercase">Periode: <span id="letterPeriodRange" class="underline">-</span></p>
                </div>

                <table class="table-laporan">
                  <thead>
                    <tr>
                      <th class="w-8">No</th>
                      <th class="w-32">Tanggal</th>
                      <th>Nomor Surat</th>
                      <th>Instansi Tujuan</th>
                      <th class="w-12">Jml</th>
                    </tr>
                  </thead>
                  <tbody id="letterTableContent"></tbody>
                </table>

                <div class="mt-20 grid grid-cols-2 gap-12 text-[12px]">
                  <div class="text-center">
                    <p class="font-bold mb-24">PETUGAS PENGIRIM</p>
                    <p class="font-bold underline">( ..................................... )</p>
                  </div>
                  <div class="text-center">
                    <p class="mb-4">Serang, <span id="letterPrintDate"></span></p>
                    <p class="font-bold mb-20 uppercase">PENERIMA / EKSPEDISI</p>
                    <p class="font-bold underline">( ..................................... )</p>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </section>
    </main>

    <footer class="no-print bg-white/50 border-t border-slate-100 py-10">
        <div class="max-w-7xl mx-auto px-6 flex flex-col md:flex-row justify-between items-center gap-4">
            <p class="text-[10px] font-bold text-slate-400 uppercase tracking-widest">© 2026 KPKNL Serang</p>
        </div>
    </footer>
  </div>

  <script>
    const SPREADSHEET_URL = "https://script.google.com/macros/s/AKfycbzh25RHh6tcCjRrixucsEVx6ugi4IwkxmJsM8-yyFCnByIzaTS3ohooIMgx7Yv7_ckP/exec";
    let db = [];

    function toast(msg, type='success') {
      const t = document.createElement('div');
      t.className = `toast ${type === 'success' ? 'bg-slate-900' : 'bg-red-600'}`;
      t.innerHTML = `<div class="w-2 h-2 rounded-full ${type === 'success' ? 'bg-indigo-400 animate-pulse' : 'bg-white'}"></div>${msg}`;
      document.body.appendChild(t);
      setTimeout(() => {
        t.style.opacity = '0';
        t.style.transform = 'translateY(20px)';
        setTimeout(() => t.remove(), 500);
      }, 3500);
    }

    function switchTab(tabId) {
      const tabs = ['Input', 'Rekap', 'Laporan'];
      tabs.forEach(x => {
        document.getElementById(`tab${x}`).classList.remove('active');
        document.getElementById(`content${x}`).classList.add('hidden');
      });
      document.getElementById(`tab${tabId}`).classList.add('active');
      document.getElementById(`content${tabId}`).classList.remove('hidden');
      if(tabId === 'Rekap') fetchAllData();
    }

    async function fetchAllData() {
      const dot = document.getElementById('syncDot');
      const text = document.getElementById('statusText');
      text.innerText = "SINKRONISASI...";
      dot.className = "w-2 h-2 bg-indigo-500 rounded-full animate-ping";
      
      try {
        const response = await fetch(`${SPREADSHEET_URL}?action=read&_t=${Date.now()}`);
        const result = await response.json();
        if (result.status === "success") {
          db = result.data || [];
          renderRekap(db);
          updateSummary();
          text.innerText = "SISTEM AKTIF";
          dot.className = "w-2 h-2 bg-emerald-500 rounded-full";
        }
      } catch (err) {
        text.innerText = "SISTEM AKTIF";
        dot.className = "w-2 h-2 bg-slate-400 rounded-full";
      }
    }

    function updateSummary() {
        const todayStr = new Date().toISOString().split('T')[0];
        const todayData = db.filter(i => i.tanggal === todayStr);
        document.getElementById('todayCount').innerText = todayData.length;
        const log = document.getElementById('recentLog');
        if(todayData.length > 0) {
            log.innerHTML = todayData.slice(0, 3).map(i => `
                <div class="flex items-start gap-3">
                    <div class="w-1.5 h-1.5 rounded-full bg-indigo-500 mt-1.5"></div>
                    <div>
                        <p class="text-[11px] font-bold text-slate-700 leading-tight truncate w-32">${i.no_surat}</p>
                        <p class="text-[10px] text-slate-400 uppercase font-medium mt-0.5">Tujuan: ${i.tujuan.substring(0,15)}...</p>
                    </div>
                </div>
            `).join('');
        }
    }

    function renderRekap(data = db) {
      const body = document.getElementById('rekapTableBody');
      if (data.length === 0) {
        body.innerHTML = '<tr><td colspan="5" class="p-32 text-center text-slate-400 font-medium">Belum ada arsip data yang tersimpan</td></tr>';
        return;
      }
      const sorted = [...data].sort((a,b) => new Date(b.tanggal) - new Date(a.tanggal));
      body.innerHTML = sorted.map(i => `
        <tr class="hover:bg-slate-50/80 transition-all group">
          <td class="px-10 py-6">
            <div class="text-[11px] font-black text-indigo-600 font-mono tracking-tighter">${i.resi || '-'}</div>
            <div class="text-[10px] font-bold text-slate-400 mt-1 uppercase tracking-tight">${i.tanggal || '-'}</div>
          </td>
          <td class="px-10 py-6 font-bold text-slate-700 text-sm">${i.no_surat || '-'}</td>
          <td class="px-10 py-6 text-slate-500 text-xs leading-relaxed max-w-xs truncate">${i.tujuan || '-'}</td>
          <td class="px-10 py-6 text-[11px] font-bold text-slate-500 uppercase">${i.pengirim || '-'}</td>
          <td class="px-10 py-6 text-center font-black text-slate-900">${i.jumlah || '1'}</td>
        </tr>
      `).join('');
    }

    function handleSearch(q) {
      const query = q.toLowerCase();
      const filtered = db.filter(i => 
        (i.tujuan?.toLowerCase().includes(query)) || 
        (i.no_surat?.toLowerCase().includes(query)) ||
        (i.resi?.toLowerCase().includes(query))
      );
      renderRekap(filtered);
    }

    document.getElementById('formPengiriman').onsubmit = async (e) => {
      e.preventDefault();
      const btn = document.getElementById('submitBtn');
      const original = btn.innerHTML;
      btn.disabled = true;
      btn.innerHTML = `<div class="loader-ring !border-white !w-4 !h-4"></div> <span class="text-xs uppercase tracking-widest">Memproses...</span>`;
      const entry = {
        id: Date.now(),
        resi: "KPKNL-" + Math.random().toString(36).substr(2, 5).toUpperCase(),
        no_surat: document.getElementById('nomorSurat').value,
        tanggal: document.getElementById('tanggalKirim').value,
        tujuan: document.getElementById('alamatTujuan').value,
        jumlah: document.getElementById('jumlahSurat').value,
        pengirim: document.getElementById('namaPengirim').value
      };
      try {
        await fetch(SPREADSHEET_URL, { method: 'POST', mode: 'no-cors', body: JSON.stringify(entry) });
        db.unshift(entry); renderRekap(db); updateSummary();
        toast("Data berhasil disimpan ke cloud!");
        e.target.reset();
        document.getElementById('tanggalKirim').valueAsDate = new Date();
      } catch (err) {
        toast("Terjadi gangguan koneksi cloud.", "error");
      } finally {
        btn.disabled = false; btn.innerHTML = original;
      }
    };

    function previewReportAsLetter() {
      const start = document.getElementById('reportStartDate').value;
      const end = document.getElementById('reportEndDate').value;
      if(!start || !end) return toast("Pilih rentang tanggal terlebih dahulu", "error");
      const filtered = db.filter(i => i.tanggal >= start && i.tanggal <= end).sort((a,b) => new Date(a.tanggal) - new Date(b.tanggal));
      if (filtered.length === 0) {
        document.getElementById('previewSuratWrapper').classList.add('hidden');
        return toast("Tidak ada data pada periode tersebut", "error");
      }
      document.getElementById('letterPeriodRange').innerText = `${start} s/d ${end}`;
      document.getElementById('letterPrintDate').innerText = `${new Date().toLocaleDateString('id-ID', {day:'numeric', month:'long', year:'numeric'})}`;
      document.getElementById('letterTableContent').innerHTML = filtered.map((i, idx) => `
        <tr>
          <td style="text-align:center">${idx + 1}</td>
          <td style="text-align:center">${i.tanggal}</td>
          <td style="font-weight:bold">${i.no_surat}</td>
          <td style="text-transform:uppercase; font-size:11px">${i.tujuan}</td>
          <td style="text-align:center; font-weight:bold">${i.jumlah || '1'}</td>
        </tr>`).join('');
      document.getElementById('previewSuratWrapper').classList.remove('hidden');
      document.getElementById('previewSuratWrapper').scrollIntoView({ behavior: 'smooth' });
    }

    async function downloadPDF() {
      const element = document.getElementById('printableArea');
      if (document.getElementById('previewSuratWrapper').classList.contains('hidden')) return toast("Tampilkan pratinjau terlebih dahulu", "error");
      toast("Membuat PDF berkualitas tinggi...");
      try {
        const { jsPDF } = window.jspdf;
        const canvas = await html2canvas(element, { scale: 2 });
        const imgData = canvas.toDataURL('image/png');
        const pdf = new jsPDF('p', 'mm', 'a4');
        const pdfWidth = pdf.internal.pageSize.getWidth();
        const pdfHeight = (canvas.height * pdfWidth) / canvas.width;
        pdf.addImage(imgData, 'PNG', 0, 0, pdfWidth, pdfHeight);
        pdf.save(`LAPORAN_EKSPEDISI_${Date.now()}.pdf`);
        toast("PDF siap diunduh!");
      } catch (err) { toast("Gagal konversi PDF", "error"); }
    }

    ['Input', 'Rekap', 'Laporan'].forEach(t => {
      document.getElementById(`tab${t}`).onclick = () => switchTab(t);
    });

    window.onload = () => {
      document.getElementById('tanggalKirim').valueAsDate = new Date();
      fetchAllData();
    };
  </script>
</body>
</html>
