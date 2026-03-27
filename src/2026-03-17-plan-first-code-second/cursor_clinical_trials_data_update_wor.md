# *** Clinical trials data update worker architecture
_Exported on 3/17/2026 at 23:04:27 GMT+4 from Cursor (2.5.20)_

---

**User**

In our system we’ve go clinical trials database , that currently has moved to mongodb. There is a dedicated script that pulls the AACT database from [clinicaltrials.gov](http://clinicaltrials.gov) website entirely and pushes this database inside our database.

Right now I’ve got a new task: create a separate worker application, which will be using the [clinicaltrials.gov](http://clinicaltrials.gov) API v2, that actually allows requesting the studies updates based on the timeframe. So the worker will be detecting , when was the last update successfully run, fetch the necessary updates range since the last date and then process that data.

Processing data looks like this:

- take a studies list item
- fetch the full item by id
- map it into the defined internal model
- push it into RabbitMQ queue

Each business app, that is interested in consuming the studies updates, will be having the RabbitMQ consumer that will be ingesting the queue data on business apps side. In our case these apps are Acme.API and OrganizationList. In scope of this task we only should prepare the RabbitMQ wiring classes on the business apps side and leave actual business side queue processing out of scope for now - just do a mock processor that logs the incoming queue items, that’s it

There is also one challenge: we’ve got 3 stage environments right now and one production. We would like one worker to do the job once for all the environments. This means we need such infrastructure setup that would make it possible for a particular container app in Azure to push items into different queues for each existing environment. This is actually an open question , how exactly should this be done. Does it mean each env has it’s own rabbitmq instance? Or should we have a single RabbitMQ for all envs and different queue names for different envs? How exactly are we going to open subnets of stage envs to connect to a ‘generic’ service that is only one for all envs?

So I would like you to create a Mermaid diagram code for me that will show the high level architecture module. I’d like you to include a worker module, rabbit mq models (or modules), 4 stage envs boundaries, each env has Acme.API and OrganizationList app inside it. And ofcourse connections between the modules with basic functional description of those connections.
Please analyze my requirements and my needs and give me a set of clarification questions that you need to be answered before you can effectively plan the architecture. Only ask questions now, don’t do anything else yet, we’ll do final decisions and diagram later.

---

**Cursor**

## Infrastructure & Deployment

1. **Worker deployment model**: Is the worker deployed as a single Azure Container App that runs periodically (cron/scheduled), or as a continuously running service? Should it handle all environments in one execution cycle, or trigger separate processing runs per environment?

2. **RabbitMQ topology**: Do you have separate RabbitMQ instances per environment (dev/stage/prod), or a single shared RabbitMQ cluster? What are your isolation/security requirements?

3. **Network connectivity**: If using a single worker for all environments, what networking model should we assume?
   - VPN/private endpoints to each environment’s RabbitMQ?
   - Public endpoints with authentication?
   - Azure Virtual Network peering?
   - Or should the diagram reflect this as a design decision point?

4. **Environment identification**: How should the worker determine which environments to target? Is this configuration-driven, or should it automatically discover environments?

## Data & Processing

5. **Last update tracking**: Should "last successful update" be tracked:
   - Per environment (each env has its own last update timestamp)?
   - Globally (single timestamp applied to all environments)?
   - Per study ID (tracking which studies were last updated in each env)?

6. **Message routing strategy**: When the worker processes a study update, should it:
   - Push the same message to queues for all 4 environments simultaneously?
   - Allow filtering (e.g., only push to specific environments based on some criteria)?
   - Support different queue names per environment (e.g., `studies-updates-dev`, `studies-updates-prod`)?

7. **Queue structure**: Should each business application (Acme.API, OrganizationList) have:
   - Separate queues per environment (e.g., `acme-api-dev`, `acme-api-prod`)?
   - Or a single queue per environment that both apps consume from (with message routing keys)?

## Application Architecture

8. **RabbitMQ integration pattern**: For the mock consumers in Acme.API and OrganizationList, should they use:
   - Direct queue consumption?
   - Exchange-based routing (topic/direct/fanout)?
   - Dead letter queues for failed messages?

9. **Shared models/libraries**: Should the RabbitMQ message models (DTOs) be in a shared library that the worker and business apps reference, or will each define its own contracts?

10. **Worker scope**: Beyond fetching and queuing, should the worker handle:
    - Rate limiting/throttling for the API?
    - Retry logic for failed API calls?
    - Metrics/telemetry?
    - Database persistence of last update timestamps?

## Operational

11. **Environment naming**: What are the exact names of the 3 stage environments? (e.g., dev, staging, qa, or custom names?)

12. **Failure scenarios**: If the worker fails mid-execution (e.g., after processing some studies but before completing all environments), how should this be handled? Should partial updates be allowed, or should it be all-or-nothing per environment?

Which of these should we prioritize, and are there other constraints or requirements to include in the architecture diagram?

---

**User**

1. Worker deployment model: let's assume it is a continuously running service
2. RabbitMQ topology: let's assume there is a single RabbitMQ cluster. In regards to isolation: I have to figure out this from our devops engineers. Let's assume that by default each acme service environment has a closed perimiter. This means I need you advice on how to open access for particular service connection outside of the acme env subnet.
3. Network connectivity: let the diagram show it as an open decision point
4. Environment identification: this should be a configurable set of environment specific config: an array of environments enabled, config object with key value pairs for each environment name
5. Last update tracking: let's start with global timestamp value. We can evolve it later, this is not an architecture level concern, I think
6. Message routing strategy: I think it should push to different queue names per environment and push the same message to whatever set of environments is configured in the worker config
7. Queue structure: separate queue per environment will be cleaner
8. RabbitMQ integration pattern: Dead letter queues for failed messages - for sure, Direct queue consumption - not sure what is it?, exchange-based routing (topic/direct/fanout) - not sure what is it?
9. Shared models/libraries: the worker will be in a separate repository. However, it would be nice to have a shared models module as a private nuget package. This nuget package could live happily beside the worker code and fetched as a dependency in the apps that consume the queue
10. Worker scope: Rate limiting/throttling for the API -definitely should, Retry logic for failed API calls - definitely should, Metrics/telemetry - definitely should, Database persistence of last update timestamps - definitely should
11. Environment naming: What are the exact names of the 3 stage environments?: dev, uat, stage, prod
12. Failure scenarios: the worker must remember the last timestamp of a study that has been successfully pushed to the queue for all environments and store it in a database. This would mean that the next sceduled run will pick up the update range starting from that timestamp

@ACME-6429 - please document your understanding of the requirements inside this folder to a markdown file. Write out the requirements and details in a structured way, that will help you to write the architecture schema in the future.
For now just write that document with all the information in mind, give me that document for review. Don't do anything else yet

---

**Cursor**

Checking the project structure to locate the ACME-6429 folder and understand the documentation conventions.


Checking the plans directory structure:


Creating the ACME-6429 folder and the requirements document:


Created the requirements document. Verifying it was created correctly:


Created the requirements document at `/Users/bunhackr/projects/acme/plans/ACME-6429/01-Requirements-Documentation.md`.

The document includes:

1. **Overview & Background** - Context about current state and new requirements
2. **Functional Requirements** - Worker functionality, data synchronization, error handling
3. **Infrastructure & Deployment** - Worker deployment model, environment configuration, multi-environment processing
4. **RabbitMQ Architecture** - Topology, queue structure, routing strategy, integration patterns
5. **Business Applications Integration** - Consumer applications, implementation scope
6. **Data Models & Shared Libraries** - Message models, NuGet package approach
7. **Technical Requirements** - API integration, observability, data persistence
8. **Architecture Decision Points** - Open questions and design decisions
9. **Environment Configuration** - Environment details and isolation
10. **Implementation Scope** - In-scope and out-of-scope components
11. **Success Criteria** - Definition of done
12. **Next Steps** - Planning for architecture design and implementation

Organized to support creating the architecture diagram. All requirements and decisions from our conversation are documented.

Ready for your review. After you review and provide any updates, we can proceed with creating the Mermaid architecture diagram.

---

**User**

few clarifications:
- 'endpoints to each environment's RabbitMQ' - I thought we agreed to have 1 rabbitmq instance with different queue names per environment. This would mean that consumers from within an acme env subnets must be able to subscribe to that rabbitmq from a different subnet
- '**Worker Repository**' and '**Shared Models Repository**' - these can be inside single repository
Please add clarifications to the document.



---

**Cursor**

Reviewing the document to identify sections to update with these clarifications.


Updating the document with the clarifications:


Updated the document with the clarifications. Summary of changes:

## Updates Made:

1. **RabbitMQ Topology (Section 3.1)**:
   - Clarified there is a single RabbitMQ cluster/instance shared across all environments
   - Updated the network access question to focus on how consumers from within Acme environment subnets connect to the shared RabbitMQ
   - Updated the options to reflect the single RabbitMQ instance architecture

2. **Shared Models Library (Section 5.1)**:
   - Updated to note it can be in the same repository as the worker application (or separate)

3. **Network Connectivity (Section 7.1)**:
   - Rewritten to reflect the single RabbitMQ instance architecture
   - Updated the question to focus on consumers connecting to the shared RabbitMQ from different subnets
   - Removed references to multiple RabbitMQ instances

4. **Worker Repository Structure (Section 7.3)**:
   - Updated to clarify that Shared Models Library can be in the same repository as the worker
   - Kept the note about NuGet package distribution

5. **Environment Components (Section 8.2)**:
   - Updated to clarify that RabbitMQ is a shared single instance used by all environments
   - Added notes that applications consume from the shared RabbitMQ

The document now reflects:
- A single RabbitMQ instance/cluster with environment-specific queue names
- The network connectivity challenge is about consumers (within Acme env subnets) connecting to the shared RabbitMQ (in a different subnet)
- Worker and Shared Models can be in the same repository

All changes are consistent throughout the document.

---

**User**

- '*Direct Queue Consumption**: To be determined (requires clarification on pattern)' - can you explain what it direct queue consumption and whether we need it or not
- '**Exchange-Based Routing**: To be determined (topic/direct/fanout - requires clarification)' - can you explain what is it exactly and whether we need it for our use case


please answer my questions so that I could make a decision on these points

---

**Cursor**

Explaining these RabbitMQ concepts and their relevance to this use case:


## Explanation of RabbitMQ Patterns

### 1. Direct Queue Consumption

**What it is:**
- "Direct queue consumption" means consumers read directly from specific named queues.
- The worker publishes directly to a queue, and consumers subscribe to that queue.
- This is the simplest pattern and is what most RabbitMQ implementations use.

**How it works:**
- Worker publishes to queue: `acme-api-studies-updates-dev`
- Acme.API consumer subscribes to queue: `acme-api-studies-updates-dev`
- Messages flow directly: Producer → Queue → Consumer

**Do you need it?**
Yes. This matches your use case. You have specific queue names per environment/application, so direct queue consumption is appropriate.

---

### 2. Exchange-Based Routing

**What it is:**
RabbitMQ exchanges route messages to queues. There are several types:

**a) Direct Exchange:**
- Routes messages to queues based on an exact routing key match
- Example: Routing key `studies.updates.dev` → Queue `acme-api-studies-updates-dev`
- You can use this, but direct publishing to queues is simpler for your case

**b) Topic Exchange:**
- Routes messages using pattern matching on routing keys (wildcards like `*` and `#`)
- Example: Routing key `studies.updates.*` → Routes to all queues matching the pattern
- Useful when you need dynamic routing or want to add new queues without code changes

