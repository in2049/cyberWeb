<template>

  <div class="content">
    <router-view/>
  </div>
</template>

<script>
export default {
  name: 'Battery',
  created() {
  },
  data(){
    return{
      tabPosition:'left'
    }
  }
}

</script>

<style lang="scss" scoped>
.content {
  flex: 1;
  background-color: #20a0ff;

}
.el-tabs{
  height: 100%;
}
</style>
