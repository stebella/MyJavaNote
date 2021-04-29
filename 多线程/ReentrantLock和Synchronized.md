**ReentrantLock**

ReentrantLock是一个可重入的互斥锁，又被称为“独占锁”。

- ReentrantLock是JDK方法，需要手动声明上锁和释放锁，因此语法相对复杂些；如果忘记释放锁容易导致*死锁*
- ReentrantLock具有更好的细粒度，可以在ReentrantLock里面设置内部Condititon类，可以实现分组唤醒需要唤醒的线程
- RenentrantLock能实现公平锁

**Synchronized**

- Synchoronized语法上简洁方便
- Synchoronized是JVM方法，由编辑器保证枷锁和释放