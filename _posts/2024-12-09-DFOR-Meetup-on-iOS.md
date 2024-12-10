## Mobile Forensics - Analyzing data stored by Meetup Application on iOS device

The aim of this blog is to understand data stored by Meetup and research on how and what kind of information is stored on a user's mobile.  

### App Description 

Meetup is a Social Events and Groups app that aggregates events and activities to build communities, connections and try new hobbies. Users can connect with people in their area (or online for activities that do not require physical presence) that share their interests and track the events that they plan to attend or find interesting. The app allows users to join groups and offers messaging and photo sharing capabilities to coordinate.    

### Testing Conditions 

#### Application Details 

Name: Meetup 
Version: v.2024.11.14 (632295) 
We logged in as a test user with the name Beaufort and added interests.  
<img src="https://github.com/user-attachments/assets/2ead24a9-56c2-44cb-b97d-2e202d6d405c" width="500" />

Using another account, we created a test event located in downtown Portland. The user joined this event and interacted with the group by texting, adding comments, sending pictures and adding this event to the calendar. 
<img src="https://github.com/user-attachments/assets/164900bf-7b58-479f-ba99-5087887b72cd" width="500" />

#### Device Details 

The device used for testing this app was iPhone 6s model MN1M2LL/A with iOS version 15.8.3. A full file system image of this mobile was acquired for the analysis. To achieve this, we used Cellebrite which ran the checkra1n jailbreak to gain privileged access to the phone to get all possible files. We took multiple images at different times to check the difference in data stored in cache.  

#### Software Used 

Due to the kind and amount of data in the iOS image, we used multiple software for our analysis. A link to all of these tools is mentioned in the reference section of the blog.  

| Software                  | Version                  | Purpose                           |
|:---------------------------:|:--------------------------:|:-----------------------------------:|
| Autopsy                   | 4.21.0                  | Analyzing image                  |
| iLeapp                    | 1.18.6                  | Analyzing image, Protobuf parsing|
| Cellebrite                | 7.65.0.247 (Educational)| Image collection                 |
| MacForensic Deserializer  | v1.5.1                  | Analyse plists                   |
| PList Editor              | 1.9.7                   | Analyse plists                   |
| DB Browser for SQLite     | 3.13.0                  | Open SQLite database files       |
| jsonformatter.org         |                          | Format JSON data                 |

### Analysis 

As per our analysis, we can see that the majority of the user-related data is stored only in 2 database files – cache and Apollo. Mainly, the portion accessed multiple times is retained in the phone's storage in the form of cache for quick access. This section will go into detail on where meetup stores its data on the device.  

#### Locations of data storage 

The following locations had the most useful data stored in it: 
```tsql
\filesystem1\private\var\mobile\Containers\Data\Application\<App ID>\Library 
```

This folder gave us the most amount of information as it included caches, user data and app metadata. All files needed for loading the application UI are in the following location of the image: 
```tsql
\filesystem1\private\var\containers\Bundle\Application\<App ID>\Meetup.app
```

This folder contains multiple .nib files. A nib file is a special type of resource file that you use to store the user interfaces of iOS and Mac apps – in short it is an Interface Builder document. We have not extensively analyzed this as we did not find any user data stored in these files. Apart from these, the folders contained information on plugins and bundles that meetup integrates with. For example, some of the bundles used here are Firebase, Google sign-in utilities, NYT Photo viewer and more. A complete list of the bundles we found in this version of meetup will be referenced in the Appendix along with an example of the bundle details stored. 

#### Application Information 

We found useful application data in multiple databases of the image. The iTunesMetadata plist file shows us details on the app installation itself. The only relevant information for us from this file is auto-download value which is set to false as this was a manually downloaded application and accountInfo which shows which apple ID downloaded this app. The location of this plist file is: 
```tsql
\filesystem1\private\var\containers\Bundle\Application\<App_Id>\iTunesMetadata
```
<img src="https://github.com/user-attachments/assets/3af7581b-09a5-42da-b00b-946ce41c32f1" width="500" />

We were also able to extract some useful information regarding the application usage from *DuetExpertCenter* database. DuetExpertCenter is an iOS Daemon which runs in the background of iOS devices. This daemon created an SQLite database called *_ATXDataStore* which we opened using DB Browser for SQLite. The path to find this is: 
```tsql
\filesystem1\private\var\mobile\Library\DuetExpertCenter\_ATXDataStore 
```
In this database, the table called appInfo had an entry for *com.meetup.iphone* which showed the following information relevant for investigation:  

