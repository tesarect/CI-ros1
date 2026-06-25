# ROS1 CI — Checkpoint 24 Task 1

## Overview
Jenkins CI pipeline that triggers on GitHub Pull Requests, builds a Docker image with ROS Noetic + TortoiseBot, runs Gazebo simulation + waypoints tests inside the container, and reports results back to GitHub.

---

## Repository Structure

| File | Purpose |
|---|---|
| `Dockerfile` | ROS Noetic + Gazebo + waypoints image |
| `Jenkinsfile` | Pipeline: build image → run Gazebo → run tests |
| `jenkins-infra/scripts/jenkins_bootstrap.sh` | Installs + starts Jenkins each session |
| `jenkins-infra/scripts/install_plugins.sh` | Installs Jenkins plugins (run once) |
| `jenkins-infra/jenkins/plugins.txt` | Pinned plugin list for Jenkins 2.504.3 |

---

## Docker Image

DockerHub: `tesarect/karthikeyanbalasubramanian-cp22:tortoisebot-noetic-gazebo-v1`

```bash
# Build
cd ~/simulation_ws/src/ros1_ci
docker build -t tortoisebot-noetic-gazebo:latest .

# Tag and push
docker tag tortoisebot-noetic-gazebo:latest tesarect/karthikeyanbalasubramanian-cp22:tortoisebot-noetic-gazebo-v1
docker push tesarect/karthikeyanbalasubramanian-cp22:tortoisebot-noetic-gazebo-v1
```

---

## Switching Between Pass and Fail Test Cases

The test case is controlled by one file:
`tortoisebot/tortoisebot_waypoints/test/waypoints_test.test`

**To run the PASS test** (robot reaches goal — build succeeds):
```xml
<test test-name="test_waypoints" pkg="tortoisebot_waypoints" type="test_waypoints_reached.py" />
<!-- <test test-name="test_waypoints" pkg="tortoisebot_waypoints" type="test_waypoints_fail.py" /> -->
```

**To run the FAIL test** (robot can't reach goal in 2s — build fails intentionally):
```xml
<!-- <test test-name="test_waypoints" pkg="tortoisebot_waypoints" type="test_waypoints_reached.py" /> -->
<test test-name="test_waypoints" pkg="tortoisebot_waypoints" type="test_waypoints_fail.py" />
```

After editing the file, commit and push, then open a new PR to trigger the pipeline.

> Currently set to: **PASS**

---

## Instructions (For Evaluation)

### Prerequisites
- The Construct cloud machine with ROS Noetic workspace at `~/simulation_ws`
- Docker installed
- GitHub repo `ros1_ci` with this code

### Step 1 — Start Jenkins

```bash
cd ~/simulation_ws/src/ros1_ci
bash jenkins-infra/scripts/jenkins_bootstrap.sh
```

Get the browser URL:
```bash
jenkins_address
```

Open the URL in your browser and log in with the admin credentials.

### Step 2 — Chnages to switch between pass and fail case

Make the changes on the main (`.test` file) and push it to construct or directly on the constructs main.

Once the pass or fail is switched based on the scenario, make a dummy commit on the branch to trigger the pipeline
```bash
git checkout jenkins-trigger
git commit --allow-empty -m "trigger fail case"
git push origin jenkins-trigger
```

### Step 2 — Trigger a Pull Request

On GitHub → `ros1_ci` repo → **Add file → Create new file** on a new branch → open a Pull Request against master.

Jenkins will trigger automatically within 1 minute.

### Step 3 — Verify Build

- Open Jenkins in browser → `ros1-ci` pipeline → click the PR build
- **Console Output** shows Gazebo launching and waypoints test running
- Build result: **SUCCESS/FAILURE**

---