<template>
    <div class="table-wrapper">
        <div class="table-options" v-if="options && clonedItems.length">
            <a v-show="options.export" @click="exportData">
                Export
            </a>
            <a v-show="options.arrange" @click="arrangeColumns">
                Arrange
            </a>
            <a v-show="options.reset" @click="resetTable">
                Reset
            </a>
        </div>
        <table :class="[tableClass, {'table-responsive': $attrs.hasOwnProperty('responsive')}]"
                :style="{'border-spacing': '0px ' + ($attrs.hasOwnProperty('spacing') ? $attrs.spacing + 'px' : '0px'), 'border-collapse': 'separate'}"
                v-if="clonedHeadings">
            <thead>
            <tr>
                <th v-for="(header, hKey) in clonedHeadings" :key="hKey" :class="theadThClass">
                    <div>
                            <span class="sort" v-if="sortBy[header.key]">
                                <i class="material-icons"
                                   :class="{'active': currentSort.name === header.key && currentSort.value === 'asc'}"
                                   @click="changeSort(header.key, 'asc')">arrow_drop_up</i>
                                <i class="material-icons"
                                   :class="{'active': currentSort.name === header.key && currentSort.value === 'desc'}"
                                   @click="changeSort(header.key, 'desc')">arrow_drop_down</i>
                            </span>
                        {{ header.label || header.key || '' }}
                    </div>
                </th>
            </tr>
            <tr class="linear-activity" v-show="isLoading">
                <td class="indeterminate" :colspan="clonedHeadings.length"></td>
            </tr>
            </thead>
            <tbody v-if="clonedItems.length">
            <tr v-for="(row, key) in clonedItems" :key="key" @click="actionWrapper(row)" :class="tbodyTrClass">
                <td v-for="(header, hKey) in clonedHeadings" :key="hKey" :class="setTdClasses(header, row)">
                        <span v-if="$scopedSlots[header.key]">
                            <slot :name="header.key" :value="row"></slot>
                        </span>
                    <span v-else>
                            {{ row[header.key] }}
                        </span>
                </td>
            </tr>
            <tr v-if="isLoading" class="overlay"></tr>
            </tbody>
            <div v-else-if="$slots.empty" v-show="!isLoading">
                <slot name="empty"></slot>
            </div>
            <div class="empty" v-else v-show="!isLoading">
                <p>There are currently no items to show.</p>
            </div>
        </table>
        <div class="pagination" v-if="pagination && clonedItems.length">
            <p>Showing {{ (pagination.currentPage - 1 ) *pagination.perPage + 1 }} - {{
                Math.min(...[((pagination.currentPage) * pagination.perPage), pagination.totalItems]) }} of {{
                pagination.totalItems }} results.</p>
            <ul>
                <li v-if="pagination.currentPage !== 1 && pagination.showJumpToFirst" @click="changePage(1)">
                    <!--eslint-disable-next-line-->
                    <span>{{ '<<' }}</span>
                </li>
                <li v-if="pagination.currentPage !== 1" @click="changePage(--pagination.currentPage)">
                    <!--eslint-disable-next-line-->
                    <span>{{ '<' }}</span>
                </li>
                <li v-for="(val, key) in totalPages" :key="key" :class="{'active': pagination.currentPage === val}"
                    @click="changePage(val)">
                    <span>{{ val }}</span>
                </li>
                <li v-if="clonedItems.length === pagination.perPage" @click="changePage(++pagination.currentPage)">
                    <span>{{ '>' }}</span>
                </li>
                <li v-if="clonedItems.length === pagination.perPage"
                    @click="changePage(Math.ceil(pagination.totalItems/pagination.perPage))">
                    <span>{{ '>>' }}</span>
                </li>
            </ul>
        </div>
        <div class="arrange" v-if="isArrange">
            <div class="arrange__overlay"></div>
            <div class="arrange__box">
                <h6 class="arrange__box__title">Arrange Columns</h6>
                <div class="arrange__box__body">
                    <div class="arrange__box__body__option" v-for="(prop, key) in tempHeadings" :key="key">
                        <div class="arrange__box__body__option__name">{{ prop.label || prop.key }}</div>
                        <div class="arrange__box__body__option__value">
                            <label :for="prop.key" class="checkbox">
                                <input :id="prop.key" type="checkbox" v-model="prop.selected">
                                <span class="checkmark"></span>
                            </label>
                        </div>
                    </div>
                    <div class="text-center">
                        <button @click="saveColumns">Save</button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    import 'material-icons/iconfont/material-icons.scss';
    export default {
        name: 'VueTable',
        props: {
            filters: {},
            headings: {
                type: Array
            },
            items: {
                type: [Array, Function],
                required: true
            },
            onRowClick: {
                type: Function
            },
            pagination: {
                type: Object
            },
            sort: {
                type: Object
            },
            theadThClass: {
                type: String
            },
            tableClass: {
                type: String
            },
            tbodyTrClass: {
                type: String
            },
            tbodyTdClass: {
                type: [String, Function]
            },
            options: {
                type: Object
            }
        },
        data() {
            return {
                clonedItems: [],
                sortedItems: [],
                clonedHeadings: this.headings || [],
                tempHeadings: this.headings,
                sortBy: {},
                currentSort: this.sort,
                isAPI: true,
                isArrange: false,
                isLoading: false
            };
        },
        methods: {
            actionWrapper(row) {
                if (this.onRowClick)
                    this.onRowClick(row);
                else
                    return false;
            },
            arrangeColumns() {
                this.isArrange = true;
            },
            callAPI() {
                this.isLoading = true;
                this.$nextTick(async () => {
                    let obj = {
                        pagination: this.pagination,
                        sort: this.currentSort,
                        filters: this.filters
                    };
                    try {
                        this.clonedItems = await this.items(obj);
                        this.isLoading = false;
                    } catch (err) {
                        this.isLoading = false;
                    }
                    this.isLoading = false;
                });
            },
            changePage(index) {
                this.$emit('update:pagination', {
                    ...this.pagination,
                    currentPage: index
                });
                if (this.isAPI) {
                    this.callAPI();
                } else {
                    this.clonedItems = this.sortedItems.slice((index - 1) * this.pagination.perPage, index * this.pagination.perPage);
                }
            },
            changeSort(key, type) {
                this.currentSort = {
                    name: key,
                    value: type
                };
                if (this.isAPI) {
                    this.callAPI();
                } else {
                    this.sortedItems = this.items.sort((a, b) => {
                        if (type === 'asc') {
                            return a[key] > b[key] ? 1 : -1;
                        } else if (type === 'desc')
                            return b[key] > a[key] ? 1 : -1;
                        return 0;
                    });
                    if (this.pagination)
                        this.changePage(this.pagination.currentPage);
                    else
                        this.clonedItems = this.sortedItems;
                }
            },
            exportData() {
                let csvContent = 'data:text/csv;charset=utf-8,';
                let includedColumns = [];
                csvContent += this.clonedHeadings.map(head => {
                    includedColumns.push(head.key);
                    return head.label || head.key;
                });
                csvContent += '\r\n';
                this.clonedItems.map((item) => {
                    includedColumns.map(key => {
                        if (typeof item[key] === 'object') {
                            csvContent += this.processObject(item[key]) + ',';
                        } else if (item[key] instanceof Date) {
                            csvContent += item[key].toLocaleString();
                        } else if (key !== '_id') {
                            csvContent += item[key] + ',';
                        }
                    });
                    csvContent += '\r\n';
                });
                let encodedUri = encodeURI(csvContent);
                window.open(encodedUri);
            },
            processObject(obj) {
                let str = '';
                for (let key in obj) {
                    if (typeof obj[key] === 'object') {
                        str += this.processObject(obj[key]);
                    } else if (obj[key] instanceof Date) {
                        str += obj[key].toLocaleString() + ';';
                    } else if (key !== '_id') {
                        str += key + ': ' + obj[key] + ';';
                    }
                }
                return str;
            },
            resetTable() {
                this.clonedHeadings = [];
            },
            saveColumns() {
                this.clonedHeadings = this.tempHeadings.filter(heading => heading.selected);
                this.isArrange = false;
            },
            setTdClasses(header, row) {
                let classes = [];
                if (header.cssClass && typeof header.cssClass === 'function') {
                    classes.push(header.cssClass(row));
                } else {
                    classes.push(header.cssClass);
                }
                if (typeof this.tbodyTdClass === 'function') {
                    classes.push(this.tbodyTdClass(header, row));
                } else {
                    classes.push(this.tbodyTdClass);
                }
                return classes;
            },
            validateAllProps() {
                let err;
                if (!this.clonedHeadings || !this.clonedHeadings.length) {
                    err = 'No headings provided in table. Please make sure to provide headings';
                } else if (!Array.isArray(this.items) && !this.pagination) {
                    err = 'items must be a function when using it with the pagination or pagination object must be present when items is a function';
                }
                return err;
            }
        },
        computed: {
            totalPages() {
                let threshold = this.pagination.threshold ? Math.ceil(this.pagination.threshold) : 2;
                let total = Math.ceil(this.pagination.totalItems / this.pagination.perPage);
                let pages = new Set();
                let counter = this.pagination.currentPage;
                while (counter >= 1 && (this.pagination.currentPage - counter) <= threshold / 2) {
                    pages.add(counter);
                    counter--;
                }
                counter = this.pagination.currentPage;
                while (counter <= total && (counter - this.pagination.currentPage) <= threshold / 2) {
                    pages.add(counter);
                    counter++;
                }
                return Array.from(pages).sort();
            }
        },
        watch: {
            filters: {
                handler() {
                    this.pagination.currentPage = 1;
                    this.callAPI();
                },
                deep: true
            }
        },
        async created() {
            // Initialise the sorts
            let err = this.validateAllProps();
            if (err) {
                throw err;
            }
            for (let key in this.clonedHeadings) {
                if (this.clonedHeadings[key].sortable) {
                    this.sortBy[this.clonedHeadings[key].key] = true;
                }
            }
            if (Array.isArray(this.items)) {
                this.isAPI = false;
                this.pagination.totalItems = this.items.length;
            }

            if (this.sort && this.sort.name) {
                this.changeSort(this.sort.name, this.sort.value);
            } else {
                if (!this.isAPI) {
                    if (this.pagination) {
                        this.clonedItems = this.items.slice((this.pagination.currentPage - 1) * this.pagination.perPage, this.pagination.perPage);
                    } else {
                        this.clonedItems = this.items;
                    }
                } else {
                    this.callAPI();
                }
            }
        }
    };
