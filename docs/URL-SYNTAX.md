# URL syntax

## Specifications

The official "URL syntax" is primarily defined in these two different
specifications:

 - [RFC 3986](https://tools.ietf.org/html/rfc3986) (although URL is called "URI" in there)
 - [The WHATWG URL Specification](https://url.spec.whatwg.org/)

RFC 3986 is the earlier one, and curl has always tried to adhere to that one
(since it shipped in January 2005).

The WHATWG URL spec was written later, is incompatible with the RFC 3986 and
changes over time.

## Variations

URL parsers as implemented in browsers, libraries and tools usually opt to
support one of the mentioned specifications. Bugs, differences in
interpretations and the moving nature of the WHATWG spec does however make it
very unlikely that multiple parsers treat URLs the exact same way!

## Security

Due to the inherent differences between URL parser implementations, it is
considered a security risk to mix different implementations and assume the
same behavior!

For example, if you use one parser to check if a URL uses a good host name or
the correct auth field, and then pass on that same URL to a *second* parser,
there will always be a risk it treats the same URL differently. There is no
right and wrong in URL land, only differences of opinions.

libcurl offers a separate API to its URL parser for among others, this reason.

## "RFC3986 plus"

curl recognizes a URL syntax that we call "RFC 3986 plus". It is grounded on
the well established RFC 3986 to make sure previously written command lines and
curl using scripts will remain working.

curl's URL parser allows a few deviations from the spec in order to
inter-operate better with URLs that appear in the wild.

### spaces

In particular `Location:` headers that indicate to the client where a
resource has been redirected, sometimes contain spaces. This is a violation of
RFC 3986 but is fine in the WHATWG spec. curl handles these by re-encoding
them to `%20`.

### non-ASCII

Byte values in a provided URL that are outside of the printable ASCII range
are percent-encoded by curl.

### multiple slashes

An absolute URL always starts with a "scheme" followed by a colon. For all the
schemes curl supports, the colon must be followed by two slashes according to
RFC 3986 but not according to the WHATWG spec - which allows one to infinity
amount.

curl allows, one, two or three slashes after the colon to still be considered
a valid URL.

### "scheme-less"

curl supports "URLs" that do not start with a scheme. This is not supported by
any of the specifications. This is a shortcut to entering URLs that was
supported by browsers early on and has been mimicked by curl.

Based on what the host name starts with, curl will "guess" what protocol to
use:

 - `ftp.` means FTP
 - `dict.` means DICT
 - `ldap.` means LDAP
 - `imap.` means IMAP
 - `smtp.` means SMTP
 - `pop3.` means POP3
 - all other means HTTP

### globbing letters

The curl command line tool supports "globbing" of URLs. It means that you can
create ranges and lists using `[N-M]` and `{one,two,three}` sequences. The
letters used for this (`[]{}`) are reserved in RFC 3986 and can therefore not
legitimately be part of such a URL.

They are however not reserved or special in the WHATWG specification, so
globbing can mess up such URLs. Globbing can be turned off for such occasions
(using `--globoff`).
