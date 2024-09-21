# BAMOE 9 Enablement - Lab

This project details the lab for the BAMOE 9 enablement.


## Important

Please keep in mind that BAMOE 9.1 is Tech Preview for stateful processes use cases and is not meant for Production. 
Therefore, some features and functionalities are either non provided or still being tested.
The 9.2 release scheduled for Q4-2024 will be the first supported version of BAMOE 9 powered by Kogito.

Note that 9.1 is supported for stateless business processes i.e. which do not require persistence.


## Objectives

In this lab, we will be using the VM already set up for BAMOE 9.1 usage in TechZone.
We will be starting all services and deploy a sample project which contains a business process.
This project is part of the BAMOE 9.1 distribution samples and includes a *docker-compose* configuration aiming at starting all needed services.


## Steps

### TechZone

Connect to the BAMOE 9.1 TechZone VM: either using Microsoft Remote Desktop or within a Chrome browser.
<TODO>XXX.services.cloud.techzone.ibm.com:XXX is the URL, replacing witht the port assigned to you.
The VM are Windows-based and the credentials to login are: user: `techzone`, password: `IBMDem0s!`


### Detail of the set up

The VM has already been set up for a BAMOE 9.1 usage, including:

- JDK 17
- A local Maven repository for the BAMOE 9.1 distribution, located in <TODO>
- Docker CLI
- A sample of the compact architecture which we will be using as a lab
- A script to run a docker-compose YAML file


### Sample project

The project that we will be using is stored in the `C:/kogito-examples/kogito-quarkus-examples/jbpm-compact-architecture-example/` folder.
In this project, a business process (defined as BPMN) and business rules (defined as DMN) can be found under the `src/resources` folder.

The business process is a basic one detail a hiring process.
The hiring process aims at identifying if a candidate is a good fit, following the given workflow:

![Hiring Process](images/HiringProcessBPMN.jpg)

The data model used by the business process is the following one. It includes 1 input object (*CandidateData*) and 1 output object (*Offer*).

![Hiring Data Model](images/HiringDataModel.jpg)

In the process, an automatic decision is made to compute the offer and is using business rules implemented as DMN.

- DMN model:

![Compute Offer DRG](images/ComputeOfferDMNDRG.jpg)

- DMN Decision Table:

![Compute Offer Decision Table](images/ComputeOfferDMNDT.jpg)

It also includes 2 human tasks corresponding to a human resource interviewer followed by an IT interviewer, both defined as human tasks.
A few other tasks are defined, mostly script tasks for logging purposes.

For more information about the project, please refer to the README.

BPMN specification and BAMOE includes many more types of tasks, but we will only be using a few of them for this lab.


### Start

On the desktop, the `start-compact.bat` batch file needs to be run. 
It starts all following services needed by the project that we will be using:

- PostgreSQL database service (port 5432) - this databse is needed for this sample because the project contains a stateful business process which requires persistence
- PgAdmin (port 8055) - the UI to management the database
- Keycloak (port 8480) - software to handle security
- Kogito example service (port 8080)
- Management Console (port 8280) - UI to manage the process instances
- Task Console (port: 8380) - UI to manage the task inbox

Please note that the docker-compose is useful for testing purposes but in Production clients would most likely not use this way to deploy.
The projects containing the business logic (i.e. business processes and business rules) are to be built and deployed on Quarkus, which is actually done for you using the `docker-compose` file.


### Steps

- In a new Chrome tab, open PGAdmin
    - URL: http://localhost:8055
    - Check out the BAMOE 9 tables by expanding `Servers > kogito > Databases > kogito > Schemas > public > Tables`
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
    - Using Swagger and the `/hiring` **POST** endpoint, create a new process instance by clicking on `Try it out` and using the following input data (Note that instead of using Swagger, you could use the `cURL` command line interface):
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

If the operation is successful, you should get a result similar to the following, depending on your input data:
```json
{
  "id": "a1b52e1d-a9c8-4478-a17c-05427cba777e",
  "offer": {
    "category": "Senior Software Engineer",
    "salary": 40450
  }
}
```

This `id` corresponds to the process instance ID which need to be used for later interaction with the process instance.


- In the management console, refresh the list of process. The newly created one should now show. 

![Process Instance List](images/ProcessInstancesList.jpg)

If you had created multiple process instances, they would all show on that console.

If you now click on the process instance ID that you just created, you will get the details.

![Process Instance Details](images/ProcessInstanceDetails.jpg)

<TODO>

- In the Task Console, refresh the list of tasks. You should now see the task that needs to be completed first, which corresponds to the Human Resource interviewer task.
Please note that there is a boundary timer on this task set to 180 seconds, so if you do not complete this task within 3mn, it will move to the next step of the process.  In such a case, you will need to recreate a new process instance as you did before.

![Task Inbox](images/TaskInboxList.jpg)





## Use the Canvas

URL: http://localhost:9090
<TODO: we will not use the Canvas for the lab, but since it is installed in the VM, we will play with it>
<TODO: load the following BPMN file stored in XXXX>
<TODO: instructions to load>

