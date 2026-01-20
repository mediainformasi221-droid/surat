<!doctype html>
<html lang="id" class="h-full">
 <head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Sistem Kirim Surat</title>
  <script src="/_sdk/data_sdk.js"></script>
  <script src="/_sdk/element_sdk.js"></script>
  <script src="https://cdn.tailwindcss.com"></script>
  <style>
    body {
      box-sizing: border-box;
    }
    
    @import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;500;600;700&display=swap');
    
    * {
      font-family: 'Plus Jakarta Sans', sans-serif;
    }
    
    .toast {
      position: fixed;
      top: 20px;
      right: 20px;
      background: #10b981;
      color: white;
      padding: 1rem 1.5rem;
      border-radius: 0.5rem;
      box-shadow: 0 4px 6px rgba(0,0,0,0.1);
      z-index: 1000;
      animation: slideIn 0.3s ease-out;
    }
    
    @keyframes slideIn {
      from {
        transform: translateX(400px);
        opacity: 0;
      }
      to {
        transform: translateX(0);
        opacity: 1;
      }
    }
    
    .tab-active {
      border-bottom: 3px solid #3b82f6;
      color: #3b82f6;
    }
    
    @media print {
      body * {
        visibility: hidden;
      }
      #printArea, #printArea * {
        visibility: visible;
      }
      #printArea {
        position: absolute;
        left: 0;
        top: 0;
        width: 100%;
      }
      #amplopPrintArea, #amplopPrintArea * {
        visibility: visible;
      }
      #amplopPrintArea {
        position: absolute;
        left: 0;
        top: 0;
      }
    }
    
    @page {
      size: 230mm 110mm;
      margin: 0;
    }
  </style>
  <style>@view-transition { navigation: auto; }</style>
 </head>
 <body class="h-full">
  <div id="app" class="h-full w-full bg-gradient-to-br from-blue-50 to-indigo-100 overflow-auto"><!-- Header -->
   <div class="bg-white shadow-md">
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-4">
     <div class="flex items-center justify-between">
      <div>
       <h1 class="text-3xl font-bold text-indigo-900" id="appTitle">Sistem Kirim Surat</h1>
       <p class="text-sm text-gray-600 mt-1" id="instansiName">Sistem Manajemen Pengiriman Surat Digital</p>
      </div>
      <div class="bg-indigo-100 px-4 py-2 rounded-lg">
       <p class="text-xs text-indigo-600 font-medium">Total Pengiriman</p>
       <p class="text-2xl font-bold text-indigo-900" id="totalCount">0</p>
      </div>
     </div>
    </div>
   </div><!-- Tabs Navigation -->
   <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 mt-6">
    <div class="bg-white rounded-lg shadow-sm p-1 flex gap-1"><button id="tabInput" class="tab-active flex-1 py-3 px-4 font-semibold transition-all rounded-md"> 📝 Input Pengiriman </button> <button id="tabPenerima" class="flex-1 py-3 px-4 font-semibold text-gray-600 hover:text-indigo-600 transition-all rounded-md"> 📇 Data Penerima </button> <button id="tabRekap" class="flex-1 py-3 px-4 font-semibold text-gray-600 hover:text-indigo-600 transition-all rounded-md"> 📋 Rekap Resi </button> <button id="tabLaporan" class="flex-1 py-3 px-4 font-semibold text-gray-600 hover:text-indigo-600 transition-all rounded-md"> 📊 Laporan </button>
    </div>
   </div><!-- Content Area -->
   <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-6"><!-- Tab 1: Input Pengiriman -->
    <div id="contentInput" class="content-tab">
     <div class="bg-white rounded-xl shadow-lg p-8">
      <h2 class="text-2xl font-bold text-gray-800 mb-6">Form Input Pengiriman Surat</h2>
      <form id="formPengiriman" class="space-y-5">
       <div class="grid grid-cols-1 md:grid-cols-2 gap-5">
        <div><label for="nomorSurat" class="block text-sm font-semibold text-gray-700 mb-2">Nomor Surat *</label> <input type="text" id="nomorSurat" required class="w-full px-4 py-3 border-2 border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition-all" placeholder="Contoh: 001/SUK/2024">
        </div>
        <div><label for="tanggalKirim" class="block text-sm font-semibold text-gray-700 mb-2">Tanggal Kirim *</label> <input type="date" id="tanggalKirim" required class="w-full px-4 py-3 border-2 border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition-all">
        </div>
       </div>
       <div><label for="pilihPenerima" class="block text-sm font-semibold text-gray-700 mb-2">Pilih Penerima Terdaftar</label> <select id="pilihPenerima" class="w-full px-4 py-3 border-2 border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition-all"> <option value="">-- Pilih dari daftar atau isi manual --</option> </select>
       </div>
       <div><label for="alamatTujuan" class="block text-sm font-semibold text-gray-700 mb-2">Alamat Instansi Tujuan *</label> <textarea id="alamatTujuan" required rows="3" class="w-full px-4 py-3 border-2 border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition-all" placeholder="Masukkan alamat lengkap instansi tujuan atau pilih dari daftar di atas"></textarea>
       </div>
       <div class="grid grid-cols-1 md:grid-cols-2 gap-5">
        <div><label for="jumlahSurat" class="block text-sm font-semibold text-gray-700 mb-2">Jumlah Surat *</label> <input type="number" id="jumlahSurat" required min="1" value="1" class="w-full px-4 py-3 border-2 border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition-all">
        </div>
        <div><label for="namaPengirim" class="block text-sm font-semibold text-gray-700 mb-2">Nama Pengirim *</label> <input type="text" id="namaPengirim" required class="w-full px-4 py-3 border-2 border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition-all" placeholder="Nama lengkap pengirim">
        </div>
       </div>
       <div><label for="keterangan" class="block text-sm font-semibold text-gray-700 mb-2">Keterangan (Opsional)</label> <textarea id="keterangan" rows="2" class="w-full px-4 py-3 border-2 border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition-all" placeholder="Catatan tambahan jika diperlukan"></textarea>
       </div>
       <div class="flex gap-3 pt-4"><button type="submit" id="submitBtn" class="flex-1 bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-3 px-6 rounded-lg transition-all shadow-md hover:shadow-lg"> ����� Simpan Pengiriman </button>
       </div>
      </form>
     </div>
    </div><!-- Tab 2: Data Penerima -->
    <div id="contentPenerima" class="content-tab hidden">
     <div class="grid grid-cols-1 lg:grid-cols-2 gap-6"><!-- Form Tambah Penerima -->
      <div class="bg-white rounded-xl shadow-lg p-8">
       <h2 class="text-2xl font-bold text-gray-800 mb-6">Tambah Data Penerima</h2>
       <form id="formPenerima" class="space-y-5">
        <div><label for="namaInstansiPenerima" class="block text-sm font-semibold text-gray-700 mb-2">Nama Instansi *</label> <input type="text" id="namaInstansiPenerima" required class="w-full px-4 py-3 border-2 border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition-all" placeholder="Contoh: Dinas Pendidikan Kota">
        </div>
        <div><label for="alamatLengkapPenerima" class="block text-sm font-semibold text-gray-700 mb-2">Alamat Lengkap *</label> <textarea id="alamatLengkapPenerima" required rows="4" class="w-full px-4 py-3 border-2 border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition-all" placeholder="Masukkan alamat lengkap dengan jalan, nomor, kota, dan kode pos"></textarea>
        </div><button type="submit" id="submitPenerimaBtn" class="w-full bg-green-600 hover:bg-green-700 text-white font-bold py-3 px-6 rounded-lg transition-all shadow-md hover:shadow-lg"> ➕ Tambah Penerima </button>
       </form>
      </div><!-- Daftar Penerima -->
      <div class="bg-white rounded-xl shadow-lg p-8">
       <h2 class="text-2xl font-bold text-gray-800 mb-6">Daftar Penerima Terdaftar</h2>
       <div class="mb-4"><input type="text" id="searchPenerima" class="w-full px-4 py-2 border-2 border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500" placeholder="🔍 Cari nama instansi...">
       </div>
       <div id="daftarPenerimaContainer" class="space-y-3 max-h-[500px] overflow-y-auto"><!-- Daftar penerima akan ditampilkan di sini -->
       </div>
      </div>
     </div>
    </div><!-- Tab 3: Rekap Resi -->
    <div id="contentRekap" class="content-tab hidden">
     <div class="bg-white rounded-xl shadow-lg p-8">
      <div class="flex justify-between items-center mb-6">
       <h2 class="text-2xl font-bold text-gray-800">Rekap Resi Pengiriman</h2>
      </div><!-- Filter & Search -->
      <div class="grid grid-cols-1 md:grid-cols-3 gap-4 mb-6"><input type="text" id="searchResi" class="px-4 py-2 border-2 border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500" placeholder="🔍 Cari nomor surat/resi..."> <input type="date" id="filterTanggalMulai" class="px-4 py-2 border-2 border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500"> <input type="date" id="filterTanggalAkhir" class="px-4 py-2 border-2 border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500">
      </div><!-- Table -->
      <div class="overflow-x-auto">
       <table class="w-full" id="tabelRekap">
        <thead class="bg-indigo-600 text-white">
         <tr>
          <th class="px-4 py-3 text-left text-sm font-semibold">No Resi</th>
          <th class="px-4 py-3 text-left text-sm font-semibold">Tgl Kirim</th>
          <th class="px-4 py-3 text-left text-sm font-semibold">No Surat</th>
          <th class="px-4 py-3 text-left text-sm font-semibold">Instansi Tujuan</th>
          <th class="px-4 py-3 text-left text-sm font-semibold">Jumlah</th>
          <th class="px-4 py-3 text-left text-sm font-semibold">Status</th>
          <th class="px-4 py-3 text-left text-sm font-semibold">Aksi</th>
         </tr>
        </thead>
        <tbody id="tabelRekapBody" class="divide-y divide-gray-200"><!-- Data akan diisi oleh JavaScript -->
        </tbody>
       </table>
      </div>
     </div>
    </div><!-- Tab 3: Laporan -->
    <div id="contentLaporan" class="content-tab hidden">
     <div class="bg-white rounded-xl shadow-lg p-8">
      <h2 class="text-2xl font-bold text-gray-800 mb-6">Unduh Laporan Periodik</h2>
      <div class="grid grid-cols-1 md:grid-cols-2 gap-6 mb-6">
       <div><label for="laporanTanggalMulai" class="block text-sm font-semibold text-gray-700 mb-2">Dari Tanggal</label> <input type="date" id="laporanTanggalMulai" class="w-full px-4 py-3 border-2 border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500">
       </div>
       <div><label for="laporanTanggalAkhir" class="block text-sm font-semibold text-gray-700 mb-2">Sampai Tanggal</label> <input type="date" id="laporanTanggalAkhir" class="w-full px-4 py-3 border-2 border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500">
       </div>
      </div>
      <div class="flex gap-3"><button id="btnPreviewLaporan" class="flex-1 bg-blue-600 hover:bg-blue-700 text-white font-bold py-3 px-6 rounded-lg transition-all shadow-md"> 👁️ Preview Laporan </button> <button id="btnUnduhLaporan" class="flex-1 bg-purple-600 hover:bg-purple-700 text-white font-bold py-3 px-6 rounded-lg transition-all shadow-md"> 💾 Unduh CSV </button> <button id="btnCetakLaporan" class="flex-1 bg-green-600 hover:bg-green-700 text-white font-bold py-3 px-6 rounded-lg transition-all shadow-md"> ����️ Cetak PDF </button>
      </div><!-- Preview Area -->
      <div id="previewArea" class="mt-8 hidden">
       <div id="printArea" class="bg-white p-8 border-2 border-gray-300 rounded-lg"><!-- Laporan akan ditampilkan di sini -->
       </div>
      </div>
     </div>
    </div>
   </div>
  </div><!-- Modal Konfirmasi Penerimaan -->
  <div id="modalPenerimaan" class="hidden fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 p-4">
   <div class="bg-white rounded-xl shadow-2xl max-w-md w-full p-6">
    <h3 class="text-xl font-bold text-gray-800 mb-4">Konfirmasi Penerimaan Surat</h3>
    <div class="space-y-4">
     <div><label for="namaPenerima" class="block text-sm font-semibold text-gray-700 mb-2">Nama Penerima *</label> <input type="text" id="namaPenerima" required class="w-full px-4 py-2 border-2 border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500">
     </div>
    </div>
    <div class="flex gap-3 mt-6"><button id="btnKonfirmasiPenerimaan" class="flex-1 bg-green-600 hover:bg-green-700 text-white font-bold py-2 px-4 rounded-lg transition-all"> ✓ Konfirmasi </button> <button id="btnBatalPenerimaan" class="flex-1 bg-gray-300 hover:bg-gray-400 text-gray-700 font-bold py-2 px-4 rounded-lg transition-all"> Batal </button>
    </div>
   </div>
  </div><!-- Modal Penolakan -->
  <div id="modalPenolakan" class="hidden fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 p-4">
   <div class="bg-white rounded-xl shadow-2xl max-w-md w-full p-6">
    <h3 class="text-xl font-bold text-gray-800 mb-4">Tolak Pengiriman Surat</h3>
    <div class="space-y-4">
     <div><label for="alasanPenolakan" class="block text-sm font-semibold text-gray-700 mb-2">Alasan Penolakan *</label> <textarea id="alasanPenolakan" required rows="4" class="w-full px-4 py-2 border-2 border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500" placeholder="Jelaskan alasan penolakan..."></textarea>
     </div>
    </div>
    <div class="flex gap-3 mt-6"><button id="btnKonfirmasiPenolakan" class="flex-1 bg-red-600 hover:bg-red-700 text-white font-bold py-2 px-4 rounded-lg transition-all"> ✓ Tolak Pengiriman </button> <button id="btnBatalPenolakan" class="flex-1 bg-gray-300 hover:bg-gray-400 text-gray-700 font-bold py-2 px-4 rounded-lg transition-all"> Batal </button>
    </div>
   </div>
  </div><!-- Modal Cetak Amplop -->
  <div id="modalAmplop" class="hidden fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 p-4">
   <div class="bg-white rounded-xl shadow-2xl max-w-4xl w-full p-6 max-h-[90vh] overflow-y-auto">
    <h3 class="text-xl font-bold text-gray-800 mb-4">Cetak Amplop Surat</h3>
    <div class="mb-4 p-4 bg-blue-50 border border-blue-200 rounded-lg">
     <p class="text-sm text-blue-800">📏 <strong>Ukuran Amplop:</strong> 230mm x 110mm (Amplop Kantor Standar - Landscape)</p>
     <p class="text-sm text-blue-800 mt-1">💡 Pastikan printer Anda sudah diatur ke ukuran kertas custom atau pilih landscape orientation</p>
    </div><!-- Preview Amplop -->
    <div id="amplopPrintArea" class="border-2 border-gray-300 bg-white" style="width: 230mm; height: 110mm; margin: 0 auto; padding: 15mm; box-sizing: border-box; position: relative;"><!-- Akan diisi oleh JavaScript -->
    </div>
    <div class="flex gap-3 mt-6"><button id="btnCetakAmplop" class="flex-1 bg-purple-600 hover:bg-purple-700 text-white font-bold py-3 px-6 rounded-lg transition-all"> 🖨️ Cetak Amplop </button> <button id="btnTutupAmplop" class="flex-1 bg-gray-300 hover:bg-gray-400 text-gray-700 font-bold py-3 px-6 rounded-lg transition-all"> Tutup </button>
    </div>
   </div>
  </div><!-- Modal Edit Penerima -->
  <div id="modalEditPenerima" class="hidden fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 p-4">
   <div class="bg-white rounded-xl shadow-2xl max-w-md w-full p-6">
    <h3 class="text-xl font-bold text-gray-800 mb-4">Edit Data Penerima</h3>
    <form id="formEditPenerima" class="space-y-4">
     <div><label for="editNamaInstansiPenerima" class="block text-sm font-semibold text-gray-700 mb-2">Nama Instansi *</label> <input type="text" id="editNamaInstansiPenerima" required class="w-full px-4 py-2 border-2 border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500">
     </div>
     <div><label for="editAlamatLengkapPenerima" class="block text-sm font-semibold text-gray-700 mb-2">Alamat Lengkap *</label> <textarea id="editAlamatLengkapPenerima" required rows="4" class="w-full px-4 py-2 border-2 border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500"></textarea>
     </div>
     <div class="flex gap-3 mt-6"><button type="submit" id="btnSimpanEditPenerima" class="flex-1 bg-blue-600 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded-lg transition-all"> ✓ Simpan </button> <button type="button" id="btnBatalEditPenerima" class="flex-1 bg-gray-300 hover:bg-gray-400 text-gray-700 font-bold py-2 px-4 rounded-lg transition-all"> Batal </button>
     </div>
    </form>
   </div>
  </div><!-- Modal Hapus Penerima -->
  <div id="modalHapusPenerima" class="hidden fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 p-4">
   <div class="bg-white rounded-xl shadow-2xl max-w-md w-full p-6">
    <h3 class="text-xl font-bold text-gray-800 mb-4">Konfirmasi Hapus</h3>
    <p class="text-gray-600 mb-6">Apakah Anda yakin ingin menghapus data penerima ini?</p>
    <div class="flex gap-3"><button id="btnKonfirmasiHapusPenerima" class="flex-1 bg-red-600 hover:bg-red-700 text-white font-bold py-2 px-4 rounded-lg transition-all"> Hapus </button> <button id="btnBatalHapusPenerima" class="flex-1 bg-gray-300 hover:bg-gray-400 text-gray-700 font-bold py-2 px-4 rounded-lg transition-all"> Batal </button>
    </div>
   </div>
  </div><!-- Modal Edit -->
  <div id="modalEdit" class="hidden fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 p-4">
   <div class="bg-white rounded-xl shadow-2xl max-w-2xl w-full p-6 max-h-[90vh] overflow-y-auto">
    <h3 class="text-xl font-bold text-gray-800 mb-4">Edit Data Pengiriman</h3>
    <form id="formEdit" class="space-y-4">
     <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
      <div><label for="editNomorSurat" class="block text-sm font-semibold text-gray-700 mb-2">Nomor Surat *</label> <input type="text" id="editNomorSurat" required class="w-full px-4 py-2 border-2 border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500">
      </div>
      <div><label for="editTanggalKirim" class="block text-sm font-semibold text-gray-700 mb-2">Tanggal Kirim *</label> <input type="date" id="editTanggalKirim" required class="w-full px-4 py-2 border-2 border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500">
      </div>
     </div>
     <div><label for="editAlamatTujuan" class="block text-sm font-semibold text-gray-700 mb-2">Alamat Instansi Tujuan *</label> <textarea id="editAlamatTujuan" required rows="3" class="w-full px-4 py-2 border-2 border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500"></textarea>
     </div>
     <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
      <div><label for="editJumlahSurat" class="block text-sm font-semibold text-gray-700 mb-2">Jumlah Surat *</label> <input type="number" id="editJumlahSurat" required min="1" class="w-full px-4 py-2 border-2 border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500">
      </div>
      <div><label for="editNamaPengirim" class="block text-sm font-semibold text-gray-700 mb-2">Nama Pengirim *</label> <input type="text" id="editNamaPengirim" required class="w-full px-4 py-2 border-2 border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500">
      </div>
     </div>
     <div><label for="editKeterangan" class="block text-sm font-semibold text-gray-700 mb-2">Keterangan</label> <textarea id="editKeterangan" rows="2" class="w-full px-4 py-2 border-2 border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500"></textarea>
     </div>
     <div class="flex gap-3 mt-6"><button type="submit" id="btnSimpanEdit" class="flex-1 bg-blue-600 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded-lg transition-all"> ✓ Simpan Perubahan </button> <button type="button" id="btnBatalEdit" class="flex-1 bg-gray-300 hover:bg-gray-400 text-gray-700 font-bold py-2 px-4 rounded-lg transition-all"> Batal </button>
     </div>
    </form>
   </div>
  </div>
  <script>
    const defaultConfig = {
      app_title: "Sistem Kirim Surat",
      instansi_name: "Sistem Manajemen Pengiriman Surat Digital",
      background_color: "#EEF2FF",
      surface_color: "#FFFFFF",
      text_color: "#1F2937",
      primary_action_color: "#4F46E5",
      secondary_action_color: "#6B7280"
    };

    let allData = [];
    let allPenerima = [];
    let currentRecord = null;
    let currentPenerima = null;

    // Generate Nomor Resi
    function generateNomorResi() {
      const timestamp = Date.now();
      const random = Math.floor(Math.random() * 1000).toString().padStart(3, '0');
      return `RSI${timestamp}${random}`;
    }

    // Toast Notification
    function showToast(message, type = 'success') {
      const toast = document.createElement('div');
      toast.className = 'toast';
      toast.style.background = type === 'success' ? '#10b981' : '#ef4444';
      toast.textContent = message;
      document.body.appendChild(toast);
      setTimeout(() => toast.remove(), 3000);
    }

    // Tab Navigation
    const tabs = {
      input: { btn: document.getElementById('tabInput'), content: document.getElementById('contentInput') },
      penerima: { btn: document.getElementById('tabPenerima'), content: document.getElementById('contentPenerima') },
      rekap: { btn: document.getElementById('tabRekap'), content: document.getElementById('contentRekap') },
      laporan: { btn: document.getElementById('tabLaporan'), content: document.getElementById('contentLaporan') }
    };

    function switchTab(activeTab) {
      Object.values(tabs).forEach(tab => {
        tab.btn.classList.remove('tab-active', 'text-indigo-600');
        tab.btn.classList.add('text-gray-600');
        tab.content.classList.add('hidden');
      });
      
      tabs[activeTab].btn.classList.add('tab-active', 'text-indigo-600');
      tabs[activeTab].btn.classList.remove('text-gray-600');
      tabs[activeTab].content.classList.remove('hidden');
    }

    tabs.input.btn.addEventListener('click', () => switchTab('input'));
    tabs.penerima.btn.addEventListener('click', () => switchTab('penerima'));
    tabs.rekap.btn.addEventListener('click', () => switchTab('rekap'));
    tabs.laporan.btn.addEventListener('click', () => switchTab('laporan'));

    // Form Submit Penerima
    document.getElementById('formPenerima').addEventListener('submit', async (e) => {
      e.preventDefault();
      
      const submitBtn = document.getElementById('submitPenerimaBtn');
      const originalText = submitBtn.textContent;
      submitBtn.disabled = true;
      submitBtn.textContent = '⏳ Menyimpan...';

      const newPenerima = {
        type: "penerima",
        nama_instansi_penerima: document.getElementById('namaInstansiPenerima').value,
        alamat_lengkap_penerima: document.getElementById('alamatLengkapPenerima').value,
        nomor_resi: "",
        nomor_surat: "",
        tanggal_kirim: "",
        alamat_tujuan: "",
        jumlah_surat: 0,
        keterangan: "",
        nama_pengirim: "",
        ttd_pengirim: "",
        status: "",
        nama_penerima: "",
        ttd_penerima: "",
        tanggal_terima: "",
        alasan_ditolak: ""
      };

      if (allData.length >= 999) {
        showToast('Batas maksimal 999 data tercapai. Hapus beberapa data terlebih dahulu.', 'error');
        submitBtn.disabled = false;
        submitBtn.textContent = originalText;
        return;
      }

      const result = await window.dataSdk.create(newPenerima);
      
      if (result.isOk) {
        showToast('✓ Data penerima berhasil ditambahkan');
        document.getElementById('formPenerima').reset();
      } else {
        showToast('Gagal menyimpan data. Silakan coba lagi.', 'error');
      }

      submitBtn.disabled = false;
      submitBtn.textContent = originalText;
    });

    // Render Daftar Penerima
    function renderDaftarPenerima(dataToRender = allPenerima) {
      const container = document.getElementById('daftarPenerimaContainer');
      
      if (dataToRender.length === 0) {
        container.innerHTML = '<p class="text-center text-gray-500 py-8">Belum ada data penerima</p>';
        return;
      }

      container.innerHTML = dataToRender.map(item => `
        <div class="border-2 border-gray-200 rounded-lg p-4 hover:border-indigo-400 transition-all">
          <div class="flex justify-between items-start">
            <div class="flex-1">
              <h3 class="font-bold text-gray-800 text-lg mb-1">${item.nama_instansi_penerima}</h3>
              <p class="text-sm text-gray-600 leading-relaxed">${item.alamat_lengkap_penerima}</p>
            </div>
            <div class="flex gap-2 ml-4">
              <button onclick="editPenerimaData('${item.__backendId}')" 
                class="bg-blue-500 hover:bg-blue-600 text-white px-3 py-1 rounded text-xs font-medium transition-all">
                ✏️
              </button>
              <button onclick="hapusPenerimaData('${item.__backendId}')" 
                class="bg-red-500 hover:bg-red-600 text-white px-3 py-1 rounded text-xs font-medium transition-all">
                🗑️
              </button>
            </div>
          </div>
        </div>
      `).join('');
    }

    // Update Dropdown Pilih Penerima
    function updateDropdownPenerima() {
      const select = document.getElementById('pilihPenerima');
      const currentValue = select.value;
      
      select.innerHTML = '<option value="">-- Pilih dari daftar atau isi manual --</option>';
      
      allPenerima.forEach(item => {
        const option = document.createElement('option');
        option.value = item.__backendId;
        option.textContent = item.nama_instansi_penerima;
        select.appendChild(option);
      });
      
      if (currentValue) {
        select.value = currentValue;
      }
    }

    // Pilih Penerima dari Dropdown
    document.getElementById('pilihPenerima').addEventListener('change', (e) => {
      const selectedId = e.target.value;
      if (selectedId) {
        const penerima = allPenerima.find(item => item.__backendId === selectedId);
        if (penerima) {
          document.getElementById('alamatTujuan').value = `${penerima.nama_instansi_penerima}\n${penerima.alamat_lengkap_penerima}`;
        }
      }
    });

    // Search Penerima
    document.getElementById('searchPenerima').addEventListener('input', (e) => {
      const searchTerm = e.target.value.toLowerCase();
      if (searchTerm) {
        const filtered = allPenerima.filter(item => 
          item.nama_instansi_penerima.toLowerCase().includes(searchTerm) ||
          item.alamat_lengkap_penerima.toLowerCase().includes(searchTerm)
        );
        renderDaftarPenerima(filtered);
      } else {
        renderDaftarPenerima();
      }
    });

    // Edit Penerima
    window.editPenerimaData = function(backendId) {
      currentPenerima = allPenerima.find(item => item.__backendId === backendId);
      
      document.getElementById('editNamaInstansiPenerima').value = currentPenerima.nama_instansi_penerima;
      document.getElementById('editAlamatLengkapPenerima').value = currentPenerima.alamat_lengkap_penerima;
      
      document.getElementById('modalEditPenerima').classList.remove('hidden');
    };

    document.getElementById('btnBatalEditPenerima').addEventListener('click', () => {
      document.getElementById('modalEditPenerima').classList.add('hidden');
      currentPenerima = null;
    });

    document.getElementById('formEditPenerima').addEventListener('submit', async (e) => {
      e.preventDefault();
      
      const btn = document.getElementById('btnSimpanEditPenerima');
      const originalText = btn.textContent;
      btn.disabled = true;
      btn.textContent = '⏳ Menyimpan...';

      currentPenerima.nama_instansi_penerima = document.getElementById('editNamaInstansiPenerima').value;
      currentPenerima.alamat_lengkap_penerima = document.getElementById('editAlamatLengkapPenerima').value;

      const result = await window.dataSdk.update(currentPenerima);

      if (result.isOk) {
        showToast('✓ Data penerima berhasil diperbarui');
        document.getElementById('modalEditPenerima').classList.add('hidden');
        currentPenerima = null;
      } else {
        showToast('Gagal memperbarui data. Silakan coba lagi.', 'error');
      }

      btn.disabled = false;
      btn.textContent = originalText;
    });

    // Hapus Penerima
    window.hapusPenerimaData = function(backendId) {
      currentPenerima = allPenerima.find(item => item.__backendId === backendId);
      document.getElementById('modalHapusPenerima').classList.remove('hidden');
    };

    document.getElementById('btnBatalHapusPenerima').addEventListener('click', () => {
      document.getElementById('modalHapusPenerima').classList.add('hidden');
      currentPenerima = null;
    });

    document.getElementById('btnKonfirmasiHapusPenerima').addEventListener('click', async () => {
      const btn = document.getElementById('btnKonfirmasiHapusPenerima');
      const originalText = btn.textContent;
      btn.disabled = true;
      btn.textContent = '⏳ Menghapus...';

      const result = await window.dataSdk.delete(currentPenerima);

      if (result.isOk) {
        showToast('✓ Data penerima berhasil dihapus');
        document.getElementById('modalHapusPenerima').classList.add('hidden');
        currentPenerima = null;
      } else {
        showToast('Gagal menghapus data. Silakan coba lagi.', 'error');
      }

      btn.disabled = false;
      btn.textContent = originalText;
    });

    // Form Submit
    document.getElementById('formPengiriman').addEventListener('submit', async (e) => {
      e.preventDefault();
      
      const submitBtn = document.getElementById('submitBtn');
      const originalText = submitBtn.textContent;
      submitBtn.disabled = true;
      submitBtn.textContent = '⏳ Menyimpan...';

      const nomorResi = generateNomorResi();
      const newData = {
        type: "surat",
        nomor_resi: nomorResi,
        nomor_surat: document.getElementById('nomorSurat').value,
        tanggal_kirim: document.getElementById('tanggalKirim').value,
        alamat_tujuan: document.getElementById('alamatTujuan').value,
        jumlah_surat: parseInt(document.getElementById('jumlahSurat').value),
        keterangan: document.getElementById('keterangan').value,
        nama_pengirim: document.getElementById('namaPengirim').value,
        ttd_pengirim: "",
        status: "Terkirim",
        nama_penerima: "",
        ttd_penerima: "",
        tanggal_terima: "",
        alasan_ditolak: "",
        nama_instansi_penerima: "",
        alamat_lengkap_penerima: ""
      };

      if (allData.length >= 999) {
        showToast('Batas maksimal 999 data tercapai. Hapus beberapa data terlebih dahulu.', 'error');
        submitBtn.disabled = false;
        submitBtn.textContent = originalText;
        return;
      }

      // Kirim ke Google Spreadsheet
      try {
        const spreadsheetUrl = 'https://script.google.com/macros/s/AKfycbzh25RHh6tcCjRrixucsEVx6ugi4IwkxmJsM8-yyFCnByIzaTS3ohooIMgx7Yv7_ckP/exec';
        
        const formData = new FormData();
        formData.append('nomor_resi', newData.nomor_resi);
        formData.append('nomor_surat', newData.nomor_surat);
        formData.append('tanggal_kirim', newData.tanggal_kirim);
        formData.append('alamat_tujuan', newData.alamat_tujuan);
        formData.append('jumlah_surat', newData.jumlah_surat);
        formData.append('keterangan', newData.keterangan);
        formData.append('nama_pengirim', newData.nama_pengirim);
        formData.append('status', newData.status);

        await fetch(spreadsheetUrl, {
          method: 'POST',
          body: formData,
          mode: 'no-cors'
        });
      } catch (error) {
        console.error('Error mengirim ke spreadsheet:', error);
      }

      // Simpan ke Data SDK
      const result = await window.dataSdk.create(newData);
      
      if (result.isOk) {
        showToast(`✓ Berhasil! Nomor Resi: ${newData.nomor_resi}`);
        document.getElementById('formPengiriman').reset();
        document.getElementById('tanggalKirim').valueAsDate = new Date();
      } else {
        showToast('Gagal menyimpan data. Silakan coba lagi.', 'error');
      }

      submitBtn.disabled = false;
      submitBtn.textContent = originalText;
    });

    // Render Tabel Rekap
    function renderTabelRekap(dataToRender = allData) {
      const tbody = document.getElementById('tabelRekapBody');
      
      if (dataToRender.length === 0) {
        tbody.innerHTML = '<tr><td colspan="7" class="text-center py-8 text-gray-500">Belum ada data pengiriman</td></tr>';
        return;
      }

      tbody.innerHTML = dataToRender.map(item => `
        <tr class="hover:bg-gray-50 transition-colors">
          <td class="px-4 py-3 text-sm font-medium text-gray-900">${item.nomor_resi}</td>
          <td class="px-4 py-3 text-sm text-gray-600">${new Date(item.tanggal_kirim).toLocaleDateString('id-ID')}</td>
          <td class="px-4 py-3 text-sm text-gray-900">${item.nomor_surat}</td>
          <td class="px-4 py-3 text-sm text-gray-600 max-w-xs truncate">${item.alamat_tujuan}</td>
          <td class="px-4 py-3 text-sm text-gray-600">${item.jumlah_surat}</td>
          <td class="px-4 py-3 text-sm">
            <span class="px-2 py-1 rounded-full text-xs font-semibold ${
              item.status === 'Diterima' ? 'bg-green-100 text-green-800' : 
              item.status === 'Ditolak' ? 'bg-red-100 text-red-800' :
              'bg-yellow-100 text-yellow-800'
            }">
              ${item.status}
            </span>
          </td>
          <td class="px-4 py-3 text-sm">
            <div class="flex gap-2 flex-wrap">
              ${item.status === 'Terkirim' ? 
                `<button onclick="konfirmasiPenerimaan('${item.__backendId}')" class="bg-green-500 hover:bg-green-600 text-white px-3 py-1 rounded text-xs font-medium transition-all">Terima</button>
                 <button onclick="tolakPengiriman('${item.__backendId}')" class="bg-red-500 hover:bg-red-600 text-white px-3 py-1 rounded text-xs font-medium transition-all">Tolak</button>
                 <button onclick="editPengiriman('${item.__backendId}')" class="bg-blue-500 hover:bg-blue-600 text-white px-3 py-1 rounded text-xs font-medium transition-all">Edit</button>
                 <button onclick="cetakAmplop('${item.__backendId}')" class="bg-purple-500 hover:bg-purple-600 text-white px-3 py-1 rounded text-xs font-medium transition-all">🖨️ Amplop</button>` 
                : item.status === 'Ditolak' ?
                `<button onclick="lihatDetail('${item.__backendId}')" class="bg-gray-500 hover:bg-gray-600 text-white px-3 py-1 rounded text-xs font-medium transition-all">Detail</button>
                 <button onclick="editPengiriman('${item.__backendId}')" class="bg-blue-500 hover:bg-blue-600 text-white px-3 py-1 rounded text-xs font-medium transition-all">Edit</button>
                 <button onclick="cetakAmplop('${item.__backendId}')" class="bg-purple-500 hover:bg-purple-600 text-white px-3 py-1 rounded text-xs font-medium transition-all">🖨️ Amplop</button>`
                :
                `<button onclick="lihatDetail('${item.__backendId}')" class="bg-blue-500 hover:bg-blue-600 text-white px-3 py-1 rounded text-xs font-medium transition-all">Detail</button>
                 <button onclick="editPengiriman('${item.__backendId}')" class="bg-blue-500 hover:bg-blue-600 text-white px-3 py-1 rounded text-xs font-medium transition-all">Edit</button>
                 <button onclick="cetakAmplop('${item.__backendId}')" class="bg-purple-500 hover:bg-purple-600 text-white px-3 py-1 rounded text-xs font-medium transition-all">🖨️ Amplop</button>`
              }
            </div>
          </td>
        </tr>
      `).join('');
    }

    // Filter & Search
    document.getElementById('searchResi').addEventListener('input', applyFilters);
    document.getElementById('filterTanggalMulai').addEventListener('change', applyFilters);
    document.getElementById('filterTanggalAkhir').addEventListener('change', applyFilters);

    function applyFilters() {
      const searchTerm = document.getElementById('searchResi').value.toLowerCase();
      const tanggalMulai = document.getElementById('filterTanggalMulai').value;
      const tanggalAkhir = document.getElementById('filterTanggalAkhir').value;

      let filtered = allData.filter(item => item.type === "surat" || !item.type);

      if (searchTerm) {
        filtered = filtered.filter(item => 
          item.nomor_resi.toLowerCase().includes(searchTerm) || 
          item.nomor_surat.toLowerCase().includes(searchTerm)
        );
      }

      if (tanggalMulai) {
        filtered = filtered.filter(item => item.tanggal_kirim >= tanggalMulai);
      }

      if (tanggalAkhir) {
        filtered = filtered.filter(item => item.tanggal_kirim <= tanggalAkhir);
      }

      renderTabelRekap(filtered);
    }

    // Konfirmasi Penerimaan
    window.konfirmasiPenerimaan = function(backendId) {
      currentRecord = allData.find(item => item.__backendId === backendId);
      document.getElementById('modalPenerimaan').classList.remove('hidden');
      document.getElementById('namaPenerima').value = '';
    };

    document.getElementById('btnBatalPenerimaan').addEventListener('click', () => {
      document.getElementById('modalPenerimaan').classList.add('hidden');
      currentRecord = null;
    });

    document.getElementById('btnKonfirmasiPenerimaan').addEventListener('click', async () => {
      const namaPenerima = document.getElementById('namaPenerima').value.trim();
      
      if (!namaPenerima) {
        showToast('Nama penerima harus diisi', 'error');
        return;
      }
      
      const btn = document.getElementById('btnKonfirmasiPenerimaan');
      const originalText = btn.textContent;
      btn.disabled = true;
      btn.textContent = '⏳ Menyimpan...';

      currentRecord.status = "Diterima";
      currentRecord.nama_penerima = namaPenerima;
      currentRecord.ttd_penerima = "";
      currentRecord.tanggal_terima = new Date().toISOString();

      const result = await window.dataSdk.update(currentRecord);

      if (result.isOk) {
        showToast('✓ Penerimaan berhasil dikonfirmasi');
        document.getElementById('modalPenerimaan').classList.add('hidden');
        currentRecord = null;
      } else {
        showToast('Gagal mengkonfirmasi penerimaan. Silakan coba lagi.', 'error');
      }

      btn.disabled = false;
      btn.textContent = originalText;
    });

    // Tolak Pengiriman
    window.tolakPengiriman = function(backendId) {
      currentRecord = allData.find(item => item.__backendId === backendId);
      document.getElementById('modalPenolakan').classList.remove('hidden');
      document.getElementById('alasanPenolakan').value = '';
    };

    document.getElementById('btnBatalPenolakan').addEventListener('click', () => {
      document.getElementById('modalPenolakan').classList.add('hidden');
      currentRecord = null;
    });

    document.getElementById('btnKonfirmasiPenolakan').addEventListener('click', async () => {
      const alasanPenolakan = document.getElementById('alasanPenolakan').value.trim();
      
      if (!alasanPenolakan) {
        showToast('Alasan penolakan harus diisi', 'error');
        return;
      }
      
      const btn = document.getElementById('btnKonfirmasiPenolakan');
      const originalText = btn.textContent;
      btn.disabled = true;
      btn.textContent = '⏳ Memproses...';

      currentRecord.status = "Ditolak";
      currentRecord.alasan_ditolak = alasanPenolakan;

      const result = await window.dataSdk.update(currentRecord);

      if (result.isOk) {
        showToast('✓ Pengiriman berhasil ditolak');
        document.getElementById('modalPenolakan').classList.add('hidden');
        currentRecord = null;
      } else {
        showToast('Gagal menolak pengiriman. Silakan coba lagi.', 'error');
      }

      btn.disabled = false;
      btn.textContent = originalText;
    });

    // Edit Pengiriman
    window.editPengiriman = function(backendId) {
      currentRecord = allData.find(item => item.__backendId === backendId);
      
      document.getElementById('editNomorSurat').value = currentRecord.nomor_surat;
      document.getElementById('editTanggalKirim').value = currentRecord.tanggal_kirim;
      document.getElementById('editAlamatTujuan').value = currentRecord.alamat_tujuan;
      document.getElementById('editJumlahSurat').value = currentRecord.jumlah_surat;
      document.getElementById('editNamaPengirim').value = currentRecord.nama_pengirim;
      document.getElementById('editKeterangan').value = currentRecord.keterangan;
      
      document.getElementById('modalEdit').classList.remove('hidden');
    };

    document.getElementById('btnBatalEdit').addEventListener('click', () => {
      document.getElementById('modalEdit').classList.add('hidden');
      currentRecord = null;
    });

    document.getElementById('formEdit').addEventListener('submit', async (e) => {
      e.preventDefault();
      
      const btn = document.getElementById('btnSimpanEdit');
      const originalText = btn.textContent;
      btn.disabled = true;
      btn.textContent = '⏳ Menyimpan...';

      currentRecord.nomor_surat = document.getElementById('editNomorSurat').value;
      currentRecord.tanggal_kirim = document.getElementById('editTanggalKirim').value;
      currentRecord.alamat_tujuan = document.getElementById('editAlamatTujuan').value;
      currentRecord.jumlah_surat = parseInt(document.getElementById('editJumlahSurat').value);
      currentRecord.nama_pengirim = document.getElementById('editNamaPengirim').value;
      currentRecord.keterangan = document.getElementById('editKeterangan').value;

      const result = await window.dataSdk.update(currentRecord);

      if (result.isOk) {
        showToast('✓ Data berhasil diperbarui');
        document.getElementById('modalEdit').classList.add('hidden');
        currentRecord = null;
      } else {
        showToast('Gagal memperbarui data. Silakan coba lagi.', 'error');
      }

      btn.disabled = false;
      btn.textContent = originalText;
    });

    // Cetak Amplop
    window.cetakAmplop = function(backendId) {
      const record = allData.find(item => item.__backendId === backendId);
      if (!record) return;

      const config = window.elementSdk ? window.elementSdk.config : defaultConfig;
      const amplopArea = document.getElementById('amplopPrintArea');

      amplopArea.innerHTML = `
        <div style="height: 100%; display: grid; grid-template-columns: 1fr 2fr; gap: 20px;">
          <!-- Kolom Kiri: Kosong atau bisa diisi logo -->
          <div style="display: flex; flex-direction: column; justify-content: flex-start;">
          </div>

          <!-- Kolom Kanan: Alamat Tujuan & Info Surat -->
          <div style="display: flex; flex-direction: column; justify-content: space-between; height: 100%;">
            <!-- Alamat Tujuan (Tengah) -->
            <div style="flex: 1; display: flex; flex-direction: column; justify-content: center;">
              <p style="font-size: 10px; font-weight: 600; color: #374151; margin: 0 0 5px 0;">
                Kepada Yth:
              </p>
              <div style="font-size: 12px; font-weight: 700; color: #111827; line-height: 1.6; white-space: pre-line;">
                ${record.alamat_tujuan}
              </div>
            </div>

            <!-- Info Surat (Bawah Kanan) -->
            <div style="text-align: right;">
              <p style="font-size: 8px; color: #6b7280; margin: 0; line-height: 1.4;">
                No. Surat: ${record.nomor_surat}
              </p>
            </div>
          </div>
        </div>
      `;

      document.getElementById('modalAmplop').classList.remove('hidden');
    };

    document.getElementById('btnTutupAmplop').addEventListener('click', () => {
      document.getElementById('modalAmplop').classList.add('hidden');
    });

    document.getElementById('btnCetakAmplop').addEventListener('click', () => {
      window.print();
    });

    // Lihat Detail
    window.lihatDetail = function(backendId) {
      const record = allData.find(item => item.__backendId === backendId);
      if (!record) return;

      const detail = `
        Nomor Resi: ${record.nomor_resi}
        Nomor Surat: ${record.nomor_surat}
        Tanggal Kirim: ${new Date(record.tanggal_kirim).toLocaleDateString('id-ID')}
        Instansi Tujuan: ${record.alamat_tujuan}
        Jumlah Surat: ${record.jumlah_surat}
        Pengirim: ${record.nama_pengirim}
        Penerima: ${record.nama_penerima || '-'}
        Tanggal Terima: ${record.tanggal_terima ? new Date(record.tanggal_terima).toLocaleDateString('id-ID') : '-'}
        Status: ${record.status}
        ${record.status === 'Ditolak' ? `Alasan Ditolak: ${record.alasan_ditolak || '-'}` : ''}
        Keterangan: ${record.keterangan || '-'}
      `;
      
      const detailDiv = document.createElement('div');
      detailDiv.className = 'toast';
      detailDiv.style.background = '#3b82f6';
      detailDiv.style.whiteSpace = 'pre-line';
      detailDiv.style.maxWidth = '400px';
      detailDiv.textContent = detail;
      document.body.appendChild(detailDiv);
      setTimeout(() => detailDiv.remove(), 5000);
    };

    // Laporan
    document.getElementById('btnPreviewLaporan').addEventListener('click', generateLaporanPreview);
    document.getElementById('btnCetakLaporan').addEventListener('click', () => {
      const tanggalMulai = document.getElementById('laporanTanggalMulai').value;
      const tanggalAkhir = document.getElementById('laporanTanggalAkhir').value;

      let dataLaporan = allData.filter(item => item.type === "surat" || !item.type);

      if (tanggalMulai && tanggalAkhir) {
        dataLaporan = dataLaporan.filter(item => 
          item.tanggal_kirim >= tanggalMulai && item.tanggal_kirim <= tanggalAkhir
        );
      }

      generateLaporanPreview();
      setTimeout(() => window.print(), 500);
    });
    
    document.getElementById('btnUnduhLaporan').addEventListener('click', () => {
      const tanggalMulai = document.getElementById('laporanTanggalMulai').value;
      const tanggalAkhir = document.getElementById('laporanTanggalAkhir').value;

      let dataLaporan = allData.filter(item => item.type === "surat" || !item.type);

      if (tanggalMulai && tanggalAkhir) {
        dataLaporan = dataLaporan.filter(item => 
          item.tanggal_kirim >= tanggalMulai && item.tanggal_kirim <= tanggalAkhir
        );
      }

      // Generate CSV content
      let csvContent = "No,Nomor Resi,Nomor Surat,Jumlah Surat,Alamat Tujuan,Tanggal Kirim,Status,Pengirim,Penerima\n";
      
      dataLaporan.forEach((item, index) => {
        const row = [
          index + 1,
          item.nomor_resi,
          item.nomor_surat,
          item.jumlah_surat,
          `"${item.alamat_tujuan.replace(/"/g, '""')}"`,
          new Date(item.tanggal_kirim).toLocaleDateString('id-ID'),
          item.status,
          item.nama_pengirim,
          item.nama_penerima || '-'
        ].join(',');
        csvContent += row + "\n";
      });

      // Create download link
      const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
      const link = document.createElement('a');
      const url = URL.createObjectURL(blob);
      
      link.setAttribute('href', url);
      link.setAttribute('download', `Laporan_Pengiriman_Surat_${new Date().toISOString().split('T')[0]}.csv`);
      link.style.visibility = 'hidden';
      
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
      
      showToast('✓ Laporan CSV berhasil diunduh!');
    });

    function generateLaporanPreview() {
      const tanggalMulai = document.getElementById('laporanTanggalMulai').value;
      const tanggalAkhir = document.getElementById('laporanTanggalAkhir').value;

      let dataLaporan = allData.filter(item => item.type === "surat" || !item.type);

      if (tanggalMulai && tanggalAkhir) {
        dataLaporan = dataLaporan.filter(item => 
          item.tanggal_kirim >= tanggalMulai && item.tanggal_kirim <= tanggalAkhir
        );
      }

      const printArea = document.getElementById('printArea');
      const config = window.elementSdk ? window.elementSdk.config : defaultConfig;
      
      printArea.innerHTML = `
        <div style="text-align: center; margin-bottom: 30px;">
          <h2 style="font-size: 24px; font-weight: bold; color: #1f2937; margin-bottom: 8px;">${config.instansi_name || defaultConfig.instansi_name}</h2>
          <h3 style="font-size: 18px; font-weight: 600; color: #4b5563;">LAPORAN PENGIRIMAN SURAT</h3>
          <p style="font-size: 14px; color: #6b7280; margin-top: 8px;">
            Periode: ${tanggalMulai ? new Date(tanggalMulai).toLocaleDateString('id-ID') : 'Semua'} - 
            ${tanggalAkhir ? new Date(tanggalAkhir).toLocaleDateString('id-ID') : 'Semua'}
          </p>
        </div>

        <table style="width: 100%; border-collapse: collapse; margin-bottom: 40px;">
          <thead>
            <tr style="background-color: #4f46e5; color: white;">
              <th style="border: 1px solid #cbd5e1; padding: 12px; text-align: left;">No</th>
              <th style="border: 1px solid #cbd5e1; padding: 12px; text-align: left;">Nomor Surat</th>
              <th style="border: 1px solid #cbd5e1; padding: 12px; text-align: left;">Tanggal Pengiriman</th>
              <th style="border: 1px solid #cbd5e1; padding: 12px; text-align: left;">Alamat Pengiriman</th>
            </tr>
          </thead>
          <tbody>
            ${dataLaporan.map((item, index) => `
              <tr>
                <td style="border: 1px solid #cbd5e1; padding: 10px;">${index + 1}</td>
                <td style="border: 1px solid #cbd5e1; padding: 10px;">${item.nomor_surat}</td>
                <td style="border: 1px solid #cbd5e1; padding: 10px;">${new Date(item.tanggal_kirim).toLocaleDateString('id-ID')}</td>
                <td style="border: 1px solid #cbd5e1; padding: 10px;">${item.alamat_tujuan}</td>
              </tr>
            `).join('')}
          </tbody>
        </table>

        <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 40px; margin-top: 60px;">
          <div style="text-align: center;">
            <p style="margin-bottom: 80px; font-weight: 600;">Pengirim</p>
            <p style="border-top: 2px solid #000; display: inline-block; padding-top: 8px; font-weight: 600;">
              ( .............................. )
            </p>
          </div>
          <div style="text-align: center;">
            <p style="margin-bottom: 80px; font-weight: 600;">Penerima</p>
            <p style="border-top: 2px solid #000; display: inline-block; padding-top: 8px; font-weight: 600;">
              ( .............................. )
            </p>
          </div>
        </div>

        <div style="text-align: right; margin-top: 40px; font-size: 12px; color: #6b7280;">
          Dicetak pada: ${new Date().toLocaleDateString('id-ID', { 
            day: 'numeric', 
            month: 'long', 
            year: 'numeric' 
          })}
        </div>
      `;

      document.getElementById('previewArea').classList.remove('hidden');
    }

    // Set tanggal hari ini sebagai default
    document.getElementById('tanggalKirim').valueAsDate = new Date();

    // Data Handler
    const dataHandler = {
      onDataChanged(data) {
        allData = data;
        
        // Pisahkan data surat dan penerima
        const dataSurat = data.filter(item => item.type === "surat" || !item.type);
        allPenerima = data.filter(item => item.type === "penerima");
        
        document.getElementById('totalCount').textContent = dataSurat.length;
        renderTabelRekap(dataSurat);
        renderDaftarPenerima();
        updateDropdownPenerima();
      }
    };

    // Element SDK Implementation
    async function onConfigChange(config) {
      document.getElementById('appTitle').textContent = config.app_title || defaultConfig.app_title;
      document.getElementById('instansiName').textContent = config.instansi_name || defaultConfig.instansi_name;
      
      document.body.style.background = `linear-gradient(to bottom right, ${config.background_color || defaultConfig.background_color}, ${config.primary_action_color || defaultConfig.primary_action_color}33)`;
    }

    window.elementSdk.init({
      defaultConfig,
      onConfigChange,
      mapToCapabilities: (config) => ({
        recolorables: [
          {
            get: () => config.background_color || defaultConfig.background_color,
            set: (value) => {
              config.background_color = value;
              window.elementSdk.setConfig({ background_color: value });
            }
          },
          {
            get: () => config.surface_color || defaultConfig.surface_color,
            set: (value) => {
              config.surface_color = value;
              window.elementSdk.setConfig({ surface_color: value });
            }
          },
          {
            get: () => config.text_color || defaultConfig.text_color,
            set: (value) => {
              config.text_color = value;
              window.elementSdk.setConfig({ text_color: value });
            }
          },
          {
            get: () => config.primary_action_color || defaultConfig.primary_action_color,
            set: (value) => {
              config.primary_action_color = value;
              window.elementSdk.setConfig({ primary_action_color: value });
            }
          },
          {
            get: () => config.secondary_action_color || defaultConfig.secondary_action_color,
            set: (value) => {
              config.secondary_action_color = value;
              window.elementSdk.setConfig({ secondary_action_color: value });
            }
          }
        ],
        borderables: [],
        fontEditable: undefined,
        fontSizeable: undefined
      }),
      mapToEditPanelValues: (config) => new Map([
        ["app_title", config.app_title || defaultConfig.app_title],
        ["instansi_name", config.instansi_name || defaultConfig.instansi_name]
      ])
    });

    // Initialize Data SDK
    (async () => {
      const result = await window.dataSdk.init(dataHandler);
      if (!result.isOk) {
        showToast('Gagal menginisialisasi sistem data', 'error');
      }
    })();
  </script>
 <script>(function(){function c(){var b=a.contentDocument||a.contentWindow.document;if(b){var d=b.createElement('script');d.innerHTML="window.__CF$cv$params={r:'9c0dd3de1306d976',t:'MTc2ODkwNDMwNC4wMDAwMDA='};var a=document.createElement('script');a.nonce='';a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";b.getElementsByTagName('head')[0].appendChild(d)}}if(document.body){var a=document.createElement('iframe');a.height=1;a.width=1;a.style.position='absolute';a.style.top=0;a.style.left=0;a.style.border='none';a.style.visibility='hidden';document.body.appendChild(a);if('loading'!==document.readyState)c();else if(window.addEventListener)document.addEventListener('DOMContentLoaded',c);else{var e=document.onreadystatechange||function(){};document.onreadystatechange=function(b){e(b);'loading'!==document.readyState&&(document.onreadystatechange=e,c())}}}})();</script></body>
</html>
