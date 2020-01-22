{% import "views/_helper.njk" as docs %}
{% import "views/_sms.njk" as sms %}
{% import "views/_parts.html" as include %}

# Short Message Service (SMS) Guide

There are plenty of scenarios in which your app may need to send text messages to your users:

- User verification: When a user attempts to log in or change their password, a text message containing verification code can be sent to the user to ensure the security of the account.
- Operation verification: For apps demanding high security levels (like banking apps), when a user performs sensitive operations like transferring money or making payments, a verification code can be helpful to confirm if the user themself is performing the operation.
- Notifications and marketing: Merchants may send users notifications containing status updates regarding their orders, or campaigns regarding new products and promotions.

After enabling SMS in your app's dashboard (see [Enabling SMS](#enabling-sms)), you will be able to send text messages to your users through your app:

```objc
AVShortMessageRequestOptions *options = [[AVShortMessageRequestOptions alloc] init];
options.templateName = @"Register_Notice"; // The name of the template stored on dashboard
options.signatureName = @"LeanCloud";      // The name of the signature stored on dashboard
// Sending text message to +19490008888 using the template and signature
[AVSMS requestShortMessageForPhoneNumber:@"+19490008888"
                                options:options
                                callback:^(BOOL succeeded, NSError * _Nullable error) {
                                    if (succeeded) {
                                        /* Request completed */
                                    } else {
                                        /* An error occurred */
                                    }
                                }];
```
```swift
_ = LCSMSClient.requestShortMessage(
    mobilePhoneNumber: "+19490008888",
    templateName: "Register_Notice", // The name of the template stored on dashboard
    signatureName: "LeanCloud")      // The name of the signature stored on dashboard
{ (result) in
    switch result {
    case .success:
        break
    case .failure(error: let error):
        print(error)
    }
}
```
```java
AVSMSOption option = new AVSMSOption();
option.setTemplateName("Register_Notice"); // The name of the template stored on dashboard
option.setSignatureName("LeanCloud");      // The name of the signature stored on dashboard
// Sending text message to +19490008888 using the template and signature
AVSMS.requestSMSCodeInBackground("+19490008888", option).subscribe(new Observer<AVNull>() {
    @Override
    public void onSubscribe(Disposable disposable) {
    }
    @Override
    public void onNext(AVNull avNull) {
        Log.d("TAG","Result: Successfully sent text message.");
    }
    @Override
    public void onError(Throwable throwable) {
        Log.d("TAG","Result: Failed to send text message. Reason: " + throwable.getMessage());
    }
    @Override
    public void onComplete() {
    }
});
```
```javascript
// Sending text message to +19490008888 using the template and signature
AV.Cloud.requestSmsCode({
  mobilePhoneNumber: '+19490008888', // Target number
  template: 'Register_Notice',      // The name of the template stored on dashboard
  sign:'LeanCloud'                  // The name of the signature stored on dashboard
}).then(function(){
  // Request completed
}, function(err){
  // An error occurred
});
```
```cs
// Sending text message to +19490008888 using the template (Register_Notice) and signature (LeanCloud)
AVCloud.RequestSMSCodeAsync("+19490008888","Register_Notice",null,"LeanCloud").ContinueWith(t =>
{
    var result = t.Result;
    // result being True means the request is completed
});
```
```php
// Sending text message to +19490008888 using the template (Register_Notice) and signature (LeanCloud)
$options = [
  "template" => "Register_Notice",
  "name" => "LeanCloud",
];
SMS::requestSMSCode("+19490008888", $options);
```
```python
from leancloud import cloud
cloud.request_sms_code("+19490008888", template="Register_Notice", sign="LeanCloud")
```

The actual message received by the user looks like this:

{% call docs.bubbleWrap() -%}
【LeanCloud】Thanks for joining LeanCloud, the leading backend-as-a-service provider.
{% endcall %}

