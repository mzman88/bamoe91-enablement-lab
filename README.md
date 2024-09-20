# BAMOE 9 Enablement - Lab

This project contains the lab for the BAMOE 9 enablement.

## Objectives

In this lab, we will be using the VM already set up for BAMOE 9.1 usage.
We will be starting all services and deploy a sample project which contains a business process.
This project is part of the BAMOE 9.1 distribution samples and includes a docker-compose configuration aiming at starting all needed services.


## Steps

### TechZone

Connect to the BAMOE 9.1 TechZone VM: 

<TODO: URL to connect>

### Detail of the set up

<TODO: This VM is using Windows as Operating System>
<TODO: the Maven repository has already been set up in the XXX folder which contains the BAMOE 9 distribution>


### Sample project

The project that we will be using is stored in the `C:/kogito-examples/kogito-quarkus-examples/jbpm-compact-architecture-example/` folder.

The business process is a basic one detail a hiring process.

![Hiring Process](images/HiringProcessBPMN.jpg)

In the process, an automatic decision is made to compute the offer and is using business rules implemented as DMN:

![Compute Offer DRG](images/ComputeOfferDMNDRG.jpg)

![Compute Offer Decision Table](images/ComputeOfferDMNDT.jpg)


### Start

On the desktop, the `start-compact.bat` batch file needs to be run. 
It starts all following services needed by the project that we will be using:

- PostgreSQL database service (port 5432) - this databse is needed for this sample because the project contains a stateful business process which requires persistence
- PgAdmin (port 8055) - the UI to management the database
- Keycloak (port 8480) - software to handle security
- Kogito example service (port 8080)
- Management Console (port 8280) - UI to manage the process instances
- Task Console (port: 8380) - UI to manage the task inbox


### Steps

- In a new Chrome tab, open PGAdmin
    - URL: http://localhost:8055
    - Check out the tables by expanding `Servers > kogito > Databases > kogito > Schemas > public > Tables`
- In a new Chrome tab, open the Management Console
    - URL: http://localhost:8280
    - Login using the following credentials: user: jdoe, password: jdoe.  This user and his business roles have been added at startup in the keycloak configuration
    - Note that there are no process instance visible yet
- In a new Chrome tab, open the Task Console
    - URL: http://localhost:8380
    - You should not need to login again as the SSO is taken care of by the keycloak component
    - Note that there are no human task assign to this user
- In a new Chrome tab, open the Swagger UI 
    - URL: http://localhost:8080/q/swagger-ui
    - Note that the REST endpoints are specific to the business process and have been automatically generated
- Start a process instance
    - Using Swagger and the `/hiring` **POST** endpoint, create a new process instance by clicking on `Try it out` and using the following input data (Note that instead of using Swagger, you could use the cURL command line interface):
```json
{
  "candidateData": {
    "name": "Jon",
    "lastName": "Snow",
    "email": "jon@snow.org",
    "experience": 5,
    "skills": ["Java", "Kogito", "Fencing"]
  }
}
```

- In the management console, refresh the list of process. The newly created one should now show.





## Use the Canvas

URL: http://localhost:9090
<TODO: we will not use the Canvas for the lab, but since it is installed in the VM, we will play with it>
<TODO: load the following BPMN file stored in XXXX>
<TODO: instructions to load>

