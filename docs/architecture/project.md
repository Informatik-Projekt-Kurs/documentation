# Architecture Overview

This section dives into the architectural framework of MeetMate, outlining the key components and technologies that drive our application. It outlines the key components and technologies that drive our application. 

## **Overall System Architecture**

Good organization is an important key to the success of any project because it ensures that all team members are informed, tasks are defined, and progress can be tracked easily.
To do that we use tools like **Jira**, **Confluence**, and **GitHub**. 
To manage tasks, keep our documentation up-to-date, and optimize our code development, it helps us to reduce the risk of miscommunication in our team.
Clear priorities, following a timeline, and adapting to changes quickly are part of organization, which is a major factor in success. 
Organization helps us work better together because everyone has access to the same information and understands their roles and responsibilities.

### **Collaboration and Version Control Tools**

- **Jira:**  We use Jira to track and manage project tasks, issues, and progress. It allows us to assign tasks and track the status of various components of our project via customizable boards and Timelines.

- **GitHub:** GitHub is a useful tool that helps us working on code together. 
We use it for thinks like version control, code management, tracking changes, and effectively managing branches. 
The integration between Jira and GitHub allows us to link issues and commits by that wie can easily gain insight into the correlation between code and tasks.

- **Confluence:** Confluence helps us serve as our primary documentation and knowledge base.
We use Confluence to save data-structures, integration of frontend and backend, general data of our project, and collect ideas for future features. 

- **Discord:** Discord serves as our primary communication hub, structured with specialized channels for different aspects of the project:
    - **INFO channels:** Includes welcome information, announcements, progress updates, and resources for quick reference
    - **TEAMS channels:** Dedicated spaces for general discussion, development topics, product planning, and improvements
    - **Automated Notifications:** A specialized GitHub notifications channel that automatically receives updates about pull requests, code reviews, and other repository activities
    - **VOICE channels:** Separate voice channels for different team meetings (Dev, Product) to facilitate real-time discussions and collaboration <br/>
  
    > This structure ensures clear communication paths and keeps all team members informed about project updates and developments.


### **Development and Deployment Infrastructure**

- **Cloud Services:** Utilizing cloud platforms for hosting, data storage, and scalability.
- **CI/CD Pipeline:** Continuous integration and continuous deployment practices for streamlined and efficient release cycles.

For hosting the API and databases of the backend in one place, we use DigitalOcean, while the frontend website runs on Netlify.

> **Note**: This structure provides a comprehensive overview of the application architecture, detailing the tools, technologies, and methodologies used in the development of MeetMate. It's designed to be informative for both technical and non-technical users, ensuring a clear understanding of the application's infrastructure.
