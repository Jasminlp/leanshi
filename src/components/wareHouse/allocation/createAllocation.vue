<template>
    <el-form  label-width="90px"  ref="form" :model="form"  :rules="rules" class="from-good">
        <el-row>
            <el-col :span="6" :xs="24" :sm="24" :md="10" :lg="6" :xl="6">
                <el-form-item label="转出仓库" prop="wareNameFrom" class="search">
                    <el-input v-model="form.wareNameFrom" class="serch-input" disabled></el-input>
                    <i class="el-icon-search" @click="DialogShow(1)"></i>
                </el-form-item>
            </el-col>
        </el-row>
        <el-row>
            <el-col :span="6" :xs="24" :sm="24" :md="10" :lg="6" :xl="6">
                <el-form-item label="转入仓库" prop="wareNameTo" class="search">
                    <el-input v-model.trim="form.wareNameTo" class="serch-input" disabled></el-input>
                    <i class="el-icon-search" @click="DialogShow(2)"></i>
                </el-form-item>
            </el-col>
        </el-row>
        <el-row>
            <el-col :span="12" :xs="24" :sm="24" :md="12" :lg="12" :xl="12">
                <el-form-item label="上传附件">
                    <el-upload 
                        action="/apis/member/uploadFile"
                        list-type="picture"
                        accept=".jpg, .png, .bmp"
                        :headers="token"
                        :on-success="uploadSuccess"
                        :on-error="uploadError"
                        v-model="form.file"
                        :limit=1
                        ref="upload"
                        :before-upload="beforeUpload">
                        <div slot="tip" class="el-upload__tip">只能上传一张格式为png/JPG/bmp文件</div>
                        <el-input><el-button slot="append" icon="el-icon-upload2"></el-button></el-input>
                    </el-upload>
                </el-form-item>
            </el-col>
        </el-row>
        <el-row>
            <el-col :span="6" :xs="24" :sm="24" :md="10" :lg="6" :xl="6">
                <el-form-item label="备注" prop="desc">
                    <el-input v-model.trim="form.desc" type="textarea" :autosize="{ minRows: 3, maxRows: 5}"></el-input>
                </el-form-item>
            </el-col>
        </el-row>
        <br/>
        <el-row>
            <el-col :span="24">
                <el-form-item label="商品信息">
                    <el-button type="primary" @click="DialogShowGoods" icon="el-icon-plus">增 加</el-button>
                    <el-button  icon="el-icon-refresh" @click="upload">刷 新</el-button>
                    <el-button type="danger" @click="deleteGoods" icon="el-icon-delete" :disabled="searchData.length==0">删 除</el-button>
                </el-form-item>
            </el-col>
        </el-row>
        <el-row>
            <el-col :span="24">
                <el-table 
                    :data="searchData" 
                    @selection-change="handleSelectionChange"
                    v-loading="loadingTable" 
                    element-loading-text="拼命加载中"
                    element-loading-spinner="el-icon-loading">
                    <el-table-column type="selection" align="center" width="50px">
                    </el-table-column>
                    <el-table-column type="index" prop="" label="序号" align="center" :show-overflow-tooltip="true">
                    </el-table-column>
                    <el-table-column prop="goodsId" label="商品ID" align="center" :show-overflow-tooltip="true">
                    </el-table-column>
                    <el-table-column prop="goodsName" label="商品名称" align="center" :show-overflow-tooltip="true">
                    </el-table-column>
                    <el-table-column label="规格值" align="center" width="100">
                        <template slot-scope="scope">
                            <p v-for="(item,index) in scope.row.specGoodsSpec2" :key="index">{{item}}</p>
                        </template>
                    </el-table-column>
                    <el-table-column label="规格" align="center" width="100">
                        <template slot-scope="scope">
                            <p v-for="(item,index) in scope.row.specName2" :key="index"> {{item}}</p>
                        </template>
                    </el-table-column>
                    <el-table-column prop="inventory" width="120" label="现有库存数量" align="center">
                    </el-table-column>
                    <el-table-column label="出入库数量" align="center" width="90">
                        <template slot-scope="scope">
                            <el-input v-model="scope.row.stockInto" type="number" min="0"></el-input>
                        </template>
                    </el-table-column>
                    <el-table-column prop="createTime" label="生产日期" align="center" width="160">
                        <template slot-scope="scope">
                            <el-date-picker 
                                v-model="scope.row.createTime" 
                                type="date" 
                                value-format="yyyy-MM-dd"
                                placeholder="请选择日期"
                                class="cell-date">
                            </el-date-picker>
                        </template>
                    </el-table-column>
                    <el-table-column prop="shelfLife" label="保质期(天)" align="center" width="90">
                    </el-table-column>
                    <el-table-column label="到期日期" align="center" width="160">
                        <template slot-scope="scope">
                            <el-date-picker 
                                disabled
                                v-model="scope.row.shelfLifeTime" 
                                type="date" 
                                value-format="yyyy-MM-dd"
                                placeholder="请选择日期"
                                class="cell-date">
                            </el-date-picker>
                        </template>
                    </el-table-column>
                </el-table>
            </el-col>
        </el-row>
        <br/>
        <el-row>
            <el-col :span="24" align="center">
                <el-button type="primary" @click="createForm('form')" :disabled="searchData.length==0">创 建</el-button>
            </el-col>
        </el-row>

        <!-- 仓库对话框 -->
        <el-dialog :visible.sync="DialogTable" width="90%" center>
            <el-form label-width="70px" label-position="left">
                <el-row type="flex" justify="center">
                    <el-col :span="5">
                        <el-form-item label="仓库代码">
                            <el-input v-model="WHCode" clearable></el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="3">
                        <el-form-item>
                            <el-button type="primary" @click="onSearchWH" icon="el-icon-search">搜 索</el-button>
                        </el-form-item>
                    </el-col>
                </el-row>
            </el-form>
            <el-table 
                ref="multipleTable" 
                :data="tableData" 
                tooltip-effect="dark" 
                border
                @row-dblclick="rowDblclick"
                v-loading="loadingWH" 
                element-loading-text="拼命加载中"
                element-loading-spinner="el-icon-loading">
                <el-table-column label="选择" type="" width="55" align="center">
                    <template slot-scope="scope">
                        <el-radio class="radio" v-model="selectNum" :disabled="scope.row.wareName==disabledSelect" :label="scope.$index" @change.native="scope.row.wareName==disabledSelect?'':getCurrentRow(scope.$index)">&nbsp;</el-radio>
                    </template>
                </el-table-column>
                <el-table-column prop="wareCode" label="仓库代码" align="center">
                </el-table-column>
                <el-table-column prop="wareName" label="仓库名称" align="center">
                </el-table-column>
                <el-table-column prop="place" label="仓库地点" align="center">
                </el-table-column>
                <el-table-column prop="createBy" label="创建人" align="center">
                </el-table-column>
                <el-table-column prop="createTime" label="创建时间" align="center">
                </el-table-column>
                <el-table-column prop="updateTime" label="最后修改时间" align="center">
                </el-table-column>
                <el-table-column prop="wareDesc" label="备注" align="center">
                </el-table-column>
            </el-table>
            <br>
            <el-row type="flex" justify="center">
                <el-col :span="8" align="center">
                    <el-pagination
                        :page-size="WHpageData.pageSize"
                        layout="total, sizes, prev, pager, next"
                        :page-sizes="[10,20,50,999]"
                        :total="WHpageData.total"
                        :current-page="WHpageData.currentPage"
                        @current-change="onWHChangePage"
                        @size-change="WHhandleSizeChange">
                    </el-pagination>
                </el-col>
            </el-row>
            <span slot="footer" class="dialog-footer">
                <el-button @click="DialogTable = false">取 消</el-button>
                <el-button type="primary" @click="onSend">确 定</el-button>
            </span>
        </el-dialog>
        
        <!-- 弹出层组件 -->
        <tableCom ref="dialog" @goodsData="getGoodsData"></tableCom>

    </el-form>
