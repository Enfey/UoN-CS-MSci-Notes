Hello i'll be talking about continental, a finite state machine dialogue system for restauraunt reservations. 
The system implements 21 dialogue states, ML-based intent classification using calibrated SVM, and TF-IDF retrieval for QA. 
I'll demonstrate its core features while showing corresponding code.

Scene 1: New User Booking flow
[System Greets] -> Main calls driver.greet(), which then calls select_template("greeting") - select_template uses random to provide conversational variability for the greeting set. Greet provides discoverability too, explaining available actions.

User: I'd like to book a table for tomorrow at 7pm for 4  Notice there is multi-slot extraction after matching on the booking intent via the nlu layer, our input fills 3 slots via [Navigate to extract_all_entities at line 320-390]. These entity extraction functions are extremely robust, and handle a variety of formats e.g., will happily parse 24 hour time, full dates, dates without the year attached, etc. The FSM in advance_booking_flow() checks each slot sequentially – None values trigger the corresponding prompt. Implicit confirmations maintain progressivity throughout.

User: Felix Riley-Kay - Just like before, simply filling the slot in accordance with the multi-part name portion of the name extraction function, and populating our transient state dictionary

User: No [First Time]
User: 07700900456
User: Gluten-free
User: Yes [Explicit confirmation] - mirror back key transactional information at thresholds to guide users.
User: Yes [Explicit confirmation] - creates profile ensuring the user has consented to this. 
User: I'd like to book for Sunday
→ Shows: "Sorry, we're closed on Sundays. Our next available day is Monday 30th December." [Navigate to generate_date_error_message in entities.py] if the driver does not provide a suggestion, we generate one using the original date as the reference, effecting both repair and progression. 


Scene 2: Repair/Recovery and Mid-flow QA
User: Book a table for Friday at 6pm - seen this before
User: Actually, change the time to 8pm - going to change the time, we invoke detect_slot_correction [Go to detect_slot_correction/handle_booking_slot] to detect repair intent, which matches keywords like "change", then determine which slot to update based on both keyword mentions and extracted entities, and then only that slot is overwritten. This is all equipped with validation. 
User: Do you have parking? Mid-flow QA is handled by check_high_priority_intents()[GOTO] – if should_treat_as_qa()[GOTO] which returns true based on either a heuristic or  trained query-intent classifier exceeding 50% confidence, the system retrieves answers via TF-IDF similarity then returns to the booking flow. This is disabled during special requests as during testing, this caused confusion with dietary queries. QA is rather complex, with a three-tier confidence fallback strategy including synoym expansion, and narrowing of query intent via a secondary classifier with lowered confidence.
User: 2 people - and as we can see, the flow continues uninterrupted without issues.


Scene 3: Rebook usual and verification - Create profule first
User: Rebook my usual - starts the rebooking flow via intent matching; generally matched perfectly as the rebooking keywords are lexically distinct from other classes.
User: [name from profile] - After name entry, handle_verify_identity requests last 4 digits or email. store.verify_user() [GOTO] matches on contact and name. 
User: 0456 [last 4 digits]
→ Shows previous bookings On success, get_last_n_bookings() retrieves history and dietary preference. One of these can then be repeated, pre-filling slots. 
User: 1 [select]
User: Yes

TRY SHOW GRACEFUL DEGRADATION IF GET CHANCE

Scene 4: Cancellation flow
User: Cancel my reservation [NAVIGATE TO CHECK_HIGH_PRIORITY_INTENTS], as we can see cancellation is permitted from most states except booking confirmation, as we don't want cancel to be perceived as cancel a booking generally, but rather cancelling the in-progress booking. Cancellation-flow is started with intent matching, transitioning the state.
User: [reference ID e.g., 07FDF9923166] - [NAVIGATE TO handle_cancel_provide_info] - this is triggered when providing information in cancellation flow, and accepts either a reference ID, contact, or list selection. We demonstrate the reference cancellation here. Unverified users must confirm booking contact details as we've seen prior. (07700900456)
User: Yes [Explicit confirmation] - A destructive action once more, via prompting, requires the user confirms before transactional go-ahead.


Scene 5: Help and Stop.
User: Help
→ [Shows capabilities] - [NAVIGATE TO is_help_request()] - help is either explicit, or intent matched at 85% threshold, providing discoverability at any transactional point as this is a high-priority check, and this clearly enumerates all system capabilities, focusing on recognition over recall. 
User: Book a table for tomorrow
User: Stop
User: Yes - Once more explicitly confirms, setting 'awaiting_stop_confirm' to prevent accidental data loss, only resetting slots after confirmation. 
→ "Alright, I've stopped what I was doing."


Continental demonstrates key conversational user interface principles: discoverability through help prompts, recovery via slot corrections and informative
errors, and progressivity through implicit and explicit confirmations. The FSM architecture ensures that dialogue is predictable, and the use of machine-learning
based intent classification via multinomial logistic regression (or a calibrated support vector machine, depending on application changes) handles linguistic variation. 