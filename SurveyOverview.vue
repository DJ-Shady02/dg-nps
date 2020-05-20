<template>
  <div class="survey-overview">
    <!-- Time Period Selection -->
    <!-- <div class="input-group--horizontal">
      <label class="input-horizontal-radio" v-for="(timeFrame, index) in timeFrames" :key="index">{{timeFrameLabels[index]}}
        <input type="radio" name="horizontal-radio-group" :checked="(index === selectedTimeFrameIndex)" @change="setSelectedTimeFrame(timeFrame, index)">
        <span class="horizontal-radio-button"></span>
      </label>
    </div> -->

    <!-- Status Container -->
    <div class="status-container">
      <div class="status-container__score-container">
        <h2 class="h1 remove--margin-top">NPS Score: <span class="primary-highlight">{{ scoreByMonth[currentMonthIndex].score }}</span></h2>
        {{ scoreByMonth[currentMonthIndex].amountAnswers }} responses
      </div>
      <div class="status-container__percentages-container">
        <div class="status-container__percentages-container__promoters">
          <v-icon class="status-icon fa-smile" name="regular/smile" scale="4"/>
          <div>
            <h2 class="h1 remove--margin-top">{{ scoreByMonth[currentMonthIndex].percentPromoters }}%</h2>
            {{ scoreByMonth[currentMonthIndex].amountPromoters }} promoters
          </div>
        </div>
        <div class="status-container__percentages-container__passives">
          <v-icon class="status-icon fa-meh" name="regular/meh" scale="4"/>
          <div>
            <h2 class="h1 remove--margin-top">{{ scoreByMonth[currentMonthIndex].percentPassives }}%</h2>
            {{ scoreByMonth[currentMonthIndex].amountPassives }} passives
          </div>
        </div>
        <div class="status-container__percentages-container__detractors">
          <v-icon class="status-icon fa-frown" name="regular/frown" scale="4"/>
          <div>
            <h2 class="h1 remove--margin-top">{{ scoreByMonth[currentMonthIndex].percentDetractors }}%</h2>
            {{ scoreByMonth[currentMonthIndex].amountDetractors }} detractors
          </div>
        </div>
      </div>
    </div>

    <!-- Score Graph Container -->
    <div class="score-graph-container">
      <line-chart :chartData="chartDataScore" :chartOptions="chartOptionsScore" style="height: 300px;"/>
    </div>

    <div class="split-graph-container">
      <!-- Likes and Dislikes Graph Container -->
      <div class="split-graph-container__graph-container">
        <h2 class="h1 remove--margin-top">Likes and Dislikes</h2>
        <horizontal-bar-chart :chartData="chartDataOpinions" :chartOptions="chartOptionsOpinions"/>
      </div>

      <!-- Grouping Percentage Graph Container -->
      <div class="split-graph-container__graph-container">
        <h2 class="h1 remove--margin-top">Group Percentage</h2>
        <horizontal-bar-chart :chartData="chartDataGroup" :chartOptions="chartOptionsGroup" style="height: 600px;"/>
      </div>
    </div>
  </div>
</template>

<script>
// import FetchingLoader from '@/components/GlobalComponents/FetchingLoader';
import { firestore } from "@/firebase";
import LineChart from './LineChart';
import HorizontalBarChart from './HorizontalBarChart';

