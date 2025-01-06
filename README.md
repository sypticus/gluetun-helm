# Gluetun VPN client

This is a helm chart for [Gluetun](https://github.com/qdm12/gluetun).
This is helm chart is still a project under construction, and is definitely beta.

Initially, this chart sets up a VPN proxy which can be reached from inside the K8s cluster, 
or from anywhere on the LAN if you add a loadbalancer.

https://github.com/sypticus/gluetun-helm.git

## Installing the Chart as a basic Mullvad OpenVPN proxy.

```console
git clone https://github.com/sypticus/gluetun-helm.git
helm install gluetun-proxy ./gluetun-helm --set vpn.openvpn.user=$MULLVAD_ACCOUNT_NUMBER -n proxy --create-namespace`
```

This may take a minute or two to start up. 

To confirm vpn proxy, port forward the pod and use it as a proxy
```console
POD=$(kubectl get pods -l app.kubernetes.io/instance=gluetun-proxy  -o jsonpath="{.items[0].metadata.name}" -n proxy)
kubectl -n proxy port-forward $POD 8888

#In another window
curl icanhazip.com --proxy localhost:8888
```
You should see a new IP address.
Check the pod logs for any failures if not.


In order to set the correct values for your instance, create a values.yaml to override the defaults.
This will also allow you to create a service that you can forward calls to.

```console
helm install gluetun-proxy ./gluetun-helm -f my_values.yaml
```


## Uninstalling the Chart

```console
helm delete gluetun-proxy
```

## Configuration

Documentation of configs can be found on the Gluetun wiki  (https://github.com/qdm12/gluetun-wiki/)
Further info can be found in the included values.yaml https://github.com/sypticus/gluetun-helm/


The following are the config values which can be set in the helm chart.
All other config values can be passed in as environmental variables using the `vpn.extra_env` field.


| Parameter                   | Description                                                            | Default                 |
|-----------------------------|------------------------------------------------------------------------|-------------------------|
| `image.repository`          | Image repository                                                       | `ghcr.io/qdm12/gluetun` |
| `image.tag`                 | Image tag                                                              | `v3.40.0`               |
| `image.pullPolicy`          | Image pull policy                                                      | `IfNotPresent`          |
| `service.type`              | Kubernetes service type                                                | `ClusterIP`             |
| `service.httpPort`          | Port to be used for proxying http(s) calls.                            | `8888`                  |
| `service.loadBalancerIP`    | If type is LoadBalancer, set the IP                                    |                         |
| `vpn.provider`              | VPN provider name                                                      | `mullvad`               |
| `vpn.type`                  | wireguard or openvpn                                                   | `openvpn`               |
| `vpn.openvpn.secret_name`   | k8s secret with openvpn credentials (openvpnuser='', openvpnpasswd='') |                         |
| `vpn.openvpn.user`          | inline openvpn username                                                |                         |
| `vpn.openvpn.password`      | inline openvpn password                                                |                         |
| `vpn.wireguard.addresses`   | Addresses given by your Wireguard provider                             |                         |
| `vpn.wireguard.secret_name` | k8s secret with wireguard private key (wgprivatekey='')                |                         |
| `vpn.wireguard.private_key` | inline wireguard private key                                           |                         |
| `vpn.dnsAddress`            | Preferred DNS address                                                  | `10.64.0.1`             |
| `vpn.provider_extra_env`    | any extra required env vars for provider                               | `[]`                    |
| `proxy.enabled`             | Set mullvad to proxy mode                                              | `true`                  |
| `proxy.secret_name`         | k8s secret proxy credentials (proxyuser='', proxypasswd='')            |                         |
| `proxy.user`                | inline proxy username                                                  |                         |
| `proxy.password`            | inline proxy password                                                  |                         |
| `proxy.stealth`             | prevent proxy headers for being added to request                       | `true`                  |
| `proxy.log`                 | log every tunnel request                                               | `false`                 |
| `proxy.listening_address`   | internal listening address                                             | `:8888`                 |
