<script setup>
import axios from 'axios';
import { computed, ref } from 'vue';
import Paginate from 'vuejs-paginate-next';

const searchtext = ref('');
const bikeList = ref(null);
const currpage = ref(1);
const totalpage = ref(1);
const perpagenumber = 20;
const sortkey = ref('');
const sortorder = ref([{ total: 1 }, { available_rent_bikes: 1 }]);
const tittleList = [
  'sno',
  'sna',
  'sarea',
  'ar',
  'total',
  'available_rent_bikes',
  'latitude',
  'longitude',
  'available_return_bikes'
];

getData();

const filteredData = computed(() => {
  let data = bikeList.value;
  let searchdata = searchtext.value;
  let nowpage = currpage.value;
  if (searchdata) {
    data = data.filter((row) => {
      return Object.keys(row).some(() => {
        return String(row['ar']).indexOf(searchdata) > -1;
      });
    });
  }

  const key = sortkey.value;
  if (key) {
    const order = sortorder.value[key];
    data = data.slice().sort((a, b) => {
      a = a[key];
      b = b[key];
      return (a === b ? 0 : a > b ? 1 : -1) * order;
    });
  }

  settotalpage(data);

  let index = 0;
  if (data) {
    data = data.filter(() => {
      index++;
      if (index > (nowpage - 1) * perpagenumber && index <= nowpage * perpagenumber) {
        return true;
      }
      return false;
    });
  }

  return data;
});

function settotalpage(data) {
  totalpage.value = data ? Math.ceil(data.length / perpagenumber) : 0;
}

function getData() {
  callapi().then((response) => {
    bikeList.value = response.data;
  });
}

async function callapi() {
  const res = await axios.get(
    'https://tcgbusfs.blob.core.windows.net/dotapp/youbike/v2/youbike_immediate.json'
  );
  return res;
}

function changepage(page) {
  currpage.value = page;
}
function totaldesc() {
  sortkey.value = 'total';
  sortorder.value[sortkey.value] = 1;
}
function totalasc() {
  sortkey.value = 'total';
  sortorder.value[sortkey.value] = -1;
}
function abikesdesc() {
  sortkey.value = 'available_rent_bikes';
  sortorder.value[sortkey.value] = 1;
}
function abikesasc() {
  sortkey.value = 'available_rent_bikes';
  sortorder.value[sortkey.value] = -1;
}
</script>

<template>
  <form class="row g-3">
    <div class="col-auto">
      <input type="text" placeholder="輸入地址" v-model="searchtext" />
    </div>
    <div class="col-auto">
      <button type="submit" class="btn btn-primary mb-3">Search</button>
    </div>
  </form>
  <Paginate
    :page-count="totalpage"
    :page-range="5"
    :initial-page="1"
    :click-handler="changepage"
  ></Paginate>
  <table class="table table-striped">
    <thead>
      <tr>
        <th class="col-1 title text-center">站點編號</th>
        <th class="col-1 title text-center">站點名稱</th>
        <th class="col-1 title text-center">站點所在區域</th>
        <th class="col-1 title text-center">站點地址</th>
        <th class="col-1 title text-center">
          <span>總車位數量</span>
          <span>
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="16"
              height="16"
              fill="currentColor"
              class="bi bi-caret-up"
              viewBox="0 0 16 16"
              @click="totaldesc"
            >
              <path
                d="M3.204 11h9.592L8 5.519zm-.753-.659 4.796-5.48a1 1 0 0 1 1.506 0l4.796 5.48c.566.647.106 1.659-.753 1.659H3.204a1 1 0 0 1-.753-1.659"
              />
            </svg>
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="16"
              height="16"
              fill="currentColor"
              class="bi bi-caret-down"
              viewBox="0 0 16 16"
              @click="totalasc"
            >
              <path
                d="M3.204 5h9.592L8 10.481zm-.753.659 4.796 5.48a1 1 0 0 0 1.506 0l4.796-5.48c.566-.647.106-1.659-.753-1.659H3.204a1 1 0 0 0-.753 1.659"
              />
            </svg>
          </span>
        </th>
        <th class="col-2 title text-center">
          <div class="flex item-center">
            <span>可租借的腳踏車數量</span>
            <span>
              <svg
                xmlns="http://www.w3.org/2000/svg"
                width="16"
                height="16"
                fill="currentColor"
                class="bi bi-caret-up"
                viewBox="0 0 16 16"
                @click="abikesdesc"
              >
                <path
                  d="M3.204 11h9.592L8 5.519zm-.753-.659 4.796-5.48a1 1 0 0 1 1.506 0l4.796 5.48c.566.647.106 1.659-.753 1.659H3.204a1 1 0 0 1-.753-1.659"
                />
              </svg>
              <svg
                xmlns="http://www.w3.org/2000/svg"
                width="16"
                height="16"
                fill="currentColor"
                class="bi bi-caret-down"
                viewBox="0 0 16 16"
                @click="abikesasc"
              >
                <path
                  d="M3.204 5h9.592L8 10.481zm-.753.659 4.796 5.48a1 1 0 0 0 1.506 0l4.796-5.48c.566-.647.106-1.659-.753-1.659H3.204a1 1 0 0 0-.753 1.659"
                />
              </svg>
            </span>
          </div>
        </th>
        <th class="col-1 title text-center">站點緯度</th>
        <th class="col-1 title text-center">站點經度</th>
        <th class="col-1 title text-center">可歸還的腳踏車數量</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="bike in filteredData" :key="bike">
        <td v-for="key in tittleList" :key="key">
          {{ bike[key] }}
        </td>
      </tr>
    </tbody>
  </table>
</template>

<style>
.title {
  border: solid black 1px;
  height: 50px;
  background-color: aqua;
  vertical-align: middle;
}
</style>
