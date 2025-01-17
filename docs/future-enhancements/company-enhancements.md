# Company Enhancements

Companies are a cornerstone of MeetMate, offering significant creative opportunities to shape their functionality. Many features have been carefully considered, with some systems partially implemented and awaiting a last few final touches. 

## Company Members
While there is a general idea backend already has an implementation in place, the frontend addition was a bit more time consuming. Therefore a design and its functional implementation are not yet completed. Some issues also arise form parity between the original idea and its backend counter part.

#### Company Member Onboarding
To clarify the onboarding process for company members, a specialized flow was designed. The process begins with an invite link, typically sent by the company administrator. When clicked, the link directs the new member to a dedicated page where they can create their password and access the company’s MeetMate account. Once registered, the new member can quickly join the platform and collaborate with their team.

![[Flowchart Picture]](../../assets/member-start.png)

#### Members in the Backend
Meanwhile the backend implementation considers the company owner as an administrator, in of fully setting up and managing the employees. This system, while fully functional, currently sees no use and may require adjustments to better align with the evolving project. The backend enforces authorisation checks to allowing company members to perform editing tasks similar to the owner while restricting access to critical company settings.

The company member section includes getting information on specific or all members, adding / creating new members and deleting existing members. These methods inherit parts of the normal client implementation while always confining the access to a companies members or owners. These checks mirror client-side restrictions but are tailored to handle company-specific cases. Notably, members should not be able to delete their own accounts messing up the internal structure of companies, therefore the clients endpoints for deletion are using similar checks to stop this issue. Common validations have been abstracted into reusable methods to maintain consistency and reduce redundancy.

```java
  private boolean isNotCompanyOwner(String email) {
    return userRepository.findUserByEmail(email)
        .orElseThrow(() -> new EntityNotFoundException("User not found!"))
        .getRole() != UserRole.COMPANY_OWNER;
  }

  private boolean isNotCompanyMember(Company company, long memberId) {
    return !company.getMemberIds().contains(memberId);
  }
```

#### Challenges and Next Steps
The existing backend system is functional but needs better synchronization with the frontend and refinements to match real-world use cases. Completing the frontend integration and resolving discrepancies between initial designs and backend implementations will help finalize this feature, making it fully usable and user-friendly.