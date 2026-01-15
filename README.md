<!doctype html>
<html lang="id" class="h-full">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Sistem Pengiriman Surat - KPKNL Serang</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- Pustaka untuk PDF -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&display=swap');
    
    :root {
      --primary: #0f172a;
      --accent: #3b82f6;
    }

    * { font-family: 'Inter', sans-serif; box-sizing: border-box; }
    
    body {
      background-color: #f8fafc;
      background-image: radial-gradient(#e2e8f0 0.5px, transparent 0.5px);
      background-size: 24px 24px;
    }

    .glass-card {
      background: rgba(255, 255, 255, 0.9);
      backdrop-filter: blur(10px);
      border: 1px solid rgba(226, 232, 240, 0.8);
    }

    .toast {
      position: fixed; top: 24px; right: 24px;
      padding: 1rem 1.5rem; border-radius: 12px;
      color: white; z-index: 1000; 
      box-shadow: 0 20px 25px -5px rgba(0,0,0,0.1);
      animation: slideIn 0.4s cubic-bezier(0.16, 1, 0.3, 1);
    }

    @keyframes slideIn { from { transform: translateX(100%); opacity: 0; } to { transform: translateX(0); opacity: 1; } }
    
    .tab-active { 
      color: var(--accent); 
      position: relative;
    }
    .tab-active::after {
      content: '';
      position: absolute; bottom: 0; left: 0; right: 0;
      height: 3px; background: var(--accent);
      border-radius: 3px 3px 0 0;
    }

    .loader {
      border: 2px solid #f3f3f3; border-top: 2px solid var(--accent);
      border-radius: 50%; width: 16px; height: 16px;
      animation: spin 0.8s linear infinite; display: inline-block;
    }
    @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }

    /* Estetika Laporan/Preview Surat */
    .paper-preview {
      background: white;
      box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.15);
      width: 210mm;
      margin: 0 auto;
      min-height: 297mm;
      padding: 2cm;
      position: relative;
      color: #000;
      font-family: "Times New Roman", Times, serif;
    }
    .letter-header {
      text-align: center;
      border-bottom: 3px double #000;
      margin-bottom: 1rem;
      padding-bottom: 0.5rem;
    }
    
    .table-laporan { width: 100%; border-collapse: collapse; font-size: 12px; margin-top: 1rem; }
    .table-laporan th, .table-laporan td { border: 1px solid #000; padding: 8px 6px; }
    .table-laporan th { background-color: #f8fafc; font-weight: bold; }
    
    input, textarea, select {
      transition: all 0.2s ease;
    }
    input:focus, textarea:focus {
      border-color: var(--accent) !important;
      box-shadow: 0 0 0 4px rgba(59, 130, 246, 0.1) !important;
    }

    .btn-gradient {
      background: linear-gradient(to right, #1e293b, #334155);
      transition: transform 0.1s active;
    }
    .btn-gradient:active { transform: scale(0.98); }

    /* KUNCI PERBAIKAN CETAK: Hanya menampilkan #printableArea saat window.print() dipanggil */
    @media print {
      body * {
        visibility: hidden;
      }
      #printableArea, #printableArea * {
        visibility: visible;
      }
      #printableArea {
        position: absolute;
        left: 0;
        top: 0;
        width: 100%;
        margin: 0 !important;
        padding: 0 !important;
        box-shadow: none !important;
        border: none !important;
      }
      /* Hilangkan margin default browser pada cetakan */
      @page {
        margin: 0;
      }
    }
  </style>
</head>
<body class="h-full">
  <div id="app" class="min-h-screen flex flex-col">
    
    <!-- Navbar Modern -->
    <header class="bg-slate-900 text-white sticky top-0 z-50 no-print shadow-2xl">
      <div class="max-w-7xl mx-auto px-6 py-4 flex items-center justify-between">
        <div class="flex items-center gap-4">
          <div class="w-10 h-10 bg-blue-500 rounded-xl flex items-center justify-center shadow-lg shadow-blue-500/30">
            <span class="text-white font-black text-xl">S</span>
          </div>
          <div>
            <h1 class="text-lg font-extrabold tracking-tight uppercase leading-none">Pengiriman Surat</h1>
            <div id="syncStatus" class="flex items-center gap-2 mt-1">
              <span id="syncDot" class="w-2 h-2 bg-emerald-400 rounded-full shadow-[0_0_8px_rgba(52,211,153,0.6)]"></span>
              <span id="statusText" class="text-[10px] font-bold text-slate-400 uppercase tracking-widest">Sinkron</span>
            </div>
          </div>
        </div>
        <div class="flex items-center gap-3">
            <button onclick="fetchAllData()" class="p-2.5 bg-slate-800 hover:bg-slate-700 rounded-xl transition-all border border-slate-700" title="Segarkan Data">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 text-slate-300" fill="none" viewbox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15" />
                </svg>
            </button>
        </div>
      </div>
      
      <!-- Tab Navigation -->
      <div class="bg-white/5 border-t border-white/10 no-print">
          <div class="max-w-7xl mx-auto px-6 flex gap-10">
            <button id="tabInput" class="tab-active py-4 font-bold text-[11px] uppercase tracking-[0.2em] transition-all">Input Data</button>
            <button id="tabRekap" class="py-4 font-bold text-[11px] uppercase tracking-[0.2em] text-slate-400 transition-all hover:text-white">Daftar Rekap</button>
            <button id="tabLaporan" class="py-4 font-bold text-[11px] uppercase tracking-[0.2em] text-slate-400 transition-all hover:text-white">Laporan PDF</button>
          </div>
      </div>
    </header>

    <main class="flex-grow max-w-7xl mx-auto w-full px-6 py-10">
      
      <!-- HALAMAN 1: FORM INPUT -->
      <section id="contentInput" class="tab-content animate-[fadeIn_0.3s_ease-out]">
        <div class="max-w-2xl mx-auto">
          <div class="glass-card rounded-3xl shadow-xl overflow-hidden border border-slate-200">
            <div class="p-8 border-b bg-slate-50/50">
              <h2 class="font-extrabold text-slate-800 text-base uppercase tracking-widest">Buat Pengiriman Baru</h2>
              <p class="text-xs text-slate-500 mt-1">Lengkapi detail surat untuk pengiriman resmi</p>
            </div>
            <form id="formPengiriman" class="p-10 space-y-8">
               <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
                <div class="space-y-2">
                  <label class="text-[11px] font-bold uppercase text-slate-500 tracking-wider">Nomor Surat</label>
                  <input type="text" id="nomorSurat" required class="w-full px-5 py-3.5 bg-white border border-slate-200 rounded-2xl outline-none" placeholder="S-000/KNL.0601/2026">
                </div>
                <div class="space-y-2">
                  <label class="text-[11px] font-bold uppercase text-slate-500 tracking-wider">Tanggal Kirim</label>
                  <input type="date" id="tanggalKirim" required class="w-full px-5 py-3.5 bg-white border border-slate-200 rounded-2xl outline-none">
                </div>
              </div>
              <div class="space-y-2">
                <label class="text-[11px] font-bold uppercase text-slate-500 tracking-wider">Instansi / Alamat Tujuan</label>
                <textarea id="alamatTujuan" required rows="3" class="w-full px-5 py-3.5 bg-white border border-slate-200 rounded-2xl outline-none" placeholder="Masukkan nama instansi dan alamat lengkap..."></textarea>
              </div>
              <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
                <div class="space-y-2">
                  <label class="text-[11px] font-bold uppercase text-slate-500 tracking-wider">Jumlah Berkas</label>
                  <input type="number" id="jumlahSurat" required value="1" min="1" class="w-full px-5 py-3.5 bg-white border border-slate-200 rounded-2xl outline-none">
                </div>
                <div class="space-y-2">
                  <label class="text-[11px] font-bold uppercase text-slate-500 tracking-wider">Petugas Pengirim</label>
                  <input type="text" id="namaPengirim" required class="w-full px-5 py-3.5 bg-white border border-slate-200 rounded-2xl outline-none" placeholder="Nama Petugas">
                </div>
              </div>
              <button type="submit" id="submitBtn" class="w-full btn-gradient text-white font-bold py-5 rounded-2xl shadow-xl shadow-slate-200 tracking-widest uppercase text-xs">Simpan Data & Sinkronkan</button>
            </form>
          </div>
        </div>
      </section>

      <!-- HALAMAN 2: REKAP -->
      <section id="contentRekap" class="tab-content hidden animate-[fadeIn_0.3s_ease-out]">
        <div class="glass-card rounded-3xl shadow-xl border border-slate-200 overflow-hidden">
          <div class="p-8 border-b bg-slate-50/50 flex flex-col md:flex-row justify-between items-center gap-6">
            <div class="relative w-full md:w-1/3">
                <div class="absolute inset-y-0 left-4 flex items-center pointer-events-none">
                    <svg class="h-4 w-4 text-slate-400" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"></path></svg>
                </div>
                <input type="text" id="searchBox" onkeyup="handleSearch(this.value)" class="w-full pl-12 pr-6 py-3 bg-white border border-slate-200 rounded-2xl outline-none text-sm transition-all" placeholder="Cari surat atau tujuan...">
            </div>
            <div class="flex items-center gap-4">
                <div class="px-6 py-3 bg-indigo-50 border border-indigo-100 rounded-2xl text-center">
                    <p class="text-[10px] font-black text-indigo-400 uppercase tracking-widest leading-none mb-1">Total Record</p>
                    <p id="recordCount" class="text-2xl font-black text-indigo-600">0</p>
                </div>
            </div>
          </div>
          <div class="overflow-x-auto">
            <table class="w-full text-left">
              <thead>
                <tr class="text-[10px] font-black uppercase text-slate-400 border-b bg-slate-50/80 tracking-widest">
                  <th class="px-8 py-5">Info Resi</th>
                  <th class="px-8 py-5">Nomor Surat</th>
                  <th class="px-8 py-5">Instansi Tujuan</th>
                  <th class="px-8 py-5">Petugas</th>
                  <th class="px-8 py-5 text-center">Jml</th>
                  <th class="px-8 py-5">Status</th>
                </tr>
              </thead>
              <tbody id="rekapTableBody" class="divide-y divide-slate-100 text-sm">
                <tr><td colspan="6" class="p-24 text-center"><span class="loader"></span><p class="mt-4 text-slate-400 font-medium">Mengambil data dari cloud...</p></td></tr>
              </tbody>
            </table>
          </div>
        </div>
      </section>

      <!-- HALAMAN 3: LAPORAN -->
      <section id="contentLaporan" class="tab-content hidden animate-[fadeIn_0.3s_ease-out]">
        <div class="max-w-5xl mx-auto space-y-10">
          <div class="glass-card p-10 rounded-3xl shadow-xl border border-slate-200 no-print">
            <h3 class="font-bold text-slate-800 mb-8 flex items-center gap-2 uppercase tracking-widest text-sm">
                <span class="w-1.5 h-6 bg-blue-500 rounded-full"></span> 
                Filter Laporan
            </h3>
            <div class="grid grid-cols-1 md:grid-cols-4 gap-6 items-end">
              <div class="space-y-3">
                <label class="text-[11px] font-bold uppercase text-slate-400 tracking-wider">Pilih Tahun</label>
                <select id="yearSelector" onchange="setYearlyFilter(this.value)" class="w-full px-5 py-3 bg-white border border-slate-200 rounded-2xl outline-none font-medium">
                    <option value="">-- Pilih --</option>
                    <script>
                        const yr = new Date().getFullYear();
                        for(let i=0; i<5; i++) document.write(`<option value="${yr-i}">${yr-i}</option>`);
                    </script>
                </select>
              </div>
              <div class="space-y-3">
                <label class="text-[11px] font-bold uppercase text-slate-400 tracking-wider">Dari Tanggal</label>
                <input type="date" id="reportStartDate" class="w-full px-5 py-3 bg-white border border-slate-200 rounded-2xl outline-none font-medium">
              </div>
              <div class="space-y-3">
                <label class="text-[11px] font-bold uppercase text-slate-400 tracking-wider">Sampai Tanggal</label>
                <input type="date" id="reportEndDate" class="w-full px-5 py-3 bg-white border border-slate-200 rounded-2xl outline-none font-medium">
              </div>
              <div class="flex gap-3">
                <button onclick="previewReportAsLetter()" class="flex-grow px-6 py-3.5 bg-indigo-600 hover:bg-indigo-700 text-white font-bold rounded-2xl transition-all shadow-lg shadow-indigo-200 text-xs uppercase tracking-widest">Tampilkan</button>
              </div>
            </div>
            <div class="mt-8 pt-8 border-t flex flex-wrap gap-4">
                <button onclick="window.print()" class="px-6 py-3 bg-slate-900 hover:bg-black text-white font-bold rounded-2xl transition-all flex items-center gap-2 text-xs uppercase tracking-widest">🖨️ Cetak</button>
                <button onclick="downloadPDF()" class="px-6 py-3 bg-emerald-600 hover:bg-emerald-700 text-white font-bold rounded-2xl transition-all flex items-center gap-2 text-xs uppercase tracking-widest shadow-lg shadow-emerald-100">💾 Unduh PDF</button>
            </div>
          </div>

          <!-- Preview -->
          <div id="previewSuratWrapper" class="hidden">
            <div id="printableArea" class="paper-preview">
              <div class="letter-header">
                <div class="flex items-center justify-center gap-4 mb-2">
                    <div class="header-text uppercase text-center">
                      <p class="text-[15px] font-bold leading-tight">KEMENTERIAN KEUANGAN REPUBLIK INDONESIA</p>
                      <p class="text-[13px] font-bold leading-tight">DIREKTORAT JENDERAL KEKAYAAN NEGARA</p>
                      <p class="text-[11px] font-bold leading-tight">KANTOR WILAYAH DJKN BANTEN</p>
                      <p class="text-[12px] font-bold leading-tight">KANTOR PELAYANAN KEKAYAAN NEGARA DAN LELANG SERANG</p>
                      <p class="text-[10px] mt-1 normal-case font-normal italic">Jalan Raya Serang-Cilegon Km. 3, Serang 42162 Telepon (0254) 216361</p>
                    </div>
                </div>
              </div>

              <div class="mt-10 space-y-8">
                <div class="text-center">
                    <p class="font-bold underline uppercase text-[14px] tracking-widest">LAPORAN REKAPITULASI PENGIRIMAN SURAT</p>
                    <p class="text-[11px] font-bold mt-2 italic">Periode: <span id="letterPeriodRange"></span></p>
                </div>

                <table class="table-laporan">
                  <thead>
                    <tr>
                      <th class="w-10">NO</th>
                      <th class="w-32">TANGGAL</th>
                      <th class="w-56">NOMOR SURAT</th>
                      <th>INSTANSI TUJUAN</th>
                      <th class="w-12">JML</th>
                    </tr>
                  </thead>
                  <tbody id="letterTableContent"></tbody>
                </table>

                <div class="mt-16 grid grid-cols-2 gap-12 text-[12px]">
                  <div class="text-center">
                    <p class="font-bold mb-24 uppercase">PETUGAS PENGIRIM</p>
                    <p class="font-bold underline">( ..................................... )</p>
                  </div>
                  <div class="text-center">
                    <p class="font-bold mb-4">Serang, <span id="letterPrintDate"></span></p>
                    <p class="font-bold mb-20 uppercase">PENERIMA SURAT / EKSPEDISI</p>
                    <p class="font-bold underline">( ..................................... )</p>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </section>
    </main>

    <!-- Footer Simple -->
    <footer class="no-print bg-white border-t py-6 text-center">
        <p class="text-[10px] font-bold text-slate-400 uppercase tracking-widest">© 2026 KPKNL Serang - Seksi Hukum dan Informasi</p>
    </footer>
  </div>

  <script>
    const SPREADSHEET_URL = "https://script.google.com/macros/s/AKfycbzh25RHh6tcCjRrixucsEVx6ugi4IwkxmJsM8-yyFCnByIzaTS3ohooIMgx7Yv7_ckP/exec";
    let db = [];

    function toast(msg, color='indigo') {
      const t = document.createElement('div');
      t.className = `toast ${color === 'indigo' ? 'bg-slate-900 border border-slate-700' : 'bg-red-600'}`;
      t.innerHTML = `<div class="flex items-center gap-3"><span class="w-2 h-2 rounded-full ${color === 'indigo' ? 'bg-blue-400' : 'bg-white'}"></span> ${msg}</div>`;
      document.body.appendChild(t);
      setTimeout(() => {
        t.style.opacity = '0';
        t.style.transform = 'translateX(20px)';
        setTimeout(() => t.remove(), 400);
      }, 3000);
    }

    function switchTab(tabId) {
      const tabs = ['Input', 'Rekap', 'Laporan'];
      tabs.forEach(x => {
        document.getElementById(`tab${x}`).classList.remove('tab-active');
        document.getElementById(`tab${x}`).classList.add('text-slate-400');
        document.getElementById(`content${x}`).classList.add('hidden');
      });
      document.getElementById(`tab${tabId}`).classList.add('tab-active');
      document.getElementById(`tab${tabId}`).classList.remove('text-slate-400');
      document.getElementById(`content${tabId}`).classList.remove('hidden');
    }

    async function fetchAllData() {
      const dot = document.getElementById('syncDot');
      const text = document.getElementById('statusText');
      
      text.innerText = "SINKRONISASI...";
      dot.className = "w-2 h-2 bg-blue-500 rounded-full animate-ping";
      
      try {
        const response = await fetch(`${SPREADSHEET_URL}?action=read&_t=${Date.now()}`);
        const result = await response.json();
        if (result.status === "success") {
          db = result.data || [];
          renderRekap(db);
          text.innerText = `AKTIF: ${db.length} DATA`;
          dot.className = "w-2 h-2 bg-emerald-400 rounded-full shadow-[0_0_8px_rgba(52,211,153,0.6)]";
        }
      } catch (err) {
        text.innerText = "";
        dot.className = "w-2 h-2 bg-slate-400 rounded-full";
      }
    }

    function renderRekap(data = db) {
      const body = document.getElementById('rekapTableBody');
      document.getElementById('recordCount').innerText = data.length;
      if (data.length === 0) {
        body.innerHTML = '<tr><td colspan="6" class="p-20 text-center text-slate-400 italic">Belum ada riwayat pengiriman data</td></tr>';
        return;
      }
      const sorted = [...data].sort((a,b) => new Date(b.tanggal) - new Date(a.tanggal));
      body.innerHTML = sorted.map(i => `
        <tr class="hover:bg-slate-50 transition-colors group">
          <td class="px-8 py-5">
            <div class="font-mono font-black text-blue-600 text-[11px] leading-tight">${i.resi || '-'}</div>
            <div class="text-[10px] font-bold text-slate-400 mt-0.5">${i.tanggal || '-'}</div>
          </td>
          <td class="px-8 py-5 font-bold text-slate-700 leading-relaxed">${i.no_surat || '-'}</td>
          <td class="px-8 py-5 text-slate-500 text-xs leading-relaxed max-w-xs truncate" title="${i.tujuan || ''}">${i.tujuan || '-'}</td>
          <td class="px-8 py-5 text-[11px] font-bold text-slate-600">${i.pengirim || '-'}</td>
          <td class="px-8 py-5 text-center font-black text-slate-800">${i.jumlah || '1'}</td>
          <td class="px-8 py-5">
            <span class="inline-flex items-center px-2.5 py-1 rounded-full bg-emerald-50 text-emerald-600 text-[9px] font-black uppercase tracking-widest border border-emerald-100">
                <span class="w-1 h-1 bg-emerald-500 rounded-full mr-1.5"></span>
                Cloud
            </span>
          </td>
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
      const originalText = btn.innerText;
      
      btn.disabled = true; 
      btn.innerHTML = `<span class="flex items-center justify-center gap-2"><span class="loader"></span> MENYIMPAN...</span>`;
      
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
        db.unshift(entry); 
        renderRekap(db);
        toast("Data Berhasil Tersimpan di Cloud!");
        e.target.reset();
        document.getElementById('tanggalKirim').valueAsDate = new Date();
        setTimeout(() => {
            switchTab('Rekap');
            fetchAllData();
        }, 500);
      } catch (err) {
        toast("Kesalahan Jaringan: Gagal Simpan", "red");
      } finally {
        btn.disabled = false; 
        btn.innerText = originalText;
      }
    };

    function setYearlyFilter(year) {
        if(!year) return;
        document.getElementById('reportStartDate').value = `${year}-01-01`;
        document.getElementById('reportEndDate').value = `${year}-12-31`;
        previewReportAsLetter();
    }

    function previewReportAsLetter() {
      const start = document.getElementById('reportStartDate').value;
      const end = document.getElementById('reportEndDate').value;
      if(!start || !end) return toast("Silakan tentukan periode tanggal", "red");

      const filtered = db.filter(i => i.tanggal >= start && i.tanggal <= end)
                         .sort((a,b) => new Date(a.tanggal) - new Date(b.tanggal));

      if (filtered.length === 0) {
        document.getElementById('previewSuratWrapper').classList.add('hidden');
        return toast("Data tidak ditemukan pada periode ini", "red");
      }
      
      document.getElementById('letterPeriodRange').innerText = `${new Date(start).toLocaleDateString('id-ID', {day:'numeric', month:'long', year:'numeric'})} s/d ${new Date(end).toLocaleDateString('id-ID', {day:'numeric', month:'long', year:'numeric'})}`;
      document.getElementById('letterPrintDate').innerText = `${new Date().toLocaleDateString('id-ID', {day:'numeric', month:'long', year:'numeric'})}`;
      
      document.getElementById('letterTableContent').innerHTML = filtered.map((i, idx) => `
        <tr>
          <td class="text-center">${idx + 1}</td>
          <td class="text-center">${i.tanggal}</td>
          <td class="font-bold">${i.no_surat}</td>
          <td class="uppercase text-[11px] leading-snug">${i.tujuan}</td>
          <td class="text-center font-bold">${i.jumlah || '1'}</td>
        </tr>
      `).join('');
      
      document.getElementById('previewSuratWrapper').classList.remove('hidden');
      document.getElementById('previewSuratWrapper').scrollIntoView({ behavior: 'smooth' });
    }

    async function downloadPDF() {
      const element = document.getElementById('printableArea');
      if (!element || document.getElementById('previewSuratWrapper').classList.contains('hidden')) {
          return toast("Tampilkan laporan terlebih dahulu", "red");
      }
      
      toast("Sedang mengonversi ke PDF...");
      try {
        const { jsPDF } = window.jspdf;
        const canvas = await html2canvas(element, { scale: 3, useCORS: true });
        const imgData = canvas.toDataURL('image/png');
        const pdf = new jsPDF('p', 'mm', 'a4');
        const pdfWidth = pdf.internal.pageSize.getWidth();
        const pdfHeight = (canvas.height * pdfWidth) / canvas.width;
        
        pdf.addImage(imgData, 'PNG', 0, 0, pdfWidth, pdfHeight);
        pdf.save(`REKAP_SURAT_${document.getElementById('reportStartDate').value}_sd_${document.getElementById('reportEndDate').value}.pdf`);
        toast("Unduhan Berhasil!");
      } catch (err) {
        toast("Gagal memproses file PDF", "red");
      }
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
