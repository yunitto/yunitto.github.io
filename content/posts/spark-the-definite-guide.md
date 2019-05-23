---
title: 스파크 완벽 가이드
date: "2019-05-23"
template: "post"
draft: false
slug: "/posts/spark-the-definite-guide/"
category: "Spark"
tags:
- "spark"
- "scala"
description: "<스파크 완벽 가이드: 스파크를 확용한 빅데이터 처리와 분석의 모든 것>"
---
한빛 미디어의 [<스파크 완벽 가이드: 스파크를 확용한 빅데이터 처리와 분석의 모든 것>](https://book.naver.com/bookdb/book_detail.nhn?bid=14300380)을 공부하며 정리한 내용입니다.

---

#### Spark Application은 **driver process**와 **executor process** 로 구성.
- driver process
  - `main()` 함수 실행
  - Application 정보 관리, exceutor process 관리
- executor process
  - driver가 할당한 코드 실행, 다시 보고. 
- JVM 위의 SparkSession 객체가 진입점이 된다.
- Spark API 는 크게 두가지 - structured, unstructured
- 하나의 SparkSession은 하나의 SparkApplication에 대응한다. 즉, 앱 하나당 하나의 드라이버 프로세스가 존재.

```scala
// def range(end: Long): org.apache.spark.sql.Dataset[Long]
// def toDF(colNames: String*): DataFrame
val spark = SparkSession.builder().getOrCreate()
val myRange = spark.range(100).toDF("Number") 
```
클러스터 모드의 경우, 숫자의 범위가 나뉘어서 서로 다른 여러 익스큐터에 할당된다.

---

### Dataframe
- Structured API
- 데이터를 테이블 형식(Row,Column)으로 표시
- Row 단위로 분산

### Partition
- 데이터의 분할 단위 == 클러스터의 물리적 머신에 존재하는 Row의 집합 
- Dataframe의 파티션은 실행 중에 데이터가 물리적으로 분산되는 방식
- 병렬성은 파티션 수와 익스큐터 수에 의해 결정.

## Spark Operations: Transformation and Action

### 1. Transformation
- 데이터가 immutable 하기 때문에 변경 방법을 계획해두는 것.
```scala
// def where(conditionExpr: String): Dataset[Row]
scala> myRange.where("number % 2 == 0") // DF에서 짝수만 찾기
res0: org.apache.spark.sql.Dataset[org.apache.spark.sql.Row] = [number: bigint]
```
- Transformation은 추상적인 변경방법이기 때문에 바로 결과가 나오지 않고, Action을 통해 실행해야한다.

#### Dependency: Narrow vs. Wide
- Narrow Dependency: 1개의 인풋 파티션 1 -> 1개의 아웃풋 파티션
  - `where` : 각 인풋 파티션에서 where()을 하여 아웃풋 파티션이 됨.
  - **메모리**에서만 실행. 파이프라이닝 수행.
- Wide Dependency: 1개의 인풋 파티션 -> n개의 아웃풋 파티션
  -  `shuffle`: 각 인풋 파티션의 데이터가 여러 아웃풋 파티션으로 분산됨.
  - 셔플의 결과를 **디스크**에 저장.

#### Lazy Evaluation(지연 연산)
- Transformation의 실행 계획 생성, 마지막 순간에 컴파일 --> **전체 데이터 흐름 최적화**

#### Predicate Pushdown
- Predicate: boolean(True or False)을 리턴하는 쿼리 조건. SQL의 경우 WHERE clause.
- 필터링을 데이터베이스로 위임하는 쿼리 최적화 기법. 데이터베이스 레벨에서 필터링을 하여 가져오는 레코드 수를 줄이면 처리비용과 시간을 최소화하여 쿼리 성능을 향상시킨다.
- 더 볼것. https://docs.datastax.com/en/dse/6.0/dse-dev/datastax_enterprise/spark/sparkPredicatePushdown.html

