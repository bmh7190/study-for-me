
## 멀티프로세스 서버와 좀비 프로세스

앞에서 본 서버 구조는 **에코 서버**로, 한 번에 하나의 클라이언트와만 연결할 수 있었다.  
하지만 실제 서버는 여러 클라이언트와 동시에 연결되어야 한다.  
즉, **멀티프로세스** 구조로 동작해야 한다.

---

### 좀비 프로세스

프로세스가 실행을 종료하면, 그에 대한 정보도 메모리에서 사라져야 한다.  
그러나 실제로는 그렇지 않다. 프로세스가 종료되면 커널은 그 프로세스의 **종료 상태와 사용 리소스 정보를 보관**하며, 프로세스는 **좀비(Zombie)** 상태가 된다.  
이 정보는 부모 프로세스가 확인(`wait()` 또는 `waitpid()`)할 때까지 남아 있게 된다.

부모 프로세스가 자식 프로세스의 종료를 처리하지 않으면,  
자식의 리소스가 메모리에 계속 남아 있게 되어 **리소스 낭비**가 발생한다.  
이 현상을 방지하기 위해 부모는 반드시 **wait 함수**를 호출해야 한다.

---

### `wait()` 함수

`wait()` 함수는 자식 프로세스가 종료될 때까지 **블로킹(대기)** 상태에 있다가,  
자식이 종료되면 그 정보를 수거하고, 커널은 자식의 PCB를 완전히 삭제한다.  
단점은 블로킹이기 때문에, 부모가 다른 작업을 진행할 수 없다는 점이다.

### `waitpid()` 함수

이 문제를 해결하기 위해 `waitpid()`를 사용할 수 있다.

```c
pid_t waitpid(pid_t pid, int* statloc, int options);
```

- `pid`: 특정 프로세스 ID를 지정한다. `-1`을 넣으면 **아무 자식이나** 대기한다.
    
- `statloc`: 자식의 종료 상태를 담을 변수 주소.
    
- `options`: 대기 방식 설정.
    
    - `0`: 블로킹 대기 (기본)
        
    - `WNOHANG`: 종료된 자식이 없으면 바로 0 반환 → **비블로킹 대기**
        

`waitpid()`는 블로킹되지 않고 종료된 자식만 수거할 수 있으므로, 서버 같은 **지속적인 동작 환경**에서 유용하다.

---

## 시그널 (Signal)

시그널은 **운영체제가 특정 상황을 프로세스에게 알리는 알림 기능**이다.  
우리가 사용할 시그널은 **`SIGCHLD`**, 즉 **자식 프로세스가 종료되었을 때** 발생하는 시그널이다.

### signal() 함수

```c
void (*signal(int signo, void (*func)(int)))(int);
```

기존의 `signal()` 함수는 단순하지만, 이식성과 세밀한 제어가 떨어진다.  
그래서 실무에서는 **`sigaction()`** 을 사용하는 것이 일반적이다.


### sigaction() 함수

```c
int sigaction(int signo, const struct sigaction* act, struct sigaction* oldact);
```

#### 구조체 정의

```c
struct sigaction
{
    void (*sa_handler)(int);
    sigset_t sa_mask;
    int sa_flags;
};
```

`sigaction()`은 특정 시그널(`signo`)이 발생했을 때 실행할 함수를 미리 등록한다.  
예를 들어, `SIGCHLD`를 감지해서 자식이 종료되면 특정 함수를 자동 실행할 수 있다.

```c
sigaction(SIGCHLD, &act, 0);
```

---

### 예제 코드

```c
void read_childproc(int sig)
{
    int status;
    pid_t id = waitpid(-1, &status, WNOHANG);
    if (WIFEXITED(status))
    {
        printf("Removed proc id: %d \n", id);
        printf("Child send: %d \n", WEXITSTATUS(status));
    }
}
```

위 함수는 **자식 프로세스가 종료될 때 자동으로 호출되는 함수**다.  
핸들러가 실행되면 `waitpid()`를 호출해 자식의 종료 정보를 수거한다.

- `status`에는 자식의 종료 정보가 기록된다.
    
    - 정상 종료 시: `exit()` 또는 `return` 값이 들어감
        
    - 비정상 종료 시: 어떤 시그널로 죽었는지 정보가 들어감
        
- `WIFEXITED(status)` : 정상 종료 여부 확인
    
- `WEXITSTATUS(status)` : 자식이 `exit()` 또는 `return`으로 넘긴 값 반환
    

---

### 동작 정리

- 자식 프로세스가 종료되면 커널이 부모에게 **`SIGCHLD` 시그널**을 보냄
    
- 부모 프로세스는 `sigaction`을 통해 등록해둔 함수(`read_childproc`)를 실행
    
- 이 함수 내에서 `waitpid()`를 호출 → 자식 종료 정보 수거
    
- 커널은 자식의 PCB를 완전히 삭제 → **좀비 프로세스 방지**
    

자식 프로세스가 종료되면 커널은 그 정보를 잠시 남겨둔다. 부모가 `wait()` 또는 `waitpid()`를 호출해야 완전히 삭제되며, 호출하지 않으면 **좀비 프로세스**로 남는다.

따라서 `sigaction(SIGCHLD, ...)`을 이용해 자식이 종료되자마자 `waitpid()`를 자동으로 호출하게 만들면 좀비 프로세스가 남지 않고 바로 소멸된다.
