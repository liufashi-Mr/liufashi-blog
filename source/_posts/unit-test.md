---
title: unit-test
date: 2023-07-26 16:38:16
tags:
---

为什么需要写测试

1. 保证代码鲁棒性
2. 验证重构是否造成破坏
3. 测试用例是另一种代码使用用例
一些测试案例
<https://github.com/postmanlabs/postman-runtime/blob/develop/test/integration/sanity/assertions.test.js>
node-jsonc-parser/json.test.ts at main · microsoft/node-jsonc-parser
protobuf.js/api_root.js at master · protobufjs/protobuf.js
怎么写测试
哪些代码需要写测试
代码示例 1
function add(a, b) {
  return a + b;
}

console.log(add(1, 2)); // 输出：3
代码示例 2
function findLargestPalindromeProduct(digits) {
  function isPalindrome(number) {
    const str = number.toString();
    return str === str.split('').reverse().join('');
  }

  let max = 0;
  const upperLimit = Math.pow(10, digits) - 1;
  const lowerLimit = Math.pow(10, digits - 1);

  for (let i = upperLimit; i >= lowerLimit; i--) {
    for (let j = i; j >= lowerLimit; j--) {
      const product = i * j;
      if (product <= max) {
        break;
      }
      if (isPalindrome(product)) {
        max = product;
      }
    }
  }

  return max;
}

console.log(findLargestPalindromeProduct(3)); // 输出：906609
单元测试与集成测试
两者之间的主要差异在于粒度和测试范围。
“单元测试”是最小粒度的测试，可以测试私有成员。
“集成测试”测试对外暴露的接口，不测试私有成员。
代码示例 1
下面一段代码如何改造成易于测试的代码。
export default function login(method: 'username-passwd' | 'email-passwd' | 'other', data: any) {
        if (method === 'username-passwd') {
                // ...
        } else if (method === 'email-passwd') {
                // ...
        } else {
                // ...
        }
}
代码示例 2
// 单元测试
export function usernamePasswdLogin(data: any) {
        // ...
}

// 单元测试
export function emailPasswdLogin(data: any) {
        // ...
}

// 单元测试
export function otherLogin(data: any) {
        // ...
}

// 集成测试
export default function login(method: 'username-passwd' | 'email-passwd' | 'other', data: any) {
        if (method === 'username-passwd') {
                return usernamePasswdLogin(data);
        } else if (method === 'email-passwd') {
                return emailPasswdLogin(data);
        } else {
                return otherLogin(data);
        }
}
代码示例 3
假如这个接口是在类中，还有另外一种做法，让 login 方法也可以做单元测试。
class Auth {
        protected usernamePasswdLogin(data: any) {
                // ...
        }

        protected emailPasswdLogin(data: any) {
                // ...
        }
        
        protected otherLogin(data: any) {
                // ...
        }

        public login(method: 'username-passwd' | 'email-passwd' | 'other', data: any) {
                if (method === 'username-passwd') {
                        return this.usernamePasswdLogin(data);
                } else if (method === 'email-passwd') {
                        return this.emailPasswdLogin(data);
                } else {
                        return this.otherLogin(data);
                }
        }
}
class AuthForTest extends Auth {
        usernamePasswdLogin(data: any) {
                return 'usernamePasswdLogin'
        }

        emailPasswdLogin(data: any) {
                return 'emailPasswdLogin'
        }
        
        otherLogin(data: any) {
                return 'otherLogin'
        }
}
TDD 与 BDD
TDD（测试驱动开发，Test-Driven Development）是一种软件开发方法，它要求在编写实际功能代码之前先编写测试用例。这种方法的核心思想是通过编写测试来驱动实际功能的开发。
BDD（行为驱动开发，Behavior-Driven Development）是一种类似于 TDD 的软件开发方法，但它更关注于软件的行为和功能，而不仅仅是测试。
TDD 不太容易实现，特别是没有代码之前，连测试用例也不知道是否正确。所以一种比较有效的开发方式是：
开发时，使用 BDD；维护时，使用 TDD。
<https://apifox.coding.net/p/apifox/d/postman-runtime/git/commit/68c93fb027c87ac963509e79b53796a8e80d5060>
it 与 test
"it" 和 "test" 都是用于编写单元测试的函数，不同的测试框架和库可能有不同的用法和约定。一般来说，它们是两种不同的测试风格：

