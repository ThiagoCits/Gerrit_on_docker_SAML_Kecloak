# Gerrit_on_docker_SAML_Kecloak
Documentation when explan as configure a gerrit run on docker and this making SAML authentication on Keycloak.  

## Envorioment
In this envoriorment we'll have 2 VMs, both with docker installed. 

How to this a enviornment test, I deployed the Keycoak in dev mode. 

### VM Keycloak 

`docker run -p 8080:8080 -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=admin quay.io/keycloak/keycloak:25.0.4 start-dev`.

Access the Keycloak with the credentials that be configure in the command abolve, in this case, user= Admin and the password= Admin.

Select the Realm, in the left bar, click on bar top and select the realm you use for to authenticate the gerrit. I'll use the Master Realm. 

We'll need to import a realm with the REALM configuration.

The file REAML its here: https://github.com/ThiagoCits/Gerrit_on_docker_SAML_Kecloak/blob/main/Keycloak-Gerrti-SAML-realm-export.json

Import this file to the Keycloak. Access the Keycloak, in the left bar, click in the REALM Settings. There is in the up side of page a box with information "Action". Click there, you'll find the option "Partial Import". 

A observation: In the file Gerrit_on_docker_SAML_keycloak, change the information about de gerrit_ip for the ip of your Gerrit server. 

We'll need to create a user that will to authenticate on Gerrit aplication, for this on Keycloak, in the left bar, click in USERS, after Add User.

In the GENERAL fild, fill the username, in this case a I will user the username "jdoe"
Fill the fild email, I'll put "john.doe@org.com".
In the First name, I'll user John and  in the Last Name, a I'll fill with "Doe".

After create the user, We'll need to add the passoword to it.
Return the Option USERS, and click on the user that you created.
Select tab Credentials, and click at Set Passowrd.

Now, We'll to configurarem a Gerrit on docker in the other VM. 

### VM Gerrit.
On the  VM with the docker installed, We'll use the docker-compose file to create a Gerrit container.


