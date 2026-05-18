# NetBox Geräteschnittstellen drucken

## Frage
Wie kann ich aus NetBox die Geräte Schnittstellen drucken?

## Antwort
Du kannst die **Geräteschnittstellen aus NetBox** auf ein paar praktikable Arten „drucken“ — je nachdem, ob du wirklich auf Papier willst oder eher eine **PDF-/CSV-Liste** brauchst.

### Möglichkeiten

1. **Direkt aus der Geräteseite im Browser drucken**
   - Gerät in NetBox öffnen
   - Zum Bereich **Interfaces / Schnittstellen** gehen
   - Browser-Druckfunktion nutzen:
     - **Windows/Linux:** `Strg + P`
     - **Mac:** `Cmd + P`
   - Am besten als **PDF speichern**, weil das Layout im Druck oft sauberer ist als auf Papier.

2. **Über Export / API**
   Wenn du viele Geräte oder saubere Listen brauchst, ist das meist besser:
   - Interfaces über die **NetBox-API** abrufen
   - Ergebnis als **CSV** oder **HTML/PDF** aufbereiten
   - Dann drucken

3. **Benutzerdefinierter Report / Script**
   Wenn du regelmäßig Interface-Listen drucken willst:
   - ein **Custom Script** oder **Report** in NetBox bauen
   - Ausgabe z. B. als tabellarische HTML-Seite
   - diese dann drucken

### Praktischer Weg mit Export
Wenn du pro Gerät eine Liste willst, könntest du die Interfaces per API holen, z. B. nach Gerät filtern, und dann in Excel oder LibreOffice öffnen und drucken.

Typischer API-Endpunkt:
```bash
/api/dcim/interfaces/?device_id=<ID>
```

Oder nach Gerätename filtern, je nach NetBox-Version/Filter:
```bash
/api/dcim/interfaces/?device=<geraetename>
```

### Wenn du es schnell und ohne Basteln willst
Der einfachste Weg ist:
- Gerät öffnen
- Interfaces anzeigen
- Browser auf **Druckansicht** oder **Als PDF speichern**

### Wenn du es ordentlich für mehrere Geräte willst
Dann empfehle ich:
- **CSV/API-Export**
- anschließend Formatierung in Excel
- dann drucken

---

## Druckfreundliche HTML-Vorlage

```html
<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8">
  <title>NetBox Geräteschnittstellen</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 20px;
      color: #000;
    }

    h1, h2 {
      margin-bottom: 5px;
    }

    .device-block {
      margin-bottom: 40px;
      page-break-inside: avoid;
    }

    .meta {
      margin-bottom: 10px;
      font-size: 14px;
    }

    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 10px;
      font-size: 13px;
    }

    th, td {
      border: 1px solid #333;
      padding: 6px 8px;
      text-align: left;
      vertical-align: top;
    }

    th {
      background: #eaeaea;
    }

    @media print {
      body {
        margin: 10mm;
      }

      .device-block {
        page-break-after: always;
      }

      .device-block:last-child {
        page-break-after: auto;
      }
    }
  </style>
</head>
<body>

  <h1>Geräteschnittstellen</h1>
  <p>Export aus NetBox</p>

  <div class="device-block">
    <h2>Device: sw-core-01</h2>
    <div class="meta">
      Standort: RZ1<br>
      Rolle: Core Switch<br>
      Hersteller: Cisco
    </div>

    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Typ</th>
          <th>Status</th>
          <th>Beschreibung</th>
          <th>VLANs / IPs</th>
          <th>Kabel / Verbindung</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td>GigabitEthernet1/0/1</td>
          <td>1000BASE-T</td>
          <td>Aktiv</td>
          <td>Uplink zu fw-01</td>
          <td>10.0.0.1/30</td>
          <td>Verbunden</td>
        </tr>
        <tr>
          <td>GigabitEthernet1/0/2</td>
          <td>1000BASE-T</td>
          <td>Aktiv</td>
          <td>Server-Netz</td>
          <td>VLAN 20</td>
          <td>Verbunden</td>
        </tr>
        <tr>
          <td>GigabitEthernet1/0/24</td>
          <td>1000BASE-T</td>
          <td>Inaktiv</td>
          <td>Reserve</td>
          <td>-</td>
          <td>Nicht verbunden</td>
        </tr>
      </tbody>
    </table>
  </div>

</body>
</html>
```

---

## Python-Skript für NetBox zu HTML

