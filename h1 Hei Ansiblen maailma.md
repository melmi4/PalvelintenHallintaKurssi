
## x)

# SSH public key - Login without password
  - SSH on työkalu, jolla voi kirjautua palvelimille turvallisesti.
  - Asennuksena käytetään: sudo apt-get -y install ssh, joka asentaa SSH palvelimen.

# Hello Ansible
  - Ansible on konfiguraationhallintatyökalu jolla kuvataan haluttu lopputila koodina IaC.
  - Asennuksena käytetään: sudo apt-get install ansible.


## a)

Asennus meni itsessään hyvin ja ilman isoja virheitä lukuunottamatta kahta käyttöönotossa tullutta ongelmaa.

Ongelma 1: Käyttäjällä ei ollut sudo oikeuksia, joten ajoin sudo apt-get update 

<img width="879" height="319" alt="image" src="https://github.com/user-attachments/assets/136c94c5-e519-4df0-a12f-093d7ef41cde" />

Tuloksena oli: elmi is not in the sudoers file. 

Korjaus: kirjauduin rootiksi ja lisäsin käyttäjän ryhmään. Ajoin komennon su - usermod -aG sudo elmi, exit

Ongelma 2: apt yritti käyttää DVD:tä pakettiläähhteenä. 

Sudo apt-get update antoi seuraavanlaisen errorin:

<img width="1317" height="489" alt="image" src="https://github.com/user-attachments/assets/9f120aeb-1866-4dd4-865e-b4105bbb36ff" />

Korjaus: Lisäsin Debianin verkkolähteet käyttämällä seuraavanlaisia komentoja:

sudo sed -i 's/^deb cdrom/# deb cdrom/' /etc/apt/sources.list
sudo nano /etc/apt/sources.list

ja lisäsin seuraavat rivit:

deb http://deb.debian.org/debian trixie main contrib non-free-firmware
deb http://deb.debian.org/debian trixie-updates main contrib non-free-firmware
deb http://security.debian.org/debian-security trixie-security main contrib non-free-firmware

Tämän jälkeen sudo apt-get update haki listat verkosta ja pakettien asennus toimi.


# b) Sshecrets

Tein puhtaan alun. Poistin ssh yhteyden ja varmistin että se ei toimi ollenkaan.

<img width="758" height="486" alt="image" src="https://github.com/user-attachments/assets/ffd186fb-09d8-4a24-9d87-42e6e580d397" />

Tämän jälkeen asensin ohjeiden mukaan ssh paketti joka asentaa ssh palvelimen. 
Enable saa demonin käynnistymään automaattisesti koneen mukana ja "--now" komento käynnistää sen heti.

<img width="859" height="496" alt="image" src="https://github.com/user-attachments/assets/5a686244-5c70-461e-bf59-766b06499353" />

Tarkistin tilan ja tuloksena oli Active ssh
<img width="870" height="496" alt="image" src="https://github.com/user-attachments/assets/ed8e9655-460f-47c9-91ed-e9ee1cb7b994" />

Lopuksi käytin "w" komentoa joka näyttää käyttäjän työpöytäistunnon sekä SSH istunnon.
<img width="876" height="116" alt="image" src="https://github.com/user-attachments/assets/b44df3a1-f1f5-4d74-90fb-7accefc22678" />

# c) Pubkey

Aluksi loin avainparin komennolla ssh-keygen. Hyväksyin oletukset ja painoin enteriä kolme kertaa.

<img width="842" height="512" alt="image" src="https://github.com/user-attachments/assets/5879fe1d-9c1d-49da-801e-2c16ba6192de" />

Tämän jälkeen komento ls -la $HOME/.ssh/ näytti kaksi uutta tiedostoa, joka oli yksityinen avain ja julkinen avain.

<img width="833" height="153" alt="image" src="https://github.com/user-attachments/assets/a30dd61c-0eb1-47c5-bc54-d11b4b0cadd2" />

Seuraavaksi vein julkisen avaimen localhostiin ja lisäsin sen komenolla: ssh-copy-id localhost. 

<img width="892" height="60" alt="image" src="https://github.com/user-attachments/assets/14cc0584-f51b-4eac-ae44-fba2e81b33c7" />

<img width="988" height="263" alt="image" src="https://github.com/user-attachments/assets/02dffe7a-5f65-41fb-be49-3bfe4c7ad8c9" />

Varmistin vielä, että avain on tallessa komennolla: cat $HOME/.ssh/authorized_keys

<img width="950" height="82" alt="image" src="https://github.com/user-attachments/assets/3af0f372-5513-402e-93c0-3bc13aebe487" />

Lopuksi oli testaus, jossa varmistin että kirjautuminen onnistuu ilman salasnaa ja komento "w" näytti ssh istunnon.
<img width="907" height="338" alt="image" src="https://github.com/user-attachments/assets/366a1dd6-a4b8-4291-bab3-44531456e7b9" />

# d) Hei Ansible

Aluksi varmistin että ssh kirjautuminen localhostiin toimii ilman salasanaa ja tämän jälkeen asensin ansiblen komenolla: sudo apt-get -y install ansible micro bash-completion tree

lopuksi tarkistin ansible version

<img width="1098" height="206" alt="image" src="https://github.com/user-attachments/assets/27723041-0bce-4ad8-9218-f890c22650cc" />

Tein kansion komenolla: mkdir ansible ja loin host.ini ivertaarion sen alle.

<img width="951" height="36" alt="image" src="https://github.com/user-attachments/assets/df01a93b-1ff8-49ba-9ab3-cbc259dd7a68" />

Testasin seuraavaksi ansiblea ssh:n yli. 

<img width="1090" height="106" alt="image" src="https://github.com/user-attachments/assets/0f6ea052-7aa1-4829-b5d6-957c30731849" />

Testasin miten toimii ilman -i komentoa. 

<img width="1086" height="69" alt="image" src="https://github.com/user-attachments/assets/4e0ecb85-74f7-4c33-8f8e-64d7f209a9cc" />

Määritin site.yml, joka määrittää mitkä roolit ajetaan millekin koneelle. 
<img width="1140" height="129" alt="image" src="https://github.com/user-attachments/assets/cc2b8bfb-9a03-466d-9e83-565a0a5ea907" />

Virhe 1. the role hello ei löytynyt
Syy: Olin kirjoittanut rivin 1 muotoon -hosts: all, eli viivan jälkeen puuttui välilyönti. Korjasin tämän jälkeen virheen.

<img width="1086" height="160" alt="image" src="https://github.com/user-attachments/assets/5019e87a-0b1a-4575-9604-f1590551d653" />

Playbook ajo.

<img width="1034" height="266" alt="image" src="https://github.com/user-attachments/assets/79859973-c5c5-4e25-8913-482931272b3a" />

Tree lopputulos.

<img width="1089" height="216" alt="image" src="https://github.com/user-attachments/assets/e69132f5-b80f-4575-93d9-421fba79b89f" />








