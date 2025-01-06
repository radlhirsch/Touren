---
layout: post
title:  "Mehrsprachige Seiten mit Jekyll ohne Plugin"
date:   2018-02-25 21:00:00
date_modified: 2018-04-18 12:00:00
excerpt: "Wie ich Mehrsprachigkeit bei meinen Github-Seiten umgesetzt habe."
image: /assets/img/wide/jekyll-logo-l10n.jpg
thumb: /assets/img/thumbs/jekyll-logo-l10n.jpg
tags: [github, gh-pages, jekyll, liquid, mehrsprachig, internationalisierung, i18n, lokalisierung, l10n, entwicklung]
categories: [posts, development]
comments: true
lang: de
ref: post-localized-jekyll-pages-without-plugin
---

<!-- MDTOC maxdepth:6 firsth1:2 numbering:1 flatten:0 bullets:0 updateOnSave:1 -->

1. [Einleitung](#einleitung)   
2. [Anforderungen](#anforderungen)   
3. [Startseite](#startseite)   
4. [Übersetzungen](#übersetzungen)   
5. [Weitere Inhalte](#weitere-inhalte)   
6. [Navigation](#navigation)   
7. [Artikel (Posts)](#artikel-posts)   
8. [Tags](#tags)   
9. [Quellcode](#quellcode)   

<!-- /MDTOC -->

## Einleitung

Endlich sind meine persönlichen Seiten mit neuem frischem Look basierend auf Andrew Banchich's [Editorial theme](https://html5up.net/editorial) online.

Bei der Umgestaltung war mir Mehrsprachigkeit sehr wichtig.
Auch wenn ich natürlich gerne in meiner deutschen Muttersprache poste, so wollte ich mir doch die Möglichkeit erhalten, auch englischsprachige Artikel zu verfassen bzw. optional einige Artikel ins Englische zu übersetzen.
Schon bei der vorherigen "Moon"-Theme-Variante schaute ich mir dazu einige Jekyll-Plugins an, fand deren Anwendung aber zu kompliziert oder unflexibel und so baute ich mir ein eigenes Konzept ohne Einsatz von Plugins zusammen.
Dieses Konzept wendete ich nun wieder bei der Neugestaltung meiner Seiten an und baute es aus.

## Anforderungen

Folgende Requirements habe ich für die Mehrsprachigkeit aufgestellt:

1. Es soll möglich sein, Posts und normale Seiten in verschiedenen Sprachen (deutsch und englisch) anzubieten.
1. Der Leser soll über die Startseite automatisch auf den Inhalt seiner bevorzugten Sprache weitergeleitet werden.
1. Ein Wechsel der Sprache soll über Verweise möglich sein.
1. Bei Auswahl einer Sprache sollen nur Inhalte in dieser Sprache angeboten werden, d.h. Inhalte in unterschiedlichen Sprachen sollen nicht vermischt werden.
1. Zur SEO-Optimierung sollen Inhalte in verschiedenen Sprachen auch durch unterschiedliche URLs repräsentiert werden.
1. Zwischen Inhalten, die sowohl in der einen als auch in einer anderen Sprache vorliegen (z.B. übersetzte Artikel), soll leicht umgeschaltet werden können (Referenzierung untereinander).

## Startseite

Los geht's!

Im Wurzelverzeichnis meiner Github-Seiten habe ich eine Markdown-Seite `index.md` plaziert, die Jekyll unter Zuhilfenahme des angegebenen Layout-Templates `home.html` zu einer `index.html` rendert. Diese `index.md` habe ich nun 2-mal kopiert und die Kopien gemäß meiner unterstützten Sprachen `de.md` und `en.md` benannt. Jekyll rendert diese Dateien nun zu `/de/index.html` und `/en/index.html`, womit ich nun die Basis für sprachspezifische URL's habe.

OK, nächster Schritt!

Um nun Übersetzung anbieten zu können, muss ich in den Layout-Templates erkennen können, welche Sprache auf der jeweiligen Seite aktiv ist. Das erledige ich über einen entprechendes Attribut im Header in den Markdown-Dateien. Ich habe mein Sprach-Attribut kurz und knackig `lang` genannt.
Während die `index.md` als einzige Datei kein `lang`-Attribut bekommt, werden alle anderen Markdown-Dateien damit ausgestattet, z.B.:

* de.md

``` markdown
---
layout: home
title: Hauptseite
lang: de
---
```

* en.md

``` markdown
---
layout: home
title: Home
lang: en
---
```

*Für eine erfolgreiche Umsetzung der Mehrsprachigkeit ist eine konsequente Trennung von Template und Inhalt (Markdown-Dateien) notwendig, da Inhalte im Gegensatz zur Gestaltung je Sprache mehrfach zu pflegen sind!*
{: .notice--info}

Nun kann ich über die Auswertung des `lang`-Attributs Übersetzung in den Templates einbauen. Das betrifft im wesentlichen Überschriften, Navigationseinträge und andere wiederkehrende Informationen (abhängig vom Theme). Mein `home.html`-Layout bindet z.B. die Views (_includes) `head.html`, `banner.html` oder `sidebar.html` ein, in denen ich solche Übersetzungen vornehme.
In `head.html`, welches als erstes in alle Seiten eingebunden wird habe ich noch ein kleines Liquid-Konstrukt platziert, welches mir immer eine gültige Sprache in einer Variable `navlang` speichert und notfalls auf die in der `_config.yml` eingestellte Standardsprache zurückfällt, falls `page.lang` nicht gesetzt wurde (navlang - "Navigation language", weil ich die Variable ursprünglich für die Navigations-Verweise eingeführt hatte).

* head.html

``` liquid
{% raw %}{% if page.lang %}
	{% assign navlang = page.lang %}
{% else %}
	{% assign navlang = site.locale %}
{% endif %}{% endraw %}
```
Somit habe ich auch für die `index.md`, bei der das `lang`-Attribut nicht gesetzt ist, eine gültige Spracheinstellung und kann den Umstand, dass ich hier `lang` nicht gesetzt habe, geschickt für die Erfüllung einer weiteren Anforderung nutzen - die automatische Weiterleitung auf den Inhalt mit der vom Benutzer bevorzugten Sprache. Dazu habe ich in das `home.html`-Template folgendes eingebaut:

``` html
{% raw %}{% unless page.lang %}
<script type="text/javascript">
    $( document ).ready(function(){
        var userLang = navigator.language || navigator.userLanguage;
        if ((userLang == "de") || (userLang == "de-DE")) {
            window.location.href = "{{ site.url }}/de/"
        }
        else {
            window.location.href = "{{ site.url }}/en/"
        }
    });
</script>
{% endunless %}{% endraw %}
```

Wenn also `page.lang` nicht vorhanden ist, wird ein Weiterleitungsskript eingebettet, welches für
deutschsprachige Benutzer auf die deutschen Seiten, für alle anderen auf die englischsprachige Repräsentation weiterleitet.

*Zu beachten ist dabei, dass einige Browser einen 2-stelligen, anderen jedoch den vollständigen Language-ISO-Code zurückgeben!*
{: .notice--warning}


## Übersetzungen

Für Strings in einzelnen Sprachen habe ich mir - der Idee von [Tuan Anh](https://tuananh.org/2014/08/13/localization-with-jekyll/)'s Blogeintrag folgend - eine Yaml-Datei namens `messages.yml` im `_data`-Unterverzeichnis erstellt. Darin werden alle zu lokalisierenden Strings - gegliedert nach Sprache - eingetragen:

``` yaml
locales:
  # English translation
  # -------------------
  en: &DEFAULT_EN
    about: "About"
    site_description: "My personal pages."
    btn_more: "More..."
    posts: "Blog"
    pages: "Pages"
    projects: "Projects"
    tags: "Tags"
    home: "Home"
    languages: "Languages"
    lang_name: "English"
    post_title: "Read this post in English..."
    page_title: "Read this page in English..."

  en_US:
    <<: *DEFAULT_EN     # use English translation for en_US
  en_UK:
    <<: *DEFAULT_EN     # use English translation for en_UK

  # German translation
  # -------------------
  de: &DEFAULT_DE
    <<: *DEFAULT_EN     # load English values as default
    about: "&Uuml;ber"
    site_description: "Meine pers&ouml;nlichen Seiten."
    btn_more: "Mehr..."
    posts: "Blog"
    pages: "Seiten"
    projects: "Projekte"
    home: "Startseite"
    languages: "Sprachen"
    lang_name: "Deutsch"
    post_title: "Diesen Artikel auf Deutsch lesen..."
    page_title: "Diese Seite auf Deutsch lesen..."
  de_DE:
    <<: *DEFAULT_DE     # use German translation for de_DE
```

In den Templates kann ich dann wie im nachfolgenden Beispiel auf die lokalisierten Strings zugreifen:

``` liquid
{{ "{{ site.data.messages.locales[navlang].home " }}}}
```

Dieser Eintrag gibt für die englische Repräsentation `Home` und für die deutsche `Startseite` zurück.

## Weitere Inhalte

Um in der sprachlichen Abgrenzung konsistent zu bleiben, müssen neben der "Hauptseite" auch alle sonstigen darstellbaren Seiten je Sprache dupliziert werden. Hier muss man sich nun entscheiden, ob man die sprachliche der inhaltlichen Abgrenzung vorzieht oder umgekehrt. Auch wenn es in der URL konsistenter erscheint, zunächst nach Sprache zu splitten und dann erst nach Inhalt, wie z.B. `/de/about` oder `/en/tags`, so ziehe ich eine inhaltliche Clusterung - wie meist bei der Softwareentwicklung - vor, sodas z.B. `/about` nun in `/about/de` und `/about/en` gesplittet wird. Das erreiche ich, indem ich meine ursprüngliche `/about.md`-Datei in eine `/about/de.md` und eine `/about/en.md` kopiere und danach die entprechenden `lang`-Tag sowie weitere Attribute setze. Nun kann ich auch den Inhalt der jeweiligen Sprache anpassen.

Die gleiche Vorgehensweise wende ich auch bei `posts`, `tags` sowie `impressum` an.

| Vorher          | Nachher          | Template       | Gerendert                |
|-----------------|------------------|----------------|--------------------------|
| index.html      | /index.md        | home.html      | /index.html              |
|                 | /de.md           |                | /de/index.html           |
|                 | /en.md           |                | /en/index.html           |
|                 |                  |                |                          |
| about.md        | /about/de.md     | page.html      | /about/de/index.html     |
|                 | /about/en.md     |                | /about/en/index.html     |
|                 |                  |                |                          |
| posts.html      | /posts/de.md     | post-list.html | /posts/de/index.html     |
|                 | /posts/en.md     |                | /posts/en/index.html     |
|                 |                  |                |                          |
| tags.html       | /tags/de.md      | tag-list.html  | /tags/de/index.html      |
|                 | /tags/en.md      |                | /tags/en/index.html      |
|                 |                  |                |                          |
| impressum.md    | /impressum/de.md | page.html      | /impressum/de/index.html |
|                 | /impressum/en.md |                | /impressum/en/index.html |

## Navigation

Natürlich ist in den Templates (layouts/Includes) nun darauf zu achten, dass Verweise, die zu sprachlich relevanten Ressourcen führen, die `navlang`-Variable verarbeiten. So binden z.B. fast alle meine Layouts die Datei `header.html` ein, in der gleich zu Beginn der Seitentitel mit einem Link zur Startseite gesetzt wird:

``` html
{% raw %}<a href="{{ site.url }}/{{ navlang }}/" class="logo"><strong>{{ site.title }}</strong></a>{% endraw %}
```

Ebenso verfahre ich mit anderen Dateien, z.B. mit `banner.html`:

``` html
{% raw %}<ul class="actions">
  <li><a href="{{ site.url }}/about/{{ navlang }}/" class="button big">{{ site.data.messages.locales[navlang].btn_more }}</a></li>
</ul>{% endraw %}
```

Die zentrale Rolle der Seitennavigation übernimmt in meinen Seiten eine Sidebar, die in allen Templates eingebunden ist. Ihren Inhalt steuere ich über eine im `_data`-Ordner liegende Yaml-Datei namens `navigation.yml`. Dort sind nach Sprache und Inhalt gegliederte Menütitel und zugehörige Links hinterlegt, die im Template `sidebar.html` verarbeitet werden.

``` yaml
locales:
  en:
    languages:
      - title: "Zu Deutsch wechseln"
        url: /de/
        lang: de
    posts:
      - title: "All posts"
        url: /posts/en/
    tags:
      - title: Tags of all posts
        url: /tags/en/
  de:
    languages:
      - title: "Switch to English"
        url: /en/
        lang: en
    posts:
      - title: "Alle Artikel"
        url: /posts/de/
    tags:
      - title: Tags aller Artikel
        url: /tags/de/
```

Eine Besonderheit gibt es beim Wechsel der Sprache. In einer Anforderung hatte ich formuliert, dass man zwischen Inhalten, die sowohl in der einen als auch in einer anderen Sprache vorliegen, leicht umschalten kann. Wenn ich also z.B. in der Liste aller deutschen Posts bin und nun per Menüeintrag zu Englisch wechsele, dann soll die Liste aller englischen Posts und nicht etwa die englische Startseite angezeigt werden. Gleiches gilt für `tags`, `about`, `impressum` sowie für Posts, die ich in eine andere Sprache übersetzt habe.

Für einen möglichst generischen Ansatz habe ich dazu in den Markdown-Dateien eine weitere Metainformation eingefügt - einen "Language referer" namens `ref`. So hat z.B. die Liste aller Posts in allen angebotenen Sprachen den gleichen Referer `post-list`:

* Datei `/posts/de.md`:

``` markdown
---
layout: post-list
title: Alle Artikel
lang: de
ref: post-list
---
```

* Datei `/posts/en.md`:

``` markdown
---
layout: post-list
title: All posts
lang: en
ref: post-list
---
```

Auch Posts können solch einen eindeutigen Identifizierer tragen. Dieser Artikel hat z.B. den Referer `post-localized-jekyll-pages-without-plugin`, den ich für eine potentielle Übersetzung ebenso verwende:

* Deutsch:

``` markdown
---
layout: post
title:  "Mehrsprachige Seiten mit Jekyll ohne Plugin"
lang: de
ref: post-localized-jekyll-pages-without-plugin
---
```

* Englisch:

``` markdown
---
layout: post
title:  "Localized jekyll pages without plugin"
lang: en
ref: post-localized-jekyll-pages-without-plugin
---
```

Bei nicht übersetzten Inhalten bleibt dieses Attribut einfach leer.

Nun kann ich im Template `sidebar.html` entscheiden, ob ich beim Umschalten der Sprache zum übersetzten Inhalt oder zur Startseite umleiten will.

Dazu ermittle ich zunächst alle Seiten und Posts und schaue dann, ob in der Zielsprache ein Inhalt mit dem gleichen Referer vorliegt.
Ansonsten trage ich einfach nur die Startseite in der anderen Sprache ein.

``` liquid
{% raw %}<ul>
  {% assign lang_ref_pages=site.pages | where:"ref", page.ref %}
  {% assign lang_ref_posts=site.posts | where:"ref", page.ref %}
  {% assign lang_ref_pages = lang_ref_pages | concat: lang_ref_posts  | sort: 'lang' %}
  {% for link in site.data.navigation.locales[navlang].languages %}
    {% assign ref_page=lang_ref_pages | where:"lang", link.lang | first %}
    {% if ref_page %}
      <li><a href="{{ site.url }}{{ ref_page.url }}">{{ link.title }}</a></li>
    {% else %}
      <li><a href="{{ site.url }}{{ link.url }}">{{ link.title }}</a></li>
    {% endif %}
  {% endfor %}
</ul>{% endraw %}
```

## Artikel (Posts)

Wie zuvor erwähnt, unterstützen auch meine Posts die sprachabhängige Darstellung, indem sie alle ein `lang`-Attribut sowie wahlweise ein `ref`-Attribut mitführen. Nun soll es aber darum gehen, wie ich in den Post- und Taglisten nur die zur Sprache zugehörigen Posts aufliste.

Bei den Artikel-Listen (post lists) ist das relativ einfach umgesetzt, indem ich alle Posts dahingehend filtere, ob sie mit der aktuellen Spracheinstellung übereinstimmen.

* Datei `post-list.html`:

``` html
{% raw %}<div class="posts">
    {% if page.lang %}
        {% assign posts=site.posts | where:"lang", page.lang %}
    {% else %}
        {% assign posts=site.posts %}
    {% endif %}

    {% for post in posts %}
    <article>
        ...
        <h3><a href="{{ site.url }}{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a></h3>
        ...
    </article>
    {% endfor %}
</div>{% endraw %}
```

## Tags

Für die Tag-Liste hatte ich zunächst nur eine Seite vorgesehen, die im Kopf die Tags aller in der jeweiligen Sprache verfügbaren Posts auflistet und darunter eine nach diesen Tags gruppierte Liste der zugehörigen Posts. Diese Liste wurde jedoch mit steigender Anzahl von Tags selbst bei wenigen Artikeln schnell groß und unübersichtlich.

Nun habe ich die Tag-Listen-Seiten dahingehend weiterentwickelt, dass diese nur eine Tag-Cloud anzeigt und jeder Tag-Eintrag zu einer sprachabhängigen eigenen Seite weiterleitet, welche dann alle Artikel auflistet, die dem Tag ud der Sprache zugeordnet sind. Dabei habe ich die Ideen von Jo Vandeginste aus seinem Artikel [Add a tag cloud to my Jekyll site](http://jovandeginste.github.io/2016/05/04/add-a-tag-cloud-to-my-jekyll-site.html) aufgegriffen und für mehrsprachige Verwendung weiterentwickelt.

Die in `tag-list.html` eingebundene Include-Datei `collecttags.html` aggregiert mir die Tags:

``` liquid
{% raw %}{% if page.lang %}
    {% assign sposts=site.posts | where:"lang", page.lang %}
{% else %}
    {% assign sposts=site.posts %}
{% endif %}
{% assign rawtags = "" %}
{% for post in sposts %}
    {% for tag in post.tags %}
        {% if rawtags == "" %}
            {% assign rawtags = tag %}
        {% else %}
            {% unless rawtags contains tag %}
                {% assign rawtags = rawtags | join:'|' | append:'|' | append:tag | split:'|' %}
            {% endunless %}
        {% endif %}
    {% endfor %}
{% endfor %}
{% assign stags=rawtags | sort %}{% endraw %}
```

Die Berechnng des Faktors für die Größendarstellung der Tags in `tag-list.html`:

``` liquid
{% raw %}{% assign asize = 0 %}
{% for stag in stags %}
    {% if page.lang %}
        {% assign ltags=site.tags[stag] | where:"lang", page.lang %}
    {% else %}
        {% assign ltags=site.tags[stag] %}
    {% endif %}
    {% assign asize = asize | plus: ltags.size %}
{% endfor %}{% endraw %}
```

Darstellung der Tag-Cloud in `tag-list.html`:

``` html
{% raw %}{% for stag in stags %}
    {% if page.lang %}
        {% assign ltags=site.tags[stag] | where:"lang", page.lang %}
    {% else %}
        {% assign ltags=site.tags[stag] %}
    {% endif %}
    {% assign rel_tag_size = ltags.size | times: 4.0 | divided_by: asize | plus: 0.75 %}
    <li>
        <a href="{{ site.url }}/tags/{{ navlang }}/{{ stag }}" style="text-decoration: none; border-bottom:none">
            <span style="white-space: nowrap; font-size: {{ rel_tag_size }}em; padding: 0.2em;">
                {{ stag }}
                <span>({{ ltags.size }})</span>
            </span>
        </a>
    </li>
{% endfor %}{% endraw %}
```

Diese Vorgehensweise bedingt, dass für alle Tags in der jeweiligen Sprache gleichnamige Markdown-Dateien angelegt werden. So gibt es für das Tag `jekyll` bei mir die Dateien `/tags/de/jekyll.md` und `/tags/en/jekyll.md`, beim deutschsprachigen Tag `entwicklung` hingegen nur die Datei `/tags/de/entwicklung.md`.

Inhalt der Datei `/tags/en/jekyll.md` als Beispiel:

``` markdown
---
layout: tag-page
title: "Tagged as: jekyll"  
tag: jekyll
lang: en
ref: tag-jekyll
---
```

Auszug aus dem zugehörigen Layout-Template `tag-page.html`:

``` html
{% raw %}{% assign tposts=site.tags[page.tag] | where:"lang", page.lang %}
...
{% for post in tposts %}
    <article>
      ...
      <h3><a href="{{ site.url }}{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a></h3>
      ...
    </article>
    {% endif %}
{% endfor %}
...{% endraw %}
```

Eine solche Seite pro Tag und Sprache anzulegen ist zwar ein zusätzlicher Schritt, "per Hand" aber schnell getan. Noch einfacher ist es, wenn man diese Arbeit mit einem Script automatisiert:

Inhalt der Datei `_gentags.rb`:

``` ruby
{% raw %}require 'yaml'

langs = []
Dir.glob(File.join('_posts','*.md')).each do |file|
	yaml_s = File.read(file).split(/^---$/)[1]
	yaml_h = YAML.load(yaml_s)
	lang = yaml_h['lang']
	if ((lang != nil) && (lang.is_a? String))
		langs += [lang]
	end
end
langs = langs.map{ |lang| lang.downcase }.uniq

messages = YAML.load_file("_data/messages.yml")

fcount = 0

langs.each do |lang|
	tags = []
	Dir.glob(File.join('_posts','*.md')).each do |file|
		yaml_s = File.read(file).split(/^---$/)[1]
		yaml_h = YAML.sload(yaml_s)
		if yaml_h['lang'] != nil && yaml_h['lang'] == lang
			tags_h = yaml_h['tags']
			if tags_h != nil
				tags += tags_h
			end
		end
	end
	tags.map{ |tag| tag.downcase if tag.is_a? String }.uniq.each do |tag|
		tag_file = File.join("tags/#{lang}", "#{tag}.md")
		puts "Writing file '#{tag_file}' for tag '#{tag}' in language '#{lang}'..."
		pretitle = messages['locales'][lang]['tagged_as']
		File.write tag_file, <<-EOF
---
layout: tag-page
title: "#{pretitle}: #{tag}"  
tag: #{tag}
lang: #{lang}
ref: tag-#{tag}
---
	EOF
	fcount = fcount + 1
	end
end

puts "#{fcount} files written!"{% endraw %}
```

_**Hinweis:**\\
Das Skript setzt das Vorhandensein der sprachabhängigen Verzeichnisse voraus - z.B. `tags/en` oder `tags/de`!_
{: .notice--warning}

## Quellcode

Alle hier vorgestellten Praktiken und den Quellcode findet ihr in meinem Git-Repository zu diesen Jekyll-Seiten auf Github:

[https://github.com/mcpride/mcpride.github.io](https://github.com/mcpride/mcpride.github.io)

Ich hoffe, ich kann kann hiermit ein paar Anregungen und Lösungsvorschläge geben - viel Erfolg beim Nach- und Bessermachen!

Konstruktive Kommentare und Anregungen lese ich immer gerne ;-)
