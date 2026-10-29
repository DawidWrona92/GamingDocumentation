---
name: "paintedart-llm"
title: "Painted art dla agentów LLM: ciągły ton bez pętli generatywnej"
type: "text/markdown"
---

# Painted art dla agentów LLM: ciągły ton bez pętli generatywnej

**Status:** wersja 1.0, walidacja wtórna zakończona, dokument kompletny pod względem źródeł i narzędzi, gotowy do wrzucenia do paczki jako materiał uniwersalny dla dowolnego agenta. Zakres: trasa malowania ciągłego tonu (painted: gradienty, tekstury, miękkie cienie, brak siatki piksela) dla ilustracji, assetów gier i spriteów w rozmiarach od kilkuset pikseli wzwyż. Trzeci dokument tryptyku, komplementarny do przewodnika pixel artu rysowanego i do pipeline pre-renderu 3D. Uwaga o atraktorach: domyślnym atraktorem tej trasy jest generator obrazowy, wygląda jak najkrótsza droga do painted looku, a w pętli retake'ów jest ślepą ulicką (mechanizm w sekcji 2); dokument traktuje generator wyłącznie jako źródło materiału, nigdy jako proces produkcji. Reguły wspólne z dokumentów poprzednich (sylwetka, test skali, raport liczb przed/po, dwa przebiegi, filmstrip zamiast GIF) obowiązują bez zmian.

## Pytanie badawcze

Jak agent LLM ma produkować painted art: obrazy i sprite'y o ciągłym tonie, z teksturami, miękkimi cieniami i światłem rzeźbionym walorami, w sposób precyzyjny, powtarzalny i skończony czasowo. Kontekst problemu: generatory obrazowe (klasy GPT-image, Midjourney, Stable Diffusion) dają malowany wygląd od pierwszego użycia, ale nie pozwalają sterować współrzędnymi, a każda kolejna edycja w pętli dryfuje od celu i niszczy to, co było już dobre; pętla „generuj aż wyjdzie" nie ma gwarancji zbieżności i jest katastrofalna czasowo. Konkretne cele: (a) zrozumieć mechanizm tej pułapki, (b) zebrać wzorce z produkcji, które painted look osiągały deterministycznie (malowane gry komercyjne), (c) przenieść je na pipeline agenta z bramkami liczbowymi, (d) określić, kiedy i jak używać generatora obrazowego tak, żeby nie wpadł w pętlę.

## Streszczenie wykonawcze

