# ACL Guide

ACL stands for Access Control List.
It defines access permissions for an object.
For example:

```json
{
  // anyone
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

The initial default ACL value for all classes is:

```json
{
  "*":{
    "read":true,
    "write":true
  }
}
```

In other words, anyone can read and write.

When creating a new class on the dashboard, you can configure its default ACL in the dialog box.

![create class dialog box](images/security/acl_template.png)

You can also modify default ACL for existing classes.
To do so, simply go to **Dashboard > LeanStorage > Objects**, select the class, and click the **Permission** tab.
Be aware that modifying default ACL will only take effect on new objects.
It will not affect the ACL field of existing objects. 

You can also configure ACL for a single object on the dashboard.
However, configure ACL for every object manually is too tedious, sometimes impossible.
Thus it is recommended that you configure **default ACL** on the dashboard, ensuring that all new objects have proper initial ACL, and configure finer ACL for a single object via setting its `ACL` field in code.

For example, a `Post` class may have the following default ACL:

![default ACL](images/acl.png)

Therefore, anyone can read the post but only the author can write the post.
Here the object creator refers to the corresponding user to the session token carried in the HTTP header of the object creation request, that is, the author of the post.

## ACL with Users

Let's continue with the example above.
Suppose you want to also allow a coauthor to modify a post:

```objc
AVACL *acl = [AVACL ACL];
// anyone can read.
[acl setPublicReadAccess:YES];
// The author and coauthor can write.
[acl setWriteAccess:YES forUser:[AVUser currentUser]];
AVUser *coauthor = [AVUser objectWithClassName:@"_User" objectId:@"55f1572460b2ce30e8b7afde"];
[acl setWriteAccess:YES forUser:coauthor];
post.ACL = acl;
```
```swift
let acl = LCACL()
// anyone can read.
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
// anyone can read.
acl.setPublicReadAccess(true);
// The author and coauthor can write.
acl.setWriteAccess(AVUser.getCurrentUser(), true);
AVUser coauthor = AVUser.createWithoutData("_User", "55f1572460b2ce30e8b7afde");
acl.setWriteAccess(coauthor, true);
post.setACL(acl);
```
```js
const acl = new AV.ACL();
// anyone can read.
acl.setPublicReadAccess(true);
// The author and coauthor can write.
acl.setWriteAccess(AV.User.current(),true);
const coauthor = AV.User.createWithoutData('_User', '55f1572460b2ce30e8b7afde');
acl.setWriteAccess(coauthor, true);
post.setACL(acl);
```
```python
acl = leancloud.ACL()
# anyone can read.
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
# anyone can read.
$acl->setPublicReadAccess(true);
# The author and coauthor can write.
$acl->setWriteAccess(User::getCurrentUser(), true);
$coauthor = LeanObject::create("_User", "55f1572460b2ce30e8b7afde");
$acl->setWriteAccess($coauthor, true);
$post->setACL($acl); 
```
```dart
LCACL acl = LCACL();
// anyone can read.
acl.setPublicReadAccess(true);
// The author and coauthor can write.
LCUser currentUser = await LCUser.getCurrent();
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

A role is a group of users.
Roles can be nested.
In other words, the member of a role is either a user or another role.

Each role has an immutable name, consist of alphanumerics and underscores.

Continuing the example above, let's see how to grant write permission to the administrator role (assuming there is an existing role named "admin"):

```objc
AVRole *admin = [AVRole objectWithClassName:@"_Role" objectId:@"55fc0eb700b039e44440016c"];
[acl setWriteAccess:YES  forRole:admin];
```
```swift
acl.setAccess([.write], allowed: true, forRoleName:"admin")
```
```java
acl.setRoleWriteAccess("admin", true);
```
```js
const admin = new AV.Role('admin');
acl.setRoleWriteAccess(admin, true);
```
```python
admin = leancloud.Role('admin')
acl.set_role_write_access(admin, True)
```
```php
$acl->setRoleWriteAccess("admin", true);
```
```dart
LCRole admin = LCRole.createWithoutData('_Role', '55fc0eb700b039e44440016c');
acl.setRoleWriteAccess(admin, true);
```

### Role Creation

Now let's see how to create a role.

