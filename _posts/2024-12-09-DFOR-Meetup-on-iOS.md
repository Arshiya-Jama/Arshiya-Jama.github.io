## Mobile Forensics - Analyzing data stored by Meetup Application on iOS device

The aim of this blog is to understand data stored by Meetup and research on how and what kind of information is stored on a user's mobile.  

### App Description 

Meetup is a Social Events and Groups app that aggregates events and activities to build communities, connections and try new hobbies. Users can connect with people in their area (or online for activities that do not require physical presence) that share their interests and track the events that they plan to attend or find interesting. The app allows users to join groups and offers messaging and photo sharing capabilities to coordinate.    

### Testing Conditions 

#### Application Details 

Name: Meetup 
Version: v.2024.11.14 (632295) 



We logged in as a test user with the name Beaufort and added interests.  
 ```tsql
 SELECT *
 FROM sys.tables
 WHERE [name] = 'SomeTable'
 ```
