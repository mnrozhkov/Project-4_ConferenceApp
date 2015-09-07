Comments 3 and revision 


Comments for  conference.py
Comment:
	ndb.Key(Conference, request.websafeConferenceKey)
	becomes
	ndb.Key(urlsafe=request.websafeConferenceKey)
Answer: 
	Done!
	

Comment: 
	Because of the change we made to session key generation we must now modify how we check for sessions in the wishlist:
	conference_key = ndb.Key(Conference, request.websafeConferenceKey)
	becomes
	conference_key = ndb.Key(urlsafe=request.websafeConferenceKey)
	This changes the key we retrieve from:
	Key('Conference', 'ahdkZXZ-Y29uZi1hcHAtcGxheWdyb3VuZHIxCxIHUHJvZmlsZSIUamFtZXNjbTMyMUBnbWFpbC5jb20MCxIKQ29uZmVyZW5jZRgBDA')
	to
	Key('Profile', email@gmail.com', 'Conference', 1)
	Which will match our session parent key (example):
	Key('Profile', 'email@gmail.com', 'Conference', 1)
Answer: 
	Changes to 'conference_key' retrieval done. However, it doesn't seem that changes are required to check for sessions in the wishlist. 
	Of course, it's not possible to match new 'conference_key' with old 'conference_keys' stored in a wishlist. But for new items in wishlist it works the same. 
	

Comment: 
	Currently this endpoint does not find any sessions because it is searching with an incorrect ancestor key, so memcache.set is never called.
	conf_key = ndb.Key(Conference, websafeConferenceKey)
	becomes
	conf_key = ndb.Key(urlsafe=websafeConferenceKey)
Answer:
	Fixed


In addition:
	Key retrieval code 'ndb.Key(urlsafe=websafeConferenceKey)' was also applied to the following methods: 
		getConferenceSessions
		getConferenceSessionsByType
		getConferenceSessionsBySpeaker