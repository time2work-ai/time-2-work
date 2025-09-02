# Kompendium Ewidencji Czasu Pracy i Urlopów w Polsce

## Podstawy prawne i wymogi dla różnych form zatrudnienia

### Obowiązki ewidencyjne według typu umowy

**Umowa o pracę** (art. 149 KP + Rozporządzenie z 10 grudnia 2018 r.)
- **Podstawa prawna**: Art. 149 § 1 Kodeksu pracy + § 6 pkt 1a Rozporządzenia MRPiPS
- **Obowiązek**: BEZWZGLĘDNY dla wszystkich pracowników
- **Wymagane dane**:
  ```
  - Godziny pracy ORAZ dokładny czas rozpoczęcia/zakończenia
  - Godziny dyżuru ORAZ dokładny czas dyżuru z lokalizacją
  - Dni wolne od pracy ze wskazaniem powodu
  - Rodzaj i wymiar urlopu
  - Rodzaj i wymiar usprawiedliwionych/nieusprawiedliwionych nieobecności
  - Praca nieletnich przy zadaniach zabronionych (gdy dotyczy)
  ```
- **Okres przechowywania**: 10 lat od końca roku kalendarzowego zakończenia zatrudnienia (zatrudnieni po 1 stycznia 2019)

**Umowa zlecenie**
- **Podstawa prawna**: Ustawa o minimalnym wynagrodzeniu
- **Obowiązek**: Tylko gdy weryfikacja minimalnego wynagrodzenia
- **Wymagane dane**: Godziny pracy dla kalkulacji minimalnej stawki
- **Algorytm**: `IF umowa_zlecenie AND minimalne_wynagrodzenie = TRUE THEN ewidencja_godzin`

**Umowa o dzieło / B2B**
- **Podstawa prawna**: Kodeks cywilny
- **Obowiązek**: BRAK obowiązku ewidencji czasu pracy
- **Wyjątek**: Ryzyko przekwalifikowania na umowę o pracę przy cechach stosunku pracy

### Uproszczona ewidencja (Art. 149 § 2 KP)
Dotyczy:
- Pracowników w zadaniowym czasie pracy
- Kadry zarządzającej
- Pracowników z ryczałtem za nadgodziny/pracę nocną

Wymaga tylko:
- Rodzaj i wymiar urlopu
- Rodzaj i wymiar nieobecności
- Dni wolne z powodami

## Systemy organizacji czasu pracy

### 1. System podstawowy (art. 128-129 KP)
```python
def podstawowy_czas_pracy():
    dzienny_limit = 8  # godzin
    tygodniowy_średnia = 40  # godzin w 5-dniowym tygodniu
    okres_rozliczeniowy = 4  # miesiące max
    odpoczynek_dobowy = 11  # godzin nieprzerwanych
    odpoczynek_tygodniowy = 35  # godzin nieprzerwanych
```

### 2. System równoważny (art. 135-137 KP)
```python
def równoważny_system():
    dzienny_limit = {
        'standardowy': 12,
        'monitoring_urządzeń': 16,
        'ochrona_straż': 24
    }
    okres_rozliczeniowy = {
        'standardowy': 1,  # miesiąc
        'uzasadniony': 3,  # miesiące
        'sezonowy': 4,  # miesiące
        'obiektywnie_uzasadniony': 12  # miesięcy
    }
    # Bilansowanie dłuższych dni krótszymi lub wolnymi
```

### 3. Ruchomy czas pracy (art. 140¹ KP)
```python
def ruchomy_czas():
    wariant_1 = "różne godziny rozpoczęcia w poszczególnych dniach"
    wariant_2 = "przedział czasu dla decyzji pracownika"
    czas_dzienny = 8  # godzin (elastyczny start)
    możliwe_godziny_rdzenne = True  # ustalone przez pracodawcę
```

