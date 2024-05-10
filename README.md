# Connect with open-sshd

This GitHub Action sets up a SSH Server through a gateway.

## Pre-requisites
The gateway server has to be accessible from internet.
The gateway ssh server configuration (usually in /etc/ssh/sshd_config) needs the following configuration
```
AllowTcpForwarding yes
GatewayPorts yes
```

## How to use

First add the action to your workflow.
```yaml
  - name: Debug through SSH
    uses: edipizarro/open-sshd@main
    with:
      sshPublicKey: ${{ secrets.SSH_PUBLIC_KEY }}
      gatewayPrivateKey: ${{ secrets.GATEWAY_PRIVATE_KEY }}
      gatewayIP: ${{ secrets.GATEWAY_IP }}
      maxLifeTime: 30
```

- **sshPublicKey**: Public SSH key that will be authorized in the github runner.
- **gatewayPrivateKey**: Private SSH Key that can access the Gateway server. The action will use this key to SSH to the gateway server and open a proxy on port 2222, consequently, the gateway server will be able to SSH with the runner.
- **gatewayIP** is the public IP of the gateway (used by the runner to ssh to the gateway)
- **maxLifeTime** is the maximum life of the connection in seconds. default value is 3600 (1 hour). When the action SSH's to the gateway, it runs "sleep ${maxLifeTime}".

After the job is run, the runner creates a proxy on the gateway on port 2222.
So, to connect to the runner, first ssh to the gateway server, and from there run "ssh -p 2222 runner@localhost".

The ssh proxy connection is run as a background process (using the ampersand &). And to keep the action alive a tmux terminal is attached.

If you want to close the connection before the maxLifeTime (to continue to the next action or just to finish the job), execute 'tmux kill-session'. This will kill the tmux session and then the action will finish.
You can also just cancel the action.
