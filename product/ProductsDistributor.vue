<template>
  <div class="holder">
    <filter-product v-bind:category="category" 
      :activeCategoryIndex="0"
      :activeSortingIndex="0"
      @changeSortEvent="retrieve($event.sort, $event.filter)"
      @changeStyle="manageGrid($event)"
      :grid="['list', 'th-large']">
    </filter-product>
    <image-view :data="data" :flag="false" v-if="listStyle === 'columns'"></image-view>
    <table-view :data="data" v-if="listStyle === 'list' && data !== null" :type="'consignments'"></table-view>
    <empty v-if="data === null" :title="empty.title" :action="empty.guide"></empty>
  </div>
</template>
<style>
  .holder{  
    width: 98%;
    float: left;
    margin-right: 2%;
    margin-top: 25px;
    margin-bottom: 50px;
  }
  .results{
    width: 100%;
    float: left;
    margin-top: 25px;
  }
  .item:hover{
    cursor: pointer;
  }

  .products-holder{
    width: 24%;
    float: left;
    height: 300px;
    margin-right: 1%;
    border: solid 1px #ddd;
    margin-bottom: 10px;
    color: #555;
    margin-top: 25px;
  }
  .products-holder:hover{
    cursor: pointer;
    border: solid 1px #ffaa81;
    background: #ffaa81;
    color: #fff;
  }

  .products-image{
    width: 100%;
    float: left;
    position: relative;
    height: 250px;
    text-align: center;
  }

  .products-image img{
    height: 250px;
    float: left;
    width: 100%;
  }
  .products-image .fa-image{
    font-size: 150px;
    line-height: 250px;

  }
  .products-details{
    height: 50px;
    width: 100%;
    float: left;
    border-top: solid 1px #ddd;
  }
  .products-title{
    width: 50%;
    float: left;
    height: 50px;
  }
  .products-title label{
    width: 100%;
    float: left;
    font-size: 12px;
    margin: 0px !important;
    padding-left: 10px;
  }
  .products-price{
    width: 50%;
    float: left;
    height: 50px;
    line-height: 50px;
    font-weight: 600;
    text-align: right;
    padding-right: 5px;
  }

  .products-wishlist{
    height: 50px;
    text-align: center;
    line-height: 50px;
    width: 100%;
    position: absolute;
    top: 50%;
    z-index: 10;
    display: none;
  }

  .products-wishlist:hover, .products-wishlist i:hover, .products-wishlist label:hover{
    cursor: pointer;
  }

  .products-wishlist label{
    line-height: 50px;
    float: left;
    width: 50%;
    text-align: center;
  }
  
  .products-wishlist i{
    font-size: 32px;
    line-height: 50px;
  }

  .products-holder:hover .products-wishlist{
    display: block;
  }

</style>
<script>
import ROUTER from 'src/router'
import AUTH from 'src/services/auth'
import CONFIG from 'src/config.js'
import axios from 'axios'
import COMMON from 'src/common.js'
export default {
  mounted(){
    this.retrieve({'title': 'asc'}, {column: 'title', value: ''})
  },
  data(){
    return {
      user: AUTH.user,
      config: CONFIG,
      errorMessage: null,
      data: null,
      selectedItem: null,
      selectedIndex: null,
      listStyle: 'list',
      category: [{
        title: 'Product',
        sorting: [{
          title: 'Created at ascending',
          payload: 'created_at',
          payload_value: 'asc',
          type: 'date'
        }, {
          title: 'Created at descending',
          payload: 'created_at',
          payload_value: 'desc',
          type: 'date'
        }]
      }],
      common: COMMON,
      currentFilter: null,
      currentSort: null,
      empty: {
        title: null,
        guide: null
      }
    }
  },
  components: {
    'create': require('components/increment/ecommerce/product/Create.vue'),
    'table-view': require('components/increment/ecommerce/product/TableView.vue'),
    'empty': require('components/increment/generic/empty/Empty.vue'),
    'filter-product': require('components/increment/ecommerce/filter/Product.vue'),
    'image-view': require('components/increment/ecommerce/product/ImageView.vue')
  },
  methods: {
    redirect(parameter){
      ROUTER.push(parameter)
    },
    retrieve(sort, filter){
      if(this.user.subAccount === null || this.user.subAccount.merchant === null || typeof this.user.subAccount.merchant === 'undefined'){
        this.empty = {
          title: 'Empty Merchant!',
          guide: 'Go to My Profile then Merchant Settings and fill up necessary information.'
        }
        return false
      }
      if(filter !== null){
        this.currentFilter = filter
      }
      if(sort !== null){
        this.currentSort = sort
      }
      let parameter = {
        condition: {
          value: this.currentFilter.value + '%',
          column: this.currentFilter.column
        },
        merchant_id: this.user.subAccount.merchant.id,
        sort: this.currentSort,
        account_id: this.user.userID,
        inventory_type: this.common.ecommerce.inventoryType,
        type: this.user.type
      }
      $('#loading').css({'display': 'block'})
      this.APIRequest('transfers/retrieve_consignments', parameter).then(response => {
        $('#loading').css({'display': 'none'})
        if(response.data.length > 0){
          this.data = response.data
          if(this.selectedItem !== null){
            this.selectedItem = this.data[this.selectedIndex]
          }
        }else{
          this.data = null
          this.selectedIndex = null
          this.selectedItem = null
          this.empty = {
            title: 'Empty Consignments!',
            guide: 'Start asking to transfer item to your account.'
          }
        }
      })
    },
    editModal(item, index){
      for (var i = 0; i < this.$children.length; i++) {
        if(this.$children[i].$el.id === 'updateProducts'){
          this.selectedItem = item
          this.selectedIndex = index
          this.$children[i].modal()
        }
      }
    },
    manageGrid(event){
      switch(event){
        case 'th-large': this.listStyle = 'columns'
          break
        case 'list': this.listStyle = 'list'
          break
      }
    }
  }
}
</script>
