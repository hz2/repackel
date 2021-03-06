## **Table** 表格文档

### 引入
```javascript
import { Table } from "@repackel/repack";
```
```javascript
components: {
  [Table.name]: Table,
},
```
```html
<rl-table :cfg="tableConfig" ref="staffTable" @getSelection="getSelection">
  <!-- <template slot="searchbox">slot</template> -->
</rl-table>
```

**插槽：*

| 名称 | 插槽 | 示例 |
| --- | --- | --- |
| pageBegin | 位于页面开始 | `slot="pageBegin"` |
| beforeTable | `Table` 标签之前 | `slot="beforeTable"` |
| searchBegin | 位于 searchList 开始 | `slot="searchBegin"` |
| searchbox | searchList 之间，由 `type='slot'` 控制 | `slot="searchbox"` |
| customTable | 自定义表格，当 `tableConfig.customTable` 为 `true` 时生效 | `slot="customTable"` |


### 配置

- 1 `tableConfig` 配置
```javascript
{
  pageAlias: { // 表格分页的别名，默认值通常不适用，需要修改为对应的接口请求的字段
    current:"pageIndex", // 当前页的别名
    size:"pageSize", // 分页大小的别名
  },
  baseConfig: {
    size: '', // 搜索框的大小，默认值为 `small`，可选
  },
  tableProps: {
    stripe: true,
  }, // 将 element-ui 的 table attributes 传入组件内，可选
  defaultPageSzie: 10, // 分页默认的 page size 大小，可选
  pageSizes: [10, 50, 100, 150], // 分页组件的 page size 列表，可选
  paginationProps: {}, // 将 element-ui 的 pagination attributes传入组件内，可选
  actionList:[], // 操作按钮列表，可选
  actionAlign: "right", // 操作按钮对齐，默认 `"right"`
  searchList:[], // 搜索列表
  searchFn: this.getList, // 搜索方法 `Function`
  fetchConditionFn: this.fetchConditionList, // 异步获取搜索条件列表的方法 `Function`，可选
  tableSelection: true, // 表格是否可勾选，默认 `false`
  selectable: (row,index)=> index > 1, // 配置可勾选的行
  customTable: true, // 自定义表格，默认 `false`
  tableList:[], // 表格表头，自定义表格时不需要
  tableCellFallbackText: '-', // 表格单元备用文本，可选。当表格的值为空时显示，对整个表格生效。无默认
  hideSearchForm: true, // 隐藏搜索条件和搜索按钮，默认显示
  hidePagination: true, // 隐藏分页组件，默认显示
  searchFormLabelWidth: '', // 搜索列表的 LabelWidth
}
```

- 1-1 `searchFn` 配置

搜索接口通过异步函数传入，最终由组件内部调用，组件外完成数据的拼装。
组件内在 `mounted()` 时调用 搜索，组件外无需再次调用。

示例：
```javascript
getList(p, reset) {
  // `p` 为组件内传入的搜索条件
  // `reset` 为重置搜索开关，当使用到了外部条件时，这里要清除
  // 最后 `resolve` 的包含 `table` 数据和分页
  let { ...pr } = p;
  if (!reset) {
    // 搜索时，带上外部的省市区数据
    const divList = this.divList;
    switch (divList.length) {
      case 1:
        pr.provinceCode = divList[0];
        break;
      case 2:
        pr.cityCode = divList[1];
        break;
      case 3:
        pr.areaCode = divList[2];
        break;
      default:
        break;
    }
  } else {
    // 重置时清空外部的省市区数组
    this.divList = [];
  }
  // 同时可以带上其他必传条件
  pr.buildingType = 1;
  return new Promise((resolve, reject) => {
    this.$req("/community/building/list", {
      method: "GET",
      params: pr
    }).then(res => {
      resolve({
        list: res.rows,
        total: res.total
      });
    });
  });
},
```

- 1-2 `fetchConditionFn` 配置

