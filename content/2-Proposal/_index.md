---
title: "Proposal"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Chrono Genesis Game  
## A Turn-Based Trading Card Web Game Built on a Serverless Architecture on AWS

### 1. Executive Summary  
The project is Chrono Genesis Game — a competitive web game (turn-based trading card game) built on a Serverless Real-time Architecture on the AWS platform.

The system simultaneously uses Amazon API Gateway (HTTP API) and Amazon API Gateway (WebSocket API). The HTTP API handles traditional functions such as login, player profile management, deck management, and match history; while the WebSocket API is responsible for synchronizing match state in real time. All game business logic is processed by multiple specialized AWS Lambda functions. Amazon DynamoDB serves as the central data hub, storing match state (Game State), WebSocket connection information, user data, decks, and match history with low latency and high scalability.


### 2. Problem Statement  
*Current Problems*  
- Traditional real-time game systems require continuous server maintenance costs even when no players are active.

- Network latency negatively impacts the player experience in a game that demands constant strategic calculation.

*Solution*  
Deploy a Serverless Real-time Architecture using Amazon API Gateway (HTTP API) and Amazon API Gateway (WebSocket API). All game logic is handled by independent AWS Lambda functions. After a match ends, tasks that do not require an immediate response are placed into Amazon SQS for asynchronous processing, reducing the load on the Game Engine and optimizing overall system performance.

### 3. Solution Architecture  
The platform applies an AWS Serverless architecture to operate a real-time competitive trading card web game capable of automatically scaling to support thousands of concurrent players. The user interface is deployed on AWS Amplify and accessed through Amazon Route 53. Players log in via Amazon Cognito to receive a JWT Token. REST requests (login, profile, deck, history, etc.) are sent to Amazon API Gateway (HTTP API), while in-match operations use Amazon API Gateway (WebSocket API) to ensure low-latency, bidirectional data transmission.

Specialized Lambda functions — Cancel Match, Start Match, Process Game Engine, Save Deck, Handle Timeout, and End Match — handle the entire game lifecycle. Process Game Engine serves as the core component, updating match state in Amazon DynamoDB and broadcasting results to players via WebSocket.

After a match ends, the Lambda End Match function sends an event to Amazon SQS. The Lambda Post Match Worker reads from the queue to handle post-match tasks such as updating Rank, EXP, Match History, and Game Logs. This asynchronous mechanism allows the Game Engine to respond faster, reduces player wait times, and increases system scalability.

![Architecture](/images/2-Proposal/arch.png)


*AWS Services Used*  
- *AWS Amplify*: Hosts and distributes the web game interface (React/TypeScript), automating CI/CD.

- *AWS Lambda*: Processes data and separates business logic into independent functions.

- *Amazon Route 53*: Routes traffic to the web application.  

- *Amazon Cognito*: Authenticates player identity, manages login sessions, and issues JWT Tokens. 

- *Amazon API Gateway (HTTP API)*: Provides REST APIs for login, player profile management, deck management, match history, and other non-real-time functions.

- *Amazon API Gateway (WebSocket API)*: Maintains a real-time, bidirectional connection between players and the Game Engine, transmitting match data with low latency.

- *AWS Lambda*: Handles centralized game logic and computational tasks.

- *Amazon SQS*: An asynchronous queue that receives data from the Lambda Engine for further processing.

- *Amazon DynamoDB*: A NoSQL database that stores match state, connection information, and user data.

- *Security & Monitoring (IAM, KMS, Secrets Manager, CloudWatch, X-Ray)*: Manages access control, key management, log tracking, and system performance monitoring.

*Component Design*  
- *Real-time Routing*: Amazon API Gateway combined with Route 53 manages the bidirectional WebSocket connection between players and the system.   

- *Game Logic Processing*: Specialized Lambda functions (Cancel Match, Start Match, Process Game Engine, Save Deck, Handle Timeout, and End Match) work together to handle the entire lifecycle of a match.

- *Asynchronous Processing*: Amazon SQS receives end-of-match events so that the Lambda worker can automatically calculate Rank, EXP, and save match history.

- *Data Processing*: Amazon DynamoDB stores the board state, connection information, and player profiles.  

- *Web Interface*: Built with React / TypeScript, packaged and distributed via AWS Amplify's CDN network.

- *User Management*: Uses Amazon Cognito User Pool to manage the complete account lifecycle (registration, authentication, password changes, and session revocation).

### 4. Technical Implementation  
*Deployment Phases*  
1. Infrastructure Initialization: Deploy the environment, configure the domain, and set up CI/CD through AWS Amplify.

2. Connection & Authentication: Configure Amazon Cognito, deploy API Gateway (HTTP API) for REST APIs, and API Gateway (WebSocket API) for real-time communication.

3. Game Engine Development: Program the core Lambda functions (Start Match, Process Action, End Match) to handle card game logic.