### 4. Zadaniowy czas pracy (art. 140 KP)
```python
def zadaniowy_system():
    pracownik_decyduje = "kiedy i jak wykonać zadania"
    zadania_odpowiadają = "8h/dziennie, 40h/tydzień średnio"
    ewidencja = "uproszczona - bez godzinowej rejestracji"
```

### 5. Przerywany czas pracy (art. 139 KP)
```python
def przerywany_system():
    max_przerw_dziennie = 1
    max_czas_przerwy = 5  # godzin
    dzienny_czas_pracy = 8  # godzin
    obecność_całkowita = czas_pracy + przerwa
```

### 6. System pracy w ruchu ciągłym (art. 138 KP)
```python
def ruch_ciągły():
    dzienny_limit = 12  # godzin
    tygodniowy_średnia = 43  # godziny
    okres_rozliczeniowy = 4  # tygodnie max
```

### 7. Praca weekendowa (art. 151¹-151¹¹ KP)
```python
def weekendowy_system():
    dni_pracy = ["piątek", "sobota", "niedziela", "święta"]
    dzienny_limit = 8  # godzin
    okres_rozliczeniowy = 1  # miesiąc
```

**Przykłady praktycznego zastosowania:**

```python
# PRZYKŁAD 1: Kasjer w supermarkecie
class KasjerWeekendowy:
    """Praca tylko w weekendy - popularny model dla studentów"""
    
    def umowa_weekendowa(self):
        return {
            'wymiar_etatu': 0.4,  # 2 dni z 5 = 40%
            'grafik': {
                'piątek': 'wolne',
                'sobota': {'godziny': '08:00-20:00', 'przerwa': 1},  # 11h pracy
                'niedziela': {'godziny': '09:00-18:00', 'przerwa': 0.5},  # 8.5h pracy
            },
            'tygodniowo': 19.5,  # godzin
            'dodatki': {
                'za_niedzielę': 'dwukrotność - sklep pow. 400m²',
                'za_święta': '200% podstawy'
            }
        }
    
    def miesięczne_rozliczenie(self):
        """Przykład: grudzień z świętami"""
        return {
            'soboty': 4 * 11,  # 44h
            'niedziele': 4 * 8.5,  # 34h
            'wigilia_24.12': 6,  # krótsza sobota
            'święta_25-26.12': 0,  # sklep zamknięty
            'sylwester_31.12': 8,  # normalna sobota
            
            'suma_godzin': 92,
            'wynagrodzenie': {
                'podstawa': 92 * stawka_godzinowa,
                'dodatek_niedzielny': 34 * stawka_godzinowa * 1.0,  # 100% dodatku
                'premia_świąteczna': 'uznaniowa'
            }
        }

# PRZYKŁAD 2: Przewodnik turystyczny
class PrzewodnikTurystyczny:
    """Sezonowa praca weekendowa"""
    
    def sezon_letni(self):
        """Maj-Wrzesień intensywna praca"""
        return {
            'piątek': {
                'wycieczki': ['14:00-18:00'],  # 4h
                'typ': 'grupy przyjeżdżające na weekend'
            },
            'sobota': {
                'wycieczki': ['09:00-13:00', '15:00-19:00'],  # 8h
                'typ': 'całodniowe zwiedzanie'
            },
            'niedziela': {
                'wycieczki': ['10:00-14:00', '16:00-18:00'],  # 6h
                'typ': 'krótsze trasy, powroty'
            },
            'pon_czw': 'wolne lub okazjonalne grupy',
            
            'miesięcznie': {
                'weekendy': 4 * 18,  # 72h
                'dodatkowe': 20,  # grupy w tygodniu
                'suma': 92,
                'średnia_tygodniowa': 23  # poniżej 40h
            }
        }
    
    def wynagrodzenie(self):
        return {
            'model': 'stawka za wycieczkę + prowizja',
            'weekendowa_stawka': 150,  # PLN/wycieczka
            'prowizja_wstępy': '10% od biletów',
            'napiwki': 'bezpośrednio od turystów'
        }

# PRZYKŁAD 3: Pracownik eventu/promocji
class PromotorEventowy:
    """Praca na eventach weekendowych"""
    
    def harmonogram_miesieczny(self):
        events = [
            {
                'nazwa': 'Targi Motoryzacyjne',
                'piątek': '14:00-20:00',  # montaż stoiska
                'sobota': '09:00-19:00',  # obsługa
                'niedziela': '10:00-18:00',  # obsługa + demontaż
                'godziny_total': 24
            },
            {
                'nazwa': 'Festiwal Piwa',
                'piątek': '16:00-23:00',
                'sobota': '12:00-23:00',
                'niedziela': '12:00-22:00',
                'godziny_total': 28
            }
        ]
        
        return {
            'liczba_eventów': len(events),
            'suma_godzin': sum(e['godziny_total'] for e in events),
            'wynagrodzenie': 'stawka dzienna 200-400 PLN',
            'dodatki': 'posiłki, transport, czasem nocleg'
        }
```

