<template>
    <div class="pt-2 pb-4">
        <div class="mx-auto px-4">
            <div class="text-right">
                <slot name="before_search"></slot>
                <el-input
                    v-if="isSearch"
                    style="width: auto"
                    placeholder="Please input"
                    v-model="search"
                    @clear="onSearch"
                    clearable>
                    <template #append>
                        <el-button icon="el-icon-search" @click="onSearch"></el-button>
                    </template>
                </el-input>
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
                                 :label="column.title"
                                 :width="column.width"
                                 :sortable="column.sortable" >
                    <template #default="scope">
                        <slot :name="column.name" :row="scope.row">{{ fromDotNotation(scope.row, column.name) }}</slot>
                    </template>
                </el-table-column>
                <el-table-column
                    v-if="isEdit || isDelete"
                    :prop="primaryKey"
                    label="Action"
                    :width="actionWidth"
                    header-align="center"
                    align="right"
                >
                    <template #default="scope">
                        <slot name="action_before" :row="scope.row"></slot>
                        <slot name="action" :row="scope.row">
                            <el-button
                                v-if="isEdit"
                                @click.native.prevent="editRow(scope.row)"
                                size="small">
                                Edit
                            </el-button>
                            <el-popconfirm
                                v-if="isDelete && !hasDeleteFalse(scope.row) "
                                title="Are you sure you want to delete?"
                                @confirm="deleteRow(scope.row)"
                            >
                                <template #reference>
                                    <el-button
                                        type="danger"
                                        size="small">
                                        Delete
                                    </el-button>
                                </template>
                            </el-popconfirm>
                        </slot>
                        <slot name="action_after" :row="scope.row"></slot>
                    </template>
                </el-table-column>
                <slot name="custom_action"></slot>
            </el-table>

            <el-pagination
                class="mt-4 text-right"
                :background="false"
                :page-size="per_page"
                v-model:currentPage="current_page"
                background
                :page-sizes="pageSize"
                layout="sizes, prev, pager, next"
                :total="total"
                @size-change="handlePageSizeChange"
                @current-change="handleCurrentPageChange">
            </el-pagination>
        </div>
    </div>
</template>

<script>
import Helpers from './Helper.vue'
export default {
    mixins: [ Helpers ],
    data() {
        return {
            search: '',
            loading: false,
            loadingText: 'Loading...',
            loadingClass: '',
            sort_by: 'id',
            sort_order: 'descending',
            per_page: 20,
            current_page: 1,
            total: 0,
            rows: []
        }
    },
    props: {
        primaryKey: { type: String, default: 'id' },
        indexRoute: { type: String, required: true },
        createRoute: { type: String, default: '' },
        updateRoute: { type: String, default: '' },
        deleteRoute: { type: String, default: '' },
        searchText: { type: String, default: '' },
        actionWidth: { type: String, default: '200px' },
        currentRowKey: { type: String },
        emptyText: { type: String, default: 'No Data' },
        size: { type: String },
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
    },
    mounted() {
        if (this.records){
            this.rows = this.records.data;
            if (!this.records.current_page && this.records.meta){
                this.total = parseInt(this.records.meta.total);
                this.current_page = parseInt(this.records.meta.current_page);
                this.per_page = parseInt(this.records.meta.per_page);
                return;
            }
            this.total = parseInt(this.records.total);
            this.current_page = parseInt(this.records.current_page);
            this.per_page = parseInt(this.records.per_page);
        }
        if (this.searchText){
            this.search = this.searchText;
        }
    },
    methods:{
        editRow(row){
            this.$inertia.get(route(this.updateRoute, row[this.primaryKey]))
        },
        deleteRow(row){
            this.$inertia.delete(route(this.deleteRoute, row[this.primaryKey]))
        },
        reloadPage(){
            const data = {
                page: this.current_page,
                r: this.per_page,
                sort: this.sort_by,
                order: this.sort_order === 'ascending' ? 'a' : 'd',
            };
            if (this.search){
                data['q'] = this.search
            }
            this.$inertia.visit(route(this.indexRoute), {
                method: 'get',
                data: data,
            })
        },
        onSearch(){
            this.current_page = 1;
            this.reloadPage();
        },
        handlePageSizeChange(val){
            this.per_page = val;
            this.reloadPage();
        },
        handleCurrentPageChange(){
            this.reloadPage();
        },

        handleSortChange: function(column) {
            this.sort_by = column.prop;
            this.sort_order = column.order;
            this.reloadPage();
        },
        handleSelectionChange: function(selected) {
            this.selected = selected;
        },
        hasDeleteFalse(row){
            return typeof row.hasDelete !== 'undefined' && row.hasDelete === false;
        },
        emitSelectAll(selection){
            this.$emit('selectAll', selection)
        },
        emitSelectionChange(selection){
            this.$emit('selectionChange', selection)
        },
        emitSelect(selection, row){
            this.$emit('select', selection, row)
        },
        emitCellMouseEnter(row, column, cell, event){
            this.$emit('cellMouseEnter', row, column, cell, event)
        },
        emitCellMouseLeave(row, column, cell, event){
            this.$emit('cellMouseLeave', row, column, cell, event)
        },
        emitCellClick(row, column, cell, event){
            this.$emit('cellClick', row, column, cell, event)
        },
        emitCellDbclick(row, column, cell, event){
            this.$emit('cellDbclick', row, column, cell, event)
        },
        emitCellContextmenu(row, column, cell, event){
            this.$emit('cellContextmenu', row, column, cell, event)
        },
        emitExpandChange(row, expandedRows){
            this.$emit('expandChange', row, expandedRows)
        },
        emitHeaderDragend(newWidth, oldWidth, column, event){
            this.$emit('headerDragend', newWidth, oldWidth, column, event)
        }
    }
}
</script>
