Growatt MIC (1500/3300) → ESPHome (Modbus RTU, RS485)
Kompakte ESPHome-Konfiguration für zwei Growatt MIC Wechselrichter (z.B. MIC 1500 TL‑X und MIC 3300 TL‑X) über RS485/Modbus RTU mit einem ESP8266 (Wemos D1 mini). Ausgelesen werden PV1-Leistung, PV1-Energie (heute/gesamt) sowie nur die IPM‑Temperatur (geteilt durch 10).

Danke an nygma2004 für das Projekt/Know-how und die Register-Recherche, die als Grundlage für dieses Setup dient.

Features
2 Wechselrichter (Modbus Slave ID 1 und ID 2) an einem RS485-Bus

PV1 Power je WR + Summe (PV1 Power Total)

PV1 Energy Today / Total je WR

IPM Temperature je WR (Registerwert /10)

WiFi Signal + Uptime als Diagnose

Keine MQTT-Pflicht (läuft über Home Assistant API)

Hardware
ESP8266: Wemos D1 mini

RS485 TTL Adapter (MAX485 / SP3485 / etc.)

Verbindung RS485 ↔ Wechselrichter (A/B an SYS COM)

Verdrahtung (wie im Setup genutzt)

Wemos D1 mini	GPIO	RS485 Modul
D6	GPIO12	DI (TX vom ESP zum RS485)
D5	GPIO14	RO (RX zum ESP vom RS485)
5V	—	VCC
G	—	GND
Hinweis: TX/RX-Bezeichnungen am RS485-Modul sind je nach Modul aufgedruckt (DI/RO). Bei vertauschten Leitungen kommen keine/komische Werte.

Register (Growatt Input Registers)
Dieses Setup nutzt folgende Input-Register (jeweils 16‑bit Words, teilweise als 32‑bit kombiniert):

PV1 Power: 5/6 (0.1 W) → als U_DWORD ab Adresse 5

PV1 Energy Today: 59/60 (0.1 kWh) → als U_DWORD ab Adresse 59

PV1 Energy Total: 61/62 (0.1 kWh) → als U_DWORD ab Adresse 61

IPM Temperature: 94 (0.1 °C) → multiply: 0.1

ESPHome YAML
Datei: wechselrichter-growatt.yaml

Hinweis: logger.baud_rate: 0 ist wichtig, damit serielle Logs nicht den Modbus-UART stören.

Erstes Flashen per USB (oder OTA, wenn schon möglich)

Danach OTA über ESPHome Dashboard

Troubleshooting
Keine Werte / timeouts: RS485 A/B vertauscht oder falsche Slave-ID.

Werte springen / instabil: update_interval erhöhen (z.B. 15s) und logger.baud_rate: 0 beibehalten.

.local nicht erreichbar: per IP flashen (mDNS abhängig vom Netz).

Credits
Danke an nygma2004 und das Projekt growatt2mqtt für die Vorarbeit/Grundlage rund um Growatt Modbus Register und MIC TL‑X Geräte.
