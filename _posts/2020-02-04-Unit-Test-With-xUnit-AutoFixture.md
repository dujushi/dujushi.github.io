---
title: 'Unit Test With xUnit and AutoFixture'
---
[AutoFixture](https://github.com/AutoFixture/AutoFixture){:target="_blank"} is created to minimize the 'arrange' phase of your unit tests. [Unit Testing: Activate Easy Mode](http://www.whoiskevinrich.com/unit-testing-activate-easy-mode){:target="_blank"} shows me how easy it is to write unit tests when you have the correct toolsets. [My demo program](https://github.com/dujushi/xUnitWithAutoFixture){:target="_blank"} follows the same idea, but with my favorite libraries. 

## AutoFixture.xUnit2.AutoMoq
[This project](https://github.com/dujushi/xUnitWithAutoFixture/tree/master/AutoFixture.xUnit2.AutoMoq){:target="_blank"} creates an `AutoMockDataAttribute` that extends `AutoDataAttribute`. It uses `Moq` to create a fixture. You can also extends `InlineAutoDataAttribute` and `MemberAutoDataAttribute` [like this](https://github.com/ObjectivityLtd/AutoFixture.XUnit2.AutoMock/tree/master/src/Objectivity.AutoFixture.XUnit2.AutoMoq/Attributes){:target="_blank"}. 

## Cannot Mock Extension Method With Moq
[ProductServiceTests](https://github.com/dujushi/xUnitWithAutoFixture/blob/master/xUnitWithAutoFixture.Business.UnitTests/Services/ProductServiceTests.cs){:target="_blank"} shows you how to mock `IMemoryCache`. The `TryGetValue` method we use in the project is an extension method. If we try to mock it directly, the library will complain. The walk around is to use the native method used by the extension method.

## Mock IHttpContextAccessor
[AcceptLanguageProviderTests](https://github.com/dujushi/xUnitWithAutoFixture/blob/master/xUnitWithAutoFixture.Api.UnitTests/Providers/AcceptLanguageProviderTests.cs){:target="_blank"} demonstrates how to mock `IHttpContextAccessor`.

## Unit Test Practices
[Unit testing best practices with .NET Core and .NET Standard](https://docs.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices){:target="_blank"} is a good article to read.