#### What is JRuby InvokeDynamic?

JRuby InvokeDynamic is designed for improving performance, however, what I observed in real environment (especially running Rails) was decreasing performance.

You can enable invokedynamic use on Java 7, but it is disabled normally due to JVM issues. On Java 8 builds, it is enabled by default.

see https://github.com/jruby/jruby/wiki/PerformanceTuning for more details

#### How to turn it off for Go server

By Java options: -Djruby.compile.invokedynamic=false

#### Performance test

Test environment:

* Standalone Go server with some simple configured pipelines: 5 simple pipelines configured (see [Go server config](https://github.com/xli/gocd-invokedynamic-testresult/blob/master/cruise-config.xml))
* One Agent connected to server, and no job is scheduled. All pipelines built once and green.
* Go server is built from latest master branch
* Java version: 1.8.0_40
* OS: Mac 10.11.2

Test tool: Apache benchmarking tool AB
Test cases:
* 1000 requests, 1 concurrency, example: ab -n 1000 -c 1 http://localhost:8153/go/admin/pipelines/<pipelinename>/stages/defaultStage/settings
* 1000 requests, 20 concurrency, example: ab -n 1000 -c 20 http://localhost:8153/go/admin/pipelines/<pipelinename>/stages/defaultStage/settings
Test pages:
* stage settings page (has 3 jobs configured in the stage)
* pipelines.json (xhr request on pipeline dashboard page)

Test result:

##### /go/admin/pipelines/hello/stages/defaultStage/settings, 1000 requests, 20 concurrency: 

command: ab -n 1000 -c 20 http://localhost:8153/go/admin/pipelines/hello/stages/defaultStage/settings

| JRuby invoke dynamic | Time taken for tests | Time per request (mean) |
|----------------------|----------------------|-------------------------|
| off                  | 1.067 sec            | 21.345ms                |
| on                   | 3.775 sec            | 75.510ms                |


##### /pipelines.json, 1000 requests, 20 concurrency:

command: ab -n 1000 -c 20 http://localhost:8153/go/pipelines.json

| JRuby invoke dynamic | Time taken for tests | Time per request (mean) |
|----------------------|----------------------|-------------------------|
| off                  | 0.945 sec            | 18.898ms                |
| on                   | 1.205 sec            | 24.095ms                |


##### /go/admin/pipelines/hello/stages/defaultStage/settings, 1000 requests, 1 concurrency: 

command: ab -n 1000 -c 1 http://localhost:8153/go/admin/pipelines/hello/stages/defaultStage/settings

| JRuby invoke dynamic | Time taken for tests | Time per request (mean) |
|----------------------|----------------------|-------------------------|
| off                  | 4.852 sec            | 4.852ms                 |
| on                   | 9.800 sec            | 9.800ms                 |

##### /pipelines.json, 1000 requests, 1 concurrency: 

command: ab -n 1000 -c 1 http://localhost:8153/go/pipelines.json

| JRuby invoke dynamic | Time taken for tests | Time per request (mean) |
|----------------------|----------------------|-------------------------|
| off                  | 4.500 sec            | 4.500ms                 |
| on                   | 5.222 sec            | 5.222ms                 |


#### Others

Another noticable performance difference in test is JVM warm-up speed. It was a lot faster to get stable response time when JRuby invoke dynamic is off.

You can go to https://github.com/xli/gocd-invokedynamic-testresult to checkout full AB test reports mentioned above

I only give 2 page tests result above for comparison. 
But most of pages should have similar performance difference with pipelines.json request.

#### Conclusion

Turning off JRuby invoke dynamic feature on Java 8 has significant performance improvement.
More and more Go user will move to Java 8 as Oracle Java 7 no longer has public updates anymore.
So we should turn it off by default on Go Server.


