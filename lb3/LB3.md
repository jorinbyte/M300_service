# LB3 Dokumentation von Jorin Bailer
![image](https://github.com/jorinbyte/M300_service/blob/main/lb3/BilderMD/Docker_Logo.png?raw=true)

# Inahltsverzeichnis
 1. [Auftrag](#auftrag)
 
 1. [Einführung](#einführung) 

 2. [Erklärung Code](#Erklärung)

 3. [Testen](#testen)

 4. [Quellenagabe](#quellenangabe)


<div id='Auftrag'/>

# Auftrag
Sie erstellen ein selbst gewähltes Projekt, welches auf der Docker Container-Technologie basiert. Dabei erstellen sie einen Service, der innerhalb eines oder mehrerer Container implementiert wird. Die Implementation des Container-Projektserfolgt als Einzelarbeit. Der erstellte Code sowie die gesamte Dokumentation müssen versioniert auf GitHub hinterlegt und der Lehrperson zugänglich sein (Lese-Rechte). Das Internet ist eine wichtige Ressource für solche Projekte. Entsprechend dürfen sie auch Codebeispiele aus dem Internet verwenden. Sie müssen solchen Code immer mit der Angabe der Quelleversehen. Der verwendete Code muss von ihnen vollständig dokumentiert sein. Das gilt auch für Code, welchen sie aus fremden Quellen verwenden. Das bedeutet,sie können über den verwendeten Code Auskunft geben.
<div id='Einführung'/>

# Einführung

<div id='Erklärung'/>

Da ich noch nie mit Docker gearbeitet habe wollte ich ein Projekt machen das mirmit meiner erfahrung realistisch vorkommt. Ich machte verschiedene Docker Toutorials duch und entschloss schlussendlich das ich eine Wordpress Seite mache verbunden mit einer SQL Datenbank. Dies sollte auf dem Localhost im Lan verfügbar sein. Natürlich kann man am diese Modifizieren und alles mögliche dort Hosten wie ein internes Wiki oder sonstiges.
# Erklärung Code

Hier ist als erstes das ganze Dockerfile abgelegt. Ich suche mir ein Paar teile raus und erkläre diese genauer
```yml   
services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: Admin123
      MYSQL_DATABASE: wordpress
      MYSQL_USER: Admin
      MYSQL_PASSWORD: Admin123
    
  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    volumes:
      - wordpress_data:/var/www/html
    ports:
      - "1234:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: Admin
      WORDPRESS_DB_PASSWORD: Admin123
      WORDPRESS_DB_NAME: wordpress
volumes:
  db_data: {}
  wordpress_data: {}
```
Als erstes 
```yml
db: #Name der Aufgabe
    image: mysql:5.7 #Name und Version von dem Image das von Docker gezogen werden soll
    volumes:
      - db_data:/var/lib/mysql #Ordner die erschaffen werden sollen 
    restart: always  # gibt an das wärend der installation ohne nachfragen restartet werden sollte
```
Envirotemnt festlegen mit Passwort und Datenbank für die DB
```yml
 environment:
      MYSQL_ROOT_PASSWORD: Admin123
      MYSQL_DATABASE: wordpress
      MYSQL_USER: Admin
      MYSQL_PASSWORD: Admin123
```
Angabe das es sich auf die DB bezieht. Und wie bei Sql die Version die man von docker ziehen will.

```yml
  wordpress:
    depends_on:
      - db
    image: wordpress:latest
```
Name des Volumes das angelegt werden soll.
```yml
    volumes:
      - wordpress_data:/var/www/html
```
Ports die wetergeleitet werden. Also im Container Port 80 ausserhalb 1234 um Überschneidungen zu vermeiden.
```yml
ports:
      - "1234:80"
```
Wie bei Sql die Datenbank angaben und der Benutzer.
```yml
environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: Admin
      WORDPRESS_DB_PASSWORD: Admin123
      WORDPRESS_DB_NAME: wordpress
```

# Testen
<div id='quellenangabe'/>



# Quellenangabe


| Objekt   |      Quelle     |
|:----------|:-------------|
| Docker Logo |  https://user-images.githubusercontent.com/21102559/41428354-d2fd1052-6fd7-11e8-8824-d4873955d89c.png|
|  |     |
|  |  |
|   |  |
