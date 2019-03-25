# iview的Modal组件点击确定后禁止直接关闭
项目中使用iview作为UI组件时，难免会遇到使用Modal对话框组件验证表单元素，点击`确定`后对话框直接关闭的问题，很是让人头大。
在iview的github仓库的issue里提到一种解决办法：`使用Modal的loading属性控制对话框的关闭状态`。
```html
<template>
    <div>
        <Button type="primary" @click="modal1 = true">blala</Button>
        <Modal v-model="modal" title="测试" @on-ok="ok"  :loading="loading">
            <p>点击确定不会自动关闭对话框！</p>
        </Modal>
    </div>
</template>
<script>
    export default {
        data () {
            return {
                modal: false,
                loading: true
            }
        },
        methods: {
            ok () {
                setTimeout(() => {
                    this.loading = false;
                    this.$nextTick(() => {
                        this.loading = true;
                    });
                }, 300);
            }
        }
    }
</script>
```
其实，如果时间允许的话，完全可以用`div`自己实现一个对话框作为公共组件在项目中使用！