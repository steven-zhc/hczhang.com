---
title: Vue2-4 template
date: 2018-04-10 17:53:59
tags: [Javascript, Vue]
---

# Template slot

```html
<!DOCTYPE html>
<html>
<body>
    <div id="root">
        <task>
            <template slot="title">hello</template>
            this will go to nameless slot tag
        </task>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <script src="./order.js"></script>
</body>
</html>
```

```javascript
Vue.component('task', {
    template: `
    <li>
        <slot name="title"></slot>
        <slot>Default content</slot>
    </li>`
})

new Vue({
    el: '#root',
})
```

> Note: template tag will be deleted when rendered in html page
```html
    <div id="root">
        <task><li>hello</li></task>
    </div>
```
> Note: other tag will be kept.

# Inline template
```html
```html
<!DOCTYPE html>
<html>
<body>
    <div id="root" class="container">
        <progress-view inline-template>
            <h1>Your progress is {{ completionRate }}</h1>
        </progress-view>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <script src="./order.js"></script>
</body>
</html>
```