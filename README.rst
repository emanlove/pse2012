The following are resources, links and other notes relating to `my talk <http://www.justin.tv/weblionab/b/319303907>`_ given at the Plone East Symposium held at Penn State, May - 2012. If any information is missing or inaccurate please feel free to correct me.

Ed Manlove

email: devPyPlTw@verizon.net

#plone irc: t55e

Recorded Talk: http://www.justin.tv/weblionab/b/319303907

Table of Contents
~~~~~~~~~~~~~~~~~
1. `Post-symposium follow-up`_

   a. `Running Robot Framework with Plone's Testing Framework and Plone.App.Testing`_
   b. `Selenium RC and Selenium IDE`_
   c. `Using both Python and Robot Framework`_

2. `Steps to get started with Plone UI Testing using Robot Framework`_
#. `Plone Testing, Javascript Statistics, QUnit`_
#. `Selenium`_
#. `Page Object Model`_
#. `Robot Framework`_
#. `Using buildout and testing Plone core information`_
#. `Others work with Selenium / Robot Framework / Plone`_
#. `(Personal) Roadmap for Plone Selenium/Robot Framework Testing`_
#. `Quick links`_

Post-symposium follow-up
~~~~~~~~~~~~~~~~~~~~~~~~

After my symposium talk I had several conversations with different people concerning a couple of topics.  I wanted to share some more information about those topics

Running Robot Framework with Plone's Testing Framework and Plone.App.Testing
----------------------------------------------------------------------------

As noted in my talk, one of the drawbacks with robotframework is that the testing framework for robotframework is seperate from Plone's testing framework.  This means there is no utility keywords that provide the tester with convenient ... like... . Also you need to run the Plone instance in addition to running the robotframework test script. You can compare this to Plone's testing framework which provides both a single test script as well as a useful set of utilities. Thus it would be desirable to integrate both frameworks.

There are essentially two ways of integrating the two frameworks; have robotframework encapsulate `plone.testing (http://pypi.python.org/pypi/plone.testing)`_ and `plone.app.testing (http://pypi.python.org/pypi/plone.app.testing/4.2)`_, which makes up the core of Plone's testing framework, or have plone.app.testing encapsulate robotframework. Both approaches have put forth by two members of the Plone community, Godefroid Chapelle (aka gotcha) and Asko Soukka (aka datakurre). 

Asko has authored `robotsuite (https://github.com/datakurre/robotsuite)`_ which he describes as a "Python unittest test suite for Robot Framework". Here he is having plone.testing and plone.app.testing encapsulate or wrap robotframework.

In comparison Godefroid is using robotframework to encapsulate plone.app.testing in his package `plone.act (https://github.com/gotcha/plone.act)`_.

***ToDo*** Write more on these two approaches.

***Note*** The following instructions are rough steps for getting started. I wanted to get this information out there sooner than later.  I will polish these steps as I and others use them more.

Step by Step instructions for incorporating robotsuite
......................................................

Setup virtual environment and sandbox 

::

    # mkdir robotsuite
    # cd robotsuite
    # virtualenv -p /usr/bin/python2.6 --no-site-packages clean-python26-env
    # git clone -b 4.3  https://github.com/plone/buildout.coredev ./p43
    # source clean-python26-env/bin/activate


Add configuration file pybot.cfg

::

    [buildout]
    
    extends = buildout.cfg
    
    parts +=
        robot
    
    allow-hosts +=
        *.googlecode.com
        code.google.com
    
    auto-checkout +=
        robotframework-selenium2library
        robotsuite
    
    [sources]
    robotframework-selenium2library = git git@github.com:rtomac/robotframework-selenium2library
    robotsuite = git git@github.com:datakurre/robotsuite
    [robot]
    recipe = zc.recipe.egg
    eggs = robotframework [test]
           robotframework-selenium2library [test]
           robotsuite
    entry-points =
        pybot=robot:run_cli
        rebot=robot.rebot:rebot_cli
    arguments = sys.argv[1:]


Run buildout

::

    # cd p43/
    # python bootstrap.py
    # bin/buildout -c pybot.cfg


Create test layer and unit test suite within ./src/Products.CMFPlone/Products/CMFPlone/tests/testPloneAcceptance.py

