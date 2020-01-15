# **Unit Test (General)**

## 以下我們會以Tennis Kata為例，進行TDD開發

> ## What is Tennis Kata ?
> A kata is an exercise in karate where you repeat a form many, many times, making little improvements in each.

## 網球計分規則 (單局)
  1. 一局比賽只有一名運動員發球，率先贏得至少4分並多出對手至少2分的運動員贏得一局。
  2. 從0至3分分別為「零」（love）、「十五」（fifteen）、「三十」（thirty）和「四十」（forty）。
  3. 當雙方運動員都得到了3分時，一般叫「平局」（deuce）而非「40比40」。
  4. 在出現平局後一名球手再得一分，被稱為「佔先」（advantage），而不再記分。如果在佔先的情況下失去一分，就再度回到平局；如佔先後再得一分，就贏得一局。

> #### 目標：寫出能知道比數的程式，即得分轉成文字。
> 下方為測試案例的假資料，當作參考。
```php=
    /**
     * @return mixed[][]
     */
    public function data()
    {
        return [
            '0-0' => [0, 0, "Love-All"],
            '1-1' => [1, 1, "Fifteen-All"],
            '2-2' => [2, 2, "Thirty-All"],
            '3-3' => [3, 3, "Deuce"],
            '4-4' => [4, 4, "Deuce"],
            '1-0' => [1, 0, "Fifteen-Love"],
            '0-1' => [0, 1, "Love-Fifteen"],
            '2-0' => [2, 0, "Thirty-Love"],
            '0-2' => [0, 2, "Love-Thirty"],
            '3-0' => [3, 0, "Forty-Love"],
            '0-3' => [0, 3, "Love-Forty"],
            '4-0' => [4, 0, "Win for player1"],
            '0-4' => [0, 4, "Win for player2"],
            '2-1' => [2, 1, "Thirty-Fifteen"],
            '1-2' => [1, 2, "Fifteen-Thirty"],
            '3-1' => [3, 1, "Forty-Fifteen"],
            '1-3' => [1, 3, "Fifteen-Forty"],
            '4-1' => [4, 1, "Win for player1"],
            '1-4' => [1, 4, "Win for player2"],
            '3-2' => [3, 2, "Forty-Thirty"],
            '2-3' => [2, 3, "Thirty-Forty"],
            '4-2' => [4, 2, "Win for player1"],
            '2-4' => [2, 4, "Win for player2"],
            '4-3' => [4, 3, "Advantage player1"],
            '3-4' => [3, 4, "Advantage player2"],
            '5-4' => [5, 4, "Advantage player1"],
            '4-5' => [4, 5, "Advantage player2"],
            '15-14' => [15, 14, "Advantage player1"],
            '14-15' => [14, 15, "Advantage player2"],
            '6-4' => [6, 4, "Win for player1"],
            '4-6' => [4, 6, "Win for player2"],
            '16-14' => [16, 14, "Win for player1"],
            '14-16' => [14, 16, "Win for player2"],
        ];
    }
```
讓我們寫下第一個測試
```php=
//TennisGame001Test

<?php

use PHPUnit\Framework\TestCase;

/**
 * TennisGame001 test case.
 */
class TennisGame001Test extends TestCase
{
    protected function setUp()
    {
        parent::setUp();
        $this->game = new TennisGame001('player1', 'player2');
    }

    protected function tearDown()
    {
        parent::tearDown();
        $this->game = null;
    }

    protected function testGetPlayer1Score()
    {
        //Arrange
        $expected = 0;

        //Act
        $actual = $this->game->getPlayer1Score();

        //Assert
        $this->assertEquals($expected, $actual);

    }
}
```

> ## 補充：單元測試撰寫的3A原則
> * Arrange : 初始化目標物件、相依物件、方法參數、預期結果，或是預期與相依物件的互動方式。
> * Act : 呼叫目標物件的方法。
> * Assert : 驗證是否符合預期。

接著跑測試
```shell=
    ./vendor/bin/phpunit
```

