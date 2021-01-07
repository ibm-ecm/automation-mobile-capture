# <u>Overview</u>


IBM Automation Mobile Capture is a platform that helps you to scan and capture data from a variety of documents. By simply using a mobile device, you can capture, edit, and upload the data in a repository of your choice in few seconds.

## Capabilities of IBM Automation Mobile Capture

- `Photo Capturing` - A feature that presents you with a simple camera screen that allows you to take a picture.

- `US Driver's Licence scanning and data extraction` - This feature helps you to scan the front and back of a driver's license, which is issued by the US's Department of Motor Vehicles, and capture a variety of properties. The document must be compliant with AAMVA DL/ID 2006 specification.

- `Passport scanning and data extraction` - This feature allows you to scan any government issued Machine Readable Passport or other TD3 sized Machine Readable Travel Document, using the camera of the device. The document must be fully compliant with Part 4 of ICAO Doc 9303.

- `Barcode scanning and data extraction` - This feature allows you to scan the following types of barcode: Aztec, Code 39, Code 39 with checksum, full ASCII Code 39, full ASCII Code 39 with checksum, Code 93 Code 93i, Code 128, Data Matrix, EAN-8, EAN-13, Interleaved 2 of 5 (ITF), Interleaved 2 of 5 (ITF) with checksum, ITF-14, PDF417, QR and UPC-E.

- `Property editor` - This feature allows you to visualize, edit and add data to any available Scenario properties.

- `Offline functionality` - By using this feature, you can use all major features of the mobile apps, except the upload functionality in an offline status.

- `Smart capturing` - This feature consists of a set of functionalities designed to help you obtain the best possible image capture and data extraction.

- `Import from device` -  This feature allows you to perform data extraction processes on images and PDF documents imported locally from a mobile device.

- `Smart filters` - This is an easy to use feature that grants you the ability to apply a variety of filters to their captured images in order to improve image quality and OCR recognition.

- `Data Provider` - This feature is meant to help you  to complete the extracted data from documents with additional data downloaded from an external service. 

- `Scenario Deep Linking` - This is a function that allows you to open the mobile app at any given scenario.
- `Smart Data Extraction` -  This feature allows you to extract data automatically from a document using Machine Learning.
- `Signature Confirmation` -  . This feature allows you to mark a page in a document as it is signed or not. 
- `Multiple-Page Document Scanning` - Using this feature, you can scan a multiple-page document within the same step. 

## Supported browsers for the Admin Console

- Apple Safari 11 (and later)
- Google Chrome 62 (and later)
- Microsoft Edge 40 (and later)
- Firefox 68 ESR (and later)

# 1. <u>Accessing and managing the Admin Console</u>


## About the Admin Console

The Admin Console represents a Web browser-based, graphical user interface application that is used to manage an IBM Automation Mobile Capture workspace by:

- Creating and managing scenarios
- Adding or removing external integrations
- Inviting and managing users
- Creating and managing other workspaces

## Accessing the Admin Console

To begin using the Admin Console, start one of the supported Web browsers in your local environment and access the URL provided to you at the moment of your IBM Mobile Capture user registration. Do the following steps:

1.	Insert the correct credentials in the **Email** and **Password** fields
2.	Click **Login** 

You can use the **Remember me** checkbox for easier access in case of repeated workspace use.

OBS: When connected to an LDAP server, insert valid credentials associated with said LDAP server.

In case of a forgotten password, click the **Forgot your Password?** link and follow the indicated instructions to reset your password.

OBS: When connected to an LDAP server, you need to reset the password function within the LDAP server configuration.



# 2. <u>Creating and managing scenarios</u>


## What is a scenario?

A scenario represents an assembly of steps, manually added together, to facilitate a document capture and data extraction, as well as their upload into a selected repository.

## Creating a scenario

After you logged in into the Admin Console, access the scenarios page by clicking **Scenarios** on the toolbar, this should be your default page when logged in. Do the following steps:

1.	Click **New Scenario** 
2.	Insert a given scenario name
3.	Click **Create Scenario** 

OBS: Note that in order to create a new scenario, the scenario name field cannot be left empty.

## Configuring a Scenario

A scenario is composed out of the following elements: `Scenario name`, `Scenario Deep Linking`,`Scenario steps`, `Properties`, and `Uploads`.

#### Scenario name

The name of a scenario can be set when creating the particular scenario or by editing it. Do the following steps:

