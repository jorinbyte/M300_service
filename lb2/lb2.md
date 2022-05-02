# LB2 Dokumentation von Jorin Bailer
![image](https://github.com/jorinbyte/M300_service/blob/main/lb2/Bilder%20MD/Vagrant_logo.png?raw=true)





# Inahltsverzeichnis
 1. [Auftrag](#auftrag)
 
 1. [Einführung](#einführung) 

 2. [Erklärung Code](#Erklärung)

 3. [Testen](#testen)

 4. [Quellenagabe](#quellenangabe)

<div id='Auftrag'/>

# Auftrag

Sie erstellen -auf Basis von VirtualBox/Vagrant-ein selbst gewähltes **«Infrastructure as Code»** Projekt, indem sie einen Service oder Serverdienst **automatisieren.**

Teamarbeit ist erwünscht. Die Implementation des **IaC-Projekts** erfolgt hingegen **als Einzelarbeit**. Der erstellte Code sowie die gesamte Dokumentation wird versioniert auf [GitHub](https://github.com/), hinterlegt und der Lehrpersonzugänglich gemacht (Lese-Rechte).

Das Internet ist eine wichtige Ressource für solche Projekte. Entsprechend dürfen sie auch Codebeispiele aus dem Internet verwenden, sofern sie entsprechende Quellenangaben machen.

Der verwendete Code muss von ihnen vollständigdokumentiert sein, das gilt auch für Code, mindestens in groben Zügen, welchen sie aus fremden Quellen verwenden. 

**Das bedeutet sie können über den verwendeten Code Auskunft geben.**

<div id='Einführung'/>

# Einführung

Ich habe mich für das IaC Projekt für eine Maria DB Datenbank mit einer auf Apache gehostete Website entschieden. Auf dieser Website laufen einerseits [Adminer](https://www.adminer.org/) dies ist ein Datenbankverwaltungstool das mittels PHP funktioniert und [OPcache](https://www.php.net/manual/en/book.opcache.php) dies ist ebefalls ein PHP Tool das die Performace von PHP drastisch verbessert. Dies alles wir mittels NAT über das LAN erreichbar sein.

Das konzept sieht wie folgt aus:

![image](https://github.com/jorinbyte/M300_service/blob/main/lb2/Bilder%20MD/Bild_Visualisierung_IaC_Jorin_Bailer.PNG?raw=true)

Port 80 der VM sollte auf Port 1234 des Hostsystems weitergeleitet werden um mögliche Komplikationen mit anderen Webservern zu vermeiden.

<div id='Erklärung'/>

# Erklärung Code

Nun kommen wir zum Code. Als erstes setzen wir die Zeitzone als Variable diese Wird säter noch öfters verwendet. 

```
time_zone = "Europe/zuerich"
```

Dann Starten wir die Konfiguration der Virtuellen Maschiene. Ich habe die erklärung kurz gehalten und es neben den Code geschriebn da ich sonst sehr viele von diesen Codeblöcke hätte.





```ruby
#standart Vagrant erschaffen
Vagrant.configure("2") do |config|  
  config.vm.box = "generic/ubuntu1804" #angabe des Betriebssystems
  config.vm.network "forwarded_port", guest: 80, host: 1234 #80 ist der Port der auf dem Gastsystem geöffnet wird und 1234 auf der VM 
  config.ssh.forward_agent = true #damit man sich mit der VM mittels vagrant ssh verbinden kann

  config.vm.synced_folder ".", "/vagrant", type: "virtualbox", #angabe des gesharten Ordners
    owner: "www-data", group: "www-data" #angabe owner und group des gesharten folders

  config.vm.provider "virtualbox" do |vb| #config VM
    vb.name = "vagrant_ubuntuM300_LB2_jb" #VM Name
    vb.memory = 1024 #Ram Size
    vb.customize ["guestproperty", "set", :id, "/VirtualBox/GuestAdd/VBoxService/--timesync-set-threshold", 10000] #synchronisation der Zeit durch Host 
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"] #nat dns von Hsot übernehmen
    vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"] # nat Proxy von Host übernehmen
  end

  config.vm.provision "shell" do |s| #shell eingabe 
    s.args = [time_zone]  #zeitzone angeben
    s.inline = <<-SHELL   
```



Als nächstes legen wir die Zeitzone fest und installieren Updates und alle Packete die wir brauchen.

```ruby
# Zeitzone 
TIME_ZONE=$1
# einfach instalieren und nicht auf y oder n warten
export DEBIAN_FRONTEND=noninteractive 

# Zeitzone updaten
timedatectl set-timezone "$TIME_ZONE"

# Alle Pakages updaten
apt-get update -q

# Vim und Git installieren für repositorys clonen und vim für danach in das WWW file schreiben
apt-get install -q -y vim git

# instalieren von Apache, Mariadb und PHP
apt-get install -q -y apache2
apt-get install -q -y php7.2 libapache2-mod-php7.2
apt-get install -q -y php7.2-curl php7.2-gd php7.2-mbstring php7.2-mysql php7.2-xml php7.2-zip php7.2-bz2 php7.2-intl
apt-get install -q -y mariadb-server mariadb-client
```


Apache starten:

 ```ruby
 systemctl restart apache2
 ```

Lokales directory erstellen  falls es noch nicht existiert und mit VM sharen:
```ruby
dir='/vagrant/www'
if [ ! -d "$dir" ]; then
  mkdir "$dir"
fi
if [ ! -L /var/www/html ]; then
  rm -rf /var/www/html
  ln -fs "$dir" /var/www/html
fi
cd "$dir"
```
Indexfile für die Navigation der Webseite erstellen:

```ruby
file='/etc/apache2/sites-available/dev.conf'
if [ ! -f "$file" ]; then
  SITE_CONF=$(cat <<EOF
<Directory /var/www/html>
  AllowOverride All
  Options +Indexes -MultiViews +FollowSymLinks
  AddDefaultCharset utf-8
  SetEnv ENVIRONMENT "development"
  php_flag display_errors On
  EnableSendfile Off
</Directory>
EOF
)
  echo "$SITE_CONF" > "$file"
fi
```
Apache neu laden:
```ruby
a2ensite dev
systemctl reload apache2
```

Composer für die Abhängigkeitshyrarchie von PHP herunterladen und installieren:
```ruby
# Composer
EXPECTED_SIGNATURE="$(wget -q -O - https://composer.github.io/installer.sig)"
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
ACTUAL_SIGNATURE="$(php -r "echo hash_file('SHA384', 'composer-setup.php');")"
php composer-setup.php --quiet
rm composer-setup.php
mv composer.phar /usr/local/bin/composer
chmod +x /usr/local/bin/composer
sudo -H -u vagrant bash -c 'composer global require hirak/prestissimo'
<div id='Testen'/>
```

OPcache File anlegen falls noch nicht vorhanden:
```ruby
# OPcache
file='opcache.php'
if [ ! -f "$file" ]; then
  wget -nv -O "$file" https://raw.githubusercontent.com/amnuts/opcache-gui/master/index.php
fi
```
Adminer File anlegen falls noch nicht vorhanden:
```ruby
# Adminer 
file='adminer.php'
if [ ! -f "$file" ]; then
  wget -nv -O "$file" http://www.adminer.org/latest.php
  wget -nv https://raw.githubusercontent.com/vrana/adminer/master/designs/pepa-linha/adminer.css
fi
```
MySQL Konto für verbindung anlegen und Rechte geben:
```ruby
#mysql Login erstellen 
sudo mysql <<-EOF

  create user 'Admin'@'localhost' identified by 'Admin123';
  Grant all privileges on *.* to 'Admin'@'localhost';
  Flush privileges;

EOF
```
Shell und File beenden:
```ruby
SHELL
  end
end
```

# Testen

Nachdem  ```vagrant up``` im Verzeichniss mit dem Vagrantfile ausgeführt wurde, und das ganze durchgelaufen ist. Nun kann man in den Browser ihrer Wahl ```localhost:1234``` eingeben und sollte auf folgende seite kommen:

![image](https://github.com/jorinbyte/M300_service/blob/main/lb2/Bilder%20MD/Bild%20Verzeichnis%20Apache.PNG?raw=true)

Wenn man nun auf adminer.php klickt sollte man auf folgende Seite kommen:
![image](https://github.com/jorinbyte/M300_service/blob/main/lb2/Bilder%20MD/Anmelden%20Adminer.PNG?raw=true)
Hier kann man sich nun mit folgenden Credentials anmelden: 

|    |        | 
|----------|:-------------:|
| Datenbank System |  MySQL | 
| Server |    localhost   | 
| Benutzer | Admin | 
| Passwort  |  Admin123 |
|Datenbank|*leerlassen für erst-anmeldung*|


Nach der Eingabe auf Login drücken und es sollte wie folgt aussehen:

![image](https://github.com/jorinbyte/M300_service/blob/main/lb2/Bilder%20MD/Angemeldet%20Adminer.PNG?raw=true)

Gratulation nun kann man die Datenbank Oline Verwalten :)



Um auf das OPcache zu kommen kann man nochmals ```localhost:1234``` im Browser eingeben oder zwei mal zurück gehen und auf opcahce.php drücken. Dann sollte folgende Seite erscheinen:

![image](https://github.com/jorinbyte/M300_service/blob/main/lb2/Bilder%20MD/OPcache.PNG?raw=true)


Natürlich kann man sich noch mittels ssh in die Maschiene selbst einloggen. Dazu kann man einfach  ```vagrant shh``` im gleichen Commandfenster wie man vagrant up ausgeführt hat ausführen. Und schon hatt man Kommandozeilen Zugriff auf die VM.

<div id='quellenangabe'/>

# Quellenangabe

| Objekt   |      Quelle     |
|:----------|:-------------|
| Vagrant Logo |  https://www.vagrantup.com/|
| Alle anderen Biler |    Privat  |
| Code snippets | https://gist.github.com/aurmil/e346aec64c3f6b6ea17259f41e3b6ab0 |
|  Auftrag |  https://tbzedu.sharepoint.com/sites/M300_Documents/Freigegebene%20Dokumente/Forms/AllItems.aspx?id=%2Fsites%2FM300%5FDocuments%2FFreigegebene%20Dokumente%2FLeistungsbeurteilung%2FM300%5FLB2%5FIaC%5F1%5F6%2Epdf&parent=%2Fsites%2FM300%5FDocuments%2FFreigegebene%20Dokumente%2FLeistungsbeurteilung&p=true |
