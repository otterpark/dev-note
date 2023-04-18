
# Git 리모트(Remote) 저장소

리모트 저장소를 관리할 수 있어야 다른 사람과 협업을 할 수 있다. 리모트 장소는 인터넷이나 네트워크 어딘가 있는 저장소라고 칭한다. 저장소는 여러 개가 있을 수 있고 다른 사람들과 함께 일한다는 것은 리모트 저장소를 관리하면서 데이터를 저장소에 `Push` 및 `Pull`을 하는 것이다.

명령어는 아래와 같다.

```bash
$ git remote
origin
```

여기에서 `-v` 옵션을 추가하면 단축 이름과 url을 함께 볼 수 있으며, 저장소가 여러 개라면 등록된 전부를 보여준다.

```bash
$ git remote -v
origin https://github.com/otterpark/gitbook-dev (fetch)
origin https://github.com/otterpark/gitbook-dev (push)
```

## 리모트 저장소 추가하기(`git remote add <단축 이름> <url>`)

```bash
$ git remote
origin
$ git remote add park https://github.com/jinwoopark/gitbook-dev
$ git remote -v
origin https://github.com/otterpark/gitbook-dev (fetch)
origin https://github.com/otterpark/gitbook-dev (push)
park https://github.com/jinwoopark/gitbook-dev (fetch)
park https://github.com/jinwoopark/gitbook-dev (push)
```

위와 같이 사용 시 `url` 대신 `park` 라는 이름을 사용할 수 있다. 다음 내용에서 배울 내용인데 로컬 저장소에 없지만 `park` 저장소에 있는 것을 가져오려면 아래와 같이 사용하면 된다.

```bash
$ git fetch park
remote: Counting objects: 43, done.
remote: Compressing objects: 100% (36/36), done.
remote: Total 43 (delta 10), reused 31 (delta 5)
Unpacking objects: 100% (43/43), done.
From https://github.com/jinwoopark/gitbook-dev
 * [new branch]      master          -> park/master
 * [new branch]      gitbook-dev     -> park/gitbook-dev
```

## 리모트 저장소를 pull 하거나 fetch 하기

```bash
git fetch <remote>
```

위 명령어는 현재 로컬에는 없지만, 리모트 장소에 있는 데이터를 모두 가져온다. 이후 모든 리모트 장소의 브런치를 로컬에서 접근할 수 있다.
> `git fetch` 명령은 해당 리모트의 저장소의 데이터를 로컬로 가져오지만, 자동으로 Merge 하지 않는다. (수동 Merge 필요)

## 리모트 저장소에 Push 하기

프로젝트 공유 시 Upstram 저장소에 Push 할 수 있다.

```bash
git push <리모트 저장소 이름> <브랜치 이름>
```

위 명령어는 해당 리모트 저장소에 쓰기 권한이 있어야 하며, remote clone 이후 아무도 Upstream 저장소에 Push 하지 않았을 때만 사용 가능하다.
> 만약 다른 사람이 먼저 작업하여 Push를 진행 시 파일을 가져온 후 Merge 해야한다.

## 리모트 저장소 이름 바꾸기

```bash
git remote rename <바꾸려는 리모트 저장소 이름> <바꾸고 싶은 리모트 저장소 이름>
```

예를 들어 `park` 리모트 이름을 `jinwoo`로 변경하고 싶다면 아래와 같이 쓰면된다.

```bash
git remote rename park jinwoo
```
