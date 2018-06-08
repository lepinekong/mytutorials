
# Red For Hopeless Programmers - Part III


### Introduction


In [part II](https://dev.to/lepinekong/red-for-hopeless-programmers---part-ii-258) you learn how to read a CSV file. 

In this tutorial, you'll learn how to do CRUD operations with CSV file as well as the ReAdABLE Human Format.


### Install VSCode and Red extension


In previous tutorials, you learnt how to use Red Console to create and test your program.

For more complex programs, it's worth to use [Visual Studio Code](https://code.visualstudio.com/) with [Red Extension](https://marketplace.visualstudio.com/items?itemName=red-auto.red).

Installing Visual Studio Code is straightforward. To install [Red Extension](https://marketplace.visualstudio.com/items?itemName=red-auto.red) click on install.


![https://i.imgur.com/Iec8luB.png](https://i.imgur.com/Iec8luB.png)
                    

### CRUD Operations for CSV File


Let's say we want to create a trading journal with these fields:

- date entry
- type order
- stock symbol
- stock quantity
- price entry


for example; create a transactions.csv file with these datas:

date entry,type order,stock symbol,stock quantity,price entry
07/12/2017,buy,GOOG,1000,1018.07
18/12/2017,buy,AAPL,100,175.95
12/03/2018,sell,AAPL,100,181.72

![https://i.imgur.com/foDdHzk.png](https://i.imgur.com/foDdHzk.png)
                    


| date entry | type order | stock symbol | stock quantity | price entry |
|------------|------------|--------------|----------------|-------------|
| 07/12/2017 | buy| GOOG | 1000   | 1018.07 |
| 18/12/2017 | buy| AAPL | 100| 175.95  |
| 12/03/2018 | sell   | AAPL | 100| 181.72  |

To append a new record use append:


```


        
```



### Read and Save Data file


We're going to use a very simple data format named 
the "ReAdABLE Human Format".

Create a transactions.read file with this data (see tips afterwards to create it directly from Red console):



```

T-2017.12.07-0001: [

    .SYMBOL: GOOG
    .CURRENCY: DOLLAR
    .BROKER: GOLDMAN-SACHS         

    .ACTION-ENTRY: BOUGHT
    .QTY-ENTRY: 1000
    .PRICE-ENTRY: 1018.07
    .DATETIME-ENTRY: 07/12/2017
    
    .ACTION-EXIT: SOLD
    .QTY-EXIT: 1000
    .PRICE-EXIT: 1018.07
    .DATETIME-EXIT: 07/12/2017

    .PARENT-TRANSACTION: none

    .COMMENT: {}
    .IMAGE: 

]

T-2017.12.07-0002: [

    .SYMBOL: ETH
    .CRYPTO-CURRENCY: YES

    .ACTION-ENTRY: BOUGHT
    .QTY-ENTRY: 1
    .PRICE-ENTRY: 0.06160000
    .DATETIME-ENTRY: 2018-01-04 01:38:01
    .FEES: 0.0000025
    .PRICE-CURRENCY: BTC
    .PRICE-CURRENCY-PRICE: 15109.07730545
    .BASE-CURRENCY: DOLLAR
    .BROKER: POLONIEX   

    .COMMENT: {}
    .IMAGE:               
    
    .ACTION-ENTRY: BOUGHT
    .QTY-ENTRY: 1
    .PRICE-ENTRY: 0.06360000
    .DATETIME-ENTRY: 2018-01-07 12:48:05
    .FEES: 0.0000025
    .PRICE-CURRENCY: BTC
    .PRICE-CURRENCY-PRICE: 14200.088
    .BASE-CURRENCY: DOLLAR
    .BROKER: POLONIEX        

    .PARENT-TRANSACTION: none

    .COMMENT: {}
    .IMAGE: 

]            
        
```


to read and save this file, use load and save command:


```

transactions: load %transactions.read    
save %transactions.read transactions        
        
```



### Create a new transaction

Let's say we want to create this new transaction:


```

T-2017.12.08-0001: [

    .SYMBOL: AAPL
    .CURRENCY: DOLLAR
    .BROKER: GOLDMAN-SACHS         

    .ACTION-ENTRY: BOUGHT
    .QTY-ENTRY: 100
    .PRICE-ENTRY: 175.95
    .DATETIME-ENTRY: 18/12/2017

]            
        
```


this is quite easy to append a new block with append/only method:


```

            append/only transactions T-2017.12.08-0001       
        
```



Since we also have to add label "T-2017.12.08-0001:" the code should be:



```

            append/only transactions to-set-word 'T-2017.12.08-0001 ; or to-set-word "T-2017.12.08-0001"
            append/only transactions T-2017.12.08-0001     
        
```


Let's create a function .append-key-value optionally with an alias append-key-value 


```

.append-key-value: function[block key value][

        append/only block to-set-word key
        append/only block value
        return block
]

append-key-value: :.append-key-value ; : will return the body of .append-key-value method      
        
```



So that you can now call it like this for our above transaction T-2017.12.07-0002:



```

.append-key-value transactions 'T-2017.12.08-0001 T-2017.12.08-0001            
        
```



### Modify a transaction

Let's say we want to change:


```

            .DATETIME-ENTRY: 18/12/2017
        
```


to:


```

            .DATETIME-ENTRY: 18/12/2017/13:23:45 ; use / for date time separator not space
        
```


First we must find the record using the key 'T-2017.12.08-0001. 
The select method allows this:


```

            transaction: select transactions 'T-2017.12.08-0001
        
```



Next, we must find .DATETIME-ENTRY in the block with find method:



```

            find block to-word key          
        
```



find is a pointer to the position of the block and return the rest of the block.
To get the index, use index? method:



```

index: index? find block '.DATETIME-ENTRY
        
```



It's then easy to get the next element index:



```

            next-index: index + 1         
        
```


which is necessary to modify its value like this:


```

            transaction/:next-index: 18/12/2017/13:23:45    
        
```


Finally we can create this function:


```

.change-key-value: function[block [block!] key val][
    
    index: index? find block to-word key

    next-index: index + 1
    block/:next-index: val
    return block
]            
        
```


We call it like this:


```

T-2017.12.08-0001: .change-key-value T-2017.12.08-0001 '.DATETIME-ENTRY 18/12/2017/13:23:45       
transactions: .change-key-value transactions 'T-2017.12.08-0001 T-2017.12.08-0001
        
```



As you can see we had to call .change-key-value twice, first time for updating T-2017.12.08-0001,
second time for transactions.

To simplify the call, we'd like to call .change-key-value only once like this: 


```
      
transactions: .change-key-value transactions 'T-2017.12.08-0001/.DATETIME-ENTRY 18/12/2017/13:23:45            
        
```



The code is rather complex as it is an iterative function. It took me a huge amount of time to debug, I'll spare you the details and will only give you the code:



```

    .change-key-value: function[.block [block!] .key val][

        block: copy .block
        
        key: form .key
        MULTI-KEYS?: (find key "/")
        if MULTI-KEYS? [

            keys: split key "/"

            count: length? keys
            reverse-keys: reverse copy keys      

            forall reverse-keys [

index: index? reverse-keys

LAST-KEY?: (index >= count)
if NOT LAST-KEY? [
    key: reverse-keys/1
    next-key: reverse-keys/2

    either count >= 3 [
        
        either (count - index) >= 2 [
            
            next-next-key: reverse-keys/3
            block: select block to-word next-next-key

        ][
            
            block: copy .block

            
        ]
    ][
        block: copy .block
    ]

    target-block: select block to-word next-key
    target-block: .change-key-value target-block to-word key val
    val: copy target-block

]
            ]

            block: .change-key-value block to-word next-key target-block

            return block 
        ]
        
        index: index? find block to-word key

        next-index: index + 1
        block/:next-index: val

        return block
    ] 

        
```


