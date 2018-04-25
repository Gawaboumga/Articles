
> Discussion sur le forum : https://zestedesavoir.com/forums/sujet/10631/les-prochaines-normes-du-c-le-c20-et-le-c23/
> Ou sur le discord : https://discordapp.com/invite/zcWp9sC

# Les prochaines normes du C++ : le C++20 et le C++23

> Le C++20 est obsolète, vive le C++23

Le comité de normalisation du C++ se réunit physiquement tous les six mois, pour préparer les futures évolutions du langage C++ et de sa bibliothèque standard. Une réunion a eu lieu la semaine dernière à Jacksonville, c'est donc une bonne occasion de discuter des travaux en cours pour les futures normes du C++ : le C++20 (qui devrait sortir en 2020) et le C++23 (qui devrait sortir, je suppose que vous avez compris le principe, en 2023).

## Petits rappels sur le fonctionnement du comité C++

Les propositions suivent le chemin :

SG -> EWG/LEWG -> CWG/LWG -> WG21 (full comite). https://isocpp.org/std/the-committee

Travaux consistent a ecrire des propositions, qui sont ensuite commentées, puis modifiées. 

2 reunions physiques par an et plusieurs reunions telephoniques. + tous les echanges entre membres et dans les groupes (emails, github, etc)

Ensemble des propositions publiées sur : http://open-std.org/JTC1/SC22/WG21/docs/papers

**création d'un groupe de travail**

3 documents importants :

- Direction for ISO C++
- Operating principles for evolving C++
- C++ Stability, Velocity, and Deployment Plans
- completement : The Design and Evolution of C++” [Stroustrup,1994]

> We see C++ in danger of losing coherency due to proposals based on differing and sometimes mutually
contradictory design philosophies and differing stylistic tastes. For that reason, we recommend that you
(re)read [Winkel,2017] before proposing a new feature (language or library).


## Les fonctionnalités déjà acceptées pour le C++20

Note : les liens vers les documents des propositions sont données dans les références en fin de billet.

**Les attributes**

- [[no_unique_address]] pour éviter le coût des variables membres sans passer par une classe de base (EBO, empty base optimization)
- [[unlikely]] et [[likely]] pour indiquer quelles branches sont le plus souvent attendues. Attention : le compilateur sait tres bien mesurer les performances attendues, cette fonctionnalité n'est pas forcement pour l'optimisation. Le but est surtout de pouvoir indiquer qu'une branche sous optimiseée est attendue.

**Ajouts dans <chrono>**

Ajout de la gestion des calendriers et des fuseaux horaires. Des petites modifications ont été apportées à <chrono> afin d'étendre son fonctionnement aux dates. Seul le calendrier grégorien [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) sera incorporé au langage, des implémentations du côté client pourront interagir avec la bibliothèque standard afin d'offrir la gestion d'autres calendriers (plus spécifiques à un pays ou une culture).
Les fuseaux horaires se baseront naturellement sur la base de données IANA et une gestion des secondes intercallaires est également proposée. Enfin, la famille des *strftime* du C se retrouvera également, afin de lire et écrire des dates de manières simplifiées et auront été renommées respectivement *parse* et *format* et prenant en compte ou non la locale.

De manière plus précise, cinq notions seront ajoutées:
- Calendiers: Précision à une journée près.
- Temps système: Gestion du temps analogue au temps UNIX.
- Temps local: Pour les temps sans aucune considération du fuseau horaire.
- Fuseau horaire: Ensemble des données spécifiques à une zone géographiques et ses variations au cours du temps. Cela inclut également le nom de la zone, son abbréviation, son décalage UTC, ...
- Temps précis: Temps système combiné à un fuseau horaire.

Une syntaxe spécialisée a également été adoptée afin de simplifier l'écriture.
```cpp
constexpr year_month_day x1 = 2015_y/mar/22;
constexpr year_month_day x2 = x1 + months{1};
constexpr year_month_weekday_last x3 = sun[last]/mar/2015;

auto zone = locate_zone("America/Los_Angeles");
auto now = floor<milliseconds>(system_clock::now());
auto local = zone->to_local(now);
cout << now << ' ' << "UTC" << ‘\n';                // 2015-09-25 16:50:06.123 UTC
cout << local.first << ' ' << local.second << '\n'; // 2015-09-25 09:50:06.123 PDT

auto tp = sys_days{2016y/May/29d} + 7h + 30min + 6s + 153ms; // 2016-05-29 07:30:06.153 UTC
zoned_time zt = {"Asia/Tokyo", tp};
cout << zt << '\n';                                          // 2016-05-29 16:30:06.153 JST

auto tp = day_point{jan/3/1970} + 7h + 33min + 20s;
assert(tp.time_since_epoch() == 200000s);

for (auto& l : get_tzdb().leaps)
  cout << l.date() << '\n'; // Affiche tous les jours où il y a eu une seconde intercallaire.
```

Cependant, la majorité des fonctions seront marquées comme *noexcept*, cela implique que nous serons capable de produire des dates invalides. Pour cela, une méthode "ok" est disponible afin de s'assurer que la date existe réellement.

