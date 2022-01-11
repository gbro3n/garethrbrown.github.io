---
public: true
layout: post
title: "Converting Between Enum Types By Value in C#"
date: 2021-07-01 00:00 +0000
tags: C# enum
---

If you find yourself having to convert between values in two identical Enums and there is no way that your code base allows you to reconcile those enums into the same type, or if you have enums with only letter case differences, this is a means of a conversion from source to target using generics.

```csharp
public static class EnumExtensions
{
    public static TTargetEnumType ConvertByName<TTargetEnumType>(this Enum source, bool verifyAllEnumMembersMatch = false, bool ignoreCase = false) where TTargetEnumType : struct, Enum
    {
        if (verifyAllEnumMembersMatch)
        {
            foreach (var sourceEnumValue in Enum.GetValues(source.GetType()))
            {
                bool parsed = Enum.TryParse<TTargetEnumType>(sourceEnumValue.ToString(), ignoreCase, out _);

                if (!parsed)
                {
                    throw new InvalidOperationException($"Cannot convert to {typeof(TTargetEnumType).Name} from {source.GetType().Name}, failure on source value {sourceEnumValue}");
                }
            }

            foreach (var targetEnumValue in Enum.GetValues(typeof(TTargetEnumType)))
            {
                bool parsed = Enum.TryParse(source.GetType(), targetEnumValue.ToString(), ignoreCase, out _);

                if (!parsed)
                {
                    throw new InvalidOperationException($"Cannot convert to {source.GetType().Name} from {typeof(TTargetEnumType).Name}, failure on target value {targetEnumValue}");
                }
            }
        }

        string sourceString = source.ToString();

        return Enum.Parse<TTargetEnumType>(sourceString, ignoreCase);
    }
}
```
Some basic tests for this class are as follows

 ```csharp
public class EnumExtensionTests
{
    private enum TestEnum1
    {
        A,
        B,
        C
    }

    private enum TestEnum2
    {
        A,
        B,
        C
    }

    private enum TestEnum3
    {
        A
    }

    [Fact]
    public void TestBasicConversion()
    {
        var testEnum2AAs1A = TestEnum1.A.ConvertByName<TestEnum2>();

        Assert.Equal(TestEnum2.A, testEnum2AAs1A);

        var testEnum2BAs1B = TestEnum1.B.ConvertByName<TestEnum2>();

        Assert.Equal(TestEnum2.B, testEnum2BAs1B);

        var testEnum2CAs1C = TestEnum1.C.ConvertByName<TestEnum2>();

        Assert.Equal(TestEnum2.C, testEnum2CAs1C);
    }

    [Fact]
    public void TestThrowsOnMismatchedEnumsAndVerifyAllEnumMembersMatch()
    {
        Assert.Throws<InvalidOperationException>(() =>
        {
            TestEnum3.A.ConvertByName<TestEnum2>(ignoreCase: false, verifyAllEnumMembersMatch: true);
        });

        Assert.Throws<InvalidOperationException>(() =>
        {
            TestEnum2.A.ConvertByName<TestEnum3>(ignoreCase: false, verifyAllEnumMembersMatch: true);
        });
    }

    [Fact]
    public void TestDoesNotThrowOnMismatchedEnumsAndVerifyAllEnumMembersMatchIsFalse()
    {
        TestEnum3.A.ConvertByName<TestEnum2>(ignoreCase: false, verifyAllEnumMembersMatch: false);

        TestEnum2.A.ConvertByName<TestEnum3>(ignoreCase: false, verifyAllEnumMembersMatch: false);
    }

    [Fact]
    public void TestThrowsOnMismatchedEnumsAndVerifyAllEnumMembersMatchFalse()
    {
        Assert.Throws<ArgumentException>(() =>
        {
            TestEnum1.C.ConvertByName<TestEnum3>(ignoreCase: false, verifyAllEnumMembersMatch: false);
        });
    }
}
```