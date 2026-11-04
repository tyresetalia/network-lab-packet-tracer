# Cisco Packet Tracer Enterprise Network Security & Segmentation Lab

![Cisco](https://img.shields.io/badge/Cisco-Packet%20Tracer-blue?logo=cisco)
![Networking](https://img.shields.io/badge/Networking-Inter--VLAN%20Routing-green)
![Security](https://img.shields.io/badge/Security-Port%20Security%20%26%20ACLs-red)
![Hardening](https://img.shields.io/badge/Hardening-SSH%20%26%20AAA-orange)

Ontwerp, implementatie, beveiliging en troubleshooting van een multi-department bedrijfsnetwerk in Cisco Packet Tracer. Omvat VLAN-segmentatie, router-on-a-stick inter-VLAN routing, dynamische DHCP pool provisioning, Port Security (MAC-limiting), enterprise Access Control Lists (ACL's) en volledige SSH/AAA apparaathardening op alle routers en switches.

---

## 📌 Inhoudsopgave
1. [Projectoverzicht & Doelstelling](#-projectoverzicht--doelstelling)
2. [Netwerktopologie & Ontwerp](#-netwerktopologie--ontwerp)
3. [VLAN- & IP-Adresseringstabel](#-vlan---ip-adresseringstabel)
4. [Beveiligingsmodel & Hardening](#-beveiligingsmodel--hardening)
5. [Stap-voor-stap Implementatie](#-stap-voor-stap-implementatie)
6. [Access Control Lists (ACL) Matrix](#-access-control-lists-acl-matrix)
7. [Systematische Troubleshooting & Incident Response](#-systematische-troubleshooting--incident-response)
8. [Documentatie & Bewijslast](#-documentatie--bewijslast)

---

## 🎯 Projectoverzicht & Doelstelling

Aantonen dat ik een realistisch bedrijfsnetwerk zelfstandig kan ontwerpen, configureren, beveiligen en systematisch troubleshooten:
* **VLAN Segmentatie:** Fysieke infrastructuur logisch opsplitsen per afdeling (Sales, IT, HR) plus een geïsoleerd Out-of-Band Management VLAN.
* **Inter-VLAN Routing:** Gecontroleerde routering via sub-interfaces (Router-on-a-Stick) met 802.1Q trunking.
* **Geautomatiseerde IP-Uitgifte:** Dedicated DHCP pools per VLAN met excluded range voor statische servers en netwerkinterfaces.
* **Layer 2 Beveiliging:** Inzet van Port Security met MAC-beperking en sticky learning om onbevoegde apparaten te weren.
* **Afdelingsisolatie (ACL's):** Granulaire verkeersfiltering ter bescherming van management- en serveromgevingen.
* **Apparaathardening:** Volledige uitschakeling van plaintext Telnet ten gunste van SSH v2 en AAA authenticatie op alle switches en router.

---

## 🏗 Netwerktopologie & Ontwerp

```
                                      [ Router0 ]
                              (Router-on-a-Stick / 802.1Q)
                                           │
                                           │ Trunk (Gi0/0 - VLANs 10,20,30,99)
                                           ▼
                                 [ Switch0 - Core Trunk ]
                                           │
                  ┌────────────────────────┼────────────────────────┐
                  │ FastEthernet Trunk     │ FastEthernet Trunk     │ FastEthernet Trunk
                  ▼                        ▼                        ▼
            [ Switch1 ]              [ Switch2 ]              [ Switch3 ]
          (Sales Access)             (IT Access)              (HR Access)
            │         │                │         │              │         │
            ▼         ▼                ▼         ▼              ▼         ▼
          [PC1]     [PC2]            [PC3]   [Server0]        [PC4]     [PC5]
                                             [Server1]
```

---

## ⚙ VLAN- & IP-Adresseringstabel

| VLAN | Afdeling / Functie | Subnet | Gateway (Router0) | Beschrijving |
| :---: | :--- | :--- | :--- | :--- |
| **10** | Sales | `192.168.10.0/24` | `192.168.10.1` | Werkstations verkoopafdeling |
| **20** | IT & Infra | `192.168.20.0/24` | `192.168.20.1` | IT-beheer en centrale applicatieservers |
| **30** | Human Resources | `192.168.30.0/24` | `192.168.30.1` | Vertrouwelijke HR werkplekken |
| **99** | Beheer / Management | `192.168.99.0/24` | `192.168.99.1` | Switch Virtual Interfaces (SVI) voor remote SSH |

---

## 🛡 Beveiligingsmodel & Hardening

1. **Layer 2 Port Security:**
   * Maximaal 2 MAC-adressen per poort (`switchport port-security maximum 2`).
   * Dynamische opslag via `switchport port-security mac-address sticky`.
   * Violation mode `restrict`: Blokkeert kwaadwillend verkeer en genereert SNMP/Syslog alerts zonder de poort plat te leggen.
2. **Device Management Hardening (SSH & AAA):**
   * Lokale AAA ingeschakeld met sterke gehashte wachtwoorden (`algorithm-type scrypt` / `secret`).
   * RSA cryptosleutels (1024-bit) gegenereerd en Telnet transport uitgeschakeld (`transport input ssh`).
   * SVI's uitsluitend geconfigureerd binnen het geïsoleerde Management VLAN 99.

---

## 🚦 Access Control Lists (ACL) Matrix

| ACL ID | Type | Richting | Doel & Werking |
| :---: | :---: | :---: | :--- |
| **ACL 1** | Extended | Inbound | Sales mag **niet** SSH'en (TCP 22) naar IT Server (`192.168.20.2`), maar ICMP ping en webverkeer blijft toegestaan. |
| **ACL 2** | Extended | Inbound | HR en Sales worden 100% geïsoleerd van elkaar om data-lekken van persoonsgegevens te voorkomen. |
| **ACL 3** | Extended | Outbound | **Strikte Management Beveiliging:** Alleen verkeer vanuit het IT-segment (`192.168.20.0/24`) heeft toegang tot Management VLAN 99. Sales en HR worden direct gedropt. |

---

## 🔍 Systematische Troubleshooting & Incident Response

Tijdens de implementatie zijn reële netwerkstoringen gestructureerd gediagnosticeerd en gedocumenteerd (*Symptoom → Diagnose → Oorzaak → Oplossing*):

* **Incident 1: VLAN toewijzing interpretatie**
  * *Symptoom:* `show vlan brief` wekte de schijn dat access-poorten in verkeerde VLANs zaten.
  * *Diagnose:* Diepere inspectie uitgevoerd met `show interfaces <poort> switchport`.
  * *Oorzaak:* Layout en wrapping van `show vlan brief` binnen de Packet Tracer CLI.
  * *Oplossing:* Switchport operational mode geverifieerd; configuratie bleek 100% correct.
* **Incident 2: 100% Packet Loss bij Inter-VLAN Ping**
  * *Symptoom:* Ping tussen PC1 en PC3 faalde direct.
  * *Diagnose:* `ipconfig` gecontroleerd op het werkstation.
  * *Oorzaak:* Interface stond nog op statisch `0.0.0.0`.
  * *Oplossing:* IP-configuratie toegekend waarna routing direct functioneerde.
* **Incident 3: APIPA Adres Toewijzing (169.254.x.x)**
  * *Symptoom:* Werkstations ontvingen geen IP van de DHCP pool.
  * *Diagnose:* DHCP discover packets kwamen niet tijdig door tijdens interface state wisseling.
  * *Oplossing:* `ipconfig /release` en `ipconfig /renew` geforceerd.
* **Incident 4: Management VLAN Onbereikbaar voor IT**
  * *Symptoom:* Zelfs geautoriseerde IT-werkplekken konden de Core Switch niet pingen op poort 99.
  * *Diagnose:* `show ip interface brief` op Switch0 toonde geen actieve SVI.
  * *Oplossing:* `interface vlan 99` geconfigureerd met IP en `ip default-gateway 192.168.99.1` ingesteld.
* **Incident 5: ACL Verkeersfiltering Richting Inversie (In vs Out)**
  * *Symptoom:* Na activeren van ACL 3 kon Sales alsnog communiceren met het managementnetwerk.
  * *Diagnose:* ACL was inbound toegepast op de management sub-interface in plaats van outbound.
  * *Oplossing:* Filterrichting gecorrigeerd naar `out` op `GigabitEthernet0/0.99`, waardoor inkomend verkeer naar de management-VLAN effectief werd tegengehouden.

---

## 📁 Documentatie & Bewijslast

* 📝 **Gedetailleerd Voortgangsverslag:** [`documentatie/Lab3_Voortgangsverslag.md`](documentatie/Lab3_Voortgangsverslag.md)
* 🌐 **Packet Tracer Topologiebestand:** [`topology.pkt`](topology.pkt)
* 📸 **Screenshots & Verificatie:** [`screenshots/`](screenshots/)
