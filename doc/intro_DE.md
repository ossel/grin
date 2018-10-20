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

In der Elliptische-Kurven-Kryptografie wählt man als private Schlüssel sehr große Zahlen. Für einen privaten Schlüssel _k_ erhält man den entsprechenden öffentlichen Schlüssel durch die Berechnung von `k*H`. Es ist kaum möglich von dem öffentlichen Schlüssel `k*H` den privaten Schlüssel _k_ herzuleiten. Dies liegt daran, dass in der Elliptischen-Kurven-Kryptografie Multiplikationen trivial, "Divisionen" allerdings extrem aufwendig sind.


Die obige Formel `(k+j)*H = k*H + j*H`, mit den privaten Schlüsseln _k_ und _j_ demonstriert, dass der öffentlicher Schlüssel den man durch das Addieren der privaten Schlüssel _k_ und _j_ erhält (`(k+j)*H`), identisch zu dem öffentlichen Schlüssel ist, den man durch die Addition der öffentlichen Schlüssel beider privaten Schlüssel (`k*H + j*H`) erhält. Sogenannte Hierarchical Deterministic Wallets der Bitcoin Blockchain (BIP 0032) als auch MinmbleWimble und die Grin Implementierung basieren auf diesem Prinzip.

### MimbleWimble-Transaktionen

Der Aufbau von MimbleWimble-Transaktionen liefert starke Privatsphäre- und Vertraulichkeitsgarantien.

Die Validierung von MimbleWimble-Transaktionen beruht auf zwei grundlegenden Eigenschaften:

* **Überprüfung der Nullsummen** Die Summe der Outputs minus die Summer der Inputs muss immer gleich Null sein. Dies beweist, dass die Transaktion kein neues Geld aus dem Nichts ersschaffen hat, ohne dabei die tatsächlichen Beträge der Transaktion zu offenbaren.
* **Besitz von privaten Schlüsseln.** Wie bei den meisten anderen Kryptowährungen muss der Besitz von Outputs durch den Besitz von privaten Schlüsseln garantiert werden. In MimbleWimble erfolgt der Beweis des Besitzes allerdings nicht durch das Signieren der Transaktion.

Die nächsten Abschnitte zu Transaktionsbeträgen, Eigentumsrechten, Wechselgeld und Range-Proofs beschreiben, wie diese beiden grundlegenden Eigenschaften werden erreicht.

#### Transaktionsbeträge

Die bereits oben eingeführten Eigenschaften der Elliptische-Kurven-Kryptografie ermöglichen es Transaktionsbeträge zu verschleiern.

Betrachtet man einen einen Input oder Output mit dem Wert _v_  und eine elliptische Kurve _H_, so kann man statt dem unverschleierten Betrag _v_ den  Wert `v*H` in der Transaktion angeben. 

Die Operationen der Elliptischen-Kurven-Kryptografie erlauben es immer noch zu prüfen, dass die Summer der Inputs gleich der Summe der Outputs der Transaktion ist, obwohl die Beträge nun verschleiert sind:

    v1 + v2 = v3 => v1 * H + v2 * H = v3 * H

Das MimbleWimble Protokoll prüft diese Eigenschaft für jede Transaktion und stellt so sicher, dass kein Geld aus dem Nichts erschaffen wird.
Problematisch dabei ist lediglicht, dass es nur eine endliche Anzahl benutzbarer v-Werte gibt und man diese der Reihe nach durchprobieren könnte um den Transaktionsbetrag zu erraten. Außerdem deckt die Kenntnis eines v1-Wertes (von einer vorherigen Transaktion) und des entsprechenden `v1*H`-Werts alle restlichen v1-Werte der Outputs der gesamten Blockchain auf. Aus diesem Grund führen wir eine weitere elliptische Kurve _G_ (Orginal: practically _G_ is just another generator point on the same curve group as _H_) und einen privaten Schlüssel _r_, der als sogenannter *blinding factor* fungiert, ein. 


