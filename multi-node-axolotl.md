## Configure sshd_config 
- Inside ~/etc/ssh/ <br>
	 ~ open sshd_config using ``sudo nano sshd_config`` <br>
	 ~ uncomment public key auth on <br>
	 ~ do this on all the nodes and server <br>
## Generate public key
- Generate key using ``ssh-keygen -t rsa`` (don't set any passphrase when asked) <br>
- copy the public key ``cat ~/.ssh/id_rsa.pub`` <br>
- paste it to authorized keys for passwordless  access ``nano ~/.ssh/authorized_keys`` <br>
- Now repeat these steps in node 2,3,... <br>
- now for an example of 2 nodes (server and node): <br>
	~ paste the generated key of node 1 in authorized keys of 1 and 2  <br>
	~ paste the generated key of node 2 in authorized keys of 1 and 2 <br>
- this will set up passwordless ssh on both sides <br>
- now you can check passwordless ssh by: <br>
  ~ inside node 1 do ``ssh <ip-node2>`` , it should open node2 inside node1 without error.  
## axolotl 
- now configure axolotl as usual on each node which has same files ``.yml`` and everything is the same  
create a hostfile inside axolotl folder using ``nano deepspeed_hostfile`` and include something like below 
```
<ip-node-1> slots=<num_gpu_in_node1>
<ip-node-2> slots=<num_gpu_in_node2> 
.
.
```
## accelerate Config 
- example of 2 nodes, both node has 4xV100, below are the inputs I gave while config. accelerate (in sequence) of node one :
- do ``accelerate config`` and fill inputs as below
> This machine <br>
> multi_gpu <br>
> 2 <br>
> 0 (for first node ie. server) <br>
> <server_ip> <br>
> 5000 <br>
> no <br>
> static <br>
> yes <br>
> no <br>
> yes <br>
> yes <br>
> deepspeed_config/zero2.json <br>
> no <br>
> pdsh <br>
> deepspeed_hostfile <br>
> no <br>
> no <br>
> 8 <br>

- below is the accelerate config file that looks like this for me for node 1 :
 ``` 
    compute_environment: LOCAL_MACHINE
    debug: true
    deepspeed_config:
      deepspeed_config_file: deepspeed_config/zero2.json
      deepspeed_hostfile: deepspeed_hostfile
      deepspeed_multinode_launcher: pdsh
      zero3_init_flag: false
    distributed_type: DEEPSPEED
    downcast_bf16: 'no'
    machine_rank: 0
    main_process_ip: 34.220.20.212
    main_process_port: 5000
    main_training_function: main
    num_machines: 2
    num_processes: 8
    rdzv_backend: static
    same_network: false
    tpu_env: []
    tpu_use_cluster: false
    tpu_use_sudo: false
    use_cpu: false 
```
 
- below are the inputs I gave while config. accelerate (in sequence) of node two :
- - do ``accelerate config`` and fill inputs as below
> This machine <br>
> multi_gpu <br>
> 2 <br>
> 1 (for second node) <br> 
> <server_ip> <br>
> 5000 <br>
> no <br>
> static <br>
> yes <br>
> no <br>
> yes <br>
> yes <br>
> deepspeed_config/zero2.json <br>
> no <br>
> pdsh <br> 
> deepspeed_hostfile <br>
> no <br>
> no <br>
> 8 <br>

- below is the accelerate config file that looks like this for me for node 2:
```  
    compute_environment: LOCAL_MACHINE
    debug: true
    deepspeed_config:
      deepspeed_config_file: deepspeed_config/zero2.json
      deepspeed_hostfile: deepspeed_hostfile
      deepspeed_multinode_launcher: pdsh
      zero3_init_flag: false
    distributed_type: DEEPSPEED
    downcast_bf16: 'no'
    machine_rank: 1
    main_process_ip: 34.220.20.212
    main_process_port: 5000
    main_training_function: main
    num_machines: 2
    num_processes: 8
    rdzv_backend: static
    same_network: false
    tpu_env: []
    tpu_use_cluster: false
    tpu_use_sudo: false
    use_cpu: false
```
## Finetuning
- now inside axolotl inside node 1 (server), you can run the finetunining process
- eg: ``accelerate launch -m axolotl.cli.train examples/llama-2/qlora.yml``  
- this will start the finetuning process and you can check different IPs before steps to see that it's running on every node. 
