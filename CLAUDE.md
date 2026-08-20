# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## O projekcie

Strona osobista (portfolio, notatki techniczne, CV, informacje o korepetycjach)
zbudowana w Astro 7, renderowana statycznie. Treść i identyfikatory są po
polsku — patrz sekcja Konwencje.

## Komendy

```bash
npm install
npm run dev          # serwer deweloperski, http://localhost:4321
npm run build         # build produkcyjny do dist/
npm run preview       # podgląd builda lokalnie
npm run check          # astro check — kontrola typów i odwołań
npm run cv             # kompiluje cv/cv.tex -> public/cv.pdf
```

Nie ma frameworka testowego w tym repo — `npm run check` jest jedynym
automatycznym sprawdzeniem poprawności.

## Architektura

- **Content collections** (`src/content.config.ts`): dwie kolekcje, `blog` i
  `projekty`, ładowane przez `glob()` loader z `src/content/{blog,projekty}/*.md`
  i walidowane schematem Zod. Błąd we frontmatterze wywala build, nie
  produkcję — to świadomy wybór. Pola blog: `title`, `opis`, `data`, `tagi`,
  `szkic`. Pola projekty: `title`, `opis`, `stack`, `repo`, `demo`,
  `kolejnosc`, `szkic`. `szkic: true` wyklucza wpis z listy i z RSS.
- **Routing** (`src/pages/`): plik = adres (standardowy routing Astro).
  Dynamiczne trasy `[...slug].astro` dla blogu i projektów.
- **Layouty** (`src/layouts/`): `Base.astro` to szkielet strony (head, nav,
  stopka, meta/OG tagi) — wszystkie strony go używają. `Wpis.astro` to układ
  pojedynczego wpisu blogowego/projektu, osadzony w `Base.astro`.
- **Matematyka w Markdown**: Astro 7 domyślnie renderuje Markdown procesorem
  `satteri` (Rust), który przyjmuje `mdastPlugins`/`hastPlugins`, ale **nie
  uruchamia wtyczek remark/rehype**. Dlatego `astro.config.mjs` jawnie
  przełącza na pipeline `unified` z `remark-math` + `rehype-katex`. Jeśli
  wzory `$...$`/`$$...$$` przestaną się renderować, to pierwsze miejsce do
  sprawdzenia.
- **CV**: `public/cv.pdf` jest commitowany do repo (obraz Dockera nie ma
  dystrybucji LaTeX-a). Po każdej zmianie `cv/cv.tex` trzeba ręcznie
  uruchomić `npm run cv` i zacommitować `public/cv.pdf` razem z `cv/cv.tex`
  — inaczej źródło i opublikowany PDF rozjadą się po cichu. Skrypt
  (`scripts/build-cv.mjs`) próbuje najpierw `latexmk`, potem `pdflatex`
  (dwa przebiegi). Wymaga MiKTeX-a (Windows) lub TeX Live (Linux/macOS).
- **Wdrożenie**: build dwustopniowy (`Dockerfile`) — Node buduje stronę,
  wynik trafia do obrazu z Caddym (`Caddyfile.internal`), który serwuje
  wyłącznie pliki statyczne po HTTP w sieci wewnętrznej `edge` (alias `site`).
  TLS i routing zewnętrzny obsługuje osobny reverse proxy z repo panelu
  ([korepetycje](https://github.com/Kamilox007/korepetycje)), z którym ten
  kontener dzieli sieć Docker `edge`. Strona musi wystartować przed panelem.
  `Caddyfile.internal` celowo nie ma fallbacku na `index.html` — nieistniejący
  adres ma zwrócić 404.

## Konwencje

- Nazwy klas CSS, zmiennych i plików — po polsku, spójnie z treścią strony.
- Style wyłącznie w `src/styles/global.css` — bez frameworka CSS, bez stylów
  lokalnych w komponentach `.astro`.
- Kolory przez zmienne CSS; motyw ciemny wynika z `prefers-color-scheme`
  (nie z atrybutu/klasy przełączanej JS-em).
- Szerokość kolumny tekstu (`--miara`) to `34rem` — dobrana pod czytelność,
  nagłówek jest pod nią wymierzony. Nie zwiększać bez powodu.
- Nowy wpis blogowy/projektowy to po prostu plik `.md` w odpowiednim
  katalogu `src/content/`; nazwa pliku staje się częścią adresu.
