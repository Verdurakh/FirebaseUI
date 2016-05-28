# FirebaseUI

Firebase UI

this tutorial is based on me following the documentation from https://github.com/firebase/FirebaseUI-Android/blob/master/auth/README.md but how I experienced it and for some reason the guide out of the box didnt work for me this also includes the result that I got working from all the googling that had to be done :)

Create a new android project
and add the following lines to your “dependencies” in build.gradle.

compile 'com.firebaseui:firebase-ui-auth:0.4.0'

To easy the process of getting google up and running

We will also need to add:

classpath 'com.google.gms:google-services:3.0.0'

And then add the 

apply plugin: 'com.google.gms.google-services'

at the bottom of the screen. If you try to sync now this will create an error like the following:

Error:Execution failed for task ':app:processDebugGoogleServices'.
> File google-services.json is missing. The Google Services Plugin cannot function without it. 
   Searched Location: 

This file we will get from the firebase console.

so head over to: https://console.firebase.google.com/ and lets create our app!

Click Create new App.


Give it a name and select country.

Then we need to add firebase to our android app.


And enter the package name and debug certificate sha-1.

You can find the package name in the manifest file, its the same that we gave our app.
for example com.example.firebaseui

The SHA-1 key is a bit different to get. You need to use the tool keytool that comes from your java install.
in the Command promt write something like this:
keytool -exportcert -list -v \
-alias androiddebugkey -keystore %USERPROFILE%\.android\debug.keystore
This is the default place where you should find your debug sha1 key. If you are like me the standard way of doing things is hardly ever working.
First I couldnt use keytool out of the box but I had to refer it to the folder where it is located.
Which for me is in :Program Files\Java\jre1.8.0_73\bin\keytool.exe
try searching for it if you cant find it.

Then after getting the tool ofcourse the search path %USERPROFILE%\.android\debug.keystore didnt work. Since I’m lazy I solved this by copying the file to for example c:/keytool/debug.keytool and refered it from there instead.
The tool will want a password to make something happen and the default password is “android”.

You will get something that looks like

and the SHA1 is the one you want.

So!
Now that we created the app we got a file called google-services.json.

Move this file into the folder as requested. do a small change to your gbuild.gradle and sync and now we should lose the pesky error.

Now firebase will ask us to add the plugin and sync, you can do that now if you have not already done so.

Click finish and our app is created! :)

Now we want to make sure to set up the sign in method in firebase console.

In the console, click Auth and then set up sign in method
Start by enabling the google provider for now.

Now back to Android studio and go to your main acitivty.
In the onCreate event we first need to check if we are aldready signed in.

FirebaseAuth auth = FirebaseAuth.getInstance();
   if (auth.getCurrentUser() != null) {
     // already signed in
   } else {
     // not signed in
   }
we are going to add the basic version with google provider if the user is not already signed in.
Add the sign in code as a constant in the class as:
private static final int RC_SIGN_IN = 9001;

then in the else clause you can add:
   // not signed in
   startActivityForResult(
          AuthUI.getInstance()
                  .createSignInIntentBuilder()
                  .setProviders(
   
                          AuthUI.GOOGLE_PROVIDER
   )
                  .build(),
          RC_SIGN_IN);

And then we need to add the method:

   protected void onActivityResult(int requestCode, int resultCode, Intent data) {
      super.onActivityResult(requestCode, resultCode, data);
      if (requestCode == RC_SIGN_IN) {
          if (resultCode == RESULT_OK) {
              // user is signed in!
              
              //finish();
          } else {
              // user is not signed in. Maybe just wait for the user to press
              // "sign in" again, or show a message
          }
      }
   }

And this is basically all that is needed to handle sign in with google!

Go ahead and build your app, compile it to your emulator or phone, I personally use a phone for development.
You should come to a screen that looks something like this:



Now to get this moving we also need a log out button.
Do do that we can use out activity_main and create a simple button:

   <Button
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:id="@+id/signOut"
      android:text="Sign Out!"/>
