# DEC3852 OPNsense – Konfigurationsdokumentation

**Datum:** 14.04.2026 (aktualisiert)  
**Durchgeführt von:** Inan Bogisch  
**Hardware:** Deciso DEC3852 (OPNsense Rack Security Appliance)  
**Zweck:** Ablösung des Cisco RV345P (192.168.80.1) als Hauptrouter in Hamburg  
**Status:** 🔧 VLANs + DHCP + DNS erledigt, Static DHCP teilweise, Firewall/NAT/Port-Forwarding offen

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

### DHCP DNS-Konfiguration

| VLAN | DNS-Modus | DNS-Server |
|------|-----------|-----------|
| VLAN1 | DNS Proxy (OPNsense) | – |
| VLAN2 | DNS Proxy (OPNsense) | – |
| VLAN42 | DNS Proxy (OPNsense) | – |
| VLAN80 | DNS Proxy (OPNsense) | – |
| VLAN88 | DNS Proxy (OPNsense) | – |
| VLAN90 | **Statische DNS** | **10.32.160.10, 10.4.250.100** |

### IPv6-Konfiguration

**Entscheidung:** IPv6 auf OPNsense erstmal weglassen, nach Umstieg prüfen ob etwas fehlt.

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

**Untagged-Zuordnung pro Port:**

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
- Admin-Passwort geändert
- Hostname und Domain gesetzt
- Zeitzone: Europe/Berlin
- DNS-Server konfiguriert

### 2. LAN-Interface (erledigt ✅)
- LAN-Interface (`igc0`): IP `192.168.1.1/24` (VLAN1, ungetaggt)
- DHCP-Server aktiviert: Range `192.168.1.10` – `192.168.1.240`

### 3. VLANs angelegt (erledigt ✅)

| Device | VLAN Tag | Description | Status |
|--------|----------|------------|--------|
| vlan01 | 2 | VLAN2 | ✅ |
| vlan02 | 4 | VLAN4 | ✅ |
| vlan03 | 42 | VLAN42 | ✅ |
| vlan04 | 88 | VLAN88 | ✅ |
| vlan05 | 90 | VLAN90 | ✅ |
| vlan06 | 188 | VLAN188 | ✅ |
| vlan07 | 189 | VLAN189 | ✅ |
| vlan08 | 80 | VLAN80 | ✅ |

### 4. VLANs als Interfaces zugewiesen + IPs vergeben (erledigt ✅)

| VLAN | IP | Status |
|------|----|--------|
| VLAN2 | 192.168.2.1/24 | ✅ |
| VLAN4 | 192.168.4.1/24 | ✅ |
| VLAN42 | 192.168.42.1/24 | ✅ |
| VLAN80 | 192.168.80.1/24 | ✅ |
| VLAN88 | 192.168.88.1/24 | ✅ |
| VLAN90 | 10.4.250.1/24 | ✅ |
| VLAN188 | 192.168.188.254/24 | ✅ |
| VLAN189 | 192.168.189.254/24 | ✅ |

### 5. DHCP-Server pro VLAN konfiguriert (erledigt ✅)

| VLAN | DHCP Range | Status |
|------|-----------|--------|
| VLAN1 (LAN) | 192.168.1.10 – 192.168.1.240 | ✅ |
| VLAN2 | 192.168.2.100 – 192.168.2.149 | ✅ |
| VLAN42 | 192.168.42.100 – 192.168.42.200 | ✅ |
| VLAN80 | 192.168.80.10 – 192.168.80.200 | ✅ |
| VLAN88 | 192.168.88.10 – 192.168.88.200 | ✅ |
| VLAN90 | 10.4.250.100 – 10.4.250.149 | ✅ |
| VLAN4 | kein DHCP | ✅ |
| VLAN188 | kein DHCP | ✅ |
| VLAN189 | kein DHCP | ✅ |

### 6. VLAN90 DNS-Server nachgetragen (erledigt ✅)
- DNS1: `10.32.160.10`
- DNS2: `10.4.250.100`

### 7. Static DHCP Einträge übernehmen (teilweise ✅ / offen ❌)

**Händisch eingetragen (4 von 21):**

