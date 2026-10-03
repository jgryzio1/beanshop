# Przepływ od dodania produktu do złożenia zamówienia

Diagram dotyczy zalogowanego klienta. Operacje koszyka wymagają autoryzacji.

```mermaid
flowchart TD
    A[Klient dodaje produkt<br/>POST /api/cart/items] --> B{Dane poprawne,<br/>produkt istnieje i jest dostępny?}
    B -- Nie --> C[Błąd: 400, 404 lub 409]
    B -- Tak --> D[Produkt trafia do koszyka]

    D --> E[Klient może zmienić ilość<br/>PATCH /api/cart/items/:productId]
    E --> F{Ilość i stan magazynowy poprawne?}
    F -- Nie --> G[Błąd walidacji lub OUT_OF_STOCK]
    F -- Tak --> H[Zaktualizowany koszyk]
    H --> I
    D --> I[System zwraca koszyk i summary]

    I --> J[Opcjonalnie: kod rabatowy<br/>POST /api/cart/discount]
    J --> K[Opcjonalnie: metoda dostawy<br/>PUT /api/cart/shipping]
    K --> L[Klient przegląda podsumowanie]
    L --> M[Klient składa zamówienie<br/>POST /api/orders]

    M --> N{Koszyk niepusty<br/>i ilości dostępne w magazynie?}
    N -- Nie --> O[Błąd: EMPTY_CART lub OUT_OF_STOCK]
    N -- Tak --> P[Utworzenie zamówienia ze statusem NEW]
    P --> Q[Odjęcie produktów ze stanu magazynowego]
    Q --> R[Wyczyszczenie koszyka]
    R --> S[Odpowiedź 201<br/>z danymi zamówienia i summary]
```

## Reguły do weryfikacji

- **BR-03:** ilość produktu wynosi od 1 do 10 sztuk i nie może przekraczać stanu magazynowego.
- **BR-04–BR-08:** podsumowanie uwzględnia rabat, dostawę i zaokrąglenia kwot.
- **BR-09:** złożone zamówienie otrzymuje status `NEW`.

**MOŻLIWY BŁĄD (BR-03):** w `PATCH /api/cart/items/:productId` kod ogranicza ilość tylko z góry do 10; nie wymusza minimum 1.

## Źródła

- `docs/wymagania.md` — BR-03–BR-09
- `docs/architektura.md` — opis przepływu „złóż zamówienie”
- `docs/api.md` — kontrakty endpointów koszyka i zamówień
- `src/routes/cart.ts` — `cartRouter.post('/items')`, `cartRouter.patch('/items/:productId')`, `cartRouter.post('/discount')`, `cartRouter.put('/shipping')`, `cartView`
- `src/routes/orders.ts` — `ordersRouter.post('/')`
