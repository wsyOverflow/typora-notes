#### 1. emplace_back vs push_back

- 相同点
  二者都是在尾部加一个元素，

- 不同点
  - 参数上
    emplace_back 支持可变参数，可以直接传对象的构造函数的参数列表添加一个新元素，
    如 `vec.emplace_back(arg1, arg2)`
    push_back 不支持可变参数，当对象的构造函数只有一个参数时，可以使用隐式类型转换添加一个新元素。如果对象的构造函数多余一个参数，只能使用初始化参数列表传递参数，
    一个参数的构造函数情况： `vec.push_back(arg)` 
    两个参数的构造函数情况：`vec.push_back({arg1, arg2})`。错误用法`vec.push_back(arg1, arg2)`
  - 内存上 emplace_back 性能要优于 push_back
    当使用上述参数列表添加一个元素时，push_back 比 emplace_back 多一次拷贝/移动构造操作
    push_back 会先创建一个临时对象，再使用移动构造函数放到 vector 尾部元素的内存区域。
    emplace_back 使用的是就地构造，即直接在 vector 尾部元素的内存区域构造。