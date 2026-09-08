# Netwerk Lab — Cisco Packet Tracer

Segmentatie, routing en beveiliging van een klein bedrijfsnetwerk met meerdere afdelingen, opgezet en getest in Cisco Packet Tracer.

## Doel

Aantonen dat ik een realistisch bedrijfsnetwerk kan ontwerpen, configureren en beveiligen: VLAN-segmentatie op meerdere niveaus, inter-VLAN routing, DHCP, port security, ACL's tussen afdelingen, en SSH/AAA-hardening op elk netwerkapparaat — geen alleen op de router, maar ook op alle switches.

## Topologie

```
                                  [Router0]
                            (router-on-a-stick, 4 sub-int)
                                      |
                              [Switch0 - Core/Trunk]
                            /        |         \
                [Switch1]      [Switch2]      [Switch3]
              (Sales-access) (IT-access)    (HR-access)
                 |    |          |    |          |    |
               PC1  PC2       PC3  Server0    PC4   PC5
                              (+Server1)
```

| VLAN | Naam | Netwerk | Gateway |
|---|---|---|---|
| 10 | Sales | 192.168.10.0/24 | 192.168.10.1 |
| 20 | IT | 192.168.20.0/24 | 192.168.20.1 |
| 30 | HR | 192.168.30.0/24 | 192.168.30.1 |
| 99 | Management | 192.168.99.0/24 | 192.168.99.1 |

Zie `screenshots/01_topologie_apparaten_geplaatst.png` voor de volledige topologie.

## Wat is gedaan

1. **Topologie-ontwerp en bekabeling** — 1 router, 1 core-switch, 3 access-switches (één per afdeling), 4 PC's, 2 servers.
2. **VLAN-segmentatie en trunking** — 4 VLAN's (Sales/IT/HR + een aparte management-VLAN, gescheiden van gebruikersverkeer) met trunking tussen core- en access-switches.
3. **Inter-VLAN routing** — router-on-a-stick met 4 sub-interfaces op Router0.
4. **DHCP** — een pool per afdelings-VLAN, met uitgesloten adressen voor vaste toestellen.
5. **Port security** — max. 2 MAC-adressen per gebruikerspoort, sticky learning, violation mode "restrict".
6. **Drie ACL's** met elk een specifiek beveiligingsdoel:
   - Sales mag niet SSH'en naar de IT-server, overig verkeer (zoals ping) blijft toegelaten.
   - HR wordt volledig gescheiden van Sales (alle verkeer geblokkeerd).
   - Enkel IT mag de management-VLAN bereiken; Sales en HR worden geweigerd.
7. **SSH + lokale AAA** op de router én alle vier switches — geen onbeveiligde Telnet, wachtwoorden versleuteld, banner bij inloggen, beheer via de aparte management-VLAN.
8. **Meerdere echte troubleshooting-scenario's** doorlopen en gedocumenteerd (zie onder).

## Skills aangetoond

- VLAN-ontwerp en -configuratie, inclusief een aparte management-VLAN als best practice
- Inter-VLAN routing (router-on-a-stick)
- DHCP-beheer per netwerksegment
- Port security
- Netwerkbeveiliging: ACL's tussen afdelingen, SSH/AAA-hardening op alle apparatuur (niet enkel de router)
- Netwerk-troubleshooting: systematisch diagnosticeren met `show`-commando's, ipconfig en ping i.p.v. gokken

## Troubleshooting-scenario's

Dit lab leverde onderweg meerdere echte troubleshooting-momenten op — het "probleem → diagnose → oplossing"-verhaal dat in een sollicitatiegesprek vaak sterker overkomt dan een diploma:

| # | Symptoom | Diagnose | Oorzaak | Oplossing |
|---|---|---|---|---|
| 1 | `show vlan brief` leek poorten onder het verkeerde VLAN te tonen | `show interfaces <poort> switchport` | Verwarrende layout van `show vlan brief`, geen echte fout | Verifiëren met een specifieker commando bevestigde de juiste VLAN-toewijzing |
| 2 | Ping tussen VLAN's gaf 100% verlies | `ipconfig` op de PC | IPv4-adres stond nog op 0.0.0.0 — statisch IP nooit toegepast op de PC zelf | IP-adres correct ingesteld via Desktop > IP Configuration |
| 3 | PC's kregen een APIPA-adres (169.254.x.x) i.p.v. een DHCP-adres | `ipconfig` | Omschakeling naar DHCP werd niet meteen actief | `ipconfig /release` + `/renew` om een nieuwe DHCP-aanvraag te forceren |
| 4 | "Invalid input" bij het plakken van meerdere CLI-commando's | Commando's manueel herhaald | Eerste teken van een regel viel weg bij snel plakken (Packet Tracer-eigenaardigheid) | Commando's opnieuw getypt, functioneel geen probleem |
| 5 | Ping naar het management-VLAN mislukte voor iedereen, ook voor IT dat toegang zou moeten hebben | Controle van de switch-configuratie | Het management-IP-adres (`interface vlan 99`) was nooit geconfigureerd op Switch0 | SVI met IP-adres en default-gateway toegevoegd op de switches |
| 6 | Na de vorige fix kon ook Sales het management-VLAN bereiken, terwijl dat net geweigerd moest worden | Analyse van de ACL-richting | ACL stond toegepast met richting "in" op de management-sub-interface — filtert verkeer *vanuit* die VLAN, niet *naartoe* | ACL-richting gewijzigd naar "out", waardoor elk pakket gecontroleerd wordt vlak vóór het de management-VLAN binnengaat |

Zie het volledige voortgangsverslag (`documentatie/Lab3_Voortgangsverslag.docx`) voor de uitgebreide beschrijving met screenshots per stap.

## Screenshots & documentatie

- `screenshots/` — alle configuratie- en testresultaten, chronologisch genummerd
- `documentatie/Lab3_Voortgangsverslag.docx` — het volledige, stap-voor-stap voortgangsverslag met uitleg en screenshots bij elke fase
- `configs/` — ruwe `show running-config`-exports per apparaat (toe te voegen)
- `topology.pkt` — het Packet Tracer-bestand (toe te voegen)
