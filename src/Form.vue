<script setup>
import AuthenticatedLayout from '@/Layouts/AuthenticatedLayout.vue';
import { ElCrudList } from "laravel-inertia-element-ui-crud-vue3";
import { ElMessage } from 'element-plus'
import { inject, onMounted, onUpdated } from "vue";
import { Inertia } from '@inertiajs/inertia'
const route = inject('appRoute');
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
        type: Object,
        default: () => {
            return {};
        }
    },
    filters: {
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
    },
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
    },
    {
        "name": "phone",
        "title": "Phone",
        "sortable": true
    },
    {
        "name": "city",
        "title": "City",
        "sortable": true
    },
    {
        "name": "state",
        "title": "State",
        "sortable": true
    },
    {
        "name": "pincode",
        "title": "Pincode",
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

    if(props.flash && props.flash.success){
        ElMessage({
            showClose: true,
            message: props.flash.success,
            type: 'success'
        });
    }

    if(props.flash && props.flash.error){
        ElMessage({
            showClose: true,
            message: props.flash.error,
            type: 'error'
        });
    }
}
function create(){
    Inertia.get(route('customers.create'));
}

</script>
<template>
    <AuthenticatedLayout title="Customers">
        <template #header>
            <h2 class="font-semibold text-xl text-gray-800 leading-tight">
                Customers
                <el-button class="float-right" size="small" @click="create">Add</el-button>
            </h2>
        </template>

        <div class="py-12">
            <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
                <div class="bg-white overflow-hidden shadow-xl sm:rounded-lg p-4">
                    <el-crud-list :records="records"
                          :filters="filters"
                          :indexRoute="'customers.index'"
                          :updateRoute="'customers.edit'"
                          :deleteRoute="'customers.destroy'"
                          :columns="columns"
                    >
                        <template #filter_email="{entity}">
                            <el-form-item label="Custom Email">
                                <el-input v-model="entity.email" placeholder="Email"></el-input>
                            </el-form-item>
                        </template>
                    </el-crud-list>
                </div>
            </div>
        </div>
    </AuthenticatedLayout>
</template>
