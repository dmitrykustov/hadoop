## Required packages for Ubuntu 22.04 LTS Desktop

* Open JDK 1.8
```bash
  $ sudo apt update
  $ sudo apt install ca-certificates-java openjdk-8-jdk
```
* Maven
```bash
  $ sudo apt install maven
```
* Native libraries
```bash
  $ sudo apt install build-essential autoconf automake libtool zlib1g-dev pkg-config libssl-dev libsasl2-dev software-properties-common cmake
```
* Protocol Buffers 2.5.0 (required to build native code)
```bash
  $ wget https://github.com/protocolbuffers/protobuf/releases/download/v2.5.0/protobuf-2.5.0.tar.bz2
  $ tar jxf protobuf-2.5.0.tar.bz2
  $ cd protobuf-2.5.0
  $ ./configure --prefix=/usr
  $ make
  $ sudo make install
```
* Boost
```bash
  $ apt install libboost*
```

Optional packages:

* Snappy compression (only used for hadoop-mapreduce-client-nativetask)
```bash
  $ sudo apt install libsnappy-dev
```
* Bzip2
```bash
  $ sudo apt install bzip2 libbz2-dev
```
* Linux FUSE
```bash
  $ sudo apt install libfuse3-dev
```
* ZStandard compression
```bash
  $ sudo apt install libzstd1-dev
```

## Build and install

### Jenkins Pipeline

```groovy
pipeline {
  agent any
  options {
    ansiColor('xterm')
  }
  stages {
    stage('build') {
      steps {
        git branch: "ver3_1_4_on_ubu2204", url: 'https://github.com/dmitrykustov/hadoop.git'
        sh '''#!/usr/bin/env bash
            export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
            export PATH=$PATH:$JAVA_HOME/bin
            mvn package -Pdist,native -DskipTests -Dtar -Dmaven.javadoc.skip=true
        '''
      }
    }
    stage('result') {
      steps {
        sh '''#!/usr/bin/env bash
            pwd
            ls -lh hadoop-dist/target
        '''
      }
    }
  }
}
```

### Install Hadoop

* Copy config
```bash
$ sudo cat <<EOF > /opt/hadoop-3.1.4/etc/hadoop/core-site.xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
EOF
```

* Init and start hadoop
```bash
$ sudo JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64 /opt/hadoop-3.1.4/bin/hdfs namenode -format
$ sudo JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64 /opt/hadoop-3.1.4/bin/hdfs --daemon start namenode
$ sudo JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64 /opt/hadoop-3.1.4/bin/hdfs --daemon start datanode
```

* Create a directory and put a file
```bash
$ sudo JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64 /opt/hadoop-3.1.4/bin/hdfs dfs -mkdir "/user"
$ sudo JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64 /opt/hadoop-3.1.4/bin/hdfs dfs -put README.txt /user/README.txt
```