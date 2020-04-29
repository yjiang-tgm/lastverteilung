# Lastverteilung

## Auswahl Load-Balancer

#### Kriterien

* Open-Source
* Load-Balancing (mit mehreren Algorithmen zur Auswahl)
* Einfache Konfiguration (für Services in Containern)
* Läuft auf Docker

##### Traefik

Traefik ist ein Open-Source Load-Balancer, dass für die Nutzung in Docker-Containern entwickelt wurde. Die kontinuierliche Aktualisierung der Konfigurationsdatei ist ein großer Vorteil, wenn man mittels Docker Microservices in vielen Containers läuft. Traefik unterstützt HTTP, HTTPS-Verbindungen und auch TCP und TCP+TLS. Durch Docker werden die Services automatisch erkannt und Load-Balancing angewendet. Folgende Load-Balancing-Algorithmen werden unterstützt: Round-Robin, Weighted-Round-Robin und zusätzlich Sticky Sessions.

##### HAProxy

HAProxy ist ein Open-Source Load-Balancer, dass für eine hohe Datenrate optimiert wurde. Es besitzt eine einfache Konfiguration und DNS-Service-Discovery. HAproxy unterstützt die typischen Protokolle: HTTP, HTTPS, TCP und TCP+TLS. HAProxy kann auf Docker laufen, besitzt jedoch im Gegensatz zu Traefik keine Docker Service-Discovery. Hingegen unterstützt HAProxy sehr viele Load-Balancing Algorithmen: Round-Robin, Least-Connections, Source, URI, Header, First. Sticky Sessions wird auch unterstützt, damit eine Sitzung von einem Client zu behalten, indem er auf demselben Server umgeleitet wird.