To Handle the buttonpresses we can easily make our whole activity an onclicklistener and handle everything at one place.
Therefore we need our activity to implement the interface. Add this to your public class declaration
implements View.OnClickListener
it should now look like:
   public class MainActivity extends AppCompatActivity implements View.OnClickListener  {
We also need to wire up the button to this class as an click listener so in the oncreate add the following:
   findViewById(R.id.signOut).setOnClickListener(this);

Now we can add the event that handles all clicks:
   @Override
   public void onClick(View v) {
      switch (v.getId()) {
          case R.id.signOut:
              signOut();
              break;
   
      }
   }

and finally we add an signOut method:


   private void signOut(){
      AuthUI.getInstance()
              .signOut(this)
              .addOnCompleteListener(new OnCompleteListener<Void>() {
                  @Override
                  public void onComplete(@NonNull Task<Void> task) {
                      if (task.isSuccessful()) {
                          //logged out
                          
                      } else {
                         //fail?
                      }
                  }
              });
   }














Now we can sign the user out of the app as well.

But after we signed out we would want to restart the log in flow so that we can try again?

We Just need to make one small change to make that happen easy.
In the onCreate method
We added code for signing in under the //not signed in.
we take that code and create a new method that contains the code needed to sign in. like:

   private void signIn(){
      startActivityForResult(
              AuthUI.getInstance()
                      .createSignInIntentBuilder()
                      .setProviders(
                              AuthUI.GOOGLE_PROVIDER)
                      .build(),
              RC_SIGN_IN);
   }

and then we call this method both from the onCreate
   // not signed in
   signIn();


Now when you sign in you should be promted to log back in again :)

Congratulations you now have a log in flow with firebase!

Now its time to add facebook to our sign in experience!



First we need to go https://developers.facebook.com/apps/  and create our app!

click new app and select android app Follow the instructions and add some random category and your email account.
Now you will be in a quick setup but since we are using firebase as login we do not need to do this so just click skip in the upper right corner.
On the next page we can find what we really need.
The AppiId
And App secret

Now go to the Firebase Consol, Auth and sign in method and click to enable Facebook.
Paste the App ID and App secret that you got eariler and copy the oAuth redirect url and go back to facebook.
here click Add Product in the left menu and select facebook Login.
In the Valid OAuth redirect URIs add the url that you got from firebase and save

Now we should be set. go back to Android studio and go to the sign in method.

Add the facebook Provider
   private void signIn(){
      startActivityForResult(
              AuthUI.getInstance()
                      .createSignInIntentBuilder()
                      .setTheme(R.style.GreenTheme)
                      .setProviders(
                              AuthUI.GOOGLE_PROVIDER,
                              AuthUI.FACEBOOK_PROVIDER)
                      .build(),
              RC_SIGN_IN);
   }


and run your app.

You should now also see and Sign in With Facebook button, you can now try to sign in with facebook.
what happend to me when I tried to log in with facebook is that I got stuck in a endless loading animation. It might be that this is due to facebook taking some time to be set up and I will try again later and see if it works.
I tried setting a break point in this method:
   protected void onActivityResult(int requestCode, int resultCode, Intent data) {
      super.onActivityResult(requestCode, resultCode, data);
      if (requestCode == RC_SIGN_IN) {

but I got no hit and no useful info from logcat. I’ll be back later when I have solved it.
Ok seems that I forgot one thing that was hidden in the documentation. To Use facebook login I need to create a string entry into the strings.xml like this:

   <string name="facebook_application_id" translatable="false">YOUR_APP_ID</string>

The app id that you use is the same as the one we put into firebase before from the facebook app page.
It is a bit strange that firebase ui doesnt supply any feedback on this and is just stuck at loading.

Ok try running the app again and select facebook.
And it is loading. we are making progress and next problem!
Atleast I got a message from facebook this time:
“One or more of the given URLs is not allowed by the app’s settings. To use this URL you must add a valid native platform in your app’s settings”

hmm ok I guess back to the facebook admin page

   keytool.exe -exportcert -alias androiddebugkey -keystore c:\keystore\debug.keystore -list -v | e:\bin\openssl.exe sha1 -binary | e:\bin\openssl.exe base64

   https://code.google.com/archive/p/openssl-for-windows/downloads

still didnt work, I guess that i got the wrong hash from the keytool, but this time the app gave me the hash that I should use in a message like “this hash is not okay blah blah”.
So i took the new hash and entered it in facebook admin page and now it is working :)



Problems:

The  guide on google wanted the sign out method to look like this:
   AuthUI.getInstance()
          .signOut(this)
          .addOnCompleteListener(new OnCompleteListener<AuthResult>() {
              public void onComplete(@NonNull Task<AuthResult> task) {
                  // user is now signed out
                  startActivity(new Intent(MainActivity.this, SignInActivity.class));
                  finish();
              });
          });

But this gave me an wierd error in the ide

But this was misleading and although the syntax was in error there was also problems with the new OnCompleteListener<AuthResult>
if we changed the code to the following it works:
   private void signOut(){
      AuthUI.getInstance()
              .signOut(this)
              .addOnCompleteListener(new OnCompleteListener<Void>() {
                  @Override
                  public void onComplete(@NonNull Task<Void> task) {
                      if (task.isSuccessful()) {
                          //logged out
                          
                      } else {
                         //fail?
                      }
                  }
              });
   }

