<template>
  <div>
    <div class="t-search mb-10">Ищем: <input type="text" v-model="search"></div>
    <div class="mb-10">
      <a class="mr-20" href="#" v-on:click="this.reverse ? sortASC(this.table.info, 'rating') : sortDESC(this.table.info, 'rating')">Рейтинг</a>
      <a href="#" v-on:click="this.reverse ? sortASC(this.table.info, 'registration_date') : sortDESC(this.table.info, 'registration_date')">Дата</a>
    </div>
    <div style="position: relative" class="mb-10">
      <div class="t-header">
        <div class="t-row">
          <div class="t-col" :key="key" v-for="(item, key) in table.header">{{item}}</div>
          <div class="t-col"></div>
        </div>
      </div>
      <div class="t-row" :key="index" v-for="(items, index) in dataTable" >
        <div class="t-col" :key="key" v-for="(item, key) in items" v-show="key !== 'id'" >
          {{item}}
        </div>
        <div class="t-col"><a href="#" v-on:click="modal(items.id)">del</a></div>
      </div>
      <div class="modal" v-show="modalStatus">
        <button @click="del(itemDel)">yas</button>
        <button @click="modalStatus = false">no</button>
      </div>
    </div>

    <button :disabled="pageNumber===0" @click="prevPage">
      Previous
    </button>
    <span class="p-number" :key="n" v-for="n in pageCount" v-on:click="page(n - 1)">{{ n }} </span>
    <button :disabled="pageNumber===pageCount-1" @click="nextPage">
      Next
    </button>
    <select>
      <option v-on:click="tableSize(5)">5</option>
      <option v-on:click="tableSize(10)">10</option>
      <option v-on:click="tableSize(20)">20</option>
    </select>

  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'Table',
  components: {
    // eslint-disable-next-line vue/no-unused-components
    axios
  },
  data: () => {
    return {
      pageNumber: 0,
      modalStatus: false,
      itemDel: '',
      sortKey: 'name',
      reverse: true,
      search: "",
      table: {
        header: [
          'Имя пользователя',
          'E-mail',
          'Дата регистрации',
          'Рейтинг'
        ],
        size: 5,
        info: []
      },
    }
  },
  computed: {
    dataTable() {
      let newTable = [];
      let obj = this.table.info;

      const serach = this.search.toLowerCase();
      const start = this.pageNumber * this.table.size,
          end = start + this.table.size;

      for (let key in obj) {
        let el = obj[key];
        let title = el.username.toLowerCase();
        let email = el.email.toLowerCase();

        if (title.indexOf(serach) !== -1 || email.indexOf(serach) !== -1) newTable.push(el)
      }

      obj = newTable;

      return newTable
          .slice(start, end);
    },
    pageCount(){
      let l,
          s = this.table.size;

      if (this.search === '') {
        l = this.table.info.length;
      } else {
        l = this.dataTable.length;
      }


      return Math.ceil(l/s);
    }
  },
  mounted() {
    axios
        .get('https://5ebbb8e5f2cfeb001697d05c.mockapi.io/users')
        .then(response => (this.table.info = response.data));
  },
  methods: {
    nextPage(){
      this.pageNumber++;
    },
    prevPage(){
      this.pageNumber--;
    },
    page(number) {
      this.pageNumber = number
    },
    tableSize(number) {
      this.table.size = number
    },
    modal: function (ID) {
      this.itemDel = ID;
      this.modalStatus = true;
    },
    del: function(ID) {
      for (let key in this.table.info) {
        let el = this.table.info[key];

        if (el.id === ID) {
          delete this.table.info[key];
        }

        this.modalStatus = false;
      }
    },
    sortASC: function(table, param) {
      table.sort(function(a, b) {
        return a[param] === b[param] ? a > b : a[param] > b[param]
      });

      this.reverse = false;
    },
    sortDESC: function(table, param) {
      table.sort((a, b) => {
        return a[param] === b[param] ? a < b : a[param] < b[param]
      });

      this.reverse = true;
    },
  }
}
</script>

<style scoped>
  .t-header {
    border-bottom: 1px solid black;
  }
  .t-row {
    display: flex;
  }
  .t-col {
    width: 100%;
    text-align: left;
    padding: 5px 0;
  }
  .p-number {
    padding-left: 5px;
    padding-right: 5px;
    cursor: pointer;
  }
  .t-search {
  }
  .mr-20 {
    margin-right: 20px;
  }
  .mb-10 {
    margin-bottom: 10px;
  }
  .modal {
    background-color: #8f8f8f;
    padding: 20px;
    max-width: 100px;
    display: flex;
    justify-content: space-between;
    position: absolute;
    right: 0;
    left: 0;
    top: 0;
    bottom: 0;
    margin: auto;
    height: 30px;
  }
  .modal button {
    cursor: pointer;
  }
</style>