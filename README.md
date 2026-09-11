# Administracja mapowaniem pól faktury KSeF per dostawca (enova365)

> Element większej całości: **[Obieg faktur zakupu z KSeF w enova365 — mapa rozwiązania](https://github.com/trynityeu/enova365-obieg-faktur-ksef)**

Dodatek do systemu ERP **enova365** (Soneta sp. z o.o.) — narzędzie
administracyjne do tworzenia i utrzymywania **schematu mapowania XML** dla
faktur zakupu odbieranych przez **Krajowy System e-Faktur (KSeF)**. Nie
importuje dokumentów — wyłącznie zarządza konfiguracją, z której korzysta
osobny dodatek importujący.

Import faktur KSeF realizują **dwa odrębne dodatki**, zależnie od rodzaju
dokumentu:

- [Import faktur zakupu materiałowego (ZME) z dopasowaniem do zamówień](https://github.com/trynityeu/enova365-import-faktur-ksef-dopasowanie)
  — **jedyny odbiorca** schematów utrzymywanych tym dodatkiem;
- [Import faktur kosztowych (ZKE) i samochodowych (ZSE)](https://github.com/trynityeu/enova365-import-faktur-ksef-koszty-pojazdy)
  — własny zestaw reguł księgowych, ze schematów mapowania **nie korzysta**.

O tym, którą z tych ścieżek pojedzie konkretna faktura, rozstrzyga
[automatyczna klasyfikacja przy pobraniu z KSeF](https://github.com/trynityeu/enova365-klasyfikacja-faktur-ksef).

To repozytorium zawiera wyłącznie **opis funkcjonalny** — bez kodu
źródłowego, bez rzeczywistych schematów dostawców, bez danych klienta.

## Problem, który rozwiązuje

Każdy dostawca inaczej wypełnia faktury w formacie KSeF (schemat **FA(3)**):
inny sposób zapisu numeru zamówienia, inne pole na indeks towaru, inne
jednostki miary, czasem numer zamówienia w ogóle nieobecny albo ukryty
w polu opisowym razem z innymi informacjami.
[Import faktur ZME](https://github.com/trynityeu/enova365-import-faktur-ksef-dopasowanie)
potrzebuje wiedzieć, **jak konkretnie czytać XML tego dostawcy** —
ale ta wiedza zmienia się w czasie (dostawca zmienia system, dopisuje nowe
pole, zaczyna wystawiać nowy rodzaj usługi) i nie powinna wymagać zmiany
kodu ani nowej wersji dodatku za każdym razem.

## Jak działa

- **Przycisk na liście pobranych plików KSeF**, aktywny wyłącznie dla
  faktur rodzaju „ZME-Magazynowe" (schematy mapowania dotyczą tylko tego
  procesu — na innych rodzajach, np. kosztowych, przycisk się nie pokazuje).
- Operator **zaznacza jedną lub więcej faktur tego samego dostawcy** i
  otwiera okno administracyjne. Wielokrotne zaznaczenie pozwala od razu
  sprawdzić schemat na próbce kilku różnych faktur naraz.
- Okno pokazuje: dane kontrahenta, listę zaznaczonych faktur, pole tekstowe
  ze **schematem** (wypełnione dotychczasową treścią, jeśli już istnieje)
  oraz **raport** — porównanie zapisanego schematu ze strukturą XML
  wybranych faktur.
- **Dla dostawcy bez schematu** raport zamienia się w **podpowiedź
  struktury**: automatycznie wypisane klucze pól opisowych obecne w
  fakturze, użyte jednostki miary, informacja czy faktura niesie indeks
  towaru / kod GTIN / numer dokumentu magazynowego dostawcy — gotowy punkt
  wyjścia do napisania schematu od zera, zamiast ręcznego przeglądania XML.
- **Zatwierdzenie schematu jest dwustopniowe**: najpierw walidacja
  **formalna** (poprawność składni — błędna reguła blokuje zapis i wraca
  do poprawy w oknie), potem **porównanie** z rzeczywistą strukturą
  faktur — tu niezgodność **nie blokuje** zapisu, tylko wymaga świadomego
  potwierdzenia, bo zmiana wzorca bywa jednorazowa (dostawca akurat wystawił
  fakturę za samą usługę, bez towaru) i nie zawsze oznacza błąd schematu.
- **Zapis zostawia ślad czasowy**: data najstarszej faktury, od której nowy
  schemat obowiązuje, oraz data najnowszej przeanalizowanej — zabezpieczenie
  przed przypadkowym cofnięciem się do analizy starszych dokumentów po
  zatwierdzeniu nowszej wersji.
- **Bez historii wersji w klasycznym sensie** — poprzednia treść schematu
  nie jest kasowana, tylko oznaczana jako archiwalna i zostaje w tym samym
  polu; dodatek pilnuje, żeby nowy zapis nie „zgubił" żadnego wcześniejszego
  wpisu.

## Mini-język schematu (bez ujawniania rzeczywistych reguł dostawców)

Schemat to **dane, nie kod** — kilka rodzajów linii tekstowych, każda
odpowiadająca na jedno pytanie o strukturę XML:

| Instrukcja | Odpowiada na pytanie |
|---|---|
| `WYMAGAJ <ścieżka>` | Czy dane pole musi wystąpić w fakturze? |
| `WZORZEC <ścieżka> ~ <wzorzec>` | Czy KAŻDE wystąpienie pola musi mieć określony kształt? |
| `WZORZEC-MIN1 <ścieżka> ~ <wzorzec>` | Czy WYSTARCZY, żeby jedno wystąpienie pasowało? |
| `JEDNOSTKA <X> => <Y>` | Jak przeliczyć jednostkę miary dostawcy na jednostkę enova365? |
| `<cel> <= <ścieżka>` | Skąd wziąć wartość konkretnego pola docelowego? |
| `<cel> = <składnik> + <składnik> + …` | Jak złożyć wartość pola docelowego z kilku fragmentów faktury (i tekstu stałego)? |

Ścieżki porównywane są po nazwie elementu XML (bez względu na przestrzeń
nazw), a klucze pól opisowych — bez względu na wielkość liter i polskie
znaki, więc schemat przenosi się między podobnymi strukturami faktur bez
przepisywania od zera.

## Co dzieje się wcześniej

- Faktura musi zostać **pobrana z KSeF** do listy plików w enova365 (moduł
  integracji KSeF wbudowany w enova365 — poza zakresem tego dodatku).
- Dla nowego dostawcy nie jest wymagany żaden wcześniejszy krok — to
  właśnie ten dodatek generuje pierwszą podpowiedź struktury.

## Co dzieje się później

- Zapisany schemat jest **wyłącznie czytany** przez osobny dodatek
  importujący — on wykonuje faktyczne dopasowanie pozycji faktury do
  zamówień zakupu i tworzy dokument ewidencji zakupu materiałowego (ZME).
  Ten dodatek administracyjny niczego nie importuje ani nie księguje.
  → [Import faktur zakupu KSeF z automatycznym dopasowaniem do zamówień](https://github.com/trynityeu/enova365-import-faktur-ksef-dopasowanie)
- Dla faktur kosztowych i samochodowych (inny proces, inny rodzaj
  dokumentu) obowiązuje odrębny import z własnym zestawem reguł — nie
  korzysta ze schematów utrzymywanych tym dodatkiem.
  → [Import faktur kosztowych i samochodowych z KSeF](https://github.com/trynityeu/enova365-import-faktur-ksef-koszty-pojazdy)

## Jakie cechy (pola konfiguracyjne enova365) są wykorzystywane

- **Na karcie kontrahenta** — jedna ukryta cecha tekstowa niosąca cały
  schemat mapowania XML tego dostawcy (treść w mini-języku opisanym wyżej,
  z nagłówkiem dat obowiązywania i ewentualnymi wpisami archiwalnymi).
  Cecha jest celowo „ukryta" — nie pojawia się w standardowych widokach
  karty kontrahenta, żeby nie mylić operatorów niezwiązanych z importem.

## Co jest do tego potrzebne

- enova365 z aktywną integracją KSeF (moduł CRM z kartoteką kontrahentów,
  lista pobranych plików KSeF).
- Zdefiniowana w bazie docelowej cecha administracyjna na kontrahentach —
  dodatek **sam wykrywa jej brak** i ukrywa swój przycisk w bazach, gdzie
  cechy nie zdefiniowano (przydatne przy współdzieleniu jednego zestawu
  dodatków między różnymi firmami/bazami bez ryzyka nieudanego zapisu).
- .NET 8 oraz referencje do bibliotek Soneta.Sdk w wersji zgodnej
  z zainstalowaną enovą.

## Zgodność

| | |
|---|---|
| System | enova365 (Soneta sp. z o.o.), wersja referencyjna **2512.9.10** |
| Platforma | .NET 8 |
| Schemat faktury | KSeF **FA(3)** |
| Zakres | wyłącznie faktury rodzaju „ZME-Magazynowe" |
| Wdrożenie | dodatek instalowany przez interfejs enova365 (Narzędzia → Opcje → Dodatki), bez wymogu podpisu cyfrowego |

## Czego tu nie ma

Kod źródłowy, rzeczywiste schematy mapowania konkretnych dostawców, dane
handlowe i wszelka konfiguracja specyficzna dla wdrożenia pozostają
w prywatnym repozytorium. To repo służy wyłącznie jako publiczny opis
funkcji dodatku.
