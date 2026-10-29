---
name: "pixelart-llm-prerender-3d"
title: "Pre-render 3D do sprite 2D dla agentów LLM: pipeline izometryczny w stylu Diablo 2"
type: "text/markdown"
---

# Pre-render 3D do sprite 2D: pipeline izometryczny w stylu Diablo 2

**Status:** wersja 1.1, druga walidacja zakończona (spójność tematyczna, atraktory i przykłady sprawdzone, znalezione błędy poprawione), dokument kompletny pod względem źródeł i narzędzi, gotowy do wrzucenia do paczki jako materiał uniwersalny dla dowolnego agenta. Zakres: trasa pre-renderu (model 3D → animacja → render → piksele) dla gier izometrycznych w stylu Diablo 2 i pochodnych. Dokument komplementarny do przewodnika pixel artu rysowanego (reguły rzemiosła wspólne: sylwetka, paleta, artefakty, przegląd filmstrip, omówione tam, tu tylko odsyłacze kontekstowe). Uwaga o atraktorach: Dead Cells w dokumencie rysunkowym pełni rolę kontrprzykładu (bo to nie jest pixel art rysowany), tu jest wzorcem trasy; obie ramy są zgodne, bo opisują różne trasy produkcji, nie różne oceny tej samej gry.

## Pytanie badawcze

Jak zbudować pipeline, w którym agent LLM modeluje i animuje postaci w 3D, a renderuje je do sprite'ów 2D dla gry izometrycznej w stylu Diablo 2 i pochodnych stylowo (Halls of Torment, linia AoE/Stronghold). Dead Cells pojawia się tu wyłącznie jako wzorzec pipeline (proste modele → pozy kluczowe → render do klatek), nie jako wzorzec stylu: to side-scroller, nie izometria, i jego estetyka nie jest celem tej trasy. Konkretne cele: jak tworzyć dobre modele pod sprite, jak robić animacje płynne i sensowne (timing, kierunki, ilość klatek), jak sprawić, żeby twarz czytała się jako ludzka twarz, a nie ziemniak, i jakie triki produkcyjne stosowali twórcy tego gatunku. Dokument agnostyczny: bez założeń o konkretnym silniku i narzędziach, gotowy do podania dowolnemu agentowi (Claude, GPT, lokalny model z narzędziami).

## Streszczenie wykonawcze