Ein Input oder Output einer Transaktion sieht somit folgendermaßen aus:

    r * G + v * H

Wobei:

* _r_ ein privater Schlüssel ist, der als *blinding factor* verwendet wird. _G_ ist eine elliptische Kurve und ihr Produkt `r * G` ist der öffentliche Schlüssel für _r_ auf _G_.
* _v_ ist der Wert eines Inputs oder Outputs. _H_ ist eine andere elliptische Kurve.

Weder _v_ noch _r_ können, dank der fundamentalen Eigenschaften der Elliptischen-Kurven-Kryptografie, abgeleitet werden.
Kurven-Kryptographie. `r*G + v*H` wird als _Pedersen Commitment_ bezeichnet.


Betrachten wir beispielsweise eine Transaktion mit zwei Inputs und einem Output. Dann haben wir (Transaktionsgebühr wird vernachlässigt):
* vi1 und vi2 als Werte der Inputs.
* vo3 als Wert des Outputs.


So dass:

    vi1 + vi2 = vo3

Nun generieren wir die privaten Schlüsseln ri1 und ri2 als *blinding factor* für die Beträge vi1 und vi2. Ersetzt man diese Beträge in der obigen Gleichung durch das entsprechende _Pedersen Commitment_, erhält man:

    (rl1 * G + vi1 * H) + (rl2 * G + vi2 * H) = (ro3 * G + vo3 * H)

Dies erfordert, dass:

    ri1 + ri2 = ro3

Dies ist die erste Säule von MimbleWimble: Die Arithmetik, die benötigt wird um Transaktionen zu validieren, kann ausgeführt werden, ohne dass die eigentlichen Werte der Transaktion offengelegt wurden.


