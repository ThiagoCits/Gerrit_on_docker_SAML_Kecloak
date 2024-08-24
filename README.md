# Gerrit_on_docker_SAML_Kecloak
Documentation when explan as configure a gerrit run on docker and this making SAML authentication on Keycloak.  

## Envorioment
In this envoriorment we'll have 2 VMs, both with docker installed. 

How to make this  enviornment test, I deployed the Keycoak in dev mode. 

### VM Keycloak 

`docker run -p 8080:8080 -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=admin quay.io/keycloak/keycloak:25.0.4 start-dev`.

Access the Keycloak with the credentials that are configure in the command abolve, in this case, user= Admin and the password= Admin.

Select the Realm, in the left bar, click on bar top and select the realm you use for to authenticate the gerrit. I'll use the Master Realm. 

We'll need to import a realm with the REALM configuration.

The file REAML its here: https://github.com/ThiagoCits/Gerrit_on_docker_SAML_Kecloak/blob/main/Keycloak-Gerrti-SAML-realm-export.json

Import this file to the Keycloak. Access the Keycloak, in the left bar, click in the REALM Settings. There is in the up side of page a box with information "Action". Click there, you'll find the option "Partial Import". 

A observation: In the file Gerrit_on_docker_SAML_keycloak, change the information about de gerrit_ip for the ip of your Gerrit server. 

We'll need to create a user that will to authenticate on Gerrit aplication, for this on Keycloak, in the left bar, click in `USERS`, after `Add User`.

In the GENERAL fild, fill the username, in this case a I will user the username `jdoe`.

Fill the fild email, I'll put `john.doe@org.com`.

In the First name, I'll user `John` and  in the Last Name, a I'll fill with `Doe`.

After create the user, We'll need to add the passoword to it.
Return the Option USERS, and click on the user that you created.
Select tab Credentials, and click at Set Passowrd.

Now, We'll to configurare a Gerrit on docker in the other VM. 

### VM Gerrit.
On the  VM with the docker installed, We'll use the docker-compose file to create a Gerrit container.
You can to get the docker-compose file in this link: https://github.com/ThiagoCits/Gerrit_on_docker_SAML_Kecloak/blob/main/docker-compose.yml
##### Observation: 
In this code, you don't forget the change the part enviroment: CANONICAL_WEB_URL, put the ip of VM. 

Make the download to the vm of docker-compose file. 

`wget https://github.com/ThiagoCits/Gerrit_on_docker_SAML_Kecloak/blob/main/docker-compose.yml`.

After change the docker-complese file, run the command below to deploy the Gerrit. 

`docker compose up -d`

For you verify that the container is run, make the command below. 

`docker ps`.

Probably you'll see same look like this:

##### Container_ID   gerritcodereview/gerrit   "/entrypoint.sh"   2 days ago   Up 43 hours   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp, 0.0.0.0:29418->29418/tcp, :::29418->29418/tcp   name_vm-gerrit-1

We see that the name of VM it is name_vm-gerrit-1.

With the containers up, we'll need to make a download of saml plugin, create the certificades and configure the gerrit.config file. 

First, we go to make a download salm plugin.

`wget https://gerrit-ci.gerritforge.com/job/plugin-saml-bazel-master/lastSuccessfulBuild/artifact/bazel-bin/plugins/saml/saml.jar`.

Take this file and import to the Gerrit container. We'll need to import to directories /var/gerrit/lib and /var/gerrit/plugins. 

`docker cp ./saml.jar name_vm-gerrit1:/var/gerrit/lib`

`docker cp ./saml.jar name_vm-gerrit1:/var/gerrit/plugins`

Access the docker container and generete the keystore in /var/gerrit/etc.

`docker exec -it name_vm-gerrit-1 bash`

`cd /var/gerrit/etc`

`keytool -genkeypair -alias pac4j -keypass pac4j-demo-password -keystore samlKeystore.jks -storepass pac4j-demo-password -keyalg RSA -keysize 2048 -validity 365`

Will be required to fill some informations for create the certificate.

Created the certificate, it name will be samlKeystore.jks

Exit the container.

`exit`.

Access the docker volume to configure the gerrit.config file. 

`cd /var/lib/docker/volumes/name_vm_etc-volume/_data/`

Edit the gerrit.config file.

`nano gerrit.config`

Add the information below in que gerrit.config file. 

```
[gerrit]
        installModule = com.googlesource.gerrit.plugins.saml.Module
[auth]
        type = HTTP
        loginUrl = http://<vm_ip>:8080/login
        logoutUrl = http://<vm_ip>:8080/
        httpHeader = X-SAML-UserName
        httpEmailHeader = X-SAML-EmailHeader
        httpExternalIdHeader = X-SAML-ExternalId
[httpd]
        listenUrl = http://*:8080/
        filterClass = com.googlesource.gerrit.plugins.saml.SamlWebFilter
[saml]
        serviceProviderEntityId = SAML2Client
        keystorePath = /var/gerrit/etc/samlKeystore.jks
        keystorePassword = pac4j-demo-password
        privateKeyPassword = pac4j-demo-password
        metadataPath = http://<keycloak_ip>:8080/realms/master/protocol/saml/descriptor
        userNameAttr = UserName
        displayNameAttr = DisplayName
        emailAddressAttr = EmailAddress
        computedDisplayName = true
        firstNameAttr = firstName
        lastNameAttr = lastName

```

Don't forget to change the informations that have been flagged in fields auth and saml. 

Restart the Gerrit container. 

`docker restart name_vm-gerrit-1`

After the gerrit start , access the application through browser. 

When you click in sing in, you will redirect for keycloak. Put the credentials of the user that you create on keycloak. Ready, you authenticated and can to start to use the Gerrit. 


