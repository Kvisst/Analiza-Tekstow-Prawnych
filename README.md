# Analiza tekstów prawnych

Projekt zaliczeniowy z przedmiotu **Projektowanie Systemów Informatycznych 2026**. Repozytorium zawiera prototypowy zestaw narzędzi do przygotowania i eksploracyjnej analizy tekstu prawnego. Na przykładzie Konstytucji RP skrypty dzielą dokument na artykuły, czyszczą tekst, a następnie badają słownictwo i jego współwystępowanie.

> **Ważne:** wyniki są orientacyjną analizą tekstu, opartą na ręcznie przygotowanych słownikach i regułach. Nie stanowią wykładni prawa ani profesjonalnej opinii prawnej.

## Do czego służy projekt?

Celem projektu jest pokazanie, jak metody analizy tekstu i eksploracji danych można zastosować do dokumentu prawnego. Analiza koncentruje się na Konstytucji RP i umożliwia m.in.:

- porównanie występowania słownictwa **liberalnego**, **restrykcyjnego** i **instytucjonalnego**;
- sprawdzenie, jak uwzględnienie negacji wpływa na klasyfikację oraz indeks liberalności artykułów;
- wyszukanie powiązań między słowami, wyodrębnienie tematów i przedstawienie sieci współwystępowania;
- porównanie częstotliwości słów z ich wagami TF-IDF.

## Jak działają skrypty?

### `Konwersja danych/Podział_artykuły.py`

Wczytuje `Konstytucja.txt`, usuwa oznaczenia Kancelarii Sejmu, dzieli dokument przy nagłówkach `Art. N.`, a treść każdego artykułu scala do jednej linii. Preambuła jest zapisywana jako osobny rekord. Wynikiem jest `konstytucja_artykuly.csv` z kolumnami `id` i `text`.

### `Kod_wersja_finalna.R`

Główny skrypt analityczny. Wczytuje `konstytucja_artykuly.csv`, normalizuje tekst (m.in. małe litery, usunięcie cyfr, interpunkcji i polskich słów funkcyjnych) oraz stosuje ręczne sprowadzanie wybranych form prawnych do wspólnych postaci. Następnie:

1. tworzy pliki z tokenami i oczyszczonym tekstem;
2. klasyfikuje słowa przy użyciu ręcznie zdefiniowanych słowników i przygotowuje zestawienia globalne oraz wyniki dla artykułów;
3. porównuje klasyfikację z negacją i bez niej oraz wylicza indeks `(liczba słów liberalnych − liczba słów restrykcyjnych) / (ich suma)`;
4. analizuje asocjacje słów, częstość terminów i tematy metodą LDA;
5. tworzy wykresy, chmury słów, sieć współwystępowania oraz zestawienia TF-IDF.

Skrypt wypisuje część wyników w konsoli, zapisuje tabele CSV i generuje wykresy w uruchomionym środowisku R. Uwaga: w skrypcie uruchamiane są modele LDA dla 3, 4 i **9** tematów (mimo komentarza mówiącego o 5 tematach).

### `Konwersja danych/Wizualizacja.py`

Wczytuje oczyszczony plik `konstytucja_czysta.csv`. Dla każdego artykułu zlicza pary różnych słów, które w nim wystąpiły, zachowuje połączenia o wadze co najmniej 10, wykrywa społeczności algorytmem Louvain i rysuje sieć. Zapisuje wykres `siec_konstytucja.png` oraz graf `konstytucja_network.gexf` (do dalszej eksploracji np. w Gephi).

## Zawartość repozytorium

```text
.
├── Kod_wersja_finalna.R                 # główny skrypt analityczny w R
├── RRaport_wersja_finalna.html           # gotowy raport z analizy
├── konstytucja_artykuly.csv              # tekst podzielony na preambułę i artykuły
├── Specyfikacja Projektowa.pdf           # specyfikacja projektu
└── Konwersja danych/
    ├── Konstytucja.txt                   # tekst źródłowy
    ├── Podział_artykuły.py               # podział źródła na artykuły
    ├── konstytucja_czysta.csv            # przykładowy wynik czyszczenia
    ├── Wizualizacja.py                   # budowa i eksport sieci współwystępowania
    ├── konstytucja_network.gexf           # wyeksportowany graf
    └── siec_konstytucja.png               # przykładowa wizualizacja grafu
```

