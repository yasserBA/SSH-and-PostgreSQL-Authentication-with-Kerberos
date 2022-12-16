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

### Kerberos Installation

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

### Adding Principals

The users and services in a realm are defined as a principal in Kerberos. These principals are managed by an admin user that we need to create manually :

Kadmin.local is a KDC database administration program. We tried to create with this tool a new principal in the TEKUP.TN realm

![image](https://user-images.githubusercontent.com/120678001/208048875-a25c051d-0ad8-45f6-8c5c-1994f1f2c3ed.png)

we can check if the root/admin principal has been created successfully or not by running the command: ``` $ list_principals ```

![image](https://user-images.githubusercontent.com/120678001/208049070-58b6944f-0242-4639-a9de-eb4d468fd799.png)

**We can observe all the information of the utilisateur principal**

After that, we want to create a host by adding a principal *host/kdc.tekup.tn*

![image](https://user-images.githubusercontent.com/120678001/208049399-e719d7c8-47fd-46bc-ba6c-903b715aa889.png)

### Creating A Keytab File

In this part, we  will create a keytab file (A database) which will be composed of entries, principals and encryption methods used for each principal.

We will begin by adding a new_entry in the keytab file for the root/admin 

``` 
$ Ktutil
ktutil: add_entry -password -p root/admin@TEKUP.TN -k 1 -e aes256-cts-hmac-sha1-96
```

![image](https://user-images.githubusercontent.com/120678001/208050495-37681de8-4359-4d94-9651-15a5780c17d0.png)

![image](https://user-images.githubusercontent.com/120678001/208050539-5ba9e693-f580-4f79-a8b2-16b69a0df24c.png)

Now we can see that the keytab is created :

 ![image](https://user-images.githubusercontent.com/120678001/208050732-3f4a3640-2e7f-42d5-b8f9-f13427fd4733.png) 

And The root/admin entry has been added successfully

Then, we will add another new_entry in the keytab file for the host/kdc.tekup.tn

![image](https://user-images.githubusercontent.com/120678001/208050942-e87a9447-ad69-4267-98fe-c50d044eab4f.png)

![image](https://user-images.githubusercontent.com/120678001/208050973-5725db9b-0df5-4c78-8726-3c4c4af7e068.png)

The host/kdc.tekup.tn entry has been added successfully

## Service SSH

### On the KDC machine 

We need to have **openssh-server** package installed on the KDC machine.

``` $ sudo apt install openssh-server ```


Then, we need to modify the /etc/ssh/ssh_config file for the ssh service to work

``` $ nano /etc/ssh/ssh_config ```


![image](https://user-images.githubusercontent.com/120678001/208051969-fe3c61dc-6ce8-453e-9774-0983fe16d027.png)

![image](https://user-images.githubusercontent.com/120678001/208052024-71286562-35b6-4795-b05b-1ee4c4814833.png)

### On the client machine

After installing the package Openssh-server and modifiying the file /etc/ssh/sshd_config as we did previously.

We will add the principal of a simple user who will try to access the SSH service using kerberos after successfully completing its configuration.

![image](https://user-images.githubusercontent.com/120678001/208052147-070891a9-34be-4e0b-b549-3f1758eb35c5.png)

We can check if the user has been added successfully or not

![image](https://user-images.githubusercontent.com/120678001/208052414-ae287988-22a5-474c-8ea5-f0239e3bbefe.png)

We will login as a simple user now and try to access the KDC machine with an ssh service

``` $ su -l utilisateur ```

![image](https://user-images.githubusercontent.com/120678001/208052665-2e29ea43-8a5d-4ce6-b79d-370d4afa83c4.png)

In order to access the KDC machine, we are still obligated to enter the machine’s password every time we want to have access to it

**Our goal is to grant a TGT to our client so he can access our KDC machine (SSH Service) without having to type its password**

![image](https://user-images.githubusercontent.com/120678001/208052854-81bc7b12-8472-4ad3-bad4-725d01d24745.png)

The user has his TGT and he will use it to grant access to the SSH Service

![image](https://user-images.githubusercontent.com/120678001/208052947-61b6d785-7718-43a3-b1b1-d3ab3af53d53.png)

*The user can access the KDC machine without having to type its password*

## Service 2: PostgreSQL

### Introduction

PostgreSQL supports many secure ways to authenticate users, and one typical way is to use GSSAPI with Kerberos. 

![image](https://user-images.githubusercontent.com/120678001/208054089-5b18968d-807f-4d79-89e7-e5b96f37a5e3.png)

When PostgreSQL authenticates a user with Kerberos, the overall processes in above diagram can be interpreted in below order.

•	Client initiates the authentication process, AS sends Client a temporary session key (grey key) encrypted with Client’s key (blue key)

•	Client uses the temporary session key to request services, TGS grants the services and sends two copies of the communication session keys (yellow key): one encrypted using temporary session key and another encrypted using Service Server’s key (green key)

•	Client forwards the communication session key to Service Server(PG) to confirm the user authentication. If it succeeded then both Client and Service Server will use the communication session key for the rest of the communication

We will to create another principal for the client machine and the service server machine.

### Creating Principals

**Create a principal for the client**

``` 
$ Kadmin.local
kadmin: add_principal rayen-yasser
```

![image](https://user-images.githubusercontent.com/120678001/208056422-c14b8d83-4663-4f78-b047-6adad915f116.png)

**Create a principal for the service server**

``` 
$ Kadmin.local
kadmin: add_principal postgres/client2.tekup.tn
```

![image](https://user-images.githubusercontent.com/120678001/208056800-cae1fe43-f3b6-48d9-b270-9276b36d5d49.png)

For an easier configuration of Postgres, I changed the Operating System login name from 'yasser' to 'postgres'.

### Configuration of Kerberos on machine server

Following are the packages that need to be installed on the Service server machine

``` $ sudo apt-get install krb5-user libpam-krb5 libpam-ccreds  ```

![image](https://user-images.githubusercontent.com/120678001/208057005-62c1ddbf-8464-4bf2-8060-00bcb2a6d02a.png)

During the installation, we will be asked for the configuration of the realm, the kerberos server and the administrative server

*PS: We need to enter the same information used for KDC Server*

### Preparation of the keytab file

We need to extract the service principal from KDC principal database to a keytab file

1. In the KDC machine run the following command to generate the keytab file in the current folder: 

``` 
$ Ktutil
ktutil: add_entry -password -p postgres/pg.tekup.tn@TEKUP.TN -k 1 -e aes256-cts-hmac-sha1-96
Password for postgres/pg.tekup.tn@TEKUP.TN:
ktutil: wkt postgres.keytab
```

![23](https://user-images.githubusercontent.com/120678001/208058572-05c82fec-563d-473f-adfe-3bcb04e22bfe.PNG)

2. Send the keytab file from the KDC machine to the Service server machine :

In the Postgres server machine make the following directories :

``` $ mkdir -p /home/postgres/pgsql/data ```

In the KDC machine send the keytab file to the Postgres server :

```$ scp postgres.keytab postgres@PG_SERVER_IP_ADDRESS:/home/postgres/pgsql/data```

*PS: We need to have openssh-server package installed on the service server*


























