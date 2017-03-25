# Docker IRL projects 
AgileJKL 29.3.2017

Notes:
- Who am I and what do I do at Gofore
- How long have I worked at Gofore
- Job history; sysadmin/consultant positions 9+ years
- Current project: Fonecta

---

## Agenda

1. Why I love Docker
2. Why should you care?
3. Daily use cases
4. Case: Fonecta CI
5. Example: Dockerizing and deploying a Node.js app

Notes:

- Structure of the presentation

---

## Why I love Docker

- Fast and agile way to try out new things
- Apps, tools, DB's + dependencies packaged as images
- No need for the download, install, upgrade, uninstall process
- Hop between versions just by changing tags
- Isolate different versions of a software from each other 

Notes:
- Example, running maven + test dependency db's
- Testing new versions of software

---

## Why should you care

- Process Isolation
- Versioning of dev and prod environments
- Lighter and faster than Vagrant
- Excellent for building and testing Microservices with
- Faster deployments

Notes:
- Java versions, Node+Npm versions
- FROM nodejs:6.9
- docker-compose up -> 15 services fire up and are ready to communicate
with each other 
- Docker not a necessity for microservices of course

---

## Daily use cases

- _"How fast can you set up a GeoServer?"_

```bash
docker run --name "postgis" -d -t kartoza/postgis:9.4-2.1
docker run -p 8080:8080 --link postgis:postgis kartoza/geoserver
```

Notes:
- Customer asks in a meeting, whether I have worked with GeoServer before,
and how long would it take to set it up. Fire up a container during the meeting,
and let them know that it's feasible and seems fairly simple.

--

- _"Can we do a migration between two MongoDB replicasets?_

```yaml
version: '2'
services:
  mongo-1:
    image: mongo:2.6
    command: mongod --replSet rs0
    ports:
      - 30001:27017

  mongo-2:
    image: mongo:2.6
    command: mongod --replSet rs0
    ports:
      - 30002:27017
....

  mongo-builder:
    image: mongo:2.6
    command: bash -c "sleep 10 && mongo --host mongo-1 --eval 'rs.initiate()' \
              && sleep 5 && mongo --host mongo-1 --eval 'rs.add(\"mongo-2\")' \
              && mongo --host mongo-3 --eval 'rs.initiate()' && sleep 5 \
              && mongo --host mongo-3 --eval 'rs.add(\"mongo-4\")'"
    depends_on:
      - mongo-1
      - mongo-2
      - mongo-3
      - mongo-4
```

Notes:
- Application needs to be migrated live from Mongo replicaset1 to replicaset2
 - How to test? Run two replicasets + mongo-connector locally

--

- _If only there was a quick way to see if this Java 6 project compiles with Java 8_

```bash
WORKDIR=/app
docker run \
--rm \
-v ${PWD}:${WORKDIR} \
-v ~/.m2:${WORKDIR}/.m2 \
--workdir ${WORKDIR} maven:3.3.9-jdk8 \
mvn -B -Dmaven.repo.local=${WORKDIR}/.m2/repository/ compile
```

Notes:
- Running Maven without installing it

---

## Case: Fonecta CI

- Jenkins v1.5 set up 5 years ago
- EC2-classic, manually installed
- 300+ jobs
- Build dependencies installed by hand as needed
- MongoDB, MySQL, PostgreSQL, Redis, Java, Scala, Node..
- Only documentation of a job is the job itself

Notes:
- Ongoing project to move all relevant jobs away from the old Jenkins server

--

### How to fix it?

- Jenkins 2, Pipelines
- Jenkins in a container for quick upgrades/downgrades
- Automated setup with Chef
- Self-healing with AWS AutoScaling

Notes:
- Totally change the way the builds are configured
- Enable recreation of the Jenkins instance
- Scripts to handle data persistence: tagged volume
- Previously system + Jenkins upgrades were high-risk => Docker solved

--

### What about the builds?

- Build configuration in version control: Jenkinsfiles
- Have I mentioned docker yet?

```groovy
stage('Build') {
  try {
    docker.image("maven:3.3.9-jdk8").inside {
      git url: 'https://github.com/lauriku/dropwizard-example.git'
      withEnv(["MAVEN_OPTS=$mavenArgs"]) {
        sh "mvn -B -s ${mavenSettingsFile} clean compile"
      }
    }
  } catch(e) {
    slackSend channel: projectSlackChannel, color: 'danger', message: slackMessageForFailedBuild
    throw e
  }
}
```

Notes:
- All job configurations and their change history now saved
- Tooling + Java in containers
- Builds isolated from each other

--

### I need a database to run my tests!

```groovy
stage('Test') {
  node {
    try {
      docker.image("mongo:2.6.12").withRun { mongo ->
        docker.image("maven:3.3.9-jdk8").inside("--link=${mongo.id}:${mongoContainerName}") {
          withEnv(["mongo.host=${mongoContainerName}"]) {
            sh "mvn test"
          }
        }
      }
    } catch(e) {
      .....
    }
  }
}
```

Notes:
- Start mongo first, and keep it running for as long as the commands inside its
closure take to execute

---

## Example: Dockerizing a Node.js app
Let's shove a Node app inside a container and deploy it to Amazon ECS!

Notes:
- Create a Dockerfile
- Build locally
- Create a Docker hub automated build
- Create ECS Cluster
- Deploy!

---

## Questions?

---

## Thanks!



