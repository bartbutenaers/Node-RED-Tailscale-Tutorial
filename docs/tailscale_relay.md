# Tailscale relay

When the Tailscale agents on two of your devices cannot communicate directly to each other (due to heavy firewall restrictions), they can also connect via a worldwide cluster of Tailscale relay DERP servers:

![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/5938796a-d9e3-4047-ab43-40490721defe)

Such a relay connection can be temporarily or permanently.  Of course it is better to have a direct connection between your Tailscale agents, because direct connections will be faster.  Via the `tailscale netcheck` command, you can see very easily the network latency to all the available Tailscale relay DERP servers:

![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/0e320718-cbca-4cab-b3f9-c47b344cb4e3)

Note that you can use `tailscale ping some-device-name` to check whether there is a DERP connection used towards that specifed device, which tells whether it is a direct connection or not.
