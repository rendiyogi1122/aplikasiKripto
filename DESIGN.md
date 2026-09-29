# Design: Glassmorphism Portal Kriptografi + Panel Analitik

## Visual Style

**Theme**: Modern Glassmorphism  
**Palette**: Dark slate base with frosted glass cards, cyan accent  
**Mood**: Secure, clean, sophisticated, slightly "Agent" themed

## Key Design Principles

1. **Glassmorphism First**: Cards use `backdrop-blur-xl`, semi-transparent backgrounds, subtle borders  
2. **Depth Hierarchy**: Cards float with soft shadows (0 8px 32px 0 rgba(0,0,0,0.37))  
3. **Cyan Accent**: Primary actions, borders, focus states (Cyan-500/Cyan-400)  
4. **Minimalist Layout**: Single-column content, generous spacing, clear visual flow  
5. **Mobile-Responsive**: Expandable sections on mobile, no sticky elements
6. **Compact Analytics**: Metrik tampil rapat, tidak eats vertical space

## Layout Architecture

```
[Header - Glassy navbar]
    |
[Hero - "Secure Crypto Portal" + 2 main action cards]
    |
[Feature Cards - Enkripsi/Dekripsi modes]
    |
[Detail Panel - Content editor + inputs]
    |
[Result Display - output + copy button]
    |
[CryptoAnalyticsPanel - 3 metric cards + accordion histogram]
    |
[Footer - Simple copyright + status]
```

## Component Specifications

### Header
- `backdrop-blur-xl bg-slate-900/60 border-b border-white/5`
- Logo: "🔒 Portal Kripto" + tagline "AES-GCM · ChaCha20-Poly1305"
- No nav menu - just clean branding

### Hero Section (Menu Mode)
- `backdrop-blur-lg bg-slate-900/70 rounded-2xl p-8 border border-white/10 shadow-2xl`
- Title: "Enkripsi & Dekripsi Aman"
- Subtitle: "Enkripsi teks atau file dengan keamanan tinggi. Gunakan algoritma kriptografi modern."

**2 Main Action Cards (Glassmorphism)**:
```
Card 1 (Enkripsi):
  - Icon: → (Cyan-400)
  - Label: Enkripsi
  - Subtext: "Amankan data Anda"
  - Hover: border-cyan-500/50, scale-102

Card 2 (Dekripsi):
  - Icon: ← (Cyan-400)
  - Label: Dekripsi
  - Subtext: "Buka data terenkripsi"
  - Hover: border-cyan-500/50, scale-102
```

### Detail Panel (Mode aktif)
- `backdrop-blur-xl bg-slate-900/80 rounded-2xl p-6 border border-white/10 shadow-xl`
- Top bar: Back button + title + spacer (flex justify-between items-center)
- Toggle chips (Text/File): `bg-slate-800/60 rounded-lg p-1`
  - Active: `bg-cyan-500 text-white shadow-[0_0_15px_rgba(6,182,212,0.5)]`
  - Inactive: `text-slate-400 hover:text-slate-200`
- Form inputs: `bg-slate-950/60 border border-white/10 rounded-lg focus:border-cyan-500 focus:ring-cyan-500/50`
- Buttons: `bg-cyan-600 hover:bg-cyan-500 text-white rounded-lg shadow-lg shadow-cyan-900/20`

### Result Display
- `bg-slate-950/80 border border-white/5 rounded-lg p-4`
- Copy button: `bg-slate-700/60 hover:bg-slate-600/60 text-sm rounded px-3 py-1.5`
- Text area: `font-mono text-xs text-slate-300 max-h-32 overflow-y-auto`

### File Upload Zone
- `border-2 border-dashed border-white/10 hover:border-cyan-500/30 rounded-xl p-8 text-center transition-colors`
- Dashed border with subtle glow on hover

### CryptoAnalyticsPanel (Baru)
Muncul dengan animasi fade-in tepat di bawah result display, setelah proses kriptografi selesai.

