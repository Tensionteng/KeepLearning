# PartialOrd 和 Ord
翻译过来就是部分等和全等的关系。在Rust中，float类型只实现了`PartialOrd`这个trait，没有实现`Ord`这个trait。原因在于，float类型中有一个`NaN`的值不能用来比较，`NaN`和任何数比较都是`NaN`。

`PartialOrd`是`Ord`的子集，这部分内容也可以用离散数学里面的偏序和全序来理解。