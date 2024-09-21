---

title: 스프링 배치
date: 2024-08-07
categories: [SpringBatch]
tags: [SpringBatch]
layout: post
toc: true
math: true
mermaid: true

---

# 참고자료

- [Spring Batch](https://docs.spring.io/spring-batch/reference/spring-batch-intro.html)
- [Terasoluna](https://terasoluna-batch.github.io/guideline/5.0.0.RELEASE/en/Ch02_SpringBatchArchitecture.html)
- [정수원](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EB%B0%B0%EC%B9%98/dashboard)

---

# Spring Batch Architecture

![](https://terasoluna-batch.github.io/guideline/5.0.0.RELEASE/en/images/ch02/SpringBatchArchitecture/Ch02_SpringBatchArchitecture_Architecture_ProcessFlow.png)

---

# 1.1 Job

Job은 여러 Step을 포함한 컨테이너로 반드시 한 개 이상의 Step으로 구성해야한다.

Step에는 실제로 배치를 돌리면서 수행할 비즈니스 로직을 책임을 갖는 역할이다.

Job은 인터페이스로 JobExecution을 매개변수로 받아 실행시키는 메서드를 갖는다.

## AbstractJob

SimpleJob, FlowJob의 부모 객체로 아래 필드를 가지고 있는다.

- `name`: Job이름
- `restartable`: 재시작 여부 (기본값 true)
- `JobRepository`: 메타데이터 저장소
- `JobExecutionListener`: 이벤트 리스너
- `JobParameterIncrementer`: JobParameter 증가기
- `JobParameterValidator`: JobParameter 검증기
- `SimpleStepHandler`: Step실행 핸들러

---

## SimpleJob

`순차적`으로 Step을 실행시키는 Job

---

## FlowJob

특정 조건과 흐름에 따라 Step을 구성하여 실행시키는 Job으로 `Flow`객체를 실행시킨다.

---

## Code. Job, Step 생성

```java
@Configuration
@RequiredArgsConstructor
public class JobInstanceConfiguration {
    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job job() {
        return jobBuilderFactory
            .get("job")
            .start(step1())
            .next(step2());
    }

    private Step step1() {
        return stepBuilderFactory
            .get("step1")
            .tasklet(new Tasklet() {
                @Override
                public RepeatStatus execute(StepContribution contribution, ChunckContext chunckContext) throws Exception {
                    return RepeatStatus.FINISHED;
                }
            })
            ;
    }

    private Step step2() {
        return stepBuilderFactory
            .get("step2")
            .tasklet(new Tasklet() {
                @Override
                public RepeatStatus execute(StepContribution contribution, ChunckContext chunckContext) throws Exception {
                    return RepeatStatus.FINISHED;
                }
            })
            .build();
    }
}
```

---

# 1.2 JobInstance

Job의 실행 단위이다. 예를 들어, 하루에 한 번씩 Job이 실행된다면 매일 실행되는 각각의 Job을 각기 다른 JobInstance라고 한다.

JobLancher에 의해 Job이 실행될 때 JobRepoisotry를 통해 Job 메타데이터를 조회하는데 이 때, 이미 JobInstance가 존재한다면 실행하지 않고 JobInstance가 존재하지 않을 때 이를 생성 후 실행하는 흐름으로 이어진다.

JobInstance는 `JobName`, `JobKey(Job Parameter의 해시값)`을 가진 객체이다. 또한 각 JobInstance마다 고유한 `JobInstanceID`를 가진다.

### Table. BATCH_JOB_INSTANCE

| JobInstance ID | Job Name | Job Key   |
|----------------|----------|-----------|
| 1              | 정산       | 123456789 |
| 2              | 정산       | 987654321 |

---

# 1.3 JobParameter

Job을 실행할 때 사용되는 파라미터를 가진 도메인 객체이다.

하나의 Job에 존재할 수 있는 여러 JobInstance를 구분하기 위한 용도이며 JobParameter와 JobInstance는 1:1 관계이다.

## JobParameters

`LinkedHashMap<String, JobParameter> parameters`를 필드로 가진 래퍼 객체이다. 여러 JobParameter를 가진다.

## JobParameter

`Object parameter`, `ParameterType parameterType`, `boolean identifying`이 3가지의 필드를 갖는다.

`ParameterType`은 Enum으로, `STRING`, `DATE`, `LONG`, `DOUBLE`을 가지며 아래와 같이 활용할 수 있다.

```java
public void run (ApplicationArguments args) {
    JobParameters jobParameters =  new JobParametersBuilder()
        .addString("name", "user1")
        .addLong("seq", 2L)
        .addDate("date", new Date())
        .addDouble("double", 12.3)
        .toJobParameters();

    jobLauncher.run(job, jobParameters);
}

// -------------------------- //

@Bean
public synchronized CommandLineRunner runJob() {
    return args -> {
        JobParameters jobParameters = buildJobParameters(args);
        Job job = context.getBean(jobName, Job.class);
        jobLauncher.run(job, jobParameters);
    };
}

private JobParameters buildJobParameters(String[] args) {
    JobParametersBuilder builder = new JobParametersBuilder().addLong("time", System.currentTimeMillis());

    for (String arg : args) {
        String[] keyValue = arg.split("=");
        if (keyValue.length == 2) {
            builder.addString(keyValue[0], keyValue[1]);
        }
    }
    return builder.toJobParameters();
}
```

### Table. BATCH_JOB_EXECUTION_PARAMS

| EXECUTION_ID | TYPE_CD | KEY_NAME |  STRING_VAL  | DATE_VAL            | LONG_VAL | DOUBLE_VAL | IDENTIFYING |
|---|---------|----------|---|---------------------|----------|------------|---|
|              |         |          |              |                     |          |            |             |
1 | STRING  | name     | user1 | 1970-01-01 09:00:00 | 0        | 0          | Y |
1 | LONG    | seq      |  | 1970-01-01 09:00:00 | 2        | 0          | Y |
1 | DATE    | date     |  | now                 | 0        | 0          | Y |
1 | DOUBLE  | double   |  | 1970-01-01 09:00:00 | 0        | 12.3       | Y |

---

# 1.3 JobExecution

`JobInstance`에 대한 한 번의 시도를 의미하는 객체로, Job 실행 중 발생한 정보를 저장하고 있는 객체이다.

따라서 하나의 `JobInstance`가 여러번 실행되면 `JobExecution`도 여러개 생성된다. (1 : N)

## JobExecution

- `JobParameters`
- `JobInstance`
- `ExecutionContext` : 실행 간 유지해야하는 데이터 (JobContext etc ...)
- `BatchStatus` : 실행 상태
  - COMPLETED: 배치 작업 실행을 성공적으로 완료
  - STARTING: 배치 작업 실행 전 상태
  - STARTED: 배치 작업이 실행중
  - STOPPING: 배치 작업을 중지하기 전에 단계가 완료되기를 기다리는 배치 작업 상태
  - STOPPED: 요청에 의해 중지된 배치 작업
  - FAILED: 배치 작업 실행 중에 실패
  - ABANDONED: 제대로 중지되지 않아 다시 시작할 수 없는 배치 작업의 상태
  - UNKNOWN: 불확실한 상태인 배치 작업의 상태
- `ExitStatus` : 실행 결과
  - UNKNOWN: 알 수 없는 상태, 배치 재시작이 불가능하다
  - EXECUTING: 배치 처리가 진행중
  - COMPLETED: 배치 실행이 정상적으로 종료되었을때
  - NOOP: 배치 작업을 처리할 게 없는경우 ex) 배치가 이미 완료가 된경우
  - FAILED: 배치 처리가 실패함
  - STOPPED: 배치 처리 작업 중간에 중단된 상태

### Table. BATCH_JOB_EXECUTION

| JOB_EXECUTION_ID | VERSION | JOB_INSTANCE_ID | CREATE_TIME | START_TIME | END_TIME | STATUS    | EXIT_CODE | EXIT_MESSAGE |
|------------------|---------|-----------------|-------------|------------|----------|-----------|-----------|--------------|
| 1                | 2       | 1               | now()       | now()      | now()    | COMPLETED | COMPLETED |              |

---

# 2.1 Step

실제 Job의 비즈니스 로직을 실행시키는 객체이다. 내부적으로는 `Tasklet`객체를 `execute()`하는 역할이다.

Step은 기본적으로 `Tasklet`이라는 인터페이스를 가지고 있는데, `Tasklet`은 `ItemReader`, `ItemProcessor`, `ItemWriter`이라는 Chunk기반 클래스들을 가진다.

## AbstractStep

모든 Step은 이 AbstractStep을 상속받고 있다. 이 객체가 포함한 필드는 아래와 같다.

- `name`: Step이름
- `startLimit`: Step 실행 제한 횟수
- `allowStartIfComplete`: Step 실행 완료 후 재실행 여부 (기본값 false)
- `stepExecutionListener`: Step 이벤트 리스너
- `jobRepository`: Step 메타데이터 저장소

---

## Step 하위 구현체

### TaskletStep

기본적인 Step이다. Tasklet 타입을 구현한다.

### PartitionStep

멀티 쓰레드 기반으로 Step을 여러 개로 분리해서 실행한다.

### JobStep

Step내에서 Job을 실행시킨다.

### FlowStep

Step내에서 Flow를 실행하도록한다.

---

# 2.2 StepExecution

StepExecution은 Step 실행 중에 발생한 정보들을 갖고 있는다. 따라서 Step이 실제로 시작됐을 때만 생성되는 객체이다.

JobExecution은 이 StepExecution이 모두 정상적으로 완료되어야 완료된다는 관계를 갖는다. (JobExecution:StepExecution = 1:N)

StepExecution이 가지는 필드는 아래와 같다.

- `JobExecution`: JobExecution객체
- `stepName`: Step이름
- `BatchStatus` : 실행 상태
    - COMPLETED: 배치 작업 실행을 성공적으로 완료
    - STARTING: 배치 작업 실행 전 상태
    - STARTED: 배치 작업이 실행중
    - STOPPING: 배치 작업을 중지하기 전에 단계가 완료되기를 기다리는 배치 작업 상태
    - STOPPED: 요청에 의해 중지된 배치 작업
    - FAILED: 배치 작업 실행 중에 실패
    - ABANDONED: 제대로 중지되지 않아 다시 시작할 수 없는 배치 작업의 상태
    - UNKNOWN: 불확실한 상태인 배치 작업의 상태
- `readCount`: read 성공한 아이템 수
- `writeCount`: write 성공한 아이템 수
- `commitCount`: 실행 중 커밋된 트랜잭션 수
- `rollbackCount`: 트랜잭션 중 롤백된 횟수
- `readSkipCount`: read에 실패해서 스킵된 횟수
- `writeSkipCount`: write에 실패해서 스킵된 횟수
- `processSkipCount`: process에 실패해서 스킵된 횟수
- `filterCount`: ItemProcessor에 필터링된 아이템 수
- `startTime`: Job 실행 시의 시스템 시간
- `endTime`: 성공 여부와 관계없는 종료 시간
- `lastUpdated`: JobExecution이 마지막 저장될 때의 시스템 시간
- `ExecutionContext`: 실행되는 동안 유지하는 메타데이터
- `ExitStatus` : 실행 결과
    - UNKNOWN: 알 수 없는 상태, 배치 재시작이 불가능하다
    - EXECUTING: 배치 처리가 진행중
    - COMPLETED: 배치 실행이 정상적으로 종료되었을때
    - NOOP: 배치 작업을 처리할 게 없는경우 ex) 배치가 이미 완료가 된경우
    - FAILED: 배치 처리가 실패함
    - STOPPED: 배치 처리 작업 중간에 중단된 상태
- `failureException`: Job실행 중 발생한 예외 리스트

### Table. BATCH_STEP_EXECUTION

| STEP_EXECUTION_ID | STEP_NAME | JOB_EXECUTION_ID | STATUS    |
|-------------------|-----------|------------------|-----------|
| 1                 | Step1     | 1                | COMPLETED |
| 2                 | Step2     | 1                | COMPLETED |
| 3                 | Step1     | 2                | COMPLETED |
| 4                 | Step2     | 2                | FAILED    |
| 5                 | Step2     | 2                | COMPLETED |

---

# 2.3 StepContribution

Chunk Process의 변경 사항을 버퍼링한 후 StepExecution 상태를 업데이트 하는 객체이다.

Chunk Commit 직전에 StepExecution의 apply()메서드를 호출하여 상태를 업데이트한다.

ExitStatus의 기본 종료코드 외 사용자 정의 종료코드를 생성하여 적용할 수 있다.

StepContribution은 아래 필드를 갖는다.

- `readCount`: read 성공한 아이템 수
- `writeCount`: write 성공한 아이템 수
- `filterCount`: ItemProcessor에 필터링된 아이템 수
- `parentSkipCount`: StepExecution의 총 skip 횟수
- `readSkipCount`: read에 실패해서 스킵된 횟수
- `writeSkipCount`: write에 실패해서 스킵된 횟수
- `processSkipCount`: process에 실패해서 스킵된 횟수
- `ExitStatus` : 실행 결과
    - UNKNOWN: 알 수 없는 상태, 배치 재시작이 불가능하다
    - EXECUTING: 배치 처리가 진행중
    - COMPLETED: 배치 실행이 정상적으로 종료되었을때
    - NOOP: 배치 작업을 처리할 게 없는경우 ex) 배치가 이미 완료가 된경우
    - FAILED: 배치 처리가 실패함
    - STOPPED: 배치 처리 작업 중간에 중단된 상태
- `StepExecution`: StepExecution객체 저장

