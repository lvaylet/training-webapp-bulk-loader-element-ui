<template>
  <div>
    <section class="section" v-if="classRequests.length==0">
      <div class="container">
        <div class="content">
          <h1 class="title">Instructions</h1>
          <p>Click the button below to manually create a class request from scratch (recommended when the volume of class requests is low):</p>
          <el-button
            type="success"
            size="large"
            icon="plus"
            @click="addClassRequest"
          >
            Add class request
          </el-button>
          <p>or upload an existing CSV file containing class requests:</p>
          <!-- FIXME Save action URL in data() -->
          <!-- FIXME Use axios for proper CORS, or use `headers` attribute -->
          <el-upload
            drag
            accept=".csv"
            :action="actionUploadOdtRequests"
            :before-upload="beforeClassRequestsUpload"
            :on-success="loadClassRequests"
            :show-file-list="false"
            :headers="{ 'Access-Control-Allow-Origin': '*' }"
          >
            <i class="el-icon-upload"></i>
            <div class="el-upload__text">Drop CSV file here or <em>click to upload</em></div>
            <div class="el-upload__tip" slot="tip">CSV files only, with a size less than 100KB</div>
          </el-upload>
        </div>
      </div>
    </section>

    <section class="section" v-else>
      <!-- Instructions -->
      <div class="container">
        <div class="content">
          <el-alert
            title="Instructions"
            type="info"
            show-icon
          >
            <ul>
              <li>Click a field or the <el-button type="primary" size="small" icon="edit"></el-button> button to edit a request.</li>
              <li>Use the <el-button type="success" size="small" icon="plus"></el-button> and <el-button type="danger" size="small" icon="minus"></el-button> buttons to add or remove requests.</li>
            </ul>
          </el-alert>
        </div>
      </div>

      <div class="content">
        <el-table :data="classRequests" :row-class-name="tableRowClassName" >
          <el-table-column width="70">
            <template scope="scope">
              <el-tooltip :content="scope.row.errorMessage" placement="bottom" effect="dark" :enterable="false" > <i class="el-icon-warning" v-show="scope.row.errorMessage"></i></el-tooltip>
            </template>
          </el-table-column>
          <el-table-column label="First Name">
            <template scope="scope">
              <span class="clickable" @click="editClassRequest(scope.row)">{{scope.row.firstName || '-'}}</span>
            </template>
          </el-table-column>
          <el-table-column label="Last Name">
            <template scope="scope">
              <span class="clickable" @click="editClassRequest(scope.row)">{{scope.row.lastName || '-'}}</span>
            </template>
          </el-table-column>
          <el-table-column label="Email">
            <template scope="scope">
              <span class="clickable" @click="editClassRequest(scope.row)">{{scope.row.email || '-'}}</span>
            </template>
          </el-table-column>
          <el-table-column label="Data Center">
            <template scope="scope">
              <span class="clickable" @click="editClassRequest(scope.row)">{{scope.row.dataCenter || '-'}}</span>
            </template>
          </el-table-column>
          <el-table-column label="Keyboard Layout">
            <template scope="scope">
              <span class="clickable" @click="editClassRequest(scope.row)">{{scope.row.keyboardLayout || '-'}}</span>
            </template>
          </el-table-column>
          <el-table-column label="Start Date">
            <template scope="scope">
              <span class="clickable" @click="editClassRequest(scope.row)">{{scope.row.startDate || '-'}}</span>
            </template>
          </el-table-column>
          <el-table-column label="Course Name">
            <template scope="scope">
              <span class="clickable" @click="editClassRequest(scope.row)">{{scope.row.courseName || '-'}}</span>
            </template>
          </el-table-column>
          <el-table-column label="Operations">
            <template scope="scope">
              <el-button-group>
                <el-button type="success" size="small" icon="plus"  @click="addClassRequest(scope.$index)"></el-button>
                <el-button type="danger"  size="small" icon="minus" @click="removeClassRequest(scope.$index)"></el-button>
                <el-button type="primary" size="small" icon="edit"  @click="editClassRequest(scope.row)"></el-button>
              </el-button-group>
            </template>
          </el-table-column>
        </el-table>
      </div>

      <div class="container">
        <div class="content">
          <h1 class="title">Class Settings</h1>
          <div class="columns">
            <div class="column is-half">
              <el-form
                :model="generalSettingsForm"
                :rules="generalSettingsRules"
                ref="generalSettingsForm"
                :label-width="generalSettingsForm.settings.labelWidth"
              >
                <el-form-item label="Company" prop="company">
                  <el-input v-model="generalSettingsForm.company" placeholder="Air France, Capgemini, Talend..." ></el-input>
                <!--
                  <el-select v-model="generalSettingsForm.company" clearable filterable placeholder="Air France, Capgemini, Talend...">
                    <el-option v-for="item in generateOptionsFromPicklist(picklists.accounts)" :key="item.value" :label="item.label" :value="item.value"></el-option>
                  </el-select>
                -->
                </el-form-item>
                <el-form-item label="Quote Number" prop="quoteNumber">
                  <el-input v-model="generalSettingsForm.quoteNumber" placeholder="QU-12345..." ></el-input>
                </el-form-item>
                <el-form-item label="Training Administrator" prop="trainingAdmin">
                  <el-select v-model="generalSettingsForm.trainingAdmin" clearable filterable placeholder="Select one">
                    <el-option v-for="item in generateOptionsFromPicklist(picklists.trainingAdmins)" :key="item.value" :label="item.label" :value="item.value"></el-option>
                  </el-select>
                </el-form-item>
                <el-form-item label="Cc" prop="ccAddress">
                  <el-input v-model="generalSettingsForm.ccAddress" placeholder="manager@company.com"></el-input>
                </el-form-item>
                <el-form-item label="Prefix" prop="prefix">
                  <el-input v-model="generalSettingsForm.prefix" placeholder="[FREE CLASS], [SKO 2017]..."></el-input>
                </el-form-item>
                <el-form-item label="Lifespan" prop="lifespan">
                  <el-input v-model.number="generalSettingsForm.lifespan" placeholder="30, 14, 1..."></el-input>
                </el-form-item>
                <el-form-item>
                  <el-button type="primary" @click="submitForm('generalSettingsForm')" :loading="loadingToSalesforce"><i class="el-icon-upload"></i> Load to Salesforce</el-button>
                  <el-button @click="resetForm('generalSettingsForm')">Reset</el-button>
                </el-form-item>
              </el-form>
            </div>
          </div>
        </div>
      </div>
    </section>

    <el-dialog title="Class Request" :visible.sync="editForm.visible">
      <el-form :model="classRequestBeingEdited" :label-width="editForm.settings.labelWidth">
        <el-form-item label="First name">
          <el-input v-model="classRequestBeingEdited.firstName" auto-complete="off" ></el-input>
        </el-form-item>
        <el-form-item label="Last name">
          <el-input v-model="classRequestBeingEdited.lastName" auto-complete="off" ></el-input>
        </el-form-item>
        <el-form-item label="Email">
          <el-input v-model="classRequestBeingEdited.email" auto-complete="off" ></el-input>
        </el-form-item>
        <el-form-item label="Data Center">
          <el-select v-model="classRequestBeingEdited.dataCenter" clearable filterable placeholder="Select">
            <el-option v-for="item in generateOptionsFromPicklist(picklists.dataCenters)" :key="item.value" :label="item.label" :value="item.value"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="Keyboard Layout">
          <el-select v-model="classRequestBeingEdited.keyboardLayout" clearable filterable placeholder="Select">
            <el-option v-for="item in generateOptionsFromPicklist(picklists.keyboardLayouts)" :key="item.value" :label="item.label" :value="item.value"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="Start Date">
          <el-date-picker v-model="classRequestBeingEdited.startDate" type="date" placeholder="Pick a day"></el-date-picker>
        </el-form-item>
        <el-form-item label="Course Name">
          <el-select v-model="classRequestBeingEdited.courseName" clearable filterable placeholder="Select">
            <el-option v-for="item in generateOptionsFromPicklist(picklists.courseNames)" :key="item.value" :label="item.label" :value="item.value"></el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="toggleEditForm">Cancel</el-button>
        <el-button type="primary" @click="editClassRequestDone(classRequestBeingEdited)">Confirm</el-button>
      </span>
    </el-dialog>

    <pre>{{ $data }}</pre>
  </div>
