<template>
  <el-container style="height: 100vh;padding:1% 3% 0 3%">


    <el-aside style="padding: 20px 30px 0 0">
      <h3>{{this.$t('config')}}</h3>

      <div class="menu">
        {{this.$t('client_num')}}
        <el-input-number
          v-model="clientNum"
          controls-position="right"
          :min="1"
          :max="20"
          size="mini"
          @change="setprocess(0)"
          :disabled="isTraining"
        ></el-input-number>
      </div>

      <div class="menu">
        {{this.$t('dataset')}}
        <el-select v-model="datasetvalue"
                   :placeholder="this.$t('select_a_dataset')"
                   size="mini"
                   @change="show = true; setprocess(0);loadfeature();loadtarget()"
                   :disabled="isTraining">
          <el-option
            v-for="item in datasetoptions"
            :key="item.value"
            :label="item.label"
            :value="item.value"
            :disabled="item.disabled"
            >
          </el-option>
        </el-select>
      </div>

      <el-divider></el-divider>

      <h3>{{this.$t('post_task')}}</h3>

      <div class="menu">
        {{this.$t('model')}}
        <el-select v-model="modelvalue"
                   :placeholder="this.$t('select_a_model')"
                   size="mini"
                   @change="setprocess(1)"
                   :disabled="isTraining">
          <el-option-group
            v-for="group in modeloptions"
            :key="group.label"
            :label="group.label">
            <el-option
              v-for="item in group.options"
              :key="item.value"
              :label="item.label"
              :value="item.value"
              :disabled="item.disabled">
            </el-option>
          </el-option-group>
        </el-select>
      </div>

      <div class="menu">
        {{this.$t('loss function')}}
        <el-select v-model="lossfuncvalue"
                   :placeholder="this.$t('select_a_loss_function')"
                   size="mini"
                   @change="setprocess(1)"
                   :disabled="isTraining">
            <el-option
              v-for="item in lossfuncoptions"
              :key="item.value"
              :label="item.label"
              :value="item.value"
              :disabled="item.disabled">
            </el-option>
        </el-select>
      </div>

      <div class="menu">
        <span>{{this.$t('train_time')}}</span>
        <el-input-number v-model="traintime"
                         :min="10"
                         :max="60"
                         :step="5"
                         size="mini"
                         @focus="setprocess(1)"
                         :disabled="isTraining">
        </el-input-number>
      </div>


      <!-- <el-divider></el-divider> -->

      <!-- <h3>{{this.$t('select features')}}</h3> -->
      <div v-show="show">
        <el-divider></el-divider>
        <div style="overflow-y: scroll; height:150px;">
          <h3>{{this.$t('select features')}}</h3>
          <div v-for="feature in features" :key="feature">
            <input type="checkbox" :id="feature" :value="feature" v-model="selectedfeatures">
            <label :for="feature">{{ feature }}</label>
          </div>
          <span>Checked names: {{ selectedfeatures }}</span>
        </div>
      </div>

      <!-- <h3>{{this.$t('select target')}}</h3> -->
      <div v-show="show">
        <el-divider></el-divider>
        <div style="overflow-y: scroll; height:150px;">
          <h3>{{this.$t('select target')}}</h3>
          <div v-for="target in targets" :key="target">
            <input type="checkbox" :id="target" :value="target" v-model="selectedtargets">
            <label :for="target">{{ target }}</label>
          </div>
          <span>Checked names: {{ selectedtargets }}</span>
        </div>
      </div>

      <el-divider></el-divider>

      <h3>{{this.$t('train_process')}}</h3>

      <div style="text-align: center;margin-top: 10px">
        <el-progress :percentage="currentpercentage" :status="currentstatus"></el-progress>
      </div>

      <div style="text-align: center;margin-top: 12px">
        <el-button type="success"
                   v-text="btnMsg"
                   @click="startTrain()"
                   size="mini"
                   :disabled="isTraining">
        </el-button>
      </div>
    </el-aside>

    <el-container style="padding: 20px 50px">
      <!-- <el-header height="130px">
        <h3>{{this.$t('process')}}</h3>
        <div>
          <el-steps :active=activestep finish-status="success" align-center>
            <el-step :title="this.$t('pro1')" description=""></el-step>
            <el-step :title="this.$t('pro2')" description=""></el-step>
            <el-step :title="this.$t('pro3')" description=""></el-step>
            <el-step :title="this.$t('pro4')" description=""></el-step>
            <el-step :title="this.$t('pro5')" description="" icon="el-icon-finished"></el-step>
          </el-steps>
        </div>
        <el-divider></el-divider>
      </el-header> -->
      <el-main style="border: lightgray dashed 1px;border-radius: 10px;background-color: #f7f8fb"
               v-loading="this.state==='synchronizing...'" element-loading-text="synchronizing...">
        <div style="display: flex;justify-content: flex-start;align-content: flex-start;flex-wrap: wrap">
          <Client v-for="(client,index) in clientList" :key="index" :id=index :state="state"
                  :readonly="isTraining" :result="result" :reward="reward" :clientnum="clientNum"
                  :blockheight="blockheight"></Client>
        </div>
      </el-main>

    </el-container>

  </el-container>
</template>

