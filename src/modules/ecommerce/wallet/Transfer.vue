<template>
  <div>
    <div class="modal fade" id="transferFunds" tabindex="-1" role="dialog" aria-labelledby="transferHead" aria-hidden="true">
      <div :class="['modal-dialog', {'modal-md': !next}]">
        <div class="modal-content">
          <div class="modal-header">
            <i class="fa fa-arrow-left" @click="back()" v-if="next"></i>
            <h5 class="modal-title"></h5>
            <button type="button" class="close" aria-label="Close" @click="hide()">
              <span aria-hidden="true">&times;</span>
            </button>
          </div>
          <span v-if="successMessage !== null" class="text-primary text-center incre-row">
            <i class="fa fa-check" style="font-size: 75px; color: #FF5B04;"></i>
            <label class="incre-row" style="color: #FF5B04;">{{successMessage}}</label>
          </span>
          <center>
            <button type="button" style="width:20%; margin-bottom: 3%; font-weight:bold;" class="btn btn-primary" @click="hideFinal()" v-if="successMessage !== null">Ok</button>
          </center> 
          <div class="modal-header" v-if="!next && successMessage === null">
            <h2 class="text-center font-weight-lighter col-12" id="transferHead">Transfer</h2>
          </div>
          <div class="modal-header" v-if="next && successMessage === null">
            <h2 class="text-center font-weight-lighter col-12" id="transferHead">Transfer to {{selectedBank.name}}</h2>
          </div>
          <div class="modal-body px-5 row m-0" v-if="!next && successMessage === null">
            <div class="container-fluid col-12" v-if="balance !== null">
              <b class="ml-1">Pick your currency</b>
              <div v-if="balance.length > 1" class="row mb-0 mt-3 mx-3 flex-column" >
                <div style="text-align: center" class="col-12 py-2 form-group bank" v-for="(item, index) in balance.filter(bal => bal.balance > 0)" :key="index">
                  <center>
                    <input type="radio" name="currency" :id="'currency-'+index" :value="item" v-model="selectedCurrency">
                    <label :for="'currency-'+index" class="ml-2 mb-0">
                        <b>{{item.currency}}</b>
                    </label>
                  </center>
                </div>
              </div>
              <div v-if="balance.length <= 1" >
                <div style="text-align: center" class="col-12 py-2 form-group bank" v-for="(item, index) in balance.filter(bal => bal.balance > 0)" :key="index">
                  <center>
                    <input type="radio" name="currency" :id="'currency-'+index" :value="item" v-model="selectedCurrency">
                    <label :for="'currency-'+index" class="ml-2 mb-0">
                        <b>{{item.currency}}</b>
                    </label>
                  </center>
                </div>
              </div>
            </div>

            <div class="container-fluid col-12">
              <small class="ml-1">Typically in 1-4 business days <b>(Fees may apply)</b></small>
              <div class="row mb-3 mt-3 mx-3 flex-column">
                <div class="col-12 py-2 form-group row mx-0 align-items-center bank">
                  <select class="form-control form-control" v-model="selectedBank">
                    <option v-for="(item, index) in banks" :key="index" :value="item">{{item.name}}&nbsp;&nbsp;&nbsp;&nbsp;{{item.number}}</option>
                  </select>
                  <!-- <input type="radio" name="bank" :id="'bank-'+index" :value="item" v-model="selectedBank">
                  <label :for="'bank-'+index" class="ml-2 mb-0 row align-items-center">
                    <div class="icon text-center mr-2">
                      <h3 class="fa fa-university"></h3>
                    </div>
                    <div>
                      <b>{{item.name}}</b><br>
                      <small class="text-muted">{{item.number}}</small>
                    </div>
                  </label> -->
                </div>
              </div>
            </div>

            <div class="container-fluid row mb-4 mt-1">
              <button type="button" @click="setAmount()" id="next" class="btn btn-primary rounded-pill col-10 py-3 mx-auto">Next</button>
            </div>
          </div>

          <div class="modal-body px-5 row m-0" v-if="next && successMessage === null">
            <div class="container-fluid col-12 row m-0">
              <input type="number" name="amount" id="amount" :disabled="!requestWithdrawal" v-model="amount" autocomplete="off">
              <label for="amount" id="hide" class="mx-auto pr-2" :currency="selectedCurrency.currency">{{currency.displayWithCurrency(amount, selectedCurrency.currency)}}</label>
              <div class="col-12 mt-2 text-center" v-if="!showFee">
                <p style="margin-bottom: 0rem !important"><b>Processing Fee:</b>
                {{currency.displayWithCurrency(0, selectedCurrency.currency)}}</p>
              </div>
              <div class="col-12 mt-3 text-center" style="font-size: 1.2rem">
                <b>Available Balance</b> 
                <br>
                {{currency.displayWithCurrency(selectedCurrency.balance, selectedCurrency.currency)}}
              </div>
              <button class="btn btn-block mt-4 mb-4 mx-auto w-75 rounded-pill py-3 btn-primary" v-if="!transferred" id="next" @click="show_Fee()">Continue</button>
              <button class="btn btn-block mt-4 mb-4 mx-auto w-75 rounded-pill py-3 btn-primary" v-else id="next" @click="transfer()">Request Withdrawal</button>
              <!-- <button class="btn btn-block mt-4 mb-4 mx-auto w-75 rounded-pill py-3 btn-outline-success" v-else id="next" @click="hide()"><i class="fas fa-check mr-1"></i>Request Sent</button> -->
            </div>
          </div>
        </div>
      </div>
    </div>
    <otp ref="otp"></otp>
  </div>