</template>

<script>
import axios from 'axios'
import moment from 'moment'
import _ from 'lodash'

const TRAINING_OPS_REST_API = 'http://10.42.100.179:5050/api'
const UPLOAD_ODT_REQUESTS = `${TRAINING_OPS_REST_API}/upload-odt-requests`
const LOAD_TO_SALESFORCE = `${TRAINING_OPS_REST_API}/load-odt-requests-to-salesforce`
const HTTP_REST_API = axios.create({
  baseURL: TRAINING_OPS_REST_API,
  headers: {
    'Access-Control-Allow-Origin': '*' // for CORS to work properly
  }
})

function logAxiosError (error) {
  if (error.response) {
    // The request was made and the server responded with a status code that
    // falls out of the range of 2xx
    console.log(error.response.data)
    console.log(error.response.status)
    console.log(error.response.headers)
  } else if (error.request) {
    // The request was made but no response was received. `error.request` is an
    // instance of XMLHttpRequest in the browser and an instance of
    // http.ClientRequest in node.js.
    console.log(error.request)
  } else {
    // Something happened in setting up the request that triggered an error
    console.log('Error', error.message)
  }
  console.log(error.config)
}

function validateQuoteNumber (rule, value, callback) {
  // Quote number can be empty or null
  // When not empty, quote number must be formatted like QU-12345
  return !value || value === '' || (value && /^QU-[0-9]+$/.test(value))
    ? callback()
    : callback(new Error('Please specify a valid quote number (like QU-12345)'))
}

