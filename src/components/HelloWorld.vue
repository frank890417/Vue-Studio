<template lang="pug">
.page-editor
  .logo 
    h3 Vue Studio
    h5 @alpha v0.0.01
  .container-fluid
    .row
      .col-4.col-render(@click="getCompClick($event)", @drop="dropElement($event)", @dragover="dragOver($event)")
        .rendered(v-if="renderedComponent" is="renderedComponent",
                  :class="{sketchMode: sketchMode}")
        .bottom
          span.mr-2 編輯模式：
          el-switch(v-model="sketchMode")
      .col-8.text-left
        .data
          h3.mt-5 Template:
          el-input(v-model="template",
                  type="textarea",
                  :rows="5")
          .row
            .col-6
              h3.mt-5 Datas
              //- div
                //- label Type
              .list-group
                .list-group-item(v-for="(dt,dtid) in datalist")
                  .row
                    .col-4
                      .col-12(draggable="true" @dragstart="dragDataSymbol($event,dt)")
                        el-input(v-model="dt.name",
                                placeholder='名字')
                      .col-12
                        .mr-2(v-for="key in getKeys(dt)", draggable="true" @dragstart="dragDataSymbol($event,dt,key)") @{{key}}
                    .col-4
                      el-select(v-model="dt.type")
                        el-option(label="Object",value="object")
                        el-option(label="API",value="api")
                    .col-3
                      el-select(v-model="dt.ref", v-if="dt.type=='object'")
                        el-option(v-for="(d,key) in datasets", :label="key" :value="key")
                      el-input(v-else, v-model="dt.ref", @change="getDataCache")
                    .col-1 
                      el-button(@click="datalist.splice(dtid,1)",
                        type="danger") -
                //.row
                  .col-4
                    el-input(v-model="newDataItem.name",
                              placeholder='名字')
                  .col-4
                    el-select(v-model="newDataItem.type")
                      el-option(label="Object",value="object")
                      el-option(label="API",value="api")
                  .col-4
                    el-select(v-model="newDataItem.ref", v-if="newDataItem.type=='object'")
                      el-option(v-for="(d,key) in datasets", :label="key" :value="key")
                    el-input(v-else, v-model="newDataItem.ref")

                .col-12.mt-3
                  el-button(@click="datalist.push({type: 'object'})", type="primary") Add
              

            .col-6
              h3.mt-5 DataSet
              ul
                li(v-for="(d,key) in datasets")
                  span {{key}}<br>
                  span {{d}}
        //.col-12
          pre(v-text="hashedTemplate")
        .col-12.mt-5
          h4 Code Generator
          el-input(v-model="generatedComponent", type="textarea", :rows="10")
</template>

