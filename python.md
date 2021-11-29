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