export default {
  name: 'hello',
  data: () => ({
    classRequests: [],
    classRequestBeingEdited: {},
    classRequestCached: {},
    picklists: {
      courseNames: [],
      dataCenters: [],
      keyboardLayouts: [],
      accounts: [], // all 40000+ accounts in Salesforce (not suitable for display in a picklist)
      accountsFiltered: [], // accounts filtered with remote search (suitable for display in a picklist)
      trainingAdmins: []
    },
    editForm: {
      visible: false,
      settings: {
        labelWidth: '120px'
      }
    },
    generalSettingsForm: {
      company: '',
      quoteNumber: '',
      trainingAdmin: '',
      ccAddress: '',
      prefix: '',
      lifespan: 30,
      settings: {
        labelWidth: '180px'
      }
    },
    generalSettingsRules: {
      company: [
        {
          required: true,
          message: 'Please specify the name of the company requesting the above classes',
          trigger: 'blur'
        }
      ],
      quoteNumber: [
        {
          validator: validateQuoteNumber,
          trigger: 'blur'
        }
      ],
      trainingAdmin: [
        {
          required: true,
          message: 'Please select a Training Administrator',
          trigger: 'blur'
        }
      ],
      ccAddress: [
        {
          type: 'email',
          message: 'Please input a correct email address',
          trigger: 'blur'
        }
      ],
      lifespan: [
        {
          type: 'number',
          required: true,
          message: 'Please specify a numeric lifespan for the lab environment (in days)',
          trigger: 'blur'
        }
      ]
    },
    // Communications with REST API
    loadingToSalesforce: false,
    actionUploadOdtRequests: UPLOAD_ODT_REQUESTS
  }),
  methods: {
    populatePicklist: function (picklist, url) {
      HTTP_REST_API.get(url)
        .then(response => { this.picklists[picklist] = response.data })
        .catch(e => logAxiosError(e))
    },

    addClassRequest: function (index = 0) {
      // FIXME Declare and initialize default fields in data() instead?
      this.classRequests.splice(index + 1, 0, {
        firstName: null,
        lastName: null,
        email: null,
        dataCenter: null,
        keyboardLayout: null,
        startDate: null,
        courseName: null
      })
    },

    beforeClassRequestsUpload: function (file) {
      const isCSV = file.type === 'text/csv' || file.type === 'application/vnd.ms-excel'
      const isLt100K = file.size / 1024 < 100

      if (!isCSV) {
        this.$notify.error({
          title: 'Error',
          message: 'File must be CSV format!'
        })
      }
      if (!isLt100K) {
        this.$notify.error({
          title: 'Error',
          message: 'File size can not exceed 100KB!'
        })
      }
      return isCSV && isLt100K
    },

    loadClassRequests: function (response, file, fileList) {
      this.classRequests = response
      this.$notify.success({
        title: 'Success',
        message: 'Class requests have been loaded successfully.'
      })
    },

    removeClassRequest: function (index) {
      this.classRequests.splice(index, 1)
    },

    editClassRequest: function (classRequest) {
      this.classRequestBeingEdited = _.cloneDeep(classRequest)
      this.classRequestCached = classRequest
      this.toggleEditForm()
    },

    editClassRequestDone: function (classRequest) {
      this.toggleEditForm()
      // Convert start date to YYYY-MM-DD before updating the class request (only if date has been set/changed)
      if (classRequest['startDate']) {
        classRequest['startDate'] = moment(classRequest['startDate']).format('YYYY-MM-DD')
      }
      // Replace class request with edited one
      var index = _.indexOf(this.classRequests, this.classRequestCached)
      this.classRequests.splice(index, 1, classRequest)
    },

    toggleEditForm: function () {
      this.editForm.visible = !this.editForm.visible
    },

    handleRowClick: function (row, event, column) {
      this.editClassRequest(row)
    },

    tableRowClassName: function (row, index) {
      return _.has(row, 'errorMessage')
        ? 'row-warning'
        : ''
    },

    highlightClassRequest: function (classRequest) {
      return _.has(classRequest, 'errorMessage')
    },

    submitForm: function (formName) {
      this.$refs[formName].validate(valid => {
        if (valid) {
          this.loadingToSalesforce = true
          HTTP_REST_API.post(LOAD_TO_SALESFORCE, {
            classRequests: this.classRequests,
            settings: this.generalSettingsForm
          })
            .then(response => {
              this.loadingToSalesforce = false
              let errors = response.data.errors
              if (_.isEmpty(errors)) {
                this.$alert(
                  'Class requests have been loaded successfully.',
                  'Congratulations',
                  {
                    type: 'success'
                  }
                )
              } else {
                // Reset all error messages by deleting the errorMessage field
                // of all class requests
                _.forEach(this.classRequests, c => {
                  this.$delete(c, 'errorMessage')
                })
                // Add back errorMessage field for class requests with actual error
                _.forIn(errors, (errorMessage, rowIndex) => {
                  // Do not add new field directly, as Vue.js will not pick it
                  // up and make it reactive:
                  // this.classRequests[rowIndex]['errorMessage'] = errorMessage
                  // Instead, use this.$set as described at https://vuejs.org/v2/guide/reactivity.html#Change-Detection-Caveats
                  // (or initialize this field in data() so it has a getter and
                  // a setter from the start)
                  this.$set(this.classRequests[rowIndex], 'errorMessage', errorMessage)
                })
                this.$alert(
                  `Some errors have been detected, so no class requests have been loaded.
                  Please review and fix the highlighted errors, then load the class requests again.`,
                  'Error',
                  {
                    type: 'error'
                  }
                )
              }
            })
            .catch(error => {
              console.log(error)
              this.loadingToSalesforce = false
            })
        }
      })
    },

    resetForm: function (formName) {
      this.$refs[formName].resetFields()
    },

    // TODO Transform picklists into computed properties (picklistsAsOptions?)
    generateOptionsFromPicklist: function (picklist) {
      return _.map(picklist, function (e) {
        return { 'label': e, 'value': e }
      })
    }
  },
  created () {
    this.populatePicklist('courseNames', 'salesforce/course-names?active=true')
    this.populatePicklist('dataCenters', 'salesforce/data-centers')
    this.populatePicklist('accounts', 'salesforce/accounts')
    this.populatePicklist('keyboardLayouts', 'salesforce/keyboard-layouts')
    this.populatePicklist('trainingAdmins', 'salesforce/training-admins')
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.row-error {
  background: #ff3860;
}

.row-warning {
  background: #ffdd57;
}

tbody span.clickable {
  cursor: pointer;
}
</style>
