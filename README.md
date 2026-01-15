<!doctype html>
<html lang="id" class="h-full">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Sistem Kirim Surat - Auto Sync Cloud</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;500;600;700&display=swap');
    * { font-family: 'Plus Jakarta Sans', sans-serif; box-sizing: border-box; }
    
    .toast {
      position: fixed; top: 20px; right: 20px;
      padding: 1rem 1.5rem; border-radius: 0.5rem;
      color: white; z-index: 1000; box-shadow: 0 10px 15px -3px rgba(0,0,0,0.1);
      animation: slideIn 0.3s ease-out;
    }

    @keyframes slideIn { from { transform: translateX(100%); opacity: 0; } to { transform: translateX(0); opacity: 1; } }
    .tab-active { border-bottom: 3px solid #4f46e5; color: #4f46e5; background: #f5f3ff; }

    .loader {
      border: 2px solid #f3f3f3; border-top: 2px solid #4f46e5;
      border-radius: 50%; width: 14px; height: 14px;
      animation: spin 1s linear infinite; display: inline-block;
    }
    @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
  </style>
</head>
<body class="h-full bg-slate-50 text-slate-900">
  <div id="app" class="min-h-screen flex flex-col">
    <!-- Header -->
    <header class="bg-white border-b sticky top-0 z-20">
      <div class="max-w-7xl mx-auto px-4 py-4 flex items-center justify-between">
        <div>
          <h1 class="text-2xl font-black text-indigo-900">SISTEM SURAT CLOUD</h1>
          <div class="flex items-center gap-2">
            <span id="syncStatus" class="flex items-center gap-1 text-[10px] text-emerald-600 font-bold uppercase">
              <span class="w-2 h-2 bg-emerald-500 rounded-full animate-pulse"></span> Menghubungkan...
            </span>
          </div>
        </div>
        <div class="flex items-center gap-3">
          <div id="autoSyncBadge" class="hidden sm:flex items-center gap-2 px-3 py-1 bg-slate-100 rounded-full text-[9px] font-bold text-slate-500 uppercase tracking-wider">
            <span class="relative flex h-2 w-2">
              <span class="animate-ping absolute inline-flex h-full w-full rounded-full bg-indigo-400 opacity-75"></span>
              <span class="relative inline-flex rounded-full h-2 w-2 bg-indigo-500"></span>
            </span>
            Auto-Sync Aktif
          </div>
          <button onclick="fetchAllData()" class="p-2 bg-indigo-50 text-indigo-700 hover:bg-indigo-100 rounded-lg transition-all" title="Refresh Manual">
            🔄
          </button>
        </div>
      </div>
    </header>

    <!-- Navigasi -->
    <nav class="bg-white border-b">
      <div class="max-w-7xl mx-auto px-4 flex gap-8">
        <button id="tabInput" class="tab-active py-4 px-2 font-bold text-sm">📝 INPUT</button>
        <button id="tabRekap" class="py-4 px-2 font-bold text-sm text-slate-500">📋 REKAP CLOUD</button>
        <button id="tabLaporan" class="py-4 px-2 font-bold text-sm text-slate-500">📊 LAPORAN</button>
      </div>
    </nav>

    <!-- Konten -->
    <main class="flex-grow max-w-7xl mx-auto w-full px-4 py-8">
      
      <!-- TAB 1: INPUT -->
      <section id="contentInput" class="tab-content">
        <div class="bg-white rounded-2xl shadow-sm border border-slate-200 max-w-2xl mx-auto overflow-hidden">
          <div class="p-6 border-b bg-slate-50/50">
            <h2 class="font-bold text-slate-700 uppercase text-xs tracking-widest">Input Pengiriman</h2>
          </div>
          <form id="formPengiriman" class="p-8 space-y-6">
             <div class="grid grid-cols-2 gap-4">
              <div class="space-y-1">
                <label class="text-[10px] font-black uppercase text-slate-400">Nomor Surat</label>
                <input type="text" id="nomorSurat" required class="w-full px-4 py-2 bg-white border border-slate-200 rounded-lg focus:ring-2 focus:ring-indigo-500 outline-none" placeholder="001/ADM/2026">
              </div>
              <div class="space-y-1">
                <label class="text-[10px] font-black uppercase text-slate-400">Tanggal Kirim</label>
                <input type="date" id="tanggalKirim" required class="w-full px-4 py-2 bg-white border border-slate-200 rounded-lg focus:ring-2 focus:ring-indigo-500 outline-none">
              </div>
            </div>
            <div class="space-y-1">
              <label class="text-[10px] font-black uppercase text-slate-400">Instansi Tujuan</label>
              <textarea id="alamatTujuan" required rows="3" class="w-full px-4 py-2 bg-white border border-slate-200 rounded-lg focus:ring-2 focus:ring-indigo-500 outline-none" placeholder="Nama instansi..."></textarea>
            </div>
            <div class="grid grid-cols-2 gap-4">
              <div class="space-y-1">
                <label class="text-[10px] font-black uppercase text-slate-400">Jumlah Surat</label>
                <input type="number" id="jumlahSurat" required value="1" min="1" class="w-full px-4 py-2 bg-white border border-slate-200 rounded-lg focus:ring-2 focus:ring-indigo-500 outline-none">
              </div>
              <div class="space-y-1">
                <label class="text-[10px] font-black uppercase text-slate-400">Nama Pengirim</label>
                <input type="text" id="namaPengirim" required class="w-full px-4 py-2 bg-white border border-slate-200 rounded-lg focus:ring-2 focus:ring-indigo-500 outline-none">
              </div>
            </div>
            <button type="submit" id="submitBtn" class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-4 rounded-xl shadow-lg transition-all flex items-center justify-center gap-2">
              KIRIM KE DATABASE CLOUD
            </button>
          </form>
        </div>
      </section>

      <!-- TAB 2: REKAP -->
      <section id="contentRekap" class="tab-content hidden">
        <div class="bg-white rounded-2xl shadow-sm border border-slate-200 overflow-hidden">
          <div class="p-4 border-b bg-slate-50/50 flex items-center gap-4">
            <div class="relative flex-grow">
              <span class="absolute inset-y-0 left-0 pl-3 flex items-center text-slate-400">🔍</span>
              <input type="text" id="searchBox" class="w-full pl-10 px-4 py-2 border rounded-lg outline-none focus:ring-2 focus:ring-indigo-500" placeholder="Cari data cloud...">
            </div>
          </div>
          <div class="overflow-x-auto">
            <table class="w-full text-left">
              <thead>
                <tr class="text-[10px] font-black uppercase text-slate-400 border-b">
                  <th class="px-6 py-4">Resi</th>
                  <th class="px-6 py-4">No Surat</th>
                  <th class="px-6 py-4">Tanggal</th>
                  <th class="px-6 py-4">Tujuan</th>
                  <th class="px-6 py-4">Status</th>
                </tr>
              </thead>
              <tbody id="rekapTableBody" class="divide-y text-sm">
                <tr><td colspan="5" class="p-10 text-center text-slate-400 italic">Menyinkronkan data...</td></tr>
              </tbody>
            </table>
          </div>
        </div>
      </section>

      <!-- TAB 3: LAPORAN -->
      <section id="contentLaporan" class="tab-content hidden">
         <div class="bg-white p-12 rounded-2xl border-2 border-dashed border-slate-200 flex flex-col items-center justify-center text-slate-400">
           <div class="w-16 h-16 bg-slate-50 rounded-full flex items-center justify-center mb-4">📊</div>
           <p class="font-bold text-slate-700">Database Cloud Sinkron</p>
           <p class="text-sm max-w-xs text-center mt-1">Gunakan tab Rekap Cloud untuk melihat data yang masuk dari semua perangkat.</p>
         </div>
      </section>

    </main>
  </div>

  <script>
    // URL SCRIPT ANDA
    const SPREADSHEET_URL = "https://script.google.com/macros/s/AKfycby2-tQBfdDfSOnLqLCKNM2oZtRNO_-wEjNbUnnp-dNiLCF21hsy9Kc0ut3hsXmcDOg/exec";
    
    let db = []; 
    let syncInterval;

    function toast(msg, color='indigo') {
      const t = document.createElement('div');
      t.className = `toast ${color === 'indigo' ? 'bg-indigo-600' : 'bg-red-600'}`;
      t.innerText = msg;
      document.body.appendChild(t);
      setTimeout(() => t.remove(), 3000);
    }

    // Fungsi Fetch Data
    async function fetchAllData() {
      const status = document.getElementById('syncStatus');
      const tableBody = document.getElementById('rekapTableBody');
      const badge = document.getElementById('autoSyncBadge');
      
      status.innerHTML = `<span class="loader"></span> SINKRON...`;
      
      try {
        const response = await fetch(`${SPREADSHEET_URL}?action=read&t=${Date.now()}`);
        const result = await response.json();
        
        if (result.status === "success") {
          db = result.data;
          renderRekap();
          status.innerHTML = `<span class="w-2 h-2 bg-emerald-500 rounded-full"></span> SINKRON: ${db.length} DATA`;
          badge.classList.remove('hidden');
        }
      } catch (err) {
        console.error("Sync error:", err);
        status.innerHTML = `<span class="w-2 h-2 bg-red-500 rounded-full"></span> GAGAL SINKRON`;
      }
    }

    // Navigasi
    const tabs = ['Input', 'Rekap', 'Laporan'];
    tabs.forEach(t => {
      document.getElementById(`tab${t}`).onclick = () => {
        tabs.forEach(x => {
          document.getElementById(`tab${x}`).classList.remove('tab-active');
          document.getElementById(`content${x}`).classList.add('hidden');
        });
        document.getElementById(`tab${t}`).classList.add('tab-active');
        document.getElementById(`content${t}`).classList.remove('hidden');
        if(t === 'Rekap') fetchAllData();
      };
    });

    // Form Submit
    document.getElementById('formPengiriman').onsubmit = async (e) => {
      e.preventDefault();
      const btn = document.getElementById('submitBtn');
      btn.disabled = true; 
      btn.innerHTML = `<span class="loader"></span> MENGIRIM...`;

      const entry = {
        action: "insert",
        id: Date.now(),
        resi: "RSI-" + Math.random().toString(36).substr(2, 6).toUpperCase(),
        no_surat: document.getElementById('nomorSurat').value,
        tanggal: document.getElementById('tanggalKirim').value,
        tujuan: document.getElementById('alamatTujuan').value,
        jumlah: document.getElementById('jumlahSurat').value,
        pengirim: document.getElementById('namaPengirim').value,
        status: "Terkirim"
      };

      try {
        await fetch(SPREADSHEET_URL, {
          method: 'POST',
          mode: 'no-cors',
          body: JSON.stringify(entry)
        });
        
        toast("✓ Data Berhasil Dikirim ke Cloud");
        e.target.reset();
        document.getElementById('tanggalKirim').valueAsDate = new Date();
        
        // Segarkan data setelah kirim
        setTimeout(fetchAllData, 1000); 
      } catch (err) {
        toast("Gagal koneksi", "red");
      } finally {
        btn.disabled = false; 
        btn.innerText = "KIRIM KE DATABASE CLOUD";
      }
    };

    function renderRekap(filterData = db) {
      const body = document.getElementById('rekapTableBody');
      if (filterData.length === 0) {
        body.innerHTML = `<tr><td colspan="5" class="p-10 text-center text-slate-400">Database cloud kosong.</td></tr>`;
        return;
      }

      // Selalu tampilkan yang terbaru di paling atas
      const sortedData = [...filterData].reverse();

      body.innerHTML = sortedData.map(item => `
        <tr class="hover:bg-slate-50 transition-colors">
          <td class="px-6 py-4 font-mono font-bold text-indigo-600">${item.resi || '-'}</td>
          <td class="px-6 py-4 font-medium">${item.no_surat || '-'}</td>
          <td class="px-6 py-4 text-slate-500">${item.tanggal || '-'}</td>
          <td class="px-6 py-4 max-w-xs truncate text-slate-600">${item.tujuan || '-'}</td>
          <td class="px-6 py-4">
             <span class="px-2 py-1 rounded text-[10px] font-bold bg-indigo-50 text-indigo-600 uppercase tracking-tighter">TERKIRIM</span>
          </td>
        </tr>
      `).join('');
    }

    // Pencarian
    document.getElementById('searchBox').oninput = (e) => {
      const q = e.target.value.toLowerCase();
      const filtered = db.filter(i => 
        (i.resi || "").toLowerCase().includes(q) || 
        (i.no_surat || "").toLowerCase().includes(q) || 
        (i.tujuan || "").toLowerCase().includes(q)
      );
      renderRekap(filtered);
    };

    // Auto-Sync Setup
    function startAutoSync() {
      // Jalankan fetch pertama kali
      fetchAllData();
      // Kemudian ulangi setiap 30 detik
      syncInterval = setInterval(fetchAllData, 30000);
    }

    window.onload = () => {
      document.getElementById('tanggalKirim').valueAsDate = new Date();
      startAutoSync();
    };
  </script>
</body>
</html>
