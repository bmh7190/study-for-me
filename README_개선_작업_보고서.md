# README 및 저장소 구조 개선 작업 보고서

## 1. 작업 개요

본 작업은 Obsidian 기반 CS 학습 노트 저장소의 README와 과목별 목차 구조를 개선하기 위해 진행하였다. 기존 저장소는 여러 과목의 Markdown 노트와 이미지 자료가 잘 축적되어 있었지만, 메인 README가 실제 저장소 구조를 충분히 반영하지 못했고, 일부 과목 인덱스 파일과 README의 역할이 중복되어 관리 기준이 명확하지 않았다.

따라서 이번 개선 작업에서는 메인 README를 저장소의 첫 화면 역할에 맞게 정리하고, 과목별 README를 별도 목차 파일로 분리하였다. 또한 파일명과 본문 heading에 남아 있던 철자 오류를 수정하고, 중복 인덱스 파일을 제거하여 향후 유지보수가 쉬운 구조로 정리하였다.

## 2. 작업 목적

이번 작업의 목적은 다음과 같다.

1. 저장소 첫 화면인 메인 README의 가독성을 높인다.
2. 과목별 목차를 각 폴더의 README로 분리하여 관리 기준을 명확히 한다.
3. 오래되었거나 중복된 인덱스 파일을 정리한다.
4. 파일명과 heading의 오타를 수정하여 링크와 문서 품질을 개선한다.
5. GitHub와 Obsidian 양쪽에서 탐색하기 쉬운 노트 구조를 만든다.

## 3. 기존 상태와 문제점

작업 전 저장소는 컴퓨터구조론, 운영체제, 데이터베이스, 병렬프로그래밍, 컴퓨터 네트워크, 컴퓨터보안, 컴파일러 등 여러 과목의 노트로 구성되어 있었다. 그러나 메인 README는 일부 과목만 반영하고 있었고, 최신 폴더 구조와 실제 노트 목록을 모두 포함하지 못했다.

또한 `컴퓨터구조론/Computer Architecture.md`, `오퍼레이팅시스템/Operating System.md`, `데이터베이스/Database.md`와 같은 과목 인덱스 파일이 존재했는데, 이 파일들은 새로 만든 과목별 README와 같은 목차 역할을 하고 있었다. 이로 인해 하나의 과목 목차를 수정할 때 여러 파일을 동시에 관리해야 하는 문제가 있었다.

파일명에도 다음과 같은 오타가 일부 남아 있었다.

- `Sotrage` → `Storage`
- `Theroretical` → `Theoretical`
- `Multiplicaton` → `Multiplication`
- `Proocol` → `Protocol`
- `Forwading` → `Forwarding`
- `Transtion` → `Transition`
- `Sily` → `Silly`
- `Controll` → `Control`

이러한 오타는 README 링크와 GitHub 화면에 그대로 노출되기 때문에, 저장소의 완성도와 신뢰도를 떨어뜨릴 수 있었다.

## 4. 진행 과정

### 4.1 저장소 역할 파악

먼저 저장소의 전체 파일 구조를 확인하였다. 확인 결과 이 저장소는 실행형 코드 프로젝트가 아니라, Obsidian Vault 형태로 관리되는 컴퓨터공학 학습 노트 저장소였다.

주요 구성은 다음과 같았다.

- `README.md`: 저장소 메인 소개 파일
- `images/`: 노트에 포함된 이미지 자료
- `Templates/`: Obsidian 템플릿
- 과목별 폴더: 컴퓨터구조론, 오퍼레이팅시스템, 데이터베이스, 병렬프로그래밍, 컴퓨터 네트워크, 컴퓨터보안, 컴파일러

이 단계에서는 저장소의 목적을 “CS 과목별 학습 노트 모음”으로 파악하고, README를 단순 파일 목록이 아니라 전체 노트의 진입점으로 정리하는 방향을 잡았다.

### 4.2 메인 README와 과목별 README 역할 분리

처음에는 메인 README에 모든 과목의 상세 목차를 직접 넣는 방식으로 정리하였다. 그러나 이 방식은 메인 README가 너무 길어지고, 새 노트가 추가될 때마다 메인 파일을 계속 수정해야 한다는 문제가 있었다.

따라서 구조를 다음과 같이 변경하였다.

- 메인 README: 과목별 README로 이동할 수 있는 전체 진입점
- 과목별 README: 해당 과목의 상세 목차와 간단한 설명 관리

