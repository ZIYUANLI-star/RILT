# RILT
'RILT' is an automated unit test generation tool that combines a requirements-driven assertion checking
 mechanism with an iterative high-coverage test generation strategy.'RILT' decomposes assertions into
 "test expressions" and "actual values," utilizing system functional requirements to guide LLM(gpt-3.5-turbo and gpt-4o) to
 generate more accurate assertions.RILT analyzes the initial test cases, extracts
 uncovered code lines, generates semantically independent code snippets, and provides targeted hints
 for LLM to generate additional tests. Mutation testing is used to further optimize the quality and
 effectiveness of the tests.
![](https://github.com/ExpertiseModel/MuTAP/blob/master/MuTAP_diagram.png)

## 1. Module Under Test Detector
The module-under-test detector is responsible for analyzing the structure of source code to extract functions,methods,classes, and other elements, as well as identifying their interdependencies and required packages. 

## 2. Test Case Generator
RILT employs a large language model (LLM) to auto matically generate initial test cases for the modules under test. 

## 3.  Syntax Corrector
We first use Python’s AST module to parse the unit test suite and generate an abstract syntax tree (AST) to ensure compliance with Python’s syntax rules.If the parsing process identifies a syntax error, we create a prompt marker based on the error type (for example, "Please correct the syntax error in the following code:") and rely on LLM’s syntax repair function for preliminary repairs. If syntax errors still exist, the runtime error corrector in the execution phase will make additional repairs to ensure that the test case can eventually be successfully executed.

## 4. Runtime Error Corrector
RILT monitors and records any runtime errors after executing the test suite. It then con structs prompt tokens from the captured error information and invokes the LLMtoperform targeted fixes. Once repairs are completed, the test suite is executed again, and this process is iterated as needed. Experiments show that this “detect–repair–retest” loop effectively resolves the majority of runtime errors, substantially improving the executability and stability of the test suite.

## 5.  Assertion Actual-Value Checker
RILT first extracts the main expression of each assertion (i.e., the left side of the assertion) and combines it with the functional annotations derived from the tested module into a hint label, while referencing the type information of the originally generated assertion value. Guided by these contextual elements, the LLMsimulates the reasoning of professional test engineers and regenerates appropriate "actual values" consistent with the functional description. This way, the updated assertions retain the original type requirements while accurately reflecting the expected functionality of the code under test,resulting in a more targeted and higher-quality unit test suite.

## 6. Test Case Augmenter
RILT first leverages Python’s Coverage tool to analyze the test suite coverage to identify specific lines in the tested module that are still not covered. Given that generating test cases at the granularity of smaller code snippets can help LLM achieve highercoverage,wecombinetheseuncoveredlineswith their surrounding context to form semantically complete uncovered code snippets. Subsequently, the function code containing these uncovered snippets is provided as a hint prefix, and hint markers are made to highlight that covering these specific snippets is a core requirement. In this process,we strategically reduce the total amount of context provided to LLM and instead increase the level of detail of certain key elements. This step-by-step approach can better guide LLM to understand the functionality and intent behind the

## 7. Mutation Testing
RILT combines MutPy with alarge language model (LLM) to generate mutants. If MutPy fails in more complex scenarios, LLM steps in to generate the required mutants. RILT then tests the generated mutants using the unittest framework and calculates the mutation score (i.e., the percentage of mutants that were killed). If the score is below 100%, it indicates that there are surviving mutations. The systemthendesignsnewhintsbasedonthese surviving mutations and creates supplementary test cases using LLM, which are added to the test suite. This iterative process gradually improves the overall quality of test cases.

## 8. Feedback Module
RILT first executes the test suite to identify and isolate all test cases where the expected and actual values do not match. It then uses these faulty test cases along with the code under test to formulate appropriate hints. By leveraging such test cases, RILT uses a large language model (LLM) to perform feedback-based inspection of the code under test and generate modification suggestions. This last step completes the full cycle of unit testing.

