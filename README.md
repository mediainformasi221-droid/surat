<!doctype html>
<html lang="id" class="h-full">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Sistem Pengiriman Surat - KPKNL Serang</title>
  
  <!-- Tailwind CSS & Fonts -->
  <script src="https://cdn.tailwindcss.com"></script>
  <link href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@300;400;500;600;700;800&display=swap" rel="stylesheet">
  
  <!-- Pustaka PDF & Canvas -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  
  <style>
    :root {
      --primary: #4f46e5;
      --secondary: #0f172a;
      --bg-app: #f8fafc;
    }

    * { 
      font-family: 'Plus Jakarta Sans', sans-serif; 
      box-sizing: border-box; 
    }
    
    body {
      background-color: var(--bg-app);
      background-image: 
        radial-gradient(circle at 0% 0%, rgba(79, 70, 229, 0.03) 0, transparent 50%),
        radial-gradient(circle at 100% 100%, rgba(99, 102, 241, 0.03) 0, transparent 50%);
      color: #334155;
      -webkit-font-smoothing: antialiased;
    }

    .modern-card {
      background: rgba(255, 255, 255, 0.7);
      backdrop-filter: blur(16px);
      -webkit-backdrop-filter: blur(16px);
      border: 1px solid rgba(255, 255, 255, 0.8);
      box-shadow: 0 4px 15px -1px rgba(0, 0, 0, 0.01), 0 10px 30px -3px rgba(0, 0, 0, 0.02);
      border-radius: 24px;
    }

    .tab-btn {
      position: relative;
      transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
      color: #64748b;
      font-weight: 600;
    }
    .tab-btn.active { color: var(--primary); }
    .tab-btn.active::after {
      content: '';
      position: absolute; bottom: -18px; left: 0; right: 0;
      height: 3px; background: var(--primary);
      border-radius: 10px;
      box-shadow: 0 4px 10px rgba(79, 70, 229, 0.3);
    }

    .toast {
      position: fixed; bottom: 2rem; right: 2rem;
      padding: 1rem 1.5rem; border-radius: 16px;
      background: #0f172a; color: white; z-index: 1000;
      display: flex; align-items: center; gap: 12px;
      box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1);
      animation: popUp 0.4s ease-out;
    }
    @keyframes popUp { from { transform: translateY(20px); opacity: 0; } to { transform: translateY(0); opacity: 1; } }

    /* Ukuran Kertas A4 Precission */
    .paper-preview {
      background: white;
      width: 210mm;
      min-height: 297mm;
      padding: 2.5cm 2cm;
      margin: 2rem auto;
      box-shadow: 0 0 50px rgba(0,0,0,0.06);
      color: #000;
      font-family: 'Times New Roman', Times, serif;
      position: relative;
      overflow: hidden;
    }
    
    .table-laporan { width: 100%; border-collapse: collapse; margin-top: 2rem; }
    .table-laporan th, .table-laporan td { 
      border: 0.5pt solid #000; 
      padding: 8px 10px; 
      font-size: 11pt; 
      line-height: 1.4;
    }
    .table-laporan th { background-color: #f8fafc; font-weight: bold; }

    .btn-primary {
      background: linear-gradient(135deg, #4f46e5 0%, #4338ca 100%);
      transition: all 0.3s ease;
    }
    .btn-primary:hover {
      transform: translateY(-2px);
      box-shadow: 0 12px 20px -5px rgba(79, 70, 229, 0.4);
    }

    @media print {
      body { background: white !important; }
      .no-print { display: none !important; }
      .paper-preview { 
        margin: 0 !important; 
        box-shadow: none !important; 
        width: 100% !important;
        padding: 0 !important;
      }
    }

    /* Scrollbar Style */
    ::-webkit-scrollbar { width: 8px; }
    ::-webkit-scrollbar-track { background: transparent; }
    ::-webkit-scrollbar-thumb { background: #e2e8f0; border-radius: 10px; }
    ::-webkit-scrollbar-thumb:hover { background: #cbd5e1; }
  </style>
</head>
<body class="h-full">

  <div id="app" class="min-h-screen flex flex-col">
    <!-- Navbar -->
    <header class="bg-white/70 backdrop-blur-xl sticky top-0 z-50 border-b border-slate-200/50 no-print">
      <div class="max-w-7xl mx-auto px-6 h-20 flex items-center justify-between">
        <div class="flex items-center gap-4">
          <div class="w-11 h-11 bg-indigo-600 rounded-xl flex items-center justify-center shadow-lg shadow-indigo-200">
            <svg class="w-6 h-6 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M3 8l7.89 5.26a2 2 0 002.22 0L21 8M5 19h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v10a2 2 0 002 2z"></path></svg>
          </div>
          <div>
            <h1 class="text-lg font-extrabold text-slate-900 tracking-tight leading-none">KPKNL SERANG</h1>
            <p class="text-[10px] font-bold text-slate-400 uppercase tracking-widest mt-1">E-Ekspedisi Surat</p>
          </div>
        </div>
        
        <nav class="hidden md:flex items-center gap-10">
          <button id="tabInput" class="tab-btn active text-sm">Input Data</button>
          <button id="tabRekap" class="tab-btn text-sm">Arsip Surat</button>
          <button id="tabLaporan" class="tab-btn text-sm">Cetak Laporan</button>
        </nav>

        <div class="flex items-center gap-2">
            <div id="statusDot" class="w-2 h-2 bg-emerald-500 rounded-full animate-pulse mr-1"></div>
            <!-- Status Text Removed as per request -->
            <span id="statusText" class="hidden">Online</span>
        </div>
      </div>
    </header>

    <main class="flex-grow w-full max-w-7xl mx-auto px-6 py-10">
      
      <!-- PAGE: INPUT -->
      <section id="contentInput" class="tab-content transition-all duration-500">
        <div class="max-w-4xl mx-auto grid grid-cols-1 lg:grid-cols-3 gap-8">
          <div class="lg:col-span-2">
            <div class="modern-card p-8 md:p-10">
              <div class="mb-8">
                <h2 class="text-2xl font-black text-slate-900">Registrasi Pengiriman</h2>
                <p class="text-slate-500 text-sm mt-1">Lengkapi detail pengiriman surat untuk arsip cloud.</p>
              </div>
              
              <form id="formPengiriman" class="space-y-5">
                <div class="grid grid-cols-1 md:grid-cols-2 gap-5">
                  <div class="space-y-2">
                    <label class="text-[11px] font-bold text-slate-500 uppercase tracking-wider ml-1">Nomor Surat</label>
                    <input type="text" id="nomorSurat" required class="w-full px-5 py-3.5 bg-slate-50 border border-slate-200 rounded-xl focus:ring-4 focus:ring-indigo-500/10 focus:border-indigo-500 outline-none transition-all" placeholder="S-000/KNL.0601/2026">
                  </div>
                  <div class="space-y-2">
                    <label class="text-[11px] font-bold text-slate-500 uppercase tracking-wider ml-1">Tanggal</label>
                    <input type="date" id="tanggalKirim" required class="w-full px-5 py-3.5 bg-slate-50 border border-slate-200 rounded-xl focus:ring-4 focus:ring-indigo-500/10 focus:border-indigo-500 outline-none transition-all">
                  </div>
                </div>
                
                <div class="space-y-2">
                  <label class="text-[11px] font-bold text-slate-500 uppercase tracking-wider ml-1">Instansi / Alamat Tujuan</label>
                  <textarea id="alamatTujuan" required rows="3" class="w-full px-5 py-3.5 bg-slate-50 border border-slate-200 rounded-xl focus:ring-4 focus:ring-indigo-500/10 focus:border-indigo-500 outline-none transition-all" placeholder="Contoh: Kanwil DJKN Banten, Jl. Raya Serang..."></textarea>
                </div>

                <div class="grid grid-cols-1 md:grid-cols-2 gap-5">
                  <div class="space-y-2">
                    <label class="text-[11px] font-bold text-slate-500 uppercase tracking-wider ml-1">Jumlah Berkas</label>
                    <input type="number" id="jumlahSurat" required value="1" min="1" class="w-full px-5 py-3.5 bg-slate-50 border border-slate-200 rounded-xl focus:ring-4 focus:ring-indigo-500/10 focus:border-indigo-500 outline-none transition-all">
                  </div>
                  <div class="space-y-2">
                    <label class="text-[11px] font-bold text-slate-500 uppercase tracking-wider ml-1">Petugas</label>
                    <input type="text" id="namaPengirim" required class="w-full px-5 py-3.5 bg-slate-50 border border-slate-200 rounded-xl focus:ring-4 focus:ring-indigo-500/10 focus:border-indigo-500 outline-none transition-all" placeholder="Nama penginput">
                  </div>
                </div>

                <button type="submit" id="submitBtn" class="btn-primary w-full py-4 text-white font-bold rounded-xl flex items-center justify-center gap-3 mt-4">
                  <span>SIMPAN DATA</span>
                  <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M14 5l7 7m0 0l-7 7m7-7H3"></path></svg>
                </button>
              </form>
            </div>
          </div>
          
          <div class="space-y-6">
            <div class="modern-card p-8 bg-indigo-600 text-white border-none overflow-hidden relative">
              <div class="relative z-10">
                <p class="text-indigo-200 text-[10px] font-black uppercase tracking-widest mb-1">Total Hari Ini</p>
                <div class="flex items-baseline gap-2">
                  <h3 id="todayCount" class="text-6xl font-black">0</h3>
                  <span class="text-indigo-200 font-bold text-sm">Surat</span>
                </div>
              </div>
              <div class="absolute -right-10 -bottom-10 w-40 h-40 bg-white/10 rounded-full blur-3xl"></div>
            </div>
            
            <div class="modern-card p-6">
              <h4 class="text-[11px] font-black text-slate-400 uppercase tracking-widest mb-4">Aktivitas Terbaru</h4>
              <div id="recentLog" class="space-y-4">
                <p class="text-xs text-slate-400 italic">Belum ada aktivitas.</p>
              </div>
            </div>
          </div>
        </div>
      </section>

      <!-- PAGE: REKAP -->
      <section id="contentRekap" class="tab-content hidden animate-in fade-in slide-in-from-bottom-4 duration-500">
        <div class="modern-card overflow-hidden">
          <div class="p-8 border-b border-slate-100 flex flex-col md:flex-row justify-between items-center gap-4">
            <div>
              <h2 class="text-xl font-black text-slate-900">Arsip Pengiriman</h2>
              <p class="text-xs text-slate-500 mt-1">Data historis pengiriman surat yang tersimpan di cloud.</p>
            </div>
            <div class="relative w-full md:w-80">
              <input type="text" id="searchBox" onkeyup="handleSearch(this.value)" class="w-full pl-12 pr-6 py-3 bg-slate-50 border border-slate-200 rounded-xl text-sm outline-none focus:bg-white focus:ring-4 focus:ring-indigo-500/10 transition-all" placeholder="Cari nomor surat atau tujuan...">
              <svg class="w-5 h-5 absolute left-4 top-3 text-slate-400" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"></path></svg>
            </div>
          </div>
          <div class="overflow-x-auto">
            <table class="w-full">
              <thead>
                <tr class="bg-slate-50/50 border-b border-slate-100 text-[10px] font-black uppercase text-slate-400 tracking-widest">
                  <th class="px-8 py-5 text-left">Resi / Tanggal</th>
                  <th class="px-8 py-5 text-left">Nomor Surat</th>
                  <th class="px-8 py-5 text-left">Tujuan</th>
                  <th class="px-8 py-5 text-left">Petugas</th>
                  <th class="px-8 py-5 text-center">Berkas</th>
                </tr>
              </thead>
              <tbody id="rekapTableBody" class="divide-y divide-slate-100">
                <tr><td colspan="5" class="py-20 text-center text-slate-400">Memuat data...</td></tr>
              </tbody>
            </table>
          </div>
        </div>
      </section>

      <!-- PAGE: LAPORAN -->
      <section id="contentLaporan" class="tab-content hidden animate-in fade-in slide-in-from-bottom-4 duration-500">
        <div class="max-w-5xl mx-auto space-y-8">
          <div class="modern-card p-8 md:p-10 no-print">
            <h3 class="text-lg font-black text-slate-900 mb-6">Generator Laporan Ekspedisi</h3>
            <div class="grid grid-cols-1 md:grid-cols-3 gap-6 items-end">
              <div class="space-y-2">
                <label class="text-[10px] font-bold text-slate-400 uppercase tracking-widest ml-1">Periode Awal</label>
                <input type="date" id="reportStartDate" class="w-full px-5 py-3 bg-slate-50 border border-slate-200 rounded-xl outline-none focus:bg-white transition-all">
              </div>
              <div class="space-y-2">
                <label class="text-[10px] font-bold text-slate-400 uppercase tracking-widest ml-1">Periode Akhir</label>
                <input type="date" id="reportEndDate" class="w-full px-5 py-3 bg-slate-50 border border-slate-200 rounded-xl outline-none focus:bg-white transition-all">
              </div>
              <button onclick="generatePreview()" class="py-3.5 bg-slate-900 text-white font-bold rounded-xl hover:bg-black transition-all text-xs tracking-widest">PRATINJAU LAPORAN</button>
            </div>
            
            <div id="reportActions" class="hidden mt-8 pt-8 border-t border-slate-100 flex flex-wrap gap-4">
              <button onclick="window.print()" class="px-6 py-3 bg-indigo-600 text-white font-bold rounded-xl flex items-center gap-2 text-xs tracking-widest hover:bg-indigo-700 transition-all">
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17 17h2a2 2 0 002-2v-4a2 2 0 00-2-2H5a2 2 0 00-2 2v4a2 2 0 002 2h2m2 4h6a2 2 0 002-2v-4a2 2 0 00-2-2H9a2 2 0 00-2 2v4a2 2 0 002 2zm8-12V5a2 2 0 00-2-2H9a2 2 0 00-2 2v4h10z"></path></svg>
                CETAK / PRINT
              </button>
              <button onclick="exportToPDF()" class="px-6 py-3 bg-white text-slate-700 border border-slate-200 font-bold rounded-xl flex items-center gap-2 text-xs tracking-widest hover:bg-slate-50 transition-all">
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4"></path></svg>
                DOWNLOAD PDF
              </button>
            </div>
          </div>

          <!-- Kertas A4 -->
          <div id="reportPreviewArea" class="hidden animate-in zoom-in-95 duration-500">
            <div id="printableArea" class="paper-preview">
              <!-- Kop Surat -->
              <div class="text-center border-b-[2.5pt] border-double border-black pb-4 mb-8">
                <p class="text-[12pt] font-bold m-0 leading-tight">KEMENTERIAN KEUANGAN REPUBLIK INDONESIA</p>
                <p class="text-[11pt] font-bold m-0 leading-tight">DIREKTORAT JENDERAL KEKAYAAN NEGARA</p>
                <p class="text-[9pt] font-bold m-0 leading-tight">KANTOR WILAYAH DJKN BANTEN</p>
                <p class="text-[10pt] font-bold m-0 leading-tight">KANTOR PELAYANAN KEKAYAAN NEGARA DAN LELANG SERANG</p>
                <p class="text-[8pt] m-0 mt-1 italic font-normal">Jalan Raya Serang-Cilegon Km. 3, Serang 42162 Telepon (0254) 216361</p>
              </div>

              <!-- Judul Laporan -->
              <div class="text-center mb-10">
                <p class="text-[14pt] font-bold underline m-0 tracking-wider">DAFTAR EKSPEDISI SURAT DINAS</p>
                <p class="text-[10pt] font-bold mt-1 uppercase">PERIODE: <span id="reportPeriodText">-</span></p>
              </div>

              <!-- Isi Tabel -->
              <table class="table-laporan">
                <thead>
                  <tr>
                    <th style="width: 30pt">No</th>
                    <th style="width: 80pt">Tanggal</th>
                    <th>Nomor Surat</th>
                    <th>Instansi Tujuan</th>
                    <th style="width: 35pt">Jml</th>
                  </tr>
                </thead>
                <tbody id="reportTableContent"></tbody>
              </table>

              <!-- Tanda Tangan -->
              <div class="mt-16 grid grid-cols-2 text-[11pt]">
                <div class="text-center">
                  <p class="font-bold mb-24">PENGIRIM / PETUGAS</p>
                  <p class="font-bold">( ..................................... )</p>
                </div>
                <div class="text-center">
                  <p class="mb-1">Serang, <span id="reportTodayDate"></span></p>
                  <p class="font-bold mb-20 uppercase">PENERIMA / EKSPEDISI</p>
                  <p class="font-bold">( ..................................... )</p>
                </div>
              </div>
            </div>
          </div>
        </div>
      </section>
    </main>

    <footer class="no-print py-8 border-t border-slate-200/50 bg-white/50 backdrop-blur-sm">
      <div class="max-w-7xl mx-auto px-6 flex flex-col md:flex-row justify-between items-center gap-4 text-[10px] font-bold text-slate-400 uppercase tracking-widest">
        <p>© 2026 KPKNL Serang</p>
        <div class="flex gap-4">
          <!-- Versions and Cloud Status Removed as per request -->
        </div>
      </div>
    </footer>
  </div>

  <script>
    const SPREADSHEET_URL = "https://script.google.com/macros/s/AKfycbzh25RHh6tcCjRrixucsEVx6ugi4IwkxmJsM8-yyFCnByIzaTS3ohooIMgx7Yv7_ckP/exec";
    let database = [];

    // UI Helper: Toast
    function notify(message, type = 'success') {
      const t = document.createElement('div');
      t.className = `toast ${type === 'success' ? 'bg-slate-900' : 'bg-red-600'}`;
      t.innerHTML = `<svg class="w-4 h-4 text-indigo-400" fill="currentColor" viewBox="0 0 20 20"><path d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z"></path></svg>${message}`;
      document.body.appendChild(t);
      setTimeout(() => {
        t.style.opacity = '0';
        setTimeout(() => t.remove(), 500);
      }, 3000);
    }

    // Navigation Logic
    function setActiveTab(id) {
      document.querySelectorAll('.tab-btn').forEach(btn => btn.classList.remove('active'));
      document.querySelectorAll('.tab-content').forEach(content => content.classList.add('hidden'));
      
      document.getElementById(`tab${id}`).classList.add('active');
      document.getElementById(`content${id}`).classList.remove('hidden');
      
      if (id === 'Rekap') refreshData();
    }

    // Data Fetching
    async function refreshData() {
      const dot = document.getElementById('statusDot');
      const text = document.getElementById('statusText');
      text.innerText = "SINKRONISASI...";
      dot.className = "w-2 h-2 bg-indigo-500 rounded-full animate-ping";
      
      try {
        const resp = await fetch(`${SPREADSHEET_URL}?action=read&_t=${Date.now()}`);
        const result = await resp.json();
        if (result.status === "success") {
          database = result.data || [];
          renderTable(database);
          updateDash();
          text.innerText = "ONLINE";
          dot.className = "w-2 h-2 bg-emerald-500 rounded-full";
        }
      } catch (err) {
        text.innerText = "OFFLINE";
        dot.className = "w-2 h-2 bg-red-500 rounded-full";
      }
    }

    function updateDash() {
      const today = new Date().toISOString().split('T')[0];
      const items = database.filter(i => i.tanggal === today);
      document.getElementById('todayCount').innerText = items.length;
      
      const log = document.getElementById('recentLog');
      if (items.length > 0) {
        log.innerHTML = items.slice(0, 3).map(i => `
          <div class="flex items-start gap-3 p-3 bg-slate-50 rounded-xl">
            <div class="w-1.5 h-1.5 bg-indigo-500 rounded-full mt-1.5"></div>
            <div>
              <p class="text-[11px] font-bold text-slate-800 truncate w-32">${i.no_surat}</p>
              <p class="text-[9px] text-slate-400 uppercase font-medium mt-0.5">${i.tujuan.substring(0, 20)}...</p>
            </div>
          </div>
        `).join('');
      }
    }

    function renderTable(data) {
      const body = document.getElementById('rekapTableBody');
      if (data.length === 0) {
        body.innerHTML = '<tr><td colspan="5" class="py-20 text-center text-slate-400 italic">Arsip masih kosong.</td></tr>';
        return;
      }
      
      const sorted = [...data].sort((a,b) => new Date(b.tanggal) - new Date(a.tanggal));
      body.innerHTML = sorted.map(i => `
        <tr class="hover:bg-slate-50/50 transition-colors group">
          <td class="px-8 py-5">
            <div class="text-[10px] font-black text-indigo-600 font-mono tracking-tighter">${i.resi || 'E-0000'}</div>
            <div class="text-[10px] text-slate-400 font-bold mt-1 uppercase">${i.tanggal}</div>
          </td>
          <td class="px-8 py-5 text-sm font-bold text-slate-700">${i.no_surat}</td>
          <td class="px-8 py-5 text-xs text-slate-500 max-w-xs truncate">${i.tujuan}</td>
          <td class="px-8 py-5 text-[10px] font-bold text-slate-500 uppercase tracking-tight">${i.pengirim}</td>
          <td class="px-8 py-5 text-center font-black text-slate-900">${i.jumlah}</td>
        </tr>
      `).join('');
    }

    function handleSearch(q) {
      const term = q.toLowerCase();
      const filtered = database.filter(i => 
        i.no_surat.toLowerCase().includes(term) || 
        i.tujuan.toLowerCase().includes(term)
      );
      renderTable(filtered);
    }

    // Form Action
    document.getElementById('formPengiriman').onsubmit = async (e) => {
      e.preventDefault();
      const btn = document.getElementById('submitBtn');
      const original = btn.innerHTML;
      
      btn.disabled = true;
      btn.innerHTML = `<span class="animate-spin w-4 h-4 border-2 border-white/20 border-t-white rounded-full"></span> <span>MEMPROSES...</span>`;
      
      const payload = {
        id: Date.now(),
        resi: "KPKNL-" + Math.random().toString(36).substring(2, 7).toUpperCase(),
        no_surat: document.getElementById('nomorSurat').value,
        tanggal: document.getElementById('tanggalKirim').value,
        tujuan: document.getElementById('alamatTujuan').value,
        jumlah: document.getElementById('jumlahSurat').value,
        pengirim: document.getElementById('namaPengirim').value
      };

      try {
        await fetch(SPREADSHEET_URL, { method: 'POST', mode: 'no-cors', body: JSON.stringify(payload) });
        database.unshift(payload);
        renderTable(database);
        updateDash();
        notify("Data tersimpan di cloud!");
        e.target.reset();
        document.getElementById('tanggalKirim').valueAsDate = new Date();
      } catch (err) {
        notify("Gagal menghubungi server.", "error");
      } finally {
        btn.disabled = false;
        btn.innerHTML = original;
      }
    };

    // Report Generation
    function generatePreview() {
      const start = document.getElementById('reportStartDate').value;
      const end = document.getElementById('reportEndDate').value;
      
      if (!start || !end) return notify("Lengkapi rentang tanggal!", "error");
      
      const filtered = database.filter(i => i.tanggal >= start && i.tanggal <= end)
                               .sort((a,b) => new Date(a.tanggal) - new Date(b.tanggal));
                               
      if (filtered.length === 0) {
        document.getElementById('reportPreviewArea').classList.add('hidden');
        document.getElementById('reportActions').classList.add('hidden');
        return notify("Data tidak ditemukan.", "error");
      }

      document.getElementById('reportPeriodText').innerText = `${start} s.d ${end}`;
      document.getElementById('reportTodayDate').innerText = new Date().toLocaleDateString('id-ID', {day:'numeric', month:'long', year:'numeric'});
      
      document.getElementById('reportTableContent').innerHTML = filtered.map((i, idx) => `
        <tr>
          <td style="text-align:center">${idx + 1}</td>
          <td style="text-align:center">${i.tanggal}</td>
          <td style="font-weight:bold">${i.no_surat}</td>
          <td style="text-transform:uppercase; font-size:10pt">${i.tujuan}</td>
          <td style="text-align:center; font-weight:bold">${i.jumlah}</td>
        </tr>
      `).join('');

      document.getElementById('reportPreviewArea').classList.remove('hidden');
      document.getElementById('reportActions').classList.remove('hidden');
      window.scrollTo({ top: document.body.scrollHeight, behavior: 'smooth' });
    }

    async function exportToPDF() {
      const element = document.getElementById('printableArea');
      notify("Mempersiapkan dokumen PDF...");
      try {
        const { jsPDF } = window.jspdf;
        const canvas = await html2canvas(element, { scale: 3, useCORS: true });
        const imgData = canvas.toDataURL('image/png');
        const pdf = new jsPDF('p', 'mm', 'a4');
        const pdfW = pdf.internal.pageSize.getWidth();
        const pdfH = (canvas.height * pdfW) / canvas.width;
        pdf.addImage(imgData, 'PNG', 0, 0, pdfW, pdfH);
        pdf.save(`EKSPEDISI_${Date.now()}.pdf`);
        notify("Download dimulai.");
      } catch (e) { notify("Gagal membuat PDF.", "error"); }
    }

    // Init
    ['Input', 'Rekap', 'Laporan'].forEach(tab => {
      document.getElementById(`tab${tab}`).onclick = () => setActiveTab(tab);
    });

    window.onload = () => {
      document.getElementById('tanggalKirim').valueAsDate = new Date();
      refreshData();
    };
  </script>
</body>
</html>
