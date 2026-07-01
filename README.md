# Connect with open-sshd

This GitHub Action opens an SSH connection to the github actions runner through an intermediary server.
It is intended to debug jobs through SSH.

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
    uses: edipizarro/runner-open-ssh@main
    with:
      sshPublicKey: ${{ secrets.SSH_PUBLIC_KEY }}
      gatewayPrivateKey: ${{ secrets.GATEWAY_PRIVATE_KEY }}
      gatewayIP: ${{ secrets.GATEWAY_IP }}
      gatewayUser: ${{ secrets.GATEWAY_USER }}
      maxLifeTime: 30
      port: 2222  # Optional: defaults to 2222
```

- **sshPublicKey**: Public SSH key that will be authorized in the github runner.
- **gatewayPrivateKey**: Private SSH Key that can access the Gateway server. The action will use this key to SSH to the gateway server and open a proxy on the specified port, consequently, the gateway server will be able to SSH with the runner.
- **gatewayIP** is the public IP of the gateway (used by the runner to ssh to the gateway).
- **gatewayUser** is the user for SSH on the gateway.
- **maxLifeTime** is the maximum life of the connection in seconds. default value is 3600 (1 hour). When the action SSH's to the gateway, it runs "sleep ${maxLifeTime}".
- **port**: The port to use for SSH connection. Optional, defaults to 2222.

After the job is run, the runner creates a proxy on the gateway on the specified port (defaults to 2222).
So, to connect to the runner, first ssh to the gateway server, and from there run "ssh -p &lt;port&gt; runner@localhost" (where &lt;port&gt; is the port you configured, or 2222 if using the default).

The ssh proxy connection is run as a background process (using the ampersand &). And to keep the action alive a tmux terminal is attached.

If you want to close the connection before the maxLifeTime (to continue to the next action or just to finish the job), execute 'tmux kill-session'. This will kill the tmux session and then the action will finish.
You can also just cancel the action.
