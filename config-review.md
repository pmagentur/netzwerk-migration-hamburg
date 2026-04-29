# VPN-Config Review DEC3852 

**Erstellt:** 28.04.2026
**Quelle:** Vergleich `RV345P_configuration_2026-04-14.xml` ↔ `config-merged-20260423.xml`
**Status:** Vorkonfiguration steht, vor Umstieg-Tag bitte folgende Punkte gegenchecken.

---

## 🔴 Bitte verifizieren

### 1. PSKs prüfen
~~Zwei Tunnel-Paare haben in der OPN-Config jeweils **identische PSKs**~~ → Copy-Paste-Fehler bestätigt und teilweise behoben.

| Tunnel | Status | Anmerkung |
|--------|--------|-----------|
| Bürkert | ✅ Korrekt | Bestätigt |
| OMC | ✅ Korrigiert | PSK in `config-merged-20260423.xml` aktualisiert |
| Berlin Office | ✅ Korrekt | Bestätigt |
| **Hagebau** | ✅ Korrigiert | PSK in `config-merged-20260423.xml` aktualisiert (29.04.2026) |

### 2. Statische Routen fehlen
~~Cisco hatte für HagebauIT zwei statische Routen, in OPN nicht angelegt~~ → **Nicht nötig.**

Beide Netze (`10.0.8.0/21`, `10.32.160.0/22`) liegen innerhalb der Hagebau-Phase-2 `10.0.0.0/10`. OPNsense/strongSwan installiert die Kernel-XFRM-Policies automatisch beim Tunnel-Aufbau — im Gegensatz zum Cisco RV345P, der explizite statische Routen benötigte. ✅

### 3. Hagebau Phase-2 Range
~~OPN-Phase2 ist `10.0.8.0/10` (= 10.0.0.0–10.63.255.255, riesig). Vom Cisco 1:1 übernommen — beabsichtigt oder enger fassen?~~ → **So lassen, aber Optimierung möglich.**

Der große Range war ein Workaround für den Cisco RV345P, der keine mehreren Phase-2-Selektoren pro Tunnel unterstützt. OPNsense unterstützt mehrere Phase-2-Einträge — nach dem Umstieg könnte man mit Hagebau IT abstimmen, ob deren Seite auf präzisere Netze umgestellt werden kann. Für den Umstieg-Tag so lassen. ✅

---

## 🟡 Korrektur empfohlen

### 4. Sonic – Subnetz fehlt ✅ OK
Cisco-IP-Gruppe `sonic` hatte 4 Netze, in OPN nur 3:
- ✅ 10.199.2.0/24, 10.199.21.0/24, 10.199.27.0/24
- ❌ **10.199.16.0/24** fehlt

Geklärt (29.04.2026): Die 3 Netze in OPNsense sind korrekt (die 4 Cisco-Netze stimmten nicht mit der Realität überein). Auf dem Cisco lief de facto nur `10.199.27.0/24` aktiv (Hardware-Limitierung: nur eine Phase-2 pro Tunnel möglich). In OPNsense: Phase-1 und Phase-2 für `.2.0/24` und `.21.0/24` bleiben **disabled** bis Sonic IT bestätigt — nur `10.199.27.0/24` ist aktiv konfiguriert. Phase-1 (Tunnel) bleibt vorerst deaktiviert bis zum Umstieg-Tag.

### 5. Berlin Office – zusätzliches Phase-2 ✅ OK
OPN hat zwei P2: `10.10.252.0/24` **und** `10.10.251.0/24`.
Cisco hatte aktiv nur `10.10.252.0/24` (251er war nur in einer ungenutzten IP-Gruppe).

Geklärt (29.04.2026): T70 wird von uns kontrolliert. Die `/251` kann auf T70-Seite aktiviert werden sobald der neue OPNsense live ist. Beide Phase-2-Einträge in OPN so lassen.

### 6. DPD nicht aktiviert ✅ Erledigt
~~Cisco hatte DPD nur für Berlin, Sonic, Tyczka aktiv. In OPN bei allen 7 Tunneln deaktiviert.~~

DPD auf allen 7 Tunneln aktiviert (29.04.2026): `dpd_delay=10` / `dpd_maxfail=5`.

---

## ❓ Klärungsbedarf

| Thema | Status |
|-------|--------|
| **BSSWedolo** (62.153.250.118, ikev2) | ✅ In OPNsense angelegt, disabled. PSK unbekannt — Platzhalter `PLACEHOLDER_PSK_UNBEKANNT` gesetzt. Vor Aktivierung echten PSK eintragen. |
| **Bürkert_2_Adacor** (ikev1, 7× Phase-2) | ✅ Bereits in OPNsense vorhanden, alle Phase-2 disabled. Cisco konnte nur 1 Phase-2 pro Tunnel → war nie aktiv. Aktuell auf T70 Berlin terminiert. Aktivierung auf DEC3852 zusammen mit Adacor IT nach dem Umstieg. |
| **PM_Remote Mobile-Clients** | ⚠️ Offen. Aktuell nutzen viele OpenVPN auf der kleinen OPN. Vor Umstieg klären: wer nutzt noch `vpnc`? (Cisco-Client, nicht kompatibel mit OPN). Mittelfristig: Migration auf WireGuard (bereits auf DEC3852 angelegt, disabled). |
