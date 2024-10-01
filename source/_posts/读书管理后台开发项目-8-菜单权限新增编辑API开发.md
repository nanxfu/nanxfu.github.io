---
title: 读书管理后台开发项目-8.菜单权限新增编辑API开发
index_img: /img/Vue.js
date: 2024-09-29 14:38:26
tags:
  - Vue.js
  - web
  - Typescript
  - Vite
  - Nest.js
---

# 新增菜单功能前端开发

开发此处页面：
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20240929144155.png)
阅读代码发现页面由 formSchema 变量动态生成,于是将其改造为如下数据

```typescript
 export const formSchema: FormSchema[] = [{
    field: 'path',
    label: '菜单路径',
    component: 'Input',
    required: true,
},
    {
        field: 'menuName',
        label: '菜单名称',
        component: 'Input',
        required: true,
    },
    {
        field: 'redirect',
        label: '重定向',
        component: 'Input',
        required: false,
    },
    {
        field: 'meta',
        label: '元数据',
        component: 'Input',
        required: false,
    },
    {
        field: 'parentMenu',
        label: '上级菜单',
        component: 'TreeSelect',
        componentProps: {
            fieldNames: {
                label: 'name',
                value: 'id',
            },
            getPopupContainer: () => document.body,
        },
    },
    {
        field: 'status',
        label: '状态',
        component: 'RadioButtonGroup',
        defaultValue: '1',
        componentProps: {
            options: [
                {label: '启用', value: '1'},
                {label: '禁用', value: '0'},
            ],
        },
    }, {
        field: 'show',
        label: '是否显示',
        component: 'RadioButtonGroup',
        defaultValue: '0',
        componentProps: {
            options: [
                {label: '是', value: '0'},
                {label: '否', value: '1'},
            ],
        },
        ifShow: ({values}) => !isButton(values.type),
    },]
```
点击新增菜单填入数据并点击确认后即可在控制台看见新增菜单的数据
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20240929145835.png)
翻阅代码可在handleSubmit处找到调用新增菜单的Api。
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20240929150250.png)
接着在后端进行开发新建菜单接口
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20240929171413.png)
在前端handleSubmit处给新增的菜单添加pid
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20240930185432.png)
然后配置前端请求接口
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20240930190734.png)
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20240930190827.png)

## update menu开发
编写nest.js后端接口
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20240930192300.png)
测试接口：
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20240930192321.png)

配置好前端表单字段：
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20240930195459.png)
在打开抽屉时做判断决定
执行更新菜单还是新增菜单
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20240930195551.png)

## 修复禁用启用状态显示异常
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20240930195752.png)
注意字段值的类型：
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20240930195829.png)
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20240930195904.png)
除此之外还有一个BUG：可以直接禁用一级菜单。如果直接禁用顶级菜单则会导致页面出错所以需要优化禁用逻辑

## 优化菜单禁用逻辑
只有在所有子菜单都被禁用，父菜单才能禁用
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20240930202757.png)
```typescript
        if(checkAllChildrenMenuDisabled(values)){
          await updateMenu({ data: values });
        }else {
          createMessage.error('请禁用所有子菜单后，再禁用主菜单')
        }
```

## 修复编辑菜单时父菜单丢失
再menu/index.vue下的handleEdit中将pid与parentMenu做映射即可
```typescript
  function handleEdit(record: Recordable) {
    record.parentMenu = record.pid
    openDrawer(true, {
      record,
      isUpdate: true,
    });
  }
```