# Ponsel Bebas Distraksi (PWA + Firebase Realtime Database)

## Struktur
- `js/sensor.js` – Accelerometer, Proximity (jika ada), State Machine (SETUP → ON_DESK ⇄ LIFTED)
- `js/student.js` – HP 1: sensor → payload → Firebase
- `js/firebase-client.js` – koneksi cloud (WebSocket) + anonymous auth + sinkron jam server
- `js/supervisor.js` – HP 2: listener realtime, kartu status, alarm, notifikasi
- `js/storage.js` – IndexedDB untuk riwayat lokal

## Setup
1. https://console.firebase.google.com → buat project → **Build → Realtime Database** (Create database).
2. **Build → Authentication → Sign-in method → Anonymous → Enable**.
3. **Project settings → Your apps → Web (</>)** → salin config ke `js/config.js`.
4. Terapkan rules: `database.rules.json` (tempel di tab Rules, atau `firebase deploy --only database`).
5. Deploy HTTPS (WAJIB agar sensor & service worker jalan):
   ```
   npm i -g firebase-tools && firebase login && firebase init hosting   # public dir = .
   firebase deploy
   ```
   Uji lokal: `npx serve .` lalu `chrome://inspect/#devices` → port forwarding (localhost dianggap secure context).
6. Buka URL di kedua HP (Chrome Android) → "Add to Home screen" (opsional).

## Alur pakai
- HP 2: pilih **Pengawas**, kode sesi, tekan **Aktifkan Notifikasi & Alarm** (wajib gestur user).
- HP 1: pilih **Mahasiswa**, nama + kode sesi sama, tekan **Mulai**, lalu taruh telungkup.
- Payload: `{deviceId, name, ts, event, status, duration, totalDuration, liftCount, liftStartedAt, proximity, mode}`

## Izin & batasan penting
- Sensor gerak butuh **HTTPS**. iOS 13+ memunculkan dialog izin; Android Chrome tanpa dialog.
- **Proximity**: Chrome Android tidak lagi membuka sensor ini ke web. Aplikasi otomatis memakai
  mode `accel-only` (Z ≈ −9.8 & HP datar). Untuk proximity asli gunakan Flutter (`proximity_sensor`) / React Native.
- Layar HP 1 harus menyala & tab di foreground (Wake Lock dipakai); browser menghentikan sensor saat tab di-background.
- Notifikasi/alarm HP 2 saat layar mati tidak andal pada PWA (tanpa FCM push); biarkan layar & tab aktif.
- Tidak perlu satu Wi-Fi/subnet karena lewat cloud. Untuk LAN-only gunakan Mosquitto (WebSocket) + `mqtt.js`.
- Ambang kalibrasi ada di `T` pada `js/sensor.js`.
