<template>
  <div id="trades-chat" class="d-flex flex-column">
    <v-dialog
    v-model="dialog2"
    max-width="max-content"
    :overlay-opacity=".9"
    content-class="modal-cryptos"
    >
      <v-card class="cryptos-card">
        <div
          class="cryptos-card__wrapper"
        >
          <v-card
            color="transparent"
            class="cryptos-card-coin space"
            @click="onSelected(item)"
          >
          <div class="center" style="gap: 14px;">
          <p style="font-size: 14px!important; text-align: justify;" >Pulse el botón "MARCAR PAGO REALIZADO" para culminar la transacción. <br/><br/> Si considera alguna actividad inusual o no ha podido culminar la operación, puede iniciar un proceso de disputa pulsando el botón reportar.</p>
        </div>
          </v-card>
        </div>
      </v-card>
    </v-dialog>

    <modal-warning
      ref="aproveModal"
    >
      <template #action>
        <v-btn class="btn-outlined-2 mt-6" :loading="btnLoading" @click="$refs.aproveModal.model = false">
          VOLVER
        </v-btn>
      </template>

      <p>
        USTED ESTA A PUNTO APROBAR EL INTERCAMBIO, ESTÁ DE ACUERDO CON LA TRANSACCIÓN
      </p>

      <!-- <v-textarea
        placeholder="COLOQUE AQUÍ LA DENUNCIA HACIA LA CONTRAPARTE"
        hide-details solo
        class="flex-grow-0"
        style="--br: 10px; --c-place: #000 !important"
      ></v-textarea> -->

      <v-btn  class="btn mt-2 mb-8" :loading="btnLoading"  @click="aprove" >
        MARCAR PAGO COMO REALIZADO
      </v-btn>

    </modal-warning>

    <modal-warning
      ref="cancelModal"
    >
      <template #action>
        <v-btn class="btn-outlined-2 mt-6" :loading="btnLoading" @click="$refs.cancelModal.model = false">
          VOLVER
        </v-btn>
      </template>

      <p>
        USTED ESTA A PUNTO CANCELAR EL INTERCAMBIO, ESTÁ DE ACUERDO CON LA TRANSACCIÓN
      </p>

      <!-- <v-textarea
        placeholder="COLOQUE AQUÍ LA DENUNCIA HACIA LA CONTRAPARTE"
        hide-details solo
        class="flex-grow-0"
        style="--br: 10px; --c-place: #000 !important"
      ></v-textarea> -->

      <v-btn  class="btn mt-2 mb-8" :loading="btnLoading"  @click="cancel" >
        CANCELAR ORDEN
      </v-btn>

    </modal-warning>
    

    <modal-warning
      ref="warningModal"
    >
      <template #action>
        <v-btn class="btn-outlined-2 mt-6" :loading="btnLoading" @click="$refs.warningModal.model = false">
          VOLVER
        </v-btn>
      </template>

      <p>
        USTED ESTA A PUNTO CANCELAR EL INTERCAMBIO DEBIDO A UN COMPORTAMIENTO INUSUAL EN LA CONTRAPARTE
      </p>

      <!-- <v-textarea
        placeholder="COLOQUE AQUÍ LA DENUNCIA HACIA LA CONTRAPARTE"
        hide-details solo
        class="flex-grow-0"
        style="--br: 10px; --c-place: #000 !important"
      ></v-textarea> -->

      <v-btn  class="btn mt-2 mb-8" :loading="btnLoading"  @click="dispute" >
        INICIAR DISPUTA
      </v-btn>
    </modal-warning>

    <Header hide-append class="mb-4">
      <template #prepend>
        <v-btn class="btn-icon-shadow ml-auto" @click="$router.back()">
          <v-icon size="18">mdi-close</v-icon>
        </v-btn>
      </template>
    </Header>

    <div class="d-flex mb-4" style="gap: 10px;">
      <v-card class="card d-flex align-center px-4 flex-grow-1" style="--bg: var(--card-2)">
        <profile-avatar
          show-name
          :profile="{
            name: 'Intercambio Seguro',
            avatar: require('@/assets/sources/avatars/avatar.png'),
            verified: true,
          }"
        ></profile-avatar>
      </v-card>

      <v-btn
        :disabled="disputeDiabled"
        class="btn"
        style="--bg: var(--card-2); --br: 10px; --f: none; box-shadow: none;"
        @click="$refs.warningModal.model = true"
      >
        <span style="color: var(--primary) !important">DISPUTA</span>
      </v-btn>
    </div>


    <v-card class="card mb-4 px-4 py-2 d-flex justify-space-between" style="--bg: var(--card-2); --br: 10px">
      <h6 class="mb-0" style="--ff: var(--font3); --fs: 16px; translate: 0 2px">{{ montoText }}</h6>
      <span style="font-family: var(--font2); font-weight: 700 !important; font-size: 11px !important; color: #333 !important">
        {{ operationAmount }} {{ tokenSymbol }}
      </span>
    </v-card>
    
    <v-card class="card mb-4 px-4 py-2 d-flex justify-space-between" style="--bg: #BFFBFA; --br: 10px">
      <h6 class="mb-0" style="--ff: var(--font3); --fs: 16px; translate: 0 2px">{{ montoText1 }}</h6>
      <span style="font-family: var(--font2); font-weight: 700 !important; font-size: 11px !important; color: #333 !important">
        {{ fiatSymbol }} {{ formatNumber(receiveAmount) }}
      </span>
    </v-card>


    <p2p-chat-trade class="mb-5"
    ></p2p-chat-trade>

    <v-btn
      :loading="btnLoading" 
      :disabled="disableButtonText"
      class="btn mt-3 mb-4"
      style="--bg: var(--primary); --br: 30px"
      @click="$refs.aproveModal.model = true"
    >{{ buttonText }}</v-btn>

    <!--<v-btn
      :loading="btnLoading" 
      class="btn mt-2 mb-4"
      style="--bg: var(--primary); --br: 30px"
      @click="sendMailDispute"
    >MARCAR PAGO REALIZADO</v-btn>-->

    <v-btn
      v-if="cancelVisible"
      :loading="btnLoading" 
      class="btn-outlined mb-4"
      style="--bg: var(--card-2); --br: 30px"
      @click="$refs.cancelModal.model = true"
    >CANCELAR ORDEN</v-btn>

    <h6 style="--ff: var(--font2); --fw: 700; --fs: 10px; --lh: 1ch">
      APENAS TERMINE DE TRANSFERIR PUEDE HACER  CLICK

      <img src="@/assets/sources/icons/warning-blue.svg" alt="info icon" class="ml-1" style="translate: 0 5px" @click="dialog2 = true">
    </h6>
  </div>