::

    import unittest2 as unittest
    import time
    
    from plone.testing import layered
    from robotsuite import RobotTestSuite
    
    from plone.testing import Layer
    from plone.app.testing import PLONE_FIXTURE
    from plone.app.testing import FunctionalTesting
    from plone.testing import z2
    
    class RobotFrameworkLayer(Layer):
        defaultBases = (z2.ZSERVER_FIXTURE, )
    
        def testSetUp(self):
            pass
    
    ROBOT_FIXTURE = RobotFrameworkLayer()
    ROBOT_PLONE_FUNCTIONAL_TESTING = FunctionalTesting(
        bases=(ROBOT_FIXTURE, PLONE_FIXTURE),
        name="RobotTesting:Functional")
    ROBOT_TESTING = FunctionalTesting(
        bases=(PLONE_FIXTURE, z2.ZSERVER_FIXTURE),
        name="PloneFixture:Robot")
    
    
    def test_suite():
        suite = unittest.TestSuite()
        suite.addTests([
            layered(RobotTestSuite('acceptance-tests'),
                    layer=ROBOT_PLONE_FUNCTIONAL_TESTING),
        ])
        return suite
    
    if __name__ == '__main__':
        unittest.main(defaultTest='test_suite')


Copy over acceptance tests from https://github.com/emanlove/buildout.coredev/tree/4.1-robot/acceptance-tests to ./src/Products.CMFPlone/Products/CMFPlone/tests/acceptance-tests
***NOTE - VERY IMPORTANT*** plone.app.testing's PLONE_FIXTURE layer puts the site root at localhost:55001/plone as in lower-case 'p' plone. In addition to copying over the tests ***YOU NEED TO MODIFY*** the variable ${PLONE_URL} to ${ZOPE_URL}/plone (noting the lower case 'p') within ./src/Products.CMFPlone/Products/CMFPlone/tests/acceptance-tests/plone.txt. Otherwise when you run the tests you will see an error similar to 'Resource not found ... Resource: no default view (root default view was probably deleted)'. I'm not sure why Firefox is case sensitive and I will try to eliminate this step by pulling in this variable.

Run tests

::

    # bin/test -s Products.CMFPlone -m testPloneAcceptance


Step by Step instructions for incorporating plone.act
.....................................................

Setup virtual environment and sandbox 

::

    # mkdir act
    # cd act
    # virtualenv -p /usr/bin/python2.6 --no-site-packages clean-python26-env
    # git clone -b 4.3  https://github.com/plone/buildout.coredev ./p43
    # source clean-python26-env/bin/activate
    # easy_install PIL

Add configuration file pybot.cfg

::

    [buildout]
    
    extends = buildout.cfg
    
    parts +=
        robot
    
    allow-hosts +=
        *.googlecode.com
        code.google.com
    
    auto-checkout +=
        robotframework-selenium2library
        plone.act
    
    [sources]
    robotframework-selenium2library = git git@github.com:rtomac/robotframework-selenium2library
    plone.act = git git@github.com:gotcha/plone.act
    [robot]
    recipe = zc.recipe.egg
    eggs = robotframework
           robotframework-selenium2library
           plone.act
    entry-points =
        pybot=robot:run_cli
        rebot=robot.rebot:rebot_cli
    arguments = sys.argv[1:]

Run buildout

::

    # cd p43/
    # python bootstrap.py
    # bin/buildout -c pybot.cfg

plone.act configures the default port for the plone instance at 5000 so we adjust our environment accordingly

::

    # export ZSERVER_PORT=5000

Run tests

::

    # ./bin/pybot src/plone.act/acceptance-tests/


Selenium RC and Selenium IDE
----------------------------

Several people have asked about several other selenium related projects, Selenium RC and Selenium IDE. Selenium RC or Remote Control is essentially Selenium 1 and, `according to the SeleniumHQ FAQ <http://seleniumhq.org/docs/01_introducing_selenium.html#selenium-1-aka-selenium-rc-or-remote-control>`_, is being maintained but not actively developed.  This makes sense as most development is on the newer Selenium 2/WebDriver project.

`Selenium IDE <http://seleniumhq.org/projects/ide/>`_ is a simplified testing enviroment, for Firefox only, which uses neither Selenium 1 nor Selenium 2/WebDriver but creates test "scripts" which then can be rerun through, and only through, the IDE. Let me quote verbatim from the FAQ ...

    "Even though Selenium IDE has a “Save” feature that allows users to keep the tests in a table-based format for later import and execution, it is not designed to run your test passes nor is it designed to build all the automated tests you will need. Specifically, Selenium IDE doesn’t provide iteration or conditional statements for test scripts. At the time of writing there is no plan to add such thing. The reasons are partly technical and partly based on the Selenium developers encouraging best practices in test automation which always requires some amount of programming. Selenium IDE is simply intended as a rapid prototyping tool. The Selenium developers recommend for serious, robust test automation either Selenium 2 or Selenium 1 to be used with one of the many supported programming languages."

