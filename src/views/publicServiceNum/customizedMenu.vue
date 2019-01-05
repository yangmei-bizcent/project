<template>
  <div class="page-container page-custommenu bgcolor-white">
    <div v-show="!operateResult" class="page-title flex-alignstart-justifystart">
      <span>自定义菜单</span>
      <span class="marginleft30 fontsize0 color-999">对微信服务号的底部菜单以及子菜单的名称、链接地址配置，将应用链接通过可视化操作配置到菜单中</span>
    </div>
    <!-- operateResult为true，显示发布结果，隐藏菜单编辑页面；false不显示发布结果，显示菜单编辑页面 -->
    <div class="paddingX20 padding20X">
      <table v-show="!operateResult" class="width100">
        <tr>
          <td class="valigntop" width="346px">
            <div v-loading="loadingMenu" class="wrap-custommenu-left textAlignCenter relative">
              <div class="color-white paddingtop30 textAlignCenter left-wxtitle">{{ wxTitle }}</div>
              <!-- 拖拽排序提示 -->
              <div v-show="sortnoticeShow" class="wrap-sortnotice absolute" :style="'bottom:'+ sortNoticeBottom+'px'">
                <div class="con-sortnotice borderRadius2 textAlignLeft relative">
                  <p class="fontsize0 color-white">可以对菜单进行拖拽排序。</p>
                  <i class="el-icon-close color-white fontsize0 absolute cursorPointer" @click="closeSortNotice">
                  </i>
                  <i class="el-icon-caret-bottom fontsize8 absolute"></i>
                </div>
              </div>
              <div class="width100 height50 absolute bgcolor-white fontsize1 color-333 view-custm-menu">
                <el-row type="flex" justify="space-between">
                  <el-col :span="8" class="flex-align-justify width53">
                    <i class="icon-keyboard"></i>
                  </el-col>
                  <el-col :span="16" class="wrap-menus height50">
                    <div class="flex-align-justify height50">
                      <table class="width100 height-per100" cellspacing="0" cellpadding="0">
                        <tr>
                          <td 
                            :id="'menu'+index" 
                            v-if="menuLength > 0" 
                            v-for="(mainItem, index) in menuList" 
                            v-dragging="{item: mainItem, list: menuList, group: 'menu'}"
                            :key="index" 
                            class="textAlignCenter" 
                            :style="(menuLength<3)?'width:'+100/(menuLength+1)+'%;':'width:'+100/menuLength+'%;' ">
                            <div class="relative flex-align-justify height-per100">
                              <div class="width100 cursorPointer flex-align-justify paddingX5 marginleft-1 wrap-menu-content"
                                @click="menuClick(index, mainItem)">
                                <i class="icon-moremenu"></i>
                                <p class="moreLineEllipsis1">{{ mainItem.name }}</p>
                              </div>
                              <section :id="'submenu'+index" class="width100 absolute bgcolor-white displayNone wrap-submenu-list">
                                <ul>
                                  <li v-if="mainItem.sub_button && mainItem.sub_button.length>0" v-for="(subItem, subIndex) in mainItem.sub_button"
                                    v-dragging="{item: subItem, list: mainItem.sub_button, group: 'submenu'+index}"
                                    :key="subIndex" @click="submenuItemClick(index, subItem, subIndex)" class="submenu-list-item cursorPointer"
                                    :id="'submenu'+index+subIndex" :class="'clickedMenu'+index+subIndex">
                                    <div class="lineEllipsis">{{ subItem.name ? subItem.name:"子菜单名称" }}</div>
                                  </li>
                                  <li v-if="!mainItem.sub_button || mainItem.sub_button.length<5" @click="addSubmenu(index)"
                                    :id="'submenuAdd'+index" class="submenu-list-item cursorPointer submenuAdd">
                                    <div>
                                      <i class="el-icon-plus"></i>
                                    </div>
                                  </li>
                                </ul>
                              </section>
                            </div>
                          </td>
                          <td v-show="menuLength<3" class="textAlignCenter cursorPointer" :style="'width:'+ 100/(menuLength+1) +'%;'"
                            @click="addMainMenu">
                            <div class="flex-align-justify wrap-menu-content">
                              <i class="el-icon-plus"></i>
                            </div>
                          </td>
                        </tr>
                      </table>
                    </div>
                  </el-col>
                </el-row>
              </div>
            </div>
          </td>
          <td class="valigntop">
            <div class="minHeight640 relative">
              <!-- <div v-show="noClickedMenu" class="textAlignCenter height-per100 margintop200 fontsize1 color-333">点击左侧菜单进行编辑操作</div>
              <div v-show="!noClickedMenu" class="wrap-custommenu-right"> -->
              <div v-show="rightMenuFormShow" class="wrap-custommenu-right">
                <div>
                  <header class="header-menu-edit paddingleft20 marginbottom15">
                    <div class="width100 flex-align-spacebetween">
                      <div class="width100 fontsize6 color-4a4a4a">
                        <table class="width100">
                          <tr>
                            <td valign="middle" class="paddingright30">
                              <table>
                                <tr>
                                  <td>
                                    <span class="moreLineEllipsis1">{{button.name}}</span>
                                  </td>
                                  <td>
                                    <span class="fontsize0 color-999 marginleft30 nowrap" v-show="menuHasSub">已添加子菜单，仅可设置菜单名称</span>
                                  </td>
                                </tr>
                              </table>
                            </td>
                            <td align="right" valign="middle">
                              <el-button type="text" v-show="button.p==0" @click="deleteMenu">删除菜单</el-button>
                              <el-button type="text" v-show="button.p==1" @click="deleteMenu">删除子菜单</el-button>
                            </td>
                          </tr>
                        </table>
                      </div>
                    </div>
                  </header>
                  <section class="paddingleft20" v-if="button">
                    <el-form :model="button" :rules="rules" ref="button" label-width="100px" label-position="left">
                      <el-form-item :label="button.p==0?'菜单名称':'子菜单名称'" prop="name">
                        <el-input v-model="button.name" :placeholder="button.p==0?'输入菜单名称':'输入子菜单名称'" class="input-menu">
                        </el-input>
                        <p class="entryTip" v-text="button.p == 0 ? '字数不超过4个汉字或8个字母' : '字数不超过7个汉字或14个字母'"></p>
                      </el-form-item>
                      <el-form-item v-if="button.type !=null" :label="button.p==0?'菜单内容':'子菜单内容'" prop="type">
                        <el-radio-group v-model="button.type" @change="defaultRadio()">
                          <!-- <el-radio :label="'click'">发送消息</el-radio> -->
                          <el-radio :label="'view'">跳转页面</el-radio>
                        </el-radio-group>
                      </el-form-item>
                      <!-- START 发送消息 -->
                      <!-- <el-card v-if="button.type == 'click'" class="box-card borderRadius0" shadow="never">
                        <div slot="header" class="clearfix">
                          <div class="flex-align-justifystart">
                            <div class="flex-align-justifystart cursorPointer" @click="showMenuEditPanel('text')">
                              <i class="menutype txt"></i>
                              <span>文字</span>
                            </div> -->
                            <!-- <div class="flex-align-justifystart cursorPointer" @click="showMenuEditPanel('textimg')">
                              <i class="menutype textimg"></i>
                              <span>图文消息</span>
                            </div> -->
                            <!-- <div class="flex-align-justifystart cursorPointer marginleft50" @click="showMenuEditPanel('text')">
                              <i class="menutype txt"></i>
                              <span>文字</span>
                            </div> -->
                            <!-- <div class="flex-align-justifystart cursorPointer marginleft50" @click="showMenuEditPanel('img')">
                              <i class="menutype img"></i>
                              <span>图片</span>
                            </div>
                            <div class="flex-align-justifystart cursorPointer marginleft50" @click="showMenuEditPanel('audio')">
                              <i class="menutype audio"></i>
                              <span>语音</span>
                            </div>
                            <div class="flex-align-justifystart cursorPointer marginleft50" @click="showMenuEditPanel('video')">
                              <i class="menutype video"></i>
                              <span>视频</span>
                            </div> -->
                          <!-- </div>
                        </div>
                        <div class="wrap-editmenu"> -->
                          <!-- 图文消息编辑区域 -->
                          <!-- <div v-show="editImgtextModuleShow">
                            <div class="flex-align-justifystart fontsize0 color-666">
                              <div class="flex-align-justify-column cursorPointer borderRadius4 wrap-imgtext-tool">
                                <i class="el-icon-plus fontsize30 color-999"></i>
                                <p class="margintop10">从素材库中选择</p>
                              </div>
                              <div class="flex-align-justify-column cursorPointer borderRadius4 marginleft15 wrap-imgtext-tool">
                                <i class="el-icon-plus fontsize30 color-999"></i>
                                <p class="margintop10">新建图文消息</p>
                              </div>
                            </div>
                          </div> -->
                          <!-- 文字编辑区域 -->
                          <!-- <div v-show="editTextModuleShow">
                            <el-input type="textarea" :rows="12" class="noborder-textarea" placeholder="请输入内容">
                            </el-input>
                          </div> -->
                          <!-- 图片编辑区域 -->
                          <!-- <div v-show="editImgModuleShow">
                            <div class="flex-align-justifystart fontsize0 color-666">
                              <div class="flex-align-justify-column cursorPointer borderRadius4 wrap-imgtext-tool">
                                <i class="el-icon-plus fontsize30 color-999"></i>
                                <p class="margintop10">从素材库中选择</p>
                              </div>
                              <div class="flex-align-justify-column cursorPointer borderRadius4 marginleft15 wrap-imgtext-tool">
                                <el-upload action="https://jsonplaceholder.typicode.com/posts/" list-type="picture-card"
                                  :limit="1" :on-preview="handlePictureCardPreview" :on-remove="handleRemove"
                                  :on-progress="onUploadProcess" :on-success="handlesuccess" :before-upload="beforeAvatarUpload"
                                  :on-exceed="handleExceed" class="relative flex-align-justifystart uploadContainer">
                                  <i class="el-icon-plus fontsize30 color-999"></i>
                                  <p class="margintop10">上传图片</p>
                                </el-upload>
                                <el-dialog :visible.sync="largeImgDlgVisible">
                                  <img width="100%" :src="dialogImgViewUrl" alt="">
                                </el-dialog>
                              </div>
                            </div>
                          </div> -->
                          <!-- 音频编辑区域 -->
                          <!-- <div v-show="editAudioModuleShow">
                            <div class="flex-align-justifystart fontsize0 color-666">
                              <div class="flex-align-justify-column cursorPointer borderRadius4 wrap-imgtext-tool">
                                <i class="el-icon-plus fontsize30 color-999"></i>
                                <p class="margintop10">从素材库中选择</p>
                              </div>
                              <div class="flex-align-justify-column cursorPointer borderRadius4 marginleft15 wrap-imgtext-tool">
                                <i class="el-icon-plus fontsize30 color-999"></i>
                                <p class="margintop10">新建语音</p>
                              </div>
                            </div>
                          </div> -->
                          <!-- 视频编辑区域 -->
                          <!-- <div v-show="editVideoModuleShow">
                            <div class="flex-align-justifystart fontsize0 color-666">
                              <div class="flex-align-justify-column cursorPointer borderRadius4 wrap-imgtext-tool">
                                <i class="el-icon-plus fontsize30 color-999"></i>
                                <p class="margintop10">从素材库中选择</p>
                              </div>
                              <div class="flex-align-justify-column cursorPointer borderRadius4 marginleft15 wrap-imgtext-tool">
                                <i class="el-icon-plus fontsize30 color-999"></i>
                                <p class="margintop10">新建视频</p>
                              </div>
                            </div>
                          </div>
                        </div>
                      </el-card> -->
                      <!-- END 发送消息 -->
                      <!-- START 跳转页面 -->
                      <el-form v-show="button.type == 'view'" :rules="rules" ref="hrefForm">
                        <el-form-item label="">
                          <el-radio-group v-model="chooseUrl">
                            <el-radio :label="0">
                              <div class="inlineBlock width120">可选服务页面：</div>
                              <el-select v-model="selectUrl" :disabled="chooseUrl==0?false:true" placeholder="请选择">
                                <el-option v-for="(item,index) in serviceHrefOptions" :key="index" :label="item.applicationName"
                                  :value="item.applicationUrl">
                                </el-option>
                                <!-- <el-option label="item.applicationName"
                                  value="item.applicationId">
                                </el-option> -->
                              </el-select>
                              <div class="entryTip marginleft152 whitespace-normal">{{selectUrl}}</div>
                            </el-radio>
                            <br>
                            <el-radio :label="1" class="custom-page">
                              <div class="inlineBlock width120 custom-page-title">自定义页面地址：</div>
                              <el-form-item class="legal-link">
                                <el-input v-model="customUrl" :disabled="chooseUrl==1?false:true" placeholder="请输入合法页面链接" class="input-menu">
                                </el-input>
                              </el-form-item>
                            </el-radio>
                          </el-radio-group>
                        </el-form-item>
                      </el-form>
                      <!-- END 跳转页面 -->
                    </el-form>
                  </section>
                </div>
              </div>
              
              <div class="flex-align-justifystart marginleft24 margintop30 absolute bottom0">
                <!-- <el-button type="primary" plain @click="submitForm('menuForm')">保存</el-button> -->
                <el-button type="primary" plain @click="regainHisMenu" v-show="historyBtnShow">恢复为历史版本</el-button>
                <el-button type="primary" plain @click="previewMenu">预览</el-button>
                <!-- <el-button type="primary" plain>放弃当前修改</el-button> -->
                <el-button type="primary" @click="publishMenu" v-show="releaseBtnShow">发布</el-button>
              </div>
            </div>
          </td>
        </tr>
      </table>
    </div>
    <!-- 预览菜单弹出框 -->
    <el-dialog width="400px" class="dlg-nowrap" :visible.sync="previewMenuShow">
      <div class="dy-phone">
        <div class="dy-phone-box" style="overflow:auto;">
          <div class="dy-phone-content">
            <div class="flex-align-justify-column">
              <div class="wrap-custommenu-left textAlignCenter relative">
                <div class="color-white paddingtop20 textAlignCenter left-wxtitle">{{ wxTitle }}</div>
                <div class="width100 height50 absolute bgcolor-white fontsize1 color-333 view-custm-menu">
                  <el-row type="flex" justify="space-between">
                    <el-col :span="8" class="flex-align-justify width53">
                      <i class="icon-keyboard"></i>
                    </el-col>
                    <el-col :span="16" class="wrap-menus height50">
                      <div class="flex-align-justify height50">
                        <table class="width100 height-per100" cellspacing="0" cellpadding="0">
                          <tr>
                            <td :id="'menu'+index" v-if="menuLength > 0" v-for="(mainItem, index) in menuList"
                              :key="index" class="textAlignCenter" :style="'width:'+100/menuLength+'%;'">
                              <div class="relative flex-align-justify height-per100">
                                <div class="width100 cursorPointer flex-align-justify paddingX5 marginleft-1 wrap-menu-content"
                                  @click="previewMenuClick(index, mainItem)">
                                  <i class="icon-moremenu"></i>
                                  <p class="moreLineEllipsis1">{{ mainItem.name }}</p>
                                </div>
                                <section :id="'submenu_dlg'+index" class="width100 absolute bgcolor-white displayNone wrap-submenu-list">
                                  <ul>
                                    <li v-if="mainItem.sub_button && mainItem.sub_button.length>0" v-for="(subItem, subIndex) in mainItem.sub_button"
                                      :key="subIndex" class="submenu-list-item cursorPointer"
                                      :id="'submenu'+index+subIndex" :class="'clickedMenu'+index+subIndex">
                                      <div class="lineEllipsis">{{ subItem.name ? subItem.name:"子菜单名称" }}</div>
                                    </li>
                                  </ul>
                                </section>
                              </div>
                            </td>
                          </tr>
                        </table>
                      </div>
                    </el-col>
                  </el-row>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </el-dialog>
    <!-- 恢复历史版本 -->
    <!-- 预览菜单弹出框 -->
    <el-dialog title="恢复为历史版本" width="800px" :visible.sync="regainHisMenuShow">
      <div class="line-e7e7e7"></div>
      <p class="fontsize0 color-333 marginbottom15">
        显示文件范围：最近
        <el-input type="tel" v-model="hisDataCount" class="hisInput">
        </el-input>
        条
        <span class="marginleft38">仅将历史版本菜单数据恢复至编辑状态，需要发布后同步线上版本</span>
      </p>
      <el-table :data="hisMenuData" class="width100" height="350" highlight-current-row @current-change="selectedHisMenuChange">
        <el-table-column prop="createdon" label="自动备份日期/时间"></el-table-column>
        <el-table-column property="releaseComment" label="发布操作者"></el-table-column>
      </el-table>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" plain @click="regainHisMenuShow = false">取 消</el-button>
        <el-button type="primary" @click="regainConfirm">恢复</el-button>
      </span>
    </el-dialog>
    <!-- 发布结果 operateResult为true，显示发布结果；false不显示发布结果 -->
    <OperationResult v-show="operateResult" :operateSuccess="operateSuccess" @successBack="successBack" @publishAgain="publishAgain"></OperationResult>
  </div>
