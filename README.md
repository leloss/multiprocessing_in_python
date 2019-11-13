# Dummy's Tutorial to Python Multiprocessing
### Lessons learned on parallelizing code in Python

I decided to create this tutorial after realizing that, as a Data Scientist just trying to "make my code run faster", I spent way too much time trying to understand and use apply multi-processing in real world problems with Python. These real world problems have characteristics that, frustratingly enough, go beyond the narrow scope of Stack Overflow answers and toy examples. These characteristics include one or more of the following:

- large amounts of data, 
- data structures with large memory overhead,
- non serializable data structures,
- logics that are non-trivial to vectorize,
- and more.

So here is a few lessons I learned in the last couple of years. Hopefully somone out there may benefit from my own struggles and experience.

We will start with an example and improve it step-by-step throughout this tutorial. 

## 1. The basic code

This example stores financial transaction data as a graph, creating nodes for the transacting parties and edges for each financial transaction. 

```
#Open file, load names to graph nodes
import networkx as nx
import csv

filename='transactions.csv'
TXN_DATA=[]
G=nx.Graph()
num=-1
def load_file(filename):
    with open(filename) as fin:
        reader=csv.reader(fin,delimeter=',',encoding='utf-8')
        for line in reader:
            num+=1
            if num == 0:                                    #skip header
                continue
            TXN_DATA.append(line)                           #save transactions
            G.add_node(num,name=line[0],address=line[1])    #create a new node in graph
        
def process(nrow,row):
    
    return (nrow,data)
    
def collect():
    return

def main():
    for nrow,row in enumerate(TXN_DATA):
        collect(process(nrow,row))

#Run all
load_file(filename)
main()
```

## 2. Adding parallelism

```
from multiprocessing import Pool, cpu_count

if parallel:
    _num_proc=int(round(cpu_count()-1))
    pool = Pool(processes=_num_proc)

def main(pool):
    for nrow,row in enumerate(TXN_DATA):
        if parallel:
            pool.apply_async(process, args=(nrow,row), callback=collect_result)
        else:
            collect_result(process(nrow,row))
            
if parallel:
    main(pool)
    pool.close()
    pool.join()
else:
    main('')

```


## 3. Adding batching

```
from multiprocessing import Pool, cpu_count

if parallel:
    _num_proc=int(round(cpu_count()-1))
    pool = Pool(processes=_num_proc)
    
def main(pool):
    for nrow,row in tqdm(enumerate(TXN_DATA)):
        if nrow == 0 or len(row) < 5:
            continue
        if parallel:
            pool.apply_async(process, args=(nrow,row), callback=collect_result)
            if nrow%_batch == 0:
                pool.close()
                pool.join()
                pool = Pool(processes=_num_proc)
        else:
            collect_result(process(nrow,row))
            
if parallel:
    main(pool)
    pool.close()
    pool.join()
else:
    main('')
```


## 4. Adding serialization

```
import functools
import threading

lock = threading.Lock()

def synchronized(lock):
    def wrapper(f):
        @functools.wraps(f)
        def inner_wrapper(*args, **kw):
            with lock:
                return f(*args, **kw)
        return inner_wrapper
    return wrapper

class Singleton(type):
    _instances = {}
    @synchronized(lock)
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super(Singleton, cls).__call__(*args, **kwargs)
        return cls._instances[cls]

class tsG(metaclass=Singleton):
    def __init__(self):
        self.G=nx.Graph()
    def add_node(self,idx,data=''):
        self.G.add_node(idx,data=data)
    def add_edge(self,idx1,idx2):
        self.G.add_edge(idx1,idx2)
    def set_data(self,idx,**data):
        for d in data:
            self.G.node[idx][d[0]]=d[1]
    def nodes(self,data=False):
        return self.G.nodes(data)
```

## 5. Putting it all together
