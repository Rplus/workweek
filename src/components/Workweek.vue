<template>
  <div class="jumbotron">
    <div class="container">
      <h1><span class="glyphicon glyphicon-wrench"></span>勞基法計算機</h1>
      <p>
        沒人搞得清楚這次修法正確的計算方式，不如來個計算機自己按一按吧。
      </p>
    </div>
  </div>
  <div class="container">
    <div class="alert alert-info">
      <span class="glyphicon glyphicon-pencil" aria-hidden="true"></span>
      <span>
        前置條件：
        <ul>
          <li>現行勞基法與一例一休的例假日假設為週日，兩例的例假日為週六與週日</li>
          <li>現行勞基法的週六假設為約定不用上班的日子</li>
          <li>一例一休的休息日假設為星期六</li>
          <li>基於以上假設，在計算輪班制度時（例如四班二輪）可能會與實際狀況有誤差</li>
        </ul>
      </span>
    </div>
    <h2>每日工時</h2>
    <p>假設勞工採月薪制，其月薪 <input class="monthly-pay" type="number" v-model="monthlyPay"> 元，每月總工時為 {{assumingWorkHours}} 計算，平均時薪為 {{hourlyPay}} 元</p>
    <div class="input">
      <label>週一 <input number class="workhours" v-model="workhours[0]"></label>
      <label>週二 <input number class="workhours" v-model="workhours[1]"></label>
      <label>週三 <input number class="workhours" v-model="workhours[2]"></label>
      <label>週四 <input number class="workhours" v-model="workhours[3]"></label>
      <label>週五 <input number class="workhours" v-model="workhours[4]"></label>
      <label>週六 <input number class="workhours" v-model="workhours[5]"></label>
      <label>週日 <input number class="workhours" v-model="workhours[6]"></label>
    </div>
    <div class="row">
      <div class="col-md-4">
        <h3>勞基法現行版本</h3>
        <ul>
          <li>週薪：{{regularPay}} 元</li>
          <li>加班費：{{currentSolution.overtimePay}} 元</li>
          <li>總計週薪：{{regularPay + currentSolution.overtimePay}} 元</li>
          <li>工時：{{totalWorkHours}}</li>
          <li v-if="workhours[6] > 0" class="warning">額外補休時數：1 日</li>
          <li class="warning" v-show="workhours[6] > 0">
            只有在天災、事變或突發事件才可在週日工作。
          </li>
        </ul>
        <table class="week">
          <th v-for="name in daynames">
            {{ name }}
          <th>
          <tr v-for="hour in currentSolution.transposed">
            <td v-for="day in hour" track-by="$index">
              <span v-if="day !== 0" class="emoji" v-bind:class="{
                'regular': day === 1,
                'overtime': day === 2,
                'overtime-two-hours': day === 3,
                'work-on-dayoff': day === 4,
                'off': day === 0
              }"></span>
              <span class="emoji" v-if="day === 0">--</span>
            </td>
          </tr>
        </table>
        <div class="alert alert-info">
          相關規定：
          <ul>
            <li>台(87)勞動二字第39675號函：例假日（通常是週日）上班低於八個小時，薪水均為 {{hourlyPay}} x 8</li>
          </ul>
        </div>
      </div>
      <div class="col-md-4">
        <h3>勞動部草案一例一休版本</h3>
        <ul>
          <li>週薪：{{regularPay}} 元</li>
          <li>加班費：{{oneRestOneOffSolution.overtimePay}} 元</li>
          <li>總計週薪：{{regularPay + oneRestOneOffSolution.overtimePay}} 元</li>
          <li>工時：{{totalWorkHours}}</li>
          <li v-if="workhours[6] > 0" class="warning">額外補休時數：1 日</li>
          <li class="warning" v-show="workhours[6] > 0">
            只有在天災、事變或突發事件才可在週日工作。
          </li>
        </ul>
        <table class="week">
          <th v-for="name in daynames">
            {{ name }}
          <th>
          <tr v-for="hour in oneRestOneOffSolution.transposed">
            <td v-for="day in hour" track-by="$index">
              <span v-if="day !== 0" class="emoji" v-bind:class="{
                'regular': day === 1,
                'overtime': day === 2,
                'overtime-two-hours': day === 3,
                'work-on-dayoff': day === 4,
                'off': day === 0
              }"></span>
              <span class="emoji" v-if="day === 0">--</span>
            </td>
          </tr>
        </table>
        <div class="alert alert-info">
          相關規則：
          <ul>
            <li>
              根據 <a href="http://www.cna.com.tw/news/firstnews/201606290106-1.aspx" target="_blank">報導</a>，休息日工資計算的部分，擬從原本的加倍發給，改為在2小時以內者，按平日工資額另給予每小時1又1/3，再繼續工作者另給予每小時1又2/3。
            </li>
            <li>工作時間計算方式為工作4小時以內，以4小時計算，超過4小時至8小時，以8小時計算，超過8小時至12小時以內者，以12小時計。</li>
          </ul>
        </div>
      </div>
      <div class="col-md-4">
        <h3>一週兩例假日版本</h3>
        <ul>
          <li>週薪：{{regularPay}} 元</li>
          <li>加班費：{{twoOffSolution.overtimePay}} 元</li>
          <li>總計週薪：{{regularPay + twoOffSolution.overtimePay}} 元</li>
          <li>工時：{{totalWorkHours}}</li>
          <li v-if="workhours[6] > 0 || workhours[5] > 0" class="warning">
            額外補休時數：{{ (workhours[6] > 0 ? 1 : 0) + (workhours[5] > 0 ? 1 : 0) }} 日
          </li>
          <li class="warning" v-show="workhours[6] > 0 || workhours[5] > 0">
            只有在天災、事變或突發事件才可在例假日（如週六、週日）工作。
          </li>
        </ul>
        <table class="week">
          <th v-for="name in daynames">
            {{ name }}
          <th>
          <tr v-for="hour in twoOffSolution.transposed">
            <td v-for="day in hour" track-by="$index">
              <span v-if="day !== 0" class="emoji" v-bind:class="{
                'regular': day === 1,
                'overtime': day === 2,
                'overtime-two-hours': day === 3,
                'work-on-dayoff': day === 4,
                'off': day === 0
              }"></span>
              <span class="emoji" v-if="day === 0">--</span>
            </td>
          </tr>
        </table>
      </div>
    </div>
  </div>
  <div class="footer">
    本專案源碼於 <a href="https://github.com/g0v/workweek">g0v/worweek</a> 以 MIT 授權釋出，
    有任何問題請提交至 <a href="https://github.com/g0v/workweek/issues">issue tracker</a>
  </div>
