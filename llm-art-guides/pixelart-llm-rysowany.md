---
name: "pixelart-llm-rysowany"
title: "Rysowany pixel art dla agentów LLM: uniwersalny przewodnik (retro i modern)"
type: "text/markdown"
---

# Rysowany pixel art dla agentów LLM: uniwersalny przewodnik

**Status:** wersja 1.0, walidacja wtórna zakończona, dokument kompletny pod względem źródeł i repozytoriów skillli, gotowy do wrzucenia do paczki jako materiał uniwersalny dla dowolnego agenta. Zakres: pixel art rysowany piksel po pikselu. Dokument komplementarny o trasie pre-renderu 3D (Diablo 2, Dead Cells) osobno.

## Pytanie badawcze

Jakie metody, reguły i wzorce architektoniczne pozwalają dowolnemu agentowi LLM rysować pixel art na poziomie rzemiosła, w dwóch stylach docelowych: retro (małe płótna, ograniczone palety, styl Settlers/AoE) i modern artystyczny (Blasphemous i Touhou Luna Nights jako kotwice: większe płótna, rozbudowane rampy, dynamiczne światło, wyrazista animacja). Nacisk na trzy problemy praktyczne: utrzymanie bryły i sylwetki, jakość animacji, ochrona przed artefaktami. Dokument jest agnostyczny: nie zakłada konkretnego silnika, edytorów ani pipeline, ma działać jako materiał wrzucany do dowolnego agenta (Claude, GPT, Gemini, lokalny model z narzędziami). Zakres ograniczony do pixel artu rysowanego piksel po pikselu; trasa pre-renderu 3D jest wyłączona i trafi do osobnego dokumentu.

## Streszczenie wykonawcze

