# DEC3852 OPNsense – Konfigurationsdokumentation

**Datum:** 09.04.2026  
**Durchgeführt von:** Inan Bogisch  
**Hardware:** Deciso DEC3852 (OPNsense Rack Security Appliance)  
**Zweck:** Ablösung des Cisco RV345P (192.168.80.1) als Hauptrouter in Hamburg  
**Status:** 🔧 Grundkonfiguration in Arbeit (offline)

---

## Netzwerk-Übersicht (aktuell – Cisco RV345P)

```
Internet
    ↓ (PPPoE)
Cisco RV345P (192.168.80.1) – WAN2 aktiv, WAN1 down
    ↓ 16 LAN-Ports mit VLAN-Zuordnung
    → Dosen im Haus (P&M + andere Unternehmen)
```

### Interfaces DEC3852

| Interface | Funktion |
|-----------|----------|
| igc0 | LAN (Trunk zum Cisco-Switch) |
| igc1 | WAN |
| igc2 | frei |
| igc3 | frei |
| ax0 | frei |
| ax1 | frei |

---

## Ziel-Aufbau (nach Umstieg)

```
Internet
    ↓ (PPPoE)
DEC3852 OPNsense (igc1 = WAN, igc0 = Trunk)
    ↓ Trunk (alle VLANs tagged)
Cisco RV345P (nur noch als Switch, kein Routing/DHCP)
    ↓ 16 Ports mit VLAN-Zuordnung wie bisher
    → Dosen im Haus
```

---

## VLAN-Konfiguration (vom Cisco übernommen)

### VLAN-Übersicht

| VLAN ID | Name | Gateway-IP | Subnetz | DHCP Range | DHCP aktiv | Inter-VLAN Routing | Device Mgmt |
|---------|------|-----------|---------|------------|------------|-------------------|-------------|
| 1 | VLAN1 | 192.168.1.1/24 | 255.255.255.0 | .10 – .240 | Ja | Nein | Ja |
| 2 | VLAN2 | 192.168.2.1/24 | 255.255.255.0 | .100 – .149 | Ja | Nein | Nein |
| 4 | VLAN4 | 192.168.4.1/24 | 255.255.255.0 | – | Nein | Nein | Nein |
| 42 | VLAN42 | 192.168.42.1/24 | 255.255.255.0 | .100 – .200 | Ja | Nein | Nein |
| 80 | VLAN80 | 192.168.80.1/24 | 255.255.255.0 | .10 – .200 | Ja | Nein | Ja |
| 88 | VLAN88 | 192.168.88.1/24 | 255.255.255.0 | .10 – .200 | Ja | Nein | Nein |
| 90 | VLAN90 | 10.4.250.1/24 | 255.255.255.0 | .100 – .149 | Ja | Nein | Nein |
| 188 | VLAN188 | 192.168.188.254/24 | 255.255.255.0 | – | Nein | Nein | Nein |
| 189 | VLAN189 | 192.168.189.254/24 | 255.255.255.0 | – | Nein | Nein | Nein |

### Cisco Port-zu-VLAN-Zuordnung

**Legende:** U = Untagged, T = Tagged, E = Excluded

| VLAN | LAN1 | LAN2 | LAN3 | LAN4 | LAN5 | LAN6 | LAN7 | LAN8 | LAN9 | LAN10 | LAN11 | LAN12 | LAN13 | LAN14 | LAN15 | LAN16 |
|------|------|------|------|------|------|------|------|------|------|-------|-------|-------|-------|-------|-------|-------|
| 1 | U | U | U | U | U | U | U | U | U | E | E | E | U | E | T | E |
| 2 | E | E | E | E | T | E | E | E | E | E | E | U | T | E | E | E |
| 4 | T | T | T | T | T | T | T | T | T | T | T | T | T | T | T | T |
| 42 | E | E | E | E | T | E | E | E | E | E | E | E | E | E | U | E |
| 80 | T | T | T | T | T | E | E | T | T | U | U | E | T | E | E | U |
| 88 | T | T | T | T | T | T | T | T | T | T | T | T | T | T | T | T |
| 90 | T | T | T | T | T | T | T | T | T | T | T | T | T | T | T | T |
| 188 | E | E | E | E | E | E | E | E | E | E | E | T | U | E | E | E |
| 189 | E | E | E | E | E | E | E | E | E | E | E | E | T | E | E | E |

**Zusammenfassung Untagged-Zuordnung pro Port:**

| Port | Untagged VLAN | Vermutete Nutzung |
|------|--------------|-------------------|
| LAN1–LAN9 | VLAN1 | Allgemeines Netz / Management |
| LAN10 | VLAN80 | P&M Hauptnetz |
| LAN11 | VLAN80 | P&M Hauptnetz |
| LAN12 | VLAN2 | Anderes Unternehmen |
| LAN13 | VLAN1 | Allgemein (+ VLAN188/189 tagged) |
| LAN14 | VLAN188 | Anderes Unternehmen |
| LAN15 | VLAN42 | Anderes Unternehmen |
| LAN16 | VLAN80 | P&M Hauptnetz |

---

## Durchgeführte Schritte auf der DEC3852

### 1. System-Grundeinstellungen (erledigt ✅)
- Admin-Passwort geändert (Default war `root` / `opnsense`)
- Hostname und Domain gesetzt
- Zeitzone: Europe/Berlin
- DNS-Server konfiguriert

### 2. LAN-Interface (erledigt ✅)
- LAN-Interface (`igc0`): IP `192.168.1.1/24` (VLAN1, ungetaggt)
- DHCP-Server aktiviert: Range `192.168.1.10` – `192.168.1.240`

