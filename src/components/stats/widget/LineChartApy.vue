<template>
    <div class="apy-chart-container">
        <v-row class="chart-header-row">
            <v-col>
                <v-row justify="start">
                    <label class="chart-title">{{ (avgApy && avgApy.value) ? ((isMobile ? '' : 'Average ') + 'USD+&nbsp;') : ''}}</label>
                    <label class="chart-title" style="margin-left: 0 !important"><abbr title="Annual Percentage Yield">APY</abbr></label>
                </v-row>

                <v-row justify="start">
                    <label class="mobile-info-title">
                        {{ (avgApy && avgApy.value) ? ($utils.formatMoneyComma(avgApy.value, 1)) + '%' : '' }}
                    </label>
                </v-row>
            </v-col>
            <v-col class="add-chart-info-col">
                <v-row justify="end">
                    <label class="chart-title-apy">
                        {{ (avgApy && avgApy.value) ? ($utils.formatMoneyComma(avgApy.value, 1)) + '%' : '' }}
                    </label>
                </v-row>
                <v-row justify="end">
                    <label class="chart-sub-title-apy">
                        {{ (avgApy && avgApy.date) ? 'from ' + avgApy.date : '' }}
                    </label>
                </v-row>
            </v-col>
        </v-row>

        <div class="chart-row" id="line-chart-apy"></div>

        <v-row class="zoom-row" style="margin-top: -40px !important;">
            <v-spacer></v-spacer>
            <v-btn
                    rounded
                    text
                    id="week-zoom-btn"
                    class="zoom-btn"
                    dark
                    @click="zoomChart('week')"
            >
                <label>Week</label>
            </v-btn>
            <v-btn
                    rounded
                    text
                    id="month-zoom-btn"
                    class="zoom-btn"
                    dark
                    @click="zoomChart('month')"
            >
                Month
            </v-btn>
            <v-btn
                    rounded
                    text
                    style="margin-right: 1%;"
                    id="all-zoom-btn"
                    class="zoom-btn"
                    dark
                    @click="zoomChart('all')"
            >
                All
            </v-btn>
        </v-row>
    </div>
</template>

<script>
/* eslint-disable no-unused-vars,no-undef */

import {mapActions, mapGetters, mapMutations} from "vuex";
import moment from "moment";

import ApexCharts from 'apexcharts'

export default {
    name: "LineChartApy",

    props: {
        data: {
            type: Object,
            default: null,
        },
    },

    watch: {
        data: function (newVal, oldVal) {
            this.redraw();
        },
    },

    components: {},

    data: () => ({
        zoom: "week",
        slice: null,
        chart: null,

        avgApy: null,
    }),

    computed: {
        ...mapGetters([]),

        isMobile() {
            return window.innerWidth < 650;
        }
    },

    mounted() {
        this.redraw();
    },

    created() {
        this.zoomChart("week");
    },

    methods: {
        ...mapMutations([]),

        async zoomChart(zoom) {
            let fetchOptions = {
                headers: {
                    "Access-Control-Allow-Origin": process.env.VUE_APP_API
                }
            };

            await fetch(process.env.VUE_APP_API + '/widget/avg-apy-info/' + zoom, fetchOptions)
                .then(value => value.json())
                .then(value => {
                    this.avgApy = value;
                    this.avgApy.date = moment(this.avgApy.date).format("DD MMM. ‘YY");
                }).catch(reason => {
                    console.log('Error get data: ' + reason);
                })

            this.zoom = zoom;

            switch (zoom) {
                case "week":
                    this.slice = -7;
                    break;
                case "month":
                    this.slice = -30;
                    break;
                case "all":
                    this.slice = null;
                    break;
                default:
                    this.slice = null;
            }

            if (this.chart) {
                this.chart.destroy();
            }

            this.redraw();
        },

        changeZoomBtnStyle() {
            document.getElementById("week-zoom-btn").classList.remove("selected");
            document.getElementById("month-zoom-btn").classList.remove("selected");
            document.getElementById("all-zoom-btn").classList.remove("selected");

            document.getElementById(this.zoom + "-zoom-btn").classList.add("selected");
        },

        redraw() {
            if (this.chart) {
                this.chart.destroy();
            }

            this.changeZoomBtnStyle();

            let values = [];
            this.data.datasets[0].data.forEach(v => values.push(v));
            values = this.slice ? values.slice(this.slice) : values;

            let labels = [];
            this.data.labels.forEach(v => labels.push(v));
            labels = this.slice ? labels.slice(this.slice) : labels;

            let averageValue = this.avgApy.value;

            let maxValue;
            try {
                maxValue = Math.max.apply(Math, values);
                maxValue = Math.round(Math.ceil(maxValue / 10)) * 10;
            } catch (e) {
                maxValue = 50;
            }

            let options = {
                series: [{
                    name: "APY",
                    data: values
                }],

                labels: labels,

                chart: {
                    type: 'area',
                    height: this.isMobile ? 100 : 230,

                    sparkline: {
                        enabled: false,
                    },

                    zoom: {
                        enabled: false
                    },

                    background: '#1D2029',

                    toolbar: {
                        show: false
                    }
                },

                annotations: {
                    position: 'front',
                    yaxis: [{
                        y: averageValue,
                        strokeDashArray: 5,
                        borderColor: '#23DD00',
                        fillColor: '#23DD00',
                        label: {
                            show: false,
                        },
                        width: this.isMobile ? '0%' : '100%',
                    }]
                },

                grid: {
                    show: false,
                },

                dataLabels: {
                    enabled: false
                },

                stroke: {
                    curve: 'straight',
                    width: this.isMobile ? 1 : 2,
                    colors: this.isMobile ? ["#23DD00"] : ["#51FF00"],
                },

                xaxis: {
                    type: 'category',

                    tickAmount: this.isMobile ? 6 : 10,
                    tickPlacement: 'between',

                    labels: {
                        show: false,
                    },

                    axisBorder: {
                        show: false,
                    },

                    axisTicks: {
                        show: true,
                    },
                },

                yaxis: {
                    opposite: false,

                    tickAmount: 5,
                    min: 0,
                    max: maxValue,

                    labels: {
                        show: false,
                    },
                },

                legend: {
                    horizontalAlign: 'left'
                },

                colors: this.isMobile ? ['#181E25'] : ['#68D55A'],

                theme: {
                    mode: 'dark',
                },

                fill: {
                    type: ['gradient'],

                    gradient: {
                        shade: 'dark',
                        type: "vertical",
                        shadeIntensity: 0.2,
                        opacityFrom: 1,
                        opacityTo: 0,
                        stops: [0, 100],
                    },
                }
            };

            this.chart = new ApexCharts(document.querySelector("#line-chart-apy"), options);
            this.chart.render();
        },
    }
}
</script>

