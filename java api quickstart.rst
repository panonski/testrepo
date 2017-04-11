==================================================
SevenBridges Java client for public API Quickstart
==================================================

This guide walks you through everything you need to get started with the Seven Bridges API Java client.

Authentication and Configuration
================================
To start, you need to authenticate with the API by passing your authentication token and the API endpoint you will be interacting with.

You can find your authentication token on the respective pages:
https://igor.sbgenomics.com/developers for the Seven Bridges Platform
https://cgc.sbgenomics.com/developers for the CGC

The API endpoints for each environment are:
https://api.sbgenomics.com/v2 for the Seven Bridges Platform
https://cgc-api.sbgenomics.com/v2 for the CGC.

For more information about the API, including details of the available parameters for each API call, you should check the API documentation before using this library:
http://docs.sevenbridges.com/docs/the-api for the Seven Bridges Platform.
http://docs.cancergenomicscloud.org/docs/the-cgc-api for the CGC.

Once you have obtained your authentication token from one of the URLs listed above, you can use the ClientBuilder class to initialize the API client::

 ClientBuilder builder = Clients.builder();

If you do not pass any other parameters, the builder will automatically attempt to find your API key values in a number of default/conventional locations and then use the discovered values.
The following locations will be each be checked in the following order:

#. The classpath of the java application which uses this client library.

|

#. Environment variables SB_API_ENDPOINT and SB_AUTH_TOKEN.

|

#. Common configuration file with the defined parameters. It is located at:

Linux, Mac OS X: **$HOME/.sevenbridges/credentials**

MS Windows: **%UserProfile%\.sevenbridges\\credentials**

This is a special credentials file that may be shared with other Seven Bridges client applications, so you should specify only these two keys in it (if any). If you have more than one profile, you can store the information for each profile in it.  For example::

  [default]
  auth_token = 7eab0822exAMPle23d7d14c2eEXampLE
  api_url = https://api.sbgenomics.com/v2

  [my_other_profile]
  auth_token = 1e43fEXampLEa5523dfd14exAMPle3e5
  api_url = https://api.sbgenomics.com/v2

#. Library-specific configuration file sevenbridges.properties which contains the same keys as above, located at:
Linux, Mac OS X: $HOME/.sevenbridges/sevenbridges-java/
MS Windows: %UserProfile%/.sevenbridges/sevenbridges-java/

If the above-mentioned methods do not satisfy your needs, you can also specify your credentials explicitly when you build a client::

  static Client initClient() {
    ClientBuilder builder = Clients.builder();
    builder.setBaseUrl("https://api.sbgenomics.com/v2");
    builder.setAuthenticationScheme(AuthenticationScheme.AUTH_TOKEN);
    builder.setApiKey(ApiKeys.builder()
      .setSecret("39bexAMPle2742338EXampLEe3dde8a7")
      .build());
      return builder.build();
  }

**NOTE: Keep your authentication token safe! It encodes all your credentials on the Platform or CGC. Generally, we recommend storing the token in a configuration file, which will then be stored in your home folder rather than in the code itself. This prevents the authentication token from being committed to source code repositories.**

Property Locations
~~~~~~~~~~~~~~~~~~

You can define stormpath property values in a number of locations.  This allows you to define a core set of properties in a primary configuration file and override values as necessary using other locations.

Configuration property values are read from the following locations, *in order*.  Values discovered in locations later (further down in the list) will automatically override values found in previous locations:

.. contents::
   :local:
   :depth: 2

If you're just starting out, we recommend that your configuration be specified in ``/WEB-INF/stormpath.properties`` and you use environment variables to specify password or secret values (e.g. for production environments).

Defining properties in these locations is covered more in detail next.

1. Plugin web.stormpath.properties
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This file resides in the stormpath-servlet-plugin-|version|.jar at:

 ``/com/stormpath/sdk/servlet/config/web.stormpath.properties``

It includes all of the plugin's default configuration and is not modifiable.  The default values within can be overridden by specifying properties in locations read later during the startup process.

2. classpath:stormpath.properties
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If a ``stormpath.properties`` file exists at the root of your web application's classpath (typically in ``/WEB-INF/classes`` or at the root of one of your .jar files in ``/WEB-INF/lib``), ``stormpath.*`` properties will be read from that file and override any identically-named properties discovered previously.

.. NOTE::
   Because this is not a web-specific location, it is only recommended to use this location if you wish to share stormpath properties configuration across multiple projects in a 'resource .jar' that is used in such projects.
After the credentials are loaded, you can use a helper method to initialize the client and obtain the User object which holds your user information, such as your name and contact details::

  Client client = initClient();
  User user = client.getCurrentUser();

To obtain your details simply use the appropriate get() method. For email address, it would be::

  user.getEmail();

For convenience, some code samples contain logging commands. To use logging, invoke a Logger instance and pass the current class as a parameter::

  private static final Logger log = LoggerFactory.getLogger(ThisClass.class);
