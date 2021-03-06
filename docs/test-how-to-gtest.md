# Google Test の使い方

## Google Test のインストール

msys2 のターミナルを起動して下記コマンドを打ってインストールします。
[Make] と [CMake] が必要なのでインストールしていない場合は先にインストールしてください。

[Make]: make-make.md
[CMake]: make-cmake.md

```bash
$ cd /tmp
$ wget 'https://github.com/google/googletest/archive/release-1.8.1.tar.gz'
$ tar zxvf release-1.8.1.tar.gz
$ mkdir -p /usr/local/src
$ mv googletest-release-1.8.1 /usr/local/src
$ cd /usr/local/src/googletest-release-1.8.1
$ mkdir build
$ cd build
$ cmake ..
$ make
$ make install
```

## 実行例

例として、 `sample.h (.cc)` に偶数を判定する関数 `IsEven` を作成し、この関数の動作をテストするには次のようにします。

=== "sample.h"

    ```cpp linenums="1"
    #ifndef SAMPLE_H_
    #define SAMPLE_H_

    /**
    * 入力値が偶数か判定する関数
    */
    bool IsEven(int x);

    #endif  // SAMPLE_H_
    ```

=== "sample.cc"

    ```cpp linenums="1"
    #include "sample.h"

    bool IsEven(int x) {
        return x % 2 == 0;
    }
    ```

=== "sample_test.cc"

    ```cpp  hl_lines="1 3" linenums="1"
    #include <gtest/gtest.h>

    #include "sample.h"

    TEST(IsEvenTest, Negative) {
        EXPECT_FALSE(IsEven(-1));
        EXPECT_TRUE(IsEven(-2));
    }

    TEST(IsEvenTest, Zero) {
        EXPECT_TRUE(IsEven(0));
    }

    TEST(IsEvenTest, Positive) {
        EXPECT_FALSE(IsEven(1));
        EXPECT_TRUE(IsEven(2));
    }
    ```

`sample_test.cc` にテストコードを記述しています。
テストコードでは、Google Test を利用するために `gtest/gtest.h` をインクルードし、テスト対象となる `sample.h` もインクルードしています。

次のコマンドでテスト実行ファイルを作成します。ここではコマンドの詳細については割愛します。

```bash
# ビルド
$ g++ -std=c++11 sample.cc sample_test.cc -o test -L/usr/local/lib -lgtest -lgtest_main
# 実行
$ ./test.exe
```

```bash
# 実行結果
Running main() from /usr/local/src/googletest-release-1.8.1/googletest/src/gtest_main.cc
[==========] Running 3 tests from 1 test case.
[----------] Global test environment set-up.
[----------] 3 tests from IsEvenTest
[ RUN      ] IsEvenTest.Negative
[       OK ] IsEvenTest.Negative (0 ms)
[ RUN      ] IsEvenTest.Zero
[       OK ] IsEvenTest.Zero (0 ms)
[ RUN      ] IsEvenTest.Positive
[       OK ] IsEvenTest.Positive (0 ms)
[----------] 3 tests from IsEvenTest (0 ms total)

[----------] Global test environment tear-down
[==========] 3 tests from 1 test case ran. (0 ms total)
[  PASSED  ] 3 tests.
```

成功したテストは `[ OK ]` と出力され、失敗したテストは `[ FAILED ]` と出力されます。
また、成功して通過したテスト数が `[ PASSED ]` に表示されます。

## 初歩的なテストコードの書き方

### テスト関数

Google Test に用意されている `TEST()` を利用してテスト関数を作成します。
`TEST()` の第1引数にテストケース名、第2引数にテスト名を記述します。

```cpp
TEST(/* テストケース名(大項目)*/, /* テスト名(小項目) */) {
  // テスト関数内は、通常通り C++ のコードを記述可能
}
```

テストケース名とテスト名には `_` を含んではいけません。

### アサーション

Google Test に用意されているアサーションを利用することで、
テスト対象コードの動作を検証することが出来ます。

