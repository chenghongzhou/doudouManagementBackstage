<template>
    <!-- 幸运树 -->
    <section>
        <el-col :span="24" class="toolbar" style="padding-bottom:0px;">
            <el-form :inline="true" style="overflow:hidden;">
                <el-row> 
                    <el-form-item>
                        <span>宝箱等级：</span>
                        <el-select style="width:200px;" v-model="box">
                            <el-option label="黄金宝箱" value="1"></el-option>
                            <el-option label="铂金宝箱" value="2"></el-option>
                            <el-option label="钻石宝箱" value="3"></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item>
                        <el-button 
                        type="primary" 
                        @click="getData(0)">查询</el-button>
                    </el-form-item>
                    <el-form-item 
                    :span="24" 
                    class="toolbar" 
                    style="padding-bottom:0px;margin-top:-10px">
                        <div class="caseStatus">状态：{{caseStatus}}</div>
                        <el-button 
                        type="primary"
                        @click="activityStatus.dialogFormVisible = true">修改状态</el-button>
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
                <el-table-column prop="level" label="排序"></el-table-column>
				<el-table-column prop="threshold" label="开奖金额"></el-table-column>
                <el-table-column prop="progress" label="当前金额"></el-table-column>
                <el-table-column prop="rate" label="概率"></el-table-column>
                <el-table-column label="操作">
                    <template slot-scope="scope">
                        <el-row>
                            <el-button 
                                size="mini" 
                                type="primary" 
                                @click="modify(scope.$index, scope.row)">编辑</el-button>
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
         <!--编辑-->
        <el-dialog title="编辑" :visible.sync="editorloading.dialogFormVisible">
			<el-form :model="editorloading.params">
                <el-form-item label="奖金" :label-width="formLabelWidth">
					<el-input 
					style="width:250px"
					v-model="editorloading.params.threshold" 
					auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="概率" :label-width="formLabelWidth">
					<el-input 
					style="width:250px"
					v-model="editorloading.params.rate" 
					auto-complete="off"></el-input>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button 
				@click.native.prevent="editorBannerSure(0)">取 消</el-button>
				<el-button 
				type="primary" 
				@click.native.prevent="editorBannerSure(1)">确 定</el-button>
			</div>
		</el-dialog>
        <!-- 修改宝箱状态 -->
		<el-dialog title="修改状态" :visible.sync="activityStatus.dialogFormVisible">
			<el-form :model="activityStatus">
				<el-form-item label="好运宝箱">
					<el-select v-model="activityStatus.status">
						<el-option label="开启" value="1"></el-option>
						<el-option label="关闭" value="0"></el-option>
					</el-select>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button 
				@click.native.prevent="changeStatusSure(0)">取 消</el-button>
				<el-button 
				type="primary" 
				@click.native.prevent="changeStatusSure(1)">确 定</el-button>
			</div>
		</el-dialog>
		<!-- 修改宝箱状态 -->
    </section>
</template>