</template>

<script>
import * as solutions from '../lib/solutions';

export default {
  data () {
    let workhours = [8, 8, 8, 8, 8, 0, 0];

    return {
      daynames: solutions.DAY_NAMES,
      workhours: workhours,
      assumingWorkHours: 240,
      monthlyPay: 36000,
      regularHoursPerDay: solutions.REGULAR_HOURS_PER_DAY
    };
  },
  computed: {
    hourlyPay: function () {
      return parseInt(this.monthlyPay / this.assumingWorkHours);
    },
    regularPay: function () {
      return this.hourlyPay * 8 * 7;
    },
    currentSolution: function () {
      return solutions.current(this.workhours, this.hourlyPay);
    },
    oneRestOneOffSolution: function () {
      return solutions.oneRestOneOff(this.workhours, this.hourlyPay);
    },
    twoOffSolution: function () {
      return solutions.twoOff(this.workhours, this.hourlyPay);
    },
    totalWorkHours: function () {
      return this.workhours.reduce((a, b) =>
              (parseInt(a) || 0) + (parseInt(b) || 0));
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
table.week {
  width: 100%;
}

table.week td, table.week th {
  text-align: center;
}

.warning {
  color: red;
  font-weight: bold;
}

.emoji {
  width: 25px;
  height: 25px;
  background-size: contain;
  background-position: center;
  background-repeat: no-repeat;
  display: block;
  margin: auto;
}

.emoji.regular {
  background-image: url('../assets/regular.png');
}

.emoji.overtime {
  background-image: url('../assets/overtime.png');
}

.emoji.overtime-two-hours {
  background-image: url('../assets/overtime-two-hours.png');
}

.emoji.work-on-dayoff {
  background-image: url('../assets/dayoff.png');
}

.footer {
  margin-top: 50px;
  padding: 5px;
  background: #EEE;
  text-align: center;
}

.monthly-pay {
  width: 80px;
  text-align: center;
}

.assuming-work-hours {
  width: 50px;
  text-align: center;
}

input.workhours {
  width: 50px;
  text-align: center;
  margin-right: 20px;
}

</style>
