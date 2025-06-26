# Website-Development-Pro

## Enterprise Level

---

### Phase 1: Strategic Planning & Architecture (The Enterprise-Grade Blueprint)

This is the most critical phase. The choices made here will determine your site's ability to scale.

#### 1. Architectural Pattern: Scalable Monolith vs. Microservices

*   **Scalable Monolith (Recommended Start):**
    *   **What it is:** A single, well-structured application (e.g., a Next.js or Django project) that is designed to be "stateless." This means it doesn't store session information in memory and can be easily replicated.
    *   **Why for this use case:** It's significantly less complex to develop and deploy than microservices, while still being highly scalable. You can run multiple instances of the same application behind a load balancer to handle traffic. This is the sweet spot for 95% of applications, including one for a large hackathon.
    *   **Key Principle:** The application must be stateless. User session data is stored in a shared cache (like Redis) or through client-side tokens (JWTs), not on the server instance itself.

*   **Microservices:**
    *   **What it is:** Breaking your application into smaller, independent services (e.g., a `user-service`, `team-service`, `notification-service`), each with its own database and deployment pipeline.
    *   **Why for this use case:** This is generally overkill for a hackathon website, even a large one. It introduces immense operational complexity. You would only consider this if the hackathon platform were a long-term, multi-feature commercial product.

#### 2. Technology Stack Selection for Scale & Professionalism

| Category         | Professional Recommendation                                     | Why It's Built for Scale                                                                                                                  |
| ---------------- | --------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| **Frontend**     | **Next.js (React)** or **Nuxt.js (Vue)** with **TypeScript**      | **SSR/SSG:** Server-Side Rendering & Static Site Generation create incredibly fast initial page loads. **TypeScript:** Enforces type safety, which is crucial for large codebases to prevent bugs. Component-based architecture is inherently scalable for development teams. |
| **UI/Component Library** | **MUI (Material-UI)**, **Ant Design**, **Chakra UI** | These libraries provide a comprehensive set of beautiful, accessible, and themeable components out-of-the-box. This ensures a consistent, professional look and saves hundreds of hours of design and CSS work. |
| **Backend**      | **Node.js (with NestJS framework)** or **Go (with Gin framework)** | **NestJS:** A highly structured, opinionated framework for Node.js using TypeScript. It promotes scalable architecture (modules, dependency injection) from the start. **Go:** Blazing fast performance and low memory footprint, ideal for handling high concurrency (many simultaneous registrations). |
| **Database**     | **PostgreSQL**                                                  | The gold standard for relational data. It's robust, reliable, and has powerful features like JSONB for flexible data, and strong indexing capabilities which are **critical** for performance with thousands of rows. |
| **Cache**        | **Redis**                                                       | An in-memory data store used for caching frequently accessed data (like user profiles, team data) to reduce database load and for managing user sessions at scale. |

---

### Phase 2: Frontend Development (The "Smooth Professional Front")

This is about creating a user experience that feels fast, intuitive, and trustworthy.

#### 1. State Management
For an app with logins, dashboards, and team data, you need a robust state management solution.
*   **React:** **Redux Toolkit** (the industry standard) or **Zustand** (a simpler, modern alternative).
*   **Vue:** **Pinia** (the official, modern choice).
*   **Benefit:** This provides a single, predictable source of truth for your application's data, preventing UI inconsistencies and making the app easier to debug.

#### 2. Performance Optimization (The "Smooth" Factor)
*   **Code Splitting:** Your framework (Next.js/Nuxt.js) does this automatically by page. This means users only download the code for the page they are visiting, not the entire site.
*   **Lazy Loading:** Lazy load images and components that are not in the initial viewport. This drastically improves initial page load speed.
*   **Optimistic UI Updates:** When a user performs an action (e.g., joins a team), update the UI *immediately* as if it succeeded, while the network request runs in the background. If it fails, roll back the change and show an error. This makes the app feel instantaneous.
*   **Image Optimization:** Use modern image formats (`.webp`) and serve properly sized images for the user's screen. Next.js has a built-in `<Image>` component for this.

#### 3. Professional UI/UX
*   **Clear Call-to-Action (CTA):** The "Register Now" button should be prominent.
*   **Intuitive Forms:** Use clear labels, validation messages, and loading indicators on buttons. For multi-step forms (like registration), show a progress bar.
*   **Responsive Design:** Use a mobile-first approach. Your component library (MUI, Ant Design) will handle most of this, but always test on real devices.

