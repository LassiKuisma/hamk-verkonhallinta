# 1. Johdanto

SNMP on protokolla jolla voidaan hakea tietoja laitteesta, mm. käyttöjärjestelmä, muistin ja prosessorin käyttö, sekä verkkoliikennetilastoja.

# 2. Asennus

SNMP asennettiin jokaiselle kohdelaitteelle apt-työkalulla. Asennuksen jälkeen muokattiin konfiguraatiotiedostoa, ja palvelu restartattiin. Tämän jälkeen SNMP oli valmis käytettäväksi.

# 3. Kerätyt tiedot

Tehtävänannossa oleva komento ei toiminut, mutta kiitos luokkatoverin vinkin saimme komennot suoritettua näillä askelilla:

1. lisää `/etc/snmp/snmpd.conf` tiedostoon "agentaddress udp:161"
2. käytä OID:n numeerista versiota komennossa. Esimerkiksi:
   - `snmpwalk -v2c -c public web1 system` muuttuu
   - `snmpwalk -v2c -c public web1 .1.3.6.1.2.1.1`

| Laite         | Nimi          | Käyttöjärjestelmä                 | Uptime                        |
| ------------- | ------------- | --------------------------------- | ----------------------------- |
| web1          | web1          | Linux web1 7.2.2-arch1-1          | Timeticks: (16623) 0:02:46.23 |
| db1           | db1           | Linux db1 7.2.2-arch1-1           | Timeticks: (2805) 0:00:28.05  |
| branch-client | branch-client | Linux branch-client 7.2.2-arch1-1 | Timeticks: (1949) 0:00:19.49  |

# 4. Verkkorajapinnat

Rajapintojen listaus:

```
# snmpwalk -v2c -c public web1 .1.3.6.1.2.1.2.2.1.2
iso.3.6.1.2.1.2.2.1.2.1 = STRING: "lo"
iso.3.6.1.2.1.2.2.1.2.2 = STRING: "eth0"
iso.3.6.1.2.1.2.2.1.2.33 = STRING: "eth1"
```

Rajapintojen statuksien listaus (ifOperStatus):

```
# snmpwalk -v2c -c public web1 1.3.6.1.2.1.2.2.1.8
iso.3.6.1.2.1.2.2.1.8.1 = INTEGER: 1
iso.3.6.1.2.1.2.2.1.8.2 = INTEGER: 1
iso.3.6.1.2.1.2.2.1.8.33 = INTEGER: 1
```

# 5. OID-analyysi

| OID          | Tarkoitus                                     |
| ------------ | --------------------------------------------- |
| sysName.0    | laitteen nimi                                 |
| sysDescr.0   | laajemmat laitetiedot                         |
| sysUpTime.0  | kuinka kauan laite on ollut päällä            |
| ifDescr      | network interfacen tiedot                     |
| ifOperStatus | onko network interface saanut luotua yhteyden |

# 6. Pohdinta

SNMP helpottaa automaattista datan keruuta laitteilta. SNMP:n käyttö ja asennus on melko yksinkertainen, mutta saatava data ei ole ihmiselle helposti luettavassa muodossa, joten datan tulkitsemiseen tarvitaan jokin toinen ohjelma.
