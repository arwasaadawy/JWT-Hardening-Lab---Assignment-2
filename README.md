# JWT-Hardening-Lab---Assignment-2
Project Overview:
This project demonstrates the hardening of a vulnerable JWT authentication system by implementing security best practices. 
The lab includes both vulnerable (port 1234) and secure (port 1235) servers to compare security implementations.  
Quick Start Prerequisites Node.js v18 or higher  npm  Git  Installation & Setup bash # Clone the project git clone <repository-url> cd "Lab 2 - JWT" 
# Install dependencies npm install  
# Set up environment configuration copy .env.example .env  
# Initialize database npm run init-db Running the Servers bash # Terminal 1 - Vulnerable Server (Port 1234) node vuln-server.js  
# Terminal 2 - Secure Server (Port 1235)  node secure-server.js 
Environment Configuration .env File Setup Create .env file with the following variables:  env JWT_SECRET=super_secure_jwt_secret_12345 JWT_REFRESH_SECRET=refresh_secret_67890 PORT=1235 ISSUER=localhost:1235 AUDIENCE=localhost:1235 ACCESS_TOKEN_EXPIRES=15m REFRESH_TOKEN_EXPIRES=7d Sample Accounts admin / admin123  alice / alicepass  bob / bobpass  🛡️ Security Implementations 1. Environment Configuration (.env) ✅ All secrets moved from source code to environment variables  
✅ Used dotenv package for configuration management
✅ Created .env.example for documentation  
2. Remove Hard-coded Secrets
✅ Replaced WEAK_SECRET with process.env.JWT_SECRET  
✅ Used cryptographically secure secret generation  
✅ Separate secrets for access and refresh tokens  
3. Token Claims & Verification javascript 
// Enhanced token signing jwt.sign({   sub: username,   role: role }, ACCESS_SECRET, {   algorithm: 'HS256',   expiresIn: process.env.ACCESS_TOKEN_EXPIRES,   issuer: process.env.ISSUER,   audience: process.env.AUDIENCE });  
// Strict token verification jwt.verify(token, ACCESS_SECRET, {   algorithms: ['HS256'],   issuer: process.env.ISSUER,   audience: process.env.AUDIENCE }); 
4. Refresh Token Strategy 
✅ Separate refresh token secret  
✅ HttpOnly cookies for refresh tokens 
✅ Token rotation on refresh  
✅ In-memory token store with expiration 
5. Frontend Preservation 
✅ Maintained original frontend interface 
✅ Minimal client-side modifications  
✅ Same UI for both vulnerable and secure servers  
Attack Demonstrations Attack 1: 
Token Forgery Vulnerable Server (1234):  javascript :
// Successfully authenticates with weak tokens POST http://localhost:1234/vuln-login Response: 200 OK with JWT token Secure Server (1235):  javascript // Rejects forged tokens POST http://localhost:1235/admin Authorization: Bearer <stolen_token> Response: 401 Unauthorized - "Invalid token" Attack 2: Token Theft Simulation Demonstrated token exposure in localStorage  Showed token visibility in network traffic  Proved token reuse prevention  
Wireshark Traffic Analysis Capture Configuration Interface: 
Adapter for loopback traffic capture  Filter: tcp.port == 1234 or tcp.port == 1235  Protocol: HTTP  Observations text GET /admin HTTP/1.1 Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...  HTTP/1.1 401 Unauthorized {"error": "Invalid token"} Key Findings JWT tokens clearly visible in HTTP headers  401 responses evident from secure server  Unencrypted traffic demonstrates need for HTTPS  
Bonus Features 
B. Helmet.js Security Headers javascript const helmet = require('helmet'); app.use(helmet()); Implemented Headers:  Content-Security-Policy - XSS protection  X-Frame-Options: DENY - Clickjacking protection  Strict-Transport-Security - HTTPS enforcement  X-Content-Type-Options: nosniff - MIME sniffing prevention  Referrer-Policy - Referrer information control  
C. Rate Limiting javascript const loginLimiter = rateLimit({   windowMs: 15 * 60 * 1000, // 15 minutes   max: 5, // 5 attempts per window   message: {      error: 'Too many login attempts, please try again after 15 minutes'    } }); app.use('/login', loginLimiter); Behavior:  1-5 attempts: 401 Unauthorized  6th attempt: 429 Too Many Requests