异步获取搜索条件列表的方法，`return` 一个 `Promise`，最后 `resolve` 一个对象。
每一个键名为 `searchList` 某项的 `key` ，键值为其对应的下拉框选项 `list`。
在 `mounted()` 时调用。

示例：
```javascript
fetchConditionList() {
  return new Promise(async (resolve, reject) => {
    const childType = await this.$req("/getList");
    // const subChildType = await this.$req("/getChildType");
    resolve({
      childType: childType,
      // subChildType: subChildType
    });
  });
},
```

- 1-3 `searchList` 配置

> 格式：
```javascript
searchList: [
  {
    name: "姓名",
    type: "input",
    key: "name"
  },
  {
    name: "性别",
    type: "select",
    key: "sex",
    list: "XB"
  },
  {
    name: "时间范围",
    type: "date",
    key1: "beginDate",
    key2: "endDate"
  }
]
```
> 属性说明：

| 键名 | 值 | 类型 | 默认值 | 示例 |
| --- | --- | --- |--- | --- |
| name | 中文名 | `String` | 必传 | `"姓名"` |
| type | 见下表 | `String` | 必传 | `"select"` |
| key | 表单的键 | `String` | 必传 | `"name"` |
| list | 下拉类型的列表 | `Array` | 必传 | `[{name:'男',val:'1'}]` |
| useLabel | 使用下拉框的 `label` 传值 | `Boolean` |  | `true` |
| key1 | `type="date"` 的开始时间 | `String` |  | `"beginDate"` |
| key2 | `type="date"` 的结束时间 | `String` |  | `"endDate"` |
| hidden | 隐藏该搜索项 | `Boolean` | - | `true` |
| width | item content 宽度，单位为 px | `String` | - | `"220px"` |
| class | item content 类名 | `String` | - | `"w250"` |
| itemClassName | item 类名 | `String` | - | `"block"` |
| - | 以下同 element-ui ||||
| size | - | `String` | `"small"` | `"small"` |
| filterable | - | `Boolean` | - | `true` |
| clearable | - | `Boolean` | `true` | `true` |
| multiple | - | `Boolean` | - | `true` |
| readonly | - | `Boolean` | - | `true` |
| disabled | - | `Boolean` | - | `true` |
| maxlength | - | `Number` | `25` | `20` |
| rows | - | `Boolean` | `"3"` | `"5"` |
| resize | - | `Boolean` | `"none"` | `"none"` |
| placeholder | - | `String` | - | `"请选择"` |
| startPlaceholder | start-placeholder | `String` | - | `"请选择开始时间"` |
| endPlaceholder | end-placeholder | `String` | - | `"请选择结束时间"` |
| valueFormat | value-format | `String` | - | `"yyyy-MM-dd HH:mm:ss"` |

> `type` 类型

| 传值 | 含义 |
| -- | -- |
| `input` | 输入框 |
| `select`| 下拉框 | 
| `date`| 日期范围 |
| `datetime` | 日期时间范围 |
| `date1`| 单个日期 |
| `slot` | 插槽 |

**注：当 `type="slot"` 的时候，可以使用 `name="searchbox"` 的具名插槽，配置更加灵活的内容。*

**下拉框列表数据的必须设为以下格式，键名 `name`, 键值 `val`*

```javascript
list: [{
  val: "1",
  name: "住宅"
}, {
  val: "2",
  name: "公共设施用房"
}, {
  val: "3",
  name: "商业用房"
}]
```

- 1-4 `tableList` 配置
> 格式：
```javascript
tableList: [
  {
    label: "姓名",
    prop: "name",
    width: 80
  },
  {
    label: "性别",
    transform: row => ["","男", "女"][row.sex],
    width: 120
  }
]
```
> 属性说明：

