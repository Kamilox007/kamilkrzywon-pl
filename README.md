# kamilkrzywon.pl

Strona osobista: portfolio projektów, notatki techniczne, CV i informacje
o korepetycjach. Statyczna — zero JavaScriptu po stronie klienta.

Wersja na żywo: <https://kamilkrzywon.pl>

## Stack

| Warstwa | Technologia |
|---|---|
| Generator | [Astro 7](https://astro.build) — build statyczny |
| Treść | Markdown w kolekcjach z walidacją schematu (Zod) |
| Matematyka | KaTeX przez `remark-math` + `rehype-katex` |
| Podświetlanie składni | Shiki (`github-light`) |
| Typografia | Newsreader Variable (tekst), IBM Plex Mono (interfejs) |
| CV | LaTeX → PDF, kompilowane skryptem |
| Serwowanie | Caddy w kontenerze, za reverse proxy |

## Uruchomienie

```bash
npm install
npm run dev          # http://localhost:4321
```

| Polecenie | Działanie |
|---|---|
| `npm run dev` | serwer deweloperski |
| `npm run build` | build produkcyjny do `dist/` |
| `npm run preview` | podgląd builda lokalnie |
| `npm run check` | kontrola typów i odwołań w Astro |
| `npm run cv` | kompiluje `cv/cv.tex` → `public/cv.pdf` |

## Struktura

```
src/
  content/
    blog/            notatki techniczne (.md)
    projekty/        opisy projektów (.md)
  content.config.ts  schematy kolekcji
  layouts/
    Base.astro       szkielet strony, nagłówek, stopka, meta
    Wpis.astro       układ pojedynczego wpisu
  pages/             trasy — plik = adres
  styles/
    global.css       całość stylów, zmienne CSS, motyw ciemny
public/
  cv.pdf             artefakt kompilacji LaTeX-a (commitowany)
cv/
  cv.tex             źródło CV
scripts/
  build-cv.mjs       kompilacja CV
```

## Dodawanie treści

Nowy wpis to plik `.md` w `src/content/blog/`. Nazwa pliku staje się adresem.
Frontmatter jest walidowany przy budowaniu — literówka w polu wywala build,
a nie produkcję:

```markdown
---
title: Tytuł wpisu
opis: Jedno zdanie, trafia do opisu meta i na listę wpisów.
data: 2026-08-05
tagi: [python, sqlite]
szkic: false
---
```

Projekty działają tak samo, w `src/content/projekty/`, ze schematem
`title`, `opis`, `stack`, `repo`, `demo`, `kolejnosc`, `szkic`.

Ustawienie `szkic: true` wyklucza wpis z listy i z RSS.

## CV

`public/cv.pdf` jest **commitowany do repozytorium**, bo obraz Dockera nie
zawiera dystrybucji LaTeX-a — dokładanie TeX Live do builda oznaczałoby
kilkaset megabajtów na artefakt zmieniający się parę razy w roku.

Konsekwencja: po każdej zmianie `cv/cv.tex` trzeba wykonać

```bash
npm run cv
git add public/cv.pdf cv/cv.tex
```

Bez tego źródło CV i opublikowany PDF rozjadą się po cichu.

Skrypt próbuje najpierw `latexmk`, a gdy go nie znajdzie — `pdflatex`
uruchomiony dwukrotnie (drugi przebieg domyka odwołania wewnętrzne).
Wymaga MiKTeX-a na Windowsie albo TeX Live na Linuksie i macOS.

## Matematyka we wpisach

Astro 7 renderuje Markdown domyślnie procesorem `satteri` (Rust), który
przyjmuje `mdastPlugins`/`hastPlugins`, ale **nie uruchamia wtyczek
remark/rehype**. KaTeX wymaga więc jawnego przełączenia na pipeline `unified`
— zrobione w `astro.config.mjs`. Jeśli wzory przestaną się renderować,
to jest pierwsze miejsce do sprawdzenia.

Składnia: `$x^2$` w linii, `$$...$$` w bloku.

## Wdrożenie

Obraz dwustopniowy: Node buduje stronę, wynik trafia do obrazu z Caddym.
Kontener serwuje wyłącznie pliki statyczne po HTTP — terminacja TLS
i certyfikaty należą do reverse proxy stojącego przed nim.

```bash
docker compose up -d --build site
```

`Caddyfile.internal` celowo **nie** ma fallbacku na `index.html`: to strona
statyczna, więc nieistniejący adres ma zwrócić 404, a nie stronę główną.

## Konwencje

- Nazwy klas, zmiennych i plików po polsku, spójnie z treścią strony.
- Style wyłącznie w `global.css` — bez frameworka, bez stylów w komponentach.
- Kolory przez zmienne CSS; motyw ciemny wynika z `prefers-color-scheme`.
- Szerokość kolumny tekstu (`--miara`) to 34rem. Nie zwiększać bez powodu —
  długość wiersza jest dobrana pod czytelność, a nagłówek pod nią wymierzony.