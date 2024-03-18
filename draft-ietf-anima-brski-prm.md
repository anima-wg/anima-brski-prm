---
title: BRSKI with Pledge in Responder Mode (BRSKI-PRM)
abbrev: BRSKI-PRM
docname: draft-ietf-anima-brski-prm-13
area: Operations and Management
wg: ANIMA WG
date: 2024
stand_alone: true
ipr: trust200902
submissionType: IETF
wg: ANIMA WG
area: Operations and Management
cat: std
pi:
  toc: 'yes'
  compact: 'yes'
  symrefs: 'yes'
  sortrefs: 'yes'
  iprnotified: 'no'
  strict: 'yes'
author:
- ins: S. Fries
  name: Steffen Fries
  org: Siemens AG
  abbrev: Siemens
  street: Otto-Hahn-Ring 6
  city: Munich
  code: '81739'
  country: Germany
  email: steffen.fries@siemens.com
  uri: https://www.siemens.com/
- ins: T. Werner
  name: Thomas Werner
  org: Siemens AG
  abbrev: Siemens
  street: Otto-Hahn-Ring 6
  city: Munich
  code: '81739'
  country: Germany
  email: thomas-werner@siemens.com
  uri: https://www.siemens.com/
- ins: E. Lear
  name: Eliot Lear
  org: Cisco Systems
  street: Richtistrasse 7
  city: Wallisellen
  code: CH-8304
  country: Switzerland
  phone: "+41 44 878 9200"
  email: lear@cisco.com
- ins: M. Richardson
  name: Michael C. Richardson
  org: Sandelman Software Works
  email: mcr+ietf@sandelman.ca
  uri: http://www.sandelman.ca/
contributor:
- ins: E. Dijk
  name: Esko Dijk
  org: IoTconsultancy.nl
  email: esko.dijk@iotconsultancy.nl
- name: Toerless Eckert
  org: Futurewei
  email: tte@cs.fau.de
- ins: M. Kovatsch
  name: Matthias Kovatsch
  org: Siemens Schweiz AG
  email: ietf@kovatsch.net
venue:
  group: ANIMA
  anima mail: {anima@ietf.org}
  github: anima-wg/anima-brski-prm
normative:
  RFC6762:
  RFC6763:
  RFC7030:
  RFC7515:
  RFC8366:
  RFC8610:
  RFC8615:
  RFC8995:
  RFC9360:
  I-D.ietf-anima-jws-voucher:
  I-D.ietf-netconf-sztp-csr:
  I-D.ietf-anima-rfc8366bis:
informative:
  RFC2986:
  RFC3629:
  RFC5272:
  RFC9525:
  RFC6241:
  RFC7252:
  RFC8040:
  RFC8407:
  RFC8792:
  RFC8990:
  RFC9052:
  RFC9110:
  RFC9238:
  I-D.ietf-anima-brski-ae:
  I-D.richardson-emu-eap-onboarding:
  I-D.eckert-anima-brski-discovery:
  IEEE-802.1AR:
    title: IEEE 802.1AR Secure Device Identifier
    author:
    - org: Institute of Electrical and Electronics Engineers
    date: 2018-06
    seriesinfo:
      IEEE: '802.1AR'
  BRSKI-PRM-abstract:
    title: 'Abstract BRSKI-PRM Protocol Overview'
    date: March 2022
    format:
      PDF: https://datatracker.ietf.org/meeting/113/materials/slides-113-anima-update-on-brski-with-pledge-in-responder-mode-brski-prm-00
  onpath:
    target: "https://mailarchive.ietf.org/arch/msg/saag/m1r9uo4xYznOcf85Eyk0Rhut598/"
    title: "can an on-path attacker drop traffic?"
    org: IETF
  androidnsd:
    target: https://developer.android.com/training/connect-devices-wirelessly
    title: "Android Developer: Connect devices wirelessly"
    org: Google
    seriesinfo:
      "archived at": https://web.archive.org/web/20230000000000*/https://developer.android.com/training/connect-devices-wirelessly
  androidtrustfail:
    target: https://developer.android.com/training/articles/security-ssl
    title: "Security with Network Protocols"
    org: Google
    seriesinfo:
      "archived at": https://web.archive.org/web/20230326153937/https://developer.android.com/training/articles/security-ssl

--- abstract

This document defines enhancements to Bootstrapping a Remote Secure Key Infrastructure (BRSKI, RFC8995) to enable bootstrapping in domains featuring no or only limited connectivity between a pledge and the domain registrar.
It specifically changes the interaction model from a pledge-initiated mode, as used in BRSKI, to a pledge-responding mode, where the pledge is in server role.
For this, BRSKI with Pledge in Responder Mode (BRSKI-PRM) introduces a new component, the Registrar-Agent, which facilitates the communication between pledge and registrar during the bootstrapping phase.
To establish the trust relation between pledge and registrar, BRSKI-PRM relies on object security rather than transport security.
The approach defined here is agnostic to the enrollment protocol that connects the domain registrar to the domain CA.

--- middle

# Introduction

BRSKI as defined in {{!RFC8995}} specifies a solution for secure zero-touch (automated) bootstrapping of devices (pledges) in a customer domain, which may be associated to a specific installation location.
This includes the discovery of the BRSKI registrar in the customer domain and the exchange of security information necessary to establish trust between a pledge and the domain.

Security information about the customer domain, specifically the customer domain certificate, are exchanged and authenticated utilizing voucher-request and voucher-response artifacts as defined in {{!RFC8995}}.
Vouchers are signed objects from the Manufacturer Authorized Signing Authority (MASA).
The MASA issues the voucher and provides it via the domain registrar to the pledge.
{{I-D.ietf-anima-rfc8366bis}} specifies the format of the voucher artifacts including the voucher-request.

For the certificate enrollment of devices, BRSKI relies on EST {{!RFC7030}} to request and distribute customer domain specific device certificates.
EST in turn relies for the authentication and authorization of the certification request on the credentials used by the underlying TLS between the EST client and the EST server.

BRSKI addresses scenarios in which the pledge initiates the bootstrapping acting as client (referred to as initiator mode by this document).
BRSKI with Pledge in Responder Mode (BRSKI-PRM) defined in this document allows the pledge to act as server, so that it can be triggered externally and at a specific time to generate bootstrapping requests in the customer domain.
For this approach, this document:

* introduces the Registrar-Agent as new component to facilitate the communication between the pledge and the domain registrar.
  The Registrar-Agent may be implemented as an integrated functionality of a commissioning tool or be co-located with the domain registrar itself.
  BRSKI-PRM supports the identification of the Registrar-Agent that was performing the bootstrapping allowing for accountability of the pledges installation, when the Registrar-Agent is a component used by an installer and not co-located with the domain registrar.

* specifies the interaction (data exchange and data objects) between a pledge acting as server, the Registrar-Agent acting as client, and the domain registrar.

* enables the usage of arbitrary transports between the pledge and the domain registrar via the Registrar-Agent; security is addressed at the application layer, and both IP-based and non-IP connectivity can be used between pledge and Registrar-Agent.

* allows the application of Registrar-Agent credentials to establish TLS connections to the domain registrar; these are different from the IDevID of the pledge.

The term endpoint used in the context of this document is equivalent to resource in HTTP {{RFC9110}} and CoAP {{RFC7252}}; it is not used to describe a device.
Endpoints are accessible via Well-Known URIs {{RFC8615}}.
For the interaction with the domain registrar, the Registrar-Agent will use existing BRSKI {{!RFC8995}} endpoints as well as additional endpoints defined in this document.
To utilize the EST server endpoints on the domain registrar, the Registrar-Agent will act as client toward the registrar.

The Registrar-Agent also acts as a client when communicating with a pledge that is in responder mode.
Here, TLS with server-side, certificate-based authentication is only optionally supported.
If TLS is optionally used between the Registrar-Agent and the pledge, the Registrar-Agent needs to identify the pledge based on its product-serial-number rather than the hostname, as the latter is not set in an IDevID certificate.

BRSKI-PRM is designed to rely on object security to support also for alternative transports for which TLS may not be available, e.g., Bluetooth or NFC.
This is achieved through an additional signature wrapping of the exchanged data objects involving the Registrar-Agent for transport.

To utilize EST {{!RFC7030}} for enrollment, the domain registrar performs pre-processing of the wrapping signature before actually using EST as defined in {{!RFC7030}}.

There may be pledges that can support both modes, initiator and responder mode.
In these cases BRSKI-PRM can be combined with BRSKI as defined in {{!RFC8995}} or BRSKI-AE {{I-D.ietf-anima-brski-ae}} to allow for more bootstrapping flexibility.


# Terminology

{::boilerplate bcp14-tagged}

This document relies on the terminology defined in {{Section 1.2 of !RFC8995}}.
The following terms are defined in addition:

authenticated self-contained object:
: Describes an object, which is cryptographically bound to the end entity (EE) certificate.
  The binding is assumed to be provided through a digital signature of the actual object using the corresponding private key of the certificate.

CA:
: Certification Authority, issues certificates.

Commissioning tool:
: Tool to interact with devices to provide configuration data.

CSR:
: Certificate Signing Request.

EE:
: End entity, as defined in {{?RFC9483}}.
  Typically a device or service that owns a public-private key pair for which it manages a public key certificate.

EE certificate:
: Either IDevID certificate or LDevID certificate of the EE.

endpoint:
: Term equivalent to resource in HTTP {{RFC9110}} and CoAP {{RFC7252}}.
  Endpoints are accessible via Well-Known URIs {{RFC8615}}.

mTLS:
: mutual Transport Layer Security.

PER:
: Pledge Enroll-Request is a signature wrapped CSR, signed by the pledge that requests enrollment to a domain.

POI:
: Proof-of-Identity, as defined in {{RFC5272}}.

POP:
: Proof-of-Possession (of a private key), as defined in {{RFC5272}}.

PVR:
: Pledge Voucher-Request is a request for a voucher sent to the domain registrar.
The PVR is signed by the Pledge.

RA:
: Registration Authority, an optional system component to which a CA delegates certificate management functions such as authorization checks.
In BRSKI-PRM this is a functionality of the domain registrar, as in BRSKI {{!RFC8995}}.

RER:
: Registrar Enroll-Request is the CSR of a PER sent to the CA by the domain registrar (in its role as PKI RA).

RVR:
: Registrar Voucher-Request is a request for a voucher signed by the domain registrar to the MASA.
It may contain the PVR received from the pledge.

This document uses the following encoding notations in the given JWS-signed artifact examples:

BASE64URL(OCTETS):
: Denotes the base64url encoding of OCTETS, per {{Section 2 of !RFC7515}}.

UTF8(STRING):
: Denotes the octets of the UTF-8 {{?RFC3629}} representation of STRING, per {{Section 1 of !RFC7515}}.

This document includes many examples that would contain many long sequences of base64-encoded objects with no content directly comprehensible to a human reader.
In order to keep those examples short, they use the token "base64encodedvalue==" as a placeholder for base64 data.
The full base64 data is included in the appendices of this document.


# Scope of Solution

## Supported Environments and Use Case Examples {#sup-env}

BRSKI-PRM is applicable to scenarios where pledges may have no direct connection to the domain registrar, may have no continuous connection, or require coordination of the pledge requests to be provided to a domain registrar.

This can be motivated by pledges deployed in environments not yet connected to the operational customer domain network, e.g., at a building construction site, or environments intentionally disconnected from the Internet, e.g., critical industrial facilities.
Another example is the assembly of electrical cabinets, which are prepared in advance before the installation at a customer domain.


### Building Automation

In building automation a typical use case exists where a detached building or the basement is equipped with sensors, actuators, and controllers, but with only limited or no connection to the central building management system.
This limited connectivity may exist during installation time or also during operation time.

During the installation, for instance, a service technician collects the device-specific information from the basement network and provides them to the central building management system.
This could be done using a laptop, common mobile device, or dedicated commissioning tool to transport the information.
The service technician may successively collect device-specific information in different parts of the building before connecting to the domain registrar for bulk bootstrapping.

A domain registrar may be part of the central building management system and already be operational in the installation network.
The central building management system can then provide operational parameters for the specific devices in the basement or other detached areas.
These operational parameters may comprise values and settings required in the operational phase of the sensors/actuators, among them a certificate issued by the operator to authenticate against other components and services.
These operational parameters are then provided to the devices in the basement facilitated by the service technician's laptop.
The Registrar-Agent, defined in this document, may be run on the technician's laptop to interact with pledges.


### Infrastructure Isolation Policy

This refers to any case in which the network infrastructure is normally isolated from the Internet as a matter of policy, most likely for security reasons.
In such a case, limited access to a domain registrar may be allowed in carefully controlled short periods of time, for example when a batch of new devices are deployed, but prohibited at other times.


### Less Operational Security in the Target-Domain

The registration authority (RA) performing the authorization of a certificate request is a critical PKI component and therefore requires higher operational security than other components utilizing the issued certificates.
CAs may also require higher security in the registration procedures.
There may be situations in which the customer domain does not offer enough physical security to operate a RA/CA and therefore this service is transferred to a backend that offers a higher level of operational security.


## Limitations

The mechanism described in this document presumes the ability of the pledge and the Registrar-Agent to communicate with another.
This may not be possible in constrained environments where, in particular, power must be conserved.
In these situations, it is anticipated that the transceiver will be powered down most of the time.
This presents a rendezvous problem: the pledge is unavailable for certain periods of time, and the Registrar-Agent is similarly presumed to be unavailable for certain periods of time.
To overcome this situation, the pledges may need to be powered on, either manually or by sending a trigger signal.


# Requirements Discussion and Mapping to Solution-Elements {#req-sol}

Based on the intended target environment described in {{sup-env}}, the following requirements are derived to support bootstrapping of pledges in responder mode (acting as server):

* To facilitate the communication between a pledge in responder mode and the registrar, additional functionality is needed either on the registrar or as a stand-alone component.
  This new functionality is defined as Registrar-Agent and acts as an agent of the registrar to trigger the pledge to generate requests for voucher and enrollment.
  These requests are then provided by the Registrar-Agent to the registrar.
  This requires the definition of pledge endpoints to allow interaction with the Registrar-Agent.

* The security of communication between the Registrar-Agent and the pledge must not rely on Transport Layer Security (TLS) to enable application of BRSKI-PRM in environments, in which the communication between the Registrar-Agent and the pledge is done over other technologies like BTLE or NFC, which may not support TLS protected communication.
  In addition, the pledge does not have a certificate that can easily be verified by {{?RFC9525}} methods.

* The use of authenticated self-contained objects addresses both, the TLS challenges and the technology stack challenge.

* By contrast, the Registrar-Agent can be authenticated by the registrar as a component, acting on behalf of the registrar.
  In addition the registrar must be able to verify, which Registrar-Agent was in direct contact with the pledge.

