---
v: 3

title: >
  CBOR Extended Diagnostic Notation (EDN):
  Application-Oriented Literals, ABNF, and Media Type
abbrev: >
  CBOR EDN: Literals and ABNF
docname: draft-ietf-cbor-edn-literals-latest
date: 2024-02-01

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
  STD94: cbor
  RFC8610: cddl
  RFC8742: seq
  STD68: abnf
  RFC7405: abnfcs
  RFC3339: datetime
  RFC3986: uri
  RFC9164: iptag
  IANA.cbor-tags: tags
  IANA.media-types:
  IANA.core-parameters:
  BCP26: ianacons
  STD80: ascii
  IEEE754:
    target: https://ieeexplore.ieee.org/document/8766229
    title: IEEE Standard for Floating-Point Arithmetic
    author:
    - org: IEEE
    date: false
    seriesinfo:
      IEEE Std: 754-2019
      DOI: 10.1109/IEEESTD.2019.8766229
  C:
    target: https://www.iso.org/standard/74528.html
    title: Information technology — Programming languages — C
    author:
    - org: International Organization for Standardization
    date: 2018-06
    seriesinfo:
      ISO/IEC: 9899:2018
    refcontent:
    - Fourth Edition
    annotation: The text of the standard is also available via https://www.open-std.org/jtc1/sc22/wg14/www/docs/n2310.pdf
  Cplusplus:
    target: https://www.iso.org/standard/79358.html
    title: Programming languages — C++
    author:
    - org: International Organization for Standardization
    date: 2020-12
    seriesinfo:
      ISO/IEC: 14882:2020
    refcontent:
    - Sixth Edition
    annotation: The text of the standard is also available via https://isocpp.org/files/papers/N4860.pdf
informative:
  RFC4648: base
  RFC9290:
  STD90: json
  RFC9165: controls
  I-D.ietf-cbor-update-8610-grammar: cddlupd

--- abstract

The Concise Binary Object Representation, CBOR (STD 94, RFC 8949), [^abs1-]

[^abs1-]: defines a "diagnostic notation" in order to
    be able to converse about CBOR data items without having to resort to
    binary data.

[^abs3-]: This document specifies how to add application-oriented extensions to
    the diagnostic notation.  It then defines two such extensions for
    text representations of epoch-based date/times and of IP addresses
    and prefixes

​[^abs3-] (RFC 9164).

[^abs4-]: A few further additions close some gaps in usability.
     To facilitate tool interoperation, this document
     specifies a formal ABNF definition for extended diagnostic notation (EDN)
     that accommodates application-oriented literals.

[^abs4-]


--- middle

