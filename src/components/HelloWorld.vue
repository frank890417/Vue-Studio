<template lang="pug">
.page-editor
  .logo 
    h3 Vue Studio
    h5 @alpha v0.0.01
  .container-fluid
    .row
      .col-4.col-render(@click="getCompClick($event)")
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
                    .col-2 {{dt.name}}
                    .col-2 {{dt.type}}
                    .col-7 {{dt.ref}}
                    .col-1 
                      el-button(@click="datalist.splice(dtid,1)",
                        type="danger") -
                .row
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
                    el-button(@click="datalist.push(newDataItem);newDataItem={type: 'object'}", type="primary") Add
              

            .col-6
              h3.mt-5 DataSet
              ul
                li(v-for="(d,key) in datasets")
                  span {{key}}<br>
                  span {{d}}
        .col-12.mt-5
          h4 Code Generator
          el-input(v-model="generatedComponent", type="textarea", :rows="10")
</template>

<script>
import Vue from 'vue'
import axios from 'axios'
export default {
  name: 'HelloWorld',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App',
      template: "<div><div v-for='data in data1'> {{ data }} </div></div>",
      renderedComponent: null,
      datalist: [
        {name: "data1", type:"object" , ref:"data1"},
        // {name: "users", type:"api", ref: "https://jsonplaceholder.typicode.com/users"}
      ],
      sketchMode: true,
      newDataItem: {type: 'object'},
      datasets: {
        data1: [
          "hello",
          "i'm",
          "majer"
        ],
        data2: [
          "hrthhtrh",
          "dfgdfgerte"
        ]

      },
      render_env: null
    }
  },
  mounted(){
    setTimeout(()=>{
      this.renderComp()
    },100)
  },
  methods:{
    renderComp(){
      let _this = this
      let compData = {}
      let mountedFunctions = []
      this.datalist.forEach(d=>{
        if (d.type=="object"){
          compData[d.name]=_this.datasets[d.ref]
        }
        if (d.type=="api"){
          compData[d.name]=null
          mountedFunctions.push(function(){
            axios.get(d.ref).then((res)=>{
              this.$set(this,d.name,res.data)
            })
          })
        }
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
        template: this.template
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
    }
  },
  watch: {
    template(){
      this.renderComp()
    },
    datalist(){
      this.renderComp()
    }
  },
  computed: {
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
        padding: 10px
</style>
