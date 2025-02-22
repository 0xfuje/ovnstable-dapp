<template>
    <v-dialog
            :value="showAccountProfile"
            :width="width"
            @input="close"
            :persistent="persistent"
            scrollable>
        <v-card class="container_body">
            <v-toolbar class="container_header" flat>
                <v-row>
                    <label class="title-modal ml-6">
                        Account
                    </label>
                    <v-spacer></v-spacer>
                    <v-btn icon class="ml-auto" @click="close" dark>
                        <v-icon>mdi-close</v-icon>
                    </v-btn>
                </v-row>
            </v-toolbar>

            <v-card-text class="px-5 pt-5">
                <v-col class="main-content-col">
                    <v-row align="center" class="account-info-row connected-row">
                        <label class="wallet-name-label">
                            Connected with {{ walletName }}
                        </label>

                        <v-spacer class="disconnect-wallet-spacer"></v-spacer>

                        <v-btn class="disconnect-wallet-btn" dark @click="disconnectWalletAction">Disconnect</v-btn>
                        <v-btn class="disconnect-wallet-btn-mobile" icon color="red" dark @click="disconnectWalletAction">
                            <v-icon>
                                mdi-logout
                            </v-icon>
                        </v-btn>
                    </v-row>

                    <v-row align="center" class="account-info-row balance-row">
                        <label class="wallet-name-label">
                            Balance: <strong>{{ $utils.formatMoney(balance.usdPlus, 2) }}</strong>&nbsp;USD+
                        </label>
                    </v-row>

                    <v-row align="center" class="account-info-row account-num-row mt-6">
                        <div class="avatar-img">
                            <v-img :src="require('@/assets/network/' + networkName + '.svg')"/>
                        </div>
                        <label class="account-label ml-5">{{ accountShort }}</label>
                    </v-row>

                    <v-row align="center" class="account-info-row actions-row mt-6">
                        <a class="view-explorer-btn" @click="viewInExplorer">
                            <v-img class="icon-img" :src="require('@/assets/icon/out.svg')"/>
                            <label class="ml-1 view-explorer-label">View on explorer</label>
                        </a>

                        <v-tooltip
                                v-model="showCopyTooltip"
                                color="#202832"
                                bottom
                        >
                            <template v-slot:activator="{on}">
                                <a class="copy-address-btn" @click="copyAddressToClipboard">
                                    <v-img class="icon-img" :src="require('@/assets/icon/link.svg')"/>
                                    <label class="ml-1">Copy Address</label>
                                </a>
                            </template>
                            <p class="my-0">Copied!</p>
                        </v-tooltip>

                        <a class="add-usd-plus-btn" @click="addUsdPlusToken">
                            <label class="add-usd-btn">Add USD+</label>
                        </a>
                    </v-row>

                    <v-row class="recent-tr-row mb-3" align="center" v-if="transactions && transactions.length > 0">
                        <label class="recent-label">
                            Recent Transactions
                        </label>
                        <v-spacer></v-spacer>
                        <v-btn class="disconnect-wallet-btn" dark @click="clearTransaction">
                            Clear all
                        </v-btn>
                    </v-row>

                    <v-row class="row mt-2" v-bind:key="i" v-for="(item, i) in transactions" v-if="transactions && transactions.length > 0">
                        <v-col cols="10">
                            <div class="transaction-link" @click="openOnExplorer(item.hash)">{{ item.text }} ↗</div>
                        </v-col>
                        <v-col cols="2">
                            <v-icon v-if="item.pending">mdi-progress-question</v-icon>
                            <v-icon color="green" v-else>mdi-check</v-icon>
                        </v-col>
                    </v-row>
                </v-col>
            </v-card-text>
        </v-card>
    </v-dialog>
</template>

<script>
import {mapActions, mapGetters} from "vuex";

