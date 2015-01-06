---
layout: post
title: Introduction to Unit Testing the Zend Framework Quickstart Application
date: 2009-02-23 16:11:48.000000000 +00:00
categories: web-dev
tags:
- php
- phpunit
- testing
- unit testing
- web dev
- zend framework
permalink: /2009/02/23/introduction-to-unit-testing-the-zend-framework-quickstart-application/
---
**Update:** Things are all change in Zend Framework 1.8, so this may require some adaptation in that version.

If you've followed the Zend Framework quickstart tutorial like me, you may be wondering how to perform unit testing using it. The tutorial does set up the bootstrap file to allow for unit testing, but does not cover unit testing itself. The aim of this post is to  provide a quick guide to unit testing using the quickstart application, which will hopefully come in useful for a few people and myself when I inevitably forget this later on.

# Step 0: Before Starting

First, you should download and set up the guestbook application from the [Zend Framework website](http://framework.zend.com/docs/quickstart "Zend Framework quickstart tutorial"). Make sure you place the Zend library in the library folder and enable the read/write permissions for the sqlite database and the db directory for access by the web server.

You will also want to install pear and, with it, install PHPUnit:

> pear channel-discover pear.phpunit.de
>
> pear install phpunit/PHPUnit

# Step 1: Create Test Folders

Create a folder called "tests" in the root folder of the application. Inside that create a folder called "controllers" and "models". The  directory structure should be as follows:

<pre>zendquickstart
|-- application
|-- data
|-- library
|-- public
|-- scripts
`--tests
 |-- controllers
 `-- models</pre>

All your controller tests should go in the controllers folder, all your model tests should go in the models folder.

# Step 2: Create Test Configuration

Create the file TestConfiguration.php in the tests folder. This file will initially set up the testing configuration, inside place the following code:

{% highlight php linenos %}
<?php
// Gets called when this file is included/required
TestConfiguration::setUp();

class TestConfiguration
{
    /**
    * Sets the environment up for testing
    */
    static function setUp()
    {
        // Set the environment constant to testing which will load the testing
        // configuration in app.ini by the bootstrap
        define('APPLICATION_ENVIRONMENT', 'testing');

        // Set the include path for locating the Zend library
        set_include_path(realpath(dirname(__FILE__)) . '/../library'
            . PATH_SEPARATOR . get_include_path());

        // Use Autoload so that we don't have to include/require every class
        require_once "Zend/Loader.php";
        Zend_Loader::registerAutoload();
    }

    static function setUpDatabase()
    {
        require '../application/bootstrap.php';

        $db = Zend_Registry::get('configuration')->database->params->dbname;

        // delete any pre-existing databases
        if(file_exists($db)) unlink($db);

        // run the database set up script to recreate the database
        require '../scripts/load.sqlite.php';
    }
}
?>
{% endhighlight %}

The setUp() function in this class will get called when the file is included or required. The APPLICATION_ENVIRONMENT constant is set to 'testing' so that the testing section of the app.ini file is read in the bootstrap, and the testing database is used. A testing database should be used so that testing does not interfere with the production or development databases, and no important  data is lost. The next step is to set the include path, which should contain the location of the Zend Framework library. The final step is to enable the Zend autoloader so that we don't have to explicitly require/include the Zend libraries.

The setUpDatabase() function resets the database to a known state. When the tests are run we may wish to insert data to the database and this should be removed before every new test. We require the bootstrap file so we can get the database configuration and delete the database file, which is an easy way to reset an SQLite database. A different method will be necessary for other databases like MySQL or PostgreSQL. Finally, the setUpDatabase() function uses a script which comes with the quick start application to (re)create the database.

# Step 3: Creating Controller Tests

To test the IndexController, create the file IndexControllerTest.php in the tests/controller directory. The following is a simple example:

{% highlight php linenos %}
<?php
// Set up the testing environment
require 'TestConfiguration.php';

class controllers_IndexControllerTest extends Zend_Test_PHPUnit_ControllerTestCase
{
    // Bootstraps the application
    public $bootstrap = '../application/bootstrap.php';

    public function testHomePageIsASuccessfulRequest()
    {
        // Runs the test on /, the homepage
        $this->dispatch('/');

        // Tests there are no exceptions on the home page
        $this->assertFalse($this->response->isException());

        // Tests for redirection to the error handler
        $this->assertNotRedirect();
    }

    public function testHomePageDisplaysCorrectContent()
    {
        // Runs the test on /
        $this->dispatch('/');

        // Tests the page title is present
        $this->assertQueryContentContains(
            'div#header-logo',
            'ZF Quickstart Application'
        );

        // Tests the guestbook link is present
        $this->assertQueryContentContains('a', 'Guestbook');
    }
}
?>
{% endhighlight %}

This is a very basic controller test. First note that you should extend Zend_Test_PHPUnit_ControllerTestCase for controller tests. This class extends PHPUnit_Framework_TestCase itself, but adds some assertions and other things specific to Zend Framework.

The bootstrap instance variable must be set in order to test the controller, and in this case it points to the location of the bootstrap file.

The testHomePageIsASuccessfulRequest() test is used to test that the homepage functions correctly; it should not contain any exceptions or redirect to the error controller. If it does then there is a problem somewhere and the test fails. The testHomePageDisplaysCorrectContent() is used to test that the page title and the link to the guestbook is present.

# Step 4: Creating Model Tests

To test the GuestBook model, create the file GuestBookTest.php in tests/models.

{% highlight php linenos %}
<?php
require 'TestConfiguration.php';
require '../application/models/GuestBook.php';

class models_GuestBookTest extends PHPUnit_Framework_TestCase
{
    public function setUp()
    {
        // Reset database state
        TestConfiguration::setUpDatabase();
    }

    public function testFetchEntries()
    {
        // Instantiate the GuestBook model
        $guestBook = new Model_GuestBook();

        // Get all entries from the database
        $entries = $guestBook->fetchEntries();

        // Test that there are 2 entries in the guestbook
        $this->assertSame(2, count($entries));
    }
}
?>
{% endhighlight %}

Again, this is very basic. The setUp() function gets called by PHPUnit before the test is run, and it will reset the database. The testFetchEntries() function tests the fetchEntries() function in the model. Two rows in the guestbook table should be present, therefore there should be two elements in the array returned by fetchEntries().

# Step 5: Edit load.sqlite.php

You may find it annoying to have to wait 5 seconds between model tests when the database is recreated. I worked around this problem by making a small change to the PHP script which loads the database schema:

{% highlight php linenos %}
<?php
if (APPLICATION_ENVIRONMENT != 'testing')
{
    echo 'Writing Database Guestbook in (control-c to cancel): ' . PHP_EOL;
    for ($x = 5; $x > 0; $x--) {
        echo $x . "\r"; sleep(1);
    }
}
?>
{% endhighlight %}

If the application environment is not testing then the timeout should not be displayed. You may prefer to simply remove the timeout altogether. Additionally, you will need to change the lines that locate the SQL files to include dirname(__FILE__):

{% highlight php linenos %}
<?php
$schemaSql = file_get_contents(dirname(__FILE__) . '/schema.sqlite.sql');
$dataSql = file_get_contents(dirname(__FILE__) . '/data.sqlite.sql');
?>
{% endhighlight %}

# Step 6: Running the Tests

To run the tests, navigate to the tests directory in a terminal/CMD, and type phpunit controllers_IndexControllerTest and models_GuestBookTest to test the IndexController and GuestBook model, respectively.

> phpunit controllers_IndexControllerTest
>
> PHPUnit 3.3.14 by Sebastian Bergmann.
>
> ..
>
> Time: 0 seconds
>
> OK (2 tests, 4 assertions)
>
> phpunit models_GuestBookTest
>
> PHPUnit 3.3.14 by Sebastian Bergmann.
>
> Database Created
>
> Data Loaded.
>
> .
>
> Time: 0 seconds
>
> OK (1 test, 1 assertion)

# Conclusion

There you have it. This isn't necessarily the only way or the best way to perform unit testing, this is merely the way I got it to work with the quickstart application having had no prior experience with PHPUnit. Any suggestions or improvements are welcome.

## Credits

I based this on the [Zend Framework manual](http://framework.zend.com/manual/en/zend.test.html) and the book Zend Framework in Action.
