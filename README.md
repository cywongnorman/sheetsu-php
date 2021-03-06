# sheetsu-php

## Installation
The Sheetsu PHP Library is available through [Composer](https://getcomposer.org/).

You can edit your composer.json file or just hit this command in your terminal
```
composer require emilianozublena/sheetsu-php
```

## Usage

### Instantianting the Sheetsu Client

You need to instantiate the main Sheetsu object and give the SheetID
You can find this URL on [Sheetsu Dashboard](https://sheetsu.com/your-apis).
Remember to use composer's autoload.

```php
require('vendor/autoload.php');
use Sheetsu\Sheetsu;

$sheetsu = new Sheetsu([
    'sheetId' => 'sheetId'
]);
```

If you have HTTP Basic Authentication turned on for your API, you should pass `key` and `secret` here, like:
```php
require('vendor/autoload.php');
use Sheetsu\Sheetsu;

$sheetsu = new Sheetsu([
    'sheetId'   => 'sheetId',
    'key'       => 'key',
    'secret'    => 'secret'
]);
```

### Initialize library

If you need, you can reinitialize (or initialize) the library after creation

```php
# Initialize after creation
$sheetsu = new Sheetsu();
$sheetsu->initialize([
    'sheetId'   => 'sheetId',
    'key'       => 'key',
    'secret'    => 'secret'
])
```

### Collection-Model

The Sheetsu PHP Library comes with a small implementation of a [Collection abstract data type](https://en.wikipedia.org/wiki/Collection_(abstract_data_type)).

Models are units of Collections (in this case, each Model represents a Row of the given sheet).

Instead of giving arrays to the Sheetsu Client (for CRUD operations), you can do something like this:

```php
$collection = new Collection();
$collection->addMultiple([
    Model::create(['name' => 'John']),
    Model::create(['name' => 'Steve'])
]);
$response = $sheetsu->create($collection);
```
Collections and Models are the 2 objects that you are going to get every time you call the api too.

### Create
[Link to docs](https://sheetsu.com/docs#post)

To add data to Google Spreadsheets, send an array, an array of arrays, or more simply, work with Models or Collections ;)

```php
# Adds single row from array
$sheetsu->create(['name' => 'John']);
# Adds multiple rows from array
$sheetsu->create([
    ['name' => 'John'],
    ['name' => 'Steve']
]);
# Adds single row from Model
$sheetsu->create(Model::create(['name' => 'John']));
# Adds multiple rows from Collection
$collection = new Collection();
$collection->addMultiple([
    Model::create(['name' => 'John']),
    Model::create(['name' => 'Steve'])
]);
$response = $sheetsu->create($collection);
```

After call is made, returns a Response object.

### Read
[Link to docs](https://sheetsu.com/docs#get)

Read the whole sheet
```php
$response = $sheetsu->read();
$collection = $response->getCollection();
```

Read function allows 2 parameters
  - `limit` - limit number of results
  - `offset` - start from N first record

```php
# Get first two rows from sheet starting from the first row
$response = $sheetsu->read(2, 0);
$collection = $response->getCollection();
```

#### search
[Link to docs](https://sheetsu.com/docs#get_search)

To get rows that match search criteria, pass an array with criteria

```php
# Get all rows where column 'id' is 'foo' and column 'value' is 'bar'
$response = $sheetsu->search([
    'id'    => 'foo',
    'value' => 'bar'
]);
$collection = $response->getCollection();

# Get all rows where column 'First name' is 'Peter' and column 'Score' is '42'
$response = $sheetsu->search([
    'First name'    => 'Peter',
    'Score'         => '42'
]);
$collection = $response->getCollection();

# Get first two row where column 'First name' is 'Peter',
# column 'Score' is '42' from sheet named "Sheet3"
$response = $sheetsu->search([
    'First name'    => 'Peter',
    'Score'         => '42'
], 2, 0);
$collection = $response->getCollection();

# Get first two row where column 'First name' is 'Peter',
# column 'Score' is '42' from sheet named "Sheet3"
# with ignore case
$response = $sheetsu->search([
    'First name'    => 'Peter',
    'Score'         => '42'
], 2, 0, true);
$collection = $response->getCollection();
```

### Update
[Link to docs](https://sheetsu.com/docs#patch)

To update row(s), pass column name and its value which is used to find row(s) and an array or model with the data to update.

```php
# Update all columns where 'name' is 'Peter' to have 'score' = 99 and 'last name' = 'Griffin'
$model = Model::create(['score' => '99', 'last name' => 'Griffin']);
$response = $sheetsu->update('name', 'Peter', $model);
```

By default, [PATCH request](https://sheetsu.com/docs#patch) is sent, which is updating only values which are in the collection passed to the method. To send [PUT request](https://sheetsu.com/docs#put), pass 4th argument being `true`. [Read more about the difference between PUT and PATCH in sheetsu docs](https://sheetsu.com/docs#patch).

### Delete
[Link to docs](https://sheetsu.com/docs#delete)

To delete row(s), pass column name and its value which is used to find row(s).

```php
# Delete all rows where 'name' equals 'Peter'
$response = $sheetsu->delete('name', 'Peter');
```

### Change Active Sheet

If you need to change the sheet you're working on, you can do it by using the sheet function and passing the new sheetsu id

```php
# Change active sheetsu to 'THIS'
$sheetsu->sheet('THIS')
```

You can also chain this function with others, like so:
```php
# Change active sheetsu to 'THIS'
$sheetsu->sheet('THIS')->read();
```

### Go back to using the whole spreadsheet again

If you need to use the whole spreadsheet is easy:

```php
# Back to whole spreadsheet
$sheetsu->whole()
```

You can also chain this function with others, like so:
```php
# Back to whole spreadsheet
$sheetsu->whole()->read();
```


### Response, Connection & Error Handling
The Sheetsu PHP Library handles the connection through the Connection class. This class uses cURL for making connections and uses the Response class as returns.
The Response object is the one responsible for giving the collections, models or errors (or any other response from the last call)
Error handling is also made through the Response object (Response uses the ErrorHandler class to abstract the try/catch block and is tightly coupled to the ErrorException php class)
```php
$response = $sheetsu->read();
#if you need only the error messages, you can get the errors like this
$errors = $response->getErrors();
$firstError = $response->getError();
#if you need to get the exceptions thrown, do it like this.
$exceptions = $response->getExceptions();
$firstException = $response->getException();
```

### Unit Testing with PHPUnit
This library has above 97% of code coverage. Some of the test are not using mock objects, this is in our to-do list and hope we'll be doing it so.
The tests are prepared to be used with PHPUnit and the test suite is configured via XML, so you'll only need to execute PHPUnit in your forked version of this repo like so:
```shell
./vendor/bin/phpunit
```

## TODO
- [x] Define and implement ErrorHandler to leverage the final user from handling http status code's
- [x] Make this repository work as package with Composer
- [x] Create Unit Test with at least 80% coverage (currently in 91%)
- [x] Add ignore_case to search
- [x] Add a way to manage the active sheet used
- [ ] Create mock objects for tests
- [ ] Make class agnostic in such a way that Collection & Model classes are replaceable for others (or maybe for a ORM like Eloquent)