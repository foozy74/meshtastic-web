# Meshtastic Web Gateway via Raspberry Pi & BLE

Verbindet einen **RAK4631 Meshtastic Node** kabellos über **Bluetooth Low Energy (BLE)** mit einem **Raspberry Pi**, der ein Web-Dashboard (**MeshMonitor**) im lokalen Netzwerk (LAN/WLAN) bereitstellt.

```
+----------------+          BLE          +---------------------------+          LAN/WLAN         +--------------------+
| RAK4631 (Node) |  <----------------->  |       Raspberry Pi        |  <--------------------->  | Laptop / Browser   |
| (nRF52840)     |   (nur Bluetooth)     | - BLE Bridge (Port 4403)  |      (HTTP Port 8080)     | http://<pi-ip>:8080|
+----------------+                       | - MeshMonitor (Web-UI)    |                           +--------------------+
                                         +---------------------------+
```

---

## 1. Voraussetzungen auf dem Raspberry Pi

Auf dem Raspberry Pi muss der Linux Bluetooth-Stack (`BlueZ`) und Docker laufen:

```bash
sudo apt update
sudo apt install -y bluez bluetooth docker.io docker-compose-plugin
sudo systemctl enable --now bluetooth
```

---

## 2. RAK4631 über Bluetooth koppeln & MAC-Adresse ermitteln

> **Wichtig:** Ein Meshtastic-Node erlaubt standardmäßig nur **eine aktive BLE-Verbindung**. Stellen Sie sicher, dass Ihr Smartphone aktuell **nicht** mit dem RAK4631 gekoppelt/verbunden ist!

Auf dem Raspberry Pi im Terminal:

```bash
bluetoothctl
```

In der `bluetoothctl`-Konsole:
```text
power on
agent on
default-agent
scan on
```
Warten Sie, bis der RAK4631 angezeigt wird (z. B. `Meshtastic_xxxx` mit einer MAC wie `D0:9D:0C:3A:3D:E4`).

Danach die Kopplung durchführen:
```text
scan off
pair D0:9D:0C:3A:3D:E4
trust D0:9D:0C:3A:3D:E4
quit
```

---

## 3. Konfiguration anpassen

Die ermittelte MAC-Adresse in die Datei `.env` eintragen:

```bash
nano .env
```
Beispiel:
```ini
BLE_ADDRESS=D0:9D:0C:3A:3D:E4
WEB_PORT=8080
```

---

## 4. Container starten

Im Verzeichnis des Projekts ausführen:

```bash
docker compose up -d
```

Logs prüfen:
```bash
docker compose logs -f
```
Sobald `meshtastic-ble-bridge` meldet, dass die BLE-Verbindung steht (`Connected to BLE device`), verbindet sich `meshmonitor` automatisch.

---

## 5. Zugriff vom Laptop / Smartphone

Öffnen Sie von jedem beliebigen Gerät im gleichen Netzwerk (WLAN oder LAN) den Browser:

```text
http://<IP-DES-RASPBERRY-PI>:8080
```

Im MeshMonitor-Dashboard können Sie nun:
- Nachrichten im Mesh senden und empfangen
- Die Knotenliste und Topologie einsehen
- Telemetriedaten und GPS-Positionen auf der Karte verfolgen
- Die Node-Konfiguration anpassen
