# Further Verification

## OAuth
MeetMate plans to introduce additional authentication methods, such as social logins (e.g., Google, GitHub), to provide users with a faster and more convenient signup process. This feature will be implemented while maintaining the highest standards of security and data protection, through the help of OAuth. First signs of this can be seen already when looking at the signup or login process of clients, although not yet functional.

![[Example of OAuth used in client sign up]](../assets/oauth.png)


## Email Verification

#### Initial Decision
To verify the identity of users and safeguard the application against malicious activities, such as the mass creation of spam accounts, we thought about implementing an email verification system, where upon account creation, a verification email is automatically sent to the user. Developing a solution for this complex automated process, which relies on interactions with external systems beyond our ecosystem, presented challenges. However, instead of diving deeply into building a custom implementation for a rather small part of the whole system, we focused on exploring other feasible integration options for the time. 

#### Plans
Some initial considerations were made regarding the implementation of this type of verification. One proposed approach involved establishing a communication flow between the frontend and backend. The idea was to generate a unique verification link for each user, containing an identifier. This link would be sent via the verification email, and when accessed, the backend would process the identifier to validate the user.

![[Table of verify route]](../assets/verify-table.png)

Additionally, designing verification emails presented its own set of challenges. Many email clients limit or outright ignore additional CSS, requiring an emphasis on pure HTML for compatibility. If implemented, this programming task would have shifted to our design team to ensure the emails maintained both functionality and an appealing design within these constraints.


## Two-Factor Authentication
Additionally, MeetMate may consider implementing advanced security features like two-factor authentication (2FA) for enhanced security and user verification. While such features would require extensive integration with external services, ensuring their proper implementation is crucial to avoid potential data leaks, especially when handling sensitive processes like generating and validating SMS codes.