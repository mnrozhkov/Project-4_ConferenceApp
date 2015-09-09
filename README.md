2015.09.01
Athor: Mikhail Rozhkov

This is a Project 4 - App Engine application for the Udacity training course.

App info:
	application: conf-app-playground
	version: 2
	runtime: python27
	api_version: 1
	


A. Configuration Instruction
---------------------------- 
App structure: 
|-conf-app-playground		- app folder
	|-static/				- contains style files and templated
		|-bootstrap/		- Bootstrap3 and CSS files
		|-fonts/			- font files
		|-js/				- JavaSctipt files
		|-img/    			- images
		|-styles.css		- main CSS 
		|-partials			- some html files 
	|-templates/			- index.html template
	|-app.yaml				- specifies how URL paths correspond to request handlers and static files
	|-conference.py |.pyc	- conference server-side Python App Engine API
	|-cron.yaml				- configuration for regularly scheduled tasks 
	|-index.yaml 			- index definitions
	|-LINCESE 				- Apache License
	|-main.py | .pyc 		- conference server-side Python App Engine HTTP controller handlers 
	|-models.py | .pyc 		- conference server-side Python App Engine data & ProtoRPC models
	|-README.txt			- this file
	|-settings.py | .pyc 	- conference server-side Python App Engine app user settings
	|-utils.py | .pyc 		- help utilities 

Requrements:
This App was developed with Google App Engine SDK for Python. It used: 
	Python 2.7
	Google App Engine



B. Installation Instructions 
----------------------------
1. Update the value of `application` in `app.yaml` to the app ID you
   have registered in the App Engine admin console and would like to use to host
   your instance of this sample.
1. Update the values at the top of `settings.py` to
   reflect the respective client IDs you have registered in the
   [Developer Console][4].
1. Update the value of CLIENT_ID in `static/js/app.js` to the Web client ID
1. (Optional) Mark the configuration files as unchanged as follows:
   `$ git update-index --assume-unchanged app.yaml settings.py static/js/app.js`
1. Run the app with the devserver using `dev_appserver.py DIR`, and ensure it's running by visiting
   your local server's address (by default [localhost:8080][5].)
1. Generate your client library(ies) with [the endpoints tool][6].
1. Deploy your application.

[1]: https://developers.google.com/appengine
[2]: http://python.org
[3]: https://developers.google.com/appengine/docs/python/endpoints/
[4]: https://console.developers.google.com/
[5]: https://localhost:8080/
[6]: https://developers.google.com/appengine/docs/python/endpoints/endpoints_tool



C. Copyright and licensing information
-----------------------------------
This code was developed based on Udacity code samples from lessons:
	"Developing Scalable Apps with Python"
Code samples are considered as "Educational Content" and follows the terms of the Creative Commons 
Attribution-NonCommercial-NoDerivs 3.0 License (http://creativecommons.org/licenses/by-nc-nd/3.0/ 
and successor locations for such license) (the "CC License")
For more info: https://www.udacity.com/legal/ 



D. Tasks for Project4
---------------------------- 

#Task 1: Add Sessions to a Conference
Question: Explain your design choices. Explain in a couple of paragraphs your design choices for session and speaker implementation

Answer:
	1) Session class is implemented as a child entity of the Conference. 
		s_key = ndb.Key(Session, new_id, parent=conf.key)
	For storing data  about Session class, the following data types were used:
	- StringProperty: for 'name', 'highlights', 'speaker', 'typeOfSession', 'venue' and 'topics' properties.
	These properties describe data about entities using text string.
	- DateProperty is required by 'date' property to have opportunity to order Sessions by date,
	align dates of Session with dates of Conferences. DateProperty also allows to easy implement other
	Google API services, like Google Calendar API (for example, to get user an opportunity to add Session
	to his calendar).
	- IntegerProperty is applied for 'timeStart', 'timeEnd' and 'duration' properties. For this case, the decision was made
	as a consensus between required functionality, time and my knoweldge of Python and Google Engine SDK.
	Initially, I've tried to implement TimeProperty, however, I've got a problem: data input in format
	HH:MM was transformed into '1970-01-01 HH:MM:00' by default. So, I had some problems with Session time: formatting,
	extracting HH:MM from '1970-01-01 HH:MM:00', and it was not comparable with time of other entities
	such as conferece with date '2015-08-30 00:00:00' (for example).
	Thus, I've decided to simplify and use simple IntegerProperty for this. It's much easier to work with this format
	of time: it is short, it can be compared with timeEnd easier, or to calculate duration of the Session.
	(Actually, 'duration' was not calculated in this task, left for further development). So, this is enough
	for the current project tasks.
	
	2) Speaker is implemented as a plain name string for simplicity and time saving.
	For further development, it can be stored as a single class. This will allow to store some
	additional info about a speaker (picture, contacts) and make the app more functional.

	3) all required methods are implemented as well
		getConferenceSessions(websafeConferenceKey) 
		etConferenceSessionsByType(websafeConferenceKey, typeOfSession) 
 		getSessionsBySpeaker(speaker) 
 		createSession(SessionForm, websafeConferenceKey)



# Task 2: Add Sessions to User Wishlist
	addSessionToWishlist(SessionKey) is implemented 
	getSessionsInWishlist() is implemented



#Task 3: Work on indexes and queries
Question: Think about other types of queries that would be useful for this application
Answer: 
	getConferenceSessionsByDate(websafeConferenceKey, date, date) - return session for a specific conference on a specific date. In case when conference lasts few days, this option may be helpful

	getConferenceSessionsByTime(websafeConferenceKey, timeStart, timeEnd) - helps to find session in specified time interval. This help to plan a part time participating in a confernce (e.g. from 9:00am tp 12:00pm)  

Question: Solve the following query related problem
	Let’s say that you don't like workshops and you don't like sessions after 7 pm. 
	How would you handle a query for all non­workshop sessions before 7 pm? 
	What is the problem for implementing this query? What ways to solve it did you
	think of?

Answer: 
	Google datastore has some limitaitons to queries. For example, "combining too many filters, using inequalities for multiple properties, or combining an inequality with a sort order on a different property are all currently disallowed" (Google, NDB Queries documentation: https://cloud.google.com/appengine/docs/python/ndb/queries)   
	In this case we requires two inequality filters on two different properties (type of session and start time of session). This will cause a BadRequestError.

	Step 1:  Inequality 'property != value' can be implemented as '(property < value) OR (property > value)'
		sessions = Session.query(ndb.OR(Session.typeOfSession < 'WORKSHOP',
										Session.typeOfSession > 'WORKSHOP')
		This finds all entities with at least one tag unequal to 'WORKSHOP'. 
	
	Step 2: To find all non­workshop sessions before 7 pm, we can use 'property IN [value1, value2, ...]' tests for membership in a list of possible values, is implemented a '(property == value1) OR (property == value2) OR ...'
		sessions = sessions.filter(Session.timeStart <= 1900)

	The special filter method was implemented to query such kind of queries:

	    def filterSessionNotTypeByTime(self, request):
	        """Return filtered Sessions which are != type of Session and
	        != timeStart
	        """

	        type = str(request.typeOfSession)
	        sessions_noType = Session.query(ndb.OR(Session.typeOfSession < str(type),
											Session.typeOfSession > str(type)))
	        sessions = [sess for sess in sessions_noType if sess.timeStart <= request.timeStart]

	        return SessionForms(items=[self._copySessionToForm(sess)
	                                   for sess in sessions])


# Task 4: Add a Task
	getFeaturedSpeaker() method to store a featured speaker per conference is implemented
	[websafeConferenceKey] was used as the memcache key