Diese Idee stammt aus [Confidential Transactions] (https://www.elementsproject.org/elements/confidential-transactions/) von Greg Maxwell, welche wiederum auf Arbeiten von Adam Back aufbauen.

#### Eigentumsrechte

Im vorherigen Abschnitt haben wir einen privaten Schlüssel als *blinding factor* eingeführt, um Transaktionsbeträge zu verschleiern. Die zweite Einsicht von MimbleWimble ist, dass die Kenntnis dieses privaten Schlüssels dazu verwendet werden kann, die Eigentumsrechte eines Werts zu beweisen.

Alice sendet dir 3 Münzen und um diesen Betrag zu verschleiern, wählst du 28 als *blinding factor*. Da es sich bei dem *blinding factor* um einen privaten Schlüssel handelt, ist dieser in der Praxis eine extrem große Zahl. Irgendwo auf der Blockchain erscheint der folgende Output:

    X = 28*G + 3*H

Dieser Output gehört dir und sollte lediglich von dir ausgegeben werden können.
_X_ als Ergebnis der Addition, ist für jeden sichtbar. Der Wert 3 ist nur dir und Alice bekannt. Der Wert 28 ist nur dir bekannt.

Um diese 3 Münzen erneut zu übertragen, muss das Protokoll den Wert 28 irgendwoher kennen. Dies wird durch die folgende Überlegung deutlich. Nehmen wir an, dass du diese drei gleichen Münzen an Carol überweisen möchtest. Dazu musst eine folgendermaßen aufgebaute Transaktion erstellt werden:

    Xi => Y

Dabei ist _Xi_ ein Input, der deinen Output _X_ ausgibt. _Y_ ist der Output für Carol. Es gibt keine Möglichkeit solch eine Transaktion zu erstellen, ohne den privaten Schlüssel 28 zu kennen. Möchte Carol solch eine Transaktion erstellen, ist dies nur möglich falls sie sowohl deinen privaten Schlüssel 28 als auch den Transaktionsbetrag 3 kennt. Kennt sie diese Werte nicht ist es für sie unmöglich die Transaktion auszubalancieren. Damit Carol eine gültige Transaktion erstellen kann, muss folgendes gelten:

    Y - Xi = (28*G + 3*H) - (28*G + 3*H) = 0*G + 0*H

Indem wir überprüfen, dass der Output-Wert minus den Input-Wert Null ergibt, prüfen wir, dass die Transaktion ausbalanciert ist und stellen dadurch sicher, dass kein neues Geld erzeugt wurde.

Halt! Stop! Jetzt kennst du den privaten Schlüssel von Carols Output (der in diesem Fall der Gleiche sein muss wie der deines Outputs, damit die Transaktion ausbalanciert sein kann) und könntest somit das Geld von Carol stehlen!

Um dieses problem zu lösen, benutzt Carol einen privaten Schlüssel ihrer Wahl.
Sie wählt beispielsweise den Wert 113, und was auf der Blockchain endet, ist:

    Y - Xi = (113 · G + 3 · H) - (28 · G + 3 · H) = 85 · G + 0 · H

Nun summiert sich die Transaktion nicht mehr zu Null und wir haben einen sogenannten _excess value_ auf der Kurve _G_ (85), der das Ergebnis der Summe aller *blinding factor*en ist. Da `85 * G` ein gültiger öffentlicher Schlüssel auf der elliptischen Kurve _G_ ist, mit privatem Schlüssel 85,
für jedes x und y, nur falls `y = 0` ist und `x*G + y*H` ein gültiger öffentlicher Schlüssel auf _G_ ist.

Das Protokoll muss also lediglich verifizieren, dass (`Y - Xi`) ein gültiger öffentlicher Schlüssel auf _G_ ist und dass beide Parteien kollektiv den privaten Schlüssel (85 in der Transaktion mit Carol) kennen. Die einfachste Art, dies zu beweisen, besteht darin, eine Signatur mit Hilfe des *excess value* (85) zu generieren. Diese Signatur beweist, dass:

* die handelnden Parteien den privaten Schlüssel gemeinsam kennen.
* die Summe der Outputs minus der Inputs der Transaktion den Wert Null ergibt.
  (weil nur ein gültiger öffentlicher Schlüssel die Signatur, die mit dem entsprechenden privaten Schlüssel erzeugt wurde, validieren kann).

Diese Signatur wird jeder Transaktion, zusammen mit einigen zusätzlichen Daten (wie z.B. der Mining-Gebühren), beigefügt und von allen Validatoren überprüft. Diese beigefügten Daten werden _transaction kernel_ genannt.

#### Einige Feinheiten

Dieser Abschnitt beschäftigt sich mit dem Aufbau von Transaktionen, indem diskutiert wird, wie Wechselgeld realisiert ist und warum man sogenannte *range proofs* benötigt, die beweisen, dass die Werte der Transaktion nicht negativ sein können. Beide Punkte sind nicht unbedingt erforderlich, um MimbleWimble und
Grin zu verstehen. Falls du es eilig hast, kannst du direkt zur
[Zusammenfassung](#zusammenfassung) springen.

##### Wechselgeld

Nehmen wir an du möchtest nur 2 Münzen von den 3 Münzen, die du von Alice erhalten hast, an Carol versenden. Dazu musst du die verbleibende Münze als Wechselgeld an dich zurücksenden. Dazu musst du einen weiteren Output generieren, den du durch einen privaten Schlüssel (sagen wir 12) als *blinding factor* schützt. Carol benutzt wie zuvor ihren eigenen privaten Schlüssel.

    Wechselgeld Output:   12*G + 1*H
    Carols Output:        113*G + 2*H

Das was auf der Blockchain endet, ist sehr ähnlich zum vorherigen Beispiel.
Wieder wird die Signatur mit dem *excess value* (in diesem Beispiel 97) erstellt.

    (12*G + 1*H) + (113*G + 2*H) - (28*G + 3*H) = 97*G + 0*H

##### Range Proofs

#### Zusammenfassung

### MimbleWimble Blöcke und Blockchain-Status
