# Multicam
Org (Eng.): https://community.octoprint.org/t/setting-up-multiple-webcams-in-octopi-the-right-way/32669

## Multicam-Einrichtung
Sie können mit dieser Methode so viele Kameras einrichten, wie Sie möchten, indem Sie einfach die Parameter jedes Mal ändern. Achten Sie jedoch auf Ihre CPU-Auslastung, da das Streaming sehr ressourcenintensiv sein kann.

## Voraussetzungen
Du brauchst:
- OctoPi 0.17 oder höher
- SSH-Zugriff oder eine alternative Möglichkeit, auf die Befehlszeile zuzugreifen
- Natürlich mehr als eine Kamera! *Raspi Cam*, *USB Webcam* o. *IP-Cam (z.b. ESP-Cam)*

**Warnung: Dieses Handbuch ist nicht kompatibel mit Obico (ehemals "The Spaghetti Detective") Premium-Webcam-Streaming. 
Sie müssen das Plugin in den "Kompatibilitätsmodus" schalten, damit diese Anleitung funktioniert.**

## 1. Erstellen der Konfigurationsdateien
OctoPi unterstützt eine *unendliche Anzahl von Konfigurationsdateien im *octopi.txt-style*. 
Sie können im Ordner "/boot/octopi.conf.d/" platziert werden.


Erstellen Sie die Dateien vom Terminal (z.b. Putty) aus:
```sh
cd /boot
sudo mkdir octopi.conf.d
sudo cp octopi.txt octopi.conf.d/webcam2.txt
# sudo cp octopi.txt octopi.conf.d/webcam3.txt
# sudo cp octopi.txt octopi.conf.d/webcam4.txt
# sudo cp octopi.txt octopi.conf.d/webcam5.txt
# .......
```
- **cd /boot** wechselt in verzeichnes /boot
- **mkdir octopi.conf.d** Erstellt den Ordner "octopi.conf.d"
- **cp octopi.txt octopi.conf.d/webcam2.txt** kopiert die Datei octopi.txt ins o. angelegtem verzeichnes "/boot/octopi.conf.d/"

## 2. Bearbeiten der ursprünglichen Konfigurationsdatei (/boot/octopi.txt) für eine bestimmte Kamera

a) Finden Sie die ID zur Kamera heraus. Dies macht es viel einfacher, Sie anzuhalten und einzuschalten

```sh
ls  /dev/v4l/by-id/
```
Output:
```sh
pi@octoprint:~ $ ls /dev/v4l/by-id/
usb-046d_0825_88C56B60-video-index0  usb-046d_0825_88C56B60-video-index1
```
Benutzen Sie die erste ID der auf *....-video-index0* enden, da dies normalerweise das Gerät mit Videostreaming ist.

b) Dann bearbeiten Sie:/boot/octopi.txt

```sh
sudo nano /boot/octopi.txt
```

suche die Zeile, #camera="auto"
```sh
#camera="auto"
```
ändere sie in 
```sh
camera="usb"
```
die zeile 
```sh
#camera_usb_options="-r 640x480 -f 10"
```
zu:

```sh
camera_usb_options="-r 640x480 -f 10 -d /dev/v4l/by-id/<device long id>"
```

** Anmerkung: Verwenden Sie eine Raspberry Pi-Kamera? 
Wird oben in der Datei festgelegt, und Sie können die Referenzierung der RPi-Kamera anhand der Geräte-ID ignorieren. 
Sie müssen für die Pi-Kamera folgendes eintragen:**
```sh
camera="raspi"
```

## 3. Bearbeiten der zweiten Konfigurationsdatei
Es gibt zwei Dinge, die hier angepasst werden müssen: der Port, unter dem mjpg_streamer ausgeführt wird, und das Gerät wieder.

Finden Sie die lange ID der zweiten Kamera, wie Sie es für die erste getan haben.

Bearbeitung der Datei: "webcam2.txt"
```sh
sudo nano /boot/octopi.conf.d/webcam2.txt
```
bzw. 
```sh
sudo nano /boot/octopi.conf.d/webcam3.txt
sudo nano /boot/octopi.conf.d/webcam4.txt
sudo nano /boot/octopi.conf.d/webcam5.txt
```
Wie oben wieder, einstellen und einstellen, aber diesmal mit der ID (ls /dev/v4l/by-id/) für die zweite/dritte Kamera: 
```sh
camera_usb_options="-r 640x480 -f 10 -d /dev/v4l/by-id/<device id>"
```
c) Passen Sie den Port an, indem Sie Folgendes bearbeiten: 
```sh
#camera_http_options="-n"
```
zu:
```sh
camera_http_options="-n -p 8081" # Cam 2
```
bzw.
```sh
camera_http_options="-n -p 8082" # Cam 3
#camera_http_options="-n -p 8083" # Cam 4
#camera_http_options="-n -p 8084" # Cam 5
# usw.
```
## 4. Testen Sie es funktioniert

Mit
```sh
sudo service webcamd restart
```
Starten Sie den Webcam Service neu, 
danach versuchen Sie, Ihre zweite Kamera unter der zu finden.
- Wenn es nicht funktioniert, überprüfen Sie das Webcam-Protokoll "/var/webcamd.log" für Details.

- Wenn es funktioniert, können Sie die Stream- und Snapshot-URLs in OctoPrint verwenden:

```sh
    Stream: http://<your-ip>:8081/?action=stream
    Snapshot: http://<your-ip>:8081/?action=snapshot
```
