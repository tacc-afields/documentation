.. _systems:

=======================================
Systems
=======================================
Once you are authorized to make calls to the various services, one of first
things you may want to do is view storage and execution resources available
to you or create your own. In Tapis a storage or execution resource is referred
to as a *system.*

-----------------
Overview
-----------------
A Tapis system represents a server or collection of servers exposed through a
single host name or IP address. Each system is associated with a specific tenant.
A system can be used for the following purposes:

* Running a job, including:

  * Staging files to an execution system in preparation for running a job.
  * Executing a job on an execution system.
  * Archiving files and data on a remote storage system after job execution.

* Storing and retrieving files and data.

Each system is of a specific type and owned by a specific user who has special
privileges for the system. The system definition also includes the user that is
used to access the system, referred to as *effectiveUserId.* This access user
can be a specific user (such as a service account) or dynamically specified as
``${apiUserId}`` in which case the user name is extracted from the identity
associated with the request to the service.

At a high level a system represents the following information:

Type of system
  LINUX or OBJECT_STORE
Owner
  A specific user set at system creation.
Host name or IP address.
  FQDN or IP address
Enabled flag
  Indicates if system is currently considered active and available for use.
  By default the system is enabled when first created.
Effective User
  The user name to use when accessing the system. Referred to as *effectiveUserId.*
  A specific user (such as a service account) or the dynamic user ``${apiUserId}``
Access method
  How access authorization is handled by default. Access method can also be
  specified as part of a request.
  Initially supported: PASSWORD, PKI_KEYS, ACCESS_KEY.
Effective root directory
  Directory to be used when listing files or moving files to and from the system.
Transfer methods
  Supported methods for moving files or objects to and from the system. Allowable entries are determined by the system
  type. Initially supported: SFTP, S3.
Various attributes related to job execution
  * Flag indicating if system can be used to run jobs.
  * List of job related capabilities supported by the system.
  * Job related directories: *LocalWorkingDir*, *LocalArchiveDir*, *RemoteArchiveSystem*, *RemoteArchiveDir*

--------------------------------
Getting Started
--------------------------------

Before going into further details about Systems, here we give some examples of how to create and view systems.
In the examples below we assume you are using the TACC tenant with a base URL of ``tacc.tapis.io`` and that you have
authenticated using PySDK or obtained an authorization token and stored it in the environment variable JWT,
or perhaps both.

Creating a System
~~~~~~~~~~~~~~~~~

Create a local file named ``system_s3.json`` with json similar to the following::

  {
    "name":"tacc-bucket-sample-<userid>",
    "description":"My Bucket",
    "host":"https://tapis-sample-test-<userid>.s3.us-east-1.amazonaws.com/",
    "systemType":"OBJECT_STORE",
    "defaultAccessMethod":"ACCESS_KEY",
    "effectiveUserId":"${owner}",
    "bucketName":"tapis-tacc-bucket-<userid>",
    "rootDir":"/",
    "jobCanExec": false,
    "transferMethods":["S3"],
    "accessCredential":
    {
      "accessKey":"***",
      "accessSecret":"***"
    }
  }

where <userid> is replaced with your user name, your S3 host name is updated appropriately and if desired you have
filled in your access key and secret. Note that credentials are stored in the Security Kernel and may also be set or
updated using a separate API call. However, only specific Tapis services are authorized to retrieve credentials.

Using PySDK:

.. code-block:: python

 import json
 from tapipy.tapis import Tapis
 t = Tapis(base_url='https://tacc.tapis.io', username='<userid>', password='************')
 with open('system_s3.json', 'r') as openfile:
     my_s3_system = json.load(openfile)
 t.systems.createSystem(**my_s3_system)

Using CURL::

   $ curl -X POST -H "content-type: application/json" -H "X-Tapis-Token: $JWT" https://tacc.tapis.io/v3/systems -d @system_s3.json

Viewing Systems
~~~~~~~~~~~~~~~

Retrieving details for a system
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To retrieve details for a specific system, such as the one above:

Using PySDK:

.. code-block:: python

 t.systems.getSystemByName(systemName='tacc-bucket-sample-<userid>')

