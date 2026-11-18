# OpenSteth

Offene Web-App (MIT) für das **Hanhong POPULAR-3** Digitalstethoskop.
Live mithören, Wellenform sehen, Herz-/Lungengeräusche als WAV aufnehmen. Ohne Cloud, ohne Konto.
Alle Aufnahmen bleiben im Browser des Handys.

> Kein Medizinprodukt. Nur zur Aufnahme und Wiedergabe, nicht zur Diagnose.

## Nutzung

1. **Alle Dateien** dieses Ordners über **https** bereitstellen, z. B. mit GitHub Pages (siehe unten).
2. Auf Android in **Chrome** öffnen → „Verbinden“ → das Stethoskop auswählen.
3. „Live starten“ zum Mithören (Kopfhörer ans Handy), „Aufnehmen“ für eine WAV-Datei.
   Die Aufnahme-Taste am Stethoskop startet und stoppt die Aufnahme ebenfalls.
4. Als App installieren: Button „Als App installieren“ in der App, oder Chrome-Menü (⋮) → „App installieren“.
   Danach startet OpenSteth wie eine normale App mit eigenem Icon, ohne Browserleiste, und funktioniert auch offline.

Web Bluetooth funktioniert nur in Chrome/Edge (Android, Windows, macOS, ChromeOS), nicht auf dem iPhone.

### GitHub Pages in 3 Minuten

1. Auf github.com ein neues öffentliches Repository anlegen, z. B. `openstethe`.
2. „Add file → Upload files“ → alle Dateien aus der ZIP hochladen (nicht den Ordner selbst) → Commit.
3. Settings → Pages → Source: „Deploy from a branch“, Branch `main`, Ordner `/ (root)` → Save.
4. Nach ca. 1 Minute ist die App unter `https://<benutzername>.github.io/openstethe/` erreichbar.

## Protokoll (aus BLE-Mitschnitt, Firmware V3.1.5)

Das Gerät nutzt den **Nordic UART Service**:

| UUID | Richtung | Zweck |
|---|---|---|
| `6e400001-b5a3-f393-e0a9-e50e24dcca9e` | – | Service |
| `6e400002-…` | App → Gerät | Befehle, Write Without Response |
| `6e400003-…` | Gerät → App | Antworten und Audio, Notify |

Befehle sind ASCII mit `\r` am Ende, Antworten enden mit `\r\n`:

| Befehl | Antwort (Beispiel) |
|---|---|
| `testdev` | `ready` |
| `model` | `POPULAR-3` |
| `uniqueid` | 26-stellige Hex-ID |
| `battery` | `voltage 2.40v` |
| `version` | `hardware V3, bootloader V3.1, firmware V3.1.5` |
| `userlogin 0000` | `User authentication OK` |
| `AutoOffTime?`, `DefaultVolume?`, `ModeSeq?`, `AdvStartOn?` | Einstellungen abfragen |
| `WirelessEarphoneEnable?`, `WirelessEarphoneAddress?` | Funkkopfhörer-Einstellungen |
| `RealtimeRecordExt` | startet den Audio-Strom (Antwort `SYNC`) |
| `RealTimeStop` | stoppt den Audio-Strom |

**Audio-Strom:** wiederholt `PCM_SYNC` (8 Bytes) + 400 Bytes = 200 Samples **int16 little-endian, mono, ca. 11025 Hz**
(gemessen ≈ 22,6 kB/s). Ereignisse vom Gerät kommen in-band direkt nach einem Sync als `PCM_CMDF<Text>\r`,
z. B. `LungMode`, `RecordStart`, `RecordStop` (Tasten am Gerät).

Die Original-App fragt im Leerlauf etwa jede Sekunde `battery` und `testdev` ab. Das Gerät schaltet sich nach `AutoOffTime` (300 s) ab.

## Offene Punkte

- Die genaue Abtastrate ist aus dem Datendurchsatz geschätzt (11025 Hz). Falls Aufnahmen zu hoch oder zu tief klingen, `SAMPLE_RATE` anpassen.
- Weitere Modus-Namen (Herz/Glocke/Membran) sind noch nicht mitgeschnitten. Unbekannte Modi werden roh angezeigt.