**Specjalne regulacje dla pracy weekendowej:**
```python
class RegulacjeWeekendowe:
    def zasady_handlu_niedziela(self):
        """Ograniczenia handlu w niedziele"""
        return {
            'niedziele_handlowe_2024': 7,  # tylko 7 w roku
            'wyjątki': [
                'ostatnia niedziela stycznia',
                'niedziela przed Wielkanocą',
                'ostatnia niedziela kwietnia',
                'ostatnia niedziela czerwca',
                'ostatnia niedziela sierpnia',
                'niedziela przed Wigilią',
                'ostatnia niedziela listopada'
            ],
            'pracownik_ma_prawo': {
                'do_2x_wynagrodzenia': True,
                'do_dnia_wolnego': 'w ciągu 6 dni',
                'odmowa_pracy': 'w wyjątkowych sytuacjach'
            }
        }
    
    def dodatki_weekendowe(self):
        return {
            'praca_w_niedzielę': '+100% podstawy',
            'praca_w_święto': '+100% podstawy',
            'lub_dzień_wolny': 'w zamian za święto/niedzielę'
        }
```

### 8. Skrócony tydzień pracy (art. 143 KP)
```python
def skrócony_tydzień():
    przykład_4_dni = 40 / 4  # = 10 godzin dziennie
    zachowanie_pełnego_etatu = True
    okres_rozliczeniowy = 1  # miesiąc
```

**Przykłady praktycznego zastosowania:**

