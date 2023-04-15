# Git CLI

Git에서 주로 사용하는 CLI에 대해서 알아보자!

## `git init`(Git 저장소 생성)

```bash
$ git init
```

위 명령어를 실행하면 `.git` 하위 디렉토리를 생성한다. \
생성 후 어떠한 프로젝트의 관리를 하지 않다가, `git add` 명령으로 파일을 추가하고 `git commit` 명령으로 커밋을 하면 파일 관리가 시작된다.

## `git clone <url>`(Git 저장소를 복사)

```bash
$ git clone <url>
```

`git clone` 실행 시 서버에 있는 모든 데이터를 복사(프로젝트 전체 히스토리)한다.

## `git status`(Git 파일 상태 확인하기)

```bash
$ git status
On branch main
Your branch is up-to-date with 'origin/main'.
nothing to commit, working directory clean
```

위 내용을 해석하면 아래와 같다.

```text
현재 브랜치는 main이며,
'origin/main' 브랜치로부터 업데이트 됐으며, 어떠한 커밋이나 진행된 작업이 없다.
```

## `git add`(Git 추가)

```bash
$ git add (파일명)     # 특정 파일 Index(Staging area)에 추가
$ git add .          # 현재 및 하위 디렉토리 모든 파일 index 추가
```

`git add` 시 `Tracked`상태이면서 커밋에 추가될 `Staged` 상태가 된다.

## `git commit`(Git 변경사항 커밋)

```bash
$ git commit -m (커밋 메세지)    # -m 욥션 추가: 커밋 메세지 작성 가능
```

`Git 기초`에서 배운대로 `Git`은 `Staging Area`에 속한 스냅샷을 커밋한다. 수정은 했지만, 아직 `Staging Area`에 넣지 않은 것은 다음에 커밋이 가능하다.
> `커밋`할 때마다 프로젝트의 스냅샷을 기록하기 때문에 비교 또는 예전 스냅샷으로 되돌릴 수 있다.

## `git rm`(Git 파일 제거)

```bash
git rm (파일명)               # 'Staging Area'에서 파일 삭제
git rm --cached(파일명)       # 'Staging Area`에서만 삭제 후 파일 디렉토리에 파일은 존재하게 할 수 있음
```

`git rm` 을 사용하여, `Tracked` 상태의 파일을 삭제한 후(`Staging Aea`에서 삭제하는 것) 커밋 해야한다.

추가적으로 이미 파일을 수정했거나 `Staging Area`에 추가했다면 `-f` 옵션을 주어 강제로 삭제해야 한다.

> 만약 `Staging Area` 에서만 제거하고 워킹 디렉토리에 있는 파일은 지우고 싶지 않다면 `--cached` 옵셕을 주어 Local의 파일은 그대로이며 Git이 추적 못하게 할 수 있다.

## `git log`(Git 커밋 이력보기)

```bash
$ git log
```

`git log`는 저장소의 커밋 히스토리를 시간순으로 보여준다. 가장 최근 커밋을 내림차순으로 정렬되어 나온다. `git log`에 여러 옵션 중 유용한 몇 개만 아래에 적어놓았다.

- `git log -p` - 각 커밋의 diff 결과를 보여줌
- `git log --stat` - 각 커밋의 통계 정보를 조회 가능

## `git commit --amend`(Git 커밋 되돌리기)

```bash
$ git commit --amend -m (메세지)    # -m 욥션 추가: 커밋 수정에 대한 메세지 작성
$ git commit --amend --no-edit    # --no-edit 욥션 추가설명에 대해 수정을 하지 않을 때 사용
```

`git commit --amend`는 일을 진행하다 어떤 것을 되돌리고 싶을 때 사용한다. 주의해야할 점은 되돌리기는 `가능`하지만, 한번 되돌리면 복구가 `불가능`하다.

> 사용 예) 조금 전 한 커밋에 실수가 있다, 오타를 살짝만 고쳤다, 빠진 파일을 넣었다 등