* It would be inaccurate for the voucher-request and voucher-response to use an assertion with value "proximity" in the voucher, as the pledge was not in direct contact with the registrar for bootstrapping.
  Therefore, a new Agent-Proximity Assertion value {#agt_prx} is necessary for distinguishing assertions the MASA can state.

At least the following properties are required for the voucher and enrollment processing:

* POI: provides data-origin authentication of a data object, e.g., a voucher-request or an Enroll-Request, utilizing an existing IDevID.
  Certificate updates may utilize the certificate that is to be updated.

* POP: proves that an entity possesses and controls the private key corresponding to the public key contained in the certification request, typically by adding a signature computed using the private key to the certification request.

Solution examples based on existing technology are provided with the focus on existing IETF RFCs:

* Voucher-Requests and Vouchers as used in {{!RFC8995}} already provide both, POP and POI, through a digital signature to protect the integrity of the voucher, while the corresponding signing certificate contains the identity of the signer.

* Enroll-Requests are data structures containing the information from a requester for a CA to create a certificate.
  The certification request format in BRSKI is PKCS#10 {{?RFC2986}}.
  In PKCS#10, the structure is signed to ensure integrity protection and POP of the private key of the requester that corresponds to the contained public key.
  In the application examples, this POP alone is not sufficient.
  A POI is also required for the certification request and therefore the certification request needs to be additionally bound to the existing credential of the pledge (IDevID).
  This binding supports the authorization decision for the certification request and may be provided directly with the certification request.
  While BRSKI uses the binding to TLS, BRSKI-PRM aims at an additional signature of the PKCS#10 using existing credentials on the pledge (IDevID). This allows the process to be independent of the selected transport.


# Architecture {#architecture}

## Overview

For BRSKI with Pledge in Responder Mode (BRSKI-PRM), the base system architecture defined in BRSKI {{!RFC8995}} is enhanced to facilitate new use cases in which the pledge acts as server.
The responder mode allows delegated bootstrapping using a Registrar-Agent instead of a direct connection between the pledge and the domain registrar.

Necessary enhancements to support authenticated self-contained objects for certificate enrollment are kept at a minimum to enable reuse of already defined architecture elements and interactions.
The format of the bootstrapping objects produced or consumed by the pledge is usually based on JSON Web Signature (JWS) {{!RFC7515}} and further specified in {{exchanges_uc2}} to address the requirements stated in {{req-sol}} above.
In constrained environments, it may be based on COSE {{?RFC9052}}.

An abstract overview of the BRSKI-PRM protocol can be found on slide 8 of {{BRSKI-PRM-abstract}}.

To support mutual trust establishment between the domain registrar and pledges not directly connected to the customer domain, this document specifies the exchange of authenticated self-contained objects with the help of a Registrar-Agent.

This leads to extensions of the logical components in the BRSKI architecture as shown in {{uc2figure}}.

Note that the Join Proxy is not shown in the figure.
In certain situations the Join Proxy may still be present and could be used by the Registrar-Agent to connect to the Registrar.
For example, a Registrar-Agent application on a smartphone often can connect to local Wi-Fi without giving up their cellular network connection {{androidnsd}}, but only can make link-local connections.

The Registrar-Agent interacts with the pledge to transfer the required data objects for bootstrapping, which are then also exchanged between the Registrar-Agent and the domain registrar.
The addition of the Registrar-Agent influences the sequences of the data exchange between the pledge and the domain registrar described in {{!RFC8995}}.
To enable reuse of BRSKI defined functionality as much as possible, BRSKI-PRM:

* uses existing endpoints where the required functionality is provided.
* enhances existing endpoints with new supported media types, e.g., for JWS voucher.
* defines new endpoints where additional functionality is required, e.g., for wrapped certification request, CA certificates, or new status information.

~~~~ aasvg
                         +---------------------------+
    ..... Drop Ship .....| Vendor Services           |
    :                    +---------------+-----------+
    :                    | M anufacturer |           |
    :                    | A uthorized   | Ownership |
    :                    | S igning      | Tracker   |
    :                    | A uthority    |           |
    :                    +---------------+-----------+
    :                                         ^
    :                                         | BRSKI-
    :                                         | MASA
    :          ...............................|.........
    V          .                              v        .
+--------+     .  +------------+        +-----------+  .
|        |     .  |            |        |           |  .
| Pledge | BRSKI- | Registrar- | BRSKI- | Domain    |  .
|        |  PRM   | Agent      |  PRM   | Registrar |  .
|        |<------>|            |<------>| (PKI RA)  |  .
|        |     .  |    EE cert |        |           |  .
|        |     .  +------------+        +-----+-----+  .
| IDevID |     .                              |        .
|        |     .           +------------------+-----+  .
+--------+     .           | Key Infrastructure     |  .
               .           | (e.g., PKI CA)         |  .
               .           +------------------------+  .
               .........................................
                            Customer Domain
~~~~
{: #uc2figure title='BRSKI-PRM architecture overview using Registrar-Agent' artwork-align="center"}

{{uc2figure}} shows the relations between the following main components:

* Pledge: Is expected to respond with the necessary data objects for bootstrapping to the Registrar-Agent.
  The protocol used between the pledge and the Registrar-Agent is assumed to be HTTP in the context of this document.
  Any other protocols (including HTTPS) can be used as long as they support the exchange of the necessary data objects.
  This includes CoAP or protocol to be used over Bluetooth or NFC connections
  A pledge acting as a server during bootstrapping leads to the following differences compared to BRSKI:
  
  * The pledge is discovered by the Registrar-Agent as defined in {#discovery_uc2_ppa}.
  * The pledge offers additional endpoints as defined in {{pledge_ep}}, so that the Registrar-Agent can request data required for bootstrapping the pledge.
  * The pledge includes additional data in the PVR, which is provided by the Registrar-Agent in the voucher-request trigger as defined in {{tpvr}}.
    This allows the registrar to identify, with which Registrar-Agent the pledge was in contact.
  * The order of exchanges in the BRSKI-PRM call flow is different from those in BRSKI {{!RFC8995}}, as the PVR and PER are collected simultaneously and provided to the registrar.
    This enables bulk bootstrapping of several devices.
  * The data objects utilized for the data exchange between the pledge and the registrar are self-contained authenticated objects (signature-wrapped objects).

* Registrar-Agent: Provides a store and forward communication path to exchange data objects between the pledge and the domain registrar.
  The Registrar-Agent acts as a broker in situations in which the domain registrar is not directly reachable by the pledge.
  This may be due to a different technology stack or due to missing connectivity.

  * The Registrar-Agent triggers one or more pledges to create bootstrapping artifacts such as the voucher-request and the Enroll-Request.
    It can then perform a (bulk) bootstrapping based on the collected data.
  * The Registrar-Agent is expected to possess information about the domain registrar: the registrar EE certificate, LDevID(CA) certificate, and IP address, either by configuration or by using the discovery mechanism defined in {{!RFC8995}}.
  * There is no trust assumption between the pledge and the Registrar-Agent as only authenticated self-contained objects are used, which are transported via the Registrar-Agent and provided either by the pledge or the domain registrar.
  * The trust assumption between the Registrar-Agent and the domain registrar may be based on an LDevID, which is provided by the PKI responsible for the customer domain.
  * The Registrar-Agent may be realized as stand-alone component supporting nomadic activities of a service technician moving between different installation sites.
  * Alternatively, the Registrar-Agent may also be realized as co-located functionality for a registrar, to support pledges in responder mode.

* Join Proxy (not shown): Has the same functionality as described in {{!RFC8995}} if needed.
  Note that a Registrar-Agent may use a join proxy to facilitate the TLS connection to the registrar in the same way that a BRSKI pledge would use a join proxy. This is useful in cases where the Registrar-Agent does not have full IP connectivity via the domain network or cases where it has no other means to locate the registrar on the network.

* Domain Registrar: In general fulfills the same functionality regarding the bootstrapping of the pledge in a customer domain by facilitating the communication of the pledge with the MASA service and the domain key infrastructure (PKI).
  In contrast to {{!RFC8995}}, a BRSKI-PRM domain registrar does not interact with a pledge directly, but through the Registrar-Agent.

* Vendor Services: Encompass MASA and Ownership Tracker and are used as defined in {{!RFC8995}}.
  A MASA is able to support enrollment via Registrar-Agent without changes unless it checks the vouchers proximity indication, in which case it would need to be enhanced to support BRSKI-PRM to also accept the Agent-Proximity Assertion {#agt_prx}.

## Nomadic Connectivity {#arch_nomadic}

In one example instance of the PRM architecture as shown in {{uc3figure}}, there is no connectivity between the location in which the pledge is installed and the location of the domain registrar.
This is often the case in the aforementioned building automation use case ({{building-automation}}).

~~~~ aasvg
                         +---------------------------+
    ..... Drop Ship .....| Vendor Services           |
    :                    +---------------------------+
    :                                         ^
........................................      |
.   v                                  .      |
. +--------+           .-.-.-.-.-.-.-. .      |
. |        |           : Registrar-  : .      |
. | Pledge |<--------->: Agent       : .      |
. +--------+ L2 or L3  :-.-.-.-.-.-.-: .      |
.          connectivity   ^            .      |
..........................!.............      |
   Pledge Installation    !                   |
   Location               ! Nomadic           |
                          ! connectivity      |
                          !                   |
               ...........!...................|.........
               .          v                   v        .
               .  .-.-.-.-.-.-.-.       +-----------+  .
               .  : Registrar-  :       | Domain    |  .
               .  : Agent       :<----->| Registrar |  .
               .  :-.-.-.-.-.-.-:       +-----+-----+  .
               .                              |        .
               .           +------------------+-----+  .
               .           | Key Infrastructure     |  .
               .           | (e.g., PKI CA)         |  .
               .           +------------------------+  .
               .........................................
                            Customer Domain
~~~~
{: #uc3figure title='Registrar-Agent nomadic connectivity example' artwork-align="center"}

PRM enables support of this case through nomadic connectivity of the Registrar-Agent.
To perform enrollment in this setup, multiple round trips of the Registrar-Agent between the pledge installation location and the domain registrar are required.

1.  Connectivity to domain registrar: preparation tasks for pledge bootstrapping not part of the BRSKI-PRM protocol definition, like retrieval of list of pledges to enroll.
2.  Connectivity to pledge installation location: retrieve information about available pledges (IDevID), collect request objects (i.e., Pledge Voucher-Requests and Pledge Enroll-Requests using the BRSKI-PRM approach described in {{tpvr}} and {{tper}}.
3.  Connectivity to domain registrar, submit collected request information of pledges, retrieve response objects (i.e., Voucher and Enroll-Response) using the BRSKI-PRM approach described in {{pvr}} and {{per}}.
4.  Connectivity to pledge installation location, provide retrieved objects to the pledges to enroll pledges and collect status using the BRSKI-PRM approach described in {{voucher}}, {{cacerts}}, and {{enroll_response}}.
5.  Connectivity to domain registrar, submit Voucher Status and Enrollment Status using the BRSKI-PRM approach described in {{vstatus}} and {{estatus}}.

Variations of this setup include cases where the Registrar-Agent uses for example WiFi to connect to the pledge installation network, and mobile network connectivity to connect to the domain registrar.
Both connections may also be possible in a single location at the same time, based on installation building conditions.

## Co-located Registrar-Agent and Domain Registrar

Compared to {{!RFC8995}} BRSKI, pledges supporting BRSKI-PRM can be completely passive and only need to react when being requested to react by a Registrar-Agent.
In {{!RFC8995}}, pledges instead need to continuously request enrollment from a domain registrar, which may result in undesirable communications pattern and possible overload of a domain registrar.

~~~~ aasvg
                         +---------------------------+
    ..... Drop Ship .....| Vendor Service            |
    :                    +---------------------------+
    :                                         ^
    :                                         |
    :          ...............................|.........
    :          .                              v        .
    v          .          +-------------------------+  .
 +--------+    .          |..............           |  .   
 |        |    .          |. Registrar- . Domain    |  .
 | Pledge |<------------->|. Agent      . Registrar |  .
 +--------+ L2 or L3      |..............           |  .   
            connectivity  +-------------------+-----+  .
               .                              |        .
               .           +------------------+-----+  .
               .           | Key Infrastructure     |  .
               .           +------------------------+  .
               .........................................
                            Customer Domain
~~~~
{: #uc4figure title='Registrar-Agent integrated into Domain Registrar example' artwork-align="center"}

The benefits of BRSKI-PRM can be achieved even without the operational complexity of standalone Registrar-Agents by integrating the necessary functionality of the Registrar-Agent as a module into the domain registrar as shown in {{uc4figure}} so that it can support the BRSKI-PRM communications to the pledge.


## Agent-Proximity Assertion {#agt_prx}

"Agent-proximity" is a statement in the PVR and in the voucher, that the registrar certificate was provided via the Registrar-Agent as defined in {{exchanges_uc2}} and not directly to the pledge.
Agent-proximity is therefore a different assertion than "proximity", which is defined in {{Section 4 of RFC8366}}.
Agent-proximity is defined as additional assertion type in {{I-D.ietf-anima-rfc8366bis}}.
This assertion can be verified by the registrar and also by the MASA during the voucher-request processing.

In BRSKI, the pledge verifies POP of the registrar via the TLS handshake and pins that public key as the "proximity-registrar-cert" into the voucher request.
This allows the MASA to verify the proximity of the pledge and registrar, facilitating a decision to assign the pledge to that domain owner.
In BRSKI, the TLS connection is considered provisional until the pledge receives the voucher.

In contrast, in BRSKI-PRM, the pledge has no direct connection to the registrar and MUST accept the registrar certificate provisionally until it receives the voucher as described in {{voucher}}.
In a similar fashion, the pledge MUST accept the Registrar-Agent EE certificate provisionally.
See also {{Section 5 of !RFC8995}} on "provisional state".

For agent-proximity, the EE certificate of the Registrar-Agent MUST be an LDevID certificate signed by the domain owner.
Akin to the proximity assertion in the BRSKI case, the agent-proximity provides pledge proximity evidence to the MASA.
But additionally, agent-proximity allows the domain registrar to be sure that the PVR collected by the Registrar-Agent was in fact collected by the Registrar-Agent, to which the registrar is connected to.

The provisioning of the Registrar-Agent LDevID certificate is out of scope for this document, but may be done in advance using a separate BRSKI run or by other means like configuration.
It is recommended to use short lived Registrar-Agent LDevIDs in the range of days or weeks as outlined in {{sec_cons_reg-agt}}.


# System Components


## Domain Registrar

In BRSKI-PRM, the domain registrar provides the endpoints already specified in {{!RFC8995}} (derived from EST {{!RFC7030}}) where suitable.
In addition, it MUST provide the endpoints defined in {{registrar_ep_table}} within the BRSKI-defined "/.well-known/brski/" URI path.
These endpoints accommodate for the signature-wrapped objects used by BRSKI-PRM for the Pledge Enroll-Request (PER) and the provisioning of CA certificates.

|Endpoint        | Operation                  | Exchange and Artifacts  |
|:---------------|:---------------------------|:------------------------|
| requestenroll  | Supply PER to Registrar    | {{per}} |
|------------------------
| wrappedcacerts | Request CA Certificates    | {{req_cacerts}} |
|===============
{: #registrar_ep_table title='Additional Well-Known Endpoints on a BRSKI-PRM Registrar' }

According to {{Section 5.3 of !RFC8995}}, the domain registrar performs the pledge authorization for bootstrapping within his domain based on the Pledge Voucher-Request.
This behavior is retained in BRSKI-PRM.

The domain registrar MUST possess and trust the IDevID (root or issuing) CA certificate 
of the pledge vendor/manufacturer.

Further, the domain registrar MUST have its own EE credentials.

### Domain Registrar with Combined Functionality

A registrar with combined BRSKI and BRSKI-PRM functionality MAY detect if the bootstrapping is performed by the pledge directly (BRSKI case) or by a Registrar-Agent (BRSKI-PRM case) based on the utilized credential for client authentication during the TLS session establishment and switch switch the operational mode from BRSKI to BRSKI-PRM.

This may be supported by a specific naming in the SAN (subject alternative name) component of the EE certificate of the Registrar-Agent.

Alternatively, this may be supported by using an LDevID certificate signed by the domain owner for the client authentication of the Registrar-Agent.
Using an LDevID certificate also allows the registrar to verify that a Registrar-Agent is authorized to perform the bootstrapping of a pledge.
See also agent-proximity assertion in {{agt_prx}}.

Using an LDevID certificate for TLS client authentication of the Registrar-Agent is a deviation from {{!RFC8995}}, in which the IDevID credential of the pledge is used to perform TLS client authentication.


## Registrar-Agent

The Registrar-Agent is a new component in BRSKI-PRM that provides a secure message passing service between pledges in responder mode and the domain registrar.

It requires the EE certificate of the domain registrar for TLS server authentication when establishing a TLS session with the domain registrar and to provide the registrar EE certificate to the pledge for creating the Pledge Voucher-Request (PVR).

The Registrar-Agent uses its own EE certificate for TLS client authentication when establishing a TLS session with the domain registrar and for signing agent-signed data.
This EE certificate MUST include a SubjectKeyIdentifier (SKID), which is used as reference in the context of an agent-signed-data object as defined in {{tpvr}}.

Note that this is an additional requirement for issuing the certificate, as {{!IEEE-802.1AR}} only requires the SKID to be included for intermediate CA certificates.
{{!RFC8995}} has a similar requirement.
In BRSKI-PRM, the SKID is used in favor of providing the complete EE certificate of the Registrar-Agent to accommodate also constrained environments and reduce bandwidth needed for communication with the pledge.
In addition, it follows the recommendation from BRSKI to use SKID in favor of a certificate fingerprint to avoid additional computations.

In addition to the EE certificates, the Registrar-Agent is provided with the product serial number(s) of the pledge(s) to be bootstrapped.
This is necessary to allow the discovery of pledge(s) by the Registrar-Agent using DNS-SD with mDNS (see {{discovery_uc2_ppa}}).
The list may be provided by prior administrative means or the Registrar-Agent may get the information via an interaction with the pledge.
For instance, {{RFC9238}} describes scanning of a QR code, where the product serial number would be initialized from the 12N B005 Product Serial Number.

In summary, the following information MUST be available at the Registrar-Agent before interaction with a pledge:

* Domain registrar EE certificate: certificate of the domain registrar to be provided to the pledge.
* Registrar-Agent EE certificate and corresponding private key: own operational key pair to sign agent-signed-data.
* Serial number(s): product serial number(s) of pledge(s) to be bootstrapped for discovery.

Further, the Registrar-Agent SHOULD have synchronized time.

Finally, the Registrar-Agent MAY possess the IDevID (root or issuing) CA certificate of the pledge vendor/manufacturer to validate the IDevID certificate on returned PVR or in case of TLS usage for pledge communication.
The distribution of IDevID CA certificates to the Registrar-Agent is out of scope of this document and may be done by a manual configuration.


### Discovery of the Registrar {#discovery_uc2_reg}

As a Registrar-Agent acts as representative of the domain registrar towards the pledge or may even be collocated with the domain registrar, a separate discovery of the registrar is likely not needed as Registrar-Agent and registrar are domain components and have a trust relation.
Moreover, other communication (not part of this document) between the Registrar-Agent and the registrar is assumed, e.g., to exchange information about product-serial-number(s) of pledges to be discovered as outlined in {{arch_nomadic}}.
Moreover, as the standard discovery described in {{Section 4 of !RFC8995}} and the {{Appendix A.2 of !RFC8995}} does not support  of registrars with an enhanced feature set (like the support of BRSKI-PRM), this standard discovery is not applicable.

As a more general solution, the BRSKI discovery mechanism can be extended to provide upfront information on the capabilities of registrars, such as the mode of operation (pledge-responder-mode or registrar-responder-mode).
Defining discovery extensions is out of scope of this document.
This may be provided in {{I-D.eckert-anima-brski-discovery}}.


### Discovery of the Pledge {#discovery_uc2_ppa}

The discovery of the pledge by Registrar-Agent in the context of this document describes the minimum discovery approach to be supported.
A more general discovery mechanism, also supporting GRASP besides DNS-SD with mDNS may be provided in {{I-D.eckert-anima-brski-discovery}}.

Discovery in BRSKI-PRM uses DNS-based Service Discovery {{RFC6763}} over Multicast DNS {{RFC6762}} to discover the pledge.
Note that {{RFC6762}} Section 9 provides support for conflict resolution in situations when an DNS-SD with mDNS responder receives a mDNS response with inconsistent data.
Note that {{RFC8990}} does not support conflict resolution of mDNS, which may be a limitation for its application.

The pledge constructs a local host name based on device local information (product-serial-number), which results in "product-serial-number._brski-pledge._tcp.local".
The product-serial-number composition is manufacturer dependent and may contain information regarding the manufacturer, the product type, and further information specific to the product instance. To allow distinction of pledges, the product-serial-number therefore needs to be sufficiently unique.

In the absence of a more general discovery as defined in {{I-D.eckert-anima-brski-discovery}} the Registrar-Agent MUST  use

* "&lt;product-serial-number&gt;._brski-pledge._tcp.local", to discover a specific pledge, e.g., when connected to a local network.
* "_brski-pledge._tcp.local" to get a list of pledges to be bootstrapped.

A manufacturer may allow the pledge to react on DNS-SD with mDNS discovery without his product-serial-number contained. This allows a commissioning tool to discover pledges to be bootstrapped in the domain. The manufacturer support this functionality as outlined in {{sec_cons_mDNS}}.

Establishing network connectivity of the pledge is out of scope of this document but necessary to apply DNS-SD with mDNS.
For Ethernet it is provided by simply connecting the network cable.
For WiFi networks, connectivity can be provided by using a pre-agreed SSID for bootstrapping, e.g., as proposed in {{I-D.richardson-emu-eap-onboarding}}.
The same approach can be used by 6LoWPAN/mesh using a pre-agreed PAN ID.
How to gain network connectivity is out of scope of this document.



## Pledge in Responder Mode {#pledge_ep}

The pledge is triggered by the Registrar-Agent to create the PVR and PER.
It is also triggered for processing of the responses and the generation of status information once the Registrar-Agent has received the responses from the registrar later in the process.

To enable interaction as responder with the Registrar-Agent, pledges in responder mode MUST act as servers and MUST provide the endpoints defined in {{pledge_ep_table}} within the BRSKI-defined "/.well-known/brski/" URI path.
The endpoints are defined with short names to also accommodate for resource-constrained devices.

| Endpoint | Operation                        | Exchange and Artifacts |
|:---------|:---------------------------------|:-----------------------|
| tpvr     | Trigger Pledge Voucher-Request   | {{tpvr}}               |
|------------------------
| tper     | Trigger Pledge Enroll-Request    | {{tper}}               |
|------------------------
| svr      | Supply Voucher to Pledge         | {{voucher}}            |
|------------------------
| scac     | Supply CA Certificates to Pledge | {{cacerts}}            |
|------------------------
| ser      | Supply Enroll-Response to Pledge | {{enroll_response}}    |
|------------------------
| qps      | Query Pledge Status              | {{query}}              |
|===============
{: #pledge_ep_table title='Well-Known Endpoints on a Pledge in Responder Mode' }

{{Section 7.2 of ?RFC9110}} makes the Host header field mandatory, so it will always be present.
The pledge MUST respond to all queries regardless of the Host header field provided by the client.

For instance, when the Registrar-Agent reaches out to the "tpvr" endpoint on a pledge in responder mode with the full URI "http://pledge.example.com/.well-known/brski/tpvr", it sets the Host header field to "pledge.example.com" and the absolute path "/.well-known/brski/tpbr".
In practice, however, the pledge often is only known by its IP address as returned by a discovery protocol, which will be included in the Host header field.

As BRSKI-PRM uses authenticated self-contained data objects between the pledge and the domain registrar, the binding of the pledge identity to the requests is provided by the data object signature employing the IDevID of the pledge.
Hence, pledges MUST have an Initial Device Identifier (IDevID) installed in them at the factory.


### Pledge with Combined Functionality

Pledges MAY support both initiator and responder mode.

A pledge in initiator mode should listen for announcement messages as described in {{Section 4.1 of !RFC8995}}.
Upon discovery of a potential registrar, it initiates the bootstrapping to that registrar.
At the same time (so as to avoid the Slowloris-attack described in {{!RFC8995}}), it SHOULD also respond to the triggers for responder mode described in this document.

Once a pledge with combined functionality has been bootstrapped, it MAY act as client for enrollment of further certificates needed, e.g., using the enrollment protocol of choice.
If it still acts as server, the defined BRSKI-PRM endpoints to trigger a Pledge Enroll-Request (PER) or to provide an Enroll-Response can be used for further certificates.




# Exchanges and Artifacts {#exchanges_uc2}

The interaction of the pledge with the Registrar-Agent may be accomplished using different transports (i.e., protocols and/or network technologies).
This specification utilizes HTTP as default transport.
Other specifications may define alternative transports such as CoAP, Bluetooth Low Energy (BLE), or Near Field Communication (NFC).
These transports may differ from and are independent of the ones used between the Registrar-Agent and the registrar.

Transport independence is realized through data objects that are not bound to specific transport security and stay the same along the communication path from the pledge via the Registrar-Agent to the registrar.
Therefore, authenticated self-contained artifacts (e.g., JWS-signed JSON structures or COSE-signed CBOR structures) are used for the data exchanges between the pledge and the registrar via the Registrar-Agent.

{{exchangesfig_uc2_all}} provides an overview of the exchanges detailed in the following subsections.

~~~~ aasvg
+--------+    +------------+    +-----------+    +--------+    +------+
| Pledge |    | Registrar- |    |  Domain   |    | Domain |    | MASA |
|        |    |   Agent    |    | Registrar |    |   CA   |    |      |
+--------+    +------------+    +-----------+    +--------+    +------+
 |                  |                 |                 |   Internet |
 |     discover     |                 |                 |            |
 |      pledge      |                 |                 |            |
 |    mDNS query    |                 |                 |            |
 |<-----------------|                 |                 |            |
 |----------------->|                 |                 |            |
 |                  |                 |                 |            |
 ~                  ~                 ~                 ~            ~
(1) Trigger Pledge Voucher-Request
 ~                  ~                 ~                 ~            ~
 |                  |                 |                 |            |
 |<----opt. TLS---->|                 |                 |            |
 |<------tPVR-------|                 |                 |            |
 |--------PVR------>|                 |                 |            |
 |                  |                 |                 |            |
 ~                  ~                 ~                 ~            ~
(2) Trigger Pledge Enroll-Request
 ~                  ~                 ~                 ~            ~
 |                  |                 |                 |            |
 |<----opt. TLS---->|                 |                 |            |
 |<------tPER-------|                 |                 |            |
 |--------PER------>|                 |                 |            |
 |                  |                 |                 |            |
 ~                  ~                 ~                 ~            ~
(3) Supply PVR to Registrar (including backend interaction)
 ~                  ~                 ~                 ~            ~
 |                  |                 |                 |            |
 |                  |<-----mTLS------>|                 |            |
 |                  |         [Registrar-Agent          |            |
 |                  |    authenticated&authorized?]     |            |
 |                  |-------PVR------>|                 |            |
 |                  |          [accept device?]         |            |
 |                  |          [contact vendor]         |            |
 |                  |                 |                 |            |
 |                  |                 |<------------mTLS------------>|
 |                  |                 |--------------RVR------------>|
 |                  |                 |              [extract DomainID]
 |                  |                 |              [update audit log]
 |                  |                 |<-----------Voucher-----------|
 |                  |<----Voucher-----|                 |            |
 |                  |                 |                 |            |
 ~                  ~                 ~                 ~            ~
(4) Supply PER to Registrar (including backend interaction)
 ~                  ~                 ~                 ~            ~
 |                  |                 |                 |            |
 |                  |<----(mTLS)----->|                 |            |
 |                  |-------PER------>|                 |            |
 |                  |                 |<-----mTLS------>|            |
 |                  |                 |-------RER------>|            |
 |                  |                 |<--Enroll-Resp---|            |
 |                  |<--Enroll-Resp---|                 |            |
 |                  |                 |                 |            |
 ~                  ~                 ~                 ~            ~
(5) Request CA Certificates
 ~                  ~                 ~                 ~            ~
 |                  |                 |                 |            |
 |                  |<----(mTLS)----->|                 |            |
 |                  |---cACert-Req--->|                 |            |
 |                  |<--cACert-Resp---|                 |            |
 |                  |                 |                 |            |
 ~                  ~                 ~                 ~            ~
(6) Supply Voucher to Pledge
 ~                  ~                 ~                 ~            ~
 |                  |                 |                 |            |
 |<----opt. TLS---->|                 |                 |            |
 |<-----Voucher-----|                 |                 |            |
 |------vStatus---->|                 |                 |            |
 |                  |                 |                 |            |
 ~                  ~                 ~                 ~            ~
(7) Supply CA Certificates to Pledge
 ~                  ~                 ~                 ~            ~
 |                  |                 |                 |            |
 |<----opt. TLS---->|                 |                 |            |
 |<-----cACerts-----|                 |                 |            |
 |                  |                 |                 |            |
 ~                  ~                 ~                 ~            ~
(8) Supply Enroll-Response to Pledge
 ~                  ~                 ~                 ~            ~
 |                  |                 |                 |            |
 |<----opt. TLS---->|                 |                 |            |
 |<---Enroll-Resp---|                 |                 |            |
 |-----eStatus----->|                 |                 |            |
 |                  |                 |                 |            |
 ~                  ~                 ~                 ~            ~
(9) Voucher Status Telemetry (including backend interaction)
 ~                  ~                 ~                 ~            ~
 |                  |                 |                 |            |
 |                  |<----(mTLS)----->|                 |            |
 |                  |-----vStatus---->|                 |            |
 |                  |                 |<-----------(mTLS)----------->|
 |                  |                 |-----req device audit log---->|
 |                  |                 |<------device audit log-------|
 |                  |        [verify audit log]         |            |
 |                  |                 |                 |            |
 ~                  ~                 ~                 ~            ~
(10) Enroll Status Telemetry
 ~                  ~                 ~                 ~            ~
 |                  |                 |                 |            |
 |                  |<----(mTLS)----->|                 |            |
 |                  |-----eStatus---->|                 |            |
 |                  |                 |                 |            |
 ~                  ~                 ~                 ~            ~
(11) Query Pledge Status
 ~                  ~                 ~                 ~            ~
 |                  |                 |                 |            |
 |<----opt. TLS---->|                 |                 |            |
 |<-----tStatus-----|                 |                 |            |
 |------pStatus---->|                 |                 |            |
 |                  |                 |                 |            |
 ~                  ~                 ~                 ~            ~
~~~~
{: #exchangesfig_uc2_all title='Overview pledge-responder-mode exchanges' artwork-align="center"}

The following sub sections split the interactions shown in {{exchangesfig_uc2_all}} between the different components into:

1. {{tpvr}} describes the acquisition exchange for the Pledge Voucher-Request initiated by the Registrar-Agent to the pledge.

2. {{tper}} describes the acquisition exchange for the Pledge Enroll-Request initiated by the Registrar-Agent to the pledge.

3. {{pvr}} describes the issuing exchange for the Voucher initiated by the Registrar-Agent to the registrar, including the interaction of the registrar with the MASA using the RVR {{rvr-proc}}, as well as the artifact processing by these entities.

4. {{per}} describes the enroll exchange initiated by the Registrar-Agent to the registrar including the interaction of the registrar with the CA using the PER as well as the artifact processing by these entities.

5. {{req_cacerts}} describes the retrival exchange for the optional CA certificate provisioning to the pledge initiated by the Registrar-Agent to the CA. 

6. {{voucher}} describes the Voucher exchange initiated by the Registrar-Agent to the pledge and the returned status information.

7. {{cacerts}} describes the certificate provisioning exchange initiated by the Registrar-Agent to the pledge. 

8. {{enroll_response}} describes the Enroll-Response exchange (containing the LDevID (Pledge) certificate) initiated by the Registrar-Agent to the pledge and the returned status information.

9. {{vstatus}} describes the Voucher status telemetry exchange initiated by the Registrar-Agent to the registrar, including the interaction of the registrar with the MASA.

10. {{estatus}} describes the Enroll Status telemetry exchange initiated by the Registrar-Agent to the registrar.

11. {{query}} describes the Pledge Status exchange about the general bootstrapping state initiated by the Registrar-Agent to the pledge.
   


## Trigger Pledge Voucher-Request {#tpvr}

This exchange assumes that the Registrar-Agent has already discovered the pledge.
This may be done as described in {{discovery_uc2_ppa}} and {{exchangesfig_uc2_all}} based on DNS-SD or similar.

Optionally, TLS MAY be used to provide privacy for this exchange between the Registrar-Agent and the pledge, see {{pledgehttps}}.

{{exchangesfig_uc2_1}} shows the acquisition of the Pledge Voucher-Request (PVR) and the following subsections describe the corresponding artifacts. 

~~~~ aasvg
+--------+    +------------+    +-----------+    +--------+    +------+
| Pledge |    | Registrar- |    |  Domain   |    | Domain |    | MASA |
|        |    |   Agent    |    | Registrar |    |   CA   |    |      |
+--------+    +------------+    +-----------+    +--------+    +------+
 |                  |                 |                 |   Internet |
 ~                  ~                 ~                 ~            ~
(1) Trigger Pledge Voucher-Request
 ~                  ~                 ~                 ~            ~
 |                  |                 |                 |            |
 |<----opt. TLS---->|                 |                 |            |
 |<------tPVR-------|                 |                 |            |
 |--------PVR------>|                 |                 |            |
 |                  |                 |                 |            |
 ~                  ~                 ~                 ~            ~
~~~~
{: #exchangesfig_uc2_1 title="PVR acquisition exchange" artwork-align="center"}

The Registrar-Agent triggers the pledge to create the PVR via HTTP POST on the well-known pledge endpoint `/.well-known/brski/tpvr`.
The request body MUST contain the JSON-based Pledge Voucher-Request Trigger (tPVR) artifact.
The request header MUST set the Content-Type field to `application/json`.

Upon receiving a valid tPVR, the pledge MUST reply with the PVR artifact in the body of a 200 OK response.
The Content-Type field header of the response MUST be set to `application/voucher-jws+json` as defined in {{!I-D.ietf-anima-jws-voucher}}.

If the pledge is unable to create the PVR, it SHOULD respond with an HTTP error code. The following client error responses MAY be used:

* 400 Bad Request: if the pledge detected an error in the format of the request, e.g. missing field, wrong data types, etc. or if the request is not valid JSON even though the PVR media type was set to `application/json`.

* 406 Not Acceptable: if the Accept request header field indicates a type that is unknown or unsupported, e.g., a type other than `application/jose+json`.

* 415 Unsupported Media Type: if the Content-Type request header field indicates a type that is unknown or unsupported, e.g., a type other than `application/json`.

### Request Artifact: Pledge Voucher-Request Trigger (tPVR)

The Pledge Voucher-Request Trigger (tPVR) artifact is an unsigned JSON structure providing the trigger parameters.
The following CDDL {{!RFC8610}} explains the Pledge Voucher-Request Trigger structure. 


~~~~ cddl
  pledgevoucherrequesttrigger = {
    "agent-provided-proximity-registrar-cert": bytes,
    "agent-signed-data": bytes
  }
~~~~
{: #tpvr_CDDL_def title='CDDL for Pledge Voucher-Request Trigger' artwork-align="left" sourcecode-markers=“true”}


The fields contained in the `pledgevoucherrequesttrigger` are:

* `agent-provided-proximity-registrar-cert`: X.509 v3 certificate structure of the domain registrar EE certificate (base64-encoded value); may be configured at the Registrar-Agent or may be fetched by the Registrar-Agent based on a prior TLS connection with this domain registrar

* `agent-signed-data`: base64-encoded JWS structure containing the SubjectKeyIdentifier of the EE (RegAgt) certificate and signing Data including the creation date and serial number of the pledge. Note that {{I-D.ietf-anima-rfc8366bis}} defines an opaque binary element for agent-signed data, for which the structure is defined in BRSKI-PRM.

~~~~
{
  "payload": BASE64URL(UTF8(prmasd)),
  "signatures": [
    {
      "protected": BASE64URL(UTF8(JWS Protected Header)),
      "signature": BASE64URL(JWS Signature)
    }
  ]
}
~~~~
{: #asd title="JWS structure for the agent-signed-data member in General JWS Serialization syntax" artwork-align="left"}

The BRSKI-PRM Agent Signed Data structure MUST be encoded in JSON as defined in {{!RFC8259}} following the CDDL definition {{prmasd_CDDL_def}}.
The JWS Payload is further base64url-encoded to become the string value of the `payload` member as described in {{Section 3.2 of RFC7515}}.

The following CDDL {{!RFC8610}} explains the BRSKI-PRM Agent Signed Data structure. 

~~~~ cddl
  prmasd = {
    "created": tdate,
    "serial-number": text
  }
~~~~
{: #prmasd_CDDL_def title='CDDL for BRSKI-PRM Agent Signed Data' artwork-align="left" sourcecode-markers=“true”}

The fields contained in the `prmasd` are:

* `created-on`: creation date and time as standard date/time string as defined in {{!RFC3339}} 

* `serial-number`: product-serial-number in the X520SerialNumber field of the IDevID certificate of the pledge as string as defined in {{Section 2.3.1 of !RFC8995}}

{{prmasd_payload}} below shows an example for unsigned BRSKI-PRM Agent Signed Data in JSON syntax. 

~~~~
{
  "created-on": "2021-04-16T00:00:01.000Z",
  "serial-number": "callee4711"
}
~~~~
{: #prmasd_payload title="Data example for prmasd" artwork-align="left"}

The JWS Protected Header of the `agent-signed-data` JWS structure MUST contain the following parameters (see {{asd_header}} for an example):

* `alg`: algorithm type used to create the signature, e.g., `ES256` as defined in {{Section 4.1.1 of RFC7515}}

* `kid`: base64-encoded bytes of the SubjectKeyIdentifier (the "KeyIdentifier" OCTET STRING value) of the EE (RegAgt) certificate.

~~~~
{
  "alg": "ES256",
  "kid": "base64encodedvalue=="
}
~~~~
{: #asd_header title="Protected Header example inside agent-signed-data" artwork-align="left"}

Note that at the time of receiving the PVR trigger, the pledge cannot verify the registrar LDevID certificate and has no proof-of-possession of the corresponding private key for the certificate.
Hence, the tPVR is an unsigned artifact and the pledge only accepts the registrar LDevID certificate provisionally until it receives the voucher as described in {{voucher}}.

The pledge will also be unable to verify the agent-signed-data itself as it does not possess the EE (RegAgt) certificate and the domain trust has not been established at this point of the communication.
Verification SHOULD be done, after the voucher has been received.


### Response Artifact: Pledge Voucher-Request (PVR)

The Pledge Voucher-Request (PVR) artifact is a JWS Voucher Request as defined in {{I-D.ietf-anima-jws-voucher}}.
Its unsigned data SHALL be constructed similar to the Voucher-Request artifact defined in {{!RFC8995}}.
It will contain additional data provided by the Registrar-Agent as specified in the following.

The payload of the PVR MUST contain the following parameters as part of the ietf-voucher-request:voucher as defined in {{I-D.ietf-anima-rfc8366bis}} and thus makes optional leaves in the YANG definition mandatory:

* `created-on`: SHALL contain the current date and time in yang:date-and-time format.
  If the pledge does not have synchronized time, it SHALL use the created-on time from the agent-signed-data, received in the trigger to create a PVR.

* `nonce`: SHALL contain a cryptographically strong pseudo-random number.

* `serial-number`: SHALL contain the pledge product-serial-number as X520SerialNumber.

* `assertion`: SHALL contain the requested voucher assertion "agent-proximity" (different value as in RFC 8995)..

The ietf-voucher-request:voucher data is extended with two additional parameters that MUST be included:

* `agent-provided-proximity-registrar-cert`: base64-encoded registrar EE certificate (provided in tPVR by the Registrar-Agent); enables the registrar to verify that it is the desired registrar for handling the PVR

* `agent-signed-data`: base64-encoded agent-signed-data (provided in tPVR by the Registrar-Agent); enables the registrar to verify and log, which Registrar-Agent was in contact with the pledge, when verifying the PVR

The enhancements of the YANG module for the ietf-voucher-request with these new leaves are defined in {{I-D.ietf-anima-rfc8366bis}}.

The PVR is signed using the pledge's IDevID credential contained as x5c parameter of the JOSE header.

~~~~
# The PVR in General JWS Serialization syntax
{
  "payload": BASE64URL(UTF8(ietf-voucher-request:voucher)),
  "signatures": [
    {
      "protected": BASE64URL(UTF8(JWS Protected Header)),
      "signature": BASE64URL(JWS Signature)
    }
  ]
}

# Example: Decoded Payload "ietf-voucher-request:voucher"
  representation in JSON syntax
{
  "ietf-voucher-request:voucher": {
     "created-on": "2021-04-16T00:00:02.000Z",
     "nonce": "eDs++/FuDHGUnRxN3E14CQ==",
     "serial-number": "callee4711",
     "assertion": "agent-proximity",
     "agent-provided-proximity-registrar-cert": "base64encodedvalue==",
     "agent-signed-data": "base64encodedvalue=="
  }
}

# Example: Decoded "JWS Protected Header" representation
  in JSON syntax
{
    "alg": "ES256",
    "typ": "voucher-jws+json",
    "x5c": [
      "base64encodedvalue==",
      "base64encodedvalue=="
    ]
}
~~~~
{: #pvr_example title='Representation of PVR' artwork-align="left"}



## Trigger Pledge Enroll-Request {#tper}

Once the Registrar-Agent has received the PVR it can trigger the pledge to generate a Pledge Enroll-Request (PER).

Optionally, TLS MAY be used to provide privacy for this exchange between the Registrar-Agent and the pledge, see {{pledgehttps}}.

{{exchangesfig_uc2_2}} shows the the acquisition of the PER and the following subsections describe the corresponding artifacts. 

~~~~ aasvg
+--------+    +------------+    +-----------+    +--------+    +------+
| Pledge |    | Registrar- |    |  Domain   |    | Domain |    | MASA |
|        |    |   Agent    |    | Registrar |    |   CA   |    |      |
+--------+    +------------+    +-----------+    +--------+    +------+
 |                  |                 |                 |   Internet |
 ~                  ~                 ~                 ~            ~
(2) Trigger Pledge Enroll-Request
 ~                  ~                 ~                 ~            ~
 |                  |                 |                 |            |
 |<----opt. TLS---->|                 |                 |            |
 |<------tPER-------|                 |                 |            |
 |--------PER------>|                 |                 |            |
 |                  |                 |                 |            |
 ~                  ~                 ~                 ~            ~
~~~~
{: #exchangesfig_uc2_2 title="PER acquisition exchange" artwork-align="center"}

The Registrar-Agent triggers the pledge to create the PER via HTTP POST on the well-known pledge endpoint `/.well-known/brski/tper`.
As the initial enrollment aims to request a generic certificate, no certificate attributes are provided to the pledge.
To avoid an empty request body an artifact is provided containing the description of the requested operation.

Upon receiving a valid tPER, the pledge MUST reply with the PER artifact in the body of a 200 OK response.
The response header MUST have the Content-Type field set to `application/jose+json`.

If the pledge is unable to create the PER, it SHOULD respond with an HTTP error code. The following 4xx client error codes MAY be used:

* 400 Bad Request: if the pledge detected an error in the format of the request.

* 406 Not Acceptable: if the Accept request header field indicates a type that is unknown or unsupported. For example, a type other than `application/jose+json`.

* 415 Unsupported Media Type: if the Content-Type request header field indicates a type that is unknown or unsupported, e.g., a type other than `application/json`.

### Request Artifact: Pledge Enroll-Request Trigger (tPER)

This document specifies the trigger for a generic certificate with no CSR attributes provided to the pledge.
If specific attributes in the certificate are required, they have to be inserted by the issuing RA/CA.

The Pledge Enroll-Request Trigger (tPVR) artifact is an unsigned JSON structure providing the trigger parameters (tPER-data).
The following CDDL {{!RFC8610}} explains the Pledge Enroll-Request Trigger structure. 

~~~~ cddl
pledgeenrollrequesttrigger = {
    "enroll-type": "enroll-generic-cert"
  }
~~~~
{: #tper_CDDL_def title='CDDL for Pledge Enroll-Request Trigger' artwork-align="left" sourcecode-markers=“true”}

The enroll-type field is an enum, identifying what is being enrolled. 
Currently only "enroll-generic-cert" for the LDevID certificate is defined. 

{{tPER_payload}} below shows an example for unsigned Pledge Enroll-Request Trigger in JSON syntax. 

~~~~
{
  "enroll-type" : "enroll-generic-cert"
}
~~~~
{: #tPER_payload title="Data example for pledgeenrollrequesttrigger" artwork-align="left"}


The Pledge Enroll-Request Trigger (tPER) artifact MUST be encoded in JSON as defined in {{!RFC8259}} following the CDDL definition {{tper_CDDL_def}}.

The Pledge Enroll-Request Trigger (tPER) artifact MAY be used to provide additional data, like CSR attributes.
How to provide and use such additional data is out of scope for this specification.

### Response Artifact: Pledge Enroll-Request (PER)

The Pledge Enroll-Request (PER) artifact is a JWS-signed PKCS#10 Certificate Signing Request (CSR) utilizing the csr-grouping of the `ietf-ztp-types` YANG module as defined in {{!I-D.ietf-netconf-sztp-csr}}.
The CSR already assures POP of the private key corresponding to the contained public key.
In addition, based on the PER signature using the IDevID, POI is provided.

The pledge constructs the Pledge Enroll-Request (PER) artifact as a JWS structure containing the PKCS#10 request wrapped in ietf-ztp-types YANG structrue as JWS payload.
Note, {{!I-D.ietf-netconf-sztp-csr}} also allows for inclusion of certification requests in different formats used by CMP or CMC.

The pledge MUST construct the PER as PKCS#10 and MUST sign it additionally with its IDevID credentials to provide proof-of-identity bound to the PKCS#10 as described below.

A successful enrollment will result in a generic LDevID certificate for the pledge in the new domain.
This generic LDevID certificate can be used to request further (application specific) LDevID certificates if necessary for operation.
The Registrar-Agent SHALL use the enrollment endpoint `requestenroll` specified in this document to provide the Pledge Enroll-Request artifact to the Registrar.

The JWS Protected Header of the PER MUST contain the following parameters as defined in {{RFC7515}}:

* `alg`: algorithm type used to create the signature, e.g., `ES256` as defined in {{Section 4.1.1 of RFC7515}}

* `x5c`: base64-encoded pledge IDevID certificate;
  it MAY optionally contain the certificate chain for this certificate; if the certificate chain is not included, it MUST be available at the registrar for verification of the IDevID certificate

The body of the Pledge Enroll-Request SHOULD contain a P10 parameter (for PKCS#10) as defined for ietf-ztp-types:p10-csr in {{I-D.ietf-netconf-sztp-csr}}:

* `p10-csr`: base64-encoded PKCS#10 of the pledge.

The JOSE object is signed using the pledge's IDevID credential, which corresponds to the certificate signaled in the JOSE header.

While BRSKI-PRM targets the initial enrollment, re-enrollment SHOULD be supported as described in a similar way as for enrollment in this document, if no other re-enrollment mechanism is supported.
Note that in this case the current LDevID credential is used instead of the IDevID credential to create the signature of the PKCS#10 request.

~~~~
# The PER in General JWS Serialization syntax
{
  "payload": "BASE64URL(ietf-ztp-types)",
  "signatures": [
    {
      "protected": "BASE64URL(UTF8(JWS Protected Header))",
      "signature": BASE64URL(JWS Signature)
    }
  ]
}

# Example: Decoded Payload "ietf-ztp-types" Representation
  in JSON Syntax
{
  "ietf-ztp-types": {
     "p10-csr": "base64encodedvalue=="
   }
}

# Example: Decoded "JWS Protected Header" Representation
  in JSON Syntax
{
  "alg": "ES256",
  "x5c": [
    "base64encodedvalue==",
    "base64encodedvalue=="
  ],
  "crit":["created-on"],
  "created-on": "2022-09-13T00:00:02.000Z"
}
~~~~
{: #per_example title='Representation of PER' artwork-align="left"}

With the collected PVR and PER, the Registrar-Agent starts the interaction with the domain registrar.

The new protected header field "created-on" is introduced to reflect freshness of the PER.
The field is marked critical "crit" to ensure that it must be understood and validated by the receiver (here the domain registrar) according to {{Section 4.1.11 of RFC7515}}.
It allows the registrar to verify the timely correlation between the PER and previously exchanged messages, i.e., created-on of PER >= created-on of PVR >= created-on of PVR trigger.
The registrar MAY consider to ignore any but the newest PER from the same pledge in the case the registrar has at any point in time more than one pending PER from the pledge.

As the Registrar-Agent is intended to facilitate communication between the pledge and the domain registrar, a collection of requests from more than one pledge is possible.
This allows bulk bootstrapping of several pledges using the same connection between the Registrar-Agent and the domain registrar.



## Supply PVR to Registrar (including backend interaction) {#pvr}

Similar to BRSKI "requestvoucher" endpoint in {{Section 5.2 of !RFC8995}}.

The Registrar-Agent has acquired one or more PVR and PER object pairs

The Registrar-Agent establishes a TLS connection to the registrar.
As already stated in {{!RFC8995}}, the use of TLS 1.3 (or newer) is encouraged.
TLS 1.2 or newer is REQUIRED on the Registrar-Agent side.
TLS 1.3 (or newer) SHOULD be available on the registrar, but TLS 1.2 MAY be used.
TLS 1.3 (or newer) SHOULD be available on the MASA, but TLS 1.2 MAY be used.

In contrast to BRSKI {{RFC8995}} TLS client authentication to the registrar is achieved by using Registrar-Agent EE credentials instead of pledge IDevID credentials.
Consequently BRSKI (pledge-initiator-mode) is distinguishable from BRSKI-PRM (pledge-responder-mode) by the registrar.
The registrar SHOULD verify that the Registrar-Agent is authorized to establish a connection to the registrar based on the TLS client authentication.
If the connection from Registrar-Agent to registrar is established, the authorization SHOULD be verified again based on agent-signed-data contained in the PVR.
This ensures that the pledge has been triggered by an authorized Registrar-Agent.

With BRSKI-PRM, the pledge generates PVR and PER as JSON-in-JWS objects and the Registrar-Agent forwards them to the registrar.
In {{!RFC8995}}, the pledge generates PVR as CMS-signed JSON and PER as PKCS#10 or PKCS#7 according to {{!RFC7030}} and inherited by {{!RFC8995}}.

{{exchangesfig_uc2_3}} shows the exchanges for the Voucher Request processing and the following subsections describe the corresponding artifacts. 

~~~~ aasvg
+--------+    +------------+    +-----------+    +--------+    +------+
| Pledge |    | Registrar- |    |  Domain   |    | Domain |    | MASA |
|        |    |   Agent    |    | Registrar |    |   CA   |    |      |
+--------+    +------------+    +-----------+    +--------+    +------+
 |                  |                 |                 |   Internet |
 ~                  ~                 ~                 ~            ~
(3) Supply PVR to Registrar (including backend interaction)
 ~                  ~                 ~                 ~            ~
 |                  |                 |                 |            |
 |                  |<-----mTLS------>|                 |            |
 |                  |         [Registrar-Agent          |            |
 |                  |    authenticated&authorized?]     |            |
 |                  |-------PVR------>|                 |            |
 |                  |          [accept device?]         |            |
 |                  |          [contact vendor]         |            |
 |                  |                 |                 |            |
 |                  |                 |<------------mTLS------------>|
 |                  |                 |--------------RVR------------>|
 |                  |                 |              [extract DomainID]
 |                  |                 |              [update audit log]
 |                  |                 |<-----------Voucher-----------|
 |                  |<----Voucher-----|                 |            |
 |                  |                 |                 |            |
 ~                  ~                 ~                 ~            ~
~~~~
{: #exchangesfig_uc2_3 title="Voucher issuing exchange" artwork-align="center"}

The HTTP request Content-Type header field for JSON-in-JWS PVR is: `application/voucher-jws+json` (see {{tpvr}} for the content definition), as defined in {{I-D.ietf-anima-jws-voucher}}.

The Registrar-Agent sets the Accept field in the request-header indicating the acceptable Content-Type for the Voucher.

The HTTP response Content-Type header field is set to `application/voucher-jws+json` as defined in {{I-D.ietf-anima-jws-voucher}} if no content negotiation is used.


### Request Artifact: Pledge Voucher-Request (PVR)

For BRSKI-PRM, the Registrar-Agent sends the PVR by HTTP POST to the same registrar endpoint as introduced by BRSKI: "/.well-
known/brski/requestvoucher", but with a Content-Type header field for JSON-in-JWS".

After receiving the PVR from Registrar-Agent, the registrar SHALL perform the verification as defined in {{Section 5.3 of RFC8995}}.
In addition, the registrar SHALL verify the following parameters from the PVR:

* agent-provided-proximity-registrar-cert: MUST contain registrar's own registrar LDevID certificate to ensure the registrar in proximity of the Registrar-Agent is the desired registrar for this PVR.

* agent-signed-data: The registrar MUST verify that the Registrar-Agent provided data has been signed with the private key corresponding to the EE (RegAgt) certificate indicated in the "kid" JOSE header parameter.
  The registrar MUST verify that the LDevID(ReAgt) certificate, corresponding to the signature, is still valid.
  If the certificate is already expired, the registrar SHALL reject the request.
  Validity of used signing certificates at the time of signing the agent-signed-data is necessary to avoid that a rogue Registrar-Agent generates agent-signed-data objects to onboard arbitrary pledges at a later point in time, see also {{sec_cons_reg-agt}}.
  The registrar MUST fetch the EE (RegAgt) certificate, based on the provided SubjectKeyIdentifier (SKID) contained in the "kid" header parameter of the agent-signed-data, and perform this verification.
  This requires, that the registrar has access to the EE (RegAgt) certificate data (including intermediate CA certificates if existent) based on the SKID.
  Note, the registrar may have stored the EE (RegAgt) certificate if used during TLS establishment between Registrar-Agent and registrar or it may be provided via a repository.

If the registrar is unable to validate the PVR, it SHOULD respond with a HTTP 4xx/5xx error code to the Registrar-Agent.

The following 4xx client error codes SHOULD be used:

* 403 Forbidden: if the registrar detected that one or more security related parameters are not valid or if the pledge-provided information could not be used with automated allowance.

* 406 Not Acceptable: if the Content-Type indicated by the Accept header is unknown or unsupported.

If the validation succeeds, the registrar performs pledge authorization according to {{Section 5.3 of !RFC8995}} followed by obtaining a voucher from the pledge's MASA according to {{Section 5.4 of !RFC8995}} with the modifications described below in {{rvr-proc}}.

### Supply RVR to MASA (backend interaction) {#rvr-proc}

The registrar needs to convert the PVR to an RVR and supply it to the MASA.

If the MASA address/URI is learned from the IDevID MASA URI extension ({{Section 2.3 of !RFC8995}}), then the MASA on that URI MUST support the procedures defined in this document if the PVR used JSON-JWS encoding.
If the MASA is only configured on the registrar, then a registrar supporting BRKSI-PRM and other voucher encoding formats (such as those in {{!RFC8995}}) SHOULD support per-message-format MASA address/URI configuration for the same IDevID trust anchor."

The registrar SHALL construct the payload of the RVR as defined in {{!RFC8995}}, Section 5.5.
The RVR encoding SHALL be JSON-in-JWS as defined in {{I-D.ietf-anima-jws-voucher}}.

The header of the RVR SHALL contain the following parameter as defined for JWS {{RFC7515}}:

* alg: algorithm used to create the object signature

* x5c: base64-encoded registrar LDevID certificate(s)
  (It optionally contains the certificate chain for this certificate)

The payload of the RVR MUST contain the following parameter as part of the voucher-request as defined in {{!RFC8995}}:

* created-on: current date and time in yang:date-and-time format of RVR creation

* nonce: copied from the PVR

* serial-number: product-serial-number of pledge.
  The registrar MUST verify that the IDevID certificate subject serialNumber of the pledge (X520SerialNumber) matches the serial-number value in the PVR.
  In addition, it MUST be equal to the serial-number value contained in the agent-signed data of PVR.

* assertion: voucher assertion requested by the pledge (agent-proximity).
  The registrar provides this information to assure successful verification of Registrar-Agent proximity based on the agent-signed-data.

* prior-signed-voucher-request: PVR as received from Registrar-Agent, see {{tpvr}}

The RVR MUST be extended with the following parameter, when the assertion "agent-proximity" is requested, as defined in {{I-D.ietf-anima-rfc8366bis}}:

* agent-sign-cert: EE (RegAgt) certificate or the EE (RegAgt) certificate including certificate chain.
  In the context of this document it is a JSON array of base64encoded certificate information and handled in the same way as x5c header objects.
  If only a single object is contained in the x5c it MUST be the base64-encoded EE (RegAgt) certificate.
  If multiple certificates are included in the x5c, the first MUST be the base64-encoded EE (RegAgt) certificate.

The MASA uses this information for verification that the Registrar-Agent is in proximity to the registrar to state the corresponding assertion "agent-proximity".

The object is signed using the registrar LDevID credentials, which corresponds to the certificate referenced in the JOSE header.

~~~~
# The RVR in General JWS Serialization syntax
{
  "payload": BASE64URL(UTF8(ietf-voucher-request:voucher)),
  "signatures": [
    {
      "protected": BASE64URL(UTF8(JWS Protected Header)),
      "signature": BASE64URL(JWS Signature)
    }
  ]
}

# Example: Decoded payload "ietf-voucher-request:voucher"
  representation in JSON syntax
{
  "ietf-voucher-request:voucher": {
     "created-on": "2022-01-04T02:37:39.235Z",
     "nonce": "eDs++/FuDHGUnRxN3E14CQ==",
     "serial-number": "callee4711",
     "assertion": "agent-proximity",
     "prior-signed-voucher-request": "base64encodedvalue==",
     "agent-sign-cert": [
       "base64encodedvalue==",
       "base64encodedvalue==",
       "..."
     ]
  }
}

# Example: Decoded "JWS Protected Header" representation
  in JSON syntax
{
  "alg": "ES256",
  "x5c": [
    "base64encodedvalue==",
    "base64encodedvalue=="
  ],
  "typ": "voucher-jws+json"
}
~~~~
{: #rvr title='Representation of RVR' artwork-align="left"}

The registrar SHALL send the RVR to the MASA endpoint by HTTP POST: "/.well-known/brski/requestvoucher"

The RVR Content-Type header field is defined in {{!I-D.ietf-anima-jws-voucher}} as: `application/voucher-jws+json`

The registrar SHOULD set the Accept header of the RVR indicating the desired media type for the voucher-response.
The media type is `application/voucher-jws+json` as defined in {{I-D.ietf-anima-jws-voucher}}.

This document uses the JSON-in-JWS format throughout the definition of exchanges and in the examples.
Nevertheless, alternative encodings of the voucher as used in BRSKI {{!RFC8995}} with JSON-in-CMS or CBOR-in-COSE_Sign {{?RFC9052}} for constraint environments are possible as well.
The assumption is that a pledge typically supports a single encoding variant and creates the PVR in the supported format.
To ensure that the pledge is able to process the voucher, the registrar MUST use the media type for Accept header in the RVR based on the media type used for the PVR.

Once the MASA receives the RVR it SHALL perform the verification as described in {{Section 5.5 of !RFC8995}}.

In addition, the following processing SHALL be performed for PVR contained in RVR "prior-signed-voucher-request" field:

* agent-provided-proximity-registrar-cert: The MASA MAY verify that this field contains the registrar LDevID certificate.
  If so, it MUST correspond to the registrar LDevID credentials used to sign the RVR.
  Note: Correspond here relates to the case that a single registrar LDevID certificate is used or that different registrar LDevID certificates are used, which are issued by the same CA.

* agent-signed-data: The MASA MAY verify this data to issue "agent-proximity" assertion.
  If so, the agent-signed-data MUST contain the pledge product-serial-number, contained in the "serial-number" field of the PVR (from "prior-signed-voucher-request" field) and also in "serial-number" field of the RVR.
  The EE (RegAgt) certificate to be used for signature verification is identified by the "kid" parameter of the JOSE header.
  If the assertion "agent-proximity" is requested, the RVR MUST contain the corresponding EE (RegAgt) certificate data in the "agent-sign-cert" field of the RVR.
  It MUST be verified by the MASA to the same domain CA as the registrar LDevID certificate.
  If the "agent-sign-cert" field is not set, the MASA MAY state a lower level assertion value, e.g.: "logged" or "verified".
  Note: Sub-CA certificate(s) MUST also be carried by "agent-sign-cert", in case the EE (RegAgt) certificate is issued by a sub-CA and not the domain CA known to the MASA.
  As the "agent-sign-cert" field is defined as array (x5c), it can handle multiple certificates.

If validation fails, the MASA SHOULD respond with an HTTP 4xx client error status code to the registrar.
The HTTP error status codes are kept the same as defined in {{Section 5.6 of !RFC8995}} and comprise the codes: 403, 404, 406, and 415.

The registrar provides the EE certificate of the Registrar-Agent identified by the SubjectKeyIdentifier (SKID) in the header of the "agent-signed-data" from the PVR in its RVR (see also {{rvr-proc}}).

The MASA in turn verifies the registrar LDevID certificate is included in the PVR (contained in the "prior-signed-voucher-request" field of RVR) in the "agent-provided-proximity-registrar-cert" leaf and may assert the PVR as "verified" or "logged".

In addition, the MASA may issue the assertion "agent-proximity" as follows:
The MASA verifies the signature of the "agent-signed-data" contained in the "prior-signed-voucher-request", based on the provided EE certificate of the Registrar-Agent in the "agent-sign-cert" leaf of the RVR.
If both can be verified successfully, the MASA can assert "agent-proximity" in the voucher.
The assertion of "agent-proximity" is similar to the proximity assertion by the MASA when using BRSKI.
Note that the different assertions do not provide a metric of strength as the security properties are not comparable.

Depending on the MASA verification policy, it may also respond with a suitable 4xx or 5xx response status codes as described in {{Section 5.6 of !RFC8995}}.
When successful, the Voucher will then be supplied via the registrar to the Registrar-Agent.

### Issue Voucher by MASA (backend interaction) {#exchanges_uc2_2_vc}

The MASA creates a voucher with Media-Type of `application/voucher-jws+json` as defined in {{I-D.ietf-anima-jws-voucher}}.
If the MASA detects that the Accept header of the PVR does not match `application/voucher-jws+json` it SHOULD respond with the HTTP status code "406 Not Acceptable" as the pledge will not be able to parse the response.
The voucher is according to {{I-D.ietf-anima-rfc8366bis}} but uses the new assertion value specified {{agt_prx}}.

{{MASA-vr}} shows an example of the contents of a voucher.

~~~~
# The MASA issued voucher in General JWS Serialization syntax
{
  "payload": BASE64URL(UTF8(ietf-voucher:voucher)),
  "signatures": [
    {
      "protected": BASE64URL(UTF8(JWS Protected Header)),
      "signature": BASE64URL(JWS Signature)
    }
  ]
}

# Example: Decoded payload "ietf-voucher:voucher" representation
  in JSON syntax
{
  "ietf-voucher:voucher": {
    "assertion": "agent-proximity",
    "serial-number": "callee4711",
    "nonce": "base64encodedvalue==",
    "created-on": "2022-01-04T00:00:02.000Z",
    "pinned-domain-cert": "base64encodedvalue=="
  }
}

# Example: Decoded "JWS Protected Header" representation
  in JSON syntax
{
  "alg": "ES256",
  "x5c": [
    "base64encodedvalue==",
    "base64encodedvalue=="
  ],
  "typ": "voucher-jws+json"
}
~~~~
{: #MASA-vr title='Representation of MASA issued voucher' artwork-align="left"}

The pinned-domain certificate to be put into the voucher is determined by the MASA as described in {{Section 5.5 of !RFC8995}}.
The MASA returns the voucher-response (voucher) to the registrar.


### Supply Voucher to Registrar (backend interaction) {#exchanges_uc2_2_vs}

After receiving the voucher from the MASA, the registrar SHOULD evaluate it for transparency and logging purposes as outlined in {{Section 5.6 of !RFC8995}}.
The registrar then prepares the artifact to be provided via the registrar-agent to the pledge as described in the following section {{exchanges_uc2_2_reg_signed_voucher}}. 


### Response Artifact: Registrar countersigned Voucher {#exchanges_uc2_2_reg_signed_voucher}
The registrar MUST add an additional signature to the MASA provided voucher using its registrar EE credentials.
The signature is created by signing the original "JWS Payload" produced by MASA and the registrar added "JWS Protected Header" using the registrar EE credentials (see {{RFC7515}}, Section 5.2 point 8.
The x5c component of the "JWS Protected Header" MUST contain the registrar EE certificate as well as potential subordinate CA certificates up to (but not including) the pinned domain certificate.
The pinned domain certificate is already contained in the voucher payload ("pinned-domain-cert").

(For many installations, with a single registrar credential, the registrar credential is what is pinned)

In {{!RFC8995}}, the Registrar proved possession of the it's credential when the TLS session was setup.
While the pledge could not, at the time, validate the certificate truly belonged the registrar, it did validate that the certificate it was provided was able to authenticate the TLS connection.

In the BRSKI-PRM mode, with the Registrar-Agent mediating all communication, the Pledge has not as yet been able to witness that the intended Registrar really does possess the relevant private key.
This second signature provides for the same level of assurance to the pledge, and that it matches the public key (of the Registrar) that the pledge received in the trigger for the PVR (see {{tpvr_CDDL_def}}).

The registrar MUST use the same registrar EE credentials used for authentication in the TLS handshake to authenticate towards the Registrar-Agent.
This has some operational implications when the registrar may be part of a scalable framework as described in {{?I-D.richardson-anima-registrar-considerations, Section 1.3.1}}.

The second signature MUST either be done with the private key associated with the registrar EE certificate provided to the Registrar-Agent, or the use of a certificate chain is necessary.
This ensures that the same registrar EE certificate can be used to verify the signature as transmitted in the voucher-request as also transferred in the PVR in the "agent-provided-proximity-registrar-cert".

{{MASA-REG-vr}} below provides an example of the voucher with two signatures.

~~~~
# The MASA issued voucher with additional registrar signature in
  General JWS Serialization syntax
{
  "payload": BASE64URL(ietf-voucher:voucher),
  "signatures": [
    {
      "protected": BASE64URL(UTF8(JWS Protected Header (MASA))),
      "signature": BASE64URL(JWS Signature)
    },
    {
      "protected": BASE64URL(UTF8(JWS Protected Header (Reg))),
      "signature": BASE64URL(JWS Signature)
    }
  ]
}

# Example: Decoded payload "ietf-voucher:voucher" representation in
  JSON syntax
{
  "ietf-voucher:voucher": {
     "assertion": "agent-proximity",
     "serial-number": "callee4711",
     "nonce": "base64encodedvalue==",
     "created-on": "2022-01-04T00:00:02.000Z",
     "pinned-domain-cert": "base64encodedvalue=="
  }
}

# Example: Decoded "JWS Protected Header (MASA)" representation
  in JSON syntax
{
  "alg": "ES256",
  "typ": "voucher-jws+json",
  "x5c": [
    "base64encodedvalue==",
    "base64encodedvalue=="
  ]
}

# Example: Decoded "JWS Protected Header (Reg)" representation
  in JSON syntax
{
  "alg": "ES256",
  "x5c": [
    "base64encodedvalue==",
    "base64encodedvalue=="
  ]
}
~~~~
{: #MASA-REG-vr title='Representation of MASA issued voucher with additional registrar signature' artwork-align="left"}

Depending on the security policy of the operator, this signature can also be interpreted as explicit authorization of the registrar to install the contained trust anchor.
The registrar returns the voucher to the Registrar-Agent.


## Supply PER to Registrar (including backend interaction) {#per}

After receiving the voucher, the Registrar-Agent sends the PER to the registrar in the same HTTP-over-TLS connection. Which is similar to the PER processing described in {{Section 5.2 of !RFC8995}}.
In case the PER cannot be send in the same HTTP-over-TLS connection the Registrar-Agent may send the PER in a new HTTP-over-TLS connection. The registrar is able to correlate the PVR and the PER based on the signatures and the contained product-serial-number information.
Note, this also addresses situations in which a nonceless voucher is used and may be pre-provisioned to the pledge.

{{exchangesfig_uc2_4}} depicts exchanges for the PER request handling and the following subsections describe the corresponding artifacts. 

~~~~ aasvg
+--------+    +------------+    +-----------+    +--------+    +------+
| Pledge |    | Registrar- |    |  Domain   |    | Domain |    | MASA |
|        |    |   Agent    |    | Registrar |    |   CA   |    |      |
+--------+    +------------+    +-----------+    +--------+    +------+
 |                  |                 |                 |   Internet |
 ~                  ~                 ~                 ~            ~
(4) Supply PER to Registrar (including backend interaction)
 ~                  ~                 ~                 ~            ~
 |                  |                 |                 |            |
 |                  |<----(mTLS)----->|                 |            |
 |                  |-------PER------>|                 |            |
 |                  |                 |<-----mTLS------>|            |
 |                  |                 |-------RER------>|            |
 |                  |                 |<--Enroll-Resp---|            |
 |                  |<--Enroll-Resp---|                 |            |
 |                  |                 |                 |            |
 ~                  ~                 ~                 ~            ~
~~~~
{: #exchangesfig_uc2_4 title="Enroll exchange" artwork-align="center"}

In case the TLS connection to the registrar is already closed, the Registrar-Agent opens a new TLS connection with the registrar as stated in {{pvr}}.

### Request Artifact: Pledge Enroll-Request (PER)

As specified in {{tper}} deviating from BRSKI the PER is not a raw PKCS#10.
As the Registrar-Agent is involved in the exchange, the PKCS#10 is wrapped in a JWS object by the pledge and signed with pledge's IDevID to ensure proof-of-identity as outlined in {{per_example}}.

EST {{RFC7030}} standard endpoints (/simpleenroll, /simplereenroll, /serverkeygen, /cacerts) on the registrar cannot be used for BRSKI-PRM.
This is caused by the utilization of signature wrapped-objects in BRSKI-PRM.
As EST requires to sent a raw PKCS#10 request to e.g., "/.well-known/est/simpleenroll" endpoint, this document makes an enhancement by utilizing EST but with the exception to transport a signature wrapped PKCS#10 request.
Therefore a new endpoint for BRSKI-PRM on the registrar is defined as "/.well-known/brski/requestenroll"

The Registrar-Agent SHALL send the PER to the registrar by HTTP POST to the endpoint: "/.well-known/brski/requestenroll"

The Content-Type header of PER is: `application/jose+json`.

This is a deviation from the Content-Type header values used in {{RFC7030}} and results in additional processing at the domain registrar (as EST server).
Note, the registrar is already aware that the bootstrapping is performed in a pledge-responder-mode due to the use of the EE (RegAgt) certificate for TLS and the provided PVR as JSON-in-JWS object.

* If the registrar receives a PER with Content-Type header: `application/jose+json`, it MUST verify the wrapping signature using the certificate indicated in the JOSE header.

* The registrar verifies that the pledge's certificate (here IDevID), carried in "x5c" header field, is accepted to join the domain after successful validation of the PVR.


### Enroll Pledge by Domain CA (backend interaction)

If both succeed, the registrar utilizes the PKCS#10 request contained in the JWS object body as "P10" parameter of "ietf-sztp-csr:csr" for further processing of the Enroll-Request with the corresponding domain CA.
It creates a Registrar Enroll-Request (RER) by utilizing the protocol expected by the domain CA.

The domain registrar may either directly forward the provided PKCS#10 request to the CA or provide additional information about attributes to be included by the CA into the requested LDevID certificate.

The approach of sending this information to the CA depends on the utilized certificate management protocol between the RA and the CA and is out of scope for this document.


### Response Artifact: Enroll-Response (Enroll-Resp)

The registrar SHOULD respond with an HTTP 200 OK in the success case or fail with HTTP 4xx/5xx status codes as defined by the HTTP standard.

A successful interaction with the domain CA will result in a pledge LDevID certificate, which is then forwarded by the registrar to the Registrar-Agent using the Content-Type header: `application/pkcs7-mime`.

Note while BRSKI-PRM targets the initial enrollment, re-enrollment may be supported in a similar way with the exception that the current LDevID certificate is used instead of the IDevID certificate to verify the wrapping signature of the PKCS#10 request (see also {{tper}}).



## Request CA Certificates {#req_cacerts}

As the pledge will verify it own certificate LDevID certificate when received, it also needs the corresponding CA certificates.
This is done in EST {{RFC7030}} using the "/.well-known/est/cacerts" endpoint, which provides the CA certificates over a TLS protected connection.
BRSKI-PRM requires a signature wrapped CA certificate object, to avoid that the pledge can be provided with arbitrary CA certificates in an authorized way.
The registrar signed CA certificate object will allow the pledge to verify the authorization to install the received CA certificate(s).
As the CA certificate(s) are provided to the pledge after the voucher, the pledge has the required information (the domain certificate) to verify the wrapped CA certificate object.

{{exchangesfig_uc2_5}} shows the request and provisioning of CA certificates in the infrastructure. 
The following subsections describe the corresponding artifacts. 

~~~~ aasvg
+--------+    +------------+    +-----------+    +--------+    +------+
| Pledge |    | Registrar- |    |  Domain   |    | Domain |    | MASA |
|        |    |   Agent    |    | Registrar |    |   CA   |    |      |
+--------+    +------------+    +-----------+    +--------+    +------+
 |                  |                 |                 |   Internet |
 ~                  ~                 ~                 ~            ~
(5) Request CA Certificates
 ~                  ~                 ~                 ~            ~
 |                  |                 |                 |            |
 |                  |<----(mTLS)----->|                 |            |
 |                  |---cACert-Req--->|                 |            |
 |                  |<--cACert-Resp---|                 |            |
 |                  |                 |                 |            |
 ~                  ~                 ~                 ~            ~
~~~~
{: #exchangesfig_uc2_5 title="CA certificates retrival exchange" artwork-align="center"}

In case the TLS connection to the registrar is already closed, the Registrar-Agent opens a new TLS connection with the registrar as stated in {{pvr}}.


### Request Artifact: cACert-Request (cACert-Req)

To support Registrar-Agents requesting a signature wrapped CA certificate(s) object, a new endpoint for BRSKI-PRM is defined on the registrar: "/.well-known/brski/wrappedcacerts"

The Registrar-Agent SHALL requests the EST CA trust anchor database information (in form of CA certificates) by HTTP GET.


### Response Artifact: cACert-Response (cACert-Resp)

The Content-Type header of the response SHALL be: `application/jose+json`.

This is a deviation from the Content-Type header values used in EST {{RFC7030}} and results in additional processing at the domain registrar (as EST server).
The additional processing is to sign the CA certificate(s) information using the registrar LDevID credentials.
This results in a signed CA certificate(s) object (JSON-in-JWS), the CA certificates are provided as base64-encoded "x5bag" (see definition in {{RFC9360}}) in the JWS payload.

~~~~
# The CA certificates data with registrar signature in 
# General JWS Serialization syntax
{
  "payload": BASE64URL(certs),
  "signatures": [
    {
      "protected": BASE64URL(UTF8(JWS Protected Header)),
      "signature": BASE64URL(JWS Signature)
    }
  ]
}

# Example: Decoded payload "certs" representation in JSON syntax
{
  "x5bag": [
    "base64encodedvalue==",
    "base64encodedvalue=="
  ]
}


# Example: Decoded "JWS Protected Header" representation
  in JSON syntax
{
  "alg": "ES256",
  "x5c": [
    "base64encodedvalue==",
    "base64encodedvalue=="
  ]
}
~~~~
{: #PCAC title='Representation of CA certificate(s) data with registrar signature' artwork-align="left"}



## Supply Voucher to Pledge {#voucher}

It is assumed that the Registrar-Agent already obtained the bootstrapping response objects from the domain registrar and can supply them to the pledge:

* voucher-response - Voucher (from MASA via Registrar)
* wrapped-CA-certificate(s)-response - CA certificates
* enrollment-response - LDevID (Pledge) certificate (from CA via registrar)

To deliver these response objects, the Registrar-Agent will re-connect to the pledge.
To contact the pledge, it may either discover the pledge as described in {{discovery_uc2_ppa}} or use stored information from the first contact with the pledge.

Preconditions in addition to {{pvr}}:

* Registrar-Agent: obtained voucher and LDevID certificate and optionally IDevID CA certificates.
  The IDevID CA certificate is necessary, when the connection between the Registrar-Agent and the pledge is established using TLS to enable the Registrar-Agent to validate the pledges' IDevID certificate during the TLS handshake as described in {{tpvr}}.

The Registrar-Agent MAY optionally use TLS to protect the communication as outlined in {{tpvr}}.

The Registrar-Agent provides the information via distinct pledge endpoints as following.
{{exchangesfig_uc2_6}} shows the provisioning of the voucher to the pledge. 
The following subsections describe the corresponding artifacts. 

~~~~ aasvg
+--------+    +------------+    +-----------+    +--------+    +------+
| Pledge |    | Registrar- |    |  Domain   |    | Domain |    | MASA |
|        |    |   Agent    |    | Registrar |    |   CA   |    |      |
+--------+    +------------+    +-----------+    +--------+    +------+
 |                  |                 |                 |   Internet |
 ~                  ~                 ~                 ~            ~
(6) Supply Voucher to Pledge
 ~                  ~                 ~                 ~            ~
 |                  |                 |                 |            |
 |<----opt. TLS---->|                 |                 |            |
 |<-----Voucher-----|                 |                 |            |
 |------vStatus---->|                 |                 |            |
 |                  |                 |                 |            |
 ~                  ~                 ~                 ~            ~
~~~~
{: #exchangesfig_uc2_6 title="Voucher exchange" artwork-align="center"}

### Request Artifact: Voucher

The Registrar-Agent SHALL send the voucher-response to the pledge by HTTP POST to the endpoint: "/.well-known/brski/svr".

The Registrar-Agent voucher-response Content-Type header is `application/voucher-jws+json` and contains the voucher as provided by the MASA. An example is given in {{MASA-vr}} for a MASA  signed voucher and in {{MASA-REG-vr}} for the voucher with the additional signature of the registrar.

A nonceless voucher may be accepted as in {{!RFC8995}} and may be allowed by a manufacture's pledge implementation.

To perform the validation of several signatures on the voucher object, the pledge SHALL perform the signature verification in the following order:

  1. Verify MASA signature as described in {{Section 5.6.1 of !RFC8995}}, against pre-installed manufacturer trust anchor (IDevID).
  2. Install trust anchor contained in the voucher ("pinned-domain-cert")  provisionally
  3. Validate the LDevID(Reg) certificate received in the agent-provided-proximity-registrar-cert in the Pledge-Voucher-Request trigger request (in the field "agent-provided-proximity-registrar-cert")
  4. Verify registrar signature of the voucher similar as described in {{Section 5.6.1 of !RFC8995}}, but take the registrar certificate instead of the MASA certificate for the verification

Step3 and step 4 have been introduced in BRSKI-PRM to enable verification of LDevID(Reg) certificate and also the proof-of-possession of the corresponding private key by the registrar, which is done in BRSKI based on the established TLS channel.
If all steps stated above have been performed successfully, the pledge SHALL terminate the "PROVISIONAL accept" state for the domain trust anchor and the registrar LDevID certificate.

If an error occurs during the verification and validation of the voucher, this SHALL be reported in the reason field of the pledge voucher status.


### Response Artifact: Voucher Status (vStatus)

After voucher verification and validation the pledge MUST reply with a status telemetry message as defined in {{Section 5.7 of !RFC8995}}.
The pledge generates the voucher-status and provides it as signed JSON-in-JWS object in response to the Registrar-Agent.

The response has the Content-Type `application/jose+json` and is signed using the IDevID of the pledge as shown in {{vstat}}.
As the reason field is optional (see {{!RFC8995}}), it MAY be omitted in case of success.
The reason-context is an arbitrary JSON object that may provide additional information specific to a failure. 
The content of this field is not subject to standardization, but examples are provided in {{vstat}}. 

~~~~
# The "pledge-voucher-status" telemetry in general JWS
  serialization syntax
{
  "payload": BASE64URL(pledge-voucher-status),
  "signatures": [
    {
      "protected": BASE64URL(UTF8(JWS Protected Header)),
      "signature": BASE64URL(JWS Signature)
    }
  ]
}

# Example: Decoded payload "pledge-voucher-status" representation
  in JSON syntax for success case
{
  "version": 1,
  "status": true,
  "reason": "Voucher successfully processed",
  "reason-context": {
    "pvs-details": "JSON"
  }
}

# Example: Decoded payload "pledge-voucher-status" representation
  in JSON syntax for error case
{
  "version": 1,
  "status": false,
  "reason": "Failed to authenticate MASA certificate because
  it starts in the future (1/1/2023).",
  "reason-context": {
    "pvs-details": "Current date: 1/1/1970"
  }
}

# Example: Decoded "JWS Protected Header" representation
  in JSON syntax
{
  "alg": "ES256",
  "x5c": [
    "base64encodedvalue==",
    "base64encodedvalue=="
  ]
}
~~~~
{: #vstat title='Representation of pledge voucher status telemetry' artwork-align="left"}

If the pledge did not did not provide voucher status telemetry information after processing the voucher, the Registrar-Agent MAY query the pledge status explicitly as described in {{query}} and MAY resent the voucher depending on the Pledge status following the procedure described in {{voucher}}.



## Supply CA Certificates to Pledge {#cacerts}

{{exchangesfig_uc2_7}} shows the provisioning of the CA certificates aquired by the pledge-agent to the pledge. 
The following subsections describe the corresponding artifacts. 

~~~~ aasvg
+--------+    +------------+    +-----------+    +--------+    +------+
| Pledge |    | Registrar- |    |  Domain   |    | Domain |    | MASA |
|        |    |   Agent    |    | Registrar |    |   CA   |    |      |
+--------+    +------------+    +-----------+    +--------+    +------+
 |                  |                 |                 |   Internet |
 ~                  ~                 ~                 ~            ~
(7) Supply CA Certificates to Pledge
 ~                  ~                 ~                 ~            ~
 |                  |                 |                 |            |
 |<----opt. TLS---->|                 |                 |            |
 |<-----cACerts-----|                 |                 |            |
 |                  |                 |                 |            |
 ~                  ~                 ~                 ~            ~
~~~~
{: #exchangesfig_uc2_7 title="Certificate provisioning exchange" artwork-align="center"}


### Request Artifact:

The Registrar-Agent SHALL provide the set of CA certificates requested from the registrar to the pledge by HTTP POST to the endpoint: "/.well-known/brski/scac".

As the CA certificate provisioning is crucial from a security perspective, this provisioning SHOULD only be done, if the voucher-response has been successfully processed by pledge as reflected in the voucher status telemetry.

The CA certificates message has the Content-Type `application/jose+json` and is signed using the credential of the registrar as shown in {{PCAC}}.

The CA certificates are provided as base64-encoded "x5bag".
The pledge SHALL install the received CA certificates as trust anchor after successful verification of the registrar's signature.


### Response (no artifact)

The verification comprises the following steps the pledge MUST perform. Maintaining the order of versification steps as indicated allows to determine, which verification has already been passed:

1. Check content-type of the CA certificates message. If no Content-Type is contained in the HTTP header, the default Content-Type utilized in this document (JSON-in-JWS) is used. If the Content-Type of the response is in an unknown or unsupported format, the pledge SHOULD reply with a 415 Unsupported media type error code.
2. Check the encoding of the payload. If the pledge detects errors in the encoding of the payload, it SHOULD reply with 400 Bad Request error code.
3. Verify that the wrapped CA certificate object is signed using the registrar certificate against the pinned-domain certificate. This MAY be done by comparing the hash that is indicating the certificate used to sign the message is that of the pinned-domain certificate. If the validation against the pinned domain-certificate fails, the client SHOULD reply with a 401 Unauthorized error code. It signals that the authentication has failed and therefore the object was not accepted.
4. Verify signature of the received wrapped CA certificate object using the domain certificate contained in the voucher. If the validation of the signature fails, the pledge SHOULD reply with a 403 Forbidden. It signals that the object could not be verified and has not been accepted.
5. If the received CA certificates are not self-signed, i.e., an intermediate CA certificate, verify them against an already installed trust anchor, as described in section 4.1.3 of {{RFC7030}}.

In case of success, the pledge SHOULD reply with HTTP 200 OK without a response body.



## Supply Enroll-Response to Pledge {#enroll_response}

{{exchangesfig_uc2_8}} shows the supply of the Enroll-Response to the pledge.
The following subsections describe the corresponding artifacts. 

~~~~ aasvg
+--------+    +------------+    +-----------+    +--------+    +------+
| Pledge |    | Registrar- |    |  Domain   |    | Domain |    | MASA |
|        |    |   Agent    |    | Registrar |    |   CA   |    |      |
+--------+    +------------+    +-----------+    +--------+    +------+
 |                  |                 |                 |   Internet |
 ~                  ~                 ~                 ~            ~
(8) Supply Enroll-Response to Pledge
 ~                  ~                 ~                 ~            ~
 |                  |                 |                 |            |
 |<----opt. TLS---->|                 |                 |            |
 |<---Enroll-Resp---|                 |                 |            |
 |-----eStatus----->|                 |                 |            |
 |                  |                 |                 |            |
 ~                  ~                 ~                 ~            ~
~~~~
{: #exchangesfig_uc2_8 title="Enroll-Response exchange" artwork-align="center"}

### Request Artifact: Enroll-Response (Enroll-Resp)

The Registrar-Agent SHALL send the Enroll-Response to the pledge by HTTP(S) POST to the endpoint: "/.well-known/brski/ser".

The Content-Type header when using EST {{RFC7030}} as enrollment protocol between the Registrar-Agent and the infrastructure is `application/pkcs7-mime`.
Note: It only contains the LDevID certificate for the pledge, not the certificate chain.

Upon reception, the pledge SHALL verify the received LDevID certificate.
The pledge SHALL generate the enroll status and provide it in the response to the Registrar-Agent.
If the verification of the LDevID certificate succeeds, the status property SHALL be set to "status": true, otherwise to "status": false


### Response Artifact: Enroll Status (eStatus)

After enrollment processing the pledge MUST reply with a enrollment status telemetry message as defined in {{Section 5.9.4 of !RFC8995}}.
The enroll-status is also a signed object in BRSKI-PRM and results in form of JSON-in-JWS here.
If the pledge verified the received LDevID certificate successfully it SHALL sign the enroll-status using its new LDevID credentials as shown in {{estat}}.
In failure case, the pledge SHALL use its IDevID credentials.
{{Section 5.9.4 of !RFC8995}} specifies the enrollment status telemetry message with two optional fields for "reason" and "reason-context". 
In BRSKI-PRM the optional fields are mandated to have a clear distinction between other status messages and MUST be provided therefore.
The reason-context is an arbitrary JSON object that provides additional information specific to a failure. 
The content of this field is not subject to standardization, but examples are provided in {{estat}}. 

The following CDDL {{!RFC8610}} explains enroll-status response structure. 
It is similar as defined in {{Section 5.9.4 of !RFC8995}} with the optional fields set to mandatory as described above.

~~~~ cddl
enrollstatus-trigger = {
    "version": uint,
    "status": bool,
    "reason": text,
    "reason-context" : { * $$arbitrary-map }
  }
~~~~
{: #e_stat_res_def title='CDDL for pledge-enrollment-status response' artwork-align="left" sourcecode-markers=“true”}

The response has the Content-Type `application/jose+json`.

~~~~
# The "pledge-enroll-status" telemetry in General JWS Serialization
  syntax
{
  "payload": BASE64URL(pledge-enroll-status),
  "signatures": [
    {
      "protected": BASE64URL(UTF8(JWS Protected Header)),
      "signature": BASE64URL(JWS Signature)
    }
  ]
}

# Example: Decoded payload "pledge-enroll-status" representation
  in JSON syntax for success case
{
  "version": 1,
  "status": true,
  "reason": "Enroll-Response successfully processed",
  "reason-context": {
    "pes-details": "JSON"
  }
}

# Example: Decoded payload "pledge-voucher-status" representation
  in JSON syntax for error case
{
  "version": 1,
  "status": false,
  "reason": "Enroll-Response could not be verified.",
  "reason-context": {
    "pes-details": "no matching trust anchor"
  }
}

# Example: Decoded "JWS Protected Header" representation
  in JSON syntax
{
  "alg": "ES256",
  "x5c": [
    "base64encodedvalue==",
    "base64encodedvalue=="
  ]
}
~~~~
{: #estat title='Representation of pledge enroll status telemetry' artwork-align="left"}

Once the Registrar-Agent has collected the information, it can connect to the registrar to provide it with the status responses.



## Voucher Status Telemetry (including backend interaction) {#vstatus}

The following description requires that the Registrar-Agent has collected the status information from the pledge.
It SHALL provide the status information to the registrar for further processing.

Preconditions in addition to {{pvr}}:

* Registrar-Agent: obtained voucher status (vStatus) and enroll status (eStatus) from pledge.

~~~~ aasvg
+--------+    +------------+    +-----------+    +--------+    +------+
| Pledge |    | Registrar- |    |  Domain   |    | Domain |    | MASA |
|        |    |   Agent    |    | Registrar |    |   CA   |    |      |
+--------+    +------------+    +-----------+    +--------+    +------+
 |                  |                 |                 |   Internet |
 ~                  ~                 ~                 ~            ~
(9) Voucher Status Telemetry (including backend interaction)
 ~                  ~                 ~                 ~            ~
 |                  |                 |                 |            |
 |                  |<----(mTLS)----->|                 |            |
 |                  |-----vStatus---->|                 |            |
 |                  |                 |<-----------(mTLS)----------->|
 |                  |                 |-----req device audit log---->|
 |                  |                 |<------device audit log-------|
 |                  |        [verify audit log]         |            |
 |                  |                 |                 |            |
 ~                  ~                 ~                 ~            ~
~~~~
{: #exchangesfig_uc2_9 title="Voucher Status telemetry exchange" artwork-align="center"}~~~~ aasvg

In case the TLS connection to the registrar is already closed, the Registrar-Agent opens a new TLS connection with the registrar as stated in {{pvr}}.

The Registrar-Agent MUST provide the collected pledge voucher status to the registrar.
This status indicates if the pledge could process the voucher successfully or not.

### Request Artifact: Voucher Status (vStatus)

The Registrar-Agent sends the pledge voucher status without modification to the registrar with an HTTP-over-TLS POST using the registrar endpoint "/.well-known/brski/voucher_status". The Content-Type header is kept as `application/jose+json` as depicted in the example in {{vstat}}.

The registrar SHOULD log the transaction provided for a pledge via Registrar-Agent and include the identity of the Registrar-Agent in these logs. For log analysis the following may be considered:

-   The registrar knows the interacting Registrar-Agent from the authentication of the Registrar-Agent towards the registrar using LDevID (RegAgt) and can log it accordingly.
-   The telemetry information from the pledge can be correlated to the voucher response provided from the registrar to the Registrar-Agent and further to the pledge.
-   The telemetry information, when provided to the registrar is provided via the Registrar-Agent and can thus be correlated.

The registrar SHALL verify the signature of the pledge voucher status and validate that it belongs to an accepted device of the domain based on the contained "serial-number" in the IDevID certificate referenced in the header of the voucher status.

### Response (no artifact)

According to {{Section 5.7 of !RFC8995}}, the registrar SHOULD respond with an HTTP 200 OK without a response body in the success case or fail with HTTP 4xx/5xx status codes.
The Registrar-Agent may use the response status code to signal success/failure to the service technician operating the Registrar-Agent.
Within the server logs the server SHOULD capture this telemetry information.

The registrar SHOULD proceed with collecting and logging status information by requesting the MASA audit-log from the MASA service as described in {{Section 5.8 of !RFC8995}}.



## Enroll Status Telemetry {#estatus}

The Registrar-Agent MUST provide the pledge's enroll status to the registrar.
The status indicates the pledge could process the Enroll-Response (certificate) and holds the corresponding private key.

~~~~ aasvg
+--------+    +------------+    +-----------+    +--------+    +------+
| Pledge |    | Registrar- |    |  Domain   |    | Domain |    | MASA |
|        |    |   Agent    |    | Registrar |    |   CA   |    |      |
+--------+    +------------+    +-----------+    +--------+    +------+
 |                  |                 |                 |   Internet |
 ~                  ~                 ~                 ~            ~
(10) Enroll Status Telemetry
 ~                  ~                 ~                 ~            ~
 |                  |                 |                 |            |
 |                  |<----(mTLS)----->|                 |            |
 |                  |-----eStatus---->|                 |            |
 |                  |                 |                 |            |
 ~                  ~                 ~                 ~            ~
~~~~
{: #exchangesfig_uc2_10 title="Enroll Status telemetry exchange" artwork-align="center"}

In case the TLS connection to the registrar is already closed, the Registrar-Agent opens a new TLS connection with the registrar as stated in {{pvr}}.

### Request Artifact: Enroll Status (eStatus)

The Registrar-Agent sends the pledge enroll status without modification to the registrar with an HTTP-over-TLS POST using the registrar endpoint "/.well-known/brski/enrollstatus".
The Content-Type header is kept as `application/jose+json` as depicted in the example in {{estat}}.

The registrar MUST verify the signature of the pledge enroll status.
Also, the registrar SHALL validate that the pledge is an accepted device of the domain based on the contained product-serial-number in the LDevID certificate referenced in the header of the enroll status.
The registrar SHOULD log this event.
In case the pledge enroll status indicates a failure, the pledge was unable to verify the received LDevID certificate and therefore signed the enroll status with its IDevID credential.
Note that the signature verification of the status information is an addition to the described handling in {{Section 5.9.4 of !RFC8995}}, and is replacing the pledges TLS client authentication by DevID credentials in {{!RFC8995}}.


### Response (no artifact)

According to {{Section 5.9.4 of !RFC8995}}, the registrar SHOULD respond with an HTTP 200 OK in the success case or fail with HTTP 4xx/5xx status codes.

Based on the failure case the registrar MAY decide that for security reasons the pledge is not allowed to reside in the domain. In this case the registrar MUST revoke the certificate.
An example case for the registrar revoking the issued LDevID for the pledge is when the pledge was not able to verify the received LDevID certificate and therefore did send a 406 (Not Acceptable) response.
In this case the registrar may revoke the LDevID certificate as the pledge did no accepted it for installation.

The Registrar-Agent may use the response to signal success / failure to the service technician operating the Registrar-Agent.
Within the server log the registrar SHOULD capture this telemetry information.


## Query Pledge Status {#query}

The following assumes that a Registrar-Agent may need to query the status of a pledge.
This information may be useful to solve errors, when the pledge was not able to connect to the target domain during the bootstrapping.
The pledge MAY provide the dedicated endpoint for the Query Pledge Status operation.

~~~~ aasvg
+--------+    +------------+    +-----------+    +--------+    +------+
| Pledge |    | Registrar- |    |  Domain   |    | Domain |    | MASA |
|        |    |   Agent    |    | Registrar |    |   CA   |    |      |
+--------+    +------------+    +-----------+    +--------+    +------+
 |                  |                 |                 |   Internet |
 ~                  ~                 ~                 ~            ~
(11) Query Pledge Status
 ~                  ~                 ~                 ~            ~
 |                  |                 |                 |            |
 |<----opt. TLS---->|                 |                 |            |
 |<-----tStatus-----|                 |                 |            |
 |------pStatus---->|                 |                 |            |
 |                  |                 |                 |            |
 ~                  ~                 ~                 ~            ~
~~~~
{: #exchangesfig_uc2_11 title="Pledge Status exchange" artwork-align="center"}

The Registrar-Agent queries the Pledge Status via HTTP POST request on the well-known pledge endpoint `/.well-known/brski/qps`.
The request body MUST contain the JWS-signed Status Trigger (tStatus) artifact.
The request header MUST set the Content-Type field `application/jose+json`.

If the pledge provides the Query Pledge Status endpoint, it MUST reply to this request with the Pledge Status (pStatus) artifact in the body of a 200 OK response.
The response header MUST have the Content-Type field set to `application/jose+json`.

### Request Artifact: Status Trigger (tStatus)

The Status Query artifact is a JWS structure signing information on the requested status-type, the time and date the request is created, and the product serial-number of the pledge contacted as shown in {{stat_req_def}}.
The following Concise Data Definition Language (CDDL) {{RFC8610}} defines the structure of the unsigned Status Query data (i.e., JWS payload):

~~~~ cddl
  statustrigger = {
      "version": uint,
      "created-on": tdate,
      "serial-number": text,
      "status-type": text
  }
~~~~
{: #stat_req_def title="CDDL for unsigned Status Trigger data (statustrigger)" artwork-align="left" sourcecode-markers=“true”}

The `version` field is included to permit significant changes to the pledge status artifacts in the future.
The format and semantics in this document follow the status telemetry definitions of {{!RFC8995}}.
Hence, the version MUST be set to `1`.
A pledge (or Registrar-Agent) that receives a version larger than it knows about SHOULD log the contents and alert a human.

The `created-on` field contains a standard date/time string following {{!RFC3339}}.

The `serial-number` field takes the product-serial-number corresponding to the X520SerialNumber field of the IDevID certificate of the pledge.

The `status-type` value defined for BRSKI-PRM Status Query is `bootstrap`.
This indicates the pledge to provide current status information regarding the bootstrapping status (voucher processing and enrollment of the pledge into the new domain).

As the Status Query artifact is defined generic, it may be used by other specifications to request further status information using other status types, e.g., for onboarding to get further information about enrollment of application specific LDevIDs or other parameters.
This is out of scope for this specification.

{{stat_req_data}} below shows an example for unsigned Status Query data in JSON syntax using status-type `bootstrap`.

~~~~
{
  "version": 1,
  "created-on": "2022-08-12T02:37:39.235Z",
  "serial-number": "pledge-callee4711",
  "status-type": "bootstrap"
}
~~~~
{: #stat_req_data title="Example of unsigned Status Query data in JSON syntax using status-type bootstrap for the Status Query artifact" artwork-align="left"}

The Status Query data MUST be signed by the Registrar-Agent using its private key corresponding to the EE (RegAgt) certificate.
When using a JWS signature, the Status Query artifact looks as shown in {{stat_req}} and the Content-Type response header MUST be set to `application/jose+json`:

~~~~
{
  "payload": BASE64URL(UTF8(status-query)),
  "signatures": [
    {
      "protected": BASE64URL(UTF8(JWS Protected Header)),
      "signature": BASE64URL(JWS Signature)
    }
  ]
}
~~~~
{: #stat_req title="Status Query Representation in General JWS JSON Serialization Syntax" artwork-align="left"}

For details on `JWS Protected Header` and `JWS Signature` see {{!I-D.ietf-anima-jws-voucher}} or {{!RFC7515}}.


### Response Artifact: Pledge Status (pStatus)

When the pledge receives a Status Query with status-type `bootstrap` it SHALL respond with previously collected telemetry information (see {{vstatus}} and {{estatus}}) in a single Pledge Status artifact.


The pledge-status response message is signed with IDevID or LDevID, depending on bootstrapping state of the pledge.

The following CDDL defines the structure of the Pledge Status (pStatus) data:

~~~~ cddl
  pledgestatus = {
    "version": uint,
    "status":
      "factory-default" /
      "voucher-success" /
      "voucher-error" /
      "enroll-success" /
      "enroll-error" /
      "connect-success" /
      "connect-error",
    ?"reason" : text,
    ?"reason-context": { * $$arbitrary-map }
  }
~~~~
{: #stat_res_def title='CDDL for unsigned Pledge Status data (pledgestatus)' artwork-align="left" sourcecode-markers=“true”}

Different cases for pledge bootstrapping status may occur, which SHOULD be reflected using the status enumeration.
This document specifies the status values in the context of the bootstrapping process and credential application.
Other documents may enhance the above enumeration to reflect further status information.



* "factory-default": Pledge has not been bootstrapped.
  Additional information may be provided in the reason or reason-context.
  The pledge signs the response message using its IDevID(Pledge).
* "voucher-success": Pledge processed the voucher exchange successfully.
  Additional information may be provided in the reason or reason-context.
  The pledge signs the response message using its IDevID(Pledge).
* "voucher-error": Pledge voucher processing terminated with error.
  Additional information may be provided in the reason or reason-context.
  The pledge signs the response message using its IDevID(Pledge).
* "enroll-success": Pledge has processed the enrollment exchange successfully.
  Additional information may be provided in the reason or reason-context.
  The pledge signs the response message using its LDevID(Pledge).
* "enroll-error": Pledge enrollment-response processing terminated with error.
  Additional information may be provided in the reason or reason-context.
  The pledge signs the response message using its IDevID(Pledge).

As the pledge is assumed to utilize its bootstrapped credentials (LDevID) in communication with other peers, additional status information is provided for the connectivity to other peers, which may be helpful in analyzing potential error cases.

* "connect-success": Pledge could successfully establish a connection to another peer.
  Additional information may be provided in the reason or reason-context.
  The pledge signs the response message using its LDevID(Pledge).
* "connect-error": Pledge connection establishment terminated with error.
  Additional information may be provided in the reason or reason-context.
  The pledge signs the response message using its LDevID(Pledge).

The pledge-status responses are cumulative in the sense that connect-success implies enroll-success, which in turn implies voucher-success.
The reason-context is an arbitrary JSON object that provides additional information specific to a failure. 
The content of this field is not subject to standardization, but examples are provided in {{stat_res}}. 

{{stat_res}} provides an example for the bootstrapping-status information.


~~~~
# The pledge "status-response" in General JWS Serialization syntax
{
  "payload": BASE64URL(UTF8(status-response)),
  "signatures": [
    {
      "protected": BASE64URL(UTF8(JWS Protected Header)),
      "signature": BASE64URL(JWS Signature)
    }
  ]
}

# Example: Decoded payload "status-response" representation
  in JSON syntax
{
  "version": 1,
  "status": "enroll-success",
  "reason-context": {
    "additional" : "JSON"
  }
}

# Example: Decoded "JWS Protected Header" representation
  in JSON syntax
{
  "alg": "ES256",
  "x5c": [
    "base64encodedvalue==",
    "base64encodedvalue=="
  ],
  "typ": "jose+json
}
~~~~
{: #stat_res title='Example of pledge-status response' artwork-align="left"}

* In case "factory-default" the pledge does not possess the domain certificate resp. the domain trust-anchor.
It will not be able to verify the signature of the Registrar-Agent in the bootstrapping-status request.

* In cases "vouchered" and "enrolled" the pledge already possesses the domain certificate (has domain trust-anchor) and can therefore validate the signature of the Registrar-Agent.
If validation of the JWS signature fails, the pledge SHOULD respond with the HTTP 403 Forbidden status code.

* The HTTP 406 Not Acceptable status code SHOULD be used, if the Accept header in the request indicates an unknown or unsupported format.
* The HTTP 415 Unsupported Media Type status code SHOULD be used, if the Content-Type of the request is an unknown or unsupported format.
* The HTTP 400 Bad Request status code SHOULD be used, if the Accept/Content-Type headers are correct but nevertheless the status-request cannot be correctly parsed.

The pledge SHOULD by default only respond to requests from nodes it can authenticate (such as registrar
agent), once the pledge is enrolled with CA certificates and a matching domain certificate.




# IANA Considerations {#iana-con}

This document requires the following IANA actions.



##  BRSKI .well-known Registry

IANA is requested to enhance the Registry entitled: "BRSKI Well-Known URIs" with the following endpoints:

| Path Segment   | Description                       | Reference |
|----------------|-----------------------------------|-----------|
| requestenroll  | Supply PER to registrar           | [THISRFC] |
| wrappedcacerts | Request wrapped CA certificates   | [THISRFC] |
| tpvr           | Trigger Pledge Voucher-Request    | [THISRFC] |
| tper           | Trigger Pledge Enroll-Request     | [THISRFC] |
| svr            | Supply Voucher to pledge          | [THISRFC] |
| scac           | Supply CA certificates to pledge  | [THISRFC] |
| ser            | Supply Enroll-Response to pledge  | [THISRFC] |
| qps            | Query Pledge Status               | [THISRFC] |
|=========
{: #iana_table title='BRSKI Well-Known URIs Additions' }



##  DNS Service Names

IANA has registered the following service names:

**Service Name:** brski-pledge<br>
**Transport Protocol(s):** tcp<br>
**Assignee:** IESG <iesg@ietf.org><br>
**Contact:** IESG <iesg@ietf.org><br>
**Description:** The Bootstrapping Remote Secure Key Infrastructure Pledge<br>
**Reference:** [THISRFC]




# Privacy Considerations

In general, the privacy considerations of {{!RFC8995}} apply for BRSKI-PRM also.
Further privacy aspects need to be considered for:

* the introduction of the additional component Registrar-Agent
* potentially no transport layer security between Registrar-Agent and pledge

{{tpvr}} describes to optional apply TLS to protect the communication between the Registrar-Agent and the pledge.
The following is therefore applicable to the communication without the TLS protection.

The credential used by the Registrar-Agent to sign the data for the pledge SHOULD NOT contain any personal information.
Therefore, it is recommended to use an LDevID certificate associated with the commissioning device instead of an LDevID certificate associated with the service technician operating the device.
This avoids revealing potentially included personal information to Registrar and MASA.

The communication between the pledge and the Registrar-Agent is performed over plain HTTP.
Therefore, it is subject to disclosure by a Dolev-Yao attacker (an "oppressive observer"){{onpath}}.
Depending on the requests and responses, the following information is disclosed.

* the Pledge product-serial-number is contained in the trigger message for the PVR and in all responses from the pledge.
  This information reveals the identity of the devices being bootstrapped and allows deduction of which products an operator is using in their environment.
  As the communication between the pledge and the Registrar-Agent may be realized over wireless link, this information could easily be eavesdropped, if the wireless network is unencrypted.
  Even if the wireless network is encrypted, if it uses a network-wide key, then layer-2 attacks (ARP/ND spoofing) could insert an on-path observer into the path.
* the Timestamp data could reveal the activation time of the device.
* the Status data of the device could reveal information about the current state of the device in the domain network.




# Security Considerations {#sec_cons}

In general, the security considerations of {{!RFC8995}} apply for BRSKI-PRM also.
Further security aspects are considered here related to:

* the introduction of the additional component Registrar-Agent
* the reversal of the pledge communication direction (push mode, compared to BRSKI)
* no transport layer security between Registrar-Agent and pledge



## Denial of Service (DoS) Attack on Pledge {#sec_cons-dos}

Disrupting the pledge behavior by a DoS attack may prevent the bootstrapping of the pledge to a new domain.
Because in BRSKI-PRM, the pledge responds to requests from real or illicit Registrar-Agents, pledges are more subject to DoS attacks from Registrar-Agents in BRSKI-PRM than they are from illicit registrars in {{!RFC8995}}, where pledges do initiate the connections.

A DoS attack with a faked Registrar-Agent may block the bootstrapping of the pledge due changing state on the pledge (the pledge may produce a voucher-request, and refuse to produce another one).
One mitigation may be that the pledge does not limited the number of voucher-requests it creates until at least one has finished.
An alternative may be that the onboarding state may expire after a certain time, if no further interaction has happened.

In addition, the pledge may assume that repeated triggering for PVR are the result of a communication error with the Registrar-Agent.
In that case the pledge MAY simply resent the PVR previously sent.
Note that in case of resending, a contained nonce and also the contained agent-signed-data in the PVR would consequently be reused.



## Misuse of acquired PVR and PER by Registrar-Agent

A Registrar-Agent that uses previously requested PVR and PER for domain-A, may attempt to onboard the device into domain-B.  This can be detected by the domain registrar while PVR processing.
The domain registrar needs to verify that the "proximity-registrar-cert" field in the PVR matches its own registrar LDevID certificate.
In addition, the domain registrar needs to verify the association of the pledge to its domain based on the product-serial-number contained in the PVR and in the IDevID certificate of the pledge. (This is just part of the supply chain integration).
Moreover, the domain registrar verifies if the Registrar-Agent is authorized to interact with the pledge for voucher-requests and enroll-requests, based on the EE (RegAgt) certificate data contained in the PVR.

Misbinding of a pledge by a faked domain registrar is countered as described in BRSKI security considerations {{Section 11.4 of !RFC8995}}.



## Misuse of Registrar-Agent Credentials {#sec_cons_reg-agt}

Concerns of misusage of a Registrar-Agent with a valid EE (RegAgt) certificate may be addressed by utilizing short-lived certificates (e.g., valid for a day) to authenticate the Registrar-Agent against the domain registrar.
The EE (RegAgt) certificate may have been acquired by a prior BRSKI run for the Registrar-Agent, if an IDevID is available on Registrar-Agent.
Alternatively, the EE (RegAgt) certificate may be acquired by a service technician from the domain PKI system in an authenticated way.

In addition it is required that the EE (RegAgt) certificate is valid for the complete bootstrapping phase.
This avoids that a Registrar-Agent could be misused to create arbitrary "agent-signed-data" objects to perform an authorized bootstrapping of a rogue pledge at a later point in time.
In this misuse "agent-signed-data" could be dated after the validity time of the EE (RegAgt) certificate, due to missing trusted timestamp in the Registrar-Agents signature.
To address this, the registrar SHOULD verify the certificate used to create the signature on "agent-signed-data".
Furthermore the registrar also verifies the EE (RegAgt) certificate used in the TLS handshake with the Registrar-Agent. If both certificates are verified successfully, the Registrar-Agent's signature can be considered as valid.



## Misuse of DNS-SD with mDNS to obtain list of pledges {#sec_cons_mDNS}

To discover a specific pledge a Registrar-Agent may request the service name in combination with the product-serial-number of a specific pledge.
The pledge reacts on this if its product-serial-number is part of the request message.

If the Registrar-Agent performs DNS-based Service Discovery without a specific product-serial-number, all  pledges in the domain react if the functionality is supported.
This functionality enumerates and reveals the information of devices available in the domain.
The information about this is provided here as a feature to support the commissioning of devices.
A manufacturer may decide to support this feature only for devices not possessing a LDevID or to not support this feature at all, to avoid an enumeration in an operative domain.



## YANG Module Security Considerations

The enhanced voucher-request described in {{!I-D.ietf-anima-rfc8366bis}} is based on {{!RFC8995}}, but uses a different encoding based on {{!I-D.ietf-anima-jws-voucher}}.
The security considerations as described in {{Section 11.7 of !RFC8995}} (Security Considerations) apply.

The YANG module specified in {{I-D.ietf-anima-rfc8366bis}} defines the schema for data that is subsequently encapsulated by a JOSE signed-data Content-type as described in {{I-D.ietf-anima-jws-voucher}}.
As such, all of the YANG-modeled data is protected against modification.

The use of YANG to define data structures via the {{?RFC8971}} "structure" statement, is relatively
new and distinct from the traditional use of YANG to define an API accessed by network management protocols such as NETCONF {{?RFC6241}} and RESTCONF {{?RFC8040}}.
For this reason, these guidelines do not follow the template described by {{Section 3.7 of ?RFC8407}} (Security Considerations).




# Acknowledgments

We would like to thank the various reviewers, in particular Brian E. Carpenter, Charlie Kaufman (Early SECDIR review), Martin Björklund (Early YANGDOCTORS review), Marco Tiloca (Early IOTDIR review), Oskar Camenzind, Hendrik Brockhaus, and Ingo Wenda for their input and discussion on use cases and call flows.
Further review input was provided by Jesser Bouzid, Dominik Tacke, and Christian Spindler.
Special thanks to Esko Dijk for the in deep review and the improving proposals.
Support in PoC implementations and comments resulting from the implementation was provided by Hong Rui Li and He Peng Jia.




--- back




# Examples {#examples}

These examples are folded according to {{RFC8792}} Single Backslash rule.



## Example Pledge Voucher-Request (PVR) - from Pledge to Registrar-Agent

The following is an example request sent from a Pledge to the Registrar-Agent, in "General JWS JSON Serialization".
The message size of this PVR is: 4649 bytes

~~~~
=============== NOTE: '\' line wrapping per RFC 8792 ================

{
  "payload":
    "eyJpZXRmLXZvdWNoZXItcmVxdWVzdC1wcm06dm91Y2hlciI6eyJhc3NlcnRpb24\
iOiJhZ2VudC1wcm94aW1pdHkiLCJzZXJpYWwtbnVtYmVyIjoiMDEyMzQ1Njc4OSIsIm5\
vbmNlIjoiTDNJSjZocHRIQ0lRb054YWFiOUhXQT09IiwiY3JlYXRlZC1vbiI6IjIwMjI\
tMDQtMjZUMDU6MTY6MTcuNzA5WiIsImFnZW50LXByb3ZpZGVkLXByb3hpbWl0eS1yZWd\
pc3RyYXItY2VydCI6Ik1JSUI0akNDQVlpZ0F3SUJBZ0lHQVhZNzJiYlpNQW9HQ0NxR1N\
NNDlCQU1DTURVeEV6QVJCZ05WQkFvTUNrMTVRblZ6YVc1bGMzTXhEVEFMQmdOVkJBY01\
CRk5wZEdVeER6QU5CZ05WQkFNTUJsUmxjM1JEUVRBZUZ3MHlNREV5TURjd05qRTRNVEp\
hRncwek1ERXlNRGN3TmpFNE1USmFNRDR4RXpBUkJnTlZCQW9NQ2sxNVFuVnphVzVsYzN\
NeERUQUxCZ05WQkFjTUJGTnBkR1V4R0RBV0JnTlZCQU1NRDBSdmJXRnBibEpsWjJsemR\
ISmhjakJaTUJNR0J5cUdTTTQ5QWdFR0NDcUdTTTQ5QXdFSEEwSUFCQmsxNksvaTc5b1J\
rSzVZYmVQZzhVU1I4L3VzMWRQVWlaSE10b2tTZHFLVzVmbldzQmQrcVJMN1dSZmZlV2t\
5Z2Vib0pmSWxsdXJjaTI1d25oaU9WQ0dqZXpCNU1CMEdBMVVkSlFRV01CUUdDQ3NHQVF\
VRkJ3TUJCZ2dyQmdFRkJRY0RIREFPQmdOVkhROEJBZjhFQkFNQ0I0QXdTQVlEVlIwUkJ\
FRXdQNElkY21WbmFYTjBjbUZ5TFhSbGMzUXVjMmxsYldWdWN5MWlkQzV1WlhTQ0huSmx\
aMmx6ZEhKaGNpMTBaWE4wTmk1emFXVnRaVzV6TFdKMExtNWxkREFLQmdncWhrak9QUVF\
EQWdOSUFEQkZBaUJ4bGRCaFpxMEV2NUpMMlByV0N0eVM2aERZVzF5Q08vUmF1YnBDN01\
hSURnSWhBTFNKYmdMbmdoYmJBZzBkY1dGVVZvL2dHTjAvand6SlowU2wyaDR4SVhrMSI\
sImFnZW50LXNpZ25lZC1kYXRhIjoiZXlKd1lYbHNiMkZrSWpvaVpYbEtjRnBZVW0xTVd\
GcDJaRmRPYjFwWVNYUmpiVlo0WkZkV2VtUkRNWGRqYlRBMldWZGtiR0p1VVhSak1teHV\
ZbTFXYTB4WFVtaGtSMFZwVDI1emFWa3pTbXhaV0ZKc1drTXhkbUpwU1RaSmFrbDNUV3B\
KZEUxRVVYUk5hbHBWVFVSVk5rMUVZelpPUkVWMVRrUlJORmRwU1hOSmJrNXNZMjFzYUd\
KRE1YVmtWekZwV2xoSmFVOXBTWGROVkVsNlRrUlZNazU2WnpWSmJqRTVJaXdpYzJsbmJ\
tRjBkWEpsY3lJNlczc2ljSEp2ZEdWamRHVmtJam9pWlhsS2NtRlhVV2xQYVVwWlkwaHd\
jMVJWZERSaVNFSkNUbXBvYWxaVVZrZFZWVEZaVmxoYWRWTldVVEpWV0dNNVNXbDNhVmx\
YZUc1SmFtOXBVbFpOZVU1VVdXbG1VU0lzSW5OcFoyNWhkSFZ5WlNJNklrY3pWM2hHU0d\
WMFdGQTRiR3hTVmkwNWRXSnlURmxxU25aUllUWmZlUzFRYWxGWk5FNWhkMW81Y0ZKaGI\
yeE9TbTlFTm1SbFpXdHVTVjlGV0daemVWWlRZbmM0VTBONlRWcE1iakJoUVhWb2FVZFp\
UakJSSW4xZGZRPT0iLCJhZ2VudC1zaWduLWNlcnQiOlsiTUlJQjFEQ0NBWHFnQXdJQkF\
nSUVZbWQ0T1RBS0JnZ3Foa2pPUFFRREFqQStNUk13RVFZRFZRUUtEQXBOZVVKMWMybHV\
aWE56TVEwd0N3WURWUVFIREFSVGFYUmxNUmd3RmdZRFZRUUREQTlVWlhOMFVIVnphRTF\
2WkdWc1EwRXdIaGNOTWpJd05ESTJNRFEwTWpNeldoY05Nekl3TkRJMk1EUTBNak16V2p\
BOU1STXdFUVlEVlFRS0RBcE5lVUoxYzJsdVpYTnpNUTB3Q3dZRFZRUUhEQVJUYVhSbE1\
SY3dGUVlEVlFRRERBNVNaV2RwYzNSeVlYSkJaMlZ1ZERCWk1CTUdCeXFHU000OUFnRUd\
DQ3FHU000OUF3RUhBMElBQkd4bHJOZmozaVJiNy9CUW9kVys1WWlvT3poK2pJdHlxdVJ\
JTy9XejdZb1czaXdEYzNGeGV3TFZmekNyNU52RDEzWmFGYjdmcmFuK3Q5b3RZNVdMaEo\
2alp6QmxNQTRHQTFVZER3RUIvd1FFQXdJSGdEQWZCZ05WSFNNRUdEQVdnQlJ2b1QxdWR\
lMmY2TEVRaFU3SEhqK3ZKL2Q3SXpBZEJnTlZIUTRFRmdRVVhwemxNS3hscEE2OGNVNUZ\
RTVhVdm5JVDZRd3dFd1lEVlIwbEJBd3dDZ1lJS3dZQkJRVUhBd0l3Q2dZSUtvWkl6ajB\
FQXdJRFNBQXdSUUlnYzJ5NnhvT3RvUUJsSnNnbE9MMVZ4SEdvc1R5cEVxUmZ6MFF2NFp\
FUHY0d0NJUUNWeWIyRjl6VjNuOTUrb2xnZkZKZ1pUV0V6NGRTYUYzaHpSUWIzWnVCMjl\
RPT0iLCJNSUlCekRDQ0FYR2dBd0lCQWdJRVhYakhwREFLQmdncWhrak9QUVFEQWpBMU1\
STXdFUVlEVlFRS0RBcE5lVUoxYzJsdVpYTnpNUTB3Q3dZRFZRUUhEQVJUYVhSbE1ROHd\
EUVlEVlFRRERBWlVaWE4wUTBFd0hoY05NVGt3T1RFeE1UQXdPRE0yV2hjTk1qa3dPVEV\
4TVRBd09ETTJXakErTVJNd0VRWURWUVFLREFwTmVVSjFjMmx1WlhOek1RMHdDd1lEVlF\
RSERBUlRhWFJsTVJnd0ZnWURWUVFEREE5VVpYTjBVSFZ6YUUxdlpHVnNRMEV3V1RBVEJ\
nY3Foa2pPUFFJQkJnZ3Foa2pPUFFNQkJ3TkNBQVRsRzBmd1QzM29leloxdmtIUWJldGV\
ibWorQm9WK1pGc2pjZlF3MlRPa0pQaE9rT2ZBYnU5YlMxcVppOHlhRVY4b2VyS2wvNlp\
YYmZ4T21CanJScmNYbzJZd1pEQVNCZ05WSFJNQkFmOEVDREFHQVFIL0FnRUFNQTRHQTF\
VZER3RUIvd1FFQXdJQ0JEQWZCZ05WSFNNRUdEQVdnQlRvWklNelFkc0Qvai8rZ1gvN2N\
CSnVjSC9YbWpBZEJnTlZIUTRFRmdRVWI2RTliblh0bitpeEVJVk94eDQvcnlmM2V5TXd\
DZ1lJS29aSXpqMEVBd0lEU1FBd1JnSWhBUG5CMHcxTkN1cmhNeEp3d2ZqejdnRGlpeGt\
VWUxQU1o5ZU45a29oTlFVakFpRUF3NFk3bHR4V2lQd0t0MUo5bmp5ZkRObDVNdUVEQml\
teFIzQ1hvWktHUXJVPSJdfX0",
    "signatures":[{
      "protected":"eyJ4NWMiOlsiTUlJQitUQ0NBYUNnQXdJQkFnSUdBWG5WanNVN\
U1Bb0dDQ3FHU000OUJBTUNNRDB4Q3pBSkJnTlZCQVlUQWtGUk1SVXdFd1lEVlFRS0RBe\
EthVzVuU21sdVowTnZjbkF4RnpBVkJnTlZCQU1NRGtwcGJtZEthVzVuVkdWemRFTkJNQ\
0FYRFRJeE1EWXdOREExTkRZeE5Gb1lEems1T1RreE1qTXhNak0xT1RVNVdqQlNNUXN3Q\
1FZRFZRUUdFd0pCVVRFVk1CTUdBMVVFQ2d3TVNtbHVaMHBwYm1kRGIzSndNUk13RVFZR\
FZRUUZFd293TVRJek5EVTJOemc1TVJjd0ZRWURWUVFEREE1S2FXNW5TbWx1WjBSbGRtb\
GpaVEJaTUJNR0J5cUdTTTQ5QWdFR0NDcUdTTTQ5QXdFSEEwSUFCQzc5bGlhUmNCalpjR\
UVYdzdyVWVhdnRHSkF1SDRwazRJNDJ2YUJNc1UxMWlMRENDTGtWaHRVVjIxbXZhS0N2T\
XgyWStTTWdROGZmd0wyM3ozVElWQldqZFRCek1Dc0dDQ3NHQVFVRkJ3RWdCQjhXSFcxa\
GMyRXRkR1Z6ZEM1emFXVnRaVzV6TFdKMExtNWxkRG81TkRRek1COEdBMVVkSXdRWU1CY\
UFGRlFMak56UFwvU1wva291alF3amc1RTVmdndjWWJNQk1HQTFVZEpRUU1NQW9HQ0NzR\
0FRVUZCd01DTUE0R0ExVWREd0VCXC93UUVBd0lIZ0RBS0JnZ3Foa2pPUFFRREFnTkhBR\
EJFQWlCdTN3UkJMc0pNUDVzTTA3MEgrVUZyeU5VNmdLekxPUmNGeVJST2xxcUhpZ0lnW\
ENtSkxUekVsdkQycG9LNmR4NmwxXC91eW1UbmJRRERmSmxhdHVYMlJvT0U9Il0sImFsZ\
yI6IkVTMjU2In0",
      "signature":"Y_ohapnmvbwjLuUicOB7NAmbGM7igBfpUlkKUuSNdG-eDI4BQ\
yuXZ2aw93zZId45R7XxAK-12YKIx6qLjiPjMw"
  }]
}
~~~~
{: #ExamplePledgeVoucherRequestfigure title='Example Pledge-Voucher-Request - PVR' artwork-align="left"}


## Example Parboiled Registrar Voucher-Request (RVR) - from Registrar to MASA

The term parboiled refers to food which is partially cooked.  In {{!RFC8995}}, the term refers to a pledge-voucher-request (PVR) which has
been received by the Registrar, and then has been processed by the Registrar ("cooked"), and is now being forwarded to the MASA.

The following is an example registrar-voucher-request (RVR) sent from the Registrar to the MASA, in "General JWS JSON Serialization".
Note that the previous PVR can be seen in the payload as "prior-signed-voucher-request".
The message size of this RVR is: 13257 bytes

~~~~
=============== NOTE: '\' line wrapping per RFC 8792 ================

{
  "payload":
    "eyJpZXRmLXZvdWNoZXItcmVxdWVzdC1wcm06dm91Y2hlciI6eyJhc3NlcnRpb24\
iOiJhZ2VudC1wcm94aW1pdHkiLCJzZXJpYWwtbnVtYmVyIjoiY2FmZmUtOTg3NDUiLCJ\
ub25jZSI6ImM1VEVPb29NTE5hNEN4L1UrVExoQ3c9PSIsInByaW9yLXNpZ25lZC12b3V\
jaGVyLXJlcXVlc3QiOiJleUp3WVhsc2IyRmtJam9pWlhsS2NGcFlVbTFNV0ZwMlpGZE9\
iMXBZU1hSamJWWjRaRmRXZW1SRE1YZGpiVEEyWkcwNU1Wa3lhR3hqYVVrMlpYbEthR01\
6VG14amJsSndZakkwYVU5cFNtaGFNbFoxWkVNeGQyTnRPVFJoVnpGd1pFaHJhVXhEU25\
wYVdFcHdXVmQzZEdKdVZuUlpiVlo1U1dwdmFWa3lSbTFhYlZWMFQxUm5NMDVFVldsTVE\
wcDFZakkxYWxwVFNUWkpiVTB4VmtWV1VHSXlPVTVVUlRWb1RrVk9ORXd4VlhKV1JYaHZ\
VVE5qT1ZCVFNYTkpiVTU1V2xkR01GcFhVWFJpTWpScFQybEplVTFFU1hsTVZFRjVURlJ\
KZVZaRVFUTlBhazE2VDJwQk5FeHFSVFZPYkc5cFRFTkthRm95Vm5Wa1F6RjNZMjA1TW1\
GWFVteGFRekYzWTIwNU5HRlhNWEJrU0d0MFkyMVdibUZZVGpCamJVWjVURmRPYkdOdVV\
XbFBhVXBPVTFWc1JGSkdVa1JSTUVacFZESmtRbVF3YkVOUlYyUktVakJHV1ZkWVRuZFR\
WMVoyVkZWR2RsSXdUa1JqVldSVVZGUlJOVkZyUms1Uk1ERkhaRE5vUkdWclJrdFJiV1J\
QVm10S1FsZFdVa0poTUZwVFZGWktTbVF3VmtKWFZWSlhWVlpHVEZKRlJuTlViVlpXVkc\
1YWFWZEZTbTlaYlRWeVpVVmFWVkZXVWtOYU1EVlhVV3RHZWxSVlVrWk5WRlpXVFRGYWN\
GbDZTbk5oTWtaWVVtNXNiRlpGVmxGVVZVVjNVakJGZUZaVlZrTmtNMlJJVmtab2MxWkh\
SbGxWYlhoT1ZXdFdNMUpJWkZwU1JscFNWVlZTUlZGWGFFOWFWbHBQWTBkU1NGWnJVbEp\
XUlVac1VtNWpkMlZWTVVWU1dHeE9Va1pHTTFSdWNFcE5WVFZGWVVkR1IyUjZRalpVVlZ\
KR1pWVXhSVlZZWkU5bGEydDRWR3RTYjFsVk1VaFRXR2hFWld0R1MxRnRaRTlXYTBwQ1Y\
xWlNRbUV3V2xOVVZrcEtaREJXUWxkVlVsZFZWa1pNVWtWR2MxUnRWbFpVYmxwcFYwVkt\
iMWx0TlhKbFJWcEZVVlpPUTFvd05WZFJhMFo2VkZWTmQwMVVWbFpOTVZwd1dYcEtjMkV\
4YkZsVGFsWk9WVlJvTTFKR1JscFNSbHBTVlZWb1JWRldjRTlhVmxwUFkwZFNTRlpZYUV\
oU1JVWllVVzFrVDFaclNrSlVWVEZGVFVSRk1WWlVTbk5OUm5CWFUyMTRZVTF0ZURaYVJ\
XaExZVWRPY1ZGc2NFNVJhekZJVVc1c2VGSXhUazVPUkd4Q1dqQldTRkV3VG5oU01VNU9\
Ua1JzUW1Rd1ZrbFJWRUpLVVZWS1NsVklhRmhrVlRGSlVucG9TMVJIV2taVlJVVTFUMWh\
hZDAxdVZrbFNWVFZxVlROc2JWa3pWVE5VUjJSYVRraEJlRTVGUmtOT01GSk9XbFJGTkU\
xSVRrWmxSV1J4VkZOME0yTnJOVFJPTURVMVdWaE9lRXQ2Um5CTlJXUlVVVEZKTlU1VVV\
UTk5SRVp0VWpKV1dGUldSbWxqVjNCWVpXdEtZVlJWU1hkU01FVjRWbGRTUzFWV1JsaFV\
WVXBTVWpCT1JHTXdaRUpWVmxaSFVXNWtUbEZyU201YU0wcERXakJXUjFGc1JtcFNSV2h\
GVVZVNVExb3dOVmRUUmtVMFVXdEdiVTlGVmtOUlZURkVVV3BTUW1Rd2RFSlhWVkpYVld\
wQ1UxRnJUa1prTUdjd1UxZFNhVmRIZURaWlZtaFRZa2RPZEZadE5XaFhSVFIzV1RJeFI\
yVlZlSFJOVkZaYVRXcHNNRmt3WkVka1YxWlVUbGR3YVUxcVFqTlJNbVJhVTFWMGRsZHJ\
iRFpoYWtKR1VWaGtTbEpHVGtKUldHUlRWVlZzYjFGV1FqVlBXRnBOVTFkR01WVnJWbEp\
qYlhnMVlteGtNRlI2VmxOaVZYQXdZVmhTVW1GNlpFOVhSRkpMVlVoV2RWVklRazlVYXp\
BeFZWUnNRbUZWUm5sa1JVNTBWRVprWVZSVmJFMVdiRTEyVFZWc1JWbFhjR2xqTVU1Sll\
tNXdkbUpYYjNkV1F6a3paVmhKY21Nd2RFdGpRM1IxVFhwU1VsQlVNR2xNUTBwb1dqSld\
kV1JETVhwaFYyUjFXbGRSZEZwSFJqQlpVMGsyU1cxV05WTnVaRnBYUjNoNldXcEtSMkV\
3YkhGaU1teGhWMGQ0VEZrd1duZFhWbFowVFZVeFdGSnVRWGxYYTFwclZESkplR05HYkZ\
SWFJrcHhXV3hhWVU1R2NFZGFSbVJzWWxaS1JWUldhR3RoYlVwVlVWUktXRlp0VW5KWmE\
yUkxaRlpXV1ZWdGNFNWlXR2d4VjFjd2VGWXlSWGRsUm1oV1lsZG9jbFZxUWxkalJsRjV\
UbGh3YUZadGREWlZNakUwVjJ4a1IxTnVUbGhoTURFMFdrY3hTMk5HVGxWWGEzQm9ZVEo\
zZWxaR1pIZFNiVkpHVFZaV1UxZEdTazlXYTFwM1ZteFNWbFZyY0U5aGVrVXlWVlpTWVZ\
Sc1NrWlNha1pWVmxaS1ExcEVSbXRqUms1WlZHdHdhV0Y2Vm5wWFZFbDRZekpHU0ZOclV\
rNVhSbHB5Vm01d1IyTkdaSE5oUlhCb1ZsUnNkMVV5TVhkWGJGbDRZMGhTV0dKRk1UTlV\
iRlUxVWxac05sRnJPVlpOUnpneFYyMTRSbUZWZUVSVGJuQm9WakpTTVZkV2FGTk5WMDU\
wVm01d1NtRnVRbWxhV0d4TFpESk9kRTlVUW1GV01EUjNWMnhrVW1GVk9YQlRiWGhzVmx\
oQ2RsZFhkR3RoYlVaV1QxaENWR0V4Y0ZkYVYzUnlaVVpTZEdKRmNHcE5SM2d3V2tWb1E\
xbFdSWGRoZWtwVVZqTm9kbFZ0ZEhwbFZsWlpVMnhTYVdKclNrcFdhMVpUVVcxV2MxSnV\
VbFJpVlZwVlZXdGFjbVZzVFhwalJ6bFhUVlpHTmxkWWNFTmhiRWw1V1ROa1ZVMUdSak5\
aVm1SaFZXdHNjR1F5YkdwTmJYaDFXVzB4UjAxSFVsbFRiWGhLWVcwNWNGZEVRazlsYXp\
sV1QxWm9VMkpyY0dGWmJHTTFaV3hOZDA5WVZsSlhSMUpUVjFSR1YyRXhiM2RqUlZaWVV\
qRndUVmxzVWt0WFZUbFhVMnhXVjFJd05URlVSazE0WXpGUmQwNVdRbWxTZWxaMlZqSjB\
SazFGT1VaWGJYUlhZa2hCZDFReFVrOWhNbEY0Vld0d1YxWlVWbGRYUkVwaFYyczVWMWR\
1UWxkaVJHdzFWWHBPVDFaV1JuSmhSa3BRVmpOQ2Vsa3haSHBsVjBsNldUSnNiVlpxUlR\
WSmFYZHBXVmRrYkdKdVVYUmpNbXh1WW1reGFscFlTakJKYW5CaVNXc3hTbE5WVGt0U1J\
VNUVVVmRPZUZvd1JqTlRWVXBDV2pCc1JsZEhlSEZSTURGRlVWVjBRMW95WkhoaFIzUnh\
WREZDVWxWVlVrSmhhMHB6VkZaR2VtUXdUbEpYVlZKWFZWWkdTRkpZWkV0UmJGWlZVbFp\
PVGxGclJraFJWRVpXVWxWT2JtUXdjRlZYUjNoRldXcEplR1F4YkZoT1ZGWk9WV3hXTTF\
KWVpGcFNSbHBTVlZWNFJWRllhRTlhVmxwUFRWWnNkVlJ1UW1GU01uaHZXVEkxY21WRlV\
qWlJWVFZEV2pBMVYxRnJSbXBVVlVweVRWUldWazF0ZDNkWGJGSkdXVlV4UTFvd1pFSk5\
WbFpHVVZoa00xVnNVbGxpUmxKb1YwWktjMVpWYUZkbGJVWkdUVmhhWVZJeFducFZWRUp\
HWkRCb2Ixa3dOVTVoYTBZelZGZHdTazVGTVVWWk0zQk9aV3RGZDFZeWFHcFVhekUyVVZ\
oa1RtRnJhekJVVlZKcVpXc3hObEZVUWxoaGEwcDBWRlpHZW1Rd1RsSlhWVkpYVlZaR1N\
GSllaRXRSYkZaVlVsWk9UbEZyUmtoUlZFWldVbFZPYm1Rd2NGVlhSM2hGV1dwSmVHUXh\
iRmhPVkZaT1ZXeFdNMUpZWkZwU1JscFNWVlY0UlZGWWFFOWFWbHBQVFZac2RWUnVRbUZ\
TTW5odldUSTFjbVZGVWpaUlZUVkRXakExVjFGclJtcFVWVXB5VFZSV1ZrMXRkM2RYYkZ\
KR1dXc3hRMkV3WkVKTlZsWkdVVmhrTTFVeFVsbGlSbEpvVjBaS2MxWlZhRmRsYlVaR1R\
WaGFZVkl4V25wVlZtaERaREF4UjJFelpFWmtNV3hKVXpJNVlWTlljSEZOUlU1Q1ZWWnN\
TbE15T1dGVFdIQnhUVVZTUWxWWFRrVlZWMlJDVWxSWmQwMVZPSEppTW5CRVlUTktSVlZ\
1WXpOYU1Hd3lWMnRWTUdGVVRUQmFSMHB2VVROR2NGSjZaSEZpTWprelYyNUJNR1ZJV2p\
aU2JsSk5XbnBhVlZaNlFtOVViVkpKWkd4Q1JWVXhVbnBrVm1oVVpWWmpOV1JJU1hwUld\
HUkVZa2RhUkdJd1VsZFVia1pRWkhwc05WUldaekpVYlRWT1VqRldNMUpIWkZwU1JscFR\
UVVpDUWxWVlozWlJhMFpTVWtWR2JscFZSazVSYW1oSVVWUkdWbHBGYkROVlZteE9VVzF\
HUWxKcmJ6TlRTRkpVWkROQ1RWUklWbEJYYW1ScVlUQkdjMVZWYUZaTk1tUkNWRmRqZGx\
Ock1VTk5SV1JDVFZaV2ExSkhaRkpXTUVwRFZXMU9WVTVVVFRCaWF6RmFaR3hTYWxKdVV\
uSmFia295VGpOb1ZrNHdVbkJpVldoeFpXdEdWVkZ0WkU5V2EyaFVWbFZXUlZKRlJreFJ\
iV1J1WTJ0S2JsSlZXa05WVjA1RlVWZHdRbE13U201YU0wWnZZVEp3VUZWR1JsSlNSVVp\
1Vkd0c1FsSkZTa2RSVjJ4R1VWaENTMDR6YUhkVWJGWXlWVlZ3U0UxRk5XOVVSMGwyV2x\
oU2FVMXFRazFTUmxWNFRtMTRkMVV3YUZCT01rWnNZbnBDVjFkWVozZGxTR1JFVTFWRmN\
sUjZWWFpYVkZwRllVTjBhVkZxU1RSTmFsSXhZVmRHVUZWWFJsWlNSRnB1VVZVMWIxZFV\
iRmRTYlZGeVlXNUtlVmt3VmpKVGJsRnBURU5LVGxOVmJFUlNNVkpFVVRCR2FVc3laRUp\
rTUd4RFVWZGtTbEpXYUhOaGEwVjJaV3RHVEZGdFpHNWpWMmh5WVdzNVVWVldSa1ZSVjN\
CRFdUQXhVbU16WkVSVlZteEZWbXhHVWxJd1ZqTlRhMHBXVmtWV1ZGUlZTa0pTTUVWNFZ\
sVldSRm96WkV0V1JtaHpVa2RKZVUxWVpGcFdlbFV4VkZaS1ZtUXdWak5YVlZKWFZWWkd\
UVkpGUmpSVWJWWlhWR3BHV21Kck5YZFhhMlJ6WVVkT2RXRXphRVZsYTBaUFVXMWtUMVp\
yU2tKWk1ERkRZWHBGTVZaVVNuTk5SbkJWVWxaS1RsRlVhRWhSVkVaV1VsVkdNMlF3YkZ\
WWFIzaFZXVlpvVTJKR1JYZFNXR1JKWVVkT1QxUlhjRUprTURGeFUxUlNUbEpIVGpWVWJ\
uQldUbFprYjFrd05VNWxhMFl6VkZkd1NrNUZNVVZaTTJ4UFpXeFZNVll5Y0VOaVJURlN\
Zek5rUkZWV2JFVldiRVpTVWpCV00xTnJTbFpXUlZaVVZGVktRbEl3UlhoV1ZWWkVXak5\
rUzFaR2FITlNSMGw1VFZoa1dsWjZWVEZVVmtwV1pEQldNMWRWVWxkVlZrWk5Va1ZHTkZ\
SdFZsZFVha1phWW1zMWQxZHJaSE5oUjA1MVlUTm9SV1ZyUms5UmJXUlBWbXRLUWxrd01\
VTmhla1V4VmxSS2MwMUdjRlZTVjBaT1VXMWtTRkZVUmxaU1ZVWXpaREZLVlZkSGVGVlp\
WbWhUWWtaV1NWWnVjR2hTVkVZeVYydGtWMk14UlhkU1dHUllWa1ZHVlZGdFpHcGpWMmh\
5WVdzNVVWVlZiRU5SYldSdVkxZG9jbUZyT1ZGVlZURkRVVzVrVDFFd1JrSlZhM0JEVm0\
wNWVscEZkRE5YVlRVMFlWWkNORk5JV25CU2JrWk1aV3RTYzA5WFdqQlVTRlpPV1ZjeGQ\
xSnNSbXBYU0dONFRXcGthRlJ0T1ZOWmJrNUpUREJhVG1OdE1UWlJNRVpKVFhwak0wMTZ\
UbXBOYlRscFZVZE9jMlJzVG5sWFZVb3lUVVZPTUZZeFJqQlpWRnBvU3pKT2RrMXNiRE5\
YYTFKQ1ZUQktibFJzV2tsVmF6RkRVVmRaTkZKVlRrVlJWV1JDVlZWbmRsRlhaRVpSVlR\
GQ1RrVmtRazFXVm10U1NHUkdVV2s1TTFWVlZrSmtNR3hFVVd0U1FscHJTbTVVYkZwSlZ\
UQXhSbEl3VWtKV01tUkRWVmh3TkdWdVpIZFZia0pOWlZNNWVWUldWbHBsYlVadlRXNU5\
lRTB5VmxaUFYyUkhaV3RHYTFGdFpFOVdhMmhTVGtWV1Ixb3hSbFppYms1c1RWVjRSR0V\
6VGpGT1JGWjFaRWhzVWxFeFdrSmFSbEpzVVZWR05WSkVhSEprTUU1dVYxVnNUR0l4Y0V\
wbGJXOTNVbFZHTTFOVlVsUlJWVVl6Vld4R1NtRkZSa3BqTVd4eldsWndUR015Y0VkVWE\
wNTZVMnQwYkZWSGVFaFVWVVpOV2xoQ1YxcFViRVpVUkdSUFlqTlJNVTFVVmpObFJ6Rlh\
aRlZ3UTFGWGJFSlpNRlpPVmxaV2IxSldUbnBVUm1SUlRsaG9WRlZXVlhkWFNFWTJWbTV\
GTkZkWVduQlNha1pwVm0wNU5sSXpjRFJPV0hCUFdqSk9lbVI2TURsSmJERTVabEVpTEN\
KemFXZHVZWFIxY21WeklqcGJleUp3Y205MFpXTjBaV1FpT2lKbGVVbzBUbGROYVU5c2M\
ybFVWV3hLVVRCb2NWRXdUa0paTVU1dVVWaGtTbEZyUm01VFZXUkNWMGRvTUUxVVVucGl\
NREZDWWpCa1JGRXpSa2hWTURBd1QxVktRbFJWVGs1U1ZtdzBVVE53UWxOclNtNVViRnB\
EVVZac1ZWRlhkRTlUVlRGVFZGaGtSbFZXYkVWV2JFWlNVekJTUW1OR1VtaFdNVm93VjJ\
4ak1XVnJiRVpTYTJoT1ZWUm9NMUpHUmxwU1JscFNWVlY0UlZGV2NFUldhMDVEVWtaV1I\
xUllhRVpXUlVaUlVXMWtUMVpyU2tKVVZURkVVbXh3YzFsdE1WTmtiVTV5Vkd0S1RsRXd\
SbGxTUmxKS1pVVXhSVlJZYkU5aGEwVXhWRmR3U21WVk5WZGlNV3hGWlcxek1WUXhVbkp\
sUlRGeFZGaG9UbUZyTUhoVU1WSldUbFprY1ZGdGFFNVZXRTR6VVRGR1dsSkdXbEpWVld\
SR1pEQndSVlV3VWtaV1JURkRVbFZrUWsxV1ZrWlJNbVF6VXpGVmVXSkhlR2xXTVZveFd\
UTnNRMUZzU2paU1ZrSk9VVlJDU0ZGVVJsWlNWVTR6WkRCa1VtSkdSbTVWVkVaRFZrVXh\
VMVZZWkVaYU1XeEZWbXhHVWxKclZqTmtSM0JhVmpGd2RGZHNUWGRPVlRsRldYcENUMVp\
GVmxoVVZVcFNVakJGZUZaVlZrSmtNMlJQVmxWYWIxSkZNVFZPVlZwUFpXeFdNRlJXVWt\
Ka01VWlZVV3h3VGxGck1VaFJibXg0VWpGT1RrNUViRUphTUZaSVVUQk9lRkl4VGs1T1J\
HeENaREJXU1ZGVVFrcFJWVXBQVFVSb2NWWXdlSHBOUjBadlV6Qm9XbGR1Vm05aVZ6Rmp\
UREpqTkdScVVsaFRNR2d5VVZoU2FGcHNSazFSVTNSS1pGVXhUMkZITVc1aFZ6RllUakJ\
HVG1OdVJtOWlWMHBWVFRCc2FGVkZUalZoUnpGaFUxWk9kMVI2V20xaVUzTXlVMWhhWTB\
3eldrcGphM1J2VlZaU01WWnRPVXhoYldSYVVWaGtiV0ZyUm5aUmJXUnVZMnRLYmxKVld\
rTlZWMDVEVTFWR1Vsa3dVa05qU0ZKYVYwVTFiMVJHYUZOaVIwMTZWVmhXYWsxdGVITlp\
iR1JYWkZkT05VNVhjR2xOYWtFeVZERlNVazFGTVRaUlZsSkRXakExVjFOR1RsWlNWVkp\
GVVZWMFExb3laSGxSYldSR1VtdEtVbGt3VWtKV1JVWlFVVzFrVDFacmFGSlBSVXBDV21\
wb1JsRnJSazVSTUVrd1VWaGtUVlZXYkVWV2JFbDNWV3RLUkZkWVpFdFRWV3h3V2tWa2I\
ySkZlSFZYYlhocFlsWktNbGt5YXpGaGJHeFlUa2hXYVdKVWEzZFVSekV3WkZkSmVsa3p\
WbXRTTW1oM1dUTnJNVTFzYkZobFJFWmhWa1ZHVEZGdFpHNWpWMmh5WVdzNVVWVldSa1Z\
SVjJSUFUxVkdSVkZyV2tKaFZVazFVVzB4ZUZRemNIRlZWR2hzV1ZkamVWTnVVblprVmx\
KdlVsWm9lVm93T1VOWFZsRjNVWHBvWVdSRGREVlBWemxKVWtad1JWbHNVbEpUVjJoQ1Z\
HMXpNbVJIT1ZOaU1sWkVXVmMxYUZSWGNFNVdSWGgwWWxaV2RXSlhTbkphYWtKNldsaGF\
jbEV3YnpSTmJXc3hWbGhHY1ZWcldsZFZVMHBrVEVOS2FHSkhZMmxQYVVwR1ZYcEpNVTV\
wU2praUxDSnphV2R1WVhSMWNtVWlPaUphWTFwa1dYbzBiMUl3UjJKc09UWnFNWGxZWm5\
kdlRYZGxVVGt6VGpCdFNVUmxjVFkyVTBacWRFdG9lR1pSWjNJMGRUWkpOVEJKWldNMmE\
xWTJhSEV3YVcxdlptTlBhVGs0VW1OSVpXUmpNVzFuZHpCWVp5SjlYWDA9IiwiY3JlYXR\
lZC1vbiI6IjIwMjItMDItMjJUMDc6MzM6MjUuMDIwWiIsImFnZW50LXNpZ24tY2VydCI\
6WyJNSUlDSkRDQ0FjcWdBd0lCQWdJRVhsakNNREFLQmdncWhrak9QUVFEQWpCbE1Rc3d\
DUVlEVlFRR0V3SkJVVEVTTUJBR0ExVUVDZ3dKVFhsRGIyMXdZVzU1TVJVd0V3WURWUVF\
MREF4TmVWTjFZbk5wWkdsaGNua3hEekFOQmdOVkJBY01CazE1VTJsMFpURWFNQmdHQTF\
VRUF3d1JUWGxUYVhSbFVIVnphRTF2WkdWc1EwRXdIaGNOTWpBd01qSTRNRGN6TXpBMFd\
oY05NekF3TWpJNE1EY3pNekEwV2pCbU1Rc3dDUVlEVlFRR0V3SkJVVEVTTUJBR0ExVUV\
DZ3dKVFhsRGIyMXdZVzU1TVJVd0V3WURWUVFMREF4TmVWTjFZbk5wWkdsaGNua3hEekF\
OQmdOVkJBY01CazE1VTJsMFpURWJNQmtHQTFVRUF3d1NUWGxUYVhSbFVIVnphRTF2Wkd\
Wc1FYQndNRmt3RXdZSEtvWkl6ajBDQVFZSUtvWkl6ajBEQVFjRFFnQUU2MDFPK29qQ2t\
yRFJ3N2dJdlpFNGkzNGRiaENxaUc3am9vd1pwNHh2ekZ0TGc2VFcwaE5kSHZQRFNUc3V\
YU3lXOXRyM0F3Q2xmQ29EVk5xT3c5eU1YNk5uTUdVd0RnWURWUjBQQVFIL0JBUURBZ2V\
BTUI4R0ExVWRJd1FZTUJhQUZKN0h0U3dwTEx1T1o3Y2tBbFFIVTNnQU1nL0pNQjBHQTF\
VZERnUVdCQlJjVDUzNG5NWXZUY0Z0a2Zydjd4VTdEaW1IanpBVEJnTlZIU1VFRERBS0J\
nZ3JCZ0VGQlFjREFqQUtCZ2dxaGtqT1BRUURBZ05JQURCRkFpRUFwSjd4cE5VdlFKRzB\
OaExiL2V0YjIwTERVMTZscFNITzdhZW8wVll4MHh3Q0lBK081L1k2RGgrYkIyODI0dWl\
hT1FhVUQ2Z0FOaFk5VkZkK2pycmNFdkp0IiwiTUlJQ0dUQ0NBYitnQXdJQkFnSUVYbGp\
BL3pBS0JnZ3Foa2pPUFFRREFqQmNNUXN3Q1FZRFZRUUdFd0pCVVRFU01CQUdBMVVFQ2d\
3SlRYbERiMjF3WVc1NU1SVXdFd1lEVlFRTERBeE5lVk4xWW5OcFpHbGhjbmt4RHpBTkJ\
nTlZCQWNNQmsxNVUybDBaVEVSTUE4R0ExVUVBd3dJVFhsVGFYUmxRMEV3SGhjTk1qQXd\
Nakk0TURjeU56VTVXaGNOTXpBd01qSTRNRGN5TnpVNVdqQmxNUXN3Q1FZRFZRUUdFd0p\
CVVRFU01CQUdBMVVFQ2d3SlRYbERiMjF3WVc1NU1SVXdFd1lEVlFRTERBeE5lVk4xWW5\
OcFpHbGhjbmt4RHpBTkJnTlZCQWNNQmsxNVUybDBaVEVhTUJnR0ExVUVBd3dSVFhsVGF\
YUmxVSFZ6YUUxdlpHVnNRMEV3V1RBVEJnY3Foa2pPUFFJQkJnZ3Foa2pPUFFNQkJ3TkN\
BQVJKQlZvc2RLd1lOeGlQeEh2aUZxS3pEbDlmdEx1TWFtcEZRY1h3MTI3YU5vUmJzSC9\
GTXJtekNBSDM3NzMzYzJvYlBjbHZTcllCdjBDdFdRdGE2YStjbzJZd1pEQVNCZ05WSFJ\
NQkFmOEVDREFHQVFIL0FnRUFNQTRHQTFVZER3RUIvd1FFQXdJQ0JEQWZCZ05WSFNNRUd\
EQVdnQlF6eHp3cFJwTHkvck1VWXphaDJzMTNlVTlnRnpBZEJnTlZIUTRFRmdRVW5zZTF\
MQ2tzdTQ1bnR5UUNWQWRUZUFBeUQ4a3dDZ1lJS29aSXpqMEVBd0lEU0FBd1JRSWhBSXN\
ZbGVaS3NqRk5Dc0pLZVBsR01BTGVwVmU5RUw3Tm90NTE1d3htVnVKQkFpQWNFTVVVaEV\
Tc0xXUDV4U1FVMFhxelZxOFl2aUYxYlZvekd6eDV6Tmdjc3c9PSJdfX0",
  "signatures":[{
    "protected":"eyJ4NWMiOlsiTUlJQjhEQ0NBWmFnQXdJQkFnSUdBWEJoTUtZSU1\
Bb0dDQ3FHU000OUJBTUNNRnd4Q3pBSkJnTlZCQVlUQWtGUk1SSXdFQVlEVlFRS0RBbE5\
lVU52YlhCaGJua3hGVEFUQmdOVkJBc01ERTE1VTNWaWMybGthV0Z5ZVRFUE1BMEdBMVV\
FQnd3R1RYbFRhWFJsTVJFd0R3WURWUVFEREFoTmVWTnBkR1ZEUVRBZUZ3MHlNREF5TWp\
Bd05qQXlNak5hRncwek1EQXlNakF3TmpBeU1qTmFNSGt4Q3pBSkJnTlZCQVlUQWtGUk1\
SSXdFQVlEVlFRS0RBbE5lVU52YlhCaGJua3hGVEFUQmdOVkJBc01ERTE1VTNWaWMybGt\
hV0Z5ZVRFUE1BMEdBMVVFQnd3R1RYbFRhWFJsTVM0d0xBWURWUVFERENWU1pXZHBjM1J\
5WVhJZ1ZtOTFZMmhsY2lCU1pYRjFaWE4wSUZOcFoyNXBibWNnUzJWNU1Ga3dFd1lIS29\
aSXpqMENBUVlJS29aSXpqMERBUWNEUWdBRUJUVFwvc1JmTDlsSnVGbVwvY24zU2pHcWp\
QXC9xdnBrNytoSTIwOE5oVkRvR2hcLzdLUCt2TXpYeVFRQStqUjZrNnJhTTI4ZlwvbHV\
FK1h1aHVwN1Vmem05Q3FNbk1DVXdFd1lEVlIwbEJBd3dDZ1lJS3dZQkJRVUhBeHd3RGd\
ZRFZSMFBBUUhcL0JBUURBZ2VBTUFvR0NDcUdTTTQ5QkFNQ0EwZ0FNRVVDSUhOK3VBbUp\
ldVhlc1wveWQxd1M0Mlo0RFhKNEptMWErZzNYa1pnZjhUaGxuQWlFQXBVdTZzZnljRWt\
veDdSWlhtZitLOHc0cDZzUldyamExUVJwWTAyNjQxSFk9IiwiTUlJQjhEQ0NBWmVnQXd\
JQkFnSUdBWEJoTUtZQk1Bb0dDQ3FHU000OUJBTUNNRnd4Q3pBSkJnTlZCQVlUQWtGUk1\
SSXdFQVlEVlFRS0RBbE5lVU52YlhCaGJua3hGVEFUQmdOVkJBc01ERTE1VTNWaWMybGt\
hV0Z5ZVRFUE1BMEdBMVVFQnd3R1RYbFRhWFJsTVJFd0R3WURWUVFEREFoTmVWTnBkR1Z\
EUVRBZUZ3MHlNREF5TWpBd05qQXlNak5hRncwek1EQXlNakF3TmpBeU1qTmFNRnd4Q3p\
BSkJnTlZCQVlUQWtGUk1SSXdFQVlEVlFRS0RBbE5lVU52YlhCaGJua3hGVEFUQmdOVkJ\
Bc01ERTE1VTNWaWMybGthV0Z5ZVRFUE1BMEdBMVVFQnd3R1RYbFRhWFJsTVJFd0R3WUR\
WUVFEREFoTmVWTnBkR1ZEUVRCWk1CTUdCeXFHU000OUFnRUdDQ3FHU000OUF3RUhBMEl\
BQkluQ3VoV1ZzZ2NONzFvWmVzMUZHXC9xZFZnTVBva01wZlMyNzFcL2V5SWNcL29EVmJ\
NRkhDYmV2SjVMTTgxOTVOYVpLTlNvSFAzS3dMeXVCaDh2MncwOVp1alJUQkRNQklHQTF\
VZEV3RUJcL3dRSU1BWUJBZjhDQVFFd0RnWURWUjBQQVFIXC9CQVFEQWdJRU1CMEdBMVV\
kRGdRV0JCUXp4endwUnBMeVwvck1VWXphaDJzMTNlVTlnRnpBS0JnZ3Foa2pPUFFRREF\
nTkhBREJFQWlCZGJIU212YW9qaDBpZWtaSUtOVzhRMGxTbGI1K0RLTlFcL05LY1I3dWx\
6dGdJZ2RwejZiUkYyREZtcGlKb3JCMkd5VmE4YVdkd2xIc0RvRVdZY0k0UEdKYmc9Il0\
sImFsZyI6IkVTMjU2In0",
    "signature":"67t3n8zyEek4IM2Ko3Y_UvE1hzp794QFNTqG-HzTrBQtE4_4-yS\
yyFd3kP6YCn35YYJ7yK35d3styo_yoiPfKA"
  }]
}
~~~~
{: #ExampleRegistrarVoucherRequestfigure title='Example Registrar-Voucher-Request - RVR' artwork-align="left"}


## Example Voucher - from MASA to Pledge, via Registrar and Registrar-Agent

The following is an example voucher-response from MASA to Pledge via Registrar and Registrar-Agent, in "General JWS JSON Serialization". The message size of this Voucher is: 1916 bytes

~~~~
=============== NOTE: '\' line wrapping per RFC 8792 ================

{
  "payload":"eyJpZXRmLXZvdWNoZXI6dm91Y2hlciI6eyJhc3NlcnRpb24iOiJhZ2V\
udC1wcm94aW1pdHkiLCJzZXJpYWwtbnVtYmVyIjoiMDEyMzQ1Njc4OSIsIm5vbmNlIjo\
iTDNJSjZocHRIQ0lRb054YWFiOUhXQT09IiwiY3JlYXRlZC1vbiI6IjIwMjItMDQtMjZ\
UMDU6MTY6MjguNzI2WiIsInBpbm5lZC1kb21haW4tY2VydCI6Ik1JSUJwRENDQVVtZ0F\
3SUJBZ0lHQVcwZUx1SCtNQW9HQ0NxR1NNNDlCQU1DTURVeEV6QVJCZ05WQkFvTUNrMTV\
RblZ6YVc1bGMzTXhEVEFMQmdOVkJBY01CRk5wZEdVeER6QU5CZ05WQkFNTUJsUmxjM1J\
EUVRBZUZ3MHhPVEE1TVRFd01qTTNNekphRncweU9UQTVNVEV3TWpNM016SmFNRFV4RXp\
BUkJnTlZCQW9NQ2sxNVFuVnphVzVsYzNNeERUQUxCZ05WQkFjTUJGTnBkR1V4RHpBTkJ\
nTlZCQU1NQmxSbGMzUkRRVEJaTUJNR0J5cUdTTTQ5QWdFR0NDcUdTTTQ5QXdFSEEwSUF\
CT2t2a1RIdThRbFQzRkhKMVVhSTcrV3NIT2IwVVMzU0FMdEc1d3VLUURqaWV4MDYvU2N\
ZNVBKaWJ2Z0hUQitGL1FUamdlbEhHeTFZS3B3Y05NY3NTeWFqUlRCRE1CSUdBMVVkRXd\
FQi93UUlNQVlCQWY4Q0FRRXdEZ1lEVlIwUEFRSC9CQVFEQWdJRU1CMEdBMVVkRGdRV0J\
CVG9aSU16UWRzRC9qLytnWC83Y0JKdWNIL1htakFLQmdncWhrak9QUVFEQWdOSkFEQkd\
BaUVBdHhRMytJTEdCUEl0U2g0YjlXWGhYTnVocVNQNkgrYi9MQy9mVllEalE2b0NJUUR\
HMnVSQ0hsVnEzeWhCNThUWE1VYnpIOCtPbGhXVXZPbFJEM1ZFcURkY1F3PT0ifX0",
  "signatures":[{
    "protected":"eyJ4NWMiOlsiTUlJQmt6Q0NBVGlnQXdJQkFnSUdBV0ZCakNrWU1\
Bb0dDQ3FHU000OUJBTUNNRDB4Q3pBSkJnTlZCQVlUQWtGUk1SVXdFd1lEVlFRS0RBeEt\
hVzVuU21sdVowTnZjbkF4RnpBVkJnTlZCQU1NRGtwcGJtZEthVzVuVkdWemRFTkJNQjR\
YRFRFNE1ERXlPVEV3TlRJME1Gb1hEVEk0TURFeU9URXdOVEkwTUZvd1R6RUxNQWtHQTF\
VRUJoTUNRVkV4RlRBVEJnTlZCQW9NREVwcGJtZEthVzVuUTI5eWNERXBNQ2NHQTFVRUF\
3d2dTbWx1WjBwcGJtZERiM0p3SUZadmRXTm9aWElnVTJsbmJtbHVaeUJMWlhrd1dUQVR\
CZ2NxaGtqT1BRSUJCZ2dxaGtqT1BRTUJCd05DQUFTQzZiZUxBbWVxMVZ3NmlRclJzOFI\
wWlcrNGIxR1d5ZG1XczJHQU1GV3diaXRmMm5JWEgzT3FIS1Z1OHMyUnZpQkdOaXZPS0d\
CSEh0QmRpRkVaWnZiN294SXdFREFPQmdOVkhROEJBZjhFQkFNQ0I0QXdDZ1lJS29aSXp\
qMEVBd0lEU1FBd1JnSWhBSTRQWWJ4dHNzSFAyVkh4XC90elVvUVwvU3N5ZEwzMERRSU5\
FdGNOOW1DVFhQQWlFQXZJYjNvK0ZPM0JUbmNMRnNhSlpSQWtkN3pPdXNuXC9cL1pLT2F\
FS2JzVkRpVT0iXSwiYWxnIjoiRVMyNTYifQ",
    "signature":"0TB5lr-cs1jqka2vNbQm3bBYWfLJd8zdVKIoV53eo2YgSITnKKY\
TvHMUw0wx9wdyuNVjNoAgLysNIgEvlcltBw"
  }]
}
~~~~
{: #ExampleVoucherResponsefigure title='Example Voucher-Response from MASA' artwork-align="left"}


## Example Voucher, MASA issued Voucher with additional Registrar signature (from MASA to Pledge, via Registrar and Registrar-Agent)

The following is an example voucher-response from MASA to Pledge via Registrar and Registrar-Agent, in "General JWS JSON Serialization".
The message size of this Voucher is: 3006 bytes

~~~~
=============== NOTE: '\' line wrapping per RFC 8792 ================

{
  "payload":"eyJpZXRmLXZvdWNoZXI6dm91Y2hlciI6eyJhc3NlcnRpb24iOiJhZ2V\
udC1wcm94aW1pdHkiLCJzZXJpYWwtbnVtYmVyIjoiMDEyMzQ1Njc4OSIsIm5vbmNlIjo\
iUUJiSXMxNTJzbkFvVzdSeVFMWENvZz09IiwiY3JlYXRlZC1vbiI6IjIwMjItMDktMjl\
UMDM6Mzc6MjYuMzgyWiIsInBpbm5lZC1kb21haW4tY2VydCI6Ik1JSUJwRENDQVVtZ0F\
3SUJBZ0lHQVcwZUx1SCtNQW9HQ0NxR1NNNDlCQU1DTURVeEV6QVJCZ05WQkFvTUNrMTV\
RblZ6YVc1bGMzTXhEVEFMQmdOVkJBY01CRk5wZEdVeER6QU5CZ05WQkFNTUJsUmxjM1J\
EUVRBZUZ3MHhPVEE1TVRFd01qTTNNekphRncweU9UQTVNVEV3TWpNM016SmFNRFV4RXp\
BUkJnTlZCQW9NQ2sxNVFuVnphVzVsYzNNeERUQUxCZ05WQkFjTUJGTnBkR1V4RHpBTkJ\
nTlZCQU1NQmxSbGMzUkRRVEJaTUJNR0J5cUdTTTQ5QWdFR0NDcUdTTTQ5QXdFSEEwSUF\
CT2t2a1RIdThRbFQzRkhKMVVhSTcrV3NIT2IwVVMzU0FMdEc1d3VLUURqaWV4MDYvU2N\
ZNVBKaWJ2Z0hUQitGL1FUamdlbEhHeTFZS3B3Y05NY3NTeWFqUlRCRE1CSUdBMVVkRXd\
FQi93UUlNQVlCQWY4Q0FRRXdEZ1lEVlIwUEFRSC9CQVFEQWdJRU1CMEdBMVVkRGdRV0J\
CVG9aSU16UWRzRC9qLytnWC83Y0JKdWNIL1htakFLQmdncWhrak9QUVFEQWdOSkFEQkd\
BaUVBdHhRMytJTEdCUEl0U2g0YjlXWGhYTnVocVNQNkgrYi9MQy9mVllEalE2b0NJUUR\
HMnVSQ0hsVnEzeWhCNThUWE1VYnpIOCtPbGhXVXZPbFJEM1ZFcURkY1F3PT0ifX0",
  "signatures":[{
    "protected":"eyJ4NWMiOlsiTUlJQmt6Q0NBVGlnQXdJQkFnSUdBV0ZCakNrWU1\
Bb0dDQ3FHU000OUJBTUNNRDB4Q3pBSkJnTlZCQVlUQWtGUk1SVXdFd1lEVlFRS0RBeEt\
hVzVuU21sdVowTnZjbkF4RnpBVkJnTlZCQU1NRGtwcGJtZEthVzVuVkdWemRFTkJNQjR\
YRFRFNE1ERXlPVEV3TlRJME1Gb1hEVEk0TURFeU9URXdOVEkwTUZvd1R6RUxNQWtHQTF\
VRUJoTUNRVkV4RlRBVEJnTlZCQW9NREVwcGJtZEthVzVuUTI5eWNERXBNQ2NHQTFVRUF\
3d2dTbWx1WjBwcGJtZERiM0p3SUZadmRXTm9aWElnVTJsbmJtbHVaeUJMWlhrd1dUQVR\
CZ2NxaGtqT1BRSUJCZ2dxaGtqT1BRTUJCd05DQUFTQzZiZUxBbWVxMVZ3NmlRclJzOFI\
wWlcrNGIxR1d5ZG1XczJHQU1GV3diaXRmMm5JWEgzT3FIS1Z1OHMyUnZpQkdOaXZPS0d\
CSEh0QmRpRkVaWnZiN294SXdFREFPQmdOVkhROEJBZjhFQkFNQ0I0QXdDZ1lJS29aSXp\
qMEVBd0lEU1FBd1JnSWhBSTRQWWJ4dHNzSFAyVkh4XC90elVvUVwvU3N5ZEwzMERRSU5\
FdGNOOW1DVFhQQWlFQXZJYjNvK0ZPM0JUbmNMRnNhSlpSQWtkN3pPdXNuXC9cL1pLT2F\
FS2JzVkRpVT0iXSwidHlwIjoidm91Y2hlci1qd3MranNvbiIsImFsZyI6IkVTMjU2In0\
",
    "signature":"ShqW8uFRkuAXIzjAhB4bolMMndcY7GYq3Kbo94yvGtjCaxEX3Hp\
6QXZUTEJ_kulQ1G7DnaU4igDPdUGtcV9Lkw"},{
    "protected":"eyJ4NWMiOlsiTUlJQjRqQ0NBWWlnQXdJQkFnSUdBWFk3MmJiWk1\
Bb0dDQ3FHU000OUJBTUNNRFV4RXpBUkJnTlZCQW9NQ2sxNVFuVnphVzVsYzNNeERUQUx\
CZ05WQkFjTUJGTnBkR1V4RHpBTkJnTlZCQU1NQmxSbGMzUkRRVEFlRncweU1ERXlNRGN\
3TmpFNE1USmFGdzB6TURFeU1EY3dOakU0TVRKYU1ENHhFekFSQmdOVkJBb01DazE1UW5\
WemFXNWxjM014RFRBTEJnTlZCQWNNQkZOcGRHVXhHREFXQmdOVkJBTU1EMFJ2YldGcGJ\
sSmxaMmx6ZEhKaGNqQlpNQk1HQnlxR1NNNDlBZ0VHQ0NxR1NNNDlBd0VIQTBJQUJCazE\
2S1wvaTc5b1JrSzVZYmVQZzhVU1I4XC91czFkUFVpWkhNdG9rU2RxS1c1Zm5Xc0JkK3F\
STDdXUmZmZVdreWdlYm9KZklsbHVyY2kyNXduaGlPVkNHamV6QjVNQjBHQTFVZEpRUVd\
NQlFHQ0NzR0FRVUZCd01CQmdnckJnRUZCUWNESERBT0JnTlZIUThCQWY4RUJBTUNCNEF\
3U0FZRFZSMFJCRUV3UDRJZGNtVm5hWE4wY21GeUxYUmxjM1F1YzJsbGJXVnVjeTFpZEM\
1dVpYU0NIbkpsWjJsemRISmhjaTEwWlhOME5pNXphV1Z0Wlc1ekxXSjBMbTVsZERBS0J\
nZ3Foa2pPUFFRREFnTklBREJGQWlCeGxkQmhacTBFdjVKTDJQcldDdHlTNmhEWVcxeUN\
PXC9SYXVicEM3TWFJRGdJaEFMU0piZ0xuZ2hiYkFnMGRjV0ZVVm9cL2dHTjBcL2p3ekp\
aMFNsMmg0eElYazEiXSwidHlwIjoidm91Y2hlci1qd3MranNvbiIsImFsZyI6IkVTMjU\
2In0",
    "signature":"N4oXV48V6umsHMKkhdSSmJYFtVb6agjD32uXpIlGx6qVE7Dh0-b\
qhRRyjnxp80IV_Fy1RAOXIIzs3Q8CnMgBgg"
  }]
}
~~~~
{: #ExampleVoucherResponseWithRegSignfigure title='Example Voucher-Response from MASA, with additional Registrar signature' artwork-align="left"}

# HTTP-over-TLS operations between Registrar-Agent and Pledge {#pledgehttps}

The use of HTTP-over-TLS between Registrar-Agent and pledge has been identified as an optional mechanism.

Provided that the key-agreement in the underlying TLS protocol connection can be properly authenticated, the use of TLS provides privacy for the voucher and enrollment operations between the pledge and the Registrar-Agent.
The authenticity of the onboarding and enrollment is not dependant upon the security of the TLS connection.

The use of HTTP-over-TLS is not mandated by this document for a number of reasons:

1. A certificate is generally required in order to do TLS.  While there are other modes of authentication including PSK, various EAP methods and raw public key, they do no help as there is no previous relationship between the Registrar-Agent.

2. The pledge can use it's IDevID certificate to authenticate itself, but {{?RFC9525}} DNS-ID methods do not apply as the pledge does not have a FQDN.  Instead a new mechanism is required, which authenticates the X520SerialNumber DN attribute which must be present in every IDevID.

If the Registrar-Agent has a preconfigured list of which product-serial-number(s), from which manufacturers it expects to see, then it can attempt to match this pledge against a list of potential devices.

In many cases only the list of manufacturers is known ahead of time, so at most the Registrar-Agent can show the X520SerialNumber to the (human) operator who may then attempt to confirm that they are standing in front of a device with that product-serial-number.
The use of scannable QRcodes may help automate this in some cases.

3. The CA used to sign the IDevID will be a manufacturer private PKI as described in {{?I-D.irtf-t2trg-taxonomy-manufacturer-anchors, Section 4.1}}.
The anchors for this PKI will never be part of the public WebPKI anchors which are distributed with most smartphone operating systems.
A Registrar-Agent application will need to use different APIs in order to initiate an HTTPS connection without performing WebPKI verification.
The application will then have to do it's own certificate chain verification against a store of manufacturer trust anchors.
In the Android ecosystem this involved use of a customer TrustManager: many application developers do not create these correctly, and there is significant push to remove this option as it has repeatedly resulted in security failures. See {{androidtrustfail}}

4. The use of the Host: (or :authority in HTTP/2) is explained in {{?RFC9110, Section 7.2}}. This header is mandatory, and so a compliant HTTPS client is going to insert it.
But, the contents of this header will at best be an IP address that came from the discovery process.
The pledge MUST therefore ignore the Host: header when it processes requests, and the pledge MUST NOT do any kind of name-base virtual hosting using the IP address/port combination.
Note that there is no requirement for the pledge to operate it's BRSKI-PRM service on port 80 or port 443, so if there is no reason for name-based virtual hosting.

5. Note that an Extended Key Usage (EKU) for TLS WWW Server authentication cannot be expected in the pledge's IDevID certificate.
IDevID certificates are intended to be widely useable and EKU does not support that use.

# History of Changes [RFC Editor: please delete] {#app_history}

Proof of Concept Code available

From IETF draft 12 -> IETF draft 13:

* Deleted figure in Section "Request Artifact: Pledge Voucher-Request Trigger (tPVR)" for JSON representation of tPVR, as it has been replaced by CDDL
* Updated reason-content description in status response messages (enroll-status, voucher-status, and status-response.
* Updated CDDL source code integration to allow for automatic verification
* Reordered description in section {{pvr}} to better match the order of communication and artifact processing.

From IETF draft 11 -> IETF draft 12:

* Updated acknowledgements to reflect early reviews
* Addressed Shepherd review part 2 (Pull Request #132); containing: terminology alignment, structural improvements of the document; deletion of leftovers from previous draft versions; change of definitions to CDDL, when no YANG is available


From IETF draft 10 -> IETF draft 11:

* issue #79, clarified that BRSKI discovery in the context of BRSKI-PRM is not needed in {{discovery_uc2_reg}}.
* issue #103, removed step 6 in verification handling for the wrapped CA certificate provisioning as only applicable after enrollment {{cacerts}}
* issue #128: included notation of nomadic operation of the Registrar-Agent in {{architecture}}, including proposed text from PR #131
* issue #130, introduced DNS service discovery name for brski_pledge to enable discovery by the Registrar-Agent in {{iana-con}}
* removed unused reference RFC 5280
* removed site terminology
* deleted duplicated text in {{pledge_ep}}
* clarified registrar discovery and relation to BRSKI-Discovery in {{discovery_uc2_reg}}
* clarified discovery of pledges by the Registrar-Agent in {{discovery_uc2_ppa}}, deleted reference to GRASP as handled in BRSKI-Discovery
* addressed comments from SECDIR early review

From IETF draft 09 -> IETF draft 10:

* issue #79, clarified discovery in the context of BRSKI-PRM and included information about future discovery enhancements in a separate draft in {{discovery_uc2_reg}}.
* issue #93, included information about conflict resolution in mDNS and GRASP in {{discovery_uc2_ppa}}
* issue #103, included verification handling for the wrapped CA certificate provisioning in {{cacerts}}
* issue #106, included additional text to elaborate more the registrar status handling in {{vstatus}} and {{estatus}}
* issue #116, enhanced DoS description in {{sec_cons-dos}}
* issue #120, included statement regarding pledge host header processing in {{pledge_ep}}
* issue #122, availability of product-serial-number information on registrar agent clarified in {{tpvr}}
* issue #123, Clarified usage of alternative voucher formats in  {{rvr-proc}}
* issue #124, determination of pinned domain certificate done as in RFC 8995 included in {{exchanges_uc2_2_vc}}
* issue #125, remove strength comparison of voucher assertions in {{agt_prx}} and {{exchanges_uc2}}
* issue #130, aligned the usage of site and domain throughout the document
* changed naming of registrar certificate from LDevID(RegAgt) to EE (RegAgt) certificate throughout the document
* change x5b to x5bag according to {{RFC9360}}
* updated JSON examples -> "signature": BASE64URL(JWS Signature)

From IETF draft 08 -> IETF draft 09:

* issue #80, enhanced {{discovery_uc2_ppa}} with clarification on the product-serial-number and the inclusion of GRASP
* issue #81, enhanced introduction with motivation for agent_signed_data
* issue #82, included optional TLS protection of the communication link between Registrar-Agent and pledge in the introduction {{req-sol}}, and {{tpvr}}
* issue #83, enhanced {{tper}} and {{pvr}} with note to re-enrollment
* issue #87, clarified available information at the Registrar-Agent in {{tpvr}}
* issue #88, clarified, that the PVR in {{tpvr}} and PER in {{tper}} may contain the certificate chain. If not contained it MUST be available at the registrar.
* issue #91, clarified that a separate HTTP connection may also be used to provide the PER in {{per}}
* resolved remaining editorial issues discovered after WGLC (responded to on the mailing list in Reply 1 and Reply 2) resulting in more consistent descriptions
* issue #92: kept separate endpoint for wrapped CSR on registrar {{req_cacerts}}
* issue #94: clarified terminology (possess vs. obtained)
* issue #95: clarified optional IDevID CA certificates on Registrar-Agent
* issue #96: updated exchangesfig_uc2_3 to correct to just one CA certificate provisioning
* issue #97: deleted format explanation in exchanges_uc2_3 as it may be misleading
* issue #99: motivated verification of second signature on voucher in {{voucher}}
* issue #100: included negative example in {{vstat}}

* issue #101: included handling if {{voucher}} voucher telemetry information has not been received by the Registrar-Agent
* issue #102: relaxed requirements for CA certs provisioning in {{cacerts}}
* issue #105: included negative example in {{estat}}
* issue #107: included example for certificate revocation in {{estatus}}
* issue	#108: renamed heading to Pledge-Status Request of {{query}}
* issue #111: included pledge-status response processing for authenticated requests in {{query}}
* issue #112: added "Example key word in pledge-status response in {{stat_res}}
* issue #113: enhanced description of status reply for "factory-default" in  {{query}}
* issue #114: Consideration of optional TLS usage in Privacy Considerations
* issue #115: Consideration of optional TLS usage in Privacy Considerations to protect potentially privacy related information in the bootstrapping like status information, etc.
* issue #116: Enhanced DoS description and mitigation options in security consideration section
* updated references


From IETF draft 07 -> IETF draft 08:

* resolved editorial issues discovered after WGLC (still open issues remaining)
* resolved first comments from the Shepherd review as discussed in PR #85 on the ANIMA github

From IETF draft 06 -> IETF draft 07:

* WGLC resulted in a removal of the voucher enhancements completely from this document to RFC 8366bis, containing all enhancements and augmentations of the voucher, including the voucher-request as well as the tree diagrams
* smaller editorial corrections

From IETF draft 05 -> IETF draft 06:

* Update of list of reviewers
* Issue #67, shortened the pledge endpoints to prepare for constraint deployments
* Included table for new endpoints on the registrar in the overview of the Registrar-Agent
* addressed review comments from SECDIR early review (terminology clarifications, editorial improvements)
* addressed review comments from IOTDIR early review (terminology clarifications, editorial improvements)

From IETF draft 04 -> IETF draft 05:

* Restructured document to have a distinct section for the object flow and handling and shortened introduction, issue #72
* Added security considerations for using mDNS without a specific product-serial-number, issue #75
* Clarified pledge-status responses are cumulative, issue #73
* Removed agent-sign-cert from trigger data to save bandwidth and remove complexity through options, issue #70
* Changed terminology for LDevID(Reg) certificate to registrar LDevID certificate, as it does not need to be an LDevID, issue #66
* Added new protected header parameter (created-on) in PER to support freshness validation, issue #63
* Removed reference to CAB Forum as not needed for BRSKI-PRM specifically, issue #65
* Enhanced error codes in section 5.5.1, issue #39, #64
* Enhanced security considerations and privacy considerations, issue #59
* Issue #50 addressed by referring to the utilized enrollment protocol
* Issue #47 MASA verification of LDevID(RegAgt) to the same registrar LDevID certificate domain CA
* Reworked terminology of "enrollment object", "certification object", "enrollment request object", etc., issue #27
* Reworked all message representations to align with encoding
* Added explanation of MASA requiring domain CA cert in section 5.5.1 and section 5.5.2, issue #36
* Defined new endpoint for pledge bootstrapping status inquiry, issue #35 in section {{query}}, IANA considerations and section {{pledge_ep}}
* Included examples for several objects in section {{examples}} including message example sizes, issue #33
* PoP for private key to registrar certificate included as mandatory, issues #32 and #49
* Issue #31, clarified that combined pledge may act as client/server for further (re)enrollment
* Issue #42, clarified that Registrar needs to verify the status responses with and ensure that they match the audit log response from the MASA, otherwise it needs drop the pledge and revoke the certificate
* Issue #43, clarified that the pledge shall use the create time from the trigger message if the time has not been synchronized, yet.
* Several editorial changes and enhancements to increasing readability.

From IETF draft 03 -> IETF draft 04:

* In deep Review by Esko Dijk lead to issues #22-#61, which are bein stepwise integrated
* Simplified YANG definition by augmenting the voucher-request from RFC 8995 instead of redefining it.
* Added explanation for terminology "endpoint" used in this document, issue #16
* Added clarification that Registrar-Agent may collect PVR or PER or both in one run, issue #17
* Added a statement that nonceless voucher may be accepted, issue #18
* Simplified structure in section {{sup-env}}, issue #19
* Removed join proxy in {{uc2figure}} and added explanatory text, issue #20
* Added description of pledge-CAcerts endpoint plus further handling of providing a wrapped CA certs response to the pledge in section {{cacerts}}; also added new required registrar endpoint (section {{pvr}} and IANA considerations) for the registrar to provide a wrapped CA certs response, issue #21
* utilized defined abbreviations in the document consistently, issue #22
* Reworked text on discovery according to issue #23 to clarify scope and handling
* Added several clarifications based on review comments

From IETF draft 02 -> IETF draft 03:

* Updated examples to state "base64encodedvalue==" for x5c occurrences
* Include link to SVG graphic as general overview
* Restructuring of section 5 to flatten hierarchy
* Enhanced requirements and motivation in {{req-sol}}
* Several editorial improvements based on review comments

From IETF draft 01 -> IETF draft 02:

* Issue #15 included additional signature on voucher from registrar in section {{pvr}} and section {{agt_prx}}
  The verification of multiple signatures is described in section {{voucher}}

* Included representation for General JWS JSON Serialization for examples

* Included error responses from pledge if it is not able to create a Pledge-Voucher-Request or an enrollment request in section {{tpvr}}

* Removed open issue regarding handling of multiple CSRs and Enroll-Responses during the bootstrapping as the initial target it the provisioning of a generic LDevID certificate. The defined endpoint on the pledge may also be used for management of further certificates.

From IETF draft 00 -> IETF draft 01:

* Issue #15 lead to the inclusion of an option for an additional signature of the registrar on the voucher received from the MASA before forwarding to the Registrar-Agent to support verification of POP of the registrars private key in section {{pvr}} and exchanges_uc2_3.

* Based on issue #11, a new endpoint was defined for the registrar to enable delivery of the wrapped enrollment request from the pledge (in contrast to plain PKCS#10 in simple enroll).

* Decision on issue #8 to not provide an additional signature on the enrollment-response object by the registrar. As the Enroll-Response will only contain the generic LDevID certificate. This credential builds the base for further configuration outside the initial enrollment.

* Decision on issue #7 to not support multiple CSRs during the bootstrapping, as based on the generic LDevID certificate the pledge may enroll for further certificates.

* Closed open issue #5 regarding verification of ietf-ztp-types usage as verified
  via a proof-of-concept in section {{tpvr}}.

* Housekeeping: Removed already addressed open issues stated in the draft directly.

* Reworked text in from introduction to section pledge-responder-mode

* Fixed "serial-number" encoding in PVR/RVR

* Added prior-signed-voucher-request in the parameter description of the
  registrar-voucher-request in {{pvr}}.

* Note added in {{pvr}} if sub-CAs are used, that the
  corresponding information is to be provided to the MASA.

* Inclusion of limitation section (pledge sleeps and needs to be waked
  up. Pledge is awake but Registrar-Agent is not available) (Issue #10).

* Assertion-type aligned with voucher in RFC8366bis, deleted related
  open issues. (Issue #4)

* Included table for endpoints in {{pledge_ep}} for better readability.

* Included registrar authorization check for Registrar-Agent during
  TLS handshake  in section {{pvr}}. Also enhanced figure
  {{exchangesfig_uc2_all}} with the authorization step on TLS level.

* Enhanced description of registrar authorization check for Registrar-Agent
  based on the agent-signed-data in section {{pvr}}. Also
  enhanced figure {{exchangesfig_uc2_all}} with the authorization step
  on Pledge-Voucher-Request level.

* Changed agent-signed-cert to an array to allow for providing further
  certificate information like the issuing CA cert for the LDevID(RegAgt)
  certificate in case the registrar and the Registrar-Agent have different
  issuing CAs in {{exchangesfig_uc2_all}} (issue #12).
  This also required changes in the YANG module in {{I-D.ietf-anima-rfc8366bis}}

* Addressed YANG warning (issue #1)

* Inclusion of examples for a trigger to create a Pledge-Voucher-Request
  and an Pledge Enroll-Request.

From IETF draft-ietf-anima-brski-async-enroll-03 -> IETF anima-brski-prm-00:

* Moved UC2 related parts defining the Pledge in Responder Mode from
  draft-ietf-anima-brski-async-enroll-03 to this document
  This required changes and adaptations in several sections to remove
  the description and references to UC1.

* Addressed feedback for voucher-request enhancements from YANG doctor
  early review, meanwhile moved to {{I-D.ietf-anima-rfc8366bis}} as well as in the security considerations (formerly named ietf-async-voucher-request).

* Renamed ietf-async-voucher-request to IETF-voucher-request-prm to
  to allow better listing of voucher related extensions; aligned with
  constraint voucher (#20)

* Utilized ietf-voucher-request-async instead of ietf-voucher-request
  in voucher exchanges to utilize the enhanced voucher-request.

* Included changes from draft-ietf-netconf-sztp-csr-06 regarding the
  YANG definition of csr-types into the enrollment request exchange.

From IETF draft 02 -> IETF draft 03:

* Housekeeping, deleted open issue regarding YANG voucher-request
  in {{tpvr}} as voucher-request was
  enhanced with additional leaf.

* Included open issues in YANG model in {{architecture}} regarding assertion
  value agent-proximity and csr encapsulation using SZTP sub module).

From IETF draft 01 -> IETF draft 02:

* Defined call flow and objects for interactions in UC2. Object format
  based on draft for JOSE signed voucher artifacts and aligned the
  remaining objects with this approach in {{exchanges_uc2}}.

* Terminology change: issue #2 pledge-agent -> Registrar-Agent to
  better underline Registrar-Agent relation.

* Terminology change: issue #3 PULL/PUSH -> pledge-initiator-mode
  and pledge-responder-mode to better address the pledge operation.

* Communication approach between pledge and Registrar-Agent
  changed by removing TLS-PSK (former section TLS establishment)
  and associated references to other drafts in favor of relying on
  higher layer exchange of signed data objects. These data objects
  are included also in the Pledge-Voucher-Request and lead to an
  extension of the YANG module for the voucher-request (issue #12).

* Details on trust relationship between Registrar-Agent and
  registrar (issue #4, #5, #9) included in {{architecture}}.

* Recommendation regarding short-lived certificates for
  Registrar-Agent authentication towards registrar (issue #7) in
  the security considerations.

* Introduction of reference to Registrar-Agent signing certificate using SKID
  in Registrar-Agent signed data (issue #37).

* Enhanced objects in exchanges between pledge and Registrar-Agent
  to allow the registrar to verify agent-proximity to the pledge
  (issue #1) in {{exchanges_uc2}}.

* Details on trust relationship between Registrar-Agent and
  pledge (issue #5) included in {{architecture}}.

* Split of use case 2 call flow into sub sections in {{exchanges_uc2}}.

From IETF draft 00 -> IETF draft 01:

* Update of scope in {{sup-env}} to include in
  which the pledge acts as a server. This is one main motivation
  for use case 2.

* Rework of use case 2 in {{architecture}} to consider the
  transport between the pledge and the pledge-agent. Addressed is
  the TLS channel establishment between the pledge-agent and the
  pledge as well as the endpoint definition on the pledge.

* First description of exchanged object types (needs more work)

* Clarification in discovery options for enrollment endpoints at
  the domain registrar based on well-known endpoints do not
  result in additional /.well-known URIs. Update of the illustrative example.
  Note that the change to /brski for the voucher related endpoints
  has been taken over in the BRSKI main document.

* Updated references.

* Included Thomas Werner as additional author for the document.

From individual version 03 -> IETF draft 00:

* Inclusion of discovery options of enrollment endpoints at
  the domain registrar based on well-known endpoints in
  new section as replacement of section 5.1.3
  in the individual draft. This is intended to support both use
  cases in the document. An illustrative example is provided.

* Missing details provided for the description and call flow in
  pledge-agent use case {{architecture}}, e.g. to
  accommodate distribution of CA certificates.

* Updated CMP example in to use lightweight CMP instead of CMP, as the draft already provides
  the necessary /.well-known endpoints.

* Requirements discussion moved to separate section in
  {{req-sol}}. Shortened description of proof
  of identity binding and mapping to existing protocols.

* Removal of copied call flows for voucher exchange and registrar
  discovery flow from {{!RFC8995}} in UC1 to avoid doubling or text or
  inconsistencies.

* Reworked abstract and introduction to be more crisp regarding
  the targeted solution. Several structural changes in the document
  to have a better distinction between requirements, use case
  description, and solution description as separate sections.
  History moved to appendix.

From individual version 02 -> 03:

* Update of terminology from self-contained to authenticated
  self-contained object to be consistent in the wording and to
  underline the protection of the object with an existing
  credential. Note that the naming of this object may be discussed.
  An alternative name may be attestation object.

* Simplification of the architecture approach for the initial use
  case having an offsite PKI.

* Introduction of a new use case utilizing authenticated
  self-contain objects to onboard a pledge using a commissioning
  tool containing a pledge-agent. This requires additional changes
  in the BRSKI call flow sequence and led to changes in the
  introduction, the application example,and also in the
  related BRSKI-PRM call flow.

From individual version 01 -> 02:

* Update of introduction text to clearly relate to the usage of
  IDevID and LDevID.

* Update of description of architecture elements and
  changes to BRSKI in {{architecture}}.

* Enhanced consideration of existing enrollment protocols in the
  context of mapping the requirements to existing solutions in
  {{req-sol}}.

From individual version 00 -> 01:

* Update of examples, specifically for building automation as
  well as two new application use cases in {{sup-env}}.

* Deletion of asynchronous interaction with MASA to not
  complicate the use case. Note that the voucher exchange can
  already be handled in an asynchronous manner and is therefore
  not considered further. This resulted in removal of the
  alternative path the MASA in Figure 1 and the associated
  description in {{architecture}}.

* Enhancement of description of architecture elements and
  changes to BRSKI in {{architecture}}.

* Consideration of existing enrollment protocols in the context
  of mapping the requirements to existing solutions in {{req-sol}}.

* New section starting with the
  mapping to existing enrollment protocols by collecting
  boundary conditions.
