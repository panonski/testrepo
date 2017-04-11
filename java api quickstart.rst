==================================================
SevenBridges Java client for public API Quickstart
==================================================

This guide walks you through everything you need to get started with the Seven Bridges API Java client.

.. contents::
   :local:
   :depth: 2

Authentication and Configuration
================================
To start, you need to authenticate with the API by passing your authentication token and the API endpoint you will be interacting with.

You can find your authentication token on the respective pages:

- https://igor.sbgenomics.com/developers for the Seven Bridges Platform
- https://cgc.sbgenomics.com/developers for the CGC

The API endpoints for each environment are:

- https://api.sbgenomics.com/v2 for the Seven Bridges Platform
- https://cgc-api.sbgenomics.com/v2 for the CGC.

For more information about the API, including details of the available parameters for each API call, you should check the API documentation before using this library:

- http://docs.sevenbridges.com/docs/the-api for the Seven Bridges Platform.
- http://docs.cancergenomicscloud.org/docs/the-cgc-api for the CGC.

Once you have obtained your authentication token from one of the URLs listed above, you can use the `ClientBuilder` class to initialize the API client::

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

After the credentials are loaded, you can use a helper method to initialize the client and obtain the `User` object which holds your user information, such as your name and contact details::

  Client client = initClient();
  User user = client.getCurrentUser();

To obtain your details simply use the appropriate `get` method. For email address, it would be::

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

Each project also has a name, a description string indicating its use, a type, some tags, a ``billing_group`` identifier representing the billing group that is attached to the project and the href. The property ``href`` is a URL on the server that uniquely identifies the resource in question. All resources have this attribute. Each of the above attributes can be obtained using the relevant `get` method.

To create a new project, you need to provide its name and the billing group ID.

The billing group ID designates which funding resource to charge for the analyses you run in the project you're about to create. Learn more about `billing groups <http://docs.sevenbridges.com/v1.0/docs/payments#section-billing-groups>`_.
::
 BillingGroupList billingGroups = user.getBillingGroups();
 firstBillingGroup = billingGroups.iterator().next().getId();

Now you can create a new project. Remember that the project will be assigned an ID which consists of your username and the project’s shortname, which is created from the name you gave it through ``setName()`` (e.g. ``rfranklin/new-test-project``).  The human readable name you set can be changed afterwards, but the project ID remains unchanged throughout the life of the project.
::
    Project newProject = client.instantiate(Project.class);
    newProject.setName("New test project")
        .setDescription("This is a project created through V2 API")
        .setBillingGroupId(user.getBillingGroups().iterator().next().getId());
    log.info("Created new project with name '{}' and project id '{}'", newProject.getName(), newProject.getId());


Managing project members
========================

Sometimes it can feel lonely to be the only person in the project. You can add other users as members of your projects and assign them permissions as necessary. You will need to know their usernames on the platform.

The ``read`` permission is assigned by default to each project member and cannot be stripped. Other permissions are modifiable. You can learn more about permissions `here <http://docs.sevenbridges.com/v1.0/docs/set-permissions>`_.

First we instantiate a new project member and then provide the username of the person we want to add and set the necessary permissions. After that we add the user to the desired project::

  Member newMember = client.instantiate(Member.class);
  newMember
    // must be an existing user!
    .setUsername("annemarie.jones")
    .setPermissions(Members.getDefaultPermissions());
  currentProject.addMember(newMember);

If you need to remind yourself of who has which permissions on your project, you can iterate through the list of the project members::

  MemberList members = currentProject.getMembers();
  Iterator<Member> memberIterator = members.iterator();
  System.out.println("Members of the project " + currentProject.getId());
  while (memberIterator.hasNext()) {
    Member currMember = memberIterator.next();
    log.info(" Username : {} Permissions : {}",
    currMember.getUsername(),
    currMember.getPermissions());
  }

Sometimes, you might want to change permissions of a certain member. Let's give our user the right to modify (the ``write`` permission) and download (the ``copy`` permission) files from our common project::

  Map<String, Boolean> permissions = newMember.getPermissions();
  permissions.put("write", true);
  permissions.put("copy", true);
  newMember.setPermissions(permissions);
  newMember.save();

