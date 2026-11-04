# Lab 3 — Cisco Packet Tracer Enterprise Network Security Lab

**VLAN-Segmentatie, Inter-VLAN Routing (802.1Q), DHCP Pools, Port Security, ACL's & SSH/AAA Hardening**  
*Tyrese Talia | 2026*

---

## Inhoudsopgave
1. [Waarvoor dit lab is & Doel](#1-waarvoor-dit-lab-is--doel)
2. [Waarom ik gekozen heb om dit lab te bouwen](#2-waarom-ik-gekozen-heb-om-dit-lab-te-bouwen)
3. [Wat ik heb geleerd & Technische Vaardigheden](#3-wat-ik-heb-geleerd--technische-vaardigheden)
4. [Stapsgewijze Uitvoering (Fasering)](#4-stapsgewijze-uitvoering-fasering)
5. [Access Control Lists (ACL) Matrix & Verkeersfiltering](#5-access-control-lists-acl-matrix--verkeersfiltering)
6. [Troubleshooting & Incident Response](#6-troubleshooting--incident-response)

---

## 1. Waarvoor dit lab is & Doel
Dit lab demonstreert het ontwerpen, configureren, beveiligen en systematisch troubleshooten van een multi-department enterprise netwerk in Cisco Packet Tracer:
* **VLAN Segmentatie:** Fysieke infrastructuur opsplitsen in 4 gescheiden logische netwerken (Sales, IT, HR en een geïsoleerd Management VLAN 99).
* **Inter-VLAN Routing:** Router-on-a-stick met 802.1Q trunking op sub-interfaces van Router0.
* **Geautomatiseerde IP-Uitgifte:** Dedicated DHCP pools per VLAN met excluded address ranges voor netwerkinfrastructuren en servers.
* **Layer 2 Beveiliging:** Port Security met MAC-limiting (maximum 2), sticky learning en violation mode 
estrict.
* **Afdelingsisolatie (ACL's):** Drie afzonderlijke Access Control Lists om vertrouwelijk HR-verkeer en management-toegang te beveiligen.
* **Volledige Apparaathardening:** Telnet uitschakelen, SSH v2 activeren en AAA-authenticatie configureren op alle routers én switches.

---

## 2. Waarom ik gekozen heb om dit lab te bouwen
Als aankomend IT Netwerk & Cloud Engineer wilde ik bewijzen dat ik netwerkinfrastructuren vanaf de basis (Layer 2 en Layer 3) begrijp en kan beveiligen:
1. **Van theorie naar tastbare netwerktopologie:** CCNA-concepten toepassen in een gesimuleerde bedrijfsomgeving met meerdere switches en routers.
2. **Security op elk apparaat implementeren:** Niet alleen de perimeter/router beveiligen, maar Defense-in-Depth toepassen tot op de switchpoorten van eindgebruikers.
3. **Echte netwerkfouten oplossen:** Zelfstandig ontdekken hoe subtiele fouten (zoals ACL traffic direction 'in' vs 'out') netwerkverkeer beïnvloeden.

---

## 3. Wat ik heb geleerd & Technische Vaardigheden
* **Cisco IOS Configuratie:** CLI commando's voor interfaces, trunks, sub-interfaces en SVI's.
* **Layer 2 Switching:** 802.1Q trunking, native VLAN's en Switchport Port Security.
* **Layer 3 Routing & Services:** Router-on-a-Stick sub-interface routing, DHCP pools en default gateways.
* **Traffic Filtering (ACL's):** Extended IP access-lists schrijven en correct binden (in vs out).
* **Device Hardening:** Lokale AAA, crypto RSA key generation en beveiligde SSH vty lines.

---

## 4. Stapsgewijze Uitvoering (Fasering)

Dit project is het derde lab van mijn IT-portfolio, naast het Windows/Linux Home Lab en het Azure Cloud Lab. Het doel is aan te tonen dat ik een klein bedrijfsnetwerk kan ontwerpen, configureren en beveiligen in Cisco Packet Tracer — een vaardigheid die rechtstreeks aansluit bij een junior IT Support / Sysadmin-functie.

Het netwerk simuleert één bedrijfslocatie met drie afdelingen die logisch van elkaar gescheiden zijn via VLAN's, plus een aparte management-VLAN voor het beheer van de netwerkapparatuur zelf:

* VLAN 10 — Sales (192.168.10.0/24)
* VLAN 20 — IT (192.168.20.0/24), met een DHCP/DNS-server en een tweede interne server
* VLAN 30 — HR (192.168.30.0/24)
* VLAN 99 — Management (192.168.99.0/24), enkel voor de switches/router zelf
De opbouw bestaat uit één router (router-on-a-stick voor inter-VLAN routing), één core-switch die trunkt naar drie access-switches (één per afdeling), en meerdere eindapparaten (PC's en servers) verdeeld over die afdelingen.

Wat ik ga doen, in volgorde:

* De topologie opbouwen in Packet Tracer: apparatuur plaatsen en bekabelen
* VLAN's aanmaken en trunking configureren tussen de switches
* Inter-VLAN routing instellen op de router (router-on-a-stick)
* DHCP configureren per afdeling
* Port security instellen op de gebruikerspoorten
* ACL's opzetten om verkeer tussen afdelingen gericht te beperken
* SSH en lokale AAA instellen op de router én alle switches
* Alles verifiëren en minstens één troubleshooting-scenario documenteren
Dit document wordt automatisch bijgewerkt na elke geslaagde stap, met screenshots als bewijs. Het dient later als basis voor het definitieve README.md en portfolio-rapport.


# 2. Topologie-ontwerp en bekabeling

STATUS: VOLTOOID

Alle apparaten neergezet in Packet Tracer volgens het ontwerp uit de lab-guide — 1 router (Router0), 1 core-switch (Switch0), 3 access-switches (Switch1 = Sales, Switch2 = IT, Switch3 = HR), en de eindapparaten per afdeling (PC1-PC2 in Sales, PC3 + Server0 + Server1 in IT, PC4-PC5 in HR). De afdelingen zijn visueel gegroepeerd en gekleurd (blauw = HR, geel = Sales, rood = IT) voor overzicht.

Daarna volledig bekabeld: koperen straight-through-kabels tussen elk eindapparaat en zijn access-switch, en (automatisch gekozen) cross-over-kabels tussen Switch0 en de drie access-switches, en tussen Switch0 en Router0. Alle poorten tonen een groen link-lampje, wat bevestigt dat elke fysieke verbinding actief is. Het rode pijltje op de link naar Router0 is verwacht op dit punt: de router-interface staat nog administratief 'shutdown' tot de configuratiestap.

Afbeelding 1 — Topologie volledig geplaatst en bekabeld.


# 3. VLAN's aangemaakt op de core-switch (Switch0)

STATUS: VOLTOOID

Op Switch0 (de core-switch) de 4 VLAN's aangemaakt en benoemd via de CLI:

* VLAN 10 — Sales
* VLAN 20 — IT
* VLAN 30 — HR
* VLAN 99 — Management
Commando's gebruikt (in global configuration mode):

vlan 10 / name Sales / exit, herhaald voor VLAN 20 (IT), 30 (HR) en 99 (Management).

Afbeelding 2 — CLI van Switch0: de 4 VLAN's succesvol aangemaakt en benoemd.


# 4. Trunking geconfigureerd op Switch0

STATUS: VOLTOOID

De vier uplink-poorten van Switch0 (Fa0/1 t/m Fa0/4, richting Router0, Switch1, Switch2 en Switch3) ingesteld als trunk-poort, met alle 4 VLAN's toegelaten op de trunk:

interface range fa0/1-4 / switchport mode trunk / switchport trunk allowed vlan 10,20,30,99

De line-protocol down/up-meldingen voor Fa0/2, Fa0/3 en Fa0/4 zijn normaal: die poorten flappen kort doordat de switch opnieuw onderhandelt na de mode-wijziging naar trunk. Fa0/1 (richting Router0) flapt niet, en verschijnt in 'show vlan brief' nog kort onder VLAN 1 — dat komt doordat de interface van Router0 aan de overkant nog administratief 'shutdown' staat, waardoor er nog geen actieve link is om als trunk te bevestigen. Dit lost zichzelf op zodra de router-interface wordt geactiveerd in een volgende stap.

Afbeelding 3 — CLI van Switch0: trunk-poorten geconfigureerd en bevestigd met 'show vlan brief'.


# 5. Switch1 (Sales) geconfigureerd + verificatie

STATUS: VOLTOOID

Op Switch1 de VLAN's 10 (Sales) en 99 (Management) aangemaakt, poort Fa0/1 ingesteld als trunk (richting Switch0, met VLAN 10 en 99 toegelaten), en Fa0/2-3 als access-poorten in VLAN 10 voor PC1 en PC2.

interface fa0/1 / switchport mode trunk / switchport trunk allowed vlan 10,99 — interface range fa0/2-3 / switchport mode access / switchport access vlan 10

Afbeelding 4 — CLI van Switch1: VLAN's, trunk- en access-poorten geconfigureerd.


## Troubleshooting-moment: verificatie van de VLAN-toewijzing

De 'show vlan brief'-output leek in eerste instantie te tonen dat Fa0/2 en Fa0/3 bij VLAN 99 (Management) hoorden in plaats van VLAN 10 (Sales) — een onduidelijke layout in die tabel, geen foutmelding. In plaats van dit zomaar aan te nemen, is dit expliciet geverifieerd met een specifieker commando:

show interfaces fa0/2 switchport

Resultaat: 'Access Mode VLAN: 10 (Sales)' — de poort staat wel degelijk correct in VLAN 10. De verwarring kwam dus door de layout van 'show vlan brief', niet door een echte configuratiefout. Dit toont het belang van verifiëren met meerdere commando's voor je een conclusie trekt.

Afbeelding 5 — Verificatie met 'show interfaces fa0/2 switchport': VLAN-toewijzing bevestigd correct.


# 6. Switch2 (IT) geconfigureerd + verificatie

STATUS: VOLTOOID

Op Switch2 de VLAN's 20 (IT) en 99 (Management) aangemaakt, Fa0/1 ingesteld als trunk (richting Switch0, VLAN 20 en 99 toegelaten), en Fa0/2-4 als access-poorten in VLAN 20 voor PC3, Server0 en Server1 (drie eindapparaten i.p.v. twee, vandaar het bredere poortbereik).

Meteen geverifieerd met 'show interfaces fa0/2 switchport': Access Mode VLAN bevestigd als 20 (IT) — in één keer correct, zonder de layout-verwarring van de vorige stap.

Afbeelding 6 — CLI van Switch2: VLAN's, trunk- en access-poorten geconfigureerd en meteen geverifieerd (Access Mode VLAN: 20 (IT)).


# 7. Switch3 (HR) geconfigureerd + verificatie

STATUS: VOLTOOID

Op Switch3 de VLAN's 30 (HR) en 99 (Management) aangemaakt, Fa0/1 ingesteld als trunk (richting Switch0, VLAN 30 en 99 toegelaten), en Fa0/2-3 als access-poorten in VLAN 30 voor PC4 en PC5. Meteen geverifieerd met 'show interfaces fa0/2 switchport': Access Mode VLAN bevestigd als 30 (HR).

Alle vier de switches (Switch0 t/m Switch3) zijn hiermee volledig geconfigureerd: VLAN's aangemaakt, trunking tussen core en access-switches ingesteld, en elk eindapparaat in de juiste VLAN via zijn access-poort.

Afbeelding 7 — CLI van Switch3: VLAN's, trunk- en access-poorten geconfigureerd en geverifieerd (Access Mode VLAN: 30 (HR)).


# 8. Router0 — Inter-VLAN routing (router-on-a-stick)

STATUS: VOLTOOID

Op Router0 vier sub-interfaces aangemaakt op de fysieke interface GigabitEthernet0/0, elk met 802.1Q-encapsulatie voor zijn VLAN en het gateway-IP van dat VLAN:

* gig0/0.10 — dot1Q 10 — 192.168.10.1/24 (Sales)
* gig0/0.20 — dot1Q 20 — 192.168.20.1/24 (IT)
* gig0/0.30 — dot1Q 30 — 192.168.30.1/24 (HR)
* gig0/0.99 — dot1Q 99 — 192.168.99.1/24 (Management)
Daarna de fysieke interface geactiveerd met 'no shutdown'. Dit loste meteen ook het eerder gesignaleerde 'shutdown'-statusje op de trunk-link naar Switch0 op — die interface staat nu ook actief.

Afbeelding 8 — CLI van Router0: de 4 sub-interfaces geconfigureerd en GigabitEthernet0/0 geactiveerd.

Geverifieerd met 'show ip interface brief': alle 4 sub-interfaces staan op status up/up met het correcte IP-adres. GigabitEthernet0/1 en Vlan1 staan terecht 'administratively down' — die interfaces worden niet gebruikt in dit ontwerp.

Afbeelding 9 — 'show ip interface brief' op Router0: alle sub-interfaces up/up met de juiste IP-adressen.


# 9. Verificatietest: ping tussen VLAN's

STATUS: VOLTOOID

Om te bewijzen dat inter-VLAN routing effectief werkt, PC1 (Sales, VLAN 10) en PC4 (HR, VLAN 30) tijdelijk een statisch IP-adres gegeven (192.168.10.10 resp. 192.168.30.10, telkens met de juiste gateway) en een ping uitgevoerd tussen beide afdelingen.


## Troubleshooting-moment: eerste ping-poging mislukte volledig

De eerste test (ping van PC1 naar PC4) gaf 100% pakketverlies. Diagnose met 'ipconfig' op de PC toonde de oorzaak meteen: het IPv4-adres stond nog op 0.0.0.0 — de statische IP-configuratie was nog niet effectief ingesteld op de PC zelf (enkel besproken, niet toegepast via Desktop > IP Configuration). Er was op dat moment ook nog geen DHCP geconfigureerd, dus de PC had helemaal geen bruikbaar adres.

Afbeelding 10 — Eerste ping-poging: 100% verlies.

Afbeelding 11 — Diagnose met ipconfig: IPv4-adres stond nog op 0.0.0.0.

Oplossing: op PC1 en PC4 via Desktop > IP Configuration een statisch IP-adres, subnetmasker en gateway ingesteld voor hun respectievelijke VLAN.


## Resultaat na de fix

Ping van PC1 naar zijn eigen gateway (192.168.10.1): 0% verlies. Ping van PC1 naar PC4 (192.168.30.10, andere VLAN): het eerste pakket 'timed out' (normaal — ARP-opzoeking), de volgende 3 pakketten geslaagd met TTL=127 in plaats van 128. Die verlaging van TTL met exact 1 bevestigt dat het pakket via Router0 geroute is — inter-VLAN routing werkt dus correct.

Afbeelding 12 — Geslaagde ping naar eigen gateway én naar PC4 in een andere VLAN (TTL=127 bevestigt routing via Router0).


# 10. DHCP-configuratie op Router0

STATUS: VOLTOOID

DHCP geconfigureerd op de router (optie A uit de lab-guide), met één pool per afdelings-VLAN. De eerste 10 adressen van elk subnet zijn uitgesloten (gereserveerd voor eventuele vaste toestellen zoals servers of switch-management-IP's):

* VLAN10-SALES — netwerk 192.168.10.0/24, gateway 192.168.10.1
* VLAN20-IT — netwerk 192.168.20.0/24, gateway 192.168.20.1
* VLAN30-HR — netwerk 192.168.30.0/24, gateway 192.168.30.1
Elke pool wijst naar 192.168.20.10 als DNS-server (de latere Server0 in de IT-VLAN).

Afbeelding 13 — CLI van Router0: de 3 DHCP-pools (Sales/IT/HR) geconfigureerd met excluded-addresses, network, default-router en dns-server.


# 11. PC's op DHCP + troubleshooting

STATUS: VOLTOOID


## Troubleshooting-moment: PC's kregen een APIPA-adres

Na het omzetten van PC1 en PC4 naar DHCP (Desktop > IP Configuration > DHCP) toonde 'ipconfig' een 'Autoconfiguration IPv4 Address' in de 169.254.x.x-reeks. Dat is een APIPA-adres: het adres dat Windows/Packet Tracer zelf toekent wanneer een DHCP-aanvraag geen antwoord krijgt binnen de time-out. De DHCP-configuratie op de router was op zich correct, maar de omschakeling naar DHCP werd niet meteen actief zonder een expliciete nieuwe aanvraag.

Afbeelding 14 — Symptoom: PC's kregen een APIPA-adres (169.254.x.x) i.p.v. een adres van de DHCP-pool.

Oplossing: op de PC's een nieuwe DHCP-aanvraag geforceerd met:

ipconfig /release

ipconfig /renew

Resultaat: PC1 kreeg 192.168.10.11, PC4 kreeg 192.168.30.11, beide met de correcte gateway en DNS-server (192.168.20.10) uit hun eigen VLAN-pool.

Afbeelding 15 — Na 'ipconfig /release' + '/renew': correct DHCP-adres, gateway en DNS ontvangen voor beide PC's.


# 12. Port security op de access-poorten

STATUS: VOLTOOID

Op elke gebruikerspoort van Switch1, Switch2 en Switch3 port security ingesteld: maximum 2 MAC-adressen, sticky learning, en violation mode 'restrict' (blokkeert onbekende extra apparaten zonder de hele poort uit te schakelen).

switchport port-security / switchport port-security maximum 2 / switchport port-security mac-address sticky / switchport port-security violation restrict

Geverifieerd op Switch1 (Fa0/2) met 'show port-security interface': Port Security enabled, Port Status 'Secure-up', Violation Mode 'Restrict', Maximum MAC Addresses 2.

Afbeelding 16 — Port security geconfigureerd en geverifieerd op Fa0/2 (Switch1/PC1).


## Troubleshooting-moment: 'Invalid input' bij het plakken van commando's op Switch3

Bij het plakken (paste) van meerdere commandoregels tegelijk in de CLI van Switch3 gaf IOS een paar keer 'Invalid input detected' — telkens omdat het eerste teken van een regel wegviel bij het plakken (een bekende eigenaardigheid van snel plakken in de Packet Tracer-CLI, geen echte configuratiefout). Elk commando werd meteen daarna manueel herhaald en correct aanvaard. Nadien bevestigd dat Switch2 en Switch3 volledig correct geconfigureerd staan.

Alle gebruikerspoorten op de drie access-switches hebben nu port security actief.


# 13. ACL 1 — Sales geblokkeerd voor SSH naar de IT-server

STATUS: VOLTOOID EN GETEST

Op Router0 een extended ACL aangemaakt die SSH-verkeer (TCP-poort 22) vanuit Sales (192.168.10.0/24) naar de IT-server (192.168.20.10) blokkeert, met een expliciete 'permit ip any any' erna zodat al het overige verkeer wel toegelaten blijft. Toegepast op de inkomende richting van gig0/0.10 (het Sales-VLAN).

ip access-list extended BLOCK-SALES-TO-IT-SSH — deny tcp 192.168.10.0 0.0.0.255 host 192.168.20.10 eq 22 — permit ip any any — interface gig0/0.10 — ip access-group BLOCK-SALES-TO-IT-SSH in

Bevestigd met 'show access-lists': de ACL staat met beide regels (deny + permit) op de juiste interface.

Afbeelding 17 — CLI van Router0: ACL 'BLOCK-SALES-TO-IT-SSH' geconfigureerd en toegepast op gig0/0.10.


## Test 1: SSH vanaf Sales naar Server0 — moet geweigerd worden

Vanaf PC1 (Sales): 'ssh -l admin 192.168.20.10' geeft 'Connection timed out; remote host not responding' — de ACL blokkeert het SSH-verkeer (poort 22) zoals bedoeld, nog vóór het Server0 kan bereiken.

Afbeelding 18 — SSH-poging van PC1 naar Server0 geweigerd door de ACL.


## Test 2: ping vanaf Sales naar Server0 — moet wél lukken

Vanaf PC1: 'ping 192.168.20.10' — eerste pakket 'timed out' (normale ARP-vertraging), daarna 3/3 geslaagd met TTL=127. Dit bewijst dat de ACL specifiek genoeg is: enkel SSH wordt geblokkeerd, ander verkeer (zoals ping) blijft gewoon toegelaten.

Afbeelding 19 — Ping van PC1 naar Server0 slaagt, ondanks de actieve ACL.


# 14. ACL 2 — HR volledig geweerd van Sales

STATUS: VOLTOOID EN GETEST

Op Router0 een tweede extended ACL aangemaakt die al het verkeer vanuit HR (192.168.30.0/24) naar Sales (192.168.10.0/24) blokkeert — een volledige scheiding tussen die twee afdelingen, in tegenstelling tot ACL 1 die enkel één specifieke poort blokkeerde. Toegepast op de inkomende richting van gig0/0.30 (het HR-VLAN).

ip access-list extended BLOCK-HR-TO-SALES — deny ip 192.168.30.0 0.0.0.255 192.168.10.0 0.0.0.255 — permit ip any any — interface gig0/0.30 — ip access-group BLOCK-HR-TO-SALES in

Bevestigd met 'show access-lists': beide ACL's staan actief. ACL 1 toont zelfs al match-tellers (24 op de deny-regel, 4 op de permit-regel) van de eerdere testen — nog een extra bewijs dat de ACL daadwerkelijk verkeer verwerkt.

Afbeelding 20 — CLI van Router0: ACL 'BLOCK-HR-TO-SALES' geconfigureerd, met 'show access-lists' die beide actieve ACL's en hun matches toont.


## Test: ping vanaf HR naar Sales — moet volledig mislukken

Vanaf PC4 (HR) naar PC1's DHCP-adres (192.168.10.11): 'Reply from 192.168.30.1: Destination host unreachable', 100% verlies (4/4). De router weigert het pakket zelf al door te sturen — dit bevestigt dat de ACL, in tegenstelling tot ACL 1, hier alle verkeer blokkeert (niet enkel een specifieke poort).

Afbeelding 21 — Ping van PC4 (HR) naar PC1 (Sales) volledig geweigerd door de router.


# 15. ACL 3 — enkel IT mag de management-VLAN bereiken

STATUS: VOLTOOID EN GETEST (na twee troubleshooting-rondes)

Op Router0 een derde ACL aangemaakt: IT (VLAN 20) mag het management-VLAN (192.168.99.0/24) bereiken, Sales en HR niet.

ip access-list extended RESTRICT-MGMT-ACCESS — permit ip 192.168.20.0 0.0.0.255 192.168.99.0 0.0.0.255 — deny ip 192.168.10.0 0.0.0.255 192.168.99.0 0.0.0.255 — deny ip 192.168.30.0 0.0.0.255 192.168.99.0 0.0.0.255 — permit ip any any

Afbeelding 22 — CLI van Router0: ACL 'RESTRICT-MGMT-ACCESS' geconfigureerd, oorspronkelijk toegepast met 'in' op gig0/0.99.


## Troubleshooting-ronde 1: management-IP ontbrak op Switch0

Eerste testronde: ping vanaf zowel PC1 (Sales) als PC3 (IT) naar 192.168.99.2 mislukte volledig (100% verlies) — ook vanuit IT, dat net toegang zou moeten hebben. Diagnose: Switch0 had nog geen IP-adres op zijn VLAN 99-interface (interface vlan 99), dus 192.168.99.2 bestond simpelweg nog niet op het netwerk. Dit was een stap uit het oorspronkelijke ontwerp die aanvankelijk over het hoofd werd gezien bij de switch-configuratie.

Oplossing: op Switch0 (en voor de volledigheid ook Switch1/2/3) een 'interface vlan 99' aangemaakt met het geplande management-IP en 'ip default-gateway 192.168.99.1' ingesteld.

interface vlan 99 — ip address 192.168.99.2 255.255.255.0 — no shutdown — ip default-gateway 192.168.99.1


## Troubleshooting-ronde 2: ACL in de verkeerde richting toegepast

Na het toevoegen van het management-IP kon IT (PC3) 192.168.99.2 nu bereiken (correct), maar onverwacht kon Sales (PC1) dat ook — 4/4 pings geslaagd, terwijl dat net geweigerd moest worden. Diagnose: de ACL stond toegepast als 'ip access-group RESTRICT-MGMT-ACCESS in' op gig0/0.99. De richting 'in' filtert verkeer dat de router binnenkomt vanuit VLAN 99 zelf (bronadres 192.168.99.x) — niet verkeer dat naar VLAN 99 onderweg is vanuit andere VLAN's. Omdat geen enkele deny-regel matchte, viel alles terug op de afsluitende 'permit ip any any', en liet de ACL in de praktijk alles door.

Oplossing: de ACL van richting veranderd naar 'out' op dezelfde interface, zodat elk pakket gecontroleerd wordt net vóór het VLAN 99 binnengaat, ongeacht van welk VLAN het afkomstig is:

interface gig0/0.99 — no ip access-group RESTRICT-MGMT-ACCESS in — ip access-group RESTRICT-MGMT-ACCESS out


## Resultaat na beide fixes

PC1 (Sales) → 192.168.99.2: 'Destination host unreachable', 100% verlies — correct geweigerd. PC3 (IT) → 192.168.99.2: 4/4 geslaagd met TTL=254 — correct toegelaten. ACL 3 werkt nu zoals bedoeld.

Afbeelding 24 — Na de fix: Sales (PC1) correct geweigerd toegang tot het management-VLAN.

Afbeelding 25 — Na de fix: IT (PC3) behoudt correct toegang tot het management-VLAN.


# 16. SSH + lokale AAA op Router0

STATUS: VOLTOOID EN GETEST

Router0 beveiligd met een sterk enable-wachtwoord (versleuteld), een lokale gebruiker voor login, en SSH in plaats van onbeveiligde Telnet voor extern beheer:

* hostname router0 (nodig vóór het genereren van de RSA-sleutels — Cisco IOS weigert dit met de standaard hostname 'Router')
* enable secret + service password-encryption
* ip domain-name lab.local + crypto key generate rsa (1024 bit)
* username admin privilege 15 secret ... (lokale AAA-gebruiker)
* line console 0: wachtwoord + login
* line vty 0 4: transport input ssh, login local, exec-timeout 5 0
* banner motd met waarschuwing tegen ongeautoriseerde toegang
Bevestigd: 'SSH-5-ENABLED: SSH 1.99 has been enabled' verscheen in de log — SSH staat actief.

Afbeelding 26 — CLI van Router0: hostname, enable secret, RSA-sleutels, lokale gebruiker, console- en vty-beveiliging en banner geconfigureerd.


## Test: SSH-verbinding vanaf PC3 naar Router0

Vanaf PC3 (IT): 'ssh -l admin 192.168.20.1'. Eerste poging gaf 'Login invalid' (tikfout in het wachtwoord), tweede poging slaagde: de banner ('Enkel geautoriseerde toegang toegelaten') verscheen en de sessie kwam uit op de prompt 'router0#' — een succesvolle, beveiligde SSH-verbinding i.p.v. onbeveiligde Telnet.

Afbeelding 27 — Geslaagde SSH-verbinding vanaf PC3 naar Router0, met banner en privileged-EXEC-prompt.

Volgende stap: dezelfde beveiliging (hostname, enable secret, RSA-sleutels, lokale gebruiker, SSH, banner) toepassen op de vier switches.


# 17. SSH + lokale AAA op de switches

STATUS: VOLTOOID EN GETEST

Dezelfde beveiligingsconfiguratie als op Router0 toegepast op alle vier switches (hostnames SW0 t/m SW3): enable secret, service password-encryption, domeinnaam + RSA-sleutels (1024 bit), lokale gebruiker 'admin', console- en vty-beveiliging met SSH i.p.v. Telnet, en de waarschuwingsbanner. 'SSH-5-ENABLED: SSH 1.99 has been enabled' bevestigde telkens dat SSH actief staat.

Afbeelding 28 — CLI van Switch0 (hostname SW0): SSH + lokale AAA volledig geconfigureerd.


## Test: SSH naar een switch via het management-VLAN

Vanaf een PC met toegang tot 192.168.99.0/24: 'ssh -l admin 192.168.99.3' (Switch1's management-IP). Login gelukt: de banner verscheen en de sessie kwam uit op de prompt 'SW1#'. Dit bevestigt dat switch-beheer volledig via SSH over de aparte management-VLAN verloopt, in plaats van via een onbeveiligd protocol of een gebruikers-VLAN.

Afbeelding 29 — Geslaagde SSH-verbinding naar Switch1 (SW1) via het management-VLAN (192.168.99.3).
