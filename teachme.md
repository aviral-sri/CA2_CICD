# Viva Preparation Guide: CI/CD Pipeline Project

This guide breaks down every component of your project to help you confidently answer questions during your viva. 

## 🌟 Overall Project Goal
You built a **Continuous Integration / Continuous Deployment (CI/CD)** pipeline. The goal is to take a simple web application (Node.js API), containerize it (Docker), and completely automate its testing and building process every time new code is pushed (GitHub Actions).

---

## 📁 File-by-File Breakdown

### 1. `index.js` (The Application)
**What it is:** The main backend code using the Express.js framework.
**What it does:**
- It starts a web server on port 3000.
- It creates a single API endpoint: `http://localhost:3000/health`. 
- **Viva Note:** We use a `/health` endpoint specifically so our CI/CD pipeline has a reliable URL to ping it and verify the server doesn't crash on startup.

### 2. `package.json` (The Blueprint)
**What it is:** The configuration file for Node.js projects.
**What it does:**
- It lists the dependencies (libraries) your project needs to function (in this case, just `express`).
- It defines "scripts". We added `"start": "node index.js"`, which provides a standardized way to start the application using `npm start`.

### 3. `Dockerfile` (The Containerizer)
**What it is:** A set of instructions for Docker to build an "image" for your app.
**What it does line-by-line:**
- `FROM node:18`: Grabs a base operating system that already has Node v18 installed.
- `WORKDIR /app`: Creates a folder called `/app` inside the container and uses it as the home folder.
- `COPY package*.json ./`: Copies `package.json` first.
- `RUN npm install`: Installs the dependencies inside the container.
- `COPY . .`: Copies all your application code (`index.js`) into the container.
- `EXPOSE 3000`: Documents that this container listens on port 3000.
- `CMD ["npm", "start"]`: The command to actually run when a container starts up from this image.

### 4. `.github/workflows/ci-cd.yml` (The Automation Engine)
**What it is:** The configuration file for GitHub Actions that defines your pipeline.
**What it does:**
- `on: push: branches: - main`: This dictates that the pipeline ONLY ruins when someone pushes code to the `main` branch.
- **Jobs / Steps:** It dictates an automated checklist for GitHub's servers to execute:
  1. **Checkout code:** Pulls your code into GitHub's runner machine.
  2. **Setup Node:** Installs Node.js on their machine.
  3. **Install dependencies:** Runs `npm install`.
  4. **Run app test (basic):** This is the integration test. It runs `node index.js` in the background (`&`), waits 3 seconds (`sleep 3`) for it to boot up, and then pings `curl http://localhost:3000/health`. If the app failed to start, this step fails and stops the pipeline.
  5. **Build Docker Image:** Runs the `Dockerfile` to verify the container image can be successfully created.

### 5. `.gitignore` (The Filter)
**What it is:** A list of files and folders Git should ignore.
**What it does:** By putting `node_modules` inside, we ensure we don't accidentally push hundreds of heavy dependency files to GitHub. GitHub only gets the actual code you wrote, and generates the `node_modules` itself when it runs `npm install`.

---

## ❓ Frequently Asked Viva Questions

**Q: Why use Docker at all?**
*Answer:* "Docker ensures that 'it works on my machine' applies everywhere. It bundles the app code, the Node.js runtime, and dependencies into one single package. If the container runs on my laptop, it will run exactly the same way on a production server."

**Q: What is the CI (Continuous Integration) part of your project?**
*Answer:* "The CI part is our GitHub Actions workflow. Every time we push code, it automatically starts an environment, installs dependencies, runs our application, and uses `curl` to perform a health check to verify we didn't push broken code."

**Q: Why does the CI test use `curl /health`?**
*Answer:* "Building the project only verifies there are no syntax errors. Using `curl /health` forces the app to actually boot up and serve network traffic, ensuring it connects properly and hasn't crashed recursively. It's a fundamental integration test."

**Q: What happens if someone pushes code with an error in `index.js`?**
*Answer:* "GitHub Actions will trigger because of the push. The 'Run app test' step will fail because `curl http://localhost:3000/health` will return an error (since the Node app crashed). The pipeline will halt, and GitHub will display a red 'X' next to the commit, notifying the team."
