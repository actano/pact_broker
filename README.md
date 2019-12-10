# Pact Broker

* [Official Doc](https://github.com/pact-foundation/pact_broker)

## How to access the pact broker ui

* [Pact Broker UI](http://pact-broker.rplan.com/)

The UI is protected by basic auth, you can get the credentials using the command bellow:  

```bash
vault read secret/rplanx/gke-dev/pact-broker
```

## How to setup a pact communication (Consumer -> Provider)

### Consumer

- (1) First thing you have to do is defining what you need from the provider (pact definition) [Example](https://github.com/actano/allex-accounts/blob/5e16b38a7cd646ee75e2ba7a5b18d5214f717051/test/pact/authentication-service-pact.js)
- (2) Run your api tests against the pact definition (it will generate a pact file) [Example](https://github.com/actano/allex-accounts/blob/1f13f3e33697a03c464fda65c785e7e53a561f89/test/rest-endpoints/user-login/post-user-login.spec.js) 
- (3) Publish your pact file to the pact broker [Example](https://github.com/actano/allex-accounts/blob/6442b8c2fe13ffe4343345cb67a8039f27dddb5a/Jenkinsfile-k8s#L55-L72)
- (4) Create a webhook between consumer -> provider [Example](https://github.com/actano/allex-accounts/blob/6442b8c2fe13ffe4343345cb67a8039f27dddb5a/Jenkinsfile-k8s#L73-L92)

### Provider

- (5) Add verify pacts test file [Example](https://github.com/actano/rplan-authentication/blob/ef6e1028d7c7c52d1e22c3265811a766aa56baca/test/pact/verify-pact-with-consumers.js)
- (6) Add "Verify Pacts" in the jenkins pipeline [Example](https://github.com/actano/rplan-authentication/blob/525019163d099a205027131b0df7efc652826acf/Jenkinsfile-k8s#L44-L64)

### Back to consumer

- (7) Add "Can I Deploy" check in the jenkins pipeline [Example](https://github.com/actano/allex-accounts/blob/6442b8c2fe13ffe4343345cb67a8039f27dddb5a/Jenkinsfile-k8s#L119-L136)

That's all!!! Your pact broker communication should work! 

### How the build/ci pipeline flow works

- Consumer changes the pact definition and pushes 
- Jenkins will kick off the consumer build 
- Consumer build will publish the new pact definition to pact broker
    - Pact broker will kick off a provider build
    - Provider build will verify pacts
- While the provider build is running, the consumer build will check periodically if it's possible to deploy via canIDeploy (timeout can be specified)          
    - After checked with success
        - The canIDeploy thread in the consumer will be green
        - The consumer will be deployed
    - If periodic checks finally failed (timeout is exceeded)
        - The provider and after that the consumer needs to be re-built
 
