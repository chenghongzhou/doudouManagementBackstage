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
                            <el-option label="待处理" value="0"></el-option>
                            <el-option label="已封号" value="2"></el-option>
                            <el-option label="已警告" value="3"></el-option>
                            <el-option label="已解禁" value="1"></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item>
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
                <el-table-column prop="create_time" label="时间"></el-table-column>
                <el-table-column prop="content_id_list" label="动态ID">
                    <template slot-scope="scope">
						{{changeArr(scope.row.content_id_list)}}
					</template>
                </el-table-column>
                <el-table-column prop="uid" label="UID"></el-table-column>
                <el-table-column prop="nickname" label="昵称"></el-table-column>
                <el-table-column prop="admin" label="操作人"></el-table-column>
                <el-table-column prop="icon" label="图像">
					<template slot-scope="scope">
						<img 
                        :src="scope.row.icon" 
                        style="width:100px;height:100px;">
					</template>
				</el-table-column>
                <el-table-column label="处理详情" width="80">
					<template slot-scope="scope">
						<div slot="reference" class="name-wrapper">
							<p v-if="scope.row.status == 0">待处理</p>
							<p v-else-if="scope.row.status==1">已警告</p>
                            <p v-else-if="scope.row.status==2">已封号</p>
                            <p v-else-if="scope.row.status==3">待处理</p>
						</div>
					</template>
				</el-table-column>
                <el-table-column label="动态详情" width="150">
                    <template slot-scope="scope">
                        <el-row>
                            <el-col>
                                <el-button 
                                size="mini" 
                                type="primary"
                                @click="dynamicDetail(scope.$index, scope.row)">查看</el-button>
                            </el-col>
                        </el-row>
                    </template>
                </el-table-column>            
                <el-table-column label="操作" width="350">
                    <template slot-scope="scope">
                        <el-row>
                            <el-button 
                            size="mini" 
                            type="primary" 
                            @click="userDetail(scope.$index, scope.row)">用户详情</el-button>
                            <el-button 
                            v-if="scope.row.status == 1 && scope.row.status != 2"
                            size="mini" 
                            type="info" 
                            >已解禁</el-button>
                            <el-button 
                            size="mini" 
                            v-if="scope.row.status != 1 && scope.row.status != 2"
                            type="primary" 
                            @click="Unlock(scope.$index, scope.row)">解禁</el-button>
                            <el-button 
                            size="mini"
                            type="warning" 
                            v-if="scope.row.status != 2"
                            @click="warnContentRecordUser(scope.$index, scope.row)">警告</el-button>
                            <el-button 
                            v-if="scope.row.status != 2"
                            size="mini" 
                            type="danger"
                            @click="title(scope.$index, scope.row)">封号</el-button>
                            <el-button 
                            v-if="scope.row.status == 2"
                            size="mini" 
                            type="info">已封号</el-button>
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
        <el-dialog title="动态详情" :visible.sync="userInfo.dialogFormVisible">
             <el-col style="height: 100%;width:100%;float:none">
                <div class="grid-content bg-purple grid-content-second">
                    <div>
                        <p class="comment-content">{{userInfo.content}}</p>
                        <div class="uer_container">
                            <div 
                            class="user-photo"  
                            v-for="(o, index) in userInfo.pic_list" 
                            :key="index">
                                <el-popover trigger="click" placement="left">
                                    <img 
                                    :src="o" 
                                    style="width:300px;height:400px;">
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
                                    @click="deletePhoto(index,userInfo.pic_list)" 
                                    size="mini">删除</el-button>
                                </div>
                            </div>
                        </div>
                        <div class="tab_page">
                            <div 
                                v-for="(item, index) in userInfo.len" 
                                :key="index" 
                                v-bind:class="{isThis:isThis==index}"
                                @click="checkThisPage(userInfo,index)">
                                {{index+1}}
                            </div>
                        </div>   
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
                                style="width:300px;height:400px;">
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
            listData: [1],
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
                id:"",
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
                warn_content: '',
                id:''
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
                content_id:null,
                data:null,
                len:null,
                content:null,
                pic_list:null,
                thisContentId:null,
            },
            userImgInfo: {
                dialogFormVisible: false, 
            },
            tableHeightOther: '200px',
            isThis:0,
        };
    },
    methods: {
        handleCurrentChange(val) {
            this.page = val;
            this.getData();
        },
        // 获取数据
        getData() {
            var _this = this;
            _this.listLoading = true;
            let url = "/NewContent/getContentSuspectList";
            let params = {
                start_date: baseConfig.changeDateTime(this.formOne.startDate[0], 0),
                end_date: baseConfig.changeDateTime(this.formOne.startDate[1], 0),
                uid: _this.uid,
                page: _this.page,
                status: _this.status,
            };
            axios.get(allget+url, {params: params})
                .then((res) => {
                    _this.listLoading = false;
                    if (res.data.ret) {
                        _this.listData = res.data.data;
                    } else {
                        baseConfig.warningTipMsg(_this, res.data.msg);
                    }
                })
                .catch((err) => {
                    console.error(err);
                });
        },
         // 获取图片
        changeData(val){
            try{
                if(val==""){
                    return '';
                }else{
                    return val.length;
                }
            }
            catch(err){
                console.error("后台返回图片格式改了"+err);
            }
        },
        changeArr(val){
             if(val=="" || !val){
                return '';
            }else{
                return  Array.from(val).join(',');
            }
        },
        // 用户详情
        dynamicDetail(index, rows) {
            var _this = this;
            _this.userInfo.dialogFormVisible=true;
            _this.isThis = 0;
            _this.userInfo.content_id = rows.content_id_list;
            _this.userInfo.len = rows.content_id_list.length;
            _this.userInfo.thisContentId = rows.content_id_list[0];
            _this.getCommentData(_this.userInfo,0);
        },
        //获取动态信息
        getCommentData(rows,index){
             var _this = this;
            _this.listLoading = true;
            let url = "/NewContent/getContentRecord";
            let params = {
                content_id : rows.content_id[index]
            };
            axios.get(allget+url, {params: params})
            .then((res) => {
                 _this.listLoading = false;
                if (res.data.ret) {
                    _this.userInfo.data = res.data.data;
                    _this.userInfo.pic_list = res.data.data.pic_list;
                     _this.userInfo.content = res.data.data.content;
                } else {
                    baseConfig.warningTipMsg(_this, res.data.msg);
                }
            })
            .catch((err) => {
                console.error(err);
            });
        },
        //查看动态翻页
        checkThisPage(rows,index){
            this.userInfo.thisContentId = rows.content_id[index];
            this.isThis = index;
            this.getCommentData(rows,index)
        },
        //删除图片
        deletePhoto(index, content_id){
            var _this = this;
            _this.listLoading = true;
            let url = "/NewContent/deleteContentRecordElement";
            let params = {
                content_id : _this.userInfo.thisContentId,
                pic_number : index,
                type : 2
            };
            axios.get(allget+url, {params: params})
            .then((res) => {
                 _this.listLoading = false;
                if (res.data.ret) {
                    baseConfig.successTipMsg(_this, res.data.msg);
                    let removePhoto = _this.userInfo.pic_list.splice(index,1);
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
            Event.$emit("show-one-user", {
                uid: rows.uid
            });
        },
        //警告
        warnContentRecordUser(index, row){
            this.warnInfo.dialogFormVisible=true;
            this.warnInfo.uid = row.uid;
            this.warnInfo.id = row.id;
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
                    if (res.data.ret == 1) {
                        axios.get(allget+'/NewContent/warnContentRecordUser', {params:{uid: _this.warnInfo.uid}})
                        .then((res) => {
                            if (res.data.ret == 1) {
                                let url2 = "/NewContent/setContentSuspectStatus";
                                let params2 = {
                                    status : 3,
                                    admin: _this.operate_user,
                                    id: _this.warnInfo.id
                                };
                                axios.get(allget+url2, {params: params2})
                                .then((res) => {
                                     _this.warnInfo.dialogFormVisible=false;
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
                    } else {
                        baseConfig.warningTipMsg(_this, res.data.msg);
                    }
                })
                .catch((err) => {
                    console.error(err);
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
                let url2 = "/NewContent/setContentSuspectStatus";
                let params2 = {
                    status : 1,
                    admin: _this.operate_user,
                    id: row.id
                };
                axios.get(allget+url2, {params: params2})
                .then((res) => {
                    if (res.data.ret) {
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
                    } else {
                        baseConfig.warningTipMsg(_this, res.data.msg);
                    }
                })   
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
            this.titleInfo.id = rows.id;
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
                        let url2 = "/NewContent/setContentSuspectStatus";
                        let params2 = {
                            status : 2,
                            admin: _this.operate_user,
                            id: _this.titleInfo.id
                        };
                        axios.get(allget+url2, {params: params2})
                        .then((res) => {
                            if (res.data.ret) {
                                _this.listLoading = false;
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
.el-dialog .el-dialog__body{
    padding: 0 !important;
}
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
    display: inline-block;
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
.tab_page{
    width: 100px;
    height:30px;
    font-size: 14px;
    color:#333333;
    margin: 20px auto 0;
}
.tab_page div{
    float: left;
    width: 31px;
    height: 30px;
    text-align: center;
    line-height: 30px;
    border: 1px solid #ebeef5;
    cursor: pointer;
}
.isThis{
    color:#409EFF;
}
</style>
