edit the old-ing.yaml / new-ing.yaml file and replace MY_DOMAIN.TLD with your
actual domain before e.g. `kubectl apply -f old-ing.yaml`.

it is assumed you setup Ingress (fixed WAN IP) and ClusterIssuer 
named `letsencryp-prod`.
