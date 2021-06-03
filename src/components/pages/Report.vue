<template>
    <div class="container">
      <div class="loading">
        <div class="centerScreen">
        <div class="content Timesroman">

            <h3 class="title"> Game Report</h3>
            <h2 class="title">Introdcution:</h2>
            <p class="body">This section presents a brief report of overall games played by different groups. In total, <strong>{{GroupFilter.group_ids.length}}</strong> group (<strong> {{GroupFilter.group_ids[0].group_id}}</strong>) has participated with <strong>{{result[0].numberPlayers}}</strong> students  from <strong>{{result[0].countries[0]}}</strong> and played  <strong>{{result.length}}</strong> game and <strong>{{result[0].chapters.length}}</strong> chapters, and more than <strong>{{n_decisions}}</strong> events.  Below presents a summary of each chapter and the risk of polarization for students regarding their provided decision-making during their games play. To better understand student psychology, we have labeled student sentiment in four categories (i.e., positive, negative, complex, and neutral). This sentiment analysis is prepared based on student decisions during gameplay.  Table 1 shows the example of annotations.</p>

            <p class="nb">[N.B. This report is automatically generated, which is intended to help teacher assessment.]</p>

            <center>
              <div class="col-md-8">
                <p class="fig">Table 1: example of event decision annotation.</p>
                <table class="table table-bordered">
                        <thead>
                          <tr>
                            <th scope="col">Event Question</th>
                            <th scope="col">Decision</th>
                            <th scope="col">Sentiment annotation</th>
                          </tr>
                        </thead>
                        <tbody>
                          <tr>
                            <td>How happy are you with your classmates?</td>
                            <td> A lot</td>
                            <td>Positive</td>
                          </tr>
                          <tr>
                            <td>Should you argue with your mom?</td>
                            <td>Shout at her</td>
                            <td>Negative</td>
                          </tr>
                          <tr>
                            <td>What shall you do now?</td>
                            <td>Run</td>
                            <td>Complex</td>
                          </tr>
                          <tr>
                            <td>Are you a boy or a girl?</td>
                            <td>Boy</td>
                            <td>Neutral</td>
                          </tr>

                        </tbody>
                  </table>
               </div>
            </center>


             <div v-for="(data, index) in all_final_data" :key="index">

                  <h2 class="title">Chapter {{data.eventid}}: Psychological evaluation</h2>
                  <p class="body">This chapter is played by gorup <strong>{{GroupFilter.group_ids[0].group_id}}</strong> with  participants <strong>{{data.total_decision}}</strong> (60% boy, 40% girls) and 80  (60% boy, 40% girls). Group_1 has a polarization risk of 80%, and group_2 has 70%. The most polarization decision has come from event ID (2,5,7), and the lowest polarization score comes from event ID (1,3), details scores of each event shown in Fig 1 and Fig 2. The most polarized decision came from event 7: How will you behave your mother and 80% of student answer was 'I will shout'.</p><br>

                  <p class="fig">Table. {{index+2}}: Emotional score of individual events decisions of chapter {{data.eventid}}.</p>
                  <table class="table table-bordered">
                          <thead>
                            <tr>
                              <th scope="col">#</th>
                              <th scope="col">Event Question</th>
                              <th scope="col">Decisions (decisions count)</th>
                              <th scope="col">Sentiment Score (total 100%)</th>
                            </tr>
                          </thead>
                          <tbody  v-for="(sentiment, index) in data.sentiment_score" :key="index">
                            <tr>
                              <th scope="row">{{sentiment.event}}</th>
                              <td>{{sentiment.event_des}}</td>
                              <td> <li class="nostyle" v-for="(choice, index) in sentiment.choice" :key="index"> {{index+1}}. {{choice.c}} ({{choice.n}})</li></td>
                              <td>
                                  <span v-if="sentiment.po>0">Positive: {{sentiment.po.toFixed(1)}}<br></span>
                                  <span v-if="sentiment.ne>0">Negative: {{sentiment.ne.toFixed(1)}}<br></span>
                                  <span v-if="sentiment.com>0">Complex: {{sentiment.com.toFixed(1)}}<br></span>
                                  <span v-if="sentiment.nu>0">Neutral: {{sentiment.nu.toFixed(1)}}<br></span>
                              </td>
                            </tr>

                          </tbody>
                    </table>

                  <center>       
                      <div class="col-md-6 center">
                            <v-chart :options="barGraph[index]" width="100%"/>
                            <p class="fig">Fig. {{index+1}}: Overall emotional statics of total student that played chapter {{data.eventid}}.  </p>
                      </div>
                  </center>

        <!--           <center>
                    
                        <div class="col-md-12 center">
                                  <v-chart :options="barGraphAll[index]" width="100%"/>
                                  <p class="fig">Fig. {{index+2}}: Emotional statics of individual events of chapter {{data.eventid}}.  </p>
                        </div>
                  </center>   -->
                  Among this decision, 60% of boys answer showed polarization risk other while 30% of girls showed polarization risk. Besides among 9, 11, 12,13 polarization risk was 30%, 10%, 15% and 20%. This indicates nine years old are more prone to be polarized.<br> 
             </div>
                <div class="row" style="padding: 20px 0">
                      <div class="col-4 text-center">
                          <button class="btn btn-primary" @click="home">Home</button>
                      </div>
                      <div class="col-4 text-right">
                          <button class="btn btn-primary export" @click="exportOpt">Export</button>
                      </div>
                </div>
            </div>

          </div>
      </div>
     </div>
