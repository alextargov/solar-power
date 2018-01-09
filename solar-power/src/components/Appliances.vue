<template>
  <div id="appliances" class="container">
    <div class="progress row" style="height: 20px;">
      <div id="success" class="progress-bar bg-success" style="width:90%">
        90%
      </div>
    </div>
    <div class="row">
      <div id="addItem" class="col">
        <h4>Add appliance</h4>
        <form class="">
          <div class="form-group row">
            <label for="selected" class="col-sm-2 col-form-label">App.</label>
            <div class="col-sm-10">
              <select class="form-control " id="selected">
                <option value="TV">TV</option>
                <option value="Refrigerator">Refrigerator</option>
                <option value="PC">PC</option>
                <option value="Monitor">Monitor</option>
                <option value="Bulb">Bulb</option>
                <option value="Microwave">Microwave</option>
              </select>
            </div>
          </div>
          <div class="form-group row">
            <label for="watt" class="col-sm-2 col-form-label">Watts</label>
            <div class="col-sm-10">
              <input class="form-control"  id="watt" type="text" name="watt" required placeholder="50" min="0"
                     maxlength="3" pattern="[1-9][0-9]?[0-9]?" title="Невалидно число.">
            </div>
          </div>
          <div class="form-group row">
            <input type="submit" class="btn btn-primary"  @click="addItem" onclick="return false" value="Add">
          </div>
        </form>
      </div>
      <div class="col-5">
        <ul class="list-group">
          <li class="list-group-item" :id="index"  v-for="(item, index) in appliances"  > {{ item.application  }} | {{ item.watts }}W
            <label @click="item.isOn = !item.isOn" class="switch" >
              <input class="inputs" @click="toggle(index)" type="checkbox" checked >
              <span  class="slider round"></span>
            </label>
            <button type="button" @click="removeItem(index)" class="btn btn-danger">Delete</button>
          </li>
        </ul>
      </div>
      <div id="items" class="col">
        <div id="consumption"></div>
        <div id="generated"></div>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'Appliances',
    mounted: function(){
      this.amps();
    },
    props: ['divider'],
    methods: {
      removeItem(index) {

        this.appliances.splice(index, 1);
      },
      toggle(index) {
        this.appliances[index].isOn = !this.appliances[index].isOn;
        document.getElementsByClassName("inputs")[index].setAttribute("checked", "checked");
        document.getElementsByClassName("inputs")[index].setAttribute("type", "checkbox");
      },
      addItem() {
          let getItem = document.getElementById("selected");
          let getWatts = document.getElementById("watt");

          if(getWatts.checkValidity()) {
            let selectedItem = getItem.options[getItem.selectedIndex].value;
            let watts = getWatts.value;
            let obj = {};

            obj["application"] = selectedItem;
            obj["watts"] = watts;
            obj["isOn"] = true;
            obj["remove"] = false;
            this.appliances.push(obj);
          }
      },
      totalConsumption: function() {
        let consumption = 0;
        let rand;
        let randPosNeg;
        for(let i = 0; i < this.appliances.length; i+= 1) {

          randPosNeg = Math.random() < 0.5 ? -1 : 1;
          if(this.appliances[i].isOn) {
            if(this.appliances[i].watts >= 100) {
              rand = (Math.random()*3 + 1);
            }
            else if(this.appliances[i].watts >= 50 && this.appliances[i].watts <= 100 ) {
              rand = (Math.random()*2 + 1);
            }
            else if(this.appliances[i].watts >= 20 && this.appliances[i].watts < 50) {
              rand = (Math.random() + 1);
            }
            else if(this.appliances[i].watts > 0 && this.appliances[i].watts < 20) {
              rand = (Math.random());
            }

            rand = Math.round( rand * 1e3 ) / 1e3;
            rand *= randPosNeg;

            consumption += 1*this.appliances[i].watts + rand;
          }
        }
        // throw the Total consumption in the DOM!!!
        document.getElementById("consumption").innerHTML = "Total consumption: " + consumption.toFixed(2) + "W";
        return consumption;
      },
      amps: function () {
        let date = new Date();
        let getHour;
        let sunrise = 6;
        let sunset = 19;
        let result = 0;
        let getSuccessEl = document.getElementById("success");
        date.setHours(0);
        date.setMinutes(0);
        const nominalBatteryAmps = 200;
        let voltage = 12;
        const watts = function (voltage) {
          let success = getSuccessEl.innerHTML.split("%")[0];
          return +((success*1)/100*(nominalBatteryAmps*voltage*1));
        };
        const setWatts = function (watts, voltage) {
          let newWatts = (watts*100/(nominalBatteryAmps*voltage)).toFixed(2);
          //console.log(newWatts);
          if(newWatts > 100) {
            getSuccessEl.innerHTML = "100.00%";
            getSuccessEl.style.width = 100 + "%";
          }
          else {
            getSuccessEl.innerHTML = newWatts.toString() + "%";
            getSuccessEl.style.width = newWatts + "%";
          }
        };

        const getWatts = function () {
          return +document.getElementById("success").innerHTML.split("%")[0];
        };

        let consume;
        self = this;
        setInterval(function () {
          consume = self.totalConsumption();
          getHour = date.getHours();

          const percentage = +document.getElementById("success").innerHTML.split("%")[0];
//          if(consume >= 0 && consume < 50) {
//            voltage = 12;
//          } else if (consume >= 50 && consume < 100) {
//            voltage = 11.8;
//          } else if (consume >= 100 && consume < 300) {
//            voltage = 11.6;
//          } else if (consume >= 300 && consume < 500) {
//            voltage = 11.4;
//          } else if (consume >= 500 && consume < 1000) {
//            voltage = 11.2;
//          } else if (consume >= 1000 && consume < 1500) {
//            voltage = 11;
//          } else if (consume >= 1500) {
//            voltage = 10.4;
//          }
          if(percentage >= 95) {
            voltage = 12.4;
          } else if(percentage >= 90 && percentage < 95) {
            voltage = 12.2;
          } else if(percentage >= 80 && percentage < 90) {
            voltage = 12;
          } else if (percentage >= 70 && percentage < 80) {
            voltage = 11.7;
          } else if (percentage >= 60 && percentage < 70) {
            voltage = 11.4;
          } else if (percentage >= 50 && percentage < 60) {
            voltage = 11.1;
          } else if (percentage >= 40 && percentage < 50) {
            voltage = 10.7;
          } else if (percentage >= 30 && percentage < 40) {
            voltage = 10.3;
          } else if (percentage >= 20 && percentage < 30) {
            voltage = 10;
          } else if (percentage >= 10 && percentage < 20) {
            voltage = 10.6;
          } else if (percentage >= 1 && percentage < 10) {
            voltage = 10.1;
          }
          result = watts(voltage);
          if(+getWatts() >= 0 && +getWatts() < 5) {
            document.getElementById("generated").innerText = "Battery life: " +(result/voltage).toFixed(2)+"Ah Voltage 9V";

            setWatts(result, voltage);
            document.getElementById("success").style.width =  getWatts()+ "%";
            document.getElementById("success").innerHTML = getWatts().toFixed(2) + "%";

            for (let i = 0; i < self.appliances.length; i += 1) {
              self.appliances[i].isOn = false;
              document.getElementsByClassName("inputs")[i].removeAttribute("checked");
              document.getElementsByClassName("inputs")[i].removeAttribute("type");
            }
          } else if(+getWatts() < 0) {
            document.getElementById("generated").innerText = "Battery life: 0Ah Voltage 9V";

            setWatts(0, voltage);
            document.getElementById("success").style.width =  "0%";
            document.getElementById("success").innerHTML = "0.00%";

            for (let i = 0; i < self.appliances.length; i += 1) {
              self.appliances[i].isOn = false;
              document.getElementsByClassName("inputs")[i].removeAttribute("checked");
              document.getElementsByClassName("inputs")[i].removeAttribute("type");
            }
          } else {
            result -= consume / 6;
            setWatts(result, voltage);
            //console.log(result);
            document.getElementById("generated").innerText = "Battery life: " +
              (result/voltage).toFixed(2) + "Ah Voltage " + voltage + "V";
          }
          if (getHour >= sunrise && getHour <= sunset) {

            if ((getHour >= sunrise && getHour < sunrise + 1) ||
              (getHour >= sunset - 1 && getHour <= sunset )) {
              result += 500 / ((sunset - sunrise)) / (6 * self.divider); // generated for 10minutes
              result = parseInt(result);
              setWatts(result, voltage);
            }
            else if ((getHour >= sunrise + 1 && getHour < sunrise + 2) ||
              (getHour >= sunset - 2 && getHour <= sunset - 1)) {
              result += 600 / ((sunset - sunrise)) / (6 * self.divider); // generated for 10minutes
              result = parseInt(result);
              setWatts(result, voltage);
            }
            else if ((getHour >= sunrise + 2 && getHour < sunrise + 3) ||
              (getHour >= sunset - 3 && getHour <= sunset - 2)) {
              result += 700 / ((sunset - sunrise)) / (6 * self.divider); // generated for 10minutes
              result = parseInt(result);
              setWatts(result, voltage);
            }
            else if ((getHour >= sunrise + 3 && getHour < sunrise + 4) ||
              (getHour >= sunset - 4 && getHour <= sunset - 3)) {
              result += 800 / ((sunset - sunrise)) / (6 * self.divider); // generated for 10minutes
              result = parseInt(result);
              setWatts(result, voltage);
            }
            else if ((getHour >= sunrise + 4 && getHour < sunrise + 5) ||
              (getHour >= sunset - 5 && getHour < sunset - 4)) {
              result += 900 / ((sunset - sunrise)) / (6 * self.divider); // generated for 10minutes
              result = parseInt(result);
              setWatts(result, voltage);
            }
            else {
              if (getWatts() < 100) {
                result += 1000 / ((sunset - sunrise)) / 6; // generated for 10minutes
                result = parseInt(result);
                setWatts(result, voltage);
              }
            }
          }

          date.setMinutes(date.getMinutes() + 10); //accelerate the time
        }, 1000);
      },
    },
    data () {
      return {
        appliances: [],
      }
    }
  }
