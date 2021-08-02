<template>
    <div class="py-6 px-4">
        <el-form v-if="entity" ref="form" :model="entity" label-width="140px">
            <div v-for="column in fields" :key="column">
                <slot name="before"></slot>
                <slot :name="'form_'+column.property" :entity="entity" :column="column" :error="errors[column.property]">
                    <field
                        :property="column.property"
                        :label="column.label"
                        :title="column.title"
                        :options="column.options"
                        :type="column.type"
                        :disabled="!!column.disabled"
                        :error="errors[column.property]"
                        v-model="entity[column.property]"
                    ></field>
                </slot>
                <slot name="after"></slot>
            </div>
            <el-form-item>
                <el-button type="primary" @click="onSubmit">{{ submitText }}</el-button>
                <el-button @click="cancel()">Cancel</el-button>
            </el-form-item>
        </el-form>
    </div>
</template>

<script>
import Field from "./Field";
export default {
    components: {
        Field
    },
    data() {
        return {
            entity: null
        }
    },
    props: {
        primaryKey: { type: String, default: 'id' },
        indexRoute: { type: String, required: true },
        addRoute: { type: String, required: true },
        updateRoute: { type: String, required: true },
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
    },
    computed: {
        submitText(){
            if(this.entity[this.primaryKey]){
                return 'Update';
            }
            return 'Create';
        },
    },
    mounted() {
        if (this.record){
            this.entity = this.record;
        }
    },
    methods:{
        onSubmit(){
            if(this.entity[this.primaryKey]){
                //Update
                this.$inertia.put(route(this.updateRoute, this.entity[this.primaryKey]), this.entity)
            } else {
                // Create
                this.$inertia.post(route(this.addRoute), this.entity)
            }
        },
        cancel(){
            this.$inertia.get(route(this.indexRoute))
        }
    }
}
</script>
