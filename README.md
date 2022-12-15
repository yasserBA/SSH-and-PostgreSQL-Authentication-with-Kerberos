# SSH and PostgreSQL Authentication with Kerberos
Introduction :
Kerberos is a network authentication protocol, which is designed to allow users to prove their identities over a non-secure network in a secure manner. This protocol is an industry-standard protocol for secure authentication with the messages designed to against spying and replay attacks. It has been built into a wide range of software such as, Chrome, Firefox, OpenSSH, Putty, OpenLDAP, Thunderbird and PostgreSQL etc. There are some open source implementations available such as, krb5 implemented by MIT used by most of the Unix-like Operating Systems, and heimdal used by OSX. Before dive into any detailed environment setup, some key concepts used in Kerberos need to be explained such as: 

•	AS : Authentication service.
•	TGS: Service issuing access tickets to services.
•	TGT: Ticket to access the TGS.
•	TS: Access ticket to the requested service.
•	Kuser: Secret key of the user, known by the user and the TGS.
•	Kservice: Secret key of the requested service, known by the service and the TGS.
•	Ktgs: Secret key of the TGS known to the TGS and the AS.
•	KsessionTGS: Session key between the user and the TGS.
•	Ksession: Session secret key between the user and the requested service.
•	TicketService: Access ticket to the requested service.


![main-qimg-3b5914f158064643ca4a10f75dd90d78](https://user-images.githubusercontent.com/120678001/207944504-ba5078ed-8cc8-4922-9bc8-c98864e307ab.gif)