Now, if you want to see the updated list of members and permissions, remember to reload it::

  members.reload();
  for (Member member : members) {
    log.info(" Username : {} Permissions : {}", member.getUsername(),
    member.getPermissions());
  }

Finally, once your collaboration comes to an end, you can easily remove the member from the project::

  currentProject.removeMember(newMember.getUsername());

Managing files on the Platform
==============================

Files are an integral part of each analysis. Before you can analyze data on the Seven Bridges Platform, the files need to be within a specific project. You can either use the files that are already publicly available on the Platform or upload your own.

You can check what files are currently in your project by iterating through a list of files::

  Project myProject = user.getProjectById(String.format("%s/source-project", user.getUsername()));
  log.info("In project {} there are {} files", myProject.getName(), myProject.getFiles().getSize());
  FileList filesInMyProject = myProject.getFiles();
  for (File file : filesInMyProject) {
    log.info(" File {} with name {} and size {}B", file.getId(), file.getName(), file.getSize());
  }


If you are starting from an empty project, one way to get going is to copy some of the public files into your project. You can utilize the file tags to find the files you need. The tags are keywords or strings that make it easier to identify and organize files you’ve imported from public datasets.

You can learn more about `public files <http://docs.sevenbridges.com/v1.0/docs/file-repositories>`_ or `tagging your files <http://docs.sevenbridges.com/docs/tag-your-files>`_.

Let’s say you want to copy all the files that are related to human genome version 19. They will be tagged with the ``hg19`` tag::

  Project destinationProject = user.getProjectById(String.format("%s/destination-project", user.getUsername()));
  log.info(" There are {} files in destination project before copy", destinationProject.getFiles().getSize());
  log.info(" There are {} files in destination project before copy", destinationProject.getFiles().getSize());
  FileList publicFiles = user.getPublicFiles(Files.criteria().withTag("hg19"));
  for (File file : publicFiles) {
    file.copy(destinationProject, file.getName() + "_copy");
  }
  log.info(" There are {} files in destination project after copy", destinationProject.getFiles().getSize());


Tags can be applied to mark files in any way you find useful. Let’s say you decided you will not use files from a certain project anymore, but do not want to delete them until someone else has checked out your project. You can then use tags to mark the files as ready for deletion::

  FileList destinationFiles = destinationProject.getFiles();
  Iterator<File> iterator = destinationFiles.iterator();
  while (iterator.hasNext()) {
    File next = iterator.next();
    next.setTags(Collections.singleton("for_deletion"));
    next.save();
  }

All files have associated metadata which makes them searchable, keeping your file collection manageable as it grows. It also enables you group files properly for analyses.

You can learn more about metadata `here <http://docs.sevenbridges.com/v1.0/docs/metadata-on-the-seven-bridges-platform>`_.

When you need to change metadata on a file, you should first obtain the file's ID. Then you can either patch the file metadata (adding new and-or changing existing metadata fields) or you can overwrite it (which means any metadata fields you do not explicitly reset will be deleted).

To edit metadata, use the method ``patchMetadata()``::

  File updatedFile = user.getFileById("584d6f2160b2a10069e40d5d");
  Map<String, String> metaPatch = new HashMap<>();
  metaPatch.put("paired_end", "1");
  metaPatch.put("batch_number", "3");
  updatedFile.patchMetadata(metaPatch);
  // save the changes you made!
  updatedFile.save();

To overwrite metadata, use the method ``setMetadata()``::

  Map<String, String> metaOver = new HashMap<>();
  metaOver.put("case_id", "CCLE-HCC1143BL");
  metaOver.put("experimental_strategy", "WGS");
  metaOver.put("investigation", "CCLE-BRCA");
  metaOver.put("paired_end", "2");
  metaOver.put("platform", "Illumina");
  metaOver.put("species", "Homo sapiens");
  updatedFile.setMetadata(metaOver);
  // save the changes you made
  updatedFile.save();

