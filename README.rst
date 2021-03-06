Runestone Interactive Tools and Content
=======================================


.. |buildstatus| image:: https://drone.io/github.com/bnmnetp/runestone/status.png

Build Status |buildstatus|

Important Notice
----------------

On July 26, 2013 We made some structural changes which involved moving the
three extisting submodules.  skulpt, codelens, and js-parson.  Submodules are
a known PITA for many git operations.  If you have an existing project here is
what you should do.

1.  git pull or follow the fetch instructions for your Fork.
2.  git submodule init
3.  git submodule sync
4.  git submodule update

If you are Forking this repo for the first time, you can just proceed to the
dependencies.

Dependencies
------------

There are a couple of prerequisites you need to satisfy before you can build and use this
eBook. The easiest/recommended way is to use `pip <http://www.pip-installer.org/en/latest/>`_.

First get `Sphinx <http://sphinx.pocoo.org>`_, version 1.1.x is current as of this writing:

::

    # pip install sphinx

Install `paver <http://paver.github.io/paver/>`_, at least version 1.2.0:

::

    # pip install paver


Once paver is installed you will also need to install sphinxcontrib-paverutils, at least version 1.5:

::

    # pip install sphinxcontrib-paverutils


If you want to run a full blown server, so you can save ActiveCode assignments, etc. you will need to download and
install `web2py <http://web2py.com>`_.

The easiest way to do so is to download the **Source Code** distribution from http://www.web2py.com/init/default/download.
`Here <http://www.web2py.com/examples/static/web2py_src.zip>`_ is a direct link to the zip archive.
After you download it, extract the zip file to some folder on your hard drive. (web2py requires no real "installation").  I avoid the web2py.app installation on OS X as it messes with the Python path.  I assume the Windows web2py.exe is the same and I would avoid it as well if I used Windows.

Within the ``web2py`` folder that was just extracted, go to the ``applications/`` folder and check out this repository
(instructions below). This will install the Runestone Tools as a web2py application automatically.

Cloning The Runestone Project and its submodules
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This project consists of the main repository, plus *submodules* for codelens, parsons-problems, and skulpt.  In order to get all of the source you need you will need to do the following:

::

    $ git clone https://github.com/bnmnetp/runestone.git
    $ cd runestone
    $ git submodule init
    $ git submodule update

If you are using a GUI git client you may simply get prompted to update the submodules and all will be taken care of for you.  Newer versions of git also support::

    $ git clone --recursive https://github.com/bnmnetp/runestone.git

Configure the Book
------------------

Although the book looks like a static website, there are quite a few AJAX calls going on in the background.  The Javascript relies on a configuration object called eBookConfig.  To get the right values in the eBookConfig object you need to configure a couple of things prior to running the ``paver`` command to build the book.  We have provided a ``paverconfig.py.prototype`` file that you can simply copy to ``paverconfig.py`` and modify.  It contains the following two lines:

::

    master_url = 'http://127.0.0.1:8000'
    master_app = 'runestone'

You can modify the master_url to be the hostname that you are running your app on, but if you are just doing local development you will probably want to leave it alone.  If you omit this step the books will build in the next step using the values above as default values, but you will get a warning message:

::

    'NOTICE:  You are using default values for master_* Make your own paverconfig.py file'


Building the Book
-----------------

Once you the above installed, you can type ``paver allbooks`` from the command
line and that will build the following targets:

* How to Think Like a Computer Scientist
* Problem Solving with algorithms and Data Structures using Python
* An overview page that shows off all the cool features of the Runestone toolkit

The books are built into ``runestone/static/thinkcspy``, ``runestone/static/pythonds`` and ``runestone/static/overview``  assuming that runestone is the name of the folder you cloned into.  When the build is done you can quickly check the build by opening the file ``static/thinkcspy/index.html`` in your browser.

Now before you start web2py its convenient to make runestone the default application.  In the top level web2py directory copy routes.example.py to routes.py and Modify the three lines that contain the word runestone to look like this::

    default_application = 'runestone'    # ordinarily set in base routes.py
    default_controller = 'default'  # ordinarily set in app-specific routes.py
    default_function = 'index'      # ordinarily set in app-specific routes.py

    # routes_app is a tuple of tuples.  The first item in each is a regexp that will
    # be used to match the incoming request URL. The second item in the tuple is
    # an applicationname.  This mechanism allows you to specify the use of an
    # app-specific routes.py. This entry is meaningful only in the base routes.py.
    #
    # Example: support welcome, admin, app and myapp, with myapp the default:


	routes_app = ((r'/(?P<app>welcome|admin|app)\b.*', r'\g<app>'),
	              (r'(.*)', r'runestone'),
	              (r'/?(.*)', r'runestone'))


Running the Server
------------------

