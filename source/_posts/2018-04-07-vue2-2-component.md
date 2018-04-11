---
title: Vue2 2 component
date: 2018-04-07 20:39:51
tags: [Javascript, Vue]
---

# Simple Component
```html
<!DOCTYPE html>
<html>
<body>
    <div id="root">
        <task></task>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <script src="./order.js"></script>
</body>
</html>
```

```javascript
Vue.component('task', {
    template: '<li>Foobar</li>'
})

new Vue({
    el: '#root',
})
```

# Sub component

```html
<!DOCTYPE html>
<html>
<body>
    <div id="root">
        <task-list title="my todo list"></task-list>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <script src="./order.js"></script>
</body>
</html>
```

```javascript
Vue.component('task-list', {
    props: ['title'],
    template: `
    <div v-show="isVisible">
        <div>{{ title }}</div>
        <task v-for="t in tasks">{{ t.task }}</task>
        <button type="button" @click="hideModel">close</button>
    </div>
    `,

    data() {
        return {
            isVisible: true,
            tasks: [
                { task: 'Go to the store', complete: true },
                { task: 'Go to the email', complete: false },
            ]
        }
    },
    methods: {
        hideModel() {
            this.isVisible = false
        }
    },
})

Vue.component('task', {
    template: '<li><slot></slot></li>',
})

new Vue({
    el: '#root',
})
```