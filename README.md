# Laravel InertiaJs Element-UI CURD

Laravel CRUD view library for Element-UI (Vue3)

This package is intended for use with Laravel + InertiaJs + Vue3

## Install

Requires:

```bash
npm install element-plus --save
```

Install:

```bash
npm install laravel-inertia-element-ui-crud-vue3 --save
```

Configuration in app.js

```js
import ElementPlus from "element-plus";
import "element-plus/lib/theme-chalk/index.css";
import locale from "element-plus/lib/locale/lang/en";

app.use(ElementPlus, { locale });
```

sample app.js:

```js
// Laravel + InertiaJs + Vue3 + Webpack
import './bootstrap';

import { createApp, h } from 'vue';
import { createInertiaApp } from '@inertiajs/inertia-vue3';
import { InertiaProgress } from '@inertiajs/progress';
const appName = window.document.getElementsByTagName('title')[0]?.innerText || 'Laravel';
import ElementPlus from 'element-plus'
import 'element-plus/theme-chalk/index.css'
import en from 'element-plus/es/locale/lang/en'
const appEl = document.getElementById('app');
const pageData = DecryptPage(appEl.dataset.page);

// Must add provide('appRoute', route)
createInertiaApp({
    title: (title) => `${title} - ${appName}`,
    resolve: (name) => import(`./Pages/${name}.vue`),
    setup({ el, app, props, plugin }) {
        return createApp({ render: () => h(app, props) })
            .use(plugin)
            .use(ElementPlus, { locale: en })
            .provide('appRoute', route) // Must required for routing
            .mixin({ methods: { route } })
            .mount(el);
    },
}).mount(appEl);

InertiaProgress.init({ color: '#4B5563' });
```

```js
// Laravel + InertiaJs + Vue3 + Vite
import './bootstrap';
import '../css/app.css';
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'
import { createApp, h } from 'vue';
import { createInertiaApp } from '@inertiajs/inertia-vue3';
import { InertiaProgress } from '@inertiajs/progress';
import { resolvePageComponent } from 'laravel-vite-plugin/inertia-helpers';
import { ZiggyVue } from '../../vendor/tightenco/ziggy/dist/vue.m';

const appName = window.document.getElementsByTagName('title')[0]?.innerText || 'Laravel';

// Must add provide('appRoute', route)
createInertiaApp({
    title: (title) => `${title} - ${appName}`,
    resolve: (name) => resolvePageComponent(`./Pages/${name}.vue`, import.meta.glob('./Pages/**/*.vue')),
    setup({ el, app, props, plugin }) {
        return createApp({ render: () => h(app, props) })
            .use(plugin)
            .use(ElementPlus)
            .use(ZiggyVue, Ziggy)
            .provide('appRoute', route) // Must required for routing
            .mount(el);
    },
});

InertiaProgress.init({ color: '#4B5563' });
```

## Usage

### 1. List page (Pages/Users/List.vue)

   Example list page:

   ```html
   <script setup>
    import BreezeAuthenticatedLayout from '@/Layouts/Authenticated.vue';
    import { ElCrudList } from "laravel-inertia-element-ui-crud-vue3";
    import { ElMessage } from 'element-plus'
    import { onMounted, onUpdated } from "vue";
    import { Inertia } from '@inertiajs/inertia'

    const props = defineProps({
        errors: {
            type: Object,
            default: () => {
                return {};
            }
        },
        records: {
            type: Object,
            default: () => {
                return {};
            }
        },
        searched: String,
        success: String,
        error: String,
        flash: {
            type: Object,
            default: () => {
                return {};
            }
        }
    })
    const columns = [
            {
                "name": "id",
                "title": "#Id",
                "sortable": true,
                "width": "80px"
            },
            {
                "name": "name",
                "title": "Name",
                "sortable": true
            },
            {
                "name": "email",
                "title": "Email",
                "sortable": true
            }
    ];

    onMounted(() => {
        handleNotifications();
    });

    onUpdated(() => {
        handleNotifications();
    });

    function handleNotifications(){
        if(props.success){
            ElMessage({
                showClose: true,
                message: props.success,
                type: 'success'
            });
        }
        if(props.error){
            ElMessage({
                showClose: true,
                message: props.error,
                type: 'error'
            });
        }

        if(props.flash && props.flash.message){
            ElMessage({
                showClose: true,
                message: props.flash.message,
            });
        }
    }

    function create(){
        Inertia.get(route('users.create'));
    }
   </script>

   <template>
       <BreezeAuthenticatedLayout>
           <template #header>
               <h2 class="font-semibold text-xl text-gray-800 leading-tight">
                   Users
                   <el-button class="float-right" size="small" @click="create">Add</el-button>
               </h2>
           </template>

           <div class="py-12">
               <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
                   <div class="bg-white overflow-hidden shadow-xl sm:rounded-lg">
                       <el-crud-list :records="records"
                            :isSearch="true"
                            :isEdit="true"
                            :isDelete="true"
                            :sort="sort"
                            :searchText="searched"
                            :indexRoute="'users.index'"
                            :updateRoute="'users.edit'"
                            :deleteRoute="'users.destroy'"
                            :action-width="'80px'"
                            :columns="columns"
                       >
                       </el-crud-list>
                   </div>
               </div>
           </div>
       </BreezeAuthenticatedLayout>
   </template>
   ```