Using CURL::

 $ curl -H "X-Tapis-Token: $JWT" https://tacc.tapis.io/v3/systems/tacc-bucket-sample-<userid>?pretty=true

The response should look similar to the following::

 {
  "result": {
     "id": 4,
     "tenant": "dev",
     "name": "tacc-bucket-sample-<userid>",
     "description": "My Bucket",
     "systemType": "OBJECT_STORE",
     "owner": "<userid>",
     "host": "https://tapis-sample-test-<userid>.s3.us-east-1.amazonaws.com/",
     "enabled": false,
     "effectiveUserId": "<userid>",
     "defaultAccessMethod": "ACCESS_KEY",
     "accessCredential": null,
     "bucketName": "tapis-tacc-bucket-<userid>",
     "rootDir": "/",
     "transferMethods": [
       "S3"
     ],
     "port": 0,
     "useProxy": false,
     "proxyHost": "",
     "proxyPort": 0,
     "jobCanExec": false,
     "jobLocalWorkingDir": null,
     "jobLocalArchiveDir": null,
     "jobRemoteArchiveSystem": null,
     "jobRemoteArchiveDir": null,
     "jobCapabilities": [],
     "tags": [],
     "notes": {},
     "deleted": false,
      "created": "2020-07-22T02:42:30.896Z",
      "updated": "2020-07-22T02:42:30.896Z"
    },
    "status": "success",
    "message": "TAPIS_FOUND System found: tacc-bucket-sample-<userid>",
    "version": "0.0.1"
  }
 }

Note that accessCredential is null. Only specific Tapis services are authorized to retrieve credentials.

Retrieving details for all systems
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To see the current list of systems that you are authorized to view:

Using PySDK:

.. code-block:: python

 t.systems.getSystems()

Using CURL::

 $ curl -H "X-Tapis-Token: $JWT" https://tacc.tapis.io/v3/systems?pretty=true

The response should look similar to the following (response truncated for brevity)::

 {
  "result": [
   {
    "id": 1,
    "tenant": "dev",
    "name": "KDevSystem1",
    "description": "Default system for DS",
    "systemType": "LINUX",
    "owner": "atestuser99",
    "host": "data.tacc.utexas.edu",
    "enabled": true,
    "effectiveUserId": "jsmith",
    "defaultAccessMethod": "PASSWORD",
    "accessCredential": null,
    "bucketName": "myBucket",
    "rootDir": "/dev/home",
    "transferMethods": [
      "SFTP",
      "S3"
    ],
    "port": 22,
    "useProxy": false,
    "proxyHost": "",
    "proxyPort": 1111,
    "jobCanExec": true,
    "jobLocalWorkingDir": "/home/testuser2",
    "jobLocalArchiveDir": "/archive/testuser2",
    "jobRemoteArchiveSystem": "FakeSystem",
    "jobRemoteArchiveDir": "/archive",
    "jobCapabilities": [
     {
      "id": 1,
      "systemid": 1,
      "category": "SCHEDULER",
      "name": "Type",
      "value": "Slurm",
      "created": "2020-06-19T15:10:43.306Z",
      "updated": "2020-06-19T15:10:43.306Z"
     },
     {
      "id": 2,
      "systemid": 1,
      "category": "SOFTWARE",
      "name": "MPI",
      "value": "",
      "created": "2020-06-19T15:10:43.306Z",
      "updated": "2020-06-19T15:10:43.306Z"
     },
     {
      "id": 3,
      "systemid": 1,
      "category": "JOB",
      "name": "MaxRunTime",
      "value": "24H",
      "created": "2020-06-19T15:10:43.306Z",
      "updated": "2020-06-19T15:10:43.306Z"
     }
     ],
     "tags": [
      "value1",
      "value2",
      "a",
      "a long tag with spaces and numbers (1 3 2) and special characters [_ $ - & * % @ + = ! ^ ? < > , . ( ) { } / \\ | ]. Backslashes must be escaped."
     ],
     "notes": {
      "jsonData": {
       "project": "myproject2",
       "testdata": "abc2"
      },
     "stringData": "{\"project\": \"myproject1\", \"testdata\": \"abc1\"}"
     },
     "deleted": false,
     "created": "2020-06-19T15:10:43.306Z",
     "updated": "2020-06-19T15:10:43.306Z"
   },
   {
    "id": 4,
    "tenant": "dev",
    "name": "tacc-bucket-sample-<userid>",
    "description": "My Bucket",
    "systemType": "OBJECT_STORE",
    ...
   },
   {
    "id": 2,
    "tenant": "dev",
    "name": "tapis-demo",
    "description": "AWS demo Bucket",
    "systemType": "OBJECT_STORE",
    ...
   }
  ],
  "status": "success",
  "message": "TAPIS_FOUND Systems found: 3 items",
  "version": "0.0.1"
 }

