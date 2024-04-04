```markdown
# Distributed Finetuning with Axolotl

This guide explains how to set up a distributed finetuning environment using Axolotl, a framework for finetuning large language models. The setup involves configuring SSH for passwordless access, generating public keys, and configuring Axolotl and Accelerate for multi-node training.

## Prerequisites

- Multiple nodes (servers or machines) with GPUs
- Ubuntu or any other Linux distribution

## Step 1: Configure SSH for Passwordless Access

1. Open the `sshd_config` file on all nodes and the server:

```bash
sudo nano ~/etc/ssh/sshd_config
```

2. Uncomment the `PubkeyAuthentication` line to enable public key authentication.
3. Save the changes and restart the SSH service:

```bash
sudo systemctl restart sshd
```

## Step 2: Generate Public Key

1. Generate a public key using `ssh-keygen`:

```bash
ssh-keygen -t rsa
```

2. Copy the public key to the clipboard:

```bash
cat ~/.ssh/id_rsa.pub
```

3. Add the public key to the `authorized_keys` file on all nodes and the server:

```bash
nano ~/.ssh/authorized_keys
```

4. Repeat steps 1-3 on all other nodes.
5. Exchange public keys between nodes:
   - Paste the public key of Node 1 into the `authorized_keys` file of Node 2, and vice versa.
   - Repeat this process for all node pairs.

6. Test passwordless SSH access:

```bash
ssh <ip-of-other-node>
```

## Step 3: Configure Axolotl

1. Configure Axolotl on each node with the same `.yml` files and settings.
2. Create a `deepspeed_hostfile` inside the Axolotl folder:

```bash
nano deepspeed_hostfile
```

3. Add the IP addresses and GPU slots for each node:

```
<ip-node-1> slots=<num_gpu_in_node1>
<ip-node-2> slots=<num_gpu_in_node2>
```

## Step 4: Configure Accelerate

Follow these steps on each node to configure Accelerate:

```bash
accelerate config
```

### Node 1 (Server) Configuration

1. Select `This machine` for the compute environment.
2. Select `multi_gpu` for the compute type.
3. Enter the number of machines (e.g., `2` for two nodes).
4. Enter the machine rank `0` for the first node (server).
5. Enter the server IP address.
6. Enter the main process port (e.g., `5000`).
7. Select `no` for setting up custom environment variables.
8. Select `static` for the rendezvous backend.
9. Select `yes` for running on the same network.
10. Select `no` for using a cluster.
11. Select `yes` for using Deepspeed.
12. Select `yes` for using Deepspeed configs.
13. Enter the Deepspeed config file (e.g., `deepspeed_configs/zero2.json`).
14. Select `no` for using Zero 3.
15. Enter `pdsh` for the Deepspeed multinode launcher.
16. Enter `deepspeed_hostfile` for the Deepspeed hostfile.
17. Select `no` for using custom launch utility options.
18. Select `no` for using a TPU.
19. Enter the number of processes (e.g., `8` for 8 GPUs).

### Node 2 Configuration

Repeat the above steps for Node 2, but change the machine rank to `1`.

## Step 5: Finetuning

On Node 1 (server), run the finetuning process using Accelerate:

```bash
accelerate launch -m axolotl.cli.train examples/llama-2/qlora.yml
```

This will start the finetuning process across all nodes. You can check the different IP addresses before each step to verify that the training is running on every node.
