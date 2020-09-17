[![Build Status](https://travis-ci.org/bl82/opencart-test-suite.svg?branch=master)](https://travis-ci.org/bl82/opencart-test-suite)

# OpenCart Testing Suite

## Supported OpenCart Versions

| OpenCart version | opencart-test-suite version |build status|
|---|---|---|
| ~3.0   | ~3.0   | [![Build Status](https://travis-ci.org/bl82/opencart-test-suite.svg?branch=3.0)](https://travis-ci.org/bl82/opencart-test-suite) |
| ~2.2.0 | ~2.2.0 | [![Build Status](https://travis-ci.org/bl82/opencart-test-suite.svg?branch=2.2)](https://travis-ci.org/bl82/opencart-test-suite) |
| ~2.3.0 | ~2.3.0 | [![Build Status](https://travis-ci.org/bl82/opencart-test-suite.svg?branch=2.3)](https://travis-ci.org/bl82/opencart-test-suite) |

## Motivation
The intend of this project is to provide a simple approach for setting up a test suite for custom OpenCart development. 

## Getting started from scratch

 - Create a new OpenCart instance (maybe follow this [guide](https://medium.com/@stefan.huber/installing-opencart-with-composer-74fe0ba121b1))
 - Add `opencart-test-suite` as a dependency `composer require bl82/opencart-test-suite --dev`
 - Use `composer require bl82/opencart-test-suite:3.0 --dev` for OpenCart version 3.0 respectively
 - Create a `tests` folder and add respective tests (see examples below)
 - Add a `phpunit.xml` which includes testsuites (e.g. admin and catalog) and set an env variable to the opencart root directory (see example phpunit.xml below)
 - Now tests can be run via `vendor/bin/phpunit --testsuite catalog-tests` command

__Our [OpenCart project template](https://github.com/bl82/opencart-project-template) might simplify setup for you.__

## Example of a phpunit.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<phpunit bootstrap="vendor/autoload.php">
    <testsuites>
        <testsuite name="catalog-tests">
            <directory suffix="Test.php">./tests/catalog/</directory>
        </testsuite>
        <testsuite name="admin-tests">
            <directory suffix="AdminTest.php">./tests/admin/</directory>
        </testsuite>
    </testsuites>
    <php>
        <env name="OC_ROOT" value="/../opencart/root-folder" />
        <env name="HTTP_SERVER" value="http://localhost:8080/" />
        <env name="TEST_CONFIG" value="test-config" />
    </php>
</phpunit>
```

## Test Examples

### Testing a Model

```php
namespace Tests;

class ModelCatalogManufacturerTest extends OpenCartTest
{
    public function testASpecificManufacturer()
    {
        // load the manufacturer model
        $model = $this->loadModel("catalog/manufacturer");
        $manufacturer = $model->getManufacturer(5);

        // test a specific assertion
        $this->assertEquals('HTC', $manufacturer['name']);

    }
}
```

### Testing a Controller
```php
namespace Tests;

class ControllerCheckoutCartTest extends OpenCartTest
{
    public function testAddingASpecificProductToTheCart()
    {
        $response = $this->dispatchAction('checkout/cart/add','POST',['product_id' => 28]);
        $output = json_decode($response->getOutput(),true);
        
        $this->assertTrue(isset($output['success']) && isset($output['total']));
        $this->assertRegExp('/HTC Touch HD/', $output['success']);
    }
}
```

### Testing with logged in Customers
```php
class ControllerAccountEditTest extends OpenCartTest {  
    public function testEditAccountWithLoggedInCustomer() {

        $this->login('somebody@test.com','password');
        
        $response = $this->dispatchAction('account/edit');
        $this->assertRegExp('/Your Personal Details/',$response->getOutput());
        
        $this->logout();
        
    }   
}
```

### Testing with logged in Users inside Admin

In order to test classes inside the admin folder just call your test class ending with `AdminTest` e.g. `ModelCatalogCategoryAdminTest`

```php
class ControllerCommonDashboardAdminTest extends OpenCartTest {  
    public function testShowDashboardWithLoggedInUser() {

        $this->login('admin','admin');
        
        $response = $this->dispatchAction('common/dashboard');
        $this->assertRegExp('/Total Sales/', $response->getOutput());
        
        $this->logout();
        
    }   
}
```
