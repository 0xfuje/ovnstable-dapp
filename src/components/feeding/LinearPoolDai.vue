<template>
    <v-col lg="4" class="pa-0 ma-0 px-2">
        <v-row class="pt-5">
            <v-card flat class="main-card">
                <v-card-text>

                    <v-row justify="start">
                        <label class="recent-label">
                            Balance DAI: <b>{{ balances.dai }}</b>
                        </label>
                    </v-row>

                    <v-row justify="start">
                        <label class="recent-label">
                            Balance aDAI: <b>{{ balances.aDai }}</b>
                        </label>
                    </v-row>

                    <v-row justify="start">
                        <label class="recent-label">
                            Balance Static DAI: <b>{{ balances.staticDai }}</b>
                        </label>
                    </v-row>

                    <v-row justify="start">
                        <label class="recent-label">
                            Balance Pool LP: <b>{{ balances.lpToken }}</b>
                        </label>
                    </v-row>
                    <v-row>
                        <v-col>
                            <v-row>
                                <label class="title-row-label ml-5 mt-3">Amount DAI</label>
                            </v-row>
                            <v-row align="center">
                                <v-text-field placeholder="0.00"
                                              @keypress="isNumber($event)"
                                              flat
                                              solo
                                              class="ml-2 field-sum"
                                              hide-details
                                              dark
                                              background-color="transparent"
                                              v-model="amountDai">
                                </v-text-field>
                                <v-spacer></v-spacer>
                            </v-row>
                        </v-col>


                        <v-col>
                            <v-row>
                                <label class="title-row-label ml-5 mt-3">Amount aDAI</label>
                            </v-row>
                            <v-row align="center">
                                <v-text-field placeholder="0.00"
                                              @keypress="isNumber($event)"
                                              flat
                                              solo
                                              class="ml-2 field-sum"
                                              hide-details
                                              dark
                                              background-color="transparent"
                                              v-model="amountADai">
                                </v-text-field>
                                <v-spacer></v-spacer>
                            </v-row>
                        </v-col>
                    </v-row>
                </v-card-text>
            </v-card>
        </v-row>


        <v-row class="main-btn-row pt-2" justify="center">
            <div style="width: 100%;">
                <v-btn dark
                       height="56"
                       class='buy enabled-buy'
                       @click="start">
                    Start
                </v-btn>
            </div>
        </v-row>

        <v-dialog
          :value="showProcessDialog"
          :width="600"
          @input="close"
          persistent
          scrollable>
            <v-card class="container_body">
                <v-toolbar class="container_header" flat>
                    <v-spacer></v-spacer>
                    <v-toolbar-title class="title">
                        Feeding linear pool&nbsp;&nbsp;
                    </v-toolbar-title>
                    <v-progress-circular v-if="!steps.endStep"
                                         width="2"
                                         size="20"
                                         color="white"
                                         indeterminate
                    ></v-progress-circular>
                    <v-spacer></v-spacer>
                </v-toolbar>

                <v-card-text class="px-5">
                    <v-col class="main-content-col">
                        <v-row align="center" justify="center">
                            <label class="recent-label">
                                Balance DAI: <b>{{ balances.dai }}</b>
                            </label>
                        </v-row>

                        <v-row align="center" justify="center">
                            <label class="recent-label">
                                Balance aDAI: <b>{{ balances.aDai }}</b>
                            </label>
                        </v-row>

                        <v-row align="center" justify="center">
                            <label class="recent-label">
                                Balance StaticADai: <b>{{ balances.staticADai }}</b>
                            </label>
                        </v-row>

                        <v-row align="center" justify="center">
                            <label class="recent-label">
                                Balance Pool LP: <b>{{ balances.lpToken }}</b>
                            </label>
                        </v-row>

                        <v-row class="row mt-5">
                            <v-col cols="10">
                                <div class="step-label">Preparing</div>
                            </v-col>
                            <v-col cols="2">
                                <v-icon color="green" v-if="steps.prepareStep">mdi-check</v-icon>
                                <v-icon color="#8FA2B7" v-else>mdi-dots-horizontal</v-icon>
                            </v-col>
                        </v-row>

                        <v-row class="row">
                            <v-col cols="10">
                                <div class="step-label">Approving DAI</div>
                            </v-col>
                            <v-col cols="2">
                                <v-icon color="green" v-if="steps.approveDai">mdi-check</v-icon>
                                <v-icon color="#8FA2B7" v-else>mdi-dots-horizontal</v-icon>
                            </v-col>
                        </v-row>

                        <v-row class="row">
                            <v-col cols="10">
                                <div class="step-label">Swapping DAI to Static</div>
                            </v-col>
                            <v-col cols="2">
                                <v-icon color="green" v-if="steps.swapDaiToStatic">mdi-check</v-icon>
                                <v-icon color="#8FA2B7" v-else>mdi-dots-horizontal</v-icon>
                            </v-col>
                        </v-row>

                        <v-row class="row">
                            <v-col cols="10">
                                <div class="step-label">Swap static to LP</div>
                            </v-col>
                            <v-col cols="2">
                                <v-icon color="green" v-if="steps.swapStaticToLp">mdi-check</v-icon>
                                <v-icon color="#8FA2B7" v-else>mdi-dots-horizontal</v-icon>
                            </v-col>
                        </v-row>

                        <v-row class="row">
                            <v-col cols="10">
                                <div class="step-label">Approve DAi to LP</div>
                            </v-col>
                            <v-col cols="2">
                                <v-icon color="green" v-if="steps.approveDaiLp">mdi-check</v-icon>
                                <v-icon color="#8FA2B7" v-else>mdi-dots-horizontal</v-icon>
                            </v-col>
                        </v-row>

                        <v-row class="row">
                            <v-col cols="10">
                                <div class="step-label">Swap DAI to LP</div>
                            </v-col>
                            <v-col cols="2">
                                <v-icon color="green" v-if="steps.swapDaiToLp">mdi-check</v-icon>
                                <v-icon color="#8FA2B7" v-else>mdi-dots-horizontal</v-icon>
                            </v-col>
                        </v-row>

                        <v-row class="row">
                            <v-col cols="10">
                                <div class="step-label">Ending process</div>
                            </v-col>
                            <v-col cols="2">
                                <v-icon color="green" v-if="steps.endStep">mdi-check</v-icon>
                                <v-icon color="#8FA2B7" v-else>mdi-dots-horizontal</v-icon>
                            </v-col>
                        </v-row>

                        <v-row class="row">
                            <v-col>
                                <v-btn dark
                                       height="40"
                                       v-if="steps.endStep"
                                       class='buy enabled-buy'
                                       @click="close">
                                    Close
                                </v-btn>
                            </v-col>
                        </v-row>
                    </v-col>
                </v-card-text>
            </v-card>
        </v-dialog>
    </v-col>
