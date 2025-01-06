---
layout: post
title:  "Aussagekräftige Komponententests"
date:   18-01-11 12:30:00
excerpt: "Aussagekräftige Komponententests schreiben."
image:
thumb: /assets/img/thumbs/AAA.jpg
tags: [komponententests, testen, entwicklung, csharp]
categories: [posts, development]
comments: true
lang: de
ref: howto-write-meaningful-unittests
---


<!-- MDTOC maxdepth:6 firsth1:0 numbering:1 flatten:0 bullets:0 updateOnSave:1 -->

1. [Grundlegende Struktur von Testmethoden](#grundlegende-struktur-von-testmethoden)   
&emsp;1.1. [Das Triple-A-Muster](#das-triple-a-muster)   
2. [Benennung von Testmethoden](#benennung-von-testmethoden)   
&emsp;2.1. [Die typisch schlechte Praxis](#die-typisch-schlechte-praxis)   
&emsp;2.2. [Empfehlung von Roy Osherove ("The Art Of Unit Testing")](#empfehlung-von-roy-osherove-the-art-of-unit-testing)   
&emsp;2.3. [BD stylish](#bd-stylish)   
3. [Verhaltensgetriebene Tests](#verhaltensgetriebene-tests)   

<!-- /MDTOC -->

Ursprünglich hatte ich die folgenden Worte aus der Perspektive eines C#-Entwicklers geschrieben, sie sind aber nicht darauf beschränkt.

## Grundlegende Struktur von Testmethoden

Komponententests sollten vorzugsweise feinkörnig sein und einen sehr spezifischen Testfall abdecken. Es sollte vermieden werden, eine Reihe verschiedener Testfälle innerhalb einer Testmethode zu betrachten, da dies die Aussagekraft für fehlgeschlagene Tests beeinträchtigt.
Einer Komponente/Klasse, welche getestet werden soll, sollten eine oder mehrere Testklassen zugeordnet werden, die jeweils ein bestimmtes Testszenario in ihren öffentlichen Testmethoden abdecken.

### Das Triple-A-Muster

Um die vorstehenden Empfehlungen umzusetzen, sollten Sie sich genau an das AAA-Muster halten.

Diese besteht aus den folgenden 3 Phasen eines Testszenarios:

<table><tbody>
<tr><th><b>ARRANGE</b></th>
<td>Aufbereitung des Tests. Ressourcen werden aggregiert und das <b>SUT</b> (System Under Test) wird in einen bestimmten Zustand versetzt.</td></tr>
<tr><th><b>ACT</b></th>
<td>Die Ausführung der Testfunktionalität. Hier wird der Eingangszustand, der durch <b>ARRANGE</b> erzeugt wurde, in einen Ausgangszustand überführt, der dann in <b>ASSERT</b> überprüft wird.</td></tr>
<tr><th><b>ASSERT</b></th>
<td>Die Überprüfung der Ergebnisse der Phase <b>ACT</b>. Hier wird der aktuelle Status gegen einen erwarteten Soll-Zustand geprüft.</td></tr>
</tbody></table>

Vorteile:

- Die Testfunktionen sind klar von den Setup- und Verifizierungsschritten getrennt.
- Klärung und Fokussierung auf ein historisch erfolgreiches und allgemein notwendiges Set von Testschritten.
- Macht einige problematische Test-Geschmacksmuster deutlicher sichtbar:
   * Überprüfungen vermischt mit Ausführungs-Code.
   * Testmethoden, die versuchen, zu viele verschiedene Dinge auf einmal zu testen.

**Achtung:**\\
Vermeide, mehrere ACT / ASSERT Phasen innerhalb einer Prüfmethode zu kombinieren!
{: .notice--warning title="Achtung"}


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

## Benennung von Testmethoden

Die Namen der Prüfmethoden spiegeln sich in den Testergebnissen der Prüfberichte wider. Sie sollten daher möglichst aussagekräftig sein und gegen das Testszenario fein granuliert werden. Leider erfüllt die weit verbreitete typische Namensgebung diese Kriterien nicht.

### Die typisch schlechte Praxis

Testklassenname = Name der zu testenden Klasse + Suffix "Tests"    (z.B. `CalculatorTests`)

Testmethodenname = Name der zu testenden Methode... + Suffix "Test"

**Fazit:**\\
Zu wenig aussagekräftig und differenzierbar, wenn mehrere Testszenarien für die Methode abgedeckt werden müssen.
{: .notice--danger}

### Empfehlung von Roy Osherove ("The Art Of Unit Testing")

Verwendung von Testmethodennamen, welche aus 3 Teilen bestehen, die durch Unterstriche getrennt sind:

1. Teil: Name der zu prüfenden Methode
2. Teil: Beschreibung, welcher Status zu prüfen ist.
3. Teil: Erwartetes Verhalten

z.B.: `Divide_ZeroDivision_ThrowsException`.

**Fazit:**\\
Feinkörnig, aber sehr technisch, was zu Vorbehalten bei Nicht-Entwicklern führt (z.B. bei der Auswertung von Testberichten).
{: .notice--info}

### BD stylish

Nutzt mehr einen verhaltensgetriebenen (Behavior Driven (BD(D)) Stil! Teile der Anforderungen sollten im Methodennamen verwendet werden (Trennung der einzelnen Wörter durch Unterstriche).

z.B.: `Divide_by_Zero_should_throw_a_DivideByZeroException_error`

**Fazit:**\\
Sehr feinkörnig und aussagekräftig. Der zu testende Rahmen wird detailliert beschrieben und zwingt den Entwickler dazu, nur diesen in der Testmethode zu testen.
{: .notice--success}

## Verhaltensgetriebene Tests

Der verhaltensgetriebene Ansatz basiert auf Akzeptanztests, kann aber auch für Komponententests als Spezifikation für Klassenanforderungen angepasst werden. Er zielt auf eine gute Lesbarkeit im Geschäftsleben ab.
Mehrere Vertreter dieses Ansatzes verwenden das "Gherkin"-Sprachmuster, dessen Schlüsselwörter auf das AAA-Muster abgebildet werden können:

| AAA         | Englisches Schlüsselwort | Deutsches Schlüsselwort  |
|-------------|--------------------------|--------------------------|
| **Arrange** | Given                    | Gegeben sei / Angenommen |
| **Act**     | When                     | Wenn                     |
| **Assert**  | Then                     | Dann                     |


- Beispiel unter Verwendung meines Test-Frameworks [MS.TestPlatform](https://github.com/mcpride/MS.TestPlatform) (siehe auch: [BasicCalculatorSpecification.cs](https://github.com/mcpride/MS.TestPlatform/blob/master/Examples/Calculator.Core/Specifications/BasicCalculatorSpecification.cs)):

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