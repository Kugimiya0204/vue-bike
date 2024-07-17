<script setup>
import axios from 'axios';
import { computed, ref } from 'vue';

const searchtext = ref('');
const bikeList = ref(null);
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
  if (searchdata) {
    data = data.filter((row) => {
      return Object.keys(row).some(() => {
        return String(row['ar']).indexOf(searchdata) > -1;
      });
    });
  }
  return data;
});

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
</script>

<template>
  <form class="row g-3">
    <div class="col-auto">
      <input type="text" placeholder="輸入地址" v-model="searchtext" />
    </div>
    <div class="col-auto">
      <button type="submit" class="btn btn-primary mb-3">Search</button>
      <h1>{{ searchtext }}</h1>
    </div>
  </form>
  <table class="table table-striped">
    <thead>
      <tr>
        <th class="col-1">站點編號</th>
        <th class="col-1">站點名稱</th>
        <th class="col-1">站點所在區域</th>
        <th class="col-1">站點地址</th>
        <th class="col-1">總車位數量</th>
        <th class="col-1">可租借的腳踏車數量</th>
        <th class="col-1">站點緯度</th>
        <th class="col-1">站點經度</th>
        <th class="col-1">可歸還的腳踏車數量</th>
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
