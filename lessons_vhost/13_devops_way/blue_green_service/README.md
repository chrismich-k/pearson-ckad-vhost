edit the bgnginx-ing.yaml file and replace the MY_DOMAIN.TLD with your
actual domain before `kubectl apply -f bgnginx.yaml`.

it is assumed you setup Ingress (fixed WAN IP) and ClusterIssuer 
named `letsencryp-prod`.
