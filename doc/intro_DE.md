# Einführung bezüglich MimbleWimble und Grin

*Andere Sprachen: [English](intro.md), [简体中文](intro.zh-cn.md), [Español](intro_ES.md).*


MimbleWimble ist ein Blockchain-Format und Protokoll, das durch die Verwendung kryptographischer Bausteine besonders gute Skalierbarkeit, Privatsphäre und Fungibilität ermöglicht.
Diese Eigenschaften fehlen in fast allen bisherigen Blockchain-Implementierungen.

Grin ist ein Open-Source-Software-Projekt, das eine MimbleWimble-Blockchain implementiert und dabei alle vorhandenen Lücken für die Bereitstellung einer Kryptowährung schließt.

Die Hauptziele und Merkmale des Grin-Projekts sind:

* Standardmäßige Privatsphäre. Private Transaktionen ermöglichen vollständige Fungibilität, erlauben es aber dennoch Informationen bei Bedarf selektiv offenzulegen.
* Skaliert mit der Anzahl Nutzer statt mit der Anzahl Transaktionen und führt somit zu großer Platzersparnis im Verglichen zu anderen Blockchains.
* Starke und bewährte Kryptographie. MimbleWimble verwendet ausschließlich Elliptische-Kurven-Kryptografie, die breits seit Jahrzehnten erprobt ist.
* Schlichtes Design, das einfache Pflege und Überprüfbarkeit ermöglicht.
* Community-gesteuert, mit einem ASIC-resistenten Mining-Algorithmus (Cuckoo Cycle), der Mining-Dezentralisierung fördert.

## Zungenknoten für alle

Dieses Dokument richtet sich an Leser mit einem guten Verständnis für Blockchains und einem grundlegenden Verständnis von Kryptographie. 
Zunächst wird der technische Aufbau von MimbleWimble vorgestellt um anschließend darauf einzugehen, wie dieser in Grin verwendet wird. Wir hoffen dieses Dokument ist für technisch versierte Leser verständlich. Unser Ziel ist es dein Interesse für Grin zu wecken und dich zu Beiträgen jeglicher Art zu ermutigen.

Um dieses Ziel zu erreichen, werden wir die wichtigsten Konzepte, die für ein gutes Verständnis von Grin als MimbleWimble-Implementierung notwendig sind, vorstellen. Wir beginnen mit einer kurzen Beschreibung der relevanten Eigenschaften der Elliptischen-Kurven-Kryptografie auf der Grin basiert. Anschließend beschreiben wir die wichtigsten Elemente aus denen eine MimbleWimble-Blockchain besteht: Transaktionen und Blöcke.

### Einführung Elliptische-Kurven-Kryptografie
Im Folgenden werden ausschließlich die Eigenschaften betrachtet, die für ein Verständnis von MimbleWimble notwendig sind ohne dabei zu tief in die Feinheiten der Elliptischen-Kurven-Kryptografie einzutauchen. Leser, die tiefer in die Annahmen der Elliptischen-Kurven-Kryptografie eintauchen möchten, finden [hier](http://andrea.corbellini.name/2015/05/17/elliptic-curve-cryptography-a-gentle-introduction/) zusätzliche Informationen.


Eine Elliptische Kurve ist im Sinne der Kryptographie einfach eine große Menge von Punkten, die wir im Folgenden _C_ nennen. Diese Punkte können addiert, subtrahiert oder mit ganzen Zahlen (auch Skalaren genannt) multipliziert werden. Für eine gegebene ganze Zahl _k_ kann man mit Hilfe der Skalarmultiplikation `k*H` berechnen, was wiederum auch ein Punkt auf der Kurve _C_ ist. Für eine weitere gegebene ganze Zahl _j_ kann man `(k+j)*H` berechnen, was gleich `k*H + j*H` ist. Die Addition und Skalarmultiplikation auf einer Elliptischen Kurve sind kommutativ und assoziativ:

    (k+j)*H = k*H + j*H



### MimbleWimble-Transaktionen


### MimbleWimble Blöcke und Blockchain-Status
