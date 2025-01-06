---
title: "SSH Port Forwarding"
description: "Port forwarding using SSH: Setup and use cases"
date: 2025-01-06
categories: [
    Networking,
    SSH
]
tags: [
    SSH,
    Port Forwarding,
    Remote Access,
]
# layout: default
---

> SSH (Secure Shell) port forwarding is a powerful technique that allows you to securely forward traffic from one machine to another over an encrypted SSH connection. It allows you to `bypass firewalls`, access internal services from the outside world, or securely connect to a `remote service` as if it were on your `local machine`. SSH port forwarding can be divided into remote-to-local and local-to-remote types, depending on the direction of traffic flow.
{: .prompt-tip }

### **Types of SSH Port Forwarding**

1. **Remote-to-Local Port Forwarding (`R`)**
2. **Local-to-Remote Port Forwarding (`L`)**

---
## 1. Remote to local
- Forward a remote machine's port to your local machine using SSH with the `-R` option.
```bash
ssh -R [remote_port]:[local_address]:[local_port] [user]@[remote_host]
```

### Example
- Lets say you are serving an app locally, on 8080.
- You have an AWS instance with a public IP 13.123.12.51.
- You can use ssh remote to local port forwarding via which others can access http://localhost:8080 through http://13.123.12.51:8080 .


## 2. Local to remote
- Forward a local port to your remote machine using SSH with the `-L` option.
```bash
ssh -L [local_port]:[local_address]:[remote_port] [user]@[remote_host]
```

### Example
- Lets say you have an app on AWS serving on port 8080.
- You want to view that app via http://localhost:8080, so you can use ssh local to remote port forwarding via which you can access another resource as if it was served on your own machine.

---

## 3. Accessing a local service from a remote machine
- To enable port forwarding on the server side, you need to set the `GatewayPorts` option to `yes` in the SSH server configuration file.

**/etc/ssh/sshd_config**
```bash
Port 22 # Default SSH port
ListenAddress 0.0.0.0 # Listen on all interfaces
AllowTcpForwarding yes # Allow TCP forwarding
GatewayPorts yes # Allow remote hosts to connect to forwarded ports
```

### Example (Rocket served locally access via AWS)
![Locally hosted Rocket app](/assets/api/rust/rocket/rocket_all_interfaces.png)
- Basic boilerplate rocket
- Make sure the app is served on 0.0.0.0 and not your loopback address
- Establish the SSH tunnel
```bash
ssh -i ~/Vaults/tlprtrx_vault/aws/c2_autoarch_keypair.pem -R 8000:localhost:8000 -N -f ubuntu@3.110.43.104
```

![Rocket SSH Port forwarded](/assets/networking/ssh/rocket_ssh_pfwd.png)
- Make sure the sshd configuration is set up to listen on 0.0.0.0
- Check the connection using `sudo netstat -tulnp | grep 8000`
- Access the local app (port forwarded) via AWS. So the traffic is forwarded from the instance to the the machine hosting the local app connected via SSH.

![Rocket Hello, World!](/assets/api/rust/rocket/rocket_hello_world.png)

## 4. Connect to DocDB from outside Amazon VPC
[ec2-bastion-host-ssh](https://repost.aws/knowledge-center/documentdb-ec2-bastion-host-ssh)
- Use the following simple ssh tunnel script to port forward `27017` to your DocDB cluster via a bastion host in the Amazon VPC.
- The associated key file needs to be placed in the same directory and the ssh forwarding will run in the background on `connect`. The name of the key file needs to be replaced in the script.

**ssh-tunnel.sh**
```bash
#!/bin/bash

# Check if the first argument is provided (either 'connect' or 'disconnect')
if [[ -z "$1" ]]; then
    echo "Error: Missing first argument. Usage: $0 {connect|disconnect} <IP_ADDRESS>"
    exit 1
fi

# If the first argument is 'connect', check if the second argument (IP address) is provided
if [[ "$1" == "connect" ]]; then
    # Check if the second argument (IP address) is provided
    if [[ -z "$2" ]]; then
        echo "Error: No IP address provided for connection. Usage: $0 connect <IP_ADDRESS>"
        exit 1
    fi

    # Validate if the second argument is a valid IP address (basic regex check for IPv4)
    if ! [[ "$2" =~ ^[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+$ ]]; then
        echo "Error: Invalid IP address format. Please provide a valid IPv4 address."
        exit 1
    fi

    # SSH command, replace <IP_ADDRESS> with the second argument ($2)
    SSH_CMD="ssh -i <keyname>.pem -L 27017:<someDB>-db-cluster.cluster.ap-south-1.docdb.amazonaws.com:27017 ubuntu@$2 -N -f"

    # Check if tunnel is already connected
    if pgrep -f "$SSH_CMD" > /dev/null; then
        echo "SSH Tunnel is already connected."
    else
        echo "Starting SSH Tunnel..."
        # Start the SSH tunnel and store the PID
        $SSH_CMD
        echo "SSH Tunnel connected."
    fi

elif [[ "$1" == "disconnect" ]]; then
    echo "Disconnecting SSH Tunnel..."

    # Find the SSH tunnel process and kill it
    pid=$(ps aux | grep 'ssh -i' | grep -v 'grep' | grep '\-L' | awk '{print $2}')
    if [[ -n "$pid" ]]; then
        kill "$pid"
        echo "SSH Tunnel disconnected."
    else
        echo "No SSH tunnel found to disconnect."
    fi

else
    echo "Usage: $0 {connect|disconnect} <IP_ADDRESS>"
    exit 1
fi
```