```objc
// ACL for the role itself.
AVACL *roleACL = [AVACL ACL];
[roleACL setPublicReadAccess:YES];
[roleACL setWriteAccess:YES forUser:[AVUser currentUser]];

AVRole *admin = [AVRole roleWithName:@"admin" acl:roleACL];
[admin save];
```
```swift
do {
  // ACL for the role itself.
  let roleACL = LCACL()
  roleACL.setAccess([.read], allowed: true)
  if let currentUserID = LCApplication.default.currentUser?.objectId?.value {
    roleACL.setAccess([.write], allowed: true, forUserID: currentUserID)
  }

  let admin = LCRole(name: "admin")
  admin.ACL = roleACL
  assert(admin.save().isSuccess)
} catch {
  print(error)
}
```
```java
// ACL for the role itself.
AVACL roleACL = new AVACL();
roleACL.setPublicReadAccess(true);
roleACL.setWriteAccess(AVUser.getCurrentUser(),true);

AVRole admin = new AVRole("admin", roleACL);
admin.saveInBackground().blockingSubscribe();
```
```js
// ACL for the role itself.
const roleAcl = new AV.ACL();
roleAcl.setPublicReadAccess(true);
roleAcl.setWriteAccess(AV.User.current(), true);

const admin = new AV.Role('admin', roleAcl);
await admin.save();
```
```python
# ACL for the role itself.
role_acl = leancloud.ACL()
role_acl.set_public_read_access(True)
current_user_id = leancloud.User.get_current().id
role_acl.set_write_access(current_user_id)

admin = leancloud.Role('admin', role_acl)
admin.save()
```
```php
// ACL for the role itself.
$roleACL = new ACL();
$roleACL->setPublicReadAccess(true);
$roleACL->setWriteAccess(User::getCurrentUser(), true);

$admin = new Role();
$admin->setName("admin");
$admin->setACL($roleACL)
$admin->save();
```
```dart
try {
  // ACL for the role itself.
  LCACL acl = LCACL();
  acl.setPublicReadAccess(true);
  LCUser currentUser = await LCUser.getCurrent();
  acl.setUserWriteAccess(currentUser);

  LCRole admin = LCRole.create('admin', roleACL);
  await admin.save();
} on LCException catch (e) {
  print('${e.code} : ${e.message}');
}
```

We have just created an empty role.
Let's add a user (the current user) to this role:

```objc
[[admin users] addObject: [AVUser currentUser]];
```
```swift
if let currentUser = LCApplication.default.currentUser {
    if let users = admin.users {
        try? users.insert(currentUser)
    } else {
        let users = admin.relationForKey("users")
        if (try? users.insert(currentUser)) != nil {
          admin.users = users
        }   
    }
}
```
```java
admin.getUsers().add(AVUser.getCurrentUser());
```
```js
admin.getUsers().add(AV.User.current());
```
```python
admin.get_users().add(leancloud.User.get_current())
```
```php
$admin->getUsers().add(User::getCurrentUser());
```
```dart
LCUser currentUser = await LCUser.getCurrent();
admin.addRelation('users', currentUser);
```

And here is how to remove a user from a role:

```objc
[[admin users] removeObject:[AVUser currentUser]];
```
```swift
if let currentUser = LCApplication.default.currentUser {
  try? admin.users?.remove(currentUser)
}
```
```java
admin.getUsers().remove(AVUser.getCurrentUser());
```
```js
admin.getUsers().remove(AV.User.current());
```
```python
admin.get_users().remove(leancloud.User.get_current())
```
```php
$admin->getUsers().remove(User::getCurrentUser());
```
```dart
LCUser currentUser = await LCUser.getCurrent();
admin.removeRelation('users', currentUser);
```

As mentioned above, a member of a role can be another role.
Suppose that we have created two roles `admin` and `moderator`, and we want to make `admin` a sub-role of `moderator` since every admin should also be a moderator:

```objc
[[moderator roles] addObject:admin];
```
```swift
if let subroles = moderator.roles {
  try? subroles.insert(admin)
} else {
  let subroles = moderator.relationForKey("roles")
  if (try? subroles.insert(currentUser)) != nil {
    admin.roles = subroles
  }
}
```
```java
moderator.getRoles().add(admin);
```
```js
moderator.getRoles().add(admin);
```
```python
moderator.get_roles().add(admin)
```
```php
$moderator->getRoles().add(admin);
```
```dart
moderator.addRelation('roles', admin);
```

