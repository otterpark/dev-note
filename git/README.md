# Git 이란?

Git은 `분산 버전 관리 시스템(DVCS)`으로 단순히 파일의 마지막 스냅샷을 Checkout 하지 않고, 저장소 히스토리와 더불어 모두 복제한다. 만약 서버에 문제가 생겼을 때 복제물로 다시 작업을 진행하거나 복원할 수 있다. 추가적으로 `분산 버전 관리 시스템(DVCS) 환경`에서는 리모트 장소가 존재하며, 여러 사람이 동시에 다양한 그룹과 방법으로 협업할 수 있다.

- `버전 관리 시스템`: 파일 변화를 시간에 따라 기록했다가 나중에 특정 시점의 버전을 다시 꺼내올 수 있는 시스템

## Git의 목표

Git은 2005년 탄생 후 초기 목표를 그대로 유지하며 쉽게 진화하고 성숙해졌다.

- 빠른 속도
- 단순한 구조
- 비선형적인 개발(수천 개의 동시 다발적인 브랜치)
- 완벽한 분산
- Linux 커널 같은 대형 프로젝트에도 유용할 것(속도나 데이터 크기 면에서)

## Git 기초

### 1. **차이가 아니라 스냅샷**

Git은 이전 상태의 파일에 대한 링크만 저장하며, 데이터를 파일 시스템 스냅샷의 연속으로 취급하고 크기가 매우 작다. Commit이나 프로젝트 상태를 저장할 때마다 파일이 중요하게 느끼며, 파일이 달라지지 않았다면 Git은 성능을 위해 파일을 새로 생성하지 않는다. Git은 데이터를 `스냅샷의 스트림`처럼 취급한다.

### 2. **모든 명령을 로컬에서 실행**

Git은 로컬 환경에서 모든 일들을 다 할 수 있다. 로컬 환경이기에 매우 속도가 빠르며, 프로젝트 진행 시 히스토리를 서버없이 조회가 가능하다.

### 3. **Git의 무결성**

Git은 데이터를 저장하기 전에 항상 `체크섬`을 구하고 `체크섬`으로 데이터를 관리한다. Git은 `SHA-1 해시`를 사용하여 `체크섬`을 만들며 Git은 파일을 이름으로 저장하지 않고 해당 파일의 해시로 저장한다.

- `체크섬` - 파일 변화를 시간에 따라 기록했다가 나중에 특정 시점의 버전을 다시 꺼내올 수 있는 시스템

### 4. **Git은 데이터를 추가할 뿐(중요)**

Git은 무얼 하든 Git 데이터베이스에 데이터가 추가 된다. Git은 `Committed`, `Modified`, `Staged` 이렇게 세 가지 상태를 관리한다.

- `Committed` - 데이터가 로컬 데이터베이스에 안전하게 저장
- `Modified` - 수정한 파일을 아직 로컬에서 커밋하지 않은 상태
- `Staged` - 현재 수정한 파일을 곧 커밋할 것이라고 표시한 상태

![git areas](./images/git-areas.png)

위 그림을 보면서 이야기를 해보자.

#### `.git directory`

프로젝트의 메타데이터와 객체 데이터베이스를 저장하는 곳이며, 다른 컴퓨터에 있는 장소를 `Clone` 할 대 Git 디렉토리가 생성된다.

#### `워킹 트리`

프로젝트의 특정 버전을 Checkout 한 것이다. Git 디렉토리는 지금 작업하는 디스크가 있고 그 디렉토리 안에 압축된 데이터베이스에서 파일을 가져와서 워킹 트리를 만듭니다.

#### `Staging Area`

Git 디렉토리 안에 있으며, 단순한 파일이고 곧 커밋할 파일에 대한 정보를 저장한다.

**Git이 하는 일은 아래와 같다.**

- 워킹 트리에서 파일을 수정한다.
- `Staging Area`에 파일을 Stage 해서 커밋할 스냅샷을 만들고 모든 파일을 추가 또는 선택하여 추가할 수 있다.
- `Staging Area`에 있는 파일들을 커밋해서 Git 디렉토리에 영구적인 스냅샷을 저장한다.

그럼 한번에 정리 해보자.

1. Git 디렉토리에 있는 파일들은 `Committed` 상태
2. 파일을 수정하고 `Staging Area`에 추가했다면 `Staged`
3. 아직 `Staging Area`에 추가하지 않았으면 `Modified`