<template>
    <!-- 用户动态添加 -->
    <section>
        <el-col :span="24" class="toolbar" style="padding-bottom:0px;">
            <el-form :inline="true" style="overflow:hidden;">
                <el-row>
                    <el-form-item>
                        <div class="block">
                            <span class="registerTime">时间</span>
                            <el-date-picker 
                            style="width:300px;" 
                            v-model="formOne.startDate" 
                            type="daterange" 
                            range-separator=" 至 " 
                            start-placeholder="开始日期" 
                            end-placeholder="结束日期"></el-date-picker>
                        </div>
                    </el-form-item>
                    <el-form-item>
                        <span>发布UID</span>
                        <el-input 
                        style="width:150px;" 
                        v-model="uid" 
                        placeholder="请输入UID"></el-input>
                    </el-form-item>
                     <el-form-item>
                        <span>关键字查找</span>
                        <el-input 
                        style="width:150px;" 
                        v-model="key_word" 
                        placeholder="请输入关键字查找"></el-input>
                    </el-form-item>
                    <el-form-item>
                        <el-button 
                        type="primary" 
                        @click="getData(0)">查询</el-button>
                        <el-button 
                        type="primary"
                        @click="addTopic" >发布新动态</el-button>
                    </el-form-item>
                </el-row>
            </el-form>
        </el-col>
        <template>
            <el-table 
            :data="listData" 
            v-loading="listLoading" 
            border fit highlight-current-row 
            @selection-change="handleSelectionChange"
            style="width:100%;" 
            :height="tableHeight">
                <el-table-column prop="create_time" label="创建时间"></el-table-column>
                <el-table-column prop="create_time" label="发布时间"></el-table-column>
                <el-table-column prop="content_id" label="动态ID" width="50px"></el-table-column>
                <el-table-column prop="topic_id" label="话题ID" width="50px">
                    <template slot-scope="scope">
                        {{scope.row.topic_id==0?'':scope.row.topic_id}}
                    </template>
                </el-table-column>
                <el-table-column prop="uid" label="发布UID"></el-table-column>
                <el-table-column prop="nickname" label="昵称"></el-table-column>
                <el-table-column prop="icon" label="图片">
                    <template slot-scope="scope">
                        <img 
                        :src="scope.row.icon"
                        style="width:100px;height:100px;">
                    </template>
                </el-table-column>
                <el-table-column prop="content" label="内容" width="400px"></el-table-column>
                <el-table-column label="操作">
                    <template slot-scope="scope">
                        <el-row>
                            <el-button 
                            size="mini" 
                            type="primary" 
                            @click="userDetail(scope.$index, scope.row.parmas)">编辑</el-button>
                            <el-button 
                            size="mini" 
                            type="danger"
                            style="margin-left:25px;" 
                            @click="deleteContentRecord(scope.$index, scope.row)">删除</el-button>
                        </el-row>
                    </template>
                </el-table-column>
            </el-table>
            <el-col :span="24" class="toolbar">
                <el-pagination 
                layout="total,prev,pager,next,jumper" 
                :page-size="20" 
                @current-change="handleCurrentChange" 
                :total="totalpage" 
                style="float:right;">
                </el-pagination>
            </el-col>
        </template>
         <!--添加-->
       <el-dialog title="发布新动态" :visible.sync="userInfo.dialogFormVisible">
				<el-form :model="userInfo.params">  
					<el-form-item label="发布UID" :label-width="formLabelWidth">
						<el-input 
						style="width:150px"
						v-model="userInfo.uid" 
                        @keyup.enter.native="reachUser"
						auto-complete="off"></el-input>
                        <span style="color:#ccc;margin-right:20px;">按回车键查找用户</span>
                        <span style="width:180px;display:inline-block;">昵称：{{userInfo.nickName}}</span>
                        <span>图像：<img v-if="userInfo.isShow" :src="userInfo.icon" width="100px;" height="100px;" style="vertical-align: top"/></span>
					</el-form-item>
					<el-form-item label="动态内容" :label-width="formLabelWidth">
						<el-input
							type="textarea"
							:autosize="{ minRows: 2, maxRows: 4}"
							placeholder="请输入内容"
							:maxlength="140"
							v-model="userInfo.content">
						</el-input>
					</el-form-item>
					 <el-form-item label="图片上传" :label-width="formLabelWidth">
                         <div class="fliebox">
                            <div 
                                class="user-photo" 
                                style="margin:10px;margin-right:30px;line-height:14px;" 
                                v-for="(o, index) in userInfo.fileList" 
                                :key="o.id">
                                    <el-popover trigger="click" placement="left">
                                        <img 
                                        :src="o" 
                                        style="width:300px;height:400px;">
                                        <div slot="reference" class="name-wrapper">
                                            <img 
                                            :src="o" 
                                            style="width:150px;height:150px;">
                                        </div>
                                    </el-popover>
                                    <el-button 
                                    type="text" 
                                    class="button" 
                                    @click="deleteAddPhoto(index,userInfo.fileList)" 
                                    size="mini">删除</el-button>
                            </div>
                            <div style="clear:both"></div>
                        </div>
                        <div class="flieBtn">
                            <input 
                            class="filepic fileinput" 
                            multiple 
                            @change="uploading($event, 0)" 
                            id="files"
                            type="file">
                            <span class="showbtn">选择文件</span>
                        </div>
					</el-form-item>
                    <el-form-item label="添加话题" :label-width="formLabelWidth" style="margin-top:40px;">
                        <el-select v-model="userInfo.topic_id" placeholder="请选择话题" style="width:100%;">
                            
                            <el-option :label="item.content" 
                                :value="item.id"
                                :key="item.id"
                                v-for="item in userInfo.topic"></el-option>
                        </el-select>
                    </el-form-item>
					<el-form-item label="发布时间" :label-width="formLabelWidth">
							<el-date-picker 
							v-model="userInfo.create_time" 
							type="datetime" 
							value-format="yyyy-MM-dd HH:mm:ss"
							placeholder="选择日期范围" style="width:100%"></el-date-picker>
					</el-form-item>
				</el-form>
				<div slot="footer" class="dialog-footer">
					<el-button 
					@click="addTopicSure(0)">取 消</el-button>
					<el-button 
					type="primary" 
					@click="addTopicSure(1)">确 定</el-button>
				</div>
			</el-dialog>
              <!--编辑-->
       <el-dialog title="编辑" :visible.sync="editorloading.dialogFormVisible">
				<el-form :model="editorloading">  
					<el-form-item label="发布UID" :label-width="formLabelWidth">
						<el-input 
						style="width:150px"
						v-model="editorloading.uid" 
                        @keyup.enter.native="reachUser"
                        disabled
						auto-complete="off"></el-input>
                        <span style="color:#ccc;margin-right:20px;">按回车键查找用户</span>
                        <span style="width:180px;display:inline-block;">昵称：{{editorloading.nickName}}</span>
                        <span>图像：<img v-if="editorloading.isShow" :src="editorloading.icon" width="100px;" height="100px;" style="vertical-align: top"/></span>
					</el-form-item>
					<el-form-item label="动态内容" :label-width="formLabelWidth">
						<el-input
							type="textarea"
							:autosize="{ minRows: 2, maxRows: 4}"
							placeholder="请输入内容"
							:maxlength="140"
							v-model="editorloading.content">
						</el-input>
					</el-form-item>
					 <el-form-item label="图片上传" :label-width="formLabelWidth">
                         <div class="fliebox">
                            <div 
                                class="user-photo" 
                                style="margin:10px;margin-right:30px;line-height:14px;" 
                                v-for="(o, index) in editorloading.fileList" 
                                :key="o.id">
                                    <el-popover trigger="click" placement="left">
                                        <img 
                                        :src="o" 
                                        style="width:300px;height:400px;">
                                        <div slot="reference" class="name-wrapper">
                                            <img 
                                            :src="o" 
                                            style="width:150px;height:150px;">
                                        </div>
                                    </el-popover>
                                    <el-button 
                                    type="text" 
                                    class="button" 
                                    @click="deletePhoto(index,editorloading.content_id)" 
                                    size="mini">删除</el-button>
                            </div>
                            <div style="clear:both"></div>
                        </div>
                        <div class="flieBtn">
                            <input 
                            class="filepic fileinput" 
                            multiple 
                            @change="uploading($event, 1)" 
                            id="files"
                            type="file">
                            <span class="showbtn">选择文件</span>
                        </div>
					</el-form-item>
                    <el-form-item label="添加话题" :label-width="formLabelWidth" style="margin-top:40px;">
                        <el-select v-model="editorloading.topic_id" placeholder="请选择话题" style="width:100%;">
                            <el-option label=" " value="0"></el-option>
                            <el-option :label="item.content" 
                                :value="item.id"
                                :key="item.id"
                                v-for="item in userInfo.topic"></el-option>
                        </el-select>
                    </el-form-item>
					<el-form-item label="发布时间" :label-width="formLabelWidth">
							<el-date-picker 
							v-model="editorloading.create_time" 
							type="datetime" 
							value-format="yyyy-MM-dd HH:mm:ss"
							placeholder="选择日期范围" style="width:100%"></el-date-picker>
					</el-form-item>
				</el-form>
				<div slot="footer" class="dialog-footer">
					<el-button 
					@click="editorloadingSure(0)">取 消</el-button>
					<el-button 
					type="primary" 
					@click="editorloadingSure(1)">确 定</el-button>
				</div>
			</el-dialog>
    </section>
