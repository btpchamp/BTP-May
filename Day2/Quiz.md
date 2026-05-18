#### SAP BTP Architecture

**Q1.** How many layers are in the SAP BTP architecture?
- a) 3
- b) 4
- c) 5
- d) 7

--b-

**Q2.** Which layer of SAP BTP contains HANA Cloud and XSUAA?
- a) Runtime Environments
- b) Platform Services
- c) Cloud Infrastructure
- d) Applications


-b--

**Q3.** What is the correct hierarchy in SAP BTP (top to bottom)?
- a) Space → Organization → Subaccount → Global Account
- b) Global Account → Subaccount → Organization → Space
- c) Global Account → Organization → Subaccount → Space
- d) Subaccount → Global Account → Space → Organization

--c-

**Q4.** An "Entitlement" in SAP BTP means:
- a) The amount of a service you can use
- b) Permission to use a specific service
- c) The cost of a service
- d) The region where a service runs

--b-

#### Multi-Cloud & Regions

**Q5.** SAP BTP runs on which cloud providers?
- a) Only AWS
- b) AWS and Azure only
- c) AWS, Azure, and Google Cloud
- d) SAP's own data centers only

-c--

**Q6.** The SAP BTP region "ap21" refers to:
- a) Europe, AWS
- b) Asia Pacific, AWS
- c) Asia Pacific, Azure
- d) Australia, GCP

-b--

**Q7.** What is an Availability Zone?
- a) A country where cloud is available
- b) A separate data center within a region for redundancy
- c) The time zone of a server
- d) A pricing tier for services

-b--

#### Cloud Foundry

**Q8.** Cloud Foundry is:
- a) A proprietary SAP technology
- b) An open-source PaaS platform
- c) A database
- d) A programming language

--b-

**Q9.** What does a Buildpack do?
- a) Builds physical servers
- b) Detects language, installs dependencies, and packages your app
- c) Creates user accounts
- d) Manages database connections

--b-

**Q10.** Which component routes HTTP traffic to the correct application in Cloud Foundry?
- a) Diego
- b) Cloud Controller
- c) GoRouter
- d) UAA

--c-

**Q11.** What happens if your app crashes in Cloud Foundry?
- a) You must manually restart it
- b) Diego automatically restarts it in a new container
- c) The app is permanently lost
- d) An email is sent to the developer

-b--

**Q12.** The command to deploy an application to Cloud Foundry is:
- a) `cf deploy`
- b) `cf push`
- c) `cf upload`
- d) `cf start`

--b-

#### Comparison & Concepts

**Q13.** Which SAP BTP runtime is Kubernetes-based?
- a) Cloud Foundry
- b) Kyma
- c) ABAP Environment
- d) Node.js Runtime

-b--

**Q14.** In Cloud Foundry, you deploy:
- a) Docker containers
- b) Virtual machine images
- c) Source code (buildpack handles the rest)
- d) Compiled binary executables only

-c--

**Q15.** Diego's "self-healing" means:
- a) The code fixes its own bugs
- b) Crashed app instances are automatically restarted
- c) The database backs itself up
- d) Errors are automatically logged

-b--

### Bonus MCQs (5 Extra)

**Q16.** Which CF component is called the "brain" of Cloud Foundry?
- a) GoRouter
- b) Diego
- c) Cloud Controller
- d) Buildpack

-c--

**Q17.** What is a "droplet" in Cloud Foundry?
- a) A small water drop
- b) A packaged, ready-to-run version of your application
- c) A type of service
- d) A logging mechanism

--b-

**Q18.** The SAP BTP service that manages authentication (login) is:
- a) HANA Cloud
- b) XSUAA
- c) Destination Service
- d) HTML5 Repository

-b--

**Q19.** Why does SAP use a multi-cloud strategy?
- a) It's cheaper
- b) To avoid vendor lock-in and meet data sovereignty requirements
- c) Because one cloud isn't powerful enough
- d) SAP owns all three cloud providers

--b-

**Q20.** In the Cloud Foundry account model, where do applications actually run?
- a) Global Account
- b) Subaccount
- c) Organization
- d) Space

-c--

#### Challenge: Name That Component

I'll describe what a component does. You name it!

1. "I receive the developer's code when they type `cf push`" → __cloud controller___________
2. "I figure out what programming language you used" → _______buildpack______
3. "I decide which physical machine should run your app" → __diego___________
4. "I send web traffic to the right application container" → _______GoRouter______
5. "I restart your app automatically if it crashes" → ______diego _______
6. "I create database instances when you request them" → _open service broker api____________
7. "I check your password and issue you a token" → _____UAA________

