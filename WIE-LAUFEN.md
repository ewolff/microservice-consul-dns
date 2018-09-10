# Beispiel starten

Die ist eine Schritt-für-Schritt-Anleitung zum Starten der Beispiele.
Informationen zu Maven und Docker finden sich im
[Cheatsheet-Projekt](https://github.com/ewolff/cheatsheets-DE).

## Installation

* Die Beispiele sind in Java implementiert. Daher muss Java
  installiert werden. Die Anleitung findet sich unter
  https://www.java.com/en/download/help/download_options.xml . Da die
  Beispiele kompiliert werden müssen, muss ein JDK (Java Development
  Kit) installiert werden. Das JRE (Java Runtime Environment) reicht
  nicht aus. Nach der Installation sollte sowohl `java` und `javac` in
  der Eingabeaufforderung möglich sein.

* Die Beispiele laufen in Docker Containern. Dazu ist eine
  Installation von Docker Community Edition notwendig, siehe
  https://www.docker.com/community-edition/ . Docker kann mit
  `docker` aufgerufen werden. Das sollte nach der Installation ohne
  Fehler möglich sein.

* Die Beispiele benötigen zum Teil sehr viel Speicher. Daher sollte
  Docker ca. 4 GB zur Verfügung haben. Sonst kann es vorkommen, dass
  Docker Container aus Speichermangel beendet werden. Unter Windows
  und macOS findet sich die Einstellung dafür in der Docker-Anwendung
  unter Preferences/ Advanced.

* Nach der Installation von Docker sollte `docker-compose` aufrufbar
  sein. Wenn Docker Compose nicht aufgerufen werden kann, ist es nicht
  als Teil der Docker Community Edition installiert worden. Dann ist
  eine separate Installation notwendig, siehe
  https://docs.docker.com/compose/install/ .

## Build

Wechsel in das Verzeichnis `microservice-consuldns-demo` und starte `./mvnw clean
package` bzw. `mvnw.cmd clean package` (Windows). Das wird einige Zeit dauern:

```
[~/microservice-consuldns/microservice-consuldns-demo]./mvnw clean package
....
[INFO] 
[INFO] --- maven-jar-plugin:2.6:jar (default-jar) @ microservice-consuldns-demo-order ---
[INFO] Building jar: /Users/wolff/microservice-consuldns/microservice-consuldns-demo/microservice-consuldns-demo-order/target/microservice-consuldns-demo-order-0.0.1-SNAPSHOT.jar
[INFO] 
[INFO] --- spring-boot-maven-plugin:1.4.5.RELEASE:repackage (default) @ microservice-consuldns-demo-order ---
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary:
[INFO] 
[INFO] microservice-consuldns-demo ........................... SUCCESS [  1.401 s]
[INFO] microservice-consuldns-demo-hystrix-dashboard ......... SUCCESS [  3.601 s]
[INFO] microservice-consuldns-demo-customer .................. SUCCESS [ 25.636 s]
[INFO] microservice-consuldns-demo-catalog ................... SUCCESS [ 36.618 s]
[INFO] microservice-consuldns-demo-order ..................... SUCCESS [ 27.781 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------- -----------------
[INFO] Total time: 01:35 min
[INFO] Finished at: 2017-09-07T18:08:13+02:00
[INFO] Final Memory: 52M/416M
[INFO] ------------------------------------------------------------------------
```

Weitere Information zu Maven gibt es im
[Maven Cheatsheet](https://github.com/ewolff/cheatsheets-DE/blob/master/MavenCheatSheet.md).

Falls es dabei zu Fehlern kommt:

* Stelle sicher, dass die Datei `settings.xml` im Verzeichnis  `.m2`
in deinem Heimatverzeichnis keine Konfiguration für ein spezielles
Maven Repository enthalten. Im Zweifelsfall kannst du die Datei
einfach löschen.

* Die Tests nutzen einige Ports auf dem Rechner. Stelle sicher, dass
  im Hintergrund keine Server laufen.

* Führe die Tests beim Build nicht aus: `./mvnw clean package
-Dmaven.test.skip=true` bzw. `mvnw.cmd clean package
-Dmaven.test.skip=true`.

* In einigen selten Fällen kann es vorkommen, dass die Abhängigkeiten
  nicht korrekt heruntergeladen werden. Wenn du das Verzeichnis
  `repository` im Verzeichnis `.m2` löscht, werden alle Abhängigkeiten
  erneut heruntergeladen.

## Docker Container starten

Weitere Information zu Docker gibt es im
[Docker Cheatsheet](https://github.com/ewolff/cheatsheets-DE/blob/master/DockerCheatSheet.md).

Zunächst musst du die Docker Images bauen. Wechsel in das Verzeichnis 
`docker` und starte `docker-compose build`. Das lädt die Basis-Images
herunter und installiert die Software in die Docker Images:

```
[~/microservice-consuldns/docker]docker-compose build 
....
Removing intermediate container 1d59f8227b12
Step 4/4 : EXPOSE 8989
 ---> Running in 11e7fbacfa01
 ---> 9cfa7772986f
Removing intermediate container 11e7fbacfa01
Successfully built 9cfa7772986f
Successfully tagged msconsuldns_hystrix-dashboard:latest
```

Danach sollten die Docker Images erzeugt worden sein. Sie haben das
Präfix `msconsul`:

```
[~/microservice-consuldns/docker]docker images
REPOSITORY                   TAG                 IMAGE ID            CREATED             SIZE
msconsuldns_hystrix-dashboard   latest              9cfa7772986f        49 seconds ago      197MB
msconsuldns_order               latest              12b279e78975        52 seconds ago      225MB
msconsuldns_apache              latest              22fac099ba93        55 seconds ago      255MB
msconsuldns_catalog             latest              c23c535ecaf6        2 minutes ago       225MB
msconsuldns_customer            latest              a780e4f49bac        2 minutes ago       225MB
```

Ermittle die IP-Adresse deines Systems und weise sie der Umgebungsvariable
CONSUL_HOST. Diese Umgebungsvariable ist notwendig, damit die Docker
Container Consul für die DNS-Anfragen nutzen können:

```
[~/microservice-consuldns/docker]ifconfig 
....
en5: flags=8863<UP,BROADCAST,SMART,RUNNING,SIMPLEX,MULTICAST> mtu 1500
....
	inet 192.168.1.152 netmask 0xffffff00 broadcast 192.168.1.255
....
[~/microservice-consuldns/docker]export CONSUL_HOST=192.168.1.152
```


Nun kannst Du die Container mit `docker-compose up -d` starten. Die
Option `-d` bedeutet, dass die Container im Hintergrund gestartet
werden und keine Ausgabe auf der Kommandozeile erzeugen.

```
[~/microservice-consuldns/docker]docker-compose up -d
Creating network "msconsuldns_default" with the default driver
Pulling consul (consul:0.7.2)...
0.7.2: Pulling from library/consul
b7f33cc0b48e: Already exists
a4ca795d20eb: Pull complete
76bc5ef06918: Pull complete
965e633cb8c2: Pull complete
64e424fcbe65: Pull complete
Digest: sha256:ce15f85417a0cf121d943563dedb873c7d6c26e9b1e8b47bc2f1b5a3e27498e1
Status: Downloaded newer image for consul:0.7.2
Creating msconsuldns_consul_1 ... 
Creating msconsuldns_consul_1 ... done
Creating msconsuldns_order_1 ... 
Creating msconsuldns_catalog_1 ... 
Creating msconsuldns_customer_1 ... 
Creating msconsuldns_hystrix-dashboard_1 ... 
Creating msconsuldns_apache_1 ... 
Creating msconsuldns_hystrix-dashboard_1
Creating msconsuldns_order_1
Creating msconsuldns_catalog_1
Creating msconsuldns_apache_1
Creating msconsuldns_apache_1 ... done
```

Wie man sieht, werden nun auch noch Docker Images heruntergeladen.

Du kannst nun überprüfen, ob alle Docker Container laufen:

```
[~/microservice-consuldns/docker]docker ps
CONTAINER ID        IMAGE                        COMMAND                  CREATED             STATUS              PORTS                                                                                              NAMES
2697aa91700b        msconsuldns_apache              "/bin/sh -c '/usr/..."   4 minutes ago       Up 4 minutes        0.0.0.0:8080->80/tcp                                                                               msconsuldns_apache_1
948f2576b0b0        msconsuldns_customer            "/bin/sh -c '/usr/..."   4 minutes ago       Up 4 minutes        8080/tcp                                                                                           msconsuldns_customer_1
0574e8dc5b11        msconsuldns_order               "/bin/sh -c '/usr/..."   4 minutes ago       Up 4 minutes        8080/tcp                                                                                           msconsuldns_order_1
144542583a05        msconsuldns_catalog             "/bin/sh -c '/usr/..."   4 minutes ago       Up 4 minutes        8080/tcp                                                                                           msconsuldns_catalog_1
15968668c5e8        msconsuldns_hystrix-dashboard   "/bin/sh -c '/usr/..."   4 minutes ago       Up 4 minutes        0.0.0.0:8989->8989/tcp                                                                             msconsuldns_hystrix-dashboard_1
c28d2a38f657        consul:0.7.2                 "docker-entrypoint..."   4 minutes ago       Up 4 minutes        8300-8302/tcp, 8400/tcp, 8600/tcp, 8301-8302/udp, 0.0.0.0:8500->8500/tcp, 0.0.0.0:8600->8600/udp   msconsuldns_consul_1
```
`docker ps -a`  zeigt auch die terminierten Docker Container an. Das
ist nützlich, wenn ein Docker Container sich sofort nach dem Start
wieder beendet..

Wenn einer der Docker Container nicht läuft, kannst du dir die Logs
beispielsweise mit `docker logs msconsuldns_apache_1` anschauen. Der Name
der Container steht in der letzten Spalte der Ausgabe von `docker
ps`. Das Anzeigen der Logs funktioniert auch dann, wenn der Container
bereits beendet worden ist. Falls im Log steht, dass der Container
`killed` ist, dann hat Docker den Container wegen Speichermangel
beendet. Du solltest Docker mehr RAM zuweisen z.B. 4GB. Unter Windows
und macOS findet sich die RAM-Einstellung in der Docker application
unter Preferences/ Advanced.

Um einen Container genauer zu untersuchen, kannst du eine Shell in dem
Container starten. Beispielsweise mit `docker exec -it
msconsuldns_catalog_1 /bin/sh` oder du kannst in dem Container ein
Kommando mit `docker exec msconsuldns_catalog_1 /bin/ls` ausführen.

Du kannst auf die Microservices unter http://localhost:8080/
zugreigen, auf das Hystrix Dashboard unter http://localhost:8989/ und
auf das Consul Dashboard unter http://localhost:8500 .

Mit `docker-compose down` kannst Du alle Container beenden.

Wenn der Order-Container die anderen Container nicht finden kann,
schalte die Firewall auf dem Docker Host aus. Zumindest unter Mac OS X
unterbindet die Firewall die DNS-Zugriffe.