이 방식은 과목별 변경이 해당 폴더 안에서만 이루어지도록 하여 유지보수성을 높인다.

이 작업은 다음 커밋으로 정리되었다.

```text
1009b4e docs: split README indexes by subject
```

### 4.3 과목별 README에 요약 설명 추가

처음 만든 과목별 README는 목차 링크만 포함하고 있어 깔끔하지만 정보량이 부족했다. 각 항목이 어떤 내용을 다루는지 한눈에 파악하기 어렵다는 문제가 있었다.

이를 개선하기 위해 각 목차 항목 아래에 1문장 정도의 간단한 설명을 추가하였다. 설명은 하위 bullet 형태로 들여쓰기하여, 링크와 설명이 시각적으로 구분되도록 구성하였다.

예시는 다음과 같다.

```markdown
- [15-4. A TCP Connection](<TCP/15-4 A TCP Connection.md>)
  - three-way handshake, data transfer, connection termination, half-close 흐름을 정리합니다.
```

이렇게 구성한 이유는 다음과 같다.

- 목차의 가독성을 유지하면서도 각 노트의 내용을 빠르게 파악할 수 있다.
- GitHub에서 렌더링했을 때 링크와 설명이 분리되어 보인다.
- 과목별 README가 단순 링크 목록이 아니라 실제 학습 안내 역할을 할 수 있다.

### 4.4 README 링크 검증

과목별 README를 만든 뒤에는 모든 README 링크가 실제 파일을 가리키는지 검증하였다. 파일명에 공백, 괄호, 한글이 포함된 경우가 많았기 때문에 Markdown 링크가 깨질 가능성이 있었다.

이를 방지하기 위해 링크 경로를 `<...>` 형태로 작성하였다.

예시는 다음과 같다.

```markdown
[컴퓨터 네트워크](<컴퓨터 네트워크/README.md>)
```

검증 결과 모든 README 링크가 정상적으로 존재하는 파일이나 폴더를 가리키는 것을 확인하였다.

```text
All README links exist.
```

### 4.5 파일명 및 heading 오타 수정

README가 정리된 뒤에는 파일명과 본문 heading에 남아 있던 오타를 점검하였다. 파일명 오타는 GitHub 화면과 README 링크에 직접 노출되므로 우선적으로 수정하였다.

수정한 대표 항목은 다음과 같다.

| 수정 전 | 수정 후 |
|---|---|
| `6. Sotrage and File Structure.md` | `6. Storage and File Structure.md` |
| `2. Theroretical Background` | `2. Theoretical Background` |
| `4-2 Matrix-Vector Multiplicaton.md` | `4-2 Matrix-Vector Multiplication.md` |
| `2-3 TCP IP Proocol suite.md` | `2-3 TCP IP Protocol suite.md` |
| `Delivery Forwading` | `Delivery Forwarding` |
| `15-5 State Transtion Diagram.md` | `15-5 State Transition Diagram.md` |
| `15-7-1 Sily Window Syndrome.md` | `15-7-1 Silly Window Syndrome.md` |
| `15-9 Congestion Controll.md` | `15-9 Congestion Control.md` |

또한 본문 heading에 있던 `Alogrithm`, `Receving`, `Instuction`, `Imediate`, `Proceduer`, `Synchoronization`, `Arithemtic`, `Mudulo`, `Unpredicability` 등의 오타도 함께 수정하였다.

이 작업은 다음 커밋으로 정리되었다.

```text
86126b6 docs: fix note title typos
```

### 4.6 중복 인덱스 파일 제거

과목별 README가 상세 목차 역할을 하게 되면서, 기존에 같은 역할을 하던 인덱스 파일들이 중복되었다.

중복된 파일은 다음과 같았다.

- `컴퓨터구조론/Computer Architecture.md`
- `오퍼레이팅시스템/Operating System.md`
- `데이터베이스/Database.md`

처음에는 삭제하지 않고 README로 안내하는 짧은 파일로 바꾸는 방식을 검토하였다. 그러나 해당 파일들이 다른 노트에서 참조되지 않고, 과목별 README와 완전히 역할이 겹친다는 점을 확인한 뒤 최종적으로 삭제하였다.

이렇게 정리한 이유는 다음과 같다.

- 목차 관리 기준을 각 과목 폴더의 `README.md` 하나로 통일할 수 있다.
- 오래된 인덱스 파일이 남아 있어 사용자가 혼동하는 문제를 줄일 수 있다.
- 새로운 노트가 추가될 때 수정해야 하는 파일 수를 줄일 수 있다.