</template>


<script>
import * as nearAPI from "near-api-js";
import axios from 'axios';
// eslint-disable-next-line import/no-named-as-default
import gql from "graphql-tag";
import encrypDecript from "@/services/encryp";
import wallet from '@/services/local-storage-user'
import walletUtils from "@/services/wallet";
// import { mont } from "bn.js";
const { utils } = nearAPI;

export default {
  name: "TradesChatPage",
  data() {
    return {
      address: wallet.getCurrentAccount().address,
      data: [],
      dataTrader: [],
      dataCancel: [],
      terms: "",
      receiveAmount: 0,
      tokenSymbol: "",
      tokenImage: "",
      fiatSymbol: "",
      crypto: "",
      exchangeRate: 0,
      operationAmount: 0,
      dialog2: false,
      btnLoading: false,
      orderId: "",
      polling: null,
      cancelVisible: false,
      disputeDiabled: false,
      operation: "",
      deleteContract: null,
      poolOrders: null,
      poolOrderHistory: null,
      countOrders: 0,
      buttonText: "MARCAR PAGO REALIZADO",
      disableButtonText: false,
      montoText: "MONTO A ENVIAR",
      montoText1: "MONTO A RECIBIR",
      signerId: null,
      ownerId: null,
    }
  },
  head() {
    const title = 'Trades Chat';
    return {
      title,
    }
  },
  beforeDestroy() {
    if(this.poolOrders) {
  		this.poolOrders.unsubscribe();
    }
    if(this.poolOrderHistory) {
      this.poolOrderHistory.unsubscribe();
    }
	},
  mounted() {
    this.getOrders();  
  },
  methods: {
    async trader( ownerId ) {
      const selects = gql`
        query MyQuery( $address : String) {
          datausers(where: {user_id: $address}) {
          name
          last_name
          email
        }
      }
      `;
      // if (sessionStorage.getItem('traderName')) {
      //   this.traderName = sessionStorage.getItem('traderName');
      // } else {
      await this.$apollo
        .watchQuery({
          query: selects,
          // fetchPolicy: 'network-only',
          pollInterval: 5000,
          variables: {
            address: ownerId,
          },
        })
        .subscribe(({ data }) => {
						Object.entries(data.datausers).forEach(([key, value]) => {
            this.dataTrader.push(value);
            this.traderName = this.dataTrader.length > 0 ? this.dataTrader[0].name + ' ' + this.dataTrader[0].last_name : ownerId;
            sessionStorage.setItem('traderName', this.traderName);
          });
        });
      // }
    },
    
    getOrders() {
      const selects = gql`
        query MyQuery( $address : String) {
          ordersells(
            where: {signer_id: $address}
          orderBy: id
          orderDirection: desc
          ) {
          id
          amount_delivered
          asset
          exchange_rate
          fee_deducted
          fiat_method
          owner_id
          payment_method
          signer_id
          terms_conditions
          time
          operation_amount
          order_id
          status
          confirmation_signer_id
        }

        orderbuys(
            where: {signer_id: $address}
          orderBy: id
          orderDirection: desc
          ) {
          id
          amount_delivered
          asset
          exchange_rate
          fee_deducted
          fiat_method
          owner_id
          payment_method
          signer_id
          terms_conditions
          time
          operation_amount
          order_id
          status
          confirmation_signer_id
        }
      }
      `;    
      this.poolOrders = this.$apollo
        .watchQuery({
          query: selects,
          // fetchPolicy: 'network-only',
          pollInterval: 5000,
          variables: {
            address: wallet.getCurrentAccount().address,
          }
        })
        .subscribe(( response ) => {
            if(!response.data?.ordersells || !response.data?.orderbuys) return

            const orderBuys = response.data.orderbuys;
            const orderSells = response.data.ordersells;
            // console.log('orderSells', orderSells.length > 0 ? "SELL" : "BUY")
            const data = orderSells.length > 0 ? orderSells :  orderBuys;
            this.operation = orderSells.length > 0 ? "SELL" : "BUY";

            if(data.length <= 0) {
              /* if(orderId === "undefined" || operation === "undefined" || orderId === undefined || operation === undefined || !orderId || !operation ) {
                this.orderHistory(orderId, operation);
              } */
              localStorage.removeItem('operation');
              sessionStorage.removeItem('orderId');
              
              // this.$router.push('/');
              return
            };

            let orderId = sessionStorage.getItem('orderId');
            let operation = sessionStorage.getItem('operation');
            
            if(orderId === "undefined" || operation === "undefined" || orderId === undefined || operation === undefined || !orderId || !operation ) {
              sessionStorage.setItem('orderId', data[0].order_id);
              sessionStorage.setItem('operation', this.operation);
              orderId = data[0].order_id;
              operation = this.operation;
            }
            
            let order =  data.find((item) => item.order_id === orderId);

            if(order === "undefined" || order === undefined || !order){
              sessionStorage.setItem('orderId', data[0].order_id);
              sessionStorage.setItem('operation', this.operation);
              orderId = data[0].order_id;
              operation = this.operation;
              order =  data.find((item) => item.order_id === orderId);
            }

            this.data.push(order); 
            sessionStorage.setItem('data', this.data.length);
            this.trader(this.data[0].owner_id);
            sessionStorage.setItem('trader', this.data[0].owner_id);
            this.terms = this.data[0].terms_conditions;
            sessionStorage.setItem('terms', this.terms);
            /// //////////////////////////////////////
            this.tokenSymbol = this.data[0].asset;
            this.tokenImage = this.data[0].asset === "USDT" ? "https://nearp2p.com/dv/portal/usdt.svg" : "https://nearp2p.com/dv/portal/near-wallet-icon.svg";
       
            this.fiatSymbol = this.data[0].fiat_method === "1" ? "Bs." : "$" ;
            this.crypto = this.data[0].fiat_method === "1" ? "VES" : "USD" ;
            
            walletUtils.getPrice(this.crypto, this.tokenSymbol).then(price => {
              this.exchangeRate = this.data[0].exchange_rate * price;
              this.receiveAmount = this.tokenSymbol === "NEAR" ? this.yoctoNEARNEAR(this.data[0].operation_amount) * this.exchangeRate: (this.data[0].operation_amount / 1e6) * this.exchangeRate;     
            }); 
            this.operationAmount = this.tokenSymbol === "NEAR" ? this.yoctoNEARNEAR(this.data[0].operation_amount) : (this.data[0].operation_amount / 1e6); 
            this.orderId = this.data[0].order_id;

            this.operation === "SELL" ? this.cancelVisible = false : this.cancelVisible = true;
            this.operation === "true" ? this.disputeDiabled = true : this.disputeDiabled = false;

            this.operation === "SELL" ? this.montoText = "CRYPTO A ENVIAR" : this.montoText = "CRYPTO A RECIBIR";
            this.operation === "BUY" ? this.montoText1 = "MONTO A PAGAR" : this.montoText1 = "MONTO A RECIBIR";

            this.data[0].confirmation_signer_id === 1 ? this.disableButtonText = true : this.disableButtonText = false;
            this.data[0].confirmation_signer_id === 1 ? this.buttonText = "ESPERANDO POR APROBACIÓN" : this.buttonText = "MARCAR PAGO REALIZADO";
            this.signerId = this.data[0].signer_id
            this.ownerId = this.data[0].owner_id

            if(!this.poolOrderHistory) {
              this.orderHistory(this.orderId, this.operation);
            }
        });
    },
    // async orderBuy() {
    //   this.operation = "BUY";
    //   localStorage.setItem('operation', this.operation);
    //   const selects = gql`
    //     query MyQuery( $address : String) {
    //       orderbuys(
    //         where: {signer_id: $address}
    //       orderBy: id
    //       orderDirection: desc
    //       ) {
    //       id
    //       amount_delivered
    //       asset
    //       exchange_rate
    //       fee_deducted
    //       fiat_method
    //       owner_id
    //       payment_method
    //       signer_id
    //       terms_conditions
    //       time
    //       operation_amount
    //       order_id
    //       status
    //     }
    //   }
    //   `;    
    //   await this.$apollo
    //     .watchQuery({
    //       query: selects,
    //       fetchPolicy: 'network-only',
    //       pollInterval: 5000,
    //       variables: {
    //         address: wallet.getCurrentAccount().address,
    //       }
    //     })
    //     .subscribe(({ data }) => {
    //       // Check if data and data.ordersells exist
    //       if (data.orderbuys.length > 0) {
    //         this.data = [];
		// 				Object.entries(data.orderbuys).forEach(([key, value]) => {
		// 						this.data.push(value); 
    //             sessionStorage.setItem('data', this.data.length);
    //             this.trader(this.data[0].owner_id);
    //             this.terms = this.data[0].terms_conditions;
    //             /// //////////////////////////////////////
    //             this.tokenSymbol = this.data[0].asset;
    //             this.tokenImage = this.data[0].asset === "USDT" ? "data:image/svg+xml;base64,PHN2ZyB4bWxucz0naHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmcnIHdpZHRoPSczMicgaGVpZ2h0PSczMic+PGcgZmlsbD0nbm9uZScgZmlsbC1ydWxlPSdldmVub2RkJz48Y2lyY2xlIGN4PScxNicgY3k9JzE2JyByPScxNicgZmlsbD0nIzI2QTE3QicvPjxwYXRoIGZpbGw9JyNGRkYnIGQ9J00xNy45MjIgMTcuMzgzdi0uMDAyYy0uMTEuMDA4LS42NzcuMDQyLTEuOTQyLjA0Mi0xLjAxIDAtMS43MjEtLjAzLTEuOTcxLS4wNDJ2LjAwM2MtMy44ODgtLjE3MS02Ljc5LS44NDgtNi43OS0xLjY1OCAwLS44MDkgMi45MDItMS40ODYgNi43OS0xLjY2djIuNjQ0Yy4yNTQuMDE4Ljk4Mi4wNjEgMS45ODguMDYxIDEuMjA3IDAgMS44MTItLjA1IDEuOTI1LS4wNnYtMi42NDNjMy44OC4xNzMgNi43NzUuODUgNi43NzUgMS42NTggMCAuODEtMi44OTUgMS40ODUtNi43NzUgMS42NTdtMC0zLjU5di0yLjM2Nmg1LjQxNFY3LjgxOUg4LjU5NXYzLjYwOGg1LjQxNHYyLjM2NWMtNC40LjIwMi03LjcwOSAxLjA3NC03LjcwOSAyLjExOCAwIDEuMDQ0IDMuMzA5IDEuOTE1IDcuNzA5IDIuMTE4djcuNTgyaDMuOTEzdi03LjU4NGM0LjM5My0uMjAyIDcuNjk0LTEuMDczIDcuNjk0LTIuMTE2IDAtMS4wNDMtMy4zMDEtMS45MTQtNy42OTQtMi4xMTcnLz48L2c+PC9zdmc+" : "/wallet-arepa/wallet-arepa/assets/sources/logos/near-icon.svg";
    //             // this.tokenImage = selectedToken.icon;
    //             this.fiatSymbol = this.data[0].fiat_method === "1" ? "Bs." : "$" ;
    //             this.crypto = this.data[0].fiat_method === "1" ? "VES" : "USD" ;

    //             walletUtils.getPrice(this.crypto, this.tokenSymbol).then(price => {
    //                this.exchangeRate = this.data[0].exchange_rate * price;
    //                this.receiveAmount = this.tokenSymbol === "NEAR" ? this.yoctoNEARNEAR(this.data[0].operation_amount) * this.exchangeRate: (this.data[0].operation_amount / 1e6) * this.exchangeRate;     
    //             }); 
    //             this.operationAmount = this.tokenSymbol === "NEAR" ? this.yoctoNEARNEAR(this.data[0].operation_amount) : (this.data[0].operation_amount / 1e6);               
    //             this.orderId = this.data[0].order_id;
    //             localStorage.setItem('ord erId', this.orderId)


    //             this.operation === "SELL" ? this.cancelVisible = false : this.cancelVisible = true;
    //             this.operation === "true" ? this.disputeDiabled = true : this.disputeDiabled = false;
    //             this.orderHistoryBuy(this.orderId);

    //         });
    //       }
    //       // } else {
    //       //   // console.log('Busco en el historico')
    //       //   this.orderHistorySell();
    //       // }
    //     });
    // },
    formatNumber(number) {
      return new Intl.NumberFormat('es-VE', { 
        style: 'decimal', 
        minimumFractionDigits: 2, 
        maximumFractionDigits: 2 
      }).format(number);
    },
    async aprove() {
      this.btnLoading = true;
      const val = sessionStorage.getItem('operation') === "SELL" ? "1" : "2";
      const type = sessionStorage.getItem('operation') === "SELL" ? "VENTA" : "COMPRA";
      const CONTRACT_NAME = process.env.VUE_APP_CONTRACT_NAME;
      // const CONTRACT_USDT = process.env.VUE_APP_CONTRACT_NAME_USDT;
      const account = await walletUtils.nearConnection();

      
      const orderConfirmation = await account.functionCall({
        contractId: CONTRACT_NAME,
        methodName: "order_confirmation",
        gas: "180000000000000",
        args: { offer_type: parseInt(val), order_id: parseInt(this.orderId) },
        attachedDeposit: "3"
      }) 
      // console.log("orderConfirmation", orderConfirmation)
      if (!orderConfirmation || orderConfirmation.status.SuccessValue !== "") {
        // console.log("Error confirmando la orden");
        return
      }
      // console.log("orderConfirmation", orderConfirmation)

      const contract = await account.viewFunctionV1(
        CONTRACT_NAME,
        "get_subcontract_type",
        { user_id: this.address }
      );
      if(contract !== 1 && val === "1"){
        this.deleteContract = await account.functionCall({
          contractId: CONTRACT_NAME,
          methodName: "delete_contract",
          gas: "150000000000000",
          args: {},     
        });
      
        // console.log("deleteContract", deleteContract)
        this.sendBotMessage(this.orderId, 5, this.signerId, this.ownerId, type);
        sessionStorage.removeItem('allTokenBalances')

        if (!this.deleteContract || this.deleteContract.status.SuccessValue !== "") {
          console.log("Error borrando el contrato");
        }
      }

      sessionStorage.clear(); // Clear all data from sessionStorage
      this.sendMail('sell', sessionStorage.getItem('orderId'));
      localStorage.removeItem('emailCounter');
      sessionStorage.removeItem('orderId');
      localStorage.removeItem('operation');
      localStorage.removeItem('MessageCounter');
      this.data = [];
      this.dataTrader = [];
      this.dataCancel = [];
      if(this.val === "SELL"){
        this.$router.push({ path: "/tx-executed" });
      } else {
        this.buttonText = "ESPERANDO POR APROBACIÓN";
        this.disableButtonText = true;
        this.$refs.aproveModal.model = false;
      }
      this.btnLoading = false;
    },
    async cancel() {
      this.btnLoading = true;
      const CONTRACT_NAME = process.env.VUE_APP_CONTRACT_NAME;
      const account = await walletUtils.nearConnection();
      const typeDesc = sessionStorage.getItem('operation') === "SELL" ? "VENTA" : "COMPRA";

      const orderConfirmation = await account.functionCall({
        contractId: CONTRACT_NAME,
        methodName: "cancel_order",
        gas: "180000000000000",
        args: { offer_type: 2, order_id: parseInt(this.orderId) },
        attachedDeposit: "3"
      });
      // console.log("orderConfirmation", orderConfirmation)
      if (!orderConfirmation || orderConfirmation.status.SuccessValue !== "") {
        // console.log("Error cancelando la orden");
        return
      }
      // console.log("orderConfirmation", orderConfirmation)
      this.sendBotMessage(this.orderId, 4, this.signerId, this.ownerId, typeDesc);
      this.sendMailCancel(sessionStorage.getItem('orderId'));
      sessionStorage.clear(); // Clear all data from sessionStorage
      sessionStorage.removeItem('orderId');
      localStorage.removeItem('emailCounter');
      localStorage.removeItem('operation');
      localStorage.removeItem('MessageCounter');
      this.btnLoading = false;
      this.data = [];
      this.dataTrader = [];
      this.dataCancel = [];
      // this.sendMail();
      this.$router.push({ path: "/tx-canceled" });
    },
    async dispute() {
      this.btnLoading = true;
      const CONTRACT_NAME = process.env.VUE_APP_CONTRACT_NAME;
      const account = await walletUtils.nearConnection();
      const type = this.operation === "SELL" ? 1 : 2;
      const typeDesc = sessionStorage.getItem('operation') === "SELL" ? "VENTA" : "COMPRA";
      const orderConfirmation = await account.functionCall({
        contractId: CONTRACT_NAME,
        methodName: "order_dispute",
        gas: "40000000000000",
        args: { offer_type: type, order_id: parseInt(this.orderId) },
        attachedDeposit: "3"
      });
      // console.log("orderConfirmation", orderConfirmation)
      if (!orderConfirmation || orderConfirmation.status.SuccessValue !== "") {
        // console.log("Error cancelando la orden");
        return
      }
      // console.log("orderConfirmation", orderConfirmation)
      this.sendBotMessage(this.orderId, 3, this.signerId, this.ownerId, typeDesc);
      this.sendMailDispute(sessionStorage.getItem('orderId'));
      this.btnLoading = false;
      // this.sendMail();
      this.$router.push({ path: "/tx-disputed" });
    },
    orderHistory(porderId, operation) {
      const val = operation === "SELL" ? "1" : "2";
      const selects = gql`
      query MyQuery( $id : String) {
          orderhistorysells(
            where: {id: $id}
          ) {
            status
          }

          orderhistorybuys(
            where: {id: $id}
          ) {
            status
          }
      }
      `;    
      this.poolOrderHistory = this.$apollo
          .watchQuery({
            query: selects,
            // fetchPolicy: 'network-only',
            pollInterval: 5000,
            variables: {
              id: porderId + '|' + val,
            }
          })
          .subscribe(( response ) => {
              if(!response.data?.orderhistorysells || !response.data?.orderhistorybuys) return

              const orderHistorySells = response.data.orderhistorysells;
              const orderHistoryBuys = response.data.orderhistorybuys;
              
              const data = operation === 'SELL' ? orderHistorySells : orderHistoryBuys;

              if(data.length <= 0) return;

              this.dataCancel = [];
            // console.log('History', data.orderhistorysells.length)
              this.dataCancel = [];
              Object.entries(data).forEach(([key, value]) => {
                this.dataCancel.push(value);
                if(this.dataCancel[0].status === 4){
                  sessionStorage.clear(); // Clear all data from sessionStorage
                  localStorage.removeItem('emailCounter');
                  sessionStorage.removeItem('orderId');
                  localStorage.removeItem('operation');
                  this.$router.push('/tx-canceled');
                }
                if(this.dataCancel[0].status === 2 || this.dataCancel[0].status === 3){
                  sessionStorage.clear(); // Clear all data from sessionStorage
                  localStorage.removeItem('emailCounter');
                  sessionStorage.removeItem('orderId');
                  localStorage.removeItem('operation');
                    this.$router.push('/tx-executed');
                }
              }); 
          });
    },
    // async orderHistoryBuy(porderId) {
    //   const val = localStorage.getItem('operation') === "SELL" ? "1" : "2";
    //   const selects = gql`
    //     query MyQuery( $id : String) {
    //       orderhistorybuys(
    //         where: {id: $id}
    //       ) {
    //       status
    //     }
    //   }
    //   `;    
    //   await  this.$apollo
    //       .watchQuery({
    //         query: selects,
    //         fetchPolicy: 'network-only',
    //         pollInterval: 5000,
    //         variables: {
    //           id: porderId + '|' + val,
    //         }
    //       })
    //       .subscribe(({ data }) => {
    //           this.dataCancel = [];
    //           Object.entries(data.orderhistorybuys).forEach(([key, value]) => {
    //             this.dataCancel.push(value);
    //             if(this.dataCancel[0].status === 4){
    //               sessionStorage.clear(); // Clear all data from sessionStorage
    //               localStorage.removeItem('emailCounter');
    //               sessionStorage.removeItem('orderId');
    //               localStorage.removeItem('operation');
    //               this.$router.push('/tx-canceled');
    //             }
    //             if(this.dataCancel[0].status === 2){
    //                 sessionStorage.clear(); // Clear all data from sessionStorage
    //                 localStorage.removeItem('emailCounter');
    //                 sessionStorage.removeItem('orderId');
    //                 localStorage.removeItem('operation');
    //                 this.$router.push('/tx-executed');
    //             }
    //           });
    //       });
    // },
    async sendMailCancel(order) { 
        const data = await walletUtils.verifyWallet();
        const email = data?.data?.email;
        if(this.dataTrader.length>0){
          const emailTrader = await encrypDecript.decryp(this.dataTrader[0].email);
          const BASE_URL_MAIL = process.env.VUE_APP_API_MAIL_URL;
          const MAILCANCEL = `${BASE_URL_MAIL}cancel/`;
          const serviceUrl = `${MAILCANCEL}admin@nearp2p.com/${emailTrader},${email}/${order}`;
          if(emailTrader){   
           await axios.get(serviceUrl);
          }
       }
		},
    async sendMailDispute(order) { 
        const data = await walletUtils.verifyWallet();
        const email = data?.data?.email;
        if(this.dataTrader.length>0){
          const emailTrader = await encrypDecript.decryp(this.dataTrader[0].email);
          const BASE_URL_MAIL = process.env.VUE_APP_API_MAIL_URL;
          const MAILDISPUTE = `${BASE_URL_MAIL}dispute/`;
          const serviceUrl = `${MAILDISPUTE}admin@nearp2p.com/${emailTrader},${email}/${order}/sell`;
          if(emailTrader){   
            await axios.get(serviceUrl);
          }
       }
		},
    async sendMail(typeOffer, order) { 
        const data = await walletUtils.verifyWallet();
        const email = data?.data?.email;
        if(this.dataTrader.length>0){
          const emailTrader = await encrypDecript.decryp(this.dataTrader[0].email);
          const BASE_URL_MAIL = process.env.VUE_APP_API_MAIL_URL;
          const MAIL = `${BASE_URL_MAIL}admin@nearp2p.com/`;
          const serviceUrlTrader = `${MAIL}${emailTrader}/${order}/${typeOffer.toString()}`;
          const serviceUrlCustomer = `${MAIL}${email}/${order}/${typeOffer.toString()}`;
          // console.log(serviceUrl);
          if(emailTrader){   
            await axios.get(serviceUrlTrader);
            await axios.get(serviceUrlCustomer);
          }
       }
		},
    async sendBotMessage(orderId, sTatus, signerId, ownerId, tYpe) {
          await axios.post(process.env.URL_BACKEND +'/api/v1/botp2p/handle_update', 
          { order_id: orderId,
            status: sTatus,
            signer_id: signerId,
            owner_id: ownerId,
            type: tYpe
          }, {
            headers: {
              'accept': 'application/json',
            },
          }).then(() => {
            console.log('Message send')
          }).catch(() => {
            console.log('Error send message')
          })
    },
    // pollData() {
		// 	this.polling = setInterval(() => {
    //     if(this.data.length > 0 && this.data[0].status === 3){
    //         this.topText = "TRANSACCIÓN MARCADA PARA";
    //         this.topBottom = "DISPUTA";
    //         this.stopCountdown();
    //         this.seconds = 0;
    //     }  
    //     if(this.dataCancel.length > 0 && this.dataCancel[0].status === 3){
    //         sessionStorage.clear(); // Clear all data from sessionStorage
    //         localStorage.removeItem('emailCounter')
    //         localStorage.removeItem('orddderId')
    //         this.$router.push('/tx-executed');
    //     }
    //     if(this.dataCancel.length > 0 && this.dataCancel[0].status === 4){
    //         sessionStorage.clear(); // Clear all data from sessionStorage
    //         localStorage.removeItem('emailCounter')
    //         localStorage.removeItem('orddderId')
    //         this.$router.push('/tx-canceled');
    //     }
		// 	}, 5000);
		// },
    yoctoNEARNEAR(yoctoNEAR) {
				const amountInNEAR = utils.format.formatNearAmount(yoctoNEAR);
				// console.log(amountInNEAR);
				return amountInNEAR.toString();
			},
		NEARyoctoNEAR(NEARyocto) {
				const amountInYocto = utils.format.parseNearAmount(NEARyocto);
				// console.log('',amountInYocto);
				return amountInYocto.toString();
			},  
  }
};
</script>

<style src="~/assets/styles/pages/trades-chat.scss" lang="scss" />
