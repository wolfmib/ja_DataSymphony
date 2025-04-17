 ## Demo: Deploying EC2 with PostgreSQL & Token-Based API
In this demo, I walk through a full-stack deployment setup on AWS:

-----



✅ Showcase Features
- Created an EC2 instance via the AWS Console

- Installed PostgreSQL directly on the EC2 instance

- Connected to the EC2 server using VSCode Remote SSH

- Created a secure database role (hello_user) and granted access to a custom DB (hello_db)

- Deployed a Python FastAPI server inside the EC2 instance

- Configured security groups to allow API access

- Implemented token-based authentication

- Backend API queries PostgreSQL using the hello_user role

- Returns a JSON response with data from hello_db
