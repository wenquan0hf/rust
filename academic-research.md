# 相关学术研究 #

如下是部分对 Rust 有影响的学术论文。  通过阅读它们可以更好的理解 Rust 的背景，甚至迸发出更精彩的想法。

## 类型系统    
 
- 
- [Region based memory management in Cyclone](http://209.68.42.137/ucsd-pages/Courses/cse227.w03/handouts/cyclone-regions.pdf)    
- [Safe manual memory management in Cyclone](http://www.cs.umd.edu/projects/PL/cyclone/scp.pdf)   
- [Typeclasses: making ad-hoc polymorphism less ad hoc
- ](http://www.ps.uni-saarland.de/courses/typen-ws99/class.ps.gz)   
- [Macros that work together](https://www.cs.utah.edu/plt/publications/jfp12-draft-fcdf.pdf)   
- [Traits: composable units of behavior](http://scg.unibe.ch/archive/papers/Scha03aTraits.pdf)    
- [Alias burying](http://www.cs.uwm.edu/faculty/boyland/papers/unique-preprint.ps) 我们试过类似的并且会将之摒弃。   
- [External uniqueness is unique enough](http://www.computingscience.nl/research/techreps/repo/CS-2002/2002-048.pdf)   
- [Uniqueness and Reference Immutability for Safe Parallelism](https://research.microsoft.com/pubs/170528/msr-tr-2012-79.pdf)   
- [Region Based Memory Management](https://research.microsoft.com/pubs/170528/msr-tr-2012-79.pdf)   


## 并发性   
- [Singularity: rethinking the software stack](http://research.microsoft.com/pubs/69431/osr2007_rethinkingsoftwarestack.pdf)   
- [Language support for fast and reliable message passing in singularity OS](http://research.microsoft.com/pubs/67482/singsharp.pdf)   
- [Scheduling multithreaded computations by work stealing](http://supertech.csail.mit.edu/papers/steal.pdf)   
- [Thread scheduling for multiprogramming multiprocessors](http://www.eecis.udel.edu/~cavazos/cisc879-spring2008/papers/arora98thread.pdf)   
- [The data locality of work stealing](http://www.aladdin.cs.cmu.edu/papers/pdfs/y2000/locality_spaa00.pdf)   
- [Dynamic circular work stealing deque](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.170.1097&rep=rep1&type=pdf) 循环双向队列   
- [Work-first and help-first scheduling policies for async-finish task parallelism](http://www.cs.rice.edu/~yguo/pubs/PID824943.pdf) 比非严格 work-steal 策略更通用。   
- [A Java fork/join calamity](http://www.coopsoft.com/ar/CalamityArticle.html) 针对 Java 中的 fork/join 库的批判, 尤其是 JAVA 中的 work-steal 策略到非严格计算的应用。 
- [Scheduling techniques for concurrent systems](http://www.ece.rutgers.edu/~parashar/Classes/ece572-papers/05/ps-ousterhout.pdf)   
- [Contention aware scheduling](http://www.blagodurov.net/files/a8-blagodurov.pdf)   
- [Balanced work stealing for time-sharing multicores](http://www.cse.ohio-state.edu/hpcs/WWW/HTML/publications/papers/TR-12-1.pdf)   
- [Three layer cake](http://www.upcrc.illinois.edu/workshops/paraplop10/papers/paraplop10_submission_8.pdf)   
- [Non-blocking steal-half work queues](http://www.cs.bgu.ac.il/~hendlerd/papers/p280-hendler.pdf)   
- [Reagents: expressing and composing fine-grained concurrency](http://www.mpi-sws.org/~turon/reagents.pdf)   
- [Algorithms for scalable synchronization of shared-memory multiprocessors](https://www.cs.rochester.edu/u/scott/papers/1991_TOCS_synch.pdf)   

##其他

[Crash-only software ](https://www.usenix.org/legacy/events/hotos03/tech/full_papers/candea/candea.pdf)     
[Composing High-Performance Memory Allocators](http://people.cs.umass.edu/~emery/pubs/berger-pldi2001.pdf)   
[Reconsidering Custom Memory Allocation](http://people.cs.umass.edu/~emery/pubs/berger-oopsla2002.pdf)    
   

##关于 Rust 的论文

[GPU programming in Rust](http://www.cs.indiana.edu/~eholk/papers/hips2013.pdf)   
[Parallel closures: a new twist on an old idea](http://www.cs.indiana.edu/~eholk/papers/hips2013.pdf) - 提到的不完全是rust, 但是是由 nmatsakis 提出的。

