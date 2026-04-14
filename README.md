# Netzwerk-Migration Hamburg

Ablösung des Cisco RV345P durch eine OPNsense DEC3852 als Hauptrouter am Standort Hamburg.

## Status

| Schritt | Status |
|---------|--------|
| Grundkonfiguration (Hostname, DNS, Zeitzone) | ✅ Erledigt |
| VLANs anlegen | ✅ Erledigt |
| VLANs als Interfaces zuweisen + IPs | ✅ Erledigt |
| DHCP-Server pro VLAN | ✅ Erledigt |
| DHCP DNS für VLAN90 nachtragen | ❌ Offen |
| Static DHCP Einträge übernehmen | ❌ Offen (MAC aus ARP-Tabelle holen) |
| Firewall-Regeln pro VLAN | ❌ Offen (Cisco-Regeln dokumentiert) |
| Port-Forwarding übernehmen | ❌ Offen (Cisco-Regeln dokumentiert) |
| Static NAT für Telefonanlage | ❌ Offen (Cisco-Regeln dokumentiert) |
| WAN / PPPoE | ❌ Offen – Umstieg-Tag |
| Firmware-Update | ❌ Offen – Umstieg-Tag |
| Cisco als Switch umkonfigurieren | ❌ Offen – Umstieg-Tag |
| VPN-Tunnel (mit Mauro) | ❌ Offen – Umstieg-Tag |
| Monitoring einrichten | ❌ Offen – nach Umstieg |

## Dokumentation

- [`DEC3852_KONFIGURATION.md`](DEC3852_KONFIGURATION.md) – Alle durchgeführten und geplanten Schritte, VLAN-Übersicht, Port-Zuordnungen, Firewall- und NAT-Regeln vom Cisco

## Beteiligte

| Wer | Rolle |
|-----|-------|
| Inan | DEC3852 Konfiguration |
| Mauro | VPN-Infrastruktur & Config-Export |
| Mathias | Infrastruktur-Fragen Hamburg |
