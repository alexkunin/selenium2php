Selenium2php
==========================

###Description
Converts HTML text of Selenium test case recorded from Selenium IDE into
PHP code for PHPUnit_Extensions_SeleniumTestCase or PHPUnit_Extensions_Selenium2TestCase as TestCase file.
Basic workflow:

1. Record HTML tests with [Selenum IDE](http://docs.seleniumhq.org/projects/ide/)
2. Store them in some directory
3. Use selenium2php [switches] <directory>
4. Use [PhantomJS](http://phantomjs.org/) or other [webdrivers](http://docs.seleniumhq.org/projects/webdriver/) for testing
5. Edit test if necessary and go to step 2

It is easier if you use continuous integration, for example, [Jenkins](http://jenkins-ci.org/).

###Usage
    selenium2php [switches] Test.html [Test.php]
    selenium2php [switches] <directory>
    
    --dest=<path>                  Destination folder.
    --selenium2                    Use Selenium2 tests format.
    --php-prefix=<string>          Add prefix to php filenames.
    --php-postfix=<string>         Add postfix to php filenames.
    --browser=<browsers string>    Set browser for tests.
    --browser-url=<url>            Set URL for tests.
    --remote-host=<host>           Set Selenium server address for tests.
    --remote-port=<port>           Set Selenium server port for tests.
    -r|--recursive                 Use subdirectories for converting.
    --class-prefix=<prefix>        Set TestCase class prefix.
    --use-hash-postfix             Add hash part to output filename
    --files-pattern=<pattern>      Glob pattern for input test files (*.html).
    --output-tpl=<file>            Template for result file. See TestExampleTpl.

###Selenium2 features
If you are going to use Selenium 2, you should know:
* You do not wait for simple page loading (links, form submits).
* You have to wait for some elements if you use ajax calls or changing visiblity via javascripts. 
Use waitFor*, for example, waitForElementPresent after click command.
* You can nott manipulate invisible elements.

###Selenium2 (PhantomJS, GhostDriver) Example
You have google.html recorded in Selenium IDE:

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
    <html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" lang="en">
    <head profile="http://selenium-ide.openqa.org/profiles/test-case">
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <link rel="selenium.base" href="https://www.google.com/" />
    <title>google</title>
    </head>
    <body>
    <table cellpadding="1" cellspacing="1" border="1">
    <thead>
    <tr><td rowspan="1" colspan="3">google</td></tr>
    </thead><tbody>
    <tr>
            <td>open</td>
            <td>/</td>
            <td></td>
    </tr>
    <tr>
            <td>waitForElementPresent</td>
            <td>css=form input[autocomplete="off"]</td>
            <td></td>
    </tr>
    <tr>
            <td>type</td>
            <td>css=form input[autocomplete="off"]</td>
            <td>github</td>
    </tr>
    <tr>
            <td>click</td>
            <td>css=form input[type="submit"]</td>
            <td></td>
    </tr>
    <tr>
            <td>waitForElementPresent</td>
            <td>id=res</td>
            <td></td>
    </tr>
    <tr>
            <td>assertTextPresent</td>
            <td>Build software better, together</td>
            <td></td>
    </tr>
    </tbody></table>
    </body>
    </html>

You run

    php selenium2php.php --selenium2 --browser=phantomjs --output-tpl=Selenium2TestCaseTpl.tpl google.html

And you get GoogleTest.php

    <?php
    /*
    * Autogenerated from Selenium html test case by Selenium2php.
    * 2013-10-17 09:43:33
    */
    class S2p_GoogleTest extends PHPUnit_Extensions_Selenium2TestCase{

        function setUp(){
            $this->setBrowser("phantomjs");
            $this->setBrowserUrl("https://www.google.com/");
            $this->setHost("127.0.0.1");
            $this->setPort(4444);
        }

        function testGoogle(){
            $this->url('/');

            $this->waitUntil(function($testCase) {
                try {
                    $testCase->byCssSelector("form input[autocomplete=\"off\"]");
                    return true;
                } catch (PHPUnit_Extensions_Selenium2TestCase_WebDriverException $e) {}
            }, 8000);

            $input = $this->byCssSelector("form input[autocomplete=\"off\"]");
            $input->value("github");

            $input = $this->byCssSelector("form input[type=\"submit\"]");
            $input->click();

            $this->waitUntil(function($testCase) {
                try {
                    $testCase->byId("res");
                    return true;
                } catch (PHPUnit_Extensions_Selenium2TestCase_WebDriverException $e) {}
            }, 8000);

            $this->assertTrue((bool)(strpos($this->byTag('body')->text(), 'Build software better, together') !== false));

        }

    }

Your test case is ready.
Prepare your selenim hub + phantomjs webdriver like [here](https://github.com/detro/ghostdriver#register-ghostdriver-with-a-selenium-grid-hub):
    java -jar selenium.jar -role hub
    phantomjs --webdriver=1408 --webdriver-selenium-grid-hub=http://127.0.0.1:4444

And run phpunit:
    php phpunit.phar GoogleTest.php

###Selenium1 (RC) Example
You have google.html recorded in Selenium IDE:

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
    <html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" lang="en">
    <head profile="http://selenium-ide.openqa.org/profiles/test-case">
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <link rel="selenium.base" href="https://www.google.ru/" />
    <title>google</title>
    </head>
    <body>
    <table cellpadding="1" cellspacing="1" border="1">
    <thead>
    <tr><td rowspan="1" colspan="3">google</td></tr>
    </thead><tbody>
    <tr>
            <td>open</td>
            <td>/</td>
            <td></td>
    </tr>
    <tr>
            <td>type</td>
            <td>id=gbqfq</td>
            <td>github</td>
    </tr>
    <tr>
            <td>waitForElementPresent</td>
            <td>id=res</td>
            <td></td>
    </tr>
    <tr>
            <td>assertTextPresent</td>
            <td>Build software better, together</td>
            <td></td>
    </tr>
    </tbody></table>
    </body>
    </html>

You run

    php selenium2php.php google.html

And you get GoogleTest.php

    <?php
    /*
    * Autogenerated from Selenium html test case by Selenium2php.
    * 2013-02-28 12:26:53
    */
    class GoogleTest extends PHPUnit_Extensions_SeleniumTestCase{
        function setUp(){
            $this->setBrowser("*firefox");
            $this->setBrowserUrl("https://www.google.ru/");
        }
        function testGoogle(){
            $this->open("/");
            $this->type("id=gbqfq", "github");
            for ($second = 0; ; $second++) {
                if ($second >= 60) $this->fail("timeout");
                try {
                    if ($this->isElementPresent("id=res")) break;
                } catch (Exception $e) {}
                sleep(1);
            }
            $this->assertTrue($this->isTextPresent("Build software better, together"));
        }
    }    


###PHPUnit built-in method for Selenium1 (RC)
If you don't need php test files, see [PHPUnit Example](http://phpunit.de/manual/3.8/en/selenium.html#selenium.seleniumtestcase.examples.WebTest4.php)

    <?php
    require_once 'PHPUnit/Extensions/SeleniumTestCase.php';

    class SeleneseTests extends PHPUnit_Extensions_SeleniumTestCase
    {
        public static $seleneseDirectory = '/path/to/files';
    }
    ?>

###License
    Copyright 2013 Roman Nix
    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at
    
    http://www.apache.org/licenses/LICENSE-2.0
    
    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.