**c) Fanout Exchange:**
- Broadcasts messages to ALL bound queues (no routing key needed)
- Example: One message goes to Queue A, Queue B, Queue C simultaneously
- Useful for pub/sub scenarios where all subscribers should receive everything

**Do you need it?**
For your current use case, probably not:
- You have a fixed set of queues (one per application per environment)
- Queue names are known at development time
- You're doing point-to-point messaging (one producer → specific queue → one consumer)
- Direct queue publishing is simpler and sufficient

You might consider exchanges later if:
- You need to add new consumers without code changes
- You need dynamic routing based on message content
- You want to implement pub/sub patterns

---

## Recommendation for Your Use Case

**Use Direct Queue Consumption:**
- Worker publishes directly to queues: `acme-api-studies-updates-{env}`, `organizationlist-studies-updates-{env}`
- Consumers subscribe directly to their queues
- Simple, straightforward, and fits your requirements

**Skip Exchange-Based Routing (for now):**
- Direct publishing to queues is simpler
- Your routing needs are static and known upfront
- No need for the added complexity of exchanges
- Can introduce exchanges later if requirements change

---

## Summary

- Direct Queue Consumption = YES, use this approach
- Exchange-Based Routing = NO, not needed for your current use case

Should I update the document to reflect this decision, or do you want to review first?

