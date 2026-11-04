# h1 Hei Ansiblen maailma

## x)

SSH public key - Login without password
- SSH:lla kirjaudutaan palvelimelle ilman salasanaa. SSH:ta käyttävät taustalla myös esimerkiksi git, rsync ja Ansible.
- Ensin asennetaan ssh-paketti ja käynnistetään palvelu.
- Sitten luodaan avainpari komennolla ssh-keygen ja julkinen avain kopioidaan kohdekoneelle komennolla ssh-copy-id.
- Julkinen avain voi olla useilla koneilla, mutta yksityinen avain pidetään vain omalla koneella.
- Ongelmatilanteissa apuna ovat ssh -v ja journalctl --follow.
- Miksi avainpari on turvallisempi kuin salasana?

Hello Ansible
- Ansible on työkalu, jolla koneiden asetukset kirjoitetaan koodina.
- Toimii SSH:n kautta, joten hallittavalla koneella tarvitaan vain SSH ja Python
- Esimerkkinä on neljä tiedostoa: hosts.ini (koneiden lista), ansible.cfg (asetukset), site.yml (mikä kone saa minkä roolin) ja roolin main.yml (varsinaiset tehtävät).


## a) 
Debian 13 asennettu aikaisemmin ongelmitta.

## b)
Aloitin toimet päivittämällä paketit ja asentamalla SSH-palvelimen 

<img width="1519" height="731" alt="VirtualBox_Debian_h1_v1" src="https://github.com/user-attachments/assets/17923a17-a947-4e73-8afa-b74047b13c97" />

Käynnistin ssh:n komennolla sudo systemctl enable --now ssh ja yhdistin omaan koneeseen komennolla ssh localhost

<img width="1520" height="718" alt="VirtualBox_Debian_h1_v2" src="https://github.com/user-attachments/assets/03b59ff9-09ec-471d-b59d-0757889846d4" />

Testasin ssh yhteyden who komennolla ja sen jälkeen suljin yhteyden exit komennolla.

<img width="663" height="125" alt="VirtualBox_Debian_h1_v3" src="https://github.com/user-attachments/assets/144dc664-a395-42ce-8446-33cbc632f6c1" />

## c)
Tein avainparin komennolla ssh-keygen jonka jälkeen kopioin julkisen avaimen koneelle ssh-copy-id localhost.

<img width="1518" height="727" alt="VirtualBox_Debian_h1_v4" src="https://github.com/user-attachments/assets/6fe77cfb-881d-4449-a361-967d517e96c5" />

Ssh localhost päästi sisään ilman salasanaa joten kirjautuminen avaimella toimi.

<img width="992" height="266" alt="VirtualBox_Debian_h1_v5" src="https://github.com/user-attachments/assets/a221b119-db0c-4e31-9c20-f5e4ae36720e" />

## d)

Aloitin tekemällä asentamalla Ansiblen komennolla sudo apt-get install ansible micro bash-completion tree. Mukana micro tekstieditori ja bash completion joka täydentää komentoja tabilla. Tree näyttää hakemiston puumaisena rakenteena.

Tein Ansible kansion komennolla mkdir ansible/ ja siirryin kansioon.

Tein hosts.ini tiedoston jossa yksi rivi, localhost, joka kertoo Ansiblelle mitä konetta hallitaan.

Seuraavaksi kokeilin komentoa ansible all -a 'uptime' -i hosts.ini. joka ajaa uptime komennon (kauan koneet ollut käynnissä ja kuinka kuormitettu se on). Korjasin myös Pythonin antaman varoituksen  lisäämällä tiedostoon ansible_python_interpreter=/usr/bin/python3

<img width="1510" height="728" alt="VirtualBox_Debian_h1_v7" src="https://github.com/user-attachments/assets/23022656-1da2-4679-bd89-14e106f326d4" />

Seuraavaksi tein playbookin (tiedosto johon kirjoitetaan automatisoitavia tehtäviä) site.yml ja roolin hello, joka luo tiedoston /tmp/hello-ansible. Ensimmäinen ajo epäonnistui, koska roolia ei ollut vielä tehty. Tein korjauksen tiedostoon ja ajo onnistui.

<img width="1504" height="551" alt="VirtualBox_Debian_h1_v8" src="https://github.com/user-attachments/assets/7d5b4a23-5162-42cd-be26-3c1e677c02ad" />

Tarkistin tuloksen komennolla ssh localhost 'cat /tmp/hello-ansible' ja sain haluamani tuloksen. 

<img width="1266" height="101" alt="VirtualBox_Debian_h1_v9" src="https://github.com/user-attachments/assets/06441709-7766-428e-9ff2-17e90dfed0da" />


## Lähteet

Karvinen 2026: SSH public key - Login without password: https://terokarvinen.com/ssh-public-key-login-without-password/

Karvinen 2026: Hello Ansible https://terokarvinen.com/hello-ansible/

Geeksforgeeks: https://www.geeksforgeeks.org/
