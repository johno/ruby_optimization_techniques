# An Analysis of Ruby Optimization Techniques

The Ruby programming language has experienced a recent period of intense adoption and growth due to its excellent speed of iteration and due, in no small part, to the acceptance of the Ruby on Rails web framework within the startup sphere. While support is growing steadily for the language, it is largely dismissed as not having effective scalability, or having far slower runtimes than more traditional strongly-typed complex languages. In this article, we propose that many sophisticated techniques exist to enhance Ruby’s performance both in using existing runtimes to compile ruby to statically typed languages, and in  using common anti-patterns to improve performance natively. Through experimentation and thorough research we conclude that Ruby performs competitively against it’s similar scripting language counterparts, and can see increases of [XXXXX]% in many cases.

## Introduction

In recent years, the Ruby programming language has grown its community and established itself as a valuable and popular tool for many tasks [O’Donoghue, 2014]. The success of Ruby on Rails as a prototyping framework as well as a full-stack solution for some larger companies has brought forth a myriad of techniques to ensure that the language’s speed differences compared to similar languages are minimal. Ruby’s slower performance as compared to C or Java is attributed to interpreted execution, dynamic typing, meta-programming support, and the Global Interpreter Lock [Odaira, Castanos, and Tomari, 2014]. This increase in popularity has caused a large number independent optimization efforts to arise from large corporations such as IBM, as well as efforts from the Ruby open-source community.

With each of these techniques there exist certain sacrifices, but in this exploration we will conclude that the best practices for stable, performant Ruby code exist by utilizing the newest versions of the core language properly, and not by utilizing other third party interpreters or solutions.

## An Overview of the Ruby Language

Ruby is an object oriented, dynamically-typed, high-level scripting language. It is a programming language that was written for humans and just happens to run on computers. It's intended to promote developer happiness through simplicity, elegant libraries, and terse, readable syntax. Ruby uses duck typing, meaning type is determined through methods and properties.

> When I see a bird that walks like a duck and swims like a duck and quacks like a duck, I call that bird a duck.

Heim, Michael (2007). Exploring Indiana Highways. Exploring America's Highway. p. 68. ISBN 978-0-9744358-3-1.

### Program Execution at a High Level

CODE => TOKENIZATION => PARSE TREE => COMPILATION => YARV INSTRUCTIONS

#### Tokenizing a Ruby program

When a Ruby program is executed, it first tokenizes the program. This means that the contents are converted into a collection of tokens with associated types.

#### Parsing the tokens

Ruby uses the LALR (Look-Ahead Left Reversed Rightmost Derivation) Parser to apply meaning to the tokens and construct the Abstract Syntax Tree.

#### The compilation step

The compilation step was introduced with Ruby 1.9, and is where the YARV (Yet Another Ruby Virtual Machine) comes into play. It translates the code into _bytecode_, or YARV instructions.

#### YARV instructions for a simple program

```
~|||$ irb
2.1.1 :001 > code = <<CODE
2.1.1 :002"> puts 1 + 2
2.1.1 :003"> CODE
 => "puts 1 + 2\n" 
2.1.1 :004 > puts RubyVM::InstructionSequence.compile(code).disasm
== disasm: <RubyVM::InstructionSequence:<compiled>@<compiled>>==========
0000 trace            1                                               (   1)
0002 putself          
0003 putobject_OP_INT2FIX_O_1_C_ 
0004 putobject        2
0006 opt_plus         <callinfo!mid:+, argc:1, ARGS_SKIP>
0008 opt_send_simple  <callinfo!mid:puts, argc:1, FCALL|ARGS_SKIP>
0010 leave            
 => nil
```

### A significant performance boost

The introduction of the compilation step and YARV have significantly helped the execution speed of Ruby programs. However, there's always room for more improvements.