| Name | IP | MAC | VLAN | Status |
|------|-----|-----|------|--------|
| fritz_telefon | 192.168.1.5 | 44:4e:6d:de:8e:27 | VLAN1 | ✅ |
| fritz_lan | 192.168.1.3 | 7c:ff:4d:c2:9e:9d | VLAN1 | ✅ |
| fritz_pm_telefon | 192.168.1.4 | 44:4e:6d:d8:26:0c | VLAN1 | ✅ |
| pma-tk01 | 192.168.80.204 | 1c:69:7a:6d:66:79 | VLAN80 | ✅ |

**Noch einzutragen (17 Einträge – per Config-Import am Donnerstag):**

#### VLAN80 (16 Einträge)

| Name | IP | MAC |
|------|-----|-----|
| VLAN80_WP_Server | 192.168.80.250 | 42:82:66:ff:7f:f0 |
| PM_Brother_L2700DW | 192.168.80.83 | 60:6d:c7:69:08:f8 |
| PM_HP_7740 | 192.168.80.185 | b4:b6:86:54:f8:f1 |
| VPN-PC | 192.168.80.59 | 00:0d:b9:56:13:90 |
| PM_Mikrotik_Switch | 192.168.80.144 | 18:fd:74:58:e3:99 |
| Buerkert Server | 192.168.80.28 | ac:1f:6b:ee:a5:d9 |
| Silver Windows Server | 192.168.80.27 | 3c:ec:ef:8c:40:14 |
| Silver Dev Server | 192.168.80.74 | ac:1f:6b:dc:f5:3a |
| Silver LDAP-DNS | 192.168.80.171 | 00:15:5d:01:29:0b |
| Silver VMware ESXi | 192.168.80.29 | 40:f2:e9:98:86:aa |
| MySQL Server iLO | 192.168.80.25 | 2c:59:e5:3b:c7:28 |
| MySQL Server | 192.168.80.26 | f0:92:1c:11:ef:cc |
| Sonic PreStage | 192.168.80.205 | 00:15:5d:01:29:09 |
| QNAP restic | 192.168.80.249 | 00:08:9b:d4:de:78 |
| QNAP HV | 192.168.80.253 | 00:08:9b:c6:80:72 |
| FritzBox 7590 | 192.168.80.44 | dc:39:6f:e0:d6:4d |

#### VLAN42 (1 Eintrag)

| Name | IP | MAC |
|------|-----|-----|
| ECSIT_SYN_NAS | 192.168.42.179 | 00:11:32:99:a7:15 |

> **Plan Donnerstag:** OPNsense-Backup hochladen → Static DHCP per Config-Import einfügen → Config zurückspielen

---

## Offene Schritte

### 8. Firewall-Regeln (offen ❌)

#### Grundeinstellungen auf OPNsense umsetzen:
- DoS Protection: Aktivieren (Firewall → Settings → Advanced)
- SIP ALG: Deaktiviert lassen
- Session Timeouts: TCP 1800s, UDP 120s, ICMP 60s

#### Basis-Regeln (analog Cisco):

**Pro VLAN-Interface:**
- Allow IPv4 All → WAN (= Internet-Zugang erlauben)

**WAN-Interface:**
- Default: Block all inbound (OPNsense Standard)

**Inter-VLAN:**
- Kein Inter-VLAN-Routing → auf jedem VLAN-Interface Block-Regel für RFC1918 als Destination

### 9. Port-Forwarding übernehmen (offen ❌)

**Portnummern vollständig aus Cisco-Config extrahiert:**

