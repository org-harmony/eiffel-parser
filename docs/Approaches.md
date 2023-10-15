# Der komplexere Ansatz (Grouped)

```json
{
  "name": "User Story Card",
  "version": "1.0.0",
  "description": "A user story card following the very common format.",
  "format": "As a <role>, I want <goal/desire> so that <benefit>.",
  "author": "John Doe",
  "parsing-nodes": {
    "UserStoryCard": {
      "type": "group",
      "name": "User Story Card",
      "preprocess": [
        "to-lowercase",
        "trim-whitespace"
      ],
      "tokenize": [
        "fill-placeholders"
      ],
      "apply": [
        "placeholders-not-empty"
      ],
      "children": [
        {
          "type": "starts-with",
          "options": {
            "equals": "as a"
          }
        },
        {
          "type": "placeholder",
          "name": "role"
        },
        {
          "type": "sequence",
          "options": {
            "equals": "i want"
          }
        },
        {
          "type": "sequence",
          "options": {
            "equals": "so that"
          }
        },
        {
          "type": "placeholder",
          "name": "goal"
        },
        {
          "type": "ends-with",
          "apply": [
            {
              "equals": "."
            }
          ]
        }
      ]
    }
  }
}
```

Der komplexere Ansatz zum Parsen von Anforderungsschablonen basiert auf Parsing-Nodes (Knoten). Ein Template selbst ist ein Knoten, der wiederum 0..n Kindknoten beinhalten kann. Diese werden rekursiv abgearbeitet. 

Jeder Knoten kann 0..n Präprozessoren, 0..1 Tokenisierer, 0..n Normalisierer, 0..n Regeln und 0..n Validatoren besitzen. 

Der Verarbeitungsablauf sieht dabei wie folgt aus:
1. Präprozessoren (Preprocessor) bearbeiten die Eingabezeichenkette des Knotens vor weiterer Verarbeitung: Konvertierung in Kleinbuchstaben, Entfernen von Leerzeichen am Anfang und am Ende der Zeichenkette, ...
2. Tokenisierer (Tokenizer) können potenziell die Eingabe-Zeichenkette weiter aufteilen und neue Tokens zur weiteren Verarbeitung generieren
3. Anschließend ruft der Knoten die Kindknoten mit seinen Tokens auf. Die Ergebnisobjekte des Verarbeitens der Zeichenkette durch die Kindknoten werden der Liste an Ergebnisobjekten des Elternknotens hinzugefügt. Ergebnisobjekte beinhalten dabei Informationen über den Erfolg/Misserfolg der Verarbeitung.
4. Normalisierer (Normalizer) sorgen in der Folge dafür die Tokens noch einmal "gerade zu ziehen": Entfernen von Sonderzeichen, ...
5. Regeln (Rules) werden angewandt, um zum Beispiel zu prüfen, dass in der Eingabezeichenkette (in Form von Tokens) eine bestimmte Sub-Zeichenkette vorkommt. Ein Beispiel dafür ist ein Schlüsselwort der Anforderungsschablone.
6. Validatoren (Validators) können im Anschluss noch weitere Ergebnisobjekte zur Liste hinzufügen, indem sie die bereits existierende Liste der Ergebnisobjekte weiterverarbeiten. Validatoren können also zum Beispiel genutzt werden, um zwischen mehreren Tokens/Ergebnissen Abhängigkeiten zu überprüfen: Wenn Token A gefunden wurde, darf Token B nicht in der Ergebnisliste vorkommen. 

Dieser Ansatz ist besonders modular und erweiterbar. Die Datenstruktur eines Anforderungsschablonen-Parsers lässt sich durch eine Baumstruktur abbilden und damit auch in einem JSON-Format speichern. Theoretisch könnten so quasi unendlich komplexe Anforderungsschablonen abgebildet werden.

Die potenziellen Ergebnisse sind mit denen der Verwendung von EBNF oder PEG vergleichbar. Dabei sind die definierenden Datenstrukturen jedoch sehr viel wortreicher/ausführlicher (im Englischen wäre das passende Wort wohl: verbose).

