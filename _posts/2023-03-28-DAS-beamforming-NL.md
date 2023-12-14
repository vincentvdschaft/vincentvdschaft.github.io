# Ultrasound beeldvorming met Delay-And-Sum beamforming
Veel mensen kunnen zich wel een voorstelling maken van hoe een röntgenfoto gemaakt wordt, maar hoe een ultrasound afbeelding tot stand komt uit geluid is minder bekend. In deze blogpost laat ik zien hoe de meestgebruikte vorm van ultrasound imaging werkt. Deze techniek heet **Delay-And-Sum beamforming** of DAS in het kort.

Laten we eerst kijken naar wat er fysiek gebeurt bij het maken van een echo en naar wat we eigenlijk aan het meten zijn. Om een plaatje te maken met ultrasound wordt er gebruikgemaakt van een _transducer_. Dit is een meetinstrument met daarin een rij sensors die geluidsgolven kunnen uitzenden en opvangen. Om een beeld te vormen wordt eerst een geluidsgolf uitgezonden. De geluidsgolf beweegt door het weefsel heen waar het op verschillende structuren reflecteert. Dit proces van reflecteren is super complex, maar gelukkig bestaat er een simpel model voor:

**Je kan het weefsel zien als een ruimte waarin een groot aantal punten liggen die een geluidsgolf absorberen en dan naar alle kanten uitzenden.**

Deze punten worden _verstrooiers_ genoemd (omdat ze het geluid alle kanten op verstrooien). Dit is slechts een model en kan niet alle aspecten van ultrasound verklaren, maar voor ons is het nu goed genoeg.

De reflecties afkomstig van de verstrooiers worden opgevangen door de elementen van de transducer. De signalen die de elementen ontvangen worden gecombineerd tot een plaatje dat laat zien hoe sterk de reflecties van elke plek in de afbeelding zijn. Dit proces waarbij je van losse geluidssignalen naar een plaatje gaat wordt _beamforming_ genoemd.

## Scenario 1 - één sensor, één verstrooier

Om het beamforming proces te begrijpen kijken we eerst naar een sterk versimpeld scenario: We hebben een transducer met maar 1 sensor element $E_0$ en we scannen een plaatje met maar 1 verstrooier $V_0$ er in.

<details open>
    <summary>(klik om in te klappen)</summary>
<img src="{{ 'assets/images/scene-0-just-one-nl-dark.gif' | relative_url }}" style="border:none;"/>
<p></p>
</details>

In het rechter plaatje zie je hoe de geluidsgolf zich voortplant vanuit $E_0$ en vestrooid wordt door $V_0$. In de linker grafiek zie je het geluidssignaal dat wordt ontvangen door $E_0$. Aan het signaal dat binnenkomt kunnen we twee dingen afleiden:

1. Omdat er een piek in het signaal zit weten we dat er een verstrooier aanwezig moet zijn.
2. Omdat we weten hoe snel een geluidsgolf beweegt kunnen we aan tijd tussen verzenden en ontvangen ook afleiden **hoe ver weg** de verstrooier zich bevindt.

<details>
  <summary>🧮 Uitwerking (klik om uit te klappen)</summary>
<p>
Het signaal is uitgezonden op $t=0$ en de reflectie wordt ontvangen op $t_r=15\mu s$. De snelheid van het geluid in weefsel $c$ is ongeveer $1540 m/s$. De afstand $d_{totaal}$ die de golf heeft afgelegd in die tijd is dus
$$d_{totaal}=t_r\cdot c=15\cdot 10^{-6}\cdot 1540=23.1 cm$$
Dit is de afstand van de heenweg en de terugweg samen. De verstrooier ligt dus half zo ver, oftewel: De afstand tussen de verstrooier en de sensor is $11.55cm$.
</p>
</details>

Maar we zijn niet alleen geïnteresseerd in **hoe ver weg** de verstrooier is. We willen weten **waar** die is. Daar gaan we op deze manier helaas nooit achter komen. Dat kunnen we zien aan dit alternatieve scenario: Als $V_0$ op een andere plek staat, maar wel even ver weg is van de sensor meten we precies het zelfde signaal! Weer zit de piek op precies $15\mu s$.

<details open>
    <summary>(klik om in te klappen)</summary>
<img src="{{ 'assets/images/scene-1-rotated-nl-dark.gif' | relative_url }}" style="border:none;"/>
<p></p>
</details>

Dit betekent dat de reflecties van alle verstrooiers die op een circkel liggen tegelijkerijd aankomen bij $E_0$. We kunnen dus geen onderscheid maken tussen de verschillende verstrooiers. Je ziet hier ook dat de gemeten piek nu veel hoger is. De reflecties van alle verstrooiers tellen hier bij elkaar op. (Daar komen we later op terug.)

<details open>
    <summary>(klik om in te klappen)</summary>
