<template>
<div class="view">
  <Panel shadow :padding="10">
    <div slot="title">
      {{title}}
    </div>
    <div slot="extra">
      <!-- <Button v-if="listVisible" type="info" @click="init" :loading="btnLoading">{{$t('m.Refresh')}}</Button> -->
      <Button v-if="listVisible" type="info" @click="gao" :loading="btnLoading">{{$t('m.Send')}}</Button>
      <Button v-else type="ghost" icon="ios-undo" @click="goBack">{{$t('m.Back')}}</Button>
      
    </div>
    <div>
    <transition-group name="Clarification-animate" mode="in-out">
      <div class="no-Clarification" v-if="!clarifications.length" key="no-Clarification">
        <p>{{$t('m.No_Clarifications')}}</p>
      </div>
      <template v-if="listVisible">
        <ul class="clarifications-container" key="list">
          <li v-for="Clarification in clarifications" :key="Clarification.messgae">
            <div class="flex-container">
              <div class="message"><a class="entry" @click="goClarification(Clarification)">
                {{Clarification.message}}</a></div>
              <div class="date">{{Clarification.create_time | localtime }}</div>
              <div class="reply">{{Clarification.reply==null?"No reply":"Replied"}}</div>
              <!-- <div class="creator"> By {{Clarification.created_by.username}}</div> -->
            </div>
          </li>
        </ul>
        <Pagination v-if="!isContest"
                    key="page"
                    :total="total"
                    :page-size="limit"
                    @on-change="getClarificationList">
        </Pagination>
      </template>
      <template v-else>
        <!-- <div class="message"> {{Clarification.message}} </div> -->\
        <div class ="clarifiation" key="detail">
          <el-form>
          <el-form-item label="Request:">
            <div v-html="Clarification.message" ></div>
          </el-form-item>
          <el-form-item label="Reply:">
            <div v-katex v-html="Clarification.reply" key="replay" class="content-container markdown-body"></div>
          </el-form-item>
          </el-form>
        </div>
      </template>
    </transition-group>
    </div>
  </Panel>
  <el-dialog title="Send Your Request"
               width="50%"
               :visible.sync="dialogVisible">
      <el-input
        type="textarea"
        :rows="2"
        placeholder="Describe your problem"
        v-model="message"
        maxlength="250"
        rows="5"
        show-word-limit
        >
      </el-input>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="sendClarification" :loading="btnLoading">Send</el-button>
      </span>
    </el-dialog>
</div>
</template>

<script>
  import api from '@oj/api'
  import Pagination from '@oj/components/Pagination'
  // import utils from '@/utils/utils'

  export default {
    name: 'Clarification',
    components: {
      Pagination
    },
    data () {
      return {
        limit: 10,
        total: 10,
        btnLoading: false,
        clarifications: [],
        Clarification: '',
        input: '',
        listVisible: true,
        dialogVisible: false,
        message: ''
        // name: 'Clarification'
      }
    },
    mounted () {
      this.init()
    },
    methods: {
      init () {
        this.getContestClarificationList()
      },
      getContestClarificationList () {
        this.btnLoading = true
        api.getContestClarificationList(this.$route.params.contestID).then(res => {
          this.btnLoading = false
          this.clarifications = res.data.data
        }, () => {
          this.btnLoading = false
        })
      },
      goClarification (Clarification) {
        this.Clarification = Clarification
        this.listVisible = false
        console.log(this.Clarification)
        console.log(this.listVisible)
      },
      goBack () {
        this.listVisible = true
        this.Clarification = ''
      },
      gao () {
        this.dialogVisible = true
      },
      sendClarification () {
        this.btnLoading = true
        let data = {}
        data.contest_id = this.$route.params.contestID
        data.message = this.message
        api.submitClarification(data).then(res => {
          this.btnLoading = false
          // this.clarifications = res.data.data
          if (res.error == null) {
            this.message = ''
            this.init()
          }
        }, () => {
          this.btnLoading = false
        })
        this.dialogVisible = false
      }
    },
    computed: {
      title () {
        if (this.listVisible) {
          return this.isContest ? 'Contest Clarifications' : 'Clarifications'
        } else {
          return this.Clarification.title
        }
      },
      isContest () {
        return !!this.$route.params.contestID
      }
    }
  }
</script>

<style scoped lang="less">
  .clarifications-container {
    margin-top: -10px;
    margin-bottom: 10px;
    li {
      padding-top: 15px;
      list-style: none;
      padding-bottom: 15px;
      margin-left: 20px;
      font-size: 16px;
      border-bottom: 1px solid rgba(187, 187, 187, 0.5);
      &:last-child {
        border-bottom: none;
      }
      .flex-container {
        .title {
          flex: 1 1;
          text-align: left;
          padding-left: 10px;
          a.entry {
            color: #495060;
            &:hover {
              color: #2d8cf0;
              border-bottom: 1px solid #2d8cf0;
            }
          }
        }
        .creator {
          flex: none;
          width: 200px;
          text-align: center;
        }
        .date {
          flex: none;
          width: 200px;
          text-align: center;
        }
      }
    }
  }

  .content-container {
    padding: 0 20px 20px 20px;
  }

  .no-Clarification {
    text-align: center;
    font-size: 16px;
  }changeLocale

  .Clarification-animate-enter-active {
    animation: fadeIn 1s;
  }
</style>
