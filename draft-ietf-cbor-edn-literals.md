---
v: 3

title: >
  CBOR Extended Diagnostic Notation (EDN):
  Application-Oriented Literals, ABNF, and Media Type
abbrev: >
  CBOR EDN: Literals and ABNF
docname: draft-ietf-cbor-edn-literals-latest
date: 2023-10-01

keyword: Internet-Draft
cat: info
stream: IETF
# consensus: true

pi: [toc, sortrefs, symrefs, compact, comments]

author:
  -
    ins: C. Bormann
    name: Carsten Bormann
    org: Universität Bremen TZI
    street: Postfach 330440
    city: Bremen
    code: D-28359
    country: Germany
    phone: +49-421-218-63921
    email: cabo@tzi.org

venue:
  mail: cbor@ietf.org
  github: cbor-wg/edn-literal
  latest: "https://cbor-wg.github.io/edn-literal/"

normative:
  STD94:
    -: cbor
    =: RFC8949
  RFC8610: cddl
  I-D.ietf-cbor-update-8610-grammar: cddlupd
  RFC8742: seq
  STD90:
    -: json
    =: RFC8259
  STD68:
   -: abnf
   =: RFC5234
  RFC7405: abnfcs
  I-D.ietf-core-href: cri
  RFC3339: datetime
  RFC3986: uri
  RFC3987: iri
  RFC9165: controls
  BCP26:
    -: ianacons
    =: RFC8126
  STD80:
    -: ascii
    =: RFC20
informative:
  RFC4648: base
  IANA.core-parameters:

--- abstract

The Concise Binary Object Representation, CBOR (STD 94, RFC 8949), [^abs1-]

[^abs1-]: defines a "diagnostic notation" in order to
    be able to converse about CBOR data items without having to resort to
    binary data.

[^abs3-]: This document specifies how to add application-oriented extensions to
    the diagnostic notation.  It then defines two such extensions for
    text representations of epoch-based date/times and of Constrained Resource Identifiers

​[^abs3-] (draft-ietf-core-href).

[^abs4-]: To facilitate tool interoperation, this document also
     specifies a formal ABNF definition for extended diagnostic notation (EDN)
     that accommodates application-oriented literals.

[^abs4-]


--- middle

