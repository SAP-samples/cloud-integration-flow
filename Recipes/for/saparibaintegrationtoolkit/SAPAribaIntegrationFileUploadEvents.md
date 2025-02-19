SAP Ariba Integration File Upload Events

# Table of Contents
[Table of Contents	1](#_Toc107414733)

[Overview	2](#_Toc107414734)

[Prerequisites	2](#_Toc107414735)

[Uploading a Master Data File	2](#_Toc107414736)

[Security	3](#_Toc107414737)

[Data center value mappings	3](#_Toc107414738)

[Upload Strategies	4](#_Toc107414739)

[Scheduling	4](#_Toc107414740)

[Failure and Recovery	4](#_Toc107414741)

[Parameter Reference	4](#_Toc107414742)

[Sender - SFTPSender	4](#_Toc107414743)

[Receiver - SFTP	8](#_Toc107414744)

[Receiver – EmailReceiver	8](#_Toc107414745)

[Receiver – SAPAriba	11](#_Toc107414746)

[More – All Parameters	12](#_Toc107414747)

[Helpful tips	14](#_Toc107414748)

[Troubleshooting	14](#_Toc107414749)




# Overview
SAP Ariba ITK package includes iFlows “SAP Ariba Integration File Upload Events – Buying” and “SAP Ariba Integration File Upload Events – Sourcing”, which are used to transfer data in batch mode between your backend system and your Ariba service. Data is transferred in the form of comma-separated-value (CSV) files, or the CSV files packaged in a compressed ZIP archive. The underlying protocol used by the iFlow is an HTTP-SOAP.

This document:

- Describes the information needed to transfer data to your Ariba service
- Provides reference information for each field in the iFlow parameters
- Provides an inventory of error codes and messages the response can return, along with tools that you can use to troubleshoot the integration

# Prerequisites
Before working with this document, make sure you have met the following prerequisites:

- You have an SFTP (Secure File Transfer Protocol) server available 
- You have an email server available 
- You know the site ID and datacenter location of your Ariba realm. Realm enablement team or Ariba deployment team can provide you with this ID
- You have configured the integration password. Follow these steps in Ariba to complete the configuration:
  - Log in to your site.
  - Choose Manage > Core Administration (for Buying), or Manage > Administration (for Sourcing)
  - In the left menu, choose Integration Administration > Integration Toolkit Security.
  - Enter the password twice.
  - Select Save. The integration password is now set, and data transfers using the Data Transfer Tool, or the other protocols described in this document are now enabled.
- You are familiar with the files required to load master data, you have a batch data file available, and you have uploaded this file by using the Data Import/Export tasks in Core Administration/Administration.
- You are familiar with the Ariba integration, import, and administration guides.

This document focuses on setting up data transfer iFlow. It does not provide information on master data itself. For details on what files should be uploaded, please refer to the guides listed above.

# Uploading a Master Data File
Uploading the master data file requires you to correctly fill out the parameters in the iFlow and schedule/submit the post to the correct URL. This section provides basic information on the protocol and gives examples of uploads.

The tool uses HTTP post to transfer data files to the Ariba system. The post contains the following information:

- The name of the batch integration event to run
- How to process the master data (that is import operation, Load, Create, Deactivate etc.)
- Information on realm and shared secret
- The file to upload 

# Security
This section describes how to use shared secret-based authentication.

SAP Ariba:

You can get the shared secret configured by your Administrator from the Integration Toolkit Security page in Ariba Administrator.

1. In Ariba Administrator, under Integration Manager workspace, click Integration Toolkit Security workspace
1. Under Select the Authentication Method, select Shared Secret from the pull-down menu
1. Under Shared Secret, enter the shared secret twice to confirm it
1. Click Save

SAP Integration Suite, Cloud Integration:

1. In Monitor, under Manage Security section, click Security Material
1. Under Create, select Secure Parameter
1. Create a new Secure Parameter with descriptive name such as “AribaBuyingSharedSecretTest” or “AribaBuyingSharedSecretProd”, enter the shared secret twice to Secure Parameter and Repeat Secure Parameter
1. Click Deploy

Note: Upload and download iFlows need to use the same shared secret. Please use secure value.

# Data center value mappings
One of the artifacts in the SAP Ariba Integration toolkit package is the SAP Ariba Datacenter Value mappings. This is a key component for the flow to work and **must be deployed!** It contains a standard list of datacenters which are mapped to the integration URL equivalent. You will use one of these datacenters when configuring your iflows. If it happens that your SAP Ariba realm is hosted on a different datacenter than the ones listed below, then please use the line marked as “other”, update the integration path next to it and redeploy the mapping.

![](Aspose.Words.d2b564e4-7e5d-4669-9d2c-40cc4957341e.001.png)

Note: no additional lines are supported in this mapping, for any nonstandard datacenters, please use the line “other”, the url must end with /

# Upload Strategies
There are additional considerations that need testing before deploying the iFlow in production.
## Scheduling
The scheduling of the upload should alternate incremental and full loads. A typical schedule loads daily incremental updates of master data and refreshes the set of master data weekly.

The latter is required because the incremental updates capture only additions and modifications to the existing set, not deletions (deactivations).
## Failure and Recovery
The return codes provide information on the type of failure and suggest possible reactions from the system. Here is an overview of three types of return codes:

- 200: Successful upload.
- 4xx: Permanent failure. The request failed, and either the request or the server must be modified before you resend this request. A typical cause is a mismatch in the integration password. To fix the mismatch, you must either set the password on the server or change the password on the client to match the password on the server.
- 5xx: Temporary failure. The server cannot process this request at this moment. The client should retry until the server can process the request. There are a number of reasons for this to happen, for example, several clients are uploading files concurrently, a node is in the process of upgrading itself, or an upgrade of the entire system is in progress.

# Parameter Reference
The following table provides reference information for each parameter of the upload, including the parameter name and type, whether it is mandatory, and the possible values it can be set to. 


### Sender - SFTPSender
- The setup is the same for Buying and Sourcing:

![](Aspose.Words.d2b564e4-7e5d-4669-9d2c-40cc4957341e.002.png)

|**Parameter**|**Example Value and Notes**|
| :- | :- |
|<p>**Directory**</p><p></p>|<p>e.g. indir</p><p></p><p>Use to specify a directory of your choice on SFTP server. This directory can contain either contain ZIP files (with CSVs inside, each of the filenames following the integration tracker for import batch data) or plain CSVs (with filenames following the integration tracker for import batch data). In case of import requisition task, the content is a double zip file, following the SAP Ariba guidelines.  </p>|
|**upload\_file\_type**|csv or zip – specify, if the files you’re storing on SFTP server, are plain csv files, or already zipped for a batch import, please note that one flow can read only one type of file|
|**Address**|Host name or IP address of the SFTP server and an optional port, for example, wdfd00213123:22.|
|**Proxy Type**|<p>Proxy type that you are using to connect to the target system.</p><p>- Select Internet if you are connecting directly to the SFTP server.</p><p>- Select On-Premise if you are connecting to an on-premise SFTP server.</p><p></p>|
|<p>**Location ID**</p><p>(only if **On-Premise** is selected for **Proxy Type**)</p>|To connect to an SAP Cloud Connector instance associated with your account, enter the location ID that you defined for this instance in the destination configuration of SAP BTP cockpit.|
|**Authentication**|<p>Authentication option for the connection to the SFTP server. You've the following options:</p><p>- Public Key</p><p>SFTP server authenticates the calling component (tenant) based on a public key.</p><p>- User Name/Password</p><p>SFTP server authenticates the calling component (tenant) based on the user name and password. To make this configuration setting work, you need to define the user name and password in a User Credential artifact and deploy the artifact on the tenant.</p><p>- Dual</p><p>SFTP server authenticates the calling component (tenant) with two authentication methods: based on a public key and based on user credentials.</p><p>At runtime, the system evaluates the values of additional parameters in the following way:</p><p>- For the authentication step based on user credentials: Credentials from the deployed artifact with the name given by the Credential Name parameter are evaluated by the system to authenticate the tenant against the SFTP server.</p><p>- For the authentication step based on public key: User name contained in the deployed artifact with name given by the Credential Name parameter and the key identified by the Private Key Alias parameter are evaluated by the system to authenticate the tenant against the SFTP server.</p><p>If selected, you can specify the User Credentials artifact (that contains user name and password) with the Credential Name parameter and the key to be used from the keystore with the Private Key Alias parameter.</p><p></p>|
|<p>**Credential Name**</p><p>(only if **User Name/Password** or **Dual** is selected for **Authentication**)</p><p></p>|Name of the deployed **User Credentials** artifact that contains the user name and password.|
|<p>**User Name**</p><p>(only if **Public Key** is selected for **Authentication**)</p><p></p>|<p>ID of the user performing the file transfer.</p><p>Make sure that the user name contains no other characters than A-z, 0-9, \_ (underscore), - (hyphen), / (slash), ? (question mark), @ (at), ! (exclamation mark), $ (dollar sign ), ' (apostrophe), (, ) (brackets), \* (asterisk), + (plus sign), , (comma), ; (semicolon), = (equality sign), . (dot), or ~ (tilde). Otherwise, an attempt for anonymous login is made which results in an error.</p><p></p>|
|<p>**Private Key Alias**</p><p>(only if **Public Key** or **Dual** is selected for **Authentication**)</p><p></p>|Alias to identify the private key in the keystore used for the communication with the SFTP server.|
|**Post-processing**|<p>Allows you to specify how files are to be handled after processing.</p><p>Note that only successfully processed messages can be post-processed. If message processing fails, the **Post-Processing** settings are not effective. Our recommendation is the deletion of the file, as the processed or failed file will be reuploaded to the outdir or logdir folders respectively. </p><p>You can select one of the following options:</p><p>- **Delete File** (default): The file is deleted after it has been processed successfully.</p><p>- **Keep File and Mark as Processed in Idempotent Repository**: Prevents a file from being consumed twice. For that purpose, an idempotent repository is activated. – not supported for this iFlow</p><p>- **Keep File and Process Again**: The file is kept on the SFTP server and file processing is repeated. You can use this option for testing purposes, for example.</p><p>- **Move File**: The file is moved to another directory. – not supported for this iFlow</p>|


|**Scheduler Option**|**Field**|**Description**|
| :- | :- | :- |
|**Schedule on Day**|On Date|Specify the date on which you want the operation to be executed.|
||At Time|Specify the time at which you want the operation to be executed.|
||Every|Specify a time period (for example, every hour) in a dedicated time window.|
||Time Zone|Select the time zone that you want the scheduler to use as a reference for the date and time settings.|
|**Schedule to Recur**|Daily|Select the time or interval and time zone for the schedule to recur.|
||Weekly|Select the checkboxes to indicate the days of the week on which the operation has to be executed. Also, specify the time or interval for the schedule to recur.|
||Monthly|Select the day of the month on which the operation has to be executed. Also indicate the time or the interval for the schedule to recur.|

The recommendation is to setup the upload once per day, outside the business hours. You should also make sure that there is enough time between the load of files into the SFTP from your backend system and the BTP upload time, so that the system doesn’t try to upload unfinished content. If more files are present in the folder, the flow will pick them one by one.

For more info, refer also to SAP Cloud Integration guide: <https://help.sap.com/docs/CLOUD_INTEGRATION/368c481cd6954bdfa5d0435479fd4eaf/2de9ee58737247969eb7dc9e68b1b121.html>

### Receiver - SFTP
As above, configurations are duplicated. The processed files are stored in the folder outdir on your SFTP, and the failed files with logs are stored in the logdir directory on your SFTP.

### Receiver – EmailReceiver
The mail adapter allows you to connect the tenant to an email server. The sender mail adapter can download e-mails and access the e-mail body content as well as attachments. The receiver mail adapter allows you to send encrypted messages by e-mail.

![](Aspose.Words.d2b564e4-7e5d-4669-9d2c-40cc4957341e.003.png)


|**Parameter**|**Example Value and Notes**|
| :- | :- |
|<p>**Address**</p><p></p>|<p>Specifies the host name and (optionally) a port number of the SMTP server.</p><p>An external mail server can be used.</p><p>Use one of the following open ports for external mail servers:</p><p>- 587 for SMTP+STARTTLS</p><p>- 465 for SMTPS</p><p>Example (to connect to Yahoo mail server):</p><p>smtp.mail.yahoo.com:465</p>|
|**Proxy Type**|<p>The type of proxy that you’re using to connect to the target system.</p><p>- Select **Internet** if you’re connecting to a cloud mail server.</p><p>- Select **On-Premise** if you’re connecting to an on-premise mail server.</p>|
|**Location ID** (only if **On-Premise** is selected for **Proxy Type**)|To connect to a cloud connector instance associated with your account, enter the location ID that you defined for this instance in the destination configuration on the cloud side.|
|**Timeout (in ms)**|<p>Specifies the network timeout for the connection attempt to the server.</p><p>The default value is 30000. The timeout must be larger than 0, but less than five minutes.</p><p></p>|
|**Protection** |<p>Defines whether encryption is used. The possible values are:</p><p>- **Off**</p><p>No encryption is initiated, whether the server requests it or not.</p><p>- **STARTTLS Mandatory**</p><p>If the server supports STARTTLS, the client initiates encryption using TLS. If the server doesn’t support this option, the connection fails.</p><p>- **STARTTLS Optional**</p><p>If the server supports the STARTTLS command, the connection is upgraded to Transport Layer Security encryption (works with the normal port 25).</p><p>If the server supports STARTTLS, the client initiates encryption using TLS. If the server doesn’t support this option, client and server remain connected but communicate without encryption.</p><p>- **SMTPS** (only when **None** has been selected for **Proxy Type**)</p><p>The TCP connection to the server is encrypted using SSL/TLS. This step usually requires an SSL proxy on the server side and access to the port it runs on.</p><p></p>|
|**Authentication**|<p>Specifies which mechanism is used to authenticate against the server with a user name and password combination. Possible values are:</p><p>- **None**</p><p>No authentication is attempted. No credential can be chosen.</p><p>- **Plain User Name/Password**</p><p>The user name and password are sent in plain text. Only use this option together with SSL or TLS, as otherwise an attacker could obtain the password.</p><p>- **Encrypted User/Password**</p><p>The user name and password are hashed before being sent to the server. This authentication mechanism (CRAM-MD5 and DIGEST-MD5) is secure even without encryption.</p><p>- **OAuth2 Authorization Code**</p><p>The authentification is done via an authorization server as an intermediary step. The client can exchange the OAuth2 Authorization Code for an access token. Your user credentials are never shared with the client.</p><p></p>|
|<p>**Credential Name**</p><p>(only if **Plain User Name/Password** or **Encrypted User/Passwor**d is selected for **Authentication**)</p>|Specifies the name of a deployed user credential to use for authentication.|
|**From**|E-mail address that the message comes from.|
|**To**|<p>E-mail address that the message is sent to.</p><p>If you want to configure multiple mail receivers, use a comma (,) to separate the addresses.</p><p>Example: name1@example.com, name2@example.com, name3@example.com</p><p></p>|
|<p>**sftp\_address**</p><p></p>|<p>Specifies the host name and (optionally) a port number of the SMTP server.</p><p>Duplicated from above configuration.</p>|

Each successfully ran or failed iflow will send a respective message to the chosen list of email addresses. In case of successful run the message is “Your integration flow was ran successfully.” Including the response from SAP Ariba.

In the case of failed flow you will get details of the error. This includes the latest payload in a form of attachment (if you reached Ariba then this contains Ariba response), short description of the exception, the latest message in the body of your flow, which would contain Ariba response if you reached it and a complete stacktrace of the exception. The same information is also stored in your sftp server in the logdir directory. We would suggest to enable monitoring on your iflow with a trace level, rerun the flow with the failed file (which is stored in the logdirectory with the same name as the logtrace file) and review the logs for this specific run.


For more info, refer also to <https://help.sap.com/docs/CLOUD_INTEGRATION/368c481cd6954bdfa5d0435479fd4eaf/f68d5e03fd574f509f89474f6a6e272a.html> 

### Receiver – SAPAriba
![](Aspose.Words.d2b564e4-7e5d-4669-9d2c-40cc4957341e.004.png)

|**Parameter**|**Example Value and Notes**|
| :- | :- |
|<p>**realm**</p><p></p>|<p>Site ID to which to direct HTTP requests </p><p>Use the actual realm ID of your site (Buying site for Buying iFlow, Sourcing one for Sourcing iFlow)</p>|
|**Timeout (in ms)**|<p>Specifies the network timeout for the connection attempt to the server.</p><p>The default value is 60000. The timeout must be larger than 0, but less than five minutes.</p><p>Bigger files might need bigger timeout.</p>|

### More – All Parameters
![](Aspose.Words.d2b564e4-7e5d-4669-9d2c-40cc4957341e.005.png)

|**Parameter**|**Example Value and Notes**|
| :- | :- |
|<p>**datacenter**</p><p></p>|<p>Datacenter of your Ariba realm(s). Value has to match input in the Value mappings</p><p>e.g. us, eu, cn, other…</p>|
|**event**|<p>The name of the integration event to run. This event must be a batch event. This is an event that will run other events with the files it finds on SFTP.</p><p>Batch events preconfigured for Ariba Buying are:</p><p>- Import Batch Data</p><p>- Import Remittance Data</p><p>- Import Requisitions</p><p>- Import Budgets</p><p>- Load Budget Adjustments</p><p>The batch event preconfigured for Ariba Sourcing is:</p><p>- Import Master Batch Data</p>|
|**operation**|<p>For each integration task in a batch, a default operation is defined that can be any of the following:</p><p>- Create</p><p>Creates a ClusterRoot object. If the object exists in the database prior to this operation, a non-fatal exception is inserted in the integration event log entry, and the object is not created.</p><p>- Delete</p><p>Deactivates a Cluster Root object. It does nothing, if the object does not exist in the database or is already deactivated.</p><p>- Update</p><p>Updates an existing ClusterRoot object. If the object does not exist in the database, a non-fatal exception is inserted in the integration event entry, and the object is not updated.</p><p>- Load</p><p>Is the combination of Create and Update. Specifically, if the object does not already exist, it is created. Otherwise, it is updated.</p><p>- Update And Delete</p><p>Like Update, updates records in the database if the records are there in the file and also there in the database. This option also does the following:</p><p>- Deletes records from the Ariba database that have the same EventSource label as the integration event and are not there in the file.</p><p>&emsp;- Does not create new elements, even if they exist in the data source file.</p><p>- Load And Delete</p><p>&emsp;- Creates records in the database if the records are there in the file and not there in the database.</p><p>&emsp;- Updates records in the database if the records are there in the file and also there in the database.</p><p>&emsp;- Deletes records in the database if the records are not there in the file. However, it will only delete records in the database, where their adapter source matches the event source of the Load and Delete event being run.</p><p>- Update Elements Only</p><p>Updates the contents of vector fields. Mostly used in header detail imports. Specifically, for a given vector field, if the vector contains the object, the object is updated. Otherwise, the updated object is inserted into the vector.</p><p>- Delete Elements Only</p><p>Deletes elements from vector fields. Mostly used in header/detail imports. Specifically, removes the object from the vector field if the vector contains the object.</p>|
|**realm**|<p>Site ID to which to direct HTTP requests </p><p>Use the actual realm ID of your site (Buying site for Buying iFlow, Sourcing one for Sourcing iFlow). </p><p>Duplicated from above configuration.</p>|
|**sftp\_address**|<p>Specifies the host name and (optionally) a port number of the SMTP server.</p><p>Duplicated from above configuration.</p>|
|**sharedsecretpath**||
|**upload\_file\_type**|<p>csv or zip – specify, if the files you’re storing on SFTP server, are plain csv files, or already zipped for a batch import</p><p>Duplicated from above configuration</p>|

# Helpful tips

- User credentials for SFTP and mail server as well as secure parameters are created under the monitor section – manage security – security material
- You can test your SFTP and email connection credentials in the same section in the tile connectivity tests, use them in your flows only if the test passed
- Please follow the SAP Ariba documentation in order to use the proper file names of your uploaded content, SAP Ariba will not accept files with unknown names
- Buying and sourcing flows are not interchangeable so please make sure to use the correct combination of flow and solution

# Troubleshooting
In case of correctly setup flow, in case of any issues you will get an email as well as log stored into your SFTP server into logdir directory. This includes the latest payload in a form of attachment (if you reached Ariba then this contains Ariba response), short description of the exception, the latest message in the body of your flow, which would contain Ariba response if you reached it and a complete stacktrace of the exception. The same information is also stored in your sftp server in the logdir directory. We would suggest to enable monitoring on your iflow with a trace level, rerun the flow with the failed file (which is stored in the logdirectory with the same name as the logtrace file) and review the logs for this specific run.

Typical issues for file upload:

- Wrong shared secret or shared secret not enabled on SAP Ariba site, SAP Ariba will throw an authentication issue in its response
- Secure Parameter not available – the shared secret was not setup following the guidelines above
- Illegal filename – SAP Ariba will refuse to process files if they contain unknown filenames, please correct them using the integration tracker or SAP Ariba documentation
- Timeout exception – if you file is too large or your servers too slow, you might need to update timeouts set
- Incorrect event name or operation – in case that you were changing these values, please make sure to use the correct ones according to the latest SAP Ariba documentation
- SFTP issues – please review the documentation of the SFTP connector
- Email issues – please review the documentation of the Email connector
- For Full Load opeartion or 'Load and Delete' operation while running the Import Batch Data task, an additional parameter 'fullload' needs to be sent in the request to Ariba. Eg. 
  ```
  <Parameter name="fullload"><value>true</value></Parameter>
  ```

