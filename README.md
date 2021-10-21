### 为什么props接收的值在created中打印不出来

在项目中,我们会经常使用到props传值,经常是由父组件传递给子组件一个值,然后我们在子组件中想去使用这个props接收的值,但是在created和mounted中打印,却发现是初始值,并不是我们在父组件中发送异步请求拿到的那个值.但是通过setTimeout设置了一定的时间之后,我们又能拿到对应的值.这是什么情况呢?

其中原因就是这样:因为父组件中的要就要传递的`props`属性 是通过发生`ajax`请求回来的, 请求的这个过程是需要时间的，但是**子组件的渲染要快于ajax请求过程**，所以此时 `created 、mounted`这样的只会执行一次的生命周期钩子，已经执行了，但是`props`还没有流进来（子组件），所以只能拿到默认值。

接下来,我们来跟着一个案例来解决这个问题吧.

案例的效果:在子组件中有一个previewUrl数据是预览的路径,他的路径是根据props传进来的值去赋值的,但是在created中直接赋值的话,previewUrl是为空.

解决:1.使用setTimeout(不推荐)。 缺点：原因是因为你不知道网络传输的具体时间有多久,直接写死一个值的话,容易有bug

```js
<template>
  <div class="son">
    我是子组件
    <div>姓名:{{transData.name}}</div>
    <div>性别:{{transData.sex}}</div>
    <div>出年年纪:{{transData.birthDay}}</div>
    <div>图片路径:{{previewUrl}}</div>
  </div>
</template>

<script>
export default {
  props:{
    transData:{
      type:Object
    }
  },
  data(){
    return{
      previewUrl:''
    }
  },
  created(){
    this.previewUrl = this.transData.image
    // 直接打印打印不出来
    console.log(this.transData);
    setTimeout(()=>{
      // 能打印出来
      console.log(this.transData);
       this.previewUrl = this.transData.image
    },300)
  }
}
</script>

<style>

</style>
```

解决2.使用computed.缺点:只能用一个单独的值来,如果我这个previewUrl是在一个对象里面的话,就没办法了.

比如我是要赋值给imgSet对象中的 previewUrl。imgSet:{previewUrl: ''}这种,使用computed就不能很好的去赋值了。

```js
<template>
  <div class="son">
    我是子组件
    <div>姓名:{{transData.name}}</div>
    <div>性别:{{transData.sex}}</div>
    <div>出年年纪:{{transData.birthDay}}</div>
    <div>图片路径:{{previewUrl}}</div>
  </div>
</template>

<script>
export default {
  props:{
    transData:{
      type:Object
    }
  },
  data(){
    return{
    }
  },
  computed:{
    previewUrl(){
      return this.transData.image
    }
  }
  // created(){
  //   this.previewUrl = this.transData.image
  //   // 直接打印打印不出来
  //   console.log(this.transData);
  //   setTimeout(()=>{
  //     // 能打印出来
  //     console.log(this.transData);
  //      this.previewUrl = this.transData.image
  //   },300)
  // }
}
</script>

<style>

</style>
```

解决3:使用watch。优点：即使是赋值给对象中的某个属性也可以很好的去赋值。

当然，有时候会监听不到，此时你可以去开启深度监听，就能监听的到了。此处就不多做展开了。

```js
<template>
  <div class="son">
    我是子组件
    <div>姓名:{{transData.name}}</div>
    <div>性别:{{transData.sex}}</div>
    <div>出年年纪:{{transData.birthDay}}</div>
    <div>图片路径:{{previewUrl}}</div>
  </div>
</template>

<script>
export default {
  props:{
    transData:{
      type:Object
    }
  },
  data(){
    return{
      previewUrl:''
    }
  },
  watch:{
    'transData.image'(val){
      this.previewUrl = val
    }
  }
  // computed:{
  //   previewUrl(){
  //     return this.transData.image
  //   }
  // }
  // created(){
  //   this.previewUrl = this.transData.image
  //   // 直接打印打印不出来
  //   console.log(this.transData);
  //   setTimeout(()=>{
  //     // 能打印出来
  //     console.log(this.transData);
  //      this.previewUrl = this.transData.image
  //   },300)
  // }
}
</script>

<style>

</style>
```

解决4：使用v-if来判断,这样子在子组件中的created中就可以将有值的transData打印出来了。

缺点：如果业务复杂，传的值很多的话，那么我们每一个数据都需要在子组件标签上v-if，显得很乱

```js
<son v-if="JSON.stringify(transData) !== '{}'" :transData="transData"></son>
```

参考地址：https://www.cnblogs.com/wmt-kilig/p/14015880.html

demo地址：https://github.com/rui-rui-an/propsIssue