```python
# PRZYKŁAD 1: Programista w trybie 4-dniowym
class Programista4Days:
    """Popularne w branży IT - work-life balance"""
    
    def harmonogram_4_10(self):
        """4 dni po 10 godzin"""
        return {
            'poniedziałek': {'start': '08:00', 'koniec': '18:30'},  # 10h + 0.5h przerwy
            'wtorek':       {'start': '08:00', 'koniec': '18:30'},  # 10h
            'środa':        {'start': '08:00', 'koniec': '18:30'},  # 10h
            'czwartek':     {'start': '08:00', 'koniec': '18:30'},  # 10h
            'piątek':       'WOLNE - długi weekend co tydzień',
            
            'zalety': [
                '3-dniowy weekend',
                'mniej dojazdów',
                'lepsza koncentracja - dłuższe bloki pracy',
                'oszczędność na kosztach biura'
            ],
            'wyzwania': [
                'dłuższe dni mogą być męczące',
                'koordynacja z klientami w piątki',
                'meetingi - wszyscy muszą być 4 dni'
            ]
        }
    
    def produktywność(self):
        """Badania pokazują często wzrost produktywności"""
        return {
            'głęboka_praca': '4 bloki po 2.5h dziennie',
            'mniej_meetingów': 'kompresja do 4 dni',
            'lepsza_regeneracja': '3 dni odpoczynku',
            'retention': 'benefit przyciągający talenty'
        }

# PRZYKŁAD 2: Pracownik produkcji - system 3x12h
class OperatorProdukcji3x12:
    """3 dni pracy, 4 dni wolnego"""
    
    def harmonogram_compressed(self):
        """Tydzień skompresowany"""
        tydzien_A = {
            'pon': 'wolne',
            'wt':  {'shift': '06:00-18:00'},  # 12h
            'śr':  {'shift': '06:00-18:00'},  # 12h
            'czw': {'shift': '06:00-18:00'},  # 12h
            'pt':  'wolne',
            'sob': 'wolne',
            'nd':  'wolne'
        }
        # 36h w tygodniu A
        
        tydzien_B = {
            'pon': {'shift': '06:00-18:00'},  # 12h
            'wt':  {'shift': '06:00-18:00'},  # 12h
            'śr':  {'shift': '06:00-18:00'},  # 12h
            'czw': {'shift': '06:00-18:00'},  # 12h
            'pt':  'wolne',
            'sob': 'wolne', 
            'nd':  'wolne'
        }
        # 48h w tygodniu B
        
        return {
            'średnia_2_tygodnie': (36 + 48) / 2,  # = 42h, ale...
            'średnia_miesięczna': 40,  # musi wyjść 40h
            'wyrównanie': '1 dzień wolny co 4 tygodnie'
        }

# PRZYKŁAD 3: Konsultant - elastyczny skrócony tydzień
class KonsultantFlex4:
    """Elastyczne 4 dni w tygodniu"""
    
    def warianty(self):
        return {
            'wariant_1': {
                'dni': 'pon-czw',
                'godziny': 10,
                'piątek': 'zawsze wolny'
            },
            'wariant_2': {
                'dni': 'wt-pt',
                'godziny': 10,
                'poniedziałek': 'zawsze wolny'
            },
            'wariant_3': {
                'dni': 'dowolne 4 dni',
                'godziny': 10,
                'wolny': 'ruchomy, do uzgodnienia'
            }
        }
    
    def przykład_elastyczny(self):
        """Różne dni wolne w różnych tygodniach"""
        miesiąc = {
            'tydzień_1': 'wolny piątek',
            'tydzień_2': 'wolna środa (sprawy osobiste)',
            'tydzień_3': 'wolny poniedziałek (długi weekend)',
            'tydzień_4': 'wolny piątek'
        }
        return miesiąc
```

**Podsumowanie wdrożenia skróconego tygodnia:**
```python
class WdrozenieSkroconego:
    def kroki(self):
        return [
            '1. Analiza możliwości organizacyjnych',
            '2. Konsultacje z pracownikami',
            '3. Okres próbny 3-6 miesięcy',
            '4. Modyfikacja regulaminu pracy',
            '5. Monitoring produktywności',
            '6. Ewaluacja i decyzja finalna'
        ]
    
    def kluczowe_metryki(self):
        return {
            'produktywność': 'czy nie spadła',
            'absencja': 'czy się zmniejszyła',
            'satysfakcja': 'ankiety pracownicze',
            'koszty': 'oszczędności na biurze i energii',
            'retention': 'czy zmniejszyła się rotacja'
        }
```

**Kiedy stosować skrócony tydzień:**
- Branża IT (programiści, testerzy, DevOps)
- Kreatywne (agencje, marketing, design)
- Konsulting (elastyczność projektów)
- Produkcja (system zmianowy skompresowany)
- Administracja (pilotaże work-life balance)

## Praca zdalna (nowelizacja z kwietnia 2023)

### Rozdział IIb KP (art. 6718-6724)

**Definicja** (art. 6718):
- Miejsce wskazane przez pracownika i uzgodnione z pracodawcą
- Całkowicie lub częściowo poza siedzibą firmy
- W szczególności z domu pracownika

**Rodzaje pracy zdalnej**:
```python
def rodzaje_pracy_zdalnej():
    return {
        'stała': "określona w umowie o pracę",
        'okazjonalna': "max 24 dni/rok kalendarzowy",
        'nakazana': "stan nadzwyczajny, epidemia, siła wyższa"
    }
```

