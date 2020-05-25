# Teoria kompilacji i kompilatory
# *Translator plików w formacie CSV na tabele HTML*

# Spis treści
- [Opis projektu](#opis-projektu)
  + [Narzędzia](#narzędzia)
  + [Literatura](#literatura)
  + [Instrukcja instalacji ANTLR dla Windows](#instrukcja-instalacji-antlr-dla-windows)
- [Podział prac i stan projektu](#podział-prac-i-stan-projektu)
- [Realizacja projektu](#realizacja-projektu)
   * [Część 1 gramatyka](#część-1-gramatyka)
     + [Rozpatrywany format danych](#rozpatrywany-format-danych)
     + [Gramatyka dla plików w formacie CSV](#gramatyka-dla-plików-w-formacie-csv)
   * [Część 2 lexer i parser](#część-2-lexer-i-parser)
     + [Generacja lexera i parsera](#generacja-lexera-i-parsera)
     + [Testy działania lexera](#testy-działania-lexera)
     + [Testy działania parsera](#testy-działania-parsera)
   * [Część 3 konwersja csv do html](#część-3-konwersja-csv-do-html)
     + [Docelowy format tabeli HTML](#docelowy-format-tabeli-html)
     + [Opis programu](#opis-programu)
     + [Testowe wykonanie programu](#testowe-wykonanie-programu)
     + [How to run](#how-to-run)

# Opis projektu
Celem projektu jest projekt i implementacja translatora plików w formacie CSV na tabele HTML. Translator został wykonany zgodnie z architekturą współczesnych interpreterów i translatorów - składa się z lexera i parsera. W wyniku działania lexera i parsera otrzymywane jest drzewo syntaktyczne. Przechodzenie po drzewie pozwala na wykonywanie działań zgodnie z regułami translacji. 

## Narzędzia 

ANTLR (ANother Tool for Language Recognition) - to generator analizatora składni, który używa algorytmu LL (*)
do parsowania języków. https://www.antlr.org/

## Literatura
+ The Definitive ANTLR 4 Reference by Terence Parr

https://pragprog.com/book/tpantlr2/the-definitive-antlr-4-reference

## Instrukcja instalacji ANTLR dla Windows
 1. Pobieramy plik .jar ze strony https://www.antlr.org/download/ - aktualnie najnowsza wersja to antlr-4.8-complete.jar
 2. Pobrany plik umieszczamy w dowolnym katalogu
 3. Klasy z pobranego pliku należy dodać do zmiennej CLASSPATH, tymczasowe dodanie do zmiennej CLASSPATH można zrealizować z linii komend:
 
 ```console
 SET CLASSPATH=.;C:\Users\Dominik\Desktop\Kompilatory\antlr\antlr-4.8-complete.jar;%CLASSPATH%
 ```
 
 4. Sprawdzamy poprawność konfiguracji z linii komend: 
 
 ```console
 java org.antlr.v4.Tool
 ```
 
 5. Wpisywanie takiej komendy za każdym razem byłoby dość męczące więc możemy utworzyć aliasy, na windows robimy to przy pomocy doskey:
 
```console
doskey antlr4=java org.antlr.v4.Tool $*
doskey grun =java org.antlr.v4.gui.TestRig $*
```
TestRig to klasa do testowania wyników wygenerowanych przez klasę Tool, będzie również często używana. 

6. Teraz możemy używać narzędzia przez komendę:

```console
antlr4
```
# Podział prac i stan projektu
Podział prac:
+ Klaudia Klepacka - zbieranie materiałów źródłowych, prowadzenie dokumentacji, projektowanie formatów danych, projektowanie gramatyki
+ Marcin Michna - instalacja i konfiguracja narzędzi projektowych, praca nad algorytmem konwersji, generacja i ocena działania klas, testowanie działania lexera
+ Dominik Wróbel - praca nad algorytmem konwersji, pisanie programu, zarządzanie projektem, integracja klas projektowych z algorytmem konwersji
+ Jakub Zyngier - projekowanie gramatyki, generacja klas, testowanie działania parsera, prowadzenia dokumentacji, generacja i ocena działania klas

Stan projektu:
Projekt został zrealizowany w pełni i spełnia wszystkie postawione na początku wymagania. Możliwym rozwinięciem projektu jest zapis wynikowej tabeli do pliku HTML (aktualnie jest ona wypisywana na standardowe wyjście).

# Realizacja projektu

## Część 1 gramatyka
Pierwszym krokiem jest określenie postaci języka wejściowego dla narzędzia ANTLR. W projekcie definiujemy język dla plików CSV przy użyciu gramatyki w formacie określanym przez ANTLR. Gramatyka jest zapisana w pliku z rozszerzeniem .g4.

### Rozpatrywany format danych
CSV to format przechowywanie danych w plikach tekstowych. Istnieje wiele implementacji standardu CSV. W naszym projekcie zakładamy, że:
+ Rekordy oddzielane są znakiem nowej linii (również ostatni wiersz musi zawierać znak nowej linii)
+ Pola oddzielane są przecinkiem
+ Pole może zawierać dowolny ciąg znaków lub być puste
+ Pierwszy wiersz pliku to nagłówek zawierający opisy kolumn

Przykład danych zgodnych z powyższym opisem (plik usernames.csv): 

*Username,Identifier,First name,Last name*

*booker12,9012,Rachel,Booker*

*grey07,2070,Laura,Grey*

*johnson81,4081,Craig,Johnson*

*jenkins46,9346,Mary,Jenkins*

*smith79,5079,Jamie,Smith*




### Gramatyka dla plików w formacie CSV 
(Zawartość pliku CSV_Grammar.g4):

```console
grammar CSV_Grammar;

csv_file: header_row row+;			 // plik csv to naglowek i przynajmniej jeden wiersz
	
header_row: cell_header (',' cell_header)* '\n'; // naglowek to jedna lub wiecej komorek naglowka oddzielone przecinkami az do nowej linii

row: cell (',' cell)* '\n';	                 // wiersz to jedna lub wiecej komorek oddzielone przecinkami az do nowej linii

cell_header: CHARS | ;	                         // komorka naglowka to ciag znakow, ale moze byc tez pusta

cell: CHARS | ;		                         // komorka to ciag znakow, ale moze byc tez pusta

CHARS: ~[,\n]+ ; 	                         // cokolwiek co nie jest przecinkiem i nowa linia 
```
## Część 2 lexer i parser
Kolejnym krokiem jest wygenerowanie lexera oraz parsera na podstawie pliku z gramatyką.

### Generacja lexera i parsera
Możemy to zrobić z linii komend:
```console
antlr4 CSV_Grammar.g4
```
W efekcie generowane jest kilka plików, najważniejsze to:
+ CSV_GrammarLexer.java - zawiera implementację lexera dla gramatyki, plik ten zawiera reguły, które przekształcają słowa wejściowe na tokeny, tokeny zapisane są w gramatyce jako reguły rozpoczynające się dużymi literami lub jako symbole w apostrofach
+ CSV_GrammarParser.java - zawiera implementację parsera dla gramatyki, plik ten zawiera reguły, które przekształcają tokeny wejściwe w drzewo syntaktyczne zgodnie z produkcjami zapisanymi w gramatyce (produkcje to reguły zaczynające się od małych liter) 
+  CSV_GrammarListener, CSVGrammarBaseListener - to interfejs i klasa, które umożliwiają przechodzenie po drzewie syntaktycznym poprzez mechanizm listenerów (analogia do reagowania na akcje użytkownika w GUI)

### Testy działania lexera

Aby przetestować działanie lexera można użyć narzędzia dostarczanego przez ANTLR o nazwie TestRig. <br/>
Przyjmuje ono:
* nazwę gramatyki - w naszym przypadku CSV_Grammar
* regułę początkową - csv_file
* opcję wyniku - w nazszym przypadku '-tokens', ponieważ chcemy zobaczyć wygenerowane tokeny
* ścieżka do pliku testowego - usernames.csv

Końcowa forma polecenia:
```console
java org.antlr.v4.gui.TestRig CSV_Grammar csv_file -tokens usernames.csv
```

Wygenerowane zostają następujące tokeny (zakończone tokenem symbolizującym koniec pliku):
![tokens](https://user-images.githubusercontent.com/33720728/80991057-bb9a0480-8e37-11ea-8324-75c4c1f1165d.png)

Przykładowy token:
![example](https://user-images.githubusercontent.com/33720728/80997491-b04bd680-8e41-11ea-961e-97af34f22a67.png)

1. numer tokenu - indeksacja od 0
2. numer początkowego i końcowego znaku z którego składa się token zaczynając od początku pliku
3. text z jakiego się składa
4. typ tokenu - reguła z gramatyki zaczynająca się z dużej litery lub w symbol w apostrofach <br>
	W naszej gramrtyce znajdują się trzy typy tokenów:
	* CHARS - ciąg znaków
	* ',' - następna kolumna
	* '\n' - nowa linia
5. pozycja tokenu - numer wiersza i numer znaku w tym wierszu (licząc od zera, tabulator jako jeden znak)

W naszym testowanym przypadku plik csv został prawidłowo podzielony na tokeny. Pierwsze 7 z nich reprezentuje nazwy pól rekordów oddzielone przecinkami, a po nich znak końca linii. Analogicznie każde kolejne 7 tokenów to kolejny rekord w pliku csv. Na końcu znajduje się token końca pliku. 



### Testy działania parsera
 
W celu przetestowania działania parsera zostało wygenerowane drzewo syntaktyczne przy użyciu narzędzia antlr4. Na początku należy wygenerować klasy dla gramatyki przy użyciu narzędzia antlr4 z poziomu linii komend:
```console
antlr4 CSV_Grammar.g4
```
Następnie trzeba skompilować wygenerowane pliki poleceniem:
```console
javac *.java
```
Na koniec wystarczy wygenerować drzewo poleceniem:
```console
java org.antlr.v4.gui.TestRig CSV_Grammar csv_file -tree usernames.csv -gui
```
W powyzszym poleceniu flaga -gui jest opcjonalna lecz przydatna, ponieważ wizualizuje nam drzewo. Bez tej flagi drzewo generowane jest w formie tekstu.

Wygenerowane drzewo syntaktyczne dla gramatyki dla przykładowych danych:

![Drzewo syntaktyczne dla gramatyki](https://i.ibb.co/XXtVx2x/syntactic-tree.png)

Fragment drzewa syntaktycznego dla gramatyki (nagłówek):

![Fragment drzewa syntaktycznego dla gramatyki](https://i.imgur.com/qZUtRnN.png)

Fragment drzewa syntaktycznego dla gramatyki (przykładowy wiersz inny niż nagłówek):

![Fragment drzewa syntaktycznego dla gramatyki](https://i.imgur.com/kOKa5HY.png)

**Analiza drzewa syntaktycznego**

Będziemy korzystać z metody generacyjnej rozbioru gramatycznego. Poniżej przedstawiony jest rozbiór gramatyczny. Pogrubiony sybol, to ten który będziemy rozkładać przez kolejne podstawienia. Poniżej pokazany jest robiór do momentu, aż otrzymamy nagłówek. Polega to na tym, że startujemy z sybolu **csv_file** gdzie przechodzimy do **(header_row) (row+)** (nawiasy zostały użyte jedynie w celu lepszej wizualizacji tego). Co oznacza, że nasz plik składa się z nagłówka i przynajmniej jednego wiersza z danymi. Później przez kolejne przekształcenia rozbijamy **header_row** na **cell_header (',' cell_header)\* '\n'** co oznacza, że nagłówek to jedna komórka lub więcej rozdzielonych przecinkami i kończy się znakiem nowej linii. Następnie **cell_header** przechodzi w **CHARS**, a on może składać się ze znaków jakichkolwiek co nie jest przecinkiem albo znakiem nowej linni, co mozemy zapisać w postaci **~[,\n]+**.


**csv_file** => <br />
**(header_row)** (row+) => <br />
**(cell_header)** ((',' cell_header)* '\n') (row+) => <br />
**(CHARS)** ((',' cell_header)* '\n') (row+) => <br />
**(~[,\n]+)** ((',' cell_header)* '\n') (row+) => <br />
Username **((',' cell_header)* '\n')** (row+) => <br />
Username, **cell_header**, cell_header, cell_header, \n (row+) => <br /> 
Username, **CHARS**, cell_header, cell_header, \n (row+) => <br />
Username, **(~[,\n]+)**, cell_header, cell_header, \n (row+) => <br />
Username, Identifier, **cell_header**, cell_header, \n (row+) => <br />
...dalej rozkładamy każdy kolejny cell_header... => <br />
Username, Identifier, First name, Last name \n (row+) <br />

W powyższym przykładzie dochodzimy do momentu, gdzie mamy już nagłówek i zostaje Nam **row+**, co oznacza, że będziemy zajmować się kolejnymi wierszami, których może być jeden lub więcej. Działamy w bardzo analogiczny sposób jak dla nagłówka.

**row+** => <br />
**row** row row row row row => <br />
(**cell** (',' cell)* '\n') row row row row row => <br />
(**CHARS** (',' cell)* '\n') row row row row row => <br />
(**~[,\n]+** (',' cell)* '\n') row row row row row => <br />
(booker12, **(',' cell)* '\n'**) row row row row row => <br />
(booker12, **cell** cell cell) row row row row row => <br />
(booker12, **CHARS** cell cell) row row row row row => <br />
(booker12, **~[,\n]+** cell cell) row row row row row => <br />
(booker12, 9012 **cell** cell) row row row row row => <br />
...robimy podstawienia w Naszej gramatyce za kolejne symbole... => <br />
booker12, 9012, Rachel, Booker \n **row** row row row row => <br />
...później zajmujemy się podstawianiem za kolejny rekord, bo w podstawianiu zaczynamy od najardziej wysuniętego sybolu na lewo... <br />


**Poniżej przedstawiony jest gif prezentujący tworzenie takiego drzewa dla nagłówka i pierwszego wiersza danych**

![Tworzenie drzewa syntaktycznego](https://s7.gifyu.com/images/ezgif-4-14e24ad17d70.gif)


## Część 3 konwersja csv do html

### Docelowy format tabeli HTML

W docelowej tabeli HTML wszystkie rzędy będą definiowane za pomocą znaczników \<tr\>. W pierwszym z nich zagnieżdżone zostaną pogrubione i wyśrodkowane nagłówki kolumn przy użyciu \<th\>, natomiast w pozostałych wierszach będą znajdować się dane zdefiniowane przez znaczniki \<td\>.

### Opis programu

W programie używane są klasy, które zostały wygnerowane przez narzędzie antlr4 z poziomu linii komend:
```console
antlr4 CSV_Grammar.g4
```
Wykonanie tego polecenia powoduje generacje lexera, parsera oraz klas i interfejsów pozwalających na przechodzenie po drzewie syntaktycznym. 

+ Określenie danych wejściowych
```java
        CharStream codePointCharStream = null; 

        try {
            codePointCharStream = CharStreams.fromFileName("usernames.csv"); // plik csv z danymi 
        } catch (IOException e) {
            System.out.println("File char stream not found.");
            e.printStackTrace();
        }
```
+ Utworzenie lexera i parsera
```java
	CSV_GrammarLexer csv_grammarLexer = new CSV_GrammarLexer(codePointCharStream);

        CommonTokenStream commonTokenStream = new CommonTokenStream(csv_grammarLexer);

        CSV_GrammarParser csv_grammarParser = new CSV_GrammarParser(commonTokenStream);
```

+ Utworzenie drzewa syntaktycznego
```java
	ParseTree parseTree = csv_grammarParser.csv_file();
```

+ Przechodzenie po wygenerowanym drzewie
```java	
	ParseTreeWalker parseTreeWalker = new ParseTreeWalker();
	parseTreeWalker.walk(new CsvToHtmlTable(), parseTree);
```

Przechodzenie po drzewie syntaktycznym jest realizowane przez klasę ParseTreeWalker. Jest to klasa z biblioteki antlr, jako parametr przyjmuje ona obiekt klasy CsvToHtmlTable. CsvToHtmlTable to utworzona w projekcie klasa, która dziedziczy po wygenerowanej przez antlr4 klasie CSV_GrammarBaseListener i nadpisuje niektóre z jej metod. 

Sposób przechodzenia po drzewie opiera się o mechanizm 'listenerów'. Listenery reagują na wydarzenia związane z przechodzeniem po drzewie syntaktycznym. Podstawowe wydarzenia to wejście do danego typu węzła i wyjście z daneg typu węzła.

Przykłady zaimplementowanych listenerów:
```java
@Override
    public void enterCsv_file(CSV_GrammarParser.Csv_fileContext ctx){
        System.out.println("<table>");
    }
```

```java
    @Override
    public void enterCell(CSV_GrammarParser.CellContext ctx) {
        System.out.printf("<td>");
        if(ctx.CHARS().getText() != null){
            System.out.printf(ctx.CHARS().getText());
        } else{
            System.out.printf("");
        }
    }
```

### Testowe wykonanie programu
Dla pliku wejściowego usernames.csv generowany jest fragment HTML zamieszczony poniżej:
```html
<table>
<tr>
<th>Username</th>
<th>Identifier</th>
<th>First name</th>
<th>Last name</th>
</tr>
<tr>
<td>booker12</td>
<td>9012</td>
<td>Rachel</td>
<td>Booker</td>
</tr>
<tr>
<td>grey07</td>
<td>2070</td>
<td>Laura</td>
<td>Grey</td>
</tr>
<tr>
<td>johnson81</td>
<td>4081</td>
<td>Craig</td>
<td>Johnson</td>
</tr>
<tr>
<td>jenkins46</td>
<td>9346</td>
<td>Mary</td>
<td>Jenkins</td>
</tr>
<tr>
<td>smith79</td>
<td>5079</td>
<td>Jamie</td>
<td>Smith</td>
</tr>
</table>
```
Tabela wynikowa składa się z czterech kolumn:
+ Username - nazwa użytkownika (String, który może zawierać litery oraz cyfry)
+ Identifier - unikalne ID (Integer składający się z czterech znaków)
+ First Name - imię (String)
+ Second Name - nazwisko (String)

![res](https://user-images.githubusercontent.com/39568472/78917202-dc687780-7a8e-11ea-8567-4adf329741ac.PNG)

### How to run

Aby uruchomić program należy dodać do projektu .jar z biblioteką antlr. 

W IntelliJ IDEA: File -> Project Structure -> Libraries -> Add -> Path to .jar

Następnie można uruchomić metodę main z klasy Main. Na standardowe wyjście wypisywana jest tabela HTML utworzona na podstawie danych z pliku usernames.csv.

