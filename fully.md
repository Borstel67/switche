# Fully Kiosk: Bildschirmschoner statt Bildschirm-Aus

## Frage
Was muss ich in Fully Kiosk einstellen, damit nur der Bildschirmschoner aktiviert wird, ohne den Hintergrundprozess zu beenden, und per Kamera wieder deaktiviert wird?

## Antwort
Stell **nicht „Screen Off“**, sondern den **Fully-Screensaver** ein. So bleibt Fully aktiv im Vordergrund und die Kameraerkennung kann den Bildschirmschoner wieder beenden.

Laut Fully gilt: Wenn der Bildschirm wirklich ausgeschaltet wird oder das Gerät in den Sleep-Modus geht, können je nach Gerät App, Kamera-Motion-Detection oder Remote-Admin gestoppt werden. Als Workaround empfiehlt Fully ausdrücklich, den **Screensaver statt Screen Off** zu verwenden.

## Empfohlene Einstellungen in Fully Kiosk

### 1. Device Management / Geräteverwaltung
- **Keep Screen On = EIN**
- **Keep Screen On (Advanced) = EIN**, falls andere Apps genutzt werden oder das Gerät trotzdem abdunkelt

Das verhindert, dass Android das Gerät schlafen legt.

### 2. Screensaver
- **Screensaver Timer** auf die gewünschte Inaktivitätszeit setzen
- **Screensaver Playlist** leer lassen, wenn du nur abdunkeln willst
- **Screensaver Wallpaper URL** optional auf Schwarz setzen, z. B. `fully://color#000000`
- **Screensaver Brightness** sehr niedrig oder `0`, wenn der Bildschirm praktisch schwarz sein soll

Fully beschreibt ausdrücklich, dass die Playlist leer bleiben kann, wenn der Bildschirm nur gedimmt werden soll.

### 3. Motion Detection / Bewegungserkennung
- **Visual Motion Detection = EIN** für Bewegungserkennung per Kamera
- optional **Face Detection = EIN**, wenn verfügbar
- **Stop Screensaver on Motion = EIN** bzw. die Bewegungserkennung so konfigurieren, dass der Screensaver bei Bewegung beendet wird

So kann der Bildschirmschoner durch erkannte Bewegung über die Kamera wieder deaktiviert werden.

### 4. Wichtig: Nicht verwenden
- **Screen Off Timer = AUS**
- keine echten Sleep-/Hibernate-Funktionen verwenden

Denn bei ausgeschaltetem Display setzen viele Geräte Prozesse aus oder beenden die App ganz. Besonders die Kamera-Motion-Detection kann dann nicht mehr zuverlässig laufen.

### 5. Falls Bewegungserkennung beim Start/Stop des Screensavers falsch auslöst
- **Ignore Motion Detection when Screensaver goes on/off = EIN**

Diese Option ist genau dafür gedacht, Fehltrigger beim Ein- oder Ausblenden des Screensavers zu vermeiden.

## Kurzfassung
Wenn du möchtest, dass **nur ein schwarzer Bildschirmschoner aktiv ist**, **Fully aber weiterläuft und per Kamera wieder aufwacht**, dann stelle Folgendes ein:

- **Keep Screen On = EIN**
- **Screensaver Timer = EIN**
- **Screen Off Timer = AUS**
- **Visual Motion Detection = EIN**
- **Screensaver Brightness = sehr niedrig / schwarz**

## Notiz
Ursprünglich wurde der Chat als Rohprotokoll gespeichert und anschließend in dieses lesbare Markdown-Format umgewandelt.