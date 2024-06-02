# TC20237-contextFreeGrammer
## Generating and Cleaning a Restricted Context Free Grammar

### Description

I chose to generate the grammar to create a simple, indefinite, one-variable integral. 
This is to be constructed with constants (numbers 0...9), the variable x, the integral symbol
which in this case is going to be represented by a back slash '\', the exponentiation 
symbol '^' and finally the differential 'dx'. All of this to construct our grammar. 

  // Some examples of how the grammar could generate include: 

  - \ 2x^2 dx + \1 dx
  - \ 58x^5 dx - \ 6x^2 dx + \ 7 dx
  - \ 15x^2 dx + 17x^8 dx - \ 10 dx

The complexity of integral expressions can go an extremely long way by introducting 
definite integrals. Double or even triple integrals, and by having more than one variable. 
This gets complex real quick, so the grammar was shortened to only accept one variable
definite integrals due to this. 

With the previous context in mind, we can create this using a RCFG (Restricted Context
Free Grammar). This is what I will be working with for this evidence. So first I will construct a base 
grammar that will be ambiguous and left-recursive. Then I'll clean the grammar in order to 
have it ready for it to be parsed using nltk in python. 


### Models of the grammar

First we need to **generate a generic grammar that recognizes de language**. 
Rochester University states that a CFG must contain the following: 

- "A set of terminal symbols, which are the characters of the alphabet that appear in the strings generated by the grammar."
- "A set of nonterminal symbols, which are placeholders for patterns of terminal symbols that can be generated by the nonterminal symbols."
- "A set of productions, which are rules for replacing (or rewriting) nonterminal symbols (on the left side of the production) in a string with other nonterminal or terminal symbols (on the right side of the production)."
- "A start symbol, which is a special nonterminal symbol that appears in the initial string generated by the grammar."

This was the first approach to generating a grammar.

- S -> INT EXPR DIFF
- EXPR -> T
- T -> VAR | CONST | T_L | T_R | EXP
- T_L -> VAR T_L | CONST T_L | EXP
- T_R -> T_R VAR | T_R CONST | T_R EXP | ε
- INT -> '\'
- DIFF -> 'dx'
- VAR -> 'x'
- CONST -> '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9'
- EXP -> '^' T | ε

This grammar is problematic as we have two main issues. Ambiguity and left recursion. 
Ambiguity happens because we can reach the same output through various different ways. 
In this case the rules T_L and T_R are able to get the same result, just the former 
through left most derivations and the latter through right most derivation. So this 
introduces ambiguity. And on the rule T_R you access yourself first on the right hand side, 
leading to a potential infinite loop, hence left recursion. Now let's clean this grammar. 

