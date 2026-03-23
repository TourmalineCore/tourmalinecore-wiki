# Legacy-project on C: survival manual for interns

## What is this article about and for whom?

Some may think that C is obsolete, but it isn’t! It is still relevant in embedded programming, when working on operating system kernels and device drivers. More than 50 years after its inception, it remains one of the most productive, efficient and minimalist programming languages. In this article, we have collected all the topics, tasks, literature and useful links related to C that we use while onboarding new colleagues. And we would like to share this information with those who are also interested in developing community C.

## Theory and practice

One of the most important materials on C language we consider book "Essential C" by Nick Parlante. In our view, it contains the maximum number of basic features of C for its small volume. This publication will appeal to beginners who are only familiar with the programming language. The benefits of the documentation from Microsoft, which describes all aspects of the language in detail, should not be overlooked.

Task:
> At this point you can try writing a program consisting of 3 files: main.c, function.h and function.c. In the last file, describe any simple function (for example, adding two numbers), and in the header file, describe its prototype. Next, by using the #include directives in desired locations, you can call a function from the function.c file in the main.c file.

The program is written, now you want to run it, but for launch it must be compiled. The compilation process and its stages are well explained in this article. Although it shows everything in the example of C++, for a programmer on C it will also benefit.

Task:
> For practice, write any program and also compile it by stopping the compilation after preprocessing and see what you get.

Next, we'll take a closer look at some C fundamentals. To learn about basic data types, we recommend checking out Chapter 1 of "Essential C," the corresponding section of the documentation, and our mini-quiz on C knowledge. At this point, it's worth learning the difference between different integer data types, how much memory they occupy, what happens when signed and unsigned variables overflow, and about real data types and how to compare them.

Task:
> Given a string of random letters of the alphabet, replace each letter of the string with the next letter in the alphabet: a = b, b = c, … , z = a.

Next, it would be a good idea to review pointers, arrays, strings, and structures, which corresponds to Chapters 3 and 6 of "Essential C" and Quizzes 1 and 2 on pointers. Note how much memory is consumed by structures containing multiple different data types. It's also worth getting a good grasp of pointers, including pointers to pointers and pointers to functions, as this topic is not the easiest to understand.

Task:
> Try to reverse the string so that all its characters are in reverse order.

When talking about arrays, it's worth mentioning memory allocation, static and dynamic arrays, as well as the stack and heap. There's a general article on this (or rather, a translation of a foreign article), not specific to any programming language. Nevertheless, it gives a good overview of how memory management works.

Task:
> Create an array of 5 integers. Increase its size to 10, fill the new cells, and print the entire array.

The next important topic to consider is bit manipulation. Information on this can be found in the final sections of the aforementioned book, as well as at this link in the documentation. Key points to consider include setting, clearing, toggling, and checking arbitrary bits, as well as the difference between bit shifting signed and unsigned variables.

Task:
> Using the define directive, try setting or toggling a bit. You can also implement a function that returns the bits of a number, starting and ending with the specified ones, as an int value.

Next, we'd like to focus on building projects with CMake, as we use it frequently in our work and think it's worth reviewing. There are some good articles on this topic: "Introduction to CMake" and "Building with CMake for Beginners."

Task:
> As a good practice, you can now try building all the projects from previously completed tasks using this tool. You can also try changing the warning flags in CMakeLists.txt, then adding some "bad" code (unused variables, etc.). After that, make sure you see warnings when you build the project.

## Conclusion

In conclusion, I'd like to point out that studying all the links and materials provided here can be a challenging task if you're just starting out in programming. Understanding all the intricacies may take more time than with some other languages. However, with enough effort, you'll gain fundamental knowledge that's often overlooked when learning higher-level programming languages.