## For Docker 
install pdsh
```
apt-get update
apt-get install pdsh
```

environment setting 
```
export PDSH_SSH_ARGS_APPEND="-p 2222"
```


change permissions 

```
ls -ld /usr/lib/x86_64-linux-gnu/pdsh
ls -ld /usr/lib
```
```
chown root:root /usr/lib/x86_64-linux-gnu/pdsh
chown root:root /usr/lib
```

```
chmod 755 /usr/lib/x86_64-linux-gnu/pdsh
chmod 755 /usr/lib
```

and run 
```
deepspeed --no_local_rank --hostfile deepspeed_hostfile --launcher pdsh --master_port 5000 --ssh_port 2222 --module axolotl.cli.train qlora.yml
```
