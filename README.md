## 修改element-ui的源码

```
node_modules\element-ui\lib\tree.js    1051行修改源码  去掉 'includeHalfChecked &&'
// if ((child.checked || includeHalfChecked && child.indeterminate) && (!leafOnly || leafOnly && child.isLeaf)) {
if ((child.checked || child.indeterminate) && (!leafOnly || leafOnly && child.isLeaf)) {
```
![20210423195709](https://devrhl.oss-cn-beijing.aliyuncs.com/note1/cb29567575bcfd60ebb72ce18a546a4d.png)

- [day15](20210109.md)
- [day16](20210109.md)
- [day01](20210109.md)
- [day01](20210109.md)
- [day01](20210109.md)
- [day01](20210109.md)
- [day01](20210109.md)
- [day01](20210109.md)