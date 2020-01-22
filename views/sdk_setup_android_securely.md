{% from "views/_data.njk" import libVersion as version %}

# Use Android SDK without appKey

Starting from 6.1.0, LeanCloud Android SDK supports initialization without appKey
if you wish to avoid exposing appKey on the client side. 

## Prerequisites

- You have already [signed your APK with a certificate][sign-your-app].
- You have JDK installed in your development environment.

[sign-your-app]: https://developer.android.com/studio/publish/app-signing

To generate the certificate fingerprint,
invoke the following command:

```sh
keytool -list -v -keystore /path/to/keystore/file
```

Then fill in the sha256 fingerprint and your Android package name in LeanCloud Dashboard (Settings > Security).

## Android SDK Initialization

### Install SDK

Please refer to [SDK Setup Guide](start.html).

### JNI Native library

To initialize SDK without appKey, you need to use LeanCloud native library.

Download [leancloud-jniLibs.zip] and extract it,
then copy the `jniLibs` directory to `src/main`:

[leancloud-jniLibs.zip]: http://lc-lhzo7z96.cn-n1.lcfile.com/84af049f980dd5e2d4c8/leancloud-jniLibs.zip 

```
src/
└── main
    ├── AndroidManifest.xml
    ├── assets
    ├── java
    ├── jniLibs
    │   ├── arm64-v8a
    │   │   └── libleancloud-core.so
    │   ├── armeabi
    │   │   └── libleancloud-core.so
    │   ├── armeabi-v7a
    │   │   └── libleancloud-core.so
    │   ├── mips
    │   │   └── libleancloud-core.so
    │   ├── mips64
    │   │   └── libleancloud-core.so
    │   ├── x86
    │   │   └── libleancloud-core.so
    │   └── x86_64
    │       └── libleancloud-core.so
    └── res
```

To reduce the APK size, you can remove directories containing architects that you are not using.

### Initialization 

Modify `build.gradle` to support auto signing:

```groovy
android {
    compileSdkVersion 29
    buildToolsVersion "29.0.2"
    defaultConfig {
        applicationId "xxxx"
        minSdkVersion 21
        targetSdkVersion 29
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }
    // auto sign
    signingConfigs {
        config {
            keyAlias '{your key alias}'
            keyPassword '{your key password}'
            storeFile file('{your store file full name}')
            storePassword '{your store password}'
        }
    }
    buildTypes {
        debug {
            // configuration for signing
            signingConfig signingConfigs.config
        }
        release {
            // configuration for signing
            signingConfig signingConfigs.config
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
}
```

Then add the following initialization code into the `onCreate` method of the `Application` class:

```java
import cn.leancloud.AVOSCloud;

public class MyLeanCloudApp extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        AVOSCloud.initializeSecurely(this, "{{appid}}", "https://xxx.example.com");
    }
}
```

Congratulations!
You have just configured your application to initialize LeanCloud SDK without appKey.
Now you can start developing your application as usual.

## LeanEngine Runtime SDK

If your Android application will invoke [Cloud Function](leanengine_cloudfunction_guide-node.html),
you need to make sure the LeanEngine runtime SDK you use supports this feature.

Currently, the following LeanEngine runtime SDK versions support android initialization without appKey:

- Python SDK: 2.3.0 and later
- Node.js SDK: 3.5.0 and later
- Java SDK (engine-core): 6.1.0 and later

## Afterword

This new way of Android SDK initialization avoids exposing appKey on the client side.
To fully ensure data security, you still need to utilize ACL to restrict data access permission.

## FAQ

#### Does Android SDK 6.1.0 still support the old way?

Yes.
If you prefer the old way, you can still use it,
and you do not need to add the native library to your project.

#### Why the application crashes after switching to the new way?

This is probably because you did not add the native library to your project.
Please refer to the section [JNI Native Library](#jni-native-library) above.

#### Why all requests return `{"code":401,"error":"Unauthorized."}` error?

This is probably because your APK did not have signing properly configured.
Please refer to the section [Initialization](#initialization) above.

#### Will switching to the new way affect old versions of my Android application?

No.

#### What if I have multiple Android applications sharing one LeanCloud application as their backend?

Currently we do not support configuring multiple pairs of package names and fingerprints for one LeanCloud application.
Therefore, if you have multiple Android applications connected to one LeanCloud application as their backend,
only one Android application can utilize the new way.