Introduction        {#intro}
============

For the Concise Binary Object Representation, CBOR,
{{Section 8 of RFC8949@-cbor}} in conjunction with {{Appendix G of -cddl}}
[^abs1-]
Diagnostic notation syntax is based on JSON, with extensions
for representing CBOR constructs such as binary data and tags.
[^abs2-]

[^abs2-]: (Standardizing this together with the actual interchange format does
    not serve to create another interchange format, but enables the use of
    a shared diagnostic notation in tools for and in documents about CBOR.)

[^abs3-] {{-iptag}}.

[^abs4-] (See {{grammar}} for an overall ABNF grammar as well as the
ABNF definitions in {{app-grammars}} for grammars for both the
byte string presentations predefined in {{-cbor}} and the application-extensions).

In addition, this document finally registers a media type identifier
and a content-format for CBOR diagnostic notation.  This does not
elevate its status as an interchange format, but recognizes that
interaction between tools is often smoother if media types can be used.

{:aside}
>
> Examples in RFCs often do not use media type identifiers, but
> special sourcecode type names that are allocated
> in <https://www.rfc-editor.org/materials/sourcecode-types.txt>.
> At the time of writing, this resource lists four sourcecode type
> names that can be used in RFCs for including CBOR data items and
> CBOR-related languages:
>
> * `cbor` (which is actually not useful, as CBOR is a binary format
>   and cannot be used in textual examples in an RFC),
> * `cbor-diag` (which is another name for EDN, as defined in the
>   present document),
> * `cbor-pretty` (which is a possibly annotated and pretty-printed
>   hexdump of an encoded CBOR data item, along the lines of the
>   grammar of {{h-grammar}}, as used for instance for some the examples
>   in {{Section A.3 of RFC9290}}), and
> * `cddl` (which is used for the Concise Data Definition Language,
>   CDDL, see {{terminology}} below).

## Terminology

{{Section 8 of RFC8949@-cbor}} defines the original CBOR diagnostic notation,
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
include the original notation from {{Section 8 of RFC8949@-cbor}} as well as the
extensions from {{Appendix G of -cddl}}, {{Section 4.2 of -seq}}, and the
present document.
However, we stick to the abbreviation "_EDN_" as it has become quite
popular and is more sharply distinguishable from other meanings than
"DN" would be.

In a similar vein, the term "ABNF" in this document refers to the
language defined in {{-abnf}} as extended in {{-abnfcs}}, where the
"characters" of {{Section 2.3 of RFC5234@-abnf}} are Unicode scalar values.

The term "CDDL" (Concise Data Definition Language) refers to the data
definition language defined in
{{-cddl}} and its registered extensions (such as those in {{-controls}}), as
well as {{-cddlupd}}.
Additional information about the relationship between the two
languages EDN and CDDL is captured in the informative {{edn-and-cddl}}.

{::boilerplate bcp14-tagged}

## (Non-)Objectives of this Document

{{Section 8 of RFC8949@-cbor}} states the objective of defining a
human-readable diagnostic notation with CBOR.
In particular, it states:

{:quote}
> All actual interchange always happens in the binary format.

One important application of EDN is the notation of CBOR data for
humans: in specifications, on whiteboards, and for entering test data.
A number of features, such as comments in string literals, are mainly
useful for people-to-people communication via EDN.
Programs also often output EDN for diagnostic purposes, such as in
error messages or to enable comparison (including generation of diffs
via tools) with test data.

For comparison with test data, it is often useful if different
implementations generate the same (or similar) output for the same
CBOR data items.
This is comparable to the objectives of deterministic serialization
for CBOR data items themselves ({{Section 4.2 of RFC8949@-cbor}}).
However, there are even more representation variants in EDN than in
binary CBOR, and there is little point in specifically endorsing a
single variant as "deterministic" when other variants may be more
useful for human understanding, e.g., the `<< >>` notation as
opposed to `h''`; an EDN generator may have quite a few options
that control what presentation variant is most desirable for the
application that it is being used for.

Because of this, a deterministic representation is not defined for
EDN, and there is no expectation for "roundtripping" from EDN to
CBOR and back, i.e., for an ability
to convert EDN to binary CBOR and back to EDN while achieving exactly
the same result as the original input EDN — the original EDN possibly
was created by humans or by a different EDN generator.

However, there is a certain expectation that EDN generators can be
configured to some basic output format, which:

* looks like JSON where that is possible;
* inserts encoding indicators only where the binary form differs from
  preferred encoding;
* uses hexadecimal representation (`h''`) for byte strings, not
  `b64''` or embedded CBOR (`<<>>`);
* does not generate elaborate blank space (newlines, indentation) for
  pretty-printing, but does use common blank spaces such as after `,`
  and `:`.

Additional features such as ensuring deterministic map ordering
({{Section 4.2 of RFC8949@-cbor}}) on output, or even deviating from the basic
configuration in some systematic way, can further assist in comparing
test data.
Information obtained from a CDDL model can help in choosing
application-oriented literals or specific string representations such
as embedded CBOR or `b64''` in the appropriate places.

Application-Oriented Extension Literals
=======================================

This document extends the syntax used in diagnostic notation for byte
string literals to also be available for application-oriented extensions.

As per {{Section 8 of RFC8949@-cbor}}, the diagnostic notation can notate byte
strings in a number of {{-base}} base encodings, where the encoded text
is enclosed in single quotes, prefixed by an identifier (»h« for
base16, »b32« for base32, »h32« for base32hex, »b64« for base64 or
base64url).

This syntax can be thought to establish a name space, with the names
"h", "b32", "h32", and "b64" taken, but other names being unallocated.
The present specification defines additional names for this namespace,
which we call *application-extension identifiers*.
For the quoted string, the same rules apply as for byte strings.
In particular, the escaping rules that were adapted from JSON strings
are applied
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

An application-extension (such as `dt`) MAY also define the meaning of
a variant of the application-extension identifier where each
lower-case character is replaced by its upper-case counterpart (such
as `DT`), for building an application-oriented extension literal using
that all-uppercase variant as the prefix of a single-quoted string.

As a convention for such definitions, using the all-uppercase variant
implies making use of a tag appropriate for this application-oriented
extension (such as tag number 1 for `DT`).

Examples for application-oriented extensions to CBOR diagnostic
notation can be found in the following sections.


The "dt" Extension {#dt}
------------------

The application-extension identifier "dt" is used to notate a
date/time literal that can be used as an Epoch-Based Date/Time as per
{{Section 3.4.2 of RFC8949@-cbor}}.

The text of the literal is a Standard Date/Time String as per
{{Section 3.4.1 of RFC8949@-cbor}}.

The value of the literal is a number representing the result of a
conversion of the given Standard Date/Time String to an Epoch-Based
Date/Time.
If fractional seconds are given in the text (production
`time-secfrac` in {{abnf-grammar-dt}}), the value is a
floating-point number; the value is an integer number otherwise.
In the all-upper-case variant of the app-prefix, the value is enclosed
in a tag number 1.

As an example, the CBOR diagnostic notation

~~~ cbor-diag
dt'1969-07-21T02:56:16Z',
dt'1969-07-21T02:56:16.5Z',
DT'1969-07-21T02:56:16Z'
~~~

is equivalent to

~~~ cbor-diag
-14159024,
-14159023.5,
1(-14159024)
~~~

See {{dt-grammar}} for an ABNF definition for the content of `dt` literals.


The "ip" Extension {#ip}
------------------

The application-extension identifier "ip" is used to notate an IP
address literal that can be used as an IP address as per {{Section 3 of
-iptag}}.

The text of the literal is an IPv4address or IPv6address as per
{{Section 3.2.2 of -uri}}.

With the lower-case app-string `ip`, the value of the literal is a
byte string representing the binary IP address.
With the upper-case app-string `IP`, the literal is such a byte string
tagged with tag number 54, if an IPv6address is used, or tag number
52, if an IPv4address is used.

As an additional case, the upper-case app-string `IP''` can be used
with a prefix such as `2001:db8::/56` or `192.0.2.0/24`, with the equivalent tag as its value.
(Note that {{-iptag}} representations of address prefixes need to
implement the truncation of the address byte string as described in
{{Section 4.2 of -iptag}}; see example below.)
For completeness, the lower-case variant `ip'2001:db8::/56'` or  `ip'192.0.2.0/24'` stands for
an unwrapped `[56,h'20010db8']` or `[24,h'c00002']`; however, in this case the information
on whether an address is IPv4 or IPv6 often needs to come from the context.

Note that there is no direct representation of an address combined
with a prefix length; this can be represented as
`52([ip'192.0.2.42',24])`, if needed.

Examples: the CBOR diagnostic notation

~~~ cbor-diag
ip'192.0.2.42',
IP'192.0.2.42',
IP'192.0.2.0/24',
ip'2001:db8::42',
IP'2001:db8::42',
IP'2001:db8::/64'
~~~

is equivalent to

~~~ cbor-diag
h'c000022a',
52(h'c000022a'),
52([24,h'c00002']),
h'20010db8000000000000000000000042',
54(h'20010db8000000000000000000000042'),
54([64,h'20010db8'])
~~~

See {{ip-grammar}} for an ABNF definition for the content of `ip` literals.



Stand-in Representations in Binary CBOR {#stand-in}
=======================================

In some cases, an EDN consumer cannot construct actual CBOR items that
represent the CBOR data intended for eventual interchange.
This document defines stand-in representation for two such cases:

* The EDN consumer does not know (or does not implement) an
  application-extension identifier used in the EDN document
  ({{unknown}}) but wants to preserve the information for a later
  processor.

* The generator of some EDN intended for human consumption (such as in
  a specification document) may not want to include parts of the final
  data item, destructively replacing complete subtrees or possibly
  just parts of a lengthy string by _elisions_ ({{elision}}).

Handling unknown application-extension identifiers {#unknown}
--------------------------------------------------

When ingesting CBOR diagnostic notation, any
application-oriented extension literals are usually decoded and
transformed into the corresponding data item during ingestion.
If an application-extension is not known or not implemented by the
ingesting process, this is usually an error and processing has to
stop.

However, in certain cases, it can be desirable to exceptionally carry an
uninterpreted application-oriented extension literal in an ingested
data item, allowing to postpone its decoding to a specific later
stage of ingestion.

This specification defines a CBOR Tag for this purpose:
The Diagnostic Notation Unresolved Application-Extension Tag, tag
number CPA999 ({{iana-standin}}).
The content of this tag is an array of two text strings: The
application-extension identifier, and the (escape-processed) content
of the single-quoted string.
For example, `dt'1969-07-21T02:56:16Z'` can be provisionally represented as
`/CPA/ 999(["dt", "1969-07-21T02:56:16Z"])`.

[^cpa]

[^cpa]: RFC-Editor: This document uses the CPA (code point allocation)
      convention described in [I-D.bormann-cbor-draft-numbers].  For
      each usage of the term "CPA", please remove the prefix "CPA"
      from the indicated value and replace the residue with the value
      assigned by IANA; perform an analogous substitution for all other
      occurrences of the prefix "CPA" in the document.  Finally,
      please remove this note.

Handling information deliberately elided from an EDN document {#elision}
-------------------------------------------------------------

When using EDN for exposition in a document or on a whiteboard, it is
often useful to be able to leave out parts of an EDN document that are
not of interest at that point of the exposition.

To facilitate this, this specification
supports the use of an _ellipsis_ (notated as three or more dots
in a row, as in `...`) to indicate parts of an EDN document that have
been elided (and therefore cannot be reconstructed).

Upon ingesting EDN as a representation of a CBOR data item for further
processing, the occurrence of an ellipsis usually is an error and
processing has to stop.

However, it is useful to be able to process EDN documents with
ellipses in the automation scripts for the documents using them.
This specification defines a CBOR Tag that can be used in the ingestion
for this purpose:
The Diagnostic Notation Ellipsis Tag, tag number CPA888 ({{iana-standin}}).
The content of this tag either is

1. null (indicating a data item entirely replaced by an ellipsis), or it is
2. an array, the elements of which are alternating between fragments
   of a string and the actual elisions, represented as ellipses
   carrying a null as content.

Elisions can stand in for entire subtrees, e.g. in:

~~~ cbor-diag
[1, 2, ..., 3]
,
{ "a": 1,
  "b": ...,
  ...: ...
}
~~~

A single ellipsis (or key/value pair of ellipses) can imply eliding
multiple elements in an array (members in a map); if more detailed
control is required, a data definition language such as CDDL can be
employed.
(Note that the stand-in form defined here does not allow multiple
key/value pairs with an ellipsis as a key: the CBOR data item would
not be valid.)

Subtree elisions can be represented in a CBOR data item by using
`/CPA/888(null)` as the stand-in:

~~~ cbor-diag
[1, 2, 888(null), 3]
,
{ "a": 1,
  "b": 888(null),
  888(null): 888(null)
}
~~~

Elisions also can be used as part of a (text or byte) string:

~~~ cbor-diag
{ "contract": "Herewith I buy" ... "gned: Alice & Bob",
  "signature": h'4711...0815',
}
~~~

The example "contract" uses string concatenation as per {{Section G.4
of -cddl}}, extending that by allowing ellipses; while the example
"signature" uses special syntax that allows the use of ellipses
between the bytes notated _inside_ `h''` literals.

String elisions can be represented in a CBOR data item by a stand-in
that wraps an array of string fragments alternating with ellipsis
indicators:

~~~ cbor-diag
{ "contract": /CPA/888(["Herewith I buy", 888(null),
                        "gned: Alice & Bob"]),
  "signature": 888([h'4711', 888(null), h'0815']),
}
~~~

Note that the use of elisions is different from "commenting out" EDN
text, e.g.


~~~ cbor-diag
{ "contract": "Herewith I buy" /.../ "gned: Alice & Bob",
  "signature": h'4711/.../0815',
  # ...: ...
}
~~~

The consumer of this EDN will ignore the comments and therefore will
have no idea after ingestion that some information has been elided;
validation steps may then simply fail instead of being informed about
the elisions.


IANA Considerations {#sec-iana}
===================

[^to-be-removed]

[^to-be-removed]: RFC Editor: please replace RFC-XXXX with the RFC
    number of this RFC, \[IANA.cbor-diagnostic-notation] with a
    reference to the new registry group, and remove this note.


## CBOR Diagnostic Notation Application-extension Identifiers Registry {#appext-iana}

IANA is requested to create an "Application-Extension Identifiers"
registry in a new "CBOR Diagnostic Notation" registry group
\[IANA.cbor-diagnostic-notation], with the policy "expert review"
({{Section 4.5 of RFC8126@-ianacons}}).

The experts are instructed to be frugal in the allocation of
application-extension identifiers that are suggestive of generally applicable semantics,
keeping them in reserve for application-extensions that are likely to enjoy wide
use and can make good use of their conciseness.
The expert is also instructed to direct the registrant to provide a
specification ({{Section 4.6 of RFC8126@-ianacons}}), but can make exceptions,
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
  entry in the registry can have the same application-extension identifier.

Description:
: a brief description

Change Controller:
: (see {{Section 2.3 of RFC8126@-ianacons}})

Reference:
: a reference document that provides a description of the
  application-extension identifier


The initial content of the registry is shown in {{tab-iana}}; all
initial entries have the Change Controller "IETF".

| Application-extension Identifier | Description                     | Reference |
|----------------------------------|---------------------------------|-----------|
| h                                | Reserved                        | RFC8949   |
| b32                              | Reserved                        | RFC8949   |
| h32                              | Reserved                        | RFC8949   |
| b64                              | Reserved                        | RFC8949   |
| dt                               | Date/Time                       | RFC-XXXX   |
| ip                               | IP Address/Prefix               | RFC-XXXX   |
{: #tab-iana title="Initial Content of Application-extension
Identifier Registry"}


## Encoding Indicators {#reg-ei}

IANA is requested to create an "Encoding Indicators"
registry in the newly created "CBOR Diagnostic Notation" registry group
\[IANA.cbor-diagnostic-notation], with the policy "specification required"
({{Section 4.6 of RFC8126@-ianacons}}).

The experts are instructed to be frugal in the allocation of
encoding indicators that are suggestive of generally applicable semantics,
keeping them in reserve for encoding indicator registrations that are likely to enjoy wide
use and can make good use of their conciseness.
If the expert becomes aware of encoding indicators that are deployed and
in use, they may also solicit a specification and initiate a registration on their own if
they deem such a registration can avert potential future collisions.
{: #de-instructions-ei}

Each entry in the registry must include:

{:vspace}
Encoding Indicator:
: an ASCII {{-ascii}} string that starts with an underscore letter and
  can contain zero or more underscores, letters and digits after that
  (`_[_A-Za-z0-9]*`). No other entry in the registry can have the same
  Encoding Indicator.

Description:
: a brief description.
  This description may employ an abbreviation of the form `ai=`nn,
  where nn is the numeric value of the field _additional information_, the
  low-order 5 bits of the initial byte (see {{Section 3 of RFC8949@-cbor}}).

Change Controller:
: (see {{Section 2.3 of RFC8126@-ianacons}})

Reference:
: a reference document that provides a description of the
  application-extension identifier


The initial content of the registry is shown in {{tab-iana-ei}}; all
initial entries have the Change Controller "IETF".

| Encoding Indicator | Description                        | Reference        |
|--------------------|------------------------------------|------------------|
| _                  | Indefinite Length Encoding (ai=31) | RFC8949, RFC-XXXX |
| _i                 | ai=0 to ai=23                      | RFC-XXXX          |
| _0                 | ai=24                              | RFC8949, RFC-XXXX |
| _1                 | ai=25                              | RFC8949, RFC-XXXX |
| _2                 | ai=26                              | RFC8949, RFC-XXXX |
| _3                 | ai=27                              | RFC8949, RFC-XXXX |
{: #tab-iana-ei title="Initial Content of Encoding Indicator Registry"}




## Media Type

IANA is requested to add the following Media-Type to the "Media Types"
registry {{IANA.media-types}}.

| Name            | Template                    | Reference              |
| cbor-diagnostic | application/cbor-diagnostic | RFC-XXXX, {{media-type}} |
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
: LIMITED USE

Restrictions on usage:
: CBOR diagnostic notation represents CBOR data items, which are the
  format intended for actual interchange.
  The media type application/cbor-diagnostic is intended to be used
  within documents about CBOR data items, in diagnostics for human
  consumption, and in other representations of CBOR data items that
  are necessarily text-based such as in configuration files or other
  data edited by humans, often under source-code control.

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
| application/cbor-diagnostic | -              | TBD1 | RFC-XXXX  |
{: align="left" title="New Content-Format"}

TBD1 is to be assigned from the space 256..9999, according to the
procedure "IETF Review or IESG Approval", preferably a number less
than 1000.

## Stand-in Tags {#iana-standin}

[^cpa]

In the "CBOR Tags" registry {{-tags}}, IANA is requested to assign the
tags in {{tab-tag-values}} from the "specification required" space
(suggested assignments: 888 and 999), with the present document as the
specification reference.

| Tag    | Data Item     | Semantics                                            | Reference |
| CPA888 | null or array | Diagnostic Notation Ellipsis                         | RFC-XXXX  |
| CPA999 | array         | Diagnostic Notation<br>Unresolved Application-Extension | RFC-XXXX  |
{: #tab-tag-values cols='r l l' title="Values for Tags"}


Security considerations {#seccons}
=======================

The security considerations of {{-cbor}} and {{-cddl}} apply.

--- back

ABNF Definitions {#grammars}
================

This appendix is normative.

It collects grammars in ABNF form ({{-abnf}} as extended in
{{-abnfcs}}) that serve to define the syntax of EDN and some
application-oriented literals.

Implementation note: The ABNF definitions in this appendix are
intended to be useful in a Parsing Expression Grammar (PEG) parser
interpretation (see {{Appendix A
of -cddl}} for an introduction into PEG).

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
  usual decimal notation.  Note that the grammar now allows `3.` for
  `3.0` and `.3` for `0.3` (also for hexadecimal floating point
  below); implementers are advised that some platform numeric parsers
  accept only a subset of the floating point syntax in this document
  and may require some preprocessing to use here.
* `basenumber` stands for an integer in the usual base 16/hexadecimal
  ("0x"), base 8/octal ("0o"), or base 2/binary ("0b") notation, unless the
  optional part containing a "p" is present, in which case it stands
  for a floating point number in the usual hexadecimal notation (which
  uses a mantissa in hexadecimal and an exponent in decimal notation,
  see Section 5.12.3 of {{IEEE754}}, Section 6.4.4.2 of {{C}}, or Section
  5.13.4 of {{Cplusplus}}; floating-suffix/floating-point-suffix from
  the latter two is not used here).
* `spec` stands for an encoding indicator.

  (In the following, an abbreviation of the form `ai=`nn gives nn as
  the numeric value of the field _additional information_, the low-order 5
  bits of the initial byte: see {{Section 3 of RFC8949@-cbor}}.)

  As per {{Section 8.1 of RFC8949@-cbor}}:

  * an underscore `_` on its own stands
    for indefinite length encoding (`ai=31`, only available behind the
    opening brace/bracket for `map` and `array`: strings have a special
    syntax `streamstring` for indefinite length encoding except for the
    special cases ''_ and ""_), and
  * `_0` to `_3` stand for `ai=24` to `ai=27`, respectively.

  Surprisingly, {{Section 8.1 of RFC8949@-cbor}} does not address `ai=0` to
  `ai=23` — the assumption seems to be that preferred serialization
  ({{Section 4.1 of RFC8949@-cbor}}) will be used when converting CBOR
  diagnostic notation to an encoded CBOR data item, so leaving out the
  encoding indicator for a data item with a preferred serialization
  will implicitly use `ai=0` to `ai=23` if that is possible.
  The present specification allows to make this explicit:

  * `_i` ("immediate") stands for encoding with `ai=0` to `ai=23`.

  While no pressing use for further values for encoding indicators
  comes to mind, this is an extension point for EDN; {{reg-ei}} defines
  a registry for additional values.

* `string` and the rules preceding it in the same block realize both
  the representation of strings that are split up into multiple chunks
  ({{Section G.4 of -cddl}}) and the use of ellipses to represent elisions
  ({{elision}}).  The semantic processing of these rules is relatively
  complex:
  * A single `...` is a general ellipsis, which can stand for any data
    item.
  * An ellipsis can be surrounded (on one or both sides) by string
    chunks, the result is a CBOR tag number CPA888 that contains an
    array with joined together spans of such chunks plus the ellipses
    represented by `888(null)`.
  * A simple sequence of string chunks is simply joined together.
    In both cases of joining strings, the rules of {{Section G.4 of
    -cddl}} need to be followed; in particular, if a text string
    results from the joining operation, that result needs to be valid
    UTF-8.
  * Some of the strings may be app-strings.
    If the type of the app-string is an actual string, joining of
    chunked strings occurs as with directly notated strings; otherwise
    the occurrence of more than one app-string or an app-string
    together with a directly notated string cannot be processed.

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
such as `h''`, `h'0815'`, or `h'/head/ 63 /contents/ 66 6f 6f'`
(another representation of `<< "foo" >>`), is described by the ABNF in {{abnf-grammar-h}}.
This syntax accommodates both lower case and upper case hex digits, as
well as blank space (including comments) around each hex digit.

~~~ abnf
app-string-h    = S *(HEXDIG S HEXDIG S / ellipsis S)
                  ["#" *non-lf]
ellipsis        = 3*"."
HEXDIG          = DIGIT / "A" / "B" / "C" / "D" / "E" / "F"
DIGIT           = %x30-39 ; 0-9
blank           = %x09 / %x0A / %x0D / %x20
non-slash       = blank / %x21-2e / %x30-10FFFF
non-lf          = %x09 / %x0D / %x20-D7FF / %xE000-10FFFF
S               = *blank *(comment *blank )
comment         = "/" *non-slash "/"
                / "#" *non-lf %x0A
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
                  ["#" *inon-lf]
b64dig          = ALPHA / DIGIT / "-" / "_" / "+" / "/"
B               = *iblank *(icomment *iblank)
iblank          = %x0A / %x20  ; Not HT or CR (gone)
icomment        = "#" *inon-lf %x0A
inon-lf         = %x20-D7FF / %xE000-10FFFF
ALPHA           = %x41-5a / %x61-7a
DIGIT           = %x30-39
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


### ip: ABNF Definition of Textual Representation of an IP Address {#ip-grammar}

The syntax of the content of `ip` literals can be described by the
ABNF for `IPv4address` and `IPv6address` in {{Section 3.2.2 of -uri}},
as included in slightly updated form in {{abnf-grammar-ip}}.

~~~ abnf
app-string-ip = IPaddress ["/" uint]

IPaddress     = IPv4address
              / IPv6address

; ABNF from RFC 3986, re-arranged for PEG compatibility:

IPv6address   =                            6( h16 ":" ) ls32
              /                       "::" 5( h16 ":" ) ls32
              / [ h16               ] "::" 4( h16 ":" ) ls32
              / [ h16 *1( ":" h16 ) ] "::" 3( h16 ":" ) ls32
              / [ h16 *2( ":" h16 ) ] "::" 2( h16 ":" ) ls32
              / [ h16 *3( ":" h16 ) ] "::"    h16 ":"   ls32
              / [ h16 *4( ":" h16 ) ] "::"              ls32
              / [ h16 *5( ":" h16 ) ] "::"              h16
              / [ h16 *6( ":" h16 ) ] "::"

h16           = 1*4HEXDIG
ls32          = ( h16 ":" h16 ) / IPv4address
IPv4address   = dec-octet "." dec-octet "." dec-octet "." dec-octet
dec-octet     = "25" %x30-35         ; 250-255
              / "2" %x30-34 DIGIT    ; 200-249
              / "1" 2DIGIT           ; 100-199
              / %x31-39 DIGIT        ; 10-99
              / DIGIT                ; 0-9

HEXDIG        = DIGIT / "A" / "B" / "C" / "D" / "E" / "F"
DIGIT         = %x30-39 ; 0-9
DIGIT1        = %x31-39 ; 1-9
uint          = "0" / DIGIT1 *DIGIT
~~~
{: #abnf-grammar-ip sourcecode-name="cbor-edn-ip.abnf"
title="ABNF Definition of Textual Representation of an IP Address"}


EDN and CDDL
============

This appendix is informative.

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
  : ~~~
    98([h'', # empty encoded protected header
        {},  # empty unprotected header
        ...  # rest elided here
       ])
    ~~~

  CDDL:
  : `COSE_Sign_Tagged = #6.98(COSE_Sign)`

* Separator character.  Like JSON, EDN requires commas as separators
  between array elements and map members (EDN also allows, but does
  not require, a trailing comma before the closing bracket/brace,
  enabling an easier to maintain "terminator" style of their use).
  CDDL's comma separators in these contexts (CDDL groups) are entirely
  optional
  (and actually are terminators, which together with their optionality
  allows them to be used like separators as well, or even not at all).

* Embedded CBOR.  EDN has a special syntax to describe the content of
  byte strings that are encoded CBOR data items.  CDDL can specify
  these with a control operator, which looks very different.

  EDN:
  : ~~~
    98([<< {/alg/ 1: -7 /ECDSA 256/} >>, # == h'a10126'
        ...                              # rest elided here
       ])
    ~~~

  CDDL:
  : `serialized_map = bytes .cbor header_map`



Acknowledgements
================
{: numbered="no"}

The concept of application-oriented extensions to diagnostic notation,
as well as the definition for the "dt" extension, were inspired by the
CoRAL work by Klaus Hartke.

<!--  LocalWords:  dedenting dedented
 -->
