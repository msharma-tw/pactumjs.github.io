# Request Making

In general, the first step in API testing is to make a request to the server. There are different methods available in **PactumJS** that allows us to make a request.

- `pactum.spec()` - General API Testing
- `pactum.flow()` - Component & Contract API Testing
- `pactum.fuzz()` - Fuzz Testing
- `pactum.e2e()` - e2e Testing

`pactum.spec()` forms the base for all the above methods. So we will learn about `spec` method first and later about the rest of them.

```plantuml
@startuml

Tests -> "API Server": GET /api/users
Tests -> "API Server": POST /api/users { "name": "snow" } 
Tests -> "API Server": DELETE /api/users/1

@enduml
```

## spec

`pactum.spec()` will return an instance of *spec* which can be used to build the request and expectations.

```js
const pactum = require('pactum');

it('<test-name>', async () => {
  await pactum.spec()
    .get('http://httpbin.org/status/200');
});
```

To pass additional parameters to the request, we can chain or use the following methods individually to build our request.

| Method                    | Description                               |
| ------------------------- | ----------------------------------------- |
| `withPathParams`          | request path parameters                   |
| `withQueryParams`         | request query parameters                  |
| `withHeaders`             | request headers                           |
| `withBody`                | request body                              |
| `withJson`                | request json object                       |
| `withGraphQLQuery`        | graphQL query                             |
| `withGraphQLVariables`    | graphQL variables                         |
| `withFile`                | file path                                 |
| `withForm`                | object to send as form data               |
| `withMultiPartFormData`   | object to send as multi part form data    |
| `withRequestTimeout`      | sets request timeout                      |
| `withCore`                | http request options                      |
| `withAuth`                | basic auth details                        |
| `withFollowRedirects`     | sets follow redirect boolean property     |
| `inspect`                 | prints request & response details         |
| `__setLogLevel`           | sets log level for troubleshooting        |
| `toss` (optional)         | runs the spec & returns a promise         |

## Request Method

The request method indicates the method to be performed on the resource identified by the given Request-URI.

```js
const pactum = require('pactum');

it('GET /user', async () => {
  await pactum.spec()
    .get('http://domain.com/user');
});

it('POST /user', async () => {
  await pactum.spec()
    .post('http://domain.com/user');
});

it('PUT /user', async () => {
  await pactum.spec()
    .put('http://domain.com/user');
});

it('PATCH /user', async () => {
  await pactum.spec()
    .patch('http://domain.com/user');
});

it('DELETE /user', async () => {
  await pactum.spec()
    .delete('http://domain.com/user');
});

it('OPTIONS /user', async () => {
  await pactum.spec()
    .delete('http://domain.com/user');
});

it('TRACE /user', async () => {
  await pactum.spec()
    .delete('http://domain.com/user');
});

it('HEAD /user', async () => {
  await pactum.spec()
    .withMethod('HEAD')
    .withPath('http://domain.com/user');
});
```

In general, we set the base url to a constant value during API Testing. See [Request Settings](request-making?id=request-settings) to learn more about default configuration.

<!-- tabs:start -->

## ** base.test.js **

```js
const { request } = require('pactum');

// global hook
before(() => {
  request.setBaseUrl('http://localhost:3000');
});
```

## ** projects.test.js **

```js
const pactum = require('pactum');

it('get projects', async () => {
  // request will be sent to http://localhost:3000/api/projects
  await pactum.spec()
    .get('/api/projects');
});
```

<!-- tabs:end -->

## Path Params

Use `withPathParams` to pass path parameters to the request. We can either pass key-value pair or object as an argument. The given path params are replaced in the request path that are represented inside flower braces - `/some/api/{<key>}/path`.

```js
it('get a repository', async () => {
  await pactum.spec()
    .get('/api/project/{project}/repo/{repo}')
    .withPathParams('project', 'project-name')
    .withPathParams({
      'repo': 'repo-name'
    })
    .expectStatus(200);
});

//  The above would result in a url like - /api/project/project-name/repo/repo-name
```

## Query Params

Use `withQueryParams` to pass query parameters to the request. We can either pass key-value pair or object as an argument.

