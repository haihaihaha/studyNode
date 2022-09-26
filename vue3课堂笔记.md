时间：2022-9-15
作者：Yang
# 分析工程结构
## mian.js入口文件
> vue2-cli
```
import Vue from 'vue'
import App from './App.vue'

const vm=new Vue({
    render:h=>h(App)
})
vm.$mount('#app')
```
>vue3-cli
```
import {creatApp} from 'vue'
import App from './App.vue'

creatApp(App).mount('#app')
```
vue2-cli的写法在vue3-cli已经完全不能使用了
## App.vue
vue3组件中的模板结构可以没有根标签，其他和vue2一致