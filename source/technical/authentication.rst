.. _authentication:

==========================
Tenancy and Authentication
==========================


Tenants
-------

Tapis is a *multi-tenant* platform, meaning that different projects (or *tenants*) can have logically isolated views of
the Tapis objects (i.e., the systems, files, actors, etc.) they create for their project. 

Each tenant is made up of the following:

1. A base URL with which to access the tenant; by default, the base URL takes the form ``https://<tenant_id>.tapis.io``
   where ``tenant_id`` is a short, unique identifier for the tenant in the Tapis system. For example, 
   ``https://tacc.tapis.io`` is the base URL for the ``tacc`` tenant.
2. An *authenticator* providing the rules for who can authenticate in the tenant. 


To see the current list of tenants registered with Tapis, we can use the tenants API.

With PySDK:

.. code-block:: plaintext

 >>> t.tenants.list_tenants()


With CURL:

.. code-block:: plaintext

 $ curl https://tacc.tapis.io/v3/tenants


The response will look similar to the following (the response below is truncated for brevity):

.. code-block:: plaintext

 allowable_x_tenant_ids: ['tacc']
 authenticator: https://tacc.tapis.io/v3/oauth2
 base_url: https://tacc.tapis.io
 create_time: Thu, 02 Jul 2020 23:45:16 GMT
 description: Production tenant for all TACC users.
 is_owned_by_associate_site: False
 last_update_time: Thu, 02 Jul 2020 23:45:16 GMT
 owner: CICSupport@tacc.utexas.edu
 public_key: -----BEGIN PUBLIC KEY-----
 MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAz7rr5CsFM7rHMFs7uKIdcczn0uL4ebRMvH8pihrg1tW/fp5Q+5ktltoBTfIaVDrXGF4DiCuzLsuvTG5fGElKEPPcpNqaCzD8Y1v9r3tfkoPT3Bd5KbF9f6eIwrGERMTs1kv7665pliwehz91nAB9DMqqSyjyKY3tpSIaPKzJKUMsKJjPi9QAS167ylEBlr5PECG4slWLDAtSizoiA3fZ7fpngfNr4H6b2iQwRtPEV/EnSg1N3Oj1x8ktJPwbReKprHGiEDlqdyT6j58l/I+9ihR6ettkMVCq7Ho/bsIrwm5gP0PjJRvaD5Flsze7P4gQT37D1c5nbLR+K6/T0QTiyQIDAQAB
 -----END PUBLIC KEY-----
 security_kernel: https://tacc.tapis.io/v3/security
 service_ldap_connection_id: None
 tenant_id: tacc
 token_service: https://tacc.tapis.io/v3/tokens
 user_ldap_connection_id: tacc-all,
 
 allowable_x_tenant_ids: ['dev']
 authenticator: https://dev.tapis.io/v3/oauth2
 base_url: https://dev.tapis.io
 create_time: Fri, 19 Jun 2020 20:36:38 GMT
 description: The dev tenant.
 is_owned_by_associate_site: False
 last_update_time: Fri, 19 Jun 2020 20:36:38 GMT
 owner: CICSupport@tacc.utexas.edu
 public_key: -----BEGIN PUBLIC KEY-----

 . . .

Here we see the first two tenants registered in the Tapis framework, the ``tacc`` and ``dev`` tenants. 

In general, the rules for authentication vary from tenant to tenant within Tapis. For example, in the ``tacc`` tenant,
any user with a valid TACC account can authenticate and use the APIs. The ``dev`` tenant is a development sandbox with 
test accounts used by the core Tapis team.

This documentation focuses on the ``tacc`` tenant; however, much of what follows in the subsequent sections will be the
same regardless of the tenant you are using. 


Authentication
--------------

The default authenticator provided by the Tapis project is based on OAuth2, and this is the authentication mechanism
in place for the ``tacc`` tenant. The OAuth2-based authentication services are available via the  ``/v3/oauth2``
endpoints.

