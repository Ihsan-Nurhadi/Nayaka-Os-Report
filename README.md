# Nayaka OS — Export untuk Pengembangan Lanjutan

Export ini berisi kode halaman (HTML/CSS/JS) dan isi database saat ini dari tools
**Nayaka OS**, supaya tim developer internal Nayaka bisa melanjutkan pengembangannya
di luar Claude.

## Isi folder

```
nayaka-os-export/
├── html/
│   └── nayaka-os.html      <- seluruh kode tampilan & logic (single file)
└── database/
    ├── team.json           <- 11 anggota tim yang sudah diisi (nama + divisi)
    ├── leads.json          <- kosong (belum ada data)
    ├── projects.json       <- kosong
    ├── pos.json            <- kosong (Purchase Order)
    ├── payments.json       <- kosong (permintaan pembayaran)
    ├── rab.json            <- kosong (Rencana Anggaran Biaya)
    ├── activity.json       <- kosong (log aktivitas)
    └── settings_target.json<- null (target PO tahunan belum diisi)
```

## PENTING: bagian yang TIDAK ikut ter-export

`nayaka-os.html` saat ini berjalan di atas platform Claude (claude.ai), yang
menyediakan dua hal secara otomatis lewat `window.claude.use(...)` di dalam kode:

1. **Database bersama** (`db` capability) — tempat leads, proyek, PO, RAB, dan
   pembayaran disimpan & disinkronkan real-time ke semua orang yang membuka
   halamannya.
2. **Identitas pengguna** (`user` capability) — dipakai untuk mengecek izin tulis.

Kedua hal ini **hanya aktif saat halaman dibuka lewat claude.ai**. Jika
`nayaka-os.html` ini dibuka langsung sebagai file (`file://...`) atau di-hosting
di server lain apa adanya, bagian database tidak akan berfungsi — halaman akan
menampilkan pesan "Penyimpanan tidak tersedia" karena `window.claude` tidak ada.

**Supaya tools ini bisa dikembangkan sebagai aplikasi mandiri, bagian yang perlu
dibangun ulang oleh tim developer adalah:**

- Backend/API sendiri (mis. Node.js + Express, atau Supabase/Firebase) untuk
  menggantikan fungsi `db.collection(...).add/update/delete/onSnapshot(...)`
- Sistem login sungguhan (saat ini peran dipilih manual oleh user sendiri lewat
  dropdown, tersimpan di `localStorage` browser — bukan autentikasi akun yang
  terkunci)
- Endpoint untuk memuat ulang `database/*.json` di atas sebagai data awal

Cari kata `window.claude` di dalam `nayaka-os.html` untuk menemukan semua titik
yang perlu diganti dengan pemanggilan API sendiri.

## Struktur data (skema setiap collection)

**team** — direktori anggota tim
```
{ id, name, role: "sales"|"project"|"procurement"|"finance"|"bod", createdAt }
```

**leads** — pipeline sales
```
{ id, customer, sector, pic, solution, value, stage: "lead_in"|"qualified"|"proposal"|"negotiation"|"won"|"lost",
  notes, closedAt, createdAt, updatedAt }
```

**projects** — proyek hasil deal yang Menang
```
{ id, leadId, name, customer, contractValue, budget, approvedRabId, pm,
  startDate, targetEndDate, status: "planning"|"ongoing"|"delayed"|"completed",
  progressPercent, notes, createdAt, updatedAt }
```

**rab** — pengajuan Rencana Anggaran Biaya per proyek
```
{ id, projectId, items: [{category, description, amount}], totalAmount,
  status: "diajukan"|"disetujui"|"ditolak", submittedBy, decidedBy, decidedAt,
  notes, createdAt }
```

**pos** — Purchase Order ke subkon/supplier
```
{ id, projectId, vendorName, vendorType: "subkon"|"supplier", description,
  amount, dueDate, status: "diajukan"|"disetujui"|"dipesan"|"diterima"|"ditolak",
  requestedBy, approvedBy, createdAt, updatedAt }
```

**payments** — permintaan pembayaran
```
{ id, projectId, poId (opsional), category, amount, notes,
  status: "pending"|"approved"|"rejected"|"paid",
  requestedBy, decidedBy, decidedAt, paidAt, createdAt }
```

**activity** — log audit lintas modul
```
{ id, entityType, entityId, action, detail, by, role, at }
```

**settings/target** — target PO tahunan (dokumen tunggal, bukan collection)
```
{ value, month, updatedAt, updatedBy }
```

## Logika kontrol anggaran yang perlu dipertahankan

Beberapa aturan bisnis penting ada di dalam `nayaka-os.html` dan sebaiknya
dibawa utuh ke backend baru, bukan dibangun ulang dari nol:

- PO baru hanya bisa dibuat untuk proyek yang sudah punya RAB **disetujui**
  (`project.budget > 0`)
- Fungsi `projectRemainingBudget()` — sisa RAB = budget disetujui dikurangi
  PO yang sudah disetujui ke atas, dikurangi biaya non-PO yang sudah
  disetujui/dibayar
- Fungsi `poRemaining()` — mencegah pembayaran ganda terhadap PO yang sama
  dengan menghitung sisa saldo PO dikurangi pembayaran yang sudah
  disetujui/dibayar untuk PO tersebut
- Saat Finance menyetujui pembayaran (`decidePayment`), sistem mengecek ulang
  sisa anggaran saat itu juga (bukan hanya saat pengajuan) untuk mencegah dua
  pengajuan yang masing-masing terlihat aman tapi jika disetujui berdua jadi
  melebihi budget

## Cara pakai file ini sekarang

Untuk sekadar melihat tampilannya tanpa backend, `html/nayaka-os.html` tetap
bisa dibuka di browser — tampilannya utuh, hanya saja data tidak akan
tersimpan (hilang saat halaman di-refresh) karena belum ada database yang
terhubung.
# Nayaka-Os-Report
