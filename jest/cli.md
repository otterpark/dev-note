# Jest CLI

Jest에서 자주 사용하는 CLI를 정리해 보려고 한다.

들어가기 전,

`Jest`는 `Camelcase`와 `dashed args` 형식을 모두 지원합니다. 다음 예제는 동일한 결과를 가져옵니다.

```bash
jest --collect-coverage
jest --collectCoverage
```

## jest (filepath)

단일 파일 테스트를 진행하고 싶을 때 파일 경로를 지정하여 사용할 수 있다.

```bash
jest my-test
jest path/to/my-test.js
```

## Jest --watch

`watch 모드`에서도 마찬가지로 특정 테스트 세트에 초점을 맞추기 위해 파일 이름이나 경로를 지정할 수도 있습니다.

```bash
jest --watch # 기본적으로 jest -o를 실행
jest --watchAll # 모든 테스트 실행
```

## jest --findRelatedTests

특정 파일과 관련된 테스트를 실행합니다.

```bash
jest --findRelatedTests path/to/fileA.js path/to/fileB.js
```

## jest --coverage

현 프로젝트에 `jest`로 테스트를 진행하면서 커버리지 데이터를 수집할 수 있습니다. 명령어 실행시 `Statements(구문)`, `Branches(분기)`, `Functions(함수)`, `Lines(줄)`을 기준으로 얼마나 테스트되고 있는지 퍼센트로 보입니다.

```bash
=====Coverage summary======
Statements   : 95.74% ( 494/516 )
Branches     : 78.25% ( 403/515 )
Functions    : 93.44% ( 114/122 )
Lines        : 95.67% ( 486/508 )
===========================

Test Suites: 29 passed, 29 total
Tests:       217 passed, 217 total
Snapshots:   1 passed, 1 total
Time:        26.702 s
```

### 커버리지 최소 기준 설정

전체 프로젝트에서 `jest.config.js`에 `coverageThreshold`를 사용하여 커버리지의 최소 기준 설정을 할 수 있다.

#### jest.config.js

```js
module.exports = {
  coverageThreshold: {
    './src/': {
      statements: 95,
      branches: 90,
      functions: 95,
      lines: 90,
    },
  },
};
```

### package.json에 스크립트 지정

편하게 나만의 스크립트를 지정하는 `package.json`에 스크립트를 등록해 놓으면 편하게 사용할 수 있다.

#### package.json

```json
{
  "scripts": {
    "test": "jest",
    "coverage": "jest --coverage"
  }
}
```