一个链表就是一串节点(Node). 每个Node有两个责任：
1. 持有一个value
2. 持有下一个Node的引用。nil表示链表最后一个Node

先写一个工具方法方便打印 
```
func example(of desc: String,block: (()->Void)){
  print("---Example of \(desc)---")
  block()
}
```

创建一个基本的`Node`类, 并重写`description`
```
public class Node<Value> {
  public var value: Value
  public var next: Node?
  
  public init(value: Value,next: Node? = nil) {
    self.value = value
    self.next = next
  }
}
extension Node: CustomStringConvertible {
  public var description: String {
    guard let next = next else {
      return "\(value)"
    }
    return "\(value) -> " + String(describing: next)
  }
}
```
尝试创建几个Node并将他们链接起来。
```
example(of: "creating and linking nodes") {
  let node1 = Node(value: 1)
  let node2 = Node(value: 2)
  let node3 = Node(value: 3)
  
  node1.next = node2
  node2.next = node3
  
  print(node1)
}
```
控制台输出
```
---Example of creating and linking nodes---
1 -> 2 -> 3
```

这个离我们想要的还差很远，如果list比较长，创建起来将会是灾难性的。所以我们应该用一个`LinkedList`将`Node`管理起来。

创建`LinkList`

```
public struct LinkedList<Value> {
  
  public var head: Node<Value>?
  public var tail: Node<Value>?
  public init(){}
  
  public var isEmpty: Bool {
    return head == nil
  }
}

extension LinkedList: CustomStringConvertible {

  public var description: String {
    guard let head = head else {
      return "Empty list"
    }
    return String(describing: head)
  }
}
```

一个`LinkedList` 只需要知道头和尾，就能确定整个链表。
如何给链表添加数据呢，三种方式：

- `push:` 从head处添加
- `append:` 从tail处添加
- `insert(after:)` 指定位置添加

#### `push` 操作

```
extension LinkedList {
  public mutating func push(_ value: Value) {
    head = Node(value: value, next: head)
    if tail == nil {
      tail = head
    }
  }
}
```
从头部添加很简单，只需要重新设置`head`并将之前的`head`设置为`next`，考虑到链表元素只有一个的情况。`tail`即使`head`。

```
example(of: "push") {
  var list = LinkedList<Int>()
  list.push(3)
  list.push(2)
  list.push(1)
  
  print(list)
}

// result: 
---Example of push---
1 -> 2 -> 3
```

#### `append` 操作

```
extension LinkedList {
  
  public mutating func append(_ value: Value){
    guard !isEmpty else {
      push(value)
      return
    }
    tail?.next = Node(value: value)
    tail = tail?.next
  }
}
```

首先考虑链表为空的情况，如果不为空则直接将`Node`赋值给当前的`tail?.next`，而此`Node`也作为新的`tail`。

```
example(of: "append") {
  var list = LinkedList<Int>()
  list.append(1)
  list.append(2)
  list.append(3)
  
  print(list)
}
// result: 
---Example of append---
1 -> 2 -> 3
```
#### `insert(after:)` 操作

再新增一个便利方法`node(at:)` 
```
extension LinkedList {
  
  // 只有空list或者index超过边界才会返回nil
  public func node(at index: Int) -> Node<Value>? {
    var currentNode = head
    var currentIndex = 0
    while currentNode != nil && currentIndex < index {
      currentNode = currentNode?.next
      currentIndex += 1
    }
    return currentNode
  }
  
  @discardableResult
  public mutating func insert(_ value: Value, after node: Node<Value>) -> Node<Value> {
    guard tail !== node else {
      append(value)
      return tail!
    }
    node.next = Node(value: value, next: node.next)
    return node.next!
  }
}
```


`insert(after:)`的思想也很简单，如果传进来的`Node`就是`tail`，则相当于执行一个`append`。剩下就是把传进来的`Node`的next设置为value， value的next设置为`node.next`。

```
example(of: "inserting at a particular index") {
  var list = LinkedList<Int>()
  list.push(3)
  list.push(2)
  list.push(1)
  
  print("Before inserting: \(list)")
  var middleNode = list.node(at: 1)!
  for _ in 1...4 {
    middleNode = list.insert(-1, after: middleNode)
  }
  print("After inserting: \(list)")
}
// result: 
---Example of inserting at a particular index---
Before inserting: 1 -> 2 -> 3
After inserting: 1 -> 2 -> -1 -> -1 -> -1 -> -1 -> 3
```
#### 方法性能分析 

