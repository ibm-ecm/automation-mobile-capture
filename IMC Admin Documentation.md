# <u>Overview</u>


IBM Automation Mobile Capture is a platform which allows users to scan and capture data from a variety of documents. By simply using a mobile device, users can capture, edit, and upload the data in a repository of their choice in a matter of seconds.

## Capabilities of IBM Automation Mobile Capture

- `Photo Capturing` - A feature that presentes the user with a simple camera screen which allows him/her to take a picture.

- `US Driver's Licence scanning and data extraction` - This feature allows the user to scan the front and the back of a driver's license, issued by the US's Department of Motor Vehicles, and capture a variety of properties. The document must be compliant with AAMVA DL/ID 2006 specification.

- `Passport scanning and data extraction` - This feature allows the scanning of any government issued Machine Readable Passport or other TD3 sized Machine Readable Travel Document, using the camera of the device. The document must be fully compliant with Part 4 of ICAO Doc 9303.

- `Barcode scanning and data extraction` - This feature offers the possibility in the scanning of the following types of barcode: Aztec, Code 39, Code 39 with checksum, full ASCII Code 39, full ASCII Code 39 with checksum, Code 93 Code 93i, Code 128, Data Matrix, EAN-8, EAN-13, Interleaved 2 of 5 (ITF), Interleaved 2 of 5 (ITF) with checksum, ITF-14, PDF417, QR and UPC-E.

- `Property editor` - This feature allows the user to visualize, edit and add data to any available Scenario properties.

- `Offline functionality` - By using this feature, the users can perform all major features of the mobile apps, except the upload functionality, in an offline status.

- `Smart capturing` - This feature consists of a set of functionalities designed to help users obtain the best possible image capture and data extraction.

- `Import from device` -  A feature that allows users to perform data extraction processes on images and PDF documents imported locally from a mobile device.

- `Smart filters` - An easy to use feature that grants users the abiity to apply a variety of filters to their captured images in order to improve image quality and OCR recognition.

- `Data Provider` - This feature is meant to help users complete the extracted data from documents with additional data downloaded from an external service. 

- `Scenario Deep Linking` - A functionality which allows the user to open the mobile app at any given scenario.

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

To begin using the Admin Console, start one of the supported Web browsers for your local environment and access the URL provided to you at the moment of your IBM Mobile Capture user registration.

- Insert the correct credentials in the **Email** and **Password** fields
- Click the **Login** button.

You can use the **Remember me** checkbox for easier access in case of repeated workspace use.

OBS: When connected to an LDAP server, please insert valid credentials associated with said LDAP server.

In case of a forgotten password, click the **Forgot your Password?** link and follow the indicated instructions to reset your password.

OBS: When connected to an LDAP server, the reset password functionality needs to be performend and completed from within the LDAP server configuration.



# 2. <u>Creating and managing scenarios</u>


## What is a scenario?

A scenario represents an assembly of steps, customly added together, to facilitate a document capture and data extraction, as well as their upload into a selected repository.

## Creating a scenario

Once you have managed to successfully login into the Admin Console, access the scenarios page by clicking the **Scenarios** button on the toolbar, this should be your default page when logged in.

- Click the **New Scenario** button
- Insert a given scenario name
- Click the **Create Scenario** button.

OBS: Note that in order to create a new scenario, the scenario name field can not be left empty.

## Configuring a Scenario

A scenario is composed out of the following elements: `Scenario name`, `Scenario Deep Linking`,`Scenario steps`, `Properties`, and `Uploads`.

#### Scenario name

The name of a scenario can be set when creating the said scenario or by editing it.

- Access a scenario
- Edit the **Name** field

#### Scenario Deep Linking 

A link situated under the Scenario name, which if copied and accessed from a mobile device with the IBM Automation Mobile Capture app installed, will open the app at the beginning of said scenario.

#### Scenario steps

The most configurable part of a scenario are its steps, which represent a selection of elements configured and arranged in a specific order so they can capture, validate and upload data from different types of documents.

To add one or multiple steps:

- Enter a given scenario
- Click the **Add Step** button
- Select an available step from the drop-down list

You can **Edit** or **Delete** a step by clicking its corresponding buttons.

##### Type of steps:

1) `US Driver's License:` This step allows users to scan and capture data from a US Driver's License. Editing this step will enable users to precisely select what data parameters they wish to extract from the document. These parameters also represent the Scenario Properties in the IBM Mobile Capture platform.

2) `Passport:` This step allows users to scan and capture data from a Passport or a Machine Readable Travel Document. Editing this step will also enable users to select what data parameters they wish to extract from this type of documents.

3) `Document:` This step allows users to create a step for scanning a custom document and extracting data from custom fields. To do so:

- Click the **Edit** button
- Click the **Choose a file** button to upload a copy of the document you would like to scan (in PNG or PDF format) and click **Upload**
- Create the areas you would like to extract data from, by dragging the cursor on the uploaded image
- Fill in the **Zones** field/s with their corresponding name. These Zones also represent the Scenario Properties of this step
- Click the **Save** button