Each file also has a ``URL`` property, which gives you the URL you can use to download the file. Again, you will need to know the file ID to do this::

  File toDownload = user.getFileById("584d6f2160b2a10069e40d5d");
  String downloadUrl = toDownload.getDownloadInfo().getUrl();
  InputStream downloadStream = null;
  try {
    URL url = new URL(downloadUrl);
    downloadStream = url.openStream();
    log.info("Downloading file...");
    long bytesNum = java.nio.file.Files.copy(downloadStream,   Paths.get("local_file"), StandardCopyOption.REPLACE_EXISTING);
    log.info("Downloaded {} bytes", bytesNum);
  } catch (MalformedURLException e) {
    log.error("Malformed exception while creating URL from {} - error message: {}", String.valueOf(downloadUrl), e.getMessage());
  } catch (IOException e) {
    log.error("Error while downloading file {} ", e);
  } finally {
    if (downloadStream != null) {
      try {
        downloadStream.close();
      } catch (IOException skip) {
        log.warn("Error while trying to close download stream - {}", skip);
      }
    }
  }


Uploading files
===============

If you want to use your private data for analysis, you can upload them securely to the Platform.
In this section, we will see how to upload files into a project and some actions you can perform on an upload object: using an ``UploadListener`` to listen for related events, polling the number of uploaded bytes while waiting for an upload to complete, blocking further work until upload is completed, cancelling an upload and pausing and restarting an upload.

The class ``AbstractProgressListener`` contains callback methods that inform you of the state of your upload. You can implement the methods for the events you want to listen to, like in this example::

  private static class MyProgressListener extends AbstractProgressListener {

    private final AtomicInteger cnt;

    MyProgressListener(AtomicInteger cnt) {
      this.cnt = cnt;
    }

    @Override
    public void uploadFailed(Exception ex) {
      cnt.incrementAndGet();
      log.error("upload failed ", ex);
    }

    @Override
    public void uploadFinished() {
      cnt.incrementAndGet();
      log.info("upload finished ");
    }

    @Override
    public void partUploadFailed(int partNumber, int retryCnt, Exception executionException) {
      log.error("part {} failed, retry {}, exception {}", partNumber, retryCnt, executionException);
    }
 }

Then you can build a synchronized upload request using ``uploadBuilder``::

  CreateUploadRequestBuilder uploadBuilder = client.getUploadRequestBuilder();

    // block on an upload (synchronized upload)
    uploadBuilder.setName("Sync-file")
        .setFile(myPrivateFile)
        .setOverwrite(true)
        .setProject(destinationProject);

Based on this, you can create an ``UploadContext`` object which will hold information about your upload::

  List<com.sbgenomics.java.file.File> uploadedFiles = new ArrayList<>();

  UploadContext upload = user.submitUpload(uploadBuilder.build());
      try {
        com.sbgenomics.java.file.File uploadedFile = upload.getFile(); //blocking call
        uploadedFiles.add(uploadedFile);
      } catch (RuntimeException e) {
        log.error("Error while waiting for the file to be uploaded - {}", e);
      }

If you have a list of files you want to have uploaded (let’s call it ``filesToUpload``), you can request upload of those files simultaneously, creating an ``UploadContext`` and ``ProgressListener`` for each file::

  static final long KB = 1024L;
  static final long MB = 1024 * KB;

  // parallel upload without blocking on get, setting up a listener for finished files
  AtomicInteger finishedCnt = new AtomicInteger(0);
  int numOfFiles = filesToUpload.size();
  List<UploadContext> uploads = new ArrayList<>(numOfFiles);
  for (int i = 0; i < numOfFiles; i++) {
    File testFile = filesToUpload(i);

    uploadBuilder.setName("File-to-upload-no-" + i)
        .setFile(testFile)
        .setOverwrite(true)
        .setProject(destinationProject)

        // if you want to cut upload in parts
        .setPartSize(64 * MB);

      UploadContext uploadContext = user.submitUpload(uploadBuilder.build(), new MyProgressListener(finishedCnt));
      uploads.add(uploadContext);
 }


When you submit an upload, a ``TransferService`` is started (if it hadn’t been previously started by an earlier upload). The ``UploadContext`` objects allow you to keep track of the progress of your uploads::

  while (finishedCnt.get() < numOfFiles) {
    Thread.sleep(1000);
    log.info("---------------------------------------------");
    for (UploadContext uploadContext : uploads) {
      long transferred = uploadContext.getBytesTransferred();
      long size = uploadContext.getUploadSize();
      log.info("Transferred {}% -  {} bytes out of {} for upload {}", (int)((transferred * 100)/size), transferred, size, uploadContext.getUploadName());
    }
  }

  for (UploadContext uploadContext : uploads) {
    if (uploadContext.isFinished()) {
      uploadedFiles.add(uploadContext.getFile());
    }
  }

