---
title: 【Angularjs文档翻译及实例】DOM事件
tags: [javascript,angularjs]
date: 2015/04/20
---

把Angularjs当中涉及DOM事件的属性整理一下，此文档是基于1.4.8英文文档整理的。

> 约定：
> 此文中ngXxx表示ng-xxx属性名。

### ngBlur

**用法**

```
<window, input, select, textarea, a
  ng-blur="expression">
...
</window, input, select, textarea, a>
```

**参数**

| 参数     | 类型         | 详情                              |      |
| :----- | :--------- | :------------------------------ | ---- |
| ngBlur | expression | 表达式将在失去焦点时被触发（事件对象与$event一样可获得） |      |

### ngFocus

**用法**

```
<window, input, select, textarea, a
  ng-focus="expression">
...
</window, input, select, textarea, a>
```

**参数**

| 参数      | 类型         | 详情                            |      |
| :------ | :--------- | :---------------------------- | ---- |
| ngFocus | expression | 表达在获得焦点时被触发（事件对象与$event一样可获得） |      |

### ngChange

**用法**

```
<input
  ng-change="expression">
...
</input>
```

**参数**

| 参数       | 类型         | 详情                 |      |
| :------- | :--------- | :----------------- | ---- |
| ngChange | expression | 表达式将在input控件被改变时触发 |      |

### ngChecked

**用法**

```
<INPUT
  ng-checked="expression">
...
</INPUT>
```

**参数**

| 参数        | 类型         | 详情                          |      |
| :-------- | :--------- | :-------------------------- | ---- |
| ngChecked | expression | 如果表达式为真，那么就会在元素上设置checked属性 |      |

### ngClick

**用法**

```
<ANY
  ng-click="expression">
...
</ANY>
```

**参数**

| 参数      | 类型         | 详情                             |      |
| :------ | :--------- | :----------------------------- | ---- |
| ngClick | expression | 当被点击的时候触发表达式（事件对象与$event一样可获得） |      |




### ngKeydown/ngKeypress/ngKeyup

**用法**

```
<ANY
  ng-keydown="expression">
...
</ANY>
<ANY
  ng-keypress="expression">
...
</ANY>
<ANY
  ng-keyup="expression">
...
</ANY>
```

**参数**

| 参数         | 类型         | 详情                                       |      |
| :--------- | :--------- | :--------------------------------------- | ---- |
| ngKeydown  | expression | 表达式在按键按下时被触发（事件对象与$event一样可获得，并且可以查询keyCode, altKey等） |      |
| ngKeypress | expression | 表达式在按键按下时被触发（事件对象与$event一样可获得，并且可以查询keyCode, altKey等） |      |
| ngKeyup    | expression | 表达式在按键释放时被触发（事件对象与$event一样可获得，并且可以查询keyCode, altKey等） |      |

### ngMousedown/ngMouseup

**用法**

```
<ANY
  ng-mousedown="expression">
...
</ANY>
<ANY
  ng-mouseup="expression">
...
</ANY>
```

**参数**

| 参数          | 类型         | 详情                             |      |
| :---------- | :--------- | :----------------------------- | ---- |
| ngMousedown | expression | 表达式在鼠标按下时被触发（事件对象与$event一样可获得） |      |
| ngMouseup   | expression | 表达式在鼠标释放时被触发（事件对象与$event一样可获得） |      |

### ngMouseenter/ngMousemove

**用法**

```
<ANY
  ng-mouseenter="expression">
...
</ANY>
<ANY
  ng-mousemove="expression">
...
</ANY>
```

**参数**

| 参数           | 类型         | 详情                                 |      |
| :----------- | :--------- | :--------------------------------- | ---- |
| ngMouseenter | expression | 表达式在鼠标进入元素时被触发（事件对象与$event一样可获得）   |      |
| ngMousemove  | expression | 表达式在鼠标在元素上移动时被触发（事件对象与$event一样可获得） |      |

### ngMouseover/ngMouseleave

**用法**

```
<ANY
  ng-mouseover="expression">
...
</ANY>
<ANY
  ng-mouseleave="expression">
...
</ANY>
```

**参数**

| 参数           | 类型         | 详情                               |      |
| :----------- | :--------- | :------------------------------- | ---- |
| ngMouseover  | expression | 表达式在鼠标穿过元素时被触发（事件对象与$event一样可获得） |      |
| ngMouseleave | expression | 表达式在鼠标离开时被触发（事件对象与$event一样可获得）   |      |

### ngSelected

**用法**

```
<OPTION
  ng-selected="expression">
...
</OPTION>
```

**参数**

| 参数         | 类型         | 详情                         |      |
| :--------- | :--------- | :------------------------- | ---- |
| ngSelected | expression | 表达式为真时，selected属性将被设置在元素上。 |      |

### ngSubmit

**用法**

```
<form
  ng-submit="expression">
...
</form>
```

**参数**

| 参数       | 类型         | 详情                             |      |
| :------- | :--------- | :----------------------------- | ---- |
| ngSubmit | expression | 提交表单时，表达式被触发（事件对象与$event一样可获得） |      |


### 综合实例

**html代码**

```
<div ng-app="myapp">
    <form name="form" ng-controller="formCtrl">
        <p>
            <input type="text" ng-model="blur" ng-blur="blur='be blured'" ng-focus="blur='got focus,click other place'" placeholder="click me" />
        </p>
        <p>
            <span ng-model="mouseEvent" ng-mouseenter="mouseEvent='mouseenter'" ng-mouseleave="mouseEvent='mouseleave'" ng-bind="mouseEvent" ng-init="mouseEvent='touch me'">touch me</span>
        </p>
        <p>
            <input type="text" ng-model="change" ng-change="changeEvent()" placeholder="change me" />
        </p>
        <p>
            msg1:{{changeStatus}},content:{{change}}
        </p>
        <p>
            <input type="button" ng-click="btnClickChangeCheckboxEvent()" value="change status of checkbox->" />
            <input type="checkbox" ng-model="checkbox" ng-checked="isStatus" />
        </p>
        <p>
            <input type="checkbox" ng-model="option" />change status of select
            <select>
                <option>A</option>
                <option ng-selected="option">B</option>
            </select>
        </p>
    </form>
</div>
```

**js代码**

```
angular.module('myapp', [])
    .controller('formCtrl', function($scope) {
        $scope.change = 'change me';
        $scope.changeStatus = 'no change';
        $scope.isStatus = false;
        $scope.option = false;

        $scope.changeEvent = function() {
            $scope.changeStatus = 'be changed';
        }

        $scope.btnClickChangeCheckboxEvent = function() {
            $scope.isStatus = !$scope.isStatus;
        }
    });
```

DEMO地址：[https://jsfiddle.net/Lionney/vLkoz9d3/](https://jsfiddle.net/Lionney/vLkoz9d3/)

如有问题，请指正。