<script>
import Event from './../../../public_js/event.js';
import store from '../../../vuex/store';
import axios from 'axios';
import { allget } from '../../../api/api.js';
export default {
    data() {
        return {
            
            tableHeight: null,
            listData: [],
            formLabelWidth: "120px",
            listLoading: false,
            nickname: "",
			box:'1',
            page: 1, // 分页
            totalpage: 1000,
			priceBox:false,
			starBox:false,
			editorloading:{
				dialogFormVisible:false,
				params:{
					rate:'',
					threshold:'',
					id:'',
				}
            },
            activityStatus:{
                status:'1',
                dialogFormVisible:false,
                dialogFormVisibleOther:false,
            },
			wp:[],
			getSort:['0'],
			getLevel:['0'],
			money:'',
			getMoneyList:['0'],
            tableHeightOther: '200px',
            caseStatus:null,
        };
    },
    methods: {
        handleCurrentChange(val) {
            this.page = val;
            this.getData(this.box);
        },
        // 获取数据
        getData() {
            var _this = this;
            _this.listLoading = true;
            let url = "/NewLuckBox/getPoolThresholdList";
            let params = {
                page: _this.page,
                box_type: _this.box
            };
            axios.get(allget+url, {params: params})
                .then((res) => {
                   _this.listLoading = false;
                    if (res.data.ret) {
                        _this.listData = res.data.data.pool_threshold_list;
                        if(res.data.data.status == 1){
                            _this.caseStatus = '开启';
                        }else{
                            _this.caseStatus = '关闭';
                        };
                    } else {
                        baseConfig.warningTipMsg(_this, res.data.msg);
                    }
                })
                .catch((err) => {
                    console.error(err);
                });
        },
		//添加时选择物品类型后请求接口获取对应的物品
		checkType(){
			var _this = this;
			_this.wp = [];
			let url = "/NewEgg/getGiftOrPropList";
            let params = {
                type: _this.addNewloading.params.type
            };
            axios.get(allget+url, {params: params})
                .then((res) => {
                    if (res.data.ret) {
                        _this.wp = res.data.data;
                    } else {
                        baseConfig.warningTipMsg(_this, res.data.msg);
                    }
                })
                .catch((err) => {
                    console.error(err);
                });
        },
         changeStatusSure(num){
				var _this = this;
				if(num == 0){
					_this.activityStatus.dialogFormVisible = false;
				}else{
					axios.get(allget+'/NewLuckBox/setStatus', {params: {status:_this.activityStatus.status}})
					.then((res) => {
						if(res.data.ret == 1) {
							_this.getData();
							baseConfig.successTipMsg(_this, '修改成功');
							_this.activityStatus.dialogFormVisible = false;
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg);
						}
					})
					.catch((err) => {
						console.log(err);
					})
				}
			},
		//添加时选择物品后请求接口获去对应的价格
		checkMoney(){
			var _this = this;
			_this.addNewloading.params.chat_gold = '';
			var id = _this.addNewloading.params.prop_id;
			_this.wp.forEach((item) => {
				if(item.id == id){
					if(_this.addNewloading.params.type == 1){
						_this.priceBox = true;
						_this.addNewloading.params.chat_gold = item.price;
					}else{
						_this.priceBox = false;
					};
					_this.addNewloading.params.icon = item.icon;
				}
			});
		},
		//上传图片
		uploading(event){
			var _this = this;
			_this.addNewloading.params.file_pic = event.target.files[0];
		  	var windowURL = window.URL || window.webkitURL;
			_this.addNewloading.params.icon = windowURL.createObjectURL(event.target.files[0]);
			_this.addNewloading.params.pic_name = event.target.files[0].name;
		},
		//编辑
		modify(index,rows){
			this.editorloading.dialogFormVisible = true;
			this.editorloading.params.rate = rows.rate;
			this.editorloading.params.id = rows.id;
			this.editorloading.params.threshold = rows.threshold;
		},
		//获取级别
		checklevel(box){
			var _this = this;
			_this.getSort = ['0'];
			let url = "/NewBox/getPrizeLevelList";
            let params = {
                box: _this.addNewloading.params.box
            };
            if(_this.editorloading.dialogFormVisible){
                params.box = _this.editorloading.params.box
            };
            axios.get(allget+url, {params: params})
                .then((res) => {
                    if (res.data.ret) {
                        _this.getLevel=res.data.data;
                    } else {
                        baseConfig.warningTipMsg(_this, res.data.msg);
                    }
                })
                .catch((err) => {
                    console.error(err);
                });	
		},
		//获取排序
		checkSort(box){
			var _this = this;
			_this.getSort = ['0'];
			let url = "/NewBox/getPrizeSortList";
            let params = {
               box: _this.addNewloading.params.box
            };
            if(_this.editorloading.dialogFormVisible){
                params.box = _this.editorloading.params.box
            };
            axios.get(allget+url, {params: params})
                .then((res) => {
                    if (res.data.ret) {
                        _this.getSort=_this.getSort.concat(res.data.data);
                    } else {
                        baseConfig.warningTipMsg(_this, res.data.msg);
                    }
                })
                .catch((err) => {
                    console.error(err);
                });	
		},
		getSortEvery(){
			this.checkSort();
		},
		editorBannerSure(num){
			var _this = this;
			if(num == 0){
				_this.editorloading.dialogFormVisible = false;
				_this.clearEditFrom();
			}else if(num == 1){
				_this.listLoading = true;
				let formData = new FormData();
				formData.append('id', _this.editorloading.params.id);
				formData.append('threshold', _this.editorloading.params.threshold);
                formData.append('rate', _this.editorloading.params.rate);
				let config = {
					headers: {
						'Content-Type': 'multipart/form-data'
					}
				};	
				axios.post(allget+'/NewLuckBox/setPoolThreshold', formData, config)
					.then((res) => {
						_this.listLoading = false;							
						if(res.data.ret == 1) {	
							_this.editorloading.dialogFormVisible = true;
							baseConfig.successTipMsg(_this, '修改成功');
							_this.getData();
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg);						
						}
						_this.clearEditFrom();
					})
					.catch((err) => {
						console.log(err);
					});
			}
		},
		//初始化添加数据
		clearAddFrom(){
			var _this = this;
			_this.addNewloading={
				dialogFormVisible:false,
				params:{
					rate:'',
					threshold:'',
					id:'',
				}
			};
		},
		//初始化编辑数据
		clearEditFrom(){
			var _this = this;
			_this.editorloading={
				dialogFormVisible:false,
				params:{
					type:'',
					prop_id:'',
					icon:'',
					probability:'',
					chat_gold:0,
					status:'',
					sort:'',
                    box:'',
					pic_name:'',
					file_pic:'',
					prize_id:'',
					name:'',
					id:'',
                    level:'',
                    rate:'',
					threshold:'',
					id:'',
				}
			};
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
    },
    mounted() {
        var _this = this;
        _this.tableHeight = baseConfig.lineNumber(searchPageHeight);
        _this.getData();
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
	width: 150px;
	height: 30px;
	line-height: 30px;
	text-align:center;
	font-size: 12px;
	color: #fff;
	background-color: #00b3ee;
	border: 2px solid #00b3ee;
}
.fileinput{
	position: absolute;
	left: 0;
	top: 0;
	width: 150px;
	height: 30px;
	opacity: 0;
	border: 2px solid red;
}
.caseStatus{
	line-height: 40px;
	font-size: 16px;
	margin-left: 20px;
	display: inline-block; 
	margin-right: 10px;
}
</style>
