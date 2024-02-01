# Opdracht 2 - Een databankserver opzetten in een virtuele machine

## Inhoud

- [Inleiding](#inleiding)
- [Probleemstelling](#probleemstelling)
- [Opdracht](#opdracht)
  - [Stap 1 - VirtualBox configureren](#stap-1---virtualbox-configureren)
  - [Stap 2 - Virtuele machine aanmaken](#stap-2---virtuele-machine-aanmaken)
  - [Stap 3 - Configuratie databankserver](#stap-3---configuratie-databankserver)
    - [Installatie en configuratie MySQL](#installatie-en-configuratie-mysql)
    - [Configuratie van de databank](#configuratie-van-de-databank)
    - [MySQL Workbench](#mysql-workbench)
  - [Stap 4 - Afsluiten](#stap-4---afsluiten)
- [Evaluatie](#evaluatie)

## Inleiding

In dit labo zal je een virtuele machine (VM) opzetten binnen VirtualBox en er een databankserver op installeren. Deze VM (of op zijn minst de werkwijze om er zo één op te zetten) zal op verschillende momenten tijdens de opleiding nog van pas komen. Je kan deze databankserver bijvoorbeeld gebruiken als testdatabank voor je programmeerproject, voor de cursussen rond databanken, enz.

De instructies in deze opdracht zijn wat bondiger dan bij de eerste opdracht en bevatten geen screenshots. Je kan echter de volledige installatie en configuratie van een databank-VM volgen via deze [video](https://hogent.cloud.panopto.eu/Panopto/Pages/Viewer.aspx?id=1ce2f9a0-4e9e-4b08-aa99-acae00da47c4).

> **Opgelet**: Deze video is al enkele jaren oud. Sommige stappen zal je dus moeten aanpassen aan de stappen hieronder.

## Probleemstelling

Voor sommige vakken in de opleiding maak je gebruik van een databank, bv. MySQL of Microsoft SQL Server. Meestal krijg je dan de instructie om de nodige software op je laptop te installeren. Dit zijn processen die voortdurend op de achtergrond blijven lopen en dus ook systeembronnen gebruiken, ook wanneer je geen databank nodig hebt.

Een mogelijke oplossing is om dit soort software te installeren in een virtuele machine. Het voordeel hiervan is dat je geen databanksysteem moet installeren op je laptop zelf. Als je de databank niet nodig hebt, zet je de virtuele machine uit zodat die je pc niet meer belast. Een bijkomend voordeel van een virtuele machine is dat je de databank dan aanspreekt over een (virtueel) netwerk. Dit is realistischer, want in de praktijk is een databankserver ook typisch een afzonderlijke machine die over het netwerk aangesproken wordt.

## Opdracht

### Stap 1 - VirtualBox configureren

Controleer of je de volgende applicaties geïnstalleerd hebt op je laptop, bij voorkeur via de package manager uit de vorige opdracht:

- VirtualBox met het VirtualBox Extension Pack
- MySQL Workbench
- Optioneel: FileZilla of Cyberduck

In VirtualBox moet je via **File** > **Tools** > **Network Manager** een host-only netwerkadapter aanmaken met volgende instellingen:

- IPv4-adres: `192.168.56.1`
- Netwerkmasker: `255.255.255.0`
- DHCP-server: `aan`
- DHCP-server adres: `192.168.56.100`
- Laagste adres: `192.168.56.101`
- Hoogste adres: `192.168.56.254`

> **Opgelet**: Op macOS dien je een host-only network (niet adapter) aan te maken. Controleer het IP-adres en netwerkmasker van het aangemaakte netwerk. Verder hoef je het niet aan te passen.

### Stap 2 - Virtuele machine aanmaken

Ga naar de website <https://www.osboxes.org/> en zoek een VirtualBox-image voor Ubuntu 22.04.

Maak in VirtualBox een nieuwe VM aan. Doorloop de wizard voor een nieuwe VM:

- Geef de VM minstens 2GB RAM.

- Koppel het **.vdi**-bestand van OSBoxes.org als Virtual Hard Disk aan de VM. Voor je dit doet, plaats je dit bestand best in dezelfde map waar de VM staat.

  Doordat je ook voor het olod "Operating Systems" een Ubuntu VM moet opzetten, zou het kunnen dat je deze foutmelding krijgt:

  ```
  Cannot register the hard disk ‘C:\_VMs\SC9u2\sc9u2.vdi’ {ca2bdc6a-a487-4e57-9fcd-509d0c31d86d} because a hard disk ‘C:\_VMs\SC9u2\sc9u1.vdi’ with UUID {ca2bdc6a-a487-4e57-9fcd-509d0c31d86d} already exists.
  ```

  Genereer in dit geval een **nieuw UUID** voor het .vdi-bestand alvorens de koppeling opnieuw te proberen. Je voert hiervoor de volgende stappen uit:

  1. Open de **Command Prompt** en navigeer naar de map waar je Oracle Virtual Box geïnstalleerd hebt. Dit is bijvoorbeeld “C:\Program Files\Oracle\VirtualBox”.

  2. Voer onderstaand commando uit waarbij je uiteraard het pad naar jouw .vdi-bestand als argument meegeeft. Bijvoorbeeld:

     ```
     VBoxManage internalcommands sethduuid "C:\_VMs\SC9u1\sc9u1.vdi"
     ```

  Een andere optie is om met dezelfde VM te werken als die voor het olod "Operating Systems".

- Eens de VM is aangemaakt, kijk je nog volgende instellingen na:
  
  - Onder **Display**: Zet Video Memory op de maximale waarde.
  - Onder **Network**: Zorg dat de VM twee netwerkadapters heeft: enerzijds een NAT interface en anderzijds de host-only adapter (of netwerk) dat je in Stap 1 hebt aangemaakt.
  
- Start de VM op en meld je aan als gebruiker `osboxes` met wachtwoord `osboxes.org`.

> **Opgelet:** De VM heeft een QWERTY-toetsenbordindeling! Je kan dit aanpassen via het **Region & Language** scherm van de **Settings** app.

Zoek op welk IP-adres je host-only verbinding gekregen heeft:

- Klik op de *system tray* (rechtsboven) of open **Settings** en ga naar **Network**.
- Je ziet twee Ethernet-interfaces, `enp0s3` en `enp0s8`. Klik op het tandwiel-symbool bij `enp0s8`.
- Zoek het IP-adres dat is toegekend aan deze interface. Dit adres begint normaliter met `192.168.56`.

Eens je dit adres gevonden hebt, open dan een command-line interface op je fysieke systeem (dus niet in de VM), en test of je kan verbinden met je VM:

```bash
ping 192.168.56.2
```

> **Opgelet:** In dit voorbeeld hebben we het IP-adres `192.168.56.2` gebruikt. Dit dien je uiteraard te vervangen door het adres dat is toegekend aan jouw VM.

Als dit niet lukt, dan heb je de netwerkadapters van je VM niet correct ingesteld. Stel deze opnieuw in alvorens verder te gaan met de volgende stappen.

Voor het gemak gaan we nu een vast IP-adres toekennen aan de VM:

- Keer terug naar het scherm met de instellingen van de interface `enp0s8`.
- Kies het tabblad **IPv4** en geef volgende instellingen in:
  - IPv4 Method: `manual`
  - Address: `192.168.56.20`
  - Netmask: `255.255.255.0`
  - Gateway: `LEEG`
- Zet de interface uit en terug aan.
- Verifieer of de netwerkinstellingen zijn toegepast door volgend commando uit te voeren in **Terminal**:

   ```bash
   ip a
   ```

Test nu opnieuw of je VM bereikbaar is vanop je fysieke systeem met volgend commando:

```bash
ping 192.168.56.20
```

Een aantal optionele, maar mogelijks handige instellingen om aan te passen zijn:

- Schakel de screen lock uit.
- Laat de gebruiker **osboxes.org** automatisch inloggen.
- Installeer handige applicaties zoals Visual Studio Code.
- Pas het wachtwoord van de gebruiker **osboxes** aan. Schrijf dit wachtwoord zeker op in de beschrijving van de VM via **Settings** > **General** > **Description**.
- Probeer of je met FileZilla/Cyberduck bestanden van/naar de VM kan kopiëren. Dit zal extra configuratie vereisen.

> **Tip:** Maak regelmatig een snapshot van de VM zodat je bij problemen altijd kan terugkeren naar een vorig punt!

### Stap 3 - Configuratie databankserver

#### Installatie en configuratie MySQL

Om de databankserver te configureren voer je volgende stappen uit in de virtuele machine zelf:

- Open **Terminal** en installeer MySQL:

  ```bash
  sudo apt update
  sudo apt install -y mysql-server
  ```

- Controleer of de service draait:

  ```bash
  systemctl status mysql
  ```

- Controleer welke netwerkpoorten in gebruik zijn en stel vast dat MySQL enkel luistert op de "loopback interface":

  ```bash
  sudo ss -tlnp
  ```

  Waaraan zie je dit?
- Zorg ervoor dat MySQL luistert naar alle netwerkinterfaces door het bestand **/etc/mysql/mysql.conf.d/mysqld.cnf** aan te passen. Zoek in dit bestand naar de regel die het **bind-address** instelt op **127.0.0.1** en verander dit naar **0.0.0.0**.
- Start MySQL opnieuw op:

  ```bash
  sudo systemctl restart mysql
  systemctl status mysql
  ```

- Controleer met `ss -tlnp` of de wijziging effect had. Waaraan zie je dit? Wat is het verschil met de vorige uitvoer van dit commando?

#### Configuratie van de databank

De verdere configuratie van de databank gebeurt via de MySQL Console. Deze kan je openen als volgt:

```bash
sudo mysql
```

Achter de prompt `mysql>` kan je commando's intikken. Voer alvast volgende commando's uit:

```mysql
use mysql;
alter user 'root'@'localhost' identified with mysql_native_password by 'letmein';
```

Deze commando's kennen een wachtwoord (**letmein**) toe aan de **root** gebruiker, wat standaard niet het geval is op Ubuntu.

De **root** gebruiker mag weliswaar enkel lokaal aanmelden, dus niet via het netwerk. Maak daarom een extra **admin** gebruiker aan met volgende commando's:

```mysql
create user 'admin'@'%' identified by 'letmein'; 
grant all privileges on *.* to 'admin'@'%' with grant option;
flush privileges;
exit;
```

Deze **admin** gebruiker kan je gebruiken om te verbinden met MySQL vanop je fysieke systeem.

Vooraleer je dat doet, werk je eerst de installatie van MySQL af met:

```bash
sudo mysql_secure_installation
```

Antwoord als volgt op de vragen die dit script jou stelt:

- Validate Password activeren? NEE
- Wachtwoord voor **root** aanpassen? NEE
- Remove anonymous users? JA
- Disallow root login remotely? JA
- Remove test database? JA
- Reload Privileges table? JA

> **Opgelet**: Doordat de **root** gebruiker nu een wachtwoord heeft, moet je de MySQL Console vanaf nu openen met `mysql -u root -p` in plaats van met `sudo mysql`.

#### MySQL Workbench

Test nu of je vanop je fysieke systeem via MySQL Workbench kan verbinden met MySQL in je VM. Start MySQL Workbench op en doe het volgende:

- Maak een nieuwe verbinding aan.
- Kies een naam voor de verbinding (bv. `UbuntuVM-admin`).
- Hostnaam: `192.168.56.20`
- Username: `admin`
- Test eerst de verbinding en sla het wachtwoord op in de "vault".

Maak nu via de Workbench een databank én een gebruiker aan die je kan gebruiken in het programmeerproject:

- Open de verbinding die je hierboven gemaakt hebt.
- Creëer een nieuw databankschema, bv. met naam `appdb`.
- Maak vervolgens een nieuwe gebruiker aan, bv. met naam `appusr`.
  - Limit to hosts matching: `%`.
  - Kies een wachtwoord.
  - Schema privileges: zorg dat de gebruiker `appusr` alle rechten heeft op `appdb`, behalve `GRANT OPTION`.

Test de nieuwe gebruiker uit door een nieuwe connectie aan te maken voor deze gebruiker. Gebruik dezelfde stappen als hierboven voor de admin-gebruiker, maar kies als "default schema" `appdb`.

### Stap 4 - Afsluiten

Om je VM veilig af te sluiten klik je op de *system tray* en kies je in het menu voor `Power Off/Log out` > `Power Off`.

In een terminal kan je het volgende commando gebruiken:

```bash
sudo poweroff
```

Gebruik telkens één van deze werkwijzen en sluit niet zomaar VirtualBox af!

## Evaluatie

Als je het labo volledig hebt uitgevoerd, toon je het resultaat aan je begeleider. 

Criteria voor beoordeling:

- De VM start op en je kan inloggen:
  - Toon de netwerkinstellingen op de VM.
  - Toon aan dat de host-only adapter correct geconfigureerd is d.m.v een ping van je fysieke systeem naar de VM.
  - Toon aan dat MySQL actief is op de VM en luistert op alle interfaces.
- Je kan MySQL Workbench gebruiken om een connectie aan te maken met de databankserver:
  - Je hebt een **werkende** connectie voor de admin-gebruiker
  - Je hebt een **werkende** connectie voor de applicatie-gebruiker
- Een degelijk en duidelijk opgemaakt verslag (in Markdown) is te vinden in de documentruimte van de groep. Hierin staan minstens volgende zaken:
  - Wie heeft wat gedaan (= taakverdeling)?
  - Wie was de verslaggever? Kies telkens een andere student.
  - De antwoorden op de vragen uit deze opdracht.
  - Struikelblokken die je hebt ervaren, en hoe je ze hebt opgelost.
- De cheat sheet werd aangevuld met nuttige commando's die je wenst te bij te houden voor later.
