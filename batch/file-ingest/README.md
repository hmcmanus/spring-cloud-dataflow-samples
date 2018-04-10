## This is an example branch for testing not to be merged

In order to provide an example of a problem that I've been having referenced here:

https://github.com/spring-cloud/spring-cloud-task/issues/414#issuecomment-380096971

## Recreate the issue by completing the following steps

1. Clone this repository and switch to this branch

```
git checkout dev-hmcmanus-batchParameterProblem
```

I've added another file in the resources directory which we are going to attempt to get the batch to consume:

```
src/main/resource/data2.csv
```

2. Build the example

```
mvn clean package 
```

You should now have target/ingest-1.0.1.jar

3. Download and run the specified version of the local SCDF server

```
java -jar spring-cloud-dataflow-server-local-1.4.0.RELEASE.jar
```

4. Download and run the specified dataflow shell:

```
java -jar ./spring-cloud-dataflow-shell-1.3.1.RELEASE.jar
```

5. Register the app

```
dataflow:> app register --name fileIngest --type task --uri file:///path/to/target/ingest-1.0.1.jar
```

6. Create the task

```
dataflow:> task create fileIngestTask --definition fileIngest
```

7. Launch the task

```
dataflow:> task launch fileIngestTask --arguments "filePath=classpath:data.csv --spring.cloud.task.closecontext_enable=false"
```

At this point all good, the batch server will consume the data.csv file no problems and you should get the following output:

> 2018-04-11 07:57:06.461  INFO 23759 --- [           main] o.s.b.a.b.JobLauncherCommandLineRunner   : Running default command line with: [filePath=classpath:data.csv, --spring.cloud.task.closecontext_enable=false, --spring.cloud.task.executionid=1]
>2018-04-11 07:57:06.507  INFO 23759 --- [           main] o.s.b.c.l.support.SimpleJobLauncher      : Job: [FlowJob: [name=ingestJob]] launched with the following parameters: [{filePath=classpath:data.csv, -spring.cloud.task.executionid=1, -spring.cloud.task.closecontext_enable=false, run.id=1}]
>2018-04-11 07:57:06.513  INFO 23759 --- [           main] o.s.c.t.b.l.TaskBatchExecutionListener   : The job execution id 1 was run within the task execution 1
>2018-04-11 07:57:06.532  INFO 23759 --- [           main] o.s.batch.core.job.SimpleStepHandler     : Executing step: [ingest]
>2018-04-11 07:57:06.601  INFO 23759 --- [           main] o.s.i.processor.PersonItemProcessor      : Processed: First name: Jill , last name: Doe into: First name: JILL , last name: DOE
>2018-04-11 07:57:06.601  INFO 23759 --- [           main] o.s.i.processor.PersonItemProcessor      : Processed: First name: Joe , last name: Doe into: First name: JOE , last name: DOE
>2018-04-11 07:57:06.601  INFO 23759 --- [           main] o.s.i.processor.PersonItemProcessor      : Processed: First name: Justin , last name: Doe into: First name: JUSTIN , last name: DOE
>2018-04-11 07:57:06.601  INFO 23759 --- [           main] o.s.i.processor.PersonItemProcessor      : Processed: First name: Jane , last name: Doe into: First name: JANE , last name: DOE
>2018-04-11 07:57:06.601  INFO 23759 --- [           main] o.s.i.processor.PersonItemProcessor      : Processed: First name: John , last name: Doe into: First name: JOHN , last name: DOE
>2018-04-11 07:57:06.638  INFO 23759 --- [           main] o.s.b.c.l.support.SimpleJobLauncher      : Job: [FlowJob: [name=ingestJob]] completed with the following parameters: [{filePath=classpath:data.csv, -spring.cloud.task.executionid=1, -spring.cloud.task.closecontext_enable=false, run.id=1}] and the following status: [COMPLETED]

8. Now the interesting part, launch the task to consume the other data2.csv

```
dataflow:> task launch fileIngestTask --arguments "filePath=classpath:data2.csv --spring.cloud.task.closecontext_enable=false"
```

You can see here that the task gets the correct arguments, but the batch job does not, therefore incorrect data is consumed that is intended. 

> 2018-04-11 07:59:21.512  INFO 23916 --- [           main] o.s.b.a.b.JobLauncherCommandLineRunner   : Running default command line with: [filePath=classpath:data2.csv, --spring.cloud.task.closecontext_enable=false, --spring.cloud.task.executionid=2]
>2018-04-11 07:59:21.576  INFO 23916 --- [           main] o.s.b.c.l.support.SimpleJobLauncher      : Job: [FlowJob: [name=ingestJob]] launched with the following parameters: [{filePath=classpath:data.csv, -spring.cloud.task.executionid=1, -spring.cloud.task.closecontext_enable=false, run.id=2}]
>2018-04-11 07:59:21.582  INFO 23916 --- [           main] o.s.c.t.b.l.TaskBatchExecutionListener   : The job execution id 2 was run within the task execution 2
>2018-04-11 07:59:21.602  INFO 23916 --- [           main] o.s.batch.core.job.SimpleStepHandler     : Executing step: [ingest]
>2018-04-11 07:59:21.646  INFO 23916 --- [           main] o.s.i.processor.PersonItemProcessor      : Processed: First name: Jill , last name: Doe into: First name: JILL , last name: DOE
>2018-04-11 07:59:21.647  INFO 23916 --- [           main] o.s.i.processor.PersonItemProcessor      : Processed: First name: Joe , last name: Doe into: First name: JOE , last name: DOE
>2018-04-11 07:59:21.647  INFO 23916 --- [           main] o.s.i.processor.PersonItemProcessor      : Processed: First name: Justin , last name: Doe into: First name: JUSTIN , last name: DOE
>2018-04-11 07:59:21.647  INFO 23916 --- [           main] o.s.i.processor.PersonItemProcessor      : Processed: First name: Jane , last name: Doe into: First name: JANE , last name: DOE
>2018-04-11 07:59:21.647  INFO 23916 --- [           main] o.s.i.processor.PersonItemProcessor      : Processed: First name: John , last name: Doe into: First name: JOHN , last name: DOE
>2018-04-11 07:59:21.666  INFO 23916 --- [           main] o.s.b.c.l.support.SimpleJobLauncher      : Job: [FlowJob: [name=ingestJob]] completed with the following parameters: [{filePath=classpath:data.csv, -spring.cloud.task.executionid=1, -spring.cloud.task.closecontext_enable=false, run.id=2}] and the following status: [COMPLETED]a


The interesting parts are how the task is called:

```
Running default command line with: [filePath=classpath:data2.csv, --spring.cloud.task.closecontext_enable=false, --spring.cloud.task.executionid=2]
```

and how the batch job is called differently:

```
Job: [FlowJob: [name=ingestJob]] launched with the following parameters: [{filePath=classpath:data.csv, -spring.cloud.task.executionid=1, -spring.cloud.task.closecontext_enable=false, run.id=2}]
```
