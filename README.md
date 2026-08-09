# nativity-simulation

Ja. Jag skulle ge analytikerna något i den här formen. Jag ändrar medvetet ordet **”korrelera” till ”kalibrera och validera mot”** på några ställen: rena korrelationer kommer annars lätt att blanda ihop exempelvis senareläggning, minskad barnönskan och partnerbrist.

## Analysförslag: generativ modell för fertilitet, reproduktiva val och populationsutfall

### 1. Syfte

Bygg en probabilistisk, agentbaserad livsloppsmodell som kan återskapa svensk fertilitetsutveckling och samtidigt separera följande mekanismer:

1. grundläggande motivation att någon gång bli förälder,
2. omvandlingen från motivation till konkret barnönskan,
3. partnerbildning och matchning mellan två individers önskningar,
4. beslut om tidpunkt,
5. faktiskt försök till graviditet,
6. biologisk sannolikhet för graviditet och födsel,
7. beslut om ytterligare barn,
8. migration och mortalitet,
9. resulterande populationsstorlek, åldersstruktur och försörjningskvot.

Modellen ska **inte förutsätta att födsel är det samhälleliga slutmålet**. Fertilitet ska behandlas som en av flera processer som formar populationen. Migration, mortalitet och fertilitet hålls därför åtskilda.

En central forskningsfråga är:

> Kan Sveriges historiska och nuvarande fertilitet förklaras med i stort sett konstant grundläggande mänsklig benägenhet till föräldraskap, men förändrade sannolikheter i de efterföljande leden – eller krävs också en faktisk förändring i själva motivationen att bli förälder?

---

## 2. Grundarkitektur

Simulera individer (i) i exempelvis månads- eller kvartalssteg från 15 till 50 års ålder.

Varje individ befinner sig i ett tillstånd:

[
X_{i,t} =
{
ålder,
kön,
utbildning,
inkomst,
arbete,
bostad,
partnerstatus,
partner,
antal\ barn,
barnönskan,
intention,
preventivmedelsstatus,
försök,
fertilitet,
identitet,
socialt\ nätverk
}.
]

Undvik en modell där ett antal faktorer bara summeras till ett ”fertility score”. Låt i stället varje steg vara en **villkorad övergångssannolikhet**:

[
P(X_{t+1}\mid X_t,Z_t)
]

där (Z_t) beskriver samhällsmiljön vid tiden (t).

En födsel blir då slutet på en kedja av stokastiska processer, inte resultatet av en enda regressionskoefficient.

---

# 3. Reproduktionskedjan

För en barnlös individ kan en förenklad kedja vara:

[
M \rightarrow D \rightarrow P \rightarrow J \rightarrow T
\rightarrow A \rightarrow C \rightarrow B
]

där:

* (M) = latent motivation till föräldraskap,
* (D) = explicit önskan om barn,
* (P) = tillgång till partner,
* (J) = joint willingness, båda partner vill,
* (T) = tidpunkten bedöms acceptabel,
* (A) = aktivt försök,
* (C) = konception och fortsatt graviditet,
* (B) = levande födsel.

För ett givet tidssteg:

[
P(B_{i,t}) =
P(D\mid M,X,Z)
P(P\mid X,Z)
P(J\mid D_i,D_j,X,Z)
P(T\mid J,X,Z)
P(A\mid T,X,Z)
P(C\mid A,age,biology)
P(B\mid C).
]

Produkten är pedagogisk. I implementationen bör detta vara **separata competing-risk/event-history-processer** snarare än en bokstavlig multiplikation av fasta sannolikheter.

Det gör modellen ”Drake-ekvationslik”: flera ganska små förändringar kan multipliceras till en stor förändring i slutligt antal födslar.

---

# 4. Latent motivation måste modelleras separat

Sätt inte

[
M = ekonomi + norm + barnlängtan - kostnad.
]

Modellera hellre (M) som en latent stokastisk egenskap:

[
M_i \sim F(\theta_{\text{cohort}},\theta_{\text{sex}},
\theta_{\text{culture}},\theta_{\text{person}})
]

