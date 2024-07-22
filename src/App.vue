<script setup>
import { computed, ref, onMounted, watch } from 'vue';

//搜尋用的字
const searchtext = ref('');
//記錄所有bike資料
const bikeList = ref([]);

//現在在第幾頁
const currpage = ref(1);
//全部有幾頁
const totalpage = ref(1);
//每頁有幾個
const perpagenumber = 20;

//記錄order的鍵是哪個
const sortkey = ref('');
//紀錄order是asc還是desc
const sortorder = ref({ total: 0, available_rent_bikes: 0 });
//紀錄title
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

//計算要呈現的table
const filteredData = computed(() => {
  //確認bike資料改變
  let data = bikeList.value;
  //確認searchtext改變
  let searchdata = searchtext.value;
  //確認現在頁數改變
  let nowpage = currpage.value;

  //如果有搜尋字
  if (searchdata) {
    //過濾要呈現table資料
    data = data.filter((row) => {
      return String(row['ar']).indexOf(searchdata) > -1;
    });
  }

  //確認order的鍵是否改變
  const key = sortkey.value;

  //如果order有鍵
  if (key) {
    //拿到order是asc或desc
    const order = sortorder.value[key];
    //使目前table排序
    data = data.slice().sort((a, b) => {
      a = a[key];
      b = b[key];
      return (a === b ? 0 : a > b ? 1 : -1) * order;
    });
  }

  //設定目前總頁數
  settotalpage(data);

  isRest.value ? resetpage : null;

  //如果有資料
  if (data) {
    data = data.slice((nowpage - 1) * perpagenumber, nowpage * perpagenumber);
  }

  return data;
});

//設定目前總頁數
function settotalpage(data) {
  totalpage.value = data ? Math.ceil(data.length / perpagenumber) : 0;
}

//呼叫api，獲得資料
async function callapi() {
  const res = await fetch(
    'https://tcgbusfs.blob.core.windows.net/dotapp/youbike/v2/youbike_immediate.json'
  );
  const data = await res.json();
  bikeList.value = data;
}

//改變當前頁數為page
function changepage(page) {
  currpage.value = page;
}
//往前一頁
function prevpage() {
  currpage.value--;
}
//往後一頁
function nextpage() {
  currpage.value++;
}
const isRest = ref(false);
//更改搜尋字串時重製頁數
watch(searchtext, () => {
  if (searchtext.value) {
    isRest.value = true;
  } else {
    isRest.value = false;
  }
});

//依照傳入的鍵跟排序調整資料
function orderData(key, order) {
  sortkey.value = key;
  for (const skey in sortorder.value) {
    skey == key ? (sortorder.value[skey] = order) : (sortorder.value[skey] = 0);
  }
}

//重製當前頁數為1
function resetpage() {
  currpage.value = 1;
}

//判斷地址是否為搜尋字串並更改css
const IsSearchText = (ar) => {
  if (searchtext.value) {
    return ar.replace(
      searchtext.value,
      `<span class="text-danger fw-bolder">${searchtext.value}</span>`
    );
  } else {
    return ar;
  }
};

onMounted(async () => {
  //呼叫api抓資料
  await callapi();
});
</script>

