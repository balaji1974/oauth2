# oauth2


## OAuth 2.0
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
Used in Server side web applications and mobile native applications
### Client Credentials
Used in Server side scripts with no UI 
### PKCE Enhanced Authorization code
Used in single page Javascript app and mobile native applications
### Device Code
Used in applications that are IoT 
### Refresh Token
Used to exchange a refresh token for an access token




