<template>
    <!-- 每日任务统计 -->
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
                    <span>渠道</span>
                    <el-select 
                    v-model="channelId" 
                    style="width:120px;"   
                    placeholder="请选择">
                        <el-option 
                        v-for="(item, key) of channelData" 
                        :key="key" 
                        :label="item" 
                        :value="key"></el-option>
                    </el-select>
                </el-form-item>
                <el-form-item>
                    <span>用户类型</span>
                    <el-select v-model="type" style="width:120px;"  placeholder="请选择">
                        <el-option label="全部" value="0"> </el-option>
                        <el-option label="付费" value="1"> </el-option>
                        <el-option label="活跃" value="2"> </el-option>
                        <el-option label="新用户" value="3"> </el-option>
                    </el-select>
                </el-form-item>
                <el-form-item style="float:right;">
                    <el-button 
                    type="primary" 
                    @click="getData(0)">查询</el-button>
                </el-form-item>
            </el-form>
        </el-col>
        <template>
            <el-table 
            ref="tableHeight" 
            :data="onePageTabData" 
            border 
            v-loading="listLoading" 
            style="width:100%;" 
            :height="tableHeight">
                <template v-for="col in cols">
                    <el-table-column :prop="col.prop" :label="col.label" :key="col.prop"></el-table-column>
                </template>
            </el-table>
            <el-col :span="24" class="toolbar">
                <el-pagination 
                layout="total,prev,pager,next,jumper" 
                @current-change="handleCurrentChange" 
                :page-size="20" 
                :total="totalpage" 
                style="float:right;"></el-pagination>
            </el-col>
        </template>
    </section>
</template>

<script>
import { allget } from "../../../api/api";
import axios from "axios";
import store from "../../../vuex/store";

export default {
    data() {
        return {
            tableHeight: null, 
            tableWidth: null,
            formOne: {
                startDate: [new Date()-7*24*60*60*1000, new Date()]
            },
            listData: [],
            cols: [],
            newData: [],
            page: 1, 
			star: '0', 
            end: '20',
            totalpage: null,
            formLabelWidth: "120px",
			listLoading: false,
            dialogTableVisible: false,
            dialogTableVisible1: false,
			gradeListData: [],
            taskList: {},   // 总任务id对应name
            task_html:'<el-table-column prop="date" label="日期"></el-table-column>',
			all_ob: null,      //数据对应的所有任务
            thead_html: "<th>日期</th>",
            tbody_html: "",
            channelData: {}, // 渠道数据
            channelId: null, // 渠道数据
            type: null,      // 用户类型
        };
    },
    computed: {
        onePageTabData() {
			var _this = this;
			return _this.newData.slice(_this.star, _this.end);
		},
    },
    methods: {
        handleCurrentChange(val) {
            var _this = this;
            val = val-1;
            _this.page = val;
            _this.star = (_this.page)*20;
            _this.end = _this.star+20;
        },
        getData(type) {
            var _this = this;
			if(type == 0){
				this.page = 0;
			}
            var _this = this;
            _this.listLoading = true;
            let url = "/NewLevel/getDayTaskData";
            if (this.formOne.startDate == null) {
                baseConfig.warningTipMsg(this, "请输入日期");
                return;
            }
            let params = {
                date_s: baseConfig.changeDateTime(this.formOne.startDate[0], 0),
                date_e: baseConfig.changeDateTime(this.formOne.startDate[1], 0),
                channel: this.channelId,
                type: this.type,
            };
            axios.get(allget+url, {params: params})
                .then((res) => {
                    _this.listLoading = false;
                    if (res.data.ret) {
                        _this.listData = res.data.data;
                        // 将后台拿来的数据进行组装
                        var newData = [];
                        var ob = {};
                        for(var i = 0; i<res.data.data.length;i++){
                            if(i == 0){
                                ob.date = res.data.data[i].date;
                            }
                            var p = res.data.data[i].task_id;
                            if(ob.date == res.data.data[i].date){
                                ob[p] = res.data.data[i].num;
                            }else{
                                newData.push(ob);
                                ob = {};
                                ob.date = res.data.data[i].date;
                                ob[p] = res.data.data[i].num;
                            }
                            if( i == res.data.data.length - 1){
                                newData.push(ob);
                            }
                        }
                        // 计算数据内总共用到的 总任务数
                        var all_ob = [];
                        for(var i = 0;i<newData.length;i++){
                            for(var key in newData[i]){
                                if(key != "date"){
                                    if(all_ob.indexOf(key)=="-1"){
                                        all_ob.push(key);    
                                    }
                                }
                            }
                        }
                        _this.all_ob = all_ob.sort();
                        // 对不存在数据的进行为0的处理
                        for(var i=0; i<newData.length; i++) {
                            for(var j=0; j<all_ob.length; j++) {
                                if(newData[i].hasOwnProperty(all_ob[j])==false) {
                                    newData[i][all_ob[j]] = '0';
                                } else {
                                    // 已经存在不进行处理
                                }
                            }
                        }
                        // 对all_ob、this.taskList进行相应的处理,得到cols
                        this.cols = [{prop: 'date', label: '日期'}];
                        for(var i=0; i<all_ob.length; i++) {
                            this.cols.push({
                                prop: all_ob[i],
                                label: this.taskList[all_ob[i]],
                            });
                        }
                        this.newData = newData;
                        this.totalpage = newData.length;
                    }else{
                        baseConfig.successTipMsg(_this, res.data.msg);
                    }
                })
                .catch((err) => {
                    console.log(err);
                });
        },
        // 获取任务列表
        getTaskList(){
            var _this = this;
            var url = '/NewLevel/getTaskIdList';
            var params = {};
            axios.get(allget+url, {params: params})
                .then((res) => {
                    if(res.data.ret){
                        for(var i = 0;i<res.data.data.length;i++){
                            var p = res.data.data[i].id;
                            _this.taskList[p] = res.data.data[i].desc;
                        }
                        var tls = JSON.stringify(_this.taskList);
                        _this.getData();
                    }else{
                        baseConfig.warningTipMsg(_this, res.data.msg);
                    }
                })
                .catch((err) => {
                    console.log(err);
                });
        },
	 
    },
    mounted() {
        var _this = this;
        _this.tableHeight = baseConfig.lineNumber(searchPageHeight);
        _this.tableWidth = baseConfig.lineNumber(lookWidth);
        this.getTaskList();
        var id = store.state.user.channelid.split(",");
        var name = store.state.user.channelname.split(","); 
        for(var i = 0; i<id.length; i++){
            this.channelData[id[i]] = name[i];
        }
    }
};
</script>

<style lang="css" scoped>
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
.el-dialog__wrapper .el-dialog .el-dialog__body{
	position: relative!important;
	border:1px solid red!important;
}
.data_tbody{
    text-align:center;
}
.d_thead{
    border: 1px solid red;
    color:red;
}
td{
    border: 1px solid blue;
}
</style>
