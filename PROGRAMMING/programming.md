# 💻 Programming Languages: Compiled vs. Interpreted

### Why do compiled languages (like C++, Go) need to be translated upfront?

They require a "Build" (Compilation) phase. Before you can even run the program, a special tool (the compiler) takes your entire source code and translates it all at once into a ready-made machine code file (e.g., an `.exe` file in Windows). Only this finished, ready-to-run file is deployed to the server.

### Why is Python different?

Python is a scripting (interpreted) language. There is no "Build" phase here because the code is translated on the fly. 

When you run a script, a special program installed on the server (called the Python Interpreter) reads your `.py` text file line by line. In a fraction of a second, it translates the code into zeros and ones for the CPU in real-time as the program executes.