# Dockerized JMeter - A Distributed Load Testing Workflow

### Supported Cloud Providers:
- Amazon (https://aws.amazon.com/free/)
- DigitalOcean (https://www.digitalocean.com/join/)

### Images used:
- JMeter Server (https://hub.docker.com/r/hhcordero/docker-jmeter-server) aka "Slave"
- JMeter Client Non-gui (https://hub.docker.com/r/hhcordero/docker-jmeter-client) aka "Master"

### Pre-requisites:
1. Docker and Docker-Machine cli installed on your host (https://docs.docker.com/installation/)
2. You can choose either Amazon or DigitalOcean
	
	Amazon:
  	* Access Key
  	* Secret Key
  	* Region
  	* VPC ID
  	* Subnet ID
  	* Zone
  	* AMI
	
	DigitalOcean:
  	* Access Token
  	
3. JMeter test plan on your host (http://jmeter.apache.org/usermanual/build-web-test-plan.html)

### Steps:
1. Provision machine for JMeter Master (Non-gui mode)
	
	For AWS:
	```
	$ ./launch_jmeter_master_aws
	```
	
	For DigitalOcean:
	```
	$ ./launch_jmeter_master_do
	```
	
	For local machine:
	```
	$ ./launch_jmeter_master_local
	```
	
2. Provision machine for JMeter Slave (Server mode)
	
	Optional parameter - indicate number of servers to provision, value between 1 to 20. Default to 1 if no parameter is given.
	
	For AWS:
	```
	$ ./launch_jmeter_slave_aws 2
	```

	For DigitalOcean:
	```
	$ ./launch_jmeter_slave_do 2
	```
	
3. Copy test plan (jmx file) to jmeter-master machine inside /load_tests
	
	Connect to jmeter-master machine and create /load_tests directory
	```
	$ eval "$(docker-machine env --swarm jmeter-master)"
	$ docker-machine ssh jmeter-master
	```
	
	For AWS:
	```
	$ sudo mkdir /load_tests && sudo chown ubuntu:ubuntu /load_tests
	```
	
	For DigitalOcean:
	```
	$ mkdir /load_tests
	```
	
	For local machine:
	```
	$ sudo mkdir /load_tests && sudo chown docker:docker /load_tests
	```
	
	Then go back to host 
	```
	$ exit
	```
	
	Copy test plan from host to jmeter-master machine
	
	Syntax:
	```
	docker-machine scp [path to test directory] [machine name]:/load_tests
	```
	
	Example:
	```
	$ docker-machine scp -r /home/user/docker-jmeter-master/load_tests/my_test jmeter-master:/load_tests
	```
	
	Then go back to host 
	```
	$ exit
	```
4. Run JMeter load test - execute the following commands, replace values if necessary before executing
	
	Get JMeter master machine ip
	```	
	$ IP=$(docker-machine ip jmeter-master)
	```
	
	Replace the value with the output from ./launch_jmeter_slave - must be comma separated jmeter slave ip addresses
	```
	$ REMOTE_HOSTS=""
	```

	Parent directory for the test plan
	```
	$ TEST_DIR="my_test"
	```
	
	Test plan without file extension
	```
	$ TEST_PLAN="test-plan"
	```
	
	Run the JMeter master non-gui and perform load test
	```
	$ docker run \
	    --detach \
	    --publish 1099:1099 \
	    --volume /load_tests/$TEST_DIR:/load_tests/$TEST_DIR \
	    --env TEST_DIR=$TEST_DIR \
	    --env TEST_PLAN=$TEST_PLAN \
	    --env IP=$IP \
	    --env REMOTE_HOSTS=$REMOTE_HOSTS \
	    --env constraint:type==master \
	    hhcordero/docker-jmeter-client
	```
	
	To monitor output, follow the logs:
	
	Syntax:
	```
	docker logs -f [container name]
	```
	
	Example:
	```
	$ docker logs -f jmeter-master/tender_feynman
	```

5. Save the result, look for the .jtl file inside jmeter-master /load_tests/$TEST_DIR
	
	Syntax:
	```
	docker-machine scp [machine name]:/load_tests/[test dir]/[test plan result] [path to test directory]
	```
	
	Example:
	```
	$ docker-machine scp jmeter-master:/load_tests/${TEST_DIR}/${TEST_PLAN}.jtl /home/user/docker-jmeter-master/load_tests/my_test/.
	```