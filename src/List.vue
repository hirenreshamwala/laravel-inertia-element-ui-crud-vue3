<template>
    <div class="pb-4">
        <div v-if="filters.length > 0" class="p-3 form-filter">
            <el-form :inline="true" label-position="top" ref="form" :model="entity">
                <div v-for="column in filters" :key="column">
                    <slot name="before_filter"></slot>
                    <slot :name="'filter_'+column.property" :entity="entity" :column="column">
                        <Field
                            :property="column.property"
                            :label="column.label"
                            :title="column.title"
                            :options="column.options"
                            :type="column.type"
                            :disabled="!!column.disabled"
                            v-model="entity[column.property]"
                            clearable
                        ></Field>
                    </slot>
                    <slot name="after_filter"></slot>
                </div>
                <slot name="filter_action">
                    <el-form-item label="&nbsp;">
                        <slot name="filter_button_before"></slot>
                        <el-button type="primary" @click="onSearch">Search</el-button>
                        <slot name="filter_button_after"></slot>
                    </el-form-item>
                </slot>
            </el-form>
        </div>
        <div class="mx-auto pt-3">
            <div :class="{'flex-row-reverse': !searchBoxLeft}" class="flex" v-if="isSearch && filters.length === 0">
                <slot v-if="isSearch && filters.length === 0" name="before_search"></slot>
                <el-input
                    style="width: auto"
                    :placeholder="searchPlaceholder"
                    v-model="search"
                    @clear="onSearch"
                    clearable>
                    <template #append>
                        <el-button :icon="Search" @click="onSearch" />
                    </template>
                </el-input>
                <slot v-if="isSearch && filters.length === 0" name="after_search"></slot>
            </div>
            <el-table
                v-loading="loading"
                :element-loading-text="loadingText"
                class="mt-4"
                :data="rows"
                :stripe="stripe"
                :border="border"
                :size="size"
                :fit="fit"
                :show-header="showHeader"
                style="width: 100%"
                :default-sort="{prop:sort.field,order:sort.order}"
                v-on:sort-change="handleSortChange"
                v-on:selection-change="handleSelectionChange"
                :row-class-name="rowClassName"
                :row-style="rowStyle"
                :highlight-current-row="highlightCurrentRow"
                :row-key="primaryKey"
                :current-row-key="currentRowKey"
                :cell-class-name="cellClassName"
                :cell-style="cellStyle"
                :header-row-class-name="headerRowClassName"
                :header-row-style="headerRowStyle"
                :header-cell-class-name="headerCellClassName"
                :header-cell-style="headerCellStyle"
                :span-method="spanMethod"
                :empty-text="emptyText"
                :default-expand-all="defaultExpandAll"
                :expand-row-keys="expandRowKeys"
                @select="emitSelect"
                @select-all="emitSelectAll"
                @selection-change="emitSelectionChange"
                @cell-mouse-enter="emitCellMouseEnter"
                @cell-mouse-leave="emitCellMouseLeave"
                @cell-click="emitCellClick"
                @cell-dblclick="emitCellDbclick"
                @cell-contextmenu="emitCellContextmenu"
                @expand-change="emitExpandChange"
                @header-dragend="emitHeaderDragend"
            >
                <el-table-column v-if="selectable" type="selection" width="55"></el-table-column>
                <el-table-column v-if="expandable" type="expand">
                    <template #default="props">
                        <slot name="expandable" :row="props.row"></slot>
                    </template>
                </el-table-column>
                <el-table-column v-for="column in columns"
                                 :key="column.name"
                                 :prop="column.name"
                                 :align="column.align"
                                 :resizable="column.resizable"
                                 :class-name="column.className"
                                 :label-class-name="column.labelClassName"
                                 :fixed="column.fixed"
                                 :label="column.title"
                                 :width="column.width"
                                 :min-width="column.minWidth"
                                 :sortable="column.sortable" >
                    <template #default="scope">
                        <slot :name="column.name" :row="scope.row">
                            <div v-if="column.type && column.type === 'status'">
                                <el-tag v-if="fromDotNotation(scope.row, column.name) === true" type="success">Enabled</el-tag>
                                <el-tag v-else type="danger">Disabled</el-tag>
                            </div>
                            <span v-else>{{ fromDotNotation(scope.row, column.name) }}</span>
                        </slot>
                    </template>
                </el-table-column>
                <el-table-column
                    v-if="isEdit || isDelete"
                    :prop="primaryKey"
                    :label="actionLabel"
                    :width="actionWidth"
                    header-align="center"
                    align="right"
                >
                    <template #default="scope">
                        <slot name="action_before" :row="scope.row"></slot>
                        <slot name="action" :row="scope.row">
                            <el-button
                                type="primary"
                                class="mx-1"
                                v-if="isEdit && !iconButtons"
                                @click.native.prevent="editRow(scope.row)"
                                size="small"
                                >
                                Edit
                            </el-button>
                            <el-tooltip
                                v-if="isEdit && iconButtons"
                                class="box-item"
                                effect="dark"
                                content="Edit"
                                placement="top"
                            >
                                <el-button
                                    class="mx-1"
                                    type="primary"
                                    @click.native.prevent="editRow(scope.row)"
                                >
                                    <el-icon><EditPen /></el-icon>
                                </el-button>
                            </el-tooltip>
                            <el-popconfirm
                                v-if="isDelete && !hasDeleteFalse(scope.row) "
                                title="Are you sure you want to delete?"
                                @confirm="deleteRow(scope.row)"
                            >
                                <template #reference>
                                    <span>
                                        <el-tooltip
                                            v-if="iconButtons"
                                            effect="dark"
                                            content="Delete"
                                            placement="top"
                                        >
                                            <el-button
                                                class="mx-1"
                                                type="danger"
                                            >
                                                <el-icon><Delete /></el-icon>
                                            </el-button>
                                        </el-tooltip>
                                        <el-button
                                            v-else
                                            type="danger"
                                            class="mx-1"
                                            size="small">
                                            Delete
                                        </el-button>
                                    </span>
                                </template>
                            </el-popconfirm>
                        </slot>
                        <slot name="action_after" :row="scope.row"></slot>
                    </template>
                </el-table-column>
                <slot name="custom_action"></slot>
            </el-table>

            <el-pagination
                v-if="isPagination"
                class="mt-4 text-right"
                :background="false"
                :page-size="per_page"
                v-model:currentPage="current_page"
                :page-sizes="pageSize"
                layout="sizes, prev, pager, next"
                :total="total"
                @size-change="handlePageSizeChange"
                @current-change="handleCurrentPageChange">
            </el-pagination>
        </div>
    </div>
