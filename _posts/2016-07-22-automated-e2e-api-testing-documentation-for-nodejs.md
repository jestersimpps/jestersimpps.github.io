---
layout: post
title: "Automated E2E API Testing and Documentation for Node.js"
description: "A guide to setting up end-to-end API testing with Mocha and automated documentation generation with APIDoc for Node.js projects."
date: 2016-07-22
author: "Jester Simpps"
categories: [Development]
tags: [nodejs, testing, mocha, api, documentation]
image: /images/mocha.jpg
---

This was my first time doing test driven API development in Node.js and I must say, I really enjoyed it. I used to fall back on a Chrome plugin called [Postman](https://www.getpostman.com/) a lot before getting used to Mocha and test driven development. It was a joy to write code for a test that described what the endpoint should do before actually coding the new endpoint. It made me think more about the code I was going to write and what it should do. It also happened sometimes that there would be a total API overhaul (like the implementation of softdeletes). And, man was I glad I had written unit tests for all my endpoints when that happened!

## But...wait.. won't I lose a lot of time doing that?!?

![unit tests](https://risingstack-blog.s3-eu-west-1.amazonaws.com/2014/Oct/unit_testing.jpeg)

Writing API tests for a node.js API can be a tedious job. We are dealing with a single event loop, so we need to take in account callbacks. Especially when doing API testing, callbacks can quickly become a pain point. A lot of times, checking whether requests work means triggering other requests sequentially to see if the database was updated by previous requests. This is why, after refactoring a bit of my tests, I was able to create a simple end-to-end testscript that I would just call whenever I wrote a new route. Every time with different input data specific to the route of course. I was later able to tie in APIDoc, a documentation generator that will create a frontend containing the documentation for all your endpoints. So in the end all my endpoints were not only fully tested, but also automatically documented in one go. This was very nice and saved me a lot of time.

## Dependencies

### Testing

- We will be using [Mocha](https://mochajs.org/) as a testing framework for our Node.js API.
- Mocha is great, but not perfect, so on top of it, we will include the [supertest](https://github.com/visionmedia/supertest) package that will provide abstraction for HTTP testing.
- As I refactored my end to end tests into one single testscript, we will also need a package called [partial compare](https://www.npmjs.com/package/partial-compare) that will do partial matching between test objects and actual return-objects from the api.
- We will also use [should](https://www.npmjs.com/package/should) to make our testing code more human readable and easier to write.

### API documentation

- A great library called [APIdoc](http://apidocjs.com/) will take care of documenting all of the endpoints
- A plugin of that library [apidoc-plugin-schema](https://www.npmjs.com/package/apidoc-plugin-schema) will make our life easier.

In the package.json file:

```json
"devDependencies": {
  "apidoc-plugin-schema": "0.0.4",
  "body-parser": "~1.8.1",
  "fs": "0.0.2",
  "jsonfile": "^2.3.1",
  "mocha": "^2.1.0",
  "partial-compare": "^1.0.1",
  "path": "^0.12.7",
  "should": "~4.6.2",
  "supertest": "^0.15.0",
  "type-of-is": "^3.4.0"
}
```

## End to end testing

Wikipedia says:

> End-to-end testing is a technique used to test whether the flow of an application right from start to finish is behaving as expected. The purpose of performing end-to-end testing is to identify system dependencies and to ensure that the data integrity is maintained between various system components and systems.

Translate this to API testing and you get:

1. GET an array of existing data
2. POST some new data
3. Test if the new data was posted
4. Change the new data using PUT
5. Test if the new data was changed
6. DELETE the new data
7. Test if the new data was removed

Of course this is the most simple scenario of RESTful API testing, and I know there's some other endpoints that can be more complex (like counters, searches, etc.) But I will not elaborate on these in this post as these most of the time imply writing custom tests.

## A test script that covers multiple endpoints

Because the sequence of testing is the same for all endpoints, I created a general script that will test different endpoints using the same code but different input based on the endpoint's data. The input data will be stored in an object that will be called from an endpoint file in the 'test/routes' folder.

```bash
├── test
|    ├── routes
|         ├── endpoint1.js
|         ├── endpoint2.js
|         ├── endpoint3.js
|    ├── e2e_tests.js
|    ├── schema_generator.js
```

The content of the endpoint file will contain the e2e_test options object which has some options for the endpoint tests and apidoc generator.
Next to the options, a testData object contains all data needed to perform the tests. The object attributes are pretty self explanatory:

```javascript
const e2e = require('./e2e_tests.js');
const app = require('express');

'use strict';

(() => {

  let options = {
    timeout: 200,
    api: app,
    apidoc: false,
    schemaDir: __dirname + `/../data/apischemas/`
  }


  let testData = {
    routeName: 'Articles',
    postObject: {
      title: 'this is an article',
    },
    expectedObjectAfterPost: {
      title: 'this is an article',
    },
    putObject: {
      title: 'this is an article too',
    },
    expectedObjectAfterPut: {
      title: 'this is an article too',
    }
  };

  e2e(testData, options);


})();
```

When we run mocha the end-to-end script will be called and run all tests for that endpoint. If you have specified a schemaDir and the apidoc boolean is set to true, the script will also automatically generate json schemas for the apidoc based on the testData.

```bash
node_modules/.bin/mocha
```

Console output for a working API should be similar to:

```bash
-------- Albums tests --------

GET /Albums 200 148.080 ms - 2
Counting Albums: 0
    ✓ Should return an array of Albums
--POST shows/: {"general":{"name":"Ascot"},"time":{"startDate":"2016-04-06T21:51:15.000Z"},"type":"album"}
POST /Albums 200 151.935 ms - 1996
Adding: 1 Album
    ✓ Should create a new Album
GET /Albums/5789159e34150fe84c463f28 200 117.938 ms - 1996
    ✓ Should get the new Album
GET /Albums 200 103.512 ms - 1998
Counting Albums: 1, should be: 1
    ✓ Should return an array of Albums with the correct size
PUT /Albums/5789159e34150fe84c463f28 200 115.422 ms - 1997
    ✓ Should change a Album
GET /Albums/5789159e34150fe84c463f28 200 102.205 ms - 1997
    ✓ Should get the changed Album
DELETE /Albums/5789159e34150fe84c463f28 200 116.241 ms - 2018
Removing: 1 Album
    ✓ Should remove a Album
GET /Albums 200 104.573 ms - 2
Counting Albums: 0, should be: 0
    ✓ Should get an array without the deleted Album

-------- Articles tests --------

GET /Articles 200 92.230 ms - 2
Counting Articles: 0
    ✓ Should return an array of Articles
POST /Articles 200 98.047 ms - 300
Adding: 1 Article
    ✓ Should create a new Article
GET /Articles/578915a134150fe84c463f39 200 99.768 ms - 300
    ✓ Should get the new Article
GET /Articles 200 107.985 ms - 302
Counting Articles: 1, should be: 1
    ✓ Should return an array of Articles with the correct size
DELETE /Articles/578915a134150fe84c463f39 200 99.193 ms - 329
Removing: 1 Article
    ✓ Should remove a Article
GET /Articles 200 101.275 ms - 304
Counting Articles: 0, should be: 0
    ✓ Should get an array without the deleted Article


  2 passing (4s)

info: Done.
```

## API documentation generator

In your route files, above your endpoints, you can then add the following comment blocks:

- [api doc helpers](https://github.com/jestersimpps/node_rest_test_doc/blob/master/test/routes/APIDOC.md)

Change them according to your endpoints of course. They will be used by the APIdoc generator to create the documentation frontend.

```javascript
/**
* @apiSchema {jsonschema=../test/data/apischemas/<routecaps>-POST-create-a-new-<routesingle>-PARAMS.json} apiParam
* @apiSchema {jsonschema=../test/data/apischemas/<routecaps>-POST-create-a-new-<routesingle>-SUCCESS.json} apiSuccess
*/
```

The apiSchema references in the comments displayed above will point at the schema jsons that are generated by the [schema_generator](https://github.com/jestersimpps/node_rest_test_doc/blob/master/test/schema_generator.js).
The output directory can be changed in the options object in the endpoint test file as illustrated earlier.

Assuming the `./routes` folder contains your api's route files and the `./public` folder is the folder that will contain the apidoc frontend, generate your api documentation by running:

```bash
apidoc -i ./routes -o ./public/
```

- These tests were run using mongoDB, which uses `_id` as a primary key. You might have to edit the test script to your needs a bit when you are using a different database or key/slug to perform individual GET,PUT,DELETE requests.
- In the e2e_test script, basic authentication was used. You might also need to change this according to the authentication methods of your API.

The entire code and scripts can be found [here](https://github.com/jestersimpps/node_rest_test_doc)
