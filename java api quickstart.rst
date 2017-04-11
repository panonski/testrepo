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

1. The classpath of the java application which uses this client library.

2. Environment variables SB_API_ENDPOINT and SB_AUTH_TOKEN.

3. Common configuration file with the defined parameters:

It is located at: Linux, Mac OS X: **$HOME/.sevenbridges/credentials**

MS Windows: **%UserProfile%\.sevenbridges\\credentials**

This is a special credentials file that may be shared with other Seven Bridges client applications, so you should specify only these two keys in it (if any). If you have more than one profile, you can store the information for each profile in it.  For example::

  [default]
  auth_token = 7eab0822exAMPle23d7d14c2eEXampLE
  api_url = https://api.sbgenomics.com/v2

  [my_other_profile]
  auth_token = 1e43fEXampLEa5523dfd14exAMPle3e5
  api_url = https://api.sbgenomics.com/v2

4. Library-specific configuration file **sevenbridges.properties** which contains the same keys as above, located at:

Linux, Mac OS X: **$HOME/.sevenbridges/sevenbridges-java/**
MS Windows: **%UserProfile%/.sevenbridges/sevenbridges-java/**

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

After the credentials are loaded, you can use a helper method to initialize the client and obtain the User object which holds your user information, such as your name and contact details::

  Client client = initClient();
  User user = client.getCurrentUser();

To obtain your details simply use the appropriate get() method. For email address, it would be::

  user.getEmail();

For convenience, some code samples contain logging commands. To use logging, invoke a Logger instance and pass the current class as a parameter::

  private static final Logger log = LoggerFactory.getLogger(ThisClass.class);

Managing projects
=================
Projects are the core building blocks of our Platform. All the analyses you design and run take place inside a project. Here are some basic methods for dealing with them.

If you are not familiar with the project structure of the Seven Bridges Platform and CGC, take a look at their respective documentation: `projects on the CGC <http://docs.cancergenomicscloud.org/docs/projects-on-the-cgc>`_ and `projects on the Seven Bridges Platform <http://docs.sevenbridges.com/docs/projects-on-the-platform>`_.

To list all the projects that you are a member of, call::

 ProjectList projects = user.getProjects();

You can then iterate through the list of projects and get their information, including names and IDs. A project is referenced through its ID, so you will need to know it in order to work with your project further::

  Iterator<Project> projectIterator = projects.iterator();
  log.info("Project names: ");
  while (projectIterator.hasNext()) {
    Project currentProject = projectIterator.next();
    log.info(" name: {} - id: {}",
    currentProject.getName(),
    currentProject.getId());
  }

The ID for a project consists of your username and the project's name, e.g. ``my-username/the-first-project``::

 wantedProject = user.getProjectById(String.format("%s/my-best-project", myUsername));

Each project also has a name, a description string indicating its use, a type, some tags, a ``billing_group`` identifier representing the billing group that is attached to the project and the href. The property ``href`` is a URL on the server that uniquely identifies the resource in question. All resources have this attribute. Each of the above attributes can be obtained using the relevant get method.

To create a new project, you need to provide its name and the billing group ID.

The billing group ID designates which funding resource to charge for the analyses you run in the project you're about to create. Learn more about `billing groups <http://docs.sevenbridges.com/v1.0/docs/payments#section-billing-groups>`_.
::
 BillingGroupList billingGroups = user.getBillingGroups();
 firstBillingGroup = billingGroups.iterator().next().getId();

Now you can create a new project. Remember that the project will be assigned an ID which consists of your username and the project’s shortname, which is created from the name you gave it through setName() (e.g. “rfranklin/new-test-project”).  The human readable name you set can be changed afterwards, but the project ID remains unchanged throughout the life of the project.

    Project newProject = client.instantiate(Project.class);
    newProject.setName("New test project")
        .setDescription("This is a project created through V2 API")
        .setBillingGroupId(user.getBillingGroups().iterator().next().getId());
    log.info("Created new project with name '{}' and project id '{}'", newProject.getName(), newProject.getId());



Managing project members
Sometimes it can feel lonely to be the only person in the project. You can add other users as members of your projects and assign them permissions as necessary. You will need to know their usernames on the platform.

The read permission is assigned by default to each project member and cannot be stripped. Other permissions are modifiable. Learn more about permissions.

First we instantiate a new project member and then provide the username of the person we want to add and set the necessary permissions. After that we add the user to the desired project.

Member newMember = client.instantiate(Member.class);
newMember
  // must be an existing user!
  .setUsername("annemarie.jones")
  .setPermissions(Members.getDefaultPermissions());
currentProject.addMember(newMember);

If you need to remind yourself of who has which permissions on your project, you can iterate through the list of the project members.

MemberList members = currentProject.getMembers();
Iterator<Member> memberIterator = members.iterator();
System.out.println("Members of the project " + currentProject.getId());
while (memberIterator.hasNext()) {
  Member currMember = memberIterator.next();
  log.info(" Username : {} Permissions : {}",
  currMember.getUsername(),
  currMember.getPermissions());
}

Sometimes, you might want to change permissions of a certain member. Let's give our user the right to modify (the write permission) and download (the copy permission) files from our common project.

Map<String, Boolean> permissions = newMember.getPermissions();
permissions.put("write", true);
permissions.put("copy", true);
newMember.setPermissions(permissions);
newMember.save();

Now, if you want to see the updated list of members and permissions, remember to reload it.

members.reload();
for (Member member : members) {
  log.info(" Username : {} Permissions : {}", member.getUsername(),
  member.getPermissions());
}

Finally, once your collaboration comes to an end, you can easily remove the member from the project.

currentProject.removeMember(newMember.getUsername());
