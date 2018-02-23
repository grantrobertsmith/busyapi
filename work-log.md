# Work Log

In summary, here's what I did:

  1. Researched local benchmarking tools
  2. Researched node server performance
  3. Forked, cloned and modify the example web server code base
  4. Tested locally
  5. Filled out this work log

## Exposition of Work Performed

First, I needed to determine how to test the throughput of my local machine acting as a web server. I came across tools like ApacheBench (ab), Seige, and gobench on the internet during this process. I ran into limitations with ab, and opted to use gobench moving forward.

Next, I learned that a node server running on my machine that was responding to a GET request, sending an empty file in response could only respond to about 750K requests per minute. If the node server was responding on a single process, using a single core, it could get up to about 350K responses per minute, and after using node's cluster package to fork the node server process, I increased this to 750K while testing using gobench. With this in mind, it is likely impossible to get 1M request served per minute on my machine or the one you will use to test with.

Then, I cloned this test repo to my machine, and tested it using go bench as it was, and I noticed that it responded to far fewer requests than the basic node server I previously tested. I also noticed that the server was logging a lot of text as it responded to requests. This was significantly slowing the processing time, so I commented out the use of the logger and the server side logging of the usage ID. This puts it on par with the basic testing I did earlier. It is still somewhat slower than the simple server I tested earlier because it's doing a bit more work (e.g. it updates an in memory array and serves actual data as a response, among other things).

One important note, is that when I fork the process locally, the server is responding with a usage ID tracked in the local process, so the functionality of a globally unique usage ID is lost. In a production setting, using a database storage system to back this ID would solve this issue. Locally, this issue could be solved by passing data between the forked server processes, but it would only hurt performance.

## Suggested changes to the software architecture/stack that may achieve the goal

Ideally, this program would be using a database to store the usage history, rather than an in memory array of usages. While this would add overhead to processing and responding to individual HTTP requests, it solves the issue I created by forking the server process to take advantages of all the cores on my machine. Beyond this, a production web server environment designed to handle millions of requests per minute would require multiple machines responding to incoming traffic. These machines should be placed behind a load balancer that has a routing scheme appropriate to the implementation of the server. In this case, it wouldn't really matter what the scheme were becuse the application itself is very basic.

It would also be possible to use a different web server module with lower overhead, perhaps written in a different, lower-level language.

## Suggested changes to the physical architecture/hardware that may achieve the goal

Some processors have more cores or have multi-threading on a per-core basis. These features would allow for increased throughput performance by further parallelizing request processing on a single machine. The load balancing thing that I talked about is also kind of a hardware thing.

## A working build of the code capable of achieving the goal

On the hardware that you described you would be using for testing, which is similar to my hardware (a 2014 MacBook Pro), I'm not sure that it's possible to serve 1M requests in a minute using node's http server module. Certainly while consuming CPU resources in order to create and send the requests from the same machine, which is the only one available to me at the moment, makes it even more challenging. Maybe if I turned off all my other software and didn't try to measure how many requests it processed, it could, but then we wouldn't know.

## Methods of measuring whether or not the goal has been achieved

To measure the number of requests I was able to serve locally, I generated requests on the same machine using a throughput testing module called gobench that I found during my research phase of this project. Alternatively, a production environment would use tools like DataDog and NewRelic to monitor the health and response times of their systmes. These tools include information about web server reponse times and throughput. Additional production-style testing can be performed through specific setups such as test harnesses or external end to end testing machines.