**Zwrot kosztów** (art. 6723-6724):
```python
def kalkulacja_zwrotu_kosztów():
    # Energia elektryczna
    laptop_kwh = 0.0575  # kWh/godzina
    cena_prądu = 1.19  # PLN/kWh
    energia_miesięcznie = laptop_kwh * godziny_pracy * cena_prądu
    
    # Telekomunikacja
    internet_miesięcznie = (abonament * dni_pracy) / dni_w_miesiącu
    
    # Całkowity zwrot (zwolniony z PIT i ZUS)
    return energia_miesięcznie + internet_miesięcznie + amortyzacja_sprzętu
```

**Grupy uprzywilejowane** (art. 6719 §6-7):
- Pracownice w ciąży
- Rodzice dzieci do 4 lat
- Opiekunowie osób niepełnosprawnych
- Rodzice dzieci ze szczególnymi potrzebami

## Urlopy wypoczynkowe

### Wymiar urlopu (art. 152 KP)
```python
def oblicz_wymiar_urlopu(lata_pracy, wykształcenie):
    # Doliczanie lat za wykształcenie (art. 155)
    lata_edukacji = {
        'zawodowe_podstawowe': min(3, faktyczne_lata),
        'średnie_zawodowe': min(5, faktyczne_lata),
        'średnie_zawodowe_po_podstawówce': 5,
        'średnie_ogólnokształcące': 4,
        'policealne': 6,
        'wyższe': 8
    }
    
    # Wybór najkorzystniejszego okresu edukacji
    bonus_edukacyjny = max(lata_edukacji.values())
    
    całkowity_staż = lata_pracy + bonus_edukacyjny
    
    if całkowity_staż < 10:
        return 20  # dni
    else:
        return 26  # dni
```

### Urlop proporcjonalny
```python
def urlop_proporcjonalny(dni_bazowe, miesiące_pracy, wymiar_etatu=1.0):
    roczny_urlop = dni_bazowe * wymiar_etatu
    roczny_urlop = math.ceil(roczny_urlop)  # zaokrąglenie w górę
    
    if miesiące_pracy < 12:
        proporcjonalny = (roczny_urlop * miesiące_pracy) / 12
        return math.ceil(proporcjonalny)
    return roczny_urlop
```

### Pierwszy urlop (art. 153 KP)
```python
def pierwszy_urlop():
    # Po każdym przepracowanym miesiącu
    miesięczne_nabycie = dni_bazowe / 12
    # Przykład dla 20 dni: 1.67 dnia/miesiąc
```

### Ekwiwalent za niewykorzystany urlop (art. 171 KP)
```python
def ekwiwalent_urlopowy(niewykorzystane_dni, pensja_miesięczna):
    # Współczynnik roczny
    dni_kalendarzowe = 365
    niedziele = 52
    święta = 13  # średnio
    wolne_soboty = 52
    
    współczynnik = (dni_kalendarzowe - niedziele - święta - wolne_soboty) / 12
    stawka_dzienna = pensja_miesięczna / współczynnik
    
    return stawka_dzienna * niewykorzystane_dni
```

### Urlop na żądanie (art. 167² KP)
```python
def urlop_na_żądanie():
    roczny_limit = 4  # dni
    zgłoszenie = "najpóźniej w dniu rozpoczęcia"
    pracodawca_musi_udzielić = True  # chyba że wyjątkowe okoliczności
    przenoszenie_na_kolejny_rok = False
```

## Urlopy rodzicielskie i okolicznościowe

### Urlop macierzyński (art. 180-182¹ KP)
```python
def urlop_macierzyński():
    wymiar = {
        'pojedyncze_dziecko': 20,  # tygodni
        'bliźnięta': 31,
        'trojaczki': 33,
        'czworaczki': 35,
        'pięcioraczki_plus': 37
    }
    obowiązkowy_dla_matki = 14  # tygodni po porodzie
    możliwość_przekazania_ojcu = "po 14 tygodniu"
    zasiłek = "100% podstawy wymiaru"
```

