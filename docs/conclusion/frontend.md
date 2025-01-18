# Frontend Development and Project Management

## Role
As the frontend developer and project manager of MeetMate, I was responsible for the implementation of the user interface, coordinating design and development, and keeping track of the overall project progress. Myself, being a professional software developer, I brought experience with modern development tools and practices to the team, while still finding many opportunities to grow and learn during the project.

## Technical Implementation and Coordination
My journey started by setting up the project infrastructure – creating the GitHub organization, establishing a Jira workspace, and structuring our Confluence documentation. While I was familiar using these tools from professional experience, using them to manage a project myself however posed a new challenge. The implementation of a Next.js frontend with custom authentication, which despite my professional experience turned out to be much more complex to develop from scratch compared to what one would expect with experience in a number of libraries.

When working with both the design and backend careful coordination was especially important. I established a Letter of Agreements (LoA) document in Confluence, which became our reference point for API specifications. This helped to coordinate the different aspects of development, ensured consistent implementation across the application and served as a quick reference for various endpoint requirements.

## Project Management Journey
Taking on the role of project manager was both challenging and enlightening. Setting up the project management infrastructure taught me the importance of clear organization from the start. Creating tickets, managing timelines, and setting deadlines required a different mindset from pure development work. The responsibility of keeping team members accountable while sometimes uncomfortable was crucial for project progress.

The challenge of balancing authority with collaboration was particularly noteworthy. Having to ensure deadlines were met while maintaining positive team dynamics taught me valuable lessons about leadership and communication. Regular meetings and clear documentation helped maintain project momentum while keeping everyone aligned with our goals.

## Development Challenges
Despite my professional background, this project presented unique technical challenges. Building core systems from scratch, particularly the authentication system, revealed complexities that are often hidden when using established libraries.

After all, state management was an interesting journey. Initially considering Redux for global state management, I eventually realized that React Contexts provided a more elegant solution. This decision led to cleaner code, better data flow throughout the application and removed any unnecessary local state management. Centralizing data fetching through contexts took some iteration to get right, but ultimately resulted in a more maintainable codebase.

One of the biggest challenges was the implementation of authentication and role-based routing using Next.js middleware. With the app router being relatively new, documentation was limited, and solutions were hard to find. This required extensive experimentation to develop a robust solution that could handle token refresh, role-based access control and proper redirect flows. The final implementation successfully manages user tokens and route protection, though reaching this point involved considerable trial and error.

The integration of both REST and GraphQL APIs required careful planning and implementation to ensure efficient data flow throughout the application. Working with the Apollo Client for GraphQL proved valuable especially its built-in caching and polling capabilities which eliminated the need for additional state management solutions.

Another technical challenge came from customizing third-party components. The Syncfusion scheduler component, while powerful in functionality, required significant customization to match our specific design requirements. Adapting its default styling and behavior to meet our needs while maintaining its core functionality was a difficult task.

The coordination between frontend and backend development demanded clear communication and documentation. Creating and maintaining the LoA document helped establish clear boundaries and expectations, making the integration process smoother despite its challenges.

## Learning Outcomes
This project significantly enhanced both my technical and management skills. While I was familiar with many of the tools and technologies used, implementing them in a leadership role provided new perspectives. Managing a project while actively developing taught me to balance immediate technical needs with longer-term project goals.

## Conclusion
The experience of leading this project while handling frontend development has been invaluable. It has enhanced my understanding of project management, improved my technical implementation skills, and taught me important lessons about team leadership. These insights will prove valuable in both my professional career and future project endeavors.
