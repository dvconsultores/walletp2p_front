<template>
  <div id="limited-permissions" class="d-flex flex-column">
    <Header
      top-text="CONECTAR"
      top-text-center
    ></Header>

    <v-btn
      class="btn-outlined mx-auto"
      style="--bg: var(--secondary); --h: 34px; margin-block: 20px 19px;"
    >
      <h5 class="mb-0">{{ shortenAddress(address) }}</h5>
    </v-btn>

    <p class="mb-0" style="--fw: 400">
      Conéctese únicamente a sitios en los que confíe. una vez conectado,
      <a href="" target="_blank">{{ domain }}</a> tendrá
      <b style="font-weight: 700 !important">permisos limitados</b>:
    </p>


    <ul class="d-flex flex-column" style="margin-block: 33px; gap: 10px;">
      <li v-for="(item, i) in checks" :key="i" :class="{ unchecked: !item.check }">
        <span>{{ item.text }}</span>
      </li>
    </ul>


    <v-card
      min-height="45"
      class="btn-outlined justify-space-between align-center flex-wrap"
      style="margin-bottom: 12px; padding-right: 15px;"
    >
      <label>CONTRATO</label>

      <a :href="'https://explorer.testnet.near.org/accounts/'+contract" target="_blank" class="center tend ml-auto" style="gap: 8px;">
        {{ contract }}
        <img src="@/assets/sources/icons/link.svg" alt="link to contract">
      </a>
    </v-card>

    <v-card class="btn-outlined flex-column" style="padding-block: 15px;">
      <div class="d-flex space" style="margin-bottom: 8px;">
        <label>ASIGNACIÓN DE TARIFA DE RED</label>
        <span
          class="tcenter"
          style="--c: #7C7B7F; --fw: 700; --ls: normal; text-align: right; font-size: 12px;"
        >0.25 NEAR</span>
      </div>

      <p class="mb-0" style="--fw: 500">
        La aplicación recibirá permiso para gastar hasta 
        <b style="font-weight: 700 !important">0.25 NEAR</b> 
        hacia las tarifas de red incurridas durante el uso.
      </p>
    </v-card>


    <aside class="d-flex justify-space-between mt-6" style="gap: 12px;">
      <v-btn
        class="btn-outlined flex-grow-1"
        style="--bg: var(--secondary)"
        @click="cancel()"
      >
        CANCELAR
      </v-btn>

      <v-btn
        class="btn flex-grow-1"
        @click="connect()"
      >
        CONECTAR
      </v-btn>
    </aside>
  </div>
</template>

<script>
import utils from '../services/utils';
import encryp from '../services/encryp';
import localStorageUser from '~/services/local-storage-user';
import walletFn from '~/services/wallet';
import { ALERT_TYPE } from '~/plugins/dictionary';