</template>

<script>
import Event from "./../../../public_js/event";
import { allget } from "../../../api/api";
import store from "../../../vuex/store";
import axios from 'axios';
export default {
    data() {
        return {
            
            tableHeight: null,
            formOne: {
                startDate: [new Date()-7*24*60*60*1000, new Date()]
            },
            formTwo: {
                startDate: [new Date()-7*24*60*60*1000, new Date()]
            },
            recordStatus: "",
            listData: [],
            kickIpData: [],
            kickMobelData: [],
            formLabelWidth: "120px",
            listLoading: false,
            uid: "",
            key_word: "",
            nickname: "",
            operate_user: '',
            page: 1, // 分页
            totalpage: 1000,
            channelData: {}, // 渠道数据
            channelId: null,
            flag:true,
            commentListId: null,
            formData:"",
            userInfo: {
                dialogFormVisible: false, 
                isShow:false,
                nickName: "",
                icon: "",
                uid: "",
                content:"",
                content_id:null,
                data:null,
                fileList:[],
                topic:[],
                topic_id:"",
                pic_list:[],
                create_time:'',
                file_pic:''
            },
            editorloading: {
                dialogFormVisible: false, 
                isShow:true,
                nickName: "",
                icon: "",
                uid: "",
                content:"",
                content_id:null,
                data:null,
                fileList:[],
                create_time:'',
                topic:[],
                new_list:[],
                topic_id:"",
                pic_list:[],
                file_pic:'',
                create_time_spare:'',
                d_pic:0,
            },
            tableHeightOther: '200px'
        };
    },
    methods: {
        handleCurrentChange(val) {
            this.page = val - 1;
            this.getData();
        },
        // 获取数据
        getData() {
            var _this = this;
            _this.listLoading = true;
            let url = "/NewContent/getAdminContentRecordList";
            let params = {
                start_date: baseConfig.changeDateTime(this.formOne.startDate[0], 0),
                end_date: baseConfig.changeDateTime(this.formOne.startDate[1], 0),
                uid: _this.uid,
                page: _this.page,
                keyword: _this.key_word,
            };
            axios.get(allget+url, {params: params})
                .then((res) => {
                   _this.listLoading = false;
                    if (res.data.ret) {
                     //   _this.listData = res.data.data;
                     var arr = [];
                        for(var i=0; i<res.data.data.length; i++) {
                            var obj = {};
                            for(var key in res.data.data[i]) {
                                obj.time = key;
                                obj.create_time = res.data.data[i].create_time;
                                obj.uid = res.data.data[i].uid;
                                obj.content = res.data.data[i].content;
                                obj.icon = res.data.data[i].icon;
                                obj.nickname = res.data.data[i].nickname;
                                obj.content_id = res.data.data[i].content_id;
                                obj.topic_id = res.data.data[i].topic_id;
                                obj.parmas = res.data.data[i];
                            }
                            arr.push(obj);
                        }
                    //_this.formOne.totalPage = arr.length;
                    _this.listData = arr;
                    } else {
                        baseConfig.warningTipMsg(_this, res.data.msg);
                    }
                })
                .catch((err) => {
                    console.error(err);
                });
        },
        //
        addTopic(){
            this.userInfo.dialogFormVisible = true;
            this.getTopic();
        },
        //查找用户
        reachUser(){
            var _this = this;
            if(_this.userInfo.uid == ""){
                baseConfig.warningTipMsg(_this, 'UID不能为空');
                return false;
            }
            _this.listLoading = true;
            let url = "/NewUser/getUserInfo";
            let params = {
                uid : _this.userInfo.uid
            };
            axios.get(allget+url, {params: params})
            .then((res) => {
                 _this.listLoading = false;
                if (res.data.ret) {
                    _this.userInfo.isShow = true;
                    _this.userInfo.nickName = res.data.data['0'].nickname;
                    _this.userInfo.icon = res.data.data['0'].icon;
                } else {
                    baseConfig.warningTipMsg(_this, res.data.msg);
                }
            })
            .catch((err) => {
                console.error(err);
            });
        },
        // 得到上传文件type(0->新增banner，1->编辑banner)
		uploading(event, type) {
            var _this = this;
            var allFiles = event.target.files;
            if(_this.editorloading.fileList.length>=9 || _this.userInfo.fileList.length>=9 || allFiles.length>9 || _this.editorloading.fileList.length+allFiles.length>9){
                baseConfig.warningTipMsg(_this, '图片上传不能超过9张');
                return false;
            };
			if(type==0) {
				_this.userInfo.file_pic = event.target.files;
          	    var windowURL = window.URL || window.webkitURL;
                for(var i=0;i<allFiles.length;i++){
                    _this.userInfo.fileList.push(windowURL.createObjectURL(event.target.files[i]));
                     _this.userInfo.pic_list.push(event.target.files[i]);
                }
			} else if(type==1) {
                _this.editorloading.file_pic = event.target.files;
          	    var windowURL = window.URL || window.webkitURL;
                for(var i=0;i<allFiles.length;i++){
                    _this.editorloading.fileList.push(windowURL.createObjectURL(event.target.files[i]));
                    _this.editorloading.new_list.push(windowURL.createObjectURL(event.target.files[i]));
                    _this.editorloading.pic_list.push(event.target.files[i]);
                }
                
            }
  	    }, 
        //获取上架话题
        getTopic(){
            var _this = this;
            _this.listLoading = true;
            let url = "/NewContent/getActivatedContentTopicList";
            let params = {
                
            };
            axios.get(allget+url, {params: params})
            .then((res) => {
                 _this.listLoading = false;
                if (res.data.ret) {
                    _this.userInfo.topic = res.data.data;
                } else {
                    baseConfig.warningTipMsg(_this, res.data.msg);
                }
            })
            .catch((err) => {
                console.error(err);
            });
        },
        //删除新加图片
        deleteAddPhoto(index, rows){
            this.userInfo.fileList.splice(index, 1);
            this.userInfo.pic_list.splice(index, 1);
        },
        addTopicSure(type){
            var _this = this;
            if(type == 0){ 
                _this.userInfo.dialogFormVisible = false;
                _this.resetForm();
            }else{
                if(_this.flag == false){
                    baseConfig.warningTipMsg(_this, '正在请求数据中，请稍候');
                    return false;
                }
                _this.flag=false;
                _this.listLoading = true;
                let formData = new FormData();
                for(var i=0; i<_this.userInfo.pic_list.length;i++){
                    let file = _this.userInfo.pic_list[i];
                    formData.append('pic_list[' + i + ']', file);
                }
				formData.append('uid', _this.userInfo.uid);
				formData.append('content', _this.userInfo.content);
				formData.append('topic_id', _this.userInfo.topic_id || 0);
			  	formData.append('create_time', _this.userInfo.create_time);
				let config = {
					headers: {
						'Content-Type': 'multipart/form-data'
					}
                };
				axios.post(allget+'/NewContent/setAdminContentRecord', formData, config)
					.then((res) => {
                        _this.listLoading = false;	
                        _this.flag=true;							
						if(res.data.ret) {	
							baseConfig.successTipMsg(_this, '添加成功！');
							_this.getData();
							_this.userInfo.dialogFormVisible = false;
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg);						
						}
						_this.resetForm();
					})
					.catch((err) => {
						console.log(err);
					});
			    }
            
        },
        resetForm(){
            var _this = this;
             _this.userInfo = {
                dialogFormVisible: false, 
                isShow:false,
                nickName: "",
                icon: "",
                uid: "",
                content:"",
                content_id:null,
                data:null,
                fileList:[],
                topic:[],
                topic_id:"",
                pic_list:[],
                file_pic:''
            };
            _this.editorloading = {
                dialogFormVisible: false, 
                isShow:true,
                nickName: "",
                icon: "",
                uid: "",
                content:"",
                content_id:null,
                fileList:[],
                topic:[],
                topic_id:"",
                pic_list:[],
                new_list:[],
                file_pic:'',
                create_time:"",
                d_pic:0,
                create_time_spare:'',
            }
        },
        //编辑
        userDetail(index, rows){
            this.editorloading.dialogFormVisible = true;
            this.getTopic();
            this.editorloading.uid = rows.uid;
            this.editorloading.nickName = rows.nickname;
            this.editorloading.icon = rows.icon;
            this.editorloading.content = rows.content;
            this.editorloading.content_id = rows.content_id;
            this.editorloading.fileList = rows.pic_list;
            this.editorloading.d_pic = rows.pic_list.length;
            this.commentListId = rows.content_id;
            this.editorloading.topic_id = rows.topic_id;
            if(rows.create_time==null||rows.create_time=='null') {//为null时特殊处理
                this.editorloading.create_time = '';
                this.editorloading.create_time_spare  = '';
            } else {
                this.editorloading.create_time = rows.create_time;
                this.editorloading.create_time_spare = new Date(rows.create_time);
            }
        },
        editorloadingSure(type){
            var _this = this;
            if(type == 0){ 
                _this.editorloading.dialogFormVisible = false;
                _this.resetForm();
            }else{
                if(_this.flag == false){
                    baseConfig.warningTipMsg(_this, '正在请求数据中，请稍候');
                    return false;
                }
                _this.flag=false;
                _this.listLoading = true;
				if(_this.editorloading.create_time.toString().indexOf('(中国标准时间)') != '-1'){
					_this.editorloading.create_time = _this.editorloading.create_time_spare;
				}else{
                    _this.editorloading.create_time = _this.editorloading.create_time;
                }
                let formData = new FormData();
                if(_this.editorloading.new_list.length == 0){
                    formData.append('pic_list', '');
                }else{
                    formData.append('pic_list', '');
                    var len = _this.editorloading.new_list.length+_this.editorloading.d_pic;
                    var n = 0;
                    for(var i=_this.editorloading.d_pic; i<len;i++){
                        let file = _this.editorloading.pic_list[n];
                        formData.append('pic_list[' + i + ']', file);
                        n++;
                        console.log(file,789)
                    }
                }
                formData.append('uid', _this.editorloading.uid);
                formData.append('content_id', _this.editorloading.content_id);
				formData.append('content', _this.editorloading.content);
				formData.append('topic_id', _this.editorloading.topic_id);
			  	formData.append('create_time', _this.editorloading.create_time);
				let config = {
					headers: {
						'Content-Type': 'multipart/form-data'
					}
                };
				axios.post(allget+'/NewContent/setAdminContentRecord', formData, config)
					.then((res) => {
                         _this.flag=true;	
						if(res.data.ret) {	
							baseConfig.successTipMsg(_this, '修改成功！');
							_this.getData();
                            _this.editorloading.dialogFormVisible = false;	
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg);						
						}
                        _this.resetForm();
                        _this.listLoading = false;
					})
					.catch((err) => {
						console.log(err);
                    });
                     
			    }
        },
        //删除图片
        deletePhoto(index, content_id){
            var _this = this;
            var addPic = _this.editorloading.new_list;
            var oldPic = '';
            _this.listLoading = true;
            let newArr = _this.listData.filter((item, index) => {
                return item.content_id == _this.commentListId;
            });
            oldPic = newArr[0].parmas.pic_list;
            if(index>=oldPic.length-addPic.length){
                _this.editorloading.fileList.splice(index, 1);
                _this.editorloading.pic_list.splice(index-_this.editorloading.d_pic, 1);
                _this.editorloading.new_list.splice(index-_this.editorloading.d_pic, 1);
            }else{
                let url = "/NewContent/deleteContentRecordElement";
                let params = {
                    content_id : content_id,
                    pic_number : index,
                    type : 2
                };
                axios.get(allget+url, {params: params})
                .then((res) => {
                    _this.listLoading = false;
                    if (res.data.ret) {
                        baseConfig.successTipMsg(_this, res.data.msg);
                        let removePhoto = newArr[0].parmas.pic_list.splice(index,1);
                        _this.editorloading.fileList = newArr[0].parmas.pic_list;
                        _this.getData();
                    } else {
                        baseConfig.warningTipMsg(_this, res.data.msg);
                    }
                })
                .catch((err) => {
                    console.error(err);
                });
            }   
        },
        //删除整条动态
        deleteContentRecord(index, row){
            var _this = this;
            _this.listLoading = true;
            this.$confirm('此操作将永久删除, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                var url = '/NewContent/deleteContentRecord';
                let params = {
                    content_id : row.content_id
                };
                axios.get(allget+url, {params: params})
                .then((res) => {
                    _this.listLoading = false;
                    if (res.data.ret) {
                         _this.getData();
                        this.$message({
                            type: 'success',
                            message: '删除成功!'
                        });
                    } else {
                        this.$message({
                            type: 'info',
                            message: res.data.msg
                        });
                    }
                })
                .catch((err) => {
                    console.error(err);
                });   
            }).catch(() => {
                _this.listLoading = false;
                this.$message({
                    type: 'info',
                    message: '已取消删除'
                });          
            });
        },
         handleSelectionChange(val) {
            var _this = this;
            var uid_list = '';
            val.forEach(function(item, index) {
                uid_list = uid_list=='' ? item.uid : uid_list+','+item.uid;
            });
            // 将对应的uid_list保留
            _this.collectiveSeal.uid_list = uid_list;
            // 判断对应的值按钮的点击控制
            if(_this.collectiveSeal.uid_list=='') {
               _this.collectiveSeal.dialogOne = false; 
            } else {
               _this.collectiveSeal.dialogOne = true; 
            }
        },
        // 获取图片
        changeData(val){
            try{
                if(val==""){
                    return "";
                }else{
                    return JSON.parse(val)[0];
                }
            }
            catch(err){
                console.error("后台返回图片格式改了"+err);
            }
        },
         // 忽略
        ignore(index, row) {
            let _this = this;
            let url = "/NewUser/ignoreUserComplain";
            let params = {
                id: row.id
            };
            axios.get(allget+url, {params: params})
                .then((res) => {
                    baseConfig.successTipMsg(_this, res.data.msg);
                })
                .catch((err) => {
                    console.log(err);
                    baseConfig.warningTipMsg(_this, 'error');
                });
        },
    },
    mounted() {
        var _this = this;
        _this.tableHeight = baseConfig.lineNumber(searchPageHeight);
        _this.getData();
        _this.operate_user = store.state.user.name;
        var id = store.state.user.channelid.split(",");
        var name = store.state.user.channelname.split(",");
        for (var i = 0; i < id.length; i++) {
            this.channelData[id[i]] = name[i];
        }
    }
};
</script>
<style lang="css" scoped>
.addTitle{
    margin-top:10px;
    margin-bottom:10px;
}
.content {
    background-color: rgba(0, 134, 139, 0.5);
}
.grid-content {
    background: #ffffff;
    padding: 10px 0;
}
.grid-content-first{
    height: 380px;
}
.grid-content-second{
    height:auto;
}
.uer_container {
    width:100%;
    height: 100%;
    overflow:hidden;
    background: #ffffff;
}

.comment-content{
    margin: 10px 0;
    font-size:14px;
    color: #333333;
}
/* 相册 */
.user-photo {
    float: left;
    margin: 10px;
    background-color: #f0f8ff;
}

/* 文字大小 */
.txt-size{
    font-size: 12px;
}
.showbtn{
	position: absolute;
	left: 0;
	top: 0;
	width: 100px;
	height: 30px;
	line-height: 30px;
	text-align:center;
	font-size: 12px;
	color: #fff;
	background-color: #00b3ee;
	border: 2px solid #00b3ee;
     cursor: pointer;
}
.fileinput{
	position: absolute;
	left: 0;
	top: 0;
	width: 100px;
	height: 30px;
	opacity: 0;
    z-index: 1;
	border: 2px solid red;
}
.flieBtn{
    position:relative;
    margin-left: 0px;
}
.fliebox{
    width: 680px;
    height: auto;
}
</style>

