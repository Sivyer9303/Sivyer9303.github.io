### 底层分析

#### 核心结构

##### hmap

map 的底层结构为hmap

```
// A header for a Go map.
type hmap struct {
	// Note: the format of the hmap is also encoded in cmd/compile/internal/gc/reflect.go.
	// Make sure this stays in sync with the compiler's definition.
	count     int // 已经存储的键值对数量，在len函数中，直接使用这个值
	flags     uint8
	B         uint8  // 桶的幂数，go中的map使用的哈希寻址方法是位运算，即通过hash&(m-1) (m为桶的个数)来决定键值对放入哪个桶中，因此桶的数量需要是2的幂。这里的B就等于幂次。
	noverflow uint16 // 溢出桶的数量
	hash0     uint32 // hash seed

	buckets    unsafe.Pointer // 常规桶的起始地址
	oldbuckets unsafe.Pointer // 扩容时旧的桶的地址，只有在扩容时才为非空
	nevacuate  uintptr        // 渐进式扩容时，记录下一个要被迁移的旧桶编号

	extra *mapextra // 额外的桶信息，在申请map内存时，为了避免频繁的扩容，申请了一部分额外的内存空间作为溢出桶，在溢出桶未被使用时会存放在extra中，一旦某个桶无法存放8个(固定)键值对时，会从extra中申请溢出桶。
}
```

##### bucket

真正存储数据的结构，每个桶可以存放8个键值对，在存放时，hash、key、value三者分开存放，三者都是连续的。

![map桶](map桶.png)

##### 溢出桶

在hmap结构的最后一个字段extra中，只想了mapextra结构体。溢出桶本质上和正常的桶是一致的，在map申请内存时，就会直接申请溢出桶。溢出桶的数量等于2^(B-4)，B等于hmap中的B，即正常桶的2次幂。

同时，常规桶和溢出桶在内存中是连续的。

在常规桶中，有一个overflow字段，用于指向当前常规桶中使用到的溢出桶。同样的，溢出桶也可以继续溢出。

因此，go中解决hash冲突的办法是拉链法。

```

type mapextra struct {
    overflow    *[]*bmap //把已经用到的溢出桶链起来
    oldoverflow *[]*bmap //渐进式扩容时，保存旧桶用到的溢出桶
    nextOverflow *bmap   //下一个尚未使用的溢出桶
}
```

#### 扩容

##### 扩容方式

1.翻倍扩容

分配一批新的桶，桶的数量是旧桶数量的两倍，hamp.oldbuckets指向旧桶，hmap.buckets指向新桶。

然后通过nevacuate标记当前迁移的位置，每次读写时，都会迁移一部分桶。

2.等量扩容

如果没有设置负载因子，就会触发等量扩容。会创建和旧桶数量一样的新桶，然后把原来的键值对迁移到新桶中。

### 与java版的相同点与不同点

