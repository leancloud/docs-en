# LeanEngine Quick Start

In this documentation, you will learn how to create a LeanEngine project, debug locally, and deploy your project to the cloud.

## Creating a Project

### Creating a Project with Command-Line Interface (Recommended)

Before creating a project, make sure you have installed the command-line interface according to [Command-Line Interface](leanengine_cli.html#installation). To check if it is installed properly, run the following command:

```sh
lean help
```

You will see the help information if everything is good.

Now run the following command to create a project:

```sh
lean init
```

Follow the prompts to enter the project information.

### Creating a Project from a Template

You can use our sample project as a basement to start your project. Run the following command to clone the sample project from GitHub:

```js
git clone https://github.com/leancloud/node-js-getting-started.git
cd node-js-getting-started
```
```python
git clone https://github.com/leancloud/python-getting-started.git
cd python-getting-started
```
```php
git clone https://github.com/leancloud/slim-getting-started.git
cd slim-getting-started
```
```java
git clone https://github.com/leancloud/java-war-getting-started.git
cd java-war-getting-started
```

Now go to the directory of the sample project and run the following command to add your App ID to it:

```sh
lean switch
```

## Running the Project Locally

Run the following command to install dependencies for your project:

```js
npm install
```
```python
pip install -Ur requirements.txt
```
```php
composer install
```
```java
mvn package
```

Now you are good to run your project:

```sh
lean up
```

## Accessing the Site

Open your browser and go to http://localhost:3000. You will see the following content:

```js
LeanEngine

这是 LeanEngine 的示例应用

当前时间：Mon Feb 01 2016 18:23:36 GMT+0800 (CST)

一个简单的「TODO 列表」示例
```
```python
LeanEngine

这是 LeanEngine 的示例应用

一个简单的动态路由示例

一个简单的「TODO 列表」示例
```
```php
LeanEngine

这是 LeanEngine 的示例应用

当前时间：2016-07-25T14:55:17+08:00

一个简单的「TODO 列表」示例
```
```java
LeanEngine

这是 LeanEngine 的示例应用

一个简单的动态路由示例

一个简单的「TODO 列表」示例
```

The router of the site is defined as follows:

```js
// ./app.js
// ...

app.get('/', function(req, res) {
  res.render('index', { currentTime: new Date() });
});

// ...
```
```python
# ./app.py
# ...

@app.route('/')
def index():
    return render_template('index.html')

# ...
```
```php
// ./src/app.php
// ...

$app->get('/', function (Request $request, Response $response) {
    return $this->view->render($response, "index.phtml", array(
        "currentTime" => new \DateTime(),
    ));
});

// ...
```
```java
// ./src/main/webapp/WEB-INF/web.xml
// ...

  <welcome-file-list>
    <welcome-file>index.html</welcome-file>
  </welcome-file-list>

// ...
```

### Creating a Todo Item

Try adding a todo item at http://localhost:3000/todos and then go to your app's dashboard. You will see a new item in the class `Todo` with its `content` as the string you just entered. See LeanStorage guides for more instructions about how it could be implemented.

## Deploying Your Project to the Cloud

Run the following command to deploy your project to the production environment (if you haven't purchased a standard instance):

```sh
lean deploy
```

If you have a [second-level domain](leanengine_webhosting_guide-node.html#setting-up-second-level-domains) set up, you will be able to access your project via `http://${your_app_domain}.avosapps.us`.

If you have purchased a standard instance, your project will be deployed to the staging environment first when you run `lean deploy`. To deploy it to the production environment, run `lean publish`:

```sh
lean publish
```
