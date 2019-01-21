<template lang="html">
  <div class="file-wrap">
    <div class="upload-image">
        <img :style="{ width : imgwidth , height : imgheight , borderRadius : borderRadius }"
          @click.stop="showMenu(index)"
          v-for="(item,index) in picArr"
          v-lazy="item"/>
        <div :style="{ width : imgwidth , height : imgheight , borderRadius : borderRadius }"
        v-show="picArr&&picArr.length<max" class="ic-camera" @click.stop="selectImg"><span class="add" @click.stop="selectImg"></span></div>
    </div>
    <input ref="imgIpt" :id="name" class="file" name="file" type="file" accept="image/*"  @change="update" :multiple="envCheck() ? false: 'multiple'"/>
    <actionsheet v-model="isShow" :menus="menu" theme="android" @on-click-menu="menuClick"></actionsheet>
    <div v-transfer-dom>
      <previewer :list="list" ref="previewer"></previewer>
    </div>
  </div>
</template>

<script>
/* 使用方法 */
/*
父组件内容
1.引入：
import vUploadImg from 'src/components/vUploadImg';

2.定义配置
  例如：<vUploadImg name="img2" :picArr="companyInfo.images" max=3 region='z0' fileType="cert"></vUploadImg>
  1.name="img2" //保证组建的唯一性；类似id
  2.:picArr="companyInfo.images" //绑定父组件图片数组 arr
  3.max=3 //max 上传图片的数量
  4.region='z0' //决定上传服务器的地区； 默认 z0 ， 还有 z1\z2\na0
  5.fileType="cert" //上传类型 resource（场地、活动）、cert(证书)、comment(评论等)

功能：1.上传图片、多张上传；2.预览图片；3.更换图片；删除图片

二、头像、使用ref的方式调用更换图片事件
    // 更换图片
    1. 隐藏该组件
        <vUploadImg ref="upload" @success="changeAvatar" fileType="comment" class="profile-avatar-upload" max=1 name="img123" :picArr="[userProfile.avatar]"></vUploadImg>
    2.this.$refs.upload.changeImg()
*/
import { Actionsheet, Previewer, TransferDom } from 'vux';
import {isMobile} from '../config/isMobile.js'
export default {
  directives: {
    TransferDom
  },
  data () {
    return {
      domain:"",
      tokenURL:"",
      uploadURL:'',
      isChange:false,
      isShow:false,
      percent : 0,
      deleteIndex:0,
      menu: [ //图片菜单选项
          {
              label: '查看原图',
              type: 'primary',
              value: 'preview'
          },
          {
              label: '更换图片',
              type: 'warn',
              value: 'change'
          },
          {
              label: '删除图片',
              type: 'warn',
              value: 'delete'
          },
      ]
    }
  },
  components:{ Actionsheet, Previewer },
  props : {
    'name' : {
      default : null ,
      require : true,
      type : String
    },
    'max' : {
      default : 1 ,
      type : String
    },
    'region': {
      default : 'z0',
      type : String
    },
    'fileType' : {
      default : 'cert',
      require : true
    },
    'imgwidth':{
      default : '1.5rem',
      type : String
    },
    'imgheight':{
      default : '1.5rem',
      type : String
    },
    'picArr' : {
      type : Array,
      default : function () {
        return []
      },
      require : true
    },
    'borderRadius' : {
      default : '0',
      type : String
    }
  },
  computed:{
    list:function(){
      let arr = [] ;
      if(this.picArr){
        arr = this.picArr.map(item=>{
          item={
              src:item,
              w:600,
              h:540
          };
          return item;
        })
      }
      return arr
    }
  },
  mounted(){
    this.initConfig()
  },
  methods:{
    // 检查是否是微信内置浏览器
    envCheck(){
      let ua = navigator.userAgent.toLowerCase();
      if( ua.match(/MicroMessenger/i) == "micromessenger" ) {
        return true;
      }else {
        return false;
      }
    },
    initConfig(){
        let { fileType ,region ,max ,picArr }=this.$props;
        // this.picArr=this.$props.picArr||[]
        //console.log(fileType,region)
        /*
          七牛图片上传地址

          // 场地、活动token
          "https://api.linhuiba.com/qiniu/app-token"
          // 上传场地、活动域名
          "https://img.linhuiba.com"

          // 证书token
          "https://api.linhuiba.com/qiniu/app-token/linhuiba-certs"
          // 上传证书域名
          "https://cert.linhuiba.com"

          // 评论token
          "https://api.linhuiba.com/qiniu/app-token/linhuiba-fields"
          // 上传评论照片域名
          "https://comment.linhuiba.com"
        */
        let tokenURL = 'https://api.linhuiba.com/qiniu/app-token'
        let  domain="https://img.linhuiba.com"
        if(fileType=='resource'){
            tokenURL = 'https://api.linhuiba.com/qiniu/app-token'
            domain="https://img.linhuiba.com"
          }
          if(fileType=='cert'){
            tokenURL = 'https://api.linhuiba.com/qiniu/app-token/linhuiba-certs'
            domain="https://cert.linhuiba.com"
          }
          if(fileType=='comment'){
            tokenURL = 'https://api.linhuiba.com/qiniu/app-token/linhuiba-fields'
            domain="https://comment.linhuiba.com"
          }
          this.tokenURL=tokenURL
          this.domain=domain
          this.setUploadURL()
      },
    setUploadURL() {
          let {region='z0'}=this.$data;
          let uploadURL = null;
          switch(region) {
              case 'ECN': uploadURL = 'https://up.qbox.me'; break;//长传图片用的此链接
              case 'NCN': uploadURL = 'https://up-z1.qbox.me'; break;
              case 'SCN': uploadURL = 'https://up-z2.qbox.me'; break;
              case 'NA': uploadURL = 'https://up-na0.qbox.me'; break;
              case 'z0': uploadURL = 'https://upload.qiniup.com'; break;//长传图片用的此链接
              case 'z1': uploadURL = 'https://upload-z1.qiniup.com'; break;//长传图片用的此链接
              case 'z2': uploadURL = 'https://upload-z2.qiniup.com'; break;//长传图片用的此链接
              case 'na0': uploadURL = 'https://upload-na0.qiniup.com'; break;//长传图片用的此链接
              default: //console.error('please make the region is with one of [ECN, SCN, NCN, NA,z0,z1,z2,na0]');
          }
          this.uploadURL=uploadURL;
      },
    update(e){
      let {max,picArr}=this.$props ;
      let files = Array.from( e.target.files);
      //console.log(e.target.files.length,picArr.length,max)
      let len = e.target.files.length ;
      if (this.isChange) {
        console.log(len+picArr.length, +max + 1)
        if(len+picArr.length> +max + 1){
          this.$vux.toast.text(`最多还可上传${max-picArr.length}张图片`, 'middle');
          return;
        }
      } else {
        if(len+picArr.length> +max){
          this.$vux.toast.text(`最多还可上传${max-picArr.length}张图片`, 'middle');
          return;
        }
      }
      // if(len+picArr.length>max&&!this.isChange){
      //     this.$vux.toast.text(`最多还可上传${max-picArr.length}张图片`, 'middle');
      //     return;
      // }
      this.uploading(files)
    },
    uploading(files){
      let that = this ;
      let file = files.shift();
      let filetype = file.type ;
      let reg = /(image\/jpeg)|(image\/svg\+xml)|(image\/gif)|(image\/jpg)|(image\/png)/gi ;
      if( !reg.test( filetype ) ){
        this.$vux.toast.text('仅支持.png|.jpeg|.jpg|.svg|.gif后缀的文件类型','middle');
        return false ;
      }
      let param = new FormData(); //创建form对象
      param.append('chunk','0');//断点传输
      param.append('chunks','1');
      param.append('file',file,file.name) ;

      //console.log(param.get('file')); //FormData私有类对象，访问不到，可以通过get判断值是否传进去
      //先从自己的服务端拿到token
      this.$vux.loading.show({
        text: '上传中'
      });
      this.$http.get(this.tokenURL).then( res => {
        // console.log( res.data )
        let token = res.data.result;
        param.append('token',token);
        this.$http.post(this.uploadURL,param,{
          headers:{'Content-Type':'multipart/form-data'},
          onUploadProgress : (evt) => {
            // 上传进度
            if ( evt.lengthComputable ) {
              let percentComplete = Math.round(evt.loaded * 100 / evt.total);
              this.percent = percentComplete ;
            }
            else {
              this.$vux.toast.text('无法获取上传进度', 'middle');
            }
          }
        }).then( response =>{
          // console.log(response)
            this.$vux.loading.hide();
            if(this.isChange){
              this.isChange = false ;
              this.$set(this.picArr,this.deleteIndex,`${this.domain}/${response.data.key}`)
            }else{
              this.picArr.push(`${this.domain}/${response.data.key}`);
            }
            // 上传成功立即返回结果
            this.$emit('success',[`${this.domain}/${response.data.key}`]);
          // 多张上传
          if( files.length ){
            that.uploading(files);
          }else{
            this.$vux.toast.text('图片上传成功', 'middle');
            return ;
          }
        }).catch( e =>{
          this.$vux.toast.text('图片上传出错，请重试', 'middle');
          this.$emit('error',e) ;
          return;
        })
      })
    },
    selectImg(){
      if( this.$refs.imgIpt.value ){
        this.$refs.imgIpt.value= ''
      }
      this.$refs.imgIpt.dispatchEvent(new MouseEvent('click'));
    },
      // 预览
    showMenu(index) {
          this.isShow = true;
          this.deleteIndex = index
      },
    menuClick(menuKey, menuItem) {
          //console.log(menuKey, menuItem)
          if (menuKey === 'delete') {
              this.picArr.splice(this.deleteIndex,1)
          }
          if (menuKey === 'preview') {
              this.preview()
          }
          if (menuKey === 'change') {
              this.changeImg()
          }
      },
    changeImg(e) {
          this.isChange = true ;
          this.selectImg(e)
      },
    preview() {
          this.$refs.previewer.show(this.deleteIndex)
      },
  }
}

</script>

<style lang="scss" scoped>
  @import '../style/mixin';
  .file-wrap{
    overflow: hidden;
    .file{
      z-index:-99999;
      visibility: hidden;
      opacity:0;
      height:0;
      position: absolute;
    }
    .upload-image {
      display: flex;
      flex-wrap:wrap;
      padding-bottom: 0.2rem;
      .vux-x-icon {
        fill: #999;
        margin-right: 0.1rem;
      }
      .ic-camera {
        position: relative;
        display: flex;
        justify-content: center;
        align-items: center;
        background-color: #f1f2f6;
        .add{
          @include wh(0.72rem,0.72rem);
          @include bg-image('../images/ic_sahngchuangtupian_three_onezero');
        }
      }
      img {
        z-index:2;
        margin-right: 0.1rem;
        margin-bottom: 0.1rem;
        background:rgba(0,0,0,0.05);
      }
    }
  }
</style>