som endast delvis observeras genom enkätfrågor.

Här är Millertraditionens **Traits → Desires → Intentions → Behaviour** användbar som teoretisk referens: motivation och önskan är inte samma sak som intention och beteende. Nyare fertilitetsforskning använder fortfarande denna uppdelning. ([Omega-PSIR][1])

Testa minst två varianter:

[
M_{i,t}=M_i
]

— relativt stabil grundmotivation —

mot

[
M_{i,t}=M_i+\gamma_{\text{cohort}}+\gamma_{\text{period},t}
]

— motivationen själv kan förändras mellan generationer eller perioder.

Det är en av modellens viktigaste empiriskt testbara skiljelinjer.

---

# 5. Barnets värde och alternativets värde ska separeras

Modellera inte bara:

[
V(\text{barn})
]

utan:

[
V^P_{i,t}=E[U_i\mid föräldraskap]
]

och

[
V^C_{i,t}=E[U_i\mid barnfrihet].
]

Beslutet påverkas av

[
\Delta V_{i,t}=V^P_{i,t}-V^C_{i,t}.
]

Detta gör det möjligt att skilja två helt olika historiska hypoteser:

**H1:** Barn har blivit mindre attraktiva.

**H2:** Barn är ungefär lika attraktiva emotionellt, men ett liv utan barn har blivit mycket mer attraktivt.

Value-of-Children-litteraturen ger stöd för att skilja ekonomiskt, socialt och psykologiskt värde. Thomson sammanfattar att barns värde kan komma från barnet självt, erfarenheten av att uppfostra det samt familjens och samhällets respons, medan både värden och kostnader förändras med ekonomiska och sociala institutioner. ([Diva Portal][2])

Utgångspunkt:

`https://www.diva-portal.org/smash/get/diva2:1284512/FULLTEXT01.pdf`

Friedman, Hechter & Kanazawa är en alternativ teori där föräldraskap bland annat behandlas som ett sätt att reducera osäkerhet och förstärka familjerelationer. ([PubMed][3])

`https://pubmed.ncbi.nlm.nih.gov/7828763/`

---

# 6. Defaultmekanismen

Inför en separat periodparameter:

[
Q_t=P(\text{födsel utan stark explicit intention}).
]

Den representerar institutionell ”reproduktiv automatik”:

* effektiv preventivmedelsanvändning,
* sexualitetens koppling till äktenskap,
* samboende,
* normtryck,
* abortmöjlighet,
* kvinnors ekonomiska självständighet,
* acceptans av permanent barnlöshet.

Hypotesen är:

[
Q_{1920} \gg Q_{2025}.
]

Det ska **inte sättas för hand för att få rätt resultat** utan identifieras indirekt från historiska data.

Ett intressant kontrafaktiskt experiment blir:

> Behåll distributionen av (M) konstant från 1920 till 2025 och ändra endast (Q), partnerbildning, timing och alternativkostnader. Hur mycket av fertilitetsförändringen kan då återskapas?

---

# 7. Parbildning måste vara en egen marknad

Skapa en separat matchingmodell.

För varje individ:

[
P(\text{ny partner}_{i,t})
==========================

f(age,education,location,network,\ldots)
]

och för potentiellt par (i,j):

[
P(i\leftrightarrow j)=g(X_i,X_j).
]

Sedan:

[
P(J_{ij,t}) =
P(D_i=1,D_j=1\mid relation).
]

Testa särskilt en **veto-modell**:

[
P(A_{ij,t}) \approx
\min(P(A_i),P(A_j))
]

mot en mer förhandlingsbaserad modell.

Detta är viktigt därför att reproduktion inom ett par ofta kräver två tillräckligt positiva beslut medan fortsatt barnlöshet bara kräver att en part säger nej eller ”inte nu”.

En liten ökning av individuell tveksamhet kan därför få en större effekt på parfertiliteten än vad enkätdata på individnivå antyder.

---

# 8. Timing ska vara en hazard, inte en åsikt