OAuth uses different *grant type flows* for generating tokens in different situations. We do not provide a comprehensive 
guide to OAuth2; for that, we refer the reader to the `OAuth2 docs <https://oauth.net/2/>`_. Instead, we provide a
guide to the two most common use cases for users: generating tokens for themselves using the *password* grant flow, and 
generating tokens on behalf of others in a web application using the *authorization code* grant flow.

In the PySDK examples that follow, we will tacitly assume the ``tapipy.tapis.Tapis`` object has been instantiated as the
Python object ``t``. There are several options in the ``Tapis`` constructor, but the basic options include ``base_url`` 
and ``username``, for example:

.. code-block:: plaintext

 >>> t = Tapis(base_url='https://tacc.tapis.io', username='jdoe')


Password Grant - Generating a Token For Yourself
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The simplest case is that you want to generate a Tapis OAuth token for yourself; to do this you can use the *password*
grant flow, providing your username and password.

Tapis v3 tries to make this process as easy as possible by providing a simplified version of the password grant flow
that does not require an OAuth client (see the :ref:`oauth-clients-label` section).

With PySDK:

.. code-block:: plaintext

 >>> t = Tapis(base_url='https://tacc.tapis.io', username='apitest', password='abcd123')
 >>> t.get_tokens()


With CURL:

.. code-block:: plaintext

 > curl -H "Content-type: application/json" -d '{"username": "apitest", "password": "abcde123", "grant_type": "password" }' \
 https://tacc.tapis.io/v3/oauth2/tokens

In the PySDK, the access token is a first-class Python object stored on the Tapis object (the ``t`` in the examples 
above). We can inspect it

.. code-block:: plaintext

 >>> t.access_token

 access_token: eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJqdGkiOiJmN2I5YjE5ZS02MDk5LTRmODItYTcyMi1iNjEwYzVkMGJhMGMiLCJpc3MiOiJodHRwczovL3RhY2MudGFwaXMuaW8vdjMvdG9rZW5zIiwic3ViIjoiYXBpdGVzdEB0YWNjIiwidGFwaXMvdGVuYW50X2lkIjoidGFjYyIsInRhcGlzL3Rva2VuX3R5cGUiOiJhY2Nlc3MiLCJ0YXBpcy9kZWxlZ2F0aW9uIjpmYWxzZSwidGFwaXMvZGVsZWdhdGlvbl9zdWIiOm51bGwsInRhcGlzL3VzZXJuYW1lIjoiYXBpdGVzdCIsInRhcGlzL2FjY291bnRfdHlwZSI6InVzZXIiLCJleHAiOjE1OTUwOTk0NTYsInRhcGlzL2NsaWVudF9pZCI6bnVsbCwidGFwaXMvZ3JhbnRfdHlwZSI6InBhc3N3b3JkIn0.alC8rRM-zNsHKcUiz3-tOJPaYtFksKb4Bit_aFE1HH_X_znnP2QkJaqc-xaRoMlQu26MN72TlJE0siIN3T38xXWBGDumHUYbvnNzT-7lk7AQU5MHSyCWx8IRDmTSbqmWOG8WBMCIV9Dh84mDd-X6eLJQ_cz1QqMAiI_cPgA9VVE22qDK3Lbz2pp9t0sm-l9XjE5y5Im8Y0B2p0ssPD0TjW20C5yngZ4-4jowDafboKlscog9ko-adrsVIjG_r-ccCUX3r8SVwQLypZFZAPKqbVzl8jt_mCi30W8AYwiaYGmH7INBbHI9hO7kwJNFMuSylejFhMslxgdzGlIAyXauwg
 claims: {'jti': 'f7b9b19e-6099-4f82-a722-b610c5d0ba0c', 'iss': 'https://tacc.tapis.io/v3/tokens', 'sub': 'apitest@tacc', 'tapis/tenant_id': 'tacc', 'tapis/token_type': 'access', 'tapis/delegation': False, 'tapis/delegation_sub': None, 'tapis/username': 'apitest', 'tapis/account_type': 'user', 'exp': 1595099456, 'tapis/client_id': None, 'tapis/grant_type': 'password'}
 expires_at: 2020-07-18 19:10:56+00:00
 expires_in: <function Tapis.set_access_token.<locals>._expires_in at 0x7f29e213c510>
 jti: f7b9b19e-6099-4f82-a722-b610c5d0ba0c
 original_ttl: 14400