-----------------
Permissions
-----------------
At system creation time the owner is given full system authorization. If the effective
access user *effectiveUserId* is a specific user (such as a service account) then this
user is given the same authorizations. If the effective access user is the dynamic user
``${apiUserId}`` then the authorizations for each user must be granted and access
credentials created in separate API calls.
Permissions for a system may be granted and revoked through the systems API. Please
note that grants and revokes through this service only impact the default role for the
user. A user may still have access through permissions in another role. So even after
revoking permissions through this service when permissions are retrieved the access may
still be listed. This indicates access has been granted via another role.

Permissions are specified as either ``*`` for all permissions or some combination of the
following specific permissions: ``("READ","MODIFY")``. Specifying permissions in all
lower case is also allowed.

------------------
Access Credentials
------------------
At system creation time the access credentials may be specified if the effective
access user *effectiveUserId* is a specific user (such as a service account) and not
a dynamic user, i.e. ``${apiUserId}``. If the effective access user is dynamic then
access credentials for any user allowed to access the system must be registered in
separate API calls. Note that the systems service does not store credentials.
Credentials are persisted by the Security Kernel service and only specific Tapis services
are authorized to retrieve credentials.

-----------------
Capabilities
-----------------
Each System definition may contain a list of capabilities supported by that system.
An Application or Job definition may then specify required capabilities. These are
used for determining eligible systems for running an application or job.

-----------------
Deletion
-----------------
A system may be soft deleted. Soft deletion means the system is marked as deleted and
is no longer available for use. It will no longer show up in searches and operations on
the system will no longer be allowed. The system definition is retained for auditing
purposes. Note this means that system names may not be re-used after deletion.

------------------------
Table of Attributes
------------------------