Gleichzeitig ist damit jedoch noch lange nicht gesagt, dass die Konfiguration der gruppierten Schablonen gänzlich einfach wäre und kaum einer Erklärung des Formats vor der Verwendung bedürfe.
Ganz im Gegenteil: Trotz zugänglichen JSON-Dateiformats und einer relativ eindeutigen Datenstruktur ist die Schablone besonders aufgrund ihrer vielen Komponenten relativ schwierig zu verstehen. Zusätzlich muss verstanden werden wie und in welcher Reihenfolge das eigentliche Anwenden der Schablone (Parsen) funktioniert, wie Knoten, Sub-Knoten und Tokenisierer zusammenspielen, damit der Anwendende selbst Templates schreiben und anpassen kann. Mindestens ein minimales technisches Verständnis wird also vorausgesetzt.

Dabei stellt sich dann also die Frage, ob es sinnvoll ist quasi unbegrenzte Parsing-Möglichkeiten gegenüber einer intuitiven Template-Syntax zu bevorzugen. Die vielen Möglichkeiten ein Template aufbauen und verarbeiten zu können müssen dabei nicht zwangsläufig ein Vorteil sein, ganz im Gegenteil: Sie können ein Hindernis für die Erstellung von Templates sein, aufgrund zu großer Komplexität und zu vieler Möglichkeiten.
Zusätzlich kann eine vereinfachte Template-Syntax durch die "limitierten" Möglichkeiten die Kreativität vielleicht sogar fördern und verhindern, dass keine neuen Templates erstellt, beziehungsweise bestehenden bearbeitet werden, weil die Konfigurationsmöglichkeiten zu umfangreich und unverständlich erscheinen. (Gedanke dazu: https://medium.com/@kiwiandroiddev/the-case-for-constraint-why-fewer-options-mean-less-procrastination-and-more-creativity-d11dc0f4e741)

Insbesondere gilt es zu berücksichtigen, dass es explizit gewünscht ist, dass Anforderungsschablonen möglichst einfach konstruiert werden. Es ist erklärtes Ziel, durch Restriktionen die Anforderung konform, eindeutig und einfach zu machen. Die Sprache dieser Anforderung soll langweilig und verständlich sein, deshalb überhaupt erst Schablonen.

Wenn also die Möglichkeiten eine Schablone zu konstruieren so komplex sind, dass sie unverständlich und "intensivst erklärenswürdig" werden, besteht das Risiko, das die mit diesem Tool definierten und prüfbaren Anforderungsschablonen ebenso komplex wie ihre Konfigurationsmöglichkeiten werden könnten. Damit wäre der Verständlichkeit der Schablonen ein absoluter Bärendienst erwiesen worden.

So scheint es also als könnte eine durchdachte Daten- und Verarbeitungsstruktur speziell für Anforderungsschablonen die Komplexität dieser deutlich reduzieren. Die Syntax einer Anforderungsschablone sollte daher ebenso einfach wie die Schablone selbst sein.

# Der einfachere Ansatz

Weg von einer generischen Lösung um Sprache zu verarbeiten, hin zu einem maßgeschneiderten Datenformat für Anforderungsschablonen.

```json
{
  "name": "Funktionale Anforderung",
  "version": "1.0.0",
  "description": "Eine funktionale Anforderung (ESFA) nach PARIS angepasst auf die Formulierungen der Firma XY.",
  "author": "John Doe",
  "format": "Das System <Systemname> muss|soll|sollte|wird <Rolle> ermöglichen, <Ziel>, damit <Begründung>.",
  "preprocess": [
    "toLowercase",
    "trimWhitespace",
    "trimPunctuation"
  ],
  "rules": {
    "das-system": {
      "name": "\"Das System\"",
      "type": "text",
      "value": "das system"
    },
    "systemname": {
      "name": "Systemname",
      "type": "placeholder"
    },
    "modalverb": {
      "name": "Modalverb",
      "type": "text-enum",
      "values": [
        "muss",
        "soll",
        "sollte",
        "kann",
        "wird"
      ]
    },
    "rolle": {
      "name": "Rolle",
      "type": "placeholder"
    },
    "ermöglichen": {
      "name": "\"ermöglichen\"",
      "type": "text",
      "value": "ermöglichen"
    },
    "ziel": {
      "name": "Ziel",
      "type": "placeholder"
    },
    "damit": {
      "name": "\"damit\"",
      "type": "text",
      "value": "damit",
      "optional": true
    },
    "begründung": {
      "name": "Begründung",
      "type": "placeholder",
      "optional": true
    },
    "wenn": {
      "name": "\"wenn\"",
      "type": "text",
      "value": "wenn"
    },
    "bedingung": {
      "name": "Bedingung",
      "type": "placeholder"
    }
  },
  "variations": {
    "Ohne Bedingung": [
      "das-system",
      "systemname",
      "modalverb",
      "rolle",
      "ermöglichen",
      "ziel",
      "damit",
      "begründung"
    ],
    "Mit Bedingung": [
      "wenn",
      "bedingung",
      "modalverb",
      "das-system",
      "systemname",
      "rolle",
      "ermöglichen",
      "ziel",
      "damit",
      "begründung"
    ]
  }
}
```

Die vereinfachte Schablone verwendet ebenso wie die komplexere Schablone Präprozessoren und Regeln. Auch hier sind die Präprozessoren gedacht, um die Eingangszeichenkette vorzuverarbeiten: Leerzeichen am Anfang und Ende entfernen, Satzzeichen entfernen, Text zu Kleinschreibung konvertieren.

In der vereinfachten Datenstruktur können dann nicht-verschachtelte Regeln statt Knoten verwendet werden. Regeln sind dabei prüfbare Elemente, die der Parser nutzt, um die Konformität einer Eingabezeichenkette mit einer Schablone zu verifizieren. Eine Regel ist dabei immer von einem bestimmten Typ: Text, Platzhalter, Aufzählung, etc.: 
- So definiert ein Platzhalter beispielsweise, dass hier ein nicht-leerer Text stehen muss, dessen Inhalt noch nicht bekannt ist.
- Der Text-Typ definiert, dass ein bestimmter Text an dieser Stelle stehen muss, dessen Inhalt definitiv bekannt ist.
- Die Aufzählung definiert hingegen, dass der vorzufindende Text einem bestimmten, in der Aufzählung definierten Text, entsprechen muss.
Eine Regel ist nur optional, wenn sie durch die Einstellung "optional" mit "true" als optional definiert wird.

Jedes Template kann Variationen definieren. Das Template ist erfüllt, wenn eine der Variationen erfüllt ist. Jede Variation kann anhand des Namens beliebige Regeln verwenden. Präprozessoren werden auf das gesamte Template und alle seine Variationen angewandt.
Die Regeln müssen in der vorgegebenen Reihenfolge erfüllt werden. Ist bspw. Text C vor Text A anzufinden, obwohl das Template die Reihenfolge A -> B -> C erwartet, würde der Parser einen Fehler ausgeben.

Somit ist eigentlich die gesamte Funktionsweise erklärt. Es können noch beliebige weitere Regeln hinzugefügt werden und auch ließe sich eine Art Meta-Regel/Validator implementieren, die im Anschluss an die anderen Regeln noch die Ergebnisse überprüft und womöglich Abhängigkeiten zwischen Regeln prüft. Dafür scheint es aktuell jedoch keinen Anwendungsfall zu geben.

Dieser Parsing-Ansatz ist aufgrund seiner flachen, nicht-hierarchischen Struktur sehr viel einfacher. Damit kann auch das Datenformat für die Definition von Parsern verständlicher sein.

Neben dem Vorteil verbesserter Verständlichkeit der Template-Datenstruktur und -Dateien ist der Hauptvorteil dieses vereinfachten Schablonen-Parsers jedoch sicherlich die Möglichkeit für bessere Fehlermeldungen: Je komplexer und verschachtelter der Parser, desto komplexer wird es auch potenziell tief verschachtelte Fehlermeldungen und Verbesserungsvorschläge sinnvoll und hilfreich zu formulieren und richtig zu verbreiten.
Mit der einfacheren Parser-Struktur ist es möglich Fehlermeldungen zu liefern die zum Beispiel wie folgt lauten könnten:
```text
Fehler in der Schablone "Funktionale Anforderung":

Gegebener Anfang der Anforderung: "In der Anwendung XYZ ..."
Erwartet wird ein Anfang aus:
- Ohne Bedingung: "Das System <Systemname> muss|soll|sollte|kann|wird ..."
- Mit Bedingung: "Wenn <Bedingung> muss|soll|sollte|kann|wird ..."
```

Aufgrund der separierten und verschachtelten Natur des komplexeren Ansatzes wären Fehlermeldungen wie diese nur schwierig möglich.
Der vereinfachte Ansatz könnte (wie in diesem Beispiel) schon beinahe die Schablone Schritt für Schritt erklären.
