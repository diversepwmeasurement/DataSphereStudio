<template>
  <Modal
    v-model="ProjectShow"
    :title="
      actionType === 'add'
        ? $t('message.common.projectDetail.createProject')
        : $t('message.common.projectDetail.editorProject')
    "
    :closable="false"
  >
    <Form
      :label-width="100"
      label-position="left"
      ref="projectForm"
      :model="projectDataCurrent"
      :rules="formValid"
      class="project_form"
    >
      <FormItem
        :label="$t('message.common.projectDetail.projectName')"
        prop="name"
      >
        <Input
          v-model="projectDataCurrent.name"
          :placeholder="$t('message.workflow.enterName')"
          :disabled="actionType === 'modify'"
        ></Input>
      </FormItem>
      <FormItem
        :label="$t('message.common.projectDetail.projectDesc')"
        prop="description"
      >
        <Input
          v-model="projectDataCurrent.description"
          type="textarea"
          :placeholder="
            $t('message.common.projectDetail.pleaseInputProjectDesc')
          "
        ></Input>
      </FormItem>
      <FormItem
        v-if="!framework"
        :label="$t('message.common.projectDetail.product')"
        prop="product"
      >
        <Input
          v-model="projectDataCurrent.product"
          :placeholder="$t('message.workflow.enterName')"
        >
        </Input>
      </FormItem>
      <FormItem
        :label="$t('message.common.projectDetail.appArea')"
        prop="applicationArea"
      >
        <Select
          v-model="projectDataCurrent.applicationArea"
          :placeholder="$t('message.common.projectDetail.selectAppArea')"
        >
          <Option
            v-for="(item, index) in applicationAreaMap"
            :label="item"
            :value="index"
            :key="index"
          />
        </Select>
      </FormItem>
      <FormItem
        :label="$t('message.common.projectDetail.devProcess')"
        prop="devProcessList"
      >
        <CheckboxGroup v-model="projectDataCurrent.devProcessList">
          <Checkbox
            v-for="item in devProcess"
            :label="item.dicValue"
            :key="item.dicKey"
          >
            <SvgIcon class="icon-style" :icon-class="item.icon" />
            <span>{{ item.dicName }}</span>
          </Checkbox>
        </CheckboxGroup>
      </FormItem>
      <FormItem
        v-if="framework"
        :label="$t('message.common.projectDetail.orchestratorMode')"
        prop="orchestratorModeList"
      >
        <CheckboxGroup v-model="projectDataCurrent.orchestratorModeList">
          <Checkbox
            v-for="item in orchestratorModeList.list"
            :label="item.dicKey"
            :key="item.dicKey"
          >
            <span class="icon-bar">
              <SvgIcon class="icon-style" :icon-class="item.icon" />
              <span style="margin-left: 10px">{{ item.dicName }}</span>
            </span>
          </Checkbox>
        </CheckboxGroup>
      </FormItem>
      <FormItem
        :label="$t('message.common.projectDetail.publishPermissions')"
        prop="releaseUsers"
      >
        <luban-select
          v-model="projectDataCurrent.releaseUsers"
          multiple
          filterable
          :placeholder="$t('message.common.projectDetail.userAllowedPublish')"
        >
          <Option
            v-for="(item, index) in workspaceUsers.releaseUsers"
            :label="item"
            :value="item"
            :key="index"
          />
        </luban-select>
      </FormItem>
      <FormItem
        :label="$t('message.common.projectDetail.editPermissions')"
        prop="editUsers"
      >
        <luban-select
          v-model="projectDataCurrent.editUsers"
          :disabled-tags="[projectDataCurrent.createBy]"
          multiple
          filterable
          :placeholder="$t('message.common.projectDetail.usersAllowedToEdit')"
        >
          <Option
            v-for="(item, index) in workspaceUsers.editUsers"
            :disabled="item == projectDataCurrent.createBy"
            :label="item"
            :value="item"
            :key="index"
          />
        </luban-select>
      </FormItem>
      <FormItem
        :label="$t('message.common.projectDetail.viewPermissions')"
        prop="accessUsers"
      >
        <luban-select
          v-model="projectDataCurrent.accessUsers"
          :disabled-tags="[projectDataCurrent.createBy]"
          multiple
          filterable
          :placeholder="$t('message.common.projectDetail.usersAllowedToView')"
        >
          <Option
            v-for="(item, index) in workspaceUsers.accessUsers"
            :disabled="item == projectDataCurrent.createBy"
            :label="item"
            :value="item"
            :key="index"
          />
        </luban-select>
      </FormItem>
      <FormItem
        :label="$t('message.common.projectDetail.business')"
        prop="business"
      >
        <we-tag
          :new-label="$t('message.common.projectDetail.addBusiness')"
          :tag-list="projectDataCurrent.business"
          @add-tag="addTag"
          @delete-tag="deleteTag"
        ></we-tag>
      </FormItem>
    </Form>
    <div slot="footer">
      <Button type="text" size="large" @click="Cancel">{{
        $t("message.workflow.cancel")
      }}</Button>
      <Button
        type="primary"
        size="large"
        :disabled="submiting"
        :loading="submiting"
        @click="Ok"
      >{{ $t("message.workflow.ok") }}</Button
      >
    </div>
  </Modal>
