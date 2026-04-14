# DEC3852 OPNsense – Konfigurationsdokumentation

**Datum:** 14.04.2026 (aktualisiert)  
**Durchgeführt von:** Inan Bogisch  
**Hardware:** Deciso DEC3852 (OPNsense Rack Security Appliance)  
**Zweck:** Ablösung des Cisco RV345P (192.168.80.1) als Hauptrouter in Hamburg  
**Status:** 🔧 VLANs + DHCP erledigt, Firewall/NAT/Port-Forwarding offen

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

> ⚠️ **TODO:** VLAN90 DNS-Server auf OPNsense nachtragen (Services → DHCPv4 → VLAN90)

### IPv6-Konfiguration (Cisco – Übernahme offen)

Der Cisco hat auf allen VLANs IPv6 mit fec0: (Site-Local) konfiguriert, aber **IPv6 DHCP ist überall deaktiviert**.

| VLAN | IPv6-Adresse | DHCPv6 |
|------|-------------|--------|
| VLAN1 | fec0::1/64 | Disabled |
| VLAN2 | fec0:2::1/64 | Disabled |
| VLAN4 | fec0:6::1/64 | Disabled |
| VLAN42 | fec0:7::1/64 | Disabled |
| VLAN80 | fec0:4::1/64 | Disabled |
| VLAN88 | fec0:5::1/64 | Disabled |
| VLAN90 | fec0:8::1/64 | Disabled |
| VLAN188 | fec0:1::1/64 | Disabled |
| VLAN189 | fec0:3::1/64 | Disabled |

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
- Admin-Passwort geändert (Default war `root` / `opnsense`)
- Hostname und Domain gesetzt
- Zeitzone: Europe/Berlin
- DNS-Server konfiguriert

### 2. LAN-Interface (erledigt ✅)
- LAN-Interface (`igc0`): IP `192.168.1.1/24` (VLAN1, ungetaggt)
- DHCP-Server aktiviert: Range `192.168.1.10` – `192.168.1.240`

### 3. VLANs angelegt (erledigt ✅)
Unter Interfaces → Devices → VLAN, alle mit Parent `igc0`:

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

> VLAN1 wird nicht separat angelegt — ungetaggter Traffic auf `igc0` ist automatisch VLAN1.  
> Die Device-Reihenfolge (vlan01–08) ist nur intern und hat keinen Einfluss auf die Funktion.

### 4. VLANs als Interfaces zugewiesen + IPs vergeben (erledigt ✅)
Unter Interfaces → Assignments alle VLANs hinzugefügt, aktiviert und IPs vergeben:

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
| VLAN1 (LAN) | 192.168.1.10 – 192.168.1.240 | ✅ (aus Schritt 2) |
| VLAN2 | 192.168.2.100 – 192.168.2.149 | ✅ |
| VLAN42 | 192.168.42.100 – 192.168.42.200 | ✅ |
| VLAN80 | 192.168.80.10 – 192.168.80.200 | ✅ |
| VLAN88 | 192.168.88.10 – 192.168.88.200 | ✅ |
| VLAN90 | 10.4.250.100 – 10.4.250.149 | ✅ |
| VLAN4 | kein DHCP | ✅ |
| VLAN188 | kein DHCP | ✅ |
| VLAN189 | kein DHCP | ✅ |

---

## Offene Schritte

### 6. VLAN90 DNS-Server nachtragen (offen ❌)
Unter Services → DHCPv4 → VLAN90 die statischen DNS-Server eintragen:
- DNS1: `10.32.160.10`
- DNS2: `10.4.250.100`

### 7. Static DHCP Einträge übernehmen (offen ❌)
MAC-Adresse der FritzBox aus der Cisco ARP-Tabelle holen und auf OPNsense eintragen:
- FritzBox 7590: `192.168.80.44` (MAC: aus ARP-Tabelle)

Weitere statische Einträge vom Cisco prüfen und übernehmen.

### 8. Firewall-Regeln (offen ❌)
Zu übernehmen basierend auf Cisco-Konfiguration. Details siehe Abschnitt "Cisco Firewall-Konfiguration" unten.

### 9. Port-Forwarding übernehmen (offen ❌)
Alle aktiven Port-Forwarding-Regeln vom Cisco auf OPNsense übertragen. Details siehe Abschnitt "Port-Forwarding" unten.

> ⚠️ **Wichtig:** Die genauen Ports hinter den Service-Namen (TK_HTTPS, SIP1, VPNPC1 etc.) müssen aus der Cisco-Config-Datei extrahiert werden. Bisher sind nur die Service-Namen und Ziel-IPs bekannt.

### 10. Static NAT für Telefonanlage einrichten (offen ❌)
Details siehe Abschnitt "Static NAT" unten.

### 11. WAN / PPPoE-Konfiguration (offen ❌ – Umstieg-Tag)
- PPPoE-Zugangsdaten eintragen (ISP-Zugangsdaten klären!)
- Ggf. VLAN-Tag auf WAN-Interface (z.B. VLAN7 bei Telekom)
- WAN-Interface: `igc1`