4. Post-Match Processing: Configure Amazon SQS and the Lambda Post Match Worker to asynchronously process Rank updates, EXP, Match History, and Game Logs, ensuring the Game Engine always responds quickly.

5. Testing & Optimization: Monitor with X-Ray and CloudWatch, optimize security with WAF/IAM, and conduct load testing (Stress Test).

*Technical Requirements*  
- *System Infrastructure*: AWS Amplify (Hosting & CI/CD), GitHub, Route 53 (domain), IAM, and VPC for deployment, management, and security.  

- *Game Platform*: Amazon Cognito (JWT authentication), Amazon API Gateway (HTTP API) for REST APIs, Amazon API Gateway (WebSocket API) for real-time connections, AWS Lambda (Cancel Match, Start Match, Process Game Engine, Save Deck, Handle Timeout, End Match, Post Match Worker), Amazon SQS for asynchronous processing, DynamoDB storing GameState, UserProfile, Connections, MatchHistory, and GameLogs, CloudWatch and X-Ray (monitoring), AWS WAF (security). The frontend uses React with a WebSocket connection to synchronize match state in real time.

### 5. Roadmap & Milestones  
- *Pre-Internship (Month 0)*: 1 month of planning.
    - Month 1: Study and learn AWS services, practice Lab exercises to consolidate knowledge.  
    - Month 2: Design and refine the architecture.  
    - Month 3: Deploy, test, and launch.  
- *Post-Deployment*: Research and develop additional features. 

### 6. Budget Estimate   

*Infrastructure Costs*  
- AWS Amplify: $0.00 – $0.02/month (Hosting ~500 MB, CI/CD a few deployments, within the 12-month Free Tier).

- Amazon Route 53: $0.50/month (1 Hosted Zone, excluding domain registration fees).

- Amazon Cognito: $0.00/month (≤ 10 users, within the Free Tier MAU).

- Amazon API Gateway (WebSocket): $0.00 – $0.02/month (~20,000 WebSocket messages and low connection time, within Free Tier).

- AWS Lambda: $0.00/month (~50,000 requests, 512 MB, below the Free Tier of 1 million requests and 400,000 GB-s).

- Amazon DynamoDB: $0.00/month (≈1 GB of data, using Provisioned mode at 25 WCU/RCU to stay within Free Tier).

- Amazon SQS: $0.00/month (~10,000 messages, below the Free Tier).

- Amazon CloudWatch: $0.00/month (≈1 GB of log storage, below the 5 GB free monthly threshold).

- AWS X-Ray: $0.00/month (low trace volume, within Free Tier).

- AWS KMS: $0.00/month (using AWS-managed keys).

- AWS Systems Manager Parameter Store: $0.00/month (replaces Secrets Manager to store 1 Secret completely free of charge).

- Internet Data Transfer: $0.00/month (~1 GB Data Transfer Out, below the 100 GB free monthly threshold).

- AWS WAF: $0.00/month (if disabled) / ≥ $5.00/month (if 1 Web ACL is enabled to filter malicious requests).

*Total Cost*:
- MVP Infrastructure (without WAF): approximately $0.54/month (~$6.50/year).

- MVP Infrastructure (with WAF): approximately $5.54/month (~$66.50/year).    

### 7. Risk Assessment  
*Risk Matrix*  
- Lambda Cold Start causing lag on the first turn: Medium likelihood, medium impact.  

- Player-side WebSocket network disconnection: High likelihood, high impact.  

- Hitting AWS Service Quota limits: Low likelihood, very high impact.

*Mitigation Strategies*  
- Mitigating Lambda Cold Start: Optimize startup time by reducing package size, reusing connections, and only configuring Provisioned Concurrency for real-time Lambda functions (Process Game Engine) when the system experiences high traffic. This approach significantly reduces first-turn latency while still optimizing operational costs.

- Mitigating WebSocket disconnection: Implement an Auto-Reconnect mechanism on the frontend combined with periodic heartbeat pings to detect connection loss. When a player reconnects, API Gateway and Lambda update the new Connection ID in DynamoDB, then re-synchronize the current Game State so the player can continue the match without creating a new session.

- Mitigating AWS Quota limits: Set up CloudWatch Metrics and CloudWatch Alarms to monitor the number of WebSocket connections, Lambda Invocations, and other critical resources. When resources reach approximately 70–80% of the limit, the system sends an email alert so administrators can proactively request a quota increase before it impacts users. 

*Contingency Plan*  
- Resource scaling: When AWS resources approach the Service Quota, administrators request a quota increase and temporarily restrict the creation of new matches, prioritizing resources for ongoing matches to ensure system stability.  

### 8. Expected Outcomes  
- *Technical Improvements*: Successfully build a fully Serverless Game Engine pipeline, replacing continuously maintained servers to reduce costs. Use Amazon SQS to decouple post-match tasks from the real-time processing pipeline, reducing latency, improving scalability, and optimizing operational costs.

- *Long-term Value*: A data platform for game development that can be reused for future projects.