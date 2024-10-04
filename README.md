# apim-3scale-demo (work in progress)
Demonstration of full end-to-end API management using Red Hat Integration

### Pre-requistis:
- Access to an OpenShift cluster
- Install 3Scale, Red Hat build of Apicurio Registry, Red Hat Integration - API Designer, and Microcks Operator

## Deploy An Application with APIs on OpenShift 

You can use this guide to deploy this Java Application which expose 2 end points: /loyalty/v1/balance/{id} and /loyalty/v1/transaction/{id} both URL parameter {id} refer to the Loyalty id.

https://github.com/osa-ora/java-ocp-demo

Once deployed you can access the route and make sure the APIs are working prpoerly (you can use any id number).

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

## API Governance using API Registry

Red Hat build of Apicurio Registry is a datastore for sharing standard event schemas and API designs across event-driven and API architectures.

<img width="1263" alt="Screenshot 2024-10-04 at 5 01 32 PM" src="https://github.com/user-attachments/assets/78918549-d40c-4378-9eb2-00f4da1fd60a">

Now, we can import the file that we designed in the previous step, it will automatically takes version 1, if you upload it again it will take version 2.

<img width="1103" alt="Screenshot 2024-10-04 at 5 03 46 PM" src="https://github.com/user-attachments/assets/7a4dab14-041c-4241-bd03-87e03bce754b">

You can enable the validation of the specifications, check the documentation, update another version, generate client SDK and others, we will use the registry as our docuemntation source later in 3Scale.


## API Mocking using Microcks

Open the Microcks route: 

<img width="1470" alt="Screenshot 2024-10-04 at 5 08 56 PM" src="https://github.com/user-attachments/assets/ed6d1de9-73fa-4d36-af3a-f170a01813d9">

Import our file as we did in the previous step, once imported you can see our 2 operations are there, if you expand any, you will see the samples that we defined while designing the APIs are there.

<img width="1495" alt="Screenshot 2024-10-04 at 5 10 19 PM" src="https://github.com/user-attachments/assets/63b83844-58e8-4d54-8078-7dd6ff24c08b">

Expand the operation and copy the URL that we need to use:

<img width="1264" alt="Screenshot 2024-10-04 at 5 11 42 PM" src="https://github.com/user-attachments/assets/6474b88b-14f6-4f7c-a6f4-793e2e956571">

Test it in the browser or by curl command:

<img width="1053" alt="Screenshot 2024-10-04 at 5 12 42 PM" src="https://github.com/user-attachments/assets/eb4ef9a8-e51e-4d83-a95f-1b2be528d810">

Now, we can start building our client that utilizes this API by providing some examples covering different business scenarios .. like account not found, account with zero credit, account locked, account with balance, etc...

## API Management using 3Scale

Now, the final stage is to manage the actual API that we deployed in the first step by the API management and apply differnet policies and security governance.

3Scale define the service as a backend, which can then offer many products (aka APIs), these APIs have differnet mapping as the end points.
You can define activeDocs/documentation for each product/API, also you can define application plans where the client apps need to subscribe in any of these plans to be able to access the API, you can define throttling or pricing configurations per plan.
Finally, you can create an application to test your APIs.

If you configured your API, defined the different policies, you can then deploy it, so the gateway can be your entry point to that API.

- Define the Backend
- Define the Product/API
- Define the ActiveDocs

