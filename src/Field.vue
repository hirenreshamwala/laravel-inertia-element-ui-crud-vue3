<template>
    <el-form-item v-if="type == 'textarea'" :label="label" :prop="property" :error="error">
        <el-input
            v-model="modelValue"
            v-on:input="$emit('update:modelValue', modelValue)"
            type="textarea"
            :placeholder="placeholder"
            :autosize="{ minRows: 6 }"
            :disabled="disabled"
            :clearable="clearable"
            :maxlength="maxlength"
            :minlength="minlength"
            :show-word-limit="showWordLimit"
        ></el-input>
    </el-form-item>

    <el-form-item v-else-if="type == 'date'" :label="label" :prop="property" :error="error">
        <el-date-picker
            v-model="modelValue"
            @change="$emit('update:modelValue', modelValue)"
            type="date"
            :placeholder="placeholder || 'Select date'"
        />
    </el-form-item>

    <el-form-item v-else-if="type === 'datetime'" :label="label" :prop="property" :error="error">
        <el-date-picker
            v-model="modelValue"
            @blur="$emit('update:modelValue', modelValue)"
            type="datetime"
            :disabled="disabled"
        ></el-date-picker>
    </el-form-item>

    <el-form-item v-else-if="type == 'checkbox'" :label="label" :prop="property" :error="error">
        <el-checkbox v-model="modelValue" @change="$emit('update:modelValue', modelValue)" :disabled="disabled">{{ title }}</el-checkbox>
    </el-form-item>

    <el-form-item v-else-if="type == 'radio_group'" :label="label" :prop="property" :error="error">
        <el-radio-group v-model="modelValue" @change="$emit('update:modelValue', modelValue)" :disabled="disabled">
            <el-radio v-for="(option, index) in options" :key="property+'-'+index" :label="option.value">{{ option.label }}</el-radio>
        </el-radio-group>
    </el-form-item>

    <el-form-item v-else-if="type == 'switch'" :label="label" :prop="property" :error="error">
        <el-switch active-text="" inactive-text="" v-model="modelValue" v-on:input="$emit('update:modelValue', modelValue)" :disabled="disabled"></el-switch>
    </el-form-item>

    <el-form-item v-else-if="type == 'select'" :label="label" :prop="property" :error="error">
        <el-select v-model="modelValue" :placeholder="placeholder" @input="$emit('update:modelValue', modelValue)" @change="$emit('update:modelValue', modelValue)" :disabled="disabled" :clearable="clearable" :filterable="filterable">
            <el-option v-for="(option, index) in options" :key="property+'-'+index" :label="option.label" :value="option.value"></el-option>
        </el-select>
    </el-form-item>

    <el-form-item v-else-if="type === 'password'" :label="label" :prop="property" :error="error">
        <el-input
            v-model="modelValue"
            @input="$emit('update:modelValue', modelValue)"
            type="password"
            :placeholder="placeholder"
            :disabled="disabled"
            :clearable="clearable"
        ></el-input>
    </el-form-item>

    <el-form-item v-else :label="label" :prop="property" :error="error">
        <el-input
            v-model="modelValue"
            @input="$emit('update:modelValue', modelValue)"
            :placeholder="placeholder"
            :disabled="disabled"
            :clearable="clearable"
            :maxlength="maxlength"
            :minlength="minlength"
            :show-word-limit="showWordLimit"
        ></el-input>
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
        placeholder: {
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
        error: {
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
        showWordLimit: {
            type: Boolean,
            default: false
        },
        clearable: {
            type: Boolean,
            default: false
        },
        filterable: {
            type: Boolean,
            default: true
        },
        options: {
            type: Array,
            default: () => {
                return []
            }
        },
        maxlength: {
            type: [String, Number],
            default: ""
        },
        minlength: {
            type: [String, Number],
            default: ""
        }
    }
}
</script>