---

**User**

yes please reflect this decision in the document

---

**Cursor**

Updating the document to reflect the decision to use Direct Queue Consumption and skip Exchange-Based Routing.


Updating the relevant sections to reflect the decision:


Updating the "Next Steps" section to reflect that RabbitMQ patterns have been decided:


Updated the document to reflect the decision. Changes:

1. **Section 3.4 (RabbitMQ Integration Patterns)**: Updated to specify:
   - **Direct Queue Consumption**: YES — with rationale
   - **Exchange-Based Routing**: NO — with rationale and note about future consideration

2. **Section 7.2 (Architecture Decision Points)**: Changed from "TO BE DETERMINED" to "DECIDED" with the decision details

3. **Next Steps**: Removed "RabbitMQ patterns" from the open questions list since it's now decided

The document reflects that:
- Direct queue consumption will be used
- Exchange-based routing is not needed for the current use case
- The decision is documented with rationale

All sections are now consistent with this decision.

---

**User**

Alright. Do you have all information you need to produce the Mermaid diagram of high level architecture ? just answer, don't change code/docs

---

**Cursor**

I have enough information to create the Mermaid diagram. I can show:

- **Worker module** (Azure Container App, continuously running)
- **ClinicalTrials.gov API v2** (external data source)
- **Database** (for last update timestamp storage)
- **Single RabbitMQ instance/cluster** (shared across all environments)
- **4 environment boundaries** (dev, uat, stage, prod)
- **Applications per environment** (Acme.API and OrganizationList in each)
- **Queue structure** (environment-specific queue names: `{app-name}-studies-updates-{env}`)
- **Connections** (worker → RabbitMQ → consumers, with functional descriptions)
- **Direct queue consumption pattern**