</template>
<style lang="scss" scoped>
  @import "~assets/style/colors.scss";

  #transfer-p2 {
    display: none;
  }

  #amount {
    position: absolute;
    pointer-events: none;
    top: 0;
    z-index: -100;
  }
  
  #hide {
    position: relative;
    font-size: 40px;
    border-bottom: 1px solid transparent;
  }

  #hide::before {
    position: absolute;
    right: 0;
    content: '|';
    opacity: 0;
  }

  #hide::after {
    position: absolute;
    left: 100%;
    font-size: 16px;
  }

  #amount:focus ~ #hide {
    border-bottom-color: black;
  }

  #amount:focus ~ #hide::before {
    animation: 400ms blink alternate infinite;
  }

  @keyframes blink {
    0% {
      opacity: 0;
    }
    100% {
      opacity: 1;
    }
  }

  #hide span {
    font-size: 14px;
  }

  .btn.btn-primary {
    height: 50px !important;
  }

  .modal-header {
    border: none !important;
  }

  .font-weight-lighter {
    font-weight: lighter;
  }

  .icon {
    position: relative;
    height: 25px;
    width: 30px;
  }

  // .row .bank {
  //   border-bottom: 1px dashed #ccc;
  // }

  // .row .bank:first-child {
  //   border-top: 2px solid #ccc;
  // }

  // .row .bank:last-child {
  //   border-bottom: 2px solid #ccc;
  // }

  .modal-dialog {
    transition: 1000ms width ease-in-out;
    width: 450px;
  }

  small {
    font-size: 95%;
  }
  
  .ml-1 {
    text-align: center;
  }

  select.form-control {
    height: 50px !important;
  }

  .col-12 {
    position: relative;
    width: 100%;
    min-height: 1px;
    padding-right: 6px;
    padding-left: 8px;
    text-align: center;
  }
  
  @media (max-width: 750px) {
    .col-12 {
      position: relative;
      width: 100%;
      min-height: 1px;
      padding-right: 6px;
      padding-left: 8px;
      text-align: center;
    }
  }
