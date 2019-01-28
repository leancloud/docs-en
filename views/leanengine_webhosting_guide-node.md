{# Specify inherited template #}
{% extends "./leanengine_webhosting_guide.tmpl" %}
{% set productName = 'LeanEngine' %}
{% set platformName = 'Node.js' %}
{% set fullName = productName + ' ' + platformName %}
{% set sdk_name = 'JavaScript' %}
{% set leanengine_middleware = '[LeanEngine Node.js SDK](https://github.com/leancloud/leanengine-node-sdk)' %}

{% block getting_started %}

Clone the sample code node-js-getting-started to local:

```sh
git clone https://github.com/leancloud/node-js-getting-started.git
```

Execute the following command in the project root directory to install dependencies:


```sh
npm install
```

{% endblock %}

{% block custom_runtime %} {% endblock %}

{% block project_constraint %}


### Project skeleton
Take the example project as an example, in the project root directory we see a `package.json` file. Note: All Node.js projects must contain `package.json` in order to be correctly recognized by LeanEngine.

For historical reasons, please make sure that there is no file named `cloud/main.js` in your project.

#### package.json

There are [many options](https://docs.npmjs.com/files/package.json) that can be specified in `package.json`. It usually looks like this:


```json
{
    "name": "node-js-getting-started",
    "scripts": {
        "start": "node server.js"
    },
    "engines": {
        "node": "8.x"
    },
    "dependencies": {
        "express": "4.12.3",
        "leanengine": "^3.0.2",
        "leancloud-storage": "^3.3.1"
    }
}
```

Options that LeanEngine will respect include:
* `scripts.start` The command to start the project; the default is `node server.js`, which can be modified if you want to attach startup options to Node (such as `--es_staging`) or use other files as entry point.
* `scripts.prepublish` will run once after the project is built; you can write build commands such as `gulp build` here.
* `engines.node` specifies the required version of Node.js; For compatibility, the default version is still 0.12, so it is recommended that you specify a higher version. 8.x or above is recommended. You can also use `*`  to always use the latest version.
* `dependencies`  for the project dependencies; LeanEngine will use `npm install --production` during deployment to install all the dependencies listed here. If a dependency has peerDependencies, please make sure they are also listed in `dependencies` (not `devDependencies`).
* `devDependencies` The dependencies for development; currently, LeanEngine does **not** install modules in `devDependencies`.

We suggest that you create your own `package.json` using our [project template](https://github.com/leancloud/node-js-getting-started/blob/master/package.json) as reference.

We also support `package-lock.json` and `yarn.lock` :

- If your application directory contains `package-lock.json`, dependencies will be installed according to versions in the file (requires Node.js 8.0 or higher).
- If your application directory contains `yarn.lock`, then `yarn install` will be used instead of `npm install` to install dependencies (requires Node.js 4.8 and above).
{% endblock %}

{% block supported_frameworks %}


At this point, you know how to deploy a site to LeanEngine that can be accessed on the Internet. Next, we will delve into the details of implementing specific features.


## Integration with web frameworks

You might have noticed that we use the popular Node web framework [Express](http://expressjs.com/) in `package.json` in the sample project.

The Node SDK provides integration for [Express](http://expressjs.com/) and [Koa](http://koajs.com/). 

To use one of those frameworks, first add the LeanEngine Node SDK to the project:


```sh
npm install --save leanengine leancloud-storage
```

And configure it in your code:

#### Express

```js
var express = require('express');
var AV = require('leanengine');

AV.init({
  appId: process.env.LEANCLOUD_APP_ID || '{{appid}}',
  appKey: process.env.LEANCLOUD_APP_KEY || '{{appkey}}',
  masterKey: process.env.LEANCLOUD_APP_MASTER_KEY || '{{masterkey}}'
});

var app = express();
app.use(AV.express());
app.listen(process.env.LEANCLOUD_APP_PORT);
```
You can define custom HTTP APIs:

```js
app.get('/', function(req, res) {
  res.render('index', {title: 'Hello world'});
});

app.get('/time', function(req, res) {
  res.json({
    time: new Date()
  });
});

app.get('/todos', function(req, res) {
  new AV.Query('Todo').find().then(function(todos) {
    res.json(todos);
  }).catch(function(err) {
    res.status(500).json({
      error: err.message
    });
  });
});

```
For best practices, please refer to our [Project template](https://github.com/leancloud/node-js-getting-started) and [LeanEngine project examples](leanengine_examples.html).

#### Koa

```js
var koa = require('koa');
var AV = require('leanengine');

AV.init({
  appId: process.env.LEANCLOUD_APP_ID || '{{appid}}',
  appKey: process.env.LEANCLOUD_APP_KEY || '{{appkey}}',
  masterKey: process.env.LEANCLOUD_APP_MASTER_KEY || '{{masterkey}}'
});

var app = koa();
app.use(AV.koa2());  // use AV.koa() if you're using version 1.
app.listen(process.env.LEANCLOUD_APP_PORT);
```

You can use koa to provide a custom HTTP API:


```js
app.use(function *(next) {
  if (this.url === '/todos') {
    return new AV.Query('Todo').find().then(todos => {
      this.body = todos;
    });
  } else {
    yield next;
  }
});

```

When using Koa, it is recommended to set the version of Node.js to 4.x or higher according to the previous [package.json](#package_json) section. 


#### Other web frameworks

You can also use other web frameworks for development, but you need to implement the logic mentioned in [the health monitoring section](#Health monitoring) yourself. Here's a simple example of using the built-in [http](https://nodejs.org/api/http.html) module of:

```js
require('http').createServer(function(req, res) {
  if (req.url == '/') {
    res.statusCode = 200;
    res.end();
  } else {
    res.statusCode = 404;
    res.end();
  }
}).listen(process.env.LEANCLOUD_APP_PORT);
```

You need to listen on `0.0.0.0`(the default behavior of Node.js and of the `http` module) instead of `127.0.0.1`.

#### Route timeout

Because the asynchronous calls in Node can be interrupted by runtime errors or bugs, in order to reduce the memory usage of the server, and let the client receive errors earlier, you should set a timeout, so that when a request takes too long to process, the server returns an HTTP error.

The default timeout is 15 seconds, which can be adjusted:

```js
// Set default timeout
app.use(timeout('15s'));
```
{% endblock %}

{% block use_leanstorage %}

## Using LeanStorage

[LeanStorage](storage_overview.html) is a service provided by LeanCloud for storing structured data. You can persist data such as users' mailboxes, profiles, comments, and posts.


The Node SDK (the `leanengine` module) provides support for cloud functions and hooks required on the server-side. It also needs the JavaScript SDK (the `leancloud-storage` module) to be installed together as peer dependency. Please also upgrade the JavaScript SDK when upgrading the Node SDK:


```bash
npm install --save leanengine leancloud-storage
```

The [API documentation](https://github.com/leancloud/leanengine-node-sdk/blob/master/API.md) and [change log](https://github.com/leancloud/leanengine-node-sdk/releases) of the Node SDK are on GitHub.

```js
// Leanengine and leancloud-storage export the same object
var AV = require('leanengine');

AV.init({
  appId: process.env.LEANCLOUD_APP_ID || '{{appid}}',
  appKey: process.env.LEANCLOUD_APP_KEY || '{{appkey}}',
  masterKey: process.env.LEANCLOUD_APP_MASTER_KEY || '{{masterkey}}'
});


// You can use the useMasterKey to turn on the masterKey permission in LeanEngine, which will skip ACLs and other permission checks.

AV.Cloud.useMasterKey();

// Use the JavaScript SDK to query data in cloud storage.
new AV.Query('Todo').find().then(function(todos) {
  console.log(todos);
}).catch(function(err) {
  console.log(err)
});

```
{{ docs.note("If you need to turn off global masterKey permissions separately in some operations, please refer to [Cloud Functions · Permissions](leanengine_cloudfunction_guide-node.html#Master_Key_and super permission)。") }}

You can see the following code in `routes/todo.js` in the sample project:

```js
var router = require('express').Router();
var AV = require('leanengine');

// ···

// Add a new Todo project
router.post('/', function(req, res, next) {
  var content = req.body.content;
  var todo = new Todo();
  todo.set('content', content);
  todo.save().then(function(todo) {
    res.redirect('/todos');
  }).catch(next);
});
```
The snippet shows how to store a Todo object to LeanStorage. For more, please refer to:[Data Storage Development Guide · JavaScript](leanstorage_guide-js.html)

{% endblock %}

{% block get_env %}

```js
var NODE_ENV = process.env.NODE_ENV || 'development';
if (NODE_ENV === 'development') { 
  // The development environment, which is started by the command line tool.
} else if(NODE_ENV == 'production') {
  // The production environment.
} else { 
  // The staging environment.
}
```
{{ docs.alert("`NODE_ENV` is a reserved system variable. Developers should not attempt to override it.") }} 
{% endblock %}

{% block cookie_session %}

### Managing session on the server-end

If your page is primarily rendered by the server (for example, using ejs, pug), and the frontend does not need to use the JavaScript SDK for data manipulation, it is recommended that you use the `CookieSession` middleware to maintain session state in the cookie:

```js
app.use(AV.Cloud.CookieSession({ secret: 'my secret', maxAge: 3600000, fetchUser: true }));
```

Koa requires a  `framework: 'koa'` parameter:

```js
app.use(AV.Cloud.CookieSession({ framework: 'koa', secret: 'my secret', maxAge: 3600000, fetchUser: true }));
```
{{ docs.alert("while you are using a CookieSession, you need a [CSRF Token](leanengine_webhosting_guide-node.html#CSRF_Token) to protect against CSRF attacks.") }}

You need to pass in a secret for signing the cookie (mandatory). This middleware will record the login status of  `AV.User` in the cookie. If the user is logged in, you can obtain the current user via  `req.currentUser`.

The options supported by `AV.Cloud.CookieSession` include:

* **fetchUser** : Whether or not to automatically fetch the currently logged in AV.User object. The default is false. If it is set to true, every HTTP request will initiate an API call to fetch the user object. If it is set to false, by default you can only access the `id` of the req.currentUser (the ObjectId recorded by `_User` table) and the `sessionToken` attribute, and you can manually fetch the entire user when needed.
* **name**: The name of the cookie. The default is `avos.sess`.
* **maxAge**: Set the expiration time of the cookie. In milliseconds.

Node SDK no longer supports getting the user information via `AV.User.current()` after version 1.0 (see [upgrade to Leanengine Node.js SDK 1.0](leanengine-node-sdk-upgrade-1.html#abandon_currentUser)），instead, you need to use `request.currentUser`, and pass the user object explicitly in subsequent steps when needed.

You can easily implement a site with login like this:

```js
// Handling login requests (may come from forms in the login interface)
app.post('/login', function(req, res) {
  AV.User.logIn(req.body.username, req.body.password).then(function(user) {
    res.saveCurrentUser(user); // Save current user to the session
    res.redirect('/profile'); // Jump to the profile page
  }, function(error) {
    // Login failed, jump to login page
    res.redirect('/login');
  });
})

// View profile
app.get('/profile', function(req, res) {
  // Determine if the user has logged in
  if (req.currentUser) {
    // If the user has already logged in, send the current login user information.
    res.send(req.currentUser);
  } else {
    // If not, jump to the login page.
    res.redirect('/login');
  }
});

// Log out
app.get('/logout', function(req, res) {
  req.currentUser.logOut();
  res.clearCurrentUser(); // Remove users from the cookie session
  res.redirect('/profile');
});
```

#### Managing session on the browser side

If your page is mainly rendered by the browser (for example, using Vue, React, Angular) and uses JavaScript SDK for data operations on the frontend, it is recommended to use `AV.User.login` to login on the frontend.

When the backend needs to do some work depending the current status of the login user, the frontend obtains a token through `user.getSessionToken()`, and then sends it to the backend via HTTP Header.

For example on the frontend:

```javascript
AV.User.login(user, pass).then( user => {
  return fetch('/profile', headers: {
    'X-LC-Session': user.getSessionToken()
  });
});
```

And on the backend:

```javascript
app.get('/profile', function(req, res) {
  //  Query current user based on sessionToken
  AV.User.become(req.headers['x-lc-session']).then( user => {
    res.send(user);
  }).catch( err => {
    res.send({error: err.message});
  });
});

app.post('/todos', function(req, res) {
  var todo = new Todo();
  // Specify sessionToken when performing data operations
  todo.save(req.body, {sessionToken: req.headers['x-lc-session']}).then( () => {
    res.send(todo);
  }).catch( err => {
    res.send({error: err.message});
  });
});

```
{% endblock %}

{% block http_client %}

It is recommended to use [request](https://www.npmjs.com/package/request), a third-party module, to implement HTTP requests.

Install request:

```sh
npm install request --save
```
Code example:

```js
var request = require('request');

request({
  method: 'POST',
  url: 'http://www.example.com/create_post',
  json: {
    title: 'Post title',
    body: 'Post body'
  }
}, function(err, res, body) {
  if (err) {
    console.error('Request failed with response code ' + res.statusCode);
  } else {
    console.log(body);
  }
});
```
{% endblock %}

{% block code_get_client_ip_address %}
```js
app.get('/', function(req, res) {
  var ipAddress = req.headers['x-real-ip'] || req.headers['x-forwarded-for'] || req.connection.remoteAddress;
  console.log(ipAddress);
  res.send(ipAddress);
});
```
{% endblock %}

{% block upload_file_special_middleware %} 
Configure the app to use the [multiparty](https://www.npmjs.com/package/multiparty) module: 

```nodejs
var multiparty = require('multiparty');
```
{% endblock %}

{% block code_upload_file_sdk_function %}

```nodejs
var fs = require('fs');
app.post('/upload', function(req, res){
  var form = new multiparty.Form();
  form.parse(req, function(err, fields, files) {
    var iconFile = files.iconImage[0];
    if(iconFile.size !== 0){
      fs.readFile(iconFile.path, function(err, data){
        if(err) {
          return res.send('Failed to read file');
        }
        var theFile = new AV.File(iconFile.originalFilename, data);
        theFile.save().then(function(theFile){
          res.send('Uploaded successfully!');
        }).catch(console.error);
      });
    } else {
      res.send('Please select a file.');
    }
  });
});
```
{% endblock %}

{% block custom_session %} If you need to save properties in the session. You can add the generic  `cookie-session` component. For details, see [express.js &middot; cookie-session](https://github.com/expressjs/cookie-session). This component and the `AV.Cloud.CookieSession` component can coexist.

 <div class="callout callout-info">`express.session.MemoryStore ` in Express can not work properly in LeanEngine. Because LeanEngine runs in multiple processes and hosts, in-memory sessions cannot be shared. It is recommended to use [express.js &middot; cookie-session middleware](https://github.com/expressjs/cookie-session).</div>
{% endblock %}

{% block csrf_token %}In express, you can implement CSRF Token by using the library of [csurf](https://github.com/expressjs/csurf).
{% endblock %}


{% block leancache %} 
First add `redis` as a dependency:

```bash
npm install --save redis
```

Then you can create a Redis connection using the following code:

```js
var client = require('redis').createClient(process.env['REDIS_URL_<Instance Name>']);
// It is recommended to increase the client's on error event processing. Otherwise, the application process may be quit due to network fluctuations or redis server master-slave switching.
client.on('error', function(err) {
  return console.error('redis err: %s', err);
});
```
{% endblock %}

{% block https_redirect %} Express:

```js
app.enable('trust proxy');
app.use(AV.Cloud.HttpsRedirect());
```
Koa:

```js
app.proxy = true;
app.use(AV.Cloud.HttpsRedirect({framework: 'koa'}));
```
{% endblock %}

{% block extra_examples %}

### Multi-process operation

Node.js itself is single-threaded, so if you have 2-CPU (or above) LeanEngine instances, you need to configure multi-process using [cluster](https://nodejs.org/api/cluster.html).

Create `server-cluster.js`:

```js
var cluster = require('cluster');

var workers = process.env.LEANCLOUD_AVAILABLE_CPUS || 1;

if (cluster.isMaster) {
  for (var i = 0; i < workers; i++) {
    cluster.fork();
  }

  cluster.on('exit', (worker, code, signal) => {
    console.log('worker %s died, restarting...', worker.process.pid);
    cluster.fork();
  });
} else {
  require('./server.js')
}
```
Then in `package.json`, change `scripts.start` to`node server-cluster.js` :

```json
"scripts": {
  "start": "node server-cluster.js"
}
```

<div class="callout callout-info"> Multi-process requires that your program does not maintain global state (such as locks) in memory. It is recommended to perform sufficient tests when switching to multi-process or multi-instance for the first time.</div>

{% endblock %}

{% block depentencyCache %} For example, if the node project is deployed twice in a row and `package.json` is not modified, then the cached dependencies are used directly. 
{% endblock %}


{% block code_calling_custom_variables %}

```nodejs
// Use custom environment variables in LeanEngine Node.js environment
var MY_CUSTOM_VARIABLE = process.env.MY_CUSTOM_VARIABLE;
console.log(MY_CUSTOM_VARIABLE);
```

{% endblock %}

{% block loggerExample %}

```nodejs
console.log('hello');
console.error('some error!');
```
{% endblock %}

{% block loggerExtraDescription %} 
You can print requests sent by the LeanCloud SDK by setting the environment variable `DEBUG=leancloud:request`. For local development you can use:

```sh
env DEBUG=leancloud:request lean up
```

When there is a request to LeanCloud, you will see a log like this:

```
leancloud:request request(0) +0ms GET https://{{host}}/1.1/classes/Todo?&where=%7B%7D&order=-createdAt { where: '{}', order: '-createdAt' }
leancloud:request response(0) +220ms 200 {"results":[{"content":"1","createdAt":"2016-08-09T06:18:13.028Z","updatedAt":"2016-08-09T06:18:13.028Z","objectId":"57a975a55bbb5000643fb690"}]}
```

We do not recommend enabling it in the production environment, otherwise a huge number of logs may be printed.

{% endblock %}

{% block section_timezone %} 
You need to pay attention to the different methods of the Date type in JavaScript. Some will return UTC time and some will return local time:

Function | Timezone | result
---------|----------|--------
`toISOString` | UTC time | 2015-04-09T03:35:09.678Z
`toJSON`（JSON Serialization）| UTC time | 2015-04-09T03:35:09.678Z
`toUTCString` | UTC time | Thu, 09 Apr 2015 03:35:09 GMT
`getHours` | UTC time | 3
`toString` (used by `console.log()`) | local time | Thu Apr 09 2015 03:35:09 GMT+0000 (UTC)
`toLocaleString` | local time | Thu Apr 09 2015 03:35:09 GMT+0000 (UTC)

When constructing a `Date` object, you should pass an object with time zone (for example, 2011-10-10T14:48:00.000Z instead of 2011-10-10T14:48:00) Otherwise, `Date` [will not know how to interpret it](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/parse).

{% endblock %}



