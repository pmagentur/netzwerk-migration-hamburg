# Netzwerk-Migration Hamburg

Ablösung des Cisco RV345P durch eine OPNsense DEC3852 als Hauptrouter am Standort Hamburg.

## Status

| Schritt | Beschreibung | Status |
|---------|-------------|--------|
| 1 | Grundkonfiguration (Hostname, DNS, Zeitzone) | ✅ Erledigt |
| 2 | LAN-Interface (igc0, VLAN1) | ✅ Erledigt |
| 3 | VLANs anlegen | ✅ Erledigt |
| 4 | VLANs als Interfaces zuweisen + IPs | ✅ Erledigt |
| 5 | DHCP-Server pro VLAN | ✅ Erledigt |
| 6 | VLAN90 DNS-Server nachtragen | ✅ Erledigt |
| 7 | Static DHCP Einträge übernehmen | ✅ Erledigt (21/21) |
| 8 | Firewall-Regeln | ✅ Erledigt (9 Interfaces, Block Inter-VLAN + Allow Internet) |
| 9 | Port-Forwarding | ✅ Erledigt (20 aktive + 2 FritzBox VPN vorbereitet) |
| 10 | Static NAT (Telefonanlage) | ⏸️ Verschoben – WAN1 auf Cisco DOWN |
| 11 | DNS Host Overrides + DNSSEC | ✅ Erledigt (6 Overrides, DNSSEC aktiv) |
| 12 | WAN / PPPoE | ❌ Offen – Umstieg-Tag |
| 13 | Firmware-Update | ❌ Offen – Umstieg-Tag |
| 14 | Cisco als Switch umkonfigurieren | ❌ Offen – Umstieg-Tag |
| 15 | VPN-Tunnel + statische Routen | ❌ Offen – mit Mauro |
| 16 | VPN-Monitoring | ❌ Offen – nach Umstieg |
| 17 | Kleinere Korrekturen | ✅ Erledigt (ECSIT_NAS Port-Range, VPN-Domain) |

## Vorkonfiguration abgeschlossen ✅

Die OPNsense ist so weit wie möglich vorkonfiguriert. Am Umstieg-Tag müssen nur noch Dinge erledigt werden, die den physischen Anschluss oder Mauro (VPN) erfordern.

### Am Umstieg-Tag zu tun:
1. PPPoE-Zugangsdaten auf WAN-Interface eintragen
2. Ggf. VLAN-Tag auf WAN (z.B. VLAN7 bei Telekom)
3. Firmware-Update durchführen
4. Cisco RV345P als reinen Switch umkonfigurieren (Routing/DHCP deaktivieren)
5. Klären ob WAN1 / Static NAT (31.172.106.157) für TK noch gebraucht wird

### Mit Mauro (VPN):
1. 8 Site-to-Site IPsec-Tunnel von Cisco übernehmen
2. Statische Routen für HagebauIT anlegen (10.0.8.0/21, 10.32.160.0/22 via VLAN90)
3. FritzBox VPN: ESP-Weiterleitung + Konflikt mit S2S auf Port 500/4500 lösen

## Dokumentation

- [`DEC3852_Doku_16-04-2026.md`](DEC3852_Doku_16-04-2026.md) – Vollständige Konfigurationsdoku

## Wichtige Erkenntnisse

### 14.04.2026
- Alle Portnummern aus der Cisco-Config (XML) erfolgreich extrahiert
- **Korrektur:** VPNWG-Ziel ist 192.168.80.48 (nicht 192.168.88.48)
- 21 Static-DHCP-Einträge aus Cisco identifiziert (inkl. MAC-Adressen)
- Cisco-Config ist nicht direkt auf OPNsense importierbar (unterschiedliches XML-Format)

### 16.04.2026
- **Bug gefunden & gefixt:** LAN DHCP-Range stand auf 192.168.80.10–200 statt 192.168.1.10–240
- Alle 21 Static DHCP Einträge per Config-Import eingespielt
- Firewall-Regeln händisch angelegt (Alias RFC1918 + Block/Allow pro Interface)
- 22 Port-Forwards händisch angelegt (inkl. 2× FritzBox VPN Vorbereitung)
- DNS Host Overrides vom Cisco übernommen (6 Einträge)
- DNSSEC aktiviert, DHCP-Lease-Registrierung aktiviert
- Static NAT verschoben: WAN1 war auch auf Cisco bereits DOWN
- Statische Routen (HagebauIT) werden erst mit Mauro bei VPN-Setup angelegt
- ECSIT_NAS Port-Forward auf Range 50005–50006 korrigiert
- VPN DNS Override Domain auf `pm-projects.de` korrigiert (Tippfehler)

## Beteiligte

| Wer | Rolle |
|-----|-------|
| Inan | DEC3852 Konfiguration |
| Mauro | VPN-Infrastruktur & Config |
| Mathias | Infrastruktur-Fragen Hamburg |