```cpp
// true/falseのアサーション
EXPECT_TRUE(condition);  // condition が true か
EXPECT_FALSE(condition);  // condition が false か

// 2つの値を比較するアサーション
EXPECT_EQ(expected, actual);  // expected == actual か
EXPECT_NE(expected, actual);  // expected != actual か
EXPECT_LT(expected, actual);  // expected < actual か
EXPECT_LE(expected, actual);  // expected <= actual か
EXPECT_GT(expected, actual);  // expected > actual か
EXPECT_GE(expected, actual);  // expected >= actual か
```

これらのアサーションを利用する場合は、
期待結果 (expected)、 テスト対象 (actual)の順で記述します。

`EXPECT_` で始まるアサーションの他に、 `ASSERT_` で始まるアサーションがあります。
`EXPECT_` の場合は、テストに失敗してもテスト関数がそのまま続行されますが、
`ASSERT_` の場合は、テストに失敗するとその時点でテストを中断してテスト関数を抜けます。

試しに、誤った実装がなされた関数 `IsEven` を利用して、テスト失敗時の出力を確認すると次のようになります。

=== "sample.h"

    ```cpp linenums="1"
    #ifndef SAMPLE_H_
    #define SAMPLE_H_

    /**
    * 入力値が偶数か判定する関数
    */
    bool IsEven(int x);

    #endif  // SAMPLE_H_
    ```

=== "sample.cc"

    ```cpp hl_lines="4" linenums="1"
    #include "sample.h"

    bool IsEven(int x) {
        return x % 2 == 1; // 誤り。 x が奇数のときに true になってしまう…
    }
    ```

=== "sample_test.cpp"

    ```cpp hl_lines="8 14" linenums="1"
    #include <iostream>

    #include <gtest/gtest.h>

    #include "sample.h"

    TEST(IsEvenTest, AssertPositive) {
        ASSERT_FALSE(IsEven(1));  // ASSERTテストは失敗すると中断
        std::cout << "中断により、この文字列は出力されない" << std::endl;
        ASSERT_TRUE(IsEven(2));
    }

    TEST(IsEvenTest, ExpectPositive) {
        EXPECT_FALSE(IsEven(1));  // EXPECTテストは失敗しても続行
        std::cout << "続行のため、この文字列は出力される" << std::endl;
        EXPECT_TRUE(IsEven(2));
    }
    ```

=== "実行結果"

    ```bash hl_lines="6 7 8 9 10 12 13 14 15 16 17 18 19 20 21" linenums="1"
    Running main() from /usr/local/src/googletest-release-1.8.1/googletest/src/gtest_main.cc
    [==========] Running 2 tests from 1 test case.
    [----------] Global test environment set-up.
    [----------] 2 tests from IsEvenTest
    [ RUN      ] IsEvenTest.AssertPositive
    sample_test.cc:8: Failure
    Value of: IsEven(1)
    Actual: true
    Expected: false
    [  FAILED  ] IsEvenTest.AssertPositive (1 ms)
    [ RUN      ] IsEvenTest.ExpectPositive
    sample_test.cc:14: Failure
    Value of: IsEven(1)
    Actual: true
    Expected: false
    続行のため、この文字列は出力される
    sample_test.cc:16: Failure
    Value of: IsEven(2)
    Actual: false
    Expected: true
    [  FAILED  ] IsEvenTest.ExpectPositive (0 ms)
    [----------] 2 tests from IsEvenTest (1 ms total)

    [----------] Global test environment tear-down
    [==========] 2 tests from 1 test case ran. (1 ms total)
    [  PASSED  ] 0 tests.
    [  FAILED  ] 2 tests, listed below:
    [  FAILED  ] IsEvenTest.AssertPositive
    [  FAILED  ] IsEvenTest.ExpectPositive

    2 FAILED TESTS
    ```

テストが失敗した場合、失敗箇所と失敗理由が出力されます。

## 参考

- [Google Test ドキュメント日本語訳][gtest-docs-jp]
- [Google Mock ドキュメント日本語訳][gmock-docs-jp]
- [サンプル集 — google/googletest (GitHub)][gtest-samples]

[gtest-docs-jp]: http://opencv.jp/googletestdocs/index.html
[gmock-docs-jp]: http://opencv.jp/googlemockdocs/index.html
[gtest-samples]: https://github.com/google/googletest/tree/master/googletest/samples
