<template>
    <div class="py-6">
        <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
            <div class="text-right">
                <el-input
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
                stripe
                style="width: 100%"
                v-on:sort-change="handleSortChange"
                v-on:selection-change="handleSelectionChange"
                :row-class-name="rowClassName"
            >
                <el-table-column v-if="selectable" type="selection" width="55"></el-table-column>
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
                    v-if="updateRoute || deleteRoute"
                    prop="id"
                    label="Action"
                    width="200px"
                    header-align="center"
                    align="right"
                >
                    <template #default="scope">
                        <slot name="action" :row="scope.row"></slot>
                        <el-button
                            v-if="updateRoute"
                            @click.native.prevent="editRow(scope.row)"
                            size="small">
                            Edit
                        </el-button>
                        <el-popconfirm
                            v-if="deleteRoute"
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

                    </template>
                </el-table-column>
            </el-table>

            <el-pagination
                class="mt-4 text-right"
                :background="false"
                :page-size="per_page"
                v-model:currentPage="current_page"
                background
                :page-sizes="[5, 20, 50, 100]"
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
        records: { type: Object, default: () => {
            return {
                data: [],
                total: 0,
                current_page: 1,
                per_page: 20
            }
        }},
        columns: Array,
        titles: { type: Object, default: () => { return {}; } },
        order: { type: Object, default: () => {} },
        selectable: { type: Boolean, default: false },
        isEdit: { type: Boolean, default: true },
        isDelete: { type: Boolean, default: true },
        rowClassName: { type:Function, default: null },
    },
    mounted() {
        if (this.records){
            this.rows = this.records.data;
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
    }
}
</script>