---

### Phase 3: Backend Development (The "Engine for Thousands")

This is where you build the system to handle the load.

#### 1. API Design: REST or GraphQL
*   **REST API:** The standard, well-understood choice. Design clear, resource-based endpoints (e.g., `POST /users`, `GET /teams/:id`, `POST /teams/:id/join`).
*   **GraphQL:** Allows the frontend to request exactly the data it needs, reducing over-fetching. It's more complex to set up but can be more efficient for complex dashboards.

#### 2. Handling High-Traffic Scenarios (Registration Spike)
This is the most important part of scaling your backend.
*   **Asynchronous Job Queues:** When a user registers, some actions can take time (e.g., sending a confirmation email, generating a certificate).
    1.  The registration API endpoint should **only** do one thing: validate the data and save it to the database. This is very fast.
    2.  It then pushes a "job" (e.g., `send-confirmation-email`) onto a message queue like **RabbitMQ** or a cloud service like **AWS SQS**.
    3.  Separate "worker" processes listen to this queue, pick up jobs, and execute them (e.g., call the email API).
    *   **Result:** The registration API endpoint remains fast and responsive even if 1,000 people sign up in one minute. The emails will be sent out steadily by the workers without crashing your main server.

*   **Rate Limiting:** Protect your API from abuse and overload by limiting the number of requests a single IP address can make in a given time period (e.g., 100 requests per minute).

#### 3. Database Strategy for Scale
*   **Connection Pooling:** Maintain a "pool" of open database connections instead of opening/closing one for every request. This dramatically reduces overhead. All major ORMs (Prisma, TypeORM, Django ORM) handle this for you.
*   **Database Indexing (CRITICAL):**
    *   An index is a special lookup table that the database search engine can use to speed up data retrieval.
    *   You **must** create indexes on columns that you frequently search by. At a minimum:
        *   `users` table: index the `email` column (for login and checking for duplicates).
        *   `teams` table: index the `joinCode` column.
    *   Without indexes, a query like `SELECT * FROM users WHERE email = '...'` would have to scan the entire table, which becomes incredibly slow with thousands of users.

---

### Phase 4: DevOps & Deployment (The "Infrastructure for Scale")

How you host and maintain the application is key to its reliability.

#### 1. Hosting Environment: Cloud Providers
Forget basic hosting. You need a real cloud provider.
*   **AWS, Google Cloud (GCP), or Azure.**
*   **Recommended Service:** Use a Platform-as-a-Service (PaaS) that manages the underlying servers for you, like **AWS Elastic Beanstalk** or **Google App Engine**. Or, for more control, use a container orchestration service.

#### 2. Containerization: Docker & Kubernetes
*   **Docker:** Package your frontend and backend applications into "containers." A container includes the application and all its dependencies, ensuring it runs identically everywhere.
*   **Kubernetes (or a managed equivalent like AWS EKS, GKE):** This is a container orchestrator. You tell it, "I want to run 3 instances of my backend container." It will automatically run them, monitor their health, and if one crashes, it will restart it.
*   **Auto-scaling:** You can configure Kubernetes to automatically add more containers (scale out) when CPU usage gets high (like during a registration rush) and remove them when traffic dies down (to save money).

#### 3. CI/CD (Continuous Integration / Continuous Deployment)
*   **What it is:** An automated pipeline that builds, tests, and deploys your code whenever you push to your Git repository (e.g., GitHub).
*   **Tools:** **GitHub Actions** (built into GitHub), GitLab CI, Jenkins.
*   **Why it's professional:** It ensures every change is tested, reduces human error in deployments, and allows for rapid, reliable updates.

### Summary: A Recommended Production-Grade Stack

*   **Frontend:** Next.js with TypeScript.
*   **UI Library:** MUI (Material-UI).
*   **State Management:** Redux Toolkit.
*   **Backend:** NestJS (Node.js/TypeScript).
*   **Database:** PostgreSQL (hosted on a managed service like AWS RDS).
*   **Cache:** Redis (hosted on a managed service like AWS ElastiCache).
*   **Job Queue:** RabbitMQ or AWS SQS.
*   **Deployment:**
    *   Package the Next.js and NestJS apps into separate **Docker** containers.
    *   Deploy them to a managed **Kubernetes** service (AWS EKS or GKE).
    *   Set up a **CI/CD pipeline** with GitHub Actions.
    *   Use a **Load Balancer** to distribute traffic across your containers.
    *   Use a **CDN** (like Cloudflare or AWS CloudFront) to serve your frontend assets globally for speed.

