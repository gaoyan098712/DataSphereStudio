<template>
  <Modal
    v-model="ProjectShow"
    :title="actionType === 'add' ? $t('message.workflow.projectDetail.createProject') : $t('message.workflow.projectDetail.editorProject')"
    :closable="false">
    <Form
      :label-width="100"
      label-position="left"
      ref="projectForm"
      :model="projectDataCurrent"
      :rules="formValid"
      v-if="ProjectShow">
      <FormItem
        :label="$t('message.workflow.projectDetail.projectName')"
        prop="name">
        <Input
          v-model="projectDataCurrent.name"
          :placeholder="$t('message.workflow.enterName')"
          :disabled="actionType === 'modify'"></Input>
      </FormItem>
      <FormItem
        v-if="!framework"
        :label="$t('message.workflow.projectDetail.product')"
        prop="product">
        <Input
          v-model="projectDataCurrent.product"
          :placeholder="$t('message.workflow.enterName')">
        </Input>
      </FormItem>
      <FormItem
        :label="$t('message.workflow.projectDetail.appArea')"
        prop="applicationArea">
        <Select
          v-model="projectDataCurrent.applicationArea"
          :placeholder="$t('message.workflow.projectDetail.selectAppArea')">
          <Option
            v-for="(item, index) in applicationAreaMap"
            :label="item"
            :value="index"
            :key="index"/>
        </Select>
      </FormItem>
      <FormItem
        :label="$t('message.workflow.projectDetail.editPermissions')"
        prop="editUsers">
        <Select
          v-model="projectDataCurrent.editUsers"
          multiple
          filterable
          :placeholder="$t('message.workflow.projectDetail.usersAllowedToEdit')">
          <Option
            v-for="(item, index) in editUsersMap"
            :label="item"
            :value="item"
            :key="index"/>
        </Select>
      </FormItem>
      <FormItem
        :label="$t('message.workflow.projectDetail.viewPermissions')"
        prop="accessUsers">
        <Select
          v-model="projectDataCurrent.accessUsers"
          multiple
          filterable
          :placeholder="$t('message.workflow.projectDetail.usersAllowedToView')">
          <Option
            v-for="(item, index) in accessUsersMap"
            :label="item"
            :value="item"
            :key="index"/>
        </Select>
      </FormItem>
      <!-- <FormItem
        :label="$t('message.workflow.projectDetail.opPermissions')"
        prop="releaseUsers">
        <Select
          v-model="projectDataCurrent.releaseUsers"
          multiple
          filterable
          :placeholder="$t('message.workflow.projectDetail.selectOPUser')">
          <Option
            v-for="(item, index) in accessUsersMap"
            :label="item"
            :value="item"
            :key="index"/>
        </Select>
      </FormItem> -->
      <FormItem :label="$t('message.workflow.projectDetail.devProcess')" prop="devProcessList">
        <CheckboxGroup v-model="projectDataCurrent.devProcessList">
          <Checkbox v-for="item in devProcess" :label="item.dicValue" :key="item.dicKey">
            <SvgIcon class="icon-style" :icon-class="item.icon"/>
            <span>{{item.dicName}}</span>
          </Checkbox>
        </CheckboxGroup>
      </FormItem>
      <FormItem v-if="framework" :label="$t('message.workflow.projectDetail.orchestratorMode')" prop="orchestratorModeList">
        <CheckboxGroup v-model="projectDataCurrent.orchestratorModeList">
          <Checkbox v-for="item in orchestratorModeList.list" :label="item.dicKey" :key="item.dicKey">
            <span class="icon-bar">
              <SvgIcon class="icon-style" :icon-class="item.icon"/>
              <span style="margin-left: 10px">{{item.dicName}}</span>
            </span>
          </Checkbox>
        </CheckboxGroup>
      </FormItem>
      <FormItem
        :label="$t('message.workflow.projectDetail.business')"
        prop="business">
        <we-tag
          :new-label="$t('message.workflow.projectDetail.addBusiness')"
          :tag-list="projectDataCurrent.business"
          @add-tag="addTag"
          @delete-tag="deleteTag"></we-tag>
      </FormItem>
      <FormItem
        :label="$t('message.workflow.projectDetail.projectDesc')"
        prop="description">
        <Input
          v-model="projectDataCurrent.description"
          type="textarea"
          :placeholder="$t('message.workflow.projectDetail.pleaseInputProjectDesc')"></Input>
      </FormItem>
    </Form>
    <div slot="footer">
      <Button
        type="text"
        size="large"
        @click="Cancel">{{$t('message.workflow.cancel')}}</Button>
      <Button
        type="primary"
        size="large"
        @click="Ok">{{$t('message.workflow.ok')}}</Button>
    </div>
  </Modal>
