# Test case'y dla BR-01, BR-03 i BR-04

## Zakres i źródła

Zakres obejmuje rejestrację hasła (BR-01), dodawanie i zmianę ilości produktu
w koszyku (BR-03) oraz koszt dostawy zależny od wartości produktów po rabacie
(BR-04). Testy wykonuje się w aplikacji uruchomionej lokalnie. Przed każdym
testem koszyka użyj `POST /api/test/reset`, aby przywrócić dane początkowe.

## Klasy równoważności i wartości brzegowe

| Reguła | Klasa / granica | Przykład | Oczekiwanie |
|---|---|---|---|
| BR-01 | Hasło krótsze niż 8 znaków | `A123456` (7) | Niepoprawne |
| BR-01 | Minimum długości | `A1234567` (8) | Poprawne |
| BR-01 | Długość 9–63 znaków, wielka litera i cyfra | `Kawa12345` | Poprawne |
| BR-01 | Maksimum długości | hasło 64-znakowe z `A` i `1` | Poprawne |
| BR-01 | Powyżej maksimum | hasło 65-znakowe z `A` i `1` | Niepoprawne |
| BR-01 | Brak wielkiej litery | `kawa12345` | Niepoprawne |
| BR-01 | Brak cyfry | `Kawaaaaa` | Niepoprawne |
| BR-03 | Ilość poniżej minimum | 0 lub -1 | Niepoprawna |
| BR-03 | Minimum ilości | 1 szt. | Poprawna, jeśli jest na stanie |
| BR-03 | Ilość 2–9 szt. | 2 szt. | Poprawna, jeśli jest na stanie |
| BR-03 | Maksimum ilości | 10 szt. | Poprawna, jeśli jest na stanie |
| BR-03 | Powyżej maksimum | 11 szt. | Niepoprawna |
| BR-03 | Ilość równa stanowi | produkt 3, 3 szt. | Poprawna |
| BR-03 | Ilość większa niż stan | produkt 3, 4 szt. | Niepoprawna |
| BR-03 | Stan magazynowy 0 | produkt 5, 1 szt. | Niepoprawna |
| BR-04 | Wartość poniżej progu | 198,99 zł | Standard 14,99 zł |
| BR-04 | Próg | 200,00 zł | Standard gratis |
| BR-04 | Wartość powyżej progu | 218,98 zł | Standard gratis |
| BR-04 | Express poniżej progu | 198,99 zł | 24,99 zł |
| BR-04 | Express przy darmowej dostawie | 200,00 zł | 10,00 zł |
| BR-04 | Rabat obniża wartość poniżej progu | 218,98 zł − 20,00 zł | Standard 14,99 zł |

## Tablica decyzyjna

| Warunek / działanie BR-04 | D1 | D2 | D3 | D4 |
|---|---:|---:|---:|---:|
| Wartość produktów po rabacie < 200,00 zł | Tak | Nie | Nie | Tak |
| Sposób dostawy EXPRESS | Nie | Nie | Tak | Nie |
| Standard | 14,99 zł | Gratis | — | 14,99 zł |
| Express | — | — | 10,00 zł | — |
| Wynik dla przykładu | 198,99 zł | 200,00 zł | 200,00 zł | 198,98 zł po rabacie |

## Test case'y

### TC-REJ-001: Odrzuca hasło krótsze niż 8 znaków

| Pole | Wartość |
|------|---------|
| Wymaganie | BR-01 |
| Technika | wartości brzegowe |
| Priorytet (ryzyko) | Wysoki (prawdopodobieństwo x wpływ) |
| Warunki wstępne | Strona rejestracji jest otwarta; e-mail nie istnieje |
| Dane testowe | `anna+tc001@beanshop.test`, imię `Test Anna`, hasło `A123456` |

**Kroki**
1. Otwórz stronę rejestracji.
2. Wpisz dane testowe i wyślij formularz.

**Oczekiwany wynik**
- Rejestracja jest odrzucona, HTTP 400 (`WEAK_PASSWORD`).
- Komunikat wskazuje wymóg co najmniej 8 znaków; konto nie zostaje utworzone.

**Wynik wykonania**: Zaliczony / Niezaliczony (link do [BUG]) / Zablokowany

### TC-REJ-002: Akceptuje hasło na granicy 8 znaków

| Pole | Wartość |
|------|---------|
| Wymaganie | BR-01 |
| Technika | wartości brzegowe |
| Priorytet (ryzyko) | Wysoki (prawdopodobieństwo x wpływ) |
| Warunki wstępne | E-mail nie istnieje |
| Dane testowe | `anna+tc002@beanshop.test`, `Test Anna`, `A1234567` |

**Kroki**
1. Wyślij formularz rejestracji z danymi testowymi.

**Oczekiwany wynik**
- Odpowiedź ma HTTP 201; konto zostaje utworzone.

**Wynik wykonania**: Zaliczony / Niezaliczony (link do [BUG]) / Zablokowany

