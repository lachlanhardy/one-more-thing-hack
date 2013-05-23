# Windows Azure Mobile Services

### The Challenge

Yes, it's iOS on Windows Azure.

In this challenge, you will use Windows Azure Mobile Services to build a connected and sclable iOS app in Objective-C.  

You'll to the Windows Azure portal, create a new Mobile Service and SQL database.  Then, you'll download the sample ToDo list app and have some fun storing data in the cloud, configuring Facebook authentication, and sending push notifications.

The walkthrough below should help you with the challenge, but you can also get in touch with @lachlanhardy with any questions--he's on site and happy to help!

####Bonus Challenge
For bonus points--add a service from the Windows Azure Store to your app.

###Validation…and Prizes!

You can get these challenges validated by Lachlan until the end of the conference or send over a screenshot of your completed app to lachlanh@microsoft.com by EOD on 5/26/2013 to be entered into a drawing for a Jawbone Big Jambox. 

You'll receive one entry for completing the challenge; two if you also complete the bonus challenge.

### Challenge Walkthrough

####Data

* Login to your Windows Azure account at manage.windowsazure.com.  (If you don't have a Windows Azure account, you can obtain a free 90-day trial at http://www.windowsazure.com/en-us/pricing/free-trial/.)

* Click New --> Compute --> Mobile Service --> Create.  Then specify URL and database login/password order to create a new Mobile Service and the associated SQL database.

![image](https://raw.github.com/lachlanhardy/one-more-thing-hack/master/mobile-create.png)

* Select your new mobile service, choose iOS, and click 'Create a New iOS app.'

![image](https://raw.github.com/lachlanhardy/one-more-thing-hack/master/mobile-portal-quickstart-ios.png)

* Click 'Create ToDoItem Table' and Download the sample app.  This will automatically create a table called 'ToDoItem' in your app's SQL database and connect your sample app to that table.

![image](https://raw.github.com/lachlanhardy/one-more-thing-hack/master/mobile-quickstart-steps-ios.png)

* Open the sample app in Xcode and run the app.  In the simulator, you'll be able to add items to the Todo list.  Add a few items like 'Take Notes on Lex's Friedman's OMT session,' 'Make dinner plans for WWDC' and 'Pick up some Tim Tams.' When you hit the (+) button, you're sending a POST your app's Mobile Services backend hosted in Windows Azure.

![image](https://raw.github.com/lachlanhardy/one-more-thing-hack/master/mobile-entered-items.png)

* If you head back to the [Windows Azure Portal](manage.windowsazure.com), you'll see that items that you added to the list are now stored in the TodoItem table in your SQL database.

![image](https://raw.github.com/lachlanhardy/one-more-thing-hack/master/mobile-items-added.png)

* When you drill down into the table, you'll see the items you entered in a table with three columns.  Next, we're going to click the script tabe and copy in the following code snippet into the 'Insert' operation to see how dynamic schemas work in Mobile Services.

![image](https://raw.github.com/lachlanhardy/one-more-thing-hack/master/mobile-script-drilldown.png)

Scripts are how you add some custom logic to your app, connect to other Windows Azure services, or work with third party APIs.  With Mobile Services, all your server scripts need to be written in JavaScript.

```js
function insert(item, user, request) {

    item.created = new Date();

    request.execute();

}
```

* Head back to the simulator and add another item--like 'Testing Dynamic Schema' and hit the (+) button.  Now, if you refresh the table in the Windows Azure portal, you will see a fourth column added that details when the item was created.

![image](https://raw.github.com/lachlanhardy/one-more-thing-hack/master/mobile-date-created.png)

* Enabling dynamic schema is great when you want to create the schema for your table but you don't want to let all your users alter it in production.  To turn off dynamic schema, head to the configure tab and select 'OFF' for dynamic schema.

![image](https://raw.github.com/lachlanhardy/one-more-thing-hack/master/mobile-dynamic-schema-off.png)

* Here, we've walked through using a SQL database with your Mobile Service, but some apps need to store binary or unstructured data. Using scripts, you could easily connect to Windows Azure Blob or Table Storage (as well as many other third party data options).

####User Authentication

* The next step is to get set up with Facebook authentication and limit access to authenticated users.  

* In the Management Portal, click the Data tab, and then click the TodoItem table.

![image](https://raw.github.com/lachlanhardy/one-more-thing-hack/master/mobile-portal-data-tables.png)

* Click the Permissions tab, set all permissions to Only authenticated users, and then click Save. This will ensure that all operations against the TodoItem table require an authenticated user. This also simplifies the scripts in the next tutorial because they will not have to allow for the possibility of anonymous users.

![image](https://raw.github.com/lachlanhardy/one-more-thing-hack/master/mobile-portal-change-table-perms.png)

* Follow the steps on [this page](http://www.windowsazure.com/en-us/develop/mobile/how-to-guides/register-for-facebook-authentication/) to register your app for Facebook authentication with Mobile Services.

* Copy over your App Key and Secret from Facebook into the appropriate slots in the 'IDENTITY' tab. Hit save.

![image](https://raw.github.com/lachlanhardy/one-more-thing-hack/master/mobile-facebook-auth.png)

* Back in Xcode, open the project file QSTodoListViewController.m and in the viewDidLoad method, remove the following code that reloads the data into the table:

```
[self refresh];
```

* Right after the viewDidLoad method, add the following:

```
- (void)viewDidAppear:(BOOL)animated
{
    MSClient *client = self.todoService.client;


if (client.currentUser != nil) {
    return;
}


[client loginWithProvider:@"facebook" controller:self animated:YES completion:^(MSUser *user, NSError *error) {
    [self refresh];
}];


}
```

* Press the Run button to build the project, start the app in the iPhone emulator, then login with Facebook.

####Push Notifications

* To get started with push notifications, head to the 'PUSH' tab, upload your Apple Developer Certificate and hit 'Save.' (More details on getting an Apple Developer Certificate can be found [here](http://www.windowsazure.com/en-us/develop/mobile/tutorials/get-started-with-push-ios/).)

![image](https://raw.github.com/lachlanhardy/one-more-thing-hack/master/mobile-push-tab-ios.png)

Note that push notifications are not supported by the iOS simulator so you must deploy the application to a device.

* Once you have an Apple developer certificate, make sure Dynamic Schema is enabled so that you can add a column for device tokens.  After you've done that, open the QSAppDelegate.h file in Xcode and add the following property below the *window property:

```
@property (strong, nonatomic) NSString *deviceToken;
```

* In QSAppDelegate.m, replace the following handler method inside the implementation:

```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:
(NSDictionary *)launchOptions
{
    // Register for remote notifications
    [[UIApplication sharedApplication] registerForRemoteNotificationTypes:
    UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeSound];
    return YES;
}
```
* In QSAppDelegate.m, add the following handler method inside the implementation:

```
// We are registered, so now store the device token (as a string) on the AppDelegate instance
// taking care to remove the angle brackets first.
- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:
(NSData *)deviceToken {
    NSCharacterSet *angleBrackets = [NSCharacterSet characterSetWithCharactersInString:@"<>"];
    self.deviceToken = [[deviceToken description] stringByTrimmingCharactersInSet:angleBrackets];
}
```

* In QSAppDelegate.m, add the following handler method inside the implementation:

```
// Handle any failure to register. In this case we set the deviceToken to an empty
// string to prevent the insert from failing.
- (void)application:(UIApplication *)application didFailToRegisterForRemoteNotificationsWithError:
(NSError *)error {
    NSLog(@"Failed to register for remote notifications: %@", error);
    self.deviceToken = @"";
}
```

* In QSAppDelegate.m, add the following handler method inside the implementation:

```
// Because toast alerts don't work when the app is running, the app handles them.
// This uses the userInfo in the payload to display a UIAlertView.
- (void)application:(UIApplication *)application didReceiveRemoteNotification:
(NSDictionary *)userInfo {
    NSLog(@"%@", userInfo);
    UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Notification" message:
    [userInfo objectForKey:@"inAppMessage"] delegate:nil cancelButtonTitle:
    @"OK" otherButtonTitles:nil, nil];
    [alert show];
}
```

* In QSTodoListViewController.m, import the QSAppDelegate.h file so that you can use the delegate to obtain the device token:

```
#import "QSAppDelegate.h"
```
* In QSTodoListViewController.m, modify the (IBAction)onAdd action by locating the following line:

```
NSDictionary *item = @{ @"text" : itemText.text, @"complete" : @(NO) };
```
And then replace it with:

```
// Get a reference to the AppDelegate to easily retrieve the deviceToken
QSAppDelegate *delegate = [[UIApplication sharedApplication] delegate];


NSDictionary *item = @{
    @"text" : itemText.text,
    @"complete" : @(NO),
    // add the device token property to our todo item payload
    @"deviceToken" : delegate.deviceToken
};
```
This adds a reference to the QSAppDelegate to obtain the device token and then modifies the request payload to include that device token. You must add this code before to the call to the addItem method.

* Back in the management portal, click on the Data tab and then your TodoItem table.  Under the 'Insert' option on the Script tab, replace the prepopulated code with teh following:

```js
function insert(item, user, request) {
    request.execute();
    // Set timeout to delay the notification, to provide time for the 
    // app to be closed on the device to demonstrate toast notifications
    setTimeout(function() {
        push.apns.send(item.deviceToken, {
            alert: "Toast: " + item.text,
            payload: {
                inAppMessage: "Hey, a new item arrived: '" + item.text + "'"
            }
        });
    }, 2500);
}
```

This registers a new insert script, which uses the apns object to send a push notification (the inserted text) to the device provided in the insert request.  This script delays sending the notification to give you time to close the app to receive a toast notification.

#####Bonus Challenge #1 Walkthrough

* The Windows Azure Store contains services and data sets that can be useful in your app.

* Click New --> Store --> SendGrid (or the service of your choice) --> Free.

* Select ChosenSendGridName from the Dashboard and then hit 'Connection Info' and copy that information to your Notepad.

* Navigate to your Mobile service then click 'DATA' --> the 'ToDoItem' table --> 'SCRIPT.'

* On the 'Insert' script, replace the Insert function with the below code in order to trigger an email each time a new item is inserted to the 'ToDoItem' table.

```js
var SendGrid = require('sendgrid').SendGrid;


function insert(item, user, request) {    
    request.execute({
        success: function() {
            // After the record has been inserted, send the response immediately to the client
            request.respond();
            // Send the email in the background
            sendEmail(item);
        }
    });


function sendEmail(item) {
    var sendgrid = new SendGrid('**username**', '**password**');       

    sendgrid.send({
        to: '**email-address**',
        from: '**from-address**',
        subject: 'New to-do item',
        text: 'A new to-do was added: ' + item.text
    }, function(success, message) {
        // If the email failed to send, log it as an error so we can investigate
        if (!success) {
            console.error(message);
        }
    });
}
```


###Resources:

* iOS Getting Started Content: www.windowsazure.com/iOS
* Mobile Dev Center: www.windowsazure.com/mobile
* Collection of helpful blog posts & tutorials: aka.ms/CommonWAMS
* Forum: http://social.msdn.microsoft.com/Forums/en-US/azuremobile/threads
* Feature Requests: mobileservices.uservoice.com
* Feedback: mobileservices (at) microsoft (dot) com