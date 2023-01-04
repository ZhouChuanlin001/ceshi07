<template>
  <div>
    <div class="ad" style="width: 100%;height: 245px">

    </div>
    <!--魅力空间-->
    <div>
      <div style="font-size: 14px;margin-top: 10px;padding: 10px;border-bottom: 2px solid black;margin-left: 20px;margin-right: 20px">
        <h4>魅力空间</h4>
      </div>
      <div class="content">
        <div class="content-item" v-for="item in imgData">
          <div class="item-img">
<!--            <div class="img" :style="{background:'url('+require(`${item.url}`)+')'}"></div>-->
            <img class="img" :src="item.url" alt="">
          </div>
          <div style="line-height: 20px;padding-top: 10px;box-sizing:border-box;text-align: center;width: 130px;height:55px;background-color: black;color: rgb(233,233,233)">
            <h5 style="letter-spacing: 1px">{{item.title}}</h5>
            <p style="margin-left: -15px;transform:scale(0.6);font-size: 10px;width: 160px;">{{item.desc}}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
    export default {
        name: "productView",
      data(){
          return{
            imgData:[]
          }
      },
      mounted() {
          this.imgData = [
            {
              title:'雅致轻奢',
              url:require('../assets/img/eb747e0406cc39b1afcc0f180a5f67f6.jpg'),
              desc:'ELEGANT LIGHT LUXURY'
            },
            {
              title:'高贵典雅',
              url:require('../assets/img/33c1861295a4f30a6831b9c307d63650.jpg'),
              desc:'NOBLE AND ELEGANT'
            },
            {
              title:'华丽恢弘',
              url:require('../assets/img/9688856c825fcd62e00be867795952a5.jpg'),
              desc:'MAGNIFICENT'
            },
            {
              title:'白润底纹',
              url:require('../assets/img/9378b8e0e9c57c6e8c9bb59a89d52406.jpg'),
              desc:'WHITENING SHADING'
            },
            {
              title:'静泊深远',
              url:require('../assets/img/c6bfb8c606f23bd269af23e7c834ccdf.jpg'),
              desc:'QUIET AND FAR-REACHING'
            },
            {
              title:'高端大气',
              url:require('../assets/img/eb747e0406cc39b1afcc0f180a5f67f6.jpg'),
              desc:'HIGH END ATMOSPHERE'
            },
            {
              title:'自然美妙',
              url:require('../assets/img/c41f433a4e034bbfdf58ed9b009bc1bf.jpg'),
              desc:'NATURAL BEAUTY'
            },
            {
              title:'剔透光泽',
              url:require('../assets/img/d6e77f15c19e7d5bbba37343fe2d91fc.jpg'),
              desc:'CLEAR LUSTER'
            },
            {
              title:'稳重清爽',
              url:require('../assets/img/d21f1b594ced57c8511a2ef27e5e8c24.jpg'),
              desc:'STEADY AND REFRESHING'
            },
            {
              title:'时尚简约',
              url:require('../assets/img/c53b913454b655a89542d4f8b2f2647d.jpg'),
              desc:'FASHION SIMPLICITY'
            },
          ]
      }
    }
</script>

<style scoped>
.ad{
  background-image: url("../assets/img/lrha.jpg");
  background-size: cover;
  background-repeat: no-repeat;
  background-position: center;
}
.content{
  display: flex;
  justify-content: center;
  flex-direction: row;
  flex-wrap: wrap;
  margin-bottom: 30px;
}
  .content-item{
    width: 130px;
    height: 205px;
    margin-top: 30px;
    margin-left: 6px;
    margin-right: 6px;
  }
  .item-img{
    width: 130px;
    height: 150px;
    overflow: hidden;
  }
  .img{
    width: 130px;
    height: 150px;
    background-image: url("../assets/img/3-4.jpg");
    background-position: center;
    background-repeat: no-repeat;
    background-size: cover;
    transition: linear 0.3s;
  }
  .img:hover{
    transform: scale(1.2);
  }
</style>
