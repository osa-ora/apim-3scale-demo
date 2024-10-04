# apim-3scale-demo (work in progress)
Demonstration of full end-to-end API management using Red Hat Integration

### Prerequisite:
- Access to an OpenShift cluster
- Install the 4 Operators: 3Scale, Red Hat build of Apicurio Registry, Red Hat Integration - API Designer, and Microcks Operators

---

## Deploy An Application with APIs on OpenShift 

You can use this guide to deploy this Java Application which expose 2 end points: /loyalty/v1/balance/{id} and /loyalty/v1/transaction/{id} both URL parameter {id} refer to the Loyalty id.

https://github.com/osa-ora/java-ocp-demo

Once deployed you can access the route and make sure the APIs are working properly (you can use any id number).
For example:
- {route}/loyalty/v1/balance/1234
- {route}/loyalty/v1/transaction/5555

Get the service url to use it later on while we configure the API management part (http://hostname:port in the following screenshot):

<img width="1184" alt="Screenshot 2024-10-04 at 8 44 45 PM" src="https://github.com/user-attachments/assets/f08b4c90-bc7a-47cd-918c-3048534a5797">

---

## API Design

API-First approach, the first step, of course, is to design the APIs. API Designer is a tool to design your APIs (OpenAPI, AsyncAPI) and schemas (Apache Avro, Google Protobuf, JSON Schema)

<img width="826" alt="Screenshot 2024-10-04 at 4 56 35 PM" src="https://github.com/user-attachments/assets/c0995ea5-d9e9-4f7d-9788-6a7e5f9a6ab0">

Open the route of API Designer (Apicure), you can design the API or import the designed API from the design folder.

<img width="1467" alt="Screenshot 2024-10-04 at 4 44 45 PM" src="https://github.com/user-attachments/assets/93bd2d1d-65bf-4b5e-a5e9-7f2c8893bedb">

The GUI will help you to design the APIs easily, you need to foucs for example on the following items:
- General: API name, version, description, security schema and security requirements.
- Data Types
- Responses (including examples)
- Paths which utilized the previous elements; methods, responses, parameters, data types, and security requirements

If you import the file: design/loyalty-api.json you can see all these elements are already populated.

<img width="1485" alt="Screenshot 2024-10-04 at 4 51 04 PM" src="https://github.com/user-attachments/assets/0dd57191-c339-4151-a15e-102b556da6e6">

Once you finalized the API design you can save the API so you can use in the coming sections.

---

## API Governance using API Registry

Red Hat build of Apicurio Registry is a datastore for sharing standard event schemas and API designs across event-driven and API architectures.

<img width="1263" alt="Screenshot 2024-10-04 at 5 01 32 PM" src="https://github.com/user-attachments/assets/78918549-d40c-4378-9eb2-00f4da1fd60a">

Now, we can import the file that we designed in the previous step, it will automatically takes version 1, if you upload it again it will take version 2.

<img width="1103" alt="Screenshot 2024-10-04 at 5 03 46 PM" src="https://github.com/user-attachments/assets/7a4dab14-041c-4241-bd03-87e03bce754b">

You can enable the validation of the specifications, check the documentation, update another version, generate client SDK and others, we will use the registry as our docuemntation source later in 3Scale.

---

## API Mocking using Microcks

API mocks will enable parallel development streams leading to rapid inner loop development, Open the Microcks route: 

<img width="1470" alt="Screenshot 2024-10-04 at 5 08 56 PM" src="https://github.com/user-attachments/assets/ed6d1de9-73fa-4d36-af3a-f170a01813d9">

Import the same file as we did in the previous step, once imported you can see our 2 operations, if you expand any of them , you will see the samples that we defined while designing these APIs.

<img width="1495" alt="Screenshot 2024-10-04 at 5 10 19 PM" src="https://github.com/user-attachments/assets/63b83844-58e8-4d54-8078-7dd6ff24c08b">

Expand the operation and copy the URL:

<img width="1264" alt="Screenshot 2024-10-04 at 5 11 42 PM" src="https://github.com/user-attachments/assets/6474b88b-14f6-4f7c-a6f4-793e2e956571">

Test it in the browser or by using the "curl" command:

<img width="1053" alt="Screenshot 2024-10-04 at 5 12 42 PM" src="https://github.com/user-attachments/assets/eb4ef9a8-e51e-4d83-a95f-1b2be528d810">

Now, we can start building our client apps that utilizes these API by hitting these mockups, these mockup examples should be covering different business scenarios .. like account not found, account with zero credit, account locked, account with some balance, etc...

---

## API Management using 3Scale

Now, the final stage is to manage the actual API that we deployed in the first step by the API management and apply differnet policies and security governance.

3Scale define the service as a backend, which can then offer many products (aka APIs), these APIs have differnet mapping as the end points.

You can define activeDocs/documentation for each product/API, also you can define application plans where the client apps need to subscribe in any of these plans to be able to access the API, you can define throttling or pricing configurations per plan.

Finally, you can create an application to test your APIs.

If you configured your API, defined the different policies, you can then deploy it, so the gateway can be your entry point to that API.

### Open 3Scale Dashboard using the tenant route

<img width="1163" alt="Screenshot 2024-10-04 at 8 46 47 PM" src="https://github.com/user-attachments/assets/c03e0bc2-ccce-4984-9b15-49dcbc162528">

### Define the API Backend

  We will use the service hostname:port as we mentioned in first step for our Java Application that expose the 2 APIs.

  Click on Create Backend and populate it as following:
  
  ```
  Name: Loyalty Backend
  System name: loyalty-backend
  Private base URL: use the service URL here, for example: http://java-ocp-demo.globex-apim-user1.svc.cluster.local:8080
  ```
  
  <img width="627" alt="Screenshot 2024-10-04 at 8 56 58 PM" src="https://github.com/user-attachments/assets/a6fe7b4c-4d04-4a49-8da8-351825674e3b">

  Then Add 1 mapping rule, where we map the root (i.e. /) : 

  <img width="259" alt="Screenshot 2024-10-04 at 8 52 03 PM" src="https://github.com/user-attachments/assets/134c4d93-8bfb-4cea-8fba-c10ac40cb2e8">

  Or you can create the backend by more GitOps methodology by creating the yaml file in 3scale/backend.yaml
  You need just to update 2 lines in that file that correspond to the tenant access token and the Java app service url
  
  ```
  privateBaseURL: "http://java-ocp-demo.globex-apim-user1.svc.cluster.local:8080"
  providerAccountRef:
    name: 3scale-tenant-secret
  ```

  how to get the access token ?
  Go to 3Scale Tenant, and create a secret "3scale-tenant-secret" on the namespace you are using with the same details.
  
  <img width="1190" alt="Screenshot 2024-10-04 at 9 06 25 PM" src="https://github.com/user-attachments/assets/3f4b3646-2731-48f0-8aa8-980c447d39be">

  Open the tenant, and go to its resources:
  
  <img width="502" alt="Screenshot 2024-10-04 at 9 08 18 PM" src="https://github.com/user-attachments/assets/f87cbddd-4d02-452d-b050-2bfb4ad6d16e">

  
### Define the Product/API

  We can also create it from the GUI or again by GitOps approach, from the GUI we need to fill much more details:
  
  <img width="467" alt="Screenshot 2024-10-04 at 9 01 26 PM" src="https://github.com/user-attachments/assets/6f5d9599-d3c0-4bdd-81ba-3b07da9454e1">
  
  Or use the GitOps approach and create the file 3scale/product-api.yaml
  It will also create the mapping rule: 

  <img width="1477" alt="Screenshot 2024-10-04 at 9 23 36 PM" src="https://github.com/user-attachments/assets/9a97777f-6057-4948-bc3f-ef4d34b03e02">

  It will also create the application plans:
  
  <img width="1475" alt="Screenshot 2024-10-04 at 9 24 02 PM" src="https://github.com/user-attachments/assets/28feb4db-026b-4aad-8c8b-895c7d78810c">


  
### Define the API ActiveDocs/Documentations
  Again we can create it from the GUI, copy and paste the documentation

  <img width="713" alt="Screenshot 2024-10-04 at 9 22 20 PM" src="https://github.com/user-attachments/assets/132f93dc-8951-40df-a44b-b3bdebae1306">

  or better use the GitOps approach as per the file: 3scale/active-docs.yaml
  But you need to change one line related the active docs location in the Apicure Registry:
  Get the Route URL of the Registry fron OpenShift Console:
  
  <img width="747" alt="Screenshot 2024-10-04 at 9 18 49 PM" src="https://github.com/user-attachments/assets/f6e4a19b-255a-4d3c-b593-b6a24fb731c8">

  
   ```
      url: "https://service-registry-user1.apps.cluster-5p2pn.5p2pn.sandbox1568.opentlc.com/apis/registry/v2/groups/osaora/artifacts/loyalty-service-apis"
      with the following: 
      url: "{ROUTE_UEL}/apis/registry/v2/groups/osaora/artifacts/loyalty-service-apis"
   ```
   
  <img width="1402" alt="Screenshot 2024-10-04 at 9 20 50 PM" src="https://github.com/user-attachments/assets/1d4608ac-edce-4fd7-8026-a396c1ba690e">

 ### Add API Policies:
   
 Now, we can add some policies to the APIs, for example let's add retry policy:
 
 <img width="1081" alt="Screenshot 2024-10-04 at 9 26 49 PM" src="https://github.com/user-attachments/assets/c391be12-f510-43eb-8c66-7bd15ae4af21">

 Click on add policy and navigate the different policies and pick the rety policy, once added, edit it and add the number of retries to 3: 
 
 <img width="454" alt="Screenshot 2024-10-04 at 9 28 03 PM" src="https://github.com/user-attachments/assets/ccc9e2ad-f3e0-4565-b0ef-4a6e42484744">

### Add Rate Limits:

 Configure Rate Limits, go to the basic plan and click on New Usage Limit:
 
 <img width="1244" alt="Screenshot 2024-10-04 at 9 30 09 PM" src="https://github.com/user-attachments/assets/e0a71579-01db-425d-a58f-c3f44c998765">
 
 Add 3 limit per minute and may be 10 per hour for testing: 
 <img width="387" alt="Screenshot 2024-10-04 at 9 31 09 PM" src="https://github.com/user-attachments/assets/2f066b39-aafa-418f-a32f-b3edf5e3bd0d">

 <img width="1258" alt="Screenshot 2024-10-04 at 9 31 58 PM" src="https://github.com/user-attachments/assets/f72d0fc8-fc6e-4b71-8f35-07d6e06056ca">

### Deplopy our APIs

Go to the Configurations section and click on promote changes.

<img width="1220" alt="Screenshot 2024-10-04 at 9 35 41 PM" src="https://github.com/user-attachments/assets/23a402a7-ffaa-4116-9724-eccb2fb062b4">

### Create Client Application

Go to applications listing and click on create application

<img width="436" alt="Screenshot 2024-10-04 at 9 45 45 PM" src="https://github.com/user-attachments/assets/d377f3b7-3b40-45c5-b9cc-550776156929">

The create application will generate also user_key that we can use to access the APIs.

<img width="1264" alt="Screenshot 2024-10-04 at 9 46 33 PM" src="https://github.com/user-attachments/assets/ac210ffa-3ee0-454e-93c9-bb1fc1330254">

### Test our APIs

Get the staging URL and replace the {id} with a valid number e.g. 12345 and add it to the browser without the user_key, you will get error: "Authentication parameters missing"

<img width="1104" alt="Screenshot 2024-10-04 at 9 40 28 PM" src="https://github.com/user-attachments/assets/cb28b2f4-9e61-4241-92c6-f056d4d04564">

Now add the user_key and test it:

<img width="1157" alt="Screenshot 2024-10-04 at 9 42 07 PM" src="https://github.com/user-attachments/assets/b060aae8-e6e2-43c3-ae78-51ecb0f02963">

Refresh the page 4-5 times.

You will get the error message: "Usage limit exceeded"

<img width="1159" alt="Screenshot 2024-10-04 at 9 42 59 PM" src="https://github.com/user-attachments/assets/68fff69e-1475-4c67-9c52-3732c08b5d59">

Wait for a minute, test again and refresh till we hit the hour limit.

We can switch our client to use the premium plan which doesn't have any rate limit and test it again and we will not hit any rate limits.

<img width="1255" alt="Screenshot 2024-10-04 at 9 48 53 PM" src="https://github.com/user-attachments/assets/b78700be-34b0-4898-ac19-73a9da86788f">

You can also try to regenerate the key or suspend/resume the client and note that it takes times to be reflected as the gateway will get the updates frequently from the management application.

This conclude our demo which covers end to end scenarios of designing APIs, Governance, Mocking it, Managing it and connecting it to the actual backend with different governance policies and rate limits.


 
