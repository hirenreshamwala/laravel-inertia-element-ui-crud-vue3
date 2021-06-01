<template>
    <el-form-item v-if="type == 'textarea'" :label="label" :prop="property">
        <el-input v-model="modelValue" v-on:input="$emit('update:modelValue', modelValue)" type="textarea" :autosize="{ minRows: 6 }"></el-input>
    </el-form-item>

    <el-form-item v-else-if="type == 'checkbox'" :label="label" :prop="property">
        <el-checkbox v-model="modelValue" @change="$emit('update:modelValue', modelValue)">{{ title }}</el-checkbox>
    </el-form-item>

    <el-form-item v-else-if="type == 'switch'" :label="label" :prop="property">
        <el-switch active-text="" inactive-text="" v-model="modelValue" v-on:input="$emit('update:modelValue', modelValue)"></el-switch>
    </el-form-item>

    <el-form-item v-else-if="type == 'select'" :label="label" :prop="property">
        <el-select v-model="modelValue" @input="$emit('update:modelValue', modelValue)" @change="$emit('update:modelValue', modelValue)">
            <el-option v-for="(option, index) in options" :key="property+'-'+index" :label="option.label" :value="option.value"></el-option>
        </el-select>
    </el-form-item>

    <el-form-item v-else-if="type === 'password'" :label="label" :prop="property">
        <el-input v-model="modelValue" @input="$emit('update:modelValue', modelValue)" type="password"></el-input>
    </el-form-item>

    <el-form-item v-else :label="label" :prop="property">
        <el-input v-model="modelValue" @input="$emit('update:modelValue', modelValue)"></el-input>
    </el-form-item>
</template>
<script>
export default {
    props: {
        type: {
            type: String,
            default: ''
        },
        title: {
            type: String,
            default: ''
        },
        modelValue: {
            required: true
        },
        property: {
            type: String,
            default: ''
        },
        label: {
            type: String,
            default: ''
        },
        options: {
            type: Array,
            default: () => {
                return []
            }
        }
    }
}
</script>
