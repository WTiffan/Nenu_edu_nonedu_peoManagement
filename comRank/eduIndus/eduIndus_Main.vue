<template>
    <el-container>
        <div id="eduHome">
        <!-- 搜索框部分 -->
            <el-card id="edu-boxCard">
                <div id="TopTitle">
                    <span>教育行业 / 往年市场开发历史纪录</span>
                </div>
                <el-form :model="searchBox">
                    <!-- 使用栅格设置input框的大小 -->
                    <el-row>
                        <el-col :span="5">
                            <el-form-item prop="searchInput">
                                <el-input v-model="searchBox.searchInput" placeholder="关键字搜索" id="searchIn"></el-input>
                            </el-form-item>
                        </el-col>
                        <el-col :span="3">
                            <el-form-item prop="detailDigital">
                            <!-- el-checkbox的返回值为 true 或 false -->
                                <el-checkbox v-model="searchBox.detailDigital" label="详细数据" ></el-checkbox>
                            </el-form-item>
                        </el-col>
                        <el-col :span="2">
                            <el-form-item prop="digitalClassification">
                                <el-checkbox v-model="searchBox.digitalClassification" label="数据分级" ></el-checkbox>
                            </el-form-item>
                        </el-col>
                        <el-col :span="5">
                            <el-form-item>
                                <el-button type="primary" icon="el-icon-search" @click="search()" plain>检索</el-button>
                            </el-form-item>
                        </el-col>
                        <el-col :span="9" id="dateIn">
                            <el-upload  
                            name="fileName" 
                            ref="upload" 
                            action="http://localhost:8080/fileUpload" 
                            :auto-upload="false"
                            :on-success="uploadSuccess"
                            :on-error="uploadError"
                            :data="UploadData">
                                <!-- 再传入一个年份 -->
                                <el-col :span="5">
                                    <el-date-picker
                                        id="choiceYear"
                                        v-model="UploadData.YEAR"
                                        type="year"
                                        placeholder="请选择年份"
                                        format="yyyy 年"
                                        value-format="yyyy">
                                    </el-date-picker>
                                </el-col>
                                <el-button slot="trigger" type="primary" plain>选取文件</el-button>
                                <el-button type="success" @click="submitUpload()" plain>导入</el-button>
                            </el-upload>
                        </el-col>
                    </el-row>
                </el-form>
            </el-card>
            
            <!-- 表格 -->
            <el-card id="eduTable">
                <!-- data需要与后端传入的数据做操作 -->
                <!-- 
                    JavaScript的slice方法： 
                    语法：arrayObject.slice(start,end)
                    返回一个包含从start到end的arrayObject中的元素

                    tableData.slice((currentPage-1)*pagesize,currentPage*pagesize) <=> 每页返回10个tableData数组的子元素
                    第一页从0开始到10结束，依次返回。因为currentPage是不断在变化的  
                -->
                <el-table :data="tableData.slice((currentPage-1)*pagesize,currentPage*pagesize)" id="elTable">
                    <!-- 创建表格各列 -->
                    <el-table-column align="center" prop="year" label="年份"></el-table-column>
                    <el-table-column align="center" prop="id" label="序号"></el-table-column>
                    <el-table-column align="center" prop="city_name" label="城市"></el-table-column>
                    <el-table-column align="center" prop="province" label="省份"></el-table-column>
                    <el-table-column align="center" prop="city_exception" label="毕业期望"></el-table-column>
                    <el-table-column align="center" prop="city_sign" label="本届签约"></el-table-column>
                    <el-table-column align="center" prop="city_studentFrom" label="毕业生源"></el-table-column>
                    <el-table-column align="center" prop="city_visit" label="往届走访"></el-table-column>
                    <el-table-column align="center" prop="city_recency" label="回访率"></el-table-column>
                    <el-table-column align="center" prop="city_grading" label="城市级别"></el-table-column>
                    <el-table-column align="center" prop="city_score" label="综合评分"></el-table-column>
                </el-table>
                <!-- 分页 -->
                <div id="pagination">
                    <el-pagination background layout="prev, pager, next" :total="total" @current-change="current_change"></el-pagination>
                </div>
            </el-card>
        </div>
    </el-container>
