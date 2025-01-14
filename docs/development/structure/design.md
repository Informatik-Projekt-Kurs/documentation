# Design Structure

The structure of a Figma project is an important consideration to make since it impacts working efficiency, clarity, and ease of collaboration throughout the design process. By organizing the project in a logical and systematic way, one  can quickly locate assets, iterate on designs, and maintain consistency across different screens and components. This structure not only streamlines the workflow for the design team but also makes it easier for developers to access design files and extract the necessary details. These are the reasons why we placed such a strong emphasis on maintaining the continuous organization of our design files.

## Flow Diagrams

Flow diagrams help us design user journeys through the application. They help us figure out where the user experience could be improved. Let's analyze the flow chart of the user onboarding, the start of a users journeys in Meetmate.

##### 1. Client Onboarding

The client onboarding process in Meetmate is designed to be easy and safe. New users start their journey on the sign-up page, where they can choose to create an account or log in if they already have an account. Existing users are directed to the login page, so they can quickly access their accounts. This approach ensures that new and returning users have a streamlined experience.
![[Flowchart Picture]](../../assets/client-start.png)
##### 2. Company Setup

Setting up a company profile on Meetmate is important for companies who want to use our platform. The process starts by making sure that only people who have permission can create and manage company profiles. Once users are authenticated, they are guided through a series of steps to set up their company details including entering important information like the company name, industry, and other relevant details.
![[Flowchart Picture]](../../assets/company-start.png)

#### Key Takeaways from the Flow Diagram

1. *Multiple Entry Points*: Our flow chart gives different start points for clients and companies to ensure that each user type starts their individual journey in the most relevant and efficient way. This improves the overall user experience and reduces confusion.

2. *Verification Steps*: We implemented important verification processes to keep data safe and secure. The system checks if the information given by the user meets certain requirements before proceeding.

3. *Logical Progression*: The user flow is designed to help users set up their basic credentials before they can use more complicated features like company setup or creating appointments. A progression like this helps users get used to the system gradually, which reduces cognitive load and improves overall usability.