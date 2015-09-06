Comments 2 

Comments for  models.py
Comment:
	class SessionForm(messages.Message):

	BadValueError: Expected integer

	Session model requires an IntegerProperty for duration, the SessionForm asks for a StringField. You can either update StringField to IntegerField or coerce the StringField into an integer before calling Session(**data).put() in the_createSessionObject method.

Answer: 
	Fixed. 
	Thank you to note this. 
	1) coerced the StringField into an integer before calling Session(**data).put() in the_createSessionObject method
	Sample of code:
		if data['duration']:
            data['duration'] = int(data['duration']

	2) coerced an integer to the StringField before copying data for SessionForm in the _copySessionToForm method
	Sample of code:
		elif field.name.endswith('duration'):
                    sf.timeEnd = str(sess.timeEnd)



Comments for  conference.py
Comment:
	Currently the way session keys are created causes some problems.
	conf_key = ndb.Key(Conference, request.websafeConferenceKey)
	becomes
	conf = ndb.Key(urlsafe=request.websafeConferenceKey).get()
	conf_key returns: Key('Conference', 'ahdkZXZ-Y29uZi1hcHAtcGxheWdyb3VuZHIxCxIHUHJvZmlsZSIUamFtZXNjbTMyMUBnbWFpbC5jb20MCxIKQ29uZmVyZW5jZRgBDA')
	conf.key returns: Key('Profile', 'account@email.com', 'Conference', 1) which is the full key including all ancestors.
Answer: 
	This is a basis of App Engine, and I did this wrong... ))) 
	

Comment: 
	new_id = ndb.Model.allocate_ids(size=1, parent=conf_key)[0]
	s_key = ndb.Key(Session, new_id, parent=conf_key)
	becomes
	new_id = Session.allocate_ids(size=1, parent=conf.key)[0]
	s_key = ndb.Key(Session, new_id, parent=conf.key)
Answer: 
	Thank you, fixed.
	

Comment: 
	IndexError: list index out of range 
	if prof.sessionKeysInWishlist[0] is None:
	becomes
	if prof.sessionKeysInWishlist is None:
Answer:
	Fixed
