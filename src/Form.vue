<template>
    <div class="py-6 px-4">
        <el-form v-if="entity" ref="form" :model="entity" :label-width="labelWidth" :label-position="labelPosition">
            <div v-for="column in fields" :key="column">
                <slot name="before" :entity="entity" :column="column" :error="errors[column.property]"></slot>
                <slot :name="'form_'+column.property" :entity="entity" :column="column" :error="errors[column.property]">
                    <Field
                        :property="column.property"
                        :label="column.label"
                        :title="column.title"
                        :placeholder="column.placeholder"
                        :options="column.options"
                        :maxlength="column.maxlength"
                        :minlength="column.minlength"
                        :filterable="column.filterable"
                        :clearable="column.clearable"
                        :show-word-limit="column.showWordLimit"
                        :type="column.type"
                        :disabled="!!column.disabled"
                        :error="errors[column.property]"
                        v-model="entity[column.property]"
                    ></Field>
                </slot>
                <slot name="after" :entity="entity" :column="column" :error="errors[column.property]"></slot>
            </div>
            <slot name="action" :entity="entity">
                <el-form-item>
                    <slot name="button_before" :entity="entity"></slot>
                    <el-button type="primary" @click="onSubmit">{{ submitTextComp }}</el-button>
                    <el-button v-if="!hideCancel" @click="cancel()">Cancel</el-button>
                    <slot name="button_after" :entity="entity"></slot>
                </el-form-item>
            </slot>
        </el-form>
    </div>
</template>

<script setup>
import { inject, ref, onMounted, computed } from 'vue';
const route = inject('appRoute');
import {Inertia} from "@inertiajs/inertia";
import Field from "./Field";
const props = defineProps({
    primaryKey: { type: String, default: 'id' },
    labelWidth: { type: String, default: '140px' },
    labelPosition: { type: String, default: 'left' },
    indexRoute: { type: String, required: false },
    addRoute: { type: String, required: false },
    updateRoute: { type: String, required: false },
    submitText: { type: String, required: false },
    hideCancel: { type: Boolean, required: false, default: false },
    record: {
        type: Object,
        default: () => {
            return {}
        }
    },
    fields: {
        type: Array,
        default: () => {
            return []
        }
    },
    errors: {
        type: Object,
        default: () => {
            return {}
        }
    }
});
const entity = ref(null);

onMounted(() => {
    if (props.record){
        entity.value = props.record;
    }
});
const submitTextComp = computed(() => {
    if (props.submitText) return props.submitText;

    if(entity.value[props.primaryKey]){
        return 'Update';
    }
    return 'Create';
});
const onSubmit = () => {
    if(entity.value[props.primaryKey]){
        //Update
        if(!props.updateRoute){
            console.error('Update route does not provided');return;
        }
        Inertia.put(route(props.updateRoute, entity.value[props.primaryKey]), entity.value)
    } else {
        // Create
        if(!props.addRoute){
            console.error('Add route does not provided');return;
        }
        Inertia.post(route(props.addRoute), entity.value)
    }
}

const cancel = () => {
    if(!props.indexRoute){
        console.error('Index route does not provided');return;
    }
    Inertia.get(route(props.indexRoute))
}
</script>