### 12. Firmware-Update (offen ❌ – Umstieg-Tag)
- Direkt nach Internetverbindung unter System → Firmware aktualisieren
- Reboot abwarten

### 13. Cisco RV345P als Switch umkonfigurieren (offen ❌ – Umstieg-Tag)
- WAN-Kabel entfernen
- DHCP-Server auf allen VLANs deaktivieren
- Routing deaktivieren
- Einen Port als Trunk zur DEC3852 konfigurieren (alle VLANs tagged)
- Restliche Port-zu-VLAN-Zuordnung beibehalten

### 14. VPN-Konfiguration (offen ❌ – mit Mauro)
- Config-Export von der kleinen OPNsense (192.168.80.59) durch Mauro
- VPN-Tunnel übernehmen/importieren
- Mauro ist Ansprechpartner für VPN-Infrastruktur

### 15. VPN-Monitoring einrichten (offen ❌ – nach Umstieg)
- OPNsense hat eingebaute Monitoring-Funktionen
- Wird nach stabilem Betrieb konfiguriert
- Alternativ: Ping-Script mit Slack-Alert

---

## Cisco Firewall-Konfiguration (Referenz für Übernahme)

### Grundeinstellungen

- Firewall: Enabled
- DoS Protection: Enabled
- Block WAN Request: Enabled
- Remote Web Management: Disabled
- SIP ALG: Disabled
- UPnP: Disabled
- Session Timeouts: TCP 1800s, UDP 120s, ICMP 60s
- Max Concurrent Connections: 40000

### Generelle Access Rules

| Prio | Action | Protokoll | Source IF | Source | Dest IF | Dest | Beschreibung |
|------|--------|-----------|----------|--------|---------|------|-------------|
| 4001 | Allow | IPv4: All Traffic | VLAN | Any | WAN | Any | Alles raus (IPv4) |
| 4002 | Deny | IPv4: All Traffic | WAN | Any | VLAN | Any | Alles rein blocken (IPv4) |
| 4001 | Allow | IPv6: All Traffic | VLAN | Any | WAN | Any | Alles raus (IPv6) |
| 4002 | Deny | IPv6: All Traffic | WAN | Any | VLAN | Any | Alles rein blocken (IPv6) |

> **Hinweis:** Es gibt keine expliziten Inter-VLAN-Block-Regeln. Die Isolation erfolgt über die VLAN-Konfiguration selbst ("Inter-VLAN Routing: Nein").

### Port-Forwarding (aktiv ✅)

#### Über WAN2 (PPPoE-Verbindung):

| Name | Service | Ziel-IP | VLAN | Beschreibung |
|------|---------|---------|------|-------------|
| MyFritz | MyFritz | 192.168.1.5 | VLAN1 | FritzBox MyFritz-Zugang |
| SIP1 | SIP1 | 192.168.1.5 | VLAN1 | SIP-Telefonie |
| SIP1-UDP | SIP1-UDP | 192.168.1.5 | VLAN1 | SIP-Telefonie UDP |
| SIP2 | SIP2 | 192.168.1.4 | VLAN1 | SIP-Telefonie |
| SIP2-UDP | SIP2-UDP | 192.168.1.4 | VLAN1 | SIP-Telefonie UDP |
| MyFritz-PM | MyFritz-PM | 192.168.1.4 | VLAN1 | FritzBox P&M MyFritz |
| SIP3 | SIP3 | 192.168.80.201 | VLAN80 | SIP-Telefonie |
| SIP3-UPD | SIP3-UPD | 192.168.80.201 | VLAN80 | SIP-Telefonie UDP |

#### Über Any (beide WANs):

| Name | Service | Ziel-IP | VLAN | Beschreibung |
|------|---------|---------|------|-------------|
| TK_HTTPS | TK_HTTPS | 192.168.80.204 | VLAN80 | Telefonanlage HTTPS |
| TK_RTP | TK_RTP | 192.168.80.204 | VLAN80 | Telefonanlage RTP |
| TK_SIP | TK_SIP | 192.168.80.204 | VLAN80 | Telefonanlage SIP |
| TK_SIP_TLS | TK_SIP_TLS | 192.168.80.204 | VLAN80 | Telefonanlage SIP-TLS |
| TK_TunnelProtocolServiceListener | TK_TunnelProtocolServiceListener | 192.168.80.204 | VLAN80 | Telefonanlage Tunnel |
| VPNPC1 | VPNPC1 | 192.168.80.59 | VLAN80 | VPN-Appliance |
| VPNPC2 | VPNPC2 | 192.168.80.59 | VLAN80 | VPN-Appliance |
| VPNPC3 | VPNPC3 | 192.168.80.59 | VLAN80 | VPN-Appliance |
| ECSIT_NAS | ECSIT_NAS | 192.168.42.179 | VLAN42 | NAS-Zugang |
| HTTPS | HTTPS | 192.168.80.74 | VLAN80 | Webserver HTTPS |
| HTTP | HTTP | 192.168.80.74 | VLAN80 | Webserver HTTP |
| VPNWG | VPNWG | 192.168.88.48 | VLAN88 | WireGuard VPN |