.. Initial table - comment out
    +------------------------+----------------+------------------------------------+----------------------------------------------------------------------------------------+
    | Attribute              | Type           | Example                            | Description                                                                            |
    +========================+================+=========+==========================+========================================================================================+
    | tenant                 | String         | designsafe                         | Name of the tenant for which the system is defined\. Tenant \+ name must be unique\.   |
    +------------------------+----------------+------------------------------------+----------------------------------------------------------------------------------------+
    | name                   | String         | designsafe1\.storage\.default      | Name of the system.  URI safe, see RFC 3986. Tenant \+ name must be unique\.           |
    |                        |                |                                    | Allowed characters: Alphanumeric \[0\-9a\-zA\-Z\] and special characters \[\-\.\_~\]\. |
    +------------------------+----------------+------------------------------------+----------------------------------------------------------------------------------------+
    | description            | String         | Default storage system             | Description                                                                            |
    |                        |                | for designsafe\.                   |                                                                                        |
    +------------------------+----------------+------------------------------------+----------------------------------------------------------------------------------------+
    | systemType             | enum           | LINUX                              | Type of system\. Initially supported: LINUX, OBJECT_STORE                              |
    +------------------------+----------------+------------------------------------+----------------------------------------------------------------------------------------+
    | owner                  | String         | jdoe                               | User name of owner. Variable references: $\{apiUserId\}                                |
    +------------------------+----------------+------------------------------------+----------------------------------------------------------------------------------------+
    | host                   | String         | data\.tacc\.utexas\.edu            | Host name or ip address of the system                                                  |
    +------------------------+----------------+------------------------------------+----------------------------------------------------------------------------------------+
    | enabled                | boolean        | FALSE                              | Indicates if system is currently enabled for use\.                                     |
    +------------------------+----------------+------------------------------------+----------------------------------------------------------------------------------------+
    | effectiveUserId        | String         | tg869834                           |                                                                                        |
    +------------------------+----------------+------------------------------------+----------------------------------------------------------------------------------------+
    | defaultAccessMethod    | enum           | PKI\_KEYS                          |                                                                                        |
    +------------------------+----------------+------------------------------------+----------------------------------------------------------------------------------------+
    | accessCredential       | Credential     |                                    |                                                                                        |
    +------------------------+----------------+------------------------------------+----------------------------------------------------------------------------------------+
    | bucketName             | String         | tapis\-$\{tenant\}\-$\{apiUserId\} |                                                                                        |
    +------------------------+----------------+------------------------------------+----------------------------------------------------------------------------------------+
    | rootDir                | String         | HPC: $HOME\,  VM: /home/jdoe       |                                                                                        |
    |                        |                |                                    |                                                                                        |
    +------------------------+----------------+------------------------------------+----------------------------------------------------------------------------------------+
    | transferMethods        | \[enum\]       |                                    |                                                                                        |
    +------------------------+----------------+------------------------------------+----------------------------------------------------------------------------------------+
    | port                   | int            | 22                                 |                                                                                        |
    +------------------------+----------------+------------------------------------+----------------------------------------------------------------------------------------+
    | useProxy               | boolean        | true                               |                                                                                        |
    +------------------------+----------------+------------------------------------+----------------------------------------------------------------------------------------+
    | proxyHost              | String         |                                    |                                                                                        |
    +------------------------+----------------+------------------------------------+----------------------------------------------------------------------------------------+
    | proxyPort              | int            |                                    |                                                                                        |
    +------------------------+----------------+------------------------------------+----------------------------------------------------------------------------------------+
    | jobCanExec             | boolean        | true                               |                                                                                        |
    +------------------------+----------------+------------------------------------+----------------------------------------------------------------------------------------+
    | jobLocalWorkingDir     | String         | HPC: $SCRATCH\,  VM:/home/jdoe     |                                                                                        |
    +------------------------+----------------+------------------------------------+----------------------------------------------------------------------------------------+
    | jobLocalArchiveDir     | String         | /archive                           |                                                                                        |
    +------------------------+----------------+------------------------------------+----------------------------------------------------------------------------------------+
    | jobRemoteArchiveSystem | String         | work\.cloud\.corral                |                                                                                        |
    +------------------------+----------------+------------------------------------+----------------------------------------------------------------------------------------+
    | jobRemoteArchiveDir    | String         | HPC: / VM: /archive                |                                                                                        |
    +------------------------+----------------+------------------------------------+----------------------------------------------------------------------------------------+
    | jobCapabilities        | \[Capability\] |                                    |                                                                                        |
    +------------------------+----------------+------------------------------------+----------------------------------------------------------------------------------------+
    | tags                   | \[String\]     |                                    |                                                                                        |
    +------------------------+----------------+------------------------------------+----------------------------------------------------------------------------------------+
    | notes                  | String         | "\{\}"                             |                                                                                        |
    +------------------------+----------------+------------------------------------+----------------------------------------------------------------------------------------+
    | id                     | int            | 202881                             |                                                                                        |
    +------------------------+----------------+------------------------------------+----------------------------------------------------------------------------------------+
    | created                | Timestamp      |                                    |                                                                                        |
    +------------------------+----------------+------------------------------------+----------------------------------------------------------------------------------------+
    | updated                | Timestamp      |                                    |                                                                                        |
    +------------------------+----------------+------------------------------------+----------------------------------------------------------------------------------------+


