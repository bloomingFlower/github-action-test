# Functioning after a crash

Stateful: Since stateful protocols need to store data regarding the sessions, once the crash occurs, all the stored data is lost. Hence, it doesn’t work very well after a crash occurs.
Stateless: In the event of a crash, stateless protocols work better because there doesn’t exist a state that needs to be restored. A server that failed during the crash can simply be restarted.