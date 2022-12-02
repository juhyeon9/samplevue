<template>
  <form id="popupfile">
    <input type="hidden" id="action" name="action" v-model="vaction" />
    <input type="hidden" id="bd_no" name="bd_no" v-model="bd_no" />
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
              <th scope="row">제목 <span class="font_red">*</span></th>
              <td>
                <input
                  type="text"
                  class="inputTxt w350"
                  name="titlefile"
                  id="titlefile"
                  v-model="title"
                />
              </td>
            </tr>
            <tr>
              <th scope="row">내용 <span class="font_red">*</span></th>
              <td>
                <textarea
                  class="inputTxt w350"
                  name="contfile"
                  id="contfile"
                  v-model="cont"
                >
                </textarea>
              </td>
            </tr>
            <tr>
              <th scope="row">파일 <span class="font_red">*</span></th>
              <td>
                <input
                  type="file"
                  id="upfile"
                  name="upfile"
                  ref="selFiles"
                  @change="fpreview"
                />
              </td>
            </tr>
            <tr>
              <th scope="row">파일 다운로드<span class="font_red">*</span></th>
              <td>
                <div
                  id="filedownloaddiv"
                  @click="fdownload()"
                  v-html="disdownload"
                ></div>
              </td>
            </tr>
            <tr>
              <th scope="row">파일 미리보기<span class="font_red">*</span></th>
              <td>
                <div id="filepewviewdiv" v-if="vaction == 'U' && previewflag">
                  <img v-if="url" :src="`http://localhost/${url}`" />
                </div>
                <div id="filepewviewdiv" v-else>
                  <img v-if="url" :src="url" />
                </div>
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
          <a @click="popclose()" class="btnType gray" id="btnClose" name="btn"
            ><span>취소</span></a
          >
        </div>
      </dd>
    </dl>
  </form>
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
      disdownload: '다운로드',
      dispreview: '미리보기',
      bd_no: '',
      filename: '',
      previewflag: true,
      url: '',
    };
  },
  mounted() {
    this.vaction = this.action;
    this.bd_no = this.noticeNo;

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

          let att_ori = response.data.searchone.att_ori;
          vm.filename = response.data.searchone.att_ori;

          if (att_ori == '' || att_ori == null) {
            vm.disdownload = '';
          } else {
            vm.disdownload =
              response.data.searchone.att_ori +
              '(' +
              response.data.searchone.att_size +
              ')';
          }

          vm.url = response.data.searchone.att_nli;
          //vm.url = '/serverfile//notice//해피잡.jpg';
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

      alert(this.vaction);

      let frm = document.getElementById('popupfile');
      let dataWithFile = new FormData(frm);

      this.axios(
        {
          url: '/admsst/saveFileuploadatt.do',
          method: 'post',
          data: dataWithFile,
        },
        {
          headers: {
            'Content-Type': 'multipart/form-data',
          },
        }
      )
        .then(function (response) {
          console.log(response);

          if (response.data.result == 'SUCESS') {
            alert('저장 되었습니다');

            closeModal(vm, '121221');
          }
        })
        .catch(function (error) {
          alert('에러! API 요청에 오류가 있습니다. ' + error);
        });
    },
    gdelete: function () {
      let vm = this;
      this.vaction = 'D';

      let params = new URLSearchParams();
      params.append('bd_no', this.noticeNo);
      params.append('action', 'D');

      this.axios
        .post('/admsst/saveFileuploadatt.do', params)
        .then(function (response) {
          console.log(response);
          alert('삭제 되었습니다');
          closeModal(vm);
        })
        .catch(function (error) {
          alert('에러! API 요청에 오류가 있습니다. ' + error);
        });
    },
    fpreview: function (event) {
      const file = event.target.files[0];

      this.previewflag = false;

      //this.files = this.$refs.selFiles.files;

      if (file) {
        alert(file.name);

        let orifile = file.name;
        let orifilearr = orifile.split('.');

        let ext = orifilearr[1];

        if (
          ext.toLowerCase() == 'jpg' ||
          ext.toLowerCase() == 'png' ||
          ext.toLowerCase() == 'gif'
        ) {
          this.url = URL.createObjectURL(file);
        } else {
          console.log('path : 미리보기 갑니다.');
          this.url = '';
        }
      }
    },
    fdownload: function () {
      // alert('download !!!!!!!!!!!!!!!!!!!' + this.noticeNo);

      let params = new URLSearchParams();
      params.append('bd_no', this.noticeNo);

      this.axios
        .post('/admsst/downloadfile.do', params, {
          responseType: 'blob',
        })
        .then((response) => {
          const url = window.URL.createObjectURL(new Blob([response.data]));
          const link = document.createElement('a');
          link.href = url;
          link.setAttribute('download', this.filename); //or any other extension
          document.body.appendChild(link);
          link.click();
        })
        .catch(function (error) {
          alert('에러! API 요청에 오류가 있습니다. ' + error);
        });
    },
    popclose: function () {
      closeModal(this);
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