| 键名 | 值 | 类型 | 默认值 | 示例 |
| --- | --- | --- |--- | --- |
| label | 表头 | `String` | 必传 |`"姓名"` |
| prop | 表单循环的 key | `String` | 必传 |`"name"` |
| hidden | 隐藏该列 | `Boolean` | `false` |`true` |
| viewImg | 取 prop 字段显示看图 | `Boolean` | | `true` |
| width | 宽度 ，同 element-ui | `Number` | | `80`  |
| minWidth | 最小宽度 ，同 element-ui | `Number` | | `120`  |
| align | 列表对齐 ，同 element-ui | `String` |  | `'right'`  |
| overflow | showOverflowTooltip ，同 element-ui | `Boolean` |  | `true` |
| fixed | 列表固定 ，同 element-ui | `Boolean` `String` | `false` | `'right'`  |
| transform | 处理显示文字的函数，传入`row` ，显示返回结果 | `Function` |  | `row => row.num + 'kg'` | 
| class | 类名 | `String` `Function` | | `row => ["", "green", "red"][row.state]`  |
| style | 内联样式 | `String` `Function` | | `row => ({ color: ["", "green", "red"][row.state]})`  |
| fn | 点击事件，传入 `(row,index)` | `Function` | | `(row,index) => this.alert(row,index)`  |
| buttonList | 操作按钮的列表 | `Array` | | 见下表  |
| type | 控制其他类型 | `String` | | 目前支持 `switch`  |
| bindAttr | 传入其他的属性 | `Object` | | 目前支持 `type='switch'` |
| change | switch 的 change 事件 | `Function` | | `(newVal,row,index)=> {}` |


- 1-5-1 `buttonList` 配置：
> 格式：
```javascript
buttonList:[
   {
     text: "编辑",
     fn: () => this.$info("编辑")
   }
 ]
```
> 属性说明：

| 键名 | 值 | 类型 | 默认值 | 示例 |
| --- | --- | --- |--- | --- |
| icon | 按钮图标 | `String` | | `"el-icon-edit"`  |
| type | 按钮类型，同 element-ui | `String` | `'text'` | `"primary"`  |
| size | 按钮大小，同 element-ui | `String` | `'default'` | `"mini"`  |
| disabled | 禁用 | `Boolean` `Function` | | `true`  |
| class | 类名 | `String` `Function` | | `row => ["", "green", "red"][row.state]`  |
| style | 内联样式 | `String` `Function` | | `row => ({ color: ["", "green", "red"][row.state]})` |
| text | 按钮文字 | `String` `Function` | | `row => ["", "查看", "编辑"][row.state]`  |
| fn | 点击事件，传入 `(row,index)` | `Function` | | `(row,index) => this.alert(row,index)`  |
| hidden | 隐藏按钮 | `Boolean` `Function` | `false` | `(row,index) => index > 5` |

### 方法

用法：

```javascript
this.$refs[ ref名 ].[ 方法名 ]
// this.$refs.staffTable.getList()
```
> 方法列表：

| 方法名 | 说明 | 
| --- | --- |
| `searchButton()` | 搜索按钮 |
| `getList()` | 搜索函数 |
| `resetQuery()` | 重置函数 |
| `handleSizeChange()` | 分页大小改变 |
| `handleCurrentChange()` | 分页索引改变 |
| `clearSelection` | 同 element-ui |
| `toggleRowSelection` | 同 element-ui |
| `toggleAllSelection` | 同 element-ui |
| `setCurrentRow` | 同 element-ui |
| `clearSort` | 同 element-ui |
| `clearFilter` | 同 element-ui |

### 事件

用法：

```javascript
@getSelection="myGetSelectionMethod"
// 即为 element 的 `selection-change` 多选方法

@getCurrent="myGetCurrentMethod"
// 即为 element 的 `current-change` 单选方法
// 需要设置  tableProps: { highlightCurrentRow: true }
```

### 样式

类名：

```css
/* 完整的页面 */
.repack-page {}

/* 操作栏和搜索的表单 */
.repack-action-search {}

/* 操作栏 */
.repack-action-bar {}

/* 搜索的表单 */
.repack-search-form {}

/* 搜索的按钮 */
.repack-search-btns {}

/* 自定义的表格 */
.repack-custom-table {}

/* 分页 */
.repack-pagination {}

/* 表格 */
.repack-table {}

/* 表格看图 */
.repack-table-view-image {}
```