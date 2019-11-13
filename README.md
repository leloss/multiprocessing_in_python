# multiprocessing_in_python
Some exercises and lessons learned on actually doing multiprocessing in Python

I decided to create this tutorial after realizing that, as a Data Scientist just trying to "make my code run faster", I spent way too much time trying to understand and use apply multi-processing in real world problems with Python. These real world problems have characteristics that, frustratingly enough, go beyond the narrow scope of Stack Overflow answers and toy examples. These characteristics include one or more of the following:

- large amounts of data, 
- data structures with large overhead,
- non serializable data structures,
- logics that are non-trivial to vectorize,
- and more.

So here is a few lessons I learned in the last couple of years. Hopefully somone out there may benefit from my own struggles and experience.

We will start with an example and improve it step-by-step throughout this tutorial. 

## 1. The basic code

This example stores financial transaction data as a graph, creating nodes for the transacting parties and edges for each financial transaction. 

```
#Open file, load names to graph nodes
import networkx
import csv

filename='transactions.csv'
TXN_DATA=[]
G=nx.Graph()
num=-1
with open(filename) as fin:
    reader=csv.reader(fin,delimeter=',',encoding='utf-8')
    for line in reader:
        num+=1
        if num == 0:
            continue
        line=line.strip().split(',')
        TXN_DATA.append(line)
        G.add_node(num,name=line[0],address='')
```


```
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
```
