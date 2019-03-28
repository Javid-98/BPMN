# Business Process Modeling Notation

## Goal

The goal of this laboratory is to practice the design and simulation of business processes using Business Process Modeling Notation (BPMN) and IBM WebSphere Business Modeler Advanced.

## Pre-requisites

You need the following tools to complete this laboratory:

- IBM WebSphere Business Modeler Advanced 7.0
- Download the [Order.xsd](../master/files/Order.xsd) file

## Material to review before the laboratory

- The [introduction about BPMN](../master/files/01-BPMN.pdf)

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
- **Right click** on the project > **Import...**, then choose **Type definition XML Schema (*.xsd)**
  - Choose the Order.xsd file downloaded from [here](../master/files/Order.xsd)
  - Review the data types found under the namespace www.ibm.com

### Create new business items

  