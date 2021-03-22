<template>
    <div class="container">
      <div class="row-inline">
        <h1>Инспекция по ресторанам</h1>
        <div class="dropdown">
          <button class="dropbtn">Редактировать таблицу</button>
            <div class="dropdown-content">
              <p v-for="field in fields" :key="field.key">
                <input type="checkbox" :value="field.key" :checked="field.checked" v-model="field.checked"> {{field.label}}
              </p>
            </div>
        </div>
        <input type="text" class="filter" placeholder="Поиск по полям таблицы" v-model="matchStr">
      </div>
      <table id="grid">
        <thead>
          <tr>
            <th v-for="field in hotFields" :key="field.key" @click="(e) => {onHeaderClick(e, field)}">{{field.label}}</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="item in hotItems" :key="item.id" :class="item.class">
            <td v-for="field in hotFields" :key="field.label">{{item[field.key]}}</td>
          </tr>
        </tbody>
      </table>
      <div class="row-inline">
        <div>
          <span>Начало проверки</span>
          <input type="date" class="input-date" v-model="startDate">
        </div>
        <div>
          <span>Конец проверки</span>
          <input type="date" class="input-date" v-model="endDate">
        </div>
        <span class="counter">Всего элементов: {{hotItems.length}}</span>
      </div>
    </div>
</template>

<script>

import Axios from 'axios';

export default {
    name: 'MainWrapper',
    components: {},
    props: {},
    data() {
        return {
          startDate: null,
          endDate: null,
          matchStr: '',
          json: [],
          fields: [
            {
              key: 'business_name',
              label: 'Название ресторана',
              checked: true,
              type: 'text',
            },
            {
              key: 'business_address',
              label: 'Адрес ресторана',
              checked: true,
              type: 'text',
            },
            {
              key: 'business_city',
              label: 'Город',
              checked: true,
              type: 'text',
            },
            {
              key: 'business_phone_number',
              label: 'Почтовый индекс',
              checked: true,
              type: 'number',
            },
            {
              key: 'business_postal_code',
              label: 'Номер ресторана',
              checked: true,
              type: 'number',
            },
            {
              key: 'inspection_date',
              label: 'Дата инспекции',
              checked: true,
              type: 'date',
            },
            {
              key: 'inspection_description',
              label: 'Статус инспекции',
              checked: true,
              type: 'text',
            },
            {
              key: 'inspection_type',
              label: 'Тип проведения инспекции',
              checked: true,
              type: 'text',
            },
          ],
          items: [],
        };
    },
    computed: {
      hotItems() {
        let hotItems = this.items;
        if (this.matchStr !=='') {
          hotItems = this.items.filter((el) => {
            for (let i in this.hotFields) {
              let key = this.hotFields[i].key;
              if (String(el[key]).toLowerCase().includes(this.matchStr.toLowerCase())) {
                return true;
              }
            }
          });
        }
        if (this.startDate !== null || this.endDate !== null) {
          let startDate = this.startDate;
          let endDate = this.endDate;
          if (startDate === null || startDate === '') {
            startDate = '1970-01-01';
          }
          if (endDate === null || endDate === '') {
            endDate = new Date().toISOString().slice(0, 10);
          }
          hotItems = this.items.filter((el) => startDate <= el.inspection_date && endDate >= el.inspection_date);
        }
        return hotItems;
      },
      hotFields() {
        let hotFields = this.fields.filter((el) => el.checked);
        return hotFields;
      },
    },
    created() {
      this.getJson();
    },
    mounted() {},
    methods: {
      getJson () {
        Axios({
          method: 'GET',
          url: '/static/json.json',
        })
          .then((response) => {
            this.json = response.data;
            this.items = this.json.map((el, index) => {
              let row = {}
              for (let field of this.fields){
                if (field.key === 'inspection_description') {
                  if (el[field.key] === 'ENFORCEMENT INSPECTION REQUIRED') {
                    row[field.key] = el[field.key];
                    this.$set(row, 'class', 'class-red');
                  } else if(el[field.key] === 'REINSPECTION REQUIRED') {
                    this.$set(row, 'class', 'class-yellow');
                    row[field.key] = el[field.key];
                  } else {
                    this.$set(row, 'class', 'class-white');
                    row[field.key] = el[field.key];
                  }
                } else {
                  row[field.key] = el[field.key];
                }
              }
              this.$set(row, 'id', index);
              return row;
            })
          })
          .catch((err) => {
            console.log(err);
          })
      },
      onHeaderClick (e, field) {
        if (e.target.tagName !== 'TH') {
          return;
        }
        this.sortGrid(field.key, field.type);
      },
      sortGrid (key, type) {
        let compare;
        switch (type) {
          case 'number':
            compare = function(rowA, rowB) {
              return rowA[key] - rowB[key];
            };
            break;
          case 'text':
            compare = function(rowA, rowB) {
              if (rowA[key] > rowB[key]) {
                return 1;
              } else if (rowA[key] < rowB[key]) {
                return -1;
              } else {
                return 0;
              }
            };
            break;
          case 'date':
            compare = function(rowA, rowB) {
              if (rowA[key] > rowB[key]) {
                return 1;
              } else if (rowA[key] < rowB[key]) {
                return -1;
              } else {
                return 0;
              }
            };
            break;
        }
        this.items.sort(compare);
      },
    },
    watch: {},
};
</script>