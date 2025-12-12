Con una cuena del directorio activo sin ser administrador podemos enumerar el directorio activo 

```
$rpcclient -U "savicorp.local\mvazquez%P@ssword1" 192.168.94.136 -c "enumdomusers"

Obtine el Usuario y su RID
rpcclient -U "savicorp.local\mvazquez%P@ssword1" 192.168.94.136 -c "enumdomusers" | grep -oP "\[.*?]"


Obtener informacion de name|description para cada uno de los usuario encontrados utilizando el rid

for rid in $(rpcclient -U "savicorp.local\mvazquez%P@ssword1" 192.168.94.136 -c "enumdomusers" | grep -oP "\[.*?]" | grep '0x' | tr -d '[]');do echo -e "\n[+] Para el RID $rid\n";rpcclient -U "savicorp.local\mvazquez%P@ssword1" 192.168.94.136 -c "queryuser $rid" | grep -E -i "user name|description|" ;done

Para obtener toda la informacion del usuario 

for rid in $(rpcclient -U "savicorp.local\mvazquez%P@ssword1" 192.168.94.136 -c "enumdomusers" | grep -oP "\[.*?]" | grep '0x' | tr -d '[]');do echo -e "\n[+] Para el RID $rid\n";rpcclient -U "savicorp.local\mvazquez%P@ssword1" 192.168.94.136 -c "queryuser $rid" ;done
```