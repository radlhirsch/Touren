---
layout: post
title:  "Syntax-Highlighting mit Jekyll"
date:   2018-03-06 09:00:00
excerpt: Github-Seiten unterstützen Source-Code-Syntax-Hervorhebung, womit man die Lesbarkeit von Code-Schnipseln in einem Blog oder einer Seite verbessern kann. Der Artikel erklärt, wie man diese aktiviert. 
image: /assets/img/wide/syntax-highlight.jpg
thumb: /assets/img/thumbs/syntax-highlight.jpg
tags: [github, gh-pages, jekyll, rouge, liquid, entwicklung]
categories: [posts, development]
comments: true
lang: de
ref: post-syntax-highlighting-with-jekyll
---

[Github-Seiten](https://pages.github.com/) unterstützen Source-Code-Syntax-Hervorhebung über das gem 
[Rouge](https://github.com/jneen/rouge), womit man die Lesbarkeit von Code-Schnipseln in einem Blog oder einer Seite verbessern kann.
[Rouge](https://github.com/jneen/rouge) ist seit Version 3 der Standard-Highlighter für [Jekyll](https://jekyllrb.com/). In folgenden Schritten kann man seiner Github/Jekyll-Seite Syntax-Highlighting mit [Rouge](https://github.com/jneen/rouge) hinzufügen.

## Schritt 1: Einbinden der gems Kramdown und Rouge

Falls noch nicht geschehen, lassen sich die Ruby-Gems Kramdown und Rouge mit einem Befehl lokal installieren (in Github sind sie standardmäßig aktiviert):

``` shell
gem install kramdown rouge
```

## Schritt 2: Konfigurieren

In der Konfigurationsdatei `_config.yml` sollten folgende Einstellungen vorgenommen werden:

``` yaml
markdown: kramdown
highlighter: rouge

kramdown:
  input: GFM
  syntax_highlighter: rouge
```

## Schritt 3: Bestimme den Style

Über das CLI-Tool `rougify` von `Rouge` kann man sich alle unterstützten Themes für die Syntax-Hervorhebung auflisten lassen:

``` shell
rougify help style
``` 

Ausgabe:

``` shell
>rougify help style
usage: rougify style [<theme-name>] [<options>]

Print CSS styles for the given theme.  Extra options are
passed to the theme.  Theme defaults to thankful_eyes.

options:
  --scope       (default: .highlight) a css selector to scope by

available themes:
  base16, base16.dark, base16.light, base16.monokai, 
  base16.monokai.dark, base16.monokai.light, base16.solarized, 
  base16.solarized.dark, base16.solarized.light, colorful, 
  github, gruvbox, gruvbox.dark, gruvbox.light, igorpro, 
  molokai, monokai, monokai.sublime, thankful_eyes, tulip
```

Für meine Seiten benutze ich das Theme `github`. 

Die passende Stylesheet-Datei `/assets/css/syntax.css` habe ich mir mit folgendem Befehl generieren lassen:

``` shell
rougify style github > assets/css/syntax.css
```

Diese Stylesheet-Datei muss noch in die Jekyll-Templates zentral eingebunden werden - bei meinen Seiten ist das die Template-Datei `/_includes/head.html`:

``` html
<head>
    ...
  <link rel="stylesheet" href="{{ 'assets/css/syntax.css' | relative_url }}" />
</head>

```

## Schritt 4: Markiere die Codeschnipsel mit dem gewünschten Highlighter

Um die Syntaxhervorhebung für die Codeschnipsel in Artikeln/Seiten zu aktivieren, muss man sie mit dem gewünschten Higlighter-/Lexer-Namen markieren. Diesen schreibt man in den Markdown-Dateien direkt neben den eröffnenden Marker für einen Codeblock - nachfolgend ein Beispiel für `html`:

    ``` html
    <html>
      <head>
		<title>
		</title>
      </head>
      <body>
      </body>
    </html>
    ```

Mit folgendem Kommandozeilenbefehl kann man sich eine Liste der unterstützten Lexer ausgeben lassen:

``` shell
rougify list
```
