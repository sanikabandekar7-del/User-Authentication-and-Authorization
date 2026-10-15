# User-Authentication-and-Authorization
User Authentication and Authorization: Implemented login functionality and access control to ensure secure access to application features.
Student Management REST API
A beginner-friendly REST API built with Node.js, Express, and MySQL. It allows authenticated users to manage student records and uses JWT authentication and role-based authorization to restrict sensitive actions.
Features
- Create, view, update, and delete student records
- Store student data in MySQL
- Register and log in users
- Hash passwords with bcryptjs
- Authenticate requests with JSON Web Tokens (JWT)
- Restrict student-management routes to authenticated users
- Restrict student deletion to administrators
Technologies Used
- Node.js
- Express.js
- MySQL
- mysql2
- bcryptjs
- jsonwebtoken
- dotenv
- Postman for API testing
Prerequisites
Install the following before starting:
- Node.js
- MySQL Server
- Postman (for testing the API)
Project Setup
1. Open the project folder
Open a terminal in the student-api folder.
2. Install dependencies
If the project already has a package.json, run:
npm install
If you have not installed the required packages yet, run:
npm install express mysql2 bcryptjs jsonwebtoken dotenv
3. Create the MySQL database
Open the MySQL command line and run:
CREATE DATABASE IF NOT EXISTS student_db;
USE student_db;
Create the students table:
CREATE TABLE IF NOT EXISTS students (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    course VARCHAR(100) NOT NULL
);
Create the users table to match the authentication code in this project:
CREATE TABLE IF NOT EXISTS users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    role ENUM('user', 'admin') NOT NULL DEFAULT 'user',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
If you already have a users table, do not run a second CREATE TABLE expecting it to change the existing table. Check its structure with DESCRIBE users; and make sure the column names match the SQL queries in server.js.

4. Configure environment variables
Create a file named .env in the same folder as server.js:
DB_HOST=localhost
DB_USER=root
DB_PASSWORD=your_mysql_password
DB_NAME=student_db
JWT_SECRET=replace_with_a_long_private_random_secret
PORT=3000
Replace your_mysql_password with your local MySQL password. Set JWT_SECRET to a long, private random value. Do not share the real values or commit .env to Git.
Create a .gitignore file containing:
node_modules/
.env
5. Start the server
Run:
node server.js
The terminal should show a message indicating that the MySQL connection succeeded and the server is running at http://localhost:3000.
Keep this terminal open while testing the API.
API Endpoints
Base URL: http://localhost:3000
Authentication
Method	Endpoint	Purpose	Authentication
POST	/api/auth/register	Register a user	Not required
POST	/api/auth/login	Log in and receive a JWT	Not required


Register a user
POST http://localhost:3000/api/auth/register
In Postman, select Body → raw → JSON:
{
  "name": "Test User",
  "email": "testuser@example.com",
  "password": "TestPass123!"
}
A successful registration should return 201 Created. Passwords should be stored as hashes, not plain text.
Log in
POST http://localhost:3000/api/auth/login
{
  "email": "testuser@example.com",
  "password": "TestPass123!"
}
Copy the token from the response. Use it for protected endpoints.
Student management
All /api/students endpoints require a valid JWT. In Postman, open Authorization, select Bearer Token, and paste the token returned by login.
Method	Endpoint	Purpose	Required role
GET	/api/students	Get all students	Authenticated user
GET	/api/students/:id	Get one student by ID	Authenticated user
POST	/api/students	Add a student	Authenticated user
PUT	/api/students/:id	Update a student	Authenticated user
DELETE	/api/students/:id	Delete a student	Admin only


The exact available endpoints depend on the routes implemented in your server.js. If your file does not define one of the routes above, add that route before testing it.
Add a student
POST http://localhost:3000/api/students
Authorization: Bearer Token
Body → raw → JSON:
{
  "name": "Aarav Sharma",
  "course": "Computer Science"
}
Update a student
PUT http://localhost:3000/api/students/1
Authorization: Bearer Token
{
  "name": "Aarav Sharma",
  "course": "Information Technology"
}
Delete a student
DELETE http://localhost:3000/api/students/1
Authorization: Bearer Token using an admin's current token.
A 401 Unauthorized response usually means the token is missing, invalid, or expired. A 403 Forbidden response means the authenticated account does not have the required role.
Make a user an administrator
New registrations normally receive the user role. To promote an account you control, run this in MySQL, changing the email if necessary:
USE student_db;

UPDATE users
SET role = 'admin'
WHERE email = 'testuser@example.com';
Verify the role:
SELECT id, name, email, role
FROM users
WHERE email = 'testuser@example.com';
Log in again after changing the role. The role is included in the JWT, so an existing token may still contain the old role.
Never allow ordinary users to promote themselves to administrator in a real application.
Testing with Postman
1. Start the server with node server.js.
2. Register a user with POST /api/auth/register.
3. Log in with POST /api/auth/login.
4. Copy the returned JWT.
5. For student endpoints, set Authorization → Bearer Token in Postman.
6. Test GET, POST, and PUT student endpoints.
7. To test DELETE, promote your test account to admin in MySQL, log in again, and use the new token.
Use the actual student IDs returned by your database. You can check them in MySQL:
USE student_db;
SELECT * FROM students;
Common Problems
Unknown column 'name' in 'field list'
The users table's columns do not match the registration SQL in server.js. Run:
DESCRIBE users;
Make the table structure and backend query use the same column names. Do not rename or delete columns until you have checked existing data and code.
Registration failed with HTTP 500
Check the terminal running node server.js for the detailed database error. Verify .env, the selected database, and the users table columns.
HTTP 401 Unauthorized
Log in again and use the latest token in Postman's Bearer Token field. Tokens may expire.
HTTP 403 Forbidden
The token may belong to a normal user. If the route requires an admin, verify the account's role in MySQL, then log in again to obtain a token containing the updated role.
MySQL connection error
Confirm that MySQL Server is running and that DB_HOST, DB_USER, DB_PASSWORD, and DB_NAME in .env are correct.
Security Notes
- Never store plain-text passwords; use password hashing.
- Keep .env private and out of version control.
- Use a long, private JWT secret.
- Keep administrator permissions limited to trusted accounts.
- For production, use HTTPS and review validation, error handling, token expiry, and database permissions.
Learning Outcomes
This project demonstrates:
- REST API design with Express
- CRUD operations
- MySQL database integration
- Password hashing
- JWT-based authentication
- Role-based authorization
- API testing with Postman
