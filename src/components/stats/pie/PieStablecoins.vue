<template>
    <vc-donut
            background="none"
            :size="size" unit="px"
            :sections="sections"
            :start-angle="0" :auto-adjust-text-size="false">
    </vc-donut>
</template>

<script>

import {mapActions, mapGetters, mapMutations} from "vuex";

/* eslint-disable no-unused-vars,no-undef */

export default {
    name: "PieStablecoins",

    props: {
        data: {
            type: Array,
            default: null,
        },

        size: {
            type: Number,
            default: 320,
        },
    },

    watch: {
        data: function (newVal, oldVal) {
            this.updateSectionsData();
        },

        size: function (newVal, oldVal) {
            this.updateSectionsData();
        },
    },

    components: {},

    data: () => ({
        sections: [],
    }),

    computed: {
    },

    mounted() {
        this.updateSectionsData();
    },

    created() {
        this.updateSectionsData();
    },

    methods: {
        ...mapActions([]),

        ...mapMutations([]),

        getPercent(item, data) {
            let sum = data.map(dataItem => dataItem.value).reduce((prev, next) => prev + next);

            return (item.value * 100 / sum) - 0;
        },

        updateSectionsData() {
            this.sections = [];

            this.data.forEach(item => {
                this.sections.push(
                    {
                        color: item.color,
                        label: item.label,
                        value: this.getPercent(item, this.data),
                    }
                )
            });
        },

        getTotal() {
            let sum = 0;
            this.data.forEach(dataItem => {
                sum += dataItem.value
            });

            return sum;
        },
    }
}
</script>

<style>

.total-label {
    font-family: 'Lato', sans-serif !important;
    font-weight: 300;
    font-size: 24px;
    line-height: 36px;
    color: white !important;
}

.total-sum-label {
    font-family: 'Lato', sans-serif !important;
    font-weight: 700;
    font-size: 34px;
    line-height: 42px;
    color: white !important;
}

</style>