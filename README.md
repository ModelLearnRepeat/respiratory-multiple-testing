# Respiratory Multiple-Testing Analysis

## Kurzbeschreibung

Reproduzierbare longitudinale Analyse eines öffentlichen klinischen Datensatzes
mit R und Quarto.

Untersucht wird, ob sich der Anteil eines guten Atemwegsstatus zwischen einer
aktiven Behandlung und einem Placebo über vier Untersuchungszeitpunkte
unterscheidet.

Die Analyse konzentriert sich auf:

- deskriptive Auswertung klinischer Daten,
- zeitpunktbezogene Gruppenvergleiche,
- Korrektur für multiples Testen,
- Kontrolle der Familywise Error Rate und False Discovery Rate,
- Analyse wiederholter binärer Messungen mit Generalized Estimating Equations
  (GEE),
- Prüfung der Robustheit gegenüber unterschiedlichen Altersmodellierungen.

## Forschungsfrage

Unterscheidet sich der beobachtete Atemwegsstatus zwischen der aktiven
Behandlungsgruppe und der Placebogruppe über die vier Untersuchungszeitpunkte?

## Datensatz

Verwendet wird der öffentliche Datensatz `respiratory` aus dem R-Paket
`geepack`.

Der Datensatz enthält:

- 111 Personen,
- 444 Beobachtungen,
- vier Untersuchungszeitpunkte pro Person,
- zwei Behandlungsgruppen,
- zwei klinische Zentren,
- einen binären Atemwegsstatus,
- wiederholte Messungen innerhalb derselben Person.

Die Daten werden direkt aus dem R-Paket geladen. Individuelle Rohdaten werden
nicht separat in diesem Repository gespeichert.

## Methodik

### Deskriptive Analyse

Zunächst werden die Behandlungsgruppen und der Atemwegsstatus nach
Untersuchungszeitpunkt beschrieben. Dabei werden absolute Häufigkeiten und
Anteile eines guten Atemwegsstatus berichtet.

Diese deskriptiven Ergebnisse beschreiben die beobachteten Daten. Sie
berücksichtigen noch nicht die Abhängigkeit wiederholter Messungen und sind
daher nicht als adjustierte Behandlungseffekte zu interpretieren.

### Multiple Tests

Für jeden der vier Untersuchungszeitpunkte wird ein separater Gruppenvergleich
durchgeführt. Dadurch entstehen vier p-Werte.

Um das Problem multipler Tests zu berücksichtigen, werden dieselben Roh-p-Werte
mit vier Verfahren korrigiert:

- Bonferroni,
- Holm,
- Benjamini-Hochberg,
- Benjamini-Yekutieli.

Bonferroni und Holm kontrollieren die Familywise Error Rate. Sie begrenzen die
Wahrscheinlichkeit, innerhalb der gesamten Testfamilie mindestens einen falsch
positiven Befund zu erhalten.

Benjamini-Hochberg kontrolliert die False Discovery Rate. Benjamini-Yekutieli
kontrolliert die False Discovery Rate auch unter allgemeiner Abhängigkeit der
Teststatistiken, ist dafür meist konservativer.

### Longitudinales Modell

Da jede Person viermal untersucht wird, sind die Beobachtungen innerhalb einer
Person nicht unabhängig.

Für die gemeinsame Analyse wird deshalb ein GEE-Modell verwendet. Das Modell
berücksichtigt die Abhängigkeit der wiederholten binären Messungen innerhalb
derselben Person.

Berücksichtigt werden:

- Behandlung,
- Untersuchungszeitpunkt,
- Alter,
- Geschlecht,
- wiederholte Messungen pro Person.

Zusätzlich wird geprüft, ob der Zusammenhang zwischen Alter und Atemwegsstatus
besser linear oder nichtlinear modelliert wird.

## Zentrale Ergebnisse

Vor der Korrektur waren die Besuche 1, 2 und 3 statistisch signifikant.

Nach der Korrektur ergaben sich folgende Entscheidungen:

| Verfahren | Signifikante Besuche |
|---|---|
| Bonferroni | 2 und 3 |
| Holm | 2 und 3 |
| Benjamini-Hochberg | 1 und 2 |
| Benjamini-Yekutieli | 2 und 3 |

Der Unterschied bei Besuch 1 hängt somit vom gewählten Korrekturverfahren ab.
Der Befund bei Besuch 2 ist gegenüber den verschiedenen
Korrekturverfahren am robustesten.

Im GEE-Modell waren die Odds für einen guten Atemwegsstatus in der Placebogruppe
niedriger als in der aktiven Behandlungsgruppe.

Der Behandlungseffekt blieb auch nach Berücksichtigung von Alter, Geschlecht
und wiederholten Messungen bestehen. Die Schlussfolgerung war außerdem robust
gegenüber einer linearen oder nichtlinearen Modellierung des Alters.

## Interpretation

Die Ergebnisse sprechen für einen Unterschied zwischen aktiver Behandlung und
Placebo. Sie sollten jedoch als explorativer Gruppenvergleich interpretiert
werden.

Die zeitpunktbezogenen Tests beantworten die Frage, zu welchen einzelnen
Besuchen sich die Gruppen unterscheiden. Das GEE-Modell beantwortet dagegen die
gemeinsame longitudinale Frage und berücksichtigt die Abhängigkeit der
wiederholten Messungen.

Die Analyse stellt keine individuelle Therapieempfehlung dar.

## Einschränkungen

- Die Stichprobe umfasst nur 111 Personen.
- Die Messungen stammen aus vier Untersuchungszeitpunkten.
- Das Geschlecht ist in der Stichprobe ungleich verteilt.
- Die Ergebnisse sind explorativ und nicht automatisch auf andere Populationen
  übertragbar.
- Die Unabhängigkeit der Zensierung ist hier nicht relevant, da es sich nicht um
  eine Survival-Analyse handelt.
- Ein GEE-Modell beschreibt durchschnittliche Effekte auf Populationsebene und
  liefert keine individuelle Therapieprognose.
- Korrelation ist nicht gleichbedeutend mit Kausalität außerhalb eines
  entsprechend geplanten Studiendesigns.

## Reproduzierbarkeit

Das Projekt kann mit R und Quarto reproduziert werden.

Benötigte R-Pakete:

```r
install.packages(c(
  "geepack",
  "tidyverse",
  "broom",
  "splines"
))


Dieses Repository ist ein Portfolio- und Lernprojekt. Es demonstriert die
strukturierte Anwendung statistischer Methoden auf öffentliche medizinische
Daten.

Die Ergebnisse beschreiben durchschnittliche Gruppenunterschiede in diesem
Datensatz. Sie berücksichtigen nicht alle individuellen klinischen Faktoren und
liefern keine individuelle Prognose. Eine statistische Signifikanz bedeutet
außerdem nicht automatisch, dass eine Behandlung für jede Person geeignet oder
sicher ist.

Daher sind die Ergebnisse als methodische und statistische Auswertung zu
verstehen. Eine Therapieentscheidung erfordert zusätzlich eine klinische
Untersuchung, die individuelle Krankengeschichte, mögliche Kontraindikationen,
Nebenwirkungen und ärztliche Beurteilung.