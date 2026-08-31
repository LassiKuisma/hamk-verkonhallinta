## 1. Johdanto

Tämän ympäristön tarkoitus on tutkia verkon toimintaa ja rakennetta. Ympäristö sisältää eri tyyppisiä laitteita: clienttejä, palvelimia, reitittimiä ja erilaisia hallintatyökaluja.

## 2. Verkkokaavio

Lisää laatimasi verkkokaavio.

## 3. Laiteluettelo

| Laite         | Tarkoitus                            |
| ------------- | ------------------------------------ |
| r1            | reititin                             |
| r2            | reititin                             |
| r3            | reititin                             |
| client1       | client                               |
| attacker      | client                               |
| web1          | palvelin                             |
| db1           | palvelin/tietokanta                  |
| branch-client | client                               |
| ansible       | hallinta/asennustyökalu palvelimille |
| prometheus    | monitorointityökalu                  |
| grafana       | datan visualisointi                  |
| zabbix        | monitorointityökalu                  |

## 4. IP-suunnitelma

| Verkko         | Tarkoitus                                       | Yhdyskäytävä |
| -------------- | ----------------------------------------------- | ------------ |
| 10.10.10.0/24  | clientit (client1 ja attacker1)                 | 172.20.20.1  |
| 10.10.20.0/24  | palvelimet (web1 ja db1)                        | 172.20.20.1  |
| 10.10.30.0/24  | branch client                                   | 172.20.20.1  |
| 10.10.99.0/24  | työkalut (ansible, prometheus, grafana, zabbix) |              |
| 10.255.12.0/30 |                                                 |              |
| 10.255.23.0/30 |                                                 |              |

## 5. Reitityksen analyysi

Sisällytä:

- ping-testit
- traceroute
- reittitauluanalyysi

## 6. Yhteenveto

Pohdi:

- Mitkä asiat verkon dokumentaation muodostamisessa kuluttivat eniten aikaa ja miksi?
- Miten dokumentaatio mielestäsi auttaa palvelusta vastaavaa it-asiantuntijaa työssään?