export default {
  name: "LimitedPermissions",
  layout: "default-variant",
  data() {
    return {
      checks: [
        { text: "Ver la dirección de su cuenta permitida", check: true },
        { text: "Ver el saldo de su cuenta permitida", check: true },
        { text: "Métodos de llamada en el contrato inteligente en nombre de su cuenta permitida", check: true },
        { text: "Esto no permite que la aplicación transfiera tokens.", check: false },
      ],
      domain: null,
      contract: null,
      token: {},
      address: sessionStorage.getItem("connectAppAddressSelect"),
      loginNear: false,
    }
  },
  head() {
    const title = 'Connect with NEAR';
    return {
      utils,
      title,
    }
  },
  created() {
    // const token = sessionStorage.getItem("token");
    /* let tokenJSON;
    if(this.$route.query.token){
      // const tokenString = window.atob(this.$route.query.token);
      const tokenString = encryp.decryp(this.$route.query.token);
      tokenJSON = JSON.parse(tokenString);
      // sessionStorage.setItem("token", tokenString);
    }

    
    this.domain = tokenJSON.domain;
    this.contract = tokenJSON.contract;
    this.token = tokenJSON; */

    if(this.$route.query.token){
      const tokenString = encryp.decryp(this.$route.query.token);
      const tokenJSON = JSON.parse(tokenString);
      
      this.domain = tokenJSON.domain;
      this.contract = tokenJSON.contract;
      this.token = tokenJSON;
    } else {
      const params = this.$route.query;
       
      this.domain = this.token.domain = this.$route.query?.success_url ? this.$route.query?.success_url.split("/")[2] : null;
      this.contract = this.token.contract = !params?.contract_id ? undefined : params?.contract_id;
      this.routeCancel = params?.failure_url;
      this.token.success = params?.success_url;
      this.token.public_key = params?.public_key;

      this.loginNear = true;
      
    }
  },
  methods: {
    async connect(){
      if (!this.address || !this.domain) return

      localStorageUser.addApp({
          _address: this.address, 
          _contract: !this.contract ? this.domain : this.contract,
          _domain: this.domain
      });
      
      const account = localStorageUser.getAccount(this.address)
    
      // sessionStorage.removeItem("token");
      // sessionStorage.removeItem("connectAppAddressSelect")
    
      if(this.token?.public_key) {
        const accountNear = await walletFn.nearConnection(this.address)
        await accountNear.addKey(
          this.token.public_key, // public key for new account
          this.contract, // contract this key is allowed to call (optional)
          // "example_method", // methods this key is allowed to call (optional)
          // "2500000000000" // allowance key can use to call methods (optional)
        ).then(() => {
          this.completeLogin(account)
        }).catch((error) => {
          this.$alert(ALERT_TYPE.ERROR, { desc: error.toString() })
        });
      } else { 
        this.completeLogin(account)
      }

    },

    completeLogin(_account) {
      let ruta = this.token.success;

      const json = JSON.stringify({
        wallet: _account.address,
        cretaDate: new Date(),
        email: _account.email,
        privateKey: _account.privateKey,
      })

      const token = window.btoa(json)

      if(this.token?.search) {
        ruta += this.token.search + "&token="+token;
      } else {
        ruta += "?token="+token;
        if(this.loginNear) {

          const callbackUrl = this.$route.query?.success_url.split('?')
          let urlParams = "";

          if(callbackUrl.length > 1) {
            urlParams = new URLSearchParams(callbackUrl[1]);
            urlParams.delete("transactionHashes");
            urlParams.delete("transactions");
            urlParams.delete("token");
          }

          const urlParamsFinal = urlParams.toString().trim() === "" ? "" : `&${urlParams.toString()}`;
          const publicKeyParam = !this.token?.public_key ? "" : `&public_key=${this.token.public_key}`;
          ruta = `${callbackUrl[0]}?account_id=${_account.address}${publicKeyParam}&all_keys=${_account.publicKey}${urlParamsFinal}`;// this.token.success;


          
          // const publicKeyParam = !this.token?.public_key ? "" : `&public_key=${this.token.public_key}`;
          // ruta =  `${this.$route.query?.success_url}?account_id=${_account.address}${publicKeyParam}&all_keys=${_account.publicKey}`;// this.token.success;
        }
      }
      
      location.replace(ruta);
    },

    connect2(){
      if (!this.address || !this.domain) return
      

      localStorageUser.addApp({
          _address: this.address, 
          _contract: this.domain, 
          _domain: this.domain
      });

      const account = localStorageUser.getAccount(this.address)

      let ruta = this.token.success;
      const json = JSON.stringify({
        wallet: account.address,
        cretaDate: new Date(),
        email: account.email,
        privateKey: account.privateKey,
      })
      const token = window.btoa(json)
      
      // sessionStorage.removeItem("token");
      // sessionStorage.removeItem("connectAppAddressSelect")
      
      // const paramsOrigin = this.$route.query?.success_url.split("?").length > 0 ? "&" : "?"
      // const ruta =  `${this.$route.query?.success_url}${paramsOrigin}account_id=${account.address}&all_keys=${account.privateKey}`;// this.token.success;



      if(this.token?.search) {
        ruta += this.token.search + "&token="+token;
      } else {
        ruta += "?token="+token;
      }
      
      location.replace(ruta);
    },
    shortenAddress(address) {
      return utils.shortenAddress(address);
    },
    cancel(){
      if(this.routeCancel) {
        /* let ruta = this.token.error;
          
        if(this.token?.searchError) {
          ruta += this.token.searchError;
        } */
        location.replace(this.routeCancel);
      } else {
        this.$router.push({ path: "/"})
      }
    }

  }
}
</script>

<style src="@/assets/styles/pages/limited-permissions.scss" lang="scss"></style>
