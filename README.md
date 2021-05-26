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


