Please forgive the poor formatting as GITHUB and I are having disagreements of proper formating of rst text. - Ed

README
~~~~~~

The following are resources, links and other notes relating to my talk given at the Plone East Symposium held at Penn State, May - 2012.

If any information is missing or inaccurate please feel free to correct me.

Ed Manlove

email: devPyPlTw@verizon.net

#plone irc: t55e

Table of Contents
~~~~~~~~~~~~~~~~~
1. `Steps to get started with Plone UI Testing using Robot Framework`_
#. `Plone Testing, Javascript Statistics, QUnit`_
#. `Selenium`_
#. `Page Object Model`_
#. `Robot Framework`_
#. `Using buildout and testing Plone core information`_

Steps to get started with Plone UI Testing using Robot Framework
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

First we will start by installing Plone. The recommended method for installing Plone for general users is to use the installer which can be found at http://plone.org/products/plone/.  The `Installation Quick Guide <http://plone.org/documentation/manual/installing-plone/installation-quick-guide>`_ provides basic instructions to get Plone downloaded, installed and started. (For more detailed installation and startup instructions see the guide for either '`Linux / Unix / OS X / BSD <http://plone.org/documentation/manual/installing-plone/installing-on-linux-unix-bsd>`_' or '`Windows OS <http://plone.org/documentation/manual/installing-plone>`_')

Once you have Plone installed you need to install two Python packages, robotframework and robotframework-selenium2library, and their dependencies, decorator, docutils, selenium, each a python package. (Note both the docutils and selenium python packages might be installed by the current plone installer used above and might not need to be reinstallled.) The best method to install these packages is to use Pip which is a python package install utility. More information about Pip and how to install packages using Pip can be found `here <http://guide.python-distribute.org/installation.html>`_. Alternatively you may need to use easy_install.

If installing robotframework package on Windows you may also need to run robot_postinstall.py (which can be found ?).  For detailed robotframework installation instructions for windows and other OSes see the `documentation <http://code.google.com/p/robotframework/wiki/Installation>`_.

If you did not create a Plone site when you downloaded and installed the Plone software then you should do so at this time and start it running. (Note installing the Plone software package does not neccessarly create a Plone site. Once Plone is running you should see "Welcome to Plone" on the main page of the Plone webite. If not review the quick install guide).

Now one can get started by writing tests. There is detailed information with the `Robot Frameworks's User Guide <http://robotframework.googlecode.com/hg/doc/userguide/RobotFrameworkUserGuide.html?r=2.7.1>`_ but let us write simple test together first. The first thing one needs to decide is where to put the test we are going to write.  There are several options for where to put your tests but for now we will put them in a directory called 'acceptence-tests' on the C drive (Linux / Mac OSX users may put the 'acceptance-tests' directory in an appropriate place on their systems). To create this new directory type the following at a DOS prompt

::

    C:\\> mkdir acceptence-tests

Using your prefered editor copy the following into a file called "hello_world.txt" (without qoutes)

::

    *** Settings ***
    Library     Selenium2Library  timeout=0 seconds  implicit_wait=5 seconds
    
    
    *** Test cases ***
    
    Hello World
        Go to  http://plone.org
        Click link  Get Involved
    
        Page should contain  Help make Plone even better
        Page should contain  Create bug fixes, develop new features

    Hello Plone
        Go to  http://localhost:8080/Plone

        Page should contain  Welcome to Plone


It is important to note that there is a double space between the keywords and the arguements in robot framework test files.  For example there is two spaces between "Go to" and "http://plone.org" as well two spaces between the keyword "Page should contain" and "Welcome to Plone". The `user's guide <http://robotframework.googlecode.com/hg/doc/userguide/RobotFrameworkUserGuide.html?r=2.7.1#creating-test-data>`_ talks more about how test case files should be formatted. Save this file, again with the filename 'hello_world.txt', within the newly created test directory 'acceptence-tests'.

::

    C:\> cd C:\acceptence-tests
    C:\acceptence-tests> dir

    hello_world.txt
    
To run this test we will use the executable or script called 'pybot' provided by robot framework. The robot framework installation should have put the executable or script in you path such that you should be able to call the executable fromany directory. You can verify this by calling the pybot executable from the root C drive.

::

    C:\acceptence-tests> cd c:\
    C:\> pybot

If you get a 'File not found.' error then you will have to either specify the full path to the pybot.exe file or change into directory containg pybot.exe. We will proceed as if the executable was within your path and was found.

To run our sample test type the following at the dos prompt,

::

    C:\> pybot acceptence-tests\

and watch robot framework and selenium run your first UI test. You may create more tests using keywords from the `robotframework-selenium2library <http://rtomac.github.com/robotframework-selenium2library/doc/Selenium2Library.html>`_ to test out your Plone site's UI.


Plone Testing, Javascript Statistics, QUnit
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For introduction to testing Plone see http://plone.org/documentation/manual/developer-manual/testing

You should also read http://pypi.python.org/pypi/plone.testing