</template>

<script setup>
import { inject, ref, reactive, onMounted, watch } from 'vue';
const route = inject('appRoute');
import {Inertia} from "@inertiajs/inertia";
import Field from "./Field";
import {
    Search,
    Delete,
    EditPen
} from '@element-plus/icons-vue'

const props = defineProps({
    primaryKey: { type: String, default: 'id' },
    indexRoute: { type: String, required: true },
    createRoute: { type: String, default: '' },
    updateRoute: { type: String, default: '' },
    deleteRoute: { type: String, default: '' },
    searchText: { type: String, default: '' },
    actionWidth: { type: String, default: '200px' },
    actionLabel: { type: String, default: 'Action' },
    currentRowKey: { type: String },
    emptyText: { type: String, default: 'No Data' },
    searchPlaceholder: { type: String, default: 'Search input' },
    searchBoxLeft: { type: Boolean, default: false },
    size: { type: String },
    filters: {
        type: Array,
        default: () => {
            return [];
        }
    },
    pageSize: { type: Array, default: () => {
            return [5, 20, 50, 100]
        }},
    records: { type: Object, default: () => {
            return {
                data: [],
                total: 0,
                current_page: 1,
                per_page: 20
            }
        }},
    sort: { type: Object, default: () => {
            return {
                field: 'id',
                order: 'descending'
            }
        }},
    params: { type: Object, default: () => {
            return {}
        }},
    columns: Array,
    expandRowKeys: Array,
    titles: { type: Object, default: () => { return {}; } },
    order: { type: Object, default: () => {} },
    selectable: { type: Boolean, default: false },
    stripe: { type: Boolean, default: false },
    border: { type: Boolean, default: false },
    expandable: { type: Boolean, default: false },
    fit: { type: Boolean, default: true },
    showHeader: { type: Boolean, default: true },
    isEdit: { type: Boolean, default: true },
    isDelete: { type: Boolean, default: true },
    isSearch: { type: Boolean, default: true },
    isPagination: { type: Boolean, default: true },
    iconButtons: { type: Boolean, default: false },
    defaultExpandAll: { type: Boolean, default: false },
    rowClassName: { type:[Function, String], default: null },
    rowStyle: { type:[Function, String], default: null },
    highlightCurrentRow: { type: Boolean, default: false },
    cellClassName: { type:[Function, String], default: null },
    cellStyle: { type:[Function, String], default: null },
    headerRowClassName: { type:[Function, String], default: null },
    headerRowStyle: { type:[Function, String], default: null },
    headerCellClassName: { type:[Function, String], default: null },
    headerCellStyle: { type:[Function, String], default: null },
    spanMethod: { type:Function, default: null },
});
const search = ref('');
const loading = ref(false);
const loadingText = ref('Loading...');
const loadingClass = ref('');
const sort_by = ref('id');
const sort_order = ref('descending');
const per_page = ref(20);
const current_page = ref(1);
const total = ref(0);
const rows = ref([]);
const emit = defineEmits(['selectAll', 'selectionChange', 'select', 'cellMouseEnter', 'cellMouseLeave', 'cellClick', 'cellDbclick', 'cellContextmenu', 'expandChange', 'headerDragend'])
const entity = reactive({});

