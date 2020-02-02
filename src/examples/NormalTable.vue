<template>
    <div>
        <Table
            :items="items"
            :headings="columns"
            :on-row-click="rowClicked"
            :pagination.sync="pagination"
            :sort.sync="sort"
            table-class="tr-split"
            :tbody-td-class="tbodyTdClass"
            :options="tableOptions"
            responsive
        >
            <template v-slot:OS="data">
                <div class="logo" :style="{'background-image': 'url(' + getImageUrl(data) +')'}"></div>
            </template>
        </Table>
    </div>
</template>

<script>
import Table from '../Vuetable';

export default {
    name: 'NormalTable',
    components: {
        Table
    },
    data() {
        return {
            columns: [
                {
                    'key': 'Name'
                },
                {
                    'key': 'Logo',
                },
                {
                    'key': 'Description'
                },
                {
                    'key': 'ItemStatus',
                    'label': 'Status',
                    'sortable': true
                },
                {
                    'key': 'Assigner',
                    cssClass: (data) => {
                        return data.Assigner;
                    }
                },
                {
                    'key': 'Actions'
                }
            ],
            items: [],
            isPagination: false,
            pagination: {
                perPage: 4,
                totalItems: 0,
                currentPage: 1,
                threshold: 2,
                showJumpToLast: true,
                showJumpToFirst: true
            },
            sort: {
                name: 'ItemStatus',
                value: 'asc'
            },
            tableOptions: {
                export: true,
                arrange: true,
                reset: true
            }
        };
    },
    methods: {
        getImageUrl(data) {
            return data.value.Logo;
        },
        rowClicked(data) {
            alert(JSON.stringify(data));
        },
        tbodyTdClass(header, row) {
            return row.ItemStatus + ' pointer';
        }
    }
};
</script>

<style lang="scss" scoped>
    .logo {
        width: 32px;
        height: 32px;
        border-radius: 50%;
        background-size: cover;
        background-position: center center;
        background-color: white;
    }
</style>