</template>

<script>

import {mapActions, mapGetters, mapMutations} from "vuex";
import {ethers} from "ethers";

export default {
    name: "LinearPoolDai",

    components: {},

    data: () => ({
        amountDai: null,
        amountADai: null,
        showProcessDialog: false,
    }),

    computed: {
        ...mapGetters('linearPoolDAI', ['balances', "steps",]
        ),

    },

    created() {

        const provider = new ethers.providers.Web3Provider(window.ethereum, "any");
        console.log('Provider: ' + provider.connection.url);
        provider.send("eth_requestAccounts", []).then(value => {

            let signer = provider.getSigner();
            this.updateBalances(signer);
        });

    },

    methods: {
        ...mapActions('linearPoolDAI', ['startProcess', 'updateBalances']),
        ...mapMutations('linearPoolDAI', ['setAmountValueDai', 'setAmountValueADai']),

        isNumber: function (evt) {
            evt = (evt) ? evt : window.event;
            let charCode = (evt.which) ? evt.which : evt.keyCode;

            if ((charCode > 31 && (charCode < 48 || charCode > 57)) && charCode !== 46) {
                evt.preventDefault();
            } else {
                if (charCode === 46 && (!this.sum || this.sum.includes('.'))) {
                    evt.preventDefault();
                } else {
                    return true;
                }
            }
        },

        start() {
            this.showProcessDialog = true;

            this.setAmountValueDai(this.amountDai);
            this.setAmountValueADai(this.amountADai);
            this.startProcess();
        },

        close() {
            this.amountDai = null;
            this.showProcessDialog = false;

            this.$emit('input', false);
            this.$emit('m-close');
        },
    },
}
</script>

<style scoped>

/* mobile version */
@media only screen and (max-width: 1400px) {
    .buy {
        height: 46px !important;
    }

    .main-btn-row {
        margin-top: 28px;
    }

    .main-content-col {
        padding-left: 16px;
        padding-right: 16px;
    }
}

@media only screen and (min-width: 1400px) {
    .buy {
        height: 56px;
    }

    .main-btn-row {
        margin-top: 40px;
    }

    .main-content-col {
        padding-left: 40px;
        padding-right: 40px;
    }

    .step-label, .recent-label {
        margin-left: 8px;
    }
}

.main-card {
    background: none !important;
    width: 100%;
    border: 1px solid #181E25 !important;
    border-radius: 16px !important;
}

.title-row-label {
    color: white;
    font-style: normal;
    font-weight: 600;
    font-size: 14px;
    line-height: 24px;
}

.v-text-field >>> input {
    font-size: 34px !important;
    font-style: normal !important;
    font-weight: 300 !important;
    min-height: 48px !important;
    max-height: 48px !important;
    height: 48px !important;
}

.v-text-field >>> label {
    font-size: 34px !important;
}

.v-text-field >>> button {
    font-size: 34px !important;
}

.field-sum {
    width: 45%;
}

.enabled-buy {
    background: var(--orange-gradient) !important;
}

.disabled-buy {
    background: #202932 !important;
}

.buy {
    width: 100%;
    border-radius: 40px;
    color: white !important;
}

.container_body {
    border-radius: 24px !important;
    background-color: var(--secondary) !important;
}

.container_header {
    background-color: var(--secondary) !important;
}

.title {
    color: white !important;
    font-weight: 300;
}

.row {
    font-size: 16px;
    line-height: 16px;
}

.step-label {
    color: white !important;
    font-style: normal;
    font-weight: normal;
    font-size: 14px;
    line-height: 18px;
}

.recent-label {
    font-style: normal;
    font-weight: normal;
    font-size: 14px;
    line-height: 18px;
    color: #8FA2B7 !important;
}
</style>
