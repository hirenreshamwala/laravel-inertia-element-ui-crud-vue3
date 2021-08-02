<template>
    <el-form-item v-if="type == 'textarea'" :label="label" :prop="property">
        <el-input v-model="modelValue" v-on:input="$emit('update:modelValue', modelValue)" type="textarea" :autosize="{ minRows: 6 }" :disabled="disabled"></el-input>
    </el-form-item>

    <el-form-item v-else-if="type == 'checkbox'" :label="label" :prop="property">
        <el-checkbox v-model="modelValue" @change="$emit('update:modelValue', modelValue)" :disabled="disabled">{{ title }}</el-checkbox>
    </el-form-item>

    <el-form-item v-else-if="type == 'radio_group'" :label="label" :prop="property">
        <el-radio-group v-model="modelValue" @change="$emit('update:modelValue', modelValue)" :disabled="disabled">
            <el-radio v-for="(option, index) in options" :key="property+'-'+index" :label="option.value">{{ option.label }}</el-radio>
        </el-radio-group>
    </el-form-item>

    <el-form-item v-else-if="type == 'switch'" :label="label" :prop="property">
        <el-switch active-text="" inactive-text="" v-model="modelValue" v-on:input="$emit('update:modelValue', modelValue)" :disabled="disabled"></el-switch>
    </el-form-item>

    <el-form-item v-else-if="type == 'select'" :label="label" :prop="property">
        <el-select v-model="modelValue" @input="$emit('update:modelValue', modelValue)" @change="$emit('update:modelValue', modelValue)" :disabled="disabled">
            <el-option v-for="(option, index) in options" :key="property+'-'+index" :label="option.label" :value="option.value"></el-option>
        </el-select>
    </el-form-item>

    <el-form-item v-else-if="type === 'password'" :label="label" :prop="property">
        <el-input v-model="modelValue" @input="$emit('update:modelValue', modelValue)" type="password" :disabled="disabled"></el-input>
    </el-form-item>

    <el-form-item v-else :label="label" :prop="property">
        <el-input v-model="modelValue" @input="$emit('update:modelValue', modelValue)" :disabled="disabled"></el-input>
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
        disabled: {
            type: Boolean,
            default: false
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
