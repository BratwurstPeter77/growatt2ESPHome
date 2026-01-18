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

text
esphome:
  name: wechselrichter-growatt
  friendly_name: Wechselrichter Growatt

esp8266:
  board: d1_mini

logger:
  baud_rate: 0
  level: INFO

api:
  encryption:
    key: "RIW6JIOznq0PurptfF8g2HKVcW8HSz5Hi40bhbHesRE="

ota:
  - platform: esphome
    password: "f2a982c2101302191f17d7d34e5e4b16"

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  min_auth_mode: WPA2

  ap:
    ssid: "Wechselrichter-Growatt"
    password: "RJ1ZNqe6DxqV"

captive_portal:

uart:
  id: mod_uart
  tx_pin: GPIO12   # D6
  rx_pin: GPIO14   # D5
  baud_rate: 9600
  stop_bits: 1

modbus:
  id: modbus1
  uart_id: mod_uart

modbus_controller:
  - id: growatt_wr1
    address: 0x01
    modbus_id: modbus1
    update_interval: 10s

  - id: growatt_wr2
    address: 0x02
    modbus_id: modbus1
    update_interval: 10s

sensor:
  # WR1
  - platform: modbus_controller
    modbus_controller_id: growatt_wr1
    name: "WR1 PV1 Power"
    id: wr1_pv1_power
    address: 5
    register_type: read
    value_type: U_DWORD
    unit_of_measurement: "W"
    device_class: power
    state_class: measurement
    accuracy_decimals: 1
    filters:
      - multiply: 0.1

  - platform: modbus_controller
    modbus_controller_id: growatt_wr1
    name: "WR1 PV1 Energy Today"
    address: 59
    register_type: read
    value_type: U_DWORD
    unit_of_measurement: "kWh"
    device_class: energy
    state_class: total_increasing
    accuracy_decimals: 1
    filters:
      - multiply: 0.1

  - platform: modbus_controller
    modbus_controller_id: growatt_wr1
    name: "WR1 PV1 Energy Total"
    address: 61
    register_type: read
    value_type: U_DWORD
    unit_of_measurement: "kWh"
    device_class: energy
    state_class: total_increasing
    accuracy_decimals: 1
    filters:
      - multiply: 0.1

  - platform: modbus_controller
    modbus_controller_id: growatt_wr1
    name: "WR1 IPM Temperature"
    address: 94
    register_type: read
    value_type: U_WORD
    unit_of_measurement: "°C"
    device_class: temperature
    state_class: measurement
    accuracy_decimals: 1
    filters:
      - multiply: 0.1

  # WR2
  - platform: modbus_controller
    modbus_controller_id: growatt_wr2
    name: "WR2 PV1 Power"
    id: wr2_pv1_power
    address: 5
    register_type: read
    value_type: U_DWORD
    unit_of_measurement: "W"
    device_class: power
    state_class: measurement
    accuracy_decimals: 1
    filters:
      - multiply: 0.1

  - platform: modbus_controller
    modbus_controller_id: growatt_wr2
    name: "WR2 PV1 Energy Today"
    address: 59
    register_type: read
    value_type: U_DWORD
    unit_of_measurement: "kWh"
    device_class: energy
    state_class: total_increasing
    accuracy_decimals: 1
    filters:
      - multiply: 0.1

  - platform: modbus_controller
    modbus_controller_id: growatt_wr2
    name: "WR2 PV1 Energy Total"
    address: 61
    register_type: read
    value_type: U_DWORD
    unit_of_measurement: "kWh"
    device_class: energy
    state_class: total_increasing
    accuracy_decimals: 1
    filters:
      - multiply: 0.1

  - platform: modbus_controller
    modbus_controller_id: growatt_wr2
    name: "WR2 IPM Temperature"
    address: 94
    register_type: read
    value_type: U_WORD
    unit_of_measurement: "°C"
    device_class: temperature
    state_class: measurement
    accuracy_decimals: 1
    filters:
      - multiply: 0.1

  # Summe PV1 Power
  - platform: template
    name: "PV1 Power Total"
    unit_of_measurement: "W"
    device_class: power
    state_class: measurement
    accuracy_decimals: 1
    lambda: |-
      return id(wr1_pv1_power).state + id(wr2_pv1_power).state;
    update_interval: 10s

  # Diagnose
  - platform: wifi_signal
    name: "WiFi Signal"
    update_interval: 60s

  - platform: uptime
    name: "Uptime"
    update_interval: 60s
Setup
YAML in ESPHome anlegen/öffnen.

secrets.yaml pflegen:

text
wifi_ssid: "DEIN_WLAN"
wifi_password: "DEIN_PASSWORT"
Install/Flash:

Erstes Flashen per USB (oder OTA, wenn schon möglich)

Danach OTA über ESPHome Dashboard

Troubleshooting
Keine Werte / timeouts: RS485 A/B vertauscht oder falsche Slave-ID.

Werte springen / instabil: update_interval erhöhen (z.B. 15s) und logger.baud_rate: 0 beibehalten.

.local nicht erreichbar: per IP flashen (mDNS abhängig vom Netz).

Credits
Danke an nygma2004 und das Projekt growatt2mqtt für die Vorarbeit/Grundlage rund um Growatt Modbus Register und MIC TL‑X Geräte.
