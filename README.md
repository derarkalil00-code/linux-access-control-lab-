# Linux Access Control Lab

## Problemstilling
I mange virksomheder har medarbejdere adgang til flere filer, end de har brug for.
Hvis en konto bliver kompromitteret, eller en medarbejder skifter afdeling uden at
miste sin gamle adgang, kan følsomme data blive set af de forkerte.
Hvordan kan man i Linux sikre, at hver afdeling kun har adgang til sine egne filer?

## Formål
At opsætte og teste adgangsstyring i Linux med brugere, grupper og filrettigheder
efter princippet om least privilege – og at løse en typisk supportsag, hvor en
medarbejder skifter afdeling.

## Miljø
- Ubuntu (WSL) på Windows
- Værktøjer: groupadd, useradd, chown, chmod, usermod, gpasswd, getent

## Design
| Bruger | Afdeling (gruppe) | Mappe       | Adgang til andre mapper |
|--------|-------------------|-------------|-------------------------|
| anna   | it                | /firma/it   | Nej                     |
| carl   | hr                | /firma/hr   | Nej                     |
| emil   | salg              | /firma/salg | Nej                     |

Mapperne har rettigheden 770: ejer og gruppe har fuld adgang, alle andre ingen.

## Opsætning
### 1. Grupper
    sudo groupadd it
    sudo groupadd hr
    sudo groupadd salg
Forklar med én sætning, hvad en gruppe er.

### 2. Brugere
    sudo useradd -m -G it anna
    sudo useradd -m -G hr carl
    sudo useradd -m -G salg emil
![Brugere og grupper](02-brugere.png)

### 3. Mapper og rettigheder
    sudo mkdir -p /firma/it /firma/hr /firma/salg
    sudo chown root:it /firma/it
    sudo chown root:hr /firma/hr
    sudo chown root:salg /firma/salg
    sudo chmod 770 /firma/it /firma/hr /firma/salg
![Mapper](03-mapper.png)
Forklar, hvad 770 betyder (7 = rwx, 0 = ingen adgang).

## Test
| Test                         | Forventet | Resultat |
|------------------------------|-----------|----------|
| anna opretter fil i /firma/it | Virker    |  Virker |
| anna åbner /firma/hr          | Afvist    |  Permission denied |
| emil åbner /firma/it          | Afvist    |  Permission denied |
![Test](04-test.png)

## Fejlfinding: medarbejdere skifter afdeling
**Sag:** Emil flytter fra Salg til HR, og Carl flytter fra HR til Salg.

    sudo usermod -aG hr emil
    sudo gpasswd -d emil salg
    sudo usermod -aG salg carl
    sudo gpasswd -d carl hr

Den gamle adgang fjernes også – ellers ville de have adgang til to afdelinger,
hvilket bryder least privilege.
![Fejlfinding](05-fejlfinding.png)

## Hvor ligger tingene?
- Brugere: /etc/passwd
- Grupper: /etc/group
- Hjemmemapper: /home/anna, /home/carl, /home/emil
- Firmamapper: /firma/
![Oversigt](06-oversigt.png)

## Fejl jeg mødte undervejs
Skriv om dine tastefejl (sodu, grouppadd) og om ls -l firma vs. /firma,
og hvad du lærte af dem.

## Hvad jeg lærte:

- **Least privilege i praksis:** Ved at give hver afdeling sin egen gruppe og mappe
  med rettigheden 770 kunne jeg sikre, at medarbejdere kun har adgang til det, de har
  brug for. "Permission denied" er ikke en fejl, men et tegn på, at sikkerheden virker.
  
- **Hvad tallene i chmod betyder:** 7 betyder læse, skrive og åbne (rwx), og 0 betyder
  ingen adgang. 770 giver ejer og gruppe fuld adgang og alle andre ingen.
  
- **Fjern altid den gamle adgang:** Da Emil og Carl skiftede afdeling, var det ikke nok
  at give dem den nye gruppe. Jeg skulle også fjerne den gamle, ellers ville de have
  adgang til to afdelinger.
  
- **Absolut og relativ sti:** `ls -l firma` virkede ikke, men `ls -l /firma` gjorde.
  Med `/` starter Linux fra roden, uden `/` leder den i den mappe, man står i.
  
- **Fejlbeskeder hjælper:** Mine tastefejl som `sodu` og `grouppadd` gav "command not
  found". Jeg lærte at læse fejlbeskeden og rette kommandoen i stedet for at gå i stå.

## Næste skridt
Fx: bruge chmod 2770, så nye filer automatisk tilhører afdelingens gruppe.