### TC-REJ-003: Odrzuca hasło bez wielkiej litery lub cyfry

| Pole | Wartość |
|------|---------|
| Wymaganie | BR-01 |
| Technika | tablica decyzyjna |
| Priorytet (ryzyko) | Wysoki (prawdopodobieństwo x wpływ) |
| Warunki wstępne | E-mail nie istnieje |
| Dane testowe | Dwa osobne zgłoszenia: `kawa12345` oraz `Kawaaaaa` |

**Kroki**
1. Zarejestruj konto z hasłem `kawa12345`.
2. Zarejestruj inne konto z hasłem `Kawaaaaa`.

**Oczekiwany wynik**
- Oba zgłoszenia kończą się HTTP 400 (`WEAK_PASSWORD`).
- Pierwszy komunikat wskazuje brak wielkiej litery, drugi brak cyfry.

**Wynik wykonania**: Zaliczony / Niezaliczony (link do [BUG]) / Zablokowany

### TC-REJ-004: Odrzuca hasło dłuższe niż 64 znaki

| Pole | Wartość |
|------|---------|
| Wymaganie | BR-01 |
| Technika | wartości brzegowe |
| Priorytet (ryzyko) | Średni (prawdopodobieństwo x wpływ) |
| Warunki wstępne | E-mail nie istnieje |
| Dane testowe | `anna+tc004@beanshop.test`, `Test Anna`, `A1234567` + 57 znaków `x` (65 znaków) |

**Kroki**
1. Wyślij formularz rejestracji z hasłem 65-znakowym.

**Oczekiwany wynik**
- Rejestracja jest odrzucona HTTP 400; konto nie zostaje utworzone.

**Wynik wykonania**: Zaliczony / Niezaliczony (link do [BUG]) / Zablokowany

### TC-KOSZ-001: Dodaje ilość 1 i 10, odrzuca 0 oraz 11

| Pole | Wartość |
|------|---------|
| Wymaganie | BR-03 |
| Technika | wartości brzegowe |
| Priorytet (ryzyko) | Wysoki (prawdopodobieństwo x wpływ) |
| Warunki wstępne | Zalogowana Anna; pusty koszyk |
| Dane testowe | Produkt 4 (`Espresso Blend 500 g`, stan 20); ilości 0, 1, 10, 11 |

**Kroki**
1. Dodaj produkt 4 w ilości 0, następnie 1, 10 i 11 (resetuj koszyk między próbami).

**Oczekiwany wynik**
- Ilości 0 i 11 są odrzucone HTTP 400.
- Ilości 1 i 10 są dodane HTTP 201, a koszyk pokazuje dokładnie podaną ilość.

**Wynik wykonania**: Zaliczony / Niezaliczony (link do [BUG]) / Zablokowany

### TC-KOSZ-002: Nie przekracza stanu magazynowego i odrzuca produkt bez stanu

| Pole | Wartość |
|------|---------|
| Wymaganie | BR-03 |
| Technika | klasy równoważności / wartości brzegowe |
| Priorytet (ryzyko) | Wysoki (prawdopodobieństwo x wpływ) |
| Warunki wstępne | Zalogowana Anna; pusty koszyk |
| Dane testowe | Produkt 3, stan 3: ilości 3 i 4; produkt 5, stan 0: ilość 1 |

**Kroki**
1. Dodaj produkt 3 w ilości 3.
2. Usuń pozycję i dodaj produkt 3 w ilości 4.
3. Usuń pozycję i dodaj produkt 5 w ilości 1.

**Oczekiwany wynik**
- Ilość 3 produktu 3 jest dodana HTTP 201.
- Ilość 4 produktu 3 jest odrzucona HTTP 409 (`OUT_OF_STOCK`).
- Produkt 5 jest odrzucony HTTP 409 (`OUT_OF_STOCK`).

**Wynik wykonania**: Zaliczony / Niezaliczony (link do [BUG]) / Zablokowany

### TC-KOSZ-003: Zmiana ilości nie pozwala ustawić zera

| Pole | Wartość |
|------|---------|
| Wymaganie | BR-03 |
| Technika | wartości brzegowe / zgadywanie błędów |
| Priorytet (ryzyko) | Wysoki (prawdopodobieństwo x wpływ) |
| Warunki wstępne | Zalogowana Anna; koszyk zawiera produkt 4 w ilości 1 |
| Dane testowe | PATCH `/api/cart/items/4`, `{ "quantity": 0 }` |

**Kroki**
1. Ustaw ilość produktu 4 na 0 w koszyku.

**Oczekiwany wynik**
- Operacja jest odrzucona HTTP 400.
- Ilość w koszyku pozostaje równa 1 (usunięcie odbywa się osobną akcją).

**Wynik wykonania**: Zaliczony / Niezaliczony (link do [BUG]) / Zablokowany

### TC-DOST-001: Nalicz opłatę standardową poniżej progu

