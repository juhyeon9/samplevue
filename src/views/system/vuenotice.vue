<template>
  <form id="popupfile">
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
                <div id="searchArea" class="d-flex justify-content-around">
                  <ComCombo
                    group_code="svScore"
                    selectid="svScoresel"
                    type="all"
                    selvalue=""
                  ></ComCombo>
                  <select
                    id="searchtype"
                    name="searchtype"
                    style="width: 150px"
                    v-model="searchKey"
                  >
                    <option value="">전체</option>
                    <option value="seq">순번</option>
                    <option value="title">제목</option>
                    <option value="writer">작성자</option>
                  </select>
                  <input
                    type="text"
                    style="width: 200px"
                    id="searchvalue"
                    name="searchvalue"
                    v-model="searchvalue"
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

                    <a
                      @click="newreg()"
                      class="btn btn-primary mx-2"
                      id="btnSearchGrpcod"
                      name="btn"
                    >
                      <span>신규등록</span>
                    </a>

                    <a
                      @click="newregfile()"
                      class="btn btn-primary mx-2"
                      id="btnSearchGrpcod"
                      name="btn"
                    >
                      <span>신규등록(파일)</span>
                    </a>
                  </span>
                </div>
              </td>
            </tr>
          </table>
        </span>
      </p>
      <!-- 리스트 -->
      <div class="divComGrpCodList">
        <table class="col">
          <caption>
            caption
          </caption>
          <colgroup>
            <col width="10%" />
            <col width="40%" />
            <col width="25%" />
            <col width="25%" />
          </colgroup>

          <thead>
            <tr>
              <th scope="col">순번</th>
              <th scope="col">제목</th>
              <th scope="col">작성자</th>
              <th scope="col">작성일</th>
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
                <td @click="newreg(item.bd_no)">{{ item.bd_no }}</td>
                <td @click="detailviewfile(item.bd_no)">{{ item.bd_title }}</td>
                <td>{{ item.loginID }}</td>
                <td>{{ item.bd_date }}</td>
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
    </div>
  </form>
</template>
<script>
import { openModal } from 'jenesius-vue-modal';
import regModal from '@/components/system/noticeModal.vue';
import regModalFile from '@/components/system/noticeModalFile.vue';
import ComCombo from '@/components/common/ComCombo.vue';
import Paginate from 'vuejs-paginate-next';
// 파라미터
export default {
  data: function () {
    return {
      samplecom: '',
      searchKey: '',
      searchvalue: '',
      currentPage: 1,
      pageSize: 10,
      totalPage: 0,
      totalCnt: 0,
      action: '',
      noticeNo: 0,
      ptitle: '',
      items: [],
    };
  },
  components: {
    paginate: Paginate,
    ComCombo: ComCombo,
  },
  mounted() {
    this.listsearch();
  },
  methods: {
    listsearch: function () {
      let vm = this;

      let params = new URLSearchParams();
      params.append('pagenum', this.currentPage);
      params.append('pagesize', this.pageSize);
      params.append('searchtype', this.searchKey);
      params.append('searchvalue', this.searchvalue);

      this.axios
        .post('/admsst/listFileuploadvue.do', params)
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

      console.log('colpage : ' + colpage + '   lit : ' + lit);

      if (colpage < 1) {
        return 1;
      }

      if (lit == 0) {
        return Math.round(colpage);
      } else {
        return Math.round(colpage) + 1;
      }
    },
    newreg: async function (no) {
      console.log('no : ' + no);

      if (no == null || no == '') {
        this.action = 'I';
        this.noticeNo = 0;
        this.ptitle = '등록';
      } else {
        this.action = 'U';
        this.noticeNo = no;
        this.ptitle = '수정';
      }

      console.log('action : ' + this.action);

      const openpop = await openModal(regModal, {
        noticeNo: this.noticeNo,
        action: this.action,
        ptitle: this.ptitle,
      });

      openpop.onclose = (popupparam) => {
        console.log('Close : ' + popupparam);
        this.listsearch();
        //return false; //Modal will not be closed
      };
    },
    newregfile: async function (no) {
      console.log('no : ' + no);

      if (no == null || no == '') {
        this.action = 'I';
        this.noticeNo = 0;
        this.ptitle = '등록';
      } else {
        this.action = 'U';
        this.noticeNo = no;
        this.ptitle = '수정';
      }

      console.log('action : ' + this.action);

      const openpop = await openModal(regModalFile, {
        noticeNo: this.noticeNo,
        action: this.action,
        ptitle: this.ptitle,
      });

      openpop.onclose = (popupparam) => {
        console.log('Close : ' + popupparam);
        this.listsearch();
        //return false; //Modal will not be closed
      };
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