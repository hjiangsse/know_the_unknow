# 1. About python package:
## pack a python package:
    a good link page i know to pack a package:  https://packaging.python.org/tutorials/packaging-projects/
## install python package faster
    pip install --force-reinstall rdsp_client-0.0.4-py3-none-any.whl -i https://pypi.tuna.tsinghua.edu.cn/simple

# 2. Panda dataframe
## pandas data frame filter by index
a pandas dataframe *data*:  
```
                         open     close      high       low     value    volume  
2005-01-01 09:00:00  0.552438  0.280006  0.332489  0.864800  0.474178  0.827809  
2005-01-01 09:01:00  0.338546  0.654993  0.659209  0.888915  0.480564  0.651820  
2005-01-01 09:02:00  0.962025  0.463558  0.092408  0.296262  0.284668  0.448517  
2005-01-01 09:03:00  0.874030  0.560000  0.261885  0.534373  0.747222  0.849829  
2005-01-01 09:04:00  0.816664  0.541799  0.288249  0.005849  0.881951  0.320970  
...                       ...       ...       ...       ...       ...       ...  
2005-01-01 10:45:00  0.018078  0.270889  0.007631  0.856010  0.886102  0.501975  
2005-01-01 10:46:00  0.299404  0.692678  0.894732  0.307676  0.606485  0.598515  
2005-01-01 10:47:00  0.116287  0.597343  0.722067  0.811261  0.061974  0.193382  
2005-01-01 10:48:00  0.434326  0.493112  0.003419  0.900076  0.049911  0.114951  
2005-01-01 10:49:00  0.524980  0.527988  0.546563  0.498139  0.226002  0.306714  
```

在上述dataframe中，我想过滤出2005-01-01 10:45:00之后的所有数据， how?
``` python
data_stamps = list(data.index.values)

last_row_stamp = '2005-01-01 10:45:00'
insert_stamps_filtered = filter(lambda x: x > last_row_stamp, data_stamps)
insert_stamps_list = list(insert_stamps_filtered)

filtered_data = data.filter(items = insert_stamps_list, axis=0)
```

# 3. python function decorators
## 3.1 functions as first class objects in python
1. A function is an instance of the Object type.
2. You can store the function in a variable.
3. You can pass the function as a parameter to another function.
4. You can return the function from a function.
5. You can store them in data structures such as hash tables, lists, …

``` python
#test function as the first class object
def reverse_sentence(sentence):
    splited = sentence.split()
    splited.reverse()
    return ' '.join(splited)

print(reverse_sentence("Principles Techniques and Tools of Compilers"))

symbols_reverse = reverse_sentence
print(symbols_reverse("Principles Techniques and Tools of Compilers"))

def upper_sentence(sentence):
    return sentence.upper()

print(upper_sentence("Principles Techniques and Tools of Compilers"))

#function(s) as parameters as other function
def sentence_factory(funcs, sentence):
    for func in funcs:
        sentence = func(sentence)
    return sentence

print(sentence_factory([reverse_sentence, upper_sentence], "Principles Techniques and Tools of Compilers"))

#return function from a function
def create_square_sum():
    def square_sum(x, y):
        return x * x + y * y
    return square_sum

square_sum = create_square_sum()
print(square_sum(1,2))
```

## 3.2 decorator as a way of modify the behaviour of functions and classes
The syntax of a function decorator: 
``` python
@gfg_decorator
def hello_decorator():
    print("Gfg")

"""
Above code is equivalent to -

def hello_decorator():
    print("Gfg")
    
hello_decorator = gfg_decorator(hello_decorator)
"""
```
The example of a simple decorator:
``` python
# defining a decorator
def hello_decorator(func):
    def inner1():
        print("Hello, this is before function execution")
        func()
        print("This is after function execution")
    return inner1
 
# defining a function, to be called inside wrapper
def function_to_be_used():
    print("This is inside the function !!")

function_to_be_used = hello_decorator(function_to_be_used) 
# calling the function
function_to_be_used()
```
A decorator as timer for a function:
``` python
import time
import math
 
# decorator to calculate duration
# taken by any function.
def calculate_time(func):
    def inner1(*args, **kwargs):
        begin = time.time()
        func(*args, **kwargs)
        end = time.time()
        print("Total time taken in : ", func.__name__, end - begin)
    return inner1

@calculate_time
def factorial(num):
    time.sleep(2)
    print(math.factorial(num))
 
# calling the function.
factorial(10)
```

# 4. Do profile in python：
## 4.1 cProfile:
### Introduction:
cProfile and profile provide deterministic profiling of Python programs. A profile is a set of statistics that describes how often and for how long various parts of the program executed. These statistics can be formatted into reports via the *pstats* module.
### Basic Usage:
#### profile a function
Run the following code snippet:
``` python
import cProfile

def accsum(n):
    """acc sum"""
    res = 0
    for i in range(n):
        res += (i+1)
    return res

if __name__ == "__main__":
    cProfile.run('accsum(10000000)')
```

