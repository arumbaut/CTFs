```
sudo groupadd promiscuous
sudo usermod -a -G promiscuous <your_user_id>   

sudo chgrp promiscuous /dev/vmnet* 
sudo chmod g+rw /dev/vmnet*
```