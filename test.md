# Jellyfin Service Toolkit

Ursprünglich war der Gedanke, meiner Jellyfin Installation, ein Windows System Tray Icon zu spendieren, um den Dienst selbiger steuern zu können.

Und wie es manchmal so ist, kommt eins zum anderen. Die Logs liesen sich immer nur schwer lesen, also baute ich dazu noch ein kleines Tool, welches die Logs farbig darstellen konnte und so Fehler und Warnungen auf einen Blick ersichtlich sind. Aus dem kleinen Logviewer wurde dann die "Console", mit der ich auch gleich den Dienst starten und stoppen konnte. Aber halt, wenn man schon mal dabei ist, kann man ja auch gleich noch ein paar nützliche Informationen einbauen. Und schon gab es den "Statistics" Button, klickt man auf ihn, öffnet sich ein kleines Panel mit Informationen rund um die Jellyfin Installation. Bestehend aus, Version, Dienst Status, CPU/RAM Benutzung, Größe des Data Verzeichnises und die Größen der einzelnen Bibliotheken (Filme, Serien, Music, usw.).

Da es immer etwas umständlich war, dem Dienst die richtigen Berechtigungen zu erteilen, war die nächste Idee gebohren. Mit Hilfe des Konigurators, kann man nun, recht einfach, den Dienst entsprechend konfigurieren. Da es nicht dabei bleiben sollte, kamen noch ein paar Ideen dazu. Warum nicht gleich die ganze Konfiguration für das Toolkit abdecken? Gesagt getan, Sprachauswahl für das Toolkit, Angabe der Verzeichnisse für die Installation, Angabe des Service Accounts des Dienstes, samt Validierung, Dienst Installation/De-Installation und das SysTray Icon mit der Windows Anmeldung starten.

Aber moment, irgendwas hat noch gefehlt, warum nicht auch gleich auf eine neue Jellyfin Version prüfen? Und wenn wir schon dabei sind, warum auch nicht gleich noch einen Updater dazu erschaffen? Ok, also rundet nun auch noch ein Updater das Paket ab.

Ziel des Ganzen ist es, wie immer, dem Admin so viel Arbeit abzunhemen, wie es geht und dem nicht ganz versierten Benutzer die Kongiuration zu erleichtern. Aus diesem Grund, ist das ganze Toolkit recht einfach zu benutzen und nach möglichkeit ohne große Interaktion.

Aber: Es wird keine Haftung für jedweder Art von Schäden Übernommen, ob seelig, körperlich oder materiell.

Da das ganze ein reines Freizeit-Projekt ist, stelle ich das Toolkit, so wie es ist zur Verfügung, ohne Garantie auf Erweiterungen oder Verbesserungen, bin aber immer für konstruktive Kritik oder Verbesserungsvorschläge offen.

So genug der vielen Worte, nun zum Eigentlichen. ;-)



## Inhalt

Vorrausetzung Toolkit

Installation Jellyfin Server

Update Jellyfin Server

Sprachen

Komponenten des Toolkit

Screenshots

## Vorrausetzung Toolkit

Um das Toolkit nutzen zu können, gibt es keine speziellen Vorraussetzungen, außer die folgenden:

#### Betriebssystem

getestet wurde unter:

Windows 10 (22H2),

Windows 11 (22H2),

Windows Server 2019,

Windows Server 2022

sollte aber unter jedem Windows mit PowerShell 5.1 laufen.

#### Software

Mal abgeshen von Jellyfin selbst, gibt es keine speziell zu installierende Software.

Das gesamte Toolkit habe ich in Powershell geschrieben und die Kompatibilität veruscht so groß wie Möglich zu halten.

Aber das Gute: PowerShell 5.1 ist bei jedem Windows ab Windows 10/Windows Server 2019 schon an Bord.

***Wichtig ist nur, das einmal der folgende Befehl, in einer administrativen Powershell Sitzung, ausgeführt werden muss.***

Das hat den Hintergrund, das Windows von Hause aus, das Ausführen von Powershell Scripten auf Windows Clients verbietet und somit auch das Toolkit nicht ausgeführt werden kann.

```powershell
 Set-ExecutionPolicy RemoteSigned -Confirm:$false -Force
```

Sollte es dennoch nicht funktionieren, muss noch folgender Befehl ausgeführt werden:

```powershell
Get-ChildItem -LiteralPath "C:\Jellyfin\Admin" -Recurse | Unblock-File -Confirm:$false
```

Das Verzeichnis ist aus dem folgenden Beispiel entnommen: "Installation Jellyfin Server - Punkt 5"



#### Windows UAC / Administrative Rechte

Das gesamte Toolkit ist so aufgebaut, das keine administrativen Rechte nötig sind, mit zwei Ausnahmen:

1. Die Installation des Dienstes

2. Das Entfernen des Dienstes

Alles andere, wie zum Beispiel das Starten und Stoppen des Dienstes, funktioniert im jeweiligen Userkontext (des Benutzers, welcher den Dienst Installiert hat), da der "Konfigurator" die benötigten Rechte direkt auf den Dienst setzt.



*Noch eine Anmerkung zu den EXE-Dateien:
Wem es nicht behagt aus dem Internet herunter geladenen EXE-Dateien auszuführen, was ich gut verstehen kann, der kann alternativ auch die RunXXXX.ps1 und die dazugehörigen RunXXXX.cmd Dateien aus diesem Repository verwenden, der Inhalt ist der gleiche, allerdings laufen sie unter dem PowerShell Prozess und nicht unter Ihrem jeweilig eigenen.*



#### Service Account

Ein großes Rätsel ist immer wieder, wie bekommt man den Jellyfin-Dienst dazu auf ein NAS zuzugreifen und auch wirklich lesen und schreiben zu drüfen?

Ich habe dazu einiges gelesen und sehr viel digitales stirnrunseln gesehen. Dabei ist die Lösung recht einfach. Es gibt zwei Varianten auf die ich jetzt kurz eingehe.

**ActiveDirectory**

Zum einen gibt es die Möglichkeit, sein NAS in ein Active Directory einzubinden (für Heimanwender ist das aber eher seltener der Fall). Das ist in meinen Augen die einfachste Variante. Hier muss lediglich ein User im AD angelegt werden, welcher auf dem NAS die entsprechenden Berechtigungen erhält. 

1. User im AD anlegen

2. Berechtigungen für den User auf der Freigabe erteilen

3. Den AD User im "Konfigurator" angeben.

**Lokaler User**

Für Heimanwender ist ein lokaler Windows Benutzer allerdings die einzige Möglichkeit, um mit geringem Aufwand an seine mediale Sammlung zu gelangen. Deswegen hier eine kurze Anleitung, welche in Verbindung mit dem "Konfigurator", recht gut funktioniert. (Getestet mit Synology und TrueNAS)

1. Auf dem NAS einen Benutzer anlegen, z.b. "JellyfinMedia"

2. Auf dem NAS diesen Benutzer entsprechend auf der Freigabe berechtigen

3. Im Windows einen Benutzer anlegen, der den selben Benutzernamen und das selbe Passwort hat.
   
   *Hier ist wichtig zu erwähnen, das dieser Benutzer ***nicht*** in die Gruppe "Lokale Administratoren" oder eine andere Gruppe aufgenommen werden muss, um die richtigen einzelnen Berechtigungen kümmert sich auch hier der "Konfigurator".* 

4. Dieser Benutzer wird dann im "Konfigurator" angegeben.



Ich habe recht oft gelesen, man solle doch das Lokale System Konto benutzen und auf dem NAS das Computerobject berechtigen. Prinzipiell mag das funktionieren, aber aus Sicherheitsgründen würde ich davon Abstand nehmen, genau wie einem Dienstkonto-Benutzer Administrator-Rechte zu verleihen. Man lässt ja auch nicht alle Türen und Fenster offen stehen, nur damit der Briefträger die Post im Urlaub auf den Esszimmertisch legen kann. ;-)

## Installation Jellyfin Server

Die einfachste Variante den Jellyfin Server zu installieren erkläre ich in den folgenden Schritten:

1. Download der Jellyfin Server Dateien
   
   Um das Toolkit vernüftig zu nutzen, empfehle ich das Combined Package, das gibts hier im Jellyfin Repo: [https://repo.jellyfin.org/releases/server/windows/versions/stable/combined](https://repo.jellyfin.org/releases/server/windows/versions/stable/combined/)

2. Anlegen eines Verzeichnisses, in dem am Ende, das ***Server***, das ***Data*** und das ***Toolkit*** Verzeichnis liegen
   
   zum Beispiel:
   
   ```powershell
   C:\Jellyfin
   ```

3. In diesem legen wir gleich noch das ***Data*** Verzeichnis an.

4. Den Inhalt (ein Verzsichnis), des heruntergeladenen Archivs, in das gerade erstellte Verzeichnis entpacken und in ***Server*** umbenennen.

5. Das Toolkit von hier aus den Releases herunterladen und wieder den Inhalt in unser erstelltes Verzeichnis entpacken.
   
   Jetzt müsste es darin so aussehen:
   
   ```powershell
   C:\Jellyfin\Admin
   C:\Jellyfin\Data
   C:\Jellyfin\Server
   ```

6. Im Verzeichnis Admin nun die "Jellyfin.Config.exe" starten
   
   Nach erfolgtem Start, musst du die folgenden Punkte zwingend ausfüllen:
   
   ***Im Bereich Basics***
   
   Data Verzeichnis: hier z.b. "C:\Jellyfin\Data" auswählen
   
   Server Verzeichnis: hier z.b. "C:\Jellyfin\Server" auswählen
   
   ***Im Bereich Service***
   
   Hier hast du die Auswahl zwischen "Lokalem Dienstkonto" oder einem eigenen Dienstkonto.
   
   Da die meisten ihre Mediadaten, wie Filme und Musik auf einem FileShare (z.b. NAS) liegen haben, muss hier ein Benutzer mit Lese- und Schreibrechten (wenn Jellyfin Cover und Co. auf dem NAS speichern soll) eingetragen werden.
   
   Unterstützt werden hier lokale Konten, aber auch ActiveDirectory-Konten.
   
   Solltest du dich für ein eigenes Dienst Konto entscheiden, trage die Kontodaten ein und klicke auf Account überprüfen. (Vorraussetzung für das Dienstkonto siehe oben)
   
   Sollte alles richtig sein, kannst du nun auf "Speichern klicken."
   
   Jetzt fehlt nur noch der klick auf den grünen Button "Installieren".
   
   *Es erscheint die UAC für einen Powershell Prozess, das ist in Ordnung, denn für das Installieren des Dienstes sind Admin Rechte erforderlich.*
   
   Nachdem der Dienst installiert wurde, hast du gleich die Möglichkeit den Konfigurationsassistenten von Jellyfin selbst zu starten. Falls du dies später tun willst, öffne später einfach deinen Browser und gehe zu dieser URL:
   
   http://127.0.0.1:8096/web/index.html#!/wizardstart.html

7. Fertig, das wars schon.
   
   Optional kannst du im Konfigurator noch angeben, ob automatisch nach Updates gesucht werden soll, ob das SysTray Icon automatisch bei jedem Anmelden gestartet werden soll und den Pfad für den installierten Jellyfin Client, dazu aber mehr weiter unten.
   
   

*Der Vorteil dieser Installationsvariante ist, man kann das Verzeichnis "C:\Jellyfin", jederzeit verschieben oder umbenennen. Einzige Vorraussetzung ist, man deinstalliert den Dienst vorher, was aber über den Konfigurator problemlos, schnell und ohne Datenverlust möglich ist und beendet alle Toolkitkomponentent (SysTray, Console, Configurator, Updater). Nach dem Verschieben oder umbenennen, geht man einfach wie in Punkt 5 vor, allerdings braucht man den Konfigurationassistenten nicht noch einmal durchlaufen, sollte dieser bereits vorher einmal ausgeführt worden sein.*



## Update Jellyfin Server

Das Update ist recht einfach, es gibt hier drei Möglichkeiten:

1. Möglichkeit: Du gehst wie oben im "Installation Jellyfin Server" - Punkt 1 und 3 vor und ersetzt das aktuelle Server Verzeichnis.

2. Möglichkeit: Du startest im "C:\Jellyfin\Admin" Verzeichnis die "Jellyfin.Update.exe" und klickst auf "Update installieren", wenn eine neue Version verfügbar ist.

3. Möglichkeit: Du aktivierst im "Konfigurator" einfach "auf Updates prüfen". So kannst du dann direkt über das SysTray Menü den Updater starten, sobald eine neue Version vorliegt.
   
   

## Sprachen

Das Toolkit kann in verschiedene Sprachen übersetzt werden, aktuell unterstützt werden Deutsch und Englisch.

Die Erweiterung ist ebenfalls recht einfach, dazu muss lediglich die Datei "Languages.ps1" geöffnet und bearteitet werden.

Und das geht so:

1. Die Datei "C:\Jellyfin\Admin\Core\Languages.ps1" in einem passenden Editor (z.b. Notepad++) öffen

2. einen bereits vorhandenen Sprachblock kopieren und am Ende einsetzen

3. die einzelnen Variablen enstprechend ausfüllen
   
   Wichtig ist jedoch, das der passende Language Code angegeben wird (Beispiele für Deutsch und Englisch):
   
   ```powershell
   # Englisch:
   $Language = "en-US"
   
   # German:
   $Language = "de-DE"
   ```

4. Noch eine Besonderheit ist die Angabe der "Performance Counter Namen" für "Prozess" und "Prozessorzeit", diese sind von der Betriebssystem-Sprache abhängig (Warum auch immer sich Microsoft dafür entschieden hat).
   
   Hier die Beispiele für Deutsch und Englisch:
   
   ```powershell
   # English
   $Texts[$Language]["ConfigCPUTimeString"] = "% Processor Time"
   $Texts[$Language]["ConfigProcessString"] = "Process"
   
   # German
   $Texts[$Language]["ConfigCPUTimeString"] = "Prozessorzeit (%)"
   $Texts[$Language]["ConfigProcessString"] = "Prozess"
   ```

5. Datei speichern, die Änderungen werden sofort wirksam.

6. Die Sprache kann direkt im "Konfigurator" oder in der "Console" gesetzt werden und werden dann auch gleich auf alle Komponenten des Toolkits angewendet.
   
   Noch ein Hinweis zu den Icons, diese befinden sich unter "C:\Jellyfin\Admin\Icons\Languages".
   
   Einige der gängigen Sprach-Icons habe ich breits abgelegt, sollten noch weitere benötigt werden, können diese hier herunter geladen werden:
   
   [https://iconarchive.com/show/flag-icons-by-gosquared.1.html](https://iconarchive.com/show/flag-icons-by-gosquared.1.html)
   
   Wichtig für den Download:
   
   1. Auf die entsprechende Flagge klicken
   
   2. "All Download Formats" auswählen
   
   3. Auf "Windows: Download ICO" klicken
   
   4. Das heruntergeladene Icon umbennen, z.b. für deutsch in "de-DE.ico"
   
   5. Icon im oben genannten Ordner ablegen
      
      Die Icons werden dann ebenfalls automatisch zum enstprechenden Eintrag in den jewiligen Sprach-Auswahlmenüs angezeigt, falls nicht "Console"" oder "Konfigurator"" einfach neustarten.



## Komponenten des Toolkits

Das Toolkit besteht aus den Komponenten "SysTray", "Console", "Updater" und "Konfigurator", im folgenden ein paar Worte dazu.

### SysTray

Die Komponente die alles ins Rollen brachte.

Wie der Name schon sagt, lässt sich der Dienst damit über den SysTray von Windows steueren, aber das Icon kann auch ein klein wenig mehr.

hier der Aufbau (von oben nach unten)

- Jellyfin Media Server - Öffnet die Weboberfläche bzw. den Jellyfin Media Client (Wenn das Client Verzeichnis im Konfigurator angegeben wurde)

- Dienst Starten - Startet den Dienst, sofern installiert und noch nicht gestartet

- Dienst neustarten - Startet den Dienst neu, sofern installiert und nicht gestoppt

- Dienst Stoppen - Stoppt den Dienst, sofern installiert und nicht gestoppt

- Konsole - Öffnet die "Console"

- Konfigurator - Öffnet den "Konfigurator"

- Update Verfügbar - Wird angezeigt, wenn "auf Updates prüfen" aktiviert und eine neue Version verfügbar ist

- Beenden - beendet den Jellyfin SysTray

- Doppelklick auf das SysTray Icon - Öffnet die "Console"

Als weiteres Feature verfügt SysTray auch über "Ballootips". Diese erscheinen, wenn der Dienst seinen Status ändert (gestartet, gestoppt, unbekannt) und auch, sofern "auf Updates prüfen" aktiviert ist, wenn eine neue Version verfügbar ist.

### Console

Die Console dient als eine Art kleines Informationszentrum. Primär zeigt sie das Logfile in Echtzeit und stellt die einzelen Warnstufen in verschiedenen Farben dar, damit wird es übersichtlicher und man hat alles schnell im Blick.

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