The context in which this was asked involved a large Plone installation with just two main developers, a third part-time developer, some designers with no programming experience and many in-house customers (non-programers, non-designers) who seemed to constantly report unreproducable issues. It was asked if IDE could be used to help reproduce these phantom issues. If the end users are working within the limited system requirements (ie Firefox) such a setup might be useful.  But again it is limited in capabilities and we should be aware of this.

Using both Python and Robot Framework
-------------------------------------

Again within the context of having an organization with both python programmers and no-python personnal like designers is using python and robot framework an either/or situation.  The answer is you can use both python and robot framework together. As robot framework is written in python, it allows you to create user keywords in python code. So using best practices we can create python page objects and then incorporate them as keywords within the robot framework tests. (Adam Goucher provides `an example <https://github.com/adamgoucher/robotframework-pageobjects>`_ or using python page objects with robot framework on `his github site <https://github.com/adamgoucher/robotframework-pageobjects>`_.) For more information on writing python-based keywords read the `section on variable files <http://robotframework.googlecode.com/hg/doc/userguide/RobotFrameworkUserGuide.html?r=2.7.1#resource-and-variable-files>`_ within the robot framework user's guide.

Steps to get started with Plone UI Testing using Robot Framework
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

First we will start by installing Plone. The recommended method for installing Plone for general users is to use the installer which can be found at http://plone.org/products/plone/.  The `Installation Quick Guide <http://plone.org/documentation/manual/installing-plone/installation-quick-guide>`_ provides basic instructions to get Plone downloaded, installed and started. (For more detailed installation and startup instructions see the guide for either '`Linux / Unix / OS X / BSD <http://plone.org/documentation/manual/installing-plone/installing-on-linux-unix-bsd>`_' or '`Windows OS <http://plone.org/documentation/manual/installing-plone>`_')

Once you have Plone installed you need to install two Python packages, robotframework and robotframework-selenium2library, and their dependencies, decorator, docutils, selenium, each a python package. (Note both the docutils and selenium python packages might be installed by the current plone installer used above and might not need to be reinstallled.) The best method to install these packages is to use Pip which is a python package install utility. More information about Pip and how to install packages using Pip can be found `here <http://guide.python-distribute.org/installation.html>`_. Alternatively you may need to use easy_install.

If installing robotframework package on Windows you may also need to run robot_postinstall.py (which can be found ?).  For detailed robotframework installation instructions for windows and other OSes see the `documentation <http://code.google.com/p/robotframework/wiki/Installation>`_.

If you did not create a Plone site when you downloaded and installed the Plone software then you should do so at this time and start it running. (Note installing the Plone software package does not neccessarly create a Plone site. Once Plone is running you should see "Welcome to Plone" on the main page of the Plone webite. If not review the quick install guide).

Now one can get started by writing tests. There is detailed information with the `Robot Frameworks's User Guide <http://robotframework.googlecode.com/hg/doc/userguide/RobotFrameworkUserGuide.html?r=2.7.1>`_ but let us write simple test together first. The first thing one needs to decide is where to put the test we are going to write.  There are several options for where to put your tests but for now we will put them in a directory called 'acceptence-tests' on the C drive (Linux / Mac OSX users may put the 'acceptance-tests' directory in an appropriate place on their systems). To create this new directory type the following at a DOS prompt

::

    C:\> mkdir acceptence-tests

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

Website: http://code.google.com/p/selenium/

Selenium 1.0 or Selenium was basically a javascript library where as WebDriver/Selenium2 actually controls the browser itself using the JSON Wire Protocal.

Plone has had a few selenium test for a while. Selenium 2 / Webdriver versions can be found at https://github.com/plone/Products.CMFPlone/tree/selenium-integration/Products/CMFPlone/tests/selenium

Other parts of Plone that have, very few at the moment, selenium tests:

- ploe.app.deco  https://github.com/plone/plone.app.deco/tree/742150c5018c6bfd67cfb8c0c3f5c48be1aa32cf/plone/app/deco/tests/selenium

- plone.app.jquerytools  (`'update-selenium-tests' branch <https://github.com/plone/plone.app.jquerytools/tree/update-selenium-tests/plone/app/jquerytools/tests/selenium>`_) (`'master' branch <https://github.com/plone/plone.app.jquerytools/tree/master/plone/app/jquerytools/tests/selenium>`_)

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

