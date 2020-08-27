<template>
  <div>
    <div class="t-search mb-10">Ищем: <input type="text" v-model="search"></div>
    <div class="mb-10">
      <a class="mr-20" href="#" v-on:click="this.reverse ? sortASC(this.table.info, 'rating') : sortDESC(this.table.info, 'rating')">Рейтинг</a>
      <a href="#" v-on:click="this.reverse ? sortASC(this.table.info, 'registration_date') : sortDESC(this.table.info, 'registration_date')">Дата</a>
    </div>
    <div class="mb-10">
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
        <div class="t-col"><a href="#" v-on:click="del(items.id)">del</a></div>
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
    del: function(table) {
      for (let key in this.table.info) {
        let el = this.table.info[key];

        if (el.id === table) {
          delete this.table.info[key];
        }
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
</style>