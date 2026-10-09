# Johdanto

Infrastructure as Code mahdollistaa asetuksien, configuraatioiden, asennettujen ohjelmien ja muun vastaavan tiedon tallentamisen tekstimuotoon, mikä taas mahdollistaa sen viemisen versionhallintajärjestelmään. Näin ollen palvelimen "tila" (tai askeleet nykyiseen tilaan pääsemiseen) voidaan helposti varmuuskopioida ja ajaa automaattisesti uudelle laitteelle, tai vikatilassa palauttaa palvelin aiempaan, toimivaan tilaan.

# Inventory

Inventoryssä olevat laitteet ja ryhmät nähdään komennolla:

```
# ansible-inventory -i /ansible/inventory.ini --graph
@all:
  |--@ungrouped:
  |--@user_network:
  |  |--client1
  |  |--attacker
  |--@server_network:
  |  |--web1
  |  |--db1
  |--@branch_office:
  |  |--branch-client
  |--@network_devices:
  |  |--@routers:
  |  |  |--r1
  |  |  |--r2
  |  |  |--r3
  |--@linux_hosts:
  |  |--@clients:
  |  |  |--client1
  |  |  |--attacker
  |  |  |--branch-client
  |  |--@servers:
  |  |  |--web1
  |  |  |--db1
  |  |--@monitoring:
  |  |  |--prometheus
  |  |  |--grafana
  |  |  |--zabbix
  |  |  |--cadvisor
  |  |--@management:
  |  |  |--ansible
  |--@ubuntu_hosts:
  |  |--client1
  |  |--web1
  |  |--db1
  |  |--branch-client
  |--@node_exporter:
  |  |--@routers:
  |  |  |--r1
  |  |  |--r2
  |  |  |--r3
  |  |--@clients:
  |  |  |--client1
  |  |  |--attacker
  |  |  |--branch-client
  |  |--@servers:
  |  |  |--web1
  |  |  |--db1
```

Inventory jaetaan ryhmiin jotta samanlaisille laitteille voidaan asentaa niiden käyttötarkoitukseen sopivat ohjelmat ja configuraatiot. Esimerkiksi servers-ryhmässä olevat laitteet luultavasti tarvitsevat erilaisia palvelinpuolen asetuksia ja ohjelmia, mutta näitä ei ole järkevää asentaa kaikille laitteille (kuten client1:lle).

# Esimerkkiplaybookit

Projektissa oli kaksi esimerkkiplaybookkia: install-snmp.yml ja install-node-exporter.yml. Kuten playbookkien nimistä voidaan päätellä, näiden tehtävä oli asentaa snmp ja node-exporter ohjelmat koneille. Molempien playbookkien rakenne oli melko samanlainen:

1. playbookin alussa määritellään playbookin nimi ja kohdepalvelimet/-ryhmät
2. määritellään muttujia, kuten haluttu ohjelmaversio
   - tämä helpottaa koodin lukemista ja vian etsintää, kun kaikki configuraatiot ovat yhdessä paikassa, eikä pitkin tiedostoa
3. asennetaan haluttu ohjelma
   - asennus voidaan hoitaa mm. apt:lla, tai lataamalla ohjelma esimerkiksi GitHubista
4. muokataan konfiguraatioita
5. (re)startataan palvelu, ja varmistetaan että se on käynnissä

# Oma playbook

```yml
---
- name: Install nginx
  hosts: web1
  become: true

  tasks:
    - name: Update package cache
      apt:
        update_cache: yes

    - name: Install Nginx
      apt:
        name: nginx
        state: present

    - name: Create basic HTML page
      copy:
        dest: /var/www/html/index.html
        content: '<html><body>Hello, my name is {{ inventory_hostname }}</body></html>'

    - name: Ensure nginx is running
      service:
        name: nginx
        state: started
        enabled: yes
        # taskissa pitää olla "use: ...", muuten taski ei toimi
        # https://forum.ansible.com/t/ansible-in-a-docker-container-service-is-in-unknown-state-error/2006
        use: 'a magic spell'

    - name: Get website content
      get_url:
        url: 'http://web1'
        dest: /tmp/index.html

    - name: Ensure website contains server name
      lineinfile:
        path: /tmp/index.html
        line: 'web1'
```

# Vertailu ja analyysi

Käsin tehtävissä asennuksissa voi olla monia riskitekijöitä: admin saattaa vahingossa ottaa yhteyden väärälle koneelle, kirjoittaa komennon väärin tai unohtaa jonkun vaiheen asennuksen aikana. Virheen sattuessa sen korjaaminen, tai jopa löytäminen voi olla haastavaa: usein tämä hoituu käymällä läpi palvelimien lokeja ja komentohistoriaa. Mikäli asennukset hoidetaan Ansiblella, adminit voivat käydä läpi tulevia muutoksia ennen kuin ne ajetaan palvelimille, vähentäen monia edellämainituista riskeistä. Lisäksi Ansiblella hoidettavista asennuksista jää selkeä jälki versionhallintaan, jolloin vian etsiminen helpottuu.

Riskien vähentämisen lisäksi Ansible nopeuttaa adminin työtä: hänen ei tarvitse kirjautua usealle eri palvelimelle ja syöttää samat komennot kerta toisensa jälkeen, vaan hän voi yhdellä komennolla ajaa kymmenille, tai jopa sadoille palvelimille asennukset.

# Yhteenveto

Infrastructure as Code tarjoaa monia hyötyjä käsin tehtävään hallintaan:

- riskien vähennys: tulevat muutokset voidaan tarkistaa ennen niiden ajamista kohteeseen
- nopeus: yhdellä komennolla voidaan hoitaa usean palvelimen huolto samaan aikaan
- kirjanpito: playbook versiot voidaan tallentaa versionhallintajärjestelmään, jolloin jää selkeä historia mitä kohteeseen on tehty
- automaatio: playbookit voidaan ajaa automaattisesti, esimerkiksi GitHub Actionseissa
