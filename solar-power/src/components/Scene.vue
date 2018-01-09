<template>
  <div id="scene">
    <div id="header">
      <img style="opacity: 0;" id="sun" src="../assets/giphy.gif">
      <img id="cloud" src="../assets/cloud.png">
      <div id="time">
        <h2 id="times">{{ time }}</h2>
      </div>
    </div>
    <div class="scene">
      <img class="panel" src="../assets/scene.png">
    </div>
    <div class="notifications">
      <h5 v-if="this.notification.isTrue">{{ notification.message }} </h5>
      <h5 v-if="this.notification2.isTrue">{{ notification2.message }} </h5>
      <h5 v-if="!this.notification2.isTrue && !this.notification.isTrue">No clouds!</h5>
    </div>
  </div>

</template>

<script>
  export default {
    name: 'Scene',
    mounted: function() {
      this.getSunCoordinates();
      this.triggerCloud();
      this.compareCoordinates();
    },
    data() {
      return {
        notification: {
          isTrue: false,
          message: 'Cloud incoming!'
        },
        notification2: {
          isTrue: false,
          message: 'Cloud going away!'
        }
      }
    },
    methods: {
      triggerCloud() {
        let self = this;
        let flag = false;
        let interval = setInterval(function () {
          let rand = Math.random();
          /* Triggers the cloud only if  (rand >= 0.8)*/

          if (rand >= 0.95) {
            clearInterval(interval);
            self.notification.isTrue = true;
            let el = document.getElementById("cloud");
            let pos = -25;
            let id = setInterval(frame, 5);
            function frame() {
              if (pos >= 100) {
                clearInterval(id);
                self.notification2.isTrue = false;
                self.triggerCloud();
              } else {
                pos += 0.01;
                if(pos > 50) {
                  self.notification.isTrue = false;
                  self.notification2.isTrue = true;
                }
                el.style.left = pos + '%';
              }
            }
          }
        }, 1000);
      },
      getScreenWidth() {
        let w = window,
          d = document,
          e = d.documentElement,
          g = d.getElementsByTagName('body')[0],
          x = w.innerWidth || e.clientWidth || g.clientWidth,
          y = w.innerHeight|| e.clientHeight|| g.clientHeight;
        return [x, y];
      },
      getSunCoordinates() {
          const result = [
            this.getOffset(document.getElementById("sun")).top,
            this.getOffset(document.getElementById("sun")).left
            ];
          return result;
      },
      getCloudCoordinates() {
        let result;
        result = [
          this.getOffset(document.getElementById("cloud")).top,
          this.getOffset(document.getElementById("cloud")).left
        ];
        return result;
      },
      compareCoordinates() {
        let sun;
        let cloud;
        let self = this;
        setInterval(function () {
          //self.triggerCloud();
          sun = self.getSunCoordinates();
          cloud = self.getCloudCoordinates();
          if(cloud[1]+400 >= sun[1] && cloud[1] <= sun[1]+50) {
            //console.log(cloud);
            //console.log(sun);
            self.$emit('generatedDivider', 2);
            //this.$emit('generatedDivider', 2);
          }
          else {
            self.$emit('generatedDivider', 1);
          }
         /* console.log(cloud);
          console.log(sun);*/
        }, 1000)
      },
      getOffset(el) {
        el = el.getBoundingClientRect();
        return {
          left: el.left + window.scrollX,
          top: el.top + window.scrollY
        }
      }
    },
    computed: {
      time: function () {

        const sunHours = {
          sunriseStart: 5,
          sunriseEnd: 7,
          sunsetStart: 18,
          sunsetEnd: 20
        };
        let date = new Date();
        date.setHours(0);
        date.setMinutes(0);
        let opacity = 0;
        let getHour;

        setInterval(function () {
          getHour = date.getHours();

          //normalizing the opacity
          if(opacity > 1) {
            document.getElementById("sun").style.opacity = "1";
          }
          if(opacity < 0) {
            document.getElementById("sun").style.opacity = "0";
          }

          // fading the sun in- simulation day
          if(getHour >= sunHours.sunriseStart && getHour <= sunHours.sunriseEnd) {
            opacity+=.1;
            document.getElementById("sun").style.opacity = opacity;
          }
          // fading the sun out- simulation night
          if(getHour >= sunHours.sunsetStart && getHour <= sunHours.sunsetEnd) {
            opacity-=.1;
            document.getElementById("sun").style.opacity = opacity;
          }

          //accelerate the time
          date.setMinutes(date.getMinutes() + 10);
          if(date.getMinutes() === 0) {
            document.getElementById("times").innerHTML = "Time " + date.getHours() + ":" + date.getMinutes() +'0' ;
          } else {
            document.getElementById("times").innerHTML = "Time " + date.getHours() + ":" + date.getMinutes() ;
          }

        },1000);
      }
    }
  }
</script>

<style scoped>
  #header {
    background-color: #529bff;
  }
  .scene {
    margin-top: 10px;
  }
  .notifications {
    height: 40px;
  }
  #cloud {
    width: 30%;
    height: 30%;
    position: absolute;
    right: 100%;
  }
  @keyframes example {
    from {transform: translate(0px, 0px)}
    to {transform: translate(1500px, 0px)}
  }
  #sun {
    height: 20%;
    width: 20%;
    position: relative;
  }
  h1, h2 {
    font-weight: normal;
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