Une implémentation canonique est disponible [ici](https://github.com/HowardHinnant/date).

**span**

Une partie de `std::array_view` et TS tableaux. Implémenté dans GSL. Probleme : lecture seule?

**typename**

moins besoin de `typename` avec les template. Detection par compilateur.

**ajout des variadiques dans les lambdas**

`[...Args](){}`

**begin et end moins contraigant**

Utilisation de begin/end membre par défaut, ou libre si n'existe pas. Pas besoin d'avoir les 2 (par exemple std::ios_base::end).

**acces pour les structured bindings**

Seulement pour le public en C++17. Tout ce qui est accessible en C++20.

**Structured bindings get<> lookup relaxations (Ville Voutilainen)**

**<=> symetrique**

**Thou shalt not specialize std function templates! (Walter E. Brown)**

**<version>**

**Range-based for statements with initializer (Thomas Köppe)**

Ajout de l'initialisation dans for.

```cpp
// avant C++17
for (int i = 0; i < 20; ++i) ...

// C++17
if (T thing = f(); thing) 
switch (T thing = f(); thing) 

// C++20
for (int i = 0; auto x: ranges) ...
````

**Bit-casting object representations (JF Bastien)**

Ajout de `bit_cast`. Similaire a `memcpy` mais safe et peut tourner en compile time.

**Operator <=>, aka “spaceship” (Herb Sutter)**

Pour declarer les 6 opérateurs en une seule fois.

**atomic<shared_ptr<T>>**

pour les threads

**Lots of other cleanup**

- core issue 1581
- P0857
- P0624


## Propositions déjà acceptées

Les 2 premieres : full comité. Les autres : par les comité de travail (Language/library evolution subgroup).

**Parallelism TS 2**

Ajout de l'extension "Data-parallel vector types and operations (Matthias Kretz)"

**ouverture du Reflection TS**

Avec le contenu initial “Static reflection” (Matúš Chochlík, Axel Naumann, David Sankel)

**contracts**

clarifications.

**reflection**

- “Static reflection of functions”
- “Standard containers and constexpr”
- “Class types in non-type template parameters”

**autres**

- “char8_t: A type for UTF-8 characters and strings”
- “explicit(bool)”
- “Checking for abstract class types”

** Ajouts de documents officiels sur l'évolution du C++**

- “C++ Stability, Velocity, and Deployment Plans” 
- “Standard library compatibility promises”

C'est a dire que les "promesses" faites par le comité C++ ne seront officiellement dans la norme.

**adoption des concepts par le LEWG**

Standard library concepts (Casey Carter)

**Le TS modules**

Les modules sont une nouvelle approche pour créer des bibliothèques logicielles réutilisables.

objectif : etre dans le C++20. Deja implementé dans plusieurs compilos

**Travaux a l'etude**

- utilisation des macros et solutions non-macros alternatives. Pour etre sur que les macros ne sont plus obligatoire.
- casser un peu la compatibilité des comparaisons pour les types natifs, par rapport a <=>. En particulier comparaison signed/unsigned.

**Autres progres**

- sur les concepts : “Concepts in-place syntax” (Herb Sutter) 
- executors : sera probablement un TS dnas C++20 (thread)
- coroutine : essaie de le mettre dans C++20
- ouverture de Library Fundamentals TS3 pour le C++23. (TS2 est dans C++20).
- premier meeting de SG15 (Tooling). En particulier pour modules, et C++ library package managers
- creation de SG16 (Unicode)


## La suite...

Les previsions de sorties : https://herbsutter.files.wordpress.com/2018/04/cpp11147020-201803.png

Timeline : https://herbsutter.files.wordpress.com/2018/04/wg21-timeline-2018-03.png?w=1000

## Références

Sur le fonctionnement du comité C++ et l'évolution du C++ :

- [IsoCPP](https://isocpp.org/std). Le site officiel du comité de normalisation du C++.
- [Direction for ISO C++](http://open-std.org/JTC1/SC22/WG21/docs/papers/2018/p0939r0.pdf). Un document expliquant la direction que devrait prendre le C++ dans l'avenir. Écrit par plusieurs membres du comité C++, dont Bjarne Stroustrup.
- [Operating principles for evolving C++](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2017/p0559r0.pdf)
- [C++ Stability, Velocity, and Deployment Plans](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2017/p0684r0.pdf)
- [The Design and Evolution of C++](https://www.amazon.com/Design-Evolution-C-Bjarne-Stroustrup/dp/0201543303) [Stroustrup 1994]
- [C++ Developer Survey](https://isocpp.org/files/papers/CppDevSurvey-2018-02-summary.pdf). Les résultats d'une sondage réalisé en février 2018, pour connaître les besoins des développeurs C++.
- [The Evils of Paradigms](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p0976r0.pdf)

Les comptes rendus du meeting C++ et les commentaires sur les propositions :

- [Trip report: Winter ISO C++ standards meeting (Jacksonville)](https://herbsutter.com/2018/04/02/trip-report-winter-iso-c-standards-meeting-jacksonville/) par [Herb Sutter](https://fr.wikipedia.org/wiki/Herb_Sutter). Le compte-rendu de la réunion de Jacksonville, début 2018.
- [Trip report: Fall ISO C++ standards meeting (Albuquerque)](https://herbsutter.com/2017/11/11/trip-report-fall-iso-c-standards-meeting-albuquerque/) par [Herb Sutter](https://fr.wikipedia.org/wiki/Herb_Sutter). Le compte-rendu de la réunion à Albuquerque, fin 2017.

Quelques extraits du mailing [post-jacksonville 2018](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/#mailing2018-04) :

- [Evolution status after Jacksonville 2018](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p1018r0.html)
- [WG21 2018-03 Jacksonville Record of Discussion](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p1015r0.html)
- [C++ IS schedule](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p1000r0.pdf)


https://www.youtube.com/watch?v=e2ZQyYr0Oi0

## Auteurs

- gbdivers
- lmghs

