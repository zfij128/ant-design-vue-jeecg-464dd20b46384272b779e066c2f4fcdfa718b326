<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="应用名称">
              <a-input placeholder="请输入应用名称" v-model="queryParam.appName"></a-input>
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="交互方法">
              <a-input placeholder="请输入交互方法" v-model="queryParam.interMethod"></a-input>
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="应用类型">
                <j-dict-select-tag placeholder="请选择应用类型" v-model="queryParam.appType" dictCode="apktype"/>
              </a-form-item>
            </a-col>
          </template>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              <a @click="handleToggleSearch" style="margin-left: 8px">
                {{ toggleSearchStatus ? '收起' : '展开' }}
                <a-icon :type="toggleSearchStatus ? 'up' : 'down'"/>
              </a>
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('apk信息表')">导出</a-button>
      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
      <!-- 高级查询区域 -->
      <j-super-query :fieldList="superFieldList" ref="superQueryModal" @handleSuperQuery="handleSuperQuery"></j-super-query>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel"><a-icon type="delete"/>删除</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down" /></a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{ selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div>

      <a-table
        ref="table"
        size="middle"
        :scroll="{x:true}"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        class="j-table-force-nowrap"
        @change="handleTableChange">

        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
        <template slot="imgSlot" slot-scope="text,record">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无图片</span>
          <img v-else :src="getImgView(text)" :preview="record.id" height="25px" alt="" style="max-width:80px;font-size: 12px;font-style: italic;"/>
        </template>
        <template slot="fileSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无文件</span>
          <a-button
            v-else
            :ghost="true"
            type="primary"
            icon="download"
            size="small"
            @click="downloadFile(text)">
            下载
          </a-button>
        </template>

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down" /></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a @click="handleDetail(record)">详情</a>
              </a-menu-item>
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>

      </a-table>
    </div>

    <ys-apk-info-modal ref="modalForm" @ok="modalFormOk"></ys-apk-info-modal>
  </a-card>
</template>

<script>

  import '@/assets/less/TableExpand.less'
  import { mixinDevice } from '@/utils/mixin'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import YsApkInfoModal from './modules/YsApkInfoModal'
  import {filterMultiDictText} from '@/components/dict/JDictSelectUtil'

  export default {
    name: 'YsApkInfoList',
    mixins:[JeecgListMixin, mixinDevice],
    components: {
      YsApkInfoModal
    },
    data () {
      return {
        description: 'apk信息表管理页面',
        // 表头
        columns: [
          {
            title: '#',
            dataIndex: '',
            key:'rowIndex',
            width:60,
            align:"center",
            customRender:function (t,r,index) {
              return parseInt(index)+1;
            }
          },
          {
            title:'应用包名称',
            align:"center",
            dataIndex: 'appPackageName'
          },
          {
            title:'应用名称',
            align:"center",
            dataIndex: 'appName'
          },
          {
            title:'应用描述',
            align:"center",
            dataIndex: 'appDesc'
          },
          {
            title:'大小',
            align:"center",
            dataIndex: 'appSize'
          },
          {
            title:'交互方法',
            align:"center",
            dataIndex: 'interMethod'
          },
          {
            title:'视频链接',
            align:"center",
            dataIndex: 'videoUrl'
          },
          {
            title:'标注链接',
            align:"center",
            dataIndex: 'sign'
          },
          {
            title:'应用下载链接',
            align:"center",
            dataIndex: 'downLoadurl',
            scopedSlots: {customRender: 'fileSlot'}
          },
          {
            title:'应用类型',
            align:"center",
            dataIndex: 'appType_dictText'
          },
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            fixed:"right",
            width:147,
            scopedSlots: { customRender: 'action' }
          }
        ],
        url: {
          list: "/ysapkinfo/ysApkInfo/list",
          delete: "/ysapkinfo/ysApkInfo/delete",
          deleteBatch: "/ysapkinfo/ysApkInfo/deleteBatch",
          exportXlsUrl: "/ysapkinfo/ysApkInfo/exportXls",
          importExcelUrl: "ysapkinfo/ysApkInfo/importExcel",
          
        },
        dictOptions:{},
        superFieldList:[],
      }
    },
    created() {
    this.getSuperFieldList();
    },
    computed: {
      importExcelUrl: function(){
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
      },
    },
    methods: {
      initDictConfig(){
      },
      getSuperFieldList(){
        let fieldList=[];
        fieldList.push({type:'string',value:'appPackageName',text:'应用包名称',dictCode:''})
        fieldList.push({type:'string',value:'appName',text:'应用名称',dictCode:''})
        fieldList.push({type:'string',value:'appDesc',text:'应用描述',dictCode:''})
        fieldList.push({type:'string',value:'appSize',text:'大小',dictCode:''})
        fieldList.push({type:'string',value:'interMethod',text:'交互方法',dictCode:''})
        fieldList.push({type:'string',value:'videoUrl',text:'视频链接',dictCode:''})
        fieldList.push({type:'string',value:'sign',text:'标注链接',dictCode:''})
        fieldList.push({type:'string',value:'downLoadurl',text:'应用下载链接',dictCode:''})
        fieldList.push({type:'int',value:'appType',text:'应用类型',dictCode:'apktype'})
        this.superFieldList = fieldList
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less';
</style>