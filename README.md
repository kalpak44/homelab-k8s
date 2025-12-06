CLOUDFLARE_API_KEY
```shell
  kubectl create secret generic cloudflare-credentials \
    --from-literal=apiKey="XXXXXXXXXXXXXXXXXXXXXXXX-XXXX-XX" \
    -n traefik
```