onMounted(() => {
    loadRecords();
    if (props.searchText){
        search.value = props.searchText;
    }
    sort_by.value = props.sort.field;
    sort_order.value = props.sort.order;
    loadEntities();
});
watch(() => props.records, (first, second) => {
    loadRecords();
});
watch(() => props.filters, (first, second) => {
    loadEntities();
});
const loadEntities = () => {
    if(props.filters && props.filters.length > 0){
        for (let i = 0; i < props.filters.length; i++) {
            const row = props.filters[i];
            entity[row.property] = row.value;
        }
    }
};
const loadRecords = () => {
    if (props.records){
        rows.value = props.records.data;
        if (!props.records.current_page && props.records.meta){
            total.value = parseInt(props.records.meta.total);
            current_page.value = parseInt(props.records.meta.current_page);
            per_page.value = parseInt(props.records.meta.per_page);
            return;
        }
        total.value = parseInt(props.records.total);
        current_page.value = parseInt(props.records.current_page);
        per_page.value = parseInt(props.records.per_page);
    }
}
const editRow = (row) => {
    Inertia.get(route(props.updateRoute, row[props.primaryKey]))
}
const deleteRow = (row) => {
    Inertia.delete(route(props.deleteRoute, row[props.primaryKey]))
}
const reloadPage = () => {
    const data = {
        page: current_page.value,
        r: per_page.value,
        sort: sort_by.value,
        order: sort_order.value === 'ascending' ? 'a' : 'd',
        filters: entity
    };
    if (search.value){
        data['q'] = search.value
    }
    const params = {...data, ...props.params }
    Inertia.visit(route(props.indexRoute), {
        method: 'get',
        data: params,
    })
}
const fromDotNotation = (obj, path) => {
    var i = 0,
        path = path.split('.');

    for (; i < path.length; i++) {
        if (typeof obj[path[i]] === 'undefined' || !obj[path[i]]) {
            return '';
        }

        obj = obj[path[i]];
    }

    return obj;
}
const onSearch = () => {
    current_page.value = 1;
    reloadPage();
}
const handlePageSizeChange = (val) => {
    per_page.value = val;
    reloadPage();
}
const handleCurrentPageChange = () => {
    reloadPage();
}

const handleSortChange = (column) => {
    sort_by.value = column.prop;
    sort_order.value = column.order;
    reloadPage();
}
const handleSelectionChange = (selected) => {

}
const hasDeleteFalse = (row) => {
    return typeof row.hasDelete !== 'undefined' && row.hasDelete === false;
}
const emitSelectAll = (selection) => {
    emit('selectAll', selection)
}
const emitSelectionChange = (selection) => {
    emit('selectionChange', selection)
}
const emitSelect = (selection, row) => {
    emit('select', selection, row)
}
const emitCellMouseEnter = (row, column, cell, event) => {
    emit('cellMouseEnter', row, column, cell, event)
}
const emitCellMouseLeave = (row, column, cell, event) => {
    emit('cellMouseLeave', row, column, cell, event)
}
const emitCellClick = (row, column, cell, event) => {
    emit('cellClick', row, column, cell, event)
}
const emitCellDbclick = (row, column, cell, event) => {
    emit('cellDbclick', row, column, cell, event)
}
const emitCellContextmenu = (row, column, cell, event) => {
    emit('cellContextmenu', row, column, cell, event)
}
const emitExpandChange = (row, expandedRows) => {
    emit('expandChange', row, expandedRows)
}
const emitHeaderDragend = (newWidth, oldWidth, column, event) => {
    emit('headerDragend', newWidth, oldWidth, column, event)
}
defineExpose({
    reloadPage
})
</script>
<style scoped>
.el-form--inline .el-form-item{
    margin-right: 12px;
}
.el-form-item{
    margin-bottom: 8px;
}
</style>
