<template>
  <div class="hello">
    <section class="content" style="background:#fff;padding-bottom:0">
      <div class="catalog">
        <div class="row row-wrap catelist">
          <div class="col col-50" v-for="(item,index) in CpList" :key="item.ApiKey">
            <div class="row ca">
              <div class="col">
                <h3>{{item.PcName}}</h3>
                <div>{{index}}</div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
export default {
  name: 'ComputerList',
  props: 'CpList',
  mounted(){
    console.log(this.props)
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