1. "it" 风格：这种风格通常用于 BDD（Behavior-Driven Development，行为驱动开发）中，它侧重于描述被测试对象的行为和功能。"it" 函数通常被用于编写针对一个具体行为的测试用例。例如，一个测试用例可能会这样描述："it should return true when the input is valid"。
2. "test" 风格：这种风格通常用于传统的单元测试中，它侧重于测试函数或模块的功能和正确性。"test" 函数通常被用于编写针对一个具体函数或模块的测试用例。例如，一个测试用例可能会这样描述："test add function should return correct sum when given two numbers"。
测试方法
常规测试
Vitest
1. 准备环境 - setup、before、beforeEach
2. 执行测试代码
3. 断言 - assert、expect
4. 清理环境 - tearDown、after、afterEach
快照测试

- 案例一（TypeScript）
  - CODING | 一站式软件研发管理平台
// cspell:ignore protos, apifox

import path from 'path';
import { describe, expect, it } from 'vitest';
import { parse } from '../src/index';

describe('Parse', () => {
  it('should parse proto file (hello-world.proto)', async () => {
    const result = await parse(path.resolve(__dirname, '../example/protos/hello-world.proto'), {
      includes: [],
      appName: 'apifox',
    });
    expect(JSON.stringify(result, null, 4)).w();
  });

  it('should parse proto file (multiple-types.proto)', async () => {
    const result = await parse(path.resolve(__dirname, '../example/protos/multiple-types.proto'), {
      includes: [],
      appName: 'apifox',
    });
    expect(JSON.stringify(result, null, 4)).toMatchSnapshot();
  });

  it('should parse proto file (empty.proto)', async () => {
    const result = await parse(path.resolve(__dirname, '../example/protos/empty.proto'), {
      includes: [],
      appName: 'apifox',
    });
    expect(JSON.stringify(result, null, 4)).toMatchSnapshot();
  });

  it('should parse proto file (root.proto)', async () => {
    const result = await parse(path.resolve(__dirname, '../example/protos/root.proto'), {
      includes: [path.resolve(__dirname, '../example/protos')],
      appName: 'apifox',
    });
    expect(JSON.stringify(result, null, 4)).toMatchSnapshot();
  });

  it('should parse proto file (pet.proto)', async () => {
    const result = await parse(path.resolve(__dirname, '../example/protos/pet.proto'), {
      includes: [path.resolve(__dirname, '../example/protos')],
      appName: 'apifox',
    });
    expect(JSON.stringify(result, null, 4)).toMatchSnapshot();
  });

  it('should parse json format proto file (root.proto.json)', async () => {
    const result = await parse(path.resolve(__dirname, '../example/protos/root.proto.json'), {
      includes: [],
      appName: 'apifox',
    });
    expect(JSON.stringify(result, null, 4)).toMatchSnapshot();
  });

  it('should parse recursive relative file', async () => {
    const result = await parse(path.resolve(__dirname, '../example/protos/recursive.proto'), {
      includes: [path.resolve(__dirname, '../example/protos')],
      appName: 'apifox',
    });
    expect(JSON.stringify(result, null, 4)).toMatchSnapshot();
  });

  it('should throw error if file not found', async () => {
    expect(() =>
      parse(path.resolve(__dirname, '../example/protos/root-with-incorrect-dependencies.proto'), {
        includes: [path.resolve(__dirname, '../example')],
        appName: 'apifox',
      }),
    ).rejects.toThrowError('File (dependency/analysis.proto) not found.');
  });

  it('should throw error if proto file not found', async () => {
    expect(() =>
      parse(
        path.join(__dirname, '../example/protos/import-not-exist-file.proto'),
        { includes: [], appName: 'apifox' }
      )
    ).rejects.toThrowError('File (any-files-not-exist.proto) not found.');
  });

  it('should throw error if proto file not found (dependent file import an not exist file.)', async () => {
    expect(() =>
      parse(
        path.join(__dirname, '../example/protos/import-not-exist-file-2.proto'),
        { includes: [], appName: 'apifox' }
      )
    ).rejects.toThrowError('File (google/protobuf/descriptor.proto) not found.');
  });

  it('should find dependency file according to one of includeDirs item', async () => {
    const result = await parse(path.resolve(__dirname, '../example/protos/root.proto'), {
      includes: [
        path.resolve(__dirname, '../example'),
        path.resolve(__dirname, '../example/protos'),
        path.resolve(__dirname, '../'),
      ],
      appName: 'apifox',
    });
    expect(JSON.stringify(result, null, 4)).toMatchSnapshot();
  });

  it('should find dependency file according to the root file if includeDirs is empty array', async () => {
    const result = await parse(path.resolve(__dirname, '../example/protos/root.proto'), {
      includes: [],
      appName: 'apifox',
    });
    expect(JSON.stringify(result, null, 4)).toMatchSnapshot();
  });

  it('should find dependency file according to the root file if all dirs in includeDirs is incorrect', async () => {
    const result = await parse(path.resolve(__dirname, '../example/protos/root.proto'), {
      includes: ['./any-incorrect-dir-path/'],
      appName: 'apifox',
    });
    expect(JSON.stringify(result, null, 4)).toMatchSnapshot();
  });

  it('should parse multiple file with same package name', async () => {
    const result = await parse(path.join(__dirname, '../example/protos/new-root.proto'), {
      includes: [path.join(__dirname, '../example/protos')],
      appName: 'apifox',
    });
    expect(JSON.stringify(result, null, 4)).toMatchSnapshot();
  });

  it('should use root file name as module name if package name is not set', async () => {
    const result = await parse(
      path.join(__dirname, '../example/protos/proto-without-package-name.proto'),
      { includes: [], appName: 'apifox' },
    );
    expect(JSON.stringify(result, null, 4)).toMatchSnapshot();
  });
});