- The content of the text message comes from the [template](#templates) named `Register_Notice` which needs to be created in your app's dashboard in advance.
- `LeanCloud` is the [signature](#signatures) which is required and also needs to be created in your app's dashboard before you use it.

## Enabling SMS

### Enabling SMS in Security Settings

To start using SMS, make sure you already have an app created, then go to the app's [Dashboard > Settings > Security](https://console.leancloud.app/app.html?appid={{appid}}#/security) and enable **SMS**:

<img src="images/sms_switch_setting.png" width="600" class="img-responsive" alt="&quot;SMS&quot; under &quot;Service switches&quot; is enabled.">

### Completing SMS Settings

Then go to [Dashboard > Messaging > SMS > Settings > SMS settings](https://console.leancloud.app/messaging.html?appid={{appid}}#/message/sms/conf) and ensure the following option is enabled:

{{ include.checkbox(true) }}**Enable SMS verification code (open up `requestSmsCode` and `verifySmsCode`)**
- Enabled: The app is able to incorporate features related to SMS, including verification when there are users performing sensitive operations, logging in at unusual locations, making payments, etc.
- Disabled: Requests for sending and verifying verification codes will be rejected. Note that this won't affect verification for user accounts.

### Setting up Default Signatures

A signature helps your users identify the sender of the messages they received. Before you start sending your first message, you need to go to your app's [Dashboard > Messaging > SMS > Settings](https://console.leancloud.app/messaging.html?appid={{appid}}#/message/sms/conf) and set up a default signature (the first signature you created automatically becomes the default one):

![The "New signature" button under "Signatures".](images/sms_create_signature.png)

{#
Simply enter a name for yourself along with the content:

<img src="images/sms_signature_edit.png" width="600" class="img-responsive" alt="在「短信签名」对话框中填入「名称」和「签名」。">
#}

After creating a signature, you will be able to call LeanCloud APIs to send text messages. You might have noticed that there is a "template" appearing in the sample code, but since it is not required by all types of text messages, we will [discuss it later](#templates).

## Using SMS for Verification

It is becoming quite common that apps rely on SMS for signing up, logging in, and performing verification for sensitive operations. Here we use a shopping app as an example to explain how you can use LeanCloud's SMS to complete a verification:

1. **The user places an order**  
  A sensitive operation is initiated.
  
2. **Call API to send text message**  
  Keep in mind that we are assuming that you have already completed the setup process mentioned earlier.

  ```objc
  AVShortMessageRequestOptions *options = [[AVShortMessageRequestOptions alloc] init];
  options.TTL = 10;                      // Make verification code valid for 10 minutes
  options.applicationName = @"App Name"; // The name of the app
  options.operation = @"some operation"; // The name of the operation
  [AVSMS requestShortMessageForPhoneNumber:@"+19490008888"
                                  options:options
                                  callback:^(BOOL succeeded, NSError * _Nullable error) {
                                      if (succeeded) {
                                          /* Request completed */
                                      } else {
                                          /* An error occurred */
                                      }
                                  }];
  ```
  ```swift
  let variables: LCDictionary = [
      "ttl": LCNumber(10),             // Make verification code valid for 10 minutes
      "name": LCString("App Name"),    // The name of the app
      "op": LCString("some operation") // The name of the operation
  ]

  _ = LCSMSClient.requestShortMessage(mobilePhoneNumber: "+19490008888", variables: variables) { (result) in
      switch result {
      case .success:
          break
      case .failure(error: let error):
          print(error)
      }
  }
  ```
  ```java
  AVSMSOption option = new AVSMSOption();
  option.setTtl(10);
  option.setApplicationName("App Name");
  option.setOperation("some operation");
  AVSMS.requestSMSCodeInBackground("+19490008888", option).subscribe(new Observer<AVNull>() {
      @Override
      public void onSubscribe(Disposable disposable) {
      }
      @Override
      public void onNext(AVNull avNull) {
          Log.d("TAG","Result: Successfully sent verification code.");
      }
      @Override
      public void onError(Throwable throwable) {
          Log.d("TAG","Result: Failed to send verification code. Reason: " + throwable.getMessage());
      }
      @Override
      public void onComplete() {
      }
  });
  ```
  ```javascript
  AV.Cloud.requestSmsCode({
      mobilePhoneNumber: '+19490008888',
      name: 'App Name',
      op: 'some operation',
      ttl: 10 // Make verification code valid for 10 minutes
  }).then(function(){
      // Request completed
  }, function(err){
      // An error occurred
  });
  ```
  ```cs
  // Here 10 means to make verification code valid for 10 minutes
  AVCloud.RequestSMSCodeAsync("+19490008888","App Name","some operation",10).ContinueWith(t =>
  {
      if(!t.Result)
      {
          // Request completed
      }
  });
  ```
  ```php
  $options = [
    "name" => "App Name",
    "op" => "some operation",
    "ttl" => 10, // Make verification code valid for 10 minutes
  ];
  SMS::requestSMSCode("+19490008888", $options);
  ```
  ```python
  from leancloud import cloud
  options = {
    "op": "some operation",
    "ttl": 10  # Make verification code valid for 10 minutes
  }
  cloud.request_sms_code("+19490008888", sign="App Name", params=options)
  ```

3. **The user receives the text message and enters the code**  
  Before continuing, we suggest that you implement verification on the client side to check if the code entered is in a valid format (has valid length and does not contain invalid characters). This helps your app avoid making unnecessary requests to the server and could potentially enhance the user experience.
  
4. **Call API to check if the code is valid**  
  Please pay attention to the sequence of the parameters being passed into the function. Here we assume the verification code is "123456":

  ```objc
  [AVOSCloud verifySmsCode:@"123456" mobilePhoneNumber:@"+19490008888" callback:^(BOOL succeeded, NSError *error) {
      if(succeeded){
          // Successfully verified
      }
  }];
  ```
  ```swift
  _ = LCSMSClient.verifyMobilePhoneNumber("+19490008888", verificationCode: "123456") { (result) in
      switch result {
      case .success:
          break
      case .failure(error: let error):
          print(error)
      }
  }
  ```
  ```java 
  AVSMS.verifySMSCodeInBackground("123456","+19490008888").subscribe(new Observer<AVNull>() {
      @Override
      public void onSubscribe(Disposable d) {
      }
      @Override
      public void onNext(AVNull avNull) {
          Log.d("TAG","Result: Successfully verified the number.");
      }
      @Override
      public void onError(Throwable throwable) {
          Log.d("TAG","Result: Failed to verify the number. Reason: " + throwable.getMessage());
      }
      @Override
      public void onComplete() {
      }
  });
  ```
  ```javascript
  AV.Cloud.verifySmsCode('123456', '+19490008888').then(function(){
      // Successfully verified
  }, function(err){
      // Failed to verify
  });
  ```
  ```cs
  AVCloud.VerifySmsCodeAsync("123456","+19490008888").ContinueWith(t =>{
      if(t.Result) 
      {
          // Successfully verified
      }
  });
  ```
  ```php
  // Note that the sequence of the parameters for PHP SDK is different than that of many other SDKs. Here verification code goes after phone number.
  SMS::verifySmsCode('+19490008888', '123456');
  ```
  ```python
  from leancloud import cloud
  # Note that the sequence of the parameters for Python SDK is different than that of many other SDKs. Here verification code goes after phone number.
  cloud.verify_sms_code('+19490008888', '123456')
  ```

The same verification logic can be applied to other scenarios like logging in at unusual locations or updating sensitive information. You would call the same API and follow the same steps. The only thing you need to do is to design a UI that fits your requirements.

### Sending Verification Code by Calling

Although the delivery rate of text messages is pretty close to 100%, some apps demand even higher rate and expects better security at the same time. That's why we also provide the service to deliver verification code via phone call. The code below shows how you can use it:

```objc
AVShortMessageRequestOptions *options = [[AVShortMessageRequestOptions alloc] init];
options.type = AVShortMessageTypeVoice;
[AVSMS requestShortMessageForPhoneNumber:@"+19490008888"
        options:options
        callback:^(BOOL succeeded, NSError * _Nullable error) {
            if (succeeded) {
                NSLog(@"A call containing verification code has been made.");
        }
}];
```
```swift
_ = LCSMSClient.requestVoiceVerificationCode(mobilePhoneNumber: "+19490008888") { (result) in
    switch result {
    case .success:
        break
    case .failure(error: let error):
        print(error)
    }
}
```
```java
AVSMSOption option = new AVSMSOption();
option.setType(AVSMS.TYPE.VOICE_SMS);
AVSMS.requestSMSCodeInBackground("+19490008888", option).subscribe(new Observer<AVNull>() {
    @Override
    public void onSubscribe(Disposable disposable) {
    }
    @Override
    public void onNext(AVNull avNull) {
        Log.d("TAG","Result: Successfully made a call.");
    }
    @Override
    public void onError(Throwable throwable) {
        Log.d("TAG","Result: Failed to make a call. Reason: " + throwable.getMessage());
    }
    @Override
    public void onComplete() {
    }
});
```
```js
AV.Cloud.requestSmsCode({
  mobilePhoneNumber: '+19490008888',
  smsType: 'voice'
}).then(function() {
  // Call made successfully
}).catch(function(error) {
  // Error handling
});
```
```cs
AVCloud.RequestVoiceCodeAsync ("+19490008888").ContinueWith(t =>{
    // Call made successfully
});
```
```php
$options = [
  "smsType": "voice",
];
SMS::requestSMSCode("+19490008888", $options);
```
```python
from leancloud import cloud
cloud.request_sms_code("+19490008888", sms_type="voice")
```

Now the user would receive a phone call telling them the 6-digit verification code. Call the following method to complete the verification:

```objc
[AVOSCloud verifySmsCode:@"123456" mobilePhoneNumber:@"+19490008888" callback:^(BOOL succeeded, NSError *error) {
    if(succeeded){
        // Successfully verified
    }
}];
```
```swift
_ = LCSMSClient.verifyMobilePhoneNumber("+19490008888", verificationCode: "123456") { (result) in
    switch result {
    case .success:
        break
    case .failure(error: let error):
        print(error)
    }
}
```
```java
AVSMS.verifySMSCodeInBackground("123456","+19490008888").subscribe(new Observer<AVNull>() {
    @Override
    public void onSubscribe(Disposable d) {
    }
    @Override
    public void onNext(AVNull avNull) {
        Log.d("TAG","Result: Successfully verified the number.");
    }
    @Override
    public void onError(Throwable throwable) {
        Log.d("TAG","Result: Failed to verify the number. Reason: " + throwable.getMessage());
    }
    @Override
    public void onComplete() {
    }
});
```
```js
AV.Cloud.verifySmsCode('123456', '+19490008888').then(function(){
    // Successfully verified
}, function(err){
    // Failed to verified
});
```
```cs
AVCloud.VerifySmsCodeAsync("123456","+19490008888").ContinueWith(t =>{
    if(t.Result) 
    {
        // Successfully verified
    }
});
```
```php
SMS::verifySmsCode('+19490008888', '123456');
```
```python
from leancloud import cloud
cloud.verify_sms_code('+19490008888', '123456')
```

The method verifies if the code entered is correct.

## Templates

If you anticipate the text messages sent out from your app to follow fixed formats, you can store these formats as templates on the cloud so that when calling LeanCloud's API to send messages, you only need to plug in the variables used in these templates rather than to pass in the entire text message as a string.

{{ sms.signature("### Signatures") }}

### Creating Templates

To create a template, go to your app's dashboard and navigate to [Messaging > SMS > Settings](https://console.leancloud.app/messaging.html?appid={{appid}}#/message/sms/conf).

### Using Templates

Assuming that you have created a template named `Order_Notice` with the following content:

{% call docs.bubbleWrap() -%}
Your package for the order {{ docs.mustache("{order_id}") }} will be delivered by the end of today.
{% endcall %}

Also assuming that you already have a signature named `sign_BuyBuyBuy` with its content being "BuyBuyBuy". Now you're good to use the following method to send a text message with the template:

```objc
AVShortMessageRequestOptions *options = [[AVShortMessageRequestOptions alloc] init];

options.templateName = @"Order_Notice";
options.signatureName = @"sign_BuyBuyBuy";
options.templateVariables = @{ @"order_id": @"7623432424540" }; // Plug the actual value into the template

[AVSMS requestShortMessageForPhoneNumber:@"+19490008888"
                                 options:options
                                callback:^(BOOL succeeded, NSError * _Nullable error) {
                                    if (succeeded) {
                                        /* Request completed */
                                    } else {
                                        /* An error occurred */
                                    }
                                }];
```
```swift
let variables: LCDictionary = [
    "order_id": LCString("7623432424540")
]

_ = LCSMSClient.requestShortMessage(
    mobilePhoneNumber: "+19490008888",
    templateName: "Order_Notice",
    signatureName: "sign_BuyBuyBuy",
    variables: variables)
{ (result) in
    switch result {
    case .success:
        break
    case .failure(error: let error):
        print(error)
    }
}
```
```java
AVSMSOption option = new AVSMSOption();
option.setTemplateName("Order_Notice");
option.setSignatureName("sign_BuyBuyBuy");
Map<String, Object> parameters = new HashMap<String, Object>();
parameters.put("order_id", "7623432424540"); // Plug the actual value into the template
option.setEnvMap(parameters);
AVSMS.requestSMSCodeInBackground("+19490008888", option).subscribe(new Observer<AVNull>() {
    @Override
    public void onSubscribe(Disposable disposable) {
    }
    @Override
    public void onNext(AVNull avNull) {
        Log.d("TAG","Result: Successfully sent text message.");
    }
    @Override
    public void onError(Throwable throwable) {
        Log.d("TAG","Result: Failed to send text message. Reason: " + throwable.getMessage());
    }
    @Override
    public void onComplete() {
    }
});
```
```javascript
AV.Cloud.requestSmsCode({
mobilePhoneNumber: '+19490008888',
template: 'Order_Notice',
sign:'sign_BuyBuyBuy',
order_id: '7623432424540'}).then(function(){
      // Request completed
    }, function(err){
      // An error occurred
});
```
```cs
var env = new Dictionary<string,object>()
{
    {"order_id","7623432424540"} // Plug the actual value into the template
};
AVCloud.RequestSMSCodeAsync("+19490008888","Order_Notice",env,"sign_BuyBuyBuy").ContinueWith(t =>
{
    var result = t.Result;
    // result being True means the request is completed
});
```
```php
$options = [
  "template" => "Order_Notice",
  "name" => "sign_BuyBuyBuy",
  "order_id" => "7623432424540", // Plug the actual value into the template
];
SMS::requestSmsCode("+19490008888", $options);
```
```python
from leancloud import cloud
options = {
  "order_id": "7623432424540" # Plug the actual value into the template
}
cloud.request_sms_code("+19490008888",
  template="Order_Notice", sign="sign_BuyBuyBuy", params=options)
```

The actual text message received by the user would look like this:

{% call docs.bubbleWrap() -%}
【BuyBuyBuy】Your package for the order 7623432424540 will be delivered by the end of today.
{% endcall %}

### Template Variables

A template can contain **custom variables** which get passed in as parameters when you are calling API to send text messages. The syntax used is [Handlebars](https://handlebarsjs.com).

{# {{ docs.alert("自定义变量的值不允许包含实心括号 `【】`。") }} #}

You can also include **system variables** in your templates which get automatically filled when sending messages. You **cannot** override these values when calling API:

{% call docs.bubbleWrap() -%}
Welcome to {{ docs.mustache("{name}") }}! Your verification code is {{ docs.mustache("{code}") }}. Please complete signing up in {{ docs.mustache("{ttl}") }} minutes.
{% endcall %}

- `name`: The name of the app
- `code`: Verification code
- `ttl`: The number of minutes the verification code is valid for (defaults to 10 and can be up to 30)
- `sign`: Signature
- `template`: Template name

## Preventing Abuse with CAPTCHA

While SMS allows developers to offer a better experience to their customers, it could also get abused by people with bad intentions, which not only causes financial loss but also brings a negative impact to the reputation of their apps. Hackers may gather a list of websites that allow users to trigger text messages without going through CAPTCHA and use programs to automatically send text messages using these sites. Here are some possible consequences:

- To website owners, their sites will continuously receive requests to send out text messages and they will have to pay more for that;
- To users of phone numbers, they will receive tons of irrelevant text messages containing verification codes.

To prevent such abuse, CAPTCHA can be used to filter out requests that are not made by humans. You might have seen something like this in many other websites which asks you to enter the text appearing in an image:

<img src="images/captcha.png" width="400" alt="A page requiring user to type in the characters in an image.">

To prevent hackers from abusing your app, you can require each user to enter the characters in an image before they can request an SMS verification code for free using LeanCloud's service.

Here is a basic workflow of CAPTCHA:

```seq
User (browser)->App server: 1. Request for sign-up or log-in page
App server->User (browser): 2. Show sign-up or log-in page
User (browser)->LeanCloud: 3. Request CAPTCHA
LeanCloud->User (browser): 4. Show CAPTCHA
User (browser)->LeanCloud: 5. Request to verify user input
LeanCloud->User (browser): 6. Return token
User (browser)->LeanCloud: 7. Request to send verification code (with token attached)
LeanCloud->User (browser): 8. Return result or send verification code
User (browser)->App server: 9. Submit form to sign up or log in
App server->LeanCloud: 10. Verify the code
LeanCloud->App server: 11. Return result of verification
App server->User (browser): 12. Return result of operation
```

1. The user opens the sign-up or log-in page and request for CAPTCHA from LeanCloud.
2. The user fills in their personal information, including the text in the CAPTCHA image, and clicks on a button to verify the input.
3. After LeanCloud verifies that the input is correct, it sends a verification code to the number.
4. The user submits the form and the app sends the verification code entered by the user to LeanCloud for further verification.
5. LeanCloud returns if the verification code is correct or not. If it is correct, the user completes signing up or logging in.

### Enabling CAPTCHA

To enable CAPTCHA for SMS verification codes, go to your app's [Dashboard > Settings > Security](https://console.leancloud.app/app.html?appid={{appid}}#/security) and turn on **CAPTCHA**.

If you wish CAPTCHA to be enabled for all types of text messages sent out from your app, go to [Dashboard > Messaging > SMS > Settings > SMS Settings](https://console.leancloud.app/messaging.html?appid={{appid}}#/message/sms/conf) and enable **Require CAPTCHA before sending messages**. Once enabled, any request for sending a text message without going through CAPTCHA will receive an error.

LeanCloud's CAPTCHA service can only provide a minimum level of abuse prevention. You are free to add third-party CAPTCHA services to your website.

### An Example of Using CAPTCHA on a Website

Now we will build a basic page with HTML and JavaScript to demonstrate how you can use our CAPTCHA service on your website.

#### Initialization

Initialize CAPTCHA component:

```js
AV.Captcha.request().then(function (captcha) {
  captcha.bind({
    textInput: 'captcha-code', // The id for textInput
    image: 'captcha-image',    // The id for image element
    verifyButton: 'verify'     // The id for verify button
  }, {
    success: function (validateCode) {
      console.log('The input is correct.');
    },
    error: function (error) {
      console.error(error.message);
    }
  });
});
```

#### Parameters

The following parameters can be passed in when initializing an `AV.Captcha` instance with `AV.Captcha.request()`:

{{ sms.paramsRequestCaptcha() }}

For example, to initialize a `captcha` instance with height being 30px and width being 80px:

```js
AV.Captcha.request({ width: 80, height: 30 });
```

`captcha.bind()` binds the `captcha` instance with the element on the user interface. It supports the following parameters:

| Name | Type | Description |
| --- | --- | --- |
| `textInput` | `string` or `HTMLInputElement` | The input box for entering text, or its `id`. |
| `image` | `string` or `HTMLImageElement` | The image component for displaying CAPTCHA image, or its `id`. |
| `verifyButton` | `string` or `HTMLElement` | The button for proceeding, or its `id`. |

#### Full Code

Here is the full code for our demo:

```html
<!DOCTYPE html>
<html>

<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width">
  <title>Testing CAPTCHA</title>
</head>

<body>
  <label>Phone Number
    <input type="text" id="phone" />
  </label>
  <br />
  <label>Verification Code
    <input type="text" id="captcha-code" />
  </label>
  <img id="captcha-image" />
  <br />
  <button id="verify">Send Code</button>
  <!-- Include LeanCloud SDK -->
  <script src="//cdn.jsdelivr.net/npm/leancloud-storage@{{jssdkversion}}/dist/av-min.js"></script>
  <script>
    var appId = '{{appid}}';  // Your appId
    var appKey = '{{appkey}}'; // Your appKey
    AV.init({ appId, appKey });
    // AV.Captcha.request() generates an 85px x 30px AV.Captcha instance by default
    AV.Captcha.request().then(function (captcha) {
      // Bind CAPTCHA with DOM elements using captcha.bind
      captcha.bind({
        textInput: 'captcha-code', // The id for textInput
        image: 'captcha-image',    // The id for image element
        verifyButton: 'verify'     // The id for verify button
      }, {
        success: function (validateCode) {
          var phoneNumber = document.getElementById('phone').value;
          console.log('The input is correct. Now send verification code to: ' + phoneNumber);
          AV.Cloud.requestSmsCode({
            mobilePhoneNumber: phoneNumber,
            name: 'App Name',
            validate_token: validateCode,
            op: 'some operation',
            ttl: 10
          }).then(function () {
            console.log('Message sent.');
          }, function (err) {
            console.error('Failed to send message.', err.message);
          });
        },
        error: function (error) {
          console.error(error.message);
        }
      });
    });
  </script>
</body>

</html>
```

### CAPTCHA APIs

#### Getting Images

```objc
AVCaptchaRequestOptions *options = [[AVCaptchaRequestOptions alloc] init];

options.width = 100;
options.height = 50;

[AVCaptcha requestCaptchaWithOptions:options
                            callback:^(AVCaptchaDigest * _Nullable captchaDigest, NSError * _Nullable error) {
                                /* URL string of captcha image. */
                                NSString *url = captchaDigest.URLString;
                            }];
```
```swift
LCCaptchaClient.requestCaptcha(width: 100, height: 50) { (result) in
    switch result {
    case .success(value: let captcha):
        if let url = captcha.url {
            print(url)
        }
        if let token = captcha.token {
            print(token)
        }
    case .failure(error: let error):
        print(error)
    }
}
```
```java
AVCaptchaOption option = new AVCaptchaOption();
option.setWidth(85);
option.setHeight(30);
AVCaptcha.requestCaptchaInBackground(option).subscribe(new Observer<AVCaptchaDigest>() {
    @Override
    public void onSubscribe(Disposable d) {
    }
    @Override
    public void onNext(AVCaptchaDigest avCaptchaDigest) {
        Log.d("TAG","The URL of the image is: " + avCaptchaDigest.getCaptchaUrl());
    }
    @Override
    public void onError(Throwable throwable) {
        Log.d("TAG","Result: Failed to request CAPTCHA image. Reason: " + throwable.getMessage());
    }
    @Override
    public void onComplete() {
    }
});
```
```javascript
AV.Captcha.request({
  width:100, // Width of the image
  height:50, // Height of the image
}).then(function(captcha) {
  console.log(captcha.url); // The URL of the image used to display the image
});
```
```cs
AVCloud.RequestCaptchaAsync(width:85, height:30).ContinueWith(t =>{
  var captchaData = t.Result;
  var url = captchaData.Url; // The URL of the image used to display the image
  var captchaToken = captchaData.captchaToken; // The server tells the CAPTCHA being used based on this token
});
```
```php
// PHP SDK does not support CAPTCHA at this time
```
```python
from leancloud import cloud
captcha = cloud.request_captcha(width=100, height=50)
```

#### Verifying User Inputs

After obtaining a CAPTCHA, you can display the image on your page (for non-web platforms like iOS and Android, use an image component to display the image). After the user enters the text into the input box, use the method below to verify the input:

```objc
[AVCaptcha verifyCaptchaCode:<#User identifier#>
            forCaptchaDigest:<#The AVCaptchaDigest object requested earlier#>
                    callback:^(NSString * _Nullable validationToken, NSError * _Nullable error) {
                        /* validationToken can be used when sending SMS verification code */
                    }];
```
```swift
LCCaptchaClient.verifyCaptcha(code: "code", captchaToken: "captcha.token") { (result) in
    switch result {
    case .success(value: let verification):
        if let token = verification.token {
            print(token)
        }
    case .failure(error: let error):
        print(error)
    }
}
```
```java
AVCaptcha.verifyCaptchaCodeInBackground("123456",avCaptchaDigest).subscribe(new Observer<AVCaptchaValidateResult>() {
    @Override
    public void onSubscribe(Disposable d) {
    }
    @Override
    public void onNext(AVCaptchaValidateResult avCaptchaValidateResult) {
        Log.d("TAG","Result: Verification completed.");
    }
    @Override
    public void onError(Throwable throwable) {
        Log.d("TAG","Result: Verification failed. Reason: " + throwable.getMessage());
    }
    @Override
    public void onComplete() {
    }
});
```
```javascript
// captcha is the CAPTCHA instance obtained earlier
captcha.verify('Plug user input here, for example: AM8N').then(function(validateToken) {});
```
```cs
AVCloud.VerifyCaptchaAsync("Plug user input here, for example: AM8N",'Plug captchaToken here').CotinuteWith(t =>{
    var validate_token = result;
});
```
```php
// PHP SDK does not support CAPTCHA at this time
```
```python
# captcha is the CAPTCHA instance obtained earlier
validate_token = captcha.verify("Plug user input here, for example: AM8N")
```

#### Sending SMS Verification Codes with `validate_token`

If the input is correct, continue to send SMS verification code with the `validate_token` obtained:

```objc
AVShortMessageRequestOptions *options = [[AVShortMessageRequestOptions alloc] init];
options.templateName = @"New_Series";
options.signatureName = @"sign_BuyBuyBuy";
options.validationToken = <#validationToken#>;

[AVSMS requestShortMessageForPhoneNumber:@"+19490008888"
                                 options:options
                                callback:^(BOOL succeeded, NSError * _Nullable error) {
                                    if (succeeded) {
                                        /* Request completed */
                                    } else {
                                        /* An error occurred */
                                    }
                                }];
```
```swift
_ = LCSMSClient.requestShortMessage(
    mobilePhoneNumber: "+19490008888",
    templateName: "New_Series",
    signatureName: "sign_BuyBuyBuy",
    captchaVerificationToken: "captcha_verification_token")
{ (result) in
    switch result {
    case .success:
        break
    case .failure(error: let error):
        print(error)
    }
}
```
```java
AVSMSOption option = new AVSMSOption();
option.setTemplateName("Order_Notice");
option.setSignatureName("sign_BuyBuyBuy");
option.setCaptchaValidateToken("validateToken");
AVSMS.requestSMSCodeInBackground("+19490008888", option).subscribe(new Observer<AVNull>() {
    @Override
    public void onSubscribe(Disposable disposable) {
    }
    @Override
    public void onNext(AVNull avNull) {
        Log.d("TAG","Result: Successfully sent verification code.");
    }
    @Override
    public void onError(Throwable throwable) {
        Log.d("TAG","Result: Failed to send verification code. Reason: " + throwable.getMessage());
    }
    @Override
    public void onComplete() {
    }
});
```
```javascript
// mobilePhoneNumber: Phone number
// template: Template name
// sign: Signature
AV.Cloud.requestSmsCode({
    mobilePhoneNumber: '+19490008888',
    template: 'New_Series',
    sign:'sign_BuyBuyBuy'
}，{
    validateToken:'The validate_token obtained earlier'
}).then(function(){
    // Request completed
}, function(err){
    // An error occurred
});
```
```cs
// +19490008888: Phone number
// New_Series: Template name
// sign_BuyBuyBuy: Signature
AVCloud.RequestSMSCodeAsync("+19490008888","New_Series",null,"sign_BuyBuyBuy","The validate_token obtained earlier").ContinueWith(t =>
{
    var result = t.Result;
    // result being True means the request is completed
});
```
```php
// PHP SDK does not support CAPTCHA at this time
```
```python
from leancloud import cloud
options = { "validate_token": validate_token }
cloud.request_sms_code("+19490008888",
  template="New_Series", sign="sign_BuyBuyBuy", params=options)
```

## International Text Messages

To send text messages to international users, simply add the country code at the beginning of the number. For example, `+1` is for the US or Canada. Make sure to turn on **Allow international numbers** in your app's [SMS Settings](https://console.leancloud.app/messaging.html?appid={{appid}}#/message/sms/conf). If no country code is provided, `+86` (China) will be used by default.

For a list of countries and regions that LeanCloud can reach out through SMS, please refer to the [Pricing](https://leancloud.app/pricing/) page on our website.

## Integrating with LeanCloud User System

LeanCloud offers a built-in [user system](leanstorage_guide-js.html#users) for you to quickly implement features that allow users to sign up, log in, and reset passwords. With the help of SMS, these operations can be done with users' phone numbers as well.

You can find the following settings in your app's [Dashboard > LeanStorage > Settings > Account](https://console.leancloud.app/storage.html?appid={{appid}}#/storage/conf):

{{ include.checkbox(true) }}**Send verification SMS when users register or change phone numbers from clients**
- Enabled: If you pass in a phone number when calling the API to create an `AVUser`, a text message with verification code will be automatically sent. The `mobilePhoneVerified` of the `_User` will be set to `true` once the verification is completed.
- Disabled: No text message will be sent when creating an `AVUser`.

{{ include.checkbox() }}**Do not allow users with unverified phone numbers to log in**
- Enabled: An `AVUser` with unverified number cannot log in using **phone number and password** or **phone number and verification code**. This user can still log in with **username and password**.
- Disabled: Whether a user's phone number is verified will not affect the methods this user can use to log in.

{{ include.checkbox() }}**Allow users with unverified phone numbers to reset passwords with SMS** 
- Enabled: Allow an `AVUser` with unverified number to reset their password with SMS verification code.
- Disabled: An `AVUser` must have their phone number verified (`mobilePhoneVerified` being `true`) before they can reset their password with SMS verification code.

{{ include.checkbox() }}**Allow users with verified phone numbers to login with SMS**
- Enabled: An `AVUser` can log in with **phone number and verification code**.
- Disabled: An `AVUser` cannot log in with **phone number and verification code**.

You can learn how to [sign up](leanstorage_guide-js.html#signing-up-with-phones) and [log in](leanstorage_guide-js.html#logging-in-with-phones) with phone numbers as well as how to verify [existing users' phone numbers](leanstorage_guide-js.html#verifying-phone-numbers) in LeanStorage Guides.

{#
### 短信计费

如果一个短信模板字数超过 70 个字（包括签名的字数），那么该短信在发送时会被运营商<u>按多条来收取费用</u>，但接收者收到的仍是<u>一条完整的短信</u>。

- 小于或等于 70 个字，按一条计费。
- 中英文标点算作一个字符。
- 超过 70 个字符则按照 67 个字符来计算条数，最长可发 400 字。
- 最长的 400 字符的短信收费计算公式为：400/67 = 5.9， 也就是要扣 6 条短信费用。

只有「调用失败」不收费，「投递失败」也要收费。每条短信的收费标准请参考 [官网价格方案](https://leancloud.app/pricing/)。

### 短信购买

短信发送采取实时扣费。如果当前账户没有足够的余额，短信将无法发送。充值请进入 [开发者账户 > 财务 > 财务概况](https://console.leancloud.app/bill.html#/bill/general)，点击 **余额充值**。

同时我们建议设定好 [余额告警](https://console.leancloud.app/settings.html#/setting/alert)，以便在第一时间收到短信或邮件获知余额不足。
#}
