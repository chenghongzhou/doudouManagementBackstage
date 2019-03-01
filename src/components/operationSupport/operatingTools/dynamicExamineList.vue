<template>
    <!-- 用户动态审核 -->
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
                        <span>UID</span>
                        <el-input 
                        style="width:150px;" 
                        v-model="uid" 
                        placeholder="请输入UID"></el-input>
                    </el-form-item>
                    <el-form-item>
                        <span>审核状态：</span>
                        <el-select style="width:100px;" v-model="status">
                            <el-option label="全部" value="-1"></el-option>
                            <el-option label="正常" value="0"></el-option>
                            <el-option label="未处理" value="1"></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item style="float:right;">
                        <el-button 
                        type="primary" 
                        @click="getData(0)">查询</el-button>
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
                <el-table-column prop="content_time" label="时间"></el-table-column>
                <el-table-column prop="uid" label="UID"></el-table-column>
                <el-table-column prop="complaint" label="被举报记录"></el-table-column>
                <!-- <el-table-column prop="complaint" label="被举报记录">
                    <template slot-scope="scope">
                        <el-button 
                        type="primary" 
                        @click="getInfo(scope.row.complain_uid)">{{scope.row.complaint}}</el-button>
                    </template>
                </el-table-column> -->
                <el-table-column prop="content" label="内容"></el-table-column>
                <el-table-column prop="img" label="图片">
                    <template slot-scope="scope">
                        <el-popover trigger="hover" placement="left">
                            <img 
                            :src="scope.row.img" 
                            style="width:300px;height:300px;">
                            <div slot="reference" class="name-wrapper">
                                <img 
                                :src="scope.row.img"
                                style="width:100px;height:100px;">
                            </div>
                        </el-popover>
                    </template>
                </el-table-column>
                <el-table-column label="操作" width="350">
                    <template slot-scope="scope">
                        <el-row>
                            <el-col :span="4">
                                <el-button 
                                size="mini" 
                                type="primary" 
                                @click="userDetail(scope.$index, scope.row.parmas)">查看详情</el-button>
                            </el-col>
                            <el-col :span="4" v-if="scope.row.status == 1">
                                <el-button 
                                size="mini" 
                                type="primary" 
                                style="margin-left:25px;"
                                @click="Unlock(scope.$index, scope.row)">解禁</el-button>
                            </el-col>
                            <el-col :span="4" v-if="scope.row.status == 0">
                                <el-button 
                                size="mini"
                                type="info"  
                                style="margin-left:25px;">解禁</el-button>
                            </el-col>
                            <el-col :span="4">
                                <el-button 
                                size="mini"
                                type="warning" 
                                style="margin-left:25px;"
                                @click="warnContentRecordUser(scope.$index, scope.row)">警告</el-button>
                            </el-col>
                            <el-col :span="4">
                                <el-button 
                                size="mini" 
                                type="danger"
                                style="margin-left:25px;" 
                                @click="deleteContentRecord(scope.$index, scope.row)">删除</el-button>
                            </el-col>
                            <el-col :span="4" v-if="scope.row.account_status == 0">
                                <el-button 
                                size="mini" 
                                type="danger"
                                style="margin-left:25px;" 
                                @click="title(scope.$index, scope.row)">封号</el-button>
                            </el-col>
                            <el-col :span="4" v-if="scope.row.account_status == 1">
                                <el-button 
                                size="mini" 
                                type="info"
                                style="margin-left:25px;">已封号</el-button>
                            </el-col>
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
         <!--详情-->
        <el-dialog title="详细内容" :visible.sync="userInfo.dialogFormVisible">
             <el-col style="height: 100%;width:100%;float:none">
                    <div class="grid-content bg-purple grid-content-second">
                        <div>
                            <p class="comment-content">{{userInfo.content}}</p>
                            <div class="uer_container">
                                <div 
                                class="user-photo"  
                                v-for="(o, index) in userInfo.srcArr" 
                                :key="index">
                                    <el-popover trigger="click" placement="left">
                                        <img 
                                        :src="o" 
                                        style="width:300px;height:300px;">
                                        <div slot="reference" class="name-wrapper">
                                            <img 
                                            :src="o" 
                                            style="width:100px;height:100px;">
                                        </div>
                                    </el-popover>
                                    <div style="padding:0px;display:inline-block;">
                                        <el-button 
                                        type="text" 
                                        class="button" 
                                        @click="deletePhoto(index,userInfo.content_id)" 
                                        size="mini">删除</el-button>
                                    </div>
                                </div>
                            </div>
                            <div class="addTitle">评论：</div>
                            <template>
                                <el-table 
                                :data="userInfo.commentList" 
                                v-loading="listLoading" 
                                border fit highlight-current-row 
                                @selection-change="handleSelectionChange"
                                style="width:100%;" 
                                :height="tableHeightOther">
                                    <el-table-column prop="uid" label="UID" ></el-table-column>
                                    <el-table-column prop="review_content" label="内容"></el-table-column>
                                    <el-table-column label="操作">
                                        <template slot-scope="scope">
                                            <el-row>
                                                <el-col>
                                                    <el-button 
                                                    size="mini" 
                                                    type="danger"
                                                    style="margin:0 auto" 
                                                    @click="deteleComment(scope.$index, scope.row)">删除</el-button>
                                                </el-col>
                                            </el-row>
                                        </template>
                                    </el-table-column>
                                </el-table> 
                            </template>     
                        </div>
                    </div>
                </el-col>
            </el-dialog>    
            <!-- 个人封号 -->
            <el-dialog title="个人封禁账号" :visible.sync="titleInfo.dialogFormVisible">
                <el-form :model="titleInfo">
                    <el-form-item label="封号时长" :label-width="formLabelWidth">
                        <el-select v-model="titleInfo.day" placeholder="请选择封号时长">
                            <el-option label="1天" value="1"></el-option>
                            <el-option label="3天" value="3"></el-option>
                            <el-option label="5天" value="5"></el-option>
                            <el-option label="7天" value="7"></el-option>
                            <el-option label="15天" value="15"></el-option>
                            <el-option label="30天" value="30"></el-option>
                            <el-option label="永远封号" value="0"></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="封号时长" :label-width="formLabelWidth">
                        <el-select v-model="titleInfo.type" placeholder="请选择封号时长">
                            <el-option label="封号" value="0"></el-option>
                            <el-option label="封号、封设备" value="1"></el-option>
                            <el-option label="封号、封IP" value="2"></el-option>
                            <el-option label="封号、封设备、封IP" value="3"></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="封号原因" :label-width="formLabelWidth">
                        <el-input 
                        v-model="titleInfo.reason" 
                        auto-complete="off"></el-input>
                    </el-form-item>   
                </el-form>
                <div slot="footer" class="dialog-footer">
                    <el-button 
                    @click="titleInfo.dialogFormVisible=false">取 消</el-button>
                    <el-button 
                    type="primary" 
                    @click="sureTitle">确 定</el-button>
                </div>
            </el-dialog>
            <!--警告-->
            <el-dialog title="警告" :visible.sync="warnInfo.dialogFormVisible">
                <el-form :model="warnInfo">
                    <el-form-item label="警告文字" :label-width="formLabelWidth">
                        <el-input 
                        v-model="warnInfo.warn_content" 
                        placeholder="此处选填"
                        auto-complete="off"></el-input>
                    </el-form-item>
                </el-form>
                <div slot="footer" class="dialog-footer">
                    <el-button 
                    @click="warnInfo.dialogFormVisible=false">取 消</el-button>
                    <el-button 
                    type="primary" 
                    @click="sureWarn">确 定</el-button>
                </div>
            </el-dialog>
            <el-dialog title="投诉详情" :visible.sync="complaint.dialogFormVisible" width="80%">
                <el-table 
                :data="complaint.tabData">
                    <el-table-column property="id" label="ID" width="150"></el-table-column>
                    <el-table-column property="time" label="日期" width="200"></el-table-column>
                    <el-table-column property="uid" label="投诉人"></el-table-column>
                    <el-table-column property="complain_uid" label="被投诉人"></el-table-column>
                    <el-table-column property="channel" label="渠道号"></el-table-column>
                    <el-table-column prop="content" label="投诉类型"></el-table-column>
                    <el-table-column prop="explains" label="详细内容"></el-table-column>
                    <el-table-column prop="evidences" label="图片">
                        <template slot-scope="scope">
                            <el-popover trigger="hover" v-if="changeData(scope.row.evidences)" placement="left">
                                <img 
                                :src="changeData(scope.row.evidences)" 
                                style="width:300px;height:500px;">
                                <div slot="reference" class="name-wrapper">
                                    <img 
                                    :src="changeData(scope.row.evidences)" 
                                    style="width:100px;height:100px;">
                                </div>
                            </el-popover>
                        </template>
                    </el-table-column>
                    <el-table-column label="操作" min-width="120">
                        <template slot-scope="scope">
                            <div v-if="scope.row.status==1">已忽略</div>
                            <div v-else-if="scope.row.status==2">已警告</div>
                            <div v-else-if="scope.row.status==3">已封号</div>
                            <el-row v-else-if="scope.row.status==0">
                                <el-col :span="8">
                                    <el-button 
                                    size="mini" 
                                    type="success" 
                                    @click="ignore(scope.$index, scope.row)">忽视</el-button>
                                </el-col>
                                <el-col :span="8">
                                    <el-button 
                                    size="mini" 
                                    type="warning" 
                                    @click="warn(scope.$index, scope.row)">警告</el-button>
                                </el-col>
                                <el-col :span="8">
                                    <el-button 
                                    size="mini" 
                                    type="info" 
                                    @click="title(scope.$index, scope.row)">封号</el-button>
                                </el-col>
                            </el-row>
                        </template>
                    </el-table-column>
                </el-table>
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
            nickname: "",
            status: "-1",
            operate_user: '',
            page: 1, // 分页
            totalpage: 1000,
            channelData: {}, // 渠道数据
            channelId: null,
            commentListId: null,
            titleInfo: {
                dialogFormVisible: false,
                uid: "",
                reason: "",
                day: "",
                type:"",
            },
            propInfo: {
                dialogFormVisible: false,
                data: [],
                prop_id: "",
                name: "",
                status: ""
            },
            resetPassword: {
                dialogFormVisible: false,
                uid: "",
            },
            warnInfo: {
                dialogFormVisible: false,
                uid: '',
                warn_content: ''
            },
            collectiveSeal: {
                dialogOne: false,
                dialogTwo: false,
                uid_list: '',
                day: '0',
                reason: '',
                operate_user: '',
                multipleSelection: [],
            },
            complaint: {
               dialogFormVisible: false, 
               tabData: [],
            },
            userInfo: {
                dialogFormVisible: false, 
                srcArr:[],
                content:null,
                content_id:null,
                data:null,
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
            let url = "/NewContent/getContentRecordList";
            let params = {
                start_date: baseConfig.changeDateTime(this.formOne.startDate[0], 0),
                end_date: baseConfig.changeDateTime(this.formOne.startDate[1], 0),
                uid: _this.uid,
                page: _this.page,
                status: _this.status
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
                                obj.content_time = res.data.data[i].content_time;
                                obj.uid = res.data.data[i].uid;
                                obj.complaint = res.data.data[i].complaint;
                                obj.content = res.data.data[i].content;
                                obj.img = res.data.data[i].pic_list[0];
                                obj.account_status = res.data.data[i].account_status;
                                obj.content_id = res.data.data[i].content_id;
                                obj.status = res.data.data[i].status;
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
        // 用户详情
        userDetail(index, rows) {
            var _this = this;
            _this.userInfo.dialogFormVisible=true;
            _this.commentListId = rows.content_id;
            _this.userInfo.content = rows.content;
            _this.userInfo.srcArr = rows.pic_list;
            _this.userInfo.content_id = rows.content_id;
            _this.userInfo.data = rows;
            _this.getCommentData(rows);
        },
        //获取评论
        getCommentData(rows){
             var _this = this;
            _this.listLoading = true;
            let url = "/NewContent/getContentReviewList";
            let params = {
                content_id : rows.content_id
            };
            axios.get(allget+url, {params: params})
            .then((res) => {
                 _this.listLoading = false;
                if (res.data.ret) {
                    _this.userInfo.commentList = res.data.data;
                } else {
                    baseConfig.warningTipMsg(_this, res.data.msg);
                }
            })
            .catch((err) => {
                console.error(err);
            });
        },
        //删除图片
        deletePhoto(index, id){
            var _this = this;
            _this.listLoading = true;
            let url = "/NewContent/deleteContentRecordElement";
            let params = {
                content_id : id,
                pic_number : index,
                type : 2
            };
            let newArr = _this.listData.filter((item, index) => {
                return item.parmas.content_id == _this.commentListId;
            });
            axios.get(allget+url, {params: params})
            .then((res) => {
                 _this.listLoading = false;
                if (res.data.ret) {
                    baseConfig.successTipMsg(_this, res.data.msg);
                    let removePhoto = newArr[0].parmas.pic_list.splice(index,1);
                    _this.userInfo.srcArr = newArr[0].parmas.pic_list;
                    _this.getData();
                } else {
                    baseConfig.warningTipMsg(_this, res.data.msg);
                }
            })
            .catch((err) => {
                console.error(err);
            });
        },
        //删除评价
        deteleComment(index, row){
            console.log(row)
            var _this = this;
            _this.listLoading = true;
            let url = "/NewContent/deleteContentReview";
            let params = {
                review_id : row.review_id
            };
            axios.get(allget+url, {params: params})
            .then((res) => {
                 _this.listLoading = false;
                if (res.data.ret) {
                    baseConfig.successTipMsg(_this, res.data.msg);
                    _this.getCommentData(_this.userInfo.data)
                } else {
                    baseConfig.warningTipMsg(_this, res.data.msg);
                }
            })
            .catch((err) => {
                console.error(err);
            });
        },
        //警告
        warnContentRecordUser(index, row){
            this.warnInfo.dialogFormVisible=true;
            this.warnInfo.uid = row.uid;
        },
        sureWarn(){ 
            var _this = this;
            var url = '/NewUser/warnComplain';
            let formData = new FormData();
            formData.append('warn_content', _this.warnInfo.warn_content);
            formData.append('uid', _this.warnInfo.uid);
            formData.append('operate_user', _this.operate_user);
            let config = {
                headers: {
                    'Content-Type': 'multipart/form-data'
                }
            };
            axios.post(allget+url, formData, config)
                .then((res) => {
                    _this.warnInfo.dialogFormVisible=false;
                    if (res.data.ret == 1) {
                        axios.get(allget+'/NewContent/warnContentRecordUser', {params:{uid: _this.warnInfo.uid}})
                        .then((res) => {
                            if (res.data.ret) {
                                baseConfig.successTipMsg(_this, res.data.msg);
                                _this.getData()
                            } else {
                                baseConfig.warningTipMsg(_this, res.data.msg);
                            }
                        })
                        .catch((err) => {
                            console.error(err);
                        });
                    } else {
                        baseConfig.warningTipMsg(_this, res.data.msg);
                    }
                })
                .catch((err) => {
                    console.error(err);
                });      
        },
        //删除整条动态
        deleteContentRecord(index, row){
            var _this = this;
            console.log(index, row,45)
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
        //解禁
        Unlock(index, row) {
            var _this = this;
            _this.listLoading = true;
            this.$confirm('是否对该条动态解禁?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                var url = '/NewContent/activateContentRecord';
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
                            message: '操作成功!'
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
                    message: '已取消操作'
                });          
            });
        },
        // 封号
        title(index, rows) {
            this.titleInfo.uid = rows.uid;
            this.titleInfo.dialogFormVisible=true;
        },
        sureTitle() {
            var _this = this;
            var url = "/User/kickUser";
            let formData = new FormData();
            formData.append('uid', _this.titleInfo.uid);
            formData.append('day', _this.titleInfo.day);
            formData.append('reason', _this.titleInfo.reason);
            formData.append('type', _this.titleInfo.type);
            formData.append('operate_user', _this.operate_user);
            let config = {
                headers: {
                    'Content-Type': 'multipart/form-data'
                }
            };
            if (formData.get('reason') == "" || formData.get('reason') == null) {
                baseConfig.warningTipMsg(this, "封人家总得有个原因吧！");
                return;
            }
            axios.post(allget+url, formData, config)
                .then((res) => {
                    if (res.data.ret) {
                        baseConfig.successTipMsg(this, res.data.msg);
                        _this.getData();
                        this.titleInfo.uid = "";
                        this.titleInfo.day = "1";
                        this.titleInfo.day = "0";
                        this.titleInfo.reason = "";
                        this.titleInfo.dialogFormVisible=false;
                    } else {
                        baseConfig.warningTipMsg(_this, res.data.msg);
                    }
                })
                .catch((err) => {
                    console.log(err);
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
        //被举报记录
        // getInfo(val) {
        //     console.log(val);
        //     var _this = this;
        //     var url = '/NewUser/getComplainById';
        //     var params = {
        //         uid: '10075',
        //     };
        //     axios.get(allget+url, {params: params})
        //         .then((res) => {
        //             console.log(res.data.data);
        //             _this.complaint.tabData = res.data.data; 
        //             _this.complaint.dialogFormVisible=true;
        //         })
        //         .catch((err) => {
        //             console.log(err);
        //         });
        // },
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
    height: 580px;
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
</style>
