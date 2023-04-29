# Git Convention

`Commit Convention`이란 쉽게 `commit`할 때 `commit message`에 대한 개발자들끼리의 약속이다. 현업에서 협업을 할 때, 또는 본인이 `commit message`를 쉽게 알아보기 위한 이유에서 커밋 컨벤션이 **필요**하다.

## Commit Convention Structure

```bash
type: Subject

body

footer
```

커밋 메시지 구조는 크게 3가지로 나뉜다

- `제목` - `type: Subject`
- `본문` - `body`
- `꼬리말` - `footer`

## Commit Type

|Type|**설명**|
|:--:||:--:|
|Feat|새로운 기능 추가|
|Fix|버그 수정|
|Refactor|리팩토링|
|Design|CSS 등 사용자 UI 디자인 변경|
|Comment|필요한 주석 추가 및 변경|
|Style|코드 포맷팅, 세미콜론 누락, 코드 변경이 없는 경우|
|Test|테스트(테스트 코드 추가, 수정, 삭제, 비즈니스 로직에 변경이 없는 경우)|
|Chore|위에 걸리지 않는 기타 변경사항(빌드 스크립트 수정, assets image, 패키지 매니저 등)|
|Init|프로젝트 초기 생성|
|Rename|파일 혹은 폴더명 수정하거나 옮기는 경우|
|Remove|파일을 삭제하는 작업만 수행하는 경우|

## Subject

- 제목은 **50자**를 넘기면 안 되고, **대문자로 작성한다**.
- **과거시제**를 사용하지 않고 **명령어**로 작성한다.

  - `Added` -> `Add`
  - `Fixed` -> `Fix`
- **마침표 및 특수기호** 사용하지 않아야한다.
- 개조식 구문으로 작성해야한다.(간결하고 요점적인 서술)

## Body

- `Body`는 선택사항으로 한 줄당 **72자 내**로 작성하고 `Subject`와 구분을 위해 한칸을 띄워서 작성한다.
- **부연 설명**이 필요하거나, **커밋의 이유**를 설명할 때 작성한다.
- 어떻게 보다는 **'무엇을', '왜'** 변경했는지에 대해 작성한다.

## Footer

- `유형: #이슈 번호`의 형식으로 작성한다.
- 이슈 트래커 ID를 작성한다.
- 여러개의 이슈 번호는 `,`로 구분한다.
- 이슈 트래커 유형은 아래와 같다

|Type|**설명**|
|:--:||:--:|
|Issue Tracker|설명|
|Fixes|이슈 수정중(아직 해결되지 않은 경우)|
|Resolves|이슈를 해결한 경우|
|Ref|참조할 이슈가 있을 때 사용|
|Related to|해당 커밋에 관련된 이슈 번호(아직 해결되지 않은 경우)|

ex) Footer에 `Fixes: #1` 이라고 작성하고 commit을 할 경우, 자동으로 issue #1과 매칭이 된다.

또한, `Resolves: #1`으로 이슈를 해결했다고 명시하면 그 이슈는 사라진다.

## Commit Template

```bash
# 제목은 최대 50글자까지 아래에 작성: ex) Feat: Add Key mapping  
# 본문은 아래에 작성  
# 꼬릿말은 아래에 작성: ex) Github issue #23  
# --- COMMIT END ---  
#   <타입> 리스트  
#   feat        : 기능 (새로운 기능)  
#   fix         : 버그 (버그 수정)  
#   refactor    : 리팩토링  
#   design      : CSS 등 사용자 UI 디자인 변경  
#   comment     : 필요한 주석 추가 및 변경  
#   style       : 스타일 (코드 형식, 세미콜론 추가: 비즈니스 로직에 변경 없음)  
#   docs        : 문서 수정 (문서 추가, 수정, 삭제, README)  
#   test        : 테스트 (테스트 코드 추가, 수정, 삭제: 비즈니스 로직에 변경 없음)  
#   chore       : 기타 변경사항 (빌드 스크립트 수정, assets, 패키지 매니저 등)  
#   init        : 초기 생성  
#   rename      : 파일 혹은 폴더명을 수정하거나 옮기는 작업만 한 경우  
#   remove      : 파일을 삭제하는 작업만 수행한 경우  
# ------------------  
#   제목 첫 글자를 대문자로  
#   제목은 명령문으로  
#   제목 끝에 마침표(.) 금지  
#   제목과 본문을 한 줄 띄워 분리하기  
#   본문은 "어떻게" 보다 "무엇을", "왜"를 설명한다.  
#   본문에 여러줄의 메시지를 작성할 땐 "-"로 구분  
# ------------------  
#   <꼬리말>  
#   필수가 아닌 optional  
#   Fixes        : 이슈 수정중 (아직 해결되지 않은 경우)  
#   Resolves     : 이슈 해결했을 때 사용  
#   Ref          : 참고할 이슈가 있을 때 사용  
#   Related to   : 해당 커밋에 관련된 이슈번호 (아직 해결되지 않은 경우)  
#   ex) Fixes: #47 Related to: #32, #21 
```

## Commit Message 적용하기

1. 본인이 원하는 경로에 `git-message.txt` 파일을 생성한다.
2. 내용은 위 템플릿 내용을 넣는다.
3. 터미널에 명령을 입력한다.

```bash
git config --global commit.template (git-message.txt 위치)
```

이 설정을 통해 앞으로 `git commit` 명령어를 통해 앞전에 만들었던 `Commit Template`의 메시지가 보인다.
