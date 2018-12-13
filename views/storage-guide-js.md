{# 指定继承模板 #}
{% extends "./storage-guide.tmpl" %}

{# --Start--变量定义，主模板使用的单词和短语在所有子模板都必须赋值 #}
{% set cloudName ="LeanCloud" %}
{% set productName ="LeanStorage" %}
{% set platform_name ="JavaScript" %}
{% set segment_code ="js" %}
{% set sdk_name ="JavaScript SDK" %}
{% set baseObjectName ="AV.Object" %}
{% set objectIdName ="id" %}
{% set updatedAtName ="updatedAt" %}
{% set createdAtName ="createdAt" %}
{% set backgroundFunctionTemplate ="xxxxInBackground" %}
{% set saveEventuallyName ="saveEventually" %}
{% set deleteEventuallyName ="deleteEventually" %}
{% set relationObjectName ="AV.Relation" %}
{% set pointerObjectName ="AV.Pointer" %}
{% set baseQueryClassName ="AV.Query" %}
{% set geoPointObjectName ="AV.GeoPoint" %}
{% set userObjectName ="AV.User" %}
{% set fileObjectName ="AV.File" %}
{% set dateType= "Date" %}
{% set byteType= "Buffer" %}
{% set acl_guide_url = "[ACL JavaScript Guide](acl_guide-js.html)" %}
{% set sms_guide_url = "[SMS JavaScript Guide](sms-guide.html#注册验证)" %}
{% set inapp_search_guide_url = "[In-App Searching JavaScript Guide](app_search_guide.html)" %}
{% set status_system_guide_url = "[In-App Socializing JavaScript Guide](status_system.html#JavaScript_SDK)" %}
{% set feedback_guide_url = "（JavaScript SDK 文档待补充）" %}
{% set funtionName_whereKeyHasPrefix = "startsWith" %}
{% set saveOptions_query= "query" %}
{% set saveOptions_fetchWhenSave= "fetchWhenSave" %}

{% block text_web_security %}
## Web Security

If you are using our JavaScript SDK in your browser based application, you must configure **whitelist domains** in your app's [LeanCloud Dashboard settings page](/dashboard/app.html?appid={{appid}}#/security) before you deploy it to a production environment. Once set up, our backend will only accept requests sent from domains in the whitelist, which prevents unauthorized usage or access to your cloud data on a certain level.

However, since your app is exposed to the internet, simply setting up whitelist domains can't totally guarantee the security of your app. LeanCloud offers a variety of ways to help you secure your apps as well as the data stored within them. The following articles give you the necessary information you need to know regarding app security.

- [Data and Security](data-security.html)
- [ACL Guide](acl-guide.html)
{% endblock %}

{# --End--变量定义，主模板使用的单词和短语在所有子模板都必须赋值 #}

{# --Start--主模板留空的代码段落，子模板根据自身实际功能给予实现 #}

{% block code_quick_save_a_todo %}
```js
// Declare Class to be Todo
var Todo = AV.Object.extend('Todo');
// Create a new Todo object
var todo = new Todo();
todo.set('title', 'R&D Weekly Meeting');
todo.set('content', 'All team members, Tue 2pm');
todo.save().then(function (todo) {
  // Run other logic after the object is saved
  console.log('New object created with objectId: ' + todo.id);
}, function (error) {
  // Error handling
  console.error('Failed to create new object, with error message: ' + error.message);
});
```
{% endblock %}

{% block code_quick_save_a_todo_with_location %}
```js
var Todo = AV.Object.extend('Todo');
var todo = new Todo();
todo.set('title', 'R&D Weekly Meeting');
todo.set('content', 'All team members, Tue 2pm');
// Simply add the following line of code and the server will automatically add the field
todo.set('location', 'Meeting Room');
todo.save().then(function (todo) {
  // Run other logic after the object is saved
}, function (error) {
  // Error handling
});
```
{% endblock %}

{% block code_create_todo_object %}
```js
// The className passed into AV.Object.extend('className') indicates the name of table
// Declare Class
var Todo = AV.Object.extend('Todo');
```

**Attention**: If you keep seeing `Maximum call stack size exceeded` exception in your log, chances are `AV.Object.extend` had been called using the same ClassName more than once within a loop or a callback. You can resolve the problem with one of the following ways:

- Switch to JavaScript SDK 1.4 or higher
- Declare the Class out of the loop or the callback to make sure that `AV.Object.extend` won't be over-instantiated

From version 1.4.0 onwards, the JavaScript SDK is compatible with ES6 classes. If you're using ES6 in your codebase, you can subclass `AV.Object` with the **extends** keyword. However, when using extends, the SDK is not automatically aware of your subclasses. You will need to register the subclasses with the SDK:

```js
class Todo extends AV.Object { }
// Register the subclass with SDK
AV.Object.register(Todo);
```
{% endblock %}

{% block code_save_object_by_cql %}
```js
// Add a new TodoFolder object with CQL
AV.Query.doCloudQuery('insert into TodoFolder(name, priority) values("Work", 1)').then(function (data) {
  // data.results contains a list of AV.Objects as the result of query
  var results = data.results;
}, function (error) {
  // Error handling
  console.error(error);
});
```
{% endblock %}

{% block code_data_type %}
```js
// Declare only once
var TestObject = AV.Object.extend('DataTypeTest');

var number = 2014;
var string = 'famous film name is ' + number;
var date = new Date();
var array = [string, number];
var object = { number: number, string: string };

var testObject = new TestObject();
testObject.set('testNumber', number);
testObject.set('testString', string);
testObject.set('testDate', date);
testObject.set('testArray', array);
testObject.set('testObject', object);
testObject.set('testNull', null);
testObject.save().then(function (testObject) {
  // Success
}, function (error) {
  // Failure
});
```

We **do not recommend** storing large pieces of binary data (like images or documents) with `AV.Object`. **The size of each `AV.Object` should be within 128 kilobytes.** We recommend using [`AV.Files`](#files) for storing images, documents, and other types of files.
{% endblock %}

{% block section_dataType_largeData %}{% endblock %}

{% block code_save_todo_folder %}
```js
// Declare Class
var TodoFolder = AV.Object.extend('TodoFolder');
// Create object
var todoFolder = new TodoFolder();
// Set name
todoFolder.set('name', 'Work');
// Set priority
todoFolder.set('priority', 1);
todoFolder.save().then(function (todo) {
  console.log('objectId is ' + todo.id);
}, function (error) {
  console.error(error);
});
```
{% endblock %}

{% block code_saveoption_query_example %}
```js
var Account = AV.Object.extend('Account');
new AV.Query(Account).first().then(function (account) {
  var amount = -100;
  account.increment('balance', amount);
  return account.save(null, {
    query: new AV.Query(Account).greaterThanOrEqualTo('balance', -amount),
    fetchWhenSave: true,
  });
}).then(function (account) {
  // Successfully updated
  console.log('Current balance: ', account.get('balance'));
}).catch(function (error) {
  if (error.code === 305) {
    console.log('Insufficient balance. Operation failed!');
  }
});
```
{% endblock %}

{% macro code_get_todo_by_objectId() %}
```js
var query = new AV.Query('Todo');
query.get('57328ca079bc44005c2472d0').then(function (todo) {
  // Successfully retrieved the instance
  // todo is the instance of Todo with id 57328ca079bc44005c2472d0
}, function (error) {
  // Error handling
});
```
{% endmacro %}

{% block code_fetch_todo_by_objectId %}
```js
// className as the first parameter; objectId as the second parameter
var todo = AV.Object.createWithoutData('Todo', '5745557f71cfe40068c6abe0');
todo.fetch().then(function () {
  var title = todo.get('title'); // Get title
  var content = todo.get('content'); // Get content
}, function (error) {
  // Error handling
});
```
{% endblock %}

{% block code_save_callback_get_objectId %}
```js
var todo = new Todo();
todo.set('title', 'R&D Weekly Meeting');
todo.set('content', 'All team members, Tue 2pm');
todo.save().then(function (todo) {
  // Run other logic after the object is saved
  // Get objectId
  var objectId = todo.id;
}, function (error) {
  // Error handling
});
```
{% endblock %}

{% block code_access_todo_folder_properties %}
```js
var query = new AV.Query('Todo');
query.get('558e20cbe4b060308e3eb36c').then(function (todo) {
  // Successfully retrieved the instance
  // todo is the instance of Todo with id 558e20cbe4b060308e3eb36c
  var priority = todo.get('priority');
  var location = todo.get('location');
  var title = todo.get('title');
  var content = todo.get('content');

  // Fetch the three special properties
  var objectId = todo.id;
  var updatedAt = todo.updatedAt;
  var createdAt = todo.createdAt;

  // Wed May 11 2016 09:36:32 GMT+0800 (CST)
  console.log(createdAt);
}, function (error) {
  // Error handling
  console.error(error);
});
```

If you need to fetch all the properties of an object at one time (which usually happens when performing data binding) without calling `get(propertyName)`, you may call `toJSON()` on an AV.Object instance (leancloud-storage@^3.0.0 required) to get a plain JSON object containing all the fields of the object:

```js
var query = new AV.Query('Todo');
query.get('558e20cbe4b060308e3eb36c').then(function (todo) {
  console.log(todo.toJSON())
  // ==== Results in console ====

  // content: "All team members, Tues 2pm"
  // createdAt: "2017-03-08T11:25:07.804Z"
  // location: "Meeting Room"
  // objectId: "558e20cbe4b060308e3eb36c"
  // priority: 1
  // title: "R&D Weekly Meeting"
  // updatedAt: "2017-03-08T11:25:07.804Z"

}).catch(function (error) {
  // Error handling
  console.error(error);
});
```
{% endblock %}

{% block code_object_fetch %}
```js
// Construct an AV.Object with the known objectId
var todo = new Todo();
todo.id = '5590cdfde4b00f7adb5860c8';
todo.fetch().then(function (todo) {
  // todo is the instance of Todo fetched from the cloud
  var priority = todo.get('priority');
}, function (error) {

});
```
{% endblock %}

{% block code_object_fetchWhenSave %}
```js
// set fetchWhenSave to be true
todo.fetchWhenSave(true);
todo.save().then(function () {
  // Successfully saved
}, function (error) {
  // Error handling
  console.error(error);
});
```
{% endblock %}

{% block code_object_fetch_with_keys %}
```js
// Construct an AV.Object with the known objectId
var todo = new Todo();
todo.id = '5590cdfde4b00f7adb5860c8';
todo.fetch({
  keys: 'priority,location'
}).then(function (todo) {
  // Fetch from the cloud
}, function (error) {
  // Error handling
  console.error(error);
});
```
{% endblock %}

{% block code_update_todo_location %}
```js
// Create AVObject with known objectId
// className as the first parameter; objectId as the second parameter
var todo = AV.Object.createWithoutData('Todo', '558e20cbe4b060308e3eb36c');
// Set properties
todo.set('location', 'Meeting Room, 2nd Floor');
// Save to the cloud
todo.save().then(function () {
  // Successfully saved
}, function (error) {
  // Error handling
  console.error(error);
});
```
{% endblock %}

{% block code_update_todo_content_with_objectId %}
```js
// className as the first parameter; objectId as the second parameter
var todo = AV.Object.createWithoutData('Todo', '5745557f71cfe40068c6abe0');
// Set properties
todo.set('content', 'R&D Weekly Meeting, rescheduled to Wed 3:30pm for this week.');
// Save to the cloud
todo.save();
```
{% endblock %}

{% block code_update_object_by_cql %}
```js
// Update a TodoFolder object with CQL
AV.Query.doCloudQuery(
  'update TodoFolder set name="Family" where objectId="558e20cbe4b060308e3eb36c"'
).then(function (data) {
  // data.results contains a list of AV.Objects as the result of query
  var results = data.results;
}, function (error) {
  // Error handling
  console.error(error);
});
```
{% endblock %}

{% block code_atomic_operation_increment %}
```js
var todo = AV.Object.createWithoutData('Todo', '57328ca079bc44005c2472d0');
todo.set('views', 0);
todo.save().then(function (todo) {
  todo.increment('views', 1);
  todo.fetchWhenSave(true);
  return todo.save();
}).then(function (todo) {
  // The latest value will be fetched when using fetchWhenSave
}, function (error) {
  // Error handling
});
```
{% endblock %}

{% block code_atomic_operation_array %}
* `AV.Object.add('arrayKey', value)`<br>
  appends the given object to the end of an array.
* `AV.Object.addUnique('arrayKey', value);`<br>
  adds the given object into an array only if it is not in it. The position inserted is not guaranteed.
* `AV.Object.remove('arrayKey', value);`<br>
  removes all instances of the given object from an array.
{% endblock %}

{% block code_set_array_value %}
```js
var reminder1 = new Date('2015-11-11 07:10:00');
var reminder2 = new Date('2015-11-11 07:20:00');
var reminder3 = new Date('2015-11-11 07:30:00');

var reminders = [reminder1, reminder2, reminder3];

var todo = new AV.Object('Todo');
// Specify that reminders is the array containing Dates
todo.addUnique('reminders', reminders);
todo.save().then(function (todo) {
  console.log(todo.id);
}, function (error) {
  // Error handling
  console.error(error);
});
```
{% endblock %}

{% block code_delete_todo_by_objectId %}
```js
var todo = AV.Object.createWithoutData('Todo', '57328ca079bc44005c2472d0');
todo.destroy().then(function (success) {
  // Success
}, function (error) {
  // Failure
});
```
{% endblock %}

{% block code_delete_todo_by_cql %}
```js
// Delete a Todo object with CQL
AV.Query.doCloudQuery('delete from Todo where objectId="558e20cbe4b060308e3eb36c"').then(function () {
  // Successfully deleted the object
}, function (error) {
  // Error handling
});
```
{% endblock %}

{% block code_batch_operation %}
```js
var objects = []; // Create an array for AV.Objects

// Batch create (save)
AV.Object.saveAll(objects).then(function (objects) {
  // Success
}, function (error) {
  // Error handling
});
// Batch delete
AV.Object.destroyAll(objects).then(function () {
  // Success
}, function (error) {
  // Error handling
});
// Batch fetch
AV.Object.fetchAll(objects).then(function (objects) {
  // Success
}, function (error) {
  // Error handling
});
```
{% endblock %}

{% block code_batch_set_todo_completed %}
```js
var query = new AV.Query('Todo');
query.find().then(function (todos) {
  todos.forEach(function (todo) {
    todo.set('status', 1);
  });
  return AV.Object.saveAll(todos);
}).then(function (todos) {
  // Success
}, function (error) {
  // Error handling
});
```
{% endblock %}

{% block text_work_in_background %}{% endblock %}
{% block save_eventually %}{% endblock %}

{% block code_relation_todoFolder_one_to_many_todo %}
```js
var todoFolder = new AV.Object('TodoFolder');
todoFolder.set('name', 'Work');
todoFolder.set('priority', 1);

var todo1 = new AV.Object('Todo');
todo1.set('title', 'R&D Weekly Meeting');
todo1.set('content', 'All team members, Tue 2pm');
todo1.set('location', 'Meeting Room');

var todo2 = new AV.Object('Todo');
todo2.set('title', 'Maintain Documentation');
todo2.set('content', 'Everyday, 4pm to 6pm');
todo2.set('location', 'Current Position');

var todo3 = new AV.Object('Todo');
todo3.set('title', 'Release SDK');
todo3.set('content', 'Every Monday afternoon, 3pm');
todo3.set('location', 'SA Position');

var todos = [todo1, todo2, todo3];
AV.Object.saveAll(todos).then(function () {
  var relation = todoFolder.relation('containedTodos'); // Create AV.Relation
  todos.map(relation.add.bind(relation));
  return todoFolder.save(); // Save to the cloud
}).then(function (todoFolder) {
  // Successfully saved
}, function (error) {
  // Error handling
});
```
{% endblock %}

{% block code_pointer_comment_one_to_many_todoFolder %}
```js
var comment = new AV.Object('Comment'); // Create Comment object
comment.set('likes', 1); // 1 is like; -1 is dislike; 0 by default
comment.set('content', 'Bro these are awesome! I want the same games as well! Shall we buy together?');
// Assume we already know the objectId of the shared TodoFolder is 5735aae7c4c9710060fbe8b0
var targetTodoFolder = AV.Object.createWithoutData('TodoFolder', '5735aae7c4c9710060fbe8b0');
comment.set('targetTodoFolder', targetTodoFolder);
comment.save(); // Save to the cloud
```
{% endblock %}

{% block code_create_geoPoint %}
```js
// First parameter is latitude
// Second parameter is longitude
var point = new AV.GeoPoint(39.9, 116.4);

// Another way to create AV.GeoPoint
var point2 = new AV.GeoPoint([12.7, 72.2]);
var point3 = new AV.GeoPoint({ latitude: 30, longitude: 30 });
```
{% endblock %}

{% block code_use_geoPoint %}
```js
todo.set('whereCreated', point);
```
{% endblock %}

{% block text_deserialize_and_serialize %}
<!-- js 以及 ts 没有序列化和反序列化的需求 -->
{% endblock %}

{% block code_data_protocol_save_date %}
```js
var testDate = new Date('2016-06-04');
var testAVObject = new AV.Object('TestClass');
testAVObject.set('testDate', testDate);
testAVObject.save();
```
{% endblock %}

{% block code_create_avfile_by_stream_data %}
```js
var data = { base64: '6K+077yM5L2g5Li65LuA5LmI6KaB56C06Kej5oiR77yf' };
var file = new AV.File('resume.txt', data);
file.save();

var bytes = [0xBE, 0xEF, 0xCA, 0xFE];
var byteArrayFile = new AV.File('myfile.txt', bytes);
byteArrayFile.save();
```
{% endblock %}

{% block code_create_avfile_from_local_path %}
Suppose there is an input on the webpage for selecting file:

```html
<input type="file" id="photoFileUpload">
```
Here is the code for uploading the file:
```js
var fileUploadControl = $('#photoFileUpload')[0];
if (fileUploadControl.files.length > 0) {
  var localFile = fileUploadControl.files[0];
  var name = 'avatar.jpg';

  var file = new AV.File(name, localFile);
  file.save().then(function (file) {
    // Successfully saved the file
    console.log(file.url());
  }, function (error) {
    // Error handling
    console.error(error);
  });
}
```
{% endblock %}

{% block code_create_avfile_from_url %}
```js
var file = AV.File.withURL('Satomi_Ishihara.gif', 'http://ww3.sinaimg.cn/bmiddle/596b0666gw1ed70eavm5tg20bq06m7wi.gif');
file.save().then(function (file) {
  // Successfully saved the file
  console.log(file.url());
}, function (error) {
  // Error handling
  console.error(error);
});
```
{% endblock %}

{% block text_upload_file %}
If you need to upload files from LeanEngine, please refer to [LeanEngine Web Hosting Guide](leanengine_webhosting_guide-node.html#uploading-files).
{% endblock %}

{% block code_upload_file_with_progress %}
```js
file.save({
  onprogress: function (e) {
    console.log(e)
    // { loaded: 1234, total: 2468, percent: 50 }
  },
}).then(/* ... */);

// The second parameter of save (callbacks) must be present if SDK version is below 2.0
file.save({
  onprogress: function (e) { console.log(e); }
}, {}).then(/* ... */);
```
{% endblock %}

{% block text_download_file_with_progress %}{% endblock %}

{% block text_file_query %}
### Queries on Files
How files are queried depends on the model used for storing files in the database. For example, when storing profile pictures of users, you may create a new field in `_User` table named `avatar` and store a url pointing to the address of the picture file. However, we highly recommend that you associate {{userObjectName}} and {{fileObjectName}} with Pointer:

{% block code_user_pointer_file %}
```js
var data = { base64: 'File in base64' };
var avatar = new AV.File('avatar.png', data);

var user = new AV.User();
var randomUsername = 'Tom';
user.setUsername(randomUsername)
user.setPassword('leancloud');
user.set('avatar', avatar);
user.signUp().then(function (u) {
});
```
{% endblock %}
{% endblock %}

{% block code_file_image_thumbnail %}
```js
// Get a 100px by 200px (width by height) thumbnail
var url = file.thumbnailURL(100, 200);
```
{% endblock %}

{% block code_file_metadata %}
```js
// Get the size of the file
var size = file.size();
// Get the objectId of uploader (AV.User) (will be null if not logged in)
var ownerId = file.ownerId();

// Get all metadata of the file
var metadata = file.metaData();
// Set the author of the file
file.metaData('author', 'LeanCloud');
// Get the format of the file
var format = file.metaData('format');
```
{% endblock %}

{% block code_file_delete %}
```js
var file = AV.File.createWithoutData('552e0a27e4b0643b709e891e');
file.destroy().then(function (success) {
}, function (error) {
});
```
{% endblock %}

{% block code_cache_operations_file %}{% endblock %}

{% block text_https_access_for_ios9 %}{% endblock %}

{% block code_create_query_by_className %}
```js
var query = new AV.Query('Todo');
```
{% endblock %}

{% block code_priority_equalTo_zero_query %}
```js
var query = new AV.Query('Todo');
// Find all Todos with priority to be 0
query.equalTo('priority', 0);
query.find().then(function (results) {
  var priorityEqualsZeroTodos = results;
}, function (error) {
});
```
{% endblock %}

{% block code_priority_equalTo_zero_and_one_wrong_example %}
```js
var query = new AV.Query('Todo');
query.equalTo('priority', 0);
query.equalTo('priority', 1);
query.find().then(function (results) {
  // The first condition will be overwritten. The query will only return Todo items with priority to be 1.
}, function (error) {
});
```
{% endblock %}

{% block table_logic_comparison_in_query %}
Logical Operation | AVQuery Method
---|---
Equal to | `equalTo`
Not Equal to | `notEqualTo`
Greater than | `greaterThan`
Greater than or Equal to | `greaterThanOrEqualTo`
Less than | `lessThan`
Less than  or Equal to | `lessThanOrEqualTo`
{% endblock %}

{% block code_query_lessThan %}
```js
var query = new AV.Query('Todo');
query.lessThan('priority', 2);
```
{% endblock %}

{% block code_query_greaterThanOrEqualTo %}
```js
query.greaterThanOrEqualTo('priority', 2);
```
{% endblock %}

{% block code_query_with_regular_expression %}
```js
var query = new AV.Query('Todo');
var regExp = new RegExp('[\u4e00-\u9fa5]', 'i');
query.matches('title', regExp);
query.find().then(function (results) {
}, function (error) {
});
```
{% endblock %}

{% block code_query_with_contains_keyword %}
```js
query.contains('title', 'Tony');
```
{% endblock %}

{% block code_query_with_not_contains_keyword_using_regex %}
<pre><code class="lang-js">  var query = new AV.Query('Todo');
  var regExp = new RegExp('{{ data.regex(true) | safe }}, 'i');
  query.matches('title', regExp);
</code></pre>
{% endblock %}
<!-- 2016-12-29 故意忽略最后一行中字符串的结尾引号，以避免渲染错误。不要使用 markdown 语法来替代 <pre><code> -->

{% block code_query_array_contains_using_equalsTo %}
```js
var query = new AV.Query('Todo');
var reminderFilter = [new Date('2015-11-11 08:30:00')];
query.containsAll('reminders', reminderFilter);

// You can also use equalTo
var targetDateTime = new Date('2015-11-11 08:30:00');
query.equalTo('reminders', targetDateTime);
```
{% endblock %}

{% block code_query_array_contains_all %}
```js
var query = new AV.Query('Todo');
var reminderFilter = [new Date('2015-11-11 08:30:00'), new Date('2015-11-11 09:30:00')];
query.containsAll('reminders', reminderFilter);
```
{% endblock %}

{% block code_query_with_not_contains_keyword %}
```js
query.notContainedIn('reminders', reminderFilter);
```
{% endblock %}

{% block code_query_whereHasPrefix %}
```js
// Find all Todos starting with "breakfast"
var query = new AV.Query('Todo');
query.startsWith('content', 'breakfast');

// Find all Todos containing "bug"
var query = new AV.Query('Todo');
query.contains('content', 'bug');
```
{% endblock %}

{% block code_query_comment_by_todoFolder %}
```js
var query = new AV.Query('Comment');
var todoFolder = AV.Object.createWithoutData('TodoFolder', '5735aae7c4c9710060fbe8b0');
query.equalTo('targetTodoFolder', todoFolder);

// Specify `key` with `include` if properties of relational objects need to be fetched
query.include('targetTodoFolder');
```
{% endblock %}

{% block code_create_tag_object %}
```js
var tag = new AV.Object('Tag');
tag.set('name', 'Todah');
tag.save();
```
{% endblock %}

{% block code_create_family_with_tag %}
```js
var tag1 = new AV.Object('Tag');
tag1.set('name', 'Today');

var tag2 = new AV.Object('Tag');
tag2.set('name', 'Urgent');

var tag3 = new AV.Object('Tag');
tag3.set('name', 'Important');

var tags = [tag1, tag2, tag3];
AV.Object.saveAll(tags).then(function (savedTags) {
  var todoFolder = new AV.Object('TodoFolder');
  todoFolder.set('name', 'Family');
  todoFolder.set('priority', 1);

  var relation = todoFolder.relation('tags');
  relation.add(tag1);
  relation.add(tag2);
  relation.add(tag3);

  todoFolder.save();
}, function (error) {
});
```
{% endblock %}

{% block code_query_tag_for_todoFolder %}
```js
var todoFolder = AV.Object.createWithoutData('TodoFolder', '5735aae7c4c9710060fbe8b0');
var relation = todoFolder.relation('tags');
var query = relation.query();
query.find().then(function (results) {
  // results is an array of AV.Objects containing all the tags of the current todoFolder
}, function (error) {
});
```
{% endblock %}

{% block code_query_todoFolder_with_tag %}
```js
var targetTag = AV.Object.createWithoutData('Tag', '5655729900b0bf3785ca8192');
var query = new AV.Query('TodoFolder');
query.equalTo('tags', targetTag);
query.find().then(function (results) {
  // results is an array of AV.Objects
  // results contains all the TodoFolders with the current tag
}, function (error) {
});
```
{% endblock %}

{% block code_query_comment_include_todoFolder %}
```js
var commentQuery = new AV.Query('Comment');
commentQuery.descending('createdAt');
commentQuery.limit(10);
commentQuery.include('targetTodoFolder'); // include tells the server to return details of the objects being found out, not just objectId
commentQuery.include('targetTodoFolder.targetAVUser'); // Return details of targetAVUser objects, not just objectId
commentQuery.find().then(function (comments) {
  // comments contains the most recent 10 comments; its targetTodoFolder field contains the relative data
  for (var i = 0; i < comments.length; i++) {
    var comment = comments[i];
    // No internet access needed
    var todoFolder = comment.get('targetTodoFolder');
    var avUser = todoFolder.get('targetAVUser');
  }
}, function (error) {
});
```
{% endblock %}

{% block code_query_comment_match_query_todoFolder %}
```js
// Construct a inner query
var innerQuery = new AV.Query('TodoFolder');
innerQuery.greaterThan('likes', 20);

// Assign the inner query to the target query
var query = new AV.Query('Comment');

// Execute the query
query.matchesQuery('targetTodoFolder', innerQuery);
query.find().then(function (results) {
  // results is an array containing all Comments of all TodoFolders with more than 20 likes
}, function (error) {
});

query.doesNotMatchQuery('targetTodoFolder', innerQuery);
// All Comments of all TodoFolders with 20 likes or less
```
{% endblock %}

{% block code_query_find_first_object %}
```js
var query = new AV.Query('Comment');
query.equalTo('priority', 0);
query.first().then(function (data) {
  // data is the first AV.Object matching the conditions
}, function (error) {
});
```
{% endblock %}

{% block code_set_query_limit %}
```js
var query = new AV.Query('Todo');
var now = new Date();
query.lessThanOrEqualTo('createdAt', now); // Returns all Todos created before today
query.limit(10); // Return at most 10 results
```
{% endblock %}

{% block code_set_skip_for_pager %}
```js
var query = new AV.Query('Todo');
var now = new Date();
query.lessThanOrEqualTo('createdAt', now); // Find all Todos created before today
query.limit(10); // Return at most 10 results
query.skip(20); // Skip 20 results
```
{% endblock %}

{% block code_query_select_keys %}
```js
var query = new AV.Query('Todo');
query.select(['title', 'content']);
query.first().then(function (todo) {
  console.log(todo.get('title')); // √
  console.log(todo.get('content')); // √
  console.log(todo.get('location')); // undefined
}, function (error) {
  // Error handling
});
```
{% endblock %}

{% block code_query_select_pointer_keys %}
```js
query.select('owner.username');
```
{% endblock %}

{% block code_query_count %}
```js
var query = new AV.Query('Todo');
query.equalTo('status', 1);
query.count().then(function (count) {
  console.log(count);
}, function (error) {
});
```
{% endblock %}

{% block code_query_orderby %}
```js
// Ascending by time
query.ascending('createdAt');

// Descending by time
query.descending('createdAt');
```
{% endblock %}

{% block code_query_orderby_on_multiple_keys %}
```js
var query = new AV.Query('Todo');
query.addAscending('priority');
query.addDescending('createdAt');
```
{% endblock %}

{% block code_query_with_or %}
```js
var priorityQuery = new AV.Query('Todo');
priorityQuery.greaterThanOrEqualTo('priority', 3);

var statusQuery = new AV.Query('Todo');
statusQuery.equalTo('status', 1);

var query = AV.Query.or(priorityQuery, statusQuery);
// Return all Todos with priority greater than or equal to 3, or status equal to 1
```
{% endblock %}

{% block code_query_with_and %}
```js
var startDateQuery = new AV.Query('Todo');
startDateQuery.greaterThanOrEqualTo('createdAt', new Date('2016-11-13 00:00:00'));

var endDateQuery = new AV.Query('Todo');
endDateQuery.lessThan('createdAt', new Date('2016-12-03 00:00:00'));

var query = AV.Query.and(startDateQuery, endDateQuery);
```
{% endblock %}

{% block code_query_where_keys_exist %}
```js
var aTodoAttachmentImage = AV.File.withURL('attachment.jpg', 'http://www.zgjm.org/uploads/allimg/150812/1_150812103912_1.jpg');
var todo = new AV.Object('Todo');
todo.set('images', aTodoAttachmentImage);
todo.set('content', 'Remember to buy tickets to home!');
todo.save();

var query = new AV.Query('Todo');
query.exists('images');
query.find().then(function (results) {
  // results is an array containing all Todos with pictures
}, function (error) {
});

// Find all Todos without pictures
query.doesNotExist('images');
```
{% endblock %}

{% block code_query_by_cql %}
```js
var cql = 'select * from Todo where status = 1';
AV.Query.doCloudQuery(cql).then(function (data) {
  // results is an array of AV.Objects containing results
  var results = data.results;
}, function (error) {
});
cql = 'select count(*) from %@ where status = 0';
AV.Query.doCloudQuery(cql).then(function (data) {
  // The number of results
  var count = data.count;
}, function (error) {
});
```
{% endblock %}

{% block code_query_by_cql_with_placeholder %}
```js
// CQL with placeholder
var cql = 'select * from Todo where status = ? and priority = ?';
var pvalues = [0, 1];
AV.Query.doCloudQuery(cql, pvalues).then(function (data) {
  // results is an array of AV.Objects containing results
  var results = data.results;
}, function (error) {
});
```
{% endblock %}

{% block text_query_cache_intro %}{% endblock %}

{% block code_set_cache_policy %}{% endblock %}

{% block table_cache_policy %}{% endblock %}

{% block code_query_geoPoint_near %}
```js
var query = new AV.Query('Todo');
var point = new AV.GeoPoint(39.9, 116.4);
query.withinKilometers('whereCreated', point, 2.0);
query.find().then(function (results) {
  var nearbyTodos = results;
}, function (error) {
});
```

In the code above, `nearbyTodos` returns an array of objects sorted according to the distance between each object and `point` (from closest to furthest). Keep in mind that **if `ascending` or `descending` is called afterwards, the objects will be sorted accordingly.**
{% endblock %}

{% block text_platform_geoPoint_notice %}{% endblock %}

{% block code_query_geoPoint_within %}
```js
var query = new AV.Query('Todo');
var point = new AV.GeoPoint(39.9, 116.4);
query.withinKilometers('whereCreated', point, 2.0);
```
{% endblock %}

{% block code_send_sms_code_for_loginOrSignup %}
```js
AV.Cloud.requestSmsCode('13577778888').then(function (success) {
}, function (error) {
});
```
{% endblock %}

{% block code_verify_sms_code_for_loginOrSignup %}
```js
AV.User.signUpOrlogInWithMobilePhone('13577778888', '123456').then(function (success) {
  // Success
}, function (error) {
  // Failure
});
```
{% endblock %}

{% block code_user_signUp_with_username_and_password %}
```js
// Create a new AVUser
var user = new AV.User();
// Set username
user.setUsername('Tom');
// Set password
user.setPassword('cat!@#123');
// Set email
user.setEmail('tom@leancloud.cn');
user.signUp().then(function (loggedInUser) {
  console.log(loggedInUser);
}, function (error) {
});
```
{% endblock %}

{% block code_send_verify_email %}
```js
AV.User.requestEmailVerify('abc@xyz.com').then(function (result) {
  console.log(JSON.stringify(result));
}, function (error) {
  console.log(JSON.stringify(error));
});
```
{% endblock %}

{% block code_user_logIn_with_username_and_password %}
```js
AV.User.logIn('Tom', 'cat!@#123').then(function (loggedInUser) {
  console.log(loggedInUser);
}, function (error) {
});
```
{% endblock %}

{% block code_user_logIn_with_mobilephonenumber_and_password %}
```js
AV.User.logInWithMobilePhone('13577778888', 'cat!@#123').then(function (loggedInUser) {
  console.log(loggedInUser);
}, function (error) {
});
```
{% endblock %}

{% block code_user_logIn_requestLoginSmsCode %}
```js
AV.User.requestLoginSmsCode('13577778888').then(function (success) {
}, function (error) {
});
```
{% endblock %}

{% block code_user_logIn_with_smsCode %}
```js
AV.User.logInWithMobilePhoneSmsCode('13577778888', '238825').then(function (success) {
}, function (error) {
});
```
{% endblock %}

{% block code_get_user_properties %}
```js
AV.User.logIn('Tom', 'cat!@#123').then(function (loggedInUser) {
  console.log(loggedInUser);
  var username = loggedInUser.getUsername();
  var email = loggedInUser.getEmail();
  // Password can only be reset but not retrieved since it is encrypted when saved to the cloud
}, function (error) {
});
```
{% endblock %}

{% block code_set_user_custom_properties %}
```js
AV.User.logIn('Tom', 'cat!@#123').then(function (loggedInUser) {
  loggedInUser.set('age', 25);
  loggedInUser.save();
}, function (error) {
  // Error handling
  console.error(error);
});
```
{% endblock %}

{% block code_update_user_custom_properties %}
```js
AV.User.logIn('Tom', 'cat!@#123').then(function (loggedInUser) {
  // 25
  console.log(loggedInUser.get('age'));
  loggedInUser.set('age', 18);
  return loggedInUser.save();
}).then(function (loggedInUser) {
  // 18
  console.log(loggedInUser.get('age'));
}).catch(function (error) {
  // Error handling
  console.error(error);
});
```
{% endblock %}

{% block code_reset_password_by_email %}
```js
AV.User.requestPasswordReset('myemail@example.com').then(function (success) {
}, function (error) {
});
```
{% endblock %}

{% block code_reset_password_by_mobilephoneNumber %}
```js
AV.User.requestPasswordResetBySmsCode('18612340000').then(function (success) {
}, function (error) {
});
```
{% endblock %}

{% block code_reset_password_by_mobilephoneNumber_verify %}
```js
AV.User.resetPasswordBySmsCode('123456', 'thenewpassword').then(function (success) {
}, function (error) {
});
```
{% endblock %}

{% block code_current_user %}
```js
var currentUser = AV.User.current();
if (currentUser) {
  // Go to homepage
}
else {
  // Go to sign up page if currentUser is null
}
```
{% endblock %}

{% block code_current_user_logout %}
```js
AV.User.logOut();
// Now currentUser is null
var currentUser = AV.User.current();
```
{% endblock %}

{% block code_query_user %}
```js
var query = new AV.Query('_User');
```
{% endblock %}

{% block code_user_isAuthenticated %}
```js
var currentUser = AV.User.current();
currentUser.isAuthenticated().then(function (authenticated) {
  // console.log(authenticated); other logics here
});
```
{% endblock %}

{% block text_subclass %}{% endblock %}

{% block text_feedback %}{% endblock %}

{% block text_js_promise %}
## Promises

Each asynchronous method in LeanCloud JavaScript SDK returns a `Promise` which can be used to handle the completion and exception of the method.

```js
// This is a complete example of Promise
// Update an AV.Object after it is being queried
var query = new AV.Query('TestObject');
query.equalTo('name', 'hjiang');
// find is asynchronous which gives back a Promise that then can be called on
query.find().then(function (results) {
  // Returns a list of objects
  var obj = results[0];
  obj.set('phone', '182xxxx5548');
  // save is also asynchronous which gives back a Promise that you can return here and chain another Promise afterwards
  return obj.save();
}).then(function () {
  // The Promise returned by save method
  console.log('Successfully updated phone number');
}).catch(function (error) {
  // Put catch at the very end of Promise chain which catches all the errors
  console.error(error);
});
```

### The `then` Method

Each Promise has a method called `then` which takes in two callbacks. The first callback is called when the Promise is `resolved` (runs successfully) while the second one is called when the Promise is `rejected` (gets error).

```js
obj.save().then(function (obj) {
  // Successfully saved the object
}, function (error) {
  // Error handling
});
```

The second callback is optional.

You may also implement your logic with `catch`:

```js
obj.save().then(function (obj) {
  // Successfully saved the object
}).catch(function (error) {
  // Error handling
});
```

### Chaining Promises Together

Promise allows you to elegantly connect asynchronous requests together according to the order they should be called. If the callback of a Promise returns another Promise, the callback in the second then will not be resolved unless the one in the first then is resolved. This is also called **Promise Chain**.

```js
// Add contents to the page following the order of chapters
var chapterIds = [
  '584e1c408e450a006c676162', // Chapter One
  '584e1c43128fe10058b01cf5', // Chapter Two
  '581aff915bbb500059ca8d0b'  // Chapter Three
];

new AV.Query('Chapter').get(chapterIds[0]).then(function (chapter0) {
  // Add contents to the page
  addHtmlToPage(chapter0.get('content'));
  // Return the new Promise
  return new AV.Query('Chapter').get(chapterIds[1]);
}).then(function (chapter1) {
  addHtmlToPage(chapter1.get('content'));
  return new AV.Query('Chapter').get(chapterIds[2]);
}).then(function (chapter2) {
  addHtmlToPage(chapter2.get('content'));
  // Done
});
```

### Error Handling with Promises

If any single Promise in the chain throws an error, all the callbacks following it meant for successful operations will be skipped until an error handling callback is encountered.

It is a common practice to attach an error handling function at the end of a Promise chain.

The code above can be rewritten with `try` and `catch` in the following way:

```js
new AV.Query('Chapter').get(chapterIds[0]).then(function (chapter0) {
  addHtmlToPage(chapter0.get('content'));

  // Force an error
  throw new Error('Error');

  return new AV.Query('Chapter').get(chapterIds[1]);
}).then(function (chapter1) {
  // The code here will be ignored
  addHtmlToPage(chapter1.get('content'));
  return new AV.Query('Chapter').get(chapterIds[2]);
}).then(function (chapter2) {
  // The code here will be ignored
  addHtmlToPage(chapter2.get('content'));
}).catch(function (error) {
  // This error handling function will be called, printing out the error message "Error"
  console.error(error.message);
});
```
{% endblock %}

{% block js_push_guide %}
## Push Notifications

JavaScript SDK allows you to send push notifications to mobile devices.

The code below sends a notification to all the devices subscribed to the channel `public`:

```js
AV.Push.send({
  channels: ['public'],
  data: {
    alert: 'public message'
  }
});
```

A message containing `public message` will be sent to all the devices subscribed to the channel `public`.

If you need to send notifications based on query conditions constructed for an `_Installation` table, you can provide an `AV.Query` object as a `where` condition. For example, to send a notification to an Android device with `installationId`:

```js
var query = new AV.Query('_Installation');
query.equalTo('installationId', installationId);
AV.Push.send({
  where: query,
  data: {
    alert: 'Public message'
  }
});
```

Beside using AV.Query, you may also write a [CQL](./cql_guide.html):

```js
AV.Push.send({
  cql: 'select * from _Installation where installationId="deviceId"',
  data: {
    alert: 'Public message'
  }
});
```

For more instructions on how to use `AV.Push`, refer to its documentation [AV.Push](https://leancloud.github.io/javascript-sdk/docs/AV.Push.html). For more information regarding query conditions and formats, refer to [Push Notifications Guide](./push_guide.html).

For iOS devices, `prod` can be used to specify the certificate used for notifications:

```js
AV.Push.send({
  prod: 'dev',
  data: {
    alert: 'public message'
  }
});
```

`dev` refers to the development certificate; `prod` refers to the production certificate.
{% endblock %}

{% block use_js_in_webview %}
## WebView

JS SDK can be used in a WebView like PhoneGap, Cordova, WeChat WebView, etc.

### Android WebView

If you are using Android WebView, the following configurations need to be done when creating WebView in you Native code.

1. Our JS SDK uses window.localStorage which means that localStorage of WebView needs to be enabled:

   ```java
   yourWebView.getSettings().setDomStorageEnabled(true);
   ```

2. If you wish to debug WebView directly in your mobile devices, remote debugging needs to be configured for generating WebView. Refer to [Google's official documentation](https://developer.chrome.com/devtools/docs/remote-debugging) for more details.

   ```java
   if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
     yourWebView.setWebContentsDebuggingEnabled(true);
   }
   ```

   Keep in mind that this method is only supported on Android 4.4 and above.
3. When developing the UI of your app with WebView, Native makes use of Hybrid for the app running on your phone. We recommend that you first design the UI of your app with Chrome Developer Tools. After the UI is done, start working on data binding with Native during which you can debug WebView on your phone with Remote debugger. This will save you a lot of time on developing and debugging. If you choose to develop UI with Remote debugger as well, you will end up spending more time on it.
4. As a security practice to prevent JavaScript from calling Java functions for accessing the file system of an Android device, Android 4.2 and later versions only allow WebView to access the methods exposed by the annotation [@JavascriptInterface](http://developer.android.com/reference/android/webkit/JavascriptInterface.html). If you have users using Android 4.2 or above, make sure the annotation is added or **Uncaught TypeError** might be triggered.
{% endblock %}

{% block code_authenticate_via_sessiontoken %}
The `sessionToken` of the current user can be retrieved with `user.getSessionToken()`.

The code below logs a user in with `sessionToken`:

```js
AV.User.become(sessionToken).then(function (user) {
  // currentUser is updated
})
```
{% endblock %}

{% block file_as_avatar %}
```js
var file = AV.File.withURL('Satomi_Ishihara.gif', 'http://ww3.sinaimg.cn/bmiddle/596b0666gw1ed70eavm5tg20bq06m7wi.gif');
var todo = new AV.Object('Todo');
todo.set('girl', file);
todo.set('topic', 'Star');
todo.save();
```
{% endblock %}

{% block query_file_as_avatar %}
```js
var query = new AV.Query('Todo');
query.equalTo('topic', 'Star');
query.include('girl');
query.find().then(list => {
  list.map(todo => {
    var file = todo.get('girl');
    console.log('file.url', file.url());
  });
});
```
{% endblock %}

{% block code_pointer_include_todoFolder %}
```js
var todo = AV.Object.createWithoutData('Todo', '5735aae7c4c9710060fbe8b0');
todo.fetch({
  include:['todoFolder']
  }).then(todoObj =>{
    let todoFolder = todoObj.get('todoFolder');
    console.log(todoFolder.get('name'));
});
```
{% endblock %}

{% block anonymous_user_login %}
```js
AV.User.loginAnonymously().then(user => {
  // Successfully logged in anonymously; user is the anonymous user logged in
}).catch(function (error) {
  // Error handling
  console.error(error);
});
```
{% endblock %}

{% block setup_username_and_password_for_anonymous_user %}
```js
const currentUser = AV.User.current();
user.setUsername('username');
user.setPassword('password');
user.signUp().then(user => {
  // user is converted to a regular user
}).catch(function (error) {
  // Error handling
  console.error(error);
});
```
{%  endblock %}

{% block determine_a_user_is_anonymous %}
```js
const currentUser = AV.User.current();
if (currentUser.isAnonymous()) {
  // enableSignUpButton();
} else {
  // enableLogOutButton();
}
```
{%  endblock %}

{% block text_using_async_methods %}{% endblock %}

{% block login_with_authdata %}
```js
var authData = {
  access_token: 'ACCESS_TOKEN',
  expires_in: 7200,
  refresh_token: 'REFRESH_TOKEN',
  openid: 'OPENID',
  scope: 'SCOPE',
};

AV.User.signUpOrlogInWithAuthData(authData, 'weixin').then(function (s) {
  // Success
}, function (error) {
  // Failure
});
```
{% endblock %}

{% block login_with_authdata_result %}
```json
{
  "ACL": {
    "*": {
      "read": true,
      "write": true
    }
  },
  "username": "y43mxrnj3kvrfkt8w5gezlep1",
  "emailVerified": false,
  "authData": {
    "weixin": {
      "openid": "oTY851aFzn4TdDgujsEl0f36Huxk",
      "expires_in": 7200,
      "access_token": "11_gaS_CfX47PH3n6g33zwONEyUsFRmiWJPIEcmWVzqS48JeZjpII6uRkTD6g36GY7_5pxKciSM-v8OGnYR26DC-VBffwMHaVx5_ik8FVQdE5Y"
    }
  },
  "mobilePhoneVerified": false,
  "objectId": "5b3def469f545400310c939d",
  "createdAt": "2018-07-05T10:13:26.310Z",
  "updatedAt": "2018-07-05T10:13:26.310Z"
}
```
{% endblock %}

{% block associate_with_authdata %}
```js
var authData = {
  access_token: 'ACCESS_TOKEN',
  expires_in: 7200,
  refresh_token: 'REFRESH_TOKEN',
  openid: 'OPENID',
  scope: 'SCOPE',
};
var user = AV.User.current();
user.associateWithAuthData(authData, 'weixin').then(function (user) {
  // Success
}, function (error) {
  // Failure
});
```
{% endblock %}

{% block associate_with_authdata_result %}
```json
{
  "ACL": {
    "*": {
      "read": true,
      "write": true
    }
  },
  "username": "y43mxrnj3kvrfkt8w5gezlep1",
  "emailVerified": false,
  "authData": {
    "weixin": {
      "access_token": "11_U4Nuh9PGpfuBJqtm7KniWn48rkJ7vBTCVN2beHcVvceswua2sLU_5Afq26ZJrRF0vpSX0xcDwI-zxeo3qcf-cMftjqEvWh7Vpp05bgxeWtc",
      "expires_in": 7200,
      "openid": "oTY851aFzn4TdDgujsEl0f36Huxk"
    },
    "qq": {
      "openid": "0395BA18A5CD6255E5BA185E7BEBA242",
      "expires_in": 7200,
      "access_token": "11_CCveaBR_Lu0lmhff6NC33Lhx662zCnbzcSYhbYZQZ01YPdFav3sjhzjoM1hxs3AMMMydhguh2M0PumUaglpzuAlpzRzQn4vEXTRaZuovEnQ"
    }
  },
  "mobilePhoneVerified": false,
  "objectId": "5b3def469f545400310c939d",
  "createdAt": "2018-07-05T10:13:26.310Z",
  "updatedAt": "2018-07-06T07:46:58.097Z"
}
```
{% endblock %}

{% block login_with_authdata_without_fail %}
```js
var authData = {
  access_token: 'ACCESS_TOKEN',
  expires_in: 7200,
  refresh_token: 'REFRESH_TOKEN',
  openid: 'OPENID',
  scope: 'SCOPE',
};

AV.User.signUpOrlogInWithAuthData(authData, 'weixin', { failOnNotExist: true }).then(function (s) {
  // Success
}, function (error) {
  // Failure
  // Jump to sign up page if error.code == 211
});

var user = new AV.User();
// Set username
user.setUsername('Tom');
// Set password
user.setMobilePhoneNumber('18666666666');
user.setPassword('cat@#123');
// Set email
user.setEmail('tom@leancloud.cn');
user.loginWithAuthData(authData, 'weixin').then(function (loggedInUser) {
  console.log(loggedInUser);
}, function (error) {
});
```
{% endblock %}

{% block login_with_authdata_unionid %}
```js
var authData = {
  access_token: 'ACCESS_TOKEN',
  expires_in: 7200,
  refresh_token: 'REFRESH_TOKEN',
  openid: 'OPENID',
  scope: 'SCOPE',
};

AV.User.loginWithAuthDataAndUnionId(
  authData,
  'wxminiprogram1', 'o6_bmasdasdsad6_2sgVt7hMZOPfL',
  { unionIdPlatform: 'weixin', asMainAccount: true }
).then(function (user) {
  // Success
}, function (error) {
  // Failure 
});
```
{% endblock %}

{% block login_with_authdata_unionid_result %}
```json
"authData": {
  "weixinapp1": {
    "platform": "weixin",
    "openid": "oTY851axxxgujsEl0f36Huxk",
    "expires_in": 7200,
    "main_account": true,
    "access_token": "10_chx_dLz402ozf3TX1sTFcQQyfABgilOa-xxx-1HZAaC60LEo010_ab4pswQ",
    "unionid": "ox7NLs06ZGfdxxxxxe0F1po78qE"
  },
  "_weixin_unionid": {
    "uid": "ox7NLs06ZGfdxxxxxe0F1po78qE"
  }
}
```
{% endblock %}

{% block login_with_authdata_unionid_result_more %}
```json
"authData": {
  "weixinapp1": {
    "platform": "weixin",
    "openid": "oTY851axxxgujsEl0f36Huxk",
    "expires_in": 7200,
    "main_account": true,
    "access_token": "10_chx_dLz402ozf3TX1sTFcQQyfABgilOa-xxx-1HZAaC60LEo010_ab4pswQ",
    "unionid": "ox7NLs06ZGfdxxxxxe0F1po78qE"
  },
  "_weixin_unionid": {
    "uid": "ox7NLs06ZGfdxxxxxe0F1po78qE"
  },
  "miniprogram1": {
    "platform": "weixin",
    "openid": "ohxoK3ldpsGDGGSaniEEexxx",
    "expires_in": 7200,
    "main_account": true,
    "access_token": "10_QfDeXVp8fUKMBYC_d4PKujpuLo3sBV_pxxxxIZivS77JojQPLrZ7OgP9PC9ZvFCXxIa9G6BcBn45wSBebsv9Pih7Xdr4-hzr5hYpUoSA",
    "unionid": "ox7NLs06ZGfdxxxxxe0F1po78qE"
  }
}
```
{% endblock %}
