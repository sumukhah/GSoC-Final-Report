# Rocket chat sponsored GSoC project
mentors: Rodrigo Nascimento\
student: Sumukha Hegde \
\
**Project tasks:**

1.  Implementing **Recording** and **Uploading** activity indicators on mobile as well as in the web application.
2.  Fixing **thread statuses** to show only on the thread on mobile as well as in the web application.

**Work done**

**Recording, Uploading indicators**
I have implemented recording, uploading activity indicators on mobile and web applications. A new stream activity type called **user-activity** is created, which handles all three activities (typing, recording, uploading). In case we need to add some additional activity indicators, this work will become an easier task.

**Thread status**
The previous version of our app displays all thread activities in the main channel rather than the performing thread. This will be fixed by the project allowing to display all thread activities only on the thread, and this fix is available both on mobile and web applications.

<img src="https://user-images.githubusercontent.com/23723464/129456739-3bd10fa6-baaf-4241-9935-2346adf67a76.png" width="500px">

### Code implementation

> step1: The client subscribes to the opened channel and it will have reading and writing permission for all user-activities performing on that channel.

On pressing the audio/video record button, or uploading a file, or Typing on messagebox __On performer side__
> step2:  We brodcast that event with some neccessary parameters such as
rid: Room Id, actionType: 'user-typing' || 'user-recording' || 'user-uploading', user: UserName, isPerforming: True | False, extras: {tmid: Thread Id(if applicable)}

For compatibility purpose, we fire either 'user-action'(compatible with only mobile and web client) or 'typing'(compatible with all clients) based on admin's requirement(option inside General -> user-activity indicator -> fire old typing event as well). \
We have used a timer and delete this user action(typing|recording|uploading) in 15 seconds and stored it in `activityTimeouts`. After this period we call the stop Broadcasting function, so Within this timeout period, we have to call the broadcast function again to show this event continuously on the channel/thread. But we cannot overload the server by calling so many times. For this reason, we have used a `activityRenews` variable and we will store the activity for 5 seconds(renew time). If another request comes within this period, we can simply ignore(simply return). After 5 second(renew time) the activity will be deleted from `activityRenews`, and we call brodcast function again and we store the activity on activityRenews and activityTimeout.
> step3: On completion of user event we call broadcast event and pass isPerforming = False along with other parameters.

__On receiver Side__
> step1: client subscribes to the channel and will have the permission to read and write all user-actions

When a user starts an action in that channel/thread
> step2: `handleStreamAction` will be called again and again and we store this performing user on an object performing user. We also use a timer here, we delete this performing user from the object at 15 seconds(activity timeout), so within this time handleStreamAction should be called again by the broadcaster to show the activity continuously.

Then we display these performing users on top of the message box. This `handleStreamAction` can handle both old and new actionTypes (i.e., 'typing' and 'user-action') and able to show it in similar way.


**Compatibility**
Since my project introduced a new activity type (user-activity) that would handle all three activities, compatibility with the other clients would not be possible. So we still need to support the older version of typing activity until all other client gets updated to use the new stream activity. To achieve compatibility we have added an option in the admin setting ( general->activity indicators -> fire old typing event) and if the user allowed, it would fire the old stream event ('typing') for user typing action and make it compatible with all other clients(such as the live chat, app engine) as well.

[![Demo video](https://user-images.githubusercontent.com/23723464/130472569-4ecefb95-6ca3-4964-8e18-8185e711bb37.png)](https://youtu.be/NMmr-WNDZEY)

Special Thanks to:
* Rocket.chat Community for giving me an extra opportunity to work with their amazing team through Sponsored GSoC project and community members for their help.
* My awesome mentor Rodrigo Nascimento (CTO & Co founder of Rocket.chat)
* Fellow GSoCers

**Links to my work**
* [Web integration](https://github.com/RocketChat/Rocket.Chat/pull/22392/files)
* [Mobile integration](https://github.com/RocketChat/Rocket.Chat.ReactNative/pull/3243)
* [Sdk improvement for mobile integration](https://github.com/RocketChat/Rocket.Chat.js.SDK/pull/139)

**Works remaining**
* Testing
* Apps engine integration
* Live chat integration