</script>

<style scoped>

#appliances {
  width: 960px;
  height:auto;
}


#items ul li{
  width: 200px;
  margin: 5px 0;
}

ul {
  padding: 0;
  margin: 0;
  text-align: left;
}

.switch {
  margin-top: 5px !important;
  position: relative;
  display: inline-block;
  width: 42px;
  height: 24px;
}

/* Hide default HTML checkbox */
.switch input {display:none;}
label.switch {margin: auto 0 auto 10px; float: right}
/* The slider */
.slider {
  position: absolute;
  cursor: pointer;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: #ccc;
  -webkit-transition: .4s;
  transition: .4s;
}

.slider:before {
  position: absolute;
  content: "";
  height: 16px;
  width: 16px;
  left: 5px;
  bottom: 4px;
  background-color: white;
  -webkit-transition: .4s;
  transition: .4s;
}

input:checked + .slider {
  background-color: #2196F3;
}

input:focus + .slider {
  box-shadow: 0 0 1px #2196F3;
}

input:checked + .slider:before {
  -webkit-transform: translateX(16px);
  -ms-transform: translateX(16px);
  transform: translateX(16px);
}

/* Rounded sliders */
.slider.round {
  border-radius: 34px;
}

.slider.round:before {
  border-radius: 50%;
}
.btn-danger {
  float:right;
}
</style>