```python
import requests
import sys
from html import escape

NETBOX_URL = "https://deine-netbox-url"
API_TOKEN = "DEIN_API_TOKEN"
DEVICE_NAME = "sw-core-01"

HEADERS = {
    "Authorization": f"Token {API_TOKEN}",
    "Accept": "application/json",
}

def get_device(device_name):
    url = f"{NETBOX_URL}/api/dcim/devices/?name={device_name}"
    r = requests.get(url, headers=HEADERS)
    r.raise_for_status()
    data = r.json()
    results = data.get("results", [])
    if not results:
        raise Exception(f"Gerät nicht gefunden: {device_name}")
    return results[0]

def get_interfaces(device_id):
    url = f"{NETBOX_URL}/api/dcim/interfaces/?device_id={device_id}&limit=200"
    r = requests.get(url, headers=HEADERS)
    r.raise_for_status()
    data = r.json()
    return data.get("results", [])

def format_connected(interface):
    connected = interface.get("connected_endpoints")
    if connected:
        names = []
        for ep_group in connected:
            if isinstance(ep_group, dict):
                name = ep_group.get("name") or ep_group.get("display")
                if name:
                    names.append(name)
        if names:
            return ", ".join(names)
    return "Nicht verbunden"

def format_ips(interface):
    ip_list = []

    if interface.get("untagged_vlan"):
        vlan = interface["untagged_vlan"]
        ip_list.append(f"Untagged VLAN: {vlan.get('vid', '-')}")
    tagged_vlans = interface.get("tagged_vlans") or []
    if tagged_vlans:
        ip_list.append("Tagged VLANs: " + ", ".join(str(v.get("vid", "-")) for v in tagged_vlans))

    return "<br>".join(escape(x) for x in ip_list) if ip_list else "-"

def render_html(device, interfaces):
    device_name = escape(device.get("name", "-"))
    site = escape((device.get("site") or {}).get("name", "-"))
    role = escape((device.get("role") or {}).get("name", "-"))
    manufacturer = escape((((device.get("device_type") or {}).get("manufacturer")) or {}).get("name", "-"))

    rows = []
    for iface in interfaces:
        name = escape(iface.get("name", "-"))
        iface_type = escape(iface.get("type", {}).get("label", str(iface.get("type", {}).get("value", "-"))) if isinstance(iface.get("type"), dict) else str(iface.get("type", "-")))
        enabled = iface.get("enabled", False)
        status = "Aktiv" if enabled else "Inaktiv"
        description = escape(iface.get("description") or "-")
        vlans_ips = format_ips(iface)
        connected = escape(format_connected(iface))

        rows.append(f"""
        <tr>
          <td>{name}</td>
          <td>{iface_type}</td>
          <td>{status}</td>
          <td>{description}</td>
          <td>{vlans_ips}</td>
          <td>{connected}</td>
        </tr>
        """)

    html = f"""<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8">
  <title>NetBox Geräteschnittstellen</title>
  <style>
    body {{
      font-family: Arial, sans-serif;
      margin: 20px;
      color: #000;
    }}
    h1, h2 {{
      margin-bottom: 5px;
    }}
    .device-block {{
      margin-bottom: 40px;
      page-break-inside: avoid;
    }}
    .meta {{
      margin-bottom: 10px;
      font-size: 14px;
    }}
    table {{
      width: 100%;
      border-collapse: collapse;
      margin-top: 10px;
      font-size: 13px;
    }}
    th, td {{
      border: 1px solid #333;
      padding: 6px 8px;
      text-align: left;
      vertical-align: top;
    }}
    th {{
      background: #eaeaea;
    }}
    @media print {{
      body {{
        margin: 10mm;
      }}
    }}
  </style>
</head>
<body>
  <h1>Geräteschnittstellen</h1>
  <p>Export aus NetBox</p>

  <div class="device-block">
    <h2>Device: {device_name}</h2>
    <div class="meta">
      Standort: {site}<br>
      Rolle: {role}<br>
      Hersteller: {manufacturer}
    </div>

    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Typ</th>
          <th>Status</th>
          <th>Beschreibung</th>
          <th>VLANs / IPs</th>
          <th>Kabel / Verbindung</th>
        </tr>
      </thead>
      <tbody>
        {''.join(rows)}
      </tbody>
    </table>
  </div>
</body>
</html>
"""
    return html

def main():
    device_name = DEVICE_NAME
    if len(sys.argv) > 1:
        device_name = sys.argv[1]

    device = get_device(device_name)
    interfaces = get_interfaces(device["id"])
    html = render_html(device, interfaces)

    output_file = f"{device_name}_interfaces.html"
    with open(output_file, "w", encoding="utf-8") as f:
        f.write(html)

    print(f"HTML-Datei erzeugt: {output_file}")

if __name__ == "__main__":
    main()
```
