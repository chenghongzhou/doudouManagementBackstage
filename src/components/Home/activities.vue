<!-- 系统管理的总体的框架 -->
<template lang="html">
    <el-row>
        <!-- 左侧的导航栏 -->
        <div class="left_active" ref="leftnav">
            <!-- 循环相应的路由表生成左侧的路由表一级路由表 -->
            <el-menu 
            theme="dark" 
            :default-active="indexPath" 
            class="el-menu-vertical-demo" 
            @open="handleOpen" 
            @close="handleClose" 
            router>
                <div v-for="(item, indexs) in dataView">
                    <el-submenu :index="indexs+''">
                        <template slot="title"><i :class="item.iconCls"></i>{{item.name}}</template>
                        <template v-for="child in item.children">
                            <el-menu-item-group >                                
                                <el-menu-item :index="child.path" v-show="!child.hidden">{{child.name}}</el-menu-item>
                            </el-menu-item-group>
                        </template>
                    </el-submenu>
                </div>
                <!-- 判断进行设置了为hidden: true;不进行展示 ,下面是没有子菜单的 -->                
                <el-menu-item 
                v-for="(item, indexs) in dataOtherView" 
                :key="indexs" 
                :index="item.path" 
                v-show="!item.hidden">
                    <i :class="item.iconCls"></i>
                    <span slot="title">{{item.name}}</span>
                </el-menu-item>
            </el-menu>
        </div>
        <el-col :span="24" class="content-wrapper">
            <transition name="fade" mode="out-in">
                <router-view></router-view>
            </transition>
        </el-col>
    </el-row>
</template>

<script>
import store from '../../vuex/store';
export default {
    data() {
        return {
            indexPath: '',
            indexPathArr: [],     
            indexOTherPathArr:[],       
        };
    },
    computed: {
        //有子菜单
        dataView() {
            var _this = this;
            let thatDdta = store.getters.addRouters;
            let data =  thatDdta.filter(data => {
                if(data.name=='活动专区'){
                    return  data;
                }
            });
            // var arr = data[0].children[0].children;            
            // for(var i=0; i<arr.length; i++) {
            //     _this.indexPathArr.push(arr[i].path);
            // } 
            // return data[0].children[0].children;
             var arr = data[0].children.slice(0,data[0].children.length-1); 
             var arrOther =  data[0].children.slice(data[0].children.length-1,data[0].children.length)[0].children;     
            for(var i=0; i<arr.length; i++) {
                for(var j=0; j<arr[i].children.length; j++) {
                    _this.indexPathArr.push(arr[i].children[j].path);
                }
            } 
             return arr;
        },
        //没有子菜单
        dataOtherView(){
            var _this = this;
            let thatDdta = store.getters.addRouters;
            let data =  thatDdta.filter(data => {
                if(data.name=='活动专区'){
                    return  data;
                }
            });
             var arrOther = data[0].children.slice(data[0].children.length-1,data[0].children.length)[0].children;
            for(var j=0; j<arrOther.length; j++) {
                _this.indexOTherPathArr.push(arrOther[j].path);
            } 
             return arrOther;
        }
    },
    methods: {
        handleOpen(key, keyPath) {
            //console.log(key, keyPath);
        },
        handleClose(key, keyPath) {
            //console.log(key, keyPath);
        }
    },
    mounted() {
        var _this = this;
        _this.$nextTick(function() {
            _this.$refs.leftnav.style.height = leftNavHeight +'px';
            var strPath = location.href;
            if(strPath.indexOf('http://')==0) {
                strPath = strPath.substring(strPath.indexOf('#/')+1, strPath.length);
            }
            var index = strPath.indexOf('/');
            strPath = strPath.substring(index, strPath.length);
            _this.indexPath = strPath;
		})
    },
    updated() {
        var _this = this;
        var strPath = location.href;
        if(strPath.indexOf('http://')==0) {
            strPath = strPath.substring(strPath.indexOf('#/')+1, strPath.length);
        }
        var index = strPath.indexOf('/');
        strPath = strPath.substring(index, strPath.length);
        for(var i=0; i<_this.indexPathArr.length; i++) {
            if(strPath.indexOf(_this.indexPathArr[i])!=-1) {
                _this.indexPath = _this.indexPathArr[i];
                break;
            }
        }
    },
}
</script>

<style lang="css">

</style>
