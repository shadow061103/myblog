---
title: "Test-Driven Development(TDD)"
date: 2022-10-24T15:51:14+08:00
draft: false
tags: ["單元測試"]
type: post
author: "Steve Lin"
description: ""
---
> 測試驅動開發
### 需求
- 透過測試（單元測試）推動整個開發的進行
- 在撰寫程式碼之前，先透過測試案例的撰寫，確定外部如何使用物件，包含了功能、類別、方法的確定。
- 以需求為出發點，開發符合需求的程式， 避免過度設計（Over Design）
### 開發流程
1.定義目前要完成的功能（確定需求）
2.撰寫測試案例。撰寫的過程應針對意圖進行命名，思考需要得到什麼結果，要得到這樣的結果會需要提供什麼資訊。
3.編譯測試程式不通過。
4.撰寫對應測試程式的功能程式碼。
5.讓測試通過。
6.對程式碼進行重構，並確保測試可以通過。
7.依照同樣的循環，完成所有的功能開發。
### 三原則
1.You are not allowed to write any production code unless it is to make a failing unit test pass.
(除非這能讓失敗的單元測試通過， 否則不允許去編寫任何的產品代碼。)
2.You are not allowed to write any more of a unit test than is sufficient to fail; and compilation failures are failures.
(只允許編寫剛好能夠導致失敗的單元測試。)
3.You are not allowed to write any more production code than is sufficient to pass the one failing unit test.
(只允許編寫剛好能夠導致一個單元測試失敗的產品代碼。)

### 範例
```C#
 [TestMethod]
        public void Test_Add_FirstNumber為1_SecondNumber為2_計算結果應為3()
        {
            // Scenario: 第一個數字為1，第二個數字為2，呼叫使用 Calculator 的 Add 方法，傳入兩個數字，
            //           應回傳兩數相加結果 3.
            // Given: 第一個數字為 1
            // And: 第二個數字為 2
            // When: 呼叫使用 Calculator 的 Add 方法，並傳入兩個數字
            // Then: 應回傳計算結果 3

            // arrange
            var firstNumber = 1;
            var secondNumber = 2;
            var expected = 3;

            Calculator sut = new Calculator();

            // act
            var actual = sut.Add(firstNumber, secondNumber);

            // assert
            Assert.AreEqual(expected, actual);
        }
```