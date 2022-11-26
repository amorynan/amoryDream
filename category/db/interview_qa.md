1. why irs: arango db
2. irs tricky: index files sync(balanced : snapshot, GC correct ) 
3. irs tradeoff? mmap or memory ? wny shut down stem?(中文没有词干， stem 需要时间，snowball算法，词)
4. irs analyzer ? Token->Stem(get or not)->norm->N-gram
5. irs format? ti, ts. doc, freq, 
6. irs performance?

problems:
 1. data cold and hot does support dynamic still in here which it should be because it is future partition.   
