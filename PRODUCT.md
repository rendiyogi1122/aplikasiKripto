# Product: Portal Kriptografi Agen Rahasia

- **Target**: Portal kriptografi aman untuk enkripsi/dekripsi teks dan file (AES-GCM dan ChaCha20-Poly1305)
- **Core features**: 
  - Enkripsi teks
  - Dekripsi teks
  - Enkripsi file
  - Dekripsi file
- **Audience**: Pengguna yang membutuhkan alat kriptografi sederhana dan aman
- **Design**: Minimalis, fungsional, tema profesional "Agen Rahasia", fokus pada dark-mode
- **Platform**: Web (Next.js, Tailwind)
- **Mode**: Operate (fokus pada utilitas)

## Requirements from the prompt:
1. Audit landing page dan perbaiki hierarki visual, kontras warna, responsivitas mobile
2. Redesign agar lebih minimalis dan fokus pada fitur utama
3. Test sebelum memberikan ke pengguna (perlu izin menjalankan terminal/browser)
4. Fix bug/error jika ada
5. Update progres.md setelah selesai

## Current State:
- Frontend di `frontend/app/page.tsx` dengan 2 mode utama (Enkripsi/Dekripsi) + toggle Text/File
- Production ready: Deploy ke Vercel & Render dengan API URL dinamis via `.env`
- Glassmorphism UI: backdrop-blur, semi-transparent cards, cyan accent
- Premium input forms: glass-textarea, glass-input, glass-select, char counter, password toggle (Eye/EyeOff)
- Dark mode dengan background Aurora (Three.js shader) + cursor gradient
- Responsif (sm:breakpoint 640px)
- BackgroundGlobal opacity diturunkan untuk readability & accessibility
- CryptoAnalyticsPanel: 3 metric cards (Waktu, Entropi, Avalanche) + Accordion Histogram (recharts)
- History system: Supabase integration dengan FIFO triggers (max 5 rows per table)
- Homepage: Scroll-animated 2x2 grid (framer-motion)
- Build: TypeScript compiles successfully

## Features Completed:

### Crypto Core (Backend)
- AES-GCM & ChaCha20-Poly1305 (AEAD authenticated encryption)
- PBKDF2 key derivation (100k iterations, SHA-256)
- Random salt (16 bytes) + nonce (12 bytes) per encryption
- Crypto metrics calculated server-side:
  - Execution time (ms via `time.perf_counter()`)
  - Shannon Entropy (float, ideal ~8.0)
  - Avalanche Effect (bit-flip on Key, target ~50% Hamming distance)
  - Histogram byte frequency (256-bin arrays for plaintext & ciphertext)

### Frontend UI
- **BackgroundGlobal**: Parallax orbs, cursor-tracking gradient (4 quadrants), pulse ring, noise overlay, vignette, 60fps RAF
- **SkewCard**: 3D skew-to-straight hover, blob gradient border, glass expansion, magnetic cursor, shimmer sweep
- **EncryptionIcons**: Gradient icon backgrounds, glow ring pulse
- **BackButton**: Pill-style, backdrop blur, violet glow ring, chevron slide, pressed scale
- **ChooseFileButton**: 3D layered sheets, folded corner, action badge, FileUp icon lift
- **Input Forms**: `input-wrapper` glassmorphism, violet glow focus, char counter, password toggle, custom select chevron (rotate 180°)
- **CryptoAnalyticsPanel**: Grid 3 metric cards + accordion histogram (BarChart overlay plaintext/ciphertext)

### History & Data Persistence
- Supabase PostgreSQL: `encrypted_texts` & `encrypted_files` tables
- FIFO triggers (`BEFORE INSERT`): auto-delete oldest row when count reaches 5
- RLS disabled for demo (anon key access)
- Client-side password encryption (AES-GCM + PBKDF2) before storing to Supabase
- History hooks with SWR-style mutate
- "Data Tersimpan" menu card + tabbed history view

### Homepage Navigation
- 2x2 card grid: Row 1 visible on load (Encrypt, Decrypt), Row 2 hidden initially
- Scroll reveal (>100px): Row 2 fades in + slides up (Visualize, Data Tersimpan)
- framer-motion staggered animation (0.1s delay)

## Technical Debt / Ponytails:
- `passwordEncryption.ts`: Hardcoded key - **use env var for production**
- No HTTPS / rate limiting / audit logging (explicitly out of scope for localhost tool)
- No unit tests yet

## Next Steps (Priority):
1. Verifikasi visual consistency dashboard dengan background low-opacity
2. Testing end-to-end flow enkripsi/dekripsi dengan panel metrics aktif
3. Polish animasi transisi panel metrics
4. Consider password strength meter
5. Consider drag & drop file upload

## Run Instructions:
```bash
# Terminal 1 - Backend
cd backend
uvicorn main:app --reload --port 8000

# Terminal 2 - Frontend
cd frontend
npm run dev
```
- Backend: http://localhost:8000 (API docs: /docs)
- Frontend: http://localhost:3000