> ⚠️ **Die genauen Portnummern hinter den Service-Namen müssen aus der Cisco-Config-Datei extrahiert werden!**

#### Deaktiviert (❌ – nicht übernehmen):

| Name | Ziel-IP | Beschreibung |
|------|---------|-------------|
| ISAKMP | 192.168.80.44 | FritzBox IPsec (WAN2) – war auf Cisco nie aktiv |
| IPSEC-UDP-ENCAP | 192.168.80.44 | FritzBox IPsec (WAN2) – war auf Cisco nie aktiv |
| IPSEC-UDP-ENCAP | 192.168.88.1 | IPsec auf VLAN88 GW |
| ISAKMP | 192.168.88.1 | IPsec auf VLAN88 GW |
| L2TP | 192.168.88.1 | L2TP |
| L2TP_500 | 192.168.88.1 | L2TP Port 500 |
| L2TP_1701 | 192.168.88.1 | L2TP Port 1701 |
| L2TP_4500 | 192.168.88.1 | L2TP Port 4500 |

### Static NAT (WAN1 – Telefonanlage)

| Private IP | Public IP | Services | Interface |
|-----------|----------|----------|-----------|
| 192.168.80.204 | 31.172.106.157 | TK_HTTPS | WAN1 |
| 192.168.80.204 | 31.172.106.157 | TK_SIP | WAN1 |
| 192.168.80.204 | 31.172.106.157 | TK_TunnelProtocolServiceListener | WAN1 |

> **Hinweis:** Die Telefonanlage (192.168.80.204) hat eine feste öffentliche IP 31.172.106.157 über WAN1. Zugehörige Access Rules (2001–2003) erlauben Traffic von 31.172.106.157 zur Telefonanlage.

### Access Rules mit spezifischer Source-IP (WAN1)

| Prio | Service | Source IF | Source IP | Dest-IP | Beschreibung |
|------|---------|----------|-----------|---------|-------------|
| 2001 | TK_HTTPS | WAN1 | 31.172.106.157 | 192.168.80.204 | TK HTTPS von Public IP |
| 2002 | TK_SIP | WAN1 | 31.172.106.157 | 192.168.80.204 | TK SIP von Public IP |
| 2003 | TK_TunnelProtocol | WAN1 | 31.172.106.157 | 192.168.80.204 | TK Tunnel von Public IP |

---

## VPN-Konfiguration (Cisco – Referenz, Übernahme durch Mauro)

### Site-to-Site Tunnel (IPsec auf Cisco RV345P)

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

### Remote Access VPN (Client-to-Site)

| # | Group/Tunnel Name | Connections | Local Group |
|---|------------------|------------|-------------|
| 1 | PM_Remote_Wedolo | 0 | 0.0.0.0/0 |
| 2 | PM_Remote | 1 | 0.0.0.0/0 |

### IPsec Profiles

| Name | IKE Version | Policy | In Use |
|------|------------|--------|--------|
| Amazon_Web_Services | IKEv1 | Auto | No |
| Default | IKEv1 | Auto | Yes |
| Microsoft_Azure | IKEv1 | Auto | No |
| BSSWedolo | IKEv1 | Auto | Yes |
| PM_L2TP | IKEv1 | Auto | Yes |
| HagebauIT | IKEv1 | Auto | No |
| hagebau2 | IKEv2 | Auto | No |
| hagebau3 | IKEv2 | Auto | No |
| hagebauCopyv1 | IKEv1 | Auto | No |
| HagebaunewVPN | IKEv2 | Auto | Yes |

> **Hinweis:** Die VPN-Konfiguration wird von Mauro verwaltet. Die Tunnel laufen aktuell auf dem Cisco und auf der kleinen OPNsense (192.168.80.59). Mauro liefert den Config-Export.

### Port-Forwarding für FritzBox VPN (neu auf OPNsense)

Auf dem Cisco war das FritzBox-Port-Forwarding deaktiviert (ESP-Limitierung). Auf der OPNsense neu einrichten:
- UDP 500 (IKE) → 192.168.80.44
- UDP 4500 (NAT-T) → 192.168.80.44
- Protokoll 50 (ESP) → 192.168.80.44

---

## Hinweise

- **Kein Firmware-Update vor Umstieg möglich** – DEC3852 ist offline, Update erfolgt am Umstieg-Tag
- **Config-Import möglich** – OPNsense speichert alles in einer XML-Datei (System → Configuration → Backups), teilweiser Import einzelner Sektionen möglich aber mit Vorsicht
- **Cisco als Switch:** Falls der Cisco nicht sauber als Switch funktioniert, muss ein Managed Switch beschafft werden
- **ISP/PPPoE-Zugangsdaten** müssen vor dem Umstieg vorliegen
- **Cisco-Config-Datei** exportieren (Administration → Configuration Management → Download to PC) und aufbewahren – enthält die genauen Portnummern für alle Service-Definitionen

---

## Ansprechpartner

| Thema | Person |
|-------|--------|
| DEC3852 Konfiguration | Inan Bogisch |
| VPN-Infrastruktur & Config | Mauro Altamura |
| Infrastruktur-Fragen Hamburg | Mathias |