### Urlop ojcowski (art. 182³ KP)
```python
def urlop_ojcowski():
    wymiar = 2  # tygodnie
    termin_wykorzystania = "do 12 miesiąca życia dziecka"
    podział = "1 lub 2 części (min. 1 tydzień każda)"
    zgłoszenie = "max 7 dni przed rozpoczęciem"
    zasiłek = "100% podstawy wymiaru"
```

### Urlop rodzicielski (art. 182¹ᵃ-182¹ᶠ KP)
```python
def urlop_rodzicielski():
    wymiar_standardowy = {
        'pojedyncze_dziecko': 41,  # tygodni
        'wieloraczki': 43
    }
    wymiar_niepełnosprawne = {
        'pojedyncze_dziecko': 65,
        'wieloraczki': 67
    }
    nieprzenoszone_na_drugiego_rodzica = 9  # tygodni dla każdego
    
    # Opcja zasiłkowa
    standardowa_stawka = 0.70  # 70% podstawy
    podwyższona_stawka = 0.815  # 81.5% przy wniosku w 21 dni od porodu
```

### Urlop wychowawczy (art. 186-186⁸ KP)
```python
def urlop_wychowawczy():
    maksymalny_wymiar = 36  # miesięcy
    indywidualny_limit = 35  # miesięcy (1 miesiąc dla drugiego rodzica)
    termin_wykorzystania = "do końca roku, w którym dziecko kończy 6 lat"
    niepełnosprawne_dziecko = "+36 miesięcy do 18 roku życia"
    status = "bezpłatny"
    ochrona_stosunku_pracy = True
```

### Urlop szkoleniowy (art. 103¹-103³ KP)
```python
def urlop_szkoleniowy():
    egzaminy_eksternistyczne = 6  # dni
    praca_dyplomowa_i_obrona = 21  # dni w ostatnim roku
    wynagrodzenie = "100% zachowane"
```

### Urlopy okolicznościowe (Rozporządzenie MPiPS z 15 maja 1996)
```python
def urlopy_okolicznościowe():
    return {
        'własny_ślub': 2,  # dni płatne
        'narodziny_dziecka': 2,  # dni płatne
        'śmierć_współmałżonka_dziecka_rodzica': 2,  # dni płatne
        'śmierć_rodzeństwa_dziadków_teściów': 1,  # dzień płatny
        'siła_wyższa': 2,  # dni lub 16 godzin rocznie (50% płatne)
        'opieka_nad_dzieckiem': 2,  # dni lub 16 godzin (100% płatne)
    }
```

## Praca w godzinach nadliczbowych

### Definicja i limity
```python
def nadgodziny():
    definicja = "praca ponad 8h/dzień LUB średnio 40h/tydzień"
    limit_roczny = 150  # godzin (może być zwiększony w układzie zbiorowym)
    limit_tygodniowy_z_nadgodzinami = 48  # średnio w okresie rozliczeniowym
```

### Algorytm wynagrodzenia za nadgodziny
```python
def wynagrodzenie_nadgodziny(stawka_godzinowa, liczba_nadgodzin):
    pierwsze_2_godziny = min(2, liczba_nadgodzin) * stawka_godzinowa * 1.5
    kolejne_godziny = max(0, liczba_nadgodzin - 2) * stawka_godzinowa * 2.0
    
    return pierwsze_2_godziny + kolejne_godziny
```

### Czas wolny za nadgodziny
```python
def czas_wolny_rekompensata(nadgodziny):
    czas_wolny = nadgodziny * 1.5  # godziny
    termin_udzielenia = "w okresie rozliczeniowym lub kolejnym"
    if not wykorzystany:
        wypłata = wynagrodzenie_nadgodziny()
```

## Praca nocna (art. 151⁷-151⁸ KP)

### Definicja i dodatki
```python
def praca_nocna():
    pora_nocna = "8 godzin między 21:00 a 7:00"  # pracodawca ustala
    pracownik_nocny = "min 3h nocą LUB 1/4 okresu rozliczeniowego nocą"
    
    dodatek_nocny = 0.20 * minimalne_wynagrodzenie_godzinowe * godziny_nocne
    
    zakazy = {
        'kobiety_w_ciąży': "bezwzględny",
        'nieletni': "bezwzględny",
        'rodzice_dzieci_do_8_lat': "wymagana zgoda"
    }
```

