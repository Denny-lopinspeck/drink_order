<template>
  <div id="app">
    <div class="container gx-2">
      <div class="row gx-3 bg-light py-3">
        <div class="col-md-4">
          <div class="list-group">
            <a href="#" class="list-group-item list-group-item-action" :class="{'active': tempSelected.name == product.name }" v-for="product in products" :key="product.name" @click.prevent="selectedProduct(product)">
              <h6 class="card-title mb-1">{{ product.name }}</h6>
              <div class="d-flex align-items-center justify-content-between">
                <p class="mb-0"><small>{{ product.engName }}</small></p>
                <p class="mb-0"><small>NT$ {{ product.price }}</small></p>
              </div>
            </a>
          </div>
        </div>
        <div class="col-md-8">
          <div class="card mb-2">
            <div v-if="!tempSelected.hasOwnProperty('name')" class="position-absolute text-white d-flex align-items-center justify-content-center" style="top: 0;bottom: 0;left: 0;right: 0;background-color: rgba(0, 0, 0, 0.65); z-index: 100;">請先選擇飲品</div>
            <div class="card-body px-4">
              <div class="mb-3">
                <label for="productNum" class="form-label">數量</label>
                <input type="number" class="form-control" id="productNum" placeholder="數量" v-model="tempSelected.count" min="0">
              </div>
              <div class="mb-3">
                <label for="productNum" class="form-label d-block">冰塊*</label>
                <div class="form-check form-check-inline" v-for="(ice, key) in iceType" :key="'ice' + key">
                  <input class="form-check-input" name="iceType" type="radio" :id="'ice' + key" :value="ice" v-model="tempSelected.ice" :disabled="!tempSelected.hasOwnProperty('defaults') || (tempSelected.defaults.ice !== '' && tempSelected.defaults.ice !== ice)">
                  <label class="form-check-label" :for="'ice' + key">{{ ice }}</label>
                </div>
              </div>
              <div class="mb-3">
                <label for="productNum" class="form-label d-block">甜度*</label>
                <div class="form-check form-check-inline" v-for="(sugar, key) in sugarType" :key="'sugar' + key">
                  <input class="form-check-input" name="sugarType" type="radio" :id="'sugar' + key" :value="sugar" v-model="tempSelected.sugar" :disabled="!tempSelected.hasOwnProperty('defaults') || (tempSelected.defaults.sugar !== '' && tempSelected.defaults.sugar !== sugar)">
                  <label class="form-check-label" :for="'sugar' + key">{{ sugar }}</label>
                </div>
              </div>
              <div class="mb-3">
                <label for="productNum" class="form-label d-block">加料</label>
                <div class="form-check form-check-inline" v-for="(topping, key) in toppingsType" :key="'topping' + key">
                  <input class="form-check-input" type="checkbox" :id="'topping' + key" :value="topping" v-model="tempSelected.toppings" :disabled="!tempSelected.hasOwnProperty('defaults')">
                  <label class="form-check-label" :for="'topping' + key">{{ topping }}</label>
                </div>
              </div>
              <div class="mb-3">
                <label for="productNotice" class="form-label">備註</label>
                <textarea class="form-control" id="productNotice" rows="2" v-model="tempSelected.notice"></textarea>
              </div>
              <div class="d-flex gap-2">
                <button class="btn btn-outline-primary w-100" type="button" @click="reset">取消</button>
                <button class="btn btn-primary w-100" type="button" :disabled="!tempSelected.hasOwnProperty('name')" @click="addToOrder(tempSelected)">加入</button>
              </div>
            </div>
          </div>
          <div class="card" v-if="orderList.length > 0">
            <div class="card-body">
              <table class="table">
                <thead>
                  <tr>
                    <th scope="col">品項</th>
                    <th scope="col">冰塊</th>
                    <th scope="col">甜度</th>
                    <th scope="col">加料</th>
                    <th scope="col">單價</th>
                    <th scope="col">數量</th>
                    <th scope="col">小計</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="(item, key) in orderList" :key="'order' + key">
                    <th scope="row">{{ item.name }}<br>
                      <small v-if="item.notice !== ''" class="text-muted fw-normal">備註：{{ item.notice }}</small>
                    </th>
                    <td>{{ item.ice }}</td>
                    <td>{{ item.sugar }}</td>
                    <td>{{ item.toppings.toString() }}</td>
                    <td>{{ item.price + item.toppings.length*10 }}</td>
                    <td>{{ item.count }}</td>
                    <td class="text-end">{{ item.total }}</td>
                  </tr>
                </tbody>
              </table>
              <p class="text-end">共 NT$ {{ orderTotal }} 元</p>
              <button class="btn btn-sm btn-secondary w-100" :disabled="orderList.length === 0" @click="generateOrder(orderList, orderTotal)">產生訂單</button>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="bg-light p-3 mt-3" v-if="checkedOrder.orders.length > 0">
      <div class="bg-white p-3 d-flex flex-column" style="min-height: 450px;">
        <table class="table">
          <thead>
            <tr>
              <th scope="col">品項</th>
              <th scope="col">冰塊</th>
              <th scope="col">甜度</th>
              <th scope="col">加料</th>
              <th scope="col">單價</th>
              <th scope="col">數量</th>
              <th scope="col">小計</th>
              <th scope="col" v-if="!checkedOrder.isPaid">操作</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(item, key) in checkedOrder.orders" :key="'order' + key">
              <th scope="row">{{ item.name }}<br>
                <small v-if="item.notice !== ''" class="text-muted fw-normal">備註：{{ item.notice }}</small>
              </th>
              <td>{{ item.ice }}</td>
              <td>{{ item.sugar }}</td>
              <td>{{ item.toppings.toString() }}</td>
              <td>{{ item.price + item.toppings.length*10 }}</td>
              <td>{{ item.count }}</td>
              <td class="text-end">{{ item.total }}</td>
              <td v-if="!checkedOrder.isPaid">
                <button class="btn btn-sm btn-outline-danger" @click="removeOrderItem(key)">
                  <i class="bi bi-trash"></i> 刪除
                </button>
              </td>
            </tr>
          </tbody>
        </table>
        <p class="mt-3 mb-1">訂單成立時間：{{ checkedOrder.time }}</p>
        <p class="mb-1">餐點數： {{ checkedOrder.orders.length }}</p>
        <p class="mb-1">付款狀態：{{ checkedOrder.isPaid ? '已付款' : '未付款' }}</p>
        <p class="text-end mt-auto">共 NT$ {{ checkedOrder.total }} 元</p>
        <div class="d-flex gap-2" v-if="!checkedOrder.isPaid">
          <button class="btn btn-danger w-100" @click="cancelOrder">取消訂單</button>
          <button class="btn btn-success w-100" @click="payOrder">結帳</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      checkedOrder: {
        time: '',
        total: 0,
        orders: [],
        isPaid: false
      },
      orderTotal: 0,
      orderList: [],
      tempSelected: {},
      iceType: ['正常冰', '少冰', '微冰', '去冰', '熱'],
      sugarType: ['全糖', '七分', '半糖', '微糖', '無糖'],
      toppingsType: ['珍珠', '粉粿', '小粉圓', '椰果', '芋頭'],
      products: [
        {
          name: '珍珠鮮奶茶',
          engName: 'Pearl Milk Tea',
          price: 60,
          defaults: {
            toppings: ['珍珠'],
            sugar: '',
            ice: '',
          }
        },
        {
          name: '椰果鮮奶茶',
          engName: 'Coconut Milk Tea',
          price: 60,
          defaults: {
            toppings: ['椰果'],
            sugar: '',
            ice: '',
          }
        },
        {
          name: '鮮奶茶',
          engName: 'Milk Tea',
          price: 50,
          defaults: {
            toppings: [''],
            sugar: '',
            ice: '',
          }
        },
        {
          name: '古意冬瓜茶 (糖固定)',
          engName: 'Winter Melon Drink',
          price: 30,
          defaults: {
            toppings: [''],
            sugar: '微糖',
            ice: '',
          }
        },
        {
          name: '蜜香紅茶',
          engName: 'Black Tea',
          price: 30,
          defaults: {
            toppings: [''],
            sugar: '',
            ice: '',
          }
        },
        {
          name: '包種青茶',
          engName: 'Black Tea',
          price: 35,
          defaults: {
            toppings: [''],
            sugar: '',
            ice: '',
          }
        },
        {
          name: '檸檬烏龍',
          engName: 'Lemon Oolong Tea',
          price: 55,
          defaults: {
            toppings: [''],
            sugar: '',
            ice: '',
          }
        },
        {
          name: '薑母茶 (熱飲)',
          engName: 'Ginger Tea',
          price: 55,
          defaults: {
            toppings: [''],
            sugar: '',
            ice: '熱',
          }
        },
        {
          name: '青草茶',
          engName: 'Herbal Tea',
          price: 35,
          defaults: {
            toppings: [''],
            sugar: '',
            ice: '',
          }
        },
        {
          name: '金桔檸檬',
          engName: 'Kumquat Lemonade',
          price: 40,
          defaults: {
            toppings: [''],
            sugar: '',
            ice: '',
          }
        },
        {
          name: '柳澄青茶',
          engName: 'Orange Mountain Tea',
          price: 45,
          defaults: {
            toppings: [''],
            sugar: '',
            ice: '',
          }
        },
      ],
    }
  },
  methods: {
    reset() {
      this.tempSelected = {};
    },
    countTotal() {
      this.orderTotal = this.orderList.reduce((acc, obj) => acc + obj.total, 0);
    },
    selectedProduct(product) {
      this.tempSelected = {
        count: '1',
        ice: product.defaults.ice !== '' ? product.defaults.ice : '正常冰',
        sugar: product.defaults.sugar !== '' ? product.defaults.sugar : '全糖',
        toppings: [],
        notice: '',
        ...product
      };
    },
    addToOrder(product) {
      this.orderList.push(Object.assign({}, {
        ...product,
        total: (product.price + product.toppings.length * 10) * product.count
      }));
      this.countTotal();
      this.reset();
    },
    generateOrder(orders, total) {
      if (this.checkedOrder.orders.length === 0) {
       
        const date = new Date().toLocaleString();
        this.checkedOrder.time = date;
        this.checkedOrder.total = total;
        this.checkedOrder.orders = orders;
        this.checkedOrder.isPaid = false;
      } else if (!this.checkedOrder.isPaid) {
        
        this.checkedOrder.orders = [...this.checkedOrder.orders, ...orders];
        this.checkedOrder.total += total;
      }
      this.orderList = [];
      this.reset();
    },
    payOrder() {
      this.checkedOrder.isPaid = true;
      setTimeout(() => {
        this.cancelOrder();
      }, 1000);
    },
    cancelOrder() {
      this.checkedOrder = {
        time: '',
        total: 0,
        orders: [],
        isPaid: false
      };
    },
    removeOrderItem(index) {
     
      this.checkedOrder.total -= this.checkedOrder.orders[index].total;
      this.checkedOrder.orders.splice(index, 1);
      
    
      if (this.checkedOrder.orders.length === 0) {
        this.cancelOrder();
      }
    }
  }
};
</script>