</template>

<script>

    import ECharts from 'vue-echarts'
    import 'echarts/lib/chart/bar'
    import 'echarts/lib/component/tooltip'
    import 'echarts-gl'
    import axios from "axios";


    export default {
        components: {
            'v-chart': ECharts
        },
        data: function(){
            return {
                choice: 'choice',
                n_decisions:0,
                decisions: [],
                max_choice: 0,
                distinct_event: [],
                distinct_event_temp: [],
                unique_decision_final:[],
                unique_decision_final_temp:[],
                type: null,
                game: null,
                version: null,
                chapter: null,
                barGraph: [],
                barGraphAll: [],
                result: [],
                chapters: [],
                filterStudent: [],
                GroupFilter: [],
                getFilterHeader: null,
                choiceList: null,
                choiceList_name: null,
                emotion:[{'positive':['Call a friend','Read','calmDown','Calm down', 'Quite a lot','A lot', 'happy','playComputer', 'Play with the computer','Happiness'],'negative':['A little','Attack the toy', 'sad','Sadness','Boringness','Complain','Complain about the director','Shout at her','Attack the toy','Anger','angry'], 'neutral':['Girl','Boy','Yes','No','indifferent','Leave the scene','Rest','youAreAToy','No. You are a toy!','doNotKnow'], 'complex':['no', 'Remain silent','silence','Run']}],

                d_count:[],
                new_score:[],
                all_final_data:[],
                d_c:[],
            }
        },
        mounted(){

          this.game = 1;
          // this.chapter = 1593;
          this.version = 1;

          const requestOne = axios.get("descriptions/games");
          const requestTwo = axios.get("filters/group");
          const requestThree = axios.get("filters/student");


          axios.all([requestOne, requestTwo, requestThree ]).then(axios.spread((...responses) => {
            this.$store.state.games = responses[0].data.games;
            this.GroupFilter = responses[1].data
            this.filterStudent = responses[2].data

            this.groupsList = this.GroupFilter.group_ids;
            this.SelectGroupOne = this.$route.params.groupOne;
            this.SelectGroupTwo = this.$route.params.groupTwo;
            this.loadData();

          })).catch(errors => {
            console.log(errors);
          })

          this.$root.$on('loadFilterHeaderSingle', (Filter) => { 
            this.getFilterHeader = {};
            if(Filter.group !== null && Filter.group !== undefined) {
              if(Filter.student === null )
                delete  Filter.student;

              this.getFilterHeader = Filter;

            } else if(Filter.student !== null && Filter.student !== undefined) {

              if(Filter.group === null )
                delete  Filter.group;

              this.getFilterHeader = Filter;
            }


          });

           var l=[1593,2377];
           l.forEach(x => this.submitData(x));

        }
        ,
        methods: {

            exportOpt(){
                window.print();
            },
            home(){
                this.$router.push('/main');
            },

            filter(){

              this.$root.$emit('loadFilterDate', [this.GroupFilter, this.filterStudent, "single"]);
              this.$modal.show('filter');

            },
 
 
            submitData(item){

                this.loading = true;
                if(this.getFilterHeader !== null && JSON.stringify(this.getFilterHeader) !== JSON.stringify({})) {

                  axios.get("decision?gameCode=" + this.game + "&gameVersion=" + this.version + "&chapterCode=" + item, {headers: {filters: JSON.stringify(this.getFilterHeader)}}).then(res => {
                    this.decisions = res.data.decisions;


                    if(this.decisions.length>0){

                    this.dataAnalysis();
                    this.d_counts(item);
       
                    }

                  });

                }else {
                  axios.get("decision?gameCode=" + this.game + "&gameVersion=" + this.version + "&chapterCode=" + item).then(res => {
                    this.decisions = res.data.decisions;

                    if(this.decisions.length>0){

                    this.dataAnalysis();
                    this.d_counts(item);             

                    }
  
                  });
                }
            },
            loadData(){
                let key = this.game;
                this.result = this.$store.state.games;

                let res = this.result.filter(
                    function(data){
                        if(data.gameCode === key)
                            return data
                    }
                );
                if(res.length > 0){
                    res = res[0];
                    this.chapters = res.chapters;
                    this.version = res.gameVersion;
                }

            },
            dataAnalysis(){
                this.distinct_event_temp = [];
                this.unique_decision_final_temp = [];
                this.unique_decision_final = [];
                var eventList = [];
                var choice = [];

                this.decisions.forEach(value => {

                  if(value.eventType === 'timed'){
                    this.unique_decision_final_temp.push([value.eventCode, value.choice, value.eventDescription]);
                    let t = 0;
                    for( ; t < this.distinct_event_temp.length; t++){
                      if(this.distinct_event_temp[t].value === value.eventCode){
                        break;
                      }
                    }
                    if(this.distinct_event_temp.length === t)
                      this.distinct_event_temp.push({value: value.eventCode, description: value.eventDescription});


                  }else{
                    let i=0;
                    for (; i < eventList.length; i++){


                      if(eventList[i].value === value.eventCode) {
                        eventList[i].count++;
                        break;
                      }
                    }
                    if(i === eventList.length)
                      eventList.push({value: value.eventCode, count: 1, description: value.eventDescription});


                    let p = 0;
                    for (; p < choice.length; p++){
                      if(choice[p].value === value.choice)
                        break;
                    }
                    if(p === choice.length)
                      choice.push({value: value.choice, choice: []});

                  }

                });

              choice.forEach(value => {
                eventList.forEach(value1 => {
                  value.choice.push({
                    value: 0,
                    name: value.value,
                    count: 0,
                    event: value1.value,
                    description: value1.description
                  });
                });

              });


              this.decisions.forEach(value => {
                if(value.eventType !== 'timed'){
                  choice.forEach(value1 => {
                    let x = 0;
                    for(; x < value1.choice.length; x++){
                      if(value1.choice[x].event === value.eventCode && value1.choice[x].name === value.choice)
                        value1.choice[x].count++;

                    }

                  });
                }

              });


              choice.forEach(value1 => {

                value1.choice.forEach(value2 => {

                  let m =0;
                  for(; m < eventList.length; m++){
                    if(eventList[m].value === value2.event){
                      value2.value = (value2.count/eventList[m].count)*100;
                    }
                  }

                });

              });
              this.unique_decision_final = choice;
              this.choiceList = eventList;
            },

      
            d_counts(item){
                  this.unique_decision_final.forEach(( message) =>{   
                     message.choice.forEach(( new_m) =>{
                        if (new_m.value > 0) {
                            this.d_count.push(new_m);
                          }      
                    });

                  });


                  this.d_count.sort((a, b) => a.event - b.event);

                  var total_answer=0;
                  var tpo=0, tne=0, tnu=0,tcom=0;
                  this.d_count.forEach((m) =>{total_answer+=m.count; });
                  this.d_count.forEach((m) =>{
                              var po=0, ne=0, nu=0, com=0;

                              

                              this.emotion[0].positive.forEach((el) =>{ if(el == m.name){ po=(m.count/total_answer)*100; tpo+=po;}});
                              this.emotion[0].negative.forEach((el) =>{ if(el == m.name){ ne=(m.count/total_answer)*100; tne+=ne;}});
                              this.emotion[0].neutral.forEach((el) =>{ if(el == m.name){ nu=(m.count/total_answer)*100; tnu+=nu;}});
                              this.emotion[0].complex.forEach((el) =>{ if(el == m.name){ com=(m.count/total_answer)*100; tcom+=com;}});
                              

                              const record = this.new_score.find(element => element.event == m.event);

                              if (record) {
                                  record.po += po;
                                  record.ne += ne;
                                  record.nu += nu;
                                  record.com += com;
                                  record.choice.push({c:m.name,n:m.count});
                              } else {
                                  this.new_score.push({
                                      event: m.event,
                                      event_des: m.description,
                                      po:po,
                                      ne:ne,
                                      nu:nu,
                                      com:com,
                                      choice:[{c:m.name,n:m.count}],
                                  });     
                              }
                
                          });
                  this.n_decisions+=total_answer;

                  this.all_final_data.push({eventid:item,total_decision:total_answer, sentiment_score:this.new_score, total_sentiment_score: [tpo,tne,tnu,tcom] });
                  this.barChartTotalScore(tpo,tne,tnu,tcom);
                  this.barAllScore(this.new_score);
                  this.total_score=[];
                  this.new_score=[];
                  this.unique_decision_final=[];
                  this.d_count=[];

                  


                  
            },


            barChartTotalScore(tpo,tne,tnu,tcom){
                this.barGraph.push( {
                                 grid: {
                                      left: '3%',
                                      right: '4%',
                                      bottom: '3%',
                                      containLabel: true
                                  },

                                xAxis: {
                                    type: 'category',
                                    data: ['Positive', 'Negative', 'Complex','Neutral']
                                },
                                yAxis: {
                                    type: 'value'
                                },
                                series: [{
                                    data: [tpo,tne,tcom,tnu],
                                    type: 'bar'
                                }]});

            },


              barAllScore(score){

                 const title=[];
                 const postive=[];
                 const negative=[];
                 const complex=[];
                 const neutral=[];

                 score.forEach((m) =>{ title.push(m.event_des); postive.push(m.po);negative.push(m.ne);complex.push(m.com);neutral.push(m.nu);});

                  this.barGraphAll.push({ 

                          tooltip: {
                              trigger: 'axis',
                              axisPointer: {
                                  type: 'shadow'
                              }
                          },

                          legend: {
                              data: ['positive', 'negative','complex','neutral']
                          },
                          grid: {
                              left: '3%',
                              right: '3%',
                              bottom: '0%',
                              containLabel: true
                          },
                          xAxis: {
                              type: 'value',
                              boundaryGap: [0, 0.01]
                          },
                          yAxis: {
                              type: 'category',
                              data: title
                          },
                          series: [
                              {
                                  name: 'positive',

                                  type: 'bar',
                                  data: postive
                              },
                              {
                                  name: 'negative',
                                  type: 'bar',
                                  data: negative
                              },
                              {
                                  name: 'complex',
                                  type: 'bar',
                                  data: complex
                              },
                              {
                                  name: 'neutral',
                                  type: 'bar',
                                  data: neutral
                              }

                          ]});
            },



        }
    }
