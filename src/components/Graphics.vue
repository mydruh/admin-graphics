<script>
import Chart from 'chart.js/auto';
import axios from 'axios'

export default {
  data() {
    return{
      defaultDateTime: '',
      payments_sum_total: null,
      payments_sum_today: null,
      payments_count: null,
      tickets_count: null,
      payments_sum_list: [],
      payments_amount_list: [],
      tickets_list: []
    }
  },
  async mounted() {
    var dateControl = document.getElementById('dateFrom');
    var dateControlTo = document.getElementById('dateTo');

    var currentDate = new Date()
    currentDate.setMonth(currentDate.getMonth() - 1);

    dateControlTo.value = new Date().toISOString().slice(0, 10);
    dateControl.value = currentDate.toISOString().slice(0, 10);

    var minMonth = this.convertDate(currentDate.toISOString().slice(0, 10))

    var today = this.convertDate(new Date().toISOString().slice(0, 10))

    const article = { 
      from: minMonth,
      to: today, 
    };

    const headers = { 
      'Authorization': 'Token ' + localStorage.getItem('token'),//'3c41289ea70c3c4f1aade7c8f467de2ae0f15872'
    };

    await axios
        .post('https://api.picnic.esquire.kz/api/picnic/statistics/', article, { headers })
        .then((response) => {
          console.log(response.data)
          this.payments_sum_total = response.data.data.payments_sum_total.toLocaleString('ru-RU')
          this.payments_sum_today = response.data.data.payments_sum_today.toLocaleString('ru-RU')
          this.payments_count = response.data.data.payments_count.toLocaleString('ru-RU')
          this.tickets_count = response.data.data.tickets_count.toLocaleString('ru-RU')

          this.payments_sum_list = JSON.parse(JSON.stringify(response.data.data.payments_sum_list).replaceAll('"date"', '"x"').replaceAll('"payments_sum_total"', '"y"').replaceAll('00:00', ''))
          this.payments_amount_list = JSON.parse(JSON.stringify(response.data.data.payments_amount_list).replaceAll('"date"', '"x"').replaceAll('"payments_count"', '"y"').replaceAll('00:00', ''))
          this.tickets_list = JSON.parse(JSON.stringify(response.data.data.tickets_list).replaceAll('"date"', '"x"').replaceAll('"tickets_count"', '"y"').replaceAll('00:00', ''))
          
          this.payments_sum_list = this.yearCut(this.payments_sum_list)
          this.payments_amount_list = this.yearCut(this.payments_amount_list)
          this.tickets_list = this.yearCut(this.tickets_list)

          this.select2Init()
          this.select2Init2()
        }).catch((err) => {
          console.log(err)
        })

    this.graphicInit()
  },
  methods: {
    graphicInit(){
      try{
        window.myChart.destroy()
        window.myChart1.destroy()
        window.myChart2.destroy()
      }catch(err){
        console.log(err)
      }

      var ctx = document.getElementById('myChart').getContext('2d');
      var ctx1 = document.getElementById('myChart1').getContext('2d');
      var ctx2 = document.getElementById('myChart2').getContext('2d');

      window.myChart = new Chart(ctx, {
      type: 'line',
      data: {
        //labels: ['Red', 'Blue', 'Yellow', 'Green', 'Purple', 'Orange'],
        datasets: [{
          label: false,
          data: this.payments_sum_list,
          backgroundColor: '#79aec8a6',
          borderColor: '#417690',
          color: '#417690',
          borderWidth: 1,
          fill: true
        }]
      },
      options: {
        scales: {
          y: {
            beginAtZero: true
          }
        },
        plugins: {
          legend: {
              display: false
          }
        }
      },
      responsive: true
    });

    window.myChart1 = new Chart(ctx1, {
      type: 'line',
      data: {
        //labels: ['Red', 'Blue', 'Yellow', 'Green', 'Purple', 'Orange'],
        datasets: [{
          label: 'Кол-во',
          data: this.payments_amount_list,
          backgroundColor: '#79aec8a6',
          borderColor: '#417690',
          color: '#417690',
          borderWidth: 1,
          fill: true
        }]
      },
      options: {
        scales: {
          y: {
            beginAtZero: true
          }
        },
        plugins: {
          legend: {
            display: false
          }
        }
      },
      responsive: true
    });

    window.myChart2 = new Chart(ctx2, {
      type: 'line',
      data: {
        //labels: ['Red', 'Blue', 'Yellow', 'Green', 'Purple', 'Orange'],
        datasets: [{
          label: 'Кол-во',
          data: this.tickets_list,
          backgroundColor: '#79aec8a6',
          borderColor: '#417690',
          color: '#417690',
          borderWidth: 1,
          fill: true
        }]
      },
      options: {
        scales: {
          y: {
            beginAtZero: true
          }
        },
        plugins: {
          legend: {
            display: false
          }
        }
      },
      responsive: true
    });
    },
    async searchFnc(){
      var inputFrom = document.getElementById('dateFrom').value
      var inputTo = document.getElementById('dateTo').value

      if(inputFrom != '' || inputTo != ''){
        inputFrom = this.convertDate(inputFrom)
        inputTo = this.convertDate(inputTo)

        const article = { 
          from: inputFrom,
          to: inputTo, 
          picnics: $("#picnicSelect").select2("val"),
          partners: $("#partnerSelect").select2("val") 
        };

        const headers = { 
          'Authorization': 'Token ' + localStorage.getItem('token'),//'3c41289ea70c3c4f1aade7c8f467de2ae0f15872'
        };

        await axios
            .post('https://api.picnic.esquire.kz/api/picnic/statistics/', article, { headers })
            .then((response) => {
              console.log(response.data)
              this.payments_sum_total = response.data.data.payments_sum_total.toLocaleString('ru-RU')
              this.payments_sum_today = response.data.data.payments_sum_today.toLocaleString('ru-RU')
              this.payments_count = response.data.data.payments_count.toLocaleString('ru-RU')
              this.tickets_count = response.data.data.tickets_count.toLocaleString('ru-RU')

              this.payments_sum_list = JSON.parse(JSON.stringify(response.data.data.payments_sum_list).replaceAll('"date"', '"x"').replaceAll('"payments_sum_total"', '"y"').replaceAll('00:00', ''))
              this.payments_amount_list = JSON.parse(JSON.stringify(response.data.data.payments_amount_list).replaceAll('"date"', '"x"').replaceAll('"payments_count"', '"y"').replaceAll('00:00', ''))
              this.tickets_list = JSON.parse(JSON.stringify(response.data.data.tickets_list).replaceAll('"date"', '"x"').replaceAll('"tickets_count"', '"y"').replaceAll('00:00', ''))
              
              this.payments_sum_list = this.yearCut(this.payments_sum_list)
              this.payments_amount_list = this.yearCut(this.payments_amount_list)
              this.tickets_list = this.yearCut(this.tickets_list)

              this.graphicInit()
            }).catch((err) => {
              console.log(err)
              alert(err.response.data.errors.detail)
            })
      }else{
        alert("Все поля должны быть заполнены")
      }
    },
    select2Init(){
      $('#partnerSelect').select2({
      placeholder: "Партнёр",
      multiple: true,
      //minimumInputLength: 3,
      ajax: {
          url: "https://api.picnic.esquire.kz/api/partners/",
          headers: {
            'Authorization': 'Token ' + localStorage.getItem('token'),//'3c41289ea70c3c4f1aade7c8f467de2ae0f15872'
          },
          dataType: "json",
          delay: 250,
          processResults: function(data) {
            var result = []

            data.data.map(function (item) {
              var node = {
                id: item.uuid,
                text: item.name
              }

              result.push(node)
            })

            return {
              results: result
            };
          },
          cache: true,
          },
          dropdownCssClass: "my-dropdown"
      });
    },
    select2Init2(){
      $('#picnicSelect').select2({
      placeholder: "Пикник",
      multiple: true,
      //minimumInputLength: 3,
      ajax: {
          url: "https://api.picnic.esquire.kz/api/picnics/",
          headers: {
            'Authorization': 'Token ' + localStorage.getItem('token'),//'3c41289ea70c3c4f1aade7c8f467de2ae0f15872'
          },
          dataType: "json",
          delay: 250,
          processResults: function(data) {
            var result = []

            data.data.map(function (item) {
              var node = {
                id: item.uuid,
                text: item.name
              }

              result.push(node)
            })

            return {
              results: result
            };
          },
          cache: true,
          },
          conditionalselect: function (node, event) {
            console.log(node)
          },
          dropdownCssClass: "my-dropdown"
      });
    },
    convertDate(dateStr){
      const isoDate = dateStr; // Replace with the actual ISO date string
      const date = new Date(isoDate);
      const day = date.getDate().toString().padStart(2, "0");
      const month = (date.getMonth() + 1).toString().padStart(2, "0");
      const year = date.getFullYear().toString();
      const hours = date.getHours().toString().padStart(2, "0");
      const minutes = date.getMinutes().toString().padStart(2, "0");
      const convertedDate = `${day}-${month}-${year} ${hours}:${minutes}`;

      return convertedDate; // Replace with your desired output method     
    },
    yearCut(arr){
      for(var i=0;i<arr.length;i++){
        arr[i].x = arr[i].x.slice(0, 5)
      }

      return arr
    }
  }
}
</script>