이 작업은 다음 커밋으로 정리되었다.

```text
0c2bebd docs: remove duplicate subject indexes
```

### 4.7 메인 README 디자인 개선

마지막으로 메인 README가 저장소 첫 화면으로 보기에는 다소 단조롭다는 문제가 있었다. 이를 개선하기 위해 GitHub에서 보기 좋은 형태로 다음 요소를 추가하였다.

- 중앙 정렬된 제목 영역
- Markdown, Obsidian, 진행 상태 배지
- 섹션 제목의 이모지
- 과목별 아이콘이 포함된 표
- Mermaid 기반 학습 지도
- 저장소 구조를 보여주는 텍스트 트리

메인 README는 단순한 설명 파일이 아니라 저장소의 첫인상이므로, 방문자가 저장소의 목적과 과목 구성을 빠르게 이해할 수 있도록 구성하였다.

이 작업은 다음 커밋으로 정리되었다.

```text
d38edb2 docs: improve main README presentation
```

## 5. 커밋 순서

이번 개선 작업은 다음 순서로 진행되었다.

| 순서 | 커밋 | 작업 내용 |
|---|---|---|
| 1 | `1009b4e` | 메인 README와 과목별 README 역할 분리 |
| 2 | `86126b6` | 파일명 및 본문 heading 오타 수정 |
| 3 | `0c2bebd` | 중복 과목 인덱스 파일 제거 |
| 4 | `d38edb2` | 메인 README 첫 화면 디자인 개선 |

각 커밋은 작업 목적별로 분리하였다. 이렇게 한 이유는 나중에 변경 내역을 추적하거나 특정 작업만 되돌릴 때 더 명확하게 관리할 수 있기 때문이다.

## 6. 검증 내용

작업 후 다음 항목을 검증하였다.

1. README 링크가 실제 파일 또는 폴더를 가리키는지 확인하였다.
2. 삭제한 중복 인덱스 파일을 참조하는 링크가 남아 있지 않은지 확인하였다.
3. 수정한 오타 패턴이 Markdown 파일명과 본문에 남아 있지 않은지 확인하였다.
4. 커밋 후 GitHub 원격 저장소에 push하여 로컬과 원격 상태가 동기화되었는지 확인하였다.

최종 상태는 다음과 같다.

```text
main...origin/main
```

이는 로컬 저장소와 원격 저장소가 동기화되어 있음을 의미한다.

## 7. 작업 결과

작업 결과 저장소 구조는 다음과 같이 개선되었다.

- 메인 README는 저장소 전체의 첫 화면 역할을 한다.
- 각 과목 폴더의 README는 해당 과목의 상세 목차 역할을 한다.
- 중복 인덱스 파일을 제거하여 목차 관리 기준이 명확해졌다.
- 파일명과 heading 오타를 수정하여 문서 품질이 개선되었다.
- GitHub에서 저장소를 열었을 때 과목 구성과 읽는 방법을 더 쉽게 파악할 수 있게 되었다.

## 8. 기대 효과

이번 개선을 통해 다음과 같은 효과를 기대할 수 있다.

1. 저장소를 처음 보는 사람도 전체 구조를 빠르게 이해할 수 있다.
2. 과목별 README를 기준으로 노트를 탐색할 수 있어 접근성이 좋아진다.
3. 중복 목차 파일이 사라져 유지보수 부담이 줄어든다.
4. 오타가 줄어들어 GitHub에 공개했을 때 문서 완성도가 높아진다.
5. 이후 GitHub Pages나 Quartz를 이용해 웹사이트 형태로 확장할 때도 더 안정적인 기반이 된다.

## 9. 결론

이번 작업은 단순히 README를 꾸미는 작업이 아니라, 저장소의 문서 구조와 탐색 흐름을 정리하는 작업이었다. 메인 README와 과목별 README의 역할을 분리하고, 중복 파일과 오타를 정리함으로써 저장소의 가독성, 유지보수성, 공개 문서로서의 완성도를 높였다.

앞으로 새로운 노트가 추가될 경우, 해당 과목 폴더의 `README.md`에만 목차를 추가하면 되므로 관리 방식도 단순해졌다. 따라서 이번 개선 작업은 현재 저장소를 더 체계적인 CS 학습 노트 저장소로 정리하는 기반 작업이라고 볼 수 있다.