<img src="{{ 'assets/images/scene-2-circle-nl-dark.gif' | relative_url }}" style="border:none;"/>
<p></p>
</details>

Om onszelf in staat te stellen te meten **waar** de verstrooier zich bevind hebben we meer informatie nodig. Die informatie kunnen we krijgen door extra sensoren toe te voegen op andere locaties.

## Scenario 2 - Meer elementen

We nemen nu twee sensoren in plaats van één. Opnieuw zend element $E_0$ een geluidsgolf uit. De sensoren vangen het zelfde signaal op, maar met verschillende vertragingen. Op basis van de vertragingen kunnen we net als eerder voor element $E_0$ berekenen hoe ver weg de verstrooier zich bevind. Nu kunnen we ook voor $E_1$ een ring vinden met mogelijke locaties. Met die twee samen kunnen we vinden waar de verstrooier zich precies bevindt: _Op het snijpunt van de twee ringen._
(Eigenlijk is het geen ring, maar een ellips. Zie het uitwerking blok voor toelichting.)

<details>
  <summary>🧮 Uitwerking (klik om uit te klappen)</summary>
<p>

Voor $E_0$ is kunnen we de afstand berekenen zoals eerder: De tijd waarna we de piek opvangen met sensor $E_0$, $\tau_0$, is de tijd van $E_0$ naar $V_0$ en terug. De totale afstand tussen $E_0$ en $V_0$ is dus $$d_{heen}+d_{terug}=c\cdot \tau_0$$
De heen- en terugweg zijn hier even lang dus alle mogelijke locaties voor de verstrooier zijn de locaties waarvoor geld dat de afstand tot $E_0$ gelijk is $\frac{1}{2}\cdot c \cdot \tau_0$.
<br>
<br>
Voor $E_1$ is het net anders omdat de puls niet vanuit $E_1$ is verzonden. Dit betekent dat $d_{heen}$ en $d_{terug}$ niet meer gelijk aan elkaar hoeven te zijn. Als de piek op tijdstip $t_{r1}$ ontvangen wordt weten we dat het geluid in die tijd de afstand van $E_0$ naar $P$ heeft afgelegd (oftewel $d_{heen}$) en daarna de afstand van $P$ naar $E_1$ (oftewel $d_{terug}$). De totale afstand is dus $$d_{heen}+d_{terug}=c\cdot \tau_1$$
De mogelijke locaties voor de verstrooier zijn nu dus de locaties waarvoor deze vergelijking geld. Als je dit oplost vind je dat de verstrooier ergens op een ellips rondom $E_0$ en $E_1$ moet liggen.

</p>
</details>

<details open>
    <summary>(klik om in te klappen)</summary>
<img src="{{ 'assets/images/scene-3-multiple-sensors-nl-dark.gif' | relative_url }}" style="border:none;"/>
<p></p>
</details>

## Delay-And-Sum beamforming

We hebben nu gezien dat je met een sensor kan bepalen hoe ver een verstrooier weg is en dat je met twee of meer sensors kan zien waar een verstrooier zich bevindt. Het probleem is alleen dat we in de praktijk niet kunnen aannemen dat er maar 1 verstrooier is. Dat is namelijk letterlijk nooit het geval. In de praktijk zijn er heel veel verstrooiers die allemaal 'door elkaar praten'. Laten we kijken naar een voorbeeld dat wat dichter bij de realiteit ligt: We nemen $9$ verstrooiers op willekeurige plekken en meer dan twee sensoren:

<details open>
    <summary>(klik om in te klappen)</summary>
<img src="{{ 'assets/images/scene-5-many-sensors-scatterers-nl-dark.gif' | relative_url }}" style="border:none;"/>
<p></p>
</details>

De signalen zien er nu veel chaotischer uit en het is zonder het rechter plaatje niet meer duidelijk welke piek bij welke verstrooier hoort. Hierdoor kunnen we niet meer zo simpel bepalen waar de verstrooiers zich bevinden.

Om dit probleem op te lossen gebruiken we _Delay-And-Sum beamforming_. Daarbij gaan we eigenlijk het tegenovergestelde doen van wat we eerder deden: In plaats van te kijken waar de piek in het signaal zit en de afstand uit te rekenen, gaan we een **afstand kiezen** om op te focussen en de **bijbehorende vertraging berekenen**. Door in het signaal van sensor $E_0$ te kijken naar een specifiek tijdstip **focussen** we op de ellips van alle punten waarvan de reflecties op dat tijdstip aankomen. De waarde die we daar meten is dus de som van alle verstrooiers op die ellips.

