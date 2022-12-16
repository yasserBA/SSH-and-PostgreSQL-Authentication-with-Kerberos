# SSH and PostgreSQL Authentication with Kerberos
## Introduction 

Kerberos is a network authentication protocol, which is designed to allow users to prove their identities over a non-secure network 
in a secure manner.

This protocol is an industry-standard protocol for secure authentication with the messages designed to against spying and replay attacks. It has been built into a wide range of software such as, Chrome, Firefox, OpenSSH, Putty, OpenLDAP, Thunderbird and PostgreSQL etc. There are some open source implementations available such as, krb5 implemented by MIT used by most of the Unix-like Operating Systems, and heimdal used by OSX. Before dive into any detailed environment setup, some key concepts used in Kerberos need to be explained such as: 

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

![image](https://user-images.githubusercontent.com/120678001/207984233-cd3cf42b-9d68-42e5-8beb-6f3c6b50544b.png)

## HostName and IP adresses 

First of all, we will prepare the working environment for our project.
We need three machines. In my case, I'm using three ubuntu virtual machines in VMware. These three machines will be the client, the service server and the KDC.

We will start by setting hostnames for each machine:

![image](https://user-images.githubusercontent.com/120678001/208042802-03febd45-6715-4fae-9a56-59b27badf1d8.png)

PS: We need to open a new terminal for changes to take effect.

Then, we will match these hostnames to the corresponding IP addresses on the three machines using the /etc/hosts file.

![image](https://user-images.githubusercontent.com/120678001/208043506-20b6b849-e630-4824-a13b-0439e199a630.png)

PS: The configuration of the Ip addresses and the hostnames must be done in all the other machines as well.

Once the configuration of the environment is complete, we need to check that everything is working properly by using the ping command to ensure that the three machines are reachable.

![image](https://user-images.githubusercontent.com/120678001/208044000-c6ee3a76-41e8-496e-9f2e-510871b8547d.png)

![image](https://user-images.githubusercontent.com/120678001/208044072-9fd8f5ee-8f5c-4831-a914-11f86fbddae4.png)

## Key Distribution Center Machine Configuration

We need to install these packages on the KDC machine

```
$ sudo apt-get update 
$ sudo apt-get install krb5-kdc krb5-admin-server 
```


![image](https://user-images.githubusercontent.com/120678001/208044422-8f58b12f-7c3e-4a23-9abf-30f346f83a04.png)

During the installation, we should configure the

•	Realm : TEKUP.TN

![image](https://user-images.githubusercontent.com/120678001/208044576-37108f5e-225a-4bbb-84cb-a5a0981d9fa4.png)

The Kerberos server : ' kdc.tekup.tn '

![image](https://user-images.githubusercontent.com/120678001/208045197-1adec3c1-d4b1-41de-ac39-ef51834145ee.png)

The administrative server : ' kdc.tekup.tn '

![image](https://user-images.githubusercontent.com/120678001/208045294-3913a9a7-eec5-4f1b-8fb0-010f2663c732.png)

- Now, we can observe that the configuration of the files /etc/krb5.conf is already ready without even trying to change it manually.

```  $ sudo nano /etc/krb5.conf   ```

This is the result for krb5.conf file : 

![image](https://user-images.githubusercontent.com/120678001/208045658-5b5f4d28-4b81-4ec7-8643-0bd446856753.png)

![image](https://user-images.githubusercontent.com/120678001/208045705-8b8be71b-d749-440f-afda-1e926a12ef46.png)

![image](https://user-images.githubusercontent.com/120678001/208045760-7b6be0f2-9a73-46de-9b40-a1d03fcf94d9.png)

As well as the configuration of the file /etc/krb5kdc/kdc.conf 

```  $ sudo nano /etc/krb5kdc/kdc.conf   ```

This is the result for kdc.conf file :

![image](https://user-images.githubusercontent.com/120678001/208046222-b4205f3a-6669-424e-b81b-c83fff432f77.png)

Realm is a logical network, similar to a domain, that all the users and servers sharing the same Kerberos database belong to.

The master key for this KDC database needs to be set once the installation is complete

```  $ krb5_newrealm  ```

![image](https://user-images.githubusercontent.com/120678001/208047353-94c36dac-3deb-4b0e-a60a-c088c4a6a3be.png)

Next, we need to grant all access rights to the Kerberos database to admin principal root/admin using the configuration file /etc/krb5kdc/kadm5.acl 

```  $ nano kadm5.acl  ```

In this file, we need to add the following line

```  */admin@tekup.tn *  ```

![image](https://user-images.githubusercontent.com/120678001/208047673-4fc43939-a1bf-426e-8339-bd60081ba50f.png)

For changes to take effect, we need to restart the following service

```  $ sudo service krb5-admin-server restart  ```

The users and services in a realm are defined as a principal in Kerberos. These principals are managed by an admin user that we need to create manually :

Kadmin.local is a KDC database administration program. We tried to create with this tool a new principal in the TEKUP.TN realm.











