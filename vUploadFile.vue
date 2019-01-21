
<template lang="html">

  <div class="file-wrap">

    <div class="upload-img" :style="{ width : imgwidth , height : imgheight  }" @click.stop="selectImg" v-if="noselect">
      <span class="add" @click.stop="selectImg"></span>
    </div>

    <div v-else="!noselect">
      <div class="contenier" v-if="percent<100">
        <x-progress :percent="percent" class="press" :show-cancel="false"></x-progress><span class="txt">{{ percent + '%' }}</span>
      </div>
      <div class="file-name" v-else>
        <span class="file-icon"></span>
        <div class="file-box">
          <span class="file-tit">{{ filename }}</span>
          <span class="cancle" @click.stop="cancleUpload"></span>
        </div>
      </div>
    </div>

    <!-- wps , xlsx , xls , ppt , pdf , word -->
    <input ref="fileIpt" :id="uploadId" class="file" name="file" type="file" accept="application/vnd.ms-works ,application/vnd.openxmlformats-officedocument.spreadsheetml.sheet , application/vnd.ms-excel , application/vnd.ms-powerpoint , application/pdf , application/vnd.openxmlformats-officedocument.wordprocessingml.document"  @change="update"/>

  </div>
</template>

<script>
/* 使用方法 */
/*

*/
import { XProgress , TransferDom } from 'vux';
import { isMobile } from '../config/isMobile.js'
import { setTimeout } from 'timers';
export default {
  directives: {
    TransferDom
  },
  data () {
    return {
      domain: "https://cert.linhuiba.com" ,  // 固定这个
      tokenURL: "https://api.linhuiba.com/qiniu/app-token/linhuiba-certs" ,
      uploadURL: '',
      fileType: 'cert',
      region:'z0',
      max: 1 ,
      fileSize : '' , // 文件大小
      filename : '', // 文件名
      percent : 0 , // 当前百分百
      noselect : true ,
    }
  },
  components:{ XProgress },

  props:{
    'fileUrls' : {
      type : Array ,
      default : function () {
        return []
      } ,
      require : true
    },
    'uploadId' : {
      type : String,
      require : true
    },
    'imgwidth':{
      default : '1.5rem',
      type : String
    },
    'imgheight':{
      default : '1.5rem',
      type : String
    }

  },
  watch: {
    fileUrls:{
      immediate: true,
      handler: function(val, oldVal){
        if( this.fileUrls[0]&&this.fileUrls[0].filename ){
          this.noselect = false ;
          this.percent = 100 ;
          this.filename = this.fileUrls[0].filename ;
        }
      }
    }
  },
  mounted(){
    this.setUploadURL();
    if( this.fileUrls[0]&&this.fileUrls[0].filename ){
      this.noselect = false ;
      this.percent = 100 ;
      this.filename = this.fileUrls[0].filename ;
    }
  },
  methods:{
    setUploadURL() {
        let region =this.region;
        let uploadURL = null;
        switch(region) {
            case 'z0': uploadURL = 'https://upload.qiniup.com'; break;//华东
            case 'z1': uploadURL = 'https://upload-z1.qiniup.com'; break;//华北
            case 'z2': uploadURL = 'https://upload-z2.qiniup.com'; break;//华南
            case 'na0': uploadURL = 'https://upload-na0.qiniup.com'; break;//北美
            default: //console.error('please make the region is with one of [ECN, SCN, NCN, NA,z0,z1,z2,na0]');
        }
        this.uploadURL = uploadURL;
      },
    update(e){

        let max = this.max ;
        let files = Array.from( e.target.files); // 保留多个文件的可能性
        let len = e.target.files.length ,type = e.target.files[0].type , filename = e.target.files[0].name ;
        let reg = /\.pdf$|\.doc$|\.docx$|\.xls$|\.xlsx$|\.ppt$|\.pptx$|\.ppsx$|\.xltx$/ ; // 判断后缀的形式限制文件类型
        //let reg = /(application\/vnd.ms-works)|(application\/vnd\.openxmlformats-officedocument\.spreadsheetml\.sheet)|(application\/vnd\.ms-excel)|(application\/vnd\.ms-powerpoint)|(application\/pdf)|(application\/vnd\.openxmlformats-officedocument\.wordprocessingml\.document)/gi ;
        if( len > max ){
          this.$vux.toast.text('只能上传一份文档', 'middle');
          return ;
        }
        if( !reg.test( filename ) ){
          this.$vux.toast.text('文档格式不支持', 'middle');
          return ;
        }
//        if( !reg.test( type ) ){
//          this.$vux.toast.text('文档格式不支持', 'middle');
//          return ;
//        }
        this.uploading(files)
      },
    uploading(files){
      this.noselect = false ; // 隐藏上传区域
        let file = files.shift() ; // 拿到第一个文件 , 保留多个文件的可能性
          // console.log(file.size);
        if (file.size > 1024 * 1024) { // MB
          this.fileSize = Math.round( file.size * 100 / (1024 * 1024)) / 100 ;
          // 限制大小
          if( this.fileSize > 20 ){
            this.$vux.toast.text('文档大小需限制在20MB以下');
            return false ;
          }
        }else{ // KB
          this.fileSize = Math.round(file.size * 100 / 1024) / 100  ;
        }

        this.filename = file.name ;
        let param = new FormData(); //创建form对象
        param.append('chunk','0');//断点传输
        param.append('chunks','1');
        param.append('file',file,file.name) ;
          // console.log(param.get('file')); //FormData私有类对象，访问不到，可以通过get判断值是否传进去
          //先从自己的服务端拿到token

          this.$http.get( this.tokenURL ).then(res=>{
            //console.log(res)
            let token = res.data.result ;
            param.append('token',token);
            this.$http.post( this.uploadURL , param ,{
                headers:{'Content-Type': 'multipart/form-data' },
                onUploadProgress : (evt) => {
                  // 上传进度
                  if ( evt.lengthComputable ) {
                    let percentComplete = Math.round( evt.loaded * 100 / evt.total );
                    this.percent = percentComplete ;
                  }
                  else {
                    this.$vux.toast.text('无法获取上传进度', 'middle');
                  }
                }
              }).then( response=>{
                // console.log(response);
                this.fileUrls.shift() ;
                this.fileUrls.push({ link : `${this.domain}/${response.data.key}?attname=${ decodeURIComponent(file.name) }` , filename : this.filename  });
                // 上传成功立即返回结果
                 this.$emit('success',[{ link : `${this.domain}/${response.data.key}?attname=${ decodeURIComponent(file.name) }` , filename : this.filename  }]) ;
                if( this.percent == 100 ){
                // 长度为100 ， 结束上传
                this.$vux.toast.text('文档上传成功', 'middle');
                return;
              }else{
                this.$vux.toast.text('文档上传失败', 'middle');
                this.$emit('error',  '' );
                return;
              }

                // 多张上传
                // this.uploading(files)
              }).catch( e => {
                this.$vux.toast.text('文件上传失败', 'middle');
                this.$emit('error',e)
                return;
              })
          })
      },
    selectImg(){
      if( this.$refs.fileIpt.value ){
        this.$refs.fileIpt.value= ''
      }
      this.$refs.fileIpt.dispatchEvent( new MouseEvent('click'));
    },
    cancleUpload(){
      this.noselect = true ;
      this.percent = 0 ;
      this.fileUrls.shift() ;
    }
  }
}

