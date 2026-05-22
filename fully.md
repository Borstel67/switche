User: was muss ich in Fully Kiosk einstellen damit nur der Bildschirmschoner aktiviert wird ohne den Hintergrundprozess zu beenden und per kammera wieder deaktiviert wird

Assistant: Stell **nicht „Screen Off“**, sondern **den Fully-Screensaver** ein. Dann bleibt Fully im Vordergrund aktiv und die Kameraerkennung kann den Screensaver wieder beenden. Laut Fully gilt: Wenn der Bildschirm wirklich ausgeht oder das Gerät schlafen geht, werden auf manchen Geräten App, Kamera-Motion-Detection oder Remote Admin gestoppt; als Workaround empfiehlt Fully ausdrücklich **Screensaver statt Screen Off**. ([fully-kiosk.com](https://www.fully-kiosk.com/en/?utm_source=openai))

**Empfohlene Einstellungen in Fully Kiosk:**

1. **Device Management / Geräteverwaltung**
   - **Keep Screen On = EIN**
   - **Keep Screen On (Advanced) = EIN**, falls andere Apps genutzt werden oder das Gerät trotzdem abdunkelt.  
   Das hält Android davon ab, das Gerät schlafen zu legen. ([fully-kiosk.com](https://www.fully-kiosk.com/en/?utm_source=openai))

2. **Screensaver**
   - **Screensaver Timer** auf die gewünschte Idle-Zeit setzen
   - **Screensaver Playlist** leer lassen, wenn du nur abdunkeln willst
   - **Screensaver Wallpaper URL** optional auf Schwarz setzen, z. B. `fully://color#000000`
   - **Screensaver Brightness** sehr niedrig oder `0`, wenn der Bildschirm praktisch schwarz sein soll  
   Fully beschreibt ausdrücklich, dass du die Playlist leer lassen kannst, wenn du den Screen nur dimmen willst. ([fully-kiosk.com](https://www.fully-kiosk.com/en/?utm_source=openai))

3. **Motion Detection / Bewegungserkennung**
   - **Visual Motion Detection = EIN** für Erkennung per Kamera
   - optional **Face Detection = EIN**, wenn verfügbar
   - **Stop Screensaver on Motion = EIN** bzw. die Bewegungserkennung so nutzen, dass der Screensaver bei Bewegung beendet wird  
   Fully nennt als Funktion, dass bei Bewegung per Kamera der Bildschirm eingeschaltet bzw. der Screensaver gestoppt werden kann. ([fully-kiosk.com](https://www.fully-kiosk.com/en/?utm_source=openai))

4. **Wichtig: Nicht verwenden**
   - **Screen Off Timer = AUS**
   - keine echten Sleep-/Hibernate-Funktionen verwenden  
   Denn Fully weist darauf hin, dass bei ausgeschaltetem Display viele Geräte Prozesse aussetzen oder die App sogar töten; besonders Kamera-Motion-Detection kann dann nicht mehr laufen. ([fully-kiosk.com](https://www.fully-kiosk.com/en/?utm_source=openai))

5. **Falls Bewegungserkennung beim Start/Stop des Screensavers falsch auslöst**
   - **Ignore Motion Detection when Screensaver goes on/off = EIN**  
   Das ist laut Fully genau für Fehltrigger beim Ein-/Ausblenden des Screensavers gedacht. ([fully-kiosk.com](https://www.fully-kiosk.com/en/?utm_source=openai))

**Kurz gesagt:**  
Wenn du willst, dass **nur ein schwarzer „Bildschirmschoner“ aktiv ist**, **Fully aber weiterläuft und per Kamera wieder aufwacht**, dann:
- **Keep Screen On EIN**
- **Screensaver Timer EIN**
- **Screen Off Timer AUS**
- **Visual Motion Detection EIN**
- **Screensaver-Helligkeit sehr niedrig / schwarz**. ([fully-kiosk.com](https://www.fully-kiosk.com/en/?utm_source=openai))

Wenn du möchtest, kann ich dir als Nächstes eine **konkrete Schritt-für-Schritt-Liste auf Deutsch mit den exakten Menüpfaden in Fully** schreiben.

User: {"url":"https://github.com/Borstel67/switche/blob/main/netboxprint.Md"}

User: Speichere den gesamten chat in borstel67/switche unter fully.md