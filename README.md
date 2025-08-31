# OAuth 2.0
It is an authroization framework 
Open Authorization (It is Federated (single place) Authentication and Delegated (gives delegation) Authorization) 

## Roles 
Resource Owner     
Resource Server     
Client Application    
Authorization Server     

## OAuth flow:   
Client Application (confidential and public clients) needs to register to the authorization server to get a client id and secret key. After successful registration the client will be provided with an access token to access the resource server. The resource server will validate this key with the authorization server before granting access to the client application.  Confidential clients can guarantee the safety of the secret key while public clients cannot guarantee this.    

## Access Token Types
### Identifier type 
This is used to retrieve the actual authorization information    
### Self contained authorization information
This is json information containing 3 parts namely the header section, payload section and signature and it is base64 encoded    

## Open ID Connect
This is a layer over the authorization server which provides an ID token to the client which contains information of the currently authenticated user

## Grant Types
It is a way an application gets an access token. They are as follows: 
### Authorization Code
Used in Server side web applications and mobile native applications. Requirements are that the client application must be capable of handling a re-direction flow. Client applications must also be capable of securely storing the secret key. 
### Client Credentials
Used in Server side scripts with no UI. Machine to machine interaction with no human intervention.
### PKCE Enhanced Authorization code
Used in single page Javascript app and mobile native applications. Since secret keys cannot be kept safely the client application generates its own key and sends it to the authorization server requesting for the access token.
### Password Grant Type
Must only be used on applications that do not support redirect  
### Device Code
Used in applications that are IoT 
### Refresh Token
Used to exchange a refresh token for an access token. This is used by the client application to request for a new access token for an expired access token and we will get back a new access token and in some cases a new refresh token. To get a refresh token that never expires send the http body parameter as scope=offline_access. 

## Keycloak server   
Starting a key clock server - ./standalone.sh


## Keycloak on docker
```xml
cd backend-keycloak-auth
Run the following command: 
docker-compose up

Lets look at the docker-compose file
name: keycloak-server
services:
  db:
    image: postgres:13.5
    environment:
      POSTGRES_HOST_AUTH_METHOD: trust
      POSTGRES_USER: my_admin
      POSTGRES_PASSWORD: my_password
      POSTGRES_DB: keycloak_db
    ports:
      - 5432:5432
  kc:
    image: quay.io/keycloak/keycloak:legacy
    environment:
      DB_VENDOR: POSTGRES
      DB_ADDR: db
      DB_DATABASE: keycloak_db
      DB_USER: my_admin
      DB_SCHEMA: public
      DB_PASSWORD: my_password
      KEYCLOAK_HOSTNAME: localhost
      KEYCLOAK_USER: kc_admin
      KEYCLOAK_PASSWORD: kc_password
    ports:
      - 8080:8080
    depends_on:
      - db

This will start the postgres database and act as a backend 
database server for our keycloak instance
It will also start a keycloak server and be connected to the 
postgres database at the backend

Finally run the following command:
docker-compose down

```