Använd exempelvis diskret tidslogit eller complementary log-log:

[
\log[-\log(1-h_{i,t})]
======================

\alpha(age_t)+\beta X_{i,t}+u_i.
]

Separata hazards för:

* första stabila relation,
* samboskap,
* barnönskan,
* treårsintention,
* försök,
* första barn,
* separation,
* andra barn,
* tredje barn.

Det gör att modellen kan skilja på:

**tempo:** samma antal barn, senare.

och

**quantum:** färre barn över hela livet.

Det är centralt eftersom nordiska data visar att den nyare nedgången inte enbart verkar vara senareläggning. Hellstrand med flera finner att senareläggning endast förklarar en del av nedgången och att utvecklingen framför allt drivits av färre första födslar. ([Duke University Press Journals][4])

---

# 9. Första svenska kalibreringspunkten: intentionsstudien 2012–2021

Lai, Neyer & Andersson 2026 bör vara en av huvudbenchmarkarna:

`https://doi.org/10.12765/CPoS-2026-04`

Studien jämför svenska GGS 2012 och 2021 och visar att nedgången i födslar åtföljdes av minskade fertilitetsintentioner och större osäkerhet. ([Jämförande befolkningsstudier][5])

### Kalibrera modellen mot minst:

**A. Genomsnittligt avsett antal barn**

[
2012: 2,26
]

[
2021: 1,79.
]

Det rapporteras direkt i artikeln. ([Ssoar][6])

Detta ska jämföras med modellens

[
E[N^{intended}*{2012}]
\quad \text{och}\quad
E[N^{intended}*{2021}].
]

Var dock försiktig med att använda just denna differens som hårt kalibreringsmål eftersom författarna diskuterar jämförbarheten mellan mätningarna.

**B. Fördelningen mellan**

* definitivt ja,
* troligen ja,
* troligen nej,
* definitivt nej

för barn inom tre år.

**C. Samma fyra nivåer för ”någon gång senare”.**

**D. Interaktioner med**

* ålder,
* kön,
* barnlöshet/paritet,
* partnerskap,
* utbildning.

Det viktiga är inte bara att modellen reproducerar medelvärdet utan **hela svarsfördelningen**.

---

# 10. Ekonomisk osäkerhet

Lindström 2025 undersöker svenska barnlösa par i GGS 2021 och finner samband mellan osäkerhet om den egna förmågan att hantera arbetslöshet och lägre fertilitetsintention bland män, medan motsvarande generella samband inte är tydligt för kvinnor; effekterna är starkare i vissa ekonomiskt sårbara grupper. ([Demographic Research][7])

DOI:

`https://doi.org/10.4054/DemRes.2025.53.31`

Låt därför inte ekonomisk osäkerhet vara en universell koefficient:

[
\beta_{\text{uncertainty}}
]

utan exempelvis:

[
\beta_{\text{uncertainty}}
==========================

\beta_0+
\beta_1 sex+
\beta_2 migrant+
\beta_3 vulnerability.
]

Validera modellens conditional marginal effects mot artikelns regressionsresultat.

---

# 11. Historiskt fertilitetsbenchmark

Först behövs en ren demografisk baseline.

Oláh & Bernhardt redovisar bland annat att svensk period-TFR föll från **2,48 år 1964 till 1,61 år 1983**, steg tillbaka mot replacementnivå 1990 och nådde omkring **1,51 1998–1999**. De visar samtidigt att fullbordad kohortfertilitet varierade betydligt mindre: omkring 1,8 barn för kvinnor födda 1904–05 och nära 2,2 för kohorter födda under första halvan av 1930-talet. 

Källa:

`https://www.demographic-research.org/volumes/vol19/28/19-28.pdf`

Det är ett utmärkt test av modellen:

> Kan modellen producera stora periodsvängningar samtidigt som completed cohort fertility förblir mycket stabilare?

Om den inte kan det blandar den sannolikt ihop timing med förändrad livstidsmotivation.

---