</script>

<style lang="scss" scoped>
    @import './scss/style';
    .table-wrapper {
        z-index: 1;

        .table-options {
            margin-bottom: 16px;
            text-align: right;

            a {
                padding: 4px 16px;
                color: #fff;
                background-color: $brand-primary;
                margin-right: 8px;
                border-radius: 4px;
                cursor: pointer;
            }
        }

        table {
            width: 100%;
            border-spacing: 0;
            position: relative;

            &.table-striped {
                tbody {
                    tr:nth-child(2n+1) {
                        td {
                            background-color: $lighter-grey;
                        }
                    }
                }
            }

            &__responsive {
                overflow-x: auto;
            }

            thead {
                tr {
                    th {
                        text-align: left;
                        background-color: $brand-primary;
                        color: $white;
                        padding: 12px 8px;

                        > div {
                            display: flex;
                            align-items: center;
                            justify-content: flex-start;
                        }

                        .sort {
                            display: flex;
                            flex-direction: column;
                            align-content: flex-start;

                            i {
                                line-height: 12px;
                                color: $white;
                                opacity: 0.5;
                                cursor: pointer;

                                &:hover {
                                    opacity: 0.8;
                                }

                                &.active {
                                    opacity: 1;
                                }
                            }
                        }
                    }

                    &:first-child {
                        th {
                            &:first-child {
                                border-top-left-radius: $border-radius;
                            }

                            &:last-child {
                                border-top-right-radius: $border-radius;
                            }
                        }
                    }
                }
            }

            > div {
                position: relative;
            }

            tbody {
                .overlay {
                    position: absolute;
                    top: 40px;
                    left: 0;
                    bottom: 0;
                    right: 0;
                    width: 100%;
                    height: calc(100% - 48px);
                    background-color: black;
                    opacity: 0.2;
                }

                tr {
                    margin-top: 8px;

                    td {
                        padding: 12px 8px;
                        border-bottom: 1px solid $lighter-grey;
                        text-align: left;
                    }

                    &:last-child {
                        td {
                            &:first-child {
                                border-bottom-left-radius: $border-radius;
                            }

                            &:last-child {
                                border-bottom-right-radius: $border-radius;
                            }
                        }
                    }
                }
            }

            .empty {
                padding: 8px 0 0 8px;
                p {
                    text-align: left;
                }
            }

            .linear-activity {
                overflow: hidden;
                width: 100%;
                height: 1px;
                background-color: $white;
                margin: 1px auto;

                .indeterminate {
                    position: relative;
                    width: 100%;
                    height: 100%;
                    padding: 1px;

                    &:before {
                        content: '';
                        position: absolute;
                        top: 0;
                        height: 100%;
                        background-color: $dark-grey;
                        animation: indeterminate_first 1.5s infinite ease-out;
                    }
                }
            }
        }

        .pagination {
            float: right;
            margin: 16px;

            p {
                text-align: right;
            }

            ul {
                list-style: none;
                padding-left: 0;

                li {
                    cursor: pointer;
                    display: inline-flex;
                    align-items: center;
                    justify-content: center;
                    padding: 8px;
                    margin-right: 4px;
                    margin-bottom: 4px;
                    width: 20px;
                    text-align: center;
                    background-color: $white;
                    color: $brand-primary;
                    border: 1px solid $lighter-grey;
                    border-radius: $border-radius;

                    &.active {
                        color: $white;
                        background-color: $brand-primary;
                    }
                }
            }
        }

        .arrange {
            justify-content: center;
            align-items: center;
            display: flex;
            width: 100%;
            position: fixed;
            top: 0;
            left: 0;
            right: 0;
            height: 100%;

            &__overlay {
                position: absolute;
                opacity: 0.4;
                background-color: black;
                z-index: 1;
                margin: 0;
                padding: 0;
                width: inherit;
                height: inherit;
            }

            &__box {
                z-index: 2;
                width: 300px;
                background-color: $white;
                border-radius: $border-radius;

                &__title {
                    background-color: $brand-primary;
                    color: $white;
                    padding: 16px;
                    margin: 0;
                    border-top-right-radius: $border-radius;
                    border-top-left-radius: $border-radius;
                }

                &__body {
                    padding: 16px;

                    &__option {
                        display: block;
                        width: 100%;
                        margin-bottom: 16px;

                        &__name {
                            display: inline-block;
                            color: $light-grey;
                            width: 70%;
                        }

                        &__value {
                            width: 30%;
                            display: inline-block;
                            text-align: right;
                        }

                        .checkbox {
                            display: block;
                            position: relative;
                            padding-left: 35px;
                            margin-bottom: 12px;
                            cursor: pointer;
                            font-size: 22px;
                            user-select: none;
                        }

                        .checkbox input {
                            position: absolute;
                            opacity: 0;
                            cursor: pointer;
                            height: 0;
                            width: 0;
                        }

                        .checkmark {
                            position: absolute;
                            top: 0;
                            right: 0;
                            height: 25px;
                            width: 25px;
                            background-color: #eee;
                        }

                        .checkbox input:checked ~ .checkmark {
                            background-color: #eee;
                        }

                        .checkmark:after {
                            content: "";
                            position: absolute;
                            display: none;
                        }

                        .checkbox input:checked ~ .checkmark:after {
                            display: block;
                        }

                        .checkbox .checkmark:after {
                            right: 9px;
                            top: 5px;
                            width: 5px;
                            height: 10px;
                            border: solid $brand-primary;
                            border-width: 0 3px 3px 0;
                            transform: rotate(45deg);
                        }
                    }

                    .inline-block {
                        display: inline-block;
                    }
                }
            }
        }
    }

    @keyframes indeterminate_first {
        0% {
            left: 0%;
            width: 10%;
        }
        100% {
            left: 90%;
            width: 10%;
        }
    }
</style>
