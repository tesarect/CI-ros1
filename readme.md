# Instructions

## Jenkins Setup & Start

### First session only
```bash
# 1. Start Jenkins (also installs Java + downloads WAR if missing)
cd ~/simulation_ws/src/ros1_ci
bash jenkins-infra/scripts/jenkins_bootstrap.sh

# 2. Install plugins (only needed once — skipped automatically on future runs)
bash jenkins-infra/scripts/install_plugins.sh

# 3. Get your browser URL
jenkins_address

# 4. Get the initial admin password
cat ~/webpage_ws/jenkins/secrets/initialAdminPassword
```
Open the Construct URL in your browser and complete the Jenkins setup wizard.

### Every session after (Jenkins restart)
```bash
# Kill previous Jenkins process first (get PID from jenkins_pid_url.txt)
kill $(cat ~/webpage_ws/jenkins/jenkins.pid)

# Then restart
cd ~/simulation_ws/src/ros1_ci
bash jenkins-infra/scripts/jenkins_bootstrap.sh
```
Jenkins URL for the new session: run `jenkins_address` again (URL changes each session).

### Restart Jenkins mid-session (after config changes)
```bash
kill $(cat ~/webpage_ws/jenkins/jenkins.pid)
cd ~/simulation_ws/src/ros1_ci
bash jenkins-infra/scripts/jenkins_bootstrap.sh
```

---

## build notiec gazebo
```bash
cd ~/simulation_ws/src/
# docker build -f ros1_ci/Dockerfile -t tortoisebot-noetic-gazebo .
docker build -t tortoisebot-noetic-gazebo:latest .
```

## run gazebo through container
Start the container
```bash
xhost +local:docker

docker run --rm \
  --name ci_tortsim \
  -e DISPLAY=$DISPLAY \
  -v /tmp/.X11-unix:/tmp/.X11-unix:rw \
  tortoisebot-noetic-gazebo \
  roslaunch tortoisebot_gazebo tortoisebot_playground.launch
```
start the waypoint action server
```bash
 docker exec -it ci_tortoise_sim bash
root@0c0bd1e11453:/catkin_ws/src# rosrun tortoisebot_waypoints tortoisebot_action_server.py
[INFO] [1781595375.957848, 1118.782000]: Action server started
```

start the fail test case
```bash
$ docker exec -it ci_tortoise_sim bash
root@0c0bd1e11453:/catkin_ws/src# rostest tortoisebot_waypoints waypoints_test.test --reuse-master
... logging to /root/.ros/log/rostest-0c0bd1e11453-853.log
[ROSUNIT] Outputting test results to /root/.ros/test_results/tortoisebot_waypoints/rostest-test_waypoints_test.xml
[Testcase: testtest_waypoints] ... ok
......
```

start the fail test case
```bash
$ docker exec -it ci_tortoise_sim bash
root@0c0bd1e11453:/catkin_ws/src# rostest tortoisebot_waypoints waypoints_test.test --reuse-master
... logging to /root/.ros/log/rostest-0c0bd1e11453-853.log
[ROSUNIT] Outputting test results to /root/.ros/test_results/tortoisebot_waypoints/rostest-test_waypoints_test.xml
[Testcase: testtest_waypoints] ... ok
......
```

## docker push
build locally as
cd ~/simulation_ws/src/ros1_ci
docker build -t tortoisebot-noetic-gazebo:latest .
then tag it and push it
docker tag tortoisebot-noetic-gazebo:latest tesarect/karthikeyanbalasubramanian-cp22:tortoisebot-noetic-gazebo-v1
docker push tesarect/karthikeyanbalasubramanian-cp22:tortoisebot-noetic-gazebo-v1

for later usage just pull it
docker pull tesarect/karthikeyanbalasubramanian-cp22:tortoisebot-noetic-gazebo-v1