<template>
    <div class="second-classify">
        <v-breadcrumb :nav="['品牌产品管理','产品分类管理',name]"></v-breadcrumb>
        <div class="table-block">
            <el-button v-if="p.addProductCategory_2" type="primary" style="margin-bottom: 20px" @click="addClassify">添加二级类目</el-button>
            <template>
                <el-table :data="tableData" :height="height" border style="width: 100%">
                    <el-table-column prop="id" label="ID" width="180" align="center"></el-table-column>
                    <el-table-column prop="name" label="二级分类" align="center"></el-table-column>
                    <el-table-column prop="product_name" label="一级分类" align="center"></el-table-column>
                    <el-table-column label="图标" width="180" align="center">
                        <template slot-scope="scope">
                            <img :src="scope.row.img" alt="">
                        </template>
                    </el-table-column>
                    <el-table-column prop="status" label="状态" align="center">
                        <template slot-scope="scope">
                            <template v-if="scope.row.status == 1">启用</template>
                            <template v-if="scope.row.status == 2">禁用</template>
                        </template>
                    </el-table-column>
                    <el-table-column v-if="isShowOperate" label="操作" align="center">
                        <template slot-scope="scope">
                            <!--<el-button type="primary" size="small" @click="toSecondClassify(scope.row.id)">参数管理</el-button>-->
                            <el-button v-if="p.updateProductCategory_2" type="warning" size="small" @click="editItem(scope.row)">编辑</el-button>
                            <el-button v-if="p.deleteProductCategory_2" type="danger" size="small" @click="delItem(scope.row.id)">删除</el-button>
                        </template>
                    </el-table-column>
                </el-table>
            </template>
            <div class="block">
                <el-pagination
                    background
                    @size-change="handleSizeChange"
                    @current-change="handleCurrentChange"
                    :current-page="page.currentPage"
                    layout="total, prev, pager, next, jumper"
                    :total="page.totalPage">
                </el-pagination>
            </div>
        </div>

        <!--添加/编辑类目弹窗-->
        <el-dialog :title="title" :visible.sync="addMask">
            <el-form v-model="addForm">
                <el-form-item label="类目名称" :label-width="formLabelWidth">
                    <el-input v-model="addForm.name" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="类目图标" :label-width="formLabelWidth" class="icon-area">
                    <el-input readonly v-model="addForm.img" auto-complete="off"></el-input>
                    <el-upload class="icon-uploader"
                               action="/commonAPI/ossClient/aliyunOSSUploadImage"
                               :on-success="handleAvatarSuccess">
                        <el-button size="small" type="primary"><i class="el-icon-upload"></i>上传</el-button>
                    </el-upload>
                </el-form-item>
                <el-form-item label="一级分类" :label-width="formLabelWidth">
                    <el-input v-model="name" auto-complete="off" disabled=""></el-input>
                </el-form-item>
                <el-form-item label="是否启用" :label-width="formLabelWidth">
                    <el-select v-model="addForm.status">
                        <el-option label="是" value="1"></el-option>
                        <el-option label="否" value="2"></el-option>
                    </el-select>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button type="primary" @click="addOrEdit('addForm')">确 认</el-button>
                <el-button @click="cancel">取 消</el-button>
            </div>
        </el-dialog>
        <el-dialog :title="title" :visible.sync="editMask">
            <el-form v-model="form">
                <el-form-item label="类目名称" :label-width="formLabelWidth">
                    <el-input v-model="form.name" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="类目图标" :label-width="formLabelWidth" class="icon-area">
                    <el-input readonly v-model="form.img" auto-complete="off"></el-input>
                    <el-upload class="icon-uploader"
                               action="/commonAPI/ossClient/aliyunOSSUploadImage"
                               :on-success="handleAvatarSuccess">
                        <el-button size="small" type="primary"><i class="el-icon-upload"></i>上传</el-button>
                    </el-upload>
                </el-form-item>
                <el-form-item label="一级分类" :label-width="formLabelWidth">
                    <el-input v-model="name" auto-complete="off" disabled=""></el-input>
                </el-form-item>
                <el-form-item label="是否启用" :label-width="formLabelWidth">
                    <el-select v-model="form.status">
                        <el-option label="是" value="1"></el-option>
                        <el-option label="否" value="2"></el-option>
                    </el-select>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button type="primary" @click="addOrEdit('form')">确 认</el-button>
                <el-button @click="cancel">取 消</el-button>
            </div>
        </el-dialog>
        <!--删除弹窗-->
        <delete-toast :id='delId' :url='delUrl' :uri='delUri' @msg='deleteToast' v-if="isShowDelToast"></delete-toast>
    </div>
</template>