## Okresy odpoczynku

### Odpoczynek dobowy i tygodniowy
```python
def okresy_odpoczynku():
    dobowy = 11  # godzin nieprzerwanych na dobę
    tygodniowy = 35  # godzin nieprzerwanych (zawiera dobowy)
    
    # Wyjątki
    po_dyżurze_16h = 16  # godzin odpoczynku
    po_dyżurze_24h = 24  # godziny odpoczynku
    
    niedziela_wolna = "co najmniej raz na 4 tygodnie"
```

## Sankcje i kary

### Hierarchia kar (art. 281 KP + 218 KK)
```python
def sankcje_pracodawcy():
    naruszenia_administracyjne = {
        'brak_ewidencji': (1000, 30000),  # PLN
        'nieprawidłowe_urlopy': (1000, 30000),
        'naruszenie_odpoczynku': (1000, 30000),
        'przekroczenie_nadgodzin': (1000, 30000)
    }
    
    mandat_karny_PIP = (1000, 2000)  # PLN
    recydywa_2_lata = 5000  # PLN max
    
    odpowiedzialność_karna = {
        'uporczywe_naruszenia': "min. 3 kolejne naruszenia miesięcznych obowiązków",
        'złośliwe_naruszenia': "nawet jednorazowe z zamiarem szkodzenia",
        'kara': "grzywna, ograniczenie wolności lub do 2 lat pozbawienia wolności"
    }
```

### Terminy przedawnienia
```python
def przedawnienie():
    roszczenia_pracownicze = 3  # lata od daty wymagalności
    urlop_zaległy = "do 30 września kolejnego roku"
    roszczenie_o_urlop = 3  # lata od nabycia prawa
```

## Kontrole PIP

### Uprawnienia inspektorów
```python
def kontrola_PIP():
    uprawnienia = [
        "niezapowiedziane kontrole 24/7",
        "dostęp do dokumentacji",
        "przesłuchania pracowników",
        "nakazy natychmiastowe",
        "mandaty karne"
    ]
    
    najczęstsze_naruszenia = [
        "brak godzin rozpoczęcia/zakończenia pracy",
        "nieprawidłowe okresy odpoczynku",
        "brak dokumentacji nadgodzin",
        "naruszenie 5-dniowego tygodnia pracy"
    ]
```

## Wymagania implementacyjne dla systemu

### Struktura danych podstawowych
```sql
CREATE TABLE ewidencja_czasu (
    id BIGSERIAL PRIMARY KEY,
    pracownik_id INTEGER NOT NULL,
    data DATE NOT NULL,
    typ_umowy VARCHAR(50) NOT NULL,
    czas_rozpoczecia TIMESTAMP,
    czas_zakonczenia TIMESTAMP,
    godziny_przepracowane DECIMAL(4,2),
    nadgodziny_50 DECIMAL(4,2),
    nadgodziny_100 DECIMAL(4,2),
    praca_nocna DECIMAL(4,2),
    praca_niedziela BOOLEAN,
    praca_swieto BOOLEAN,
    dyzur_godziny DECIMAL(4,2),
    dyzur_lokalizacja VARCHAR(200),
    nieobecnosc_typ VARCHAR(50),
    nieobecnosc_wymiar DECIMAL(4,2),
    utworzono_timestamp TIMESTAMP DEFAULT NOW(),
    INDEX idx_pracownik_data (pracownik_id, data)
);
```