Introduction        {#intro}
============

For the Concise Binary Object Representation, CBOR,
{{Section 8 of -cbor}} in conjunction with {{Appendix G of -cddl}}
[^abs1-]
Diagnostic notation syntax is based on JSON, with extensions
for representing CBOR constructs such as binary data and tags.
[^abs2-]

[^abs2-]: (Standardizing this together with the actual interchange format does
    not serve to create another interchange format, but enables the use of
    a shared diagnostic notation in tools for and in documents about CBOR.)

[^abs3-] {{-cri}}.

[^abs4-] (See {{grammar}} for an overall ABNF grammar as well as the
ABNF definitions in {{app-grammars}} for grammars for both the
byte string presentations predefined in {{-cbor}} and the application-extensions).

[^cri-later]

[^cri-later]: Note that {{cri}} and {{cri-grammar}} about CRIs may move to the {{-cri}}
    specification, depending on the relative speed of approval; the
    later document gets the section.

## Terminology

{{Section 8 of -cbor}} defines the original CBOR diagnostic notation,
and {{Appendix G of -cddl}} supplies a number of extensions to the
diagnostic notation that result in the Extended Diagnostic Notation
(EDN).
The diagnostic notation extensions include popular features such as
embedded CBOR (encoded CBOR data items in byte strings) and comments.
A simple diagnostic notation extension that enables representing CBOR
sequences was added in {{Section 4.2 of -seq}}.
As diagnostic notation is not used in the kind of interchange
situations where backward compatibility would pose a significant
obstacle, there is little point in not using these extensions.

Therefore, when we refer to "_diagnostic notation_", we mean to
include the original notation from {{Section 8 of -cbor}} as well as the
extensions from {{Appendix G of -cddl}}, {{Section 4.2 of -seq}}, and the
present document.
However, we stick to the abbreviation "_EDN_" as it has become quite
popular and is more sharply distinguishable from other meanings than
"DN" would be.

In a similar vein, the term "ABNF" in this document refers to the
language defined in {{-abnf}} as extended in {{-abnfcs}}, where the
"characters" of {{Section 2.3 of -abnf}} are Unicode scalar values.
The term "CDDL" refers to the data definition language defined in
{{-cddl}} and its registered extensions (such as those in {{RFC9165}}), as
well as {{-cddlupd}}.

Application-Oriented Extension Literals
=======================================

This document extends the syntax used in diagnostic notation for byte
string literals to also be available for application-oriented extensions.

As per {{Section 8 of -cbor}}, the diagnostic notation can notate byte
strings in a number of {{-base}} base encodings, where the encoded text
is enclosed in single quotes, prefixed by an identifier (»h« for
base16, »b32« for base32, »h32« for base32hex, »b64« for base64 or
base64url).

This syntax can be thought to establish a name space, with the names
"h", "b32", "h32", and "b64" taken, but other names being unallocated.
The present specification defines additional names for this namespace,
which we call *application-extension identifiers*.
For the quoted string, the same rules apply as for byte strings.
In particular, the escaping rules of JSON strings are applied
equivalently for application-oriented extensions, e.g., within the
quoted string `\\` stands
for a single backslash and `\'` stands for a single quote.

An application-extension identifier is a name consisting of a
lower-case ASCII letter (a-z) and zero or more additional ASCII
characters that are either lower-case letters or digits (a-z0-9).

Application-extension identifiers are registered in a registry
({{appext-iana}}).
Prefixing a single-quoted string, an application-extension identifier
is used to build an application-oriented extension literal, which
stands for a CBOR data item the value of which is derived from the
text given in the single-quoted string using a procedure defined in
the specification for an application-extension identifier.

Examples for application-oriented extensions to CBOR diagnostic
notation can be found in the following sections.

In addition, this document finally registers a media type identifier
and a content-format for CBOR diagnostic notation.  This does not
elevate its status as an interchange format, but recognizes that
interaction between tools is often smoother if media types can be used.


The "cri" Extension {#cri}
===================

The application-extension identifier "cri" is used to notate a
Constrained Resource Identifier literal as per {{-cri}}.

The text of the literal is a URI Reference as per {{-uri}} or an IRI
Reference as per {{-iri}}.

The value of the literal is a CRI that can be converted to the text of
the literal using the procedure of {{Section 6.1 of -cri}}.
Note that there may be more than one CRI that can be converted to the
URI/IRI given; implementations are expected to favor the simplest
variant available and make non-surprising choices otherwise.


As an example, the CBOR diagnostic notation

~~~ cbor-diag
cri'https://example.com/bottarga/shaved'
~~~

is equivalent to

~~~ cbor-diag
[-4, ["example", "com"], ["bottarga", "shaved"]]
~~~

See {{cri-grammar}} for an ABNF definition for the content of CRI literals.


The "dt" Extension {#dt}
==================

The application-extension identifier "dt" is used to notate a
date/time literal that can be used as an Epoch-Based Date/Time as per
{{Section 3.4.2 of -cbor}}.

The text of the literal is a Standard Date/Time String as per
{{Section 3.4.1 of -cbor}}.

The value of the literal is a number representing the result of a
conversion of the given Standard Date/Time String to an Epoch-Based
Date/Time.
If fractional seconds are given in the text (production
`time-secfrac` in {{abnf-grammar-dt}}), the value is a
floating-point number; the value is an integer number otherwise.

As an example, the CBOR diagnostic notation

~~~ cbor-diag
[dt'1969-07-21T02:56:16Z',
 dt'1969-07-21T02:56:16.5Z']
~~~

is equivalent to

~~~ cbor-diag
[-14159024, -14159023.5]
~~~

See {{dt-grammar}} for an ABNF definition for the content of DT literals.

IANA Considerations {#sec-iana}
===================

[^to-be-removed]

[^to-be-removed]: RFC Editor: please replace RFCthis with the RFC
    number of this RFC, \[IANA.cbor-diagnostic-notation] with a
    reference to the new registry group, and remove this note.


## CBOR Diagnostic Notation Application-extension Identifiers Registry {#appext-iana}

IANA is requested to create an "Application-Extension Identifiers"
registry in a new "CBOR Diagnostic Notation" registry group
\[IANA.cbor-diagnostic-notation], with the policy "expert review"
({{Section 4.5 of -ianacons}}).

The experts are instructed to be frugal in the allocation of
application-extension identifiers that are suggestive of generally applicable semantics,
keeping them in reserve for application-extensions that are likely to enjoy wide
use and can make good use of their conciseness.
The expert is also instructed to direct the registrant to provide a
specification ({{Section 4.6 of -ianacons}}), but can make exceptions,
for instance when a specification is not available at the time of
registration but is likely forthcoming.
If the expert becomes aware of application-extension identifiers that are deployed and
in use, they may also initiate a registration on their own if
they deem such a registration can avert potential future collisions.
{: #de-instructions}

Each entry in the registry must include:

{:vspace}
Application-Extension Identifier:
: a lower case ASCII {{-ascii}} string that starts with a letter and can
  contain letters and digits after that (`[a-z][a-z0-9]*`). No other
  entry in the registry can have the same function name.

Description:
: a brief description

Change Controller:
: (see {{Section 2.3 of -ianacons}})

Reference:
: a reference document that provides a description of the
  application-extension identifier


The initial content of the registry is shown in {{tab-iana}}; all
entries have the Change Controller "IETF".

| Application-extension Identifier | Description                     | Reference |
|----------------------------------|---------------------------------|-----------|
| h                                | Reserved                        | RFC8949   |
| b32                              | Reserved                        | RFC8949   |
| h32                              | Reserved                        | RFC8949   |
| b64                              | Reserved                        | RFC8949   |
| cri                              | Constrained Resource Identifier | RFCthis   |
| dt                               | Date/Time                       | RFCthis   |
{: #tab-iana title="Initial Content of Application-extension
Identifier Registry"}



## Media Type

IANA is requested to add the following Media-Type to the "Media Types"
registry {{!IANA.media-types}}.

| Name            | Template                    | Reference              |
| cbor-diagnostic | application/cbor-diagnostic | RFC XXXX, {{media-type}} |
{: #new-media-type align="left" title="New Media Type application/cbor-diagnostic"}

{:compact}
Type name:
: application

Subtype name:
: cbor-diagnostic

Required parameters:
: N/A

Optional parameters:
: N/A

Encoding considerations:
: binary (UTF-8)

Security considerations:
: {{seccons}} of RFC XXXX

Interoperability considerations:
: none

Published specification:
: {{media-type}} of RFC XXXX

Applications that use this media type:
: Tools interchanging a human-readable form of CBOR

Fragment identifier considerations:
: The syntax and semantics of fragment identifiers is as specified for
  "application/cbor".  (At publication of RFC XXXX, there is no
  fragment identification syntax defined for "application/cbor".)

Additional information:
: \\

  Deprecated alias names for this type:
  : N/A

  Magic number(s):
  : N/A

  File extension(s):
  : .diag

  Macintosh file type code(s):
  : N/A

Person & email address to contact for further information:
: CBOR WG mailing list (cbor@ietf.org),
  or IETF Applications and Real-Time Area (art@ietf.org)

Intended usage:
: COMMON

Restrictions on usage:
: none

Author/Change controller:
: IETF

Provisional registration:
: no

## Content-Format

IANA is requested to register a Content-Format number in the
{{content-formats ("CoAP Content-Formats")<IANA.core-parameters}}
sub-registry, within the "Constrained RESTful Environments (CoRE)
Parameters" Registry {{IANA.core-parameters}}, as follows:

| Content-Type                | Content Coding | ID   | Reference |
| application/cbor-diagnostic | -              | TBD1 | RFC XXXX  |
{: align="left" title="New Content-Format"}

TBD1 is to be assigned from the space 256..999.


Security considerations {#seccons}
=======================

The security considerations of {{-cbor}} and {{-cddl}} apply.

[^todo2]

[^todo2]: Anything else meaningful to say here?

--- back

ABNF Definitions {#grammars}
================

Overall ABNF Definition for Extended Diagnostic Notation {#grammar}
--------------------------------------------------------

This appendix provides an overall ABNF definition for the syntax of
CBOR extended diagnostic notation.

To complete the parsing of an `app-string` with prefix, say, `p`, the
processed `sqstr` inside it is further parsed using the ABNF definition specified
for the production `app-string-p` in {{app-grammars}}.

For simplicity, the internal parsing for the built-in EDN prefixes is
specified in the same way.
ABNF definitions for `h''` and `b64''` are provided in {{h-grammar}} and
{{b64-grammar}}.
However, the prefixes `b32''` and `h32''` are not in wide use and an
ABNF definition in this document could therefore not be based on
implementation experience.

~~~ abnf
{::include cbor-diag-parser.abnf}
~~~
{: #abnf-grammar "ABNF Definition of CBOR EDN" sourcecode-name="cbor-edn.abnf"}

While an ABNF grammar defines the set of character strings that are
considered to be valid EDN by this ABNF, the mapping of these
character strings into the generic data model of CBOR is not always
obvious.

The following additional items should help in the interpretation:

* `decnumber` stands for an integer in the usual decimal notation, unless at
  least one of the optional parts starting with "." and "e" are
  present, in which case it stands for a floating point value in the
  usual decimal notation.
* `basenumber` stands for an integer in the usual base 16/hexadecimal
  ("0x"), base 8/octal ("0o"), or base 2/binary ("0b") notation, unless the
  optional part containing a "p" is present, in which case it stands
  for a floating point number in the usual hexadecimal notation (which
  uses a mantissa in hexadecimal and an exponent in decimal notation).
* `spec` stands for an encoding indicator.
  As per {{Section 8.1 of -cbor}}:

  * an underscore `_` on its own stands
    for indefinite length encoding (`ai=31`, only available behind the
    opening brace/bracket for `map` and `array`: strings have a special
    syntax `streamstring` for indefinite length encoding except for the
    special cases ''_ and ""_), and
  * `_0` to `_3` stand for `ai=24` to `ai=27`, respectively.

  Surprisingly, {{Section 8.1 of -cbor}} does not address `ai=0` to
  `ai=23` — the assumption seems to be that preferred serialization
  (Section 4.1 of {{-cbor}}) will be used when converting CBOR
  diagnostic notation to an encoded CBOR data item, so leaving out the
  encoding indicator for a data item with a preferred serialization
  will implicitly use `ai=0` to `ai=23` if that is possible.
  The present specification allows to make this explicit:

  * A double underscore `__` stands for encoding with `ai=0` to `ai=23`.

  While no immediate use for further values for encoding indicators
  comes to mind, this is an extension point for EDN; {reg-ei}} defines
  a registry for additional values.

ABNF Definitions for app-string Content {#app-grammars}
---------------------------------------

This appendix provides ABNF definitions for application-oriented extension
literals defined in {{-cbor}} and in this specification.
These grammars describe the *decoded* content of the `sqstr` components that
combine with the application-extension identifiers to form
application-oriented extension literals.
Each of these may make use of rules defined in {{abnf-grammar}}.

### h: ABNF Definition of Hexadecimal representation of a byte string {#h-grammar}


The syntax of the content of byte strings represented in hex,
such as `h''`, `h'0815`, or `h'/head/ 63 /contents/ 66 6f 6f'`
(another representation of `<< "foo" >>`), is described by the ABNF in {{abnf-grammar-h}}.
This syntax accommodates both lower case and upper case hex digits, as
well as blank space (including comments) around each hex digit.

~~~ abnf
app-string-h    = S *(HEXDIG S HEXDIG S)
~~~
{: #abnf-grammar-h sourcecode-name="cbor-edn-h.abnf"
title="ABNF Definition of Hexadecimal Representation of a Byte String"
}


### b64: ABNF Definition of Base64 representation of a byte string {#b64-grammar}


The syntax of the content of byte strings represented in base64 is
described by the ABNF in {{abnf-grammar-h}}.

This syntax allows both the classic ({{Section 4 of RFC4648}}) and the
URL-safe ({{Section 5 of RFC4648}}) alphabet to be used.
It accommodates, but does not require base64 padding.
Note that inclusion of classic base64 makes it impossible to have
in-line comments in b64, as "/" is valid base64-classic.

~~~ abnf
app-string-b64  = B *(4(b64dig B))
                  [b64dig B b64dig B ["=" B "=" / b64dig B ["="]] B]
b64dig          = ALPHA / DIGIT / "-" / "_" / "+" / "/"
B               = *iblank *(icomment *iblank)
iblank          = %x0A / %x20  ; Not HT or CR (gone)
icomment        = "#" *inon-lf %x0A
inon-lf         = %x20-D7FF / %xE000-10FFFF
~~~
{: #abnf-grammar-b64 sourcecode-name="cbor-edn-b64.abnf"
title="ABNF definition of Base64 Representation of a Byte String"
}

### dt: ABNF Definition of RFC 3339 Representation of a Date/Time {#dt-grammar}

The syntax of the content of `dt` literals can be described by the
ABNF for `date-time` from {{RFC3339}} as summarized in {{Section 3 of -controls}}:

~~~ abnf
app-string-dt   = date-time

date-fullyear   = 4DIGIT
date-month      = 2DIGIT  ; 01-12
date-mday       = 2DIGIT  ; 01-28, 01-29, 01-30, 01-31 based on
                          ; month/year
time-hour       = 2DIGIT  ; 00-23
time-minute     = 2DIGIT  ; 00-59
time-second     = 2DIGIT  ; 00-58, 00-59, 00-60 based on leap sec
                          ; rules
time-secfrac    = "." 1*DIGIT
time-numoffset  = ("+" / "-") time-hour ":" time-minute
time-offset     = "Z" / time-numoffset

partial-time    = time-hour ":" time-minute ":" time-second
                  [time-secfrac]
full-date       = date-fullyear "-" date-month "-" date-mday
full-time       = partial-time time-offset

date-time       = full-date "T" full-time
DIGIT           =  %x30-39 ; 0-9
~~~
{: #abnf-grammar-dt sourcecode-name="cbor-edn-dt.abnf"
title="ABNF Definition of RFC3339 Representation of a Date/Time"
}


### cri: ABNF Definition of URI Representation of a CRI {#cri-grammar}

The syntax of the content of `cri` literals can be described by the
ABNF for `URI-reference` in {{Section 4.1 of -uri}}, as reproduced
in {{abnf-grammar-cri}}.
If the content is not ASCII only (i.e., for IRIs), first apply
{{Section 3.1 of RFC3987}} and apply this grammar to the result.

~~~ abnf
app-string-cri = URI-reference
; ABNF from RFC 3986:

URI           = scheme ":" hier-part [ "?" query ] [ "#" fragment ]

hier-part     = "//" authority path-abempty
                 / path-absolute
                 / path-rootless
                 / path-empty

URI-reference = URI / relative-ref

absolute-URI  = scheme ":" hier-part [ "?" query ]

relative-ref  = relative-part [ "?" query ] [ "#" fragment ]

relative-part = "//" authority path-abempty
                 / path-absolute
                 / path-noscheme
                 / path-empty

scheme        = ALPHA *( ALPHA / DIGIT / "+" / "-" / "." )

authority     = [ userinfo "@" ] host [ ":" port ]
userinfo      = *( unreserved / pct-encoded / sub-delims / ":" )
host          = IP-literal / IPv4address / reg-name
port          = *DIGIT

IP-literal    = "[" ( IPv6address / IPvFuture  ) "]"

IPvFuture     = "v" 1*HEXDIG "." 1*( unreserved / sub-delims / ":" )

IPv6address   =                            6( h16 ":" ) ls32
                 /                       "::" 5( h16 ":" ) ls32
                 / [               h16 ] "::" 4( h16 ":" ) ls32
                 / [ *1( h16 ":" ) h16 ] "::" 3( h16 ":" ) ls32
                 / [ *2( h16 ":" ) h16 ] "::" 2( h16 ":" ) ls32
                 / [ *3( h16 ":" ) h16 ] "::"    h16 ":"   ls32
                 / [ *4( h16 ":" ) h16 ] "::"              ls32
                 / [ *5( h16 ":" ) h16 ] "::"              h16
                 / [ *6( h16 ":" ) h16 ] "::"

h16           = 1*4HEXDIG
ls32          = ( h16 ":" h16 ) / IPv4address
IPv4address   = dec-octet "." dec-octet "." dec-octet "." dec-octet
dec-octet     = DIGIT                 ; 0-9
                 / %x31-39 DIGIT         ; 10-99
                 / "1" 2DIGIT            ; 100-199
                 / "2" %x30-34 DIGIT     ; 200-249
                 / "25" %x30-35          ; 250-255

reg-name      = *( unreserved / pct-encoded / sub-delims )

path          = path-abempty    ; begins with "/" or is empty
                 / path-absolute   ; begins with "/" but not "//"
                 / path-noscheme   ; begins with a non-colon segment
                 / path-rootless   ; begins with a segment
                 / path-empty      ; zero characters

path-abempty  = *( "/" segment )
path-absolute = "/" [ segment-nz *( "/" segment ) ]
path-noscheme = segment-nz-nc *( "/" segment )
path-rootless = segment-nz *( "/" segment )
path-empty    = 0<pchar>

segment       = *pchar
segment-nz    = 1*pchar
segment-nz-nc = 1*( unreserved / pct-encoded / sub-delims / "@" )
                 ; non-zero-length segment without any colon ":"

pchar         = unreserved / pct-encoded / sub-delims / ":" / "@"

query         = *( pchar / "/" / "?" )

fragment      = *( pchar / "/" / "?" )

pct-encoded   = "%" HEXDIG HEXDIG

unreserved    = ALPHA / DIGIT / "-" / "." / "_" / "~"
reserved      = gen-delims / sub-delims
gen-delims    = ":" / "/" / "?" / "#" / "[" / "]" / "@"
sub-delims    = "!" / "$" / "&" / "'" / "(" / ")"
                 / "*" / "+" / "," / ";" / "="
~~~
{: #abnf-grammar-cri sourcecode-name="cbor-edn-cri.abnf"
title="ABNF Definition of URI Representation of a CRI"
}

EDN and CDDL
============

EDN was designed as a language to provide a human-readable
representation of an instance, i.e., a single CBOR data item or CBOR
sequence.
CDDL was designed as a language to describe an (often large) set of
such instances (which itself constitutes a language), in the form of a
_data definition_ or _grammar_ (or sometimes called _schema_).

The two languages share some similarities, not the least because they
have mutually inspired each other.
But they have very different roots:

* EDN syntax is an extension to JSON syntax {{-json}}.
  (Any (interoperable) JSON text is also valid EDN.)
* CDDL syntax is inspired by ABNF's syntax {{-abnf}}.

For engineers that are using both EDN and CDDL, it is easy to write
"CDDLisms" or "EDNisms" into their drafts that are meant to be in the
other language.
(This is one more of the many motivations to always validate formal
language instances with tools.)

Important differences include:

* Comment syntax.  CDDL inherits ABNF's semicolon-delimited end of
  line characters, while EDN finds nothing in JSON that could be inherited here.
  Inspired by JavaScript, EDN simplifies JavaScript's copy of the
  original C comment syntax to be delimited by single slashes (where
  line ends are not of interest); it also adds end-of-line comments
  starting with `#`.

  {:compact}
  EDN:
  : ~~~ cbor-diag
    { / alg / 1: -7 / ECDSA 256 / }
    ,
    { 1:   # alg
        -7 # ECDSA 256
    }
    ~~~

  CDDL:
  : `? 1 => int / tstr,  ; algorithm identifier`

* Syntax for tags.  CDDL's tag syntax is part of the system for
  referring to CBOR's fundamentals (the major type 6, in this case)
  and (with {{-cddlupd}}) allows specifying the actual tag number
  separately, while EDN's tag syntax is a simple decimal number and a
  pair of parentheses.

  EDN:
  : `98(['', {}, /rest elided here: …/])`

  CDDL:
  : `COSE_Sign_Tagged = #6.98(COSE_Sign)`

* Separator character.  Like JSON, EDN requires commas as separators
  between array elements and map members and doesn't allow a trailing
  comma before the closing bracket/brace.
  CDDL's comma separators in these contexts (CDDL groups) are optional
  (and actually are terminators, which together with their optionality
  allows them to be used like separators as well or even not at all).

* Embedded CBOR.  EDN has a special syntax to describe the content of
  byte strings that are encoded CBOR data items.  CDDL can specify
  these with a control operator, which looks very different.

  EDN:
  : `98([/h'a10126'/ << {/alg/ 1: -7 /ECDSA 256/ } >>, /…/])`

  CDDL:
  : `serialized_map = bytes .cbor header_map`



Acknowledgements
================
{: numbered="no"}

The concept of application-oriented extensions to diagnostic notation,
as well as the definition for the "dt" extension were inspired by the
CoRAL work by Klaus Hartke.

<!--  LocalWords:  dedenting dedented
 -->
