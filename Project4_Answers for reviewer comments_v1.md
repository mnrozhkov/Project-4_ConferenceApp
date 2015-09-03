Comments regardign README.md 

#Task 1: Add Sessions to a Conference
Comment:
	Great job explaining your high level design decisions for Sessions and Speaker.

	But, I would like to see a little more detail on your actual implementation of the Session class. For example, there are some non-obvious decisions to be made regarding the data type used to store information like startTime and endTime. A couple sentences explaining your reasoning behind these decisions will make sure that you clearly " shows understanding of the process of data modeling and justifies their implementation decisions for the chosen data types." - to borrow from the grading rubric.

Answer: 
	1) Session class is implemented as single class, not as a child of a Conference class. This desision is 
	made for for simplicity.
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

	

#Task 3: Work on indexes and queries
Comment: 
	However, both the solutions you provided aren't able to get around this problem (note that both the typeOfSession and timeStart filters are inequalities). If I run the first code snippet this is the error that is logged: "BadRequestError: Only one inequality filter per query is supported. Encountered both typeOfSession and timeStart."

	Try and come up with a way around this problem - there are many ways to do it using the Datastore, python code, or both. If you're feeling adventurous, implement your solution to Exceed Specifications.

Answer: 
	Step 1:  Inequality 'property != value' can be implemented as '(property < value) OR (property > value)'
		sessions = Session.query(ndb.OR(Session.typeOfSession < 'WORKSHOP',
										Session.typeOfSession > 'WORKSHOP')
		It finds all entities with at least one tag unequal to 'WORKSHOP'. 
	
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



Comments regardign conference.py
Comment 1:
	While this way of copying data between objects certainly works and is not difficult to understand, a much better and more robust way to accomplish the same thing is to loop over the fields. One reason why is if you decided to add or rename a field. Say, change 'venue' to location. If you were looping over the data all you'd need to change is the variable names in the Session and SessionForm class definitions and the copy function would continue to work without any additional (and tedious variable renaming).
	You can see Udacity's implementation of this in the copyConferenceToForm and copyProfileToForm functions.
Answer 1:
	This section of code was implement using the loop: 

    def _copySessionToForm(self, sess):
        """Copy relevant fields from Session to SessionForm."""
        
        sf = SessionForm()
        for field in sf.all_fields():
            print("field: ", field)
            if hasattr(sess, field.name):
                if field.name == 'date':
                    sf.date = str(sess.date)
                elif field.name.endswith('timeStart'):
                    sf.timeStart = str(sess.timeStart)
                elif field.name.endswith('timeEnd'):
                    sf.timeEnd = str(sess.timeEnd)
                elif field.name.endswith('typeOfSession'):
                    try:
                        setattr(sf, field.name, getattr(SessionType, getattr(sess, field.name)))
                    except AttributeError:
                        setattr(sf, field.name, getattr(SessionType, 'NOT_SPECIFIED'))
                else:
                    setattr(sf, field.name, getattr(sess, field.name))
        sf.websafeKey = sess.key.urlsafe()
        sf.check_initialized()

        return sf


Comment 2: 
	Look out, you're performing a check to see whether the newly created SessionForm not Session contains a date field - you'll want to do it the other way around.
Anser 2: 
	Thank you for poin it out. In revised code, this fragment check to see whether a passed Session 'sess' has an field.name
	'date' instead. 


Comment 3:
	In the models.py file, the format for timeStart and timeEnd is given as 'HH:MM'. If an API user sends their request following that format the function will crash because there is no way to convert '14:00' for example to an integer value. Please clarify your documentation so that users will be able to provide input that your endpoints expect.
Answer 3: 
	Documentation was clearofied. The format for timeStart and timeEnd was choosen as 'HHMM'



Comment 4:
	Great job testing for the presence of a date field in the request before performing a conversion, but your else branch is redundant. Also, it would improve the durability of your app if a similar check was performed before converting your timeStart and timeEnd fields. As it is, if they are not included in the request the endpoint will crash.
Answer 4: 
	The 'else branch' was removed. At the same time, it seems not necessary to do similar check for timeStart and timeEnd
	with new version of code. I've tested this endpoing with empty timeEnd and timeStart parameters in a request and it was
	working fine. 



Comment 5: 
	Awesome job implementing this feature with memcache (although I recommend you watch out for "Unknown" speakers being flagged as featured)!
	In order to Meet Specifications we need you to perform this task using taskqueue. Take a look at the implementation for the SendConfirmationEmailHandler and how that task is generated from conference.py (near the end of ~createConferenceObject) for some hints on how to accomplish this.
Answer 5: 
	Good pint! I've added a check for "Unknown" speaker. I've also implemented task using taskqueue.

		# set memcache for featured speaker and session
        if data['speaker'] and data['speaker'] != "Unknown":
            taskqueue.add(params={'websafeConferenceKey': request.websafeConferenceKey,
                                  'speaker': data['speaker']},
                          url='/tasks/set_featured_speaker')


Comment 6:
	Remember to update your docstrings when you are modifying functions! These are very important for your eventual users to understand your API. I'm sure you noticed, but docstrings are captured automatically by the API explorer:
Answer:
	Have done! 


Comment 7:
	You've designed this query well, but this line is actually preventing the entire thing from working. Play around with the logging functionality of app engine (details in the 2nd answer on how to enable it) and investigate the value your date variable takes. Then use the Datastore Viewer (App Engine admin page located on the admin port) to compare how the Datastore is storing the date variable of the Session objects. Those should point you in the right direction for solving this tricky bug!
Answer 7: 
	I have not found any tricky bug. Date is stored in date(datetime) format ('date: ', datetime.date(2015, 9, 4)). And, it works fine. 
	I would appreciate, if you can explain this point with more details. 


Comment 8:
	Looks like this is a snippet from something else as it isn't used in the function.
Answer 8:
	Deleted


Comment 9: 
	A minor point, but I think you'll find these query expressions easier to read if you add whitespace on either side of the inequality sign. It is also consistent with the PEP8 style guide https://www.python.org/dev/peps/pep-0008/#other-recommendations.
Answer 9: 
	Thanks. Whitespaces have been added. 


Comment 10:
	Careful with trying to access lists when it's possible the element doesn't exist. This if statement results in a 'list index out of range' exception and a crash because the sessionKeysInWishlist list is empty. Take a look at the code in getConferencesToAttend for some clues on how to implement this sort of lookup, and make sure to test and debug thoroughly. The logging window is your friend!
Answer 10: 
	Have no clues from the code  the code in getConferencesToAttend. Do not understand this point. 
 