## Working with Keycloak 
```xml
cd backend-keycloak-auth
Run the following command: 
docker-compose up

Login to the admin console of keycloak:
http://localhost:8080/auth
Click on Admin Console
Enter user id and password:
kc_admin
kc_password

First thing that you will see is a Master Realm. 
Realm is something similar to a tenant. 
We will create our own realm called My_Realm
This will also move us to this this newly created realm

Now go to My_Realm -> Realm Settings -> EndPoints -> 
OpenID Endpoint configurations (click)
This will open a new tab on the browser
Copy the URL on this browser tab. Eg. in my case it is:
http://localhost:8080/auth/realms/My_Realm/.well-known/openid-configuration

Next create a user account:
Go to My_Realm -> Users -> Create New User 
Username: my_user
Firstname: Balaji
Lastname: Thiagarajan
Create (click)

Next set user password:
Go to Password (tab) -> Set Password (click)
Password: my_password
Confirm Password: my_password
Save (click)
Are you sure? Yes

Next create the scope needed by the client to access the resource server:
With My_Realm -> Client Scope -> Create Client Scope (Click)
Name: message.read
Type: Default
Save (Click)

With My_Realm -> Client Scope -> Create Client Scope (Click)
Name: message.write
Type: Default
Save (Click)

Next create the client:
With My_Realm -> Clients -> Create Client (Click)
Client ID: my_client
Name: My Client
Save (Click)
Next (Click)
Client authentication: On
Authorization: On
Authentication flow: OAuth 2.0 Device Authorization Grant(tick)
Save (Click)
Valid redirect URIs: 
http://localhost:8083/login/oauth2/code/gateway (add one more)
http://localhost:8083/login/authorized
Login settings -> Login Theme -> Select on of the login forms to display (eg. keycloak)
Save (click)

Next add scopes:
Go to Client Scope Tab -> Search for message
message.read (tick)
message.write(tick)

Finally Go to credentials Tab -> 
Copy 'client secret' key to be used on the client side.  


Next we have 2 spring boot applications:
Spring Cloud Gateway Server -> Client Server (gateway-client)
Spring Boot Application behind protected behind keycloak -> Resource Server (my-resource-server)

Gateway Client (gateway-client):
--------------------------------
application.properties
server.port=8083
spring.application.name=gateway-client

spring.cloud.gateway.server.webflux.routes[0].id=messages_route
spring.cloud.gateway.server.webflux.routes[0].uri=http://localhost:8082/messages
spring.cloud.gateway.server.webflux.routes[0].predicates[0]=Path=/messages/**
spring.cloud.gateway.server.webflux.routes[0].filters=TokenRelay=

spring.security.oauth2.client.registration.gateway.provider=my-provider
spring.security.oauth2.client.registration.gateway.client-id=my_client
spring.security.oauth2.client.registration.gateway.client-secret=xH7a9aEyM3LuJiLJoUVTmenWOOuTQBaH
spring.security.oauth2.client.registration.gateway.authorization-grant-type=authorization_code
spring.security.oauth2.client.registration.gateway.redirect-uri=http://localhost:8083/login/oauth2/code/{registrationId}
spring.security.oauth2.client.registration.gateway.scope=openid, message.read
spring.security.oauth2.client.provider.my-provider.issuer-uri=http://localhost:8080/auth/realms/My_Realm


pom.xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>

<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-gateway-server-webflux</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-devtools</artifactId>
	<scope>runtime</scope>
	<optional>true</optional>
</dependency>


Resource Server (my-resource-server):
-------------------------------------

pom.xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-security</artifactId>
</dependency>

<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-devtools</artifactId>
	<scope>runtime</scope>
	<optional>true</optional>
</dependency>


application.properties
server.port=8082
spring.application.name=my-resource-server
spring.security.oauth2.resourceserver.jwt.issuer-uri=http://localhost:8080/auth/realms/My_Realm

SecurityConfig.java
@EnableWebSecurity
public class SecurityConfig {

    /**
     * For the backend-resources, I indicate that all the endpoints are protected.
     * To request any endpoint, the OAuth2 protocol is necessary, using the server configured and with the given scope.
     * Thus, a JWT will be used to communicate between the backend-resources and backend-auth when backend-resources
     * needs to validate the authentication of a request.
     */
	
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http.authorizeHttpRequests(authorize -> authorize
                .requestMatchers("/**").hasAuthority("SCOPE_message.read"))
        		.oauth2ResourceServer(oauth2ResourceServer ->
        			oauth2ResourceServer.jwt(Customizer.withDefaults()) // Or .opaqueToken(Customizer.withDefaults())
        		);
        
        return http.build();
    }
	
}

ResourceController.java
@RestController
public class ResourceController {
    @GetMapping("/messages")
    public String getMessages() {
        return "You are now accessing the protected resource";
    }
}

Next start both the gateway and resource servers.
Finally access the resource using url:
http://localhost:8083/messages

You will be redirected to the Keycloak login page where you need 
to enter the user id and password:
User: my_user
Password: my_password

You will now be authenticated using Keycloak server and 
redirected to the resource and you will see the successfull
message as below:
You are now accessing the protected resource

```


### References
```xml
https://www.youtube.com/watch?v=YHWfJHKGYGI

```





