# Chat-Protokoll

## Benutzer
Wie kann ich Windows Server auslesen welche patches Kb installiert sind

## Assistent
Du kannst auf einem Windows Server installierte **KB-Patches** auf mehrere Arten auslesen.

### Schnell per PowerShell
```powershell
Get-HotFix | Select-Object HotFixID, Description, InstalledOn
```

Das zeigt dir die installierten Hotfixes inklusive **KB-Nummer**.

Wenn du nur KBs sehen willst:
```powershell
Get-HotFix | Select-Object -ExpandProperty HotFixID
```

### Über WMI / CIM
```powershell
Get-CimInstance Win32_QuickFixEngineering | Select-Object HotFixID, Description, InstalledOn
```

### Über die Eingabeaufforderung
```cmd
wmic qfe list brief /format:table
```

Oder nur die KBs:
```cmd
wmic qfe get HotFixID
```

### Nach bestimmter KB suchen
Zum Beispiel für KB5031364:
```powershell
Get-HotFix -Id KB5031364
```

Oder:
```powershell
Get-CimInstance Win32_QuickFixEngineering | Where-Object {$_.HotFixID -eq "KB5031364"}
```

### In Datei exportieren
```powershell
Get-HotFix | Select-Object HotFixID, Description, InstalledOn | Export-Csv C:\temp\patches.csv -NoTypeInformation -Encoding UTF8
```

### Über die GUI
- **Einstellungen / Systemsteuerung**
- **Programme und Features**
- **Installierte Updates anzeigen**

### Wichtig
`Get-HotFix` bzw. `Win32_QuickFixEngineering` zeigen **nicht immer alle** Updates vollständig an, besonders bei:
- kumulativen Updates
- Servicing Stack Updates
- manchen rollen-/featurebezogenen Updates

Wenn du wirklich eine vollständige Update-Sicht willst, ist zusätzlich oft hilfreich:
```powershell
Get-WindowsPackage -Online
```

Oder mit DISM:
```cmd
dism /online /get-packages
```

Wenn du möchtest, kann ich dir auch ein **fertiges PowerShell-Skript** geben, das:
- alle installierten KBs ausliest,
- nach Datum sortiert,
- und in CSV oder TXT exportiert.

## Benutzer
Fertiges powershell script

## Assistent
Klar — hier ist ein fertiges PowerShell-Skript, das:

- installierte **KBs** ausliest
- nach **Installationsdatum** sortiert
- als **CSV** und **TXT** exportiert
- zusätzlich eine zweite Liste über **DISM/Packages** erzeugt

## Skript
```powershell
# Ordner für Export
$ExportPath = "C:\Temp\PatchReport"

if (-not (Test-Path $ExportPath)) {
    New-Item -Path $ExportPath -ItemType Directory -Force | Out-Null
}

$DateStamp = Get-Date -Format "yyyy-MM-dd_HH-mm-ss"

# Hotfixes / KBs über WMI/CIM auslesen
$Hotfixes = Get-CimInstance Win32_QuickFixEngineering |
    Select-Object @{
        Name = 'ComputerName'
        Expression = { $env:COMPUTERNAME }
    }, @{
        Name = 'KB'
        Expression = { $_.HotFixID }
    }, Description, @{
        Name = 'InstalledOn'
        Expression = {
            try {
                [datetime]$_.InstalledOn
            }
            catch {
                $null
            }
        }
    } |
    Sort-Object InstalledOn -Descending

# CSV Export
$CsvFile = Join-Path $ExportPath "Installed_KBs_$DateStamp.csv"
$Hotfixes | Export-Csv -Path $CsvFile -NoTypeInformation -Encoding UTF8

# TXT Export
$TxtFile = Join-Path $ExportPath "Installed_KBs_$DateStamp.txt"
$Hotfixes | Format-Table -AutoSize | Out-String | Set-Content -Path $TxtFile -Encoding UTF8

# DISM Paketliste exportieren
$DismFile = Join-Path $ExportPath "Installed_Packages_$DateStamp.txt"
dism /online /get-packages > $DismFile

# Ausgabe auf Konsole
Write-Host ""
Write-Host "Patch-Report erstellt:" -ForegroundColor Green
Write-Host "CSV : $CsvFile"
Write-Host "TXT : $TxtFile"
Write-Host "DISM: $DismFile"
Write-Host ""

# Optional: Anzeige in der Konsole
$Hotfixes | Format-Table -AutoSize
```

## Verwendung
1. PowerShell **als Administrator** öffnen
2. Skript z. B. als `Get-InstalledKBs.ps1` speichern
3. Ausführen:
```powershell
.\Get-InstalledKBs.ps1
```

## Nur KB-Liste ohne Export
Falls du eine ganz schlanke Version willst:
```powershell
Get-CimInstance Win32_QuickFixEngineering |
Select-Object HotFixID, Description, InstalledOn |
Sort-Object InstalledOn -Descending
```

## Nur KB-Nummern in TXT
```powershell
Get-CimInstance Win32_QuickFixEngineering |
Select-Object -ExpandProperty HotFixID |
Set-Content C:\Temp\KB-Liste.txt
```

Wenn du willst, kann ich dir als Nächstes noch eine Version bauen für:
- **Remote-Server**
- **mehrere Server aus TXT-Liste**
- **HTML-Report**
- **nur Sicherheitsupdates**