</script>

<style scoped lang="scss">
    .title{
      font-weight: bold;
      font-size: 18px;
      color: #e35219;

    }
    .sub-title{
      color: #e35219;
      margin: 26px 0 6px;
    }
    .btn-success, .btn-dark{
        padding: 4px 19px;
        border: 0;
    }
    .btn-primary{
        padding: 4px 19px;
        background: #e35219;
        border: 0;
    }
    .custom-select{
        text-transform: capitalize;
        font-weight: bold;
    }
    .content{
        margin: 50px 0;
    .custom-control{
        margin-top: 16px;
    }
    }
    .eventlist{
        list-style: none;
        padding: 0;
        li {
            padding: 1px 10px;
            background: #325973;
            color: white;
            border: 1px solid white;
            cursor: pointer;
        }
        li:hover{
            background: #848484;
        }
    }
    
    .echarts {width: 100% !important;height: 400px!important; }



    h2.title[data-v-16f032e2] {
    font-family: serif;
    color: black;
    font-weight: bold;
    }

    h3.title {
    text-align: center;
    font-family: serif;
    color: black!important;
    font-size: 29px!important;
    padding-bottom: 26px;
    }

    p.body {
    font-family: serif;
    font-size: 17px;
    text-align: justify;
    }

    .table th, .table td {
    padding: 2px 5px 2px 10px;
    vertical-align: top;
    }

    tbody {
    font-size: 15px;
    font-family: serif;
    }

    p.fig {
    font-family: serif;
    font-style: italic;
    text-align: center;
    font-weight: bold;
   }

   li.nostyle {
    list-style: none;
  }

  p.nb {
      font-size: 12px;
      font-family: serif;
      font-style: italic;
      font-weight: bold;
  }
</style>