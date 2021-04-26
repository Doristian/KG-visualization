<template>
  <div class="home">
    <template>

      <h1 class='typewriter'>{{typewriter}}</h1>

      <el-carousel :interval="4000" type="card" height="300px">
        <el-carousel-item v-for="item in pics" :key="item.url">
          <img :src="item.url" style="width: 100%;height: 100%">
        </el-carousel-item>
      </el-carousel>
    </template>
  </div>
</template>

<script>
export default {
  data() {
    return {
      pics: [
        { url: require("../images/roll-picture/1.png") },
        { url: require("../images/roll-picture/2.png") },
        { url: require("../images/roll-picture/3.png") },
        { url: require("../images/roll-picture/4.png") },
        { url: require("../images/roll-picture/5.png") },
        { url: require("../images/roll-picture/6.png") }
      ],
      typewriter: '',
      i: 0,
      timer: 0,
      str: '军事知识图谱'
    };
  },
  mounted () {
    this.typeing()
  },
  methods: {
    typeing () {
      if (this.i <= this.str.length) {
        this.typewriter = this.str.slice(0, this.i++) + '_'
        this.timer = setTimeout(() => {
          this.typeing()
        }, 150)
      } else {
        this.typewriter = this.str.substr(0,this.str.length)
        clearTimeout(this.timer)
      }
    }
  }
}
</script>

<style>
.typewriter{
  font-family: 隶书;
  font-size: 60px;
  color: #FFFFFF;
}

.el-carousel__item h3 {
  color: #475669;
  font-size: 14px;
  opacity: 0.75;
  line-height: 200px;
  margin: 0;
}

.el-carousel__item:nth-child(2n) {
  background-color: #99a9bf;
}

.el-carousel__item:nth-child(2n+1) {
  background-color: #d3dce6;
}
</style>