
# Jenkins CI/CD Pipeline for Building cURL

This project sets up a CI/CD pipeline using Jenkins and Docker to build and test the [cURL](https://github.com/curl/curl) application from source, and archive the resulting binary and test logs.

## 🛠 Project Overview

### Objectives:
- Run Jenkins master and agent in Docker containers.
- Only the Jenkins master can access the WAN (to pull from GitHub).
- The agent runs in a private LAN.
- Build and test the `curl` project from GitHub.
- Archive the built executable and test results as Jenkins artifacts.

---

## 🐳 Docker Architecture

```
                        +------------------------+
                        |  jenkins/jenkins:lts   |
                        |     (jenkins-master)   |
                        |  - Access to WAN       |
                        |  - Runs Jenkins UI     |
                        |  - Clones GitHub repo  |
                        +-----------+------------+
                                    |
                                    | Private LAN
                                    v
                      +----------------------------+
                      |      jenkins-agent         |
                      | - Custom Ubuntu container  |
                      | - Builds & tests curl      |
                      +----------------------------+
```

---

## 📁 Directory Structure

```
project-root/
│
├── Dockerfile.agent          # Defines Jenkins build agent
├── docker-compose.yml        # Brings up Jenkins master and agent
├── Jenkinsfile               # Jenkins pipeline definition
├── workspace/                # Shared volume for Jenkins builds
```

---

## ⚙️ Setup Instructions

### 1. Enable IPv6 (required for cURL tests)
Edit Docker daemon config:

```bash
sudo vim /etc/docker/daemon.json
```

Add this:

```json
{
  "ipv6": true
}
```

Then restart Docker:

```bash
sudo systemctl reload docker
```

---

### 2. Prepare workspace directory

```bash
mkdir -p workspace
sudo chown 1000:1000 workspace
```

This ensures both Jenkins master and agent (running as UID 1000) can write to the shared volume.

---

### 3. Start the containers

```bash
docker-compose up --build -d
```

---

### 4. Set up Jenkins

1. Open Jenkins at `http://localhost:8080`.
2. Follow the instructions to unlock Jenkins and create an admin user.
3. Install suggested plugins.
4. Go to **Manage Jenkins → Nodes & Clouds → New Node**.
5. Create a new node named `docker-agent`:
   - Type: "Permanent Agent"
   - Remote root directory: `/workspace`
   - **Label the agent with "docker-agent"**
   - Usage: "Use this node as much as possible"
   - Launch method: "Launch agent by connecting it to the master"

---

### 5. Connect the agent

In a terminal, execute this to connect the `jenkins-agent` container to `jenkins-master`:

```bash
- Download the agent.jar file to the workspace
docker exec -it jenkins-agent nohup curl -sO http://jenkins-master:8080/jnlpJars/agent.jar > /dev/null 2>&1 &

- Connect the agent to master
docker exec -u jenkins jenkins-agent bash -c 'nohup java -jar agent.jar \
  -url http://jenkins-master:8080/ \
  -secret <YOUR-AGENT-SECRET> \
  -name "docker-agent" \
  -webSocket \
  -workDir "/workspace" > /workspace/agent.log 2>&1 &'
```

> Replace `<your-agent-secret>` with the actual secret key shown in the Jenkins UI for the agent.

---

### 6. Create the Jenkins Pipeline

1. In Jenkins UI, click **New Item** → Choose **Pipeline**.
2. Name it (e.g., `build-curl`) and click OK.
3. Scroll to the Pipeline section.
4. Choose "Pipeline script" and paste the contents of `Jenkinsfile`.
5. Save and click **Build Now**.

---

## ✔️ Pipeline Breakdown

### Stages:
- **Fetch source**: Clones the latest cURL repo (WAN access allowed only for master).
- **Build**: Configures and compiles the code using the agent.
- **Test**: Runs `make test`, capturing the output into `test-results.txt`.
- **Archive**: Archives the compiled `curl` binary and `build-info.txt` as artifacts.

---

## 🧪 Output

After a successful run:
- Artifacts are archived under **Build Artifacts** in Jenkins.
- Includes:
  - `curl` — the compiled executable.
  - `test-results.txt` — the result log of the test run.
  - `build-info.txt` — metadata about the build and version.

---

## 📌 Notes

- Ensure IPv6 is enabled in Docker to pass all tests.
- The agent container uses `sleep infinity` as its default entrypoint so you can connect and configure it manually.
- Communication between Jenkins master and agent is done over a private LAN (`lan-net` in `docker-compose.yml`).

---

## 🧹 Cleanup

To stop and remove the containers:

```bash
docker-compose down
```

To also remove volumes:

```bash
docker-compose down -v
```

---

## 📝 License

MIT — feel free to reuse and modify.# Jenkins CI/CD Pipeline for Building cURL