# 12. SCB ska vara observationslagrets huvudkälla

SCB har nu tidsserier för bland annat:

* population sedan 1749,
* födslar och dödsfall sedan 1851,
* födslar efter moderns ålder sedan 1968,
* TFR efter region,
* ålder vid födsel efter födelseordning,
* migration efter ålder och kön,
* civilstånd och partnerskap,
* mortalitet,
* försörjningskvot. ([Statistikmyndigheten SCB][8])

Startpunkt:

`https://www.scb.se/en/finding-statistics/statistics-by-subject-area/population-and-living-conditions/population-composition-and-development/population-statistics/`

Detta bör inte vara en enda målfunktion. Matcha samtidigt:

[
TFR_t
]

[
ASFR_{a,t}
]

[
P(first\ birth\mid age)
]

[
P(second\ birth\mid age,time\ since\ first)
]

[
completed\ cohort\ fertility_c
]

[
childlessness_c
]

[
mean\ age(first\ birth)_t.
]

En modell som bara reproducerar TFR är underidentifierad.

---

# 13. Modellera populationen separat från fertiliteten

Efter att reproduktionsmodellen producerat födslar läggs den in i en vanlig cohort-component-modell:

[
N_{a+1,s,t+1}
=============

## N_{a,s,t}

D_{a,s,t}
+
I_{a,s,t}
---------

E_{a,s,t}.
]

För ålder 0:

[
N_{0,t+1}=B_t-D_{0,t}+I_{0,t}-E_{0,t}.
]

SCB:s officiella statistik innehåller samtidigt åldersstrukturerade serier för både mortalitet och migration. ([Statistikmyndigheten SCB][8])

Detta gör att migration aldrig behöver ”översättas” till motsvarande TFR.

---

# 14. Håll den politiska målfunktionen utanför den positiva modellen

Skapa inte automatiskt:

[
\max TFR.
]

Låt modellen först producera demografiska konsekvenser.

Därefter kan en separat evaluator beräkna exempelvis:

[
U =
w_NN+
w_WW+
w_RR+
w_FF+
w_GG-
C
]

med:

* (N) = populationsstorlek,
* (W) = arbetsför befolkning,
* (R) = försörjningskvot,
* (F) = offentliga finanser,
* (G) = individvälfärd,
* (C) = kostnader.

Om någon vill lägga till ”kulturell kontinuitet”, ”andel inrikes födda” eller något annat normativt värde ska det vara en **explicit separat komponent**.

Annars finns risk att modellen i smyg antar:

[
1\ födsel > 1\ invandrare
]

trots att den uttalade målfunktionen bara är populationsstorlek.

Det är precis den premiss som bör exponeras snarare än byggas in.

---

# 15. Algoritm

En första Monte Carlo-version kan se ut så här:

```text
INITIALISERA historisk population vid år t0

FÖR varje simulerat år t:
    uppdatera samhällstillstånd Z(t)

    FÖR varje individ i:
        uppdatera ålder
        uppdatera utbildning
        uppdatera arbete/inkomst
        uppdatera bostad

        om ensam:
            dra partnerbildning ~ Bernoulli(h_partner(i,t))
            om träff:
                matcha med j enligt matchingkernel(i,j)

        uppdatera latent/observerad barnönskan:
            D(i,t) ~ P(D | M_i, identitet, nätverk, Z_t)

        om partner:
            beräkna joint willingness J(i,j,t)

        om J är tillräckligt:
            dra timing/intention
            dra försök

        om försök:
            dra konception baserat på ålder m.m.
            dra graviditetsutfall

        vid födsel:
            lägg till ny agent
            uppdatera paritet
            ändra framtida hazards

        dra separation/död/utvandring

    generera invandring som separat process
    lägg in immigranter med egen ålder, kön och tillstånd

    registrera:
        births
        ASFR
        TFR
        cohort fertility
        childlessness
        intended fertility
        partnership
        migration
        population age structure
```

Kör minst (10^3)–(10^4) Monte Carlo-replikat per parameteruppsättning.

---

