---
layout: default
title: Interessanter Fehler
topimage: top00.jpg
---

Für alle die denken, dass ich keine anderen Hobbys neben jonglieren habe.

Ende letzten Semesters bin ich in Java auf einen interessanten Fehler gestoßen.
Hier ist der Code zwecks didaktischer Vereinfachung radikal verkürzt.
Dadurch wäre das Problem mit Vector oder ArrayList trivial zu lösen, aber darauf kommt es hier nicht an.

Zuerst definieren wir einen Term.
Dieser enthält eine Liste mit Zahlen und die Anzahl, wieviel Zellen der Liste wirklich belegt sind.

    class Term {
        private int count = 0;
        private int[] values = new int[1];

Die Methode `getIndex ()` soll den Index eines freien Platzes in `values` zurückliefern.
Falls values schon gefüllt ist (d.h. `count_values == values.length`), wird ein doppelt so großes Array erzeugt und die alten Werte werden übertragen.

        private int getIndex () {
            if ( count == values.length ) {
                int[] tmp = new int[2*count];
                for ( int i = 0; i < count; i++ ) {
                    tmp[i] = values[i];
                }
                values = tmp;
            }
            return count++;
        }

Nun werden zwei Methoden definiert, um einen Wert in die Liste einzufügen.
Einmal wird der Index zwischengespeichert und einmal direkt verwendet.
Sollte keinen Unterschied machen, oder?

        // So gehts
        public void addTerm_correct (int i) {
            int index = getIndex ();
            values[index] = i;
        }
    
        // So nicht
        public void addTerm_wrong (int i) {
            values[getIndex ()] = i;
        }
    }

Die Hauptklasse erzeugt jeweils einen Term und versucht, zehn Werte einzufügen.
Nur der erste Fall ist erfolgreich, der zweite erzeugt eine Exception.

    class InteressanterFehler {
    
        public static void main (String[] args) {
            System.out.print ("Testfall 1 ... ");
            Term root1 = new Term ();
            for ( int i = 0; i < 10; i++ ) {
                root1.addTerm_correct (i);
            }
            System.out.println ("erfolgreich.");
    
            System.out.print ("Testfall 2 ... ");
            Term root2 = new Term ();
            for ( int i = 0; i < 10; i++ ) {
                root2.addTerm_wrong (i);
            }
            System.out.println ("erfolgreich.");
        }
    }

Warum?
`Term.getIndex ()` hat den Seiteneffekt, dass der "Zeiger" auf values umgebogen wird.
Eine miese kleine Fehlerquelle.

Link zum [Quellcode](./studium/InteressanterFehler.java) (968 B)