If you want to pause an upload, you can do it through ``UploadContext.pauseTransfer()``. This will pause the current upload operation and store the information that can be used to resume the upload. The paused ``UploadContext`` object can later be passed to another ``UploadContext`` object which will resume upload from the reference point::

    uploadBuilder
        .setName("File-to-pause-and-resume")
        .setFile(testFile)
        .setOverwrite(true)
        .setProject(destinationProject)
        .setPartSize(32 * MB);
    UploadContext toPause = user.submitUpload(uploadBuilder.build());

    log.info("toPause state {}", toPause.getState());
    Thread.sleep(10_000); // let it work a while
    toPause.pauseTransfer();
    log.info("toPause state {}", toPause.getState());
    // waiting for the upload to be paused
    while (UploadState.PAUSING.equals(toPause.getState())) {
      Thread.sleep(5_000);
    }
    log.info("toPause state {}", toPause.getState());
    UploadContext resumedUpload = user.resumeUpload(toPause, testFile);
    log.info("resumedUpload state {}", resumedUpload.getState());


Trying to obtain the file from a paused upload will throw a ``PausedUploadException``. You can use this to resume the paused upload when needed.

On some occasions your app (or an external factor) might need to abort an upload. If you have instantiated an ``UploadContext`` in the above described manner, you can obtain the ID of your upload and abort it. Please keep in mind that trying to obtain the file from the ``UploadContext`` after the abort will throw an exception.
::
    String uploadId = uploadContextToBeAborted.getUploadId();
    Upload uploadById = user.getUploadById(uploadId);
    log.info("Thread {} is aborting running upload {}", Thread.currentThread().getName(), uploadId);
    uploadById.abortUpload();

Once you have uploaded all the files you needed, it’s time to close the ``TransferService`` to make sure you gracefully shutdown daemon threads and release resources::

  user.shutdownTransferService();

Managing apps
=============

The concept of apps on the platform includes both the **tools** (individual bioinformatics utilities) and the **workflows** (chains or pipelines of connected tools), either previously existing or user-created. In the section, we will see how to get information about publicly available apps, to check which apps are currently in a given project and how to install a new app based on a pre-formatted JSON file.

Getting information about publicly available apps is simple.	The following excerpt shows you how to get the name and the ID. You will need to know the ID of an app in order to work further with it::

  AppList publicApps = user.getPublicApps();
  log.info("There are {} public apps", publicApps.getSize());
  for (App publicApp : publicApps) {
    log.info("App id {} name {}", publicApp.getId(), publicApp.getName());
  }

If you want to use a certain app inside your project, you can copy it into the project.

If you try to copy an app that already exists in the given project, the API will issue an error message, which you can use to take appropriate action and inform the user as necessary.

Here is `the list of API status codes and descriptions <http://docs.sevenbridges.com/reference#api-status-codes>`_.
::
  Project destProject = user.getProjectById(String.format("%s/destination-project", user.getUsername()));
  App appToCopy = user.getAppById("admin/sbg-public-data/fastqc-analysis/2");

      // copy action will fail if there is already app with same id in the project
      App copiedApp;
      try {
        copiedApp = appToCopy.copy(destProject);
      } catch (ResourceException e) {
        log.debug("Error while trying to copy app - " + e.toString());
        if (e.getStatus() == 409) { // CONFLICT, app with same ID already exists in project
          copiedApp = user.getAppById(String.format("%s/fastqc-analysis", destProject.getId()));
          log.info("App already exists in destination project, with id {}", copiedApp.getId());
        } else {
          log.error("Error while getting app by id, expected success code or 409 HTTP status, got {}", e.getMessage());
          throw e;
        }
      }


To check which apps are currently in your project, call::

  AppList destProjectApps = destProject.getApps();

