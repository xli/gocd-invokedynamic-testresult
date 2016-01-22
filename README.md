#### What is JRuby InvokeDynamic?

JRuby InvokeDynamic is designed for improving performance, however, what I observed in in real environment (especially running Rails) was decreasing performance.

You can enable invokedynamic use on Java 7, but it is disabled normally due to JVM issues. On Java 8 builds, it is enabled by default.

see https://github.com/jruby/jruby/wiki/PerformanceTuning for more details

#### How to turn it off for Go server

BY JAVA OPTIONS: -Djruby.compile.invokedynamic=false

#### Performance test

Test environment:

* Standalone Go server with some simple configured pipelines: 5 simple pipelines configured.
* No Agent connected to server, and not job is scheduled.
* Go server is built from latest master branch
* Java version: 1.8.0_40
* OS: Mac 10.11.2

Test tool: Apache benchmarking tool AB
Test cases:
* 1000 requests, 1 concurrency, example: ab -n 1000 -c 1 http://localhost:8153/go/admin/pipelines/aaaaa/stages/defaultStage/settings
* 1000 requests, 20 concurrency, example: ab -n 1000 -c 20 http://localhost:8153/go/admin/pipelines/aaaaa/stages/defaultStage/settings
Test pages:
* stage settings page (has 3 jobs configured in the stage)
* pipelines.json (xhr request on pipeline dashboard page)

Test result:

##### /go/admin/pipelines/<pipelinename>/stages/defaultStage/settings, 1000 requests, 20 concurrency: 

| JRuby invoke dynamic | Time taken for tests | Time per request (mean) |
|----------------------|----------------------|-------------------------|
| off                  | 1.067 sec            | 21.345ms                |
| on                   | 3.775 sec            | 75.510ms                |


##### /pipelines.json, 1000 requests, 20 concurrency:

| JRuby invoke dynamic | Time taken for tests | Time per request (mean) |
|----------------------|----------------------|-------------------------|
| off                  | 0.945 sec            | 18.898ms                |
| on                   | 1.205 sec            | 24.095ms                |


##### /go/admin/pipelines/<pipelinename>/stages/defaultStage/settings, 1000 requests, 1 concurrency: 

| JRuby invoke dynamic | Time taken for tests | Time per request (mean) |
|----------------------|----------------------|-------------------------|
| off                  | 4.852 sec            | 4.852ms                 |
| on                   | 9.800 sec            | 9.800ms                 |

##### /pipelines.json, 1000 requests, 1 concurrency: 

| JRuby invoke dynamic | Time taken for tests | Time per request (mean) |
|----------------------|----------------------|-------------------------|
| off                  | 4.500 sec            | 4.500ms                 |
| on                   | 5.222 sec            | 5.222ms                 |


#### Others

Another noticable performance difference in test is JVM warm-up speed. 
There is almost no warm-up need when invoke dynamic is off. 
During the test, I need do 30,000 requests to warm-up JVM when invokedynamic is on.

You can go to https://github.com/xli/gocd-invokedynamic-testresult to checkout full AB test reports mentioned above
