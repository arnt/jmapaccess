---
stand_alone: true
ipr: trust200902
cat: std
submissiontype: IETF
area: Applications
wg: EXTRA

docname: draft-ietf-extra-jmapaccess-02

title: The JMAPACCESS Extension for IMAP
abbrev: IMAP JMAPACCESS
lang: en
kw:
  - IMAP
  - JMAP
author:
- name: Arnt Gulbrandsen
  org: ICANN
  street: 6 Rond Point Schumann, Bd. 1
  city: Brussels
  code: 1040
  country: Belgium
  email: arnt@gulbrandsen.priv.no
  uri: https://icann.org/ua
- name: Bron Gondwana
  org: Fastmail
  street: Level 2, 114 William St.
  city: Melbourne VIC
  code: 3000
  country: Australia
  email: brong@fastmailteam.com
  uri: https://fastmail.com


normative:
  RFC3501:
  RFC8474:
  RFC9051:

informative:
  RFC2444:
  RFC6530:
  RFC6855:
  RFC6857:
  RFC6858:
  RFC6857:
  RFC6858:
  RFC7628:
  RFC8620:

--- abstract

This document defines an IMAP extension to let clients know that the
messages in this IMAP server are also available via JMAP, and how. It is
intended for clients that want to migrate gradually to JMAP.

--- middle

# Introduction

A few IMAP client maintainers have asked for ways to use features that
are available in JMAP without having to drop their expensively tested
IMAP code.

This document provides a server with a way to declare that the
messages in its mailstore are also available via JMAP. For simplicity,
only a complete equivalence is supported (the same set of messages are
available via both IMAP and JMAP).

# Requirements Language

{::boilerplate bcp14-tagged}

# Details

By advertising the JMAPACCESS capability, the server asserts that if a
mailbox or message has a particular object ID when accessed via either
IMAP or JMAP (see {{RFC3501}}, {{RFC9051}} and {{RFC8620}}), then the
same mailbox or message is accessible via the other protocol, and it
has the same ID.

The server MUST also advertise the OBJECTID extension, defined by
{{RFC8474}}. The JMAP session resource that allows access to the same
messages is called "the JMAP server" below.

This specification does not affect message lifetime: If a client
accesses a message via IMAP and half a second later via JMAP, then the
message may have been deleted.

When the server processes the client's LOGIN/AUTHENTICATE command and
enters Authenticated state, the server considers the way the client
authenticated. If the same authentication would work with the JMAP
server, then the server MUST also send an untagged OK response with a
JMAPACCESS response code containing a link to the JMAP server.

If the authentication would not succeed with the JMAP server, then the
server SHOULD send an untagged OK response with a DEBUGGING response
code and some human-readable text to help client developers understand
why this authentication would not work with the JMAP server.

Some authentication methods use tokens that change depending on time
or sequence. One-time passwords (see {{RFC2444}}) and Oauth (see
{{RFC7628}}) are good examples. In these cases, JMAPACCESS requires
that this server and the JMAP server use the same sequence. To take
Oauth as an example, an access token is equally valid with both
protocols, no matter which server issued it.

Servers are encouraged to report the same message flags and other data
via both protocols, as far as possible.

This specification does not require mailboxes to have the same name in
IMAP and JMAP, even if they share mailbox ID. However, the JMAP
specification regulates that, in the text about the name and role
properties in {{RFC8620}} section 2.

Note that all JMAP servers support internationalized email addresses
(see {{RFC6530}}).  If this IMAP server does not, or the IMAP client
does not issue ENABLE UTF8=ACCEPT (see {{RFC6855}}), then there is a
possibility that the client receives accurate address fields via JMAP
and downgraded fields via IMAP (see (see {{RFC6857}} and {{RFC6858}}
for examples).

# The JMAPACCESS and DEBUGGING Response Codes

The JMAPACCESS response code is followed by a single link to a JMAP
session resource. The server/mailstore at that location is referenced
as "the JMAP server" in this document.

The DEBUGGING response code asserts that when used with a status
response, the client may safely forward the human-readable text to the
client maintainers. The human-readable text MUST NOT contain any
message contents or other personal information.

The formal syntax in {{RFC9051}} is extended thus:

resp-code-jmap = "JMAPACCESS" SP string / "DEBUGGING"

resp-text-code =/ resp-code-jmap

Note that the link cannot contain a "]" character.

The syntax in {{RFC3501}} is extended similarly (this extension may be
used with IMAP4rev1 as well as IMAP4rev2).


# IANA Considerations {#IANA}

The IANA is requested to add the JMAPACCESS and DEBUGGING response
codes to the IMAP Response Codes registry.


# Security Considerations {#Security}

This extension reveals to clients why they cannot auhenticate to the
JMAP server. One normally does not want to reveal anything about why a
client cannot authenticate, for fear of giving useful information to
an intruder.

However, in this case the client has already authenticated via
IMAP. By doing so the client already gained access to all of the same
mail. The authors believe that the debugging value of the OK response
far outweighs its security concerns.


--- back