Occasionally we may want to remove a sub-role.
For example, later we changed our mind and decided that administrators should focus on administrative tasks, leaving post-editing to moderators.

```objc
[[moderator roles] removeObject:admin];
```
```swift
try? moderator.roles?.remove(admin)
```
```java
moderator.getRoles().remove(admin);
```
```js
moderator.getRoles().remove(admin);
```
```python
moderator.get_roles().remove(admin)
```
```php
$moderator->getRoles().remove(admin);
```
```dart
moderator.removeRelation('roles', admin);
```

Lastly, do not forget to save it after modifying a role.

### Role Query

Often we want to know the roles a user belongs to:

```objc
AVUser *user = [AVUser currentUser];
[user getRolesInBackgroundWithBlock:^(NSArray<AVRole *> * _Nullable avRoles, NSError * _Nullable error) {
  // avRoles is the result
}];
```
```swift
if let user = LCApplication.default.currentUser {  
    let roleQuery = LCQuery(className: LCRole.objectClassName())   
    roleQuery.whereKey("users", .equalTo(user))   
    _ = roleQuery.find { result in
        switch result {
        case .success(objects: let roles):
            print(roles)
        case .failure(error: let error):
            print(error)
        }
    }
}
```
```java
AVUser user = AVUser.getCurrentUser();
user.getRolesInBackground().subscribe(new Observer<List<AVRole>>() {
    @Override public void onSubscribe(Disposable d) {}
    @Override public void onNext(List<AVRole> avRoles) {
        // avRoles is the result
    }
    @Override public void onError(Throwable e) {}
    @Override public void onComplete() {}
});
```
```js
const roles = await AV.User.current().getRoles();
```
```python
roles = leancloud.User.get_current().get_roles()
```
```php
$roles = User::getCurrentUser().getRoles();
```
```dart
try {
  LCUser currentUser = await LCUser.getCurrent();
  LCQuery roleQuery = LCRole.getQuery();
  roleQuery.whereEqualTo('users', currentUser);
  List<LCRole> roles = await roleQuery.find();
} on LCException catch (e) {
  print('${e.code} : ${e.message}');
}
```

Sometimes we want to know the users contained in a rule.
To do so, we can construct a query as below:

```objc
AVUser *userQuery = [[moderator users] query];
```
```swift
let *userQuery = moderator.users?.query
```
```java
AVQuery<AVUser> userQuery = moderator.getUsers().getQuery();
```
```js
const userQuery = moderator.getUsers().query();
```
```python
user_query = moderator.get_users().query
```
```php
$userQuery = $moderator->getUsers().getQuery();
```
```dart
LCQuery userQuery = moderator.users.query();
```

However, the above queries do not consider users in the sub-roles of the moderator role.
To query all users belongs to the moderator role, directly and indirectly, we need to recursively query all sub-roles of the moderator roles (including sub-sub-roles and so on), then query users of all these roles.
For brevity, we only demonstrate how to query direct sub-roles of a role here:

```objc
AVRole *subroleQuery = [[moderator roles] query];
```
```swift
let *subRoleQuery = moderator.roles?.query
```
```java
AVQuery<AVRole> subroleQuery = moderator.getRoles().getQuery();
```
```js
const subroleQuery = moderator.getRoles().query();
```
```python
subrole_query = moderator.get_roles().query
```
```php
$subRoleQuery = $moderator->getRoles().getQuery();
```
```dart
LCQuery subroleQuery = moderator.roles.query();
```

You can also query roles based on other attributes,
just like querying a normal object.

## Special Rules

The `_User` class has a special rule.
Without using masterKey (which will skip all permission checks), each user can only modify their user data, no matter what write permission granted in the ACL.

## Retrieving ACL value

The ACL value will not return to the client-side by default.
To also retrieve the ACL field of an object, you have to enable **Include ACL with objects being queried** in **Dashboard > LeanStorage > Settings**, and explicitly specify so when constructing a query:

```objc
query.includeACL = YES;
```
```swift
query.includeACL = true
```
```java
query.includeACL(true);
```
```js
query.includeACL(true);
```
```python
query.include_acl(True)
```
```php
// not supported yet
```
```dart
query.includeACL(true);
```

