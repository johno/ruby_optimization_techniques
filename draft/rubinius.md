## What is Rubinius?
Rubinius is an implementation of the Ruby programming language and includes a bytecode virtual machine, Ruby syntax parser, bytecode compiler, generational garbage collector, just-in-time (JIT) native machine code compliler, and Ruby Core and Standard Libraries. [1](http://rubini.us/doc) Rubinius is written using Ruby and C++.

## History
Rubinius was originally created to be a Ruby virtual machine and runtime written in pure ruby. The current ruby interpreter is primarily writen in non-Ruby langauges such as C. From 2007 to 2013, the software company Engine Yard was a primary backer of Rubinius. During that time the focus of Rubinius evolved from creating a completely  bootstrapped Ruby VM to instead offering an implementation of Ruby with increased performance. Under this new direction, Rubinius partially abandoned the idea of bootstrapping the Ruby VM in all Ruby code, and instead sought to use C++ to increase performance and establish Rubinius as the fastest Ruby implementation. Sadly this effort was not successful, as is shown below. [2](http://programmingzen.com/2010/07/19/the-great-ruby-shootout-july-2010/)

https://www.dropbox.com/s/2og3qad0d05wryo/Screenshot%202014-03-31%2017.28.28.png

The one area that Rubinius has shined is in it's support for concurrency and multi-threading, which has become the focus of the Rubinius project. 

https://www.dropbox.com/s/ahdzxveyhcubg89/Screenshot%202013-11-06%2013.12.50.png?m=
https://www.dropbox.com/s/dx5s3zbntfsis04/Screenshot%202014-03-31%2017.36.58.png

## How does Rubinius Work?