What we see is that the ``access_token.access_token`` is a Python string representing a JSON Web Token (JWT_). 
JWTs are
cryptographically signed with the private key associated with the tenant, and anyone can validate the signature by
using the corresponding public key associated with the tenant (see Tenants section above). 
The public key for each tenant is available from the Tenants
API. The core Tapis services will validate the access token sent on a given API call using the public key associated with
the tenant to verify the JWT signature.


Using a Token
~~~~~~~~~~~~~

In order to use an access token in an API request to Tapis, pass the token in as the value of the ``X-Tapis-Token`` header.
The PySDK will automatically send the token via this header for you. 
In CURL examples used throughout this documentation, we assume the raw JWT string representing an access token (like the 
above) has been exported as a shell variable; i.e.,

.. code-block:: plaintext

 $ export JWT=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJqdGkiOiJmN2I5YjE5ZS02MDk5LTRmODItYTcyMi1iNjEwYzVkMGJhMGMiLCJpc3MiOiJodHRwczovL3RhY2MudGFwaXMuaW8vdjMvdG9rZW5zIiwic3ViIjoiYXBpdGVzdEB0YWNjIiwidGFwaXMvdGVuYW50X2lkIjoidGFjYyIsInRhcGlzL3Rva2VuX3R5cGUiOiJhY2Nlc3MiLCJ0YXBpcy9kZWxlZ2F0aW9uIjpmYWxzZSwidGFwaXMvZGVsZWdhdGlvbl9zdWIiOm51bGwsInRhcGlzL3VzZXJuYW1lIjoiYXBpdGVzdCIsInRhcGlzL2FjY291bnRfdHlwZSI6InVzZXIiLCJleHAiOjE1OTUwOTk0NTYsInRhcGlzL2NsaWVudF9pZCI6bnVsbCwidGFwaXMvZ3JhbnRfdHlwZSI6InBhc3N3b3JkIn0.alC8rRM-zNsHKcUiz3-tOJPaYtFksKb4Bit_aFE1HH_X_znnP2QkJaqc-xaRoMlQu26MN72TlJE0siIN3T38xXWBGDumHUYbvnNzT-7lk7AQU5MHSyCWx8IRDmTSbqmWOG8WBMCIV9Dh84mDd-X6eLJQ_cz1QqMAiI_cPgA9VVE22qDK3Lbz2pp9t0sm-l9XjE5y5Im8Y0B2p0ssPD0TjW20C5yngZ4-4jowDafboKlscog9ko-adrsVIjG_r-ccCUX3r8SVwQLypZFZAPKqbVzl8jt_mCi30W8AYwiaYGmH7INBbHI9hO7kwJNFMuSylejFhMslxgdzGlIAyXauwg

With that variable set, we can use the ``-H`` flag with curl to set the ``X-Tapis-Token`` header as follows:

.. code-block:: plaintext

 $ curl -H "X-Tapis-Token: $JWT" ....


Note also the *claims* associated with the access token. These claims provide information about the token, including the
user it represents (``apitest`` in the above example), the tenant it belongs to (``tacc`` above) when it expires, etc. Tapis
tokens always include the following standard claims:

+----------------------+-----------------------------------+--------------------------------------+
| Claim                | Description                       | Example Value                        |
+======================+===================================+======================================+
| sub                  | The subject of the token; the     |                                      |
|                      | subject uniquely identifies the   | apitest@tacc                         |  
|                      | user in a Tapis installation. The |                                      |
|                      | format is ``user`` @ ``tenant``   |                                      |
+----------------------+-----------------------------------+--------------------------------------+
| exp                  | The expiry associated with the    |  1595099456                          |
|                      | token.                            |                                      |
+----------------------+-----------------------------------+--------------------------------------+
| jti                  | Unique identifier for the token.  | f7b9b19e-6099-4f82-a722-b610c5d0ba0c |
+----------------------+-----------------------------------+--------------------------------------+
| iss                  | The identifier (URL) of the       |                                      |
|                      | issuer of the JWT. For Tapis, the | https://tacc.tapis.io/v3/tokens      |
|                      | issuer will be a Tokens API.      |                                      |
+----------------------+-----------------------------------+--------------------------------------+

