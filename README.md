## DataReceipt.js test implementation 
 DataReceipt.js library uses fds.js library to send consent receipt files over Swarm to another account
 
 1. create account  
 2. unlock account 
 3. load project file 
 4. load private key 
 5. generate cr.jwt token
 6. send cr.jwt token to accountname/yourself and wait for delivery 
 7. get all received files
 8. for each received file check if its cr.jwt token 
 9. decode token and verify 
 10. log out all messages that are cr.jwt tokens
  
