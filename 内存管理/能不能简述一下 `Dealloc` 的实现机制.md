## 能不能简述一下 `Dealloc` 的实现机制

`Dealloc` 的实现机制是内容管理部分的重点，把这个知识点弄明白，对于全方位的理解内存管理的只是很有必要。

#### 1.`Dealloc` 调用流程


* 1.首先调用 `_objc_rootDealloc()`
* 2.接下来调用 `rootDealloc()`
* 3.这时候会判断是否可以被释放，判断的依据主要有5个，判断是否有以上五种情况
  * `NONPointer_ISA`
  * `weakly_reference`
  * `has_assoc`
  * `has_cxx_dtor`
  * `has_sidetable_rc`
* 4-1.如果有以上五中任意一种，将会调用 `object_dispose()`方法，做下一步的处理。
* 4-2.如果没有之前五种情况的任意一种，则可以执行释放操作，C函数的 `free()`。
* 5.执行完毕。
  

#### 2.`object_dispose()` 调用流程。

- 1.直接调用 `objc_destructInstance()`。
- 2.之后调用 C函数的 `free()`。

#### 3.`objc_destructInstance()` 调用流程

- 1.先判断 `hasCxxDtor`，如果有 `C++` 的相关内容，要调用 `object_cxxDestruct()` ，销毁 `C++` 相关的内容。
- 2.再判断 `hasAssocitatedObjects`，如果有的话，要调用 `object_remove_associations()`，销毁关联对象的一系列操作。
- 3.然后调用 `clearDeallocating()`。
- 4.执行完毕。 


#### 4.`clearDeallocating()` 调用流程。
- 1.先执行 `sideTable_clearDellocating()`。
- 2.再执行 `weak_clear_no_lock`,在这一步骤中，会将指向该对象的弱引用指针置为 `nil`。
- 3.接下来执行 `table.refcnts.eraser()`，从引用计数表中擦除该对象的引用计数。
- 4.至此为止，`Dealloc` 的执行流程结束。