We kiezen nu een focuspunt $P$. We willen weten hoe sterk de reflecties vanaf punt $P$ zijn. We kunnen **focussen** op punt $P$ door bij elke sensor in het opgevangen signaal te kijken voor het moment dat de reflecties vanuit $P$ precies bij die sensor aankomen. (Dit is het _delay_ deel.) Zo vinden we voor elke sensor $E_i$ een waarde die ons vertelt hoe sterk de reflecties van zijn ellips door $P$ zijn. Als we deze waardes allemaal bij elkaar optellen komen we uit op een schatting voor hoe sterk de reflectie vanuit punt $P$ is. (Dit is het _sum_ deel.)

<details>
  <summary>🧮 Uitwerking (klik om uit te klappen)</summary>
<p>
Als we een focuspunt $P$ hebben gekozen kunnen we als volgt de bijbehorende vertraginen en daarmee de locaties in de de opgevangen signalen vinden.
Voor elk element $E_i$ kunnen we de totale afstand uitrekenen die de golf af heeft moeten leggen om van $E_2$ (de verzender) naar $P$ en dan naar $E_i$ te reizen. De totale afstand bestaat dus uit een afstand heen $d_{heen}$ en een afstand terug $d_{terug}$. De vertraging $\tau_i$ die hierbij hoort is
$$\tau_i=\frac{d_{heen}+d_{terug}}{c}$$
De berekende vertragingen zijn aangegeven in het onderstaande plaatje.
</p>
</details>

Het plaatje hieronder laat zien wat er gebeurt als je op deze manier op verschillende locaties $P$ kijkt en alle signalen optelt. Aan de gestippelde lijnen in de linker grafiekjes zien we welk deel van het ontvangen signaal gebruikt wordt om op $P$ te focussen. De balk aan de rechterkant laat zien hoe hoog de totale respons is. Het aandeel van de verschillende sensoren in de hoogte van de balk is steeds gescheiden met een lijn. Om het duidelijker te maken welke sensor een sterk signaal ontvangt verschijnt de ellips en een lijn naar het bijbehorende element als het signaal dat dat element opvangt sterker wordt. Op de achtergrond zie je het plaatje dat gevormd wordt wanneer je $P$ over alle mogelijke locaties laat lopen en de pixel helderheid bepaalt door de totale respons op die locatie.

<details open>
    <summary>(klik om in te klappen)</summary>
<img src="{{ 'assets/images/scene-6-focussing-nl-dark.gif' | relative_url }}" style="border:none;"/>
<p></p>
</details>
Aan dit plaatje kunnen we een aantal dingen zien.
- We zien dat de respons veruit het grootste is wanneer we daadwerkelijk op een verstrooier gefocust zijn en dat de verstrooiers te herkennen zijn in het plaatje.
- We zien echter ook dat de uitslag niet altijd nul is wanneer we focussen op een punt waar niks is. Dat komt omdat het focuspunt $P$ dan op de zelfde ellips als een of meer van de verstrooiers ligt.
- Als het focuspunt van een verstrooier af beweegt zien we dat de opgetelde respons niet in één keer naar beneden springt. Dit is omdat de puls die we verzonden hebben niet oneindig scherp is, maar een zekere breedte heeft. We moeten eerste even 'de berg af lopen' voor we bij nul zijn. Dit is een limiet aan hoe scherp het plaatje uiteindelijk kan worden.
- Verstrooiers $V_7$ en $V_8$ hebben een meer uitgesmeerde stip in het plaatje dan bijvoorbeeld $V_5$. Dit komt omdat $V_7$ en $V_8$ meer naar de zijkant liggen waardoor de ellipsen van de verschillende elementen meer op elkaar lijken en dus voor een groter gebied overlappen. De resulotie van een ultrasound plaatje is dus niet overal hetzelfde!

Hoewel de kwaliteit van dit plaatje nog te wensen overlaat is het duidelijk dat het principe werkt en een manier biedt om uit een reeks schijnbaar chaotische signalen een afbeelding maken.

<details>
  <summary>Sidenote</summary>
<p>
Sommigen zullen misschien opmerken dat de wereld niet 2-dimensionaal, maar 3-dimensionaal is. Dit betekend dat alle punten met gelijke afstand tot een sensor niet een circkel, maar een boloppervlak vormen. Werkt dit dan nog wel? Nou nee! Sensor elementen die op een lijn liggen kunnen onderscheid maken tussen alle locaties in een vlak in 2D, maar niet in 3D. Om reflecties van buiten het beeldvlak te onderdrukken hebben ultrasound probes daarom in de praktijk een raster aan elementen in plaats van een lijn. Deze zelfde beamforming technieken worden gebruikt om te focussen op het gewenste vlak en signalen van buiten dat vlak te onderdrukken.
</p>
</details>

## Interessante links

- Het paper [So you think you can DAS](https://www.sciencedirect.com/science/article/abs/pii/S0041624X20302444) geeft een uitgebreide, maar relatief toegankelijke uitleg over het implementeren van DAS-beamforming.
