---
layout: post
title:  "Localized jekyll pages without plugin"
date:   2018-02-25 21:00:00
date_modified: 2018-04-18 12:00:00
excerpt: "How I added localization to my personal github pages"
image: /assets/img/wide/jekyll-logo-l10n.jpg
thumb: /assets/img/thumbs/jekyll-logo-l10n.jpg
tags: [github, gh-pages, jekyll, liquid, multilanguage, internationalization, i18n, localization, l10n, development]
categories: [posts, development]
comments: true
lang: en
ref: post-localized-jekyll-pages-without-plugin
---

<!-- MDTOC maxdepth:6 firsth1:2 numbering:1 flatten:0 bullets:0 updateOnSave:1 -->

1. [Introduction](#introduction)   
2. [Requirements](#requirements)   
3. [Homepage](#homepage)   
4. [Translations](#translations)   
5. [Further contents](#further-contents)   
6. [Navigation](#navigation)   
7. [Posts](#posts)   
8. [Tags](#tags)   
9. [Source code](#source-code)   

<!-- /MDTOC -->

## Introduction

Finally my personal pages with a fresh new look based on Andrew Banchich's [Editorial theme](https://html5up.net/editorial) are online.

Multi-language support was very important to me for the refactoring.
Even though I like to post in german as a native speaker, I wanted to have the opportunity to write articles in english or optionally translate some articles into english.
Already with the previous "Moon" theme variant I reviewed some Jekyll plugins for this, but found their usage too complicated or inflexible and so I built up my own concept without using plugins.
I used this concept again for the redesign of my pages and extended it.

## Requirements

I have the following requirements for multi-language support:

1. It should be possible to publish posts and normal pages in different languages (German and English).
1. The reader should be automatically forwarded to the content of his preferred language via homepage.
1. A change of language should be possible via links.
1. When selecting a language, only content in this language should be offered, i.e. content in different languages shouldn't be mixed.
1. For SEO optimization, content in different languages should also be represented by different URLs.
1. It should be possible to easily switch between content in one language and that in another (e. g. translated articles) (referencing among each other).

## Homepage

Let's go!

In the root directory of my github pages I have placed a markdown page `index.md` which Jekyll renders to an `index.html` using the given layout template `home.html`. I have now copied this `index.md` two times and named the copies according to my supported languages `de.md` and `en.md`. Jekyll renders these files to `/en/index.html` and `/en/index.html`, which gives me the basis for language-specific URLs.

OK, next step!

In order to be able to provide translation, I have to recognize in the layout templates which language is active on the respective page. I do this via a corresponding attribute in the header of the markdown files. I have called my language attribute short and sweet `lang`.
While `index.md` is the only file that doesn't get a `lang` attribute, all other markdown files are decorated with it, e.g.:

* en.md

``` markdown
---
layout: home
title: Home
lang: en
---
```

* de.md

``` markdown
---
layout: home
title: Hauptseite
lang: de
---
```

*For a successful implementation of multi-language support, a consistent separation of template and content (markdown files) is necessary, as content has to be maintained several times for each language in contrast to the design!*
{: .notice--info}

Now I can build translation in the template files via evaluation of the `lang` attribute. This mainly concerns headings, navigation entries and other recurring information (depending on the theme). E.g. my `home.html` layout integrates the views (_includes) `head.html`, `banner.html` and `sidebar.html`, in which I do such translations.
In `head.html`, which is the first one to be included in all pages, I've placed a small liquid construct, which always stores a valid language id in a variable named `navlang` and, if necessary, falls back to the default language set in `_config.yml` if `page.lang` was not set (navlang - "Navigation Language" because I originally introduced the variable for navigation links).

* head.html

``` liquid
{% raw %}{% if page.lang %}
	{% assign navlang = page.lang %}
{% else %}
	{% assign navlang = site.locale %}
{% endif %}{% endraw %}
```

So for the `index.md`, where the `lang` attribute is not set, I have a valid language setting and can use the fact that I haven't set `lang` here to fulfill another requirement - the automatic forwarding to the content with the user-preferred language. For this purpose I have included the following script in the template `home.html`:

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

It means - if `page.lang` is not available, a forwarding script is embedded, which will be used for german-speaking users to forward to the german pages, for all others to the english content.

*Please note that some browsers return a 2-digit code, others return the complete language ISO code!*
{: .notice--warning}


## Translations

For strings in individual languages I've created a yaml file called `messages.yml` in the subdirectory `_data` - following the idea of [Tuan Anh](https://tuananh.org/2014/08/13/localization-with-jekyll/)'s blog entry. It contains all strings to be localized - structured by language:

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

In the templates, I can then access the localized strings as in the following example:

``` liquid
{{ "{{ site.data.messages.locales[navlang].home " }}}}
```

This entry returns `Home` for the english representation and `Startseite` for the german language.

## Further contents

In order to remain consistent in language segregation, all other displayable pages must also be duplicated per language in addition to the "main page". Here you have to decide now whether you prefer the language separation to the content or vice versa. Even if it appears more consistent in the URL, split by language first and then by content, such as `/de/about` or `/en/tags`, I prefer content clustering - as usually in software development - so that e.g. `/about` is now split into `/about/de` and `/about/en`. I do this by copying my original `/about.md` file into a `/about/en.md` and a `/about/de.md` and then setting the corresponding `lang` tag and other attributes. Now I can also adapt the content to the respective language.

I also use the same procedure for `posts`, `tags` and `impressum`.

| Before          | After            | Template       | Rendered                 |
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

Of course, in the templates (layouts/Includes) you have to make sure that references that link to language-relevant resources process the `navlang` variable. For example, almost all my layouts include the file `header.html`, in which page title is set at the beginning with a link to homepage:

``` html
{% raw %}<a href="{{ site.url }}/{{ navlang }}/" class="logo"><strong>{{ site.title }}</strong></a>{% endraw %}
```

Likewise, I am going with other files, e.g. with `banner.html`:

``` html
{% raw %}<ul class="actions">
  <li><a href="{{ site.url }}/about/{{ navlang }}/" class="button big">{{ site.data.messages.locales[navlang].btn_more }}</a></li>
</ul>{% endraw %}
```

The central role of page navigation in my pages is taken over by a sidebar, which is integrated in all templates. I control its content via a yaml file named `navigation.yml` in the folder `_data`. There you can find menu titles and related links sorted by language and content, which are processed in template `sidebar.html`.

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

There is a special feature when changing the language. In one of my requirements, I had stated that you can easily switch between content that exists in one language and another. For example, if I am in the list of all german posts and now switch to english by menu entry, then the list of all english posts should be displayed and not the english homepage. The same applies to `tags`, `about`, `impressum` as well as to posts that I have translated into another language.

For a more generic approach, I've added another meta-information to the markdown files - a "language referer" called `ref`. For example, the list of all posts in all offered languages has the same referer `post-list`:

* File `/posts/en.md`:

``` markdown
---
layout: post-list
title: All posts
lang: en
ref: post-list
---
```

* File `/posts/de.md`:

``` markdown
---
layout: post-list
title: Alle Artikel
lang: de
ref: post-list
---
```

Posts can also contain such a unique identifier. For example this post has the referer `post-localized-jekyll-pages-without-plugin`, which I also use for the translation:

* English:

``` markdown
---
layout: post
title:  "Localized jekyll pages without plugin"
lang: en
ref: post-localized-jekyll-pages-without-plugin
---
```

* German:

``` markdown
---
layout: post
title:  "Mehrsprachige Seiten mit Jekyll ohne Plugin"
lang: de
ref: post-localized-jekyll-pages-without-plugin
---
```

For untranslated content, this attribute simply remains empty.

Now I can decide in the template `sidebar.html` whether I want to redirect to the translated content or to the homepage when switching the language.

To do this, I first determine all pages and posts and then check whether there is content in the target language with the same referer.
Otherwise I just enter the homepage in the other language.

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

## Posts

As mentioned before, my posts also support a language-dependent representation by carrying a `lang` attribute and optionally a `ref` attribute. But now it should be about how I list only the posts belonging to the language in the post and tag lists.

For the post lists, this is relatively easy to do by filtering all posts to see if they match the current language setting.

* File `post-list.html`:

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

For the tag list I initially had just one page which in the header lists the tags of all posts available in the respective language and below it shows a list of the associated posts grouped by these tags. However, this list quickly became large and confusing with an increasing number of tags, even for a few articles.

Now I have developed the tag list pages so that they only display a tag cloud and each tag entry forwards to a language-dependent page, which then lists all articles that are assigned to the tag and the language. I took these ideas of Jo Vandeginste from his article [Add a tag cloud to my Jekyll site](http://jovandeginste.github.io/2016/05/04/add-a-tag-cloud-to-my-jekyll-site.html) and developed them further for multi-language usage.

The file `collecttags.html` - included in `tag-list.html` - aggregates the tags:

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

The calculation of the factor for sizing tags in `tag-list.html`:

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

Display of the tag cloud in `tag-list.html`:

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

This procedure requires that markdown files with the same name are created for all tags in the respective language. For example, I have the files `/tags/de/jekyll.md` and `/tags/en/jekyll.md` for the tag `jekyll`, but only the file `/tags/de/entwicklung.md` for the german tag `entwicklung`.

Content of the file `/tags/en/jekyll.md` as an example:

``` markdown
---
layout: tag-page
title: "Tagged as: jekyll"
tag: jekyll
lang: en
ref: tag-jekyll
---
```

Excerpt from the corresponding layout template `tag-page.html`:

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

Yes, I know - creating such a page per tag and language is an additional step, but quickly done "by hand". It is even easier to automate this work with a script:

Content of the `_gentags.rb` file:

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

_**Hint:**\\
The script requires the presence of language-dependent directories - e.g. `tags/en` or `tags/de`!_
{: .notice--warning}

## Source code

All practices presented here and the source code can be found in my git repository for these Jekyll pages on Github:

[https://github.com/mcpride/mcpride.github.io](https://github.com/mcpride/mcpride.github.io)

I hope I can give you some suggestions and solutions - good luck in adapting and improving!

I always like to read constructive comments and suggestions ;-)