</template>

<script>
  import OperationResult from '../../components/operationResult'
  import draggable from 'vuedraggable'
  import { Loading } from 'element-ui';
  import {mapState, mapGetters} from 'vuex'
  import fecha from 'fecha'
  export default {
    computed: {
      ...mapState({
        // menuList:state=>JSON.parse(JSON.stringify(state.wechat.menuConfigInfo)).button,
        hisMenuData: state => state.wechat.wechatMenuHistoryList,
        serviceHrefOptions: state => state.application.serviceHrefOptions
      }),
      ...mapGetters({moduleper:'moduleper'}),
    },
    data() {
      var validateSubmenuName = (rule, value, callback) => {
        if (!value) {
          return callback(new Error('请输入子菜单名称'));
        }
        return callback();
      };

      let validateName = (rule, value, callback) => {
        //获取输入框字节长度 （采取将255以外的字符替换成"aa"的做法，取长度）
        function getByteLength(str) {
          return str.replace(/[^\x00-\xff]/g,"aa").length;
        } 

        let length = this.button.p == 0 ? 8 : 14
        // let errorMsg = this.button.p == 0 ? 8 : 14
        if(getByteLength(value) > length) {
          callback(new Error(`不超过${length / 2}个汉字或${length}个字母`))
        }
        callback()
      }
      return {
        loadingMenu: true,
        customUrl: '',
        selectUrl: '',
        chooseUrl: 0,
        menuList: [],
        // START-------- 发布结果
        operateResult: false, // 是否显示发布结果页
        operateSuccess: true, // 发布是否成功 成功true，失败false
        // END-------- 发布结果
        rightMenuFormShow: false, // 右侧表单区域是否显示
        // noClickedMenu: true, // 刚进入页面，未点击菜单，右侧不显示表单内容
        menuHasSub: false, // 编辑主菜单时，标题后面的提示‘已添加子菜单，仅可设置菜单名称’是否显示

        hasMenuChange: false, // 标记菜单是否有更改
        wxTitle: '', // 左侧微信顶部标题
        menuLength: 0,
        sortnoticeShow: true, // 菜单拖拽排序提示显示
        sortNoticeBottom: 70, // 排序提示定位的bottom值

        // submenuFormShow: false, // 子菜单编辑表单
        // menuFormShow: true, // 主菜单编辑表单

        editImgtextModuleShow: false, // 图文消息编辑区域
        editTextModuleShow: true, // 文字编辑区域
        editImgModuleShow: false, // 图片编辑区域
        editAudioModuleShow: false, // 音频编辑区域
        editVideoModuleShow: false, // 视频编辑区域
        menuSort: false, //是否默认菜单序列
        menuConTypeMsgShow: true, // 菜单内容‘发送消息’编辑区域是否显示
        // 子菜单 图片 编辑
        dialogImgViewUrl: '',
        largeImgDlgVisible: false,

        // 预览菜单弹窗显示
        previewMenuShow: false,
        // 恢复历史版本弹窗显示
        regainHisMenuShow: false,
        hisDataCount: 10, //历史版本 显示文件范围
        // 菜单历史版本数据
        selectedHisMenuRow: null, // 弹出框选中的历史版本
        button: {

        },
        rules: {
          editSubmenuName: [{
            validator: validateSubmenuName,
            trigger: 'blur'
          }],
          name: [
             { required: true, message: '请输入子菜单名称', trigger: 'blur' },
             { max: 14, message: '不超过7个汉字或14个字母', trigger: 'blur' },
             { validator: validateName, trigger: 'blur' },
          ],
          customUrl: [
            { required: true, message: '请输入合法页面链接', trigger: 'blur' }
            // { validator: validateCustomUrl, trigger: 'blur'}
          ]
        },
        //权限控制
        historyBtnShow:false,
        releaseBtnShow:false,
      }
    },
    async mounted() {
      // let loadingMenu = Loading.service();
      this.Jurisdiction();
      await this.$store.dispatch({
        type: 'wechatInfo'
      })
      //console.log("this.$store.getters.wechatInfo", this.$store.getters.wechatInfo)
      this.wxTitle = this.$store.getters.wechatInfo.nickName
      await this.$store.dispatch({
        type: 'wxMenuConfig'
      }).then(() => {
        // loadingMenu.close();
        this.loadingMenu = false;
        this.menuList = JSON.parse(JSON.stringify(this.$store.getters.menuConfigInfo)).button;
        this.menuLength = this.menuList ? this.menuList.length : 0;
        switch(this.menuLength) {
          case 0:
            this.rightMenuFormShow = false;
            break;
          default:
            // START---有一级菜单的话，初始化右侧表单内容为第一个主菜单的信息
            this.rightMenuFormShow = true;
            // this.initMenuRightForm(0);
            let menuItem0 = this.menuList[0];
            this.$nextTick(() => {
              document.getElementById('menu0').style.border = '1px solid #35bf35';
            })
            if (menuItem0.sub_button && menuItem0.sub_button.length > 0) {
              // 有子菜单的话，主菜单右侧编辑区域只能修改主菜单名称
              this.menuHasSub = true;
            } else {
              this.menuHasSub = false;
            }
            this.button = menuItem0;
            this.button.p = 0;
            this.button.editMenuIndex = 0
            this.defaultRadio();
            // END---初始化右侧表单内容为第一个主菜单的信息
            break;
        }
      });
      await this.$store.dispatch({
        type: 'serviceHrefOptionsWx',
        req: {
          platformHospitalId: JSON.parse(sessionStorage.getItem("chooseOrgInfo")).platformHospitalId
        }
      })
      //console.log('applicationUrl ', this.serviceHrefOptions)
      // 拖拽排序
      this.$dragging.$on('dragged', ({ value }) => {
        //console.log('开始拖拽');
      })
      this.$dragging.$on('dragend', () => {
        //console.log('拖拽结束');
        // //console.log(JSON.stringify(this.menuList[2].sub_button));  
      })
      //console.log('菜单数据：', this.menuList);
    },
    components: {
      OperationResult,
      draggable,
    },
    // 组件内导航钩子，处理表单未保存退出的情况
    beforeRouteLeave: function (to, from, next) {
      if (this.hasMenuChange) {
        next(false)
        this.$confirm('系统将不会保存您所做的更改，确定要离开此页吗？', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          // 选择确定
          next()
        })
      } else {
        next()
      }
    },
    methods: {
      Jurisdiction(){
        //权限控制
        // //console.log('jinlai',this.moduleper)
        if((this.moduleper!=undefined || this.moduleper!=null) && this.moduleper.length>0){
          this.moduleper.map(item=>{
            item.children.map(item2=>{
              if(item2.feature_id==351013){//"发布自定义菜单"
                this.releaseBtnShow=true;
              }else if(item2.feature_id==351012){//"恢复为历史版本"
                this.historyBtnShow=true;
              }

            })
          })
        }
      },
      dateFormater: function(row, column, cellValue){
        return cellValue ? fecha.format(new Date(cellValue), 'YYYY-MM-DD hh:mm') : '';
      },
      // 拖拽排序
      // 关闭排序提示
      closeSortNotice() {
        this.sortnoticeShow = false;
      },
      // 添加一级菜单
      addMainMenu() {
        // this.noClickedMenu = false;
        this.hasMenuChange = true;
        this.button = {
          name: '菜单名称',
          type: 'click',
          url: '',
          key: '',
          p: 0,
          sub_button: [],
          editMenuIndex: this.menuLength, // 当前编辑的主菜单序号
          editSubmenuIndex: 0, // 当前编辑的子菜单序号 
        }
        this.defaultRadio();
        this.menuList ? this.menuList.push(this.button) : this.menuList = [this.button];
        this.menuLength = this.menuLength + 1;
        this.menuHasSub = false;
      },
      // 一级菜单点击事件
      menuClick(index, item) {
        // this.noClickedMenu = false;
        this.hasMenuChange = true;
        //console.log("menuClick" + index, this.button);

        for (let i = 0; i < this.menuList.length; i++) {
          document.getElementById('menu' + i).style.border = 'none';
        }
        this.removeSubmenuBorder(index);
        document.getElementById('menu' + index).style.border = '1px solid #35bf35';

        if (this.menuList[index].sub_button && this.menuList[index].sub_button.length > 0) {
          // 有子菜单的话，主菜单右侧编辑区域只能修改主菜单名称
          this.menuHasSub = true;
        } else {
          this.menuHasSub = false;
        }

        this.button = item;
        this.button.p = 0;
        this.button.editMenuIndex = index
        let _thissubmenu = document.getElementById('submenu' + index);
        this.defaultRadio();
        for (var i = 0; i < this.menuLength; i++) {
          if (i != index) {
            document.getElementById('submenu' + i).style.display = "none";
            this.sortNoticeBottom = 70;
          }
        }
        //console.log(_thissubmenu.style.display);
        switch (_thissubmenu.style.display) {
          case 'block':
            _thissubmenu.style.display = "none";
            if (index == 0) {
              this.sortNoticeBottom = 70;
            }
            break;
          case 'none':
            _thissubmenu.style.display = "block";
            if (index == 0) {
              let sublen = this.menuList[0].sub_button.length;
              //console.log(sublen, sublen < 5 ? (sublen + 1) * 40 : sublen * 40);
              this.sortNoticeBottom = 80 + (sublen < 5 ? (sublen + 1) * 40 : sublen * 40)
            }
            break;
          case '':
            _thissubmenu.style.display = "block";
            if (index == 0) {
              let sublen = this.menuList[0].sub_button.length;
              this.sortNoticeBottom = 80 + (sublen < 5 ? (sublen + 1) * 40 : sublen * 40)
            }
            break;
        }
      },
      // 子菜单点击事件，点击后右侧表单内容更新为当前子菜单信息
      submenuItemClick(index, subItem, subIndex) {
        this.menuHasSub = false;
        // this.noClickedMenu = false;
        this.hasMenuChange = true;

        this.menuList[index].type = null;
        this.removeSubmenuBorder(index);
        document.getElementById('submenu' + index + '0').style.borderTop = '1px solid #cecece';
        document.getElementById('submenu' + index + subIndex).style.border = '1px solid #35bf35';
        //console.log(subItem)
        // 
        this.button = subItem
        this.button.p = 1;
        this.button.editMenuIndex = index; // 记录当前编辑的主菜单的序号
        this.button.editSubmenuIndex = subIndex; // 记录当前编辑的子菜单的序号
        this.defaultRadio();

      },
      // 清除序号为index的主菜单下的子菜单的高亮
      removeSubmenuBorder(index) {
        if(this.menuList[index].sub_button) {
          let currentSubMenuLen = this.menuList[index].sub_button.length; // 当前主菜单下子菜单的个数
          //console.log('子菜单长度：', currentSubMenuLen);
          for (let i = 0; i < currentSubMenuLen; i++) {
            document.getElementById('submenu' + index + i).style.borderColor = '#cecece';
            document.getElementById('submenu' + index + i).style.borderTop = '1px solid transparent';
            document.getElementById('submenu' + index + i).style.borderBottom = '1px solid transparent';
          }
        }
      },
      // 添加子菜单
      addSubmenu(index) {
        // this.noClickedMenu = false;
        this.hasMenuChange = true;
        let curMainMenu = this.menuList[index];
        let defaultButton = {
          name: '子菜单名称',
          type: 'click',
          url: '',
          key: '',
          p: 1,
          editMenuIndex: index, // 当前编辑的主菜单序号
          editSubmenuIndex: this.menuList[index].sub_button ? this.menuList[index].sub_button.length : 0, // 当前编辑的子菜单序号 
        }
        this.menuList[index].type = null;
        if (curMainMenu.url || curMainMenu.key) {
          this.$confirm('此操作将清除主菜单的配置信息，请确认', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }).then(() => {
            this.menuList[index].url = '';
            this.menuList[index].key = '';
            this.button = defaultButton;
            this.defaultRadio();
            this.menuList[index].sub_button ? this.menuList[index].sub_button.push(this.button) : this.menuList[
              index].sub_button = [this.button];
            if (this.menuList[index].sub_button == 5) {
              document.getElementById('submenuAdd' + index).style.display = "none";
            }
          }).catch(() => {
            return false
          });
        } else {
          this.button = defaultButton;
          this.defaultRadio();
          this.menuList[index].sub_button ? this.menuList[index].sub_button.push(this.button) : this.menuList[index].sub_button = [
            this.button
          ];
          if (this.menuList[index].sub_button == 5) {
            document.getElementById('submenuAdd' + index).style.display = "none";
          }
        }


      },
      // 右侧子菜单 图文/文字/图片/音频/视频 点击事件
      showMenuEditPanel(menutype) {
        var self = this;
        this.editImgtextModuleShow = false;
        this.editTextModuleShow = false;
        this.editImgModuleShow = false;
        this.editAudioModuleShow = false;
        this.editVideoModuleShow = false;
        switch (menutype) {
          case 'textimg':
            this.editImgtextModuleShow = true;
            break;
          case 'text':
            this.editTextModuleShow = true;
            break;
          case 'img':
            this.editImgModuleShow = true;
            break;
          case 'audio':
            this.editAudioModuleShow = true;
            break;
          case 'video':
            this.editVideoModuleShow = true;
            break;
        }
      },
      // START---------右侧子菜单-图片
      // 格式上传
      beforeAvatarUpload(file) {
        const isImg = file.type === 'image/jpeg' || file.type === 'image/png';
        const isLt2M = file.size / 1024 / 1024 < 20;
        if (!isLt2M) {
          this.$message.error('单个上传图片大小不能超过 20MB!');
          return false;
        }
        if (isImg) {
          return true;
        } else {
          this.$alert('请上传pdf，jpg，jpeg，png，doc，docx，xls，xlsx，txt格式的文件', '提示');
          return false;
        }
      },
      handleRemove(file, fileList) {
        //console.log(file, fileList);
      },
      handlePictureCardPreview(file) {
        this.dialogImgViewUrl = file.url;
        this.largeImgDlgVisible = true;
      },
      onUploadProcess() {
        var self = this;
        self.isUploadDisabled = true;
      },
      handlesuccess(file, fileList, res) {
        var self = this;
        self.isUploadDisabled = false;
        //console.log('地址', file);
        self.fileurlarr.push({
          name: fileList.name,
          url: file.extra.url,
          size: fileList.size
        });
        self.fileUrl.push(file.extra.url);
        ////console.log('uploadPC:'+self.fileurlarr);
        //console.log(self.fileUrl);
      },
      handleExceed(files, fileList) {
        // this.$message.warning(`当前限制选择 3 个文件，本次选择了 ${files.length} 个文件，共选择了 ${files.length + fileList.length} 个文件`);
      },
      // END------------右侧子菜单-图片
      // 本地保存当前编辑的子菜单
      submitForm(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            //console.log('submit!');
            // menuForm: {
            //      editMenuIndex: 0,   // 当前编辑的主菜单序号
            //      editSubmenuIndex: 0,    // 当前编辑的子菜单序号
            //      editSubmenuName: '',    // 当前编辑的子菜单名称
            //      submenuContentType: '', // 当前编辑的子菜单类型
            //      submenuContent: 0,  // 当前编辑的子菜单内容
            // }
          } else {
            //console.log('error submit!!');
            return false;
          }
        });
      },
      // 预览菜单
      previewMenu() {
        this.previewMenuShow = true;
      },
      // 预览菜单窗口中主菜单点击显示子菜单
      previewMenuClick(index, item) {
        let _thissubmenu = document.getElementById('submenu_dlg' + index);
        for (var i = 0; i < this.menuLength; i++) {
          if (i != index) {
            document.getElementById('submenu_dlg' + i).style.display = "none";
          }
        }
        //console.log(_thissubmenu.style.display);
        switch (_thissubmenu.style.display) {
          case 'block':
            _thissubmenu.style.display = "none";
            break;
          case 'none':
            _thissubmenu.style.display = "block";
            break;
          case '':
            _thissubmenu.style.display = "block";
            break;
        }
      },
      // 恢复历史版本
      regainHisMenu() {
        if(JSON.parse(sessionStorage.getItem("chooseOrgInfo"))) {
          this.$store.dispatch({
            type: 'wechatMenuHistory',
            req: {
              menutype: "wx",
              platformHospitalId: JSON.parse(sessionStorage.getItem("chooseOrgInfo")).platformHospitalId,
              lastestCount: this.hisDataCount
            }
          }).then(() => {
            this.regainHisMenuShow = true;
            //console.log('hisMenuData', this.hisMenuData)
          });
        }
        else{
          this.$message({
            message: '请先选择机构',
            type: 'warning'
          });
        }
      },
      regainConfirm() {
        this.regainHisMenuShow = false;
        //console.log('确认恢复历史版本');
        //console.log(this.selectedHisMenuRow)
        this.menuList = JSON.parse(this.selectedHisMenuRow.menuJson).button
        // this.noClickedMenu = true; //页面右侧表单隐藏，仅显示提示
        for (let i = 0; i < this.menuLength; i++) {
          document.getElementById('submenu' + i).style.display = 'none';
        }
        this.menuLength = this.menuList.length;
      },
      // 历史版本表格选择行事件
      selectedHisMenuChange(val) {
        //console.log(val);
        this.selectedHisMenuRow = val;
      },
      // 发布
      publishMenu() {
        //console.log("publishMenu", this.menuList)
        if (this.button.type == null || this.button.type == 'click') {
          this.$alert('请选择子菜单内容', '提示', {
            confirmButtonText: '确定',
          })
          return false;
        }
        switch(this.chooseUrl) {
          case 0:
            if(this.button.type=='view' && this.selectUrl == '') {
              this.$alert('请选择可选服务页面地址', '提示', {
                confirmButtonText: '确定',
              })
              return false;
            }
            break;
          case 1:
            if(this.button.type=='view' &&this.customUrl == '') {
              this.$alert('请输入自定义页面地址', '提示', {
                confirmButtonText: '确定',
              })
              return false;
            }
            break;
        }
        this.$refs['button'].validate((valid) => {
          if(!valid) {
            //console.log('error submit')
            return false
          } else if (!this.customUrl  && this.chooseUrl == 1) {
            this.$message.error('请输入自定义页面地址')
            return false
          }
        if(JSON.parse(sessionStorage.getItem("chooseOrgInfo"))) {
          let menu = {
            menuJson: {
              button: this.menuList
            },
            menutype: "wx",
            platformHospitalId: JSON.parse(sessionStorage.getItem("chooseOrgInfo")).platformHospitalId
          }
          this.$store.dispatch({
            type: 'configWxMenu',
            menu: menu
          }).then((d) => {
            if (d.hasOwnProperty('data') && d.data.success) {
              // Message.success("配置成功")
              this.operateResult = true;
              this.operateSuccess = true;
            } else {
              this.$message.error(d.data.errmsg)
            }
          }).catch(() => {
            this.operateResult = false
          })
        }
        else{
          this.$message({
            message: '请先选择机构',
            type: 'warning'
          });
        }
        })
      },
      // 初始化子菜单右侧表单内容
      initSubmenuRightForm(meunIndex, subMenuIndex) {
        let subItem = this.menuList[meunIndex].sub_button[subMenuIndex];
        this.menuList[meunIndex].type = null;
        this.removeSubmenuBorder(meunIndex);
        document.getElementById('submenu' + meunIndex + '0').style.borderTop = '1px solid #cecece';
        document.getElementById('submenu' + meunIndex + subMenuIndex).style.border = '1px solid #35bf35';
        // 
        this.button = subItem
        this.button.p = 1;
        this.button.editMenuIndex = meunIndex; // 记录当前编辑的主菜单的序号
        this.button.editSubmenuIndex = subMenuIndex; // 记录当前编辑的子菜单的序号
        this.defaultRadio();
      },
      // 初始化主菜单右侧表单内容
      initMenuRightForm(meunIndex) {
        let menuItem = this.menuList[meunIndex];
        this.removeSubmenuBorder(meunIndex);
        for (let i = 0; i < this.menuList.length; i++) {
          document.getElementById('menu' + i).style.border = 'none';
        }
        document.getElementById('menu' + meunIndex).style.border = '1px solid #35bf35';
        if (menuItem.sub_button && menuItem.sub_button.length > 0) {
          // 有子菜单的话，主菜单右侧编辑区域只能修改主菜单名称
          this.menuHasSub = true;
        } else {
          this.menuHasSub = false;
        }
        this.button = menuItem;
        this.button.p = 0;
        this.button.editMenuIndex = meunIndex
        this.defaultRadio();
      },
      // 删除菜单后，设置接着高亮的菜单项，以及右侧表单显示高亮菜单项的内容
      initHighlightMenu(meunIndex, subMenuIndex) {
        // meunIndex, subMenuIndex（代表被删除的主菜单编号，或被删除的某个主菜单下的子菜单编号）
        switch(subMenuIndex) {
          case -1:
            // 删除的主菜单
            if(this.menuLength > 0) {
              // 删除的是否是最后一个主菜单
              if(meunIndex==this.menuLength) {
                // 是 删除的最后一个主菜单，删除后设置第一个主菜单高亮
                this.initMenuRightForm(0);
              }else {
                // 否 删除的不是最后一个主菜单，则初始化右侧表单内容为下一项主菜单的内容
                this.initMenuRightForm(meunIndex);
              }
            }else {
              // 主菜单全部被删除的话，右侧表单隐藏
              this.rightMenuFormShow = false;
            }
            break;
          default:
            let subMenuLength = this.menuList[meunIndex].sub_button.length;
            // 删除的子菜单
            if(subMenuLength > 0) {
              // 此主菜单下的子菜单还有余项
                // 删除的子菜单是否是最后一项
                if(subMenuIndex == subMenuLength) {
                  // 是最后一项，则设置第一个此主菜单下的第一个子菜单高亮
                  this.initSubmenuRightForm(meunIndex, 0);
                }else {
                  // 否，不是最后一项，则初始化右侧表单内容为下一项子菜单的内容
                  this.initSubmenuRightForm(meunIndex, subMenuIndex);
                }
            }else {
              // 此主菜单下的子菜单已全部删除，高亮定位到第一个主菜单
              this.initMenuRightForm(0);
            }
            break;
        }
      },
      deleteMenu() {
        this.hasMenuChange = true;
        //console.log("deleteMenu:", this.button);

        let meunIndex = this.button.editMenuIndex;
        let subMenuIndex = this.button.editSubmenuIndex;
        if (this.button.p == 0) {
          // TODO  弹出提示，是否删除主菜单（包含子菜单）
          this.$confirm('是否确认删除该菜单（包含子菜单）?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }).then(() => {
            this.menuList.splice(meunIndex, 1);
            this.menuLength = this.menuLength - 1;
            this.$message({
              type: 'success',
              message: '删除成功'
            });
            // 删除的主菜单，初始化高亮
            if(this.menuLength > 0) {
              this.initHighlightMenu(meunIndex, -1);
            }
          })
        } else {
          // TODO  删除子菜单
          this.$confirm('是否确认删除该菜单?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }).then(() => {
            this.menuList[meunIndex].sub_button.splice(subMenuIndex, 1);
            if(this.menuList[meunIndex].sub_button.length == 0) {
              //console.log('全删完了');
              this.menuList[meunIndex].type='view';
            }
            this.$message({
              type: 'success',
              message: '删除成功'
            });
            // 删除的子菜单，初始化高亮
            this.initHighlightMenu(meunIndex, subMenuIndex);
          })
        }
      },
      // START---------发布结果按钮点击事件
      successBack() {
        //console.log('发布成功返回');
        this.operateResult = false;
        // 发布成功后需要刷新页面，mounted中的初始化代码需要再执行一下
        // await this.$store.dispatch({type:'wechatInfo'})
        // //console.log("this.$store.getters.wechatInfo",this.$store.getters.wechatInfo)
        // this.wxTitle = this.$store.getters.wechatInfo.nickName
        // await this.$store.dispatch({type:'wxMenuConfig'}).then(()=>{
        //     this.menuLength =this.menuList? this.menuList.length:0;
        // }); 
      },
      publishAgain() {
        //console.log('重新发布');
        this.operateResult = false;
        // 重新发布，不需要刷新页面，因为要保留页面填写的信息
      },
      // END---------发布结果按钮点击事件

      defaultRadio() {
        if (this.button.type == "view") {
          this.chooseUrl = 0;
          if (this.serviceHrefOptions) {
            let tmp = this.serviceHrefOptions.find((item) => {
              return item.applicationUrl == this.button.url
            })
            //console.log('tmp', tmp)
            if (tmp) {
              this.selectUrl = tmp.applicationUrl
            } else {
              this.selectUrl = ''
              if (this.button.url) {
                this.chooseUrl = 1;
              }
            }
          }
          this.customUrl = this.button.url
        }
      }
    },
    watch: {
      customUrl: function (newVal, oldVal) {
        if (this.chooseUrl == 1) {
          this.button.url = newVal
        }
      },
      selectUrl: function (newVal, oldVal) {
        if (this.chooseUrl == 0) {
          this.button.url = newVal
        }
      },
      hisDataCount: function (newVal) {
        if (parseFloat(newVal).toString() == "NaN") {
          //alert("请输入数字……");注掉，放到调用时，由调用者弹出提示。
          this.$message.error("请输入一个数字")
          return false;
        } else {
          this.$store.dispatch({
            type: 'wechatMenuHistory',
            req: {
              menutype: "wx",
              platformHospitalId: JSON.parse(sessionStorage.getItem("chooseOrgInfo")).platformHospitalId,
              lastestCount: this.hisDataCount
            }
          })
        }
      }
    }
  }

