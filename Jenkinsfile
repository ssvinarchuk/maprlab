cd /root/maprlab

# Destroy old cluster if needed
sudo docker-compose down

# Up cluster nodes
sudo docker-compose up -d --scale node=2
sudo docker rename maprlab_node_1 node1.cluster.com
sudo docker rename maprlab_node_2 node2.cluster.com

# Change hostnames
sudo docker exec -i node1.cluster.com bash -c 'cp /etc/hosts ~/hosts.new ; sed -i 's/'$(hostname)'/node1.cluster.com/g'  ~/hosts.new ; cp -f ~/hosts.new /etc/hosts'
sudo docker exec -i node1.cluster.com bash -c 'cp /etc/hostname ~/hostname.new ; sed -i 's/'$(hostname)'/node1.cluster.com/g'  ~/hostname.new ; cp -f ~/hostname.new /etc/hostname'
sudo docker exec -i node1.cluster.com hostname node1.cluster.com

sudo docker exec -i node2.cluster.com bash -c 'cp /etc/hosts ~/hosts.new ; sed -i 's/'$(hostname)'/node2.cluster.com/g'  ~/hosts.new ; cp -f ~/hosts.new /etc/hosts'
sudo docker exec -i node2.cluster.com bash -c 'cp /etc/hostname ~/hostname.new ; sed -i 's/'$(hostname)'/node2.cluster.com/g'  ~/hostname.new ; cp -f ~/hostname.new /etc/hostname'
sudo docker exec -i node2.cluster.com hostname node2.cluster.com

sleep 30s

# Change repo to current version
sudo docker exec -i --user mapr maprlab_maprinstaller_1 sed -i '/repo_core_url/c\   \"repo_core_url\" : \"http://cv:Artifactory4CV!@34.197.167.22/artifactory/prestage/releases-dev\",' /opt/mapr/installer/data/properties.json
sudo docker exec -i --user mapr maprlab_maprinstaller_1 sed -i '/repo_eco_url/c\   \"repo_eco_url\" : \"http://cv:Artifactory4CV!@34.197.167.22/artifactory/prestage/releases-dev\",' /opt/mapr/installer/data/properties.json
sudo docker exec -i --user mapr maprlab_maprinstaller_1 sudo service mapr-installer restart

sleep 30s

# Run installation via Installer CLI
sudo docker exec -i --user mapr maprlab_maprinstaller_1 ./opt/mapr/installer/bin/mapr-installer-cli install -nv -t /opt/mapr/installer/docker/conf/installer_stanza.yaml -f

sleep 30s

# Create mapr ticket
sudo docker exec -i --user mapr node1.cluster.com bash -c 'echo -e "mapr\n" | maprlogin password'
sudo docker exec -i --user mapr node2.cluster.com bash -c 'echo -e "mapr\n" | maprlogin password'

# Reduce MFS memory consumption
sudo docker exec -i node1.cluster.com bash -c 'sed -i 's/mfs.heapsize.maxpercent=85/mfs.heapsize.maxpercent=50/g' /opt/mapr/conf/warden.conf'
sudo docker exec -i node1.cluster.com bash -c 'sed -i 's/mfs.heapsize.percent=35/mfs.heapsize.percent=20/g' /opt/mapr/conf/warden.conf'

sudo docker exec -i node2.cluster.com bash -c 'sed -i 's/mfs.heapsize.maxpercent=85/mfs.heapsize.maxpercent=50/g' /opt/mapr/conf/warden.conf'
sudo docker exec -i node2.cluster.com bash -c 'sed -i 's/mfs.heapsize.percent=35/mfs.heapsize.percent=20/g' /opt/mapr/conf/warden.conf'

# Uncomment Linux resources limit (Added and commented by Installer for some reason see IN-2014)
sudo docker exec -i node1.cluster.com bash -c 'sed -i 's/#mapr/mapr/g' /etc/security/limits.conf'
sudo docker exec -i node2.cluster.com bash -c 'sed -i 's/#mapr/mapr/g' /etc/security/limits.conf'

# Restart all services after MFS and Linux properties changing
sudo docker exec -i node1.cluster.com service mapr-warden stop
sudo docker exec -i node1.cluster.com service mapr-zookeeper stop
sudo docker exec -i node1.cluster.com service mapr-zookeeper start
sudo docker exec -i node1.cluster.com service mapr-warden start