- 案例二（Python）
  - CODING | 一站式软件研发管理平台
import json
from state_machine.agg_summary_daily.state_machine import AggSummaryDailyStateMachine

def test_state_machine_with_snapshot(snapshot):
    snapshot.snapshot_dir = 'snapshots'
    snapshot.assert_match(
        json.dumps(AggSummaryDailyStateMachine().definition, indent=4, ensure_ascii=False),
        'state_machine_agg_summary_daily_definition.json',
    )
表格驱动测试

- 案例一（TypeScript）
  - CODING | 一站式软件研发管理平台
import trimPropertiesDeep from '../trimPropertiesDeep';
import { cloneDeep } from 'lodash';

const data: {input: any, properties: string[], output: any}[] = [{
  input: {a: {b: 3}},
  properties: ['a.b'],
  output: {a: {}},
}, {
  input: {a: {b: 3, c: 4}},
  properties: ['a.b'],
  output: {a: {c: 4}},
}, {
  input: {a: {b: 3, d: [1, 2]}},
  properties: ['a.d'],
  output: {a: {b: 3}},
}, {
  input: {a: {b: 3, d: [{e: 10, f: 20}, {e: 30, f: 40}]}},
  properties: ['a.d[]'],
  output: {a: {b: 3, d: []}},
}, {
  input: {a: {b: 3, d: [1, 2]}},
  properties: ['a.d[0]'],
  output: {a: {b: 3, d: [2]}},
}, {
  input: {a: {b: 3, d: [{e: 10, f: 20}, {e: 30, f: 40}]}},
  properties: ['a.d[].e'],
  output: {a: {b: 3, d: [{f: 20}, {f: 40}]}},
}, {
  input: {a: {b: 3, d: [{e: 10, f: 20}, {e: 30, f: 40}]}},
  properties: ['a.d[0].e'],
  output: {a: {b: 3, d: [{f: 20}, {e: 30, f: 40}]}},
}, {
  input: {a: {b: 3, d: [{e: 10, f: 20}, {e: 30, f: 40}]}},
  properties: ['a.d[0].e', 'a.d[1].f'],
  output: {a: {b: 3, d: [{f: 20}, {e: 30}]}},
}, {
  input: {a: {b: 3, d: [{e: 10, f: 20}, {e: 30, f: 40}]}},
  properties: ['a.d[]', 'a.d[0].e', 'a.d[1].f'],
  output: {a: {b: 3, d: []}},
}, {
  input: {a: {b: 3, d: [{e: 10, f: 20}, {e: 30, f: 40}]}},
  properties: ['a.d[0].e', 'a.d[1].f', 'a.d'],
  output: {a: {b: 3}},
}, {
  input: {a: {b: 3, d: [{e: 10, f: [{g: 50}, {i: [{k: 50, h: 60}]}]}, {e: 30, f: 40}]}},
  properties: ['a.d[0].f[1].i[].h'],
  output: {a: {b: 3, d: [{e: 10, f: [{g: 50}, {i: [{k: 50}]}]}, {e: 30, f: 40}]}},
}, {
  input: {a: {b: 3, d: [{e: 10, f: [{g: 50}, {i: [{k: 50, h: 60}]}]}, {e: 30, f: 40}]}},
  properties: ['a.d[0].f[1].unknown-property'],
  output: {a: {b: 3, d: [{e: 10, f: [{g: 50}, {i: [{k: 50, h: 60}]}]}, {e: 30, f: 40}]}},
}, {
  input: {a: {b: 3, d: [{e: 10, f: [{g: 50}, {i: [{k: 50, h: 60}]}]}, {e: 30, f: 40}]}},
  properties: ['a.d[0][]'],
  output: {a: {b: 3, d: [{e: 10, f: [{g: 50}, {i: [{k: 50, h: 60}]}]}, {e: 30, f: 40}]}},
}, {
  input: {a: {b: [[1, 2], [3, 4]]}},
  properties: ['a.b[][1]'],
  output: {a: {b: [[1], [3]]}},
}, {
  input: {a: {b: [[1, 2], [3, 4]]}},
  properties: ['a.b[0][1]'],
  output: {a: {b: [[1], [3, 4]]}},
}, {
  input: {a: {b: [[1, 2], [3, 4]]}},
  properties: ['a.b[][]'],
  output: {a: {b: [[], []]}},
}, {
  input: [{id: 1, a: 1}, {id: 2, b: 2, c: 3}],
  properties: ['[].id'],
  output: [{a: 1}, {b: 2, c: 3}],
}]

