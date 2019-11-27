# Table Of Contents
1. [Motivation](#motivation)
1. [Definitions](#definitions)
1. [Use Cases](#usecases)
   1. [Article Suggestion Use Case](#suggestusecase)
   1. [Search Use Case](#searchusecase)
1. [Problem Statement](#problemstatement)
1. [Proposal](#proposal)
   1. [User agent Enhacements](#enhancements)
   1. [Feature Detection](#detection)
   1. [Summary](#summary)
1. [Alternatives](#alternatives)
1. [Previous Discussion](#discussion)
1. [Security](#security)
1. [Privacy](#privacy)
1. [Considerations outside the scope](#scope)

# API for hinting translation to native UA component
An explainer to define the ability for a page author to hint to the UA's translation engine that a href should be translated if followed.

## Motivation <a name="motivation"></a>

Irrespective of the language that the user has set in their browser, the internet predominantly tends to be presented in a handful of languages.
[W3Techs](https://w3techs.com/technologies/overview/content_language/all) attributes English as being the most popular, and the top 8
languages capture 83.7% of the pages online.

When you contrast this against the popularity of native languages spoken, it’s clear that certain language speakers are under-served. Take
for example that [0.05%](https://w3techs.com/technologies/details/cl-hi-/all/all) of sites are presented in Hindi yet just under
[5%](https://en.wikipedia.org/wiki/List_of_languages_by_number_of_native_speakers) of the world’s population speaks Hindi.

Two use cases are presented below:
- [Article suggestion](#suggestusecase)
- [Search](#searchusecase)

## Definitions <a name="definitions"></a>

This document makes reference to two types of translation services:

### Server Based
<a name="server-side"></a>Server-side translation service is where the service is entirely implemented on a server. The page you wish to translate
is provided as a parameter to the server and the page returned is from the translation server's origin. Think of this
as typing `https://translate.google.com/translate?sl=en&tl=hi&u=https%3A%2F%2Fen.wikipedia.org` into the location bar
of a browser. The page (and storage) all appears from `translate.google.com` but the content inside is actually from
`wikipedia.org`.

### Client Based
<a name="client-side"></a>Client-side translation services are implemented in the browser and the origin of the page is actually the origin
of the original content. Coookies, storage all continue to work. The translation service (User Agent) selects text out
of the DOM and decides to translate certain parts modifying the DOM once a translation of a piece has been
encountered. Think of this as a piecemeal translation based on the presented content to the user. This model is
more powerful since a fair amount of documents are generated from script. (ie. web components).

Client based translation services may be either offline or online. Offline services use translation dictionaries available
to the user agent and the translation will occur without communication to a remote endpoint. Online services use a API
to a server to request translation of a piece of text. A User Agent may make multiple calls to translate the individual
pieces of the DOM.

## Use Cases  <a name="usecases"></a>

## Article Suggestion Use Case <a name="suggestusecase"></a>

Facebook routinely surfaces web page links users might be interested in
exploring. It could take advantage of a translation hint if it had
a relationship with the user and an article that might be a good
match for the user but in the wrong language. The website could use
the hint to suggest to the user agent that it would be a good idea to
translate the linked article.

## Search Use Case <a name="searchusecase"></a>

Considering this let’s look at the amount of information provided in [Hindi for W3C on wikipedia](https://hi.wikipedia.org/wiki/%E0%A4%B5%E0%A4%BF%E0%A4%B6%E0%A5%8D%E0%A4%B5_%E0%A4%B5%E0%A5%8D%E0%A4%AF%E0%A4%BE%E0%A4%AA%E0%A5%80_%E0%A4%B5%E0%A5%87%E0%A4%AC_%E0%A4%B8%E0%A4%82%E0%A4%98).

![Wikipedia Hindi W3C](https://github.com/dtapuska/html-translate/raw/master/HindiWikipediaW3C.png "Hindi Wikipedia W3C")

Compare this with the results of the [English version](https://en.wikipedia.org/wiki/World_Wide_Web_Consortium):

![Wikipedia English W3C](https://github.com/dtapuska/html-translate/raw/master/WikipediaScreenshotW3C.png "English Wikipedia W3C")

The Hindi version contains 192 words whereas the English version is much richer containing 11 times more content at 2235 words.


Search Engines wish to return the best results which may be from other languages. Consider this
[live example](https://www.google.com/search?hl=id&gl=id&q=eksperimen+cairan+yang+menetes+selama+90+tahun) of an Indonesian
user searching for results about long term experiments of viscous fluids. The first returned search result points to
https://translate.google.com/translate?u=https://en.wikipedia.org/wiki/Pitch_drop_experiment&hl=id&sl=en&tl=id&client=srp

![Search Results](https://github.com/dtapuska/html-translate/raw/master/SearchResults.png "Search Results")

This result links to a server-based translation service, which is problematic because they break the origin model of the web,
and produce poor rendering because they execute work on the static content of the page without javascript enhancement. It also
links to a specific translation service, denying choice to the user or user agent. Although this approach surfaces content to the
user that may not have been accessible before, it presents a page in low fidelity. [Client-side](#client-side) translation services are much
more effective as they can iterate the DOM and translate text nodes directly.

Notice also that in this example, the query was written in Indonesian, not English. This is a strong hint to the Search Engine to ask
for translation of the results into Indonesian. For other queries, the same user may type the search in English, and expect results in
that language.

![Translated Search Results](https://github.com/dtapuska/html-translate/raw/master/TranslatedResults.png "Search Results")

## Problem Statement <a name="problemstatement"></a>

It is currently impossible to reliably invoke the request for [client-side](#client-side) translation for a specific language. There are
situations where the currently viewed website author wishes a specific target language for the destination of a linked page (via an anchor).

Website authors currently can redirect pages to online server translation engines (such as translate.bing.com or translate.google.com).
But approaches such as these suffer from three main issues.
* **Ambiguation** of the origin. Sites all appear to come from a single host, this seems to further **centralize** search engines to the web.
* **Removal of attribution** and **trust**. As we have seen with projects such as AMP cache, taking attribution away from the website **erodes** user trust and attribution.
* **Poor rendering**. A UX study conducted showed that over 42% of sites sent through a server side translation proxy had **issues** with the rendering.

Another issue is that website authors wish to be able to automatically provide a translation. Google had a
[translation widget](https://en.support.wordpress.com/google-translate-widget/) that could be installed on websites that allowed server side translation.

[Client-side](#client-side) translation addresses the above issues, however it cannot be reliably invoked. In addition to it being difficult to obtain proper
language settings from users, the User Agent may not have enough information about the user intent, and a site may know much better than a User Agent
whether a particular link presented to the user has a strong intent to be translated (see example below). In order to improve fidelity of browsing for
a non-top language monolingual user, providing a way of triggering client side language translation is needed.

**Example**: Input occurs in many other forms than keyboard. An assistant/search site where the user uses voice input "please load this page and translate into
my language" cannot be serviced by the User Agent at all because it is unaware of the context of a site interaction.


## Proposal - HrefTranslate attribute  <a name="proposal"></a>

Define a new attribute “hrefTranslate” that can be used by the User Agent to know that the website wishes to present the
linked page in a desired language. It is a hint only, the User Agent will need to decide whether to invoke the client
translation service or not. The client-base translation service can be offline or online, depending
on which is available. For untrusted sites the User Agent should have an affirmitive confirmation to the user that
translation should be performed because the page may contain sensitive user data. If the User Agent trusts that site
(perhaps by the user previously agreeing to the prompts), it may invoke a translation service automatically on the results.

For pages that are presented as translated to the user the user agent should maintain the desired target language and apply
that to subsequent navigations if the hrefTranslate attribute is not present.

Example:

```HTML
<a href=”https://example.com” hrefTranslate=”hi”>उदाहरण</a>
```

Example with hreflang:

```HTML
<a href=”https://example.com” hrefLang=”en” hrefTranslate=”hi”>उदाहरण</a>

```
[Draft HTML Specification](https://whatpr.org/html/3870/links.html#attr-hyperlink-hreftranslate) has been proposed in [pull request 3870](https://github.com/whatwg/html/pull/3870).

### Possible enhancements for User Agent <a name="enhancements"></a>

The user agent may also wish to keep track of how often a user clicks on links
that have the same `hrefTranslate` and if the user continues to interact with
those translated sites. This information may provide strong inference that
the user agent may want to suggest to the user to adding the target
language to the list of languages the browser supports that is sent in the
Accept-Language.

### Feature Detection and Fallback <a name="detection"></a>

The hrefTranslate should not be supported in browsers that do not have
client side translation or it has been disabled.

Clients should be able to feature detect if the attribute is supported
on the anchor prototype via:

```Javascript
HTMLAnchorElement.prototype.hasOwnProperty('hrefTranslate');
```

Clients will need to dynamically decide via (javascript) how to present
the link to the user. Consider the example below presenting the English
Wikipedia site in German.

```HTML

<a id="anchor1" hrefTranslate="de" href="https://en.wikipedia.org">Wikipedia</a>

<script>
  // Detect if hrefTranslate is supported or not.
  if (!HTMLAnchorElement.prototype.hasOwnProperty('hrefTranslate')) {

    // It is not supported we might want to redirect the user
    // to a server based translation service. We can always read
    // custom properties via getAttribute so grab the hrefTranslate
    // attribute in browsers that don't support it and place it as
    // the target language on a server translation link.

    var anchor = document.getElementById('anchor1');
    anchor.href = "http://translate.google.com/translate?tl=" + anchor.getAttribute('hrefTranslate') + '&u=' + encodeURI(anchor.href);
  }
</script>

```

### Summary  <a name="summary"></a>

Pros:
* **Predictable** outcomes
* **Maintains attribution** and **avoids ambiguating** the origin
* Supports **decentralizing** the web by decoupling with search engines
* Browsers could be written such that they could **choose** its own translation service
* Fallback to [server-side](#server-side) translation if client side isn’t supported via [feature detection](#detection)
* It is only a hint. The final decision of **how to translate** rests with the User Agent. For example, Chrome intends to have user
permission prompts when hrefTranslate is encountered from non-trusted sites.

Cons:
* Can be laborious for manually written content


## Previous Discussion <a name="discussion"></a>

HTML Spec [issue 2945](https://github.com/whatwg/html/issues/2945) presents this topic. A number of solutions were discussed and
guidance was indicated that focus should be placed on ensuring the user’s language is correctly set.

Having the user provide correct language settings is difficult, because the users don’t understand the implications very well.
Research has shown that in some regions users will set their language to English because it is a status symbol, or because
software tends to work better in English. This explainer is an attempt to clarify the problem from prior discussion because
the problem exists even if the user’s language is set correctly. For many such users, there is a large deficit of content and
content quality in their language

## Alternate Solutions Considered  <a name="alternatives"></a>

### Lang attribute
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

# Security Considerations  <a name="security"></a>

User Agents should only use the attribute as a hint to invoke the translation service. The user agent might apply policies to not to invoke the
translation service. (eg. incognito mode, certain domains). Some browsers already support auto-translation of pages when navigating to a
page in a different language so this is just a modification of that flow.

# Privacy Considerations  <a name="privacy"></a>

No additional privacy concerns beyond those that should already be implemented
for a user agent supporting a client side translation service.

While it may not be obvious to a vendor implementing this attribute,
the client side translation service has the following properties:

## Informed user

The user should be informed about the implications of using a translation
service. Vendors should provide a policy explainer document so users can
understand the implications of translating a page. For an example see
[Chrome's Privacy Whitepaper](https://www.google.com/chrome/privacy/whitepaper.html).

## Affirmitive confirmation

Depending on the implementation of the client-based translation service it may
be either offline or online. For online user agents must follow good security
and privacy practices to protect the user.

Concerns that should be considered are:
- Retransmission of sensitive data
- History tracking

Since the page may contain sensitive data user agents should have a confirmation
prompt to the user before performing the translation. If the user has automatically
translated the page based on a previous confirmation from the user it should make it
clear to the user that page was machine translated.

# Considerations outside the scope  <a name="scope"></a>

While we believe that this is a positive move forward for the web to address concerns
of users in emerging markets we are mindful of a few concerns:

## Walled gardens and barriers for entry

The web should be open and have no barriers for entry. There are concerns raised by the
[W3C Tag](https://github.com/w3ctag/design-reviews/issues/301#issuecomment-553180605)
that hints such as `hrefTranslate` add to the power of popular sites and entrench a
view of the walled gardens. While some web services may have a good understanding
of their user, the user agent should also use the repetition of these hints as a
strong signal for all websites. (See [enhancements](#enhancements))

## Detection of translated page

Client-based translation services leak additional information about the user because the
translation can be detected. This proposal which makes client-based translation service easier
to invoke may cause this issue to become more prevalant. This may not be tractable because
the ideal scenario (user agent has correct `Accept-Language`) also leaks information in
the request but it is worth highlighting.
