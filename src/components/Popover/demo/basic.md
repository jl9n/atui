---
order: 0
title:
  zh-CN: 基本
  en-US: Type
---

## zh-CN



## en-US


````jsx
<popover
  effect="fade"
  placement="bottom"
  title="我是标题"
  content="我是内容，我是内容，我是内容">
  <v-button>Popover on bottom</v-button>
</popover>
````

````vue-script
new Vue({
  el: 'body',
  components: {
    popover: atui.Popover,
    vButton: atui.Button
  }
})
````
