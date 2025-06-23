# Präsentationsfolien 

In der Datei `slides.md` werden die Inhalte der Präsentation in
Markdown geschrieben.

Marp generiert dann aus `slides.md` ➡ `slides.html`.

Das HTML Dokument muss nicht commited werden. Dieses wird automatisch von
GitHub Actions generiert und deployed.

## VS-Code Extension

VS-Code Marp Erweiterung [installieren](https://marketplace.visualstudio.com/items?itemName=marp-team.marp-vscode).

## Ohne VS-Code

> [!NOTE]  
> Marp benötigt [NPM](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm) um die Präsentationsfolien zu bauen.

### Marp Live Preview

> [!NOTE]  
> Folgende Befehle in `docs/slides` ausführen.

Um während dem Arbeiten an `slides.md` Änderungen in Echtzeit zu sehen:
```sh
npx @marp-team/marp-cli@latest --server .
```

### Bauen der Folien

> [!NOTE]  
> Folgende Befehle in `docs/slides` ausführen.

Das Command zum bauen ist im Makefile integriert.

Bauen der Präsentationsfolien `make`:
```sh
make
```

Präsentationsfolien als `.pdf`:
```sh
make pdf
```

Beides (PDF & HTML):
```sh
make pdf slides
```

Für Lokale Verwendung der Präsentation einfach `slides.html` im Browser öffnen.