- Grid layout: `grid-cols-3 gap-3` untuk 3 kartu metrik
- Card 1 — **Waktu Komputasi**: angka tebal, misal `42.5 ms`
- Card 2 — **Entropi**: nilai float misal `7.98`; hijau jika `> 7.9`, merah jika di bawahnya
- Card 3 — **Avalanche Effect**: persentase misal `50.2%`; hijau jika `45%–55%`
- Histogram: Bar Chart di dalam Accordion "Lihat Perbandingan Histogram Data"
  - Plaintext: biru/transparan
  - Ciphertext: oranye/merah solid (overlay)
  - Library: recharts atau chart.js
- Loading: skeleton loader selagi menunggu respons FastAPI

## Backend: Metrik Analitik (Python FastAPI)

Endpoint enkripsi dan dekripsi mengembalikan output **plus** objek JSON metrik.

### Waktu Eksekusi
- `time.perf_counter()` sebelum dan sesudah operasi
- Return dalam milidetik (ms)

### Entropy Shannon
- Fungsi matematis untuk mengukur keacakan data ciphertext
- Return float, ideal mendekati `8.0`

### Avalanche Effect (khusus Enkripsi)
**Penting**: bit-flip dilakukan pada **Key**, bukan plaintext.

Mengapa: AES-GCM dan ChaCha20-Poly1305 adalah stream cipher. Flip 1 bit pada plaintext dengan key + IV yang sama hanya mengubah ~1 bit pada ciphertext (hasil 0.00%). Agar valid pada mode ini, modifikasi harus pada Key atau IV.

Alur:
1. **Baseline** — enkripsi dengan plaintext, key asli, IV asli → `ciphertext_A`
2. **Bit-flip pada key** — salinan key ke `modified_key` (bytearray), lalu `modified_key[0] ^= 0x01`
3. **Sekunder** — enkripsi plaintext yang sama dengan `modified_key` + IV asli → `ciphertext_B`
4. **Hamming distance** — iterasi per byte/per bit antara A dan B, hitung bit berbeda
5. **Persentase** — `(Hamming Distance / Total Bit Panjang Ciphertext) * 100`, target ~50%

Catatan kriptografi: `modified_key` **tidak boleh** dipakai untuk penyimpanan atau dekripsi nyata. Isolasi penuh di service analitik, sesaat setelah enkripsi asli selesai.

### Histogram
- Frekuensi kemunculan setiap byte (0–255) dari plaintext dan ciphertext
- Return dua array integer panjang 256
- Dihitung di server agar browser tidak membebani ekstraksi frekuensi byte dari file besar

## Typography

- **Headings**: Inter / Geist Sans (bold, uppercase labels optional)
- **Body**: Inter / Geist Sans
- **Code/Result**: Geist Mono
- **Sizing**: Mobile first, scale up to desktop

## Interactions

1. **Hover**: Subtle scale (1.02), border glow, shadow lift
2. **Focus**: Cyan ring (2px), internal glow
3. **Loading**: Button disable, spinner atau "Memproses..."
4. **Error**: Red accent border + background (500/10)
5. **Success**: Cyan success state, visual feedback
6. **Accordion**: Chart library render hanya saat accordion dibuka

## Mobile Optimization

- Single column stack
- Padding: 1rem on mobile → 2rem desktop
- Buttons full width on mobile
- Inputs full width
- Toggle chips tetap horizontal
- `grid-cols-3` → `grid-cols-1` atau `grid-cols-3` compact di mobile

## Glassmorphism Details

```
Card background: bg-slate-900/70 backdrop-blur-xl
Border: border-white/5 (subtle) → border-cyan-500/50 (active)
Shadow: 0 10px 40px -10px rgba(0,0,0,0.5)
Inner depth: Subtle gradient overlay (top: rgba(255,255,255,0.02), bottom: transparent)
```

## Final Output

File: `frontend/app/page.tsx`  
Backend: `backend/` (FastAPI) + endpoint metrik analitik  
Preview: Browser preview (localhost:3000)  
Detector: Run impeccable.cmd detect --json frontend/app/page.tsx after finish
