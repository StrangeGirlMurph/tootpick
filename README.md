# Tootpick

Tootpick is a privacy-preserving tool to create shareable links for Mastodon posts and provide an easy link target for a "Mastodon share button" similar to other social media share buttons or [mailto](https://en.wikipedia.org/wiki/Mailto) links.

Try this demo: [Share on Mastodon <img src="https://raw.githubusercontent.com/mastodon/mastodon/main/app/javascript/images/app-icon.svg" width=12>](http://tootpick.org/#text=Checkout%20%23tootpick%20a%20privacy-preserving%20tool%20to%20create%20shareable%20links%20for%20Mastodon%20posts%20and%20provide%20an%20easy%20link%20target%20for%20a%20%22Mastodon%20share%20button%22%20similar%20to%20other%20social%20media%20share%20buttons%20%F0%9F%8E%89)

## Using Tootpick

tl;dr Visit <https://tootpick.org/> and share the generated link.

To allow visitors to share a message via Mastodon, provide a link to <https://tootpick.org/> with the text parameter being set to your URI encoded message. Something like this: `https://tootpick.org/#text=Everyone%20should%20visit%20https://example.org/`. In the link the message after `https://tootpick.org/#text=` must be URI encoded, i.e. spaces must be replaced by `%20`, ampersands by `%26`, etcetera, just like it is common for URLs. The good news is tootpick can do that for you. If you visit <https://tootpick.org/> without providing the `#text=...` parameter you get a textfield to enter a message and generate a link for it!

Visitor of that link with the text parameter set will then pick their Mastodon instance via tootpick and post the message "Everyone should visit https://example.org/" on the specified instance. 

Most Mastodon instances usually have a character limit of 500 so it's wise to keep your message shorter than that. Tootpick will warn you when you go over that.

## Design goals

### Nothing fancy

The functionality is simple and does not requires bells and whistles. In true
KISS spirit, Tootpick is a single static HTML page. It can be used through
the hosted tootpick.org, or by self-hosting the single HTML file.

### Privacy

Tootpick uses the "hash" or "fragment" part of the URL, which is not sent to
the HTTP server, for passing the message. Javascript is used to parse the URL
and pass the data onto the chosen Mastodon instance. Only the instance server
will actually receive the contents of the message to be posted.

Because Tootpick is a static page in between the article that has the "share"
link and the mastodon instance, the original Referer URL won't make it to the
instance, hiding any possibly sensitive tracking information from the instance
admins. (It is not possible for Tootpick to prevent sending the Referer to the
server that hosts Tootpick.)

> Hi, it's me, the author of Tootpick. Because I'm not collecting any data
> from Tootpick users, I have no idea who uses it. If you're using Tootpick on
> your website, and want to let me know, please "star" the project on GitHub or
> send me a message on Mastodon (`@whreq@hsnl.social`) - that way I know
> whether I should keep maintaining it. Thanks!

### Multiple instances

Most existing instance pickers will provide an option to remember the last used
instance, but that's not sufficiently helpful for users who use multiple
instances, which is not entirely uncommon. Tootpick remembers multiple recently
used instances.

### Browser support

If the main feature works for 99% of all internet users, and 98% get the site
with the intended aesthetics, that's good enough. Tootpick does not support
ancient browsers like IE, and may look ugly and have degraded feature
availability on old browsers. (The percentages are completely made up and just
for illustrating the point.)

It should also be usable by people who use assistive technologies, but that
comes for free by using normal HTML.

### Fix PEBCAKs

Real people make mistakes, and some common mistakes can be recognised and fixed
by the script. A user may provide the wrong domain, a URL instead of the bare
domain name, an account, or the account domain instead of the web domain, and
Tootpick tries to recover from that and do the right thing.

<details><summary>

## Why not just link directly to Mastodon?

</summary>
Mastodon is part of the fediverse, a federated network. By design, there is no
central server or central domain name. That direct link option you might be
thinking about, does not exist. Please don't make the mistake of picking a
large server and linking directly to its /share URL, because the majority of
Mastodon users will not be on that server.
</details>

<details><summary>

## Self-hosting

</summary>
Instead of using the central tootpick.org service, tootpick can also be
self-hosted by downloading the static HTML file. Tootpick is AGPL licensed, but
as long as it remains a static HTML page, the requirement of sharing the source
code of a derivative work is automatically fulfilled.

Self-hosting comes with a privacy trade-off: a self-hosted (modified) copy of
Tootpick could be used to collect the Mastodon instances used by the visitor,
but on the other hand hides visitor IP addresses from the server that hosts the
service on tootpick.org.
</details>

<details><summary>

## Button image

</summary>
No button image is provided at this point, but the <a href="https://github.com/mastodon/mastodon/tree/main/app/javascript/images"> official Mastodon logo</a> is probably a good starting point for designing your own.
</details>

<details><summary>

## Fragment parameters

</summary>
Although Tootpick currently only uses a single parameter, `text`, it is ready
for accepting more than that.

The parsing of the URI fragment, that is part after the `#`, is done as
described in the <a href="https://www.w3.org/TR/media-frags/#processing-name-value-components">Media Fragments URI
specification</a>
which assumes RFC 3986 URI escaping. (Note: Tootpick does not use media
fragments, just the syntax for parameters in URI fragments.)

This means that the reserved characters `&` and `=` are used as delimiters.
They must be unescaped when used as delimiters, and must be escaped when used
in a key or value. The fragment must not be encoded as a whole, because that
would encode the delimiters as well; only its part should be individually
encoded. Additionally, `+`, while "reserved", is not treated as a special
character, which means that a space must be encoded as `%20`, not `+`. RFC 3986
URI encoding functions always encode spaces as `%20`.

Most programming languages come with a function that does the right thing
for use with Tootpick.

```JavaScript
// JavaScript example
document.location = 'https://tootpick.org/#text=' + encodeURIComponent(
  document.title + ' ' + document.URL
);
```

| Language   | Function                             |
| ---------- | ------------------------------------ |
| Go         | url.PathEscape() from net/url        |
| JavaScript | EncodeURIComponent()                 |
| PHP        | rawurlencode()                       |
| Perl       | uri\_escape\_utf8() from URI::Escape |
| Python     | quote() from urllib.parse            |
| Raku       | uri-escape() from URI::Escape        |
| Ruby       | url\_encode() from ERB::Util         |

Some environments don't provide a URI encoding function that can be used for
fragments. For these, Tootpick provides a parameter `plustospace`, which when
set to `yes`, will cause `+` signs to be changed to space characters, which
would normally be wrong. It can be used like
`https://tootpick.org/#plustospace=yes&text=Your+text+here`.

| Language | Function that can be used with `plustospace=yes` |
| -------- | ------------------------------------------------ |
| Java     | java.net.URLEncoder.encode()                     |
| Jekyll   | cgi\_escape()                                    |
</details>

<details><summary>

## Similar projects

</summary>
Tootpick is not the first of its kind. It draws inspiration from:

- [toot](https://codeberg.org/kytta/toot)
- [Advanced Sharer for Mastodon](https://sharetomastodon.github.io/about/)
- [Mastodon Share Button](https://aly-ve.github.io/Mastodon-share-button/)
</details>

<details><summary>

## Future improvements

</summary>
At some point in the future, I hope to:

- Detect the visitor's language and show translated texts.
- Supply a list of existing instances. Not sure if it's useful enough to add
  extra weight, and not sure how to determine which instances to include or
  exclude. Please let me know what you think!
</details>