<style>

/* mobile */
@media only screen and (max-width: 1400px) {

    .chart-title {
        margin-left: 4%;
        margin-top: 30px !important;
        font-family: 'Raleway', sans-serif;
        font-style: normal;
        font-weight: 800;
        font-size: 16px;
        line-height: 24px;
        color: #FFFFFF;
    }

    .mobile-info-title {
        margin-top: 5px !important;
        margin-left: 4%;
        font-family: 'Lato', sans-serif;
        font-style: normal;
        font-weight: 700;
        font-size: 20px;
        line-height: 28px;
        color: #FFFFFF;
        z-index: 2 !important;
    }

    .add-chart-info-col, .zoom-row {
        display: none !important;
    }

    .chart-row {
        margin-bottom: -10px !important;
    }
}

@media only screen and (min-width: 1400px) {

    .chart-title {
        margin-left: 4%;
        margin-top: 30px !important;
        font-family: 'Raleway', sans-serif;
        font-style: normal;
        font-weight: 800;
        font-size: 24px;
        line-height: 36px;
        color: #FFFFFF;
    }

    .chart-title-apy {
        margin-right: 4%;
        margin-top: 30px !important;
        font-family: 'Lato', sans-serif;
        font-style: normal;
        font-weight: 700;
        font-size: 34px;
        line-height: 42px;
        color: #FFFFFF;
    }

    .chart-sub-title-apy {
        margin-right: 4%;
        font-family: 'Lato', sans-serif;
        font-style: normal;
        font-weight: 400;
        font-size: 16px;
        line-height: 24px;
        color: #707A8B;
    }

    .mobile-info-title {
        display: none !important;
    }

    .zoom-row {
        height: 20px !important;
    }

    .chart-header-row {
        height: 150px !important;
    }

    .chart-row {
        height: 250px !important;
    }

    .apy-chart-container {
        height: 420px !important;
    }
}

#line-chart-apy {
    overflow-x: hidden !important;
    overflow-y: hidden !important;
}

.yaxis-label {
    font-size: 12px !important;
    line-height: 12px !important;
    font-weight: 400;
    fill: rgba(255, 255, 255, 0.6) !important;
}

.zoom-btn {
    border: none !important;
    font-family: 'Lato', sans-serif;
    font-style: normal;
    font-weight: 400;
    font-size: 16px;
    line-height: 24px;
    color: #707A8B !important;
}

.selected {
    color: #FCCA46 !important;
    background-color: rgba(255, 213, 5, 0.15);
}

.yaxis-label {
    display: none !important;
}

.chart-header-row, .chart-row, .zoom-row {
    margin-left: 4%;
    margin-right: 4%;
}

</style>