### 2. Create page (Pages/Users/Create.vue)

   Example create page:

   ```html
   <script setup>
    import { Head } from '@inertiajs/inertia-vue3';
    import BreezeAuthenticatedLayout from '@/Layouts/Authenticated.vue';
    import { ElCrudForm } from "laravel-inertia-element-ui-crud-vue3";
    import { onMounted } from "vue";
    import { genFileId, ElMessage } from 'element-plus';
    import { Inertia } from '@inertiajs/inertia';

    const props = defineProps({
        errors: {
            type: Object,
            default: () => {
                return {};
            }
        },
        record: {
            type: Object,
            default: () => {
                return {};
            }
        },
        records: {
            type: Array,
            default: () => {
                return [];
            }
        },
        searched: String,
        success: String,
        error: String,
        flash: {
            type: Object,
            default: () => {
                return {};
            }
        },
    })

    const fields = [
        {
            "property": "name",
            "label": "Name",
            "type": "text",
            "rule": [
                { required: true, message: 'Please input name', trigger: 'blur' },
                { min: 3, max: 5, message: 'Length should be 3 to 5', trigger: 'blur' },
            ],
            "disabled": false //Default false
        },
        {
            "property": "email",
            "label": "Email",
            "type": "text",
            "disabled": true
        },
        {
            "property": "address",
            "label": "Address",
            "type": "textarea",
            "disabled": false
        },
        {
            "property": "status",
            "label": "Status",
            "type": "switch",
            "disabled": false
        }
    ];

    onMounted(() => {
        handleNotifications();
    });

    function handleNotifications(){
        if(props.success){
            ElMessage({
                showClose: true,
                message: props.success,
                type: 'success'
            });
        }
        if(props.error){
            ElMessage({
                showClose: true,
                message: props.error,
                type: 'error'
            });
        }

        if(props.flash && props.flash.message){
            ElMessage({
                showClose: true,
                message: props.flash.message,
            });
        }
    }

    function create(){
        Inertia.get(route('users.create'));
    }
   </script>
   <template>
       <BreezeAuthenticatedLayout>
           <template #header>
               <h2 class="font-semibold text-xl text-gray-800 leading-tight">
                   Users
               </h2>
           </template>

           <div class="py-6">
               <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
                   <div class="bg-white overflow-hidden shadow-xl sm:rounded-lg">
                       <el-crud-form
                           :index-route="'users.index'"
                           :add-route="'users.store'"
                           :update-route="'users.update'"
                           :errors="errors"
                           :record="record"
                           :fields="fields"
                       ></el-crud-form>
                   </div>
               </div>
           </div>
       </BreezeAuthenticatedLayout>
   </template>
   ```

### Column options

- `property`  field name
- `label`  column display name
- `type`  [Supported field types](#supported-form-field-types)
- `disabled` Boolean (default `false`)
- `rule` Array [Doc](https://element-plus.org/en-US/component/form.html#validation)


## Customizing Display of Columns

The display of columns can be over-ridden using Vue3's Template/Slot system. An example is below showing how to over-ride the "id" column in the "List" component.

You can remove `updateRoute` if you don't want edit button in action. Same for delete button, remove `deleteRoute` if you don't require.

For more list options check [document](https://element-plus.org/#/en-US/component/table)

Example:

```html
<el-crud-list :records="records"
    :searchText="searched"
    :isEdit="true"
    :isDelete="true"
    :isSearch="true"
    :indexRoute="'users.index'"
    :updateRoute="'users.edit'"
    :deleteRoute="'users.destroy'"
    :columns="columns"
>
    <!-- Custom column -->
    <template #id="scope">
        ID #{{ scope.row.id }} 
    </template>

    <!-- Custom action button -->
    <template #action="scope">
        <el-button size="small" type="primary" v-on:click="onClickInfo">Info</el-button>
    </template>
</el-crud-list>
```


## Customizing Display of Form element

If you don't want to use your own component, you can override it using slot

Example:

```html
<el-crud-form
    :index-route="'users.index'"
    :add-route="'users.store'"
    :update-route="'users.update'"
    :errors="errors"
    :record="record"
    :fields="fields"
>
    <!-- Custom Form Field -->
    <template #form_name="scope">
        <el-form-item :label="scope.column.label" :error="errors['name']">
            <el-input clearable v-model="scope.entity.name" placeholder="Enter full name"></el-input>
        </el-form-item>
    </template>
</el-crud-form>
```


### Supported form field types

- text
- textarea
- date
- datetime
- password
- checkbox
- radio_group
- switch


## Credits

-   [Hiren Reshamwala](https://github.com/hirenreshamwala)

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.