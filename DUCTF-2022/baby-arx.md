# baby arx

Utmaningen består av ett egenskapat så kallat "add-rotate-xor"-krypto, där man har fått koden och den krypterade flaggan

Kod:
```python
class baby_arx():
    def __init__(self, key):
        assert len(key) == 64
        self.state = list(key)

    def b(self):
        b1 = self.state[0]
        b2 = self.state[1]
        b1 = (b1 ^ ((b1 << 1) | (b1 & 1))) & 0xff
        b2 = (b2 ^ ((b2 >> 5) | (b2 << 3))) & 0xff
        b = (b1 + b2) % 256
        self.state = self.state[1:] + [b]
        return b

    def stream(self, n):
        return bytes([self.b() for _ in range(n)])


FLAG = open('./flag.txt', 'rb').read().strip()
cipher = baby_arx(FLAG)
out = cipher.stream(64).hex()
print(out)

# cb57ba706aae5f275d6d8941b7c7706fe261b7c74d3384390b691c3d982941ac4931c6a4394a1a7b7a336bc3662fd0edab3ff8b31b96d112a026f93fff07e61b
```
Det viktiga är här funktionen `b()`, som tar de första två tecknen i flaggan, flyttar om alla bits på olika sätt, gör en xor med originalet, och slutligen adderar de två och ser till att det bara är en byte. Därefter roteras hela strängen, den nyss krypterade byten flyttas till slutet av strängen, och därefter upprepas samma procedur så många gånger som man anger till `stream()`-funktionen, i det här fallet samma antal gånger som antalet tecken i flaggan (`assert len(key) == 64` innebär att programmet kommer krascha om strängen är någon annan längd)

I vissa av krypteringsstegen, framförallt vid `b = (b1 + b2) % 256` riskerar en del information att förloras (några bits kastas bort helt enkelt). Detta gör att det i praktiken blir omöjligt att vända tillbaka resultatet.

Istället får vi använda en annan teknik. Vi vet att en komination av två tecken blir till en byte. Då kan vi göra en rainbow-table genom att kryptera alla möjliga kombinationer av två tecken. Om vi förutsätter att flaggan enbart innehåller skrivbara ASCII-tecken finns det totalt 94 * 94 = 8836 olika kombinationer. Eftersom antalet möjliga bytes däremot är begränsat till 256 kommer därför vissa kombinationer ge samma byte. Men om vi kan ta reda på de första två tecknena kommer vi därmed ha det första tecknet i kombinationen `FLAG[1:2]`, vilket gör att vi inte behöver söka igenom alla kombinationer. Vi får därför anpassa var rainbow-table efter detta, och kan skapa den med följande kod (observera att vi då måste kommentera bort `assert len(key) == 64` i `baby_arx`-klassen):
```python
import string

def rainbow_table():
    alphabet = bytes(string.printable, 'ascii')

    table = {}
    # Gå igenom alla tecken...
    for letter1 in alphabet:
        combinations = {}
        # ...kombinerat med alla tecken en gång till
        for letter2 in alphabet:
            comb = bytes([letter1, letter2])
            arx = baby_arx(comb)
            # Kryptera kombinationen och sätt som nyckel
            # och kombinationen som värde
            combinations[arx.b()] = comb
        # Sätt första bokstaven i kombinationen som nyckel,
        # och alla kombinationer med den som värde
        table[letter1] = combinations
    return table
```
Därefter kan vi börja söka igenom. Första byten i den krypterade flaggan är `0xcb`, så vi börjar att söka efter den:
```python
table = rainbow_table()

first = 0xcb
for comb in table.values():
    if first in comb:
        print(comb[first])
```
```
PS D:\ctf\ductf> python .\baby-arx.py
b'0*'
b'4E'
b'6b'
b'9['
b'cM'
b'en'
b'f '
b'iK'
b'kl'
b'l&'
b'qj'
b'rO'
b'w"'
b'y\t'
b'B$'
b'DU'
b'GI'
b'Hp'
b'JW'
b'N8'
b'PQ'
b'St'
b'T>'
b'Yr'
b'Z<'
b'!('
b'"\r'
b'$.'
b"'`"
b'(\x0b'
b'*,'
b'-f'
b'.C'
b':G'
b'<6'
b'_:'
b'`h'
b'|d'
b'\t0'
b'\x0c]'
```
Som väntat hittar vi flera kombinationer, men en är särskilt intressant: `DU`, eftersom flaggorna i tävlingen som regel börjar med `DUCTF`. Vi gör därför antagandet att första tecknet i flaggan är `D`, och använder det för att söka igenom resten:
```python
table = rainbow_table()

encrypted = 'cb57ba706aae5f275d6d8941b7c7706fe261b7c74d3384390b691c3d982941ac4931c6a4394a1a7b7a336bc3662fd0edab3ff8b31b96d112a026f93fff07e61b'
flag = bytes('D', 'ascii')
for b in bytes.fromhex(encrypted):
    try:
        # Använd sista hittade tecknet för att
        # begränsa sökningen, och lägg till
        # andra tecknet i den hittade kombinationen
        next_letter = table[flag[-1]][b][1]
        flag += bytes([next_letter])
    except KeyError:
        break

print(flag.decode('ascii'))
```
Så får vi slutligen ut flaggan (sista tecknet kan vi bortse ifrån):
```
PS D:\ctf\ductf> python .\baby-arx.py
DUCTF{i_d0nt_th1nk_th4ts_h0w_1t_w0rks_actu4lly_92f45fb961ecf420}4
```