# 16. Kalibreringsalgoritm

Jag skulle börja Bayesianiskt.

Parametrar:

[
\Theta=
{
\theta_M,
\theta_D,
\theta_P,
\theta_J,
\theta_T,
\theta_A,
\theta_C,
\theta_Q,\ldots
}.
]

Observationer:

[
Y=
{
GGS,
TFR,
ASFR,
cohort\ fertility,
childlessness,
partnering,
migration,\ldots
}.
]

Målet är:

[
p(\Theta\mid Y)
\propto
p(Y\mid\Theta)p(\Theta).
]

Om simulatorns likelihood är svår att skriva explicit:

**börja med ABC-SMC** — Approximate Bayesian Computation med Sequential Monte Carlo.

Definiera summary statistics:

[
S(Y)=
[
TFR_t,
ASFR_{a,t},
CFR_c,
childlessness_c,
intentions_{age,sex,parity,t},
partnering_{age,t}
].
]

Acceptera parameteruppsättningar där:

[
d(S(Y_{\text{sim}}),S(Y_{\text{obs}}))<\epsilon.
]

Det ger dessutom något som en vanlig regression inte ger: **vilka kombinationer av mekanismer som faktiskt är identifierbara från befintlig forskning.**

---

# 17. Kör fyra huvudmodeller mot varandra

Detta tycker jag ska vara projektets första riktiga test.

### Modell A — konstant motivation

[
M_{cohort}=constant.
]

Endast omständigheter, partnerbildning, preventivmedel och timing förändras.

### Modell B — attraktivare barnfritt liv

(M) och värdet av barn är stabila, men:

[
V^C_t\uparrow.
]

### Modell C — norm/default-modell

Grundmotivation stabil men:

[
Q_t\downarrow
]

samt normtryck och automatisk reproduktion minskar.

### Modell D — motivationsförändring

Utöver A–C tillåts:

[
E[M\mid cohort]\downarrow.
]

Jämför modeller med out-of-sample predictive performance, posterior predictive checks och Bayes factors/LOO där det är möjligt.

Den intressanta frågan blir inte vilken modell som passar **2025** bäst, utan vilken som samtidigt kan förklara:

[
1920 \rightarrow 1950 \rightarrow 1980 \rightarrow 2012 \rightarrow 2021 \rightarrow 2025.
]

---

# 18. Viktigaste korrelationerna/momenten att undersöka

Jag skulle ge analytikerna denna konkreta matris som första arbetsorder:

| Modellvariabel        | Observerad storhet att matcha             | Huvudkälla            |
| --------------------- | ----------------------------------------- | --------------------- |
| latent barnmotivation | indirekt via desires/intended family size | GGS/Miller            |
| önskan om barn        | definitiv/trolig framtida intention       | GGS 2012/2021         |
| kort timing           | intention inom 3 år                       | GGS                   |
| osäkerhet             | ”probably” vs ”definitely”                | Lai et al.            |
| partnerflaskhals      | partnerstatus × intention                 | GGS/register          |
| ekonomisk risk        | resilience × intention                    | Lindström 2025        |
| tempo                 | åldersspecifik fertilitet                 | SCB                   |
| quantum               | completed cohort fertility                | SCB/Hellstrand        |
| första barn           | first-birth hazards                       | SCB/registerstudier   |
| fortsatt fertilitet   | parity progression ratios                 | SCB                   |
| barnlöshet            | slutlig barnlöshet per kohort             | SCB                   |
| default/norm          | restparameter + historiska institutioner  | historisk kalibrering |
| migration             | nettoflöden efter ålder/kön               | SCB                   |
| population            | ålder × kön                               | SCB                   |
| försörjningskvot      | faktisk historisk serie                   | SCB                   |

---

# 19. Ett viktigt metodkrav

Analytikerna ska **inte tillåtas kalibrera alla latenta parametrar fritt mot samma TFR-serie**.

Annars kan nästan vad som helst förklara vad som helst.

Gör i stället en *measurement map*:

[
\text{studie} \rightarrow
\text{observerad variabel} \rightarrow
\text{latent modellkomponent}.
]

Exempel:

[
GGS\ intention
\rightarrow
P(T\mid D,\ldots)
]

inte direkt

[
GGS\ intention\rightarrow M.
]

Och:

[
TFR \rightarrow slututfall
]

inte

[
TFR\rightarrow barnlängtan.
]

Det är sannolikt det viktigaste skyddet mot att bygga en elegant modell som bara reproducerar forskarnas egna antaganden.

---

# 20. Första leveransen jag skulle beställa

**Fas 1, utan avancerad agentmodell:** bygg en probabilistisk DAG/state-transition-modell och skapa en datakatalog över varje observerbar nod.

**Fas 2:** estimera empiriska transition hazards där mikrodata finns.

**Fas 3:** fyll de oobserverade delarna med priors från internationell litteratur och genomför identifikationsanalys.

**Fas 4:** bygg agentmodellen.

**Fas 5:** kör historiska kontrafaktiska experiment:

* 2025 års människor i 1950 års institutioner,
* 1950 års reproduktiva miljö med 2025 års partnerbildning,
* konstant latent motivation 1900–2025,
* konstant partnerbildning men förändrad motivation,
* ingen nettoinvandring,
* faktisk migration men olika fertilitetsbanor.

Det sista är viktigt: **först när reproduktionsmodellen kopplas till populationsmodellen går det att avgöra vilken samhällsfråga fertiliteten faktiskt löser och vilka problem migration kan respektive inte kan ersätta den för.**

En bra intern projekttitel vore exempelvis **”Generative Reproductive Choice Model — Sweden 1900–2050”**. Jag skulle låta den första milstolpen vara mycket konkret: *”Vilken minsta uppsättning förändrade transition probabilities krävs för att samtidigt reproducera svensk cohort fertility, period fertility och intentionsförändringen 2012–2021 utan att anta en förändring i latent barnmotivation?”*

Det är ett falsifierbart första experiment, vilket gör hela projektet betydligt skarpare än en bred jakt på ”orsaker till låga födelsetal”.

[1]: https://bazawiedzy.uksw.edu.pl/docstore/download/UKSW3be623e22c0647f79b1a86e52a07b1c4/Brini-2026-Childbearing-Motivations-and-Fertility-Desires.pdf?entityId=UKSWb508a57764be41b481d336a1eb1bacce&entityType=article&utm_source=chatgpt.com "Childbearing Motivations and Fertility Desires"
[2]: https://www.diva-portal.org/smash/get/diva2%3A1284512/FULLTEXT01.pdf "Children, Value of"
[3]: https://pubmed.ncbi.nlm.nih.gov/7828763/ "A theory of the value of children - PubMed"
[4]: https://read.dukeupress.edu/demography/article/58/4/1373/174063/Not-Just-Later-but-Fewer-Novel-Trends-in-Cohort?utm_source=chatgpt.com "Novel Trends in Cohort Fertility in the Nordic Countries"
[5]: https://comparativepopulationstudies.de/index.php/CPoS/article/view/760 "Not Only Births, But Also Intentions: The Decline of Fertility Intentions and Increasing Uncertainties in Sweden, 2012-2021 | Comparative Population Studies"
[6]: https://www.ssoar.info/ssoar/bitstream/handle/document/109230/ssoar-cpos-2026-lai_et_al-Not_Only_Births_But_Also.pdf?isAllowed=y&sequence=1&utm_source=chatgpt.com "Not Only Births, But Also Intentions: The Decline of Fertility ..."
[7]: https://www.demographic-research.org/articles/volume/53/31 "Uncertainty, resilience, and fertility: Perceived capacity to overcome loss of employment and fertility intentions in Sweden, 2021 (Volume 53 - Article 31 | Pages 1003–1044) - Demographic Research"
[8]: https://www.scb.se/en/finding-statistics/statistics-by-subject-area/population-and-living-conditions/population-composition-and-development/population-statistics/ "Population statistics"
