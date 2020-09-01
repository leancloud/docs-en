# Using ACL in LeanEngine

LeanEngine offers a way for you to define logic on the cloud to perform certain actions when certain events happen. When you need to print logs, verify permissions, or enforce ACL settings on data operations initiated by clients, this could be very helpful. The documentation listed below offer more detailed explanations on such functionality:

- [Node.js Cloud Function Guide](leanengine_cloudfunction_guide-node.html){% if platformName === "Node.js" %} (current){% endif %}
- [Python Cloud Function Guide](leanengine_cloudfunction_guide-python.html){% if platformName === "Python" %} (current){% endif %}
- [PHP Cloud Function Guide](leanengine_cloudfunction_guide-php.html){% if platformName === "PHP" %} (current){% endif %}
- [Java Cloud Function Guide](leanengine_cloudfunction_guide-java.html){% if platformName === "Java" %} (current){% endif %}

In this documentation, you will learn how to enforce ACL settings on objects created by clients.

Imagine that you are building an application for iOS, Android, and web (JavaScript), and you need to implement a function that adds permission settings to all the objects created. Traditionally, you will need to write the same function in different languages for each platform. But now you can write the same function only once and put it on the cloud, which makes the development process way easier.

Let's say that your application wants the administrators of the website to have the permission to read and write all the posts created by users. The first thing you need to do is to create a [hook](leanengine_cloudfunction_guide-node.html#beforesave) that can be triggered before a post is saved:

```js
AV.Cloud.beforeSave('Post', (request) => {
  const post = request.object;
  if (post) {
    var acl = new AV.ACL();
    acl.setPublicReadAccess(true);
    // Assuming a role named "admin" exists.
    acl.setRoleWriteAccess('admin', true);
    post.setACL(acl);
  } else {
    throw new AV.Cloud.Error('Invalid Post object.');
  }
});
```
```python
@engine.before_save('Post')
def before_post_save(post):
    acl = leancloud.ACL()
    acl.set_public_read_access(True)
    # Assuming a role named "admin" exists.
    admin = leancloud.Role('admin')
    acl.set_role_write_access(admin, True)
    post.set_acl(acl)
```
```php
Cloud::beforeSave("Post", function($post, $user) {
  $acl = new ACL();
  $acl->setPublicReadAccess(true);
  $acl->setRoleWriteAccess("admin", true);
  $post->setACL($acl);
});
```
```java
@EngineHook(className = "Post", type = EngineHookType.beforeSave)
public static AVObject postBeforeSaveHook(AVObject post) throws Exception {
  AVACL acl = new AVACL();
  acl.setPublicReadAccess(true);
  acl.setRoleWriteAccess("admin", true);
  post.setACL(acl);
  return post;
}
```

After creating the hook, deploy the code to the cloud.
From now on, for each post created on the client-side, it will have the following ACL automatically:

```json
{"*":{"read":true},"role:admin":{"write":true}}
```