You will have to set a few configuration values in the file ``models/1.py``. Copy ``models/1.py.prototype`` to
``models/1.py`` and open the newly created 1.py. If you don't wish to use a local SQLite database, change the
``database_uri`` to match your actual credentials.

If you wish to use Janrain Engage to provide social network authentication integration, you will also have to set your
Janrain API key and domain in 1.py.

Note: If you do *not* wish to use Janrain, you must comment out these lines in ``models/0.db``::

    janrain_form = RPXAccount(request,
                              api_key=settings.janrain_api_key, # set in 1.py
                              domain=settings.janrain_domain, # set in 1.py
                              url=janrain_url)
    auth.settings.login_form = ExtendedLoginForm(auth, janrain_form) # uncomment this to use both Janrain and web2py auth

and uncomment the line below. This will disable Janrain and only use Web2Py integrated authentication. ::

    auth.settings.login_form = auth # uncomment this to just use web2py integrated authentication

Once you've built the book using the steps above.  You can start the web2py development server by simply running ::

    python web2py.py

This will bring up a little GUI where you can make up an admin password and click "start server".
When the server is running your browser will open to the welcome application, unless you've changed
the default application as described above.  To see this app simply use the url:  http://127.0.0.1/runestone
From there, you can click on the link for "How To Think Like A Computer Scientist" or "Problem Solving With
Algorithms and Data Structures". (See the section Final Configuration below for instructions on registering
for one of the courses. Registering allows you to save your progress and work.)

If you get an error at this point the most likely reason is that the settings file isn't recognizing your host and is not setting the database correctly.  These lines in models/0.py are important::

    if 'local' in uname()[1] or 'Darwin' in uname()[0]:
        settings.database_uri = 'sqlite://storage.sqlite'
    elif 'webfaction' in uname()[1]:  # production is on webfaction
        settings.database_uri = 'postgres://production_db:secret@production_server.com/production_db'
    elif 'luther' in uname()[1]:   # this is my beta machine
        settings.database_uri = 'sqlite://storage.sqlite'
    else:
        raise RuntimeError('Host unknown, settings not configured')

For your own personal development, you want the first clause of the if statement to match. If you are on a Unix-like system,
you can replace 'Darwin' with the result of running ``uname`` at a terminal. Another option is to replace 'local' with
your computer's hostname.

Final Configuration
-------------------
To use the admin functionality you are going to want to do one more bit of configuration:

* Click the "Register" link in the user menu in the upper right corner of the browser window.
* Fill in the form to create a user account for yourself. You can register for either "How To Think..." (use the course name ``thinkcspy``) or "Problem Solving With..." (use the course name ``pythonds``).

Now, add your new user account to the 'instructors' group using the appadmin
functionality of web2py:

* Open ``http://127.0.0.1:8000/runestone/appadmin``. Login using the password you supplied when you ran web2py.
* Click on ``insert new auth_membership``. Select your user account and the instructor group as the two values and click submit.  You are now an instructor.


How to Contribute
-----------------

#. Get a github (free) account.
#. Make a fork of this project.  That will create a repository in your
   account for you to have read/write access to.  Very nice, complete
   instructions for making a fork are here:  ``https://help.github.com/articles/fork-a-repo``
#. Clone the repository under your account to your local machine.
#. Check the issues list, or add your own favorite feature.  commit and pull to your fork at will!
#. test
#. Make a Pull Request.  This will notify me that I should look at your changes and merge them into the main repository.
#. Repeat!


How to Contribute $$
--------------------

As our popularity has grown we have server costs.  We
were also able to make great progress during the Summer of 2013
thanks to a generous grant from ACM-SIGCSE that supported one of our
undergraduate students. It would be great if we could have a student
working on this all the time.

If this system or these books have helped you, please consider making a small
donation using `gittip <https://www.gittip.com/bnmnetp/>`_


More Documentation
------------------

I have begun a project to document the `Runestone Interactive <http://docs.runestoneinteractive.org/build/html/index.html>`_ tools

* All of the Runestone Interactive extensions to sphinx:

    * Activecode -- Interactive Python in the browser
    * Codelens  -- Step through code examples and see variables change
    * mchoicemf  -- multiple choice questions with feedback
    * mchoicema  -- multiple choice question with multiple answers and multiple feedback
    * fillintheblank  -- fill in the blank questiosn with regular expression matching answers
    * parsonsproblem  -- drag and drop blocks of code to complete a simple programming assignment
    * datafile -- create datafiles for activecode

* How to write your own extension for Runestone Interactive


Creating Your Own Textbook
--------------------------

To find instructions on using the Runestone Tools to create your own interactive textbook, see the
file in this directory named README_new_book.rst.


Browser Notes
-------------

Note, because this interactive edition makes use of lots of HTML 5 and Javascript
I highly recommend either Chrome, or Safari.  Firefox 6+ works too, but has
proven to be less reliable than the first two.  I have no idea whether this works
at all under later versions of Internet Explorer.

