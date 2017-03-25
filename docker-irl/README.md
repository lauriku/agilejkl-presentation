# Docker IRL projects 
AgileJKL 29.3.2017

Notes:
- Who am I and what do I do at Gofore
-  

---

## Agenda

1. Why I love Docker
2. Why should you care?
3. Daily use cases
4. Production environments
5. Case: Fonecta CI
6. Example: Dockerizing and deploying a Node.js application to Amazon ECS

Notes:

- Structure of the presentation
- 

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
- Excellent for building Microservices with
- Faster deployments

Notes:
- Java versions, Node+Npm versions
- FROM nodejs:6.9
- docker-compose up -> 15 services fire up and are ready to communicate
with each other 

---

## Daily use cases

- _"How fast can you set up a GeoServer?"_
```bash
docker run --rm -p 8080:8080 kartoza/geoserver
```

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
- Customer asks in a meeting, whether I have worked with GeoServer before,
and how long would it take to set it up. Fire up a container during the meeting,
and let them know that it's feasible and seems fairly simple.
- Application needs to be migrated live from Mongo replicaset1 to replicaset2
 - How to test? Run two replicasets + mongo-connector locally

---

## Code block

You can even add code examples:

```python
def print_names(names):
  for name in names:
    print(name)
```

---

## Inline code

Or show inline code example such as `sudo apt-get upgrade`.

---

## Tables

If you want to get fancy, you can add tables:

| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |

---

## Blockquotes

> This is a blockquote
> spanning multiple lines

---

## Reveal.js examples

For more examples, see the [Reveal.js demo](http://lab.hakim.se/reveal-js/)
