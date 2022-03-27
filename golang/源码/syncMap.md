#### 简介

go基础数据类型map是线程不安全的，在sync包中，有一个线程安全版的map。



#### 实体结构

```
type Map struct {
   // 锁，用于在进行写操作时进行加锁
   mu Mutex

   // 读视图，类型是atomic.Value，但是实际上指向了readOnly这个struct。
   // 数据被删除时，在read视图中是软删除
   read atomic.Value // readOnly

   // dirty contains the portion of the map's contents that require mu to be
   // held. To ensure that the dirty map can be promoted to the read map quickly,
   // it also includes all of the non-expunged entries in the read map.
   //
   // Expunged entries are not stored in the dirty map. An expunged entry in the
   // clean map must be unexpunged and added to the dirty map before a new value
   // can be stored to it.
   //
   // If the dirty map is nil, the next write to the map will initialize it by
   // making a shallow copy of the clean map, omitting stale entries.
   // 脏视图，在进行写入时会写入到这个map中
   dirty map[interface{}]*entry

   // miss次数，用于记录read和dirty的数据匹配相差次数,达到阈值时，将同步read和dirty
   misses int
}

// readOnly is an immutable struct stored atomically in the Map.read field.
type readOnly struct {
   m       map[interface{}]*entry
   amended bool // true if the dirty map contains some key not in m.
}

// expunged is an arbitrary pointer that marks entries which have been deleted
// from the dirty map.
var expunged = unsafe.Pointer(new(interface{}))

// An entry is a slot in the map corresponding to a particular key.
type entry struct {
   // 真正存储的数据，有三种可能
   // p == null ，数据已经被删除了
   // p == expunged 。read视图中仍然有该值，但是dirty中已经没了
   // 正常值，read视图和dirty视图都有，且相等。
   p unsafe.Pointer // *interface{}
}
```





#### 主要方法

```
var smp sync.Map
	// 保存数据
	smp.Store("v1", "hhhh")
	// 读取数据，同map一样，ok表示是否存在该entry
	value, ok := smp.Load("v1")
	if ok {
		fmt.Println(value)
	}
	// 保存或读取该entry，当map中存在该值，则返回值，如果不存在，则放入到map中
	smp.LoadOrStore("v1", "ssss")
	// 删除entry
	smp.Delete("v1")
	// 读取并删除entry
	smp.LoadAndDelete("v1")
	// 批量操作，注意是O(N)操作，直接使用的是for循环进行操作
	// 当range传入的func返回false，停止该循环
	smp.Range(func(key, value interface{}) bool {
		fmt.Println(value)
		return true
})
```



#### 核心思路

读取数据时，从read中读取，如果read视图中存在，则直接返回，否则检查read视图是否缺少了dirty中的entry，如果是的话，则从dirty中读取。

写入数据时，在写入时，依次判断是否在read、在dirty中，如果在两者中，则直接更新，如果都不在，则将read标记amended为true，并插入到dirty中。

```
// Load returns the value stored in the map for a key, or nil if no
// value is present.
// The ok result indicates whether value was found in the map.
func (m *Map) Load(key interface{}) (value interface{}, ok bool) {
	// 先从read视图中读取
	read, _ := m.read.Load().(readOnly)
	e, ok := read.m[key]
	if !ok && read.amended {
		// 如果read视图中不存在，且read视图缺少了一些entry
		m.mu.Lock()
		// Avoid reporting a spurious miss if m.dirty got promoted while we were
		// blocked on m.mu. (If further loads of the same key will not miss, it's
		// not worth copying the dirty map for this key.)
		// 加锁后重新尝试从read中读取，避免在判断过程中，read视图已经更新（当map中的misscount达到阈值时更新）了。
		read, _ = m.read.Load().(readOnly)
		e, ok = read.m[key]
		if !ok && read.amended {
			e, ok = m.dirty[key]
			// Regardless of whether the entry was present, record a miss: this key
			// will take the slow path until the dirty map is promoted to the read
			// map.
			// 更新map中的misscount
			m.missLocked()
		}
		m.mu.Unlock()
	}
	if !ok {
		return nil, false
	}
	return e.load()
}


func (m *Map) missLocked() {
	m.misses++
	// 居然是用dirty的长度来做判断....
	if m.misses < len(m.dirty) {
		return
	}
	// 将dirty的值赋值给read。
	m.read.Store(readOnly{m: m.dirty})
	// 将dirty清空
	m.dirty = nil
	m.misses = 0
}
```