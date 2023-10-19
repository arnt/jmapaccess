---
stand_alone: true
ipr: trust200902
cat: std
submissiontype: IETF
area: Applications
wg: EXTRA

docname: draft-ietf-extra-jmapaccess

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
intended for clients that want to migrate gradually to JMAP or use
JMAP extensions within an IMAP client.

--- middle

# Introduction

An IMAP server can declare that the messages in its mailstore are also
available via JMAP. For simplicity, only a complete equivalence is
supported (the same set of messages are available via both IMAP and
JMAP).

This document also defines a way to provide debugging information that
can be forwarded to client developers without privacy concerns, which
is used by JMAPACCESS but can also be used by others.

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
message may have been deleted between the two accesses.

When the server processes the client's LOGIN/AUTHENTICATE command and
enters Authenticated state, the server considers the way the client
authenticated. If the IMAP server can infer from the client's
authentication process that its credentials suffice to authenticate
via JMAP, then the server MUST also send a JMAPACCESS response code
containing a link to the JMAP server.

If the credentials might not succeed with the JMAP server, then the
server SHOULD send an untagged OK response with a DEBUGGING response
code with a message to help client developers understand why this
authentication would not work with the JMAP server.

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

The DEBUGGING response code asserts that the client may forward the
string argument to the client maintainers without privacy concerns,
and that the argument relates to a currently active command. The
argument MUST NOT contain any message contents or other personal
information. The DEBUGGING response code is generically named because
it is expected to be useful for more than just JMAPACCESS.

The formal syntax in {{RFC9051}} is extended thus:

resp-code-jmapaccess = "JMAPACCESS" SP (atom / quoted)

resp-code-debugging = "DEBUGGING" SP (atom / quoted)

resp-text-code =/ resp-code-jmapaccess / resp-code-debugging

The syntax in {{RFC3501}} is extended similarly (this extension may be
used with IMAP4rev1 as well as IMAP4rev2).

# Examples {#Examples}

Lines sent by the client are preceded by C:, lines sent by the server
by S:. Each example starts with the IMAP banner issued by the server
on connection, and generally abbreviates the capability lists to
what's required by the example itself.

Real connections use longer capability lists, much longer AUTHENTICATE
arguments and of course use TLS. These examples focus on JMAPACCESS,
though.

Example 1. A client connects, sees that SASL OAUTH is available, and
authenticates in that way.

S: * OK [CAPABILITY IMAP4rev1 AUTH=OAUTHBEARER SASL-IR] example1<br>
C: 1 AUTHENTICATE OAUTHBEARER bixhPXVzZ...QEB

The server processes the command successfully. It knows that the
client used Oauth, and that it and its JMAP alter ego use the same
Oauth backend subsystem. Because of that it infers that the (next)
access token is just as usable via JMAP as via IMAP. It issues a
JMAPACCESS response code in its reply:

S: 1 OK [JMAPACCESS https://example.com/jmap] done

SASL OAUTH is specified by {{RFC7628}}, and the argument in this
example is abbreviated from the more realistic length used in RFC7628.

Example 2. A client connects, sees no SASL method it recognises, and
issues a LOGIN command.

S: * OK [CAPABILITY IMAP4rev2] example2<br>
C: 2 LOGIN "arnt" "trondheim"

The server sees that the password is accepted, knows that it and its
JMAP alter ego use the same password database, and issues a JMAPACCESS
response code:

S: * OK [JMAPACCESS "https://example.com/.s/[jmap]"] For JMAP access
S: 2 OK done

The URL is quoted since the ] character must be quoted. The URL uses
the same quoting rules as most other IMAP strings.

Example 3. A client connects, sees no SASL method it recognises, and
issues a LOGIN command with a correct password.

S: * OK [CAPABILITY IMAP4rev1 IMAP4rev2] example3<br>
C: 3 LOGIN "arnt" "trondheim"

The server operator has decided to disable password use with JMAP, but
allow it for a while with IMAP to cater to older clients. The server
issues a DEBUGGING response code in its reply:

S: 3 OK [DEBUGGING "Cleartext passwords are being disabled"] done

The message is quoted since it contains spaces. The message uses the
same quoting rules as most other IMAP strings.

Example 4. A client connects, sees no SASL method it recognises, and
issues a LOGIN command. Its password is incorrect.

S: * OK [CAPABILITY IMAP4rev2 AUTH=GSS] example4<br>
C: 4 LOGIN "arnt" "oslo"

The server does not enter Authenticated state, so nothing requires it
to issue either JMAPACCESS or DEBUGGING. It replies curtly:

S: 4 NO done

# IANA Considerations {#IANA}

The IANA is requested to add the JMAPACCESS and DEBUGGING response
codes to the IMAP Response Codes registry, with this document as
reference.

# Security Considerations {#Security}

This document suggests that servers reveal to clients why they cannot
authenticate to the JMAP server. One normally does not want to reveal
anything about why a client cannot authenticate, for fear of giving
useful information to an intruder.

However, in this case the client has already authenticated via
IMAP. By doing so the client already gained access to all of the same
mail. The authors believe that the debugging value of sending the
DEBUGGING response code far outweighs its security concerns.

DEBUGGING also reveals infomation about the connection state to the
client developer. It's essential that this never includes information
about the user or the email, such as for example the name of a related
mailbox or a command argument that might contain personal information
due to a client bug.

--- back