This stack is a modern, powerful, and scalable approach that can comfortably handle thousands of registrations and provide a truly professional experience for your hackathon participants.
-------------------------------------------------------------------------------------------

## Initial 

### Phase 1: Planning & Scoping (The "Blueprint" Strategy)

Before writing a single line of code, this is the most critical phase.

#### 1. Define the Core Features (Minimum Viable Product - MVP)
Your goal is to get a functional site up quickly. Don't try to build everything at once.

*   **Landing Page:** A simple, attractive page explaining the hackathon (what, when, where, why).
*   **User Registration:** A form to collect essential participant data.
    *   Fields: Full Name, Email, Password, University/Company, T-shirt size, Dietary Restrictions, Resume/Portfolio Link.
*   **User Login/Authentication:** Allow registered users to log back in.
*   **User Dashboard:** A simple, private page a user sees after logging in.
    *   Shows their registration status (e.g., "Registered").
    *   Allows them to view/update their information.
*   **Team Formation:**
    *   Create a new team.
    *   Join an existing team using a unique code/link.
    *   View team members.
*   **Admin View (Simple):** A protected route where organizers can view and export a list of all registered participants as a CSV/Excel file.

#### 2. Define "Nice-to-Have" Features (Post-MVP)
List these separately. You can add them if you have time.

*   Project Submission Form.
*   Real-time announcements section.
*   Social logins (Google, GitHub).
*   Password reset functionality.
*   A "judging" portal.
*   Automated confirmation emails.

#### 3. Choose a Technology Stack
This choice depends on your team's skills and the need for speed.

| Category      | Popular Options                                       | Pros for a Hackathon Site                                                              | Cons                                                              |
|---------------|-------------------------------------------------------|----------------------------------------------------------------------------------------|-------------------------------------------------------------------|
| **Frontend**  | **React (Next.js)**, **Vue (Nuxt.js)**, Svelte (SvelteKit) | Component-based, great for dynamic UIs, rich ecosystems. Frameworks like Next/Nuxt handle routing and server-side rendering easily. | Can have a steeper learning curve if you're new.                  |
|               | **HTML/CSS + Vanilla JS + Bootstrap/Tailwind**        | Extremely fast to set up for simple pages. No complex build steps.                     | Becomes hard to manage as the app grows more dynamic (e.g., dashboard). |
| **Backend**   | **Node.js (Express/NestJS)**                          | Fast, uses JavaScript (same language as frontend), huge library (npm) ecosystem.       | Asynchronous nature can be tricky for beginners.                   |
|               | **Python (Django/Flask)**                             | Django is "batteries-included" with a built-in admin panel (huge time saver!). Flask is minimal and flexible. | Can be slightly slower to set up than a Node/Express server.      |
|               | **Go (Gin)**                                          | Extremely performant and creates a single binary for easy deployment.                  | Smaller ecosystem, stricter typing can slow initial development. |
| **Database**  | **PostgreSQL / MySQL (SQL)**                          | Structured, reliable, great for relational data (users belong to teams).               | Requires defining a schema upfront, can be less flexible.         |
|               | **MongoDB (NoSQL)**                                   | Flexible schema, great for rapid prototyping where data structures might change.       | Can be harder to manage complex relationships.                     |
| **All-in-One**| **Firebase / Supabase (Backend-as-a-Service)**        | **(Highly Recommended for Speed)** Handles Auth, Database, and Hosting in one place. Reduces backend code you need to write by 90%. | Less control, potential vendor lock-in (not a big deal for a short-term project). |

---

### Phase 2: Core Development Strategies

This is how you'll build the features you defined.

#### 1. Architecture Strategy: Monolith vs. Serverless

*   **Monolithic Approach:**
    *   **What it is:** Your frontend and backend code are in the same project, deployed as a single unit (e.g., a Next.js app with API routes, or a Django app serving templates).
    *   **Why for a hackathon site:** **Simplicity.** It's faster to develop, easier to test, and has a single deployment process. This is the most practical choice for a small team on a deadline.

*   **Serverless / BaaS (Backend-as-a-Service) Approach:**
    *   **What it is:** You build your frontend (e.g., in React) and connect it directly to a third-party service like **Firebase** or **Supabase** for all backend needs.
    *   **Why for a hackathon site:** **Extreme speed.**
        *   **Authentication:** Use their pre-built, secure login and registration flows in minutes.
        *   **Database:** Use Firestore or Supabase's Postgres DB without writing any backend server code for CRUD (Create, Read, Update, Delete) operations.
        *   **Hosting:** Deploy your frontend with one command.

