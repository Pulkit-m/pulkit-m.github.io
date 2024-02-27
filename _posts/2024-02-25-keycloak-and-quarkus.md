---
layout: post
title: Key-Cloak & Quarkus
subtitle: Outsourcing Authentication & Authorization
# gh-repo: daattali/beautiful-jekyll
# gh-badge: [star, fork, follow]
thumbnail-img: /assets/img/keycloak-thumbnail.png
tags: [Java, Back-End Development, Security]
comments: true
author: Pulkit Mahajan
---

As you would find on Quarkus' website, it is a supersonic and subatomic framework. Well so should be your project. What is it that is required in every project, and is of utmost importance -- Security. Who should be allowed into your application (Authentication), and what should he/she be allowed to see or do (Authorization)? This is where Quarkus Security comes into picture, or you could say Spring Security in case of Spring Framework. Instead of handling all that in our application, we can use KeyCloak to provide authentication and authorization services for our application. Its similar to using OAuth2 with Google. In KeyCloak there are a number of other methods other than OAuth2 with which you can authenticate your application.  

## Setting up the KeyCloak Server 
This part is fairly simply, and there is vast amount of documentation available to configure the keycloak server on your machine, straight forward using the User Interface. To summarize the steps: 
- Start the Keycloak server, and setup the master-userid-password for the first time. 
- Create a new Realm. You can think of a Realm as an organization. So there can be a single keycloak server providing authentication services to multiple organisations. 
- Within the new Realm, Create a new Client. You can think of a Client as an application by the organisation. A Realm can hence have multiple clients. 
    - When you create a new client, its better if you toggle ON the `authentication` toggle. 
    - You can keep the toggle for `authorization` ON or OFF depending upon whether you wish to handle authorisation at your application level or you with keycloak to take care of it. 
- Configure users for the realm. Users are realm-specific. Users in a realm can access all its client applications, unless otherwise restricted(RBAC). 
- Create some roles to assign to the users. Roles can be either realm-based or client-specific. A user can have multiple roles, both Realm-based and client-specific at the same time. 
- You can see all the configuration in detail from this [YouTube](https://www.youtube.com/playlist?list=PLHXvj3cRjbzsVyj6Pxfu4uRE1PtWa2CIw) playlist that I found extremely useful. 

## User Authentication 
To authenticate users, we wanted them to send a `POST` request to our API-Endpoints, containing username and password. We then relay the login request to KeyCloak server. The server responds with a Resposne body consisting of `access_token` (which is the bearer_token) and `refresh_token`, along with some other details. To relay the request is slightly tricky, but not at all difficult, and only requires a few lines of code over a couple files. 

**This is done by using Microprofile Rest Client**.  
These are the exact dependencies that would get you started: 
```xml 
<!--pom.xml-->
<dependency>  
  <groupId>io.quarkus</groupId>  
  <artifactId>quarkus-rest-client</artifactId>  
</dependency>  
<dependency>  
  <groupId>io.quarkus</groupId>  
  <artifactId>quarkus-resteasy-client</artifactId>  
</dependency> 
<dependency>  <!-- and this one is for serialization-->
  <groupId>io.quarkus</groupId>  
  <artifactId>quarkus-rest-client-jackson</artifactId>  
</dependency>  
```

```java 
// Resource file: This is the same as controller if your familiar with Spring framework. 
// LoginResource.java
@Path("/login/token")  
public class LoginResource {  
    @Inject  
    @RestClient KeycloakAuthClient keycloakAuthClient;  

    @ConfigProperty(name = "client.secret") 
    String clientSecret; 

    @ConfigProperty(name = "client.id") 
    String clientId; 
	
    @GET    
    public Response loginRedirect(@RequestBody AuthRequestDTO authRequestDTO){
        AuthResponseDTO authResponse = keycloakAuthClient.login(clientId, clientSecret,
                        "password",authRequestDTO.getusername(), authRequestDTO.getPassword());  
        return Response.ok(authResponse).build();   
    }    
}
```
```java
// Proxy interface to call the rest services. : 
// RemoteServerProxy.java 
@RegisterRestClient(baseUri = "http://localhost:8081")  
@Path("/realms/<RealmName>/protocol/openid-connect")  
public interface RemoteServerProxy {  

    @POST  
    @Path("/token")  
    public AuthResponseDTO login(@FormParam("client_id") String clientId,
                                @FormParam("client_secret") String clientSecret,
                                @FormParam("grant_type") String grantType,
                                @FormParam("username") String username,
                                @FormParam("password") String password)
}
```
```java
// Simple Java POJO for mapping KeyCloak's Response:
// AuthResponseDTO.java
@Getter  
@Setter 
public class AuthResponseDTO {  
    String access_token; 
    Stirng refresh_token; 
    // ... and other fields.  
}
```


 
 This source of information for this article is one of the projects that I have been working on lately, So naturally this page is also under work. I'll keep updating as I learn. Until then, thanks for your patience!!