describe('trimProperty', () => {
  data.forEach((item, i) => {
    it(`trimProperty-${i}`, () => {
      const res = trimPropertiesDeep(cloneDeep(item.input), item.properties);
      expect(res).toEqual(item.output);
    });
  })
});

- 案例二（Python）
  - CODING | 一站式软件研发管理平台
from typing_extensions import NotRequired
import pytest
from typing import List, Tuple, TypedDict
from lambda_function import lambda_handler

class Input(TypedDict):
    date: NotRequired[str]
    biz_date: NotRequired[str]
    period: str
    days_offset_in_period: NotRequired[int]

test_cases_with_unknown_period: List[Tuple[Input, bool]] = [(
    {
        "date": "2022-12-01",
        "period": "",
    },
    False
), (
    {
        "date": "2022-12-02",
        "period": "foo",
    },
    False
), (
    {
        "date": "2022-12-03",
        "period": "bar",
    },
    False
)]

test_cases_of_day: List[Tuple[Input, bool]] = [(
    {
        "date": "2022-12-01",
        "period": "day",
    },
    True
), (
    {
        "date": "2022-12-02",
        "period": "day",
    },
    True
), (
    {
        "date": "2022-12-03",
        "period": "day",
    },
    True
), (
    {
        "date": "2022-12-04",
        "period": "day",
    },
    True
)]

test_cases_of_week: List[Tuple[Input, bool]] = [(
    {
        "date": "2022-12-01",
        "period": "week",
    },
    False
), (
    {
        "date": "2022-12-02",
        "period": "week",
    },
    False
), (
    {
        "date": "2022-12-03",
        "period": "week",
    },
    False
), (
    {
        "date": "2022-12-04",
        "period": "week",
    },
    False
), (
    {
        "date": "2022-12-05",
        "period": "week",
    },
    True
), (
    {
        "date": "2022-12-06",
        "period": "week",
    },
    False
)]

test_cases_of_week_biz_date: List[Tuple[Input, bool]] = [(
    {
        "biz_date": "2022-12-01",
        "period": "week",
    },
    False
), (
    {
        "biz_date": "2022-12-02",
        "period": "week",
    },
    False
), (
    {
        "biz_date": "2022-12-03",
        "period": "week",
    },
    False
), (
    {
        "biz_date": "2022-12-04",
        "period": "week",
    },
    True
), (
    {
        "biz_date": "2022-12-05",
        "period": "week",
    },
    False
), (
    {
        "biz_date": "2022-12-06",
        "period": "week",
    },
    False
)]