</template>


<script>
import tableCom from '../dialogCom';
import Cookies from 'js-cookie';
export default {
    name:"createAllocation",
    data() {
        return {
            token:{},
            form:{
                wareNameFrom:"", //转出仓库
                wareNameTo:"",    //转入仓库
                wareCode:"",
                file:[], //附件
                desc:""
            },
            getWHType:null, //选择仓库类型
            disabledSelect:false, //禁止选择仓库
            loadingTable:false, //加载列表
            searchData: [], //商品列表数据
            deleteGoodsData:[] , //删除商品数据 

            loadingWH:false, //加载仓库列表
            DialogTable:false, //仓库对话框
            WHCode:"",    //仓库代码搜索
            tableData: [],  //仓库数据
            selectNum:null,
            //仓库分页数据
            WHpageData:{
                currentPage:1,
                pageSize:10,
                total:0,
            },
            //表单验证规则
            rules: {
                wareNameFrom: [
                    { required: true, message: "请选择入库地点", trigger: ['blur'] },
                ],
                wareNameTo: [
                    { required: true, message: "请选择入库地点", trigger: ['blur'] },
                ],
                desc: [
                    { required: true, message: "请输入备注", trigger: ['blur'] },
                ],
            }
        };
    },
    components:{
        tableCom
    },
    methods: {
        //改变仓库页数
        onWHChangePage(currentPage) {
            this.WHpageData.currentPage = currentPage;
            this.onSearchWH();
        },
        //仓库每页条数改变
        WHhandleSizeChange(pageSize) {
            this.WHpageData.pageSize = pageSize;
            this.onSearchWH();
        },
        //仓库搜索弹框
        DialogShow(val){
            this.DialogTable = true;
            this.selectNum = null;
            this.getWHType = val;
            if(val==1){
                this.disabledSelect = this.form.wareNameTo;
            }else if(val==2){
                this.disabledSelect = this.form.wareNameFrom;
            }
            this.onSearchWH();
        },
        //仓库搜索
        onSearchWH() {
            this.tableData = [];
            this.loadingWH = true;  
            this.$request({
                method:'post',
                url:"/apis/member/findWare",
                params:{
                    wareCode:this.WHCode,
                    currentPage:this.WHpageData.currentPage,
                    pageSize:this.WHpageData.pageSize
                }
            })     
            .then(response=>{
                if(response.data.code){
                    this.tableData = response.data.data.list;
                    for(let i in this.tableData){
                        this.tableData[i].place = this.tableData[i].provinceCode+"-"+this.tableData[i].cityCode+"-"+this.tableData[i].countryCode+"-"+this.tableData[i].wareDetial;
                    }
                    this.WHpageData.currentPage = response.data.data.pageNum,
                    this.WHpageData.pageSize = response.data.data.pageSize,
                    this.WHpageData.total = response.data.data.total
                }
                setTimeout(()=>{
                    this.loadingWH = false;
                },200)
            })
        },
        //双击仓库某行
        rowDblclick(row){
            if(row.wareName==this.disabledSelect){
                return;
            }
            if(this.getWHType==1){
                this.form.wareNameFrom = row.wareName;
                this.form.wareCode = row.wareCode;
            }else if(this.getWHType==2){
                this.form.wareNameTo = row.wareName;
            }
            this.searchData = [];
            this.DialogTable = false;
        },
        //搜索层选中数据,返回选中行下标
        getCurrentRow(val) {
            this.selectNum = val;
        },
        //选中数据 
        onSend(){
            //如果未选择数据
            if(this.selectNum==null){
                this.$message({
                    showClose: true,
                    message: '请先选择仓库',
                    type: 'error'
                });
            }else {
                if(this.getWHType==1){
                    this.form.wareNameFrom = this.tableData[this.selectNum].wareName;
                    this.form.wareCode = this.tableData[this.selectNum].wareCode;
                }else if(this.getWHType==2){
                    this.form.wareNameTo = this.tableData[this.selectNum].wareName;
                }
                this.searchData = [];
                this.DialogTable = false;
            }
        },
        //图片上传成功
        uploadSuccess(response) {
            if(response.code){
                this.form.file = response.data;
            }
        },
        //图片上传失败
        uploadError() {
            this.$message({
                showClose: true,
                message: '服务器未响应,上传失败',
                type: 'error'
            });
        },
        //上传文件之前验证类型和大小
        beforeUpload(file) {
            const fileType = file.type=="image/jpeg"||file.type=="image/png"||file.type=="image/bmp";
            const fileSize = file.size / 1024 / 1024 <= 1;
            if(!fileType){
                this.$message({
                    showClose: true,
                    message: '只能上传png/JPG/bmp文件',
                    type: 'error'
                });
            }
            if(!fileSize){
                this.$message({
                    showClose: true,
                    message: '图片不超过1M',
                    type: 'error'
                });
            }
            return fileType && fileSize;
        },
        //显示商品信息弹框
        DialogShowGoods(){
            if(!this.form.wareNameFrom || !this.form.wareNameTo) {     //未选择用户
                this.$message({
                    showClose: true,
                    message: '请先选择仓库地点',
                    type: 'info'
                });       
            }else{
                this.$refs.dialog.DialogShowGoods(this.form.wareCode, this.searchData);
            }
        },
        //接收商品
        getGoodsData(data){
            this.searchData = data;
        },
        //选中删除商品
        handleSelectionChange(row) {
            this.deleteGoodsData = row;
        },
        //刷新
        upload(){
            this.loadingTable = true;
            setTimeout(()=>{
                this.loadingTable = false;
            },800)
        },
        //删除商品
        deleteGoods(){
            if(this.deleteGoodsData.length!=0){
                this.loadingTable = true;
                //选中商品id
                let arrDelGoodsId = this.deleteGoodsData.map((item)=>{
                    return item.goodsId;
                });
                
                //找出选中商品并删除
                for(let i = 0; i < this.searchData.length; i++){
                    for(let j = 0; j < arrDelGoodsId.length; j++){
                        if(arrDelGoodsId[j].indexOf(this.searchData[i].goodsId) != -1){
                            this.searchData.splice(i,1);
                            i--; 
                            break;
                        }
                    }
                }
                setTimeout(()=>{
                    this.$message({
                        showClose: true,
                        message: '删除成功!',
                        type: 'success'
                    });
                    this.loadingTable = false;
                },500)
            }else{
                this.$message({
                    showClose: true,
                    message: '请先选择删除的商品!',
                    type: 'wraning'
                });
            }
        },
        //创建调整单
        createForm(form){
            this.$refs[form].validate((valid) => {
                if (valid) {
                    //判断是否有空生产日期
                    let createTimeRight = this.searchData.every((item)=>{
                        return item.createTime != null;
                    })
                    //判断是否有空或0出入库数量
                    let stockIntoRight = this.searchData.every((item)=>{
                        return  item.stockInto > 0;
                    })

                    //判断是否出入库数量大于现有库存数量
                    let stockRight = this.searchData.every((item)=>{
                        return  item.stockInto <= item.inventory;
                    })

                    if(!createTimeRight){
                        this.$message({
                            showClose: true,
                            message: '生产日期不为空',
                            type: 'error'
                        });
                        return;
                    }
                     if(!stockRight){
                        this.$message({
                            showClose: true,
                            message: '出入库数量大于现有库存数量',
                            type: 'error'
                        });
                        return;
                    }
                    if(!stockIntoRight){
                        this.$message({
                            showClose: true,
                            message: '出入库数量不为空或0',
                            type: 'error'
                        });
                        return;
                    }
                    let success = true;
                    new Promise((resolve,reject)=>{
                        this.$request({
                            method:'post',
                            url:"/apis/member/addAllocation",
                            params:{
                                wareNameOut:this.form.wareNameFrom,
                                wareNameIn:this.form.wareNameTo,
                                attachAdd:this.form.file,
                                autohrizeDesc:this.form.desc
                            }
                        })     
                        .then(response=>{
                            resolve(response.data.msg);
                        });
                    })
                    .then((wid)=>{
                        new Promise((resolve,reject)=>{
                            for(let i = 0; i < this.searchData.length; i++){
                                //当保质期和过期日期为空时，传0过去
                                if(!this.searchData[i].shelfLife){
                                    this.searchData[i].shelfLife = 0;
                                    this.searchData[i].shelfLifeTime = 0;
                                }
                                this.$request({
                                    method:'post',
                                    url:"/apis/member/addGA",
                                    params:{
                                        wId:parseInt(wid),
                                        sign:2,
                                        goodIdS:this.searchData[i].goodsId,
                                        goodsName:this.searchData[i].goodsName,
                                        specificationId:this.searchData[i].id,
                                        specName:this.searchData[i].specName,
                                        goodsSpec:this.searchData[i].specGoodsSpec,
                                        stockNow:parseInt(this.searchData[i].inventory),
                                        stockInto:parseInt(this.searchData[i].stockInto),
                                        createTime:this.searchData[i].createTime,
                                        qualityTime:parseInt(this.searchData[i].shelfLife),
                                        shelfLifeTime:this.searchData[i].shelfLifeTime,
                                        precautiousLine:0,
                                    }
                                })     
                                .then(response=>{
                                    if(!response.data.code){
                                        success = false;
                                    }
                                })
                            }
                            resolve(success);
                        })
                        .then((result)=>{
                            if(result){
                                this.$message({
                                    showClose: true,
                                    message: '调拨单创建成功!',
                                    type: 'success'
                                });
                                setTimeout(()=>{
                                    this.onreset();
                                },800)
                            }else{
                                this.$message({
                                    showClose: true,
                                    message: '调拨单创建失败!',
                                    type: 'error'
                                });
                            }
                        })
                    })                                                           
                }else{
                    this.$message({
                        showClose: true,
                        message: '请输入必填信息!',
                        type: 'error'
                    });
                    return false;
                }
            })
        },
        //出初化数据 
        onreset(){
            this.$refs.upload.clearFiles();
            this.searchData = [];
            this.tableData = [];
            this.form = {
                wareNameFrom:"",
                wareNameTo:"",
                file:[], //附件
                desc:""
            }
        }
    },
    created(){
        //上传添加token
        this.token = {
            token:Cookies.get('Authorization')
        }
    }
};
</script>

<style scoped>
.from-good  .serch-input >>> .el-form-item__content{
    position: relative;
}
.from-good .search >>> .el-icon-search{
    position: absolute;
    right: 10px;
    top: 12px;
    font-size: 18px;
    color: #666;
    cursor: pointer;
}
.from-good .cell-date >>> .el-input__prefix{
    top:-6px;
}
.from-good >>> .el-icon-search{
    display: block;
    width:100%;
    height: 23px;
    text-align: right;
}
.el-dialog__wrapper >>> .el-icon-search{
    display: inline;
    width:auto;
    height: auto;
    text-align: center;
}
</style>