![imagen](https://github.com/SebastianCo1126/TC20237-contextFreeGrammer/assets/150994751/57a7a5d4-9183-450c-a698-4df26f58aefe)

In this image we can see that we can create to different trees to arrive to the same result of 
\ 2x^2 dx, which indicates ambiguity. This is because the grammar was written to have right 
and left most derivation. Now let's remove ambiguity first by just sticking to one 
way of derivation. And let's take care of left recursion as well, present in: 

  - T_R -> T_R VAR | T_R CONST | T_R EXP | ε

As T_R can be called indefinetly. So re-working the grammar first we add the posibility of 
concatenating the expressions with + or -, as the previous grammar only considered one expression:

  - S -> E
  - E -> P E'
  - E' -> P '+' E | P '-' E | ε
  - P -> INT T DIFF

Next, letst just stick to one way of derivation, and add a way for constants to access a variable 
after them with or without and exponent: 

  - T -> VAR EXP | CONST T'
  - T' -> VAR EXP | ε

And with that we can add the rest of the variables that just access the different terminals: 

  - INT -> '\'
  - DIFF -> 'dx'
  - VAR -> 'x'
  - CONST -> '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9'
  - EXP -> '^' T | ε

Finally we construct our unambiguous, and non-left-recursive grammar, in which I additionaly
I substituted the integral symbol for a '%' for better readability in the tests: 

- S -> E
- E -> P E'
- E' -> P '+' E | P '-' E | ε
- P -> INT T DIFF
- T -> VAR EXP | CONST T'
- T' -> VAR EXP | CONST T | ε
- INT -> '%'
- DIFF -> 'dx'
- VAR -> 'x'
- CONST -> '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9'
- EXP -> '^' T | ε

Nos using Princeton University's LL(1) visualization tool 
(available in https://www.cs.princeton.edu/courses/archive/spring20/cos320/LL1/) 
we can check if the grammar is correct. The followinf is how the grammar was written 
in a syntax for the website: 

- E ::= P E'
- E' ::= + E 
- E' ::= - E
- E' ::=  ''
- P ::= INT T DIFF
- T ::= VAR EXP 
- T ::= CONST T'
- T ::= ''
- T' ::= VAR EXP 
- T' ::= CONST T
- T' ::= ''
- INT ::= \
- DIFF ::= dx
- VAR ::= x
- CONST ::= 0 
- CONST ::= 1
- CONST ::= 2
- CONST ::= 3
- CONST ::= 4
- CONST ::= 5
- CONST ::= 6
- CONST ::= 7
- CONST ::= 8
- CONST ::= 9
- EXP ::= ^ T
- EXP ::= ''

It passed the test, and is availbe as an LL(1). First here are the 'First and Follow table': 

![imagen](https://github.com/SebastianCo1126/TC20237-contextFreeGrammer/assets/150994751/ab6e1bac-2237-4756-8f1e-b358ef2a4856)

And the transitions table, which is cropped as it didn't completely fit on the screenshot, but the rest 
of the columns are just other constants: 

![imagen](https://github.com/SebastianCo1126/TC20237-contextFreeGrammer/assets/150994751/a3043f04-5476-4c44-94ac-df57ac657d53)


### Implementation

My implementation was made using python and the library nltk. For ease of use, this was made
in a Jupiter Notebook, using Google Colab, so the tests are recommended to be run in this environment.
However the pure python code file is included as well for the tests to be runned locally. 

The files are : 

- CFG.ipynb
- cfg.py (for locally running the tests)

The tests for the model are present as a coded implementation inside the Jupiter Notebook, and 
are the same tests validated in the LL(1) princeton parser. The Princeton test's are presented
in the following section (Valid Tests and Invalid Tests) of this README file. 


### Valid tests

Here is a test for the following expression:
- \ 2 x ^ 2 dx

![imagen](https://github.com/SebastianCo1126/TC20237-contextFreeGrammer/assets/150994751/2ed3f534-8b9b-421d-854a-23e4dd52853a)

And another one for a concatenated integral:
- \ 2 x ^ 2 dx + \ 7 dx

![imagen](https://github.com/SebastianCo1126/TC20237-contextFreeGrammer/assets/150994751/21239771-d43f-47fb-a918-72559df27cb0)

Another concatenated integral:
- \ 2 x ^ 2 dx + \ 5 x ^ 3 dx

![imagen](https://github.com/SebastianCo1126/TC20237-contextFreeGrammer/assets/150994751/fd661822-e002-4373-bbf9-7a7939644350)


And another test passed for a more complex concatenated integral:
- \ 8 1 x ^ 8 dx - \ 7 2 1 x ^ 2 dx + \ 5 dx

![imagen](https://github.com/SebastianCo1126/TC20237-contextFreeGrammer/assets/150994751/4a537888-172d-4cae-83c1-ef73ffdec44d)

Finally a final test with a bigger concatenated integral:
- \ 1 0 2 1 x dx - \ 1 2 x dx + \ 1 3 x ^ 9 dx - \ 1 dx

![imagen](https://github.com/SebastianCo1126/TC20237-contextFreeGrammer/assets/150994751/17b41976-0144-4e8c-a0be-b8f67927fb03)

### Invalid tests

First a test that doesn't end in dx, making it invalid:
- \ 1 3 x ^ 2

![imagen](https://github.com/SebastianCo1126/TC20237-contextFreeGrammer/assets/150994751/eaa9ddbb-b8ac-4549-9e44-f29b3bb66484)

Now a test with the variable 'y' which is not accepted:
- \ 7 x ^ 2 dx + \ 1 7 y dx

![imagen](https://github.com/SebastianCo1126/TC20237-contextFreeGrammer/assets/150994751/47b0d4aa-4bfb-49c8-b352-1a386b4734d3)

Now a test with an exponent greater than > 10 which is not defined in this grammar: 
- \ 4 5 1 x ^ 1 0 dx 

![imagen](https://github.com/SebastianCo1126/TC20237-contextFreeGrammer/assets/150994751/19f982fc-f4a7-48cc-8096-6b1cb3a2a7b7)

Now a test with a missing integral symbol :
- \ 1 0 dx + \ 9 x ^ 4 - 10 dx

![imagen](https://github.com/SebastianCo1126/TC20237-contextFreeGrammer/assets/150994751/03a77ab0-17aa-4eac-b696-b0be08c15ca6)

Finally a test that a number not separated by a space:
- \ 9 8 x ^ 5 dx - \ 1 x ^ 2 + \ 1 0 0 dx + \ 12 x dx

![imagen](https://github.com/SebastianCo1126/TC20237-contextFreeGrammer/assets/150994751/12852a16-9dd9-4eb0-95a4-63544dbdbf2b)

### Analysis 

A context free grammar (according to Geeks for Geeks) is "Context Free 
Grammar is formal grammar, the syntax or structure of a formal language can be described 
using context-free grammar (CFG), a type of formal grammar. The grammar has four tuples: (V,T,P,S)." 
https://www.geeksforgeeks.org/what-is-context-free-grammar/

  A -> bAcD
  D -> aaCb
  C -> b

  * in this example, the uppercase letters represent non-terminals, the lowercase letters
    represent termials and each line represents a rule. 

A restricted CFG is just a CFG that has additional restrictions to make it simpler. My implementation
of a grammar to write simple integrals is a CFG because, first of all, on the left hand side of the grammar
we only have non-terminal symbols. When you live in a higher level of the Extended Chomsky's Hierarchy (context-sensitive, 
or higher) you have terminals on the left hand side of the production. And it is not a regular grammar,
because it can have ambiguity, as it was modeled first. It was then later changed with a certain 
set of rules for it not to be ambiguous. And this language cannot be represented by a nomral finite
state automata, it need to be represented with a pushdown automata so we can observe the states because it has
stacked storage. 

### Conclusions

After modeling two different solutions. One with an ambiguous grammar and one with an unambiguous grammar, 
we can conclude that the latter is the more optimal solution, as we only have one parse tree for each input string. 
This also helped us eliminate left recursion and avoid infinite loops. The time complexity of this solution is simple 
to analyze as it only includes a linear for loop of the tokens separated by the tokenizer, giving a time complexity
of O(n):

    for i < n // n being the number of tokens produced by the tokenizer
            if i in n 
                i++ // parsing continues 
            else 
                break;

This solution is the most efficient one for this case because we eliminated ambiguity, left recursion, and it correctly 
displays the single parsing trees of each string. It is adequate for the proposed portion of the integrals to be 
taken into account, and it accepts concatenation. 

### Changes notes
For correction of my evidence, I added a python file with a simplified version of the tests to be runned locally. For this 
to work we need to install nltk. For this, I recommend using the pip installer. You can run the following command to have the 
library installed: 

- pip install nltk

Once this is installed. You should be able to run the script by simply heading to the command prompt, and navigating to the 
directory in which the file is located and writing in the terminal: 

- python cfg.py

### Bibliography 

- Geeks for Geeks. (22 nov 2021). _Construct Pushdown Automata for given languages_. https://www.geeksforgeeks.org/construct-pushdown-automata-given-languages/
- Benjamín Valdés. (n.d.). _Grammars and Languages_. https://docs.google.com/document/d/1XqKP5Wj0_NDnmkb36AsJ5AD1qPX38Hvb5yQwNp9zrFI/edit
- Rochester University. (n.d.) _Context-Free Grammars_. https://www.cs.rochester.edu/u/nelson/courses/csc_173/grammars/cfg.html








