---
title: "Book Explorer: Building a DevOps-Enabled Book Discovery Platform with Node.js"
description: "Explore how we developed a full-stack book search platform using Node.js, MySQL, Docker, GitHub Actions, and Google Cloud."
date: 2025-04-18
tags: [devops, nodejs, express, mysql, docker, github-actions, cloud]
---

# Book Explorer: Building a DevOps-Enabled Book Discovery Platform with Node.js

## Table of Contents
1. [Introduction](#1-introduction)  
2. [Setting Up Continuous Integration with GitHub Actions](#2-setting-up-continuous-integration-with-github-actions)  
3. [Containerizing the Application with Docker](#3-containerizing-the-application-with-docker)  
4. [Deploying the Application to a Cloud Provider](#4-deploying-the-application-to-a-cloud-provider)  
5. [Writing API Tests with Jest](#5-writing-api-tests-with-jest)  
6. [Managing Secrets in Every Environment](#6-managing-secrets-in-every-environment)  
7. [Challenges and Future Improvements](#7-challenges-and-future-improvements)  
8. [Conclusion](#8-conclusion)  
9. [References](#9-references)  

## 1. Introduction

Book Explorer is a system developed to help users explore books and their details online. It uses a Node.js backend with Express, connected to a MySQL database. Our team focused on API development, testing, CI with GitHub Actions, containerization via Docker, and cloud deployment.

### Technology Stack
- Backend: Node.js with Express  
- Database: MySQL  
- DevOps: GitHub Actions, Docker  
- Cloud: Google Cloud Platform  
- Testing: Jest  
- Secret Management: Environment variables and GitHub Secrets  

## 2. Setting Up Continuous Integration with GitHub Actions

GitHub Actions was used for CI. It builds and tests the app on every push to `main`, ensuring smooth collaboration and preventing errors.

### Benefits
- Automated testing  
- Early bug detection  
- Standardized code quality  
- Simple deployment to cloud  

### Sample Workflow
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
        options: --health-cmd="mysqladmin ping" --health-interval=10s --health-timeout=5s --health-retries=5
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

## 3. Containerizing the Application with Docker

Docker packages our backend to run in isolated environments, avoiding version conflicts.

### Docker Workflow (GitHub Action)
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

### Pros:
- Isolated environments  
- Easy scalability  
- Package management is streamlined  

### Cons:
- Steeper learning curve  
- Large image size  
- Overkill for small apps  

## 4. Deploying the Application to a Cloud Provider

The app was deployed using **Google Cloud Run**, a fully managed container hosting platform.

### Advantages:
- Auto resource scaling  
- Cost-efficiency  
- High availability and global presence  
- Built-in data backups and security

### Disadvantages:
- Steep learning curve for cloud IAM roles  
- Slight latency in remote regions  
- Cost optimization may be challenging  

## 5. Writing API Tests with Jest

We used Jest for writing unit and integration tests for the API endpoints.

### Sample Test Code
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
    expect(res.body.map(book => book.title)).toStrictEqual(["Book Title 1", "Book Title 2", "Book Title 3"]);
  });

  it('should fetch correct book by ID', async () => {
    const res = await request(app).get('/books/1');
    expect(res.statusCode).toBe(200);
    expect(res.body.title).toBe("Book Title 1");
    expect(res.body.author).toBe("Author 1");
    expect(new Date(res.body.publish_date).toISOString().slice(0, 10)).toBe("2025-01-01");
  });
});
```

## 6. Managing Secrets in Every Environment

### Local Environment
- Stored in `.env` file (e.g., `MYSQL_DB_PASSWORD=yourpassword`)

### GitHub Actions
- Stored in repository secrets
```yaml
${{ secrets.MYSQL_DB_PASSWORD }}
```

### Cloud Run
- IAM-based secret access configuration

## 7. Challenges and Future Improvements

### Challenges
- First exposure to CI/CD concepts  
- CI pipeline debugging took effort  

### Solutions
- Helpful instruction and guidance from our professor  
- Used GitHub Docs and DevOps best practices  

### Future Improvements
- Enhanced UI/UX  
- Add secure authentication (JWT)  
- Implement personalized book lists  

## 8. Conclusion

Building the Book Explorer app deepened our understanding of DevOps, cloud technologies, and modern software deployment workflows. From CI/CD pipelines to testing and cloud deployment, this project gave us real-world experience and skills to use in our future careers.

## 9. References

- https://docs.docker.com/  
- https://docs.github.com/en/actions  
- https://cloud.google.com/run  
- https://jestjs.io/  
