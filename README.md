# API for hinting translation to native UA component
An explainer to define ability to for a page author to hint to the UA's translation engine that a href should be translated if followed.

## Motivation

Irrespective of the language that the user has set in their browser the internet predominantly tends to be presented in a handful of languages.
[W3Techs](https://w3techs.com/technologies/overview/content_language/all) attributes English as being the most popular, and the top 8
languages capture 83.7% of the pages online.

When you contrast this against the popularity of native languages spoken, it’s clear that certain language speakers are diserviced. Take
for example that [0.05%](https://w3techs.com/technologies/details/cl-hi-/all/all) of sites are presented in Hindi yet just under
[5%](https://en.wikipedia.org/wiki/List_of_languages_by_number_of_native_speakers) of the world’s population speaks Hindi.

Search Engines may return results from other languages.  Consider this example:

![Multi Lang Search Results](https://github.com/dtapuska/html-translate/raw/master/search.png "Multi Lang Search Results")

The presented text in blue beside the livescience.com URL is “Translate this page” in Hindi and this anchor directs to a server side
translation service of the linked article.

Server based translation services are problematic because they operate on the page and don’t execute the JavaScript of the page.
Although this surfaces content to the user that may not have been accessible before, it presents a page in low fidelity.
Client side based translation services are much more effective as they can iterate the DOM and translate text nodes directly.

## Problem Statement
In order to improve fidelity of browsing for a non-top language monolingual user, providing a way of triggering client side
language translation is needed.

## Previous Discussion

HTML Spec [issue 2945](https://github.com/whatwg/html/issues/2945) presents this topic. A number of solutions were discussed and
guidance was indicated that focus should be placed on ensuring the user’s language is correctly set.

This is an attempt to clarify the problem from that discussion because it wasn’t clear that the problem exists even if the user’s
language is set correctly because of the disproportion of articles actually in the language of the user.

## Possible Solutions

### Solution 1 - Lang attribute
Use the [lang](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/lang) attribute of the href (note this is not the
[hreflang](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/a#attr-hreflang) attribute which describes the language of
the link) as a hint to trigger client side translation.

Pros:
* No spec changes

Cons:
* Kind of overloads the current use of an attribute
* Unpredictable outcomes because of the use of the lang attribute in the web today
* Lang attribute appears in approximately 0.21% of pages in HTTP Archive

Example Wikipedia uses:
``<li class="interlanguage-link interwiki-bar"><a href="https://bar.wikipedia.org/wiki/" title="Bairisch" lang="bar" hreflang="bar" class="interlanguage-link-target">Boarisch</a></li>``

This doesn’t necessarily mean that the user actually wants this URI translated.

### Solution 2 - Use a meta tag on the page
Use a [meta](https://html.spec.whatwg.org/multipage/semantics.html#the-meta-element) tag to indicate that links should be followed in a specific
language.

Pros:
* You don't have to specify it on every link

Cons:
* You can't control which links are translated and which aren't. The page may only wish to translate different origin links but not same
origin links. eg. Searching for a term might lead to multiple pages of search terms and you won't necessarily want to translate the second
page of the search results.

### Solution 3 - HrefTranslate attribute (preferred)
Define a new attribute “hrefTranslate” that can be used by the user agent to know that the website wishes to present the linked page in a desired language. 

Example:
``<a href=”https://example.com” hrefTranslate=”hi”>उदाहरण</a>``

Example with hreflang:
``<a href=”https://example.com” hrefLang=”en” hrefTranslate=”hi”>उदाहरण</a>``

Pros:
* Predictable outcomes

Cons:
* Can be laborious for manually written content

[Draft HTML Specification](https://whatpr.org/html/3870/links.html#attr-hyperlink-hreftranslate) has been proposed in [pull request 3870](https://github.com/whatwg/html/pull/3870).


# Security Considerations

User Agents should only use the attribute as a hint to invoke the translation service. The user agent might apply policies to not to invoke the
translation service. (eg. incognito mode, certain domains). Some browsers already support auto translation of pages when navigating to a page
in a different language so this is just a modification of that flow.

# Privacy Considerations

No additional privacy concerns beyond those that should already be implemented for a user agent supporting a client side translation service.
