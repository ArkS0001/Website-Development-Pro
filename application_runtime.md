This model is split into two parts:
1.  **Application Runtime Flow:** How the live system processes user requests at scale.
2.  **Development & Deployment Flow:** The professional CI/CD pipeline used to build and release the software.

---

### I. Application Runtime Flow

This flowchart illustrates how the live, deployed system handles user registration, login, and role-based actions, emphasizing scalability and reliability through asynchronous processing and clear architectural separation.

```markdown
+------------------------------------+
|   Start: User arrives at the Site  |
+------------------------------------+
                 |
                 V
+------------------------------------+
|   Frontend App (React/Next.js) is  |
|   served via a Global CDN        |
+------------------------------------+
                 |
                 V
+------------------------------------+
|      New or Returning User?        |
+------------------------------------+
     |                               |
  New User                         Login
     |                               |
     V                               V
+-----------------------------+     +-----------------------------+
| User submits Registration Form |     | User submits Login Credentials|
| (Calls Backend Registration API) |     | (Calls Backend Session API)   |
+-----------------------------+     +-----------------------------+
     |                               |
     V                               V
+-----------------------------+     +-----------------------------+
| Backend API: Validate Data  |     | Backend API: Find user record |
|   Integrity and Format      |     | in Primary DB               |
+-----------------------------+     +-----------------------------+
     |                               |
     |                               V
     |                      +-----------------------------+
     |                      | Verify user credentials     |
     |                      +-----------------------------+
     |                               |
     V                               |
+-----------------------------+     |
| Check for duplicate records |     |
| in Primary DB (PostgreSQL)  |     |
+-----------------------------+     |
     |                               |
     V                               V
+-------------------------------------------------------+
|   If valid, create an active User Session             |
|   (Could be stateful via Redis or stateless via token)|
+-------------------------------------------------------+
     |
     | (Only on New Registration)
     V
+-----------------------------+
| Transaction: Create User    |
| Record in Primary DB        |
+-----------------------------+
     |
     V
+-----------------------------+
| **Push non-critical tasks** |
| (e.g., "Send Welcome Email")|
| **to a Job Queue (RabbitMQ/SQS)** |
+-----------------------------+
     |
     |
     V
+-------------------------------------------------------+
|      User Session is Active & User is on Dashboard    |
|      (Backend Workers process Job Queue asynchronously) |
+-------------------------------------------------------+
                    |
                    V
+-------------------------------------------------------+
|         User attempts a role-specific action          |
+-------------------------------------------------------+
                    |
                    V
+-------------------------------------------------------+
|  Backend API receives request & verifies active session |
+-------------------------------------------------------+
                    |
                    V
+-------------------------------------------------------+
|     Architectural Logic: Check User Role              |
|     ('participant', 'admin', etc.)                    |
+-------------------------------------------------------+
     |                               |
 (Participant)                   (Admin)
     |                               |
     V                               V
+-----------------------------+     +-----------------------------+
| Fetch user-specific data    |     | Fetch aggregated or all data|
| from DB or Cache (Redis)    |     | (e.g., for Admin Export)    |
+-----------------------------+     +-----------------------------+
     |                               |
     V                               V
+-----------------------------+     +-----------------------------+
| Return tailored response    |     | Process and return data     |
| for standard user dashboard |     | in required format (CSV)    |
+-----------------------------+     +-----------------------------+
     |                               |
     V                               V
+-----------------------------+     +-----------------------------+
|    End of Participant Flow  |     |       End of Admin Flow     |
+-----------------------------+     +-----------------------------+

```

---

### II. Development & Deployment (CI/CD) Flow

This flowchart illustrates the professional process for getting code from a developer's machine into a scalable production environment. This is the core of an enterprise development strategy.

```markdown
+------------------------------------+
|   Start: Developer writes code on  |
|   a local machine in a Git branch  |
+------------------------------------+
                 |
                 V
+------------------------------------+
|  Developer pushes branch to a      |
|  central repository (GitHub/GitLab)|
+------------------------------------+
                 |
                 V
+------------------------------------+
|  Developer opens a Pull Request (PR)|
|  to merge into the main branch     |
+------------------------------------+
                 |
                 V
+------------------------------------+
| **Continuous Integration (CI) Kicks Off**|
| (Using GitHub Actions, Jenkins, etc.)|
|                                    |
|   1. Run Code Linters & Formatters |
|   2. Build the Application       |
|   3. Run Automated Tests         |
|      (Unit, Integration, E2E)    |
+------------------------------------+
                 |
                 V
+------------------------------------+      No      +---------------------------+
|      Did all CI checks pass?       |------------->|  Pipeline Fails.          |
+------------------------------------+              |  PR is blocked from merging.|
                 | Yes                                +---------------------------+
                 V
+------------------------------------+
|   Team members perform a           |
|   manual code review on the PR     |
+------------------------------------+
                 |
                 V
+------------------------------------+      No      +---------------------------+
|        Is PR approved?             |------------->|  Developer makes changes  |
+------------------------------------+              |  and pushes again.        |
                 | Yes                                +-----------^---------------+
                 V
+------------------------------------+
|   PR is merged into `main` branch  |
+------------------------------------+
                 |
                 V
+------------------------------------+
| **Continuous Deployment (CD) Kicks Off**|
+------------------------------------+
                 |
                 V
+------------------------------------+
|  Pipeline builds production-ready  |
|  Docker containers for the app    |
+------------------------------------+
                 |
                 V
+------------------------------------+
|  Push Docker images to a           |
|  Container Registry (AWS ECR, etc.)|
+------------------------------------+
                 |
                 V
+------------------------------------+
|  Deployment system (Kubernetes)    |
|  pulls new image and performs a    |
|  rolling update in production      |
+------------------------------------+
                 |
                 V
+------------------------------------+
|   New application version is live! |
|   System is monitored for errors.  |
+------------------------------------+
                 |
                 V
+------------------------------------+
|             End of Flow            |
+------------------------------------+
```
