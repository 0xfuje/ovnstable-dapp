<template>
    <v-dialog
            :value="showDeposit"
            :width="width"
            @input="close"
            :persistent="persistent"
            scrollable>
        <v-card class="container_body">
            <v-toolbar class="container_header" flat>
                <v-row>
                    <v-spacer></v-spacer>
                    <label class="title-modal ml-4">
                        Deposit
                    </label>
                    <v-spacer></v-spacer>
                    <v-btn icon class="ml-auto" @click="close" dark>
                        <v-icon>mdi-close</v-icon>
                    </v-btn>
                </v-row>
            </v-toolbar>

            <v-card-text class="px-5 pt-2">
                <v-row>
                    <v-col class="ml-4 mr-4">
                        <v-row align="center" class="mt-1 mb-2">
                            <div style="display: flex">
                                <div class="currency-icon">
                                    <v-img :src="selectedPool.poolData.token0Icon"/>
                                </div>
                                <div class="currency-icon ml-1">
                                    <v-img :src="selectedPool.poolData.token1Icon"/>
                                </div>
                            </div>

                            <label class="pool-title-label ml-2" v-if="selectedPool && selectedPool.poolData">
                                {{ selectedPool.poolData.title }}
                            </label>
                        </v-row>
                    </v-col>
                </v-row>

                <v-row class="input-row mx-1">
                    <v-col cols="10" class="mt-2 mb-2">
                        <v-row align="center" justify="start">
                            <v-text-field placeholder="0.00"
                                          @keypress="isNumber($event)"
                                          flat
                                          solo
                                          class="ml-2 field-sum"
                                          hide-details
                                          dark
                                          background-color="transparent"
                                          v-model="sum">
                            </v-text-field>
                        </v-row>

                        <v-row class="balance-row">
                            <label class="balance-label ml-5 mb-1 mt-2">Balance LP token: {{selectedPool.userData.lpTokensBalance}}</label>
                        </v-row>
                    </v-col>

                    <v-col cols="2" class="mt-2 mb-2">
                        <v-row align="center" justify="end" style="height: 72px">
                            <div>
                                <label class="max mr-5" @click="max">Max</label>
                            </div>
                        </v-row>
                    </v-col>
                </v-row>

                <v-row class="mt-12 mb-2">
                    <v-col class="ml-4 mr-4">
                        <v-row align="center">
                            <PromoSection/>
                        </v-row>
                    </v-col>
                </v-row>

                <v-row>
                    <v-col class="ml-4 mr-4">
                        <v-row align="center" class="mb-2">
                            <v-btn dark
                                   v-if="!lpApproved"
                                   height="56"
                                   @click="approveAction"
                                   :disabled="!isBuy"
                                   :class="isBuy ? 'enabled-buy' : 'disabled-buy'"
                                   class="buy">
                                Approve LP
                            </v-btn>
                            <v-btn dark
                                   v-else
                                   height="56"
                                   @click="depositAction"
                                   :disabled="!isBuy"
                                   :class="isBuy ? 'enabled-buy' : 'disabled-buy'"
                                   class="buy">
                                Deposit LP
                            </v-btn>
                        </v-row>
                    </v-col>
                </v-row>
            </v-card-text>
        </v-card>

        <WaitingModal/>
        <SuccessModal/>
        <ErrorModal/>
    </v-dialog>
</template>

<script>
import {mapActions, mapGetters, mapMutations} from "vuex";
import ItemSelector from "../../common/ItemSelector";
import ErrorModal from "@/components/common/ErrorModal";
import WaitingModal from "@/components/common/WaitingModal";
import SuccessModal from "@/components/common/SuccessModal";
import PromoSection from "@/components/farm/section/PromoSection";

