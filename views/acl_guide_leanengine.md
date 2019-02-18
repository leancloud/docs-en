# Using ACL in LeanEngine

LeanEngine offers a way for you to define logic on the cloud to perform certain actions when certain events happen. When you need to print logs, verify permissions, or enforce ACL settings on data operations initiated by clients, this could be very helpful. The documentations listed below offer more detailed explanations on such functionality:

- [Node.js Cloud Function Guide](leanengine_cloudfunction_guide-node.html){% if platformName === "Node.js" %} (current){% endif %}
- [Python Cloud Function Guide](leanengine_cloudfunction_guide-python.html){% if platformName === "Python" %} (current){% endif %}
- [PHP Cloud Function Guide](leanengine_cloudfunction_guide-php.html){% if platformName === "PHP" %} (current){% endif %}
- [Java Cloud Function Guide](leanengine_cloudfunction_guide-java.html){% if platformName === "Java" %} (current){% endif %}

In this documentation, you will learn how to enforce ACL settings on objects created by clients.

Imagine that you are building an application for iOS, Android, and web (JavaScript), and you need to implement a function that adds permission settings to all the objects created. Traditionally, you will need to write the same function in different languages for each platform. But now you can write the same function only once and put it on the cloud, which makes the development process way easier.

Let's say that your application wants the administrators of the website to have the permission to read and write all the posts created by users. The first thing you need to do is to create a [hook](leanengine_cloudfunction_guide-node.html#beforesave) that can be triggered before a post is saved:

```js
AV.Cloud.beforeSave('Post', function (request, response) {
  // If the role Administrator already exists, you can find out the AV.Role instance for it with the following code:
  /*
    var roleQuery = new AV.Query(AV.Role);
    roleQuery.equalTo('name', 'Administrator');
    roleQuery.find().then(function (results) {
      var administratorRole = results[0];
    });
  */
  var post = request.object;
  if (post) {
    // Create an ACL instance
    var acl = new AV.ACL();
    acl.setPublicReadAccess(true);
    // Set permission with the name of the role
    acl.setRoleWriteAccess('Administrator', true);
    // You can also use an AV.Role instance instead of a name:
    // acl.setRoleWriteAccess(administratorRole, true);

    post.setACL(acl);

    // Save the object
    response.success();
  } else {
    // Return an error without saving
    response.error('Invalid Post object.');
  }
});
```

After creating the hook, [deploy](leanengine_webhosting_guide-node.html#deploying-and-publishing) the code to the cloud. Now try to create a `Post` on a client:

**iOS**

```objc
// Create a new post
AVObject *post = [AVObject objectWithClassName:@"Post"];
[post setObject:@"Hello" forKey:@"title"];
[post setObject:@"Is there anyone here?" forKey:@"content"]
[post saveInBackground]; // Save
```

**Android**

```java
// Create a new post
AVObject post = new AVObject('Post');
post.put('title', 'Hello');
post.put('content', 'Is there anyone here?');
post.saveInBackground(); // Save
```

Then open the web console and take a look at the ACL of the new `Post`:

```json
{"*":{"read":true},"role:Administrator":{"write":true}}
```

You will see that the permission setting is already applied to the new `Post`.
