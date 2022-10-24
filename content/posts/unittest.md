---
title: "Unittest"
date: 2022-10-24T15:50:08+08:00
draft: true
tags: ["單元測試"]
type: post
author: "Steve Lin"
description: ""
---

# 單元測試

## 定義
- 一個測試案例只測一種情境
- 最小的測試單位
- 不跟外部資源相依
- 沒有邏輯判斷
- 測試案例之間的相依性為0
## 特性FIRST
- Fast：快速。每個單元測試的執行時間應該要很短。
- Independent：獨立。單元測試不與外部資源相依，單元測試之間也不能夠相依，要能夠單獨針對測試目標進行測試。
- Repeatable：可重複。所有的單元測試都可重複被執行，且不影響預期的結果。
- Self-Validating：自我驗證（可反應驗證結果）。單元測試執行不論成功或失敗，都應該要能夠從測試報告裡直接瞭解其意義或失敗的原因。
- Timely：及時。單元測試應該在產品程式碼完成的當下就可以驗證執行結果是否符合預期。
## 驗證方式
- 驗證測試目標物件的方法回傳值是否符合預期
- 驗證測試目標物件的狀態改變是否符合預期
- 驗證測試目標物件與相依物件的互動是否符合預期
## 3A原則
- Arrange
    - 初始化目標物件
    - 初始化方法參數
    - 建立模擬物件行為
    - 設定環境變數期望結果
- Act
    - 實際呼叫測試目標物件的方法 
- Assert
    - 驗證政目標是否如預期運作
