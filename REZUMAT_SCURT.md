# Rezumat Scurt - Proiect CrowdSec + PostgreSQL HA

## Ce s-a implementat
- Cluster HA pe 2 containere LXC: `crowdsec1 (10.10.100.171)` si `crowdsec2 (10.10.100.172)`.
- PostgreSQL HA cu Patroni (Raft intern) + Keepalived (VIP `10.10.100.21`).
- CrowdSec configurat cu backend PostgreSQL prin VIP.
- Deploy complet automatizat prin Ansible (roluri + playbook-uri + verificari).

## Timp estimat
- Aproximativ 2-3 ore (configurare, ajustari, troubleshooting iterativ).

## Probleme principale intampinate
- Incompatibilitati Ansible/Python (`apt_key`, `get_url`, PEP 668).
- Bootstrap Patroni blocat (quorum/ordine rulare, system ID mismatch dupa clone LXC).
- Incompatibilitati schema config CrowdSec intre versiuni.
- Permisiuni PostgreSQL insuficiente pe schema `public` pentru user-ul CrowdSec.
- Conditii de startup in lant (VIP/CrowdSec/Keepalived), rezolvate prin ordinea corecta a rolurilor.

## Rezultat final
- Deploy-ul ruleaza cu succes (`failed=0` pe ambele noduri).
- Patroni functional: 1 leader + 1 replica.
- Cluster pregatit pentru failover automat.