</script>
<style lang="scss" scoped>
  @import '../style/mixin';
  .file-wrap{
    /*overflow: hidden;*/
    padding-bottom: 0.2rem;
    .file{
        z-index:-99999;
        visibility: hidden;
        opacity:0;
        height:0;
        position: absolute;
    }
    .upload-img {
      position: relative;
      display: flex;
      justify-content: center;
      align-items: center;
      background-color: #f1f2f6;
      .add{
        @include wh(0.72rem,0.72rem);
        @include bg-image('../images/ic_add_three_one');
      }
    }
    .contenier{
      display: flex;
      align-items: center;
      .press{
        width: 90%;
      }
      .txt{
        margin-left: 0.1rem;
        @include sc(0.28rem,$fontColor);
      }
    }
    .file-name{
      display: flex;
      align-items: flex-end;
      height: 0.8rem;
      .file-icon{
        @include wh(0.44rem,0.4rem);
        margin:0 0.2rem 0.1rem 0;
        @include bg-image('../images/ic_folder_three_one');
      }
      .file-box{
        position: relative;
        height: 0.6rem;
        line-height: 0.6rem;
        max-width: 4.2rem;
        padding: 0 0.1rem;
        border-radius: 0.08rem;
        background-color: #f1f2f7;
        .file-tit{
          display: block;
          width: 100%;
          overflow: hidden;
          white-space: nowrap;
          text-overflow: ellipsis;
        }
        .cancle{
          position: absolute;
          top: -0.2rem;
          right: -0.2rem;
          @include wh(0.38rem,0.38rem);
          @include bg-image('../images/ic_delete_three_one');
        }
      }
    }
  }
</style>