<script>
import Vue from 'vue'
import axios from 'axios'
import $ from 'jquery'
export default {
  name: 'HelloWorld',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App',
      template: `
        <div class='list-group text-left'>
          <div v-for='user in users' class='list-group-item'> 
            <span>{{ user.name }}</span>
            <span> .</span>
            <button class='btn btn-danger float-right'> - </button> 
          </div>
        </div>`,
      renderedComponent: null,
      datalist: [
        {name: "posts", type:"object" , ref:"postlist"},
        {name: "title", type:"object" , ref:"title"},
        {name: "users", type:"api", ref: "https://jsonplaceholder.typicode.com/users"}
      ],
      sketchMode: true,
      newDataItem: {type: 'object'},
      datasets: {
        title: "Hello world!",  
        postlist: [
          {title: "hello world", author: "Frank", date: "2018-5-2"},
          {title: "bakaka museum", author: "John", date: "2015-3-21"},
          {title: "vue studio beta", author: "Mark", date: "2017-4-4"}
        ],
      // {name: "users", type:"api", ref: "https://jsonplaceholder.typicode.com/users"}
        // data2: [
        //   "hrthhtrh",
        //   "dfgdfgerte"
        // ],

      },
      render_env: null
    }
  },
  mounted(){
    setTimeout(()=>{
      this.renderComp()
      this.getDataCache()
    },100)
  },
  methods:{
    dragDataSymbol(evt,dataSymbol,key){
      evt.dataTransfer.setData("dataSymbol",JSON.stringify(dataSymbol) );
      evt.dataTransfer.setData("key",key?key:"");
    },
    dragOver(evt){
      evt.preventDefault();
      console.log(evt)

      //顯示目標範圍
      let hash = evt.target.getAttribute('data-editor-hash')
      $("[data-editor-hash").removeClass("drag-over-active")
      $("[data-editor-hash='"+hash+"']").addClass("drag-over-active")

    },
    dropElement(evt){
      console.log(evt.dataTransfer.getData("dataSymbol"))

      let nowTemplate =  $(this.hashedTemplate)
      var dataSymbol = JSON.parse(evt.dataTransfer.getData("dataSymbol"));
      var key = evt.dataTransfer.getData("key");
      let hash = evt.target.getAttribute('data-editor-hash')
      let useName = key?dataSymbol.name.substr(0,dataSymbol.name.length-1):dataSymbol.name
      // console.log("[data-editor-hash='"+hash+"']",nowTemplate.find("[data-editor-hash='"+hash+"']"))
      let element = nowTemplate.find("[data-editor-hash='"+hash+"']")
      element.text(`{{ ${ useName + (key?("."+key):"") } }}`)
      if (key){
        element.parent().attr('v-for',useName + " in "+dataSymbol.name)
      }
      // console.log(nowTemplate)
      
      this.template = (nowTemplate.prop('outerHTML')).replace(/data-editor-hash=\".*?\"/g,"")
      $("[data-editor-hash").removeClass("drag-over-active")
    },
    renderComp(){
      let _this = this
      let compData = {}
      let mountedFunctions = []
      this.datalist.forEach(d=>{
        if (d.type=="object"){
          compData[d.name]=_this.datasets[d.ref]
        }
        if (d.type=="api"){
          compData[d.name]=d.cache
        }
        //再mounted時候下載api資料
        // if (d.type=="api"){
        //   compData[d.name]=null
        //   mountedFunctions.push(function(){
        //     axios.get(d.ref).then((res)=>{
        //       this.$set(this,d.name,res.data)
        //     })
        //   })
        // }
      })
      console.log(compData)
      let cmp = {
        mounted(){
          mountedFunctions.forEach(func=>{
            func.call(this)
          })
        },
        data(){
          return compData
        },
        template: this.hashedTemplate
      }
      this.renderedComponent = Vue.component("renderedComponent",cmp)
      // this.render_env=new Vue({
      //   el: "#component_field",
      //   components: {
      //     renderComponent: cmp
      //   }
      // })
    },
    getCompClick(evt){
      console.log(evt)
    },
    getKeys(dataSymbol){
      let refData = dataSymbol.cache || this.datasets[dataSymbol.ref]
      if (typeof refData =="object"){
        return Object.keys( (refData[0]) || {})
      }
    },
    getDataCache(){
      let _this = this
      this.datalist.forEach(dt=>{
        if (dt.type=="api"){
          axios.get(dt.ref).then((res)=>{
            _this.$set(dt,"cache",res.data)
          })
        }
      })
    }
  },
  watch: {
    template(){
      this.renderComp()
    },
    datalist(){
      this.renderComp()
      this.getDataCache()
    },
    // "datalist.ref": function(){
      // this.getDataCache()
    // }
  },
  computed: {
    //add hash for editor
    hashedTemplate(){
      let hashed = $(this.template)
      hashed.find("h1,h2,h3,h4,h5,h6,li,p,span,a").each((id,obj)=>{
        // console.log(obj)
        $(obj).attr("data-editor-hash",parseInt(Math.random()*1000000000))
      })
      return hashed.prop('outerHTML')
    },
    generatedComponent(){
      return `<template>\n${this.template}\n</template>
      \n<script>\n<\/script>
      \n<style>\n</style>`
    }

  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="sass">
.page-editor
  .logo
    position: fixed
    left: 20px
    top: 20px
    z-index: 1
  .container-fluid > .row
    height: 100vh
  .col-render
    background-color: #eee
    display: flex
    justify-content: center
    align-items: center
    .bottom
      position: absolute
      bottom: 0
      padding: 10px 10px
      text-align-last: left
      background-color: #ccc
      width: 100%
      left: 0
  .rendered
    border: solid 1px #aaa
    &.sketchMode
      box-shadow: 0px 0px 20px rgba(0,0,0,0.1)
      *,&
        border: solid 1px #aaa
        padding: 5px
    [data-editor-hash]
      // border: solid 1px #999
      // transition: 0.5s
      &.drag-over-active
        border: 2px solid red
</style>