## Uruchomienie

### Wymagania

- Python 3 oraz pakiety `pandas`, `numpy`, `networkx`, `matplotlib`, `python-louvain` i `adjustText`.
- R oraz pakiety: `tm`, `stringr`, `dplyr`, `readr`, `stopwords`, `tidyr`, `ggplot2`, `tibble`, `tidyverse`, `tidytext`, `widyr`, `igraph`, `ggraph`, `topicmodels`, `wordcloud`, `reshape2` i `ggwordcloud`.

W repozytorium nie ma plików deklarujących lub instalujących zależności, dlatego trzeba zainstalować je przed uruchomieniem. Przykładowa instalacja:

```bash
python -m pip install pandas numpy networkx matplotlib python-louvain adjustText
```

W R:

```r
install.packages(c(
  "tm", "stringr", "dplyr", "readr", "stopwords", "tidyr", "ggplot2",
  "tibble", "tidyverse", "tidytext", "widyr", "igraph", "ggraph",
  "topicmodels", "wordcloud", "reshape2", "ggwordcloud"
))
```

### 1. Przygotowanie danych (opcjonalne)

Plik `konstytucja_artykuly.csv` znajduje się już w katalogu głównym. Jeśli chcesz odtworzyć go z tekstu źródłowego, uruchom konwerter z katalogu `Konwersja danych`, ponieważ ścieżki w skrypcie są względne:

```bash
cd "Konwersja danych"
python "Podział_artykuły.py"
```

Skrypt zapisze CSV w tym samym katalogu. Główny skrypt R oczekuje tego pliku w katalogu głównym repozytorium, więc przed jego uruchomieniem skopiuj wygenerowany plik poziom wyżej (albo odpowiednio zmień ścieżkę w kodzie). Przykładowo, z katalogu głównego repozytorium:

```bash
cp "Konwersja danych/konstytucja_artykuly.csv" .
```

### 2. Analiza w R

Uruchom `Kod_wersja_finalna.R` w RStudio lub z katalogu głównego repozytorium, aby względna ścieżka do danych była poprawna:

```bash
Rscript Kod_wersja_finalna.R
```

Skrypt zapisuje wygenerowane tabele CSV w bieżącym katalogu. Wykresy są tworzone przez polecenia `ggplot` i pozostałe funkcje graficzne; gotowy raport HTML znajduje się już w repozytorium jako `RRaport_wersja_finalna.html`.

### 3. Wizualizacja sieci w Pythonie (opcjonalnie)

`Wizualizacja.py` oczekuje `konstytucja_czysta.csv` w bieżącym katalogu i zapisuje tam obraz oraz plik GEXF. Przykładowy plik wejściowy jest w `Konwersja danych`, więc uruchom skrypt stamtąd:

```bash
cd "Konwersja danych"
python Wizualizacja.py
```

## Ograniczenia

- Czyszczenie i sprowadzanie form do wspólnych postaci jest oparte na regułach oraz ręcznych listach; nie jest pełną analizą morfologiczną języka polskiego.
- Klasyfikacja zależy od zawartości słowników i uproszczonej obsługi negacji. Indeks liberalności jest wskaźnikiem projektu, a nie obiektywną oceną aktu prawnego.
- Skrypty używają ścieżek względnych i zakładają konkretne katalogi uruchomienia. Pliki wynikowe mogą zostać zapisane w innym miejscu, jeśli uruchomisz je z innego katalogu.
- Tekst źródłowy i jego konwersja mogą zawierać elementy paginacji lub inne artefakty ekstrakcji, które wpływają na wyniki.

## Autorzy

- Maciej Gwizd
- Jakub Ładno
- Igor Rosiński