預期要失敗 (TDD中第一個紅燈)
```shell=
PHPUnit 6.5.14 by Sebastian Bergmann and contributors.

Runtime:       PHP 7.1.24 with Xdebug 2.6.1
Configuration: C:\tennis-kata\phpunit.xml.dist

E                                                                   1 / 1 (100%)

Time: 87 ms, Memory: 4.00MB

There was 1 error:

1) TennisGame001Test::testGetPlayer1Score
Error: Call to undefined method TennisGame001::getPlayer1Score()

C:\tennis-kata\tests\TennisGame001Test.php:25

ERRORS!
Tests: 1, Assertions: 0, Errors: 1.
```

一切符合預期，因為我們還沒實作

```php=
//TennisGame001

<?php

class TennisGame001
{
    private $player1_score = 0;
    private $player2_score = 0;
    private $player1_name = '';
    private $player2_name = '';

    public function __construct($player1_name, $player2_name)
    {
        $this->player1_name = $player1_name;
        $this->player2_name = $player2_name;
    }

    public function getGameScore()
    {
        //
    }

    public function getPlayer1Score()
    {
        return $this->player1_score;
    }
}
```

讓我們再跑一次測試

```shell=
PHPUnit 6.5.14 by Sebastian Bergmann and contributors.

Runtime:       PHP 7.1.24 with Xdebug 2.6.1
Configuration: C:\tennis-kata\phpunit.xml.dist

.                                                                   1 / 1 (100%)

Time: 75 ms, Memory: 4.00MB

OK (1 test, 1 assertion)
```

> ## 補充：單元測試的5種命名
> 1. 待測函數名稱：testGetPlayer1Score()
> 2. 待測函數名稱加上測試狀態與預期行為：getPlayer1Score_0Score_0Score()
> 3. 待測功能（feature）作為測試案例名稱：player1Score_GameStart_0Score
> 4. 預期行為加上測試狀態：Should_Get0Score_When_GameStart
> 5. GWT格式：Given_TennisGame_When_GameStart_Then_Player1Score0

個人目前偏好第二種，但若不以test開頭，需加上@test。

```php=
//TennisGame001Test

/**
 * @test
 */
public function getPlayer1Score_0Score_0Score()
{
    //Arrange
    $expected = 0;

    //Act
    $actual = $this->game->getPlayer1Score();

    //Assert
    $this->assertEquals($expected, $actual);
}
```

讓我們繼續寫測試
```php=
//TennisGame001Test

/**
 * @test
 */
public function addOnePointPlayer1Score_0Score_1Score()
{
    //Arrange
    $expected = 1;

    //Act
    $this->game->addOnePointPlayer1Score();
    $actual = $this->game->getPlayer1Score();

    //Assert
    $this->assertEquals($expected, $actual);

}
```

> 注意：因為剛剛有通過 getPlayer1Score() 的測試，所以可以放心使用。

讓測試先失敗

```shell=
PHPUnit 6.5.14 by Sebastian Bergmann and contributors.

Runtime:       PHP 7.1.24 with Xdebug 2.6.1
Configuration: C:\tennis-kata\phpunit.xml.dist

.E                                                                  2 / 2 (100%)

Time: 76 ms, Memory: 4.00MB

There was 1 error:

1) TennisGame001Test::addOnePointPlayer1Score_0Score_1Score
Error: Call to undefined method TennisGame001::addOnePointPlayer1Score()

C:\tennis-kata\tests\TennisGame001Test.php:43

ERRORS!
Tests: 2, Assertions: 1, Errors: 1.
```

實作方法

```php=
//TennisGame001

public function addOnePointPlayer1Score()
{
    $this->player1_score++;
}
```

通過測試
```shell=
PHPUnit 6.5.14 by Sebastian Bergmann and contributors.

Runtime:       PHP 7.1.24 with Xdebug 2.6.1
Configuration: C:\tennis-kata\phpunit.xml.dist

..                                                                  2 / 2 (100%)

Time: 79 ms, Memory: 4.00MB

OK (2 tests, 2 assertions)
```

> ## * Test Driven Development
> * 紅燈 -> 綠燈 -> 重構
> * **紅燈**：寫一個測試並且執行讓他 test fail
> * **綠燈**：使程式碼可以執行，並且 test pass
> * **重構**：重構程式碼，並且保持 test pass

透過這樣的過程，讓我們逐步完成程式碼。