*accsum* is a function to calculate the cumulative sum of the number from 1 to n(n > 1), we use cProfile to profile this function, follow is running result and my mechine messages:
``` bash
  4 function calls in 0.531 seconds

   Ordered by: standard name

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.000    0.000    0.531    0.531 <string>:1(<module>)
        1    0.531    0.531    0.531    0.531 cpython_test.py:3(accsum)
        1    0.000    0.000    0.531    0.531 {built-in method builtins.exec}
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
```

* ncalls: number of calls of the function[3/1 stand 1 call but 3 recur call]
* tottime: total time spend in the function, not include time spend in sub functions
* percall: tottime / calls
* cumtime: cumulative time spend in this function and subfunctions(even for recur functions)
* percall: cumtime / calls
* filename:lineno(function): which function in which file's which line
  
#### use *pstat* module's Stat to manipulate profile result:
Change privious code, this will generate a cumsum.stat file in current work directory:
``` python
if __name__ == "__main__":
    cProfile.run('accsum(10000000)', 'cumsum.stat')
```

Write a new script to analysize profile result
``` python
import pstats
from pstats import SortKey

if __name__ == "__main__":
    p = pstats.Stats('cumsum.stat')
    p.strip_dirs().sort_stats(-1).print_stats()
    p.sort_stats(SortKey.NAME).print_stats()
    p.sort_stats(SortKey.CUMULATIVE).print_stats()
    p.sort_stats(SortKey.TIME).print_stats()
```
The running result is:
``` bash
Fri Dec 31 09:02:46 2021    cumsum.stat

         4 function calls in 0.531 seconds

   Ordered by: standard name

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.000    0.000    0.531    0.531 <string>:1(<module>)
        1    0.531    0.531    0.531    0.531 cpython_test.py:3(accsum)
        1    0.000    0.000    0.531    0.531 {built-in method builtins.exec}
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}


Fri Dec 31 09:02:46 2021    cumsum.stat

         4 function calls in 0.531 seconds

   Ordered by: function name

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.000    0.000    0.531    0.531 {built-in method builtins.exec}
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
        1    0.000    0.000    0.531    0.531 <string>:1(<module>)
        1    0.531    0.531    0.531    0.531 cpython_test.py:3(accsum)


Fri Dec 31 09:02:46 2021    cumsum.stat

         4 function calls in 0.531 seconds

   Ordered by: cumulative time

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.000    0.000    0.531    0.531 {built-in method builtins.exec}
        1    0.000    0.000    0.531    0.531 <string>:1(<module>)
        1    0.531    0.531    0.531    0.531 cpython_test.py:3(accsum)
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}


Fri Dec 31 09:02:46 2021    cumsum.stat

         4 function calls in 0.531 seconds

   Ordered by: internal time

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.531    0.531    0.531    0.531 cpython_test.py:3(accsum)
        1    0.000    0.000    0.531    0.531 {built-in method builtins.exec}
        1    0.000    0.000    0.531    0.531 <string>:1(<module>)
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
```
#### profile a full script or a module
``` bash
python -m cProfile [-o output_file] [-s sort_order] (-m module | myscript.py)
```

* -o: wirte the profile result to a file
* -s: how to sort the profile result
* -m: module name

We add another function to the code snippet:
``` bash
import cProfile

def acc_sum(n):
    """acc sum"""
    res = 0
    for i in range(n):
        res += (i+1)
    return res

def acc_square_sum(n):
    """acc square sum"""
    res = 0
    for i in range(n):
        res += (i+1) * (i+1)
    return res

if __name__ == "__main__":
    acc_sum(1000000)
    acc_square_sum(1000000)
```

run follwing command:
``` bash
python -m cProfile -s tottime cpython_test.py
```
running result:
``` bash
         279 function calls (278 primitive calls) in 0.148 seconds

   Ordered by: internal time

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.094    0.094    0.094    0.094 cpython_test.py:10(acc_square_sum)
        1    0.053    0.053    0.053    0.053 cpython_test.py:3(acc_sum)
        6    0.000    0.000    0.000    0.000 {built-in method nt.stat}
        1    0.000    0.000    0.000    0.000 {built-in method io.open_code}
        4    0.000    0.000    0.000    0.000 <frozen importlib._bootstrap_external>:1431(find_spec)
        1    0.000    0.000    0.000    0.000 {built-in method marshal.loads}
        1    0.000    0.000    0.000    0.000 {method 'read' of '_io.BufferedReader' objects}
        1    0.000    0.000    0.000    0.000 <frozen importlib._bootstrap_external>:969(get_data)
       20    0.000    0.000    0.000    0.000 <frozen importlib._bootstrap_external>:62(_path_join)
        1    0.000    0.000    0.000    0.000 {built-in method builtins.__build_class__}
       20    0.000    0.000    0.000    0.000 <frozen importlib._bootstrap_external>:64(<listcomp>)
        1    0.000    0.000    0.148    0.148 cpython_test.py:1(<module>)
```
OK! the result is sorted by *tottime*!
## 4.2 line-by-line profile:
### install line-profiler:
``` bash
pip install line_profiler
```
### do line profile:
``` python
def acc_square_add(n):
    """acc square sum"""
    res = 0
    for i in range(n):
        res += (i+1) * (i+1)
    return res

@profile
def slow_function():
    print("slow function begin")
    acc_square_add(10000000)
    print("slow function end")

if __name__ == "__main__":
    slow_function()
```

