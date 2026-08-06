# Uczenie maszynowe od zera w Excelu

Pięć algorytmów uczenia maszynowego policzonych **wyłącznie na formułach arkusza** - bez Pythona, bez bibliotek, bez `model.fit()`. Propagacja przez warstwy sieci, normalizacja, liczenie odległości euklidesowych, dopasowanie metodą najmniejszych kwadratów - każdy krok jest widoczny w komórce i można go prześledzić.

To projekty z okresu studiów. Wrzucam je publicznie, bo pokazują coś, czego nie widać w notatniku wywołującym `sklearn`: **że rozumiem, co te algorytmy liczą w środku.**

*[English version below](#machine-learning-from-scratch-in-excel)*

## Wyniki

| # | Projekt | Dane | Metoda | Wynik |
|---|---|---|---|---|
| [01](01-siec-1-warstwa) | Sieć neuronowa, 1 warstwa ukryta | 683 rekordy, 9 cech, 2 klasy | 3 neurony, sigmoida, normalizacja min-max | **97,3% na zbiorze testowym** (5 błędów / 183) |
| [02](02-lda-iris) | Liniowa analiza dyskryminacyjna | Iris - 150 rekordów, 4 cechy, 3 klasy | LDA Fishera | 98,7% (2 błędy / 150) |
| [03](03-k-srednich) | Grupowanie k-średnich | 50 obiektów, 4 cechy | k = 4, iteracje Lloyda | zbieżność w 5. iteracji |
| [04](04-siec-2-warstwy) | Sieć neuronowa, 2 warstwy ukryte | 172 rekordy, 4 cechy, 3 klasy | 3 + 3 neurony, sigmoida | 72,1% (48 błędów / 172) |
| [05](05-regresja-liniowa) | Regresja liniowa | 8 punktów, 1 zmienna | MNK z formuł + diagnostyka | R² = 0,758 |

### Jak czytać te wyniki

**Tylko projekt 01 ma prawdziwy podział na zbiór uczący i testowy.** Model dopasowano na 500 rekordach, a 97,3% to skuteczność na 183 rekordach, których optymalizacja nigdy nie widziała.

Projekty 02, 04 i 05 raportują dopasowanie **na tych samych danych, na których je uczono**. Te liczby są więc optymistyczne i nie mówią nic o zdolności do generalizacji. Zostawiam je takimi, jakie były - z zaznaczeniem, że dziś rozdzieliłbym dane przed dopasowaniem w każdym z nich, tak jak zrobiłem to w projekcie 01.

---

## 01 · Sieć neuronowa z jedną warstwą ukrytą

**Dane:** 683 rekordy, 9 cech w skali 1-10, dwie klasy (2 i 4). Charakterystyka odpowiada zbiorowi *Breast Cancer Wisconsin (Original)* z repozytorium UCI.

**Architektura:** 9 wejść → 3 neurony ukryte → 1 wyjście. Suma ważona liczona przez `SUMPRODUCT` z `INDEX`, normalizacja min-max, aktywacja sigmoidalna `1/(1+EXP(-x))`, warstwa wyjściowa przez `MMULT`. Próg decyzyjny wyznaczony jako średnia ważona średnich wyjścia w obu klasach.

**Ewaluacja:** wiersze 11-510 to zbiór uczący (suma kwadratów błędów = 199,85, minimalizowana przez Solver). Wiersze 511-693 to **zbiór testowy, nieużywany podczas dopasowania** - 5 błędnych klasyfikacji na 183 rekordy.

**To najmocniejszy projekt w zestawie** - jako jedyny mierzy to, co naprawdę ma znaczenie.

![Przepływ obliczeń w sieci: suma ważona, normalizacja, sigmoida, wyjście, błąd, predykcja](obrazy/siec-1-warstwa-propagacja.png)

*Pełna ścieżka obliczeń dla kilku rekordów: ważona suma trzech neuronów → normalizacja min-max → aktywacja sigmoidalna → warstwa wyjściowa → błąd kwadratowy → predykcja klasy → trafienie (0 = poprawnie).*

![Wagi warstwy wyjściowej oraz podsumowanie: suma błędów, próg odcięcia i liczba pomyłek](obrazy/siec-1-warstwa-wagi-i-wynik.png)

*Wagi warstwy wyjściowej i podsumowanie. `Suma błędów 199,853` dotyczy zbioru uczącego (500 rekordów), a `pomyłki 5` - wstrzymanego zbioru testowego (183 rekordy).*

> ⚠️ Liczby `317` i `183` w wierszu „liczba" to **liczebność klas 2 i 4 wewnątrz zbioru uczącego** (317 + 183 = 500), a nie podział na zbiór uczący i testowy. To, że zbiór testowy również liczy 183 rekordy, jest przypadkiem.

## 02 · Liniowa analiza dyskryminacyjna (Iris)

**Dane:** klasyczny zbiór Iris - 150 rekordów, 4 cechy, 3 gatunki po 50 rekordów.

**Metoda:** dyskryminanta Fishera. Rzutowanie 4 cech na jedną oś, wagi dobrane tak, by maksymalizować stosunek wariancji międzygrupowej do wewnątrzgrupowej. Klasyfikacja przez dwa progi odcięcia, wyznaczone jako średnie ważone środków sąsiadujących klas.

**Wynik:** 2 błędy na 150 rekordów. Stosunek wariancji między/wewnątrz = **0,645** - arkusz liczy obie wariancje osobno, więc widać, skąd ta liczba się bierze.

**Ograniczenie:** ocena na danych uczących, bez walidacji krzyżowej.

![Wagi dyskryminanty, progi odcięcia i rozbicie wariancji na międzygrupową i wewnątrzgrupową](obrazy/lda-iris-metryki.png)

*Wagi dyskryminanty, średnie rzutowania w każdej klasie, progi odcięcia oraz rozbicie wariancji: międzygrupowa 0,522, wewnątrzgrupowa 0,808, stosunek **0,645**. `Różnica = 2` to liczba błędnych klasyfikacji na 150 rekordów.*

## 03 · Grupowanie metodą k-średnich

**Dane:** 50 obiektów, 4 cechy dotyczące przestępczości i urbanizacji. Charakterystyka odpowiada zbiorowi *USArrests* (stany USA); etykiety zanonimizowano do `L1`-`L50`.

**Metoda:** algorytm Lloyda dla k = 4, wykonany ręcznie. **Każda iteracja to osobny arkusz** (`k1`…`k6`) - widać w nich pełny cykl: liczenie odległości euklidesowych do każdego centroidu (`SQRT`), przypisanie do najbliższego (`INDEX`+`MATCH`+`MIN`), przeliczenie centroidów (`AVERAGEIFS`).

**Zbieżność:** kolumna „różnica" porównuje przypisanie z poprzednią iteracją. Liczba zmian: **4 → 4 → 2 → 0 → 0**. Algorytm ustabilizował się w piątej iteracji, szósta to potwierdzenie.

Ten projekt najlepiej pokazuje mechanikę algorytmu, bo nic nie dzieje się „w środku funkcji" - każdy krok iteracji jest osobną, widoczną tabelą.

![Piąta iteracja k-średnich: centroidy, odległości, przypisania i zerowa suma różnic](obrazy/k-srednich-zbieznosc.png)

*Iteracja piąta. Po lewej centroidy czterech skupień, po prawej odległości każdego obiektu do wszystkich centroidów i wynikające z nich przypisanie. Kolumna `różnica` porównuje przypisanie z poprzednią iteracją - **`Konwergencja = 0`** oznacza, że żaden z 50 obiektów nie zmienił skupienia i algorytm się zatrzymał.*

## 04 · Sieć neuronowa z dwiema warstwami ukrytymi

**Dane:** 172 rekordy, 4 zmienne numeryczne, 3 klasy o etykietach liczbowych 39 / 69 / 84 (rozkład 42 / 53 / 77). Projekt studencki, **źródła zbioru nie udało się odtworzyć** - zakresy zmiennych: x1 ∈ [26,3; 60,4], x2 ∈ [-9,3; -3,3], x3 ∈ [-4,7; -0,4], x4 ∈ {3…9}.

**Architektura:** 4 wejścia → 3 neurony (warstwa 1) → 3 neurony (warstwa 2) → 1 wyjście. Sigmoida po obu warstwach, agregacja przez `SUMPRODUCT`/`INDEX`, wyjście przez `MMULT`. Sieć uczona jak regresja na wartościach klas, a wynik progowany na trzy przedziały.

**Wynik:** 48 błędów na 172 rekordy (72,1%), bez podziału na zbiór uczący i testowy.

**Uczciwie:** to najbardziej rozbudowana implementacja w całym zestawie i najsłabszy rezultat. Wynik nie jest porównywalny z projektem 01 - to inne dane i inne zadanie. Zostawiam go, bo sama konstrukcja dwuwarstwowej sieci w arkuszu jest tu ciekawsza niż jej skuteczność.

## 05 · Regresja liniowa

**Dane:** 8 obserwacji, jedna zmienna objaśniająca (temperatura → sprzedaż). Dane poglądowe.

**Metoda:** współczynniki wyprowadzone z równań normalnych metody najmniejszych kwadratów, policzone bezpośrednio z sum (`SUMPRODUCT`, `SUM`, `COUNT`) - bez użycia gotowej funkcji regresji. Wynik: `y = 6,906·x - 130,236`.

**Diagnostyka:** obok, dla porównania, pełny blok `LINEST` - błędy standardowe współczynników (1,592 i 52,642), **R² = 0,758**, statystyka F = 18,82 przy 6 stopniach swobody, rozkład sumy kwadratów na wyjaśnioną (1760,8) i resztową (561,3).

**Ograniczenie:** osiem punktów to zdecydowanie za mało, by traktować te przedziały ufności poważnie. Projekt pokazuje wyprowadzenie i odczyt diagnostyki, nie wnioskowanie.

![Wykres punktowy ośmiu obserwacji: temperatura a sprzedaż](obrazy/regresja-wykres.png)

*Osiem obserwacji - temperatura na osi poziomej, sprzedaż na pionowej.*

![Sumy do równań normalnych, wyliczone współczynniki, predykcje oraz blok LINEST](obrazy/regresja-linest.png)

*U góry sumy potrzebne do równań normalnych i wyliczone z nich współczynniki (`a = 6,906`, `b = -130,236`) wraz z trzema predykcjami. Na dole blok `LINEST` do porównania: błędy standardowe współczynników (1,592 i 52,642), **R² = 0,758**, statystyka F = 18,82 przy 6 stopniach swobody oraz rozkład sumy kwadratów na wyjaśnioną (1760,8) i resztową (561,3).*

---

## Dane źródłowe

Katalog [`dane/`](dane) zawiera dane wejściowe wyeksportowane do CSV, żeby dało się je obejrzeć bez Excela i użyć do sprawdzenia wyników w dowolnym narzędziu.

Wszystkie zbiory są publicznymi zbiorami wzorcowymi albo danymi poglądowymi. Nie ma tu żadnych danych osobowych ani firmowych.

## Uwagi techniczne

Arkusze zapisano razem z wynikami, więc widać je od razu po otwarciu - nie trzeba niczego przeliczać.

Wagi obu sieci neuronowych (projekty 01 i 04) dopasowano dodatkiem **Solver**; jego ustawienia są nadal zapisane w tych plikach. Pozostałe projekty liczą się wprost z formuł.

---

# Machine learning from scratch in Excel

Five machine learning algorithms implemented **using spreadsheet formulas only** - no Python, no libraries, no `model.fit()`. Forward propagation, normalisation, Euclidean distance, least-squares fitting - every step is visible in a cell and can be traced.

These are university projects. I am publishing them because they show something a notebook calling `sklearn` cannot: **that I understand what these algorithms actually compute.**

| # | Project | Data | Method | Result |
|---|---|---|---|---|
| [01](01-siec-1-warstwa) | Neural network, 1 hidden layer | 683 records, 9 features, 2 classes | 3 neurons, sigmoid, min-max scaling | **97.3% on the test set** (5 errors / 183) |
| [02](02-lda-iris) | Linear discriminant analysis | Iris - 150 records, 4 features, 3 classes | Fisher's LDA | 98.7% (2 errors / 150) |
| [03](03-k-srednich) | K-means clustering | 50 objects, 4 features | k = 4, Lloyd's iterations | converged at iteration 5 |
| [04](04-siec-2-warstwy) | Neural network, 2 hidden layers | 172 records, 4 features, 3 classes | 3 + 3 neurons, sigmoid | 72.1% (48 errors / 172) |
| [05](05-regresja-liniowa) | Linear regression | 8 points, 1 predictor | OLS from formulas + diagnostics | R² = 0.758 |

**How to read these numbers.** Only project 01 uses a genuine train/test split: the model was fitted on 500 records, and 97.3% is measured on 183 records the optimisation never saw. Projects 02, 04 and 05 report accuracy **on their own training data**, so those figures are optimistic and say nothing about generalisation. I left them as they were, with the caveat that today I would split the data before fitting in every one of them - as I did in project 01.

Project 01 (single hidden layer, 9 inputs → 3 hidden neurons → 1 output, sigmoid activation, threshold from class-weighted means) is the strongest piece here, precisely because it is the only one that measures what matters. Project 03 is the most instructive: each iteration of Lloyd's algorithm lives on its own worksheet, so the whole loop - distances, reassignment, centroid update, convergence check - is laid out as visible tables rather than hidden inside a function call. Project 04 is the most elaborate construction and the weakest result; its dataset source could not be recovered, and its score is not comparable to project 01 since the task and data differ.

Source data is exported to CSV in [`dane/`](dane). All datasets are public benchmark sets or illustrative data - no personal or company information.