## 測試方法的命名
- 被測試方法名稱+測試場景描述+預期行為
- 可以用中文
- 受測對象:可使用target或sut(system under test)
- 預期結果 - expected
- 實際執行結果 - actual
- 範例:ToInt_傳入為正整數的物件_應傳回正確的正整數
### 快捷鍵
- testc建立測試類別
- testm 建立測試方法 
- Ctrl+R, T: 執行單一測試
- Ctrl+R, A: 執行所有測試（開發時最常使用）
- Ctrl+R, Ctrl+T: 偵錯單一測試（測試失敗時，最常使用）
- Ctrl+R, Ctrl+A: 偵錯所有測試
### 測試相關標籤Attribute
- [TestClass()]
- [TestMethod()]
- [Owenr(“User_Name”)]
- [TestCategory(“Calculator”)]
- [TestProperty(“Calculator”, “Add”)]
- [Ignore]
### 建立群組
- 可以依據類別、持續期間、執行結果、TestCategory內容、測試專案區分群組
- 也可以建立播放清單
## Assert類別
- Assert.AreEqual(expected, actual);
- Assert.AreNotEqual(expected, actual);
- Assert.IsTrue(actual);
- Assert.IsFalse(actual);
- Assert.IsNull(actual);
- Assert.IsNotNull(actual);
- Assert.IsInstanceOfType(actual, typeof(expectedType));
- Assert.IsNotInstanceOfType(actual, typeof(wrongType));
驗證集合是否包含指定的項目
- CollectionAssert.Contains(actual, element);
- CollectionAssert.DoesNotContain(actual, element);
驗證集合的項目與順序是否相等
- CollectionAssert.AreEqual(expected, actual);
- CollectionAssert.AreNotEqual(expected, actual);
驗證集合的項目是否對等（不按順序）
- CollectionAssert.AreEquivalent(expected, actual);
- CollectionAssert.AreNotEquivalent(expected, actual);
- Assert.AreEquals()方法:確認兩個指定的物件相等，如果這些物件都不相等，判斷提示就會失敗。
- Assert.AreSame()方法:確認兩個指定的物件變數參考相同的物件。 如果它們參考不同的物件，判斷提示就會失敗。
### FluentAssertions擴充方法
[文件連結](https://fluentassertions.com/introduction)
#### 範例
```C#
[TestMethod()]
        public void IsNullOrEmpty_傳入空白的字串_Should_be_True()
        {
            // arrange
            var sut = "";

            // act
            var actual = sut.IsNullOrEmpty();

            // assert
            actual.Should().BeTrue();
        }
```
## private,internal方法的測試
- 盡量不對private方法做測試
- 如果真的要測試 Private 方法，可使用 PrivateObject 類別或 PrivateType 類別
- PrivateObject類別:允許測試程式碼呼叫受測試之程式碼上可能因為不是 public 而無法存取的方法和屬性。
- PrivateType類別:表示可用來存取私用靜態實作之私用類別的型別
- 被測試專案的AssemblyInfo要加上`[assembly: InternalsVisibleTo("測試專案名稱")]
`
```C#
[TestMethod]
        public void TestPrivateAdd()
        {
            // 使用 PrivateObject 類別，測試私有方法 Add
            PrivateObject po = new PrivateObject(new Calculate());

            var expected = 3;

            var actual = po.Invoke("Add", 1, 2);

            Assert.AreEqual(expected, actual);
        }

        [TestMethod]
        public void TestInternalStaticAdd()
        {
            // 使用 PrivateType 類別，測試靜態Internal方法 AddStatic
            PrivateType po = new PrivateType(typeof(Calculate));

            var expected = 3;

            var actual = po.InvokeStatic("AddStatic", 1, 2);

            Assert.AreEqual(expected, actual);
        }
```
## Unit Test Event Hooks
- ClassInitializeAttribute 類別: 該方法所包含的程式碼須**用於測試類別中的所有測試都完成執行之前**，以便配置此測試類別所使用的資源。
- ClassCleanupAttribute 類別: 該方法所包含的程式碼**用於測試類別中的所有測試都完成執行之後**，以便釋放此測試類別所佔用的資源。
- TestInitializeAttribute 類別: 此方法要**在測試之前執行**，以便配置和設定測試類別中所有測試所需的資源。
- TestCleanupAttribute 類別: 該方法所包含的程式碼必須**於測試完成執行之後使用**，以便釋放此測試類別中所有測試所佔用的資源。

## 使用 Nsubstitute 產生 Stub 物件
- 加入Nsubstisute套件
- Substitue.For< T >()產生 Interface的Stub物件
- 定義Stub物件的行為與回傳值Stub物件.方法(你要輸入參數).Returns(你想要回傳的值)
```
 var profileRepository = Substitute.For<IProfileRepository>();
 profileRepository.GetPassword(account).Returns("000");
```
#### 驗證是否有互動 使用Received()跟DidNotReceive()
```C#
public void IsValid_驗證為False的情境()
        {
            // arrange
            var account = "kevin";
            var validateCode = "000222";

            var profileRepository = Substitute.For<IProfileRepository>();
            profileRepository.GetPassword(account).Returns("000");

            var hashTokenService = Substitute.For<IHashTokenService>();
            hashTokenService.GetRandom(account).Returns("111");

            // step 1: arrange, 建立 mock object
            ILog log = Substitute.For<ILog>();

            var sut = new AuthenticationService(profileRepository, hashTokenService, log);

            // Step 2: act
            var actual = sut.IsValid(account, validateCode);

            // step 3: assert, mock object 是否有正確互動
            actual.Should().BeFalse();
            log.Received(1).Save(Arg.Is<string>(x => x.Contains(account)));
        }

```
## Expected Object
> 比對兩個物件是否相等 https://github.com/derekgreer/expectedObjects
```C#
public void TestComposedObject()
        {
            var expectedPerson = new Person
            {
                Id = 1234,
                Name = "Boo",
                Order = new Order { Id = 101, Price = 201 }
            }.ToExpectedObject();

            var actualPerson = new Person
                               {
                                   Id = 1234,
                                   Name = "Boo",
                                   Order = new Order { Id = 101, Price = 201 }
                               };

           //Assert.AreEqual(expectedPerson, actualPerson);
            expectedPerson.ShouldEqual(actualPerson);
            //比較不完全相同物件用這個方法
             expected.ShouldMatch(actual);
        }

```
## 產生測試資料
### AutoFixture
> https://github.com/AutoFixture/AutoFixture
範例
```C#
        public async Task UpdateAsync_傳入一筆帳戶主檔_修改成功應回傳1()
        {
            //arrange
            var fixture = new Fixture();
            var sut = this.GetSystemUnderTest();
            var accountDto = fixture.Build<AccountDto>()
                .OmitAutoProperties()
                .Create();

            var account = fixture.Build<Account>()
                .OmitAutoProperties()
                .CreateMany(3)
                .AsQueryable()
                .BuildMock();

            this._uow.GetRepository<Account>()
                .Get(Arg.Any<Expression<Func<Account, bool>>>())
                .Returns(account);

            this._uow.SaveChangesAsync().Returns(Task.FromResult(1));
            //act
            var actual = await sut.UpdateAsync(accountDto);

            //assert
            actual.Should().Be(1);
        }
        public void UpdateAsync_傳入一筆資料_查無此筆資料應拋出ObjectNotFoundException()
        {
            //arrange
            var sut = this.GetSystemUnderTest();
            var fixture = new Fixture();
            var accountDto = fixture.Build<AccountDto>()
                .OmitAutoProperties()
                .Create();
            var account = new List<Account>();
            var mock = account.AsQueryable().BuildMock();

            this._uow.GetRepository<Account>()
                .Get(Arg.Any<Expression<Func<Account, bool>>>())
                .Returns(mock);
            //act
            Func<Task> action = async () => await sut.UpdateAsync(accountDto);

            //assert
            action.Should().Throw<ObjectNotFoundException>()
                .Where(e => e.Message.Contains($"找不到帳戶主檔"));
        }
        [TestMethod()]
        public void RelativeSortArrayTest_輸入IP字串超過5_應拋出ArgumentException()
        {
            //arrange
            var fixture = new Fixture();
            var model = fixture.Build<InputParameter>().With(c => c.IP, "123456").Create();

            var sut = GetSystemUnderTest();

            //act
            var exception = Assert.ThrowsException<ArgumentException>(
                () => sut.RelativeSortArray(model));

            //assert
            exception.ParamName.Should().Be("IP");
            exception.Message.Should().Contain("The field IP must be a string with a maximum length of 5.");
        }
```

### XUnit
- `[Fact]`:標註為測試方法，不能有傳入參數
- `[Theory]`: 代表執行相同程式碼但具有不同輸入引數的測試套件，可以用InlineData傳入測試參數
- `[InlineData]`: 屬性會指定這些輸入的值。

```C#
[Fact]
public void IsPrime_InputIs1_ReturnFalse()
{
    var primeService = new PrimeService();
    bool result = primeService.IsPrime(1);

    Assert.False(result, "1 should not be prime");
}
[Theory]
[InlineData(-1)]
[InlineData(0)]
[InlineData(1)]
public void IsPrime_ValuesLessThan2_ReturnFalse(int value)
{
    var result = _primeService.IsPrime(value);

    Assert.False(result, $"{value} should not be prime");
}

```