```js
it('get random male user from India with age 17', async () => {
  await pactum.spec()
    .get('https://randomuser.me/api')
    .withQueryParams('gender', 'male')
    .withQueryParams({
      'country': 'IND',
      'age': 17
    })
    .expectStatus(200);
});

//  The above would result in a url like - https://randomuser.me/api?gender=male&country=IND&age=17
```

## Headers

Use `withHeaders` to pass headers to the request. We can either pass key-value pair or object as an argument.

```js
it('get all comments', async () => {
  await pactum.spec()
    .get('https://jsonplaceholder.typicode.com/comments')
    .withHeaders('Authorization', 'Basic abc')
    .withHeaders({
      'Content-Type': 'application/json'
    })
    .expectStatus(200);
});
```

In general, we set default headers to all the requests that are sent during API Testing. For example, authorization headers.

<!-- tabs:start -->

#### ** base.test.js **

```js
const { request } = require('pactum');

// global hook
before(() => {
  request.setBaseUrl('http://localhost:3000');
  request.setDefaultHeaders('Authorization', 'Basic xxxxx');
});
```

#### ** projects.test.js **

```js
const pactum = require('pactum');

it('get projects', async () => {
  // request will be sent with authorization header.
  await pactum.spec()
    .get('/api/projects');
});
```

<!-- tabs:end -->

## Body

Use `withBody` or `withJson` *(preferred)* methods to pass the body to the request.

```js
it('post body', async () => {
  await pactum.spec()
    .post('https://jsonplaceholder.typicode.com/posts')
    .withBody('{ "title": "foo", "content": "bar"}')
    .expectStatus(201);
});
```

```js
it('post json object', async () => {
  await pactum.spec()
    .post('https://jsonplaceholder.typicode.com/posts')
    .withJson({
      title: 'foo',
      body: 'bar',
      userId: 1
    })
    .expectStatus(201);
});
```

## File Uploads

