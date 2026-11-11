# h2 - Voileipä

## x)

  ## Sudo without password
  
  - Ansible tarvitsee orjalla rootin, ja yksi tapa on sallia sudo ilman salasanaa tietylle ryhmälle.
  - Sääntö kirjoitetaan visudolla /etc/sudoers.d/-hakemistoon, ja toinen root-shell toimii varmistuksena.
  
  ## Passwordless Sudo with Ansible
  
  - Rooli luo ryhmän ja käyttäjän, lisää ssh-avaimen ja kirjoittaa sudoers-säännön.
  - Ensimmäisellä ajolla tarvitaan --ask-become-password, koska sääntö ei vielä ole voimassa.
  
  ## xkcd 149: Sandwich
  
  - Sarjakuvassa “Make me a sandwich” saa kielteisen vastauksen, mutta “sudo make me a sandwich” onnistuu.
  - Kuvaa sitä, että sudo on oikeuksien korotus

  ## Ansiblen sisäänrakennettu dokumentaatio

  MODULE

  - ansible.builtin.copy kopioi tiedoston tai hakemistorakenteen hallintakoneelta orjakoneelle. 
  - Samalla voi asettaa tiedoston omistajan, ryhmän ja oikeudet, vaikka tiedosto olisi jo orjalla. 
  - Jos tiedostoon pitää sijoittaa muuttujia, tulosteen mukaan kannattaa käyttää template-moduulia.
  - Windows-koneille on oma win_copy

  OPTIONS

  - content: kirjoittaa tiedoston sisällön suoraan ilman src:tä. Toimii vain, kun dest on tiedosto. Tiedosto luodaan, jos sitä ei ole.
  - dest: Absoluuttinen polku orjalla, jonne tiedosto kirjoitetaan. Jos src ja dest ovat tiedostoja, dest:n ylähakemistoa ei luoda ja tehtävä epäonnistuu.
  - src: Paikallinen polku tiedostoon, joka kopioidaan etäpalvelimelle. Polku voi olla joko absoluuttinen tai suhteellinen. Jos polku osoittaa kansioon, koko kansio kopioidaan sisältöineen (rekursiivisesti).
  - owner: Käyttäjä, joka omistaa tiedoston. 

## a)

Aloitin tehtävän tekemällä käyttäjän testaaja, ryhmän sudoless ja sudoers-säännön käsin komennoilla sudo adduser testaaja, sudo groupadd sudoless, sudo adduser testaaja sudoless

<img width="774" height="444" alt="VirtualBox_Debian_h2_v1" src="https://github.com/user-attachments/assets/a07f4bd4-7e84-4ee7-aa9a-ae6bce13d27f" />

Seuraavaksi ajoin komennon sudo visudo /etc/sudoers.d/sudoless ja lisäsin komennon '%sudoless ALL = (ALL) NOPASSWD: ALL' joka tarkoittaa, että ryhmän sudoless jäsenet saavat suorittaa kaikki komennot kenenä tahansa käyttäjänä ilman salasanaa. Otin ssh yhteyden ja testasin toimivuuden: sudo -k ja sudo echo "toimii" ja ne toimivat ilman salasanakyselyä.

<img width="898" height="318" alt="VirtualBox_Debian_h2_v3" src="https://github.com/user-attachments/assets/f5016239-7446-4d9e-aa34-9582c6189492" />

## b)

Aloitin luomalla tiedoston mkdir -p roles/testaaja/tasks. Sen jälkeen avasin tiedoston editorilla komennolla nano roles/testaaja/tasks/main.yml ja kirjoitin siihen roolin tehtävät.

<img width="944" height="439" alt="h2_b nano" src="https://github.com/user-attachments/assets/02c22c38-498e-409e-a39c-2e5e6e1f5c40" />

Seuraavaksi loin playbookin site.yml joka kertoo Ansiblelle mitä ajetaan ja kenelle. hosts: all ajaa tehtävät kaikille koneille, become: true ajaa ne sudolla, koska käyttäjän luonti ja sudoers-tiedoston kirjoitus vaativat root-oikeudet, ja roles: testaaja ottaa käyttöön roolin.

<img width="1647" height="773" alt="VirtualBox_Debian_h2_v5" src="https://github.com/user-attachments/assets/7a178e36-6fbc-4328-80b7-cde5fc562fca" />

Testasin toimivuuden ajamalla ansible-playbook site.yml --ask-become-pass. Ensimmäisellä kerralla tuli muutoksia ja toisella ei.

<img width="1792" height="839" alt="h2_idempotenssi" src="https://github.com/user-attachments/assets/ad076080-8244-4eab-9b32-df36a0c82c37" />

