# Improper markup sanitization in popular software

_[Featuring](#affected-software): GitHub, GitLab, {TBA}, Redmine, Gogs, JetBrains, and others._

There are several common markup (mostly Markdown) sanitization issues outlined in this article, but I would be mostly speaking about the single, most common one.

_If you don't care about the technical stuff, just skip to the [screenshots](#github) and scroll down._

## Introduction

Most solutions that deal with social coding, issues and comments obviously allow some sort of formatting in those comments and in
other places (e.g., the README files in the repos).
Accepting untrusted content from user, processing the markup language, and producing sanitized safe HTML is a common usage scenario.
Allowing a subset of HTML markup is also a commonly used feature (this is even included in the Markdown spec).

### How it should be done

The only correct way to produce sanitized HTML on the output is to parse everything to a tree structure, make sure that that tree
is sanitized, then encode it to HTML. Sounds pretty simple, doesn't it?

A fast way to achieve that is to add a sanitizer at the very last step of the transformations, and define the ruleset for that
sanitizer — this way, no matter what else is broken in the transformation pipeline, it won't affect the safety of the produced
html. Refer to [this article](https://michelf.ca/blog/2010/markdown-and-xss/) for some explanations.

_In some cases, the sanitizer might not be needed, when the markup library builds a safe tree structure from scratch, ensuring that
no unsafe content is put into that tree. It's still better to apply a dedicated sanitization step afterwards, as I would trust
specialized sanitizers more than markup libraries that might miss some specific things._

So, what could go wrong?

### Ways to break things

 1. **Not having sanitization at all.**
    Check if your markup library sanitizes things (e.g. already utilizes a good sanitizer).
    If it doesn't (some treat this as not their problems, and they are correct) — you need an external sanitizer.
 2. **Applying any transformations after sanitization.**
    Any transformations applied after the sanitization are not sanitized themselves, and could produce unsafe code.
 3. **Sanitizing in the process of transformations.**
    This usually gets broken very quickly, for the reasons stated above.
    There are more places to make mistakes this way, also there could be various interplay between some transformations.
    Also, some user-supplied content might even end up untransformed, and will be unsanitized because of that.
 4. **Having an insecure ruleset passed to the sanitizer.**
    E.g. allowing some elements/attributes that really shouldn't be allowed.
    Most issues present in this post are related to that.
 5. **Having the serialization broken.**
    If you are using a sanitizer (_you should be_) — it's mainly their task.
    The tree structure should be correctly encoded in a HTML form, e.g. forgetting to encode HTML attributes and putting `"`
    there unescaped will break things very badly.
 6. **Mistakes through transferring.**
    This is tangentially related to applying transformations after sanitization.
    E.g. if the HTML is stored/passed in a JSON form, it should be property encoded to that JSON form when doing so.
    If it's not — the attacker could use JSON escape sequences that will get unescaped by the time the HTML will be inserted to
    the web page.

### Ways to fix things

Make sure that:

 1. You or your library are using a specialized sanitizer.
 2. That sanitizer is known to be good enough.
 3. Your ruleset for the sanitizer is fine. There should be a **whitelist** of allowed tags and a **whitelist** of their allowed attributes,
    if you use those.
    For some attributes, including `id` and `class` (see below), there should be a whitelist or additional validation.
    Attributes like `href` also should never be allowed blindly — but the sanitizer should take care of that.
    Btw, if you are using just the defaults — you should make sure what exactly are those «defaults»,
    I have seen issues related to that (not mentioned here).
 4. You don't apply any transformation after the sanitization, including transfer errors.
 5. You use CSP which is not relaxed to a state where it does almost nothing.
 6. You have carefully checked the OWASP [cheatsheet](https://www.owasp.org/index.php/XSS_Filter_Evasion_Cheat_Sheet)
    (and other similar ones that you might find on the internet). Better to add those examples to your tests, btw.

## Unsanitized `class` attribute
[tl;dr](#tldr).

The most common issue had two prerequisites:
 1. Allowing a subset of HTML markup (that feature is included in Markdown spec),
 2. Code syntax highlighting.

Markdown parsers implementing (1) have to support both Markdown and HTML, and they have to sanitize the code in some way to avoid malicious content (e.g. embedded JS). The most obvious place for the sanitizer is the last step, after the markup is processed — that makes the logic resistant to various errors in the markup processor, which is rather complex and could potentially produce unsafe or invalid HTML if some bug is present in that processor.

So, the common rule of thumb is — after (or in the process) of the markup being processed, convert everything to DOM, then run the sanitizer to allow only whitelisted set of HTML tags and attributes, then serialize that DOM to HTML.

So far so good, what could go wrong there? And what does code syntax highlighting has to do with that?

Let's say we want to add syntax code highlight to our shiny and secure model (as described above). How would we do that? There are existing JS libraries for syntax code highlighting on the clientside, let's take one of those and just use that. It should just work, shouldn't it? _(Minor remark: that doesn't literally mean just throwing in thirparty software blindly, just making an in-house solution that behaves similar to already existing ones gives the same result here)._

But wait, our library wants us to specify a `class` attribute on `code` or `pre` elements to hint the language which syntax do we want to highlight.

So, what a programmer that doesn't think much about security at that point _or at least doesn't question why the rules for the sanitizer were written in the way they were present_, would do there?

### tl;dr

**It looks like most of the affected projects (those with the common issue of `class` not being sanitized) carefully landed that loophole for the sake of code syntax highlighting, and allowed arbitrary `class` attributes on `code` and/or `pre` elements.**

So `class` attribute was added to the exceptions for specific tags and was not sanitized for those tags, but why exactly is this bad? In fact, if there were no css or js on the webpage at all, there wouldn't be any difference. But in real world, there is a bunch of css rules and js code on all those pages, and both of those in most cases heavily rely on `class` properties of DOM elements.

### Utilizing existing CSS rules

Websites usually have a large number of CSS rules that allow styling of elements using their `class`es, and those rules could be utilized and combined to achieve any desired look of the webpage, by overriding different rules with other classes. The ability to use nested elements with different `class` attributes gives the attacker even more flexibility.

### Reusing existing JavaScript event listeners

`class` attributes are also widely used to attach JavaScript event listeners, e.g. `$('#someclass').click(function() {…})` or `$(document).on('click', '.someclass', function() {…})` (yeah, many sites still use jQuery, so I'm using those as an example).

Those already existing event handlers could be used by the attacker to execute some potentially malicious actions on user behalf.

## Affected software

I have found and reported markup sanitization related issues in: \
[GitHub](#github), [GitLab](#gitlab),
 _[TBA](#tba)_,
[Gogs](#gogs), [Gitea](#gitea), [Redmine](#redmine),
_[TBA 2](#tba-2)_,
_[TBA 3](#tba-3)_,
[Upsource](#upsource),
[JIRA](#jira).

Of those, [TBA](#tba), [TBA 2](#tba-2), [TBA 3](#tba-3), and [Upsource](#upsource) issues are not related to unsanitized `class` atribute.
Note that the █████ ██████ █████ █████ ███████ ███ █████████████ — █████████ ████ ███ ███████ ████ ██████ ██ ████████ ██████, ███ ███████ ███████ █████████ ██████
████████████. ████ ███████ ███████ ████████ ████████████ ████████ ████, though, and ██████ ███ ██████ ███ ████ ██ ██ ████████ ███████ writeup.

See below for the detailed vulnerabilities information, PoC examples and screenshots.

### GitHub

Fixed in [v2.9.2](https://enterprise.github.com/releases/2.9.2/notes).

The disclosure is available
[here](https://bounty.github.com/researchers/ChALkeR.html#user-controlled--class--attribute-on-some-markdown-tags-20170321).

#### Forged content
![Forged content](/media/github.forged.png)

The most simple PoC (as seen above) was achieved with this code:
```html
<code class="highlight org-migration-settings-icon">
<code class="highlight position-absolute survey-container org-migration-settings-icon">
<img src="https://cloud.githubusercontent.com/assets/291301/24167234/18c0446e-0e87-11e7-91bd-4f197b5481c1.png">
</code>
</code>
1
1
```
_And more padding with «1» below to adjust the next message / comment form placement._

This has several drawbacks — the image was clickable, on this exact screenshot the «1 participant» label wasn't affected, etc.
That all could have been avoided, but I was lazy there, and the above version was enough for the report.

#### Fullsize
![Fullsize message](/media/github.fullsize.png)

PoC:
```html
<code class="highlight facebox-overlay facebox-overlay-active bg-white rounded-0 text-gray-dark btn-block">
<code class="highlight d-block form-cards"></code>
<code class="highlight d-block"><img src="https://assets-cdn.github.com/pinned-octocat.svg" height="120"></code>
<code class="highlight h1 d-block">Hello there! Something has gone wrong, we are working on it.</code>
<code class="highlight h2 d-block">In the meantime, play a game with us at <a href="http://example.com">http://example.com/</a>.</code>
</code>
```

#### JS event handlers

GitHub binds event handlers by class name at many places in their JS client-side code (e.g. `…(document).on("click",".js-add-gist-file",function()…`), so it was possible to bind the desired pre-existing `onclick` event handler to the user content. I did not produce a full-featured PoC for this as it was not important for fixing the issue, so I preferred to report it faster instead. I made a simple demo that just logged an error to the console on click, though.

### GitLab

Fixed in [v9.0.4/v8.17.5/v8.16.9](https://about.gitlab.com/2017/04/05/gitlab-9-dot-0-dot-4-security-release/).

Disclosed on [HackerOne](https://hackerone.com/reports/216453).

![Fullsize message](/media/gitlab.fullsize.png)

Pretty much the same issue as in GitHub, so I won't repeat myself.

PoC:
```html
<pre class="zen-backdrop fullscreen sherlock-line-samples-table center">
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<img src="https://about.gitlab.com/images/press/logo/wm_no_bg.svg" width="500">
<h2>Hello there! Something has gone wrong, we are working on it.</h2>
<h3>In the meantime, play a game with us at <a href="http://example.com/">example.com</a>.</h3>
</pre>
```

I also made a simple JS action example just to demonstrate the concept, but it's unimpressive (hiding one element and showing another one on click), so I won't include a screenshot here.

### Gogs

![Fullsize message](/media/gogs.fullsize.png)

Fullsize message is pretty similar to the already mentioned ones.

PoC:
```html
<code class="ui tab active menu attached animating sidebar following bar center">
<code class="ui container input huge basic segment center">&nbsp;</code>
<img src="https://try.gogs.io/img/favicon.png" width="200" height="200">
<code class="ui container input massive basic segment">Hello there! Something has gone wrong, we are working on it.</code>
<code class="ui container input huge basic segment">In the meantime, play a game with us at&nbsp;<a href="http://example.com/">example.com</a>.</code>
</code>
```

![Basic JavaScript](/media/gogs.js.png)

That is a demo of content (actually, a same page) being loaded inside the comment on click. I did not go further, that was not relevant to fixing the issue.
That was a simple `<code class="ajax-load-button">Click me!</code>`.

### Gitea

A community-managed Gogs fork.

_Fixed in [this PR](https://github.com/go-gitea/gitea/pull/1461)._

![Fullsize message](/media/gitea.fullsize.png)

PoC is mostly the same as in Gogs, but with a different logo.

![Basic JavaScript](/media/gitea.js.png)

That one executes a basic POST request (and fails) on click. I did not go further.

### Redmine

Fixed in [v3.3.3/v3.2.6](http://www.redmine.org/news/111).

![Fullsize message](/media/redmine.fullsize.png)

PoC:
```html
<code class="total-hours ui-helper-reset ui-widget-overlay role-visibility ui-front"><pre>Hello there! Something has gone wrong, we are working on it.</pre><pre>In the meantime, play a game with us by clicking the link below:</pre> http://example.com/</code>
{repeated several times}
```

Could you spot the difference with the screenshots for other projects? This one looks very suspicious.

Redmine (in default setup) has a relatively small number of css rules and a small client-side JS file (which doesn't bind onclick handlers solely by the `class` attribute). Thus, the issue for the default Redmine setup has lower severity than for all other mentioned projects. The above screenshot was made with the default setup, though.

The severity could be raised in presence of various Redmine plugins that include their own js/css to the page, and installing thirdparty plugins is pretty popular in Redmine ecosystem.

### TBA

The issue here is different and is not related to `class` not being sanitized.

![XSS](/media/tba.xss.png)

This is a full-featured XSS, with arbitrary JS code, achievable through █████████████████████ █████████████.

PoC:
```html
TBA
```

_Details TBA._

██████████████████████████████████████████████████████████████████████████████████████.

Note: ███████████████████████ managed to disclose the XSS vulnerability before fixing it by testing the PoC in a pub███████████
█████ production server, soon after I reported the issue and provided the PoC privately, and it was public until I noticed that
after 19 hours.

### TBA 2

An issue in _TBA 2_ allowed arbitrary user content in `style` attribute (not related to `class` not being sanitized), which could
be used to obtain any desired look of the page (forging other people content, etc).

![Fullsize message on TBA2](/media/tba2.1.fullsize.png)

PoC:
```html
TBA
```

![Fullsize message on TBA2](/media/tba2.2.fullsize.png)

PoC:
```html
TBA
```

_Details TBA._

### TBA 3

The issues here are not related to `class` not being sanitized.

![XSS](/media/tba3.xss.png)

This project ha██████████████████████████████XSS██████████████████████ arbitrary JS code, ██████████████████████████████████████████████████████.

_Details TBA._

#### Using██████████████

PoC:
```html
TBA
```

#### ███████████████

████████ ██████████ ████████.

PoC:
```html
TBA
```

Note that ██████████████████████████████████████████,
███████████████ ████████████████████ ███████████.

### Upsource

_Fixed in: v2017.1.1876/v2017.2.703_.

The issue here is not related to `class` not being sanitized.

![XSS](/media/upsource.xss.png)

This is a full-featured XSS, with arbitrary JS code, achievable through comments (among other places).

This has already been publically disclosed by the JetBrains team prior to the fix, [link](https://github.com/OWASP/java-html-sanitizer/issues/110).\
tl;dr: I reported this issue to JetBrains privately with a XSS PoC against Upsource. They reported their PoC publically to the sanitizer library that they are using. Which turned out to be not the culprit.

The issue was caused by transformations that were applied to the content _after_ it pas passed through the sanitizer. _Don't do that, the sanitizer should be the very last thing you have in your pipeline._

At first, JetBrains rejected to acknowledge that `<iframe>` or `<script>Set.constructor&#96;alert\x28document.domain\x29&#96;&#96;&#96;</script>` appearing in their «sanitized» content means that their sanitizer is not functioning correctly (_that script on the screenshot below didn't get executed, but only because their comments apparently are populated dynamically through something similar to setting `.innerHTML`_), stated that the sanitizer is supposed to work that way and closed the issue as WONTFIX. I had to spend a bit more time because of that ignorance on their part — PoC with `onmouseover` which actually executed JS code was filed afterwards and the issue was reopened immediately after that and was fixed the same day.

![XSS](/media/upsource.xss.script.png)

#### PoCs:

* Injects a `<script>` tag that is not executed due to content dynamically spawned: `<a href='//example.com"></a><script>Set.constructor&#96;alert\x28document.domain\x29&#96;&#96;&#96;</script>`.
* Injects an `<iframe>` tag: `<img src='/"><iframe>`, or any other content without spaces.
* Injects a script that executes onmouseover: `<a href='/"&nbsp;xxx=&#39;'>xxx' onmouseover="alert('xss')" style="position:fixed;left:0;right:0;top:0;bottom:0;z-index:100;background:white"></a>`. Can be used to inject any other attributes.

### JIRA

_Not fixed, not treated as a security issue._

_This has been publically disclosed on [jira.atlassian.com](https://jira.atlassian.com/) — see [below](#reporting-issues) for more info._

JIRA is affected with the same unsanitized `class` attribute problem as the other projects mentioned above.

I notified them that _it gives the attacker an ability to exploit already existing CSS rules to style the content in an arbitrary way, and also allows them to use already present JS event listeners that are attached by class names_ (this is a citation).

I did not try hard to actually perform a full attack through this, and, as usual, sent only basic PoC [samples](#samples) to Atlassian and explained why that is dangerous with links to other projects explanations.

Atlassian decided that this is not a security issue, so I am now duplicating the disclosure here so that their users would be aware.

> We are currently not looking for reports of issues of this type.
> This page https://www.atlassian.com/trust/security/vulnerability-information-report shows what information we are looking for in a security report.

And:

> This is currently considered a bug and not a security issue. If you want to raise this bug with the team you can do so by creating a ticket at https://jira.atlassian.com

#### Full-size message with an event listener

This is for the current version of JIRA on [jira.atlassian.com](https://jira.atlassian.com/) (v7.3.0):

![Fullsize message on JIRA](/media/jira.fullsize.png)

PoC:
```
!http://oserv.org/bugs/jira/m.svg|class=jira-dialog jira-dialog-modeless basic issueaction-fields-add!
```

It attaches sample JS listener that pops up a dialog on click.

#### Samples

* `!http://oserv.org/i.png|class=aui-blanket!` — blocks JIRA in issue comments on v6.3.9.
* `!http://oserv.org/i.png|class=select2-drop-mask!` — blocks Servicedesk in issue comments.
* `!http://oserv.org/i.png|aui-dialog2-header-close issueaction-fields-add!` — attaches to a sample JS listener on JIRA and Servicedesk.

One can combine classes with `…|class=a b c d!`.

I leave it up to the readers to combine existing classes to forge on-screen content like I did with [GitHub](#forged-content).
I mentioned that this is possible to Atlassian, they apparently do not think that it is dangerous.

#### Reporting issues

~I am not sure in which direction I should continue from here, regarding the report to JIRA.~

~Atlassian security team closed the issue as «answered» and redirected me to [jira.atlassian.com](https://jira.atlassian.com/),
saying that I should open a regular bugreport there instead.
But I don't have permissions to open an issue there — it doesn't let me to.
Instead, https://jira.atlassian.com/ redirects me to [support.atlassian.com](https://support.atlassian.com/), which demands a
«JIRA Support Entitlement Number (SEN)» in order to report an issue — but I don't have anything like that.~

~I tried asking at various places how I should continue, but I didn't get any answer.~

_Update: Atlassian security team helped me to open an issue on [jira.atlassian.com](https://jira.atlassian.com/):
[JRACLOUD-66742](https://jira.atlassian.com/browse/JRACLOUD-66742), ~~it is treated as low-priority non-security issue.~~_

_Update 2: The issue linked above is private now, and it's treated as a medium priority security issue.
There is no sense in removing the PoC from this writeup now, but it is understandable why the discussion in the ticket itself is private._

## Notes

No other user content was involved in the PoCs.
All the screenshots that look like interaction from other users are actually just forged DOM content on the specific web page that did not involve that users.

### Managing security issues

I can not say that I agree with how the abovementioned security issues were treated for some of the reports here.

For example, I consider stuff like [this](https://www.atlassian.com/trust/security/vulnerability-information-report) harmful when
it is blindly followed — e.g. it basically says that clickjacking issues are not security issues that should be reported.
_Note: I did not discover and was not searching for clickjacking issues in Atlassian software, I am just commenting their policy._

As another example, I think that PoCs and vulnerability details disclosure should be avoided until the issue is actually
fixed, if possible — and at least two abovementioned projects managed to disclosed their PoCs prior to the fix while still
treating the corresponding issues as security vulnerabilities.

I believe there should be a good instruction on how to deal with security reports somewhere on the internet — if you are aware of
a good one, send me a link, I will include that link here.

---

Published (partially): 2017-04-13, 9:01 UTC. \
Updated: _TBA_.

If you have any questions to me, contact me over Gitter ([@ChALkeR](https://gitter.im/ChALkeR)) or IRC (ChALkeR@freenode).