| Name | Protokoll | Ext. Port(s) | Ziel-IP | Int. Port(s) | VLAN | Beschreibung |
|------|-----------|-------------|---------|-------------|------|-------------|
| MyFritz | TCP | 39888 | 192.168.1.5 | 39888 | VLAN1 | FritzBox MyFritz-Zugang |
| SIP1 | TCP+UDP | 5060 | 192.168.1.5 | 5060 | VLAN1 | SIP-Telefonie FritzBox 1 |
| SIP1-UDP | UDP | 30000–30006 | 192.168.1.5 | 30000–30006 | VLAN1 | SIP RTP FritzBox 1 |
| SIP2 | TCP+UDP | 5160 | 192.168.1.4 | 5160 | VLAN1 | SIP-Telefonie FritzBox 2 |
| SIP2-UDP | UDP | 30007–30012 | 192.168.1.4 | 30007–30012 | VLAN1 | SIP RTP FritzBox 2 |
| MyFritz-PM | TCP | 47341 | 192.168.1.4 | 47341 | VLAN1 | FritzBox P&M MyFritz |
| SIP3 | TCP+UDP | 5260 | 192.168.80.201 | 5260 | VLAN80 | SIP-Telefonie TK |
| SIP3-UPD | UDP | 50007–50200 | 192.168.80.201 | 50007–50200 | VLAN80 | SIP RTP TK |
| TK_HTTPS | TCP | 5001 | 192.168.80.204 | 5001 | VLAN80 | Telefonanlage HTTPS |
| TK_RTP | UDP | 9000–10999 | 192.168.80.204 | 9000–10999 | VLAN80 | Telefonanlage RTP |
| TK_SIP | TCP+UDP | 5061 | 192.168.80.204 | 5061 | VLAN80 | Telefonanlage SIP |
| TK_SIP_TLS | TCP | 5062 | 192.168.80.204 | 5062 | VLAN80 | Telefonanlage SIP-TLS |
| TK_TunnelProtocol | TCP+UDP | 5091 | 192.168.80.204 | 5091 | VLAN80 | Telefonanlage Tunnel |
| VPNPC1 | TCP | 51820 | 192.168.80.59 | 51820 | VLAN80 | VPN-Appliance WireGuard |
| VPNPC2 | TCP+UDP | 1194 | 192.168.80.59 | 1194 | VLAN80 | VPN-Appliance OpenVPN |
| VPNPC3 | TCP+UDP | 11194 | 192.168.80.59 | 11194 | VLAN80 | VPN-Appliance OpenVPN alt |
| ECSIT_NAS | TCP | 50005–50006 | 192.168.42.179 | 50005–50006 | VLAN42 | NAS-Zugang |
| HTTPS | TCP | 443 | 192.168.80.74 | 443 | VLAN80 | Webserver HTTPS |
| HTTP | TCP | 80 | 192.168.80.74 | 80 | VLAN80 | Webserver HTTP |
| VPNWG | UDP | 47111 | **192.168.80.48** | 47111 | VLAN80 | WireGuard VPN |

> ⚠️ **Korrektur:** VPNWG-Ziel ist laut Cisco-Config **192.168.80.48** (nicht 192.168.88.48 wie in der alten Doku).

#### Deaktiviert (NICHT übernehmen):

| Name | Protokoll | Port | Ziel-IP | Grund |
|------|-----------|------|---------|-------|
| ISAKMP | UDP | 500 | 192.168.80.44 / 192.168.88.1 | War nie aktiv |
| IPSEC-UDP-ENCAP | UDP | 4500 | 192.168.80.44 / 192.168.88.1 | War nie aktiv |
| L2TP | UDP | 1701 | 192.168.88.1 | Nicht mehr benötigt |
| L2TP_500 | UDP | 500 | 192.168.88.1 | Nicht mehr benötigt |
| L2TP_1701 | UDP | 1701 | 192.168.88.1 | Nicht mehr benötigt |
| L2TP_4500 | UDP | 4500 | 192.168.88.1 | Nicht mehr benötigt |

### 10. Static NAT für Telefonanlage einrichten (offen ❌)

Telefonanlage (192.168.80.204) hat feste öffentliche IP **31.172.106.157** über WAN1.

| Privat-IP | Public IP | Service | Protokoll | Port |
|-----------|----------|---------|-----------|------|
| 192.168.80.204 | 31.172.106.157 | TK_HTTPS | TCP | 5001 |
| 192.168.80.204 | 31.172.106.157 | TK_SIP | TCP+UDP | 5061 |
| 192.168.80.204 | 31.172.106.157 | TK_TunnelProtocol | TCP+UDP | 5091 |

> **Voraussetzung:** WAN1 mit IP 31.172.106.157 muss verfügbar sein.

### 11. WAN / PPPoE-Konfiguration (offen ❌ – Umstieg-Tag)
- PPPoE-Zugangsdaten eintragen
- Ggf. VLAN-Tag auf WAN-Interface (z.B. VLAN7 bei Telekom)
- WAN-Interface: `igc1`

### 12. Firmware-Update (offen ❌ – Umstieg-Tag)

