<template>
    <div class="mt-2 ml-5" style="width: 40px; height: 40px">
        <v-menu offset-y>
            <template v-slot:activator="{ on, attrs }">
                <div class="btn-icon"
                     v-click-outside="clickMenuOutside"
                     @click="openedList = !openedList"
                     style="display: flex"
                     v-bind="attrs"
                     v-on="on">
                    <v-img :src="icon"/>
                    <v-icon color="white">
                        {{ openedList ? 'mdi-chevron-up' : 'mdi-chevron-down' }}
                    </v-icon>
                </div>
            </template>
            <v-list class="network-select-list">
                <v-list-item style="cursor: pointer" target="_blank" href="https://app.overnight.fi">
                    <v-list-item-avatar>
                        <div class="list-item-icon">
                            <v-img :src="require('@/assets/network/polygon.svg')"/>
                        </div>
                    </v-list-item-avatar>
                    <v-list-item-title class="network-select-list-item">
                        Polygon
                    </v-list-item-title>
                </v-list-item>
                <v-list-item style="cursor: pointer"  target="_blank" href="https://avax.overnight.fi">
                    <v-list-item-avatar>
                        <div class="list-item-icon">
                            <v-img :src="require('@/assets/network/avalanche.svg')"/>
                        </div>
                    </v-list-item-avatar>
                    <v-list-item-title class="network-select-list-item">
                        Avalanche
                    </v-list-item-title>
                </v-list-item>
                <v-list-item disabled>
                    <v-list-item-avatar>
                        <div class="list-item-icon">
                            <v-img :src="require('@/assets/network/arbitrum.svg')"/>
                        </div>
                    </v-list-item-avatar>
                    <v-list-item-title class="list-item-disabled">
                        Arbitrum (Soon)
                    </v-list-item-title>
                </v-list-item>
                <v-list-item disabled>
                    <v-list-item-avatar>
                        <div class="list-item-icon">
                            <v-img :src="require('@/assets/network/ftm.png')"/>
                        </div>
                    </v-list-item-avatar>
                    <v-list-item-title class="list-item-disabled">
                        Fantom (Soon)
                    </v-list-item-title>
                </v-list-item>
                <v-list-item disabled>
                    <v-list-item-avatar>
                        <div class="list-item-icon">
                            <v-img :src="require('@/assets/currencies/eth.svg')"/>
                        </div>
                    </v-list-item-avatar>
                    <v-list-item-title class="list-item-disabled">
                        Ethereum (Soon)
                    </v-list-item-title>
                </v-list-item>
            </v-list>
        </v-menu>
    </div>
</template>

<script>
import {mapGetters} from "vuex";

let polygonIcon = require('@/assets/network/polygon.svg');
let avaxIcon = require('@/assets/network/avalanche.svg');

export default {
    name: "NetworkSelect",

    components: {},

    data: () => ({
        openedList: false,
    }),

    computed: {
        ...mapGetters('web3', ['networkId']),


        icon: function (){

            if (this.networkId === 137){
                return polygonIcon;
            }else if (this.networkId === 43114) {
                return avaxIcon;
            }

        }
    },

    methods: {
        clickMenuOutside() {
            this.openedList = false;
        }
    }
}
</script>

<style scoped>

.network-select-list {
    background-color: var(--secondary) !important;
    border-radius: 10px;
    margin-top: 10px;
}

.network-select-list-item {
    color: white;
    font-weight: 600;
    font-size: 17px;
    margin-left: -5px;
}

.list-item-icon {
    width: 32px;
    height: 32px;
}

.list-item-disabled {
    color: rgba(255, 255, 255, 0.15);
    font-weight: 600;
    font-size: 17px;
    margin-left: -5px;
}

.btn-icon {
    width: 32px;
    height: 32px;
}
</style>