1. **Fundament rzemiosła jest jeden dla obu stylów:** każdy piksel to świadoma decyzja, nie render niskiej rozdzielczości. Test sylwetki jednym płaskim kolorem przed cieniowaniem, jeden stały kierunek światła w całym zbiorze assetów, kolejność pracy: płaskie bazy → test sylwetki → jeden cień → jedno światło → dopiero detale. Ta kolejność jest najskuteczniejszą pojedynczą bramką jakościową dla LLM.
2. **Retro i modern różni budżet, nie zasady.** Retro (≤32 px): dominuje luka jasności, 2 kroki cienia przy 16×16, 3 przy 32×32, ≤16 kolorów, dithering oszczędnie. Modern (48 px+): 4-5 kroków rampy, pełny hue-shift, sel-out (kontur kolorowany światłem), mocniejsze światło kolorowe. Pomiar na 865 assetach pokazał, że hue-shift w małych rozmiarach realnie występuje w ~2% par kolorów i zaczyna się opłacać od ramp o 3+ krokach, to korekta popularnego hasła "hue-shiftuj zawsze".
3. **Animacja to pozy kluczowe, nie płynność:** energy > smoothness, nadmiar klatek pośrednich zabiera energię ruchu. Timing nienormowany (atak gracza: uderzenie 40-60 ms, hold 150-250 ms, recovery dłuższy niż swing), sekwencja ataku: anticipation → smear → hit (hold) → follow-through → recovery → overshoot, smear to zwykle jedna klatka namalowana najjaśniejszymi kolorami rampy. Potwierdzone krzyżowo przez dwóch niezależnych autorów (Saint11 z Celeste, Slynyrd).
4. **Artefakty są katalogizowane i mierzalne:** mixels, banding, poduszka, osierocone piksele, półprzezroczysta alfa, zupa kolorów, gotujące się klatki. Istnieją progi liczbowe do wbudowania w walidator (0 kolorów poza paletą, 0 półprzezroczystych, 0 osieroconych, near-duplikaty ΔE<3 = 0) i standardowa kolejność napraw: skala → paleta → sylwetka/światło → klastery/banding → krawędzie → detale.
5. **Największa różnica między agentem a człowiekiem to pętla spojrzenia:** agent musi po każdej zmianie "patrzeć" na wynik (render → look → fix), a animację przeglądać wyłącznie filmstripem (wszystkie klatki w poziomym pasie), bo modele wzrokowe czytają zwykle tylko pierwszą klatkę GIF. Bez filmstripu agent nie widzi własnej animacji i nie może jej ocenić.
6. **Kotwica stylu modern musi być naprawdę rysowana.** Przykład podany w prompcie działa na agenta jak atraktor, ciągnie skojarzenia także metodą produkcji. Najpopularniejsze wzorce "pięknego modern pixel artu" mają trasy nieczysto rysunkowe: Dead Cells renderuje modele 3D do klatek, Eastward nakłada oświetlenie 3D i malowane bump mapy na rysowane klatki. Kotwice tego dokumentu to gry potwierdzone jako rysowane piksel po pikselu: Blasphemous (9 artystów, udokumentowane time-lapsy klatka po klatce) i Touhou Luna Nights / Deedlit in Wonder Labyrinth (wzorzec płynności animacji sprite'owej).
7. **Krajobraz gotowych skillli dzieli się na trzy trasy** (deterministyczny kod, edytor w pętli MCP, image-gen z wykończeniem). Żadna nie produkuje rzemiosła bez deterministycznego komponentu; dla stylów retro i modern rysunkowych właściwa jest trasa kodowa lub edytorowa, image-gen maksymalnie do konceptów.

## Metodologia

- Kąty wyszukiwania: (1) skille i repozytoria pixel artowe budowane pod agentów LLM, (2) klasyczne rzemiosło pixel artu sformalizowane pod maszyny: sylwetka, bryła, kolor, cieniowanie, (3) techniki animacji: pozy kluczowe, timing, smear, sub-pixel, (4) katalog artefaktów i metody detekcji, (5) produkcja stylu modern (Dead Cells) i jej różnice wobec retro, (6) ograniczenia modeli dyfuzyjnych.
- Typy źródeł: baza wiedzy rzemieślniczej pisana pod agentów (Pixeli), repozytoria GitHub z działającymi pipeline, klasyczne tutoriale uznanych autorów (Saint11/Pedro Medeiros z MiniBoss, Slynyrd/Raymond Schlitter), wywiady i deep divy z artystami (Game Developer, Nintendo Life), artykuły rzemieślnicze i pomiary na zbiorach assetów (Pixnote).
- **Walidacja wtórna:** główne twierdzenia wersji pierwotnej skrzyżowano z niezależnymi źródłami. Wyniki:
  - Pose-to-pose i prymat póz kluczowych: potwierdzone u Saint11 (straight-ahead vs pose-to-pose, klatka "still") i Slynyrd (energia z póz kluczowych, usuwanie klatek oszczędnościowych z zachowaniem contact/pass).
  - Timingi ataku, smear, overshoot, freeze: potwierdzone u Slynyrd (smear zwykle jedna klatka, freeze na anticipation i trafieniu, overshoot jako fizyka wahadła, gracz oczekuje natychmiastowej odpowiedzi na przycisk).
  - Zachowanie objętości przy squash/stretch: potwierdzone u Saint11 (spłaszczasz w osi ruchu, zachowujesz łączną objętość).
  - Poduszka i spójny kierunek światła: potwierdzone przez Pixnote (złota zasada spójności światła w całej scenie/grze).
  - "Hue-shift zawsze": **skorygowane** pomiarem Pixnote (865 assetów 16×16, mediana przesunięcia barwy tylko 8°+ w 2% par). Prawidłowa reguła: małe płótna = najpierw luka jasności, hue-shift rośnie z długością rampy.
  - "Dead Cells jako wzór rysowany": **skorygowane** z primary sources (deep dive artysty Thomasa Vasseura, wywiad z producentem Steve'em Filbym): postaci i animacje z 3D, tła rysowane ręcznie. W tej samej fazie sprawdzono kandydatów na kotwicę stylu modern: Eastward (ok. 70 000 rysowanych klatek, ale oświetlenie 3D i bump mapy), Katana ZERO (animacje rysowane w Aseprite, ale pojedyncze propsy z voxeli/Blendera), Blasphemous (rysowany piksel po pikselu, time-lapsy procesu, 9 artystów), produkcje Team Ladybug (wzorzec płynności animacji). Konsekwencja: kotwicą dokumentu jest Blasphemous z Team Ladybug, nie Dead Cells.
- Ograniczenia: progi liczbowe z bazy Pixeli wyprowadzono dla sprite'ów ≤32 px, przy większych płótnach wymagają przeskalowania. Pomiar Pixnote obejmuje wyłącznie 16×16, dla stylu modern brakuje analogicznego pomiaru. Artykuł Kokutech o pipeline z krytykami LLM pozostał nieprzeczytany (blokada antybotowa).

## Wyniki

### 1. Krajobraz podejść dla agentów

| Podejście | Mechanizm | Przedstawiciele | Ocena |
| --- | --- | --- | --- |
| Edytor w pętli (MCP) | Agent pisze klatki znak po znaku, po każdej operacji patrzy na wynik, bramki validate, filmstrip, onion | aseprite-ai-artist / baza Pixeli | Najlepsza znana jakość dla rysunkowej trasy; wymaga edytora w pętli |
| Deterministyczny kod | Każdy piksel decyzją w skrypcie, renderer wypisuje PNG, bramki liczbowe w kodzie | pixel-art-studio (Pillow), pixel-art (siatka znaków), Sprite (spec zablokowany paletą + stdlib renderer), claude-pxl (klatki jako patche nad bazą), PixelSpriteAI (biblioteka pośrednicząca), agent-sprites (kształty parametryczne z z-order) | Najlepsza powtarzalność i audytowalność, niezależna od edytora |
| Image-gen + wykończenie | Master still z modelu obrazowego → deterministyczny keying, snap palety, retiming, color lock | agent-sprite-forge (metryki przed/po, bramki per take) | Sensowne tylko z pełnym łańcuchem wykończenia; bez niego znamiona sztuki maszynowej |
| Czysta dyfuzja | Pixel art LoRA-e, modele trenowane pod pixel art | RetroDiffusion, LoRA-e SD | Fundamentalnie ograniczona: model uczy się wyglądu decyzji, nie decyzji; chwiejny grid, zupa kolorów, gotujące się animacje |

Wniosek przekrojowy: niezależnie od trasy, jakość wynika z trzech elementów: (1) wiedza rzemieślnicza zapisana jako reguły z progi liczbowymi, (2) pętla render → spojrzenie → poprawka, minimum dwa przebiegi, (3) bramki walidacji między etapami. Te elementy da się zbudować w każdym agencie.

### 2. Fundament rzemiosła (oba style)

**Dyscyplina podstawowa:** pixel art to nie mały obrazek, to obraz, w którym każdy piksel był decyzją. Agent traktujący płótno 32×32 jak render niskiej rozdzielczości wyprodukuje coś technicznie poprawnego i jednoznacznie złego.

**Test sylwetki:** wypełnij sprite jednym płaskim kolorem. Jeśli nie widać, co to jest, żadne cieniowanie tego nie uratuje. Wykonywać przed cieniowaniem. Trzy typowe przyczyny porażki: idealna symetria (czyta się jak posąg, łamie się ramieniem w przód, bronią z boku, głową w bok), tangenty (kontur ramienia dotykający konturu torsu zlewa się w blob, zostaw przerwę lub nachodzenie), jednolita grubość kończyn (patyk).

**Proporcje i rozmiar płótna:** 2 głowy wysokości = chibi, 3-4 = klasyczny JRPG, 5-6 = realistyczne i wymaga od ~48 px wzwyż. Wybór proporcji to decyzja projektowa podjęta świadomie, nie uboczny efekt rozmiaru płótna.

**Światło:** jeden kierunek ustalony z góry, konsekwentny w całym zbiorze assetów (złota zasada: niezgodne światła na postaciach, tłach i przedmiotach tworzą dysonans). Rzut 3/4: głowa pokazuje twarz i czubek czaszki, barki szersze niż w czystym froncie, dalsza kończyna częściowo zasłonięta torsa (to sprzedaje głębię).

**Cienie i rampy:** cień to nie "ciemniejszy ten sam kolor". Trzy elementy cienia: jasność w dół, saturacja lekko w górę (obniżenie saturacji daje błotnisty efekt), barwa ku chłodnemu. Walidacja wtórna doprecyzowuje: przy 2-3 krokach rampy dominującym nośnikiem formy jest luka jasności (mediana ~16,5 punktu HSL L w pomiarze na 865 assetach), hue-shift zaczyna pracować od ramp o 3+ krokach i rośnie z rozmiarem płótna. Reguła praktyczna: mały sprite = światło niosące formę przez kontrast jasności, duży sprite = pełne hue-shiftowanie kroków.

**Kolejność pracy:** płaskie kolory bazowe per materiał → test sylwetki → jeden krok cienia od strony przeciwnej światłu → jeden krok światła → dopiero drugi cień, spekular, światło odbite. Większość sprite'ów jest skończona po jednym cieniu i jednym świetle. Detal dodany przed poprawną formą utrudnia tylko dostrzeżenie błędu.

**Klastery i linie:** sprite czyta się jako kształty koloru, nie jako piksele. Osierocone pojedyncze piksele, kropki rozsiane po krawędziach, klastry 2×2 na linii 1 px czytają się jako brud. Linia to schody, oko czyta jej rytm: jeden przebieg krótszy od sąsiednich to garb (jaggy). Ręczny anti-aliasing to jeden pośredni kolor na ostrym skosie, nigdy mieszanie alfa, nigdy filtry. Zero pikseli półprzezroczystych w wyjściu.

**Dithering:** kupuje pośredni odcień, którego nie ma w palecie, ale dodaje piksele, które nie są sylwetką ani światłem. Tylko wzory uporządkowane (bayer), tylko między sąsiednimi kolorami rampy, tylko na dużych płaskich obszarach od ~32×32. Noise dither = brud. W animacji dither migocze.

### 3. Profil retro (do ~32 px)

- Budżet kolorów: ≤16 przy ≤32 px, ≤32 przy 64 px. Zero kolorów poza paletą, zero near-duplikatów (ΔE<3).
- Kroje cienia wg rozmiaru: 16×16 → baza + cień (2 wartości), 32×32 → highlight + baza + cień (3), 48×48 → 3-4, 64×64+ → 4-5. Przy dwóch krokach hue-shift jest niesłyszalny, liczy się luka jasności.
- 16×16 to budżet 256 komórek: nie ma sub-pixel geometry, shadera ani rampy alfa, które zamaskowałyby złą formę. Każda reguła tego rozmiaru jest konsekwencją budżetu.
- Dithering oszczędnie lub wcale, AA minimalne albo żadne, detale tylko tam, gdzie niosą czytelność (twarz, broń), nie na butach.
- Czytelność testuje się w rozmiarze docelowym na ekranie, nie w powiększeniu 800%: detal, którego skala nie utrzyma, zamienia się w plamę, a więcej pikseli nie naprawia złej formy.

### 4. Profil modern artystyczny (kotwice: Blasphemous, Touhou Luna Nights)

Co definiuje wygląd: duże, gęsto cieniowane sprite'y (w Blasphemous bossy w skali ekranu), rozbudowane rampy (4-5 kroków), mocne kierunkowe światło z wyrazistym kontrastem, sel-out (kontur rozjaśniany od strony światła i przyciemniany w cieniu, technika efektywna od 32×32 wzwyż), animacje snappy z dominacją póz kluczowych i smearów, warstwy VFX budujące poczucie uderzenia. Traktuj to jako zestaw parametrów stylu, nie jako przepis produkcyjny.

Dlaczego te kotwice, a nie popularniejsze tytuły: przykład w prompcie agenta jest atraktorem, ciągnie skojarzenia także metodą produkcji, więc kotwica musi być zgodna z trasą, której uczysz. Najczęściej wskazywane wzorce "pięknego modern pixel artu" mają trasy nieczysto rysunkowe: Dead Cells renderuje modele 3D do klatek ("dynamic pixel art", oświetlenie pięknie dostrojone, pipeline spoza trasy rysunkowej), Eastward łączy ~70 000 rysowanych klatek z oświetleniem 3D i ręcznie malowanymi bump mapami, w Katana ZERO pojedyncze propsy (pojazdy) powstawały z voxeli i Blendera. Przeciwaga: Blasphemous jest rysowany piksel po pikselu przez zespół 9 artystów, proces jest udokumentowany time-lapsami klatka po klatce, a twórcy określają podejście jako purystyczne (w grze nie ma nic, co nie jest pixel artem); Touhou Luna Nights i Deedlit in Wonder Labyrinth wskazuje się jako jedne z najlepiej animowanych pixel artowych metroidvanii, klinikę animacji sprite'owej. Zastrzeżenie: pojedyncza niezweryfikowana anegdota twierdzi, że w Blasphemous zdarzały się bazy 3D pod pojedynczymi animacjami; ciężar dowodów (time-lapsy, portfolio artystów, wypowiedzi twórców) przemawia za trasą rysunkową, ale dlatego w prompcie warto podawać parametry stylu jawnie, a nazwę gry traktować jako etykietę, nie definicję.

Wskazówki rysunkowe do tego stylu: rigging warstwowy przed animacją (torso, głowa, kończyny bliskie/dalsze, broń, peleryna, włosy, cień jako osobna warstwa), sub-pixel animation dla idle (value tweening: światło wędruje po powierzchni przy zamrożonym konturze), smear z gradacją koloru (wygładza ruch bez doklejania klatek), elongacja ramienia lub broni o 1-2 px w maksymalnym zasięgu dla luzu, kontrastowa klatka trafienia (biała lub czarna sylwetka, 33-50 ms) przy mocnych ciosach.

### 5. Animacja: reguły potwierdzone krzyżowo

**Pozy kluczowe przed klatkami pośrednimi.** Straight-ahead (klatka po klatce) zostawia się małym, prostym ruchom i efektom; wszystko powyżej liczy się pose-to-pose: rysujesz skrajności, sprawdzasz czytelnik samego zestawu póz, potem doprawiasz. Nadmiar klatek pośrednich zabiera energię: 8-klatkowy cykl biegowy można ciąć do 6 (usuwasz recoil, bliski contactowi, wnosi najmniej) i 4 (usuwasz jeszcze high point), podnosząc czas każdej pozy proporcjonalnie (80 ms → 160 ms). Kluczowe dla cięcia: contact i pass zostają zawsze. Mega Man biega na 3 pozach i ma więcej energii niż wiele wieloklatkowych cykli.

**Timing to performance.** Równomierne czasy klatek czytają się mechanicznie niezależnie od jakości póz. Referencja: idle 2-4 klatki po 200-400 ms, walk 4-8 klatek po 100-150 ms z dłuższym holdem na contact, run 60-100 ms na klatkę, attack: samo uderzenie 40-60 ms, hold 150-250 ms.

**Atak w grze, nie w animowanym GIF-ie.** Kolejność faz: anticipation → smear → hit (hold) → follow-through → recovery → overshoot. Atak gracza: minimalny windup (≤50 ms albo wcale), bo windup po naciśnięciu przycisku czuje się jak input lag, pierwsza klatka po przycisku to uderzenie lub smear. Potwory telegrafują 150-400 ms trzymaną, czytelną sylwetką (sygnał gameplayowy). Ciężar broni niosą anticipation i recovery, nigdy wydłużony smear: smear zawsze krótki, zwykle jedna klatka, namalowana ścieżką od poprzedniej do następnej pozycji, najjaśniejszymi kolorami rampy, bez przezroczystego rozmycia; gradacja koloru w smearze wygładza ruch bez dodatkowych klatek. Overshoot: obiekt hamując przelatuje pół piksela za punkt docelowy i wraca, nawet 1 px jest wyczuwalny. Freeze/hold na pozie dodaje ciężar, ale subtelnie. Rozciąganie kończyny o 1-2 px w maksymalnym zasięgu daje luz przy stabilnej sylwetce ciała.

**Squash/stretch zachowuje objętość:** spłaszczenie w osi ruchu z kompensacją w osi prostopadłej, łączna objętość stała, inaczej obiekt "rośnie" i "kurczy się" między klatkami. To reguła, którą się łamie świadomie na wyższym poziomie, nie przypadkiem.

**Ruch wtórny i sub-pixel:** włosy, peleryna, plecak i końcówka broni osiadają 1-2 klatki po zatrzymaniu ciała. Idle nie rusza konturem tylko obrysem: value tweening przesuwa światło i cień po powierzchni przy zamrożonym konturze (oddech, dryf), outline tweening faluje o 1 px.

**Kontrola krzyżowa klatek (checklist):** stała wysokość poza zamierzonym squashem, stała grubość kończyn, stały kierunek światła, domknięta pętla (klatka N jest wiarygodnym poprzednikiem klatki 1), brak kolizji części, które nie mogą kolidować.

**Rig przed animacją:** piksele wypalone w jednej warstwie wymagają przerysowania, żeby je ruszyć. Dziel przed: spis warstw od dołu: cień, noga dalsza, ręka dalsza, tors, głowa, noga bliższa, ręka bliższa, plus broń, peleryna, włosy. Klatka = obraz jednej warstwy w jednej klatce, przesuwanie warstwy między klatkami jest tanie, przerysowanie nie. Części naprawdę nieruchome w cyklu łącz w linki (edycja raz, aktualizacja wszędzie).

### 6. Artefakty: katalog, progi, naprawa

**Znamiona (tells) sztuki maszynowej:**

- Grid i geometria: mixels (bloki nierównego rozmiaru, krawędzie dryfujące ±20%), plik pre-skalowany (przebiegi pikseli wielokrotnością k>1), ślady skalowania bilinearnego (kolory pośrednie na każdej krawędzi).
- Kolor: zupa kolorów (setki bliskich odcieni zamiast kilkunastu), rampy tylko ciemniejsze, czyste czarne kontury wszędzie.
- Piksele: osierocone pojedyncze piksele, półprzezroczysta alfa, banding (równoległe pasy sąsiednich kroków rampy), poduszka (równomierne przyciemnienie wzdłuż całego konturu, brak czytelnego kierunku światła), noise dither, jaggies.
- Animacja: gotowanie klatek, migotanie kolorów, utrata grubości kończyn między klatkami.

**Progi walidatora (punkt wyjścia, przeskalować do rozmiaru płótna):** użyte kolory ≤16 przy ≤32 px, poza paletą 0, near-duplikaty ΔE<3 → 0, półprzezroczyste 0, osierocone 0 (max 1 uzasadnione oko lub highlight na obiekt), banding 0, kontur albo ≥60% jednego koloru, albo celowo brak. Czasomierze klatek tagowane, czasy nienormowane w cyklu.

**Test wiersza dla importów:** odczytaj jeden rząd pikseli, wypisz długości przebiegów tego samego koloru. Wszystkie przebiegi wielokrotnością k>1 → plik pre-skalowany, odzyskiwalny przez downscale nearest-neighbour. Przebiegi bez wspólnego dzielnika → mixels, nie odzyskiwalne snapowaniem, przerysować w 1×. Cokolwiek przychodzi z zewnątrz (image-gen, skan, referencja) traktować jako koncept, nie asset: importować jako warstwę referencyjną i rysować po niej w docelowym rozmiarze, nigdy filtrować, wygładzać ani "pixelizować".

**Kolejność napraw:** pitch i skala → paleta → sylwetka i światło → klastery i banding → krawędzie → detale. Naprawa przed porządkiem w palecie powtarza pracę. Naprawa wyłącznie na natywnej skali, skalowanie tylko całkowitoliczbowe nearest-neighbour, żadnego rozmywania dla zamaskowania. Po każdej naprawie ponowny detektor z raportem przed/po.

**Pętla spojrzenia dla LLM (element krytyczny):** po każdej operacji, która zmieniła piksele, agent renderuje podgląd i patrzy. Animację przegląda filmstripem, bo model wzrokowy czyta zwykle pierwszą klatkę GIF. Onion skin (nałożenie klatki na wyśnione sąsiadki) do oceny spacingu. Dump tekstowy/ASCII siatki do wyłapania piksela o wiersz za nisko i przebiegu 3 w linii dwójek. Raport końcowy: co sprawdzono, co znaleziono, nazwane kompromisy; użytkownik naprawi ujawniony kompromis, ukryty zostanie w grze.

### 7. Blueprint: jak zbudować to w dowolnym agencie

Niezależnie od trasy (skrypty rysujące, edytor przez MCP, API), sprawdzona architektura ma pięć elementów:

1. **Plik reguł rzemiosła:** sekcje 2-6 tego dokumentu jako zwięzłe reguły z progi liczbowymi, które agent czyta przed rysowaniem. Reguły z liczbami (nie "unikaj bandingu", tylko "0 pasów sąsiednich kroków rampy równoległych do konturu").
2. **Brief z kryteriami ukończenia:** temat, styl (retro/modern), rozmiar, paleta, rzut, proporcje w głowach, 2-6 konkretnych kryteriów "gotowe". Pytać tylko o to, co zmienia wynik, resztę podać jako domyślne.
3. **Bramki między etapami:** test sylwetki po block-inie, walidator palety po cieniowaniu, kontrola krzyżowa po klatkach animacji, filmstrip przed eksportem. Bramka to skrypt lub operacja o określonym progu, nie "sprawdź, czy ładne".
4. **Pętla render → spojrzenie → poprawka:** minimum dwa pełne przebiegi przed oddaniem, każdy przebieg z listą znalezionych problemów.
5. **Eksport z metadanymi:** sprite sheet + atlas (JSON) + czasy klatek + tagi akcji w konwencji silnika docelowego.

Gotowy zestaw reguł do wklejenia w system prompt lub plik skilla (skondensowany):

- Każdy piksel to decyzja. Rysuj na natywnej skali, skaluj tylko całkowitoliczbowo nearest-neighbour.
- Przed cieniowaniem zrób test sylwetki jednym kolorem. Sylwetka do naprawy = brak symetrii idealnej, brak stycznych konturów, zróżnicowana grubość kończyn.
- Jedno światło, jeden kierunek, w całym zbiorze assetów, we wszystkich klatkach. Cień chłodniejszy, światło cieplejsze od bazy.
- Kolejność: bazy → sylwetka → 1 cień → 1 światło → detale. Większość prac kończy się na 1 cieniu i 1 świetle.
- Zero: pikseli półprzezroczystych, kolorów poza paletą, osieroconych pikseli, bandingu, poduszki, noise ditheru.
- Animacja pose-to-pose: contact i pass zawsze, in-betweeny tylko po zatwierdzeniu póz, czasy nienormowane.
- Atak gracza bez windupu; potwór telegrafuje 150-400 ms; smear 1 klatka, ścieżką między pozycjami, najjaśniejszymi kolorami rampy; recovery dłuższy niż swing; overshoot 1 px.
- Objętość stała między klatkami: wysokość i grubość kończyn kontrolowane, squash kompensowany.
- Przeglądaj filmstrip, nie GIF; po każdej zmianie renderuj i patrz; oddajesz z raportem: co sprawdzono, co poprawiono, jakie kompromisy zostały.

## Notatki o źródłach

| Źródło | Wiarygodność | Ostatnia aktualizacja |
| --- | --- | --- |
| [Pixeli: baza wiedzy pixel artu](https://pixeli.pebbly.space/knowledge) | 4/5 | - |
| [Pixeli: sylwetka i bryła](https://pixeli.pebbly.space/knowledge/rules/03-silhouette-and-form) | 4/5 | - |
| [Pixeli: cieniowanie i światło](https://pixeli.pebbly.space/knowledge/rules/02-shading-and-light) | 4/5 | - |
| [Pixeli: animacja](https://pixeli.pebbly.space/knowledge/rules/05-animation) | 4/5 | - |
| [Pixeli: warstwy i rigging](https://pixeli.pebbly.space/knowledge/rules/06-layers-and-rigging) | 4/5 | - |
| [Pixeli: checklista przeglądu](https://pixeli.pebbly.space/knowledge/rules/07-review-checklist) | 4/5 | - |
| [Pixeli: ataki i trafienia](https://pixeli.pebbly.space/knowledge/rules/44-attacks-and-impacts) | 4/5 | - |
| [Pixeli: znamiona sztuki generowanej](https://pixeli.pebbly.space/knowledge/rules/92-generated-art-tells) | 4/5 | - |
| [with-pebbly/aseprite-ai-artist (GitHub)](https://github.com/with-pebbly/aseprite-ai-artist) | 4/5 | - |
| [Saint11: A Basic Aseprite Animation](https://saint11.art/pixel_art_articles/article3/) | 4/5 | - |
| [Slynyrd: Pixelblog 8, Intro to Animation](https://www.slynyrd.com/blog/2018/8/19/pixelblog-8-intro-to-animation) | 4/5 | 2018-08-19 |
| [Slynyrd: Pixelblog 9, Melee Attacks](https://www.slynyrd.com/blog/2018/9/8/pixelblog-9-melee-attacks) | 4/5 | 2018-09-08 |
| [Pixnote: Shading & Lighting (z pomiarem 865 assetów)](https://pixnote.net/en/learn/shading/) | 3/5 | - |
| [Pixnote: Pixel Art Glossary](https://pixnote.net/en/learn/glossary/) | 3/5 | - |
| [Game Developer: Using a 3D pipeline for 2D animation in Dead Cells (Thomas Vasseur)](https://www.gamedeveloper.com/production/art-design-deep-dive-using-a-3d-pipeline-for-2d-animation-in-i-dead-cells-i-) | 5/5 | 2018-01-25 |
| [Nintendo Life: wywiad z Motion Twin (Steve Filby)](https://www.nintendolife.com/news/2018/05/feature_reanimating_the_roguelike_with_dead_cells_developer_motiontwin) | 4/5 | 2018-05 |
| [Blasphemous: Pixel Art Animation Time-Lapses (Game Anim)](https://www.gameanim.com/2021/02/19/blasphemous-pixel-art-animation-time-lapses/) | 3/5 | 2021-02-19 |
| [The Game Kitchen AMA (r/NintendoSwitch)](https://www.reddit.com/r/NintendoSwitch/comments/d2af0j/hey_were_the_game_kitchen_the_team_behind/) | 3/5 | - |
| [Blasphemous designer on purist pixel art (Hollywood Reporter)](https://www.hollywoodreporter.com/movies/movie-news/blasphemous-designer-purist-approach-games-pixel-art-style-1238370/) | 3/5 | - |
| [The Best Pixel Art Metroidvanias, real sprites (the-pixel.art)](https://the-pixel.art/articles/best-pixel-art-metroidvanias/) | 3/5 | 2026-08-29 |
| [Eastward's creators on pixel art pipeline (Game Developer)](https://www.gamedeveloper.com/art/eastward-s-creators-share-insights-on-making-pixel-art-adventures) | 4/5 | - |
| [Eastward preview, 70 000 hand-drawn frames (Wireframe)](https://wireframe.raspberrypi.com/articles/eastward-preview-promising) | 3/5 | - |
| [Katana ZERO van, artefakty produkcji (Dribbble, A. Samoilov)](https://dribbble.com/shots/19492174-Katana-Zero-van-pixelart-animation) | 2/5 | - |
| [Gamezxz/pixel-art-studio (GitHub)](https://github.com/Gamezxz/pixel-art-studio) | 3/5 | - |
| [junyudev/pixel-art (GitHub)](https://github.com/junyudev/pixel-art) | 3/5 | - |
| [ehartye/agent-sprites (GitHub)](https://github.com/ehartye/agent-sprites) | 3/5 | 2026-10-25 |
| [0x0funky/agent-sprite-forge (GitHub)](https://github.com/0x0funky/agent-sprite-forge) | 3/5 | 2026-10-06 |
| [claude-pxl, przewodnik (Chen Media)](https://chen.media/guides/let-claude-paint-pixel-art-from-a-text-file) | 3/5 | - |
| [Sprite, skill Melodic (Skills Directory)](https://www.skillsdirectory.com/skills/melodic-software-sprite) | 3/5 | 2026-10-04 |
| [Jackmaster98/pixelart-generator-for-ai-agents (GitHub)](https://github.com/Jackmaster98/pixelart-generator-for-ai-agents) | 2/5 | - |
| [Can AI Actually Make Real Pixel Art? (the-pixel.art)](https://the-pixel.art/articles/can-ai-make-pixel-art/) | 3/5 | 2026-08-15 |
| [Sub-Pixel Animation (the-pixel.art)](https://the-pixel.art/articles/sub-pixel-animation-pixel-art/) | 3/5 | - |
| [Sub-Pixel Animation (Tiny Warrior Games)](https://tinywarriorgames.com/2019/01/04/game-development-pixel-art-sub-pixel-animation/) | 3/5 | 2019-01-04 |
| [Neta Studio: The Rules of 16x16 Pixel Art](https://neta.art/blog/16x16-pixel-art-rules) | 3/5 | - |
| [Let's talk about pixel art (r/StableDiffusion)](https://www.reddit.com/r/StableDiffusion/comments/1i6k7pp/lets_talk_about_pixel_art/) | 3/5 | - |
| [Retro Diffusion Models (Scenario KB)](https://help.scenario.com/en/articles/retro-diffusion-models-the-essentials) | 3/5 | - |

Konflikty i zastrzeżenia:

- "Hue-shift zawsze" (Pixeli, klasyczne tutoriale) kontra pomiar Pixnote (2% par z przesunięciem ≥8° przy 16×16). Rozstrzygnięcie w dokumencie: reguła skaluje się z długością rampy i rozmiarem płótna, obie strony mówią o innych budżetach.
- Smear: Pixeli podaje 1 klatkę przy 16-24 px i 2-3 przy 32+ px, Slynyrd pisze, że zwykle wystarczy jedna. Praktycznie: jedna domyślnie, druga dopiero gdy ruch nadal się rwie.
- Progi liczbowe Pixeli wyprowadzono dla ≤32 px, dla stylu modern trzeba je przeskalować eksperymentalnie, brak pomiaru analogicznego do Pixnote dla dużych płócien.
- Blasphemous jako kotwica: dokumentowane time-lapsy i wypowiedzi twórców kontra jedna niezweryfikowana anegdota o bazach 3D pod pojedynczymi animacjami. Kotwica utrzymana, zastrzeżenie w sekcji 4; w prompcie preferuj parametry stylu nad samą nazwą gry.
- Liczba gwiazdek repozytoriów nie świadczy o jakości rzemiosła, oceny w sekcji 1 są merytoryczne.

## Otwarte pytania

- Ile dokładnie hue-shiftu na krok rampy przy płótnach 48-64 px+? Pomiar Pixnote obejmuje tylko 16×16, dla stylu modern brakuje zdanych liczb; wymaga własnego pomiaru na reference'owym zbiorze.
- Jak przeskalować progi walidatora (cap kolorów, ΔE) na większe płótna? Przeskalowanie linearne z rozmiarem to hipoteza, nie zweryfikowana reguła.
- Czy benchmarki typu Pixel Bench dają porównywalne metryki między modelami? Nie zweryfikowano.
- Czy w Blasphemous zdarzały się bazy 3D pod pojedynczymi animacjami? Anegdota bez źródła pierwotnego; przy ciężarze dowodów (time-lapsy, wypowiedzi twórców) nie zmienia to wyboru kotwicy, ale wartoby ją potwierdzić w wywiadach z The Game Kitchen.
- Dokładny post-processing Dead Cells po eksporcie z 3D (snap palety? ręczne domalówki?) nie jest w pełni udokumentowany w otwartych źródłach; pytanie zamknie dokument o pre-renderze 3D.

## Rekomendacje / następne kroki

1. Wbuduj sekcję 2 (fundament) jako plik reguł w skillu/system prompcie agenta, sekcje 3 i 4 jako dwa przełączalne profile stylu (retro, modern), z jasnymi parametrami: rozmiar płótna, liczba kroków rampy, cap kolorów, dopuszczalny dithering, sel-out tak/nie.
2. Napisz walidator z progami z sekcji 6 jako skrypt, nie jako instrukcję tekstową agenta; agent ma raportować liczby, nie deklarować "wygląda dobrze".
3. Wymuś filmstrip w przeglądzie animacji i onion skin przy poprawianiu spacingu; to najtańsza ubezpieczka od najczęstszej ślepoty modeli.
4. Trzymaj timingi akcji w danych (manifest/atlas), nie w kodzie silnika: retiming ataku bez przerysowania klatek.
5. Do stylu modern podawaj w prompcie parametry wyglądu (duże gęste sprite'y, 4-5 kroków rampy, sel-out, snappy timingi, smear), a nazwy gier traktuj jako drugorzędne etykiety: parametry są odporne na skojarzenia, sama nazwa może pociągnąć agenta w stronę obcej metody produkcji.
6. Minimum dwa przebiegi render → spojrzenie → poprawka przed oddaniem każdego assetu, z raportem znalezionych problemów i nazwanymi kompromisami.