</template>

<script>
// 引入表格

export default {
    data() {
        return{
            input: '',
            tableData: [],
            //当前页数
            currentPage: 1,
            //每页装的元组
            pagesize: 10,
            //元组总数目
            total: 0,

            //搜索部分
            searchBox: {
                //搜索输入框
                searchInput: '',
                //详细数据
                detailDigital: 0,
                //城市分级
                digitalClassification: 0,
            },
            //年份
            UploadData: {
                YEAR: '',
            },
            // _year: String(year),
        }
    },
    methods: {
        //传入分页的当前页，令定义的当前页=分页的当前页
        current_change(currentPage){
            this.currentPage = currentPage;
        },
        //进行搜索
        search(){
            console.log(this.searchBox.detailDigital);
            this.$ajax({
                method: "post",
                url: "http://localhost:8088/testBoot/selectEducationByKeyword",
                //keyword与后端代码中的局部变量相同
                data:{
                    keyword: this.searchBox.searchInput,
                },
                crossDomain: true,
                cache: false,
                // 加"transformRequest"属性对请求数据进行格式化
                transformRequest(obj){
                    var str = [];
                    for(var p in obj){
                        str.push(encodeURIComponent(p) + "=" + encodeURIComponent(obj[p]));
                    }
                    return str.join("&");
                },
            }).then(response => {
                this.tableData = response.data;
                this.total = response.data.length;
                console.log(response.data);
            },reject =>{
                console.log("失败"+reject);
            })
        },
        //导入
        uploadSuccess(response, file, fileList){
            console.log(response);
        },
        uploadError(response, file, fileList){
            console.log("失败");
        },
        submitUpload(){
            // console.log(this.$refs.upload);
            this.$refs.upload.submit();
            // console.log(this.year);
            // this.$ajax({
            //     method: "post",
            //     url: "http://localhost:8081/file/fileUpload",
            //     // dataType: ""
            //     data: {
            //         year: this.year,
            //     },
            //     crossDomain: true,
            //     //保证每次请求得到的数据都是最新的而不是缓存的数据
            //     cache: false,
            // }).then(resolve => {
            //     // this.tableData = resolve.data;
            //     // //将元组总数目设为数据的总数目
            //     // this.total = resolve.data.length;
            //     console.log(resolve.data);
            // }, reject => {
            //     console.log("失败",reject);
            // })
        }
    },
    created() {
        this.$ajax({
            method: "post",
            url: "http://localhost:8088/testBoot/listAll",
            dataType: "json",
            //跨域
            crossDomain: true,
            //保证每次请求得到的数据都是最新的而不是缓存的数据
            cache: false,
        }).then(resolve => {
            this.tableData = resolve.data;
            //将元组总数目设为数据的总数目
            this.total = resolve.data.length;
            console.log(resolve.data);
        }, reject => {
            console.log("失败",reject);
        })
    },
}
</script>
<style scoped>
    #eduHome {
        margin-top: 1%;        
    }
    #edu-boxCard {
        /* position: relative; */
        border-radius: 10px;
        text-align: center;
    }
    .el-form-item {
        margin-bottom: 0;
    }
    #TopTitle {
        position: relative;
        text-align: left;
        /* border: 1px solid red; */
        font-size: 19px;
        margin-bottom: 1.7%;
    }
    /* 使用选择器设置input的高度 */
    #CheckBtn {
        display: inline-block;
        /* border: 1px solid red; */
        /* position: absolute; */
    }
    .dataIO {
        height: 30px;
    }
    #eduTable{
        /* padding: 1%; */
        /* border: #cbcaca 1px solid; */
        color: #333;
        margin-top: 1%;
        border-radius: 10px;
    }
    /* #elTable {
        border: 1px solid red;
    }
    .el-table-column {
        width: 75px;
    } */
    #pagination {
        text-align: center;
    }
    #dateIn {
        /* border: 1px solid red; */
        text-align: right;
    }
    /* 修改年份框的大小 */
    .el-date-editor.el-input {
        width: 154px;
    }
</style>

