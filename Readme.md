---------------
* Create a Jenkins pipeline which deploys spring petclinic application into some linux machine
* Suggestions:
* Run springboot as a linux daemon
* Use git flow as branching strategy
* Create a jenkins job for merging pull requests into develop branch
* Fork the spring petclinic into your account
* Have Jenkinsfile in your branches
-------------------
* Build and Deploy of spring petclinic manual steps
=========================
git clone https://github.com/spring-projects/spring-petclinic.git
cd spring-petclinic
./mvnw package
java -jar target/*.jar

You can then access petclinic at http://localhost:8080/

![preview](spc1.png)
* Create a Jenkins pipeline which deploys spring petclinic application into some linux machine

* Jenkins pipeline to push the artifactory into S3  bucket 
```
pipeline {
  agent { node { label 'node_all' } }
  stages {
      stage ('git') {
        steps {
          git url: 'https://github.com/spring-projects/spring-petclinic.git',
          branch: 'main'
      }
      }
      stage ('build') {
        steps {
          sh './mvnw package'
      }
    }
      stage ('download the package') {
        steps {
          sh 'sudo cp ${WORKSPACE}/target/spring-petclinic-3.0.0-SNAPSHOT.jar /tmp/archive'
          }
    }
      stage ('upload to s3') {
        steps {
          sh 'aws s3 cp /tmp/archive/spring-petclinic-3.0.0-SNAPSHOT.jar s3://sunghoon123/site/'
        }
      }
    }
}

```
To deploy the  jar file we need ansible to be installed on the node

Ansible playbook to deploy the application 

```
---

- name: spc deployment
  hosts: localhost
  become: yes
  tasks:
    - name: install java
      ansible.builtin.apt:
        name: openjdk-17-jdk
        state: present
        update_cache: true
    - name: edit the service file
      ansible.builtin.copy:
        dest: /etc/systemd/system/spc.service
        content: |
         [Unit]
         Description=Manage Java service
         [Service]
         WorkingDirectory=/home/ubuntu/build/target
         ExecStart=java -jar spring-petclinic-3.0.0-SNAPSHOT.jar
         Type=simple
         Restart=on-failure
         RestartSec=10
         [Install]
         WantedBy=multi-user.target
        mode: '777'
    - name: Start service httpd, if not running
      service:
        enabled: true
        name: spc.service
        state: "started"
    - name: daemon reload
      ansible.builtin.systemd:
        name: spc.service
        daemon_reload: true
        enabled: true
        state: "restarted"
```
Jenkins pipeline to deploy the artifact 

```
pipeline {
  agent { node { label 'node_all' } }
  stages {
      stage ('git') {
        steps {
          git url: 'https://github.com/spring-projects/spring-petclinic.git',
          branch: 'main'
      }
      }
      stage ('build') {
        steps {
          sh './mvnw package'
      }
    }
      stage ('download the package') {
        steps {
          sh 'sudo cp ${WORKSPACE}/target/spring-petclinic-3.0.0-SNAPSHOT.jar /tmp/archive'
          }
    }
      stage ('upload to s3') {
        steps {
          sh 'aws s3 cp /tmp/archive/spring-petclinic-3.0.0-SNAPSHOT.jar s3://sunghoon123/site/'
        }
      }
      stage ('deploy the artifact') {
        steps {
            sh ' ansible-playbook -i hosts spc.yaml'
        }
      }
    }
}
```
Applied Suggestions
-------------------
* Use git flow as branching strategy:
* For using Git Flow strtegy we need to create multiple branches  , for each branch we create seperate Jenkinsfile .
[referhere](https://github.com/lakshmi164585/spring-petclinic.git)

![preview](spc2.png)
To run all the jenkinsfile at a time  we need multibranch pipeline strategy .
```
screenshot of branch
```
4.  Create a jenkins job for merging pull requests into develop branch

For this we need to fork the repository into another account and we add changes to the forked repository then we create the pull request for this we create a Jenkins job in free style




