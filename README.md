# Gerrit_on_docker_SAML_Kecloak
Documentation when explan as configure a gerrit run on docker and this making SAML authentication on Keycloak.  

## Envorioment
In this envoriorment we'll have 2 VMs, both with docker installed. 

How to this a enviornment test, I deployed the Keycoak in dev mode. 
VM Keycloak: 

`docker run -p 8080:8080 -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=admin quay.io/keycloak/keycloak:25.0.4 start-dev`

