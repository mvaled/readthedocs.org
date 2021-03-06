Read the Docs Public API
=========================

We have a limited public API that is available for you to get data out of the site. This page will only show a few of the basic parts, please file a ticket or ping us on IRC (#readthedocs on `Freenode (chat.freenode.net) <http://webchat.freenode.net>`_) if you have feature requests.

This document covers only part of the API provided. We have plans to create a read/write API, so that you can easily automate interactions with your project.

The API is written in Tastypie, which provides a nice ability to browse the API from your browser. If you go to http://readthedocs.org/api/v1/?format=json and just poke around, you should be able to figure out what is going on.

A basic API client using slumber
--------------------------------

You can use `Slumber <http://slumber.readthedocs.io/>`_ to build basic API wrappers in python. Here is a simple example of using slumber to interact with the RTD API::

    from __future__ import print_function
    import slumber
    import json

    show_objs = True
    api = slumber.API(base_url='http://readthedocs.org/api/v1/')

    val = api.project.get(slug='pip')

    if show_objs:
        for obj in val['objects']:
            print(json.dumps(obj, indent=4))
    else:
        print(json.dumps(val, indent=4))
    
Alternatively you can try with the following value::

    # fetch project pip without metadata.
    val = api.project('pip').get()

    # get a specific build
    val = api.build(2592228).get()

    # get the build of a specific project.
    val = api.build.get(project__slug='read-the-docs')

    # get a specific user by `username`
    val = api.user.get(username='eric')

    #val = api.version('pip').get()
    #val = api.version('pip').get(slug='1.0.1')

    #val = api.version('pip').highest.get()
    #val = api.version('pip').highest('0.8').get()

Example of adding a user to a project
-------------------------------------

You can use the api to add user to a project,
to authenticate with `slumber`, use the following:

::

    from __future__ import print_function
    import slumber

    USERNAME = 'eric'
    PASSWORD = 'test'
    
    user_to_add = 'coleifer'
    project_slug = 'read-the-docs'

    api = slumber.API(base_url='http://readthedocs.org/api/v1/', auth=(USERNAME,PASSWORD))


::

    from __future__ import print_function

    project = api.project.get(slug=project_slug)
    user = api.user.get(username=user_to_add)
    project_objects = project['objects'][0]
    user_objects = user['objects'][0]

    data = {'users': project_objects['users'][:]}
    data['users'].append(user_objects['resource_uri'])

    print("Adding %s to %s" % (user_objects['username'], project_objects['slug']))
    api.project(project_objects['id']).put(data)

    project2 = api.project.get(slug=project_slug)
    project2_objects = project2['objects'][0]
    print("Before users: %s" % project_objects['users'])
    print("After users: %s" % project2_objects['users'])


API Endpoints
-------------

Feel free to use cURL and python to look at formatted json examples. You can also look at them in your browser, if it handles returned json.

::

    curl http://readthedocs.org/api/v1/project/pip/?format=json | python -m json.tool

Root
----
.. http:get::  /api/v1/

    Retrieve a list of resources.
   
   .. sourcecode:: js
  
      {
          "build": {
              "list_endpoint": "/api/v1/build/", 
              "schema": "/api/v1/build/schema/"
          }, 
          "file": {
              "list_endpoint": "/api/v1/file/", 
              "schema": "/api/v1/file/schema/"
          }, 
          "project": {
              "list_endpoint": "/api/v1/project/", 
              "schema": "/api/v1/project/schema/"
          }, 
          "user": {
              "list_endpoint": "/api/v1/user/", 
              "schema": "/api/v1/user/schema/"
          }, 
          "version": {
              "list_endpoint": "/api/v1/version/", 
              "schema": "/api/v1/version/schema/"
          }
      }
      
   :>json string list_endpoint: API endpoint for resource.
   :>json string schema: API endpoint for schema of resource.

Builds
------
.. http:get::  /api/v1/build/

    Retrieve a list of Builds.

   .. sourcecode:: js

      {
          "meta": {
              "limit": 20, 
              "next": "/api/v1/build/?limit=20&offset=20", 
              "offset": 0, 
              "previous": null, 
              "total_count": 86684
          }, 
          "objects": [BUILDS]
      }

   :>json integer limit: Number of Builds returned.
   :>json string next: URI for next set of Builds.
   :>json integer offset: Current offset used for pagination.
   :>json string previous: URI for previous set of Builds.
   :>json integer total_count: Total number of Builds.
   :>json array objects: Array of `Build`_ objects.


Build
-----
.. http:get::  /api/v1/build/{id}/

   :arg id: A Build id.

    Retrieve a single Build.

   .. sourcecode:: js

      {
          "date": "2012-03-12T19:58:29.307403", 
          "error": "SPHINX ERROR", 
          "id": "91207", 
          "output": "SPHINX OUTPUT", 
          "project": "/api/v1/project/2599/", 
          "resource_uri": "/api/v1/build/91207/", 
          "setup": "HEAD is now at cd00d00 Merge pull request #181 from Nagyman/solr_setup\n", 
          "setup_error": "", 
          "state": "finished", 
          "success": true, 
          "type": "html", 
          "version": "/api/v1/version/37405/"
      }


   :>json string date: Date of Build.
   :>json string error: Error from Sphinx build process.
   :>json string id: Build id.
   :>json string output: Output from Sphinx build process.
   :>json string project: URI for Project of Build.
   :>json string resource_uri: URI for Build.
   :>json string setup: Setup output from Sphinx build process.
   :>json string setup_error: Setup error from Sphinx build process.
   :>json string state: "triggered", "building", or "finished"
   :>json boolean success: Was build successful?
   :>json string type: Build type ("html", "pdf", "man", or "epub")
   :>json string version: URI for Version of Build.

Files
-----
.. http:get::  /api/v1/file/

    Retrieve a list of Files.

   .. sourcecode:: js

      {
          "meta": {
              "limit": 20, 
              "next": "/api/v1/file/?limit=20&offset=20", 
              "offset": 0, 
              "previous": null, 
              "total_count": 32084
          }, 
          "objects": [FILES]
      }


   :>json integer limit: Number of Files returned.
   :>json string next: URI for next set of Files.
   :>json integer offset: Current offset used for pagination.
   :>json string previous: URI for previous set of Files.
   :>json integer total_count: Total number of Files.
   :>json array objects: Array of `File`_ objects.

File
----
.. http:get::  /api/v1/file/{id}/

   :arg id: A File id.

    Retrieve a single File.

   .. sourcecode:: js

      {
          "absolute_url": "/docs/keystone/en/latest/search.html", 
          "id": "332692", 
          "name": "search.html", 
          "path": "search.html", 
          "project": {PROJECT},
          "resource_uri": "/api/v1/file/332692/"
        }


   :>json string absolute_url: URI for actual file (not the File object from the API.)
   :>json string id: File id.
   :>json string name: Name of File.
   :>json string path: Name of Path.
   :>json object project: A `Project`_ object for the file's project.
   :>json string resource_uri: URI for File object.

Projects
--------
.. http:get::  /api/v1/project/

    Retrieve a list of Projects.

   .. sourcecode:: js

      {
          "meta": {
              "limit": 20, 
              "next": "/api/v1/project/?limit=20&offset=20", 
              "offset": 0, 
              "previous": null, 
              "total_count": 2067
          }, 
          "objects": [PROJECTS]
      }


   :>json integer limit: Number of Projects returned.
   :>json string next: URI for next set of Projects.
   :>json integer offset: Current offset used for pagination.
   :>json string previous: URI for previous set of Projects.
   :>json integer total_count: Total number of Projects.
   :>json array objects: Array of `Project`_ objects.

   
Project
-------
.. http:get::  /api/v1/project/{id}

   :arg id: A Project id.

    Retrieve a single Project.

   .. sourcecode:: js

      {
          "absolute_url": "/projects/docs/", 
          "analytics_code": "", 
          "copyright": "", 
          "crate_url": "", 
          "default_branch": "", 
          "default_version": "latest", 
          "description": "Make docs.readthedocs.io work :D", 
          "django_packages_url": "", 
          "documentation_type": "sphinx", 
          "id": "2599", 
          "modified_date": "2012-03-12T19:59:09.130773", 
          "name": "docs", 
          "project_url": "", 
          "pub_date": "2012-02-19T18:10:56.582780", 
          "repo": "git://github.com/rtfd/readthedocs.org", 
          "repo_type": "git", 
          "requirements_file": "", 
          "resource_uri": "/api/v1/project/2599/", 
          "slug": "docs", 
          "subdomain": "http://docs.readthedocs.io/", 
          "suffix": ".rst", 
          "theme": "default", 
          "use_virtualenv": false, 
          "users": [
              "/api/v1/user/1/"
          ], 
          "version": ""
      }


   :>json string absolute_url: URI for project (not the Project object from the API.)
   :>json string analytics_code: Analytics tracking code.
   :>json string copyright: Copyright
   :>json string crate_url: Crate.io URI.
   :>json string default_branch: Default branch.
   :>json string default_version: Default version.
   :>json string description: Description of project.
   :>json string django_packages_url: Djangopackages.com URI.
   :>json string documentation_type: Either "sphinx" or "sphinx_html". 
   :>json string id: Project id.
   :>json string modified_date: Last modified date.
   :>json string name: Project name.
   :>json string project_url: Project homepage.
   :>json string pub_date: Last published date.
   :>json string repo: URI for VCS repository.
   :>json string repo_type: Type of VCS repository.
   :>json string requirements_file: Pip requirements file for packages needed for building docs.
   :>json string resource_uri: URI for Project.
   :>json string slug: Slug.
   :>json string subdomain: Subdomain.
   :>json string suffix: File suffix of docfiles. (Usually ".rst".)
   :>json string theme: Sphinx theme.
   :>json boolean use_virtualenv: Build project in a virtualenv? (True or False)
   :>json array users: Array of readthedocs.org user URIs for administrators of Project.
   :>json string version: DEPRECATED. 


Users
-----
.. http:get::  /api/v1/user/

    Retrieve List of Users

   .. sourcecode:: js
   
      {
          "meta": {
              "limit": 20, 
              "next": "/api/v1/user/?limit=20&offset=20", 
              "offset": 0, 
              "previous": null, 
              "total_count": 3200
          }, 
          "objects": [USERS]
      }

   :>json integer limit: Number of Users returned.
   :>json string next: URI for next set of Users.
   :>json integer offset: Current offset used for pagination.
   :>json string previous: URI for previous set of Users.
   :>json integer total_count: Total number of Users.
   :>json array USERS: Array of `User`_ objects.
 
 
User
----
.. http:get::  /api/v1/user/{id}/

   :arg id: A User id.
   
    Retrieve a single User

   .. sourcecode:: js
   
      {
          "first_name": "", 
          "id": "1", 
          "last_login": "2010-10-28T13:38:13.022687", 
          "last_name": "", 
          "resource_uri": "/api/v1/user/1/", 
          "username": "testuser"
      }
      
   :>json string first_name: First name.
   :>json string id: User id.
   :>json string last_login: Timestamp of last login.
   :>json string last_name: Last name.
   :>json string resource_uri: URI for this user.
   :>json string username: User name.
   
 
Versions
--------
.. http:get::  /api/v1/version/

    Retrieve a list of Versions.

   .. sourcecode:: js

      {
          "meta": {
              "limit": 20, 
              "next": "/api/v1/version/?limit=20&offset=20", 
              "offset": 0, 
              "previous": null, 
              "total_count": 16437
          }, 
          "objects": [VERSIONS]
      }


   :>json integer limit: Number of Versions returned.
   :>json string next: URI for next set of Versions.
   :>json integer offset: Current offset used for pagination.
   :>json string previous: URI for previous set of Versions.
   :>json integer total_count: Total number of Versions.
   :>json array objects: Array of `Version`_ objects.


Version
-------
.. http:get::  /api/v1/version/{id}

   :arg id: A Version id.

    Retrieve a single Version.

   .. sourcecode:: js

      {
          "active": false, 
          "built": false, 
          "id": "12095", 
          "identifier": "remotes/origin/zip_importing", 
          "project": {PROJECT}, 
          "resource_uri": "/api/v1/version/12095/", 
          "slug": "zip_importing", 
          "uploaded": false, 
          "verbose_name": "zip_importing"
      }


   :>json boolean active: Are we continuing to build docs for this version? 
   :>json boolean built: Have docs been built for this version?
   :>json string id: Version id.
   :>json string identifier: Identifier of Version.
   :>json object project: A `Project`_ object for the version's project.
   :>json string resource_uri: URI for Version object.
   :>json string slug: String that uniquely identifies a project
   :>json boolean uploaded: Were docs uploaded? (As opposed to being build by Read the Docs.)
   :>json string verbose_name: Usually the same as Slug.


Filtering Examples
------------------

Find Highest Version
~~~~~~~~~~~~~~~~~~~~
::

    http://readthedocs.org/api/v1/version/pip/highest/?format=json
    
.. http:get::  /api/v1/version/{id}/highest/

   :arg id: A Version id.

    Retrieve highest version.

   .. sourcecode:: js

      {
          "is_highest": true, 
          "project": "Version 1.0.1 of pip (5476)", 
          "slug": [
              "1.0.1"
          ], 
          "url": "/docs/pip/en/1.0.1/", 
          "version": "1.0.1"
      }


Compare Highest Version
~~~~~~~~~~~~~~~~~~~~~~~

This will allow you to compare whether a certain version is the highest version of a specific project. The below query should return a `'is_highest': false` in the returned dictionary.

::

    http://readthedocs.org/api/v1/version/pip/highest/0.8/?format=json 

.. http:get::  /api/v1/version/{id}/highest/{version}

   :arg id: A Version id.
   :arg version: A Version number or string.

    Retrieve highest version.

   .. sourcecode:: js

      {
          "is_highest": false, 
          "project": "Version 1.0.1 of pip (5476)", 
          "slug": [
              "1.0.1"
          ], 
          "url": "/docs/pip/en/1.0.1/", 
          "version": "1.0.1"
      }
 

File Search
~~~~~~~~~~~
::

    http://readthedocs.org/api/v1/file/search/?format=json&q=virtualenvwrapper
    
.. http:get::  /api/v1/file/search/?q={search_term}

   :arg search_term: Perform search with this term.

    Retrieve a list of File objects that contain the search term.

   .. sourcecode:: js
   
      {
          "objects": [
              {
                  "absolute_url": "/docs/python-guide/en/latest/scenarios/virtualenvs/index.html", 
                  "id": "375539", 
                  "name": "index.html", 
                  "path": "scenarios/virtualenvs/index.html", 
                  "project": {
                      "absolute_url": "/projects/python-guide/", 
                      "analytics_code": null, 
                      "copyright": "Unknown", 
                      "crate_url": "", 
                      "default_branch": "", 
                      "default_version": "latest", 
                      "description": "[WIP] Python best practices...", 
                      "django_packages_url": "", 
                      "documentation_type": "sphinx_htmldir", 
                      "id": "530", 
                      "modified_date": "2012-03-13T01:05:30.191496", 
                      "name": "python-guide", 
                      "project_url": "", 
                      "pub_date": "2011-03-20T19:40:03.599987", 
                      "repo": "git://github.com/kennethreitz/python-guide.git", 
                      "repo_type": "git", 
                      "requirements_file": "", 
                      "resource_uri": "/api/v1/project/530/", 
                      "slug": "python-guide", 
                      "subdomain": "http://python-guide.readthedocs.io/", 
                      "suffix": ".rst", 
                      "theme": "kr", 
                      "use_virtualenv": false, 
                      "users": [
                          "/api/v1/user/130/"
                      ], 
                      "version": ""
                  }, 
                  "resource_uri": "/api/v1/file/375539/", 
                  "text": "...<span class=\"highlighted\">virtualenvwrapper</span>\n..."
              },
              ...
          ]
      }

Anchor Search
~~~~~~~~~~~~~
::

    http://readthedocs.org/api/v1/file/anchor/?format=json&q=virtualenv

.. http:get::  /api/v1/file/anchor/?q={search_term}

   :arg search_term: Perform search of files containing anchor text with this term.

    Retrieve a list of absolute URIs for files that contain the search term.

   .. sourcecode:: js

      {
          "objects": [
              "http//django-fab-deploy.readthedocs.io/en/latest/...", 
              "http//dimagi-deployment-tools.readthedocs.io/en/...", 
              "http//openblock.readthedocs.io/en/latest/install/base_install.html#virtualenv", 
              ...
          ]
      }

