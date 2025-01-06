---
layout: post
title:  "How to write meaningful unit tests"
date:   18-01-11 12:30:00
excerpt: "How to write meaningful unit tests."
image:
thumb: /assets/img/thumbs/AAA.jpg
tags: [unittests, testing, development, csharp]
categories: [posts, development]
comments: true
lang: en
ref: howto-write-meaningful-unittests
---


<!-- MDTOC maxdepth:6 firsth1:0 numbering:1 flatten:0 bullets:0 updateOnSave:1 -->

1. [Basic structure of test methods](#basic-structure-of-test-methods)   
&emsp;1.1. [Triple-A-Pattern](#triple-a-pattern)   
2. [Naming of test methods](#naming-of-test-methods)   
&emsp;2.1. [Typical bad practice](#typical-bad-practice)   
&emsp;2.2. [Recommendation by Roy Osherove ("The Art Of Unit Testing")](#recommendation-by-roy-osherove-the-art-of-unit-testing)   
&emsp;2.3. [BD stylish](#bd-stylish)   
3. [Behavior Driven Tests](#behavior-driven-tests)   

<!-- /MDTOC -->

I originally wrote the following words from a C# developer perspective but it's not limited to this.

## Basic structure of test methods

Unit tests should preferably fine-grained covering a very specific test case. It should be avoided to consider a number of different test cases within a test method because it suffers the meaningfulness for failed tests.
A unit/class to be tested should one or more tests classes assigned, each covering one specific test scenario in their public testing methods.

### Triple-A-Pattern

To implement the preceding recommendations, you should follow closely the AAA pattern.

This consists of the following 3 phases of a test scenario:

<table><tbody>
<tr><th><b>ARRANGE</b></th>
<td>Precondition of the test. Resources are aggregated and the <b>SUT</b> (System Under Test) is put into a particular state.</td></tr>
<tr><th><b>ACT</b></th>
<td>The execution of the test functionality. Here, the input state, which was produced by <b>ARRANGE</b>, is transferred to an output state, which is then checked in <b>ASSERT</b></td></tr>
<tr><th><b>ASSERT</b></th>
<td>The inspection of the results of the <b>ACT</b> phase. Here, the current status is checked against an expected target state.</td></tr>
</tbody></table>

Benefits:

- Clearly separates what is being tested from the setup and verification steps.
-  Clarifies and focuses attention on a historically successful and generally necessary set of test steps.
-  Makes some test smells more obvious:
   * Assertions intermixed with "Act" code.
   * Test methods that try to test too many different things at once.

**Attention:**\\
Avoid to combine several ACT / ASSERT phases within a test method!
{: .notice--warning title="Attention"}

```csharp
public class CalculatorTests
{
    [Fact]
    public void Calculates_correct_summary()
    {
        //// ARRANGE-Phase -------------------------------------
        var op1 = 4;
        var op2 = 3;
        var sut = new Calculator();

        //// ACT-Phase -----------------------------------------
        var actual = sut.Add(op1, op2);

        //// ASSERT-Phase --------------------------------------
        Assert.AreEqual(7, actual);
    }
}
```

## Naming of test methods

The names of the test methods are reflected in the test results in the test reports. Therefore they should be as informative as possible and finely granulated against the test scenario. Unfortunately, the widespread typical naming does not satisfy these criteria.

### Typical bad practice

Test class name = Name of class to test + Suffix "Tests"    (e.g. `CalculatorTests`)

Test method name = Name of method to test... + Suffix "Test"

**Conclusion:**\\
Not enough meaningful and differentiable if several test scenarios for the method must be covered.
{: .notice--danger}

### Recommendation by Roy Osherove ("The Art Of Unit Testing")

Test method name which consists of 3 parts, which are divided by underscores:

1. part: Name of the method to test
2. part: Description, which status has to be tested
3. part: Expected behavior

e.g..: `Divide_ZeroDivision_ThrowsException`

**Conclusion:**\\
Fine-grained, but very technical, resulting in caveats by non-developers (for example, in evaluation of test reports).
{: .notice--info}

### BD stylish

More Behavior Driven (BD(D)) style! Parts of the requirements should be used in the method name (separating the individual words with underscores).

e.g.: `Divide_by_Zero_should_throw_a_DivideByZeroException_error`

**Conclusion:**\\
Very fine-grained and meaningful. The test fact is described in detail and will force the developer just to test this in the test method.
{: .notice--success}

## Behavior Driven Tests

The beahvior driven approach is rooted in acceptance tests but can also be adapted for unit tests as specifications for class requirements. It aims to good business readability.
Several approaches use the "Gherkin" speech pattern whose keywords can be mapped to the AAA-pattern:


| AAA         | english key word | german key word          |
|-------------|------------------|--------------------------|
| **Arrange** | Given            | Gegeben sei / Angenommen |
| **Act**     | When             | Wenn                     |
| **Assert**  | Then             | Dann                     |

- Example which uses my test framework [MS.TestPlatform](https://github.com/mcpride/MS.TestPlatform) (see also: [BasicCalculatorSpecification.cs](https://github.com/mcpride/MS.TestPlatform/blob/master/Examples/Calculator.Core/Specifications/BasicCalculatorSpecification.cs)):

```csharp
using Microsoft.VisualStudio.TestPlatform.UnitTestFramework;
using MS.TestPlatform.UnitTestFramework.Specifications;

namespace Calculator.Core.Specifications
{
    [TestClass]
    [SpecificationDescription("As a user I want to perform mathematical calculations so that my head doesn't hurt.")]
    public class BasicCalculatorSpecification : Specification
    {
        [TestMethod]
        [ScenarioDescription("Add two to a calculator with zero on the accumulator.")]
        public void AddTwoToACalculatorWithZeroOnTheAccumulator()
        {
            Given("a calculator with zero on the accumulator", x => x.State.Calculator = new BasicCalculator(0))
                .When("I add two to the accumulator", x => x.State.Calculator.Add(2))
                .Then("the accumulator should show two", x => x.State.Calculator.Accumulator == 2);
        }

        //...
    }
}
```