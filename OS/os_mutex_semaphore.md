# 공유 자원 & 임계 영역(Semaphore & Mutex)

## 동기화(Synchronization)
- 여러 프로세스/스레드를 동시에 실행해도 공유 데이터의 일관성을 유지하는 것
    - **실행 순서 제어** : 프로세스를 올바른 순서대로 실행
    - **상호 배제** : 동시에 접근해서는 안 되는 자원에 하나의 프로세스만 접근
    


## 공유 자원과 임계 구역

### 공유 자원(Share Section)

- 공유 자원은 여러 프로세스나 스레드가 동시에 접근하거나 사용할 수 있는 자원
- 전역 변수, 파일, 입출력장치, 보조기억장치, 네트워크 소켓
- 특징
    - 자원을 공유할 경우, 잘못된 동기화로 인해 데이터의 일관성이 깨질 수 있다.
    - 예: 한 스레드가 공유 자원을 읽는 동안 다른 스레드가 해당 자원을 수정하면 데이터가 손상될 수 있다.  👉 **경쟁 상태(Race Condition)**
    

> **경쟁 상태(Race Condition)** 
두 개 이상의 스레드가 동시에 공유 자원에 접근하면서 올바른 실행 순서가 보장되지 않아 데이터 불일치 또는 손상이 발생하는 것
(예: 은행 계좌 입금 처리 중 두 명이 동시에 입금하면 최종 금액이 올바르지 않을 수 있음)
> 

### 임계 구역(Critical Section)

- 공유 자원에 접근하고 수정하는 코드 영역
- 자원의 일관성을 위해 한 번에 하나의 스레드만 실행할 수 있어야 한다.  → 상호 배제
- 임계 구역을 보호하기 위한 대표적인 도구로 뮤텍스(Mutex)와 세마포(Semaphore)가 있다.

![](/OS/img/os_mutex_semaphore_1.png)

## **Mutex**

![](/OS/img/os_mutex_semaphore_2.png)

- **한 번에 하나의 스레드**만 공유 자원(임계 영역)에 접근할 수 있도록 보장하는 동기화 도구
- 특징
    - 스레드가 락을 획득하면 다른 스레드는 대기.
    - 사용 후 반드시 **Unlock**해야 함.
    - 주로 단일 자원의 보호에 사용.
    

### **동작 원리**

1. **Lock**
    - 스레드는 임계 영역에 접근하기 전에 뮤텍스에 락을 요청한다.
    - 락이 획득되면 스레드는 자원에 접근한다.
2. **임계 영역 실행**
    - 스레드는 공유 자원을 사용하거나 처리한다.
3. **Unlock**
    - 스레드가 자원 사용을 마치면 락을 해제하여 다른 스레드가 자원을 사용할 수 있도록 한다.
4. **대기(WAIT)**
    - 만약 자원이 이미 다른 스레드에 의해 락이 걸려 있다면, 새로운 스레드는 해제될 때까지 대기 한다.

```java
acquire();

{ 임계 구역 }

release();
```

```java
acquire() {
	while(lock == true) {}
	
	lock = true;
	// 임계 구역에 진입
}

release() {
	lock == false;
	// 임계 구역 해제
}
```

**acquire()** 

- 락 요청. 자원 사용 가능 시 진입.

**release()** 

- 락 해제. 다른 스레드가 접근 가능하도록 허용.

## **Semaphore**

![](/OS/img/os_mutex_semaphore_3.png)


- 하나 이상의 스레드가 자원에 접근할 수 있도록 제한하는 동기화 도구
- 공유 자원이 여러 개 있는 경우에도 적용 가능
- 특징
    - **카운터 기반**: 특정 개수의 허용 가능한 자원 개수를 관리한다.
    - 프로세스의 순서를 정할 수 있다.

### 동작 원리

```java
int value = 3; // 사용 가능한 공유 자원의 개수

wait();

{ 임계 구역 }

signal();
```

```java
wait() {
	while(value <= 0) {}
	
	value--;
	// 자원 사용
}

signal() {
	value++;
}
```

**wait()**

- 자원이 없으면 대기 상태로 진입.
- 자원이 있으면 카운터 감소 후 접근 허용.

**signal()**

- 자원을 반환하고 카운터 증가.



| 항목 | Mutex | Semaphore |
| --- | --- | --- |
| **자원 개수** | 1개 (Binary, 락/언락) | 여러 개 (카운트 기반) |
| **사용 목적** | 단일 공유 자원 보호 | 제한된 자원 관리 |
| **소유권** | 소유권 있음 (락을 획득한 스레드만 해제) | 소유권 없음 (누구나 해제 가능) |

상호 배제만 필요하다면 뮤텍스를, 작업 간의 실행 순서 동기화가 필요하다면 세마포를 권장한다.

---

## 사용 사례

- **Mutex**
    - 파일 시스템 접근
        - 하나의 프로세스가 파일을 읽거나 수정할 때 다른 프로세스 접근 차단.
    - 프린터 공유
        - 출력 장치 접근을 순차적으로 관리.
- **Semaphore**
    - 서버 연결 풀 관리
        - DB 연결을 제한된 개수로 유지하여 과부하 방지.
    - 리소스 대기열 관리
        - 제한된 자원(예: API 호출)에 대한 접근 동시 처리.

---
 


참고

https://hapen385.tistory.com/55

https://www.youtube.com/watch?v=0YDjblJn30k

https://www.youtube.com/watch?v=gTkvX2Awj6g

https://www.youtube.com/watch?v=4u13f9Umq7Y&t=58s

https://mierminusone.tistory.com/4

https://one-armed-boy.tistory.com/entry/Mutex-Semaphore

https://www.javatpoint.com/mutex-vs-semaphore