Finally, if you have a JSON file which describes your app `through CWL <http://docs.sevenbridges.com/docs/sdk-overview#section-the-common-workflow-language>`_, you can use it to install the app in a project::

  try {
    String raw = new String(Files.readAllBytes(Paths.get("my-app-raw.json")));
        destProject.installApp("my-installed-app", raw);
  } catch (IOException e) {
    log.error("Error while reading file 'my-app-raw.json', {}", e);
  }


Managing tasks
==============

An app execution is called a task. Each task is associated with a set of input files and chosen settings for the tool(s) in the app. In this part, we will see how to copy files that satisfy certain criteria, copy a relevant app into the project and build a task. We will see how to poll for task status during execution and how to get other useful information about a task.

In order to create a new task, you can create a ``TaskBuilder`` and set its fields appropriately. Let’s say you want to index a FASTA file with our public app ``SAMtools Index FASTA``::

  // find a fasta file from public repo and copy it to your project
  File fastaFile = user.getProjectById("admin/sbg-public-data")
    .getFiles(Files.criteria()
    .withName("HG19_Broad_variant.fasta"))
    .single();

  Project tasksProject  = user.getProjectById(String.format("%s/task-test", user.getUsername()));

  File input = fastaFile.copy(tasksProject);

  // copy the app to the project
  App sourceApp =
  user.getAppById("admin/sbg-public-data/samtools-index-fasta-1-3");

  App myApp = sourceApp.copy(tasksProject);

  TaskRequestFactory taskFactory = user.getTaskRequestFactory();
  CreateTaskRequestBuilder taskBuilder = taskFactory.createTaskBuilder();
  taskBuilder.setApp(myApp)
    .setDescription("run from public API v2")
    .setName("API_task_samtools")
    .setProject(tasksProject)
    .addInput("input_fasta_file", input)
  // to run immediately after task creation
    .runNow(true);

  Task task = user.createTask(taskBuilder.build());

When the task is executed successfully, its status will change to ``COMPLETED``. Until that happens, you can occasionally poll its status. *(Just bear in mind that each check of the status fires up an API request so it shouldn’t be done every second :)*
::
  while (!TaskStatus.isFinished(task.getStatus())) {
    try {
      Thread.sleep(30_000);
    } catch (InterruptedException e) {
      log.error("Interrupted from sleep, but task job {} is not finished yet", task.getId());
      throw new RuntimeException(e);
    }
    task.reload();
  }

If you want to check which tasks have completed, you can use their status as a search criterion.
You can also get other information about each task, e.g. its name, inputs and outputs::

  TaskList tasks =   user.getTasks(Tasks.criteria().withStatus(TaskStatus.COMPLETED));
  log.info("List of completed tasks for current user");
  for (Task completedTasks : tasks) {
     log.info("  Task name: {}", task.getName());
     log.info("    Inputs: {}", task.getInputs());
     log.info("    Outputs: {}", task.getOutputs());
  }


Batch tasks
===========

Sometimes you will need to run identical analyses on different data. You can do so through `batching <http://docs.sevenbridges.com/perform-batch-analysis>`_, where you enter multiple input files and group them by specified metadata criteria.

This segment on batch tasks follows `this API tutorial <http://docs.sevenbridges.com/reference#api-batch-tutorial>`_, which contains more detailed information. Here we will focus on implementing a batch task using our Java library. We will see how to set up a batch analysis in which we align reads based on their sample metadata. We will see how to copy input and reference files as well as the relevant apps into the project and then create a task and check for errors during creation of the task. We will also show how to check the status of the task while it is running and how to collect the output files once the task is completed.

Previously, we have discussed creating a project so we will assume that you have a project with a short name “batch-test” in which you want to perform the batch analysis::

  Project batchProject = user.getProjectById(String.format("%s/batch-test", user.getUsername()));

We'll analyze data that is hosted in the `Cancer Cell Line Encyclopedia (CCLE) <http://docs.sevenbridges.com/ccle>`_ public project on the Seven Bridges Platform. The CCLE public project is specified by the ID of ``sevenbridges/cancer-cell-line-encyclopedia-ccle-1``.  Inside it, we want to find BAM files with an experimental strategy of RNA-Seq. It is possible to filter by `metadata fields <http://docs.sevenbridges.com/metadata-on-the-seven-bridges-platform>`_ to retrieve files with certain properties. In this tutorial, however, we already know that we want to find the following three files:

- G30630.VM-CUB1.3.bam
- G30603.TUHR4TKB.1.bam
- G28034.MDA-MB-361.1.bam

::

  FileList inputCcleFiles = user.getProjectById("sevenbridges/cancer-cell-line-encyclopedia-ccle-1")
        .getFiles(Files.criteria()
        .withName("G30630.VM-CUB1.3.bam")
        .withName("G30603.TUHR4TKB.1.bam")
        .withName("G28034.MDA-MB-361.1.bam"));

The next step is to copy the files into our project::

  List<File> inputFiles = new ArrayList<>(3);
  for (File file : inputCcleFiles) {
    File copy = file.copy(batchProject);
    inputFiles.add(copy);
  }

Many bioinformatics tools require certain data, such as reference genomes or annotation files, to execute properly. Seven Bridges maintains a collection of the latest and most frequently used reference genomes and annotation files in the Public Reference Files repository. The Seven Bridges Public Reference Files repository is specified in the same way as a project on the Platform by an id of `admin/sbg-public-data`.

For this analysis, we need to supply the workflow with the following two reference files:

- HG19_Broad_variant.fasta
- Homo_sapiens.GRCh37.75.gtf

::

  FileList inputPublicFiles = user.getPublicFiles(Files.criteria()
    .withName("HG19_Broad_variant.fasta")
    .withName("Homo_sapiens.GRCh37.75.gtf"));
  for (File file : inputPublicFiles) {
     file.copy(batchProject);
  }