<style scoped>

body {
font-family: Arial, sans-serif;
margin: 0;
padding: 0;
background-color: #f8f9fa;
}

.container {
width: 100%;
padding-right: 15px;
padding-left: 15px;
margin-right: auto;
margin-left: auto;
}

.card {
position: relative;
display: flex;
flex-direction: column;
min-width: 0;
word-wrap: break-word;
background-color: #fff;
background-clip: border-box;
border: 1px solid rgba(0,0,0,.125);
border-radius: 0.25rem;
}

.card {
border: 1px solid #e0e0e0;
border-radius: 8px;
background-color: #fff;
box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.card-body {
padding: 20px;
}

.list-group {
padding: 0;
margin: 0;
list-style: none;
}

.list-group-item {
padding: 10px 15px;
border: 1px solid #e0e0e0;
border-radius: 4px;
margin-bottom: 8px;
cursor: pointer;
transition: background-color 0.3s, transform 0.2s;
}

.list-group-item:hover {
background-color: #f1f1f1;
}

.list-group-item.active {
background-color: #007bff;
color: white;
border-color: #007bff;
}

.form-label {
font-weight: bold;
margin-bottom: 5px;
}

.form-control {
border: 1px solid #ced4da;
border-radius: 4px;
padding: 8px 12px;
}

.form-check-input {
margin-right: 8px;
}

.form-check-label {
margin-bottom: 0;
}

.table {
width: 100%;
border-collapse: collapse;
margin: 20px 0;
}

.table th, .table td {
padding: 12px;
text-align: left;
border-bottom: 1px solid #dee2e6;
}

.table th {
background-color: #f8f9fa;
}

.text-end {
text-align: right;
}

.bg-light {
background-color: #f8f9fa !important;
}

.text-white {
color: white !important;
}

.btn {
display: inline-block;
font-weight: 400;
text-align: center;
vertical-align: middle;
user-select: none;
border: 1px solid transparent;
padding: 10px 20px;
font-size: 14px;
border-radius: 4px;
transition: all 0.3s ease;
}

.btn-primary {
color: #fff;
background-color: #007bff;
border-color: #007bff;
}

.btn-primary:hover {
background-color: #0056b3;
border-color: #0056b3;
}

.btn-outline-primary {
color: #007bff;
background-color: transparent;
border-color: #007bff;
}

.btn-outline-primary:hover {
color: #fff;
background-color: #007bff;
}

.btn-secondary {
color: #fff;
background-color: #6c757d;
border-color: #6c757d;
}

.btn-secondary:hover {
background-color: #5a6268;
border-color: #545b62;
}

.btn-danger {
color: #fff;
background-color: #dc3545;
border-color: #dc3545;
}

.btn-danger:hover {
background-color: #c82333;
border-color: #bd2130;
}

.btn-success {
color: #fff;
background-color: #28a745;
border-color: #28a745;
}

.btn-success:hover {
background-color: #218838;
border-color: #1e7e34;
}

.position-absolute {
position: absolute;
}

.d-flex {
display: flex !important;
}

.flex-column {
flex-direction: column !important;
}

.align-items-center {
align-items: center !important;
}

.justify-content-center {
justify-content: center !important;
}

.mt-3 {
margin-top: 1rem !important;
}

.mb-3 {
margin-bottom: 1rem !important;
}

.gap-2 {
gap: 0.5rem !important;
}

.text-muted {
color: #6c757d !important;
}

.btn-outline-danger {
  color: #dc3545;
  background-color: transparent;
  border-color: #dc3545;
}

.btn-outline-danger:hover {
  color: #fff;
  background-color: #dc3545;
  border-color: #dc3545;
}

.btn-sm {
  padding: 0.25rem 0.5rem;
  font-size: 0.875rem;
  line-height: 1.5;
  border-radius: 0.2rem;
}
</style>