|  方法   | 时间复杂度  |
|  ----  | ----  |
| push  | O(1) |
| append  | O(1) |
| insert(after:)  | O(1) |
| node(at:)  | O(i)  i表示给定的index |

性能看起来都不错。

### 从链表中删除数据

也是三个方法

- `pop` 从表头删除
- `removeLast` 从结尾删除
- `remove(after:)` 任意位置删除

#### pop 操作 

跟`push`一样 非常简单。
```
extension LinkedList {
  
  public mutating func pop() -> Value?{
    defer {
      head = head?.next
      if isEmpty {
        tail = nil
      }
    }
    return head?.value
  }
}
```

如果是只有一个元素或者空元素的列表`head?.next`就为`nil`。这时候需要设置`tail`为`nil`。

```
example(of: "pop") {
  var list = LinkedList<Int>()
  list.push(3)
  list.push(2)
  list.push(1)
  
  print("Before popping list: \(list)")
  let poppedValue = list.pop()
  print("After popping list: \(list)")
  print("Popped value: " + String(describing: poppedValue))
}
// result: 
---Example of pop---
Before popping list: 1 -> 2 -> 3
After popping list: 2 -> 3
Popped value: Optional(1)
```
#### removeLast 操作

这个方法会有点麻烦，我们虽然保存了`tail`但是并不知道`tail`的上一个元素。所以这里只能用遍历的方法
```
extension LinkedList {
  
  public mutating func removeLast() -> Value? {
    guard let head = head else {
      return nil
    }
    guard head.next != nil else {
      // 只有一个元素的情况
      return pop()
    }
    var prev = head
    var current = head
    // 循环取next直到没有next
    while let next = current.next {
      prev = current
      current = next
    }
    prev.next = nil
    tail = prev
    return current.value
  }
}
```

注释已经很清楚了。
```
example(of: "removing the last node") {
  var list = LinkedList<Int>()
  list.push(3)
  list.push(2)
  list.push(1)

  print("Before removing last node: \(list)")
  let removedValue = list.removeLast()

  print("After removing last node: \(list)")
  print("Removed value: " + String(describing: removedValue))
}
// result: 
---Example of removing the last node---
Before removing last node: 1 -> 2 -> 3
After removing last node: 1 -> 2
Removed value: Optional(3)
```

`removeLast`需要把整个list遍历一遍，所以这里的时间复杂度为 `O(n)`

#### `remove(after:)` 操作

这个跟`insert(after:)`差不多，都是比较简单

```
extension LinkedList {
  
  @discardableResult
  public mutating func remove(after node: Node<Value>) -> Value? {
    defer {
      if node.next === tail {
        tail = node
      }
      node.next = node.next?.next
    }
    return node.next?.value
  }
}
```

如果移除的`Node`的下一个是`tail`，直接将`tail`设置为此`node`。并将`node`的下一个设置为下下个。

```
example(of: "removing a node after a particular node") {
  var list = LinkedList<Int>()
  list.push(3)
  list.push(2)
  list.push(1)
  
  print("Before removing at particular index: \(list)")
  let index = 1
  let node = list.node(at: index - 1)!
  let removedValue = list.remove(after: node)
  
  print("After removing at index \(index): \(list)")
  print("Removed value: " + String(describing: removedValue))
}

// result: 
---Example of removing a node after a particular node---
Before removing at particular index: 1 -> 2 -> 3
After removing at index 1: 1 -> 3
Removed value: Optional(2)
```

#### 方法性能分析 

|  方法   | 时间复杂度  |
|  ----  | ----  |
| pop  | O(1) |
| removeLast  | O(n) |
| remove(after:)  | O(1) |

### 更*swifty*的方式

`Collection` protocol提供了很多便利的方式去访问和操作列表。这里也可以让自己的`LinkedList`实现`Collection` 协议。

因为链表操作的是`Node`，所以我们的`Index`需要自定义下。

```

extension LinkedList: Collection {
  
  public struct Index: Comparable {
    
    public var node: Node<Value>?
    
    static public func == (lhs: Index, rhs: Index) -> Bool{
      switch (lhs.node,rhs.node) {
      case let (left?,right?):
        return left.next === right.next
      case (nil,nil):
        return true
      default:
        return false
      }
    }
    
    static public func < (lhs: Index,rhs: Index) -> Bool {
      guard lhs != rhs else {
        return false
      }
      let nodes = sequence(first: lhs.node) { $0?.next }
      return nodes.contains(where: { $0 === rhs.node })
    }
  }
  
  public var startIndex: Index {
    return Index(node: head)
  }
  
  public var endIndex: Index {
    return Index(node: tail?.next)
  }
  
  public func index(after i: Index) -> Index {
    return Index(node: i.node?.next)
  }
  
  public subscript(position: Index) -> Value {
    return position.node!.value
  }
}
```

