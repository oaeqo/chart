<template>
  <div class="date-analysis">
    <h2>일자별 출결현황</h2>
    <input type="date" v-model="selectedDate"/>
    <div class="chart-container" v-if="isDate">
      <canvas ref="chartCanvas"></canvas>
    </div>
    <!-- 데이터가 없다면 -->
   <p v-else> 해당 날짜의 출결 데이터가 없습니다 </p>
  </div>
</template>

<script setup>
import { Chart, registerables } from 'chart.js';
import { nextTick, ref, watch } from 'vue';

const props = defineProps({
  students: Array
});

//선택한 날짜
const selectedDate = ref('');
const isDate = ref(null);
//차트 그리기
let chartInst = null;
const chartCanvas = ref('');
Chart.register(...registerables);
//선택한 날짜가 변경되면
watch(selectedDate, async ()=>{
  // console.log( selectedDate.value ); //2025-02-11로 출력
  const attendanceCount = { present:0, leave:0 , absent:0, late:0 };
  isDate.value = false;
  props.students.forEach((list)=>{
    // console.log(list);
    list.attendance.forEach((item)=>{
      if( item.date === selectedDate.value ){
        attendanceCount[item.status]++;
        isDate.value = true;
      }
    });
  });

  //새로 만들기 위해, 기존 차트 삭제
  if( chartInst ) {
    chartInst.destroy();
  }
  //데이터가 없다면 차트를 생성할 필요가 없음
  if( !isDate.value ) return;
  // DOM 업데이트가 되면 재갱신
  nextTick();
  //차트 그리기 : bar
  chartInst = new Chart(chartCanvas.value,{
    type: "bar",
    data:{
      labels: ['출석', '지각', '결석', '조퇴'],
      datasets: [{
        label:"출결현황",
        data: [attendanceCount.present, attendanceCount.late, attendanceCount.absent, attendanceCount.leave],
        backgroundColor: ['yellow', 'green', 'blue', 'red']
      }]
    },
    options: {
      responsive: true,
      plugins: {
        legend: {position:'bottom'}
      }
    }
  });
});
</script>

<style lang="scss" scoped>
.date-analysis{
  input{
    padding: 5px;
  }
  p{
    padding: 30px;
  }
}

.chart-container{
  width: 100%;
  max-width: 400px;
  margin: auto;

}
select{
  width: 70%;
  height: 2rem;
  font-size: 1.5rem;
}
</style>