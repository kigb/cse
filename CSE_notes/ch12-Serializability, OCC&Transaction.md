# Serializability, OCC&Transaction

## Serializability

![image-20231024160617216](C:\Users\fyx\AppData\Roaming\Typora\typora-user-images\image-20231024160617216.png)

final state serializabliity 只要结果正确即可，但并不serializable T1,T2交替执行，任何一个线性执行是不会有这样的情况。

![image-20231024160753612](C:\Users\fyx\AppData\Roaming\Typora\typora-user-images\image-20231024160753612.png)

conflict 可以证明符合serializability

![image-20231024161841836](C:\Users\fyx\AppData\Roaming\Typora\typora-user-images\image-20231024161841836.png)

构建conflict graph来判断是否一样

怎么构建？如下：

![image-20231024161944622](C:\Users\fyx\AppData\Roaming\Typora\typora-user-images\image-20231024161944622.png)

例子：![image-20231024162004820](C:\Users\fyx\AppData\Roaming\Typora\typora-user-images\image-20231024162004820.png)

无环一定是serializable，但有环不一定不是serializable：

![image-20231024162127936](C:\Users\fyx\AppData\Roaming\Typora\typora-user-images\image-20231024162127936.png)

先是T1->T2，再是T2->T1，再是T1,T2->T3，存在环

但这个结果和线性跑T1 T2 T3结果是一致的。因为最终结果只用考虑T3写的值就够了。初始状态只用考虑T1读的是初始状态。

因此使用conflict graph是一个充分条件，并不是必要条件，因此需要对这个条件进行放松，即view serializability

![image-20231024162518712](C:\Users\fyx\AppData\Roaming\Typora\typora-user-images\image-20231024162518712.png)

总结

![image-20231024162703692](C:\Users\fyx\AppData\Roaming\Typora\typora-user-images\image-20231024162703692.png)

final state没有考虑过程中读的问题。view虽然好，但非常难判断，是np难问题。conflict很好判断，虽然很严格，但其实用的最多的还是conflict，并且conflict可以实现，比如2PL。

目前数据库中的serializability就是指conflict

2PL即conflict serializability：

![image-20231024163053470](C:\Users\fyx\AppData\Roaming\Typora\typora-user-images\image-20231024163053470.png)

即证无环。用反证法，若有环T1,T2,...,Tn,T1 ，然后可以把这些conflict通过锁展开。如T2要拿T1的锁，T3要拿T2的锁... 而T1必须要release，T2才能拿到锁，而T1最后还需要拿Tn的锁，不符合2PL的定义（2PL拿完锁后就不能放锁）因此矛盾

但2PL不一定能全部保证serializability。比如一个list，中间有两个元素，两个元素自己拿锁修改，但list并没有锁，现在依然可以在list后面插入元素。但真实场景往往不会考虑这种麻烦。

## Deadlock

例子：

![image-20231024165751326](C:\Users\fyx\AppData\Roaming\Typora\typora-user-images\image-20231024165751326.png)

![image-20231024165934508](C:\Users\fyx\AppData\Roaming\Typora\typora-user-images\image-20231024165934508.png)

1. 拿锁顺序如果是一样，则不会出现死锁问题。但不一定所有transaction都可以满足这个条件

2. 拿锁的时候可以把等待的环给拿出来，然后abort一个transaction。但开销太大，要去构建一个环
3. 用时间来判断是否死锁，若超时则直接abort。但可能会出现误判。

拿锁是一种悲观的判断，那么也有乐观的控制方法

![image-20231024170622096](C:\Users\fyx\AppData\Roaming\Typora\typora-user-images\image-20231024170622096.png)

先不拿锁，如果满足，则继续，如果不满足则直接abort

![image-20231024170728172](C:\Users\fyx\AppData\Roaming\Typora\typora-user-images\image-20231024170728172.png)

1. 把读写都放入一个缓存集合
2. 验证是否序列化
3. 成功则提交，失败则abort重新来一次

![image-20231024171001603](C:\Users\fyx\AppData\Roaming\Typora\typora-user-images\image-20231024171001603.png)

先初始化read/write set

![image-20231024171049542](C:\Users\fyx\AppData\Roaming\Typora\typora-user-images\image-20231024171049542.png)

直接读，放入读集合。若后面直接接了一个相同的read（可重复读）则直接到set中去寻找，不然可能被改动。

![image-20231024171215158](C:\Users\fyx\AppData\Roaming\Typora\typora-user-images\image-20231024171215158.png)

在write中，同时需要去改变read set。

![image-20231024171407414](C:\Users\fyx\AppData\Roaming\Typora\typora-user-images\image-20231024171407414.png)

![image-20231024171420496](C:\Users\fyx\AppData\Roaming\Typora\typora-user-images\image-20231024171420496.png)

phase2，phase3如上

![image-20231024171439066](C:\Users\fyx\AppData\Roaming\Typora\typora-user-images\image-20231024171439066.png)

验证和写入必须要原子操作

![image-20231024171702131](C:\Users\fyx\AppData\Roaming\Typora\typora-user-images\image-20231024171702131.png)

最简单方式：直接拿全局锁

效果比2PL好，因为phase1可以直接并发执行

同样可以加更加细粒度的锁

![image-20231024171926949](C:\Users\fyx\AppData\Roaming\Typora\typora-user-images\image-20231024171926949.png)

在这里可以很好解决死锁问题。可以直接按照顺序拿锁

![image-20231024171951746](C:\Users\fyx\AppData\Roaming\Typora\typora-user-images\image-20231024171951746.png)

但如果read和write都要拿锁，其实和2PL的效率是一样的。其实可以直接不拿read的锁

![image-20231024172126207](C:\Users\fyx\AppData\Roaming\Typora\typora-user-images\image-20231024172126207.png)

但这个依然有问题。如果d不在这里被拿了锁，而在其他地方被上锁，那么读d时可能不是最新的结果。如下：

![image-20231024172508563](C:\Users\fyx\AppData\Roaming\Typora\typora-user-images\image-20231024172508563.png)

### OCC advantage

![image-20231024172802886](C:\Users\fyx\AppData\Roaming\Typora\typora-user-images\image-20231024172802886.png)

![image-20231024172811582](C:\Users\fyx\AppData\Roaming\Typora\typora-user-images\image-20231024172811582.png)

### OCC problem

OCC中可能会出现误判的情况

首先是误以为出现矛盾的误判，如下

![image-20231024210409567](C:\Users\fyx\AppData\Roaming\Typora\typora-user-images\image-20231024210409567.png)

在这个情境中，两个事务实际上并没有冲突。尽管T1读取了A的值，但它并没有修改A；而T2只修改了A的值，没有读取或修改B的值。

但是，当T2先于T1提交时，OCC可能会认为T1读取的A的值已经过时，因为T2已经修改了A的值。因此，T1可能会被中止，即使它和T2实际上是可以并发执行的，不会导致任何数据不一致。

同时，OCC在执行一个耗时很长的操作时，非常容易出现错误。

## 2PL v.s. OCC

![image-20231024210721342](C:\Users\fyx\AppData\Roaming\Typora\typora-user-images\image-20231024210721342.png)

可以看出，在小数量事务时OCC快于2PL，但随着事务变多，OCC效率不如2PL

## Lock preliminary

Lock是由计算机原子操作实现的，而lock的原子性操作往往比其余正常操作更加费时，不仅本身操作时间长，还需要阻隔其余cpu的管线。因此使用lock会降低性能

![image-20231024205845617](C:\Users\fyx\AppData\Roaming\Typora\typora-user-images\image-20231024205845617.png)