</template>
<script>
import tag from "@dataspherestudio/shared/components/tag/index.vue";
import lubanSelect from "@dataspherestudio/shared/components/select/index.vue";
import _ from "lodash";
import {
  GetDicList,
  CheckProjectNameRepeat
} from '@dataspherestudio/shared/common/service/apiCommonMethod.js';
export default {
  components: {
    "we-tag": tag,
    "luban-select": lubanSelect,
  },
  props: {
    projectData: {
      type: Object,
      default: null,
    },
    actionType: {
      type: String,
      default: "",
    },
    workspaceUsers: {
      type: Object,
      default: () => {
        return {
          accessUsers: [],
          releaseUsers: [],
          editUsers: []
        }
      },
    },
    applicationAreaMap: {
      type: Array,
      default: () => [],
    },
    framework: {
      type: Boolean,
      default: false,
    },
    orchestratorModeList: {
      type: Object,
      default: () => {}
    }
  },
  data() {
    return {
      ProjectShow: false,
      originBusiness: "",
      devProcess: [
        {
          id: 1,
          icon: "",
          title: this.$t('message.common.Development'),
          checked: false,
        },
        {
          id: 2,
          icon: "",
          title: this.$t('message.common.Production'),
          checked: false,
        },
      ],
      devProcessList: [],
      selectCompiling: [],
      projectDataCurrent: {},
      submiting: false,
    };
  },
  computed: {
    formValid() {
      let validateName = async (rule, value, callback) => {
        // 校验是否重名
        let repeat = false
        try {
          if (this.actionType === 'add') {
            const res = await CheckProjectNameRepeat(value)
            repeat = res.repeat
          }
        } catch (error) {
          //
        }
        if (repeat && this.actionType === 'add') {
          callback(
            new Error(this.$t("message.common.projectDetail.nameUnrepeatable"))
          );
        } else {
          callback();
        }
      };
      return {
        name: [
          {
            required: true,
            message: this.$t("message.workflow.enterName"),
            trigger: "blur",
          },
          { message: `${this.$t("message.workflow.nameLength")}64`, max: 64 },
          {
            type: "string",
            pattern: /^[a-zA-Z][a-zA-Z0-9_]*$/,
            message: this.$t("message.workflow.validNameDesc"),
            trigger: "blur",
          },
          { validator: validateName, trigger: "blur" },
        ],
        description: [
          {
            required: true,
            message: this.$t(
              "message.common.projectDetail.pleaseInputProjectDesc"
            ),
            trigger: "blur",
          },
        ],
        product: [
          {
            required: true,
            message: this.$t("message.common.projectDetail.selectProduct"),
            trigger: "change",
          },
        ],
        applicationArea: [
          {
            required: true,
            message: this.$t("message.common.projectDetail.selectAppArea"),
            trigger: "change",
            type: "number",
          },
        ],
        devProcessList: [
          {
            required: true,
            message: this.$t("message.common.projectDetail.pleaseSelect"),
            trigger: "blur",
            type: "array",
          },
        ],
        orchestratorModeList: [
          {
            required: true,
            message: this.$t("message.common.projectDetail.pleaseSelect"),
            trigger: "blur",
            type: "array",
          },
        ],
      };
    },
  },
  mounted() {
    this.getData();
  },
  watch: {
    ProjectShow(val) {
      if (val) {
        this.originBusiness = this.projectDataCurrent.business;
      }
    },
    projectData(value) {
      const cloneObj = _.cloneDeep(value);
      this.projectDataCurrent = cloneObj;
    },
  },
  methods: {
    getData() {
      const params = {
        parentKey: "p_develop_process",
        workspaceId: this.$route.query.workspaceId,
      };
      GetDicList(params).then((res) => {
        this.devProcess = res.list;
        this.$emit("getDevProcessData", this.devProcess);
      });
    },
    Ok() {
      this.$refs.projectForm.validate((valid) => {
        if (valid) {
          this.submiting = true;
          this.$emit("confirm", this.projectDataCurrent, (success) => {
            if (success) {
              this.$refs.projectForm.resetFields();
              this.ProjectShow = false;
            }
            this.submiting = false;
          });

        } else {
          this.submiting = false;
          this.$Message.warning(this.$t("message.workflow.failedNotice"));
        }
      });
    },
    Cancel() {
      this.ProjectShow = false;
      this.$refs.projectForm.resetFields();
      this.projectDataCurrent.business = this.originBusiness;
    },
    addTag(label) {
      if (this.projectDataCurrent.business) {
        this.projectDataCurrent.business += `,${label}`;
      } else {
        this.projectDataCurrent.business = label;
      }
    },
    deleteTag(label) {
      const tmpArr = this.projectDataCurrent.business.split(",");
      const index = tmpArr.findIndex((item) => item === label);
      tmpArr.splice(index, 1);
      this.projectDataCurrent.business = tmpArr.toString();
    },
    showProject(params) {
      this.ProjectShow = true
      // 新增只有一项自动勾选
      if (this.orchestratorModeList && this.orchestratorModeList.list.length === 1 && !params.name) {
        params.orchestratorModeList = [this.orchestratorModeList.list[0].dicKey]
      }
      this.projectDataCurrent = {...params}
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
.project_form {
  height: 60vh;
  overflow-y: auto;
  padding: 10px;
  max-height: 500px;
}
</style>