Next, we need to copy the RNA-seq Alignment STAR app from the list of public apps. Check here how to obtain the ID of the app::

  App appToCopy = user.getAppById("admin/sbg-public-data/rna-seq-alignment-star/16
  ");
  App appRnaSeqAlign = appToCopy.copy(batchProject);


At this point, we need to edit the app so that it can accept the BAM files we copied as input. You can do it by editing its CWL or via the visual interface. `Here are the instructions <http://docs.sevenbridges.com/reference#section-5c-modify-a-workflow-on-the-visual-interface>`_ for using Workflow editor on the visual interface.

After the app is edited, we can build the task and set appropriate fields::

    TaskRequestFactory taskFactory = user.getTaskRequestFactory();
    CreateTaskRequestBuilder taskBuilder = taskFactory.createTaskBuilder();
    taskBuilder
        .setApp(appRnaSeqAlign)
        .setDescription("run from public API v2 with batch")
        .setName("Batch run RNA seq - api client java")
        .setProject(batchProject)
        // the input port on which you wish to batch
        .setBatchInput("input_file")
        .addBatchBy("CRITERIA", java.util.Collections.singletonList("metadata.sample_id"))
        .addInput("input_file", inputFiles.toArray(new File[0]))
        .addInput("reference_or_index", batchProject.getFiles(Files.criteria().withName("HG19_Broad_variant.fasta")).single())
        .addInputAsSingletonList("sjdbGTFfile", batchProject.getFiles(Files.criteria().withName("Homo_sapiens.GRCh37.75.gtf")).single())
    .runNow(true);

Since we set the ``runNow`` parameter to true, the task will run as soon as we create it. If you leave out this field, the task will be created and stay in `DRAFT` status until you call ``run()`` on it.
::
  //creates and runs the task, since runNow is true
  Task task = user.createTask(taskBuilder.build());

If task runs into errors during creation, they will be attached to the task object::

    List<Map<String, Object>> errors = task.getErrors();
    if (errors != null && errors.size() > 0) {
      log.error("Error while validating task with id {}", task.getId());
      for (Map<String, Object> error : errors) {
        log.error("    {}", error);
      }
    }

While the task is being executed, you can occasionally check its status. You will also be notified by email once your task is completed.
::
 while (!TaskStatus.isFinished(task.getStatus())) {
        log.info("Sleeping");
        try {
          Thread.sleep(30_000);
        } catch (InterruptedException e) {
          Thread.interrupted();
          log.error("Interrupted while waiting for task to finish");
          throw new RuntimeException(e);
        }
        task.reload();
        log.info("Current status {}", task.getStatus().toString());
  }


When the task is completed, you can obtain the list of files that make up its output. Or you can get information about the task status::

  if (TaskStatus.COMPLETED.equals(task.getStatus())) {
      FileList producedFiles = batchProject.getFiles(Files.criteria().withTaskOrigin(task));
      for (File producedFile : producedFiles) {
        log.info("Produced file name {} and id {}", producedFile.getName(), producedFile.getId());
      }
  } else {
    log.warn("Task is finished, but with status {}", task.getStatus());
  }


Managing volumes
================

Volumes authorize the Platform to access and query objects on a specified cloud storage (S3 - `Amazon Web Services <http://docs.sevenbridges.com/docs/aws-cloud-storage-tutorial>`_ or GCS - `Google Cloud Storage <http://docs.sevenbridges.com/docs/google-cloud-storage-tutorial>`_) on your behalf. We will examine setting up a volume on each of the services, listing your volumes, importing and exporting files and polling for an import or export status while waiting for a job to finish.

The S3 and GCS volumes each have their own builder. Before you can use them, you will need to set up the `AWS_ACCESS_KEY` and the `AWS_SECRET_KEY` for S3 or the `GCS_PRIVATE_KEY` for the GCS. Then an approapriate builder can be used to create a volume.

Here is the S3 example::

    CreateS3VolumeRequestBuilder s3Builder = user.getVolumeRequestFactory().s3Builder();
    s3Builder
        .setName("my_new_S3_volume_for_test")
        .setAccessMode(AccessMode.RW)
        .setAccessKey(AWS_ACCESS_KEY)
        .setSecretKey(AWS_SECRET_KEY)
        .setBucket("outer_user_test_bucket")
        .setDescription("S3 Volume for testing imports and exports from and to external (non sbg) S3 bucket")
        .setSseAlgorithm("AES256");
    Volume testVolume = user.createVolume(s3Builder.build());


This is the corresponding GCS example::

    CreateGcsVolumeRequestBuilder gcsBuilder = user.getVolumeRequestFactory().gcsBuilder();
      gcsBuilder
        .setName("my_new_GCS_volume_for_test")
        .setAccessMode(AccessMode.RO) //only read mode for GCS
        .setBucket("outer_user_GCS_import")
        .setClientEmail(CLIENT_GOOGLE_EMAIL)
        .setPrivateKey("-----BEGIN PRIVATE KEY-----"+ GCS_PRIVATE_KEY + "-----END PRIVATE KEY-----\\n");
    Volume gcsVolume = user.createVolume(gcsBuilder.build());

Each file is imported using an import job. Once you have created it, you can refer to it by its ID::

    ImportJob gcsImportJob = user.startImport(gcsVolume, "files/google_file_key", destProject, "gcs_file", false);
    log.info("GCS volume import id : {}", gcsImportJob.getId());

If you have a list of import jobs (called `imports` in this example), you can poll for their status occasionally, until they are all finished::

    int finishedCnt = 0;
    while (finishedCnt < imports.size()) {
      try {
        Thread.sleep(5000);
        Iterator<ImportJob> iterator = imports.iterator();
        while (iterator.hasNext()) {
          ImportJob next = iterator.next();
          next.reload();
          if (VolumeJobState.isFinished(next.getState())) {
            log.info("Volume import id : {} name {} done", next.getId(), next.getDestinationName());
            finishedCnt++;
            iterator.remove();
          }
        }
      } catch (Exception e) {
        log.error(" Error while waiting for imports to finish - {}", e.getMessage());
        break;
      }
    }


To export a file, refer to it by its ID and create an export job::

    File fileToExport = user.getFileById("584d6f160b2a10069e40d5d");
    ExportJob exportJob = user.startExport(fileToExport, testVolume, "exported-file", false);

Similarly to imports, you can check on the status of your export job occasionally::

    try {
      while (!VolumeJobState.isFinished(exportJob.getState())) {
        Thread.sleep(5000);
        exportJob.reload();
      }
    } catch (Exception e) {
      log.error(" Error while waiting for export to finish = {}", e.getMessage());
    }

    if (VolumeJobState.COMPLETED.equals(exportJob.getState())) {
      log.info("Volume export with id {} is completed successfully", exportJob.getId());
    } else {
      log.warn("Volume export with id {} failed", exportJob.getId());
    }
