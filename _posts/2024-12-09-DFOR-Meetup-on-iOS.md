## Mobile Forensics - Analyzing data stored by Meetup Application on iOS device

The aim of this blog is to understand data stored by Meetup and research on how and what kind of information is stored on a user's mobile.  

### App Description 

Meetup is a Social Events and Groups app that aggregates events and activities to build communities, connections and try new hobbies. Users can connect with people in their area (or online for activities that do not require physical presence) that share their interests and track the events that they plan to attend or find interesting. The app allows users to join groups and offers messaging and photo sharing capabilities to coordinate.    

### Testing Conditions 

#### Application Details 

Name: Meetup 
Version: v.2024.11.14 (632295) 
We logged in as a test user with the name Beaufort and added interests.  
![Screenshot 2024-12-08 063540](https://github.com/user-attachments/assets/2ead24a9-56c2-44cb-b97d-2e202d6d405c)
Using another account, we created a test event located in downtown Portland. The user joined this event and interacted with the group by texting, adding comments, sending pictures and adding this event to the calendar. 
![Screenshot 2024-12-08 064103](https://github.com/user-attachments/assets/164900bf-7b58-479f-ba99-5087887b72cd)


We logged in as a test user with the name Beaufort and added interests.  
 ```tsql
 SELECT *
 FROM sys.tables
 WHERE [name] = 'SomeTable'
 ```