run the following command:
``` bash
kernprof -l -v line_profiler_test.py
```
"-v" will show the result in the stdout, this is the result:
``` bash
slow function begin
slow function end
Wrote profile results to line_profiler_test.py.lprof
Timer unit: 1e-06 s

Total time: 2.51533 s
File: line_profiler_test.py
Function: slow_function at line 8

Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
     8                                           @profile
     9                                           def slow_function():
    10         1         84.0     84.0      0.0      print("slow function begin")
    11         1    2515077.9 2515077.9    100.0      acc_square_add(10000000)
    12         1        169.8    169.8      0.0      print("slow function end")
```
You can easily find *acc_square_add* is the bottleneck of our *slow_function*.
Also kernprof will generate a *.lprof* in current working directory, we can 
check it use the following command:
``` bash
python -m line_profiler line_profiler_test.py.lprof
```
the running result:
``` bash
Timer unit: 1e-06 s

Total time: 2.51533 s
File: line_profiler_test.py
Function: slow_function at line 8

Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
     8                                           @profile
     9                                           def slow_function():
    10         1         84.0     84.0      0.0      print("slow function begin")
    11         1    2515077.9 2515077.9    100.0      acc_square_add(10000000)
    12         1        169.8    169.8      0.0      print("slow function end")
```
Same as the privious running result!

## 4.3 memory profile:
Do memory profiling could be very slow compare to cpu profiling, be careful! 
### package install:
pip install -U memory_profiler
### profile line-by-line memory usage:
Example Code: mem_prifile.py
``` python
@profile
def my_func():
    a = [1] * (10 ** 6)
    b = [2] * (2 * 10 ** 7)
    del b
    return a

if __name__ == '__main__':
    my_func()
```

Run code:
``` bash
python -m memory_profiler mem_profile.py
```

Running result:
``` bash
Filename: mem_profile.py
Line #    Mem usage    Increment  Occurrences   Line Contents
=============================================================
     1   40.422 MiB   40.422 MiB           1   @profile
     2                                         def my_func():
     3   48.055 MiB    7.633 MiB           1       a = [1] * (10 ** 6)
     4  200.645 MiB  152.590 MiB           1       b = [2] * (2 * 10 ** 7)
     5   48.055 MiB -152.590 MiB           1       del b
     6   48.055 MiB    0.000 MiB           1       return a
```
* Mem usage: after the line of code, the memory usage of the programe.
* Increment: how many memory difference caused by this line of code.

### profile with decorator:
Code:
``` python
from memory_profiler import profile

@profile
def my_func():
    a = [1] * (10 ** 6)
    b = [2] * (2 * 10 ** 7)
    del b
    return a
	
if __name__ == '__main__':
    my_func()
```

Run code:
``` bash
python mem_profile.py
```

Running result:
``` bash
Filename: mem_profile.py
Line #    Mem usage    Increment  Occurrences   Line Contents
=============================================================
     3     40.4 MiB     40.4 MiB           1   @profile
     4                                         def my_func():
     5     48.0 MiB      7.6 MiB           1       a = [1] * (10 ** 6)
     6    200.6 MiB    152.6 MiB           1       b = [2] * (2 * 10 ** 7)
     7     48.0 MiB   -152.6 MiB           1       del b
     8     48.0 MiB      0.0 MiB           1       return a
```
The result is just the same as the line-by-line mode.
### time-based memory usage:
Sometimes it is useful to have full memory usage reports as a function of time (not line-by-line) 
of external processes (be it Python scripts or not). In this case the executable mprof might be useful.
Code:
``` python
@profile
def my_func():
    a = [1] * (10 ** 6)
    b = [2] * (2 * 10 ** 7)
    del b
    return a

@profile
def your_func():
    c = [1] * (10 ** 6)
    d = [2] * (2 * 10 ** 7)
    del d
    return c

if __name__ == '__main__':
    my_func()
    your_func()
```

Run time-based profile:
``` bash
mprof run mem_profile.py
```
This is the command output:
```
mprof: Sampling memory every 0.1s
running new process
running as a Python program...
```

Now do memory usage plot:
```bash
mprof plot --flame
```

This is the memory profile graph:
![mem_profile_graph](./pic/memprofile/mem_profile.png)
# 5. Python grpc:
python -m grpc_tools.protoc -I../proto --python_out=. --grpc_python_out=. ../proto/service.proto