export default {
  components: {
    LineChart,
    HorizontalBarChart,
  },
  data() {
    return {
      answers: [],
      // selectedTimeFrame: 7,
      // selectedTimeFrameIndex: 0,
      // timeFrames: [7, 14, 28],
      // timeFrameLabels: ["7 days", "2 weeks", "1 month"],
    }
  },
  computed: {
    currentMonthIndex() {
      return new Date(Date.now()).getMonth();
    },
    answersByMonth() {
      let answers = this.answers;
      let sortedAnswers = {};

      // If no date, remove answer, as we dont know where to put it.
      answers.forEach((answer, index) => {
        if (!answer.timestamp) {
          answers.splice(index, 1);
        }
      })      
      
      // get month and filter to correct placement
      sortedAnswers.jan = answers.filter(answer => new Date(answer.timestamp).getMonth() === 0);
      sortedAnswers.feb = answers.filter(answer => new Date(answer.timestamp).getMonth() === 1);
      sortedAnswers.mar = answers.filter(answer => new Date(answer.timestamp).getMonth() === 2);
      sortedAnswers.apr = answers.filter(answer => new Date(answer.timestamp).getMonth() === 3);
      sortedAnswers.may = answers.filter(answer => new Date(answer.timestamp).getMonth() === 4);
      sortedAnswers.jun = answers.filter(answer => new Date(answer.timestamp).getMonth() === 5);
      sortedAnswers.jul = answers.filter(answer => new Date(answer.timestamp).getMonth() === 6);
      sortedAnswers.aug = answers.filter(answer => new Date(answer.timestamp).getMonth() === 7);
      sortedAnswers.sep = answers.filter(answer => new Date(answer.timestamp).getMonth() === 8);
      sortedAnswers.oct = answers.filter(answer => new Date(answer.timestamp).getMonth() === 9);
      sortedAnswers.nov = answers.filter(answer => new Date(answer.timestamp).getMonth() === 10);
      sortedAnswers.dec = answers.filter(answer => new Date(answer.timestamp).getMonth() === 11);


      return sortedAnswers;
    },
    scoreByMonth() {
      let npsScoreMonthly = [];

      // console.log(Object.keys(this.answersByMonth));
      
      for (const month in this.answersByMonth) {
        let answers = this.answersByMonth[month]; //this.answersByMonth.month
        let amountAnswers = answers.length;
        let percentPromoters = 0;
        let percentPassives = 0;
        let percentDetractors = 0;
        let amountPromoters = 0;
        let amountPassives = 0;
        let amountDetractors = 0;
        
        // If not empty array
        if (answers.length > 0) {          
          let promoters = answers.filter(answer => answer.score >= 9);
          let passives = answers.filter(answer => answer.score == 7 || answer.score == 8);
          let detractors = answers.filter(answer => answer.score <= 6);

          percentPromoters = promoters.length / answers.length * 100;
          percentPassives = passives.length / answers.length * 100;
          percentDetractors = detractors.length / answers.length * 100;
          amountPromoters = promoters.length;
          amountPassives = passives.length;
          amountDetractors = detractors.length;
        }

        let npsScore = percentPromoters - percentDetractors;
        
        // round to 1 decimal(s)
        // npsScore = Math.round(npsScore * 10) / 10;

        npsScore = Math.round(npsScore);
        percentPromoters = Math.round(percentPromoters);
        percentPassives = Math.round(percentPassives);
        percentDetractors = Math.round(percentDetractors);

        npsScoreMonthly.push({
          score: npsScore,
          percentPromoters: percentPromoters,
          percentPassives: percentPassives,
          percentDetractors: percentDetractors,
          amountAnswers: amountAnswers,
          amountPromoters: amountPromoters,
          amountPassives: amountPassives,
          amountDetractors: amountDetractors,
        });
        
      }
      
      return npsScoreMonthly;
    },
    opinionsThisMonth() {
      // Get answers for current month
      let labels = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'];
      let currentMonth = labels[this.currentMonthIndex];
      currentMonth = currentMonth.toLowerCase();      
      const answers = this.answersByMonth[currentMonth]; // This.answersByMonth.month      

      // Simplify the count by removing individual answers
      let likes = [];
      let dislikes = [];

      // For every answer, loop through array of likes/dislikes and add to array
      answers.forEach(answer => {
        answer.likes.forEach(like => {
          likes.push(like);
        })
        answer.dislikes.forEach(dislike => {
          dislikes.push(dislike);
        })
      })
      
      // accumulate counts of each element
      let likesCount = this.accumulateCounts(likes);
      let dislikesCount = this.accumulateCounts(dislikes);

      return {likes: likesCount, dislikes: dislikesCount};
      
    },
    chartDataScore() {
      let data = [
        this.scoreByMonth[0].score,
        this.scoreByMonth[1].score,
        this.scoreByMonth[2].score,
        this.scoreByMonth[3].score,
        this.scoreByMonth[4].score,
        this.scoreByMonth[5].score,
        this.scoreByMonth[6].score,
        this.scoreByMonth[7].score,
        this.scoreByMonth[8].score,
        this.scoreByMonth[9].score,
        this.scoreByMonth[10].score,
        this.scoreByMonth[11].score
      ];
      let labels = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'];

      return {
        labels: labels,
        datasets: [{
          label: 'NPS Score',
          data: data,
          fill: false,
          borderColor: '#48B9BA',
          borderWidth: '2',
          lineTension: '0',
          pointBackgroundColor: '#48B9BA',
          pointBorderWidth: '0',
          pointRadius: '4',
        }],
      }
    },
    chartOptionsScore() {
      return {
        maintainAspectRatio: false,
        legend: {
          display: false,
        },
        tooltips: {
          intersect: false,
          displayColors: false,
          xPadding: 25,
          yPadding: 8,
          titleFontSize: 12,
          bodyFontSize: 16,
          titleFontFamily: "'Poppins', sans-serif",
          bodyFontFamily: "'Poppins', sans-serif",
          bodyAlign: 'center',
          titleAlign: 'center',
          backgroundColor: "#48b9ba",
        },
        scales: {
          yAxes: [{
            ticks: {
              beginAtZero: true,
              fontFamily: "'Poppins', sans-serif",
              fontSize: 16,
              fontColor: '#AAAAAA'
            },
            gridLines: {
              zeroLineColor: '#707070',
              color: '#AAAAAA',
              drawBorder: false,
            }
          }],
          xAxes: [{
            ticks: {
              fontFamily: "'Poppins', sans-serif",
              fontSize: 16,
              fontColor: '#AAAAAA'
            },
            gridLines: {
              display: false,
            } 
          }]
          
        }
      }
    },
    chartDataOpinions() {
      let dataLikes = [];
      let dataDislikes = [];
      let labels = [];

      const opinions = this.opinionsThisMonth;

      // First, sort to make sure both likes and dislikes are in the same order
      let sortedLikes = opinions.likes.sort((a, b) => {
        let valueA = a.value;
        let valueB = b.value;

        if (valueA.toLowerCase() < valueB.toLowerCase()) {
          return -1;
        }
        if (valueA.toLowerCase() > valueB.toLowerCase()) {
          return 1;
        }
        return 0;
      });
      let sortedDislikes = opinions.dislikes.sort((a, b) => {
        let valueA = a.value;
        let valueB = b.value;

        if (valueA.toLowerCase() < valueB.toLowerCase()) {
          return -1;
        }
        if (valueA.toLowerCase() > valueB.toLowerCase()) {
          return 1;
        }
        return 0;
      });

      console.log(sortedLikes);
      console.log(sortedDislikes);
      
      // construct data and labels
      sortedLikes.forEach(category => {
        dataLikes.push(category.count);
        labels.push(category.value);
      })
      sortedDislikes.forEach(category => {
        dataDislikes.push(category.count);
      })
      

      return {
        labels: labels,
        datasets: [{
          label: '# of Likes',
          data: dataLikes,
          borderWidth: 0,
          backgroundColor: '#48B9BA',
          barThickness: 24,
        },
        {
          label: '# of Dislikes',
          data: dataDislikes,
          borderWidth: 0,
          backgroundColor: '#D95D5D',
          barThickness: 24,
        }
        ],
      }
    },
    chartOptionsOpinions() {
      return {
        responsive: true,
        maintainAspectRatio: false,
        legend: {
          display: false,
        },
        tooltips: {
          displayColors: false,
          xPadding: 25,
          yPadding: 8,
          titleFontSize: 12,
          bodyFontSize: 16,
          titleFontFamily: "'Poppins', sans-serif",
          bodyFontFamily: "'Poppins', sans-serif",
          bodyAlign: 'center',
          titleAlign: 'center',
          backgroundColor: "#48b9ba",
        },
        scales: {
          yAxes: [{
            gridLines: {
              display: false,
            },
            ticks: {
              fontFamily: "'Poppins', sans-serif",
              fontSize: 16,
              fontColor: '#AAAAAA'
            },
          }],
          xAxes: [{
            position: 'top',
            ticks: {
              beginAtZero: true,
              fontFamily: "'Poppins', sans-serif",
              fontSize: 16,
              fontColor: '#AAAAAA'
            },
            gridLines: {
              zeroLineColor: '#707070',
              color: '#AAAAAA',
              drawBorder: false
            }
          }]
        }
      }
    },
    chartDataGroup() {
      let dataPromoter = [
        this.scoreByMonth[0].percentPromoters,
        this.scoreByMonth[1].percentPromoters,
        this.scoreByMonth[2].percentPromoters,
        this.scoreByMonth[3].percentPromoters,
        this.scoreByMonth[4].percentPromoters,
        this.scoreByMonth[5].percentPromoters,
        this.scoreByMonth[6].percentPromoters,
        this.scoreByMonth[7].percentPromoters,
        this.scoreByMonth[8].percentPromoters,
        this.scoreByMonth[9].percentPromoters,
        this.scoreByMonth[10].percentPromoters,
        this.scoreByMonth[11].percentPromoters
      ];
      let dataPassives = [
        this.scoreByMonth[0].percentPassives,
        this.scoreByMonth[1].percentPassives,
        this.scoreByMonth[2].percentPassives,
        this.scoreByMonth[3].percentPassives,
        this.scoreByMonth[4].percentPassives,
        this.scoreByMonth[5].percentPassives,
        this.scoreByMonth[6].percentPassives,
        this.scoreByMonth[7].percentPassives,
        this.scoreByMonth[8].percentPassives,
        this.scoreByMonth[9].percentPassives,
        this.scoreByMonth[10].percentPassives,
        this.scoreByMonth[11].percentPassives
      ];
      let dataDetractors = [
        this.scoreByMonth[0].percentDetractors,
        this.scoreByMonth[1].percentDetractors,
        this.scoreByMonth[2].percentDetractors,
        this.scoreByMonth[3].percentDetractors,
        this.scoreByMonth[4].percentDetractors,
        this.scoreByMonth[5].percentDetractors,
        this.scoreByMonth[6].percentDetractors,
        this.scoreByMonth[7].percentDetractors,
        this.scoreByMonth[8].percentDetractors,
        this.scoreByMonth[9].percentDetractors,
        this.scoreByMonth[10].percentDetractors,
        this.scoreByMonth[11].percentDetractors
      ];

      let labels = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'];

      return {
        labels: labels,
        datasets: [{
          label: '% of Promoters',
          data: dataPromoter,
          borderWidth: 0,
          backgroundColor: '#48B9BA',
          barThickness: 32,
        },
        {
          label: '% of Passives',
          data: dataPassives,
          borderWidth: 0,
          backgroundColor: '#AAAAAA',
          barThickness: 32,
        },
        {
          label: '% of Detractors',
          data: dataDetractors,
          borderWidth: 0,
          backgroundColor: '#D95D5D',
          barThickness: 32,
        }
        ],
      }
    },
    chartOptionsGroup() {
      return {
        maintainAspectRatio: false,
        legend: {
          display: false,
        },
        tooltips: {
          displayColors: false,
          xPadding: 25,
          yPadding: 8,
          titleFontSize: 12,
          bodyFontSize: 16,
          titleFontFamily: "'Poppins', sans-serif",
          bodyFontFamily: "'Poppins', sans-serif",
          bodyAlign: 'center',
          titleAlign: 'center',
          backgroundColor: "#48b9ba",
        },
        scales: {
          yAxes: [{
            stacked: true,
            gridLines: {
              display: false,
            },
            ticks: {
              fontFamily: "'Poppins', sans-serif",
              fontSize: 16,
              fontColor: '#AAAAAA'
            },
          }],
          xAxes: [{
            position: 'top',
            stacked: true,
            ticks: {
              beginAtZero: true,
              fontFamily: "'Poppins', sans-serif",
              fontSize: 16,
              fontColor: '#AAAAAA'
            },
            gridLines: {
              zeroLineColor: '#707070',
              color: '#AAAAAA',
              drawBorder: false
            }
          }]
        }
      }
    },
  },
  created() {
    firestore.collection("survey")
    .doc(this.$hotelId)
    .collection("answers")
    .orderBy("timestamp", "desc")
    .get()
    .then(snapshot => {        
      snapshot.forEach(doc => {
        let answer = doc.data();
        answer.id = doc.id;

        this.answers.push(answer)
      })
    })
    .catch(error => {
      console.log(error);
    })
  },
  methods: {
    // setSelectedTimeFrame(value, index) {
    //   this.selectedTimeFrame = value;
    //   this.selectedTimeFrameIndex = index;
    // },
    accumulateCounts(array) {
      return array.reduce((accumulator, currentValue) => {
        (
          // Get element in array with the same value as currentValue
          accumulator[accumulator.findIndex(item => item.value === currentValue)]
          ||
          // If none, create new array, then push new object to array, and get the created element
          accumulator[accumulator.push({value: currentValue, count: 0}) - 1]
        ).count++ // Either way, increase the count of expression in parenthesis

        // Return accumulator array, as reduce requires
        return accumulator;
      }, [])
    }
  }
}
</script>

<style scoped>

.primary-highlight {
  color: var(--primary);
}

.fa-smile {
  color: var(--primary);
}

.fa-meh {
  color: var(--grey);
}

.fa-frown {
  color: var(--alert);
}

.status-container {
  line-height: 1.25;
  height: 64px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 32px;
}

.status-container__percentages-container {
  display: flex;
}

.status-container__percentages-container__promoters,
.status-container__percentages-container__passives,
.status-container__percentages-container__detractors {
  display: flex;
  margin-left: 48px;
}

.status-icon {
  margin-right: 8px;
}

.score-graph-container {
  margin-bottom: 32px;
}

.split-graph-container {
  display: flex;
}

.split-graph-container__graph-container {
  width: 50%;
}

.split-graph-container__graph-container:first-child {
  padding-right: 16px;
}

.split-graph-container__graph-container:last-child {
  padding-left: 16px;
}

</style>