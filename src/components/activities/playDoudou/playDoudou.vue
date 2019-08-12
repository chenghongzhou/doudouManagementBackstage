<template>
    <!-- 幸运树 -->
    <section>
        <el-col :span="24" class="toolbar" style="padding-bottom:0px;">
            <el-form :inline="true" style="overflow:hidden;">
                <el-row>
					<el-form-item>
                        <el-button 
                        type="primary" 
                        @click="addNewloading.dialogFormVisible=true;">添加</el-button>
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
                <el-table-column prop="sort" label="排序"></el-table-column>
                <el-table-column prop="type" label="物品类型">
					<template slot-scope="scope">
						<div slot="reference" class="name-wrapper">
							<p v-if="scope.row.type == 1">背包礼物</p>
							<p v-else-if="scope.row.type==2">装扮</p>
							<p v-else-if="scope.row.type==3">座驾</p>
							<p v-else-if="scope.row.type==4">称号</p>
						</div>
					</template>
				</el-table-column>
                <el-table-column prop="name" label="物品名称"></el-table-column>
				<el-table-column prop="probability_1" label="普通概率"></el-table-column>
				<el-table-column prop="probability_2" label="暴走概率"></el-table-column>
                <el-table-column prop="img" label="图片">
                    <template slot-scope="scope">
                        <div slot="reference" class="name-wrapper">
							<img 
							:src="scope.row.icon"
							style="width:100px;height:100px;">
						</div>
                    </template>
                </el-table-column>
				<el-table-column prop="chat_gold" label="价值豆币"></el-table-column>
				<el-table-column prop="on_time" label="上架时间"></el-table-column>
                <el-table-column label="上架状态" width="80">
					<template slot-scope="scope">
						<div slot="reference" class="name-wrapper">
							<p v-if="scope.row.status == 1">已上架</p>
							<p v-else-if="scope.row.status==0">已下架</p>
						</div>
					</template>
				</el-table-column>
				<el-table-column prop="off_time" label="下架时间"></el-table-column>
                <el-table-column label="操作">
                    <template slot-scope="scope">
                        <el-row>
                            <el-button 
                                size="mini" 
                                type="primary" 
                                @click="modify(scope.$index, scope.row)">编辑</el-button>
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
		<!-- 添加 -->
		<el-dialog title="添加" :visible.sync="addNewloading.dialogFormVisible">
			<el-form :model="addNewloading.params"> 
				<el-form-item label="物品类型" :label-width="formLabelWidth">
					<el-select v-model="addNewloading.params.type" @change="checkType">
						<el-option label="背包礼物" value="1"></el-option>
						<el-option label="装扮" value="2"></el-option>
						<el-option label="座驾" value="3"></el-option>
						<el-option label="称号" value="4"></el-option>
					</el-select>
				</el-form-item> 
				<el-form-item label="物品" :label-width="formLabelWidth">
					<el-select v-model="addNewloading.params.prop_id" @change="checkMoney">
						<el-option 
						:label="item.name" 
						:value="item.id"
						:key="index"
						v-for="(item,index) in wp">
						</el-option>
					</el-select>
				</el-form-item> 
				<el-form-item label="物品价格" :label-width="formLabelWidth" v-show="priceBox == true">
					{{addNewloading.params.chat_gold}}
				</el-form-item>
				<!-- <el-form-item label="图片上传" :label-width="formLabelWidth">
					<span class="showbtn">选择文件</span>
					<input 
					class="filepic fileinput" 
					@change="uploading($event)" 
					type="file">
					<span class="showname">{{addNewloading.params.pic_name}}</span>
			        <img 
					:src="addNewloading.params.icon" 
					style="width:200px;height:auto;margin-left:200px;"/>
				</el-form-item> -->
				<el-form-item label="图片" :label-width="formLabelWidth">
					<!-- <span class="showname">{{editorloading.pic_name}}</span> -->
			        <img 
					:src="addNewloading.params.icon" 
					style="width:200px;height:auto;margin-left:200px;"/>
				</el-form-item>
				<el-form-item label="普通概率" :label-width="formLabelWidth">
					<el-input 
					style="width:250px"
					v-model="addNewloading.params.probability_1" 
					auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="暴走概率" :label-width="formLabelWidth">
					<el-input 
					style="width:250px"
					v-model="addNewloading.params.probability_2" 
					auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="上线设置" :label-width="formLabelWidth">
					<el-select v-model="addNewloading.params.status">
						<el-option label="上线" value="1"></el-option>
						<el-option label="暂不上线" value="0"></el-option>
					</el-select>
				</el-form-item>
				<el-form-item label="排序ID" :label-width="formLabelWidth">
					<el-select v-model="addNewloading.params.sort" @focus="getSortEvery">
						<el-option 
							:label="item" 
							:value="item"
							:key="index"
							v-for="(item,index) in getSort">
							</el-option>
					</el-select><span style="color:#ccc">上线选择大于0，不上线选择0</span>
				</el-form-item> 
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button 
				@click.native.prevent="addBannerSure(0)">取 消</el-button>
				<el-button 
				type="primary" 
				@click.native.prevent="addBannerSure(1)">确 定</el-button>
			</div>
		</el-dialog>
         <!--编辑-->
        <el-dialog title="编辑" :visible.sync="editorloading.dialogFormVisible">
			<el-form :model="editorloading.params">
				<el-form-item label="物品类型" :label-width="formLabelWidth">
					<el-select v-model="editorloading.params.type" disabled>
						<el-option label="背包礼物" value="1"></el-option>
						<el-option label="装扮" value="2"></el-option>
						<el-option label="座驾" value="3"></el-option>
						<el-option label="称号" value="4"></el-option>
					</el-select>
				</el-form-item> 
				<el-form-item label="物品" :label-width="formLabelWidth" disabled>
					<el-input 
					style="width:250px"
					v-model="editorloading.params.name" 
					auto-complete="off" disabled></el-input>
				</el-form-item>
				<el-form-item label="物品价格" :label-width="formLabelWidth" v-show="priceBox == true">
					<el-input 
					style="width:250px"
					v-model="editorloading.params.chat_gold" 
					auto-complete="off" disabled></el-input>
				</el-form-item>
				<el-form-item label="图片" :label-width="formLabelWidth">
					<!-- <span class="showname">{{editorloading.pic_name}}</span> -->
			        <img 
					:src="editorloading.params.icon" 
					style="width:200px;height:auto;margin-left:200px;"/>
				</el-form-item>
				<el-form-item label="普通概率" :label-width="formLabelWidth">
					<el-input 
					style="width:250px"
					v-model="editorloading.params.probability_1" 
					auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="暴走概率" :label-width="formLabelWidth">
					<el-input 
					style="width:250px"
					v-model="editorloading.params.probability_2" 
					auto-complete="off"></el-input>
				</el-form-item>

				<el-form-item label="上线设置" :label-width="formLabelWidth">
					<el-select v-model="editorloading.params.status">
						<el-option label="上线" value="1"></el-option>
						<el-option label="暂不上线" value="0"></el-option>
					</el-select>
				</el-form-item>
				<el-form-item label="排序ID" :label-width="formLabelWidth">
					<el-select v-model="editorloading.params.sort" @focus="getSortEvery">
						<el-option 
							:label="item" 
							:value="item"
							:key="index"
							v-for="(item,index) in getSort">
							</el-option>
					</el-select><span style="color:#ccc">上线选择大于0，不上线选择0</span>
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
			addNewloading:{
				dialogFormVisible:false,
				params:{
					type:'',
					prop_id:'',
					icon:'',
					probability:'',
					chat_gold:0,
					status:'',
					sort:'',
					pic_name:'',
					file_pic:'',
					prize_id:'',
					threshold:'',
					probability_1:'',
					probability_2:'',
				}
			},
			editorloading:{
				dialogFormVisible:false,
				params:{
					type:'',
					prop_id:'',
					icon:'',
					probability:'',
					status:'',
					sort:'',
					pic_name:'',
					prize_id:'',
					name:'',
					threshold:'',
					probability_1:'',
					probability_2:'',
				}
			},
			wp:[],
			getSort:['0'],
			money:'',
			getMoneyList:['0'],
            tableHeightOther: '200px'
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
            let url = "/NewBean/getPrizeList";
            let params = {
                page: _this.page,
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
		//添加时选择物品类型后请求接口获取对应的物品
		checkType(){
			var _this = this;
			_this.wp = [];
			let url = "/NewBean/getGiftOrPropList";
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
		//添加
		addBannerSure(num){
			var _this = this;
			if(num == 0){
				_this.addNewloading.dialogFormVisible = true;
				_this.clearAddFrom();
			}else if(num == 1){
				_this.listLoading = true;
				let formData = new FormData();
				formData.append('prize_id', _this.addNewloading.params.prize_id);
				formData.append('type', _this.addNewloading.params.type);
				formData.append('prop_id', _this.addNewloading.params.prop_id);
				formData.append('sort', _this.addNewloading.params.sort);
				formData.append('threshold', _this.addNewloading.params.threshold);
				formData.append('status', _this.addNewloading.params.status);
				formData.append('probability_1', _this.addNewloading.params.probability_1);
				formData.append('probability_2', _this.addNewloading.params.probability_2);
			  	formData.append('icon', _this.addNewloading.params.file_pic); //提交的新增图标的文件
				let config = {
					headers: {
						'Content-Type': 'multipart/form-data'
					}
				};	
				axios.post(allget+'/NewBean/setPrize', formData, config)
					.then((res) => {
						_this.listLoading = false;							
						if(res.data.ret == 1) {	
							_this.addNewloading.dialogFormVisible = true;
							baseConfig.successTipMsg(_this, '添加成功');
							_this.getData();
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg);						
						}
						_this.clearAddFrom();
					})
					.catch((err) => {
						console.log(err);
					});
			}
		},
		//编辑
		modify(index,rows){
			this.editorloading.dialogFormVisible = true;
			if(rows.type == 4){
				this.priceBox = true;
				this.starBox = true;
			}else if(rows.type == 1){
				this.priceBox = true;
			}else{
				this.priceBox = false;
				this.starBox = false;
			};
			this.editorloading.params.type = rows.type;
			this.editorloading.params.name = rows.name;
			this.editorloading.params.id = rows.prop_id;
			this.editorloading.params.chat_gold = rows.chat_gold;
			this.editorloading.params.icon = rows.icon;
			this.editorloading.params.status = rows.status;
			this.editorloading.params.probability_1 = rows.probability_1;
			this.editorloading.params.probability_2 = rows.probability_2;
			this.editorloading.params.sort = rows.sort;
			this.editorloading.params.prize_id = rows.prize_id;
			this.editorloading.params.threshold = rows.threshold;
		},
		//获取排序
		checkSort(box){
			var _this = this;
			_this.getSort = ['0'];
			let url = "/NewBean/getPrizeSortList";
            let params = {
               
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
				formData.append('prize_id', _this.editorloading.params.prize_id);
				formData.append('type', _this.editorloading.params.type);
				formData.append('prop_id', _this.editorloading.params.prop_id);
				formData.append('sort', _this.editorloading.params.sort);
				formData.append('status', _this.editorloading.params.status);
				formData.append('probability_1', _this.editorloading.params.probability_1);
				formData.append('probability_2', _this.editorloading.params.probability_2);
				formData.append('threshold', _this.editorloading.params.threshold);
			  	formData.append('icon', _this.editorloading.params.icon); //提交的新增图标的文件
				let config = {
					headers: {
						'Content-Type': 'multipart/form-data'
					}
				};	
				axios.post(allget+'/NewBean/setPrize', formData, config)
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
					type:'',
					prop_id:'',
					id:'',
					icon:'',
					probability_1:'',
					probability_2:'',
					chat_gold:0,
					status:'',
					sort:'',
					pic_name:'',
					file_pic:'',
					prize_id:'',
					name:'',
					threshold:''
				}
			};
			_this.priceBox=false;
			_this.starBox=false;
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
					probability_1:'',
					probability_2:'',
					chat_gold:0,
					status:'',
					sort:'',
					pic_name:'',
					file_pic:'',
					prize_id:'',
					name:'',
					id:'',
					threshold:''
				}
			};
			_this.priceBox=false;
			_this.starBox=false;
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
                var url = '/NewBean/deletePrize';
                let params = {
                    prize_id : row.prize_id
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
</style>
