# Jellyfin Service Toolset

Ursprünglich war der Gedanke, meiner Jellyfin Installation, ein Windows System Tray Icon zu spendieren, um den Dienst selbiger steuern zu können.

Und wie es manchmal so ist, kommt eins zum anderen. Die Logs liesen sich immer nur schwer lesen, also baute ich dazu noch ein kleines Tool, welches die Logs farbig darstellen konnte und so Fehler und Warnungen auf einen Blick ersichtlich sind. Aus dem kleinen Logviewer wurde dann die "Console", mit der ich auch gleich den Dienst starten und stoppen konnte. Aber halt, wenn man schon mal dabei ist, kann man ja auch gleich noch ein paar nützliche Informationen einbauen. Und schon gab es den "Statistics" Button, klickt man auf ihn, öffnet sich ein kleines Panel mit Informationen rund um die Jellyfin Installation. Bestehend aus, Version, Dienst Status, CPU/RAM Benutzung, Größe des Data Verzeichnises und die Größen der einzelnen Bibliotheken (Filme, Serien, Music, usw.).

Da es immer etwas umständlich war, dem Dienst die richtigen Berechtigungen zu erteilen, war die nächste Idee gebohren. Mit Hilfe des Konigurators, kann man nun, recht einfach, den Dienst entsprechend konfigurieren. Da es nicht dabei bleiben sollte, kamen noch ein paar Ideen dazu. Warum nicht gleich die ganze Konfiguration für das Toolset abdecken? Gesagt getan, Sprachauswahl für das Toolset, Angabe der Verzeichnisse für die Installation, Angabe des Service Accounts des Dienstes, samt Validierung, Dienst Installation/De-Installation und das SysTray Icon mit der Windows Anmeldung starten.

Aber moment, irgendwas hat noch gefehlt, warum nicht auch gleich auf eine neue Jellyfin Version prüfen? Und wenn wir schon dabei sind, warum auch nicht gleich noch einen Updater dazu erschaffen? Ok, also rundet nun auch noch ein Updater das Paket ab.

Ziel des Ganzen ist es, wie immer, dem Admin so viel Arbeit abzunhemen, wie es geht. Aus diesem Grund, ist das ganze Toolset recht einfach zu benutzen und nach möglichkeit ohne große Interaktion.



## Inhalt

Vorrausetzung Toolset

Installation Jellyfin Server

Update Jellyfin Server

Komponenten des Toolsets



## Vorrausetzung Toolset

Um das Toolset nutzen zu können, gibt es keine speziellen Vorraussetzungen, außer die folgenden:

#### Betriebssystem

getestet wurde unter:

Windows 10 (22H2),

Windows 11 (22H2),

Windows Server 2019,

Windows Server 2022

sollte aber unter jedem Windows mit PowerShell 5.1 laufen.

#### Software

Mal abgeshen von Jellyfin selbst, gibt es keine speziell zu installierende Software. PowerShell 5.1 ist bei jedem Windows ab Windows 10/Windows Server 2019 schon an Bord.

Wichtig ist nur, das einmal der folgende Befehl, in einer administrativen Powershell Sitzung, ausgeführt werden muss. Das hat den Hintergrund, das Windows von Hause aus, das Ausführen von Powershell Scripten auf Windows Clients verbietet und somit auch das Toolset nicht ausgeführt werden kann.

```powershell
 Set-ExecutionPolicy RemoteSigned -Confirm:$false -Force
```

Sollte es dennoch nicht funktionieren, muss noch folgender Befehl ausgeführt werden:

```powershell
Get-ChildItem -LiteralPath "C:\Jellyfin\Admin" -Recurse | Unblock-File -Confirm:$false
```

Das Verzeichnis ist aus dem folgenden Beispiel "Installation Jellyfin Server - Punkt 4"



*Noch eine Anmerkung zu den EXE-Dateien:
Wem es nicht behagt aus dem Internet herunter geladenen EXE-Dateien auszuführen, was ich gut verstehen kann, der kann alternativ auch die RunXXXX.ps1 und die dazugehörigen RunXXXX.cmd Dateien aus diesem Repository verwenden, der Inhalt ist der gleiche, allerdings laufen sie unter dem PowerShell Prozess und nicht unter Ihrem jeweilig eigenen.*



## Installation Jellyfin Server

Die einfachste Variante den Jellyfin Server zu installieren erkläre ich in den folgenden Schritten:

1. Download der Jellyfin Server Dateien
   
   Um das Toolset vernüftig zu nutzen, empfehle ich das Combined Package, das gibts hier im Jellyfin Repo: [https://repo.jellyfin.org/releases/server/windows/versions/stable/combined](https://repo.jellyfin.org/releases/server/windows/versions/stable/combined/)

2. Anlegen eines Verzeichnisses, in dem am Ende, der Server, das Data und das Toolset Verzeichnis liegen
   
   zum Beispiel:
   
   ```powershell
   C:\Jellyfin
   ```
   
   In diesem legen wir gleich noch das ***Data*** Verzeichnis an.

3.  Den Inhalt (ein Verzsichnis) des heruntergeladenen Archives in das gerade erstellte Verzeichnis entpacken und in ***Server*** umbenennen.

4. Das Toolset von hier aus den Releases herunterladen und wieder den Inhalt in unser erstelltes Verzeichnis entpacken.
   
   Jetzt müsste es darin so aussehen:
   
   ```powershell
   C:\Jellyfin\Admin
   C:\Jellyfin\Data
   C:\Jellyfin\Server
   ```

5. Im Verzeichnis Admin nun die "Jellyfin.Config.exe" starten
   
   Nach erfolgtem Start, musst du die folgenden Punkte zwingend ausfüllen:
   
   ***Im Bereich Basics***
   
   Data Verzeichnis: hier "C:\Jellyfin\Data" auswählen
   Server Verzeichnis: hier "C:\Jellyfin\Server" auswählen
   
   ***Im Bereich Service***
   
   Hier hast du die Auswahl zwischen "Lokalem Dienstkonto" oder einem eigenen Dienstkonto.
   
   Da die meisten ihre Mediadaten, wie Filme und Musik auf einem FileShare (z.b. NAS) liegen haben, muss hier ein Benutzer mit Lese- und Schreibrechten (wenn Jellyfin Cover und Co. auf dem NAS speichern soll) eingetragen werden.
   
   Unterstützt werden hier lokale Konten, aber auch ActiveDirectory-Konten.
   
   Solltest du dich für ein eigenes Dienst Konto entscheiden, trage die Kontodaten ein und klicke auf Account überprüfen.
   
   Sollte alles richtig sein, kannst du nun auf "Speichern klicken."
   
   Jetzt fehlt nur noch der klick auf den grünen Button "Installieren".
   
   Nachdem der Dienst installiert wurde, hast du gleich die Möglichkeit den Konfigurationsassistenten von Jellyfin selbst zu starten. Falls du dies später tun willst, öffne später einfach deinen Browser und gehe zu dieser URL:
   
   http://127.0.0.1:8096/web/index.html#!/wizardstart.html

6. Fertig, das wars schon.
   
   Optional kannst du im Konfigurator noch angeben, ob automatisch nach Updates gesucht werden soll, ob das SysTray Icon automatisch bei jedem Anmelden gestartet werden soll und den Pfad für den installierten Jellyfin Client, dazu aber mehr weiter unten.
   
   

*Der Vorteil dieser Installationsvariante ist, man kann das Verzeichnis "C:\Jellyfin", jederzeit verschieben oder umbenennen. Einzige Vorraussetzung ist, man deinstalliert den Dienst vorher, was aber über den Konfigurator problemlos, schnell und ohne Datenverlust möglich ist und beendet alle Toolsetkomponentent. Nach dem Verschieben oder umbenennen, geht man einfach wie in Punkt 5 vor, allerdings braucht man den Konfigurationassistenten nicht noch einmal durchlaufen, sollte dieser bereits vorher einmal ausgeführt worden sein.*



## Update Jellyfin Server

Das Update ist recht einfach, es gibt hier zwei Möglichkeiten:

1. Du startest im "C:\Jellyfin\Admin" Verzeichnis die "Jellyfin.Update.exe" und klickst auf "Update installieren", wenn ein neue Version verfügbar ist.

2. Du aktivierst im "Konfigurator" einfach "auf Updates prüfen". So kannst du dann direkt über das SysTray Menü den Updater starten, sobald eine neue Version vorliegt.
   
   

## Screenshot

<img src="https://raw.githubusercontent.com/GingerBreadInc/Jellyfin-Service-SysTray/main/Images/Overview.png" title="" alt="" data-align="center">

## Komponenten

An dieser Stelle gehe ich etwas genauer auf die einzelnen Komponenten ein

### SysTray Icon with Menu and Balloon Tips

##### SysTray Icon with Menu

![](https://raw.githubusercontent.com/GingerBreadInc/Jellyfin-Service-SysTray/main/Images/SysTray.png)

##### 

##### Balloon Tips

![](https://raw.githubusercontent.com/GingerBreadInc/Jellyfin-Service-SysTray/main/Images/SysTray_balloon.png)



### Service Configuration

![](https://raw.githubusercontent.com/GingerBreadInc/Jellyfin-Service-SysTray/main/Images/Config.png)



### Console

![](https://raw.githubusercontent.com/GingerBreadInc/Jellyfin-Service-SysTray/main/Images/Console.png)

### Header 3

#### Header 4

##### Header 5

###### Header 6