One open question: network connectivity details (how consumers in Acme env subnets connect to the shared RabbitMQ), but that can be noted as a design decision point in the diagram without blocking the architecture visualization.

Ready to create the diagram when you are.

---

**User**

Alright, please create the diagram code. The mermaid syntax should be supported by the official online mermaid editor https://www.mermaidchart.com/ . Put the diagram code as a code block section of a markdow file in the same folder as requirements @plans/ACME-6429/ (create new markdown file with mermaid)

---

**Cursor**

Creating the Mermaid architecture diagram and saving it as a markdown file.


Created the Mermaid architecture diagram and saved it to `/Users/bunhackr/projects/acme/plans/ACME-6429/02-Architecture-Diagram.md`.

The diagram includes:

1. **Worker Module** - Azure Container App with its responsibilities
2. **ClinicalTrials.gov API v2** - External data source
3. **Database** - For timestamp storage
4. **Single RabbitMQ Instance/Cluster** - Shared infrastructure with queues organized by environment
5. **4 Environment Boundaries** - dev, uat, stage, prod (each as a closed perimeter)
6. **Applications per Environment** - Acme.API and OrganizationList in each environment
7. **Queue Structure** - All queues with proper naming convention
8. **Connections** - Labeled connections showing:
   - Worker fetching from API
   - Worker reading/writing to database
   - Worker publishing to RabbitMQ
   - Queues connecting to consumers
   - Network connectivity marked as TBD

The diagram uses:
- **Subgraphs** for environment boundaries and queue organization
- **Solid lines** for direct connections
- **Dashed lines** for network connections requiring infrastructure configuration
- **Color coding** for different component types
- **Labels** describing the functionality of each connection

The Mermaid syntax is compatible with mermaidchart.com. You can copy the code block and paste it into the editor to view and edit the diagram.

---

**User**

there is a layout glitch, the title of the box gets overlaped (see screenshot) please help mitigate this somehow

---

**Cursor**

Reviewing the diagram to fix the overlapping text issue.


Fixing the overlap by simplifying the Worker subgraph structure. Removing the unnecessary nested subgraph and consolidating the information into the node itself:


Fixed the layout issue by removing the nested subgraph structure. The Worker is now a single node instead of a subgraph containing another node, which was causing the text overlap.

The Worker node now contains all the information in a single label:
- Title: "Worker Application"
- Deployment info: "Azure Container App - Continuously Running"
- Separator line
- Responsibilities as bullet points

This should render cleanly in Mermaid Chart without any text overlap. The diagram functionality remains the same, but the layout should be cleaner and more readable.

---

**User**

- you need to reflect that the worker service and the database for timestamps belong to one service cluster/subnet
- please write out only 'dev' and 'uat' module sets and show somehow that same module sets can be for any number of environments. writing out 4 packs of exactly same modules just makes the chart hard to read

---

**Cursor**

Updating the diagram to group the Worker and Database in a service cluster/subnet, and showing only dev and uat environments with a note indicating the pattern repeats.


Updating the diagram to group the Worker and Database in a service cluster and show only dev and uat environments:


Simplifying the notes approach. Mermaid notes may not render as intended, so trying a cleaner alternative:


Updated the diagram:

1. **Worker Service Cluster**: Wrapped Worker and Database in a subgraph labeled "Worker Service Cluster/Subnet" to show they belong to the same service cluster/subnet.