<script>
  import Client from "../components/Client";
  import {run} from '~/assets/js/script.js'

  export default {
    name: "home",
    components: {
      Client
    },
    data() {
      return {

        activestep: 0,

        //client
        clientNum: 2,
        clientList: [],

        //model
        modeloptions: [{
          label: 'Neural Network',
          options: [{
            value: 'NN',
            label: 'NN'
          }, {
            value: 'CNN',
            label: 'CNN'
          }, {
            value: 'RNN',
            label: 'RNN',
            // disabled: true
          }],
        }, {
          label: 'Linear Models',
          options: [{
            value: 'Linear-Regression',
            label: 'Linear Regression',
            // disabled: true
          }, {
            value: 'Logistic-Regression',
            label: 'Logistic Regression',
            // disabled: true
          }],
        }, {
          label: 'Decision Trees',
          options: [{
            value: 'Classification-Decision-Tree',
            label: 'Classification Decision Tree',
            // disabled: true
          }, {
            value: 'Regression-Decision-Tree',
            label: 'Regression Decision Tree',
            // disabled: true
          }]
        }, {
          label: 'Support Vector Machines',
          options: [{
            value: 'SVR',
            label: 'SVR',
            // disabled: true
          }, {
            value: 'SVC',
            label: 'SVC',
            // disabled: true
          }]
        }
        ],
        modelvalue: '',

        //loss function
        lossfuncoptions: [
          {
            value: 'mean_squared_error',
            label: 'mean_squared_error'
          },
          {
            value: 'mean_absolute_error',
            label: 'mean_absolute_error'
          },
          {
            value: 'mean_absolute_percentage_error',
            label: 'mean_absolute_percentage_error'
          },
          {
            value: 'mean_squared_logarithmic_error',
            label: 'mean_squared_logarithmic_error'
          },
          {
            value: 'squared_hinge',
            label: 'squared_hinge'
          },
          {
            value: 'hinge',
            label: 'hinge'
          },
          {
            value: 'logcosh',
            label: 'logcosh'
          }
        ],
        lossfuncvalue: '',

        //dataset
        datasetoptions: [
          {
            value: 'dataset1',
            label: 'dataset1'
          },
          {
            value: 'dataset2',
            label: 'dataset2'
          }
        ],
        datasetvalue: '',
        // isdatasetselected: false,

        reward: 1000,

        traintime: 30,

        btnMsg: 'Run',

        state: 'free',

        isTraining: false,

        currentpercentage: 0,

        currentstatus: null,

        result: null,

        updatetime: '',

        blockheight: 1023,

        show: false,
        features: [],
        targets: [],
        selectedfeatures: [],
        selectedtargets: []
      }
    },
    computed: {
      totalpower() {
        let a = this.$store.state.powerList
        let sum = 0
        for (let i = 0; i < a.length; i++) {
          sum += a[i].power
        }
        return sum
      },
    },
    created() {
      this.initClient()

      var time = new Date();
      var y = time.getFullYear();
      var m = time.getMonth() + 1;
      var d = time.getDate();
      var h = time.getHours();
      var mm = time.getMinutes();
      var s = time.getSeconds();

      this.updatetime = this.getcurrenttime()
    },
    watch: {
      clientNum: function () {
        this.initClient()
      },
      result: function () {
      }
    },
    methods: {
      add0(m) {
        return m < 10 ? '0' + m : m
      },

      getcurrenttime() {
        var time = new Date();
        var y = time.getFullYear();
        var m = time.getMonth() + 1;
        var d = time.getDate();
        var h = time.getHours();
        var mm = time.getMinutes();
        var s = time.getSeconds();

        return y + '-' + this.add0(m) + '-' + this.add0(d) + ' ' + this.add0(h) + ':' + this.add0(mm) + ':' + this.add0(s)
      },

      initClient() {
        this.clientList = new Array(this.clientNum)
      },
      setprocess(step) {
        this.activestep = step
      },
      startTrain() {
        if (this.datasetvalue === '') {
          this.$message.error('Please select a dataset.');
          return
        }

        if (this.modelvalue === '') {
          this.$message.error('Please select a model.');
          return
        }

        this.setprocess(2)
        this.btnMsg = 'Running'
        this.state = 'training...'
        this.isTraining = true
        this.currentpercentage = 0
        this.currentstatus = null

        let interval = setInterval(() => {
          this.currentpercentage += 1
          if (this.currentpercentage === 100) {
            clearInterval(interval)
            this.currentstatus = 'success'
            // this.state = 'mining...'
            this.state = 'training...'
            this.setprocess(3)
            this.result = run()

            this.$notify({
              title: 'Message',
              message: 'Train Finished!',
              type: 'success'
            })
            this.startMining()
          }
        }, this.traintime * 1000 / 100)

      },

      startMining() {
        setTimeout(() => {
          this.$notify({
            title: 'Message',
            message: 'Mine Finished!',
            type: 'success'
          })
          this.btnMsg = 'Run'
          this.isTraining = false
          this.state = 'synchronizing...'
          this.setprocess(4)
          setTimeout(() => {
            this.state = 'finish'
            this.blockheight += 1
            this.updatetime = this.getcurrenttime()
            this.$notify({
              title: 'Message',
              message: 'Synchronize Finished!',
              type: 'success'
            })
          }, 5000)
        }, 10000)
      },
      loadfeature() {
        // this.isdatasetselected = true
        if (this.datasetvalue === 'dataset1') {
          this.selectedfeatures = []
          this.features = ['group','month','weekday','day','dayofyear','holiday','movieId','20','userId','pct','20_label']
        }
        else if (this.datasetvalue === 'dataset2') {
          this.selectedfeatures = []
          this.features = ['group1','month1','weekday1','day1','dayofyear1']
        }
      },
      loadtarget() {
        if (this.datasetvalue === 'dataset1') {
          this.selectedtargets = []
          this.targets = ['group','month','weekday','day','dayofyear','holiday','movieId','20','userId','pct','20_label']
        }
        else if (this.datasetvalue === 'dataset2') {
          this.selectedtargets = []
          this.targets = ['group1','month1','weekday1','day1','dayofyear1']
        }
      }
    }
  }
</script>

<style scoped>
  .menu {
    margin-top: 12px;
    display: flex;
    justify-content: space-between;
    align-items: center
  }

</style>
