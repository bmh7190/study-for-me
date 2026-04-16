# 오퍼레이팅시스템

운영체제의 구조, 프로세스와 스레드, CPU 스케줄링, 동기화, 교착상태, 메모리 관리를 정리한 노트입니다.

## 목차

- [Operating System](<Operating System.md>)
  - 운영체제 과목의 전체 목차와 학습 흐름을 정리한 인덱스입니다.
- [1. Introduction to OS](<1. Introduction to OS.md>)
  - 운영체제의 역할, 목적, 역사, 시스템 자원 관리 개념을 소개합니다.
- [2. OS Structure](<2. OS Structure.md>)
  - 하드웨어 구조, I/O, interrupt, system call, kernel mode와 user mode, 운영체제 설계 구조를 다룹니다.
- [3. Processes](<3. Processes.md>)
  - 프로세스 상태, PCB, scheduler, context switch, process creation과 IPC의 기본 개념을 정리합니다.
- [4. Threads and Concurrency](<4. Threads and Concurrency.md>)
  - thread 모델, concurrency, multicore 환경, thread library와 관련 이슈를 다룹니다.
- [5. CPU Scheduling](<5. CPU Scheduling.md>)
  - FCFS, SJF, priority, round-robin 등 CPU scheduling 알고리즘과 평가 기준을 정리합니다.
- [6. Synchronization Tools (1)](<6. Synchronization Tools (1).md>)
  - race condition과 critical section 문제를 중심으로 동기화의 필요성과 기본 해결 기법을 다룹니다.
- [7. Synchronization Tools (2)](<7. Synchronization Tools (2).md>)
  - mutex, semaphore 등 동기화 도구를 통해 공유 자원 접근을 제어하는 방법을 정리합니다.
- [8. Monitor](<8. Monitor.md>)
  - monitor와 condition variable을 이용한 고수준 동기화 구조를 다룹니다.
- [9. Synchronization Examples](<9. Synchronization Examples.md>)
  - bounded-buffer, readers-writers, dining philosophers 같은 대표 동기화 문제를 예제로 정리합니다.
- [10. Deadlock](<10. Deadlock.md>)
  - deadlock의 발생 조건, resource-allocation graph, prevention, avoidance, detection, recovery를 다룹니다.
- [11. Memory Management](<11. Memory Management.md>)
  - address binding, MMU, contiguous allocation, segmentation, paging, page table 구조를 정리합니다.
- [12. Virtual Memory](<12. Virtual Memory.md>)
  - demand paging, page fault, page replacement, frame allocation, thrashing, kernel memory allocation을 다룹니다.
