
Book Explorer: Developed system for exploring books online with Node and DevOps concepts

Table of Contents
TABLE OF CONTENTS	1
1. INTRODUCTION	2
2. SETTING UP CONTINUOUS INTEGRATION WITH GITHUB ACTIONS	2
3. CONTAINERIZING THE APPLICATION WITH DOCKER	3
4. DEPLOYING THE APPLICATION TO A CLOUD PROVIDER	5
5. WRITING API TESTS WITH JEST	6
6. MANAGING SECRETS IN EVERY ENVIRONMENT	7
7. CHALLENGES AND FUTURE IMPROVEMENTS	7
8. CONCLUSION	8
9. REFERENCES	8

1. Introduction

This system will develop to explore the vast information about books which helps users to find and look up books and their basic details.
Technology Stack: 
- Backend: Node.js with Express
- Database: MySQL
- DevOps: GitHub Actions, Docker
- Cloud: Google Cloud Platform
- Testing: Jest 
- Secret Management: Environment variables and GitHub Secrets
My application developed using Nodejs as backend with Express and MySQL database in which we worked as team on this along with my 2 other team members, we setup basic API flows and test case for that and along with this we setup the Continuous Integration using GitHub actions and Google Cloud Platform

2. Setting Up Continuous Integration with GitHub Actions

Continuous Integration (CI) has been done using GitHub Actions which build and test the code on each push to repository that ensures error free code and ultimately it will provide code quality and smooth and scalable application.
This functionality implies strict rules on the working repository, especially when a team of multiple people are working on same project and such rules stopping individuals to pushing error code or such code that is not normalized.
The major advantages of CI are as follows:
- Automated testing
- Early error detection
- Smooth team collaboration
- Easy deployment to server 

Following is our actual GitHub Actions workflow for CI:

```yaml
name: CI

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  main:
    runs-on: ubuntu-latest

    services:
      mysql:
        image: mysql:8.0
        env:
          MYSQL_ROOT_PASSWORD: ${{ secrets.MYSQL_DB_PASSWORD }}
          MYSQL_DATABASE: bookstore_test
        options:
          --health-cmd="mysqladmin ping"
          --health-interval=10s
          --health-timeout=5s
          --health-retries=5
        ports:
          - 3306:3306

    steps:
    - uses: actions/checkout@v4

    - name: Set up Node.js 18.20.5
      uses: actions/setup-node@v4
      with:
        node-version: '18.20.5'
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Run Linter
      run: npm run lint

    - name: Run Jest tests
      env:
        DATABASE_HOST: 127.0.0.1
        DATABASE_PORT: 3306
        DATABASE_USER: root
        DATABASE_PASSWORD: ${{ secrets.MYSQL_DB_PASSWORD }}
        DATABASE_DATABASE: bookstore_test
      run: npm test 
```

3. Containerizing the Application with Docker

Docker was used to package the backend Express application, allowing it to run consistently across environments. This ensured that dependencies, configurations, and system libraries are isolated and reproducible.
It also reduces the version conflicts, dependencies issue from one platform to another

Sample Dockerfile:

```yaml
name: Docker Build
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v1
      - name: Build Docker image
        run: docker build -t bookstore-app . 
```

Pros:
•	Isolated Environment which helps to reduce risks of version conflictions  
•	Easy to scale and deploy  
•	Automatically install all the required files and packages  

Cons:
•	Needs to learn docker concepts  
•	Sometimes image size is bit larger.  
•	Its not ideal for small applications  

4. Deploying the Application to a Cloud Provider

We deployed the application on Google Cloud Run. It allowed us to run the containerized backend in a fully managed serverless environment, it offers scalability, reliability and UI to manage and look up the logs.

Advantages:
•	Cloud providers adjust resources by monitoring usage of your app automatically.  
•	Cost efficiently as you only pay for what you use and also scale resources down when server is ideal.  
•	Cloud network spread globally so availability will increase significantly.  
•	Data is more secure and managed as cloud providers give facilities of auto backups.  

Disadvantages:
•	Sometimes due to location of server latency may low  
•	The concept of clouds is a bit complex and need to learn new concepts.  
•	Sometimes it is hard to make it cost-effective as different cloud providers provide different features at different cost.  

5. Writing API Tests with Jest

We used Jest to write unit and integration tests for our backend API endpoints. Automated testing helped ensure the application behaved as expected during development and deployment.

Example Test:

```javascript
require('dotenv').config();

const request = require('supertest');
const express = require('express');
const mysql = require('mysql2');
const defineBookRoutes = require('../bookRoutes');
const fs = require('fs');
const app = express();
app.use(express.json());

describe('Books API', () => {
  it('should fetch all books', async () => {
    const res = await request(app).get('/books');
    expect(res.statusCode).toEqual(200);
    expect(Array.isArray(res.body)).toBe(true);
    expect(res.body.map(book => book.title)).toStrictEqual(["Book Title 1", "Book Title 2", "Book Title 3"])
  });

  it('should fetch correct book by ID', async () => {
    const res = await request(app).get('/books/1');
    expect(res.statusCode).toBe(200);
    expect(res.body.title).toBe("Book Title 1")
    expect(res.body.author).toBe("Author 1")
    expect(new Date(res.body.publish_date).toISOString().slice(0, 10)).toBe("2025-01-01");
  });
});
```

6. Managing Secrets in Every Environment

In local environment secret variables such as DB passwords etc., can be stored in .env file while on GitHub it will be stored in GitHub secrets by following ways:

In GitHub Actions:  
`${{ secrets. MYSQL_DB_PASSWORD }}`  

While on the GCP it will be managed using IAM role based access.

7. Challenges and Future Improvements

Challenges:
•	It is first time we are working on DevOps. Everything is new, so it took time to understand and learn new fundamentals.  
•	Creating CI pipelines from the GitHub, I personally face lots of issues  

Solutions:
•	Thanks to our professor Lulu Sheng, she teaches us in very unique ways and creative assignment helps a lot to the concepts of DevOps.  
•	GitHub documentation helps a lot to set up CI in clear and manageable ways.  

Future Improvements:
•	Adding eye catching User Interface  
•	Implementing secure authentication and token-based APIs.  

8. Conclusion

To Sum up, by developing book explorer application it gives me deeper understanding of DevOps practices. I was exposed to many new concepts such as CI/CD automation, testing application with jest, exploring cloud technology and vast advantages of using it.
I personally feels, after working on this project feels confident and gain more advanced knowledge which helps me to hunt for my next job with deeper understanding of DevOps concepts.

9. References

•	https://docs.docker.com/  
•	https://docs.github.com/en/actions  
•	https://cloud.google.com/run  
•	https://jestjs.io/  
