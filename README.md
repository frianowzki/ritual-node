# Ritual Infernet Node Tutorial

![meta](https://github.com/user-attachments/assets/c8cb65fd-a714-4b06-8ed1-65171f6eb13b)

#
## Hardware Requirements to run Ritual Infernet Node
#
![Screenshot 2024-12-11 144832](https://github.com/user-attachments/assets/979233a6-1e44-4361-bbd3-68d1aa2bb0b8)
#
- ## Update Packages & build tools
  
```
sudo apt update && sudo apt upgrade -y
```

```
sudo apt -qy install curl git jq lz4 build-essential screen
```

- ## Install Docker

```
sudo apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
```
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```
```
sudo docker run hello-world
```
- ## Install Docker Compose
```
sudo curl -L "https://github.com/docker/compose/releases/download/v2.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```
```
sudo chmod +x /usr/local/bin/docker-compose
```
```
DOCKER_CONFIG=${DOCKER_CONFIG:-$HOME/.docker}
mkdir -p $DOCKER_CONFIG/cli-plugins
curl -SL https://github.com/docker/compose/releases/download/v2.20.2/docker-compose-linux-x86_64 -o $DOCKER_CONFIG/cli-plugins/docker-compose
```
```
chmod +x $DOCKER_CONFIG/cli-plugins/docker-compose
```
```
docker compose version
```
```
sudo usermod -aG docker $USER
```
```
docker run hello-world
```
- ## Clone Starter Repository
```
git clone https://github.com/ritual-net/infernet-container-starter
```
```
cd infernet-container-starter
```
```
screen -S ritual
```
```
project=hello-world make deploy-container
```
If it run successfully, you can detach the screen by pressing ctrl A + D


Now let's open new Terminal

#
- ## Node Configuration

Copy this code and replace private keys with yours (add 0x before your PK if your PK don't have it)

```
{
    "log_path": "infernet_node.log",
    "server": {
        "port": 4000,
        "rate_limit": {
            "num_requests": 100,
            "period": 100
        }
    },
    "chain": {
        "enabled": true,
        "trail_head_blocks": 3,
        "rpc_url": "https://mainnet.base.org/",
        "registry_address": "0x3B1554f346DFe5c482Bb4BA31b880c1C18412170",
        "wallet": {
          "max_gas_limit": 4000000,
          "private_key": "your-private-keys-here-with-0x",
          "allowed_sim_errors": []
        },
        "snapshot_sync": {
          "sleep": 3,
          "batch_size": 800,
          "starting_sub_id": 160000,
          "sync_period": 30
        }
    },
    "startup_wait": 1.0,
    "redis": {
        "host": "redis",
        "port": 6379
    },
    "forward_stats": true,
    "containers": [
        {
            "id": "hello-world",
            "image": "ritualnetwork/hello-world-infernet:latest",
            "external": true,
            "port": "3000",
            "allowed_delegate_addresses": [],
            "allowed_addresses": [],
            "allowed_ips": [],
            "command": "--bind=0.0.0.0:3000 --workers=2",
            "env": {},
            "volumes": [],
            "accepted_payments": {},
            "generates_proofs": false
        }
    ]
}
```

Now edit these files, remove all texts and paste with the code above

```
nano ~/infernet-container-starter/deploy/config.json
```
```
nano ~/infernet-container-starter/projects/hello-world/container/config.json
```

ctrl + X > Y > Enter

Copy this code

```
// SPDX-License-Identifier: BSD-3-Clause-Clear
pragma solidity ^0.8.13;

import {Script, console2} from "forge-std/Script.sol";
import {SaysGM} from "../src/SaysGM.sol";

contract Deploy is Script {
    function run() public {
        // Setup wallet
        uint256 deployerPrivateKey = vm.envUint("PRIVATE_KEY");
        vm.startBroadcast(deployerPrivateKey);

        // Log address
        address deployerAddress = vm.addr(deployerPrivateKey);
        console2.log("Loaded deployer: ", deployerAddress);

        address registry = 0x3B1554f346DFe5c482Bb4BA31b880c1C18412170;
        // Create consumer
        SaysGM saysGm = new SaysGM(registry);
        console2.log("Deployed SaysHello: ", address(saysGm));

        // Execute
        vm.stopBroadcast();
        vm.broadcast();
    }
}
```

Open this

```
nano ~/infernet-container-starter/projects/hello-world/contracts/script/Deploy.s.sol
```

Remove all texts and paste with the code above

ctrl + X > Y > Enter

- ## Edit Makefile

Copy this code
```
# phony targets are targets that don't actually create a file
.phony: deploy

# anvil's third default address
sender := your-private-keys-here-with-0x
RPC_URL := https://mainnet.base.org/

# deploying the contract
deploy:
	@PRIVATE_KEY=$(sender) forge script script/Deploy.s.sol:Deploy --broadcast --rpc-url $(RPC_URL)

# calling sayGM()
call-contract:
	@PRIVATE_KEY=$(sender) forge script script/CallContract.s.sol:CallContract --broadcast --rpc-url $(RPC_URL)
```

Edit this 
```
nano ~/infernet-container-starter/projects/hello-world/contracts/Makefile
```

Remove all texts and paste with the code above

ctrl + X > Y > Enter

- ## Edit Node Version to ```1.4.0```
```
nano ~/infernet-container-starter/deploy/docker-compose.yaml
```

ctrl + X > Y > Enter

- ## Initialize & Applying New Configuration

Restart Docker Containers

```
docker compose -f infernet-container-starter/deploy/docker-compose.yaml down
```
```
docker compose -f infernet-container-starter/deploy/docker-compose.yaml up
```
#


- ## Install Foundry in a new Terminal

```
cd
```
```
mkdir foundry
```
```
cd foundry
```
```
curl -L https://foundry.paradigm.xyz | bash
```
```
source ~/.bashrc
```
```
foundryup
```


- ## Install required libraries and SDKs into our project

```
cd ~/infernet-container-starter/projects/hello-world/contracts
```
```
forge install --no-commit foundry-rs/forge-std
```
```
forge install --no-commit ritual-net/infernet-sdk
```
```
cd ../../../
```
#
- ## Note: If you having these issues check the fix here 
#
#### ```git submodule exited with code 128```

```
rm -rf projects/hello-world/contracts/lib/forge-std
```
```
forge install --no-commit foundry-rs/forge-std
```
```
cd ~/infernet-container-starter/projects/hello-world/contracts
```
```
rm -rf lib/forge-std
```
```
forge install --no-commit foundry-rs/forge-std
```
```
ls lib/forge-std
```
```
foundryup
```

#### ```ZOE ERROR (from forge): zoeParseOptions: unknown option (--no-commit)```

```
rm /usr/bin/forge
export PATH="/root/.foundry/bin:$PATH"
```
```
forge install --no-commit ritual-net/infernet-sdk
```

### ```infernet-sdk already exists and is not a valid git repo Error: git submodule exited with code 1```

```
cd ~/infernet-container-starter/projects/hello-world/contracts
```
```
rm -rf lib/infernet-sdk
```
```
forge install --no-commit ritual-net/infernet-sdk
```
```
ls lib/infernet-sdk
```

- ## Deploy Consumer Contract

Restart Docker Containers again from second Terminal we open earlier
```
docker compose -f infernet-container-starter/deploy/docker-compose.yaml down
```
```
docker compose -f infernet-container-starter/deploy/docker-compose.yaml up
```

Now we back again to third Terminal and Deploy a Contract
```
cd ~/infernet-container-starter
```
```
project=hello-world make deploy-contracts
```
#
#
#### If you see like this then congrats!
![Run-saysgm](https://github.com/user-attachments/assets/85e4f98f-03e6-47aa-98eb-268b8b26f78b)
#
Copy the Contract Address because we're gonna need it for next step. 
You also can check it on BaseScan https://basescan.org/
#
- ## Initiating a Request to Infernet Node / Call Contract

Edit ```CallContract.s.sol``` file by inserting the new Contract Address.
```
nano ~/infernet-container-starter/projects/hello-world/contracts/script/CallContract.s.sol
```
```SaysGM saysGm = SaysGM(change-with-your-contract-address)```

And it should be like 
```SaysGM saysGm = SaysGM(0x13D69Cf7d6CE4218F646B759Dcf334D82c023d8e)```

ctrl + X > Y > Enter

#
Now let's call it
```
project=hello-world make call-contract
```
#
#### And you should get like this 

![Call-Contract](https://github.com/user-attachments/assets/0380a06f-b36a-4586-a532-5e6f45e55d34)

#
#
### Congrats your Infernet Node is now running!
You can check if the containers are work now with

```
docker ps
```

And you can check the logs with 

```
docker logs infernet-node
```

Now if you check https://basescan.org/ you have 2 transactions now. 




