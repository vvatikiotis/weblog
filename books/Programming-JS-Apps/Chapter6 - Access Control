
###Authentication

####Passwords
Passwords: one-way encryption hash, 512 (or more) bit long.
Passwords are vulnerable to:

#####rainbow tables
Tables with stolen **precomputed** hashes. Very easy to crack up to 14 characters length passwords. 

Solution is *password salting*. *Don't use the same salt* for every password.


#####brute force
Solution is *key-stretching*, makes brute force impractical by increasing the time it takes to hash a password. This can be done by applying the hashing function in a loop.

Use the `bcrypt` or the PBKDF2 (HMAC-SHA1 type) hashing function.

#####variable time equality
Solution is to use a *constant time* hash equality check. So return after a *constant time* regardless of how soon the hashing function has taken.

#####passwords stolen from 3rd parties
We use the same passwords for different services.

#####npm packages

- `credential`
- `passport`
- `passport-local`

####Multifactor auth
Requires the user to present auth proof from two or more auth factors: the *knowledge* factor, something the user knows (a password), the `possession` factor, something the user has (a dongle, an app) and the *inheritance* factor, something the user is (fingerprint, etc).

#####Possession factor
Node package `speakeasy` implements Google Authenticator OTP (One Time Password). This returns a number. This number this should match the number from the *Google Authenticator* mobile application.

#####Inheritance factor
Geofencing: Checks whether the user tries to login from a known location. There is a Geolocation HTML API.

####Federated and Delegated Auth
WebID, Mozilla Persona, OpenID.

###Authorisation

Techniques: ACL, MAC, RBAC.

#####ACL
Table which list each user access to a particular resource.

#####MAC
Every resource has a minimum trust level. Each user wanting to access this resource should match or exceed this level.

#####RBAC
Authorize users with specific roles

It's possible to implement MAC with RBAC, ACL with RBAC, or mix.

####OAuth2
OAuth 2.0 is an open standard for application authorization that allows clients to access resources on behalf of the resource owner. It's the dominant form of third party API auth.

###Conclusion
- Allow multifactor auth
- HTTPS
- Use a `authorize` middleware for all your routes.
- Enable OAuth2  to discourage third-party application vendors from re‐ questing your user’s login credentials.
