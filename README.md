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
| 7 | Static DHCP Einträge übernehmen | 🔧 4/21 händisch, Rest per Config-Import |
| 8 | Firewall-Regeln | ❌ Offen |
| 9 | Port-Forwarding | ❌ Offen (Portnummern extrahiert) |
| 10 | Static NAT (Telefonanlage) | ❌ Offen |
| 11 | WAN / PPPoE | ❌ Offen – Umstieg-Tag |
| 12 | Firmware-Update | ❌ Offen – Umstieg-Tag |
| 13 | Cisco als Switch umkonfigurieren | ❌ Offen – Umstieg-Tag |
| 14 | VPN-Tunnel (mit Mauro) | ❌ Offen |
| 15 | VPN-Monitoring | ❌ Offen – nach Umstieg |
| 16 | FritzBox VPN Port-Forwarding (neu) | ❌ Offen |

## Nächste Session: Donnerstag 16.04.2026

- OPNsense-Backup mitbringen → restliche Static DHCP per Config-Import
- Firewall-Regeln händisch anlegen
- Port-Forwarding händisch anlegen
- Static NAT für Telefonanlage

## Dokumentation

- [`DEC3852_Doku_14-04-2026.md`](DEC3852_Doku_14-04-2026.md) – Vollständige Konfigurationsdoku mit allen Schritten, VLAN-Übersicht, Port-Zuordnungen, extrahierten Portnummern

## Wichtige Erkenntnisse 14.04.2026

- Alle Portnummern aus der Cisco-Config (XML) erfolgreich extrahiert
- **Korrektur:** VPNWG-Ziel ist 192.168.80.48 (nicht 192.168.88.48)
- 21 Static-DHCP-Einträge aus Cisco identifiziert (inkl. MAC-Adressen)
- Cisco-Config ist nicht direkt auf OPNsense importierbar (unterschiedliches XML-Format)

## Beteiligte

| Wer | Rolle |
|-----|-------|
| Inan | DEC3852 Konfiguration |
| Mauro | VPN-Infrastruktur & Config-Export |
| Mathias | Infrastruktur-Fragen Hamburg |
