----
math: katex
header: Gefahrengut | ERP-Systeme | Agreš, Kiriakou, Koch, Nur
paginate: false
style: |
  pre {
    font-size: 16px;
  }
  code {
    font-size: 22px;
  }
  li, p, td, th {
    font-size: 26px;
  }
  .columns {
    display: flex;
    gap: 1rem;
  }
  .columns > div {
    flex: 1 1 0;
  }

----

# Stammdaten - Gefahrengut
 
## ERP-Systeme

Dominik Agreš, Chris Kiriakou, David Koch, Berkan Nur

<style scoped>
h1 {
    font-size: 80px;
    text-align: center;
    padding: 10px;
    margin: 10px;
}

h2 {
    font-size: 50px;
    text-align: center;
    padding: 10px;
    margin: 10px;
}

section {
    text-align: center;
}

header {
    color: #FFFFFF00;
}
</style>

----

# Gliederung

1. Einführung Gefahrengut
2. Bezeichnungsanalyse
3. Fehlererkennung & Behebung (KI)
4. Fehlererkennung & Behebung (ohne KI)
8. Fazit 

<!-- paginate: true -->

<!--
-->

----

# Einführung Gefahrengut

| Bezeichnung        | Beschreibung                                                        |
|--------------------|---------------------------------------------------------------------|
| Landecode für GG   | z.B. CA = Kanada, HU = Ungarn                                       |
| Art_IdentNr        | Einzige Art in Stammdaten = UN                                      |
| IdentNr            | UN-Nummern, Stoffnummern - jedes Gefahrengut hat eig. Nummer        |
| Klasse             | Gefahrgutklassen Unterteilt in 13 Klasen, mit Unterklassen (.2, .1) |
| GG_Vorschrift See  | IMDG, International Maritime Dangerous Goods Code                   |
| Verp. Methode See  |                                                                     |
| GG_Vorschrift Luft | IATA_C, IATA_P, International Air Transport Association             |
| Verp. Methode Luft |                                                                     |

<!--
-->

---- 
 
# Bezeichnungsanalyse

![bg right:47% height:80% vertical](./assets/gg-top-50-words-masterdata.png)
![bg right:47% height:80%](./assets/gg-top-50-words-un-numbers.png)

- Analyse semantische Beziehungen **'Material-Bezeichnung'** in den Stammdaten mit den Bezeichnern in UN-Nummern hergestellt
- Durch Beziehungen zuordnen Materialien v. Modulgruppen zu UN-Nummern 
- Für effizienteres Verarbeiten der Daten *Grundrauschen* entfernen

<!--
-->

----

# Bezeichnungsanalyse

![bg right:47% height:80% vertical](./assets/gg-top-50-words-masterdata-keywords.png)
![bg right:47% height:80%](./assets/gg-top-50-words-un-numbers-keywords.png)

- Filtern `stop_words` aus Bezeichnungsfeldern
- `stop_words` haben keinen Semantischen Mehrwert $\rightarrow$ entfernen!
```python
stop_words = {
    'und', 'mit', 'für', 'von',
    'in', 'an', 'auf', 'im',
    'vst', 'stg', 'nag' ...
    }
    
def sanitize_text(text):
    if not isinstance(text, str):
        return set()
    text = (
        text.lower()
        .replace('ae', 'ä')
        .replace('oe', 'ö')
        .replace('ue', 'ü')
        ...
    )
```

<!--
-->

----

# Bezeichnungsanalyse

- Vergleichen jedes Schlüsselworts aus **'Material-Bezeichnung'** mit allen Schlüsselwörtern der UN-Nummern $\rightarrow$ festgestellt welche Zeilen welchen der UN-Nummern ähneln
- Verfahren $m \times n$: schlechte Laufzeitkomplexität
- **Fuzzy-Algorithmus**: Erkennen gleicher Substrings der Schlüsselwörter
- **Nachteil**: Kontext wird nicht berücksichtigt, trotz ähnlichkeit entstehen Fehler

