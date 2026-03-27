# dB Guard — Lautstärkewächter

Eine mobile-optimierte Web-App zur Echtzeit-Überwachung der Umgebungslautstärke mit konfigurierbaren Alarmschwellenwerten.

## Features

### Live-Monitoring
- **Echtzeit-dB-Messung** über das Geräte-Mikrofon (Web Audio API)
- **Kreisförmiges Gauge** (SVG-Ring) mit farbcodierter Anzeige:
  - Grün < 50 dB, Gelb < 70 dB, Orange < 85 dB, Rot >= 85 dB
- **Geglättete dB-Anzeige** (EMA-Smoothing, alpha=0.1) — die große Zahl bewegt sich ruhig und lesbar, der Ring reagiert in Echtzeit
- **Exakter Wert** in kleiner Schrift (0.1 dB Auflösung) unter der Hauptanzeige
- **30-Segment-Bargraph** für die letzten Frames
- **Peak-Tracking** mit dauerhafter Anzeige des Spitzenwerts

### Lautstärkenverlauf
- **10-Minuten-Chart** (SVG-Polyline) mit Sekunden-Auflösung
- Y-Achse 0–120 dB, Zeitachse mit 2-Minuten-Markierungen
- **Schwellenwert-Linien** als gestrichelte Overlays im Chart
- Rolling Buffer (max. 600 Datenpunkte = 10 Minuten)

### Konfigurierbare Schwellenwerte
- Beliebig viele Schwellenwerte (30–130 dB) mit Name
- **6 Alarmtöne**: Beep, Alarm, Glocke, Buzzer, Horn, Sirene
- **Vibration** pro Schwellenwert ein-/ausschaltbar
- **Visueller Alarm**: Roter Vollbild-Flash + pulsierende Kartenränder
- **Cooldown**: 2 Sekunden zwischen Alarmen
- Schwellenwerte werden in `localStorage` persistiert

### Auto-Kalibrierung
Dreiphasiger Kalibrierungsflow mit UX-optimiertem Ablauf:

1. **Countdown (5 Sek.)**: Große Countdown-Zahl, Aufforderung die Geräte-Lautstärke auf Maximum zu stellen
2. **Messung (~3 Sek.)**: White-Noise-Prüfsignal wird abgespielt, Mikrofon misst den Pegel. Outlier-Bereinigung (obere/untere 10%)
3. **Bestätigung**: "War die Geräte-Lautstärke auf Maximum?" — Ja speichert den Offset, Nein zeigt eine Warnung mit Retry-Option

Referenzpegel: 80 dB. Kalibrierungs-Offset wird in `localStorage` gespeichert.

Tab-Wechsel ist während der Kalibrierung gesperrt.

### Hintergrund-Betrieb
- **Screen Wake Lock API** — Display bleibt an während der Überwachung
- **Silent Keepalive-Oszillator** — verhindert `AudioContext`-Suspension im Hintergrund
- **Visibility-Change-Handler** — reaktiviert Wake Lock und AudioContext bei Tab-Rückkehr

## Technik

| Aspekt | Detail |
|---|---|
| **Architektur** | Single-file HTML-App (~540 Zeilen), kein Build-Step |
| **Audio** | Web Audio API, `AnalyserNode` (FFT 2048, smoothing 0.3) |
| **Mikrofon** | `getUserMedia` ohne echoCancellation/noiseSuppression/autoGainControl |
| **Rendering** | Hybrid: Full-Render bei User-Aktionen, DOM-Patching (`getElementById`) im 60-FPS-Loop |
| **Persistenz** | `localStorage` für Schwellenwerte + Kalibrierungs-Offset |
| **Fonts** | JetBrains Mono (Zahlen), Outfit (UI-Text) via Google Fonts |
| **Design** | Dark-Mode-Only, CSS Custom Properties, mobile-first (max-width 480px) |

## Verwendung

1. `index.html` im Browser öffnen (oder auf einem Webserver hosten)
2. Mikrofon-Berechtigung erteilen
3. **Optional**: Auto-Kalibrierung durchführen (Geräte-Lautstärke auf Maximum!)
4. Schwellenwerte nach Bedarf anpassen
5. Überwachung starten

> **Hinweis**: Für HTTPS ist ein Webserver erforderlich (Mikrofon-API). Lokal funktioniert `localhost` oder `file://`.

## Browser-Kompatibilität

Getestet für moderne Chromium-Browser (Chrome, Edge) und Safari auf iOS/macOS. Benötigt:
- Web Audio API
- `getUserMedia`
- Screen Wake Lock API (optional, Fallback ohne)
- Vibration API (optional, nur auf Mobilgeräten)

## Dateistruktur

```
index.html   — Gesamte App (HTML + CSS + JS)
README.md    — Diese Dokumentation
```

## Commit-Historie

| Commit | Beschreibung |
|---|---|
| `c900d2a` | Initiale Version der dB Guard Web-App |
| `75d244b` | Vibration-Fix, 10-Min-Verlaufschart, Background-Keepalive |
| `ef44201` | Manuelle Kalibrierung entfernt, Auto-Kalibrierung mit White Noise |
| `6d457a8` | Referenzpegel auf 80 dB, lauteres Prüfsignal |
| `c6d8c18` | Kalibrierungs-Countdown + Bestätigung, EMA-geglättete dB-Anzeige |

## Lizenz

Privates Projekt.