### Walidacje systemowe
```python
def walidacje_systemowe():
    return {
        'odpoczynek_dobowy': lambda h: h >= 11,
        'odpoczynek_tygodniowy': lambda h: h >= 35,
        'nadgodziny_dzienne': lambda h: h <= 8,  # ponad normę
        'nadgodziny_roczne': lambda h: h <= 150,
        'urlop_na_zadanie_roczny': lambda d: d <= 4,
        'praca_zdalna_okazjonalna': lambda d: d <= 24,
        'pierwszy_urlop': lambda m: m * (20/12 or 26/12),
        'urlop_niewykorzystany': lambda data: data <= "30-09-następny-rok",
        'dokumentacja_przechowanie': lambda lata: lata >= 10
    }
```

### Algorytmy obliczeniowe - podsumowanie
```python
class KalkulatorCzasuPracy:
    def oblicz_wymiar_pracy(self, okres_rozliczeniowy):
        """Oblicza wymiar czasu pracy w okresie"""
        tygodnie = okres_rozliczeniowy.tygodnie
        dni_robocze = okres_rozliczeniowy.dni_pon_pt
        święta_niedziela = okres_rozliczeniowy.święta_poza_niedziela
        
        return (40 * tygodnie) + (8 * dni_robocze) - (8 * święta_niedziela)
    
    def oblicz_nadgodziny(self, godziny_pracy, system_pracy):
        """Kalkuluje nadgodziny według systemu"""
        if system_pracy == 'podstawowy':
            dzienny_limit = 8
            tygodniowy_limit = 40
        elif system_pracy == 'równoważny':
            dzienny_limit = 12
            tygodniowy_limit = 40
        # ... pozostałe systemy
        
        return self._calculate_overtime(godziny_pracy, dzienny_limit, tygodniowy_limit)
    
    def oblicz_urlop_proporcjonalny(self, dni_bazowe, miesiące, etat):
        """Urlop proporcjonalny do wymiaru i czasu"""
        return math.ceil((dni_bazowe * etat * miesiące) / 12)
```

### Integracje wymagane
```python
def integracje_systemowe():
    return {
        'ZUS': "eksport danych o czasie pracy i absencjach",
        'Płace': "automatyczne naliczanie nadgodzin i dodatków",
        'Kadry': "synchronizacja danych pracowniczych",
        'BHP': "monitoring przekroczeń norm czasu pracy",
        'Kontrola_dostępu': "import danych wejść/wyjść",
        'Kalendarz': "planowanie urlopów i nieobecności"
    }
```

## Podsumowanie kluczowych wymogów

### Minimalne wymagania systemu ewidencji
1. **Rejestracja precyzyjna**: Godziny rozpoczęcia i zakończenia z dokładnością do minuty
2. **Różnicowanie systemów**: Obsługa wszystkich 8+ systemów organizacji czasu
3. **Automatyczne wyliczenia**: Nadgodziny, dodatki nocne, urlopy proporcjonalne
4. **Walidacje prawne**: Kontrola limitów i okresów odpoczynku
5. **Retencja danych**: 10-letnie przechowywanie z pełną integralnością
6. **Dostępność**: Pracownik musi mieć dostęp do swoich danych
7. **Audyt**: Ślad rewizyjny wszystkich zmian i dostępów
8. **Raporty kontrolne**: Gotowość na kontrolę PIP w każdej chwili

### Priorytety wdrożeniowe

*Poniższa lista priorytetów pozwala zaplanować wdrożenie systemu w sposób minimalizujący ryzyko prawne. Funkcjonalności oznaczone jako KRYTYCZNE muszą być gotowe przed uruchomieniem systemu, podczas gdy elementy o niższym priorytecie mogą być dodawane w kolejnych iteracjach.*

1. **KRYTYCZNE**: Ewidencja czasu dla umów o pracę z walidacjami
2. **WYSOKI**: Kalkulatory urlopowe i nadgodzin
3. **ŚREDNI**: Obsługa pracy zdalnej i systemów specjalnych
4. **NISKI**: Zaawansowana analityka i predykcje

Wszystkie powyższe wymagania oparte są wyłącznie o oficjalne źródła rządowe (gov.pl, pip.gov.pl, dziennikustaw.gov.pl) i obowiązujące akty prawne.