<template>
  <div
    name="all"
    style="
      background-image: url('https://as2.ftcdn.net/v2/jpg/01/66/56/07/1000_F_166560713_jsAL3tE7oc2ROtpAIvMkDkHFzlTdg0SF.jpg');
      background-size: cover;
    "
  >
    <nav class="navbar bg-body-tertiary bg-info-subtle mb-2">
      <div class="container-fluid">
        <span
          class="myfont2"
          style="
            border: solid black 1px;
            background-color: aquamarine;
            font-weight: 900;
            font-size: 30px;
            margin-left: 5%;
            width: 20%;
          "
          ><img
            src="https://images.pexels.com/photos/686230/pexels-photo-686230.jpeg"
            style="width: 18%; border: solid green 2px; border-radius: 99em"
          />
          <a> Youbike Data Table </a>
        </span>
        <img
          src="https://as2.ftcdn.net/v2/jpg/01/35/12/95/1000_F_135129528_glAKm25XD0kLcxbjz98SrD2AUlHnpT7L.jpg"
          style="
            width: 5%;
            border: solid black 1px;
            border-radius: 99em;
            position: absolute;
            top: 1px;
            left: 500px;
            animation: moveToRight;
            animation-duration: 10s;
            animation-timing-function: cubic-bezier(0.4, 0.5, 0.9, 1);
          "
        />
        <form class="d-flex">
          <input type="text" class="my-1 me-3" placeholder="輸入地址" v-model="searchtext" />
          <button type="submit" class="btn btn-primary">Search</button>
        </form>
      </div>
    </nav>
    <div name="table" style="width: 80%; margin-left: 10%; padding-bottom: 2%; margin-bottom: 1%">
      <table class="table table-striped border border-black">
        <thead>
          <tr>
            <th class="col-1 myfont1 title text-center bg-warning-subtle">站點編號</th>
            <th class="col-2 myfont1 title text-center bg-warning-subtle">站點名稱</th>
            <th class="col-1 myfont1 title text-center bg-warning-subtle">站點所在區域</th>
            <th class="col-2 myfont1 title text-center bg-warning-subtle">站點地址</th>
            <th class="col-1 myfont1 title text-center bg-warning-subtle">
              <span>總車位數量</span>
              <span>
                <svg
                  v-if="sortorder.total != 1"
                  xmlns="http://www.w3.org/2000/svg"
                  width="16"
                  height="16"
                  fill="currentColor"
                  class="bi bi-caret-up"
                  viewBox="0 0 16 16"
                  @click="orderData('total', 1)"
                >
                  <path
                    d="M3.204 11h9.592L8 5.519zm-.753-.659 4.796-5.48a1 1 0 0 1 1.506 0l4.796 5.48c.566.647.106 1.659-.753 1.659H3.204a1 1 0 0 1-.753-1.659"
                  />
                </svg>
                <svg
                  v-if="sortorder.total == 1"
                  xmlns="http://www.w3.org/2000/svg"
                  width="16"
                  height="16"
                  fill="currentColor"
                  class="bi bi-caret-up-fill"
                  viewBox="0 0 16 16"
                >
                  <path
                    d="m7.247 4.86-4.796 5.481c-.566.647-.106 1.659.753 1.659h9.592a1 1 0 0 0 .753-1.659l-4.796-5.48a1 1 0 0 0-1.506 0z"
                  />
                </svg>
                <svg
                  v-if="sortorder.total != -1"
                  xmlns="http://www.w3.org/2000/svg"
                  width="16"
                  height="16"
                  fill="currentColor"
                  class="bi bi-caret-down"
                  viewBox="0 0 16 16"
                  @click="orderData('total', -1)"
                >
                  <path
                    d="M3.204 5h9.592L8 10.481zm-.753.659 4.796 5.48a1 1 0 0 0 1.506 0l4.796-5.48c.566-.647.106-1.659-.753-1.659H3.204a1 1 0 0 0-.753 1.659"
                  />
                </svg>
                <svg
                  v-if="sortorder.total == -1"
                  xmlns="http://www.w3.org/2000/svg"
                  width="16"
                  height="16"
                  fill="currentColor"
                  class="bi bi-caret-down-fill"
                  viewBox="0 0 16 16"
                >
                  <path
                    d="M7.247 11.14 2.451 5.658C1.885 5.013 2.345 4 3.204 4h9.592a1 1 0 0 1 .753 1.659l-4.796 5.48a1 1 0 0 1-1.506 0z"
                  />
                </svg>
              </span>
            </th>
            <th class="col-1 myfont1 title text-center bg-warning-subtle">
              <div class="flex item-center">
                <span>可租借的腳踏車數量</span>
                <span>
                  <svg
                    v-if="sortorder.available_rent_bikes != 1"
                    xmlns="http://www.w3.org/2000/svg"
                    width="16"
                    height="16"
                    fill="currentColor"
                    class="bi bi-caret-up"
                    viewBox="0 0 16 16"
                    @click="orderData('available_rent_bikes', 1)"
                  >
                    <path
                      d="M3.204 11h9.592L8 5.519zm-.753-.659 4.796-5.48a1 1 0 0 1 1.506 0l4.796 5.48c.566.647.106 1.659-.753 1.659H3.204a1 1 0 0 1-.753-1.659"
                    />
                  </svg>
                  <svg
                    v-if="sortorder.available_rent_bikes == 1"
                    xmlns="http://www.w3.org/2000/svg"
                    width="16"
                    height="16"
                    fill="currentColor"
                    class="bi bi-caret-up-fill"
                    viewBox="0 0 16 16"
                  >
                    <path
                      d="m7.247 4.86-4.796 5.481c-.566.647-.106 1.659.753 1.659h9.592a1 1 0 0 0 .753-1.659l-4.796-5.48a1 1 0 0 0-1.506 0z"
                    />
                  </svg>
                  <svg
                    v-if="sortorder.available_rent_bikes != -1"
                    xmlns="http://www.w3.org/2000/svg"
                    width="16"
                    height="16"
                    fill="currentColor"
                    class="bi bi-caret-down"
                    viewBox="0 0 16 16"
                    @click="orderData('available_rent_bikes', -1)"
                  >
                    <path
                      d="M3.204 5h9.592L8 10.481zm-.753.659 4.796 5.48a1 1 0 0 0 1.506 0l4.796-5.48c.566-.647.106-1.659-.753-1.659H3.204a1 1 0 0 0-.753 1.659"
                    />
                  </svg>
                  <svg
                    v-if="sortorder.available_rent_bikes == -1"
                    xmlns="http://www.w3.org/2000/svg"
                    width="16"
                    height="16"
                    fill="currentColor"
                    class="bi bi-caret-down-fill"
                    viewBox="0 0 16 16"
                  >
                    <path
                      d="M7.247 11.14 2.451 5.658C1.885 5.013 2.345 4 3.204 4h9.592a1 1 0 0 1 .753 1.659l-4.796 5.48a1 1 0 0 1-1.506 0z"
                    />
                  </svg>
                </span>
              </div>
            </th>
            <th class="col-1 myfont1 title text-center bg-warning-subtle">站點緯度</th>
            <th class="col-1 myfont1 title text-center bg-warning-subtle">站點經度</th>
            <th class="col-1 myfont1 title text-center bg-warning-subtle">可歸還的腳踏車數量</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="bike in filteredData" :key="bike">
            <td class="text-center" v-for="key in tittleList" :key="key">
              <span v-if="key == 'ar'" v-html="IsSearchText(bike[key])"></span>
              <span v-else>
                {{ bike[key] }}
              </span>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
  <div name="page" style="width: 40%; margin-left: 40%">
    <ul class="pagination">
      <li v-if="currpage != 1" class="page-item">
        <a class="page-link" href="#" @click="prevpage">Previous</a>
      </li>
      <li v-else class="page-item">
        <a class="page-link bg-secondary-subtle">Previous</a>
      </li>
      <li v-for="index in totalpage" :key="index" class="page-item">
        <button
          v-if="index == currpage"
          class="page-link bg-primary text-white"
          href="#"
          @click="changepage(index)"
        >
          {{ index }}
        </button>
        <button
          v-else-if="index <= currpage + 2 && index >= currpage - 2"
          class="page-link"
          href="#"
          @click="changepage(index)"
        >
          {{ index }}
        </button>
      </li>
      <li v-if="currpage + 3 < totalpage"><a class="page-link" href="#">...</a></li>
      <li v-if="currpage != totalpage" class="page-item">
        <a class="page-link" href="#" @click="nextpage">Next</a>
      </li>
      <li v-else class="page-item"><a class="page-link bg-secondary-subtle">Next</a></li>
    </ul>
  </div>
</template>

<style>
.title {
  border: solid black 1px;
  height: 50px;
  vertical-align: middle;
  font-weight: 900;
}
@keyframes moveToRight {
  0% {
    left: 500px;
    transform: rotateY(0deg);
    opacity: 1;
  }
  25% {
    left: 1500px;
    transform: rotateY(0deg);
    opacity: 0.8;
  }
  50% {
    left: 1500px;
    transform: rotateY(180deg);
    opacity: 1;
  }
  75% {
    left: 500px;
    transform: rotateY(180deg);
    opacity: 0.8;
  }
  100% {
    left: 500px;
    transform: rotateY(0deg);
    opacity: 1;
  }
}
</style>