</style>
<script>
import ROUTER from 'src/router'
import AUTH from 'src/services/auth'
import COMMON from 'src/common.js'
import CONFIG from 'src/config.js'
import CURRENCY from 'src/services/currency.js'
export default {
  mounted(){
    if(!this.user || this.user.type === 'USER') {
      // ROUTER.push('/featured')
      ROUTER.push('/marketplace')
    }
  },
  data(){
    return {
      successMessage: null,
      user: AUTH.user,
      common: COMMON,
      config: CONFIG,
      currency: CURRENCY,
      amount: 0,
      transferred: false,
      next: false,
      showFee: true,
      requestWithdrawal: true,
      selectedBank: null,
      selectedCurrency: null,
      banks: [
        {id: 12, name: 'GCash Main', type: 'gcash', number: '09668816743'},
        {id: 24, name: 'Landbank', type: 'bank', number: '4153-2842-12'},
        {id: 32, name: 'BDO', type: 'bank', number: '1482-1928-23'}
      ]
    }
  },
  props: {
    balance: {
      required: true
    }
  },
  components: {
    'otp': require('src/modules/ecommerce/wallet/OtpWithdraw.vue')
  },
  methods: {
    retrieve(){
    },
    back(){
      $('#transferFunds .error').remove()
      this.next = false
      this.showFee = true
      this.transferred = false
      this.requestWithdrawal = true
      this.amount = 0
    },
    show_Fee(){
      $('#transferFunds .error').remove()
      if(this.amount === 0 || this.amount === null || this.amount === '') {
        $('<div>', {
          class: 'text-danger col-12 pl-3 mb-3 error text-center',
          html: 'Please enter an amount.'
        }).insertBefore('#transferFunds #next')
      } else if(this.amount > this.selectedCurrency.balance) {
        $('<div>', {
          class: 'text-danger col-12 pl-3 mb-3 error text-center',
          html: 'Your balance is not enough.'
        }).insertBefore('#transferFunds #next')
      } else {
        this.showFee = false
        this.transferred = true
        this.requestWithdrawal = false
      }
    },
    show(){
      $('#transferFunds .error').remove()
      this.requestWithdrawal = true
      this.amount = 0
      this.next = false
      this.showFee = true
      this.transferred = false
      if(this.balance.length <= 1){
        this.selectedCurrency = this.balance[0]
      }
      $('#transferFunds').modal('show')
    },
    hide() {
      $('#transferFunds .error').remove()
      this.successMessage = null
      this.selectedBank = null
      this.selectedCurrency = null
      this.next = false
      this.transferred = false
      this.showFee = true
      this.amount = 0
      $('#transferFunds').modal('hide')
    },
    hideFinal(){
      $('#transferFunds .error').remove()
      this.successMessage = null
      this.selectedBank = null
      this.selectedCurrency = null
      this.next = false
      this.transferred = false
      this.showFee = true
      this.amount = 0
      $('#transferFunds').modal('hide')
      ROUTER.push('/withdrawalHistory')
    },
    setAmount() {
      $('#transferFunds .error').remove()
      if(!this.selectedCurrency) {
        $('<div>', {
          class: 'text-danger col-8 pl-3 ml-3 mb-3 error',
          html: 'Please choose a currency for transfer.'
        }).insertBefore('#transferFunds #next')
      } else if(!this.selectedBank){
        $('<div>', {
          class: 'text-danger col-8 pl-3 ml-3 mb-3 error',
          html: 'Please choose a bank for transfer.'
        }).insertBefore('#transferFunds #next')
      } else {
        this.next = true
      }
    },
    transfer() {
      $('#transferFunds .error').remove()
      if(this.amount === 0 || this.amount === null) {
        $('<div>', {
          class: 'text-danger col-12 pl-3 mb-3 error text-center',
          html: 'Please enter an amount.'
        }).insertBefore('#transferFunds #next')
      } else if(this.amount > this.selectedCurrency.balance) {
        $('<div>', {
          class: 'text-danger col-12 pl-3 mb-3 error text-center',
          html: 'Your balance is not enough.'
        }).insertBefore('#transferFunds #next')
      } else {
        $('#loading').css({display: 'block'})
        let par = {
          amount: this.amount,
          account_id: this.user.userID,
          account_code: this.user.code,
          currency: this.selectedCurrency.currency,
          payment_payload: this.selectedBank.type,
          payment_payload_value: this.selectedBank.number,
          notes: 'test',
          stage: 1,
          charge: 0
        }
        this.APIRequest('withdrawals/create', par).then(response => {
          $('#loading').css({display: 'none'})
          if(response.data === true) {
            this.$refs.otp.show()
          } else {
            $('<div>', {
              class: 'text-danger col-12 pl-3 text-center mb-3 error',
              html: 'There was an error sending in your request. Please try again.'
            }).insertBefore('#transferFunds #next')
          }
        })
      }
    },
    successOTP(otpval){
      $('#transferFunds .error').remove()
      // $('<div>', {
      //   class: 'text-success col-12 pl-3 text-center mb-3 error',
      //   html: 'Money transfer handling not yet made.'
      // }).insertBefore('#transferFunds #next')
      $('#loading').css({display: 'block'})
      let par = {
        amount: this.amount,
        account_id: this.user.userID,
        account_code: this.user.code,
        currency: this.selectedCurrency.currency,
        payment_payload: this.selectedBank.type,
        payment_payload_value: this.selectedBank.number,
        notes: 'test',
        stage: 2,
        otp: otpval,
        charge: 0
      }
      this.APIRequest('withdrawals/create', par).then(response => {
        $('#loading').css({display: 'none'})
        if(response.data > 0) {
          this.transferred = true
          this.next = null
          this.successMessage = 'Successfully sent!'
        } else {
          $('<div>', {
            class: 'text-danger col-12 pl-3 text-center mb-3 error',
            html: 'There was an error sending in your request. Please try again.'
          }).insertBefore('#transferFunds #next')
        }
      })
    }
  }
}
</script>