1. **Painted art to dla agenta inna trasa niż pixel art, ale ta sama strategia: agent pisze program.** W pixel arcie program działa na siatce i paletach, w painted art program maluje: kształty wektorowe, gradienty walorów, szum tekstur, miękkie cienie, pociągnięcia pędzli. Wynik jest deterministyczny, edycja jest punktowa (zmiana parametru), nic nie regeneruje się samo. Strategia z dokumentów poprzednich przenosi się w całości, zmienia się tylko materiał rysunkowy.
2. **Pułapka pętli generatywnej ma udokumentowany mechanizm, to nie jest subiektywne wrażenie.** Generatory modelowe nie potrafią mapować współrzędnych liczbowych na płótno (MetaPoint, 2026), edycje wielokrotne kumulują własne błędy, bo model trenowany na pojedynczych parach wejście→wyjście w inferencji dostaje własne niedoskonałe wyniki (exposure bias, MT Forcing: średni dryf pikselowy 6,34 px na ośmiokrokowych sesjach edycyjnych, zredukowany do 0,74 px dopiero specjalnym treningiem), a „atraktory" to rozkład treningowy, który każdą iterację ciągnie z powrotem ku kliszom. Każdy retake w generatorze regeneruje cały obraz, więc decyzja poprawiona w iteracji 3 ginie w iteracji 4.
3. **Zasada kardynalna trasy: struktura deterministyczna, malowanie proceduralne, generator tylko jako materiał.** Końcówka produkcyjna nigdy nie jest pętlą generacyjną. Retake to edycja parametru, warstwy albo obiektu w programie malującym, nie „wygeneruj jeszcze raz, może będzie lepiej". Jeśli generator musi być użyty, to na wejściu (tekstura, koncepcja) albo w chirurgicznej edycji z maską i weryfikacją pikselową poza maską.
4. **Malowany styl ma udokumentowane komercyjne wzorce, każde z przenośną lekcją.** Curse of Monkey Island: tła malowane w całości cyfrowo, szkice na papierze, 640×480 przy wciąż 256 kolorach; charakter niosła rozdzielczość (linie tuszu widoczne dopiero w wysokiej rozdzielczości), nie liczba kolorów. Battle Brothers: z rezygnacji z animacji i pełnych figurek zrobioną metodę: malowane popiersia z warstwami wymiennymi (głowa, hełm, broń, pancerz, warianty uszkodzeń). Rayman Origins i UbiArt: koncepcja graficzna wprost jako asset, stylizacja obniża koszt (im bardziej realistyczny design, tym więcej rysunków potrzeba, by trzymać się rzeczywistości). Hollow Knight: cała grafika malowana w Photoshopie, bez specjalistycznego oprogramowania, dyscyplina walorów zamiast narzędzi.
5. **Odpowiednikiem siatki piksela jest tu plan walorów.** Trzy, cztery rodziny walorów (światło, mid, cień), squint test, dominanta 70/30, audyt w skali szarości: to standard rzemiosła malarskiego i to wszystko da się skryptować: kwantyzacja luminancji, histogramy, próg „szarej zupy". Obraz, który po zredukowaniu do 4 wartości przestaje się czytać, jest zły niezależnie od liczby detali.
6. **Malarstwo proceduralne ma 35 lat badań i gotową recepturę kolejności.** Stroke-based rendering od Haeberliego (1990) i Hertzmanna (1998): pociągnięcia rozmieszcza się według mapy różnicy względem celu, warstwami od grubych pędzli do cienkich, zakrzywia je według gradientu obrazu. To wprost kolejność pracy dla agenta: sylwetka → walory → kolor → tekstura → detale, a gęstość detalu rośnie tylko tam, gdzie niesie czytelność.
7. **Ekosystem narzędzi pozwala zrealizować trasę już dziś.** Krita od wersji 5.3 ma Python API do malowania pociągnięć pędzli (prawdziwy silnik malarski sterowany skryptem), Inkscape działa z linii poleceń na plikach SVG, LLM pisze SVG bezpośrednio (filtrami feTurbulence i feGaussianBlur uzyskuje się teksturę i miękkie cienie), a badane modele (IconShop, StarVector) generują wektory z tekstu i z obrazu. Spellburst ze Stanforda pokazuje wzorzec interfejsu: suwaki parametrów zamiast repromptowania.
8. **Gdy generator obrazowy jest niezbędny, są reguły minimalizujące dryf.** Lokalny Stable Diffusion z zapisanym seedem, inpainting zamiast regeneracji (maska trzyma resztę obrazu w każdym kroku denoisingu, dosłownie, a nie „podobnie"), denoise 0,4-0,6 przy edycji twarzy, feather maski 12-16 px przy 1024 px, ControlNet i IP-Adapter dla spójności postaci. To nadal materiał i wypełnienia, nie końcówka.

## Metodologia

- Kąty wyszukiwania: (1) mechanizmy ograniczeń generatorów obrazowych (kontrola przestrzenna, dryf edycji wielokrotnych, praktyka inpaintingu), (2) produkcja malowanych gier komercyjnych (wywiady z artystami, dev blogi, GDC), (3) malarstwo proceduralne i stroke-based rendering (badania od 1990 do synergii z LLM), (4) wektory i kod jako medium dla LLM (SVG, programy malujące, ewolucja kodu), (5) reguły rzemiosła malarskiego nadające się na bramki liczbowe (walory, kompozycja), (6) narzędzia skryptowalne (Krita, Inkscape, ComfyUI).
- Typy źródeł: primary od twórców (dev blog Battle Brothers, wywiad z Billem Tillerem, FAQ Team Cherry, wypowiedzi Ancela), dokumentacja narzędzi (release notes Krity, dokumentacja Pythona Krity, man Inkscape), prace badawcze (MetaPoint, MT Forcing, przegląd stroke-based rendering, IconShop, StarVector, Spellburst, AI Co-Artist), poradniki rzemieślnicze (walory, squint test), praktyka społeczności (inpainting, ControlNet, ograniczenia GPT-image-1).
- Ograniczenia: liczby MT Forcing pochodzą z explainera pracy SenseTime, benchmark jest autorski (ten sam zespół); twierdzenia Ideogramu o braku dryfu to claim sprzedawcy; badania stroke-based rendering dotyczą renderowania z fotografii, przeniesienie na tworzenie od zera jest adaptacją; IconShop i StarVector operują na ikonach i pojedynczych obiektach, nie pełnych ilustracjach; progi liczbowe bramek walorów (udział midtonów, dominanta 70/30) są propozycją wyprowadzoną z poradników malarskich, nie normą z badań.

## Wyniki

### 1. Definicja trasy i różnica wobec dwóch poprzednich

Painted art w tym dokumencie to obraz o ciągłym tonie: gradienty i przejścia zamiast płaskich płaszczyzn, tekstura zamiast gładkiego wypełnienia, miękkie krawędzie mile widziane, rozmiary od kilkuset pikseli wzwyż, paleta nieograniczona liczbowo. Czytelność niosą walory i kompozycja, nie pojedyncze piksele.

| Aspekt | Pixel art rysowany | Pre-render 3D | Painted art |
| --- | --- | --- | --- |
| Medium | siatka piksela, limit kolorów | geometria 3D, kamera orto | kształt, walor, tekstura |
| Źródło spójności | reguły pikselowe | model i światło sceny | plan walorów i warstwy |
| Miękkie krawędzie | artefakt | usuwane przy downscale | zamierzony efekt |
| Retake | przerysowanie pikseli | przesunięcie keyframes | edycja parametru lub warstwy |
| Główna bramka | mixele, paleta, sylwetka | wobble, alfa, światło | rodziny walorów, squint, strefy czytelności |

Wspólna infrastruktura przenosi się bez zmian: sylwetka oceniana w skali docelowej, test jednego assetu całą trasą zanim ruszy zestaw, raport liczb przed/po, minimum dwa przebiegi pętli poprawek, przegląd filmstripem, nie GIF.

### 2. Pułapka pętli generatywnej: mechanizm i liczby

**Brak kontroli współrzędnych.** Modele generatywne przetwarzają tekstowy opis przestrzeni, ale nie potrafią zmapować współrzędnych liczbowych na płótno; MetaPoint (2026) cytuje to wprost jako fundamentalną słabość i buduje wokół niej metodę: współrzędna jako specjalny token, pozycja obiektu jednym tokenem, bounding box dwoma. Wniosek dla agenta: prośba „daj cień 12 px w lewo pod krawędzią" jest dla generatora poza zakresem możliwości, jest w zakresie możliwości programu malującego.

**Dryf edycji wielokrotnych.** Model edytujący obraz jest trenowany na pojedynczych parach (źródło, instrukcja, cel); w pętli retake'ów na wejściu dostaje własne wyniki z poprzedniej iteracji, wraz z błędami. Błędy się kumulują: tekstury degradują, regiony, które miały zostać nietknięte, się zmieniają, detale giną. Praca MT Forcing (SenseTime, HKUST, Stanford, Tsinghua, 2026) kwantyfikuje to na 100 sesjach ośmiokrokowych: średni dryf pikselowy 6,34 px, spójność treści 7,09/10; specjalny trening (student edytuje własną historię, nauczyciel liczy całość z oryginału jednym przejściem) zredukował dryf do 0,74 px i podniósł spójność do 8,40/10. Dwie rzeczy z tego wynikają: dryf jest realny i mierzalny w standardowych modelach, a jego redukcja wymaga specjalnego treningu, którego domyślny generator nie przeszedł.

**Atraktor jako dryf semantyczny.** Każde ponowne wygenerowanie losuje z rozkładu treningowego, który ma swoje klisze (kolor, kompozycja, typowy detal). Iterując prompt albo edycję, obraz dryfuje ku średniej rozkładu, czyli ku kliszy, a nie ku celowi projektowemu. To dokładnie ten sam mechanizm, który w dokumentach pixelartowych opisywaliśmy przy atraktorach stylu (Dead Cells wygląda jak pixel art, ale jest pipeline 3D), tu tylko działa w czasie: pętla iteracji jest atraktorem.

**Dowody z praktyki.** W przemyśle reklamowym opisuje się to jako regułę dwóch edycji: pierwsze dwie poprawki wychodzą dobrze, szósta edycja ma przesunięte twarze i „plastikowe" tekstury. Użytkownicy API GPT-image-1 raportują problemy z edycją maskową (zmiany wychodzące poza maskę, utrata detali). Ideogram robi z „edits that don't drift" slogan marketingowy produktu, czyli rynek sam potwierdził, że dryf to rozpoznany problem klientów.

**Rachunek czasowy.** Pętla generacyjna: każdy retake to pełna regeneracja i pełna rewalidacja wszystkiego, co było już zaakceptowane; koszt rośnie liniowo z liczbą poprawek, a szansa na jednoczesne spełnienie wszystkich wymagań maleje wykładniczo. Trasa programistyczna: retake to zmiana jednego parametru, koszt poprawki jest stały i lokalny. To różnica między O(n) ze spadającą jakością a O(1) z jakością rosnącą.

### 3. Malowane wzorce komercyjne i ich lekcje

**Curse of Monkey Island (1997), Bill Tiller, lead background artist.** Wszystkie tła malowane cyfrowo (Painter i Photoshop), na papierze powstawały tylko rysunki; szkicowy charakter linii tuszu i ołówka był możliwy, dopiero gdy rozdzielczość wzrosła z 320×200 do 640×480, przy czym paleta pozostała taka sama: 256 kolorów. Tiller wprost: dwie sąsiadujące piksele w wysokiej rozdzielczości zlewają się w oku w nowy kolor, którego nie ma w palecie, więc nie potrzeba było więcej kolorów, tylko wyższej rozdzielczości. Inspiracje malarstwem (NC Wyeth, Howard Pyle, Peter De Seve) wskazują kierunek stylizacji. Lekcja dla agenta: charakter painted looku niosą rozdzielczość i linia oraz rzemiosło walorów, nie rozmiar palety; szkic-rysunek struktury przed malowaniem ma udokumentowaną rolę produkcyjną.

**Battle Brothers (2017), Overhype Studios.** Zespół trzech osób zaczął od pełnych figurek malowanych i animowanych we Flashu, uznał to za nierealizowalne kosztowo i przeszedł na malowane popiersia na cokołach, bez animacji, za to z pełnym ekwipunkiem widocznym na postaci: każda część (głowa, hełm, broń, pancerz) jest osobną warstwą, wymienną między postaciami, z wariantami uszkodzeń dla każdego typu pancerza. Cokoły pomagają postaci odcinać się od tła, ale zaprojektowano je niskie i stonowane, bo zbyt masywne zabierają uwagę i mylą poziomy wysokości w grze. Lekcja dla agenta: warstwowość i modułowość to sposób na malowany koszt; rezygnacja z animacji to legalna decyzja projektowa, nie porażka; czytelność rozwiązuje się tłem i oprawą cokołu, nie gęstszym detalem.

**Rayman Origins (2010), Ubisoft Montpellier, silnik UbiArt.** Koncepcja graficzna malowana ręcznie była digitalizowana w wysokiej rozdzielczości i używana bezpośrednio jako asset tła albo animowana grafika postaci; system zbudowano, by mały zespół produkował grafikę gameplayową szybko. Christophe Villez na GDC: im bardziej realistyczny design, tym więcej rysunków potrzeba, żeby animacja trzymała się rzeczywistości; styl kartunowy (inspiracje: Ren i Stimpy) obniża ten koszt radykalnie. Lekcja dla agenta: stylizacja to optymalizacja kosztu, nie kompromis; i: granica koncepcja→asset może być krótka, jeśli format pośredni (warstwy) jest zaplanowany.

**Hollow Knight (2017), Team Cherry.** Cała grafika, łącznie z animacjami, malowana w Photoshopie; gra zbudowana w Unity na gotowych rozszerzeniach. Lekcja: painted look nie wymaga specjalistycznego oprogramowania, wymaga dyscypliny walorów i spójności stylistycznej trzymanej ręcznie.

Wnioski przekrojowe: (a) żadna z tych produkcji nie oszczędzała przez rezygnację z malowanego charakteru, oszczędzały przez warstwy, moduły, stylizację i świadome cięcia zakresu; (b) kolejność pracy była wszędzie ta sama: struktura (szkic, sylwetka), potem walory, potem kolor, detale na końcu; (c) malowany charakter to odpowiedzialność planu, nie szczęśliwego trafienia.

### 4. Pipeline malowania proceduralnego krok po kroku

Krok 0, brief (raz na zestaw): anchor stylu (dokumentacja z sekcji 3 albo własna biblia stylu), rozmiar docelowy, liczba kierunków jeśli sprite, spis warstw (cień odbity, okluzja, korpus, światło, tekstura, detale), plan walorów: trzy albo cztery rodziny, dominanta, punkt ogniskowy, lista akcji przed rigowaniem jeśli animowany.

Krok 1, struktura: sylwetka jako kształt wektorowy, oceniana w skali docelowej (test z dokumentu rysunkowego, obowiązujący wprost), brak stycznych, czytelność konturu. To jest odpowiednik modelu z trasy pre-renderu: najtańsze miejsce na poprawkę.

Krok 2, walory w skali szarości: podział na rodziny (światło, mid, cień; ewentualnie cztery), squint test skryptowy (sekcja 6), kierunek światła ustalony raz na cały zestaw. Zasada malarska przeniesiona na pipeline: dodanie detalu do złych walorów buduje na wadliwym fundamencie, detale na końcu.

Krok 3, kolor: hue i nasycenie w rodzinach walorów; w painted nie limituje się liczby kolorów, limituje się liczbę rodzin hue i zachowanie logiki światła (cień chłodniejszy, światło cieplejsze to domyślna, nie prawo). Audyt: obraz w skali szarości dalej czyta się poprawnie, kolor niczego nie ratuje.

Krok 4, bryła i miękkie cienie: przejścia walorów przez gradienty i maski, blur kontrolowany promieniem w pikselach, cienie rzucane jako osobna warstwa z własną przezroczystością. Konwencja światła jak w dokumencie pre-renderu: albo obraca się z modelem, albo stoi w scenie, nigdy mieszane w zbiorze.

Krok 5, tekstura: proceduralny szum (value noise, fBm o 2-4 oktawach, w SVG filtr feTurbulence), nałożony jako nakładka z kontrolowaną alfa, moc ograniczona w strefach czytelności (twarz, punkt ogniskowy, krawędzie sylwetki). Tekstura globalna o jednolitej mocy to odpowiednik zupy kolorów z pixel artu.

Krok 6, detale: od dużych do małych (recepta z sekcji 5), tylko w strefach, które niosą czytelność; ostatnie 10% detali kosztuje najwięcej, a w rozmiarze docelowym często znika.

Krok 7, bramki (sekcja 6) i eksport: arkusz z warstwami osobno (cień osobno od korpusu, jak w Battle Brothers), metadane kierunków i pivotów jeśli sprite, plik źródłowy programu malującego w repozytorium jako element assetu.

### 5. Stroke-based rendering: 35 lat receptury

Haeberli (Paint by Numbers, SIGGRAPH 1990) zapoczątkował podejście: obraz powstaje z rozmieszczonych pociągnięć, których kolor i pozycja są próbkowane z obrazu źródłowego. Hertzmann (SIGGRAPH 1998) dodał pociągnięcia zakrzywiane według gradientu obrazu i wielkość zależną od lokalnego szczegółu, malowane warstwami od grubych do cienkich. Przegląd z 2023 (Stroke-based Rendering: From Heuristics to Deep Learning) porządkuje pole: algorytmy zachłanne (pociągnięcie po pociągnięciu według mapy błędu) i oparte na optymalizacji (funkcja energii mierzy dopasowanie do celu, parametry pociągnięć są minimalizowane), a nowsze prace uczą malować agentów RL z różniczkowalnym renderowaniem wektorowym.

Przeniesienie na agenta, bez fotografii jako wejścia: (a) kolejność warstw od grubego do cienkiego jest wprost kolejką zadań agenta; (b) gęstość detalu skaluje się z lokalnym kontrastem: tam gdzie kontrast niski, pociągnięcia grube i rzadkie, tam gdzie wysoki, cienkie i gęste; (c) mapy różnicy między aktualnym obrazem a celem to gotowa bramka stopu (pętla malowania kończy się, gdy poprawa spada poniżej progu, a nie „gdy wygląda dobrze"); (d) agent malujący programem ma tę samą architekturę co DRL painterzy z badań, tylko funkcją decyzyjną jest LLM z regułami zamiast wytrenowanej sieci, a to jest wygodniejsze, bo reguły da się czytać i zmieniać.

### 6. Bramki i artefakty trasy painted

- **Squint test skryptowy:** downscale ×8, rozmycie, kwantyzacja luminancji do 4 poziomów; obraz ma się nadal czytać jako wzór dużych mas. Odpowiednik patrzenia przez przymrużone powieki; egzamin, czy struktura istnieje w ogóle.
- **Szara zupa (fail):** ponad 60% pikseli w przedziale midtonów (luminancja 40-70%) i brak wyraźnych rodzin; obraz dokładny i martwy jednocześnie.
- **Dominanta 70/30:** udział największej rodziny walorów 60-80% obszaru; jeśli wszystko jest równe, nie ma hierarchii; jeśli dominanta bliska 100%, obraz jest płaski.
- **Test miniatury:** obraz w rozmiarze znaczka pocztowego ma zachować punkt ogniskowy; odpowiednik testu 1× z pixel artu.
- **Strefy czytelności:** lokalna wariancja (szum tekstury) w twarzy i punkcie ogniskowym poniżej progu; tekstura nie może konkurować z informacją.
- **Spójność zestawu:** histogram walorów i hue między assetami zbioru w tolerancji; dryf między assetami to painted odpowiednik przełączenia światła z dokumentu pre-renderu.
- **Bramka edycji generatorowej (gdy użyty):** diff pikselowy poza maską inpaintingu równy zero; wszystko poza maską nietknięte, inaczej edycja odrzucona.

Kolejność napraw: struktura → walory → kolor → tekstura → detale, na warstwach, raport liczb przed/po. Przegląd agenta: filmstrip i pasy wersji obok siebie (porównanie przed i po w jednym pasie pokazuje, czy poprawka nie zepsuła tego, co było dobrze).

### 7. Krajobraz narzędzi dla agentów (trasa painted)

| Narzędzie / skill | Co robi | Uwagi |
| --- | --- | --- |
| Krita + Python (API pędzli od 5.3) | Prawdziwy silnik malarski skryptowany: pociągnięcia pędzli, warstwy, filtry, eksport | Najbliższy „pędzel dla agenta"; brak makrorekordera (od 3.0 wyłączony), pluginy GPL |
| Inkscape CLI | Operacje wsadowe na SVG z linii poleceń: konwersje, akcje, render | Do pipeline bez GUI, wektor jako format pośredni |
| SVG pisany przez LLM + filtry | Kształty, gradienty, feTurbulence (tekstura), feGaussianBlur (miękkie cienie) | Zero zależności, w pełni deterministyczne, edytowalne tekstem |
| IconShop (badanie) | Ikony wektorowe z tekstu, autoregresyjnie, tokeny ścieżek SVG | Ikony, nie ilustracje; materiał do adaptacji |
| StarVector (badanie) | Kod SVG z obrazu rastrowego | Trasa odwrotna: rysunek→wektor do dalszej edycji kodem |
| Spellburst (Stanford) | LLM generuje kod, suwaki parametrów zamiast repromptowania, scalanie wersji | Wzorzec interfejsu: retake to suwak, nie regeneracja |
| AI Co-Artist (badanie) | Ewolucja kodu shaderów GLSL z selekcją LLM | Dowód, że program i ewolucja zastępują generator obrazowy |
| ComfyUI + MCP | Lokalny Stable Diffusion sterowany z rozmowy: seed, inpainting, ControlNet, IP-Adapter | Materiał i wypełnienia; seed zawsze zapisany |
| SD inpainting (przewodniki) | Maska trzyma resztę obrazu; denoise 0,4-0,8; twarze 0,4-0,5; feather 12-16 px @1024 | Zamiast regeneracji całości; bramka diff poza maską obowiązkowa |

### 8. Blueprint: skille agenta dla trasy painted

Architektura (zgodna z dokumentami poprzednimi):

1. **Plik reguł trasy:** sekcje 4-6 jako reguły z liczbami (rodziny walorów, dominanta, progi tekstury w strefach, kolejność warstw).
2. **Brief z planem walorów:** anchor stylu, warstwy, rodziny walorów i hue, dominanta, strefy czytelności, 2-6 kryteriów ukończenia.
3. **Bramki per warstwa:** struktura (test sylwetki), walory (squint skrypt, szara zupa), kolor (audyt grayscale), tekstura (wariancja w strefach), całość (test miniatury, spójność zestawu).
4. **Pętla maluj → spójrz → popraw:** poprawka przez edycję parametrów i warstw (wzorzec suwaków Spellburst), maksimum dwa przebiegi na warstwę przed eskalacją do człowieka.
5. **Reguła generatora:** image-gen tylko jako (a) materiał tekstury/koncepcji na wejściu albo (b) edycja z maską z bramką diff poza maską; seed, prompt i maska archiwizowane przy każdym użyciu; zero pętli generacyjnych na finalce.
6. **Eksport warstwowy:** cień, okluzja, korpus, tekstura, detale osobno; program malujący w repozytorium assetu.

Gotowy zestaw reguł do wklejenia w system prompt lub plik skilla (skondensowany):

- Painted to trasa programistyczna: piszesz program malujący (SVG, Canvas, Krita API), nie prosisz generatora o obraz; wynik musi być odtwarzalny z kodu.
- Plan walorów przed kolorem: 3-4 rodziny, dominanta 60-80%, kierunek światła jeden na cały zestaw; squint skrypt (downscale, blur, 4 poziomy) jako pierwsza bramka.
- Kolejność: sylwetka → walory → kolor → tekstura → detale; detale tylko w strefach czytelności; tekstura z alfa kontrolowaną, nie globalna.
- Retake to edycja parametru albo warstwy, nie regeneracja; po każdej poprawce diff względem wersji zaakceptowanej: nic poza celem poprawki nie może się zmienić.
- Generator obrazowy: tylko materiał na wejściu albo inpainting z maską; denoise 0,4-0,6 przy twarzach, feather 12-16 px @1024; poza maską zero pikseli różnicy; seed i prompt w archiwum.
- Między assetami: histogramy walorów i hue w tolerancji, konwencja światła ta sama; spójność zestawu ponad indywidualny błysk pojedynczego obrazu.
- Oddajesz warstwowo z programem źródłowym i raportem liczb przed/po; jeden asset całą trasą do silnika zanim ruszy zestaw.

## Notatki o źródłach

| Źródło | Wiarygodność | Ostatnia aktualizacja |
| --- | --- | --- |
| [MetaPoint: Unlocking Precise Spatial Control in Agentic Visual Generation (arXiv)](https://arxiv.org/abs/2606.05031) | 4/5 | 2026-06-03 |
| [MT Forcing: Multi-Turn Image Edit Drift (explainer pracy SenseTime/HKUST/Stanford/Tsinghua)](https://explainx.ai/blog/mt-forcing-multi-turn-image-editing-autoregressive-diffusion-sensetime-2026) | 3/5 | 2026 |
| [Stroke-based Rendering: From Heuristics to Deep Learning (przegląd, arXiv)](https://arxiv.org/pdf/2302.00595) | 4/5 | 2023-02 |
| [Hertzmann: Stroke-Based Rendering, notatki kursowe (SIGGRAPH 02)](https://www.cs.ucdavis.edu/~ma/SIGGRAPH02/course23/notes/S02c23_3.pdf) | 4/5 | 2002 |
| [IconShop: Text-Guided Vector Icon Synthesis with Autoregressive Transformers (arXiv 2304.14400)](https://www.alphaxiv.org/abs/2304.14400) | 4/5 | 2023-04 |
| [StarVector: Generating Scalable Vector Graphics Code from Images (arXiv)](https://arxiv.org/html/2312.11556v1) | 4/5 | 2023-12 |
| [Spellburst: LLM-Powered Interactive Canvas for Generative Artists (Stanford HAI)](https://hai.stanford.edu/news/spellburst-large-language-model-powered-interactive-canvas-generative-artists) | 4/5 | - |
| [AI Co-Artist: LLM-Powered Framework for GLSL Shader Animation Evolution (arXiv)](https://arxiv.org/html/2512.08951v1) | 3/5 | 2025-12 |
| [Battle Brothers, Dev Blog #5: Explaining Character Art Style (Overhype)](https://battlebrothersgame.com/dev-blog-5-concept-art-explaining-battle-brothers-character-art-style/) | 4/5 | - |
| [LucasArts' Secret History: The Curse of Monkey Island, wywiad z Billem Tillerem (MixnMojo)](https://web.archive.org/web/20161029003120/https:/mixnmojo.com/features/sitefeatures/LucasArts-Secret-History-The-Curse-of-Monkey-Island/5) | 4/5 | 2009 |
| [Team Cherry, FAQ: czym i jak robiona grafika](https://www.teamcherry.com.au/faq) | 4/5 | - |
| [Game Developer: Ubisoft's Ancel Planning To Open Up Rayman Origins Tech](https://www.gamedeveloper.com/business/ubisoft-s-ancel-planning-to-open-up-i-rayman-origins-i-tech) | 4/5 | - |
| [Polygon: Ubisoft Montpellier developers discuss Reinventing Rayman (GDC)](https://www.polygon.com/2013/3/28/4157732/ubisoft-montpellier-developers-discuss-reinventing-rayman/) | 3/5 | 2013-03-28 |
| [Polygon: Rayman Legends' design philosophy: breathe life into concept art](https://www.polygon.com/2013/7/12/4517060/rayman-legends-design-philosophy-breathe-life-into-concept-art/) | 3/5 | 2013-07-12 |
| [Krita 5.3 and 6.0 Release Notes: Python API for painting brushstrokes](https://krita.org/en/release-notes/krita-5-3-release-notes/) | 5/5 | - |
| [Krita: Python Scripting (dokumentacja)](https://docs.krita.org/en/user_manual/python_scripting/introduction_to_python_scripting.html) | 5/5 | - |
| [Inkscape Manual Page: praca z linii poleceń](https://inkscape.org/doc/inkscape-man.html) | 5/5 | - |
| [ComfyUI MCP Server (GitHub, shawnrushefsky)](https://github.com/shawnrushefsky/comfyui-mcp) | 3/5 | - |
| [MindStudio: ComfyUI's Official MCP for Claude and Codex](https://www.mindstudio.ai/blog/comfyui-mcp-claude-codex-workflow) | 3/5 | - |
| [QWE AI Academy: Stop Generating 1000 Images: Use Inpainting Instead](https://www.qwe.edu.pl/tutorial/stable-diffusion-inpainting-tutorial/) | 2/5 | - |
| [MimicPC: Img2Img in Stable Diffusion, przewodnik z liczbami](https://www.mimicpc.com/learn/img2img-in-stable-diffusion) | 2/5 | - |
| [OpenAI Developer Community: GPT-image-1 problems with mask edits](https://community.openai.com/t/gpt-image-1-problems-with-mask-edits/1240639) | 2/5 | - |
| [AdControlCenter: The 2-edit rule for AI ad images](https://www.adcontrolcenter.com/learn/the-2-edit-rule-for-ai-ad-images) | 2/5 | - |
| [each::labs: Ideogram 4.5 Image Edits That Don't Drift](https://www.eachlabs.ai/blog/ideogram-4-5-edits-that-dont-drift) | 2/5 | - |
| [Online Art Lessons: How to Judge Tonal Values (squint test, szara zupa)](https://onlineartlessons.com/tutorial/how-to-judge-tonal-values-in-your-paintings/) | 2/5 | - |
| [Russell Collection: What Is Value in Painting (Carlson, grupowanie walorów)](https://russell-collection.com/what-is-value-in-painting/) | 2/5 | - |
| [MontCarta: 70/30 Rule, test miniatury, audyt w szarości](https://montcarta.com/blogs/art-guide-inspiration/what-is-70-30-rule) | 2/5 | - |
| [Klipse: Procedural Texture Generator in JavaScript](https://blog.klipse.tech/javascript/2020/09/10/procedural-texture-generator-javascript.html) | 2/5 | 2020-09-10 |
| [aixplain: Evolving Art, LLM jako krytyk w pętli promptów](https://aixplain.com/blog/evolving-art-how-visual-llms-are-autonomously-guiding-generative-ai/) | 2/5 | - |

Konflikty i zastrzeżenia:

- Liczby MT Forcing (6,34/0,74 px, spójność 7,09/8,40) pochodzą z explainera; benchmark MTEdit-Bench jest autorstwa tego samego zespołu co metoda, traktować jako kierunkowe, nie niezależne.
- „Edits that don't drift" Ideogramu to twierdzenie marketingowe sprzedawcy; przywołane jako dowód, że problem dryfu jest rozpoznany rynkowo, nie jako dowód skuteczności.
- Badania stroke-based rendering dotyczą renderowania stylizowanego z istniejącego obrazu (fotografii); ich przeniesienie na tworzenie painted art od zera jest adaptacją z tego dokumentu, nie wynikiem z badań.
- IconShop (ikony) i StarVector (pojedyncze obiekty, obrazy→SVG) nie pokazują, że LLM utrzyma spójność pełnej ilustracji; skala scen pozostaje pytaniem otwartym.
- Progi bramek walorów (60% midtonów, dominanta 60-80%) wyprowadzono z poradników malarskich; to propozycje do kalibracji, nie normy.

## Otwarte pytania

- Jak duże ilustracje SVG LLM utrzyma spójnie (limity tokenów, liczba obiektów, głębokość grup) i gdzie przekracza to koszt pamiętania całości; brak benchmarku.
- Czy skryptowane malowanie pędzlami (Krita API) daje jakościowo przekonujący painted look, czy wykrywalnie proceduralny szum; wymaga testu na własnym pipeline.
- Nie istnieje niezależny benchmark porównujący precyzję edycji maskowej GPT-image z lokalnym SD inpainting; reguły diff poza maską są obronne, ale liczby brakuje.
- Bramki walorów nie były walidowane na zbiorze assetów gier (w przeciwieństwie do reguł pixelowych z dokumentu pierwszego); progi wymagają kalibracji na własnym zestawie.
- Nie znaleziono produkcji, która malowałaby assety gier programem pędzli sterowanym przez agenta; trasa jest złożeniem udokumentowanych elementów (Krita API, SBR, Spellburst), nie repliką istniejącej.

## Rekomendacje / następne kroki

1. Przyjmij definicję trasy: painted art produkujesz programem malującym, a generator obrazowy traktujesz jak materiał rysunkowy (tekstury, koncepcje), nigdy jak proces główny. Decyzja w briefie przed pierwszą klatką.
2. Value plan przed kolorem, zawsze: rodziny walorów, dominanta, kierunek światła; squint skrypt jako pierwsza bramka, szara zupa jako fail. Złote walory to najtańsze miejsce na poprawkę, jak sylwetka w pixel arcie.
3. Wszystko, co się powtarza między assetami (cienie, tła, fragmenty zbroi, twarze), prowadź jako warstwy i moduły we wspólnym pliku źródłowym; to lekcja Battle Brothers i największa znana oszczędność trasy.
4. Retake przez parametr, nie regenerację: skoro program maluje, każda poprawka jest edycją liczby, koloru albo kształtu z diffem ograniczonym do celu poprawki; wzorzec suwaków ze Spellburst.
5. Jeśli generator: seed zapisany, inpainting z maską, denoise 0,4-0,6 przy twarzach, weryfikacja diff poza maską, archiwum promptów; jedna edycja naraz, weryfikacja po każdej, nigdy seria nieweryfikowanych.
6. Kolejność warstw od grubego do cienkiego (recepta SBR): sylwetka, walory, kolor, tekstura, detale; stop na bramce mapy różnicy, nie na „wygląda dobrze".
7. Jeden asset (postać, tło, prop) przeprowadź całą trasą do silnika zanim ruszy zestaw; lista problemów z tego przebiegu to lista bramek do wpisania w walidator; raport liczb przed/po bez zmian względem dokumentów poprzednich.