# Business Process Modeling Notation

## Goal

The goal of this laboratory is to practice the design and simulation of business processes using Business Process Modeling Notation (BPMN) and IBM WebSphere Business Modeler Advanced.

## Pre-requisites

You need the following tools to complete this laboratory:

- IBM WebSphere Business Modeler Advanced 7.0
- Download the [Order.xsd](../master/files/Order.xsd) file

## Material to review before the laboratory

- The [introduction about BPMN](../master/files/01-BPMN.pdf)

## Submitting your solution

Please upload a document containing the specified screenshots showing the solution of the [Standalone tasks](#standalone-tasks)

## The example business process

Our example is an order handling process at a webshop. Each order can be fulfilled or declined based on certain conditions.The main logic of the process:

- The process is started by receiving an order
- Decision is made, whether an automatic approval is sufficient (total price is under $750)
- If automatic approval is enabled, and customer has enough credit, the order is shipped and DB is updated
- If automatic approval is not enabled, or the the automatic approval rejects the order because of insufficient customer credit, an Order Manager reviews the order
- The decision of the Order Manager can be positive, in this case the order is shipped and DB is updated
- If the decision is negative, the order is canceled and a notification is sent about it

## Joint tasks

### Setup

- Start **WebSphere Business Modeler** from the Start menu
- If you find existing projects, delete them all
- Apply **4-pane layout** with an icon in the toolbar
- From the menu bar select **Modeling > Mode > Advanced > WebSphere Process Server**

### Create project and import type definitions

- From the menu bar select **File > New Business Modeling Project**
  - The project name can be Webshop, and the business process name Order Handling
- **Right click** on the project **> Import...**, then choose **Type definition XML Schema (*.xsd)**
  - Choose the Order.xsd file downloaded from [here](../master/files/Order.xsd)
  - Review the data types found under the namespace www.ibm.com

### Create new business item

- **Right click** on the project **> New > Business Item**
- Name: **Notification**
  - add 2 properties: **email, text**, with type Text

### Create Resources

Our process has two human tasks: the order review and the shipping is done by employees of the webshop. We have to define the cost and availability of the corresponding human resources.

- Right click the **Resources** folder in the project tree, and select **New > Timetable**. Enter **Day Shift** as name and click Finish. The Day Shift opens in the editor.
  -  Leave 1 day as repetition period
  - Click Select time and set Beginning on to **January 1, 8 am**.
  - Click Select duration and set the value to **9 hours**.
  - Select Start time and set the time to **January 1, 8 am**.
  - Save and close the Day Shift.
- Right click the **Resources** folder in the project tree, and select **New > Role**
  - The name of the role is **OrderManager**
  - The cost is **20.00/hour**
  - Availability is the **Day Shift**
- Repeat the previous steps to create a second role called **Shipper**, with the same availability, but with a cost of **10.00/hour**

### Define the business process

We add the following tasks and gateways to the business process editor via drag and drop. For each task, we define a duration in the **Attributes** view **> Duration\Processing Time > Specific Duration**

- Business Rules Task: **Check Order Handling Policy for Automatic Approval**, duration: 1 second

- Simple Decision: **Approve Without Review?** 

- Task: **Check Customer Account Status,** duration: 1 second

- Simple Decision: **Account in Good Standing?**

- Merge

- Human task: **Review Order**, duration: 15 minutes

- Simple Decision: **Acceptable Credit Risk?**

- Merge

- Human task: **Ship Order**, duration: 20 minutes

- Task: **Cancel Order** **and Send Notification,** duration:1 second

- Task: **Update Order Database**, duration: 1 second

- 2 Terminate nodes

Next we connect the elements above according to the control flow. We start with connecting the bounding border of the process to the first task. While drawing the connector lines, we set the associated data in the **Attributes** view to the **Order** data type, except for the one after the **Cancel...** task, where we choose **Notification**.

Next, we assign costs to the automatic tasks and resources to the human tasks:

- Check Customer Account Status: $2
- Review Order: Additional Resources\Role\Add... : Order Manager, 15 minutes
- Ship Order: Shipper, 20 minutes

We edit the Simple Decision gateways to define the probability of the Yes and No branches, and define the expression for the Yes branch.

- **Approve Without Review:** 
  - Yes: 65%, No: 35%
  - Choose the Yes branch, set the expression as Modeling artifact: automaticApproval is Equal to true (boolean)
- **Account in Good Standing:**
  - Yes: 85%, No: 15%
  - Modeling artifact: TotalPrice is less than AvailableCredit (Modeling artifact)
- **Acceptable Credit Risk:**
  - Yes: 70%, No: 30%
  - Modeling artifact: OrderStatus is equal to Approved (Text)

Because our process is a long running process, the caller of the process won't wait until the process is finished. We define this behavior this way:

- Click any free area of the canvas
- Navigate to **Attributes > Technical Attributes > Request tab > Input Criterion**. Choose **One way operation**

The tool is able to generate an implementation skeleton of the process from our BPMN model. If we add annotations (practically notes) to the model, they will appear as comments in this implementation skeleton. We add two annotations:

- Connected to **Update Order Database:** Update Order in Order Database to SHIPPED
- Connected to **Order Handling Policy for Automatic Approval**: Orders under $750 are automatically approved

The **Check Order Handling Policy for Automatic Approval** task is a Business rules task. Business rules are evaluated external to the process, e.g. via a business rule engine, instead of hardwiring them into the process. This way, when business rules e.g. taxes, discounts, price limits change, we do not have to modify and redeploy the whole process, just change the value in the business rule engine. Hence, we do not implement them in our model, just describe the rule in the annotation. But we still have to define that the input of the rule is copied to the output of the rule:

- Select **Check Order Handling Policy for Automatic Approval**
- Navigate to **Attributes/Business Rules** and click **Add **to define a new rule
- Accept the default name for the rule (Copy Input to Output)
- If-Then Rules -> Add Rule
- OK
- Select this newly created rule at Business Rule

### Simulation

The tool can *simulate* the business process, so we can evaluate the business measures of our process in the modeling phase, without the need to implement it. We can even do some optimization by exploring alternative versions of the business process. During the simulation, tokens represent the orders, running through the tasks and gateways of the process. We first run an animated simulation with 10 tokens, then another one with 100 tokens, without animation.

- **Right click** the project **> Simulate**
  - This way a snapshot was created of the business process. Later, we can modify the original process, but this snapshot remains there and we can see the corresponding simulation results as well. By repeating Right click > Simulate, new snapshots can be created based on the actual state of the process.
- Edit the simulation profile
  - General
    - Random number seed: 1
    - Process availability begins: **January 1, 8 am**.
  - Input
    - Number of tokens per bundle: 1
    - Total Number of tokens: 10
    - Time trigger
      - Start time: **January 1, 8 am**
      - Time between bundles: 5 minutes
  - Resource pool:
    - Order Manager and Shipper should be set to 1 (uncheck unlimited)
- Run the simulation
  - Navigate to Simulation Control Panel, and choose the dropdown menu in the upper right corner > Setting > Display animation
  - Click the Run icon and watch the tokens running
  - Review the results
  - Repeat the run with 100 tokens and the animation switched off
- Analysis
  - Right click on the simulation result, select Dynamic Analysis > Process Cases Analysis > Process Cases Summary
    - We should see 5 cases here, each case can be selected and the corresponding path in the process is highlighted
  - Dynamic Analysis > Aggregated Analysis > Resource Usage
    - Shortage duration shows, which roles mean the bottleneck in our process
  - Dynamic Analysis > Process Cases Analysis > Process Cost

## Standalone tasks

After running the simulation and investigating the analysis results, our goal is to lower the costs. For this reason we want to introduce a Classification of the customers. (GOLD, SILVER, BRONZE) Higher classification means a higher limit for automatic approval. This way we expect that the Order Manager will have less work causing lower cost. Implement this change by:

- Adding a Classificaton attribute of type Text to the Customer Record data type
- Modifying the annotation to this text:
  - Orders are automatically approved if:
    - the total order price is less than $750
    - the total order price is less than $1250 and the customer is SILVER
    - the total order price is less than $1750 and the customer is GOLD
  - It is important that you should *not* implement this rule, just change text in the corresponding annotation
- Modifying the branching percentages at the gateways
  - **Approve Without Review:** Yes: 80%, No: 20%
  - **Account in Good Standing:** Yes: 90%, No: 10%
  - **Acceptable Credit Risk:** Yes: 75%, No: 25%

Introduce a second modification as well so that the Order Manager should have even less work. In the current process, when the **Check Customer Account Status** automatic task finds that the account is not OK, the order is sent to an Order Manager, who can override this decision. You should change this control flow, so that the automatic rejection causes the cancellation of the order directly.

### Screenshots to be included in the report

**Important:** for each screenshot, explain in a few sentences, what has been changed compared to the first version of the process.

- Create a screenshot about the modified process.

- Run a simulation on the modified process, with the same settings as the first time, but only with 100 tokens, no animation. View **Dynamic Analysis > Process Cases Analysis > Process Cases Summary**, then create a screenshot while the first case is selected, and the corresponding path is highlighted in the process diagram. Repeat this for all the other cases as well.
- Create a screenshot of **Dynamic Analysis > Aggregated Analysis > Resource Usage**