### 3. VLANs angelegt (erledigt ✅)
Unter Interfaces → Devices → VLAN, alle mit Parent `igc0`:

| VLAN Tag | Description | Status |
|----------|------------|--------|
| 2 | VLAN2 | ✅ Angelegt |
| 4 | VLAN4 | ✅ Angelegt |
| 42 | VLAN42 | ✅ Angelegt |
| 80 | VLAN80 | ✅ Angelegt |
| 88 | VLAN88 | ✅ Angelegt |
| 90 | VLAN90 | ✅ Angelegt |
| 188 | VLAN188 | ✅ Angelegt |
| 189 | VLAN189 | ✅ Angelegt |

> VLAN1 wird nicht separat angelegt — ungetaggter Traffic auf `igc0` ist automatisch VLAN1.

---

## Offene Schritte

### 4. VLANs als Interfaces zuweisen und IPs vergeben (offen ❌)
Unter Interfaces → Assignments die angelegten VLANs als neue Interfaces hinzufügen und IPs zuweisen:

| VLAN | Zuzuweisende IP |
|------|----------------|
| VLAN2 | 192.168.2.1/24 |
| VLAN4 | 192.168.4.1/24 |
| VLAN42 | 192.168.42.1/24 |
| VLAN80 | 192.168.80.1/24 |
| VLAN88 | 192.168.88.1/24 |
| VLAN90 | 10.4.250.1/24 |
| VLAN188 | 192.168.188.254/24 |
| VLAN189 | 192.168.189.254/24 |

### 5. DHCP-Server pro VLAN konfigurieren (offen ❌)
Für jedes VLAN mit aktivem DHCP den Range einstellen:

| VLAN | DHCP Range |
|------|-----------|
| VLAN2 | 192.168.2.100 – 192.168.2.149 |
| VLAN42 | 192.168.42.100 – 192.168.42.200 |
| VLAN80 | 192.168.80.10 – 192.168.80.200 |
| VLAN88 | 192.168.88.10 – 192.168.88.200 |
| VLAN90 | 10.4.250.100 – 10.4.250.149 |

VLAN4, VLAN188, VLAN189: kein DHCP.

### 6. Static DHCP Einträge übernehmen (offen ❌)
Vom Cisco unter LAN → Static DHCP alle Einträge dokumentieren und übernehmen.  
Bekannte Einträge:
- FritzBox 7590: `192.168.80.44`

### 7. Firewall-Regeln pro VLAN (offen ❌)
- Grundregel pro VLAN: Allow outbound (Internet)
- Inter-VLAN Routing nur für VLAN1 und VLAN80 (wie auf dem Cisco)
- Alle anderen VLANs: kein Zugriff auf andere VLANs (Isolation der anderen Unternehmen)

### 8. Port-Forwarding für FritzBox (offen ❌)
Auf der DEC3852 einrichten (auf dem Cisco wegen ESP-Limitierung gescheitert):
- UDP 500 (IKE) → 192.168.80.44
- UDP 4500 (NAT-T) → 192.168.80.44
- Protokoll 50 (ESP) → 192.168.80.44

### 9. WAN / PPPoE-Konfiguration (offen ❌ – Umstieg-Tag)
- PPPoE-Zugangsdaten eintragen (ISP-Zugangsdaten klären!)
- Ggf. VLAN-Tag auf WAN-Interface (z.B. VLAN7 bei Telekom)
- WAN-Interface: `igc1`

### 10. Firmware-Update (offen ❌ – Umstieg-Tag)
- Direkt nach Internetverbindung unter System → Firmware aktualisieren
- Reboot abwarten

### 11. Cisco RV345P als Switch umkonfigurieren (offen ❌ – Umstieg-Tag)
- WAN-Kabel entfernen
- DHCP-Server auf allen VLANs deaktivieren
- Routing deaktivieren
- Einen Port als Trunk zur DEC3852 konfigurieren (alle VLANs tagged)
- Restliche Port-zu-VLAN-Zuordnung beibehalten

### 12. VPN-Konfiguration (offen ❌ – mit Mauro)
- Config-Export von der kleinen OPNsense (192.168.80.59) durch Mauro
- VPN-Tunnel übernehmen/importieren (Bürkert, Tyczka, Adacor, Sonic etc.)
- Mauro ist Ansprechpartner für VPN-Infrastruktur

### 13. VPN-Monitoring einrichten (offen ❌ – nach Umstieg)
- OPNsense hat eingebaute Monitoring-Funktionen
- Wird nach stabilem Betrieb konfiguriert
- Alternativ: Ping-Script mit Slack-Alert

---

## Hinweise

- **Kein Firmware-Update vor Umstieg möglich** – DEC3852 ist offline, Update erfolgt am Umstieg-Tag
- **Config-Import möglich** – OPNsense speichert alles in einer XML-Datei (System → Configuration → Backups), teilweiser Import einzelner Sektionen möglich aber mit Vorsicht
- **Cisco als Switch:** Falls der Cisco nicht sauber als Switch funktioniert, muss ein Managed Switch beschafft werden
- **ISP/PPPoE-Zugangsdaten** müssen vor dem Umstieg vorliegen

---

## Ansprechpartner

| Thema | Person |
|-------|--------|
| DEC3852 Konfiguration | Inan Bogisch |
| VPN-Infrastruktur & Config | Mauro Altamura |
| Infrastruktur-Fragen Hamburg | Mathias |
