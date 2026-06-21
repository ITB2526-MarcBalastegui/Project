# Errors abans d'instal·lar Proxmox

## Hardware

La BIOS no detectaba el disc

# Errors al instal·lar Proxmox
### Freeze de la GPU NVIDIA
L'instal·lador de Proxmox es congelaba al arrencar. La causa era que el processador i5-10400F no té gràfics integrats, així que el sistema depenia de la GPU de NVIDIA, i el driver del kernel col·lapsava amb ella al iniciar l'entorn gràfic de l'instal·lador
#### Solució: 
- Afegir el paràmetre `nomodeset` al arranc de GRUB, que li diu al kernel que no carregui el driver de vídeo natiu i utilitzi un de genèric.

### Disc no detectat
La instal·lació es realitzava correctament al disc seleccionat. El problema apareixia a l'hora del reboot: la BIOS no detectaba el disc, llavors era impossible arrencar Proxmox.
#### Solució:
- Problema del hardware: el cable SATA no funcionava bé (encara que era nou). Al canviar-lo per un altre, si que es va detectar.
  
<img width="625" height="257" alt="image" src="https://github.com/user-attachments/assets/06cf25b4-e4c8-4729-8960-3ffc19ebbf39" />
