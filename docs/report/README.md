# GG Dokumentation

Die Dokumentation ist in LaTeX geschrieben.

Es wird eine LaTeX Distribution benötigt, z.B  TeX-Live:
* [Windows Installation](https://www.tug.org/texlive/windows.html)
* [MacOS Installation](https://www.tug.org/mactex/)

## Linux/MacOS

Generierung der Doku:
Nach der Installation können die Dokumente, im `report` Ordner, mit dem `Makefile` gebaut werden:
```sh
make
```

## Windows

Oder alternativ, falls kein Make vorhanden ist, können folgende Befehle im `report` Ordner ausgeführt werden.

Einmalig (Zum erzeugen des Ordners):

```sh
mkdir build
```

Generierung der Doku:
```sh
pdflatex -output-directory=./build -jobname=$(REPORT_NAME) main.tex 
```
