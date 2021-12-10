# NextCloud Anleitung (OHNE reverse Proxy, LetsEncrypt, DNS - Einfach und Lokal)

Eine Anleitung zur Lokalen Installation von Nextcloud auf basis von OpenMediaVault und Docker.
# Table Of Contents
- [NextCloud Anleitung (OHNE reverse Proxy, LetsEncrypt, DNS - Einfach und Lokal)](#nextcloud-anleitung--ohne-reverse-proxy--letsencrypt--dns---einfach-und-lokal-)
  * [1 Vorbereitung](#1-vorbereitung)
    + [1.1 Hardware zusammenstellung](#11-hardware-zusammenstellung)
    + [1.2 Meine Zusammenstellung](#12-meine-zusammenstellung)
  * [2 Software](#2-software)
  * [3 Software Raid](#3-software-raid)
  * [3.1 Einrichtung von OMV](#31-einrichtung-von-omv)
    + [3.1.1 Raid-Verwaltung](#311-raid-verwaltung)
    + [3.1.2 Dateisystem](#312-dateisystem)
    + [3.1.3 Sharedfolder / Freigebener Ordner](#313-sharedfolder---freigebener-ordner)
    + [3.1.3.1 RAID1-Ort](#3131-raid1-ort)
    + [3.1.4 Docker und Portainer Installieren](#314-docker-und-portainer-installieren)
  * [4 Docker](#4-docker)
    + [4.1 Nextcloud und MariaDB installatieren](#41-nextcloud-und-mariadb-installatieren)
    + [4.1.1 Anpassungen](#411-anpassungen)
    + [4.1.2 Warum sind diese Anpassungen wichtig?](#412-warum-sind-diese-anpassungen-wichtig-)
    + [4.3 Finale Installation](#43-finale-installation)
  * [5 Letzte Schritte](#5-letzte-schritte)
    + [5.1 Automatisches Hoch- und runterfahren](#51-automatisches-hoch--und-runterfahren)
  * [6 Todo](#6-todo)
  * [7 Quellen](#7-quellen)


## 1 Vorbereitung
### 1.1 Hardware zusammenstellung
Für bestmögliche Performance, wird eine gute Hardware vorausgesetztv.
Da Nextcloud wenig Ressourcen benötigt, kann auch ein Raspberry Pi 4 benutzt werden.
Ich habe mich gegen ein Raspberry Pi entschieden, 
da die Benutzung einer SD-Karte als OS-Storage zu Problemen führen kann (Nicht sehr hohe Schreib/Lese Geschwindigkeiten,
kurze Lebensdauer, etc.)
Zudem ist es empfehlenswert, ein Server zu wählen, der über SATA Anschlüsse verfügt, um das RAID bestmöglich nutzen zu können.
<br>
### 1.2 Meine Zusammenstellung
Ich habe mich für die Folgendenteile entschieden:

* [Mainboard](https://www.mindfactory.de/product_info.php/ASRock-H310CM-HDV-M-2-Intel-H310-So-1151-Dual-Channel-DDR4-mATX-Retail_1291052.html) (ASRock H310CM-HDV/M.2)
* [CPU](https://ark.intel.com/content/www/de/de/ark/products/126688/intel-core-i3-8100-processor-6m-cache-3-60-ghz.html) (Intel I3 8100)
* [RAM](https://www.mindfactory.de/product_info.php/8GB-G-Skill-Aegis-DDR4-2666-DIMM-CL19-Single_1237317.html) (8gb DDR4-2666)
* [HDD](https://www.mindfactory.de/product_info.php/4000GB-WD-Red-Plus-WD40EFZX-128MB-3-5Zoll--8-9cm--SATA-6Gb-_1393203.html)(WD Red Plus 2x 4TB)
* [SSD](https://www.mindfactory.de/product_info.php/500GB-Samsung-SSD-980-M-2-PCIe-3-0-x4-3D-NAND-TLC--MZ-V8V500BW-_1401240.html)(M2 ist mit dem MB kompatibel)
* [SATA 4x Kabel](https://www.mindfactory.de/product_info.php/0-30m-Delock-Strom-Adapterkabel-SATA-SATA-Stecker-auf-4xSATA-Buchse-Sch_1217830.html)(1x SATA zu 4x SATA)
* [CPU POWER ADAPTER](https://www.mindfactory.de/product_info.php/0-20m-Good-Connections-Stromkabel-intern-8pol-Stecker-auf-4pol-Buchse-Sc_991032.html)(4pin auf 8pin)
* [PSU](https://www.amazon.de/PicoPSU-150-XT-DC-DC-Netzteil-power-supply/dp/B0045IXKTQ/ref=sr_1_5?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=pico+psu&qid=1635528774&sr=8-5)
* [NETZTEIL](https://www.amazon.de/Laufwerke-Lichtschl%C3%A4uche-LED-Strips-TFT-Monitore-LED-Beleuchtungen/dp/B00YXXAG7C/ref=pd_sbs_1/262-5325128-6073649?pd_rd_w=RvsQg&pf_rd_p=840402ae-0fda-4c49-9fb0-db7f87dd6eac&pf_rd_r=8NENW0WEPTE1E8Q8EYTB&pd_rd_r=1783bd2d-79a0-4f65-b64f-daaacaaa60d5&pd_rd_wg=BDT5Y&pd_rd_i=B00YXXAG7C&psc=1)

Eine SSD musste ich nicht kaufen, da ich noch eine rumliegen hatte. Eine SSD ist aber aufjdenfall notwendig, um darauf das Betriebssystem laufen zulassen. 
Die Adapterkabel,PSU und das oben genannte Netzteil sind natürlich nicht nötig, wenn man ein normales PC-Netzteil verbaubt.
Ich habe mich aber aufgrund von Lautstärke und Größe gegen ein PC Netzteil entschieden.

## 2 Software
Für dieses Projekt wird OpenMediaVault benutzt.
OMV hat viele coole Extras und kann von einer WEB-Oberfläche gesteuert werden.
Um die Software auf den Server aufzuspielen, empfielt es sich eine Ofizielle Image Datei herunterzuladen und diese dann mithilfe von [Etcher](https://www.balena.io/etcher/) auf einen USB Stick zuflashen.
Es gibt dazu viele Tutorials im Internet.

## 3 Software Raid
Nun müssen/können wir uns für ein RAID entscheiden.
Da mir die Daten wichtig sind, will ich diese Spiegeln.
Zur verfügung stehen mehrere RAID optionen (es gibt noch mehr, aber hier die wichtigsten), sie unterscheiden sich in Performance, Sicherungsmölgichkeiten und Speicherplatz.
Ich habe mich für ein RAID 1 entschieden, da ich nur 2x HDDs habe. Wenn mir mehr HDDs zur verfügung stünden, hätte ich möglicherweise RAID 5 genommen.
WICHTIG: Ein RAID ist kein Backup. Wenn der Server z.B geklaut wird, sind beide Festplatten weg. 
Besser ist, wenn man eine exterene Sicherung hat, welche sich am besten nicht zuhause befindet.

| Raid | min. Festplatten | Wie viele Festplatten(x) können tatsächlich genutzt werden | Funktion                                                                     | Vorteile                                                                            |
|------|------------------|------------------------------------------------------------|------------------------------------------------------------------------------|-------------------------------------------------------------------------------------|
| 0    | 1                | x                                                          | Mann kann die alle Festplatten beschreiben, hat aber keine Sicherung         | Viel Speicherplatz                                                                  |
| 1    | 2                | x/2                                                        | Hier wird auf eine Festplatte geschrieben, auf der anderen gespiegelt        | Daten werden gesichert. Bei einem Ausfall können Daten schnell überschrieben werden |
| 5    | 3                | x-1                                                        | Die Platten werden in Partitionen unterteilt und sichern sich gegenseitig ab | Sicherung + Viel Speicherplatz                                                      |
| 10   | 4                | x/2                                                        | Eine Kombination aus RAID 1 und RAID 0                                       | Schneller Zugriff + Sicherung                                                       |

## 3.1 Einrichtung von OMV
Nach Installation von OMV kann man kann mit der Erstellung des RAID anfangen.

### 3.1.1 Raid-Verwaltung
1. Im Linken Reiter unter "Datenspeicher" auf "Raid-Verwaltung"
2. Dann auf Erstellen klicken
3. Nun einen Namen ausdenken z.B Raid1NAS
4. Das RAID Level auswählen, in meinem Fall RAID 1
5. Die zur Verfügung stehenden Festplatten auswählen
7. "Erstellen"

Die Erstellung kann einige Zeit in Anspruch nehmen.
Bei mir hat das für 2x 4TB Festplatten 6 Stunden gebraucht.

### 3.1.2 Dateisystem
Nun muss man aus dem RAID noch ein Dateisystem Erstellen.
Die beliebtesten sind ext4 und btrfs.
EXT4 wird von vielen bevorzugt, da es schon seit Jahren in vielen Servern Anwenung findet und wenig Probleme macht.
BTRFS ist ein relativ neues Dateisystem und auch sehr robust, wurde aber noch nicht so ausgiebig getestet wie ext4.
Ich habe mich einfach für ext4 entschieden, aber man kann bei dieser Entscheidung eigentlich nichts falsch machen.

Nun zur Erstellung des Dateisystems.
1. Im Linken Reiter unter "Dateisystem"
2. Dann auf Erstellen klicken
3. Unser ersteltes RAID bei der geräte Auswahl wählen
4. Dem ganzen einen Namen geben z.B EXT4NAS
5. Und das Dateisystem auswählen z.B EXT4
6. "OK"

### 3.1.3 Sharedfolder / Freigebener Ordner
Damit der spätere Docker Container auf das Raid zugreifen kann, muss ein Sharedfolder angelegt werden.
1. Im Linken Reiter unter "Freigegebene Ordner"
2. Dann auf Hinzufügen
3. Einen Namen auswählen z.B HomeNAS
4. Bei Geräte das eben erstellte Dateisystem auswählen
5. Einen Pfad angeben z.B HomeNAS/
6. "Speichern"

### 3.1.3.1 RAID1-Ort
***Das RAID-System kann man via FTP oder SSH unter folgendem Dateipfad finden***
```bash
/srv/xxxxxxxxxxx/HomeNAS
```
xxxxx sind bei mir 2 Disklabels. Um herauszufinden welches das richtige ist,
muss man einfach nur schauen in welchem sich der Sharedfolder(in unserem Fall HomeNAS/) befindet.

### 3.1.4 Docker und Portainer Installieren
Die Nextcloud instanz werden wir später in einem Docker Container installieren.
Docker ist vereinfacht gesagt eine in sich selbst Isolierte Anwendungsumgebung.
Somit können in Containern verschiedene Programme geladen werden. Bei Ausfall eines Containers, sind die anderen Container nicht betroffen.

Um Docker zu installieren braucht man die OMV-Extras.
Dazu den folgenden Befehl in die Console eingeben. 

```bash
wget -O - https://github.com/OpenMediaVault-Plugin-Developers/packages/raw/master/install | bash

```
Um sich mit der Konsole zuverbinden, muss man eine SSH verbindung erstellen. Tutorials hierfür findet man wieder im Internet.

Nun sieht man in OMV unter dem Reiter OMV-Extras den Docker Tab.
1. Auf den Docker TAb klicken
2. Ein speicherort für Docker auswählen (am besten einfach leer lassen)
3. Dann auf den Docker Knopf drücken
4. "Installieren"
5. Dann auf den Portainer Knopf drücken
6. "Installieren"

Mit Portainer lassen sich die Docker Container einfach konfigurieren und Visualisieren.
## 4 Docker
Nach der Docker Installation müssen wir nun via SSH die notwendigen Container installieren.
Normalerweise läuft der [Nextcloud Container](https://hub.docker.com/_/nextcloud) auf der SQLite Datenbank. 
Diese ist aber verglichen mit z.B MariaDB deutlich langsamer. Aus diesem Grund werden wir Nextcloud mit MaraiDB installieren.
Zwar macht es die Installation etwas aufwendiger, es lohnt sich aber trotzdem.

### 4.1 Nextcloud und MariaDB installatieren
Via SSH muss man man eine .yml Datei an einem beliebigen Ort erstellen.
```batch
sudo touch docker-compose.yml
```
Nun öffnen wir die Datei um sie beschreiben zu könnnen.
```batch
sudo nano docker-compose.yml
```
In dieser Datei kommt jetzt der Befehl für Docker was installiert werden soll
```yml
# NextCLoud with MariaDB/MySQL
#
# Access via "http://localhost:8080" (or "http://$(docker-machine ip):8080" if using docker-machine)
#
# During initial NextCLoud setup, select "Storage & database" --> "Configure the database" --> "MySQL/MariaDB"
#
# When you want to change the login data, you must change it also under "db: enviroment:".
# Database user: nextcloudNAS
# Database password: nextcloudNAS
# Database name: ncdb
# Database host: replace "localhost" with "maria-db" the same name as the data base container name.

version: '2'

services:

  nextcloud:
    container_name: nextcloud
    restart: unless-stopped
    image: nextcloud
    ports:
      - 8080:80 #Port change
    volumes:
      - /srv/xxxxxxxx/HomeNAS/nextcloud/apps:/var/www/html/apps     #Das muss angepasst werden (Anleitung 4.1.1)
      - /srv/xxxxxxxx/HomeNAS/nextcloud/config:/var/www/html/config #Das muss angepasst werden (Anleitung 4.1.1)
      - /srv/xxxxxxxx/HomeNAS/nextcloud/data:/var/www/html/data     #Das muss angepasst werden (Anleitung 4.1.1)
    depends_on:
      - db

  db:
    container_name: maria-db
    restart: unless-stopped
    image: mariadb
    command: --innodb-read-only-compressed=OFF #Maria-DB FIX
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: ncdb
      MYSQL_USER: nextcloudNAS
      MYSQL_PASSWORD: nextcloudNAS
    volumes:
      - /srv/xxxxxxxx/HomeNAS/nextcloud/mariadb:/var/lib/mysql     #Das muss angepasst werden (Anleitung 4.1.1)
      
```
Das wird nun einfach kopiert und in den docker-compose.yml eingefügt.

### 4.1.1 Anpassungen
***WCIHTIG***<br>
Der Link der beiden Volumes muss angepasst werden, damit das RAID benutzt wird:
Ihr müsst wie bei 3.1.3.1 beschrieben, den Pfad ermitteln und ihn hier vor /nextcloud/... einfügen.
Den Pfad kann man dann einfach einfügen.
Den Editor kann man mit einer Tastenkomination verlassen und die Datei Speichern. Die Tastenkombination seht am unteren Rand des Editors.


***UNWICHTIG***<br>
In dem Skript unter "db:" ist folgender command:
```yml
    command: --innodb-read-only-compressed=OFF #Maria-DB FIX
```
Dieser Command ist erstmal nötig, da Nextcloud mit der neuen MariaDB Version nicht klarkommt. 
Falls Nextcloud das irgendwann packt, kann dieser command aus dem Skript entfernt werden.

### 4.1.2 Warum sind diese Anpassungen wichtig?
Da Docker selber auf der SSD liegt, müssen wir den Volumes (Speicher von MariaDB und Nextcloud) sagen, dass sie ihre daten auf das RAID Verzeichniss legen sollen.
Hier ein kleines Schaubild dazu:

![OMV-SCHEMA](https://github.com/Domepo/nextcloud-manual/blob/main/assets/img/OMV.png)

### 4.3 Finale Installation
Um nun endlich Nextcloud zu installeren fehlt nur noch ein kleiner command, der via SSH in dem gleichen Ort wie die yml Date ausgeführt werden muss.
```bash
docker-compose up -d
```
Feritg :)


## 5 Letzte Schritte
Nun kann man Nextcloud unter der LokalenIP:8080 öffnen.
Als Admin Passwort sollte man sich ein starkes überlegen.

Unterhalb des Admin Logins muss man dann noch die Datenbank auswählen, in unserem Beispiel MariaDB.
Die Daten dafür sind in der docker-compose.yml festgehalten:
```bash
# Database user: nextcloudNAS
# Database password: nextcloudNAS
# Database name: ncdb
# Database host: maria-db
```
### 5.1 Automatisches Hoch- und runterfahren
Hier noch ein Anleitung um den Server Automatisch Hoch- und runterzufahren:
https://www.youtube.com/watch?v=6fXNsu9I-h4
1. Auf System (OMV)
2. Geplante Aufgaben
3. Hinzufügen
4. Daten Eingeben (wann das NAS herunterfahren soll)
5. Dann folgenden Kommand unter Command eingeben
6. ``` rtcwake -m off -s60```
7. 60 = Sekunden

Jetzt fährt das NAS geregelt runter und wartet 1 Minute bis er wieder hochfährt.
Das wars :)

## 6 Todo
* OpenVPN integration
* Englische Version
* Version controll
* Weitere Anleitung für LetsEncrypt, Reverse Proxy, DNS 

## 7 Quellen
Q1: https://dannyda.com/2020/04/15/how-to-install-docker-on-openmediavault-5-omv5-easily-how-to-configure-docker-to-use-specific-location-to-store-files-container-images-other-than-default-location-easily/?__cf_chl_captcha_tk__=pmd_2Z1R_00BQ58MuXGLtd.fOMbeOOTtDXmOEGxsUDo4AL0-1635461667-0-gqNtZGzNA9CjcnBszQm9
Q2 :https://www.cloudsavvyit.com/12476/how-to-self-host-a-collaborative-cloud-with-nextcloud-and-docker/
Q3 :https://gist.github.com/ichiTechs/83e228fa1e6c83543623a1bf06f3eb32
