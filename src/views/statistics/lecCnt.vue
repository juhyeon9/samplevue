<template>
  <form id="chartvue">
    <input type="hidden" id="titlenm" name="titlenm" value="" />
    <input type="hidden" id="titleap" name="titleap" value="" />
    <input type="hidden" id="titlegarden" name="titlegarden" value="" />
    <div id="comnCodMgr">
      <p class="Location">
        <a href="/dashboard/home" class="btn_set home"></a>
        <span class="btn_nav bold">학습관리</span>
        <span class="btn_nav bold">게시판</span>
        <a href="../system/comnCodMgr.do" class="btn_set refresh">새로고침</a>
      </p>

      <p class="conTitle">
        <span>게시판</span>
        <span>
          <table
            style="collapse; border: 1px #50bcdf;"
            width="100%"
            cellpadding="5"
            cellspacing="0"
            border="1"
            align="left"
          >
            <tr style="border: 0px; border-color: blue">
              <td
                width="50"
                height="25"
                style="font-size: 100%; text-align: left"
              >
                <!-- https://kongmemo.tistory.com/7 -->
                <div id="searchArea" class="d-flex justify-content-start">
                  <input
                    type="date"
                    style="width: 100px"
                    id="startdate"
                    name="startdate"
                    v-model="startdate"
                  />
                  <input
                    type="date"
                    style="width: 100px"
                    id="enddate"
                    name="enddate"
                    v-model="enddate"
                  />

                  <span class="fr">
                    <a
                      @click="listsearch"
                      class="btn btn-primary mx-2"
                      id="btnSearchGrpcod"
                      name="btn"
                    >
                      <span>검 색</span>
                    </a>
                  </span>
                </div>
              </td>
            </tr>
          </table>
        </span>
      </p>

      <div class="divComGrpCodList">
        <table class="col">
          <caption>
            caption
          </caption>
          <colgroup>
            <col width="10%" />
            <col width="30%" />
            <col width="20%" />
            <col width="20%" />
            <col width="10%" />
            <col width="10%" />
          </colgroup>

          <thead>
            <tr>
              <th scope="col">순번</th>
              <th scope="col">과목명</th>
              <th scope="col">강의시작</th>
              <th scope="col">강의종료</th>
              <th scope="col">신청인원</th>
              <th scope="col">정원</th>
            </tr>
          </thead>
          <tbody>
            <template v-if="totalCnt == 0">
              <tr>
                <td colspan="6">일치하는 검색 결과가 없습니다</td>
              </tr>
            </template>
            <template v-else>
              <tr v-for="item in items" :key="item.noticeNo">
                <td>{{ item.li_no }}</td>
                <td>{{ item.li_nm }}</td>
                <td>{{ item.li_date }}</td>
                <td>{{ item.li_redate }}</td>
                <td>{{ item.li_ap }}</td>
                <td>{{ item.li_garden }}</td>
              </tr>
            </template>
          </tbody>
        </table>
      </div>

      <div id="noticedPagination">
        <paginate
          class="justify-content-center"
          v-model="currentPage"
          :page-count="totalPage"
          :page-range="10"
          :margin-pages="0"
          :click-handler="clickpagenum"
          :prev-text="'Prev'"
          :next-text="'Next'"
          :container-class="'pagination'"
          :page-class="'page-item'"
        >
        </paginate>
      </div>
      <div id="chartapp">
        <column-chart :data="[['Sun', 32], ['Mon', 46], ['Tue', 28]]"></column-chart>
        <!-- <column-chart :data="[[titlenm[0], titleap[0], titlegarden[0]]]"></column-chart> -->
      </div>
    </div>
  </form>
</template>
<script>
import Paginate from 'vuejs-paginate-next';

export default {
  props: {titlenm: String, titleap: Number, titlegarden: Number},
  data() {
    return {
      startdate: '',
      enddate: '',
      currentPage: 1,
      pageSize: 5,
      totalPage: 0,
      totalCnt: 0,
      action: '',
      noticeNo: 0,
      items: [],
      // chartData: {
      //   labels: [ 'January', 'February', 'March'],
      //   datasets: [
      //     {
      //       label: 'Data One',
      //       backgroundColor: '#f87979',
      //       data: [40, 20, 12]
      //     }
      //   ]
      // },
    };
  },
  components: {
    paginate: Paginate,
  },
  mounted() {
    this.listsearch();
    this.li_nm = this.titlenm;
    this.li_ap = this.titleap;
    this.li_garden = this.titlegarden;
  },
  methods: {
    listsearch: function () {
      let vm = this;

      let params = new URLSearchParams();
      params.append('currentPage', this.currentPage);
      params.append('pagesize', this.pageSize);
      params.append('startdate', this.startdate);
      params.append('enddate', this.enddate);

      this.axios
        .post('/statistics/listLecCntvue.do', params)
        .then(function (response) {
          console.log(response);
          vm.items = response.data.searchlist;
          vm.totalCnt = response.data.searchlistcnt;
          vm.totalPage = vm.page();
        })
        .catch(function (error) {
          alert('에러! API 요청에 오류가 있습니다. ' + error);
        });
    },
    clickpagenum: function (pagenum) {
      this.currentPage = pagenum;
      this.listsearch();
    },
    page: function () {
      let colpage = this.totalCnt / this.pageSize;
      let lit = this.totalCnt % this.pageSize;

      console.log(
        'colpage : ' +
          colpage +
          '   lit : ' +
          lit +
          '    totalCnt : ' +
          this.totalCnt +
          '   pageSize : ' +
          this.pageSize
      );

      console.log(
        'titlenm : ' +
          this.titlenm +
          ' titleap : ' +
          this.titleap +
          ' titlegarden : ' +
          this.titlegarden
      );

      if (colpage < 1) {
        return 1;
      }

      if (lit == 0) {
        return Math.round(colpage);
      } else {
        return Math.round(colpage) + 1;
      }
    },
  },
};
</script>

<style>
#searchArea {
  margin-top: 25px;
  margin-bottom: 25px;
}
#searchArea > * {
  height: auto;
}

#groupTitle {
  text-decoration: underline;
  font-weight: bold;
  cursor: pointer;
}
</style>
