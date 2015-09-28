---
title: Developer Documentation for Cozy

language_tabs:
  - javascript

toc_footers:
  - <a href='http://github.com/tripit/slate'>Documentation Powered by Slate</a>

search: true
---


# Introduction

# Getting Started (Hello World)

## Set up development environment
## Hello World
## The Cozy architecture
## Access to other applications data
## Realtime events

# Authentication and Permissions

## Data access permissions
## Authentication for external applications

# Front-only application
# Full SPA Tutorial

# Standalone application
# Americano API
# CozyDB API
# Data System API
# Cookbooks

## Debug your application

### In development environment
### In production environment

## Localize your applications
## The Controller API
## Using nodemon

# Connectors


## Introduction

First, let's explain what a Konnector is. It's a piece of software to fetch your data from a vendor website or API. It's useful when you want to retrieve information like your sleep activity, your phone bills or your last tweets.

Fetched data are stored in your Cozy with the aim to be reused by other applications. Data are stored as records in the database. Data provided as PDF files (like bills) can be downloaded too. They will appear in your Files application like any file.

Adding a konnector doesn't require to understand the full Cozy app architecture. You only need to be familiar with the asynchronous programming of Node.js. 

All the logic will be contained in one file. To add your konnector, you have to add this file to the [Konnectors folder](https://github.com/cozy-labs/konnectors/tree/master/server/konnectors). 
Let's say you want to submit a Digital Ocean konnector, you will just need to add a `digital_ocean.coffee` file to that folder.

Your file should follow a specific structure to work properly. 
A konnector file is divided in 4 parts: 

* dependency declaration
* data description (models)
* konnector description 
* fetching logic. 

In the following we'll explain you what to put in this four parts.

## File Structure

### Dependencies

```coffeescript
cozydb = require 'cozydb'
moment = require 'moment'
requestApi = require 'request-json'
request = require 'request'
cheerio = require 'cheerio'
```

Like in any Node.js module you have to declare your external dependencies at the beginning. For that simply use the require function. 

Dependencies listed above are the most commonly used. Here is why these libs are interesting for your konnector:

* [cozydb](https://www.npmjs.com/package/americano-cozy) will allow you to declare your data models. It's mandatory.
* With [moment](http://momentjs.com/) you will manage date easily.
* [request](https://www.npmjs.com/package/request) is great to fetch webpages you will need to scrap.
* [request-json](https://www.npmjs.com/package/request-json) is a tool to query JSON-based API.
* [cheerio](https://www.npmjs.com/package/cheerio) will allow you to select elements from a downloaded html page like if you were using Jquery.

## Data description (models)


```coffeescript
# NB: this model is written here as example. If you want to declare bills,
# it's better if you use a pre-defined model.
# Ex: Bill = require '../models/bill'
Bill = cozydb.getModel 'Bill',    
    date: Date
    vendor: String
    type: {type: String, default: 'Internet'}
    amount: Number
    fileId: String
```

The second step is to describe the structure of the data you are going to store.

Here we declare a model through the `cozydb` module. You tell here which field can be found on any instance of this model. For every field, we give a type and sometimes additional information (like the default value).
Once you build an instance from data fetched on your target website/API, you have to create new model instances. Then you can save each model to store the data inside the Cozy. The best way for that is to use the `.create` at the class level.


```coffeescript
bill = 
    date: moment '01-02-2015'
    vendor: 'Free'
    amount: 20.0

# It's just here as an example: the saveDataAndFile layer will automatically do this for you
Bill.create bill, (err) ->
    if not err
        console.log 'Bill created'
```


## Konnector description

```coffeescript
module.exports =
    name: 'Free'
    slug: 'free'
    description: 'konnector description free'
    vendorLink: 'https://www.free.fr/'

    models:
        Bill: Bill

    fields:
        login: 'text'
        password: 'password'
        folderPath: 'folder'
```

The application needs a description for your Konnector. This description is composed of:

* **name**: the displayed name of the Konnector.
* **slug**: the slug of the konnector, its name without space and in lowercase.
* **description**: a text describing the konnector.
* **vendorLink**: the website of the vendor related to the konnector.
* **models**: List of models used by the konnector (we just link previously declared models to the konnector).
* **fields**: the list of field proposed to the user. That way the user can give required data to allow Konnector to connect to the target website/API on the user behalf.

Here is the list of available fields:

* *text*: simple text field.
* *password*: hidden text field.
* *folderPath*: a selection box to chose a folder from the file application (required to save downloaded files).


## Fetching logic

```coffeescript
fetch: (requiredFields, callback) ->
    login = requiredFields.login
    password = requiredFields.password
    folderPath = requiredFields.folderPath

    log.info "Import started"
    # Do stuff
    log.info "Import finished"

    callback err, "Here is the notification text."
```

The fetching is done through another field in your konnector. This field is called `fetch` and has a function as value. This function takes two parameters:

* *requiredFields*: a JS object containing the value given by the user via the fields from the form displayed to him.
* *callback*: the termination function to call when everything is finished.

Here you are free to operate in any way you want to scrap your data. In the following we'll give you an example of webpage scrapping and an example of API requesting. Finally we'll introduce you to the built-in data fetcher that will help you with helper to scrap or retrieve data.


## Scrap a webpage

```coffeescript
options =
    method: 'POST'
    jar: true
    url: "https://subscribe.free.fr/login/login.pl"
    form: 
        pass: requiredFields.password
        login: requiredFields.login

request options, (err, res, body) ->
     # Now we are logged in.
```

Through the next example, we'll show you the basics of web scrapping.

The first step is to login like if you were the user. For that you must go to the target login page and activate your browser network watcher (ctrl+shift+Q in Firefox). Then simply login into your target website. You will see in the network watcher query list a POST query that performs the login action. So to allow your konnector to fetch user information you will have to mimic this request. The analyze of the network watcher will give you the target url for login and the required fields required. 
Ex: For the Free website the login url is `https://subscribe.free.fr/login/login.pl` and the required fields are `pass` and `login`.

With that information, you can now use the `request` library to send a POST request to the login page. Don't forget to activate the cookie jar. It's an option that makes `request` store session information.


Once you are logged in you can fetch the page on the behalf of the user and parse it to grab his data. You will use `request` to fetch the page and `cheerio` to get the content of the part that interests you. Once again you will have to connect with your browser first. With the Inspector (ctrl+shift+C), you will see which part of the id and classes of the html tag of which the content should be extracted.

```coffeescript
billsToSave = []
url = "https://adsl.free.fr/liste-factures.pl"
request.get url, (err, res, body) ->

    $ = cheerio.load body
    $('.pane li.bill-line').each ->
         amount = $('.amount').html()
         amount = parseFloat(amount.replace ' Euros', '')

         month = $(this).find('.date')
         date = moment month.html(), 'YYYYMM'

         billsToSave.push new Bill
             date: date
             amount: amount
             vendor: 'Free'
```

Once you're done with the parsing, you can save the bill data to database.

## Request an API

```javascript
var client = requestJson.newClient('https://api.github.com');

var username = requiredFields.login;
var pass = requiredFields.password;
client.setBasicAuth(username, pass);

var path = "users/";
client.get(path, function(err, res, users) {
     console.log(users);
});
```

If the target vendor did his job well, it provides an API. This makes things simpler to fetch the user data. Now you need to refer to the vendor API documentation to see what you could do. Most of API are JSON-based. For this job we recommend the use of `request-json` which is better to request APIs than `request`.

Many APIs simply requires a basic authentification. Here is an example to connect on Github that will show you how to set auth and query a page with the Github API.


## Operation Layers

```coffeescript
fetcher.new()
    .use(logIn)
    .use(parsePage)
    .use(filterExisting log, PhoneBill)
    .use(saveDataAndFile log, PhoneBill, 'bouygues', ['facture'])
    .use(linkBankOperation
         log: log
         model: Bill
         identifier: 'bouyg'
         minDateDelta: 4
         maxDateDelta: 20
         amountDelta: 0.1
    )
    .args(requiredFields, {}, {})
    .fetch (err, fields, entries) ->
         console.log entries
```

We propose you an architecture for your konnectors. It's based on layers. Layers are operations that will lead to the fetching and storage of the data. They work the same way as [Express middlewares](http://expressjs.com/guide/using-middleware.html). All layers share the same parameter. They change that parameters to communicate each other. The execution order matters. Every layer will add new thing to the parameters and allow the next ones to perform their own operation based on the added information.

That way we can reuse operation layers between konnector and avoid to rewrite always the same thing. See in the following example. Our konnector will create a fetcher object that will run operation layers described before. Each layer is a simple function. Only the logIn and the parsePage layer are specific to the current Konnector. All others are already written and can be used in other konnectors.


```coffeescript
# login layer we log in and save the session cookie in the request library.
login = (requiredFields, entries, data, next) ->
    options =
         method: 'POST'
         jar: true
         url: "https://subscribe.free.fr/login/login.pl"
         form: 
             pass: requiredFields.password
             login: requiredFields.login

     request options, (err, res, body) ->
          next()
          
# Parse page layer: we parse the page and store the result in the bills field.
parsePage = (requiredFields, entries, data, next) ->
    url = "https://adsl.free.fr/liste-factures.pl"
    entries.fetched = []
    request.get url, (err, res, body) ->

        $ = cheerio.load body
        $('.pane li').each ->
            amount = $($(this).find('strong').get(1)).html()
            amount = parseFloat(amount.replace ' Euros', '')
            month = $(this).find('.date')
            date = moment month, 'YYYYMM'

            entries.fetched.push new Bill
                date: date
                amount: amount
                vendor: 'Free'
            next()
```

As you noticed the fetcher require initial arguments, they are always the same, just put `requiredFields, {}, {}` here. They are the arguments which will be passed to layers. Each layer will populate the two empty objects.


## Common layers

```coffeescript
filterExisting = require '../lib/filter_existing'
saveDataAndFile = require '../lib/save_data_and_file'
linkBankOperation = require '../lib/link_bank_operation'
```

Common layers can be required from the lib directory. They act a little bit differently than other layers. When you require them you import a function that will generate the layer that will be used. You 


#### filterExisting

```coffeescript
.use(filterExisting log, Bill)
```

*Action*

It requires that the `entries` object has a `fetched` attribute which is an array of fetched data to save. It will build a `filtered` field which will be an array of data that will contain only data that are not currently stored in the database. It checks for data existence through the data attribute.

*Parameters*

* `log`: a `printit` logger. See the [printit documentation](https://github.com/cozy/printit#it-began-with-a-consolelog) for details
* `model`: a model object to check for.


#### saveDataAndFile

```coffeescript
.use(saveDataAndFile log, Bill, 'github', ['bill'])
```

*Action*

This layer will persist data as a Cozy document for each entries listed in the `filtered` field of the entries object. If a `pdfurl` field is set on the entry object. The pdf is downloaded and a file is created for the File application.

*Parameters*

* `log`: a `printit` logger. See the [printit documentation](https://github.com/cozy/printit#it-began-with-a-consolelog) for details
* `model`: a model object used to save data.
* `suffix`: added to filename. This allows you to omit the "vendor" property on your bill object.
* `tags`: to apply to created files


#### linkBankOperation

```coffeescript
.use(linkBankOperation
    log: log
    model: Bill
    identifier: 'github'
    dateDelta: 4
    amountDelta: 5
)
```

*Action*

This layer has sense only for bill data.

It takes entries stored in the `fetched` field of the `entries` object. Then for each entry it will look for an operation that could match this entry. Once found, it attachs a binary to the bank operation. It's the same binary that is attached to the corresponding file object. 

The criterias to find a matching operation 
are:

* Operation label should contain the identifier given in parameter.
* The date should be between (bill date - dateDelta) and (bill date + dateDelta). Where dateDelta is given as a parameter and is in days.
* The amount should be between (bill amount - amountDelta) and (bill amount + amountDelta). Where amountDelta is given as a parameter.

*Parameters*

* `log`: a `printit` logger. See the [printit documentation](https://github.com/cozy/printit#it-began-with-a-consolelog) for details
* `model`: a model object to check for.
* `identifier`: a string to look for in the operation label (case insensitive: the layer will automatically set it to lowercase).
* `dateDelta`: the number of days allowed between the bank operation date and the bill date  (15 by default). 
* `amountDelta`: the difference between the bank operation amount and the bill amount (useful when the currency is not the same) (0 by default).



# Available Data

All Cozy apps produce many data that can be reused in your own application.The
main data available are the following one:


Doc Type | Main Fields
-------- | -----------
BankOperation | title, date, amount
Bill | type, vendor, date, amount
BloodPressure | systolic, diastolic
Bookmark | title, url, tag
Commit | sha, parent, tree, url, author, email, message, additions, deletions, files
Contact  | datapoints, n, fn, org, title, bday, department
Event    | start, end, description, place, rrule
File     | name, path, class, lastModification, creationDate, size, mime, checksum
Folder   | name, path, lastModification, creationDate
Email    | headers, from, to, cc, bcc, replyTo, subject, text, html, date, attachments, inReplyTo
Note     | title, content, creationDate, lastModificationDate
Photo    | title, description, orientation, binary
Sleep    | deepSleepDuration, lightSleepDuration, sleepDuration
Steps    | distance, steps
UseTracker | dateStart, dateEnd, duration, app
Weight | weight, leanWeight, fatWeight
Tasky | creationDate, completionDate, description, tags



# Architecture

## How works the development environment
## Components
## Authentication and Authorization workflows
## Encryption management


