## DataReceipt.js 
A helper library to create FDS accounts, create Consent Receipts JWT tokens, send them over Swarm, decode and verify tokens with additional layer to support **Consent Manager** smart contract and consent signing and verification of **Consent** smart contract on blockchain. 


 DataReceipt.js library uses fds.js library to send consent receipt files over Swarm to another account
 ** BEWARE **
 uses FDS.js multibox branch https://github.com/fairDataSociety/fds.js/tree/multibox 
 
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
 
## fds.js settings 
you need to provide FDS object in window.FDS property for fds.js and datareceipt.js to work with Noordung blockchain 
View network: https://noordung.fairdatasociety.org/  
Block explorer: https://blockscout.noordung.fairdatasociety.org/

```
window.FDS = new FDS({
    swarmGateway: 'https://swarm.fairdatasociety.org',
    ethGateway: 'https://geth-noordung.fairdatasociety.org',
    faucetAddress: 'https://dfaucet-testnet-prod.herokuapp.com/gimmie',
    httpTimeout: 1000,
    gasPrice: 0.1,
    ensConfig: {
        domain: 'datafund.eth',
        registryAddress: '0xc11f4427a0261e5ca508c982e747851e29c48e83',
        fifsRegistrarContractAddress: '0x01591702cb0c1d03b15355b2fab5e6483b6db9a7',
        resolverContractAddress: '0xf70816e998819443d5506f129ef1fa9f9c6ff5a7'
    },
    // multibox extension
    applicationDomain: "/shared/consents/"
}); 
```

##### First steps
Initialize library
`let fd = new DataReceiptLib();`

Create account (will fail if it exists)
`let newAccount  = await fd.createAccount(accountName, password);`

Unlock account
`let account     = await fd.unlockAccount(accountName, password);`

Load Private Key
`let loadPrivKey = await fd.loadPrivateKey(privateKey);`

Load existing project
`let loadProject = await fd.loadProject(project);`

Generate token
`let signedToken = await fd.generateToken();`

Send token to accountName 
`let swarmHash   = await fd.sendDataReceipt(signedToken, accountName);`

##### How to use Consent Manager and Consent smart contracts
This part uses blockchain addresses directly.

Get account address `let userAddress    = fd.account.address;`

Get address of receiver subject `let subjectAddress = await fd.account.getAddressOf(subjectName);`

Get consent manager contract `let CM = await fd.getConsentManager();`

Create consent contract `let tx = await CM.createConsent(userAddress, subjectAddress, "0x" + swarmHash);`

Get array of existing consent contract addresses where account is user `let uc = await CM.getUserConsents();`

Get array of existing consent contract addresses where account is subject `let sc = await CM.getSubjectConsents();`

Get consent contract for for swarmHash `let cf = await CM.getConsentsFor("0x" + swarmHash);`


##### Consent Contract Interface class
Get consent contract with address
`let consent = await fd.getConsent(consentContractAddress);`

Consent contract interface class allows:
 - status query
 - signing for subject and user
 
For consent to be valid, both parties must sign it. To check if its signed by user
`let us = await consent.isUserSigned();` 
or if subject signed it 
`let ss = await consent.isSubjectSigned();`

Alternatively you user or subject can sign directly using 
`await consent.signUser();` or `await consent.signSubject();`

Check consent status meanings
   - 0 - waiting for signatures
   - 1 - active 
   - 2 - expired
   - 3 - revoked
  
`let status = await consent.status();`

##### Updating existing consents
Existing consents can be updated with new consents. Old consent is invalidated and references new consent
`let tx = await CM.updateConsent(prevConsentAddress, "0x" + swarmHash);`

To get updated consent contract address use:
`let isUpdated = await consent.isUpdatedWith();` 
In order to get latest consent, recursevly interate until address returned is 0x0. 




### Sample usage
`let fd = new DataReceiptLib();`
`let password = 'test';`
`let accountName = 'testAccountName123123';`
##### Account must exist
`let subjectName = 'testAccountName1231234';` 
##### Create account (will fail if it exists)
`let newAccount  = await fd.createAccount(accountName, password);` 
##### get account
`let account     = await fd.unlockAccount(accountName, password);` 
##### Load private key
`let loadPrivKey = await fd.loadPrivateKey(privateKey);` 
##### Load Consent Receipt Project
`let loadSuccess = await fd.loadProject(project);` 
##### Sign token in project
`let signedToken = await fd.generateToken();`
##### Send file ower Swarm and retrieve its Swarm hash
`let swarmHash   = await fd.sendDataReceipt(signedToken, subjectName);`  where is cr.jwt stored?

### How to send from newAccount to newAccount using blockchain 
##### Get account address
`let userAddress    = fd.account.address;`
##### Retrieve subjects address, this looks subdomain name in ENS
`let subjectAddress = await fd.account.getAddressOf(subjectName);`

##### get consent manager
`let CM = await fd.getConsentManager();` 
##### create consent for swarm hash
`let tx = await CM.createConsent(userAddress, subjectAddress, "0x" + swarmHash);`
once transaction is finished

##### get existing consents where account is DataUser
`let uc = await CM.getUserConsents();`

##### get existing consents where account is DataSubject
`let sc = await CM.getSubjectConsents();`

##### get consents for swarmHash 
`let cf = await CM.getConsentsFor("0x" + swarmHash);` 

#### How to check signatures

##### Sign last consent as user 
`let consent = await fd.getConsent(uc[uc.length - 1]);` 
##### Get data location
`let location = consent.swarmHash;`
##### check if its signed by user
`let us = await consent.isUserSigned();`
##### Signing last consent as user
`await consent.signUser();`

`let ss = await consent.isSubjectSigned();`
##### Signing  consent as subject
`await consent.signSubject();`

##### Update existing consent with new consent and data at new swarm location
`let tx = await CM.updateConsent(prevConsentAddress, "0x" + swarmHash);`

     
##### display and check all user consents 
`let uc = await CM.getUserConsents();`
##### Iterate through consents
`await fd.asyncForEach(uc, async (consentAddress) => { ... }` 

`let consent = await fd.getConsent(consentAddress);`

##### Check is user signed consent
`let us = await consent.isUserSigned();`
##### Check if subject signed consent
`let ss = await consent.isSubjectSigned();`
##### Check if both parties signed
`let s = await consent.isSigned();`
##### Check if consent is still valid
`letv = await consent.isValid();`

##### Consent update trigger previous consent to be revoked
if updated anything else than 0x0000000000000000000000000000000000000000
then consent was updated with another consent.

`let updated = await consent.isUpdatedWith();` 

##### Get consent status
Return values 
         0 - waiting for signatures
         1 - active 
         2 - expired
         3 - revoked 
`status = await consent.status();` 

##### get all received messages 
`let messages = await fd.getReceivedMessages(true);` 
