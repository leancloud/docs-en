# ACL Guide

ACL stands for Access Control List.
It defines access permissions for an object.
For example:

```json
{
  // everyone
  "*":{
    "read":true,
    "write":false
  },
  "role:admin":{
    "read":true,
    "write":true
  },
  // a certain user
  "58113fbda0bb9f0061ddc869":{
    "read":true,
    "write":true
  }
}
```

## Default ACL

When creating a new class on the dashboard, you can configure its default ACL in the dialog box.

![create class dialog box](images/security/acl_template.png)

You can also modify default ACL for existing classes.
To do so, simply go to **Dashboard > LeanStorage > Objects**, select the class, and click the **Permission** tab.

You can also configure ACL for a single object on the dashboard.
However, configure ACL for every object manually is too tedious, sometimes impossible.
Thus it is recommended that you configure **default ACL** on the dashboard, ensuring that all new objects have proper initial ACL, and configure finer ACL for a single object via setting its `ACL` field in code.


For example, a `Post` class may have the following default ACL:

![default ACL](images/acl.png)

Therefore, everyone can read the post but only the author can write the post.
Here the object creator refers to the corresponding user to the session token carried in the HTTP header of the object creation request, that is, the author of the post.

## ACL with Users

Let's continue with the example above.
Suppose you want to also allow a coauthor to modify a post:

```objc
AVACL *acl = [AVACL ACL];
// Everyone can read.
[acl setPublicReadAccess:YES];
// The author and coauthor can write.
[acl setWriteAccess:YES forUser:[AVUser currentUser]];
AVUser *coauthor = [AVUser objectWithClassName:@"_User" objectId:@"55f1572460b2ce30e8b7afde"];
[acl setWriteAccess:YES forUser:coauthor];
post.ACL = acl;
```
```swift
let acl = LCACL()
// Everyone can read.
acl.setAccess([.read], allowed: true)
// The author and coauthor can write.
if let currentUserID = LCApplication.default.currentUser?.objectId?.value {
    acl.setAccess([.write], allowed: true, forUserID: currentUserID)
}
let coauthorID = "55f1572460b2ce30e8b7afde"
acl.setAccess([.write], allowed: true, forUserID: coauthorID)
post.ACL = acl
```
```java
AVACL acl = new AVACL();
// Everyone can read.
acl.setPublicReadAccess(true);
// The author and coauthor can write.
acl.setWriteAccess(AVUser.getCurrentUser(), true);
AVUser coauthor = AVUser.createWithoutData("_User", "55f1572460b2ce30e8b7afde");
acl.setWriteAccess(coauthor, true);
post.setACL(acl);
```
```js
const acl = new AV.ACL();
// Everyone can read.
acl.setPublicReadAccess(true);
// The author and coauthor can write.
acl.setWriteAccess(AV.User.current(),true);
const coauthor = AV.User.createWithoutData('_User', '55f1572460b2ce30e8b7afde');
acl.setWriteAccess(coauthor, true);
post.setACL(acl);
```
```python
acl = leancloud.ACL()
# Everyone can read.
acl.set_public_read_access(True)
# The author and coauthor can write.
current_user_id = leancloud.User.get_current().id
acl.set_write_access(current_user_id)
coauthor_id = '55f1572460b2ce30e8b7afde' 
acl.set_write_access(coauthor_id, True)
post.set_acl(acl)
```
```php
$acl = new ACL();
# Everyone can read.
$acl->setPublicReadAccess(true);
# The author and coauthor can write.
$acl->setWriteAccess(User::getCurrentUser(), true);
$coauthor = LeanObject::create("_User", "55f1572460b2ce30e8b7afde");
$acl->setWriteAccess($coauthor, true);
$post->setACL($acl) 
```
```dart
LCACL acl = LCACL();
// Everyone can read.
acl.setPublicReadAccess(true);
// The author and coauthor can write.
acl.setUserWriteAccess(currentUser, true);
LCUser coauthor = LCUser.createWithoutData('_User', '55f1572460b2ce30e8b7afde');
acl.setUserWriteAccess(coauthor, true);
post.acl = acl;
```

The `ACL` field of this post will be something like:

```json
{
  "*":{
    "read":true
  },
  "55b9df0400b0f6d7efaa8801":{
    "write":true
  },
  "55f1572460b2ce30e8b7afde":{
    "write":true
  }
}
```

Similarly, if you want to allow the administrator to modify and delete all posts, you can grant write permission to them for every post:

```objc
[acl setWriteAccess:YES forUser:anAdministrator];
```
```swift
acl.setAccess([.write], allowed: true, forUserID: anAdministratorID)
```
```java
acl.setWriteAccess(anAdministrator, true);
```
```js
acl.setWriteAccess(anAdministrator, true);
```
```python
acl.set_write_access(an_administrator_id, True)
```
```php
$acl->setReadAccess($anAdministrator, true);
```
```dart
acl.setUserWriteAccess(anAdministrator, true);
```

However, this is inflexible in practice.
There may be multiple administrators and administrators come and go.
Thus let's introduce a new concept for ACL, *role*.

## ACL with Roles