test_cases_of_month: List[Tuple[Input, bool]] = [(
    {
        "date": "2022-11-30",
        "period": "month",
    },
    False
), (
    {
        "date": "2022-12-01",
        "period": "month",
    },
    True
), (
    {
        "date": "2022-12-02",
        "period": "month",
    },
    False
), (
    {
        "date": "2023-01-01",
        "period": "month",
    },
    True
), (
    {
        "date": "2023-01-02",
        "period": "month",
    },
    False
)]

test_cases_of_month_biz_date: List[Tuple[Input, bool]] = [(
    {
        "biz_date": "2022-11-30",
        "period": "month",
    },
    True
), (
    {
        "biz_date": "2022-12-01",
        "period": "month",
    },
    False
), (
    {
        "biz_date": "2022-12-02",
        "period": "month",
    },
    False
), (
    {
        "biz_date": "2022-12-31",
        "period": "month",
    },
    True
), (
    {
        "biz_date": "2023-01-01",
        "period": "month",
    },
    False
), (
    {
        "biz_date": "2023-01-02",
        "period": "month",
    },
    False
)]

test_cases_of_week_with_offset: List[Tuple[Input, bool]] = [(
    {
        "date": "2022-12-01",
        "period": "week",
        "days_offset_in_period": 1
    },
    False
), (
    {
        "date": "2022-12-02",
        "period": "week",
        "days_offset_in_period": 1
    },
    False
), (
    {
        "date": "2022-12-03",
        "period": "week",
        "days_offset_in_period": 1
    },
    False
), (
    {
        "date": "2022-12-04",
        "period": "week",
        "days_offset_in_period": 1
    },
    False
), (
    {
        "date": "2022-12-05",
        "period": "week",
        "days_offset_in_period": 1
    },
    False
), (
    {
        "date": "2022-12-06",
        "period": "week",
        "days_offset_in_period": 1
    },
    True
)]

test_cases_of_month_with_offset: List[Tuple[Input, bool]] = [(
    {
        "date": "2022-11-30",
        "period": "month",
        "days_offset_in_period": 29
    },
    True
), (
    {
        "date": "2022-12-01",
        "period": "month",
        "days_offset_in_period": 29
    },
    False
), (
    {
        "date": "2022-12-02",
        "period": "month",
        "days_offset_in_period": 1
    },
    True
), (
    {
        "date": "2023-01-01",
        "period": "month",
        "days_offset_in_period": 1
    },
    False
)]

@pytest.mark.parametrize("input,expected", [
    *test_cases_with_unknown_period,
    *test_cases_of_day,
    *test_cases_of_week,
    *test_cases_of_week_biz_date,
    *test_cases_of_month,
    *test_cases_of_month_biz_date,
    *test_cases_of_week_with_offset,
    *test_cases_of_month_with_offset,
])
def test_can_execute_now(input: Input, expected: bool):
    assert lambda_handler(input, {}) == expected
文档测试

- 案例一（antd）
  - ant-design/badge.md at 3.x-stable · ant-design/ant-design
- 案例二（Rust）
/// 第一行是对函数的简短描述。
///
/// 接下来数行是详细文档。代码块用三个反引号开启，Rust 会隐式地在其中添加
/// `fn main()` 和 `extern crate <cratename>`。比如测试 `doccomments` crate：
///
/// ```
/// let result = doccomments::add(2, 3);
/// assert_eq!(result, 5);
/// ```
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}

/// 文档注释通常可能带有 "Examples"、"Panics" 和 "Failures" 这些部分。
///
/// 下面的函数将两数相除。
///
/// # Examples
///
/// ```
/// let result = doccomments::div(10, 2);
/// assert_eq!(result, 5);
/// ```
///
/// # Panics
///
/// 如果第二个参数是 0，函数将会 panic。
///
/// ```rust,should_panic
/// // panics on division by zero
/// doccomments::div(10, 0);
///```
pub fn div(a: i32, b: i32) -> i32 {
    if b == 0 {
        panic!("Divide-by-zero error");
    }

    a / b
}
要点
代码可测试性 - 副作用

1. 依赖全局变量
2. 依赖系统文件
3. <https://apifox.coding.net/p/apifox/d/protobuf-parser/git/commit/1ecb1ff3d2012b3cc3ec9484c24408ff92fe4867>
4. 依赖不稳定输出的函数 - Date.now()、Math.random()

