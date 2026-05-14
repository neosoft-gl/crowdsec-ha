# Rezumat Cu Detalii - Proiect CrowdSec + PostgreSQL HA

## Context
S-a construit un stack HA pe 2 containere LXC:
- `crowdsec1` - `10.10.100.171`
- `crowdsec2` - `10.10.100.172`
- VIP Keepalived - `10.10.100.21`

Obiectivul a fost deploy complet prin Ansible pentru:
- PostgreSQL HA (Patroni + Raft intern)
- CrowdSec cu backend PostgreSQL
- Keepalived pentru VIP si failover

## Ce s-a livrat
- Structura Ansible completa (`roles/`, `inventory/`, `group_vars/`, `site.yml`).
- Roluri separate pentru `common`, `postgresql`, `keepalived`, `crowdsec`.
- Playbook-uri de operare si verificare (`check_status.yml`, `patroni_ops.yml`).
- Documentatie operationala in `README.md`.

## Probleme intampinate si cum au fost rezolvate
1. **Ansible config / callback incompatibil**
- Eroare legata de `community.general.yaml`.
- Rezolvare: trecere la callback `default` + `result_format=yaml`.

2. **Module Ansible HTTPS incompatibile in runtime-ul remote**
- `apt_key`/`get_url` au esuat cu erori legate de SSL (`cert_file`).
- Rezolvare: import chei GPG prin `curl | gpg --dearmor` in `/etc/apt/keyrings`.

3. **PEP 668 (pip blocat in system Python)**
- Instalarea Patroni cu `pip3` global a fost refuzata.
- Rezolvare: virtualenv dedicat (`/opt/patroni-venv`) + servicii/operatii mutate pe binarele din venv.

4. **Bootstrap Patroni instabil pe noduri clonate LXC**
- Timeout la pornire + `system ID mismatch`.
- Rezolvare:
  - rulare paralela pentru PostgreSQL/Patroni (fara `serial: 1`),
  - mecanism de `patroni_force_reinit=true` pentru curatare raft + PGDATA la bootstrap,
  - validari/loguri mai clare la start.

5. **Config CrowdSec incompatibil cu versiunea instalata**
- Campuri invalide in `config.yaml` (`pid_file`, `flush_ca`).
- Rezolvare: simplificare config la schema compatibila.

6. **CrowdSec nu putea crea schema in PostgreSQL**
- `permission denied for schema public`.
- Rezolvare: grant-uri explicite pe schema `public`, ownership DB, default privileges.

7. **Race condition la initializarea CrowdSec pe ambele noduri**
- Eroare SQL tip duplicate in schema bootstrap.
- Rezolvare: rulare `Deploy CrowdSec` cu `serial: 1`.

## Timp estimat
- Aproximativ 2-3 ore efective, incluzand deploy initial, multiple iteratii de debugging si hardening al playbook-urilor.

## Stare finala
- Deploy final reusit pe ambele noduri (`failed=0`).
- Verificare PostgreSQL:
  - `crowdsec1` -> `pg_is_in_recovery() = f` (PRIMARY)
  - `crowdsec2` -> `pg_is_in_recovery() = t` (REPLICA)
- Arhitectura HA functionala, cu failover automat pregatit.