<template>
  <div>
    <div class="container" style="margin-top: 50px;">
      <div class="row row-cols-3">
        <div class="col">
      <div class="card" style="border: none">
        <p class="card-text">От (дата)</p>
        <input id="dateFrom" type="date" class="form-control" placeholder="От (дата)" aria-label="От (дата)">
      </div>
    </div>

    <div class="col">
      <div class="card" style="border: none">
        <p class="card-text">До (дата)</p>
        <input id="dateTo" type="date" class="form-control" placeholder="До (дата)" aria-label="До (дата)">
      </div>
    </div>

    <div class="col" style="text-align: -webkit-center;">
      <div class="card" style="width: 50%;">
        <button @click="searchFnc" type="button" class="btn btn-dark btn-search">Поиск</button>
      </div>
    </div>
  </div>

  <div class="row row-cols-3" style="margin-top: 30px">
  <div class="col">
      <div class="card" style="height: 100%; border: none">
        <p class="card-text">Пикник</p>
        <select id="picnicSelect" class="form-control js-example-basic-single" name="state"></select>
    </div>

    </div>
    <div class="col">
      <div class="card" style="height: 100%; border: none">
        <p class="card-text">Партнеры</p>
        <select id="partnerSelect" class="form-control js-example-basic-single" name="state"></select>
      </div>
    </div>
  </div>

  <div class="row row-cols-4" style="margin-top: 40px;">
    <div class="col">
      <div class="card">
        <div class="card-body">
          <h5 class="card-title">{{this.payments_sum_total}} ₸</h5>
          <p class="card-text">Оплаты за период</p>
        </div>
      </div>
    </div>

    <div class="col">
      <div class="card">
        <div class="card-body">
          <h5 class="card-title">{{this.payments_sum_today}} ₸</h5>
          <p class="card-text">Оплаты сегодня</p>
        </div>
    </div>

    </div>
    <div class="col">
      <div class="card">
        <div class="card-body">
          <h5 class="card-title">{{this.payments_count}}</h5>
          <p class="card-text">Транзакции</p>
        </div>
      </div>
    </div>

    <div class="col">
      <div class="card">
        <div class="card-body">
          <h5 class="card-title">{{this.tickets_count}}</h5>
          <p class="card-text">Продано билетов</p>
        </div>
      </div>
    </div>
  </div>

  <div class="row row-cols-3" style="margin-top: 40px;">
    <div class="col">
      <h5 class="card-title" style="margin-bottom: 20px; text-transform: none;">Сумма оплат</h5>
      <canvas id="myChart"></canvas>
    </div>
    <div class="col">
      <h5 class="card-title" style="margin-bottom: 20px; text-transform: none;">Количество оплат</h5>
      <canvas id="myChart1"></canvas>
    </div>
    <div class="col">
      <h5 class="card-title" style="margin-bottom: 20px; text-transform: none;">Количество билетов</h5>
      <canvas id="myChart2"></canvas>
    </div>
  </div>
</div>
  </div>
</template>

<style>

.select2-container{
  height: 100% !important;
}

.select2-selection{
  height: 100% !important;
}

.select2-selection__choice{
  background-color: #79aec8 !important;
  color: white;
}

.select2-selection__choice button{
  color: white !important;
  border-right: 1px solid #fff !important;
}

.select2-selection__rendered{
  margin-left: 5px !important;
}

h5, p {
  color: #417690 !important
}

.btn-search{
  background: #417690 !important;
  border: none;
  border: none !important;
}

.btn-search:hover{
  background: #79aec8 !important;
  border: none !important;
}

.my-bootstrap-namespace .card-text:last-child{
  padding: 0 !important;
}

.select2-search__field{
  height: 24px !important;
}
</style>