sudo docker exec -i node2.cluster.com service mapr-warden restart

# Run tests
TEST_RES_FILE_PATH_MAPREDUCE_SMOKE="/home/mapr/private-qa/new-ats/mapreduce/surefire-reports/smoke/mapreduce-tests/testng-results.xml"
TEST_RES_FILE_PATH_MAPREDUCE_FUNCTIONAL="/home/mapr/private-qa/new-ats/mapreduce/surefire-reports/functional/mapreduce-tests/testng-results.xml"

TEST_RES_FILE_PATH_HADOOP_SMOKE="/home/mapr/private-qa/new-ats/hadoop2/surefire-reports/smoke/hadoop2-tests/testng-results.xml"
TEST_RES_FILE_PATH_HADOOP_FUNCTIONAL="/home/mapr/private-qa/new-ats/hadoop2/surefire-reports/functional/hadoop2-tests/testng-results.xml"
TEST_RES_FILE_PATH_HADOOP_BUGS="/home/mapr/private-qa/new-ats/hadoop2/surefire-reports/bugs/hadoop2-tests/testng-results.xml"


JOB_TEST_RESULT_PATH="/var/lib/jenkins/workspace/Hadoop_test/test_results"
sudo docker exec -i --user mapr node1.cluster.com git clone -b $testBranch --depth 1 --single-branch https:\/\/o-shevchenko:maprgithub1@github.com\/mapr\/private-qa.git /home/mapr/private-qa
sudo docker exec -i --user mapr node1.cluster.com bash -c "cd /home/mapr/private-qa/new-ats/ ; chmod +x hadoop.sh ; ./hadoop.sh $testBranch ; mvn clean install -fae"

sudo docker exec -i --user mapr node1.cluster.com bash -c 'cd /home/mapr/private-qa/new-ats/mapreduce/mapreduce-tests/ && mvn test -Psmoke -DfailIfNoTests=false -Dmaven.test.failure.ignore=true'
#sudo docker exec -i --user mapr node1.cluster.com bash -c 'cd /home/mapr/private-qa/new-ats/mapreduce/mapreduce-tests/ && mvn test -Pfunctional -DfailIfNoTests=false -Dmaven.test.failure.ignore=true'

sudo docker exec -i --user mapr node1.cluster.com bash -c 'cd /home/mapr/private-qa/new-ats/hadoop2/hadoop2-tests/ && mvn test -Psmoke -DfailIfNoTests=false -Dmaven.test.failure.ignore=true'
sudo docker exec -i --user mapr node1.cluster.com bash -c 'cd /home/mapr/private-qa/new-ats/hadoop2/hadoop2-tests/ && mvn test -Pfunctional -DfailIfNoTests=false -Dmaven.test.failure.ignore=true'
sudo docker exec -i --user mapr node1.cluster.com bash -c 'cd /home/mapr/private-qa/new-ats/hadoop2/hadoop2-tests/ && mvn test -Pbugs -DfailIfNoTests=false -Dmaven.test.failure.ignore=true'


# Copy test result from Docker container
sudo mkdir $JOB_TEST_RESULT_PATH
sudo docker cp node1.cluster.com:$TEST_RES_FILE_PATH_MAPREDUCE_SMOKE $JOB_TEST_RESULT_PATH/mapreduce_smoke_testng-results.xml
#sudo docker cp node1.cluster.com:$TEST_RES_FILE_PATH_MAPREDUCE_FUNCTIONAL $JOB_TEST_RESULT_PATH/mapreduce_functional_testng-results.xml

sudo docker cp node1.cluster.com:$TEST_RES_FILE_PATH_HADOOP_SMOKE $JOB_TEST_RESULT_PATH/hadoop_smoke_testng-results.xml
sudo docker cp node1.cluster.com:$TEST_RES_FILE_PATH_HADOOP_FUNCTIONAL $JOB_TEST_RESULT_PATH/hadoop_functional_testng-results.xml
sudo docker cp node1.cluster.com:$TEST_RES_FILE_PATH_HADOOP_BUGS $JOB_TEST_RESULT_PATH/hadoop_bugs_testng-results.xml

# Halt cluster
sudo docker-compose stop