2. **Simplified Environments**: Only showing **dev** and **uat** environments explicitly. Removed stage and prod from the visual diagram to improve readability.

3. **Documentation**: Added notes in the "Architecture Components" section explaining that:
   - The diagram shows dev and uat as examples
   - Stage and prod follow the same pattern (same queue structure and applications)
   - This pattern applies to any number of environments

The diagram is cleaner and easier to read, while the documentation clarifies that the pattern repeats for all environments. The Worker Service Cluster grouping shows the Worker and Database belong together in their own service cluster/subnet.

---

**User**

- remove 'mock processor' words
- please suggest a type of light weight database that can be used for storing the last timestamp thing. It look tiny , there won't be a need for any complex db management. Something that does not require separate cloud resources would be best

---

**Cursor**

Removing "mock processor" from the diagram and suggesting a lightweight database option.


Removing "mock processor" and suggesting SQLite as the lightweight database option:


Made the requested changes:

1. **Removed "Mock Processor"**: Removed from all diagram nodes and documentation references
2. **Suggested SQLite Database**: Added SQLite as the recommended database option with rationale

**SQLite** is a good fit because:
- **No separate cloud resources**: It's a file-based, embedded database (single file)
- **Lightweight**: Minimal overhead, no database server to manage
- **Simple**: Perfect for storing a single timestamp value
- **Zero configuration**: Works out-of-the-box
- **Container-friendly**: Works well in Azure Container Apps (can use Azure Files for persistent storage if needed)
- **Well-supported in .NET**: Native support via `Microsoft.Data.Sqlite`

The diagram and documentation now reflect SQLite as the database choice, and all "Mock Processor" references have been removed. The database is shown as "SQLite Database" in the diagram, and the documentation includes a section explaining why SQLite is the recommended choice.

---

**User**

will sqlite survive worker service deployment? or will it die together with the container itself? 

---

**Cursor**

Clarifying the SQLite persistence behavior in containerized deployments:


SQLite in the container filesystem will not survive redeployment. The database file is stored in the container's filesystem, which is ephemeral. When the container is recreated, restarted, or redeployed, the file is lost.

Updating the documentation to clarify this and provide options:


Updated the documentation to clarify the persistence issue.

**Short answer**: SQLite stored in the container filesystem will not survive container redeployment. The data is lost when the container is recreated.

**Implications**:
- The last update timestamp would be lost
- After redeployment, the worker wouldn't know where it left off
- This could cause duplicate processing or require manual intervention

**Options**:

1. **SQLite with Azure Files volume** (recommended if persistence is needed)
   - Mount an Azure Files share to store the SQLite file
   - Data persists across deployments
   - Requires an Azure Storage Account (minimal resource, ~$1/month)

2. **SQLite in-container only** (no persistence)
   - Accept that the timestamp resets on redeployment
   - Only viable if occasional resets are acceptable

3. **Azure Table Storage** (alternative)
   - Fully managed, pay-per-use
   - Free tier often covers this use case
   - Simple API for storing a single timestamp

Given that the timestamp is critical for the worker's incremental sync, you'll likely need either Azure Files (for SQLite) or Azure Table Storage for persistence. Both are minimal, low-cost resources. Which approach do you prefer?

---

**User**

let's mention sqlite with azurefiles as a finalized decision

---

**Cursor**

Updating the documentation to reflect SQLite with Azure Files as the finalized decision.


Updated the documentation to reflect SQLite with Azure Files as the finalized decision. Changes:

1. **Diagram label**: Updated to show "SQLite Database - Azure Files Volume"
2. **Worker Service Cluster section**: Updated database description
3. **Database Choice section**: Changed from "options" to a "DECIDED" section that:
   - Clearly states it's the selected solution
   - Explains the implementation approach
   - Provides rationale for the choice
   - Notes the minimal infrastructure requirements

The documentation now clearly indicates that SQLite with Azure Files volume is the finalized architectural decision for timestamp persistence.

---

**User**

- remove '(Direct Queue Consumption)<br/>[Network connectivity TBD]' words - they are repetitive unnecessary
- remove '(closed perimiter)' words - they are unnecessary

---

**Cursor**

Removing the repetitive text from the diagram.


Removed the repetitive text from the diagram:

1. Removed `(Closed Perimeter)` from the environment subgraph labels (Dev Environment and UAT Environment)
2. Removed `(Direct Queue Consumption)<br/>[Network connectivity TBD]` from the consumer connection labels (now just says "Consume messages")

The diagram is cleaner and more concise while maintaining the essential information.

