<template>
  <div class="templete-editor">
    <!-- 非编辑模式 -->
    <el-row v-if="!isEdit">
      <slot name="showTemplete"></slot>
    </el-row>
    <!-- 编辑器模式 -->
    <el-row v-if="isEdit">
      <!-- 编辑器可见区域 -->
      <el-col class="tpl-viewer-container">
        <slot name="showComponent"></slot>
      </el-col>
      <!-- 编辑器表单区域 -->
      <el-col class="tpl-editor-container">
        <div>
          <el-form ref="tplForm" :model="value" v-bind="mergeTplForm" class="tpl-editor-form">
            <template v-for="(item, index) in tplFormElems">
              <!-- 包含group就说明是分组类型 -->
              <template v-if="item.group">
                <slot :name="'form_group_' + item.group.name" :data='item' v-if="isVisible(item.group.visible)">
                  <el-divider>{{item.group.label}}</el-divider>
                </slot>
                <div v-for="(gm, i) in item.elems" :key="'tplForm_group_' + index + '_' + i">
                  <FormItemEditor :binForm="value" :value="gm" :visable="isVisible(gm.visible)">
                    <template :slot="'form_item_' + gm.name">
                      <slot :name="'form_item_' + gm.name" :data="gm"></slot>
                    </template>
                  </FormItemEditor>
                </div>
                <slot :name="'form_group_' + item.group.name + '_bottom'" :data='item' v-if="isVisible(item.group.bottom?item.group.bottom.visible:false)">
                  <el-divider></el-divider>
                </slot>
              </template>
              <template v-else>
                <FormItemEditor :binForm="value" :value="item" :visable="isVisible(item.visible)">
                  <template :slot="'form_item_' + item.name">
                    <slot :name="'form_item_' + item.name" :data="item"></slot>
                  </template>
                </FormItemEditor>
              </template>
            </template>
            <!-- 按钮组 -->
            <el-form-item v-if="mergeTplForm.bottomBtns && mergeTplForm.bottomBtns.length > 0">
              <slot v-for="(item, index) in mergeTplForm.bottomBtns" :name="'bottomBtns_' + index" :data="mergeTplForm.bottomBtns">
                <el-button v-bind="item" v-on="item.events?item.events:null">{{item.label}}</el-button>
              </slot>
            </el-form-item>
          </el-form>
        </div>
      </el-col>
    </el-row>
  </div>
</template>
<script>
import Vue from 'vue'
import _ from 'lodash'
import FormItemEditor from './form-item-editor'
export default {
  name: 'TempleteEditor', // 模板编辑器
  components: {
    FormItemEditor
  },
  data() {
    return {
      defTplForm:{
        labelWidth: "30%", // 默认表单文字列宽
        bottomBtns: []
      }
    };
  },
  props: {
    isEdit: {
      type: Boolean,
      default () {
        return true;
      }
    },
    value: {
      type: Object,
      required: true,
      default () {
        return {}
      }
    },
    tplForm: {
      type: Object,
      default () {
        return {
        };
      }
    },
    tplFormElems: {
      type: Array,
      default () {
        return ;
      }
    }
  },
  computed: {
    mergeTplForm(){
      // 合并form表单props
      var props = {};
      _.merge(props, this.defTplForm, this.tplForm);
      return props;
    }
  },
  methods: {
    isVisible(vis){
      let typ = typeof vis;
      // 如果是布尔值
      if('boolean' === typ){
        return vis;
      }

      // 如果是表达式
      if(typeof vis === 'function'){
        let express = vis.call(this);
        return this.isVisible(express);
      }

      let isEmpty = _.isEmpty(vis);
      if(isEmpty){
        return true;
      }

      // 如果是表达式
      if(typeof vis === 'string'){
        let express = this.value[vis];
        return this.isVisible(express);
      }

      return true;
    }
  },
  created(){

  }
}
</script>
<style type="text/css" scoped>
.templete-editor {
  --pageViewHeight: 55rem;
  height: var(--pageViewHeight);
  --viwer-width: 65%;
}

.tpl-viewer-container {
  max-height: var(--pageViewHeight);
  overflow-y: auto;
  width: var(--viwer-width);
}

.tpl-editor-container {
  max-height: var(--pageViewHeight);
  overflow-y: auto;
  width: calc(100% - var(--viwer-width));
}

</style>
