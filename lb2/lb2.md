# LB2 Dokumentation von Jorin Bailer
![image](https://github.com/jorinbyte/M300_service/blob/main/lb2/Bilder%20MD/Bild_Visualisierung_IaC_Jorin_Bailer.PNG)





# Inahltsverzeichnis
 1. [Auftrag](#auftrag)
 
 1. [Einführung](#einführung) 

 2. [Erklärung Code](#Erklärung)
    
 - 2.1 [Konfiguration VM](#kofnigcode)

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

Ich habe mich für das IaC Projekt für eine Maria DB Datenbank mit einer auf Apache gehostete Website entschieden. Auf dieser Website laufen einerseits [Adminer](https://www.adminer.org/) dies ist ein Datenbankverwaltungstool das mittels PHP lauft und [OPcache](https://www.php.net/manual/en/book.opcache.php) dies ist ebefalls ein PHP Tool das die Performace von PHP drastisch verbessert. Dies alles wir mittels NAT über das LAN erreichbar sein.


<div id='Erklärung'/>

# Erklärung Code

Nun kommen wir zum Code. Als erstes setzen wir die Zeitzone als Variable diese Wird säter noch öfters verwendet. 

```
time_zone = "Europe/zuerich"
```

Dann Starten wir die Konfiguration der Virtuellen Maschiene. Ich habe die erklärung kurz gehalten und es neben den Code geschriebn da ich sonst sehr viele von diesen Codeblöcke hätte.

<div id='kofnigcode'/>

## Konfiguration VM

```ruby
#standart Vagrant erschaffen
Vagrant.configure("2") do |config|  
  config.vm.box = "generic/ubuntu1804" #angabe des Betriebssystems
  config.vm.network "forwarded_port", guest: 80, host: 1234 #80 ist der Port der auf dem Gaystsystem geöffnet wird und 1234 auf der VM 
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
 


<div id='Testen'/>

# Testen

<div id='quellenangabe'/>

# Quellenangabe