1.	Access a scenario
2.	Edit the **Name** field

#### Scenario deep linking 

A link situated under the Scenario name, which  is copied and accessed from a mobile device with the IBM Automation Mobile Capture app installed,  that opens the app at the beginning of the scenario.

#### Scenario steps

The most configurable part of a scenario are its steps, which represent a selection of elements configured and arranged in a specific order so they can capture, validate and upload data from different types of documents.

Do the following steps to add one or multiple steps:

1.	Enter a given scenario
2.	Click **Add Step** 
3.	Select an available step from the drop-down list

You can **Edit** or **Delete** a step by clicking its corresponding buttons.

##### Type of steps:

1) `US Driver's License:` This step allows you to scan and capture data from a US Driver's License. In this step you can select what data parameters precisely you want to extract from the document. The parameters also represent the Scenario properties in the IBM Mobile Capture platform.

2) `Passport:` This step allows you to scan and capture data from a Passport or a Machine Readable Travel Document. In this step you can select what data parameters you want to extract from this type of documents.

3) `Document:` This step allows you to scan a document with multiple pages, all from within the same step. To configure the step click **Edit**  and do the following steps:

1.	Select the number of pages, which need to be scanned by selecting **Any** (if the number is unknown) or by inserting a specific number
2.	Check or Uncheck the **Show capture preview screen after each page** box, in case of wanting to see a preview each capture when using the mobile app
3.	Check or Uncheck the **Allow user to import from device** box, in case of wanting to import the document/s from the mobile device


4) `Page Inspector:` This step allows you to create a step for scanning a custom document and extracting data from custom fields. To do so, do the following steps:

1.	Click **Edit** 
2.	Click **Choose a file** to upload a copy of the document you like to scan (in PNG or PDF format) and click **Upload**
3.	Create the areas you like to extract data from, by dragging the cursor on the uploaded image
4.	Fill in the **Zones** field/s with their corresponding name. These Zones also represent the Scenario Properties of this step
5.	Check or  Uncheck the **Allow user to import from device** box, in case of wanting to import the document from the mobile device.
6.	Check or  Uncheck the **Ask for signature confirmation** box, in case of wanting a to confirm a signature from the document.
7.	Click  **Save** 


5) `Property Editor:` This step allows users to verify and edit the extracted data of a scenario as well as manually adding data before uploading it to a given repository. To do so, do the following steps:

1.	Click **Edit** 
2.	Click **Add Property** 
3.	Select any available Property from the drop-down list
4.	Click **Save** 

OBS: For adding a custom property select the **New Property** option and customize its name.

In this step, you can configure the Smart Data Extraction feature. To do so, do the following steps:

1.	Select the data type that have to be extracted from the drop-down menu located on the respective property.

2.	Currently, IBM Automation Mobile Capture, supports smart data extraction for the following data types:

- Date
- E-mail
- Phone Number
- Currency (USD, EUR, GBP, SAR)


6)  `Barcode:` This step allows you for scanning of a multitude of barcode formats.

OBS: This step has no editing options.

7)  `Photo:` This step allows you to take one or more photos as part of a scenario. For editing you can do the following steps:

1.	Check or Uncheck the **Allow user to import from device** box, in case of wanting to import the photo/s from the mobile device's gallery
2.	Check or Uncheck the **Allow user to take multiple photos** box in case of wanting to capture multiple photos in this step.

8)  `Data Provider:` This step is meant to support the use case of augmenting data with resort to an external data provider service. To do so, do the following steps:

1.	Click  **Edit** 
2.	Fill in the **URL** of the external data provider service
3.	Configure the **Request** properties
4.	Configure the **Response** properties
5.	Click **Save** 

9)  `Upload:` This is a mandatory step, which allows you to select an additional repository for uploading the extracted data and images of a given scenario. To do so, do the following steps:

a.	Click **Edit** 
b.	Select any available Connection from the drop-down list
c.	Configure the selected Connection
d.	Specify the Properties data (if necessary)

##### Properties

You can view this scenario element in an easy to comprehend format, and the correlation between all Steps and Properties of a scenario. To do so, do the following steps:

1.	Enter a given scenario
2.	Click **Properties**

##### Uploads

This scenario element shows a history of every upload. You can see at a glance: the date and hour of the upload, the number of attachments and the status of the upload. To do so, do the following steps

1.	Enter a given scenario
2.	Click **Uploads**

