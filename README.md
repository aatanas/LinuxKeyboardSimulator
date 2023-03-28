# LinuxKeyboardSimulator

Cilj programa je napraviti simulaciju tastature. Program je napisan u C i assembly, i to za Linux 0.0.1.  

Program podržava sledeće funkcionalnosti:

- Čitanje rasporeda karaktera po sken
  kodovima.

- Čitanje tabele mnemonika.

- Čitanje niza sken kodova i njihova interpretacija
  
  - Posebne funkcionalnosti za shift, ctrl i alt

- Prikaz odgovarajućih karaktera na ekranu.

### Opis sistema

Kod svake tastature, pritisak i otpuštanje tastera se vezuje za
jedinstveni broj - scan code (***sc***  dalje u tekstu). Tastatura
operativnom sistemu prosleđuje ***sc***  pri svakom pritisku ili otpuštanju
tastera, operativni sistem ih tumači i prosleđuje dalje aplikacijama. U
ovoj simulaciji, posebni tasteri - shift, ctrl i alt će imati fiksirane
***sc*** . Za sve ostale tastere postoji spisak koji je naveden u datoteci
scancodes.tbl koji navodi odgovarajući ***sc***  za sva "mala" i "velika"
slova.



Fiksirane ***sc***  vrednosti:

| \-   | shift | ctrl | alt |
| ---- | ----- | ---- | --- |
| down | 200   | 201  | 202 |
| up   | 300   | 301  | 302 |



Scan code 400 u ovom sistemu predstavlja kraj datoteke.

Pored toga, ovaj sistem podržava ispis proizvoljnog ASCII karaktera
pri držanju alt, i ispisivanje unapred definisanih poruka pomoću prečica
uz ctrl taster.

Program treba prvo da pročita dve datoteke koje opisuju konfiguraciju
sistema - scancodes.tbl i ctrl.map. Nakon toga program počinje sa
interaktivnim radom i korisnik unosi naziv datoteke (na primer:
test1.tst) koja sadrži niz ***sc***  vrednosti. Ovi ***sc***  se tumače i
odgovarajući rezultat se ispisuje na ekranu. Tastatura ima četiri režima
rada:

- Bez kontrolnih tastera - ispisuju se "mala" slova pročitana iz
  scancodes.tbl. 

-  Pritisnuto shift - ispisuju se "velika" slova pročitana
  iz scancodes.tbl. 

-  Pritisnuto ctrl - unosi se mnemonik naveden u
  ctrl.map.

- Pritisnuto alt - unosi se ASCII kod.

*Napomena:* kod kontrolnog tastera alt se rezultat prikazuje tek kada se
pročita ***sc***  za otpuštanje tog tastera (302).

### Ulazni podaci

Ulazni podaci se navode preko tri odvojene datoteke:

- scancodes.tbl

- ctrl.map

- test1.tst - ime ove datoteke se unosi pri startu programa

#### scancodes.tbl

U ovom sistemu, datoteka scancodes.tbl će da ima spisak karaktera koji
treba da se vežu za ***sc*** , pritom, ***sc***  karaktera je dat implicitno -
počinje od 0 i inkrementira se za 1 za svaki karakter. Spisak karaktera
za "mala" i "velika" slova je odvojen znakom za novi red.

Primer datoteke:

```
abcd()1234                 
ABCD{}!"\#\$ 
```

Ovo bi bila datoteka koja opisuje sistem sa 10 ***sc*** . Karakter 'a' je
opisan sa ***sc***  0, karakter 'b' sa ***sc***  1, i tako dalje, do karaktera
'\$' koji je opisan sa ***sc***  9 dok je pritisnuto shift.

Maksimalan broj ***sc***  u sistemu je 128.

#### ctrl.map

Pored običnih slova program podržava i mnemonike, ili tzv. prečice.
Mnemonici su zadati u posebnoj datoteci - ctrl.map - koja je oblika:

```
3
q tekst koji se ispisuje kada se pritisne ctrl+q
d dobro jutro
Q neki treci tekst
```

Na početku datoteke se navodi broj N (1 ≤ N ≤ 16) koji predstavlja
koliko ima mnemonika. Nakon toga sledi N linija gde prvi karakter
predstavlja prečicu, a ostatak teksta je ono što treba ispisati kada se
unese taj karakter uz pritisnuto ctrl. Tekst koji treba ispisati će biti
maksimalne dužine 64 karaktera. Prečica se navodi kao ASCII karakter.
Ako ASCII karaktera nema u scancodes.tbl, onda ne može da se pristupi
mnemoniku.

#### \*.tst

Kada je sistem uspešno konfigurisan, program traži od korisnika da
navede ime neke .tst datoteke koja će sadržati niz ***sc***  vrednosti. Ove
vrednosti se tumače prema prethodnoj konfiguraciji i rezultat tumačenja
se ispisuje na ekran. Vrednost 400 predstavlja kraj datoteke.

Primer datoteke:

```
200 
1 
300 
0 
1 
0 
202 
8 
7 
302 
201 
3 
301 
400
```

Uz konfiguracije navedene u prethodnim delovima, ispis za ovu
datoteku će biti:

```
Baba dobro jutro
```

Tumačenje:

```
200 - shift down 
1 - B 
300 - shift up 
0 - a 
1 - b 
0 - a 
202 - alt down 
8 - 3 
7 - 2 
302 - alt up (32 je razmak, tj. space) 
201 - ctrl down 
3 - ctrl + d je “dobro jutro” 
301 - ctrl up 
400 - kraj
```

Program radi po sledećem toku:

1. Čitanje konfiguracionih datoteka i popunjavanje globalnih nizova /
tabela se izvršava jednom na početku programa.

2. Program pita korisnika za ime datoteke koja sadrži ***sc***  vrednosti.

3. Scan code vrednosti se čitaju jedna po jedna.

4. Parsiranje ***sc***  vrednosti se radi poptuno u assembly. C funkcija
koja radi parsiranje ima potpis: <br>`int process\_scancode(int scancode, char\* buffer)`

	1. Prvi argument predstavlja scancode koji se parsira.

	2. Drugi argument je bafer u koji treba smestiti rezultat parsiranja (tipično samo jedan karakter ćemo smestiti u bafer - više karaktera se smešta samo pri unosu mnemonika).

	3. Povratna vrednost predstavlja broj karaktera koji su upisani u bafer. 
	4. Dozvoljeno je deklarisati jednu int promenljivu koja će biti povratna vrednost za funkciju. Osim toga, sva logika funkcije treba da je napisana u inline assembly bloku.

	4. Dozvoljeno je otvoriti više od jednog inline assembly bloka.

5. Nakon što se isparsira jedna ***sc***  vrednost, dobijeni bafer se
ispisuje na terminalu, i čita se naredna vrednost iz datoteke.

6. Kada se datoteka pročita u celosti, vraćamo se na korak 2.

7. Ako se kao naziv datoteke unese komanda "exit", program se završava.