1. **Trasa ma 30 lat udokumentowanych sukcesów.** Donkey Kong Country (render ze stacji SGI do sprite'ów SNES), Age of Empires (cała grafika z modeli 3D Studio, „nawet jeśli ekran był 2D"), Diablo 2 (pre-render z paper dollem ekwipunku), Dead Cells (proste modele 3D → klatki pixelowe), Halls of Torment (współczesny dowód żywotności: assety 3D w Blender renderowane skryptami Python do sprite'ów, w samej grze 100% 2D). Dlaczego to ważne dla agenta: bryła, światło i spójność między klatkami przychodzą z geometrii za darmo, czyli dokładnie to, czego LLM nie umie utrzymać z ręki.
2. **Zasada kardynalna trasy: kamera stoi, model się obraca.** Kamera ortograficzna przykręcona sztywno (w D2 kierunki co 45°), render powtarzany per kierunek przez obrót modelu. Przy obrocie kamery zamiast modelu światło trzeba obrócić razem z kamerą, inaczej oświetlenie się „przełącza" między kierunkami; obie konwencje działają, ale trzeba jedną wybrać i trzymać się jej w całym zbiorze assetów.
3. **Sensowność animacji robi się w 3D na pozach kluczowych.** Recepta Dead Cells: animacja projektowana jak 2D, na pozach kluczowych, dopiero gdy pozy i timing są przekonujące przy minimalnej liczbie klatek, dodaje się klatki interpolacyjne przed lub za pozami kluczowymi, nigdy pomiędzy. Retake to przesunięcie keyframes na osi czasu, nie przerysowanie klatek. Potwory w D2: 8 kierunków, gracze 16.
4. **Model pod sprite: topologia obojętna, sylwetka święta.** Sprite wybacza brudną siatkę i wysoki polycount, nie wybacza kształtu, który ginie w downscale'u. Detal, który znika w rozmiarze docelowym, to stracona praca; test czytelności robi się w 1×, nie w podglądzie pełnoekranowym.
5. **Twarz bez ziemniaka rozwiązuje się na dwóch poziomach.** W 3D: twarz musi mieć zdefiniowane płaszczyzny (czoło, oczodoły, grzbiet nosa, kości policzkowe) i światło kierunkowe, które je modeluje, nie płaski shading; Dead Cells wydobywa objętość normal mapami i toon shaderem. W pikselach (bo finalny sprite to i tak kilka pikseli twarzy): oczy dostają swoje piksele pierwsze i nigdy nie są tym, co się wycina, biel oczu dopiero od 3 px szerokości oka, jeden catch-light, brwi niosą emocje przechyłką 1 px, cechy nigdy się nie dotykają, symetrię psuje się o 1 px (idealna symetria czyta się jak maska). Przy ≤16 px włosy i kaptur są twarzą, to nie lenistwo, to reguła: tak robił Diablo 2.
6. **Izometria to matematyka, nie wrażenie:** krawędź 2:1 (26,57°), kafel diament dwukrotnie szerszy niż wyższy, piony pozostają pionowe bez punktu zbiegu, trzy wartości ścian przy jednym świetle sceny (góra najjaśniejsza, lewa średnia, prawa najciemniejsza). Każda krawędź w scenie biegnie tym samym rytmem 2,2,2; ryt 2,3,2 to bug.
7. **Artefakty specyficzne trasy:** wobble (kamera lub pivot minimalnie różne między klatkami, postać „pływa"), cienie wypalone w klatce walczące z oświetleniem gry, miękkie krawędzie AA z renderu (bledną na tle pixel artu), zupa kolorów z pełnego renderu, kolory tła zlewające się z modelem przy kwantyzacji (w tutorialu D2 tło renderu to dokładnie 170,170,170, celowo dalekie od kolorów modelu, przy zlaniu się palety fragment obrazu staje się przezroczysty). Wszystkie wykrywalne tymi samymi bramkami liczbowymi co w dokumencie rysunkowym, plus test wiersza.
8. **Ekosystem narzędzi dla agentów już istnieje i jest dojrzały:** Sorceress 3D to 2D (FBX/GLB → arkusze sprite w przeglądarce, z normal mapami), skille agentowe typu puretty-sprites (koncept z image-gen → auto-rig → render z 8 kierunków w minuty) i 3d-to-pixel-sprite (odtworzenie modelu w headless Blender z bramką porównawczą do 0,95 podobieństwa z referencją), Blender MCP do sterowania sceną z rozmowy. Trasa jest dostępna dla agenta bez ręcznej pracy w edytorze 3D.

## Metodologia

- Kąty wyszukiwania: (1) produkcja oryginalnego Diablo 2 i technikalia formatów (DC6/DCC/COF, kierunki, palety 256 kolorów), (2) pipeline Dead Cells z deep dive'u artysty (Game Developer) i wywiadów, (3) współczesne pochodne w stylu D2 (Halls of Torment), w tym cytaty deweloperów o metodzie, (4) klasyczne korzenie trasy (DKC, AoE), (5) dobre praktyki renderu 3D→sprite (kamery, liczby klatek, typowe pułapki), (6) narzędzia i skille agentowe dla trasy, (7) reguły rzemiosła pixelowego dla twarzy, oczu i izometrii (baza Pixeli, walidowana już w dokumencie rysunkowym).
- Typy źródeł: primary sources od twórców (deep dive artysty Dead Cells, cytaty dewelopera Halls of Torment na Steam i Reddicie, wywiad Vicarious Visions o D2R), dokumentacja i tutoriale moddingowe D2 (Phrozen Keep, OpenDiablo2), dekodery formatów jako źródło strukturalne, poradniki pipeline 3D→2D (AssetHub, Sorceress, kursy), reguły rzemieślnicze (Pixeli, walidowane krzyżowo w dokumencie pierwszym).
- Ograniczenia: oryginalne parametry kamery D2 (dokładny pitch, ogniskowa) nie są znane z materiałów źródłowych Blizzarda, tylko z rekonstrukcji moddingowej. Skrypty renderujące Halls of Torment są niepubliczne, znane z opisów dewelopera. Post-processing Dead Cells po eksporcie z 3D nie jest w pełni udokumentowany.

## Wyniki

### 1. Linia krwi: od czego zaczęła się trasa i co z niej wynika

Donkey Kong Country renderował modele ze stacji Silicon Graphics do sprite'ów SNES. Age of Empires: „chociaż wszystkie obrazy na ekranie są 2D, postanowiliśmy wcześnie, że cała grafika powstanie z modeli 3D", 3D Studio i 3D Studio MAX w produkcji; każdy artysta miał renderer, bo render był wąskim gardłem. Diablo 2: postaci, potwory i część budynków jako modele 3D renderowane do arkuszy, próbkowane w poziomach detalu (n klatek × n kierunków), ekwipowanie przez paper doll, każdy przedmiot w każdej pozycji i rotacji, te same modele co do filmów przerywnikowych. Dead Cells: prosty model + szkielet w 3DS Max, render bez anti-aliasingu w rozmiarze docelowym, objętość przez normal mapy i toon shader. Halls of Torment: „sto procent 2D, wszystko działa na sprite'ach w przestrzeni 2D, tworzymy assety 3D w Blender i renderujemy je skryptami do sprite'ów", estetyka świadomie nawiązująca do pre-renderów końca lat 90. i początku 2000. Diablo 2 Resurrected: nałożenie modeli 3D na oryginalną logikę 2D, z przełącznikiem między trybami; zespół Vicarious Visions opisywał regułę 70/30: 70% wierności oryginalnym sprite'om i zachowanym assetom źródłowym, 30% nowego.

Wnioski przekrojowe dla agenta: (a) trasa jest powtarzalna i opisana liczbowo, nadaje się do automatyzacji, (b) w każdym udokumentowanym przypadku model był prosty, a ciężar jakości leżał na sylwetce, pozie i timingach, nie na wierności 3D, (c) kompozycja pipeline'u była zawsze ta sama: model → szkielet → pozy → sztywna kamera → render per kierunek → kwantyzacja → bramki.

### 2. Pipeline krok po kroku (z liczbami)

Krok 0, decyzje projektowe (raz na cały projekt): styl sprite'a i kamera (izometryczny: ortograficzna, stały pitch, kierunki co 45°), liczba kierunków (4 lub 8; w D2 potwory 8, gracze 16, DCC obsługuje 8/16/32), czy mirroring wchodzi w grę (połowa pracy, ale łamie asymetrie: blizna, naramiennik), spis akcji przed rigowaniem (rigging jest drogi, dokładanie akcji po), rozmiar docelowy sprite'a i wielokrotność renderu.

Krok 1, model: prosty, czytelna sylwetka, bryła ponad detal. Topologia i polycount bez znaczenia dla wyniku (wyjście to płaski obraz), znaczenie ma kształt, który przeżyje redukcję do setek pikseli. Test: render jednej klatki, downscale, oglądanie w 1×; detal, którego nie widać, się usuwa.

Krok 2, szkielet i pozy: animacja projektowana jak 2D, pose-to-pose. Pozy kluczowe na szkieletie, minimalna liczba klatek, sprawdzenie czytelności samych póz, dopiero potem klatki interpolacyjne i to tylko przed lub za pozami kluczowymi (recepta Dead Cells, omówiona w sekcji 4). Timing bez osobnych reguł dla tej trasy, liczby z dokumentu rysunkowego mają zastosowanie: idle 200-400 ms, walk 100-150 ms z holdem na contact, atak 40-60 ms na uderzeniu i hold 150-250 ms.

Krok 3, scena renderu (raz na cały zestaw): kamera ortograficzna (perspektywa zmienia kształt przy ruchu i psuje zgodność kafli), kamera przykręcona na stałe, model centrowany ze stałym paddingiem w każdej klatce, tło przezroczyste, cień nie wypalony w klatce (albo osobna warstwa cienia), światło o ustalonej konwencji: albo obraca się z modelem (efekt kamery orbitującej, światło zawsze z tej samej strony ekranu, to konwencja D2 i klasycznego izometrycznego pixel artu), albo stoi w scenie (efekt świata z własnym słońcem). Konwencję wybiera się raz i nie miesza w jednym zbiorze.

Krok 4, render per kierunek: obrót modelu co 45° (w D2 kierunek 0 to dół-lewo ekranu, kolejne 45° zgodnie z ruchem wskazówek zegara; potwory 0-7, postacie i pociski więcej), rozmiar klatki pod limitem docelowego formatu (w D2: 256×256 na część animacji, DCC praktycznie do ~200×200; w ogólnym pipeline: render nad rozmiar docelowy, downscale później), tło oddalone chromowo od kolorów modelu (w D2 170,170,170), żadnego anti-aliasingu w renderze, żadnych filtrów.

Krok 5, redukcja i kwantyzacja: downscale wyłącznie całkowitoliczbowym nearest-neighbour, snap do palety projektu (limit kolorów jak w dokumencie rysunkowym: ≤16 przy małych sprite'ach, ≤32 przy większych), alfa binarna (0 lub 1, bez półprzezroczystości), ewentualne wygenerowanie konturu selektywnego na tym etapie.

Krok 6, bramki (omówione w sekcji 6), eksport: arkusz + atlas JSON + czasy klatek + kierunki + pivoty, nazewnictwo zgodne z silnikiem, jedna akcja całkowicie doprowadzona do gry zanim zrenderuje się cały zestaw (test jednego sprite'a w grze to najtańsza kontrola całego pipeline'u).

Błędy typowe z tutoriali D2 (dziennik krytycznych): fragment obrazu przezroczysty = kolor tła zlał się z kolorem modelu w palecie; brak przezroczystości = źle ustawiony kolor przezroczystości; potwór chodzi tyłem = złej kolejności kierunki; dziwna liczba klatek (83 zamiast 128) = popsuty jeden kierunek.

### 3. Dobre modele pod sprite

- Sylwetka ponad wszystko: model ocenia się po tym, czy jego zarys czyta się w 1×. Zasady z dokumentu rysunkowego przenoszą się wprost: brak idealnej symetrii pozy, brak stycznych (kontur ramienia dotykający konturu torsu zlewa się w blob), zróżnicowana grubość kończyn.
- Bryła: duże czytelne płaszczyzny wartości (jasna strona światła, średnia, ciemna) zamiast miękkich gradientów; przy downscale'u gradient staje się szumem, a trzy wartości zostają trzema wartościami. Dead Cells wydobywa objętość przez normal mapy renderowane z animacji i toon shader; to tania imitacja rzemiosła: objętość zamiast fotorealizmu.
- Charakterowość ponad realizm: w D2 modele były „wystarczająco dobre", bo w grze liczyły się piksele. Model ma komunikować: role (gruba masa potwora, sucha sylwetka nieumarłego), skalę względem kafli, punkt słabości czytelny z sylwetki (w D2 klimat robi też to, że broń i zbroje są przewymiarowane, żeby w ogóle były widoczne po redukcji).
- Reuse jako trik ekonomiczny: Dead Cells oszczędzało setki godzin doklejając pancerz do modelu i reuse'ując części starych modeli do nowych potworów. Dla agenta to naturalna strategia: biblioteka modularnych części (torsy, głowy, kończyny, bronie) zamiast modelowania od zera.

### 4. Animacja: płynność i sensowność w tracie pre-renderu

**Sensowność zaczyna się od listy akcji i pozy kluczowej.** Dead Cells: animacje projektowane jak 2D, pozy kluczowe dopóki ruch nie jest przekonujący i dobrze wytimingowany przy minimalnej liczbie klatek; klatki interpolacyjne dodawane przed lub za pozami kluczowymi, nigdy pomiędzy (wniosek praktyczny: te klatki interpolacyjne to odpowiednik in-betweenów, ale wstawiane tylko na krańcach, żeby nie rozmyć energii póz kluczowych). Poczucie ruchu, uderzenia i siły dają VFX, nie klatki: to jest trik numer jeden tej trasy.

**Timingi i pozy**: te same reguły co w pixel arcie rysowanym (dokument pierwszy, sekcja animacji): contact/down/pass/up w chodzie, anticipation → smear → hit → follow-through → recovery → overshoot w ataku, atak gracza bez windupu, potwór telegrafujący 150-400 ms. Różnica: w pre-renderze pozę i timing poprawia się w 3D, przesuwając keyframes, bez dotykania pikseli; retake timingów to minuty.

**Smear i motion blur:** w erze D2 smeary nie występowały (klimat bardziej ciężki, mechaniczny), Dead Cells buduje dynamikę przez VFX i blend modes na eksportowanych klatkach. W pre-renderze smear można uzyskać dwójkowo: klasycznie dorysować w pikselach (reguły z dokumentu pierwszego) albo wyrenderować rozmycie kierunkowe tylko na wybranych klatkach i doprowadzić do binarnej alfy. Druga metoda jest ryzykowna (łatwo o błoto), domyślnie lepszy jest smear dorysowany.

**Kierunki i mirroring:** 8 kierunków pokrywa pełny obrót co 45°; trzy pary kierunków powstają przez mirroring (lewo-prawo), więc realny koszt renderu to często 5 póz: pełny front, pełny tył i trzy pozostałe (dwa skosy plus profil). Pułapka: mirroring łamie asymetrie (blizna zmienia stronę, broń w lewej ręce skacze do prawej); decyzję o mirroringu podejmuje się przed modelowaniem, nie po.

**Płynność w silniku:** płynność odczuwa gracz, nie widz GIFa. Klatki tagowane, timingi w danych, retiming bez rerenderu; pętla idle na innym okresie niż mrugnięcie (2-4 s) i oddech. Kontrola krzyżowa: stała wysokość, stała grubość, stałe światło między kierunkami i klatkami (w pre-renderze łapie to bramka wobble, sekcja 6).

**Wobble, wróg numer jeden tej trasy:** minimalne różnice pozycji kamery, pivota albo osi obrotu między klatkami sprawiają, że postać „pływa" na ekranie, artefakt praktycznie niewidoczny w pojedynczej klatce i bardzo widoczny w ruchu. Lekarstwo: kamera przykręcona, pivot w stałym punkcie stóp, centrowanie każdej klatki skryptem, bramka porównująca środek masy między klatkami.

### 5. Twarz, która nie jest ziemniakiem

Problem ma dwie warstwy: model i piksele. W 3D ziemniak powstaje z płaskiego modelu twarzy i płaskiego światła. Leki modelowe:

- Zdefiniowane płaszczyzny: czoło, oczodoły wciśnięte pod łuki brwiowe, grzbiet nasady nosa, kości policzkowe, żuchwa. Normal mapy i toon shader (Dead Cells) wzmacniają te płaszczyzny przy renderze, objętość twarzy jest w płaszczyznach, nie w detalu.
- Światło kierunkowe pod stałym kątem buduje objętość (kierunkowy key light, nie ambient): w izometrycznym sprite'ie światło zgodne z konwencją sceny (góra-lewo ekranu), twarz pokazuje tę samą logikę co reszta.
- Oczy w modelu zawsze modelowane, nawet jeśli w sprite'ach znikną: przy większych rozmiarach sprite'a (Halls of Torment, portrety D2) to one niosą człowieczeństwo.

Warstwa pikselowa (finalny sprite i tak jest mały, więc reguły pixelowe decydują, walidowana baza reguł):

- Oczy dostają piksele pierwsze i nigdy nie są tym, co się tnie. Drabinka rozmiaru oka względem wysokości głowy: głowa 8 px → oko 1×1 albo brak, 16 px → 2×2, 24 px → 3×3, 32 px → 5×4; gdy brak miejsca nawet na 1 px, okolica oczu to cień sugerujący oko, nie subpikselowa plamka.
- Linia oczu na 50-56% wysokości głowy (mierzonym od czubka włosów), punkty charakterystyczne jako ułamki wysokości: brwi 38-42%, nasada nosa w połowie brwi-podbródek, usta 77-83%.
- Cechy nigdy się nie dotykają: minimum 1 px skóry między ustami a nosem, brwią a okiem, okiem a konturem; minimum 2 px między oczami (1 px zlewa je w jedną brew).
- Biel oczu dopiero od 3 px szerokości oka, i to nie biel, tylko najjaśniejszy ton skóry lub blada szarość; jeden catch-light na górze-lewo ucznia, w tym samym punkcie obu oczu; źrenica o 1 px od środka (wyśrodkowana na twarzy frontowej czyta się jak martwe spojrzenie).
- Brwi niosą emocje przechyłką 1 px: wewnętrzny koniec w dół to złość, w górę smutek, uniesiona brew ze zdziwieniem; symetrię wyrazu psuje się o 1 px (idealna symetria czyta się jak maska).
- Skóra na trzy tony (światło, baza, cień), cień pod brwiąmi i w oczodole, nie koło konturu (to poduszka, znamię sztuki początkującej i maszynowej).
- Przy ≤16 px wysokości głowy twarzą są włosy i kaptur: minimum 2 px włosów po bokach, linia włosów 1-2 px nad oczami, włosy od skóry oddzielone kolorem, nie linią. Diablo 2 rozwiązywało twarze bohaterów właśnie tak: kaptury, hełmy, kościane maski; czytelność sylwetki głowy była ważniejsza niż rysy.
- Profil potrzebuje profilu: czoło na zewnątrz, oko w środku, nos na zewnątrz, usta, broda, minimum trzy garby na przednim konturze; prosta krawędź frontu to najsłabsza sylwetka.

Triki pokrewne: portrety i twarze w dużym zbliżeniu (ekran ekwipunku, dialogi) renderuje się z osobnej wersji głowy o wyższym poziomie detalu; twarz w rozmiarze gry nie ma nic wspólnego z twarzą w porciecie i to jest w porządku.

### 6. Bramki i artefakty trasy pre-renderu

Znamiona do wykrycia skryptem (rozszerzenie katalogu z dokumentu rysunkowego):

- **Wobble:** środek masy lub pivot różny między klatkami; bramka porównuje barycentrum maski alfa klatka do klatki, tolerancja 0-1 px poza zamierzonym squash.
- **Mixels i ślady skalowania:** test wiersza (długości przebiegów jednego rzędu pikseli, wspólny dzielnik >1 to plik pre-skalowany, brak dzielnika to mixels z błędnego downscale'u); downscale wyłącznie całkowitoliczbowy NN.
- **Zupa kolorów:** licznik unikalnych kolorów i near-duplikatów (ΔE<3) po kwantyzacji; snap palety obowiązkowy po każdym renderze.
- **Półprzezroczysta alfa:** zero przypadków; render z AA wygładza krawędzie do stu poziomów alfy, trzeba wymusić binarną alfa i/lub render bez AA.
- **Cień wypalony:** cień w klatce wykryty jako ciemna obwódka stykająca się z linią gruntu we wszystkich kierunkach; cień albo osobna warstwa, albo wcale (blob od silnika).
- **Przełączenie światła między kierunkami:** kierunek jasny/ciemny niespójny między kierunkami; bramka histogramu jasności sylwetki per kierunek, szeroka rozbieżność to sygnał, że konwencja światła się zmieniła.
- **Gotowanie się animacji:** piksele migoczące między klatkami poza zamiarem (zostawiony noise z renderu); bramka porównania klatek idle: różnica pikselowa poza obszarem ruchu powinna być bliska zeru.

Kolejność napraw jak w dokumencie rysunkowym: skala → paleta → sylwetka i światło → klastery → krawędzie → detale, wszystko na natywnej skali, raport liczbowy przed/po. Przegląd agenta: filmstrip (model wzrokowy czyta pierwszą klatkę GIF), dodatkowo w tej trasie wartościowo: porównanie klatek i kierunków w jednym pasie, żeby wobble i przełączenie światła było widoczne gołym okiem.

### 7. Krajobraz narzędzi dla agentów (trasa 3D → sprite)

| Narzędzie / skill | Co robi | Uwagi |
| --- | --- | --- |
| Blender + skrypty Python (wzorzec Halls of Torment) | Assety 3D renderowane skryptami do sprite'ów, automatyzacja całości | Wzorzec produkcyjny udokumentowany cytatami dewelopera; skrypty niepubliczne, do odtworzenia |
| Sorceress 3D to 2D | Import FBX/GLB/GLTF z animacjami, ortograficzny viewport, arkusze sprite, normal mapy, metadane | Działa w przeglądarce, gotowa alternatywa dla własnych skryptów |
| Lowkey333/puretty-sprites | MCP + pipeline'y: koncept z ComfyUI, TRELLIS do modelu, blender_auto_rig (szkielet + 8 animacji w kilkanaście sekund), blender_render_sprites | Najbliższy gotowy stack agentowy dla tej trasy; lokalny |
| atou42/3d-to-pixel-sprite | Skill agentowy + skrypty: odtworzenie sprite'a jako modelu 3D w headless Blender i render z powrotem do pixel artu zgodnego z paletą i sylwetką oryginału, bramka compare 0,95 | Trasa odwrotna (sprite → 3D → sprite), przydatna do remasterów i iteracji na istniejących assetach |
| Blendkit Pixel Kit | One-click pixelacja renderu Blendera: siatka, kontury, retro palety, niedestruktywnie w kompozytorze | Szybkie prototypowanie wyglądu; bramki nadal poza nim |
| Modly sprite-pipeline | 6 węzłów: generacja 3D (Hunyuan3D, TripoSG, Trellis) → gotowe sprite'y pixel artowe | Trasa z image-gen 3D, wymaga pełnego wykończenia jak w dokumencie pierwszym |
| Blender MCP (sterowanie agentem) | Agent introspektuje scenę i buduje przez API z rozmowy | Do sklejania własnych pipeline'ów; odpowiednik „edytora w pętli" z trasy rysunkowej |
| Kursy i poradniki (GameDev.tv, QWeb) | Kamery orto/perspektywa per typ gry, rig izometryczny w Blenderze | Materiał edukacyjny do zapożyczenia reguł do skilla |

### 8. Blueprint: skille agenta dla trasy pre-renderu

Architektura skilla (dopasowana do wzorca z dokumentu rysunkowego, sekcja 7):

1. **Plik reguł trasy:** sekcje 2-6 tego dokumentu jako reguły z liczbami (pitch kamery stały, 45° na kierunek, downscale całkowitoliczbowy, alfa binarna, tło oddalone chromowo).
2. **Brief projektowy:** gatunek, kamera, liczba kierunków, mirroring tak/nie, rozmiar docelowy, paleta, lista akcji przed rigowaniem, 2-6 kryteriów ukończenia.
3. **Bramki per etap:** test sylwetki modelu w 1× przed rigowaniem, filmstrip póz kluczowych przed interpolacją, walidator po downscale (kolory, alfa, wobble), test jednego sprite'a w grze przed pełnym renderem zestawu.
4. **Pętla render → spojrzenie → poprawka:** minimum dwa przebiegi, poprawki póz i timingów w 3D (keyframes), nie w pikselach.
5. **Eksport:** arkusz + atlas (kierunki, czasy, pivoty) + osobne normal mapy jeśli silnik ich używa do efektów.

Gotowy zestaw reguł do wklejenia w system prompt lub plik skilla (skondensowany):

- Kamera ortograficzna, przykręcona; obraca się model, nie kamera; kierunki co 45°; kamera i padding identyczne w każdej klatce.
- Konwencja światła wybrana raz: światło obraca się z modelem (kamera orbitująca) albo stoi w scenie; nie mieszać w jednym zbiorze assetów.
- Model prosty, bryła i sylwetka ponad detal; test czytelności w 1× przed rigowaniem; detal znikający w downscale'u usuwasz, nie renderujesz.
- Animacja pose-to-pose w 3D: pozy kluczowe i timing dopóki ruch nie czyta się przy minimalnej liczbie klatek; interpolacja tylko przed lub za pozami kluczowymi; retake timingów przez przesunięcie keyframes.
- Liczby: idle 200-400 ms/klatkę, walk 100-150 ms z holdem na contact, atak gracza bez windupu, hit 40-60 ms, hold 150-250 ms, recovery dłuższy niż swing, potwór telegrafuje 150-400 ms.
- Render: bez AA, bez filtrów, tło przezroczyste i oddalone chromowo od modelu, cień nie wypalony w klatce.
- Downscale całkowitoliczbowym nearest-neighbour, snap do palety, alfa binarna, zero półprzezroczystych pikseli.
- Twarz: płaszczyzny i światło w modelu, oczy pierwsze w pikselach, biel od 3 px szerokości oka, jeden catch-light, cechy nie dotykają się, symetria psuta o 1 px, przy małych głowach twarzą są włosy/kaptur.
- Izometria: krawędź 2:1, piony pionowe, trzy wartości ścian, jedno światło sceny; kafl diament W×W/2.
- Bramki po każdym renderze: barycentrum masy klatka do klatki (wobble), unikalne kolory i ΔE<3 (paleta), binarna alfa, histogram jasności per kierunek (światło), różnica idle między klatkami (gotowanie).
- Przeglądaj filmstrip i pas kierunków, nie GIF; jeden sprite doprowadź do gry zanim zrenderujesz cały zestaw; oddajesz z raportem liczb przed/po.

## Notatki o źródłach

| Źródło | Wiarygodność | Ostatnia aktualizacja |
| --- | --- | --- |
| [Game Developer: Using a 3D pipeline for 2D animation in Dead Cells (Thomas Vasseur)](https://www.gamedeveloper.com/production/art-design-deep-dive-using-a-3d-pipeline-for-2d-animation-in-i-dead-cells-i-) | 5/5 | 2018-01-25 |
| [Nintendo Life: wywiad z Motion Twin (Steve Filby)](https://www.nintendolife.com/news/2018/05/feature_reanimating_the_roguelike_with_dead_cells_developer_motiontwin) | 4/5 | 2018-05 |
| [Phrozen Keep: TUTORIAL How to make a 3D model be used by Diablo II](http://d2mods.info/forum/viewtopic.php?t=14662) | 4/5 | 2003 |
| [Phrozen Keep: Monster Converting Tutorial (Alkalund)](https://d2mods.info/forum/kb/viewarticle?a=152) | 4/5 | - |
| [Phrozen Keep: some stats on .cof](https://d2mods.info/forum/viewtopic.php?t=2714) | 3/5 | - |
| [r/pathofdiablo: technical insight how sprites are rendered in D2](https://www.reddit.com/r/pathofdiablo/comments/8k37tj/could_someone_give_me_some_technical_insight_as/) | 3/5 | - |
| [OpenDiablo2/dc6 (GitHub, dekoder formatu)](https://github.com/OpenDiablo2/dc6) | 4/5 | - |
| [Halls of Torment, Steam: How did they do the graphics (cytat dewelopera)](https://steamcommunity.com/app/2218750/discussions/0/3803901984159893279/) | 3/5 | - |
| [r/gamedev: pre-rendered sprites (cytat z Halls of Torment)](https://www.reddit.com/r/gamedev/comments/1hsnrin/whats_the_best_way_to_implement_prerendered/) | 3/5 | - |
| [r/hallsoftorment: nazwa stylu i proces](https://www.reddit.com/r/hallsoftorment/comments/18d52fy/could_someone_tell_me_what_is_the_name_of_the/) | 2/5 | - |
| [Wikipedia (EN): Diablo II Resurrected](https://en.wikipedia.org/wiki/Diablo_II:_Resurrected) | 4/5 | - |
| [FTW/USA Today: Q&A Vicarious Visions o D2R, reguła 70/30](https://ftw.usatoday.com/story/tech/gaming/2021/09/22/diablo-2-resurrected-vicarious-visions/81449421007/) | 3/5 | 2021-09-22 |
| [Medium: Donkey Kong Country, pre-render na SGI](https://medium.com/@newplayerready/donkey-kong-country-how-pre-rendered-graphics-saved-the-snes-late-in-its-life-new-player-ready-90a07cb2b76f) | 2/5 | - |
| [r/aoe2: Are aoe2 buildings originally 3d models (cytat dewelopera AoE)](https://www.reddit.com/r/aoe2/comments/3clb8r/are_aoe2_buildings_originally_3d_models/) | 3/5 | - |
| [AssetHub: Turning AI 3D Models into 2D Sprites](https://assethub.io/blog/3d-to-2d-sprites-ai-assets) | 3/5 | 2026-09-12 |
| [Sorceress: 3D to 2D, konwerter](https://sorceress.games/pages/3d-to-2d) | 3/5 | - |
| [Sorceress Wiki: 3D to 2D, specyfikacja](https://sorceress.games/wiki/sorceress/3d-to-2d) | 3/5 | - |
| [GameDev.tv: Blender 2D Sprites (kurs, kamery per typ gry)](https://gamedev.tv/courses/blender-sprites) | 2/5 | - |
| [QWeb: Creating an isometric rig in Blender](https://www.qweb.co.uk/blog/creating-an-isometric-rig-in-blender) | 2/5 | 2018-09-22 |
| [r/gamedev: 3D Animations in isometric game](https://www.reddit.com/r/gamedev/comments/aua8i8/3d_animations_in_isometric_game/) | 2/5 | - |
| [Pixeli: heads and faces](https://pixeli.pebbly.space/knowledge/rules/32-heads-and-faces) | 4/5 | - |
| [Pixeli: eyes and expressions](https://pixeli.pebbly.space/knowledge/rules/33-eyes-and-expressions) | 4/5 | - |
| [Pixeli: isometric (2:1)](https://pixeli.pebbly.space/knowledge/rules/71-isometric) | 4/5 | - |
| [Pixeli: sylwetka i bryła](https://pixeli.pebbly.space/knowledge/rules/03-silhouette-and-form) | 4/5 | - |
| [Pixeli: animacja](https://pixeli.pebbly.space/knowledge/rules/05-animation) | 4/5 | - |
| [Pixeli: ataki i trafienia](https://pixeli.pebbly.space/knowledge/rules/44-attacks-and-impacts) | 4/5 | - |
| [Pixeli: znamiona sztuki generowanej](https://pixeli.pebbly.space/knowledge/rules/92-generated-art-tells) | 4/5 | - |
| [Lowkey333/puretty-sprites (GitHub)](https://github.com/Lowkey333/puretty-sprites) | 3/5 | - |
| [atou42/3d-to-pixel-sprite (GitHub)](https://github.com/atou42/3d-to-pixel-sprite) | 3/5 | - |
| [Blendkit: Pixel Kit, pixelacja renderu Blendera](https://www.blendkit.com/asset-gallery-detail/2afada68-f363-438e-8beb-7d3b12129594/) | 2/5 | - |
| [Modly: sprite-pipeline](https://modly3d.app/extensions/modly-sprite-pipeline) | 2/5 | - |
| [Saved Pixel: Blender Pixel Art guide](https://savedpixel.com/blog/blender-pixel-art) | 2/5 | - |
| [skills.cat: godot-adapt-3d-to-2d](https://skills.cat/skills/thedivergentai/gd-agentic-skills/godot-adapt-3d-to-2d) | 2/5 | - |

Konflikty i zastrzeżenia:

- Liczba kierunków w D2: tutoriale moddingowe mówią o 8 kierunkach potworów i 16 dla postaci gracza (COF), dekodery potwierdzają obsługę 8/16/32; niektóre odpowiedzi forowe upraszczają do "8". W dokumencie przyjęto: potwory 8, gracze 16, jako konsensus źródeł moddingowych.
- Światło przy obrocie: tutorial D2 wskazuje, że przy obrocie kamery światło musi się obracać razem z nią, inaczej oświetlenie „przełącza się" między kierunkami; przy obrocie modelu zamiast kamery ten konkretny problem znika, ale wybór konwencji światła (obraca się z modelem czy stoi w scenie) pozostaje do decyzji, patrz krok 3. Obie konwencje spotykane w źródłach; dokument wymaga świadomego wyboru jednej.
- Ziemniak twarzy: reguły pikselowe pochodzą z bazy rzemieślniczej walidowanej w dokumencie pierwszym; warstwa modelowa (płaszczyzny, toon shader, normal mapy) z deep dive'u Dead Cells; nie znaleziono źródła opisującego modelowanie twarzy oryginalnego D2 (materiały produkcyjne Blizzarda niepubliczne).

## Otwarte pytania

- Dokładne parametry kamery oryginalnego D2 (pitch, dystans, czy pełne 2:1) nie są znane z materiałów źródłowych; rekonstrukcje moddingowe zakładają klasyczny widok izometryczny z pitchem zbliżonym do 30-45°, bez potwierdzenia.
- Post-processing Dead Cells po eksporcie (ręczne poprawki klatek? snap palety? kto i kiedy) nie jest udokumentowany.
- Skrypty renderujące Halls of Torment są niepubliczne; znany jest tylko opis procesu od dewelopera. Do odtworzenia: skrypty Blendera (model → 8 kierunków → arkusz), wzorce z AssetHub i Sorceress.
- Czy twarze postaci D2 miały osobne modele do portretów (prawdopodobne, niepotwierdzone).
- Reguły widoku izometrycznego (2:1) są sformalizowane dla pixel artu rysowanego; ile z nich wprost przenosi się na render ortograficzny z modelu (a ile robi za niego kamera), wymaga testu na własnym pipeline.

## Rekomendacje / następne kroki

1. Zbuduj pipeline jako kod od pierwszego dnia: skrypt renderujący (Blender headless albo Sorceress), skrypt downscale+snap, walidator z bramkami z sekcji 6; agent steruje skryptami, nie klika w edytorze.
2. Decyzje projektowe w briefie przed modelem: kierunki (8 domyślnie), mirroring, rozmiar docelowy, konwencja światła, lista akcji; zmiana którejś z nich po rigowaniu kosztuje pełny rerender.
3. Twarze planuj per rozmiar: poniżej ~16 px wysokości głowy twarzą jest kaptur i włosy, powyżej drabinka oczu z sekcji 5; portret to osobny, wysoko detalowy render, nie wycinek sprite'a gry.
4. Animację waliduj na filmstripie póz kluczowych zanim zrenderujesz interpolacje; poprawki póz i timingów rób w 3D przez keyframes.
5. Jeden sprite (idle, 1 kierunek) przeprowadź całą trasą do silnika zanim wyrenderujesz cokolwiek więcej; lista problemów z tego przebiegu to lista bramek do wpisania w walidator.
6. Prowadź bibliotekę modularną (torsy, głowy, kończyny, bronie) i reuse'uj części między potworami, to największa znana oszczędność czasu na tej trasie (Dead Cells: setki godzin).
7. Reguły wspólne z dokumentu rysunkowego (sylwetka, paleta, artefakty, raport przed/po, dwa przebiegi) stosuj bez zmian: pre-render zmienia sposób produkcji pikseli, nie standardy ich jakości.