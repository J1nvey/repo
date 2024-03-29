## 父子组件通信

### - props / $emit / $on / .sync


子组件使用props来接收父组件绑定的数据;  
子组件使用$emit来通知父组件更新数据;
```vue
    // 父组件
    <template>
        <div>
            <son :msg='msg' @update:msg='msg = $event'></son>
            <!-- <son :msg='msg' @msg='msg = $event'></son> -->
            <!-- <son :msg='msg' @msg='updateMsg'></son> -->
            <!-- <son :msg.sync='msg'></son> -->
        </div>
    </template>

    <script>
    import son from './son'
    export default {
        components: {
            son
        },
        data () {
            return {
                msg: 'from father to son',
            }
        },
        methods: {
            updateMsg (msg) {
                this.msg = msg;
            }
        }
    }
    </script>
```
```vue
    // 子组件
    <template>
        <div>
            {{ msg }}
        </div>
    </template>

    <script>
    export default {
        props: {
            msg: {
                type: String,
                default: 'no msg'
            }
        },
        mounted () {
            setTimeout(() => {
                this.$emit('update:msg', 'from son to father');
                // this.$emit('msg', 'from son to father');
            }, 3000)
        }
    }
    </script>
```
其中如果子组件不使用'update'来更新，则父组件也不使用'update'。  
可以使用.sync修饰符来双向绑定props，但是子组件需要使用'update'来更新。  
为了语义化，建议无论是否使用.sync修饰符都应该改使用'update'来更新props。
### - $parent / $children
```js
    // 在子组件中
    this.$parent.msg = '$parent';
    // 在父组件中
    this.$children[0].msg = '$children';
```
### - ref
```js
    // 能实现 但是页面会警告
    this.$refs.son.msg =='$ref'
```
### - Vue.observable
::: tip 官方文档  
让一个对象可响应。Vue 内部会用它来处理 data 函数返回的对象。

返回的对象可以直接用于渲染函数和计算属性内，并且会在发生改变时触发相应的更新。也可以作为最小化的跨组件状态存储器，用于简单的场景。
:::
```js
    // store.js
    const state = Vue.observable({
        age: 18,
        name: 'J1nvey'
    });
    const mutations = {
        setAge (age) {
            state.age = age;
        },
        setName (name) {
            state.name = name;
        }
    }
    export { state, mutations }
```

## 兄弟组件通信

### - 公共父组件传递
其中一个子组件可以用$emit来调用父组件的方法去更新props，使得另一个子组件的props更新。
### - eventBus
全局定义eventBus
```js
    // main.js 全局
    Vue.prototype.$bus = new Vue();
    // 触发
    this.$bus.$emit('updateMsg', 'eventBus');    
    // 接收
    this.$bus.$on('updateMsa', msg => {
        this.msg = msg;
    });
    // 取消
    beforeDestroy () {
        this.$bus.$off('updateMsa');
    }
    // $off
    // 如果没有提供参数，则移除所有的事件监听器；
    // 如果只提供了事件，则移除该事件所有的监听器；
    // 如果同时提供了事件与回调，则只移除这个回调的监听器。
```
### - vuex
待补充
### - sessionStorage / localStorage
localStorage 和 sessionStorage 属性允许在浏览器中存储 key/value 对的数据。  
sessionStorage 用于临时保存同一窗口(或标签页)的数据，在关闭窗口或标签页之后将会删除这些数据。  
localStorage 用于长久保存整个网站的数据，保存的数据没有过期时间，直到手动去删除。
```js
    // 保存数据
    localStorage.setItem("key", "value");
    sessionStorage.setItem("key", "value");
    // 读取数据
    const lastname = localStorage.getItem("key");
    const lastname = sessionStorage.getItem("key");
    // 删除数据
    localStorage.removeItem("key");
    sessionStorage.removeItem("key");
    // 清空数据
    localStorage.clear();
    sessionStorage.clear();
```

## 隔代组件通信

### - provide / inject

```js
    // 父组件
    provide () {
        return {
            msg: 'provide'
        }
    }
    // 后代组件
    inject: ['msg'],
    // 设置默认值和来源
    // inject: {
    //     msg: {
    //         from: 'father',
    //         default: 'inject',
    //     }
    // }
```
### 提示
但是官网中有说到:  
provide 和 inject 绑定并不是可响应的。这是刻意为之的。  
然而，如果你传入了一个可监听的对象，那么其对象的属性还是可响应的。 

```js
    // 父组件
    data () {
        return {
            obj: {
                msg: 'from father to son'
            }
        }
    },
    provide () {
        return {
            father: this.obj
        }
    },
    // 后代组件
    inject: {
        father: {
            default: 'provide'
        }
    }
    // 在后代组件中修改father是可以响应的
    this.father.msg = 'change in son component';
    // 父组件可以$watch('obj.msg', cb)回调来响应
```


### - $attr / $listeners (inheritAttrs: false)  
$attrs 能传给后代组件中props没绑定的;  
$listeners 能传给后代组件非原生事件;(通过$emit来更新数据)
```vue
    // 父组件
    <template>
        <son
            :name='name'
            :age='age'
            :sex='sex'
            @click.native='click'
            @updateName='updateName'
        ></son>
    </template>

    <script>
        import son from './son'
        export default {
        components: {
            son
        },
        data () {
            return {
                name: 'J1nvey',
                age: 18,
                sex: '男'
            },
        },
        methods: {
            click () {
                // $listeners不包含原生事件(.native)
            },
            updateName () {
                this.$emit('updateName', 'new Name');
            }
        }
</script>
```

```vue
    // 子组件
    <template>
        <div v-on='$listeners'>
            {{ name }}
        </div>
    </template>

    <script>
    export default {
        props: ['name'],
        inheritAttrs: false,// 关闭挂载在组件根元素上属性(没在props中声明的会挂载在组件)
        data () {
            return {
                
            }
        },
        mounted () {
            console.log(this.$attrs)
            // {age: 18, sex: "男"}
            // 此时这个$attrs可以继续传到下一个组件中
        }
    }
    </script>

```

