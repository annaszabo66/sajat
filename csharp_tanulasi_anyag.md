# 🚗 C# Windows Forms Tanulási Anyag
## Autók Feladat Alapján

---

## Tartalomjegyzék

1. [Osztály létrehozása](#1-osztály-létrehozása)
2. [Konstruktor](#2-konstruktor)
3. [Statikus metódus - Fájl beolvasás](#3-statikus-metódus---fájl-beolvasás)
4. [LINQ alapok](#4-linq-alapok)
5. [Windows Forms alapok](#5-windows-forms-alapok)
6. [Gyakori hibák és megoldások](#6-gyakori-hibák-és-megoldások)
7. [Gyors referencia](#7-gyors-referencia)
8. [Tippek vizsgára](#8-tippek-vizsgára)

---

## 1. OSZTÁLY LÉTREHOZÁSA

### Mi az osztály?
Az osztály egy **sablon**, amely meghatározza, hogy egy objektum milyen **adatokat** (property-k) és **viselkedéseket** (metódusok) tartalmaz.

### Property-k (tulajdonságok)
Ezek tárolják az objektum adatait.

```csharp
public class Auto
{
    public int Sorszam { get; set; }
    public string Marka { get; set; }
    public string Modell { get; set; }
    public int Gyartasi_ev { get; set; }
    public string Szin { get; set; }
    public int Eladott_darabszam { get; set; }
    public double Atlagos_eladasi_ar { get; set; }
}
```

#### Szintaxis magyarázat:
- **`public`** - bárhonnan elérhető
- **Típus** (int, string, double) - milyen adat tárolható
- **Név** - a property neve (nagy betűvel kezdődik!)
- **`{ get; set; }`** - olvasható és írható

#### Gyakori adattípusok:
| Típus | Mit tárol | Példa |
|-------|-----------|-------|
| `int` | Egész szám | 42, -10, 0 |
| `double` | Tizedes tört | 3.14, -0.5 |
| `string` | Szöveg | "Toyota", "Piros" |
| `bool` | Logikai érték | true, false |

---

## 2. KONSTRUKTOR

### Mi a konstruktor?
Speciális metódus, ami **lefut**, amikor új objektumot hozol létre. Inicializálja az objektum adatait.

### Példa:

```csharp
public Auto(string adatok)
{
    // 1. Feldaraboljuk a CSV sort
    string[] lista = adatok.Split(';');
    
    // 2. Átkonvertáljuk és hozzárendeljük az értékeket
    Sorszam = int.Parse(lista[0]);
    Marka = lista[1];
    Modell = lista[2];
    Gyartasi_ev = int.Parse(lista[3]);
    Szin = lista[4];
    Eladott_darabszam = int.Parse(lista[5]);
    Atlagos_eladasi_ar = double.Parse(lista[6]);
}
```

### Mit csinál?
1. **`Split(';')`** - feldarabolja a string-et pontosvessző mentén
2. **`int.Parse()`** - string-et számmá alakítja
3. **Hozzárendeli** az értékeket a property-khez

### Használat:
```csharp
string sor = "1;Toyota;Corolla;2019;Fehér;500;20000;";
Auto auto = new Auto(sor);  // Létrehoz egy Auto objektumot

// Most már használhatod:
Console.WriteLine(auto.Marka);  // Toyota
Console.WriteLine(auto.Eladott_darabszam);  // 500
```

---

## 3. STATIKUS METÓDUS - FÁJL BEOLVASÁS

### Mi a statikus metódus?
Olyan metódus, amit az **osztály nevén** keresztül hívhatsz meg, objektum létrehozása nélkül.

### Példa:

```csharp
public static List<Auto> LoadAuto(string fajl)
{
    var sorok = File.ReadAllLines(fajl)
                    .Skip(1)
                    .Select(sor => new Auto(sor))
                    .ToList();
    return sorok;
}
```

### Lépésről lépésre:
1. **`File.ReadAllLines(fajl)`** - beolvassa az összes sort a fájlból
2. **`.Skip(1)`** - kihagyja az első sort (mert az a fejléc)
3. **`.Select(sor => new Auto(sor))`** - minden sorból csinál egy Auto objektumot
4. **`.ToList()`** - lista formátumba alakítja

### Használat:
```csharp
var autok = Auto.LoadAuto("autok.csv");

// Most 'autok' egy lista, ami Auto objektumokat tartalmaz
Console.WriteLine(autok.Count);  // Hány autó van
Console.WriteLine(autok[0].Marka);  // Első autó márkája
```

### ⚠️ NE FELEJTSD:
```csharp
using System.IO;  // A File osztályhoz kell!
```

---

## 4. LINQ ALAPOK

**LINQ** = Language Integrated Query - adatlekérdezés C#-ban

### 4.1 Count - számolás
Megszámolja a lista elemeit.

```csharp
int db = autok.Count;
Console.WriteLine($"5. feladat: {db} autó található a listában");
```

**Kimenet:**
```
5. feladat: 15 autó található a listában
```

---

### 4.2 Average - átlag
Átlagot számol egy tulajdonság alapján.

```csharp
double atlag = autok.Average(a => a.Eladott_darabszam);
Console.WriteLine($"6. feladat: Átlag: {atlag:F1}");
```

**Magyarázat:**
- `a => a.Eladott_darabszam` - **lambda kifejezés**: "minden 'a' autóra, vedd az Eladott_darabszam-ot"
- `:F1` - **formázás**: 1 tizedesjegy (F2 = 2 tizedesjegy, stb.)

**Kimenet:**
```
6. feladat: Átlag: 625.3
```

---

### 4.3 Where - szűrés
Csak azokat az elemeket adja vissza, amik megfelelnek a feltételnek.

```csharp
var ujAutok = autok.Where(a => a.Gyartasi_ev >= 2017);

foreach (var auto in ujAutok)
{
    Console.WriteLine($"    - {auto.Marka} {auto.Modell}: {auto.Gyartasi_ev}");
}
```

**Feltétel operátorok:**
| Operátor | Jelentés | Példa |
|----------|----------|-------|
| `==` | Egyenlő | `a.Szin == "Piros"` |
| `!=` | Nem egyenlő | `a.Szin != "Fehér"` |
| `>` | Nagyobb | `a.Ar > 20000` |
| `>=` | Nagyobb vagy egyenlő | `a.Ev >= 2020` |
| `<` | Kisebb | `a.Darab < 100` |
| `<=` | Kisebb vagy egyenlő | `a.Ev <= 2015` |

**Kimenet:**
```
    - Toyota Corolla: 2019
    - Volkswagen Golf: 2020
    - Hyundai i30: 2021
```

---

### 4.4 GroupBy - csoportosítás
Csoportokba rendezi az elemeket egy tulajdonság szerint.

```csharp
var markak = autok
    .GroupBy(a => a.Marka)
    .Select(g => new { 
        Marka = g.Key, 
        Osszesen = g.Sum(a => a.Eladott_darabszam) 
    })
    .OrderByDescending(x => x.Osszesen);

foreach (var m in markak)
{
    Console.WriteLine($"    - {m.Marka}: {m.Osszesen} darab");
}
```

**Mit csinál:**
1. **`GroupBy(a => a.Marka)`** - csoportosít márka szerint
2. **`Select(g => new {...})`** - minden csoportból csinál egy **névtelen objektumot**
   - `g.Key` - a csoport kulcsa (márka neve)
   - `g.Sum()` - összegzi a csoport elemeit
3. **`OrderByDescending()`** - csökkenő sorrendbe rendez

**Kimenet:**
```
    - Tesla: 1200 darab
    - Mercedes-Benz: 800 darab
    - Volkswagen: 800 darab
```

---

### 4.5 OrderBy - rendezés

```csharp
// Növekvő sorrend
var novekvo = autok.OrderBy(a => a.Ar);

// Csökkenő sorrend
var csokkeno = autok.OrderByDescending(a => a.Ar);
```

---

### 4.6 Take - első N elem

```csharp
// Első 3 legdrágább autó
var top3 = autok.OrderByDescending(a => a.Ar).Take(3);
```

---

### 4.7 Sum - összegzés

```csharp
int osszes = autok.Sum(a => a.Eladott_darabszam);
Console.WriteLine($"Összesen eladva: {osszes} darab");
```

---

### 4.8 Min / Max - minimum / maximum

```csharp
int legkisebb = autok.Min(a => a.Ar);
int legnagyobb = autok.Max(a => a.Ar);

// Vagy az egész objektum:
var legolcsobb = autok.OrderBy(a => a.Ar).First();
Console.WriteLine($"Legolcsóbb: {legolcsobb.Marka} - {legolcsobb.Ar} Ft");
```

---

## 5. WINDOWS FORMS ALAPOK

### 5.1 Osztályszintű változó

Ha egy változót **több metódusból** is el akarsz érni, tedd **osztályszintűvé**.

```csharp
public partial class Form1 : Form
{
    // IDE! Osztályszintű változó - minden metódus látja
    private List<Auto> autok;
    
    public Form1()
    {
        InitializeComponent();
    }
    
    private void btnBetolt_Click(object sender, EventArgs e)
    {
        // Itt használhatod
        autok = Auto.LoadAuto("autok.csv");
    }
    
    private void txtGyartasiEv_TextChanged(object sender, EventArgs e)
    {
        // Itt is használhatod ugyanazt az 'autok' változót!
        var szurt = autok.Where(a => a.Gyartasi_ev == 2020);
    }
}
```

---

### 5.2 OpenFileDialog - fájl megnyitás

```csharp
private void btnBetolt_Click(object sender, EventArgs e)
{
    OpenFileDialog ofd = new OpenFileDialog();
    ofd.Filter = "CSV fájlok|*.csv|Minden fájl|*.*";
    
    if (ofd.ShowDialog() == DialogResult.OK)
    {
        autok = Auto.LoadAuto(ofd.FileName);
        dgvAutok.DataSource = autok;
    }
}
```

**Magyarázat:**
- **`OpenFileDialog`** - párbeszédablak fájl választáshoz
- **`Filter`** - milyen fájlokat mutasson (formátum: "Leírás|minta")
- **`ShowDialog()`** - megnyitja a dialógust
- **`DialogResult.OK`** - ha a user az OK-t nyomta
- **`ofd.FileName`** - a kiválasztott fájl **teljes elérési útja**

---

### 5.3 DataGridView - táblázat megjelenítés

```csharp
dgvAutok.DataSource = autok;
```

Ez **automatikusan** megjeleníti a listát táblázat formában!

**Opcionális testreszabás:**
```csharp
// Oszlop elrejtése
dgvAutok.Columns["Sorszam"].Visible = false;

// Oszlop fejléc átnevezése
dgvAutok.Columns["Gyartasi_ev"].HeaderText = "Gyártási év";

// Oszlop szélesség
dgvAutok.Columns["Marka"].Width = 150;
```

---

### 5.4 TextBox TextChanged esemény

```csharp
private void txtGyartasiEv_TextChanged(object sender, EventArgs e)
{
    int ev;
    if (int.TryParse(txtGyartasiEv.Text, out ev))
    {
        // Sikerült! Most már van egy 'ev' számunk
        var szurtAutok = autok.Where(a => a.Gyartasi_ev == ev);
        
        lstSzurtAutok.Items.Clear();
        foreach (var auto in szurtAutok)
        {
            lstSzurtAutok.Items.Add($"{auto.Marka} {auto.Modell}");
        }
    }
}
```

---

### 5.5 int.TryParse - biztonságos konverzió

**Miért használjuk?**

❌ **`int.Parse()` - veszélyes:**
```csharp
int ev = int.Parse(txtGyartasiEv.Text); 
// HA nem szám → HIBA, leáll a program! 💥
```

✅ **`int.TryParse()` - biztonságos:**
```csharp
int ev;
if (int.TryParse(txtGyartasiEv.Text, out ev))
{
    // Sikerült! Az 'ev' változóban benne van a szám
}
else
{
    // Nem sikerült, nem szám volt
}
```

**Hogyan működik:**
- **1. paraméter:** mit próbálsz átalakítani (string)
- **2. paraméter (`out ev`):** ide kerül az eredmény, ha sikerült
- **Visszatérési érték:** `true` ha sikerült, `false` ha nem

---

### 5.6 ListBox használata

```csharp
// Elem hozzáadása
lstSzurtAutok.Items.Add("Toyota Corolla");

// Lista ürítése
lstSzurtAutok.Items.Clear();

// Kiválasztott elem lekérése
if (lstSzurtAutok.SelectedItem != null)
{
    string kivalasztott = lstSzurtAutok.SelectedItem.ToString();
}
```

---

### 5.7 MessageBox - üzenet ablak

```csharp
private void btnBezar_Click(object sender, EventArgs e)
{
    DialogResult eredmeny = MessageBox.Show(
        "Biztosan ki szeretne lépni?",    // Üzenet szövege
        "Kilépés",                         // Ablak címe
        MessageBoxButtons.YesNo,           // Gombok
        MessageBoxIcon.Question            // Ikon
    );
    
    if (eredmeny == DialogResult.Yes)
    {
        this.Close();  // Bezárja az alkalmazást
    }
}
```

**MessageBoxButtons opciók:**
- `MessageBoxButtons.OK`
- `MessageBoxButtons.OKCancel`
- `MessageBoxButtons.YesNo`
- `MessageBoxButtons.YesNoCancel`

**MessageBoxIcon opciók:**
- `MessageBoxIcon.Information`
- `MessageBoxIcon.Warning`
- `MessageBoxIcon.Error`
- `MessageBoxIcon.Question`

---

### 5.8 ComboBox - legördülő lista

```csharp
// Elemek hozzáadása
cmbKategoria.Items.Add("Elit");
cmbKategoria.Items.Add("Amatőr");
cmbKategoria.Items.Add("Veterán");

// Kiválasztott elem
string kivalasztott = cmbKategoria.SelectedItem?.ToString();

// SelectedIndexChanged esemény
private void cmbKategoria_SelectedIndexChanged(object sender, EventArgs e)
{
    if (cmbKategoria.SelectedItem != null)
    {
        string kategoria = cmbKategoria.SelectedItem.ToString();
        // Szűrés a kiválasztott kategória alapján
    }
}
```

---

## 6. GYAKORI HIBÁK ÉS MEGOLDÁSOK

### ❌ Probléma #1: Fájl nem található
**Hibaüzenet:** `FileNotFoundException`

**Megoldás:**
1. Jobb klikk a fájlra (pl. `autok.csv`) a Solution Explorer-ben
2. Válaszd a **Properties**-t
3. **Copy to Output Directory** → állítsd **Copy if newer**-re

---

### ❌ Probléma #2: "Index was outside the bounds of the array"
**Hibaüzenet:** `IndexOutOfRangeException`

**OK:** Kevesebb adat van a CSV sorban, mint várnád.

**Megoldás:** 
- Ellenőrizd a CSV formátumot
- Nézd meg, hogy van-e pontosvessző a sor végén
- Számold meg a mezőket

**Példa:**
```
Helyes: 1;Toyota;Corolla;2019;Fehér;500;20000;
Hibás:  1;Toyota;Corolla;2019;Fehér;500        (hiányzik az ár!)
```

---

### ❌ Probléma #3: Változó nem látható más metódusból
**Hibaüzenet:** `The name 'autok' does not exist in the current context`

**OK:** Egy metódusban deklarált változót másik metódusból nem érsz el.

**Megoldás:** Tedd **osztályszintű változóvá**:

```csharp
public partial class Form1 : Form
{
    private List<Auto> autok;  // IDE! (metódusokon kívülre)
    
    private void btnBetolt_Click(...)
    {
        autok = Auto.LoadAuto("autok.csv");  // Most már itt is látszik
    }
    
    private void btnKereses_Click(...)
    {
        var eredmeny = autok.Where(...);  // És itt is!
    }
}
```

---

### ❌ Probléma #4: NullReferenceException
**Hibaüzenet:** `Object reference not set to an instance of an object`

**OK:** Egy változó `null`, de próbálod használni.

**Megoldás:** Ellenőrizd, hogy a változó nem `null`-e:

```csharp
if (autok != null)
{
    var szurt = autok.Where(...);
}
```

---

### ❌ Probléma #5: Format exception Parse-nál
**Hibaüzenet:** `FormatException: Input string was not in a correct format`

**OK:** A string nem szám formátumú, de megpróbáltad számmá alakítani.

**Megoldás:** Használj `TryParse`-t:

```csharp
// Rossz:
int ev = int.Parse(txtEv.Text);  // Ha "abc" van benne → hiba!

// Jó:
int ev;
if (int.TryParse(txtEv.Text, out ev))
{
    // Használd az 'ev' változót
}
```

---

## 7. GYORS REFERENCIA

### 7.1 String műveletek

```csharp
// Feldarabolás
string[] darabok = szoveg.Split(';');

// Összefűzés
string teljes = $"{nev} {kor}";  // String interpoláció
string teljes2 = nev + " " + kor;  // Konkatenáció

// Konverzió
int szam = int.Parse("123");
double tort = double.Parse("12.5");

// Formázás
Console.WriteLine($"{ar:F2}");  // 2 tizedesjegy: 1234.56
Console.WriteLine($"{ar:N0}");  // Ezres elválasztó: 1,234
```

---

### 7.2 Lista műveletek

```csharp
// Létrehozás
List<Auto> lista = new List<Auto>();

// Elem hozzáadása
lista.Add(auto);

// Elemek száma
int db = lista.Count;

// Lista ürítése
lista.Clear();

// Elem elérése index alapján
Auto elsoAuto = lista[0];

// Tartalmaz-e egy elemet
bool van = lista.Contains(auto);
```

---

### 7.3 LINQ láncolás

```csharp
var eredmeny = lista
    .Where(x => x.Ev > 2000)           // Szűrés
    .OrderBy(x => x.Ar)                // Rendezés növekvő
    .OrderByDescending(x => x.Ar)      // Rendezés csökkenő
    .Select(x => x.Nev)                // Transzformáció
    .Take(5)                           // Első 5 elem
    .Skip(2)                           // Első 2 kihagyása
    .ToList();                         // Lista formátumba
```

---

### 7.4 Ciklusok

**foreach - lista bejárása:**
```csharp
foreach (var auto in autok)
{
    Console.WriteLine(auto.Marka);
}
```

**for - számlálós ciklus:**
```csharp
for (int i = 0; i < autok.Count; i++)
{
    Console.WriteLine(autok[i].Marka);
}
```

**while - feltételes ciklus:**
```csharp
int i = 0;
while (i < 10)
{
    Console.WriteLine(i);
    i++;
}
```

---

### 7.5 Feltételes elágazások

**if-else:**
```csharp
if (ev >= 2020)
{
    Console.WriteLine("Új autó");
}
else if (ev >= 2010)
{
    Console.WriteLine("Közepesen új");
}
else
{
    Console.WriteLine("Régi autó");
}
```

**switch:**
```csharp
switch (szin)
{
    case "Piros":
        Console.WriteLine("Piros autó");
        break;
    case "Kék":
        Console.WriteLine("Kék autó");
        break;
    default:
        Console.WriteLine("Más színű");
        break;
}
```

---

## 8. TIPPEK VIZSGÁRA

### ✅ Kódolási gyakorlatok

1. **Használj beszédes változóneveket**
   - ❌ `var a = autok.Where(x => x.b > 2000);`
   - ✅ `var ujAutok = autok.Where(auto => auto.Gyartasi_ev > 2000);`

2. **Kommenteld a bonyolultabb részeket**
   ```csharp
   // Csoportosítás márka szerint és összegzés
   var markak = autok.GroupBy(a => a.Marka)...
   ```

3. **Teszteld minden feladat után!**
   - Ne írd meg az összes feladatot egyben
   - Mindig futtasd le, és nézd meg, működik-e

4. **Figyelj a formázásra**
   - Tizedesjegyek száma (`F1`, `F2`)
   - Kimenet formátuma (szóközök, kötőjelek)

5. **Ne felejtsd el a `using` direktívákat**
   ```csharp
   using System;
   using System.Collections.Generic;
   using System.Linq;
   using System.IO;  // File műveletekhez!
   ```

---

### ✅ Gyakori vizsgafeladat típusok

**1. Adatok beolvasása CSV-ből**
- Konstruktor Split-tel
- Statikus LoadXXX metódus

**2. Számolások**
- Count, Average, Sum, Min, Max

**3. Szűrések**
- Where feltétellel

**4. Csoportosítás**
- GroupBy + Sum/Count

**5. Rendezés**
- OrderBy, OrderByDescending

**6. Top N elem**
- OrderBy().Take(N)

---

### ✅ Idő beosztás vizsgán

**Konzolos rész (kb. 40 perc):**
- 10 perc: Osztály + konstruktor + beolvasás
- 30 perc: Feladatok (5-8)

**Windows Forms rész (kb. 50 perc):**
- 10 perc: Form dizájn
- 20 perc: Betöltés + DataGridView
- 10 perc: Szűrés (TextBox/ComboBox + ListBox)
- 10 perc: Bezárás gomb + MessageBox

---

### ✅ Vizsga előtti ellenőrző lista

- [ ] Tudom, mi az osztály, property, konstruktor
- [ ] Tudom használni a Split és Parse metódusokat
- [ ] Tudom, hogyan kell fájlt beolvasni
- [ ] Ismerem a LINQ alapokat (Where, GroupBy, OrderBy, Average, Sum)
- [ ] Tudom, mi az osztályszintű változó
- [ ] Tudom használni az OpenFileDialog-ot
- [ ] Tudom, hogyan kell DataGridView-t feltölteni
- [ ] Ismerem a TryParse használatát
- [ ] Tudom, hogyan kell MessageBox-ot megjeleníteni
- [ ] Ellenőriztem, hogy a fájl másolódik-e a kimeneti mappába

---

## ÖSSZEFOGLALÁS

**Mit tanultál:**

1. ✅ **OOP alapok:** Osztály, property, konstruktor
2. ✅ **Fájlkezelés:** CSV beolvasás, Split, Parse
3. ✅ **LINQ:** Where, GroupBy, OrderBy, Average, Sum
4. ✅ **Windows Forms:** OpenFileDialog, DataGridView, ListBox, MessageBox
5. ✅ **Eseménykezelés:** Click, TextChanged
6. ✅ **Hibaelhárítás:** TryParse, null ellenőrzés

---

## Példa feladat végső kód

### Auto.cs - teljes kód

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;

public class Auto
{
    public int Sorszam { get; set; }
    public string Marka { get; set; }
    public string Modell { get; set; }
    public int Gyartasi_ev { get; set; }
    public string Szin { get; set; }
    public int Eladott_darabszam { get; set; }
    public double Atlagos_eladasi_ar { get; set; }

    public Auto(string adatok)
    {
        string[] lista = adatok.Split(';');
        Sorszam = int.Parse(lista[0]);
        Marka = lista[1];
        Modell = lista[2];
        Gyartasi_ev = int.Parse(lista[3]);
        Szin = lista[4];
        Eladott_darabszam = int.Parse(lista[5]);
        Atlagos_eladasi_ar = double.Parse(lista[6]);
    }

    public static List<Auto> LoadAuto(string fajl)
    {
        var sorok = File.ReadAllLines(fajl)
                        .Skip(1)
                        .Select(sor => new Auto(sor))
                        .ToList();
        return sorok;
    }
}
```

### Program.cs - konzolos feladatok

```csharp
using System;
using System.Linq;

class Program
{
    static void Main(string[] args)
    {
        // Beolvasás
        var autok = Auto.LoadAuto("autok.csv");

        // 5. feladat
        Console.WriteLine($"5. feladat: {autok.Count} autó található a listában");

        // 6. feladat
        var atlag = autok.Average(a => a.Eladott_darabszam);
        Console.WriteLine($"6. feladat: Az autók esetében az átlagosan eladott darabszám {atlag:F1}");

        // 7. feladat
        Console.WriteLine("7. feladat: Az elmúlt 5 évben gyártott autók:");
        var ujAutok = autok.Where(a => a.Gyartasi_ev >= 2017);
        foreach (var auto in ujAutok)
        {
            Console.WriteLine($"    - {auto.Marka} {auto.Modell}: {auto.Gyartasi_ev}");
        }

        // 8. feladat
        Console.WriteLine("8. feladat: Legsikeresebb márkák listája az eladott darabszám alapján:");
        var markak = autok
            .GroupBy(a => a.Marka)
            .Select(g => new { Marka = g.Key, Osszesen = g.Sum(a => a.Eladott_darabszam) })
            .OrderByDescending(x => x.Osszesen);

        foreach (var m in markak)
        {
            Console.WriteLine($"    - {m.Marka}: {m.Osszesen} darab");
        }

        Console.ReadKey();
    }
}
```

### Form1.cs - Windows Forms

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Windows.Forms;

public partial class Form1 : Form
{
    private List<Auto> autok;

    public Form1()
    {
        InitializeComponent();
    }

    private void btnBetolt_Click(object sender, EventArgs e)
    {
        OpenFileDialog ofd = new OpenFileDialog();
        ofd.Filter = "CSV fájlok|*.csv";
        
        if (ofd.ShowDialog() == DialogResult.OK)
        {
            autok = Auto.LoadAuto(ofd.FileName);
            if (autok != null)
            {
                dgvAutok.DataSource = autok;
            }
        }
    }

    private void txtGyartasiEv_TextChanged(object sender, EventArgs e)
    {
        if (autok == null) return;
        
        int ev;
        if (int.TryParse(txtGyartasiEv.Text, out ev))
        {
            var szurtAutok = autok.Where(a => a.Gyartasi_ev == ev);
            
            lstSzurtAutok.Items.Clear();
            foreach (var auto in szurtAutok)
            {
                lstSzurtAutok.Items.Add($"{auto.Marka} {auto.Modell}");
            }
        }
    }

    private void btnBezar_Click(object sender, EventArgs e)
    {
        DialogResult eredmeny = MessageBox.Show(
            "Biztosan ki szeretne lépni?",
            "Kilépés",
            MessageBoxButtons.YesNo,
            MessageBoxIcon.Question
        );
        
        if (eredmeny == DialogResult.Yes)
        {
            this.Close();
        }
    }
}
```

---

**Sok sikert a vizsgához! 🎓🍀**

Ha bármit nem értesz, nézd át újra ezt az anyagot, és próbáld ki a kódokat gyakorlásképpen!
