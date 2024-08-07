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

This can be useful for clients that have well-tested IMAP code and want to
use one or more JMAP features. For example, JMAP offers a way to be
notified about new incoming mail without having to maintain an open TCP
connection.

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
via JMAP, then the server MUST include a JMAPACCESS capability in any
capability list sent after that point. This includes the capability
list that some servers send immediately when authentication succeeds.

Servers are encouraged to report the same message flags and other data
via both protocols, as far as possible.

The JMAP specification ({{RFC8622}} sections 1.6 and 2) specifies how
mailbox IDs and names in JMAP relate to those in IMAP.

Note that all JMAP servers support internationalized email addresses
(see {{RFC6530}}).  If this IMAP server does not, or the IMAP client
does not issue ENABLE UTF8=ACCEPT (see {{RFC6855}}), then there is a
possibility that the client receives accurate address fields via JMAP
and downgraded fields via IMAP (see (see {{RFC6857}} and {{RFC6858}}
for examples). Issuing ENABLE UTF8=ACCEPT is a simple way to sidestep
the issue.

# The GETJMAPACCESS command and the JMAPACCESS response

The GETJMAPACCESS command requests that the server respond with the
session URL for the JMAP server that provides access to the same mail.

If such a JMAP server is known to this server, the server MUST respond
with an untagged JMAPACCESS response containing the JMAP server's
session resource followed
by a tagged OK response.

If such a JMAP server is not known, the server MUST respond with a
tagged BAD response (and MUST NOT include JMAPACCESS in the capability
list).

The JMAPACCESS response is followed by a single link to a JMAP session
resource, as described in {{RFC8620}} section 2. The server/mailstore
at that location is referenced as "the JMAP server" in this document.

The formal syntax in {{RFC9051}} is extended thus:

command-auth =/ "GETJMAPACCESS"

mailbox-data =/ resp-jmapaccess

resp-jmapaccess = "JMAPACCESS" SP quoted

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
access token is just as usable via JMAP as via IMAP. It includes a
JMAPACCESS capability in its reply (again, real capability lists are
much longer):

S: 1 OK [CAPABILITY IMAP4rev1 JMAPACCESS] done<br>
C: 1b GETJMAPACCESS<br>
S: * JMAPACCESS "https://example.com/jmap"<br>
S: 1b OK done

SASL OAUTH is specified by {{RFC7628}}, and the argument in this
example is abbreviated from the more realistic length used in RFC7628.

Example 2. A client connects, sees no SASL method it recognises, and
issues a LOGIN command.

S: * OK [CAPABILITY IMAP4rev2] example2<br>
C: 2 LOGIN "arnt" "trondheim"

The server sees that the password is accepted, knows that it and its
JMAP alter ego use the same password database, and issues a JMAPACCESS
capability:

S: * OK [CAPABILITY IMAP4rev2 JMAPACCESS] done<br>
S: 2 OK done<br>
C: 2b GETJMAPACCESS<br>
S: * JMAPACCESS "https://example.com/.s/[jmap]"<br>
S: 2b OK done

The URL uses the same quoting rules as most other IMAP strings.

Example 3. A client connects, sees no SASL method it recognises, and
issues a LOGIN command with a correct password.

S: * OK [CAPABILITY IMAP4rev1 IMAP4rev2] example3<br>
C: 3 LOGIN "arnt" "trondheim"

The server operator has decided to disable password use with JMAP, but
allow it for a while with IMAP to cater to older clients, so the login
succeeds, but there is no JMAPACCESS capability.

S: 3 OK done

Example 4. A client connects, sees no SASL method it recognises, and
issues a LOGIN command. Its password is incorrect.

S: * OK [CAPABILITY IMAP4rev2 AUTH=GSS] example4<br>
C: 4 LOGIN "arnt" "oslo"

The server does not enter Authenticated state, so nothing requires it
to mention JMAPACCESS. It replies curtly:

S: 4 NO done

# Security Considerations {#Security}

JMAPACCESS reveals to authenticated IMAP clients that they would be
able to authenticate via JMAP using the same credentials, and that the
object IDs match. One normally does not want to reveal anything at all
about why a client cannot authenticate, for fear of giving useful
information to an intruder.

However, in this case information is revealed to an authenticated
client, the revealed URL can usually be found via JMAP autodiscovery,
and an attacker would only need to try the credentials it has with an
autodiscovered JMAP URL (a matter of a second or two). Therefore, it
is believed that JMAPACCESS does not benefit an attacker noticeably,
and its value for migration far outweighs its risk.

# IANA Considerations {#IANA}

The IANA is requested to add the JMAPACCESS capability the IMAP
Capabilities registry, with this document as reference.

--- back
