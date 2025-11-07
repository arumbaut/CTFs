Conectarnos

```
mysql -u seeddms -p -h 192.168.1.113

Nos topamos con este error 

ERROR 2026 (HY000): TLS/SSL error: self-signed certificate in certificate chain

Lo solventamos con este comando y logramos conectarnos
mysql -u seeddms -p -h 192.168.1.113 --ssl=0 
```

Ver las BD
```
MySQL [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| seeddms            |
| sys                |
+--------------------+
5 rows in set (0.007 sec)

MySQL [(none)]> 

```

Conectarnos  a la BD
```
MySQL [(none)]> use seeddms;
```

Ver las tablas de la BD a la cual nos conectamos
```
MySQL [seeddms]> show tables;
```

Seleccionar lo que queramos de las tablas
```
MySQL [seeddms]> select * from users;

```

Para ver como esta confomada una tabla de la BD
```
MySQL [seeddms]> describe tblUsers;
```