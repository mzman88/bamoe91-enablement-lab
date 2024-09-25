# BAMOE 9 Enablement - Lab

This project provide details and instructions for the lab for the BAMOE 9 enablement.

Note that this lab does not intend to teach what BPMN or DMN specifications are, nor to teach how to design business processes or business rules. Its objectives is to show what features and tooling are featured as part of BAMOE 9.1 release, and to get some practice on a specific simple use case.


## Important

Please keep in mind that BAMOE 9.1 release is **Tech Preview** for stateful processes use cases and is not meant for Production environment. 
Therefore, some features and functionalities are either non provided or still being implemented and tested.
The 9.2 release which is scheduled for Q4-2024 will be the first supported PAMOE version of BAMOE 9 powered by Kogito.

Note that DMOE 9.1 is already supported for stateless business processes i.e. use cases which do not require persistence.


## Objectives

In this lab, we will be using the Virtual Machine already set up for BAMOE 9.1 usage in IBM Technology Zone.
We will be starting all services and deploy a sample project which contains a business process.
This project is part of the BAMOE 9.1 distribution samples and includes a *docker-compose* configuration aiming at starting all needed services.


## Steps

### Technology Zone

Connect to the BAMOE 9.1 Technology Zone Virtual Machine, either using Microsoft Remote Desktop or within a Chrome browser.
`useast.services.cloud.techzone.ibm.com:<your_assigned_port_number>` is the URL, replacing the port number with the one assigned to you.
The VMs are Windows-based and the credentials to login are: user: `techzone`, password: `IBMDem0s!`


### Detail of the set up

Your VM has already been set up for a BAMOE 9.1 usage, including:

- JDK 17
- Visual Studio Code & BAMOE editors
- Maven
- GIT client
- A local Maven repository for the BAMOE 9.1 distribution, located in <TODO>
- Podman
- A sample of the compact architecture which we will be using as a lab
- A script to run a *docker-compose* file in charge of starting all services and tools 
- Canvas


### Sample project

- We will take a look at the project that we will benusing for this lab. 
The project that is located in the `C:\kogito-examples\kogito-quarkus-examples\jbpm-compact-architecture-example` folder.

