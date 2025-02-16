<script>
import { mapGetters, mapActions } from 'vuex';
import tokenAvatar from 'components/native/TokenAvatar.vue';

export default {
    props: ['showSendDlg', 'coins', 'selectedCoin', 'showSendAmountDlg'],
    components: {
        TokenAvatar: tokenAvatar,
    },
    data() {
        return {
            searchCoinName: '',
        };
    },
    computed: {
        ...mapGetters('account', ['isAuthenticated', 'accountName']),
        searchCoins() {
            return this.availableCoins.filter(coin => (
                coin.name.toLowerCase().includes(this.searchCoinName.toLowerCase()) ||
          coin.symbol.toLowerCase().includes(this.searchCoinName.toLowerCase())
            ));
        },
        availableCoins() {
            let filteredCoins = this.coins.filter(coin => coin.amount > 0);

            if (filteredCoins.findIndex(coin => coin.symbol === 'TLOS' && coin.account === 'eosio.token') === -1) {
                filteredCoins = [{
                    name: 'Telos',
                    symbol: 'TLOS',
                    amount: 0,
                    price: 0,
                    icon: 'telos',
                    network: 'eos',
                }, ...filteredCoins];
            }

            return filteredCoins;
        },
        showEmptyTelos() {
            console.log('showEmptyTelos() this.availableCoins', this.availableCoins);
            console.log('showEmptyTelos() this.availableCoins.findIndex(coin => coin.symbol === TLOS && coin.account === eosio.token)', this.availableCoins.findIndex(coin => coin.symbol === 'TLOS' && coin.account === 'eosio.token'));
            return this.availableCoins.findIndex(coin => coin.symbol === 'TLOS' && coin.account === 'eosio.token') === -1;
        },
        showDlg: {
            get() {
                return this.showSendDlg;
            },
            set(value) {
                this.$emit('update:showSendDlg', value);
            },
        },
    },
    methods: {
        selectCoin(coin) {
            if (coin.amount > 0) {
                this.$emit('update:showSendAmountDlg', true);
                this.$emit('update:selectedCoin', coin);
            }
        },
    },
    watch: {
        showSendDlg: function (val, oldVal) {
            if (val) {
                this.searchCoinName = '';
            } else {
                this.$emit('update:selectedCoin', null);
            }
        },
    },
};
</script>

<template>
<q-dialog
    v-model="showDlg"
    persistent
    maximized
    transition-show="slide-up"
    transition-hide="slide-down"
>
    <div class="main-background native-font-color">
        <div class="dialogPage">
            <div class="dialogPageContent">
                <div class="dialogPageHeading">
                    <div>
                        <q-btn
                            v-close-popup
                            round
                            flat
                            dense
                            class="closebBtn"
                            icon="west"
                        />
                    </div>
                    <div class="text-subtitle1 text-weight-medium text-center">
                        {{$t('components.send')}}
                    </div>
                    <div ></div>
                </div>
                <div class="">
                    <div class="text-subtitle2 text-grey-4 text-center q-pb-md">
                        {{$t('components.select_a_coin')}}
                    </div>
                    <div class="text-center">
                        <q-input
                            v-model="searchCoinName"
                            borderless
                            label-color="white"
                            color="white"
                            placeholder="Search coin"
                            dense
                            input-style="color: white"
                            input-class="text-white"
                        >
                            <template v-slot:append>
                                <img src="~/assets/icons/search.svg" >
                            </template>
                        </q-input>
                        <q-separator dark class="" />
                    </div>
                    <q-list>
                        <div
                            v-for="(coin, index) in searchCoins"
                            :key="`${coin.name}_${index}`"
                        >
                            <div
                                v-if="index === 0 && coin.suggested"
                                header
                                class="text-subtitle1 q-pt-lg q-pb-xs"
                            >
                                {{$t('components.suggested')}}
                            </div>
                            <q-item-label
                                v-if="
                                    index === searchCoins.findIndex((c) => !c.suggested) &&
                                        !coin.suggested
                                "
                                header
                            >All coins</q-item-label>
                            <q-item
                                v-ripple
                                clickable
                                class="list-item"
                                @click="selectCoin(coin)"
                            >
                                <q-item-section avatar>
                                    <q-avatar size="45px" class="q-my-sm">
                                        <div
                                            v-if="coin.symbol === 'TLOS' && coin.network !== 'tevm'"
                                            class="flex absolute full-width full-height"
                                        >
                                            <img
                                                class="flex q-ml-auto q-mt-auto evm-logo"
                                                alt="tEVM"
                                                src='/branding/telos.png'
                                            >
                                        </div>
                                        <div
                                            v-else-if="coin.symbol === 'TLOS' && coin.network === 'tevm'"
                                            class="flex absolute full-width full-height"
                                        >
                                            <img
                                                class="flex q-ml-auto q-mt-auto evm-logo"
                                                alt="tEVM"
                                                src='/branding/telos-scan.png'
                                            >
                                        </div>
                                        <TokenAvatar v-else :token="coin.icon" :avatarSize="45" />
                                    </q-avatar>
                                </q-item-section>
                                <q-item-section class="coin-info">
                                    <div class="text-white text-left display-grid">
                                        <label
                                            class="text-subtitle1 text-weight-medium text-white h-20 self-end wraplabel"
                                        >{{ coin.name }}</label>
                                        <label class="text-subtitle2 text-white wraplabel">{{
                                            coin.symbol
                                        }}</label>
                                    </div>
                                </q-item-section>
                                <q-item-section side>
                                    <div class="text-white text-right display-grid">
                                        <label
                                            class="text-subtitle1 text-weight-medium text-white h-20"
                                        >{{
                                            `${getFixed(coin.amount, 8)} ${coin.symbol}`
                                        }}</label>
                                        <label
                                            class="text-caption text-white"
                                        >${{ getFixed(coin.amount * coin.price, 2) }}</label>
                                    </div>
                                </q-item-section>
                            </q-item>
                        </div>
                    </q-list>
                </div>
            </div>
        </div>
    </div>
</q-dialog>
</template>

<style scoped>
.evm-logo {
    width: 50%;
    height: 50%;
    margin-right: -10%;
    margin-bottom: -5%;
}

.coin-info {
    justify-content: start;
    display: grid
}

/* .list-item {
  border-left: none;
  border-right: none;
}
.display-grid {
  display: grid;
}
.h-20 {
  height: 20px;
}
.wraplabel {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
} */
</style>