**LastLaunchDate** value shows the last time the application was launched. The value is shown in Core Data timestamp which is the number of seconds (or nanoseconds) since midnight, January 1, 2001. To convert this, we can use a CoreDate online calculator which is referenced at the end of this blog.  

**InstallDate** value is also shown in timestamp format similar to lastLauchDate. This value shows when the application was first installed.  

The **subsequentLaunchCounts** and **subsequentAppActionLaunchCounts** provided blob data which we extracted as a separate file. The extracted data closely resembled plist data as showed in the following screenshot: 

<img src="https://github.com/user-attachments/assets/356f77f6-81fc-4234-996c-e781e2449248" width="300" />

To look at this data, we saved it, ran it through deserializer.exe and opened the output using plist viewer.  

During the testing we opened a few apps after using meetup, however the values the application is showing are not in whole numbers for which we currently do not have an explanation: 

<img src="https://github.com/user-attachments/assets/72a26b6d-7e6b-42eb-970d-1c2f6c0ab006" width="300" />

We also launched an action from the Meetup app to create a calendar entry for the upcoming event, which is reflected in the below subsequentAppActionLaunchCounts blob: 

<img src="https://github.com/user-attachments/assets/65ee045f-74af-4467-9e51-6a08033f8e84" width="300" />

*com.apple.mobilecal* is the bundle ID for calendar application and CreateEventIntent seems like a function to create a calendar entry although we did not find any documentation available for this.  

#### Messages 

We created messaging test data by interacting with the user that has created the test event. 
<img src="https://github.com/user-attachments/assets/485984c4-b83b-4231-b22c-80b39d9f1ca5" width="500" />

Initially, we were unable to locate any messaging data stored on the phone itself. We then opened the message view about 5 times and then acquired a new image. We were able to locate certain messaging information in the cache of this new image in the following database file:  
```tsql
 \filesystem1\private\var\mobile\Containers\Data\Application\<App_ID>\Library\Caches\com.meetup.iphone\cache.db  
```
We opened this database using DB browser for SQLite and navigated to a table called *cache_receiver_data*. One of the rows held text messages as json data as a value of the *key: conversations*. The json data was too large to be shown here, so we have only shown the parts of the information that is forensically useful for us.  

**Chat Metadata:**

The details of the chat between the users are under *data->conversations->edges* value of the json. Some of the useful fields are as follows: 

```tsql
"muted": false 
"hasUnreadMessages": false 
"archived": false 
"lastMessageDate": "2024-12-05T19:04:29-05:00" 
"created": "2024-11-24T23:36:05-05:00" 
```
**Message details:**

The message itself can be found under *data->conversations->edges->lastMessage->edges->node* value of the json 
```tsql
"updated": "2024-12-05T19:04:28-05:00" 
"text": "Hey hey! Message2" 
"read": false 
```
**User Details:**

The user that sent the message was mentioned in *data->conversations->edges->lastMessage->edges->node->member* and the other member can be found in *data->conversations->edges->lastMessage->edges->node->members*. User data include the names and locations. The json also tells us if the users are a part of the same group under the *commonGroups* key under members.  

The cache only provided us with the last message, however we were able to locate a few other messages using the notification details logged by the iPhone. The notification information can be found at: 
```tsql
filesystem1\private\var\mobile\Library\DuetExpertCenter\streams\userNotificationEvents\local
```
This was analyzed using iLeapp. As this is extracted from notification data, we can only see the messages sent to the user and not the messages sent by the user: 

<img src="https://github.com/user-attachments/assets/2a4fd2ee-613f-4d0d-af5b-0351cf61cefc" width="500" />

#### Groups and events 

The cache database might include information on the groups joined on the application. Similar to messaging data, the information is stored in json format under the value *self*. The json data was too large to be shown here, so we have extracted parts of the information as shown in this section. The groups joined by the user can be found under the value of *data->group and data-> stepUpInfo*. Some of the fields extracted are as follows: 
```tsql
"name": "DFOR 672 project test group", 
"eventsHosted": 1, 
"rsvps": 2, 
"members": 4 
```
The name of the organizer can be found under *data->latestOrganizer->name*

