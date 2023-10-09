# PartialOrd 和 Ord

翻译过来就是部分等和全等的关系。在Rust中，float类型只实现了`PartialOrd`这个trait，没有实现`Ord`这个trait。原因在于，float类型中有一个`NaN`的值不能用来比较，`NaN`和任何数比较都是`NaN`。

`PartialOrd`是`Ord`的子集，这部分内容也可以用离散数学里面的偏序和全序来理解。

[参考资料](https://rustcc.cn/article?id=6b1d9149-b557-45ea-81f2-8bd4fd9c8e6f)


# 异步编程模型
[参考资料](https://course.rs/advance/async/getting-started.html)
## OS线程模型
> Rust语言的多线程模型是 1:1 OS线程的的

OS 线程非常适合少量任务并发，因为线程的创建和上下文切换是非常昂贵的，甚至于空闲的线程都会消耗系统资源。虽说线程池可以有效的降低性能损耗，但是也无法彻底解决问题。

当然，线程模型也有其优点，例如它不会破坏你的代码逻辑和编程模型，你之前的顺序代码，经过少量修改适配后依然可以在新线程中直接运行，同时在某些操作系统中，你还可以改变线程的优先级，这对于实现驱动程序或延迟敏感的应用(例如硬实时系统)很有帮助。

对于长时间运行的 CPU 密集型任务，例如并行计算，使用线程将更有优势。 这种密集任务往往会让所在的线程持续运行，任何不必要的线程切换都会带来性能损耗，因此高并发反而在此时成为了一种多余。同时你所创建的线程数应该等于 CPU 核心数，充分利用 CPU 的并行能力，甚至还可以将线程绑定到 CPU 核心上，进一步减少线程上下文切换。
## async模型
高并发更适合 IO 密集型任务，例如 web 服务器、数据库连接等等网络服务，因为这些任务绝大部分时间都处于等待状态，如果使用多线程，那线程大量时间会处于无所事事的状态，再加上线程上下文切换的高昂代价，让多线程做 IO 密集任务变成了一件非常奢侈的事。而使用async，既可以有效的降低 CPU 和内存的负担，又可以让大量的任务并发的运行，一个任务一旦处于IO或者其他等待(阻塞)状态，就会被立刻切走并执行另一个任务，而这里的任务切换的性能开销要远远低于使用多线程时的线程上下文切换。

事实上, async 底层也是基于线程实现，但是它基于线程封装了一个运行时，可以将多个任务映射到少量线程上，然后将线程切换变成了任务切换，后者仅仅是内存中的访问，因此要高效的多。

不过async也有其缺点，原因是编译器会为async函数生成状态机，然后将整个运行时打包进来，这会造成我们编译出的二进制可执行文件体积显著增大。

## 如何选择异步编程模型
- 有大量 IO 任务需要并发运行时，选 async 模型
- 有部分 IO 任务需要并发运行时，选多线程，如果想要降低线程创建和销毁的开销，可以使用线程池
- 有大量 CPU 密集任务需要并行运行时，例如并行计算，选多线程模型，且让线程数等于或者稍大于 CPU 核心数
- 无所谓时，统一选多线程

# 生命周期
一言以蔽之，被引用的数据的生命周期**必须大于**引用本身的生命周期。

## 函数与生命周期
若一个参数返回了引用类型，那么它的生命周期只可能来源于：

- 传入参数的生命周期
- 函数中某个新建引用的生命周期(极易引起悬垂引用)

## 结构体与生命周期
为结构体中每个引用显式地标注生命周期
```
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().expect("Could not find a '.'");
    let i = ImportantExcerpt {
        part: first_sentence,
    };
}
```
该生命周期标注说明，结构体 `ImportantExcerpt` 所引用的字符串 `str` 必须比该结构体活得更久。

# Rust宏元标签
[参考资料](https://zjp-cn.github.io/tlborm/decl-macros/minutiae/fragment-specifiers.html)
- item: 程序项
- block: 块表达式
- stmt: 语句，注意此选择器不匹配句尾的分号（如果匹配器中提供了分号，会被当做分隔符），但碰到分号是自身的一部分的程序项语句的情况又会匹配。
- pat: 模式
- expr: 表达式
- ty: 类型
- ident: 标识符或关键字
- path: 类型表达式 形式的路径
- tt: token树 (单个 token 或宏匹配定界符 ()、[] 或{} 中的标记)
- meta: 属性，属性中的内容
- lifetime: 生存期token
- vis: 可能为空的可见性限定符
- literal: 匹配 -?字面量表达式

# 闭包三父子
> Fn & FnMut & FnOnce

写闭包的时候，经常会遇到上述三个类型，其实从字面上来理解，就是:
- Fn: 可以重复执行多次，类似于不可变引用
- FnMut: 可以重复执行多次，但是只同时允许同一个闭包执行，类似于可变引用
- FnOnce: 只能执行一次，类似于拿走所有权

官方解释：
- FnOnce约束是call_once(self)，只能调用一次，一旦调用，Closure将丧失所有权
- FnMut是call_mut(&mut self)，能调用多次，每次调用Closure的内部状态会变化
- Fn是call(&self)，能多次调用，每次调用Closure不变

配合上`move`关键字，事情就有趣了。。。
这里先贴一个[参考资料](https://rustcc.cn/article?id=8b6c5e63-c1e0-4110-8ae8-a3ce1d3e03b9)
。可以看看自己是不是有资料中的提到的常见误区。

说说个人理解。我们可以把闭包当作是一个结构体，从外界捕获的变量当作是结构体的字段，若不更改字段，或者字段实现了`copy trait`，都可以认为是Fn；若需要更改某个字段，就说明我们需要创建“可变引用”了，也就是FnMut；最后，如果需要结构体对所有权，那自然就是FnOnce了。需要注意的是，`move`关键字起到了显式移动所有权的作用，不用move的时候也会发生所有权转移，使用move的时候，也不一定发生所有权转移（如i32类型）。

三个闭包的关系是：Fn 继承于 FuMut 继承于 FnOnce。这么看，三父子的称号名副其实。

# Rust并发原语
Rust 中的原子类型位于`std::sync::atomic`中。

这个 module 的文档中对原子类型有如下描述:
> Rust 中的原子类型在线程之间提供原始的共享内存通信，并且是其他并发类型的构建基础。

`std::sync::atomic`提供了以下12种原子类型
```Rust
AtomicBool
AtomicI8
AtomicI16
AtomicI32
AtomicI64
AtomicIsize
AtomicPtr
AtomicU8
AtomicU16
AtomicU32
AtomicU64
AtomicUsize
```
以`AtomicI32`为例，其定义是一个结构体，具有以下方法：
```Rust
// 对原子类型进行加(或减)运算
pub fn fetch_add(&self, val: i32, order: Ordering) -> i32 
// compare and swap
pub fn compare_exchange(&self, current: i32, new: i32, success: Ordering, failure: Ordering) -> Result<i32, i32>
// 从原子类型内部读取值
pub fn load(&self, order: Ordering) -> i32
// 向原子类型内部写入值
pub fn store(&self, val: i32, order: Ordering)
// 交换
pub fn swap(&self, val: i32, order: Ordering) -> i32
```
可以看到每个方法都有一个 `Ordering` 类型的参数，`Ordering` 是一个枚举，表示该操作的内存屏障的强度，用于控制原子操作使用的内存顺序。
> 注：内存顺序是指 CPU 在访问内存时的顺序，该顺序可能受以下因素的影响：
> - 代码中的先后顺序
> - 编译器优化导致在编译阶段发生改变(内存重排序 reordering)
> - 运行阶段因 CPU 的缓存机制导致顺序被打乱

```Rust
pub enum Ordering {
    Relaxed,
    Release,
    Acquire,
    AcqRel,
    SeqCst,
}
```
- **Relaxed**，这是最宽松的规则，它对编译器和 CPU 不做任何限制，可以乱序
- **Release**，释放，设定内存屏障(Memory barrier)，保证它之前的操作永远在它之前，但是它后面的操作可能被重排到它前面（用于写入）
- **Acquire**，获取，设定内存屏障，保证在它之后的访问永远在它之后，但是它之前的操作却有可能被重排到它后面，往往和Release在不同线程中联合使用（用于读取）
- **AcqRel**，是 `Acquire` 和 `Release` 的结合，同时拥有它们俩提供的保证。对于load，它使用的是 Acquire 命令，对于store，它使用的是 Release 命令，希望该操作之前和之后的读取或写入操作不会被重新排序。AcqRel一般用在fetch_add上
- **SeqCst**，顺序一致性，SeqCst就像是AcqRel的加强版，它不管原子操作是属于读取还是写入的操作，只要某个线程有用到SeqCst的原子操作，线程中该SeqCst操作前的数据操作绝对不会被重新排在该SeqCst操作之后，且该SeqCst操作后的数据操作也绝对不会被重新排在SeqCst操作前；它还保证所有线程看到的所有的 SeqCst 操作的顺序是一致的（虽然性能低，但是最保险）

为什么会有内存乱序呢，我们写同步代码不是已经保证了代码的同步吗，答案是否定的，看看Memory Ordering的定义。
> 注：什么是 Memory Ordering, 摘录维基百科中的定义:
>
> Memory Ordering (内存排序) 是指 CPU 访问主存时的顺序。可以是编译器在编译时产生，也可以是 CPU 在运行时产生。反映了内存操作重排序，乱序执行，从而充分利用不同内存的总线带宽。现代处理器大都是乱序执行。因此需要内存屏障以确保多线程的同步。
>
> 关于对Memory Ordering 的理解，有两个线程都要操作 AtomicI32 类型，假设 AtomicI32 类型数据初始值是0，一个线程执行读操作，另一个线程执行写操作要将数据写为10。假设写操作执行完成后，读线程再执行读操作就一定能读到数据10吗? 答案是不确定的，由于不同编译器的实现和CPU的优化策略，可能会出现虽然写线程执行完写操作了，但最新的数据还存在CPU的寄存器中，还没有同步到内存中。为了确保寄存器到内存中的数据同步，就需要Memory Ordering了。 Release 可以理解为将寄存器的值同步到内存，Acquire 是忽略当前寄存器中存的值，而直接去内存中读取最新的值。
> 
>  例如当我们调用原子类型的 store 方法时提供的 Ordering 是 release，在调用原子类型的load 方法时提供的 Ordering 是 Acquire 就可以保证执行读操作的线程一定会读到寄存器里最新的值(**常用操作**)。
