# VPN-Config Review DEC3852 → für Mauro

**Erstellt:** 28.04.2026
**Quelle:** Vergleich `RV345P_configuration_2026-04-14.xml` ↔ `config-merged-20260423.xml`
**Status:** Vorkonfiguration steht, vor Umstieg-Tag bitte folgende Punkte gegenchecken.

---

## 🔴 Bitte verifizieren

### 1. PSKs prüfen
Zwei Tunnel-Paare haben in der OPN-Config jeweils **identische PSKs**:

- **Bürkert** und **OMC** → gleicher PSK
- **Berlin Office** und **Hagebau** → gleicher PSK

Sehr wahrscheinlich Copy-Paste-Fehler. Bitte gegen die echten Werte (T70 / Bürkert-Doku / Hagebau) prüfen und korrigieren.

### 2. Statische Routen fehlen
Cisco hatte für HagebauIT zwei statische Routen, in OPN nicht angelegt:

| Ziel | Next Hop | Interface |
|------|----------|-----------|
| `10.0.8.0/21` | 10.4.250.1 | VLAN90 |
| `10.32.160.0/22` | 10.4.250.1 | VLAN90 |

Ohne diese Routen geht Hagebau nicht, auch wenn der Tunnel up ist.

### 3. Hagebau Phase-2 Range
OPN-Phase2 ist `10.0.8.0/10` (= 10.0.0.0–10.63.255.255, riesig). Vom Cisco 1:1 übernommen — beabsichtigt oder enger fassen?

---

## 🟡 Korrektur empfohlen

### 4. Sonic – Subnetz fehlt
Cisco-IP-Gruppe `sonic` hatte 4 Netze, in OPN nur 3:
- ✅ 10.199.2.0/24, 10.199.21.0/24, 10.199.27.0/24
- ❌ **10.199.16.0/24** fehlt

### 5. Berlin Office – zusätzliches Phase-2
OPN hat zwei P2: `10.10.252.0/24` **und** `10.10.251.0/24`.
Cisco hatte aktiv nur `10.10.252.0/24` (251er war nur in einer ungenutzten IP-Gruppe).

→ Stimmt die WatchGuard-Seite (T70) auf beide Netze? Falls nein: 251er in OPN entfernen oder T70 anpassen.

### 6. DPD nicht aktiviert
Cisco hatte DPD (Action „restart") für Berlin, Sonic, Tyczka aktiv. In OPN bei allen 7 Tunneln deaktiviert (kein `dpd_delay`/`dpd_maxfail` gesetzt).

Empfehlung: überall DPD ein, z.B. delay=10s / maxfail=5.

---

## ❓ Klärungsbedarf

| Thema | Frage |
|-------|-------|
| **BSSWedolo** (62.153.250.118, ikev1) | Im Cisco enabled (Status DOWN). In OPN gar nicht angelegt. Bewusst weggelassen oder vergessen? |
| **Bürkert_2_Adacor** (ikev1) | In beiden disabled. Komplett raus oder noch behalten? |
| **PM_Remote Mobile-Clients** (192.168.81.x) | Cisco-IPsec-Pool entfällt mit Umstieg. Wie connecten Entwickler künftig? Weiter über kleine OPN `192.168.80.59`? WireGuard auf DEC3852 ist angelegt aber disabled. |