The cache file also included json dump of the event that was attended by the user. This data also resembles the group json as it is stored under the self key. The potential information that can be extracted from here includes: 
```tsql
"dateTime": "2024-11-25T15:00:00-08:00" 
"endTime": "2024-11-25T17:00:00-08:00" 
"eventType": "PHYSICAL" 
"title": "Testing 1" 
```
This json dump also gives us the name and interests of the organizer and the users attending the event.  

We were also able to find most of the event details in another SQLite database file called Apollo located at: 
```tsql
\filesystem1\private\var\mobile\Containers\Data\Application\<App_ID>\Library\Application Support\Apollo 
````
This database shows the key and its corresponding record of the details that are loaded into the application itself. We were able to correlate this information and understand it by using the API documentation provided by Meetup. The event details were stored here under the key called *QUERY_ROOT.event(id:<id_number>)*. The event related information was stored in this database as a json: 

<img src="https://github.com/user-attachments/assets/7f68d19f-f12b-4520-a6f9-159938acd0f5" width="500" />

The meetup event page is a central location for attendees to interact. Users can send comments and upload images. Some of these interactions are stored by Apollo database. We have created a simple python script that will extract the relevant user and event information from the Apollo database. The screenshot below shows the user sending an input of 6 on the main menu which is - Event Details. The script also has the capability to collect past notification prompts, user interests and more.  

<img src="https://github.com/user-attachments/assets/f742cd59-b407-4fed-a800-cffae81307a0" width="500" />

We extract information such as comments, users involved in the conversation, location details, event settings and a sample of the output parsed from the SQLite file and json data is shown below: 

<img src="https://github.com/user-attachments/assets/f4dccdd3-557f-4156-8aa2-800660e7b1e7" width="500" />

### Images 

We found certain images stored by meetup in the cache folders. In this section, we will analyze what these images might be and where we can find them. We found most of the images in the following directory: 
```tsql
\filesystem1\private\var\mobile\Containers\Data\Application\<App_ID>\Library\Caches\com.hackemist.SDImageCache\default 
```

<img src="https://github.com/user-attachments/assets/fb5b089b-cffc-4a4f-9f13-a46f2274d293" width="400" />

The images blurred out with a grey shape seemed like profile pictures of legitimate users on the application. Our test user has not visited any other profiles apart from the organizer of an event hosted by the other test profile (marked as 1). This leads us to believe that the images may be users who have either visited our test profile or share similar interests which may have led meetup to cache these profile pictures.  

The images marked with “1” is profile picture of the test profile that has created the event that my user has joined. We have also interacted with this user (by commenting and messaging on the app) which is why there might be multiple images of this user in our cache folder.  

The images marked with 2 seem like display pictures of events. For example, one of the images was as follows: 

<img src="https://github.com/user-attachments/assets/018e680f-1777-4ca5-a987-c4db8abbe6ab" width="400" />

This particular image seems like a display picture of a speed dating event. During our testing, my user did not click or follow this event, however, this may have been stored in cache by the application for recommendation or advertising purposes. In conclusion the cache stores images that our user may not have interacted with; however we did notice multiple images of the user that we have been interacting with.  

### References:

* Apple developer archives - .nib files: https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/NibFile.html
* Cocoa Core Data Timestamp Converter: https://www.epochconverter.com/coredata
* Duet for iOS and Mac OS X - A hands-on review: https://www.computerworld.com/article/1490190/hands-on-with-duet-for-ios-and-mac-os-x.html
* DB Browser for SQLite: https://sqlitebrowser.org/
* Plist Editor Pro: https://www.fatcatsoftware.com/plisteditpro/
* Apple Bundle Identifiers List: https://github.com/joeblau/apple-bundle-identifiers
* Meetup API: https://www.meetup.com/api/schema/#PayloadError
* Autppsy: https://www.autopsy.com/
* Deserializer: https://github.com/ydkhatri/MacForensics/tree/master/Deserializer
* Script for parsing Apollo: https://github.com/Arshiya-Jama/parseApolloDbMeetup/blob/main/parseApolloDB.py)
* Format JSON: jsonformatter.org
* Cellebrite: https://cellebrite.com/en/premium/