export default {
    name: "AccountProfile",

    data: () => ({
        showCopyTooltip: false,
    }),

    props: {
        persistent: {
            type: Boolean,
            default: false,
        },

        value: {
            type: Boolean,
            default: false,
        },

        width: {
            type: String,
            default: '500',
        }
    },

    computed: {
        ...mapGetters('web3', ['walletName']),
        ...mapGetters('accountUI', ['showAccountProfile']),
        ...mapGetters('transaction', ['transactions']),
        ...mapGetters('accountData', ['balance', 'account']),

        accountShort: function () {
            if (this.account) {
                return this.account.substring(0, 5) + '...' + this.account.substring(this.account.length - 4);
            } else {
                return null;
            }
        },

        networkName() {
            return process.env.VUE_APP_POLYGON;
        },
    },


    watch: {

        showAccountProfile: function (newValue, oldValue){

            console.log('Watch: show ' + newValue);
            if (newValue){
                this.loadTransaction();
            }

        },
    },

    methods: {
        ...mapActions('accountUI', ['hideAccountProfile']),
        ...mapActions('web3', ['disconnectWallet', 'addUsdPlusToken']),
        ...mapActions('transaction', ['clearTransaction', 'loadTransaction']),

        openOnExplorer(hash) {
            window.open(process.env.VUE_APP_NETWORK_EXPLORER + `tx/${hash}`, '_blank').focus();
        },

        disconnectWalletAction() {
            this.disconnectWallet();
            this.close();
        },

        close() {
            this.hideAccountProfile();

            this.$emit('input', false);
            this.$emit('m-close');
        },

        viewInExplorer() {
            let url = process.env.VUE_APP_NETWORK_EXPLORER + 'address/' + this.account;
            window.open(url, '_blank');
        },

        async copyAddressToClipboard() {
            this.showCopyTooltip = true;

            await navigator.clipboard.writeText(this.account);

            await new Promise(resolve => setTimeout(resolve, 1000));
            this.showCopyTooltip = false;
        },
    },
}
</script>

<style scoped>

/* mobile */
@media only screen and (max-width: 1400px) {
    .main-content-col {
        padding-left: 16px;
        padding-right: 16px;
    }

    .disconnect-wallet-btn, .disconnect-wallet-spacer {
        display: none !important;
    }

    .copy-address-btn, .view-explorer-btn-btn, .add-usd-plus-btn {
        margin-left: 30px;
        margin-right: 30px;
        margin-top: 4px;
        margin-bottom: 4px;
    }

    .actions-row {
        margin-top: 24px;
        margin-bottom: 24px;
    }

    .balance-row {
        margin-top: 8px;
        margin-bottom: 8px;
    }

    .account-num-row {
        margin-bottom: 8px;
    }

    .account-info-row {
        justify-content: center !important;
    }

    .recent-tr-row {
        margin-top: 48px;
    }
}

@media only screen and (min-width: 1400px) {
    .main-content-col {
        padding-left: 40px;
        padding-right: 40px;
    }

    .disconnect-wallet-btn-mobile, .balance-row {
        display: none !important;
    }

    .copy-address-btn, .account-action-btn {
        margin-left: 20px;
    }

    .recent-tr-row {
        margin-top: 40px;
    }

    .recent-label, .transaction-link, .recent-label {
        margin-left: 8px;
    }

    .add-usd-plus-btn {
        margin-left: 20px;
    }
}

.row {
    font-size: 17px;
}

.title {
    color: white;
    font-weight: 300;
}

.recent-label {
    font-style: normal;
    font-weight: normal;
    font-size: 14px;
    line-height: 18px;
    color: #8FA2B7 !important;
}

.wallet-name-label {
    color: white !important;
    font-style: normal;
    font-weight: 600;
    font-size: 16px;
    line-height: 24px;
}

.account-label {
    color: white !important;
    font-style: normal;
    font-weight: 600;
    font-size: 14px;
    line-height: 24px;
}

.view-explorer-btn > label, .copy-address-btn > label, .add-usd-btn > label {
    cursor: pointer !important;
}

.view-explorer-btn {
    cursor: pointer;
    background: none !important;
    color: var(--link);
    font-weight: 600;
    font-size: 14px;
    line-height: 24px;
    display: flex;
}

.view-explorer-label:hover {
    text-decoration: underline;
}

.copy-address-btn {
    cursor: pointer;
    background: none !important;
    color: white;
    font-weight: 600;
    font-size: 14px;
    line-height: 24px;
    display: flex;
}

.add-usd-btn {
    cursor: pointer;
    color: white;
    font-weight: 600;
    font-size: 14px;
    line-height: 24px;
    background: var(--orange-gradient);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
}

.disconnect-wallet-btn {
    height: 40px !important;
    background: none !important;
    border: 1px solid var(--link);
    border-radius: 40px;
    text-transform: none !important;
    color: var(--link);
    font-weight: 600;
    font-size: 14px;
    line-height: 24px;
}

.avatar-img {
    width: 40px;
    height: 40px;
}

.icon-img {
    width: 24px;
    height: 24px;
}

.account-info-row {
    height: 56px;
}

.container_body {
    border-radius: 24px !important;
    background-color: var(--secondary) !important;
}

.container_header {
    background-color: var(--secondary) !important;
}

.transaction-link {
    cursor: pointer;
    color: var(--link);
    font-style: normal;
    font-weight: normal;
    font-size: 14px;
    line-height: 18px;
}

.transaction-link:hover {
    text-decoration: underline;
}
</style>
