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

##### /pipelines.json, 1000 requests, 20 concurrency:

| JRuby invoke dynamic | Time taken for tests | Time per request (mean) |
|----------------------|----------------------|-------------------------|
| off                  | 0.996 sec            | 19.9ms                  |
| on                   | 1.59 sec             | 31.8ms                  |


##### /go/admin/pipelines/<pipelinename>/stages/defaultStage/settings, 1000 requests, 20 concurrency: 

| JRuby invoke dynamic | Time taken for tests | Time per request (mean) |
|----------------------|----------------------|-------------------------|
| off                  | 1.122 sec            | 22.438ms                |
| on                   | 2.145 sec            | 42.9ms                  |


##### /pipelines.json, 1000 requests, 1 concurrency: 

| JRuby invoke dynamic | Time taken for tests | Time per request (mean) |
|----------------------|----------------------|-------------------------|
| off                  | 4.346 sec            | 4.346ms                 |
| on                   | 5.928 sec            | 5.928ms                 |


##### /go/admin/pipelines/<pipelinename>/stages/defaultStage/settings, 1000 requests, 1 concurrency: 

| JRuby invoke dynamic | Time taken for tests | Time per request (mean) |
|----------------------|----------------------|-------------------------|
| off                  | 4.772 sec            | 4.772ms                 |
| on                   | 10.714 sec           | 10.714ms                |

#### Others

Another noticable performance difference in test is JVM warm-up speed. It was a lot faster to get stable response time when JRuby invoke dynamic is off.

You can go to https://github.com/xli/gocd-invokedynamic-testresult to checkout full AB test reports mentioned above
