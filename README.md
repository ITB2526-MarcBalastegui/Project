# Projecte: Sistema complet de Proxmox (EN PROGRÉS)

## Pasos previs a la instal·lació
- Per començar, utilitzarem Rufus per fer que l'USB amb l'imatge de Proxmox dins sigui Bootable.

### Hardware utilitzat:

Crucial BX500 240 Gb - Un disc SSD SATA3, amb velocitat de lectura de 540 MB/s. Suficient per al context: laboratori lleuger

![Crucial](img/preview.webp)

Veure: [Errors](Project/errors.md)


## Instal·lació

- Proxmox en mode Terminal UI amb el paràmetre "nomodeset" activat (per evitar un error de freeze per la GPU NVIDIA=
- En Target Harddisk Crucial BX500, amb el sistema de fitxers ext4
- IP del servidor: `192.168.1.50/24`

## Primera màquina del sistema: OPNSense

Abans de crear la màquina, haviem de deshabilitar el repositori enterprise, que requereix una suscripció pagada, i afegir el repositori no-subscription, el repo gratuit de Proxmox, vàlid pel que voldrem fer.

Aquesta es la màquina que farà de router/firewall.
Per a la ISO, seleccionem la arquitectura amd64 tipus dvd i fem la descàrrega directa a Proxmox (Opció `Download from URL`)

### Disseny de la xarxa

Com OPNSense necessita dos "cares" de la xarxa (WAN, la que apunta a internet, i LAN, la que mira a la xarxa interna), hem de crear una xarxa virtual més. 
- `vmbr0`: la que ja tenim que farà de WAN i per on OPNSense sortirà a internet.
- `vmbr1`: nova, sense tarjeta física, intern, que farà de LAN.

#### Diagrama de xarxa

El `vtnet0` (WAN) té la ip `192.168.1.18`, assignada pel router mitjançant DHCP. Mentre que el `vtnet1` (LAN) té la ip que hem escrit manualment, `10.10.10.1`

![Diagrama xarxa](img/diagrama-xarxa.png)

#### Conflicte de subxarxa

La LAN venía per defecte en `192.168.1.1/24` (Mateix rang que la xarxa física de casa (`192.168.1.X`). Això crea un conflicte d'enrutament ja que un dispositiu no sabria si un destí a `192.168.1.X` està en la xarxa local o a un altre costat de OPNSense. Per això vam canviar la LAN a `10.10.10.1/24`

### Paràmetres

| Paràmetres | Selecció | Explicació |
|-----------|-----------|-----------|
| ID | 100  |  |
| Nom | opnsense |  | 
| Machine | Default (1440fx) | Machine* clàssic, molt compatible i senzill, suficient per a un router. |
| BIOS | Default (SeaBIOS) | Firmware que arrenca la VM. BIOS clàsic/legacy. Igual, simple i compatible. |
| Bus/Device | SCSI | Tipus de controladora a la que es connecta el Virtual Disk. Aquesta es la opció recomanada. |
| SCSI Controller | VirtIO SCSI single | Model de controladora SCSI que s'emula. En paravirtualització* (Més ràpid)|
| Storage | local-lvm | On es guarda físicament el disc virtual de la VM |
| Disk size | 20 GiB | Mida del disc. Amb 20 GiB sobren ja que OPNSense es lleuger | 
| Sockets | 1 | CPUs físiques simulades | 
| Cores | 1 | Nuclis per socket | 
| Type | host | Model de CPU que veu la VM. Host significa que la VM veu la CPU real del servidor. Màxim rendiment | 
| Memory | 2048 MiB | RAM assignada | 
| Balloning Device | No | Tècinca de gestió dinàmica de RAM. Permet a Proxmoxreclamar RAM  que njo està utilitzant i donarsela auna altra VM que ho necessiti, optimitzant l'ús de la memòria. Tot i que es un concepte molt interessant, la base de OPNSense, FreeBSD no gestiona bé el ballooning i pot donar problemes d'estabilitat | 
| Bridge | vmbr0 | A quin switch virtual es connecta. Aquest es el bridge connectat a la tarjeta física, sortida a internet | 
| Network Model | VirtIO | El tipus de tarjeta de xarxa. No emula una tarjeta de xarxa real. Paravirtualitzada, més ràpida |  

**Paravirtualització:** La VM sap que es una VM i utilitza drivers per parlar directament amb el hipervisor sense fingir hardware real.

**Machine:** chipset de la placa base virtual que Proxmox li emula a la VM.

### Web UI

Per defecte OPNSense bloqueja l'accès al seu panel web desde la WAN (Mesura de seguretat llògica). 
Per solucionar-ho: 
- Desactivem temporalment el firewall, en el shell de la màquina, amb `pfctl -d`
- Entrem a la Web i creem una regla de firewall en `Firewall -> Rules -> WAN` que permet tràfic  TCP entrant desde 192.168.1.0 fins a la WAN address pel port HTTPS (443).
- Reactivem el firewall amb `pfctl -e`
  
<img width="1589" height="480" alt="image" src="https://github.com/user-attachments/assets/131b2f5e-5fc9-4bd5-ab7f-5e4bfe7f422e" />


#### Comprovació del funcionament

Per comprovar que OPNSense assigna IPs i funciona la sortida a internet, crearem una màquina Linux de prova (`Debian 13`)

## Segona màquina del sistema: Windows Server 2022

La màquina server que serà Active Directory


OPNSense ja porta per defecte una regla `Default allow LAN to any` que permet a las VMS internes sortir a internet. No necessitem configuració extra