#### 2. User Registration & Authentication Strategy

*   **Data Flow:**
    1.  User fills out the registration form on the frontend.
    2.  On submit, the frontend sends the data to a backend API endpoint (e.g., `/api/register`).
    3.  **Backend Logic:**
        *   **Validate the data:** Is the email valid? Is the password strong enough? Are all required fields present? **Never trust frontend validation alone!**
        *   **Check for existing user:** Query the database to see if the email is already in use.
        *   **Hash the password:** **NEVER** store passwords in plain text. Use a strong, salted hashing algorithm like **bcrypt**.
        *   **Store user in the database:** Insert the new user's data (including the *hashed* password) into your `users` table.
        *   **Generate a token:** Create a JSON Web Token (JWT) that contains the user's ID. This token proves they are logged in.
    4.  The backend sends the JWT back to the frontend.
    5.  The frontend stores the JWT (in `localStorage` or a cookie) and redirects the user to their dashboard.

*   **Session Management:** For every subsequent request to a protected page (like the dashboard), the frontend must include the JWT in the request headers. The backend then verifies the token before returning any private data.

#### 3. Data Storage & Modeling Strategy

*   **Define your Data Models (Schema):**
    *   `User`:
        *   `id` (Primary Key)
        *   `fullName` (String)
        *   `email` (String, Unique)
        *   `passwordHash` (String)
        *   `university` (String)
        *   `teamId` (Foreign Key, nullable)
        *   ...other fields
    *   `Team`:
        *   `id` (Primary Key)
        *   `teamName` (String, Unique)
        *   `joinCode` (String, Unique)
*   **Use an ORM (Object-Relational Mapper):**
    *   An ORM like **Prisma**, **Sequelize** (for Node.js), or the **Django ORM** lets you interact with your database using simple code instead of writing raw SQL queries. This is a massive time-saver and reduces errors.

---

### Phase 3: Deployment & Operations Strategy

How your site will go live and stay live.

#### 1. Hosting Strategy

*   **PaaS (Platform-as-a-Service):** The best choice for ease of use.
    *   **Vercel / Netlify:** Perfect for frontend-focused or full-stack Next.js/Nuxt.js apps. They offer "Git Push to Deploy." You push to your main branch, and the site redeploys automatically.
    *   **Heroku / Render:** Great for traditional backend servers (Node/Express, Django/Flask). They also offer simple deployment from a Git repository.
*   **Serverless Platforms:**
    *   **Firebase Hosting / Supabase:** If you use their BaaS tools, using their hosting is the most seamless option.

#### 2. Security Best Practices (Non-Negotiable)

*   **HTTPS/SSL:** Your site must be served over HTTPS to encrypt data in transit. All modern hosting providers offer this for free.
*   **Environment Variables:** Store all sensitive information (database URLs, API keys, JWT secrets) in environment variables (`.env` file), not hard-coded in your source code.
*   **Input Validation:** Validate all incoming data on the **server-side**.
*   **Password Hashing:** Use **bcrypt** as mentioned.
*   **CORS (Cross-Origin Resource Sharing):** If your frontend and backend are on different domains, configure your backend server to only accept requests from your frontend's domain.

---

### Recommended "Fast-Track" Stack & Workflow

For a typical hackathon page where speed is everything, here is a highly recommended path:

1.  **Framework:** **Next.js (React)**. It gives you a great structure for pages, a simple API routing system, and is perfect for modern deployment platforms.
2.  **Backend & Database:** **Supabase**.
    *   Use **Supabase Auth** for user registration and login. Their client library makes it incredibly simple. It handles password hashing, social logins, and JWTs for you.
    *   Use the **Supabase Postgres Database**. You can create your tables in their UI and interact with them directly from your Next.js frontend using their client library. **You write almost no custom backend code.**
3.  **Styling:** **Tailwind CSS**. A utility-first CSS framework that lets you build modern designs quickly without writing custom CSS files.
4.  **Deployment:** **Vercel**. Connect your GitHub repository. Every time you `git push`, Vercel will automatically build and deploy your Next.js site.

**This combination (Next.js + Supabase + Vercel) allows a single developer to build and deploy a secure, dynamic registration site in a weekend.**



