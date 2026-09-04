# Recruitment Tasks

Piotr Dróżdż  
September 2026

## Introduction

Below are the solution to recruitment task that I was given. I also included the links to regex101
as required by the task instructions.

## Task 1: Capturing group extraction

In order to solve 1st task and extract PID, the following regex was used:

```regex
(?x)
^
.*
\[(?<process_pid>\d+)\]
.*
$
```

**Regex 1**: Solution for task 1.

Link to the solution as mentioned in a task:

• [Solution 1][]

## Task 2: Understanding "Mazurka" pattern

### a) and b)

Both a) and b) parts of this task can be solved by the same regex:

```regex
(?x)
^
.*
(?<cat>kota)
.*
$
```

**Regex 2**: Solution for task 2 a) and b).

Links to the solutions as mentioned in the task:

• [Solution 2a][]  
• [Solution 2b][]

### c)

My solution is Regex 2. The regex proposed in this point is:

```regex
(?x)
^
(?:k*+[^k]++)*? # "Mazurka" pattern
(?<cat_group>kota)
.*
```

**Regex 3**: Solution for task 2 a) and b) with use of "Mazurka" pattern.

Regex 3 consists of the beginning of the line "^", non capturing group `(?:k*+[^k]++)` repeated
0 or more times in a lazy way (the so called "Mazurka" pattern), named capturing group `(?<cat_group>kota)` and `.*` operator, the two last elements are the same as in Regex 2, except of a
endline `$` that is additionally present in Regex 2. The main difference is the "Mazurka" pattern,
that consists of abovementioned non capturing group. `k*+` means "search for 0 or more k letters
in a possesive way". Possesive way means preventing so called "backtracking". If regex engine
fits the pattern with possesive quantifier next to it, it will not go back to see if some parts of
fitted string fits the next pattern, thus potentially reducing the number of steps. `[^k]++` means
"search for anything but k appearing at least once and do it in a possesive way". The latter is
why the empty string and string consisting only of "k" letters are the only things that cannot
be matched by the content of non capturing group. Since this non capturing group is repeated
0 or more times in a lazy way, the whole "Mazurka" pattern can fit literally everything in a way
that prevents backtracking.
So the "Mazurka" pattern in Regex 3 is constructed to consume everything before "kota"
substring and prevent backtracking to reduce number of steps. Its effectiveness depends on how
many substring similar to "kota" appears before "kota". The more such substrings and the
more they are similar to "kota", the bigger number of steps the engine has to perform. On the
other hand, Regex 2 works differently. There is `.*` greedy operator on the beginning, that
takes the whole line at once, and than the engine backtracks until it will find "kota" substring.
It’s effectiveness depends linearly on the number of characters between the beginning of "kota"
substring and the end of the line. Since that number is similar in all the matched lines, the
number of steps is similar as well (around 31 – 32). In case of the last line, Regex 2 and Regex 3
are done in the similar number of steps (32 and 31, respectively), because there are more words
partially similar to "kota" before "kota" itself, and thus the effectiveness of Regex 3 decreases.

## Task 3: Applying "Mazurka" pattern in logs

### General solution

This task can be solved in a few ways. The simplest and most naive way would be to use c
letter as the basics of the "Mazurka" pattern. It would work with the example logs given in
the task, since all process names begin with "c", but it is not the general case since the process
name can begin with any other character. The second and most intuitive way would be to use
`\w` characters or `[[:alnum:]]` character class as a base for "Mazurka" pattern. This will work
as expected, however, it is not the most effective solution. As it was mentioned in Task 2 c),
the more substrings partially similar to the one that has to be fitted stand before this actual
substring, and the more they are similar to this substring, the more effectiveness of Regex 3
decreases. When one realises this, it is obvious that the most effective solutions would use the
characters that occur before the fitted substring rarely. In this situation, whitespace is such
character, thus giving the solution:

```regex
(?x)
^
(?:\s*+\S++)*?
\s(?<process_name>[\w.\/:]+)
\[(?<process_id>\d+)\]
.*
```

**Regex 4**: Application of "Mazurka" pattern to logs.

Link to the solution according to the task instruction:  
• [Solution 3][]  
In Regex 4 `[\w.\/:]` character class is used to fit the process name, because in general, the
process name can use more characters than calss `\w` contains. So Regex 4 is the solution that
works as "Mazurka" pattern is expected, consuming everything before the fitted substring and
preventing backtracking.

### Some other interesting solutions

The other group of characters that occur rarely before the fitted substring belongs to punctation
class (`\p{P}`). Thus one may ask: wouldn’t using of `\p{P}` character class as a basics for the
"Mazurka" pattern be more effective? The answer is: in case of this logs yes, but not in general.
The simplest implementation is as follows:

```regex
(?x)
^
(?:\p{P}*+[^\p{P}]++)*?
\p{P}[\w\s]+
\s(?<process_name>[\w.\/:]+)
\[(?<process_pid>\d+)\]
.*
```

**Regex 5**: Another application of "Mazurka" pattern to logs.

Link to the solution according to the task instruction:  
• [Another solution of task 3][]  
Regex 5 solve task 3 in smaller number of steps than Regex 4 (109 vs 132, respectively), but it
does not prevent backtracking since `\p{P}[\w\s]+` would fit at least part of the process name at
some point, and after that the engine would backtrack to the closest already fitted space to begin
fitting the `process_name`. So the final effectiveness of this solution would decrease linearly with
number of occurances of `\p{P}` characters before fitted string, as in calssic "Mazurka" pattern,
but it would also decrease linearly with the number of characters between the beginning of the
process name and the first `\p{P}` character in the process name. In order to prevent backtracking,
one may use lazy operator `\p{P}[\w\s]+?`, leading to the solution:

```regex
(?x)
^
(?:\p{P}*+[^\p{P}]++)*?
\p{P}[\w\s]+?
\s(?<process_name>[\w.\/:]+)
\[(?<process_pid>\d+)\]
.*
```

**Regex 6**: Yet another application of "Mazurka" pattern to logs.

Link to this solution:  
• [Yet another solution of task 3][]  
However, effectiveness of Regex 6 depends linearly on the number of characters before the beginning of the fitted substring and `\p{P}` prior to it. In this case, the number of steps is 134, which
is a little more than for Regex 4 (132).

[Solution 1]: https://regex101.com/?regex=%28%3Fx%29%0A%5E%0A.*%0A%5C%5B%28%3F%3Cprocess_pid%3E%5Cd%2B%29%5C%5D%0A.*%0A%24&testString=%3C30%3EFeb++1+07%3A46%3A49+gn21rs01+chronyd%5B2600%5D%3A+Source+10.146.65.226+online%0A%3C30%3EFeb++1+07%3A46%3A49+gn21crs01+chronyd%5B2601%5D%3A+Source+10.146.65.226+online%0A%3C30%3EFeb++1+07%3A46%3A49+gn21c+rs01+chronyd%5B2602%5D%3A+Source+10.146.65.226+online&flags=gm&flavor=pcre2&delimiter=%2F
[Solution 2a]: https://regex101.com/?regex=%28%3Fx%29%0A%5E%0A.*%0A%28%3F%3Ccat%3Ekota%29%0A.*%0A%24&testString=ala+ma+kota+Psota&flags=gm&flavor=pcre2&delimiter=%2F
[Solution 2b]: https://regex101.com/?regex=%28%3Fx%29%0A%5E%0A.*%0A%28%3F%3Ccat%3Ekota%29%0A.*%0A%24&testString=ala+ma+kota+Psota%0Aala+ma+kolorowego+kota+Psota%0Aala+ma+bardzo+kolorowego+kud%C5%82atego+kota+Psota&flags=gm&flavor=pcre2&delimiter=%2F
[Solution 3]: https://regex101.com/?regex=%28%3Fx%29%0A%5E%0A%28%3F%3A%5Cs*%2B%5CS%2B%2B%29*%3F%0A%5Cs%28%3F%3Cprocess_name%3E%5B%5Cw.%5C%2F%3A%5D%2B%29%0A%5C%5B%28%3F%3Cprocess_id%3E%5Cd%2B%29%5C%5D%0A.*&testString=%3C30%3EFeb++1+07%3A46%3A49+gn21rs01+chronyd%5B2600%5D%3A+Source+10.146.65.226+online%0A%3C30%3EFeb++1+07%3A46%3A49+gn21crs01+chronyd%5B2601%5D%3A+Source+10.146.65.226+online%0A%3C30%3EFeb++1+07%3A46%3A49+gn21c+rs01+chronyd%5B2602%5D%3A+Source+10.146.65.226+online&flags=gm&flavor=pcre2&delimiter=%2F
[Another solution of task 3]: https://regex101.com/?regex=%28%3Fx%29%0A%5E%0A%28%3F%3A%5Cp%7BP%7D*%2B%5B%5E%5Cp%7BP%7D%5D%2B%2B%29*%3F%0A%5Cp%7BP%7D%5B%5Cw%5Cs%5D%2B%0A%5Cs%28%3F%3Cprocess_name%3E%5B%5Cw.%5C%2F%3A%5D%2B%29%0A%5C%5B%28%3F%3Cprocess_pid%3E%5Cd%2B%29%5C%5D%0A.*&testString=%3C30%3EFeb++1+07%3A46%3A49+gn21rs01+chronyd%5B2600%5D%3A+Source+10.146.65.226+online%0A%3C30%3EFeb++1+07%3A46%3A49+gn21crs01+chronyd%5B2601%5D%3A+Source+10.146.65.226+online%0A%3C30%3EFeb++1+07%3A46%3A49+gn21c+rs01+chronyd%5B2602%5D%3A+Source+10.146.65.226+online&flags=gm&flavor=pcre2&delimiter=%2F
[Yet another solution of task 3]: https://regex101.com/?regex=%28%3Fx%29%0A%5E%0A%28%3F%3A%5Cp%7BP%7D*%2B%5B%5E%5Cp%7BP%7D%5D%2B%2B%29*%3F%0A%5Cp%7BP%7D%5B%5Cw%5Cs%5D%2B%3F%0A%5Cs%28%3F%3Cprocess_name%3E%5B%5Cw.%5C%2F%3A%5D%2B%29%0A%5C%5B%28%3F%3Cprocess_pid%3E%5Cd%2B%29%5C%5D%0A.*&testString=%3C30%3EFeb++1+07%3A46%3A49+gn21rs01+chronyd%5B2600%5D%3A+Source+10.146.65.226+online%0A%3C30%3EFeb++1+07%3A46%3A49+gn21crs01+chronyd%5B2601%5D%3A+Source+10.146.65.226+online%0A%3C30%3EFeb++1+07%3A46%3A49+gn21c+rs01+chronyd%5B2602%5D%3A+Source+10.146.65.226+online&flags=gm&flavor=pcre2&delimiter=%2F
