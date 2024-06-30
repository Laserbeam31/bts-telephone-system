BTS VoIP Telephone System Documentation
=======================================

The telephone system has the following internal handsets. These are linked through a central phone server
to be able to ring each other directly as well as receive calls from the public phone network:

- Office: Internal extension `101`
- Drama Store: Internal extension `102`
- Shed: Internal extension `103`
- Proj: Internal extension `111`
- Edge Workshop: Internal extension `112`
- Underrake: Internal extension `113`
- Box200: Internal extension `114`

**The Office and Proj telephones can be dialled externally from any mobile/landline using the following
public telephone number: `(01225) 666076`.**

An interactive voice response (IVR) menu behind the public number allows callers to dial "1" for the 
Office, or "2" for the Proj.

An answering service is provided for incoming calls to the public number. This allows external callers 
to leave a voicemail if the Office is empty. Voicemail recoridngs are e-mailed to the committee using 
the _committee@bts-crew.com_ e-mail.

Introduction to VoIP telephone systems in general:
--------------------------------------------------

Traditionally, large institutions utilise a _private branch exchange_ (PBX), colloquially termed a
"switchboard" for their internal telephone systems. The role of the PBX is to allow internal handsets to
dial one another without needing to traverse the public phone network. The PBX also provides centralised
voicemail services for each handset. Additionally, the PBX takes in a small number of connections to
external telephone lines. This means that if an internal handset wishes to make an external call
(often indicated by prefixing a dialled number with a "9" digit), it can do so. Likewise, if an external
incoming call arrives from the public phone system, this can be routed to an internal handset(s).

Historically, telephone handsets would connect to the PBX using dedicated analogue wiring. This would carry
all telephone handset communications as a combination of baseband analogue audio and high-frequency signalling
tones. Corresponding analogue PBXs for such systems were large, specialised, and difficult to maintain.

However, with Ethernet networks commonplace in institutional environments, it is now far more common for
telephone handsets to connect to a PBX using _Voice over Internet Protocol_ (VoIP). This means that voice
traffic to/from handsets is digitally encoded and sent over the same Ethernet network as is used for
computer networking. Each VoIP-capable telephone handsets connects to an Ethernet port rather than to an
analogue line.

VoIP phones communicate with a corresponding VoIP-capable PBX. This is typically just an Ethernet-connected
computer/server running special software. Telephone-PBX communication is most commonly done over the
_Session Initiation Protocol_, or SIP. This is a network protocol which allows each telephone to "register" 
to the PBX using a pre-programmed username/password, and which can be used to signal call activity (e.g. 
"begin call", "end call", or "transfer call", etc.).

The BTS telephone handsets:
---------------------------

These are _Mitel_ 5312 telephones.

Whilst most telephone system configuration is carried out on the central PBX (see below), the telephones
require some basic configuration directly, in order to be able to connect to the PBX in the first place.
This configuration may be done by connecting a handset to a network, typing its IP address into a web
browser on a PC on the same network, and entering the following credentials when prompted:

- Username: `admin`
- Password: `5312`

The following configuration options are set locally per-handset via this web interface:

- SIP username: Used for connecting to the PBX
- SIP password: Used for connecting to the PBX
- Caller ID: Displayed to other local phones when making an outgoing call
- SIP server IP address(es): IP address used for connecting to the PBX (see below)

How the external telephone number works:
----------------------------------------

The external incoming number - (01225) 666076 - is provided by _Net-Telco_, a provider of public
VoIP-capable telephone numbers. The BTS PBX (see below) connects out using SIP over the public internet 
to Net-Telco to receive calls on this number.

Intuitively speaking, the PBX can be thought of as "appearing to Net-Telco as a VoIP handset". Since
all the local BTS handsets are connected to the internal PBX, Net-Telco only sees one single connection
from the BTS phone system - that from the PBX.

Since Net-Telco provides us with a public number, this number costs money. This cost, as of 2024, is
£10/yr (ex. VAT). This is considerably cheaper than the £100/yr charged by the University for a
university-provided public telephone number (not to mention that the University's telephone systems
would not allow Backstage to use its own PBX, in all likelihood). This cost must be paid to Net-Telco
every year (multiple years' worth of payment in one go is possible) if the number is to remain
operational.

Net-Telco provides an online portal for managing their VoIP numbers. BTS's account on this portal
has credentials which may be ascertained by asking a committee member (John Lucas e-mailed account
details to the whole of Committee in December 2023).

The BTS PBX server:
-------------------

The BTS handsets are all connected to the University's wired _Docking_ network (for all intents and
purposes, this may be regarded as "Eduroam on a wire"). Over this network connection, they establish SIP
connections to the BTS _PBX server_. This PBX server is a Linux VM on the SU server in a DDaT server room.

The PBX server has the following IP address: `138.38.11.60`. This is accessible from Docking and Eduroam.

Access to the PBX server is possible over SSH, for administrative purposes. Use the following credentials:

- Username: `su2bc`
- Password: `[REDACTED]`

The PBX server runs _Asterisk_. This is an incredibly powerful piece of open-source PBX software which
allows any Linux PC/VM to act as a VoIP-capable PBX, complete with support for incoming connections from
SIP telephones.

Asterisk is configured using a number of text-based configuration files in the `/etc/asterisk/`
directory. For our purposes, the following Asterisk configuration files are of interest:

- `pjsip.conf`: This contains a number of blocks of directives, each with a [...] header. Each section
  refers to the SIP settings used for receiving connections from each handset or for making a connection
  out to Net-Telco for the public telephone number.
  
- `extensions.conf`: This is a set of instructions describing the logic used for routing calls between
  handsets internally, as well as between internal handsets and Net-Telco (for incoming external calls).
  It is in this configuration file where the telephone numbers used to access each handset locally are
  defined, as well as settings relating to "number of rings", voice menu selections/prompts (for incoming
  external calls), and voicemail (for incoming external calls).

- `voicemail.conf`: This is a relatively sparse configuration file containing details about the
  voicemail "mailboxes" used for answering services. In the case of the BTS phone system, only one
  mailbox is defined - that of the Office handset. It is in this file where the e-mail address, used by
  Asterisk to send voicemail recordings to the committee inbox, is defined.

As well as Asterisk, the PBX server also runs _Postfix_. This is a common Linux e-mail program, most
often used for sending e-mails between servers. Asterisk leverages this program for its
voicemail-to-e-mail functionality.

Postfix is configured using the _/etc/postfix/main.cf_ file. It is in this file where the credentials
are specified for connecting to the BTS GSuite over SMTP/IMAP.

E-mail notes:
-------------

This section contains a few brief notes about how Postfix interfaces with the BTS GSuite in order to
reliably send voicemails over e-mail.

Whilst in principle, Postfix could send e-mails directly out over port 25 to the committee address, this
method generally results in e-mails being automatically marked as spam by the recipients' mail providers, 
owing to their not emanating from a reputable e-mail service/address.

Postfix instead connects to a special _pbx@bts-crew.com_ e-mail account on the BTS GSuite. This account
is then used to distribute voicemail-related e-mails. This account is solely for the use of the PBX server.
For the credentials used to access this account for administrative purposes, please contact Committee.

Since Postfix's connection out to the pbx@bts-crew.com account is automated, this appears to Google as
a "Less secure app access". Therefore, an _App Password_ is in place on this account for use by the Postfix
connection. This app password is separate to the main password used by a human being to log into the account,
and is used _only_ by Postfix on the PBX server to make the outgoing Google connection.
