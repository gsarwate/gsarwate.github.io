---
layout: post
title:  "Check the Examination"
date:   2019-05-08 14:26:00 -0700
categories: elixir
---

## Introduction

This is a problem from [Codewars][1].

## Problem

The first input array contains the correct answers to an exam, like ["a", "a", "b", "d"]. The second one is "answers" array and contains student's answers.

The two arrays are not empty and are the same length. Return the score for this array of answers, giving +4 for each correct answer, -1 for each incorrect answer, and +0 for each blank answer(empty string).

If the score < 0, return 0.

For example:

```text
checkExam(["a", "a", "b", "b"], ["a", "c", "b", "d"]) → 6
checkExam(["a", "a", "c", "b"], ["a", "a", "b",  ""]) → 7
checkExam(["a", "a", "b", "c"], ["a", "a", "b", "c"]) → 16
checkExam(["b", "c", "b", "a"], ["",  "a", "a", "c"]) → 0
```

## Manual Solution (Human Intelligence)

The problem mentions two arrays. Since we are using Elixir, we will refer to them as lists.

Steps:

1. Start with a score of 0
2. Look at the first entry in the _correct_answer list_
3. Look at the entry in the _answer list_ at the same position as _correct_answer list_
   * If the answer entry is empty, add 0 to score
   * Otherwise compare the entry in the _correct_answer list_ with the entry in the _answer list_
     * If both entries match, then add 4 to the score
     * Otherwise, subtract 1 from the score
4. Look at the next entry in the _correct_answer list_ and repeat Step 3 if the entry exists
5. If there are no more entries in the _correct_answer list_
   * If the score is negative, then score is 0
   * Otherwise, report score and stop

## Solution (Elixir)

We could walk through the _correct_answer list_ and the _answer list_ in parallel and compare them as outlined in the manual solution. But, there is a better way.

We can combine both lists so that we can walk through that combined list and compare entries in the list to score.

The structure of our code will be:

```text
Combine list
|> Check answers and determine the score of the exam
|> Report final score
```

### Combine lists

The Elixir function [Enum.zip/2][ezip2] allows us to zip corresponding elements from two enumerables (in our case lists) into one list of tuples.

```elixir
Enum.zip(correct_answer_list, answer_list)
```

In our example, we are combining two enumerables with the Enum.zip/2 function, but the Elixir function [Enum.zip/1][ezip1] allows us to combine more than two enumerables, as well.

### Check answers and determine the score of the exam

Next, we want to check the entries in the list. Each entry is a tuple in the form of {correct_answer, answer}. We can compare elements within a tuple with each other. There are several ways to do so in Elixir. Let's take a look at a few of those approaches.

#### Build our own function to walk through list

```elixir
defp compare([], acc), do: acc

defp compare([head | tail], acc) do
  compare(tail, score(head, acc))
end

defp score({_expected, answer}, acc) when answer == "", do: acc
defp score({expected, answer}, acc) when expected == answer, do: acc + 4
defp score({expected, answer}, acc) when expected != answer, do: acc - 1
  ```

In the above code, we have created two functions to walk through the combined list. They both are private functions which we will call from our main function.

These functions accept two parameters. The first one is a tuple in the form of _{expected_answer, answer}_ which is an element of the combined list we built earlier. The second parameter is an accumulator, _acc_, with which we accumulate the results of our comparisions.

These functions allow us to walk through the list __recursively__ and, at the same time, accumulate the result of each of the comparisions.

The first function is a 'stop' condition for our recurssion. When we reach the end of the list, we stop and return the score. Note that, as the second function calls itself as the last thing, it is a _tail call_ and Elixir/Erlang will accordingly perform tail-call optimization.

The next three private score/2 functions are called from compare/2. They allow us to compare the expected answer with the actual answer. They allow us branching of conditional logical using multiclause functions.

We could have used conditional constructs such as if, case, and cond etc. in the compare/2 function. However, by using multiclause functions, the logic for handling the conditions is seperated from the higher level (calling) function. The code for conditions is cleaner and self-descriptive.

These three functions will update the _acc_ (score) and return it to the compare/2 function.

#### Use Enum module to reduce the list to score

```elixir
Enum.zip(correct, answer)
|> Enum.reduce(0, &score/2)
```

With this approach, we use the Elixir [Enum.reduce/3][ered3] module to walk through the combined list. The function takes in three arguments:

1. enumerable (combined list produced by Enum.zip/2)
2. accumulator (to store the result of reduction)
3. function (to compare the answers)

The accumulator will start with 0 (as per our manual Step 1). We will use the score/2 function to calculate the score. The score/2 function will get two parameters - a tuple for comparision and the accumulator.

### Report final score

```elixir
defp final_score(score) when score < 0, do: 0
defp final_score(score), do: score
```

Lastly, we can compare the score to check the final tally. If the score is negative, we return the tally as 0. Otherwise, we return the score as a tally.

## Final code

```elixir
defmodule CheckExam do
  def check_exam(correct, answer) do
    Enum.zip(correct, answer)
    |> Enum.reduce(0, &score/2)
    |> final_score()
  end

  defp score({_expected, answer}, acc) when answer == "", do: acc
  defp score({expected, answer}, acc) when expected == answer, do: acc + 4
  defp score({expected, answer}, acc) when expected != answer, do: acc - 1

  defp final_score(score) when score < 0, do: 0
  defp final_score(score), do: score
end
```

[1]: https://www.codewars.com/kata/check-the-exam
[ezip1]: https://hexdocs.pm/elixir/Enum.html#zip/1
[ezip2]: https://hexdocs.pm/elixir/Enum.html#zip/2
[ered3]: https://hexdocs.pm/elixir/Enum.html#reduce/3
