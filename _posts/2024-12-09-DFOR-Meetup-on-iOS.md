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



 
 ```tsql
 SELECT *
 FROM sys.tables
 WHERE [name] = 'SomeTable'
 ```
