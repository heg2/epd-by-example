# PIX Feed

Transaction to register a patient in a community. Primary systems shall use this transaction to register
patient data to be able to provide and retrieve documents to the patients EPR.

- [Overview](#overview)
- [Transaction](#transaction)
	* [Message Semantics](#message-semantics)
		- [Request Message](#request-message)
		- [Response Message](#response-message)
	* [Transport Protocol](#transport-protocol)
	* [Audit Log](#audit-log)
- [Security Requirements](#security-requirements)
- [Test Opportunity](#test-opportunity)

# Overview

Primary systems shall use this transaction to register patient data with the local ID, the patient is
registered in the primary system. In the Swiss EPR the **[IHE PIX V3](https://profiles.ihe.net/ITI/TF/Volume1/ch-23.html)**
profile and transactions shall be used to register the patient data.  

To register the patient data the primary system shall perform a
**[Patient Identity Feed HL7 V3 \[ITI-44\]](https://profiles.ihe.net/ITI/TF/Volume2/ITI-44.html)**
transaction. In the feed request the primary system must provide the demographic data as provided by
the ZAS central service, which includes the name, birthdate, gender, nationality and the EPR-SPID.
Primary systems may provide other demographic data (e.g., address and other contact data).

The community response includes a success confirmation, or a error code in the case an error
occurred in the community during registration. In case of success the community stores the patient data
provided by the primary system, matches the data set to other patient data set registered by other
primary systems and assigns the patient data set to a master patient record and the master patient ID
(XAD-PID).

To perform the PIX V3 feed fo the EPR, primary systems must retrieve the demographic data and the
EPR-SPID from the ZAS central service. While the interface to be used by the communities is specified in
the ordinances to the Swiss electronic patient dossier, the interface for primary systems is not, since
communities provide simplified interfaces for primary systems to retrieve the data or included the
interface in the registration workflow. Please contact the community you want to connect to on
implementation details.   

# Transaction

## Message Semantics

Messages are encoded as described in the HL7 V3 standard with restrictions defined in the
**[IHE Patient Identity Feed HL7 V3](https://profiles.ihe.net/ITI/TF/Volume2/ITI-44.html)** profile
and the ordinances to the Swiss EPR.

### Request Message

Due to the genericity of the underlying **[HL7 V3](http://www.hl7.org)** standard, the request
message is quite lengthy. A raw version of a request message may be found
**[here](../samples/ITI-44_request_raw.xml)**.

For a step by step interpretation of the request message, see section below.

#### Message Interpretation

The request message is not very complex, but lengthy due to the genericity of the HL7 V3 standard.

The SOAP *Header* element shall convey the following information:

- *To* element: The URL of the provide and register document set service.
- *MessageID* element: a UUID of the message.
- *Action* element: The SOAP action identifier of the request as defined in the IHE ITI Technical Framework.

Optional elements may be included according to the specification in the **[W3C SOAP specification](https://www.w3.org/TR/2007/REC-soap12-part0-20070427/#L26866)**.

```
2  <soap:Header>
3   <Action xmlns="http://www.w3.org/2005/08/addressing">urn:hl7-org:v3:PRPA_IN201301UV02</Action>
4   <MessageID xmlns="http://www.w3.org/2005/08/addressing">urn:uuid:7a180388-6ba7-4cbc-bffe-dfcdc4e602b7</MessageID>
5   <To xmlns="http://www.w3.org/2005/08/addressing">http://epd-core.int.adswissnet.healthcare/mpi/pixmanager</To>
6   <ReplyTo xmlns="http://www.w3.org/2005/08/addressing">
7    <Address>http://www.w3.org/2005/08/addressing/anonymous</Address>
8   </ReplyTo>
9  </soap:Header>
```

For the patient identity feed no *Security* header element is required, since in the Swiss EPR the access to the patient
data is authorized for all applications, which are registered in the community and authenticate with a client certificate
(see section **[Security Requirements](PIXFeed.md#security-requirements)**).

The SOAP *Body* element conveys the administrative information required for a PRPA_IN201305UV02 message in HL7 V3 syntax.

Primary systems shall set the following values:
- *creationTime*: A timestamp in unix time format.
- *sender* : The OID of the sender application initiating the request.
- *receiver*: The OID of the receiver application which shall respond to the request.

```
10  <soap:Body>
11   <PRPA_IN201301UV02 xmlns="urn:hl7-org:v3" ITSVersion="XML_1.0">
12    <id root="647aee99-56e7-46f5-ac26-bb691834204a"/>
13    <creationTime value="20200923113348"/>
14    <interactionId root="2.16.840.1.113883.1.6" extension="PRPA_IN201301UV02"/>
15    <processingCode code="P"/>
16    <processingModeCode code="T"/>
17    <acceptAckCode code="AL"/>
18    <receiver typeCode="RCV">
19     <device classCode="DEV" determinerCode="INSTANCE">
20      <id root="1.3.6.1.4.1.21367.2017.2.4.98"/>
21     </device>
22    </receiver>
23    <sender typeCode="SND">
24     <device classCode="DEV" determinerCode="INSTANCE">
25      <id root="1.3.6.1.4.1.21367.2017.2.2.100"/>
26     </device>
27    </sender>
```

The patient data are encoded in a HL7 V3 *controlAct* object as follows:

```
30     <subject typeCode="SUBJ" contextConductionInd="false">
31      <registrationEvent classCode="REG" moodCode="EVN">
32       <id nullFlavor="NA"/>
33       <statusCode code="active"/>
34       <subject1 typeCode="SBJ">
35        <patient classCode="PAT">
36         <id root="1.3.6.1.4.1.21367.2017.2.5.75" extension="T944"/>
37         <statusCode code="active"/>
38         <patientPerson classCode="PSN" determinerCode="INSTANCE">
39          <name>
40           <given>OVIE</given>
41           <family qualifier="">BERGAN</family>
42          </name>
43          <administrativeGenderCode code="1" codeSystem="2.16.840.1.113883.5.1"/>
44          <birthTime value="20020329"/>
45          <addr>
46           <streetAddressLine>KONIZBERGSTRASSE</streetAddressLine>
47           <city>Bern</city>
48           <postalCode>3018</postalCode>
49          </addr>
50          <asOtherIDs classCode="ACCESS">
51           <id root="2.16.756.5.30.1.127.3.10.3" extension="761338420435200768"/>
52           <scopingOrganization classCode="ORG" determinerCode="INSTANCE">
53            <id root="2.16.756.5.30.1.127.3.10.3"/>
54           </scopingOrganization>
55          </asOtherIDs>
56         </patientPerson>
57         <providerOrganization classCode="ORG" determinerCode="INSTANCE">
58          <id root="1.3.6.1.4.1.21367.2017.2.5.75"/>
59          <name>Spital Administration</name>
60          <contactParty classCode="CON">
61           <telecom value="mailto:myHospital@test.ch"/>
62          </contactParty>
63         </providerOrganization>
64        </patient>
65       </subject1>
66       <custodian typeCode="CST">
67        <assignedEntity classCode="ASSIGNED">
68         <id root="1.3.6.1.4.1.21367.2017.2.5.75"/>
69         <assignedOrganization classCode="ORG" determinerCode="INSTANCE">
70          <name>Spital Administration</name>
71         </assignedOrganization>
72        </assignedEntity>
73       </custodian>
74      </registrationEvent>
75     </subject>
76    </controlActProcess>
```

The *subject* child element conveys the following information in its child elements.

The *patientPerson* child element conveys the patient data including:  
- *name*: conveying the given and the family names of the matching patient data.
- *administrativeGenderCode*: conveying the coded value of patient gender, taken from the value sets defined in
**[Annex 3](https://www.fedlex.admin.ch/eli/oc/2023/221/de/annexes)**.  
- *birthTime*: the data of birth of the matching patient data.
- *addr* : The address data of the patient.
- *asOtherIDs*: The assigned patient ID's.   
- *custodian*: Information on the provider organization.

The *asOtherId* elements shall include the

- local ID the patient is registered in the primary system, with the OID of the primary system in the *root* attribute, and the ID value in the *extension* attribute.

```
50          <asOtherIDs classCode="ACCESS">
51           <id root="2.16.756.5.30.1.127.3.10.3" extension="761338420435200768"/>
52           <scopingOrganization classCode="ORG" determinerCode="INSTANCE">
53            <id root="2.16.756.5.30.1.127.3.10.3"/>
54           </scopingOrganization>
55          </asOtherIDs>
```

- the EPR-SPID with the OID of the ZAS in the *root* attribute, and the value in the *extension* attribute.

The *custodian* element shall convey the OID of the provider organization in the *id* child element:

```
66       <custodian typeCode="CST">
67        <assignedEntity classCode="ASSIGNED">
68         <id root="1.3.6.1.4.1.21367.2017.2.5.75"/>
69         <assignedOrganization classCode="ORG" determinerCode="INSTANCE">
70          <name>Spital Administration</name>
71         </assignedOrganization>
72        </assignedEntity>
73       </custodian>
```

### Response Message

The PIX V3 Feed service responds with a message indicating the success of the transaction. A raw version of a response message may be found **[here](../samples/ITI-44_response.xml)**.

## Transport Protocol

The primary system shall send the request messages to the registry of the community using the http POST binding as defined in the **[W3C SOAP specification](https://www.w3.org/TR/2007/REC-soap12-part0-20070427/#L26866)**. It may look like:  

```
POST /PIXV3FeedService HTTP/1.1
Host: company.example.org
Accept-Encoding: gzip, deflate
Connection: Keep-Alive
Content-Type: application/soap+xml; charset="utf-8"
Content-Length: nnn  
```

## Audit Log

Primary systems shall store syslog messages to the audit record repository of the community using TLS transport protocol.
The audit message uses XML formatting as specified in **[RFC 3881](https://tools.ietf.org/html/rfc3881)** with restrictions
specified in the **[IHE ITI TF](https://ehealthsuisse.ihe-europe.net/gss/audit-messages/view.seam?id=701)** and the
**[Extension 1 to Annex5](https://www.fedlex.admin.ch/eli/oc/2023/221/de/annexes)** in the ordinances of the Swiss electronic patient record (see Section
1.5 "Requirements on ATNA").  

The following snippet shows a example audit message to be written by the primary system:

```
1 <?xml version='1.0' encoding='utf-8'?>
2 <AuditMessage>
3  <EventIdentification EventDateTime="2020-09-21T15:25:53.616+02:00" EventOutcomeIndicator="0" EventActionCode="C">
4   <EventID csd-code="110110" codeSystemName="DCM" originalText="Patient Record"/>
5   <EventTypeCode csd-code="ITI-44" codeSystemName="IHE Transactions" originalText="Patient Identity Feed"/>
6   <EventOutcomeDescription/>
7  </EventIdentification>
8  <ActiveParticipant AlternativeUserID="primary.system.alt.ID" UserID="primary.system.ID" UserIsRequestor="true" NetworkAccessPointID="https://my_primary_system.com/" NetworkAccessPointTypeCode="1">
9   <RoleIDCode csd-code="110153" codeSystemName="DCM" originalText="Source"/>
10  </ActiveParticipant>
11  <ActiveParticipant AlternativeUserID="plattform.mpi.alt.ID" UserID="plattform.mpi.ID" UserIsRequestor="false" NetworkAccessPointID="https://platform.com/mock/patientRegister" NetworkAccessPointTypeCode="1">
12   <RoleIDCode csd-code="110152" codeSystemName="DCM" originalText="Destination"/>
13  </ActiveParticipant>
14  <AuditSourceIdentification AuditSourceID="primary.system.alt.ID" AuditEnterpriseSiteID="2.16.756.5.30.1.174.1.5.1">
15   <AuditSourceTypeCode csd-code="4" codeSystemName="DCM" originalText="Client Process"/>
16  </AuditSourceIdentification>
17  <ParticipantObjectIdentification ParticipantObjectID="11234^^^&amp;2.16.756.5.30.1.174.1.9999.1&amp;ISO" ParticipantObjectTypeCode="1" ParticipantObjectTypeCodeRole="1">
18   <ParticipantObjectIDTypeCode csd-code="2" codeSystemName="RFC-3881" originalText="Patient Number"/>
19   <ParticipantObjectDetail type="II" value="ZGVmYXVsdA=="/>
20  </ParticipantObjectIdentification>
21  <ParticipantObjectIdentification ParticipantObjectID="urn:uuid:4d42d62d-30b7-4ad4-a9e2-c3f0dd1d50f9" ParticipantObjectTypeCode="2" ParticipantObjectTypeCodeRole="24">
22   <ParticipantObjectIDTypeCode csd-code="ITI-44" codeSystemName="IHE Transactions" originalText="Patient Identity Feed"/>
23   <ParticipantObjectQuery>
24    <!-- omitted for brevity -->
25   </ParticipantObjectQuery>
26   <ParticipantObjectDetail type="II" value="ZGVmYXVsdA=="/>
27  </ParticipantObjectIdentification>
28 </AuditMessage>
```

The message is made of the following blocks:
- *EventIdentification*: Event related information including the timestamp (line 3 .. 7).
- *ActiveParticipant*: Information related to the primary system performing the query (line 8 .. 10).
- *ActiveParticipant*: Information on the responding service endpoint (line 11 .. 13).
- *AuditSourceIdentification*: Information related to the primary system performing the query (line 14 .. 16)
- *ParticipantObjectIdentification*: Information on the patients EPR accessed (line 17 .. 20)
- *ParticipantObjectIdentification*: Request message related information including a UUencoded copy of the query (line 21 .. 27).

## Security Requirements  

To ensure privacy the transaction must be secured using https with mutual authentication, with X.509 certificates
(extended validation required) and client and server side certificate validation.

Note:
- Some test environments dropped the mutual authentication or TLS for testing purposes. Please contact your test system provider on the details.

# Test Opportunity

The transaction can be tested with the test suite of the **[EPR reference environment](./gazelle.md)**, test systems of the EPR communities or the **[EPR Playground](./playground.md)**.
