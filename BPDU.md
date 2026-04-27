# Spanning Tree / BPDU – Unterhaltung (2026-04-26)

**Datum:** 2026-04-26  
**User-Login:** Borstel67

---

## BPDU-Filter – was macht das?

Ein **BPDU-Filter** sorgt dafür, dass ein Switchport **keine BPDUs (Bridge Protocol Data Units)** verarbeitet – je nach Implementierung entweder:

- **BPDUs werden nicht gesendet und/oder nicht empfangen/ausgewertet**, oder
- eingehende BPDUs führen dazu, dass der Port **anders behandelt wird** (Verhalten hersteller-/modusabhängig).

**Wichtig:** BPDUs sind STP/RSTP/MSTP-Steuerpakete. Wenn du sie filterst, “sieht” STP an diesem Port ggf. keine Topologie mehr.

### Typische Wirkung (herstellerabhängig, grob)
- **Port-spezifisches BPDU-Filter (am Interface):** BPDUs werden unterdrückt; STP wirkt dort praktisch nicht mehr → **Loop-Gefahr**, wenn doch ein Switch angeschlossen wird.
- **Global mit PortFast (bei manchen Herstellern):** Port sendet/ignoriert anfangs BPDUs; **wenn BPDUs empfangen werden**, wird das Verhalten ggf. zurückgenommen (Schutzmechanismus – abhängig von Plattform/Version).

---

## BPDU-Filter auf HPE Aruba (ArubaOS-Switch/ProVision)

Auf **HPE ArubaOS-Switch (ProVision)** bedeutet **BPDU-Filter** im Kern:

- Der Port **ignoriert/verwift empfangene BPDUs** (sie werden nicht an STP übergeben).
- Dadurch kann STP **keine Nachbarschaft erkennen** und **nicht korrekt gegen Loops schützen**.

**Folge:** Wenn an so einem Port unerwartet ein Switch/Bridge hängt oder es zu Fehlverkabelung kommt, steigt das **Loop-/Storm-Risiko** deutlich.

### Unterschied zu BPDU-Protection / BPDU-Guard
- **BPDU Guard/Protection:** Wenn der Port eine BPDU sieht, wird er **blockiert/deaktiviert** (je nach Konfig) → schützt vor “Rogue Switch”.
- **BPDU Filter:** BPDUs werden **weggeschluckt** → STP merkt nichts → **Loop-Risiko steigt**.

---

## “Stich”-Switches aus STP ausnehmen – BPDU-Filter?

Wenn Switches als **Stich (nur ein Uplink, keine Redundanz)** betrieben werden, ist BPDU-Filter in der Regel **nicht empfehlenswert**, weil du damit STP-Schutz aushebelst.

- Wenn das Ziel **schnelles Forwarding** für Clients ist: nutze **Admin-Edge/PortFast** (auf Clientports).
- Wenn das Ziel ist, dass am Uplink “kein STP” läuft: BPDU-Filter wäre das Werkzeug, aber mit **hohem Risiko** (Fehlpatching/Loops).

### Wenn BPDU-Filter dennoch eingesetzt wird: Absicherung
Wenn du BPDU-Filter auf Uplinks nutzt, sollten zusätzliche Schutzmechanismen erwogen werden:
- **Loop Protect** (falls verfügbar/aktiviert)
- **Storm Control** (Broadcast/Multicast/Unknown-Unicast Limits)
- Strenge VLAN-Freigaben (nur notwendige VLANs)
- Saubere Doku/Change-Prozesse

---

## “STP anlassen, aber Port als Point-to-Point behandeln” (ArubaOS)

Bei ArubaOS-Switch ist “Point-to-Point” bei RSTP/MSTP normalerweise **automatisch**, wenn der Port **Full-Duplex** ist (typischer Switch-zu-Switch-Link). Häufig gibt es dafür keinen zwingenden manuellen “P2P”-Schalter.

**Prüfen:**
- Duplex/Link:
  - `show interfaces brief`
  - `show interfaces <port>`
- STP-Status:
  - `show spanning-tree`
  - `show spanning-tree <port>` (modell/versionabhängig)

---

## MSTP – Priorität setzen (ArubaOS-Switch)

### MSTP aktivieren
```text
conf t
spanning-tree
spanning-tree mode mstp
```

### Priority für CIST / IST (Instance 0)
```text
spanning-tree priority 4096
```

### Priority pro MST-Instanz (MSTI 1..n)
```text
spanning-tree mst 1 priority 4096
spanning-tree mst 2 priority 8192
```

### Prüfen
```text
show spanning-tree
show spanning-tree mst
```

**Hinweis:** Für MSTP müssen in einer Region übereinstimmen:
- Region-Name
- Revision
- VLAN→Instanz-Zuordnung (Mapping)

---

## MSTP vs STP/RSTP – kurz erklärt

### STP (802.1D)
- Ein Spanning Tree für alles
- Langsame Konvergenz

### RSTP (802.1w)
- Auch “ein Tree”, aber **schneller**
- Oft beste Wahl, wenn Einfachheit wichtig ist oder wenig Bedarf an VLAN-Load-Balancing besteht

### MSTP (802.1s / in 802.1Q)
- Mehrere Instanzen möglich
- VLANs werden Instanzen zugeordnet → **besseres Load-Balancing** bei Redundanz
- Mehr Komplexität (Region/Mapping muss konsistent sein)

---

## Beispiel: 10 VLANs, HP 2920-24G, 3 Cores, redundante Uplinks

**Grundidee:** Mit 2 MST-Instanzen kann man Last über 2 Cores verteilen; der 3. Core ist Fallback.

Beispiel VLAN-Mapping:
- MSTI 1: VLAN 10,20,30,40,50
- MSTI 2: VLAN 60,70,80,90,100

Beispiel Prioritäten:
- Core A: Root für MSTI 1
- Core B: Root für MSTI 2
- Core C: Fallback

---

## Ring-Topologie A–B–C–A

In einem L2-Ring (A–B–C–A) sorgt STP/MSTP dafür, dass pro Instanz **ein Link/Port blockiert** wird, um die Schleife zu verhindern.

Mit **2 Instanzen** kann man erreichen, dass für unterschiedliche VLAN-Gruppen **unterschiedliche Links** blockiert werden (Lastverteilung).

---

## RSTP auf HP/Aruba 2930 aktivieren (ArubaOS-Switch)

### RSTP aktivieren
```text
conf t
spanning-tree
spanning-tree mode rstp
write memory
```

### (Optional) Root-Priorität setzen
Root:
```text
conf t
spanning-tree priority 4096
write memory
```

Backup-Root:
```text
conf t
spanning-tree priority 8192
write memory
```

### (Optional) Admin-Edge / PortFast für Clientports
```text
conf t
spanning-tree <PORTLISTE> admin-edge-port
write memory
```

---

## Root Guard auf “Stich”-Uplinks am Core (Aruba 2930)

Root Guard wird auf Ports gesetzt, die **niemals Root-Port werden sollen** (typisch: Downlinks Richtung Access).

```text
conf t
spanning-tree <PORTLISTE> root-guard
write memory
```

Beispiel:
```text
conf t
spanning-tree 23-24 root-guard
write memory
```

**Prüfen (modell/versionabhängig):**
```text
show spanning-tree
show spanning-tree root-guard
```

**Hinweis:** Root Guard verhindert, dass ein Downstream-Switch Root wird; es ersetzt nicht grundsätzlich ein sauberes STP-Design und schützt nicht vor allen Loop-Szenarien.
