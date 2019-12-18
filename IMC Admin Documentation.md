# Overview


IBM Mobile Capture is a platform which allows users to scan and capture data from a variety of documents. By simply using a mobile device, users can capture, edit, and upload the data in a repository of their choice in a matter of seconds.

## Capabilities of IBM Mobile Capture

- `Photo Capturing` - A feature that presentes the user with a simple camera screen which allows him/her to take a picture.

- `US Driver's Licence scanning and data extraction` - This feature allows the user to scan the front and the back of a driver's license, issued by the US's Department of Motor Vehicles, and capture a variety of properties. The document must be compliant with AAMVA DL/ID 2006 specification.

- `Passport scanning and data extraction` - This feature allows the scanning of any government issued Machine Readable Passport or other TD3 sized Machine Readable Travel Document, using the camera of the device. The document must be fully compliant with Part 4 of ICAO Doc 9303.

- `Barcode scanning and data extraction` - This feature offers the possibility in the scanning of the following types of barcode: Aztec, Code 39, Code 39 with checksum, full ASCII Code 39, full ASCII Code 39 with checksum, Code 93 Code 93i, Code 128, Data Matrix, EAN-8, EAN-13, Interleaved 2 of 5 (ITF), Interleaved 2 of 5 (ITF) with checksum, ITF-14, PDF417, QR and UPC-E.

- `Property editor` - This feature allows the user to visualize, edit and add data to any available Scenario properties.

- `Offline Functionality` - By using this feature, the users can perform all major features of the platform, except the upload functionality, in an offline status.

- `Smart capturing` - This feature consists of a set of functionalities designed to help users obtain the best possible image capture and data extraction.

- `Import from device` -  A feature that allows users to perform data extraction processes on images and PDF documents imported locally from a mobile device.

- `Smart filters` - An easy to use feature that grants users the abiity to apply a variety of filters to their captured images in order to improve image quality and OCR recognition.

## Supported browsers for the Admin Console

- Google Chrome
- Safari
- Mozilla Firefox


# Accessing and managing the Admin Console


## About the Admin Console

The Admin Console represents a Web browser-based, graphical user interface application that is used to manage an IBM Mobile Capture workspace by:

- Creating and managing scenarios
- Adding or removing external integrations
- Inviting and managing users
- Creating and managing other workspaces

## Accessing the Admin Console

To begin using the Admin Console, start one of the supported Web browsers for your local environment and access the URL provided to you at the moment of your IBM Mobile Capture user registration.

- Insert the correct credentials in the **Email** and **Password** fields
- Click the **Login** button.

You can use the **Remember me** checkbox for easier access in case of repeated workspace use.

OBS: In case of a forgotten password, click the **Forgot your Password?** link and follow the indicated instructions to reset your password.


# Creating and managing scenarios


## What is a scenario?

A scenario represents an assembly of steps, customly added together, to facilitate a document capture and data extraction, as well as their upload into a selected repository.

## Creating a scenario

Once you have managed to successfully login into the Admin Console, access the scenarios page by clicking the **Scenarios** button on the toolbar, this should be your default page when logged in.

- Click the **New Scenario** button
- Insert a given scenario name
- Click the **Create Scenario** button.

OBS: Note that in order to create a new scenario, the scenario name field can not be left empty.

## Configuring a Scenario

A scenario is composed out of the following elements: `Scenario name`, `Scenario steps`, `Properties`, and `Uploads`.

#### Scenario name

The name of a scenario can be set when creating the said scenario or by editing it.

- Access a scenario
- Edit the **Name** field

#### Scenario steps

The most configurable part of a scenario are its steps, which represent a selection of elements configured and arranged in a specific order so they can capture, validate and upload data from different types of documents.

To add one or multiple steps:

- Enter a given scenario
- Click the **Add Step** button
- Select an available step from the drop-down list

You can **Edit** or **Delete** a step by clicking its corresponding buttons.

##### Type of steps:

1) <u>_US Driver's License:_</u>  This step allows users to scan and capture data from a US Driver's License. Editing this step will enable users to precisely select what data parameters they wish to extract from the document. These parameters also represent the Scenario Properties in the IBM Mobile Capture platform.

2) <u>_Passport:_</u> This step allows users to scan and capture data from a Passport or a Machine Readable Travel Document. Editing this step will also enable users to select what data parameters they wish to extract from this type of documents.

3) <u>_Document:_</u> This step allows users to create a step for scanning a custom document and extracting data from custom fields. To do so:

- Click the **Edit** button
- Click the **Choose a file** button to upload a copy of the document you would like to scan (in PNG or PDF format) and click **Upload**
- Create the areas you would like to extract data from, by dragging the cursor on the uploaded image
- Fill in the **Zones** field/s with their corresponding name. These Zones also represent the Scenario Properties of this step
- Click the **Save** button

4) <u>_Property Editor:_</u> This step allows users to verify and edit the extracted data of a scenario as well as manually adding data before uploading it to a given repository. To do so:

- Click the **Edit** button
- Click the **Add Property** button
- Select any available Property from the drop-down list
- Click the **Save** button

OBS: For adding a custom property select the **New Property** option and customize its name.

5)  <u>_Barcode:_</u> This step allows users the possibility of scanning a multitude of barcode formats.

OBS: This step has no editing options.

6)  <u>_Photo:_</u> This step allows users to take one or more photos as part of a scenario.

OBS: By editing this step, the user can select or unselect the Multi-Photo capture option

7)  <u>_Data Provider:_</u> This step is meant to support the use case of augmenting data with resort to an external data provider service. 

8)  <u>_Upload:_</u> This is a mandatory step which allows users to select an additional repository for uploading the extracted data and images of a given scenario. To do so:

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


# Inviting and managing users


Click the **Team** button on the toolbar to access the team page. On this page, an administrator can invite and remove users, as well as managing their permissions.

## Inviting users

From the Team page, Click the **Invite User** button. Insert the email address of the person that you would like to invite using the following format [_user@example.com_](mailto:user@example.com)_._ And click the **Send Invitation** button.

OBS: Note that you can only invite one user at a time.

To remove a user, simply click the **Delete** button associated with the user and confirm the action in the dialog box.

## Assigning user roles

To assign a role to a user, click on the current role of a user and select a new one from the drop-down list.

## User roles and permissions

| User Role | Permissions |
| --- | --- |
| App user | Users that can only login in the mobile app. |
| Workspace User | Users that can login in the Admin Console and have permissions to create scenarios and connections. |
| Workspace Admin | Users that can login in the Admin Console, invite people to teams as well as create scenarios and connections. |
| Platform Admin | Users that can login in the Admin Console, invite people to teams as well as create scenarios, connections and workspaces. |


# Integrations


IBM Mobile Capture offers integrations with following file and data governance products

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



##  Webhook integration

IBM Mobile Capture

How to set up a Webhook integration:

- Click on **Connections** button on the toolbar
- Go to **New Connection** and select **Webhook**
- Insert a Connection name and the additional webhook details
- Click the **Save** button



## Creating and managing other workspaces

With IBM Mobile Capture, a Workspace Admin can create or delete different workspaces within the same platform. To create a new workspace:

- Click the Workspaces button on the toolbar
- Click the **New Workspace** button
- Insert the Account name and Owner's Email ([_user@example.com_](mailto:user@example.com)_._)
- Click the Create Account button