Use `withFile` method to upload a file. Under the hood, it uses [form-data](https://www.npmjs.com/package/form-data).

### Basic File Upload

```js
it('post a file', async () => {
  await pactum.spec()
    .post('https://httpbin.org/forms/posts')
    .withFile('./path/to/the/file')
    .expectStatus(201);
});

// => (new FormData()).append('file', file-buffer, { filename });
```

### File Upload with custom options

```js
it('post a file with custom options', async () => {
  await pactum.spec()
    .post('https://httpbin.org/forms/posts')
    .withFile('./path/to/the/file', { contentType: 'image/png' })
    .expectStatus(201);
});

// => (new FormData()).append('file', file-buffer, { filename, contentType });
```

### File Upload with custom key & options

```js
it('post a file with custom key & options', async () => {
  await pactum.spec()
    .post('https://httpbin.org/forms/posts')
    .withFile('file-image', './path/to/the/file', { contentType: 'image/png' })
    .expectStatus(201);
});

// => (new FormData()).append('file-image', file-buffer, { filename, contentType });
```

For more advanced usage, see [withMultiPartFormData](#withMultiPartFormData)

## Form Data

Use `withForm` or `withMultiPartFormData` to pass form data to the request.

### withForm

* Under the hood, pactum uses `phin.form`
* `content-type` header will be auto updated to `application/x-www-form-urlencoded`

```js 
it('post with form', async () => {
  await pactum.spec()
    .post('https://httpbin.org/forms/posts')
    .withForm({
      title: 'foo',
      body: 'bar',
      userId: 1
    })
    .expectStatus(201);
});
```

### withMultiPartFormData

* Under the hood it uses [form-data](https://www.npmjs.com/package/form-data)
* `content-type` header will be auto updated to `multipart/form-data`

```js
it('post with multipart form data', async () => {
  await pactum.spec()
    .post('https://httpbin.org/forms/posts')
    .withMultiPartFormData('file', fs.readFileSync('a.txt'), { contentType: 'application/js', filename: 'a.txt' })
    .expectStatus(201);
});
```

We can also directly use the form-data object.

```js
const form = new pactum.request.FormData();
form.append(/* form data */);

it('post with multipart form data', async () => {
  await pactum.spec()
    .post('https://httpbin.org/forms/posts')
    .withMultiPartFormData(form)
    .expectStatus(201);
});
```

## GraphQL

Use `withGraphQLQuery` or `withGraphQLVariables` to pass GraphQL data to the request. *Works for only POST requests.*

```js
it('post graphql query & variables', async () => {
  await pactum.spec()
    .post('https://jsonplaceholder.typicode.com/posts')
    .withGraphQLQuery(
      `
        query HeroNameAndFriends($episode: Episode) {
          hero(episode: $episode) {
            name
            friends {
              name
            }
          }
        }
      `
    )
    .withGraphQLVariables({
      "episode": "JEDI"
    })
    .expectStatus(201);
});
```

## Request Timeout

By default, pactum's request will timeout after 3000 ms. To increase the timeout for the current request use the `withRequestTimeout` method. **Make Sure To Increase The Test Runners Timeout As Well**


```js
it('some action that will take more time to complete', async () => {
  // increase mocha timeout here
  await pactum.spec()
    .post('https://jsonplaceholder.typicode.com/posts')
    .withJson({
      title: 'foo',
      body: 'bar',
      userId: 1
    })
    .withRequestTimeout(5000)
    .expectStatus(201);
});
```

# Request Settings

This library also offers us to set default options for all the requests that are sent through it.

## setBaseUrl

Sets the base URL for all the HTTP requests.

```js
const pactum = require('pactum');
const request = pactum.request;

request.setBaseUrl('http://localhost:3000');

it('get projects', async () => {
  // request will be sent to http://localhost:3000/api/projects
  await pactum.spec()
    .get('/api/projects');
});
```

## setDefaultTimeout

Sets the default timeout for all the HTTP requests.
The default value is **3000 ms**

```js
pactum.request.setDefaultTimeout(5000);
```

## setDefaultHeaders

Sets default headers for all the HTTP requests.

```js
pactum.request.setDefaultHeaders('Authorization', 'Basic xxxxx');
pactum.request.setDefaultHeaders({ 'content-type': 'application/json'});
```

## setDefaultFollowRedirects

Sets default follow redirect option for HTTP requests.

```js
pactum.request.setDefaultFollowRedirects(true);
```

# Spec Handlers

Handlers is a powerful concept in **pactum**. It helps us to reuse different features in this library like expectations, assertions, retry mechanisms, specs and many more.

Spec handlers helps us to reuse similar kind of request making & response validation across different test cases.

To define a common spec, use `handler.addSpecHandler` function.

The function accepts two arguments

- handler name - a string to refer the spec later in the test cases
- callback function - receives a context object containing spec & optional custom data properties.

<!-- tabs:start -->

## ** spec.handlers.js **

```js
const { addSpecHandler } = require('pactum').handler;

addSpecHandler('get user', (ctx) => {
  const { spec, data } = ctx;
  const { userId, status } = data;
  spec.get('/api/users');
  spec.withHeaders('Authorization', 'Basic abc');
  spec.withQueryParams('id', userId);
  spec.expectStatus(status);
});
```

## ** base.test.js **

```js
const { request } = require('pactum');

// load handlers
require('./spec.handlers');

// global hook
before(() => {
  request.setBaseUrl('http://localhost:3000');
});
```

## ** users.test.js **

```js
const pactum = require('pactum');

it('get valid user', async () => {
  await pactum.spec('get user', { userId: 10, status: 200 })
    .expectJson({ id: 10 });
});

it('get invalid user', async () => {
  // alternatively we can call spec handler using `use` method
  await pactum.spec()
    .use('get user', { userId: 9999, status: 400 })
    .expectJson({ error: 'user not found' });
});
```

<!-- tabs:end -->

- We are allowed to use request making methods & expectations while using spec handlers.

----

<a href="#/api-testing" >
  <img src="https://img.shields.io/badge/PREV-API%20Testing-orange" alt="API Testing" align="left" style="display: inline;" />
</a>
<a href="#/response-validation" >
  <img src="https://img.shields.io/badge/NEXT-Response%20Validation-blue" alt="Response Validation" align="right" style="display: inline;" />
</a>
