# Aruba 6300 Überwachung mit Zabbix

Diese Anleitung beschreibt, wie Du einen Aruba 6300 Switch in Zabbix für die Überwachung der MSTP Root-Bridge-Umschaltung und allgemeines Monitoring einbindest.

---

## 1. SNMP auf dem Aruba 6300 konfigurieren

**Auf dem Aruba 6300 (AOS-CX CLI):**

```
sw(config)# snmp-server community public ro
sw(config)# snmp-server host <zabbix_server_ip> traps version 2c public
sw(config)# snmp-server enable
```

Setze `<zabbix_server_ip>` auf die IP des Zabbix Servers und ändere `public` ggf. auf dein SNMP-Community-Passwort.

Optional: Syslog-Meldungen schicken (nur wenn Zabbix Syslog verarbeitet):

```
sw(config)# logging <zabbix_server_ip>
```


---

## 2. Den Switch als SNMP-Host in Zabbix hinzufügen

- Gehe zu **Configuration → Hosts → Create host**
- **IP-Adresse** des Switches eintragen
- **SNMP-Schnittstelle** auswählen
- **Template verknüpfen:**
  - `Template Net Network Generic Device SNMP`
  - Oder ein Aruba-spezifisches Template von [Zabbix Share](https://share.zabbix.com/network_devices/aruba)
- Community-String anpassen (z. B. `public` wie oben)

---

## 3. Überwachung der MSTP Root-Bridge-Umschaltung

### a) Per SNMP-Item:

1. Füge ein neues SNMP-Item hinzu:
   - Typ: `SNMPv2 agent`
   - OID für Root Bridge ID auslesen (z. B. `dot1dStpRootPort`):
     - `1.3.6.1.2.1.17.2.5.0` (klassisches STP; MSTP OID individuell prüfen)
   - Name: z. B. "MSTP Root Bridge ID"
2. Trigger anlegen, der auf Änderung dieses Wertes reagiert.

### b) Per SNMP Trap:

1. Zabbix für das Empfangen von Traps einrichten (snmptrapd installieren & konfigurieren).
2. Traps vom Switch konfigurieren (siehe oben).
3. In Zabbix Items & Trigger für einschlägige Trap-Meldungen anlegen (z. B. "root has changed").

### c) OIDs für MSTP auf der Aruba 6300 ermitteln

- Nutze `snmpwalk`:

  ```
snmpwalk -v2c -c public <switch_ip> 1.3.6.1.4.1
  ```
  Suche nach OIDs wie `hh3cMstpMIB`, `mstpCISTRootId`, o. ä. (Firmware beachten).


---

## 4. Alarmierung/Test

- Teste, ob bei einer bewusst ausgelösten Root-Bridge-Änderung (z. B. Uplink ausstecken) eine Alarmierung erfolgt.
- Passe bei Bedarf die Items/OIDs für den Switch an.

---

## 5. Ressourcen

- [Aruba AOS-CX SNMP Guide](https://www.arubanetworks.com/techdocs/AOS-CX/10.09/html/snmp/index.html)
- [Zabbix SNMP Trap Konfiguration](https://www.zabbix.com/documentation/current/en/manual/config/items/snmptrap)
- [Zabbix Share Aruba Templates](https://share.zabbix.com/network_devices/aruba)

---

Für weitergehende OID-Details nutze einen aktuellen `snmpwalk` des Switches!