------------------------------------------------------------------------

## Security

Of course. Integrating robust security is non-negotiable for a scaled application handling personal data. Security must be a core principle woven into every phase of development, not a feature added at the end.

Here is a comprehensive strategy for maintaining security, layered on top of the plan for a scalable, professional hackathon website. This approach is known as **"Defense in Depth."**

---

### The Guiding Principle: Defense in Depth

This means creating multiple, redundant layers of security. If one layer is breached, another is there to stop the attack. We will apply this principle across your application, data, infrastructure, and operational processes.

### Layer 1: Application-Level Security (The Code Itself)

This is about writing secure code and preventing common web vulnerabilities. This is where your developers have the most control.

#### 1. Secure Authentication & Session Management (The Front Door)
*   **Use an Established Authentication Library:** Don't write your own authentication from scratch. Use battle-tested solutions provided by your framework or a service.
    *   **Recommended:** Use **Supabase Auth**, **Auth0**, or **AWS Cognito**. They handle password hashing, social logins, JWT management, and Multi-Factor Authentication (MFA) securely out of the box.
*   **JWT Security Best Practices:** If you implement your own JWT logic:
    *   **Use Strong Secret Keys:** The secret used to sign your JWTs must be long, random, and stored in a secure secrets manager (like AWS Secrets Manager), **not** in your `.env` file in production.
    *   **Short-Lived Access Tokens:** Access tokens should have a short expiry (e.g., 15-30 minutes).
    *   **Use Refresh Tokens:** Implement a secure refresh token mechanism to get a new access token without forcing the user to log in again. Refresh tokens should be stored securely in the database and have a longer expiry.
    *   **Store Tokens Securely on the Client:** Store JWTs in **secure, `HttpOnly` cookies**. This prevents them from being accessed by client-side JavaScript, which is the primary defense against Cross-Site Scripting (XSS) token theft. Avoid `localStorage`.
*   **Enforce Strong Password Policies:** On your registration form, enforce password complexity rules (length, numbers, special characters) on both the frontend (for UX) and backend (for security).
*   **Implement Multi-Factor Authentication (MFA/2FA):** For a truly professional and secure site, offer MFA using an authenticator app (TOTP). This is a massive step up in account security.

#### 2. Authorization & Access Control (Who Can Do What)
*   **Implement Role-Based Access Control (RBAC):**
    *   Add a `role` field to your `User` model (e.g., 'participant', 'admin', 'judge').
    *   **On the backend**, every single API endpoint that performs a sensitive action or returns sensitive data must verify the user's role before proceeding. For example, the `GET /api/admin/registrations` endpoint must check that `user.role === 'admin'`.
    *   **Never rely on the frontend to hide links or buttons.** A malicious user can always call the API directly.

#### 3. Input Validation and Sanitization (Trust No One)
*   **Validate Everything on the Backend:** This is the most critical rule. The backend must validate every piece of data it receives from the client.
    *   **Use a validation library:** For a NestJS backend, use the built-in `class-validator` library. For any Node.js app, `Zod` is an excellent, modern choice.
    *   **What this prevents:**
        *   **SQL Injection:** By using an ORM (like Prisma or TypeORM) and validating input types, you are largely protected from SQL injection.
        *   **Malformed Data:** Prevents corrupted data from entering your database.
