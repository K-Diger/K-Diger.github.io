---

title: Kafka Partition 할당 내부 동작 파해치기
date: 2024-12-17
categories: [Kafka]
tags: [Kafka]
layout: post
toc: true
math: true
mermaid: true

---

### 참고자료

- [KIP-480 StickyPartitioner](https://cwiki.apache.org/confluence/display/KAFKA/KIP-480%3A+Sticky+Partitioner)
- [KIP-429 살짝 관련성은 떨어지지만 Consumer Rebalance 관련](https://cwiki.apache.org/confluence/display/KAFKA/KIP-429:+Kafka+Consumer+Incremental+Rebalance+Protocol#KIP429:KafkaConsumerIncrementalRebalanceProtocol-Consumer)

## 배경

카프카는 컨슈머 그룹내의 컨슈머들에 파티션을 배정하는 할당 전략이 있다. 컨슈머 그룹이 어떤 토픽을 바라보고 있을 때 토픽 내의 파티션들을 각 컨슈머들에게 어떻게 나눠줄지에 대한 알고리즘이다.

이 글을 쓰게된 시작점은 파티션을 이미 할당 받은상태에서 컨슈머 혹은 파티션에 변화가 일어났을 때 아주 균일하게 분배할 수 있다는 것은 쉽게 알 수 있었지만

- 어떻게 균등하게 분배를 하는건지
- 공식문서에 설명으로는 이미 파티션이 컨슈머에 할당되어 있는 상태만을 이야기하는데, 그러면 초기에 기동했을 때 파티션을 할당하는 것은 어떻게하는지?

이 두 가지 궁금증을 해결하고자 직접 구현체를 뜯어봤다.

---

## 전략의 네이밍을 통한 동작방식 알아내기

먼저 네이밍을 알면 이 전략이 어떻게 동작하는지 알 수 있다. `협력적`이라는 단어는 재쳐두고 `스티키(Sticky)`

라는 네이밍은 `들러붙다`라는 뉘앙스이다. 조금 더 확대 해석해보자면 고정적이라는 의미로도 해석되는데 이미 스티키 할당 전략에 대해 알고있는 상황에서,

엄청나게 확대 해석을 하면 이미 할당되어있는 파티션과 컨슈머는 고정시키고 새로운 파티션이나 컨슈머에 대해서만 리밸런싱을 적용하겠다는 전략의 네이밍으로 보인다.

그러면 `협력적`은 무슨 의미일까? 각 컨슈머들이 서로의 정보를 주고받아서 (정확히는 컨슈머 그룹의 리더가 관제) 리밸런싱 시 모든 파티션 할당을 취소하는게 아니라 필요한 파티션에 대해서만 리밸런싱을 수행하는 것이다.

따라서 협력적 스티키 파티션 할당 전략은 컨슈머 그룹 내의 컨슈머들이 서로의 정보를 주고받으면서 필요한 파티션에 대해서만 리밸런싱을 수행하고 이미 할당받은 파티션들은 가급적 유지하도록 하는 전략이다.

---

## 스티키 파티션 할당 전략

1. 스티키 파티션 할당은 기존 파티션 할당 정보를 저장하고
2. 리밸런싱 시 컨슈머 그룹 리더에 이전 할당 정보를 전달한다.

### StickyAssignor.java

구현체의 코드는 너무 길기 때문에 우선 주요 메서드로 보이는 내용만 발췌했다.

```java
package org.apache.kafka.clients.consumer;

public class StickyAssignor extends AbstractStickyAssignor {

    ...

    // 리밸런싱 후 할당된 파티션 정보와 세대 ID를 저장하는 메서드
    @Override
    public void onAssignment(Assignment assignment, ConsumerGroupMetadata metadata) {
        memberAssignment = assignment.partitions();
        this.generation = metadata.generationId();
    }

    // 컨슈머가 가진 파티션 할당 정보를 ByteBuffer로 직렬화하여 반환하는 메서드
    // 이는 다음 리밸런싱에서 그룹 리더가 파티션을 할당할 때 참고하는 데이터가 됨
    @Override
    public ByteBuffer subscriptionUserData(Set<String> topics) {
        if (memberAssignment == null)
            return null;

        return serializeTopicPartitionAssignment(new MemberData(memberAssignment, Optional.of(generation)));
    }

    // 컨슈머의 subscription 데이터로부터 현재 할당된 파티션 정보를 역직렬화하여 추출하는 메서드
    // userData가 없으면 빈 MemberData 반환
    @Override
    protected MemberData memberData(Subscription subscription) {
        // Always deserialize ownedPartitions and generation id from user data
        // since StickyAssignor is an eager rebalance protocol that will revoke all existing partitions before joining group
        ByteBuffer userData = subscription.userData();
        if (userData == null || !userData.hasRemaining()) {
            return new MemberData(Collections.emptyList(), Optional.empty());
        }
        return deserializeTopicPartitionAssignment(userData);
    }

    // 컨슈머의 파티션 할당 정보(MemberData)를 ByteBuffer로 직렬화하는 메서드
    // 토픽별로 파티션을 그룹화하여 구조체 형태로 변환 후 ByteBuffer에 저장
    static ByteBuffer serializeTopicPartitionAssignment(MemberData memberData) {
        Struct struct = new Struct(STICKY_ASSIGNOR_USER_DATA_V1);
        List<Struct> topicAssignments = new ArrayList<>();
        for (Map.Entry<String, List<Integer>> topicEntry : CollectionUtils.groupPartitionsByTopic(memberData.partitions).entrySet()) {
            Struct topicAssignment = new Struct(TOPIC_ASSIGNMENT);
            topicAssignment.set(TOPIC_KEY_NAME, topicEntry.getKey());
            topicAssignment.set(PARTITIONS_KEY_NAME, topicEntry.getValue().toArray());
            topicAssignments.add(topicAssignment);
        }
        struct.set(TOPIC_PARTITIONS_KEY_NAME, topicAssignments.toArray());
        if (memberData.generation.isPresent())
            struct.set(GENERATION_KEY_NAME, memberData.generation.get());
        ByteBuffer buffer = ByteBuffer.allocate(STICKY_ASSIGNOR_USER_DATA_V1.sizeOf(struct));
        STICKY_ASSIGNOR_USER_DATA_V1.write(buffer, struct);
        buffer.flip();
        return buffer;
    }

    // ByteBuffer로 직렬화된 파티션 할당 정보를 MemberData 객체로 역직렬화하는 메서드
    // V1, V0 두 가지 버전의 스키마를 지원하며 파싱 실패시 빈 MemberData 반환
    private static MemberData deserializeTopicPartitionAssignment(ByteBuffer buffer) {
        Struct struct;
        ByteBuffer copy = buffer.duplicate();
        try {
            struct = STICKY_ASSIGNOR_USER_DATA_V1.read(buffer);
        } catch (Exception e1) {
            try {
                // fall back to older schema
                struct = STICKY_ASSIGNOR_USER_DATA_V0.read(copy);
            } catch (Exception e2) {
                // ignore the consumer's previous assignment if it cannot be parsed
                return new MemberData(Collections.emptyList(), Optional.of(DEFAULT_GENERATION));
            }
        }

        List<TopicPartition> partitions = new ArrayList<>();
        for (Object structObj : struct.getArray(TOPIC_PARTITIONS_KEY_NAME)) {
            Struct assignment = (Struct) structObj;
            String topic = assignment.getString(TOPIC_KEY_NAME);
            for (Object partitionObj : assignment.getArray(PARTITIONS_KEY_NAME)) {
                Integer partition = (Integer) partitionObj;
                partitions.add(new TopicPartition(topic, partition));
            }
        }
        // make sure this is backward compatible
        Optional<Integer> generation = struct.hasField(GENERATION_KEY_NAME) ? Optional.of(struct.getInt(GENERATION_KEY_NAME)) : Optional.empty();
        return new MemberData(partitions, generation);
    }
}
```

위 코드에서 한글로 주석을 달아놓은 부분이 주요 메서드이다. 해당 메서드를 통해 파티션 할당 정보를 기록하여 파티션 변경에서 어느정도 `스티키`한 속성을 구현한다.

---

그렇다면 이제 협력적 스티키 할당 전략으로 어떻게 협력을 한다는 것인지 코드로 확인해본다. 이 역시도 중

### CooperateiveSticyAssignor.java

```java
package org.apache.kafka.clients.consumer;

public class CooperativeStickyAssignor extends AbstractStickyAssignor {

    private int generation = DEFAULT_GENERATION; // consumer group generation

    // 리밸런싱 후 현재 세대 ID를 저장하는 메서드
    @Override
    public void onAssignment(Assignment assignment, ConsumerGroupMetadata metadata) {
        this.generation = metadata.generationId();
    }

    // 컨슈머의 세대 정보를 ByteBuffer로 직렬화하여 반환하는 메서드
    @Override
    public ByteBuffer subscriptionUserData(Set<String> topics) {
        Struct struct = new Struct(COOPERATIVE_STICKY_ASSIGNOR_USER_DATA_V0);
        struct.set(GENERATION_KEY_NAME, generation);
        ByteBuffer buffer = ByteBuffer.allocate(COOPERATIVE_STICKY_ASSIGNOR_USER_DATA_V0.sizeOf(struct));
        COOPERATIVE_STICKY_ASSIGNOR_USER_DATA_V0.write(buffer, struct);
        buffer.flip();
        return buffer;
    }

    // subscription 데이터로부터 컨슈머가 소유한 파티션 정보와 세대 정보를 추출하는 메서드
    @Override
    protected MemberData memberData(Subscription subscription) {
        // ConsumerProtocolSubscription v2 이상에서는 직접 필드에서 데이터 추출
        if (subscription.generationId().isPresent()) {
            return new MemberData(subscription.ownedPartitions(), subscription.generationId());
        }

        ByteBuffer buffer = subscription.userData();
        Optional<Integer> encodedGeneration;
        if (buffer == null) {
            encodedGeneration = Optional.empty();
        } else {
            try {
                Struct struct = COOPERATIVE_STICKY_ASSIGNOR_USER_DATA_V0.read(buffer);
                encodedGeneration = Optional.of(struct.getInt(GENERATION_KEY_NAME));
            } catch (Exception e) {
                encodedGeneration = Optional.of(DEFAULT_GENERATION);
            }
        }
        return new MemberData(subscription.ownedPartitions(), encodedGeneration);
    }

    // 협력적 리밸런싱을 위한 파티션 할당 메서드
    // 기존 할당 정보를 기반으로 재할당이 필요한 파티션들을 조정
    @Override
    public Map<String, List<TopicPartition>> assign(Map<String, Integer> partitionsPerTopic,
                                                    Map<String, Subscription> subscriptions) {
        // 부모 클래스의 스티키 할당 전략을 사용하여 초기 파티션 할당을 수행
        Map<String, List<TopicPartition>> assignments = super.assign(partitionsPerTopic, subscriptions);

        // 소유권이 이전되는 파티션 정보를 계산하거나 이미 계산된 정보를 사용
        // 이 정보는 현재 소유자로부터 새로운 소유자로 이전되어야 하는 파티션들의 맵
        Map<TopicPartition, String> partitionsTransferringOwnership = super.partitionsTransferringOwnership == null ?
            computePartitionsTransferringOwnership(subscriptions, assignments) :
            super.partitionsTransferringOwnership;

        // 파티션 할당 적용
        adjustAssignment(assignments, partitionsTransferringOwnership);
        return assignments;
    }

    // 협력적 리밸런싱 프로토콜에 따라 먼저 취소되어야 하는 파티션들을 할당에서 제거하는 메서드
    private void adjustAssignment(Map<String, List<TopicPartition>> assignments,
                                  Map<TopicPartition, String> partitionsTransferringOwnership) {
        for (Map.Entry<TopicPartition, String> partitionEntry : partitionsTransferringOwnership.entrySet()) {
            assignments.get(partitionEntry.getValue()).remove(partitionEntry.getKey());
        }
    }
}
```

그런데 여기서, 두 객체에서 아주 중요한 역할로 보이는 assign()메서드에서 부모 객체의 assign메서드를 사용하는 구문이 있다.

그렇다면 부모의 assign 메서드에는 어떤 내용이 있길래 공통으로 사용하는지 알게 되면 구체적으로 어떻게 균일하게 분배하는지 알 수 있을 것으로 보인다.

### AbstractStickyAssignor.java

```java
public abstract class AbstractStickyAssignor extends AbstractPartitionAssignor {

    // 파티션 할당의 메인 메서드
    // 모든 컨슈머의 구독이 동일한지 여부에 따라 다른 할당 전략을 사용
    @Override
    public Map<String, List<TopicPartition>> assign(Map<String, Integer> partitionsPerTopic,
                                                    Map<String, Subscription> subscriptions) {
        Map<String, List<TopicPartition>> consumerToOwnedPartitions = new HashMap<>();
        Set<TopicPartition> partitionsWithMultiplePreviousOwners = new HashSet<>();
        if (allSubscriptionsEqual(partitionsPerTopic.keySet(), subscriptions, consumerToOwnedPartitions, partitionsWithMultiplePreviousOwners)) {

            // 모든 컨슈머가 동일한 토픽 세트를 구독 중임을 감지, 최적화된 할당 알고리즘 사용
            log.debug("Detected that all consumers were subscribed to same set of topics, invoking the "
                + "optimized assignment algorithm");

            partitionsTransferringOwnership = new HashMap<>();
            return constrainedAssign(partitionsPerTopic, consumerToOwnedPartitions, partitionsWithMultiplePreviousOwners);
        } else {
            log.debug("일부 컨슈머가 서로 다른 토픽을 구독 중임을 감지, 일반 할당 알고리즘으로 대체");
            partitionsTransferringOwnership = null;
            return generalAssign(partitionsPerTopic, subscriptions, consumerToOwnedPartitions);
        }
    }

    // 제약 기반 할당 메서드
    // 모든 컨슈머가 동일한 토픽을 구독할 때 사용되는 최적화된 할당 알고리즘
    //
    // 1. 이전 소유 파티션 재할당:
    //   - minQuota보다 적게 가진 경우: 모든 파티션 유지
    //   - maxQuota 이상 가진 경우: maxQuota만큼만 유지
    //   - minQuota 이상 가진 경우: minQuota만큼 유지
    // 2. 남은 컨슈머들에 대해 maxQuota 또는 minQuota까지 채움
    private Map<String, List<TopicPartition>> constrainedAssign(Map<String, Integer> partitionsPerTopic,
                                                                Map<String, List<TopicPartition>> consumerToOwnedPartitions,
                                                                Set<TopicPartition> partitionsWithMultiplePreviousOwners) {
        Set<TopicPartition> allRevokedPartitions = new HashSet<>();

        // 아직 파티션을 할당받을 수 있는 컨슈머들을 관리하는 리스트
        List<String> unfilledMembersWithUnderMinQuotaPartitions = new LinkedList<>();
        LinkedList<String> unfilledMembersWithExactlyMinQuotaPartitions = new LinkedList<>();

        // 전체 컨슈머 수
        int numberOfConsumers = consumerToOwnedPartitions.size();

        // 전체 파티션 수
        int totalPartitionsCount = partitionsPerTopic.values().stream().reduce(0, Integer::sum);

        // 컨슈머당 최소/최대 파티션 할당량 계산
        int minQuota = (int) Math.floor(((double) totalPartitionsCount) / numberOfConsumers);
        int maxQuota = (int) Math.ceil(((double) totalPartitionsCount) / numberOfConsumers);

        // minQuota 이상의 파티션을 받을 컨슈머 수 계산
        int expectedNumMembersWithOverMinQuotaPartitions = totalPartitionsCount % numberOfConsumers;
        int currentNumMembersWithOverMinQuotaPartitions = 0;

        // 모든 컨슈머에 대해 빈 할당 맵 초기화
        Map<String, List<TopicPartition>> assignment = new HashMap<>(
            consumerToOwnedPartitions.keySet().stream().collect(Collectors.toMap(c -> c, c -> new ArrayList<>(maxQuota))));

        List<TopicPartition> assignedPartitions = new ArrayList<>();

        // 이전 할당 상태를 기반으로 재할당 수행
        for (Map.Entry<String, List<TopicPartition>> consumerEntry : consumerToOwnedPartitions.entrySet()) {
            // ... (implementation details)
        }

        List<TopicPartition> unassignedPartitions = getUnassignedPartitions(totalPartitionsCount, partitionsPerTopic, assignedPartitions);

        // Round-Robin 방식으로 남은 파티션 할당
        Iterator<String> unfilledConsumerIter = unfilledMembersWithUnderMinQuotaPartitions.iterator();
        for (TopicPartition unassignedPartition : unassignedPartitions) {
            // ... (implementation details)
        }

        return assignment;
    }

    // 일반 할당 메서드
    // 1. 현재 유효한 할당 상태 유지
    // 2. 구독 변경으로 무효화된 할당 제거
    // 3. 미할당 파티션을 전체적으로 균형있게 할당
    // 4. 필요시 재조정을 통해 더 균형잡힌 상태 달성
    private Map<String, List<TopicPartition>> generalAssign(Map<String, Integer> partitionsPerTopic,
                                                            Map<String, Subscription> subscriptions,
                                                            Map<String, List<TopicPartition>> currentAssignment) {
        Map<TopicPartition, ConsumerGenerationPair> prevAssignment = new HashMap<>();
        partitionMovements = new PartitionMovements();

        prepopulateCurrentAssignments(subscriptions, prevAssignment);

        // 토픽과 컨슈머 간의 매핑 초기화
        final Map<String, List<String>> topic2AllPotentialConsumers = new HashMap<>(partitionsPerTopic.keySet().size());
        final Map<String, List<String>> consumer2AllPotentialTopics = new HashMap<>(subscriptions.keySet().size());

        // ... (중간 구현 생략)

        // 재할당 수행
        boolean reassignmentPerformed = performReassignments(sortedPartitions, currentAssignment, prevAssignment,
            sortedCurrentSubscriptions, consumer2AllPotentialTopics, topic2AllPotentialConsumers,
            currentPartitionConsumer, partitionsPerTopic, totalPartitionCount);

        // 재할당 후 밸런스 점수가 개선되지 않았다면 이전 상태로 롤백
        if (!initializing && reassignmentPerformed &&
            getBalanceScore(currentAssignment) >= getBalanceScore(preBalanceAssignment)) {
            deepCopy(preBalanceAssignment, currentAssignment);
            currentPartitionConsumer.clear();
            currentPartitionConsumer.putAll(preBalancePartitionConsumers);
        }

        return currentAssignment;
    }

    // 현재 할당의 밸런스 점수를 계산
    // 모든 컨슈머 쌍 간의 할당된 파티션 수 차이의 합으로 계산
    // 점수가 낮을수록 더 균형 잡힌 상태
    // 완벽하게 균형잡힌 경우 점수는 0
    private int getBalanceScore(Map<String, List<TopicPartition>> assignment) {
        int score = 0;

        Map<String, Integer> consumer2AssignmentSize = new HashMap<>();
        for (Entry<String, List<TopicPartition>> entry: assignment.entrySet())
            consumer2AssignmentSize.put(entry.getKey(), entry.getValue().size());

        Iterator<Entry<String, Integer>> it = consumer2AssignmentSize.entrySet().iterator();
        while (it.hasNext()) {
            Entry<String, Integer> entry = it.next();
            int consumerAssignmentSize = entry.getValue();
            it.remove();
            for (Entry<String, Integer> otherEntry: consumer2AssignmentSize.entrySet())
                score += Math.abs(consumerAssignmentSize - otherEntry.getValue());
        }

        return score;
    }
}
```

위 `assign()` 메서드를 구현한 모든 내용을 요약하면 아래와 같다.

모든 **컨슈머가 동일한 토픽을 구독**할 때 `constrainedAssign()`를 호출한다.
- 이 메서드는 컨슈머당 최소(minQuota)와 최대(maxQuota) 할당량을 계산 한다.
  - 이전 할당 상태를 검토하여 다음과 같이 처리한다.
  - minQuota보다 적게 가진 컨슈머: 기존 파티션 모두 유지
  - maxQuota 이상 가진 컨슈머: maxQuota만큼만 유지하고 나머지는 반환
  - 그 사이인 컨슈머: minQuota만큼 유지
- 남은 파티션들은 Round-Robin 방식으로 재분배한다.

```java
// 전체 컨슈머 수
int numberOfConsumers = consumerToOwnedPartitions.size();

// 전체 파티션 수
int totalPartitionsCount = partitionsPerTopic.values().stream().reduce(0, Integer::sum);

// 컨슈머당 최소/최대 파티션 할당량 계산
int minQuota = (int) Math.floor(((double) totalPartitionsCount) / numberOfConsumers);
int maxQuota = (int) Math.ceil(((double) totalPartitionsCount) / numberOfConsumers);

// minQuota 이상의 파티션을 받을 컨슈머 수 계산
int expectedNumMembersWithOverMinQuotaPartitions = totalPartitionsCount % numberOfConsumers;
int currentNumMembersWithOverMinQuotaPartitions = 0;
```

**컨슈머마다 다른 토픽을 구독**할 때 `generalAssign`를 호출한다.
- 현재 유효한 할당은 최대한 유지한다.
- 구독이 변경된 경우 해당 할당은 제거한다.
- 미할당된 파티션을 분배한다.
- 분배 후 밸런스 점수를 계산하여 필요시 재조정한다.
- 재조정 후 밸런스가 개선되지 않으면 이전 상태로 롤백

---

## 궁금증 해결 1. Sticky 기반의 파티션 할당 전략은 어떻게 균등하게 분배를 하는건가?

Sticky 파티션 할당 전략은 다음과 같은 공식으로 균등 분배를 수행한다.

- minQuota = 반내림(전체 파티션 수 ÷ 전체 컨슈머 수)
- maxQuota = 반올림(전체 파티션 수 ÷ 전체 컨슈머 수)
- 추가 파티션이 필요한 컨슈머 수 == 전체 파티션 수를 전체 컨슈머 수로 나눈 나머지

이 공식을 통해 모든 컨슈머는 최소 minQuota개의 파티션을 할당받고, 나머지 파티션이 있는 경우 일부 컨슈머가 maxQuota개까지 할당받는데

예를 들어 파티션이 7개이고 컨슈머가 3개라면

- minQuota == 2 (내림(7÷3))
- maxQuota == 3 (올림(7÷3))
- 추가 파티션이 필요한 컨슈머 수 == 1 (7%3)

따라서 1개의 컨슈머가 3개, 나머지 2명의 컨슈머가 2개씩 할당받아 최대한 균등하게 분배한다.

## 궁금증 해결 2. 파티션, 컨슈머를 초기에 기동했을 때 파티션을 할당하는 것은 어떻게하는지?

초기 파티션 할당도 동일한 assign() 메서드를 사용하지만, 이전 할당 정보가 없는 상태이므로 Sticky 특성은 사용되지 않을 수 있다.

즉, 초기에는 균등하게 분배되지 않을 수 있다고 **보인다.**

그래도 마냥 불균등하진 않을 것으로 **보이는데** 아래 추가 조건이 있었기 때문이다.

- 모든 컨슈머가 동일한 토픽을 구독하는 경우 constrainedAssign()
  - quota 기반의 분배 공식을 사용해서 각 컨슈머가 minQuota를 채울 때까지 할당하고, 남은 파티션은 maxQuota 범위 내에서 추가 할당
- 컨슈머마다 다른 토픽을 구독하는 경우 generalAssign()
  - 각 파티션별로 구독 중인 컨슈머들을 파악
  - 현재 가장 적은 파티션을 할당받은 컨슈머에게 우선 할당
  - 할당 후 밸런스 점수를 계산하여 최적의 분배 상태 유지
