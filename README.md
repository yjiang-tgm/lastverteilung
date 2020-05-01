# Lastverteilung

## Auswahl Load-Balancer

#### Kriterien

* Open-Source
* Load-Balancing (mit mehreren Algorithmen zur Auswahl)
* Einfache Konfiguration (für Services in Containern)
* Läuft auf Docker

##### Traefik

Traefik ist ein Open-Source Load-Balancer, dass für die Nutzung in Docker-Containern entwickelt wurde. Die kontinuierliche Aktualisierung der Konfigurationsdatei ist ein großer Vorteil, wenn man mittels Docker Microservices in vielen Containers läuft. Traefik unterstützt HTTP, HTTPS-Verbindungen und auch TCP und TCP+TLS. Durch Docker werden die Services automatisch erkannt und Load-Balancing angewendet. Folgende Load-Balancing-Algorithmen werden unterstützt: Round-Robin, Weighted-Round-Robin und zusätzlich Sticky Sessions. [1]

##### HAProxy

HAProxy ist ein Open-Source Load-Balancer, dass für eine hohe Datenrate optimiert wurde. Es besitzt eine einfache Konfiguration und DNS-Service-Discovery. HAproxy unterstützt die typischen Protokolle: HTTP, HTTPS, TCP und TCP+TLS. HAProxy kann auf Docker laufen, besitzt jedoch im Gegensatz zu Traefik keine Docker Service-Discovery. Hingegen unterstützt HAProxy sehr viele Load-Balancing Algorithmen: Round-Robin, Least-Connections, Source, URI, Header und First [2]. Sticky Sessions wird auch unterstützt, damit eine Sitzung von einem Client zu behalten, indem er auf demselben Server umgeleitet wird.

<hr>

Da Traefik durch die Docker Service-Discovery sehr leicht konfigurieren lässt und zusätzlich sehr gut mit Microservices zusammenarbeitet, wurde dies als Load-Balancer verwendet. Es bietet zwar nicht sehr viele Load-Balancing Algorithmen zur Auswahl, jedoch reicht für unserem Deployment Round-Robin.

## Implementierung

##### Docker Swarm

Um Load-Balancing über mehrere Maschine zu ermöglichen, muss Docker Swarm auf denen initialisiert werden. Auf dem "Manager"-Node wurde folgendes ausgeführt:

````
docker swarm init
````

Um einen "Worker"-Node hinzuzufügen, muss der Token und die IP-Adresse eingegeben werden:

````
docker swarm join --token <token> <ip>:2377
````

##### Traefik

Auschnitt **docker-compose.yaml** für Traefik:

````yaml
version: '3'

services:
  reverse-proxy:
    # The official v2 Traefik docker image
    image: traefik:v2.2
    # Enables the web UI and tells Traefik to listen to docker
    command: --api.insecure=true --providers.docker --providers.docker.swarmMode=true
    ports:
      # The HTTP port
      - "80:80"
      # The Web UI (enabled by --api.insecure=true)
      - "8080:8080"
    volumes:
      # So that Traefik can listen to the Docker events
      - /var/run/docker.sock:/var/run/docker.sock
    deploy:
      placement:
        constraints:
          - node.role == manager
````

Die Standard-Konfiguration muss so geändert werden, dass der Treafik-Container auf dem Manager-Node läuft:

````yaml
placement:
        constraints:
          - node.role == manager
````

Zusätzlich muss der Docker-Provider und der Docker Swarm Modus aktiviert werden:

````yaml
command: --api.insecure=true --providers.docker --providers.docker.swarmMode=true
````

Um die Compose-Datei anzuwenden, muss man mittels ``docker stack`` diesem Service deployen:

````
docker stack deploy --compose-file docker-compose.yml mystack
````

##### GlusterFS

Gluster File System wird verwendet, damit Sessions persistieren. 

Installation:

````
sudo apt install glusterfs-server glusterfs-client
````

Um die Verbindung mit anderen Nodes zu testen:

````
sudo gluster peer probe 192.168.0.63
````

Bricks  (Auf alle Nodes):

````
sudo mkdir -p /gluster/brick 
````

Volume erstellen:

````
sudo gluster volume create swarm-gfs replica 3 transport tcp manager-ip:/gluster/brick node1-ip:/gluster/brick node2-ip:/gluster/brick force
````

Volume starten:

````
sudo gluster volume start swarm-gfs
````

Volume Information:

````
sudo gluster volume info
````

Mount:

````
sudo mount.glusterfs localhost:/swarm-gfs /mnt 
sudo chown -R yjiang01:docker /mnt
````

# Quellen

[1] Traefik, "Services," [Online]. Available: https://docs.traefik.io/routing/services/ [Accessed : 29.04.20]

[2] HAProxy, "Basic Features: Load Balancing," [Online]. Available: http://cbonte.github.io/haproxy-dconv/2.1/intro.html#3.3.5 [Accessed: 29.04.20]