![height:250px](./assets/gg-designation-analysis.png)

<style scoped>
p { text-align: center; }
</style>

<!--
-->

----
 
# Fehlererkennung & Behebung (KI)

<!--
-->

----

# Fehlererkennung & Behebung (ohne KI)

- Erkennen von Unregemäßigkeiten:
    - Spalte **'Art_IdentNr'** prüfen ob Wert `UN` gesetzt
    - Wenn ja, aus Spalte 'Material-Bezeichnung' der gleichen Zeile das erste Wort entnehmen
- Masken: Verundung durch gesetzte & ungesetzte **'Art_IdentNr'**-Spalte $\rightarrow$ Gruppieren "ähnlicher" Material-Bezeichner

![height:200px](./assets/gg-inconsistency-detection.png)

<style scoped>
p { text-align: center; }
</style>

<!--
-->

----

# Fehlererkennung & Behebung (ohne KI)

<div class="columns">
<div>

- Leere Zeilen mit Werten aus gleicher Wortgruppe ausfüllen
- **Annahme**: Bereits ausgefüllte Zeilen mit `UN` sind korrekt

**Definition Wortgruppe**:
> Teilwort umschließt jedes Materialbezeichnungsfeld welches dieses besitzt
 
</div>
<div>

```Python
for first_word in inconsistent_first_words:
    group = grouped.get_group(first_word)

    # Find valid row: must be 'UN'
    valid_source_rows = group[
        (group['Art_IdentNr'] == 'UN') &
        (group['IdentNr'].notna()) &
        (group['Klasse'].notna())
    ]
```
```Python
for idx, row in group.iterrows():
    row_dict = row.to_dict()
    row_dict['Ursprungsindex'] = idx

    if row['Art_IdentNr'] != 'UN':
        row_dict['Art_IdentNr'] = 'UN'
        if pd.isna(row['IdentNr']):
            row_dict['IdentNr'] = correct_ident
        if pd.isna(row['Klasse']):
            row_dict['Klasse'] = correct_klasse
```
</div>
</div>

<!--
-->

----

# Fehlererkennung & Behebung (ohne KI)

- Zuordnung zu **1. Teilwort** funktioniert im Falle von Stoßdämpfern gut
- Bei Aktuatoren z.B. "AKTUATOR SOUNDSYS" & "AKTUATOR PARKSPERRE" nicht:
    - Soundsystem ist kein Gefahrengut
    - **'IdentNr'** 3268 $\rightarrow$ *Airbag-Gasgeneratoren, pyrotechnisch oder Airbag-Module, pyrotechnisch oder Gurtstraffer, pyrotechnisch oder Sicherheitseinrichtungen, elektrische Auslösung*
 
![height:200px](./assets/gg-inconsistency-detection-correct.png)

<style scoped>
p { text-align: center; }
</style>
 
<!--
-->

----

# Fazit

- KI generiert viele Fehler z.B. 'HINWEISSCHILD BATTERIE'
- Ohne KI (Teilwortgruppierung) übersieht Kontext, daher auch Fehleranfällig
- Mögliche Erweiterungen/Alternativen:
    - Maschinelles Lernen, Klassifizierung (UN-Nummern als Label für **'Material-Bezeichnung'**)
    - Für Klassifizierung werden aber korrekte Daten benötigt

<!--
-->

----

# Quellen

- UN-Nummer Liste [https://de.wikipedia.org/wiki/Liste_der_UN-Nummern](https://de.wikipedia.org/wiki/Liste_der_UN-Nummern)
- IATA [https://www.iata.org/](https://www.iata.org/)
- Github Repository mit Notebook und Präsentation [https://github.com/ckiri/gg](https://github.com/ckiri/gg)

<!--
Bitte fehlende Quellen hinzufügen
-->
 
<style scoped>
li {
    font-size: 14px;
}
</style>