+------------------------+--------------+----------------------+--------------------------------------------------------------------------------------+
| Attribute              | Type         | Example              | Notes                                                                                |
+========================+==============+======================+======================================================================================+
| tenant                 | String       | designsafe           | - Name of the tenant for which the system is defined.                                |
|                        |              |                      | - *tenant* + *name* must be unique.                                                  |
|                        |              |                      |                                                                                      |
+------------------------+--------------+----------------------+--------------------------------------------------------------------------------------+
| name                   | String       | ds1.storage.default  | - Name of the system. URI safe, see RFC 3986.                                        |
|                        |              |                      | - *tenant* + *name* must be unique.                                                  |
|                        |              |                      | - Allowed characters: Alphanumeric [0-9a-zA-Z] and special characters [-._~].        |
+------------------------+--------------+----------------------+--------------------------------------------------------------------------------------+
| description            | String       | Default storage      | - Description                                                                        |
+------------------------+--------------+----------------------+--------------------------------------------------------------------------------------+
| systemType             | enum         | LINUX                | - Type of system.                                                                    |
|                        |              |                      | - Initially supported: LINUX, OBJECT_STORE                                           |
|                        |              |                      |                                                                                      |
+------------------------+--------------+----------------------+--------------------------------------------------------------------------------------+
| owner                  | String       | jdoe                 | - User name of *owner*.                                                              |
|                        |              |                      | - Variable references: ${apiUserId}                                                  |
|                        |              |                      |                                                                                      |
+------------------------+--------------+----------------------+--------------------------------------------------------------------------------------+
| host                   | String       | data.tacc.utexas.edu | - Host name or ip address of the system                                              |
+------------------------+--------------+----------------------+--------------------------------------------------------------------------------------+
| enabled                | boolean      | FALSE                | - Indicates if system currently enabled for use.                                     |
+------------------------+--------------+----------------------+--------------------------------------------------------------------------------------+
| effectiveUserId        | String       | tg869834             | - User to use when accessing the system.                                             |
|                        |              |                      | - May be a static string or a variable reference.                                    |
|                        |              |                      | - Variable references: ${apiUserId}, ${owner}                                        |
|                        |              |                      | - On output variable reference will be resolved.                                     |
+------------------------+--------------+----------------------+--------------------------------------------------------------------------------------+
| defaultAccessMethod    | enum         | PKI_KEYS             | - How access authorization is handled by default.                                    |
|                        |              |                      | - Can be overridden as part of a request to get a system or credentials.             |
|                        |              |                      | - Initially supported: PASSWORD, PKI_KEYS, ACCESS_KEY                                |
+------------------------+--------------+----------------------+--------------------------------------------------------------------------------------+
| accessCredential       | Credential   |                      | - On input credentials to be stored in Security Kernel.                              |
|                        |              |                      | - *effectiveUserId* must be static, either a string constant or ${owner}.            |
|                        |              |                      | - May not be specified if *effectiveUserId* is dynamic, i.e. ${apiUserId}.           |
|                        |              |                      | - On output contains credentials for *effectiveUserId*.                              |
|                        |              |                      | - Returned credentials contain relevant information based on *systemType*.           |
+------------------------+--------------+----------------------+--------------------------------------------------------------------------------------+
| bucketName             | String       | tapis-ds1-jdoe       | - Name of bucket for OBJECT_STORAGE system.                                          |
|                        |              |                      | - Required if *systemType* is OBJECT_STORAGE.                                        |
|                        |              |                      | - Variable references: ${apiUserId}, ${owner}, ${tenant}                             |
+------------------------+--------------+----------------------+--------------------------------------------------------------------------------------+
| rootDir                | String       | $HOME                | - Required if *systemType* is LINUX. Must be an absolute path.                       |
|                        |              |                      | - Serves as effective root directory when listing or moving files.                   |
|                        |              |                      | - NOTE: Used for *jobLocalArchiveDir* but not for *jobLocalWorkingDir*.              |
|                        |              |                      | - Optional for an OBJECT_STORE system but may be used for a similar purpose.         |
|                        |              |                      | - Variable references: ${apiUserId}, ${owner}, ${tenant}                             |
+------------------------+--------------+----------------------+--------------------------------------------------------------------------------------+
| transferMethods        | [enum]       |                      | - Supported methods for moving files or objects to and from the system.              |
|                        |              |                      | - Allowable entries are determined by *systemType*.                                  |
|                        |              |                      | - Initially supported: SFTP, S3                                                      |
+------------------------+--------------+----------------------+--------------------------------------------------------------------------------------+
| port                   | int          | 22                   | - Port number used to access the system                                              |
+------------------------+--------------+----------------------+--------------------------------------------------------------------------------------+
| useProxy               | boolean      | TRUE                 | - Indicates if system should be accessed through a proxy.                            |
+------------------------+--------------+----------------------+--------------------------------------------------------------------------------------+
| proxyHost              | String       |                      | - Name of proxy host.                                                                |
+------------------------+--------------+----------------------+--------------------------------------------------------------------------------------+
| proxyPort              | int          |                      | - Port number for *proxyHost*.                                                       |
+------------------------+--------------+----------------------+--------------------------------------------------------------------------------------+
| jobCanExec             | boolean      |                      | - Indicates if this system will be used to execute jobs.                             |
+------------------------+--------------+----------------------+--------------------------------------------------------------------------------------+
| jobLocalWorkingDir     | String       | $SCRATCH             | - Parent directory local to execution system on which a job is run.                  |
|                        |              |                      | - Where inputs and application assets are staged.                                    |
|                        |              |                      | - Each job will use a separate sub-directory with a name based on the job ID.        |
|                        |              |                      | - Required if *jobCanExec* is true.                                                  |
|                        |              |                      | - Note that this path **IS NOT** relative to *rootDir*.                              |
|                        |              |                      | - Variable references: ${apiUserId}, ${owner}, ${tenant}                             |
+------------------------+--------------+----------------------+--------------------------------------------------------------------------------------+
| jobLocalArchiveDir     | String       | /archive             | - Parent directory local to execution system used for archiving job output files.    |
|                        |              |                      | - Each job will use a separate sub-directory with a name based on the job ID.        |
|                        |              |                      | - Job definition will specify which files to archive.                                |
|                        |              |                      | - Note that this path **IS** relative to *rootDir*.                                  |
|                        |              |                      | - Variable references: ${apiUserId}, ${owner}, ${tenant}                             |
+------------------------+--------------+----------------------+--------------------------------------------------------------------------------------+
| jobRemoteArchiveSystem | String       | work.cloud.corral    | - A system remote from execution system where job output files are to be archived.   |
+------------------------+--------------+----------------------+--------------------------------------------------------------------------------------+
| jobRemoteArchiveDir    | String       | /archive             | - Parent directory on the remote system used for archiving job output files.         |
|                        |              |                      | - Job definition will specify which files to archive.                                |
|                        |              |                      | - Each job will use a separate sub-directory with a name based on the job ID.        |
|                        |              |                      | - Required if *jobCanExec* is true and *jobRemoteArchiveSystem* is set               |
|                        |              |                      | - Note that this path **IS** relative to the target remote system's *rootDir*.       |
|                        |              |                      | - Variable references: ${apiUserId}, ${owner}, ${tenant}                             |
+------------------------+--------------+----------------------+--------------------------------------------------------------------------------------+
| jobCapabilities        | [Capability] |                      | - List of job related capabilities supported by the system.                          |
+------------------------+--------------+----------------------+--------------------------------------------------------------------------------------+
| tags                   | [String]     |                      | - List of tags as simple strings.                                                    |
+------------------------+--------------+----------------------+--------------------------------------------------------------------------------------+
| notes                  | String       | "{}"                 | - Simple metadata in the form of a Json object.                                      |
+------------------------+--------------+----------------------+--------------------------------------------------------------------------------------+
| id                     | int          | 20281                | - Auto-generated by service.                                                         |
+------------------------+--------------+----------------------+--------------------------------------------------------------------------------------+
| created                | Timestamp    | 2020-06-19T15:10:43Z | - When the system was created. Maintained by service.                                |
+------------------------+--------------+----------------------+--------------------------------------------------------------------------------------+
| updated                | Timestamp    | 2020-07-04T23:21:22Z | - When the system was last updated. Maintained by service.                           |
+------------------------+--------------+----------------------+--------------------------------------------------------------------------------------+


Heading 2
~~~~~~~~~

Heading 3
^^^^^^^^^

