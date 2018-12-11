<template>
    <!-- 家族查询 -->
    <section>
        <el-col :span="24" class="toolbar" style="padding-bottom:0px;">
            <el-form :inline="true" style="overflow:hidden;">
                <el-form-item>
                    <div class="block">
                        <span class="registerTime">日期</span>
                        <el-date-picker 
                        v-model="formOne.startDate" 
                        type="daterange" 
                        range-separator=" 至 " 
                        start-placeholder="开始日期" 
                        end-placeholder="结束日期"></el-date-picker>
                    </div>
                </el-form-item>
                <el-form-item>
                    <span>家族ID</span>
                    <el-input 
                    style="width:150px;" 
                    placeholder="请输入内容" 
                    v-model="searchUid"></el-input>
                </el-form-item>
                <el-form-item style="float:right;">
                    <el-button 
                    type="primary" 
                    @click="getData()">查询</el-button>
                </el-form-item>
            </el-form>
        </el-col>
        <template>
            <el-table 
            :data="listData" 
            v-loading="listLoading" 
            border fit highlight-current-row 
            style="width:100%;" 
            :height="tableHeight">
                <el-table-column prop="create_time" label="创建时间"></el-table-column>
                <el-table-column prop="family_id" label="家族ID"></el-table-column>
                <el-table-column prop="family_name" label="家族名称"></el-table-column>
                <el-table-column prop="owner_uid" label="创建者"></el-table-column>
                <el-table-column prop="nickname" label="创建者昵称"></el-table-column>
                <el-table-column prop="family_icon" label="家族头像">
                    <template slot-scope="scope">
                        <el-popover trigger="hover" placement="left">
                            <img 
                            :src="scope.row.family_icon" 
                            style="width:300px;height:300px;">
                            <div slot="reference" class="name-wrapper">
                                <img 
                                :src="scope.row.family_icon" 
                                style="width:100px;height:100px;">
                            </div>
                        </el-popover>
                    </template>
                </el-table-column>
                <el-table-column prop="level" label="家族等级"></el-table-column>
                <el-table-column prop="yun_xin_room_id" :formatter="judgeAuthority" label="房间权限"></el-table-column>
                <el-table-column prop="member_num" label="成员数量"></el-table-column>
                <el-table-column label="操作" width="180px">
                    <template slot-scope="scope">
                        <el-col :span="12">
                            <el-button 
                            size="mini" 
                            type="primary" 
                            @click="familydetail(scope.$index, scope.row)">家族详情</el-button>
                        </el-col>
                        <el-col :span="12">
                            <el-button 
                            size="mini" 
                            type="danger" 
                            @click="lister(scope.$index, scope.row)">成员列表</el-button>
                        </el-col>
                    </template>
                </el-table-column>
            </el-table>
            <el-col :span="24" class="toolbar">
                <el-pagination 
                layout="total,prev,pager,next,jumper" 
                :page-size="20"  
                @current-change="handleCurrentChange" 
                :total="totalpage" 
                style="float:right;"></el-pagination>
            </el-col>
        </template>
        <!-- Table -->
        <el-dialog title="家族详情" :visible.sync="dialogTableVisible" center>
            <div class="family_content">
                <div class="image_container">
                    <div class="image_box" v-if="form.family_icon">
                        <div class="avator">头像</div>
                        <div class="picture">
                            <el-popover trigger="hover" placement="left">
                                <img 
                                :src=form.family_icon
                                style="width:400px;height:400px;">
                                <div slot="reference" class="name-wrapper">
                                    <img 
                                    :src=form.family_icon 
                                    style="width:120px;height:120px;">
                                </div>
                            </el-popover>
                        </div>
                        <div class="delete">
                            <el-button 
                            type="primary" 
                            size="mini" 
                            @click="familyChange(1)">删除</el-button>
                        </div>
                    </div>
                    <div class="image_box" v-if="form.index_background_pic">
                        <div class="avator">主页背景图</div>
                        <div class="picture">
                            <el-popover trigger="hover" placement="left">
                                <img 
                                :src=form.index_background_pic
                                style="width:400px;height:400px;">
                                <div slot="reference" class="name-wrapper">
                                    <img 
                                    :src=form.index_background_pic
                                    style="width:120px;height:120px;">
                                </div>
                            </el-popover>
                        </div>
                        <div class="delete">
                            <el-button 
                            type="primary" 
                            @click="familyChange(4)" 
                            size="mini">删除</el-button>
                        </div>
                    </div>
                    <div class="image_box" v-if="form.chat_background_pic">
                        <div class="avator">聊天背景图</div>
                        <div class="picture">
                            <el-popover trigger="hover" placement="left">
                                <img 
                                :src=form.chat_background_pic 
                                style="width:400px;height:400px;">
                                <div slot="reference" class="name-wrapper">
                                    <img 
                                    :src=form.chat_background_pic 
                                    style="width:120px;height:120px;">
                                </div>
                            </el-popover>
                        </div>
                        <div class="delete">
                            <el-button 
                            type="primary" 
                            @click="familyChange(5)" 
                            size="mini">删除</el-button>
                        </div>
                    </div>
                </div>
                <div class="family_detail">
                    <el-form :model="form">
                        <el-form-item label="家族ID" :label-width="formLabelWidth">
                            <el-input 
                            v-model="form.family_id" 
                            :disabled="true" 
                            style="width:200px;" 
                            auto-complete="off"></el-input>
                        </el-form-item>
                        <el-form-item label="家族名称" :label-width="formLabelWidth">
                            <el-input 
                            v-model="form.family_name" 
                            @focus="judgeChange(0)" 
                            @blur="familyChange(0)" 
                            style="width:200px;" 
                            auto-complete="off"></el-input>
                            <p style="margin: 0;font-size:12px;height:20px;color:red;"># 输入完成后，点击输入框外即可完成修改！</p>
                        </el-form-item>
                        <el-form-item label="创建者UID" :label-width="formLabelWidth">
                            <el-input 
                            v-model="form.owner_uid" 
                            :disabled="true" 
                            style="width:200px;" 
                            auto-complete="off"></el-input>
                        </el-form-item>
                        <el-form-item label="创建者昵称" :label-width="formLabelWidth">
                            <el-input 
                            v-model="form.nickname" 
                            :disabled="true" 
                            style="width:200px;" 
                            auto-complete="off"></el-input>
                        </el-form-item>
                        <el-form-item label="家族等级" :label-width="formLabelWidth">
                            <el-input 
                            v-model="form.level" 
                            :disabled="true" 
                            style="width:200px;" 
                            auto-complete="off"></el-input>
                        </el-form-item>
                        <el-form-item label="家族人数" :label-width="formLabelWidth">
                            <el-input 
                            v-model="form.member_num" 
                            :disabled="true" 
                            style="width:200px;" 
                            auto-complete="off"></el-input>
                        </el-form-item>
                        <el-form-item label="荣耀值" :label-width="formLabelWidth">
                            <el-input 
                            v-model="form.honour_num" 
                            :disabled="true" 
                            style="width:200px;" 
                            auto-complete="off"></el-input>
                        </el-form-item>
                        <el-form-item label="家族介绍" :label-width="formLabelWidth">
                            <el-input 
                            v-model="form.intro" 
                            @focus="judgeChange(2)"
                            @blur="familyChange(2)" 
                            style="width:200px;" 
                            auto-complete="off"></el-input>
                            <p style="margin: 0;font-size:12px;height:20px;color:red;"># 输入完成后，点击输入框外即可完成修改！</p>
                        </el-form-item>
                        <el-form-item label="家族公告" :label-width="formLabelWidth">
                            <el-input 
                            v-model="form.notice" 
                            @focus="judgeChange(3)" 
                            @blur="familyChange(3)" 
                            style="width:200px;" 
                            auto-complete="off"></el-input>
                            <p style="margin: 0;font-size:12px;height:20px;color:red;"># 输入完成后，点击输入框外即可完成修改！</p>
                        </el-form-item>
                        <el-form-item label="创建时间" :label-width="formLabelWidth">
                            <el-input 
                            v-model="form.create_time" 
                            :disabled="true" 
                            style="width:200px;" 
                            auto-complete="off"></el-input>
                        </el-form-item>
                    </el-form>
                </div>
            </div>
        </el-dialog>
        <!-- 家族详情 -->
        <el-dialog :title=familyName :visible.sync="dialogTableVisible1">
            <el-table 
            :data="familyListData" 
            border>
                <el-table-column type="index" label="成员"></el-table-column>
                <el-table-column property="nickname" label="昵称"></el-table-column>
                <el-table-column property="uid" label="uid"></el-table-column>
                <el-table-column property="level" :formatter="judgeLevel" label="职位"></el-table-column>
                <el-table-column property="contribute" label="贡献值"></el-table-column>
                <el-table-column property="yesterday_call" label="昨日通话时长"></el-table-column>
                <el-table-column property="yesterday_ticket" label="昨日获得豆票数量"></el-table-column>
            </el-table>
        </el-dialog>
    </section>