- We will use VSCode to edit all projects artifacts.  In a Windows Command Prompt (type `cmd` in the search bar), navigate to the `C:\kogito-examples\kogito-quarkus-examples\jbpm-compact-architecture-example\` folder, and type `code .`.  This will open an instance of VSCode and load the project.  You could also load the project from VSCode using the "*File > Open Folder...*" option.   Make sure you are not seeing the project that opens by default as it is not the right project!

- In this project, a business process (defined as BPMN) and business rules (defined as DMN) can be found under the `src\main\resources` folder.
Open both `src\main\resources\hiring.bpmn` and `src\main\resources\NewHiringOffer.dmn` file in VSCode and get familiar with them.

- The business process is a basic one detail a hiring process.
The hiring process aims at identifying if a candidate is a good fit, following the given workflow:
![Hiring Process](images/HiringProcessBPMN.jpg)

- The data model used by the business process is the following one. It includes 1 input object (*CandidateData*) and 1 output object (*Offer*).
![Hiring Data Model](images/HiringDataModel.jpg)

- In the process, an automatic decision is made to compute the offer and is using business rules implemented as DMN.

    - DMN model:

    ![Compute Offer DRG](images/ComputeOfferDMNDRG.jpg)

    - DMN Decision Table:

    ![Compute Offer Decision Table](images/ComputeOfferDMNDT.jpg)

- It also includes 2 human tasks corresponding to a human resource interviewer followed by an IT interviewer, both defined as human tasks.
A few other tasks are defined, mostly script tasks for logging purposes.

For more information about the project, please refer to the README.

BPMN specification and BAMOE includes many more types of tasks, but we will only be using a few of them for this lab.

This project also includes a POM file which contains all libraries dependencies needed for the project, as well as BAMOE 9 extensions, such as the ones for the 2 consoles.


### Start

On the Windows desktop, the `start-compact.bat` batch file needs to be run. 
This will start all following services needed by the project that we will be using:

- PostgreSQL database service (port **5432**): this database is needed for this sample because the project contains a stateful business process which requires persistence
- PgAdmin (port **8055**): the UI to manage the database
- Keycloak (port **8480**): UI for the component handling security
- Kogito example service (port **8080**)
- Management Console (port **8280**): UI to manage the process instances
- Task Console (port: **8380**): UI to manage the task inbox

All these services are started as containers and deployed onto Podman.

Please note that the *docker-compose* is useful for testing purposes but in Production clients would most likely not deploy this way.
The projects containing the business logic (i.e. business processes and business rules) are to be built and deployed on Quarkus, which is actually done for you using the *docker-compose* file.


### Interact with the process

- In a new Chrome tab, open keycloak UI 
    - URL: http://localhost:8480
    - Click on *Administration Console* 
    - Login using the following credentials: user: `admin`,  pwd: `admin`
    - Change the realm from *Master* to *Kogito* (top-left menu)
    - Click on *Users*, you should see the different users pre-configured as part of the docker-compose set up
    - For this example, no groups/role have been created, so the business process needs to grant access to human tasks using users directly (this is set up at design). It is actually not a good practice as it is recommended to use groups/roles.
- In a new Chrome tab, open PGAdmin
    - URL: http://localhost:8055
    - Check out the BAMOE 9 tables by expanding `Servers > kogito > Databases > kogito > Schemas > public > Tables`
    - You can see that PAMOE 9 uses many database tables to guarantee that process instances states and data are not lost and can be retrieved
- In a new Chrome tab, open the Management Console
    - URL: http://localhost:8280
    - Login using the following credentials: user: `jdoe`, password: `jdoe`.  This user and his business roles have been added at startup in the keycloak configuration
    - Note that there are no process instance visible yet
- In a new Chrome tab, open the Task Console
    - URL: http://localhost:8380
    - You should not need to login again as the keycloak module is taken care of the SSO
    - Note that there are no human task assign to this user yet
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

This `id` corresponds to the process instance ID which will need to be used for later interaction with the process instance.


- In the management console, refresh the list of process instances. The newly created one should now show. 

![Process Instance List](images/ProcessInstancesList.jpg)

If you had created multiple process instances, they would all show on that console.

- Click on the process instance ID that you just created, you will get its details.

![Process Instance Details](images/ProcessInstanceDetails.jpg)

You can see the information related to this process instance, including the data, the timeline, 
Keep in mind that all this information is persisted in the BAMOE database.

- In the Command shell where the deployment took place, take a look at the logs in the console. You will notice some traces that were generated as the process instances moves forward.

- In the Task Console, refresh the list of tasks. You should now see the task that needs to be completed first, which corresponds to the Human Resource interviewer task.
Please note that there is a boundary timer on this task set to 180 seconds, so if you do not complete this task within 3mn, it will move to the next step of the process.  In such a case, you will need to recreate a new process instance as you did before.

![Task Inbox](images/TaskInboxList.jpg)

- Click on the task instance you want to work on, this will take you to a form that list the data for this task to complete. You may approve or not the condidate. 
Click on the `Complete` button at the bottom of the page. This will tell the process that this task instance is complete and needs to move to the next one, which is the IT interview.

- Switch back to the Process Console, you should see in the diagram that the process instance is now waiting on the IT interview task.

![Process Instance Details](images/ProcessInstanceDetails2.jpg)



## GraphQL queries

BAMOE 9 features GraphQL queries which will allow to send requests to fetch information about process instances.
In a new tab of your browser, go to the following URL: http://localhost:8080/q/graphql-ui/
This will open the GraphQL UI which will let you send queries.
Send a query requesting for information about a specific process instance, by typing the following query:
```graphql
{ ProcessInstances {
  id,
  processId,
  processName,
  state,
  nodes {
    name,
    type,
    enter,
    exit
  }
} }
```

You should get the requested information about all process instances:
```json
{
  "data": {
    "ProcessInstances": [
      {
        "id": "56158ef1-b53c-4b43-8404-e59aa3472566",
        "processId": "hiring",
        "processName": "hiring",
        "state": "ACTIVE",
        "nodes": [
          {
            "name": "Start",
            "type": "StartNode",
            "enter": "2024-09-21T03:39:13.452Z",
            "exit": "2024-09-21T03:39:15.749Z"
          },
          {
            "name": "New Hiring",
            "type": "ActionNode",
            "enter": "2024-09-21T03:39:13.455Z",
            "exit": "2024-09-21T03:39:15.749Z"
          },
          {
            "name": "Split",
            "type": "Split",
            "enter": "2024-09-21T03:39:13.465Z",
            "exit": "2024-09-21T03:39:15.749Z"
          },
          {
            "name": "Generate base offer",
            "type": "RuleSetNode",
            "enter": "2024-09-21T03:39:13.466Z",
            "exit": "2024-09-21T03:39:15.748Z"
          },
          {
            "name": "Log Offer",
            "type": "ActionNode",
            "enter": "2024-09-21T03:39:13.979Z",
            "exit": "2024-09-21T03:39:15.747Z"
          },
          {
            "name": "HR Interview",
            "type": "HumanTaskNode",
            "enter": "2024-09-21T03:39:13.981Z",
            "exit": null
          }
        ]
      }
    ]
  }
}
```



## Use the Canvas

In this lab, we have used VSCode to view and edit our business process and business rules. 

BAMOE 9 also features Canvas, a web application UI for designing and comitting artifacts. Canvas ia accessible on the TechZone VMs at the following URL:  http://localhost:9090

![Canvas](images/Canvas1.jpg)

For IBM BAMOE customers, Canvas is provided as a tool which can connect to a remote enterprise GIT repository.
It also allows non-technical users to participate in a BAMOE project without the need to use an IDE such as VSCode.


### Load and test a DMN

Canvas allows to test DMN on the fly using an extended services.  In the TechZone VM, this service has already been set up (on port 21345).

![Extended Service](images/ExtendedService.jpg)


In the main Canvas panel, load the following local DMN file: `C:\kogito-examples\kogito-quarkus-examples\jbpm-compact-architecture-example\src\main\resources\NewHiringOffer.dmn`
Note that this DMN is the same one that the business process is using, but we now want to test this DMN by itself.

![New Hiring Offer DMN](images/NewHiringOfferDMN.jpg)

In the menu (top-right section) select the "Run > As Form" option. This will generate a form that includes the input data expected by the DMN.
Enter some test data, for instance:

- Candidate name: **John**
- Candidate last name: **Doe**
- Candidate email: **john@doe.com**
- Number of years of experience: **10**
- Skills: **Java**, **NodeJS**

As you enter this data, your will notice that the "Output" section on the right-handside is getting populated. That is because the DMN engine fires the rules as you type the input data in.  The offer `category` and `salary` values are dynamically recomputed based on the logic that is in the business rules of that DMN.

![DMN testing](images/DMNTesting.jpg)


### Load a business process

Upload the following file into Canvas: `C:\kogito-examples\kogito-quarkus-examples\jbpm-compact-architecture-example\src\main\resources\hiring.bpmn`

In another tab, follow the same steps to upload the `C:\kogito-examples\kogito-quarkus-examples\jbpm-compact-architecture-example\src\main\resources\NewHiringOffer.bpmn` file. 

For now, it is not possible to unit test a business process like we just did for a DMN. Future BAMOE releases may offer that functionality.


### Notes on Canvas

- In the TechZone VM, Canvas was not set up to use a remote GIT repository. But clients will want to do so. For now, the following options are avilable within Canvas:

![GIT Providers](images/GitProviders.jpg)

- Similarly, it is possible to deploy directly a business process onto a dedicated DEV environment. For now, the following options are available within Canvas:

![Cloud Runtime Providers](images/CloudRuntimeProviders.jpg)

Note that for Production environment, CI/CD pipelines would be needed because deployment should not be done directly from Canvas.




**This completes the BAMOE 9.1 lab.**