- 案例
  - 修改前：
const fs = require('fs');

// 副作用：写文件、输出日志
function processData(input) {
  const processedData = input.toUpperCase();
  const fileName = 'output.txt';
  fs.writeFileSync(fileName, data);
  console.log(`Data saved to ${fileName}`);
}

processData('Hello, world!');

- 修改后：
- 将副作用（写入文件和输出日志）从 processData 函数中分离出来，并将它们作为参数传递给 saveDataToFile 函数。这使得我们可以使用依赖注入来测试 saveDataToFile 函数，而无需实际与文件系统和控制台交互。
// 副作用由外部注入
function processData(input, saveData, log) {
        const fileName = 'output.txt';
  const processedData = input.toUpperCase();
  saveData(processedData, fileName, fs.writeFileSync);
        log(`Data saved to ${fileName}`);
}

// 使用依赖注入进行测试
function testableSaveDataToFile(data, fileName, writeFile) {
  let fileWritten = false;
  let logMessage = '';

  writeFile = (name, content) => {
    fileWritten = name === fileName && content === data;
  };

  log = (message) => {
    logMessage = message;
  };

  saveDataToFile(data, fileName, writeFile, log);

  return {
    fileWritten: fileWritten,
    logMessage: logMessage,
  };
}

const testData = 'Hello, world!';
const testFileName = 'test_output.txt';
const testResult = testableSaveDataToFile(testData, testFileName, null, null);

console.log(`Test result: ${testResult.fileWritten && testResult.logMessage.includes(testFileName) ? 'PASSED' : 'FAILED'}`);

- 措施
  1. 高内聚低耦合
  2. 依赖注入
  - 面向接口 / 依赖抽象
  - <https://apifox.coding.net/p/apifox/d/tarslib/git/tree/feature%2Fdubbo/packages/app-renderer/src/utils/edas-loader/api.ts>
type Fetcher<T, O> = (options: O) => Promise<PaginationList<T>>;
export async function fetchAllData<T, O extends PaginationOptions & RequesterOptions>(
  fn: Fetcher<T, O>,
  options: O,
) {
  const result: T[] = [];
  let pageNumber = 1;
  let totalPages = 1;
  while (pageNumber <= totalPages) {
    const { Content, TotalPages } = await fn({ ...options, page: pageNumber });
    if (!Array.isArray(Content) || Content.length === 0) {
      break;
    }
    result.push(...Content);
    totalPages = TotalPages;
    pageNumber++;
    await sleep(options.requester?.delay);
  }
  return result;
}
测试的性能

1. sleep / setTimeout
2. 全局变量依赖，可能导致无法并行
测试的稳定性

- number 的精度
- Vitest
- JSON.stringify 与 DeepEqual
- CODING | 一站式软件研发管理平台
it('should trigger all handlers mounted on "onResult"', async () => {
  const { client, handleResult } = ctx;
  const handlers = [vi.fn(), vi.fn(), vi.fn()];
  handlers.forEach((fn) => { client.onResult(fn); });
  await client.request();
  const expectedResult = {
      code: 0,
      status: 'OK',
      body: expect.jsonContaining({
          message: 'OK',
                                        data: '',
      }),
      responseSize: 25,
      responseTime: 1
  };
  expect.hasAssertions();
  [...handlers, handleResult].forEach((fn) => {
      expect(fn).toHaveBeenCalledOnce();
      expect(fn).toHaveBeenCalledWith(expect.objectContaining(expectedResult));
  });
});
- 关注核心测试点
- Vitest
- 定时与时间判断
- 不能通过定时拿到精确的时间，放弃吧。
测试的质量
- if-else 可能导致分支非预期地跳过断言
- 回调中的断言可能没有被执行
- Vitest
- 突变测试 - 测试测试的测试
  - 测试也是需要被测试的，测试实现成功了，不代表测试逻辑是对的。
  - CODING | 一站式软件研发管理平台
测试的陷阱

1. 只追求覆盖率
2. 不注重测试代码的质量
3. 只用测试保证代码正确性
4. Mock 使用不当

- 使用官方提供的 mock 对象。
- Testing - Rocket Programming Guide

5. 面向接口 / 依赖抽象