</template>

<script>
import { allget } from "../../../api/api";
import axios from "axios";
export default {
    data() {
        return {
            tableHeight: null, 
            formOne: {
                startDate: [new Date()-7*24*60*60*1000, new Date()] 
            },
            page: 0,
            totalpage: 1000,
            listData: [],
            formLabelWidth: "120px",
            listLoading: false,
            searchUid: null,
            dialogTableVisible: false,
            dialogTableVisible1: false,
            detialData: null,
            form: {},
            familyListData: [],
            familyName: null,
            familyId: null,
            changeBeforForm: {
                family_name: null,
                intro: null,
                notice: null,
            }
        };
    },
    methods: {
        handleCurrentChange(val) {
            this.page = val-1;
            this.getData();
        },
        // 获取数据
        getData(type) {
            var _this = this;
            _this.listLoading = true;
            var url = "/NewFamily/getFamilyData";
            var params = {
                date_s: baseConfig.changeDateTime(this.formOne.startDate[0], 0),
                date_e: baseConfig.changeDateTime(this.formOne.startDate[1], 0),
                family_id: _this.searchUid
            };
            if(params.family_id!==null&&params.family_id!==''){
            	params.date_s=null;
            	params.date_e=null;
            }else{
               	params.date_s = baseConfig.changeDateTime(this.formOne.startDate[0], 0);
                params.date_e = baseConfig.changeDateTime(this.formOne.startDate[1], 0);
            }
            axios.get(allget+url, {params: params})
                .then((res) => {
                    _this.listLoading = false;
                    if (res.data.ret) {
                        this.listData = res.data.data;
                    }else{
                        baseConfig.successTipMsg(_this, res.data.msg);
                    }
                })
                .catch((err) => {
                    console.log(err);
                });
        },
        // 判断权限
        judgeAuthority(row) {
            return row.yun_xin_room_id == 0 ? "未开启" : "开启";
        },
        judgeLevel(row) {
            if (row.level == 0) {
                return "普通成员";
            } else if (row.level == 1) {
                return "管理员";
            } else if (row.level == 2) {
                return "族长";
            }
        },
        // 家族详情信息
        familydetail(index, row) {
            var _this = this;
            this.familyId = row.family_id;
            this.dialogTableVisible = true;
            var url = "/Family/getFamilyInfo";
            let params = {
                family_id: row.family_id
            };
            axios.get(allget+url, {params: params})
                .then((res) => {
                    if (res.data.ret) {
                        _this.form = res.data.data[0];
                    }else{
                        baseConfig.successTipMsg(_this, res.data.msg);
                    }
                })
                .catch((err) => {
                    console.log(err);
                });
        },
        lister(index, row) {
            var _this = this;
            this.dialogTableVisible1 = true;
            this.familyName = row.family_name + "家族成员列表";
            var url = "/Family/getFamilyMembers";
            let params = {
                family_id: row.family_id
            };
            axios.get(allget+url, {params: params})
                .then((res) => {
                    if (res.data.ret) {
                        _this.familyListData = res.data.data;
                    }else{
                        baseConfig.successTipMsg(_this, res.data.msg);
                    }
                })
                .catch((err) => {
                    console.log(err);
                });
        },
        // 家族信息修改
        familyChange(type) {
            var _this = this;
            var url = "/NewFamily/changeFamilyData";
            // 家族名称修改
            if (type == 0) {
                var params = {
                    family_id: _this.familyId,
                    type: type,
                    family_name: _this.form.family_name,
                };
                // 如果数值一样就不发送修改数据请求
                if(params.family_name == this.changeBeforForm.family_name){
                    return;
                }
                axios.get(allget+url, {params: params})
                    .then((res) => {
                        if (res.data.ret) {
                            baseConfig.successTipMsg(_this, "修改"+ res.data.msg);
                            _this.getData();    
                        }else{
                            baseConfig.warningTipMsg(_this, res.data.msg);
                        }
                    })
                    .catch((err) => {
                        console.log(err);
                    });
            }else if(type == 2){        
                // 家族介绍修改 
                var params = {
                    family_id: _this.familyId,
                    type: type,
                    intro: _this.form.intro,
                };
                // 如果数值一样就不发送修改数据请求
                if(params.intro == this.changeBeforForm.intro){
                    return;
                }
                axios.get(allget+url, {params: params})
                    .then((res) => {
                        if (res.data.ret) {
                            baseConfig.successTipMsg(_this, "修改" + res.data.msg);
                            _this.getData();    
                        }else{
                            baseConfig.warningTipMsg(_this, res.data.msg);
                        }
                    })
                    .catch((err) => {
                        console.log(err);
                    });
            }else if(type == 3) {
                // 家族公告修改
                var params = {
                    family_id: _this.familyId,
                    type: type,
                    notice: _this.form.notice,
                };
                // 如果数值一样就不发送修改数据请求
                if(params.notice == this.changeBeforForm.notice){
                    return;
                }
                axios.get(allget+url, {params: params})
                    .then((res) => {
                        if (res.data.ret) {
                            baseConfig.successTipMsg(_this, "修改" + res.data.msg);
                            _this.getData();    
                        }else{
                            baseConfig.warningTipMsg(_this, res.data.msg);
                        }
                    })
                    .catch((err) => {
                        console.log(err);
                    });
            }else if(type==1){
                // 删除家族头像
                var params = {
                    family_id: _this.familyId,
                    type: type,
                }
                axios.get(allget+url, {params: params})
                    .then((res) => {
                        if (res.data.ret) {
                            console.log(res.data)
                            baseConfig.successTipMsg(_this, "删除" + res.data.msg);
                            _this.dialogTableVisible = false;
                            _this.getData();    
                        }else{
                            baseConfig.warningTipMsg(_this, res.data.msg);
                        }
                    })
                    .catch((err) => {
                        console.log(err);
                    });
            }else if(type==4){
                // 删除主页背景图
                var params = {
                    family_id: _this.familyId,
                    type: type,
                }
                axios.get(allget+url, {params: params})
                    .then((res) => {
                        if (res.data.ret) {
                            console.log(res.data)
                            baseConfig.successTipMsg(_this, "删除" + res.data.msg);
                            _this.dialogTableVisible = false;
                            _this.getData();    
                        }else{
                            baseConfig.warningTipMsg(_this, res.data.msg);
                        }
                    })
                    .catch((err) => {
                        console.log(err);
                    });
            }else if(type==5){
                // 删除背景聊天图
                var params = {
                    family_id: _this.familyId,
                    type: type,
                }
                axios.get(allget+url, {params: params})
                    .then((res) => {
                        if (res.data.ret) {
                            console.log(res.data)
                            baseConfig.successTipMsg(_this, "删除" + res.data.msg);
                            _this.dialogTableVisible = false;
                            _this.getData();    
                        }else{
                            baseConfig.warningTipMsg(_this, res.data.msg);
                        }
                    })
                    .catch((err) => {
                        console.log(err);
                    });
            }
        },
        // 获取焦点存值，失去焦点时判断，如果值没有变化，就不发送请求 
        judgeChange(type){
            if(type==0){
                this.changeBeforForm.family_name = this.form.family_name;
            }else if(type == 2) {
                this.changeBeforForm.intro = this.form.intro;
            }else if(type == 3) {
                this.changeBeforForm.notice = this.form.notice;
            }
        }
    },
    mounted() {
        var _this = this;
        _this.tableHeight = baseConfig.lineNumber(searchPageHeight);
        this.getData();
    }
};
</script>

<style lang="css" scoped>
.family_content {
    overflow:hidden;
}
.image_container {
    float: left;
    /* border: 1px solid orange; */
    width: 40%;
    height: 100%;
}
.image_container img {
    width:300px;
    height:200px;
}
.image_container .image_box {
    margin-top: 10px;
    border: 1px solid #666;
}
.image_container .image_box .avator {
    display:inline-block;
    width: 80px;
    height: 120px;
    line-height: 120px;
    text-align:center;
    vertical-align: top;
    /* border: 1px solid aqua; */
}
.image_container .image_box .picture {
    display:inline-block;
    vertical-align: middle;
    /* border: 1px solid saddlebrown; */
}
.image_container .image_box .delete {
    display:inline-block;
    /* border: 1px solid darkblue; */
}
.family_detail {
    float:right;
    width: 59%;
    height: 100%;
    /* border: 1px solid red; */
}
.dialog-footer {
    display:block;
    margin-top: 0px;
    margin-left: -50px;
    /* border: 1px solid red; */
}
</style>