OBS: If the Upload Step is configured for uploading into an external repository (in DataCap or IBM navigator), the content of each upload can be accessible only inside said repository.


# 3. <u>Inviting and managing users</u>

To invite and manage users, do the following steps:
1.	Click **Team** on the toolbar to access the team page. 
2.	On this page, an administrator can invite and remove users, as well as managing their permissions.


Do the following steps to invite users
1.	From the Team page, Click **Invite User**. 
2.	Insert the email address of the person that you would like to invite using the following format [_user@example.com_](mailto:user@example.com)_._ 
Click **Send Invitation**. Note that you can only invite one user at a time.To remove a user, simply click **Delete** associated with the user and confirm the action in the dialog box.

OBS: When connected to an LDAP server, this features cannot be available. These actions need to be done and completed from within the LDAP server configuration.

## Assigning user roles

To assign a role to a user, click on the current role of a user and select a new one from the drop-down list.

OBS: When connected to an LDAP server, this feature cannot be available. This action needs to be done and completed from within the LDAP server configuration.

## User roles and permissions

| User Role | Permissions |
| --- | --- |
| App user | Users that can only log in in the mobile apps. |
| Workspace User | Users that can log in in the mobile apps as well as the Admin Console and have permissions to create scenarios and connections. |
| Workspace Admin | Users that also have the privilege to invite people to teams as well as create scenarios and connections. |
| Platform Admin | Users that also have the privilege to create new workspaces. |

OBS: When connected to an LDAP server, only the App User and Workspace Admin roles can be available.

# 4. <u>Integrations</u>


IBM Mobile Capture offers integrations with following file and data governance products.

## FileNet Integration

FileNet is a powerful enterprise content management system. Connected to IBM  Navigator it provides users with a web console for working with content that is stored on IBM Content Manager, IBM Content Manager OnDemand, or IBM FileNet® Content Manager repositories.

Do the following steps to set up an IBM® Content Navigator (with FileNet) integration:

1.	Click **Connections** on the toolbar
2.	Go to **New Connection** and select **FileNet**
3.	Insert a Connection name and the credentials associated with your repository
4.	Click **Connect to Navigator** 
5.	Select the repository were you want the data and images to be allocated
6.	Click  **Save** 

To create a connection to a specific IBM Navigator desktop, specifiy the desired desktop in the Navigator URL using the following fromat `/navigator/?desktop=desktopname`


OBS: The uploaded images in the IBM® Content Navigator can be available as a single PDF format file.

## Content Services Integration

Content Services are an easy to use web client for content management. Conected to IBM Navigator on Cloud it provides users with a web console for working with content that is stored on IBM Content Manager, IBM Content Manager OnDemand, or IBM FileNet® Content Manager repositories.

Do the following steps to set up an IBM® Content Navigator integration:

1.	Click **Connections** on the toolbar
2.	Go to **New Connection** and select **Content Services**
3.	Insert a Connection name and the credentials associated with your repository
4.	Click **Connect to Navigator** 
5.	Select the repository were you want the data and images to be allocated
6.	Click  **Save** 

To create a connection to a specific IBM Navigator on Cloud desktop, specifiy the desired desktop in the Navigator URL using the following fromat `/navigator/?desktop=desktopname`

OBS: The uploaded images in the IBM® Content Navigator can be available as a single PDF format file

## Datacap integration

Datacap is a complete solution for data and documents, which helps you to streamline the capture, recognition and classification of business documents and extract important information.

Do the following steps to set up an IBM® Datacap integration:

1.	Click **Connections** on the toolbar
2.	Go to **New Connection** and select **Datacap**
3.	Insert a Connection name and the credentials associated with your Datacap account
4.	Click **Save** 



# 5. <u>Creating and managing other workspaces</u>

With IBM Automation Mobile Capture, a Platform Admin user can create or delete different workspaces within the same platform. 

Do the following steps to create a new workspace:

1.	Click Workspaces on the toolbar
2.	Click **New Workspace** 
3.	Insert the Account name and Owner's Email ([_user@example.com_](mailto:user@example.com))_._
4.	Click **Create Account** 

OBS: When connected to an LDAP server, this feature cannot be available.

# 6. <u>LDAP integration</u>

IBM Automation Mobile Capture supports integration with an LDAP server. As of version 3.1.0, this feature can only be accessible via a Kubernets deployment. Refer to the opportune guide to install Automation Mobile Capture with LDAP support.