<script>
    import vBreadcrumb from "../../common/Breadcrumb.vue";
    import icon from "../../common/ico.vue";
    import deleteToast from "../../common/DeleteToast";
    import * as api from "../../../api/api";
    import utils from '../../../utils/index.js'
    import * as pApi from '../../../privilegeList/index.js';

    export default {
        components: {
            vBreadcrumb,
            icon,
            deleteToast
        },
        data() {
            return {
                // 权限控制
                p:{
                    addProductCategory_2:false,
                    updateProductCategory_2:false,
                    deleteProductCategory_2:false,
                },
                isShowOperate:true,

                tableData: [],
                page: {
                    currentPage: 1,
                    totalPage: 20
                },
                height: "",
                addMask: false,
                editMask: false,
                isShowDelToast: false,
                formLabelWidth: "100px",
                form: {
                    name: "",
                    status: "1",
                    img: ""
                },
                addForm: {
                    name: "",
                    status: "1",
                    img: ""
                },
                title: "添加二级类目",
                id: "",
                itemId: "",
                name: "",
                itype: "",
                delId: 66,
                delUrl: "http://api",
                delUri:''
            };
        },
        created() {
            let winHeight = window.screen.availHeight - 500;
            this.height = winHeight;
            this.pControl();
        },
        activated() {
            this.pControl();
            this.name =
                this.$route.query.name ||
                JSON.parse(sessionStorage.getItem("secondClassify").name);
            this.id =
                this.$route.query.id ||
                JSON.parse(sessionStorage.getItem("secondClassify").id);
            this.getList(this.page.currentPage);
        },
        methods: {
            // 权限控制
            pControl() {
                for (const k in this.p) {
                    this.p[k] = utils.pc(pApi[k]);
                }
                if (!this.p.updateProductCategory_2 && !this.p.deleteProductCategory_2) {
                    this.isShowOperate = false;
                }
            },
            //获取列表
            getList(val) {
                let that = this;
                let data = {
                    page: val,
                    fatherid: this.id,
                    url:pApi.queryProductCategoryPageList_2
                };
                this.$axios
                    .post(api.getCategoryList, data)
                    .then(res => {
                        if (res.data.code == 200) {
                            this.tableData = [];
                            this.tableData = res.data.data.data;
                            this.page.totalPage = res.data.data.resultCount;
                        } else {
                            this.$message.warning(res.data.msg);
                        }
                        console.log(res.data);
                    })
                    .catch(err => {
                        console.log(err);
                    });
            },
            //分页
            handleSizeChange(val) {
                console.log(`每页 ${val} 条`);
            },
            handleCurrentChange(val) {
                console.log(`当前页: ${val}`);
                this.page.currentPage = val;
                this.getList(val);
            },
            // 添加二级类目
            addClassify() {
                this.title = "添加二级类目";
                this.addMask = true;
                this.addForm.name='';
                this.addForm.img='';
                this.addForm.status='1';
                this.itype = "add";
            },
            //编辑
            editItem(row) {
                this.title = "编辑二级类目";
                this.editMask = true;
                row.status = row.status.toString();
                this.form = row;
                this.itemId = row.id;
                this.itype = "edit";
            },
            //添加修改确定
            addOrEdit(formName) {
                let url = "";
                let data = {};
                data.name = this[formName].name;
                data.img = this[formName].img;
                data.status = this[formName].status;
                if(!data.name){
                    this.$message.warning('请输入类目名称!');
                    return
                }
                if(!data.img){
                    this.$message.warning('请上传类目图标!');
                    return
                }
                if (this.itype == "add") {
                    url = api.addCategory;
                    data.fatherid = this.id;
                    data.url = pApi.addProductCategory_2;
                } else {
                    url = api.editCategory;
                    data.id = this.itemId;
                    data.url = pApi.updateProductCategory_2;
                }
                this.btnLoading = true;
                this.$axios
                    .post(url, data)
                    .then(res => {
                        if (res.data.code == 200) {
                            this.$message.success(res.data.data);
                            this.btnLoading = false;
                            this.addMask = false;
                            this.editMask = false;
                            this.getList(this.page.currentPage);
                        } else {
                            this.btnLoading = false;
                            this.$message.warning(res.data.msg);
                            this.getList(this.page.currentPage);
                        }
                    })
                    .catch(err => {
                        console.log(err);
                    });
            },
            //删除
            delItem(id) {
                this.delId = id;
                this.delUrl = api.deleteCategory;
                this.delUri = pApi.deleteProductCategory_2;
                this.isShowDelToast = true;
            },
            // 删除弹窗
            deleteToast(msg) {
                this.isShowDelToast = msg;
                this.getList(this.page.currentPage);
            },
            //上传图片
            handleAvatarSuccess(res, file) {
                if (this.itype == "add") {
                    this.addForm.img = URL.createObjectURL(file.raw);
                } else {
                    this.form.img = URL.createObjectURL(file.raw);
                }
            },
            //取消
            cancel() {
                this.addMask = false;
                this.editMask = false;
                this.getList(this.page.currentPage);
            }
        }
    };
</script>

<style lang="less">
    .second-classify {
        /*表格样式*/
        .table-block {
            padding: 20px 20px 60px;
            background: #fff;
        }
        img {
            width: 38px;
            height: 38px;
            border-radius: 5px;
            vertical-align: middle;
        }
        .block {
            float: right;
            margin-top: 10px;
        }

        /*弹窗样式*/
        .el-dialog {
            width: 530px;
            border-radius: 10px;
        }
        .el-dialog__header {
            border-bottom: 1px solid #eee;
            padding: 20px 20px 10px 50px;
        }
        .el-dialog__title {
            color: #ff6868;
        }
        .el-dialog .el-input {
            display: inline;
        }
        .el-dialog .el-input__inner {
            width: 360px;
        }
        .el-select .el-input__inner {
            width: 200px;
        }
        .el-dialog .el-upload--text {
            width: 100px;
            height: 40px;
            border: none;
        }
        .icon-area .el-input__inner {
            width: 240px;
        }
        .el-input__suffix {
            line-height: 24px;
        }
        .icon-uploader {
            float: right;
            margin-right: 31px;
            height: 33px;
        }
        .icon-uploader .el-button--small {
            border-radius: 5px;
            width: 100px;
        }
        .el-upload--text .el-icon-upload {
            line-height: 0;
            margin: 0;
            color: #fff;
            font-size: 14px;
        }
        .el-dialog__footer {
            margin-right: 30px;
        }
        .el-upload-list {
            display: none;
        }
    }
</style>