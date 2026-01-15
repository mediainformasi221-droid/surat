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

    .paper-preview {
      background: white;
      box-shadow: 0 0 20px rgba(0,0,0,0.1);
      width: 210mm;
      margin: 0 auto;
      min-height: 297mm;
      padding: 2cm 2.5cm;
      position: relative;
      color: #000;
      font-family: Arial, Helvetica, sans-serif;
    }
    .letter-header {
      text-align: center;
      border-bottom: 3px solid #000;
      margin-bottom: 0.5rem;
      padding-bottom: 0.5rem;
    }
    .header-text p { margin: 0; padding: 0; }
    
    .table-laporan { width: 100%; border-collapse: collapse; font-size: 11px; }
    .table-laporan th, .table-laporan td { border: 1px solid #000; padding: 6px 4px; }
    .table-laporan th { background-color: #f2f2f2; font-weight: bold; text-transform: uppercase; }
    
    @media print {
      body { background: none; }
      .no-print { display: none; }
      .paper-preview { box-shadow: none; margin: 0 !important; padding: 1cm !important; width: 100% !important; }
    }
  </style>
</head>
<body class="h-full bg-slate-100 text-slate-900">
  <div id="app" class="min-h-screen flex flex-col">
    <!-- Header Aplikasi -->
    <header class="bg-white border-b sticky top-0 z-20 no-print">
      <div class="max-w-7xl mx-auto px-4 py-4 flex items-center justify-between">
        <div>
          <h1 class="text-2xl font-black text-indigo-900 italic tracking-tighter uppercase leading-none">Sistem Pengiriman Surat</h1>
          <div class="flex items-center gap-2 mt-1">
            <span id="syncStatus" class="flex items-center gap-1 text-[10px] text-emerald-600 font-bold uppercase">
              <!-- Hanya dot indikator, teks akan diisi lewat JS -->
              <span id="syncDot" class="w-2 h-2 bg-emerald-500 rounded-full animate-pulse"></span> 
              <span id="statusText">Menyambungkan...</span>
            </span>
          </div>
        </div>
        <button onclick="fetchAllData()" class="p-2 bg-slate-100 hover:bg-slate-200 rounded-lg transition-all" title="Segarkan Data">🔄</button>
      </div>
    </header>

    <!-- Navigasi -->
    <nav class="bg-white border-b shadow-sm no-print">
      <div class="max-w-7xl mx-auto px-4 flex gap-8">
        <button id="tabInput" class="tab-active py-4 px-2 font-bold text-xs uppercase tracking-widest transition-all">📝 Input</button>
        <button id="tabRekap" class="py-4 px-2 font-bold text-xs uppercase tracking-widest text-slate-400 transition-all">📋 Rekap</button>
        <button id="tabLaporan" class="py-4 px-2 font-bold text-xs uppercase tracking-widest text-slate-400 transition-all">📊 Laporan</button>
      </div>
    </nav>

    <!-- Konten Utama -->
    <main class="flex-grow max-w-7xl mx-auto w-full px-4 py-8">
      
      <!-- HALAMAN 1: FORM INPUT -->
      <section id="contentInput" class="tab-content">
        <div class="bg-white rounded-2xl shadow-sm border border-slate-200 max-w-2xl mx-auto overflow-hidden">
          <div class="p-6 border-b bg-slate-50/50">
            <h2 class="font-bold text-slate-700 text-sm uppercase tracking-wider text-center">Formulir Pengiriman Baru</h2>
          </div>
          <form id="formPengiriman" class="p-8 space-y-6">
             <div class="grid grid-cols-2 gap-6">
              <div class="space-y-2">
                <label class="text-[10px] font-black uppercase text-slate-400 tracking-widest">Nomor Surat</label>
                <input type="text" id="nomorSurat" required class="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl outline-none focus:ring-2 focus:ring-indigo-500 transition-all" placeholder="001/ADM/2026">
              </div>
              <div class="space-y-2">
                <label class="text-[10px] font-black uppercase text-slate-400 tracking-widest">Tanggal Kirim</label>
                <input type="date" id="tanggalKirim" required class="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl outline-none focus:ring-2 focus:ring-indigo-500">
              </div>
            </div>
            <div class="space-y-2">
              <label class="text-[10px] font-black uppercase text-slate-400 tracking-widest">Instansi Tujuan</label>
              <textarea id="alamatTujuan" required rows="3" class="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl outline-none focus:ring-2 focus:ring-indigo-500" placeholder="Contoh: KPP Pratama Serang..."></textarea>
            </div>
            <div class="grid grid-cols-2 gap-6">
              <div class="space-y-2">
                <label class="text-[10px] font-black uppercase text-slate-400 tracking-widest">Jumlah Berkas</label>
                <input type="number" id="jumlahSurat" required value="1" min="1" class="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl outline-none focus:ring-2 focus:ring-indigo-500">
              </div>
              <div class="space-y-2">
                <label class="text-[10px] font-black uppercase text-slate-400 tracking-widest">Petugas Pengirim</label>
                <input type="text" id="namaPengirim" required class="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl outline-none focus:ring-2 focus:ring-indigo-500" placeholder="Nama Petugas">
              </div>
            </div>
            <button type="submit" id="submitBtn" class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-4 rounded-xl shadow-lg transition-all">SIMPAN & SINKRONKAN</button>
          </form>
        </div>
      </section>

      <!-- HALAMAN 2: REKAP -->
      <section id="contentRekap" class="tab-content hidden">
        <div class="bg-white rounded-2xl shadow-sm border border-slate-200 overflow-hidden">
          <div class="p-6 border-b bg-slate-50/50 flex flex-col md:flex-row justify-between items-center gap-4">
            <div class="relative w-full md:w-96">
                <span class="absolute left-3 top-2.5 text-slate-400">🔍</span>
                <input type="text" id="searchBox" onkeyup="handleSearch(this.value)" class="w-full pl-10 pr-4 py-2 bg-white border border-slate-200 rounded-xl outline-none focus:ring-2 focus:ring-indigo-500 transition-all" placeholder="Cari nomor surat atau tujuan...">
            </div>
            <div class="text-right">
                <p class="text-[10px] font-black text-slate-400 uppercase tracking-widest leading-none">Total Data</p>
                <p id="recordCount" class="text-xl font-bold text-indigo-600">0</p>
            </div>
          </div>
          <div class="overflow-x-auto">
            <table class="w-full text-left">
              <thead>
                <tr class="text-[10px] font-black uppercase text-slate-500 border-b bg-slate-100/50">
                  <th class="px-6 py-4 w-32">Resi & Tgl</th>
                  <th class="px-6 py-4">Nomor Surat</th>
                  <th class="px-6 py-4">Tujuan</th>
                  <th class="px-6 py-4 w-28">Pengirim</th>
                  <th class="px-6 py-4 w-16 text-center">Jml</th>
                  <th class="px-6 py-4 w-20">Status</th>
                </tr>
              </thead>
              <tbody id="rekapTableBody" class="divide-y text-sm">
                <tr><td colspan="6" class="p-20 text-center"><span class="loader"></span> Memuat data...</td></tr>
              </tbody>
            </table>
          </div>
        </div>
      </section>

      <!-- HALAMAN 3: LAPORAN -->
      <section id="contentLaporan" class="tab-content hidden">
        <div class="max-w-5xl mx-auto space-y-6">
          <div class="bg-white p-8 rounded-2xl shadow-sm border border-slate-200 no-print">
            <div class="grid grid-cols-1 md:grid-cols-4 gap-4 items-end">
              <div class="space-y-2">
                <label class="text-[10px] font-black uppercase text-slate-400 tracking-widest">Tahun Laporan</label>
                <select id="yearSelector" onchange="setYearlyFilter(this.value)" class="w-full px-4 py-2 bg-slate-50 border border-slate-200 rounded-lg outline-none focus:ring-2 focus:ring-indigo-500">
                    <option value="">-- Pilih Tahun --</option>
                    <script>
                        const yr = new Date().getFullYear();
                        for(let i=0; i<5; i++) document.write(`<option value="${yr-i}">${yr-i}</option>`);
                    </script>
                </select>
              </div>
              <div class="space-y-2">
                <label class="text-[10px] font-black uppercase text-slate-400 tracking-widest">Mulai</label>
                <input type="date" id="reportStartDate" class="w-full px-4 py-2 bg-slate-50 border border-slate-200 rounded-lg outline-none">
              </div>
              <div class="space-y-2">
                <label class="text-[10px] font-black uppercase text-slate-400 tracking-widest">Sampai</label>
                <input type="date" id="reportEndDate" class="w-full px-4 py-2 bg-slate-50 border border-slate-200 rounded-lg outline-none">
              </div>
              <div class="flex gap-2">
                <button onclick="previewReportAsLetter()" class="flex-grow px-4 py-2 bg-indigo-600 hover:bg-indigo-700 text-white font-bold rounded-lg transition-all">Lihat</button>
                <button onclick="window.print()" class="px-4 py-2 bg-slate-900 hover:bg-black text-white font-bold rounded-lg transition-all">Cetak</button>
                <button onclick="downloadPDF()" class="px-4 py-2 bg-emerald-600 hover:bg-emerald-700 text-white font-bold rounded-lg transition-all" title="Unduh PDF">💾 PDF</button>
              </div>
            </div>
          </div>

          <!-- Preview -->
          <div id="previewSuratWrapper" class="hidden">
            <div id="printableArea" class="paper-preview">
              <div class="letter-header">
                <div class="header-text">
                  <p class="text-[14px] font-bold">KEMENTERIAN KEUANGAN REPUBLIK INDONESIA</p>
                  <p class="text-[13px] font-bold">DIREKTORAT JENDERAL KEKAYAAN NEGARA</p>
                  <p class="text-[11px] font-bold">KANTOR WILAYAH DJKN BANTEN</p>
                  <p class="text-[12px] font-bold">KANTOR PELAYANAN KEKAYAAN NEGARA DAN LELANG SERANG</p>
                  <p class="text-[10px] mt-1">Jalan Raya Serang-Cilegon Km. 3, Serang 42162 Telepon (0254) 216361</p>
                </div>
              </div>

              <div class="mt-8 space-y-6">
                <div class="text-center">
                    <p class="font-bold underline uppercase text-[12px] tracking-widest">LAPORAN REKAPITULASI PENGIRIMAN SURAT</p>
                    <p class="text-[10px] font-bold mt-1">Periode: <span id="letterPeriodRange"></span></p>
                </div>

                <table class="table-laporan">
                  <thead>
                    <tr>
                      <th class="w-8">NO</th>
                      <th class="w-24">TANGGAL KIRIM</th>
                      <th class="w-48">NOMOR SURAT</th>
                      <th>INSTANSI TUJUAN</th>
                      <th class="w-12">JML</th>
                    </tr>
                  </thead>
                  <tbody id="letterTableContent"></tbody>
                </table>

                <div class="mt-12 grid grid-cols-2 gap-8 text-[11px]">
                  <div class="text-center">
                    <p class="font-bold mb-20 uppercase">PETUGAS PENGIRIM</p>
                    <p class="font-bold underline">( ..................................... )</p>
                  </div>
                  <div class="text-center">
                    <p class="font-bold mb-4 uppercase">Serang, <span id="letterPrintDate"></span></p>
                    <p class="font-bold mb-16 uppercase">PENERIMA SURAT / EKSPEDISI</p>
                    <p class="font-bold underline">( ..................................... )</p>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </section>
    </main>
  </div>

  <script>
    const SPREADSHEET_URL = "https://script.google.com/macros/s/AKfycbzh25RHh6tcCjRrixucsEVx6ugi4IwkxmJsM8-yyFCnByIzaTS3ohooIMgx7Yv7_ckP/exec";
    let db = [];

    function toast(msg, color='indigo') {
      const t = document.createElement('div');
      t.className = `toast ${color === 'indigo' ? 'bg-indigo-600' : 'bg-red-600'}`;
      t.innerText = msg;
      document.body.appendChild(t);
      setTimeout(() => t.remove(), 3000);
    }

    function switchTab(tabId) {
      const tabs = ['Input', 'Rekap', 'Laporan'];
      tabs.forEach(x => {
        document.getElementById(`tab${x}`).classList.replace('tab-active', 'text-slate-400');
        document.getElementById(`content${x}`).classList.add('hidden');
      });
      document.getElementById(`tab${tabId}`).classList.replace('text-slate-400', 'tab-active');
      document.getElementById(`content${tabId}`).classList.remove('hidden');
    }

    async function fetchAllData() {
      const statusContainer = document.getElementById('syncStatus');
      const dot = document.getElementById('syncDot');
      const text = document.getElementById('statusText');
      
      text.innerText = "SINKRONISASI...";
      dot.className = "w-2 h-2 bg-indigo-500 rounded-full animate-pulse";
      
      try {
        const response = await fetch(`${SPREADSHEET_URL}?action=read&_t=${Date.now()}`);
        const result = await response.json();
        if (result.status === "success") {
          db = result.data || [];
          renderRekap(db);
          text.innerText = `AKTIF: ${db.length} DATA`;
          dot.className = "w-2 h-2 bg-emerald-500 rounded-full";
        }
      } catch (err) {
        // Hapus teks offline/terputus, hanya dot warna abu-abu
        text.innerText = "";
        dot.className = "w-2 h-2 bg-slate-300 rounded-full";
      }
    }

    function renderRekap(data = db) {
      const body = document.getElementById('rekapTableBody');
      document.getElementById('recordCount').innerText = data.length;
      if (data.length === 0) {
        body.innerHTML = '<tr><td colspan="6" class="p-10 text-center text-slate-400">Belum ada data</td></tr>';
        return;
      }
      const sorted = [...data].sort((a,b) => new Date(b.tanggal) - new Date(a.tanggal));
      body.innerHTML = sorted.map(i => `
        <tr class="hover:bg-indigo-50 border-b border-slate-50">
          <td class="px-6 py-3">
            <div class="font-mono font-bold text-indigo-600 text-[11px]">${i.resi || '-'}</div>
            <div class="text-[10px] text-slate-400">${i.tanggal || '-'}</div>
          </td>
          <td class="px-6 py-3 font-bold text-slate-700">${i.no_surat || '-'}</td>
          <td class="px-6 py-3 text-slate-600 text-xs">${i.tujuan || '-'}</td>
          <td class="px-6 py-3 text-[11px] font-semibold italic text-slate-500">${i.pengirim || '-'}</td>
          <td class="px-6 py-3 text-center font-bold">${i.jumlah || '1'}</td>
          <td class="px-6 py-3"><span class="px-2 py-0.5 rounded bg-emerald-100 text-emerald-700 text-[9px] font-bold">TERKIRIM</span></td>
        </tr>
      `).join('');
    }

    function handleSearch(q) {
      const query = q.toLowerCase();
      const filtered = db.filter(i => 
        (i.tujuan?.toLowerCase().includes(query)) || 
        (i.no_surat?.toLowerCase().includes(query))
      );
      renderRekap(filtered);
    }

    document.getElementById('formPengiriman').onsubmit = async (e) => {
      e.preventDefault();
      const btn = document.getElementById('submitBtn');
      btn.disabled = true; btn.innerHTML = `<span class="loader"></span> MENYIMPAN...`;
      
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
        toast("Data Berhasil Disimpan!");
        e.target.reset();
        document.getElementById('tanggalKirim').valueAsDate = new Date();
        setTimeout(() => {
            switchTab('Rekap');
            fetchAllData();
        }, 800);
      } catch (err) {
        toast("Gagal Simpan ke Cloud", "red");
      } finally {
        btn.disabled = false; btn.innerText = "SIMPAN & SINKRONKAN";
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
      if(!start || !end) return toast("Pilih Tanggal Laporan", "red");

      const filtered = db.filter(i => i.tanggal >= start && i.tanggal <= end)
                         .sort((a,b) => new Date(a.tanggal) - new Date(b.tanggal));

      if (filtered.length === 0) {
        document.getElementById('previewSuratWrapper').classList.add('hidden');
        return toast("Tidak ada data", "red");
      }
      
      document.getElementById('letterPeriodRange').innerText = `${new Date(start).toLocaleDateString('id-ID')} s/d ${new Date(end).toLocaleDateString('id-ID')}`;
      document.getElementById('letterPrintDate').innerText = `${new Date().toLocaleDateString('id-ID', {day:'numeric', month:'long', year:'numeric'})}`;
      
      document.getElementById('letterTableContent').innerHTML = filtered.map((i, idx) => `
        <tr>
          <td class="text-center font-bold">${idx + 1}</td>
          <td class="text-center">${i.tanggal}</td>
          <td class="font-bold">${i.no_surat}</td>
          <td class="uppercase">${i.tujuan}</td>
          <td class="text-center">${i.jumlah || '1'}</td>
        </tr>
      `).join('');
      
      document.getElementById('previewSuratWrapper').classList.remove('hidden');
      document.getElementById('previewSuratWrapper').scrollIntoView({ behavior: 'smooth' });
    }

    async function downloadPDF() {
      const element = document.getElementById('printableArea');
      if (!element) return toast("Tampilkan laporan dahulu", "red");
      
      toast("Sedang memproses PDF...");
      try {
        const { jsPDF } = window.jspdf;
        const canvas = await html2canvas(element, { scale: 2 });
        const imgData = canvas.toDataURL('image/png');
        const pdf = new jsPDF('p', 'mm', 'a4');
        const imgProps = pdf.getImageProperties(imgData);
        const pdfWidth = pdf.internal.pageSize.getWidth();
        const pdfHeight = (imgProps.height * pdfWidth) / imgProps.width;
        
        pdf.addImage(imgData, 'PNG', 0, 0, pdfWidth, pdfHeight);
        pdf.save(`Laporan_Pengiriman_${document.getElementById('reportStartDate').value}.pdf`);
        toast("PDF Berhasil Diunduh!");
      } catch (err) {
        console.error(err);
        toast("Gagal membuat PDF", "red");
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
