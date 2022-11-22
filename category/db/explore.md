### Explore

1. how data store: Liner (array, linkedList)or Non-Liner (tree, skipList, Map...) 
2. how data explore for common sense : (binary-search...)
3. Non-Liner data store structure: (extra pointer to access other data, make explore balance in data distribution) 
    SkipList :
   1. key points: 
      1. every node has itself next pointer array to access next node ;
      2. how many next pointers dose every node has is random, which is also called level  
   2. operations: 
      1. build:
      
      make a sentinel always maintain a pointer to first listNode in every level linkedList 
      
      every node choose itself will exist in cur level
      ![img.png](../imgs/skiplist.png)
      END with level has only two listNodes 
      ![img.png](../imgs/sl.png)
      
      2. search : from high level to search , if current level nodes split this whole data balanced , which time cost less
      3. insert : search first with recording first node in suitable range space from high level to 0, 
                  if 0 level does not exist this data, just insert, but if exist , insertion will choose to u,   
                  and then IMPORT STUFF is make a random for this data max level, max level is the main factor to make ur skipList balanced,
                  if I want log(n) , every level may has (1/2)^n nodes， every node could has (1/2) chance to choose itself will exist in this level, 
                  but If I want log3(n), every level may has (1/3)^n nodes, every node could has (1/3) chance to choose itself will exist in this level.     
                  then from low level to high level 
                  to insert node with linking first node and first node's next node  
      4. delete 