Additional custom claims specific to Tapis are namespaced with ``tapis/`` at the beginning of the claim name. The 
authenticator for each tenant may optionally choose to support one or more of these additional claims. The following
claims are encouraged and supported by the default OAuth2 Tapis authenticator.

+----------------------+-----------------------------------+--------------------------------------+
| Claim                | Description                       | Example Value                        |
+======================+===================================+======================================+
| tapis/tenant_id      | The tenant of the subject.        | tacc                                 |
+----------------------+-----------------------------------+--------------------------------------+
| tapis/username       | The username of the subject.      | apitest                              |
+----------------------+-----------------------------------+--------------------------------------+
| tapis/token_type     | Type of token: ``access`` or      | access                               |
|                      | ``refresh``                       |                                      |
+----------------------+-----------------------------------+--------------------------------------+
| tapis/account_type   | Type of account: ``user`` or      | user                                 |
|                      | ``service``                       |                                      |
+----------------------+-----------------------------------+--------------------------------------+
| tapis/delegation     | Whether a delegation flow was used|                                      |
|                      | to generate this token. (``true`` | false                                |
|                      | or ``false``).                    |                                      |
+----------------------+-----------------------------------+--------------------------------------+
| tapis/delegation_sub | For a delegation token, the       |                                      |
|                      | subject who actually generated the| superuser@tacc                       |
|                      | token. In form                    |                                      |
|                      | ``user`` @ ``tenant``             |                                      |
+----------------------+-----------------------------------+--------------------------------------+
| tapis/client_id      | The id of the OAuth client used to|                                      |
|                      | generate the token.               | tacc.CIC.tokenapp                    |
+----------------------+-----------------------------------+--------------------------------------+
| tapis/grant_type     | The grant type used to generate   | authorization_code                   |
|                      | the token.                        |                                      |
+----------------------+-----------------------------------+--------------------------------------+

The authenticator for your tenant may include additional claims not listed here.


.. _JWT: https://jwt.io/introduction/ 

.. _oauth-clients-label:
OAuth Clients
~~~~~~~~~~~~~

In order to use the more advanced OAuth2 flows, including any use of the authorization code grant type and to generate
refresh tokens with the password grant type, you must generate an OAuth2 *client*. Clients in OAuth2 represent
applications (for example, a web or mobile application) that will interact with the OAuth2 server to generate tokens
on behalf of one or more users. Clients are created and managed using the ``/v3/oauth2/clients`` endpoints.


Creating Clients
^^^^^^^^^^^^^^^^

To create a client, make a POST request the the Clients API. All fields are optional; if you do not pass a 
``client_id`` or ``client_key`` in the request, the clients API will generate random ones for you. In order to
use the ``authorize_code`` grant type you will need to set the ``callback_url`` when registering your client (see :ref: `_auth_code`).
For a complete list of available parameters, see the API live-docs for Clients_.

With PySDK:

.. code-block:: plaintext

 >>> t.authenticator.create_client(client_id='test', callback_url='https://foo.example.com/oauth2/callback') 


With CURL:

.. code-block:: plaintext

 $ curl -H "X-Tapis-Token: $JWT" -H "Content-type: application/json" -d '{"client_id": "test", "callback_url": "https://foo.example.com/oauth2/callback" https://tacc.tapis.io/v3/oauth2/clients


The response will be similar to

.. code-block:: platintext

 callback_url: https://foo.example.com/oauth2/callback
 client_id: test
 client_key: WQZlQlMoxOynW
 create_time: Sat, 18 Jul 2020 19:09:47 GMT
 description:
 display_name: https://foo.example.com/oauth2/callback
 last_update_time: Sat, 18 Jul 2020 19:09:47 GMT
 owner: apitest
 tenant_id: tacc


.. _Clients: https://tapis-project.github.io/live-docs/#tag/Clients


Listing Clients
^^^^^^^^^^^^^^^

With PySDK:

.. code-block:: plaintext
 
 >>> t.authenticator.list_clients()


With CURL:

.. code-block:: plaintext

 $ curl -H "X-Tapis-Token: $JWT" https://tacc.tapis.io/v3/oauth2/clients

The response will be similar to

.. code-block:: plaintext

 [
 callback_url: https://foo.example.com/oauth2/callback
 client_id: test
 client_key: WQZlQlMoxOynW
 create_time: Sat, 18 Jul 2020 19:09:47 GMT
 description: 
 display_name: https://foo.example.com/oauth2/callback
 last_update_time: Sat, 18 Jul 2020 19:09:47 GMT
 owner: apitest
 tenant_id: tacc]


Deleting Clients
^^^^^^^^^^^^^^^^

You can also delete clients you are no longer using; just pass the ``client_id`` of the client to be deleted:

With PySDK:

.. code-block:: plaintext

 >>> t.authenticator.delete_client(client_id='test')


With CURL:

.. code-block:: plaintext

 $ curl -H "X-Tapis-Token: $JWT" https://tacc.tapis.io/v3/oauth2/clients/test

A null response is returned from a successful delete request.


.. _auth_code:
Authorization Code Grant - Generating Tokens For Users
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

An important aspect of OAuth2 is that it enables applications to generate tokens on behalf of users without the applications
needing to possess user credentials (i.e., passwords). In this section, we discuss using the OAuth2 *authorization code* grant
type to do just that.

Assuming a Model-View-Controller (MVC) architecture, there are two controllers that must be written to support the
authorization code grant type flow. 

1. A controller to determine if the user already has a valid access token and direct them to the OAuth2 authorization 
   server when they do not. This controller starts the authorization code process. To do so, it should:

  * First inform the user that they will be asked to authenticate with their tenant 
    username and password and then be asked to grant authorization to your client application. 

  * Redirect the user to the OAuth2 server's authorization URL. In the default Tapis authenticator, the 
    authorization URL path is ``/v3/oauth2/authorize``; for example, ``https://tacc.tapis.io/v3/oauth2/authorize`` in the 
    ``tacc`` tenant.

  * The redirect request should include the following query parameters:

    * ``client_id``: the id of your client.
    * ``client_redirect_uri``: the URI to redirect back to with the authorization code. This must match the 
      ``callback_url`` parameter associated with your client.
    * ``response_type``: should always have the value ``code``. 


2. A controller to process the authorization code returned and retrieve an access token on the user’s behalf. This 
   controller receives requests containing authorization codes from the OAuth2 server after the user has successfully 
   authenticated with said OAuth2 server, and it immediately exchanges the code for a token.

  * Responds to ``GET`` requests to the URL defined in the ``callback_url`` parameter of your client.
  * Retrieves the ``code`` query parameter from the request.
  * Makes a ``POST`` request to the OAuth2 server's tokens endpoint to generate a token. In the default Tapis 
    authenticator, the tokens URL path is ``/v3/oauth2/tokens``; for example, ``https://tacc.tapis.io/v3/oauth2/tokens``
    in the ``tacc`` tenant. The POST body must include the following parameters:

    * ``code``: the code the controller just received in the request from the OAuth2 server.
    * ``redirect_uri``: should be the same as the ``callback_url`` parameter of your client.
    * ``grant_type``: should always have the value ``authorization_code``.


Note that many popular web frameworks support OAuth2 flows with minimal custom coding required.

The final step to using the authorization code grant type is to register a client (see above) with a ``callback_url``
parameter equal to the URL within your web application where it will handle converting authorization codes into access
tokens (i.e., controller 2 above).


The Tapis Token Web Application
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Tapis provides a graphical interface via a web application that enables users to generate tokens. The Tapis Web 
Application is available by default for any tenant using the default Tapis authenticator, including the ``tacc`` tenant.
The Tapis Token Web Application serves as an example of an application using the authorization code grant type.

The Tapis Token Web Application and its source code are available at the following URLs:

* Token App (``tacc`` tenant): https://tacc.tapis.io/v3/oauth2/webapp
* Token App source code: https://github.com/tapis-project/authenticator
 

