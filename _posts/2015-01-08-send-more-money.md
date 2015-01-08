---
layout: default
title: Send More Money!
topimage: top00.jpg
---

Herausforderung
---------------

Über Weihnachten bin ich über dieses Puzzle gestolpert:

    ABC - DFG = HBE
     +     -     +
      D + GCH = GCA
    ---------------
    AAE - GIE = IGF

Hier wird nach einer (eindeutigen) Zuordnung von Zahlen zu den Buchstaben gefragt.
Beim Suchen nach dem Namen des Puzzle ("Symbolrästel") bin ich bei der [Wikipedia](http://de.wikipedia.org/wiki/Mathematisches_R%C3%A4tsel) auf dieses nette Problem gestossen:

      SEND
    + MORE
    ------
     MONEY

Es wird gefragt, welchen Wert "MONEY" hat.
In diesem Artikel wird ein Weg vorgestellt, diese Art von Puzzle relativ elegant zu lösen.

Spoiler-Alert: Da unten stehen auch Lösungen.


Alternative Lösungen
--------------------

Natürlich bin ich beim Lösen per Hand erstmal gescheitert.
Natürlich habe ich mich gefragt, ob das nicht jemand anderes für mich machen kann, vielleicht jemand, der schneller rechnet.
Also ein schlaues Computerprogramm.

Den Brute Force Ansatz habe ich mir überlegt, weil bei nur (maximal) zehn Variablen gibt es nur 10! = 3628800 Möglichkeiten, diese zu setzen.
Allerdings wär das auch langweilig.
Als nächstes habe ich versucht, die Formeln in eine SAT-Formel zu pressen, um einen SAT-Solver anwenden zu können.
Diese Formeln werden etwas länglich.
Das hat sich zum Glück als so langwierig herausstellt, dass ich mich nach etwas anderem umgeguckt habe.
Dadurch bin ich auf die SMT Solver gestossen.


SMT Solver
----------

Diese basieren auf SAT Solvern, lösen aber nicht nur Probleme mit boolschen Variablen, sondern können auch auf ganze oder reelle Zahlen losgelassen werden.
Nach etwas Herumprobieren habe ich mich für [Yices](http://yices.csl.sri.com/) entschieden.
Als Sprache nehme ich [SMT-LIB Version 2](http://smtlib.cs.uiowa.edu/language.shtml), um nicht von einem SMT Solver abhängig zu sein.
Außerdem sieht die wie Lisp aus, das ist ganz praktisch.


Behauptungen
------------

Der Quellcode liegt auf GitHub in meinem [puzzle-solver](https://github.com/ocirne/puzzle-solver/tree/master/symbolraetsel) Repository.
Hier werden nur die zum Verständig wichtigen Schritte im "Send More Money"- Puzzle dargestellt.

Als einen der ersten Schritte wird die verwendete Logik gesetzt:

    (set-logic QF_AUFLIA)

Es gibt viele verschiedene.
QF_AUFLIA ist eine Quantoren-freie, die mit linearen Funktionen und beliebigen Sortierungen und Funktionen arbeiten kann.
Eine Allzweckwaffe, langsam, aber hierfür ausreichend.

Als nächstes brauchen wir Variablen für die Menge (S E N D M O R E M O N E Y) bzw. ohne Dupletten (S E N D M O R Y).
Das führt zu acht Zeilen der Form:

    (declare-fun s () Int)
       ...

Das sieht nicht nur zufällig so aus, als ob wir S als Funktion ohne Parameter mit einem ganzzahligen Rückgabewert deklariert haben.

Als nächstes müssen wir sicherstellen, dass unsere Variablen S..Y alle verschiedene Werte annehmen.
Weil Yices distinct nicht unterstützt, wird in der FAQ dieser Trick beschrieben:

    (declare-fun u (Int) Int)
    (assert (and (= (u s) 0) (= (u e) 1) (= (u n) 2) (= (u d) 3)
                 (= (u m) 4) (= (u o) 5) (= (u r) 6) (= (u y) 7)))

Dabei definiert U eine bijektive Abbildung, die durch das assert S..Y auf die ersten sieben Ziffern abbildet, es muss also für jede Ziffer eine Variable vorhanden sein.
Wohlgemerkt können S..Y andere Werte annehmen, weil da nicht s = 0, sondern u(s) = 0 steht.
Das nenne ich elegant.

Als nächstes schränken wir den Wertebereich der S..Y auf Werte von 0 bis 9 ein:

    (assert (and (<= 0 s) (<= s 9)))
       ...

und verhindern, dass die jeweils ersten Ziffern 0 sein können:

    (assert (not (= s 0)))
    (assert (not (= m 0)))

Nachdem wir drei Variablen mit den sprechenden Namen SEND, MORE und MONEY deklariert haben, stellen wir eine Verbindung zwischen der zusammengesetzten und den einzelnen Variablen her:

    (assert (is money m o n e y))

Dabei wird eine Hilfsfunktion IS verwendet, die

    money = m * 10000 + o * 1000 + n * 100 + e * 10 + y * 1

berechnet bzw. als Relation zwischen MONEY und M..Y festzurrt.
Den Abschluss der Behauptungen bildet:

    (assert (= (+ send more) money))

Das ist die obige Rätselformulierung in Syntax gegossen.


Lösung
------

Für die Lösung bitten wir den Solver, zu überprüfen, ob die Relationen zwischen den Variablen irgendeine Belegung der Variablen erlauben bzw. satisfiable sind.

    (check-sat)

Als Lösung kommt `sat` heraus, das heißt es gibt (mindestens) eine gültige Belegung für die Variablen.
Den gefundene Belegung kann man sich mit

    (get-value (m o n e y))

anzeigen lassen:

    ((m 1) (o 0) (n 6) (e 5) (y 2))

Allerdings ist jetzt noch unbekannt, ob dies die einzige Lösung ist.


Eindeutigkeit
-------------

Man kann dies recht einfach beweisen, indem man zusätzlich eine Klausel angibt, die die bekannte Lösung ausschließt:

    (assert (not (and (= m 1) (= o 0) (= n 6) (= e 5) (= y 2))))

Mit diesem Zusatz kommt beim erneuten Prüfen `unsat` heraus, das heißt es gibt keine weiteren Lösungen bzw. keine gültige Belegung, die alle Relationen erfüllt.


Schluss
-------

Dieses distinct hätte man vielleicht auch in Yices einbauen können.
Naja, da wird es einen Grund für geben, dass das fehlt.
Die SMT Solver können noch einiges mehr - vielleicht kann man die Beschränkung auf einstellige Ziffern auch in einen Datentyp einbauen?
Warum gibt es eigentlich verschiedene Logik-Einstellungen?
Wozu sollen diese Sorts gut sein?
Wieviele Fehler hab ich hier eingebaut?
Fragen über Fragen ... vielleicht muss ich noch ein paar Puzzle damit lösen.

Eigentlich ist das alles ganz nett.

