---
layout: post
title:  IPython in Terms of Pry
date:   2016-12-11 14:00:00 -0600
categories: python ruby
---

One of the awesome things about Ruby is how flexible it is. With its dynamic typing, method\_missing, reflection, and open classes, you can pretty much do anything, from anywhere, at any point in your programs! As the programmer, you can ask Ruby for a very sharp scalpel and it will gladly hand you one. [This can be problematic if you're not careful](http://weblog.rubyonrails.org/2013/1/8/Rails-3-2-11-3-1-10-3-0-19-and-2-3-15-have-been-released/), but this same flexibility is what allows Ruby code to be so concise and elegant. Ruby's flexibility also allows for tools like [Pry](http://pryrepl.org/). Pry is a powerful [REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop) that can also serve as a debugger by allowing you to embed it inside your Ruby programs.

My team at work primarily uses Python. I haven't worked with Python much prior to my current position, but fortunately for me Ruby and Python have a lot in common; they're both dynamically typed and interpreted languages. And they've both borrowed lots of ideas from Perl and Lisp. Yet, even though Ruby and Python are very similar, the tooling, libraries, and philosophy surrounding the two languages are different.

In my limited experience with Python, [IPython](https://ipython.org/) is the closest thing to Pry I've found in Python so far. In addition to providing an improved REPL, IPython also has "support for interactive data visualization and use of GUI toolkits". There's also a "notebook" interface which looks very similar to [Mathematica](https://en.wikipedia.org/wiki/Wolfram_Mathematica).

Here's some useful stuff I've figured out how to do in IPython and how to do the same in Pry:


|I want to                   | In Pry         | In IPython                                   |
|:---------------------------|:---------------|:---------------------------------------------|
| Embed a REPL in a program  | `binding.pry`  | `IPython.embed()`                            |
| Show the call stack        | `show-stack`   | `import traceback; traceback.print_stack()`  |
| Run a shell command        | `.ls $HOME`    | `!ls $HOME`                                  |
| Show methods and variables | `ls`           | `whos` or `who` or `locals()` or `globals()` |
| Exit the current REPL      | `exit`         | `exit` or `exit()` or `quit()`               |
| Kill the process           | `exit-program` | `import os; os._exit(0)`                     |
| Show `foo`'s documentation | `? foo`        | `foo?`                                       |
| Show `foo`'s source        | `$ foo`        | `foo??`                                      |
| Inspect `foo`              | `cd foo; ls`   | `foo.__dict__`                               |
