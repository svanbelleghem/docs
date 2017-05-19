---
section: libraries
toc_title: Passwordless Authentication with Lock for Android
description: Guide on implementing Passwordless authentication with Lock for Android
---

# Implementing Lock Passwordless

`PasswordlessLockActivity` authenticates users by sending them an Email or SMS (similar to how WhatsApp authenticates you). In order to be able to authenticate the user, your application must have the Email/SMS connection enabled and configured in your [Auth0 dashboard](${manage_url}/#/connections/passwordless).

The first step is to add the `PasswordlessLockActivity` to your `AndroidManifest.xml` inside the `application` tag. In case the Passwordless mode is **LINK** instead of CODE you will need to add the corresponding **Intent-Filters** to capture the result. You can skip them otherwise.

```xml
<activity
    android:name="com.auth0.android.lock.PasswordlessLockActivity"
    android:theme="@style/Lock.Theme"
    android:label="@string/app_name"
    android:screenOrientation="portrait"
    android:launchMode="singleTask">
    <intent-filter>
        <action android:name="android.intent.action.VIEW"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <category android:name="android.intent.category.BROWSABLE"/>
        <data
          android:host="${account.namespace}"
          android:pathPrefix="/android/{YOUR_APP_PACKAGE_NAME}/email"
          android:scheme="https" />
    </intent-filter>
    <intent-filter>
        <action android:name="android.intent.action.VIEW"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <category android:name="android.intent.category.BROWSABLE"/>
        <data
          android:host="${account.namespace}"
          android:pathPrefix="/android/{YOUR_APP_PACKAGE_NAME}/sms"
          android:scheme="https" />
    </intent-filter>
</activity>
```

Replace `{YOUR_APP_PACKAGE_NAME}` with your actual application's package name. Make sure the Activity's `launchMode` is declared as `"singleTask"` or the result won't come back after the authentication.

Notice in case we only use one Passwordless method (either SMS or Email) you can delete the other Intent-Filter. (see the last segment of the pathPrefix: `/email` or `/sms`).

Next, if the Passwordless connection is SMS you must add the `CountryCodeActivity` to allow the user to change the **Country Code** prefix of the phone number.

```xml
<activity
    android:name="com.auth0.android.lock.CountryCodeActivity"
    android:theme="@style/Lock.Theme.ActionBar" />
```

The last change on the AndroidManifest is adding the *Internet* permission to your application:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

Then in any of your activities, you need to initialize **PasswordlessLock**

```java
public class MainActivity extends Activity {

  private PasswordlessLock lock;

  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    // Your own Activity code
    Auth0 account = new Auth0("${account.clientId}", "${account.namespace}");
    auth0.setOIDCConformant(true);
    lock = PasswordlessLock.newBuilder(auth0, callback)
      //Customize Lock
      .build(this);
  }

  @Override
  protected void onDestroy() {
    super.onDestroy();
    // Your own Activity code
    lock.onDestroy(this);
    lock = null;
  }

  private LockCallback callback = new AuthenticationCallback() {
     @Override
     public void onAuthentication(Credentials credentials) {
        //Authenticated
     }

     @Override
     public void onCanceled() {
        //User pressed back
     }

     @Override
     public void onError(LockException error) {
        //Exception occurred
     }
  };
}
```

Finally, just start `PasswordlessLock` from inside your activity and perform the login.

```java
startActivity(lock.newIntent(this));
```

Depending on the chosen Passwordless Mode an Email/SMS with a Code/Link will be delivered to the user as the first step in the flow. The user must enter the Code in the prompt or click the Link in order to the second step be called.