</template>
<script>
import storage from "@/common/helper/storage";
import tag from '@component/tag/index.vue';
import { GetWorkspaceUserList, GetDicList } from '@/common/service/apiCommonMethod.js';
export default {
  components: {
    'we-tag': tag,
  },
  props: {
    projectData: {
      type: Object,
      default: null,
    },
    addProjectShow: {
      type: Boolean,
      default: false,
    },
    actionType: {
      type: String,
      default: '',
    },
    applicationAreaMap: {
      type: Array,
      default: () => []
    },
    framework: {
      type: Boolean,
      default: false
    },
    orchestratorModeList: {
      type: Object,
      default: () => {}
    }
  },
  data() {
    return {
      ProjectShow: false,
      originBusiness: '',
      editUsersMap: [],
      accessUsersMap: [],
      devProcess: [
        {
          id: 1,
          icon: '',
          title: '开发中心',
          checked: false
        },
        {
          id: 2,
          icon: '',
          title: '生产中心',
          checked: false
        }
      ],
      devProcessList: [],
      selectCompiling: []
    };
  },
  computed: {
    projectDataCurrent() {
      return this.projectData;
    },
    formValid() {
      let validateName = (rule, value, callback) => {
        let currentWorkspaceName = storage.get("currentWorkspace") ? storage.get("currentWorkspace").name : null;
        let username = storage.get("baseInfo", 'local') ? storage.get("baseInfo", 'local').username : null;
        if (currentWorkspaceName && username && value.match(currentWorkspaceName) || value.match(username)) {
          callback(new Error(this.$t('message.workflow.projectDetail.validateName')))
        } else {
          callback()
        }
      };
      return {
        name: [
          { required: true, message: this.$t('message.workflow.enterName'), trigger: 'blur' },
          { message: `${this.$t('message.workflow.nameLength')}128`, max: 128 },
          { type: 'string', pattern: /^[a-zA-Z][a-zA-Z0-9_]*$/, message: this.$t('message.workflow.validNameDesc'), trigger: 'blur' },
          { validator: validateName, trigger: 'blur' }
        ],
        description: [
          { required: true, message: this.$t('message.workflow.projectDetail.pleaseInputProjectDesc'), trigger: 'blur' },
        ],
        product: [
          { required: true, message: this.$t('message.workflow.projectDetail.selectProduct'), trigger: 'change' },
        ],
        applicationArea: [
          { required: true, message: this.$t('message.workflow.projectDetail.selectAppArea'), trigger: 'change', type: 'number' },
        ],
        devProcessList: [
          { required: true, message: this.$t('message.workflow.projectDetail.pleaseSelect'), trigger: 'blur', type: "array" },
        ],
        orchestratorModeList: [
          { required: true, message: this.$t('message.workflow.projectDetail.pleaseSelect'), trigger: 'blur', type: "array" }
        ]
      }
    }
  },
  mounted() {
    GetWorkspaceUserList({workspaceId: +this.$route.query.workspaceId}).then((res) => {
      this.accessUsersMap = this.editUsersMap = res.users;
    })
    this.getData();
  },
  watch: {
    addProjectShow(val) {
      this.ProjectShow = val;
    },
    ProjectShow(val) {
      if (val) {
        this.originBusiness = this.projectDataCurrent.business;
      }
      this.$emit('show', val);
    },
  },
  methods: {
    getData() {
      const params = {
        parentKey: "p_develop_process",
        workspaceId: this.$route.query.workspaceId
      }
      GetDicList(params).then((res) => {
        this.devProcess = res.list;
        this.$emit('getDevProcessData', this.devProcess);
      })
    },
    Ok() {
      this.$refs.projectForm.validate((valid) => {
        if (valid) {
          this.$emit('confirm', this.projectDataCurrent);
          this.ProjectShow = false;
        } else {
          this.$Message.warning(this.$t('message.workflow.failedNotice'));
        }
      });
    },
    Cancel() {
      this.ProjectShow = false;
      this.projectData.business = this.originBusiness;
    },
    addTag(label) {
      if (this.projectDataCurrent.business) {
        this.projectDataCurrent.business += `,${label}`;
      } else {
        this.projectDataCurrent.business = label;
      }
    },
    deleteTag(label) {
      const tmpArr = this.projectDataCurrent.business.split(',');
      const index = tmpArr.findIndex((item) => item === label);
      tmpArr.splice(index, 1);
      this.projectData.business = tmpArr.toString();
    }
  },
};
</script>
<style lang="scss" scoped>
  .icon-style {
    vertical-align: middle;
    margin-left: 10px;
    font-size: 16px;
    color: black;
  }
</style>