## c) 

Aloitin siirtymällä ansible hakemistoon ja tein uuden kansion mkdir -p roles/paketit/tasks. Sitten muokkasin main.yml tiedostoa nano roles/paketit/tasks/main.yml ja lisäsin rooleihin paketit.

<img width="1920" height="936" alt="h2_c-kohta2" src="https://github.com/user-attachments/assets/e3464e30-d960-43fb-a642-a2d0e3d1d1cb" />

Asensin kaksi pakettia apt-moduulilla (roles/paketit/tasks/main.yml). 

<img width="1792" height="898" alt="h2_c-kohta" src="https://github.com/user-attachments/assets/67d22a10-4541-4984-b606-209214b2f984" />

Ajoin ansible-playbook site.yml --ask-become-pass ja ensimmäiseillä ajolla tuli muutoksia ja toisella ei.

Testasin pakettien toimivuuden ja kaikki toimivat.

<img width="647" height="266" alt="h2_c4" src="https://github.com/user-attachments/assets/99726670-9767-4814-bc54-4894ed96a398" />

## d)

Kirjoitin orjalle usean rivin tiedoston copy-moduulilla. dest määrää polun, content sisällön, ja owner, group ja mode omistajan, ryhmän ja oikeudet. Oikeudet annoin oktaalilukuna(0640). oikeudet ovat siis -rw-r-----. Omistaja (testaaja) saa lukea ja muokata tiedostoa, group vain lukea ja muut eivät mitään.

<img width="955" height="925" alt="h2_d1" src="https://github.com/user-attachments/assets/64c674ba-6a7c-4ea8-a402-915bc7f2dbce" />

Lisäsin site.yml:ään roolin tiedosto ja ajoin playbookin. Ansiblen copy-tehtävä loi tiedoston /home/testaaja/viesti.txt ajon aikana ja asetti omistajan ja oikeudet. 

<img width="955" height="925" alt="h2_d2" src="https://github.com/user-attachments/assets/12fba2a7-d25a-4f18-b2e5-3a2b0fecea94" />

Omalla tunnuksellani (tuomasm) tiedoston tarkastelu epäonnistui (Permission denied), koska olen ryhmässä “muut”. Kirjauduin ssh:lla testaajaksi ilman salasanaa, jolloin stat viesti.txt näytti oikeudet 0640/-rw-r-----, omistajan ja ryhmän testaaja, ja cat viesti.txt tulosti tiedoston kolme riviä.

<img width="955" height="628" alt="h2_d3" src="https://github.com/user-attachments/assets/69e3ed9c-36be-40e0-8649-804bf6af4d84" />

## e)

Kokeilin e-kohdassa lineinfile-moduulia, jota ei ole käsitelty kurssilla. Moduuli varmistaa, että tietty rivi on tiedostossa: jos riviä ei ole, se lisätään, ja jos se on jo olemassa, mitään ei tehdä. Loin roolin rivi (roles/rivi/tasks/main.yml) ja lisäsin sen site.yml:ään.

<img width="955" height="925" alt="h2_e2 korjattu" src="https://github.com/user-attachments/assets/447f85a7-dfab-4458-bca3-8f5f0688829b" />

Lisäsin uuden roolin 'rivi' playbookiin site.yml. Ansible ajaa sen vasta, kun se on mainittu roles:-listassa, ja nimen pitää vastata kansion nimeä roles/rivi/. become: true ajaa kaikki roolit sudolla, joten erillistä sudo-asetusta ei tarvittu.

<img width="955" height="925" alt="h2_e3" src="https://github.com/user-attachments/assets/f23540ba-ddfb-4050-a4ef-1b8568047e55" />

Ajoin playbookin komennolla ansible-playbook site.yml --ask-become-pass. Aiemmat tehtävät (apt, copy) olivat ok, koska ne oli jo tehty edellisissä kohdissa, ja uusi tehtävä rivi : lineinfile oli changed: se loi tiedoston /tmp/ansible-testi.txt ja lisäsi siihen rivin. Yhteenvedossa oli ok=8 changed=1 failed=0, eli vain uusi tehtävä muutti tilaa eikä mikään epäonnistunut.

<img width="955" height="925" alt="h2_e5" src="https://github.com/user-attachments/assets/5891cc13-d84a-4756-b134-bb2520843544" />


## Lähteet

Karvinen 2026: Sudo without password: https://terokarvinen.com/passwordless-sudo/

Karvinen 2026: Passwordless Sudo with Ansible: https://terokarvinen.com/passwordless-sudo-with-ansible/

Munroe 2006: xkcd 149: Sandwich: https://xkcd.com/149/