The functional browser testing which Plone performs is done using zope.testbrowser (see http://pypi.python.org/pypi/zope.testbrowser). You will see code like::

    >>> from Testing.testbrowser import Browser
    >>> browser = Browser()

This is an programmable HTML browser which emulates using a browser like Firefox or IE. As we will see using Selenium 2 / WebDriver the test will communicate directly with the actual browser and the browser will respond just as if you were interacting with it.

For more statistics about Plone internals see http://www.ohloh.net/p/plone

To search through the Plone's github repositories goto github's Advanced Search page (the little gear symbol next to the search box) and search using the search term "repo:plone/* <searchterm>" AND set "Search for" to "Code". For example to search for "Testing.testbrowser" try::

     repo:plone/* Testing.testbrowser

QUnit is a javascript test suite. See http://docs.jquery.com/Qunit


Selenium
~~~~~~~~

Website: 

Selenium 1.0 or Selenium was basically a javascript library where as WebDriver/Selenium2 actually controls the browser itself using the JSON Wire Protocal.

Plone has had a few selenium test for a while. Selenium 2 / Webdriver versions can be found at https://github.com/plone/Products.CMFPlone/tree/selenium-integration/Products/CMFPlone/tests/selenium


Page Object Model
~~~~~~~~~~~~~~~~~

The page object model is a method for seperating the objects that will be tested from the tests.  So if the underlying objects change, for example the element's id changes, then one can easily change the locator without having to modify the test.

Adam Goucher has both writen spoken often about using the Page Object model when writing Selenium tests. For a good introduction to Page Objects and the arguement for them see his article

  "Page Objects in Python" http://pragprog.com/magazines/2010-08/page-objects-in-python

and for a good broad overview of best practices using Selenium listen to a folow-up interview done with Adam after the 2011 Selenium Conference at Push to Test website,

  http://www.pushtotest.com/selenium-you-are-doing-it-wrong

To see several examples of page objects within Plone see https://github.com/plone/plone.seleniumtesting/tree/master/plone/seleniumtesting 

Robot Framework
~~~~~~~~~~~~~~~

Website: http://code.google.com/p/robotframework/

User Guide (latest): http://robotframework.googlecode.com/hg/doc/userguide/RobotFrameworkUserGuide.html?r=2.7.1

Selenium2Library: http://rtomac.github.com/robotframework-selenium2library/doc/Selenium2Library.html

A list of the standard test libraries for robot framework can be found at http://code.google.com/p/robotframework/wiki/TestLibraries

Several robot framework tests were written during the post-Plone 2011 Conference sprints and can be found at

  https://github.com/emanlove/buildout.coredev/tree/4.1-robot/acceptance-tests

An example of using the Page Object method with Robot Framework can be found on Adam Goucher's github site. See https://github.com/adamgoucher/robotframework-pageobjects

Plone's robot framework documentation can be found at https://github.com/gotcha/plone-robot-documentation

Using buildout and testing Plone core information
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~



Steps to add robotframework and robotframework-selenium2library to buildout. Note these were written using Linux and should be adjusted accordingly to your OS. (Please send me any OS specific differences so I can add them here).

0. Note I have used `Buildout.coredev <https://github.com/plone/buildout.coredev/>`_ branches 4.1, 4.2, and 4.3 in the following steps for my testing setup. Also I have placed my eggs directly into core.cfg, checkouts.cfg, and sources.cfg which might not be best practices.  Right now I am working on easily demonstarting robot framework/selenium testing with Plone.  Any recommendations for better buildout configuration is welcomed.

1. Add the following to ...

core.cfg
::

    [buildout]
    parts =
        ...
        robot
    
    
    [robot]
    recipe = zc.recipe.egg
    eggs = robotframework
           robotframework-selenium2library
    entry-points = 
        pybot=robot:run_cli
        rebot=robot.rebot:rebot_cli


checkouts.cfg 
::

    auto-checkout =
        ...
        robotframework-selenium2library

sources.cfg
::

    [sources]
    ...
    robotframework-selenium2library     = git git://github.com/emanlove/robotframework-selenium2library.git

Note I am currently pulling in robotframework-selenium2library from my github repository for reasons of installation dependencies and this may change in the soon future (End of May 2012).

2. Run buildout

::
    
    ~/plone42$ ./bin/buildout

3. In the buildout bin directory edit the script 'pybot'

::

    ...
    
    import robot
    
    if __name__ == '__main__':
        robot.run_cli(sys.argv[1:])

This is neccessary because robotframework does not currently configure its scripts properly under buildout.  This is partially fixed by the entry_points attribute in core.cfg above.  By the script requires an argument (sys.argv[1:]) which is not allowed by entry_points or buildout.  So this poor workaround is required. Godefroid also has presented an alternative solution `here <https://github.com/gotcha/robotentrypoints>`_.

4. Create tests. Or better yet start of by copying the selenium2library based tests created at the last Plone Conference sprint which can be found `here <https://github.com/emanlove/buildout.coredev/tree/4.1-robot>`_.

These test were placed in directory just below the buildout directory, called ./acceptance-tests. This choice has been made and copied be several Plone developers as a simple hack if you will.  Robot framework does not have a test discovery recipe as does Plone testing does with the eggs in Plone buildout.  Thus we have used this ease of use solution.  But it is temporary and we always welcome a better test discovery recipe.

5. Run your tests

::

    ~/plone42$ ./bin/pybot acceptance-tests/