export default {
    name: "DepositModal",

    components: {
        PromoSection,
        ItemSelector,
        ErrorModal,
        WaitingModal,
        SuccessModal,
    },

    props: {
        persistent: {
            type: Boolean,
            default: false,
        },

        width: {
            type: String,
            default: '500',
        }
    },

    computed: {
        ...mapGetters('farmData', ['selectedPool']),
        ...mapGetters('farmUI', ['showDeposit', 'lpApproved']),
        ...mapGetters("web3", ["web3"]),
        ...mapGetters('accountData', ['account']),

        isBuy: function () {
            return this.account && this.sum > 0 && this.numberRule;
        },

        numberRule: function () {

            let v = this.sum;

            if (!v)
                return false;

            if (!v.trim()) return false;

            v = parseFloat(v.trim().replace(/\s/g, ''));

            if (!isNaN(parseFloat(v)) && v >= 0 && v <= parseFloat(this.selectedPool.userData.lpTokensBalance)) return true;

            return false;
        },
    },

    data: () => ({
        sum: null,
    }),

    created() {
    },

    methods: {
        ...mapActions('farmUI', ['hideDepositModal', 'approveLP']),
        ...mapActions("gasPrice", ['refreshGasPrice']),
        ...mapActions('transaction', ['putTransaction']),

        ...mapActions("errorModal", ['showErrorModal']),
        ...mapActions("waitingModal", ['showWaitingModal', 'closeWaitingModal']),
        ...mapActions("successModal", ['showSuccessModal']),

        max() {
            this.sum = this.selectedPool.userData.lpTokensBalance + "";
        },

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

        async approveAction() {

            console.log('Call deposit approve');
            this.showWaitingModal('Approving in process');

            try {
                let approveSum = 10000000;

                let sum = approveSum * 10 ** 6;
                sum = Math.floor(sum);

                let allowApprove = await this.checkAllowance(sum);

                if (!allowApprove) {
                    this.closeWaitingModal();
                    this.showErrorModal('approve');
                    return;
                } else {
                    await this.approveLP();
                    this.closeWaitingModal();
                }
            } catch (e) {
                console.log(e)
                this.showErrorModal('approve');
            }
        },

        async checkAllowance(sum) {

            let from = this.account;
            let self = this;
            let token = this.selectedPool.token;
            let contract = this.selectedPool.contract;

            let allowanceValue = await token.methods.allowance(from, contract.options.address).call();

            if (allowanceValue < sum) {
                try {
                    await this.refreshGasPrice();
                    let approveParams = {gasPrice: this.gasPriceGwei, from: from};

                    let tx = await token.methods.approve(contract.options.address, this.sum * 10 ** 18).send(approveParams);

                    let done = true;
                    while (done) {
                        await new Promise(resolve => setTimeout(resolve, 2000));
                        let receipt = await this.web3.eth.getTransactionReceipt(tx.transactionHash);

                        if (receipt) {
                            if (receipt.status) {
                                let tx = {
                                    text: 'Approve LP',
                                    hash: receipt.transactionHash,
                                    pending: true,
                                };

                                self.putTransaction(tx);

                                return true;
                            } else {
                                return false;
                            }
                        }
                    }

                    return true;
                } catch (e) {
                    console.log(e)
                    return false;
                }
            }

            return true;
        },

        async depositAction() {

            console.log('Call deposit');
            this.showWaitingModal('Depositing ' + this.sum + ' LP tokens');

            try {
                let from = this.account;
                let self = this;

                let pool = this.selectedPool.contract;

                await this.refreshGasPrice();
                let params = {gasPrice: this.gasPriceGwei, from: from};

                let buyResult = await pool.methods.stake(this.sum * 10 ** 18).send(params).on('transactionHash', function (hash) {
                    let tx = {
                        text: `Stake ${self.sum} LP tokens`,
                        hash: hash,
                        pending: true,
                    };
                    self.putTransaction(tx);
                });

                this.closeWaitingModal();
                await this.close();
                this.showSuccessModal(buyResult.transactionHash);
                this.sum = null;
            } catch (e) {
                console.log(e)
                this.showErrorModal('depositLP');
            }
        },

        close() {
            this.hideDepositModal();

            this.$emit('input', false);
            this.$emit('m-close');
        },
    },
}
</script>

<style scoped>

/* mobile */
@media only screen and (max-width: 1400px) {
}

@media only screen and (min-width: 1400px) {
}


.container_body {
    border-radius: 24px !important;
    background-color: var(--secondary) !important;
}

.container_header {
    background-color: var(--secondary) !important;
}

.v-text-field >>> input {
    min-height: 42px !important;
    max-height: 42px !important;
    height: 42px !important;
}

.v-text-field >>> input::placeholder {
    color: #353E4C !important;
}

.v-text-field >>> input, .v-text-field >>> label, .v-text-field >>> button {
    font-family: 'Lato', sans-serif;
    font-style: normal !important;
    font-weight: 700 !important;
    font-size: 34px !important;
    line-height: 42px !important;

    color: #FE7F2D !important;
}

.field-sum {
    width: 45%;
}

.balance-label {
    color: white;
    font-family: 'Lato', sans-serif;
    font-style: normal;
    font-weight: 400;
    font-size: 14px;
    line-height: 24px;
}

.max {
    color: var(--link);
    font-style: normal;
    font-weight: 600;
    font-size: 14px;
    line-height: 24px;
    cursor: pointer;
}

.balance-row {
    margin-top: -12px !important;
}

.enabled-buy {
    background: var(--orange-gradient) !important;
}

.disabled-buy {
    background: #181A21 !important;
    box-shadow: none !important;
    border: 1px solid #1D2029 !important;
}

.buy {
    width: 100%;
    border-radius: 40px;
    color: white !important;
    font-family: 'Lato', sans-serif;
    font-style: normal;
    font-weight: 600;
    font-size: 16px !important;
    line-height: 24px !important;
    text-transform: none !important;
}

.input-row {
    border: 1px solid #13151C;
    border-radius: 16px;
}

.currency-icon {
    width: 40px;
    height: 40px;
}

.pool-title-label {
    font-family: 'Lato', sans-serif;
    font-style: normal;
    font-weight: 600;
    font-size: 16px;
    line-height: 24px;
    font-feature-settings: 'pnum' on, 'lnum' on;
    color: #FFFFFF;
}
</style>