*   **Sanitize Output to Prevent Cross-Site Scripting (XSS):**
    *   If you ever display user-generated content (e.g., a team name, a user's profile bio), you must sanitize it before rendering it in HTML.
    *   Modern frameworks like React and Vue do this automatically for most content, but be extremely careful if you ever use functions like `dangerouslySetInnerHTML`.

#### 4. Harden Your HTTP Headers
*   Use a library like `helmet` for Express/NestJS to automatically set secure HTTP headers. These headers instruct the browser to enable security features, preventing attacks like clickjacking and cross-site scripting.

---

### Layer 2: Data Security (Protecting the Crown Jewels)

#### 1. Encryption Everywhere
*   **Encryption in Transit:** Your site **must** use **HTTPS (SSL/TLS)**. This is standard and provided by all modern hosting platforms (Vercel, AWS Load Balancers, etc.). It encrypts data between the user's browser and your server.
*   **Encryption at Rest:** The data files sitting on your database server's disk should be encrypted.
    *   **Solution:** Use a managed database service like **AWS RDS** or **Google Cloud SQL**. They provide encryption at rest with the click of a button, managing the encryption keys for you.
*   **Application-Level Field Encryption:** For extremely sensitive data (e.g., API keys for a project submission), consider encrypting the specific field within your application *before* it's stored in the database.

#### 2. Database Security
*   **Principle of Least Privilege:** Create a specific database user for your application. This user should **only** have the permissions it needs (`SELECT`, `INSERT`, `UPDATE`, `DELETE`) on the specific tables required. It should **not** be a database administrator or superuser.
*   **Secrets Management:**
    *   Your database credentials, API keys, and JWT secrets must not be in your code or even in a source-controlled `.env` file.
    *   **Use a dedicated secrets manager:** **AWS Secrets Manager**, **Google Secret Manager**, or **HashiCorp Vault**. Your application fetches these secrets at startup.

---

### Layer 3: Infrastructure & Network Security (Securing the Fortress)

#### 1. Secure Network Architecture
*   **Use a Virtual Private Cloud (VPC):** Deploy your resources inside a VPC on your cloud provider (AWS, GCP).
*   **Private Subnets for Databases:** Your database and cache (Redis) should be in a **private subnet**, meaning they are **not accessible from the public internet**.
*   **Public Subnets for Web Servers:** Only your load balancer and web server instances (or serverless functions) should be in a public subnet, able to receive traffic from the outside world.
*   **Firewall Rules (Security Groups):** Configure strict firewall rules.
    *   Allow public traffic only on ports `80` (HTTP, which should redirect to HTTPS) and `443` (HTTPS).
    *   Allow your web servers to connect to your database's port (e.g., `5432` for PostgreSQL) but block all other access to that port.

#### 2. Dependency & Supply Chain Security
*   **Automated Dependency Scanning:** A huge number of attacks come from vulnerable open-source packages.
    *   **Use GitHub's Dependabot or Snyk** to automatically scan your dependencies (`package.json`) for known vulnerabilities and create pull requests to update them.
    *   Regularly run `npm audit fix` during development.

#### 3. Logging, Monitoring, and Alerting
*   **Comprehensive Logging:** Log all security-sensitive events:
    *   Successful and failed login attempts (with IP address).
    *   Password reset requests and completions.
    *   Changes to user roles or permissions.
*   **Centralized Logging:** Ship your logs to a centralized service like **AWS CloudWatch**, **Datadog**, or an **ELK Stack**.
*   **Set Up Alerts:** Create automated alerts for suspicious activity. For example:
    *   Alert if a single IP address has more than 10 failed login attempts in 5 minutes (potential brute-force attack).
    *   Alert on any attempt to access an admin endpoint by a non-admin user.

---

### Layer 4: Operational Security (The Human Element)

*   **Git Security:**
    *   Protect your `main` branch. Enforce a policy where all code must be reviewed via a **Pull Request (PR)** before being merged.
    *   Never commit secrets or credentials to Git. Use a tool like `git-secrets` to scan for accidental commits.
*   **Principle of Least Privilege for Humans:** Not every developer on the team needs access to production keys or the production database. Restrict access on an as-needed basis.
*   **Regular Security Audits:**
    *   **Static Application Security Testing (SAST):** Integrate tools into your CI/CD pipeline that scan your source code for potential security flaws.
    *   **Penetration Testing:** For a truly large-scale or high-stakes event, consider hiring a third-party security firm to perform a penetration test on your application before launch.

### Ultimate Security Checklist for Go-Live

1.  **[ ] HTTPS is enabled and enforced everywhere.**
2.  **[ ] A strong password policy is enforced on the backend.**
3.  **[ ] All user passwords are being hashed with `bcrypt` or a similar algorithm.**
4.  **[ ] All API endpoints have backend authorization checks (RBAC).**
5.  **[ ] All incoming data is validated on the backend with a library like `Zod` or `class-validator`.**
6.  **[ ] Secure HTTP headers are being set (e.g., using `helmet`).**
7.  **[ ] JWTs are stored in `HttpOnly` cookies, not `localStorage`.**
8.  **[ ] Production secrets (DB credentials, API keys) are in a dedicated secrets manager, not source control.**
9.  **[ ] The database is in a private network subnet, inaccessible from the public internet.**
10. **[ ] Firewall rules are restrictive and follow the principle of least privilege.**
11. **[ ] Automated dependency scanning (Dependabot/Snyk) is active on the repository.**
12. **[ ] Security event logging and alerting are configured.**
13. **[ ] The `main` Git branch is protected, requiring PRs for all changes.**