| Pole | Wartość |
|------|---------|
| Wymaganie | BR-04 |
| Technika | wartości brzegowe / tablica decyzyjna |
| Priorytet (ryzyko) | Wysoki (prawdopodobieństwo x wpływ) |
| Warunki wstępne | Zalogowana Anna; pusty koszyk; brak rabatu |
| Dane testowe | Produkt 6 za 159,00 zł + produkt 2 za 39,99 zł = 198,99 zł; STANDARD |

**Kroki**
1. Dodaj produkt 6 w ilości 1 i produkt 2 w ilości 1.
2. Wybierz „Kurier standard”.

**Oczekiwany wynik**
- Wartość produktów: 198,99 zł; dostawa: 14,99 zł; suma: 213,98 zł.

**Wynik wykonania**: Zaliczony / Niezaliczony (link do [BUG]) / Zablokowany

### TC-DOST-002: Daje darmową dostawę dokładnie od 200,00 zł

| Pole | Wartość |
|------|---------|
| Wymaganie | BR-04 |
| Technika | wartości brzegowe |
| Priorytet (ryzyko) | Wysoki (prawdopodobieństwo x wpływ) |
| Warunki wstępne | Zalogowana Anna; pusty koszyk; brak rabatu |
| Dane testowe | Produkt 8 za 100,00 zł, ilość 2; STANDARD |

**Kroki**
1. Dodaj produkt 8 w ilości 2.
2. Wybierz „Kurier standard”.

**Oczekiwany wynik**
- Wartość produktów: 200,00 zł; dostawa: Gratis (0,00 zł); suma: 200,00 zł.

**Wynik wykonania**: Zaliczony / Niezaliczony (link do [BUG]) / Zablokowany

### TC-DOST-003: Express kosztuje 10,00 zł, gdy dostawa standardowa jest darmowa

| Pole | Wartość |
|------|---------|
| Wymaganie | BR-04 |
| Technika | tablica decyzyjna |
| Priorytet (ryzyko) | Wysoki (prawdopodobieństwo x wpływ) |
| Warunki wstępne | Zalogowana Anna; koszyk z produktem 8 x 2 |
| Dane testowe | Wartość produktów 200,00 zł; EXPRESS |

**Kroki**
1. Wybierz „Kurier express”.

**Oczekiwany wynik**
- Dostawa kosztuje 10,00 zł, a suma wynosi 210,00 zł.

**Wynik wykonania**: Zaliczony / Niezaliczony (link do [BUG]) / Zablokowany

### TC-DOST-004: Sprawdza próg po rabacie

| Pole | Wartość |
|------|---------|
| Wymaganie | BR-04 |
| Technika | tablica decyzyjna |
| Priorytet (ryzyko) | Wysoki (prawdopodobieństwo x wpływ) |
| Warunki wstępne | Zalogowana Anna; pusty koszyk |
| Dane testowe | Produkty 6 x 1 i 2 x 1 = 198,99 zł (wariant poniżej progu) albo 6 x 1, 2 x 1 i 9 x 1 = 218,98 zł; kod `MINUS20`; STANDARD |

**Kroki**
1. Dodaj produkt 6 x 1, produkt 2 x 1 i produkt 9 x 1.
2. Zastosuj kod `MINUS20`.
3. Wybierz „Kurier standard”.

**Oczekiwany wynik**
- Wartość produktów: 218,98 zł; rabat: 20,00 zł; wartość po rabacie: 198,98 zł.
- Dostawa kosztuje 14,99 zł, a suma wynosi 213,97 zł.

**Wynik wykonania**: Zaliczony / Niezaliczony (link do [BUG]) / Zablokowany

## Rozbieżności kodu z wymaganiami

- **MOŻLIWY BŁĄD (BR-01):** `src/domain/password.ts:7-12` sprawdza minimum,
  wielką literę i cyfrę, ale nie sprawdza maksimum 64 znaków. Rejestracja
  wywołuje tę funkcję w `src/routes/auth.ts:24-26`.
- **MOŻLIWY BŁĄD (BR-03):** `src/routes/cart.ts:21` waliduje zmianę ilości
  tylko przez `.max(10)`, bez minimum 1. W efekcie `PATCH` z ilością 0 może
  przejść (`src/routes/cart.ts:55-73`).
- **MOŻLIWY BŁĄD (BR-04):** `src/domain/pricing.ts:50` używa `>` zamiast
  `>=`, więc dokładnie 200,00 zł nie spełnia warunku darmowej dostawy.

## Pytania do PO

- Brak pytań blokujących testy; wymagania BR-01, BR-03 i BR-04 określają
  oczekiwane zachowanie jednoznacznie.

## Źródła

- `docs/wymagania.md`: BR-01, BR-03, BR-04.
- `src/domain/password.ts`: `validatePassword`.
- `src/routes/auth.ts`: endpoint `POST /api/auth/register`.
- `src/routes/cart.ts`: walidacja i obsługa ilości koszyka.
- `src/domain/pricing.ts`: `shippingCost` i `priceCart`.
- `src/store.ts`: `seedProducts` i stany magazynowe.