这里`Index`持有`Node`并实现 `Comparable`协议 。 再实现`Collection` 的几个必要的方法。就可以使用了。

```
example(of: "using collection") {
  var list = LinkedList<Int>()
  for i in 0...0{
    list.append(i)
  }
  print("List: \(list)")
  print("First element: \(list[list.startIndex])")
  print("Array containing first 3 elements: \(Array(list.prefix(3)))")
  print("Array containing last 3 elements: \(Array(list.suffix(3)))")

  let sum = list.reduce(0, +)
  print("Sum of all values: \(sum)")
}

// result: 
---Example of using collection---
List: 0
First element: 0
Array containing first 3 elements: [0]
Array containing last 3 elements: [0]
Sum of all values: 0
```

Swift中的Collection为值类型，使用copy-on-write机制实现的。

例子： 
```
example(of: "array cow") {
  let array1 = [1, 2]
  var array2 = array1
  
  print("array1: \(array1)")
  print("array2: \(array2)")
  
  print("---After adding 3 to array 2---")
  array2.append(3)
  print("array1: \(array1)")
  print("array2: \(array2)")
}

// result: 
---Example of array cow---
array1: [1, 2]
array2: [1, 2]
---After adding 3 to array 2---
array1: [1, 2]
array2: [1, 2, 3]
```

这里可以看到`array2`执行`append`方法，列表改变了，但是`array1`并没有改变。这是值类型的特质。`array2` 本质是在 `append` 方法调用的时候才真正`copy`。在`append`方法执行前，`array1`和`array2`都指向同一个地址。

再来看下`LinkedList`

```
example(of: "linked list cow") {
  var list1 = LinkedList<Int>()
  list1.append(1)
  list1.append(2)
  var list2 = list1
  print("List1: \(list1)")
  print("List2: \(list2)")
  
  print("After appending 3 to list2")
  list2.append(3)
  print("List1: \(list1)")
  print("List2: \(list2)")
}

// result: 
---Example of linked list cow---
List1: 1 -> 2
List2: 1 -> 2
After appending 3 to list2
List1: 1 -> 2 -> 3
List2: 1 -> 2 -> 3
```
我们自己写的链表并没有值类型的特质，是因为我们的`Node`是使用class（引用类型）实现的。

一个方法就是在我们每次操作新增或者删除某个`Node`之前，将所有元素更新一遍
```
extension LinkedList {
  
  private mutating func copyNodes(){
    guard var oldNode = head else {
      return
    }
    head = Node(value: oldNode.value)
    var newNode = head
    while let nextOldNode = oldNode.next {
      newNode?.next = Node(value: nextOldNode.value)
      newNode = newNode?.next
      oldNode = nextOldNode
    }
    tail = newNode
  }
  
}
```
在所有包含`mutating`的方法前调用`copyNodes` 。但是这样相当于在所有方法时间复杂度前提下增加了一个`O(n)`。明显是不可接受的。

所以这里还可以对此进行优化。

- `isKnownUniquelyReferenced` 

`isKnownUniquelyReferenced`方法可以判断是否只有被一个对象引用。没有共享，这样我们只需要在两个对象同时引用的情况下才需要使用值类型的特点。 
在`copyNodes`最上面加上 
```
guard !isKnownUniquelyReferenced(&head) else {
    return
}
```
这样大部分情况下就不会复制。

- 对方法进行评估

`push`方法并不会造成两者不一样。
```
example(of: "linked list cow") {
  var list1 = LinkedList<Int>()
  list1.append(1)
  list1.append(2)
  var list2 = list1
  print("List1: \(list1)")
  print("List2: \(list2)")
  
  print("After appending 3 to list2")
  list2.push(0)
  print("List1: \(list1)")
  print("List2: \(list2)")
}
// result: 
---Example of linked list cow---
List1: 1 -> 2
List2: 1 -> 2
After appending 3 to list2
List1: 1 -> 2
List2: 0 -> 1 -> 2
```

所以在`push`中并不需要调用`copyNodes`


---
参考自《Data Structure & Algorithms in Swift》
