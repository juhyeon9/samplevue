<template>
  <dl>
    <dt>
      <strong>{{ ptitle }}</strong>
    </dt>
    <dd class="content">
      <!-- s : 여기에 내용입력 -->
      <table class="row">
        <caption>
          caption
        </caption>
        <colgroup>
          <col width="10%" />
          <col width="*" />
        </colgroup>

        <tbody>
          <tr>
            <th colspan="2">{{ ptitle }}</th>
          </tr>
          <tr>
            <th scope="row">제목 <span class="font_red">*</span></th>
            <td>
              <input
                type="text"
                class="inputTxt w350"
                name="title"
                id="title"
                v-model="title"
              />
            </td>
          </tr>
          <tr>
            <th scope="row">내용 <span class="font_red">*</span></th>
            <td>
              <textarea
                class="inputTxt w350"
                name="cont"
                id="cont"
                v-model="cont"
              >
              </textarea>
            </td>
          </tr>
        </tbody>
      </table>

      <!-- e : 여기에 내용입력 -->
      <div class="btn_areaC mt30">
        <a class="btnType blue" id="btnSave" name="btn" @click="save()"
          ><span>저장</span></a
        >
        <a
          class="btnType blue"
          id="btnDelete"
          name="btn"
          v-show="delshow"
          @click="gdelete()"
          ><span>삭제</span></a
        >
        <a class="btnType gray" id="btnClose" name="btn"><span>취소</span></a>
      </div>
    </dd>
  </dl>
</template>

<script>
import { closeModal } from 'jenesius-vue-modal';
export default {
  props: { ptitle: String, noticeNo: Number, action: String },
  data: function () {
    return {
      delshow: false,
      title: '',
      cont: '',
      vaction: '',
    };
  },
  mounted() {
    this.vaction = this.action;

    let vm = this;

    console.log(
      'ptitle : ' +
        this.ptitle +
        ' noticeNo : ' +
        this.noticeNo +
        ' action : ' +
        this.action
    );

    if (this.action == 'U') {
      this.delshow = true;

      let params = new URLSearchParams();
      params.append('bd_no', this.noticeNo);

      this.axios
        .post('/admsst/selectFileupload.do', params)
        .then(function (response) {
          console.log(response);
          vm.title = response.data.searchone.bd_title;
          vm.cont = response.data.searchone.bd_contents;
        })
        .catch(function (error) {
          alert('에러! API 요청에 오류가 있습니다. ' + error);
        });
    } else {
      this.delshow = false;
    }
  },
  methods: {
    save: function () {
      let vm = this;

      let params = new URLSearchParams();
      params.append('title', this.title);
      params.append('cont', this.cont);
      params.append('action', this.vaction);
      params.append('bd_no', this.noticeNo);

      this.axios
        .post('/admsst/saveFileupload.do', params)
        .then(function (response) {
          console.log(response);

          if (response.data.result == 'SUCESS') {
            if (vm.vaction == 'D') {
              alert('삭제 되었습니다');
            } else {
              alert('저장 되었습니다');
            }

            closeModal(vm, '121221');
          }
        })
        .catch(function (error) {
          alert('에러! API 요청에 오류가 있습니다. ' + error);
        });
    },
    gdelete: function () {
      this.vaction = 'D';
      this.save();
    },
  },
};
</script>
<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
.modal-item.widget__modal-wrap {
  padding: 30px 30px;
  background: #fff;
  opacity: 1;
}

dt > strong {
  display: inline-block;
  width: 100%;
  margin-bottom: 20px;
  text-align: center;
  font-size: 20px;
}

textarea.inputTxt {
  border: 1px solid #cdcdcd;
}
</style>
