<template>
  <div id="appliances" class="container">
    <div class="progress row" style="height: 20px;">
      <div id="success" class="progress-bar bg-success" style="width:90%">
        90%
      </div>

    </div>
    <div class="row">
      <div id="addItem" class="col">
        <h3>Добави електроуред</h3>
        <form class="">
          <div class="form-group row">
            <label for="selected" class="col-sm-2 col-form-label">Уред</label>
            <div class="col-sm-10">
              <select class="form-control " id="selected">
                <option value="Телевизор">Телевизор</option>
                <option value="Хладилник">Хладилник</option>
                <option value="Компютър">Компютър</option>
                <option value="Ел. крушка">Ел. крушка</option>
                <option value="Микровънлнова">Микровълнова</option>
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
            <input type="submit" class="btn btn-primary"  @click="addItem" onclick="return false" value="Добави">
          </div>
        </form>
      </div>
      <div class="col">
        <ul class="list-group">
          <li class="list-group-item" :id="index"  v-for="(item, index) in appliances"  > {{ item.application  }} | {{ item.watts }}W
            <label @click="item.isOn = !item.isOn" class="switch" >
              <input class="inputs" @click="toggle(index)" type="checkbox" checked >
              <span   class="slider round"></span>
            </label>
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
        const watts = function () {
          const MAX_WATT= 2400;
          let success = getSuccessEl.innerHTML.split("%")[0].toString();
          return (success*1)/100*MAX_WATT;
        };
        const setWatts = function (watts) {
          let newWatts = (watts*100/2400).toFixed(2);
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
          return getSuccessEl.innerHTML.split("%")[0].toString();
        };
        let consume;
        self = this;
        setInterval(function () {
          consume = self.totalConsumption();
          getHour = date.getHours();
          result = watts();
          if(getWatts() <= 0) {
            document.getElementById("generated").innerText = "Battery life: 0W 0Ah";
            getSuccessEl.style.width =  "0%";
            getSuccessEl.innerHTML = 0 + "%";
            setWatts(0);
            console.log(self.appliances);

            console.log(self.appliances[0].isOn);
            for (let i = 0; i < self.appliances.length; i += 1) {
              self.appliances[i].isOn = false;

              document.getElementsByClassName("inputs")[i].removeAttribute("checked");
              document.getElementsByClassName("inputs")[i].removeAttribute("type");
            }
          } else {
            result -= consume / 6;
            setWatts(result);
            //console.log(result);
            document.getElementById("generated").innerText = "Battery life: " + result.toFixed(2) + "W " +
              (result/12).toFixed(2) + "Ah";
          }
          if (getHour >= sunrise && getHour <= sunset) {

            if ((getHour >= sunrise && getHour < sunrise + 1) ||
              (getHour >= sunset - 1 && getHour <= sunset )) {
              result += 500 / ((sunset - sunrise)) / (6 * self.divider); // generated for 10minutes
              result = parseInt(result);
              setWatts(result);
            }
            else if ((getHour >= sunrise + 1 && getHour < sunrise + 2) ||
              (getHour >= sunset - 2 && getHour <= sunset - 1)) {
              result += 600 / ((sunset - sunrise)) / (6 * self.divider); // generated for 10minutes
              result = parseInt(result);
              setWatts(result);
            }
            else if ((getHour >= sunrise + 2 && getHour < sunrise + 3) ||
              (getHour >= sunset - 3 && getHour <= sunset - 2)) {
              result += 700 / ((sunset - sunrise)) / (6 * self.divider); // generated for 10minutes
              result = parseInt(result);
              setWatts(result);
            }
            else if ((getHour >= sunrise + 3 && getHour < sunrise + 4) ||
              (getHour >= sunset - 4 && getHour <= sunset - 3)) {
              result += 800 / ((sunset - sunrise)) / (6 * self.divider); // generated for 10minutes
              result = parseInt(result);
              setWatts(result);
            }
            else if ((getHour >= sunrise + 4 && getHour < sunrise + 5) ||
              (getHour >= sunset - 5 && getHour < sunset - 4)) {
              result += 900 / ((sunset - sunrise)) / (6 * self.divider); // generated for 10minutes
              result = parseInt(result);
              setWatts(result);
            }
            else {
              if (getWatts() < 100) {
                result += 1000 / ((sunset - sunrise)) / 6; // generated for 10minutes
                result = parseInt(result);
                setWatts(result);
              }
            }
          }

          console.log(self.divider);

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
  float: left;
  padding: 0;
  margin: 0;;
  text-align: left;
}

.switch {
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




</style>
