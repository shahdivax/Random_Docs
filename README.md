### Configure sshd_config 
> Inside ~/etc/ssh/
	> ``sudo nano sshd_config``
	> uncomment public key auth on
	> do this on all the nodes and server 
### Generate public key
> ``ssh-keygen -t rsa`` (don't set any passphrase when asked)
> copy the public key ``cat ~/.ssh/id_rsa.pub``
> paste it to authorized keys for passwordless  access ``nano ~/.ssh/authorized_keys``
> Now repeat this steps in node 2,3,...
> now for example of 2 node (server and node):
	- paste the generated key of node 1 in authorized keys of 1 and 2 
	- paste the generated key of node 2 in authorized keys of 1 and 2
> this will set up passwordless ssh on both sides
> now you can check passwordless ssh buy :
> inside node 1 do ``ssh <ip-node2>`` , it should open node2 inside node 1 withput error.  
### axolotl 
> now configure axolotl as usual on each node with has same files ``.yml`` and everything same  
> create a hostfile inside axolotl folder using ``nano deepspeed_hostfile`` and include something like below 

    <ip-node-1> slots=<num_gpu_in_node1>
	<ip-node-2> slots=<num_gpu_in_node2> 
	.
	.

### accelerate Config 
> example of 2 nodes, both node has 4xV100, below are the inputs I gave while config. accelerate (in sequence) of node one : 
This machine
multi_gpu 
2
0 (for first node ie. server)
<server_ip>
5000
no
static
yes
no
yes
yes
deepspeed_config/zero2.json
no
pdsh
deepspeed_hostfile
no
no
8
> below is the accelerate config file looks like for me for node 1 :

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

 
> below are the inputs I gave while config. accelerate (in sequence) of node two : 
This machine
multi_gpu 
2
1 (for second node)
<server_ip>
5000
no
static
yes
no
yes
yes
deepspeed_config/zero2.json
no
pdsh
deepspeed_hostfile
no
no
8
> below is the accelerate config file looks like for me for node 2:
>  
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
### Finetuning
> now inside axolotl inside node 1 (server), you can run the finetunining process
> eg : ``accelerate launch -m axolotl.cli.train examples/llama-2/qlora.yml``  
> this will start finetuning process and you can check different ips befor steps to see that its running on every node. 