The following steps add robotframework and robotframework-selenium2library to buildout. These were written using Linux and should be adjusted accordingly to your OS. (Please send me any OS specific differences so I can add them here). I have used `Buildout.coredev <https://github.com/plone/buildout.coredev/>`_ branches 4.1, 4.2, and 4.3 in the following steps for my testing setup. The goal is to easily demonstrate robot framework/selenium testing with Plone so any recommendations for better buildout configuration is welcomed.

1. In the buildout directory create a file entitled, pybot.cfg, and place the following into that file,

::

    [buildout]
    
    extends = buildout.cfg
    
    parts +=
        robot
    
    allow-hosts +=
        *.googlecode.com
        code.google.com
    
    auto-checkout +=
        robotframework-selenium2library
    
    [sources]
    robotframework-selenium2library = git git@github.com:rtomac/robotframework-selenium2library
    
    [robot]
    recipe = zc.recipe.egg
    eggs = robotframework
           robotframework-selenium2library
           plone.act
    entry-points =
        pybot=robot:run_cli
        rebot=robot.rebot:rebot_cli
    arguments = sys.argv[1:]

The entry_points and arguments attributes are necessary because robotframework does not currently configure its scripts properly under buildout.  Thanks to Héctor Velarde and Mikko Ohtama for the complete solution. Godefroid also has presented an alternative solution `here <https://github.com/gotcha/robotentrypoints>`_.

2. Run buildout

::
    
    ~/plone42$ ./bin/buildout

3. Create tests. Or better yet start of by copying `my fork of the acceptance tests <https://github.com/emanlove/buildout.coredev/tree/4.1-robot/acceptance-tests>`_ created at the last Plone Conference sprint. The tests in my fork have been converted to use the newer Selenium 2 library.

These test were placed in directory just below the buildout directory, called ./acceptance-tests. This choice has been made and copied by several Plone developers as a simple hack if you will.  Robot framework does not have a test discovery recipe as does Plone testing.  Thus we have used this current solution and we always welcome a better test discovery recipe.

4. Run your tests

::

    ~/plone42$ ./bin/pybot acceptance-tests/

Others work with Selenium / Robot Framework / Plone
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

I was not the first nor only person working to test Plone using either selenium or robot framework.  Here is a list of others and some links to their work.

Godefroid Chapelle (gotcha)
    has been a strong advocate for robotframework and for writing tests for Plone's javascript/UI code.
    
    - https://github.com/gotcha/plone-robot-documentation

    - https://github.com/gotcha/robotentrypoints

    - https://github.com/gotcha/plone.act

Asko Soukka (datakurre)
    has done great exploritory work with robotframework and Plone as well as other robotframework library. Also Asko is looking at wrapping robotframework into unittest

    - https://github.com/datakurre/robotsuite

    - https://github.com/datakurre/corejet.robot

    - https://github.com/datakurre/phantomrobot

Héctor Velarde (hvelarde)
    has mentioned that he is going to look into multilingual capability for robotframework

    - https://github.com/hvelarde

    - http://hvelarde.blogspot.com/

.. Note to self: I wonder how mutli-ligual solutions might work to solve
   the issue of test fagility/page object solutions and strings like
   "We’re sorry, but there seems to be an error" which can be seen if you
   traverse to http://plone.org/search_form

Mikko Ohtama (moo-_-)
    writing some selenium tests and base page objects for plone

(Personal) Roadmap for Plone Selenium/Robot Framework Testing
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Here is my personal list of To Do's for Plone Selenium/Robot Framework Testing

- work towards the 1.1 release of `robotframework-selenium2libary <https://github.com/rtomac/robotframework-selenium2library>`_
- document, document document
- work with datakurre on p.a.testing wrapper
- work with hvelarde on multilingual robot
- work on `PLIP11708 <https://dev.plone.org/ticket/11708>`_
- work to standardize and create page objects for Plone
- work with Matthew, Michelle and WebLion to create accessibility tests
- work with Eric and Timo to get selenium/rf into Jenkins
- ...


Quick links
~~~~~~~~~~~

Keyword Documentation for ...

robotframework-selenium2library http://rtomac.github.com/robotframework-selenium2library/doc/Selenium2Library.html

robotframework  http://code.google.com/p/robotframework/wiki/TestLibraries

Sample test written for Plone (using selenium2library)  https://github.com/emanlove/buildout.coredev/tree/4.1-robot/acceptance-tests

Recorded Talk: http://www.justin.tv/weblionab/b/319303907
