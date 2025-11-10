Primero haremos un escaneo en la red para ver los host activos 

[[Bash HostDisvovery]]

Vale destacar que veremos solo aquellos equipos que permitan hacer ping pero aquellos que tengan el firewall activo no podremos leerlos con estos metodos antes descrito por lo que recomiendo hacer un host discoered con nmap uno que nos permitira identificar estos host filtrados. Para los efectos de este entorno se deshabilitara el Defender en los equipos para facilitar el entendimiento de los ataques y no preocuparnos por el antivirus.

```
nmap -sn -PR 10.10.11.0/24
nmap -sn -PE 10.10.11.0/24
nmap -sn -PU 10.10.11.0/24
```