</script>

<style lang="scss">
  @import '@/assets/css/var.scss';

  .page-custommenu {
    .el-form-item {
      margin-bottom: 10px;
    }

    label.el-radio.custom-page {
      .custom-page-title, span.el-radio__inner {
        margin-top: -10px;
      }
      display: flex;
      // justify-content: space-around;
      align-items: center;
      span.el-radio__label {
        display: flex;
        justify-content: space-around;
        align-items: center;
      }
    }
    // .legal-link.el-form-item {
    //   display: inline
    // }

    .el-form-item__label {
      padding-top: 4px;
    }

    .input-menu {
      width: 300px;
    }

    .width53 {
      width: 53px;
    }

    .width120 {
      width: 120px;
    }
    .height39 {
      height: 39px;
    }

    .height50 {
      height: 50px;
    }
    .minHeight640 {
      min-height: 640px;
    }

    .height-per100 {
      height: 100%;
    }

    .valigntop {
      vertical-align: top;
    }

    .marginleft-1 {
      margin-left: -1px;
    }

    .margintop200 {
      margin-top: 200px;
    }

    .marginleft38 {
      margin-left: 38px;
    }
    .marginleft24 {
      margin-left: 24px;
    }

    .marginleft152 {
      margin-left: 152px;
    }
    .bottom0 {
      bottom: 0;
    }

    .header-menu-edit {
      padding: 9px 0;
      border-bottom: 1px solid $color-e6e6e6;
    }

    .wrap-custommenu-left {
      width: 346px;
      height: 640px;
      background: url('../../assets/img/publicServiceNum/customized-menu-phone.png') center no-repeat;
      background-size: contain;
    }

    .wrap-menu-content {
      height: 36px;
      border-left: 1px solid $color-c1c1c1;
    }

    .wrap-custommenu-right {
      margin-left: 24px;
      padding-bottom: 30px;
    }

    .wrap-menus {
      width: 291px;
    }

    .menu-devide {
      height: 100%;
      border-left: 1px solid $color-c1c1c1;
    }

    .view-custm-menu {
      left: 0;
      bottom: 0;
      border: 1px solid $color-cccccc;
      z-index: 190;

      p {
        margin-left: 8px;
      }
    }

    .icon-keyboard {
      width: 34px;
      height: 34px;
      background: url('../../assets/img/publicServiceNum/customized-menu-keyboard.png') center no-repeat;
      background-size: contain;
    }

    .icon-moremenu {
      width: 8px;
      min-width: 8px;
      height: 7px;
      background: url('../../assets/img/publicServiceNum/customized-menu-moremenu.png') center no-repeat;
      background-size: contain;
    }

    .menutype {
      width: 20px;
      height: 20px;
      margin-right: 8px;
    }

    .textimg {
      background: url('../../assets/img/publicServiceNum/custommenu-textimg.png') center no-repeat;
      background-size: contain;
    }

    .img {
      background: url('../../assets/img/publicServiceNum/custommenu-img.png') center no-repeat;
      background-size: contain;
    }

    .txt {
      background: url('../../assets/img/publicServiceNum/custommenu-text.png') center no-repeat;
      background-size: contain;
    }

    .audio {
      background: url('../../assets/img/publicServiceNum/custommenu-audio.png') center no-repeat;
      background-size: contain;
    }

    .video {
      background: url('../../assets/img/publicServiceNum/custommenu-video.png') center no-repeat;
      background-size: contain;
    }

    .el-card__header {
      height: 47px;
      line-height: 47px;
      padding: 0 24px;
    }

    .wrap-imgtext-tool {
      width: 270px;
      height: 180px;
      background-color: $color-f2f2f2;
    }

    .noborder-textarea {
      textarea {
        border: none;
      }
    }

    .box-card {
      height: 314px;
    }

    .box-card,
    .el-card__header {
      border-color: $color-e6e6e6;
    }

    .wrap-submenu-list {
      min-height: 40px;
      bottom: 50px;
      left: 0;
    }

    .submenu-list-item {
      padding: 0 11px;
      border-left: 1px solid $color-cecece;
      border-right: 1px solid $color-cecece;
      border-top: 1px solid transparent;
      border-bottom: 1px solid transparent;
      margin-bottom: -1px;

      &:first-child {
        border-top: 1px solid $color-cecece;
      }

      &:last-child {
        border-bottom: 1px solid $color-cecece;
      }

      div {
        height: 40px;
        line-height: 40px;
        border-top: 1px solid transparent;
      }

      &+.submenu-list-item {
        div {
          border-top: 1px solid $color-cecece;
        }
      }
    }

    .el-upload--picture-card {
      width: 270px;
      height: 180px;
      line-height: initial;
      background-color: $color-f2f2f2;
      border-radius: 4px;
      border: none;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      float: left;

      * {
        color: #666;
      }

      &:hover {
        * {
          color: #666;
        }
      }
    }

    .el-upload-list--picture-card {
      margin-top: 1px;

      .el-upload-list__item {
        margin-bottom: 0;
        border-color: $color-f2f2f2;
      }
    }

    .uploadContainer {
      height: 180px;

      .el-upload-list__item {
        width: 270px;
        height: 180px;
      }
    }

    .left-wxtitle {
      width: 220px;
      margin-left: 63px;
      overflow: hidden;
      white-space: nowrap;
      text-overflow: ellipsis;
    }

    .el-table {
      border-left: 1px solid #ebeef5;
      border-right: 1px solid #ebeef5;

      th {
        background-color: #E6E6E6;
        font-size: 14px;
        color: #333333;
      }
    }

    .line-e7e7e7 {
      height: 0;
      border-top: 1px solid #e7e7e7;
      margin: -25px -20px 14px -20px;
    }

    .hisInput {
      width: 40px;

      input {
        height: 24px;
        line-height: 24px;
        padding: 0 5px;
      }
    }

    .wrap-sortnotice {
      left: 80px;
      z-index: 200;
    }

    .con-sortnotice {
      width: 230px;
      height: 56px;
      line-height: 56px;
      padding: 0 15px;
      background-color: #424242;
      box-shadow: 0 2px 6px 0 rgba(0, 0, 0, 0.14);

      .el-icon-close {
        right: 5px;
        top: 5px;
      }

      .el-icon-caret-bottom {
        color: #424242;
        bottom: -13px;
        left: 8px;
      }
    }

    .el-table__body tr.current-row>td {
      background-color: #E7F5E5;
    }
    .dlg-nowrap {
      .el-dialog {
        background: transparent;
        box-shadow: none;
      }
      .el-dialog__headerbtn {
        display: none;
      }
      .left-wxtitle {
        margin-left: 33px;
      }
    }
    .dy-phone {
      background: url('../../assets/img/phone.png') no-repeat;
      width: 306px;
      height: 620px;
      z-index: -1;
      .dy-phone-box {
        background: #F6F8FD;
        position: relative;
        cursor: pointer;
        z-index: 1;
        word-break: break-all;
        word-wrap: break-word;
        top: 72px;
        width: 270px;
        height: 477px;
        margin: 0 auto;
      }
      .dy-phone-content {
        position: absolute;
        width: 270px;
        min-height: 477px;
        overflow: hidden;
      }
      .wrap-custommenu-left {
        width: 100%;
        height: 477px;
        background-position-x: center;
        background-size: 100% 100%;
      }
    }
  }

</style>
