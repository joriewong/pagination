<template>
    <table class="table table-striped table-hover">
      <thead>
      <tr>
        <td v-for="(th,index) in info[0]">{{index}}</td>
      </tr>
      </thead>
      <tbody>
      <tr v-for="td in showInfo">
        <td>{{td.name}}</td>
        <td>{{td.id}}</td>
      </tr>
      </tbody>
      <tfoot>
      <tr>
        <td colspan="2">
            <button class="btn btn-default" @click="pageShow(1)"><a>首页</a></button>
            <button :class="{disabled:isPrev}" class="btn btn-default" @click="shift('prev')"><a>上一页</a></button>
            <div class="show-part">
              <ul class="pagination">
                <li v-for="page in pageNumbers" :class="{active:page.isActive}">
                  <a @click="pageShow(page.index)">{{page.index}}</a>
                </li>
              </ul>
            </div>
            <button class="btn btn-default">
              <a>
                <input placeholder="页码" v-model="inputPage" class="inputPage" type="text" @keyup="pageShow(inputPage)">
              </a>
            </button>
            <button class="btn btn-default" :class="{disabled:isNext}" @click="shift('next')"><a>下一页</a></button>
            <button class="btn btn-default" @click="pageShow(len)"><a>尾页</a></button>
        </td>
      </tr>
      </tfoot>
    </table>
</template>
<script>
import $ from 'jquery'
export default {
    data () {
        return {
          perPage: 10,//每页显示
          info: [],//所有数据
          pageNumbers: [],//页码
          len: 0,//总页码
          cutInfo: [],//数据切分结果
          showInfo: [],//第一次展示的数据
          page: 1,//当前页
          isPrev: true,//能否前一页
          isNext: false,//能否后一页
          inputPage: '',//输入页码
          marginLeft: 0,//显示隐藏
        }
    },
    mounted () {
      this.$http.get('http://rap.taobao.org/mockjsdata/22379/api/data').then(function (res) {
        this.info = res.data.data;
        this.pageNumber(this.perPage, this.info);
        this.cutData(this.perPage, this.info);
        this.showInfo = this.cutInfo[0];
        if(this.len>10) this.isOver = true;
      });
    },
    methods: {
      //格式化数据
      cutData(count,result) {
        for(var i = 0;i < this.len;i++){
          this.cutInfo.push(result.slice(i, i+count));
        }
      },
      //页码
      pageNumber (count, result) {
        if(result.length <= count){
            this.len = 1;
        }else {
            this.len = Math.ceil(result.length/count);
        }
        for(var i = 1;i <= this.len;i++){
            this.pageNumbers.push({index:i,isActive:false});
        }
        this.pageNumbers[0].isActive = true;
      },
      //页码点击展示
      pageShow (page) {
        if(!page||isNaN(page)||page<1||page>this.len) return;
        //上一页禁用
        this.isPrev = page==1?true:false;
        //下一页禁用
        this.isNext = page==this.len?true:false;
        this.page = page;
        //激活
        this.toggleClass();
        this.showInfo = this.cutInfo[page-1];
        if(this.page >3 && this.page < this.len-2) {
          //页码居中
          this.marginLeft = -(this.page-3)*41;
          $('ul.pagination').css('marginLeft', this.marginLeft+'px');
        }else if(this.page <= 3){
          $('ul.pagination').css('marginLeft', 0);
        }else {
          $('ul.pagination').css('marginLeft', -(this.len-5)*41+'px');
        }
      },
      //上一页/下一页切换
      shift(str) {
          if(str == 'prev'){
              if(this.page > 1){
                this.page--;
                this.pageShow(this.page);
              }else {
                  this.isPrev = true;
              }
              this.showOrHide(str);
          }else {
              if(this.page < this.len){
                this.page++;
                this.pageShow(this.page);
              }else {
                  this.isNext = true;
              }
              this.showOrHide(str);
          }
      },
      //class切换
      toggleClass () {
          var that = this;
          this.pageNumbers.map(function (value, index, array) {
            value.isActive = (index+1) == that.page?true:false;
            return array;
          })
      },
      //页码显示隐藏
      showOrHide (str) {
        if(str == 'prev'){
          if(this.page >3 && this.page < this.len-2){
            $('ul.pagination').css('marginLeft', this.marginLeft+'px');
            this.marginLeft += 41;
          }else if(this.page <= 3){
            $('ul.pagination').css('marginLeft', 0);
          }
        }else {
          if(this.page >3 && this.page < this.len-2){
            $('ul.pagination').css('marginLeft', this.marginLeft+'px');
            this.marginLeft -= 41;
          } else if(this.page >= this.len-2){
            $('ul.pagination').css('marginLeft', -(this.len-5)*41+'px');
          }
        }
      }
    }
}
</script>
<style scoped>
  a {
    cursor: pointer;
    text-decoration: none;
  }
  input.inputPage {
    border: 0;
    outline: 0;
    padding-top: 0;
    padding-bottom: 0;
    width: 34px;
  }
  button.btn {
    outline: 0;
  }
  ul.pagination {
    width: 1280px;
    margin: 0;
    vertical-align: top;
  }
  div.show-part {
    width: 205px;
    overflow: hidden;
    display: inline-block;
    vertical-align: top;
  }
  ul.pagination li a{
    width: 42px;
    text-align: center;
  }
</style>