### 13. Cisco RV345P als Switch umkonfigurieren (offen ❌ – Umstieg-Tag)

### 14. VPN-Konfiguration (offen ❌ – mit Mauro)

### 15. VPN-Monitoring einrichten (offen ❌ – nach Umstieg)

### 16. Port-Forwarding für FritzBox VPN (offen ❌ – neu auf OPNsense)
- UDP 500 (IKE) → 192.168.80.44
- UDP 4500 (NAT-T) → 192.168.80.44
- Protokoll 50 (ESP) → 192.168.80.44

---

## Cisco Service-Definitionen (Referenz)

| Service-Name | Protokoll | Port(s) |
|-------------|-----------|---------|
| MyFritz | TCP | 39888 |
| MyFritz-PM | TCP | 47341 |
| SIP1 | TCP+UDP | 5060 |
| SIP1-UDP | UDP | 30000–30006 |
| SIP2 | TCP+UDP | 5160 |
| SIP2-UDP | UDP | 30007–30012 |
| SIP3 | TCP+UDP | 5260 |
| SIP3-UPD | UDP | 50007–50200 |
| TK_HTTPS | TCP | 5001 |
| TK_RTP | UDP | 9000–10999 |
| TK_SIP | TCP+UDP | 5061 |
| TK_SIP_TLS | TCP | 5062 |
| TK_TunnelProtocolServiceListener | TCP+UDP | 5091 |
| VPNPC1 | TCP | 51820 |
| VPNPC2 | TCP+UDP | 1194 |
| VPNPC3 | TCP+UDP | 11194 |
| VPNWG | UDP | 47111 |
| ECSIT_NAS | TCP | 50005–50006 |
| HTTP | TCP | 80 |
| HTTPS | TCP | 443 |
| ISAKMP | UDP | 500 |
| IPSEC-UDP-ENCAP | UDP | 4500 |
| L2TP | UDP | 1701 |
| ESP | IP-Protokoll | 50 |

---

## VPN-Konfiguration (Cisco – Referenz, Übernahme durch Mauro)

### Site-to-Site Tunnel

| # | Name | Status | Encryption | Local Group | Remote Group | Remote Gateway |
|---|------|--------|-----------|-------------|-------------|---------------|
| 1 | Tyczka | UP | aes256-sha256-modp2048 | 192.168.80.74 | 10.113.15.0/24 | 185.213.32.21 |
| 2 | Sonic | UP | aes256-sha256-modp2048 | 192.168.80.74 | 10.199.27.0/24 | 62.209.53.124 |
| 3 | Buerkert_2_Adacor | DISABLED | aes256-sha256-modp1024 | 192.168.80.28 | Adacor | 130.0.79.136 |
| 4 | OMC | UP | aes256-sha256-modp2048 | 192.168.80.0/24 | 10.20.30.0/24 | 212.77.230.96 |
| 5 | Buerkert_1 | UP | aes256-sha256-modp2048 | 192.168.80.28 | 10.40.0.0/16 | 212.184.15.33 |
| 6 | Berlin_Office | UP | aes256-sha256-modp2048 | 192.168.80.0/24 | 10.10.252.0/24 | 87.234.222.66 |
| 7 | HagebauIT | UP | aes256-sha256-modp2048 | 10.4.250.0/24 | 10.0.8.0/10 | 217.237.163.70 |
| 8 | BSSWedolo | DOWN | aes256-sha256-modp1536 | 192.168.88.0/24 | 10.100.118.0/24 | 62.153.250.118 |

---

## Nächste Session: Donnerstag 16.04.2026

**Plan:**
1. OPNsense-Backup hochladen → restliche 17 Static DHCP per Config-Import
2. Schritt 8: Firewall-Regeln händisch anlegen (Lerneffekt!)
3. Schritt 9: Port-Forwarding händisch anlegen (Lerneffekt!)
4. Schritt 10: Static NAT für Telefonanlage

**Vorbereitung bis Donnerstag:**
- OPNsense-Backup runterladen: System → Configuration → Backups → Download
- Backup am Donnerstag hier hochladen

---

## Ansprechpartner

| Thema | Person |
|-------|--------|
| DEC3852 Konfiguration | Inan Bogisch |
| VPN-Infrastruktur & Config | Mauro Altamura |
| Infrastruktur-Fragen Hamburg | Mathias |
