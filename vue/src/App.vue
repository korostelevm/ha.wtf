<template>

  <div class='main'>
  <div v-for="(r) in reqs" :key="r.id">
    
    <b-alert class='req' variant="success" show >
        <b-row 
        v-on:click="collapse_show(r)"
        >
      <b-col cols="1">
        <b-badge variant='success'>{{r.req.method}}</b-badge>
      </b-col>
            <b-col cols="2">
              <span>{{r.req.path}}</span>
            </b-col>
            <b-col>
              <span>{{r.req.source ? r.req.source.asn.name : r.req.sourceIp}}</span>
            </b-col>
            
            <b-col>
              <span>{{r.req.userAgent}}</span>
            </b-col>
            
            <b-col class='text-right'>
              <div>{{r.time_ago}}</div>
              <div>{{r.ts}}</div>
            </b-col>
        </b-row>
        <b-row>
      <b-collapse :id="r.id" class="mt-2" accordion="my-accordion" >
        <b-card >
          <div class="text-center" v-if="loading==r.id">
            <b-spinner label="Spinning"></b-spinner>
          </div>
          <pre class='payload'>{{req_body}}</pre>
        </b-card>
      </b-collapse>
      </b-row>

    </b-alert>

    <!-- {{r.id}} -->
  </div>
    
  </div>

</template>
 
<script>
var moment = require('moment')
export default {
    name: 'microfrontend',
    data() {
      return {
        error: false,
        loading: false,
        reqs:[],
        req_body:false,
        active: null
      }
    },
    mounted: function() {
      this.stub()
    },
    created: function() {
    },
    methods: {
      get_req:function(r){
        return new Promise((resolve,reject)=>{
          this.loading = r.id
          fetch(this.$api + '/integration/'+r.id, {
              method: 'GET',
              headers: {
                'Content-Type': 'application/json',
                'Authorization': this.get_auth_header()
              },
              // body: JSON.stringify(d),
            })
            .then(res => res.json()) 
            .then(data => {
              this.loading = false;
              if("req_body" in data && data.req.headers['content-type'] == 'application/json'){
                try{
                  data.req_body =  JSON.parse(data.req_body)
                }catch(e){
                  console.warn(e)
                }
              }
              resolve(data)
            })
          })
      },
      collapse_show: async function(e){
        this.$root.$emit('bv::toggle::collapse',e.id)
        console.log(e.id)
        if(this.active != e.id){
          this.req_body = null
          this.req_body = await this.get_req(e)
        }
        this.active = e.id
      },
       stub: function(d) {
        return new Promise((resolve,reject)=>{
          fetch(this.$api + '/integrations', {
              method: 'GET',
              headers: {
                'Content-Type': 'application/json',
                'Authorization': this.get_auth_header()
              },
              // body: JSON.stringify(d),
            })
            .then(res => res.json()) 
            .then(data => {
              this.reqs=data.map(r=>{
                var ts  = moment.utc(r.d + ' ' + r.t, 'YYYY-MM-DD HH:mm:ss:SSSS' )
                r.time_ago = ts.fromNow()
                r.ts = ts.format('LLLL')
                return r
              })
              resolve(data)
            }).catch(e => {
              this.error = e; console.error('exception:', e);
            })
          })
      },
      },
  }
</script>

<style scoped>
.main {
  margin:20px  auto ;
  width:1000px;
}
.req{
  cursor: pointer;
}
.payload{
    white-space: pre-wrap;
    width: -webkit-fill-available;
    width: 945px;
}

</style>