OBS: By editing this step, the user can select or unselect the import from device feature.

4) `Property Editor:` This step allows users to verify and edit the extracted data of a scenario as well as manually adding data before uploading it to a given repository. To do so:

- Click the **Edit** button
- Click the **Add Property** button
- Select any available Property from the drop-down list
- Click the **Save** button

OBS: For adding a custom property select the **New Property** option and customize its name.

5)  `Barcode:` This step allows users the possibility of scanning a multitude of barcode formats.

OBS: This step has no editing options.

6)  `Photo:` This step allows users to take one or more photos as part of a scenario.

OBS: By editing this step, the user can select or unselect the Multi-Photo capture option as well as the import from device feature.

7)  `Data Provider:` This step is meant to support the use case of augmenting data with resort to an external data provider service. To do so:

- Click the **Edit** button
- Fill in the **URL** of the external data provider service
- Configure the **Request** properties
- Configure the **Response** properties
- Click the **Save** button

8)  `Upload:` This is a mandatory step which allows users to select an additional repository for uploading the extracted data and images of a given scenario. To do so:

- Click the **Edit** button
- Click the **Connection** drop-down list
- Select any available Connection
- Configure said Connection
- Specify the Properties data (if necessary)

##### Properties

This scenario element displays to the user, in an easy to comprehend format, the correlation between all Steps and Properties of a scenario. To do so:

- Enter a given scenario
- Click the **Properties**

##### Uploads

This scenario element shows a history of every upload. The user can see at a glance, the date and hour of the upload, the number of attachments and the status of the upload. To do so:

- Enter a given scenario
- Click the **Uploads**

OBS: If the Upload Step is configured for uploading into an external repository (i.e DataCap or IBM navigator), the content of each upload will be accessible only inside said repository.


# 3. <u>Inviting and managing users</u>


Click the **Team** button on the toolbar to access the team page. On this page, an administrator can invite and remove users, as well as managing their permissions.

## Inviting users

From the Team page, Click the **Invite User** button. Insert the email address of the person that you would like to invite using the following format [_user@example.com_](mailto:user@example.com)_._ And click the **Send Invitation** button. Please note that you can only invite one user at a time.

To remove a user, simply click the **Delete** button associated with the user and confirm the action in the dialog box.

OBS: When connected to an LDAP server, this features will not be available. These actions need to be performend and completed from within the LDAP server configuration.

## Assigning user roles

To assign a role to a user, click on the current role of a user and select a new one from the drop-down list.

OBS: When connected to an LDAP server, this feature will not be available. This action needs to be performend and completed from within the LDAP server configuration.

## User roles and permissions

| User Role | Permissions |
| --- | --- |
| App user | Users that can only login in the mobile apps. |
| Workspace User | Users that can login in the mobile apps as well as the Admin Console and have permissions to create scenarios and connections. |
| Workspace Admin | Users that also have the privilege to invite people to teams as well as create scenarios and connections. |
| Platform Admin | Users that also have the privilege to create new workspaces. |

OBS: When connected to an LDAP server, only the App User and Workspace Admin roles will be available.

# 4. <u>Integrations</u>


IBM Mobile Capture offers integrations with following file and data governance products.

## IBM® Content Navigator Integration

IBM® Content Navigator is an easy to use web client for content management. IBM Content Navigator provides users with a web console for working with content that is stored on IBM Content Manager, IBM Content Manager OnDemand, or IBM FileNet® Content Manager repositories.

How to set up an IBM® Content Navigator integration:

- Click on **Connections** button on the toolbar
- Go to **New Connection** and select **IBM Navigator**
- Insert a Connection name and the credentials associated with your repository
- Click the **Connect to Navigator** button
- Select the repository were you want the data and images to be allocated
- Click the **Save** button

OBS: The uploaded images in the IBM® Content Navigator will be available as a single PDF format file.

## IBM® Datacap integration

IBM® Datacap is a complete solution for data and documents, which helps you to streamline the capture, recognition and classification of business documents and extract important information.

How to set up an IBM® Datacap integration:

- Click on **Connections** button on the toolbar
- Go to **New Connection** and select **Datacap**
- Insert a Connection name and the credentials associated with your Datacap account
- Click the **Save** button



# 5. <u>Creating and managing other workspaces</u>

With IBM Automation Mobile Capture, a Platform Admin user can create or delete different workspaces within the same platform. To create a new workspace:

- Click the Workspaces button on the toolbar
- Click the **New Workspace** button
- Insert the Account name and Owner's Email ([_user@example.com_](mailto:user@example.com))_._
- Click the **Create Account** button

OBS: When connected to an LDAP server, this feature will not be available.

# 6. <u>LDAP integration</u>

IBM Automation Mobile Capture supports integration with an LDAP server. As of version 3.1.0, this feature can only be accessible via a Kubernets deployment. 


